---
tip: translate by baidu@2024-01-17 19:38:42
title: Utilization Clamping
---

# 1. Introduction

`Utilization clamping`, also known as `util clamp` or `uclamp`, is a scheduler feature that allows user space to help in managing the performance requirement of tasks. It was introduced in v5.3 release. The CGroup support was merged in v5.4.

> `Utilization clamping`，也称为 `util clamp` 或 `uclamp`，是一种调度器功能，允许用户空间帮助管理任务的性能要求。它是在 v5.3 版本中引入的。CGroup 支持在 v5.4 中合并。

`uclamp` is a hinting mechanism that allows the scheduler to understand the performance requirements and restrictions of the tasks, thus it helps the scheduler to make a better decision. And when `schedutil cpufreq` governor is used, `util clamp` will influence the CPU frequency selection as well.

> `uclamp` 是一种提示机制，允许调度器了解任务的性能要求和限制，从而帮助调度器做出更好的决策。当使用 `schedutil cpufreq` 调速器时，`util clamp` 位也会影响 CPU 的频率选择。

Since the scheduler and schedutil are both driven by `PELT` (util_avg) signals, `util clamp` acts on that to achieve its goal by clamping the signal to a certain point; hence the name. That is, by clamping utilization we are making the system run at a certain performance point.

> 由于 scheduler 和 schedutil 都是由 `PELT` (util_avg)信号驱动的，所以 `util clamp` 通过将信号箝位到某一点来实现其目标；因此得名。也就是说，通过钳制利用率，我们使系统在某个性能点上运行。

The right way to view `util clamp` is as a mechanism to make request or hint on performance constraints. It consists of two tunables:

> 查看 `util clamp` 的正确方法是将其作为一种机制来请求或提示性能约束。它由两个可调项组成：

> - `UCLAMP_MIN`, which sets the lower bound.
> - `UCLAMP_MAX`, which sets the upper bound.

These two bounds will ensure a task will operate within this performance range of the system. `UCLAMP_MIN` implies boosting a task, while `UCLAMP_MAX` implies capping a task.

> 这两个界限将确保任务将在系统的这个性能范围内运行 `UCLAMP_MIN` 表示提升任务，而 `UCLAMP_MAX` 表示限制任务。

One can tell the system (scheduler) that some tasks require a minimum performance point to operate at to deliver the desired user experience. Or one can tell the system that some tasks should be restricted from consuming too much resources and should not go above a specific performance point. Viewing the `uclamp` values as performance points rather than utilization is a better abstraction from user space point of view.

> 可以告诉系统(scheduler)，一些任务需要最低的性能点来进行操作，以提供所需的用户体验。或者可以告诉系统，应该限制某些任务消耗太多资源，并且不应超过特定的性能点。从用户空间的角度来看，将 `uclamp` 值视为性能点而非利用率是更好的抽象。

As an example, a game can use `util clamp` to form a feedback loop with its perceived Frames Per Second (FPS). It can dynamically increase the minimum performance point required by its display pipeline to ensure no frame is dropped. It can also dynamically `prime` up these tasks if it knows in the coming few hundred milliseconds a computationally intensive scene is about to happen.

> 例如，游戏可以使用 `util clamp` 与其感知的每秒帧数(FPS)形成反馈循环。它可以动态地增加其显示管道所需的最小性能点，以确保没有帧丢失。如果它知道在未来几百毫秒内将发生计算密集型场景，它也可以动态地“启动”这些任务。

On mobile hardware where the capability of the devices varies a lot, this dynamic feedback loop offers a great flexibility to ensure best user experience given the capabilities of any system.

> 在设备能力变化很大的移动硬件上，这种动态反馈回路提供了很大的灵活性，以确保在任何系统的能力下都能获得最佳的用户体验。

Of course a static configuration is possible too. The exact usage will depend on the system, application and the desired outcome.

> 当然，静态配置也是可能的。确切的用途将取决于系统、应用程序和期望的结果。

Another example is in Android where tasks are classified as background, foreground, top-app, etc. `util clamp` can be used to constrain how much resources background tasks are consuming by capping the performance point they can run at. This constraint helps reserve resources for important tasks, like the ones belonging to the currently active app (top-app group). Beside this helps in limiting how much power they consume. This can be more obvious in heterogeneous systems (e.g. Arm big.LITTLE); the constraint will help bias the background tasks to stay on the little cores which will ensure that:

> 另一个例子是在安卓系统中，任务分为后台、前台、顶部应用程序等。`util clamp` 可以用于通过限制后台任务的运行性能来限制后台任务消耗的资源量。这种限制有助于为重要任务保留资源，如属于当前活动应用程序(顶部应用程序组)的任务。除此之外，这有助于限制它们的功耗。这在异构系统中更为明显(例如 Arm big.LITTLE)；该约束将有助于使背景任务偏向于停留在小核心上，这将确保：

1. The big cores are free to run top-app tasks immediately. top-app tasks are the tasks the user is currently interacting with, hence the most important tasks in the system.

> 1.大核心可以立即免费运行顶级应用程序任务。顶级应用程序任务是用户当前正在交互的任务，因此是系统中最重要的任务。

2. They don\'t run on a power hungry core and drain battery even if they are CPU intensive tasks.

> 2.即使是 CPU 密集型任务，它们也不会在耗电的核心上运行，也不会耗尽电池。

::: note
::: title
Note
:::

**little cores**: CPUs with capacity \< 1024

**big cores**:

: CPUs with capacity = 1024
:::

By making these uclamp performance requests, or rather hints, user space can ensure system resources are used optimally to deliver the best possible user experience.

> 通过提出这些 uclamp 性能请求，或者更确切地说是提示，用户空间可以确保系统资源得到最佳使用，以提供尽可能好的用户体验。

Another use case is to help with **overcoming the ramp up latency inherit in how scheduler utilization signal is calculated**.

> 另一个用例是帮助**克服如何计算调度器利用率信号**中继承的斜坡上升延迟。

On the other hand, a busy task for instance that requires to run at maximum performance point will suffer a delay of `~200ms` (PELT HALFIFE = 32ms) for the scheduler to realize that. This is known to affect workloads like gaming on mobile devices where frames will drop due to slow response time to select the higher frequency required for the tasks to finish their work in time. Setting `UCLAMP_MIN=1024` will ensure such tasks will always see the highest performance level when they start running.

> 另一方面，例如，需要以最大性能点运行的繁忙任务将遭受“~200ms”(PELT HALFIFE=32ms)的延迟，以便调度器实现这一点。众所周知，这会影响移动设备上的游戏等工作负载，因为选择任务及时完成工作所需的更高频率的响应时间较慢，帧会下降。设置“UCLAMP_MIN=1024”将确保此类任务在开始运行时始终达到最高性能级别。

The overall visible effect goes beyond better perceived user experience/performance and stretches to help achieve a better overall performance/watt if used effectively.

> 整体可见效果超越了更好的感知用户体验/性能，如果有效使用，可以帮助实现更好的整体性能/瓦数。

User space can form a feedback loop with the thermal subsystem too to ensure the device doesn\'t heat up to the point where it will throttle.

> 用户空间也可以与热力子系统形成反馈回路，以确保设备不会加热到节流的程度。

Both `SCHED_NORMAL/OTHER` and `SCHED_FIFO/RR` honour uclamp requests/hints.

> “SCHED_NORM/OTHER”和“SCHED_FIFO/RR”都遵守 uclamp 请求/提示。

In the `SCHED_FIFO/RR` case, uclamp gives the option to run RT tasks at any performance point rather than being tied to MAX frequency all the time. Which can be useful on general purpose systems that run on battery powered devices.

> 在“SCHED_FIFO/RR”的情况下，uclamp 提供了在任何性能点运行 RT 任务的选项，而不是一直与 MAX 频率绑定。这对于在电池供电设备上运行的通用系统非常有用。

Note that by design RT tasks don\'t have per-task PELT signal and must always run at a constant frequency to combat undeterministic DVFS rampup delays.

> 注意，根据设计，RT 任务没有每个任务的 PELT 信号，必须始终以恒定频率运行，以对抗无法确定的 DVFS 上升延迟。

Note that using schedutil always implies a single delay to modify the frequency when an RT task wakes up. This cost is unchanged by using uclamp. Uclamp only helps picking what frequency to request instead of schedutil always requesting MAX for all RT tasks.

> 请注意，当 RT 任务唤醒时，使用 schedutil 总是意味着修改频率的单个延迟。使用 uclamp 后，此成本保持不变。Uclamp 只帮助选择请求的频率，而不是 schedutil 总是为所有 RT 任务请求 MAX。

See `section 3.4 <uclamp-default-values>`{.interpreted-text role="ref"} for default values and `3.4.1 <sched-util-clamp-min-rt-default>`{.interpreted-text role="ref"} on how to change RT tasks default value.

> 关于如何更改 rt 任务的默认值，请参见“第 3.4 节＜ uclamp 默认值＞”｛.explored text role=“ref”｝了解默认值，以及“3.4.1<sched `util clamp` min rt default>`｛.expered text rol=“ref”}了解。

# 2. Design

`util clamp` is a property of every task in the system. It sets the boundaries of its utilization signal; acting as a bias mechanism that influences certain decisions within the scheduler.

> `util clamp` 是系统中每个任务的属性。它设置了其利用信号的边界；充当影响调度器内的某些决策的偏置机制。

The actual utilization signal of a task is never clamped in reality. If you inspect PELT signals at any point of time you should continue to see them as they are intact. Clamping happens only when needed, e.g: when a task wakes up and the scheduler needs to select a suitable CPU for it to run on.

> 任务的实际利用率信号在现实中从未被钳制。如果您在任何时间点检查 PELT 信号，您应该继续看到它们完好无损。箝位仅在需要时发生，例如：当任务唤醒并且调度器需要选择合适的 CPU 来运行时。

Since the goal of `util clamp` is to allow requesting a minimum and maximum performance point for a task to run on, it must be able to influence the frequency selection as well as task placement to be most effective. Both of which have implications on the utilization value at CPU runqueue (rq for short) level, which brings us to the main design challenge.

> 由于 `util clamp` 的目标是允许请求任务运行的最小和最大性能点，因此它必须能够影响频率选择和任务放置，才能最有效。这两者都对 CPU 运行队列(简称 rq)级别的利用率值有影响，这给我们带来了主要的设计挑战。

When a task wakes up on an rq, the utilization signal of the `rq` will be affected by the uclamp settings of all the tasks enqueued on it. For example if a task requests to run at `UTIL_MIN = 512`, then the util signal of the `rq` needs to respect to this request as well as all other requests from all of the enqueued tasks.

> 当一个任务在 rq 上唤醒时，“rq”的利用率信号将受到其上排队的所有任务的 uclamp 设置的影响。例如，如果一个任务请求以“UTIL_MIN=512”运行，那么“rq’的 UTIL 信号需要与该请求以及来自所有排队任务的所有其他请求相关。

To be able to aggregate the `util clamp` value of all the tasks attached to the rq, uclamp must do some housekeeping at every enqueue/dequeue, which is the scheduler hot path. Hence care must be taken since any slow down will have significant impact on a lot of use cases and could hinder its usability in practice.

> 为了能够聚合所有附加到 rq 的任务的 `util clamp` 值，uclamp 必须在每个入队/出队时进行一些内务处理，这是调度器的热路径。因此，必须小心，因为任何放缓都会对许多用例产生重大影响，并可能阻碍其在实践中的可用性。

The way this is handled is by dividing the utilization range into buckets (struct uclamp_bucket) which allows us to reduce the search space from every task on the `rq` to only a subset of tasks on the top-most bucket.

> 处理这一问题的方法是将利用率范围划分为多个桶(struct uclamp_bucket)，这使我们能够将搜索空间从“rq”上的每个任务减少到最顶部桶上的一个子集。

When a task is enqueued, the counter in the matching bucket is incremented, and on dequeue it is decremented. This makes keeping track of the effective uclamp value at `rq` level a lot easier.

> 当任务入队时，匹配存储桶中的计数器会递增，出队时则递减。这使得跟踪“rq”级别的有效 uclamp 值变得容易得多。

As tasks are enqueued and dequeued, we keep track of the current effective uclamp value of the rq. See `section 2.1 <uclamp-buckets>`{.interpreted-text role="ref"} for details on how this works.

> 当任务入队和出队时，我们跟踪 rq 的当前有效 uclamp 值。有关其工作原理的详细信息，请参阅`section 2.1<uclamp buckets>`{.depreted text role=“ref”}。

Later at any path that wants to identify the effective uclamp value of the rq, it will simply need to read this effective uclamp value of the `rq` at that exact moment of time it needs to take a decision.

> 稍后，在任何想要识别 rq 的有效 uclamp 值的路径上，它只需要在需要做出决定的确切时刻读取“rq”的有效 uclamp 值。

For task placement case, only Energy Aware and Capacity Aware Scheduling (`EAS/CAS`) make use of uclamp for now, which implies that it is applied on heterogeneous systems only. When a task wakes up, the scheduler will look at the current effective uclamp value of every `rq` and compare it with the potential new value if the task were to be enqueued there. Favoring the `rq` that will end up with the most energy efficient combination.

> 对于任务布局情况，目前只有能量感知和容量感知调度(“EAS/CAS”)使用 uclamp，这意味着它仅应用于异构系统。当任务唤醒时，调度器将查看每个“rq”的当前有效 uclamp 值，并将其与任务在那里排队时的潜在新值进行比较。支持“rq”，它最终将成为最节能的组合。

Similarly in schedutil, when it needs to make a frequency update it will look at the current effective uclamp value of the `rq` which is influenced by the set of tasks currently enqueued there and select the appropriate frequency that will satisfy constraints from requests.

> 类似地，在 schedutil 中，当它需要进行频率更新时，它将查看“rq”的当前有效 uclamp 值，该值受当前排队在那里的任务集的影响，并选择满足请求约束的适当频率。

Other paths like setting overutilization state (which effectively disables EAS) make use of uclamp as well. Such cases are considered necessary housekeeping to allow the 2 main use cases above and will not be covered in detail here as they could change with implementation details.

> 其他路径，如设置过度使用状态(有效地禁用 EAS)也使用 uclamp。这样的案例被认为是必要的内务管理，以允许上面的两个主要用例，这里不会详细介绍，因为它们可能会随着实现细节的变化而变化。

## 2.1. Buckets {#uclamp-buckets}

```
    [struct rq]

    (bottom)                                                    (top)

    0                                                          1024
    |                                                           |
    +-----------+-----------+-----------+----   ----+-----------+
    |  Bucket 0 |  Bucket 1 |  Bucket 2 |    ...    |  Bucket N |
    +-----------+-----------+-----------+----   ----+-----------+
    :           :                                   :
    +- p0       +- p3                               +- p4
    :                                               :
    +- p1                                           +- p5
    :
    +- p2
```

::: note
::: title
Note
:::

The diagram above is an illustration rather than a true depiction of the internal data structure.

> 上图是对内部数据结构的说明，而非真实描述。
> :::

To reduce the search space when trying to decide the effective uclamp value of an `rq` as tasks are enqueued/dequeued, the whole utilization range is divided into N buckets where N is configured at compile time by setting CONFIG_UCLAMP_BUCKETS_COUNT. By default it is set to 5.

> 当任务入队/出队时，为了在尝试决定“rq”的有效 uclamp 值时减少搜索空间，将整个利用率范围划分为 N 个桶，其中 N 在编译时通过设置 CONFIG_uclamp_buckets_COUNT 进行配置。默认情况下，它设置为 5。

The `rq` has a bucket for each uclamp_id tunables: [`UCLAMP_MIN`, `UCLAMP_MAX`].

> “rq”为每个 uclamp_id 可调项都有一个 bucket：[`uclamp_MIN`，`uclamp_MAX`]。

The range of each bucket is 1024/N. For example, for the default value of 5 there will be 5 buckets, each of which will cover the following range:

> 每个铲斗的范围为 1024/N。例如，对于默认值 5，将有 5 个 bucket，每个 bucket 将覆盖以下范围：

```
    DELTA = round_closest(1024/5) = 204.8 = 205

    Bucket 0: [0:204]
    Bucket 1: [205:409]
    Bucket 2: [410:614]
    Bucket 3: [615:819]
    Bucket 4: [820:1024]
```

When a task p with following tunable parameters

```
    p->uclamp[UCLAMP_MIN] = 300
    p->uclamp[UCLAMP_MAX] = 1024
```

is enqueued into the `rq`, bucket 1 will be incremented for `UCLAMP_MIN` and bucket 4 will be incremented for `UCLAMP_MAX` to reflect the fact the `rq` has a task in this range.

> 则桶 1 将为“UCLAMP_MIN”递增，而桶 4 将为“UCLAMP_MAX”递增，以反映“rq”具有在此范围内的任务的事实。

The `rq` then keeps track of its current effective uclamp value for each `uclamp_id`.

> 然后，“rq”跟踪每个“uclamp_id”的当前有效 uclamp 值。

When a task `p` is enqueued, the `rq` value changes to:

```
    // update bucket logic goes here
    rq->uclamp[UCLAMP_MIN] = max(rq->uclamp[UCLAMP_MIN], p->uclamp[UCLAMP_MIN])
    // repeat for UCLAMP_MAX
```

Similarly, when `p` is dequeued the `rq` value changes to:

```
    // update bucket logic goes here
    rq->uclamp[UCLAMP_MIN] = search_top_bucket_for_highest_value()
    // repeat for UCLAMP_MAX
```

When all buckets are empty, the `rq` uclamp values are reset to system defaults. See `section 3.4 <uclamp-default-values>`{.interpreted-text role="ref"} for details on default values.

> 当所有存储桶都为空时，“rq”uclamp 值将重置为系统默认值。有关默认值的详细信息，请参阅“第 3.4 节＜ uclamp 默认值＞”｛.explored text role=“ref”｝。

## 2.2. Max aggregation

`util clamp` is tuned to honour the request for the task that requires the highest performance point.

> `util clamp` 经过调整以满足对需要最高性能点的任务的请求。

When multiple tasks are attached to the same `rq`, then `util clamp` must make sure the task that needs the highest performance point gets it even if there\'s another task that doesn\'t need it or is disallowed from reaching this point.

> 当多个任务附加到同一个“rq”时，`util clamp` 必须确保需要最高性能点的任务得到它，即使有另一个任务不需要它或被禁止达到这一点。

For example, if there are multiple tasks attached to an `rq` with the following values:

> 例如，如果有多个任务附加到具有以下值的“rq”：

```
    p0->uclamp[UCLAMP_MIN] = 300
    p0->uclamp[UCLAMP_MAX] = 900

    p1->uclamp[UCLAMP_MIN] = 500
    p1->uclamp[UCLAMP_MAX] = 500
```

then assuming both `p0` and `p1` are enqueued to the same `rq`, both `UCLAMP_MIN` and `UCLAMP_MAX` become:

> 则假设“p0”和“p1”都被排队到相同的“rq”，则“UCLAMP_MIN”和“UCLAMP_MAX”都变为：

```
    rq->uclamp[UCLAMP_MIN] = max(300, 500) = 500
    rq->uclamp[UCLAMP_MAX] = max(900, 500) = 900
```

As we shall see in `section 5.1 <uclamp-capping-fail>`{.interpreted-text role="ref"}, this max aggregation is the cause of one of limitations when using `util clamp`, in particular for `UCLAMP_MAX` hint when user space would like to save power.

> 正如我们将在“section 5.1 ＜ uclamp capping fail ＞”｛.explored text role=“ref”｝中看到的那样，这种最大聚合是使用 `util clamp` 时的一个限制的原因，特别是对于用户空间希望节省电力时的“uclamp_max”提示。

## 2.3. Hierarchical aggregation

As stated earlier, `util clamp` is a property of every task in the system. But the actual applied (effective) value can be influenced by more than just the request made by the task or another actor on its behalf (middleware library).

> 如前所述，`util clamp` 是系统中每个任务的属性。但是，实际应用的(有效的)值可能受到的影响不仅仅是任务或代表它的另一个参与者(中间件库)发出的请求。

The effective `util clamp` value of any task is restricted as follows:

1. By the uclamp settings defined by the cgroup CPU controller it is attached to, if any.

> 1.通过连接到的 cgroup CPU 控制器定义的 uclamp 设置(如果有的话)。

2. The restricted value in (1) is then further restricted by the system wide uclamp settings.

> 2.然后，(1)中的限制值进一步受到系统范围 uclamp 设置的限制。

`Section 3 <uclamp-interfaces>`{.interpreted-text role="ref"} discusses the interfaces and will expand further on that.

For now suffice to say that if a task makes a request, its actual effective value will have to adhere to some restrictions imposed by cgroup and system wide settings.

> 现在可以说，如果任务提出请求，其实际有效值必须遵守 cgroup 和系统范围设置施加的一些限制。

The system will still accept the request even if effectively will be beyond the constraints, but as soon as the task moves to a different cgroup or a sysadmin modifies the system settings, the request will be satisfied only if it is within new constraints.

> 即使有效地超出了限制，系统仍然会接受请求，但一旦任务移动到另一个 cgroup 或系统管理员修改了系统设置，只有在新的限制范围内，请求才会得到满足。

In other words, this aggregation will not cause an error when a task changes its uclamp values, but rather the system may not be able to satisfy requests based on those factors.

> 换句话说，当任务改变其 uclamp 值时，这种聚合不会导致错误，而是系统可能无法满足基于这些因素的请求。

## 2.4. Range

Uclamp performance request has the range of 0 to 1024 inclusive.

For cgroup interface percentage is used (that is 0 to 100 inclusive). Just like other cgroup interfaces, you can use `max` instead of 100.

> 对于 cgroup，使用百分比(包括 0 到 100)。就像其他 cgroup 接口一样，您可以使用“max”而不是 100。

# 3. Interfaces {#uclamp-interfaces}

## 3.1. Per task interface

sched_setattr() syscall was extended to accept two new fields:

- `sched_util_min`: requests the minimum performance point the system should run at when this task is running. Or lower performance bound.

> -`sched_util_min`：请求运行此任务时系统应运行的最低性能点。或者性能下限。

- `sched_util_max`: requests the maximum performance point the system should run at when this task is running. Or upper performance bound.

> -`sched_util_max`：请求此任务运行时系统应运行的最大性能点。或性能上限。

For example, the following scenario have 40% to 80% utilization constraints:

> 例如，以下场景具有 40%到 80%的利用率限制：

```
    attr->sched_util_min = 40% * 1024;
    attr->sched_util_max = 80% * 1024;
```

When task \@p is running, **the scheduler should try its best to ensure it starts at 40% performance level**. If the task runs for a long enough time so that its actual utilization goes above 80%, the utilization, or performance level, will be capped.

> 当任务\\@p 正在运行时，**调度程序应尽最大努力确保它以 40%的性能级别**启动。如果任务运行足够长的时间，使其实际利用率超过 80%，则利用率或性能级别将受到限制。

The special value -1 is used to reset the uclamp settings to the system default.

> 特殊值-1 用于将 uclamp 设置重置为系统默认值。

Note that resetting the uclamp value to system default using -1 is not the same as manually setting uclamp value to system default. This distinction is important because as we shall see in system interfaces, the default value for RT could be changed. `SCHED_NORMAL/OTHER` might gain similar knobs too in the future.

> 请注意，使用-1 将 uclamp 值重置为系统默认值与手动将 uclamps 值设置为系统默认不同。这种区别很重要，因为正如我们将在系统接口中看到的那样，RT 的默认值可能会更改`SCHED_NORM/OTHER`在未来也可能获得类似的旋钮。

## 3.2. cgroup interface

There are two uclamp related values in the CPU cgroup controller:

- cpu.uclamp.min
- cpu.uclamp.max

When a task is attached to a CPU controller, its uclamp values will be impacted as follows:

> 当任务连接到 CPU 控制器时，其 uclamp 值将受到以下影响：

- cpu.uclamp.min is a protection as described in section 3-3 of cgroup

v2 documentation <cgroupv2-protections-distributor>`{.interpreted-text role="ref"}.

> v2 文档<cgroupv2 protections distributor>`｛.explored text role=“ref”｝。

If a task uclamp_min value is lower than `cpu.uclamp.min`, then the task will inherit the cgroup `cpu.uclamp.min` value.

> 如果任务 uclamp_min 的值低于“cpu.uclamp.min”，则该任务将继承 cgroup“cpu.uclamp.min'值。

In a cgroup hierarchy, effective `cpu.uclamp.min` is the max of (child, parent).

> 在 cgroup 层次结构中，有效的“cpu.uclamp.min”是(child，parent)的最大值。

- `cpu.uclamp.max` is a limit as described in `section 3-2 of cgroup v2

documentation <cgroupv2-limits-distributor>`{.interpreted-text role="ref"}.

> documentation ＜ cgroupv2 limits distributor ＞`{.depredicted text role=“ref”}。

If a task uclamp_max value is higher than `cpu.uclamp.max`, then the task will inherit the cgroup `cpu.uclamp.max` value.

> 如果任务 uclamp_max 值高于“cpu.uclamp.max”，则该任务将继承 cgroup“cpu.uclamp.max’值。

In a cgroup hierarchy, effective cpu.uclamp.max is the min of (child, parent).

> 在 cgroup 层次结构中，有效 cpu.uclamp.max 是(child，parent)的最小值。

For example, given following parameters:

```
    p0->uclamp[UCLAMP_MIN] = // system default;
    p0->uclamp[UCLAMP_MAX] = // system default;

    p1->uclamp[UCLAMP_MIN] = 40% * 1024;
    p1->uclamp[UCLAMP_MAX] = 50% * 1024;

    cgroup0->cpu.uclamp.min = 20% * 1024;
    cgroup0->cpu.uclamp.max = 60% * 1024;

    cgroup1->cpu.uclamp.min = 60% * 1024;
    cgroup1->cpu.uclamp.max = 100% * 1024;
```

when p0 and p1 are attached to cgroup0, the values become:

```
    p0->uclamp[UCLAMP_MIN] = cgroup0->cpu.uclamp.min = 20% * 1024;
    p0->uclamp[UCLAMP_MAX] = cgroup0->cpu.uclamp.max = 60% * 1024;

    p1->uclamp[UCLAMP_MIN] = 40% * 1024; // intact
    p1->uclamp[UCLAMP_MAX] = 50% * 1024; // intact
```

when p0 and p1 are attached to cgroup1, these instead become:

```
    p0->uclamp[UCLAMP_MIN] = cgroup1->cpu.uclamp.min = 60% * 1024;
    p0->uclamp[UCLAMP_MAX] = cgroup1->cpu.uclamp.max = 100% * 1024;

    p1->uclamp[UCLAMP_MIN] = cgroup1->cpu.uclamp.min = 60% * 1024;
    p1->uclamp[UCLAMP_MAX] = 50% * 1024; // intact
```

Note that cgroup interfaces allows `cpu.uclamp.max` value to be lower than `cpu.uclamp.min`. Other interfaces don\'t allow that.

> 请注意，cgroup 接口允许“cpu.uclamp.max”值低于“cpu.uclamp.min”。其他接口不允许这样做。

## 3.3. System interface

## 3.3.1 sched_util_clamp_min

System wide limit of allowed `UCLAMP_MIN` range. By default it is set to 1024, which means that permitted effective `UCLAMP_MIN` range for tasks is [0:1024]. By changing it to 512 for example the range reduces to [0:512]. This is useful to restrict how much boosting tasks are allowed to acquire.

> 允许的“UCLAMP_MIN”范围的系统范围限制。默认情况下，它设置为 1024，这意味着任务的允许有效“UCLAMP_MIN”范围为[0:1024]。例如，通过将其更改为 512，范围减小到[0:512]。这对于限制允许获取多少助推任务非常有用。

Requests from tasks to go above this knob value will still succeed, but they won\'t be satisfied until it is more than `p->uclamp[UCLAMP_MIN]`.

> 来自任务的超过该旋钮值的请求仍然会成功，但在超过“p->uclamp[uclamp_MIN]”之前不会得到满足。

The value must be smaller than or equal to `sched_util_clamp_max`.

## 3.3.2 `sched_util_clamp_max`

System wide limit of allowed `UCLAMP_MAX` range. By default it is set to 1024, which means that permitted effective `UCLAMP_MAX` range for tasks is [0:1024].

> 允许的“UCLAMP_MAX”范围的系统范围限制。默认情况下，它设置为 1024，这意味着任务的允许有效“UCLAMP_MAX”范围为[0:1024]。

By changing it to 512 for example the effective allowed range reduces to [0:512]. This means is that no task can run above 512, which implies that all rqs are restricted too. IOW, the whole system is capped to half its performance capacity.

> 例如，通过将其更改为 512，有效允许范围减小到[0:512]。这意味着没有任务可以在 512 以上运行，这意味着所有 rq 也受到限制。IOW，整个系统的性能容量上限为其性能容量的一半。

This is useful to restrict the overall maximum performance point of the system. For example, it can be handy to limit performance when running low on battery or when the system wants to limit access to more energy hungry performance levels when it\'s in idle state or screen is off.

> 这对于限制系统的总体最大性能点非常有用。例如，当电池电量不足时，或者当系统处于空闲状态或屏幕关闭时，想要限制对更耗电性能级别的访问时，限制性能是很方便的。

Requests from tasks to go above this knob value will still succeed, but they won\'t be satisfied until it is more than `p->uclamp[UCLAMP_MAX]`.

> 来自任务的超过该旋钮值的请求仍然会成功，但在超过“p->uclamp[uclamp_MAX]”之前不会得到满足。

The value must be greater than or equal to sched_util_clamp_min.

## 3.4. Default values {#uclamp-default-values}

By default all SCHED_NORMAL/SCHED_OTHER tasks are initialized to:

```
    p_fair->uclamp[UCLAMP_MIN] = 0
    p_fair->uclamp[UCLAMP_MAX] = 1024
```

That is, by default they\'re boosted to run at the maximum performance point of changed at boot or runtime. No argument was made yet as to why we should provide this, but can be added in the future.

> 也就是说，默认情况下，它们被提升到在启动或运行时更改的最大性能点运行。目前还没有关于我们为什么应该提供这一点的争论，但可以在未来添加。

For `SCHED_FIFO/SCHED_RR` tasks:

```
    p_rt->uclamp[UCLAMP_MIN] = 1024
    p_rt->uclamp[UCLAMP_MAX] = 1024
```

That is by default they\'re boosted to run at the maximum performance point of the system which retains the historical behavior of the RT tasks.

> 默认情况下，它们被提升到系统的最大性能点运行，从而保留 RT 任务的历史行为。

RT tasks default uclamp_min value can be modified at boot or runtime via sysctl. See below section.

> RT 任务的默认 uclamp_min 值可以在启动或运行时通过 sysctl 进行修改。请参阅以下部分。

## 3.4.1 sched_util_clamp_min_rt_default {#sched-util-clamp-min-rt-default}

Running RT tasks at maximum performance point is expensive on battery powered devices and not necessary. To allow system developer to offer good performance guarantees for these tasks without pushing it all the way to maximum performance point, this sysctl knob allows tuning the best boost value to address the system requirement without burning power running at maximum performance point all the time.

> 在电池供电的设备上，以最高性能点运行 RT 任务是昂贵的，而且没有必要。为了让系统开发人员能够为这些任务提供良好的性能保证，而不必将其一直推到最大性能点，这个 sysctl 旋钮允许调整最佳提升值以满足系统需求，而不会一直以最大性能点运行。

Application developer are encouraged to use the per task `util clamp` interface to ensure they are performance and power aware. Ideally this knob should be set to 0 by system designers and leave the task of managing performance requirements to the apps.

> 鼓励应用程序开发人员使用每任务 `util clamp` 接口，以确保他们具有性能和电源意识。理想情况下，系统设计者应将此旋钮设置为 0，并将管理性能要求的任务留给应用程序。

# 4. How to use `util clamp`

`util clamp` promotes the concept of user space assisted power and performance management. At the scheduler level there is no info required to make the best decision. However, with `util clamp` user space can hint to the scheduler to make better decision about task placement and frequency selection.

> `util clamp` 推广了用户空间辅助电源和性能管理的概念。在调度器级别，不需要任何信息来做出最佳决策。然而，使用 `util clamp`，用户空间可以提示调度器对任务放置和频率选择做出更好的决定。

Best results are achieved by not making any assumptions about the system the application is running on and to use it in conjunction with a feedback loop to dynamically monitor and adjust. Ultimately this will allow for a better user experience at a better perf/watt.

> 通过不对应用程序运行的系统进行任何假设，并将其与反馈回路结合使用以动态监控和调整，可以获得最佳结果。最终，这将以更好的性能/瓦提供更好的用户体验。

For some systems and use cases, static setup will help to achieve good results. Portability will be a problem in this case. How much work one can do at 100, 200 or 1024 is different for each system. Unless there\'s a specific target system, static setup should be avoided.

> 对于某些系统和用例，静态设置将有助于获得良好的结果。在这种情况下，便携性将是一个问题。一个人在 100、200 或 1024 下能做多少工作对于每个系统来说是不同的。除非有特定的目标系统，否则应避免静态设置。

There are enough possibilities to create a whole framework based on `util clamp` or self contained app that makes use of it directly.

> 有足够的可能性来创建一个基于 `util clamp` 或直接使用它的自包含应用程序的完整框架。

## 4.1. Boost important and DVFS-latency-sensitive tasks

A GUI task might not be busy to warrant driving the frequency high when it wakes up. However, it requires to finish its work within a specific time window to deliver the desired user experience. The right frequency it requires at wakeup will be system dependent. On some underpowered systems it will be high, on other overpowered ones it will be low or 0.

> GUI 任务可能不忙，无法保证在它醒来时将频率驱动得很高。然而，它需要在特定的时间窗口内完成工作，以提供所需的用户体验。唤醒时所需的正确频率将取决于系统。在一些动力不足的系统上，它将是高的，在其他动力过大的系统上它将是低的或 0。

This task can increase its `UCLAMP_MIN` value every time it misses the deadline to ensure on next wake up it runs at a higher performance point. It should try to approach the lowest `UCLAMP_MIN` value that allows to meet its deadline on any particular system to achieve the best possible perf/watt for that system.

> 此任务可以在每次错过最后期限时增加其“UCLAMP_MIN”值，以确保在下次唤醒时以更高的性能点运行。它应该尝试接近允许在任何特定系统上满足其最后期限的最低“UCLAMP_MIN”值，以实现该系统的最佳性能/瓦。

On heterogeneous systems, it might be important for this task to run on a faster CPU.

> 在异构系统上，此任务在更快的 CPU 上运行可能很重要。

**Generally it is advised to perceive the input as performance level or point which will imply both task placement and frequency selection**.

> **通常，建议将输入视为绩效水平或点，这意味着任务安排和频率选择**。

## 4.2. Cap background tasks

Like explained for Android case in the introduction. Any app can lower `UCLAMP_MAX` for some background tasks that don\'t care about performance but could end up being busy and consume unnecessary system resources on the system.

> 就像介绍中为 Android 案例所解释的那样。任何应用程序都可以降低某些后台任务的“UCLAMP_MAX”，这些任务不关心性能，但最终可能会很忙，并消耗系统上不必要的系统资源。

## 4.3. Powersave mode

`sched_util_clamp_max` system wide interface can be used to limit all tasks from operating at the higher performance points which are usually energy inefficient.

This is not unique to uclamp as one can achieve the same by reducing max frequency of the `cpufreq` governor. It can be considered a more convenient alternative interface.

> 这并不是 uclamp 独有的，因为可以通过降低“cpufreq”调速器的最大频率来实现这一点。它可以被认为是一个更方便的替代接口。

## 4.4. Per-app performance restriction

Middleware/Utility can provide the user an option to set `UCLAMP_MIN/MAX` for an app every time it is executed to guarantee a minimum performance point and/or limit it from draining system power at the cost of reduced performance for these apps.

> 中间件/实用程序可以为用户提供每次执行应用程序时为其设置“UCLAMP_MIN/MAX”的选项，以保证最低性能点和/或限制其以降低这些应用程序的性能为代价消耗系统功率。

If you want to prevent your laptop from heating up while on the go from compiling the kernel and happy to sacrifice performance to save power, but still would like to keep your browser performance intact, uclamp makes it possible.

> 如果你想防止你的笔记本电脑在编译内核时发热，并乐于牺牲性能来节省电力，但仍然想保持浏览器性能不变，uclamp 可以实现这一点。

# 5. Limitations

## 5.1. Capping frequency with uclamp_max fails under certain conditions {#uclamp-capping-fail}

If task p0 is capped to run at 512:

```
    p0->uclamp[UCLAMP_MAX] = 512
```

and it shares the `rq` with p1 which is free to run at any performance point:

> 并且它与 p1 共享“rq”，p1 可以在任何性能点自由运行：

```
    p1->uclamp[UCLAMP_MAX] = 1024
```

then due to max aggregation the `rq` will be allowed to reach max performance point:

> 则由于最大聚合，“rq”将被允许达到最大性能点：

```
    rq->uclamp[UCLAMP_MAX] = max(512, 1024) = 1024
```

Assuming both `p0` and `p1` have `UCLAMP_MIN = 0`, then the frequency selection for the `rq` will depend on the actual utilization value of the tasks.

> 假设“p0”和“p1”都具有“UCLAMP_MIN ＝ 0”，则“rq”的频率选择将取决于任务的实际利用值。

If `p1` is a small task but `p0` is a CPU intensive task, then due to the fact that both are running at the same rq, `p1` will cause the frequency capping to be left from the `rq` although p1, which is allowed to run at any performance point, doesn\'t actually need to run at that frequency.

> 如果“p1”是一个小任务，但“p0”是 CPU 密集型任务，则由于两者都以相同的 rq 运行，“p1”将导致从“rq”中保留频率上限，尽管允许在任何性能点运行的 p1 实际上不需要以该频率运行。

## 5.2. UCLAMP_MAX can break PELT (util_avg) signal

`PELT` assumes that frequency will always increase as the signals grow to ensure there\'s always some idle time on the CPU. But with `UCLAMP_MAX`, this frequency increase will be prevented which can lead to no idle time in some circumstances. When there\'s no idle time, a task will stuck in a busy loop, which would result in util_avg being 1024.

Combing with issue described below, this can lead to unwanted frequency spikes when severely capped tasks share the `rq` with a small non capped task.

> 结合下面描述的问题，当严重封顶的任务与小型非封顶任务共享“rq”时，这可能会导致不必要的频率峰值。

As an example if task `p`, which have:

```
    p0->util_avg = 300
    p0->uclamp[UCLAMP_MAX] = 0
```

wakes up on an idle CPU, then it will run at min frequency (Fmin) this CPU is capable of. The max CPU frequency (Fmax) matters here as well, since it designates the shortest computational time to finish the task\'s work on this CPU.

> 在空闲的 CPU 上唤醒，然后它将以该 CPU 所能达到的最小频率(Fmin)运行。最大 CPU 频率(Fmax)在这里也很重要，因为它指定了在该 CPU 上完成任务的最短计算时间。

```
    rq->uclamp[UCLAMP_MAX] = 0
```

If the ratio of Fmax/Fmin is 3, then maximum value will be:

```
    300 * (Fmax/Fmin) = 900
```

which indicates the CPU will still see idle time since 900 is < 1024. The `[actual]()` util_avg will not be 900 though, but somewhere between 300 and 900. As long as there\'s idle time, `p->util_avg` updates will be off by a some margin, but not proportional to Fmax/Fmin.

> 这表示 CPU 仍将看到空闲时间，因为 900 小于 1024。“[实际]()”util_avg 将不是 900，而是介于 300 和 900 之间。只要有空闲时间，“p->util_avg”更新将在一定程度上关闭，但与 Fmax/Fmin 不成正比。

```
    p0->util_avg = 300 + small_error
```

Now if the ratio of Fmax/Fmin is 4, the maximum value becomes:

```
    300 * (Fmax/Fmin) = 1200
```

which is higher than 1024 and indicates that the CPU has no idle time. When this happens, then the `[actual]()` util_avg will become:

> 高于 1024，表示 CPU 没有空闲时间。发生这种情况时，`[actual]()`util_avg 将变为：

```
    p0->util_avg = 1024
```

If task `p1` wakes up on this CPU, which have:

```
    p1->util_avg = 200
    p1->uclamp[UCLAMP_MAX] = 1024
```

then the effective `UCLAMP_MAX` for the CPU will be 1024 according to max aggregation rule. But since the capped `p0` task was running and throttled severely, then the `rq->util_avg` will be:

> 则根据最大聚合规则，CPU 的有效“UCLAMP\_ MAX”将是 1024。但是，由于有上限的“p0”任务正在运行并被严重抑制，因此“rq->util_avg”将为：

```
    p0->util_avg = 1024
    p1->util_avg = 200

    rq->util_avg = 1024
    rq->uclamp[UCLAMP_MAX] = 1024
```

Hence lead to a frequency spike since if p0 wasn\'t throttled we should get:

> 因此导致频率尖峰，因为如果 p0 没有被抑制，我们应该得到：

```
    p0->util_avg = 300
    p1->util_avg = 200

    rq->util_avg = 500
```

and run somewhere near mid performance point of that CPU, not the Fmax we get.

> 并运行在接近 CPU 性能中点的地方，而不是我们得到的 Fmax。

## 5.3. Schedutil response time issues

schedutil has three limitations:

1. Hardware takes non-zero time to respond to any frequency change request. On some platforms can be in the order of few ms.

> 1.硬件需要非零时间来响应任何频率改变请求。在一些平台上可以是几毫秒的数量级。

2. Non fast-switch systems require a worker deadline thread to wake up and perform the frequency change, which adds measurable overhead.

> 2.非快速切换系统需要工人截止日期线程来唤醒并执行频率改变，这增加了可测量的开销。

3. schedutil `rate_limit_us` drops any requests during this `rate_limit_us` window.

> 3.schedutil“rate_limit_us”在该“rate_limit \_us”窗口期间丢弃任何请求。

If a relatively small task is doing critical job and requires a certain performance point when it wakes up and starts running, then all these limitations will prevent it from getting what it wants in the time scale it expects.

> 如果一个相对较小的任务正在做关键的工作，并且在它醒来并开始运行时需要一定的性能点，那么所有这些限制都会阻止它在预期的时间范围内得到它想要的东西。

This limitation is not only impactful when using uclamp, but will be more prevalent as we no longer gradually ramp up or down. We could easily be jumping between frequencies depending on the order tasks wake up, and their respective uclamp values.

> 这种限制不仅在使用 uclamp 时有影响，而且随着我们不再逐渐上升或下降，这种限制将更加普遍。根据任务唤醒的顺序和它们各自的 uclamp 值，我们可以很容易地在频率之间跳跃。

We regard that as a limitation of the capabilities of the underlying system itself.

> 我们认为这是对基础系统本身能力的限制。

There is room to improve the behavior of schedutil `rate_limit_us`, but not much to be done for 1 or 2. They are considered hard limitations of the system.

> schedutil“rate_limit_us”的行为还有改进的空间，但对于 1 或 2 没有太多改进之处。它们被认为是系统的硬性限制。
