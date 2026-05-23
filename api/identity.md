---
description: HTTP endpoints for ERC-8004 identity registration and reputation lookup.
---

# Identity

Two endpoints: register an address in the ERC-8004 registry, and look up an address's current reputation score.

→ Conceptual model: [Reputation gate](../concepts/reputation.md).

---

## POST /emei/register

Register the caller's address in the ERC-8004 identity registry.

**Auth:** ✅ `X-Private-Key`

### Request

```json
{ "score": 500 }
```

| Field | Type | Required | Default | Constraints |
|---|---|---|---|---|
| `score` | uint256 | no | 100 | 0–10000. Testnet convenience — the production registry would not allow self-asserted scores. |

### Response (201 Created)

```json
{
  "address": "0xCALLER",
  "tx_hash": "0x...",
  "score": 500
}
```

### Common errors

| HTTP | Code | When |
|---|---|---|
| 401 | `UNAUTHORIZED` | Missing/invalid `X-Private-Key` |
| 409 | `ALREADY_REGISTERED` | Address already exists in the registry |

---

## GET /emei/reputation/{address}

Read an address's current reputation score from `Bay8004`.

**Auth:** ❌

### Response (200 OK)

```json
{
  "address": "0xQUERIED",
  "score": 500,
  "threshold": 0,
  "passes_gate": true
}
```

`threshold` reflects the current `minReputation` setting on `EMEIInvoice`. Returns `0` for unregistered addresses.

---

## See also

- [Reputation gate](../concepts/reputation.md)
- [Bay8004 contract](../contracts/bay8004.md)
- [CLI: emei wallet](../cli/wallet.md)
