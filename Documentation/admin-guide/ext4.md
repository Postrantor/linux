---
tip: translate by baidu@2024-01-30 21:33:43
---
---
title: ext4 General Information
---


Ext4 is an advanced level of the ext3 filesystem which incorporates scalability and reliability enhancements for supporting large filesystems (64 bit) in keeping with increasing disk capacities and state-of-the-art feature requirements.

> Ext4是ext3文件系统的高级版本，它结合了可扩展性和可靠性增强功能，可支持大型文件系统（64位），以满足不断增加的磁盘容量和最先进的功能要求。


Mailing list: <linux-ext4@vger.kernel.orgWeb site: <http://ext4.wiki.kernel.org>

> 邮件列表：<linux-ext4@vger.kernel.orgWeb地点http://ext4.wiki.kernel.org>

# Quick usage instructions


Note: More extensive information for getting started with ext4 can be found at the ext4 wiki site at the URL: <http://ext4.wiki.kernel.org/index.php/Ext4_Howto>

> 注意：有关开始使用ext4的更多详细信息，请访问ext4wiki网站的URL：<http://ext4.wiki.kernel.org/index.php/Ext4_Howto>

-   The latest version of e2fsprogs can be found at:

    <https://www.kernel.org/pub/linux/kernel/people/tytso/e2fsprogs/>

    or

    <http://sourceforge.net/project/showfiles.php?group_id=2406>

    or grab the latest git repository from:

<https://git.kernel.org/pub/scm/fs/ext2/e2fsprogs.git>

-   Create a new filesystem using the ext4 filesystem type:

    \# mke2fs -t ext4 /dev/hda1

    Or to configure an existing ext3 filesystem to support extents:

    \# tune2fs -O extents /dev/hda1

    If the filesystem was created with 128 byte inodes, it can be converted to use 256 byte for greater efficiency via:

    \# tune2fs -I 256 /dev/hda1

-   Mounting:

    \# mount -t ext4 /dev/hda1 /wherever


-   When comparing performance with other filesystems, it\'s always important to try multiple workloads; very often a subtle change in a workload parameter can completely change the ranking of which filesystems do well compared to others. When comparing versus ext3, note that ext4 enables write barriers by default, while ext3 does not enable write barriers by default. So it is useful to use explicitly specify whether barriers are enabled or not when via the \'-o barriers=\[0\|1\]\' mount option for both ext3 and ext4 filesystems for a fair comparison. When tuning ext3 for best benchmark numbers, it is often worthwhile to try changing the data journaling mode; \'-o data=writeback\' can be faster for some workloads. (Note however that running mounted with data=writeback can potentially leave stale data exposed in recently written files in case of an unclean shutdown, which could be a security exposure in some situations.) Configuring the filesystem with a large journal can also be helpful for metadata-intensive workloads.

> -当将性能与其他文件系统进行比较时，尝试多个工作负载总是很重要的；通常情况下，工作负载参数的细微变化会完全改变哪些文件系统与其他文件系统相比表现良好的级别。当与ext3进行比较时，请注意ext4默认启用写屏障，而ext3默认不启用写屏障。因此，当通过ext3和ext4文件系统的\'-o-barrier=\[0\|1\]\'装载选项进行公平比较时，使用显式指定是否启用屏障是很有用的。当为最佳基准数据调优ext3时，通常值得尝试更改数据日志记录模式；\'-o data=writeback\'对于某些工作负载可能会更快。（但是请注意，如果不干净地关闭，使用data=writeback运行mounted可能会在最近写入的文件中暴露过时的数据，这在某些情况下可能是一种安全暴露。）使用大型日志配置文件系统也有助于元数据密集型工作负载。

# Features

## Currently Available

-   ability to use filesystems \16TB (e2fsprogs support not available yet)
-   extent format reduces metadata overhead (RAM, IO for access, transactions)
-   extent format more robust in face of on-disk corruption due to magics,
-   internal redundancy in tree
-   improved file allocation (multi-block alloc)
-   lift 32000 subdirectory limit imposed by i_links_count\[1\]
-   nsec timestamps for mtime, atime, ctime, create time
-   inode version field on disk (NFSv4, Lustre)
-   reduced e2fsck time via uninit_bg feature
-   journal checksumming for robustness, performance
-   persistent file preallocation (e.g for streaming media, databases)
-   ability to pack bitmaps and inode tables into larger virtual groups via the flex_bg feature
-   large file support
-   inode allocation using large virtual block groups via flex_bg
-   delayed allocation
-   large block (up to pagesize) support
-   efficient new ordered mode in JBD2 and ext4 (avoid using buffer head to force the ordering)
-   Case-insensitive file name lookups
-   file-based encryption support (fscrypt)
-   file-based verity support (fsverity)


\[1\] Filesystems with a block size of 1k may see a limit imposed by the directory hash tree having a maximum depth of two.

> \[1\]块大小为1k的文件系统可能会受到最大深度为2的目录哈希树的限制。

# case-insensitive file name lookups


The case-insensitive file name lookup feature is supported on a per-directory basis, allowing the user to mix case-insensitive and case-sensitive directories in the same filesystem. It is enabled by flipping the +F inode attribute of an empty directory. The case-insensitive string match operation is only defined when we know how text in encoded in a byte sequence. For that reason, in order to enable case-insensitive directories, the filesystem must have the casefold feature, which stores the filesystem-wide encoding model used. By default, the charset adopted is the latest version of Unicode (12.1.0, by the time of this writing), encoded in the UTF-8 form. The comparison algorithm is implemented by normalizing the strings to the Canonical decomposition form, as defined by Unicode, followed by a byte per byte comparison.

> 每个目录都支持不区分大小写的文件名查找功能，允许用户在同一文件系统中混合使用不区分大小大小写和区分大小写目录。它是通过翻转空目录的+F inode属性来启用的。只有当我们知道文本在字节序列中的编码方式时，才定义不区分大小写的字符串匹配操作。因此，为了启用不区分大小写的目录，文件系统必须具有casefold功能，该功能存储所使用的文件系统范围的编码模型。默认情况下，采用的字符集是最新版本的Unicode（在撰写本文时为12.1.0），以UTF-8形式编码。比较算法是通过将字符串标准化为Unicode定义的规范分解形式，然后进行逐字节比较来实现的。


The case-awareness is name-preserving on the disk, meaning that the file name provided by userspace is a byte-per-byte match to what is actually written in the disk. The Unicode normalization format used by the kernel is thus an internal representation, and not exposed to the userspace nor to the disk, with the important exception of disk hashes, used on large case-insensitive directories with DX feature. On DX directories, the hash must be calculated using the casefolded version of the filename, meaning that the normalization format used actually has an impact on where the directory entry is stored.

> 大小写感知是在磁盘上保留名称，这意味着用户空间提供的文件名与磁盘中实际写入的文件名是逐字节匹配的。因此，内核使用的Unicode规范化格式是一种内部表示，既不向用户空间公开，也不向磁盘公开，磁盘哈希是一个重要的例外，它用于具有DX功能的不区分大小写的大型目录。在DX目录上，必须使用文件名的casefolded版本来计算哈希，这意味着所使用的规范化格式实际上会影响目录项的存储位置。


When we change from viewing filenames as opaque byte sequences to seeing them as encoded strings we need to address what happens when a program tries to create a file with an invalid name. The Unicode subsystem within the kernel leaves the decision of what to do in this case to the filesystem, which select its preferred behavior by enabling/disabling the strict mode. When Ext4 encounters one of those strings and the filesystem did not require strict mode, it falls back to considering the entire string as an opaque byte sequence, which still allows the user to operate on that file, but the case-insensitive lookups won\'t work.

> 当我们从将文件名视为不透明的字节序列改为将其视为编码字符串时，我们需要解决程序试图创建具有无效名称的文件时会发生的情况。内核中的Unicode子系统将在这种情况下做什么的决定权留给文件系统，文件系统通过启用/禁用严格模式来选择其首选行为。当Ext4遇到其中一个字符串，并且文件系统不需要严格模式时，它会将整个字符串视为不透明的字节序列，这仍然允许用户对该文件进行操作，但不区分大小写的查找不起作用。

# Options


When mounting an ext4 filesystem, the following option are accepted: (\*) == default

> 安装ext4文件系统时，接受以下选项：（\*）==默认

ro


:   Mount filesystem read only. Note that ext4 will replay the journal (and thus write to the partition) even when mounted \"read only\". The mount options \"ro,noload\" can be used to prevent writes to the filesystem.

> ：以只读方式装载文件系统。请注意，即使在挂载“只读”时，ext4也会重放日志（从而写入分区）。装载选项“ro，noload”可用于防止写入文件系统。

journal_checksum


:   Enable checksumming of the journal transactions. This will allow the recovery code in e2fsck and the kernel to detect corruption in the kernel. It is a compatible change and will be ignored by older kernels.

> ：启用日记账交易记录的校验和。这将允许e2fsck中的恢复代码和内核检测内核中的损坏。这是一个兼容的更改，将被较旧的内核忽略。

journal_async_commit


:   Commit block can be written to disk without waiting for descriptor blocks. If enabled older kernels cannot mount the device. This will enable \'journal_checksum\' internally.

> ：提交块可以在不等待描述符块的情况下写入磁盘。如果启用，旧的内核将无法装载设备。这将在内部启用“journal_checksum”。

journal_path=path, journal_dev=devnum


:   When the external journal device\'s major/minor numbers have changed, these options allow the user to specify the new journal location. The journal device is identified through either its new major/minor numbers encoded in devnum, or via a path to the device.

> ：当外部日记账设备的主要/次要编号发生更改时，这些选项允许用户指定新的日记账位置。日志设备通过其在devnum中编码的新的主要/次要编号或通过设备的路径进行识别。

norecovery, noload


:   Don\'t load the journal on mounting. Note that if the filesystem was not unmounted cleanly, skipping the journal replay will lead to the filesystem containing inconsistencies that can lead to any number of problems.

> ：不要在装载时加载轴颈。请注意，如果文件系统没有干净地卸载，则跳过日志重放将导致文件系统包含不一致，从而可能导致许多问题。

data=journal


:   All data are committed into the journal prior to being written into the main file system. Enabling this mode will disable delayed allocation and O_DIRECT support.

> ：所有数据在写入主文件系统之前都会提交到日志中。启用此模式将禁用延迟分配和O_DIRECT支持。

data=ordered (\*)


:   All data are forced directly out to the main file system prior to its metadata being committed to the journal.

> ：在将元数据提交到日志之前，所有数据都会被强制直接输出到主文件系统。

data=writeback


:   Data ordering is not preserved, data may be written into the main file system after its metadata has been committed to the journal.

> ：不保留数据排序，数据的元数据提交到日志后，数据可能会写入主文件系统。

commit=nrsec (\*)


:   This setting limits the maximum age of the running transaction to \'nrsec\' seconds. The default value is 5 seconds. This means that if you lose your power, you will lose as much as the latest 5 seconds of metadata changes (your filesystem will not be damaged though, thanks to the journaling). This default value (or any low value) will hurt performance, but it\'s good for data-safety. Setting it to 0 will have the same effect as leaving it at the default (5 seconds). Setting it to very large values will improve performance. Note that due to delayed allocation even older data can be lost on power failure since writeback of those data begins only after time set in /proc/sys/vm/dirty_expire_centisecs.

> ：此设置将正在运行的事务的最长使用期限限制为“rsec”秒。默认值为5秒。这意味着，如果您失去电源，您将失去最近5秒的元数据更改（但由于日志记录，您的文件系统不会损坏）。这个默认值（或任何低值）都会影响性能，但它有利于数据安全。将其设置为0将具有与将其保留在默认值（5秒）相同的效果。将其设置为非常大的值将提高性能。请注意，由于分配延迟，甚至较旧的数据也可能在电源故障时丢失，因为这些数据的写回仅在/proc/sys/vm/dirty_expire_centisecs中设置的时间之后才开始。

barrier=\<0\|1(*)\>, barrier(*), nobarrier


:   This enables/disables the use of write barriers in the jbd code. barrier=0 disables, barrier=1 enables. This also requires an IO stack which can support barriers, and if jbd gets an error on a barrier write, it will disable again with a warning. Write barriers enforce proper on-disk ordering of journal commits, making volatile disk write caches safe to use, at some performance penalty. If your disks are battery-backed in one way or another, disabling barriers may safely improve performance. The mount options \"barrier\" and \"nobarrier\" can also be used to enable or disable barriers, for consistency with other ext4 mount options.

> ：这将启用/禁用jbd代码中写入屏障的使用。barrier=0禁用，barrier=1启用。这还需要一个可以支持屏障的IO堆栈，如果jbd在屏障写入时出错，它将再次禁用并发出警告。写屏障强制执行日志提交的正确磁盘顺序，使易失性磁盘写缓存可以安全使用，但会带来一些性能损失。如果您的磁盘以某种方式由电池供电，禁用屏障可以安全地提高性能。装载选项\“barrier”和\“nobarrier \”也可用于启用或禁用屏障，以与其他ext4装载选项保持一致。

inode_readahead_blks=n


:   This tuning parameter controls the maximum number of inode table blocks that ext4\'s inode table readahead algorithm will pre-read into the buffer cache. The default value is 32 blocks.

> ：此调整参数控制ext4的inode表预读算法将预读取到缓冲区缓存中的最大inode表块数。默认值为32个块。

nouser_xattr


:   Disables Extended User Attributes. See the attr(5) manual page for more information about extended attributes.

> ：禁用扩展用户属性。有关扩展属性的更多信息，请参阅attr（5）手册页面。

noacl


:   This option disables POSIX Access Control List support. If ACL support is enabled in the kernel configuration (CONFIG_EXT4_FS_POSIX_ACL), ACL is enabled by default on mount. See the acl(5) manual page for more information about acl.

> ：此选项禁用POSIX访问控制列表支持。如果在内核配置（CONFIG_EXT4_FS_POSIX_ACL）中启用了ACL支持，则在装载时默认启用ACL。有关acl的更多信息，请参阅acl（5）手册页面。

bsddf (\*)

:   Make \'df\' act like BSD.

minixdf

:   Make \'df\' act like Minix.

debug

:   Extra debugging information is sent to syslog.

abort


:   Simulate the effects of calling ext4_abort() for debugging purposes. This is normally used while remounting a filesystem which is already mounted.

> ：模拟调用ext4_abort（）进行调试的效果。这通常在重新安装已安装的文件系统时使用。

errors=remount-ro

:   Remount the filesystem read-only on an error.

errors=continue

:   Keep going on a filesystem error.

errors=panic


:   Panic and halt the machine if an error occurs. (These mount options override the errors behavior specified in the superblock, which can be configured using tune2fs)

> ：如果出现错误，请惊慌失措并停止机器。（这些装载选项覆盖超级块中指定的错误行为，可以使用tune2fs进行配置）

data_err=ignore(\*)


:   Just print an error message if an error occurs in a file data buffer in ordered mode.

> ：如果在有序模式下文件数据缓冲区中发生错误，只需打印一条错误消息。

data_err=abort


:   Abort the journal if an error occurs in a file data buffer in ordered mode.

> ：如果在有序模式下文件数据缓冲区发生错误，则中止日志。

grpid \| bsdgroups

:   New objects have the group ID of their parent.

nogrpid (\*) \| sysvgroups

:   New objects have the group ID of their creator.

resgid=n

:   The group ID which may use the reserved blocks.

resuid=n

:   The user ID which may use the reserved blocks.

sb=

:   Use alternate superblock at this location.

quota, noquota, grpquota, usrquota


:   These options are ignored by the filesystem. They are used only by quota tools to recognize volumes where quota should be turned on. See documentation in the quota-tools package for more details (<http://sourceforge.net/projects/linuxquota>).

> ：文件系统将忽略这些选项。它们仅由配额工具用于识别应打开配额的卷。有关详细信息，请参阅配额工具包中的文档(<http://sourceforge.net/projects/linuxquota>).

jqfmt=\<quota type\>, usrjquota=\<file\>, grpjquota=\<file\>


:   These options tell filesystem details about quota so that quota information can be properly updated during journal replay. They replace the above quota options. See documentation in the quota-tools package for more details (<http://sourceforge.net/projects/linuxquota>).

> ：这些选项告诉文件系统有关配额的详细信息，以便在日志重放期间正确更新配额信息。它们取代了上述配额选项。有关更多详细信息，请参阅配额工具包中的文档(<http://sourceforge.net/projects/linuxquota>).

stripe=n


:   Number of filesystem blocks that mballoc will try to use for allocation size and alignment. For RAID5/6 systems this should be the number of data disks \* RAID chunk size in file system blocks.

> ：mballoc将尝试用于分配大小和对齐的文件系统块数。对于RAID5/6系统，这应该是文件系统块中数据磁盘的数量\*RAID块大小。

delalloc (\*)


:   Defer block allocation until just before ext4 writes out the block(s) in question. This allows ext4 to better allocation decisions more efficiently.

> ：将块分配推迟到ext4写出有问题的块之前。这使得ext4能够更有效地进行更好的分配决策。

nodelalloc


:   Disable delayed allocation. Blocks are allocated when the data is copied from userspace to the page cache, either via the write(2) system call or when an mmap\'ed page which was previously unallocated is written for the first time.

> ：禁用延迟分配。当通过write（2）系统调用将数据从用户空间复制到页面缓存时，或者当首次写入以前未分配的mmap页面时，就会分配块。

max_batch_time=usec


:   Maximum amount of time ext4 should wait for additional filesystem operations to be batch together with a synchronous write operation. Since a synchronous write operation is going to force a commit and then a wait for the I/O complete, it doesn\'t cost much, and can be a huge throughput win, we wait for a small amount of time to see if any other transactions can piggyback on the synchronous write. The algorithm used is designed to automatically tune for the speed of the disk, by measuring the amount of time (on average) that it takes to finish committing a transaction. Call this time the \"commit time\". If the time that the transaction has been running is less than the commit time, ext4 will try sleeping for the commit time to see if other operations will join the transaction. The commit time is capped by the max_batch_time, which defaults to 15000us (15ms). This optimization can be turned off entirely by setting max_batch_time to 0.

> ：ext4应等待额外文件系统操作与同步写入操作一起批处理的最长时间。由于同步写入操作将强制提交，然后等待I/O完成，这不会花费太多成本，而且可能会带来巨大的吞吐量优势，因此我们需要等待一小段时间，看看是否有其他事务可以利用同步写入。所使用的算法旨在通过测量完成事务提交所需的时间（平均）来自动调整磁盘速度。将此时间称为“提交时间”。如果事务运行的时间小于提交时间，ext4将尝试在提交时间内休眠，以查看其他操作是否会加入事务。提交时间的上限为max_batch_time，默认值为15000us（15ms）。通过将max_batch_time设置为0，可以完全关闭此优化。

min_batch_time=usec


:   This parameter sets the commit time (as described above) to be at least min_batch_time. It defaults to zero microseconds. Increasing this parameter may improve the throughput of multi-threaded, synchronous workloads on very fast disks, at the cost of increasing latency.

> ：此参数将提交时间（如上所述）设置为至少为min_batch_time。它默认为0微秒。增加此参数可能会以增加延迟为代价，提高速度极快的磁盘上的多线程同步工作负载的吞吐量。

journal_ioprio=prio


:   The I/O priority (from 0 to 7, where 0 is the highest priority) which should be used for I/O operations submitted by kjournald2 during a commit operation. This defaults to 3, which is a slightly higher priority than the default I/O priority.

> ：I/O优先级（从0到7，其中0是最高优先级），应用于kjournald2在提交操作期间提交的I/O操作。默认值为3，这是一个略高于默认I/O优先级的优先级。

auto_da_alloc(\*), noauto_da_alloc


:   Many broken applications don\'t use fsync() when replacing existing files via patterns such as fd = open(\"foo.new\")/write(fd,..)/close(fd)/ rename(\"foo.new\", \"foo\"), or worse yet, fd = open(\"foo\", O_TRUNC)/write(fd,..)/close(fd). If auto_da_alloc is enabled, ext4 will detect the replace-via-rename and replace-via-truncate patterns and force that any delayed allocation blocks are allocated such that at the next journal commit, in the default data=ordered mode, the data blocks of the new file are forced to disk before the rename() operation is committed. This provides roughly the same level of guarantees as ext3, and avoids the \"zero-length\" problem that can happen when a system crashes before the delayed allocation blocks are forced to disk.

> ：许多损坏的应用程序在通过模式替换现有文件时不使用fsync（），例如fd=打开（“foo.new\”）/write（fd，..）/close（fd）/rename（“foo.new\”，“foo\”），或者更糟的是，fd=打开，ext4将通过rename和truncate模式检测replace，并强制分配任何延迟的分配块，以便在下一次日志提交时，在默认data=ordered模式下，在提交rename（）操作之前，将新文件的数据块强制放到磁盘上。这提供了与ext3大致相同的保证级别，并避免了在延迟分配块被强制到磁盘之前系统崩溃时可能发生的“零长度”问题。

noinit_itable


:   Do not initialize any uninitialized inode table blocks in the background. This feature may be used by installation CD\'s so that the install process can complete as quickly as possible; the inode table initialization process would then be deferred until the next time the file system is unmounted.

> ：不要在后台初始化任何未初始化的inode表块。安装CD可以使用此功能，以便尽快完成安装过程；inode表初始化过程将推迟到下一次卸载文件系统时进行。

init_itable=n


:   The lazy itable init code will wait n times the number of milliseconds it took to zero out the previous block group\'s inode table. This minimizes the impact on the system performance while file system\'s inode table is being initialized.

> ：惰性init代码将等待n倍于将上一个块组的inode表清零所花费的毫秒数。这将在初始化文件系统的inode表时最大限度地减少对系统性能的影响。

discard, nodiscard(\*)


:   Controls whether ext4 should issue discard/TRIM commands to the underlying block device when blocks are freed. This is useful for SSD devices and sparse/thinly-provisioned LUNs, but it is off by default until sufficient testing has been done.

> ：控制释放块时ext4是否应向底层块设备发出丢弃/TRIM命令。这对于SSD设备和稀疏/精简配置的LUN很有用，但在进行足够的测试之前，它在默认情况下是关闭的。

nouid32


:   Disables 32-bit UIDs and GIDs. This is for interoperability with older kernels which only store and expect 16-bit values.

> ：禁用32位UID和GID。这是为了与只存储和期望16位值的旧内核进行互操作。

block_validity(\*), noblock_validity


:   These options enable or disable the in-kernel facility for tracking filesystem metadata blocks within internal data structures. This allows multi- block allocator and other routines to notice bugs or corrupted allocation bitmaps which cause blocks to be allocated which overlap with filesystem metadata blocks.

> ：这些选项启用或禁用用于跟踪内部数据结构中的文件系统元数据块的内核内功能。这允许多块分配器和其他例程注意到错误或损坏的分配位图，这些错误或损坏导致要分配的块与文件系统元数据块重叠。

dioread_lock, dioread_nolock


:   Controls whether or not ext4 should use the DIO read locking. If the dioread_nolock option is specified ext4 will allocate uninitialized extent before buffer write and convert the extent to initialized after IO completes. This approach allows ext4 code to avoid using inode mutex, which improves scalability on high speed storages. However this does not work with data journaling and dioread_nolock option will be ignored with kernel warning. Note that dioread_nolock code path is only used for extent-based files. Because of the restrictions this options comprises it is off by default (e.g. dioread_lock).

> ：控制ext4是否应使用DIO读取锁定。如果指定了dioread_nolock选项，ext4将在缓冲区写入之前分配未初始化的数据块，并在IO完成后将数据块转换为初始化数据块。这种方法允许ext4代码避免使用inode互斥，从而提高了高速存储的可伸缩性。然而，这不适用于数据日志记录，并且diored_nolock选项将被忽略，并发出内核警告。请注意，diored_nolock代码路径仅用于基于扩展数据块的文件。由于此选项包含的限制，它在默认情况下是关闭的（例如diored_lock）。

max_dir_size_kb=n


:   This limits the size of directories so that any attempt to expand them beyond the specified limit in kilobytes will cause an ENOSPC error. This is useful in memory constrained environments, where a very large directory can cause severe performance problems or even provoke the Out Of Memory killer. (For example, if there is only 512mb memory available, a 176mb directory may seriously cramp the system\'s style.)

> ：这会限制目录的大小，因此任何试图将其扩展到以KB为单位的指定限制之外的操作都会导致ENOSPC错误。这在内存受限的环境中非常有用，在这种环境中，非常大的目录可能会导致严重的性能问题，甚至引发内存不足的杀手。（例如，如果只有512mb的可用内存，176mb的目录可能会严重限制系统的风格。）

i_version

:   Enable 64-bit inode version support. This option is off by default.

dax


:   Use direct access (no page cache). See Documentation/filesystems/dax.rst. Note that this option is incompatible with data=journal.

> ：使用直接访问（无页面缓存）。请参阅Documentation/filess/dax.rst。请注意，此选项与data=journal不兼容。

inlinecrypt


:   When possible, encrypt/decrypt the contents of encrypted files using the blk-crypto framework rather than filesystem-layer encryption. This allows the use of inline encryption hardware. The on-disk format is unaffected. For more details, see Documentation/block/inline-encryption.rst.

> ：在可能的情况下，使用blk加密框架而不是文件系统层加密来加密/解密加密文件的内容。这允许使用内联加密硬件。磁盘上的格式不受影响。有关更多详细信息，请参见Documentation/block/inline-encryption.rst。

# Data Mode

There are 3 different data modes:

-   writeback mode

    In data=writeback mode, ext4 does not journal data at all. This mode provides a similar level of journaling as that of XFS, JFS, and ReiserFS in its default mode - metadata journaling. A crash+recovery can cause incorrect data to appear in files which were written shortly before the crash. This mode will typically provide the best ext4 performance.

-   ordered mode

    In data=ordered mode, ext4 only officially journals metadata, but it logically groups metadata information related to data changes with the data blocks into a single unit called a transaction. When it\'s time to write the new metadata out to disk, the associated data blocks are written first. In general, this mode performs slightly slower than writeback but significantly faster than journal mode.

-   journal mode

    data=journal mode provides full data and metadata journaling. All new data is written to the journal first, and then to its final location. In the event of a crash, the journal can be replayed, bringing both data and metadata into a consistent state. This mode is the slowest except when data needs to be read from and written to disk at the same time where it outperforms all others modes. Enabling this mode will disable delayed allocation and O_DIRECT support.

# /proc entries


Information about mounted ext4 file systems can be found in /proc/fs/ext4. Each mounted filesystem will have a directory in /proc/fs/ext4 based on its device name (i.e., /proc/fs/ext4/hdc or /proc/fs/ext4/dm-0). The files in each per-device directory are shown in table below.

> 有关挂载的ext4文件系统的信息可以在/proc/fs/ext4中找到。每个挂载的文件系统都将根据其设备名称（即/proc/fs/ext4/hdc或/proc/fs/ext4/dm-0）在/proc/fsext4中有一个目录。每个设备目录中的文件如下表所示。

Files in /proc/fs/ext4/\<devname\>

mb_groups

:   details of multiblock allocator buddy cache of free blocks

# /sys entries


Information about mounted ext4 file systems can be found in /sys/fs/ext4. Each mounted filesystem will have a directory in /sys/fs/ext4 based on its device name (i.e., /sys/fs/ext4/hdc or /sys/fs/ext4/dm-0). The files in each per-device directory are shown in table below.

> 有关安装的ext4文件系统的信息可以在/sys/fs/ext4中找到。每个安装的文件系统都将根据其设备名称（即/sys/fs/ext4/hdc或/sys/fs/ext4/dm-0）在/sys/fs/sext4中有一个目录。每个设备目录中的文件如下表所示。

Files in /sys/fs/ext4/\<devname\>:

(see also Documentation/ABI/testing/sysfs-fs-ext4)

delayed_allocation_blocks


:   This file is read-only and shows the number of blocks that are dirty in the page cache, but which do not have their location in the filesystem allocated yet.

> ：此文件是只读的，显示页面缓存中脏块的数量，但这些块在文件系统中的位置尚未分配。

inode_goal


:   Tuning parameter which (if non-zero) controls the goal inode used by the inode allocator in preference to all other allocation heuristics. This is intended for debugging use only, and should be 0 on production systems.

> ：优化参数，该参数（如果非零）控制索引节点分配器使用的目标索引节点，优先于所有其他分配启发式方法。这仅用于调试，在生产系统上应为0。

inode_readahead_blks


:   Tuning parameter which controls the maximum number of inode table blocks that ext4\'s inode table readahead algorithm will pre-read into the buffer cache.

> ：调整参数，用于控制ext4的inode表预读算法将预读取到缓冲区缓存中的最大inode表块数。

lifetime_write_kbytes


:   This file is read-only and shows the number of kilobytes of data that have been written to this filesystem since it was created.

> ：此文件是只读的，显示自创建文件系统以来写入该文件系统的数据的千字节数。

max_writeback_mb_bump


:   The maximum number of megabytes the writeback code will try to write out before move on to another inode.

> ：在移到另一个索引节点之前，写回代码将尝试写入的最大兆字节数。

mb_group_prealloc


:   The multiblock allocator will round up allocation requests to a multiple of this tuning parameter if the stripe size is not set in the ext4 superblock

> ：如果ext4超级块中没有设置条带大小，则多块分配器会将分配请求四舍五入为该调优参数的倍数

mb_max_to_scan


:   The maximum number of extents the multiblock allocator will search to find the best extent.

> ：多块分配器将搜索以查找最佳数据块的最大数据块数。

mb_min_to_scan


:   The minimum number of extents the multiblock allocator will search to find the best extent.

> ：多块分配器将搜索以查找最佳数据块的最小数据块数。

mb_order2_req


:   Tuning parameter which controls the minimum size for requests (as a power of 2) where the buddy cache is used.

> ：调整参数，用于控制使用伙伴缓存的请求的最小大小（2的幂）。

mb_stats


:   Controls whether the multiblock allocator should collect statistics, which are shown during the unmount. 1 means to collect statistics, 0 means not to collect statistics.

> ：控制多块分配器是否应收集在卸载过程中显示的统计信息。1表示收集统计信息，0表示不收集统计信息。

mb_stream_req


:   Files which have fewer blocks than this tunable parameter will have their blocks allocated out of a block group specific preallocation pool, so that small files are packed closely together. Each large file will have its blocks allocated out of its own unique preallocation pool.

> ：块数少于此可调参数的文件将从特定于块组的预分配池中分配块，以便将小文件紧密地打包在一起。每个大文件都将从其唯一的预分配池中分配其块。

session_write_kbytes


:   This file is read-only and shows the number of kilobytes of data that have been written to this filesystem since it was mounted.

> ：此文件是只读的，显示自装入此文件系统以来写入该文件系统的数据的千字节数。

reserved_clusters


:   This is RW file and contains number of reserved clusters in the file system which will be used in the specific situations to avoid costly zeroout, unexpected ENOSPC, or possible data loss. The default is 2% or 4096 clusters, whichever is smaller and this can be changed however it can never exceed number of clusters in the file system. If there is not enough space for the reserved space when mounting the file mount will \_[not]() fail.

> ：这是RW文件，包含文件系统中保留的簇数，这些簇将在特定情况下使用，以避免代价高昂的清零、意外的ENOSPC或可能的数据丢失。默认值为2%或4096个集群，以较小者为准，此值可以更改，但永远不能超过文件系统中的集群数。如果装载时没有足够的空间用于保留空间，则文件装载将\_[not]（）失败。

# Ioctls


Ext4 implements various ioctls which can be used by applications to access ext4-specific functionality. An incomplete list of these ioctls is shown in the table below. This list includes truly ext4-specific ioctls (`EXT4_IOC_*`) as well as ioctls that may have been ext4-specific originally but are now supported by some other filesystem(s) too (`FS_IOC_*`).

> Ext4实现了各种ioctl，应用程序可以使用这些ioctl来访问Ext4特定的功能。下表显示了这些ioctl的不完整列表。此列表包括真正的ext4特定ioctl（`ext4_IOC_*`），以及最初可能是ext4特定的ioctl，但现在也被一些其他文件系统支持（`FS_IOC_*`。

Table of Ext4 ioctls

FS_IOC_GETFLAGS


:   Get additional attributes associated with inode. The ioctl argument is an integer bitfield, with bit values described in ext4.h.

> ：获取与inode关联的其他属性。ioctl参数是一个整数位字段，其位值在ext4.h中描述。

FS_IOC_SETFLAGS


:   Set additional attributes associated with inode. The ioctl argument is an integer bitfield, with bit values described in ext4.h.

> ：设置与inode关联的其他属性。ioctl参数是一个整数位字段，其位值在ext4.h中描述。

EXT4_IOC_GETVERSION, EXT4_IOC_GETVERSION_OLD


:   Get the inode i_generation number stored for each inode. The i_generation number is normally changed only when new inode is created and it is particularly useful for network filesystems. The \'\_OLD\' version of this ioctl is an alias for FS_IOC_GETVERSION.

> ：获取为每个索引节点存储的索引节点i_generation编号。i_generation编号通常只有在创建新的inode时才会更改，它对网络文件系统特别有用。此ioctl的“\\_OLD”版本是FS_IOC_GETVERSION的别名。

EXT4_IOC_SETVERSION, EXT4_IOC_SETVERSION_OLD


:   Set the inode i_generation number stored for each inode. The \'\_OLD\' version of this ioctl is an alias for FS_IOC_SETVERSION.

> ：设置为每个索引节点存储的索引节点i_generation编号。此ioctl的“\\_OLD”版本是FS_IOC_SETVERSION的别名。

EXT4_IOC_GROUP_EXTEND


:   This ioctl has the same purpose as the resize mount option. It allows to resize filesystem to the end of the last existing block group, further resize has to be done with resize2fs, either online, or offline. The argument points to the unsigned logn number representing the filesystem new block count.

> ：此ioctl与resize mount选项具有相同的用途。它允许将文件系统的大小调整到最后一个现有块组的末尾，进一步的大小调整必须使用resize2fs完成，无论是在线还是离线。该参数指向无符号的logn数，表示文件系统的新块计数。

EXT4_IOC_MOVE_EXT


:   Move the block extents from orig_fd (the one this ioctl is pointing to) to the donor_fd (the one specified in move_extent structure passed as an argument to this ioctl). Then, exchange inode metadata between orig_fd and donor_fd. This is especially useful for online defragmentation, because the allocator has the opportunity to allocate moved blocks better, ideally into one contiguous extent.

> ：将块扩展数据块从orig_fd（此ioctl所指向的）移动到donor_fd（作为参数传递到此ioctl的Move_extent结构中指定的）。然后，在orig_fd和donor_fd之间交换inode元数据。这对于在线碎片整理特别有用，因为分配器有机会更好地分配移动的块，最好是分配到一个连续的数据块中。

EXT4_IOC_GROUP_ADD


:   Add a new group descriptor to an existing or new group descriptor block. The new group descriptor is described by ext4_new_group_input structure, which is passed as an argument to this ioctl. This is especially useful in conjunction with EXT4_IOC_GROUP_EXTEND, which allows online resize of the filesystem to the end of the last existing block group. Those two ioctls combined is used in userspace online resize tool (e.g. resize2fs).

> ：将新组描述符添加到现有或新组描述符块中。新的组描述符由ext4_new_group_input结构描述，该结构作为参数传递给该ioctl。这与EXT4_IOC_GROUP_EXTEND结合使用尤其有用，它允许将文件系统的在线大小调整到最后一个现有块组的末尾。这两个ioctl的组合用于用户空间在线调整大小工具（例如resize2fs）。

EXT4_IOC_MIGRATE


:   This ioctl operates on the filesystem itself. It converts (migrates) ext3 indirect block mapped inode to ext4 extent mapped inode by walking through indirect block mapping of the original inode and converting contiguous block ranges into ext4 extents of the temporary inode. Then, inodes are swapped. This ioctl might help, when migrating from ext3 to ext4 filesystem, however suggestion is to create fresh ext4 filesystem and copy data from the backup. Note, that filesystem has to support extents for this ioctl to work.

> ：此ioctl在文件系统本身上操作。它通过遍历原始索引节点的间接块映射并将连续块范围转换为临时索引节点的ext4范围，将ext3间接块映射索引节点转换（迁移）为ext4范围映射索引节点。然后，交换索引节点。当从ext3文件系统迁移到ext4文件系统时，这个ioctl可能会有所帮助，但建议创建新的ext4文件并从备份中复制数据。请注意，该文件系统必须支持扩展数据块才能使ioctl工作。

EXT4_IOC_ALLOC_DA_BLKS


:   Force all of the delay allocated blocks to be allocated to preserve application-expected ext3 behaviour. Note that this will also start triggering a write of the data blocks, but this behaviour may change in the future as it is not necessary and has been done this way only for sake of simplicity.

> ：强制分配所有延迟分配的块，以保留应用程序预期的ext3行为。请注意，这也将开始触发对数据块的写入，但这种行为在未来可能会发生变化，因为这不是必要的，并且只是为了简单起见才这样做。

EXT4_IOC_RESIZE_FS


:   Resize the filesystem to a new size. The number of blocks of resized filesystem is passed in via 64 bit integer argument. The kernel allocates bitmaps and inode table, the userspace tool thus just passes the new number of blocks.

> ：将文件系统调整为新大小。调整大小的文件系统的块数通过64位整数参数传入。内核分配位图和索引节点表，因此用户空间工具只传递新的块数。

EXT4_IOC_SWAP_BOOT


:   Swap i_blocks and associated attributes (like i_blocks, i_size, i_flags, \...) from the specified inode with inode EXT4_BOOT_LOADER_INO (#5). This is typically used to store a boot loader in a secure part of the filesystem, where it can\'t be changed by a normal user by accident. The data blocks of the previous boot loader will be associated with the given inode.

> ：使用索引节点EXT4_BOOT_LOADER_NO（#5）交换指定索引节点的i_block和相关属性（如i_block、i_size、i_flags等）。这通常用于将引导加载程序存储在文件系统的安全部分，普通用户不会意外更改它。上一个引导加载程序的数据块将与给定的inode相关联。

# References

kernel source: \<<file:fs/ext4/>\>

:   \<<file:fs/jbd2/>\>

programs: <http://e2fsprogs.sourceforge.net/>

useful links: <https://fedoraproject.org/wiki/ext3-devel>


:   <http://www.bullopensource.org/ext4/<http://ext4.wiki.kernel.org/index.php/Main_Page<https://fedoraproject.org/wiki/Features/Ext4>

> :   <http://www.bullopensource.org/ext4/<http://ext4.wiki.kernel.org/index.php/Main_Page<https://fedoraproject.org/wiki/Features/Ext4>
