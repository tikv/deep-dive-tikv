# Distributed SQL over TiKV

TiKV is the storage layer for [TiDB], a distributed HTAP SQL database. So far,
we have only explained how a distributed transactional Key-Value database is
implemented. However this is still far from serving a SQL database. We will
explore and cover the following things in this chapter:

- Storage

  In this section we will see how the TiDB relational structure (i.e. SQL table
  records and indexes) are encoded into the Key-Value form in the latest
  version. We will also explore a new Key-Value format that is going to be
  implemented soon and some insights on even better Key-Value formats in future.

- Distributed SQL (DistSQL)

  Storing data in a distributed manner using TiKV only utilizes distributed I/O
  resources, while the TiDB node that receives SQL query is still in charge of
  processing all rows. We can go a step further by delegating some processing
  tasks into TiKV nodes. This way, we can utilize distributed CPU resources! In
  this section, we will take a look at these supported physical plan executors
  so far in TiKV and see how they enable TiDB executing SQL queries in a
  distributed way.

- TiKV Query Execution Engine

  When talking about executors we cannot ignore discussing the execution engine.
  Although executors running on TiKV are highly simplified and limited, we still
  need to carefully design the execution engine. It is critical to the
  performance of the system. This section will cover the traditional Volcano
  model execution engine used before TiKV 3.0, for example, how it works, pros
  and cons, and the architecture.

- Vectorization

  Vectorization is a technique that performs computing over a batch of values.
  By introducing vectorization into the execution engine, we will achieve higher
  performance. This section will introduce its theory and the architecture of
  the vectorized execution engine introduced in TiKV 3.0.

[TiDB]: https://github.com/pingcap/tidb
