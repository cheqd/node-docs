# Building your own Docker image

## Context

> ℹ️ We provide [installation instructions using pre-built Docker images](../setup-and-configure/docker.md) if you just want to setup and use a Docker-based node.

These advanced instructions are intended for developers who want to build their own custom Docker image.

## Pre-requisites

[Install Docker](https://docs.docker.com/engine/install/) pre-requisites below, either as individual installs or using [Docker Desktop](https://docs.docker.com/desktop/) (if running on a developer machine):

1. Docker Engine v20.10.x and above (use `docker -v` to check)
2. Docker Compose v2.3.x and above (use `docker compose version` to check)

Our Docker Compose files [use Compose v2 syntax](https://docs.docker.com/compose/compose-v2/). The primary difference in usage is that Docker Compose's new implementation uses `docker compose` commands (with a space), rather than the legacy `docker-compose` although they are supposed to be drop-in replacements for each other.

### Special guidance for Mac OS running on Apple silicon (M-series chips)

Most issues with Docker that get raised with us are typically with [developers running Mac OS with Apple M-series chips, which Docker has special guidance](https://docs.docker.com/desktop/install/mac-install/) for.

Other issues are due to developers [using the legacy `docker-compose` CLI rather than the new `docker compose` CLI](https://stackoverflow.com/q/66514436/314088). If your issues are specifically with Docker Compose, make sure the command used is `docker compose` (with a space).

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