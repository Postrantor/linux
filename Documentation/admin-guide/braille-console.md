---
tip: translate by baidu@2024-01-30 21:30:05
title: Linux Braille Console
---

To get early boot messages on a braille device (before userspace screen readers can start), you first need to compile the support for the usual serial console (see `Documentation/admin-guide/serial-console.rst <serial_console>`{.interpreted-text role="ref"}), and for braille device (in `Device Drivers --> Accessibility support --> Console on braille device`{.interpreted-text role="menuselection"}).

> 要在盲文设备上获得早期启动消息（在用户空间屏幕阅读器启动之前），您首先需要编译对常用串行控制台的支持（请参阅`Documentation/admin guide/serial-console.rst<serial_console>`{.depredicted text role=“ref”}），和用于盲文设备（在`device Drivers-->Accessibility support-->Console on braille device`{.depredicted text role=“menuselection”}中）。

Then you need to specify a `console=brl`, option on the kernel command line, the format is:

> 然后，您需要在内核命令行上指定一个`console=brl`选项，格式为：

```
    console=brl,serial_options...
```

where `serial_options...` are the same as described in `Documentation/admin-guide/serial-console.rst <serial_console>`{.interpreted-text role="ref"}.

> 其中`serial_options…`与`Documentation/admin-guide/serial-console.rst＜serial_console>`{.depredicted text role=“ref”}中所述相同。

So for instance you can use `console=brl,ttyS0` if the braille device is connected to the first serial port, and `console=brl,ttyS0,115200` to override the baud rate to 115200, etc.

> 因此，例如，如果盲文设备连接到第一个串行端口，则可以使用“console=brl，ttyS0”，而“console=brl，TTyS0115200”则可以将波特率改写为 115200，等等。

By default, the braille device will just show the last kernel message (console mode). To review previous messages, press the Insert key to switch to the VT review mode. In review mode, the arrow keys permit to browse in the VT content, `PAGE-UP`{.interpreted-text role="kbd"}/`PAGE-DOWN`{.interpreted-text role="kbd"} keys go at the top/bottom of the screen, and the `HOME`{.interpreted-text role="kbd"} key goes back to the cursor, hence providing very basic screen reviewing facility.

> 默认情况下，盲文设备将只显示最后一条内核消息（控制台模式）。要查看以前的消息，请按插入键切换到 VT 查看模式。在回顾模式中，箭头键允许浏览 VT 内容，`PAGE-UP`｛.depredicted text role=“kbd”｝/`PAGE-DOWN`｛-depredicted textrole=”kbd“｝键位于屏幕的顶部/底部，`HOME`｛.epredicted text role=‘kbd’｝键返回光标，因此提供了非常基本的屏幕回顾功能。

Sound feedback can be obtained by adding the `braille_console.sound=1` kernel parameter.

> 可以通过添加“braille_console.Sound=1”内核参数来获得声音反馈。

For simplicity, only one braille console can be enabled, other uses of `console=brl,...` will be discarded. Also note that it does not interfere with the console selection mechanism described in `Documentation/admin-guide/serial-console.rst <serial_console>`{.interpreted-text role="ref"}.

> 为了简单起见，只能启用一个盲文控制台，其他使用`console=brl，…`将被丢弃。另请注意，它不会干扰`Documentation/admin guide/serial-console.rst＜serial-console＞`{.depreted text role=“ref”}中描述的控制台选择机制。

For now, only the VisioBraille device is supported.

Samuel Thibault \<<samuel.thibault@ens-lyon.org>\>
