
# How to Perform Direct Transactions

*this demo is intended to be an acceptance test for M8.1 and M8.2. The underlying features are not production-ready. M8.1 and M8.2 are just our first milestones towards direct invocation and sidechains according to our [scalability roadmap](https://polkadot.polkassembly.io/post/111)*

This direct invocation demo is similar to [Private Transaction](./howto_private_tx.md): We demonstrate a change of ownership of tokens where no one but the involved parties can learn who sent how many tokens to whom. The difference lies in the way the transaction is transferred. With indirect invocation, calls are sent to the blockchain to get consensus over tx order. With direct invocation (this demo), the client call is directly sent to the TEE which takes care of tx ordering as a trusted entity.

For direct invocation, our worker exposes a rpc interface for submitting and watching a `TrustedCall`. Our client uses direct invocation whenever the `--direct` flag is present.

Substrate blockchains are usually account-based and pseudonymous by nature: The whole world can see all transactions and their source, destination and amount. Integritee offers confidentiality in a similar way that Zcash does: it offers users a way to move their tokens into a dark pool (shielding process) where they can transact privately and later retrieve tokens on their public accounts (unshielding). In the case of Zcash, privacy is guaranteed by zk-SNARKS, in Integritee it is guaranteed by TEEs.

The detailed design of the shielding and unshielding process is described in the chapter [Token Shielding](./token_shielding.md).

In the following demo we show how Alice can send tokens to Bob privately with a direct invocation call. The demo will run in our docker container so you don't need any special hardware. 

## Setup

You'll need SGX enabled hardware and to [register with Intel](./howto_worker.md#intel-sgx-development-and-production-commercial-license) and obtain your own KEY.

Build worker, client and node in our docker:

```bash
# get the docker image
# check for updates on https://hub.docker.com/r/integritee/integritee-dev/tags
docker pull integritee/integritee-dev:0.1.7

# create a dedicated demo directory and start the docker container (with sgx support)
mkdir demo && cd demo
docker run -it -v $(pwd):/root/work integritee/integritee-dev:0.1.7 /bin/bash
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
cargo build --release
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
./demo_direct_call.sh -m file
```

Now you can watch the process of

1. Sudo prefunding Alice
2. creating a new *incognito* account
3. Alice privately sending funds to new *incognito* account

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
