# Trusted Execution

We have become accustomed to the fact that we have to trust our IT administrators. While these administrators used to be in-house employees at our companies, today we often work on rented cloud platforms.

These administrators can read and modify all the data processed on any machine they manage. Unfortunately, this ability extends not only to known administrators we trust but also to hackers who can obtain administrator privileges. No company, no matter how qualified, is immune to such attacks.

Enter TEEs.

You may envision a TEE as a co-processor that manages its own cryptographic keys and only executes programs whose hash, or fingerprint, corresponds to the original code. The manufacturer of the processor guarantees, by the design of their hardware, that nobody has access to the internal keys of the TEE or can read its memory. Moreover, the manufacturer can authenticate each TEE and provide remote attestation to a user to confirm that her untampered program is actually running on a genuine TEE, even if the machine is physically located in an off-site data center.

TEEs promise, in short, integrity and confidentiality of (remote) computation. You should be aware, however, of possible [security threats](./security.md).

Assuming we trust TEE manufacturers’ integrity and design competence, TEEs allow us to execute any state update without sharing our data with the blockchain validator or other users. Private token transfers, private smart contracts and private state channels thus become possible and relatively cheap.