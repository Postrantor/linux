---
tip: translate by baidu@2024-01-16 22:50:24
title: Scheduler Domains
---

Each CPU has a "base" scheduling domain (struct sched_domain). The domain hierarchy is built from these base domains via the `->parent` pointer. `->parent` MUST be NULL terminated, and domain structures should be per-CPU as they are locklessly updated.

> 每个 CPU 都有一个“基本”调度域（结构体 sched_domain）。域层次结构是通过`->parent`指针从这些基本域构建的。`->parent`必须以 NULL 结尾，并且域结构应按 CPU 进行无锁定更新。

Each scheduling domain spans a number of CPUs (stored in the ->span field). A domain\'s span MUST be a superset of it child\'s span (this restriction could be relaxed if the need arises), and a base domain for CPU i MUST span at least i. The top domain for each CPU will generally span all CPUs in the system although strictly it doesn\'t have to, but this could lead to a case where some CPUs will never be given tasks to run unless the CPUs allowed mask is explicitly set. A sched domain\'s span means "balance process load among these CPUs".

> 每个调度域跨越多个 CPU（存储在->span 字段中）。域的跨度必须是其子跨度的超集（如果需要，可以放宽这一限制），CPU i 的基域必须至少跨越 i。每个 CPU 的顶级域通常会跨越系统中的所有 CPU，尽管严格来说不必如此，但这可能会导致某些 CPU 永远不会被赋予运行任务，除非明确设置了 CPU 允许的掩码。sched 域的跨度意味着“在这些 CPU 之间平衡进程负载”。

Each scheduling domain must have one or more CPU groups (struct sched_group) which are organised as a circular one way linked list from the ->groups pointer. The union of `cpumasks` of these groups MUST be the same as the domain\'s span. The group pointed to by the ->groups pointer MUST contain the CPU to which the domain belongs. Groups may be shared among CPUs as they contain read only data after they have been set up. The intersection of `cpumasks` from any two of these groups may be non empty. If this is the case the SD_OVERLAP flag is set on the corresponding scheduling domain and its groups may not be shared between CPUs.

> 每个调度域必须有一个或多个 CPU 组（struct-sched_group），这些组从->groups 指针组织为循环单向链表。这些组的“cpumasks”的并集必须与域的跨度相同。->groups 指针指向的组必须包含域所属的 CPU。组可以在 CPU 之间共享，因为它们在设置后包含只读数据。来自这些组中任意两个组的“cpumasks”的交集可以是非空的。如果是这种情况，则在相应的调度域上设置 SD_OVERLAP 标志，并且其组可能不会在 CPU 之间共享。

Balancing within a sched domain occurs between groups. That is, each group is treated as one entity. The load of a group is defined as the sum of the load of each of its member CPUs, and only when the load of a group becomes out of balance are tasks moved between groups.

> 调度域内的平衡发生在组之间。也就是说，每个组都被视为一个实体。组的负载被定义为其每个成员 CPU 的负载之和，只有当组的负载变得不平衡时，任务才会在组之间移动。

In "kernel/sched/core.c", `trigger_load_balance()` is run periodically on each CPU through `scheduler_tick()`. It raises a softirq after the next regularly scheduled rebalancing event for the current runqueue has arrived. The actual load balancing workhorse, `run_rebalance_domains()->rebalance_domains()`, is then run in softirq context (`SCHED_SOFTIRQ`).

> 在“kernel/sched/core.c”中，“trigger_load_balance（）”通过“schedule_tick（）”在每个 CPU 上定期运行。在当前运行队列的下一个定期安排的再平衡事件到达后，它会引发一个 softirq。然后，在 softirq 上下文（`SCHED_softirq`）中运行实际的负载平衡主力，`run_rebalance_domains（）->rebalance_ddomains（）`。

The latter function takes two arguments: the runqueue of current CPU and whether the CPU was idle at the time the scheduler_tick() happened and iterates over all sched domains our CPU is on, starting from its base domain and going up the `->parent` chain. While doing that, it checks to see if the current domain has exhausted its rebalance interval. If so, it runs `load_balance()` on that domain. It then checks the parent sched_domain (if it exists), and the parent of the parent and so forth.

> 后一个函数接受两个参数：当前 CPU 的运行队列，以及在 schedule_tick（）发生时 CPU 是否空闲，并在 CPU 所在的所有调度域上迭代，从其基域开始，向上遍历`->parent`链。在执行此操作时，它会检查当前域是否已耗尽其重新平衡间隔。如果是，它将在该域上运行“load_balance（）”。然后，它检查父级 sched_domain（如果存在）和父级的父级，依此类推。

Initially, `load_balance()` finds the busiest group in the current sched domain. If it succeeds, it looks for the busiest runqueue of all the CPUs\' runqueues in that group. If it manages to find such a runqueue, it locks both our initial CPU\'s runqueue and the newly found busiest one and starts moving tasks from it to our runqueue. The exact number of tasks amounts to an imbalance previously computed while iterating over this sched domain\'s groups.

> 最初，“load_balance（）”查找当前调度域中最繁忙的组。如果成功，它将查找该组中所有 CPU 的运行队列中最繁忙的运行队列。如果它设法找到这样一个运行队列，它会锁定我们初始 CPU 的运行队列和新发现的最繁忙的运行队列，并开始将任务从它移动到我们的运行队列。任务的确切数量相当于先前在该计划域的组上迭代时计算的不平衡。

# Implementing sched domains

The "base" domain will "span" the first level of the hierarchy. In the case of SMT, you\'ll span all siblings of the physical CPU, with each group being a single virtual CPU.

> “基本”域将“跨越”层次结构的第一级。在 SMT 的情况下，您将跨越物理 CPU 的所有同级，每个组都是一个虚拟 CPU。

In SMP, the parent of the base domain will span all physical CPUs in the node. Each group being a single physical CPU. Then with NUMA, the parent of the SMP domain will span the entire machine, with each group having the cpumask of a node. Or, you could do multi-level NUMA or Opteron, for example, might have just one domain covering its one NUMA level.

> 在 SMP 中，基域的父级将跨越节点中的所有物理 CPU。每个组都是一个物理 CPU。然后使用 NUMA，SMP 域的父级将覆盖整个机器，每个组都有一个节点的 cpumask。或者，您可以进行多级 NUMA 或 Opteron，例如，可能只有一个域覆盖其一个 NUMA 级别。

The implementor should read comments in include/linux/sched/sd_flags.h: `[SD]()*` to get an idea of the specifics and what to tune for the SD flags of a sched_domain.

> 实现者应该阅读 include/linux/sched/sd_flags.h:`[sd]（）*`中的注释，以了解 sched_domain 的 sd 标志的细节和调整内容。

Architectures may override the generic domain builder and the default SD flags for a given topology level by creating a sched_domain_topology_level array and calling set_sched_topology() with this array as the parameter.

> 体系结构可以通过创建 sched_domain_topology_level 数组并以该数组为参数调用 set_sched_topology（）来覆盖给定拓扑级别的通用域生成器和默认 SD 标志。

The sched-domains debugging infrastructure can be enabled by enabling `CONFIG_SCHED_DEBUG` and adding \'sched_verbose\' to your cmdline. If you forgot to tweak your cmdline, you can also flip the "/sys/kernel/debug/sched/verbose" knob. This enables an error checking parse of the sched domains which should catch most possible errors (described above). It also prints out the domain structure in a visual format.

> 可以通过启用“CONFIG_sched_DEBUG”并将“ched_verbose”添加到命令行来启用计划域调试基础结构。如果忘记调整 cmdline，也可以翻转“/sys/kernel/debug/sched/verbose”旋钮。这使得能够对 sched 域进行错误检查解析，从而捕获大多数可能的错误（如上所述）。它还以可视化格式打印出域结构。
