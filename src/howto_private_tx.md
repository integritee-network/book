# How to Perform Private Transactions

Private transactions are a change of ownership of tokens where no one but the involved parties can learn who sent how many tokens to whom.

Substrate blockchains are usually account-based and pseudonymous by nature: The whole world can see all transactions and their source, destination and amount. Integritee offers confidentiality in a similar way that Zcash does: it offers users a way to move their tokens into a dark pool (shielding process) where they can transact privately and later retrieve tokens on their public accounts (unshielding). In the case of Zcash, privacy is guaranteed by zk-SNARKS, in Integritee it is guaranteed by TEEs.

The detailed design of the shielding and unshielding process is described in the chapter [Token Shielding](./token_shielding.md).

In the following demo we show how Alice can send tokens to Bob privately. The demo will run in our docker container so you don't need to setup a complete SGX development machine (although the Intel SGX driver and SDK needs to be installed making `/dev/isgx` and the aesm service available).

## Setup

You'll need SGX enabled hardware and to [register with Intel](./howto_worker.md#intel-sgx-development-and-production-commercial-license) and obtain your own SPID and KEY.

Build worker, client and node in our docker:

```bash
# get the docker image
# check for updates on https://hub.docker.com/repository/docker/scssubstratee/substratee_dev
docker pull scssubstratee/substratee_dev:1804-2.12-1.1.3-001

# create a dedicated demo directory and start the docker container
mkdir demo && cd demo
docker run -it -v $(pwd):/root/work scssubstratee/substratee_dev:1804-2.12-1.1.3-001 /bin/bash

# now you are inside the container
# clone and build the worker and the client
cd work
git clone https://github.com/integritee-network/worker.git
cd worker
SGX_MODE=SW make
# this might take 10min+ on a fast machine

# clone and build the node
cd ..
git clone https://github.com/integritee-network/integritee-node.git
cd integritee-node
# initialize wasm build environment
./scripts/init.sh
# build the node
cargo build --release
# another 10min
```

For a nicer overview of the demo, let's install tmux and split our docker console into multiple terminals

```bash
apt update
apt install -y tmux
tmux
tmux split-window -v
tmux split-window -h
```

You should now see three terminals

## Launch node in terminal 1

```bash
cd ~/work/integritee-node/
./target/release/integritee-node --dev -lruntime=debug
```

blocks should be produced...

## Launch worker in terminal 2

use `Ctrl-B + cursors` to move between terminals

```bash
cd ~/work/worker/bin
# create empty INTEL key files
touch spid.txt key.txt
./substratee-worker init-shard
./substratee-worker shielding-key
./substratee-worker signing-key
./substratee-worker mrenclave > ~/mrenclave.b58
./substratee-worker run --skip-ra
```

## Play in terminal 3

```bash
cd ~/work/substraTEE-worker/client
./demo_shielding_unshielding.sh
```

Now you can watch the process of

1. Alice creating a new *incognito* account. This account is never disclosed to the public.
2. Alice shielding funds onto her *incognito* account
3. Alice privately sending funds to Bobs *incognito* account
4. Alice unshielding some funds back onto her public account

## Cleanup
The files created in the docker container belong to `root`. This makes it hard to delete them on your normal system. We now give them back to your standard user.

Note: This step is optional.

```bash
cd /root/work
ls -la

# write down the numbers on the line containing '.'
# example output: drwxrwxr-x 17 1002 1002   4096 Nov  2 15:10 .
#  where the numbers are 1002 (NUMBER1) and 1002 (NUMBER2)

# give all files back to the external user
chown -R <NUMBER1>:<NUMBER2> substraTEE-worker integritee-node
```
