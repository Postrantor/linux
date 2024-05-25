---
tip: translate by baidu@2024-01-30 21:33:03
---
---
title: Dell Remote BIOS Update driver (dell_rbu)
---

# Purpose


Document demonstrating the use of the Dell Remote BIOS Update driver for updating BIOS images on Dell servers and desktops.

> 演示使用Dell Remote BIOS Update驱动程序更新Dell服务器和台式机上的BIOS映像的说明文件。

# Scope


This document discusses the functionality of the rbu driver only. It does not cover the support needed from applications to enable the BIOS to update itself with the image downloaded in to the memory.

> 本文档仅讨论rbu驱动程序的功能。它不包括应用程序所需的支持，以使BIOS能够使用下载到内存中的映像进行自我更新。

# Overview


This driver works with Dell OpenManage or Dell Update Packages for updating the BIOS on Dell servers (starting from servers sold since 1999), desktops and notebooks (starting from those sold in 2005).

> 此驱动程序与Dell OpenManage或Dell Update Packages配合使用，用于更新Dell服务器（自1999年起销售的服务器）、台式机和笔记本电脑（自2005年起销售）上的BIOS。


Please go to <http://support.dell.com> register and you can find info on OpenManage and Dell Update packages (DUP).

> 请转到<http://support.dell.com>注册，您可以找到OpenManage和Dell Update软件包（DUP）的信息。


Libsmbios can also be used to update BIOS on Dell systems go to <https://linux.dell.com/libsmbios/> for details.

> Libsmbios也可以用于更新Dell系统上的BIOS。请转到<https://linux.dell.com/libsmbios/>详细信息。


Dell_RBU driver supports BIOS update using the monolithic image and packetized image methods. In case of monolithic the driver allocates a contiguous chunk of physical pages having the BIOS image. In case of packetized the app using the driver breaks the image in to packets of fixed sizes and the driver would place each packet in contiguous physical memory. The driver also maintains a link list of packets for reading them back.

> Dell_RBU驱动程序支持使用单片映像和分组映像方法进行BIOS更新。在单片的情况下，驱动程序分配具有BIOS映像的物理页的连续块。在打包的情况下，使用驱动程序的应用程序会将图像分解为固定大小的数据包，并且驱动程序会将每个数据包放置在连续的物理内存中。驱动程序还维护数据包的链接列表，以便将其读回。

If the dell_rbu driver is unloaded all the allocated memory is freed.


The rbu driver needs to have an application (as mentioned above) which will inform the BIOS to enable the update in the next system reboot.

> rbu驱动程序需要有一个应用程序（如上所述），该应用程序将通知BIOS在下次系统重新启动时启用更新。


The user should not unload the rbu driver after downloading the BIOS image or updating.

> 用户不应在下载BIOS映像或更新后卸载rbu驱动程序。


The driver load creates the following directories under the /sys file system:

> 驱动程序加载在/sys文件系统下创建以下目录：

    /sys/class/firmware/dell_rbu/loading
    /sys/class/firmware/dell_rbu/data
    /sys/devices/platform/dell_rbu/image_type
    /sys/devices/platform/dell_rbu/data
    /sys/devices/platform/dell_rbu/packet_size


The driver supports two types of update mechanism; monolithic and packetized. These update mechanism depends upon the BIOS currently running on the system. Most of the Dell systems support a monolithic update where the BIOS image is copied to a single contiguous block of physical memory.

> 驱动程序支持两种类型的更新机制；单片和打包。这些更新机制取决于系统上当前运行的BIOS。大多数Dell系统支持单片更新，其中BIOS映像被复制到单个连续的物理内存块。


In case of packet mechanism the single memory can be broken in smaller chunks of contiguous memory and the BIOS image is scattered in these packets.

> 在分组机制的情况下，单个存储器可以被分割成更小的连续存储器块，并且BIOS映像分散在这些分组中。


By default the driver uses monolithic memory for the update type. This can be changed to packets during the driver load time by specifying the load parameter image_type=packet. This can also be changed later as below:

> 默认情况下，驱动程序使用单片内存进行更新类型。通过指定加载参数image_type=packet，可以在驱动程序加载时间将其更改为数据包。这也可以在以后进行更改，如下所示：

    echo packet > /sys/devices/platform/dell_rbu/image_type


In packet update mode the packet size has to be given before any packets can be downloaded. It is done as below:

> 在数据包更新模式中，必须在下载任何数据包之前给定数据包大小。具体操作如下：

    echo XXXX > /sys/devices/platform/dell_rbu/packet_size


In the packet update mechanism, the user needs to create a new file having packets of data arranged back to back. It can be done as follows: The user creates packets header, gets the chunk of the BIOS image and places it next to the packetheader; now, the packetheader + BIOS image chunk added together should match the specified packet_size. This makes one packet, the user needs to create more such packets out of the entire BIOS image file and then arrange all these packets back to back in to one single file.

> 在包更新机制中，用户需要创建具有背靠背排列的数据包的新文件。可以这样做：用户创建数据包头，获取BIOS映像的块，并将其放在数据包头旁边；现在，添加在一起的packetheader+BIOS映像块应该与指定的packet_size匹配。这就形成了一个数据包，用户需要从整个BIOS映像文件中创建更多这样的数据包，然后将所有这些数据包背靠背地排列到一个文件中。


This file is then copied to /sys/class/firmware/dell_rbu/data. Once this file gets to the driver, the driver extracts packet_size data from the file and spreads it across the physical memory in contiguous packet_sized space.

> 然后将此文件复制到/sys/class/ffirmware/dell_rbu/data。一旦该文件到达驱动程序，驱动程序就会从文件中提取packet_size数据，并将其分布在连续的packet_sizespace中的物理内存中。


This method makes sure that all the packets get to the driver in a single operation.

> 此方法确保所有数据包在一次操作中到达驱动程序。


In monolithic update the user simply get the BIOS image (.hdr file) and copies to the data file as is without any change to the BIOS image itself.

> 在单片更新中，用户只需获得BIOS映像（.hdr文件）并按原样复制到数据文件，而不对BIOS映像本身进行任何更改。

Do the steps below to download the BIOS image.

1)  echo 1 \> /sys/class/firmware/dell_rbu/loading
2)  cp bios_image.hdr /sys/class/firmware/dell_rbu/data
3)  echo 0 \> /sys/class/firmware/dell_rbu/loading


The /sys/class/firmware/dell_rbu/ entries will remain till the following is done.

> /sys/class/ffirmware/dell_rbu/条目将保留，直到完成以下操作。

    echo -1 > /sys/class/firmware/dell_rbu/loading

Until this step is completed the driver cannot be unloaded.


Also echoing either mono, packet or init in to image_type will free up the memory allocated by the driver.

> 此外，将mono、packet或init回显到image_type将释放驱动程序分配的内存。


If a user by accident executes steps 1 and 3 above without executing step 2; it will make the /sys/class/firmware/dell_rbu/ entries disappear.

> 如果用户意外地执行了上述步骤1和3而没有执行步骤2；它将使/sys/class/ffirmware/dell_rbu/条目消失。

The entries can be recreated by doing the following:

    echo init > /sys/devices/platform/dell_rbu/image_type

::: note
::: title
Note
:::

echoing init in image_type does not change its original value.
:::


Also the driver provides /sys/devices/platform/dell_rbu/data readonly file to read back the image downloaded.

> 此外，驱动程序还提供/sys/devices/platform/dell_rbu/data只读文件来读回下载的图像。

::: note
::: title
Note
:::


After updating the BIOS image a user mode application needs to execute code which sends the BIOS update request to the BIOS. So on the next reboot the BIOS knows about the new image downloaded and it updates itself. Also don\'t unload the rbu driver if the image has to be updated.

> 在更新BIOS映像之后，用户模式应用程序需要执行向BIOS发送BIOS更新请求的代码。因此，在下次重新启动时，BIOS会知道下载的新映像，并自行更新。如果图像必须更新，也不要卸载rbu驱动程序。
:::
