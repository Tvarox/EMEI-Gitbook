---
description: HTTP endpoint for verifying receipt Merkle inclusion against the on-chain anchor.
---

# Receipts & verification

→ Conceptual model: [Receipt anchoring](../concepts/receipts.md).

---

## GET /emei/verify/{id}

Verify that an invoice's receipt is included in the anchored Merkle batch on-chain.

**Auth:** ❌

### Path parameters

| Name | Type |
|---|---|
| `id` | uint256 (invoice ID) |

### Response (200 OK)

```json
{
  "invoice_id": 1,
  "batch_number": 42,
  "merkle_root": "0xabc...",
  "leaf": "0xdef...",
  "proof": ["0x...", "0x..."],
  "verified": true,
  "anchored_at_block": 12345678
}
```

| Field | Description |
|---|---|
| `batch_number` | Which batch contains the receipt |
| `merkle_root` | The on-chain root for that batch |
| `leaf` | This invoice's receipt commitment |
| `proof` | Sibling hashes needed to reconstruct the root |
| `verified` | Result of calling `EMEIReceipt.verifyInclusion` on-chain |
| `anchored_at_block` | Block where the root was posted |

`verified: true` means: the leaf, combined with the proof, hashes to the on-chain root. This is independent of the Facilitator — anyone can re-derive the leaf and re-call `verifyInclusion` themselves.

### Common errors

| HTTP | Code | When |
|---|---|---|
| 404 | `INVOICE_NOT_FOUND` | No invoice with that ID |
| 409 | `RECEIPT_NOT_BATCHED_YET` | Invoice is `PAID` but the next batch hasn't run (give it ≤30s) |
| 409 | `INVOICE_NOT_PAID` | Invoice is not in terminal `PAID` state |

---

## See also

- [Receipt anchoring](../concepts/receipts.md)
- [EMEIReceipt contract](../contracts/emei-receipt.md)
- [Verify a receipt's Merkle inclusion (guide)](../guides/verify-receipt.md)
