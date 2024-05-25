---
tip: translate by baidu@2024-01-17 20:17:55
title: Schedutil
---

::: note
::: title
Note
:::

All this assumes a linear relation between frequency and work capacity, we know this is flawed, but it is the best workable approximation.

> 所有这些都假设频率和工作能力之间存在线性关系，我们知道这是有缺陷的，但这是最好的可行近似。
> :::

# PELT (Per Entity Load Tracking)

With PELT we track some metrics across the various scheduler entities, from individual tasks to task-group slices to CPU runqueues. As the basis for this we use an Exponentially Weighted Moving Average (EWMA), each period (1024us) is decayed such that `y^32 = 0.5`. That is, the most recent 32ms contribute half, while the rest of history contribute the other half.

> 使用 PELT，我们可以跟踪各种调度器实体的一些指标，从单个任务到任务组切片再到 CPU 运行队列。作为这一点的基础，我们使用指数加权移动平均线(EWMA)，每个周期(1024us)衰减，使得 `y^32=0.5`。也就是说，最近的 32ms 贡献了一半，而历史的其余部分贡献了另一半。

Specifically:

> `ewma_sum(u) := u_0 + u_1*y + u_2*y^2 + ewma(u) = ewma_sum(u) / ewma_sum(1)`

Since this is essentially a progression of an infinite geometric series, the results are composable, that is `ewma(A) + ewma(B) = ewma(A+B)`. This property is key, since it gives the ability to recompose the averages when tasks move around.

> 由于这本质上是无限几何级数的级数，因此结果是可组合的，即`ewma(a)+ewma(B)=ewma(a+B)`。此属性是关键，因为它提供了在任务移动时重新计算平均值的能力。

Note that blocked tasks still contribute to the aggregates (task-group slices and CPU runqueues), which reflects their expected contribution when they resume running.

> 请注意，被阻止的任务仍然对聚合(任务组切片和 CPU 运行队列)有贡献，这反映了它们在恢复运行时的预期贡献。

Using this we track 2 key metrics: 'running' and 'runnable'. 'Running' reflects the time an entity spends on the CPU, while 'runnable' reflects the time an entity spends on the runqueue. When there is only a single task these two metrics are the same, but once there is contention for the CPU 'running' will decrease to reflect the fraction of time each task spends on the CPU while 'runnable' will increase to reflect the amount of contention.

> 使用它，我们跟踪两个关键指标：“运行” 和 “可运行” Running 反映实体在 CPU 上花费的时间，而 runnable 反映实体在 runqueue 上花费的时光。当只有一个任务时，这两个指标是相同的，但一旦出现争用，CPU 的 “运行” 将减少以反映每个任务在 CPU 上花费的时间，而 “可运行” 将增加以反映争用量。

For more detail see: "kernel/sched/pelt.c"

# Frequency / CPU Invariance

Because consuming the CPU for 50% at 1GHz is not the same as consuming the CPU for 50% at 2GHz, nor is running 50% on a LITTLE CPU the same as running 50% on a big CPU, we allow architectures to scale the time delta with two ratios, one Dynamic Voltage and Frequency Scaling (DVFS) ratio and one microarch ratio.

> 因为在 1GHz 下消耗 50%的 CPU 与在 2GHz 下消耗一半的 CPU 不同，在小 CPU 上运行 50%也与在大 CPU 上运行一半不同，所以我们允许体系结构使用两种比率来缩放时间增量，一种是动态电压和频率缩放(DVFS)比率，另一种是微芯片比率。

For simple DVFS architectures (where software is in full control) we trivially compute the ratio as:

> 对于简单的 DVFS 架构(其中软件处于完全控制中)，我们简单地将比率计算为：

```
    f_cur
    r_dvfs := -----
        f_max
```

For more dynamic systems where the hardware is in control of DVFS we use hardware counters (Intel APERF/MPERF, ARMv8.4-AMU) to provide us this ratio. For Intel specifically, we use:

> 对于硬件控制 DVFS 的更动态的系统，我们使用硬件计数器(Intel APERF/MPERF，ARMv8.4-AMU)来提供此比率。具体针对英特尔，我们使用：

```
    APERF
    f_cur := ----- * P0
    MPERF

      4C-turbo;  if available and turbo enabled
    f_max := { 1C-turbo;  if turbo enabled
      P0;    otherwise

                 f_cur
    r_dvfs := min( 1, ----- )
                 f_max
```

We pick 4C turbo over 1C turbo to make it slightly more sustainable.

`r_cpu` is determined as the ratio of highest performance level of the current CPU vs the highest performance level of any other CPU in the system.

> `r_cpu` 被确定为当前 cpu 的最高性能水平与系统中任何其他 cpu 的最高功能水平的比率。

> `r_tot = r_dvfs * r_cpu`

The result is that the above 'running' and 'runnable' metrics become invariant of DVFS and CPU type. IOW. we can transfer and compare them between CPUs.

> 结果是，上面的 “正在运行” 和 “可运行” 指标变为 DVFS 和 CPU 类型不变。换句话说我们可以在 CPU 之间传输和比较它们。

For more detail see:

> - kernel/sched/pelt.h:update_rq_clock_pelt()
> - arch/x86/kernel/smpboot.c:\"APERF/MPERF frequency ratio computation.\"
> - Documentation/scheduler/sched-capacity.rst:\"1. CPU Capacity + 2. Task utilization\"

# UTIL_EST / UTIL_EST_FASTUP

Because periodic tasks have their averages decayed while they sleep, even though when running their expected utilization will be the same, they suffer a (DVFS) ramp-up after they are running again.

> 因为周期性任务的平均值在睡眠时会衰减，即使在运行时它们的预期利用率相同，但它们在再次运行后会出现(DVFS)上升。

To alleviate this (a default enabled option) `UTIL_EST` drives an Infinite Impulse Response (IIR) EWMA with the 'running' value on dequeue -- when it is highest. A further default enabled option `UTIL_EST_FASTUP` modifies the IIR filter to instantly increase and only decay on decrease.

> 为了缓解这种情况(默认启用的选项)，“UTIL_EST” 在出列时使用 “running” 值驱动无限脉冲响应(IIR)EWMA(当它最高时)。另一个默认启用的选项 “UTIL_EST_FASTUP” 将 IIR 过滤器修改为立即增加，并且仅在减少时衰减。

A further runqueue wide sum (of runnable tasks) is maintained of:

> util_est := Sum_t max( t_running, t_util_est_ewma )

For more detail see: kernel/sched/fair.c:util_est_dequeue()

# UCLAMP

It is possible to set effective `u_min` and `u_max` clamps on each CFS or RT task; the runqueue keeps an max aggregate of these clamps for all running tasks.

> 可以在每个 CFS 或 RT 任务上设置有效的“u_min”和“u_max”箝位；runqueue 为所有正在运行的任务保留这些 clamp 的最大聚合。

For more detail see: `include/uapi/linux/sched/types.h`

# Schedutil / DVFS

Every time the scheduler load tracking is updated (task wakeup, task migration, time progression) we call out to schedutil to update the hardware DVFS state.

> 每次更新调度程序负载跟踪(任务唤醒、任务迁移、时间进度)时，我们都会调用 schedutil 来更新硬件 DVFS 状态。

The basis is the CPU runqueue's 'running' metric, which per the above it is the frequency invariant utilization estimate of the CPU. From this we compute a desired frequency like:

> 其基础是 CPU 运行队列的“运行”度量，根据上述度量，它是 CPU 的频率不变利用率估计。由此，我们计算出所需的频率，如：

```
    max( running, util_est );  if UTIL_EST
    u_cfs := { running;           otherwise

      clamp( u_cfs + u_rt , u_min, u_max );    if UCLAMP_TASK
    u_clamp := { u_cfs + u_rt;                otherwise

    u := u_clamp + u_irq + u_dl;      [approx. see source for more detail]

    f_des := min( f_max, 1.25 u * f_max )
```

XXX IO-wait: when the update is due to a task wakeup from IO-completion we boost 'u' above.

> XXX IO 等待：当更新是由于 IO 完成后的任务唤醒时，我们将 “u” 提升到上面。

This frequency is then used to select a P-state/OPP or directly munged into a `CPPC` style request to the hardware.

> 然后，该频率被用于选择 P 状态/OPP，或者被直接调制成对硬件的 “CPPC” 式请求。

XXX: deadline tasks (Sporadic Task Model) allows us to calculate a hard `f_min` required to satisfy the workload.

> XXX： 最后期限任务(零星任务模型)允许我们计算满足工作负载所需的硬 `f_min`。

Because these callbacks are directly from the scheduler, the DVFS hardware interaction should be 'fast' and non-blocking. Schedutil supports rate-limiting DVFS requests for when hardware interaction is slow and expensive, this reduces effectiveness.

> 因为这些回调直接来自调度器，所以 DVFS 硬件交互应该是“快速”且无阻塞的。当硬件交互缓慢且昂贵时，Scheduleutil 支持速率限制 DVFS 请求，这会降低效率。

For more information see: kernel/sched/cpufreq_schedutil.c

# NOTES

- On low-load scenarios, where DVFS is most relevant, the 'running' numbers will closely reflect utilization.

> - 在低负载情况下，DVFS 最相关，“运行”数字将密切反映利用率。

- In saturated scenarios task movement will cause some transient dips, suppose we have a CPU saturated with 4 tasks, then when we migrate a task to an idle CPU, the old CPU will have a 'running' value of 0.75 while the new CPU will gain 0.25. This is inevitable and time progression will correct this. XXX do we still guarantee `f_max` due to no idle-time?

> -在饱和场景中，任务移动会导致一些短暂的下降，假设我们的 CPU 饱和了 4 个任务，那么当我们将任务迁移到空闲 CPU 时，旧 CPU 的“运行”值将为 0.75，而新 CPU 将获得 0.25。这是不可避免的，时间的推移将纠正这一点。XXX 由于没有空闲时间，我们还保证 `f_max` 吗？

- Much of the above is about avoiding DVFS dips, and independent DVFS domains having to re-learn / ramp-up when load shifts.

> - 以上大部分内容都是关于避免 DVFS 下降，以及当负载发生变化时，独立的 DVFS 域必须重新学习/提升。
