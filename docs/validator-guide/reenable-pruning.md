# Guide for re-enabling pruning and recovering node db

This guide is specifically made for the the validators/node operators affected by the pruning issue encountered following our v4.x upgrade. This issue required some validators/node operators having to disable pruning entirely.

Re-enabling pruning (to `default`/`custom` from `nothing`) on the affected nodes will cause the nodes to halt again as the database pruning resumes its operation. To avoid this it is recommended to reset your node db and recover it using state sync or a db snapshot. The nodes that weren't affected by the pruning issue or that had recovered by the time the pruning fix was rolled out do not have to undergo this procedure.

Following this procedure will significantly reduce the disk space required for your node’s regular operations, thereby lowering operational costs (on the nodes we manage, we observed storage usage drop from 700+ GB to under 10 GB). Additionally, running a node with less disk usage will likely improve performance.

You have two options here:

1) Reset via State Sync
2) Reset by using DB snapshot

## State Sync

Stop the systemd service of the node.
`sudo systemctl stop cheqd-cosmovisor.service`

Take a backup of the priv_validator_state.json. **This step is very Important for validator nodes:**

```bash
cp ~/.cheqdnode/data/priv_validator_state.json ~/priv_validator_state.json
```

Turn the pruning strategy to default or custom based on your preference in the `~/.cheqdnode/config/app.toml` file.

Reset the database:

```bash
cheqd-noded tendermint unsafe-reset-all --home ~/.cheqdnode/ --keep-addr-book
```

Restore the priv_validator_state.json. **This step is very Important for validator nodes:**

```bash
cp ~/priv_validator_state.json ~/.cheqdnode/data/priv_validator_state.json
```

Enable statesync on the node and provide the required variables:

```bash
STATESYNC_RPC="https://rpc.cheqd.net:443"
LATEST_HEIGHT=$(curl -s $STATESYNC_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$STATESYNC_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$STATESYNC_RPC,$STATESYNC_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" ~/.cheqdnode/config/config.toml
```

Start the node:

```bash
sudo systemctl restart cheqd-cosmovisor.service
```

The node should start looking for statesync chunks from it's peers and begin the restoration process in a few minutes. After some time, it should catch up with the network and continue siging blocks.

## Snapshot

Stop the systemd service of the node:

```bash
sudo systemctl stop cheqd-cosmovisor.service`
```

Take a backup of the priv_validator_state.json. **This step is very Important for validator nodes:**

```bash
cp ~/.cheqdnode/data/priv_validator_state.json ~/priv_validator_state.json
```

Turn the pruning strategy to default or custom based on your preference in the `~/.cheqdnode/config/app.toml` file.

Reset the database:

```bash
cheqd-noded tendermint unsafe-reset-all --home ~/.cheqdnode/ --keep-addr-book`
```

Restore the priv_validator_state.json. **This step is very Important for validator nodes:**

```bash
cp ~/priv_validator_state.json ~/.cheqdnode/data/priv_validator_state.json
```

Download the latest lz4 tar archive for mainnet from [our snapshots page](https://snapshots.cheqd.net/#mainnet/)

```bash
wget https://cheqd-node-backups.ams3.digitaloceanspaces.com/mainnet/<timestamp>/cheqd-mainnet-1_<timestamp>.tar.lz4
```

Unpack the tar archive and restore the db:

```bash
lz4 -c -d cheqd-mainnet-1_<timestamp>.tar.lz4 | tar -x -C ~/.cheqdnode` 
```

Start the node

```bash
sudo systemctl restart cheqd-cosmovisor.service
```
