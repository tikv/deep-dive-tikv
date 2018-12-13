# Introduction

In a distributed database environment, resource scheduling needs to meet the following requirements:
- Keeping data highly available: The scheduler needs to be able to manage data redundancy to keep the cluster available when some nodes fail.
- Balance server load: The scheduler needs to balance the load to prevent a single node from becoming a performance bottleneck for the entire system.
- Scalability: The scheduler needs to be able to scale to thousands of nodes.
- Fault tolerance: The scheduling process must not be stopped by the breaking down caused by a single node failure.

In the TiKV cluster, resource scheduling is done by the PD. In this chapter, we will first introduce the design of two scheduling systems (Kubernetes and Mesos), followed by the design and implementation of scheduler and placement in PD.
