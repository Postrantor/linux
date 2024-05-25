---
tip: translate by baidu@2024-05-25 19:54:21
title: hrtimers - subsystem for high-resolution kernel timers
---

This patch introduces a new subsystem for high-resolution kernel timers.

One might ask the question: we already have a timer subsystem (kernel/timers.c), why do we need two timer subsystems? After a lot of back and forth trying to integrate high-resolution and high-precision features into the existing timer framework, and after testing various such high-resolution timer implementations in practice, we came to the conclusion that the timer wheel code is fundamentally not suitable for such an approach. We initially didn\'t believe this (\'there must be a way to solve this\'), and spent a considerable effort trying to integrate things into the timer wheel, but we failed. In hindsight, there are several reasons why such integration is hard/impossible:

> 有人可能会问这样一个问题：我们已经有了一个定时器子系统(kernel/timers.c)，为什么我们需要两个子系统呢？在反复尝试将高分辨率和高精度功能集成到现有的定时器框架中，并在实践中测试了各种此类高分辨率定时器实现后，我们得出的结论是，定时器轮代码根本不适合这种方法。我们最初不相信这一点("一定有办法解决这个问题")，并花费了相当大的努力试图将事物集成到计时器轮中，但我们失败了。事后看来，这种整合很难/不可能的原因有几个：

- the forced handling of low-resolution and high-resolution timers in the same way leads to a lot of compromises, macro magic and #ifdef mess. The timers.c code is very \"tightly coded\" around jiffies and 32-bitness assumptions, and has been honed and micro-optimized for a relatively narrow use case (jiffies in a relatively narrow HZ range) for many years - and thus even small extensions to it easily break the wheel concept, leading to even worse compromises. The timer wheel code is very good and tight code, there\'s zero problems with it in its current usage - but it is simply not suitable to be extended for high-res timers.

> - 以同样的方式强制处理低分辨率和高分辨率定时器会导致很多妥协、宏魔术和#ifdef 混乱。timers.c 代码围绕着 jiffies 和 32 位的假设进行了非常"严格的编码"，多年来一直在相对较窄的用例(在相对较短的赫兹范围内的 jiffies)中进行优化和微优化，因此即使是对它的小扩展也很容易打破轮子的概念，导致更糟糕的妥协。定时器轮代码是非常好和紧凑的代码，在目前的使用中没有任何问题，但它根本不适合扩展到高分辨率定时器。

- the unpredictable \[O(N)\] overhead of cascading leads to delays which necessitate a more complex handling of high resolution timers, which in turn decreases robustness. Such a design still leads to rather large timing inaccuracies. Cascading is a fundamental property of the timer wheel concept, it cannot be \'designed out\' without inevitably degrading other portions of the timers.c code in an unacceptable way.

> - 级联的不可预测的\[O(N)\]开销导致延迟，这需要对高分辨率定时器进行更复杂的处理，这反过来又降低了鲁棒性。这样的设计仍然导致相当大的时序不准确性。级联是定时器轮概念的一个基本特性，如果不以不可接受的方式不可避免地降低定时器.c 代码的其他部分，就无法"设计出来"。

- the implementation of the current posix-timer subsystem on top of the timer wheel has already introduced a quite complex handling of the required readjusting of absolute CLOCK_REALTIME timers at settimeofday or NTP time - further underlying our experience by example: that the timer wheel data structure is too rigid for high-res timers.

> - 在定时器轮顶部的当前 posix 定时器子系统的实现已经引入了在 settimeofday 或 NTP 时间对绝对 CLOCK_REALTIME 定时器的所需重新调整的相当复杂的处理-通过示例进一步加深了我们的经验：定时器轮数据结构对于高分辨率定时器来说过于僵化。

- the timer wheel code is most optimal for use cases which can be identified as \"timeouts\". Such timeouts are usually set up to cover error conditions in various I/O paths, such as networking and block I/O. The vast majority of those timers never expire and are rarely recascaded because the expected correct event arrives in time so they can be removed from the timer wheel before any further processing of them becomes necessary. Thus the users of these timeouts can accept the granularity and precision tradeoffs of the timer wheel, and largely expect the timer subsystem to have near-zero overhead. Accurate timing for them is not a core purpose - in fact most of the timeout values used are ad-hoc. For them it is at most a necessary evil to guarantee the processing of actual timeout completions (because most of the timeouts are deleted before completion), which should thus be as cheap and unintrusive as possible.

> - 定时器轮代码对于可以识别为"超时"的用例来说是最理想的。这种超时通常是为了覆盖各种 I/O 路径中的错误情况而设置的，如网络和块 I/O。这些定时器中的绝大多数永远不会过期，也很少被重新调度，因为预期的正确事件及时到达，因此可以在需要对其进一步处理之前将其从定时器轮中删除。因此，这些超时的用户可以接受定时器轮的粒度和精度权衡，并在很大程度上期望定时器子系统具有接近零的开销。对它们来说，准确的计时并不是核心目的——事实上，使用的大多数超时值都是临时的。对他们来说，保证实际超时完成的处理最多是一种必要的邪恶(因为大多数超时都是在完成之前删除的)，因此应该尽可能便宜且不具侵入性。

The primary users of precision timers are user-space applications that utilize nanosleep, posix-timers and itimer interfaces. Also, in-kernel users like drivers and subsystems which require precise timed events (e.g. multimedia) can benefit from the availability of a separate high-resolution timer subsystem as well.

> 精密定时器的主要用户是使用 nanosleep、posix 定时器和 itimer 接口的用户空间应用程序。此外，像驱动程序和子系统这样需要精确定时事件(例如多媒体)的内核内用户也可以受益于单独的高分辨率定时器子系统的可用性。

While this subsystem does not offer high-resolution clock sources just yet, the hrtimer subsystem can be easily extended with high-resolution clock capabilities, and patches for that exist and are maturing quickly. The increasing demand for realtime and multimedia applications along with other potential users for precise timers gives another reason to separate the \"timeout\" and \"precise timer\" subsystems.

> 虽然该子系统目前还没有提供高分辨率时钟源，但 hrtimer 子系统可以很容易地扩展高分辨率时钟功能，并且其补丁已经存在并正在迅速成熟。实时和多媒体应用程序以及其他潜在用户对精确计时器的需求不断增加，这为分离"超时"和"精确计时器"子系统提供了另一个理由。

Another potential benefit is that such a separation allows even more special-purpose optimization of the existing timer wheel for the low resolution and low precision use cases - once the precision-sensitive APIs are separated from the timer wheel and are migrated over to hrtimers. E.g. we could decrease the frequency of the timeout subsystem from 250 Hz to 100 HZ (or even smaller).

> 另一个潜在的好处是，一旦精度敏感的 API 与计时器轮分离并迁移到 hrtimers，这种分离就可以针对低分辨率和低精度的用例对现有计时器轮进行更特殊的优化。例如，我们可以将超时子系统的频率从 250 赫兹降低到 100 赫兹(甚至更小)。

# hrtimer subsystem implementation details

the basic design considerations were:

- simplicity
- data structure not bound to jiffies or any other granularity. All the kernel logic works at 64-bit nanoseconds resolution - no compromises.
- simplification of existing, timing related kernel code

> - 不绑定到 jiffies 或任何其他粒度的数据结构。所有内核逻辑都以 64 位纳秒的分辨率工作——没有任何妥协。

another basic requirement was the immediate enqueueing and ordering of timers at activation time. After looking at several possible solutions such as radix trees and hashes, we chose the red black tree as the basic data structure. Rbtrees are available as a library in the kernel and are used in various performance-critical areas of e.g. memory management and file systems. The rbtree is solely used for time sorted ordering, while a separate list is used to give the expiry code fast access to the queued timers, without having to walk the rbtree.

> 另一个基本要求是在激活时立即对定时器进行排队和排序。在研究了几种可能的解决方案(如基数树和哈希)后，我们选择了红黑树作为基本数据结构。Rbtree 可作为内核中的库使用，并用于内存管理和文件系统等各种性能关键领域。rbtree 仅用于时间排序，而单独的列表用于让到期代码快速访问排队的计时器，而不必遍历 rbtree。

(This separate list is also useful for later when we\'ll introduce high-resolution clocks, where we need separate pending and expired queues while keeping the time-order intact.)

> (这个单独的列表也很有用，稍后我们将引入高分辨率时钟，在这里我们需要单独的挂起和过期队列，同时保持时间顺序不变。)

Time-ordered enqueueing is not purely for the purposes of high-resolution clocks though, it also simplifies the handling of absolute timers based on a low-resolution CLOCK_REALTIME. The existing implementation needed to keep an extra list of all armed absolute CLOCK_REALTIME timers along with complex locking. In case of settimeofday and NTP, all the timers (!) had to be dequeued, the time-changing code had to fix them up one by one, and all of them had to be enqueued again. The time-ordered enqueueing and the storage of the expiry time in absolute time units removes all this complex and poorly scaling code from the posix-timer implementation - the clock can simply be set without having to touch the rbtree. This also makes the handling of posix-timers simpler in general.

> 时间有序排队不仅仅是为了高分辨率时钟，它还简化了基于低分辨率 CLOCK_REALTIME 的绝对定时器的处理。现有的实现需要保留所有武装的绝对 CLOCK_REALTIME 定时器的额外列表以及复杂的锁定。在设置日期和 NTP 的情况下，所有计时器(！)都必须退出队列，时间更改代码必须逐一修复它们，并且所有计时器都必须重新排队。按时间顺序排队和以绝对时间为单位存储到期时间，从 posix 定时器实现中消除了所有这些复杂且扩展性差的代码——只需设置时钟，无需触摸 rbtree。这也使 posix 定时器的处理总体上更简单。

The locking and per-CPU behavior of hrtimers was mostly taken from the existing timer wheel code, as it is mature and well suited. Sharing code was not really a win, due to the different data structures. Also, the hrtimer functions now have clearer behavior and clearer names - such as hrtimer_try_to_cancel() and hrtimer_cancel() \[which are roughly equivalent to timer_delete() and timer_delete_sync()\] - so there\'s no direct 1:1 mapping between them on the algorithmic level, and thus no real potential for code sharing either.

> hrtimers 的锁定和每个 CPU 的行为大多取自现有的定时器轮代码，因为它是成熟的，非常适合。由于数据结构不同，共享代码并不是真正的胜利。此外，hrtimer 函数现在有了更清晰的行为和更清晰的名称，如 hrtimer_try_to_cancel()和 hrtimer_cancel(。

Basic data types: every time value, absolute or relative, is in a special nanosecond-resolution 64bit type: ktime_t. (Originally, the kernel-internal representation of ktime_t values and operations was implemented via macros and inline functions, and could be switched between a \"hybrid union\" type and a plain \"scalar\" 64bit nanoseconds representation (at compile time). This was abandoned in the context of the Y2038 work.)

> 基本数据类型：每个时间值，无论是绝对值还是相对值，都是一种特殊的纳秒分辨率 64 位类型：ktime_t。这是在 2038 年工作的背景下放弃的。)

# hrtimers - rounding of timer values

the hrtimer code will round timer events to lower-resolution clocks because it has to. Otherwise it will do no artificial rounding at all.

> hrtimer 代码将计时器事件四舍五入到较低分辨率的时钟，因为它必须这样做。否则，它将根本不进行人工四舍五进。

one question is, what resolution value should be returned to the user by the clock_getres() interface. This will return whatever real resolution a given clock has - be it low-res, high-res, or artificially-low-res.

> 一个问题是，`clock_geters()` 接口应该向用户返回什么分辨率值。这将返回给定时钟的实际分辨率，无论是低分辨率、高分辨率还是人工低分辨率。

# hrtimers - testing and verification

We used the high-resolution clock subsystem on top of hrtimers to verify the hrtimer implementation details in praxis, and we also ran the posix timer tests in order to ensure specification compliance. We also ran tests on low-resolution clocks.

> 我们使用 hrtimers 之上的高分辨率时钟子系统来验证 praxis 中的 hrtimer 实现细节，我们还运行了 posix 定时器测试，以确保符合规范。我们还对低分辨率时钟进行了测试。

The hrtimer patch converts the following kernel functionality to use hrtimers:

> hrtimer 修补程序将以下内核功能转换为使用 hrtimers：

> - nanosleep
> - itimers
> - posix-timers

The conversion of nanosleep and posix-timers enabled the unification of nanosleep and clock_nanosleep.

> nanosleep 和 posix 定时器的转换实现了 nanosleep 与 clock_nanosleep 的统一。

The code was successfully compiled for the following platforms:

> i386, x86_64, ARM, PPC, PPC64, IA64

The code was run-tested on the following platforms:

> i386(UP/SMP), x86_64(UP/SMP), ARM, PPC

hrtimers were also integrated into the -rt tree, along with a hrtimers-based high-resolution clock implementation, so the hrtimers code got a healthy amount of testing and use in practice.

> hrtimers 还集成到 `-rt` 树中，以及基于 hrtimers 的高分辨率时钟实现，因此 hrtimers 代码在实践中得到了大量的测试和使用。

> Thomas Gleixner, Ingo Molnar
