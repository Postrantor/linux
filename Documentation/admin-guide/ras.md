---
tip: translate by baidu@2024-01-30 21:37:48
---
---
title: Reliability, Availability and Serviceability
---

# RAS concepts


Reliability, Availability and Serviceability (RAS) is a concept used on servers meant to measure their robustness.

> 可靠性、可用性和可服务性（RAS）是一个用于服务器的概念，旨在衡量其稳健性。

Reliability

:   is the probability that a system will produce correct outputs.

    -   Generally measured as Mean Time Between Failures (MTBF)
    -   Enhanced by features that help to avoid, detect and repair hardware faults

Availability

:   is the probability that a system is operational at a given time

    -   Generally measured as a percentage of downtime per a period of time
    -   Often uses mechanisms to detect and correct hardware faults in runtime;

Serviceability (or maintainability)


:   is the simplicity and speed with which a system can be repaired or maintained

> ：是系统维修或维护的简单性和速度

    -   Generally measured on Mean Time Between Repair (MTBR)

## Improving RAS


In order to reduce systems downtime, a system should be capable of detecting hardware errors, and, when possible correcting them in runtime. It should also provide mechanisms to detect hardware degradation, in order to warn the system administrator to take the action of replacing a component before it causes data loss or system downtime.

> 为了减少系统停机时间，系统应该能够检测硬件错误，并在可能的情况下在运行时纠正这些错误。它还应提供检测硬件退化的机制，以便警告系统管理员在组件导致数据丢失或系统停机之前采取更换组件的措施。

Among the monitoring measures, the most usual ones include:

-   CPU -- detect errors at instruction execution and at L1/L2/L3 caches;
-   Memory -- add error correction logic (ECC) to detect and correct errors;
-   I/O -- add CRC checksums for transferred data;

-   Storage -- RAID, journal file systems, checksums, Self-Monitoring, Analysis and Reporting Technology (SMART).

> -存储--RAID、日志文件系统、校验和、自我监控、分析和报告技术（SMART）。


By monitoring the number of occurrences of error detections, it is possible to identify if the probability of hardware errors is increasing, and, on such case, do a preventive maintenance to replace a degraded component while those errors are correctable.

> 通过监测错误检测的发生次数，可以识别硬件错误的概率是否在增加，并且在这种情况下，进行预防性维护以更换降级的组件，同时这些错误是可纠正的。

## Types of errors


Most mechanisms used on modern systems use technologies like Hamming Codes that allow error correction when the number of errors on a bit packet is below a threshold. If the number of errors is above, those mechanisms can indicate with a high degree of confidence that an error happened, but they can\'t correct.

> 现代系统上使用的大多数机制使用诸如汉明码之类的技术，该技术允许在比特分组上的错误数量低于阈值时进行纠错。如果错误数量在以上，这些机制可以高度确信地表明发生了错误，但它们无法纠正。


Also, sometimes an error occur on a component that it is not used. For example, a part of the memory that it is not currently allocated.

> 此外，有时在未使用的组件上会出现错误。例如，内存中当前未分配的部分。

That defines some categories of errors:


-   **Correctable Error (CE)** - the error detection mechanism detected and corrected the error. Such errors are usually not fatal, although some Kernel mechanisms allow the system administrator to consider them as fatal.

> -**可纠正错误（CE）**-错误检测机制检测并纠正错误。这样的错误通常不是致命的，尽管一些内核机制允许系统管理员认为它们是致命的。


-   **Uncorrected Error (UE)** - the amount of errors happened above the error correction threshold, and the system was unable to auto-correct.

> -**未更正的错误（UE）**-发生的错误数量超过错误更正阈值，系统无法自动更正。


-   **Fatal Error** - when an UE error happens on a critical component of the system (for example, a piece of the Kernel got corrupted by an UE), the only reliable way to avoid data corruption is to hang or reboot the machine.

> -**致命错误**-当UE错误发生在系统的关键组件上时（例如，某个内核被UE损坏），避免数据损坏的唯一可靠方法是挂起或重新启动机器。


-   **Non-fatal Error** - when an UE error happens on an unused component, like a CPU in power down state or an unused memory bank, the system may still run, eventually replacing the affected hardware by a hot spare, if available.

> -**非致命错误**-当UE错误发生在未使用的组件上时，如CPU处于断电状态或未使用的内存组，系统可能仍在运行，最终用热备盘替换受影响的硬件（如果可用）。

    Also, when an error happens on a userspace process, it is also possible to kill such process and let userspace restart it.


The mechanism for handling non-fatal errors is usually complex and may require the help of some userspace application, in order to apply the policy desired by the system administrator.

> 处理非致命错误的机制通常很复杂，可能需要一些用户空间应用程序的帮助，以便应用系统管理员所需的策略。

## Identifying a bad hardware component


Just detecting a hardware flaw is usually not enough, as the system needs to pinpoint to the minimal replaceable unit (MRU) that should be exchanged to make the hardware reliable again.

> 仅仅检测硬件缺陷通常是不够的，因为系统需要精确定位到应该更换的最小可更换单元（MRU），以使硬件再次可靠。


So, it requires not only error logging facilities, but also mechanisms that will translate the error message to the silkscreen or component label for the MRU.

> 因此，它不仅需要错误日志记录功能，还需要将错误消息转换为MRU的丝网印刷或组件标签的机制。


Typically, it is very complex for memory, as modern CPUs interlace memory from different memory modules, in order to provide a better performance. The DMI BIOS usually have a list of memory module labels, with can be obtained using the `dmidecode` tool. For example, on a desktop machine, it shows:

> 通常，这对内存来说非常复杂，因为现代CPU将内存与不同的内存模块交织在一起，以提供更好的性能。DMI BIOS通常有一个内存模块标签列表，可以使用“dmidecode”工具获得。例如，在台式机上，它显示：

    Memory Device
        Total Width: 64 bits
        Data Width: 64 bits
        Size: 16384 MB
        Form Factor: SODIMM
        Set: None
        Locator: ChannelA-DIMM0
        Bank Locator: BANK 0
        Type: DDR4
        Type Detail: Synchronous
        Speed: 2133 MHz
        Rank: 2
        Configured Clock Speed: 2133 MHz


On the above example, a DDR4 SO-DIMM memory module is located at the system\'s memory labeled as \"BANK 0\", as given by the *bank locator* field. Please notice that, on such system, the *total width* is equal to the *data width*. It means that such memory module doesn\'t have error detection/correction mechanisms.

> 在上面的示例中，DDR4 SO-DIMM内存模块位于系统内存中，标记为“BANK 0\”，如*BANK locator*字段所示。请注意，在这样的系统上，*总宽度*等于*数据宽度*。这意味着这样的内存模块没有错误检测/校正机制。


Unfortunately, not all systems use the same field to specify the memory bank. On this example, from an older server, `dmidecode` shows:

> 不幸的是，并非所有系统都使用相同的字段来指定内存组。在本例中，来自旧服务器的“dmidecode”显示：

    Memory Device
        Array Handle: 0x1000
        Error Information Handle: Not Provided
        Total Width: 72 bits
        Data Width: 64 bits
        Size: 8192 MB
        Form Factor: DIMM
        Set: 1
        Locator: DIMM_A1
        Bank Locator: Not Specified
        Type: DDR3
        Type Detail: Synchronous Registered (Buffered)
        Speed: 1600 MHz
        Rank: 2
        Configured Clock Speed: 1600 MHz


There, the DDR3 RDIMM memory module is located at the system\'s memory labeled as \"DIMM_A1\", as given by the *locator* field. Please notice that this memory module has 64 bits of *data width* and 72 bits of *total width*. So, it has 8 extra bits to be used by error detection and correction mechanisms. Such kind of memory is called Error-correcting code memory (ECC memory).

> 在那里，DDR3 RDIMM内存模块位于系统内存中，标记为“DIMM_A1\”，如*定位器*字段所示。请注意，此内存模块具有64位的*数据宽度*和72位的*总宽度*。因此，它有8个额外的比特供错误检测和校正机制使用。这种存储器被称为纠错码存储器（ECC存储器）。


To make things even worse, it is not uncommon that systems with different labels on their system\'s board to use exactly the same BIOS, meaning that the labels provided by the BIOS won\'t match the real ones.

> 更糟糕的是，系统板上有不同标签的系统使用完全相同的BIOS并不罕见，这意味着BIOS提供的标签与实际标签不匹配。

## ECC memory


As mentioned in the previous section, ECC memory has extra bits to be used for error correction. In the above example, a memory module has 64 bits of *data width*, and 72 bits of *total width*. The extra 8 bits which are used for the error detection and correction mechanisms are referred to as the *syndrome*[^1][^2].

> 如前一节所述，ECC存储器具有用于纠错的额外位。在上述示例中，存储器模块具有64位的*数据宽度*和72位的*总宽度*。用于错误检测和校正机制的额外8位被称为*综合症*[^1][^2]。


So, when the cpu requests the memory controller to write a word with *data width*, the memory controller calculates the *syndrome* in real time, using Hamming code, or some other error correction code, like SECDED+, producing a code with *total width* size. Such code is then written on the memory modules.

> 因此，当cpu请求存储器控制器写入具有*数据宽度*的字时，存储器控制器使用汉明码或一些其他纠错码（如SECDED+）实时计算*综合症*，产生具有*总宽度*大小的代码。这样的代码随后被写入到存储器模块上。


At read, the *total width* bits code is converted back, using the same ECC code used on write, producing a word with *data width* and a *syndrome*. The word with *data width* is sent to the CPU, even when errors happen.

> 在读取时，使用写入时使用的相同ECC码，将*总宽度*位代码转换回，产生具有*数据宽度*和*综合症*的字。即使发生错误，带有*数据宽度*的字也会发送到CPU。


The memory controller also looks at the *syndrome* in order to check if there was an error, and if the ECC code was able to fix such error. If the error was corrected, a Corrected Error (CE) happened. If not, an Uncorrected Error (UE) happened.

> 存储器控制器还查看*综合症*，以便检查是否存在错误，以及ECC代码是否能够修复这种错误。如果已更正错误，则会发生更正错误（CE）。如果没有，则发生未更正错误（UE）。


The information about the CE/UE errors is stored on some special registers at the memory controller and can be accessed by reading such registers, either by BIOS, by some special CPUs or by Linux EDAC driver. On x86 64 bit CPUs, such errors can also be retrieved via the Machine Check Architecture (MCA)[^3].

> 关于CE/UE错误的信息存储在存储器控制器的一些特殊寄存器上，并且可以通过BIOS、一些特殊CPU或Linux EDAC驱动程序读取这些寄存器来访问。在x86 64位CPU上，也可以通过机器检查体系结构（MCA）[^3]检索此类错误。

# EDAC - Error Detection And Correction

::: note
::: title
Note
:::


\"bluesmoke\" was the name for this device driver subsystem when it was \"out-of-tree\" and maintained at <http://bluesmoke.sourceforge.net>. That site is mostly archaic now and can be used only for historical purposes.

> \“bluesmoke”是该设备驱动程序子系统的名称，当它“脱离树”并维护在<http://bluesmoke.sourceforge.net>. 那个遗址现在大多已经过时，只能用于历史目的。


When the subsystem was pushed upstream for the first time, on Kernel 2.6.16, it was renamed to `EDAC`.

> 当该子系统在内核2.6.16上首次向上游推送时，它被重命名为“EDAC”。
:::

## Purpose


The `edac` kernel module\'s goal is to detect and report hardware errors that occur within the computer system running under linux.

> “edac”内核模块的目标是检测和报告在linux下运行的计算机系统中发生的硬件错误。

## Memory


Memory Correctable Errors (CE) and Uncorrectable Errors (UE) are the primary errors being harvested. These types of errors are harvested by the `edac_mc` device.

> 内存可纠正错误（CE）和不可纠正错误是正在获取的主要错误。这些类型的错误由“edac_mc”设备获取。


Detecting CE events, then harvesting those events and reporting them, **can** but must not necessarily be a predictor of future UE events. With CE events only, the system can and will continue to operate as no data has been damaged yet.

> 检测CE事件，然后收集这些事件并报告它们，**可以**，但不一定是未来UE事件的预测因素。仅通过CE事件，系统可以并且将继续运行，因为尚未损坏任何数据。


However, preventive maintenance and proactive part replacement of memory modules exhibiting CEs can reduce the likelihood of the dreaded UE events and system panics.

> 然而，对表现出CE的存储器模块进行预防性维护和主动部件更换可以降低可怕的UE事件和系统恐慌的可能性。

## Other hardware elements


A new feature for EDAC, the `edac_device` class of device, was added in the 2.6.23 version of the kernel.

> 在2.6.23版本的内核中添加了EDAC的一个新功能，即“EDAC_device”类设备。


This new device type allows for non-memory type of ECC hardware detectors to have their states harvested and presented to userspace via the sysfs interface.

> 这种新的设备类型允许非内存类型的ECC硬件检测器通过sysfs接口获取其状态并呈现给用户空间。


Some architectures have ECC detectors for L1, L2 and L3 caches, along with DMA engines, fabric switches, main data path switches, interconnections, and various other hardware data paths. If the hardware reports it, then a edac_device device probably can be constructed to harvest and present that to userspace.

> 一些体系结构具有用于L1、L2和L3高速缓存的ECC检测器，以及DMA引擎、结构交换机、主数据路径交换机、互连和各种其他硬件数据路径。如果硬件报告了它，那么可能可以构建edac_device设备来获取它并将其呈现给用户空间。

## PCI bus scanning


In addition, PCI devices are scanned for PCI Bus Parity and SERR Errors in order to determine if errors are occurring during data transfers.

> 此外，扫描PCI设备的PCI总线奇偶校验和SERR错误，以确定数据传输过程中是否发生错误。


The presence of PCI Parity errors must be examined with a grain of salt. There are several add-in adapters that do **not** follow the PCI specification with regards to Parity generation and reporting. The specification says the vendor should tie the parity status bits to 0 if they do not intend to generate parity. Some vendors do not do this, and thus the parity bit can \"float\" giving false positives.

> PCI奇偶校验错误的存在必须谨慎检查。有几个附加适配器在奇偶校验生成和报告方面不遵循PCI规范。该规范规定，如果供应商不打算生成奇偶校验，则应将奇偶校验状态位绑定到0。有些供应商不这样做，因此奇偶校验位可能“浮动”，从而给出误报。


There is a PCI device attribute located in sysfs that is checked by the EDAC PCI scanning code. If that attribute is set, PCI parity/error scanning is skipped for that device. The attribute is:

> 位于sysfs中的PCI设备属性由EDAC PCI扫描代码检查。如果设置了该属性，则跳过该设备的PCI奇偶校验/错误扫描。属性为：

    broken_parity_status


and is located in `/sys/devices/pci<XXX>/0000:XX:YY.Z` directories for PCI devices.

> 并且位于`/sys/devices/pci<XXX>/000:XX:YY中。用于PCI设备的Z`目录。

## Versioning


EDAC is composed of a \"core\" module (`edac_core.ko`) and several Memory Controller (MC) driver modules. On a given system, the CORE is loaded and one MC driver will be loaded. Both the CORE and the MC driver (or `edac_device` driver) have individual versions that reflect current release level of their respective modules.

> EDAC由一个“核心”模块（`EDAC_core.ko`）和几个内存控制器（MC）驱动程序模块组成。在给定的系统上，加载CORE，并加载一个MC驱动程序。CORE和MC驱动程序（或“edac_device”驱动程序）都具有反映其各自模块的当前发布级别的单独版本。


Thus, to \"report\" on what version a system is running, one must report both the CORE\'s and the MC driver\'s versions.

> 因此，要“报告”系统运行的版本，必须报告CORE和MC驱动程序的版本。

## Loading


If `edac` was statically linked with the kernel then no loading is necessary. If `edac` was built as modules then simply modprobe the `edac` pieces that you need. You should be able to modprobe hardware-specific modules and have the dependencies load the necessary core modules.

> 如果“edac”与内核静态链接，则不需要加载。如果“edac”是作为模块构建的，那么只需对所需的“edac’部件进行modprobe。您应该能够对特定于硬件的模块进行modprobe，并让依赖项加载必要的核心模块。

Example:

    $ modprobe amd76x_edac


loads both the `amd76x_edac.ko` memory controller module and the `edac_mc.ko` core module.

> 加载“amd76x_edac.ko”存储器控制器模块和“edac_mc.ko”核心模块。

## Sysfs interface


EDAC presents a `sysfs` interface for control and reporting purposes. It lives in the /sys/devices/system/edac directory.

> EDAC提供了一个“sysfs”接口，用于控制和报告目的。它位于/sys/devices/system/edac目录中。

Within this directory there currently reside 2 components:

>   --------- ---------------------------
>   mc memo   ry controller(s) system
>   pci PCI   control and status system
>   --------- ---------------------------

## Memory Controller (mc) Model


Each `mc` device controls a set of memory modules[^4]. These modules are laid out in a Chip-Select Row (`csrowX`) and Channel table (`chX`). There can be multiple csrows and multiple channels.

> 每个“mc”设备控制一组内存模块[^4]。这些模块被布置在芯片选择行（“csrowX”）和通道表（“chX”）中。可以有多个csrow和多个通道。


Memory controllers allow for several csrows, with 8 csrows being a typical value. Yet, the actual number of csrows depends on the layout of a given motherboard, memory controller and memory module characteristics.

> 内存控制器允许多个csrow，其中8个csrow是一个典型值。然而，csrows的实际数量取决于给定主板的布局、内存控制器和内存模块的特性。


Dual channels allow for dual data length (e. g. 128 bits, on 64 bit systems) data transfers to/from the CPU from/to memory. Some newer chipsets allow for more than 2 channels, like Fully Buffered DIMMs (FB-DIMMs) memory controllers. The following example will assume 2 channels:

> 双通道允许双数据长度（例如，在64位系统上为128位）从存储器到CPU的数据传输。一些较新的芯片组允许2个以上通道，如全缓冲DIMM（FB DIMM）内存控制器。以下示例假设有2个通道：

> +------------+-----------------------+
> | CS Rows    | > Channels            |
> +------------+-----------------------+
> |            | > `ch0` \| `ch1`      |
> +------------+-----------------------+
>
> +============+===========+===========+ \| **DIMM_B0**\| +\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\--+ \| `csrow0` \| rank0 \| rank0 \| +\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\--+ \| `csrow1` \| rank1 \| rank1 \| +\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\--+ \| **DIMM_B1**\| +\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\--+ \| `csrow2` \| rank0 \| rank0 \| +\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\--+ \| `csrow3` \| rank1 \| rank1 \| +\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\--+


In the above example, there are 4 physical slots on the motherboard for memory DIMMs:

> 在上面的示例中，主板上有4个用于内存DIMM的物理插槽：

>   --------- ---------
>   DIMM_A0   DIMM_B0
>
>   DIMM_A1   DIMM_B1
>   --------- ---------


Labels for these slots are usually silk-screened on the motherboard. Slots labeled `A` are channel 0 in this example. Slots labeled `B` are channel 1. Notice that there are two csrows possible on a physical DIMM. These csrows are allocated their csrow assignment based on the slot into which the memory DIMM is placed. Thus, when 1 DIMM is placed in each Channel, the csrows cross both DIMMs.

> 这些插槽的标签通常是在主板上丝网印刷的。在本例中，标记为“A”的插槽为通道0。标有“B”的插槽是通道1。请注意，在一个物理DIMM上可能有两个csrow。这些csrow根据内存DIMM所在的插槽分配csrow分配。因此，当每个通道中放置1个DIMM时，csrows会穿过两个DIMM。


Memory DIMMs come single or dual \"ranked\". A rank is a populated csrow. In the example above 2 dual ranked DIMMs are similarly placed. Thus, both csrow0 and csrow1 are populated. On the other hand, when 2 single ranked DIMMs are placed in slots DIMM_A0 and DIMM_B0, then they will have just one csrow (csrow0) and csrow1 will be empty. The pattern repeats itself for csrow2 and csrow3. Also note that some memory controllers don\'t have any logic to identify the memory module, see `rankX` directories below.

> 内存DIMM有单排或双排。等级是一个填充的csrow。在上面的示例中，类似地放置了2个双列DIMM。因此，csrow0和csrow1都被填充。另一方面，当2个单列DIMM放置在插槽DIMM_A0和DIMM_B0中时，它们将只有一个csrow（csrow0），csrow1将为空。对于csrow2和csrow3，该模式会自行重复。还要注意，一些内存控制器没有任何逻辑来识别内存模块，请参阅下面的“rankX”目录。


The representation of the above is reflected in the directory tree in EDAC\'s sysfs interface. Starting in directory `/sys/devices/system/edac/mc`, each memory controller will be represented by its own `mcX` directory, where `X` is the index of the MC:

> 上面的表示方式反映在EDAC\的sysfs接口中的目录树中。从目录“/sys/devices/system/edac/mc”开始，每个存储器控制器将由其自己的“mcX”目录表示，其中“X”是mc的索引：

    ..../edac/mc/
           |
           |->mc0
           |->mc1
           |->mc2
           ....


Under each `mcX` directory each `csrowX` is again represented by a `csrowX`, where `X` is the csrow index:

> 在每个“mcX”目录下，每个“csrowX”再次由“csrowX'”表示，其中“X”是csrow索引：

    .../mc/mc0/
        |
        |->csrow0
        |->csrow2
        |->csrow3
        ....


Notice that there is no csrow1, which indicates that csrow0 is composed of a single ranked DIMMs. This should also apply in both Channels, in order to have dual-channel mode be operational. Since both csrow2 and csrow3 are populated, this indicates a dual ranked set of DIMMs for channels 0 and 1.

> 请注意，没有csrow1，这表示csrow0由单列DIMM组成。这也应适用于两个通道，以便使双通道模式可操作。由于csrow2和csrow3都已填充，这表示通道0和1的DIMM为双列排列。


Within each of the `mcX` and `csrowX` directories are several EDAC control and attribute files.

> 在每个“mcX”和“csrowX”目录中都有几个EDAC控制和属性文件。

## `mcX` directories


In `mcX` directories are EDAC control and attribute files for this `X` instance of the memory controllers.

> 在“mcX”目录中是存储器控制器的这个“X”实例的EDAC控制和属性文件。

For a description of the sysfs API, please see:

> Documentation/ABI/testing/sysfs-devices-edac

## `dimmX` or `rankX` directories


The recommended way to use the EDAC subsystem is to look at the information provided by the `dimmX` or `rankX` directories[^5].

> 建议使用EDAC子系统的方法是查看“dimmX”或“rankX”目录[^5]提供的信息。


A typical EDAC system has the following structure under `/sys/devices/system/edac/`[^6]:

> 典型的EDAC系统在“/sys/devices/system/EDAC/”[^6]下具有以下结构：

    /sys/devices/system/edac/
    ├── mc
    │   ├── mc0
    │   │   ├── ce_count
    │   │   ├── ce_noinfo_count
    │   │   ├── dimm0
    │   │   │   ├── dimm_ce_count
    │   │   │   ├── dimm_dev_type
    │   │   │   ├── dimm_edac_mode
    │   │   │   ├── dimm_label
    │   │   │   ├── dimm_location
    │   │   │   ├── dimm_mem_type
    │   │   │   ├── dimm_ue_count
    │   │   │   ├── size
    │   │   │   └── uevent
    │   │   ├── max_location
    │   │   ├── mc_name
    │   │   ├── reset_counters
    │   │   ├── seconds_since_reset
    │   │   ├── size_mb
    │   │   ├── ue_count
    │   │   ├── ue_noinfo_count
    │   │   └── uevent
    │   ├── mc1
    │   │   ├── ce_count
    │   │   ├── ce_noinfo_count
    │   │   ├── dimm0
    │   │   │   ├── dimm_ce_count
    │   │   │   ├── dimm_dev_type
    │   │   │   ├── dimm_edac_mode
    │   │   │   ├── dimm_label
    │   │   │   ├── dimm_location
    │   │   │   ├── dimm_mem_type
    │   │   │   ├── dimm_ue_count
    │   │   │   ├── size
    │   │   │   └── uevent
    │   │   ├── max_location
    │   │   ├── mc_name
    │   │   ├── reset_counters
    │   │   ├── seconds_since_reset
    │   │   ├── size_mb
    │   │   ├── ue_count
    │   │   ├── ue_noinfo_count
    │   │   └── uevent
    │   └── uevent
    └── uevent


In the `dimmX` directories are EDAC control and attribute files for this `X` memory module:

> 在“dimmX”目录中是此“X”内存模块的EDAC控制和属性文件：

-   `size` - Total memory managed by this csrow attribute file

    > This attribute file displays, in count of megabytes, the memory that this csrow contains.

-   `dimm_ue_count` - Uncorrectable Errors count attribute file

    > This attribute file displays the total count of uncorrectable errors that have occurred on this DIMM. If panic_on_ue is set this counter will not have a chance to increment, since EDAC will panic the system.

-   `dimm_ce_count` - Correctable Errors count attribute file

    > This attribute file displays the total count of correctable errors that have occurred on this DIMM. This count is very important to examine. CEs provide early indications that a DIMM is beginning to fail. This count field should be monitored for non-zero values and report such information to the system administrator.

-   `dimm_dev_type` - Device type attribute file

    > This attribute file will display what type of DRAM device is being utilized on this DIMM. Examples:
    >
    > > -   x1
    > > -   x2
    > > -   x4
    > > -   x8

-   `dimm_edac_mode` - EDAC Mode of operation attribute file

    > This attribute file will display what type of Error detection and correction is being utilized.

-   `dimm_label` - memory module label control file

    > This control file allows this DIMM to have a label assigned to it. With this label in the module, when errors occur the output can provide the DIMM label in the system log. This becomes vital for panic events to isolate the cause of the UE event.
    >
    > DIMM Labels must be assigned after booting, with information that correctly identifies the physical slot with its silk screen label. This information is currently very motherboard specific and determination of this information must occur in userland at this time.

-   `dimm_location` - location of the memory module

    > The location can have up to 3 levels, and describe how the memory controller identifies the location of a memory module. Depending on the type of memory and memory controller, it can be:
    >
    > > -   *csrow* and *channel* - used when the memory controller doesn\'t identify a single DIMM - e. g. in `rankX` dir;
    > > -   *branch*, *channel*, *slot* - typically used on FB-DIMM memory controllers;
    > > -   *channel*, *slot* - used on Nehalem and newer Intel drivers.

-   `dimm_mem_type` - Memory Type attribute file

    > This attribute file will display what type of memory is currently on this csrow. Normally, either buffered or unbuffered memory. Examples:
    >
    > > -   Registered-DDR
    > > -   Unbuffered-DDR

## `csrowX` directories


When CONFIG_EDAC_LEGACY_SYSFS is enabled, sysfs will contain the `csrowX` directories. As this API doesn\'t work properly for Rambus, FB-DIMMs and modern Intel Memory Controllers, this is being deprecated in favor of `dimmX` directories.

> 启用CONFIG_EDAC_LEGACY_SYSFS时，SYSFS将包含“csrowX”目录。由于此API不适用于Rambus、FB-DIMM和现代Intel内存控制器，因此不赞成使用“dimmX”目录。


In the `csrowX` directories are EDAC control and attribute files for this `X` instance of csrow:

> 在“csrowX”目录中是csrow的此“X”实例的EDAC控制和属性文件：

-   `ue_count` - Total Uncorrectable Errors count attribute file

    > This attribute file displays the total count of uncorrectable errors that have occurred on this csrow. If panic_on_ue is set this counter will not have a chance to increment, since EDAC will panic the system.

-   `ce_count` - Total Correctable Errors count attribute file

    > This attribute file displays the total count of correctable errors that have occurred on this csrow. This count is very important to examine. CEs provide early indications that a DIMM is beginning to fail. This count field should be monitored for non-zero values and report such information to the system administrator.

-   `size_mb` - Total memory managed by this csrow attribute file

    > This attribute file displays, in count of megabytes, the memory that this csrow contains.

-   `mem_type` - Memory Type attribute file

    > This attribute file will display what type of memory is currently on this csrow. Normally, either buffered or unbuffered memory. Examples:
    >
    > > -   Registered-DDR
    > > -   Unbuffered-DDR

-   `edac_mode` - EDAC Mode of operation attribute file

    > This attribute file will display what type of Error detection and correction is being utilized.

-   `dev_type` - Device type attribute file

    > This attribute file will display what type of DRAM device is being utilized on this DIMM. Examples:
    >
    > > -   x1
    > > -   x2
    > > -   x4
    > > -   x8

-   `ch0_ce_count` - Channel 0 CE Count attribute file

    > This attribute file will display the count of CEs on this DIMM located in channel 0.

-   `ch0_ue_count` - Channel 0 UE Count attribute file

    > This attribute file will display the count of UEs on this DIMM located in channel 0.

-   `ch0_dimm_label` - Channel 0 DIMM Label control file

    > This control file allows this DIMM to have a label assigned to it. With this label in the module, when errors occur the output can provide the DIMM label in the system log. This becomes vital for panic events to isolate the cause of the UE event.
    >
    > DIMM Labels must be assigned after booting, with information that correctly identifies the physical slot with its silk screen label. This information is currently very motherboard specific and determination of this information must occur in userland at this time.

-   `ch1_ce_count` - Channel 1 CE Count attribute file

    > This attribute file will display the count of CEs on this DIMM located in channel 1.

-   `ch1_ue_count` - Channel 1 UE Count attribute file

    > This attribute file will display the count of UEs on this DIMM located in channel 0.

-   `ch1_dimm_label` - Channel 1 DIMM Label control file

    > This control file allows this DIMM to have a label assigned to it. With this label in the module, when errors occur the output can provide the DIMM label in the system log. This becomes vital for panic events to isolate the cause of the UE event.
    >
    > DIMM Labels must be assigned after booting, with information that correctly identifies the physical slot with its silk screen label. This information is currently very motherboard specific and determination of this information must occur in userland at this time.

## System Logging


If logging for UEs and CEs is enabled, then system logs will contain information indicating that errors have been detected:

> 如果启用了UE和CE的日志记录，则系统日志将包含指示已检测到错误的信息：

    EDAC MC0: CE page 0x283, offset 0xce0, grain 8, syndrome 0x6ec3, row 0, channel 1 "DIMM_B1": amd76x_edac
    EDAC MC0: CE page 0x1e5, offset 0xfb0, grain 8, syndrome 0xb741, row 0, channel 1 "DIMM_B1": amd76x_edac

The structure of the message is:

>   ---------------------------------------------------------------------------------------------------
>   Content                                                                               Example
>   ------------------------------------------------------------------------------------- -------------
>   The memory controller                                                                 MC0
>
>   Error type                                                                            CE
>
>   Memory page                                                                           0x283
>
>   Offset in the page                                                                    0xce0
>
>   The byte granularity or resolution of the error                                       grain 8
>
>   The error syndrome                                                                    0xb741
>
>   Memory row                                                                            row 0
>
>   Memory channel                                                                        channel 1
>
>   DIMM label, if set prior                                                              DIMM B1
>
>   And then an optional, driver-specific message that may have additional information.   
>   ---------------------------------------------------------------------------------------------------


Both UEs and CEs with no info will lack all but memory controller, error type, a notice of \"no info\" and then an optional, driver-specific error message.

> 没有信息的UE和CE都将缺少除内存控制器、错误类型、“没有信息”的通知以及可选的特定于驱动程序的错误消息之外的所有信息。

## PCI Bus Parity Detection


On Header Type 00 devices, the primary status is looked at for any parity error regardless of whether parity is enabled on the device or not. (The spec indicates parity is generated in some cases). On Header Type 01 bridges, the secondary status register is also looked at to see if parity occurred on the bus on the other side of the bridge.

> 在标头类型00设备上，无论设备上是否启用奇偶校验，都会查看主状态中是否存在任何奇偶校验错误。（规范表示在某些情况下会生成奇偶校验）。在标头类型01桥接器上，还查看辅助状态寄存器，以查看桥接器另一侧的总线上是否发生奇偶校验。

## Sysfs configuration


Under `/sys/devices/system/edac/pci` are control and attribute files as follows:

> 在“/sys/devices/system/edac/pci”下是控制和属性文件，如下所示：

-   `check_pci_parity` - Enable/Disable PCI Parity checking control file

    > This control file enables or disables the PCI Bus Parity scanning operation. Writing a 1 to this file enables the scanning. Writing a 0 to this file disables the scanning.
    >
    > Enable:
    >
    >     echo "1" >/sys/devices/system/edac/pci/check_pci_parity
    >
    > Disable:
    >
    >     echo "0" >/sys/devices/system/edac/pci/check_pci_parity

-   `pci_parity_count` - Parity Count

    > This attribute file will display the number of parity errors that have been detected.

## Module parameters

-   `edac_mc_panic_on_ue` - Panic on UE control file

    > An uncorrectable error will cause a machine panic. This is usually desirable. It is a bad idea to continue when an uncorrectable error occurs - it is indeterminate what was uncorrected and the operating system context might be so mangled that continuing will lead to further corruption. If the kernel has MCE configured, then EDAC will never notice the UE.
    >
    > LOAD TIME:
    >
    >     module/kernel parameter: edac_mc_panic_on_ue=[0|1]
    >
    > RUN TIME:
    >
    >     echo "1" > /sys/module/edac_core/parameters/edac_mc_panic_on_ue

-   `edac_mc_log_ue` - Log UE control file

    > Generate kernel messages describing uncorrectable errors. These errors are reported through the system message log system. UE statistics will be accumulated even when UE logging is disabled.
    >
    > LOAD TIME:
    >
    >     module/kernel parameter: edac_mc_log_ue=[0|1]
    >
    > RUN TIME:
    >
    >     echo "1" > /sys/module/edac_core/parameters/edac_mc_log_ue

-   `edac_mc_log_ce` - Log CE control file

    > Generate kernel messages describing correctable errors. These errors are reported through the system message log system. CE statistics will be accumulated even when CE logging is disabled.
    >
    > LOAD TIME:
    >
    >     module/kernel parameter: edac_mc_log_ce=[0|1]
    >
    > RUN TIME:
    >
    >     echo "1" > /sys/module/edac_core/parameters/edac_mc_log_ce

-   `edac_mc_poll_msec` - Polling period control file

    > The time period, in milliseconds, for polling for error information. Too small a value wastes resources. Too large a value might delay necessary handling of errors and might loose valuable information for locating the error. 1000 milliseconds (once each second) is the current default. Systems which require all the bandwidth they can get, may increase this.
    >
    > LOAD TIME:
    >
    >     module/kernel parameter: edac_mc_poll_msec=[0|1]
    >
    > RUN TIME:
    >
    >     echo "1000" > /sys/module/edac_core/parameters/edac_mc_poll_msec

-   `panic_on_pci_parity` - Panic on PCI PARITY Error

    > This control file enables or disables panicking when a parity error has been detected.
    >
    > module/kernel parameter:
    >
    >     edac_panic_on_pci_pe=[0|1]
    >
    > Enable:
    >
    >     echo "1" > /sys/module/edac_core/parameters/edac_panic_on_pci_pe
    >
    > Disable:
    >
    >     echo "0" > /sys/module/edac_core/parameters/edac_panic_on_pci_pe

## EDAC device type


In the header file, edac_pci.h, there is a series of edac_device structures and APIs for the EDAC_DEVICE.

> 在头文件edac_pci.h中，有一系列用于edac_device的edac_device结构和API。

User space access to an edac_device is through the sysfs interface.


At the location `/sys/devices/system/edac` (sysfs) new edac_device devices will appear.

> 在“/sys/devices/system/edac”（sysfs）位置，将出现新的edac_device设备。


There is a three level tree beneath the above `edac` directory. For example, the `test_device_edac` device (found at the <http://bluesmoke.sourceforget.net> website) installs itself as:

> 在上面的“edac”目录下有一个三级树。例如，“test_device_edac”设备（位于<http://bluesmoke.sourceforget.net>网站）将自己安装为：

    /sys/devices/system/edac/test-instance


in this directory are various controls, a symlink and one or more `instance` directories.

> 这个目录中有各种控件、符号链接和一个或多个“实例”目录。

The standard default controls are:

> +------------------+---------------------------------------------------+
> | log_ce bool      | ean to log CE events                              |
> +------------------+---------------------------------------------------+
> | log_ue bool      | ean to log UE events                              |
> +------------------+---------------------------------------------------+
> | panic_on_ue bool | ean to `panic` the system if an UE is encountered |
> +------------------+---------------------------------------------------+
> | > (default       | > off, can be set true via startup script)        |
> +------------------+---------------------------------------------------+
> | poll_msec time   | > period between POLL cycles for events           |
> +------------------+---------------------------------------------------+

The test_device_edac device adds at least one of its own custom control:

> +----------------+-----------------------------------------------+
> | test_bits whic | h in the current test driver does nothing but |
> +----------------+-----------------------------------------------+
> | > show how     | > it is installed. A ported driver can        |
> +----------------+-----------------------------------------------+
> | > add one      | or more such controls and/or attributes       |
> +----------------+-----------------------------------------------+
> | > for spec     | ific uses.                                    |
> +----------------+-----------------------------------------------+
> | > One out-     | of-tree driver uses controls here to allow    |
> +----------------+-----------------------------------------------+
> | > for ERRO     | R INJECTION operations to hardware            |
> +----------------+-----------------------------------------------+
> | > injectio     | n registers                                   |
> +----------------+-----------------------------------------------+


The symlink points to the \'struct dev\' that is registered for this edac_device.

> 符号链接指向为此edac_设备注册的“struct dev”。

## Instances


One or more instance directories are present. For the `test_device_edac` case:

> 存在一个或多个实例目录。对于“test_device_edac”情况：

>   ----------------
>   test-instance0
>
>   ----------------


In this directory there are two default counter attributes, which are totals of counter in deeper subdirectories.

> 在这个目录中有两个默认的计数器属性，它们是更深子目录中计数器的总数。

>   --------------- ----------------------------------
>   ce_count tota   l of CE events of subdirectories
>   ue_count tota   l of UE events of subdirectories
>   --------------- ----------------------------------

## Blocks


At the lowest directory level is the `block` directory. There can be 0, 1 or more blocks specified in each instance:

> 在最低的目录级别是“块”目录。每个实例中可以指定0个、1个或多个块：

>   -------------
>   test-block0
>
>   -------------

In this directory the default attributes are:

> +---------------+--------------------------------------------+
> | ce_count whic | h is counter of CE events for this `block` |
> +---------------+--------------------------------------------+
> | > of hardw    | are being monitored                        |
> +---------------+--------------------------------------------+
> | ue_count whic | h is counter of UE events for this `block` |
> +---------------+--------------------------------------------+
> | > of hardw    | are being monitored                        |
> +---------------+--------------------------------------------+

The `test_device_edac` device adds 4 attributes and 1 control:

> +-------------------+---------------------------------------------------+
> | test-block-bits-0 | > for every POLL cycle this counter               |
> +-------------------+---------------------------------------------------+
> | > is incr         | emented                                           |
> +-------------------+---------------------------------------------------+
> | test-block-bits-1 | > every 10 cycles, this counter is bumped once,   |
> +-------------------+---------------------------------------------------+
> | > and tes         | t-block-bits-0 is set to 0                        |
> +-------------------+---------------------------------------------------+
> | test-block-bits-2 | > every 100 cycles, this counter is bumped once,  |
> +-------------------+---------------------------------------------------+
> | > and tes         | t-block-bits-1 is set to 0                        |
> +-------------------+---------------------------------------------------+
> | test-block-bits-3 | > every 1000 cycles, this counter is bumped once, |
> +-------------------+---------------------------------------------------+
> | > and tes         | t-block-bits-2 is set to 0                        |
> +-------------------+---------------------------------------------------+
>
> +----------------+------------------------------------------+
> | reset-counters | > writing ANY thing to this control will |
> +----------------+------------------------------------------+
> | > reset a      | ll the above counters.                   |
> +----------------+------------------------------------------+


Use of the `test_device_edac` driver should enable any others to create their own unique drivers for their hardware systems.

> “test_device_edac”驱动程序的使用应使任何其他人能够为其硬件系统创建自己的唯一驱动程序。


The `test_device_edac` sample driver is located at the <http://bluesmoke.sourceforge.net> project site for EDAC.

> “test_device_edac”示例驱动程序位于<http://bluesmoke.sourceforge.net>EDAC的项目现场。

## Usage of EDAC APIs on Nehalem and newer Intel CPUs


On older Intel architectures, the memory controller was part of the North Bridge chipset. Nehalem, Sandy Bridge, Ivy Bridge, Haswell, Sky Lake and newer Intel architectures integrated an enhanced version of the memory controller (MC) inside the CPUs.

> 在旧的英特尔体系结构上，内存控制器是北桥芯片组的一部分。Nehalem、Sandy Bridge、Ivy Bridge、Haswell、Sky Lake和更新的Intel架构在CPU中集成了增强版内存控制器（MC）。


This chapter will cover the differences of the enhanced memory controllers found on newer Intel CPUs, such as `i7core_edac`, `sb_edac` and `sbx_edac` drivers.

> 本章将介绍在较新的英特尔CPU上发现的增强型内存控制器的差异，如“i7core_edac”、“sb_edac”和“sbx_edac”驱动程序。

::: note
::: title
Note
:::


The Xeon E7 processor families use a separate chip for the memory controller, called Intel Scalable Memory Buffer. This section doesn\'t apply for such families.

> Xeon E7处理器系列使用一个单独的芯片作为内存控制器，称为Intel可扩展内存缓冲区。本节不适用于此类家庭。
:::


1)  There is one Memory Controller per Quick Patch Interconnect (QPI). At the driver, the term \"socket\" means one QPI. This is associated with a physical CPU socket.

> 1） 每个快速补丁互连（QPI）有一个内存控制器。在驱动程序中，术语“套接字”表示一个QPI。这与物理CPU插槽相关联。

    Each MC have 3 physical read channels, 3 physical write channels and 3 logic channels. The driver currently sees it as just 3 channels. Each channel can have up to 3 DIMMs.

    The minimum known unity is DIMMs. There are no information about csrows. As EDAC API maps the minimum unity is csrows, the driver sequentially maps channel/DIMM into different csrows.

    For example, supposing the following layout:

        Ch0 phy rd0, wr0 (0x063f4031): 2 ranks, UDIMMs
          dimm 0 1024 Mb offset: 0, bank: 8, rank: 1, row: 0x4000, col: 0x400
          dimm 1 1024 Mb offset: 4, bank: 8, rank: 1, row: 0x4000, col: 0x400
            Ch1 phy rd1, wr1 (0x063f4031): 2 ranks, UDIMMs
          dimm 0 1024 Mb offset: 0, bank: 8, rank: 1, row: 0x4000, col: 0x400
        Ch2 phy rd3, wr3 (0x063f4031): 2 ranks, UDIMMs
          dimm 0 1024 Mb offset: 0, bank: 8, rank: 1, row: 0x4000, col: 0x400

    The driver will map it as:

        csrow0: channel 0, dimm0
        csrow1: channel 0, dimm1
        csrow2: channel 1, dimm0
        csrow3: channel 2, dimm0

    exports one DIMM per csrow.

    Each QPI is exported as a different memory controller.


2)  The MC has the ability to inject errors to test drivers. The drivers implement this functionality via some error injection nodes:

> 2） MC有能力注入错误来测试驱动程序。驱动程序通过一些错误注入节点实现此功能：

    For injecting a memory error, there are some sysfs nodes, under `/sys/devices/system/edac/mc/mc?/`:

    -   

        `inject_addrmatch/*`:

        :   Controls the error injection mask register. It is possible to specify several characteristics of the address to match an error code:

                dimm = the affected dimm. Numbers are relative to a channel;
                rank = the memory rank;
                channel = the channel that will generate an error;
                bank = the affected bank;
                page = the page address;
                column (or col) = the address column.

            each of the above values can be set to \"any\" to match any valid value.

            At driver init, all values are set to any.

            For example, to generate an error at rank 1 of dimm 2, for any channel, any bank, any page, any column:

                echo 2 >/sys/devices/system/edac/mc/mc0/inject_addrmatch/dimm
                echo 1 >/sys/devices/system/edac/mc/mc0/inject_addrmatch/rank

    > To return to the default behaviour of matching any, you can do:
    >
    >     echo any >/sys/devices/system/edac/mc/mc0/inject_addrmatch/dimm
    >     echo any >/sys/devices/system/edac/mc/mc0/inject_addrmatch/rank

    -   

        `inject_eccmask`:

        :   specifies what bits will have troubles,

    -   

        `inject_section`:

        :   specifies what ECC cache section will get the error:

                3 for both
                2 for the highest
                1 for the lowest

    -   

        `inject_type`:

        :   specifies the type of error, being a combination of the following bits:

                bit 0 - repeat
                bit 1 - ecc
                bit 2 - parity

    -   

        `inject_enable`:

        :   starts the error generation when something different than 0 is written.

    All inject vars can be read. root permission is needed for write.

    Datasheet states that the error will only be generated after a write on an address that matches inject_addrmatch. It seems, however, that reading will also produce an error.

    For example, the following code will generate an error for any write access at socket 0, on any DIMM/address on channel 2:

        echo 2 >/sys/devices/system/edac/mc/mc0/inject_addrmatch/channel
        echo 2 >/sys/devices/system/edac/mc/mc0/inject_type
        echo 64 >/sys/devices/system/edac/mc/mc0/inject_eccmask
        echo 3 >/sys/devices/system/edac/mc/mc0/inject_section
        echo 1 >/sys/devices/system/edac/mc/mc0/inject_enable
        dd if=/dev/mem of=/dev/null seek=16k bs=4k count=1 >& /dev/null

    For socket 1, it is needed to replace \"mc0\" by \"mc1\" at the above commands.

    The generated error message will look like:

        EDAC MC0: UE row 0, channel-a= 0 channel-b= 0 labels "-": NON_FATAL (addr = 0x0075b980, socket=0, Dimm=0, Channel=2, syndrome=0x00000040, count=1, Err=8c0000400001009f:4000080482 (read error: read ECC error))

3)  Corrected Error memory register counters

    Those newer MCs have some registers to count memory errors. The driver uses those registers to report Corrected Errors on devices with Registered DIMMs.

    However, those counters don\'t work with Unregistered DIMM. As the chipset offers some counters that also work with UDIMMs (but with a worse level of granularity than the default ones), the driver exposes those registers for UDIMM memories.

    They can be read by looking at the contents of `all_channel_counts/`:

        $ for i in /sys/devices/system/edac/mc/mc0/all_channel_counts/*; do echo $i; cat $i; done
        /sys/devices/system/edac/mc/mc0/all_channel_counts/udimm0
        0
        /sys/devices/system/edac/mc/mc0/all_channel_counts/udimm1
        0
        /sys/devices/system/edac/mc/mc0/all_channel_counts/udimm2
        0

    What happens here is that errors on different csrows, but at the same dimm number will increment the same counter. So, in this memory mapping:

        csrow0: channel 0, dimm0
        csrow1: channel 0, dimm1
        csrow2: channel 1, dimm0
        csrow3: channel 2, dimm0

    The hardware will increment udimm0 for an error at the first dimm at either csrow0, csrow2 or csrow3;

    The hardware will increment udimm1 for an error at the second dimm at either csrow0, csrow2 or csrow3;

    The hardware will increment udimm2 for an error at the third dimm at either csrow0, csrow2 or csrow3;

4)  Standard error counters

    The standard error counters are generated when an mcelog error is received by the driver. Since, with UDIMM, this is counted by software, it is possible that some errors could be lost. With RDIMM\'s, they display the contents of the registers

## Reference documents used on `amd64_edac`

`amd64_edac` module is based on the following documents (available from <http://support.amd.com/en-us/search/tech-docs>):

1.  

    Title

    :   BIOS and Kernel Developer\'s Guide for AMD Athlon 64 and AMD Opteron Processors

    AMD publication \#

    :   26094

    Revision

    :   3.26

    Link

    :   <http://support.amd.com/TechDocs/26094.PDF>

2.  

    Title

    :   BIOS and Kernel Developer\'s Guide for AMD NPT Family 0Fh Processors

    AMD publication \#

    :   32559

    Revision

    :   3.00

    Issue Date

    :   May 2006

    Link

    :   <http://support.amd.com/TechDocs/32559.pdf>

3.  

    Title

    :   BIOS and Kernel Developer\'s Guide (BKDG) For AMD Family 10h Processors

    AMD publication \#

    :   31116

    Revision

    :   3.00

    Issue Date

    :   September 07, 2007

    Link

    :   <http://support.amd.com/TechDocs/31116.pdf>

4.  

    Title

    :   BIOS and Kernel Developer\'s Guide (BKDG) for AMD Family 15h Models 30h-3Fh Processors

    AMD publication \#

    :   49125

    Revision

    :   3.06

    Issue Date

    :   2/12/2015 (latest release)

    Link

    :   <http://support.amd.com/TechDocs/49125_15h_Models_30h-3Fh_BKDG.pdf>

5.  

    Title

    :   BIOS and Kernel Developer\'s Guide (BKDG) for AMD Family 15h Models 60h-6Fh Processors

    AMD publication \#

    :   50742

    Revision

    :   3.01

    Issue Date

    :   7/23/2015 (latest release)

    Link

    :   <http://support.amd.com/TechDocs/50742_15h_Models_60h-6Fh_BKDG.pdf>

6.  

    Title

    :   BIOS and Kernel Developer\'s Guide (BKDG) for AMD Family 16h Models 00h-0Fh Processors

    AMD publication \#

    :   48751

    Revision

    :   3.03

    Issue Date

    :   2/23/2015 (latest release)

    Link

    :   <http://support.amd.com/TechDocs/48751_16h_bkdg.pdf>

### Credits

-   Written by Doug Thompson \<<dougthompson@xmission.com>\>
    -   7 Dec 2005
    -   17 Jul 2007 Updated
-   Mauro Carvalho Chehab
    -   05 Aug 2009 Nehalem interface
    -   26 Oct 2016 Converted to ReST and cleanups at the Nehalem section
-   EDAC authors/maintainers:
    -   Doug Thompson, Dave Jiang, Dave Peterson et al,
    -   Mauro Carvalho Chehab
    -   Borislav Petkov
    -   original author: Thayne Harbaugh


[^1]: Please notice that several memory controllers allow operation on a mode called \"Lock-Step\", where it groups two memory modules together, doing 128-bit reads/writes. That gives 16 bits for error correction, with significantly improves the error correction mechanism, at the expense that, when an error happens, there\'s no way to know what memory module is to blame. So, it has to blame both memory modules.

> [^1]：请注意，几个内存控制器允许在名为“锁定步骤”的模式下操作，在该模式下，它将两个内存模块分组在一起，进行128位读/写。这为纠错提供了16位，大大改进了纠错机制，但代价是，当错误发生时，无法知道是什么内存模块造成的。因此，它不得不责怪这两个内存模块。


[^2]: Some memory controllers also allow using memory in mirror mode. On such mode, the same data is written to two memory modules. At read, the system checks both memory modules, in order to check if both provide identical data. On such configuration, when an error happens, there\'s no way to know what memory module is to blame. So, it has to blame both memory modules (or 4 memory modules, if the system is also on Lock-step mode).

> [^2]：某些内存控制器还允许在镜像模式下使用内存。在这种模式下，相同的数据被写入两个存储器模块。读取时，系统会检查两个内存模块，以检查两者是否提供相同的数据。在这样的配置中，当发生错误时，无法知道应该归咎于哪个内存模块。因此，它必须归咎于两个内存模块（或者4个内存模块，如果系统也处于锁定步骤模式）。


[^3]: For more details about the Machine Check Architecture (MCA), please read Documentation/arch/x86/x86_64/machinecheck.rst at the Kernel tree.

> [^3]：有关机器检查体系结构（MCA）的更多详细信息，请阅读内核树上的Documentation/arch/x86/x86_64/machinecheck.rst。


[^4]: Nowadays, the term DIMM (Dual In-line Memory Module) is widely used to refer to a memory module, although there are other memory packaging alternatives, like SO-DIMM, SIMM, etc. The UEFI specification (Version 2.7) defines a memory module in the Common Platform Error Record (CPER) section to be an SMBIOS Memory Device (Type 17). Along this document, and inside the EDAC subsystem, the term \"dimm\" is used for all memory modules, even when they use a different kind of packaging.

> [^4]：如今，DIMM（双列直插式内存模块）一词被广泛用于指代内存模块，尽管还有其他内存封装替代品，如SO-DIMM、SIMM等。UEFI规范（2.7版）在通用平台错误记录（CPER）部分将内存模块定义为SMBIOS内存设备（17型）。在本文档中，在EDAC子系统中，术语“dimm”用于所有内存模块，即使它们使用不同类型的封装。


[^5]: On some systems, the memory controller doesn\'t have any logic to identify the memory module. On such systems, the directory is called `rankX` and works on a similar way as the `csrowX` directories. On modern Intel memory controllers, the memory controller identifies the memory modules directly. On such systems, the directory is called `dimmX`.

> [^5]：在某些系统上，内存控制器没有任何逻辑来识别内存模块。在这样的系统中，该目录被称为“rankX”，其工作方式与“csrowX”目录类似。在现代英特尔内存控制器上，内存控制器直接识别内存模块。在这样的系统中，该目录被称为“dimmX”。


[^6]: There are also some `power` directories and `subsystem` symlinks inside the sysfs mapping that are automatically created by the sysfs subsystem. Currently, they serve no purpose.

> [^6]：在sysfs映射中还有一些由sysfs子系统自动创建的“电源”目录和“子系统”符号链接。目前，它们没有任何作用。
