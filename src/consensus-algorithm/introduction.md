# Consensus Algormithm

When building a distributed system one principal goal is often to build in fault-tolerance. That is, if one particular node in a network goes down, or if there is a network partition, the entire cluster does not fall over. The cluster of nodes taking part in a distributed consensus protocol must come to agreement regarding values, and once that decision is reached, that choice is final.

Distributed Consensus Algorithms often take the form of a replicated state machine and log. Each state machine accepts inputs from its log, and represents the value(s) to be replicated, for example, a hash table. They allow a collection of machines to work as a coherent group that can survive the failures of some of its members.

Two well known Distributed Consensus Algorithms are [Paxos](https://lamport.azurewebsites.net/pubs/paxos-simple.pdf) and [Raft](https://raft.github.io/raft.pdf). Paxos is used in systems like [Chubby](http://research.google.com/archive/chubby.html) by Google, and Raft is used in things like [TiKV](https://github.com/pingcap/tikv) or [etcd](https://github.com/coreos/etcd/tree/master/raft). Raft is generally seen as a more understandable and simpler to implement than Paxos.

In TiKV we harness Raft for distributed consensus. We found it much easier to understand both the algorithm, and how it will behave in even truly perverse scenarios.

