# Chapter 4: Tree-Structured Indexing and B+Tree
## Tree-Structured Indexing
**Tree-structured indexing** supports both equality and range search. When performing equality or range search, directly doing binary search on all of the data records is not wise because that will lead to too many disk accesses. So, index file containting indices on the search key is created. 

One index node can tell information about many records. For example, a node may contain 2 keys and 3 pointers: 
- the left pointer points to a page of records whose keys are smaller than the left key of the node
- the middle pointer points to a page of records whose keys are larger than the left key but smaller than the right key of the node
- the right pointer points to a page of records whose keys are larger than than the right key

In this case, 1 index node is in charge of 3 data pages. With more nodes, all data pages will be indexed. The index file is smaller than the entire database, so doing binary search on index file is less expensive.

As introduced previously, index file is also stored on disk, so directly performing binary search on index file may still be too expensive if the index file is still too large. In this case, recursively index the index file to improve the search efficiency, leading to a tree structured index. 

## B+Tree Basics
### Structure
In **B+Tree**, each index node is as large as one disk page, containing at most $m$ keys and $m+1$ pointers. One **index entry** consists of 1 key and 2 pointers. Theses pointers of index entry point to children of this node: a DE page or an index page. Between DE pages, there are pointers to each other. 

The B+Tree has many properties:
- All entries of the subtree pointed by the pointer left to a key $k$ have key $< k$.
- All entries of the subtree pointed by the pointer right to a key $k$ have key $\ge k$.
- Insert/delete at cost $O(log(B))$ where $B$ is the number of DE pages (detail in later part).
- Except root, each node contains $d \leq m \leq 2d$ (including DE page). $d$ is called the order of the tree. 
- The height is balanced. 
- B+Tree supports equality and range-seasrches.

### Search in B+Tree
The search of in B+Tree is very fast, costing only $O(h)$ where $h$ is the height of the tree because the height is balanced. 
        
    The search begins at root. 
    While there is still next level:
        Key comparisons occur in each internal node via binary search
        Identifying one unique child to follow.
    If the DE with the required key is found:
        return the records of the DE

### B+Tree in Practice
Typically, the order is 100 ($d = 100$), and this means for each node: $100 \leq n_{entry} \leq 200$. With such a $d$, a 5 level (height 4) B+tree can accommodate $2.352$ TB data. And usually the tree is not totally filled. The meansurement for the filled proportion is called **fill factor**, and is calculated using $fill\ factor = n_{entry} / 2d$. In majority of the cases, The average fill factor is 66.5%. 

It is required to be able to calculate the height of the tree given disk information, $d$ and the fill factor. Suppose a node is implemented as a block, and the size of the block is $b$ byte. A key is $k$ bytes, and a pointer is $p$ byte. The data entry file occupies $B$ blocks. Given this information, what is the max/min height of a B+Tree index?

The max number of (key, pointer) pairs per page is $n_{kpmax}\lfloor b / (k + p) \rfloor$.

$$d = \lfloor \frac{n_{kpmax}}{2}\rfloor$$

To compute the maximum height, the fill factor of every internal note is assumed to be 50% and there is only 1 entry in the root. That is, there are $d$ entries in each internal node. In this case:

$$h_{max} = \lceil log_d(B)\rceil + 1$$

To compute the minimum height, the fill factor of every internal node is assumed to be 100% except the root. That is, there are $2d$ entries in each internal node. In this case:

$$h_{min} = \lceil log_{2d}(B) \rceil$$

In addition to master the fomulas above, it is also required to be able to verify the fomula by computing the number of index page for each levels of the tree using a bottom-up approach.

## Insertion & Delation of B+Tree
### Insertion Algorithm
    Find correct leaf L
    Put DE onto L
    If L has enough space:
        then success
    Else:
        Redistribute entries evenly into L and a new node L2
        Copy up the middle key
        Insert index entry pointing to L2 into the parent of L

The above algorithm can happen recursively when there is no enough space in the parent of L. Attention: L and L2 are DE pages, while the parent of L is index page. When the index page is splitted, unlike the *copy up* of the middle key for DE pages, the middle key is *bumped up*. Split of node grows the tree in width, and the root split increases height. This algorithm enforces all the properties and restriction of B+Tree.

### Deletion Algorithm
    Find the leaf L where the DE belongs
    Remove the entry
    If L has at least d entries:
        then success
    Else:
        Try redistribution:
            Borrow from adjacent node with same parent as L
            Copy up the new middle key
        Catch if redistribution fails:
            Merge L and sibling
            Delete the index entry between the pointer to L and its sibling in the parent of L

The above algorithm can happen recursively when there is any page is no longer half-filled after deletion. 

### Analysis of Insertion/Deletion Time
In the best case, insertion/deletion just happens after finding the leaf L, so $O(1)$. However, in the worst case, insertion/deletion requires action at every level of the tree, and it may case the tree to increase/decrease one level. Therefore, the worst case is $O(h)$ or $O(log(B))$.

## B+Tree Optimization
### Prefix Key Compression
Key values in index entries only serve the role of guiding search. So they can be compressed to only their prefix. While compressing, the properties of B+Tree should still hold. Compressing long string valued keys to only the prefix of the keys in index entries can help pack more index entries per page. Therefore, there will be more children per nod, and finally lead to a shorter B+Tree.

### Bulk Loading
Suppose there are a huge amount of data and a B+Tree indexing is needed. Building the tree via repetitive inserting can be too slow. The efficient way to build the tree is to perform **Bulk Loading**. 

First sort all data entries and creat a index page. Then point the left most pointer to the first DE page, and point the second left most pointer to the second DE page, filling the index entry with the first DE of the second DE page. Then continuously points pointers to later DE pages and fill index entries. When the index page is full, split it so that the height grows and there are new places for further DE pages. Because the DE pages are sorted, only root and the right most index page will be modified. Other index pages can be wrote to disk while building rest of B+Tree index. 

Bulk loading is much faster than repetitive insertion. Also, it creates a tree whose leaves are stored sequentially, sorted, and therefore clustered.