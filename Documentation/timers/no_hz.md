---
tip: translate by baidu@2024-05-25 19:54:27
title: "NO_HZ: Reducing Scheduling-Clock Ticks"
---

This document describes Kconfig options and boot parameters that can reduce the number of scheduling-clock interrupts, thereby improving energy efficiency and reducing OS jitter. Reducing OS jitter is important for some types of computationally intensive high-performance computing (HPC) applications and for real-time applications.

> 本文档描述了 Kconfig 选项和引导参数，它们可以减少调度时钟中断的数量，从而提高能源效率并减少操作系统抖动。减少操作系统抖动对于某些类型的计算密集型高性能计算(HPC)应用程序和实时应用程序非常重要。

There are three main ways of managing scheduling-clock interrupts (also known as \"scheduling-clock ticks\" or simply \"ticks\"):

> 管理调度时钟中断的主要方法有三种(也称为"调度时钟信号"或简称"信号")：

1. Never omit scheduling-clock ticks (CONFIG_HZ_PERIODIC=y or CONFIG_NO_HZ=n for older kernels). You normally will -not-want to choose this option.
2. Omit scheduling-clock ticks on idle CPUs (CONFIG_NO_HZ_IDLE=y or CONFIG_NO_HZ=y for older kernels). This is the most common approach, and should be the default.
3. Omit scheduling-clock ticks on CPUs that are either idle or that have only one runnable task (CONFIG_NO_HZ_FULL=y). Unless you are running realtime applications or certain types of HPC workloads, you will normally -not- want this option.

> 1. 永远不要省略调度时钟信号(CONFIG_HZ_PERIODIC=y 或 CONFIG_NO_HZ=n 用于旧内核)。您通常不会选择此选项。
> 2. 省略空闲 CPU 上的调度时钟滴答声(对于旧内核，CONFIG_NO_HZ_idle=y 或 CONFIG_NO_HZ=y)。这是最常见的方法，应该是默认的方法。
> 3. 忽略空闲或只有一个可运行任务的 CPU 上的调度时钟信号(CONFIG_NO_HZ_FULL=y)。除非您正在运行实时应用程序或某些类型的 HPC 工作负载，否则您通常不希望使用此选项。

These three cases are described in the following three sections, followed by a third section on RCU-specific considerations, a fourth section discussing testing, and a fifth and final section listing known issues.

> 以下三节介绍了这三种情况，第三节介绍 RCU 的具体考虑因素，第四节讨论测试，第五节也是最后一节列出了已知问题。

# Never Omit Scheduling-Clock Ticks

Very old versions of Linux from the 1990s and the very early 2000s are incapable of omitting scheduling-clock ticks. It turns out that there are some situations where this old-school approach is still the right approach, for example, in heavy workloads with lots of tasks that use short bursts of CPU, where there are very frequent idle periods, but where these idle periods are also quite short (tens or hundreds of microseconds). For these types of workloads, scheduling clock interrupts will normally be delivered any way because there will frequently be multiple runnable tasks per CPU. In these cases, attempting to turn off the scheduling clock interrupt will have no effect other than increasing the overhead of switching to and from idle and transitioning between user and kernel execution.

> 20 世纪 90 年代和 21 世纪初的非常旧的 Linux 版本无法省略调度时钟信号。事实证明，在某些情况下，这种老派方法仍然是正确的方法，例如，在大量任务使用短 CPU 的繁重工作负载中，有非常频繁的空闲期，但这些空闲期也很短(几十或数百微秒)。对于这些类型的工作负载，调度时钟中断通常会以任何方式传递，因为每个 CPU 通常会有多个可运行的任务。在这些情况下，试图关闭调度时钟中断除了增加切换到空闲和从空闲切换到空闲以及在用户和内核执行之间转换的开销之外，没有任何效果。

This mode of operation can be selected using CONFIG_HZ_PERIODIC=y (or CONFIG_NO_HZ=n for older kernels).

> 可以使用 CONFIG_HZ_PERIODIC=y(或 CONFIG_NO_HZ=n 用于旧内核)来选择此操作模式。

However, if you are instead running a light workload with long idle periods, failing to omit scheduling-clock interrupts will result in excessive power consumption. This is especially bad on battery-powered devices, where it results in extremely short battery lifetimes. If you are running light workloads, you should therefore read the following section.

> 但是，如果您运行的是具有长空闲期的轻型工作负载，则未能省略调度时钟中断将导致过度功耗。这在电池供电的设备上尤其糟糕，会导致电池寿命极短。如果您运行的是轻型工作负载，那么您应该阅读以下部分。

In addition, if you are running either a real-time workload or an HPC workload with short iterations, the scheduling-clock interrupts can degrade your applications performance. If this describes your workload, you should read the following two sections.

> 此外，如果您正在运行实时工作负载或具有短迭代的 HPC 工作负载，则调度时钟中断可能会降低应用程序的性能。如果这描述了您的工作负载，您应该阅读以下两部分。

# Omit Scheduling-Clock Ticks For Idle CPUs

If a CPU is idle, there is little point in sending it a scheduling-clock interrupt. After all, the primary purpose of a scheduling-clock interrupt is to force a busy CPU to shift its attention among multiple duties, and an idle CPU has no duties to shift its attention among.

> 如果 CPU 空闲，那么向它发送调度时钟中断就没有什么意义了。毕竟，调度时钟中断的主要目的是迫使繁忙的 CPU 在多个任务之间转移注意力，而空闲的 CPU 没有任务可以在其中转移注意力。

An idle CPU that is not receiving scheduling-clock interrupts is said to be \"dyntick-idle\", \"in dyntick-idle mode\", \"in nohz mode\", or \"running tickless\". The remainder of this document will use \"dyntick-idle mode\".

> 没有接收调度时钟中断的空闲 CPU 被称为"dyntick idle"、"in dyntick idle mode"、"在 nohz mode"或"running tickless"。本文档的其余部分将使用"动态刻度空闲模式"。

The CONFIG_NO_HZ_IDLE=y Kconfig option causes the kernel to avoid sending scheduling-clock interrupts to idle CPUs, which is critically important both to battery-powered devices and to highly virtualized mainframes. A battery-powered device running a CONFIG_HZ_PERIODIC=y kernel would drain its battery very quickly, easily 2-3 times as fast as would the same device running a CONFIG_NO_HZ_IDLE=y kernel. A mainframe running 1,500 OS instances might find that half of its CPU time was consumed by unnecessary scheduling-clock interrupts. In these situations, there is strong motivation to avoid sending scheduling-clock interrupts to idle CPUs. That said, dyntick-idle mode is not free:

> CONFIG_NO_HZ_IDLE=y Kconfig 选项使内核避免向空闲 CPU 发送调度时钟中断，这对电池供电的设备和高度虚拟化的大型机都至关重要。运行 CONFIG_HZ_PERIODIC=y 内核的电池供电设备的电池消耗速度非常快，很容易是运行 CONFIG_NO_HZ_IDLE=y 内核的相同设备的 2-3 倍。运行 1500 个操作系统实例的大型机可能会发现其一半的 CPU 时间被不必要的调度时钟中断所消耗。在这些情况下，有强烈的动机避免将调度时钟中断发送到空闲 CPU。也就是说，dyntick 空闲模式不是免费的：

1. It increases the number of instructions executed on the path to and from the idle loop.
2. On many architectures, dyntick-idle mode also increases the number of expensive clock-reprogramming operations.

> 1. 它增加了在往返空闲循环的路径上执行的指令数量。
> 2. 在许多架构上，动态时钟空闲模式也增加了昂贵的时钟重新编程操作的数量。

Therefore, systems with aggressive real-time response constraints often run CONFIG_HZ_PERIODIC=y kernels (or CONFIG_NO_HZ=n for older kernels) in order to avoid degrading from-idle transition latencies.

> 因此，具有积极实时响应约束的系统通常运行 CONFIG_HZ_PERIODIC=y 内核(或者对于较旧的内核，CONFIG_NO_HZ=n)，以避免因空闲转换延迟而降级。

There is also a boot parameter \"nohz=\" that can be used to disable dyntick-idle mode in CONFIG_NO_HZ_IDLE=y kernels by specifying \"nohz=off\". By default, CONFIG_NO_HZ_IDLE=y kernels boot with \"nohz=on\", enabling dyntick-idle mode.

> 还有一个引导参数"nohz="，可以通过指定"nohz ＝ off"来禁用 CONFIG_NO_HZ_idle=y 内核中的动态时钟空闲模式。默认情况下，CONFIG_NO_HZ_IDLE=y 内核以"nohz=on\"启动，启用 dyntick 空闲模式。

# Omit Scheduling-Clock Ticks For CPUs With Only One Runnable Task

If a CPU has only one runnable task, there is little point in sending it a scheduling-clock interrupt because there is no other task to switch to. Note that omitting scheduling-clock ticks for CPUs with only one runnable task implies also omitting them for idle CPUs.

> 如果一个 CPU 只有一个可运行的任务，那么向它发送调度时钟中断几乎没有意义，因为没有其他任务可切换。请注意，对于只有一个运行任务的 CPU，省略调度时钟节拍意味着也会对空闲 CPU 省略它们。

The CONFIG_NO_HZ_FULL=y Kconfig option causes the kernel to avoid sending scheduling-clock interrupts to CPUs with a single runnable task, and such CPUs are said to be \"adaptive-ticks CPUs\". This is important for applications with aggressive real-time response constraints because it allows them to improve their worst-case response times by the maximum duration of a scheduling-clock interrupt. It is also important for computationally intensive short-iteration workloads: If any CPU is delayed during a given iteration, all the other CPUs will be forced to wait idle while the delayed CPU finishes. Thus, the delay is multiplied by one less than the number of CPUs. In these situations, there is again strong motivation to avoid sending scheduling-clock interrupts.

> CONFIG_NO_HZ_FULL=y Kconfig 选项使内核避免向具有单个可运行任务的 CPU 发送调度时钟中断，并且这样的 CPU 被称为"自适应时钟 CPU"。这对于具有积极实时响应约束的应用程序来说很重要，因为这允许它们通过调度时钟中断的最大持续时间来提高最坏情况下的响应时间。对于计算密集型的短迭代工作负载来说，这一点也很重要：如果任何 CPU 在给定的迭代过程中被延迟，则所有其他 CPU 将被迫等待空闲，而延迟的 CPU 将完成。因此，延迟被乘以比 CPU 的数量少一个。在这些情况下，再次存在避免发送调度时钟中断的强烈动机。

By default, no CPU will be an adaptive-ticks CPU. The \"nohz_full=\" boot parameter specifies the adaptive-ticks CPUs. For example, \"nohz_full=1,6-8\" says that CPUs 1, 6, 7, and 8 are to be adaptive-ticks CPUs. Note that you are prohibited from marking all of the CPUs as adaptive-tick CPUs: At least one non-adaptive-tick CPU must remain online to handle timekeeping tasks in order to ensure that system calls like gettimeofday() returns accurate values on adaptive-tick CPUs. (This is not an issue for CONFIG_NO_HZ_IDLE=y because there are no running user processes to observe slight drifts in clock rate.) Therefore, the boot CPU is prohibited from entering adaptive-ticks mode. Specifying a \"nohz_full=\" mask that includes the boot CPU will result in a boot-time error message, and the boot CPU will be removed from the mask. Note that this means that your system must have at least two CPUs in order for CONFIG_NO_HZ_FULL=y to do anything for you.

> 默认情况下，没有任何 CPU 是自适应 ticks CPU。"nohz_full=\"引导参数指定自适应 ticks CPU。例如，"nohz_full=1.6-8\"表示 CPU 1、6、7 和 8 将是自适应 ticks CPU。请注意，禁止将所有 CPU 标记为自适应刻度 CPU：至少有一个非自适应刻度 CPU 必须保持联机以处理计时任务，以确保像 gettimeofday()这样的系统调用在自适应刻度 CPU 上返回准确的值。(这不是 CONFIG_NO_HZ_IDLE=y 的问题，因为没有正在运行的用户进程来观察时钟速率的轻微漂移。)因此，禁止引导 CPU 进入自适应滴答模式。指定包含引导 CPU 的"nohz_full=\"掩码将导致引导时间错误消息，并且引导 CPU 将从掩码中删除。请注意，这意味着您的系统必须至少有两个 CPU，CONFIG_NO_HZ_FULL=y 才能为您做任何事情。

Finally, adaptive-ticks CPUs must have their RCU callbacks offloaded. This is covered in the \"RCU IMPLICATIONS\" section below.

> 最后，自适应 ticks CPU 必须卸载其 RCU 回调。这在下面的"RCU 含义"一节中有介绍。

Normally, a CPU remains in adaptive-ticks mode as long as possible. In particular, transitioning to kernel mode does not automatically change the mode. Instead, the CPU will exit adaptive-ticks mode only if needed, for example, if that CPU enqueues an RCU callback.

> 通常情况下，CPU 会尽可能长时间地保持在自适应滴答模式下。特别是，转换到内核模式不会自动更改模式。相反，只有在需要时，CPU 才会退出自适应 ticks 模式，例如，如果 CPU 将 RCU 回调排入队列。

Just as with dyntick-idle mode, the benefits of adaptive-tick mode do not come for free:

> 与 dyntick 空闲模式一样，自适应滴答模式的好处也不是免费的：

1. CONFIG_NO_HZ_FULL selects CONFIG_NO_HZ_COMMON, so you cannot run adaptive ticks without also running dyntick idle. This dependency extends down into the implementation, so that all of the costs of CONFIG_NO_HZ_IDLE are also incurred by CONFIG_NO_HZ_FULL.
2. The user/kernel transitions are slightly more expensive due to the need to inform kernel subsystems (such as RCU) about the change in mode.
3. POSIX CPU timers prevent CPUs from entering adaptive-tick mode. Real-time applications needing to take actions based on CPU time consumption need to use other means of doing so.
4. If there are more perf events pending than the hardware can accommodate, they are normally round-robined so as to collect all of them over time. Adaptive-tick mode may prevent this round-robining from happening. This will likely be fixed by preventing CPUs with large numbers of perf events pending from entering adaptive-tick mode.
5. Scheduler statistics for adaptive-tick CPUs may be computed slightly differently than those for non-adaptive-tick CPUs. This might in turn perturb load-balancing of real-time tasks.

Although improvements are expected over time, adaptive ticks is quite useful for many types of real-time and compute-intensive applications. However, the drawbacks listed above mean that adaptive ticks should not (yet) be enabled by default.

> 1. CONFIG_NO_HZ_FULL 选择 CONFIG \_NO_HZ_COMMON，因此如果不同时运行 dyntick idle，则无法运行自适应 tick。这种依赖性向下扩展到实现中，因此 CONFIG_NO_HZ_IDLE 的所有成本也由 CONFIG_NO_HZ_FULL 承担。
> 2. 由于需要向内核子系统(如 RCU)通知模式的变化，用户/内核转换的成本略高。
> 3. POSIX CPU 定时器防止 CPU 进入自适应滴答模式。需要根据 CPU 时间消耗采取行动的实时应用程序需要使用其他方法。
> 4. 如果挂起的 perf 事件比硬件所能容纳的要多，则通常会对它们进行舍入，以便随着时间的推移收集所有这些事件。自适应滴答模式可以防止这种循环鸣叫的发生。这可能会通过防止具有大量性能事件挂起的 CPU 进入自适应勾选模式来解决。
> 5. 自适应滴答 CPU 的调度器统计数据的计算可能与非自适应滴答 CPU 略有不同。这可能反过来干扰实时任务的负载平衡。

> 尽管预计会随着时间的推移而有所改进，但自适应记号对于许多类型的实时和计算密集型应用程序来说非常有用。但是，上面列出的缺点意味着默认情况下不应该(还)启用自适应记号。

# RCU Implications

There are situations in which idle CPUs cannot be permitted to enter either dyntick-idle mode or adaptive-tick mode, the most common being when that CPU has RCU callbacks pending.

> 在某些情况下，不能允许空闲 CPU 进入动态时钟空闲模式或自适应时钟模式，最常见的情况是当该 CPU 有 RCU 回调挂起时。

Avoid this by offloading RCU callback processing to \"rcuo\" kthreads using the CONFIG_RCU_NOCB_CPU=y Kconfig option. The specific CPUs to offload may be selected using The \"rcu_nocbs=\" kernel boot parameter, which takes a comma-separated list of CPUs and CPU ranges, for example, \"1,3-5\" selects CPUs 1, 3, 4, and 5. Note that CPUs specified by the \"nohz_full\" kernel boot parameter are also offloaded.

> 通过使用 CONFIG_RCU_NOCB_CPU=y Kconfig 选项将 RCU 回调处理卸载到"rcuo"kthreads 来避免这种情况。可以使用"rcu_nocbs="内核引导参数来选择要卸载的特定 CPU，该参数采用逗号分隔的 CPU 和 CPU 范围列表，例如，"1,3,5"选择 CPU 1、3、4 和 5。请注意，由"nohz_full"内核引导参数指定的 CPU 也会被卸载。

The offloaded CPUs will never queue RCU callbacks, and therefore RCU never prevents offloaded CPUs from entering either dyntick-idle mode or adaptive-tick mode. That said, note that it is up to userspace to pin the \"rcuo\" kthreads to specific CPUs if desired. Otherwise, the scheduler will decide where to run them, which might or might not be where you want them to run.

> 卸载的 CPU 永远不会对 RCU 回调进行排队，因此 RCU 永远不会阻止卸载的 CPU 进入动态时钟空闲模式或自适应时钟模式。也就是说，请注意，如果需要，将"rcuo"k 线程固定到特定的 CPU 取决于用户空间。否则，调度程序将决定在哪里运行它们，可能是也可能不是您希望它们在哪里运行。

# Testing

So you enable all the OS-jitter features described in this document, but do not see any change in your workload\'s behavior. Is this because your workload isn\'t affected that much by OS jitter, or is it because something else is in the way? This section helps answer this question by providing a simple OS-jitter test suite, which is available on branch master of the following git archive:

> 因此，您启用了本文档中描述的所有操作系统抖动功能，但没有看到工作负载行为的任何变化。这是因为你的工作负载没有受到操作系统抖动的太大影响，还是因为其他因素阻碍了你的工作？本节通过提供一个简单的操作系统抖动测试套件来帮助回答这个问题，该套件可在以下 git 归档的分支主机上获得：

[git://git.kernel.org/pub/scm/linux/kernel/git/frederic/dynticks-testing.git](git://git.kernel.org/pub/scm/linux/kernel/git/frederic/dynticks-testing.git)

Clone this archive and follow the instructions in the README file. This test procedure will produce a trace that will allow you to evaluate whether or not you have succeeded in removing OS jitter from your system. If this trace shows that you have removed OS jitter as much as is possible, then you can conclude that your workload is not all that sensitive to OS jitter.

> 克隆此存档并按照自述文件中的说明进行操作。此测试过程将生成一个跟踪，使您能够评估是否成功地从系统中消除了操作系统抖动。如果这个跟踪显示您已经尽可能多地消除了操作系统抖动，那么您可以得出结论，您的工作负载对操作系统抖动并不那么敏感。

Note: this test requires that your system have at least two CPUs. We do not currently have a good way to remove OS jitter from single-CPU systems.

> 注意：此测试要求您的系统至少有两个 CPU。我们目前没有一个好的方法来消除单 CPU 系统的操作系统抖动。

# Known Issues

- Dyntick-idle slows transitions to and from idle slightly. In practice, this has not been a problem except for the most aggressive real-time workloads, which have the option of disabling dyntick-idle mode, an option that most of them take. However, some workloads will no doubt want to use adaptive ticks to eliminate scheduling-clock interrupt latencies. Here are some options for these workloads:

> - Dyntick idle 略微减慢了到 idle 和从 idle 转换的速度。在实践中，这并不是一个问题，除了最激进的实时工作负载，它们可以选择禁用 dyntick 空闲模式，这是大多数工作负载都会选择的选项。然而，一些工作负载无疑希望使用自适应记号来消除调度时钟中断延迟。以下是这些工作负载的一些选项：

a. Use PMQOS from userspace to inform the kernel of your latency requirements (preferred).
b. On x86 systems, use the \"idle=mwait\" boot parameter.
c. On x86 systems, use the \"intel_idle.max_cstate=\" to limit \` the maximum C-state depth.
d. On x86 systems, use the \"idle=poll\" boot parameter. However, please note that use of this parameter can cause your CPU to overheat, which may cause thermal throttling to degrade your latencies \-- and that this degradation can be even worse than that of dyntick-idle. Furthermore, this parameter effectively disables Turbo Mode on Intel CPUs, which can significantly reduce maximum performance.

> a.使用用户空间的 PMQOS 向内核通知您的延迟要求(首选)。
> c. 在 x86 系统上，使用\"intel_idle.max_state=\"来限制最大 C 状态深度。
> d.在 x86 系统上，使用"idle=poll"引导参数。但是，请注意，使用此参数可能会导致 CPU 过热，这可能会导致热节流降低延迟，而且这种降低可能比 dyntick 空闲更严重。此外，此参数有效地禁用了英特尔 CPU 上的 Turbo 模式，这会显著降低最大性能。

- Adaptive-ticks slows user/kernel transitions slightly. This is not expected to be a problem for computationally intensive workloads, which have few such transitions. Careful benchmarking will be required to determine whether or not other workloads are significantly affected by this effect.

> - 自适应记号会略微减慢用户/内核转换的速度。对于计算密集型工作负载来说，这预计不会是一个问题，因为它们很少有这样的转换。需要仔细进行基准测试，以确定其他工作负载是否受到这种影响的显著影响。

- Adaptive-ticks does not do anything unless there is only one runnable task for a given CPU, even though there are a number of other situations where the scheduling-clock tick is not needed. To give but one example, consider a CPU that has one runnable high-priority SCHED_FIFO task and an arbitrary number of low-priority SCHED_OTHER tasks. In this case, the CPU is required to run the SCHED_FIFO task until it either blocks or some other higher-priority task awakens on (or is assigned to) this CPU, so there is no point in sending a scheduling-clock interrupt to this CPU. However, the current implementation nevertheless sends scheduling-clock interrupts to CPUs having a single runnable SCHED_FIFO task and multiple runnable SCHED_OTHER tasks, even though these interrupts are unnecessary.

> - 除非给定的 CPU 只有一个可运行的任务，否则自适应滴答声不会起任何作用，即使在许多其他情况下不需要调度时钟滴答声。只举一个例子，考虑一个 CPU，它有一个可运行的高优先级 SCHED_FIFO 任务和任意数量的低优先级 SCHED_OTHER 任务。在这种情况下，CPU 需要运行 SCHED_FIFO 任务，直到它阻塞或某个其他更高优先级的任务在该 CPU 上唤醒(或被分配给)，所以向该 CPU 发送调度时钟中断没有意义。然而，当前实现仍然向具有单个可运行的 SCHED_FIFO 任务和多个可运行的 SHEED_OTHER 任务的 CPU 发送调度时钟中断，即使这些中断是不必要的。

And even when there are multiple runnable tasks on a given CPU, there is little point in interrupting that CPU until the current running task\'s timeslice expires, which is almost always way longer than the time of the next scheduling-clock interrupt.

> 即使在一个给定的 CPU 上有多个可运行的任务，在当前运行任务的时间段到期之前中断该 CPU 也没有什么意义，这几乎总是比下一次调度时钟中断的时间长得多。

Better handling of these sorts of situations is future work.

- A reboot is required to reconfigure both adaptive idle and RCU callback offloading. Runtime reconfiguration could be provided if needed, however, due to the complexity of reconfiguring RCU at runtime, there would need to be an earthshakingly good reason. Especially given that you have the straightforward option of simply offloading RCU callbacks from all CPUs and pinning them where you want them whenever you want them pinned.

> - 需要重新启动以重新配置自适应空闲和 RCU 回调卸载。如果需要，可以提供运行时重新配置，但是，由于在运行时配置 RCU 的复杂性，需要有一个令人震惊的好理由。特别是考虑到您可以简单地从所有 CPU 中卸载 RCU 回调，并在任何时候将它们固定在您想要的位置。

- Additional configuration is required to deal with other sources of OS jitter, including interrupts and system-utility tasks and processes. This configuration normally involves binding interrupts and tasks to particular CPUs.

> - 需要额外的配置来处理操作系统抖动的其他来源，包括中断和系统实用程序任务和进程。这种配置通常包括将中断和任务绑定到特定的 CPU。

- Some sources of OS jitter can currently be eliminated only by constraining the workload. For example, the only way to eliminate OS jitter due to global TLB shootdowns is to avoid the unmapping operations (such as kernel module unload operations) that result in these shootdowns. For another example, page faults and TLB misses can be reduced (and in some cases eliminated) by using huge pages and by constraining the amount of memory used by the application. Pre-faulting the working set can also be helpful, especially when combined with the mlock() and mlockall() system calls.

> - 一些操作系统抖动的来源目前只能通过限制工作负载来消除。例如，消除全局 TLB 故障切换引起的操作系统抖动的唯一方法是避免导致这些故障切换的取消映射操作(如内核模块卸载操作)。例如，通过使用巨大的页面和限制应用程序使用的内存量，可以减少(在某些情况下消除)页面错误和 TLB 未命中。预错操作工作集也很有帮助，尤其是当与 mlock()和 mlockall()系统调用结合使用时。

- Unless all CPUs are idle, at least one CPU must keep the scheduling-clock interrupt going in order to support accurate timekeeping.

> - 除非所有 CPU 都处于空闲状态，否则至少有一个 CPU 必须保持调度时钟中断，以支持精确的计时。

- If there might potentially be some adaptive-ticks CPUs, there will be at least one CPU keeping the scheduling-clock interrupt going, even if all CPUs are otherwise idle.

> - 如果可能存在一些自适应 tick CPU，则至少有一个 CPU 保持调度时钟中断，即使所有 CPU 都处于空闲状态。

Better handling of this situation is ongoing work.

- Some process-handling operations still require the occasional scheduling-clock tick. These operations include calculating CPU load, maintaining sched average, computing CFS entity vruntime, computing avenrun, and carrying out load balancing. They are currently accommodated by scheduling-clock tick every second or so. On-going work will eliminate the need even for these infrequent scheduling-clock ticks.

> - 一些进程处理操作仍然需要偶尔的调度时钟滴答声。这些操作包括计算 CPU 负载、维护 sched 平均值、计算 CFS 实体 vruntime、计算 averun 和执行负载平衡。目前，它们是通过每隔一秒左右安排一次时钟来适应的。继续工作将消除对这些不频繁的安排时钟的需求。
