---
description: Register an ERC-8004 identity with the emei CLI.
---

# emei wallet

## `emei wallet create`

Register the caller's address in the ERC-8004 registry.

```bash
emei wallet create [--score N]
```

| Flag | Default | Description |
|---|---|---|
| `--score N` | `100` | Initial reputation score (testnet only — ignored on a production registry) |

**Auth:** ✅ requires `EMEI_PRIVATE_KEY`.

### Output

```json
{
  "address": "0xCALLER",
  "tx_hash": "0x...",
  "score": 100
}
```

### Errors

- `ALREADY_REGISTERED` — address already exists in the registry.

## See also

- [HTTP API: Identity](../api/identity.md)
- [Reputation gate](../concepts/reputation.md)
