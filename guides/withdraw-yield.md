---
description: Withdraw vault balance plus accrued yield from EMEISettlement back to your wallet.
---

# Withdraw earned yield

Settled mUSD lives in a yield vault. Withdrawals move funds (principal + yield) back to your wallet as plain mUSD.

## Prerequisites

- You are the payee on at least one settled invoice.
- The Facilitator is reachable.

## Steps

### 1. Check your balance

```bash
emei balance 0xYOUR_ADDRESS
```

Output:

```json
{
  "address": "0xYOUR_ADDRESS",
  "balance":       "100037420000000000000",
  "accrued_yield":  "37420000000000000",
  "vault": "MUSD_REBASE"
}
```

`balance` is your full claim, including yield. `accrued_yield` is what you've earned since first deposit.

### 2. Withdraw

```bash
emei withdraw 50
```

This calls `EMEISettlement.withdraw(50e18)`. Funds land back in your wallet as plain mUSD.

To withdraw everything, pass the full balance:

```bash
emei withdraw $(emei balance 0xYOUR_ADDRESS | jq -r '.balance' | awk '{ printf("%d", $1 / 1e18) }')
```

### 3. Confirm

```bash
emei balance 0xYOUR_ADDRESS
# balance and accrued_yield should reflect the withdrawal
```

## Notes

- Withdrawals come from your **own** vault — `EMEISettlement.withdraw` checks `msg.sender`'s balance only. There's no way to withdraw someone else's funds.
- Insufficient balance reverts with `InsufficientVaultBalance(payee, requested, available)`.
- After withdrawal, the withdrawn portion stops earning yield (it's now plain mUSD in your wallet, but mUSD is still rebasing — you continue earning at the wallet level).

## See also

- [Settlement & yield](../concepts/settlement.md)
- [HTTP API: Settlement](../api/settlement.md)
- [EMEISettlement contract](../contracts/emei-settlement.md)
