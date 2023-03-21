# Backup/Restore validator keys

If you're running a validator node, it's important to backup your validator's keys and state - especially before attempting any updates or shifting nodes.

## What to backup from a validator node

Each validator node has **three files/secrets** that must be backed up, in case you want to restore or move a node. Anything *not* in scope listed below can be easily restored from snapshot or otherwise replaced with fresh copies, and as such this list is the **bare minimum** that needs to be backed up.

> `$CHEQD_HOME` is the data directory for your node, which defaults to `/home/cheqd/.cheqdnode`

### Validator private key

The **validator private key** is one of the most important secrets that uniquely identifies your validator, and what this node uses to sign blocks, participate in consensus etc. This file is stored under `$CHEQD_HOME/config/priv_validator_key.json`.

### Node key

In the same folder as your validator private key, there's another key called `$CHEQD_HOME/config/node_key.json`. This key is used to derive the **node ID** for your validator.

Backing up this key means if you move or restore your node, you don't have to change the node ID in the configuration files any peers have. This is only relevant (usually) if you're running multiple nodes, e.g., a sentry or seed node.

For most node operators who run a singular validator node, this node key is **NOT** important and can be refreshed/created as new. It is only used for Tendermint peer-to-peer communication. Hypothetically, if you created a new node key (say when moving a node from one machine to another), and then restored the `priv_validator_key.json`, this is absolutely fine.

### Validator private state

The **validator private state** is stored in the **`data`** folder, not the `config` folder where most other configuration files are kept - and therefore *often* gets missed by validator operators during backup. This file is stored at `$CHEQD_HOME/data/priv_validator_state.json`.

This file stores the **last block height** signed by the validator and is updated every time a new block is created. Therefore, this should only be backed up **after** stopping the node service, otherwise, the data stored within this file will be in an inconsistent state. An example validator state file is shown below:

```json
{
  "height": "3968024",
  "round": 0,
  "step": 3,
  "signature": "<signature>",
  "signbytes": "<sign-bytes>"
}
```

If you forget to restore to the validator state file when restoring a node, or when restoring a node from snapshot, your validator will **double-sign** blocks it has already signed, and **get jailed permanently ("tombstoned")** with no way to re-establish the validator.

### Upgrade height information

The software upgrades and block height they were applied at is stored in `$CHEQD_HOME/data/upgrade-info.json`. This file is used by Cosmovisor to track automated updates, and informs it whether it should attempt an upgrade/migration or not.

```json
{"name":"v0.6","height":2478827,"info":"{\"binaries\":{\"linux/amd64\":\"https://github.com/cheqd/cheqd-node/releases/download/v0.6.0/cheqd-noded\"}}"}
```

## Manual backup and restore

The simplest way to backup your validator secrets listed above is to display them in your terminal:

```bash
cat <filename>
```

You can copy the contents of the file displayed in terminal off the server and store it in a secure location.

To restore the files, open the equivalent file on the machine where you want to restore the files to using a text editor (e.g., `nano`) and paste in the contents:

```bash
nano <filename>
```

## Backing up and restoring via HashiCorp Vault

[HashiCorp Vault](https://www.hashicorp.com/products/vault) is an open-source project that allows server admins to run a secure, access-controlled off-site backup for secrets. You can either [run a self-managed HashiCorp Vault server](https://developer.hashicorp.com/vault/tutorials/getting-started/getting-started-install) or [use a hosted/managed HashiCorp Vault service](https://www.hashicorp.com/products/vault/pricing).

### Pre-requisites

Before you get started with this guide, make sure you've [installed HashiCorp Vault CLI](https://developer.hashicorp.com/vault/docs/install) on the validator you want to run backups from.

You also need a running HashiCorp Vault server cluster you can use to proceed with this guide.

> Setting up a HashiCorp Vault cluster is outside the scope of this documentation, since it can vary a *lot* depending on your setup. If you don't already have this set up, [HashiCorp Vault documentation](https://developer.hashicorp.com/vault) and [Vault tutorials](https://developer.hashicorp.com/vault/tutorials) is the best place to get started.

#### Setting up Vault CLI environment

Once you have Vault CLI set up on the validator, you need to set up environment variables in your terminal to configure which Vault server the secrets need to be backed up to.

Add the following variables to your terminal environment. Depending on which terminal you use (e.g., bash, shell, zsh, fish etc), you may need to modify the statements accordingly. You'll also need to modify the values according to your validator and Vault server configuration.

```bash
export VAULT_ADDR="https://vault.example.com"
export VAULT_NAMESPACE="admin/testnet"
export VAULT_TOKEN="[VAULT_TOKEN]"
export CHEQD_HOME=/home/cheqd
export MONIKER=[NODE_MONIKER]
```

### Backup procedure for HashiCorp Vault

#### Configure Vault backup script

Download the [`vault-backup.sh` script](https://raw.githubusercontent.com/cheqd/infra/main/scripts/tools/vault-backup.sh) from Github:

```bash
wget -c https://raw.githubusercontent.com/cheqd/infra/main/scripts/tools/vault-backup.sh
```

Make the script executable:

```bash
chmod +x vault-backup.sh
```

We recommend that you open the script using an editor such as `nano` and confirm that you're happy with the environment variables and settings in it.

#### Stop the cheqd service

Before backing up your secrets, it's important to stop the cheqd node service or Cosmovisor service; otherwise, the validator private *state* will be left in an inconsistent state and result in an incorrect backup.

If you're running via Cosmovisor (the default option), this can be stopped using:

```bash
sudo systemctl stop cheqd-cosmovisor
```

Or, if running as a standalone service:

```bash
sudo systemctl stop cheqd-node
```

#### Run the Vault backup script

Once you've confirmed the cheqd service is stopped, execute the Vault backup script:

```bash
./vault.sh -b true
```

> We use HashiCorp Vault KV v2 secrets engine. Please make sure that it's enabled and mounted under `cheqd` path.

### Restoring from HashiCorp Vault

To restore backed-up secrets from a Vault server, you can use the same script using the `-r` ("restore") flag:

```bash
./vault.sh -r true
```

#### Restoring secrets to a different machine

If you're restoring to a different machine than the original machine the backup was done from, you'll need to go through the pre-requisites, CLI setup step, and download the Vault backup script to the new machine as well.

In this scenario, you're also also recommended to *disable* the service (e.g., `cheqd-cosmovisor`) on the **original** machine. This ensures that if the (original) machine gets restarted, `systemd` does not try and start the node service as this can potentially result in two validators running with the same validator keys (which will result in tombstoning).

```bash
sudo systemctl disable cheqd-cosmovisor
```

Once you've successfully restored, you can *enable* the service (e.g., `cheqd-cosmovisor`) on the **new** machine:

```bash
sudo systemctl enable cheqd-cosmovisor
```
