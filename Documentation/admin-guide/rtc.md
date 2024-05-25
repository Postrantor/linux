---
tip: translate by baidu@2024-01-30 21:39:40
---
---
title: Real Time Clock (RTC) Drivers for Linux
---


When Linux developers talk about a \"Real Time Clock\", they usually mean something that tracks wall clock time and is battery backed so that it works even with system power off. Such clocks will normally not track the local time zone or daylight savings time \-- unless they dual boot with MS-Windows \-- but will instead be set to Coordinated Universal Time (UTC, formerly \"Greenwich Mean Time\").

> 当Linux开发人员谈论“实时时钟”时，他们通常指的是跟踪墙上时钟时间并有电池支持的东西，这样即使在系统断电的情况下也能工作。这种时钟通常不会跟踪本地时区或夏令时，除非它们与MS Windows双重启动，而是设置为协调世界时（UTC，以前的“格林尼治标准时间”）。


The newest non-PC hardware tends to just count seconds, like the time(2) system call reports, but RTCs also very commonly represent time using the Gregorian calendar and 24 hour time, as reported by gmtime(3).

> 最新的非PC硬件往往只计算秒数，就像时间（2）系统调用报告一样，但RTC通常也使用格里高利日历和24小时时间来表示时间，正如gmtime（3）所报告的那样。


Linux has two largely-compatible userspace RTC API families you may need to know about:

> Linux有两个大型兼容用户空间RTC API系列，您可能需要了解：

> \* /dev/rtc \... is the RTC provided by PC compatible systems, so it\'s not very portable to non-x86 systems.
>
> \* /dev/rtc0, /dev/rtc1 \... are part of a framework that\'s supported by a wide variety of RTC chips on all systems.


Programmers need to understand that the PC/AT functionality is not always available, and some systems can do much more. That is, the RTCs use the same API to make requests in both RTC frameworks (using different filenames of course), but the hardware may not offer the same functionality. For example, not every RTC is hooked up to an IRQ, so they can\'t all issue alarms; and where standard PC RTCs can only issue an alarm up to 24 hours in the future, other hardware may be able to schedule one any time in the upcoming century.

> 程序员需要明白，PC/AT功能并不总是可用的，有些系统可以做得更多。也就是说，RTC使用相同的API在两个RTC框架中进行请求（当然使用不同的文件名），但硬件可能不提供相同的功能。例如，并不是每个RTC都连接到IRQ，所以它们不能全部发出警报；并且在标准PC RTC在未来最多只能发出24小时的警报的情况下，其他硬件可能能够在下个世纪的任何时间调度警报。

# Old PC/AT-Compatible driver: /dev/rtc


All PCs (even Alpha machines) have a Real Time Clock built into them. Usually they are built into the chipset of the computer, but some may actually have a Motorola MC146818 (or clone) on the board. This is the clock that keeps the date and time while your computer is turned off.

> 所有的电脑（甚至阿尔法机器）都内置了实时时钟。通常，它们内置在计算机的芯片组中，但有些芯片板上可能实际上有摩托罗拉MC146818（或克隆）。这是一个在电脑关机时保持日期和时间的时钟。


ACPI has standardized that MC146818 functionality, and extended it in a few ways (enabling longer alarm periods, and wake-from-hibernate). That functionality is NOT exposed in the old driver.

> ACPI已经标准化了MC146818的功能，并以几种方式对其进行了扩展（允许更长的警报周期和从休眠中唤醒）。该功能未在旧驱动程序中公开。


However it can also be used to generate signals from a slow 2Hz to a relatively fast 8192Hz, in increments of powers of two. These signals are reported by interrupt number 8. (Oh! So *that* is what IRQ 8 is for\...) It can also function as a 24hr alarm, raising IRQ 8 when the alarm goes off. The alarm can also be programmed to only check any subset of the three programmable values, meaning that it could be set to ring on the 30th second of the 30th minute of every hour, for example. The clock can also be set to generate an interrupt upon every clock update, thus generating a 1Hz signal.

> 然而，它也可以用于以2的幂为增量生成从慢速2Hz到相对快速8192Hz的信号。这些信号由中断编号8报告。（哦！所以*这就是IRQ 8的含义\…）它也可以作为24小时警报，在警报响起时发出IRQ 8。警报也可以编程为只检查三个可编程值的任何子集，这意味着它可以设置为在每小时第30分钟的第30秒响起。时钟也可以设置为在每次时钟更新时产生中断，从而产生1Hz信号。


The interrupts are reported via /dev/rtc (major 10, minor 135, read only character device) in the form of an unsigned long. The low byte contains the type of interrupt (update-done, alarm-rang, or periodic) that was raised, and the remaining bytes contain the number of interrupts since the last read. Status information is reported through the pseudo-file /proc/driver/rtc if the /proc filesystem was enabled. The driver has built in locking so that only one process is allowed to have the /dev/rtc interface open at a time.

> 中断通过/dev/rtc（主要10，次要135，只读字符设备）以无符号长的形式报告。低位字节包含引发的中断类型（更新完成、警报响起或周期性），其余字节包含自上次读取以来的中断数。如果/proc文件系统已启用，则通过伪文件/proc/driver/rtc报告状态信息。驱动程序内置了锁定，因此一次只允许一个进程打开/dev/rtc接口。


A user process can monitor these interrupts by doing a read(2) or a select(2) on /dev/rtc \-- either will block/stop the user process until the next interrupt is received. This is useful for things like reasonably high frequency data acquisition where one doesn\'t want to burn up 100% CPU by polling gettimeofday etc. etc.

> 用户进程可以通过在/dev/rtc上执行读取（2）或选择（2）来监视这些中断，这两种操作都将阻止/停止用户进程，直到接收到下一个中断。这对于相当高频率的数据采集很有用，因为人们不想通过轮询gettimeofday等来消耗100%的CPU。


At high frequencies, or under high loads, the user process should check the number of interrupts received since the last read to determine if there has been any interrupt \"pileup\" so to speak. Just for reference, a typical 486-33 running a tight read loop on /dev/rtc will start to suffer occasional interrupt pileup (i.e. \> 1 IRQ event since last read) for frequencies above 1024Hz. So you really should check the high bytes of the value you read, especially at frequencies above that of the normal timer interrupt, which is 100Hz.

> 在高频或高负载下，用户进程应检查自上次读取以来接收到的中断数量，以确定是否存在任何中断“堆积”。仅供参考，对于1024Hz以上的频率，在/dev/rtc上运行紧密读取循环的典型486-33将开始遭受偶尔的中断堆积（即，自上次读取以来发生1次IRQ事件）。所以你真的应该检查你读取的值的高字节，尤其是在高于正常定时器中断（100Hz）的频率下。


Programming and/or enabling interrupt frequencies greater than 64Hz is only allowed by root. This is perhaps a bit conservative, but we don\'t want an evil user generating lots of IRQs on a slow 386sx-16, where it might have a negative impact on performance. This 64Hz limit can be changed by writing a different value to /proc/sys/dev/rtc/max-user-freq. Note that the interrupt handler is only a few lines of code to minimize any possibility of this effect.

> 只有root用户才允许编程和/或启用大于64Hz的中断频率。这可能有点保守，但我们不希望邪恶的用户在缓慢的386sx-16上生成大量IRQ，这可能会对性能产生负面影响。可以通过向/proc/sys/dev/rtc/max-user-freq写入不同的值来更改此64Hz限制。请注意，中断处理程序只有几行代码，可以最大限度地减少这种影响的可能性。


Also, if the kernel time is synchronized with an external source, the kernel will write the time back to the CMOS clock every 11 minutes. In the process of doing this, the kernel briefly turns off RTC periodic interrupts, so be aware of this if you are doing serious work. If you don\'t synchronize the kernel time with an external source (via ntp or whatever) then the kernel will keep its hands off the RTC, allowing you exclusive access to the device for your applications.

> 此外，如果内核时间与外部源同步，内核将每11分钟将时间写回CMOS时钟。在执行此操作的过程中，内核会短暂关闭RTC周期性中断，因此如果您正在进行严肃的工作，请注意这一点。如果您不将内核时间与外部源同步（通过ntp或其他方式），那么内核将不干涉RTC，允许您为应用程序独占访问设备。


The alarm and/or interrupt frequency are programmed into the RTC via various ioctl(2) calls as listed in ./include/linux/rtc.h Rather than write 50 pages describing the ioctl() and so on, it is perhaps more useful to include a small test program that demonstrates how to use them, and demonstrates the features of the driver. This is probably a lot more useful to people interested in writing applications that will be using this driver. See the code at the end of this document.

> 警报和/或中断频率通过中列出的各种ioctl（2）调用编程到RTC中/include/linux/rtc.h与其写50页描述ioctl（）等等，不如包含一个小的测试程序来演示如何使用它们，并演示驱动程序的功能。对于那些对编写将使用此驱动程序的应用程序感兴趣的人来说，这可能更有用。请参阅本文档末尾的代码。

(The original /dev/rtc driver was written by Paul Gortmaker.)

# New portable \"RTC Class\" drivers: /dev/rtcN


Because Linux supports many non-ACPI and non-PC platforms, some of which have more than one RTC style clock, it needed a more portable solution than expecting a single battery-backed MC146818 clone on every system. Accordingly, a new \"RTC Class\" framework has been defined. It offers three different userspace interfaces:

> 因为Linux支持许多非ACPI和非PC平台，其中一些平台有不止一个RTC式时钟，所以它需要一个更便携的解决方案，而不是期望在每个系统上都有一个电池支持的MC146818克隆。因此，定义了一个新的“RTC类”框架。它提供了三种不同的用户空间界面：

> -   /dev/rtcN \... much the same as the older /dev/rtc interface
>
> \* /sys/class/rtc/rtcN \... sysfs attributes support readonly access to some RTC attributes.
>
> \* /proc/driver/rtc \... the system clock RTC may expose itself using a procfs interface. If there is no RTC for the system clock, rtc0 is used by default. More information is (currently) shown here than through sysfs.


The RTC Class framework supports a wide variety of RTCs, ranging from those integrated into embeddable system-on-chip (SOC) processors to discrete chips using I2C, SPI, or some other bus to communicate with the host CPU. There\'s even support for PC-style RTCs \... including the features exposed on newer PCs through ACPI.

> RTC类框架支持多种RTC，从集成到可嵌入片上系统（SOC）处理器的RTC到使用I2C、SPI或某些其他总线与主机CPU通信的离散芯片。甚至支持PC风格的RTC。。。包括通过ACPI在较新的PC上公开的功能。


The new framework also removes the \"one RTC per system\" restriction. For example, maybe the low-power battery-backed RTC is a discrete I2C chip, but a high functionality RTC is integrated into the SOC. That system might read the system clock from the discrete RTC, but use the integrated one for all other tasks, because of its greater functionality.

> 新框架还取消了“每个系统一个RTC”的限制。例如，低功率电池支持的RTC可能是一个分立的I2C芯片，但高功能RTC集成到SOC中。该系统可能从分立的RTC读取系统时钟，但由于其更大的功能，将集成的RTC用于所有其他任务。


Check out tools/testing/selftests/rtc/rtctest.c for an example usage of the ioctl interface.

> 有关ioctl接口的示例用法，请查看tools/testing/selftests/rtc/rtctest.c。
