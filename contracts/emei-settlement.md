---
description: EMEISettlement contract reference — token transfers, USDC swap, vault routing, and withdrawal.
---

# EMEISettlement

Moves tokens, swaps USDC→mUSD with slippage protection, and routes settled funds to a yield vault.

| | |
|---|---|
| Address (Mantle Sepolia) | [`0xfdCb7bA077069A7Da44711Ee6bdB49174AFA4dD0`](https://sepolia.mantlescan.xyz/address/0xfdCb7bA077069A7Da44711Ee6bdB49174AFA4dD0) |
| Source | [`src/EMEISettlement.sol`](https://github.com/Tvarox/EMEI-Contracts/blob/main/src/EMEISettlement.sol) |
| Interface | [`src/interfaces/IEMEISettlement.sol`](https://github.com/Tvarox/EMEI-Contracts/blob/main/src/interfaces/IEMEISettlement.sol) |

→ Conceptual model: [Settlement & yield](../concepts/settlement.md).

## Types

```solidity
enum VaultType { MUSD_REBASE, SUSDE }
```

## Functions

### `settle`

```solidity
function settle(
    uint256 invoiceId,
    address payer,
    address payee,
    uint256 amount,
    address asset
) external returns (bytes32 proof);
```

**Caller:** `EMEIInvoice` only. Reverts otherwise.

Transfers `amount` of `asset` from `payer` to the contract, swaps to mUSD if `asset == MockUSDC`, and deposits into `payee`'s preferred vault. Returns a 32-byte settlement proof.

**Reverts:**
| Error | When |
|---|---|
| `Unauthorized()` | Caller is not `EMEIInvoice` |
| `TransferFailed(token, from, to, amount)` | ERC-20 transferFrom returned false / reverted |
| `SwapFailed(expected, received)` | Slippage exceeded on USDC→mUSD swap |

**Events:** `SettlementExecuted`. May emit `VaultDepositFailed` if the vault deposit fails (proof is still returned and funds are recoverable by `withdraw`).

---

### `setVaultPreference`

```solidity
function setVaultPreference(VaultType vault) external;
```

Caller sets *their own* vault preference. Default is `MUSD_REBASE`.

**Events:** `VaultPreferenceSet`.

---

### `withdraw`

```solidity
function withdraw(uint256 amount) external;
```

Caller withdraws `amount` mUSD from their vault to their wallet. Withdrawal includes any accrued yield.

**Reverts:** `InsufficientVaultBalance(payee, requested, available)`.

**Events:** `WithdrawalExecuted`.

---

### Views

```solidity
function getVaultBalance(address payee) external view returns (uint256);
function getAccruedYield(address payee) external view returns (uint256);
```

`getAccruedYield = getVaultBalance − sum_of_deposits`. Strictly cumulative.

---

### Owner-only setters

```solidity
function setSwapRouter(address router) external;
function setSlippageTolerance(uint256 bps) external;  // default 100 (1%)
function setEscrow(address escrow) external;
```

## Events

```solidity
event SettlementExecuted(uint256 indexed invoiceId, uint256 amount, address inputAsset, address outputAsset, VaultType destinationVault);
event VaultDepositFailed(uint256 indexed invoiceId, uint256 amount);
event VaultPreferenceSet(address indexed payee, VaultType vault);
event WithdrawalExecuted(address indexed payee, uint256 amount, VaultType vault);
```

## Custom errors

```solidity
error TransferFailed(address token, address from, address to, uint256 amount);
error SwapFailed(uint256 expected, uint256 received);
error InsufficientVaultBalance(address payee, uint256 requested, uint256 available);
error Unauthorized();
```

## See also

- [Settlement & yield](../concepts/settlement.md)
- [HTTP API: Settlement](../api/settlement.md)
- [Withdraw earned yield (guide)](../guides/withdraw-yield.md)
