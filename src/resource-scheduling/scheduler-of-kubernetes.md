# Scheduler of Kubernetes

## Overview

Kubernetes is a Docker-based open source container cluster management system initiated and maintained by the Google team. It supports not only common cloud platforms but also internal data centers.

Kubernetes built a container scheduling service which is designed to allow users to manage cloud container clusters through Kubernetes clusters without the need for complex setup tasks. The system will automatically select the appropriate working node to perform specific container cluster scheduling processing.

The scheduler needs to take into account individual and collective resource requirements, quality of service requirements, hardware/software/policy constraints, affinity and anti-affinity specifications, data locality, inter-workload interference, deadlines, and so on.

## Scheduling process

The scheduling process is mainly divided into 2 steps. In the _predicate_ step, the scheduler filters out nodes that do not satisfy required conditions. And in the _priority_ step, the scheduler sorts the nodes that meet all of the fit predicates, and then chooses the best one.

### Predicate stage

Kubernetes scheduler provides some predicates algorithms by default. For instance, the `HostNamePred` predicate checks if the hostname matches the requested hostname; the `PodsFitsResourcePred` checks if a node has sufficient resources, such as CPU, memory, GPU, opaque int resources and so on, to run a pod. Relevant code can be found in [kubernetes/pkg/scheduler/algorithm/predicates/](https://github.com/kubernetes/kubernetes/tree/master/pkg/scheduler/algorithm/predicates).

### Priority stage

In the priority step, the scheduler uses the `PrioritizeNodes` function to rank all nodes by calling each priority functions sequentially:
- Each priority function is expected to set a score of 0-10 where 0 is the lowest priority score (least preferred node) and 10 is the highest.
- Add all (weighted) scores for each node to get a total score.
- Select the node with the highest score.

## References
1. [Kubernetes document](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)
2. [Kubernetes introduction (in Chinese)](https://yeasy.gitbooks.io/docker_practice/kubernetes/)
3. [How does Kubernetes' scheduler work](http://carmark.github.io/2015/12/21/How-does-Kubernetes-scheduler-work/)
4. [Kubernetes Scheduling (in Chinese)](https://zhuanlan.zhihu.com/p/27754017)
