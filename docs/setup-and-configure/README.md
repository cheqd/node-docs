# Setup & configure a node

## Context

This document describes how to use install and configure a new instance of `cheqd-node` using an **interactive installer** which supports the following functionality:

1. Setup a new observer/validator node from scratch
2. Configure a node to work with the testnet/mainnet network
3. Upgrade an existing node installation

Alternatively, if you want to manage network upgrades manually, you can also opt for a standadalone installation.

## Pre-requisites for using interactive installer

> ⚠️ Read our guidance on [hardware, software, and networking pre-requisites for nodes](requirements.md) before you get started!
>
> This document specifies the CPU/RAM requirements, firewall ports, and operating system requirements for running cheqd-node.

The interactive installer is written in **Python 3** and is designed to work on **Ubuntu Linux 20.04 LTS** systems. The script has been written to work pre-installed Python 3.x libraries generally available on Ubuntu 20.04.

### Software installed by installer

1. **Cosmovisor** (default, but can be skipped): The installer [configures Cosmovisor](https://docs.cosmos.network/main/tooling/cosmovisor) by default, which is a standard Cosmos SDK tool that makes network upgrades happen in an automated fashion. This makes the process of upgrading to new releases for network-wide upgrades easier.
2. **`cheqd-noded` binary** (mandatory): This is the main piece of ledger-side code each node runs.
3. **Dependencies**: In case you request the installer to restore from a snapshot, dependencies such as `pv` will be installed so that a progress bar can be shown for snapshot extraction. Otherwise, no additional software is installed by the installer.

### External URLs accessed by the installer

* **Github.com**: Fetch latest releases, configuration files, and network settings.
* **Cloudflare DNS** (optional): Used to fetch an externally-resolvable IP address, if this option is selected during install.
* **Network snapshot server** (optional): If requested by the user, the script will fetch latest network snapshots published on [snapshots.cheqd.net](https://snapshots.cheqd.net) and then download snapshot files from the snapshot CDN endpoint ([snapshots-cdn.cheqd.net](https://snapshots-cdn.cheqd.net/))

## Usage

> ⚠️ The guidance below is intended for straightforward new installations or upgrades.
>
> If your scenario is more complex, such as in case of [**upgrading a validator**](../upgrades/README.md) or [**moving a validator**](../validator-guide/move-validator.md), please review the guidance under [our validator guide](../validator-guide/README.md).

By default, the installer will attempt to create a backup of the `~/.cheqdnode/config/` directory and important files under `~/.cheqdnode/data/` before making any destructive changes. These backups are created under the cheqd user's home directory in a folder called `backup` (default location: `/home/cheqd/backup`). However, for safety, **you're recommended to also make manual backups** when upgrading a node.

If you're **setting up a new node from scratch**, you can safely ignore the advice above.

### Stop any running services related to node services

Stop the running services related to your node. If running via Cosmovisor:

```bash
sudo systemctl stop cheqd-cosmovisor.service
```

Or if running standalone:

```bash
sudo systemctl stop cheqd-noded.service
```

### Download and start interactive installer

To get started, download the interactive installer script:

```bash
curl -O https://raw.githubusercontent.com/cheqd/cheqd-node/main/installer/installer.py
```

Then, start the interactive installer:

```bash
sudo python3 installer.py
```

> ℹ️ **NOTE**: You need to execute this as root or *at least* a user with super-user privileges using the `sudo` prefix to the command.

### (Fresh install) Answer prompts for installing from scratch (or overwriting existing)

The interactive installer guides you through setting up and configuring a node installation by asking as series of questions.

All the questions specify the default answer/value for that question in square (`[]`) brackets, for example, `[default: 1]`. If a default value exists, you can just press `Enter` without needing to type the whole answer.

#### 1. Choose release version

Binary release version to install, automatically fetched from Github. The first release displayed in the list will always be the latest stable version. Other versions displayed below it are pre-release/beta versions.
  
```text
Latest stable cheqd-noded release version is Name: v1.3.0
List of cheqd-noded releases: 
1. v1.3.0
2. v1.4.0-develop.1
3. v1.3.1-develop.1
4. v1.3.0-develop.3
5. v1.3.0-develop.2
Choose list option number above to select version of cheqd-node to install [default: 1]:
```

#### 2. Set home directory for cheqd user

By default, a new user/group called `cheqd` will be created and a home directory created for it. The default location is `/home/cheqd` and any configuration data directories being created under this path at `/home/cheqd/.cheqdnode`.

#### 3. Select network to join

Join either the existing [mainnet](https://explorer.cheqd.io) (chain ID: `cheqd-mainnet-1`) or [testnet](https://testnet-explorer.cheqd.io) (chain ID: `cheqd-testnet-6`) network.

```text
Select cheqd network to join:
1. Mainnet (cheqd-mainnet-1)
2. Testnet (cheqd-testnet-6) [default: 1]:
```

#### 4. Choose Cosmovisor configuration options

The next few questions are used to configure Cosmovisor-related options. Read [an explanation of Cosmovisor configuration options in  Cosmos SDK documentation](https://docs.cosmos.network/main/tooling/cosmovisor), or choose to install with the default settings.

1. `Install cheqd-noded using Cosmovisor? (yes/no) [default: yes]`: Use Cosmovisor to run node
2. `Do you want Cosmovisor to automatically download binaries for scheduled upgrades? (yes/no) [default: yes]`: By default, Cosmovisor will attempt to automatically download new binaries that have passed [software upgrade proposals voted on the network](../upgrades/README.md). You can choose to do this manually if you want more control.
3. `Do you want Cosmovisor to automatically restart after an upgrade? (yes/no) [default: yes]`: By default, Cosmovisor will automatically restart the node after an [upgrade height](../upgrades/README.md) is reached and an upgrade carried out.

You can also choose `no` to installing with Cosmovisor on the first question, in which case a standalone binary installation is carried out.

#### 5. Define node configuration

The next set of questions sets common node configuration parameters. These are the *minimal* configuration parameters necessary for a node to function, but advanced users can later customise other settings.

Answers to these prompts are saved in the `app.toml` and `config.toml` files, which are written under `/home/cheqd/.cheqdnode/config/` by default (but can be different if a different home directory was set above). An explanation of some these settings are available in [requirements for running a node](requirements.md) and the [validator guide](../validator-guide/README.md).
See more details about `app.toml` and `config.toml` configuration parameters on [this page](./configure-cheqd-node.md).

1. `Provide a moniker for your cheqd-node [default: <hostname>]:`: Moniker is a human-readable name for your cheqd-node. This is NOT the same as your [validator name](../validator-guide/README.md), and is only used to uniquely identify your node for Tendermint P2P address book.
2. `What is the externally-reachable IP address or DNS name for your cheqd-node? [default: Fetch automatically via DNS resolver lookup]:`: External address is the publicly accessible IP address or DNS name of your cheqd-node. This is used to advertise your node's P2P address to other nodes in the network. If you are running your node behind a NAT, you should set this to your public IP address or DNS name. If you are running your node on a public IP address, you can leave this blank to automatically fetch your IP address via DNS resolver lookup. (Automatic fetching sends a `dig` request to `whoami.cloudflare.com`)
3. `Specify your node's P2P port [default: 26656]`: [Tendermint peer-to-peer traffic port](requirements.md)
4. `Specify your node's RPC port [default: 26657]`: [Tendermint RPC port](requirements.md)
5. `Specify persistent peers [default: none]`: Persistent peers are nodes that you want to always keep connected to. Values for persistent peers should be specified in format: `<nodeID>@<IP>:<port>,<nodeID>@<IP>:<port>`.
6. `Specify minimum gas price [default: 5000ncheq]`: Minimum gas prices is the price you are willing to accept as a validator to process a transaction. Values should be entered in format `<number>ncheq` (e.g., [`5000ncheq`](../../architecture/adr-list/adr-004-token-fractions.md))
7. `Specify log level (trace|debug|info|warn|error|fatal|panic) [default: error]:`: The default log level of `error` is generally recommended for normal operation. You may temporarily need to change to more verbose logging levels if trying to diagnose issues if the node isn't behaving correctly.
8. `Specify log format (json|plain) [default: json]:`: JSON log format allows parsing log files more easily if there's an issue with your node, hence it's set as the default.

#### 6. Choose whether to restore from snapshot

When setting up a new node, you typically need to download all past blocks on the network, including any upgrades that were done along the way with the specific binary releases those upgrades went through.

Since this can be quite cumbersome and take a really long time, the installer offers the ability to download a recent blockchain snapshot for the selected network from [snapshots.cheqd.net](https://snapshot.cheqd.net).

If you skip this step, you'll need to manually synchronise with the network.

> ⚠️ Chain snapshots can range from 10 GBs (for testnet) to 100 GBs (for mainnet). Therefore, this step can take a long time.
>
> If you choose this option, you can step away and return to the installer while it works in the background to complete the rest of the installation. You might want to [change settings in your SSH client / server to keep SSH connections alive](https://www.baeldung.com/linux/ssh-keep-alive), since some hosts terminate connection due to inactivity.

### (Alternative path) Answer prompts for upgrading an existing installation

If you're running the installer on a machine where an existing installation is already present, you'll be prompted whether you want to update/upgrade the existing installation:

```text
Existing cheqd-node binary detected.

Do you want to upgrade an existing cheqd-node installation? (yes/no) [default: yes]:
```

If you choose `no`, this will treat the installation as if installing from scratch and prompt with the questions in section above.

If you choose `yes`, this will retain existing node configuration and prompt with a different set of questions as outlined below. Choosing "yes" is the default since in most cases, you would want to retain the existing configuration while updating the node binary to a newer version.

#### 1. Choose release version for upgrade

Choose binary release version to [upgrade the node](../upgrades/README.md) to.

#### 2. Install or bump Cosmovisor version

If Cosmovisor is detected as installed, you'll be offered the option to bump it to the latest default version. Otherwise, you will be given the option of installing it.

#### 3. Configure Cosmovisor settings

The next section allows you to customise Cosmovisor settings. The explanations of the options are same those given above.

#### 4. Update systemd configuration

By default, the installer will update the `systemd` system service settings for the following:

* `cheqd-cosmovisor.service` (if installed with Cosmovisor) or `cheqd-noded.service` (if installed without Cosmovisor): This is the service that runs the node in the background 24/7.
* `rsyslog.service`: Configures node-specific logging directories and settings.
* `logrotate.service` and `logrotate.timer`: Configures log rotation for node service to limit the duration/size of logs retained to sensible values. By default, this keeps 7 days worth of logs, and compresses logs if they grow larger than 100 MB in size.

### Actions taken by the installer after prompts

Once all prompts have been answered, the installer attempts to carry out the changes requested. This includes:

1. Setting up a new `cheqd` user/group.
2. Downloading `cheqd-noded` and Cosmovisor binaries, as applicable.
3. Setting environment variables required for node binary / Cosmovisor to function.
4. Creating directories for node data and configuration.
5. If present, backing up existing node directories and configuration.
6. Downloading and extracting snapshots (if requested).

The installer is designed to terminate the installation process and stop making changes if it encounters an error. If this happens, please reach out to us on our [community Slack](http://cheqd.link/join-cheqd-slack) or [Discord](http://cheqd.link/discord-github) for how to proceed and recover from errors (if any).

## What to do after the installer finishes

> ⚠️ The guidance below is intended for straightforward new installations or upgrades.
>
> If your scenario is more complex, such as in case of [**upgrading a validator**](../upgrades/README.md) or [**moving a validator**](../validator-guide/move-validator.md), please review the guidance under [our validator guide](../validator-guide/README.md).

If the installer finishes successfully, it will exit with a success message:

```text
[INFO]: Installation steps completed successfully
[INFO]: Installation of cheqd-noded v1.3.0 completed successfully!

[INFO]: Please review the configuration files manually and use systemctl to start the node.

[INFO]: Documentation: https://docs.cheqd.io/node
```

Otherwise, if the installation **failed**, it will exit with an error message which elaborates on the specific error encountered during setup.

The following steps are only recommended if installation has been **successful**.

### Check systemd service is enabled

Check that the node-related `systemd` service is **`enabled`**. This ensures that the node service automatically restarted, even if the service fails or if the machine is rebooted.

If installed *with* Cosmovisor:

```bash
root@hostname ~# systemctl status cheqd-cosmovisor.service
● cheqd-cosmovisor.service - Service for running cheqd-node daemon
  Loaded: loaded (/lib/systemd/system/cheqd-cosmovisor.service; enabled; vendor preset: enabled)
```

The output line after the `systemctl status cheqd-cosmovisor.service` command should say **`enabled`** after `Loaded` path and `vendor preset`.

If installed *without* Cosmovisor (standalone binary install):

```bash
root@hostname ~# systemctl status cheqd-noded.service
● cheqd-noded.service - Service for running cheqd-node daemon
  Loaded: loaded (/lib/systemd/system/cheqd-noded.service; enabled; vendor preset: enabled)
```

The output line after the `systemctl status cheqd-noded.service` command should say **`enabled`** after `Loaded` path and `vendor preset`.

### Start systemd service

Once the node is installed/upgraded, restart the `systemd` service to get the node running. These steps require `root` or super-user privileges as a pre-requisite.

If installed *with* Cosmovisor:

```bash
sudo systemctl start cheqd-cosmovisor.service
```

If installed *without* Cosmovisor:

```bash
sudo systemctl start cheqd-cosmovisor.service
```

### Check node service is running (and stays running)

The command above should start the node service. Ideally, the node service should start running and *remain* running. You can check this by running the command below **a couple of times** in succession and checking that the output line remains as `Active: running` rather than any other status.

(Previous commands can be recalled in bash by pressing the `up` arrow key on your keyboard to repeat or cycle through previous commands.)

If installed *with* Cosmovisor:

```bash
root@hostname ~# systemctl status cheqd-cosmovisor.service
● cheqd-cosmovisor.service - Service for running cheqd-node daemon
  Loaded: loaded (/lib/systemd/system/cheqd-cosmovisor.service; enabled; vendor preset: enabled)
  Active: active (running) since Mon 2023-03-13 14:48:57 GMT; 1 weeks 0 days ago
```

The output line after the `systemctl status cheqd-cosmovisor.service` command should say **`enabled`** after `Loaded` path and `vendor preset`.

If installed *without* Cosmovisor (standalone binary install):

```bash
root@hostname ~# systemctl status cheqd-noded.service
● cheqd-noded.service - Service for running cheqd-node daemon
  Loaded: loaded (/lib/systemd/system/cheqd-noded.service; enabled; vendor preset: enabled)
  Active: active (running) since Mon 2023-03-13 14:48:57 GMT; 1 weeks 0 days ago
```

### Confirm that node is gaining new blocks

Once the `systemd` service is confirmed as running, check that the node is catching up on new blocks by **repeating this command 3-5 times**:

(Previous commands can be recalled in bash by pressing the `up` arrow key on your keyboard to repeat or cycle through previous commands.)

```bash
cheqd-noded status
```

**Note**: The `cheqd-noded status` may not return a successful response immediately after starting the `systemd` service. For instance, you might get the following output:

```bash
root@hostname ~# cheqd-noded status
Error: post failed: Post "http://localhost:26657": dial tcp [::1]:26657: connect: connection refused
```

If you encounter the output above, *as long as `systemctl status ...` returns `Active`*, this "error" above is completely normal. This is because it takes a few minutes after `systemctl start` for the node services to properly start running. Please wait for a few minutes, and then re-run the `cheqd-noded status` command.

The output might say `catching_up: true` if the node is still catching up, or `catching_up: false` if it's fully caught up.

```bash
root@hostnames ~# cheqd-noded status
{"NodeInfo":{"protocol_version":{"p2p":"8","block":"11","app":"0"},"id":"<node-id>","listen_addr":"<external-address>:26656","network":"cheqd-mainnet-1","version":"0.34.26","channels":"40202122233038606100","moniker":"<moniker>","other":{"tx_index":"on","rpc_address":"tcp://0.0.0.0:26657"}},"SyncInfo":{"latest_block_hash":"082E9ACC41E72BCB566DDF9132B9011C470629F54E9F0F32B0A619265A42F1F6","latest_app_hash":"302F62AB35B8D463D398E83CF7C0597A2DCC27DF7C3C5165F7AEC4CA6B4C3C7F","latest_block_height":"7153781","latest_block_time":"2023-03-20T18:02:41.952748008Z","earliest_block_hash":"A20320D647F4EC8E6D7EC95A3F25B13B86807907BD8231A24BF3EA8171E645D3","earliest_app_hash":"2B7DA548AFF3FA8886E602993BB13CFCD0E97E7F943AF129B5DBA92284B9284A","earliest_block_height":"6903781","earliest_block_time":"2023-03-03T16:12:02.371081673Z","catching_up":false},"ValidatorInfo":{"Address":"9CE4242862F7CB95B22D2EC60CB625F6B09C9562","PubKey":{"type":"tendermint/PubKeyEd25519","value":"CGe/qgutfjKuPSFQK3ZBraNRjcyKzEvy6hd2JX73Vns="},"VotingPower":"35373490103"}}
```

If the node is catching up, the time needed to fully catch up will depend on how far behind your node is. The `latest_block_height` value in the output shown above will indicate how far behind the node is. This number should display a larger value every time you re-run the command.

> ❓ The absolute newest block height across the entire network is displayed in the block explorer. Check the [mainnet explorer](https://explorer.cheqd.io) or the [testnet explorer](https://testnet-explorer.cheqd.io) (depending on which network you've joined) to understand the network-wide latest block height vs your node's delta.

## Next steps

If you're configuring a validator, check out [**our validator guide**](../validator-guide/README.md) for further configuration steps to carry out.

Additonally, you read [**here**](./cosmovisor-configuration.md) about cosmovisor-specific configuration parameters and how to change them.
