# Introduction

In the database field, scalability is the term we use to describe the capability of a system to handle a growing amount of work. Even if a system is working reliably and fast today, it doesn't mean it will necessarily work well in the future. One common reason for degradation is the increased load which exceeds what the system can process. In modern systems, the amount of data we handle can far outgrow our original expectations, so scalability is a critical consideration for the design of a database.

A system whose performance improves after adding hardware, proportionally to the capacity added, is said to be a scalable system. TiKV is a highly scalable key-value store, especially comparing with other stand-alone key-value stores like [RocksDB](https://rocksdb.org/) and [LevelDB](https://github.com/google/leveldb). In this chapter we will talk about the two main ways of scaling, horizontal and vertical, and how TiKV provide strong scalability based on Raft.
