---
tip: translate by baidu@2024-01-30 21:40:22
---
---
title: USB4 and Thunderbolt
---


USB4 is the public specification based on Thunderbolt 3 protocol with some differences at the register level among other things. Connection manager is an entity running on the host router (host controller) responsible for enumerating routers and establishing tunnels. A connection manager can be implemented either in firmware or software. Typically PCs come with a firmware connection manager for Thunderbolt 3 and early USB4 capable systems. Apple systems on the other hand use software connection manager and the later USB4 compliant devices follow the suit.

> USB4是基于Thunderbolt 3协议的公共规范，在寄存器级别等方面存在一些差异。连接管理器是在主机路由器（主机控制器）上运行的实体，负责枚举路由器和建立隧道。连接管理器可以用固件或软件来实现。通常，PC配有适用于Thunderbolt 3和早期支持USB4的系统的固件连接管理器。另一方面，苹果系统使用软件连接管理器，后来兼容USB4的设备也紧随其后。


The Linux Thunderbolt driver supports both and can detect at runtime which connection manager implementation is to be used. To be on the safe side the software connection manager in Linux also advertises security level `user` which means PCIe tunneling is disabled by default. The documentation below applies to both implementations with the exception that the software connection manager only supports `user` security level and is expected to be accompanied with an IOMMU based DMA protection.

> Linux Thunderbolt驱动程序同时支持这两种功能，并且可以在运行时检测要使用的连接管理器实现。为了安全起见，Linux中的软件连接管理器还宣传安全级别“用户”，这意味着默认情况下禁用PCIe隧道。以下文档适用于这两种实现，但软件连接管理器仅支持“用户”安全级别，并应附带基于IOMMU的DMA保护。

# Security levels and how to use them


The interface presented here is not meant for end users. Instead there should be a userspace tool that handles all the low-level details, keeps a database of the authorized devices and prompts users for new connections.

> 此处提供的界面不适用于最终用户。相反，应该有一个用户空间工具来处理所有底层细节，保存授权设备的数据库，并提示用户进行新的连接。


More details about the sysfs interface for Thunderbolt devices can be found in `Documentation/ABI/testing/sysfs-bus-thunderbolt`.

> 有关Thunderbolt设备的sysfs接口的更多详细信息，请参阅“Documentation/ABI/testing/sysfs bus Thunderbolt”。


Those users who just want to connect any device without any sort of manual work can add following line to `/etc/udev/rules.d/99-local.rules`:

> 那些只想连接任何设备而不需要任何手动操作的用户可以在“/etc/udev/rules.d/99 local.rules”中添加以下行：

    ACTION=="add", SUBSYSTEM=="thunderbolt", ATTR{authorized}=="0", ATTR{authorized}="1"


This will authorize all devices automatically when they appear. However, keep in mind that this bypasses the security levels and makes the system vulnerable to DMA attacks.

> 这将在所有设备出现时自动授权它们。但是，请记住，这会绕过安全级别，使系统容易受到DMA攻击。


Starting with Intel Falcon Ridge Thunderbolt controller there are 4 security levels available. Intel Titan Ridge added one more security level (usbonly). The reason for these is the fact that the connected devices can be DMA masters and thus read contents of the host memory without CPU and OS knowing about it. There are ways to prevent this by setting up an IOMMU but it is not always available for various reasons.

> 从Intel Falcon Ridge Thunderbolt控制器开始，有4个安全级别可用。英特尔Titan Ridge又增加了一个安全级别（usbonly）。原因是连接的设备可以是DMA主机，从而在CPU和操作系统不知情的情况下读取主机内存的内容。有一些方法可以通过设置IOMMU来防止这种情况，但由于各种原因，它并不总是可用的。


Some USB4 systems have a BIOS setting to disable PCIe tunneling. This is treated as another security level (nopcie).

> 一些USB4系统具有禁用PCIe隧道的BIOS设置。这被视为另一个安全级别（nopcie）。

The security levels are as follows:

> none
>
> :   All devices are automatically connected by the firmware. No user approval is needed. In BIOS settings this is typically called *Legacy mode*.
>
> user
>
> :   User is asked whether the device is allowed to be connected. Based on the device identification information available through `/sys/bus/thunderbolt/devices`, the user then can make the decision. In BIOS settings this is typically called *Unique ID*.
>
> secure
>
> :   User is asked whether the device is allowed to be connected. In addition to UUID the device (if it supports secure connect) is sent a challenge that should match the expected one based on a random key written to the `key` sysfs attribute. In BIOS settings this is typically called *One time saved key*.
>
> dponly
>
> :   The firmware automatically creates tunnels for Display Port and USB. No PCIe tunneling is done. In BIOS settings this is typically called *Display Port Only*.
>
> usbonly
>
> :   The firmware automatically creates tunnels for the USB controller and Display Port in a dock. All PCIe links downstream of the dock are removed.
>
> nopcie
>
> :   PCIe tunneling is disabled/forbidden from the BIOS. Available in some USB4 systems.


The current security level can be read from `/sys/bus/thunderbolt/devices/domainX/security` where `domainX` is the Thunderbolt domain the host controller manages. There is typically one domain per Thunderbolt host controller.

> 当前安全级别可以从“/sys/bus/threnbolt/devices/domainX/security”读取，其中“domainX”是主机控制器管理的thunderbolt域。每个Thunderbolt主机控制器通常有一个域。


If the security level reads as `user` or `secure` the connected device must be authorized by the user before PCIe tunnels are created (e.g the PCIe device appears).

> 如果安全级别读作“用户”或“安全”，则在创建PCIe隧道（例如，PCIe设备出现）之前，连接的设备必须由用户授权。


Each Thunderbolt device plugged in will appear in sysfs under `/sys/bus/thunderbolt/devices`. The device directory carries information that can be used to identify the particular device, including its name and UUID.

> 每个插入的Thunderbolt设备都将出现在sysfs中的“/sys/bus/threnderbolt/devices”下。设备目录包含可用于识别特定设备的信息，包括其名称和UUID。

# Authorizing devices when security level is `user` or `secure`

When a device is plugged in it will appear in sysfs as follows:

    /sys/bus/thunderbolt/devices/0-1/authorized   - 0
    /sys/bus/thunderbolt/devices/0-1/device   - 0x8004
    /sys/bus/thunderbolt/devices/0-1/device_name  - Thunderbolt to FireWire Adapter
    /sys/bus/thunderbolt/devices/0-1/vendor   - 0x1
    /sys/bus/thunderbolt/devices/0-1/vendor_name  - Apple, Inc.
    /sys/bus/thunderbolt/devices/0-1/unique_id    - e0376f00-0300-0100-ffff-ffffffffffff


The `authorized` attribute reads 0 which means no PCIe tunnels are created yet. The user can authorize the device by simply entering:

> “authorized”属性读取为0，这意味着尚未创建PCIe隧道。用户只需输入以下内容即可对设备进行授权：

    # echo 1 > /sys/bus/thunderbolt/devices/0-1/authorized

This will create the PCIe tunnels and the device is now connected.


If the device supports secure connect, and the domain security level is set to `secure`, it has an additional attribute `key` which can hold a random 32-byte value used for authorization and challenging the device in future connects:

> 如果设备支持安全连接，并且域安全级别设置为“安全”，则它具有一个附加属性“密钥”，该属性可以保存一个随机的32字节值，用于授权和在未来的连接中挑战设备：

    /sys/bus/thunderbolt/devices/0-3/authorized   - 0
    /sys/bus/thunderbolt/devices/0-3/device   - 0x305
    /sys/bus/thunderbolt/devices/0-3/device_name  - AKiTiO Thunder3 PCIe Box
    /sys/bus/thunderbolt/devices/0-3/key      -
    /sys/bus/thunderbolt/devices/0-3/vendor   - 0x41
    /sys/bus/thunderbolt/devices/0-3/vendor_name  - inXtron
    /sys/bus/thunderbolt/devices/0-3/unique_id    - dc010000-0000-8508-a22d-32ca6421cb16

Notice the key is empty by default.


If the user does not want to use secure connect they can just `echo 1` to the `authorized` attribute and the PCIe tunnels will be created in the same way as in the `user` security level.

> 如果用户不想使用安全连接，则他们可以仅“回显1”到“授权”属性，并且PCIe隧道将以与“用户”安全级别相同的方式创建。


If the user wants to use secure connect, the first time the device is plugged a key needs to be created and sent to the device:

> 如果用户想要使用安全连接，则在第一次插入设备时，需要创建密钥并将其发送到设备：

    # key=$(openssl rand -hex 32)
    # echo $key > /sys/bus/thunderbolt/devices/0-3/key
    # echo 1 > /sys/bus/thunderbolt/devices/0-3/authorized


Now the device is connected (PCIe tunnels are created) and in addition the key is stored on the device NVM.

> 现在设备已连接（创建PCIe隧道），此外密钥存储在设备NVM上。

Next time the device is plugged in the user can verify (challenge) the device using the same key:

    # echo $key > /sys/bus/thunderbolt/devices/0-3/key
    # echo 2 > /sys/bus/thunderbolt/devices/0-3/authorized


If the challenge the device returns back matches the one we expect based on the key, the device is connected and the PCIe tunnels are created. However, if the challenge fails no tunnels are created and error is returned to the user.

> 如果设备返回的挑战与我们根据密钥预期的挑战相匹配，则设备已连接并创建PCIe隧道。但是，如果质询失败，则不会创建隧道，并将错误返回给用户。


If the user still wants to connect the device they can either approve the device without a key or write a new key and write 1 to the `authorized` file to get the new key stored on the device NVM.

> 如果用户仍然想连接该设备，他们可以在没有密钥的情况下批准该设备，或者写入新密钥并将1写入“授权”文件以获得存储在设备NVM上的新密钥。

# De-authorizing devices


It is possible to de-authorize devices by writing `0` to their `authorized` attribute. This requires support from the connection manager implementation and can be checked by reading domain `deauthorization` attribute. If it reads `1` then the feature is supported.

> 可以通过将“0”写入设备的“已授权”属性来取消对设备的授权。这需要连接管理器实现的支持，并且可以通过读取域“取消授权”属性进行检查。如果其读数为“1”，则支持该功能。


When a device is de-authorized the PCIe tunnel from the parent device PCIe downstream (or root) port to the device PCIe upstream port is torn down. This is essentially the same thing as PCIe hot-remove and the PCIe toplogy in question will not be accessible anymore until the device is authorized again. If there is storage such as NVMe or similar involved, there is a risk for data loss if the filesystem on that storage is not properly shut down. You have been warned!

> 当设备被取消授权时，从父设备PCIe下游（或根）端口到设备PCIe上游端口的PCIe隧道被拆除。这与PCIe热插拔基本相同，在设备再次获得授权之前，有问题的PCIe拓扑将无法再访问。如果涉及NVMe或类似存储，如果该存储上的文件系统未正确关闭，则存在数据丢失的风险。你已经被警告了！

# DMA protection utilizing IOMMU


Recent systems from 2018 and forward with Thunderbolt ports may natively support IOMMU. This means that Thunderbolt security is handled by an IOMMU so connected devices cannot access memory regions outside of what is allocated for them by drivers. When Linux is running on such system it automatically enables IOMMU if not enabled by the user already. These systems can be identified by reading `1` from `/sys/bus/thunderbolt/devices/domainX/iommu_dma_protection` attribute.

> 从2018年起的最新系统以及之前的Thunderbolt端口可能会原生支持IOMMU。这意味着Thunderbolt安全性由IOMMU处理，因此连接的设备无法访问驱动程序为其分配的内存区域之外的内存区域。当Linux在这样的系统上运行时，如果用户尚未启用IOMMU，它会自动启用IOMMU。可以通过从“/sys/bus/threnbolt/devices/domainX/iommu_dma_protection”属性读取“1”来识别这些系统。


The driver does not do anything special in this case but because DMA protection is handled by the IOMMU, security levels (if set) are redundant. For this reason some systems ship with security level set to `none`. Other systems have security level set to `user` in order to support downgrade to older OS, so users who want to automatically authorize devices when IOMMU DMA protection is enabled can use the following `udev` rule:

> 在这种情况下，驱动程序不执行任何特殊操作，但由于DMA保护由IOMMU处理，因此安全级别（如果设置）是冗余的。出于这个原因，一些系统的安全级别设置为“无”。其他系统的安全级别设置为“user”，以支持降级到较旧的操作系统，因此希望在启用IOMMU DMA保护时自动授权设备的用户可以使用以下“udev”规则：

    ACTION=="add", SUBSYSTEM=="thunderbolt", ATTRS{iommu_dma_protection}=="1", ATTR{authorized}=="0", ATTR{authorized}="1"

# Upgrading NVM on Thunderbolt device, host or retimer


Since most of the functionality is handled in firmware running on a host controller or a device, it is important that the firmware can be upgraded to the latest where possible bugs in it have been fixed. Typically OEMs provide this firmware from their support site.

> 由于大多数功能都是在主机控制器或设备上运行的固件中处理的，因此重要的是，固件可以升级到最新版本，其中可能的错误已经修复。OEM通常从其支持站点提供此固件。


There is also a central site which has links where to download firmware for some machines:

> 还有一个中心网站，它有一些链接可以下载一些机器的固件：

> [Thunderbolt Updates](https://thunderbolttechnology.net/updates)


Before you upgrade firmware on a device, host or retimer, please make sure it is a suitable upgrade. Failing to do that may render the device in a state where it cannot be used properly anymore without special tools!

> 在升级设备、主机或重定时器上的固件之前，请确保它是合适的升级。如果不这样做，可能会使设备处于没有专用工具就无法正常使用的状态！

Host NVM upgrade on Apple Macs is not supported.


Once the NVM image has been downloaded, you need to plug in a Thunderbolt device so that the host controller appears. It does not matter which device is connected (unless you are upgrading NVM on a device - then you need to connect that particular device).

> 下载NVM映像后，您需要插入Thunderbolt设备，以便显示主机控制器。连接的设备并不重要（除非您正在升级设备上的NVM，否则您需要连接该特定设备）。


Note an OEM-specific method to power the controller up (\"force power\") may be available for your system in which case there is no need to plug in a Thunderbolt device.

> 请注意，您的系统可能有OEM特定的控制器通电方法（“强制通电”），在这种情况下，无需插入Thunderbolt设备。


After that we can write the firmware to the non-active parts of the NVM of the host or device. As an example here is how Intel NUC6i7KYK (Skull Canyon) Thunderbolt controller NVM is upgraded:

> 之后，我们可以将固件写入主机或设备的NVM的非活动部分。以下是Intel NUC6i7KYK（Skull Canyon）Thunderbolt控制器NVM的升级示例：

    # dd if=KYK_TBT_FW_0018.bin of=/sys/bus/thunderbolt/devices/0-0/nvm_non_active0/nvmem


Once the operation completes we can trigger NVM authentication and upgrade process as follows:

> 操作完成后，我们可以触发NVM身份验证和升级过程，如下所示：

    # echo 1 > /sys/bus/thunderbolt/devices/0-0/nvm_authenticate


If no errors are returned, the host controller shortly disappears. Once it comes back the driver notices it and initiates a full power cycle. After a while the host controller appears again and this time it should be fully functional.

> 如果没有返回错误，主机控制器将很快消失。一旦它回来，驾驶员就会注意到它，并启动一个完整的电源循环。过了一段时间，主机控制器再次出现，这一次它应该完全正常工作。


We can verify that the new NVM firmware is active by running the following commands:

> 我们可以通过运行以下命令来验证新的NVM固件是否处于活动状态：

    # cat /sys/bus/thunderbolt/devices/0-0/nvm_authenticate
    0x0
    # cat /sys/bus/thunderbolt/devices/0-0/nvm_version
    18.0


If `nvm_authenticate` contains anything other than 0x0 it is the error code from the last authentication cycle, which means the authentication of the NVM image failed.

> 如果“nvm_authenticate”包含0x0以外的任何内容，则它是上一个身份验证周期的错误代码，这意味着nvm映像的身份验证失败。


Note names of the NVMem devices `nvm_activeN` and `nvm_non_activeN` depend on the order they are registered in the NVMem subsystem. N in the name is the identifier added by the NVMem subsystem.

> 注意NVMem设备的名称“nvm_activeN”和“nvm_non_activeN”取决于它们在NVMem子系统中的注册顺序。名称中的N是NVMem子系统添加的标识符。

# Upgrading on-board retimer NVM when there is no cable connected


If the platform supports, it may be possible to upgrade the retimer NVM firmware even when there is nothing connected to the USB4 ports. When this is the case the `usb4_portX` devices have two special attributes: `offline` and `rescan`. The way to upgrade the firmware is to first put the USB4 port into offline mode:

> 如果平台支持，即使没有连接到USB4端口，也可以升级重定时器NVM固件。在这种情况下，“usb4_portX”设备有两个特殊属性：“脱机”和“重新扫描”。升级固件的方法是首先将USB4端口置于离线模式：

    # echo 1 > /sys/bus/thunderbolt/devices/0-0/usb4_port1/offline


This step makes sure the port does not respond to any hotplug events, and also ensures the retimers are powered on. The next step is to scan for the retimers:

> 此步骤确保端口不响应任何热插拔事件，并确保重定时器已通电。下一步是扫描重定时器：

    # echo 1 > /sys/bus/thunderbolt/devices/0-0/usb4_port1/rescan


This enumerates and adds the on-board retimers. Now retimer NVM can be upgraded in the same way than with cable connected (see previous section). However, the retimer is not disconnected as we are offline mode) so after writing `1` to `nvm_authenticate` one should wait for 5 or more seconds before running rescan again:

> 这将枚举并添加板载重定时器。现在，重定时器NVM可以以与连接电缆相同的方式升级（请参阅上一节）。然而，重定时器并没有断开连接，因为我们处于脱机模式），因此在将“1”写入“nvm_authenticate”后，应等待5秒或更长时间，然后再运行重新扫描：

    # echo 1 > /sys/bus/thunderbolt/devices/0-0/usb4_port1/rescan


This point if everything went fine, the port can be put back to functional state again:

> 此时，如果一切顺利，端口可以再次恢复到功能状态：

    # echo 0 > /sys/bus/thunderbolt/devices/0-0/usb4_port1/offline

# Upgrading NVM when host controller is in safe mode


If the existing NVM is not properly authenticated (or is missing) the host controller goes into safe mode which means that the only available functionality is flashing a new NVM image. When in this mode, reading `nvm_version` fails with `ENODATA` and the device identification information is missing.

> 如果现有NVM未正确验证（或丢失），主机控制器将进入安全模式，这意味着唯一可用的功能是闪烁新的NVM映像。在此模式下，读取“nvm_version”失败，并显示“ENODATA”，并且设备标识信息丢失。


To recover from this mode, one needs to flash a valid NVM image to the host controller in the same way it is done in the previous chapter.

> 要从该模式恢复，需要以与前一章中相同的方式将有效的NVM映像闪存到主机控制器。

# Networking over Thunderbolt cable


Thunderbolt technology allows software communication between two hosts connected by a Thunderbolt cable.

> Thunderbolt技术允许通过Thunderbolt电缆连接的两台主机之间进行软件通信。


It is possible to tunnel any kind of traffic over a Thunderbolt link but currently we only support Apple ThunderboltIP protocol.

> 可以通过Thunderbolt链路传输任何类型的流量，但目前我们只支持Apple ThunderboltIP协议。


If the other host is running Windows or macOS, the only thing you need to do is to connect a Thunderbolt cable between the two hosts; the `thunderbolt-net` driver is loaded automatically. If the other host is also Linux you should load `thunderbolt-net` manually on one host (it does not matter which one):

> 如果另一台主机运行的是Windows或macOS，您只需要在两台主机之间连接一根Thunderbolt电缆；“雷电网”驱动程序是自动加载的。如果另一台主机也是Linux，则应在一台主机上手动加载“雷电网”（无论是哪台）：

    # modprobe thunderbolt-net


This triggers module load on the other host automatically. If the driver is built-in to the kernel image, there is no need to do anything.

> 这会自动触发另一台主机上的模块加载。如果驱动程序内置在内核映像中，则无需执行任何操作。


The driver will create one virtual ethernet interface per Thunderbolt port which are named like `thunderbolt0` and so on. From this point you can either use standard userspace tools like `ifconfig` to configure the interface or let your GUI handle it automatically.

> 驱动程序将为每个Thunderbolt端口创建一个虚拟以太网接口，这些接口的名称如“thunderbolt0”等。从这一点开始，您可以使用标准的用户空间工具（如“ifconfig”）来配置接口，也可以让GUI自动处理。

# Forcing power


Many OEMs include a method that can be used to force the power of a Thunderbolt controller to an \"On\" state even if nothing is connected. If supported by your machine this will be exposed by the WMI bus with a sysfs attribute called \"force_power\".

> 许多原始设备制造商都提供了一种方法，即使没有连接任何设备，也可以使用该方法强制Thunderbolt控制器的电源处于“打开”状态。如果您的计算机支持，则WMI总线将使用名为“force_power\”的sysfs属性公开此属性。

For example the intel-wmi-thunderbolt driver exposes this attribute in:

:   /sys/bus/wmi/devices/86CCFD48-205E-4A77-9C48-2021CBEDE341/force_power

    To force the power to on, write 1 to this attribute file. To disable force power, write 0 to this attribute file.


Note: it\'s currently not possible to query the force power state of a platform.

> 注意：目前无法查询平台的强制力状态。
