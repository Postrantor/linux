---
tip: translate by baidu@2024-01-30 21:40:51
---
---
author: |
  -   Shuah Khan \<<skhan@linuxfoundation.org>\>
  -   Shefali Sharma \<<sshefali021@gmail.com>\>
maintained-by: |
  Shuah Khan \<<skhan@linuxfoundation.org>\>
title: Discovering Linux kernel subsystems used by a workload
---

# Key Points

> -   Understanding system resources necessary to build and run a workload is important.
> -   Linux tracing and strace can be used to discover the system resources in use by a workload. The completeness of the system usage information depends on the completeness of coverage of a workload.
> -   Performance and security of the operating system can be analyzed with the help of tools such as: [perf](https://man7.org/linux/man-pages/man1/perf.1.html), [stress-ng](https://www.mankier.com/1/stress-ng), [paxtest](https://github.com/opntr/paxtest-freebsd).
> -   Once we discover and understand the workload needs, we can focus on them to avoid regressions and use it to evaluate safety considerations.

# Methodology


[strace](https://man7.org/linux/man-pages/man1/strace.1.html) is a diagnostic, instructional, and debugging tool and can be used to discover the system resources in use by a workload. Once we discover and understand the workload needs, we can focus on them to avoid regressions and use it to evaluate safety considerations. We use strace tool to trace workloads.

> [strace](https://man7.org/linux/man-pages/man1/strace.1.html)是一种诊断、指导和调试工具，可用于发现工作负载正在使用的系统资源。一旦我们发现并了解了工作量需求，我们就可以专注于它们以避免倒退，并将其用于评估安全考虑因素。我们使用strace工具来跟踪工作负载。


This method of tracing using strace tells us the system calls invoked by the workload and doesn\'t include all the system calls that can be invoked by it. In addition, this tracing method tells us just the code paths within these system calls that are invoked. As an example, if a workload opens a file and reads from it successfully, then the success path is the one that is traced. Any error paths in that system call will not be traced. If there is a workload that provides full coverage of a workload then the method outlined here will trace and find all possible code paths. The completeness of the system usage information depends on the completeness of coverage of a workload.

> 这种使用strace跟踪的方法会告诉我们工作负载调用的系统调用，但不包括它可以调用的所有系统调用。此外，这种跟踪方法只会告诉我们这些系统调用中被调用的代码路径。例如，如果工作负载打开一个文件并从中成功读取，那么成功路径就是被跟踪的路径。不会跟踪该系统调用中的任何错误路径。如果有一个工作负载可以提供工作负载的全部覆盖范围，那么这里概述的方法将跟踪并找到所有可能的代码路径。系统使用信息的完整性取决于工作负载覆盖范围的完整性。


The goal is tracing a workload on a system running a default kernel without requiring custom kernel installs.

> 目标是在运行默认内核的系统上跟踪工作负载，而不需要自定义内核安装。

# How do we gather fine-grained system information?


strace tool can be used to trace system calls made by a process and signals it receives. System calls are the fundamental interface between an application and the operating system kernel. They enable a program to request services from the kernel. For instance, the open() system call in Linux is used to provide access to a file in the file system. strace enables us to track all the system calls made by an application. It lists all the system calls made by a process and their resulting output.

> strace工具可用于跟踪进程进行的系统调用及其接收到的信号。系统调用是应用程序和操作系统内核之间的基本接口。它们使程序能够从内核请求服务。例如，Linux中的open（）系统调用用于提供对文件系统中文件的访问。strace使我们能够跟踪应用程序进行的所有系统调用。它列出了进程进行的所有系统调用及其结果输出。


You can generate profiling data combining strace and perf record tools to record the events and information associated with a process. This provides insight into the process. \"perf annotate\" tool generates the statistics of each instruction of the program. This document goes over the details of how to gather fine-grained information on a workload\'s usage of system resources.

> 您可以结合strace和perf记录工具生成分析数据，以记录与流程相关的事件和信息。这提供了对过程的深入了解\“perf-annotation\”工具生成程序的每条指令的统计信息。本文档详细介绍了如何收集有关工作负载使用系统资源的细粒度信息。


We used strace to trace the perf, stress-ng, paxtest workloads to illustrate our methodology to discover resources used by a workload. This process can be applied to trace other workloads.

> 我们使用strace来跟踪perf、stress ng、paxtest工作负载，以说明我们发现工作负载使用的资源的方法。此过程可用于跟踪其他工作负载。

# Getting the system ready for tracing


Before we can get started we will show you how to get your system ready. We assume that you have a Linux distribution running on a physical system or a virtual machine. Most distributions will include strace command. Let's install other tools that aren't usually included to build Linux kernel. Please note that the following works on Debian based distributions. You might have to find equivalent packages on other Linux distributions.

> 在我们开始之前，我们将向您展示如何准备好您的系统。我们假设您有一个在物理系统或虚拟机上运行的Linux发行版。大多数发行版将包括strace命令。让我们安装构建Linux内核通常不包含的其他工具。请注意，以下内容适用于基于Debian的发行版。您可能必须在其他Linux发行版上找到等效的软件包。


Install tools to build Linux kernel and tools in kernel repository. scripts/ver_linux is a good way to check if your system already has the necessary tools:

> 安装构建Linux内核的工具和内核存储库中的工具。scripts/ver_linux是检查系统是否已经拥有必要工具的好方法：

    sudo apt-get build-essentials flex bison yacc
    sudo apt install libelf-dev systemtap-sdt-dev libaudit-dev libslang2-dev libperl-dev libdw-dev

cscope is a good tool to browse kernel sources. Let\'s install it now:

    sudo apt-get install cscope

Install stress-ng and paxtest:

    apt-get install stress-ng
    apt-get install paxtest

# Workload overview


As mentioned earlier, we used strace to trace perf bench, stress-ng and paxtest workloads to show how to analyze a workload and identify Linux subsystems used by these workloads. Let\'s start with an overview of these three workloads to get a better understanding of what they do and how to use them.

> 如前所述，我们使用strace来跟踪perf-bbench、stress ng和paxtest工作负载，以展示如何分析工作负载并识别这些工作负载使用的Linux子系统。让我们从这三种工作负载的概述开始，以更好地了解它们的作用以及如何使用它们。

## perf bench (all) workload


The perf bench command contains multiple multi-threaded microkernel benchmarks for executing different subsystems in the Linux kernel and system calls. This allows us to easily measure the impact of changes, which can help mitigate performance regressions. It also acts as a common benchmarking framework, enabling developers to easily create test cases, integrate transparently, and use performance-rich tooling subsystems.

> perf-beek命令包含多个多线程微内核基准测试，用于执行Linux内核中的不同子系统和系统调用。这使我们能够轻松地衡量更改的影响，这有助于缓解性能倒退。它还充当了一个通用的基准测试框架，使开发人员能够轻松地创建测试用例，透明地集成，并使用性能丰富的工具子系统。

## Stress-ng netdev stressor workload


stress-ng is used for performing stress testing on the kernel. It allows you to exercise various physical subsystems of the computer, as well as interfaces of the OS kernel, using \"stressor-s\". They are available for CPU, CPU cache, devices, I/O, interrupts, file system, memory, network, operating system, pipelines, schedulers, and virtual machines. Please refer to the [stress-ng man-page](https://www.mankier.com/1/stress-ng) to find the description of all the available stressor-s. The netdev stressor starts specified number (N) of workers that exercise various netdevice ioctl commands across all the available network devices.

> stress ng用于对内核执行压力测试。它允许您使用\“stress-s”来锻炼计算机的各种物理子系统以及操作系统内核的接口。它们可用于CPU、CPU缓存、设备、I/O、中断、文件系统、内存、网络、操作系统、管道、调度器和虚拟机。请参阅[强调ng手册页](https://www.mankier.com/1/stress-ng)找到所有可用压力的描述。netdev压力源启动指定数量（N）的工作者，他们在所有可用的网络设备上执行各种netdevice ioctl命令。

## paxtest kiddie workload


paxtest is a program that tests buffer overflows in the kernel. It tests kernel enforcements over memory usage. Generally, execution in some memory segments makes buffer overflows possible. It runs a set of programs that attempt to subvert memory usage. It is used as a regression test suite for PaX, but might be useful to test other memory protection patches for the kernel. We used paxtest kiddie mode which looks for simple vulnerabilities.

> paxtest是一个测试内核中缓冲区溢出的程序。它测试内核对内存使用的强制执行。通常，在某些内存段中执行会导致缓冲区溢出。它运行一组试图破坏内存使用的程序。它被用作PaX的回归测试套件，但可能有助于测试内核的其他内存保护补丁。我们使用了paxtest kiddie模式，该模式查找简单的漏洞。

# What is strace and how do we use it?


As mentioned earlier, strace which is a useful diagnostic, instructional, and debugging tool and can be used to discover the system resources in use by a workload. It can be used:

> 如前所述，strace是一种有用的诊断、指导和调试工具，可用于发现工作负载正在使用的系统资源。它可以用于：

> -   To see how a process interacts with the kernel.
> -   To see why a process is failing or hanging.
> -   For reverse engineering a process.
> -   To find the files on which a program depends.
> -   For analyzing the performance of an application.
> -   For troubleshooting various problems related to the operating system.


In addition, strace can generate run-time statistics on times, calls, and errors for each system call and report a summary when program exits, suppressing the regular output. This attempts to show system time (CPU time spent running in the kernel) independent of wall clock time. We plan to use these features to get information on workload system usage.

> 此外，strace可以为每个系统调用生成时间、调用和错误的运行时统计信息，并在程序退出时报告摘要，从而抑制常规输出。这试图显示独立于墙上时钟时间的系统时间（在内核中运行的CPU时间）。我们计划使用这些功能来获取有关工作负载系统使用情况的信息。


strace command supports basic, verbose, and stats modes. strace command when run in verbose mode gives more detailed information about the system calls invoked by a process.

> strace命令支持基本、详细和统计模式。strace命令在详细模式下运行时，会提供有关进程调用的系统调用的更详细信息。


Running strace -c generates a report of the percentage of time spent in each system call, the total time in seconds, the microseconds per call, the total number of calls, the count of each system call that has failed with an error and the type of system call made.

> 运行strace-c会生成一个报告，其中包括每次系统调用所花费的时间百分比、总时间（以秒为单位）、每次调用的微秒数、调用总数、每次系统调用因错误而失败的次数以及所进行的系统调用类型。

> -   Usage: strace \<command we want to trace\>
> -   Verbose mode usage: strace -v \<command\>
> -   Gather statistics: strace -c \<command\>


We used the "-c" option to gather fine-grained run-time statistics in use by three workloads we have chose for this analysis.

> 我们使用“-c”选项来收集我们为该分析选择的三个工作负载所使用的细粒度运行时统计信息。

> -   perf
> -   stress-ng
> -   paxtest

# What is cscope and how do we use it?


Now let's look at [cscope](https://cscope.sourceforge.net/), a command line tool for browsing C, C++ or Java code-bases. We can use it to find all the references to a symbol, global definitions, functions called by a function, functions calling a function, text strings, regular expression patterns, files including a file.

> 现在让我们来看[ccope](https://cscope.sourceforge.net/)，一个用于浏览C、C++或Java代码库的命令行工具。我们可以使用它来查找对符号的所有引用、全局定义、函数调用的函数、调用函数的函数、文本字符串、正则表达式模式、包括文件在内的文件。


We can use cscope to find which system call belongs to which subsystem. This way we can find the kernel subsystems used by a process when it is executed.

> 我们可以使用cscope来查找哪个系统调用属于哪个子系统。通过这种方式，我们可以找到进程在执行时使用的内核子系统。

Let's checkout the latest Linux repository and build cscope database:

    git clone git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git linux
    cd linux
    cscope -R -p10  # builds cscope.out database before starting browse session
    cscope -d -p10  # starts browse session on cscope.out database


Note: Run \"cscope -R -p10\" to build the database and c\"scope -d -p10\" to enter into the browsing session. cscope by default cscope.out database. To get out of this mode press ctrl+d. -p option is used to specify the number of file path components to display. -p10 is optimal for browsing kernel sources.

> 注意：运行“cscope-R-p10\”构建数据库，运行“scope-d-p10\“进入浏览会话。cscope默认情况下为cscope.out数据库。要退出此模式，请按ctrl+d-p选项用于指定要显示的文件路径组件的数量-p10是浏览内核源代码的最佳选择。

# What is perf and how do we use it?


Perf is an analysis tool based on Linux 2.6+ systems, which abstracts the CPU hardware difference in performance measurement in Linux, and provides a simple command line interface. Perf is based on the perf_events interface exported by the kernel. It is very useful for profiling the system and finding performance bottlenecks in an application.

> Perf是一个基于Linux 2.6+系统的分析工具，它抽象了Linux中CPU硬件性能测量的差异，并提供了一个简单的命令行接口。Perf基于内核导出的Perf_events接口。它对于分析系统和查找应用程序中的性能瓶颈非常有用。


If you haven\'t already checked out the Linux mainline repository, you can do so and then build kernel and perf tool:

> 如果您还没有检查出Linux主线存储库，您可以这样做，然后构建内核和性能工具：

    git clone git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git linux
    cd linux
    make -j3 all
    cd tools/perf
    make


Note: The perf command can be built without building the kernel in the repository and can be run on older kernels. However matching the kernel and perf revisions gives more accurate information on the subsystem usage.

> 注意：perf命令可以在不在存储库中构建内核的情况下构建，并且可以在较旧的内核上运行。然而，匹配内核和性能修订版可以提供关于子系统使用情况的更准确信息。


We used \"perf stat\" and \"perf bench\" options. For a detailed information on the perf tool, run \"perf -h\".

> 我们使用了“perf-stat”和“perf-bbench”选项。有关perf工具的详细信息，请运行“perf-h”。

## perf stat


The perf stat command generates a report of various hardware and software events. It does so with the help of hardware counter registers found in modern CPUs that keep the count of these activities. \"perf stat cal\" shows stats for cal command.

> perf-stat命令生成各种硬件和软件事件的报告。它是在现代CPU中的硬件计数器寄存器的帮助下实现的，这些寄存器可以记录这些活动的计数\“perf-stat cal\”显示cal命令的统计信息。

## Perf bench


The perf bench command contains multiple multi-threaded microkernel benchmarks for executing different subsystems in the Linux kernel and system calls. This allows us to easily measure the impact of changes, which can help mitigate performance regressions. It also acts as a common benchmarking framework, enabling developers to easily create test cases, integrate transparently, and use performance-rich tooling.

> perf-beek命令包含多个多线程微内核基准测试，用于执行Linux内核中的不同子系统和系统调用。这使我们能够轻松地衡量更改的影响，这有助于缓解性能倒退。它还充当了一个通用的基准测试框架，使开发人员能够轻松地创建测试用例，透明地集成，并使用性能丰富的工具。

\"perf bench all\" command runs the following benchmarks:

> -   sched/messaging
> -   sched/pipe
> -   syscall/basic
> -   mem/memcpy
> -   mem/memset

# What is stress-ng and how do we use it?


As mentioned earlier, stress-ng is used for performing stress testing on the kernel. It allows you to exercise various physical subsystems of the computer, as well as interfaces of the OS kernel, using stressor-s. They are available for CPU, CPU cache, devices, I/O, interrupts, file system, memory, network, operating system, pipelines, schedulers, and virtual machines.

> 如前所述，stress ng用于对内核执行压力测试。它允许您使用stress-s来锻炼计算机的各种物理子系统以及操作系统内核的接口。它们可用于CPU、CPU缓存、设备、I/O、中断、文件系统、内存、网络、操作系统、管道、调度器和虚拟机。


The netdev stressor starts N workers that exercise various netdevice ioctl commands across all the available network devices. The following ioctls are exercised:

> netdev压力源启动N个工作程序，在所有可用的网络设备上执行各种netdevice ioctl命令。行使以下ioctl：

> -   SIOCGIFCONF, SIOCGIFINDEX, SIOCGIFNAME, SIOCGIFFLAGS
> -   SIOCGIFADDR, SIOCGIFNETMASK, SIOCGIFMETRIC, SIOCGIFMTU
> -   SIOCGIFHWADDR, SIOCGIFMAP, SIOCGIFTXQLEN

The following command runs the stressor:

    stress-ng --netdev 1 -t 60 --metrics command.


We can use the perf record command to record the events and information associated with a process. This command records the profiling data in the perf.data file in the same directory.

> 我们可以使用perfrecord命令来记录与进程相关联的事件和信息。此命令将分析数据记录在同一目录的perf.data文件中。


Using the following commands you can record the events associated with the netdev stressor, view the generated report perf.data and annotate the to view the statistics of each instruction of the program:

> 使用以下命令，您可以记录与netdev压力源相关的事件，查看生成的报告perf.data，并对进行注释以查看程序的每条指令的统计信息：

    perf record stress-ng --netdev 1 -t 60 --metrics command.
    perf report
    perf annotate

# What is paxtest and how do we use it?


paxtest is a program that tests buffer overflows in the kernel. It tests kernel enforcements over memory usage. Generally, execution in some memory segments makes buffer overflows possible. It runs a set of programs that attempt to subvert memory usage. It is used as a regression test suite for PaX, and will be useful to test other memory protection patches for the kernel.

> paxtest是一个测试内核中缓冲区溢出的程序。它测试内核对内存使用的强制执行。通常，在某些内存段中执行会导致缓冲区溢出。它运行一组试图破坏内存使用的程序。它被用作PaX的回归测试套件，用于测试内核的其他内存保护补丁。


paxtest provides kiddie and blackhat modes. The paxtest kiddie mode runs in normal mode, whereas the blackhat mode tries to get around the protection of the kernel testing for vulnerabilities. We focus on the kiddie mode here and combine \"paxtest kiddie\" run with \"perf record\" to collect CPU stack traces for the paxtest kiddie run to see which function is calling other functions in the performance profile. Then the \"dwarf\" (DWARF\'s Call Frame Information) mode can be used to unwind the stack.

> paxtest提供kiddie和blackhat模式。paxtest kiddie模式在正常模式下运行，而blackhat模式则试图绕过内核测试的漏洞保护。我们在这里关注kiddie模式，并将“paxtest kiddie”run与“perf record”结合起来，为paxtest kiddie run收集CPU堆栈跟踪，以查看哪个函数正在调用性能配置文件中的其他函数。然后可以使用“dwarf”（dwarf的调用帧信息）模式来展开堆栈。


The following command can be used to view resulting report in call-graph format:

> 以下命令可用于以调用图格式查看结果报告：

    perf record --call-graph dwarf paxtest kiddie
    perf report --stdio

# Tracing workloads

Now that we understand the workloads, let\'s start tracing them.

## Tracing perf bench all workload

Run the following command to trace perf bench all workload:

    strace -c perf bench all

**System Calls made by the workload**


The below table shows the system calls invoked by the workload, number of times each system call is invoked, and the corresponding Linux subsystem.

> 下表显示了工作负载调用的系统调用、每个系统调用的调用次数以及相应的Linux子系统。

  ---------------------------------------------------------------------------
  System Call         \# calls    Linux Subsystem   System Call (API)
  ------------------- ----------- ----------------- -------------------------
  getppid             10000001    Process Mgmt      sys_getpid()

  clone               1077        Process Mgmt.     sys_clone()

  prctl               23          Process Mgmt.     sys_prctl()

  prlimit64           7           Process Mgmt.     sys_prlimit64()

  getpid              10          Process Mgmt.     sys_getpid()

  uname               3           Process Mgmt.     sys_uname()

  sysinfo             1           Process Mgmt.     sys_sysinfo()

  getuid              1           Process Mgmt.     sys_getuid()

  getgid              1           Process Mgmt.     sys_getgid()

  geteuid             1           Process Mgmt.     sys_geteuid()

  getegid             1           Process Mgmt.     sys_getegid

  close               49951       Filesystem        sys_close()

  pipe                604         Filesystem        sys_pipe()

  openat              48560       Filesystem        sys_opennat()

  fstat               8338        Filesystem        sys_fstat()

  stat                1573        Filesystem        sys_stat()

  pread64             9646        Filesystem        sys_pread64()

  getdents64          1873        Filesystem        sys_getdents64()

  access              3           Filesystem        sys_access()

  lstat               1880        Filesystem        sys_lstat()

  lseek               6           Filesystem        sys_lseek()

  ioctl               3           Filesystem        sys_ioctl()

  dup2                1           Filesystem        sys_dup2()

  execve              2           Filesystem        sys_execve()

  fcntl               8779        Filesystem        sys_fcntl()

  statfs              1           Filesystem        sys_statfs()

  epoll_create        2           Filesystem        sys_epoll_create()

  epoll_ctl           64          Filesystem        sys_epoll_ctl()

  newfstatat          8318        Filesystem        sys_newfstatat()

  eventfd2            192         Filesystem        sys_eventfd2()

  mmap                243         Memory Mgmt.      sys_mmap()

  mprotect            32          Memory Mgmt.      sys_mprotect()

  brk                 21          Memory Mgmt.      sys_brk()

  munmap              128         Memory Mgmt.      sys_munmap()

  set_mempolicy       156         Memory Mgmt.      sys_set_mempolicy()

  set_tid_address     1           Process Mgmt.     sys_set_tid_address()

  set_robust_list     1           Futex             sys_set_robust_list()

  futex               341         Futex             sys_futex()


  sched_getaffinity   79          Scheduler         sys_sched_getaffinity()

> sched_getaffinity 79调度程序sys_sched_getaffinity（）


  sched_setaffinity   223         Scheduler         sys_sched_setaffinity()

> sched_set-affinity 223计划程序sys_sched_stafffinity（）

  socketpair          202         Network           sys_socketpair()

  rt_sigprocmask      21          Signal            sys_rt_sigprocmask()

  rt_sigaction        36          Signal            sys_rt_sigaction()

  rt_sigreturn        2           Signal            sys_rt_sigreturn()

  wait4               889         Time              sys_wait4()

  clock_nanosleep     37          Time              sys_clock_nanosleep()

  capget              4           Capability        sys_capget()
  ---------------------------------------------------------------------------

## Tracing stress-ng netdev stressor workload

Run the following command to trace stress-ng netdev stressor workload:

    strace -c  stress-ng --netdev 1 -t 60 --metrics

**System Calls made by the workload**


The below table shows the system calls invoked by the workload, number of times each system call is invoked, and the corresponding Linux subsystem.

> 下表显示了工作负载调用的系统调用、每个系统调用的调用次数以及相应的Linux子系统。

  -------------------------------------------------------------------------
  System Call        \# calls    Linux Subsystem   System Call (API)
  ------------------ ----------- ----------------- ------------------------
  openat             74          Filesystem        sys_openat()

  close              75          Filesystem        sys_close()

  read               58          Filesystem        sys_read()

  fstat              20          Filesystem        sys_fstat()

  flock              10          Filesystem        sys_flock()

  write              7           Filesystem        sys_write()

  getdents64         8           Filesystem        sys_getdents64()

  pread64            8           Filesystem        sys_pread64()

  lseek              1           Filesystem        sys_lseek()

  access             2           Filesystem        sys_access()

  getcwd             1           Filesystem        sys_getcwd()

  execve             1           Filesystem        sys_execve()

  mmap               61          Memory Mgmt.      sys_mmap()

  munmap             3           Memory Mgmt.      sys_munmap()

  mprotect           20          Memory Mgmt.      sys_mprotect()

  mlock              2           Memory Mgmt.      sys_mlock()

  brk                3           Memory Mgmt.      sys_brk()

  rt_sigaction       21          Signal            sys_rt_sigaction()

  rt_sigprocmask     1           Signal            sys_rt_sigprocmask()

  sigaltstack        1           Signal            sys_sigaltstack()

  rt_sigreturn       1           Signal            sys_rt_sigreturn()

  getpid             8           Process Mgmt.     sys_getpid()

  prlimit64          5           Process Mgmt.     sys_prlimit64()

  arch_prctl         2           Process Mgmt.     sys_arch_prctl()

  sysinfo            2           Process Mgmt.     sys_sysinfo()

  getuid             2           Process Mgmt.     sys_getuid()

  uname              1           Process Mgmt.     sys_uname()

  setpgid            1           Process Mgmt.     sys_setpgid()

  getrusage          1           Process Mgmt.     sys_getrusage()

  geteuid            1           Process Mgmt.     sys_geteuid()

  getppid            1           Process Mgmt.     sys_getppid()

  sendto             3           Network           sys_sendto()

  connect            1           Network           sys_connect()

  socket             1           Network           sys_socket()

  clone              1           Process Mgmt.     sys_clone()

  set_tid_address    1           Process Mgmt.     sys_set_tid_address()

  wait4              2           Time              sys_wait4()

  alarm              1           Time              sys_alarm()

  set_robust_list    1           Futex             sys_set_robust_list()
  -------------------------------------------------------------------------

## Tracing paxtest kiddie workload

Run the following command to trace paxtest kiddie workload:

    strace -c paxtest kiddie

**System Calls made by the workload**


The below table shows the system calls invoked by the workload, number of times each system call is invoked, and the corresponding Linux subsystem.

> 下表显示了工作负载调用的系统调用、每个系统调用的调用次数以及相应的Linux子系统。

  ------------------------------------------------------------------------
  System Call         \# calls    Linux Subsystem   System Call (API)
  ------------------- ----------- ----------------- ----------------------
  read                3           Filesystem        sys_read()

  write               11          Filesystem        sys_write()

  close               41          Filesystem        sys_close()

  stat                24          Filesystem        sys_stat()

  fstat               2           Filesystem        sys_fstat()

  pread64             6           Filesystem        sys_pread64()

  access              1           Filesystem        sys_access()

  pipe                1           Filesystem        sys_pipe()

  dup2                24          Filesystem        sys_dup2()

  execve              1           Filesystem        sys_execve()

  fcntl               26          Filesystem        sys_fcntl()

  openat              14          Filesystem        sys_openat()

  rt_sigaction        7           Signal            sys_rt_sigaction()

  rt_sigreturn        38          Signal            sys_rt_sigreturn()

  clone               38          Process Mgmt.     sys_clone()

  wait4               44          Time              sys_wait4()

  mmap                7           Memory Mgmt.      sys_mmap()

  mprotect            3           Memory Mgmt.      sys_mprotect()

  munmap              1           Memory Mgmt.      sys_munmap()

  brk                 3           Memory Mgmt.      sys_brk()

  getpid              1           Process Mgmt.     sys_getpid()

  getuid              1           Process Mgmt.     sys_getuid()

  getgid              1           Process Mgmt.     sys_getgid()

  geteuid             2           Process Mgmt.     sys_geteuid()

  getegid             1           Process Mgmt.     sys_getegid()

  getppid             1           Process Mgmt.     sys_getppid()

  arch_prctl          2           Process Mgmt.     sys_arch_prctl()
  ------------------------------------------------------------------------

# Conclusion


This document is intended to be used as a guide on how to gather fine-grained information on the resources in use by workloads using strace.

> 本文档旨在指导如何使用strace收集有关工作负载使用的资源的细粒度信息。

# References

> -   [Discovery Linux Kernel Subsystems used by OpenAPS](https://elisa.tech/blog/2022/02/02/discovery-linux-kernel-subsystems-used-by-openaps)
> -   [ELISA-White-Papers-Discovering Linux kernel subsystems used by a workload](https://github.com/elisa-tech/ELISA-White-Papers/blob/master/Processes/Discovering_Linux_kernel_subsystems_used_by_a_workload.md)
> -   [strace](https://man7.org/linux/man-pages/man1/strace.1.html)
> -   [perf](https://man7.org/linux/man-pages/man1/perf.1.html)
> -   [paxtest README](https://github.com/opntr/paxtest-freebsd/blob/hardenedbsd/0.9.14-hbsd/README)
> -   [stress-ng](https://www.mankier.com/1/stress-ng)
> -   [Monitoring and managing system status and performance](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/monitoring_and_managing_system_status_and_performance/index)
