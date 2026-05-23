---
description: All environment variables read by the Facilitator at startup.
---

# Configuration & environment

The Facilitator is configured entirely via environment variables, loaded from `.env` on startup.

## Network

| Variable | Required | Default | Description |
|---|---|---|---|
| `RPC_URL` | ✅ | — | Mantle Sepolia JSON-RPC endpoint |
| `CHAIN_ID` | — | `5003` | Chain ID; safety check vs. RPC |
| `BIND_ADDRESS` | — | `0.0.0.0:8080` | HTTP bind address |

## Operator keys

| Variable | Required | Description |
|---|---|---|
| `OPERATOR_PRIVATE_KEY` | ✅ | Used by Auto-Collector and Overdue Scanner |
| `RECEIPT_BATCHER_PRIVATE_KEY` | ✅ | Used by Receipt Batcher (may equal `OPERATOR_PRIVATE_KEY`) |

Both keys need MNT for gas only — they never custody user funds.

## Contract addresses (Mantle Sepolia)

| Variable | Value |
|---|---|
| `EMEI_INVOICE_ADDRESS` | `0xC35f709255D7199394655F16008e8d1A3AD80005` |
| `EMEI_MANDATE_ADDRESS` | `0xF48C3bd4FE046629A9c12A39693f39c297893bD8` |
| `EMEI_SETTLEMENT_ADDRESS` | `0xfdCb7bA077069A7Da44711Ee6bdB49174AFA4dD0` |
| `BAY_8004_ADDRESS` | `0xE61B57D84fb55E2601ab47B83c367612E348d409` |
| `EMEI_RECEIPT_ADDRESS` | `0x558a20766d5998765B056597b8b78fe1914f3969` |
| `MOCK_ERC8004_ADDRESS` | `0x4B560970423B08632bC2Aa31D0a70e29e66Fca37` |
| `MOCK_MUSD_ADDRESS` | `0xb4C74657Ef45AA95E91BBac1db7f9C964D1cAeAD` |
| `MOCK_USDC_ADDRESS` | `0x2a2f576b67773d4CFe6157C3150BbAB8fFb0FDF6` |

## Storage

| Variable | Default | Description |
|---|---|---|
| `DATABASE_URL` | `sqlite://./emei.db` | SQLite path. Use absolute paths in production. |
| `INDEXER_START_BLOCK` | (genesis or last checkpoint) | Override starting block for fresh sync |

## Background services

| Variable | Default | Description |
|---|---|---|
| `AUTO_COLLECTOR_INTERVAL_SEC` | `10` | Auto-Collector tick |
| `OVERDUE_SCANNER_INTERVAL_SEC` | `60` | Overdue Scanner tick |
| `RECEIPT_BATCHER_INTERVAL_SEC` | `30` | Receipt Batcher tick |

## Logging

| Variable | Example | Description |
|---|---|---|
| `RUST_LOG` | `emei_facilitator=info` | `tracing` filter spec |

## Example `.env`

```bash
RPC_URL=https://rpc.sepolia.mantle.xyz
CHAIN_ID=5003
BIND_ADDRESS=0.0.0.0:8080

OPERATOR_PRIVATE_KEY=0xYOUR_OPERATOR_KEY
RECEIPT_BATCHER_PRIVATE_KEY=0xYOUR_BATCHER_KEY

EMEI_INVOICE_ADDRESS=0xC35f709255D7199394655F16008e8d1A3AD80005
EMEI_MANDATE_ADDRESS=0xF48C3bd4FE046629A9c12A39693f39c297893bD8
EMEI_SETTLEMENT_ADDRESS=0xfdCb7bA077069A7Da44711Ee6bdB49174AFA4dD0
BAY_8004_ADDRESS=0xE61B57D84fb55E2601ab47B83c367612E348d409
EMEI_RECEIPT_ADDRESS=0x558a20766d5998765B056597b8b78fe1914f3969
MOCK_ERC8004_ADDRESS=0x4B560970423B08632bC2Aa31D0a70e29e66Fca37
MOCK_MUSD_ADDRESS=0xb4C74657Ef45AA95E91BBac1db7f9C964D1cAeAD
MOCK_USDC_ADDRESS=0x2a2f576b67773d4CFe6157C3150BbAB8fFb0FDF6

DATABASE_URL=sqlite:///var/lib/emei/emei.db

RUST_LOG=emei_facilitator=info
```

## See also

- [Run the Facilitator locally](../getting-started/run-facilitator.md)
- [Deployment](deployment.md)
- [Deployed addresses](../contracts/addresses.md)
