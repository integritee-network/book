# How to Perform Private Transactions

Private transactions are a change of ownership of tokens where no one but the involved parties can learn who sent how many tokens to whom.

Substrate blockchains are usually account-based and pseudonymous by nature: The whole world can see all transactions and their source, destination and amount. substraTEE offers confidentiality in a similar way that Zcash does: it offers users a way to move their tokens into a dark pool (shielding process) where they can transact privately and later retrieve tokens on their public accounts (unshielding). In the case of Zcash, privacy is guaranteed by zk-SNARKS, in substraTEE it is guaranteed by TEEs.

The detailed design of the shielding and unshielding process is described in the chapter [Token Shielding](./token_shielding.md).

In the following demo we show how Alice can send tokens to Bob privately. The demo will run in our docker container so you don't need to setup a complete SGX development machine (although the Intel SGX driver and SDK needs to be installed making `/dev/isgx` and the aesm service available).

## Setup

You'll need SGX enabled hardware and to [register with Intel](./howto_worker.md#intel-sgx-development-and-production-commercial-license) and obtain your own SPID and KEY.

Build worker, client and node in our docker:

```bash
# get the docker image
# check for updates on https://hub.docker.com/repository/docker/scssubstratee/substratee_dev
docker pull scssubstratee/substratee_dev:18.04-2.9.1-1.1.2

# create a dedicated demo directory and start the docker container
mkdir demo && cd demo
docker run -it -v $(pwd):/root/work -v /var/run/aesm:/var/run/aesm --device /dev/isgx scssubstratee/substratee_dev:18.04-2.9.1-1.1.2 /bin/bash

# now you are inside the container
# clone and build the worker and the client
# info: change the tag to the latest
cd work
git clone https://github.com/scs/substraTEE-worker.git
cd substraTEE-worker
git checkout v0.6.12-sub2.0.0
make
# this might take 10min+ on a fast machine
```

In case you get the error:
```bash
error: failed to get `sgx-runtime` as a dependency of package `substratee-stf v0.6.12-sub2.0.0 (/root/work/substraTEE-worker/stf)`
Caused by:
  failed to load source for dependency `sgx-runtime`
```
you need to edit the file Cargo.lock manually. Within the file, find the inclusion of the package sgx-runtime. It should look similiar to:
```rust
[[package]]
name = "sgx-runtime"
version = "0.6.12-sub2.0.0"
source = "git+https://github.com/scs/sgx-runtime?tag=v0.6.12-sub2.0.0#daace7e56a250e79132962311ac0e7935faa8385"
dependencies = [
*--snip
]
```
Delete the last bit after (and with) the # tag of the source:
```rust
source = "git+https://github.com/scs/sgx-runtime?tag=v0.6.12-sub2.0.0"
```
Save your changes. If you can not save the changes due to "permission denied", follow the steps described in the Cleanup section below. Since the directory substraTEE-node has not yet been cloned only change the permission of substraTEE-worker. Saving should work after the cleanup.

Re-enter the substraTEE-worker directory and try to run the make command again:
```bash
cd substraTEE-worker
make
# this might take 10min+ on a fast machine
```

The rest of the setup should now work without further errors:

```bash
# use your SPID and KEY from Intel
echo "<YOUR SPID>" > bin/spid.txt
echo "<YOUR KEY>" > bin/key.txt

# clone and build the node
# info: change the tag to the latest
cd ..
git clone https://github.com/scs/substraTEE-node.git
cd substraTEE-node
git checkout v0.6.12-sub2.0.0
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
cd ~/work/substraTEE-node/
./target/release/substratee-node --dev -lruntime=debug
```

blocks should be produced...

## Launch worker in terminal 2

use `Ctrl-B + cursors` to move between terminals

```bash
cd ~/work/substraTEE-worker/bin
./substratee-worker init-shard
./substratee-worker shielding-key
./substratee-worker signing-key
./substratee-worker run
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
chown -R <NUMBER1>:<NUMBER2> substraTEE-worker substraTEE-node
```

## Troubleshooting

When launching the worker from within the docker environment, the following error may occurr:
```bash
thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: SGX_ERROR_NO_DEVICE', worker/src/main.rs:180:31
```
To workaround this problem enter:
```bash
LD_LIBRARY_PATH=/opt/intel/sgx-aesm-service/aesm/ /opt/intel/sgx-aesm-service/aesm/aesm_service & 
./substratee-worker init-shard
```
