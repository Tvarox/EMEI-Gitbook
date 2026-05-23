---
description: Run the EMEI Facilitator with Docker Compose.
---

# Run the Facilitator in Docker

The `EMEI-Facilitator` repo ships a multi-stage `Dockerfile` and a `docker-compose` setup.

## Prerequisites

- Docker 24+
- A populated `.env` (see [Configuration](../ops/configuration.md))

## Build & run

```bash
git clone https://github.com/Tvarox/EMEI-Facilitator.git
cd EMEI-Facilitator
docker compose up --build
```

The container exposes port `8080` and persists `./data` for the SQLite event store.

## Health check

```bash
curl http://localhost:8080/health
```

## Notes

- The Dockerfile uses a slim runtime image and statically-linked binaries where possible.
- For production, mount your `.env` as a secret and put the container behind a TLS reverse proxy.
- If the container can't reach the RPC, ensure your network setup allows outbound HTTPS to `rpc.sepolia.mantle.xyz`.

## See also

- [Run the Facilitator locally](../getting-started/run-facilitator.md)
- [Configuration & environment](../ops/configuration.md)
- [Deployment](../ops/deployment.md)
