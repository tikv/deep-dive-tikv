# Byzantine Failure

Consensus algorithms are typically either *Byzantine Fault Tolerant*, or not. Succinctly, systems which can withstand Byzantine faults are able to withstand misbehaving peers. Most distributed systems you would use inside of a VLAN, such as Kafka, TiKV, and etcd, are not Byzantine Fault Tolerant.

In order to withstand Byzantine faults, the system must tolerate peers:

* actively spreading incorrect information,
* deliberately not spreading correct information,
* modifying information that would otherwise be correct.

This extends far beyond situations where the network link degrades and starts corrupting packets at the TCP layer. Those kinds of issues are easily tractable compared to a system being able to withstand active internal subversion.

In order to better understand Byzantine Fault Tolerance it helps to imagine the Byzantine Generals Problem:

> Several Byzantine generals and their armies have surrounded an enemy army inside a deep forest. Separate, they are not strong enough to defeat the enemy, but if they attack in a coordinated fashion they will succeed. They must all agree on a time to attack the enemy.
>
> In order to communicate, the generals can send messengers through the forest. These messages may or may not reach their destination. They could be kidnapped and replaced with imposters, converted to the enemy cause, or outright killed.
>
> How can the generals confidently coordinate a time to attack?

After thinking on the problem for a time, you can conclude that tackling such a problem introduces a tremendous amount of complexity and overhead to a system.

Separating Byzantine tolerant systems from non-tolerant systems helps with evaluation of systems. A non-tolerant system will almost always outperform a tolerant system.

