---
tip: translate by baidu@2024-01-30 21:29:51
author:
  - Masami Hiramatsu \<<mhiramat@kernel.org>\>
title: Boot Configuration
---

# Overview

The boot configuration expands the current kernel command line to support additional key-value data when booting the kernel in an efficient way. This allows administrators to pass a structured-Key config file.

> 引导配置扩展了当前内核命令行，以便在以有效方式引导内核时支持额外的键值数据。这允许管理员传递结构化的密钥配置文件。

# Config File Syntax

The boot config syntax is a simple structured key-value. Each key consists of dot-connected-words, and key and value are connected by `=`. The value has to be terminated by semi-colon (`;`) or newline (`\n`). For array value, array entries are separated by comma (`,`). :

> 引导配置语法是一个简单的结构化键值。每个键由点连接的单词组成，键和值由“=”连接。该值必须以分号（`；`）或换行符（`\n`）结尾。对于数组值，数组条目用逗号（`，`）分隔。：

```
KEY[.WORD[...]] = VALUE[, VALUE2[...]][;]
```

Unlike the kernel command line syntax, spaces are OK around the comma and `=`.

> 与内核命令行语法不同，逗号和“=”周围可以有空格。

Each key word must contain only alphabets, numbers, dash (`-`) or underscore (`_`). And each value only contains printable characters or spaces except for delimiters such as semi-colon (`;`), new-line (`\n`), comma (`,`), hash (`#`) and closing brace (`}`).

> 每个关键字只能包含字母、数字、短划线（`-`）或下划线（`_`）。每个值只包含可打印的字符或空格，但分隔符除外，如分号（`；`）、换行符（`\n`）、逗号（`，`）、散列（`#`）和大括号（`}`）。

If you want to use those delimiters in a value, you can use either double-quotes (`"VALUE"`) or single-quotes (`'VALUE'`) to quote it. Note that you can not escape these quotes.

> 如果要在值中使用这些分隔符，可以使用双引号（`“value”`）或单引号（`'value'`）对其进行引号引用。请注意，不能转义这些引号。

There can be a key which doesn\'t have value or has an empty value. Those keys are used for checking if the key exists or not (like a boolean).

> 可能有一个键没有值或值为空。这些键用于检查键是否存在（就像布尔值一样）。

## Key-Value Syntax

The boot config file syntax allows user to merge partially same word keys by brace. For example:

> 引导配置文件语法允许用户通过大括号合并部分相同的单词键。例如

```
foo.bar.baz = value1
foo.bar.qux.quux = value2
```

These can be written also in:

```
foo.bar {
    baz = value1
    qux.quux = value2
}
```

Or more shorter, written as following:

```
foo.bar { baz = value1; qux.quux = value2 }
```

In both styles, same key words are automatically merged when parsing it at boot time. So you can append similar trees or key-values.

> 在这两种风格中，相同的关键字在启动时解析时会自动合并。因此，您可以附加类似的树或键值。

## Same-key Values

It is prohibited that two or more values or arrays share a same-key. For example,:

> 禁止两个或多个值或数组共享同一密钥。例如

```
foo = bar, baz
foo = qux  # !ERROR! we can not re-define same key
```

If you want to update the value, you must use the override operator `:=` explicitly. For example:

> 如果要更新值，则必须显式使用重写运算符“：=”。例如

```
foo = bar, baz
foo := qux
```

then, the `qux` is assigned to `foo` key. This is useful for overriding the default value by adding (partial) custom bootconfigs without parsing the default bootconfig.

> 然后，将“qux”分配给“foo”键。这对于在不解析默认 bootconfig 的情况下添加（部分）自定义 bootconfig 来覆盖默认值非常有用。

If you want to append the value to existing key as an array member, you can use `+=` operator. For example:

> 如果要将值作为数组成员附加到现有键，可以使用“+=”运算符。例如

```
foo = bar, baz
foo += qux
```

In this case, the key `foo` has `bar`, `baz` and `qux`.

Moreover, sub-keys and a value can coexist under a parent key. For example, following config is allowed.:

> 此外，子关键字和值可以在父关键字下共存。例如，允许以下配置。：

```
foo = value1
foo.bar = value2
foo := value3 # This will update foo's value.
```

Note, since there is no syntax to put a raw value directly under a structured key, you have to define it outside of the brace. For example:

> 请注意，由于没有语法可以将原始值直接放在结构化键下，因此必须在大括号之外进行定义。例如

```
foo {
    bar = value1
    bar {
        baz = value2
        qux = value3
    }
}
```

Also, the order of the value node under a key is fixed. If there are a value and subkeys, the value is always the first child node of the key. Thus if user specifies subkeys first, e.g.:

> 此外，键下的值节点的顺序是固定的。如果有一个值和子键，则该值始终是该键的第一个子节点。因此，如果用户首先指定子密钥，例如：

```
foo.bar = value1
foo = value2
```

In the program (and /proc/bootconfig), it will be shown as below:

```
foo = value2
foo.bar = value1
```

## Comments

The config syntax accepts shell-script style comments. The comments starting with hash (\"#\") until newline (\"n\") will be ignored.

> config 语法接受 shell 脚本样式的注释。从 hash（\“#\”）开始直到换行符（\“n\”）的注释将被忽略。

```
# comment line
foo = value # value is set to foo.
bar = 1, # 1st element
        2, # 2nd element
        3  # 3rd element
```

This is parsed as below:

```
foo = value
bar = 1, 2, 3
```

Note that you can not put a comment between value and delimiter(`,` or `;`). This means following config has a syntax error :

> 请注意，不能在值和分隔符（`，`或`；`）之间放置注释。这意味着以下配置存在语法错误：

```
key = 1 # comment
        ,2
```

# /proc/bootconfig

/proc/bootconfig is a user-space interface of the boot config. Unlike /proc/cmdline, this file shows the key-value style list. Each key-value pair is shown in each line with following style:

> /proc/bootconfig 是引导配置的用户空间接口。与/proc/cmdline 不同，此文件显示键值样式列表。每一行中都显示了每个键值对，样式如下：

```
KEY[.WORDS...] = "[VALUE]"[,"VALUE2"...]
```

# Boot Kernel With a Boot Config

There are two options to boot the kernel with bootconfig: attaching the bootconfig to the initrd image or embedding it in the kernel itself.

> 使用 bootconfig 引导内核有两种选择：将 bootconfig 附加到 initrd 映像或将其嵌入内核本身。

## Attaching a Boot Config to Initrd

Since the boot configuration file is loaded with initrd by default, it will be added to the end of the initrd (initramfs) image file with padding, size, checksum and 12-byte magic word as below.

> 由于默认情况下引导配置文件是用 initrd 加载的，因此它将被添加到 initrd（initramfs）映像文件的末尾，并带有填充、大小、校验和和 12 字节的魔法字，如下所示。

```
[initrd][bootconfig][padding][size(le32)][checksum(le32)][#BOOTCONFIGn]
```

The size and checksum fields are unsigned 32bit little endian value.

When the boot configuration is added to the initrd image, the total file size is aligned to 4 bytes. To fill the gap, null characters (`\0`) will be added. Thus the `size` is the length of the bootconfig file + padding bytes.

> 当引导配置被添加到 initrd 映像时，总文件大小被调整为 4 个字节。为了填补空白，将添加空字符（“\0”）。因此，“大小”是 bootconfig 文件的长度+填充字节。

The Linux kernel decodes the last part of the initrd image in memory to get the boot configuration data. Because of this \"piggyback\" method, there is no need to change or update the boot loader and the kernel image itself as long as the boot loader passes the correct initrd file size. If by any chance, the boot loader passes a longer size, the kernel fails to find the bootconfig data.

> Linux 内核对内存中 initrd 映像的最后一部分进行解码，以获得引导配置数据。由于这种“piggyback”方法，只要引导加载程序通过正确的 initrd 文件大小，就不需要更改或更新引导加载程序和内核映像本身。如果引导加载程序通过了更长的大小，内核将无法找到 bootconfig 数据。

To do this operation, Linux kernel provides `bootconfig` command under tools/bootconfig, which allows admin to apply or delete the config file to/from initrd image. You can build it by the following command:

> 为了执行此操作，Linux 内核在 tools/bootconfig 下提供了“bootconfig”命令，允许管理员将配置文件应用到 initrd 映像或从 initrd 映像删除配置文件。您可以通过以下命令构建它：

```
# make -C tools/bootconfig
```

To add your boot config file to initrd image, run bootconfig as below (Old data is removed automatically if exists):

> 要将引导配置文件添加到 initrd 映像，请按如下方式运行引导配置（如果存在旧数据，则会自动删除）：

```
# tools/bootconfig/bootconfig -a your-config /boot/initrd.img-X.Y.Z
```

To remove the config from the image, you can use -d option as below:

```
# tools/bootconfig/bootconfig -d /boot/initrd.img-X.Y.Z
```

Then add \"bootconfig\" on the normal kernel command line to tell the kernel to look for the bootconfig at the end of the initrd file. Alternatively, build your kernel with the `CONFIG_BOOT_CONFIG_FORCE` Kconfig option selected.

> 然后在普通内核命令行上添加“bootconfig\”，告诉内核在 initrd 文件的末尾查找 bootconfig。或者，使用选定的“CONFIG_BOOT_CONFIG_FORCE”Kconfig 选项构建内核。

## Embedding a Boot Config into Kernel

If you can not use initrd, you can also embed the bootconfig file in the kernel by Kconfig options. In this case, you need to recompile the kernel with the following configs:

> 如果不能使用 initrd，也可以通过 Kconfig 选项将 bootconfig 文件嵌入到内核中。在这种情况下，您需要使用以下配置重新编译内核：

```
CONFIG_BOOT_CONFIG_EMBED=y
CONFIG_BOOT_CONFIG_EMBED_FILE="/PATH/TO/BOOTCONFIG/FILE"
```

`CONFIG_BOOT_CONFIG_EMBED_FILE` requires an absolute path or a relative path to the bootconfig file from source tree or object tree. The kernel will embed it as the default bootconfig.

Just as when attaching the bootconfig to the initrd, you need `bootconfig` option on the kernel command line to enable the embedded bootconfig, or, alternatively, build your kernel with the `CONFIG_BOOT_CONFIG_FORCE` Kconfig option selected.

> 就像将 bootconfig 附加到 initrd 时一样，您需要内核命令行上的“bootconfig”选项来启用嵌入式 bootconfig，或者，使用选定的“CONFIG_BOOT_CONFIG_FORCE”Kconfig 选项构建内核。

Note that even if you set this option, you can override the embedded bootconfig by another bootconfig which attached to the initrd.

> 请注意，即使设置了此选项，也可以通过附加到 initrd 的另一个 bootconfig 覆盖嵌入的 bootconfig。

# Kernel parameters via Boot Config

In addition to the kernel command line, the boot config can be used for passing the kernel parameters. All the key-value pairs under `kernel` key will be passed to kernel cmdline directly. Moreover, the key-value pairs under `init` will be passed to init process via the cmdline. The parameters are concatenated with user-given kernel cmdline string as the following order, so that the command line parameter can override bootconfig parameters (this depends on how the subsystem handles parameters but in general, earlier parameter will be overwritten by later one.):

> 除了内核命令行之外，引导配置还可以用于传递内核参数。“kernel”键下的所有键值对将直接传递给内核 cmdline。此外，“init”下的键值对将通过 cmdline 传递给 init 进程。这些参数按以下顺序与用户给定的内核 cmdline 字符串连接，以便命令行参数可以覆盖 bootconfig 参数（这取决于子系统如何处理参数，但通常情况下，较早的参数将被较晚的参数覆盖。）：

```
[bootconfig params][cmdline params] -- [bootconfig init params][cmdline init params]
```

Here is an example of the bootconfig file for kernel/init parameters.:

```
kernel {
    root = 01234567-89ab-cdef-0123-456789abcd
}
init {
    splash
}
```

This will be copied into the kernel cmdline string as the following:

```
root="01234567-89ab-cdef-0123-456789abcd" -- splash
```

If user gives some other command line like,:

```
ro bootconfig -- quiet
```

The final kernel cmdline will be the following:

```
root="01234567-89ab-cdef-0123-456789abcd" ro bootconfig -- splash quiet
```

# Config File Limitation

Currently the maximum config size size is 32KB and the total key-words (not key-value entries) must be under 1024 nodes. Note: this is not the number of entries but nodes, an entry must consume more than 2 nodes (a key-word and a value). So theoretically, it will be up to 512 key-value pairs. If keys contains 3 words in average, it can contain 256 key-value pairs. In most cases, the number of config items will be under 100 entries and smaller than 8KB, so it would be enough. If the node number exceeds 1024, parser returns an error even if the file size is smaller than 32KB. (Note that this maximum size is not including the padding null characters.) Anyway, since bootconfig command verifies it when appending a boot config to initrd image, user can notice it before boot.

> 目前，配置大小的最大值为 32KB，总关键字（而非键值条目）必须在 1024 个节点以下。注意：这不是条目的数量，而是节点的数量，一个条目必须消耗 2 个以上的节点（一个关键字和一个值）。因此，从理论上讲，它将多达 512 个键值对。如果关键字平均包含 3 个单词，则可以包含 256 个键值对。在大多数情况下，配置项的数量将低于 100 个条目，并且小于 8KB，因此这就足够了。如果节点数超过 1024，即使文件大小小于 32KB，解析器也会返回错误。（注意，这个最大大小不包括填充的 null 字符。）无论如何，由于 bootconfig 命令在将引导配置附加到 initrd 映像时会验证它，所以用户可以在引导前注意到它。

# Bootconfig APIs

User can query or loop on key-value pairs, also it is possible to find a root (prefix) key node and find key-values under that node.

> 用户可以查询或循环键值对，也可以找到根（前缀）键节点并在该节点下找到键值。

If you have a key string, you can query the value directly with the key using xbc_find_value(). If you want to know what keys exist in the boot config, you can use xbc_for_each_key_value() to iterate key-value pairs. Note that you need to use xbc_array_for_each_value() for accessing each array\'s value, e.g.:

> 如果您有一个键字符串，您可以使用 xbc_find_value（）直接用键查询值。如果您想知道引导配置中存在哪些密钥，可以使用 xbc_for_each_key_value（）来迭代键值对。请注意，您需要使用 xbc_array_for_each_value（）来访问每个数组的值，例如：

```
vnode = NULL;
xbc_find_value("key.word", &vnode);
if (vnode && xbc_node_is_array(vnode))
    xbc_array_for_each_value(vnode, value) {
        printk("%s ", value);
    }
```

If you want to focus on keys which have a prefix string, you can use xbc_find_node() to find a node by the prefix string, and iterate keys under the prefix node with xbc_node_for_each_key_value().

> 如果您想关注具有前缀字符串的键，可以使用 xbc_find_node（）根据前缀字符串查找节点，并使用 xbc_node_for_each_key_value（）迭代前缀节点下的键。

But the most typical usage is to get the named value under prefix or get the named array under prefix as below:

> 但最典型的用法是在前缀下获取命名值或在前缀下获得命名数组，如下所示：

```
root = xbc_find_node("key.prefix");
value = xbc_node_find_value(root, "option", &vnode);
...
xbc_node_for_each_array_value(root, "array-option", value, anode) {
    ...
}
```

This accesses a value of \"key.prefix.option\" and an array of \"key.prefix.array-option\".

> 这访问一个值“keyprefix.option\”和一个数组“keyprefix.array option\”。

Locking is not needed, since after initialization, the config becomes read-only. All data and keys must be copied if you need to modify it.

> 不需要锁定，因为初始化后，配置将变为只读。如果需要修改，则必须复制所有数据和密钥。

# Functions and structures

::: kernel-doc
include/linux/bootconfig.h
:::

::: kernel-doc
lib/bootconfig.c
:::
