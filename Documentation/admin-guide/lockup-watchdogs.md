---
tip: translate by baidu@2024-01-30 21:35:30
---
---
title: Softlockup detector and hardlockup detector (aka nmi_watchdog)
---


The Linux kernel can act as a watchdog to detect both soft and hard lockups.

> Linux内核可以充当看门狗来检测软锁定和硬锁定。


A \'softlockup\' is defined as a bug that causes the kernel to loop in kernel mode for more than 20 seconds (see \"Implementation\" below for details), without giving other tasks a chance to run. The current stack trace is displayed upon detection and, by default, the system will stay locked up. Alternatively, the kernel can be configured to panic; a sysctl, \"kernel.softlockup_panic\", a kernel parameter, \"softlockup_panic\" (see \"Documentation/admin-guide/kernel-parameters.rst\" for details), and a compile option, \"BOOTPARAM_SOFTLOCKUP_PANIC\", are provided for this.

> “软件锁定”被定义为一种错误，它会导致内核在内核模式下循环超过20秒（有关详细信息，请参阅下面的“实现”），而不给其他任务运行的机会。当前堆栈跟踪在检测时显示，默认情况下，系统将保持锁定状态。或者，内核可以配置为死机；为此提供了sysctl、“kernel.softlockup_panic”、内核参数“softlockup_panis”（有关详细信息，请参阅“Documentation/admin guide/kernel parameters.rst”）和编译选项“BOOTPARAM_softlockup_panic\”。


A \'hardlockup\' is defined as a bug that causes the CPU to loop in kernel mode for more than 10 seconds (see \"Implementation\" below for details), without letting other interrupts have a chance to run. Similarly to the softlockup case, the current stack trace is displayed upon detection and the system will stay locked up unless the default behavior is changed, which can be done through a sysctl, \'hardlockup_panic\', a compile time knob, \"BOOTPARAM_HARDLOCKUP_PANIC\", and a kernel parameter, \"nmi_watchdog\" (see \"Documentation/admin-guide/kernel-parameters.rst\" for details).

> “硬件锁定”被定义为一种错误，它导致CPU在内核模式下循环超过10秒（有关详细信息，请参阅下面的“实现”），而不让其他中断有机会运行。与软锁定情况类似，检测时会显示当前堆栈跟踪，除非默认行为发生更改，否则系统将保持锁定状态。默认行为可以通过sysctl“hardlockup_panic”、编译时旋钮“BOOTPARAM_HARDLLOCKUP_panic\”和内核参数“nmi_doggdog”来完成（有关详细信息，请参阅“Documentation/admin guide/kernel parameters.rst”）。


The panic option can be used in combination with panic_timeout (this timeout is set through the confusingly named \"kernel.panic\" sysctl), to cause the system to reboot automatically after a specified amount of time.

> panic选项可以与panic_timeout组合使用（此超时是通过名称令人困惑的“kernel.parance\”sysctl设置的），以使系统在指定的时间后自动重新启动。

# Implementation


The soft and hard lockup detectors are built on top of the hrtimer and perf subsystems, respectively. A direct consequence of this is that, in principle, they should work in any architecture where these subsystems are present.

> 软锁定检测器和硬锁定检测器分别构建在hrtimer和perf子系统的顶部。这样做的直接后果是，原则上，它们应该在存在这些子系统的任何架构中工作。


A periodic hrtimer runs to generate interrupts and kick the watchdog job. An NMI perf event is generated every \"watchdog_thresh\" (compile-time initialized to 10 and configurable through sysctl of the same name) seconds to check for hardlockups. If any CPU in the system does not receive any hrtimer interrupt during that time the \'hardlockup detector\' (the handler for the NMI perf event) will generate a kernel warning or call panic, depending on the configuration.

> 一个周期性的hr计时器运行以生成中断并启动看门狗作业。每“watchdog_thresh\”（编译时间初始化为10，可通过同名sysctl配置）秒生成一个NMI perf事件，以检查硬锁定。如果系统中的任何CPU在此期间没有接收到任何hrtimer中断，“硬件锁定检测器”（NMI perf事件的处理程序）将根据配置生成内核警告或调用死机。


The watchdog job runs in a stop scheduling thread that updates a timestamp every time it is scheduled. If that timestamp is not updated for 2\*watchdog_thresh seconds (the softlockup threshold) the \'softlockup detector\' (coded inside the hrtimer callback function) will dump useful debug information to the system log, after which it will call panic if it was instructed to do so or resume execution of other kernel code.

> 看门狗作业在停止调度线程中运行，该线程每次调度它时都会更新时间戳。如果该时间戳在2 \*watchdog_thresh秒（软锁定阈值）内未更新，“软锁定检测器”（编码在hrtimer回调函数内）将向系统日志转储有用的调试信息，之后，如果指示它这样做或恢复执行其他内核代码，它将调用panic。


The period of the hrtimer is 2\*watchdog_thresh/5, which means it has two or three chances to generate an interrupt before the hardlockup detector kicks in.

> hrtimer的周期为2\*watchdog_thresh/5，这意味着在硬锁定检测器启动之前，它有两到三次机会生成中断。


As explained above, a kernel knob is provided that allows administrators to configure the period of the hrtimer and the perf event. The right value for a particular environment is a trade-off between fast response to lockups and detection overhead.

> 如上所述，提供了一个内核旋钮，允许管理员配置hrtimer和perf事件的周期。特定环境的正确值是在对锁定的快速响应和检测开销之间进行权衡。


By default, the watchdog runs on all online cores. However, on a kernel configured with NO_HZ_FULL, by default the watchdog runs only on the housekeeping cores, not the cores specified in the \"nohz_full\" boot argument. If we allowed the watchdog to run by default on the \"nohz_full\" cores, we would have to run timer ticks to activate the scheduler, which would prevent the \"nohz_full\" functionality from protecting the user code on those cores from the kernel. Of course, disabling it by default on the nohz_full cores means that when those cores do enter the kernel, by default we will not be able to detect if they lock up. However, allowing the watchdog to continue to run on the housekeeping (non-tickless) cores means that we will continue to detect lockups properly on those cores.

> 默认情况下，看门狗在所有在线内核上运行。但是，在配置了NO_HZ_FULL的内核上，默认情况下，看门狗仅在内务核心上运行，而不是在“nohz_FULL”引导参数中指定的核心上运行。如果我们允许看门狗在默认情况下在“nohz_full”内核上运行，我们将不得不运行计时器来激活调度程序，这将阻止“nohz _full“功能从内核保护这些内核上的用户代码。当然，在nohz_full内核上默认禁用它意味着当这些内核进入内核时，默认情况下我们将无法检测它们是否锁定。然而，允许看门狗继续在内务（非挠痒）内核上运行意味着我们将继续在这些内核上正确检测锁定。


In either case, the set of cores excluded from running the watchdog may be adjusted via the kernel.watchdog_cpumask sysctl. For nohz_full cores, this may be useful for debugging a case where the kernel seems to be hanging on the nohz_full cores.

> 在任何一种情况下，都可以通过kernel.watchdog_cpumpask sysctl调整被排除在运行看门狗之外的内核集。对于nohz_full内核，这可能有助于调试内核似乎挂在nohz_full内核上的情况。
