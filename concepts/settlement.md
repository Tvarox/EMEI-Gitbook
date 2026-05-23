---
description: How EMEI moves money — token transfers, USDC→mUSD swap, vault routing, and yield accrual.
---

# Settlement & yield

`EMEISettlement` is the contract that actually moves tokens. `EMEIInvoice` calls `settle()` on it whenever an invoice transitions to `PAID` (via either `pay` or `collect`).

Two assets are supported:

- **mUSD** — settled directly. 18 decimals.
- **USDC** — auto-swapped to mUSD inside `settle()`. 6 decimals.

Settled funds are routed to a yield-bearing vault selected per-payee.

## The settle path

```mermaid
flowchart LR
    P[Payer wallet] -->|approve(settlement, amount)| ALW[ERC20 allowance]
    INV[EMEIInvoice.pay/collect] -->|settle| SET[EMEISettlement]
    SET -->|transferFrom| MUSD[mUSD]
    SET -->|or transferFrom + swap| USDC[USDC]
    USDC -->|×1e12, max 1% slippage| MUSD
    SET -->|deposit| VAULT[Vault: MUSD_REBASE / SUSDE]
    VAULT -.yield accrues automatically.-> VAULT
```

### Direct mUSD path

```
Payer → transferFrom(payer, settlement, amount) → mUSD
                                                    ↓
                                          deposit into payee's vault
```

### USDC cross-asset path

```
Payer → transferFrom(payer, settlement, amount_usdc) → USDC
                                                    ↓
                                          swap: amount_usdc × 1e12 → expected_musd
                                          require output ≥ expected_musd × 0.99   (1% slippage cap)
                                                    ↓
                                          deposit expected_musd into payee's vault
```

The `×1e12` is decimal normalization (USDC has 6 decimals, mUSD has 18). The 1% slippage cap is configurable by the contract owner.

## Vault preference

Each payee chooses a vault destination once. Default is `MUSD_REBASE`.

| Vault type | What it is | Token used |
|---|---|---|
| `MUSD_REBASE` (default) | mUSD's native rebasing — `balanceOf` grows over time as `rebaseIndex` advances | mUSD |
| `SUSDE` | Alternative yield vault | mUSD wrapped into sUSDe |

Set preference:

```bash
emei wallet set-vault SUSDE          # via CLI (when implemented)
# or directly on-chain:
cast send --rpc-url $RPC_URL --private-key $YOUR_KEY \
  0xfdCb7bA077069A7Da44711Ee6bdB49174AFA4dD0 \
  "setVaultPreference(uint8)" 1   # 0=MUSD_REBASE, 1=SUSDE
```

## Yield accrual

For `MUSD_REBASE`:

- `MockmUSD` carries a configurable `apyBps` and a `rebaseIndex` that advances on `rebase()` (or lazily on transfers).
- A payee's `balanceOf` = `shares × rebaseIndex / 1e18`. Shares are constant; `rebaseIndex` grows.
- `accruedYield = currentBalance − initialDeposits`. Read it at `getAccruedYield(payee)`.

```bash
emei balance 0xPAYEE
```

```json
{
  "address": "0xPAYEE",
  "balance":        "100037420000000000000",   // 100 + 0.0374 yield
  "accrued_yield":  "37420000000000000",
  "vault": "MUSD_REBASE"
}
```

## Withdrawing

```bash
emei withdraw 50
```

Calls `EMEISettlement.withdraw(50e18)`. Funds leave the vault and land in the payee's wallet as plain mUSD.

The HTTP equivalent:

```bash
curl -X POST http://localhost:8080/emei/withdraw \
  -H "X-Private-Key: 0xYOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{"amount": "50000000000000000000"}'
```

## Authorization

| Function | `msg.sender` constraint |
|---|---|
| `settle` | **`EMEIInvoice` only.** Reverts otherwise. |
| `setVaultPreference` | Any address (sets *their own* preference) |
| `withdraw` | Any address (withdraws *their own* balance) |
| `getVaultBalance`, `getAccruedYield` | Anyone |
| Slippage / vault config | Owner only |

## Slippage protection

USDC→mUSD swap requires:

```
output_musd ≥ amount_usdc × 1e12 × (10000 - maxSlippageBps) / 10000
```

Default `maxSlippageBps = 100` (1%). Below that, `settle` reverts and the invoice does not transition to `PAID`.

## Events

```solidity
event SettlementExecuted(uint256 indexed invoiceId, uint256 amount, address inputAsset, address outputAsset, VaultType vault);
event WithdrawalExecuted(address indexed payee, uint256 amount, VaultType vault);
```

## Common questions

**Can I bypass the vault?**
Not at settlement time. The contract routes settled funds to your vault preference. To "skip yield", set your preference and `withdraw` to a regular wallet immediately.

**What happens if my vault preference changes mid-flow?**
New settlements use the new preference. Existing balances stay in the old vault until withdrawn.

**Is the swap on-chain?**
Yes. `EMEISettlement` performs the conversion using `MockmUSD`'s mint capability on testnet (a 1:1 + ×1e12 normalization). On mainnet this would route through a real DEX.

## Next steps

- [Withdraw earned yield](../guides/withdraw-yield.md)
- [Pay with USDC](../guides/pay-with-usdc.md)
- [EMEISettlement (contracts reference)](../contracts/emei-settlement.md)

## See also

- [Receipt anchoring](receipts.md) — what gets anchored after settle
- [Mocks: MockmUSD](../contracts/mocks.md)
