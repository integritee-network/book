<p align="center"><img src="./fig/substraTEE-logo-scs.png" height="300"/></p>

substraTEE is a framework for [Parity Substrate](https://docs.substrate.dev/), allowing to call a custom state transition function (STF) inside a Trusted Execution Environment (TEE), namely an Intel SGX enclave thereby providing confidentiality and integrity. The enclaves operate on an encrypted state which can be read and written only by a set of provisioned and remote-attested enclaves.

What substraTEE aims to enable:

* confidential decentralized state transition functions
  * private transactions
  * private smart contracts
  * off-chain confidential personal data records (GDPR)
  * decentalized identity with selective disclosure
  * subscription-based content delivery networks
* scalability by providing a 2nd layer to substrate-based blockchains
  * off-chain smart contracts
  * payment hubs
* trusted chain bridges
* trusted oracles

substraTEE is developed by [Supercomputing Systems AG](https://www.scs.ch) and has been supported by grants from the [Web3 Foundation](https://web3.foundation/).

We also thank the teams at

* [Parity Technologies](https://www.parity.io/) for building [substrate](https://github.com/paritytech/substrate) and supporting us during development.
* [Teaclave's Rust-SGX-SDK](https://github.com/apache/incubator-teaclave-sgx-sdk) for their very helpful support and contributions.

<p align="center"><img src="./fig/web3_foundation_grants_badge_black.svg" width="300" height="300"/></p>
