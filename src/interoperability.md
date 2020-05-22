# Interoperability

Chain interoperability is considered one of the more pressing challenges for blockchain technology. With [substraTEE](https://github.com/scs/substraTEE), we've built a tool that is well suited to solve trusted chain bridges, because the integrity of code execution is guaranteed by a TEE. 

## Example Concept for ETH Bridge

SubstraTEE-bridge will allow to:

* transfer ETH to a pegged token on substrate and back (PolkaETH).
* transfer ERC-20 token on Ethereum to a pegged token on substrate and back (PolkaXYZ).
* transfer any on-chain information between Ethereum and substrate.
* transfer any token on Polkadot to an Ethereum ERC20 token.

substraTEE-bridge will build light clients of both chains. Block headers are stored in SGX sealed storage and transaction inclusion proofs are verified in Intel SGX enclaves. Backed value will be in custody of a set of TEEs. Correct execution is guaranteed by TEEs (Intel SGX).

In contrast to an approach like [XClaim](https://https//www.xclaim.io/), substraTEE-bridge provides the following advantages:

* No full collateral needed. XClaim needs vaults (or *banks*) to lock collateral to the amount of backed value transferred through the bridge in order to punish misbehaving vaults. Because of the opportunity cost of locked capital, this would lead to higher fees for using such a bridge. In substraTEE-bridge, SGX guarantees integrity of computation. Therefore there is no need for full collateralization.
* No relay-contract with on-chain registry of block headers needed. Block headers are stored in the enclave's sealed storage. Less onchain storage is needed on the issuing chain.
* Less transactions needed as there is no need for a collateralized issue commitment.

The term ***light client*** is used abiguously in the scene. We use the following definition of light client:

> a light client can be viewed as a client that downloads block headers by default, and verifies only a small portion of what needs to be verified, using a distributed hash table as a database for trie nodes in place of its local hard drive
([source](https://github.com/ethereum/wiki/wiki/Light-client-protocol))

### Technical Concept

With regard to the document [Polkadot Bridges: Design Considerations for bridging to PoW chains](https://hackmd.io/UVzp6Z-bRAOo9Ny531yhmA) as well as the [XClaim Paper](https://https//www.xclaim.io/), we intend to implement the *CentralClaim* Protocol with a single SGX instance as a first step (Protocols: *Issue* and *Redeem*. *Transfer* and *Swap* are off-topic). Because we leverage SGX, our CentralClaim already achieves *Consistency*, as there is no way to fraudulently issue PolkaETH without breaking SGX guarantees.

As pointed out in XClaim, the CentralClaim protocol doesn't achieve *Redeemability* nor *Liveness*. In contrast to Xclaim we don't suggest a punishment scheme for fraudulent vaults as they have no economic incentive to misbehave in our scheme. Instead, we implement a shared custody of locked backing chain tokens (ETH) among a set of substraTEE-bridge enclaves. The set of substraTEE-bridge instances is unpermissioned, so anyone with suitable HW can register as a substraTEE-bridge along the design of [substraTEE](https://github.com/scs/substraTEE). A threshold signature scheme is applied in order to supply *Redeemability* and *Liveness*.

The set of active substraTEE-bridge instances changes over time. Some instances will go offline, others will join. The multisig wallets will have to be updated with every mutation. One approach with improved forward security would be to generate new multisig wallets with every mutation (instead of updating members). This way TEE's that do not update their SGX platform with the most recent patches will lose access to funds as they will be rejected by the remote attestation process and therefore not be included in the new multisig set.

We also need to guarantee *all-or-nothing fairness* (tx1 on the backing chain and tx2 on the issuing chain are either both included or both rejected). By using a TEE, we can make sure that bridges will correctly generate tx2 upon being shown a proof of inclusion for tx1. However, we can't guarantee that tx2 is included in the other chain by relying on a single TEE as the operator could prevent tx2 from being sent to the network among other attacks.  
If users have many substraTEE-bridge operators to chose from, we rely on the fact that these operators have an economic incentive to follow the protocol: They can't access locked funds but they can collect fees. We only need M-of-N (configurable threshold) of these operators to play fair in order to guarantee *all-or-nothing fairness*.

Further hardening (including timelocked transactions) is described in the [Tesseract Paper](https://eprint.iacr.org/2017/1153.pdf) and is left for future work if security audits require it.

#### Reference Light Client Implementations

Substrate as well as Ethereum clients geth and parity come with the option to run as light clients (i.e. `parity --light`). We suggest to port ethereum parity's and substrate's light clients' validation logic into SGX enclaves. While the networking can be conventional untrusted code, the tracking of block headers as well as the validation of merkle proofs need to be performed in SGX enclaves, so the bridge operator can convince others (the receiving chain) of the validity of a statement.

#### Risks

Vulnerabilities in SGX may lead to total loss of locked funds as well as infinite issuance of unredeemable PolkaETH. This risk is mitigated by our M-of-N threshold signature scheme. At least M enclaves must be compromised in order to do any harm.

If remote attestation is compromised as well, fake enclaves can join the set of substraTEE-bridges and outnumber the existing set of bridges.

### Related Work

#### parity-bridge

[Parity bridge](https://github.com/paritytech/parity-bridge) allows an Ethereum PoA chain to bridge to the Ethereum Mainnet using [contracts](https://github.com/parity-contracts/bridge/blob/master/contracts/bridge.sol) on both chains. The bridge is generic (arbitrary information can be passed) and bidirectional. Trust in the PoA chain validators is required.

substraTEE-bridge would replace this trust in PoA validators by trust in a single TEE or a set of TEE's (Intel SGX).

#### XClaim

[XClaim](https://https//www.xclaim.io/) solves the bridge problem not by trusting PoA validators but by incentivized and punishable actors. However, this approach demands full collateralization and is therefore economically less attractive at scale than substraTEE-bridge.

#### Tesseract

The [Tesseract Paper](https://eprint.iacr.org/2017/1153.pdf) describes a trusted cryptocurrency exchange design using Intel SGX. In section 7 they describe a bridge protocol.