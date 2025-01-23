# Network-wide Software Upgrades

## Context

Updates to the ledger code running on cheqd mainnet/testnet is voted in via governance proposals on-chain for "breaking" software changes.

We use [semantic versioning](https://semver.org/) to define our software release versions. The latest software version running on chain is in the **v2.x.x** family. Any new release versions that bump only the minor/fix version digits (the second and the third part of release version numbers) is intended to be **compatible** within the family and **does not require** an on-chain upgrade proposal to be made.

## Creating a new software upgrade proposal

Network-wide software upgrades are typically initiated by the core development team behind the cheqd project. The process followed for the network upgrade is defined in our [guide on creating a Software Upgrade proposal](propose-software-upgrade.md) via network governance.

Additionally, these software upgrades are discussed off-chain on [our discussion forum](https://commonwealth.im/cheqd/discussions) and on [our Discord server](http://cheqd.link/discord-github).

## Upgrade Process

You can find more details on the actual upgrade process under in our [Upgrade Guide](upgrade.md)

## Previous software upgrade guides

[This section](upgrade-guides) lists **previous** software upgrade proposals and specific instructions on how to execute them.

- [v0.6.0 Upgrade Guide](upgrade-guides/v0.6-upgrade.md)
- [v2.0.2 Upgrade Guide](upgrade-guides/v2.x-upgrade.md)
- [v3.0.1 Upgrade Guide](upgrade-guides/v3.x-upgrade.md)
- [v3.1.4 Upgrade Guide](upgrade-guides/v3.1.x-upgrade.md)
