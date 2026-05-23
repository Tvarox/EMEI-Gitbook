---
description: emei mandate subcommands — create, revoke.
---

# emei mandate

## `emei mandate create`

```bash
emei mandate create \
  --spend-cap 5000 \
  --counterparties 0xA,0xB \
  --categories compute,data-services \
  --valid-from <unix_seconds> \
  --valid-until <unix_seconds>
```

| Flag | Required | Notes |
|---|---|---|
| `--spend-cap` | ✅ | Human-readable; converted to wei |
| `--counterparties` | ✅ | 1–50 addresses, comma-separated |
| `--categories` | ✅ | 1–20 strings, comma-separated |
| `--valid-from` | ✅ | Unix seconds |
| `--valid-until` | ✅ | Unix seconds; > `--valid-from` |

**Auth:** ✅. Caller becomes the mandate's payer.

### Output

```json
{
  "mandate_id": 7,
  "tx_hash": "0x...",
  "remaining_cap": "5000000000000000000000",
  "status": "ACTIVE"
}
```

## `emei mandate revoke <id>`

Revoke an active mandate. Terminal — cannot be re-activated.

**Auth:** ✅. Must be the mandate's payer.

## See also

- [Mandates](../concepts/mandates.md)
- [HTTP API: Mandates](../api/mandates.md)
- [Set up mandate auto-collection](../guides/setup-mandate.md)
