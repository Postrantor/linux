---
tip: translate by baidu@2024-01-30 21:35:25
---
---
author:
- Originally Written by FlatCap - Richard Russon \<<ldm@flatcap.org>\>.
last updated: |
  Anton Altaparmakov on 30 March 2007 for Windows Vista.
title: LDM - Logical Disk Manager (Dynamic Disks)
---

# Overview


Windows 2000, XP, and Vista use a new partitioning scheme. It is a complete replacement for the MSDOS style partitions. It stores its information in a 1MiB journalled database at the end of the physical disk. The size of partitions is limited only by disk space. The maximum number of partitions is nearly 2000.

> Windows2000、XP和Vista使用了一种新的分区方案。它完全取代了MSDOS风格的分区。它将信息存储在物理磁盘末端的1MiB日志数据库中。分区的大小仅受磁盘空间的限制。最大分区数接近2000个。


Any partitions created under the LDM are called \"Dynamic Disks\". There are no longer any primary or extended partitions. Normal MSDOS style partitions are now known as Basic Disks.

> 在LDM下创建的任何分区都称为“动态磁盘”。不再有任何主分区或扩展分区。普通MSDOS风格的分区现在被称为基本磁盘。


If you wish to use Spanned, Striped, Mirrored or RAID 5 Volumes, you must use Dynamic Disks. The journalling allows Windows to make changes to these partitions and filesystems without the need to reboot.

> 如果要使用跨度卷、条带卷、镜像卷或RAID 5卷，则必须使用动态磁盘。日志记录允许Windows在不需要重新启动的情况下对这些分区和文件系统进行更改。


Once the LDM driver has divided up the disk, you can use the MD driver to assemble any multi-partition volumes, e.g. Stripes, RAID5.

> LDM驱动程序分割磁盘后，您就可以使用MD驱动程序来组装任何多分区卷，例如条带、RAID5。


To prevent legacy applications from repartitioning the disk, the LDM creates a dummy MSDOS partition containing one disk-sized partition. This is what is supported with the Linux LDM driver.

> 为了防止遗留应用程序重新分区磁盘，LDM创建了一个包含一个磁盘大小分区的虚拟MSDOS分区。这是Linux LDM驱动程序所支持的。


A newer approach that has been implemented with Vista is to put LDM on top of a GPT label disk. This is not supported by the Linux LDM driver yet.

> Vista实现的一种新方法是将LDM放在GPT标签磁盘上。Linux LDM驱动程序还不支持这一点。

# Example

Below we have a 50MiB disk, divided into seven partitions.

::: note
::: title
Note
:::


The missing 1MiB at the end of the disk is where the LDM database is stored.

> 磁盘末尾缺少的1MiB是LDM数据库的存储位置。
:::

+\-\-\-\-\-\--++\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\--+\-\-\-\--++\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\--+\-\-\--+ \| Offset Bytes \| Sectors \| MiB \|\| Size Bytes \| Sectors \| MiB\| +=======++==============+=========+=====++==============+=========+====+ \| 0 \| 0 \| 0 \|\| 52428800 \| 102400 \| 50\| +\-\-\-\-\-\--++\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\--+\-\-\-\--++\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\--+\-\-\--+ \| 51380224 \| 100352 \| 49 \|\| 1048576 \| 2048 \| 1\| +\-\-\-\-\-\--++\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\--+\-\-\-\--++\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\--+\-\-\--+ \| 16384 \| 32 \| 0 \|\| 6979584 \| 13632 \| 6\| +\-\-\-\-\-\--++\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\--+\-\-\-\--++\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\--+\-\-\--+ \| 6995968 \| 13664 \| 6 \|\| 10485760 \| 20480 \| 10\| +\-\-\-\-\-\--++\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\--+\-\-\-\--++\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\--+\-\-\--+ \| 17481728 \| 34144 \| 16 \|\| 4194304 \| 8192 \| 4\| +\-\-\-\-\-\--++\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\--+\-\-\-\--++\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\--+\-\-\--+ \| 21676032 \| 42336 \| 20 \|\| 5242880 \| 10240 \| 5\| +\-\-\-\-\-\--++\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\--+\-\-\-\--++\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\--+\-\-\--+ \| 26918912 \| 52576 \| 25 \|\| 10485760 \| 20480 \| 10\| +\-\-\-\-\-\--++\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\--+\-\-\-\--++\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\--+\-\-\--+ \| 37404672 \| 73056 \| 35 \|\| 13959168 \| 27264 \| 13\| +\-\-\-\-\-\--++\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\--+\-\-\-\--++\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\--+\-\-\--+


The LDM Database may not store the partitions in the order that they appear on disk, but the driver will sort them.

> LDM数据库可能不会按照分区在磁盘上出现的顺序存储分区，但驱动程序会对它们进行排序。

When Linux boots, you will see something like:

    hda: 102400 sectors w/32KiB Cache, CHS=50/64/32
    hda: [LDM] hda1 hda2 hda3 hda4 hda5 hda6 hda7

# Compiling LDM Support

To enable LDM, choose the following two options:

> -   \"Advanced partition selection\" CONFIG_PARTITION_ADVANCED
> -   \"Windows Logical Disk Manager (Dynamic Disk) support\" CONFIG_LDM_PARTITION


If you believe the driver isn\'t working as it should, you can enable the extra debugging code. This will produce a LOT of output. The option is:

> 如果您认为驱动程序没有正常工作，可以启用额外的调试代码。这将产生大量的产出。选项是：

> -   \"Windows LDM extra logging\" CONFIG_LDM_DEBUG

N.B. The partition code cannot be compiled as a module.


As with all the partition code, if the driver doesn\'t see signs of its type of partition, it will pass control to another driver, so there is no harm in enabling it.

> 与所有分区代码一样，如果驱动程序没有看到其分区类型的标志，它会将控制权传递给另一个驱动程序，因此启用它没有害处。


If you have Dynamic Disks but don\'t enable the driver, then all you will see is a dummy MSDOS partition filling the whole disk. You won\'t be able to mount any of the volumes on the disk.

> 如果您有动态磁盘，但没有启用驱动程序，那么您将看到一个伪MSDOS分区填充整个磁盘。您将无法在磁盘上装载任何卷。

# Booting


If you enable LDM support, then lilo is capable of booting from any of the discovered partitions. However, grub does not understand the LDM partitioning and cannot boot from a Dynamic Disk.

> 如果启用LDM支持，那么lilo能够从任何发现的分区启动。但是，grub不了解LDM分区，并且不能从动态磁盘启动。

# More Documentation


There is an Overview of the LDM together with complete Technical Documentation. It is available for download.

> LDM概述以及完整的技术文档。它可以下载。

> <http://www.linux-ntfs.org/>


If you have any LDM questions that aren\'t answered in the documentation, email me.

> 如果您有文档中没有回答的LDM问题，请给我发电子邮件。

Cheers,

:   FlatCap - Richard Russon <ldm@flatcap.org>
