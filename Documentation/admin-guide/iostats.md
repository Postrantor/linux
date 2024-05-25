---
tip: translate by baidu@2024-01-30 21:34:48
---
---
title: I/O statistics fields
---


Since 2.4.20 (and some versions before, with patches), and 2.5.45, more extensive disk statistics have been introduced to help measure disk activity. Tools such as `sar` and `iostat` typically interpret these and do the work for you, but in case you are interested in creating your own tools, the fields are explained here.

> 自2.4.20（以及之前的一些版本，带有补丁）和2.5.45以来，引入了更广泛的磁盘统计数据来帮助测量磁盘活动。“sar”和“iostat”等工具通常会解释这些内容并为您完成工作，但如果您有兴趣创建自己的工具，则会在此处解释这些字段。


In 2.4 now, the information is found as additional fields in `/proc/partitions`. In 2.6 and upper, the same information is found in two places: one is in the file `/proc/diskstats`, and the other is within the sysfs file system, which must be mounted in order to obtain the information. Throughout this document we\'ll assume that sysfs is mounted on `/sys`, although of course it may be mounted anywhere. Both `/proc/diskstats` and sysfs use the same source for the information and so should not differ.

> 现在，在2.4中，信息作为附加字段出现在“/proc/partitions”中。在2.6及更高版本中，在两个位置可以找到相同的信息：一个在文件“/proc/diskstats”中，另一个在sysfs文件系统中，必须安装该文件系统才能获得信息。在整个文档中，我们将假设sysfs安装在“/sys”上，当然它也可以安装在任何地方。“/proc/diskstats”和sysfs使用相同的信息源，因此不应存在差异。

Here are examples of these different formats:

    2.4:
       3     0   39082680 hda 446216 784926 9550688 4382310 424847 312726 5922052 19310380 0 3376340 23705160
       3     1    9221278 hda1 35486 0 35496 38030 0 0 0 0 0 38030 38030

    2.6+ sysfs:
       446216 784926 9550688 4382310 424847 312726 5922052 19310380 0 3376340 23705160
       35486    38030    38030    38030

    2.6+ diskstats:
       3    0   hda 446216 784926 9550688 4382310 424847 312726 5922052 19310380 0 3376340 23705160
       3    1   hda1 35486 38030 38030 38030

    4.18+ diskstats:
       3    0   hda 446216 784926 9550688 4382310 424847 312726 5922052 19310380 0 3376340 23705160 0 0 0 0


On 2.4 you might execute `grep 'hda ' /proc/partitions`. On 2.6+, you have a choice of `cat /sys/block/hda/stat` or `grep 'hda ' /proc/diskstats`.

> 在2.4上，您可以执行“grep”hda“/proc/partitions”。在2.6+上，您可以选择“cat/sys/block/hda/stat”或“grep”hda“/proc/diskstats”。


The advantage of one over the other is that the sysfs choice works well if you are watching a known, small set of disks. `/proc/diskstats` may be a better choice if you are watching a large number of disks because you\'ll avoid the overhead of 50, 100, or 500 or more opens/closes with each snapshot of your disk statistics.

> 其中一个优于另一个的优点是，如果您正在观察一组已知的小磁盘，那么sysfs选项可以很好地工作`/如果您正在观看大量磁盘，proc/diskstats`可能是一个更好的选择，因为您可以避免磁盘统计信息的每个快照打开/关闭50、100或500个或更多的开销。


In 2.4, the statistics fields are those after the device name. In the above example, the first field of statistics would be 446216. By contrast, in 2.6+ if you look at `/sys/block/hda/stat`, you\'ll find just the 15 fields, beginning with 446216. If you look at `/proc/diskstats`, the 15 fields will be preceded by the major and minor device numbers, and device name. Each of these formats provides 15 fields of statistics, each meaning exactly the same things. All fields except field 9 are cumulative since boot. Field 9 should go to zero as I/Os complete; all others only increase (unless they overflow and wrap). Wrapping might eventually occur on a very busy or long-lived system; so applications should be prepared to deal with it. Regarding wrapping, the types of the fields are either unsigned int (32 bit) or unsigned long (32-bit or 64-bit, depending on your machine) as noted per-field below. Unless your observations are very spread in time, these fields should not wrap twice before you notice it.

> 在2.4中，统计信息字段是在设备名称之后的字段。在上面的例子中，第一个统计字段是446216。相比之下，在2.6+中，如果你查看“/sys/block/hda/stat”，你会发现只有15个字段，从446216开始。如果查看“/proc/diskstats”，15个字段前面将有主要和次要设备编号以及设备名称。每种格式都提供15个统计字段，每个字段的含义完全相同。除字段9以外的所有字段都是自启动以来累积的。I/O完成时，字段9应为零；所有其他只会增加（除非它们溢出并包裹）。包裹可能最终发生在非常繁忙或寿命很长的系统上；所以应用程序应该做好处理它的准备。关于包装，字段的类型是无符号int（32位）或无符号long（32位或64位，取决于您的机器），如下文每个字段所示。除非你的观察结果在时间上非常分散，否则在你注意到之前，这些场不应该被包裹两次。


Each set of stats only applies to the indicated device; if you want system-wide stats you\'ll have to find all the devices and sum them all up.

> 每组统计数据仅适用于所指示的设备；如果你想要全系统的统计数据，你必须找到所有的设备并将其汇总。

Field 1 \-- \# of reads completed (unsigned long)

:   This is the total number of reads completed successfully.


Field 2 \-- \# of reads merged, field 6 \-- \# of writes merged (unsigned long)

> 字段2\-\#的读取被合并，字段6\-\#写入被合并（无符号长）


:   Reads and writes which are adjacent to each other may be merged for efficiency. Thus two 4K reads may become one 8K read before it is ultimately handed to the disk, and so it will be counted (and queued) as only one I/O. This field lets you know how often this was done.

> ：为了提高效率，可以合并彼此相邻的读取和写入。因此，在最终将两个4K读取交给磁盘之前，它可能会变成一个8K读取，因此它将被计数（并排队）为仅一个I/O。此字段可以让您知道执行此操作的频率。

Field 3 \-- \# of sectors read (unsigned long)

:   This is the total number of sectors read successfully.

Field 4 \-- \# of milliseconds spent reading (unsigned int)


:   This is the total number of milliseconds spent by all reads (as measured from blk_mq_alloc_request() to \_\_blk_mq_end_request()).

> ：这是所有读取所花费的总毫秒数（从blk_mq_alloc_request（）到\_\blk_mq_end_request（）测量）。

Field 5 \-- \# of writes completed (unsigned long)

:   This is the total number of writes completed successfully.

Field 6 \-- \# of writes merged (unsigned long)

:   See the description of field 2.

Field 7 \-- \# of sectors written (unsigned long)

:   This is the total number of sectors written successfully.

Field 8 \-- \# of milliseconds spent writing (unsigned int)


:   This is the total number of milliseconds spent by all writes (as measured from blk_mq_alloc_request() to \_\_blk_mq_end_request()).

> ：这是所有写入所花费的总毫秒数（从blk_mq_alloc_request（）到\_\_blk_mq_end_request（）测量）。

Field 9 \-- \# of I/Os currently in progress (unsigned int)


:   The only field that should go to zero. Incremented as requests are given to appropriate struct request_queue and decremented as they finish.

> ：唯一应为零的字段。随着请求被提供给适当的结构request_queue而递增，随着请求完成而递减。

Field 10 \-- \# of milliseconds spent doing I/Os (unsigned int)

:   This field increases so long as field 9 is nonzero.

    Since 5.0 this field counts jiffies when at least one request was started or completed. If request runs more than 2 jiffies then some I/O time might be not accounted in case of concurrent requests.

Field 11 \-- weighted \# of milliseconds spent doing I/Os (unsigned int)


:   This field is incremented at each I/O start, I/O completion, I/O merge, or read of these stats by the number of I/Os in progress (field 9) times the number of milliseconds spent doing I/O since the last update of this field. This can provide an easy measure of both I/O completion time and the backlog that may be accumulating.

> ：此字段在每次I/O开始、I/O完成、I/O合并或读取这些统计信息时递增进行中的I/O数（字段9）乘以自上次更新此字段以来执行I/O所花费的毫秒数。这可以提供I/O完成时间和可能累积的积压工作的简单度量。

Field 12 \-- \# of discards completed (unsigned long)

:   This is the total number of discards completed successfully.

Field 13 \-- \# of discards merged (unsigned long)

:   See the description of field 2

Field 14 \-- \# of sectors discarded (unsigned long)

:   This is the total number of sectors discarded successfully.

Field 15 \-- \# of milliseconds spent discarding (unsigned int)


:   This is the total number of milliseconds spent by all discards (as measured from blk_mq_alloc_request() to \_\_blk_mq_end_request()).

> ：这是所有丢弃所花费的总毫秒数（从blk_mq_alloc_request（）到\_\_blk_mq_end_request（）测量）。

Field 16 \-- \# of flush requests completed

:   This is the total number of flush requests completed successfully.

    Block layer combines flush requests and executes at most one at a time. This counts flush requests executed by disk. Not tracked for partitions.

Field 17 \-- \# of milliseconds spent flushing

:   This is the total number of milliseconds spent by all flush requests.


To avoid introducing performance bottlenecks, no locks are held while modifying these counters. This implies that minor inaccuracies may be introduced when changes collide, so (for instance) adding up all the read I/Os issued per partition should equal those made to the disks \... but due to the lack of locking it may only be very close.

> 为了避免引入性能瓶颈，在修改这些计数器时不持有锁。这意味着，当更改发生冲突时，可能会引入一些小的不准确之处，因此（例如）将每个分区发出的所有读取I/O相加，应该等于对磁盘的读取I/O。。。但是由于没有锁定，它可能只是非常接近。


In 2.6+, there are counters for each CPU, which make the lack of locking almost a non-issue. When the statistics are read, the per-CPU counters are summed (possibly overflowing the unsigned long variable they are summed to) and the result given to the user. There is no convenient user interface for accessing the per-CPU counters themselves.

> 在2.6+中，每个CPU都有计数器，这使得缺少锁定几乎不是问题。当读取统计信息时，每个CPU的计数器会被求和（可能会溢出它们被求和到的无符号长变量），并将结果提供给用户。没有方便的用户界面来访问每CPU计数器本身。


Since 4.19 request times are measured with nanoseconds precision and truncated to milliseconds before showing in this interface.

> 由于4.19的请求时间是以纳秒的精度测量的，并且在显示在此界面之前被截断为毫秒。

# Disks vs Partitions


There were significant changes between 2.4 and 2.6+ in the I/O subsystem. As a result, some statistic information disappeared. The translation from a disk address relative to a partition to the disk address relative to the host disk happens much earlier. All merges and timings now happen at the disk level rather than at both the disk and partition level as in 2.4. Consequently, you\'ll see a different statistics output on 2.6+ for partitions from that for disks. There are only *four* fields available for partitions on 2.6+ machines. This is reflected in the examples above.

> I/O子系统在2.4和2.6+之间发生了显著变化。结果，一些统计信息消失了。从相对于分区的磁盘地址到相对于主机磁盘的磁盘地址的转换发生得更早。现在，所有合并和定时都发生在磁盘级别，而不是像2.4中那样同时发生在磁盘和分区级别。因此，您将在2.6+上看到与磁盘不同的分区统计输出。在2.6以上的机器上，分区只有*4*个字段可用。这反映在上面的例子中。

Field 1 \-- \# of reads issued

:   This is the total number of reads issued to this partition.

Field 2 \-- \# of sectors read


:   This is the total number of sectors requested to be read from this partition.

> ：这是请求从此分区读取的扇区总数。

Field 3 \-- \# of writes issued

:   This is the total number of writes issued to this partition.

Field 4 \-- \# of sectors written


:   This is the total number of sectors requested to be written to this partition.

> ：这是请求写入此分区的扇区总数。


Note that since the address is translated to a disk-relative one, and no record of the partition-relative address is kept, the subsequent success or failure of the read cannot be attributed to the partition. In other words, the number of reads for partitions is counted slightly before time of queuing for partitions, and at completion for whole disks. This is a subtle distinction that is probably uninteresting for most cases.

> 请注意，由于地址被转换为磁盘相对地址，并且不保留分区相对地址的记录，因此后续读取的成功或失败不能归因于分区。换言之，分区的读取次数在分区排队之前进行计数，在整个磁盘完成时进行计数。这是一个微妙的区别，对于大多数情况来说可能是不感兴趣的。


More significant is the error induced by counting the numbers of reads/writes before merges for partitions and after for disks. Since a typical workload usually contains a lot of successive and adjacent requests, the number of reads/writes issued can be several times higher than the number of reads/writes completed.

> 更重要的是，对分区合并前和磁盘合并后的读/写次数进行计数会导致错误。由于典型的工作负载通常包含许多连续和相邻的请求，因此发出的读/写次数可能比完成的读/写入次数高出几倍。


In 2.6.25, the full statistic set is again available for partitions and disk and partition statistics are consistent again. Since we still don\'t keep record of the partition-relative address, an operation is attributed to the partition which contains the first sector of the request after the eventual merges. As requests can be merged across partition, this could lead to some (probably insignificant) inaccuracy.

> 在2.6.25中，完整的统计集再次可用于分区，并且磁盘和分区的统计信息再次一致。由于我们仍然不记录分区的相对地址，因此操作被归因于包含最终合并后请求的第一个扇区的分区。由于请求可以跨分区合并，这可能会导致一些（可能无关紧要）不准确。

# Additional notes


In 2.6+, sysfs is not mounted by default. If your distribution of Linux hasn\'t added it already, here\'s the line you\'ll want to add to your `/etc/fstab`:

> 在2.6+版本中，默认情况下不挂载sysfs。如果您的Linux发行版还没有添加，下面是您要添加到“/etc/fstab”中的行：

    none /sys sysfs defaults 0 0


In 2.6+, all disk statistics were removed from `/proc/stat`. In 2.4, they appear in both `/proc/partitions` and `/proc/stat`, although the ones in `/proc/stat` take a very different format from those in `/proc/partitions` (see proc(5), if your system has it.)

> 在2.6+中，所有磁盘统计信息都从“/proc/stat”中删除。在2.4中，它们同时出现在“/proc/partitions”和“/proc/stat”中，尽管“/proc/tat”中的格式与“/proc/partitions”中的非常不同（如果您的系统有，请参阅proc（5））。）

\-- <ricklind@us.ibm.com>
