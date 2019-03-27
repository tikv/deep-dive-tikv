# Pessimistic & Optimistic Locking

To prevent lost updates and dirty reads, locking is employed to manage the actions of multiple concurrent users on a database. The two types of locking are pessimistic locking and optimistic locking.

## Pessimistic Locking

A user who reads a record with the intention of updating it places an exclusive lock on the record to prevent other users from manipulating it. This means no one else can manipulate that record until the user releases the lock. The downside is that users can be locked out for a very long time, thereby slowing the overall system response and causing frustration.

Pessimistic locking is mainly used in environments where write contention is heavy, where the cost of protecting data through locks is less than the cost of rolling back transactions if concurrency conflicts occur. Pessimistic concurrency is best implemented when lock times will be short, as in programmatic processing of records. Pessimistic concurrency requires a persistent connection to the database and is not a scalable option when users are interacting with data, because records might be locked for relatively large periods of time. It is not appropriate for use in web application development.

## Optimistic Locking

This allows multiple concurrent users access to the database whilst the system keeps a copy of the initial-read made by each user. When a user wants to update a record, the application first determines whether another user has changed the record since it was last read. The application does this by comparing the initial-read held in memory to the database record to verify any changes made to the record. Any discrepancies between the initial-read and the database record violates concurrency rules and causes the system to disregard any update request; an error message is reported and the user must start the update process again. It improves database performance by reducing the amount of locking required, thereby reducing the load on the database server. It works efficiently with tables that require limited updates since no users are locked out. However, some updates may fail. The downside is constant update failures due to high volumes of update requests from multiple concurrent users - it can be frustrating for users.

Optimistic locking is appropriate in environments where there is low write contention for data, or where read-only access to data is required.

## Practice in TiKV

We use the Percolator Transaction model in TiKV, which uses an optimistic locking strategy. It reads database values, writes them tentatively, then checks whether other transactions have modified data that this transaction has used (read or written). This includes transactions that completed after this transaction's start time, and optionally, transactions that are still active at validation time. If there is no conflict, all changes take effect. If there is a conflict it is resolved, typically by aborting the transaction. In TiKV, the whole transaction is restarted with a new timestamp.
