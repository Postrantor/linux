---
tip: translate by baidu@2024-01-30 21:29:20
title: A block layer cache (bcache)
---

Say you\'ve got a big slow raid 6, and an ssd or three. Wouldn\'t it be nice if you could use them as cache\... Hence bcache.

> 假设你有一个大的慢速突袭 6，和一个或三个 ssd。如果你能把它们用作缓存，那不是很好吗。。。因此 bcache。

The bcache wiki can be found at:

: <https://bcache.evilpiepirate.org>

This is the git repository of bcache-tools:

: <https://git.kernel.org/pub/scm/linux/kernel/git/colyli/bcache-tools.git/>

The latest bcache kernel code can be found from mainline Linux kernel:

: <https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/>

It\'s designed around the performance characteristics of SSDs - it only allocates in erase block sized buckets, and it uses a hybrid btree/log to track cached extents (which can be anywhere from a single sector to the bucket size). It\'s designed to avoid random writes at all costs; it fills up an erase block sequentially, then issues a discard before reusing it.

> 它是围绕固态硬盘的性能特征设计的——它只在擦除块大小的存储桶中进行分配，并使用混合 btree/log 来跟踪缓存的扩展数据块（可以是从单个扇区到存储桶大小的任何位置）。它旨在不惜一切代价避免随机写入；它按顺序填充一个擦除块，然后在重用它之前发出丢弃。

Both writethrough and writeback caching are supported. Writeback defaults to off, but can be switched on and off arbitrarily at runtime. Bcache goes to great lengths to protect your data - it reliably handles unclean shutdown. (It doesn\'t even have a notion of a clean shutdown; bcache simply doesn\'t return writes as completed until they\'re on stable storage).

> 同时支持写通和写回缓存。写回默认为关闭，但可以在运行时任意打开和关闭。Bcache 竭尽全力保护您的数据——它可靠地处理不干净的关闭。（它甚至没有干净关闭的概念；bcache 只是在写入到稳定存储上之前不会将其返回为已完成写入）。

Writeback caching can use most of the cache for buffering writes - writing dirty data to the backing device is always done sequentially, scanning from the start to the end of the index.

> 写回缓存可以使用大部分缓存来缓冲写操作——将脏数据写入备份设备总是按顺序进行，从索引的开始到结束进行扫描。

Since random IO is what SSDs excel at, there generally won\'t be much benefit to caching large sequential IO. Bcache detects sequential IO and skips it; it also keeps a rolling average of the IO sizes per task, and as long as the average is above the cutoff it will skip all IO from that task - instead of caching the first 512k after every seek. Backups and large file copies should thus entirely bypass the cache.

> 由于 SSD 擅长随机 IO，因此缓存大型顺序 IO 通常不会有太大好处。Bcache 检测到顺序 IO 并跳过它；它还保持每个任务的 IO 大小的滚动平均值，只要平均值高于截止值，它就会跳过该任务的所有 IO，而不是在每次查找后缓存第一个 512k。因此，备份和大文件副本应该完全绕过缓存。

In the event of a data IO error on the flash it will try to recover by reading from disk or invalidating cache entries. For unrecoverable errors (meta data or dirty data), caching is automatically disabled; if dirty data was present in the cache it first disables writeback caching and waits for all dirty data to be flushed.

> 如果闪存上出现数据 IO 错误，它将尝试通过从磁盘读取或使缓存项无效来进行恢复。对于不可恢复的错误（元数据或脏数据），将自动禁用缓存；如果缓存中存在脏数据，它首先会禁用写回缓存，并等待所有脏数据被刷新。

Getting started: You\'ll need bcache util from the bcache-tools repository. Both the cache device and backing device must be formatted before use:

> 入门：您需要 bcache 工具存储库中的 bcache-util。在使用之前，必须对缓存设备和备份设备进行格式化：

```
bcache make -B /dev/sdb
bcache make -C /dev/sdc
```

[bcache make]{.title-ref} has the ability to format multiple devices at the same time - if you format your backing devices and cache device at the same time, you won\'t have to manually attach:

> [bcache-make]{.title-ref}能够同时格式化多个设备-如果同时格式化备份设备和缓存设备，则不必手动附加：

```
bcache make -B /dev/sda /dev/sdb -C /dev/sdc
```

If your bcache-tools is not updated to latest version and does not have the unified [bcache]{.title-ref} utility, you may use the legacy [make-bcache]{.title-ref} utility to format bcache device with same -B and -C parameters.

> 如果您的 bcache 工具未更新到最新版本，并且没有统一的[bcache]{.title-ref}实用程序，则可以使用传统的[make-bcache]{.title-ref}实用程序使用相同的-B 和-C 参数格式化 bcache 设备。

bcache-tools now ships udev rules, and bcache devices are known to the kernel immediately. Without udev, you can manually register devices like this:

> bcache 工具现在提供 udev 规则，内核可以立即知道 bcache 设备。没有 udev，您可以手动注册设备，如下所示：

```
echo /dev/sdb > /sys/fs/bcache/register
echo /dev/sdc > /sys/fs/bcache/register
```

Registering the backing device makes the bcache device show up in /dev; you can now format it and use it as normal. But the first time using a new bcache device, it\'ll be running in passthrough mode until you attach it to a cache. If you are thinking about using bcache later, it is recommended to setup all your slow devices as bcache backing devices without a cache, and you can choose to add a caching device later. See \'ATTACHING\' section below.

> 注册备份设备会使 bcache 设备显示在/dev/；您现在可以格式化它并正常使用它。但第一次使用新的 bcache 设备时，它将以直通模式运行，直到您将其连接到缓存。如果您正在考虑以后使用 bcache，建议将所有慢速设备设置为没有缓存的 bcache 备份设备，并且您可以选择以后添加缓存设备。请参阅下面的“跟踪”部分。

The devices show up as:

```
/dev/bcache<N>
```

As well as (with udev):

```
/dev/bcache/by-uuid/<uuid>
/dev/bcache/by-label/<label>
```

To get started:

```
mkfs.ext4 /dev/bcache0
mount /dev/bcache0 /mnt
```

You can control bcache devices through sysfs at /sys/block/bcache\<N\>/bcache . You can also control them through /sys/fs//bcache/\<cset-uuid\>/ .

> 您可以通过位于/sys/block/bcache\<N\>/bcache 的 sysfs 控制 bcache 设备。您也可以通过/sys/fs//bcache/\<cset uuid\>/来控制它们。

Cache devices are managed as sets; multiple caches per set isn\'t supported yet but will allow for mirroring of metadata and dirty data in the future. Your new cache set shows up as /sys/fs/bcache/\<UUID\>

> 缓存设备按集进行管理；目前还不支持每组多个缓存，但将来可以镜像元数据和脏数据。您的新缓存集显示为/sys/fs/bcache/\<UUID\>

# Attaching

After your cache device and backing device are registered, the backing device must be attached to your cache set to enable caching. Attaching a backing device to a cache set is done thusly, with the UUID of the cache set in /sys/fs/bcache:

> 注册缓存设备和备份设备后，备份设备必须连接到缓存集才能启用缓存。将备份设备连接到缓存集是这样完成的，缓存集的 UUID 位于/sys/fs/bcache 中：

```
echo <CSET-UUID> > /sys/block/bcache0/bcache/attach
```

This only has to be done once. The next time you reboot, just reregister all your bcache devices. If a backing device has data in a cache somewhere, the /dev/bcache\<N\> device won\'t be created until the cache shows up - particularly important if you have writeback caching turned on.

> 这只需要做一次。下次重新启动时，只需重新注册所有 bcache 设备即可。如果备份设备的某个缓存中有数据，则在缓存出现之前不会创建/dev/bcache\<N\>设备，这在打开写回缓存的情况下尤为重要。

If you\'re booting up and your cache device is gone and never coming back, you can force run the backing device:

> 如果你正在启动，而你的缓存设备不见了，再也回不来了，你可以强制运行备份设备：

```
echo 1 > /sys/block/sdb/bcache/running
```

(You need to use /sys/block/sdb (or whatever your backing device is called), not /sys/block/bcache0, because bcache0 doesn\'t exist yet. If you\'re using a partition, the bcache directory would be at /sys/block/sdb/sdb2/bcache)

> （您需要使用/sys/block/sdb（或您的备份设备的名称），而不是/sys/back/bcache0，因为 bcache0 还不存在。如果使用分区，则 bcache 目录将位于/sys/block/sdb/sdb2/bcache）

The backing device will still use that cache set if it shows up in the future, but all the cached data will be invalidated. If there was dirty data in the cache, don\'t expect the filesystem to be recoverable - you will have massive filesystem corruption, though ext4\'s fsck does work miracles.

> 如果该缓存集将来出现，备份设备仍将使用该缓存集，但所有缓存的数据都将无效。如果缓存中有脏数据，不要指望文件系统是可恢复的——尽管 ext4 的 fsck 确实起到了奇迹般的作用，但你会有大量的文件系统损坏。

# Error Handling

Bcache tries to transparently handle IO errors to/from the cache device without affecting normal operation; if it sees too many errors (the threshold is configurable, and defaults to 0) it shuts down the cache device and switches all the backing devices to passthrough mode.

> Bcache 试图在不影响正常操作的情况下透明地处理到缓存设备的 IO 错误；如果它看到太多错误（阈值是可配置的，默认为 0），它会关闭缓存设备，并将所有备份设备切换到直通模式。

> - For reads from the cache, if they error we just retry the read from the backing device.
> - For writethrough writes, if the write to the cache errors we just switch to invalidating the data at that lba in the cache (i.e. the same thing we do for a write that bypasses the cache)
> - For writeback writes, we currently pass that error back up to the filesystem/userspace. This could be improved - we could retry it as a write that skips the cache so we don\'t have to error the write.
> - When we detach, we first try to flush any dirty data (if we were running in writeback mode). It currently doesn\'t do anything intelligent if it fails to read some of the dirty data, though.

# Howto/cookbook

A) Starting a bcache with a missing caching device

If registering the backing device doesn\'t help, it\'s already there, you just need to force it to run without the cache:

> 如果注册备份设备没有帮助，它已经存在，您只需要强制它在没有缓存的情况下运行：

```
host:~# echo /dev/sdb1 > /sys/fs/bcache/register
[  119.844831] bcache: register_bcache() error opening /dev/sdb1: device already registered
```

Next, you try to register your caching device if it\'s present. However if it\'s absent, or registration fails for some reason, you can still start your bcache without its cache, like so:

```
host:/sys/block/sdb/sdb1/bcache# echo 1 > running
```

Note that this may cause data loss if you were running in writeback mode.

B) Bcache does not find its cache:

```
host:/sys/block/md5/bcache# echo 0226553a-37cf-41d5-b3ce-8b1e944543a8 > attach
[ 1933.455082] bcache: bch_cached_dev_attach() Couldn't find uuid for md5 in set
[ 1933.478179] bcache: __cached_dev_store() Can't attach 0226553a-37cf-41d5-b3ce-8b1e944543a8
[ 1933.478179] : cache set not found
```

In this case, the caching device was simply not registered at boot or disappeared and came back, and needs to be (re-)registered:

> 在这种情况下，缓存设备只是在启动时没有注册，或者消失后又回来了，需要（重新）注册：

```
host:/sys/block/md5/bcache# echo /dev/sdh2 > /sys/fs/bcache/register
```

C) Corrupt bcache crashes the kernel at device registration time:

This should never happen. If it does happen, then you have found a bug! Please report it to the bcache development list: <linux-bcache@vger.kernel.org>

> 这不应该发生。如果真的发生了，那么你已经发现了一个 bug！请向 bcache 开发列表报告：<linux-bcache@vger.kernel.org>

Be sure to provide as much information that you can including kernel dmesg output if available so that we may assist.

> 请确保提供尽可能多的信息，包括内核 dmesg 输出（如果可用），以便我们提供帮助。

D) Recovering data without bcache:

If bcache is not available in the kernel, a filesystem on the backing device is still available at an 8KiB offset. So either via a loopdev of the backing device created with \--offset 8K, or any value defined by \--data-offset when you originally formatted bcache with [bcache make]{.title-ref}.

> 如果 bcache 在内核中不可用，则备份设备上的文件系统仍然可用，偏移量为 8KiB。因此，通过使用\-offset 8K 创建的备份设备的 loopdev，或者当您最初使用[bcache-make]{.title-ref}格式化 bcache 时，通过\-data offset 定义的任何值。

For example:

```
losetup -o 8192 /dev/loop0 /dev/your_bcache_backing_dev
```

This should present your unmodified backing device data in /dev/loop0

If your cache is in writethrough mode, then you can safely discard the cache device without losing data.

> 如果缓存处于写通模式，则可以安全地丢弃缓存设备而不会丢失数据。

E) Wiping a cache device

```
host:~# wipefs -a /dev/sdh2
16 bytes were erased at offset 0x1018 (bcache)
they were: c6 85 73 f6 4e 1a 45 ca 82 65 f5 7f 48 ba 6d 81
```

After you boot back with bcache enabled, you recreate the cache and attach it:

> 在启用 bcache 的情况下重新启动后，重新创建缓存并附加它：

```
host:~# bcache make -C /dev/sdh2
UUID:                   7be7e175-8f4c-4f99-94b2-9c904d227045
Set UUID:               5bc072a8-ab17-446d-9744-e247949913c1
version:                0
nbuckets:               106874
block_size:             1
bucket_size:            1024
nr_in_set:              1
nr_this_dev:            0
first_bucket:           1
[  650.511912] bcache: run_cache_set() invalidating existing data
[  650.549228] bcache: register_cache() registered cache device sdh2
```

start backing device with missing cache:

```
host:/sys/block/md5/bcache# echo 1 > running
```

attach new cache:

```
host:/sys/block/md5/bcache# echo 5bc072a8-ab17-446d-9744-e247949913c1 > attach
[  865.276616] bcache: bch_cached_dev_attach() Caching md5 as bcache0 on set 5bc072a8-ab17-446d-9744-e247949913c1
```

F) Remove or replace a caching device:

```
host:/sys/block/sda/sda7/bcache# echo 1 > detach
[  695.872542] bcache: cached_dev_detach_finish() Caching disabled for sda7

host:~# wipefs -a /dev/nvme0n1p4
wipefs: error: /dev/nvme0n1p4: probing initialization failed: Device or resource busy
Ooops, it's disabled, but not unregistered, so it's still protected
```

We need to go and unregister it:

```
host:/sys/fs/bcache/b7ba27a1-2398-4649-8ae3-0959f57ba128# ls -l cache0
lrwxrwxrwx 1 root root 0 Feb 25 18:33 cache0 -> ../../../devices/pci0000:00/0000:00:1d.0/0000:70:00.0/nvme/nvme0/nvme0n1/nvme0n1p4/bcache/
host:/sys/fs/bcache/b7ba27a1-2398-4649-8ae3-0959f57ba128# echo 1 > stop
kernel: [  917.041908] bcache: cache_set_free() Cache set b7ba27a1-2398-4649-8ae3-0959f57ba128 unregistered
```

Now we can wipe it:

```
host:~# wipefs -a /dev/nvme0n1p4
/dev/nvme0n1p4: 16 bytes were erased at offset 0x00001018 (bcache): c6 85 73 f6 4e 1a 45 ca 82 65 f5 7f 48 ba 6d 81
```

G) dm-crypt and bcache

First setup bcache unencrypted and then install dmcrypt on top of /dev/bcache\<N\> This will work faster than if you dmcrypt both the backing and caching devices and then install bcache on top. \[benchmarks?\]

> 首先将 bcache 设置为未加密，然后在/dev/bcache\<N\>上安装 dmcrypt。这将比将备份和缓存设备都 dmcrypt.然后在顶部安装 bcache 更快\[基准？\]

H) Stop/free a registered bcache to wipe and/or recreate it

Suppose that you need to free up all bcache references so that you can fdisk run and re-register a changed partition table, which won\'t work if there are any active backing or caching devices left on it:

> 假设您需要释放所有 bcache 引用，以便 fdisk 运行并重新注册更改后的分区表，如果上面还有任何活动的备份或缓存设备，则该表将不起作用：

1.  Is it present in /dev/bcache\* ? (there are times where it won\'t be)

If so, it\'s easy:

```
host:/sys/block/bcache0/bcache# echo 1 > stop
```

2.  But if your backing device is gone, this won\'t work:

```
host:/sys/block/bcache0# cd bcache
bash: cd: bcache: No such file or directory
```

In this case, you may have to unregister the dmcrypt block device that references this bcache to free it up:

```
host:~# dmsetup remove oldds1
bcache: bcache_device_free() bcache0 stopped
bcache: cache_set_free() Cache set 5bc072a8-ab17-446d-9744-e247949913c1 unregistered
```

This causes the backing bcache to be removed from /sys/fs/bcache and then it can be reused. This would be true of any block device stacking where bcache is a lower device.

3.  In other cases, you can also look in /sys/fs/bcache/:

```
host:/sys/fs/bcache# ls -l */{cache?,bdev?}
lrwxrwxrwx 1 root root 0 Mar  5 09:39 0226553a-37cf-41d5-b3ce-8b1e944543a8/bdev1 -> ../../../devices/virtual/block/dm-1/bcache/
lrwxrwxrwx 1 root root 0 Mar  5 09:39 0226553a-37cf-41d5-b3ce-8b1e944543a8/cache0 -> ../../../devices/virtual/block/dm-4/bcache/
lrwxrwxrwx 1 root root 0 Mar  5 09:39 5bc072a8-ab17-446d-9744-e247949913c1/cache0 -> ../../../devices/pci0000:00/0000:00:01.0/0000:01:00.0/ata10/host9/target9:0:0/9:0:0:0/block/sdl/sdl2/bcache/
```

The device names will show which UUID is relevant, cd in that directory and stop the cache:

```
host:/sys/fs/bcache/5bc072a8-ab17-446d-9744-e247949913c1# echo 1 > stop
```

This will free up bcache references and let you reuse the partition for other purposes.

# Troubleshooting performance

Bcache has a bunch of config options and tunables. The defaults are intended to be reasonable for typical desktop and server workloads, but they\'re not what you want for getting the best possible numbers when benchmarking.

> Bcache 有一堆配置选项和可调参数。对于典型的桌面和服务器工作负载，默认值是合理的，但它们不是您在进行基准测试时获得尽可能好的数字所希望的。

> - Backing device alignment
>
>   The default metadata size in bcache is 8k. If your backing device is RAID based, then be sure to align this by a multiple of your stride width using [bcache make \--data-offset]{.title-ref}. If you intend to expand your disk array in the future, then multiply a series of primes by your raid stripe size to get the disk multiples that you would like.
>
>   For example: If you have a 64k stripe size, then the following offset would provide alignment for many common RAID5 data spindle counts:
>
>       64k * 2*2*2*3*3*5*7 bytes = 161280k
>
>   That space is wasted, but for only 157.5MB you can grow your RAID 5 volume to the following data-spindle counts without re-aligning:
>
>       3,4,5,6,7,8,9,10,12,14,15,18,20,21 ...
>
> - Bad write performance
>
>   If write performance is not what you expected, you probably wanted to be running in writeback mode, which isn\'t the default (not due to a lack of maturity, but simply because in writeback mode you\'ll lose data if something happens to your SSD):
>
>       # echo writeback > /sys/block/bcache0/bcache/cache_mode
>
> - Bad performance, or traffic not going to the SSD that you\'d expect
>
>   By default, bcache doesn\'t cache everything. It tries to skip sequential IO -because you really want to be caching the random IO, and if you copy a 10 gigabyte file you probably don\'t want that pushing 10 gigabytes of randomly accessed data out of your cache.
>
>   But if you want to benchmark reads from cache, and you start out with fio writing an 8 gigabyte test file - so you want to disable that:
>
>       # echo 0 > /sys/block/bcache0/bcache/sequential_cutoff
>
>   To set it back to the default (4 mb), do:
>
>       # echo 4M > /sys/block/bcache0/bcache/sequential_cutoff
>
> - Traffic\'s still going to the spindle/still getting cache misses
>
>   In the real world, SSDs don\'t always keep up with disks - particularly with slower SSDs, many disks being cached by one SSD, or mostly sequential IO. So you want to avoid being bottlenecked by the SSD and having it slow everything down.
>
>   To avoid that bcache tracks latency to the cache device, and gradually throttles traffic if the latency exceeds a threshold (it does this by cranking down the sequential bypass).
>
>   You can disable this if you need to by setting the thresholds to 0:
>
>       # echo 0 > /sys/fs/bcache/<cache set>/congested_read_threshold_us
>       # echo 0 > /sys/fs/bcache/<cache set>/congested_write_threshold_us
>
>   The default is 2000 us (2 milliseconds) for reads, and 20000 for writes.
>
> - Still getting cache misses, of the same data
>
>   One last issue that sometimes trips people up is actually an old bug, due to the way cache coherency is handled for cache misses. If a btree node is full, a cache miss won\'t be able to insert a key for the new data and the data won\'t be written to the cache.
>
>   In practice this isn\'t an issue because as soon as a write comes along it\'ll cause the btree node to be split, and you need almost no write traffic for this to not show up enough to be noticeable (especially since bcache\'s btree nodes are huge and index large regions of the device). But when you\'re benchmarking, if you\'re trying to warm the cache by reading a bunch of data and there\'s no other traffic - that can be a problem.
>
>   Solution: warm the cache by doing writes, or use the testing branch (there\'s a fix for the issue there).

# Sysfs - backing device

Available at /sys/block/\<bdev\>/bcache, /sys/block/bcache\*/bcache and (if attached) /sys/fs/bcache/\<cset-uuid\>/bdev\*

> 可在/sys/block/\<bdev\>/bcache、/sys/block/bcache\*/bcache 和（如果已连接）/sys/fs/bcache/\<cset uuid\>/bdev 上获得\*

attach

: Echo the UUID of a cache set to this file to enable caching.

cache_mode

: Can be one of either writethrough, writeback, writearound or none.

clear_stats

: Writing to this file resets the running total stats (not the day/hour/5 minute decaying versions).

> ：写入此文件将重置运行的总统计数据（而不是天/小时/5 分钟的衰减版本）。

detach

: Write to this file to detach from a cache set. If there is dirty data in the cache, it will be flushed first.

> ：写入此文件以从缓存集分离。如果缓存中有脏数据，将首先对其进行刷新。

dirty_data

: Amount of dirty data for this backing device in the cache. Continuously updated unlike the cache set\'s version, but may be slightly off.

> ：缓存中此备份设备的脏数据量。与缓存集的版本不同，它不断更新，但可能略有偏离。

label

: Name of underlying device.

readahead

: Size of readahead that should be performed. Defaults to 0. If set to e.g. 1M, it will round cache miss reads up to that size, but without overlapping existing cache entries.

> ：应执行的预读大小。默认值为 0。如果设置为例如 1M，它将把缓存未命中读取四舍五入到该大小，但不会重叠现有的缓存条目。

running

: 1 if bcache is running (i.e. whether the /dev/bcache device exists, whether it\'s in passthrough mode or caching).

> ：1 如果 bcache 正在运行（即/dev/bcache 设备是否存在，是否处于直通模式或缓存）。

sequential_cutoff

: A sequential IO will bypass the cache once it passes this threshold; the most recent 128 IOs are tracked so sequential IO can be detected even when it isn\'t all done at once.

> ：顺序 IO 一旦超过此阈值，就会绕过缓存；跟踪最近的 128 个 IO，因此即使不是一次完成所有 IO，也可以检测到顺序 IO。

sequential_merge

: If non zero, bcache keeps a list of the last 128 requests submitted to compare against all new requests to determine which new requests are sequential continuations of previous requests for the purpose of determining sequential cutoff. This is necessary if the sequential cutoff value is greater than the maximum acceptable sequential size for any single request.

> ：如果不是零，bcache 会保留最后提交的 128 个请求的列表，以便与所有新请求进行比较，以确定哪些新请求是以前请求的连续请求，从而确定顺序截止。如果顺序截止值大于任何单个请求的最大可接受顺序大小，则这是必要的。

state

: The backing device can be in one of four different states:

    no cache: Has never been attached to a cache set.
    clean: Part of a cache set, and there is no cached dirty data.
    dirty: Part of a cache set, and there is cached dirty data.
    inconsistent: The backing device was forcibly run by the user when there was dirty data cached but the cache set was unavailable; whatever data was on the backing device has likely been corrupted.

stop

: Write to this file to shut down the bcache device and close the backing device.

> ：写入此文件以关闭 bcache 设备并关闭备份设备。

writeback_delay

: When dirty data is written to the cache and it previously did not contain any, waits some number of seconds before initiating writeback. Defaults to 30.

> ：当脏数据被写入缓存，并且以前不包含任何脏数据时，在启动写回之前等待一些秒。默认为 30。

writeback_percent

: If nonzero, bcache tries to keep around this percentage of the cache dirty by throttling background writeback and using a PD controller to smoothly adjust the rate.

> ：如果非零，bcache 会尝试通过限制后台写回并使用 PD 控制器平滑调整速率来保持大约这个百分比的缓存脏。

writeback_rate

: Rate in sectors per second - if writeback_percent is nonzero, background writeback is throttled to this rate. Continuously adjusted by bcache but may also be set by the user.

> ：以扇区每秒为单位的速率-如果 writeback_percent 为非零，则后台写回将被限制为此速率。通过 bcache 连续调整，但也可以由用户设置。

writeback_running

: If off, writeback of dirty data will not take place at all. Dirty data will still be added to the cache until it is mostly full; only meant for benchmarking. Defaults to on.

> ：如果禁用，则根本不会进行脏数据的写回。脏数据仍将被添加到缓存中，直到它基本上已满；仅用于基准测试。默认为启用。

## Sysfs - backing device stats

There are directories with these numbers for a running total, as well as versions that decay over the past day, hour and 5 minutes; they\'re also aggregated in the cache set directory as well.

> 有一些目录包含这些数字，用于连续总数，以及在过去一天、一小时和五分钟内衰减的版本；它们也聚集在缓存集目录中。

bypassed

: Amount of IO (both reads and writes) that has bypassed the cache

cache_hits, cache_misses, cache_hit_ratio

: Hits and misses are counted per individual IO as bcache sees them; a partial hit is counted as a miss.

> ：命中和未命中按 bcache 看到的每个 IO 进行计数；部分命中算作未命中。

cache_bypass_hits, cache_bypass_misses

: Hits and misses for IO that is intended to skip the cache are still counted, but broken out here.

> ：用于跳过缓存的 IO 的命中和未命中仍在计算中，但在此处细分。

cache_miss_collisions

: Counts instances where data was going to be inserted into the cache from a cache miss, but raced with a write and data was already present (usually 0 since the synchronization for cache misses was rewritten)

> ：统计数据将从缓存未命中插入到缓存中的实例，但在写入时进行了竞争，并且数据已经存在（通常为 0，因为缓存未命中的同步已重写）

## Sysfs - cache set

Available at /sys/fs/bcache/\<cset-uuid\>

average_key_size

: Average data per key in the btree.

bdev\<0..n\>

: Symlink to each of the attached backing devices.

block_size

: Block size of the cache devices.

btree_cache_size

: Amount of memory currently used by the btree cache

bucket_size

: Size of buckets

cache\<0..n\>

: Symlink to each of the cache devices comprising this cache set.

cache_available_percent

: Percentage of cache device which doesn\'t contain dirty data, and could potentially be used for writeback. This doesn\'t mean this space isn\'t used for clean cached data; the unused statistic (in priority_stats) is typically much lower.

> ：不包含脏数据并且可能用于写回的缓存设备的百分比。这并不意味着这个空间不用于干净的缓存数据；未使用的统计数据（在 priority_stats 中）通常要低得多。

clear_stats

: Clears the statistics associated with this cache

dirty_data

: Amount of dirty data is in the cache (updated when garbage collection runs).

> ：缓存中的脏数据量（在垃圾回收运行时更新）。

flash_vol_create

: Echoing a size to this file (in human readable units, k/M/G) creates a thinly provisioned volume backed by the cache set.

> ：对此文件回显一个大小（以人类可读的单位，k/M/G）将创建一个由缓存集支持的精简配置卷。

io_error_halflife, io_error_limit

: These determines how many errors we accept before disabling the cache. Each error is decayed by the half life (in \# ios). If the decaying count reaches io_error_limit dirty data is written out and the cache is disabled.

> ：这些决定了在禁用缓存之前我们接受的错误数。每个错误都会衰减半衰期（在\#ios 中）。如果衰减计数达到 io_error_limit，则会写出脏数据并禁用缓存。

journal_delay_ms

: Journal writes will delay for up to this many milliseconds, unless a cache flush happens sooner. Defaults to 100.

> ：除非缓存刷新发生得更早，否则日志写入最多会延迟几毫秒。默认值为 100。

root_usage_percent

: Percentage of the root btree node in use. If this gets too high the node will split, increasing the tree depth.

> ：正在使用的根 btree 节点的百分比。如果这个值太高，节点将被拆分，从而增加树的深度。

stop

: Write to this file to shut down the cache set - waits until all attached backing devices have been shut down.

> ：写入此文件以关闭缓存集-等待，直到所有连接的备份设备都关闭为止。

tree_depth

: Depth of the btree (A single node btree has depth 0).

unregister

: Detaches all backing devices and closes the cache devices; if dirty data is present it will disable writeback caching and wait for it to be flushed.

> ：分离所有备份设备并关闭缓存设备；如果存在脏数据，它将禁用写回缓存并等待刷新。

## Sysfs - cache set internal

This directory also exposes timings for a number of internal operations, with separate files for average duration, average frequency, last occurrence and max duration: garbage collection, btree read, btree node sorts and btree splits.

> 该目录还公开了许多内部操作的时间安排，并为平均持续时间、平均频率、最后一次出现和最大持续时间提供了单独的文件：垃圾收集、btree 读取、btree 节点排序和 btree 拆分。

active_journal_entries

: Number of journal entries that are newer than the index.

btree_nodes

: Total nodes in the btree.

btree_used_percent

: Average fraction of btree in use.

bset_tree_stats

: Statistics about the auxiliary search trees

btree_cache_max_chain

: Longest chain in the btree node cache\'s hash table

cache_read_races

: Counts instances where while data was being read from the cache, the bucket was reused and invalidated - i.e. where the pointer was stale after the read completed. When this occurs the data is reread from the backing device.

> ：统计从缓存中读取数据时，存储桶被重用和无效的实例，即指针在读取完成后过期的实例。发生这种情况时，将从备份设备中重新读取数据。

trigger_gc

: Writing to this file forces garbage collection to run.

## Sysfs - Cache device

Available at /sys/block/\<cdev\>/bcache

block_size

: Minimum granularity of writes - should match hardware sector size.

btree_written

: Sum of all btree writes, in (kilo/mega/giga) bytes

bucket_size

: Size of buckets

cache_replacement_policy

: One of either lru, fifo or random.

discard

: Boolean; if on a discard/TRIM will be issued to each bucket before it is reused. Defaults to off, since SATA TRIM is an unqueued command (and thus slow).

> ：布尔值；如果处于报废状态，则在重新使用之前，将向每个铲斗发出 TRIM。默认为关闭，因为 SATA TRIM 是一个未激活的命令（因此速度较慢）。

freelist_percent

: Size of the freelist as a percentage of nbuckets. Can be written to to increase the number of buckets kept on the freelist, which lets you artificially reduce the size of the cache at runtime. Mostly for testing purposes (i.e. testing how different size caches affect your hit rate), but since buckets are discarded when they move on to the freelist will also make the SSD\'s garbage collection easier by effectively giving it more reserved space.

> ：免费列表的大小，以 nbuckets 的百分比表示。可以写入以增加自由列表中保留的存储桶的数量，这使您可以在运行时人为地减少缓存的大小。主要用于测试目的（即测试不同大小的缓存如何影响命中率），但由于存储桶在移动到空闲列表时会被丢弃，因此也会有效地为 SSD 提供更多的保留空间，从而使其垃圾收集更容易。

io_errors

: Number of errors that have occurred, decayed by io_error_halflife.

metadata_written

: Sum of all non data writes (btree writes and all other metadata).

nbuckets

: Total buckets in this cache

priority_stats

: Statistics about how recently data in the cache has been accessed. This can reveal your working set size. Unused is the percentage of the cache that doesn\'t contain any data. Metadata is bcache\'s metadata overhead. Average is the average priority of cache buckets. Next is a list of quantiles with the priority threshold of each.

> ：有关最近访问缓存中数据的时间的统计信息。这可以显示您的工作集大小。Unused 是不包含任何数据的缓存的百分比。元数据是 bcache 的元数据开销。Average 是缓存桶的平均优先级。接下来是一个分位数列表，每个分位数的优先级阈值都是。

written

: Sum of all data that has been written to the cache; comparison with btree_written gives the amount of write inflation in bcache.

> ：已写入缓存的所有数据的总和；与 btree_writen 的比较给出了 bcache 中的写入通货膨胀量。
