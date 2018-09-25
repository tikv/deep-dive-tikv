# Consistency, Availability, & Partitioning

In 2000, Eric Brewer presented [“Towards Robust Distributed Systems”](http://awoc.wolski.fi/dlib/big-data/Brewer_podc_keynote_2000.pdf) which detailed the CAP Theorem. Succinctly, the theorem declares that a system may only choose two of the following three attributes:

* **Consistency:** Every read receives the most recent write or an error
* **Availability:** Every request receives a (non-error) response – without necessarily having the most recent write
* **Partitioning:** The system continues to operate despite an arbitrary number of messages being dropped (or delayed) between nodes

If we compared a traditional RDBMS, like PostgreSQL, which provides [ACID](http://jimgray.azurewebsites.net/papers/thetransactionconcept.pdf) guarantees favors consistency over availability. BASE (Basic Availability, soft-state, eventual consistency)  systems, like MongoDB, choose availability over consistency.

In 2012, Daniel Abadi proposed that CAP was not sufficient to describe the trade-offs which occur when choosing the attributes of a distributed system. They described an expanded [PACELC](http://cs-www.cs.yale.edu/homes/dna/papers/abadi-pacelc.pdf) Theorem:

> Availability (A) and consistency (C) (as per the CAP theorem), but else (E), even when the system is running normally in the absence of partitions, one has to choose between latency (L) and consistency (C).

In order to support greater availability of data, most systems will replicate data between multiple peers. The system may also replicate a write ahead log offsite. In order to fulfill these availability guarantees the system must ensure a certain number of replications have occured before confirming an action. **More replication means more consistency and more latency.**
