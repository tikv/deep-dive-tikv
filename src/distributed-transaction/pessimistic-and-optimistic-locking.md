# Pessimistic & Optimistic locking

There are mechanisms employed to manage the actions of multiple concurrent users on a database—the purpose is to prevent lost updates and dirty reads. The two types of locking are pessimistic locking and optimistic locking.

## Pessimistic locking

Pessimistic locking: a user who reads a record with the intention of updating it places an exclusive lock on the record to prevent other users from manipulating it. This means no one else can manipulate that record until the user releases the lock. The downside is that users can be locked out for a very long time, thereby slowing the overall system response and causing frustration.

Where to use pessimistic locking: this is mainly used in environments where data-contention (the degree of users request to the database system at any one time) is heavy; where the cost of protecting data through locks is less than the cost of rolling back transactions, if concurrency conflicts occur. Pessimistic concurrency is best implemented when lock times will be short, as in programmatic processing of records. Pessimistic concurrency requires a persistent connection to the database and is not a scalable option when users are interacting with data, because records might be locked for relatively large periods of time. It is not appropriate for use in Web application development.

## Optimistic locking

Optimistic locking: this allows multiple concurrent users access to the database whilst the system keeps a copy of the initial-read made by each user. When a user wants to update a record, the application determines whether another user has changed the record since it was last read. The application does this by comparing the initial-read held in memory to the database record to verify any changes made to the record. Any discrepancies between the initial-read and the database record violates concurrency rules and hence causes the system to disregard any update request. An error message is generated and the user is asked to start the update process again. It improves database performance by reducing the amount of locking required, thereby reducing the load on the database server. It works efficiently with tables that require limited updates since no users are locked out. However, some updates may fail. The downside is constant update failures due to high volumes of update requests from multiple concurrent users - it can be frustrating for users.

Where to use optimistic locking: this is appropriate in environments where there is low contention for data, or where read-only access to data is required. 

## Practice in TiKV

We use Percolator Transaction model in TiKV, and this model uses optimistic locking strategy. Read database values, and tentatively write changes at first. And then check whether other transactions have modified data that this transaction has used (read or written). This includes transactions that completed after this transaction's start time, and optionally, transactions that are still active at validation time. If there is no conflict, make all changes take effect. If there is a conflict, resolve it, typically by aborting the transaction, in TiKV, we use a new timestamp to restart the whole transaction.
