---
tip: translate by baidu@2024-01-30 21:39:54
---
---
copyright: |
  1995\--1999 Martin Mares, \<<mj@ucw.cz>\>
title: Video Mode Selection Support 2.13
---

# Intro


This small document describes the \"Video Mode Selection\" feature which allows the use of various special video modes supported by the video BIOS. Due to usage of the BIOS, the selection is limited to boot time (before the kernel decompression starts) and works only on 80X86 machines that are booted through BIOS firmware (as opposed to through UEFI, kexec, etc.).

> 本小文档介绍了“视频模式选择”功能，该功能允许使用视频BIOS支持的各种特殊视频模式。由于使用BIOS，该选项仅限于引导时间（在内核解压缩开始之前），并且仅适用于通过BIOS固件（而不是通过UEFI、kexec等）引导的80X86机器。

::: note
::: title
Note
:::


Short intro for the impatient: Just use vga=ask for the first time, enter `scan` on the video mode prompt, pick the mode you want to use, remember its mode ID (the four-digit hexadecimal number) and then set the vga parameter to this number (converted to decimal first).

> 简短介绍：只需第一次使用vga=询问，在视频模式提示下输入“扫描”，选择您想要使用的模式，记住其模式ID（四位十六进制数字），然后将vga参数设置为该数字（先转换为十进制）。
:::


The video mode to be used is selected by a kernel parameter which can be specified in the kernel Makefile (the SVGA_MODE=\... line) or by the \"vga=\...\" option of LILO (or some other boot loader you use) or by the \"xrandr\" utility (present in standard Linux utility packages). You can use the following values of this parameter:

> 要使用的视频模式由内核参数选择，该内核参数可以在内核Makefile（SVGA_mode=\…行）中指定，也可以由LILO的“vga=\…\”选项（或您使用的其他引导加载程序）或“xrandr”实用程序（存在于标准Linux实用程序包中）指定。您可以使用此参数的以下值：

    NORMAL_VGA - Standard 80x25 mode available on all display adapters.

    EXTENDED_VGA - Standard 8-pixel font mode: 80x43 on EGA, 80x50 on VGA.

    ASK_VGA - Display a video mode menu upon startup (see below).

    0..35 - Menu item number (when you have used the menu to view the list of
       modes available on your adapter, you can specify the menu item you want
       to use). 0..9 correspond to "0".."9", 10..35 to "a".."z". Warning: the
       mode list displayed may vary as the kernel version changes, because the
       modes are listed in a "first detected -- first displayed" manner. It's
       better to use absolute mode numbers instead.

    0x.... - Hexadecimal video mode ID (also displayed on the menu, see below
       for exact meaning of the ID). Warning: LILO doesn't support
       hexadecimal numbers -- you have to convert it to decimal manually.

# Menu


The ASK_VGA mode causes the kernel to offer a video mode menu upon bootup. It displays a \"Press \<RETURN\> to see video modes available, \<SPACE\> to continue or wait 30 secs\" message. If you press \<RETURN\>, you enter the menu, if you press \<SPACE\> or wait 30 seconds, the kernel will boot up in the standard 80x25 mode.

> ASK_VGA模式使内核在启动时提供视频模式菜单。它显示一条“按\<RETURN\>查看可用的视频模式，按\<SPACE\>继续或等待30秒”的消息。如果按\<RETURN\>进入菜单，如果按\<SPACE\>或等待30秒，内核将以标准80x25模式启动。

The menu looks like:

    Video adapter: <name-of-detected-video-adapter>
    Mode:    COLSxROWS:
    0  0F00  80x25
    1  0F01  80x50
    2  0F02  80x43
    3  0F03  80x26
    ....
    Enter mode number or ``scan``: <flashing-cursor-here>


\<name-of-detected-video-adapter\> tells what video adapter did Linux detect \-- it\'s either a generic adapter name (MDA, CGA, HGC, EGA, VGA, VESA VGA \[a VGA with VESA-compliant BIOS\]) or a chipset name (e.g., Trident). Direct detection of chipsets is turned off by default as it\'s inherently unreliable due to absolutely insane PC design.

> \<name-of-detected-video-adapter\>neneneba告诉Linux检测到了什么视频适配器\-它要么是通用适配器名称（MDA、CGA、HGC、EGA、VGA、VESA-VGA \[带有符合VESA的BIOS的VGA \]），要么是芯片组名称（例如Trident）。芯片组的直接检测在默认情况下是关闭的，因为它本质上是不可靠的，因为电脑的设计非常疯狂。


\"0 0F00 80x25\" means that the first menu item (the menu items are numbered from \"0\" to \"9\" and from \"a\" to \"z\") is a 80x25 mode with ID=0x0f00 (see the next section for a description of mode IDs).

> \“0 0F00 80x25\”表示第一个菜单项（菜单项的编号从“0\”到“9\”，从“a\”到“z\”）是ID为0x0f00的80x25模式（有关模式ID的描述，请参阅下一节）。


\<flashing-cursor-here\> encourages you to enter the item number or mode ID you wish to set and press \<RETURN\>. If the computer complains something about \"Unknown mode ID\", it is trying to tell you that it isn\'t possible to set such a mode. It\'s also possible to press only \<RETURN\> which leaves the current mode.

> \＜在此处闪烁光标＞鼓励您输入要设置的项目编号或模式ID，然后按＜返回＞。如果计算机抱怨“未知模式ID”，它会试图告诉您无法设置这样的模式。也可以只按\<RETURN\>，退出当前模式。


The mode list usually contains a few basic modes and some VESA modes. In case your chipset has been detected, some chipset-specific modes are shown as well (some of these might be missing or unusable on your machine as different BIOSes are often shipped with the same card and the mode numbers depend purely on the VGA BIOS).

> 模式列表通常包含一些基本模式和一些VESA模式。如果检测到您的芯片组，还会显示一些特定于芯片组的模式（其中一些模式可能在您的机器上丢失或不可用，因为不同的BIOS通常与同一张卡一起提供，并且模式编号完全取决于VGA BIOS）。


The modes displayed on the menu are partially sorted: The list starts with the standard modes (80x25 and 80x50) followed by \"special\" modes (80x28 and 80x43), local modes (if the local modes feature is enabled), VESA modes and finally SVGA modes for the auto-detected adapter.

> 菜单上显示的模式有部分排序：列表从标准模式（80x25和80x50）开始，然后是“特殊”模式（80x28和80x43）、本地模式（如果启用了本地模式功能）、VESA模式，最后是自动检测适配器的SVGA模式。


If you are not happy with the mode list offered (e.g., if you think your card is able to do more), you can enter \"scan\" instead of item number / mode ID. The program will try to ask the BIOS for all possible video mode numbers and test what happens then. The screen will be probably flashing wildly for some time and strange noises will be heard from inside the monitor and so on and then, really all consistent video modes supported by your BIOS will appear (plus maybe some `ghost modes`). If you are afraid this could damage your monitor, don\'t use this function.

> 如果你对提供的模式列表不满意（例如，如果你认为你的卡可以做更多），你可以输入“scan”而不是项目编号/模式ID。该程序将尝试向BIOS询问所有可能的视频模式编号，然后测试会发生什么。屏幕可能会疯狂闪烁一段时间，并且会从显示器内部听到奇怪的噪音等等，然后，BIOS支持的所有一致视频模式都会出现（可能还有一些“幽灵模式”）。如果您担心这会损坏您的显示器，请不要使用此功能。


After scanning, the mode ordering is a bit different: the auto-detected SVGA modes are not listed at all and the modes revealed by `scan` are shown before all VESA modes.

> 扫描后，模式顺序有点不同：完全没有列出自动检测的SVGA模式，“扫描”显示的模式显示在所有VESA模式之前。

# Mode IDs


Because of the complexity of all the video stuff, the video mode IDs used here are also a bit complex. A video mode ID is a 16-bit number usually expressed in a hexadecimal notation (starting with \"0x\"). You can set a mode by entering its mode directly if you know it even if it isn\'t shown on the menu.

> 由于所有视频内容的复杂性，这里使用的视频模式ID也有点复杂。视频模式ID是一个16位数字，通常以十六进制表示（以\“0x\”开头）。如果您知道，即使菜单上没有显示，也可以直接输入模式来设置模式。

The ID numbers can be divided to those regions:

    0x0000 to 0x00ff - menu item references. 0x0000 is the first item. Don't use
     outside the menu as this can change from boot to boot (especially if you
     have used the ``scan`` feature).

    0x0100 to 0x017f - standard BIOS modes. The ID is a BIOS video mode number
     (as presented to INT 10, function 00) increased by 0x0100.

    0x0200 to 0x08ff - VESA BIOS modes. The ID is a VESA mode ID increased by
     0x0100. All VESA modes should be autodetected and shown on the menu.

    0x0900 to 0x09ff - Video7 special modes. Set by calling INT 0x10, AX=0x6f05.
     (Usually 940=80x43, 941=132x25, 942=132x44, 943=80x60, 944=100x60,
     945=132x28 for the standard Video7 BIOS)

    0x0f00 to 0x0fff - special modes (they are set by various tricks -- usually
     by modifying one of the standard modes). Currently available:
     0x0f00  standard 80x25, don't reset mode if already set (=FFFF)
     0x0f01  standard with 8-point font: 80x43 on EGA, 80x50 on VGA
     0x0f02  VGA 80x43 (VGA switched to 350 scanlines with a 8-point font)
     0x0f03  VGA 80x28 (standard VGA scans, but 14-point font)
     0x0f04  leave current video mode
     0x0f05  VGA 80x30 (480 scans, 16-point font)
     0x0f06  VGA 80x34 (480 scans, 14-point font)
     0x0f07  VGA 80x60 (480 scans, 8-point font)
     0x0f08  Graphics hack (see the VIDEO_GFX_HACK paragraph below)

    0x1000 to 0x7fff - modes specified by resolution. The code has a "0xRRCC"
     form where RR is a number of rows and CC is a number of columns.
     E.g., 0x1950 corresponds to a 80x25 mode, 0x2b84 to 132x43 etc.
     This is the only fully portable way to refer to a non-standard mode,
     but it relies on the mode being found and displayed on the menu
     (remember that mode scanning is not done automatically).

    0xff00 to 0xffff - aliases for backward compatibility:
     0xffff  equivalent to 0x0f00 (standard 80x25)
     0xfffe  equivalent to 0x0f01 (EGA 80x43 or VGA 80x50)


If you add 0x8000 to the mode ID, the program will try to recalculate vertical display timing according to mode parameters, which can be used to eliminate some annoying bugs of certain VGA BIOSes (usually those used for cards with S3 chipsets and old Cirrus Logic BIOSes) \-- mainly extra lines at the end of the display.

> 如果您在模式ID中添加0x8000，程序将尝试根据模式参数重新计算垂直显示时间，这可以用来消除某些VGA BIOS（通常用于带有S3芯片组和旧Cirrus Logic BIOS的卡）的一些恼人错误，主要是显示器末端的额外行数。

# Options


Build options for arch/x86/boot/\* are selected by the kernel kconfig utility and the kernel .config file.

> arch/x86/boot/\*的构建选项由内核kconfig实用程序和kernel.config文件选择。


VIDEO_GFX_HACK - includes special hack for setting of graphics modes to be used later by special drivers. Allows to set \_[any]() BIOS mode including graphic ones and forcing specific text screen resolution instead of peeking it from BIOS variables. Don\'t use unless you think you know what you\'re doing. To activate this setup, use mode number 0x0f08 (see the Mode IDs section above).

> VIDEO_GFX_HACK-包括特殊破解，用于设置稍后由特殊驱动程序使用的图形模式。允许设置\_[any]（）BIOS模式，包括图形模式和强制特定的文本屏幕分辨率，而不是从BIOS变量中窥探。除非你认为自己知道自己在做什么，否则不要使用。要激活此设置，请使用模式编号0x0f08（请参阅上面的“模式ID”部分）。

# Still doesn\'t work?


When the mode detection doesn\'t work (e.g., the mode list is incorrect or the machine hangs instead of displaying the menu), try to switch off some of the configuration options listed under \"Options\". If it fails, you can still use your kernel with the video mode set directly via the kernel parameter.

> 当模式检测不起作用时（例如，模式列表不正确或机器挂起而不是显示菜单），请尝试关闭“选项”下列出的一些配置选项。如果失败，您仍然可以使用内核，并通过内核参数直接设置视频模式。


In either case, please send me a bug report containing what \_[exactly]() happens and how do the configuration switches affect the behaviour of the bug.

> 无论哪种情况，请向我发送一份错误报告，其中包含\_[确切地]（）发生了什么，以及配置开关如何影响错误的行为。


If you start Linux from M\$-DOS, you might also use some DOS tools for video mode setting. In this case, you must specify the 0x0f04 mode (\"leave current settings\") to Linux, because if you don\'t and you use any non-standard mode, Linux will switch to 80x25 automatically.

> 如果您从M\$-DOS启动Linux，您也可以使用一些DOS工具来设置视频模式。在这种情况下，您必须将0x0f04模式（“保留当前设置”）指定给Linux，因为如果您不指定，并且使用任何非标准模式，Linux将自动切换到80x25。


If you set some extended mode and there\'s one or more extra lines on the bottom of the display containing already scrolled-out text, your VGA BIOS contains the most common video BIOS bug called \"incorrect vertical display end setting\". Adding 0x8000 to the mode ID might fix the problem. Unfortunately, this must be done manually \-- no autodetection mechanisms are available.

> 如果您设置了一些扩展模式，并且显示器底部有一行或多行额外的文字，其中包含已滚动的文字，则VGA BIOS会包含最常见的视频BIOS错误，称为“不正确的垂直显示端设置”。将0x8000添加到模式ID可能会解决此问题。不幸的是，这必须手动完成\-没有可用的自动检测机制。

# History

+-----------------+-------------------------------------------------------------------+
| 1.0 (??-Nov-95) | First version supporting all adapters supported by the old        |
+-----------------+-------------------------------------------------------------------+
| > setup.S       | \+ Cirrus Logic 54XX. Present in some 1.3.4? kernels              |
+-----------------+-------------------------------------------------------------------+
| > and then      | > removed due to instability on some machines.                    |
+-----------------+-------------------------------------------------------------------+
| 2.0 (28-Jan-96) | Rewritten from scratch. Cirrus Logic 64XX support added, almost   |
+-----------------+-------------------------------------------------------------------+
| > everythi      | ng is configurable, the VESA support should be much more          |
+-----------------+-------------------------------------------------------------------+
| > stable,       | explicit mode numbering allowed, \"scan\" implemented etc.        |
+-----------------+-------------------------------------------------------------------+
| 2.1 (30-Jan-96) | VESA modes moved to 0x200-0x3ff. Mode selection by resolution     |
+-----------------+-------------------------------------------------------------------+
| > supporte      | d\. Few bugs fixed. VESA modes are listed prior to                |
+-----------------+-------------------------------------------------------------------+
| > modes su      | pplied by SVGA autodetection as they are more reliable.           |
+-----------------+-------------------------------------------------------------------+
| > CLGD aut      | odetect works better. Doesn\'t depend on 80x25 being              |
+-----------------+-------------------------------------------------------------------+
| > active w      | hen started. Scanning fixed. 80x43 (any VGA) added.               |
+-----------------+-------------------------------------------------------------------+
| > Code cle      | aned up.                                                          |
+-----------------+-------------------------------------------------------------------+
| 2.2 (01-Feb-96) | EGA 80x43 fixed. VESA extended to 0x200-0x4ff (non-standard 02XX  |
+-----------------+-------------------------------------------------------------------+
| > VESA mod      | es work now). Display end bug workaround supported.               |
+-----------------+-------------------------------------------------------------------+
| > Special       | modes renumbered to allow adding of the \"recalculate\"           |
+-----------------+-------------------------------------------------------------------+
| > flag, 0x      | ffff and 0xfffe became aliases instead of real IDs.               |
+-----------------+-------------------------------------------------------------------+
| > Screen c      | ontents retained during mode changes.                             |
+-----------------+-------------------------------------------------------------------+
| 2.3 (15-Mar-96) | Changed to work with 1.3.74 kernel.                               |
+-----------------+-------------------------------------------------------------------+
| 2.4 (18-Mar-96) | Added patches by Hans Lermen fixing a memory overwrite problem    |
+-----------------+-------------------------------------------------------------------+
| > with som      | e boot loaders. Memory management rewritten to reflect            |
+-----------------+-------------------------------------------------------------------+
| > these ch      | anges. Unfortunately, screen contents retaining works             |
+-----------------+-------------------------------------------------------------------+
| > only wit      | h some loaders now.                                               |
+-----------------+-------------------------------------------------------------------+
| > Added a       | Tseng 132x60 mode.                                                |
+-----------------+-------------------------------------------------------------------+
| 2.5 (19-Mar-96) | Fixed a VESA mode scanning bug introduced in 2.4.                 |
+-----------------+-------------------------------------------------------------------+
| 2.6 (25-Mar-96) | Some VESA BIOS errors not reported \-- it fixes error reports on  |
+-----------------+-------------------------------------------------------------------+
| > several       | cards with broken VESA code (e.g., ATI VGA).                      |
+-----------------+-------------------------------------------------------------------+
| 2.7 (09-Apr-96) | \- Accepted all VESA modes in range 0x100 to 0x7ff, because some  |
+-----------------+-------------------------------------------------------------------+
| > cards         | use very strange mode numbers.                                    |
+-----------------+-------------------------------------------------------------------+
| > -   Added     | Realtek VGA modes (thanks to Gonzalo Tornaria).                   |
+-----------------+-------------------------------------------------------------------+
| > -   Hardwa    | re testing order slightly changed, tests based on ROM             |
+-----------------+-------------------------------------------------------------------+
| > conten        | ts done as first.                                                 |
+-----------------+-------------------------------------------------------------------+
| > -   Added     | support for special Video7 mode switching functions               |
+-----------------+-------------------------------------------------------------------+
| > (thank        | s to Tom Vander Aa).                                              |
+-----------------+-------------------------------------------------------------------+
| > -   Added     | 480-scanline modes (especially useful for notebooks,              |
+-----------------+-------------------------------------------------------------------+
| > origin        | al version written by <hhanemaa@cs.ruu.nl>, patched by            |
+-----------------+-------------------------------------------------------------------+
| > Jeff C        | hua, rewritten by me).                                            |
+-----------------+-------------------------------------------------------------------+
| > -   Screen    | > store/restore fixed.                                            |
+-----------------+-------------------------------------------------------------------+
| 2.8 (14-Apr-96) | \- Previous release was not compilable without CONFIG_VIDEO_SVGA. |
+-----------------+-------------------------------------------------------------------+
| > -   Better    | > recognition of text modes during mode scan.                     |
+-----------------+-------------------------------------------------------------------+
| 2.9 (12-May-96) | \- Ignored VESA modes 0x80 - 0xff (more VESA BIOS bugs!)          |
+-----------------+-------------------------------------------------------------------+
| 2.10(11-Nov-96) | \- The whole thing made optional.                                 |
+-----------------+-------------------------------------------------------------------+
| > -   Added     | the CONFIG_VIDEO_400_HACK switch.                                 |
+-----------------+-------------------------------------------------------------------+
| > -   Added     | the CONFIG_VIDEO_GFX_HACK switch.                                 |
+-----------------+-------------------------------------------------------------------+
| > -   Code c    | leanup.                                                           |
+-----------------+-------------------------------------------------------------------+
| 2.11(03-May-97) | \- Yet another cleanup, now including also the documentation.     |
+-----------------+-------------------------------------------------------------------+
| > -   Direct    | > testing of SVGA adapters turned off by default, `scan`          |
+-----------------+-------------------------------------------------------------------+
| > offere        | d explicitly on the prompt line.                                  |
+-----------------+-------------------------------------------------------------------+
| > -   Remove    | d the doc section describing adding of new probing                |
+-----------------+-------------------------------------------------------------------+
| > functi        | ons as I try to get rid of \_[all]() hardware probing here.       |
+-----------------+-------------------------------------------------------------------+
| 2.12(25-May-98) | Added support for VESA frame buffer graphics.                     |
+-----------------+-------------------------------------------------------------------+
| 2.13(14-May-99) | Minor documentation fixes.                                        |
+-----------------+-------------------------------------------------------------------+
