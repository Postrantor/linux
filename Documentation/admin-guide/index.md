---
tip: translate by baidu@2024-01-30 21:34:23
---
---
title: The Linux kernel user\'s and administrator\'s guide
---


The following is a collection of user-oriented documents that have been added to the kernel over time. There is, as yet, little overall order or organization here --- this material was not written to be a single, coherent document! With luck things will improve quickly over time.

> 以下是随着时间的推移添加到内核中的面向用户的文档的集合。到目前为止，这里还没有什么整体的秩序或组织——这些材料并不是为了成为一个单一、连贯的文件而写的！运气好的话，随着时间的推移，情况会很快好转。


This initial section contains overall information, including the README file describing the kernel as a whole, documentation on kernel parameters, etc.

> 这个初始部分包含总体信息，包括将内核描述为一个整体的README文件、关于内核参数的文档等。

::: {.toctree maxdepth="1"}
README kernel-parameters devices sysctl/index

abi features
:::

This section describes CPU vulnerabilities and their mitigations.

::: {.toctree maxdepth="1"}
hw-vuln/index
:::


Here is a set of documents aimed at users who are trying to track down problems and bugs in particular.

> 这里有一组文档，专门针对那些试图追踪问题和bug的用户。

::: {.toctree maxdepth="1"}

reporting-issues reporting-regressions quickly-build-trimmed-linux bug-hunting bug-bisect tainted-kernels ramoops dynamic-debug-howto init kdump/index perf/index pstore-blk

> 报告问题报告回归快速构建修剪过的linux错误搜寻错误平分受污染的内核ramoops动态调试如何初始化kdump/index-perf/index-pstore blk
:::


This is the beginning of a section with information of interest to application developers. Documents covering various aspects of the kernel ABI will be found here.

> 这是应用程序开发人员感兴趣的信息部分的开头。涵盖内核ABI各个方面的文档将在这里找到。

::: {.toctree maxdepth="1"}
sysfs-rules
:::


This is the beginning of a section with information of interest to application developers and system integrators doing analysis of the Linux kernel for safety critical applications. Documents supporting analysis of kernel interactions with applications, and key kernel subsystems expectations will be found here.

> 这是应用程序开发人员和系统集成商为安全关键应用程序分析Linux内核所感兴趣的信息的一节的开头。支持分析内核与应用程序的交互以及关键内核子系统期望的文档将在此处找到。

::: {.toctree maxdepth="1"}
workload-tracing
:::


The rest of this manual consists of various unordered guides on how to configure specific aspects of kernel behavior to your liking.

> 本手册的其余部分包括各种无序的指南，介绍如何根据您的喜好配置内核行为的特定方面。

::: {.toctree maxdepth="1"}

acpi/index aoe/index auxdisplay/index bcache binderfs binfmt-misc blockdev/index bootconfig braille-console btmrvl cgroup-v1/index cgroup-v2 cifs/index clearing-warn-once cpu-load cputopology dell_rbu device-mapper/index edid efi-stub ext4 filesystem-monitoring nfs/index gpio/index highuid hw_random initrd iostats java jfs kernel-per-CPU-kthreads laptops/index lcd-panel-cgram ldm lockup-watchdogs LSM/index md media/index mm/index module-signing mono namespaces/index numastat parport perf-security pm/index pnp rapidio ras rtc serial-console svga syscall-user-dispatch sysrq thermal/index thunderbolt ufs unicode vga-softcursor video-output xfs

> acpi/index aoe/index auxdisplay/index bcache binderfs binfmt misc blockdev/index bootconfig盲文控制台btmrvl cgroup-v1/index cgroup-v2 cifs/index clearing warn once cpu加载cputopology dell_rbu设备映射器/index edid efi stub ext4文件系统监视nfs/index gpio/index highuid hw_random initrd iostat每个cpu的java jfs内核kthreads笔记本电脑/index lcd面板cgram ldm锁定看门狗LSM/index md media/index mm/index模块签名mono-namespaces/index numastat parport perf security pm/index pnp rapidio ras rtc串行控制台svga syscall用户调度sysrq thermal/index thunderbolt ufs unicode vga软光标视频输出xfs
:::

::: only
subproject and html

# Indices

-   `genindex`{.interpreted-text role="ref"}
:::
