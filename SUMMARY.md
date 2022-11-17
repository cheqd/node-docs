# Table of contents

* [cheqd: Node Documentation](README.md)

## Guides <a href="#docs" id="docs"></a>

* [Setting up a new cheqd node](docs/setup-and-configure/README.md)
  *
  * [Interactive installer](docs/setup-and-configure/interactive/interactive-installer.md)
  * [Installing cheqd-node with Docker](docs/setup-and-configure/docker-install.md)
* [Setting up and configuring validators](docs/validator-guide/README.md)
  * [FAQs for validator operators](docs/validator-guide/validator-faq.md)
  * [Backup and restore node keys with Hashicorp Vault](docs/validator-guide/backup-and-restore-node-keys.md)
* [cheqd Cosmos CLI](docs/cheqd-cli/README.md)
  *
  * [Using cheqd Cosmos CLI to manage keys](docs/cheqd-cli/cheqd-cli-key-management.md)
  * [Using cheqd Cosmos CLI to manage accounts](docs/cheqd-cli/cheqd-cli-accounts.md)
  * [Using cheqd Cosmos CLI to manage a node](docs/cheqd-cli/cheqd-cli-node-management.md)
  * [Using cheqd Cosmos CLI for token transactions](docs/cheqd-cli/cheqd-cli-token-transactions.md)
* [Building from source](docs/build-and-networks/README.md)
  *
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
  * [ADR 002: cheqd DID method](architecture/adr-list/adr-002-cheqd-did-method.md)
  * [ADR 003: Command Line Interface (CLI) tools](architecture/adr-list/adr-003-cli-tools.md)
  * [ADR 004: Token fractions](architecture/adr-list/adr-004-token-fractions.md)
  * [ADR 005: Genesis parameters](architecture/adr-list/adr-005-genesis-parameters.md)
  * [ADR 006: Community tax](architecture/adr-list/adr-006-community-tax.md)
  * [ADR 007: Revocation registry](architecture/adr-list/adr-007-revocation-registry.md)
  * [ADR 008: cheqd on-ledger resources](architecture/adr-list/adr-008-ledger-resources.md)
  * [ADR 009: Importing/exporting mnemonic keys from Cosmos](architecture/adr-list/adr-009-mnemonic-keys-cosmos.md)
  * [ADR {ADR-NUMBER}: {TITLE}](architecture/adr-list/adr-template.md)

***

* [License](LICENSE.md)
* [Code of Conduct](CODE\_OF\_CONDUCT.md)
* [Security Policy](SECURITY.md)
