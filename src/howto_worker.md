# How To Run Your Own Worker

## HW requirements

While SGX is supported by most Intel CPU's that is not the case for all chipsets. Here we're just telling you what HW we are using and is known to work. 

### Dell PowerEdge R340 Server

CPU **has to be** Intel(R) Xeon(R) E-2276G CPU @ 3.80 GHz

### Enable SGX support in BIOS

TODO

## Intel SGX development and production (commercial) license
In order to perform a remote attestation of the enclave, an [Intel SGX Attestation Enhanced Service Privacy ID (EPID)](https://api.portal.trustedservices.intel.com/EPID-attestation) is needed. We use unlinkable quotes in our code. Developers need to [register an account with Intel](https://software.intel.com/en-us/form/sgx-onboarding)

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
TODO


## Setup Server with Ansible

TODO

..ansible scripts on github.... 

## Build Worker

```
git clone https://github.com/scs/substraTEE-worker.git
cd substraTEE-worker
make
```

this might take 10min+ on a fast machine.

then you'll have to provide your SPID and KEY (see above)

```
echo "<YOUR SPID>" > bin/spid.txt
echo "<YOUR KEY>" > bin/key.txt
```

## Run Worker

```
cd bin
./substratee-worker init-shard
./substratee-worker shielding-key
./substratee-worker signing-key
./substratee-worker run --ns <yournodeip>
```

## Using Docker

```
docker pull scssubstratee/substratee_dev:18.04-2.9.1-1.2
docker run -it -v $(pwd):/root/work scssubstratee/substratee_dev:18.04-2.9.1-1.1.2 /bin/bash
```
Now you can build and run your worker inside docker. Just replace the make command above with 
```
SGX_MODE=SW make
```
to run in [Simulation Mode](https://software.intel.com/en-us/blogs/2016/05/30/usage-of-simulation-mode-in-sgx-enhanced-application) without the need of an actual SGX platform.

### Enabling SGX HW support
If you are on a platform that supports SGX, you can enable HW support by:
  * Installing the Intel SGX Driver 2.9 and make sure that `/dev/isgx` appears
  * Start the docker with SGX device support:
    ```bash
    docker run -it -v $(pwd):/root/work --device /dev/isgx scssubstratee/substratee_dev:18.04-2.9.1-1.1.2 /bin/bash
    ```
  * Start the aesm service inside the docker:
    ```bash
    LD_LIBRARY_PATH=/opt/intel/sgx-aesm-service/aesm/ /opt/intel/sgx-aesm-service/aesm/aesm_service &
    ```
  * Compile the substraTEE-worker with HW support:
    ```bash
    make
    ```
  * run worker like described above

If you run the Hardware Mode on a platform that does not support SGX, you get the following error from the substraTEE-worker
```
*** Start the enclave
[2019-05-15T05:15:03Z ERROR substratee_worker::enclave_wrappers] [-] Init Enclave Failed SGX_ERROR_NO_DEVICE!
```