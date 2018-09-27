# B-Tree vs The Log-Structured Merge-Tree

[B-tree](https://en.wikipedia.org/wiki/B-tree) and [The Log-Structured Merge-tree](https://en.wikipedia.org/wiki/Log-structured_merge-tree) (LSM-tree) are the two most widely used data structures for data-intensive applications to organize and store data. However, each of them has its own advantages and disadvantages. This article aims to use the quantitative approaches to compare these two data structures.

## Metrics

In general, there are three most important metrics to measure the performance of a data structure, which include write amplification, read amplification, and space amplification. This section aims to describes these metrics.

### Write Amplification

`Write amplification` is the ratio of the amount of data written to the storage medium versus the amount of data written to the database. 

For example, if you are writing 10 MB to the database and you observe 30 MB disk write rate, your write amplification is 3.

Flash memory and solid-state drives (SSDs) can be written to only a finite number of times, so that write amplification will decrease the flash lifetime.

There is another write amplification phenomenon associated with the flash memory and SSDs because flash memory must be erased before it can be rewritten.

### Read Amplification

`Read amplification` is the number of disk reads per query. 

For example, if you need to read 5 pages to answer a query, read amplification is 5.

### Space Amplification

`Space amplification` is the ratio of the amount of data on the storage medium versus the amount of data on the database. 

For example, if you put 10MB in the database and this database uses 100MB on the disk, then the space amplification is 10.

## Analysis

The term B-tree may refer to a specific design or a general class of designs. In the narrow sense, a B-tree stores keys in its internal nodes but need not store those keys in the records at the leaves. The [B+ tree](https://en.wikipedia.org/wiki/B%2B_tree#Insertion) is one of the most famous variations of B-tree, behind which the idea is that internal nodes only contain keys, and to which an additional level which contains values is added at the bottom with linked leaves.

LSM-tree performs `compaction` to merge several `sstable`s into one new `sstable` which contains only the live data from the input `sstable`s. Compaction helps LSM-tree to recycle space and reduce read amplification. There are two kinds of `compaction strategy` which is `Size-Tiered compaction strategy` (STCS) and `Level-Based compaction strategy` (LBCS). The idea behind STCS is compacting small `sstable`s into medium `sstable`s when LSM-tree has enough small `sstable`s and compacting medium `sstable`s into large `sstable`s when LSM-tree has enough medium `sstable`s. The idea of LBCS is to organize data into levels which contains one sorted run..

This section discusses about the write amplification and read amplification of B+tree and Level-Based LSM-tree. 

### B+ Tree

In the B+ tree, copies of the keys are stored in the internal nodes; the keys and records are stored in leaves; in addition, a leaf node may include a pointer to the next leaf node to increase sequential access performance.

To simplify the analysis, assume that the block size of the tree is `B`, and keys, pointers, and records are constant size, so that each internal node contains `O(B)` children and each leaf contains `O(B)` data records. (The root node is a special case, and can be nearly empty in some situations.) Under all these assumptions, the depth of a B tree is 

<center><code>O(log<sub>B</sub>N/B)</code></center>

where `N` is the size of the database.

#### Write Amplification

For the worst-case insertion workloads, every insertion requires writing the leaf block containing the record, so the write amplification is `B`.

#### Read Amplification

The number of disk reads for any query is at most <code>O(log<sub>B</sub>N/B)</code> which is the depth of the tree. 

### Level-Based LSM-tree

In the Level-based LSM-tree, data is organized into levels. Each level contains one sorted run. Data starts in level 0, then gets merged into the level 1 run. Eventually the level 1 run is merged into the level 2 run, and so forth. Each level is constrained in its sizes. Growth factor `k` is specified as the magnification of data size at each level 

<center><code>level<sub>i</sub> = level<sub>i-1</sub>*k</code></center>

We can analyze the Level-based LSM-tree as follows. If the growth factor is `k` and the smallest level is a single file of size `B`, then the number of levels is 

<center><code>Θ(log<sub>k</sub>N/B)</code></center>

#### Write Amplification

Data must be moved out of each level once, but data from a given level is merged repeatedly with data from the previous level. On average, after being first written into a level, each data item is remerged back into the same level about `k/2` times. So the total write amplification is 

<center><code>Θ(k*log<sub>k</sub>N/B)</code><center>

#### Read Amplification

To perform a short range query in the cold cache case, we must perform a binary search
on each of the levels.

For the highest <code>level<sub>i</sub></code>, the data size is `O(N)`, so that it performs `O(logN/B)` disk reads. 

For the previous <code>level<sub>i-1</sub></code>, the data size is is `O(N/k)`, so that it performs `O(log(N/(kB))` disk reads. 

For <code>level<sub>i-2</sub></code>, the data size is <code>O(N/k<sup>2</sup>)</code>, so that it performs <code>O(log(N/k<sup>2</sup>B)</code> disk reads.

…

For <code>level<sub>i-n</sub></code>, the data size is <code>O(N/k<sup>n</sup>)</code>, so that it performs <code>O(log(N/k<sup>n</sup>B)</code> disk reads.

So that the total number of disk reads is 

<center><code>R = O(logN/B) + O(log(N/(kB)) + O(log(N/k<sup>2</sup>B) + ... + O(log(N/k<sup>n</sup>B) + 1 = O((log<sup>2</sup>N/B)/logk)</code></center>

## Summary

The following table shows the summary of various kinds of amplification

|    Data Structure    |  Write Amplification   |      Read Amplification      |
| :------------------: | :--------------------: | :--------------------------: |
|       B+ tree        |          Θ(B)          |    O(log<sub>B</sub>N/B)     |
| Level-Based LSM-tree | Θ(klog<sub>k</sub>N/B) | Θ((log<sup>2</sup>N/B)/logk) |

Through comparing various kinds of amplification between B+ tree and Level-based LSM-tree, We can come to a conclusion that Level-based LSM-tree has a better write performance than B+ tree while its read performance is not as good as B+ tree. The main purpose for TiKV to use LSM-tree instead of B-tree as its underlying storage engine is because using cache technology to promote read performance is much easier than promote write performance.