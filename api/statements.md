---
description: Query indexed contract events from the Facilitator's local SQLite event store.
---

# Statements

The Event Indexer streams every emitted event from the five core contracts into a local SQLite database. `GET /emei/statement` is the read API for that database.

→ Conceptual model: [Event indexer & SQLite schema](../architecture/indexer.md).

---

## GET /emei/statement

Query indexed events with filters.

**Auth:** ❌

### Query parameters

| Param | Type | Description |
|---|---|---|
| `payer` | address | Filter to events involving this payer |
| `issuer` | address | Filter to events involving this issuer |
| `invoice_id` | uint256 | Filter to one invoice |
| `mandate_id` | uint256 | Filter to one mandate |
| `event_type` | string | One of `InvoiceCreated`, `InvoicePresented`, `InvoicePaid`, `InvoiceOverdue`, `InvoiceRejected`, `MandateCreated`, `MandateCollected`, `MandateRevoked`, `SettlementExecuted`, `WithdrawalExecuted`, `MerkleRootPosted`, `ReputationChecked` |
| `from_block` | uint256 | Inclusive lower bound |
| `to_block` | uint256 | Inclusive upper bound |
| `limit` | uint256 | Default 100, max 1000 |
| `cursor` | string | Pagination cursor returned in the previous response |

Filters compose with `AND`.

### Response (200 OK)

```json
{
  "events": [
    {
      "event_type": "InvoicePaid",
      "block_number": 12345678,
      "tx_hash": "0x...",
      "log_index": 3,
      "timestamp": 1716000456,
      "data": {
        "invoice_id": 1,
        "payer": "0xPAYER",
        "amount": "100000000000000000000",
        "settlement_proof": "0xabc..."
      }
    }
  ],
  "next_cursor": "eyJibG9jayI6MTIzNDU2NzgsImxvZyI6M30="
}
```

When `next_cursor` is present, more results exist. Pass it back via `?cursor=...` to continue.

---

## See also

- [Event indexer](../architecture/indexer.md)
- [Events reference](../contracts/events.md)
- [Query the event statement (guide)](../guides/query-events.md)
