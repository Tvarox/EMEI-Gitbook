---
description: Pre-authorize an agent to bill you automatically by creating a scoped EMEI mandate.
---

# Set up mandate auto-collection

A mandate lets a payer pre-authorize a scoped budget for one or more counterparties. Once active, matching invoices auto-settle without further interaction from the payer.

## Prerequisites

- Both payer and target issuers registered — see [Quickstart §3](../getting-started/quickstart.md#step-3--register-the-two-identities).
- The payer holds enough mUSD to cover at least the first expected charge.

## Plan your scope

| Question | Answer |
|---|---|
| Who can bill? | List of issuer addresses (≤ 50) |
| For what? | List of categories (≤ 20) — must match line-item `category` strings |
| How much in total? | `spend_cap` in wei (e.g. `5000000000000000000000` = 5,000 mUSD) |
| For how long? | `valid_from` / `valid_until` (unix seconds) |

## Steps

### 1. Create the mandate (payer key)

```bash
export EMEI_PRIVATE_KEY=$PAYER_KEY

NOW=$(date +%s)
END=$((NOW + 2592000))   # 30 days

emei mandate create \
  --spend-cap 5000 \
  --counterparties 0xAGENT_A,0xAGENT_B \
  --categories compute,data-services \
  --valid-from $NOW \
  --valid-until $END
```

Output:

```json
{
  "mandate_id": 7,
  "tx_hash": "0x...",
  "spend_cap": "5000000000000000000000",
  "remaining_cap": "5000000000000000000000",
  "status": "ACTIVE"
}
```

### 2. Approve the Settlement contract for mUSD

The Settlement contract pulls funds from your wallet on collection. Approve it once for the cap:

```bash
cast send --rpc-url https://rpc.sepolia.mantle.xyz \
  --private-key $PAYER_KEY \
  0xb4C74657Ef45AA95E91BBac1db7f9C964D1cAeAD \
  "approve(address,uint256)" \
  0xfdCb7bA077069A7Da44711Ee6bdB49174AFA4dD0 \
  5000000000000000000000
```

### 3. (Issuer) Create + present an invoice in MANDATE mode

```bash
export EMEI_PRIVATE_KEY=$ISSUER_KEY

emei invoice create \
  --payer 0xPAYER \
  --amount 100 \
  --asset 0xb4C74657Ef45AA95E91BBac1db7f9C964D1cAeAD \
  --terms due_on_receipt \
  --mode mandate \
  --line-item "Compute hours:100:compute"

emei invoice present 1
```

### 4. Auto-Collector picks it up

The Facilitator's Auto-Collector runs every 10 seconds. It scans presented invoices, matches them against active mandates, and calls `collect`. Within ≤10s of `present`, the invoice will be `PAID`.

To trigger collection manually instead:

```bash
emei collect 1 7
```

### 5. Watch the cap decrease

```bash
curl http://localhost:8080/emei/mandate/7   # if exposed
# or
cast call --rpc-url https://rpc.sepolia.mantle.xyz \
  0xF48C3bd4FE046629A9c12A39693f39c297893bD8 \
  "getMandate(uint256)" 7
```

The `remainingCap` field will have decremented by the invoice amount.

## Pause / cancel

There is no "pause". To stop accepting new collections, revoke:

```bash
emei mandate revoke 7
```

`REVOKED` is terminal. To resume, create a new mandate.

## See also

- [Mandates](../concepts/mandates.md)
- [HTTP API: Mandates](../api/mandates.md)
- [Background services: Auto-Collector](../architecture/background-services.md)
