# Configure State Sync

State Sync allows node operators to initialize their nodes quickly with much less storage consumed.
This is really useful in case you don't need full (or significant) chain history and it significantly reduces costs. For comparison, our state DB snapshots published on snapshots.cheqd.net, typically have around 600GB of data, while after State Sync initialization, node consumes only around 540 MB of data.

## Modifying config.toml file

In order to enable State Sync, you need to update the Comet BFT configuration file, like this:

```toml
#######################################################
###         State Sync Configuration Options        ###
#######################################################
[statesync]
# State sync rapidly bootstraps a new node by discovering, fetching, and restoring a state machine
# snapshot from peers instead of fetching and replaying historical blocks. Requires some peers in
# the network to take and serve state machine snapshots. State sync is not attempted if the node
# has any local state (LastBlockHeight > 0). The node will have a truncated block history,
# starting from the height of the snapshot.
enable = true

# RPC servers (comma-separated) for light client verification of the synced state machine and
# retrieval of state data for node bootstrapping. Also needs a trusted height and corresponding
# header hash obtained from a trusted source, and a period during which validators can be trusted.
#
# For Cosmos SDK-based chains, trust_period should usually be about 2/3 of the unbonding time (~2
# weeks) during which they can be financially punished (slashed) for misbehavior.
rpc_servers = "https://eu-rpc.cheqd.net:443,https://ap-rpc.cheqd.net:443"
trust_height = 20748000
trust_hash = "BD43534FB20E7C163917BC1A501349C68656A9C24EE48C92BB82FFEE687EEE14"
trust_period = "168h0m0s"
```

As you can see, first you need to **enable** the State Sync module.
After that, you need to provide **RPC endpoints** that expose and serve state sync snapshots. You can use our public RPC endpoints, as we generate and serve state sync snapshots from them.
After that, you need to add the `trust_height` - this is usually $CURRENT_BLOCK_HEIGHT - 2000 blocks (which is the interval at we generate new snapshots on our RPC nodes).
Finally, you need to add the block ID hash, as `trust_hash` parameter. This can be fetched via RPC, like this - `https://rpc.cheqd.net/block?height=20748000`. You will find block hash under `.result.block_id.hash`.

Reference: [CometBFT State Sync docs](https://docs.cometbft.com/v0.34/core/state-sync)

You can also use this simple bash script to update your node configuration:

```bash
#!/bin/bash
set -euo pipefail

# set environment variables
export RPC1=https://eu-rpc.cheqd.net:443 # use https://eu-rpc.cheqd.network:443 for testnet
export RPC2=https://ap-rpc.cheqd.net:443 # use https://ap-rpc.cheqd.network:443 for testnet

INTERVAL=2000
CONFIG=/home/cheqd/.cheqdnode/config/config.toml

# GET TRUST HASH AND TRUST HEIGHT
LATEST_HEIGHT=$(curl -s "$RPC1/block" | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT-INTERVAL))
TRUST_HASH=$(curl -s "$RPC1/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

# UPDATE [statesync] block in config.toml (in-place, no backup)
sed -i -E '/^\[statesync\]/,/^\[/{s/^enable\s*=.*/enable = true/}' "$CONFIG"
sed -i -E "/^\[statesync\]/,/^\[/{s|^rpc_servers\s*=.*$|rpc_servers = \"${RPC1},${RPC2}\"|}" "$CONFIG"
sed -i -E "/^\[statesync\]/,/^\[/{s|^trust_height\s*=.*$|trust_height = ${BLOCK_HEIGHT}|}" "$CONFIG"
sed -i -E "/^\[statesync\]/,/^\[/{s|^trust_hash\s*=.*$|trust_hash = \"${TRUST_HASH}\"|}" "$CONFIG"

echo "Configuration file updated successfully."
```
