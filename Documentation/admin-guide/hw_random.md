---
tip: translate by baidu@2024-01-30 21:34:19
---
---
title: Hardware random number generators
---

# Introduction


The hw_random framework is software that makes use of a special hardware feature on your CPU or motherboard, a Random Number Generator (RNG). The software has two parts: a core providing the /dev/hwrng character device and its sysfs support, plus a hardware-specific driver that plugs into that core.

> hw_random框架是利用CPU或主板上的一个特殊硬件功能，即随机数生成器（RNG）的软件。该软件有两个部分：一个核心提供/dev/hwrng字符设备及其sysfs支持，再加上一个插入该核心的硬件特定驱动程序。


To make the most effective use of these mechanisms, you should download the support software as well. Download the latest version of the \"rng-tools\" package from the hw_random driver\'s official Web site:

> 为了最有效地利用这些机制，您还应该下载支持软件。从hw_random驱动程序的官方网站下载“rng tools”包的最新版本：

> <http://sourceforge.net/projects/gkernel/>


Those tools use /dev/hwrng to fill the kernel entropy pool, which is used internally and exported by the /dev/urandom and /dev/random special files.

> 这些工具使用/dev/hwrng来填充内核熵池，内核熵池在内部使用，并由/dev/urandom和/dev/random特殊文件导出。

# Theory of operation


CHARACTER DEVICE. Using the standard open() and read() system calls, you can read random data from the hardware RNG device. This data is NOT CHECKED by any fitness tests, and could potentially be bogus (if the hardware is faulty or has been tampered with). Data is only output if the hardware \"has-data\" flag is set, but nevertheless a security-conscious person would run fitness tests on the data before assuming it is truly random.

> 字符设备。使用标准的open（）和read（）系统调用，您可以从硬件RNG设备读取随机数据。这些数据没有经过任何适应性测试的检查，可能是伪造的（如果硬件有故障或被篡改）。只有当设置了硬件“有数据”标志时，才会输出数据，但尽管如此，有安全意识的人会在假设数据是真正随机的之前对数据进行适应性测试。


The rng-tools package uses such tests in \"rngd\", and lets you run them by hand with a \"rngtest\" utility.

> rng工具包在\“rngd\”中使用这样的测试，并允许您使用\“rngtest\”实用程序手动运行它们。

/dev/hwrng is char device major 10, minor 183.


CLASS DEVICE. There is a /sys/class/misc/hw_random node with two unique attributes, \"rng_available\" and \"rng_current\". The \"rng_available\" attribute lists the hardware-specific drivers available, while \"rng_current\" lists the one which is currently connected to /dev/hwrng. If your system has more than one RNG available, you may change the one used by writing a name from the list in \"rng_available\" into \"rng_current\".

> 类设备。有一个/sys/class/misc/hw_random节点，它具有两个唯一的属性“rng_available”和“rng_current”。“rng_available”属性列出了可用的硬件特定驱动程序，而“rng_current”列出了当前连接到/dev/hwrng的驱动程序。如果您的系统有多个可用的RNG，您可以通过将“RNG_available”中的列表中的名称写入“RNG_current”来更改所使用的RNG。

------------------------------------------------------------------------

Hardware driver for Intel/AMD/VIA Random Number Generators (RNG)

:   -   Copyright 2000,2001 Jeff Garzik \<<jgarzik@pobox.com>\>
    -   Copyright 2000,2001 Philipp Rumpf \<<prumpf@mandrakesoft.com>\>

# About the Intel RNG hardware, from the firmware hub datasheet


The Firmware Hub integrates a Random Number Generator (RNG) using thermal noise generated from inherently random quantum mechanical properties of silicon. When not generating new random bits the RNG circuitry will enter a low power state. Intel will provide a binary software driver to give third party software access to our RNG for use as a security feature. At this time, the RNG is only to be used with a system in an OS-present state.

> 固件集线器集成了一个随机数发生器（RNG），使用硅固有随机量子力学特性产生的热噪声。当不生成新的随机比特时，RNG电路将进入低功率状态。英特尔将提供一个二进制软件驱动程序，让第三方软件可以访问我们的RNG，用作安全功能。此时，RNG仅用于处于OS当前状态的系统。

# Intel RNG Driver notes

FIXME: support poll(2)

::: note
::: title
Note
:::

request_mem_region was removed, for three reasons:

1)  Only one RNG is supported by this driver;

2)  The location used by the RNG is a fixed location in MMIO-addressable memory;

> 2） RNG使用的位置是MMIO可寻址存储器中的固定位置；

3)  users with properly working BIOS e820 handling will always have the region in which the RNG is located reserved, so request_mem_region calls always fail for proper setups. However, for people who use mem=XX, BIOS e820 information is **not** in /proc/iomem, and request_mem_region(RNG_ADDR) can succeed.

> 3） 具有正常工作的BIOS e820处理的用户将始终保留RNG所在的区域，因此request_mem_region调用对于正确的设置总是失败。但是，对于使用mem=XX的人来说，/proc/iomem中的BIOS e820信息是**而不是**，并且request_mem_region（RNG_ADDR）可以成功。
:::

# Driver details

Based on:


:   Intel 82802AB/82802AC Firmware Hub (FWH) Datasheet May 1999 Order Number: 290658-002 R

> ：Intel 82802AB/88202AC固件集线器（FWH）产品介绍1999年5月订单号：290658-002 R

Intel 82802 Firmware Hub:


:   Random Number Generator Programmer\'s Reference Manual December 1999 Order Number: 298029-001 R

> ：随机数发生器编程器参考手册1999年12月订单号：298029-001 R

Intel 82802 Firmware HUB Random Number Generator Driver

:   Copyright (c) 2000 Matt Sottek \<<msottek@quiknet.com>\>


Special thanks to Matt Sottek. I did the \"guts\", he did the \"brains\" and all the testing.

> 特别感谢Matt Sottek。我做了“内脏”，他做了“大脑”和所有的测试。
