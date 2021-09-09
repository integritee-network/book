# How To Run Your Own Worker

## HW requirements

While SGX is supported by most Intel CPU's that is not the case for all chipsets. Here we're just telling you what HW we are using and is known to work.

### Dell PowerEdge R340 Server

CPU **has to be** Intel(R) Xeon(R) E-2276G CPU @ 3.80 GHz

### Enable SGX support in BIOS

To enable SGX support in the Dell BIOS, enter the BIOS, go to `System Security` and set the following values:
* `Intel SGX` to `On`
* `SGX Launch Control Policy` to `Unlocked`

## Intel SGX development and production (commercial) license

In order to perform a remote attestation of the enclave, an [Intel SGX Attestation Enhanced Service Privacy ID (EPID)](https://api.portal.trustedservices.intel.com/EPID-attestation) is needed. We use unlinkable quotes in our code. Developers need to [register an account with Intel](https://api.portal.trustedservices.intel.com/EPID-attestation)

### Development Access

Copy your SPID and key to the following files (use Linux line endings):

* `bin/spid.txt`: SPID of your subscription
* `bin/key.txt`: Key of your subscription (primary or secondary works)

The enclave will be signed with the development key found under `enclave/Enclave_private.pem` and uses the configuration found under `enclave/Enclave.config.xml`.

### Production Access

You need a commercial license with Intel to run your enclaves in production mode (the only mode that really is confidential). Only legal entities can get a commercial license with Intel. Get in touch with them to obtain one.

Copy your SPID and key to the following files (use Linux line endings):

* `bin/spid_production.txt`: SPID of your subscription
* `bin/key_production.txt`: Key of your subscription (primary or secondary works)

These files are used to access the Intel Remote Attestation Service.

The enclave will be signed with the private key that was also registered and whitelisted at Intel's (in the process of obtaining a commercial license). Make sure that the key is exported as an environment variable called `SGX_COMMERCIAL_KEY`.

The enclave in production mode uses the configuration found under `enclave/Enclave.config.production.xml`.

The only difference is that the option `DisableDebug` is set to `1`.

## SW Requirements

You need the following components installed to start developing/compiling the code:

* [Intel SGX driver](https://github.com/intel/linux-sgx-driver), [SGX and PSW](https://github.com/intel/linux-sgx)
* [Rust](https://www.rust-lang.org/)
* [Rust SGX SDK](https://github.com/apache/incubator-teaclave-sgx-sdk)
* [IPFS](https://ipfs.io/)

### Setup SGX hardware with Ansible

You find a sample Ansible playbook [here](https://github.com/scs/intel_sgx_setup)

Open the playbook with your editor and replace all the variables with `<...>` with your own settings.

To execute the playbook and configure the remote machine, use the following command:

```bash
ansible-playbook site.yml -k
```

### Using Docker
We provide docker images with all the required tools installed. They can be found on [dockerhub](https://hub.docker.com/repository/docker/scssubstratee/substratee_dev).

The tag has the following format: `<Ubuntu version>-<Intel SGX SDK version>-<Rust SGX SDK version>`. We don't provide any *latest* so you must specify the tag.

If you execute

```bash
docker pull scssubstratee/substratee_dev:1804-2.12-1.1.3-001
```

you get a docker image with

* Ubuntu 18.04
* Intel SGX SDK 2.12
* Rust SGX SDK 1.1.3 (which includes the correct Rust version)
* container version 001
* IPFS 0.4.21

The following builds the code inside the docker, but the compiled binaries are stored on your local working copy.

```bash
docker run -it -v $(pwd):/root/work scssubstratee/substratee_dev:1804-2.12-1.1.3-001 /bin/bash
```

Now you can build and run your worker inside docker.

#### Enabling SGX HW Support in Docker

If you are on a platform that supports SGX, you can enable HW support by:

* Enable the SGX support in the BIOS
* Instal the [Intel SGX Driver](https://github.com/intel/linux-sgx-driver) and the [Intel SGX SDK](https://github.com/intel/linux-sgx) and make sure that `/dev/isgx` appears
* Start the docker with SGX device support:

  ```bash
  docker run -it -v $(pwd):/root/work --device /dev/isgx scssubstratee/substratee_dev:1804-2.12-1.1.3-001 /bin/bash
  ```

* Start the aesm service inside the docker:

  ```bash
  LD_LIBRARY_PATH=/opt/intel/sgx-aesm-service/aesm/ /opt/intel/sgx-aesm-service/aesm/aesm_service &
  ```

* Compile the substraTEE-worker:

  ```bash
  make
  ```

* run worker like described below

If you run the Hardware Mode on a platform that does not support SGX, you get the following error from the substraTEE-worker

```bash
*** Start the enclave
[2019-05-15T05:15:03Z ERROR substratee_worker::enclave_wrappers] [-] Init Enclave Failed SGX_ERROR_NO_DEVICE!
```

## Build Worker

In order to compile *ring* into wasm, you'll need LLVM-9 or above or you'll get linker errors. Here the instructions for Ubuntu 18.04. Skip this if you're building in our docker.

```bash
wget https://apt.llvm.org/llvm.sh
chmod +x llvm.sh
sudo ./llvm.sh 10
export CC=/usr/bin/clang-10
export AR=/usr/bin/llvm-ar-10
# if you already built, make sure to run cargo clean
```

```bash
git clone https://github.com/integritee-network/worker.git
cd worker
make
```

this might take 10min+ on a fast machine.

then you'll have to provide your SPID and KEY (see above)

```bash
echo "<YOUR SPID>" > bin/spid.txt
echo "<YOUR KEY>" > bin/key.txt
```

## Run Worker

```bash
cd bin
./substratee-worker init-shard
./substratee-worker shielding-key
./substratee-worker signing-key
./substratee-worker run --ns <yournodeip>
```


