---
tip: translate by baidu@2024-01-30 21:33:23
---
---
title: Dynamic debug
---

# Introduction


Dynamic debug allows you to dynamically enable/disable kernel debug-print code to obtain additional kernel information.

> 动态调试允许您动态启用/禁用内核调试打印代码，以获取额外的内核信息。


If `/proc/dynamic_debug/control` exists, your kernel has dynamic debug. You\'ll need root access (sudo su) to use this.

> 如果存在“/proc/dynamic/debug/control”，则内核具有动态调试功能。您需要root访问权限（sudo-su）才能使用此功能。

Dynamic debug provides:

> -   a Catalog of all *prdbgs* in your kernel. `cat /proc/dynamic_debug/control` to see them.
> -   a Simple query/command language to alter *prdbgs* by selecting on any combination of 0 or 1 of:
>     -   source filename
>     -   function name
>     -   line number (including ranges of line numbers)
>     -   module name
>     -   format string
>     -   class name (as known/declared by each module)

# Viewing Dynamic Debug Behaviour

You can view the currently configured behaviour in the *prdbg* catalog:

    :#> head -n7 /proc/dynamic_debug/control
    # filename:lineno [module]function flags format
    init/main.c:1179 [main]initcall_blacklist =_ "blacklisting initcall %s\012
    init/main.c:1218 [main]initcall_blacklisted =_ "initcall %s blacklisted\012"
    init/main.c:1424 [main]run_init_process =_ "  with arguments:\012"
    init/main.c:1426 [main]run_init_process =_ "    %s\012"
    init/main.c:1427 [main]run_init_process =_ "  with environment:\012"
    init/main.c:1429 [main]run_init_process =_ "    %s\012"


The 3rd space-delimited column shows the current flags, preceded by a `=` for easy use with grep/cut. `=p` shows enabled callsites.

> 第三列用空格分隔，显示当前标志，前面加一个“=”，便于与grep/cut一起使用`=p`显示启用的调用站点。

# Controlling dynamic debug Behaviour


The behaviour of *prdbg* sites are controlled by writing query/commands to the control file. Example:

> 通过向控制文件中写入查询/命令来控制*prdbg*站点的行为。实例

    # grease the interface
    :#> alias ddcmd='echo $* > /proc/dynamic_debug/control'

    :#> ddcmd '-p; module main func run* +p'
    :#> grep =p /proc/dynamic_debug/control
    init/main.c:1424 [main]run_init_process =p "  with arguments:\012"
    init/main.c:1426 [main]run_init_process =p "    %s\012"
    init/main.c:1427 [main]run_init_process =p "  with environment:\012"
    init/main.c:1429 [main]run_init_process =p "    %s\012"

Error messages go to console/syslog:

    :#> ddcmd mode foo +p
    dyndbg: unknown keyword "mode"
    dyndbg: query parse failed
    bash: echo: write error: Invalid argument


If debugfs is also enabled and mounted, `dynamic_debug/control` is also under the mount-dir, typically `/sys/kernel/debug/`.

> 如果debugfs也被启用和装载，那么“dynamic_debug/control”也在装载目录下，通常为“/sys/kernel/debug/”。

# Command Language Reference


At the basic lexical level, a command is a sequence of words separated by spaces or tabs. So these are all equivalent:

> 在基本的词汇层面上，命令是由空格或制表符分隔的单词序列。所以这些都是等价的：

    :#> ddcmd file svcsock.c line 1603 +p
    :#> ddcmd "file svcsock.c line 1603 +p"
    :#> ddcmd '  file   svcsock.c     line  1603 +p  '


Command submissions are bounded by a write() system call. Multiple commands can be written together, separated by `;` or `\n`:

> 命令提交受write（）系统调用的限制。多个命令可以写在一起，用`；`分隔或`\n`：

    :#> ddcmd "func pnpacpi_get_resources +p; func pnp_assign_mem +p"
    :#> ddcmd <<"EOC"
    func pnpacpi_get_resources +p
    func pnp_assign_mem +p
    EOC
    :#> cat query-batch-file > /proc/dynamic_debug/control


You can also use wildcards in each query term. The match rule supports `*` (matches zero or more characters) and `?` (matches exactly one character). For example, you can match all usb drivers:

> 您也可以在每个查询项中使用通配符。匹配规则支持“*”（匹配零个或多个字符）和“？”（正好匹配一个字符）。例如，您可以匹配所有usb驱动程序：

    :#> ddcmd file "drivers/usb/*" +p # "" to suppress shell expansion


Syntactically, a command is pairs of keyword values, followed by a flags change or setting:

> 从语法上讲，命令是一对关键字值，后面跟着标志更改或设置：

    command ::= match-spec* flags-spec


The match-spec\'s select *prdbgs* from the catalog, upon which to apply the flags-spec, all constraints are ANDed together. An absent keyword is the same as keyword \"\*\".

> 匹配规范从目录中选择*prdbgs*，在其上应用标志规范，所有约束都将进行AND运算。不存在的关键字与关键字“\*\”相同。


A match specification is a keyword, which selects the attribute of the callsite to be compared, and a value to compare against. Possible keywords are::

> 匹配规范是一个关键字，用于选择要比较的调用站点的属性和要进行比较的值。可能的关键字有：

    match-spec ::= 'func' string |
           'file' string |
           'module' string |
           'format' string |
           'class' string |
           'line' line-range

    line-range ::= lineno |
           '-'lineno |
           lineno'-' |
           lineno'-'lineno

    lineno ::= unsigned-int

::: note
::: title
Note
:::

`line-range` cannot contain space, e.g. \"1-30\" is valid range but \"1 - 30\" is not.
:::

The meanings of each keyword are:

func


:   The given string is compared against the function name of each callsite. Example:

> ：将给定字符串与每个调用站点的函数名进行比较。实例

    func svc_tcp_accept func *recv* \# in rfcomm, bluetooth, ping, tcp

file


:   The given string is compared against either the src-root relative pathname, or the basename of the source file of each callsite. Examples:

> ：将给定字符串与src根相对路径名或每个调用站点的源文件的基本名称进行比较。示例：

    file svcsock.c file kernel/freezer.c \# ie column 1 of control file file drivers/usb/\* \# all callsites under it file [inode.c:start]()\* \# parse :tail as a func (above) file inode.c:1-100 \# parse :tail as a line-range (above)

module


:   The given string is compared against the module name of each callsite. The module name is the string as seen in `lsmod`, i.e. without the directory or the `.ko` suffix and with `-` changed to `_`. Examples:

> ：将给定字符串与每个调用站点的模块名称进行比较。模块名称是“lsmod”中的字符串，即没有目录或“.ko”后缀，“-”改为“_”。示例：

    module sunrpc module nfsd module drm\* \# both drm, drm_kms_helper

format


:   The given string is searched for in the dynamic debug format string. Note that the string does not need to match the entire format, only some part. Whitespace and other special characters can be escaped using C octal character escape `\ooo` notation, e.g. the space character is `\040`. Alternatively, the string can be enclosed in double quote characters (`"`) or single quote characters (`'`). Examples:

> ：在动态调试格式字符串中搜索给定的字符串。请注意，字符串不需要与整个格式匹配，只需要与部分格式匹配。空格和其他特殊字符可以使用C八进制字符转义符“ooo”表示法进行转义，例如空格字符为“\040”。或者，字符串可以用双引号字符（`“`）或单引号字符（`'`）括起来。示例：

    format svcrdma: // many of the NFS/RDMA server pr_debugs format readahead // some pr_debugs in the readahead cache format nfsd:040SETATTR // one way to match a format with whitespace format \"nfsd: SETATTR\" // a neater way to match a format with whitespace format \'nfsd: SETATTR\' // yet another way to match a format with whitespace

class


:   The given class_name is validated against each module, which may have declared a list of known class_names. If the class_name is found for a module, callsite & class matching and adjustment proceeds. Examples:

> ：针对每个模块验证给定的类名称，这些模块可能已经声明了已知类名称的列表。如果找到模块的class_name，则调用站点和类匹配并进行调整。示例：

    class DRM_UT_KMS \# a DRM.debug category class JUNK \# silent non-match // class [TLD]()\* \# NOTICE: no wildcard in class names

line


:   The given line number or range of line numbers is compared against the line number of each `pr_debug()` callsite. A single line number matches the callsite line number exactly. A range of line numbers matches any callsite between the first and last line number inclusive. An empty first number means the first line in the file, an empty last line number means the last line number in the file. Examples:

> ：将给定的行号或行号范围与每个“pr_debug（）”调用站点的行号进行比较。单个行号与调用站点行号完全匹配。行号范围与第一个行号和最后一个行号（包括第一个行号）之间的任何调用站点匹配。空的第一个数字表示文件中的第一行，空的最后一个行号表示文件中最后一行。示例：

    line 1603 // exactly line 1603 line 1600-1605 // the six lines from line 1600 to line 1605 line -1605 // the 1605 lines from line 1 to line 1605 line 1600- // all lines from line 1600 to the end of the file


The flags specification comprises a change operation followed by one or more flag characters. The change operation is one of the characters:

> 标志规范包括后面跟着一个或多个标志字符的改变操作。更改操作是以下字符之一：

    -    remove the given flags
    +    add the given flags
    =    set the flags to the given flags

The flags are:

    p    enables the pr_debug() callsite.
    _    enables no flags.

    Decorator flags add to the message-prefix, in order:
    t    Include thread ID, or <intr>
    m    Include module name
    f    Include the function name
    s    Include the source file name
    l    Include line number


For `print_hex_dump_debug()` and `print_hex_dump_bytes()`, only the `p` flag has meaning, other flags are ignored.

> 对于“print_hex_dump_debug（）”和“print_hex_dump_bytes（）”，只有“p”标志有意义，其他标志被忽略。


Note the regexp `^[-+=][fslmpt_]+$` matches a flags specification. To clear all flags at once, use `=_` or `-fslmpt`.

> 请注意，正则表达式“^[-+=][fslmpt_]+$”与标志规范相匹配。要一次清除所有标志，请使用`=_`或`-fslmpt`。

# Debug messages during Boot Process


To activate debug messages for core code and built-in modules during the boot process, even before userspace and debugfs exists, use `dyndbg="QUERY"` or `module.dyndbg="QUERY"`. QUERY follows the syntax described above, but must not exceed 1023 characters. Your bootloader may impose lower limits.

> 要在引导过程中激活核心代码和内置模块的调试消息，甚至在用户空间和调试存在之前，请使用`dyndbg=“QUERY”`或`module.dyndbg=“QUERY”`。QUERY遵循上述语法，但不得超过1023个字符。您的引导程序可能会设置较低的限制。


These `dyndbg` params are processed just after the ddebug tables are processed, as part of the early_initcall. Thus you can enable debug messages in all code run after this early_initcall via this boot parameter.

> 这些“dyndbg”参数是在处理ddebug表之后处理的，作为early_init调用的一部分。因此，您可以通过这个引导参数在这个early_initcall之后运行的所有代码中启用调试消息。

On an x86 system for example ACPI enablement is a subsys_initcall and:

    dyndbg="file ec.c +p"


will show early Embedded Controller transactions during ACPI setup if your machine (typically a laptop) has an Embedded Controller. PCI (or other devices) initialization also is a hot candidate for using this boot parameter for debugging purposes.

> 如果您的机器（通常是笔记本电脑）具有嵌入式控制器，将在ACPI设置期间显示早期嵌入式控制器事务。PCI（或其他设备）初始化也是使用此引导参数进行调试的热门候选者。


If `foo` module is not built-in, `foo.dyndbg` will still be processed at boot time, without effect, but will be reprocessed when module is loaded later. Bare `dyndbg=` is only processed at boot.

> 如果“foo”模块不是内置的，则“foo.dyndbg”仍将在启动时进行处理，而不会产生任何效果，但将在稍后加载模块时进行重新处理。仅在启动时处理空的“dyndbg=”。

# Debug Messages at Module Initialization Time


When `modprobe foo` is called, modprobe scans `/proc/cmdline` for `foo.params`, strips `foo.`, and passes them to the kernel along with params given in modprobe args or `/etc/modprobe.d/*.conf` files, in the following order:

> 当调用“modprobe foo”时，modprobe扫描“/proc/cmdline”中的“foo.params”，剥离“foo.”，并将它们与modprobe args或“/etc/modprobe.d/*.conf”文件中给定的参数一起按以下顺序传递给内核：

1.  parameters given via `/etc/modprobe.d/*.conf`:

        options foo dyndbg=+pt
        options foo dyndbg # defaults to +p

2.  `foo.dyndbg` as given in boot args, `foo.` is stripped and passed:

        foo.dyndbg=" func bar +p; func buz +mp"

3.  args to modprobe:

        modprobe foo dyndbg==pmf # override previous settings


These `dyndbg` queries are applied in order, with last having final say. This allows boot args to override or modify those from `/etc/modprobe.d` (sensible, since 1 is system wide, 2 is kernel or boot specific), and modprobe args to override both.

> 这些“dyndbg”查询按顺序应用，最后说了算。这允许引导参数覆盖或修改“/etc/modprobe.d”中的那些（合理的，因为1是系统范围的，2是内核或引导特定的），并且modprobe参数覆盖两者。


In the `foo.dyndbg="QUERY"` form, the query must exclude `module foo`. `foo` is extracted from the param-name, and applied to each query in `QUERY`, and only 1 match-spec of each type is allowed.

> 在`foo.dyndbg=“QUERY”`形式中，查询必须排除`module foo``foo'是从参数名称中提取的，并应用于“query”中的每个查询，并且每个类型只允许有一个匹配规范。

The `dyndbg` option is a \"fake\" module parameter, which means:

-   modules do not need to define it explicitly
-   every module gets it tacitly, whether they use pr_debug or not

-   it doesn\'t appear in `/sys/module/$module/parameters/` To see it, grep the control file, or inspect `/proc/cmdline.`

> -它不会出现在“/sys/module/$module/parameters/`”中。要查看它，请grep控制文件，或检查“/proc/cmdline”`


For `CONFIG_DYNAMIC_DEBUG` kernels, any settings given at boot-time (or enabled by `-DDEBUG` flag during compilation) can be disabled later via the debugfs interface if the debug messages are no longer needed:

> 对于“CONFIG_DYNAMIC_DEBUG”内核，如果不再需要调试消息，则可以稍后通过debugfs接口禁用在启动时给定的任何设置（或在编译期间由“-DDEBUG”标志启用）：

    echo "module module_name -p" > /proc/dynamic_debug/control

# Examples

    // enable the message at line 1603 of file svcsock.c
    :#> ddcmd 'file svcsock.c line 1603 +p'

    // enable all the messages in file svcsock.c
    :#> ddcmd 'file svcsock.c +p'

    // enable all the messages in the NFS server module
    :#> ddcmd 'module nfsd +p'

    // enable all 12 messages in the function svc_process()
    :#> ddcmd 'func svc_process +p'

    // disable all 12 messages in the function svc_process()
    :#> ddcmd 'func svc_process -p'

    // enable messages for NFS calls READ, READLINK, READDIR and READDIR+.
    :#> ddcmd 'format "nfsd: READ" +p'

    // enable messages in files of which the paths include string "usb"
    :#> ddcmd 'file *usb* +p' > /proc/dynamic_debug/control

    // enable all messages
    :#> ddcmd '+p' > /proc/dynamic_debug/control

    // add module, function to all enabled messages
    :#> ddcmd '+mf' > /proc/dynamic_debug/control

    // boot-args example, with newlines and comments for readability
    Kernel command line: ...
      // see what's going on in dyndbg=value processing
      dynamic_debug.verbose=3
      // enable pr_debugs in the btrfs module (can be builtin or loadable)
      btrfs.dyndbg="+p"
      // enable pr_debugs in all files under init/
      // and the function parse_one, #cmt is stripped
      dyndbg="file init/* +p #cmt ; func parse_one +p"
      // enable pr_debugs in 2 functions in a module loaded later
      pc87360.dyndbg="func pc87360_init_device +p; func pc87360_find +p"

# Kernel Configuration

Dynamic Debug is enabled via kernel config items:

    CONFIG_DYNAMIC_DEBUG=y    # build catalog, enables CORE
    CONFIG_DYNAMIC_DEBUG_CORE=y   # enable mechanics only, skip catalog


If you do not want to enable dynamic debug globally (i.e. in some embedded system), you may set `CONFIG_DYNAMIC_DEBUG_CORE` as basic support of dynamic debug and add `ccflags := -DDYNAMIC_DEBUG_MODULE` into the Makefile of any modules which you\'d like to dynamically debug later.

> 如果您不想全局启用动态调试（即在某些嵌入式系统中），可以将“CONFIG_dynamic_debug_CORE”设置为动态调试的基本支持，并将“ccflags:=-DDYNAMIC_debug_MODULE”添加到以后要动态调试的任何模块的Makefile中。

# Kernel *prdbg* API


The following functions are cataloged and controllable when dynamic debug is enabled:

> 启用动态调试时，以下功能是可编目和控制的：

    pr_debug()
    dev_dbg()
    print_hex_dump_debug()
    print_hex_dump_bytes()


Otherwise, they are off by default; `ccflags += -DDEBUG` or `#define DEBUG` in a source file will enable them appropriately.

> 否则，默认情况下它们处于关闭状态`源文件中的ccflags+=-DDEBUG`或`#define DEBUG'将适当地启用它们。


If `CONFIG_DYNAMIC_DEBUG` is not set, `print_hex_dump_debug()` is just a shortcut for `print_hex_dump(KERN_DEBUG)`.

> 如果未设置“CONFIG_DYNAMIC_DEBUG”，则“print_hex_dump_DEBUG（）”只是“print_hex_dump（KERN_DEBUG）”的快捷方式。


For `print_hex_dump_debug()`/`print_hex_dump_bytes()`, format string is its `prefix_str` argument, if it is constant string; or `hexdump` in case `prefix_str` is built dynamically.

> 对于`print_hex_dump_debug（）`/`print_hex_dump_bytes（）`，如果格式字符串是常量字符串，则格式字符串是其`prefix_str`参数；或者在动态构建“prefix_str”的情况下为“hexdump”。
