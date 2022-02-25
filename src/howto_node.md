# How to Build and Run a integritee-node

You don't need SGX to run a node (only workers do).

## Build

Install Rust:
```bash
curl https://sh.rustup.rs -sSf | sh
```

In order to compile *ring* into wasm, you'll need LLVM-9 or above or you'll get linker errors. Here the instructions for Ubuntu 18.04

```bash
wget https://apt.llvm.org/llvm.sh
chmod +x llvm.sh
sudo ./llvm.sh 10
export CC=/usr/bin/clang-10
export AR=/usr/bin/llvm-ar-10
# if you already built, make sure to run cargo clean
```

Initialize your Wasm Build environment:
```bash
rustup show
```

Build Wasm and native code:
```bash
cargo build --release
```

### with docker

If you prefer to skip the setup, you can just use our docker and build the node like this (run in the repo root):

```bash
docker pull integritee/integritee-dev:0.1.8
docker run -it -v $(pwd):/root/work integritee/integritee-dev:0.1.8 /bin/bash
rustup show
cargo build --release
```

## Run

### Single Node Development Chain

Purge any existing developer chain state:

```bash
./target/release/integritee-node purge-chain --dev
```

Start a development chain with:

```bash
./target/release/integritee-node --dev
```

If you want the integritee-node to expose a different websocket port, use the option `--ws-port xxx`. If external workers or clients need to access, add the option `--ws-external`.

Detailed logs may be shown by running the node with the following environment variables set: `RUST_LOG=debug RUST_BACKTRACE=1 cargo run -- --dev`.

### Node as a System Service
If you want to run your node as a system service in Linux, create (as root or user with sudo permissions) a file in `/etc/systemd/system` called `integritee-node.service` with the following content:
```bash
[Unit]
Description=Integritee Node
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=always
RestartSec=10
User=<USER TO RUN THE NODE>
ExecStart=<ABSOLUTE PATH TO>/integritee-node --chain <PATH TO CHAIN JSON> --name <NAME OF THE NODE>

[Install]
WantedBy=multi-user.target
```
Where:
* `StartLimitIntervalSec` makes that systemd will try to re-start the service forever.
* `RestartSec` indicates the interval between two restarts.
* `User` specifies the user that should run the node.
* `ExecStart` specifies the path to the binary. Use absolute paths here.

Update the systemd daemon with `systemctrl daemon-reload`.

Use the following commands:
* `systemctrl start integritee-node.service` to start the node.
* `systemctrl stop integritee-node.service` to stop the node.
* `systemctrl status integritee-node.service` to check the status of the node/service.
