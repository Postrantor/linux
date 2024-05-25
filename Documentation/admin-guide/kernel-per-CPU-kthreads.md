---
tip: translate by baidu@2024-01-30 21:35:12
---
---
title: Reducing OS jitter due to per-cpu kthreads
---


This document lists per-CPU kthreads in the Linux kernel and presents options to control their OS jitter. Note that non-per-CPU kthreads are not listed here. To reduce OS jitter from non-per-CPU kthreads, bind them to a \"housekeeping\" CPU dedicated to such work.

> 本文档列出了Linux内核中每个CPU的k线程，并提供了控制其操作系统抖动的选项。请注意，此处未列出非每CPU k线程。为了减少非每CPU k线程的操作系统抖动，请将它们绑定到专门用于此类工作的“内务管理”CPU。

# References

-   Documentation/core-api/irq/irq-affinity.rst: Binding interrupts to sets of CPUs.

-   Documentation/admin-guide/cgroup-v1: Using cgroups to bind tasks to sets of CPUs.

-   man taskset: Using the taskset command to bind tasks to sets of CPUs.


-   man sched_setaffinity: Using the sched_setaffinity() system call to bind tasks to sets of CPUs.

> -man sched_setafinity：使用sched_setaaffinity（）系统调用将任务绑定到CPU集。


-   /sys/devices/system/cpu/cpuN/online: Control CPU N\'s hotplug state, writing \"0\" to offline and \"1\" to online.

> -/sys/devices/system/cpu/cpuN/online：控制cpu N的热插拔状态，将“0\”写入脱机，将“1\”写入联机。

-   In order to locate kernel-generated OS jitter on CPU N:

    > cd /sys/kernel/tracing echo 1 \> max_graph_depth \# Increase the \"1\" for more detail echo function_graph \> current_tracer \# run workload cat per_cpu/cpuN/trace

# kthreads

Name:

:   ehca_comp/%u

Purpose:

:   Periodically process Infiniband-related work.

To reduce its OS jitter, do any of the following:


1.  Don\'t use eHCA Infiniband hardware, instead choosing hardware that does not require per-CPU kthreads. This will prevent these kthreads from being created in the first place. (This will work for most people, as this hardware, though important, is relatively old and is produced in relatively low unit volumes.)

> 1.不要使用eHCA Infiniband硬件，而是选择不需要每个CPU k线程的硬件。这将从一开始就阻止创建这些k线程。（这将适用于大多数人，因为这种硬件虽然很重要，但相对较旧，单位产量相对较低。）

2.  Do all eHCA-Infiniband-related work on other CPUs, including interrupts.

> 2.在其他CPU上执行所有与eHCA Infiniband相关的工作，包括中断。

3.  Rework the eHCA driver so that its per-CPU kthreads are provisioned only on selected CPUs.

> 3.返工eHCA驱动程序，使其每个CPU的k线程仅在选定的CPU上提供。

Name:

:   irq/%d-%s

Purpose:

:   Handle threaded interrupts.

To reduce its OS jitter, do the following:


1.  Use irq affinity to force the irq threads to execute on some other CPU.

> 1.使用irq仿射来强制irq线程在其他CPU上执行。

Name:

:   [kcmtpd_ctr]()%d

Purpose:

:   Handle Bluetooth work.

To reduce its OS jitter, do one of the following:


1.  Don\'t use Bluetooth, in which case these kthreads won\'t be created in the first place.

> 1.不要使用蓝牙，在这种情况下，这些kthread将不会首先创建。

2.  Use irq affinity to force Bluetooth-related interrupts to occur on some other CPU and furthermore initiate all Bluetooth activity on some other CPU.

> 2.使用irq亲和性来强制蓝牙相关中断发生在某些其他CPU上，并进一步启动某些其他CPU的所有蓝牙活动。

Name:

:   ksoftirqd/%u

Purpose:

:   Execute softirq handlers when threaded or when under heavy load.


To reduce its OS jitter, each softirq vector must be handled separately as follows:

> 为了减少其操作系统抖动，必须按照以下方式分别处理每个softirq矢量：

## TIMER_SOFTIRQ

Do all of the following:


1.  To the extent possible, keep the CPU out of the kernel when it is non-idle, for example, by avoiding system calls and by forcing both kernel threads and interrupts to execute elsewhere.

> 1.在可能的范围内，当CPU非空闲时，将其排除在内核之外，例如，通过避免系统调用和强制内核线程和中断在其他地方执行。

2.  Build with CONFIG_HOTPLUG_CPU=y. After boot completes, force the CPU offline, then bring it back online. This forces recurring timers to migrate elsewhere. If you are concerned with multiple CPUs, force them all offline before bringing the first one back online. Once you have onlined the CPUs in question, do not offline any other CPUs, because doing so could force the timer back onto one of the CPUs in question.

> 2.使用CONFIG_HOTPLUG_CPU=y构建。引导完成后，强制CPU脱机，然后使其重新联机。这迫使重复出现的计时器迁移到其他地方。如果您关心多个CPU，请强制它们全部脱机，然后再将第一个恢复联机。一旦您将有问题的CPU联机，就不要使任何其他CPU脱机，因为这样做可能会迫使计时器回到有问题的一个CPU上。

## NET_TX_SOFTIRQ and NET_RX_SOFTIRQ

Do all of the following:

1.  Force networking interrupts onto other CPUs.
2.  Initiate any network I/O on other CPUs.

3.  Once your application has started, prevent CPU-hotplug operations from being initiated from tasks that might run on the CPU to be de-jittered. (It is OK to force this CPU offline and then bring it back online before you start your application.)

> 3.一旦您的应用程序启动，请防止从可能在要消除抖动的CPU上运行的任务启动CPU热插拔操作。（在启动应用程序之前，可以强制此CPU脱机，然后使其重新联机。）

## BLOCK_SOFTIRQ

Do all of the following:

1.  Force block-device interrupts onto some other CPU.
2.  Initiate any block I/O on other CPUs.

3.  Once your application has started, prevent CPU-hotplug operations from being initiated from tasks that might run on the CPU to be de-jittered. (It is OK to force this CPU offline and then bring it back online before you start your application.)

> 3.一旦您的应用程序启动，请防止从可能在要消除抖动的CPU上运行的任务启动CPU热插拔操作。（在启动应用程序之前，可以强制此CPU脱机，然后使其重新联机。）

## IRQ_POLL_SOFTIRQ

Do all of the following:

1.  Force block-device interrupts onto some other CPU.
2.  Initiate any block I/O and block-I/O polling on other CPUs.

3.  Once your application has started, prevent CPU-hotplug operations from being initiated from tasks that might run on the CPU to be de-jittered. (It is OK to force this CPU offline and then bring it back online before you start your application.)

> 3.一旦您的应用程序启动，请防止从可能在要消除抖动的CPU上运行的任务启动CPU热插拔操作。（在启动应用程序之前，可以强制此CPU脱机，然后使其重新联机。）

## TASKLET_SOFTIRQ

Do one or more of the following:


1.  Avoid use of drivers that use tasklets. (Such drivers will contain calls to things like tasklet_schedule().)

> 1.避免使用使用小任务的驱动程序。（这样的驱动程序将包含对tasklet_schedule（）之类的调用。）
2.  Convert all drivers that you must use from tasklets to workqueues.

3.  Force interrupts for drivers using tasklets onto other CPUs, and also do I/O involving these drivers on other CPUs.

> 3.将使用小任务的驱动程序的中断强制到其他CPU上，并在其他CPU上执行涉及这些驱动程序的I/O。

## SCHED_SOFTIRQ

Do all of the following:


1.  Avoid sending scheduler IPIs to the CPU to be de-jittered, for example, ensure that at most one runnable kthread is present on that CPU. If a thread that expects to run on the de-jittered CPU awakens, the scheduler will send an IPI that can result in a subsequent SCHED_SOFTIRQ.

> 1.避免将调度程序IPI发送到要消除抖动的CPU，例如，确保该CPU上最多有一个可运行的kthread。如果期望在消除抖动的CPU上运行的线程唤醒，则调度程序将发送一个IPI，该IPI可能导致后续的SCHED_SOFTIRQ。

2.  CONFIG_NO_HZ_FULL=y and ensure that the CPU to be de-jittered is marked as an adaptive-ticks CPU using the \"nohz_full=\" boot parameter. This reduces the number of scheduler-clock interrupts that the de-jittered CPU receives, minimizing its chances of being selected to do the load balancing work that runs in SCHED_SOFTIRQ context.

> 2.CONFIG_NO_HZ_FULL=y，并确保使用“nohz_FULL=\”引导参数将要消除抖动的CPU标记为自适应ticks CPU。这减少了消除抖动的CPU接收到的调度程序时钟中断的数量，最大限度地减少了其被选中执行在SCHED_SOFTIRQ上下文中运行的负载平衡工作的机会。

3.  To the extent possible, keep the CPU out of the kernel when it is non-idle, for example, by avoiding system calls and by forcing both kernel threads and interrupts to execute elsewhere. This further reduces the number of scheduler-clock interrupts received by the de-jittered CPU.

> 3.在可能的范围内，当CPU处于非空闲状态时，将其排除在内核之外，例如，通过避免系统调用和强制内核线程和中断在其他地方执行。这进一步减少了去抖动CPU接收到的调度器时钟中断的数量。

## HRTIMER_SOFTIRQ

Do all of the following:


1.  To the extent possible, keep the CPU out of the kernel when it is non-idle. For example, avoid system calls and force both kernel threads and interrupts to execute elsewhere.

> 1.当CPU处于非空闲状态时，尽可能将其排除在内核之外。例如，避免系统调用，并强制内核线程和中断在其他地方执行。

2.  Build with CONFIG_HOTPLUG_CPU=y. Once boot completes, force the CPU offline, then bring it back online. This forces recurring timers to migrate elsewhere. If you are concerned with multiple CPUs, force them all offline before bringing the first one back online. Once you have onlined the CPUs in question, do not offline any other CPUs, because doing so could force the timer back onto one of the CPUs in question.

> 2.使用CONFIG_HOTPLUG_CPU=y构建。引导完成后，强制CPU脱机，然后使其重新联机。这迫使重复出现的计时器迁移到其他地方。如果您关心多个CPU，请强制它们全部脱机，然后再将第一个恢复联机。一旦您将有问题的CPU联机，就不要使任何其他CPU脱机，因为这样做可能会迫使计时器回到有问题的一个CPU上。

## RCU_SOFTIRQ

Do at least one of the following:


1.  Offload callbacks and keep the CPU in either dyntick-idle or adaptive-ticks state by doing all of the following:

> 1.通过执行以下所有操作，卸载回调并保持CPU处于动态时钟空闲或自适应时钟状态：
    a.  CONFIG_NO_HZ_FULL=y and ensure that the CPU to be de-jittered is marked as an adaptive-ticks CPU using the \"nohz_full=\" boot parameter. Bind the rcuo kthreads to housekeeping CPUs, which can tolerate OS jitter.
    b.  To the extent possible, keep the CPU out of the kernel when it is non-idle, for example, by avoiding system calls and by forcing both kernel threads and interrupts to execute elsewhere.

2.  Enable RCU to do its processing remotely via dyntick-idle by doing all of the following:

> 2.通过执行以下所有操作，使RCU能够通过dyntick idle远程进行处理：
    a.  Build with CONFIG_NO_HZ=y.
    b.  Ensure that the CPU goes idle frequently, allowing other CPUs to detect that it has passed through an RCU quiescent state. If the kernel is built with CONFIG_NO_HZ_FULL=y, userspace execution also allows other CPUs to detect that the CPU in question has passed through a quiescent state.
    c.  To the extent possible, keep the CPU out of the kernel when it is non-idle, for example, by avoiding system calls and by forcing both kernel threads and interrupts to execute elsewhere.

Name:

:   kworker/%u:%d%s (cpu, id, priority)

Purpose:

:   Execute workqueue requests

To reduce its OS jitter, do any of the following:


1.  Run your workload at a real-time priority, which will allow preempting the kworker daemons.

> 1.以实时优先级运行工作负载，这将允许抢占kworker守护进程。

2.  A given workqueue can be made visible in the sysfs filesystem by passing the WQ_SYSFS to that workqueue\'s alloc_workqueue(). Such a workqueue can be confined to a given subset of the CPUs using the `/sys/devices/virtual/workqueue/*/cpumask` sysfs files. The set of WQ_SYSFS workqueues can be displayed using \"ls /sys/devices/virtual/workqueue\". That said, the workqueues maintainer would like to caution people against indiscriminately sprinkling WQ_SYSFS across all the workqueues. The reason for caution is that it is easy to add WQ_SYSFS, but because sysfs is part of the formal user/kernel API, it can be nearly impossible to remove it, even if its addition was a mistake.

> 2.通过将WQ_sysfs传递给给定工作队列的alloc_workqueue（），可以使该工作队列在sysfs文件系统中可见。这样的工作队列可以使用“/sys/device/virtual/workqueue/*/cpumask”sysfs文件限制在给定的CPU子集内。WQ_SYSFS工作队列集可以使用\“ls/sys/devices/virtual/workqueue\”显示。也就是说，工作队列维护人员希望提醒人们不要在所有工作队列中不加区分地散布WQ_SYSFS。需要注意的原因是添加WQ_SYSFS很容易，但由于SYSFS是正式用户/内核API的一部分，因此几乎不可能删除它，即使添加它是一个错误。

3.  Do any of the following needed to avoid jitter that your application cannot tolerate:

> 3.执行以下任何必要操作，以避免应用程序无法容忍的抖动：
    a.  Build your kernel with CONFIG_SLUB=y rather than CONFIG_SLAB=y, thus avoiding the slab allocator\'s periodic use of each CPU\'s workqueues to run its cache_reap() function.

    b.  Avoid using oprofile, thus avoiding OS jitter from wq_sync_buffer().

    c.  Limit your CPU frequency so that a CPU-frequency governor is not required, possibly enlisting the aid of special heatsinks or other cooling technologies. If done correctly, and if you CPU architecture permits, you should be able to build your kernel with CONFIG_CPU_FREQ=n to avoid the CPU-frequency governor periodically running on each CPU, including cs_dbs_timer() and od_dbs_timer().

        WARNING: Please check your CPU specifications to make sure that this is safe on your particular system.

    d.  As of v3.18, Christoph Lameter\'s on-demand vmstat workers commit prevents OS jitter due to vmstat_update() on CONFIG_SMP=y systems. Before v3.18, is not possible to entirely get rid of the OS jitter, but you can decrease its frequency by writing a large value to /proc/sys/vm/stat_interval. The default value is HZ, for an interval of one second. Of course, larger values will make your virtual-memory statistics update more slowly. Of course, you can also run your workload at a real-time priority, thus preempting vmstat_update(), but if your workload is CPU-bound, this is a bad idea. However, there is an RFC patch from Christoph Lameter (based on an earlier one from Gilad Ben-Yossef) that reduces or even eliminates vmstat overhead for some workloads at <https://lore.kernel.org/r/00000140e9dfd6bd-40db3d4f-c1be-434f-8132-7820f81bb586-000000@email.amazonses.com>.

    e.  If running on high-end powerpc servers, build with CONFIG_PPC_RTAS_DAEMON=n. This prevents the RTAS daemon from running on each CPU every second or so. (This will require editing Kconfig files and will defeat this platform\'s RAS functionality.) This avoids jitter due to the rtas_event_scan() function. WARNING: Please check your CPU specifications to make sure that this is safe on your particular system.

    f.  If running on Cell Processor, build your kernel with CBE_CPUFREQ_SPU_GOVERNOR=n to avoid OS jitter from spu_gov_work(). WARNING: Please check your CPU specifications to make sure that this is safe on your particular system.

    g.  If running on PowerMAC, build your kernel with CONFIG_PMAC_RACKMETER=n to disable the CPU-meter, avoiding OS jitter from rackmeter_do_timer().

Name:

:   rcuc/%u

Purpose:

:   Execute RCU callbacks in CONFIG_RCU_BOOST=y kernels.

To reduce its OS jitter, do at least one of the following:


1.  Build the kernel with CONFIG_PREEMPT=n. This prevents these kthreads from being created in the first place, and also obviates the need for RCU priority boosting. This approach is feasible for workloads that do not require high degrees of responsiveness.

> 1.使用CONFIG_PREEMPT=n构建内核。这从一开始就防止了这些k线程的创建，也避免了RCU优先级提升的需要。这种方法对于不需要高度响应的工作负载是可行的。

2.  Build the kernel with CONFIG_RCU_BOOST=n. This prevents these kthreads from being created in the first place. This approach is feasible only if your workload never requires RCU priority boosting, for example, if you ensure frequent idle time on all CPUs that might execute within the kernel.

> 2.使用CONFIG_RCU_BOOST=n构建内核。这从一开始就阻止了这些k线程的创建。只有当您的工作负载从不需要RCU优先级提升时，这种方法才是可行的，例如，如果您确保所有可能在内核内执行的CPU都有频繁的空闲时间。

3.  Build with CONFIG_RCU_NOCB_CPU=y and boot with the rcu_nocbs= boot parameter offloading RCU callbacks from all CPUs susceptible to OS jitter. This approach prevents the rcuc/%u kthreads from having any work to do, so that they are never awakened.

> 3.使用CONFIG_RCU_NOCB_CPU=y进行构建，并使用RCU_nocbs=引导参数进行引导，从而从易受操作系统抖动影响的所有CPU卸载RCU回调。这种方法可以防止rcc/%u kthread有任何工作要做，因此它们永远不会被唤醒。

4.  Ensure that the CPU never enters the kernel, and, in particular, avoid initiating any CPU hotplug operations on this CPU. This is another way of preventing any callbacks from being queued on the CPU, again preventing the rcuc/%u kthreads from having any work to do.

> 4.确保CPU永远不会进入内核，特别是避免在此CPU上启动任何CPU热插拔操作。这是防止任何回调在CPU上排队的另一种方法，再次防止rcc/%u k线程有任何工作要做。

Name:

:   rcuop/%d and rcuos/%d

Purpose:

:   Offload RCU callbacks from the corresponding CPU.

To reduce its OS jitter, do at least one of the following:


1.  Use affinity, cgroups, or other mechanism to force these kthreads to execute on some other CPU.

> 1.使用亲和性、cgroups或其他机制强制这些k线程在其他CPU上执行。

2.  Build with CONFIG_RCU_NOCB_CPU=n, which will prevent these kthreads from being created in the first place. However, please note that this will not eliminate OS jitter, but will instead shift it to RCU_SOFTIRQ.

> 2.使用CONFIG_RCU_NOCB_CPU=n进行构建，这将从一开始就阻止创建这些k线程。但是，请注意，这不会消除操作系统抖动，而是会将其转换为RCU_SOFTIRQ。
