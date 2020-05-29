# Scalability

substraTEE with [direct invocation](./design.md#direct-invocation-vision) is a 2nd layer technology: It allows to do state transitions without the individual transactions hitting the blockchain.

* more transactions per second because no global consensus must be reached on individual transactions (we trust the TEEs integrity)
* less latency because there's no need to wait for block inclusion
* disk usage and load balancing can be scaled horizontally: substraTEE also features [sharding](./sharding.md)
