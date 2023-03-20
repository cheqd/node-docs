# Requirements for setting up a new node

## Context

This document describes the hardware and software pre-requisites for setting up a new `cheqd-node` instance and joining the existing testnet/mainnet. The **recommended** installation method is to use the [interactive installer script](README.md).

Alternative installation methods and a developer guide to building from scratch are covered at the end of this page.

## Hardware requirements

For most nodes, the RAM/vCPU requirements are relatively static and do not change over time. However, the disk storage space needs to grow as the chain grows and will evolve over time.

It is recommended to **mount disk storage for blockchain data as an expandable volume/partition separate from your root partition**. This allows you mount the node data/configuration path on `/home` (for example) and increase the storage if necessary *independent* of the root `/` partition since hosting providers typically force an increase in CPU/RAM specifications to grow the root partition.

Extended information on [recommended hardware requirements is available in Tendermint documentation](https://docs.tendermint.com/main/tendermint-core/running-in-production.html#hardware). The figures below have been updated from the default Tendermint recommendations to account for current cheqd network chain size, real-world usage accounting for requests nodes need to handle, etc.

### Recommended hardware specifications

* 4 GB RAM (2 GB RAM minimum)
* x64 2.0 GHz 2 vCPU or equivalent (x64 1.4 GHz 1 vCPU or equivalent minimum)
* 650 GB SSD (500 GB minimum)

> ⚠️ **Storage requirements for the blockchain grows with time**. Therefore, these minimum storage figures are expected to increase over time. Read our validator guide for "pruning" settings to optimise storage consumed.

## Storage volumes

We recommend using a storage path that can be kept persistent and restored/remounted (if necessary) for the configuration, data, and log directories associated with a node. This allows a node to be restored along with configuration files such as node keys and for the node's copy of the ledger to be restored without triggering a full chain sync.

The default directory location for `cheqd-node` installations is `$HOME/.cheqdnode`, which computes to `/home/cheqd/.cheqdnode` when [using the interactive installer](interactive/interactive-installer.md). Custom paths can be defined if desired.

## Operating system

Our [packaged releases](https://github.com/cheqd/cheqd-node/releases) are currently compiled and tested for `Ubuntu 20.04 LTS`, which is the recommended operating system for installation using interactive installer or binaries.

For other operating systems, we recommend using [pre-built Docker image releases for `cheqd-node`](https://github.com/orgs/cheqd/packages?repo_name=cheqd-node).

We plan on supporting other operating systems in the future, based on demand for specific platforms by the community.

## Networking configuration

### Ports

To function properly, `cheqd-node` requires two types of ports to be configured. Depending on the setup, you may also need to configure firewall rules to allow the correct ingress/egress traffic.

Node operators should ensure there are no existing services running on these ports before proceeding with installation.

### P2P port

The P2P port is used for peer-to-peer communication between nodes. This port is used for your node to discover and connect to other nodes on the network. It should allow traffic to/from any IP address range.

* By default, the P2P port is set to `26656`.
* Inbound TCP connections on port `26656` (or your custom port) should be allowed from *any* IP address range.
* Outbound TCP connections must be allowed on *all* ports to *any* IP address range.
* The default P2P port can be changed in `$HOME/.cheqdnode/config/config.toml`.

Further details on [how P2P settings work is defined in Tendermint documentation](https://docs.tendermint.com/main/tendermint-core/running-in-production.html#p2p).

### RPC port

The RPC port is intended to be used by client applications as well as the cheqd-node CLI. Your RPC port **must** be active and available on localhost to be able to use the CLI. It is up to a node operator whether they want to expose the RPC port to public internet.

The [RPC endpoints for a node](https://docs.tendermint.com/main/rpc/) provide REST, JSONRPC over HTTP, and JSONRPC over WebSockets. These API endpoints can provide useful information for node operators, such as healthchecks, network information, validator information etc.

* By default, the RPC port is set to `26657`
* Inbound and outbound TCP connections should be allowed from destinations desired by the node operator. The default is to allow this from any IPv4 address range.
* TLS for the RPC port can also be setup separately. Currently, TLS setup is not automatically carried out in the install process described below.
* The default RPC port can be changed in `$HOME/.cheqdnode/config/config.toml`.

### Firewall configuration

In addition to the P2P/RPC ports above, you need to allow the following ports in terms of firewall access for the node to function correctly:

1. **Domain Name System (DNS)**: Allow *outbound* traffic on **TCP & UDP port 53** which allows your node to make DNS queries. Without this, your node will fail to make DNS lookups necessary to reach the peer-to-peer traffic ports for other nodes.
2. **Network Time Protocol (NTP)**: Allow outbound traffic on **UDP port 123**, which allows the NTP service to keep your system clock synchronised. Without this, [if your node's system clock drifts too far away from actual time in your node's timezone](https://www.digitalocean.com/community/tutorials/how-to-configure-ntp-for-use-in-the-ntp-pool-project-on-ubuntu-16-04#troubleshooting-connectivity-issues), it can reject peer-to-peer connectivity.

### Sentry nodes (optional)

Tendermint allows more complex setups in production, where the ingress/egress to a validator node is [proxied behind a "sentry" node](https://docs.tendermint.com/main/tendermint-core/validators.html).

While this setup is not compulsory, node operators with higher stakes or a need to have more robust network security may consider setting up a sentry-validator node architecture.

## Alternative installation methods

The [interactive installer](README.md) is designed to setup/configure node installation as a service that runs on a virtual machine. **This is the recommended setup for most scenarios when running as a validator**. A validator node is expected to run 24/7 for network stability and security, and therefore cannot be autoscaled up/down across multiple instances.

If you're not running a validator node, or if you want more advanced control on your setup, the following installation methods are also supported:

* [**Docker image**](docker.md): Docker images can be useful for running non-validator nodes on existing testnet/mainnet. They are also useful for running a localnet or when running a node on non-Linux systems for development purposes.
* [**Manual, binary-only install**](manual.md): Manual/binary-only installs are recommended if you want more control on system service setup than the installer allows (described below), or if you want a binary for development purposes.

## Further information

* Tendermint documentation has [best practices for running a Cosmos node in production](https://docs.tendermint.com/main/tendermint-core/running-in-production.html).
* [Сosmovisor could be used for automatic upgrades](https://docs.cosmos.network/main/tooling/cosmovisor); however in our testing so far this method has not been reliable and is therefore currently not recommended.
