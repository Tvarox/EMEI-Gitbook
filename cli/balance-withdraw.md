---
description: emei balance and emei withdraw — read vault balance and pull funds back to your wallet.
---

# emei balance & withdraw

## `emei balance [address]`

Read vault balance and accrued yield.

```bash
emei balance 0xPAYEE
```

**Auth:** ❌. Defaults to the caller's address if `EMEI_PRIVATE_KEY` is set and `[address]` is omitted.

### Output

```json
{
  "address": "0xPAYEE",
  "balance":        "100037420000000000000",
  "accrued_yield":  "37420000000000000",
  "vault": "MUSD_REBASE"
}
```

## `emei withdraw <amount>`

Withdraw `<amount>` from your own vault to your wallet.

```bash
emei withdraw 50
```

**Auth:** ✅. Withdraws the caller's funds only.

### Output

```json
{
  "tx_hash": "0x...",
  "amount":  "50000000000000000000",
  "remaining_balance": "50037420000000000000"
}
```

## See also

- [Settlement & yield](../concepts/settlement.md)
- [Withdraw earned yield (guide)](../guides/withdraw-yield.md)
- [HTTP API: Settlement](../api/settlement.md)
