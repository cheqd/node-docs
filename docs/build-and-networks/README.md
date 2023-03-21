# Building your own node software binaries

## Building from source

Prerequisites:

* Install [Go](https://golang.org/doc/install) (currently, our builds are done for Golang v1.18)

To build the executable, run:

```bash
make proto-gen
make swagger
make build
```

## Build using Docker

We have an [in-depth guide for making custom Docker image builds](docker-build.md).

## Running a localnet using Docker Compose

We also have an [in-depth guide on running a localnet with multiple nodes](docker-localnet.md) using Docker / Docker Compose.
