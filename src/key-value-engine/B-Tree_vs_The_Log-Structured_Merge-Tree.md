# B-Tree vs The Log-Structured Merge-Tree

[B-tree](https://en.wikipedia.org/wiki/B-tree) and [The Log-Structured Merge-tree](https://en.wikipedia.org/wiki/Log-structured_merge-tree) (LSM-tree) are the two most widely used data structures for data intensive applications to organize and store data. However, each of them has its own advantages and disadvantages. This article is aim to use the quantitative approaches to compare these two data structures.

## Metrics

In general, there are three most important metrics to measure the performance of a data structure, which include write amplification, read amplification, and space amplification. This section is aimed to describe these metrics.

### Write Amplification

`Write amplification` is the ratio of the amount of data written to storage medium compared to the amount of data written to database. 

For example, if you are writing 10 MB/s to the database and you observe 30 MB/s disk write rate, your write amplification is 3.

Flash memory and solid-state drives (SSDs) can be written to only a finite number of times, so that write amplification will decreases flash lifetime.

There is another write amplification phenomenon associated with flash memory and SSDs because  flash memory must be erased before it can be rewritten.

### Read Amplification

`Read amplification` is the number of I/O's per query. 

For example, if you need to read 5 pages to answer a query, read amplification is 5.

### Space Amplification

`Space amplification` is the ratio of the amount of data on storage medium compared to the amount of data on database. 

For example, if you Put 10MB in the database and it uses 100MB on disk, then the space amplification is 10.

## Analysis

The term B-tree may refers to a specific design or it may refers to a general class of designs. In the narrow sense, a B-tree stores keys in its internal nodes but need not store those keys in the records at the leaves. The [B+ tree](https://en.wikipedia.org/wiki/B%2B_tree#Insertion) is one of the most famous variation of B-tree, behind which the idea is that internal nodes only contains keys, and to which an additional level which contains values is added at the bottom with linked leaves.

LSM-tree performs `compaction`  to merges several sstables into one new sstable which contains only the live data from the input sstables. Compation helps LSM-tree to recycle space and reduce read amplification.  There are two kinds of `compaction strategy` which is `Size-Tiered compaction strategy` (STCS) and `Level-Based compaction strategy` (LBCS). The idea behind STCS is compacting small sstables into medium sstables when LSM-tree has enough small sstables and compacting medium sstables into large sstables when LSM-tree has enough medium sstables. The idea of LBCS is to organize data into levels which contains one sorted run.

This section is going to discuss about the write amplification and read amplification of B+tree and Level-Based LSM-tree. 

### B+ Tree

In the B+ tree, copies of the keys are stored in the internal nodes; the keys and records are stored in leaves; in addition, a leaf node may include a pointer to the next leaf node to speed sequential access.

To simplify the analysis, let's say that the block size of the tree as `B`, and assumes that keys, pointers, and records are constant sized, so that each internal node contains `O(B)` children and each leaf contains `O(B)` data records. (The root node is a special case, and can be nearly empty in some situations.) Under all these assumptions, the depth of a B tree is 
$$
O(log_BN/B)
$$
where `N` is the size of database.

#### Write Amplification

For the worst-case insertion workloads, every insertion requires writing the leaf block containing the record, so the write amplification is `B`.

#### Read Amplification

The number of I/O's for any query is at most `O(logB N/B)` which is the depth of the tree. 

### Level-Based LSM-tree

In level-based LSM-tree, data is organized into levels. Each level contains one run. Data starts in level 0, then gets merged into the level 1 run. Eventually the level 1 run is merged into the level 2 run, and so forth. Each level is constrained in its sizes. Growth factor `k`  is specified as the magnification of data size at each level 
$$
level_i = level_{i-1}*k
$$
We can analyze level-based LSM-tree as follows. If the growth factor is `k` and the smallest level is a single file of size `B`, then the number of levels is 

$$
Θ(log_kN/B)
$$

#### Write Amplification

Data must be moved out of each level once, but data from a given level is merged repeatedly with data from the previous level. On average, after being first written into a level, each data item is remerged back into the same level about `k/2` times. So the total write amplification is 
$$
Θ(k*log_kN/B)
$$

#### Read Amplification

To perform a short range query in the cold cache case, we must perform a binary search
on each of the levels.

For the highest level i, the data size is O(N), so that it performs O(log N/B)disk reads. 
For the previous level i - 1, the data size is is O(N/k), so that it performs O(log(N/(kB))disk reads. 
For level i - 2, the data size is O(N/k2), so that it performs O(log(N/k 2B)disk reads.
…
For level i - n, the data size is O(N/kn), so that it performs O(log(N/k nB)disk reads.

So that the total number disk reads is 
$$
R=O(logN/B)+O(log(N/(kB))+O(log(N/k^2B)+...+O(log(N/k^nB)+1=O((log^2N/B)/logk)
$$

# Summary

The following table shows the summary of various kinds of amplification

|    Data Structure    | Write Amplification | Read Amplification |
| :------------------: | :-----------------: | :----------------: |
|       B+ tree        |        Θ(B)         |     O(logBN/B)     |
| Level-Based LSM-tree |     Θ(klogkN/B)     | Θ((log2N/B)/logk)  |

Through comparing various kinds of amplification between B+ tree and Level-based LSM-tree, We can come to conclusion that Level-based LSM-tree has a better write performance than B+ tree while its read performance is not as good as B+ tree. The main purpose for TiKV to use LSM-tree instead of B-tree as its underlying storage engine is because using cache technology to promote read performance is much easier than promote write performance.