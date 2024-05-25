---
tip: translate by baidu@2024-01-30 21:34:16
---
---
author:
- Chris Wing \<<wingc@umich.edu>\>
last updated: |
  January 11, 2000
title: Notes on the change from 16-bit UIDs to 32-bit UIDs
---


-   kernel code MUST take into account \_\_kernel_uid_t and \_\_kernel_uid32_t when communicating between user and kernel space in an ioctl or data structure.

> -在ioctl或数据结构中的用户和内核空间之间进行通信时，内核代码必须考虑\_\_kernel_uid_t和\_\_kernel _uid32_t。
-   kernel code should use uid_t and gid_t in kernel-private structures and code.

What\'s left to be done for 32-bit UIDs on all Linux architectures:


-   Disk quotas have an interesting limitation that is not related to the maximum UID/GID. They are limited by the maximum file size on the underlying filesystem, because quota records are written at offsets corresponding to the UID in question. Further investigation is needed to see if the quota system can cope properly with huge UIDs. If it can deal with 64-bit file offsets on all architectures, this should not be a problem.

> -磁盘配额有一个有趣的限制，与最大UID/GID无关。它们受到底层文件系统上最大文件大小的限制，因为配额记录是以与所讨论的UID相对应的偏移量写入的。需要进一步调查，看看配额制度是否能妥善应对巨大的UID。如果它能够处理所有体系结构上的64位文件偏移，这应该不是问题。


-   Decide whether or not to keep backwards compatibility with the system accounting file, or if we should break it as the comments suggest (currently, the old 16-bit UID and GID are still written to disk, and part of the former pad space is used to store separate 32-bit UID and GID)

> -决定是否与系统记帐文件保持向后兼容性，或者我们是否应该按照注释的建议破坏它（目前，旧的16位UID和GID仍写入磁盘，以前的部分pad空间用于存储单独的32位UID和GID）


-   Need to validate that OS emulation calls the 16-bit UID compatibility syscalls, if the OS being emulated used 16-bit UIDs, or uses the 32-bit UID system calls properly otherwise.

> -如果被仿真的操作系统使用了16位UID，或者正确使用了32位UID系统调用，则需要验证操作系统仿真是否调用了16位UID兼容性系统调用。

    This affects at least:

    > -   iBCS on Intel
    > -   sparc32 emulation on sparc64 (need to support whatever new 32-bit UID system calls are added to sparc32)

-   Validate that all filesystems behave properly.

    At present, 32-bit UIDs \_[should]() work for:

    > -   ext2
    > -   ufs
    > -   isofs
    > -   nfs
    > -   coda
    > -   udf

    Ioctl() fixups have been made for:

    > -   ncpfs
    > -   smbfs

    Filesystems with simple fixups to prevent 16-bit UID wraparound:

    > -   minix
    > -   sysv
    > -   qnx4

    Other filesystems have not been checked yet.


-   The ncpfs and smpfs filesystems cannot presently use 32-bit UIDs in all ioctl()s. Some new ioctl()s have been added with 32-bit UIDs, but more are needed. (as well as new user\<-\>kernel data structures)

> -ncpfs和smpfs文件系统目前不能在所有ioctl（）中使用32位UID。一些新的ioctl（）已经添加了32位UID，但还需要更多。（以及新的用户\<-\>内核数据结构）


-   The ELF core dump format only supports 16-bit UIDs on arm, i386, m68k, sh, and sparc32. Fixing this is probably not that important, but would require adding a new ELF section.

> -ELF核心转储格式仅支持arm、i386、m68k、sh和sparc32上的16位UID。解决这个问题可能没有那么重要，但需要添加一个新的ELF部分。


-   The ioctl()s used to control the in-kernel NFS server only support 16-bit UIDs on arm, i386, m68k, sh, and sparc32.

> -用于控制内核内NFS服务器的ioctl（）仅支持arm、i386、m68k、sh和sparc32上的16位UID。


-   make sure that the UID mapping feature of AX25 networking works properly (it should be safe because it\'s always used a 32-bit integer to communicate between user and kernel)

> -确保AX25网络的UID映射功能正常工作（它应该是安全的，因为它总是使用32位整数在用户和内核之间进行通信）
