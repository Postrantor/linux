---
tip: translate by baidu@2024-01-30 21:35:35
---
---
title: RAID arrays
---

# Boot time assembly of RAID arrays

Tools that manage md devices can be found at

:   <https://www.kernel.org/pub/linux/utils/raid/>

You can boot with your md device with the following kernel command lines:

for old raid arrays without persistent superblocks:

    md=<md device no.>,<raid level>,<chunk size factor>,<fault level>,dev0,dev1,...,devn

for raid arrays with persistent superblocks:

    md=<md device no.>,dev0,dev1,...,devn

or, to assemble a partitionable array:

    md=d<md device no.>,dev0,dev1,...,devn

## `md device no.`

The number of the md device

+-----------------+--------+
| `md device no.` | device |
+=================+========+
| 0             | md0  |
+-----------------+--------+
| 1 md          | 1      |
+-----------------+--------+
| 2 md          | 2      |
+-----------------+--------+
| 3 md          | 3      |
+-----------------+--------+
| 4 md          | 4      |
+-----------------+--------+

## `raid level`

level of the RAID array

  `raid level`   level
  -------------- -------
  -1 linear m    ode
  0 striped      mode

other modes are only supported with persistent super blocks

## `chunk size factor`

(raid-0 and raid-1 only)

Set the chunk size as 4k \<\< n.

## `fault level`

Totally ignored

## `dev0` to `devn`

e.g. `/dev/hda1`, `/dev/hdc1`, `/dev/sda1`, `/dev/sdb1`


A possible loadlin line (Harald Hoyer \<<HarryH@Royal.Net>\>) looks like this:

> 一条可能的载重线（Harald Hoyer\<<HarryH@Royal.Net>\>)看起来是这样的：

    e:\loadlin\loadlin e:\zimage root=/dev/md0 md=0,0,4,0,/dev/hdb2,/dev/hdc3 ro

# Boot time autodetection of RAID arrays


When md is compiled into the kernel (not as module), partitions of type 0xfd are scanned and automatically assembled into RAID arrays. This autodetection may be suppressed with the kernel parameter `raid=noautodetect`. As of kernel 2.6.9, only drives with a type 0 superblock can be autodetected and run at boot time.

> 当md被编译到内核中（而不是作为模块）时，0xfd类型的分区会被扫描并自动组装到RAID阵列中。可以使用内核参数“raid=noautodetect”来抑制这种自动检测。从2.6.9内核开始，只有类型为0的超级块的驱动器才能在启动时自动检测并运行。


The kernel parameter `raid=partitionable` (or `raid=part`) means that all auto-detected arrays are assembled as partitionable.

> 内核参数“raid=partitionable”（或“raid=part”）表示所有自动检测到的数组都被组装为partitionable。

# Boot time assembly of degraded/dirty arrays


If a raid5 or raid6 array is both dirty and degraded, it could have undetectable data corruption. This is because the fact that it is `dirty` means that the parity cannot be trusted, and the fact that it is degraded means that some datablocks are missing and cannot reliably be reconstructed (due to no parity).

> 如果raid5或raid6阵列既脏又降级，则可能存在无法检测到的数据损坏。这是因为它是“脏的”这一事实意味着奇偶校验不可信，而它被降级的事实意味着一些数据块丢失，无法可靠地重建（由于没有奇偶校验）。


For this reason, md will normally refuse to start such an array. This requires the sysadmin to take action to explicitly start the array despite possible corruption. This is normally done with:

> 因此，md通常会拒绝启动这样的数组。这要求系统管理员采取措施显式启动阵列，尽管可能发生损坏。这通常是通过以下方式完成的：

    mdadm --assemble --force ....


This option is not really available if the array has the root filesystem on it. In order to support this booting from such an array, md supports a module parameter `start_dirty_degraded` which, when set to 1, bypassed the checks and will allows dirty degraded arrays to be started.

> 如果阵列上有根文件系统，则此选项实际上不可用。为了支持从这样的阵列启动，md支持模块参数“start_dirty_dialized”，当设置为1时，该参数将绕过检查，并允许脏的降级阵列启动。

So, to boot with a root filesystem of a dirty degraded raid 5 or 6, use:

    md-mod.start_dirty_degraded=1

# Superblock formats


The md driver can support a variety of different superblock formats. Currently, it supports superblock formats `0.90.0` and the `md-1` format introduced in the 2.5 development series.

> md驱动程序可以支持各种不同的超级块格式。目前，它支持超级块格式“0.90.0”和2.5开发系列中引入的“md-1”格式。

The kernel will autodetect which format superblock is being used.


Superblock format `0` is treated differently to others for legacy reasons - it is the original superblock format.

> 由于遗留的原因，超级块格式“0”与其他格式不同——它是原始的超级块格式。

# General Rules - apply for all superblock formats

An array is `created` by writing appropriate superblocks to all devices.


It is `assembled` by associating each of these devices with an particular md virtual device. Once it is completely assembled, it can be accessed.

> 它是通过将这些设备中的每一个与特定的md虚拟设备相关联来“组装”的。一旦它完全组装好，就可以访问它。


An array should be created by a user-space tool. This will write superblocks to all devices. It will usually mark the array as `unclean`, or with some devices missing so that the kernel md driver can create appropriate redundancy (copying in raid 1, parity calculation in raid 4/5).

> 数组应该由用户空间工具创建。这将向所有设备写入超级块。它通常会将数组标记为“不干净”，或者缺少一些设备，以便内核md驱动程序可以创建适当的冗余（在raid 1中进行复制，在raid 4/5中进行奇偶校验计算）。


When an array is assembled, it is first initialized with the SET_ARRAY_INFO ioctl. This contains, in particular, a major and minor version number. The major version number selects which superblock format is to be used. The minor number might be used to tune handling of the format, such as suggesting where on each device to look for the superblock.

> 组装数组时，首先使用SET_array_INFO ioctl对其进行初始化。其中特别包含主要版本号和次要版本号。主版本号选择要使用的超级块格式。次要数字可能用于调整格式的处理，例如建议在每个设备上的何处查找超级块。


Then each device is added using the ADD_NEW_DISK ioctl. This provides, in particular, a major and minor number identifying the device to add.

> 然后使用ADD_NEW_DISK ioctl添加每个设备。这尤其提供了标识要添加的设备的主要和次要编号。

The array is started with the RUN_ARRAY ioctl.


Once started, new devices can be added. They should have an appropriate superblock written to them, and then be passed in with ADD_NEW_DISK.

> 一旦启动，就可以添加新设备。它们应该有一个适当的超级块写入，然后与ADD_NEW_DISK一起传入。


Devices that have failed or are not yet active can be detached from an array using HOT_REMOVE_DISK.

> 可以使用HOT_REMOVE_DISK将出现故障或尚未激活的设备从阵列中分离。

# Specific Rules that apply to format-0 super block arrays, and arrays with no superblock (non-persistent)


An array can be `created` by describing the array (level, chunksize etc) in a SET_ARRAY_INFO ioctl. This must have `major_version==0` and `raid_disks != 0`.

> 可以通过在SET_array_INFO ioctl中描述数组（级别、块大小等）来“创建”数组。它必须具有`major_version==0`和`raid_disks！=0`.


Then uninitialized devices can be added with ADD_NEW_DISK. The structure passed to ADD_NEW_DISK must specify the state of the device and its role in the array.

> 然后可以使用ADD_NEW_DISK添加未初始化的设备。传递给ADD_NEW_DISK的结构必须指定设备的状态及其在数组中的角色。


Once started with RUN_ARRAY, uninitialized spares can be added with HOT_ADD_DISK.

> 使用RUN_ARRAY启动后，可以使用HOT_ADD_DISK添加未初始化的备件。

# MD devices in sysfs

md devices appear in sysfs (`/sys`) as regular block devices, e.g.:

    /sys/block/md0


Each `md` device will contain a subdirectory called `md` which contains further md-specific information about the device.

> 每个“md”设备将包含一个名为“md”的子目录，其中包含有关该设备的更多md特定信息。

All md devices contain:

 level


 :   a text file indicating the `raid level`. e.g. raid0, raid1, raid5, linear, multipath, faulty. If no raid level has been set yet (array is still being assembled), the value will reflect whatever has been written to it, which may be a name like the above, or may be a number such as `0`, `5`, etc.

> ：表示“raid级别”的文本文件。例如raid0、raid1、raid5、线性、多路径、故障。如果尚未设置raid级别（数组仍在组装中），则该值将反映已写入其中的内容，该值可以是与上面类似的名称，也可以是诸如“0”、“5”等数字。

 raid_disks


 :   a text file with a simple number indicating the number of devices in a fully functional array. If this is not yet known, the file will be empty. If an array is being resized this will contain the new number of devices. Some raid levels allow this value to be set while the array is active. This will reconfigure the array. Otherwise it can only be set while assembling an array. A change to this attribute will not be permitted if it would reduce the size of the array. To reduce the number of drives in an e.g. raid5, the array size must first be reduced by setting the `array_size` attribute.

> ：一个文本文件，带有一个简单的数字，表示功能齐全的阵列中的设备数量。如果还不知道这一点，则该文件将为空。如果正在调整阵列的大小，则会包含新数量的设备。某些raid级别允许在阵列处于活动状态时设置此值。这将重新配置阵列。否则，只能在组装数组时进行设置。如果此属性会减小数组的大小，则不允许对此属性进行更改。要减少raid5中的驱动器数量，必须首先通过设置“array_size”属性来减少阵列大小。

 chunk_size


 :   This is the size in bytes for `chunks` and is only relevant to raid levels that involve striping (0,4,5,6,10). The address space of the array is conceptually divided into chunks and consecutive chunks are striped onto neighbouring devices. The size should be at least PAGE_SIZE (4k) and should be a power of 2. This can only be set while assembling an array

> ：这是“chunks”的字节大小，仅与涉及条带化的raid级别（0,4,5,6,10）相关。阵列的地址空间在概念上被划分为块，并且连续的块被分带到相邻的设备上。大小至少应为PAGE_size（4k），并且应为2的幂。这只能在组装阵列时设置

 layout


 :   The `layout` for the array for the particular level. This is simply a number that is interpreted differently by different levels. It can be written while assembling an array.

> ：特定级别的数组的“布局”。这只是一个不同层次对其解释不同的数字。它可以在组装数组时写入。

 array_size


 :   This can be used to artificially constrain the available space in the array to be less than is actually available on the combined devices. Writing a number (in Kilobytes) which is less than the available size will set the size. Any reconfiguration of the array (e.g. adding devices) will not cause the size to change. Writing the word `default` will cause the effective size of the array to be whatever size is actually available based on `level`, `chunk_size` and `component_size`.

> ：这可用于人为地将阵列中的可用空间限制为小于组合设备上的实际可用空间。写入一个小于可用大小的数字（以千字节为单位）将设置大小。阵列的任何重新配置（例如添加设备）都不会导致大小发生变化。写入单词“default”将导致数组的有效大小为基于“level”、“chunk_size”和“component_size”的实际可用大小。

     This can be used to reduce the size of the array before reducing the number of devices in a raid4/5/6, or to support external metadata formats which mandate such clipping.

 reshape_position


 :   This is either `none` or a sector number within the devices of the array where `reshape` is up to. If this is set, the three attributes mentioned above (raid_disks, chunk_size, layout) can potentially have 2 values, an old and a new value. If these values differ, reading the attribute returns:

> ：这要么是“none”，要么是数组设备中“整形”最大值的扇区号。如果设置了这一点，上述三个属性（raid_disks、chunk_size、layout）可能有两个值，一个旧值和一个新值。如果这些值不同，读取属性将返回：

         new (old)

     and writing will effect the `new` value, leaving the `old` unchanged.

 component_size


 :   For arrays with data redundancy (i.e. not raid0, linear, faulty, multipath), all components must be the same size - or at least there must a size that they all provide space for. This is a key part or the geometry of the array. It is measured in sectors and can be read from here. Writing to this value may resize the array if the personality supports it (raid1, raid5, raid6), and if the component drives are large enough.

> ：对于具有数据冗余的阵列（即非raid0、线性、故障、多路径），所有组件的大小必须相同，或者至少必须有一个它们都能提供空间的大小。这是阵列的关键部分或几何图形。它是以扇区为单位测量的，可以从这里读取。如果个性支持（raid1、raid5、raid6），并且组件驱动器足够大，写入该值可能会调整阵列的大小。

 metadata_version


 :   This indicates the format that is being used to record metadata about the array. It can be 0.90 (traditional format), 1.0, 1.1, 1.2 (newer format in varying locations) or `none` indicating that the kernel isn\'t managing metadata at all. Alternately it can be `external:` followed by a string which is set by user-space. This indicates that metadata is managed by a user-space program. Any device failure or other event that requires a metadata update will cause array activity to be suspended until the event is acknowledged.

> ：这表示用于记录有关数组的元数据的格式。它可以是0.90（传统格式）、1.0、1.1、1.2（不同位置的较新格式）或“none”，表示内核根本不管理元数据。或者，它可以是“external:”，后面跟着一个由用户空间设置的字符串。这表示元数据由用户空间程序管理。任何设备故障或其他需要元数据更新的事件都将导致阵列活动暂停，直到事件得到确认。

 resync_start


 :   The point at which resync should start. If no resync is needed, this will be a very large number (or `none` since 2.6.30-rc1). At array creation it will default to 0, though starting the array as `clean` will set it much larger.

> ：重新同步应该开始的点。如果不需要重新同步，这将是一个非常大的数字（或者自2.6.30-rc1以来为“none”）。在创建数组时，它将默认为0，尽管以“clean”启动数组会使其更大。

 new_dev


 :   This file can be written but not read. The value written should be a block device number as major:minor. e.g. 8:0 This will cause that device to be attached to the array, if it is available. It will then appear at md/dev-XXX (depending on the name of the device) and further configuration is then possible.

> ：此文件可以写入，但不能读取。写入的值应该是块设备编号，如major:minor。例如8:0这将导致该设备连接到阵列（如果可用的话）。然后它将显示在md/dev XXX（取决于设备的名称），然后可以进行进一步的配置。

 safe_mode_delay


 :   When an md array has seen no write requests for a certain period of time, it will be marked as `clean`. When another write request arrives, the array is marked as `dirty` before the write commences. This is known as `safe_mode`. The `certain period` is controlled by this file which stores the period as a number of seconds. The default is 200msec (0.200). Writing a value of 0 disables safemode.

> ：当md数组在一段时间内没有看到写入请求时，它将被标记为“clean”。当另一个写入请求到达时，在写入开始之前，数组被标记为“脏”。这被称为“安全模式”。“特定周期”由该文件控制，该文件将周期存储为秒数。默认值为200毫秒（0.200）。写入值0将禁用安全模式。

 array_state


 :   This file contains a single word which describes the current state of the array. In many cases, the state can be set by writing the word for the desired state, however some states cannot be explicitly set, and some transitions are not allowed.

> ：此文件包含一个单词，用于描述阵列的当前状态。在许多情况下，可以通过为所需状态写入单词来设置状态，但是有些状态不能显式设置，并且不允许某些转换。

     Select/poll works on this file. All changes except between Active_idle and active (which can be frequent and are not very interesting) are notified. active-\>active_idle is reported if the metadata is externally managed.

     clear

     :   No devices, no size, no level

         Writing is equivalent to STOP_ARRAY ioctl

     inactive

     :   May have some settings, but array is not active all IO results in error

         When written, doesn\'t tear down array, but just stops it

     suspended (not supported yet)

     :   All IO requests will block. The array can be reconfigured.

         Writing this, if accepted, will block until array is quiescent

     readonly

     :   no resync can happen. no superblocks get written.

         Write requests fail

     read-auto

     :   like readonly, but behaves like `clean` on a write request.

     clean

     :   no pending writes, but otherwise active.

         When written to inactive array, starts without resync

         If a write request arrives then if metadata is known, mark `dirty` and switch to `active`. if not known, block and switch to write-pending

         If written to an active array that has pending writes, then fails.

     active

     :   fully active: IO and resync can be happening. When written to inactive array, starts with resync

     write-pending

     :   clean, but writes are blocked waiting for `active` to be written.

     active-idle

     :   like active, but no writes have been seen for a while (safe_mode_delay).

 bitmap/location


 :   This indicates where the write-intent bitmap for the array is stored.

> ：这指示阵列的写意图位图的存储位置。

     It can be one of `none`, `file` or `[+-]N`. `file` may later be extended to `file:/file/name` `[+-]N` means that many sectors from the start of the metadata.

     This is replicated on all devices. For arrays with externally managed metadata, the offset is from the beginning of the device.

 bitmap/chunksize


 :   The size, in bytes, of the chunk which will be represented by a single bit. For RAID456, it is a portion of an individual device. For RAID10, it is a portion of the array. For RAID1, it is both (they come to the same thing).

> ：将由单个位表示的块的大小（以字节为单位）。对于RAID456，它是单个设备的一部分。对于RAID10，它是阵列的一部分。对于RAID1来说，两者都是（它们有相同的意义）。

 bitmap/time_base


 :   The time, in seconds, between looking for bits in the bitmap to be cleared. In the current implementation, a bit will be cleared between 2 and 3 times `time_base` after all the covered blocks are known to be in-sync.

> ：在位图中查找要清除的位之间的时间（以秒为单位）。在当前的实施方式中，在已知所有覆盖的块是同步的之后，将在2到3倍“time_base”之间清除一个比特。

 bitmap/backlog


 :   When write-mostly devices are active in a RAID1, write requests to those devices proceed in the background - the filesystem (or other user of the device) does not have to wait for them. `backlog` sets a limit on the number of concurrent background writes. If there are more than this, new writes will by synchronous.

> ：当RAID1中的大多数写入设备处于活动状态时，对这些设备的写入请求会在后台进行——文件系统（或设备的其他用户）不必等待它们`backlog对并发后台写入的数量设置了限制。如果超过此数量，则新写入将是同步的。

 bitmap/metadata

 :   This can be either `internal` or `external`.

     `internal`

     :   is the default and means the metadata for the bitmap is stored in the first 256 bytes of the allocated space and is managed by the md module.

     `external`

     :   means that bitmap metadata is managed externally to the kernel (i.e. by some userspace program)

 bitmap/can_clear


 :   This is either `true` or `false`. If `true`, then bits in the bitmap will be cleared when the corresponding blocks are thought to be in-sync. If `false`, bits will never be cleared. This is automatically set to `false` if a write happens on a degraded array, or if the array becomes degraded during a write. When metadata is managed externally, it should be set to true once the array becomes non-degraded, and this fact has been recorded in the metadata.

> ：这要么是“真”，要么是“假”。如果为“true”，则当相应的块被认为是同步的时，位图中的位将被清除。如果为“false”，则位将永远不会被清除。如果写入发生在降级的阵列上，或者阵列在写入过程中降级，则会自动设置为“false”。当从外部管理元数据时，一旦阵列未降级，并且此事实已记录在元数据中，则应将其设置为true。

 consistency_policy


 :   This indicates how the array maintains consistency in case of unexpected shutdown. It can be:

> ：这指示阵列在意外关闭的情况下如何保持一致性。它可以是：

     none

     :   Array has no redundancy information, e.g. raid0, linear.

     resync

     :   Full resync is performed and all redundancy is regenerated when the array is started after unclean shutdown.

     bitmap

     :   Resync assisted by a write-intent bitmap.

     journal

     :   For raid4/5/6, journal device is used to log transactions and replay after unclean shutdown.

     ppl

     :   For raid5 only, Partial Parity Log is used to close the write hole and eliminate resync.

     The accepted values when writing to this file are `ppl` and `resync`, used to enable and disable PPL.

 uuid


 :   This indicates the UUID of the array in the following format: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx

> ：这表示阵列的UUID，格式如下：xxxxxxxx-xxxx-xxxx.xxxx-xxxxxxxxxxxxx


As component devices are added to an md array, they appear in the `md` directory as new directories named:

> 当组件设备被添加到md数组时，它们在“md”目录中显示为名为的新目录：

    dev-XXX


where `XXX` is a name that the kernel knows for the device, e.g. hdb1. Each directory contains:

> 其中“XXX”是内核知道的设备的名称，例如hdb1。每个目录包含：

 block

 :   a symlink to the block device in /sys/block, e.g.:

         /sys/block/md0/md/dev-hdb1/block -> ../../../../block/hdb/hdb1

 super


 :   A file containing an image of the superblock read from, or written to, that device.

> ：包含从该设备读取或写入的超级块的映像的文件。


 state A file recording the current state of the device in the array which can be a comma separated list of:

> state记录阵列中设备当前状态的文件，可以是逗号分隔的列表，包括：

 faulty


 :   device has been kicked from active use due to a detected fault, or it has unacknowledged bad blocks

> ：由于检测到故障，或者设备有未确认的坏块，设备已从活动使用中退出

 in_sync

 :   device is a fully in-sync member of the array

 writemostly


 :   device will only be subject to read requests if there are no other options.

> ：只有在没有其他选项的情况下，设备才会收到读取请求。

     This applies only to raid1 arrays.

 blocked


 :   device has failed, and the failure hasn\'t been acknowledged yet by the metadata handler.

> ：设备出现故障，元数据处理程序尚未确认故障。

     Writes that would write to this device if it were not faulty are blocked.

 spare

 :   device is working, but not a full member.

     This includes spares that are in the process of being recovered to

 write_error

 :   device has ever seen a write error.

 want_replacement


 :   device is (mostly) working but probably should be replaced, either due to errors or due to user request.

> ：设备（大部分）正在工作，但可能由于错误或用户请求而应该更换。

 replacement


 :   device is a replacement for another active device with same raid_disk.

> ：device替换具有相同raid_disk的另一个活动设备。

 This list may grow in future.

 This can be written to.

 Writing `faulty` simulates a failure on the device.

 Writing `remove` removes the device from the array.

 Writing `writemostly` sets the writemostly flag.

 Writing `-writemostly` clears the writemostly flag.

 Writing `blocked` sets the `blocked` flag.


 Writing `-blocked` clears the `blocked` flags and allows writes to complete and possibly simulates an error.

> 写入“-blocked”将清除“blocked”标志，并允许写入完成，并可能模拟错误。

 Writing `in_sync` sets the in_sync flag.

 Writing `write_error` sets writeerrorseen flag.

 Writing `-write_error` clears writeerrorseen flag.


 Writing `want_replacement` is allowed at any time except to a replacement device or a spare. It sets the flag.

> 允许在任何时候写入“want_replacement”，但替换设备或备用设备除外。它树立了旗帜。

 Writing `-want_replacement` is allowed at any time. It clears the flag.


 Writing `replacement` or `-replacement` is only allowed before starting the array. It sets or clears the flag.

> 只允许在启动数组之前写入“replacement”或“-replacement”。它设置或清除标志。


 This file responds to select/poll. Any change to `faulty` or `blocked` causes an event.

> 此文件响应选择/轮询。对“故障”或“阻塞”的任何更改都会导致事件。


 errors An approximate count of read errors that have been detected on this device but have not caused the device to be evicted from the array (either because they were corrected or because they happened while the array was read-only). When using version-1 metadata, this value persists across restarts of the array.

> errors在该设备上检测到但未导致设备从阵列中移出的读取错误的近似计数（可能是因为这些错误已更正，也可能是因为它们发生在阵列为只读时）。当使用版本1元数据时，此值会在阵列重新启动时持续存在。


 This value can be written while assembling an array thus providing an ongoing count for arrays with metadata managed by userspace.

> 该值可以在组装数组时写入，从而为具有由用户空间管理的元数据的数组提供持续计数。

 slot

 :   This gives the role that the device has in the array. It will

 either be `none` if the device is not active in the array

 :   (i.e. is a spare or has failed) or an integer less than the


 `raid_disks` number for the array indicating which position it currently fills. This can only be set while assembling an array. A device for which this is set is assumed to be working.

> `raid_disks’数组的编号，指示当前填充的位置。这只能在组装数组时设置。设置此项的设备被认为正在工作。

 offset


 :   This gives the location in the device (in sectors from the start) where data from the array will be stored. Any part of the device before this offset is not touched, unless it is used for storing metadata (Formats 1.1 and 1.2).

> ：这给出了设备中存储阵列数据的位置（从开始的扇区）。除非用于存储元数据（格式1.1和1.2），否则不会触摸此偏移之前设备的任何部分。

 size


 :   The amount of the device, after the offset, that can be used for storage of data. This will normally be the same as the

> ：偏移后可用于存储数据的设备数量。这通常与

 component_size. This can be written while assembling an


 :   array. If a value less than the current component_size is written, it will be rejected.

> 大堆如果写入的值小于当前component_size，则会被拒绝。

 recovery_start

 :   When the device is not `in_sync`, this records the number of


 sectors from the start of the device which are known to be correct. This is normally zero, but during a recovery operation it will steadily increase, and if the recovery is interrupted, restoring this value can cause recovery to avoid repeating the earlier blocks. With v1.x metadata, this value is saved and restored automatically.

> 从设备开始的已知正确的扇区。这通常为零，但在恢复操作期间，它会稳步增加，如果恢复中断，恢复此值可以导致恢复，以避免重复以前的块。使用v1.x元数据时，会自动保存和恢复此值。


 This can be set whenever the device is not an active member of the array, either before the array is activated, or before the `slot` is set.

> 无论何时设备不是阵列的活动成员，无论是在阵列被激活之前，还是在“槽”被设置之前，都可以设置这一点。


 Setting this to `none` is equivalent to setting `in_sync`. Setting to any other value also clears the `in_sync` flag.

> 将其设置为“none”相当于设置“in_sync”。设置为任何其他值也会清除“in_sync”标志。


 bad_blocks This gives the list of all known bad blocks in the form of start address and length (in sectors respectively). If output is too big to fit in a page, it will be truncated. Writing `sector length` to this file adds new acknowledged (i.e. recorded to disk safely) bad blocks.

> bad_blocks以起始地址和长度（分别以扇区为单位）的形式给出所有已知坏块的列表。如果输出太大，无法放入页面，则会被截断。将“扇区长度”写入该文件会添加新的已确认（即安全记录到磁盘）坏块。


 unacknowledged_bad_blocks This gives the list of known-but-not-yet-saved-to-disk bad blocks in the same form of `bad_blocks`. If output is too big to fit in a page, it will be truncated. Writing to this file adds bad blocks without acknowledging them. This is largely for testing.

> unknowled_bad_blocks这以“bad_blocks”的相同形式给出了已知但尚未保存到磁盘的坏块的列表。如果输出太大，无法放入页面，则会被截断。写入此文件会在不确认的情况下添加坏块。这主要是为了测试。

 ppl_sector, ppl_size


 :   Location and size (in sectors) of the space used for Partial Parity Log on this device.

> ：此设备上用于部分奇偶校验日志的空间的位置和大小（以扇区为单位）。


An active md device will also contain an entry for each active device in the array. These are named:

> 活动md设备还将包含阵列中每个活动设备的条目。它们被命名为：

    rdNN


where `NN` is the position in the array, starting from 0. So for a 3 drive array there will be rd0, rd1, rd2. These are symbolic links to the appropriate `dev-XXX` entry. Thus, for example:

> 其中“NN”是数组中的位置，从0开始。因此，对于3驱动器阵列，将有rd0、rd1、rd2。这些是指向相应“dev XXX”条目的符号链接。因此，例如：

    cat /sys/block/md*/md/rd*/state

will show `in_sync` on every line.


Active md devices for levels that support data redundancy (1,4,5,6,10) also have

> 支持数据冗余级别（1,4,5,6.10）的活动md设备也具有

 sync_action


 :   a text file that can be used to monitor and control the rebuild process. It contains one word which can be one of:

> ：可用于监视和控制重建过程的文本文件。它包含一个单词，可以是以下单词之一：

     resync
    
     :   
    
         redundancy is being recalculated after unclean
    
         :   shutdown or creation
    
     recover
    
     :   a hot spare is being built to replace a failed/missing device
    
     idle
    
     :   nothing is happening
    
     check
    
     :   
    
         A full check of redundancy was requested and is
    
         :   happening. This reads all blocks and checks them. A repair may also happen for some raid levels.
    
     repair
    
     :   A full check and repair is happening. This is similar to `resync`, but was requested by the user, and the write-intent bitmap is NOT used to optimise the process.
    
     This file is writable, and each of the strings that could be read are meaningful for writing.

     `idle` will stop an active resync/recovery etc. There is no guarantee that another resync/recovery may not be automatically started again, though some event will be needed to trigger this.

     `resync` or `recovery` can be used to restart the

     :   corresponding operation if it was stopped with `idle`.

     `check` and `repair` will start the appropriate process providing the current state is `idle`.

     This file responds to select/poll. Any important change in the value triggers a poll event. Sometimes the value will briefly be `recover` if a recovery seems to be needed, but cannot be achieved. In that case, the transition to `recover` isn\'t notified, but the transition away is.

 degraded


 :   This contains a count of the number of devices by which the arrays is degraded. So an optimal array will show `0`. A single failed/missing drive will show `1`, etc.

> ：这包含阵列降级的设备数量的计数。因此，最佳数组将显示“0”。单个故障/丢失的驱动器将显示“1”等。

     This file responds to select/poll, any increase or decrease in the count of missing devices will trigger an event.

 mismatch_count


 :   When performing `check` and `repair`, and possibly when performing `resync`, md will count the number of errors that are found. The count in `mismatch_cnt` is the number of sectors that were re-written, or (for `check`) would have been re-written. As most raid levels work in units of pages rather than sectors, this may be larger than the number of actual errors by a factor of the number of sectors in a page.

> ：执行“check”和“repair”时，以及可能执行“resync”时，md将计算发现的错误数。“mismatch_cnt”中的计数是重新写入的扇区数，或者（对于“check”）将被重新写入。由于大多数raid级别以页面而非扇区为单位工作，因此这可能比实际错误数大一倍于页面中扇区数。

 bitmap_set_bits


 :   If the array has a write-intent bitmap, then writing to this attribute can set bits in the bitmap, indicating that a resync would need to check the corresponding blocks. Either individual numbers or start-end pairs can be written. Multiple numbers can be separated by a space.

> ：如果数组具有写意图位图，则写入此属性可以设置位图中的位，表示重新同步需要检查相应的块。可以写入单个数字或开始-结束对。多个数字可以用空格分隔。

     Note that the numbers are `bit` numbers, not `block` numbers. They should be scaled by the bitmap_chunksize.

 sync_speed_min, sync_speed_max


 :   This are similar to `/proc/sys/dev/raid/speed_limit_{min,max}` however they only apply to the particular array.

> ：这类似于“/proc/sys/dev/raid/speed_limit_｛min，max｝”，但它们只适用于特定的阵列。

     If no value has been written to these, or if the word `system` is written, then the system-wide value is used. If a value, in kibibytes-per-second is written, then it is used.

     When the files are read, they show the currently active value followed by `(local)` or `(system)` depending on whether it is a locally set or system-wide value.

 sync_completed


 :   This shows the number of sectors that have been completed of whatever the current sync_action is, followed by the number of sectors in total that could need to be processed. The two numbers are separated by a `/` thus effectively showing one value, a fraction of the process that is complete.

> ：这显示了无论当前同步操作是什么，都已完成的扇区数，然后是可能需要处理的扇区总数。这两个数字用“/”分隔，因此有效地显示了一个值，即完成过程的一小部分。

     A `select` on this attribute will return when resync completes, when it reaches the current sync_max (below) and possibly at other times.

 sync_speed


 :   This shows the current actual speed, in K/sec, of the current sync_action. It is averaged over the last 30 seconds.

> ：这显示当前同步动作的当前实际速度，单位为K/秒。它是最后30秒的平均值。

 suspend_lo, suspend_hi


 :   The two values, given as numbers of sectors, indicate a range within the array where IO will be blocked. This is currently only supported for raid4/5/6.

> ：这两个值以扇区数表示，表示阵列中IO将被阻止的范围。这目前只支持raid4/5/6。

 sync_min, sync_max


 :   The two values, given as numbers of sectors, indicate a range within the array where `check`/`repair` will operate. Must be a multiple of chunk_size. When it reaches `sync_max` it will pause, rather than complete. You can use `select` or `poll` on `sync_completed` to wait for that number to reach sync_max. Then you can either increase `sync_max`, or can write `idle` to `sync_action`.

> ：这两个值以扇区数表示，表示数组中“check”/“repair”将运行的范围。必须是chunk_size的倍数。当它达到“sync_max”时，它将暂停，而不是完成。您可以在“sync_completed”上使用“select”或“poll”来等待该数字达到sync_max。然后，您可以增加“sync_max”，也可以将“idle”写入“sync_action”。

     The value of `max` for `sync_max` effectively disables the limit. When a resync is active, the value can only ever be increased, never decreased. The value of `0` is the minimum for `sync_min`.


Each active md device may also have attributes specific to the personality module that manages it. These are specific to the implementation of the module and could change substantially if the implementation changes.

> 每个活动的md设备也可能具有特定于管理它的个性模块的属性。这些属性特定于模块的实现，如果实现发生变化，这些属性可能会发生重大变化。

These currently include:

 stripe_cache_size (currently raid5 only)


 :   number of entries in the stripe cache. This is writable, but there are upper and lower limits (32768, 17). Default is 256.

> ：条带缓存中的条目数。这是可写的，但有上限和下限（32768，17）。默认值为256。

 strip_cache_active (currently raid5 only)

 :   number of active entries in the stripe cache

 preread_bypass_threshold (currently raid5 only)


 :   number of times a stripe requiring preread will be bypassed by a stripe that does not require preread. For fairness defaults to 1. Setting this to 0 disables bypass accounting and requires preread stripes to wait until all full-width stripe-writes are complete. Valid values are 0 to stripe_cache_size.

> ：需要预读取的条带将被不需要预读的条带绕过的次数。为了公平起见，默认为1。将其设置为0将禁用旁路记帐，并要求预读取条带等待，直到所有全宽条带写入完成。有效值为0到stripe_cache_size。

 journal_mode (currently raid5 only)


 :   The cache mode for raid5. raid5 could include an extra disk for caching. The mode can be \"write-throuth\" and \"write-back\". The default is \"write-through\".

> ：raid5的缓存模式。raid5可能包括一个用于缓存的额外磁盘。模式可以是“通过写入”和“回写”。默认值为“write-through”。

 ppl_write_hint

 :   NVMe stream ID to be set for each PPL write request.
