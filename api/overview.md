---
description: Overview of the EMEI Facilitator HTTP API — base URL, authentication, request/response conventions, error envelope.
---

# Overview & authentication

The EMEI Facilitator exposes a REST API on port 8080 by default. All endpoints accept and return JSON.

## Base URL

```
http://localhost:8080
```

For production deployments, replace with your facilitator's public host.

## Authentication

Endpoints that send transactions on the caller's behalf require an `X-Private-Key` header carrying the caller's private key (hex, with or without `0x` prefix).

| Endpoints requiring `X-Private-Key` | Endpoints that don't |
|---|---|
| `POST /emei/register` | `POST /emei/collect` |
| `POST /emei/invoice` | `GET /emei/invoice/{id}` |
| `POST /emei/present` | `GET /emei/balance/{address}` |
| `POST /emei/pay` | `GET /emei/reputation/{address}` |
| `POST /emei/mandate` | `GET /emei/statement` |
| `DELETE /emei/mandate/{id}` | `GET /emei/verify/{id}` |
| `POST /emei/withdraw` | `GET /emei/paylink/{id}` |

{% hint style="warning" %}
The Facilitator is intended to run on **infrastructure you control**. The `X-Private-Key` header sends a private key in plaintext to a service. Use TLS, restrict network access, and only ever use disposable testnet keys until mainnet is available.
{% endhint %}

## Request format

All `POST` and `DELETE` requests with bodies use `Content-Type: application/json`.

Amounts are sent as **strings of base units** (`uint256` semantics):
- 100 mUSD → `"100000000000000000000"` (18 decimals)
- 100 USDC → `"100000000"` (6 decimals)

Addresses are sent as **lowercase hex strings** with `0x` prefix.

Timestamps are **Unix seconds** as integers.

## Response format

Success responses return a 2xx status with a JSON object specific to the endpoint. Most write operations return:

```json
{
  "tx_hash": "0x...",
  "block_number": 12345678,
  ...endpoint-specific fields
}
```

## Error envelope

All errors return a non-2xx status with a structured body:

```json
{
  "error": {
    "code": "REPUTATION_TOO_LOW",
    "message": "Account 0x... score 200 below threshold 500",
    "details": {
      "account": "0x...",
      "score": 200,
      "threshold": 500
    }
  }
}
```

Decoded contract reverts (e.g. `ReputationTooLow`, `InsufficientMandateCap`) preserve their structured fields under `details`. → [Errors](errors.md)

## Status codes

| Code | Meaning |
|---|---|
| `200 OK` | Request succeeded; transaction confirmed where applicable |
| `201 Created` | Resource created (e.g., new invoice, new mandate) |
| `400 Bad Request` | Malformed request, missing field, invalid amount/address |
| `401 Unauthorized` | Missing or invalid `X-Private-Key` |
| `404 Not Found` | Invoice/mandate/address not found |
| `409 Conflict` | Decoded contract revert (e.g., `InvalidStatusTransition`) |
| `500 Internal Server Error` | Unexpected — RPC failure, indexer down, etc. |

## Rate limits

There are no built-in rate limits in the open-source Facilitator. Add a reverse proxy (nginx, Cloudflare, Caddy) if you need them.

## Endpoint groups

| Group | Endpoints | Reference |
|---|---|---|
| Identity | `/emei/register`, `/emei/reputation/{addr}` | [Identity](identity.md) |
| Invoices | `/emei/invoice`, `/emei/present`, `/emei/pay`, `/emei/collect`, `/emei/invoice/{id}` | [Invoices](invoices.md) |
| Mandates | `/emei/mandate`, `/emei/mandate/{id}` (DELETE) | [Mandates](mandates.md) |
| Settlement | `/emei/withdraw`, `/emei/balance/{addr}` | [Settlement](settlement.md) |
| Statements | `/emei/statement` | [Statements](statements.md) |
| Receipts | `/emei/verify/{id}` | [Receipts](receipts.md) |
| Pay-links | `/emei/paylink/{id}` | [Pay-links](paylinks.md) |

## See also

- [Errors](errors.md) — full error code reference.
- [Quickstart](../getting-started/quickstart.md) — call the API end-to-end in 5 minutes.
