# How To Run Your Own Worker

## HW requirements

While SGX is supported by most Intel CPU's that is not the case for all chipsets. Here we're just telling you what HW we are using and is known to work. 

### Dell PowerEdge R340 Server

CPU **has to be** Intel(R) Xeon(R) E-2276G CPU @ 3.80 GHz

### Enable SGX support in BIOS

TODO

## Register With Intel

TODO

....Developer Account....

### Obtaining a Commercial SGX License

TODO

* only legal entities


## Setup Server with Ansible

TODO

..ansible scripts on github.... 


## If using Docker

### Enabling SGX HW support
The demos are by default compiled for [Simulation Mode](https://software.intel.com/en-us/blogs/2016/05/30/usage-of-simulation-mode-in-sgx-enhanced-application) meaning that you don't need an actual SGX platform to run the example. This is specified in the `DockerfileM*` on line 99 (`SGX_MODE=SW make`). If you are on a platform that supports the SGX, you can enable HW support by:
  * Installing the Intel SGX Driver 2.5 and make sure that `/dev/isgx` appears
  * Start the docker with SGX device support:
    ```bash
    $ docker run -v $(pwd):/substraTEE/backup -ti --device /dev/isgx substratee
    ```
  * Start the aesm service inside the docker:
    ```bash
    root@<DOCKERID>:/# LD_LIBRARY_PATH=/opt/intel/libsgx-enclave-common/aesm /opt/intel/libsgx-enclave-common/aesm/aesm_service &
    ```
  * Compile the substraTEE-worker with HW support:
    ```bash
    root@<DOCKERID>:/substraTEE/substraTEE-worker-M1# make
    ```
  * Re-run the demos.

If you run the Hardware Mode on a platform that does not support SGX, you get the following error from the substraTEE-worker
```
*** Start the enclave
[2019-05-15T05:15:03Z ERROR substratee_worker::enclave_wrappers] [-] Init Enclave Failed SGX_ERROR_NO_DEVICE!
```
