---
tip: translate by baidu@2024-01-30 21:33:39
---
---
title: The EFI Boot Stub
---


On the x86 and ARM platforms, a kernel zImage/bzImage can masquerade as a PE/COFF image, thereby convincing EFI firmware loaders to load it as an EFI executable. The code that modifies the bzImage header, along with the EFI-specific entry point that the firmware loader jumps to are collectively known as the \"EFI boot stub\", and live in arch/x86/boot/header.S and drivers/firmware/efi/libstub/x86-stub.c, respectively. For ARM the EFI stub is implemented in arch/arm/boot/compressed/efi-header.S and drivers/firmware/efi/libstub/arm32-stub.c. EFI stub code that is shared between architectures is in drivers/firmware/efi/libstub.

> 在x86和ARM平台上，内核zImage/bzImage可以伪装成PE/COFF映像，从而说服EFI固件加载程序将其加载为EFI可执行文件。修改bzImage标头的代码，以及固件加载程序跳转到的EFI特定入口点，统称为“EFI引导存根”，位于arch/x86/boot/header中。S和驱动程序/固件/efi/libstub/x86 stub.c。对于ARM，EFI存根在arch/ARM/boot/compressed/EFI头中实现。S和drivers/ffirmware/efi/libstub/arm32 stub.c.体系结构之间共享的efi stub代码位于drivers/fFirmware/efi/libstub中。


For arm64, there is no compressed kernel support, so the Image itself masquerades as a PE/COFF image and the EFI stub is linked into the kernel. The arm64 EFI stub lives in drivers/firmware/efi/libstub/arm64.c and drivers/firmware/efi/libstub/arm64-stub.c.

> 对于arm64，没有压缩内核支持，因此映像本身伪装成PE/COFF映像，EFI存根链接到内核中。arm64 EFI存根存在于drivers/ffirmware/EFI/libstub/arm64.c和drivers/fifirmware-EFI/libstrub/arm64 stub.c中。


By using the EFI boot stub it\'s possible to boot a Linux kernel without the use of a conventional EFI boot loader, such as grub or elilo. Since the EFI boot stub performs the jobs of a boot loader, in a certain sense it *IS* the boot loader.

> 通过使用EFI引导存根，可以在不使用常规EFI引导加载程序（如grub或elilo）的情况下引导Linux内核。由于EFI引导存根执行引导加载程序的工作，在某种意义上，它*IS*是引导加载程序。

The EFI boot stub is enabled with the CONFIG_EFI_STUB kernel option.

# How to install bzImage.efi


The bzImage located in arch/x86/boot/bzImage must be copied to the EFI System Partition (ESP) and renamed with the extension \".efi\". Without the extension the EFI firmware loader will refuse to execute it. It\'s not possible to execute bzImage.efi from the usual Linux file systems because EFI firmware doesn\'t have support for them. For ARM the arch/arm/boot/zImage should be copied to the system partition, and it may not need to be renamed. Similarly for arm64, arch/arm64/boot/Image should be copied but not necessarily renamed.

> 位于arch/x86/boot/bzImage中的bzImage必须复制到EFI系统分区（ESP），并用扩展名“.EFI”重命名。如果没有扩展名，EFI固件加载程序将拒绝执行它。无法从通常的Linux文件系统执行bzImage.EFI，因为EFI固件不支持它们。对于ARM，arch/ARM/boot/zImage应该复制到系统分区，并且可能不需要重命名。与arm64类似，应该复制arch/arm64/boot/Image，但不一定要重命名。

# Passing kernel parameters from the EFI shell

Arguments to the kernel can be passed after bzImage.efi, e.g.:

    fs0:> bzImage.efi console=ttyS0 root=/dev/sda4

# The \"initrd=\" option


Like most boot loaders, the EFI stub allows the user to specify multiple initrd files using the \"initrd=\" option. This is the only EFI stub-specific command line parameter, everything else is passed to the kernel when it boots.

> 与大多数引导加载程序一样，EFI存根允许用户使用\“initrd=\”选项指定多个initrd文件。这是唯一一个EFI存根特定的命令行参数，其他所有参数都会在内核启动时传递给内核。


The path to the initrd file must be an absolute path from the beginning of the ESP, relative path names do not work. Also, the path is an EFI-style path and directory elements must be separated with backslashes (). For example, given the following directory layout:

> initrd文件的路径必须是从ESP开始的绝对路径，相对路径名不起作用。此外，该路径是EFI样式的路径，目录元素必须用反斜杠（）分隔。例如，给定以下目录布局：

    fs0:>
      Kernels\
              bzImage.efi
              initrd-large.img

      Ramdisks\
              initrd-small.img
              initrd-medium.img


to boot with the initrd-large.img file if the current working directory is fs0:Kernels, the following command must be used:

> 如果当前工作目录是fs0:Kernels，则必须使用以下命令才能使用initrd-large.img文件启动：

    fs0:\Kernels> bzImage.efi initrd=\Kernels\initrd-large.img


Notice how bzImage.efi can be specified with a relative path. That\'s because the image we\'re executing is interpreted by the EFI shell, which understands relative paths, whereas the rest of the command line is passed to bzImage.efi.

> 请注意如何使用相对路径指定bzImage.efi。这是因为我们正在执行的映像由EFI shell解释，EFI shell理解相对路径，而命令行的其余部分则传递给bzImage.EFI。

# The \"dtb=\" option


For the ARM and arm64 architectures, a device tree must be provided to the kernel. Normally firmware shall supply the device tree via the EFI CONFIGURATION TABLE. However, the \"dtb=\" command line option can be used to override the firmware supplied device tree, or to supply one when firmware is unable to.

> 对于ARM和arm64体系结构，必须向内核提供设备树。通常固件应通过EFI配置表提供设备树。但是，\“dtb=\”命令行选项可用于覆盖固件提供的设备树，或者在固件无法覆盖时提供一个。


Please note: Firmware adds runtime configuration information to the device tree before booting the kernel. If dtb= is used to override the device tree, then any runtime data provided by firmware will be lost. The dtb= option should only be used either as a debug tool, or as a last resort when a device tree is not provided in the EFI CONFIGURATION TABLE.

> 请注意：固件在引导内核之前将运行时配置信息添加到设备树中。如果使用dtb=覆盖设备树，则固件提供的任何运行时数据都将丢失。dtb=选项只能用作调试工具，或者在EFI CONFIGURATION TABLE中未提供设备树时作为最后手段。


\"dtb=\" is processed in the same manner as the \"initrd=\" option that is described above.

> \“dtb=\”的处理方式与上面描述的“initrd=\”选项相同。
