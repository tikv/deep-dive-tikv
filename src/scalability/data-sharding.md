# Data sharding

## What is the partition

For fault tolerance, TiKV replicates data to multiple nodes via Raft Consensus. For large datasets or high query throughput, having all the data is managed by one Raft state machine is not efficient and flexible, so we need to break the data up into partitions, also known as *sharding*, in TiKV a partition is called a `Region`.

With partitioning, we can move partitions to balance the load among all nodes of a cluster. How to decide which partitions to store on which nodes isn't covered here, it will be discussed later in a chapter related to PD.

## Partitioning

Different partitions can be placed on different nodes, thus a large dataset can be distributed across many disks, and the query load can be distributed across many processors. But partitioning suffers from the issue of *hot spots*, where high load may end up on one partition but leave the other nodes idle. To avoid that, we can simply assign records to nodes randomly, but when reading a particular item there is no fast way of knowing which node is on. 

There two general ways to map keys to partitions:

### Partitioning by hash of key

One way of partitioning is to assign a continuous range of hashes of keys to each partition. The partition boundaries can be evenly spaced, or they can be chosen by some kind of algorithm like [consistent hashing](https://en.wikipedia.org/wiki/Consistent_hashing). Due to the hash, keys can be distributed fairly among the partitions, effectively reducing hot spots, though not totally avoiding them. Unfortunately, with this method unable to do efficient range queries compared to partitioning by key range.

### Partitioning by key range

The other way of partitioning is to assign a continuous range of keys to each partition. It needs to record the boundaries between the ranges to determine which partition contains a given key, then requests can be sent to the appropriate node. One key feature of key range partition is that it is friendly to range scans. However, the downside of the sorted order is that sequential reads and writes can lead to hot spots. 

Partitioning by range is the approach that TiKV uses. The main reason to choose it is scan-friendliness, meanwhile, for the split or merge of regions, TiKV only needs to change meta-information about the range of the Region which avoids moving actual data in a large extent.
