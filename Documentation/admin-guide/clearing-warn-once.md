---
tip: translate by baidu@2024-01-30 21:32:55
---
---
title: Clearing WARN_ONCE
---

WARN_ONCE / WARN_ON_ONCE / printk_once only emit a message once.

echo 1 \> /sys/kernel/debug/clear_warn_once


clears the state and allows the warnings to print once again. This can be useful after test suite runs to reproduce problems.

> 清除状态并允许再次打印警告。这在测试套件运行以重现问题后非常有用。
