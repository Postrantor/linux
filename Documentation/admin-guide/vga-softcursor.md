---
tip: translate by baidu@2024-01-30 21:40:49
---
---
title: Software cursor for VGA
---


by Pavel Machek \<<pavel@atrey.karlin.mff.cuni.cz>\> and Martin Mares \<<mj@atrey.karlin.mff.cuni.cz>\>

> Pavel Machek\<<pavel@atrey.karlin.mff.cuni.cz>\>和Martin Mares\<<mj@atrey.karlin.mff.cuni.cz>\>


Linux now has some ability to manipulate cursor appearance. Normally, you can set the size of hardware cursor. You can now play a few new tricks: you can make your cursor look like a non-blinking red block, make it inverse background of the character it\'s over or to highlight that character and still choose whether the original hardware cursor should remain visible or not. There may be other things I have never thought of.

> Linux现在有一些操作光标外观的能力。通常情况下，您可以设置硬件光标的大小。现在，您可以玩一些新技巧：您可以使光标看起来像一个不闪烁的红色块，使其与所处角色的背景相反，或者突出显示该角色，然后仍然选择原始硬件光标是否应保持可见。可能还有其他事情我从未想过。


The cursor appearance is controlled by a `<ESC>[?1;2;3c` escape sequence where 1, 2 and 3 are parameters described below. If you omit any of them, they will default to zeroes.

> 光标外观由`<ESC>[？1；2；3c`转义序列控制，其中1、2和3是下面描述的参数。如果省略其中任何一个，它们将默认为零。

first Parameter

:   specifies cursor size:

        0=default
        1=invisible
        2=underline,
        ...
        8=full block
        + 16 if you want the software cursor to be applied
        + 32 if you want to always change the background color
        + 64 if you dislike having the background the same as the
             foreground.

    Highlights are ignored for the last two flags.

second parameter


:   selects character attribute bits you want to change (by simply XORing them with the value of this parameter). On standard VGA, the high four bits specify background and the low four the foreground. In both groups, low three bits set color (as in normal color codes used by the console) and the most significant one turns on highlight (or sometimes blinking \-- it depends on the configuration of your VGA).

> ：选择要更改的字符属性位（只需将它们与此参数的值进行异或运算）。在标准VGA上，高四位指定背景，低四位指定前景。在这两组中，低三位设置颜色（如控制台使用的正常颜色代码），最重要的一位打开高亮显示（或有时闪烁\——这取决于VGA的配置）。

third parameter

:   consists of character attribute bits you want to set.

    Bit setting takes place before bit toggling, so you can simply clear a bit by including it in both the set mask and the toggle mask.

# Examples

To get normal blinking underline, use:

    echo -e '\033[?2c'

To get blinking block, use:

    echo -e '\033[?6c'

To get red non-blinking block, use:

    echo -e '\033[?17;0;64c'
