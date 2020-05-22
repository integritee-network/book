# Roadmap

## Roadmap

|    Milestone    	|    Request    Invocation    	|    STF                      	|    # Workers per STF    	|    On-chain tx per invocation    	|    read chain state from STF  | Supported TEE Manufact.                   	|  Remote Attestation Registry  |
|-----------------	|-----------------------------	|-----------------------------	|-------------------------	|----------------------------------	|----------------------------------------------	| ---|---|
|    M1 &#9745;          	|    Proxy                    	|    Rust                     	|    1                    	|    2                             	|   - 	|    Intel                                     	|   -  |
|    M2 &#9745;          	|    Proxy                    	|    Rust or WASM             	|    1                    	|    2                             	|   - 	|    Intel                                     	|  -  |
|    M3 &#9745;          	|    Proxy                    	|    Rust or WASM             	|    1                    	|    2                             	|   - 	|    Intel                                     	|  X  |
|    M4 &#9745;          	|    Proxy                    	|    Rust or WASM             	|    N (redundant)        	|    1+N                           	|   - 	|    Intel                                     	|  X  |
|    M5 &#9745;          	|    Proxy                    	|    Rust modular           	|    N (redundant)        	|    1+N                           	|   - 	|    Intel                                     	|  X  |
|    M6 &#9744;          	|    Proxy                    	|    Rust modular           	|    N (redundant)        	|    1+N                           	|   X 	|    Intel                                     	|  X  |
|    future &#9744;      	|    Proxy                    	|    Rust or **Ink**	|    N (redundant)        	|    2                             	|   X 	|    Intel + ARM TrustZone + Keystone   (?)    	|  X  |
|    future &#9744;       |    **Direct**               	|    Rust or **Ink**	|    N (master + failover)    	|    **<< 1**                	|   X 	|    Intel + ARM TrustZone + Keystone   (?)    	|  X  |


### M1 PoC1: single-TEE confidential state transition function
off-chain worker runs STF within an Intel SGX enclave. The state is persisted in a sealed file which can only be read by that very enclave.

The demo STF will be a simple counter.

### M2 PoC2: single-TEE confidential state transition function in WASM
In addition to M1, the STF is defined by WASM code which is run by a WASMI interpreter within an Intel SGX enclave.

The demo STF will be a simple counter.

### M3 Remote Attestation Registry

substraTEE-worker can remote-attest its own enclave with Intel Attestation Service (IAS). The report signed by IAS is then registered on-chain with substraTEE-registry runtime module. Users can verify a worker’s IAS report before interacting with it. So they can be certain that the correct code is running on a genuine SGX CPU.

### M4 Redundancy and Secret Provisioning

Several substraTEE-workers running on different machines can redundantly operate on the same STF. This guarantees that the STF survives the loss of a few SGX machines (going offline, breaking down, denial-of-service). Moreover, this improves integrity guarantees as all the workers register call receipts including the hash of the new state. A single compromised enclave can therefore only break confidentiality, but not integrity, as manipulation would be evident to anyone.
Secret sharing among a dynamic set of worker enclaves must be implemented for such redundancy.

### M5 Modular STF with private-tx example
Since M5, the STF is modular and has its own crate which can easily be swapped. An example for private transactions has been added

### M6 read chain state from STF
From M6 onwards, SubstraTEE STF can access chain state in a trustless way. A substrate light client verification logic will be included in the worker enclave that allows the STF to query chain state by means of subscribing to storage over RPC and verifying returned values within the enclave.

### *FUTURE*

#### support for ink contracts

*(development not yet funded)*

[ink!](https://medium.com/block-journal/introducing-substrate-smart-contracts-with-ink-d486289e2b59) is substrate's domain specific contract language on top of Rust. This milestone shall bring ink! contracts to substraTEE.

### other

* direct invocation
* performance benchmarks and optimization
* testnet for stress-tests and showcasing
* use cases: bridges, payment hubs, ...

## Indirect Invocation (M1-M5)

The high level architecture of the current implementation can be seen in the following diagram:

![Diagram](./substraTEE-worker-overview.svg)

The main building blocks can be found in the following repositories:

* [substraTEE-node](https://github.com/scs/substraTEE-node): (custom substrate node) A substrate node with a custom runtime module
* [substraTEE-worker](https://github.com/scs/substraTEE-worker): (client, worker-app, worker-enclave): A SGX-enabled service that performs a confidential state-transition-function

## Redundancy (M3-M5)
The goal of redundancy is to allow multiple workers to operate on the same state to be resilient against outage of one or more workers.

The high level architecture of the proposed architecture for M3 and M4 can be seen in the following diagram:
![Diagram](./substraTEE-architecture-M4.svg)

where M3 includes only the *docker image 1* and the *Intel Attestation Service (IAS)* and M4 includes the three *docker images* and the *Intel Attestation Service (IAS)*.
