---
description: Query the Facilitator's indexed event log to reconstruct invoice and mandate history.
---

# Query the event statement

Every contract event is streamed into the Facilitator's local SQLite by the Event Indexer. `GET /emei/statement` is the read API.

## Prerequisites

- The Facilitator running and indexing.

## Common queries

### All events for an invoice

```bash
curl "http://localhost:8080/emei/statement?invoice_id=1"
```

Returns the full audit trail: `InvoiceCreated → InvoicePresented → InvoicePaid → MerkleRootPosted`.

### All events for a payer

```bash
curl "http://localhost:8080/emei/statement?payer=0xPAYER&limit=100"
```

### Failed mandate collections

```bash
curl "http://localhost:8080/emei/statement?event_type=CollectionRejected"
```

Useful for debugging "why isn't this invoice auto-collecting?".

### Pagination

When the response includes `next_cursor`, pass it back:

```bash
curl "http://localhost:8080/emei/statement?payer=0xPAYER&cursor=eyJibG9j..."
```

## Tips

- The API filters compose with `AND`. To OR conditions, issue separate queries and merge client-side.
- The Indexer runs continuously. Newly mined events appear within ≤1 block.
- For high-volume queries, point at the SQLite directly (read-only) — see [Event indexer & SQLite schema](../architecture/indexer.md).

## See also

- [HTTP API: Statements](../api/statements.md)
- [Events reference](../contracts/events.md)
- [Event indexer](../architecture/indexer.md)
