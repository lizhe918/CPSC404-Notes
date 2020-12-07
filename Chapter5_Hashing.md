# Chapter 5: Hash-Based Indexes
## Overview
**Hash-Based Indexes** are best for equality selections because there is no traversal, and the location of DE $k^*$ is directly computed geven key value $k$. However, it does not support range searches. There are mainly two techniques for hash-based indexes:

- Static Hashing: the index file does not change or does not change much.
- Dynamic hashing: the index file undergo many and frequent updates.

## Static Hashing
Static hashing is similar to the hash table in previous courses with the slot turned to bucket. In static hashing, the number of **primary bucket**, $M$, is fixed. Primary bucket pages are allocated sequentially and never get deallocated. The hash function $h(k)$ is used to determine which bucket a DE should go. It works on search key field of record $r$, and distributes result values over range: $[0, M-1]$. Bucket size is fixed and can be more than 1 block large, containing data entries. **Overflow pages** may be used if there are more DEs with the same hash value than a primary bucket can contain. Overflow pages are pointed by the primary bucket, and they point later overflow pages if there is any. 

However, long overflow chains can develop and degrade performance. So dynamic hashing techniques can fix this problem.

## Dynamic Hashing
### Extendible Hashing
**Extendible Hashing** intends to deal with the overflow pages by *reorganizing* or *rehashing* the file. It employs a directory of pointers to buckets, in which the DEs live. The pointer in the $i$th slot of the directory points to the bucket $i$. When a bucket is full, the *logical* number of buckets is doubled by doubling the directory and splitting the bucket that overflowed (detail below). Directory is small, so the cost is relatively low. Extendible hashing guarantees there is no overflow page. The idea is simple, but the trick lies in how hash function is adjusted. 

Before the detailed discussion about the mechanism, a few important concepts should be introduced:

- Depth: denotes how many bits from the hash address suffix should be examined at a given time.
- Global Depth: the minimum number of bits needed to correctly find the home bucket for an arbitrary DE in general.
- Local Depth of Bucket B: the number of bits that is *really* needed to be looked at to get to B.

Suppose the element $e$ is to be inserted, and $h(e)$ assigns it to a full bucket $A$ whose local depth is $D_A$:
    
1. Split $A$ into two buckets $A$ and $A_2$ using $D_A + 1$ bits.
2. Double the directory.
3. Add 1 to local depth of $A$ and $A_2$.
4. Add 1 to global depth.
5. Point free pointers in the doubled directory to buckets based on their local depths.

**NOTE**:
- Other buckets stay impact from the insertion of $e$ and split of $A$. 
- Multiple pointers may point to the same bucket for some of the buckets. 

For deleting elements, just delete it and do nothing more to leave room for future insertions. There is a more complex deleting algorithm with merging pages, but it is rarely done in practice. 

Extendible hashing is fantastic. However, it maintains a directory, and it can grow exponentially with skewed data. Is there a method that grows slower while avoiding the long overflow chain?

### Linear Hashing
**Linear Hashing** is conceptually an extension to extendible hushing. It grows slowlier and does not keep a directory. To achieve this goal, linear hashing exploits a family of hash functions, allows overflow pages, and chooses the bucket to split based on the *round robin* splitting logic. 

Suppose initially there are $N$ buckets where $N = 2^{p_0}$ for some $p$. 

Linear hashing uses a family of hash functions $h_0$, $h_1$, $h_2$, ... Here, $h$ is some hash function that produces result in $[0, N-1]$. $h_i$ consists of applying $h$ and looking at the last $p_i$ bits where $p_i = p_0 + i$. Obviously, $h_{i+1}$ doubles the range of $h_i$. Linear hashing also allows overflow page. 

Overflow page is kept in check by splitting pages, which are chosen based on a rule of *round robin*. The *round robin* splitting logic keeps a *next* pointer which points to the next candidate bucket for being split. Splitting proceeds in *rounds*. At the beginning of round $R$, there are buckets $0$ to $N_R - 1$. Notice that buckets $0$ to $next - 1$ have been split; while buckets $next$ to $N_R - 1$ have yet to be split. In this round, $h_R$ is used for insertion.

When one bucket overflows, the inserted element is placed into the overflow page of $h_R(e)$. Possibly, the split of the bucket pointed by *next* occurs. The split trigger can be defined based on needs. When splitting, DEs are assigned to new buckets based on $h_{R+1}(e)$, and then *next* is incremented by 1. The new bucket is attached at the end as **split image**. 

To search an element, use $h_R(k)$ to identify the location. However, if $h_R(k) < next$, $h_{R+1}(k)$ is used to determine if the data entry should be assigned to split image. 

When deleting elements, if the bucket of the element is empty, destroy the bucket and decrement *next* by 1.

Round $R$ ends when all its $N_R$ buckets have been split. After the round ends, the *next* goes back to $0$.  

**NOTE**:  rule of *round robin* may not choose the bucket that is overflowed. 