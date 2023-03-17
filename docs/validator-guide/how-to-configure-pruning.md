# Pruning Configuration

In order to reduce disk utilization on your node, you can always use enable pruning to reduce the size of data saved to your disk.

This can be done by modifying the pruning parameters inside `/home/cheqd/.cheqdnode/config/app.toml` file.

> :warning: In order to make pruning configuration work, you need to run **cheqd-noded v1.2.5+** Find more details on our [releases page](https://github.com/cheqd/cheqd-node/releases).
  
There are 4 pruning strategies you can use:

* `pruning="default"` - Keeps the last 100 states and prunes on 10-block intervals. This configuration is safe to use on all types of nodes.
* `pruning="nothing"` - This will disable pruning and set your node to archive mode. Although it's totally fine to use this configuration, note that it will significantly increase your disk space usage.
*`pruning="everything"` - This will only keep the current state and also prune on 10 blocks intervals.

> :exclamation: **It's not recommended to use this configuration on validator nodes!**

* `pruning="custom"` - If you set pruning parameters to custom, you will have to directly specify 2 more parameters to make it work:
    * `pruning-keep-recent` - This will define how many recent states are kept. On some of our nodes, we're setting this to `250` .
    * `pruning-interval` - This will define how often state pruning happens. On some of our nodes, we're setting this to `50`, which means it will start pruning operations after every 50 blocks.

        > :memo: In case you see `pruning-keep-every`, feel free to remove it from the configuration, since it's been [deprecated](https://github.com/cosmos/cosmos-sdk/pull/11152) for a while.

> :warning: Make sure to comment out `pruning-interval`, `pruning-keep-every` and `pruning-keep-recent` in case you're not using `pruning="custom"`.

* `min_retain_blocks` - Configuring this parameter will help you to configure Tendermint block pruning - specify the minimum block height offset from the current block being committed. If not enabled previously, this configuration will remove the significant amount of historic data from your node.

    > :exclamation: This configuration is in conjunction with `Unbonding time` the parameter. Based on that and our average block time, we suggest setting the value to not lower than **`250000`** to assure the data integrity on your node.
    > It is calculated by dividing Unbonding period (1,210,000 seconds or 14 days) by the average block time (which is around 6 seconds for most of the time) + adding a significant buffer (~50000 blocks) just for safety.
