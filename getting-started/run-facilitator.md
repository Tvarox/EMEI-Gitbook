---
description: Run the EMEI Facilitator locally — Axum HTTP server, 14 REST endpoints, 4 background services, SQLite event index.
---

# Run the Facilitator locally

The Facilitator is the Rust HTTP server in `crates/emei-facilitator`. It exposes 14 REST endpoints, runs 4 background services, and indexes events into a local SQLite database.

## Prerequisites

- **Rust 1.75+**
- Mantle Sepolia RPC URL (default `https://rpc.sepolia.mantle.xyz` works)
- A funded operator wallet (needs MNT for gas — used by the auto-collector and overdue scanner)

## Run from source

```bash
git clone https://github.com/Tvarox/EMEI-Facilitator.git
cd EMEI-Facilitator
cp crates/emei-facilitator/.env .env
cargo run -p emei-facilitator --bin emei-server
```

You should see:

```
emei-server listening on 0.0.0.0:8080
event-indexer started (continuous)
auto-collector started (10s)
overdue-scanner started (60s)
receipt-batcher started (30s)
```

## Run with Docker

From the repo root:

```bash
docker compose up --build
```

The provided `Dockerfile` is multi-stage and produces a minimal runtime image. See [Run in Docker](../guides/docker.md) for production deployment.

## Configuration

The Facilitator is configured via environment variables loaded from `.env`. The most important are:

| Variable | Description |
|---|---|
| `RPC_URL` | Mantle Sepolia JSON-RPC endpoint. |
| `OPERATOR_PRIVATE_KEY` | Key used by the auto-collector and overdue scanner background services. Fund with MNT. |
| `RECEIPT_BATCHER_PRIVATE_KEY` | Key used by the receipt batcher to call `EMEIReceipt.postMerkleRoot`. May be the same as the operator key. |
| `DATABASE_URL` | SQLite path, e.g. `sqlite://./emei.db`. |
| `BIND_ADDRESS` | HTTP bind address (default `0.0.0.0:8080`). |
| `RUST_LOG` | Log level, e.g. `emei_facilitator=debug`. |

See [Configuration & environment](../ops/configuration.md) for the full list, including the addresses of all 8 contracts.

## Health check

```bash
curl http://localhost:8080/health
```

Expected response:

```json
{"status": "ok"}
```

## Next steps

- [Connect to Mantle Sepolia](connect-mantle.md)
- [Get testnet tokens](get-testnet-tokens.md)
- [Quickstart](quickstart.md)

## See also

- [Background services](../architecture/background-services.md)
- [Configuration & environment](../ops/configuration.md)
- [Run in Docker](../guides/docker.md)
