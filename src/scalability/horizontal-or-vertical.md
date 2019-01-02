# Horizontal or vertical

Methods of adding more resources for a particular application fall into two broad categories: horizontal and vertical scaling.

## Horizontal Scaling

Horizontal scaling, which is also known as scaling out, means to add more machines to a system and distribute the load across multiple smaller machines. As computer prices have dropped and performance continues to increase, high-performance computing applications have adopted low-cost commodity systems for tasks. System architects may configure hundreds of small computers in a cluster to obtain aggregate computing power that often exceeds that of computers based on a single traditional processor. The development of high-performance interconnects such as [Gigabit Ethernet](https://en.wikipedia.org/wiki/Gigabit_Ethernet), [InfiniBand](https://en.wikipedia.org/wiki/InfiniBand) further fueled this model. 

## Vertical Scaling

Vertical scaling, which is also known as scaling up, means adding resources to a single node in a system, typically involving the addition of CPUs or memory to a more powerful single computer. Such vertical scaling of existing systems also enables them to use virtualization technology more effectively, as it provides more resources for the hosted set of the operating system and application modules to share. 

## Tradeoff

There are tradeoffs between the above two models. Larger numbers of computers mean increased management complexity, as well as a more complex programming model and issues such as throughput and latency between nodes. A light workload running on scaled-out systems maybe is even slower than on a single machine due to communication overhead. But the problem with a scale-up approach is that the performance doesn't grow in linearly proportional to cost. A system that runs on a single machine is often simpler, but high-end machines can become very expensive, so most intensive workloads cannot avoid scaling out.
