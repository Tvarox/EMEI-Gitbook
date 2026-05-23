---
description: The emei CLI — single binary, structured JSON output, every command maps 1:1 to the Facilitator's HTTP API.
---

# CLI overview

The `emei` CLI is a thin HTTP client over the Facilitator. It outputs structured JSON suited for piping into agent runtimes.

Source: `crates/emei-cli` in [`EMEI-Facilitator`](https://github.com/Tvarox/EMEI-Facilitator).

## Configuration

Two environment variables:

| Variable | Required | Description |
|---|---|---|
| `EMEI_API_URL` | yes | Base URL of the Facilitator (e.g. `http://localhost:8080`) |
| `EMEI_PRIVATE_KEY` | for write commands | Hex private key, sent as `X-Private-Key` |

→ Setup: [Install the CLI](../getting-started/install-cli.md).

## Command map

```
emei <COMMAND> [OPTIONS]
```

| Command | Reference |
|---|---|
| `wallet create [--score N]` | [emei wallet](wallet.md) |
| `invoice create / present / pay / get / list` | [emei invoice](invoice.md) |
| `mandate create / revoke` | [emei mandate](mandate.md) |
| `collect <invoice_id> <mandate_id>` | [emei collect](collect.md) |
| `balance [address]` and `withdraw <amount>` | [emei balance & withdraw](balance-withdraw.md) |
| `reputation <address>` | [emei reputation](reputation.md) |
| `pay <id>` (shortcut for `invoice pay <id>`) | [emei invoice](invoice.md#pay) |

## Output format

All commands print a single JSON object on stdout. Errors print to stderr and exit non-zero.

```bash
emei invoice get 1 | jq '.status'
# "PAID"
```

## Conventions

- **Amounts are human-readable** in CLI flags (e.g. `--amount 100` for 100 mUSD), but the underlying API uses base units. The CLI handles conversion using the asset's decimals.
- **Addresses** must be 0x-prefixed hex.
- **Categories and counterparties** are passed comma-separated: `--counterparties 0xAAA,0xBBB`.

## See also

- [HTTP API: Overview](../api/overview.md)
- [Quickstart](../getting-started/quickstart.md)
