---
tip: translate by baidu@2024-01-30 21:40:50
---
---
title: Video Output Switcher Control
---

2006 <luming.yu@intel.com>


The output sysfs class driver provides an abstract video output layer that can be used to hook platform specific methods to enable/disable video output device through common sysfs interface. For example, on my IBM ThinkPad T42 laptop, The ACPI video driver registered its output devices and read/write method for \'state\' with output sysfs class. The user interface under sysfs is:

> 输出sysfs类驱动程序提供了一个抽象的视频输出层，可用于挂接特定于平台的方法，通过公共sysfs接口启用/禁用视频输出设备。例如，在我的IBM ThinkPad T42笔记本电脑上，ACPI视频驱动程序使用output sysfs类注册了“状态”的输出设备和读/写方法。sysfs下的用户界面是：

    linux:/sys/class/video_output # tree .
    .
    |-- CRT0
    |   |-- device -> ../../../devices/pci0000:00/0000:00:01.0
    |   |-- state
    |   |-- subsystem -> ../../../class/video_output
    |   `-- uevent
    |-- DVI0
    |   |-- device -> ../../../devices/pci0000:00/0000:00:01.0
    |   |-- state
    |   |-- subsystem -> ../../../class/video_output
    |   `-- uevent
    |-- LCD0
    |   |-- device -> ../../../devices/pci0000:00/0000:00:01.0
    |   |-- state
    |   |-- subsystem -> ../../../class/video_output
    |   `-- uevent
    `-- TV0
       |-- device -> ../../../devices/pci0000:00/0000:00:01.0
       |-- state
       |-- subsystem -> ../../../class/video_output
       `-- uevent
