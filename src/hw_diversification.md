# HW Diversification

As of the time of writing, Intel SGX is the only TEE that allows remote attestation. This dependency on Intel is a single point of failure and as such undesirable for Integritee.

The Integritee team is investigating ways to provide remote attestation for ARM TrustZone and open source TEEs like Keystone.

Diversification would have a positive effect on TEE *integrity* because a vulnerability in one type of TEE would only affect a fraction of all TEEs. A pretty simple consensus algorithm could ensure integrity even in presence of large-scale attacks exploiting that vulnerability.

However, diversification could have a negative impact on *confidentiality*. If secrets are provisioned to several types of TEEs, it only takes a single TEE to leak the secret to compromise it for all.

## Open Source Remote Attestation Process

See [Distributor-Level Remote Attestation](./hw_ra_by_distributor.md)