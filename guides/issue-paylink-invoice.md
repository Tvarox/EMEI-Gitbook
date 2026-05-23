---
description: Issue an EMEI invoice in pay-link mode, then have your payer settle it via the x402-compatible browser flow.
---

# Issue a pay-link invoice

This guide creates a `PAY_LINK`-mode invoice and shows how to hand it off to a payer who will pay manually with two wallet signatures.

## Prerequisites

- The Facilitator running locally — see [Run the Facilitator](../getting-started/run-facilitator.md).
- The `emei` CLI configured — see [Install the CLI](../getting-started/install-cli.md).
- Both issuer and payer registered — see [Quickstart §3](../getting-started/quickstart.md#step-3--register-the-two-identities).
- The payer holds enough mUSD or USDC to cover the invoice.

## Steps

### 1. Create the invoice

```bash
emei invoice create \
  --payer 0xPAYER \
  --amount 100 \
  --asset 0xb4C74657Ef45AA95E91BBac1db7f9C964D1cAeAD \
  --terms net_n_days --net-days 7 \
  --mode pay_link \
  --line-item "API access (1000 calls):100:data-services"
```

Note the returned `invoice_id`.

### 2. Present the invoice

```bash
emei invoice present 1
```

### 3. Generate the pay-link payload

```bash
curl http://localhost:8080/emei/paylink/1
```

The response contains pre-encoded `approve` and `pay` calldata your payer's wallet can submit directly.

### 4. Hand it off

How you deliver the pay-link is up to you:

- **Hosted page.** Build a small page that calls `/emei/paylink/{id}`, renders the amount and counterparty, and submits the two transactions via the connected wallet.
- **HTTP 402.** Return the calldata in a `WWW-Authenticate: x402` response.
- **DM/email/QR.** Encode the invoice ID + facilitator URL in a deep link.

### 5. Confirm payment

```bash
emei invoice get 1
```

When `status == "PAID"`, settlement is final on-chain.

## See also

- [Pay-links (x402)](../concepts/paylinks.md)
- [HTTP API: Pay-links](../api/paylinks.md)
- [Set up mandate auto-collection](setup-mandate.md) — alternative for recurring relationships
