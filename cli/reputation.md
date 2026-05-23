---
description: emei reputation — query an address's current Bay8004 score.
---

# emei reputation

```bash
emei reputation 0xADDRESS
```

**Auth:** ❌.

## Output

```json
{
  "address": "0xADDRESS",
  "score": 500,
  "threshold": 0,
  "passes_gate": true
}
```

`threshold` reflects the current `minReputation` setting on `EMEIInvoice`. Returns `0` for unregistered addresses.

## See also

- [Reputation gate](../concepts/reputation.md)
- [HTTP API: Identity](../api/identity.md)
- [Bay8004 contract](../contracts/bay8004.md)
