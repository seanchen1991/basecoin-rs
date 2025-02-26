# CI-related scripts

This folder contains scripts and configuration relating to integration testing
for basecoin-rs (which could also be modified to act as an integration test for
ibc-rs).

## Docker image

[`Dockerfile`](./Dockerfile) contains the primary script for building a Docker
image to test basecoin-rs.

### Building

From the root of this repository:

```bash
docker build -f ci/Dockerfile -t informaldev/basecoin-rs-ci ./ci
```

### Running

Running the image with appropriate parameters will allow you to test a build of
basecoin-rs with a particular build of ibc-rs (specifically the relayer,
Hermes). At present, by default, this executes the
[`create-connection`](./tests/create-connection.sh) script as a test if no `CMD`
is supplied to the image when running it.

From the root of this repository:

```bash
# Build basecoin-rs (located at `pwd`) and a local version of ibc-rs located at
# `/path/to/local/ibc-rs`.
docker run --rm -it \
    -v `pwd`:/src/basecoin-rs \
    -v /path/to/local/ibc-rs/:/src/ibc-rs \
    informaldev/basecoin-rs-ci

# If no local ibc-rs source volume is mounted, the image will automatically pull
# the latest ibc-rs code on master from the ibc-rs repository on GitHub.
docker run --rm -it \
    -v `pwd`:/src/basecoin-rs \
    informaldev/basecoin-rs-ci

# Specify the branch/tag/commit at which to clone the ibc-rs repository from
# which to build Hermes. In this case, we use branch "basecoin/phase-4-1":
docker run --rm -it \
    -v `pwd`:/src/basecoin-rs \
    -e IBC_COMMITISH=basecoin/phase-4-1 \
    informaldev/basecoin-rs-ci

# If you don't want to execute the tests, and rather want a BASH prompt from
# which you can manually interact with the various running binaries.
docker run --rm -it \
    -v `pwd`:/src/basecoin-rs \
    informaldev/basecoin-rs-ci \
    /bin/bash

```

### What does this image do?

For even more detail, see [`entrypoint.sh`](./entrypoint.sh). In sequence, a
container run from this image will:

1. Clone the [ibc-rs repository][ibc-rs-repo] if no ibc-rs sources have been
   mounted into the container.
2. Build the Hermes binary from the ibc-rs sources in the container.
3. Build the basecoin-rs binary from the basecoin-rs source volume mounted into
   the container.
4. Set up a single [Gaia] instance to act as a foreign chain for interacting
   with basecoin-rs. The ID of this chain will be `ibc-0`.
5. Configure Hermes.
6. Start a [Tendermint] node and the basecoin-rs binary (the Tendermint node
   will automatically connect to the basecoin-rs binary, providing a chain with
   ID `basecoin-0`).
7. If no `CMD` arguments are provided for the container, it will automatically
   execute the [`create-connection.sh`](./tests/create-connection.sh) script,
   which creates and updates an IBC channel between `basecoin-0` and `ibc-0`. If
   `CMD` arguments are provided for the container, that test will not be
   executed and the relevant `CMD` arguments will be executed instead.

[ibc-rs-repo]: https://github.com/informalsystems/ibc-rs
[Gaia]: https://github.com/cosmos/gaia
[Tendermint]: https://github.com/tendermint/tendermint
