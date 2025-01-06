# Optimising disk storage with pruning

## Context

Cosmos SDK and Tendermint has a concept of _pruning_, which allows reducing the disk utilisation and [storage required on a node](../setup-and-configure/requirements.md).

There are two kinds of pruning controls available on a node:

1. **Tendermint pruning**: This impacts the `~/.cheqdnode/data/blockstore.db/` folder by only retaining the last _n_ specified blocks. Controlled by the `min-retain-blocks` parameter in `~/.cheqdnode/config/app.toml`.
2. **Cosmos SDK pruning**: This impacts the `~/.cheqdnode/data/application.db/` folder and prunes Cosmos SDK app-level state (a logical layer higher than Tendermint, which is just peer-to-peer). These are set by the `pruning` parameters in the `~/.cheqdnode/config/app.toml` file.

This can be done by modifying the pruning parameters inside `/home/cheqd/.cheqdnode/config/app.toml` file.

## Instructions

> ⚠️ In order for either type of pruning to work, your node should be running the [latest stable release of cheqd-node](https://github.com/cheqd/cheqd-node/releases/latest) (at least **v1.3.0+**).

You can check which version of `cheqd-noded` you're running using:

```bash
cheqd-noded version
```

The output should be a version higher than v1.3.0. If you're on a lower version, you either manually upgrade the node binary or [use the interactive installer to execute an upgrade](../setup-and-configure/) while retaining settings.

The instructions below assume that the home directory for the `cheqd` user is set to the default value of `/home/cheqd`. If this is _not_ the case for your node, please modify the commands below to the correct path.

### Check systemd service status

Follow [similar instructions as mentioned in the installer guide](../setup-and-configure/) to check systemd service status:

```bash
systemctl status cheqd-cosmovisor.service
```

(Substitute with `cheqd-noded.service` if you're running a standlone node rather than with Cosmovisor.)

### Switch to the `cheqd` user and configuration directory

Switch to the `cheqd` user and then the `.cheqdnode/config/` directory.

```bash
sudo su cheqd
cd ~
```

### Display current directory usage/size for the node data folder

Before you make changes to pruning configuration, you might want to capture the _existing_ usage first (only copy the command bit, not the full line):

```bash
root@hostname ~# du -h -d 1 /home/cheqd/.cheqdnode/data/ 2>/dev/null
60G     /home/cheqd/.cheqdnode/data/blockstore.db
80K     /home/cheqd/.cheqdnode/data/evidence.db
1021M   /home/cheqd/.cheqdnode/data/cs.wal
274G    /home/cheqd/.cheqdnode/data/application.db
50G     /home/cheqd/.cheqdnode/data/tx_index.db
72K     /home/cheqd/.cheqdnode/data/snapshots
45G     /home/cheqd/.cheqdnode/data/state.db
428G    /home/cheqd/.cheqdnode/data/
```

The `du -h -d 1 ...` command above prints the disk usage for the specified folder down to one folder level depth (`-d 1` parameter) and prints the output in GB/MB (`-h` parameter, which prints in human-readable values).

### Open the `app.toml` file for editing

Open the `app.toml` file once you've switched to the `~/.cheqdnode/config/` folder using your preferred text editor, such as `nano`:

```bash
cd ~/.cheqdnode/config/
nano app.toml
```

Instructions on how to use text editors such as `nano` is out of the scope of this document. If you're unsure how to use it, consider following [guides on how to use `nano`](https://serverpilot.io/docs/how-to-use-nano-to-edit-files/).

### Choose a Cosmos SDK pruning strategy

> ⚠️ If your node was configured to work with **release version v1.2.2 or earlier**, you may have been advised to run in `pruning="nothing"` mode due to a bug in Cosmos SDK.
>
> Ensure you've upgraded to the [latest stable release](https://github.com/cheqd/cheqd-node/releases/latest) ([using the installer](../setup-and-configure/)) or otherwise. When running a validator node, you're recommended to change this value to `pruning="default"`.

The file should already be populated with values. Edit the `pruning` parameter value to one of following:

1. `pruning="nothing"` (_highest_ disk usage): This will _disable_ Cosmos SDK pruning and set your node to behave like an "archive" node. This mode consumes the highest disk usage.
2. `pruning="default"` (**recommended**, moderate disk usage): This keeps the last 100 states in addition to every 500th state, and prunes on 10-block intervals. This configuration is safe to use on all types of nodes, especially validator nodes.
3. `pruning="everything"` (_lowest_ disk usage): This mode is **not recommended** when running validator nodes. This will keep the current state and also prune on 10 blocks intervals. This settings is useful for nodes such as seed/sentry nodes, as long as they are not used to query RPC/REST API requests.
4. `pruning="custom"` (custom disk usage): If you set the `pruning` parameter to `custom`, you will have to modify two additional parameters:
   * `pruning-keep-recent`: This will define how many recent states are kept, e.g., `250` (contrast this against `default`).
   * `pruning-interval`: This will define how often state pruning happens, e.g., `50` (contrast against `default`, which does it every 10 blocks)
   * `pruning-keep-every`: This parameter is deprecated in newer versions of Cosmos SDK. You can delete this line if it's present in your `app.toml` file.

Although the paramters named `pruning-*` are _only_ supposed to take effect if the pruning strategy is `custom`, in practice it seems that in Cosmos SDK v0.46.10 these settings still impact pruning. Therefore, you're advised to **comment out these lines** when using `default` pruning.

Example configuration file with recommended settings:

```toml
pruning = "default"

# These are applied if and only if the pruning strategy is custom.
#pruning-keep-recent = "0"
#pruning-interval = "0"
```

### Choose a Tendermint pruning strategy

Configuring `min-retain-blocks` parameter to a non-zero value activates Tendermint pruning, which specifies minimum block height to retain. By default, this parameter is set to `0`, which disables this feature.

Enabling this feature can reduce disk usage significantly. **Be careful in setting a value**, as it must be _at least_ higher than 250,000 as calculated below:

* [Unbonding time](../../architecture/adr-list/adr-005-genesis-parameters.md) (14 days) converted to seconds = 1,210,000 seconds
* ...divided by average block time = approx. 6s / block
* \= approx. 210,000 blocks
* Adding a safety margin (in case average block time goes down) = approx. 250,000 blocks

Therefore, this setting must always be updated to carefully match a valid value in case the unbonding time on the network you're running on is different. (E.g., this value is different on mainnet vs testnet due to different unbonding period.)

Using the recommended values, on the current cheqd mainnet this section would look like the following:

```toml
# Note: Tendermint block pruning is dependant on this parameter in conunction
# with the unbonding (safety threshold) period, state pruning and state sync
# snapshot parameters to determine the correct minimum value of
# ResponseCommit.RetainHeight.
min-retain-blocks = 250000
```

### Save changes in the file

Save and exit from the `app.toml` file. Working with text editors is outside the scope of this document, but in general under `nano` this would be `Ctrl+X`, "yes" to `Save modified buffer`, then `Enter`.

### Switch user and restart service

> ℹ️ **NOTE**: You need root or _at least_ a user with super-user privileges using the `sudo` prefix to the commands below when interact with systemd.

If you switched to the cheqd user, exit out to a root/super-user:

```bash
exit
```

Usually, this will switch you back to `root` or other super-user (e.g., `ubuntu`).

Restart systemd service:

```bash
sudo systemctl restart cheqd-cosmovisor.service
```

(Substitute with `cheqd-noded.service` above if you're running without Cosmovisor)

Check the systemd service status and confirm that it's running:

```bash
systemctl status cheqd-cosmovisor.service
```

Our [installer guide](../setup-and-configure/) has a section on how to check service status.

## Next steps

If you activate/modify any pruning configuration above, the changes to disk usage are NOT immediate. Typically, it **may take 1-2 days** over which the disk usage reduction is progressively applied.

If you've gone from a higher disk usage setting to a lower disk usage setting, re-run the disk usage command to comapre the breakdown of disk usage in the node data directory:

```bash
du -h -d 1 /home/cheqd/.cheqdnode/data/ 2>/dev/null
```

The output shown _should_ show a difference in disk usage from the previous run before settings were changed for the `application.db` folder (if the `pruning` parameters were changed) and/or the `blockstore.db` folder (if `min-retain-blocks`) was changed.

## Third-Party Snapshots for Reference

Instead of syncing a full node from scratch, validators can leverage pruned snapshots provided by trusted third-party services. These snapshots are substantially smaller (usually 3-10 GB compared to 100s of GBs for a full node), allowing faster setup and reduced storage requirements.

### **Available Third-Party Snapshots**

1. **NodeStake**
   * Website: [https://nodestake.org/cheqd](https://nodestake.org/cheqd)
   * Instructions: Navigate to the **Snapshot** tab for detailed steps on downloading and using their snapshots.
2. **Nysa Network**
   * Website: [https://snapshots.nysa.network/#cheqd-mainnet-1/](https://snapshots.nysa.network/#cheqd-mainnet-1/)
   * Snapshot details and setup instructions are provided on the site.
3. **Lavender.Five Nodes**
   * Website: [https://services.lavenderfive.com/mainnet/cheqd/snapshot](https://services.lavenderfive.com/mainnet/cheqd/snapshot)
   * Visit the page for links and commands to restore a node from their pruned snapshots.
