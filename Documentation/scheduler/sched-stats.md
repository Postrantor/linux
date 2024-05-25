---
tip: translate by baidu@2024-01-17 09:01:08
title: Scheduler Statistics
---

Version 15 of schedstats dropped counters for some sched_yield: yld_exp_empty, yld_act_empty and yld_both_empty. Otherwise, it is identical to version 14.

> 第 15 版的 schedstat 删除了一些 sched_feld 的计数器：yld_exp_empty、yld_act_empty 和 yld_both_empty。否则，它与版本 14 相同。

Version 14 of schedstats includes support for sched_domains, which hit the mainline kernel in 2.6.20 although it is identical to the stats from version 12 which was in the kernel from 2.6.13-2.6.19 (version 13 never saw a kernel release). Some counters make more sense to be per-runqueue; other to be per-domain. Note that domains (and their associated information) will only be pertinent and available on machines utilizing CONFIG_SMP.

> schedstats 的第 14 版包括对 sched_domains 的支持，它在 2.6.20 中进入主线内核，尽管它与 2.6.13-2.6.19 内核中的第 12 版的统计数据相同（第 13 版从未发布过内核）。有些计数器按运行队列更有意义；其他按域。请注意，域（及其相关信息）仅在使用 CONFIG_SMP 的计算机上是相关的和可用的。

In version 14 of schedstat, there is at least one level of domain statistics for each cpu listed, and there may well be more than one domain. Domains have no particular names in this implementation, but the highest numbered one typically arbitrates balancing across all the cpus on the machine, while domain0 is the most tightly focused domain, sometimes balancing only between pairs of cpus. At this time, there are no architectures which need more than three domain levels. The first field in the domain stats is a bit map indicating which cpus are affected by that domain.

> 在 schedstat 的版本 14 中，列出的每个 cpu 至少有一个级别的域统计信息，而且很可能有多个域。在这个实现中，域没有特定的名称，但编号最高的域通常仲裁机器上所有 cpu 之间的平衡，而 domain0 是关注度最高的域，有时只在 cpu 对之间进行平衡。目前，没有任何体系结构需要超过三个域级别。域统计中的第一个字段是一个位图，指示哪些 cpu 受到该域的影响。

These fields are counters, and only increment. Programs which make use of these will need to start with a baseline observation and then calculate the change in the counters at each subsequent observation. A perl script which does this for many of the fields is available at

> 这些字段是计数器，并且只能递增。利用这些的程序需要从基线观察开始，然后计算每次后续观察时计数器的变化。为许多字段执行此操作的 perl 脚本可在

> <http://eaglet.pdxhosts.com/rick/linux/schedstat/>

Note that any such script will necessarily be version-specific, as the main reason to change versions is changes in the output format. For those wishing to write their own scripts, the fields are described here.

> 请注意，任何此类脚本都必须是特定于版本的，因为更改版本的主要原因是输出格式的更改。对于那些希望编写自己的脚本的人，这里描述了这些字段。

# CPU statistics

```
cpu\<N\> 1 2 3 4 5 6 7 8 9
```

First field is a sched_yield() statistic:

> 1.  \# of times sched_yield() was called

Next three are schedule() statistics:

> 2.  \# This field is a legacy array expiration count field used in the O(1) scheduler. We kept it for ABI compatibility, but it is always set to zero.
> 3.  \# of times schedule() was called
> 4.  \# of times schedule() left the processor idle

Next two are try_to_wake_up() statistics:

> 5.  \# of times try_to_wake_up() was called
> 6.  \# of times try_to_wake_up() was called to wake up the local cpu

Next three are statistics describing scheduling latency:

> 7.  sum of all time spent running by tasks on this processor (in nanoseconds)
> 8.  sum of all time spent waiting to run by tasks on this processor (in nanoseconds)
> 9.  \# of timeslices run on this cpu

# Domain statistics

One of these is produced per domain for each cpu described. (Note that if CONFIG*SMP is not defined, \_no* domains are utilized and these lines will not appear in the output.)

> 其中一个是为所描述的每个 cpu 的每个域生成的。（请注意，如果未定义 CONFIG_SMP，则会使用*no*域，并且这些行不会出现在输出中。）

```
domain\<N\> \<cpumask\> 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36
```

The first field is a bit mask indicating what cpus this domain operates over.

> 第一个字段是一个位掩码，指示此域在哪个 cpu 上操作。

The next 24 are a variety of load_balance() statistics in grouped into types of idleness (idle, busy, and newly idle):

> 接下来的 24 个是各种 load_balance（）统计数据，分为空闲类型（空闲、繁忙和新空闲）：

> 1.  \# of times in this domain load_balance() was called when the cpu was idle
> 2.  \# of times in this domain load_balance() checked but found the load did not require balancing when the cpu was idle
> 3.  \# of times in this domain load_balance() tried to move one or more tasks and failed, when the cpu was idle
> 4.  sum of imbalances discovered (if any) with each call to load_balance() in this domain when the cpu was idle
> 5.  \# of times in this domain pull_task() was called when the cpu was idle
> 6.  \# of times in this domain pull_task() was called even though the target task was cache-hot when idle
> 7.  \# of times in this domain load_balance() was called but did not find a busier queue while the cpu was idle
> 8.  \# of times in this domain a busier queue was found while the cpu was idle but no busier group was found
> 9.  \# of times in this domain load_balance() was called when the cpu was busy
> 10. \# of times in this domain load_balance() checked but found the load did not require balancing when busy
> 11. \# of times in this domain load_balance() tried to move one or more tasks and failed, when the cpu was busy
> 12. sum of imbalances discovered (if any) with each call to load_balance() in this domain when the cpu was busy
> 13. \# of times in this domain pull_task() was called when busy
> 14. \# of times in this domain pull_task() was called even though the target task was cache-hot when busy
> 15. \# of times in this domain load_balance() was called but did not find a busier queue while the cpu was busy
> 16. \# of times in this domain a busier queue was found while the cpu was busy but no busier group was found
> 17. \# of times in this domain load_balance() was called when the cpu was just becoming idle
> 18. \# of times in this domain load_balance() checked but found the load did not require balancing when the cpu was just becoming idle
> 19. \# of times in this domain load_balance() tried to move one or more tasks and failed, when the cpu was just becoming idle
> 20. sum of imbalances discovered (if any) with each call to load_balance() in this domain when the cpu was just becoming idle
> 21. \# of times in this domain pull_task() was called when newly idle
> 22. \# of times in this domain pull_task() was called even though the target task was cache-hot when just becoming idle
> 23. \# of times in this domain load_balance() was called but did not find a busier queue while the cpu was just becoming idle
> 24. \# of times in this domain a busier queue was found while the cpu was just becoming idle but no busier group was found
>
> Next three are active_load_balance() statistics:
>
> 25. \# of times active_load_balance() was called
> 26. \# of times active_load_balance() tried to move a task and failed
> 27. \# of times active_load_balance() successfully moved a task
>
> Next three are sched_balance_exec() statistics:
>
> 28. sbe_cnt is not used
> 29. sbe_balanced is not used
> 30. sbe_pushed is not used
>
> Next three are sched_balance_fork() statistics:
>
> 31. sbf_cnt is not used
> 32. sbf_balanced is not used
> 33. sbf_pushed is not used
>
> Next three are try_to_wake_up() statistics:
>
> 34. \# of times in this domain try_to_wake_up() awoke a task that last ran on a different cpu in this domain
> 35. \# of times in this domain try_to_wake_up() moved a task to the waking cpu because it was cache-cold on its own cpu anyway
> 36. \# of times in this domain try_to_wake_up() started passive balancing

# /proc/\<pid\>/schedstat

schedstats also adds a new /proc/\<pid\>/schedstat file to include some of the same information on a per-process level. There are three fields in this file correlating for that process to:

> schedstat 还添加了一个新的/proc/\<pid\>/schedstat 文件，以在每个进程级别上包含一些相同的信息。此文件中有三个字段与该进程相关：

> 1.  time spent on the cpu (in nanoseconds)
> 2.  time spent waiting on a runqueue (in nanoseconds)
> 3.  \# of timeslices run on this cpu

A program could be easily written to make use of these extra fields to report on how well a particular process or set of processes is faring under the scheduler\'s policies. A simple version of such a program is available at

> 可以很容易地编写程序，利用这些额外的字段来报告特定进程或一组进程在调度器策略下的运行情况。此类程序的简单版本可在

> <http://eaglet.pdxhosts.com/rick/linux/schedstat/v12/latency.c>
