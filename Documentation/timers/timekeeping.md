---
tip: translate by baidu@2024-05-25 19:54:40
title: Clock sources, Clock events, sched_clock() and delay timers
---

This document tries to briefly explain some basic kernel timekeeping abstractions. It partly pertains to the drivers usually found in drivers/clocksource in the kernel tree, but the code may be spread out across the kernel.

> 本文试图简要解释一些基本的内核计时抽象。它在一定程度上与通常在内核树中的驱动程序/时钟源中找到的驱动程序有关，但代码可能分布在整个内核中。

If you grep through the kernel source you will find a number of architecture-specific implementations of clock sources, clockevents and several likewise architecture-specific overrides of the sched_clock() function and some delay timers.

> 如果您通过内核源代码进行 grep，您将发现时钟源、时钟事件的许多特定于体系结构的实现，以及 sched_clock()函数和一些延迟定时器的一些类似的特定于体系架构的覆盖。

To provide timekeeping for your platform, the clock source provides the basic timeline, whereas clock events shoot interrupts on certain points on this timeline, providing facilities such as high-resolution timers. sched_clock() is used for scheduling and timestamping, and delay timers provide an accurate delay source using hardware counters.

> 为了为您的平台提供计时，时钟源提供了基本的时间线，而时钟事件在该时间线上的某些点上拍摄中断，提供了高分辨率计时器等设施。sched_clock()用于调度和时间戳，延迟计时器使用硬件计数器提供准确的延迟源。

# Clock sources

The purpose of the clock source is to provide a timeline for the system that tells you where you are in time. For example issuing the command \'date\' on a Linux system will eventually read the clock source to determine exactly what time it is.

> 时钟源的目的是为系统提供一个时间线，告诉你在时间上的位置。例如，在 Linux 系统上发出“日期”命令最终会读取时钟源，以确定它的确切时间。

Typically the clock source is a monotonic, atomic counter which will provide n bits which count from 0 to (2\^n)-1 and then wraps around to 0 and start over. It will ideally NEVER stop ticking as long as the system is running. It may stop during system suspend.

> 通常，时钟源是一个单调的原子计数器，它将提供从 0 到(2n)-1 计数的 n 位，然后循环到 0 并重新开始。理想情况下，只要系统在运行，它就永远不会停止滴答作响。它可能在系统挂起期间停止。

The clock source shall have as high resolution as possible, and the frequency shall be as stable and correct as possible as compared to a real-world wall clock. It should not move unpredictably back and forth in time or miss a few cycles here and there.

> 时钟源应具有尽可能高的分辨率，与真实世界的挂钟相比，频率应尽可能稳定和正确。它不应该在时间上不可预测地来回移动，也不应该错过一些周期。

It must be immune to the kind of effects that occur in hardware where e.g. the counter register is read in two phases on the bus lowest 16 bits first and the higher 16 bits in a second bus cycle with the counter bits potentially being updated in between leading to the risk of very strange values from the counter.

> 它必须不受硬件中发生的那种影响，其中例如计数器寄存器在第二总线周期中分两个阶段在总线上读取最低的 16 位第一个和较高的 16 位，计数器位可能在这两个阶段之间被更新，从而导致来自计数器的非常奇怪的值的风险。

When the wall-clock accuracy of the clock source isn\'t satisfactory, there are various quirks and layers in the timekeeping code for e.g. synchronizing the user-visible time to RTC clocks in the system or against networked time servers using NTP, but all they do basically is update an offset against the clock source, which provides the fundamental timeline for the system. These measures does not affect the clock source per se, they only adapt the system to the shortcomings of it.

> 当时钟源的墙上时钟精度不令人满意时，在计时代码中有各种各样的怪癖和层次，例如将用户可见时间同步到系统中的 RTC 时钟或使用 NTP 的网络时间服务器，但它们所做的基本上只是更新时钟源偏移量，这为系统提供了基本的时间线。这些措施并不影响时钟源本身，它们只是使系统适应其缺点。

The clock source struct shall provide means to translate the provided counter into a nanosecond value as an unsigned long long (unsigned 64 bit) number. Since this operation may be invoked very often, doing this in a strict mathematical sense is not desirable: instead the number is taken as close as possible to a nanosecond value using only the arithmetic operations multiply and shift, so in clocksource_cyc2ns() you find:

> 时钟源结构应提供将所提供的计数器转换为纳秒值的方法，该值为无符号长-长(无符号 64 位)数。由于此操作可能经常被调用，因此在严格的数学意义上执行此操作是不可取的：相反，仅使用算术运算乘法和移位将数字取为尽可能接近纳秒的值，因此在 clocksource_cyc2ns()中，您会发现：

> ns \~= (clocksource \* mult) \>\> shift

You will find a number of helper functions in the clock source code intended to aid in providing these mult and shift values, such as clocksource_khz2mult(), clocksource_hz2mult() that help determine the mult factor from a fixed shift, and clocksource_register_hz() and clocksource_register_khz() which will help out assigning both shift and mult factors using the frequency of the clock source as the only input.

> 您将在时钟源代码中找到许多辅助函数，这些函数旨在帮助提供这些多值和移位值，例如 clocksource_khz2mult()、clocksource_hz2mult()，它们有助于从固定移位中确定多因子，以及 clocksource_register_hz()和 clocksource_Registerkhz()，它将有助于使用时钟源的频率作为唯一输入来分配移位因子和多因子。

For real simple clock sources accessed from a single I/O memory location there is nowadays even clocksource_mmio_init() which will take a memory location, bit width, a parameter telling whether the counter in the register counts up or down, and the timer clock rate, and then conjure all necessary parameters.

> 对于从单个 I/O 内存位置访问的真正简单的时钟源，现在甚至有 clocksource_mmio_init()，它将采用内存位置、位宽、告诉寄存器中计数器是向上计数还是向下计数的参数以及定时器时钟速率，然后变出所有必要的参数。

Since a 32-bit counter at say 100 MHz will wrap around to zero after some 43 seconds, the code handling the clock source will have to compensate for this. That is the reason why the clock source struct also contains a \'mask\' member telling how many bits of the source are valid. This way the timekeeping code knows when the counter will wrap around and can insert the necessary compensation code on both sides of the wrap point so that the system timeline remains monotonic.

> 由于 100MHz 的 32 位计数器将在大约 43 秒后恢复为零，因此处理时钟源的代码将不得不对此进行补偿。这就是为什么时钟源结构还包含一个“ask”成员来告诉源有多少位是有效的。通过这种方式，计时代码知道计数器何时环绕，并且可以在环绕点的两侧插入必要的补偿代码，使得系统时间线保持单调。

# Clock events

Clock events are the conceptual reverse of clock sources: they take a desired time specification value and calculate the values to poke into hardware timer registers.

> 时钟事件与时钟源的概念相反：它们采用所需的时间规范值，并计算这些值以插入硬件定时器寄存器。

Clock events are orthogonal to clock sources. The same hardware and register range may be used for the clock event, but it is essentially a different thing. The hardware driving clock events has to be able to fire interrupts, so as to trigger events on the system timeline. On an SMP system, it is ideal (and customary) to have one such event driving timer per CPU core, so that each core can trigger events independently of any other core.

> 时钟事件与时钟源正交。时钟事件可以使用相同的硬件和寄存器范围，但本质上是不同的。硬件驱动时钟事件必须能够触发中断，从而触发系统时间线上的事件。在 SMP 系统上，每个 CPU 核心有一个这样的事件驱动定时器是理想的(也是习惯的)，这样每个核心都可以独立于任何其他核心触发事件。

You will notice that the clock event device code is based on the same basic idea about translating counters to nanoseconds using mult and shift arithmetic, and you find the same family of helper functions again for assigning these values. The clock event driver does not need a \'mask\' attribute however: the system will not try to plan events beyond the time horizon of the clock event.

> 您会注意到，时钟事件设备代码基于相同的基本思想，即使用 mult 和 shift 算术将计数器转换为纳秒，您会再次找到相同的辅助函数族来分配这些值。但是，时钟事件驱动程序不需要“任务”属性：系统不会试图计划超出时钟事件时间范围的事件。

# sched_clock()

In addition to the clock sources and clock events there is a special weak function in the kernel called sched_clock(). This function shall return the number of nanoseconds since the system was started. An architecture may or may not provide an implementation of sched_clock() on its own. If a local implementation is not provided, the system jiffy counter will be used as sched_clock().

> 除了时钟源和时钟事件之外，内核中还有一个特殊的弱函数，称为 sched_clock()。此函数应返回自系统启动以来的纳秒数。一个体系结构可能会也可能不会单独提供 sched_clock()的实现。如果没有提供本地实现，则系统 jiffy 计数器将用作 sched_clock()。

As the name suggests, sched_clock() is used for scheduling the system, determining the absolute timeslice for a certain process in the CFS scheduler for example. It is also used for printk timestamps when you have selected to include time information in printk for things like bootcharts.

> 顾名思义，sched_clock()用于对系统进行调度，例如确定 CFS 调度器中某个进程的绝对时间片。当您选择在 printk 中包含诸如 bootcharts 之类的时间信息时，它也用于 printk 时间戳。

Compared to clock sources, sched_clock() has to be very fast: it is called much more often, especially by the scheduler. If you have to do trade-offs between accuracy compared to the clock source, you may sacrifice accuracy for speed in sched_clock(). It however requires some of the same basic characteristics as the clock source, i.e. it should be monotonic.

> 与时钟源相比，sched_clock()必须非常快：它的调用频率要高得多，尤其是调度程序。如果您必须在与时钟源相比的准确性之间进行权衡，那么您可能会在 sched_clock()中牺牲准确性来换取速度。然而，它需要与时钟源相同的一些基本特性，即它应该是单调的。

The sched_clock() function may wrap only on unsigned long long boundaries, i.e. after 64 bits. Since this is a nanosecond value this will mean it wraps after circa 585 years. (For most practical systems this means \"never\".)

> sched_clock()函数可以仅在无符号长-长边界上换行，即在 64 位之后。由于这是一个纳秒的数值，这意味着它将在大约 585 年后结束。(对于大多数实际系统，这意味着“永远不会”。)

If an architecture does not provide its own implementation of this function, it will fall back to using jiffies, making its maximum resolution 1/HZ of the jiffy frequency for the architecture. This will affect scheduling accuracy and will likely show up in system benchmarks.

> 如果一个体系结构不提供自己的该功能实现，它将重新使用 jiffies，使其最大分辨率为该体系结构的 jiffy 频率的 1/HZ。这将影响调度的准确性，并可能出现在系统基准测试中。

The clock driving sched_clock() may stop or reset to zero during system suspend/sleep. This does not matter to the function it serves of scheduling events on the system. However it may result in interesting timestamps in printk().

> 在系统暂停/睡眠期间，驱动 sched_clock()的时钟可能会停止或重置为零。这与它在系统上调度事件的功能无关。然而，它可能会在 printk()中产生有趣的时间戳。

The sched_clock() function should be callable in any context, IRQ- and NMI-safe and return a sane value in any context.

> sched_clock()函数应该在任何上下文中都是可调用的，IRQ 和 NMI 是安全的，并且在任何上下文下都返回一个合理的值。

Some architectures may have a limited set of time sources and lack a nice counter to derive a 64-bit nanosecond value, so for example on the ARM architecture, special helper functions have been created to provide a sched_clock() nanosecond base from a 16- or 32-bit counter. Sometimes the same counter that is also used as clock source is used for this purpose.

> 一些体系结构可能具有有限的时间源集，并且缺乏一个很好的计数器来导出 64 位纳秒值，因此，例如，在 ARM 体系结构上，已经创建了特殊的帮助函数来从 16 位或 32 位计数器提供 sched_clock()纳秒基。有时，也用作时钟源的同一计数器用于此目的。

On SMP systems, it is crucial for performance that sched_clock() can be called independently on each CPU without any synchronization performance hits. Some hardware (such as the x86 TSC) will cause the sched_clock() function to drift between the CPUs on the system. The kernel can work around this by enabling the CONFIG_HAVE_UNSTABLE_SCHED_CLOCK option. This is another aspect that makes sched_clock() different from the ordinary clock source.

> 在 SMP 系统上，对性能至关重要的是，可以在每个 CPU 上独立调用 sched_clock()，而不会影响任何同步性能。某些硬件(如 x86 TSC)会导致 sched_clock()函数在系统上的 CPU 之间漂移。内核可以通过启用 CONFIG_HAVE_INSTABLE_SCHED_CLOCK 选项来解决此问题。这是使 sched_clock()不同于普通时钟源的另一个方面。

# Delay timers (some architectures only)

On systems with variable CPU frequency, the various kernel delay() functions will sometimes behave strangely. Basically these delays usually use a hard loop to delay a certain number of jiffy fractions using a \"lpj\" (loops per jiffy) value, calibrated on boot.

> 在 CPU 频率可变的系统上，各种内核延迟()函数有时会表现得很奇怪。基本上，这些延迟通常使用一个硬循环来延迟一定数量的 jiffy 分数，使用“lpj”(每 jiffy 循环)值，在引导时进行校准。

Let\'s hope that your system is running on maximum frequency when this value is calibrated: as an effect when the frequency is geared down to half the full frequency, any delay() will be twice as long. Usually this does not hurt, as you\'re commonly requesting that amount of delay _or more_. But basically the semantics are quite unpredictable on such systems.

> 当这个值被校准时，我们希望你的系统在最大频率上运行：当频率降到全频率的一半时，任何延迟()的时间都将是它的两倍。通常情况下，这不会有什么坏处，因为你通常会要求延迟一段时间或更长时间。但基本上，这种系统的语义是不可预测的。

Enter timer-based delays. Using these, a timer read may be used instead of a hard-coded loop for providing the desired delay.

> 输入基于计时器的延迟。使用这些，可以使用定时器读取来代替硬编码环路，以提供期望的延迟。

This is done by declaring a struct delay_timer and assigning the appropriate function pointers and rate settings for this delay timer.

> 这是通过声明一个结构 delay_timer 并为该延迟定时器分配适当的函数指针和速率设置来完成的。

This is available on some architectures like OpenRISC or ARM.
