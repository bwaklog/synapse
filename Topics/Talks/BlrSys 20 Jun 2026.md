# Optimising Data Ingestion in Apache Pinot
- https://docs.pinot.apache.org/architecture-and-concepts/concepts/architecture
- Segments are self contained files, unit of storage, replication and printing
	- Inverted/range/bloom 
	- Forward index + dictionary
	- Contains metadata for each column in segment, cardinality, etc. 
- multilevel pruning at query time. Have a broker pruning to reduce fanout to reduce tail latency. Rely on partial meta data. 
- We also have server pruning
- Have raw file input and need to process it to pinot format
- reading row by row would be inefficient. Large number of random IO, slows down page ingestion. Row by row ingestion will not let us parallelise segment building 
- Move to a columnar input, output is columnar. Column by column processing. 
- Input file, we read selected columns. Add column to the right partition. No intermediate storage here. All columns in partition is merged into a segment. Everything is in memory. 
- We don’t have to box into java objects, read and write as primitive types. At any point of time, we are reading a file sequentially and adding files to the right partition in memory. More efficient for large segment and wide column. 
- No disk IO bottleneck. 
- Sorting by sequential disk read and random memory reads. While reading input file we read sort and partition columns. This sort columns is a global list of document for each partition and what is the doc id mapping to map the order of ingestion. We read column sequentially, store in memory and do random reads of in memory (sort order) and then pass column to right partition. We random access column per partition in sort order. 
- Derived columns (columns that are a fuction of multiple columns). Having multiple derived column. Having multiple lookups for the same column. 
- What could be bad about column based
	- Dynamic segment size ingestion. Row based can but for column oriented, we need to pre determine the size. 
	- Narrow tables have no gains. It’s memory efficiency gains over IO is not visible L
- atomic visibility of data replacement. We don’t want intermittent data loss or duplicate results. Swap old segment set, with new ones automatically over segment lineage. The broker determines what detriment to query. The broker and controller coordinate what segment to retrieve. 
- Use watermarks for data reliability. Clearing data based on watermarks. Retries work on checkpoint and watermark to ensure we don’t duplicate our ingestion during failures. 

# Unwinding Stack Traces

What exactly is stack unwinding
```c
void b();
void a() {
	b();
}
int main() {
	a();
}
```

How do you maintain a trace of the functions that are called, this is done with the stack. `b -> a -> main -> ...`

ELF maintains the entry address of a function. Something like `_start`, this is the first function. `_start` ends up invoking dynamic linker functions. Lots of methods that are called just before `main`

How do you unwind the stack???

We already maintain a trace in a way - the return address of some function. 

`_fini` contains a pointer to final functions that are to be called. These are all the destructors. Same way we have a `_init` that has all the constructors. 

Each call has a link to the previous function that was the parent caller. We sort of have a linked list in memory

We can do a linked list frame walk because of frame pointers

But what to do with `-fomit-frame-pointer`. We wont have a similar stack trace. For example at the start of a function, preamble, we dont have any instruction wrt the frame pointer and pushing the LR to the frame pointers. We cant do our regular stack walking technique.


Within a section in elf, we have a section called `eframe`, there is something called `DWARF`, a debugging data format used by compilers. How do you do mapping between assembly from C and assembly. `.debug_info` and `.debug_line` is used. 

The `.eh_frame` section in the ELF file. A DWARF is essentially like a register virtual machine (think about SQLite - a bunch of compiled virtual machine instructions with registers). 

> We also have something called stack virtual machine

`libunwind` helps us do stack unwinding even with `-fomit-frame-pointer`.  Compiled languages use something called `libunwind`. 

- Stack walking: space and time trade-offs [maskray.me](https://maskray.me/blog/2025-10-26-stack-walking-space-and-time-trade-offs)
- Getting the call stack without a frame pointer [yosefk.com](https://yosefk.com/cgi-bin/comments.cgi?post=blog/getting-the-call-stack-without-a-frame-pointer)
- Stack unwinding [maskray.me](https://maskray.me/blog/2020-11-08-stack-unwinding)