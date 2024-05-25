---
tip: translate by baidu@2024-05-25 19:54:19
title: High Precision Event Timer Driver for Linux
---

The High Precision Event Timer (HPET) hardware follows a specification by Intel and Microsoft, revision 1.

> 高精度事件定时器(HPET)硬件遵循 Intel 和 Microsoft 的规范(修订版 1)。

Each HPET has one fixed-rate counter (at 10+ MHz, hence \"High Precision\") and up to 32 comparators. Normally three or more comparators are provided, each of which can generate oneshot interrupts and at least one of which has additional hardware to support periodic interrupts. The comparators are also called \"timers\", which can be misleading since usually timers are independent of each other \... these share a counter, complicating resets.

> 每个 HPET 都有一个固定速率计数器(频率为 10+MHz，因此为“高精度”)和多达 32 个比较器。通常提供三个或多个比较器，每个比较器可以产生一个热中断，并且至少一个比较器具有额外的硬件来支持周期性中断。比较器也被称为“定时器”，这可能会产生误导，因为通常定时器是相互独立的... 这些共享一个计数器，使重置复杂化。

HPET devices can support two interrupt routing modes. In one mode, the comparators are additional interrupt sources with no particular system role. Many x86 BIOS writers don\'t route HPET interrupts at all, which prevents use of that mode. They support the other \"legacy replacement\" mode where the first two comparators block interrupts from 8254 timers and from the RTC.

> HPET 设备可以支持两种中断路由模式。在一种模式中，比较器是附加的中断源，没有特定的系统作用。许多 x86 BIOS 编写器根本不路由 HPET 中断，这阻止了该模式的使用。它们支持另一种“传统替换”模式，其中前两个比较器阻止来自 8254 定时器和 RTC 的中断。

The driver supports detection of HPET driver allocation and initialization of the HPET before the driver module_init routine is called. This enables platform code which uses timer 0 or 1 as the main timer to intercept HPET initialization. An example of this initialization can be found in arch/x86/kernel/hpet.c.

> 驱动程序支持在调用驱动程序 module_init 例程之前检测 HPET 驱动程序分配和初始化 HPET。这启用了使用计时器 0 或 1 作为主计时器来拦截 HPET 初始化的平台代码。这种初始化的一个例子可以在 arch/x86/kernel/hpet.c 中找到。

The driver provides a userspace API which resembles the API found in the RTC driver framework. An example user space program is provided in [file:samples/timers/hpet_example.c](file:samples/timers/hpet_example.c)

> 驱动程序提供了一个用户空间 API，类似于 RTC 驱动程序框架中的 API。[file:samples/timers/hpet_example.c](file:samples/timers/hpet-example.c)中提供了示例用户空间程序
