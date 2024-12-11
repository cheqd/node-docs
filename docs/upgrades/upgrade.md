# Upgrade

After the proposal status is marked as passed, the upgrade plan will become active. You can query the upgrade plan details using the following command:

```bash
cheqd-noded query upgrade plan --chain-id cheqd-mainnet-1
```

This will return output similar to:

```bash
height: "1000"
info: ""
name: <name of proposal>
```

At block height 1000, the `BeginBlocker` will be triggered. At this point, the node will be out of consensus, awaiting the upgrade to the new version.

The log messages like these should be expected:

```bash
5:17PM ERR UPGRADE "<proposed upgrade name>" NEEDED at height: 1000:
5:17PM ERR UPGRADE "<proposed upgrade name>" NEEDED at height: 1000:
panic: UPGRADE "<proposed upgrade name>" NEEDED at height: 1000:
```

Once the new application version is installed and running and 1/3 of the voting power on the network is restored, the node will resume normal operations.

## Automatic upgrades with Cosmovisor

If your node is configured to use **Cosmovisor**, the upgrade action will be performed **automatically** at the specified block height.

To check if your node is configured to run with Cosmovisor, run the following command:

```bash
systemctl status cheqd-cosmovisor.service
```

If there's a running systemd service, running this sub-process `/usr/bin/cosmovisor run start`, then your node is using Cosmovisor.

## Manual Upgrades for Standalone Nodes

For **standalone nodes**, follow the instructions in [our installation guide](../setup-and-configure/README.md). Make sure to choose the release suggested in the software upgrade proposal.

## Node Built from Source Code

If you are running a node built from source, you will need to:

1. Refer to the upgrade proposal details.
2. Check out the **git tag** corresponding to the latest release. This is important in cases code in our **main** branch doesn't match the latest release.
3. Build the updated binary.

## Docker Users

Additionally, we're also publishing the docker images in our [GitHub Container Registry](https://github.com/cheqd/cheqd-node/pkgs/container/cheqd-node), so in case you're running cheqd node in Docker, you can always find the latest image there.
