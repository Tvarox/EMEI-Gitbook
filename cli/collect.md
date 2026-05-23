---
description: emei collect — manually trigger a mandate-based collection.
---

# emei collect

Manually trigger mandate-based collection of an invoice. Permissionless — useful for debugging or if you don't want to wait the ≤10s the Auto-Collector takes.

```bash
emei collect <invoice_id> <mandate_id>
```

**Auth:** ❌ — the Facilitator signs with `OPERATOR_PRIVATE_KEY`.

## Output

```json
{
  "invoice_id": 1,
  "mandate_id": 7,
  "tx_hash": "0x...",
  "status": "PAID",
  "remaining_cap": "4900000000000000000000"
}
```

## Errors

See the [collect endpoint errors](../api/invoices.md#post-emeicollect) — same set: `MANDATE_NOT_ACTIVE`, `MANDATE_EXPIRED`, `COUNTERPARTY_NOT_APPROVED`, `CATEGORY_NOT_APPROVED`, `INSUFFICIENT_MANDATE_CAP`, `INVALID_STATUS_TRANSITION`, `REPUTATION_TOO_LOW`.

## See also

- [Mandates](../concepts/mandates.md)
- [HTTP API: POST /emei/collect](../api/invoices.md#post-emeicollect)
- [Background services: Auto-Collector](../architecture/background-services.md)
