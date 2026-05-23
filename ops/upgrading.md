---
description: How to upgrade the EMEI Facilitator and what to expect when contract addresses change.
---

# Upgrading & versioning

## Facilitator upgrades

The Facilitator is a regular Rust binary. To upgrade:

1. Pull the new release.
2. Stop the running container/service.
3. Start with the same `.env`.
4. Verify `/health` and check the latest indexed block against the RPC head.

The local SQLite is forward-compatible across patch versions. Major versions may include schema migrations — see the release notes.

## Contract address changes

EMEI's contracts are not currently upgradeable. Any new deployment is a **new address set**. When that happens:

1. The new addresses will be published in [Deployed addresses](../contracts/addresses.md).
2. Update `EMEI_*_ADDRESS` env vars.
3. Wipe `emei.db` (the new contracts have a fresh event history).
4. Restart.

In-flight invoices on the old addresses remain queryable directly via Mantlescan but will not surface in your statements after the migration.

## Mainnet (future)

Mainnet deployment is planned. When it ships:

- Distinct `CHAIN_ID` and `RPC_URL`.
- Real ERC-8004 registry replacing `MockERC8004`.
- Real yield-bearing stablecoin replacing `MockmUSD`.

The HTTP/CLI surface will remain stable across the testnet → mainnet transition.

## See also

- [Configuration & environment](configuration.md)
- [Changelog](../resources/changelog.md)
