---
description: How the Facilitator indexes contract events into SQLite to power fast statement queries and read endpoints.
---

# Event indexer & SQLite schema

The Event Indexer is the long-running Tokio task that streams events from the five core contracts into a local SQLite database.

## Why SQLite?

- **Single-binary deploy.** No external dependency.
- **Read-heavy workload.** `/emei/statement` is the hot path; SQLite handles thousands of concurrent reads.
- **Cheap durability.** WAL mode + periodic checkpoints.
- **Embeddable.** Tools can query the DB directly (read-only) for analytics.

## Watched contracts

The Indexer subscribes to the address set:

- `EMEIInvoice`
- `EMEIMandate`
- `EMEISettlement`
- `Bay8004`
- `EMEIReceipt`

Each event is decoded with the `alloy-sol-types`-generated bindings.

## Schema (canonical, may evolve)

```sql
CREATE TABLE events (
    id              INTEGER PRIMARY KEY AUTOINCREMENT,
    event_type      TEXT NOT NULL,            -- e.g. 'InvoicePaid'
    contract        TEXT NOT NULL,            -- contract address
    block_number    INTEGER NOT NULL,
    block_timestamp INTEGER NOT NULL,
    tx_hash         TEXT NOT NULL,
    log_index       INTEGER NOT NULL,
    data            TEXT NOT NULL,            -- JSON-serialized decoded event
    UNIQUE (block_number, log_index)
);

CREATE INDEX idx_events_block       ON events(block_number);
CREATE INDEX idx_events_event_type  ON events(event_type);

CREATE TABLE invoices (
    invoice_id        INTEGER PRIMARY KEY,
    issuer            TEXT NOT NULL,
    payer             TEXT NOT NULL,
    amount            TEXT NOT NULL,
    asset             TEXT NOT NULL,
    status            TEXT NOT NULL,
    collection_mode   TEXT NOT NULL,
    settlement_proof  TEXT,
    receipt           TEXT,
    presented_at      INTEGER,
    created_at        INTEGER NOT NULL
);

CREATE INDEX idx_invoices_payer  ON invoices(payer);
CREATE INDEX idx_invoices_issuer ON invoices(issuer);
CREATE INDEX idx_invoices_status ON invoices(status);

CREATE TABLE receipts (
    invoice_id       INTEGER PRIMARY KEY,
    leaf             TEXT NOT NULL,
    batch_number     INTEGER,                  -- NULL until included in a batch
    proof            TEXT,                     -- JSON array, set when batched
    FOREIGN KEY (invoice_id) REFERENCES invoices(invoice_id)
);

CREATE TABLE indexer_checkpoint (
    contract     TEXT PRIMARY KEY,
    last_block   INTEGER NOT NULL
);
```

## Resync

If the database is deleted, the Indexer resyncs from genesis (or `INDEXER_START_BLOCK` if set). For a fresh sync from a recent block, set `INDEXER_START_BLOCK` to a known-good height.

## Querying directly

Read-only access is safe:

```bash
sqlite3 ./emei.db "SELECT event_type, COUNT(*) FROM events GROUP BY event_type;"
```

For application code, prefer `GET /emei/statement` — its filter/cursor semantics are stable; the schema is not.

## See also

- [Background services](background-services.md)
- [HTTP API: Statements](../api/statements.md)
- [Events reference](../contracts/events.md)
