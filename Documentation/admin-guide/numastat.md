---
tip: translate by baidu@2024-01-30 21:36:19
---
---
title: Numa policy hit/miss statistics
---

/sys/devices/system/node/node\*/numastat

All units are pages. Hugepages have separate counters.


The numa_hit, numa_miss and numa_foreign counters reflect how well processes are able to allocate memory from nodes they prefer. If they succeed, numa_hit is incremented on the preferred node, otherwise numa_foreign is incremented on the preferred node and numa_miss on the node where allocation succeeded.

> numa_hit、numa_miss和numa_foreign计数器反映了进程从它们喜欢的节点分配内存的能力。如果它们成功，则在首选节点上递增numa_hit，否则在首选节点上将递增numa_eforeign，在分配成功的节点上将递增numa_emiss。


Usually preferred node is the one local to the CPU where the process executes, but restrictions such as mempolicies can change that, so there are also two counters based on CPU local node. local_node is similar to numa_hit and is incremented on allocation from a node by CPU on the same node. other_node is similar to numa_miss and is incremented on the node where allocation succeeds from a CPU from a different node. Note there is no counter analogical to numa_foreign.

> 通常，首选节点是进程执行的CPU本地节点，但mempolicy等限制可能会改变这一点，因此也有两个基于CPU本地节点的计数器。local_node类似于numa_hit，并且在同一节点上的CPU从一个节点分配时递增。other_node类似于numa_miss，并且在来自不同节点的CPU的分配成功的节点上递增。请注意，没有与numa_foreign相反的类比。

In more detail:

+-----------------+----------------------------------------------------+
| numa_hit A pr   | ocess wanted to allocate memory from this node,    |
+-----------------+----------------------------------------------------+
| > and succ      | eeded.                                             |
+-----------------+----------------------------------------------------+
| numa_miss A pr  | ocess wanted to allocate memory from another node, |
+-----------------+----------------------------------------------------+
| > but ende      | d up with memory from this node.                   |
+-----------------+----------------------------------------------------+
| numa_foreign    | A process wanted to allocate on this node,         |
+-----------------+----------------------------------------------------+
| > but ende      | d up with memory from another node.                |
+-----------------+----------------------------------------------------+
| local_node A pr | ocess ran on this node\'s CPU,                     |
+-----------------+----------------------------------------------------+
| > and got       | memory from this node.                             |
+-----------------+----------------------------------------------------+
| other_node A pr | ocess ran on a different node\'s CPU               |
+-----------------+----------------------------------------------------+
| > and got       | memory from this node.                             |
+-----------------+----------------------------------------------------+
| interleave_hit  | Interleaving wanted to allocate from this node     |
+-----------------+----------------------------------------------------+
| > and succ      | eeded.                                             |
+-----------------+----------------------------------------------------+


For easier reading you can use the numastat utility from the numactl package (<http://oss.sgi.com/projects/libnuma/>). Note that it only works well right now on machines with a small number of CPUs.

> 为了便于阅读，您可以使用numactl包中的numastat实用程序(<http://oss.sgi.com/projects/libnuma/>). 请注意，它现在只在具有少量CPU的机器上运行良好。


Note that on systems with memoryless nodes (where a node has CPUs but no memory) the numa_hit, numa_miss and numa_foreign statistics can be skewed heavily. In the current kernel implementation, if a process prefers a memoryless node (i.e. because it is running on one of its local CPU), the implementation actually treats one of the nearest nodes with memory as the preferred node. As a result, such allocation will not increase the numa_foreign counter on the memoryless node, and will skew the numa_hit, numa_miss and numa_foreign statistics of the nearest node.

> 请注意，在具有无内存节点（其中节点有CPU但没有内存）的系统上，numa_hit、numa_miss和numa_foreign统计数据可能会严重失真。在当前的内核实现中，如果进程更喜欢无内存节点（即，因为它在其一个本地CPU上运行），则该实现实际上将具有内存的最近节点之一视为首选节点。因此，这样的分配将不会增加无存储器节点上的numa_foreign计数器，并且将使最近节点的numa_hit、numa_miss和numa_foreign统计数据偏斜。
