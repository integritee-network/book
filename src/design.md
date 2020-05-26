# Design

TODO: update and explain better

## Direct Invocation (Vision)

![vision](./fig/substraTEE-vision.png)
*SubstraTEE Target Architecture with Direct Invocation (future scenario)*

* Shielding key: used by the substraTEE-client to encrypt the call in order to protect caller privacy. It is common to all enclaves.
* State encryption key: used to encrypt and decrypt the state storage. It is common to all enclaves.
* Signing key: used to sign transactions for the substraTEE-node. The corresponding account must be funded in order to pay for chain fees. It is unique for every enclave.

### How it works

The *substraTEE-node* is a substrate blockchain node with an additional runtime module:

* substraTEE-registry module: proxies requests to workers, verifies IAS reports and keeps track of the registered enclaves. It provides the following API interfaces:
  * Register an enclave
  * Remove an enclave
  * Get the list of enclaves
  * invoke worker
  * confirm processing of requests

The *substraTEE-worker* checks on the first start-up if "his" enclave is already registered on the chain. If this is not the case, it requests a remote attestion from the Intel Attestation Service (IAS) and sends the report to the *substraTEE-registry module* to register his enclave. 

If there is already an enclave (belonging to a different substraTEE-worker) registered on the chain, the substraTEE-worker requests provisioning of secrets (the *shielding and state encryption private key*) from the already registered enclave. The exchange of critical information between the enclaves is performed over a secure connection (TLS). The two enclaves perform a mutual remote attestation before exchanging any secrets.

## Indirect Invocation (M1-M5)

The high level architecture of the current implementation can be seen in the following diagram:

![Diagram](./fig/substraTEE-worker-overview.svg)

The main building blocks can be found in the following repositories:

* [substraTEE-node](https://github.com/scs/substraTEE-node): (custom substrate node) A substrate node with a custom runtime module
* [substraTEE-worker](https://github.com/scs/substraTEE-worker): (client, worker-app, worker-enclave): A SGX-enabled service that performs a confidential state-transition-function

### Request Lifetime end-to-end

![request-end-to-end](./fig/substraTEE_request_format_end2end.svg)

## Redundancy (M3 onwards)

The goal of redundancy is to allow multiple workers to operate on the same state to be resilient against outage of one or more workers.

The high level architecture for M3 and M4 can be seen in the following diagram:

![Diagram](./fig/substraTEE-architecture-M4.svg)

where M3 includes only the *docker image 1* and the *Intel Attestation Service (IAS)* and M4 includes the three *docker images* and the *Intel Attestation Service (IAS)*.
