---
tip: translate by baidu@2024-01-13 11:20:54
title: CFS Bandwidth Control
---

::: note
::: title
Note
:::

This document only discusses CPU bandwidth control for `SCHED_NORMAL`. The `SCHED_RT` case is covered in "Documentation/scheduler/sched-rt-group.rst"

> 本文档仅讨论`SCHED_NORMAL`的 CPU 带宽控制。"Documentation/scheduler/sched-rt-group.rst" 中介绍了`SCHED_RT`的情况

:::

`CFS` bandwidth control is a `CONFIG_FAIR_GROUP_SCHED` extension which allows the specification of the maximum CPU bandwidth available to a group or hierarchy.

> `CFS`带宽控件是`CONFIG_FAIR_GROUP_SCHED`扩展名，允许对组或层次结构可用的最大 CPU 带宽规范。

The bandwidth allowed for a group is specified using a quota and period. Within each given \"period\" (microseconds), a task group is allocated up to \"quota\" microseconds of CPU time. That quota is assigned to per-cpu run queues in slices as threads in the cgroup become runnable. Once all quota has been assigned any additional requests for quota will result in those threads being throttled. Throttled threads will not be able to run again until the next period when the quota is replenished.

> 组允许的带宽是使用配额和时段指定的。在每个给定的`周期`(微秒)内，一个任务组被分配最多`配额`微秒的 CPU 时间。当 cgroup 中的线程变得可运行时，该配额被分配给片中的每 cpu 运行队列。一旦分配了所有配额，任何额外的配额请求都将导致这些线程被限制。被阻塞的线程将无法再次运行，直到下一个时段配额得到补充。

A group\'s unassigned quota is globally tracked, being refreshed back to cfs_quota units at each period boundary. As threads consume this bandwidth it is transferred to cpu-local \"silos\" on a demand basis. The amount transferred within each of these updates is tunable and described as the \"slice\".

> 全局跟踪组的未分配配额，并在每个时段边界刷新回 cfs_quota 单位。当线程消耗这个带宽时，它会按需传输到 cpu 本地`思洛存储器`。每个更新中传输的量是可调的，并被描述为`切片`。

# Burst feature

This feature borrows time now against our future underrun, at the cost of increased interference against the other system users. All nicely bounded.

> 这一功能现在借用了时间来应对我们未来的不足，代价是增加了对其他系统用户的干扰。所有的边界都很好。

Traditional (UP-EDF) bandwidth control is something like:

> (U = Sum u_i) \<= 1

This guaranteeds both that every deadline is met and that the system is stable. After all, if U were > 1, then for every second of walltime, we\'d have to run more than a second of program time, and obviously miss our deadline, but the next deadline will be further out still, there is never time to catch up, unbounded fail.

> 这既保证了每一个截止日期的到来，也保证了系统的稳定。毕竟，如果 U>1，那么每一秒的 walltime，我们就必须运行一秒以上的程序时间，显然会错过最后期限，但下一个最后期限会更晚，永远没有时间赶上，无限失败。

The burst feature observes that a workload doesn\'t always executes the full quota; this enables one to describe u_i as a statistical distribution.

> 突发特性观察到工作负载并不总是执行全部配额；这使得人们能够将 ui 描述为统计分布。

For example, have `u_i = {x,e}_i`, where x is the p(95) and x+e p(100) (the traditional WCET). This effectively allows u to be smaller, increasing the efficiency (we can pack more tasks in the system), but at the cost of missing deadlines when all the odds line up. However, it does maintain stability, since every overrun must be paired with an underrun as long as our x is above the average.

> 例如，具有`u_i={x,e}_i`，其中 x 是 p(95)和 x+e p(100)(传统 WCET)。这有效地允许 u 变小，提高了效率(我们可以在系统中打包更多的任务)，但代价是在所有几率一致的情况下错过最后期限。然而，它确实保持了稳定性，因为只要我们的 x 高于平均值，每次超限都必须与欠载配对。

That is, suppose we have 2 tasks, both specify a `p(95)` value, then we have a `p(95)*p(95) = 90.25%` chance both tasks are within their quota and everything is good. At the same time we have a `p(5)p(5) = 0.25%` chance both tasks will exceed their quota at the same time (guaranteed deadline fail). Somewhere in between there\'s a threshold where one exceeds and the other doesn\'t underrun enough to compensate; this depends on the specific `CDFs`.

> 也就是说，假设我们有两个任务，都指定了一个`p(95)`值，那么我们有一个`p(95)*p(95%)=90.25%`的机会，两个任务都在它们的配额内，并且一切都很好。同时，我们有`p(5)p(5)=0.25%`的机会，两项任务同时超过其配额(保证的截止日期失败)。在两者之间的某个地方，有一个阈值，一个超过，另一个不足，足以弥补；这取决于特定的`CDF`。

At the same time, we can say that the worst case deadline miss, will be Sum `e_i`; that is, there is a bounded tardiness (under the assumption that x+e is indeed `WCET`).

> 同时，我们可以说，错过最后期限的最坏情况是总和`e_i`；也就是说，存在有界延迟(在 x+e 确实是`WCET`的假设下)。

The interferenece when using burst is valued by the possibilities for missing the deadline and the average `WCET`. Test results showed that when there many `cgroups` or CPU is under utilized, the interference is limited. More details are shown in: <https://lore.kernel.org/lkml/5371BD36-55AE-4F71-B9D7-B86DC32E3D2B@linux.alibaba.com/>

> 当使用突发时的干扰通过错过最后期限的可能性和平均`WCET`来评估。测试结果表明，当有许多`cgroup`或 CPU 使用不足时，干扰是有限的。更多详细信息显示在：<https://lore.kernel.org/lkml/5371BD36-55AE-4F71-B9D7-B86DC32E3D2B@linux.aliba.com/>

# Management

Quota, period and burst are managed within the cpu subsystem via `cgroupfs`.

> 配额、周期和突发通过`cgroupfs`在 cpu 子系统中进行管理。

::: note
::: title
Note
:::

The cgroupfs files described in this section are only applicable to cgroup v1. For cgroup v2, see `Documentation/admin-guide/cgroup-v2.rst <cgroup-v2-cpu>`{.interpreted-text role="ref"}.

> 本节中描述的 cgroupfs 文件仅适用于 cgroup v1。有关 cgroup v2，请参阅`Documentation/admin guide/cgroup-v2.cpu>`{.depreted text role=`ref`}。
> :::

- cpu.cfs_quota_us: run-time replenished within a period (in microseconds)
- cpu.cfs_period_us: the length of a period (in microseconds)
- cpu.stat: exports throttling statistics \[explained further below\]
- cpu.cfs_burst_us: the maximum accumulated run-time (in microseconds)

The default values are:

```
cpu.cfs_period_us=100ms
cpu.cfs_quota_us=-1
cpu.cfs_burst_us=0
```

A value of `-1` for `cpu.cfs_quota_us` indicates that the group does not have any bandwidth restriction in place, such a group is described as an unconstrained bandwidth group. This represents the traditional work-conserving behavior for `CFS`.

> `cpu.cfs_quota_us`的值`-1`表示该组没有任何带宽限制，这样的组被描述为不受约束的带宽组。这代表了`CFS`的传统工作保存行为。

Writing any (valid) positive value(s) no smaller than `cpu.cfs_burst_us` will enact the specified bandwidth limit. The minimum quota allowed for the quota or period is 1ms. There is also an upper bound on the period length of 1s. Additional restrictions exist when bandwidth limits are used in a hierarchical fashion, these are explained in more detail below.

> 写入任何不小于`cpu.cfs_burst_us`的(有效)正值将制定指定的带宽限制。该配额或时段允许的最小配额为 1ms。1 的周期长度也有一个上限。当以分级方式使用带宽限制时，会存在额外的限制，这些限制将在下面进行更详细的解释。

Writing any negative value to `cpu.cfs_quota_us` will remove the bandwidth limit and return the group to an unconstrained state once more.

> 将任何负值写入`cpu.cfs_quota_us`将删除带宽限制，并使组再次返回到不受约束的状态。

A value of 0 for `cpu.cfs_burst_us` indicates that the group can not accumulate any unused bandwidth. It makes the traditional bandwidth control behavior for CFS unchanged. Writing any (valid) positive value(s) no larger than cpu.cfs_quota_us into `cpu.cfs_burst_us` will enact the cap on unused bandwidth accumulation.

> `cpu.cfs_burst_us`的值为 0 表示该组不能累积任何未使用的带宽。它使得传统的 CFS 带宽控制行为没有改变。将任何不大于 cpu.cfs_quota_s 的(有效)正值写入`cpu.cfs_burst_us`将对未使用的带宽累积设置上限。

dockAny updates to a group\'s bandwidth specification will result in it becoming unthrottled if it is in a constrained state.

> Dock 对组的带宽规范的任何更新都会导致它在处于约束状态时变得不受约束。

# System wide settings

For efficiency run-time is transferred between the global pool and CPU local \"silos\" in a batch fashion. This greatly reduces global accounting pressure on large systems. The amount transferred each time such an update is required is described as the \"slice\".

> 为了提高效率，运行时间以批处理方式在全局池和 CPU 本地`思洛存储器`之间传输。这大大降低了大型系统的全球会计压力。每次需要更新时转移的金额被描述为`切片`。

This is tunable via procfs:

```
/proc/sys/kernel/sched_cfs_bandwidth_slice_us (default=5ms)
```

Larger slice values will reduce transfer overheads, while smaller values allow for more fine-grained consumption.

> 较大的片值将减少传输开销，而较小的值允许更细粒度的消耗。

# Statistics

A group\'s bandwidth statistics are exported via 5 fields in cpu.stat.

cpu.stat:

- `nr_periods`: Number of enforcement intervals that have elapsed.
- `nr_throttled`: Number of times the group has been throttled/limited.
- `throttled_time`: The total time duration (in nanoseconds) for which entities of the group have been throttled.
- `nr_bursts`: Number of periods burst occurs.
- `burst_time`: Cumulative wall-time (in nanoseconds) that any CPUs has used above quota in respective periods.

> - `throttled_time`：组的实体被抑制的总持续时间(以纳秒为单位)。
> - `burst_time`：任何 CPU 在各个时段内使用的超过配额的累计壁时间(以纳秒为单位)。

This interface is read-only.

# Hierarchical considerations

The interface enforces that an individual entity\'s bandwidth is always attainable, that is: max(c_i) \<= C. However, over-subscription in the aggregate case is explicitly allowed to enable work-conserving semantics within a hierarchy:

> 接口强制单个实体的带宽始终是可实现的，即：max(c_i)\<=c。然而，明确允许在聚合情况下过度订阅，以在层次结构中启用工作保留语义：

> e.g. Sum (c_i) may exceed C

\[ Where C is the parent\'s bandwidth, and c_i its children \]

There are two ways in which a group may become throttled:

> a. it fully consumes its own quota within a period
> b. a parent\'s quota is fully consumed within its period

In case b) above, even though the child may have runtime remaining it will not be allowed to until the parent\'s runtime is refreshed.

> 在上面的情况 b)中，即使子级可能还有运行时，在刷新父级的运行时之前，也不允许这样做。

# CFS Bandwidth Quota Caveats

Once a slice is assigned to a cpu it does not expire. However all but 1ms of the slice may be returned to the global pool if all threads on that cpu become unrunnable. This is configured at compile time by the min_cfs_rq_runtime variable. This is a performance tweak that helps prevent added contention on the global lock.

> 一旦一个片被分配给一个 cpu，它就不会过期。但是，如果该 cpu 上的所有线程都无法运行，则除 1ms 外的所有切片都可能返回到全局池。这是在编译时由 min_cfs_rq_runtime 变量配置的。这是一个性能调整，有助于防止在全局锁上增加争用。

The fact that cpu-local slices do not expire results in some interesting corner cases that should be understood.

> cpu 本地片不会过期这一事实导致了一些有趣的角落情况，应该理解这些情况。

For cgroup cpu constrained applications that are cpu limited this is a relatively moot point because they will naturally consume the entirety of their quota as well as the entirety of each cpu-local slice in each period. As a result it is expected that nr_periods roughly equal nr_throttled, and that cpuacct.usage will increase roughly equal to cfs_quota_us in each period.

> 对于受 cpu 限制的 cgroup cpu 约束的应用程序来说，这是一个相对无意义的问题，因为它们自然会在每个时段消耗其全部配额以及每个 cpu 本地片的全部配额。因此，预计 nr_periods 大致等于 nr_throttled，并且 cpuacct.usage 将在每个周期中增加大致等于 cfs_quota_s。

For highly-threaded, non-cpu bound applications this non-expiration nuance allows applications to briefly burst past their quota limits by the amount of unused slice on each cpu that the task group is running on (typically at most 1ms per cpu or as defined by min_cfs_rq_runtime). This slight burst only applies if quota had been assigned to a cpu and then not fully used or returned in previous periods. This burst amount will not be transferred between cores. As a result, this mechanism still strictly limits the task group to quota average usage, albeit over a longer time window than a single period. This also limits the burst ability to no more than 1ms per cpu. This provides better more predictable user experience for highly threaded applications with small quota limits on high core count machines. It also eliminates the propensity to throttle these applications while simultaneously using less than quota amounts of cpu. Another way to say this, is that by allowing the unused portion of a slice to remain valid across periods we have decreased the possibility of wastefully expiring quota on cpu-local silos that don\'t need a full slice\'s amount of cpu time.

> 对于高度线程化、非 cpu 绑定的应用程序，这种不过期的细微差别允许应用程序通过任务组运行的每个 cpu 上未使用的片的数量(通常每个 cpu 最多 1ms 或 min_cfs_rq_runtime 定义)短暂突破其配额限制。这种轻微的突发仅适用于配额已分配给 cpu，但在以前的时段中未完全使用或返回的情况。此突发数量将不会在核心之间传输。因此，这种机制仍然严格地将任务组限制为配额平均使用量，尽管时间窗口比单个时段更长。这也将突发能力限制在每 cpu 不超过 1ms。这为在高核数机器上具有小配额限制的高线程应用程序提供了更好、更可预测的用户体验。它还消除了在使用少于配额数量的 cpu 的同时限制这些应用程序的倾向。另一种说法是，通过允许片的未使用部分在不同时间段内保持有效，我们降低了不需要完整片的 cpu 时间的 cpu 本地竖井上配额浪费到期的可能性。

The interaction between cpu-bound and non-cpu-bound-interactive applications should also be considered, especially when single core usage hits 100%. If you gave each of these applications half of a cpu-core and they both got scheduled on the same CPU it is theoretically possible that the non-cpu bound application will use up to 1ms additional quota in some periods, thereby preventing the cpu-bound application from fully using its quota by that same amount. In these instances it will be up to the CFS algorithm (see sched-design-CFS.rst) to decide which application is chosen to run, as they will both be runnable and have remaining quota. This runtime discrepancy will be made up in the following periods when the interactive application idles.

> 还应考虑 cpu 绑定和非 cpu 绑定交互应用程序之间的交互，尤其是当单核使用率达到 100%时。如果你给这些应用程序中的每一个提供一半的 cpu 核心，并且它们都被安排在同一个 cpu 上，那么理论上，非 cpu 绑定的应用程序可能会在某些时间段内使用高达 1ms 的额外配额，从而阻止 cpu 绑定的应用程序以相同的数量完全使用其配额。在这些情况下，将由 CFS 算法(参见计划设计 CFS.rst)来决定选择运行哪个应用程序，因为它们都是可运行的，并且有剩余的配额。当交互式应用程序空闲时，这种运行时差异将在以下时间段内得到弥补。

# Examples

1.  Limit a group to 1 CPU worth of runtime:

        If period is 250ms and quota is also 250ms, the group will get 1 CPU worth of runtime every 250ms.

        # echo 250000 > cpu.cfs_quota_us /* quota = 250ms */
        # echo 250000 > cpu.cfs_period_us /* period = 250ms */

2.  Limit a group to 2 CPUs worth of runtime on a multi-CPU machine

    With 500ms period and 1000ms quota, the group can get 2 CPUs worth of runtime every 500ms:

        # echo 1000000 > cpu.cfs_quota_us /* quota = 1000ms */
        # echo 500000 > cpu.cfs_period_us /* period = 500ms */

        The larger period here allows for increased burst capacity.

3.  Limit a group to 20% of 1 CPU.

    With 50ms period, 10ms quota will be equivalent to 20% of 1 CPU:

        # echo 10000 > cpu.cfs_quota_us /* quota = 10ms */
        # echo 50000 > cpu.cfs_period_us /* period = 50ms */

    By using a small period here we are ensuring a consistent latency response at the expense of burst capacity.

4.  Limit a group to 40% of 1 CPU, and allow accumulate up to 20% of 1 CPU additionally, in case accumulation has been done.

> 4.将一组限制为 1 个 CPU 的 40%，并允许额外累积 1 个 CPU，以防累积完成。

    With 50ms period, 20ms quota will be equivalent to 40% of 1 CPU. And 10ms burst will be equivalent to 20% of 1 CPU:

        # echo 20000 > cpu.cfs_quota_us /* quota = 20ms */
        # echo 50000 > cpu.cfs_period_us /* period = 50ms */
        # echo 10000 > cpu.cfs_burst_us /* burst = 10ms */

    Larger buffer setting (no larger than quota) allows greater burst capacity.
