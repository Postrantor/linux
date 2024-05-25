---
tip: translate by baidu@2024-05-25 19:54:47
title: delays - Information on the various kernel delay / sleep mechanisms
---

This document seeks to answer the common question: \"What is the RightWay (TM) to insert a delay?\"

> 本文件旨在回答一个常见问题:“插入延迟的 RightWay(TM)是什么？”

This question is most often faced by driver writers who have to deal with hardware delays and who may not be the most intimately familiar with the inner workings of the Linux Kernel.

> 这个问题最常见的是驱动程序编写者，他们必须处理硬件延迟，并且可能不太熟悉 Linux 内核的内部工作。

# Inserting Delays

The first, and most important, question you need to ask is \"Is my code in an atomic context?\" This should be followed closely by \"Does it really need to delay in atomic context?\" If so\...

> 你需要问的第一个也是最重要的问题是“我的代码在原子上下文中吗？”后面应该紧跟着“它真的需要在原子上下文延迟吗？”如果是的话。。。

ATOMIC CONTEXT:

: You must use the [\*delay]{.title-ref} family of functions. These functions use the jiffie estimation of clock speed and will busy wait for enough loop cycles to achieve the desired delay:

> :必须使用[\*delay]｛.title-ref｝函数族。这些函数使用时钟速度的 jiffie 估计，并将忙于等待足够的循环周期以实现所需的延迟:

```
ndelay(unsigned long nsecs) udelay(unsigned long usecs) mdelay(unsigned long msecs)

udelay is the generally preferred API; ndelay-level precision may not actually exist on many non-PC devices.

mdelay is macro wrapper around udelay, to account for possible overflow when passing large arguments to udelay. In general, use of mdelay is discouraged and code should be refactored to allow for the use of msleep.
```

NON-ATOMIC CONTEXT:

: You should use the [\*sleep\[\_range\]]{.title-ref} family of functions. There are a few more options here, while any of them may work correctly, using the \"right\" sleep function will help the scheduler, power management, and just make your driver better :)

> :您应该使用[\*sleep\[\_range\]]函数族。这里还有一些选项，虽然其中任何一个都可以正常工作，但使用“正确”睡眠功能将有助于调度程序、电源管理，并使您的驱动程序变得更好:)

```
\-- Backed by busy-wait loop:

> udelay(unsigned long usecs)

\-- Backed by hrtimers:

> usleep_range(unsigned long min, unsigned long max)

\-- Backed by jiffies / legacy_timers

> msleep(unsigned long msecs) msleep_interruptible(unsigned long msecs)

Unlike the [\*delay]{.title-ref} family, the underlying mechanism driving each of these calls varies, thus there are quirks you should be aware of.

SLEEPING FOR \"A FEW\" USECS ( \< \~10us? ):

:   -   Use udelay

    -

        Why not usleep?

        :   On slower systems, (embedded, OR perhaps a speed-stepped PC!) the overhead of setting up the hrtimers for usleep *may* not be worth it. Such an evaluation will obviously depend on your specific situation, but it is something to be aware of.

SLEEPING FOR \~USECS OR SMALL MSECS ( 10us - 20ms):

:   -   Use usleep_range

    -

        Why not msleep for (1ms - 20ms)?

        :

            Explained originally here:

            :   <https://lore.kernel.org/r/15327.1186166232@lwn.net>

            msleep(1\~20) may not do what the caller intends, and will often sleep longer (\~20 ms actual sleep for any value given in the 1\~20ms range). In many cases this is not the desired behavior.

    -

        Why is there no \"usleep\" / What is a good range?

        :   Since usleep_range is built on top of hrtimers, the wakeup will be very precise (ish), thus a simple usleep function would likely introduce a large number of undesired interrupts.

            With the introduction of a range, the scheduler is free to coalesce your wakeup with any other wakeup that may have happened for other reasons, or at the worst case, fire an interrupt for your upper bound.

            The larger a range you supply, the greater a chance that you will not trigger an interrupt; this should be balanced with what is an acceptable upper bound on delay / performance for your specific code path. Exact tolerances here are very situation specific, thus it is left to the caller to determine a reasonable range.

SLEEPING FOR LARGER MSECS ( 10ms+ )

:   -   Use msleep or possibly msleep_interruptible

    -

        What\'s the difference?

        :   msleep sets the current task to TASK_UNINTERRUPTIBLE whereas msleep_interruptible sets the current task to TASK_INTERRUPTIBLE before scheduling the sleep. In short, the difference is whether the sleep can be ended early by a signal. In general, just use msleep unless you know you have a need for the interruptible variant.

FLEXIBLE SLEEPING (any delay, uninterruptible)

:   -   Use fsleep
```
