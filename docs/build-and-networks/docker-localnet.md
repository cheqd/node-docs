# Docker Compose Based Localnet

## Description

The set of scripts to generate configurations for a network of four nodes and run it locally in docker-compose.

## Building your own Docker image

In case you don't want to use the pre-built Docker image published to Github Container registry, use the following steps:

1. **Clone the [`cheqd-node` repository](https://github.com/cheqd/cheqd-node)** from Github
2. (Optional) **Inspect the [Dockerfile](https://github.com/cheqd/cheqd-node/blob/main/docker/Dockerfile)** to understand build arguments and variables.
   1. This is only *really* necessary if you want to modify the Docker build.
   2. You can also [modify the Docker Compose file](https://github.com/cheqd/cheqd-node/blob/main/docker/persistent-chains/docker-compose.yml) to uncomment the `build` section. Note that a valid Docker Compose file will only have *one* `image` section, so modify/comment this as necessary.
   3. If you want to use [Docker `buildx` engine](https://docs.docker.com/engine/reference/commandline/buildx/), look at the usage/configuration in [our Github build workflow](https://github.com/cheqd/cheqd-node/blob/main/.github/workflows/build.yml).
3. **Build the image**: Again, you have two options here:
   1. Sample command **using `docker build`** (modify as necessary): `docker build . -f docker/Dockerfile --target runner --tag cheqd-node:build-local`.
   2. Sample command **using [`docker compose build`](https://docs.docker.com/engine/reference/commandline/compose_build/)** (modify as necessary): `docker compose -f docker/persistent-chains/docker-compose.yml build`

If you're planning on passing/modifying a lot of build arguments from their defaults, the Docker Compose method allows defining them easily in the `docker-compose.yml` file.

> **Note**: If you are using M1 Macbook you should modify the `FROM` statement in the Dockerfile, should be like this `FROM --platform=linux/amd64 FROM golang:1.18-alpine AS builder`

## Prerequisites

* [Starport](https://docs.starport.network/guide/install.html)
* docker-compose

## How to run

1. Build docker image:

    See [the instruction](../setup-and-configure/docker-install.md).

2. Build cheqd-node:

    ```bash
    starport chain build
    ```

3. Generate node configurations:

    Run: `gen_node_configs.sh`.

4. Run docker-compose:

    Run: `run_docker.sh`.

## Result

### Nodes

This will setup 4 nodes listening on the following ports:

* Node0:
  * p2p: 26656
  * rpc: 26657
* Node1:
  * p2p: 26659
  * rpc: 26660
* Node2:
  * p2p: 26662
  * rpc: 26663
* Node3:
  * p2p: 26665
  * rpc: 26666

You can tests connection to a node using browser: `http://localhost:<rpc_port>`. Example for the first node: `http://localhost:26657`.

### Accounts

Also, there will be 4 keys generated and corresponding genesis accounts created for node operators:

* operator0;
* operator1;
* operator2;
* operator3;

When connecting using CLI, point path to home directory of the node corresponding to the operator: `--home network-config/validator-x`.

## CLI commands

See [the cheqd CLI guide](../cheqd-cli/README.md) to learn about the most common CLI flows.
