# cheqd-testnet docker image

## Description

This is a Debian-based docker image. `cheqd-noded` is executable within this docker image which has been preconfigured to run a network of 2 nodes. It is intended for use in CI pipelines.

## Prerequisites

* Build `cheqd-node` image first. See the [instructions](../setup-and-configure/docker-install.md) here.

## Building

To build the image:

* Go to the repository root
* Run `docker build -f tests/networks/docker-localnet/Dockerfile -t cheqd-testnet .`

## Running

* Run `docker run -it --rm -p "26657:26657" -p "1317:1317" cheqd-testnet`
* RPC apis are exposed on the folowing ports:
  * node\_0: `26657`
* Try to connect to any node in your browser, for instance: `http://localhost:26657/`

