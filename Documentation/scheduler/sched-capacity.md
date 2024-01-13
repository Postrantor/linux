---
tip: translate by baidu@2024-01-13 11:27:36
title: Capacity Aware Scheduling
---

# 1. CPU Capacity

## 1.1 Introduction

Conventional, homogeneous SMP platforms are composed of purely identical CPUs. Heterogeneous platforms on the other hand are composed of CPUs with different performance characteristics - on such platforms, not all CPUs can be considered equal.

> 传统的同类 SMP 平台由完全相同的 CPU 组成。另一方面，异构平台是由具有不同性能特征的 CPU 组成的——在这样的平台上，并不是所有的 CPU 都可以被认为是平等的。

CPU capacity is a measure of the performance a CPU can reach, normalized against the most performant CPU in the system. Heterogeneous systems are also called asymmetric CPU capacity systems, as they contain CPUs of different capacities.

> CPU 容量是衡量 CPU 可以达到的性能的指标，与系统中性能最高的 CPU 进行归一化。异构系统也称为非对称 CPU 容量系统，因为它们包含不同容量的 CPU。

Disparity in maximum attainable performance (IOW in maximum CPU capacity) stems from two factors:

> 最大可实现性能(最大 CPU 容量的 IOW)的差异源于两个因素：

- not all CPUs may have the same microarchitecture (µarch).

- with Dynamic Voltage and Frequency Scaling (DVFS), not all CPUs may be physically able to attain the higher Operating Performance Points (OPP).

> - 通过动态电压和频率缩放(DVFS)，并非所有 CPU 都能够在物理上获得更高的操作性能点(OPP)。

Arm big.LITTLE systems are an example of both. The big CPUs are more performance-oriented than the LITTLE ones (more pipeline stages, bigger caches, smarter predictors, etc), and can usually reach higher OPPs than the LITTLE ones can.

> 手臂很大。小系统就是两者的一个例子。大 CPU 比小 CPU 更注重性能(更多的流水线级、更大的缓存、更智能的预测器等)，并且通常可以达到比小 CPU 高的 OPP。

CPU performance is usually expressed in Millions of Instructions Per Second (MIPS), which can also be expressed as a given amount of instructions attainable per Hz, leading to:

> CPU 性能通常以每秒数百万条指令(MIPS)表示，也可以表示为每 Hz 可获得的给定数量的指令，从而导致：

    capacity(cpu) = work_per_hz(cpu) * max_freq(cpu)

## 1.2 Scheduler terms

Two different capacity values are used within the scheduler. A CPU\'s `original capacity` is its maximum attainable capacity, i.e. its maximum attainable performance level. This original capacity is returned by the function arch_scale_cpu_capacity(). A CPU\'s `capacity` is its `original capacity` to which some loss of available performance (e.g. time spent handling IRQs) is subtracted.

> 调度程序中使用了两个不同的容量值。CPU 的“原始容量”是指其可达到的最大容量，即可达到的最高性能水平。此原始容量由函数 arch_scale_cpu_capacity()返回。CPU 的“容量”是其“原始容量”，减去一些可用性能损失(如处理 IRQ 所花费的时间)。

Note that a CPU\'s `capacity` is solely intended to be used by the CFS class, while `original capacity` is class-agnostic. The rest of this document will use the term `capacity` interchangeably with `original capacity` for the sake of brevity.

> 请注意，CPU 的“容量”仅用于 CFS 类，而“原始容量”与类无关。为简洁起见，本文件其余部分将“能力”一词与“原始能力”互换使用。

## 1.3 Platform examples

### 1.3.1 Identical OPPs

Consider an hypothetical dual-core asymmetric CPU capacity system where

- work_per_hz(CPU0) = W
- work_per_hz(CPU1) = W/2
- all CPUs are running at the same fixed frequency

By the above definition of capacity:

- capacity(CPU0) = C
- capacity(CPU1) = C/2

To draw the parallel with Arm big.LITTLE, CPU0 would be a big while CPU1 would be a LITTLE.

> 用大臂画平行线。小，CPU0 会很大，而 CPU1 会很小。

With a workload that periodically does a fixed amount of work, you will get an execution trace like so:

> 对于周期性地完成固定工作量的工作负载，您将获得如下执行跟踪：

```
    CPU0 work ^
              |     ____                ____                ____
              |    |    |              |    |              |    |
              +----+----+----+----+----+----+----+----+----+----+-> time

    CPU1 work ^
              |     _________           _________           ____
              |    |         |         |         |         |
              +----+----+----+----+----+----+----+----+----+----+-> time
```

CPU0 has the highest capacity in the system (C), and completes a fixed amount of work W in T units of time. On the other hand, CPU1 has half the capacity of CPU0, and thus only completes W/2 in T.

> CPU0 在系统(C)中具有最高的容量，并且以 T 个时间单位完成固定量的工作 W。另一方面，CPU1 具有 CPU0 的一半容量，因此仅完成 T 中的 W/2。

### 1.3.2 Different max OPPs

Usually, CPUs of different capacity values also have different maximum OPPs. Consider the same CPUs as above (i.e. same work_per_hz()) with:

> 通常，不同容量值的 CPU 也具有不同的最大 OPP。考虑与上述相同的 CPU(即相同的 work_per_hz())，其中：

- max_freq(CPU0) = F
- max_freq(CPU1) = 2/3 \* F

This yields:

- capacity(CPU0) = C
- capacity(CPU1) = C/3

Executing the same workload as described in 1.3.1, which each CPU running at its maximum frequency results in:

> 执行 1.3.1 中所述的相同工作负载，每个 CPU 以其最大频率运行会导致：

```
    CPU0 work ^
              |     ____                ____                ____
              |    |    |              |    |              |    |
              +----+----+----+----+----+----+----+----+----+----+-> time

                               workload on CPU1
    CPU1 work ^
              |     ______________      ______________      ____
              |    |              |    |              |    |
              +----+----+----+----+----+----+----+----+----+----+-> time
```

## 1.4 Representation caveat

It should be noted that having a _single_ value to represent differences in CPU performance is somewhat of a contentious point. The relative performance difference between two different µarchs could be X% on integer operations, Y% on floating point operations, Z% on branches, and so on. Still, results using this simple approach have been satisfactory for now.

> 应该注意的是，使用*single*值来表示 CPU 性能的差异在某种程度上是有争议的。两个不同的 µarch 之间的相对性能差异可能是整数运算的 X%、浮点运算的 Y%、分支运算的 Z%等等。不过，目前使用这种简单方法的结果是令人满意的。

# 2. Task utilization

## 2.1 Introduction

Capacity aware scheduling requires an expression of a task\'s requirements with regards to CPU capacity. Each scheduler class can express this differently, and while task utilization is specific to CFS, it is convenient to describe it here in order to introduce more generic concepts.

> 容量感知调度需要表达任务对 CPU 容量的要求。每个调度器类都可以用不同的方式表达这一点，虽然任务利用率是特定于 CFS 的，但为了引入更通用的概念，在这里描述它是很方便的。

Task utilization is a percentage meant to represent the throughput requirements of a task. A simple approximation of it is the task\'s duty cycle, i.e.:

> 任务利用率是表示任务的吞吐量要求的百分比。它的一个简单近似值是任务的占空比，即：

    task_util(p) = duty_cycle(p)

On an SMP system with fixed frequencies, 100% utilization suggests the task is a busy loop. Conversely, 10% utilization hints it is a small periodic task that spends more time sleeping than executing. Variable CPU frequencies and asymmetric CPU capacities complexify this somewhat; the following sections will expand on these.

> 在具有固定频率的 SMP 系统上，100%的利用率表明任务是一个繁忙的循环。相反，10%的利用率表明这是一个小的周期性任务，睡眠时间比执行时间长。可变的 CPU 频率和不对称的 CPU 容量在一定程度上使这一点复杂化；以下部分将对此进行扩展。

## 2.2 Frequency invariance

One issue that needs to be taken into account is that a workload\'s duty cycle is directly impacted by the current OPP the CPU is running at. Consider running a periodic workload at a given frequency F:

> 需要考虑的一个问题是，工作负载的占空比直接受到 CPU 运行的当前 OPP 的影响。考虑以给定频率 F 运行周期性工作负载：

```
    CPU work ^
             |     ____                ____                ____
             |    |    |              |    |              |    |
             +----+----+----+----+----+----+----+----+----+----+-> time
```

This yields duty_cycle(p) == 25%.

Now, consider running the _same_ workload at frequency F/2:

```
    CPU work ^
             |     _________           _________           ____
             |    |         |         |         |         |
             +----+----+----+----+----+----+----+----+----+----+-> time
```

This yields duty_cycle(p) == 50%, despite the task having the exact same behaviour (i.e. executing the same amount of work) in both executions.

> 这产生 duty_cycle(p)==50%，尽管任务在两次执行中具有完全相同的行为(即执行相同的工作量)。

The task utilization signal can be made frequency invariant using the following formula:

> 可以使用以下公式使任务利用信号频率不变：

    task_util_freq_inv(p) = duty_cycle(p) * (curr_frequency(cpu) / max_frequency(cpu))

Applying this formula to the two examples above yields a frequency invariant task utilization of 25%.

> 将这个公式应用于上面的两个例子，产生 25%的频率不变任务利用率。

## 2.3 CPU invariance

CPU capacity has a similar effect on task utilization in that running an identical workload on CPUs of different capacity values will yield different duty cycles.

> CPU 容量对任务利用率的影响相似，因为在不同容量值的 CPU 上运行相同的工作负载将产生不同的占空比。

Consider the system described in 1.3.2., i.e.:

    - capacity(CPU0) = C
    - capacity(CPU1) = C/3

Executing a given periodic workload on each CPU at their maximum frequency would result in:

> 在每个 CPU 上以其最大频率执行给定的周期性工作负载将导致：

```
    CPU0 work ^
              |     ____                ____                ____
              |    |    |              |    |              |    |
              +----+----+----+----+----+----+----+----+----+----+-> time

    CPU1 work ^
              |     ______________      ______________      ____
              |    |              |    |              |    |
              +----+----+----+----+----+----+----+----+----+----+-> time
```

IOW,

- duty_cycle(p) == 25% if p runs on CPU0 at its maximum frequency
- duty_cycle(p) == 75% if p runs on CPU1 at its maximum frequency

The task utilization signal can be made CPU invariant using the following formula:

> 任务利用率信号可以使用以下公式使 CPU 不变：

    task_util_cpu_inv(p) = duty_cycle(p) * (capacity(cpu) / max_capacity)

with `max_capacity` being the highest CPU capacity value in the system. Applying this formula to the above example above yields a CPU invariant task utilization of 25%.

> 其中“max_capacity”是系统中的最高 CPU 容量值。将这个公式应用于上面的例子，可以得到 25%的 CPU 不变任务利用率。

## 2.4 Invariant task utilization

Both frequency and CPU invariance need to be applied to task utilization in order to obtain a truly invariant signal. The pseudo-formula for a task utilization that is both CPU and frequency invariant is thus, for a given task p:

> 为了获得真正不变的信号，需要将频率和 CPU 不变性都应用于任务利用率。因此，对于给定任务 p，CPU 和频率不变的任务利用率的伪公式为：

```
    curr_frequency(cpu)   capacity(cpu)
    task_util_inv(p) = duty_cycle(p) * ------------------- * -------------
    max_frequency(cpu)    max_capacity
```

In other words, invariant task utilization describes the behaviour of a task as if it were running on the highest-capacity CPU in the system, running at its maximum frequency.

> 换句话说，不变任务利用率描述了任务的行为，就好像它在系统中最高容量的 CPU 上运行，以其最大频率运行一样。

Any mention of task utilization in the following sections will imply its invariant form.

> 在下面的小节中，任何关于任务利用率的提及都意味着它的不变形式。

## 2.5 Utilization estimation

Without a crystal ball, task behaviour (and thus task utilization) cannot accurately be predicted the moment a task first becomes runnable. The CFS class maintains a handful of CPU and task signals based on the Per-Entity Load Tracking (PELT) mechanism, one of those yielding an _average_ utilization (as opposed to instantaneous).

> 如果没有水晶球，任务行为(以及任务利用率)就无法在任务首次可运行时准确预测。CFS 类基于每个实体负载跟踪(PELT)机制维护少数 CPU 和任务信号，其中一个机制产生*average*利用率(与瞬时相比)。

This means that while the capacity aware scheduling criteria will be written considering a "true" task utilization (using a crystal ball), the implementation will only ever be able to use an estimator thereof.

> 这意味着，虽然容量感知调度标准将在考虑“真实”任务利用率(使用水晶球)的情况下编写，但实现将只能使用其估计器。

# 3. Capacity aware scheduling requirements

## 3.1 CPU capacity

Linux cannot currently figure out CPU capacity on its own, this information thus needs to be handed to it. Architectures must define arch_scale_cpu_capacity() for that purpose.

> Linux 目前无法单独计算 CPU 容量，因此需要将这些信息交给它。架构必须为此定义 arch_scale_CPU_capacity()。

The arm, arm64, and RISC-V architectures directly map this to the arch_topology driver CPU scaling data, which is derived from the capacity-dmips-mhz CPU binding; see Documentation/devicetree/bindings/cpu/cpu-capacity.txt.

> arm、arm64 和 RISC-V 架构直接将其映射到 arch_topology 驱动程序 CPU 缩放数据，该数据源自容量 dmips-mhz-CPU 绑定；请参阅文档/devicetree/bindings/cpu/cpu-caccability.txt。

## 3.2 Frequency invariance

As stated in 2.2, capacity-aware scheduling requires a frequency-invariant task utilization. Architectures must define arch_scale_freq_capacity(cpu) for that purpose.

> 如 2.2 中所述，容量感知调度需要频率不变的任务利用率。体系结构必须为此定义 arch_scale_freq_capacity(cpu)。

Implementing this function requires figuring out at which frequency each CPU have been running at. One way to implement this is to leverage hardware counters whose increment rate scale with a CPU\'s current frequency (APERF/MPERF on x86, AMU on arm64). Another is to directly hook into cpufreq frequency transitions, when the kernel is aware of the switched-to frequency (also employed by arm/arm64).

> 实现此功能需要弄清楚每个 CPU 的运行频率。实现此功能的一种方法是利用硬件计数器，其递增速率与 CPU 的当前频率成比例(x86 上的 APERF/MPERF，arm64 上的 AMU)。另一种是当内核知道切换到频率(也由 arm/arm64 使用)时，直接挂入 cpufreq 频率转换。

# 4. Scheduler topology

During the construction of the sched domains, the scheduler will figure out whether the system exhibits asymmetric CPU capacities. Should that be the case:

> 在构建 sched 域的过程中，调度器将判断系统是否表现出不对称的 CPU 容量。如果是这样的话：

- The sched_asym_cpucapacity static key will be enabled.
- The SD_ASYM_CPUCAPACITY_FULL flag will be set at the lowest sched_domain level that spans all unique CPU capacity values.
- The SD_ASYM_CPUCAPACITY flag will be set for any sched_domain that spans CPUs with any range of asymmetry.

> - SD_ASYM_CPUCAPACITY_FULL 标志将设置在跨越所有唯一 CPU 容量值的最低 sched_domain 级别。
> - 将为跨越具有任何不对称范围的 CPU 的任何 sched_domain 设置 SD_ASYM_CPUCAPITY 标志。

The sched*asym_cpucapacity static key is intended to guard sections of code that cater to asymmetric CPU capacity systems. Do note however that said key is \_system-wide*. Imagine the following setup using cpusets:

> sched*asym_cpucapacity 静态密钥旨在保护满足不对称 CPU 容量系统的代码段。但是，请注意，所说的关键是\_system-wide*。想象一下使用 cpusets 的以下设置：

```
    capacity    C/2          C
              ________    ________
             /        \  /        \
    CPUs     0  1  2  3  4  5  6  7
             \__/  \______________/
    cpusets   cs0         cs1
```

Which could be created via:

```sh
mkdir /sys/fs/cgroup/cpuset/cs0
echo 0-1 > /sys/fs/cgroup/cpuset/cs0/cpuset.cpus
echo 0 > /sys/fs/cgroup/cpuset/cs0/cpuset.mems

mkdir /sys/fs/cgroup/cpuset/cs1
echo 2-7 > /sys/fs/cgroup/cpuset/cs1/cpuset.cpus
echo 0 > /sys/fs/cgroup/cpuset/cs1/cpuset.mems

echo 0 > /sys/fs/cgroup/cpuset/cpuset.sched_load_balance
```

Since there _is_ CPU capacity asymmetry in the system, the sched_asym_cpucapacity static key will be enabled. However, the sched_domain hierarchy of CPUs 0-1 spans a single capacity value: SD_ASYM_CPUCAPACITY isn\'t set in that hierarchy, it describes an SMP island and should be treated as such.

> 由于系统中存在\_is_CPU 容量不对称，将启用 sched_asm_cpucapacity 静态密钥。然而，CPU 0-1 的 sched_domain 层次结构跨越单个容量值：SD_ASYM_CPUCAPITATION 没有在该层次结构中设置，它描述了 SMP 岛，应该按此方式处理。

Therefore, the \'canonical\' pattern for protecting codepaths that cater to asymmetric CPU capacities is to:

> 因此，保护满足不对称 CPU 容量的代码路径的“经典”模式是：

- Check the sched_asym_cpucapacity static key
- If it is enabled, then also check for the presence of SD_ASYM_CPUCAPACITY in the sched_domain hierarchy (if relevant, i.e. the codepath targets a specific CPU or group thereof)

> - 如果已启用，则还检查调度域层次结构中是否存在 SD_ASYM_CPUCAPITATION(如果相关，即代码路径以特定 CPU 或其组为目标)

# 5. Capacity aware scheduling implementation

## 5.1 CFS

### 5.1.1 Capacity fitness

The main capacity scheduling criterion of CFS is:

    task_util(p) < capacity(task_cpu(p))

This is commonly called the capacity fitness criterion, i.e. CFS must ensure a task "fits" on its CPU. If it is violated, the task will need to achieve more work than what its CPU can provide: it will be CPU-bound.

> 这通常被称为容量适应度标准，即 CFS 必须确保任务“适合”其 CPU。如果违反了它，任务将需要完成比其 CPU 所能提供的更多的工作：它将被 CPU 绑定。

Furthermore, uclamp lets userspace specify a minimum and a maximum utilization value for a task, either via sched_setattr() or via the cgroup interface (see Documentation/admin-guide/cgroup-v2.rst). As its name imply, this can be used to clamp task_util() in the previous criterion.

> 此外，uclamp 允许用户空间通过 sched_settr()或 cgroup 接口(请参阅 Documentation/admin guide/cgroup-v2.rst)为任务指定最小和最大利用率值。顾名思义，这可以用于在前面的标准中钳制 task_util()。

### 5.1.2 Wakeup CPU selection

CFS task wakeup CPU selection follows the capacity fitness criterion described above. On top of that, uclamp is used to clamp the task utilization values, which lets userspace have more leverage over the CPU selection of CFS tasks. IOW, CFS wakeup CPU selection searches for a CPU that satisfies:

> CFS 任务唤醒 CPU 的选择遵循上述容量适应度标准。最重要的是，uclamp 用于钳制任务利用率值，这让用户空间对 CFS 任务的 CPU 选择有更多的影响力。IOW，CFS 唤醒 CPU 选择搜索满足以下条件的 CPU：

    clamp(task_util(p), task_uclamp_min(p), task_uclamp_max(p)) < capacity(cpu)

By using uclamp, userspace can e.g. allow a busy loop (100% utilization) to run on any CPU by giving it a low uclamp.max value. Conversely, it can force a small periodic task (e.g. 10% utilization) to run on the highest-performance CPUs by giving it a high uclamp.min value.

> 通过使用 uclamp，用户空间可以通过给任何 CPU 一个低 uclamp.max 值来允许繁忙循环(100%利用率)在任何 CPU 上运行。相反，它可以通过给一个高 uclamp.min 值来强制一个小的周期性任务(例如 10%的利用率)在最高性能的 CPU 上运行。

::: note
::: title
Note
:::

Wakeup CPU selection in CFS can be eclipsed by Energy Aware Scheduling (EAS), which is described in Documentation/scheduler/sched-energy.rst.

> CFS 中的唤醒 CPU 选择可以被能量感知调度(EAS)所掩盖，该调度在 Documentation/scheller/sched-Energy.rst 中进行了描述。
> :::

### 5.1.3 Load balancing

A pathological case in the wakeup CPU selection occurs when a task rarely sleeps, if at all - it thus rarely wakes up, if at all. Consider:

> 当任务很少睡眠(如果有的话)时，就会出现唤醒 CPU 选择的病态情况，因此它很少醒来(如果有的时候)。考虑：

```
    w == wakeup event

    capacity(CPU0) = C
    capacity(CPU1) = C / 3

                             workload on CPU0
    CPU work ^
             |     _________           _________           ____
             |    |         |         |         |         |
             +----+----+----+----+----+----+----+----+----+----+-> time
                  w                   w                   w

                             workload on CPU1
    CPU work ^
             |     ____________________________________________
             |    |
             +----+----+----+----+----+----+----+----+----+----+->
                  w
```

This workload should run on CPU0, but if the task either:

- was improperly scheduled from the start (inaccurate initial utilization estimation)
- was properly scheduled from the start, but suddenly needs more processing power

then it might become CPU-bound, IOW `task_util(p) > capacity(task_cpu(p))`; the CPU capacity scheduling criterion is violated, and there may not be any more wakeup event to fix this up via wakeup CPU selection.

> 则它可能成为 CPU 绑定，IOW `task_util(p)>capacity(task_CPU(p))`；违反了 CPU 容量调度标准，并且可能不再有任何唤醒事件来通过唤醒 CPU 选择来解决此问题。

Tasks that are in this situation are dubbed "misfit" tasks, and the mechanism put in place to handle this shares the same name. Misfit task migration leverages the CFS load balancer, more specifically the active load balance part (which caters to migrating currently running tasks). When load balance happens, a misfit active load balance will be triggered if a misfit task can be migrated to a CPU with more capacity than its current one.

> 这种情况下的任务被称为“不匹配”任务，用于处理这种情况的机制同名。Misfit 任务迁移利用了 CFS 负载均衡器，更具体地说是活动负载平衡部分(用于迁移当前运行的任务)。当负载平衡发生时，如果一个不匹配的任务可以迁移到比其当前容量更大的 CPU，则会触发不匹配的活动负载平衡。

## 5.2 RT

### 5.2.1 Wakeup CPU selection

RT task wakeup CPU selection searches for a CPU that satisfies:

    task_uclamp_min(p) <= capacity(task_cpu(cpu))

while still following the usual priority constraints. If none of the candidate CPUs can satisfy this capacity criterion, then strict priority based scheduling is followed and CPU capacities are ignored.

> 同时仍然遵循通常的优先级约束。如果没有一个候选 CPU 能够满足该容量标准，则遵循严格的基于优先级的调度，并且忽略 CPU 容量。

## 5.3 DL

### 5.3.1 Wakeup CPU selection

DL task wakeup CPU selection searches for a CPU that satisfies:

    task_bandwidth(p) < capacity(task_cpu(p))

while still respecting the usual bandwidth and deadline constraints. If none of the candidate CPUs can satisfy this capacity criterion, then the task will remain on its current CPU.

> 同时仍然遵守通常的带宽和最后期限限制。如果没有一个候选 CPU 能够满足此容量标准，则任务将保留在其当前 CPU 上。
