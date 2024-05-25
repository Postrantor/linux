---
tip: translate by baidu@2024-01-30 21:36:51
---
---
title: pstore block oops/panic logger
---

# Introduction


pstore block (pstore/blk) is an oops/panic logger that writes its logs to a block device and non-block device before the system crashes. You can get these log files by mounting pstore filesystem like:

> pstore block（pstore/blk）是一个oops/panic记录器，它在系统崩溃之前将日志写入块设备和非块设备。您可以通过安装pstore文件系统来获得这些日志文件，如：

    mount -t pstore pstore /sys/fs/pstore

# pstore block concepts


pstore/blk provides efficient configuration method for pstore/blk, which divides all configurations into two parts, configurations for user and configurations for driver.

> pstore/blk为pstore/blk提供了一种高效的配置方法，它将所有配置分为两部分，用户配置和驱动配置。


Configurations for user determine how pstore/blk works, such as pmsg_size, kmsg_size and so on. All of them support both Kconfig and module parameters, but module parameters have priority over Kconfig.

> 用户的配置决定了pstore/blk的工作方式，如pmsg_size、kmsg_size等。它们都支持Kconfig和模块参数，但模块参数优先于Kconfig。


Configurations for driver are all about block device and non-block device, such as total_size of block device and read/write operations.

> 驱动程序的配置都是关于块设备和非块设备的，例如块设备的total_size和读/写操作。

# Configurations for user


All of these configurations support both Kconfig and module parameters, but module parameters have priority over Kconfig.

> 所有这些配置都支持Kconfig和模块参数，但模块参数的优先级高于Kconfig。

Here is an example for module parameters:

    pstore_blk.blkdev=/dev/mmcblk0p7 pstore_blk.kmsg_size=64 best_effort=y

The detail of each configurations may be of interest to you.

## blkdev


The block device to use. Most of the time, it is a partition of block device. It\'s required for pstore/blk. It is also used for MTD device.

> 要使用的块设备。大多数时候，它是块设备的一个分区。它是pstore/blk所必需的。它也用于MTD设备。


When pstore/blk is built as a module, \"blkdev\" accepts the following variants:

> 当pstore/blk被构建为一个模块时，\“blkdev\”接受以下变体：

1.  /dev/\<disk_name\> represents the device number of disk

2.  /dev/\<disk_name\>\<decimal\> represents the device number of partition - device number of disk plus the partition number

> 2./dev/\<disk_name\>\<decimal\>表示分区的设备号-磁盘的设备号加上分区号

3.  /dev/\<disk_name\>p\<decimal\> - same as the above; this form is used when disk name of partitioned disk ends with a digit.

> 3./dev/\<disk_name\>p\<decimal\>-同上；当分区磁盘的磁盘名称以数字结尾时，使用此形式。


When pstore/blk is built into the kernel, \"blkdev\" accepts the following variants:

> 当pstore/blk内置到内核中时，\“blkdev\”接受以下变体：


1.  \<hex_major\>\<hex_minor\> device number in hexadecimal representation, with no leading 0x, for example b302.

> 1.以十六进制表示的设备编号，不带前导0x，例如b302。

2.  PARTUUID=00112233-4455-6677-8899-AABBCCDDEEFF represents the unique id of a partition if the partition table provides it. The UUID may be either an EFI/GPT UUID, or refer to an MSDOS partition using the format SSSSSSSS-PP, where SSSSSSSS is a zero-filled hex representation of the 32-bit \"NT disk signature\", and PP is a zero-filled hex representation of the 1-based partition number.

> 2.PARTUUID=00112233-4455-6677-8899-ABBCCDDEFF表示分区的唯一id（如果分区表提供）。UUID可以是EFI/GPT UUID，也可以是使用SSSSSSSS-P格式的MSDOS分区，其中SSSSSSSS是32位“NT磁盘签名”的零填充十六进制表示，PP是基于1的分区号的零填充六进制表示。

3.  PARTUUID=\<UUID\>/PARTNROFF=\<int\> to select a partition in relation to a partition with a known unique id.

> 3.PARTUUID=\<UUID\>/PARTNROFF=\<int\>选择与具有已知唯一id的分区相关的分区。

4.  \<major\>:\<minor\> major and minor number of the device separated by a colon.

> 4.用冒号分隔的设备的主要和次要编号。

It accepts the following variants for MTD device:

1.  \<device name\> MTD device name. \"pstore\" is recommended.
2.  \<device number\> MTD device number.

## kmsg_size


The chunk size in KB for oops/panic front-end. It **MUST** be a multiple of 4. It\'s optional if you do not care about the oops/panic log.

> oops/panic前端的区块大小（KB）。它必须是4的倍数。如果您不关心oops/panic日志，这是可选的。


There are multiple chunks for oops/panic front-end depending on the remaining space except other pstore front-ends.

> 根据除其他pstore前端之外的剩余空间，有多个块用于oops/panic前端。


pstore/blk will log to oops/panic chunks one by one, and always overwrite the oldest chunk if there is no more free chunk.

> pstore/blk将一个接一个地登录到oops/panic块，如果没有更多的可用块，则始终覆盖最旧的块。

## pmsg_size


The chunk size in KB for pmsg front-end. It **MUST** be a multiple of 4. It\'s optional if you do not care about the pmsg log.

> pmsg前端的区块大小（KB）。它必须是4的倍数。如果您不关心pmsg日志，这是可选的。

Unlike oops/panic front-end, there is only one chunk for pmsg front-end.


Pmsg is a user space accessible pstore object. Writes to */dev/pmsg0* are appended to the chunk. On reboot the contents are available in */sys/fs/pstore/pmsg-pstore-blk-0*.

> Pmsg是一个用户空间可访问的pstore对象。对*/dev/pmg0*的写入被附加到块中。重新启动时，内容在*/sys/fs/pstore/pmg-pstore-blk-0*中可用。

## console_size


The chunk size in KB for console front-end. It **MUST** be a multiple of 4. It\'s optional if you do not care about the console log.

> 控制台前端的区块大小（KB）。它必须是4的倍数。如果您不关心控制台日志，这是可选的。

Similar to pmsg front-end, there is only one chunk for console front-end.


All log of console will be appended to the chunk. On reboot the contents are available in */sys/fs/pstore/console-pstore-blk-0*.

> 控制台的所有日志都将附加到块中。重新启动时，内容可在*/sys/fs/pstore/console-store-blk-0*中找到。

## ftrace_size


The chunk size in KB for ftrace front-end. It **MUST** be a multiple of 4. It\'s optional if you do not care about the ftrace log.

> ftrace前端的区块大小（KB）。它必须是4的倍数。如果您不关心ftrace日志，这是可选的。


Similar to oops front-end, there are multiple chunks for ftrace front-end depending on the count of cpu processors. Each chunk size is equal to ftrace_size / processors_count.

> 与oops前端类似，根据cpu处理器的数量，ftrace前端有多个块。每个区块大小都等于ftrace_size/processors_count。


All log of ftrace will be appended to the chunk. On reboot the contents are combined and available in */sys/fs/pstore/ftrace-pstore-blk-0*.

> ftrace的所有日志都将附加到块中。重新启动时，内容会组合在*/sys/fs/pstore/ftrace-pstore-blk-0*中。


Persistent function tracing might be useful for debugging software or hardware related hangs. Here is an example of usage:

> 持久函数跟踪可能有助于调试与软件或硬件相关的挂起。下面是一个用法示例：

    # mount -t pstore pstore /sys/fs/pstore
    # mount -t debugfs debugfs /sys/kernel/debug/
    # echo 1 > /sys/kernel/debug/pstore/record_ftrace
    # reboot -f
    [...]
    # mount -t pstore pstore /sys/fs/pstore
    # tail /sys/fs/pstore/ftrace-pstore-blk-0
    CPU:0 ts:5914676 c0063828  c0063b94  call_cpuidle <- cpu_startup_entry+0x1b8/0x1e0
    CPU:0 ts:5914678 c039ecdc  c006385c  cpuidle_enter_state <- call_cpuidle+0x44/0x48
    CPU:0 ts:5914680 c039e9a0  c039ecf0  cpuidle_enter_freeze <- cpuidle_enter_state+0x304/0x314
    CPU:0 ts:5914681 c0063870  c039ea30  sched_idle_set_state <- cpuidle_enter_state+0x44/0x314
    CPU:1 ts:5916720 c0160f59  c015ee04  kernfs_unmap_bin_file <- __kernfs_remove+0x140/0x204
    CPU:1 ts:5916721 c05ca625  c015ee0c  __mutex_lock_slowpath <- __kernfs_remove+0x148/0x204
    CPU:1 ts:5916723 c05c813d  c05ca630  yield_to <- __mutex_lock_slowpath+0x314/0x358
    CPU:1 ts:5916724 c05ca2d1  c05ca638  __ww_mutex_lock <- __mutex_lock_slowpath+0x31c/0x358

## max_reason


Limiting which kinds of kmsg dumps are stored can be controlled via the `max_reason` value, as defined in include/linux/kmsg_dump.h\'s `enum kmsg_dump_reason`. For example, to store both Oopses and Panics, `max_reason` should be set to 2 (KMSG_DUMP_OOPS), to store only Panics `max_reason` should be set to 1 (KMSG_DUMP_PANIC). Setting this to 0 (KMSG_DUMP_UNDEF), means the reason filtering will be controlled by the `printk.always_kmsg_dump` boot param: if unset, it\'ll be KMSG_DUMP_OOPS, otherwise KMSG_DUMP_MAX.

> 可以通过“max_reason”值来控制存储哪种类型的kmsg转储，该值在include/linux/kmsg_dump.h的“enum-kmsg_dmp_reason’s”中定义。例如，要同时存储Oopse和Panics，“max_reason”应设置为2（KMSG_DUMP_OOPS），要仅存储Panics，应将“max_reeson”设置为1（KMSG_DUMP_PANIC）。将其设置为0（KMSG_DUMP_UNDEF），意味着原因筛选将由“printk.always_KMSG_DUMP”引导参数控制：如果未设置，则为KMSG_DUMP_OOPS，否则为KMSG_DUMP_MAX。

# Configurations for driver


A device driver uses `register_pstore_device` with `struct pstore_device_info` to register to pstore/blk.

> 设备驱动程序使用“register_pstore_device”和“struct pstore_device_info”来注册到pstore/blk。

::: {.kernel-doc export=""}
fs/pstore/blk.c
:::

# Compression and header


Block device is large enough for uncompressed oops data. Actually we do not recommend data compression because pstore/blk will insert some information into the first line of oops/panic data. For example:

> 块设备足够大，可以容纳未压缩的oops数据。实际上，我们不建议进行数据压缩，因为pstore/blk会在oops/panic数据的第一行中插入一些信息。例如

    Panic: Total 16 times


It means that it\'s OOPSpanic since the first booting is important to judge whether the system is stable.

> 这意味着它是OOPSpanic，因为第一次启动对于判断系统是否稳定很重要。

The following line is inserted by pstore filesystem. For example:

    Oops#2 Part1

It means that it\'s OOPS for the 2nd time on the last boot.

# Reading the data


The dump data can be read from the pstore filesystem. The format for these files is `dmesg-pstore-blk-[N]` for oops/panic front-end, `pmsg-pstore-blk-0` for pmsg front-end and so on. The timestamp of the dump file records the trigger time. To delete a stored record from block device, simply unlink the respective pstore file.

> 转储数据可以从pstore文件系统中读取。这些文件的格式为`dmesg-pstore-blk-[N]`（用于oops/panic前端），`pmsg-pstore-blk-0`（用于pmsg前端），依此类推。转储文件的时间戳记录触发时间。要从块设备中删除存储的记录，只需取消相应pstore文件的链接即可。

# Attentions in panic read/write APIs


If on panic, the kernel is not going to run for much longer, the tasks will not be scheduled and most kernel resources will be out of service. It looks like a single-threaded program running on a single-core computer.

> 如果处于紧急状态，内核将不会运行更长的时间，任务将不会被调度，并且大多数内核资源将停止服务。它看起来像是在单核计算机上运行的单线程程序。

The following points require special attention for panic read/write APIs:


1.  Can **NOT** allocate any memory. If you need memory, just allocate while the block driver is initializing rather than waiting until the panic.

> 1.不能分配任何内存。如果您需要内存，只需在块驱动程序初始化时进行分配，而不是等到死机。

2.  Must be polled, **NOT** interrupt driven. No task schedule any more. The block driver should delay to ensure the write succeeds, but NOT sleep.

> 2.必须轮询，**NOT***中断驱动。不再有任务计划。块驱动程序应该延迟以确保写入成功，但不能休眠。

3.  Can **NOT** take any lock. There is no other task, nor any shared resource; you are safe to break all locks.

> 3.不能**拿走任何锁。没有其他任务，也没有任何共享资源；你可以安全地打破所有的锁。

4.  Just use CPU to transfer. Do not use DMA to transfer unless you are sure that DMA will not keep lock.

> 4.只需使用CPU即可传输。除非您确信DMA不会保持锁定，否则不要使用DMA进行传输。

5.  Control registers directly. Please control registers directly rather than use Linux kernel resources. Do I/O map while initializing rather than wait until a panic occurs.

> 5.直接控制寄存器。请直接控制寄存器，而不是使用Linux内核资源。在初始化时执行I/O映射，而不是等到出现死机。

6.  Reset your block device and controller if necessary. If you are not sure of the state of your block device and controller when a panic occurs, you are safe to stop and reset them.

> 6.如有必要，请重置您的块设备和控制器。如果您不确定发生恐慌时块设备和控制器的状态，可以安全地停止并重置它们。


pstore/blk supports psblk_blkdev_info(), which is defined in *linux/pstore_blk.h*, to get information of using block device, such as the device number, sector count and start sector of the whole disk.

> pstore/blk支持在*linux/pstore_blk.h*中定义的psblk_blkdev_info（），以获取使用块设备的信息，如整个磁盘的设备号、扇区数和起始扇区。

# pstore block internals

For developer reference, here are all the important structures and APIs:

::: {.kernel-doc internal=""}
fs/pstore/zone.c
:::

::: {.kernel-doc internal=""}
include/linux/pstore_zone.h
:::

::: {.kernel-doc internal=""}
include/linux/pstore_blk.h
:::
