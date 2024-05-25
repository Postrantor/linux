---
tip: translate by baidu@2024-01-30 21:40:09
---
---
title: Linux Magic System Request Key Hacks
---

Documentation for sysrq.c

# What is the magic SysRq key?


It is a \'magical\' key combo you can hit which the kernel will respond to regardless of whatever else it is doing, unless it is completely locked up.

> 这是一个“大”键组合，除非它被完全锁定，否则无论内核在做什么，它都会响应它。

# How do I enable the magic SysRq key?


You need to say \"yes\" to \'Magic SysRq key (CONFIG_MAGIC_SYSRQ)\' when configuring the kernel. When running a kernel with SysRq compiled in, /proc/sys/kernel/sysrq controls the functions allowed to be invoked via the SysRq key. The default value in this file is set by the CONFIG_MAGIC_SYSRQ_DEFAULT_ENABLE config symbol, which itself defaults to 1. Here is the list of possible values in /proc/sys/kernel/sysrq:

> 配置内核时，您需要对“Magic SysRq key（CONFIG_Magic_SysRq）”说“是”。当运行在中编译了SysRq的内核时，/proc/sys/kernel/SysRq控制允许通过SysRq键调用的函数。此文件中的默认值由CONFIG_MAGIC_SYSRQ_default_ENABLE配置符号设置，该符号本身默认为1。以下是/proc/sys/kernel/sysrq中可能的值列表：

> -   0 - disable sysrq completely
>
> -   1 - enable all functions of sysrq
>
> -   \>1 - bitmask of allowed sysrq functions (see below for detailed function description):
>
>         2 =   0x2 - enable control of console logging level
>         4 =   0x4 - enable control of keyboard (SAK, unraw)
>         8 =   0x8 - enable debugging dumps of processes etc.
>         16 =  0x10 - enable sync command
>         32 =  0x20 - enable remount read-only
>         64 =  0x40 - enable signalling of processes (term, kill, oom-kill)
>         128 =  0x80 - allow reboot/poweroff
>         256 = 0x100 - allow nicing of all RT tasks

You can set the value in the file by the following command:

    echo "number" >/proc/sys/kernel/sysrq


The number may be written here either as decimal or as hexadecimal with the 0x prefix. CONFIG_MAGIC_SYSRQ_DEFAULT_ENABLE must always be written in hexadecimal.

> 这里的数字可以写为十进制，也可以写为前缀为0x的十六进制。CONFIG_MAGIC_SYSRQ_DEFAULT_ENABLE必须始终以十六进制写入。


Note that the value of `/proc/sys/kernel/sysrq` influences only the invocation via a keyboard. Invocation of any operation via `/proc/sysrq-trigger` is always allowed (by a user with admin privileges).

> 请注意，“/proc/sys/kernel/sysrq”的值仅影响通过键盘进行的调用。始终允许（具有管理员权限的用户）通过“/proc/sysrq触发器”调用任何操作。

# How do I use the magic SysRq key?

On x86


:   You press the key combo `ALT-SysRq-<command key>`{.interpreted-text role="kbd"}.

> ：您按下组合键`ALT SysRq-<command key>`｛.explored text role=“kbd”｝。

    ::: note
    ::: title
    Note
    :::

    Some keyboards may not have a key labeled \'SysRq\'. The \'SysRq\' key is also known as the \'Print Screen\' key. Also some keyboards cannot handle so many keys being pressed at the same time, so you might have better luck with press `Alt`{.interpreted-text role="kbd"}, press `SysRq`{.interpreted-text role="kbd"}, release `SysRq`{.interpreted-text role="kbd"}, press `<command key>`{.interpreted-text role="kbd"}, release everything.
    :::

On SPARC


:   You press `ALT-STOP-<command key>`{.interpreted-text role="kbd"}, I believe.

> ：我相信您按下了`ALT-STOP-<command key>`｛.explored text role=“kbd”｝。

On the serial console (PC style standard serial ports only)


:   You send a `BREAK`, then within 5 seconds a command key. Sending `BREAK` twice is interpreted as a normal BREAK.

> ：您发送一个“BREAK”，然后在5秒内发送一个命令键。发送两次“BREAK”被解释为正常的BREAK。

On PowerPC

:   

    Press `ALT - Print Screen`{.interpreted-text role="kbd"} (or `F13`{.interpreted-text role="kbd"}) - `<command key>`{.interpreted-text role="kbd"}.

    :   `Print Screen`{.interpreted-text role="kbd"} (or `F13`{.interpreted-text role="kbd"}) - `<command key>`{.interpreted-text role="kbd"} may suffice.

On other


:   If you know of the key combos for other architectures, please submit a patch to be included in this section.

> ：如果你知道其他架构的关键组合，请提交一个补丁，包括在本节中。

On all

:   Write a character to /proc/sysrq-trigger. e.g.:

        echo t > /proc/sysrq-trigger

The `<command key>`{.interpreted-text role="kbd"} is case sensitive.

# What are the \'command\' keys?

+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Command | Function                                                                                                                                                                                                             |
+=========+======================================================================================================================================================================================================================+
| `b`     | Will immediately reboot the system without syncing or unmounting your disks.                                                                                                                                         |
+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| `c`     | Will perform a system crash and a crashdump will be taken if configured.                                                                                                                                             |
+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| `d`     | Shows all locks that are held.                                                                                                                                                                                       |
+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| `e`     | Send a SIGTERM to all processes, except for init.                                                                                                                                                                    |
+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| `f`     | Will call the oom killer to kill a memory hog process, but do not                                                                                                                                                    |
+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| > pani  | c if nothing can be killed.                                                                                                                                                                                          |
+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| `g`     | Used by kgdb (kernel debugger)                                                                                                                                                                                       |
+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| `h`     | Will display help (actually any other key than those listed here will display help. but `h` is easy to remember :-)                                                                                                  |
+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| `i`     | Send a SIGKILL to all processes, except for init.                                                                                                                                                                    |
+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| `j`     | Forcibly \"Just thaw it\" - filesystems frozen by the FIFREEZE ioctl.                                                                                                                                                |
+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| `k`     | Secure Access Key (SAK) Kills all programs on the current virtual console. NOTE: See important comments below in SAK section.                                                                                        |
+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| `l`     | Shows a stack backtrace for all active CPUs.                                                                                                                                                                         |
+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| `m`     | Will dump current memory info to your console.                                                                                                                                                                       |
+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| `n`     | Used to make RT tasks nice-able                                                                                                                                                                                      |
+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| `o`     | Will shut your system off (if configured and supported).                                                                                                                                                             |
+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| `p`     | Will dump the current registers and flags to your console.                                                                                                                                                           |
+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| `q`     | Will dump per CPU lists of all armed hrtimers (but NOT regular timer_list timers) and detailed information about all clockevent devices.                                                                             |
+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| `r`     | Turns off keyboard raw mode and sets it to XLATE.                                                                                                                                                                    |
+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| `s`     | Will attempt to sync all mounted filesystems.                                                                                                                                                                        |
+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| `t`     | Will dump a list of current tasks and their information to your console.                                                                                                                                             |
+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| `u`     | Will attempt to remount all mounted filesystems read-only.                                                                                                                                                           |
+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| `v`     | Forcefully restores framebuffer console                                                                                                                                                                              |
+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| `v`     | Causes ETM buffer dump \[ARM-specific\]                                                                                                                                                                              |
+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| `w`     | Dumps tasks that are in uninterruptible (blocked) state.                                                                                                                                                             |
+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| `x`     | Used by xmon interface on ppc/powerpc platforms. Show global PMU Registers on sparc64. Dump all TLB entries on MIPS.                                                                                                 |
+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| `y`     | Show global CPU Registers \[SPARC-64 specific\]                                                                                                                                                                      |
+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| `z`     | Dump the ftrace buffer                                                                                                                                                                                               |
+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| `0`-`9` | Sets the console log level, controlling which kernel messages will be printed to your console. (`0`, for example would make it so that only emergency messages like PANICs or OOPSes would make it to your console.) |
+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

# Okay, so what can I use them for?


Well, unraw(r) is very handy when your X server or a svgalib program crashes.

> 当您的X服务器或svgalib程序崩溃时，unraw（r）非常方便。


sak(k) (Secure Access Key) is useful when you want to be sure there is no trojan program running at console which could grab your password when you would try to login. It will kill all programs on given console, thus letting you make sure that the login prompt you see is actually the one from init, not some trojan program.

> sak（k）（安全访问密钥）在您希望确保控制台上没有运行木马程序时非常有用，当您尝试登录时，该程序可能会获取您的密码。它将杀死给定控制台上的所有程序，从而确保您看到的登录提示实际上是来自init的，而不是某个特洛伊木马程序。

::: important
::: title
Important
:::


In its true form it is not a true SAK like the one in a c2 compliant system, and it should not be mistaken as such.

> 在它的真实形式中，它不像c2兼容系统中的那样是一个真正的SAK，不应该被误认为是这样。
:::


It seems others find it useful as (System Attention Key) which is useful when you want to exit a program that will not let you switch consoles. (For example, X or a svgalib program.)

> 其他人似乎觉得它作为（系统注意键）很有用，当你想退出一个不允许你切换控制台的程序时，它很有用。（例如，X或svgalib程序。）

`reboot(b)` is good when you\'re unable to shut down, it is an equivalent of pressing the \"reset\" button.

`crash(c)` can be used to manually trigger a crashdump when the system is hung. Note that this just triggers a crash if there is no dump mechanism available.

`sync(s)` is handy before yanking removable medium or after using a rescue shell that provides no graceful shutdown \-- it will ensure your data is safely written to the disk. Note that the sync hasn\'t taken place until you see the \"OK\" and \"Done\" appear on the screen.

`umount(u)` can be used to mark filesystems as properly unmounted. From the running system\'s point of view, they will be remounted read-only. The remount isn\'t complete until you see the \"OK\" and \"Done\" message appear on the screen.


The loglevels `0`-`9` are useful when your console is being flooded with kernel messages you do not want to see. Selecting `0` will prevent all but the most urgent kernel messages from reaching your console. (They will still be logged if syslogd/klogd are alive, though.)

> 当控制台充斥着不想看到的内核消息时，日志级别“0”-“9”非常有用。选择“0”将阻止除最紧急的内核消息外的所有内核消息到达控制台。（不过，如果syslogd/klogd处于活动状态，它们仍将被记录。）

`term(e)` and `kill(i)` are useful if you have some sort of runaway process you are unable to kill any other way, especially if it\'s spawning other processes.


\"just thaw `it(j)`\" is useful if your system becomes unresponsive due to a frozen (probably root) filesystem via the FIFREEZE ioctl.

> \如果您的系统由于FIFREEZE ioctl冻结（可能是根）文件系统而变得没有响应，则“解冻`it（j）`\”非常有用。

# Sometimes SysRq seems to get \'stuck\' after using it, what can I do?


When this happens, try tapping shift, alt and control on both sides of the keyboard, and hitting an invalid sysrq sequence again. (i.e., something like `alt-sysrq-z`{.interpreted-text role="kbd"}).

> 当这种情况发生时，请尝试在键盘两侧敲击shift、alt和control，然后再次敲击无效的sysrq序列。（例如，类似于`alt-sysrq-z`{.解释文本角色=“kbd”}）。


Switching to another virtual console (`ALT+Fn`{.interpreted-text role="kbd"}) and then back again should also help.

> 切换到另一个虚拟控制台（`ALT+Fn`｛.explored text role=“kbd”｝），然后再切换回来也会有所帮助。

# I hit SysRq, but nothing seems to happen, what\'s wrong?


There are some keyboards that produce a different keycode for SysRq than the pre-defined value of 99 (see `KEY_SYSRQ` in `include/uapi/linux/input-event-codes.h`), or which don\'t have a SysRq key at all. In these cases, run `showkey -s` to find an appropriate scancode sequence, and use `setkeycodes <sequence> 99` to map this sequence to the usual SysRq code (e.g., `setkeycodes e05b 99`). It\'s probably best to put this command in a boot script. Oh, and by the way, you exit `showkey` by not typing anything for ten seconds.

> 有些键盘为SysRq生成的键代码与预定义值99不同（请参阅`include/uapi/linux/input event codes.h`中的`KEY_SysRq`），或者根本没有SysRq键。在这些情况下，运行“showkey-s”以找到适当的扫描代码序列，并使用“setkeycodes＜sequence＞99”将此序列映射到常用的SysRq代码（例如，“setkeycode e05b 99”）。最好把这个命令放在启动脚本中。哦，顺便说一句，你退出“showkey”时十秒钟内什么都不键入。

# I want to add SysRQ key events to a module, how does it work?


In order to register a basic function with the table, you must first include the header `include/linux/sysrq.h`, this will define everything else you need. Next, you must create a `sysrq_key_op` struct, and populate it with A) the key handler function you will use, B) a help_msg string, that will print when SysRQ prints help, and C) an action_msg string, that will print right before your handler is called. Your handler must conform to the prototype in \'sysrq.h\'.

> 为了在表中注册一个基本函数，您必须首先包含头`include/linux/sysrq.h `，这将定义您需要的所有其他内容。接下来，您必须创建一个“sysrq_key_op”结构，并用a）将要使用的密钥处理程序函数、B）将在sysrq打印帮助时打印的help_msg字符串和C）将在调用处理程序之前打印的action_msg串号填充它。您的处理程序必须符合“ysrq.h”中的原型。


After the `sysrq_key_op` is created, you can call the kernel function `register_sysrq_key(int key, const struct sysrq_key_op *op_p);` this will register the operation pointed to by `op_p` at table key \'key\', if that slot in the table is blank. At module unload time, you must call the function `unregister_sysrq_key(int key, const struct sysrq_key_op *op_p)`, which will remove the key op pointed to by \'op_p\' from the key \'key\', if and only if it is currently registered in that slot. This is in case the slot has been overwritten since you registered it.

> 创建“sysrq_key_op”后，可以调用内核函数“register_sysrq_key（int key，const struct sysrq_6key_op*op_p）；”如果表中的插槽为空，这将在表键“key”处注册“opp”指向的操作。在模块卸载时，必须调用函数“unregister_sysrq_key（int key，const struct sysrq_key_op*op_p）”，该函数将从密钥“key”中删除“p_p”指向的密钥op，如果且仅当该密钥当前已在该插槽中注册。这是为了防止插槽在您注册后被覆盖。


The Magic SysRQ system works by registering key operations against a key op lookup table, which is defined in \'drivers/tty/sysrq.c\'. This key table has a number of operations registered into it at compile time, but is mutable, and 2 functions are exported for interface to it:

> Magic SysRQ系统的工作原理是根据密钥操作查找表注册密钥操作，该表在\'drivers/tty/SysRQ.c\'中定义。该密钥表在编译时有许多操作注册到其中，但是可变的，并且导出了2个函数作为接口：

    register_sysrq_key and unregister_sysrq_key.


Of course, never ever leave an invalid pointer in the table. I.e., when your module that called register_sysrq_key() exits, it must call unregister_sysrq_key() to clean up the sysrq key table entry that it used. Null pointers in the table are always safe. :)

> 当然，永远不要在表中留下无效的指针。也就是说，当调用register_sysrq_key（）的模块退出时，它必须调用unregister_sysrx_key（）来清理它使用的sysrq密钥表条目。表中的空指针总是安全的。：）


If for some reason you feel the need to call the handle_sysrq function from within a function called by handle_sysrq, you must be aware that you are in a lock (you are also in an interrupt handler, which means don\'t sleep!), so you must call `__handle_sysrq_nolock` instead.

> 如果出于某种原因，您觉得需要从handle_sysrq调用的函数中调用handle_sysr q函数，则必须注意您处于锁定中（您也处于中断处理程序中，这意味着不要休眠！），因此必须改为调用`__handle_sysrx_nolock`。

# When I hit a SysRq key combination only the header appears on the console?


Sysrq output is subject to the same console loglevel control as all other console output. This means that if the kernel was booted \'quiet\' as is common on distro kernels the output may not appear on the actual console, even though it will appear in the dmesg buffer, and be accessible via the dmesg command and to the consumers of `/proc/kmsg`. As a specific exception the header line from the sysrq command is passed to all console consumers as if the current loglevel was maximum. If only the header is emitted it is almost certain that the kernel loglevel is too low. Should you require the output on the console channel then you will need to temporarily up the console loglevel using `alt-sysrq-8`{.interpreted-text role="kbd"} or:

> Sysrq输出与所有其他控制台输出受到相同的控制台日志级别控制。这意味着，如果内核像发行版内核上常见的那样“安静”启动，则输出可能不会出现在实际控制台上，即使它会出现在dmesg缓冲区中，并且可以通过dmesg命令和“/proc/kmsg”的消费者访问。作为一个特定的例外，sysrq命令的头行被传递给所有控制台使用者，就好像当前日志级别是最大的一样。如果只发出头，那么几乎可以肯定内核日志级别太低。如果您需要控制台通道上的输出，则需要使用`alt-sysrq-8`｛.depreted text role=“kbd”｝或临时提升控制台日志级别：

    echo 8 > /proc/sysrq-trigger


Remember to return the loglevel to normal after triggering the sysrq command you are interested in.

> 请记住，在触发您感兴趣的sysrq命令后，将日志级别恢复正常。

# I have more questions, who can I ask?

Just ask them on the linux-kernel mailing list:

:   <linux-kernel@vger.kernel.org>

# Credits

-   Written by Mydraal \<<vulpyne@vulpyne.net>\>
-   Updated by Adam Sulmicki \<<adam@cfar.umd.edu>\>
-   Updated by Jeremy M. Dolan \<<jmd@turbogeek.org>\> 2001/01/28 10:15:59
-   Added to by Crutcher Dunnavant \<<crutcher+kernel@datastacks.com>\>
