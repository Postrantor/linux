---
tip: translate by baidu@2024-01-30 21:30:08
title: Bisecting a bug
---

Last updated: 28 October 2016

# Introduction

Always try the latest kernel from kernel.org and build from source. If you are not confident in doing that please report the bug to your distribution vendor instead of to a kernel developer.

> 始终尝试 kernel.org 中的最新内核，并从源代码构建。如果您对此没有信心，请将错误报告给您的发行版供应商，而不是内核开发人员。

Finding bugs is not always easy. Have a go though. If you can\'t find it don\'t give up. Report as much as you have found to the relevant maintainer. See MAINTAINERS for who that is for the subsystem you have worked on.

> 发现 bug 并不总是那么容易。不过还是试一试吧。如果你找不到，不要放弃。向相关维护人员报告您所发现的内容。请参阅维护人员，了解您所处理的子系统的维护人员。

Before you submit a bug report read \'Documentation/admin-guide/reporting-issues.rst\'.

> 在提交错误报告之前，请阅读“文档/管理指南/报告问题.rst”。

# Devices not appearing

Often this is caused by udev/systemd. Check that first before blaming it on the kernel.

> 这通常是由 udev/systemd 引起的。在将其归咎于内核之前，请先检查一下。

# Finding patch that caused a bug

Using the provided tools with `git` makes finding bugs easy provided the bug is reproducible.

> 将提供的工具与“git”一起使用，只要 bug 是可复制的，就可以很容易地找到 bug。

Steps to do it:

- build the Kernel from its git source

- start bisect with[^1]:

        ```
        $ git bisect start
        ```

- mark the broken changeset with:

        ```
        $ git bisect bad [commit]
        ```

- mark a changeset where the code is known to work with:

        ```
        $ git bisect good [commit]
        ```

- rebuild the Kernel and test

- interact with git bisect by using either:

        ```
        $ git bisect good
        ```

  or:

        ```
        $ git bisect bad
        ```

  depending if the bug happened on the changeset you\'re testing

- After some interactions, git bisect will give you the changeset that likely caused the bug.

- For example, if you know that the current version is bad, and version 4.8 is good, you could do:

> -例如，如果您知道当前版本不好，而 4.8 版本很好，您可以执行以下操作：

```
$ git bisect start
$ git bisect bad                 # Current version is bad
$ git bisect good v4.8
```

For further references, please read:

- The man page for `git-bisect`
- [Fighting regressions with git bisect](https://www.kernel.org/pub/software/scm/git/docs/git-bisect-lk2009.html)
- [Fully automated bisecting with \"git bisect run\"](https://lwn.net/Articles/317154)
- [Using Git bisect to figure out when brokenness was introduced](http://webchick.net/node/99)

[^1]: You can, optionally, provide both good and bad arguments at git start with `git bisect start [BAD] [GOOD]`
