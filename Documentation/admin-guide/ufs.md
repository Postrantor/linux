---
tip: translate by baidu@2024-01-30 21:40:40
---
---
title: Using UFS
---

mount -t ufs -o ufstype=type_of_ufs device dir

# UFS Options

ufstype=type_of_ufs


:   UFS is a file system widely used in different operating systems. The problem are differences among implementations. Features of some implementations are undocumented, so its hard to recognize type of ufs automatically. That\'s why user must specify type of ufs manually by mount option ufstype. Possible values are:

> ：UFS是一种广泛用于不同操作系统的文件系统。问题在于实现之间的差异。一些实现的特性是未记录的，因此很难自动识别uf的类型。这就是为什么用户必须通过挂载选项ufstype手动指定ufs的类型。可能的值为：

    old

    :   old format of ufs default value, supported as read-only

    44bsd

    :   used in FreeBSD, NetBSD, OpenBSD supported as read-write

    ufs2

    :   used in FreeBSD 5.x supported as read-write

    5xbsd

    :   synonym for ufs2

    sun

    :   used in SunOS (Solaris) supported as read-write

    sunx86

    :   used in SunOS for Intel (Solarisx86) supported as read-write

    hp

    :   used in HP-UX supported as read-only

    nextstep

    :   used in NextStep supported as read-only

    nextstep-cd

    :   used for NextStep CDROMs (block_size == 2048) supported as read-only

    openstep

    :   used in OpenStep supported as read-only

## Possible Problems

See next section, if you have any.

## Bug Reports


Any ufs bug report you can send to <daniel.pirkl@email.cz> or to <dushistov@mail.ru> (do not send partition tables bug reports).

> 您可以发送到的任何ufs错误报告<daniel.pirkl@email.cz>或至<dushistov@mail.ru>（不要发送分区表错误报告）。
