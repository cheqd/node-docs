# Guide for re-enabling pruning and recovering node db  
This guide is specifically made for the the validators/node operators affected by the pruning issue which caused a mainnet halt on September 6th 2025. 

Re enabling pruning (to `default`/`custom` from `nothing`) on the affected nodes will cause the nodes to halt again as the database pruning resumes its operation. To avoid this it is recommended to reset your node db and recover it using state sync or a db snapshot. The nodes that weren't affected by the pruning issue or that had recovered by the time the `nothing`  pruning fix was rolled out do not have to undergo this procedure.

## State Sync:-
Stop the systemd service of the node. 
`sudo systemctl stop cheqd-cosmovisor.service`

Take a backup of the priv_validator_state.json. **This step is very Important for validator nodes.**
`cp ~/.cheqdnode/data/priv_validator_state.json ~/priv_validator_state.json`

Turn the pruning strategy to default or custom based on your preference in the `~/.cheqdnode/config/app.toml` file.

Reset the database.
`cheqd-noded tendermint unsafe-reset-all --home ~/.cheqdnode/ --keep-addr-book`

Restore the priv_validator_state.json . **This step is very Important for validator nodes**
`cp ~/priv_validator_state.json ~/.cheqdnode/data/priv_validator_state.json` 

Enable statesync on the node and provide the required variables.
```
STATESYNC_RPC="https://rpc.cheqd.net:443"
LATEST_HEIGHT=$(curl -s $STATESYNC_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$STATESYNC_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$STATESYNC_RPC,$STATESYNC_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" ~/.cheqdnode/config/config.toml
```

Start the node
`sudo systemctl restart cheqd-cosmovisor.service`

The node should start looking for statesync chunks from it's peers and begin the restoration process in a few minutes. 

## Snapshot:-
Stop the systemd service of the node. 
`sudo systemctl stop cheqd-cosmovisor.service`
 
Take a backup of the priv_validator_state.json. **This step is very Important for validator nodes.**
`cp ~/.cheqdnode/data/priv_validator_state.json ~/priv_validator_state.json`
 
Turn the pruning strategy to default or custom based on your preference in the `~/.cheqdnode/config/app.toml` file.
 
Reset the database.
`cheqd-noded tendermint unsafe-reset-all --home ~/.cheqdnode/ --keep-addr-book`
 
Restore the priv_validator_state.json. **This step is very Important for validator nodes**
`cp ~/priv_validator_state.json ~/.cheqdnode/data/priv_validator_state.json` 

Download the latest lz4 tar archive for mainnet from this link https://snapshots.cheqd.net/#mainnet/
`wget https://cheqd-node-backups.ams3.digitaloceanspaces.com/mainnet/<timestamp>/cheqd-mainnet-1_<timestamp>.tar.lz4`  

Unpack the tar archive and restore the db
`lz4 -c -d cheqd-mainnet-1_<timestamp>.tar.lz4 | tar -x -C ~/.cheqdnode` 
 
Start the node
`sudo systemctl restart cheqd-cosmovisor.service`