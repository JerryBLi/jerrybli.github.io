---
title: "Online Hardware/Software Task Partitioning"
layout: post
date: 2018-12-06 12:00
tag: 
- Java
- Hardware/Software Partitioning
headerImage: false
projects: true
hidden: true # don't count this post in blog pagination
description: "Class project implementation of the algorithm described in “Online Hardware/Software Partitioning in Networked Embedded Systems” (Streichert, Haubelt & Teich, 2005)"
category: project
author: Jerry Li
externalLink: false
--- 

## Overview
Given a set of tasks that can be performed in either software or in hardware and a connected network of nodes, how does 
one partition the tasks between hardware and software and between the nodes so that 
1. The total load \(hardware and software\) is balanced between all the nodes
2. The hardware load and the software load is balanced on an individual node.

The paper “Online Hardware/Software Partitioning in Networked Embedded Systems” *\(Streichert, Haubelt & Teich, 2005\)* proposes
a method to partition the tasks between software and hardware as well as the individual nodes.

A few definitions and clarifications on the algorithm should be made: we have a set of nodes connected together, represented by a graph,
g<sub>a</sub> = (N, C) where *N* is the the nodes in the graph and *C* is the connections between the nodes. Each node is
a system that can run a task in either software or hardware \(think FPGA\). The graph, g<sub>a</sub>, can be represented with an adjacency matrix, *B*.
We also have a set of tasks P = {p<sub>1</sub>, ..., p<sub>m</sub>} that has to be distributed across the network to individual nodes. Each
task has both a software load, w<sub>s</sub>, and hardware load, w<sub>h</sub>, which corresponds to a pre-defined weight if the task is executed in software or hardware, respectively.

An implementation of the algorithm described by *Streichert et. al* was developed for one of my classes at Stony Brook as an assignment.

## Algorithm Description
The algorithm is divided into two parts - an iterative diffusion algorithm and a selection algorithm. The diffusion algorithm determines which tasks
from a given node, n<sub>i</sub>, is sent over to a neighboring node, n<sub>j</sub>. The selection algorithm takes the tasks on a node and
determines whether it is executed in hardware or software.

The diffusion algorithm determines the amount of load that has to go from one node, n<sub>i</sub>, to another node, n<sub>j</sub>, with the formula
*y = α\(w<sub>i</sub> - w<sub>j</sub>\)* where α is the inverse of λ where λ  are the non-zero eigenvalues of the Laplacian matrix, L, of
the network. The Laplacian matrix can be calculated with the matrix *D*, which represents the node degrees as diagonal entries,
and the adjacency matrix *B* as *L = D - B*.

The selection algorithm focuses on balancing the hardware and software tasks so that the load between the two are as small as possible while
simultaneously trying to keep the overall load as small as possible.

## Implementation
For the implementation of this algorithm, a representation of a Node is created and a representation of a Task is created. The algorithm takes in the adjacency graph of the
nodes as input. It also takes in the list of tasks with their hardware and software load values. For the project, we simulated the inputs with randomly generated adjacency graphs
and randomly generated tasks.

Since our implementation is far from the ideal theoretical implementation, we stopped the program once the balance of the node loads were under a specified threshold.
The balance was measured by the difference of the average load and the maximum load \(hardware or software\). The diffusion algorithm iterated through each node
and calculated an amount of load, *y* to send over to another node. This selection is an NP-complete problem and is inefficient to attempt all combinations, thus the algorithm 
 incrementally selects current tasks from smallest to largest load until the next task would have surpassed the load limit *y*. 
 
The selection algorithm then goes through every node after the diffusion and balances the loads. This was implemented by going through each task and choosing the smaller value between hardware and software.
Afterwards, either hardware tasks were changed to software or vice versa depending on which load is larger. By the end of this algorithm, the tasks in a node should be both minimized and balanced.
 
The diffusion and selection is run in a loop until the threshold for balance is reached.

## Final Thoughts
* This algorithm assumes all nodes are homogeneous and can handle the same amount of loads - this may not be the case in a real world network.
* This algorithm requires that all tasks to be able to be implemented in both hardware and software to benefit. We can of course set the load value
of a task to INFINITY if it cannot be implemented in one of the methods.
* Each task must be executed on a single node - there could be a need for a task to be able to be partially executed on multiple nodes.

**Find the project [here](https://github.com/JerryBLi/HWSW-Task-Partitioning)**

**For more information, read the paper [_Online Hardware/Software Partitioning in Networked Embedded Systems_](https://ieeexplore.ieee.org/document/1466504)**