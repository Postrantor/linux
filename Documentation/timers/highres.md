---
tip: translate by baidu@2024-05-25 19:54:10
title: High resolution timers and dynamic ticks design notes
---

Further information can be found in the paper of the OLS 2006 talk \"hrtimers and beyond\". The paper is part of the OLS 2006 Proceedings Volume 1, which can be found on the OLS website: [pdf](https://www.kernel.org/doc/ols/2006/ols2006v1-pages-333-346.pdf)

> 更多信息可在 OLS 2006 talk \"hrtimers 及以后\" 的论文中找到。该论文是 OLS 2006 年论文集第 1 卷的一部分，可在 OLS 网站上找到：[pdf](https://www.kernel.org/doc/ols/2006/ols2006v1-pages-333-346.pdf)

The slides to this talk are available from: [ols2006-hrtimers-slides](http://www.cs.columbia.edu/~nahum/w6998/papers/ols2006-hrtimers-slides.pdf)

> 本演讲的幻灯片可从以下位置获取：[hrtimers](http://www.cs.columbia.edu/~nahum/w6998/papers/ols2006-hrtimers-slides.pdf)

The slides contain five figures (pages 2, 15, 18, 20, 22), which illustrate the changes in the time(r) related Linux subsystems. Figure #1 (p. 2) shows the design of the Linux time(r) system before hrtimers and other building blocks got merged into mainline.

> 幻灯片包含五个图(第 2、15、18、20、22 页)，它们说明了与时间(r)相关的 Linux 子系统的变化。图#1(第 2 页)显示了在 hrtimers 和其他构建块被合并到主线中之前 Linux 时间(r)系统的设计。

Note: the paper and the slides are talking about \"clock event source\", while we switched to the name \"clock event devices\" in meantime.

> 注意：论文和幻灯片都在谈论"时钟事件源"，同时我们切换到了"时钟事件设备"的名称。

The design contains the following basic building blocks:

- hrtimer base infrastructure
- timeofday and clock source management
- clock event management
- high resolution timer functionality
- dynamic ticks

# hrtimer base infrastructure

The hrtimer base infrastructure was merged into the 2.6.16 kernel. Details of the base implementation are covered in Documentation/timers/hrtimers.rst. See also figure #2 (OLS slides p. 15)

> hrtimer 基础架构已合并到 2.6.16 内核中。有关基本实现的详细信息，请参阅 Documentation/timers/hrtimers.rst。另见图 2(OLS 幻灯片第 15 页)

The main differences to the timer wheel, which holds the armed timer_list type timers are:

> 与容纳武装 timer_list 类型计时器的计时器轮的主要区别在于：

> - time ordered enqueueing into a rb-tree
> - independent of ticks (the processing is based on nanoseconds)

# timeofday and clock source management

John Stultz\'s Generic Time Of Day (GTOD) framework moves a large portion of code out of the architecture-specific areas into a generic management framework, as illustrated in figure #3 (OLS slides p. 18). The architecture specific portion is reduced to the low level hardware details of the clock sources, which are registered in the framework and selected on a quality based decision. The low level code provides hardware setup and readout routines and initializes data structures, which are used by the generic time keeping code to convert the clock ticks to nanosecond based time values. All other time keeping related functionality is moved into the generic code. The GTOD base patch got merged into the 2.6.18 kernel.

> John Stultz 的通用时间(GTOD)框架将大部分代码从体系结构特定区域转移到通用管理框架中，如图#3 所示(OLS 幻灯片第 18 页)。架构特定部分被简化为时钟源的低级别硬件细节，这些时钟源被注册在框架中并根据基于质量的决策进行选择。低级代码提供硬件设置和读出例程，并初始化数据结构，通用计时代码使用这些数据结构将时钟信号转换为基于纳秒的时间值。所有其他与计时相关的功能都被移到通用代码中。GTOD 基础补丁被合并到 2.6.18 内核中。

Further information about the Generic Time Of Day framework is available in the OLS 2005 Proceedings Volume 1:
The paper \"We Are Not Getting Any Younger: A New Approach to Time and Timers\" was written by J. Stultz, D.V. Hart, & N. Aravamudan.

> [pdf](http://www.linuxsymposium.org/2005/linuxsymposium_procv1.pdf)

> 关于通用时间框架的更多信息，请参阅 OLS 2005 年会议记录第 1 卷：
> 《我们不再年轻：时间和时间的新方法》一文由 J.Stultz、D.V.Hart 和 N.Aravamudan 撰写。

Figure #3 (OLS slides p.18) illustrates the transformation.

# clock event management

While clock sources provide read access to the monotonically increasing time value, clock event devices are used to schedule the next event interrupt(s). The next event is currently defined to be periodic, with its period defined at compile time. The setup and selection of the event device for various event driven functionalities is hardwired into the architecture dependent code. This results in duplicated code across all architectures and makes it extremely difficult to change the configuration of the system to use event interrupt devices other than those already built into the architecture. Another implication of the current design is that it is necessary to touch all the architecture-specific implementations in order to provide new functionality like high resolution timers or dynamic ticks.

> 当时钟源提供对单调增加的时间值的读取访问时，时钟事件设备用于调度下一个事件中断。下一个事件当前定义为周期性事件，其周期在编译时定义。用于各种事件驱动功能的事件设备的设置和选择被硬连接到依赖于体系结构的代码中。这导致所有体系结构中的代码重复，并使更改系统配置以使用体系结构中已内置的事件中断设备以外的事件中断装置变得极其困难。当前设计的另一个含义是，有必要接触所有特定于体系结构的实现，以便提供新的功能，如高分辨率定时器或动态节拍。

The clock events subsystem tries to address this problem by providing a generic solution to manage clock event devices and their usage for the various clock event driven kernel functionalities. The goal of the clock event subsystem is to minimize the clock event related architecture dependent code to the pure hardware related handling and to allow easy addition and utilization of new clock event devices. It also minimizes the duplicated code across the architectures as it provides generic functionality down to the interrupt service handler, which is almost inherently hardware dependent.

> 时钟事件子系统试图通过提供通用解决方案来管理时钟事件设备及其对各种时钟事件驱动内核功能的使用来解决这个问题。时钟事件子系统的目标是将与时钟事件相关的体系结构相关的代码最小化为纯硬件相关的处理，并允许容易地添加和利用新的时钟事件设备。它还最大限度地减少了跨体系结构的重复代码，因为它提供了向下到中断服务处理器的通用功能，而中断服务处理器几乎本质上依赖于硬件。

Clock event devices are registered either by the architecture dependent boot code or at module insertion time. Each clock event device fills a data structure with clock-specific property parameters and callback functions. The clock event management decides, by using the specified property parameters, the set of system functions a clock event device will be used to support. This includes the distinction of per-CPU and per-system global event devices.

> 时钟事件设备通过依赖于体系结构的引导代码或在模块插入时进行注册。每个时钟事件设备都用特定于时钟的属性参数和回调函数填充数据结构。时钟事件管理通过使用指定的属性参数来决定时钟事件设备将用于支持的一组系统功能。这包括每个 CPU 和每个系统全局事件设备的区别。

System-level global event devices are used for the Linux periodic tick. Per-CPU event devices are used to provide local CPU functionality such as process accounting, profiling, and high resolution timers.

> 系统级全局事件设备用于 Linux 周期性勾选。每 CPU 事件设备用于提供本地 CPU 功能，如进程记帐、分析和高分辨率计时器。

The management layer assigns one or more of the following functions to a clock event device:

> 管理层将以下一个或多个功能分配给时钟事件设备：

> - system global periodic tick (jiffies update)
> - cpu local update_process_times
> - cpu local profiling
> - cpu local next event interrupt (non periodic mode)

The clock event device delegates the selection of those timer interrupt related functions completely to the management layer. The clock management layer stores a function pointer in the device description structure, which has to be called from the hardware level handler. This removes a lot of duplicated code from the architecture specific timer interrupt handlers and hands the control over the clock event devices and the assignment of timer interrupt related functionality to the core code.

> 时钟事件设备将那些定时器中断相关功能的选择完全委托给管理层。时钟管理层在设备描述结构中存储一个函数指针，该指针必须从硬件级处理程序调用。这从体系结构特定的定时器中断处理程序中删除了许多重复的代码，并将对时钟事件设备的控制权和定时器中断相关功能的分配交给了核心代码。

The clock event layer API is rather small. Aside from the clock event device registration interface it provides functions to schedule the next event interrupt, clock event device notification service and support for suspend and resume.

> 时钟事件层 API 相当小。除了时钟事件设备注册接口外，它还提供了安排下一个事件中断的功能、时钟事件设备通知服务以及暂停和恢复支持。

The framework adds about 700 lines of code which results in a 2KB increase of the kernel binary size. The conversion of i386 removes about 100 lines of code. The binary size decrease is in the range of 400 byte. We believe that the increase of flexibility and the avoidance of duplicated code across architectures justifies the slight increase of the binary size.

> 该框架添加了大约 700 行代码，这导致内核二进制大小增加了 2KB。i386 的转换删除了大约 100 行代码。二进制大小的减少在 400 字节的范围内。我们认为，增加灵活性和避免跨体系结构的重复代码证明了二进制大小的轻微增加是合理的。

The conversion of an architecture has no functional impact, but allows to utilize the high resolution and dynamic tick functionalities without any change to the clock event device and timer interrupt code. After the conversion the enabling of high resolution timers and dynamic ticks is simply provided by adding the kernel/time/Kconfig file to the architecture specific Kconfig and adding the dynamic tick specific calls to the idle routine (a total of 3 lines added to the idle function and the Kconfig file)

> 架构的转换没有功能影响，但允许在不改变时钟事件设备和定时器中断代码的情况下利用高分辨率和动态滴答功能。转换后，只需将内核/time/Kconfig 文件添加到特定于体系结构的 Kconfig 中，并将特定于动态 tick 的调用添加到空闲例程中(总共有 3 行添加到空闲函数和 Kconfig 文件中)，即可启用高分辨率定时器和动态 tick

Figure #4 (OLS slides p.20) illustrates the transformation.

# high resolution timer functionality

During system boot it is not possible to use the high resolution timer functionality, while making it possible would be difficult and would serve no useful function. The initialization of the clock event device framework, the clock source framework (GTOD) and hrtimers itself has to be done and appropriate clock sources and clock event devices have to be registered before the high resolution functionality can work. Up to the point where hrtimers are initialized, the system works in the usual low resolution periodic mode. The clock source and the clock event device layers provide notification functions which inform hrtimers about availability of new hardware. hrtimers validates the usability of the registered clock sources and clock event devices before switching to high resolution mode. This ensures also that a kernel which is configured for high resolution timers can run on a system which lacks the necessary hardware support.

> 在系统引导期间，不可能使用高分辨率定时器功能，而使其成为可能将是困难的，并且将没有任何有用的功能。必须完成时钟事件设备框架、时钟源框架(GTOD)和 hrtimers 本身的初始化，并且必须注册适当的时钟源和时钟事件设备，才能使用高分辨率功能。在 hrtimers 初始化之前，系统以通常的低分辨率周期模式工作。时钟源和时钟事件设备层提供通知功能，通知 hr 定时器新硬件的可用性。hrtimers 在切换到高分辨率模式之前验证已注册时钟源和时钟事件设备的可用性。这也确保了为高分辨率定时器配置的内核可以在缺乏必要硬件支持的系统上运行。

The high resolution timer code does not support SMP machines which have only global clock event devices. The support of such hardware would involve IPI calls when an interrupt happens. The overhead would be much larger than the benefit. This is the reason why we currently disable high resolution and dynamic ticks on i386 SMP systems which stop the local APIC in C3 power state. A workaround is available as an idea, but the problem has not been tackled yet.

> 高分辨率定时器代码不支持只有全局时钟事件设备的 SMP 机器。这种硬件的支持将涉及中断发生时的 IPI 调用。开销将远远大于收益。这就是为什么我们目前在 i386 SMP 系统上禁用高分辨率和动态滴答声的原因，这些系统会在 C3 电源状态下停止本地 APIC。一个变通办法是可行的，但这个问题还没有得到解决。

The time ordered insertion of timers provides all the infrastructure to decide whether the event device has to be reprogrammed when a timer is added. The decision is made per timer base and synchronized across per-cpu timer bases in a support function. The design allows the system to utilize separate per-CPU clock event devices for the per-CPU timer bases, but currently only one reprogrammable clock event device per-CPU is utilized.

> 定时器的按时间顺序插入提供了所有基础设施，以决定在添加定时器时是否必须对事件设备进行重新编程。在支持功能中，决策是按每个定时器基数进行的，并在每个 cpu 定时器基数之间同步。该设计允许系统使用单独的每 CPU 时钟事件设备作为每 CPU 定时器基础，但目前每个 CPU 仅使用一个可重新编程的时钟事件设备。

When the timer interrupt happens, the next event interrupt handler is called from the clock event distribution code and moves expired timers from the red-black tree to a separate double linked list and invokes the softirq handler. An additional mode field in the hrtimer structure allows the system to execute callback functions directly from the next event interrupt handler. This is restricted to code which can safely be executed in the hard interrupt context. This applies, for example, to the common case of a wakeup function as used by nanosleep. The advantage of executing the handler in the interrupt context is the avoidance of up to two context switches - from the interrupted context to the softirq and to the task which is woken up by the expired timer.

> 当定时器中断发生时，从时钟事件分发代码调用下一个事件中断处理程序，并将过期的定时器从红黑树移动到一个单独的双链表中，并调用 softirq 处理程序。hrtimer 结构中的一个附加模式字段允许系统直接从下一个事件中断处理程序执行回调函数。这仅限于可以在硬中断上下文中安全执行的代码。例如，这适用于纳米睡眠所使用的唤醒功能的常见情况。在中断上下文中执行处理程序的优点是避免了最多两个上下文切换——从中断的上下文切换到 softirq，再切换到被过期计时器唤醒的任务。

Once a system has switched to high resolution mode, the periodic tick is switched off. This disables the per system global periodic clock event device -e.g. the PIT on i386 SMP systems.

> 一旦系统切换到高分辨率模式，周期性勾号就会关闭。这会禁用每个系统的全局周期性时钟事件设备，例如 i386SMP 系统上的 PIT。

The periodic tick functionality is provided by an per-cpu hrtimer. The callback function is executed in the next event interrupt context and updates jiffies and calls update_process_times and profiling. The implementation of the hrtimer based periodic tick is designed to be extended with dynamic tick functionality. This allows to use a single clock event device to schedule high resolution timer and periodic events (jiffies tick, profiling, process accounting) on UP systems. This has been proved to work with the PIT on i386 and the Incrementer on PPC.

> 周期性刻度功能由每 cpu hr 计时器提供。回调函数在下一个事件中断上下文中执行，更新 jiffies 并调用 update_process_times 和 profiling。基于 hrtimer 的周期性刻度的实现旨在通过动态刻度功能进行扩展。这允许使用单个时钟事件设备来调度 UP 系统上的高分辨率计时器和周期性事件(jiffies tick、profiling、process accounting)。这已经被证明可以与 i386 上的 PIT 和 PPC 上的 Incrementer 一起工作。

The softirq for running the hrtimer queues and executing the callbacks has been separated from the tick bound timer softirq to allow accurate delivery of high resolution timer signals which are used by itimer and POSIX interval timers. The execution of this softirq can still be delayed by other softirqs, but the overall latencies have been significantly improved by this separation.

> 用于运行 hrtimer 队列和执行回调的 softirq 已与 tick-bound 定时器 softirq 分离，以允许精确传递 itimer 和 POSIX 间隔定时器使用的高分辨率定时器信号。这种软 irq 的执行仍然可以被其他软 irq 延迟，但通过这种分离，总体延迟已经显著改善。

Figure #5 (OLS slides p.22) illustrates the transformation.

# dynamic ticks

Dynamic ticks are the logical consequence of the hrtimer based periodic tick replacement (sched_tick). The functionality of the sched_tick hrtimer is extended by three functions:

> 动态刻度是基于 hrtimer 的定期刻度替换(sched_tick)的逻辑结果。sched_tick hr 计时器的功能由三个功能扩展：

- hrtimer_stop_sched_tick
- hrtimer_restart_sched_tick
- hrtimer_update_jiffies

hrtimer_stop_sched_tick() is called when a CPU goes into idle state. The code evaluates the next scheduled timer event (from both hrtimers and the timer wheel) and in case that the next event is further away than the next tick it reprograms the sched_tick to this future event, to allow longer idle sleeps without worthless interruption by the periodic tick. The function is also called when an interrupt happens during the idle period, which does not cause a reschedule. The call is necessary as the interrupt handler might have armed a new timer whose expiry time is before the time which was identified as the nearest event in the previous call to hrtimer_stop_sched_tick.

> hrtimer_stop_sched_tick()在 CPU 进入空闲状态时调用。该代码评估下一个定时定时器事件(来自 hr 定时器和定时器轮)，如果下一个事件比下一个刻度更远，则将 sched_tick 重新编程为该未来事件，以允许更长的空闲睡眠时间，而不会因周期性刻度而中断。当空闲期间发生中断时，也会调用该函数，这不会导致重新安排。该调用是必要的，因为中断处理程序可能已经武装了一个新计时器，该计时器的到期时间早于上次调用 hrtimer_stop_sched_tick 时被识别为最近事件的时间。

hrtimer_restart_sched_tick() is called when the CPU leaves the idle state before it calls schedule(). hrtimer_restart_sched_tick() resumes the periodic tick, which is kept active until the next call to hrtimer_stop_sched_tick().

> hrtimer_restart_sched_tick()是在 CPU 在调用 schedule()之前离开空闲状态时调用的。hrtimer_restart_sched_tick()恢复周期性刻度，该刻度保持活动状态，直到下一次调用 hrtimer_stop_sched_tick()为止。

hrtimer_update_jiffies() is called from irq_enter() when an interrupt happens in the idle period to make sure that jiffies are up to date and the interrupt handler has not to deal with an eventually stale jiffy value.

> 当空闲期间发生中断时，从 irq_enter()调用 hrtimer_update_jiffies()，以确保 jiffies 是最新的，并且中断处理程序不必处理最终过时的 jiffy 值。

The dynamic tick feature provides statistical values which are exported to userspace via /proc/stat and can be made available for enhanced power management control.

> 动态刻度功能提供统计值，这些值通过/proc/stat 导出到用户空间，并可用于增强电源管理控制。

The implementation leaves room for further development like full tickless systems, where the time slice is controlled by the scheduler, variable frequency profiling, and a complete removal of jiffies in the future.

> 该实现为进一步开发留下了空间，如完全无挠系统，其中时间片由调度器控制，可变频率分析，并在未来完全消除抖动。

Aside the current initial submission of i386 support, the patchset has been extended to x86_64 and ARM already. Initial (work in progress) support is also available for MIPS and PowerPC.

> 除了目前首次提交的 i386 支持外，补丁集已经扩展到 x86_64 和 ARM。MIPS 和 PowerPC 也可以获得初始(正在进行的工作)支持。

> Thomas, Ingo
