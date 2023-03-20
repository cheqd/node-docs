# Installing a cheqd node from binary package releases

## Context

This document provides guidance on how to install and configure a node for the cheqd testnet from [our binary package releases](https://github.com/cheqd/cheqd-node/releases/latest).

If you are planning to install a node on a host machine running Ubuntu 20.04 LTS, our recommended method is to use the [Interactive installation guide](interactive/interactive-installer.md) instead.

The [node setup guide provides pre-requisites](README.md) needed before the steps below should be attempted.

## Installation steps for `cheqd-node` binary

1. **Get binary**

   You can get the binary in several ways:

   * [Compile from source](../build-and-networks/README.md)
   * Getthe binary compiled for Ubuntu 20.04 in [releases](https://github.com/cheqd/cheqd-node/releases);

2. **Define configuration for running the `cheqd-noded` binary as a service**

   It is highly recommended to run the `cheqd-node` as a system service using a supervisor such as `systemd`.

   You can use such [postinst](https://github.com/cheqd/cheqd-node/blob/main/build-tools/node-standalone.service) script for setting up our binary as a service. The same tool can be used to set up the binary as a service.

   There is only one input parameter for `postinst` script, it's a path to where binary is.

   To set up the binary using `postint`, execute the following with sudo privileges:

   ```bash
   bash postinst <path/to/cheqd-noded/binary>
   ```

   This will add a service file and prepare all needed directories for `configs/keys` and `data`. The script also creates a new service user called `cheqd`, to ensure that all processes and directories related to `cheqd-noded` are isolated under that service user.

3. **Initialise the node configuration files**

   The "moniker" for your node is a "friendly" name that will be available to peers on the network in their address book, and allows easily searching a peer's address book.

   ```bash
   cheqd-noded init <node-moniker>
   ```

4. **Download the genesis file for a persistent chain, such as the cheqd testnet**

   Download the `genesis.json` file for the relevant [persistent chain](https://github.com/cheqd/cheqd-node/tree/main/networks/) and put it in the `$HOME/.cheqdnode/config` directory.

   For cheqd mainnet:

   ```bash
   wget -O $HOME/.cheqdnode/config/genesis.json https://raw.githubusercontent.com/cheqd/cheqd-node/main/networks/mainnet/genesis.json
   ```

   For cheqd testnet:

   ```bash
   wget -O $HOME/.cheqdnode/config/genesis.json https://raw.githubusercontent.com/cheqd/cheqd-node/main/networks/testnet/genesis.json
   ```

5. **Define the seed configuration for populating the list of peers known by a node**

   Update `seeds` with a comma separated list of seed node addresses specified in `seeds.txt` for the relevant [network](https://github.com/cheqd/cheqd-node/tree/main/networks/).

   For cheqd mainnet, set the `SEEDS` environment variable:

   ```bash
   SEEDS=$(wget -qO- https://raw.githubusercontent.com/cheqd/cheqd-node/main/networks/mainnet/seeds.txt)
   ```

   For cheqd testnet, set the `SEEDS` environment variable:

   ```bash
   SEEDS=$(wget -qO- https://raw.githubusercontent.com/cheqd/cheqd-node/main/networks/testnet/seeds.txt)
   ```

   After the `SEEDS` variable is defined, pass the values to the `cheqd-noded configure` tool to set it in the configuration file.

   ```bash
   $ echo $SEEDS
   # Comma separated list should be printed
   
   $ cheqd-noded configure p2p seeds "$SEEDS"
   ```

6. **Set gas prices accepted by the node**

   Update `minimum-gas-prices` parameter if you want to use custom value. The default is `25ncheq`.

   ```bash
   cheqd-noded configure min-gas-prices "50ncheq"
   ```

7. **Define the external peer-to-peer address**

   Unless you are running a node in a sentry/validator two-tier architecture, your node should be reachable on its peer-to-peer (P2P) port by other other nodes. This can be defined by setting the `external-address` property which defines the externally reachable address. This can be defined using either IP address or DNS name followed by the P2P port (Default: 26656).

   ```bash
   cheqd-noded configure p2p external-address <ip-address-or-dns-name:p2p-port>
   # Example
   # cheqd-noded configure p2p external-address 8.8.8.8:26656
   ```

   This is especially important if the node has no public IP address, e.g., if it's in a private subnet with traffic routed via a load balancer or proxy. Without the `external-address` property, the node will report a private IP address from its own host network interface as its `remote_ip`, which will be unreachable from the outside world. The node still works in this configuration, but only with limited unidirectional connectivity.

8. **Make the RPC endpoint available externally** (optional)

      This step is necessary only if you want to allow incoming client application connections to your node. Otherwise, the node will be accessible only locally. Further details about the RPC endpoints is available in the [cheqd node setup guide](../setup-and-configure/README.md).

      ```bash
      cheqd-noded configure rpc-laddr "tcp://0.0.0.0:26657"
      ```

9. **Enable and start the `cheqd-noded` system service**

      If you are prompted for a password for the `cheqd` user, type `exit` to logout and then attempt to execute this as a privileged user (with `sudo` privileges or as root user, if necessary).

      ```bash
      $ systemctl enable cheqd-noded
      Created symlink /etc/systemd/system/multi-user.target.wants/cheqd-noded.service → /lib/systemd/system/cheqd-noded.service.

      $ systemctl start cheqd-noded
      ```

      Check that the `cheqd-noded` service is running. If successfully started, the status output should return `Active: active (running)`

      ```bash
      systemctl status cheqd-noded
      ```

## Post-installation checks

Once the `cheqd-noded` daemon is active and running, check that the node is connected to the cheqd testnet and catching up with the latest updates on the ledger.

### Checking node status via terminal

```bash
cheqd-noded status
```

In the output, look for the text `latest_block_height` and note the value. Execute the status command above a few times and make sure the value of `latest_block_height` has increased each time.

The node is fully caught up when the parameter `catching_up` returns the output `false`.

### Checking node status via the RPC endpoint

An alternative method to check a node's status is via the RPC interface, if it has been configured.

* Remotely via the RPC interface: `cheqd-noded status --node <rpc-address>`
* By opening the JSONRPC over HTTP status page through a web browser: `<node-address:rpc-port>/status`

## Next steps

At this stage, your node would be connected to the cheqd testnet as an observer node. Learn [how to configure your node as a validator node](../validator-guide/README.md) to participate in staking rewards, block creation, and governance.


## Further information

After successful installation, learn [how to configure your node as a validator node](../validator-guide/README.md) to participate in staking rewards, block creation, and governance.
