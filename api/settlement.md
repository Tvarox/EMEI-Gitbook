---
description: HTTP endpoints for vault balance lookups and withdrawals.
---

# Settlement & withdrawal

Two endpoints: read a payee's vault balance (including accrued yield), and withdraw funds from the vault to the payee's wallet.

→ Conceptual model: [Settlement & yield](../concepts/settlement.md).

---

## GET /emei/balance/{address}

Read a payee's current vault balance and accrued yield.

**Auth:** ❌

### Response (200 OK)

```json
{
  "address": "0xPAYEE",
  "balance": "100037420000000000000",
  "accrued_yield": "37420000000000000",
  "vault": "MUSD_REBASE"
}
```

| Field | Description |
|---|---|
| `balance` | Current vault balance, including yield |
| `accrued_yield` | `balance − sum_of_deposits`. Strictly cumulative; never decreases except via withdrawal |
| `vault` | `MUSD_REBASE` (default) or `SUSDE` |

---

## POST /emei/withdraw

Withdraw mUSD from the caller's vault.

**Auth:** ✅ `X-Private-Key` — caller is the payee.

### Request

```json
{ "amount": "50000000000000000000" }
```

### Response (200 OK)

```json
{
  "tx_hash": "0x...",
  "amount": "50000000000000000000",
  "remaining_balance": "50037420000000000000",
  "vault": "MUSD_REBASE"
}
```

### Common errors

| HTTP | Code | When |
|---|---|---|
| 400 | `INVALID_AMOUNT` | `amount = 0` or non-numeric |
| 409 | `INSUFFICIENT_BALANCE` | Caller's vault balance < `amount` |

---

## See also

- [Settlement & yield](../concepts/settlement.md)
- [Withdraw earned yield](../guides/withdraw-yield.md)
- [EMEISettlement contract](../contracts/emei-settlement.md)
