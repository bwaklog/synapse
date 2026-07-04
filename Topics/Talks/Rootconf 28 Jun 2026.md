# Cascade — build a toy LSM Tree from scratch 
> Workshop by Anirudh Rowjee ([hasgeek](https://hasgeek.com/rootconf/build-a-toy-lsm-tree-from-scratch-workshop/))

- (part of write path if we have immut. mem tables) Immutable mem tables are swapped out with the current active mem table. The swap will change which table accepts writes while the other is prepared to be flushed to L0
- Rocks db does flushes inlines (sometimes) and uses compactions as a way of **throttling writes** in line
	- Rate Limiter in rocks db ([github.com/facebook/rocksdb](https://github.com/facebook/rocksdb/wiki/Rate-Limiter)) 
	- Compaction in rocks db ([github.com/facebook/rocksdb](https://github.com/facebook/rocksdb/wiki/Compaction))
	- Background Compactions in YugabyteDB: An Exploration of File-Level Optimization [yugabyte.com](https://www.yugabyte.com/blog/background-data-compaction/)
- The decisions on whether or not we do inline flushes is a matter of design decision on trade-offs of the write path
- The interface of the SSTable (Sorted String table) is that we are accepting writes in a sorted fashion. The implementation specifics of how this is encoded format can differ based on what optimisations on searching (ex - having some sort of serialised tree to give us a binary index natively on the data). The encoding format could be anything that obeys this interface
- Our formal measures for performance is on
	- Read amplification
	- Write amplification
- Space amplification
- amp can be thought of the how many ops (read/write) we would have to do for a given operation. Ratio of the bytes read/write for the read/write op that was specified. 
## Useful links
- The Log-Structured Merge-Tree (LSM-Tree) [cs.umb.edu](https://www.cs.umb.edu/~poneil/lsmtree.pdf)
- How do LSM Trees work? [rowjee.com](https://rowjee.com/blog/lsmtrees)
- Cascade repo [github.com/anirudhrowjee](https://github.com/anirudhRowjee/cascade)
- Google LevelDB [github.com/google](https://github.com/google/leveldb)
