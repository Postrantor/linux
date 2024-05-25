---
tip: translate by baidu@2024-01-30 21:34:27
author:
  - Andreas Mohr \<andi at lisas period de\> Cristian Souza \<cristianmsbr at gmail period com\>
title: Explaining the \"No working init found.\" boot hang message
---

This document provides some high-level reasons for failure (listed roughly in order of execution) to load the init binary.

> 本文档提供了加载 init 二进制文件失败的一些高级原因（大致按执行顺序列出）。

1.  **Unable to mount root FS**: Set \"debug\" kernel parameter (in bootloader config file or CONFIG_CMDLINE) to get more detailed kernel messages.

> 1） **无法装载根 FS**：设置\“debug\”内核参数（在引导程序配置文件或 config_CMDLINE 中）以获取更详细的内核消息。

2.  **init binary doesn\'t exist on rootfs**: Make sure you have the correct root FS type (and `root=` kernel parameter points to the correct partition), required drivers such as storage hardware (such as SCSI or USB!) and filesystem (ext3, jffs2, etc.) are builtin (alternatively as modules, to be pre-loaded by an initrd).

> 2） **init 二进制文件在 rootfs 上不存在**：确保您有正确的根 FS 类型（并且'root='kernel 参数指向正确的分区），所需的驱动程序，如存储硬件（如 SCSI 或 USB！）和文件系统（ext3、jffs2 等）都是内置的（或者作为模块，由 initrd 预加载）。

3.  **Broken console device**: Possibly a conflict in `console= setup` \--\> initial console unavailable. E.g. some serial consoles are unreliable due to serial IRQ issues (e.g. missing interrupt-based configuration). Try using a different `console= device` or e.g. `netconsole=`.

> 3） **控制台设备损坏**：可能是“console=setup”中的冲突\-->初始控制台不可用。例如，由于串行 IRQ 问题（例如，缺少基于中断的配置），一些串行控制台不可靠。请尝试使用不同的“console=device”或例如“netconsole=”。

4.  **Binary exists but dependencies not available**: E.g. required library dependencies of the init binary such as `/lib/ld-linux.so.2` missing or broken. Use `readelf -d <INIT>|grep NEEDED` to find out which libraries are required.

> 4） **二进制文件存在，但依赖项不可用**：例如，init 二进制文件的必需库依赖项，如“/lib/ld linux.so.2”丢失或损坏。使用`readelf-d<INIT>|grepNeed`找出需要哪些库。

5.  **Binary cannot be loaded**: Make sure the binary\'s architecture matches your hardware. E.g. i386 vs. x86_64 mismatch, or trying to load x86 on ARM hardware. In case you tried loading a non-binary file here (shell script?), you should make sure that the script specifies an interpreter in its shebang header line (`#!/...`) that is fully working (including its library dependencies). And before tackling scripts, better first test a simple non-script binary such as `/bin/sh` and confirm its successful execution. To find out more, add code `to init/main.c` to display kernel_execve()s return values.

> 5） **无法加载二进制文件**：请确保二进制文件的体系结构与您的硬件相匹配。例如，i386 与 x86_64 不匹配，或者试图在 ARM 硬件上加载 x86。如果您尝试在此处加载非二进制文件（shell 脚本？），则应确保该脚本在其 shebang 头行（`#！/…`）中指定了一个完全可用的解释器（包括其库依赖项）。在处理脚本之前，最好先测试一个简单的非脚本二进制文件，如“/bin/sh”，并确认其成功执行。要了解更多信息，请添加代码“到 init/main.c”以显示 kernel_execute()的返回值。

Please extend this explanation whenever you find new failure causes (after all loading the init binary is a CRITICAL and hard transition step which needs to be made as painless as possible), then submit a patch to LKML. Further TODOs:

> 每当您发现新的故障原因时，请扩展此解释（毕竟加载 init 二进制文件是一个关键且艰难的过渡步骤，需要尽可能轻松），然后向 LKML 提交补丁。其他 TODO:

- Implement the various `run_init_process()` invocations via a struct array which can then store the `kernel_execve()` result value and on failure log it all by iterating over **all** results (very important usability fix).

> - 通过结构数组实现各种“run_init_process()”调用，然后结构数组可以存储“kernel_execute()”结果值，并在失败时通过迭代**所有**结果来记录所有结果（非常重要的可用性修复）。

- Try to make the implementation itself more helpful in general, e.g. by providing additional error messages at affected places.

> - 一般来说，尽量使实现本身更有帮助，例如在受影响的地方提供额外的错误消息。
