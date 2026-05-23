---
description: Logs, metrics, and health checks for a production EMEI Facilitator.
---

# Monitoring & logs

## Health endpoint

```
GET /health  →  {"status": "ok"}
```

Hook this into your liveness probe.

## Logs

The Facilitator uses `tracing`. Configure with `RUST_LOG`:

| Setting | What you'll see |
|---|---|
| `emei_facilitator=info` | Request/response, service ticks, tx hashes (default) |
| `emei_facilitator=debug` | RPC calls, decoded events, mandate match attempts |
| `emei_facilitator=trace` | Wire-level details — verbose |

In production, ship logs to your aggregator (Loki, Datadog, CloudWatch).

## Metrics to watch

| Metric | What it tells you |
|---|---|
| HTTP error rate | Bad requests, decoded reverts, RPC failures |
| Auto-Collector failures | `CollectionRejected` events accumulating → consumer mandate misconfigured |
| Receipt Batcher cadence | Should fire every ~30s when there is activity |
| Operator wallet balance | Below ~0.1 MNT and Auto-Collector / Scanner will start failing |
| Indexer lag | Latest indexed block vs. RPC head — lag > 10 blocks is suspicious |

## See also

- [Background services](../architecture/background-services.md)
- [Configuration & environment](configuration.md)
