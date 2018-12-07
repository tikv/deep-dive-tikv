# RocksDB

RocksDB is a persistent key-value store for fast storage environment.
Here are some highlight features from RocksDB:

1. RocksDB uses a log structured database engine, written entirely in
   C++, for maximum performance. Keys and values are just
   arbitrarily-sized byte streams.
2. RocksDB is optimized for fast, low latency storage such as flash
   drives and high-speed disk drives. RocksDB exploits the full
   potential of high read/write rates offered by flash or RAM.
3. RocksDB is adaptable to different workloads. From database storage
   engines such as MyRocks to application data caching to embedded
   workloads, RocksDB can be used for a variety of data needs.
4. RocksDB provides basic operations such as opening and closing a
   database, reading and writing to more advanced operations such as
   merging and compaction filters.

TiKV uses RocksDB because RocksDB is mature and high-performance. In
this section, we will explore how TiKV uses RocksDB. We won't talk
about basic features like `Get`, `Put`, `Delete`, and `Iterate` here
because their usage is simple and clear and works well too. Instead,
we'll focus some special features used in TiKV below.

## [Prefix Bloom Filter](https://github.com/facebook/rocksdb/wiki/RocksDB-Bloom-Filter)

A Bloom Filter is a magical data structure that uses a little resource
but helps a lot. We won't explain the whole algorithm here. If you are
not familiar with Bloom Filters, you can think it as a black box
inside a dataset, which can tell you if a key *probably* exists or
*definitely* does not without actually searching the
dataset. Sometimes Bloom Filter gives you a false-positive answer
although it rarely happens.

TiKV uses a Bloom Filter as well as a variant which is called Prefix
Bloom Filter (PBF). Instead of telling you if a whole key exists in a
dataset or not, PBF tells you if there are some other keys with the
same prefix exists. Since PBF only stores the unique prefixes instead
of all unique whole keys, it can save some memory too with the down
side of having larger false positive rate.

TiKV supports MVCC, which means that there can be multiple versions
for the same row stored in RocksDB. All versions of the same row share
the same prefix (the row key) but have different timestamps as a suffix. When
we want to read a row, we usually don't know about the exact version
to read, but only want to read the latest version at a specific
timestamp. This is where PBF shines. PBF can filter out data which is
impossible to contain keys with the same prefix as the row key we
provided. Then we just need to search in the data that may contain
different versions of the row key and locate the specific version we
want.

## TableProperties

RocksDB allows us to register some table properties collectors.  When
RocksDB builds an SST file, it passes the sorted key-values one by one
to the callback of each collector so that we can collect whatever we
want. Then when the SST file is finished, the collected properties
will be stored inside the SST file too.

We use this feature to optimize two functionalities.

The first one is for Split Check. Split check is a worker to check if
regions are large enough to split.  We have to scan all the data
within a region to calculate the size of the region at the original
implementation, which is resource consuming. With the `TableProperties`
feature, we record the size of small sub-ranges in each SST file so
that we can calculate the approximate size of a region from the table
properties without scanning any data at all.

Another one is for MVCC Garbage Collection (GC). GC is a process to
clean up garbage versions (versions older than the configured
lifetime) of each row. If we have no idea whether a region contains
some garbage versions or not, we have to check all regions
periodically. To skip unnecessary garbage collection, we record some
MVCC statistics (e.g. the number of rows and the number of versions)
in each SST file. So before checking every region row by row, we check
the table properties to see if it is necessary to do garbage
collection on a region.

## CompactRange

From time to time, some regions may contain a lot of tombstone entries
because of GC or other delete operations. Tombstone entries are not
good for scan performance and waste disk space as well.

So with the `TableProperties` feature, we can check every region
periodically to see if it contains a lot of tombstones. If it does, we
will compact the region range manually to drop tombstone entries and
release disk space.

We also use `CompactRange` to recover RocksDB from some mistakes like
incompatible table properties across different TiKV versions.

## [EventListener](https://github.com/facebook/rocksdb/wiki/EventListener)

`EventListener` allows us to listen to some special events, like
flush, compaction or write stall condition change. When a specific
event is triggered or finished, RocksDB will invoke our callbacks with
some information about the event.

TiKV listens to the compaction event to observe the region size
changes. As mentioned above, we calculate the approximate size of each
region from the table properties. The approximate size will be
recorded in the memory so that we don't need to calculate it again and
again if nothing has changed. However, during compactions, some
entries are dropped so the approximate size of some regions should be
updated. That's why we listen to the compaction events and recalculate
the approximate size of some regions when necessary.

## [IngestExternalFile](https://github.com/facebook/rocksdb/wiki/Creating-and-Ingesting-SST-files)

RocksDB allows us to generate an SST file outside and then ingest the
file into RocksDB directly. This feature can potentially save a lot of
IO because RocksDB is smart enough to ingest a file to the bottom
level if possible, which can reduce write amplification because the
ingested file doesn't need to be compacted again and again.

We use this feature to handle Raft snapshot. For example, when we want
to add a replica to a new server. We can first generate a snapshot
file from another server and then send the file to the new
server. Then the new server can ingest that file into its RocksDB
directly, which saves lots of work!

We also use this feature to import a huge amount of data into TiKV. We
have some tools to generate sorted SST files from different data
sources and then ingest those files into different TiKV servers. This
is super fast compared to writing key-values to a TiKV cluster in the
usual way.

## DeleteFilesInRange

Previously, TiKV used the straightforward way to delete a range of data,
which is scanning all the keys in the range and then delete them one
by one. However, disk space would not release until the tombstones have
been compacted. Even worse, disk space usage will actually increase
temporarily because of newly written tombstones.

As time goes on, users store more and more data in TiKV until their
disk space is insufficient. Then users will try to drop some tables or
add more stores and expect the disk space usage to decrease in a short
time. But TiKV didn't meet expectations with this method. We first tried
to solve this by using the `DeleteRange` feature in RocksDB. However,
`DeleteRange` turns out to be unstable and can not release disk space
fast enough.

A faster way to release disk space is to delete some files directly,
which leads us to the `DeleteFilesInRange` feature. But this feature
is not perfect, it is quite dangerous because it breaks the snapshot
consistency. If you acquire a snapshot from RocksDB,
use `DeleteFilesInRange` to delete some files, then try to read that data
you will find that some of it is missing. So we
should use this feature carefully.

TiKV uses `DeleteFilesInRange` to destroy tombstone regions and GC
dropped tables. Both cases have a prerequisite that the dropped range
must not be accessed anymore.
