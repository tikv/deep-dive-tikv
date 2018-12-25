# Introduction

TiKV is a distributed transactional key-value database, transaction is a foundational function for TiKV. This chapter we will talk about general implementations about distributed transaction and some implementation details in TiKV.

A database transaction, by definition, must be atomic, consistent, isolated and durable. Database practitioners often refer to these properties of database transactions using the acronym ACID.

Transactions provide an "all-or-nothing" proposition, stating that each work-unit performed in a database must either complete in its entirety or have no effect whatsoever. Further, the system must isolate each transaction from other transactions, results must conform to existing constraints in the database, and transactions that complete successfully must get written to durable storage.

A distributed transaction is a database transaction in which two or more network hosts are involved. Usually, hosts provide transactional resources, while the transaction manager is responsible for creating and managing a global transaction that encompasses all operations against such resources. Distributed transactions, as any other transactions, must have all four ACID properties.

A common algorithm for ensuring correct completion of a distributed transaction is the two-phase commit (2PC).

TiKV adopts google Percolator transaction model, a variant of 2PC.