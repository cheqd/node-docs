# Backup/Restore keys for nodes (using Hashicorp Vault)

It's very important that we maintain a backup of our private keys and secrets. When it comes to Cosmos based networks,
it basically boils down to the following files:

* `$CHEQD_NODE_HOME_DIR/config/node_key.json`
* `$CHEQD_NODE_HOME_DIR/config/priv_validator_key.json`
* `$CHEQD_NODE_HOME_DIR/data/priv_validator_state.json`
* `$CHEQD_NODE_HOME_DIR/data/upgrade-info.json`

> `$CHEQD_NODE_HOME_DIR` is the data directory for your node, which defaults to `/home/cheqd/.cheqdnode`

If we ever accidentally delete/lose these files, there's no way to recover them. So to avoid such situations we must
back these files up and store them securely. In this case, we're going to use Hashicorp's Vault,
which is an Open Source, battle tested and well recognised tool to manage secrets, encryption keys, certifications, etc.
We're going to assume that we have a vault cluster deployment running for us.

> If you don't have an existing deployment of Hashicorp Vault, [Learn by Hashicorp Vault](https://learn.hashicorp.com/vault)
is a good starting point to get your own cluster running

How To:
-------

First of all, we to install Vault CLI on our machine, the best way to do this is by following the guide on Hashicorp's Website:

[Install Vault ![](https://www.google.com/s2/favicons?domain_url=https%3A%2F%2Fwww.vaultproject.io%2Fdocs%2Finstall)
https://www.vaultproject.io/docs/install](https://www.vaultproject.io/docs/install)

After installing the CLI, we need to do some setup, set some environment variables in your shell environment, so that
the Vault CLI knows where to store the credentials and where to pull them from. Export the following variable in your
shell (Sh/Bash/ZSH) (don't forget to replace the placeholder values with real values)

```bash
export VAULT_ADDR="[VAULT_CLUSTER_ENDPOINT]"
export VAULT_NAMESPACE=[NAMESPACE]
export VAULT_TOKEN=[VAULT_TOKEN]
export CHEQD_HOME=[CHEQD_NODE_HOME_DIR]
export MONIKER=[NODE_MONIKER]
```

```bash
# for example
# export VAULT_ADDR="https://vault.example.com"
# export VAULT_NAMESPACE="admin/testnet"
# export VAULT_TOKEN=s.slkjaklfjsdf98dfsd7sd.lskdj
# export CHEQD_HOME=/cheqd
# export MONIKER=seed1-ap
```

Let's create a script named `vault.sh`, to easily backup and restore the credentials from/to our machine. Copy the
following content inside your script:

```bash
#!/bin/bash

set -oex
# VAULT_ADDR
# VAULT_NAMESPACE
# VAULT_TOKEN
# CHEQD_HOME
# MONIKER

show_help=false
restore=false
backup=false
NODE_KEY=$CHEQD_HOME/.cheqdnode/config/node_key.json
VALIDATOR_PRIV_KEY=$CHEQD_HOME/.cheqdnode/config/priv_validator_key.json

while getopts b:r: flag; do
    case "${flag}" in
        b) backup=${OPTARG};;
        r) restore=${OPTARG};;
        *) show_help=true;;
    esac
done

if $restore; then
    echo "restoring keys from vault. You have 10 second to cancel this (use CTRL+c)"
    sleep 10
    if [ -f "$NODE_KEY" ]; then
            echo "node_key.json is alreay present at the path $NODE_KEY, creating a backup"
            echo
            cp $NODE_KEY $NODE_KEY.bak
    fi
    if [ -f "$VALIDATOR_PRIV_KEY" ]; then
            echo "priv_validator_key.json is alreay present at the path $VALIDATOR_PRIV_KEY, creating a backup"
            echo
            cp $VALIDATOR_PRIV_KEY $VALIDATOR_PRIV_KEY.bak
    fi

    vault kv get -field=passcode cheqd/$ONIKER-node-key > $NODE_KEY
    vault kv get -field=passcode cheqd/$MONIKER-priv-validator-key > $VALIDATOR_PRIV_KEY
fi

if "$backup"; then
    echo "backing up keys to vault. You have 10 second to cancel this (use CTRL+c)"
    sleep 10
    vault kv put cheqd/$MONIKER-node-key passcode=@$CHEQD_HOME/.cheqdnode/config/node_key.json
    vault kv put cheqd/$MONIKER-priv-validator-key passcode=@$CHEQD_HOME/.cheqdnode/config/priv_validator_key.json
fi

if $show_help; then
    echo "use -b for backup and -r for restore"
fi
```

> We use Vault KV v2 secrets engine, please make sure that it's enabled and mounted under `cheqd` path.

Now, if we want to backup the credentials, we can run:

```bash
bash vault.sh -b true
```

To restore the credentials (assuming we already backed them up previously):

```bash
bash vault.sh -r true
```
