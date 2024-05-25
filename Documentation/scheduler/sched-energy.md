---
tip: translate by baidu@2024-01-16 22:56:03
title: Energy Aware Scheduling
---

# 1. Introduction

Energy Aware Scheduling (or EAS) gives the scheduler the ability to predict the impact of its decisions on the energy consumed by CPUs. EAS relies on an Energy Model (EM) of the CPUs to select an energy efficient CPU for each task, with a minimal impact on throughput. This document aims at providing an introduction on how EAS works, what are the main design decisions behind it, and details what is needed to get it to run.

> 能量感知调度（或 EAS）使调度器能够预测其决策对 CPU 消耗的能量的影响。EAS 依靠 CPU 的能量模型（EM）为每个任务选择一个节能的 CPU，对吞吐量的影响最小。本文档旨在介绍 EAS 是如何工作的，其背后的主要设计决策是什么，以及运行它所需的详细信息。

Before going any further, please note that at the time of writing:

    /!\ EAS does not support platforms with symmetric CPU topologies /!\

EAS operates only on heterogeneous CPU topologies (such as Arm big.LITTLE) because this is where the potential for saving energy through scheduling is the highest.

> EAS 仅在异构 CPU 拓扑（如 Arm big.LITTLE）上运行，因为这是通过调度节省能源的潜力最大的地方。

The actual EM used by EAS is [not]() maintained by the scheduler, but by a dedicated framework. For details about this framework and what it provides, please refer to its documentation (see "Documentation/power/energy-model.rst").

> EAS 使用的实际 EM[不是]（）由调度器维护，而是由专用框架维护。有关该框架及其提供的详细信息，请参阅其文档（请参阅“documentation/power/energy model.rst”）。

# 2. Background and Terminology

To make it clear from the start:

```
:   -   energy = \[joule\] (resource like a battery on powered devices)
    -   power = energy/time = \[joule/second\] = \[watt\]
```

The goal of EAS is to minimize energy, while still getting the job done. That is, we want to maximize:

> EAS 的目标是最大限度地减少能源，同时仍然完成任务。也就是说，我们希望最大化：

```
    performance [inst/s]
    --------------------
        power [W]
```

which is equivalent to minimizing:

```
    energy [J]
    -----------
    instruction
```

while still getting 'good' performance. It is essentially an alternative optimization objective to the current performance-only objective for the scheduler. This alternative considers two objectives: energy-efficiency and performance.

> 同时仍然获得“良好”的性能。它本质上是调度器当前仅性能目标的替代优化目标。这一备选方案考虑了两个目标：能源效率和性能。

The idea behind introducing an EM is to allow the scheduler to evaluate the implications of its decisions rather than blindly applying energy-saving techniques that may have positive effects only on some platforms. At the same time, the EM must be as simple as possible to minimize the scheduler latency impact.

> 引入 EM 背后的想法是允许调度器评估其决策的影响，而不是盲目地应用可能仅在某些平台上产生积极影响的节能技术。同时，EM 必须尽可能简单，以最大限度地减少对调度器延迟的影响。

In short, EAS changes the way CFS tasks are assigned to CPUs. When it is time for the scheduler to decide where a task should run (during wake-up), the EM is used to break the tie between several good CPU candidates and pick the one that is predicted to yield the best energy consumption without harming the system's throughput. The predictions made by EAS rely on specific elements of knowledge about the platform's topology, which include the 'capacity' of CPUs, and their respective energy costs.

> 简而言之，EAS 改变了将 CFS 任务分配给 CPU 的方式。当调度器决定任务应该在哪里运行时（在唤醒期间），EM 用于打破几个好的 CPU 候选者之间的联系，并选择一个预测能在不损害系统吞吐量的情况下产生最佳能耗的 CPU。EAS 的预测依赖于有关平台拓扑结构的特定知识元素，其中包括 CPU 的“容量”及其各自的能源成本。

# 3. Topology information

EAS (as well as the rest of the scheduler) uses the notion of 'capacity' to differentiate CPUs with different computing throughput. The 'capacity' of a CPU represents the amount of work it can absorb when running at its highest frequency compared to the most capable CPU of the system. Capacity values are normalized in a 1024 range, and are comparable with the utilization signals of tasks and CPUs computed by the Per-Entity Load Tracking (PELT) mechanism. Thanks to capacity and utilization values, EAS is able to estimate how big/busy a task/CPU is, and to take this into consideration when evaluating performance vs energy trade-offs. The capacity of CPUs is provided via arch-specific code through the arch_scale_cpu_capacity() callback.

> EAS（以及调度器的其余部分）使用“容量”的概念来区分具有不同计算吞吐量的 CPU。CPU 的“容量”表示与系统中能力最强的 CPU 相比，在最高频率下运行时可以吸收的工作量。容量值在 1024 范围内进行标准化，并可与通过每实体负载跟踪（PELT）机制计算的任务和 CPU 的利用率信号进行比较。得益于容量和利用率值，EAS 能够估计任务/CPU 的大小/繁忙程度，并在评估性能与能源的权衡时将其考虑在内。cpu 的容量是通过 arch_scale_cpu_capacity（）回调通过 arch 特定代码提供的。

The rest of platform knowledge used by EAS is directly read from the Energy Model (EM) framework. The EM of a platform is composed of a power cost table per 'performance domain' in the system (see "Documentation/power/energy-model.rst" for further details about performance domains).

> EAS 使用的其余平台知识直接从能源模型（EM）框架中读取。平台的 EM 由系统中每个“性能域”的功率成本表组成（有关性能域的更多详细信息，请参阅“文档/功率/能量模型.rst”）。

The scheduler manages references to the EM objects in the topology code when the scheduling domains are built, or re-built. For each root domain (rd), the scheduler maintains a singly linked list of all performance domains intersecting the current `rd->span`. Each node in the list contains a pointer to a struct em_perf_domain as provided by the EM framework.

> 当构建或重新构建调度域时，调度程序管理对拓扑代码中 EM 对象的引用。对于每个根域（rd），调度程序维护一个与当前“rd->span”相交的所有性能域的单独链接列表。列表中的每个节点都包含一个指向 em 框架提供的结构 em_perf_domain 的指针。

The lists are attached to the root domains in order to cope with exclusive cpuset configurations. Since the boundaries of exclusive `cpusets` do not necessarily match those of performance domains, the lists of different root domains can contain duplicate elements.

> 这些列表附加到根域，以便处理独占的 cpuset 配置。由于独占“cpusets”的边界不一定与性能域的边界匹配，因此不同根域的列表可能包含重复的元素。

Example 1.

: Let us consider a platform with 12 CPUs, split in 3 performance domains (pd0, pd4 and pd8), organized as follows:

> ：让我们考虑一个具有 12 个 CPU 的平台，分为 3 个性能域（pd0、pd4 和 pd8），组织如下：

```
CPUs:   0 1 2 3 4 5 6 7 8 9 10 11
PDs:   |--pd0--|--pd4--|---pd8---|
RDs:   |----rd1----|-----rd2-----|
```

Now, consider that `userspace` decided to split the system with two exclusive `cpusets`, hence creating two independent root domains, each containing 6 CPUs. The two root domains are denoted rd1 and rd2 in the above figure. Since pd4 intersects with both rd1 and rd2, it will be present in the linked list '->pd' attached to each of them:

> 现在，考虑一下“用户空间”决定用两个互斥的“cpu”来拆分系统，从而创建两个独立的根域，每个根域包含 6 个 CPU。在上图中，这两个根域分别表示为 rd1 和 rd2。由于 pd4 与 rd1 和 rd2 相交，因此它将出现在它们各自的链表“->pd”中：

> - rd1->pd: pd0 -> pd4
> - rd2->pd: pd4 -> pd8

Please note that the scheduler will create two duplicate list nodes for pd4 (one for each list). However, both just hold a pointer to the same shared data structure of the EM framework.

> 请注意，调度程序将为 pd4 创建两个重复的列表节点（每个列表一个）。然而，两者都只是持有一个指向 EM 框架的相同共享数据结构的指针。

Since the access to these lists can happen concurrently with hotplug and other things, they are protected by RCU, like the rest of topology structures manipulated by the scheduler.

> 由于对这些列表的访问可以与热插拔和其他事情同时进行，因此它们受到 RCU 的保护，就像调度器操作的其他拓扑结构一样。

EAS also maintains a static key (sched_energy_present) which is enabled when at least one root domain meets all conditions for EAS to start. Those conditions are summarized in Section 6.

> EAS 还维护静态密钥（sched_energy_present），当至少一个根域满足 EAS 启动的所有条件时启用该静态密钥。第 6 节概述了这些条件。

# 4. Energy-Aware task placement

EAS overrides the CFS task wake-up balancing code. It uses the EM of the platform and the PELT signals to choose an energy-efficient target CPU during wake-up balance. When EAS is enabled, `select_task_rq_fair()` calls `find_energy_efficient_cpu()` to do the placement decision. This function looks for the CPU with the highest spare capacity (CPU capacity - CPU utilization) in each performance domain since it is the one which will allow us to keep the frequency the lowest. Then, the function checks if placing the task there could save energy compared to leaving it on prev_cpu, i.e. the CPU where the task ran in its previous activation.

> EAS 覆盖 CFS 任务唤醒平衡代码。在唤醒平衡期间，它使用平台的 EM 和 PELT 信号来选择节能的目标 CPU。启用 EAS 时，“select_task_rq_fair（）”调用“find_energy_effective_cpu（）”来进行放置决策。该函数在每个性能域中寻找具有最高备用容量（CPU 容量-CPU 利用率）的 CPU，因为它可以使我们保持最低的频率。然后，该功能检查将任务放置在那里是否可以比将其保留在 prev_cpu（即任务上次激活时运行的 cpu）上节省能量。

`find_energy_efficient_cpu()` uses `compute_energy()` to estimate what will be the energy consumed by the system if the waking task was migrated. `compute_energy()` looks at the current utilization landscape of the CPUs and adjusts it to 'simulate' the task migration. The EM framework provides the `em_pd_energy()` API which computes the expected energy consumption of each performance domain for the given utilization landscape.

An example of energy-optimized task placement decision is detailed below.

Example 2.

: Let us consider a (fake) platform with 2 independent performance domains composed of two CPUs each. CPU0 and CPU1 are little CPUs; CPU2 and CPU3 are big.

> ：让我们考虑一个（假的）平台，它有两个独立的性能域，每个域由两个 CPU 组成。CPU0 和 CPU1 是小 CPU；CPU2 和 CPU3 很大。

The scheduler must decide where to place a task P whose util_avg = 200 and prev_cpu = 0.

> 调度器必须决定将 util_avg=200 且 prev_cpu=0 的任务 P 放置在何处。

The current utilization landscape of the CPUs is depicted on the graph below. CPUs 0-3 have a util_avg of 400, 100, 600 and 500 respectively Each performance domain has three Operating Performance Points (OPPs). The CPU capacity and power cost associated with each OPP is listed in the Energy Model table. The util_avg of P is shown on the figures below as 'PP':

> CPU 的当前使用情况如下图所示。CPU 0-3 的 util_avg 分别为 400、100、600 和 500。每个性能域都有三个操作性能点（OPP）。能量模型表中列出了与每个 OPP 相关的 CPU 容量和电源成本。P 的 util_avg 在下图中显示为“PP”：

```
        CPU util.
         1024                 - - - - - - -              Energy Model
                                                  +-----------+-------------+
                                                  |  Little   |     Big     |
          768                 =============       +-----+-----+------+------+
                                                  | Cap | Pwr | Cap  | Pwr  |
                                                  +-----+-----+------+------+
          512  ===========    - ##- - - - -       | 170 | 50  | 512  | 400  |
                                ##     ##         | 341 | 150 | 768  | 800  |
          341  -PP - - - -      ##     ##         | 512 | 300 | 1024 | 1700 |
                PP              ##     ##         +-----+-----+------+------+
          170  -## - - - -      ##     ##
                ##     ##       ##     ##
              ------------    -------------
               CPU0   CPU1     CPU2   CPU3

         Current OPP: =====       Other OPP: - - -     util_avg (100 each): ##
```

`find_energy_efficient_cpu()` will first look for the CPUs with the maximum spare capacity in the two performance domains. In this example, CPU1 and CPU3. Then it will estimate the energy of the system if P was placed on either of them, and check if that would save some energy compared to leaving P on CPU0. EAS assumes that OPPs follow utilization (which is coherent with the behaviour of the schedutil CPUFreq governor, see Section 6. for more details on this topic).

```
    **Case 1. P is migrated to CPU1**:

        1024                 - - - - - - -

                                              Energy calculation:
         768                 =============     * CPU0: 200 / 341 * 150 = 88
                                               * CPU1: 300 / 341 * 150 = 131
                                               * CPU2: 600 / 768 * 800 = 625
         512  - - - - - -    - ##- - - - -     * CPU3: 500 / 768 * 800 = 520
                               ##     ##          => total_energy = 1364
         341  ===========      ##     ##
                      PP       ##     ##
         170  -## - - PP-      ##     ##
               ##     ##       ##     ##
             ------------    -------------
              CPU0   CPU1     CPU2   CPU3

    **Case 2. P is migrated to CPU3**:

        1024                 - - - - - - -

                                              Energy calculation:
         768                 =============     * CPU0: 200 / 341 * 150 = 88
                                               * CPU1: 100 / 341 * 150 = 43
                                      PP       * CPU2: 600 / 768 * 800 = 625
         512  - - - - - -    - ##- - -PP -     * CPU3: 700 / 768 * 800 = 729
                               ##     ##          => total_energy = 1485
         341  ===========      ##     ##
                               ##     ##
         170  -## - - - -      ##     ##
               ##     ##       ##     ##
             ------------    -------------
              CPU0   CPU1     CPU2   CPU3

    **Case 3. P stays on prev_cpu / CPU 0**:

        1024                 - - - - - - -

                                              Energy calculation:
         768                 =============     * CPU0: 400 / 512 * 300 = 234
                                               * CPU1: 100 / 512 * 300 = 58
                                               * CPU2: 600 / 768 * 800 = 625
         512  ===========    - ##- - - - -     * CPU3: 500 / 768 * 800 = 520
                               ##     ##          => total_energy = 1437
         341  -PP - - - -      ##     ##
               PP              ##     ##
         170  -## - - - -      ##     ##
               ##     ##       ##     ##
             ------------    -------------
              CPU0   CPU1     CPU2   CPU3

```

From these calculations, the Case 1 has the lowest total energy. So CPU 1 is be the best candidate from an energy-efficiency standpoint.

> 根据这些计算，情况 1 具有最低的总能量。因此，从能源效率的角度来看，CPU1 是最佳的候选者。

Big CPUs are generally more power hungry than the little ones and are thus used mainly when a task doesn't fit the littles. However, little CPUs aren't always necessarily more energy-efficient than big CPUs. For some systems, the high OPPs of the little CPUs can be less energy-efficient than the lowest OPPs of the bigs, for example. So, if the little CPUs happen to have enough utilization at a specific point in time, a small task waking up at that moment could be better of executing on the big side in order to save energy, even though it would fit on the little side.

> 大 CPU 通常比小 CPU 更耗电，因此主要在任务不适合小 CPU 时使用。然而，小型 CPU 并不总是比大型 CPU 更节能。例如，对于一些系统，小 CPU 的高 OPP 可能不如大 CPU 的最低 OPP 节能。因此，如果小 CPU 在特定的时间点碰巧有足够的利用率，那么在那一刻醒来的小任务可能更适合在大的方面执行，以节省能源，即使它适合在小的方面。

And even in the case where all OPPs of the big CPUs are less energy-efficient than those of the little, using the big CPUs for a small task might still, under specific conditions, save energy. Indeed, placing a task on a little CPU can result in raising the OPP of the entire performance domain, and that will increase the cost of the tasks already running there. If the waking task is placed on a big CPU, its own execution cost might be higher than if it was running on a little, but it won't impact the other tasks of the little CPUs which will keep running at a lower OPP. So, when considering the total energy consumed by CPUs, the extra cost of running that one task on a big core can be smaller than the cost of raising the OPP on the little CPUs for all the other tasks.

> 即使在大 CPU 的所有 OPP 都不如小 CPU 的节能的情况下，在特定条件下，使用大 CPU 执行小任务仍然可以节省能源。事实上，将任务放在一个小 CPU 上可能会提高整个性能域的 OPP，这将增加已经在那里运行的任务的成本。如果唤醒任务放在一个大 CPU 上，它本身的执行成本可能比运行在一个小 CPU 上更高，但它不会影响小 CPU 的其他任务，这些任务将以较低的 OPP 运行。因此，当考虑 CPU 消耗的总能量时，在大核心上运行一个任务的额外成本可能小于在小 CPU 上为所有其他任务提高 OPP 的成本。

The examples above would be nearly impossible to get right in a generic way, and for all platforms, without knowing the cost of running at different OPPs on all CPUs of the system. Thanks to its EM-based design, EAS should cope with them correctly without too many troubles. However, in order to ensure a minimal impact on throughput for high-utilization scenarios, EAS also implements another mechanism called 'over-utilization'.

> 如果不知道在系统的所有 CPU 上以不同的 OPP 运行的成本，对于所有平台来说，上述示例几乎不可能以通用的方式正确。由于其基于 EM 的设计，EAS 应该能够正确地应对这些问题，而不会遇到太多麻烦。然而，为了确保高利用率场景对吞吐量的影响最小，EAS 还实现了另一种称为“过度利用”的机制。

# 5. Over-utilization

From a general standpoint, the use-cases where EAS can help the most are those involving a light/medium CPU utilization. Whenever long CPU-bound tasks are being run, they will require all of the available CPU capacity, and there isn't much that can be done by the scheduler to save energy without severely harming throughput. In order to avoid hurting performance with EAS, CPUs are flagged as 'over-utilized' as soon as they are used at more than 80% of their compute capacity. As long as no CPUs are over-utilized in a root domain, load balancing is disabled and EAS overridess the wake-up balancing code. EAS is likely to load the most energy efficient CPUs of the system more than the others if that can be done without harming throughput. So, the load-balancer is disabled to prevent it from breaking the energy-efficient task placement found by EAS. It is safe to do so when the system isn't overutilized since being below the 80% tipping point implies that:

> 从一般的角度来看，EAS 可以提供最大帮助的用例是那些涉及轻/中等 CPU 利用率的用例。每当运行长时间的 CPU 限制任务时，它们都需要所有可用的 CPU 容量，而且调度器在不严重损害吞吐量的情况下无法节省能源。为了避免使用 EAS 损害性能，一旦 CPU 的计算容量超过 80%，就会将其标记为“过度使用”。只要根域中没有 CPU 被过度利用，负载平衡就会被禁用，EAS 就会覆盖唤醒平衡代码。如果可以在不影响吞吐量的情况下加载系统中最节能的 CPU，EAS 可能会比其他 CPU 加载更多。因此，负载均衡器被禁用，以防止它破坏 EAS 发现的节能任务布局。当系统没有被过度使用时，这样做是安全的，因为低于 80%的临界点意味着：

a. there is some idle time on all CPUs, so the utilization signals used by EAS are likely to accurately represent the 'size' of the various tasks in the system;

> a.所有 CPU 都有一些空闲时间，因此 EAS 使用的利用率信号可能准确地表示系统中各种任务的“大小”；

b. all tasks should already be provided with enough CPU capacity, regardless of their nice values;

> b.所有任务都应该已经提供了足够的 CPU 容量，无论它们的值如何；

c. since there is spare capacity all tasks must be blocking/sleeping regularly and balancing at wake-up is sufficient.

> c.由于有空闲容量，所有任务都必须定期阻塞/睡眠，并且在唤醒时进行平衡就足够了。

As soon as one CPU goes above the 80% tipping point, at least one of the three assumptions above becomes incorrect. In this scenario, the 'overutilized' flag is raised for the entire root domain, EAS is disabled, and the load-balancer is re-enabled. By doing so, the scheduler falls back onto load-based algorithms for wake-up and load balance under CPU-bound conditions. This provides a better respect of the nice values of tasks.

> 一旦一个 CPU 超过 80%的临界点，以上三个假设中至少有一个是不正确的。在这种情况下，将为整个根域引发“过度使用”标志，禁用 EAS，并重新启用负载平衡器。通过这样做，调度器回到基于负载的算法上，以便在 CPU 受限的条件下进行唤醒和负载平衡。这提供了对任务的美好价值观的更好尊重。

Since the notion of overutilization largely relies on detecting whether or not there is some idle time in the system, the CPU capacity 'stolen' by higher (than CFS) scheduling classes (as well as IRQ) must be taken into account. As such, the detection of overutilization accounts for the capacity used not only by CFS tasks, but also by the other scheduling classes and IRQ.

> 由于过度利用的概念在很大程度上取决于检测系统中是否有一些空闲时间，因此必须考虑更高（比 CFS）调度类（以及 IRQ）“窃取”的 CPU 容量。因此，过度利用的检测不仅考虑了 CFS 任务使用的容量，还考虑了其他调度类和 IRQ 使用的容量。

# 6. Dependencies and requirements for EAS

Energy Aware Scheduling depends on the CPUs of the system having specific hardware properties and on other features of the kernel being enabled. This section lists these dependencies and provides hints as to how they can be met.

> 能量感知调度取决于具有特定硬件属性的系统的 CPU 以及正在启用的内核的其他功能。本节列出了这些依赖关系，并提供了如何满足这些依赖关系的提示。

## 6.1 - Asymmetric CPU topology

As mentioned in the introduction, EAS is only supported on platforms with asymmetric CPU topologies for now. This requirement is checked at run-time by looking for the presence of the `SD_ASYM_CPUCAPACITY_FULL` flag when the scheduling domains are built.

> 如引言中所述，EAS 目前仅在具有不对称 CPU 拓扑的平台上受支持。在构建调度域时，通过查找“SD_ASYM_CPUCAPACITY_FULL”标志的存在，在运行时检查此要求。

See "Documentation/scheduler/sched-capacity.rst" for requirements to be met for this flag to be set in the sched_domain hierarchy.

> 有关在 sched_domain 层次结构中设置此标志所需满足的要求，请参阅“Documentation/scheduller/sched capacity.rst”。

Please note that EAS is not fundamentally incompatible with SMP, but no significant savings on SMP platforms have been observed yet. This restriction could be amended in the future if proven otherwise.

> 请注意，EAS 与 SMP 并非根本不兼容，但尚未观察到 SMP 平台上的显著节约。如果有其他证明，这一限制将来可能会修改。

## 6.2 - Energy Model presence

EAS uses the EM of a platform to estimate the impact of scheduling decisions on energy. So, your platform must provide power cost tables to the EM framework in order to make EAS start. To do so, please refer to documentation of the independent EM framework in "Documentation/power/energy-model.rst".

> EAS 使用平台的 EM 来估计调度决策对能源的影响。因此，为了启动 EAS，您的平台必须向 EM 框架提供功率成本表。为此，请参阅“documentation/power/energy model.rst”中独立 EM 框架的文档。

Please also note that the scheduling domains need to be re-built after the EM has been registered in order to start EAS.

> 还请注意，为了启动 EAS，需要在 EM 注册后重新构建调度域。

EAS uses the EM to make a forecasting decision on energy usage and thus it is more focused on the difference when checking possible options for task placement. For EAS it doesn't matter whether the EM power values are expressed in milli-Watts or in an 'abstract scale'.

> EAS 使用 EM 对能源使用做出预测决策，因此在检查任务布局的可能选项时，它更关注差异。对于 EAS 来说，EM 功率值是以毫瓦表示还是以“抽象尺度”表示并不重要。

## 6.3 - Energy Model complexity

EAS does not impose any complexity limit on the number of PDs/OPPs/CPUs but restricts the number of CPUs to `EM_MAX_NUM_CPUS` to prevent overflows during the energy estimation.

> EAS 不对 PD/OPS/CPU 的数量施加任何复杂度限制，而是将 CPU 的数量限制为“EM_MAX_NUM_CPUs”以防止在能量估计期间溢出。

## 6.4 - Schedutil governor

EAS tries to predict at which OPP will the CPUs be running in the close future in order to estimate their energy consumption. To do so, it is assumed that OPPs of CPUs follow their utilization.

> EAS 试图预测 CPU 在不久的将来将以何种 OPP 运行，以估计其能耗。为此，假设 CPU 的 OPP 遵循它们的利用率。

Although it is very difficult to provide hard guarantees regarding the accuracy of this assumption in practice (because the hardware might not do what it is told to do, for example), schedutil as opposed to other CPUFreq governors at least [requests]() frequencies calculated using the utilization signals. Consequently, the only sane governor to use together with EAS is schedutil, because it is the only one providing some degree of consistency between frequency requests and energy predictions.

> 尽管在实践中很难提供关于该假设准确性的硬保证（例如，因为硬件可能不会按照要求执行），但与其他 CPUFreq 调速器相比，schedutil 至少[请求]（）使用利用信号计算的频率。因此，与 EAS 一起使用的唯一合理的调速器是 schedutil，因为它是唯一一个在频率请求和能量预测之间提供一定程度一致性的调速器。

Using EAS with any other governor than schedutil is not supported.

## 6.5 Scale-invariant utilization signals

In order to make accurate prediction across CPUs and for all performance states, EAS needs frequency-invariant and CPU-invariant PELT signals. These can be obtained using the architecture-defined arch_scale{cpu,freq}capacity() callbacks.

> 为了在 CPU 之间和所有性能状态下进行准确的预测，EAS 需要频率不变和 CPU 不变的 PELT 信号。这些可以使用体系结构定义的 arch_scale 获得{cpu,freq}capacity（）回调。

Using EAS on a platform that doesn't implement these two callbacks is not supported.

> 不支持在不实现这两个回调的平台上使用 EAS。

## 6.6 Multithreading (SMT)

EAS in its current form is SMT unaware and is not able to leverage multithreaded hardware to save energy. EAS considers threads as independent CPUs, which can actually be counter-productive for both performance and energy.

> 目前形式的 EAS 是 SMT 不知道的，不能利用多线程硬件来节省能源。EAS 将线程视为独立的 CPU，这实际上会对性能和能量产生反作用。

EAS on SMT is not supported.
