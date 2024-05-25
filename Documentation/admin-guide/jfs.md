---
tip: translate by baidu@2024-01-30 21:35:01
---
---
title: IBM\'s Journaled File System (JFS) for Linux
---

JFS Homepage: <http://jfs.sourceforge.net/>

The following mount options are supported:

(\*) == default

iocharset=name


:   Character set to use for converting from Unicode to ASCII. The default is to do no conversion. Use iocharset=utf8 for UTF-8 translations. This requires CONFIG_NLS_UTF8 to be set in the kernel .config file. iocharset=none specifies the default behavior explicitly.

> ：用于从Unicode转换为ASCII的字符集。默认情况是不进行转换。使用iocharset=utf8进行UTF-8翻译。这需要在kernel.CONFIG文件中设置CONFIG_NLS_UTF8。iocharset=none显式指定默认行为。

resize=value


:   Resize the volume to \<value\> blocks. JFS only supports growing a volume, not shrinking it. This option is only valid during a remount, when the volume is mounted read-write. The resize keyword with no value will grow the volume to the full size of the partition.

> ：将音量调整为\<value\>块。JFS只支持增长卷，而不支持收缩卷。此选项仅在以读写方式装载卷的重新装载过程中有效。不带值的resize关键字将使卷增长到分区的完全大小。

nointegrity


:   Do not write to the journal. The primary use of this option is to allow for higher performance when restoring a volume from backup media. The integrity of the volume is not guaranteed if the system abnormally abends.

> ：不要给日记本写信。此选项的主要用途是在从备份介质恢复卷时提供更高的性能。如果系统异常异常终止，则无法保证卷的完整性。

integrity(\*)


:   Commit metadata changes to the journal. Use this option to remount a volume where the nointegrity option was previously specified in order to restore normal behavior.

> ：将元数据更改提交到日志。使用此选项可以重新装载以前指定了nointegrity选项的卷，以便恢复正常行为。

errors=continue

:   Keep going on a filesystem error.

errors=remount-ro(\*)

:   Remount the filesystem read-only on an error.

errors=panic

:   Panic and halt the machine if an error occurs.

uid=value

:   Override on-disk uid with specified value

gid=value

:   Override on-disk gid with specified value

umask=value


:   Override on-disk umask with specified octal value. For directories, the execute bit will be set if the corresponding read bit is set.

> ：使用指定的八进制值重写磁盘上的umask。对于目录，如果设置了相应的读取位，则会设置执行位。

discard=minlen, discard/nodiscard(\*)


:   This enables/disables the use of discard/TRIM commands. The discard/TRIM commands are sent to the underlying block device when blocks are freed. This is useful for SSD devices and sparse/thinly-provisioned LUNs. The FITRIM ioctl command is also available together with the nodiscard option. The value of minlen specifies the minimum blockcount, when a TRIM command to the block device is considered useful. When no value is given to the discard option, it defaults to 64 blocks, which means 256KiB in JFS. The minlen value of discard overrides the minlen value given on an FITRIM ioctl().

> ：这将启用/禁用丢弃/TRIM命令的使用。当块被释放时，丢弃/TRIM命令被发送到底层块设备。这对于SSD设备和稀疏/精简配置的LUN非常有用。FITRIM ioctl命令也可与nodiscard选项一起使用。当对块设备的TRIM命令被认为有用时，minlen的值指定最小块计数。当没有为丢弃选项提供值时，它默认为64个块，这意味着JFS中的256KiB。discard的minlen值将覆盖在FITRIM ioctl（）上给定的最小len值。


The JFS mailing list can be subscribed to by using the link labeled \"Mail list Subscribe\" at our web page <http://jfs.sourceforge.net/>

> JFS邮件列表可以使用我们网页上标记为“邮件列表订阅”的链接进行订阅<http://jfs.sourceforge.net/>
