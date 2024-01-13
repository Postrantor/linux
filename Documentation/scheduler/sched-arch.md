---
tip: translate by baidu@2024-01-11 23:03:17
title: CPU Scheduler implementation hints for architecture specific code
---

> Nick Piggin, 2005

# Context switch

1. Runqueue locking By default, the switch_to arch function is called with the runqueue locked. This is usually not a problem unless switch_to may need to take the runqueue lock. This is usually due to a wake up operation in the context switch.

> 1.运行队列锁定默认情况下，在运行队列锁定的情况下调用 switch_to-arch 函数。这通常不是问题，除非 switch_to 可能需要获取 runqueue 锁。这通常是由于上下文切换中的唤醒操作造成的。

To request the scheduler call switch_to with the runqueue unlocked, you must [`#define __ARCH_WANT_UNLOCKED_CTXSW`] in a header file (typically the one where switch_to is defined).

> 要在运行队列未锁定的情况下请求调度程序调用 switch_To，必须在头文件（通常是定义 switch_To 的文件）中[`#define __ARCH_WANT_unlocked_CTXSW`]。

Unlocked context switches introduce only a very minor performance penalty to the core scheduler implementation in the CONFIG_SMP case.

> 在 CONFIG_SMP 的情况下，未锁定的上下文开关只会给核心调度程序实现带来非常小的性能损失。

# CPU idle

Your cpu_idle routines need to obey the following rules:

1.  Preempt should now disabled over idle routines. Should only be enabled to call schedule() then disabled again.

> 1.现在应该在空闲例程上禁用抢占。应仅启用以调用 schedule（），然后再次禁用。

2.  `need_resched/TIF_NEED_RESCHED` is only ever set, and will never be cleared until the running task has called schedule(). Idle threads need only ever query need_resched, and may never set or clear it.

> 2.“need_sched/TIF_need_resched”仅被设置，并且在运行的任务调用 schedule（）之前永远不会被清除。空闲线程只需要查询 need_sched，并且可能永远不会设置或清除它。

3.  When cpu_idle finds (`need_resched() == \'true\'`), it should call schedule(). It should not call schedule() otherwise.

> 3.当 cpu_idle 找到（`need_sched（）==\'true\'`）时，它应该调用 schedule（）。否则不应调用 schedule（）。

4.  The only time interrupts need to be disabled when checking need_resched is if we are about to sleep the processor until the next interrupt (this doesn\'t provide any protection of need_resched, it prevents losing an interrupt):

> 4.在检查 need_sched 时，唯一需要禁用中断的时间是，如果我们即将休眠处理器直到下一次中断（这不会提供 need_schedd 的任何保护，它可以防止丢失中断）：

4a. Common problem with this type of sleep appears to be:

```c
local_irq_disable();
if (!need_resched()) {
        local_irq_enable();
        *** resched interrupt arrives here ***
        __asm__("sleep until next interrupt");
}
```

1.  TIF_POLLING_NRFLAG can be set by idle routines that do not need an interrupt to wake them up when need_resched goes high. In other words, they must be periodically polling need_resched, although it may be reasonable to do some background work or enter a low CPU priority.

> 1.TIF_POLLING_NRFLAG 可以由空闲例程设置，当 need_sched 变高时，这些例程不需要中断来唤醒它们。换句话说，他们必须定期轮询 need_sched，尽管做一些后台工作或输入低 CPU 优先级可能是合理的。

5a. If TIF_POLLING_NRFLAG is set, and we do decide to enter an interrupt sleep, it needs to be cleared then a memory barrier issued (followed by a test of need_resched with interrupts disabled, as explained in 3).

> 5a。如果设置了 TIF_POLLING_NRFLAG，并且我们确实决定进入中断睡眠，则需要清除它，然后发出内存屏障（然后在禁用中断的情况下测试 need_scheded，如 3 所述）。

arch/x86/kernel/process.c has examples of both polling and sleeping idle functions.

> arch/x86/kernel/process.c 提供了轮询和休眠空闲函数的示例。

# Possible arch/ problems

Possible arch problems I found (and either tried to fix or didn\'t):

sparc - IRQs on at this point(?), change local_irq_save to \_disable.

: - TODO: needs secondary CPUs to disable preempt (See #1)
