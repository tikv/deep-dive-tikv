# Paxos

Paxos is a protocol that Leslie Lamport and others have written extensively about. The most succinct paper describing Paxos is ["Paxos Made Easy"](https://lamport.azurewebsites.net/pubs/paxos-simple.pdf) published by Lamport in 2001. The original paper ["The Part-Time Parliment"](http://lamport.azurewebsites.net/pubs/pubs.html#lamport-paxos) was published in 1989.

Paxos defines several roles, and each node in a cluster may perform in one or many roles. Each cluster has a **single eventually chosen leader**, and then some number of **learners** (which take action on the agreed upon request), **Acceptors** (which form quorums and act as "memory"), and **proposers** (which advocate for client requests and coordinate).

Unlike Raft, which represents a relatively concrete protocol, Paxos represents a *family* of protocols. Each variant has different tradeoffs.

A few variants of Paxos:

* Basic Paxos: The basic protocol, allowing consensus about a single value.
* Multi Paxos: Allow the protocol to handle a stream of messages with less overhead than Basic Paxos.
* [**Cheap Paxos**](https://lamport.azurewebsites.net/pubs/web-dsn-submission.pdf): Reduce number of nodes needed via dynamic reconfiguration in exchange for reduced burst fault tolerance.
* [**Fast Paxos**](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/tr-2005-112.pdf): Reduce the number of round trips in exchange for reduced fault tolerance.
* [**Byzantine Paxos**](http://pmg.csail.mit.edu/papers/osdi99.pdf): Withstanding Byzantine failure scenarios.
* [**Raft**](https://raft.github.io/raft.pdf): Described in the next chapter.

It has been noted in the industry that Paxos is notoriously difficult to learn. Algorithms such as Raft are designed deliberately to be more easy to understand.

Due to complexities in the protocol, and the range of possibilities, it can be difficult to ascertain the state of a system when things go wrong (or are going right).

The basic Paxos algorithm itself does not define things like how to handle leader failures or membership changes.
