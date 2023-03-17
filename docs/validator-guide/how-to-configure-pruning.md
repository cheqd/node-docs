# Pruning Configuration

## Overview

In order to reduce disk utilization on your node, you can enable pruning to reduce the size of data saved to your disk.

This can be done by modifying the pruning parameters inside `/home/cheqd/.cheqdnode/config/app.toml` file.

> :warning: In order to make pruning configuration work, you need to run **cheqd-noded v1.2.5+** Find more details on our [releases page](https://github.com/cheqd/cheqd-node/releases).
  
## Pruning Strategies

There are 4 pruning strategies you can use:

1. `pruning="default"` - This keeps the last 100 states and prunes on 10-block intervals. This configuration is safe to use on all types of nodes.
2. `pruning="nothing"` - This will disable pruning and set your node to archive mode. Although it's totally fine to use this configuration, note that it will significantly increase your disk space usage.
3. `pruning="everything"` - This will only keep the current state and also prune on 10 blocks intervals.
   > ⚠️ It's not recommended to use the `pruning="everything"` configuration on validator nodes!

4. `pruning="custom"` - If you set pruning parameters to custom, you will have to directly specify 2 more parameters to make it work:
   * `pruning-keep-recent` - This will define how many recent states are kept. On some of our nodes, we're setting this to `250`.
   * `pruning-interval` - This will define how often state pruning happens. On some of our nodes, we're setting this to `50`, which means it will start pruning operations after every 50 blocks.

## Reminders

1. If you see `pruning-keep-every`, feel free to remove it from the configuration, since it's been [deprecated](https://github.com/cosmos/cosmos-sdk/pull/11152) for a while.
2. If you're NOT using `pruning="custom"` you should comment out `pruning-interval`, `pruning-keep-every` and `pruning-keep-recent`.
3. Configure `min_retain_blocks` - configuring this parameter will help you to configure Tendermint block pruning and specify the minimum block height offset from the current block being committed. If not enabled previously, this configuration will remove the significant amount of historic data from your node.

   > ℹ️ This configuration is in conjunction with `Unbonding time` the parameter. Based on that and our average block time, we suggest setting the value to not lower than **`250000`** to assure the data integrity on your node. It is calculated by dividing Unbonding period (1,210,000 seconds or 14 days) by the average block time (which is around 6 seconds for most of the time) + adding a significant buffer (~50000 blocks) just for safety.
