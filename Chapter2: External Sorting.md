# External Sorting
## Fundamentals
### Why External Sorting
Database usually deals with huge amount of data, and the data may not be able to fit into the RAM. Therefore, sorting algorithms from previous CPSC courses cannot be used for database as they all take place in RAM. So disk is needed for help. The sorting algorithm using disk is called **external sorting**.

### I/O Computation Model and Time
Since disk is involved in external sorting, READ and WRITE are involved too. They are both very expensive operations. Specifically, random accesses are even more expensive. Disk I/Os takes so much time that the cost of in-memory operation can be out of consideration. So the reasonable cost model for computation using secondary sotrage in general (external sorting in this case) is to count only disk I/Os with the distinction between random and sequential I/Os. This model is called **I/O Computation Model**.

### Guidelines for Designing External Sorting Algorithm
To design good external sorting algorithm, the following guidelines should be carely considered:
- Overlap I/O and CPU processing as much as possible
- Use as much data from each page read as possible.
- Cluster records that are accessed together in consecutive pages to reduce random access.
- Buffer frequently accessed blocks in memory to reduce repeated disk accesses.

## Multiway External Mergesort
The external sorting algorithm in this course is the **Multiway External Mergesort** algorithm. It is important not only because it does a good job for sorting, but also because many furture algorithms are very similar to the Multiway External Mergesort algorithm. Detailed understanding of each step and the ability to calculate the time cost are required.

### 2-Way External Mergesort
Suppose RAM can contain $B$ pages of data, and there are $m \times B$ pages of data in total.
- Sort Phase

        while there is data left:
            Read in B pages of data from disk
            Sort them in memory with some in-memory sorting algorithm
            Write the sorted sublists (SSL) of B pages to temp disk space.

So, $m$ sorted sublists of $B$ pages each is produced.

- Merge Phase
        
        Pass 1: produces m/2 SSLs of 2B pages each by merging.
        Pass 2: m/4 SSLs of 4B pages each by merging.
        ...
        continue until there is only 1 SSL of m*B pages is produced.

Currently, with $n$ records, $O(log_(n))$ passes is needed. This is because:
$$O(log(m)) = O(log(\frac{n \times record \ size}{B}) = O(log(Const \times n)) = O(log(n))$$
So, each record is read/written from disk $O(log(n))$ times. It is not good enough since $n$ can be really big.
 
### 2-Phase, Multiway Merge Sort (2PMMS)
In fact, with certain restrictions, it is possible. **2PMMS** is an optimized version of above algorithm and requires only 2 reads + 2 writes per block. This is achieved by making the merge phase only 1 pass. 

- Same Sorting Phase
        
        while there is data left:
            Read in B pages of data from disk
            Sort them in memory with some in-memory sorting algorithm
            Write the sorted sublists (SSL) of B pages to temp disk space.

So, $m$ sorted sublists of $B$ pages each is produced.

- Merge Phase
        
        divide RAM into equally sized buffers
        use one buffer for each of the SSL as input buffers and one buffer for output
        load input buffers with the first blocks of their respective SSLs
        merge m buffers simultaneously and put the merged result into the output buffer:
            merge as usual
            if output buffer is full:
                write (flush) the output buffer to disk
            if one buffer exhausts:
                load the next block of this SSL to this buffer
     
This algorithm finishes with 2 reads and 2 writes if and only if:
$$\lceil{\frac{n \times record \ size}{B}}\rceil < B - 1$$
If this requirement is not satisfied, then more passes are needed and more time will be required. With **cylindrification** and **prefetching** discussed below, if more than 1 pass is needed for phase 2, the approximate total time for phase 2 will be $(number \ of \ pass \times \ time \ per \ pass)$.

### Cylindrification, Prefetching, and Double Buffering
There are some tricks to optimize the algorithm even better:

- Cylindrification:
  
  Grouping blocks by cylinder can reduce random access, thus making the seek and rotational latency negligible for Phase 1.

- Prefetching
  
  On top of cylindrification, prefetching additional pages to RAM minimizes the seek and rotational latency for phase 2 when buffer size is larger than 1 page.

- Double Buffering
    
    Overlaping I/O and CPU processing is possible in this case. Divide each buffer in phase 2 into 2 equally sized buffers. For each pair of buffers, only one is active at a time. For input buffers, when the first buffer exhausts, the second buffer is actived and used for merging, and at the same time the first buffer starts to refill so that the the waiting time for refill is reduced. The same thing happens for the output buffer. 

