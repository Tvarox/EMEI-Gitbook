---
description: Deploy the EMEI Facilitator to a server you control.
---

# Deployment

The Facilitator is a single Rust binary or Docker image. The recommended shape is one process behind a TLS-terminating reverse proxy.

## Recommended topology

```
[ Clients ] --HTTPS--> [ nginx / Caddy / Cloudflare ] --HTTP--> [ emei-server ]
                                                                       |
                                                                   ./data
                                                                       |
                                                                  emei.db (SQLite)
```

## Steps

1. **Provision a small VM** (1 vCPU, 1 GB RAM, 10 GB disk is sufficient for testnet).
2. **Install Docker** or Rust 1.75+.
3. **Create operator wallets**:
   - `OPERATOR_PRIVATE_KEY` — fund with MNT.
   - `RECEIPT_BATCHER_PRIVATE_KEY` — fund with MNT.
4. **Authorize the batcher key** on `EMEIReceipt` (one-time setup, requires the contract owner key).
5. **Write `.env`** per [Configuration & environment](configuration.md).
6. **Mount persistent storage** for SQLite at `DATABASE_URL`.
7. **Run** with Docker Compose or systemd.
8. **Front with TLS** (LetsEncrypt + nginx, or Cloudflare).
9. **Confirm**: `curl https://your.host/health` returns `{"status":"ok"}`.

## Hardening

- Restrict the HTTP bind address to localhost; expose only via the proxy.
- Rotate operator keys quarterly. Existing receipts remain verifiable; only the *new* batcher key needs to be authorized.
- Back up `emei.db` periodically — losing it forces a resync but does not lose receipts (those are anchored on-chain).

## See also

- [Configuration & environment](configuration.md)
- [Monitoring & logs](monitoring.md)
- [Run in Docker](../guides/docker.md)
