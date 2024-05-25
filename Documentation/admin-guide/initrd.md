---
tip: translate by baidu@2024-01-30 21:34:31
title: Using the initial RAM disk (initrd)
---

Written 1996,2000 by Werner Almesberger \<<werner.almesberger@epfl.ch>\> and Hans Lermen \<<lermen@fgan.de>\>

initrd provides the capability to load a RAM disk by the boot loader. This RAM disk can then be mounted as the root file system and programs can be run from it. Afterwards, a new root file system can be mounted from a different device. The previous root (from initrd) is then moved to a directory and can be subsequently unmounted.

> initrd 提供了通过引导加载程序加载 RAM 磁盘的功能。然后，这个 RAM 磁盘可以作为根文件系统安装，并可以从中运行程序。然后，可以从不同的设备安装新的根文件系统。然后将上一个根（来自 initrd）移到一个目录中，然后可以卸载。

initrd is mainly designed to allow system startup to occur in two phases, where the kernel comes up with a minimum set of compiled-in drivers, and where additional modules are loaded from initrd.

> initrd 主要设计用于允许系统启动分两个阶段进行，其中内核提供一组最小的编译驱动程序，以及从 initrd 加载其他模块。

This document gives a brief overview of the use of initrd. A more detailed discussion of the boot process can be found in[^1].

> 本文档简要概述了 initrd 的使用。有关引导过程的详细讨论，请参见[^1]。

# Operation

When using initrd, the system typically boots as follows:

> 1. the boot loader loads the kernel and the initial RAM disk
> 2. the kernel converts initrd into a \"normal\" RAM disk and frees the memory used by initrd
> 3. if the root device is not `/dev/ram0`, the old (deprecated) change_root procedure is followed. see the \"Obsolete root change mechanism\" section below.
> 4. root device is mounted. if it is `/dev/ram0`, the initrd image is then mounted as root
> 5. /sbin/init is executed (this can be any valid executable, including shell scripts; it is run with uid 0 and can do basically everything init can do).
> 6. init mounts the \"real\" root file system
> 7. init places the root file system at the root directory using the pivot_root system call
> 8. init execs the `/sbin/init` on the new root filesystem, performing the usual boot sequence
> 9. the initrd file system is removed

Note that changing the root directory does not involve unmounting it. It is therefore possible to leave processes running on initrd during that procedure. Also note that file systems mounted under initrd continue to be accessible.

> 请注意，更改根目录并不需要卸载它。因此，在该过程中，可以让进程在 initrd 上运行。还要注意，安装在 initrd 下的文件系统仍然可以访问。

# Boot command-line options

initrd adds the following new options:

    initrd=<path>    (e.g. LOADLIN)

      Loads the specified file as the initial RAM disk. When using LILO, you
      have to specify the RAM disk image file in /etc/lilo.conf, using the
      INITRD configuration variable.

    noinitrd

      initrd data is preserved but it is not converted to a RAM disk and
      the "normal" root file system is mounted. initrd data can be read
      from /dev/initrd. Note that the data in initrd can have any structure
      in this case and doesn't necessarily have to be a file system image.
      This option is used mainly for debugging.

      Note: /dev/initrd is read-only and it can only be used once. As soon
      as the last process has closed it, all data is freed and /dev/initrd
      can't be opened anymore.

    root=/dev/ram0

      initrd is mounted as root, and the normal boot procedure is followed,
      with the RAM disk mounted as root.

# Compressed cpio images

Recent kernels have support for populating a ramdisk from a compressed cpio archive. On such systems, the creation of a ramdisk image doesn\'t need to involve special block devices or loopbacks; you merely create a directory on disk with the desired initrd content, cd to that directory, and run (as an example):

> 最近的内核支持从压缩的 cpio 档案中填充 ramdisk。在这样的系统上，创建 ramdisk 映像不需要涉及特殊的块设备或环回；您只需在磁盘上创建一个包含所需 initrd 内容的目录，cd 到该目录，然后运行（例如）：

```
find . | cpio --quiet -H newc -o | gzip -9 -n > /boot/imagefile.img
```

Examining the contents of an existing image file is just as simple:

```
mkdir /tmp/imagefile
cd /tmp/imagefile
gzip -cd /boot/imagefile.img | cpio -imd --quiet
```

# Installation

First, a directory for the initrd file system has to be created on the \"normal\" root file system, e.g.:

> 首先，必须在\“normal\”根文件系统上创建 initrd 文件系统的目录，例如：

```
# mkdir /initrd
```

The name is not relevant. More details can be found on the `pivot_root(2)`{.interpreted-text role="manpage"} man page.

> 名称不相关。更多详细信息可以在`pivot_root（2）`｛.depreted text role=“manpage”｝手册页上找到。

If the root file system is created during the boot procedure (i.e. if you\'re building an install floppy), the root file system creation procedure should create the `/initrd` directory.

> 如果根文件系统是在引导过程中创建的（即，如果您正在构建安装软盘），则根文件系统创建过程应创建“/initrd”目录。

If initrd will not be mounted in some cases, its content is still accessible if the following device has been created:

> 如果在某些情况下不会装载 initrd，那么如果创建了以下设备，则仍然可以访问其内容：

```
# mknod /dev/initrd b 1 250
# chmod 400 /dev/initrd
```

Second, the kernel has to be compiled with RAM disk support and with support for the initial RAM disk enabled. Also, at least all components needed to execute programs from initrd (e.g. executable format and file system) must be compiled into the kernel.

> 其次，必须在支持 RAM 磁盘和支持初始 RAM 磁盘的情况下编译内核。此外，至少所有从 initrd 执行程序所需的组件（如可执行文件格式和文件系统）都必须编译到内核中。

Third, you have to create the RAM disk image. This is done by creating a file system on a block device, copying files to it as needed, and then copying the content of the block device to the initrd file. With recent kernels, at least three types of devices are suitable for that:

> 第三，您必须创建 RAM 磁盘映像。这是通过在块设备上创建一个文件系统，根据需要将文件复制到其中，然后将块设备的内容复制到 initrd 文件来完成的。对于最近的内核，至少有三种类型的设备适用于此：

> - a floppy disk (works everywhere but it\'s painfully slow)
> - a RAM disk (fast, but allocates physical memory)
> - a loopback device (the most elegant solution)

We\'ll describe the loopback device method:

> 1.  make sure loopback block devices are configured into the kernel
>
> 2.  create an empty file system of the appropriate size, e.g.:
>
>         # dd if=/dev/zero of=initrd bs=300k count=1
>         # mke2fs -F -m0 initrd
>
>     (if space is critical, you may want to use the Minix FS instead of Ext2)
>
> 3.  mount the file system, e.g.:
>
>         # mount -t ext2 -o loop initrd /mnt
>
> 4.  create the console device:
>
>         # mkdir /mnt/dev
>         # mknod /mnt/dev/console c 5 1
>
> 5.  copy all the files that are needed to properly use the initrd environment. Don\'t forget the most important file, `/sbin/init`
>
>     ::: note
>     ::: title
>     Note
>     :::
>
>     `/sbin/init` permissions must include \"x\" (execute).
>     :::
>
> 6.  correct operation the initrd environment can frequently be tested even without rebooting with the command:
>
>         # chroot /mnt /sbin/init
>
>     This is of course limited to initrds that do not interfere with the general system state (e.g. by reconfiguring network interfaces, overwriting mounted devices, trying to start already running demons, etc. Note however that it is usually possible to use pivot_root in such a chroot\'ed initrd environment.)
>
> 7.  unmount the file system:
>
>         # umount /mnt
>
> 8.  the initrd is now in the file \"initrd\". Optionally, it can now be compressed:
>
>         # gzip -9 initrd

For experimenting with initrd, you may want to take a rescue floppy and only add a symbolic link from `/sbin/init` to `/bin/sh`. Alternatively, you can try the experimental newlib environment[^2] to create a small initrd.

> 对于 initrd 的实验，您可能需要一张救援软盘，只添加一个从“/sbin/init”到“/bin/sh”的符号链接。或者，您可以尝试实验性的 newlib 环境[^2]来创建一个小的 initrd。

Finally, you have to boot the kernel and load initrd. Almost all Linux boot loaders support initrd. Since the boot process is still compatible with an older mechanism, the following boot command line parameters have to be given:

> 最后，您必须启动内核并加载 initrd。几乎所有的 Linux 引导加载程序都支持 initrd。由于引导过程仍然与旧机制兼容，因此必须给定以下引导命令行参数：

```
root=/dev/ram0 rw
```

(rw is only necessary if writing to the initrd file system.)

With `LOADLIN`, you simply execute:

```
LOADLIN <kernel> initrd=<disk_image>
```

e.g.:

```
LOADLIN C:\LINUX\BZIMAGE initrd=C:\LINUX\INITRD.GZ root=/dev/ram0 rw
```

With LILO, you add the option `INITRD=<path>` to either the global section or to the section of the respective kernel in `/etc/lilo.conf`, and pass the options using APPEND, e.g.:

> 使用 LILO，您可以将选项“INITRD=<path>'添加到全局部分或“/etc/LILO.conf”中相应内核的部分，并使用 APPEND 传递选项，例如：

```
image = /bzImage
    initrd = /boot/initrd.gz
    append = "root=/dev/ram0 rw"
```

and run `/sbin/lilo`

For other boot loaders, please refer to the respective documentation.

Now you can boot and enjoy using initrd.

# Changing the root device

When finished with its duties, init typically changes the root device and proceeds with starting the Linux system on the \"real\" root device.

> 完成任务后，init 通常会更改根设备，并在“真正的”根设备上启动 Linux 系统。

The procedure involves the following steps:

: - mounting the new root file system - turning it into the root file system - removing all accesses to the old (initrd) root file system - unmounting the initrd file system and de-allocating the RAM disk

Mounting the new root file system is easy: it just needs to be mounted on a directory under the current root. Example:

> 安装新的根文件系统很容易：它只需要安装在当前根目录下的一个目录上。实例

```
# mkdir /new-root
# mount -o ro /dev/hda1 /new-root
```

The root change is accomplished with the pivot_root system call, which is also available via the `pivot_root` utility (see `pivot_root(8)`{.interpreted-text role="manpage"} man page; `pivot_root` is distributed with util-linux version 2.10h or higher [^3]). `pivot_root` moves the current root to a directory under the new root, and puts the new root at its place. The directory for the old root must exist before calling `pivot_root`. Example:

> 根目录的更改是通过 pivot_root 系统调用完成的，该调用也可通过`pivot_root`实用程序获得（请参阅`pivot_root（8）`{.depredicted text role=“manpage”}手册页`pivot_root`与 util-linux 版本 2.10h 或更高版本一起分发[^3]）`pivot_root`将当前根移动到新根下的一个目录，并将新根放在其位置。在调用“pivot_root”之前，旧根目录必须存在。实例

```
# cd /new-root
# mkdir initrd
# pivot_root . initrd
```

Now, the init process may still access the old root via its executable, shared libraries, standard input/output/error, and its current root directory. All these references are dropped by the following command:

> 现在，init 进程仍然可以通过其可执行文件、共享库、标准输入/输出/错误及其当前根目录访问旧的根目录。以下命令将删除所有这些引用：

```
# exec chroot . what-follows <dev/console >dev/console 2>&1
```

Where what-follows is a program under the new root, e.g. `/sbin/init` If the new root file system will be used with udev and has no valid `/dev` directory, udev must be initialized before invoking chroot in order to provide `/dev/console`.

> 下面是新根目录下的程序，例如“/sbin/init”。如果新的根文件系统将与 udev 一起使用，并且没有有效的“/dev/”目录，则必须在调用 chroot 之前初始化 udev，以便提供“/dev/console”。

Note: implementation details of pivot_root may change with time. In order to ensure compatibility, the following points should be observed:

> 注意：pivot_root 的实现细节可能会随着时间的推移而变化。为了确保兼容性，应遵守以下几点：

> - before calling pivot_root, the current directory of the invoking process should point to the new root directory
> - use . as the first argument, and the \_[relative]() path of the directory for the old root as the second argument
> - a chroot program must be available under the old and the new root
> - chroot to the new root afterwards
> - use relative paths for dev/console in the exec command

Now, the initrd can be unmounted and the memory allocated by the RAM disk can be freed:

> 现在，可以卸载 initrd，并释放 RAM 磁盘分配的内存：

```
# umount /initrd
# blockdev --flushbufs /dev/ram0
```

It is also possible to use initrd with an NFS-mounted root, see the `pivot_root(8)`{.interpreted-text role="manpage"} man page for details.

> 还可以将 initrd 与 NFS 安装的根目录一起使用，请参阅`pivot_root（8）`{.depreced text role=“manpage”}手册页了解详细信息。

# Usage scenarios

The main motivation for implementing initrd was to allow for modular kernel configuration at system installation. The procedure would work as follows:

> 实现 initrd 的主要动机是允许在系统安装时进行模块化内核配置。该程序的工作原理如下：

> 1. system boots from floppy or other media with a minimal kernel (e.g. support for RAM disks, initrd, a.out, and the Ext2 FS) and loads initrd
> 2. `/sbin/init` determines what is needed to (1) mount the \"real\" root FS (i.e. device type, device drivers, file system) and (2) the distribution media (e.g. CD-ROM, network, tape, \...). This can be done by asking the user, by auto-probing, or by using a hybrid approach.
> 3. `/sbin/init` loads the necessary kernel modules
> 4. `/sbin/init` creates and populates the root file system (this doesn\'t have to be a very usable system yet)
> 5. `/sbin/init` invokes `pivot_root` to change the root file system and execs - via chroot - a program that continues the installation
> 6. the boot loader is installed
> 7. the boot loader is configured to load an initrd with the set of modules that was used to bring up the system (e.g. `/initrd` can be modified, then unmounted, and finally, the image is written from `/dev/ram0` or `/dev/rd/0` to a file)
> 8. now the system is bootable and additional installation tasks can be performed

The key role of initrd here is to re-use the configuration data during normal system operation without requiring the use of a bloated \"generic\" kernel or re-compiling or re-linking the kernel.

> initrd 在这里的关键作用是在正常系统操作期间重用配置数据，而不需要使用臃肿的“通用”内核或重新编译或链接内核。

A second scenario is for installations where Linux runs on systems with different hardware configurations in a single administrative domain. In such cases, it is desirable to generate only a small set of kernels (ideally only one) and to keep the system-specific part of configuration information as small as possible. In this case, a common initrd could be generated with all the necessary modules. Then, only `/sbin/init` or a file read by it would have to be different.

> 第二种情况是 Linux 在单个管理域中具有不同硬件配置的系统上运行的安装。在这种情况下，希望仅生成一小组内核（理想情况下仅生成一个），并保持配置信息的系统特定部分尽可能小。在这种情况下，可以生成一个带有所有必要模块的通用 initrd。那么，只有“/sbin/init”或它读取的文件必须不同。

A third scenario is more convenient recovery disks, because information like the location of the root FS partition doesn\'t have to be provided at boot time, but the system loaded from initrd can invoke a user-friendly dialog and it can also perform some sanity checks (or even some form of auto-detection).

> 第三种情况是更方便的恢复磁盘，因为在启动时不必提供根 FS 分区的位置等信息，但从 initrd 加载的系统可以调用用户友好的对话框，还可以执行一些健全性检查（甚至某种形式的自动检测）。

Last not least, CD-ROM distributors may use it for better installation from CD, e.g. by using a boot floppy and bootstrapping a bigger RAM disk via initrd from CD; or by booting via a loader like `LOADLIN` or directly from the CD-ROM, and loading the RAM disk from CD without need of floppies.

> 最后，CD-ROM 发行商可以使用它来更好地从 CD 安装，例如使用引导软盘，并通过 initrd 从 CD 引导更大的 RAM 磁盘；或者通过诸如“LOADLIN”之类的加载程序或者直接从 CD-ROM 引导，并且在不需要软盘的情况下从 CD 加载 RAM 盘。

# Obsolete root change mechanism

The following mechanism was used before the introduction of pivot_root. Current kernels still support it, but you should \_[not]() rely on its continued availability.

> 在引入 pivot_root 之前使用了以下机制。当前的内核仍然支持它，但您应该\_[不]（）依赖于它的持续可用性。

It works by mounting the \"real\" root device (i.e. the one set with rdev in the kernel image or with root=\... at the boot command line) as the root file system when linuxrc exits. The initrd file system is then unmounted, or, if it is still busy, moved to a directory `/initrd`, if such a directory exists on the new root file system.

> 它的工作方式是在 linuxrc 退出时安装\“real\”根设备（即在内核映像中使用 rdev 或在引导命令行中使用 root=\…设置的根设备）作为根文件系统。然后卸载 initrd 文件系统，或者，如果它仍然繁忙，则移动到目录“/initrd”（如果新的根文件系统上存在这样的目录）。

In order to use this mechanism, you do not have to specify the boot command options root, init, or rw. (If specified, they will affect the real root file system, not the initrd environment.)

> 为了使用此机制，您不必指定引导命令选项 root、init 或 rw。（如果指定，它们将影响真正的根文件系统，而不是 initrd 环境。）

If /proc is mounted, the \"real\" root device can be changed from within linuxrc by writing the number of the new root FS device to the special file /proc/sys/kernel/real-root-dev, e.g.:

> 如果安装了/proc，则可以通过将新根 FS 设备的编号写入特殊文件/proc/sys/kernel/real-root-dev 来从 linuxrc 中更改“real”根设备，例如：

```
# echo 0x301 >/proc/sys/kernel/real-root-dev
```

Note that the mechanism is incompatible with NFS and similar file systems.

> 请注意，该机制与 NFS 和类似的文件系统不兼容。

This old, deprecated mechanism is commonly called `change_root`, while the new, supported mechanism is called `pivot_root`.

> 这种旧的、不推荐使用的机制通常被称为“change_root”，而新的、受支持的机制称为“pivot_root”。

# Mixed change_root and pivot_root mechanism

In case you did not want to use `root=/dev/ram0` to trigger the pivot_root mechanism, you may create both `/linuxrc` and `/sbin/init` in your initrd image.

> 如果您不想使用“root=/dev/ram0”来触发 pivot_root 机制，您可以在 initrd 映像中同时创建“/linuxrc”和“/sbin/init”。

`/linuxrc` would contain only the following:

```
#! /bin/sh
mount -n -t proc proc /proc
echo 0x0100 >/proc/sys/kernel/real-root-dev
umount -n /proc
```

Once linuxrc exited, the kernel would mount again your initrd as root, this time executing `/sbin/init`. Again, it would be the duty of this init to build the right environment (maybe using the `root= device` passed on the cmdline) before the final execution of the real `/sbin/init`.

> 一旦 linuxrc 退出，内核将再次以 root 身份装载 initrd，这次执行“/sbin/init”。同样，这个 init 的职责是在最终执行真正的“/sbin/init”之前构建正确的环境（可能使用 cmdline 上传递的“root=device”）。

# Resources

[^1]: Almesberger, Werner; \"Booting Linux: The History and the Future\" <https://www.almesberger.net/cv/papers/ols2k-9.ps.gz>

<!-- ./ref/Booting Linux_The History and the Future@June 25, 2000/ -->

[^2]: newlib package (experimental), with initrd example <https://www.sourceware.org/newlib/>
[^3]: util-linux: Miscellaneous utilities for Linux <https://www.kernel.org/pub/linux/utils/util-linux/>
