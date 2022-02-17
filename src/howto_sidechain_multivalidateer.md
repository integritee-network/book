# How to run a sidechain multi-validateer setup

*This demo is intended to be an acceptance test for M8.3. It showcases a pair of validateers synchronizing their state using the sidechain as well as the on-boarding process according to our [scalability roadmap](https://polkadot.polkassembly.io/post/111).*

This demo sets up an Integritee node and two validateers (i.e. Integritee workers that produce sidechain blocks). The second validateer will be started 1 minute after the first one, so it needs to catch up on what has happened in the meantime on the first one. This process is called 'on-boarding'.

After this setup is fully up and running, we run a demo script that transfers funds from an Alice account to a Bob account. We do that by sending a direct request (or direct transaction) to each validateer in sequence. Because the validateers synchronize their state using the sidechain, the transactions will be known to both validateers. As a result they will have a consistent and identical view of the state.

## Setup

First build the Integritee node and the worker (validateer) in a docker container and software mode. This way no Intel SGX hardware is required.

Run on a Linux console (or WSL 2 with docker integration enabled):

```bash
# get the docker image
# check for updates on https://hub.docker.com/r/integritee/integritee-dev/tags
docker pull integritee/integritee-dev:0.1.7

# create a dedicated demo directory
mkdir demo && cd demo
# start the docker container (with sgx support)
# maps the current directory (demo) into the docker container and runs a bash shell
docker run -it -v $(pwd):/root/work integritee/integritee-dev:0.1.7 /bin/bash

# now inside the docker container
cd work

# clone and build the worker and the client
git clone https://github.com/integritee-network/worker.git
cd worker
# Install the correct rust-toolchain
rustup show
SGX_MODE=SW make
# this might take 10min+ on a fast machine

# clone and build the node
cd ..
git clone https://github.com/integritee-network/integritee-node.git
cd integritee-node
# Install the correct rust-toolchain
rustup show
# build the node
cargo build --release --features skip-ias-check,skip-extrinsic-filtering
# another 10min
```

For a nicer overview of the demo, let's use tmux and split our docker console into multiple terminals

```bash
tmux
tmux split-window -v
tmux split-window -h
```