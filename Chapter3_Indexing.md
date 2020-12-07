# CPSC 404 Chapter 3: Indexing Fundamentals
## Motivation
Suppose the following query is to be performed:
    
    SELECT M.moviename
    FROM Movie M
    WHERE M.rating > 3

The database is on disk, and the table Movie is huge. Without other information, to respond to this query, the database has to read all of the pages of table Movie into the memory and examing each record one by one. As discussed previously, disk read can take considerable amount of time, For huge tables, response for the query will be too slow. Is it possible to not read in all pages of the table but still return the correct result?

**Indexing** helps. **Index** is specially designed node which has keys and pointers to other indicies or data records. The index directly pointing to data records is also called **Data Entry (DE)**. By arranging the indicies in certain pattern to form structure with good properties, they can guide the database to quickly locate the data records and only read in the necessary pages.  The file containing the indeices is called **Index File**, which is stored on disk when not used. 

In this courses, there are two indexing methods:

- **tree-structured index** (Chapter 4)
- **Hashing index** (Chapter 5)

The take-away idea for here is that index help improve the performance of database. The reasons will be introduced when specific index is discussed.




## Fundamental Concepts
Suppose there is such relational schema: Student(<ins>sid</ins>, name, YOB). From the schema, it is obvious that the rid is the primary key. 
### Record ID
**Record id (rid)** is the pointer to the location of record. It is usually in the form (Disk Page Address, slot number of record in the page).
### Data Entries
As a special kind of index, **Data entry (DE)** is a pair of key and the method to locate record. For a key value $k$, we denote its corresponding data entry (DE) as $k^*$. The key $k$ may be the primary key or other attribute of the record. For any index, there are 3 alternatives for DEs.

NOTE: the key $k$ is the key of the DE, not the "key" in "primary key".

#### Alt 1: Data record with key value $k$:
The record itself can be an DE because it contains the key $k$ and the method to locate a record: the entire content of the record. This alternative is used mainly when $k$ is the premary key. A DE for one example record of the example schema is:

    <314, "George", 1999>

#### Alt 2: <$k$, rid of data record with chosen attribute valued $k$>
This alternative contains the key $k$ and the rid of the data record with the chosen attribute valued $k$. As discussed above, rid is the location of the record on disk. This alternative works for both primary key and other attributes. Some DEs for several example records of the example schema are:

    <YOB = 1999, rid1>
    <YOB = 1999, rid2>
    <YOB = 1999, rid3>

#### Alt 3: <$k$, list of rids of data records with chosen attribute valued $k$>
This alternative contains the key $k$ and a list of rids of data records with the chosen attribute valued $k$. This list of rids can be used to locate the records. This alternative mainly works for non-key attributes. An example DE for the example schema is:

    <YOB = 1999, [rid1, rid2, rid3]>

### Clustered Index
**Clustered Index** means the data records on disk are physically sorted with respect to indexed attribute $A$. That is, the indices next to each other point to records next to each other. Otherwise, the index is non-clustered. Clustered index is prefered because when doing range search based on clustered index, sequential accesses incurs. However, if the index is unclustered, accessing records based on that index may require one random access per record in the worst case. 

 