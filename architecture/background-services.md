---
description: The four background services that run inside the Facilitator — Auto-Collector, Overdue Scanner, Receipt Batcher, Event Indexer.
---

# Background services

The Facilitator runs four background Tokio tasks alongside the HTTP server. Each is responsible for one piece of protocol operation.

| Service | Interval | Signs with | Purpose |
|---|---|---|---|
| **Event Indexer** | continuous | — (read-only) | Streams events from all 5 contracts into SQLite |
| **Auto-Collector** | every 10s | `OPERATOR_PRIVATE_KEY` | Calls `EMEIInvoice.collect` on due invoices that match an active mandate |
| **Overdue Scanner** | every 60s | `OPERATOR_PRIVATE_KEY` | Calls `EMEIInvoice.markOverdue` on past-due `PRESENTED` invoices |
| **Receipt Batcher** | every 30s | `RECEIPT_BATCHER_PRIVATE_KEY` | Builds a Merkle tree of unbatched receipts and calls `EMEIReceipt.postMerkleRoot` |

## Auto-Collector

```
every 10 seconds:
  1. Query SQLite for invoices in (PRESENTED, OVERDUE) with collection_mode=MANDATE
  2. For each invoice, find the best-fit ACTIVE mandate (matching counterparty + category + cap)
  3. If due (per Terms): call collect(invoiceId, mandateId)
  4. On revert: log `CollectionRejected` reason; do not retry until next interval
```

If no mandate matches, the invoice stays untouched; the *issuer* and *payer* are responsible for setting the right mandate up.

## Overdue Scanner

```
every 60 seconds:
  1. Query SQLite for invoices in PRESENTED with dueDate < now
  2. For each, call markOverdue(invoiceId)
```

`markOverdue` is permissionless — anyone can call it. The Scanner is just a convenience operator.

## Receipt Batcher

```
every 30 seconds:
  1. Query SQLite for paid invoices not yet assigned to a batch
  2. If none, skip (no on-chain call)
  3. Build leaves: keccak256(abi.encode(invoiceId, payer, amount, settlementProof))
  4. Construct a Merkle tree, compute root
  5. Call postMerkleRoot(latestBatch + 1, root)
  6. On success: assign batch number to the included receipts in SQLite
```

## Event Indexer

```
on startup:
  - Determine the highest indexed block from SQLite
  - Replay missed blocks via eth_getLogs
  - Subscribe to new logs (or poll) for the 5 contracts
  - Insert decoded events into SQLite, transactionally
```

The Indexer maintains the local view that powers fast `/emei/statement` queries and read-side endpoints like `GET /emei/balance/{address}`.

## Failure modes

- **RPC down.** All services log and back off. The Indexer resumes from its checkpoint when RPC returns.
- **Operator key out of MNT.** Auto-Collector and Overdue Scanner fail; user-driven flows still work. Refill the operator wallet.
- **SQLite locked.** Should not occur with `sqlx`'s pool, but if it does, services queue.

## See also

- [System overview](system-overview.md)
- [Event indexer](indexer.md)
- [Configuration & environment](../ops/configuration.md)
