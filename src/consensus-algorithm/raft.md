# Raft

In 2014, Diego Ongaro and John Ousterhout presented the Raft algorithm. It is explained succinctly in a [paper](https://raft.github.io/raft.pdf) and detailed at length in a [thesis](https://ramcloud.stanford.edu/~ongaro/thesis.pdf).

Raft defines a strong, single leader and number of followers in a group of peers. The group represents a **replicated state machine**. Only the leader may service client requests. The leader replicates actions to the followers.

Each peer has a **durable write ahead log**. All peers append each action as an entry in the log immediately as they recieve it. When the quorum (the majority) of peers have confirmed that that the entry exists in their log, the leader commits the log, each peer then can apply the action to their state machine.

Raft guarantees **strong consistency** by having only one ‘leader’ of the group which services all requests.  All requests are then replicated to a quorum before being acted on, then confirmed with the requester. From the perspective of the cluster, the leader always has an up to date state machine.

The group is **available** when a majority of peers are able to coordinate. If the group is partitioned, only a partition containing the majority of the group can recover and resume servicing requests. If the cluster is split into three equal subgroups, for example, none of the subgroups will recover and service requests.

Raft supports **leader elections**. If the group leader fails, one of the followers will be elected the new leader. It’s not possible for a stale leader to be elected. If a leader candidate is aware of requests which the other peers of a particular subgroup are not, it will be elected over those peers. Since only the majority of peers can form a quorum this means that in order to be elected a peer must be up to date.

Because of how leader elections work, Raft is not Byzantine fault tolerant. Any node is able to lie and subvert the cluster by starting an election and claiming to have log entries it didn’t have.

It’s possible to support actions such as changing the peer group’s membership, or replicating log entries to non-voting followers (learners). In addition, several optimizations can be applied to Raft. Prevote can be used to introduce a pre-election by possible leaders, allowing them to gauge their ability to become a leader before potentially disrupting a cluster. Joint Consensus can support arbitrary group changes, allowing for better scaling. Batching and pipelining can help high throughput systems perform better.