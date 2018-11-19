# Key-Value Engine

A key-value engine serves at the bottommost layer in a key-value
database, unless you are going to build your own file system or
operating system. A key-value engine is crucial for a database because
it manages all the persistent data directly.

Most key-value engines provide some common interfaces like `Get`,
`Put` and `Delete`. Some engines also allow you to iterate the
key-values in order efficiently. Usually, a storage engine provides
some special features more or less to let you take good use of it too.

Choosing a key value engine is the first step to build a
database. Here are some important things we need to consider:

- The data structure. Different data structures are optimized for
  different workloads. Some are good for reads and some are good for
  writes, etc.
- Maturity. We don't need a storage engine to be fancy but we want it
  to be reliable. Buggy engines ruin everything you build on top of
  them. We recommend using a battle-tested storage engine which has
  been adopted by a lot of users.
- Performance. The performance of the storage engine limits the
  overall performance of the whole database. So make sure the storage
  engine meet your expectation and have the potential to improve along
  with the database.

In this chapter, we will do a comparison between two well-known data
structures and guide you through the storage engine used in TiKV.
