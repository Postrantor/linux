---
tip: translate by baidu@2024-01-30 21:29:43
title: The Android binderfs Filesystem
---

Android binderfs is a filesystem for the Android binder IPC mechanism. It allows to dynamically add and remove binder devices at runtime. Binder devices located in a new binderfs instance are independent of binder devices located in other binderfs instances. Mounting a new binderfs instance makes it possible to get a set of private binder devices.

> Android 绑定器是一个用于 Android 绑定器 IPC 机制的文件系统。它允许在运行时动态添加和删除绑定器设备。位于新绑定器实例中的绑定器设备独立于位于其他绑定器实例的绑定器装置。安装一个新的 binderfs 实例可以获得一组私有绑定设备。

# Mounting binderfs

Android binderfs can be mounted with:

```
mkdir /dev/binderfs
mount -t binder binder /dev/binderfs
```

at which point a new instance of binderfs will show up at `/dev/binderfs`. In a fresh instance of binderfs no binder devices will be present. There will only be a `binder-control` device which serves as the request handler for binderfs. Mounting another binderfs instance at a different location will create a new and separate instance from all other binderfs mounts. This is identical to the behavior of e.g. `devpts` and `tmpfs`. The Android binderfs filesystem can be mounted in user namespaces.

> 此时，binderfs 的一个新实例将显示在“/dev/binderfs”中。在装订器的新实例中，将不存在装订器装置。将只有一个“绑定器控制”设备作为绑定器的请求处理程序。在不同的位置装载另一个 binderfs 实例将创建一个新的实例，并与所有其他 binderfs 装载分离。这与例如“devpts”和“tmpfs”的行为相同。Android binderfs 文件系统可以安装在用户名称空间中。

# Options

max

: binderfs instances can be mounted with a limit on the number of binder devices that can be allocated. The `max=<count>` mount option serves as a per-instance limit. If `max=<count>` is set then only `<count>` number of binder devices can be allocated in this binderfs instance.

> ：binderfs 实例可以在限制可以分配的绑定器设备数量的情况下安装。“max=<count>”装载选项用作每个实例的限制。如果设置了“max=<count>'”，则在该绑定器实例中只能分配“<count>`数量的绑定器设备。

stats

: Using `stats=global` enables global binder statistics. `stats=global` is only available for a binderfs instance mounted in the initial user namespace. An attempt to use the option to mount a binderfs instance in another user namespace will return a permission error.

> ：使用“stats=global”可启用全局活页夹统计信息`stats=global`仅适用于安装在初始用户命名空间中的 binderfs 实例。尝试使用该选项在另一个用户命名空间中装载 binderfs 实例将返回权限错误。

# Allocating binder Devices

To allocate a new binder device in a binderfs instance a request needs to be sent through the `binder-control` device node. A request is sent in the form of an [ioctl()](http://man7.org/linux/man-pages/man2/ioctl.2.html).

> 要在绑定器实例中分配新的绑定器设备，需要通过“绑定器控制”设备节点发送请求。请求以[ioctl()]的形式发送(http://man7.org/linux/man-pages/man2/ioctl.2.html).

What a program needs to do is to open the `binder-control` device node and send a `BINDER_CTL_ADD` request to the kernel. Users of binderfs need to tell the kernel which name the new binder device should get. By default a name can only contain up to `BINDERFS_MAX_NAME` chars including the terminating zero byte.

> 程序需要做的是打开“绑定器控制”设备节点，并向内核发送“BINDER_CTL_ADD”请求。绑定器的用户需要告诉内核新的绑定器设备应该取哪个名称。默认情况下，名称最多只能包含“BINDERFS_MAX_NAME”个字符，包括终止的零字节。

Once the request is made via an [ioctl()](http://man7.org/linux/man-pages/man2/ioctl.2.html) passing a `struct binder_device` with the name to the kernel it will allocate a new binder device and return the major and minor number of the new device in the struct (This is necessary because binderfs allocates a major device number dynamically.). After the [ioctl()](http://man7.org/linux/man-pages/man2/ioctl.2.html) returns there will be a new binder device located under /dev/binderfs with the chosen name.

> 一旦通过[ioctl()]发出请求(http://man7.org/linux/man-pages/man2/ioctl.2.html)将带有名称的“structbinder_device”传递给内核，它将分配一个新的绑定器设备，并返回结构中新设备的主要和次要编号（这是必要的，因为binderfs动态分配主要设备编号。）(http://man7.org/linux/man-pages/man2/ioctl.2.html)返回将有一个新的绑定器设备位于/dev/binderfs下，名称为所选名称。

# Deleting binder Devices

Binderfs binder devices can be deleted via [unlink()](http://man7.org/linux/man-pages/man2/unlink.2.html). This means that the [rm()](http://man7.org/linux/man-pages/man1/rm.1.html) tool can be used to delete them. Note that the `binder-control` device cannot be deleted since this would make the binderfs instance unusable. The `binder-control` device will be deleted when the binderfs instance is unmounted and all references to it have been dropped.

> Binderfs 绑定器设备可以通过[unlink()]删除(http://man7.org/linux/man-pages/man2/unlink.2.html). 这意味着[rm()](http://man7.org/linux/man-pages/man1/rm.1.html)工具可以用来删除它们。请注意，不能删除“绑定器控制”设备，因为这将使绑定器实例不可用。当 binderfs 实例被卸载并且所有对它的引用都被删除时，“binder control”设备将被删除。

# Binder features

Assuming an instance of binderfs has been mounted at `/dev/binderfs`, the features supported by the binder driver can be located under `/dev/binderfs/features/`. The presence of individual files can be tested to determine whether a particular feature is supported by the driver.

> 假设 binderfs 的一个实例已安装在“/dev/binderfs”中，则绑定驱动程序支持的功能可以位于“/dev/binders/features/”下。可以测试单个文件的存在，以确定驱动程序是否支持特定功能。

Example:

```
cat /dev/binderfs/features/oneway_spam_detection
1
```
