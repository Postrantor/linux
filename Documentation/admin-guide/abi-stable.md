---
tip: translate by baidu@2024-01-30 21:29:16
title: ABI stable symbols
---

Documents the interfaces that the developer has defined to be stable.

Userspace programs are free to use these interfaces with no restrictions, and backward compatibility for them will be guaranteed for at least 2 years.

> 用户空间程序可以自由使用这些接口，没有任何限制，并且它们的向后兼容性将保证至少 2 年。

Most interfaces (like syscalls) are expected to never change and always be available.

> 大多数接口（如系统调用）预计永远不会更改，并且始终可用。

::: {.kernel-abi rst=""}
\$srctree/Documentation/ABI/stable
:::
