# Table of contents

* [cheqd: Node Documentation](README.md)

## Guides

* [Setting up a new cheqd node](docs/setup-and-configure/README.md)
  * [Interactive installer](docs/setup-and-configure/interactive/interactive-installer.md)
  * [Installing cheqd-node with Docker](docs/setup-and-configure/docker-install.md)
  * [Installing a cheqd node from binary package releases](docs/setup-and-configure/binary-install.md)
* [Setting up and configuring validator nodes](docs/validator-guide/README.md)
  * [FAQs for validator operators](docs/validator-guide/validator-faq.md)
  * [How to backup and restore node keys](docs/validator-guide/backup-and-restore-node-keys.md)
  * [How to configure pruning settings](docs/validator-guide/pruning.md)
  * [How to move a validator node](docs/validator-guide/how-to-move-node.md)
  * [How to unjail a node](docs/validator-guide/how-to-unjail.md)
* [cheqd Cosmos CLI](docs/cheqd-cli/README.md)
  * [Using cheqd Cosmos CLI to manage keys](docs/cheqd-cli/cheqd-cli-key-management.md)
  * [Using cheqd Cosmos CLI to manage accounts](docs/cheqd-cli/cheqd-cli-accounts.md)
  * [Using cheqd Cosmos CLI to manage a node](docs/cheqd-cli/cheqd-cli-node-management.md)
  * [Using cheqd Cosmos CLI for token transactions](docs/cheqd-cli/cheqd-cli-token-transactions.md)
* [Building from source](docs/build-and-networks/README.md)
  * [Docker Based Localnet](docs/build-and-networks/local-docker-network.md)
  * [Docker Compose Based Localnet](docs/build-and-networks/local-docker-compose-network.md)
  * [Setting up a new network](docs/build-and-networks/manual-network-setup.md)
* [Client-app Identity APIs](docs/identity-api/README.md)
  * [Account and key management in VDR Tools SDK](docs/identity-api/vdr-tools-sdk-accounts-keys.md)
  * [Ledger connections in VDR Tools SDK](docs/identity-api/vdr-tools-sdk-ledger-connection.md)
  * [Error messages](docs/identity-api/identity-api-error-messages.md)

## Architecture

* [Architecture Decision Record (ADR) Process](architecture/README.md)
* [List of ADRs](architecture/adr-list/README.md)
  * [ADR 001: Payment mechanism for issuing credentials](architecture/adr-list/adr-001-payment-mechanism-for-issuing-credentials.md)
  * [ADR 002: Importing/exporting mnemonic keys from Cosmos](adr-002-mnemonic-keys-cosmos.md)
  * [ADR 003: Command Line Interface (CLI) tools](architecture/adr-list/adr-003-cli-tools.md)
  * [ADR 004: Token fractions](architecture/adr-list/adr-004-token-fractions.md)
  * [ADR 005: Genesis parameters](architecture/adr-list/adr-005-genesis-parameters.md)
  * [ADR 006: Community tax](architecture/adr-list/adr-006-community-tax.md)
  * [ADR 007: Revocation registry](architecture/adr-list/adr-007-revocation-registry.md)
  * [ADR 011: AnonCreds: Schemas and Credential Definitions](adr-011-anoncreds.md)
  * [ADR {ADR-NUMBER}: {TITLE}](architecture/adr-list/adr-template.md)
* [Identity ADRs](https://docs.cheqd.io/identity/architecture/adr-list)

***

* [License](LICENSE.md)
* [Code of Conduct](CODE_OF_CONDUCT.md)
* [Security Policy](SECURITY.md)
