# Running a localnet using Docker images

## Context

This document provides instructions on how to run a localnet with multiple validator/non-validator nodes. This can be useful if you are developing applications to work on cheqd network, or in automated testing pipelines.

The techniques described here are used in CI/CD contexts, for example, in this repository itself in the [`test.yml` Github workflow](https://github.com/cheqd/cheqd-node/blob/main/.github/workflows/test.yml).

## Pre-requisites

* A clone of the [`cheqd-node` repository](https://github.com/cheqd/cheqd-node)
* Either [a pre-built Docker image downloaded from Github Container Registry](https://github.com/cheqd/cheqd-node/pkgs/container/cheqd-node), or [a custom-built Docker image](docker-build.md) (the latter is mandatory if you've modified any code in the repository cline).
* Docker Engine and Docker Compose (same versions as described in [configuration instructions for Docker setup](../setup-and-configure/docker.md))
* A cheqd-node binary to run the network config generation script below.

## Instructions

Our localnet setup instructions are designed to set up a local network with the following node types:

* 3x validator nodes
* 1x non-validator/observer node
* 1x seed node

The definition for this network is described in a [localnet Docker Compose file](https://github.com/cheqd/cheqd-node/blob/main/docker/localnet/docker-compose.yml), which can be modified as required for your specific use case. Since it's not possible to cover all possible localnet setups, the following instructions describe the steps necessary to execute a setup similar to that using in [our Github `test` workflow](https://github.com/cheqd/cheqd-node/blob/main/.github/workflows/test.yml).

### Generate localnet configuration (one-time)

Execute the [bash script `gen-network-config.sh`](https://github.com/cheqd/cheqd-node/blob/main/docker/localnet/gen-network-config.sh) to generate validator keys and node configuration for the node types above.

```bash
./docker/localnet/gen-network-config.sh
```

You may modify the output if you want a different mix of node types.

#### Import the generated keys (one-time)

Import the keys generated using the [`import-keys.sh` bash script](https://github.com/cheqd/cheqd-node/blob/main/docker/localnet/import-keys.sh):

```bash
./docker/localnet/import-keys.sh
```

### Bring up the localnet using Docker Compose

Modify the `docker-compose.yml` file if necessary, along with the per-container environment variables under the `container-env` folder.

```bash
docker compose --env-file build-latest.env up --detach --no-build
```

## Interacting with localnet

The default Docker localnet will configure a running network with a pre-built image or custom image.

### Nodes

The five nodes and corresponding ports set up by the default Docker Compose setup will be:

1. **Validator nodes**
   1. `validator-0`
      1. *P2P*: 26656
      2. *RPC*: 26657
   2. `validator-1`
      1. *P2P*: 26756
      2. *RPC*: 26757
   3. `validator-2`
      1. *P2P*: 26856
      2. *RPC*: 26857
   4. `validator-3`
      1. *P2P*: 26956
      2. *RPC*: 26957
2. **Seed node**
   1. `seed-0`
      1. *P2P*: 27056
      2. *RPC*: 27057
3. **Observer node**
   1. `observer-0`
      1. *P2P*: 27156
      2. *RPC*: 27157

You can tests connection to a node using browser: `http://localhost:<rpc_port>`. Example for the first node: `http://localhost:26657`.

### Accounts

Key and corresponding accounts will be placed in the network config folder by the `import-keys.sh` script, which are used within the nodes configured above.

When connecting using CLI, provide the `--home` parameter to any CLI command to point to the specific home directory of the corresponding node: `--home network-config/validator-x`.

## CLI commands

See [the cheqd CLI guide](../cheqd-cli/README.md) to learn about the most common CLI commands.
