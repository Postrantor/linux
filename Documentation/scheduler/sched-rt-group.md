---
tip: translate by baidu@2024-01-17 08:59:24
title: Real-Time group scheduling
---

- [0. WARNING](#0-warning)
- [1. Overview](#1-overview)
  - [1.1 The problem](#11-the-problem)
  - [1.2 The solution](#12-the-solution)
- [2. The Interface](#2-the-interface)
  - [2.1 System wide settings](#21-system-wide-settings)
  - [2.2 Default behaviour](#22-default-behaviour)
  - [2.3 Basis for grouping tasks](#23-basis-for-grouping-tasks)
- [3. Future plans](#3-future-plans)

# 0. WARNING

Fiddling with these settings can result in an unstable system, the knobs are root only and assumes root knows what he is doing.

> 篡改这些设置可能会导致系统不稳定，旋钮只是 root 用户，并假设 root 用户知道自己在做什么。

Most notable:

- very small values in `sched_rt_period_us` can result in an unstable system when the period is smaller than either the available hrtimer resolution, or the time it takes to handle the budget refresh itself.

> -当周期小于可用的 hrtimer 分辨率或处理预算刷新本身所需的时间时，“sched_rt_period_us”中的非常小的值可能会导致系统不稳定。

- very small values in `sched_rt_runtime_us` can result in an unstable system when the runtime is so small the system has difficulty making forward progress (NOTE: the migration thread and kstopmachine both are real-time processes).

> -“sched_rrt_runtime_us”中的非常小的值可能会导致系统不稳定，因为运行时太小，系统很难向前推进（注意：迁移线程和 kstopmachine 都是实时进程）。

# 1. Overview

## 1.1 The problem

Real-time scheduling is all about determinism, a group has to be able to rely on the amount of bandwidth (eg. CPU time) being constant. In order to schedule multiple groups of real-time tasks, each group must be assigned a fixed portion of the CPU time available. Without a minimum guarantee a real-time group can obviously fall short. A fuzzy upper limit is of no use since it cannot be relied upon. Which leaves us with just the single fixed portion.

> 实时调度是关于确定性的，一个组必须能够依赖于恒定的带宽量（例如 CPU 时间）。为了安排多组实时任务，必须为每组分配固定的 CPU 可用时间。如果没有最低限度的保证，实时组显然会功亏一篑。模糊的上限是没有用的，因为它不能被依赖。这只剩下一个固定的部分。

## 1.2 The solution

CPU time is divided by means of specifying how much time can be spent running in a given period. We allocate this "run time" for each real-time group which the other real-time groups will not be permitted to use.

> CPU 时间是通过指定在给定时间段内可以花费多少时间来划分的。我们为每个实时组分配这个“运行时间”，其他实时组将不被允许使用。

Any time not allocated to a real-time group will be used to run normal priority tasks (SCHED_OTHER). Any allocated run time not used will also be picked up by SCHED_OTHER.

> 任何未分配给实时组的时间都将用于运行正常优先级任务（SCHED_OTHER）。任何未使用的已分配运行时间也将由 SCHED_OTHER 获取。

Let\'s consider an example: a frame fixed real-time renderer must deliver 25 frames a second, which yields a period of 0.04s per frame. Now say it will also have to play some music and respond to input, leaving it with around 80% CPU time dedicated for the graphics. We can then give this group a run time of `0.8 * 0.04s = 0.032s`.

> 让我们考虑一个例子：一个固定帧的实时渲染器必须每秒提供 25 帧，每帧产生 0.04 秒的周期。现在说，它还必须播放一些音乐和响应输入，剩下大约 80%的 CPU 时间用于图形。然后我们可以给这个组一个运行时间‘0.8\*0.04s=0.032s’。

This way the graphics group will have a 0.04s period with a 0.032s run time limit. Now if the audio thread needs to refill the DMA buffer every 0.005s, but needs only about 3% CPU time to do so, it can do with a `0.03 * 0.005s = 0.00015s`. So this group can be scheduled with a period of 0.005s and a run time of 0.00015s.

> 这样，图形组将有一个 0.04 秒的周期，运行时间限制为 0.032 秒。现在，如果音频线程需要每 0.005 秒重新填充 DMA 缓冲区，但只需要大约 3%的 CPU 时间，它可以用“0.03\*0.005 秒=0.00015 秒”来完成。因此，可以用 0.005 秒的周期和 0.00015 秒的运行时间来调度该组。

The remaining CPU time will be used for user input and other tasks. Because real-time tasks have explicitly allocated the CPU time they need to perform their tasks, buffer underruns in the graphics or audio can be eliminated.

> 剩余的 CPU 时间将用于用户输入和其他任务。由于实时任务明确分配了执行任务所需的 CPU 时间，因此可以消除图形或音频中的缓冲区不足。

NOTE: the above example is not fully implemented yet. We still lack an EDF scheduler to make non-uniform periods usable.

> 注意：上述示例尚未完全实现。我们仍然缺乏一个 EDF 调度器来使非均匀周期可用。

# 2. The Interface

## 2.1 System wide settings

The system wide settings are configured under the /proc virtual file system:

> 系统范围的设置是在/proc 虚拟文件系统下配置的：

```
/proc/sys/kernel/`sched_rt_period_us`:
```

: The scheduling period that is equivalent to 100% CPU bandwidth.

```
/proc/sys/kernel/sched_rt_runtime_us:
```

: A global limit on how much time real-time scheduling may use. This is always less or equal to the period_us, as it denotes the time allocated from the period_us for the real-time tasks. Even without CONFIG_RT_GROUP_SCHED enabled, this will limit time reserved to real-time processes. With CONFIG_RT_GROUP_SCHED=y it signifies the total bandwidth available to all real-time groups.

> ：实时调度可能使用的时间的全局限制。这总是小于或等于 period_us，因为它表示从 period_us 分配给实时任务的时间。即使没有启用 CONFIG_RT_GROUP_SCHED，这也会限制为实时进程保留的时间。CONFIG_RT_GROUP_SCHED=y 表示所有实时组可用的总带宽。

- Time is specified in us because the interface is s32. This gives an operating range from 1us to about 35 minutes.

> -时间是由我们指定的，因为接口是 s32。这提供了从 1us 到大约 35 分钟的操作范围。

- `sched_rt_period_us` takes values from 1 to INT_MAX.
- `sched_rt_runtime_us` takes values from -1 to `sched_rt_period_us`.
- A run time of -1 specifies runtime == period, ie. no limit.

## 2.2 Default behaviour

The default values for `sched_rt_period_us` (1000000 or 1s) and `sched_rt_runtime_us` (950000 or 0.95s). This gives 0.05s to be used by `SCHED_OTHER` (non-RT tasks). These defaults were chosen so that a run-away real-time tasks will not lock up the machine but leave a little time to recover it. By setting runtime to -1 you\'d get the old behaviour back.

> “sched_rt_period_us”（1000000 或 1s）和“sched_rrt_runtime_us”（950000 或 0.95s）的默认值。这为“sched_OTHER”（非 rt 任务）提供了 0.05 秒的使用时间。选择这些默认值是为了使运行中的实时任务不会锁定机器，而是留出一点时间进行恢复。通过将运行时设置为-1，您将恢复原来的行为。

By default all bandwidth is assigned to the root group and new groups get the period from `/proc/sys/kernel/sched_rt_period_us` and a run time of 0. If you want to assign bandwidth to another group, reduce the root group\'s bandwidth and assign some or all of the difference to another group.

> 默认情况下，所有带宽都分配给根组，新组从“/proc/sys/kernel/sched_rt_period_us”获得周期，运行时间为 0。如果要将带宽分配给另一个组，请减少根组的带宽，并将部分或全部差异分配给其他组。

Real-time group scheduling means you have to assign a portion of total CPU bandwidth to the group before it will accept real-time tasks. Therefore you will not be able to run real-time tasks as any user other than root until you have done that, even if the user has the rights to run processes with real-time priority!

> 实时组调度意味着在组接受实时任务之前，必须将总 CPU 带宽的一部分分配给组。因此，在完成此操作之前，您将无法以 root 以外的任何用户身份运行实时任务，即使该用户有权以实时优先级运行进程！

## 2.3 Basis for grouping tasks

Enabling CONFIG_RT_GROUP_SCHED lets you explicitly allocate real CPU bandwidth to task groups.

> 启用 CONFIG_RT_GROUP_SCHED 可以显式地将实际 CPU 带宽分配给任务组。

This uses the cgroup virtual file system and "cgroup/cpu.rt_runtime_us" to control the CPU time reserved for each control group.

> 这使用 cgroup 虚拟文件系统和“cgroup/cmp.rt_runtime_us”来控制为每个控制组保留的 cpu 时间。

For more information on working with control groups, you should read "Documentation/admin-guide/cgroup-v1/cgroups.rst" as well.

> 有关使用控制组的更多信息，您还应该阅读“Documentation/admin guide/cgroup-v1/cgroups.rst”。

Group settings are checked against the following limits in order to keep the configuration schedulable:

> 根据以下限制检查组设置，以保持配置的可调度性：

> S[um](){i} [runtime](){i} / global_period = global_runtime / global_period

For now, this can be simplified to just the following (but see Future plans):

> 目前，这可以简化为以下内容（但请参阅未来计划）：

> S[um](){i} [runtime](){i} = global_runtime

# 3. Future plans

There is work in progress to make the scheduling period for each group ("cgroup/cpu.rt_period_us") configurable as well.

> 还有一些工作正在进行中，以使每个组的调度周期（“cgroup/cpu.rt_period_us”）也可配置。

The constraint on the period is that a subgroup must have a smaller or equal period to its parent. But realistically its not very useful [yet]() as its prone to starvation without deadline scheduling.

> 对周期的约束是，子组的周期必须小于或等于其父组的周期。但实际上，它还不是很有用，因为如果没有最后期限的安排，它很容易饿死。

Consider two sibling groups A and B; both have 50% bandwidth, but A\'s period is twice the length of B\'s.

> 考虑两个兄弟组 A 和 B；两者都有 50%的带宽，但 A 的周期是 B 的两倍。

- group A: period=100000us, runtime=50000us

  > - this runs for 0.05s once every 0.1s

- group B: period= 50000us, runtime=25000us

  > - this runs for 0.025s twice every 0.1s (or once every 0.05 sec).

This means that currently a while (1) loop in A will run for the full period of B and can starve B\'s tasks (assuming they are of lower priority) for a whole period.

> 这意味着当前 a 中的 while（1）循环将在 B 的整个周期内运行，并可能在整个周期内耗尽 B 的任务（假设它们的优先级较低）。

The next project will be `SCHED_EDF` (Earliest Deadline First scheduling) to bring full deadline scheduling to the linux kernel. Deadline scheduling the above groups and treating end of the period as a deadline will ensure that they both get their allocated time.

> 下一个项目将是“SCHED_EDF”（最早截止日期优先调度），为 linux 内核带来完整的截止日期调度。最后期限安排上述小组，并将期末视为最后期限，将确保他们都能获得分配的时间。

Implementing `SCHED_EDF` might take a while to complete. Priority Inheritance is the biggest challenge as the current linux PI infrastructure is geared towards the limited static priority levels 0-99. With deadline scheduling you need to do deadline inheritance (since priority is inversely proportional to the deadline delta (deadline - now)).

> 实现“SCHED_EDF”可能需要一段时间才能完成。优先级继承是最大的挑战，因为当前的 linux PI 基础结构面向有限的静态优先级 0-99。对于最后期限调度，您需要进行最后期限继承（因为优先级与最后期限 delta（现在的最后期限）成反比）。

This means the whole PI machinery will have to be reworked - and that is one of the most complex pieces of code we have.

> 这意味着整个 PI 机制将不得不重新设计——这是我们拥有的最复杂的代码之一。
