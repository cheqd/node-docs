# Installing cheqd-node with Docker

## Description

We provide [a pre-built `cheqd-node` Docker image](https://github.com/cheqd/cheqd-node/pkgs/container/cheqd-node) for installation by those who want to run on Docker-based systems.

In general, it is **NOT recommended** to run a validator node using Docker since you need to be **absolutely** certain about not running two Docker containers as validator nodes simultaneously. If two Docker containers with the same validator keys are active at the same time, this could be perceived by the network as a validator compromise / double-signing infraction and result in jailing / slashing.

Running non-validator nodes via Docker is less risky, as the validator keys don't matter here.

## Usage

Pull the pre-built `cheqd-node` Docker image at the required `version-tag` number:

```bash
docker pull ghcr.io/cheqd/cheqd-node:<version-tag>
```

We provide [a simple Docker Compose file to bring a node up with the required configuration](https://github.com/cheqd/cheqd-node/tree/main/docker/persistent-chains). This is broken down into three files that need to be modified with the configuration parameters:

1. [`docker-compose.yml`](https://github.com/cheqd/cheqd-node/blob/main/docker/persistent-chains/docker-compose.yml): Docker Compose file
2. [`docker-compose.env`](https://github.com/cheqd/cheqd-node/blob/main/docker/persistent-chains/docker-compose.env): Environment variables used **in `docker-compose.yml`**
3. [`container.env`](https://github.com/cheqd/cheqd-node/blob/main/docker/persistent-chains/container.env): Environment variables used **inside the `cheqd-node` container**

Both of the `.env` files are signposted with the `REQUIRED` and `OPTIONAL` parameters that can be defined. You **must** fill out the required configuration parameters.

Once the environment variable files are edited, bringing up a Docker container is as simple as:

```bash
docker-compose -f docker/persistent-chains/docker-compose.yml --env-file docker/persistent-chains/docker-compose.env up --no-build
```

**Note**: The file paths above for the `-f` and `--env-file` parameter are relative to the [root folder of the `cheqd-node` repository](https://github.com/cheqd/cheqd-node). Please modify the file paths for the correct relative/absolute paths on the system where you are executing the commands.

## Building

To build the image:

* Go to the repository root;
* Run `docker build --target node -t cheqd-node -f docker/Dockerfile .`.

Default home directory for `cheqd` user is `/cheqd`. It can be overridden via `CHEQD_HOME_DIR` build argument. Example: `--build-arg CHEQD_HOME_DIR=/home/cheqd`.

Note: If you are using M1 Macbook you should modify the `FROM` statement in the Dockerfile, should be like this `FROM --platform=linux/amd64 golang:buster as builder`
