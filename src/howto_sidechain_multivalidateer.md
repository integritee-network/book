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
docker pull integritee/integritee-dev:0.1.9

# create a dedicated demo directory
mkdir demo && cd demo
# start the docker container (with sgx support)
# maps the current directory (demo) into the docker container and runs a bash shell
docker run -it -v $(pwd):/root/work integritee/integritee-dev:0.1.9 /bin/bash

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
tmux split-window -h
```

## Launch local setup in terminal 1

Prepare the worker by generating all the necessary keys and files used for startup:

```bash
cd ~/work/worker/bin
# create empty INTEL key files
touch spid.txt key.txt
./integritee-service mrenclave > ~/mrenclave.b58
```

Use the `local-setup` scripts to launch an integritee node and 2 workers. The workers are started with a 1 minute delay in between them.

```bash
cd ~/work/worker
./local-setup/launch.py ./local-setup/tutorial-config.json
```

Wait until you see the message "`Starting worker 2 in background`" and then wait another minute or so.  

## Launch sidechain demo script in terminal 2

Switch to terminal 2 (using `Ctrl-B + cursor right`) and run the demo script `sidechain.sh`.

```bash
cd ~/work/worker/scripts
source ./init_env.sh && ./sidechain.sh
```

You will see output from the demo script, transferring funds using both workers and in the end verifying that the balances of both accounts (Alice and Bob) match the expected result.

The `tmux` session can be ended using `Ctrl-B + : ` to enter command mode, and then `kill-session`.

## Cleanup (optional)
The files created in the docker container belong to `root`. This can make it hard to delete them from your host system. You can change ownership of those folders back to your regular user.

```bash
cd /root/work
ls -la

# write down the numbers on the line containing '.'
# example output: drwxrwxr-x 17 1002 1002   4096 Nov  2 15:10 .
#  where the numbers are 1002 (NUMBER1) and 1002 (NUMBER2)

# give ownership back to the external user
chown -R <NUMBER1>:<NUMBER2> integritee-service integritee-node
```