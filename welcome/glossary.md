---
description: Canonical definitions for every term used across EMEI documentation.
---

# Glossary

Use these names exactly. Inconsistent naming is the #1 source of integration bugs.

## Protocol terms

**Bay8004**
Reputation adapter contract. Reads scores from the ERC-8004 registry, applies weighted blending (`txSizeWeight + timeDecayFactor + categoryWeight`), and gates invoice operations. Address: `0xE61B57D84fb55E2601ab47B83c367612E348d409`.

**Category**
A short string (e.g. `"data-services"`, `"compute"`) attached to each line item and matched against a mandate's `approvedCategories` list.

**Collection mode**
How an invoice gets paid. One of `MANDATE` (auto-collect via `EMEIInvoice.collect`) or `PAY_LINK` (manual `EMEIInvoice.pay`).

**EMEIInvoice**
Central contract managing the invoice state machine. Address: `0xC35f709255D7199394655F16008e8d1A3AD80005`.

**EMEIMandate**
Contract storing payer mandates. Address: `0xF48C3bd4FE046629A9c12A39693f39c297893bD8`.

**EMEIReceipt**
Contract storing the Merkle root of each receipt batch. Address: `0x558a20766d5998765B056597b8b78fe1914f3969`.

**EMEISettlement**
Contract that moves tokens, swaps USDC→mUSD, and routes to the yield vault. Address: `0xfdCb7bA077069A7Da44711Ee6bdB49174AFA4dD0`.

**ERC-8004**
The identity + reputation registry standard EMEI uses. On testnet, the `MockERC8004` implementation is deployed at `0x4B560970423B08632bC2Aa31D0a70e29e66Fca37`.

**Facilitator**
The Rust HTTP server (`emei-facilitator` crate) that exposes 14 REST endpoints, runs 4 background services, and indexes events into SQLite. Default port 8080.

**Invoice**
An on-chain object created by `EMEIInvoice.createInvoice`. Carries `issuer`, `payer`, `amount`, `asset`, `lineItems`, `terms`, `status`, `collectionMode`, `settlementProof`, `receipt`.

**Issuer**
The party billing — calls `createInvoice` and `present`. Receives settled funds in their vault.

**Line item**
One row of an invoice. Fields: `description`, `amount`, `category`. Max 50 per invoice.

**Mandate**
A scoped pre-authorization stored on `EMEIMandate`. Fields: `spendCap`, `remainingCap`, `approvedCounterparties` (max 50), `approvedCategories` (max 20), `validFrom`, `validUntil`, `status`.

**Mandate status**
`ACTIVE` → `EXHAUSTED` (cap = 0) | `EXPIRED` (past `validUntil`) | `REVOKED` (manually cancelled).

**Milestone**
A sub-payment tied to an invoice with `MILESTONES` terms. Fields: `amount`, `dueDate`, `description`. Max 10. Sum of `amount`s must equal invoice `amount`.

**mUSD**
Yield-bearing rebasing stablecoin (18 decimals). On testnet the `MockmUSD` contract is at `0xb4C74657Ef45AA95E91BBac1db7f9C964D1cAeAD`. Balances grow over time as `rebaseIndex` increases.

**Pay-link**
A URL representing an invoice that any wallet can pay. Returns approve + pay calldata via `GET /emei/paylink/{id}`. x402-compatible.

**Payer**
The party billed — calls `pay` (or has invoices auto-collected via mandate). Their reputation is checked at invoice creation and re-checked at payment.

**Pay-link mode**
See *Collection mode*. The "manual" path: payer calls `pay` themselves with two wallet sigs (approve + pay).

**Receipt**
A 32-byte commitment to a settled invoice, batched by the Receipt Batcher. Receipts are aggregated into a Merkle tree and the root is posted on-chain via `EMEIReceipt.postMerkleRoot`.

**Reputation score**
A `uint256` in `[0, 10000]`. Returned by `Bay8004.scoreOf(address)`. Applied as a gate by `EMEIInvoice` using `minReputation` (default 0 = no gate).

**Settlement proof**
`bytes32` returned by `EMEISettlement.settle`, stored on the invoice. Cryptographic commitment to the settled transfer.

**Status (invoice)**
One of `ISSUED`, `PRESENTED`, `PAID`, `OVERDUE`, `REJECTED`. `PAID` and `REJECTED` are terminal.

**Term type**
One of `DUE_ON_RECEIPT`, `NET_N_DAYS` (1–365), `MILESTONES` (1–10).

**USDC**
Standard stablecoin (6 decimals). Auto-swapped to mUSD on payment by `EMEISettlement`. On testnet the `MockUSDC` contract is at `0x2a2f576b67773d4CFe6157C3150BbAB8fFb0FDF6`.

**Vault**
A yield-bearing destination for settled mUSD. Two options: `MUSD_REBASE` (default — uses mUSD's native rebase) and `SUSDE` (alternative). Selected per-payee via `setVaultPreference`.

## Network terms

**Mantle Sepolia**
Testnet where EMEI is currently deployed. Chain ID `5003`. RPC: `https://rpc.sepolia.mantle.xyz`. Explorer: `https://sepolia.mantlescan.xyz`.

**MNT**
Native gas token of Mantle. Get testnet MNT from the [Mantle faucet](https://faucet.sepolia.mantle.xyz/).

## Service terms

**Auto-Collector**
Background service in the Facilitator that runs every 10 seconds. Matches due invoices to active mandates and calls `EMEIInvoice.collect`.

**Event Indexer**
Continuous background service that streams contract events into SQLite, powering `GET /emei/statement`.

**Overdue Scanner**
Background service that runs every 60 seconds. Calls `markOverdue` on invoices past their due date.

**Receipt Batcher**
Background service that runs every 30 seconds. Builds a Merkle tree from paid invoices in the current batch and calls `EMEIReceipt.postMerkleRoot`.
