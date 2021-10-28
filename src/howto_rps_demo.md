
# How to Play Rock Paper Scissors onchain

*this demo is intended to show a rock paper scissors game on the integriTEE sidechain*

## Setup

Build worker, client and node in our docker:

```bash
# get the docker image
# check for updates on https://hub.docker.com/repository/docker/scssubstratee/substratee_dev
docker pull scssubstratee/substratee_dev:1804-2.12-1.1.3-001

# create a dedicated demo directory and start the docker container (with sgx support)
mkdir demo && cd demo

# clone and build the node
git clone https://github.com/integritee-network/integritee-node.git
cd integritee-node
# build the node
cargo build --release --features skip-ias-check
# this might take 10min+ on a fast machine
cd ..

# run docker for sgx
docker run -it -v $(pwd):/root/work scssubstratee/substratee_dev:1804-2.12-1.1.3-001 /bin/bash
cd work

# clone and build the worker and the client
git clone https://github.com/integritee-network/worker.git
cd worker
git fetch origin rps-demo
git checkout rps-demo
./ci/install_rust.sh
SGX_MODE=SW make
# another 10min
```

For a nicer overview of the demo, let's use tmux and split our docker console into multiple terminals

```bash
tmux
tmux split-window -v
tmux split-window -h
```

You should now see three terminals

## Launch node in terminal 1

```bash
cd ~/work/integritee-node/
./target/release/integritee-node --tmp --dev
```

wait until you see blocks being produced...

## Launch worker in terminal 2

use `Ctrl-B + cursors` to move between terminals

```bash
cd ~/work/worker/bin
# create empty INTEL key files
touch spid.txt key.txt
./integritee-service init-shard
./integritee-service shielding-key
./integritee-service signing-key
./integritee-service mrenclave > ~/mrenclave.b58
./integritee-service run --skip-ra
```
wait until you see blocks being synched

## Run client in terminal 3

```bash
cd ~/work/worker/cli
./demo_rps.sh -m file
```

And now you should see Alice and Bob playing rock paper scissor quite rapidely (faster than the blocks are finalized on the substrate node)

## Cleanup
The files created in the docker container belong to `root`. This can make it impossible to delete them on your host system. We now give them back to your standard user. (Alternatively, you can just delete everything in `work`)

Note: This step is optional.

```bash
cd /root/work
ls -la

# write down the numbers on the line containing '.'
# example output: drwxrwxr-x 17 1002 1002   4096 Nov  2 15:10 .
#  where the numbers are 1002 (NUMBER1) and 1002 (NUMBER2)

# give all files back to the external user
chown -R <NUMBER1>:<NUMBER2> integritee-service integritee-node
```
