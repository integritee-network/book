# How to Perform Private Transactions

Private transactions are a change of ownership of tokens where no one but the involved parties can learn who sent how many tokens to whom.

Substrate blockchains are usually account-based and pseudonymous by nature: The whole world can see all transactions and their source, destination and amount. SubstraTEE offers confidentiality in a similar way that Zcash does: it offers users a way to move their tokens into a dark pool (shielding process) where they can transact privately and later retrieve tokens on their public accounts (unshielding). In the case of Zcash, privacy is guaranteed by zk-SNARKS, in SubstraTEE it is guaranteed by TEEs.

The detailed design of the shielding and unshielding process is described in the chapter [Token Shielding](./token_shielding.md).

In the following demo we show how Alice can send tokens to Bob privately. The demo will run in our docker container so you don't need SGX HW to run the demo.

## Setup

You'll need to [register with Intel](./howto_worker.md#intel-sgx-development-and-production-commercial-license) and obtain your own SPID and KEY.

Build node, worker and client in our docker:

```bash
docker pull scssubstratee/substratee_dev:18.04-2.9.1-1.1.2
mkdir demo && cd demo
docker run -it -v $(pwd):/root/work scssubstratee/substratee_dev:18.04-2.9.1-1.1.2 /bin/bash
# now you are inside the container
cd work
git clone https://github.com/scs/substraTEE-worker.git
cd substraTEE-worker
git checkout M6
SGX_MODE=SW make
# this might take 10min+ on a fast machine

echo "<YOUR SPID>" > bin/spid.txt
echo "<YOUR KEY>" > bin/key.txt

cd ..
git clone https://github.com/scs/substraTEE-node.git
cd substraTEE-node
git checkout M6
cargo build --release
# another 10min
```

For a nicer overview of the demo, let's install tmux and split our docker console into multiple terminals

```bash
apt update
apt install tmux
tmux
tmux split-window -v
tmux split-window -h
```

You should now see three terminals

## launch node in terminal 1

```bash
cd ~/work/substraTEE-node/
./target/release/substratee-node --dev -lruntime=debug
```

blocks should be produced...

## launch worker in terminal 2

use `Ctrl-B + cursors` to move between terminals

```bash
cd ~/work/substraTEE-worker/bin
./substratee-worker init-shard
./substratee-worker shielding-key
./substratee-worker signing-key
./substratee-worker run
```

## play in terminal 3

```bash
cd ~/work/substraTEE-worker/bin
./demo_shielding_unshielding.sh
```

Now you can watch the process of

1. Alice creating a new *incognito* account. This account is never disclosed to the public.
2. Alice shielding funds onto her *incognito* account
3. Alice privately sending funds to Bobs *incognito* account
4. Alice unshielding some funds back onto her public account
