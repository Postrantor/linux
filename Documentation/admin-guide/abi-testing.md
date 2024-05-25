---
tip: translate by baidu@2024-01-30 21:29:17
title: ABI testing symbols
---

Documents interfaces that are felt to be stable, as the main development of this interface has been completed.

> 文档接口被认为是稳定的，因为这个接口的主要开发已经完成。

The interface can be changed to add new features, but the current interface will not break by doing this, unless grave errors or security problems are found in them.

> 可以更改接口以添加新功能，但当前接口不会因此而中断，除非在其中发现严重错误或安全问题。

Userspace programs can start to rely on these interfaces, but they must be aware of changes that can occur before these interfaces move to be marked stable.

> 用户空间程序可以开始依赖这些接口，但在这些接口被标记为稳定之前，它们必须意识到可能发生的更改。

Programs that use these interfaces are strongly encouraged to add their name to the description of these interfaces, so that the kernel developers can easily notify them if any changes occur.

> 强烈鼓励使用这些接口的程序将其名称添加到这些接口的描述中，这样内核开发人员就可以在发生任何更改时轻松地通知他们。

::: {.kernel-abi rst=""}
\$srctree/Documentation/ABI/testing
:::
