---
tip: translate by baidu@2024-01-13 11:30:59
title: Scheduler debugfs
---

Booting a kernel with `CONFIG_SCHED_DEBUG=y` will give access to scheduler specific debug files under `/sys/kernel/debug/sched`. Some of those files are described below.

> 使用“CONFIG_SCHED_DEBUG=y”启动内核将允许访问“/sys/kernel/DEBUG/SCHED”下的调度程序特定调试文件。其中一些文件描述如下。

# numa_balancing

[numa_balancing]{.title-ref} directory is used to hold files to control NUMA balancing feature. If the system overhead from the feature is too high then the rate the kernel samples for NUMA hinting faults may be controlled by the [scan_period_min_ms, scan_delay_ms, scan_period_max_ms, scan_size_mb]{.title-ref} files.

> [numa_blaning]｛.title-ref｝目录用于保存控制 numa 平衡功能的文件。如果来自该功能的系统开销太高，则 NUMA 提示故障的内核采样率可以由[scan_period_ms、scan_delay_ms、scan_period_max_ms、scan_size_mb]{.title-ref}文件控制。

## scan_period_min_ms, scan_delay_ms, scan_period_max_ms, scan_size_mb

Automatic NUMA balancing scans tasks address space and unmaps pages to detect if pages are properly placed or if the data should be migrated to a memory node local to where the task is running. Every "scan delay" the task scans the next "scan size" number of pages in its address space. When the end of the address space is reached the scanner restarts from the beginning.

> 自动 NUMA 平衡扫描任务地址空间并取消映射页面，以检测页面是否正确放置，或者数据是否应迁移到任务运行所在的本地内存节点。每“扫描延迟”一次，任务就会扫描其地址空间中的下一个“扫描大小”页数。当到达地址空间的末尾时，扫描仪会从头开始重新启动。

In combination, the "scan delay" and "scan size" determine the scan rate. When "scan delay" decreases, the scan rate increases. The scan delay and hence the scan rate of every task is adaptive and depends on historical behaviour. If pages are properly placed then the scan delay increases, otherwise the scan delay decreases. The "scan size" is not adaptive but the higher the "scan size", the higher the scan rate.

> 在组合中，“扫描延迟”和“扫描大小”决定了扫描速率。当“扫描延迟”减少时，扫描速率增加。每个任务的扫描延迟和扫描速率是自适应的，并且取决于历史行为。如果页面放置正确，则扫描延迟增加，否则扫描延迟减小。“扫描大小”不是自适应的，但“扫描尺寸”越高，扫描速率就越高。

Higher scan rates incur higher system overhead as page faults must be trapped and potentially data must be migrated. However, the higher the scan rate, the more quickly a tasks memory is migrated to a local node if the workload pattern changes and minimises performance impact due to remote memory accesses. These files control the thresholds for scan delays and the number of pages scanned.

> 更高的扫描速率会导致更高的系统开销，因为必须捕获页面故障并迁移潜在的数据。然而，扫描速率越高，如果工作负载模式发生变化，则任务内存迁移到本地节点的速度就越快，并将远程内存访问对性能的影响降至最低。这些文件控制扫描延迟的阈值和扫描的页数。

`scan_period_min_ms` is the minimum time in milliseconds to scan a tasks virtual memory. It effectively controls the maximum scanning rate for each task.

> `scan_period_min_ms`是扫描任务虚拟内存的最小时间。它有效地控制了每个任务的最大扫描率。

`scan_delay_ms` is the starting "scan delay" used for a task when it initially forks.

> `scan_delay_ms`是最初叉时用于任务的启动“扫描延迟”。

`scan_period_max_ms` is the maximum time in milliseconds to scan a tasks virtual memory. It effectively controls the minimum scanning rate for each task.

> `scan_period_max_ms`是扫描任务虚拟内存的最大时间。它有效地控制着每个任务的最小扫描率。

`scan_size_mb` is how many megabytes worth of pages are scanned for a given scan.

> `scan_size_mb`是在给定的扫描中扫描了多少兆字节的页面。
