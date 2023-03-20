# Building your own Docker image

## Context

> ℹ️ We provide [installation instructions using pre-built Docker images](../setup-and-configure/docker.md) if you just want to setup and use a Docker-based node.

These advanced instructions are intended for developers who want to build their own custom Docker image. You can also [build a binary using Golang](README.md), or [run a Docker-based localnet](docker-localnet.md).

## Pre-requisites

[Install Docker](https://docs.docker.com/engine/install/) pre-requisites below, either as individual installs or using [Docker Desktop](https://docs.docker.com/desktop/) (if running on a developer machine):

1. Docker Engine v20.10.x and above (use `docker -v` to check)
2. Docker Compose v2.3.x and above (use `docker compose version` to check)

Our Docker Compose files [use Compose v2 syntax](https://docs.docker.com/compose/compose-v2/). The primary difference in usage is that Docker Compose's new implementation uses `docker compose` commands (with a space), rather than the legacy `docker-compose` although they are supposed to be drop-in replacements for each other.

### Special guidance for Mac OS running on Apple silicon (M-series chips)

Most issues with Docker that get raised with us are typically with [developers running Mac OS with Apple M-series chips, which Docker has special guidance](https://docs.docker.com/desktop/install/mac-install/) for.

Other issues are due to developers [using the legacy `docker-compose` CLI rather than the new `docker compose` CLI](https://stackoverflow.com/q/66514436/314088). If your issues are specifically with Docker Compose, make sure the command used is `docker compose` (with a space).

## Instructions

### Clone the repository

Clone the [`cheqd-node` repository](https://github.com/cheqd/cheqd-node) from Github. (Github has [instructions on how to clone a repo](https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository).)

### (Optional) Inspect and modify Dockerfile

Inspect the [Dockerfile](https://github.com/cheqd/cheqd-node/blob/main/docker/Dockerfile) to understand build arguments and variables. This is only *really* necessary if you want to modify the Docker build.

Or, If you want to use [Docker `buildx` engine](https://docs.docker.com/engine/reference/commandline/buildx/), look at the usage/configuration in [our Github build workflow](https://github.com/cheqd/cheqd-node/blob/main/.github/workflows/build.yml).

> **Note**: If you're building on a Mac OS system with Apple M-series chips, you should modify the `FROM` statement in the Dockerfile to `FROM --platform=linux/amd64 golang:1.18-alpine AS builder`. Otherwise, Docker will try to download the Mac OS `darwin` image for the base Golang image and fail during the build process.

### Build the image

#### Using Docker Compose

If you're planning on passing/modifying a lot of build arguments from their defaults, you can [modify the Docker Compose file](https://github.com/cheqd/cheqd-node/tree/main/docker/localnet) and the associated environment files to define the build/run-time variables in a one place. This is the recommended method.

Note that a valid Docker Compose file will only have *one* `build` and `image` section, so modify/comment this as necessary. See our [instructions for how to use Docker Compose for mainnet/testnet](../setup-and-configure/docker.md) to understand how this works.

Sample command (modify as necessary):

```bahs
docker compose -f docker/localnet/docker-compose.yml build
```

#### Using Docker

If you don't want to use `docker compose build`, or build using `docker build` and then run it using Docker Compose, a sample command you could use is (modify as necessary)

```bash
docker build . -f docker/Dockerfile --target runner --tag cheqd-node:build-local`
```
