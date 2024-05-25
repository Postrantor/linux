---
ttip: translate by baidu@2024-01-17 08:55:16
title: Scheduler Nice Design
---

This document explains the thinking about the revamped and streamlined nice-levels implementation in the new Linux scheduler.

> 本文档解释了在新的 Linux 调度器中改进和精简的漂亮级别实现的想法。

Nice levels were always pretty weak under Linux and people continuously pestered us to make nice +19 tasks use up much less CPU time.

> 在 Linux 下，好的级别总是很弱，人们不断纠缠我们，让我们完成好的+19 任务，占用更少的 CPU 时间。

Unfortunately that was not that easy to implement under the old scheduler, (otherwise we\'d have done it long ago) because nice level support was historically coupled to timeslice length, and timeslice units were driven by the HZ tick, so the smallest timeslice was 1/HZ.

> 不幸的是，在旧的调度器下，这并不是那么容易实现的（否则我们早就这么做了），因为从历史上看，良好的级别支持与时间片长度有关，而时间片单位是由 HZ 刻度驱动的，所以最小的时间片是 1/HZ。

In the O(1) scheduler (in 2003) we changed negative nice levels to be much stronger than they were before in 2.4 (and people were happy about that change), and we also intentionally calibrated the linear timeslice rule so that nice +19 level would be [exactly]() 1 jiffy. To better understand it, the timeslice graph went like this (cheesy ASCII art alert!):

> 在 O（1）调度器中（2003 年），我们将负尼斯水平更改为比 2.4 中的前一个更强（人们对此变化感到高兴），我们还有意校准了线性时间片规则，使尼斯+19 水平[恰好]（）1 jiffy。为了更好地理解它，时间片图是这样的（俗气的 ASCII 艺术警报！）：

```
    A
    \     | [timeslice length]
    \    |
    \   |
    \  |
    \ |
    \|___100msecs
    |^ . _
    |      ^ . _
    |            ^ . _
    -*----------------------------------*-----> [nice level]
    -20               |                +19
    |
    |
```

So that if someone wanted to really renice tasks, +19 would give a much bigger hit than the normal linear rule would do. (The solution of changing the ABI to extend priorities was discarded early on.)

> 因此，如果有人真的想放弃任务，+19 会比正常的线性规则带来更大的打击。（改变 ABI 以扩展优先级的解决方案很早就被放弃了。）

This approach worked to some degree for some time, but later on with HZ=1000 it caused 1 jiffy to be 1 msec, which meant 0.1% CPU usage which we felt to be a bit excessive. Excessive [not]() because it\'s too small of a CPU utilization, but because it causes too frequent (once per millisec) rescheduling. (and would thus trash the cache, etc. Remember, this was long ago when hardware was weaker and caches were smaller, and people were running number crunching apps at nice +19.)

> 这种方法在一定程度上有效了一段时间，但后来当 HZ=1000 时，它导致 1 瞬间变为 1 毫秒，这意味着 0.1%的 CPU 使用率，我们觉得这有点过高。过度[不是]（），因为它的 CPU 利用率太小，但它会导致过于频繁（每毫秒一次）的重新调度。（因此会破坏缓存等。记住，这是很久以前的事了，当时硬件更弱，缓存更小，人们以 nice+19 的速度运行数字运算应用程序。）

So for HZ=1000 we changed nice +19 to 5msecs, because that felt like the right minimal granularity - and this translates to 5% CPU utilization. But the fundamental HZ-sensitive property for nice+19 still remained, and we never got a single complaint about nice +19 being too [weak]() in terms of CPU utilization, we only got complaints about it (still) being too `[strong]() :-)`

> 因此，对于 HZ=1000，我们将 nice+19 更改为 5msecs，因为这感觉是正确的最小粒度-这转化为 5%的 CPU 利用率。但 nice+19 的基本 HZ 敏感特性仍然存在，我们从未收到过任何关于 nice+119 在 CPU 利用率方面太[弱]（）的投诉，我们只收到过关于它（仍然）太[强]（）：-）的投诉`

To sum it up: we always wanted to make nice levels more consistent, but within the constraints of HZ and jiffies and their nasty design level coupling to timeslices and granularity it was not really viable.

> 总之：我们一直想让好的级别更加一致，但在 HZ 和 jiffies 的限制下，以及它们与时间片和粒度的恶劣设计级别耦合，这是不可行的。

The second (less frequent but still periodically occurring) complaint about Linux\'s nice level support was its asymmetry around the origin (which you can see demonstrated in the picture above), or more accurately: the fact that nice level behavior depended on the `[absolute]()` nice level as well, while the nice API itself is fundamentally "relative":

> 关于 Linux 良好级别支持的第二个（不太常见，但仍会周期性地出现）抱怨是它在原点周围的不对称性（你可以在上面的图片中看到），或者更准确地说：良好级别的行为也依赖于`[绝对]（）`良好级别，而良好的 API 本身从根本上是“相对的”：

```
> int nice(int inc);
>
> asmlinkage long sys_nice(int increment)
```

(the first one is the glibc API, the second one is the syscall API.) Note that the `inc` is relative to the current nice level. Tools like bash\'s "nice" command mirror this relative API.

> （第一个是 glibc API，第二个是 syscall API。）请注意，“inc”是相对于当前的 nice 级别的。像 bash 的“nice”命令这样的工具镜像了这个相对的 API。

With the old scheduler, if you for example started a niced task with +1 and another task with +2, the CPU split between the two tasks would depend on the nice level of the parent shell - if it was at nice -10 the CPU split was different than if it was at +5 or +10.

> 使用旧的调度程序，例如，如果您用+1 启动一个漂亮的任务，用+2 启动另一个任务，则两个任务之间的 CPU 分配将取决于父 shell 的漂亮级别-如果是漂亮的-10，则 CPU 分配与+5 或+10 不同。

A third complaint against Linux\'s nice level support was that negative nice levels were not `punchy enough\', so lots of people had to resort to run audio (and other multimedia) apps under RT priorities such as SCHED_FIFO. But this caused other problems: SCHED_FIFO is not starvation proof, and a buggy SCHED_FIFO app can also lock up the system for good.

> 对 Linux 良好级别支持的第三个抱怨是，负良好级别不够“有力”，因此许多人不得不求助于在 RT 优先级下运行音频（和其他多媒体）应用程序，如 SCHED_FIFO。但这也引发了其他问题：SCHED_FIFO 不能防止饥饿，有缺陷的 SCHED_FFIFO 应用程序也可以永远锁定系统。

The new scheduler in v2.6.23 addresses all three types of complaints:

To address the first complaint (of nice levels being not "punchy" enough), the scheduler was decoupled from `time slice` and HZ concepts (and granularity was made a separate concept from nice levels) and thus it was possible to implement better and more consistent nice +19 support: with the new scheduler nice +19 tasks get a HZ-independent 1.5%, instead of the variable 3%-5%-9% range they got in the old scheduler.

> 为了解决第一个抱怨（尼斯级别不够“有力”），调度器与“时间片”和 HZ 概念解耦（粒度与尼斯级别是一个独立的概念），因此可以实现更好、更一致的尼斯+19 支持：使用新的调度器尼斯+19 任务获得独立于 HZ 的 1.5%，而不是他们在旧调度程序中得到的变量 3%-5%-9%的范围。

To address the second complaint (of nice levels not being consistent), the new scheduler makes `nice(1)` have the same CPU utilization effect on tasks, regardless of their absolute nice levels. So on the new scheduler, running a nice +10 and a nice 11 task has the same CPU utilization "split" between them as running a nice -5 and a nice -4 task. (one will get 55% of the CPU, the other 45%.) That is why nice levels were changed to be "multiplicative" (or exponential) - that way it does not matter which nice level you start out from, the `relative result` will always be the same.

> 为了解决第二个抱怨（好级别不一致），新的调度器使“好（1）”对任务具有相同的 CPU 利用率影响，而不管它们的绝对好级别如何。因此，在新的调度器上，运行一个漂亮的+10 和漂亮的 11 任务与运行一个好看的-5 和漂亮的-4 任务具有相同的 CPU 利用率“分割”。（一个将获得 55%的 CPU，另一个获得 45%的 CPU。）这就是为什么漂亮的级别被改为“乘法”（或指数）——这样，无论你从哪个漂亮的级别开始，“相对结果”总是一样的。

The third complaint (of negative nice levels not being "punchy" enough and forcing audio apps to run under the more dangerous `SCHED_FIFO` scheduling policy) is addressed by the new scheduler almost automatically: stronger negative nice levels are an automatic side-effect of the recalibrated dynamic range of nice levels.

> 第三个抱怨（负面好水平不够“有力”，迫使音频应用程序在更危险的“SCHED_FIFO”调度策略下运行）几乎是由新的调度器自动解决的：更强的负面好水平是重新校准的好水平动态范围的自动副作用。
