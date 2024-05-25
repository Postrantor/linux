---
tip: translate by baidu@2024-01-30 21:30:11
title: Bug hunting
---

Kernel bug reports often come with a stack dump like the one below:

```log
------------[ cut here ]------------
WARNING: CPU: 1 PID: 28102 at kernel/module.c:1108 module_put+0x57/0x70
Modules linked in: dvb_usb_gp8psk(-) dvb_usb dvb_core nvidia_drm(PO) nvidia_modeset(PO) snd_hda_codec_hdmi snd_hda_intel snd_hda_codec snd_hwdep snd_hda_core snd_pcm snd_timer snd soundcore nvidia(PO) [last unloaded: rc_core]
CPU: 1 PID: 28102 Comm: rmmod Tainted: P        WC O 4.8.4-build.1 #1
Hardware name: MSI MS-7309/MS-7309, BIOS V1.12 02/23/2009
    00000000 c12ba080 00000000 00000000 c103ed6a c1616014 00000001 00006dc6
    c1615862 00000454 c109e8a7 c109e8a7 00000009 ffffffff 00000000 f13f6a10
    f5f5a600 c103ee33 00000009 00000000 00000000 c109e8a7 f80ca4d0 c109f617
Call Trace:
    [<c12ba080>] ? dump_stack+0x44/0x64
    [<c103ed6a>] ? __warn+0xfa/0x120
    [<c109e8a7>] ? module_put+0x57/0x70
    [<c109e8a7>] ? module_put+0x57/0x70
    [<c103ee33>] ? warn_slowpath_null+0x23/0x30
    [<c109e8a7>] ? module_put+0x57/0x70
    [<f80ca4d0>] ? gp8psk_fe_set_frontend+0x460/0x460 [dvb_usb_gp8psk]
    [<c109f617>] ? symbol_put_addr+0x27/0x50
    [<f80bc9ca>] ? dvb_usb_adapter_frontend_exit+0x3a/0x70 [dvb_usb]
    [<f80bb3bf>] ? dvb_usb_exit+0x2f/0xd0 [dvb_usb]
    [<c13d03bc>] ? usb_disable_endpoint+0x7c/0xb0
    [<f80bb48a>] ? dvb_usb_device_exit+0x2a/0x50 [dvb_usb]
    [<c13d2882>] ? usb_unbind_interface+0x62/0x250
    [<c136b514>] ? __pm_runtime_idle+0x44/0x70
    [<c13620d8>] ? __device_release_driver+0x78/0x120
    [<c1362907>] ? driver_detach+0x87/0x90
    [<c1361c48>] ? bus_remove_driver+0x38/0x90
    [<c13d1c18>] ? usb_deregister+0x58/0xb0
    [<c109fbb0>] ? SyS_delete_module+0x130/0x1f0
    [<c1055654>] ? task_work_run+0x64/0x80
    [<c1000fa5>] ? exit_to_usermode_loop+0x85/0x90
    [<c10013f0>] ? do_fast_syscall_32+0x80/0x130
    [<c1549f43>] ? sysenter_past_esp+0x40/0x6a
---[ end trace 6ebc60ef3981792f ]---
```

Such stack traces provide enough information to identify the line inside the Kernel\'s source code where the bug happened. Depending on the severity of the issue, it may also contain the word **Oops**, as on this one:

> 这样的堆栈跟踪提供了足够的信息来识别内核源代码中发生错误的行。根据问题的严重程度，它还可能包含单词**Oops**，如本例所示：

```
BUG: unable to handle kernel NULL pointer dereference at   (null)
IP: [<c06969d4>] iret_exc+0x7d0/0xa59
*pdpt = 000000002258a001 *pde = 0000000000000000
Oops: 0002 [#1] PREEMPT SMP
...
```

Despite being an **Oops** or some other sort of stack trace, the offended line is usually required to identify and handle the bug. Along this chapter, we\'ll refer to \"Oops\" for all kinds of stack traces that need to be analyzed.

> 尽管是**Oops**或其他类型的堆栈跟踪，但通常需要被冒犯的行来识别和处理错误。在本章中，我们将参考“Oops”来了解需要分析的各种堆栈跟踪。

If the kernel is compiled with `CONFIG_DEBUG_INFO`, you can enhance the quality of the stack trace by using <file:%60scripts/decode_stacktrace.sh>\`.

> 如果使用“CONFIG_DEBUG_INFO”编译内核，则可以使用<file:%60scripts/decode_stacktrace.sh>\`来提高堆栈跟踪的质量。

# Modules linked in

Modules that are tainted or are being loaded or unloaded are marked with \"(\...)\", where the taint flags are described in <file:%60Documentation/admin-guide/tainted-kernels.rst>\`, \"being loaded\" is annotated with \"+\", and \"being unloaded\" is annotated with \"-\".

> 被污染或正在加载或卸载的模块用“（\…）\”标记，其中污染标志在<file:%60Documentation/admin-guide/spoted kernels.rst>\`中描述，“正在加载”用“+”注释，“正在卸载”用“-”注释。

# Where is the Oops message is located?

Normally the Oops text is read from the kernel buffers by klogd and handed to `syslogd` which writes it to a syslog file, typically `/var/log/messages` (depends on `/etc/syslog.conf`). On systems with systemd, it may also be stored by the `journald` daemon, and accessed by running `journalctl` command.

> 通常，Oops 文本由 klogd 从内核缓冲区读取，并交给“syslogd”，后者将其写入系统日志文件，通常为“/var/log/messages”（取决于“/etc/syslog.conf”）。在带有 systemd 的系统上，它也可以由“journal”守护进程存储，并通过运行“journalctl”命令进行访问。

Sometimes `klogd` dies, in which case you can run `dmesg > file` to read the data from the kernel buffers and save it. Or you can `cat /proc/kmsg > file`, however you have to break in to stop the transfer, since `kmsg` is a \"never ending file\".

> 有时“klogd”会失效，在这种情况下，您可以运行“dmesg>file”从内核缓冲区读取数据并保存它。或者，您可以使用“cat/proc/kmsg>file”，但您必须闯入以停止传输，因为“kmsg”是一个“永不终止的文件”。

If the machine has crashed so badly that you cannot enter commands or the disk is not available then you have three options:

> 如果机器严重崩溃，无法输入命令或磁盘不可用，则有三个选项：

(1) Hand copy the text from the screen and type it in after the machine has restarted. Messy but it is the only option if you have not planned for a crash. Alternatively, you can take a picture of the screen with a digital camera - not nice, but better than nothing. If the messages scroll off the top of the console, you may find that booting with a higher resolution (e.g., `vga=791`) will allow you to read more of the text. (Caveat: This needs `vesafb`, so won\'t help for \'early\' oopses.)

> （1） 手动复制屏幕上的文本，并在机器重新启动后键入。混乱，但如果你没有计划撞车，这是唯一的选择。或者，你可以用数码相机拍一张屏幕的照片——这并不好，但总比什么都没有好。如果消息从控制台顶部滚动，您可能会发现以更高的分辨率（例如“vga=791”）引导将允许您阅读更多文本。（注意：这需要“vesafb”，所以对“早期”错误没有帮助。）

(2) Boot with a serial console (see `Documentation/admin-guide/serial-console.rst <serial_console>`{.interpreted-text role="ref"}), run a null modem to a second machine and capture the output there using your favourite communication program. Minicom works well.

> （2） 使用串行控制台启动（请参阅`Documentation/admin guide/serial-console.rst<serial_console>`{.depreced text role=“ref”}），将空调制解调器运行到第二台机器，并使用您喜爱的通信程序在那里捕获输出。Minicom 运行良好。

(3) Use Kdump (see Documentation/admin-guide/kdump/kdump.rst), extract the kernel ring buffer from old memory with using dmesg gdbmacro in Documentation/admin-guide/kdump/gdbmacros.txt.

> （3） 使用 Kdump（请参阅 Documentation/admin guide/Kdump/Kdump.rst），使用 Documentation/admin-guide/Kdump/gdbmacros.txt 中的 dmesg gdbmacro 从旧内存中提取内核环形缓冲区。

# Finding the bug\'s location

Reporting a bug works best if you point the location of the bug at the Kernel source file. There are two methods for doing that. Usually, using `gdb` is easier, but the Kernel should be pre-compiled with debug info.

> 如果将错误的位置指向内核源文件，则报告错误的效果最好。有两种方法可以做到这一点。通常，使用“gdb”更容易，但内核应该使用调试信息进行预编译。

## gdb

The GNU debugger (`gdb`) is the best way to figure out the exact file and line number of the OOPS from the `vmlinux` file.

> GNU 调试器（“gdb”）是从“vmlinux”文件中找出 OOPS 的确切文件和行号的最佳方法。

The usage of gdb works best on a kernel compiled with `CONFIG_DEBUG_INFO`. This can be set by running:

> gdb 的使用在使用“CONFIG_DEBUG_INFO”编译的内核上效果最好。这可以通过运行来设置：

```
    $ ./scripts/config -d COMPILE_TEST -e DEBUG_KERNEL -e DEBUG_INFO
```

On a kernel compiled with `CONFIG_DEBUG_INFO`, you can simply copy the EIP value from the OOPS:

> 在使用“CONFIG_DEBUG_INFO”编译的内核上，您可以简单地从 OOPS 复制 EIP 值：

```
    EIP:    0060:[<c021e50e>]    Not tainted VLI
```

And use GDB to translate that to human-readable form:

```
    $ gdb vmlinux
    (gdb) l *0xc021e50e
```

If you don\'t have `CONFIG_DEBUG_INFO` enabled, you use the function offset from the OOPS:

> 如果没有启用“CONFIG_DEBUG_INFO”，则使用 OOPS 的函数偏移量：

```
    EIP is at vt_ioctl+0xda8/0x1482
```

And recompile the kernel with `CONFIG_DEBUG_INFO` enabled:

```
    $ ./scripts/config -d COMPILE_TEST -e DEBUG_KERNEL -e DEBUG_INFO
    $ make vmlinux
    $ gdb vmlinux
    (gdb) l *vt_ioctl+0xda8
    0x1888 is in vt_ioctl (drivers/tty/vt/vt_ioctl.c:293).
    288   {
    289       struct vc_data *vc = NULL;
    290       int ret = 0;
    291
    292       console_lock();
    293       if (VT_BUSY(vc_num))
    294           ret = -EBUSY;
    295       else if (vc_num)
    296           vc = vc_deallocate(vc_num);
    297       console_unlock();
```

or, if you want to be more verbose:

```
    (gdb) p vt_ioctl
    $1 = {int (struct tty_struct *, unsigned int, unsigned long)} 0xae0 <vt_ioctl>
    (gdb) l *0xae0+0xda8
```

You could, instead, use the object file:

```
    $ make drivers/tty/
    $ gdb drivers/tty/vt/vt_ioctl.o
    (gdb) l *vt_ioctl+0xda8
```

If you have a call trace, such as:

```
    Call Trace:
     [<ffffffff8802c8e9>] :jbd:log_wait_commit+0xa3/0xf5
     [<ffffffff810482d9>] autoremove_wake_function+0x0/0x2e
     [<ffffffff8802770b>] :jbd:journal_stop+0x1be/0x1ee
     ...
```

this shows the problem likely is in the :jbd: module. You can load that module in gdb and list the relevant code:

> 这表明问题可能出现在：jbd:模块中。您可以在 gdb 中加载该模块，并列出相关代码：

    $ gdb fs/jbd/jbd.ko
    (gdb) l *log_wait_commit+0xa3

::: note
::: title
Note
:::

You can also do the same for any function call at the stack trace, like this one:

> 您也可以对堆栈跟踪中的任何函数调用执行同样的操作，如下所示：

    [<f80bc9ca>] ? dvb_usb_adapter_frontend_exit+0x3a/0x70 [dvb_usb]

The position where the above call happened can be seen with:

    $ gdb drivers/media/usb/dvb-usb/dvb-usb.o

(gdb) l \*dvb_usb_adapter_frontend_exit+0x3a
:::

## objdump

To debug a kernel, use objdump and look for the hex offset from the crash output to find the valid line of code/assembler. Without debug symbols, you will see the assembler code for the routine shown, but if your kernel has debug symbols the C code will also be available. (Debug symbols can be enabled in the kernel hacking menu of the menu configuration.) For example:

> 要调试内核，请使用 objdump 并查找崩溃输出的十六进制偏移量，以找到有效的代码行/汇编程序。如果没有调试符号，您将看到所示例程的汇编代码，但如果您的内核有调试符号，C 代码也将可用。（调试符号可以在菜单配置的内核破解菜单中启用。）例如：

    $ objdump -r -S -l --disassemble net/dccp/ipv4.o

::: note
::: title
Note
:::

You need to be at the top level of the kernel tree for this to pick up your C files.

> 您需要处于内核树的顶层，才能获取 C 文件。
> :::

If you don\'t have access to the source code you can still debug some crash dumps using the following method (example crash dump output as shown by Dave Miller):

> 如果您没有访问源代码的权限，您仍然可以使用以下方法调试一些崩溃转储（例如 Dave Miller 所示的崩溃转储输出）：

    EIP is at  +0x14/0x4c0
     ...
    Code: 44 24 04 e8 6f 05 00 00 e9 e8 fe ff ff 8d 76 00 8d bc 27 00 00
    00 00 55 57  56 53 81 ec bc 00 00 00 8b ac 24 d0 00 00 00 8b 5d 08
    <8b> 83 3c 01 00 00 89 44  24 14 8b 45 28 85 c0 89 44 24 18 0f 85

    Put the bytes into a "foo.s" file like this:

           .text
           .globl foo
    foo:
           .byte  .... /* bytes from Code: part of OOPS dump */

    Compile it with "gcc -c -o foo.o foo.s" then look at the output of
    "objdump --disassemble foo.o".

    Output:

    ip_queue_xmit:
        push       %ebp
        push       %edi
        push       %esi
        push       %ebx
        sub        $0xbc, %esp
        mov        0xd0(%esp), %ebp        ! %ebp = arg0 (skb)
        mov        0x8(%ebp), %ebx         ! %ebx = skb->sk
        mov        0x13c(%ebx), %eax       ! %eax = inet_sk(sk)->opt

<file:%60scripts/decodecode>\` can be used to automate most of this, depending on what CPU architecture is being debugged.

# Reporting the bug

Once you find where the bug happened, by inspecting its location, you could either try to fix it yourself or report it upstream.

> 一旦你找到错误发生的地方，通过检查它的位置，你可以尝试自己修复它，也可以向上游报告。

In order to report it upstream, you should identify the mailing list used for the development of the affected code. This can be done by using the `get_maintainer.pl` script.

> 为了向上游报告，您应该确定用于开发受影响代码的邮件列表。这可以通过使用“get_maintainer.pl”脚本来完成。

For example, if you find a bug at the gspca\'s sonixj.c file, you can get its maintainers with:

> 例如，如果您在 gspca\的 sonixj.c 文件中发现一个 bug，您可以通过以下方式获得它的维护者：

    $ ./scripts/get_maintainer.pl -f drivers/media/usb/gspca/sonixj.c
    Hans Verkuil <hverkuil@xs4all.nl> (odd fixer:GSPCA USB WEBCAM DRIVER,commit_signer:1/1=100%)
    Mauro Carvalho Chehab <mchehab@kernel.org> (maintainer:MEDIA INPUT INFRASTRUCTURE (V4L/DVB),commit_signer:1/1=100%)
    Tejun Heo <tj@kernel.org> (commit_signer:1/1=100%)
    Bhaktipriya Shridhar <bhaktipriya96@gmail.com> (commit_signer:1/1=100%,authored:1/1=100%,added_lines:4/4=100%,removed_lines:9/9=100%)
    linux-media@vger.kernel.org (open list:GSPCA USB WEBCAM DRIVER)
    linux-kernel@vger.kernel.org (open list)

Please notice that it will point to:

- The last developers that touched the source code (if this is done inside a git tree). On the above example, Tejun and Bhaktipriya (in this specific case, none really involved on the development of this file);

> -最后一个接触源代码的开发人员（如果这是在 git 树中完成的）。在上面的例子中，Tejun 和 Bhaktipriya（在这个特定的案例中，没有人真正参与这个文件的开发）；

- The driver maintainer (Hans Verkuil);
- The subsystem maintainer (Mauro Carvalho Chehab);
- The driver and/or subsystem mailing list (<linux-media@vger.kernel.org>);
- the Linux Kernel mailing list (<linux-kernel@vger.kernel.org>).

Usually, the fastest way to have your bug fixed is to report it to mailing list used for the development of the code (linux-media ML) copying the driver maintainer (Hans).

> 通常，修复错误的最快方法是将其报告给用于代码开发的邮件列表（linux media ML），复制驱动程序维护者（Hans）。

If you are totally stumped as to whom to send the report, and `get_maintainer.pl` didn\'t provide you anything useful, send it to <linux-kernel@vger.kernel.org>.

> 如果您完全不知道该向谁发送报告，而“get_maintainer.pl”没有为您提供任何有用的信息，请将其发送到<linux-kernel@vger.kernel.org>.

Thanks for your help in making Linux as stable as humanly possible.

# Fixing the bug

If you know programming, you could help us by not only reporting the bug, but also providing us with a solution. After all, open source is about sharing what you do and don\'t you want to be recognised for your genius?

> 如果你知道编程，你不仅可以报告错误，还可以为我们提供解决方案，从而帮助我们。毕竟，开源就是分享你所做的事情，你不想因为你的天才而被认可吗？

If you decide to take this way, once you have worked out a fix please submit it upstream.

> 如果你决定采取这种方式，一旦你制定了解决方案，请向上游提交。

Please do read `Documentation/process/submitting-patches.rst <submittingpatches>`{.interpreted-text role="ref"} though to help your code get accepted.

> 请阅读`Documentation/process/submitting-patches.rst＜submittingpatches＞`{.depredicted text role=“ref”}，以帮助您的代码被接受。

---

# Notes on Oops tracing with `klogd`

In order to help Linus and the other kernel developers there has been substantial support incorporated into `klogd` for processing protection faults. In order to have full support for address resolution at least version 1.3-pl3 of the `sysklogd` package should be used.

> 为了帮助 Linus 和其他内核开发人员，在“klogd”中加入了大量的处理保护故障的支持。为了完全支持地址解析，至少应使用“sysklogd”包的 1.3-pl3 版本。

When a protection fault occurs the `klogd` daemon automatically translates important addresses in the kernel log messages to their symbolic equivalents. This translated kernel message is then forwarded through whatever reporting mechanism `klogd` is using. The protection fault message can be simply cut out of the message files and forwarded to the kernel developers.

> 当发生保护故障时，“klogd”守护进程会自动将内核日志消息中的重要地址转换为其符号等效地址。然后，通过“klogd”正在使用的任何报告机制来转发这个翻译后的内核消息。保护故障消息可以简单地从消息文件中截取并转发给内核开发人员。

Two types of address resolution are performed by `klogd`. The first is static translation and the second is dynamic translation. Static translation uses the System.map file. In order to do static translation the `klogd` daemon must be able to find a system map file at daemon initialization time. See the klogd man page for information on how `klogd` searches for map files.

> “klogd”执行两种类型的地址解析。第一种是静态翻译，第二种是动态翻译。静态转换使用 System.map 文件。为了进行静态转换，“klogd”守护程序必须能够在守护程序初始化时找到系统映射文件。有关“klogd”如何搜索地图文件的信息，请参阅 klogd 手册页。

Dynamic address translation is important when kernel loadable modules are being used. Since memory for kernel modules is allocated from the kernel\'s dynamic memory pools there are no fixed locations for either the start of the module or for functions and symbols in the module.

> 当使用内核可加载模块时，动态地址转换非常重要。由于内核模块的内存是从内核的动态内存池中分配的，因此模块的开始或模块中的函数和符号都没有固定的位置。

The kernel supports system calls which allow a program to determine which modules are loaded and their location in memory. Using these system calls the klogd daemon builds a symbol table which can be used to debug a protection fault which occurs in a loadable kernel module.

> 内核支持系统调用，允许程序确定加载了哪些模块及其在内存中的位置。使用这些系统调用，klogd 守护进程构建一个符号表，该表可用于调试可加载内核模块中发生的保护故障。

At the very minimum klogd will provide the name of the module which generated the protection fault. There may be additional symbolic information available if the developer of the loadable module chose to export symbol information from the module.

> klogd 将至少提供产生保护故障的模块的名称。如果可加载模块的开发人员选择从模块导出符号信息，则可能存在可用的附加符号信息。

Since the kernel module environment can be dynamic there must be a mechanism for notifying the `klogd` daemon when a change in module environment occurs. There are command line options available which allow klogd to signal the currently executing daemon that symbol information should be refreshed. See the `klogd` manual page for more information.

> 由于内核模块环境可以是动态的，因此当模块环境发生变化时，必须有一种机制来通知“klogd”守护进程。有可用的命令行选项，允许 klogd 向当前执行的守护进程发出信号，表示应该刷新符号信息。有关详细信息，请参阅“klogd”手册页。

A patch is included with the sysklogd distribution which modifies the `modules-2.0.0` package to automatically signal klogd whenever a module is loaded or unloaded. Applying this patch provides essentially seamless support for debugging protection faults which occur with kernel loadable modules.

> sysklogd 发行版中包含一个补丁，该补丁修改“modules-2.0.0”包，以便在加载或卸载模块时自动向 klogd 发出信号。应用此补丁为调试内核可加载模块中发生的保护故障提供了基本上无缝的支持。

The following is an example of a protection fault in a loadable module processed by `klogd`:

> 以下是由“klogd”处理的可加载模块中的保护故障示例：

    Aug 29 09:51:01 blizard kernel: Unable to handle kernel paging request at virtual address f15e97cc
    Aug 29 09:51:01 blizard kernel: current->tss.cr3 = 0062d000, %cr3 = 0062d000
    Aug 29 09:51:01 blizard kernel: *pde = 00000000
    Aug 29 09:51:01 blizard kernel: Oops: 0002
    Aug 29 09:51:01 blizard kernel: CPU:    0
    Aug 29 09:51:01 blizard kernel: EIP:    0010:[oops:_oops+16/3868]
    Aug 29 09:51:01 blizard kernel: EFLAGS: 00010212
    Aug 29 09:51:01 blizard kernel: eax: 315e97cc   ebx: 003a6f80   ecx: 001be77b   edx: 00237c0c
    Aug 29 09:51:01 blizard kernel: esi: 00000000   edi: bffffdb3   ebp: 00589f90   esp: 00589f8c
    Aug 29 09:51:01 blizard kernel: ds: 0018   es: 0018   fs: 002b   gs: 002b   ss: 0018
    Aug 29 09:51:01 blizard kernel: Process oops_test (pid: 3374, process nr: 21, stackpage=00589000)
    Aug 29 09:51:01 blizard kernel: Stack: 315e97cc 00589f98 0100b0b4 bffffed4 0012e38e 00240c64 003a6f80 00000001
    Aug 29 09:51:01 blizard kernel:        00000000 00237810 bfffff00 0010a7fa 00000003 00000001 00000000 bfffff00
    Aug 29 09:51:01 blizard kernel:        bffffdb3 bffffed4 ffffffda 0000002b 0007002b 0000002b 0000002b 00000036
    Aug 29 09:51:01 blizard kernel: Call Trace: [oops:_oops_ioctl+48/80] [_sys_ioctl+254/272] [_system_call+82/128]
    Aug 29 09:51:01 blizard kernel: Code: c7 00 05 00 00 00 eb 08 90 90 90 90 90 90 90 90 89 ec 5d c3

---

    Dr. G.W. Wettstein           Oncology Research Div. Computing Facility
    Roger Maris Cancer Center    INTERNET: greg@wind.rmcc.com
    820 4th St. N.
    Fargo, ND  58122
    Phone: 701-234-7556
