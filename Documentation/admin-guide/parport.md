---
tip: translate by baidu@2024-01-30 21:36:21
---
---
title: Parport
---


The `parport` code provides parallel-port support under Linux. This includes the ability to share one port between multiple device drivers.

> “parport”代码在Linux下提供并行端口支持。这包括在多个设备驱动程序之间共享一个端口的能力。


You can pass parameters to the `parport` code to override its automatic detection of your hardware. This is particularly useful if you want to use IRQs, since in general these can\'t be autoprobed successfully. By default IRQs are not used even if they **can** be probed. This is because there are a lot of people using the same IRQ for their parallel port and a sound card or network card.

> 您可以将参数传递给“parport”代码，以覆盖其对硬件的自动检测。如果您想使用IRQ，这一点尤其有用，因为通常情况下，这些IRQ无法成功地进行自动探测。默认情况下，即使可以**探测IRQ，也不会使用IRQ。这是因为有很多人使用相同的IRQ作为并行端口和声卡或网卡。


The `parport` code is split into two parts: generic (which deals with port-sharing) and architecture-dependent (which deals with actually using the port).

> “parport”代码分为两部分：通用（处理端口共享）和依赖体系结构（处理实际使用端口）。

# Parport as modules

If you load the [parport]{.title-ref}\` code as a module, say:

    # insmod parport


to load the generic `parport` code. You then must load the architecture-dependent code with (for example):

> 以加载通用的“parport”代码。然后，您必须加载依赖于体系结构的代码（例如）：

    # insmod parport_pc io=0x3bc,0x378,0x278 irq=none,7,auto


to tell the `parport` code that you want three PC-style ports, one at 0x3bc with no IRQ, one at 0x378 using IRQ 7, and one at 0x278 with an auto-detected IRQ. Currently, PC-style (`parport_pc`), Sun `bpp`, Amiga, Atari, and MFC3 hardware is supported.

> 告诉“parport”代码您想要三个PC风格的端口，一个位于0x3bc，没有IRQ，一个处于0x378，使用IRQ 7，一个在0x278，具有自动检测的IRQ。目前，支持PC风格（“parport_PC”）、Sun“bpp”、Amiga、Atari和MFC3硬件。


PCI parallel I/O card support comes from `parport_pc`. Base I/O addresses should not be specified for supported PCI cards since they are automatically detected.

> PCI并行I/O卡支持来自“parport_pc”。不应为支持的PCI卡指定基本I/O地址，因为它们是自动检测到的。

## modprobe


If you use modprobe , you will find it useful to add lines as below to a configuration file in /etc/modprobe.d/ directory:

> 如果您使用modprobe，您会发现在/etc/modprobe.d/目录中的配置文件中添加以下行非常有用：

    alias parport_lowlevel parport_pc
    options parport_pc io=0x378,0x278 irq=7,auto


modprobe will load `parport_pc` (with the options `io=0x378,0x278 irq=7,auto`) whenever a parallel port device driver (such as `lp`) is loaded.

> modprobe将在加载并行端口设备驱动程序（如“lp”）时加载“parport_pc”（选项为“io=0x378，x278 irq=7，auto”）。


Note that these are example lines only! You shouldn\'t in general need to specify any options to `parport_pc` in order to be able to use a parallel port.

> 请注意，这些只是示例行！通常，您不需要为“parport_pc”指定任何选项即可使用并行端口。

## Parport probe \[optional\]


In 2.2 kernels there was a module called `parport_probe`, which was used for collecting IEEE 1284 device ID information. This has now been enhanced and now lives with the IEEE 1284 support. When a parallel port is detected, the devices that are connected to it are analysed, and information is logged like this:

> 在2.2内核中，有一个名为“parport_probe”的模块，用于收集IEEE 1284设备ID信息。这一点现在已经得到了增强，现在有了IEEE 1284的支持。当检测到并行端口时，将分析连接到该端口的设备，并按如下方式记录信息：

    parport0: Printer, BJC-210 (Canon)


The probe information is available from files in `/proc/sys/dev/parport/`.

> 探测信息可从“/proc/sys/dev/parport/”中的文件中获得。

# Parport linked into the kernel statically


If you compile the `parport` code into the kernel, then you can use kernel boot parameters to get the same effect. Add something like the following to your LILO command line:

> 如果将“parport”代码编译到内核中，则可以使用内核引导参数来获得相同的效果。在LILO命令行中添加以下内容：

    parport=0x3bc parport=0x378,7 parport=0x278,auto,nofifo


You can have many `parport=...` statements, one for each port you want to add. Adding `parport=0` to the kernel command-line will disable parport support entirely. Adding `parport=auto` to the kernel command-line will make `parport` use any IRQ lines or DMA channels that it auto-detects.

> 你可以有很多`parport=…`语句，每个要添加的端口一个。将“parport=0”添加到内核命令行将完全禁用parport支持。将“parport=auto”添加到内核命令行将使“parport”使用它自动检测到的任何IRQ行或DMA通道。

# Files in /proc


If you have configured the `/proc` filesystem into your kernel, you will see a new directory entry: `/proc/sys/dev/parport`. In there will be a directory entry for each parallel port for which parport is configured. In each of those directories are a collection of files describing that parallel port.

> 如果在内核中配置了“/proc”文件系统，您将看到一个新的目录条目：“/proc/sys/dev/parport”。在中，每个配置了parport的并行端口都将有一个目录条目。在每个目录中都有一组描述该并行端口的文件。

The `/proc/sys/dev/parport` directory tree looks like:

    parport
    |-- default
    |   |-- spintime
    |   `-- timeslice
    |-- parport0
    |   |-- autoprobe
    |   |-- autoprobe0
    |   |-- autoprobe1
    |   |-- autoprobe2
    |   |-- autoprobe3
    |   |-- devices
    |   |   |-- active
    |   |   `-- lp
    |   |       `-- timeslice
    |   |-- base-addr
    |   |-- irq
    |   |-- dma
    |   |-- modes
    |   `-- spintime
    `-- parport1
    |-- autoprobe
    |-- autoprobe0
    |-- autoprobe1
    |-- autoprobe2
    |-- autoprobe3
    |-- devices
    |   |-- active
    |   `-- ppa
    |       `-- timeslice
    |-- base-addr
    |-- irq
    |-- dma
    |-- modes
    `-- spintime

::: tabularcolumns
p{13.5cm}\|
:::

+-----------------------------+-----------------------------------------------------+
| File Contents               |                                                     |
+=============================+=====================================================+
| `devices/active` A li       | st of the device drivers using that port. A \"+\"   |
+-----------------------------+-----------------------------------------------------+
| > will appear               | by the name of the device currently using           |
+-----------------------------+-----------------------------------------------------+
| > the port (it              | > might not appear against any). The                |
+-----------------------------+-----------------------------------------------------+
| > string \"none             | \" means that there are no device drivers           |
+-----------------------------+-----------------------------------------------------+
| > using that p              | ort.                                                |
+-----------------------------+-----------------------------------------------------+
| `base-addr` Para            | llel port\'s base address, or addresses if the port |
+-----------------------------+-----------------------------------------------------+
| > has more tha              | n one in which case they are separated              |
+-----------------------------+-----------------------------------------------------+
| > with tabs.                | These values might not have any sensible            |
+-----------------------------+-----------------------------------------------------+
| > meaning for               | some ports.                                         |
+-----------------------------+-----------------------------------------------------+
| `irq` Parallel              | > port\'s IRQ, or -1 if none is being used.         |
+-----------------------------+-----------------------------------------------------+
| `dma` Parallel              | > port\'s DMA channel, or -1 if none is being       |
|                             |                                                     |
| :   used.                   |                                                     |
+-----------------------------+-----------------------------------------------------+
| `modes` Parallel            | > port\'s hardware modes, comma-separated,          |
|                             |                                                     |
| :   meaning:                |                                                     |
|                             |                                                     |
|     -   PCSPP               |                                                     |
+-----------------------------+-----------------------------------------------------+
| > PC-style                  | > SPP registers are available.                      |
| >                           |                                                     |
| > -   TRISTATE              |                                                     |
+-----------------------------+-----------------------------------------------------+
| > Port is                   | bidirectional.                                      |
| >                           |                                                     |
| > -   COMPAT                |                                                     |
+-----------------------------+-----------------------------------------------------+
| > Hardware                  | > acceleration for printers is                      |
+-----------------------------+-----------------------------------------------------+
| > availabl                  | e and will be used.                                 |
| >                           |                                                     |
| > -   EPP                   |                                                     |
+-----------------------------+-----------------------------------------------------+
| > Hardware                  | > acceleration for EPP protocol                     |
+-----------------------------+-----------------------------------------------------+
| > is avail                  | able and will be used.                              |
| >                           |                                                     |
| > -   ECP                   |                                                     |
+-----------------------------+-----------------------------------------------------+
| > Hardware                  | > acceleration for ECP protocol                     |
+-----------------------------+-----------------------------------------------------+
| > is avail                  | able and will be used.                              |
| >                           |                                                     |
| > -   DMA                   |                                                     |
+-----------------------------+-----------------------------------------------------+
| > DMA is a                  | vailable and will be used.                          |
+-----------------------------+-----------------------------------------------------+
| > Note that th              | e current implementation will only take             |
+-----------------------------+-----------------------------------------------------+
| > advantage of line to use. | > COMPAT and ECP modes if it has an IRQ             |
+-----------------------------+-----------------------------------------------------+
| `autoprobe` Any             | IEEE-1284 device ID information that has been       |
+-----------------------------+-----------------------------------------------------+
| > acquired fro              | m the (non-IEEE 1284.3) device.                     |
+-----------------------------+-----------------------------------------------------+
| `autoprobe[0-3]` IEEE       | > 1284 device ID information retrieved from         |
+-----------------------------+-----------------------------------------------------+
| > daisy-chain               | devices that conform to IEEE 1284.3.                |
+-----------------------------+-----------------------------------------------------+
| `spintime` The              | number of microseconds to busy-loop while waiting   |
+-----------------------------+-----------------------------------------------------+
| > for the peri              | pheral to respond. You might find that              |
+-----------------------------+-----------------------------------------------------+
| > adjusting th              | is improves performance, depending on your          |
+-----------------------------+-----------------------------------------------------+
| > peripherals.              | > This is a port-wide setting, i.e. it              |
+-----------------------------+-----------------------------------------------------+
| > applies to a              | ll devices on a particular port.                    |
+-----------------------------+-----------------------------------------------------+
| `timeslice` The             | number of milliseconds that a device driver is      |
+-----------------------------+-----------------------------------------------------+
| > allowed to k              | eep a port claimed for. This is advisory,           |
+-----------------------------+-----------------------------------------------------+
| > and driver c              | an ignore it if it must.                            |
+-----------------------------+-----------------------------------------------------+
| `default/*` The             | defaults for spintime and timeslice. When a new     |
+-----------------------------+-----------------------------------------------------+
| > port is regi              | stered, it picks up the default spintime.           |
+-----------------------------+-----------------------------------------------------+
| > When a new d              | evice is registered, it picks up the                |
+-----------------------------+-----------------------------------------------------+
| > default time              | slice.                                              |
+-----------------------------+-----------------------------------------------------+

# Device drivers


Once the parport code is initialised, you can attach device drivers to specific ports. Normally this happens automatically; if the lp driver is loaded it will create one lp device for each port found. You can override this, though, by using parameters either when you load the lp driver:

> 一旦parport代码初始化，您就可以将设备驱动程序连接到特定的端口。通常情况下，这是自动发生的；如果加载了lp驱动程序，它将为找到的每个端口创建一个lp设备。不过，您可以在加载lp驱动程序时使用参数来覆盖此项：

    # insmod lp parport=0,2

or on the LILO command line:

    lp=parport0 lp=parport2


Both the above examples would inform lp that you want `/dev/lp0` to be the first parallel port, and /dev/lp1 to be the **third** parallel port, with no lp device associated with the second port (parport1). Note that this is different to the way older kernels worked; there used to be a static association between the I/O port address and the device name, so `/dev/lp0` was always the port at 0x3bc. This is no longer the case - if you only have one port, it will default to being `/dev/lp0`, regardless of base address.

> 上面的两个例子都会通知lp，您希望“/dev/lp0”是第一个并行端口，/dev/lp1是**第三个**并行端口，并且没有与第二个端口（parport1）相关联的lp设备。请注意，这与旧内核的工作方式不同；I/O端口地址和设备名称之间过去存在静态关联，因此“/dev/lp0”始终是0x3bc处的端口。现在不再是这种情况了——如果您只有一个端口，那么无论基地址如何，它都将默认为“/dev/lp0”。

Also:

> -   If you selected the IEEE 1284 support at compile time, you can say `lp=auto` on the kernel command line, and lp will create devices only for those ports that seem to have printers attached.
> -   If you give PLIP the `timid` parameter, either with `plip=timid` on the command line, or with `insmod plip timid=1` when using modules, it will avoid any ports that seem to be in use by other devices.
> -   IRQ autoprobing works only for a few port types at the moment.

# Reporting printer problems with parport


If you are having problems printing, please go through these steps to try to narrow down where the problem area is.

> 如果您在打印时遇到问题，请执行以下步骤，尽量缩小问题区域的范围。


When reporting problems with parport, really you need to give all of the messages that `parport_pc` spits out when it initialises. There are several code paths:

> 当报告parport的问题时，实际上您需要给出“parport_pc”在初始化时发出的所有消息。有几个代码路径：

-   polling
-   interrupt-driven, protocol in software
-   interrupt-driven, protocol in hardware using PIO
-   interrupt-driven, protocol in hardware using DMA


The kernel messages that `parport_pc` logs give an indication of which code path is being used. (They could be a lot better actually..)

> “parport_pc”日志中的内核消息指示正在使用的代码路径。（事实上，它们可能会好得多。）


For normal printer protocol, having IEEE 1284 modes enabled or not should not make a difference.

> 对于正常的打印机协议，启用或不启用IEEE 1284模式应该没有什么区别。


To turn off the \'protocol in hardware\' code paths, disable `CONFIG_PARPORT_PC_FIFO`. Note that when they are enabled they are not necessarily **used**; it depends on whether the hardware is available, enabled by the BIOS, and detected by the driver.

> 要关闭“硬件中的协议”代码路径，请禁用“CONFIG_PARPORT_PC_FIFO”。请注意，当它们被启用时，它们不一定**被使用**；这取决于硬件是否可用、是否由BIOS启用以及是否由驱动程序检测到。


So, to start with, disable `CONFIG_PARPORT_PC_FIFO`, and load `parport_pc` with `irq=none`. See if printing works then. It really should, because this is the simplest code path.

> 因此，首先，禁用“CONFIG_PARPORT_PC_FIFO”，然后加载“PARPORT_PC”并使用“irq=none”。看看打印是否有效。它真的应该这样做，因为这是最简单的代码路径。


If that works fine, try with `io=0x378 irq=7` (adjust for your hardware), to make it use interrupt-driven in-software protocol.

> 如果效果良好，请尝试使用“io=0x378 irq=7”（根据硬件进行调整），使其使用软件协议中的中断驱动。


If **that** works fine, then one of the hardware modes isn\'t working right. Enable `CONFIG_FIFO` (no, it isn\'t a module option, and yes, it should be), set the port to ECP mode in the BIOS and note the DMA channel, and try with:

> 如果**那个**工作正常，那么其中一个硬件模式工作不正常。启用“CONFIG_FIFO”（不，它不是模块选项，是的，它应该是），在BIOS中将端口设置为ECP模式，并记下DMA通道，然后尝试：

    io=0x378 irq=7 dma=none (for PIO)
    io=0x378 irq=7 dma=3 (for DMA)

------------------------------------------------------------------------

<philb@gnu.org> <tim@cyberelk.net>
