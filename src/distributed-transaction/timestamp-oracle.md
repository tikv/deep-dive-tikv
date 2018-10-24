# Timestamp Oracle

The timestamp oracle plays a significant role in Percolator Transaction model, it is a server that hands out timestamps in strictly increasing order: a property required for correct operation of the snapshot isolation protocol. 

Since every transaction requires contacting the timestamp oracle twice, this service must scale well. The oracle periodically allocates a range of timestamps by writing the highest allocated timestamp to stable storage; given an allocated range of timestamps, the oracle can satisfy future requests strictly from memory. If the oracle restarts, the timestamps will jump forward to the maximum allocated timestamp (but will never go backwards).

To save RPC overhead (at the cost of increasing transaction latency) each timestamp requester batches timestamp requests across transactions by maintaining only one pending RPC to the oracle. As the oracle becomes more loaded, the batching naturally increases to compensate. Batching increases the scalability of the oracle but does not affect the timestamp guarantees. 

The transaction protocol uses strictly increasing timestamps to guarantee that Get() returns all committed writes before the transaction’s start timestamp. To see how it provides this guarantee, consider a transaction R reading at timestamp TR and a transaction W that committed at timestamp TW < TR; we will show that R sees W’s writes. Since TW < TR, we know that the timestamp oracle gave out TW before or in the same batch as TR; hence, W requested TW before R received TR. We know that R can’t do reads before receiving its start timestamp TR and that W wrote locks before requesting its commit timestamp TW . Therefore, the above property guarantees that W must have at least written all its locks before R did any reads; R’s Get() will see either the fully committed write record or the lock, in which case W will block until the lock is released. Either way, W’s write is visible to R’s Get().

In our system, the timestamp oracle has embeded into Placement Drivers (PD) componet. PD is a manager component with god view that is responsible for storing metadata and conduct load balancing.

## Practice in TiKV

Since we use batching and preallocating technique to increase the timestamp oracle’s throughput, and also we use a raft group to tolerate node failure, but there are still some disadvantages for allocating timestamps from a single node. Obviously one disadvantage is the scalability of this component, and another is when the current leader crash, there is a gap that can’t allocate timestamp before new leader has been elected. The 3rd disadvantage is when the timestamp requestor is located at a remote datacenter which has a long distance from the datacenter which the timestamp oracle at, the requestor must tolerate the high latency caused by the network round trip. There are some potential solutions for this case, such as google spanner’s TureTime mechanism and HLC(Hybrid Logical Clock).
