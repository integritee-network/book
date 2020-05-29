# Privacy

substraTEE leverages the confidentiality guarantees of TEEs, namely Intel SGX.

Whatever is computed inside substraTEE worker enclaves can't be observed by the network nor by the operator of the worker service nor by the root user nor a cloud provider admin.

What substraTEE can't provide is network layer privacy. That's what [Tor](https://www.torproject.org/) or [Nym](https://nymtech.net) are for.
