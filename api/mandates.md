---
description: HTTP endpoints for creating and revoking payer mandates.
---

# Mandates

Two endpoints: create a scoped mandate, and revoke one. Mandate validation happens on-chain in [`EMEIMandate.validateAndDecrement`](../contracts/emei-mandate.md) at collection time.

→ Conceptual model: [Mandates](../concepts/mandates.md).

---

## POST /emei/mandate

Create a new mandate. Caller becomes the `payer`.

**Auth:** ✅ `X-Private-Key`

### Request

```json
{
  "spend_cap": "5000000000000000000000",
  "approved_counterparties": ["0xAGENT_A", "0xAGENT_B"],
  "approved_categories": ["compute", "data-services"],
  "valid_from": 1716000000,
  "valid_until": 1718592000
}
```

| Field | Type | Required | Constraints |
|---|---|---|---|
| `spend_cap` | uint256 string | ✅ | > 0 |
| `approved_counterparties` | address[] | ✅ | 1–50 addresses |
| `approved_categories` | string[] | ✅ | 1–20 strings |
| `valid_from` | unix seconds | ✅ | ≥ now (with reasonable clock skew) |
| `valid_until` | unix seconds | ✅ | > `valid_from` |

### Response (201 Created)

```json
{
  "mandate_id": 7,
  "tx_hash": "0x...",
  "payer": "0xCALLER",
  "spend_cap": "5000000000000000000000",
  "remaining_cap": "5000000000000000000000",
  "status": "ACTIVE"
}
```

### Common errors

| HTTP | Code | When |
|---|---|---|
| 400 | `INVALID_MANDATE_PARAMS` | Empty arrays, > 50 counterparties, > 20 categories, bad time window, zero spend cap |
| 401 | `UNAUTHORIZED` | Missing/invalid key |

---

## DELETE /emei/mandate/{id}

Revoke an active mandate. Terminal — once revoked, a mandate cannot be re-activated.

**Auth:** ✅ `X-Private-Key` — must equal the mandate's `payer`.

### Response (200 OK)

```json
{
  "mandate_id": 7,
  "tx_hash": "0x...",
  "status": "REVOKED"
}
```

### Common errors

| HTTP | Code | When |
|---|---|---|
| 401 | `UNAUTHORIZED` | Caller is not the mandate's payer |
| 404 | `MANDATE_NOT_FOUND` | No mandate with that ID |
| 409 | `MANDATE_NOT_ACTIVE` | Already `EXHAUSTED`, `EXPIRED`, or `REVOKED` |

---

## See also

- [Mandates](../concepts/mandates.md)
- [Set up mandate auto-collection](../guides/setup-mandate.md)
- [EMEIMandate contract](../contracts/emei-mandate.md)
