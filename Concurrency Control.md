# Transaction Management of Database: Concurrency Control
## Transactions
### Fundamentals
Databases(DB) are usually not static: INSERT, DELETE, and UPDATE in SQL are frequently used to update the data in database. For these updates, a dababase management system(DBMS) is *ONLY* concerned about what data is read/written from/to the database.  So, operations like:

    Transfer $100 from account A to B;
    Transfer $200 from account A to B;

may both be regarded as a sequence of reads and writes: 
    
    read(A); read(B); write(A); write(B)

This sequence is expected to be executed as a unit. That is, either all of the reads and writes in one sequence is executed or none is executed. Such a unit of read/write sequence is called **transaction**. In many cases, at one moment, there are many transactions to be executed, and A DBMS schedules these transactions by interleaving read/write actions of different transactions. Some ways of interleaving are sound; while others are not. To help reason about correctness of schedules more easily, the details like $100 and $200 are stripped away. 

Interleaving the actions of transactions means concurrent execution of user program, and it is essential to good DBMS performance. It is well-known that disk accesses are frequent for database and relatively slow. Without concurrency, CPU will stays largely idle waiting for disk accesses.

### Transaction States
- **Active**: makes progress or waits for resources.
- **Failed**: normal execution cannot continue and evetually aboted. May occur because of:
  - run-time error
  - system crash
  - cancellation by the user
- **Aborted**: after rolling back the DB to the state prior to the execution of the transaction.
  - may be restarted as a new transaction (system failures)
  - may be killed (internal logical errors)
- **Committed**: after successful completion of a *commit* command.

### Transaction Properties (ACID)
- **Atomicity**: either all or none of the operations of one transaction are completed.
  - System has to undo the actions of aborted transactions. 
- **Consistency**: the DB state is consistent after the transaction.
  - Transfer from A to B: A+B after = A+B before
  - Controlled by the user as DBMS does not understnad the semantics of the data.
- **Isolation**: concurrent transactions must not interfere with each other.
  - DBMS must perform concurrency control.
  - Interleaved transactions should have same result as serially performed transactions. 
- **Durability**: changes from successful transactions must persist through failtuers.
  - Once committed, the DB data has been changed.
  - System has to redo the actions of committed transactions after crash.
  - Keep enough information on the disk to recover.

NOTE: After executing consistent transactions in isolation, the DB is still consistent.

## Schedules
### Fundamentals
**Schedule** is a time-ordered sequence of the actions for one or more transactions such that actions from transaction T appear in the schedule in the same order that they do in T. When designing schedules, only read/write actions and commit/abort states are of interest, and it is assumed that:
  - Transactions only interact with the DB and don't interact directly with each other.
  - A DB is a fixed collection of independent objects. 

Schedules may be **concurrent**, interleaving actions of two or more transactions. Scheduels can also be **serial**, executing actions of one transaction together. Schedules may be **equivalent** when two or more schedules leads the same result. Schedules may be **serializable** when a concurrent schedule is equivalent to some serial schedule. 

For schedule design, it is required that after executing all transactions, no matter what schedule is used, the database should stay consistent. Clearly, when all transactions preverve consistency, every serial and serializable schedule preserves consistency. So, serializable schedule is the goal of schedule design.

### Anomalies with Concurrent Execution
It is very difficult to check that a concurrent schedule is serializable. To make the design easier, a stronger notion of serializability is introduced: **conflict serializablility**. Two actions **conflict** with each other if both act on the same data object and one of them is a write. There are three conflict types:
- **Write-Read(WR) Conflict (dirty reads)**:
  - Reading uncommitted data
  - T1 writes data A, then T2 reads and uses data A which are written by T1 before T1 committs.
  - Issue: if T1 aborts later, T2 has used the wrong data.

- **Read-Write(RW) conflict:**
  - Unrepeatable reads
  - T1 reads and calculates on A, then T2 changes the value of A
  - Issue: if T1 reads A again after T2 changes A and calculates on the new data, T1 will get a different result. 
  
- **Write-Write(WW) conflict (blind write)**:
  - Overwritting uncommitted data.
  - T1 write A, then T2 writes A before T1 commits.
  - Issue: T1 and T2 interfere with each other. 

Beside the definitions and issues above, it is very important to notice and understand that:
- swapping two conflicting actions changes the results of the schedule.
- swapping two non-conflicting actions does not change the results of the schedule.

Two schedules are **conflict equivalent** if every pair of conflicting actions is ordered the same way. Schedules S is **conflict serializable** if S is conflict equivalent to some serial schedule. 

### Serializablity
A conflict serializable schedule can be converted into a serial schedule by swaping adjacent non-conflicting actions. To check if a schedule is conflict serializable, a trick called **precedence graph** is used. In a precedence graph, there is one node per transaction, and there is an edge from $T_i$ to $T_j$ if:
- $T_i$ executes write(A) before $T_j$ executes read(A) or write(A)
- $T_i$ executes read(A) before $T_j$ executes write(A)

A schedule is conflict serializable if and only if its precedence graph is acyclic.

## Lock-Based Concurrency Control
### Fundamentals
To ensure serializability, a DBMS uses **locks**, which are structures that restrict access to certain DB objects. There are two kinds of locks:
- **Shared Locks(S)**: can be shared by many transactions
- **Exclusive Locks(X)**: can only be held by a single transaction at a time. 

For each transaction, it must request and obtain an S or X lock on an object before reading it, and an X lock on an object before writing it. A transaction holding a shared lock can **upgrade** it to an exclusive lock. Similarly, a transaction holding an exclusive lock can **downgrade** it to a shared lock. Locks held by a transaction may be unlocked/released. When a transaction T waits for a lock, it is **blocked**.

### Locking Protocols
DBMS uses **locking protocols** to define the guideline of how locks are used. Different locking protocols has different features. In this course, there are two kinds of locking protocols:
- **Strict Two-Phase Locking Protocal (Strict 2PL)**
  - All locks held by a transaction are released when the transaction completes (commits or aborts)
  - Strict 2PL allows only some conflict serializable schedules (very limited number of situations).
  - Too strict for many cases. 
  - Transactions are ordered by the order they commit or abort.
- **Two-Phase Locking Protocal(2PL)**
  - A transaction can release locks any time, but it cannot request additional locks once it releases any lock.
  - Each transaction has two phases in a 2PL schedule: **growing** and **shrinking phase**.
  - 2PL also produces only conflict serializable schedules, but works in many more situations than Strict 2PL does.
  - lock upgrade and downgrade can happen in the growing phase.
  - Transactions ordered by the order of their entrance of the shrinking phase.
  
### Lock Management
The lock and unlock requests are handled by the **lock manager**. The lock manager maintains two tables:
- lock table:
  - hash table with one entry for each data object
  - Each entry contains 
    - the number of transaction holding a lock
    - type of lock held
    - pointer to queue of lock requests.
- Transaction Table: for each transaction contains a list of locks held by it.

When locking and unlocking, the lock and transaction tables must be updated as a unit. 

### Deadlock
A **deadlock** occurs when 2 more more transactions are blocked, waiting for each other to release the locks. It is very important to notice that 2PL can produce deadlocks. To detect deadlocks, a trick called **waits-for graph** is used. In waits-for graph, there is one node per transaction. There is an edge from $T_i$ to $T_j$ if $T_i$ is waiting for $T_j$ to release a lock. A deadlock is present if and only the waits-for graph has a cycle. 

With the waits-for graph, lock manager adds edges when queueing a lock request, and removes edges when granting a lock requet. It periodically check for cycles in the waits-for graph. 

To resolve deadlock, a transaction in the cycle is picked and aborted. The choice of transaction is made based on needs.  

