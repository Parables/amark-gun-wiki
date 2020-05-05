This is not a discussion on what the [CAP Theorem](https://en.wikipedia.org/wiki/CAP_theorem) is, but on the tradeoffs that GUN decides to default to. Mark gave a great talk on this in Berlin, which [explains everything in easy to understand](https://gun.eco/distributed/matters.html) terms. Kyle Kingsbury (Aphyr) of the Jepsen tests (which inspired our [distributed testing](https://github.com/gundb/panic-server) framework) was there, and he [tweeted](https://twitter.com/aphyr/status/646302398575587332) about us. And don't forget to check out our security series on [P2P cryptography](https://gun.eco/explainers/data/security.html).

### AP

GUN is an AP system by default, this means GUN can successfully read and write data even if other peers are offline. As a result, GUN is not strongly consistent in linearizable ways. Instead, it is eventually consistent in order to be highly available to users regardless of connectivity. When connectivity is or becomes available again, GUN will synchronize data at low latency using at least once delivery to guarantee that all peers will deterministically converge to the same data within a time frame, without any extra coordination or gossip.

### Eventual Consistency

As mentioned, GUN opts for eventual consistency. This means that if two people on opposite sides of the world modified the same piece of data at the "same time", they would not know about each other's conflicting updates until the information had propagated to them over the slow network. Meaning their updates will not resolve to a "consistent" view until some amount of time has passed. Why is this the case? Because physics cannot break the speed of light, meaning instantaneous communication is impossible in all known sciences.

### Strong Consistency

So how is strong consistency possible, then? It is built ontop of a messaging system that is eventually consistent. These messages broadcast a "lock" request to all peers for the data to be modified (this example ignores potential lock request conflicts). The peers "ack" back accepting the data lock, meaning they will not allow local users to read or write on that data, resulting in high latency for the user. Once the original peers has received unanimous approval, it will then write the data and wait till it hears back success from the peers. The following two things can be observed from this:

 - That system is not highly available, if there is a network partition (aka if any one peer cannot be reached) the data cannot be said to be consistent without waiting an indefinite amount time.
 - Strong consistency is an emergent property of an eventually consistent messaging system. Meaning GUN could evolve into a CP system at the loss of being highly available.

### Linearizability

While the last point is interesting, it is not recommended that one evolves GUN into a locking system. Instead, new research in Convergent Replicated Data Types (CRDTs), Directed Acyclic Graphs (DAGs), Operational Transformation (OT or WOOTs), and some Block Chain Ledgers provide similar guarantees without simulating any centralized mechanisms. These systems declare the "dependent causality" explicitly in the data, rather than the protocol - and thus should be preferred since they give you some form of linearizability without sacrificing the advantages of a peer-to-peer system. See [this article](https://queue.acm.org/detail.cfm?id=2610533) for more details.

### Summary

GUN is AP with eventual consistency by default, but it is possible to upgrade into a CP system but it is not recommended. Instead, one should apply some form of "dependent causality" into the data itself at the application layer manually or as a module that wraps it for you.

Again, we recommend you check out the [tech talk](https://gun.eco/distributed/matters.html). After you have read it, check out the documentation on how it is implemented, [here](Conflict-Resolution-with-Guns).