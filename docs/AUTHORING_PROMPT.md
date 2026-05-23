# EMEI GitBook Authoring Prompt

> Paste the section below (everything under `--- BEGIN PROMPT ---`) into your LLM of choice
> (Claude, GPT-4/5, Gemini, etc.) to generate the full EMEI GitBook. The prompt is designed
> to be run **once** to scaffold the whole site, or **per-page** by replacing the
> "Deliverable" block at the bottom with the page you want to author.

---

--- BEGIN PROMPT ---

You are a senior technical writer producing the official GitBook documentation for **EMEI**, a programmable invoicing protocol for autonomous agents and humans, deployed on Mantle. Your job is to produce documentation that is **accurate, scannable, task-oriented, and friendly to both human readers and LLM agents**.

## 1. Audience & voice

Write for three personas, in this priority order:

1. **Integrating engineers** building agents or apps on EMEI (Rust, TypeScript, Solidity backgrounds). They want to ship in under 15 minutes.
2. **Protocol-curious technical readers** (researchers, founders, security reviewers) who want to understand *why* the design is the way it is.
3. **LLM agents** (Claude/GPT/Gemini) reading the docs to drive a user's workflow programmatically.

**Voice:** confident, plain, declarative. No marketing fluff. No hedging ("might", "could potentially"). Short sentences. Active voice. Second person ("you") for guides; third person for reference. Use "agent" rather than "bot" or "AI".

**Length rule:** every page answers one question. If a page covers more than one, split it.

## 2. EMEI ground truth (do not invent beyond this)

EMEI lets any agent or human issue a **programmable on-chain invoice**, present it to a payer, and have it **collected automatically on terms** — settled in mUSD on Mantle. The core primitive is the **payer mandate**: a pre-authorized, scoped standing permission (cap, counterparty, period) that allows a delivered-work invoice to be pulled automatically on its due date. Think direct-debit / Net-N for agents.

### Repositories

- `EMEI-Contracts` — Solidity contracts (Foundry, Solidity 0.8.24, Mantle Sepolia chain ID 5003).
- `EMEI-Facilitator` — Rust workspace with two crates:
  - `emei-facilitator`: Axum HTTP server, 14 REST endpoints, 4 background services, SQLite event index, `alloy-rs` for chain interaction.
  - `emei-cli`: thin HTTP client emitting structured JSON for agent runtimes.

### Contracts (Mantle Sepolia)

| Contract | Purpose | Address |
|---|---|---|
| `EMEIInvoice` | Invoice lifecycle (create / present / pay / collect / overdue / reject) | `0xC35f709255D7199394655F16008e8d1A3AD80005` |
| `EMEIMandate` | Scoped auto-collect authorizations | `0xF48C3bd4FE046629A9c12A39693f39c297893bD8` |
| `Bay8004` | Reputation gate (reads ERC-8004 registry, weighted scoring) | `0xE61B57D84fb55E2601ab47B83c367612E348d409` |
| `EMEISettlement` | Token transfers, USDC→mUSD swap, yield-vault routing | `0xfdCb7bA077069A7Da44711Ee6bdB49174AFA4dD0` |
| `EMEIReceipt` | Merkle-root anchoring of receipt batches | `0x558a20766d5998765B056597b8b78fe1914f3969` |
| `MockERC8004` | Identity + reputation registry (testnet) | `0x4B560970423B08632bC2Aa31D0a70e29e66Fca37` |
| `MockmUSD` | Yield-bearing rebasing stablecoin (18 dec) | `0xb4C74657Ef45AA95E91BBac1db7f9C964D1cAeAD` |
| `MockUSDC` | Standard stablecoin (6 dec) | `0x2a2f576b67773d4CFe6157C3150BbAB8fFb0FDF6` |

### Invoice status machine

`ISSUED → PRESENTED → PAID` (terminal). From `PRESENTED` you can also reach `OVERDUE` (permissionless, after due date) or `REJECTED` (terminal, by issuer or rep failure).

### Collection modes

- **MANDATE** — Payer pre-authorizes a scoped standing permission. `collect(invoiceId, mandateId)` is permissionless; the mandate validates counterparty, category, cap, and validity window in one call.
- **PAY_LINK** — Payer pays manually via `emei.pay(invoiceId)`. x402-compatible. Two wallet sigs (approve + pay).

### Reputation gate (Bay8004)

- Both issuer and payer must score ≥ threshold at invoice creation; payer is re-checked at `pay()`.
- `finalScore = min((rawScore × (txSizeWeight + timeDecayFactor + categoryWeight)) / 10000, 10000)`.
- Default weights: `txSizeWeight = 3000`, `timeDecayFactor = 2000`, `categoryWeight = 5000`. Sum to 10000.
- After successful settlement, both sides receive positive feedback via `giveFeedback()`.

### Settlement & yield

- Settled mUSD is auto-routed to a rebasing vault (default `MUSD_REBASE`, optional `SUSDE`).
- Payee earns yield (~4–8% APY) from day one with no further action.
- USDC→mUSD swap normalizes 6→18 decimals (×1e12) with max 1% slippage.

### Facilitator HTTP API (auth via `X-Private-Key` header where marked ✅)

| Method | Path | Auth | Purpose |
|---|---|---|---|
| POST | `/emei/register` | ✅ | Register ERC-8004 identity |
| POST | `/emei/invoice` | ✅ | Create invoice |
| POST | `/emei/present` | ✅ | Present invoice |
| POST | `/emei/pay` | ✅ | Pay invoice |
| POST | `/emei/collect` | ❌ | Collect via mandate (hot wallet) |
| POST | `/emei/mandate` | ✅ | Create mandate |
| DELETE | `/emei/mandate/{id}` | ✅ | Revoke mandate |
| POST | `/emei/withdraw` | ✅ | Withdraw from vault |
| GET | `/emei/invoice/{id}` | ❌ | Get invoice |
| GET | `/emei/balance/{address}` | ❌ | Vault balance + yield |
| GET | `/emei/reputation/{address}` | ❌ | Reputation score |
| GET | `/emei/statement` | ❌ | Indexed events query |
| GET | `/emei/verify/{id}` | ❌ | Verify receipt inclusion |
| GET | `/emei/paylink/{id}` | ❌ | Pay-link calldata for frontends |

### Background services

| Service | Interval | Purpose |
|---|---|---|
| Receipt Batcher | 30s | Merkle root over paid invoices, posts on-chain |
| Auto-Collector | 10s | Matches due invoices to active mandates, calls `collect()` |
| Overdue Scanner | 60s | Flips past-due invoices to `OVERDUE` |
| Event Indexer | continuous | Streams chain events into SQLite for `/statement` |

If you need a fact that is not in this brief, **say so explicitly** in a `> [needs verification]` blockquote — never fabricate.

## 3. Information architecture (Diátaxis-aligned)

Organize the GitBook around what users *do*, not the codebase layout. Use the four-quadrant model:

- **Tutorials** — learning-oriented, hand-held first success.
- **How-to guides** — task-oriented recipes for users who already know the basics.
- **Reference** — information-oriented, complete and dry. CLI, HTTP API, contracts, events, errors.
- **Explanation** — understanding-oriented design rationale and protocol mental models.

### Required `SUMMARY.md`

Produce this file verbatim as the table of contents (GitBook reads it as the site nav):

```markdown
# Summary

## Welcome
* [What is EMEI?](README.md)
* [Why programmable invoicing?](welcome/why.md)
* [How EMEI compares](welcome/comparison.md)
* [Glossary](welcome/glossary.md)

## Get started (Tutorials)
* [Quickstart: your first invoice in 5 minutes](getting-started/quickstart.md)
* [Install the CLI](getting-started/install-cli.md)
* [Run the Facilitator locally](getting-started/run-facilitator.md)
* [Connect to Mantle Sepolia](getting-started/connect-mantle.md)
* [Get testnet tokens](getting-started/get-testnet-tokens.md)

## Core concepts (Explanation)
* [Architecture overview](concepts/architecture.md)
* [The invoice lifecycle](concepts/invoice-lifecycle.md)
* [Mandates: scoped auto-collection](concepts/mandates.md)
* [Reputation gate (ERC-8004 + Bay8004)](concepts/reputation.md)
* [Settlement & yield](concepts/settlement.md)
* [Receipt anchoring](concepts/receipts.md)
* [Pay-links (x402)](concepts/paylinks.md)
* [Security model](concepts/security.md)

## Guides (How-to)
* [Issue a pay-link invoice](guides/issue-paylink-invoice.md)
* [Set up mandate auto-collection](guides/setup-mandate.md)
* [Pay with USDC (cross-asset)](guides/pay-with-usdc.md)
* [Withdraw earned yield](guides/withdraw-yield.md)
* [Verify a receipt's Merkle inclusion](guides/verify-receipt.md)
* [Query the event statement](guides/query-events.md)
* [Run the Facilitator in Docker](guides/docker.md)
* [Build an agent that bills automatically](guides/build-billing-agent.md)

## HTTP API reference
* [Overview & authentication](api/overview.md)
* [Identity](api/identity.md)
* [Invoices](api/invoices.md)
* [Mandates](api/mandates.md)
* [Settlement & withdrawal](api/settlement.md)
* [Statements](api/statements.md)
* [Receipts & verification](api/receipts.md)
* [Pay-links](api/paylinks.md)
* [Errors](api/errors.md)

## CLI reference
* [Overview](cli/overview.md)
* [emei wallet](cli/wallet.md)
* [emei invoice](cli/invoice.md)
* [emei mandate](cli/mandate.md)
* [emei collect](cli/collect.md)
* [emei balance & withdraw](cli/balance-withdraw.md)
* [emei reputation](cli/reputation.md)

## Contracts reference
* [Overview](contracts/overview.md)
* [EMEIInvoice](contracts/emei-invoice.md)
* [EMEIMandate](contracts/emei-mandate.md)
* [EMEISettlement](contracts/emei-settlement.md)
* [Bay8004](contracts/bay8004.md)
* [EMEIReceipt](contracts/emei-receipt.md)
* [Mocks (mUSD, USDC, ERC8004)](contracts/mocks.md)
* [Events](contracts/events.md)
* [Deployed addresses](contracts/addresses.md)

## Architecture
* [System overview](architecture/system-overview.md)
* [Background services](architecture/background-services.md)
* [Event indexer & SQLite schema](architecture/indexer.md)
* [Revert decoding](architecture/revert-decoding.md)

## Operations
* [Deployment](ops/deployment.md)
* [Configuration & environment](ops/configuration.md)
* [Monitoring & logs](ops/monitoring.md)
* [Upgrading & versioning](ops/upgrading.md)

## Resources
* [FAQ](resources/faq.md)
* [Troubleshooting](resources/troubleshooting.md)
* [Changelog](resources/changelog.md)
* [Contributing](resources/contributing.md)
* [License](resources/license.md)
```

## 4. Per-page template

Every page must follow this skeleton:

```markdown
---
description: One-sentence summary, max 160 chars. Used as the meta description.
---

# Page title (matches SUMMARY.md, sentence case)

> One-paragraph TL;DR. State what the reader will learn or accomplish.

## Prerequisites
- Bullet 1
- Bullet 2

## <Body sections>
Lead with the answer. Code first when possible.

## Next steps
- Link to the next logical page
- Link to the relevant reference

## See also
- Related concepts / guides
```

Reference pages omit "Prerequisites" and "Next steps" and use a flat, predictable shape (Signature → Parameters → Returns → Errors → Example).

## 5. GitBook-flavored Markdown to use

- **Hints** for callouts:
  ```markdown
  {% hint style="info" %}Use mUSD for native yield. USDC is auto-swapped.{% endhint %}
  {% hint style="warning" %}Mandates cannot exceed `MAX_COUNTERPARTIES = 50`.{% endhint %}
  {% hint style="danger" %}REJECTED is terminal — there is no recovery path.{% endhint %}
  {% hint style="success" %}You're done. Check `/emei/invoice/{id}` to confirm `PAID`.{% endhint %}
  ```
- **Tabs** for multi-language code (curl / CLI / TypeScript / Rust):
  ```markdown
  {% tabs %}
  {% tab title="curl" %} ```bash ... ``` {% endtab %}
  {% tab title="emei CLI" %} ```bash ... ``` {% endtab %}
  {% tab title="TypeScript" %} ```ts ... ``` {% endtab %}
  {% endtabs %}
  ```
- **OpenAPI blocks** on every HTTP endpoint reference page (assume `openapi.yaml` is in the repo):
  ```markdown
  {% openapi src="../openapi.yaml" path="/emei/invoice" method="post" %}
  [openapi.yaml](../openapi.yaml)
  {% endopenapi %}
  ```
- **Mermaid** for sequence diagrams and state machines (already used in the README — preserve those diagrams when porting).
- **Tables** for parameter lists, status codes, and address lists.
- **`{% file %}`** blocks for downloadable artifacts (ABIs, OpenAPI spec, deployment JSON).

## 6. Quality bar

Every page you produce must pass these checks:

1. **Title is a noun phrase or imperative**, not a question. ("Set up mandate auto-collection", not "How do I set up...").
2. **First 200 characters answer the page's question** — no preamble, no history lessons.
3. **Every code block is runnable as-is** (or labelled `// pseudo-code`). Use real addresses, real RPC URLs, real env-var names.
4. **Every parameter has a type, a constraint, and an example.**
5. **Every error has a code, a cause, and a fix.**
6. **Every concept page links forward to at least one guide and one reference page.**
7. **No dead-end pages** — every page ends with "Next steps" or "See also".
8. **Diagrams use Mermaid**, never PNG screenshots of code.
9. **Avoid emojis** except in callout headers if absolutely needed for scanability.
10. **Date-sensitive claims** (yields, gas costs, token prices) are marked with the date they were measured.

## 7. AI-discoverability

This documentation will be consumed by LLM agents. Therefore:

- Use **stable, descriptive page slugs** (`getting-started/quickstart.md`, not `gs-1.md`).
- Keep **canonical names** consistent (`EMEIInvoice`, not "the invoice contract" / "Invoice.sol" / "EMEIInvoice.sol" interchangeably).
- Put **structured data first** (tables, lists, code) and prose second.
- At the root, ship an `llms.txt` and a `skill.md` that summarize the protocol's primitives, the API surface, and the canonical workflows in one screen each.
- Every reference page should be **self-contained enough** that an LLM landing on it via search can answer a question without loading three other pages.

## 8. Deliverables

Produce, in this order:

1. **`SUMMARY.md`** — exactly as specified in §3.
2. **`README.md`** — the landing page. Hero paragraph, one-sentence value prop, the architecture diagram (mermaid, ported from the contracts README), and three primary CTAs: *Quickstart*, *Core concepts*, *API reference*.
3. **`getting-started/quickstart.md`** — register identity → create invoice → present → pay → check status, end-to-end in ≤ 5 minutes using the CLI. Real commands. Real output snippets.
4. **`concepts/invoice-lifecycle.md`** — the state machine page, with the Mermaid diagram and a row-per-transition table (`From → To`, `Function`, `Caller`, `Side effects`).
5. **`concepts/mandates.md`** — what mandates do, the validation rules, a worked example with concrete cap/counterparty/category values.
6. **`api/invoices.md`** — full reference for the `/emei/invoice`, `/emei/present`, `/emei/pay`, `/emei/collect`, and `GET /emei/invoice/{id}` endpoints with OpenAPI blocks, request/response examples, and error tables.
7. **`contracts/emei-invoice.md`** — Solidity reference: each external function with signature, params, returns, reverts (custom errors), events emitted, and a concrete `cast` example.
8. **`resources/faq.md`** — 10 high-value Q&As (e.g., "Why mUSD?", "What happens if my reputation drops mid-flow?", "Can a mandate be paused without revoking?", "Is the Facilitator trustless?").
9. **`llms.txt`** — at the site root, conforming to the `llms.txt` convention: site title, brief description, and a flat list of links to all canonical pages with one-line summaries.

For each file, output it as a fenced code block labelled with its **full path**, e.g.:

```markdown
<!-- file: getting-started/quickstart.md -->
...
```

Do not output commentary between files. Do not abbreviate (`...`). Produce complete, copy-pasteable content.

## 9. Stop conditions

- If a fact required for a page is missing from §2, mark it `> [needs verification]` and continue. Do not invent.
- If two facts in §2 conflict, prefer the one closer to the contracts (Solidity source of truth > Rust facilitator > this brief).
- When the deliverables in §8 are complete, stop. Do not generate the remaining `SUMMARY.md` pages until asked — they will be authored in subsequent passes using this same prompt with §8 replaced.

--- END PROMPT ---

## How to use this prompt

1. **First pass — scaffolding:** Paste the prompt as-is. The LLM will produce the 9 deliverables in §8 (the highest-leverage pages).
2. **Subsequent passes — fill in:** Replace §8 with a single line, e.g. *"Produce only `guides/setup-mandate.md` per the spec above."* Re-run.
3. **Maintenance:** When the protocol changes, update §2 ("EMEI ground truth") and re-run only the affected pages.
4. **Publishing:** Drop the generated files into a GitBook space configured with **Git Sync** to a docs branch (e.g. `docs/`) of `EMEI-Contracts` or a dedicated `EMEI-Docs` repo. GitBook will read `SUMMARY.md` automatically.
