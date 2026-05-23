---
description: How EMEI's reputation gate works — Bay8004 reads ERC-8004 scores, applies a weighted blend, and gates both invoice creation and payment.
---

# Reputation gate (ERC-8004 + Bay8004)

EMEI gates every invoice operation on reputation. The gate has two parts:

1. **ERC-8004 registry** — the canonical identity + reputation source. On testnet this is `MockERC8004`.
2. **Bay8004** — an adapter that reads from the registry, applies a weighted scoring blend, and exposes a single `scoreOf(address) → uint256` function.

The score range is `[0, 10000]`. Higher is better.

## When the gate fires

| Moment | Who is checked | What happens on fail |
|---|---|---|
| `createInvoice` | **Both** issuer and payer | Reverts with `ReputationTooLow(account, score, threshold)`. No invoice is created. |
| `pay` | Payer (re-checked) | Reverts with `ReputationTooLow`. Invoice stays `PRESENTED` — not auto-rejected. |
| `collect` | Payer (re-checked) | Same as `pay`. |

The threshold is the global `minReputation` setting on `EMEIInvoice`, set by the contract owner via `setMinReputation(uint256)`. Default is `0` — no gate.

## The scoring formula

`Bay8004.scoreOf(address)` returns:

```
finalScore = min(
  (rawScore × (txSizeWeight + timeDecayFactor + categoryWeight)) / 10000,
  10000
)
```

`rawScore` comes from the ERC-8004 registry — the cumulative result of `giveFeedback` calls.

The weights are owner-configurable on `Bay8004`. Their sum must be `10000` (100%).

| Weight | Default | Default % | What it represents |
|---|---|---|---|
| `txSizeWeight` | 3000 | 30% | Bias toward large completed transactions |
| `timeDecayFactor` | 2000 | 20% | Bias toward recent activity |
| `categoryWeight` | 5000 | 50% | Bias toward category-specific track record |

{% hint style="info" %}
The current testnet implementation applies the weights as a uniform multiplier, not as separate per-axis components — i.e. `(txSizeWeight + timeDecayFactor + categoryWeight)` is always `10000` and the formula collapses to `min(rawScore, 10000)`. The structure is in place for future expansion to true per-axis scoring.
{% endhint %}

## Feedback after settlement

After every successful `pay` or `collect`, `EMEIInvoice` calls:

```solidity
Bay8004.giveFeedback(issuer, invoiceId, amount);
Bay8004.giveFeedback(payer,  invoiceId, amount);
```

`Bay8004` forwards to the ERC-8004 registry, where the cumulative score for both parties increments. **Both sides** earn reputation by completing transactions. There is no penalty mechanism today — only positive accrual.

## Setting the threshold

The owner of `EMEIInvoice` can raise or lower the bar:

```bash
cast send --rpc-url $RPC_URL --private-key $OWNER_KEY \
  0xC35f709255D7199394655F16008e8d1A3AD80005 \
  "setMinReputation(uint256)" 500
```

A higher threshold rejects more counterparties at creation time but produces fewer mid-flow `ReputationTooLow` reverts.

## Querying a score

{% tabs %}
{% tab title="emei CLI" %}
```bash
emei reputation 0xPAYER_ADDRESS
```
{% endtab %}

{% tab title="curl" %}
```bash
curl http://localhost:8080/emei/reputation/0xPAYER_ADDRESS
```
{% endtab %}

{% tab title="cast" %}
```bash
cast call --rpc-url https://rpc.sepolia.mantle.xyz \
  0xE61B57D84fb55E2601ab47B83c367612E348d409 \
  "scoreOf(address)(uint256)" 0xPAYER_ADDRESS
```
{% endtab %}
{% endtabs %}

## Why double-check on payment?

Reputation is dynamic. A payer who passed the gate at invoice creation may have accumulated bad signal between then and the payment moment. The second check at `pay` / `collect` ensures **value never moves to or from a counterparty whose current score is below threshold**.

If the payer's score has dropped, the issuer can either wait for it to recover or call `reject(invoiceId, "rep below threshold")` to terminate the invoice.

## Events

```solidity
event ReputationChecked(address indexed account, uint256 score, uint256 threshold);
```

Emitted on every `scoreOf` call performed by `EMEIInvoice`. Useful for auditing why an invoice operation succeeded or failed.

## Next steps

- [Security model](security.md)
- [The invoice lifecycle](invoice-lifecycle.md)

## See also

- [Bay8004 (contracts reference)](../contracts/bay8004.md)
- [HTTP API: GET /emei/reputation/{address}](../api/identity.md)
