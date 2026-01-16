# Setup and configuration with Docker

## Context

We provide [a pre-built `cheqd-node` Docker image](https://github.com/cheqd/cheqd-node/pkgs/container/cheqd-node) for installation by those who want to run on Docker-based systems.

Docker-based installations are useful when running non-validator (observer) nodes that can be auto-scaled according to demand, or if you're a developer who setup a localnet / access node CLI without running a node.

> ⚠️ It is **NOT recommended** to run a validator node using Docker since you need to be **absolutely** certain about not running two Docker containers as validator nodes simultaneously. If two Docker containers with the same validator keys are active at the same time, this could be perceived by the network as a validator compromise / double-signing infraction and result in jailing / slashing.

## Pre-requisites

[Install Docker](https://docs.docker.com/engine/install/) pre-requisites below, either as individual installs or using [Docker Desktop](https://docs.docker.com/desktop/) (if running on a developer machine):

1. Docker Engine v20.10.x and above (use `docker -v` to check)
2. Docker Compose v2.3.x and above (use `docker compose version` to check)

Our Docker Compose files [use Compose v2 syntax](https://docs.docker.com/reference/compose-file/). The primary difference in usage is that Docker Compose's new implementation uses `docker compose` commands (with a space), rather than the legacy `docker-compose` although they are supposed to be drop-in replacements for each other.

### Special guidance for Mac OS running on Apple silicon (M-series chips)

Most issues with Docker that get raised with us are typically with [developers running Mac OS with Apple M-series chips, which Docker has special guidance](https://docs.docker.com/desktop/install/mac-install/) for.

Other issues are due to developers [using the legacy `docker-compose` CLI rather than the new `docker compose` CLI](https://stackoverflow.com/q/66514436/314088). If your issues are specifically with Docker Compose, make sure the command used is `docker compose` (with a space).

## Usage

### Fetch the latest stable Docker image

Pull a [pre-built `cheqd-node` Docker image from Github Container Registry](https://github.com/cheqd/cheqd-node/pkgs/container/cheqd-node) (replace `latest` with a different version tag if you want to pull something other than the latest version):

```bash
docker pull ghcr.io/cheqd/cheqd-node:latest
```

### Modify environment variables used by Docker Compose

We provide [a simple Docker Compose file to bring a node up with the required configuration](https://github.com/cheqd/cheqd-node/tree/main/docker/persistent-chains). This is broken down into three files that need to be modified with the configuration parameters:

1. [`docker-compose.yml`](https://github.com/cheqd/cheqd-node/blob/main/docker/persistent-chains/docker-compose.yml): Docker Compose file
2. [`docker-compose.env`](https://github.com/cheqd/cheqd-node/blob/main/docker/persistent-chains/docker-compose.env): Environment variables used **in `docker-compose.yml`**
3. [`container.env`](https://github.com/cheqd/cheqd-node/blob/main/docker/persistent-chains/container.env): Environment variables used **inside the `cheqd-node` container**

Both of the `.env` files are signposted with the `REQUIRED` and `OPTIONAL` parameters that can be defined. You **must** fill out the required configuration parameters.

### Start Docker container

#### Using Docker Compose

Once the environment variable files are edited, bringing up a Docker container is as simple as:

```bash
docker compose -f docker/persistent-chains/docker-compose.yml --env-file docker/persistent-chains/docker-compose.env up --detach
```

**Note**: The file paths above for the `-f` and `--env-file` parameter are relative to the [root folder of the `cheqd-node` repository](https://github.com/cheqd/cheqd-node). Please modify the file paths for the correct relative/absolute paths on the system where you are executing the commands.

#### Using Docker

If you decide not to use the Docker Compose method, you'll need to configure node settings and volumes for the container manually.

Once you've configured these manually, start using

```bash
docker run ghcr.io/cheqd/cheqd-node:latest
```

Alternatively, if you want to just start with the bash terminal without actually starting a node, you could use:

```bash
docker run -it --entrypoint /bin/bash ghcr.io/cheqd/cheqd-node:latest
```

### Stop running container

To stop a detached container that was started using Docker Compose, use:

```bash
docker compose -f docker/persistent-chains/docker-compose.yml down
```

If you *also* want to remove the container volumes when stopping, add the `--volumes` flag to the command:

```bash
docker compose -f docker/persistent-chains/docker-compose.yml down --volumes
```

Be careful with removing volumes, since critical data such as node/validator keys will *also* be removed when volumes are removed. There's no way to get these back, unless you've backed them up independently.

## Advanced usage

We have additional guides for the following advanced usage scenarios:

* [Build your own image using Docker](../build-and-networks/docker-build.md) to create custom images
* [Run a Docker-based localnet](../build-and-networks/docker-localnet.md) with multiple nodes to simulate a network
