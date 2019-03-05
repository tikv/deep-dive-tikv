# Simulator

## Overview

With its flexibility and significant benefits of reducing the time and the
cost, the simulator plays an important role in studying and designing a
computer architecture. It is often used to validate specific design schemes
and evaluate the effectiveness of design schemes.

## Workflow

Since this book is mainly targeted towards the people who are interesting in
TiKV, we will focus on using the simulator to deal with the scheduling problem.
In general, when there is a lack of resources or the problem is hard to
reproduce, we might consider using the simulator to make it.

A simulation for the scheduling problem in the distributed system
usually consists of the following steps:

1. Define the system model of the simulator.
2. Set up the simulation environment.
3. Run the simulation.
4. Inspect the result to check whether it is in line with the expectation.

The first step is mainly to figure out which part of your system you want to
simulate. And the model should be as simple as possible. In the second step, you
should set up the environment including the scale of your system and the
characteristic of the workload. The third step will run the simulation which
will give the output of the scheduling. In the final step, you can check the
result and dig into the scheduling problem if the result is not as expected.

## PD simulator

In PD, we also need to use the simulator to find the scheduling problem.
It can be used to simulate a large-scale cluster and different users' scenarios.
And for some special scenarios, we can keep it so that we can quickly verify the
correctness of the scheduling in PD under different scenarios when we
reconstruct the code or add some new features in the future. Without the
simulator, if we want to reproduce some scenarios, we need to apply for
machines, load data, and then wait for the scheduling. It is tedious and might
waste a lot of time.

### Architecture

The basic architecture of PD Simulator is shown in *Figure 1*.

![Figure 1](pd-simulator.png)

### Components

PD Simulator is consist of the following components:

- Driver

  _Driver_ is the most important part of PD Simulator. It is used to do some
  initialization and trigger the heartbeat and the corresponding event according
  to the tick count.

- Node

  _Node_ is used to simulate a TiKV. It contains the basic information of a
  store and can communicate with PD by using the heartbeat through gRPC.

- Raft Engine

  _Raft Engine_ records all Raft related information. It is a shared Raft
  engine, which PD can't know about it.

- Event Runner

  For every tick, _Event Runner_ will run the function to check if there is an
  event need to execute. if true, it will execute the corresponding event.

### Process

The basic process about how PD Simulator works is as follows

1. When starting PD Simulator, it will create a driver and initialize a mocked
TiKV cluster which consists of nodes.
2. After PD is bootstrapped, it will start to run a timer.
3. For each tick, the mocking TiKV cluster will execute some operations, such as
executing Raft commands on the shared Raft engine or sending the heartbeat. And
according to the different cases, it will perform the different events.
4. Finally, it will verify if the result is in line with our expectations.

PD Simulator doesn't care about how TiKV actually works in details. It just
needs to send the messages which PD wants to know about.
