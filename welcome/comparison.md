---
description: How EMEI relates to x402, ERC-7710, traditional SaaS billing, and direct on-chain transfers.
---

# How EMEI compares

EMEI sits next to several adjacent primitives. The differences below are based on the current testnet deployment.

## EMEI vs. x402

[x402](https://www.x402.org/) is a payment protocol that returns HTTP 402 with payment requirements; the client pays and retries. EMEI's pay-link mode is **x402-compatible**: the `GET /emei/paylink/{id}` endpoint returns the calldata a browser or client can use to satisfy a 402 challenge.

| Capability | x402 | EMEI pay-link | EMEI mandate |
|---|---|---|---|
| Per-request payment | ✅ | ✅ | ✅ (auto-collect) |
| Pre-authorization (no human in loop) | ❌ | ❌ | ✅ |
| On-chain invoice object | ❌ | ✅ | ✅ |
| Reputation gating | ❌ | ✅ | ✅ |
| Receipts (anchored) | ❌ | ✅ | ✅ |
| Yield on settled funds | ❌ | ✅ | ✅ |

**Use x402 when:** you want a pure HTTP 402 flow with no on-chain state.
**Use EMEI pay-link when:** you want x402's UX *and* an on-chain invoice with terms, reputation, and receipts.
**Use EMEI mandate when:** you want recurring or programmatic spend without a wallet popup per call.

## EMEI vs. ERC-7710 / session keys

ERC-7710 (and similar session-key designs) lets a smart account delegate scoped permissions. EMEI's mandate is **complementary**, not competing:

- ERC-7710 delegates *signing authority*. EMEI delegates *invoice acceptance*.
- ERC-7710 lives on the payer's account. EMEI mandates live on a shared `EMEIMandate` contract.
- A payer using ERC-7710 *and* EMEI gets two layers of scoping: their session key signs the mandate, then the mandate gates per-invoice collection.

## EMEI vs. raw `transferFrom`

You can already build "billing" by calling `approve` + `transferFrom`. What you don't get:

- A structured invoice object (line items, terms, milestones).
- Reputation gating.
- Auto-collection on a due date.
- Receipts, statements, or cross-asset normalization.
- Yield routing on settled funds.

EMEI is the productized version of that pattern.

## EMEI vs. SaaS billing (Stripe, Paddle)

| | Stripe / Paddle | EMEI |
|---|---|---|
| Counterparty | Human + card | Agent or human + wallet |
| Settlement time | T+1 to T+7 days | Same block |
| Chargebacks | Yes | No (terminal `PAID`) |
| Programmable terms | Limited | First-class |
| Reputation | Implicit (KYC) | Explicit (ERC-8004) |
| Yield while idle | No | Yes (vault rebase) |
| Network | Card rails | Mantle (any-asset via swap) |

Use Stripe for human consumer flows. Use EMEI for agent-to-agent flows and machine-to-human invoicing where settlement finality and programmability matter.

## See also

- [Pay-links (x402)](../concepts/paylinks.md)
- [Mandates](../concepts/mandates.md)
