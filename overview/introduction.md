# Introduction

[TiKV](https://github.com/tikv/tikv) is a distributed, transactional key-value database. It has been widely used in the world, especially in many critical production environments. It has also been adopted by CNCF as a [sandbox project](https://www.cncf.io/blog/2018/08/28/cncf-to-host-tikv-in-the-sandbox/).

TiKV provides a fully [ACID](https://en.wikipedia.org/wiki/ACID_(computer_science)) support, automatic horizontal scalability, global data consistency, Geo-replication, and many other useful features, it can be used as a building block to build other high-level services. E.g, we have already used TiKV to support [TiDB](https://github.com/pingcap/tidb) - a next-generation [HTAP](https://en.wikipedia.org/wiki/Hybrid_transactional/analytical_processing_(HTAP)) database. 

In this book, we will give you an introduction of TiKV. You can know why we build it, what problems do we meet, which technical do we choose, etc. We hope through this book, you can have a deep understanding of TiKV, learn enough knowledge of distributed programming, or even start to build your own distributed system. :-)

## History 

In the middle of 2015, we wanted to build a database which solved the scaling problem of MySQL. At that time, the most used way was to build a proxy on top of the MySQL servers, we didn't think proxy was an elegant way (Definitely, we still don't think either now). 

As far as we knew, the proxy had following problems:

+ The proxy can't satisfy a fully ACID compliance easily, especially when the transaction crosses different MySQL servers.
+ The users still need to care the data distribution and design their sharding key carefully to avoid inefficient query.
+ The high availability and data consistency of MySQL can't be guaranteed easily based on the traditional Master-Salve replication. 

Although building a proxy based on MySQL directly might be easy at the beginning, we still decided to choose another way, a more hardcore way - to build a distributed, MySQL compatible database from scratch.  

Fortunately, Google met the same problem and had already published some papers to describe how they built [Spanner](http://static.googleusercontent.com/media/research.google.com/en//archive/spanner-osdi2012.pdf) and [F1](https://storage.googleapis.com/pub-tools-public-publication-data/pdf/41344.pdf) to solve it. Spanner is a globally distributed, externally consistent database and F1 is a distributed SQL database based on Spanner. Refer to Spanner and F1, we knew we could do the same thing. So we started to build TiDB - a stateless MySQL layer like F1. After we released TiDB, we knew we need a Spanner-like database, then we began to develop TiKV. 

## Architecture

Following is the architecture of TiKV: 

![Architecture](./architecture.png)

According to the picture, there are three TiKV instances in the cluster, each instance uses one [RocksDB](https://github.com/facebook/rocksdb) to save data. Beyond the RocksDB, we use [Raft](https://raft.github.io/) consensus algorithm to replica the data. Mostly, we should use at lease three replicas to keep data safety and consistence, and these replicas form a Raft group. 

We use the traditional [MVCC](https://en.wikipedia.org/wiki/Multiversion_concurrency_control) mechanism and build a distributed transaction layer above the Raft layer, we also provide a coprocessor framework to let the users push down their computing logic to the storage layer and let TiKV calculate directly and return the result. 

All the network communications are through [gRPC](https://grpc.io/) to let the contributor develop their own clients easily.  

The whole cluster is managed and scheduled by a center service - [placement driver(PD)](https://github.com/pingcap/pd). 

As you can see, the hierarchy of TiKV is clear and easy to understand, and we will give more detailed explanation later.
