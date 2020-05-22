# How to Build and Run a SubstraTEE-node

You don't need SGX to run a node (only workers do).

## Build

In order to compile *ring* into wasm, you'll need LLVM-9 or above or you'll get linker errors. Here the instructions for ubuntu 18.04

```bash
wget https://apt.llvm.org/llvm.sh
chmod +x llvm.sh
sudo ./llvm.sh 10
export CC=/usr/bin/clang-10
export AR=/usr/bin/llvm-ar-10
cargo clean
```

Install Rust:

```bash
curl https://sh.rustup.rs -sSf | sh
```

Initialize your Wasm Build environment:

```bash
./scripts/init.sh
```

Build Wasm and native code:

```bash
cargo build --release
```

### with docker 

If you prefer to skip the setup, you can just use our docker and build the node like this (run in the repo root):

```
docker pull scssubstratee/substratee_dev:18.04-2.9.1-1.1.2
docker run -it -v $(pwd):/root/work scssubstratee/substratee_dev:18.04-2.9.1-1.1.2 /bin/bash
cargo build --release
```

## Run

### Single Node Development Chain

Purge any existing developer chain state:

```bash
./target/release/substratee-node purge-chain --dev
```

Start a development chain with:

```bash
./target/release/substratee-node --dev
```

Detailed logs may be shown by running the node with the following environment variables set: `RUST_LOG=debug RUST_BACKTRACE=1 cargo run -- --dev`.




