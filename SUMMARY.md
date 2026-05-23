# Summary

## Welcome

* [What is EMEI?](README.md)
* [Why programmable invoicing?](welcome/why.md)
* [How EMEI compares](welcome/comparison.md)
* [Glossary](welcome/glossary.md)

## Get started

* [Quickstart: your first invoice in 5 minutes](getting-started/quickstart.md)
* [Install the CLI](getting-started/install-cli.md)
* [Run the Facilitator locally](getting-started/run-facilitator.md)
* [Connect to Mantle Sepolia](getting-started/connect-mantle.md)
* [Get testnet tokens](getting-started/get-testnet-tokens.md)

## Core concepts

* [Architecture overview](concepts/architecture.md)
* [The invoice lifecycle](concepts/invoice-lifecycle.md)
* [Mandates: scoped auto-collection](concepts/mandates.md)
* [Reputation gate (ERC-8004 + Bay8004)](concepts/reputation.md)
* [Settlement & yield](concepts/settlement.md)
* [Receipt anchoring](concepts/receipts.md)
* [Pay-links (x402)](concepts/paylinks.md)
* [Security model](concepts/security.md)

## Guides

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
