#distributed algorithm

## Two-Phase Commit

In transaction processing, databases, and computer networking, the two-phase commit protocol (2PC) is a type of atomic commitment protocol (ACP). It is a distributed algorithm that coordinates all the processes that participate in a distributed atomic transaction on whether to commit or abort (rollback) the transaction (it is a specialized type of consensus protocol). The protocol achieves its goal even in many cases of temporary system failure (involving either process, network node, communication, etc. failures), and is thus widely used. However, it is not resilient to all possible failure configurations, and in rare cases, user (e.g., a system's administrator) intervention is needed to remedy an outcome. To accommodate recovery from failure (automatic in most cases) the protocol's participants use logging of the protocol's states. Log records, which are typically slow to generate but survive failures, are used by the protocol's recovery procedures. Many protocol variants exist that primarily differ in logging strategies and recovery mechanisms. Though usually intended to be used infrequently, recovery procedures compose a substantial portion of the protocol, due to many possible failure scenarios to be considered and supported by the protocol.

### Basic Algorithm of 2PC

#### prepare phase

The coordinator sends a prepare message to all cohorts and waits until it has received a reply from all cohorts.

#### commit phase

If the coordinator received an agreement message from all cohorts during the commit-request phase, the coordinator sends a commit message to all the cohorts.

If any cohort votes No during the prepare phase (or the coordinator's timeout expires), the coordinator sends a rollback message to all the cohorts.

### Disadvantages of 2PC

The greatest disadvantage of the two-phase commit protocol is that it is a blocking protocol. If the coordinator fails permanently, some cohorts will never resolve their transactions: After a cohort has sent an agreement message to the coordinator, it will block until a commit or rollback is received.

### 2PC Practice in TiKV

In TiKV we adopt percolator transaction model which is a variant of two phase commit(we will describe this transaction model deeply later). To address the disadvantage of coordinator fails permanently, percolator doesn’t use any node as coordinator, instead it uses one of the transaction involved keys as coordinator, which we call it primary key, and the other keys we call them secondary keys. Since each key has multiple replicas, and data is keep consistent between these replicas by using consensus protocol(raft in TiKV), one node’s failure doesn’t affect the accessibility of data. So percolator can tolerate node fails permanently.

## Three-Phase Commit 

Unlike the two-phase commit protocol (2PC) however, 3PC is non-blocking. Specifically, 3PC places an upper bound on the amount of time required before a transaction either commits or aborts. This property ensures that if a given transaction is attempting to commit via 3PC and holds some resource locks, it will release the locks after the timeout.

#### 1st phase
The coordinator receives a transaction request. If there is a failure at this point, the coordinator aborts the transaction. Otherwise, the coordinator sends a canCommit? message to the cohorts and moves to the waiting state.

#### 2st phase
If there is a failure, timeout, or if the coordinator receives a No message in the waiting state, the coordinator aborts the transaction and sends an abort message to all cohorts. Otherwise the coordinator will receive Yes messages from all cohorts within the time window, so it sends preCommit messages to all cohorts and moves to the prepared state.

#### 3rd phase
If the coordinator succeeds in the prepared state, it will move to the commit state. However if the coordinator times out while waiting for an acknowledgement from a cohort, it will abort the transaction. In the case where an acknowledgement is received from the majority of cohorts, the coordinator moves to the commit state as well.

A two-phase commit protocol cannot dependably recover from a failure of both the coordinator and a cohort member during the Commit phase. If only the coordinator had failed, and no cohort members had received a commit message, it could safely be inferred that no commit had happened. If, however, both the coordinator and a cohort member failed, it is possible that the failed cohort member was the first to be notified, and had actually done the commit. Even if a new coordinator is selected, it cannot confidently proceed with the operation until it has received an agreement from all cohort members, and hence must block until all cohort members respond.

The three-phase commit protocol eliminates this problem by introducing the Prepared to commit state. If the coordinator fails before sending preCommit messages, the cohort will unanimously agree that the operation was aborted. The coordinator will not send out a doCommit message until all cohort members have ACKed that they are Prepared to commit. This eliminates the possibility that any cohort member actually completed the transaction before all cohort members were aware of the decision to do so (an ambiguity that necessitated indefinite blocking in the two-phase commit protocol).

### Disadvantages of 3PC

The main disadvantage to this algorithm is that it cannot recover in the event the network is segmented in any manner. The original 3PC algorithm assumes a fail-stop model, where processes fail by crashing and crashes can be accurately detected, and does not work with network partitions or asynchronous communication.

The protocol requires at least three round trips to complete, needing a minimum of three round trip times (RTTs). This is potentially a long latency to complete each transaction.

## Paxos Commit

The Paxos Commit algorithm runs a Paxos consensus algorithm on the commit/abort decision of each participant to obtain a transaction commit protocol that uses 2F + 1 coordinators and make progress if at least F + 1 of them are working properly. Paxos Commit has the same stable-storage write delay, and can be implemented to have the same message delay in the fault-free case, as Two-Phase Commit, but it uses more messages. The classic Two-Phase Commit algorithm is obtained as the special F = 0 case of the Paxos Commit algorithm.

In the Two-Phase Commit protocol, the coordinator decides whether to abort or commit, records that decision in stable storage, and informs the cohorts of its decision. We could make that fault-tolerant by simply using a consensus algorithm to choose the committed/aborted decision, letting the cohorts be the client that proposes the consensus value. This approach was apparently first proposed by Mohan, Strong, and Finkelstein, who used a synchronous consensus protocol. However, in the normal case, the leader must learn that each cohort has prepared before it can try to get the value committed chosen. Having the cohorts tell the leader that they have prepared requires at least one message delay. 
