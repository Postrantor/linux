---
tip: translate by baidu@2024-01-30 21:40:40
---
---
title: Unicode support
---

> Last update: 2005-01-17, version 1.4


Note: The original version of this document, which was maintained at lanana.org as part of the Linux Assigned Names And Numbers Authority (LANANA) project, is no longer existent. So, this version in the mainline Linux kernel is now the maintained main document.

> 注：作为Linux名称和数字分配机构（lanana）项目的一部分，本文档的原始版本已不存在。因此，主流Linux内核中的这个版本现在是维护的主文档。

# Introduction


The Linux kernel code has been rewritten to use Unicode to map characters to fonts. By downloading a single Unicode-to-font table, both the eight-bit character sets and UTF-8 mode are changed to use the font as indicated.

> Linux内核代码已被重写为使用Unicode将字符映射到字体。通过下载单个Unicode到字体表，八位字符集和UTF-8模式都将更改为使用所示的字体。


This changes the semantics of the eight-bit character tables subtly. The four character tables are now:

> 这微妙地改变了八位字符表的语义。现在的四个字符表是：


  Map symbol Map                                           name Escape code (G0)                                                                         

> 地图符号地图名称转义码（G0）

  -------------------------------------------------------- --------------------------------------------------------------------------------------------- --

> -------------------------------------------------------- --------------------------------------------------------------------------------------------- --

  LAT1_MAP Lati GRAF_MAP DEC IBMPC_MAP IBM USER_MAP User   n-1 (ISO 8859-1) ESC ( B VT100 pseudographics ESC ( 0 code page 437 ESC ( U defined ESC ( K   

> LAT1_MAP Lati GRAF_MAP DEC IBMPC_MAP IBM USER_MAP USER n-1（ISO 8859-1）ESC（B VT100伪图形ESC（0代码页437 ESC（U定义的ESC（K


In particular, ESC ( U is no longer \"straight to font\", since the font might be completely different than the IBM character set. This permits for example the use of block graphics even with a Latin-1 font loaded.

> 特别是，ESC（U）不再是“直接到字体”，因为字体可能与IBM字符集完全不同。例如，即使加载了Latin-1字体，也可以使用块图形。


Note that although these codes are similar to ISO 2022, neither the codes nor their uses match ISO 2022; Linux has two 8-bit codes (G0 and G1), whereas ISO 2022 has four 7-bit codes (G0-G3).

> 请注意，尽管这些代码与ISO 2022相似，但代码及其用途都与ISO 2022不匹配；Linux有两个8位代码（G0和G1），而ISO 2022有四个7位代码（GO-G3）。


In accordance with the Unicode standard/ISO 10646 the range U+F000 to U+F8FF has been reserved for OS-wide allocation (the Unicode Standard refers to this as a \"Corporate Zone\", since this is inaccurate for Linux we call it the \"Linux Zone\"). U+F000 was picked as the starting point since it lets the direct-mapping area start on a large power of two (in case 1024- or 2048-character fonts ever become necessary). This leaves U+E000 to U+EFFF as End User Zone.

> 根据Unicode标准/ISO 10646，范围U+F000到U+F8FF已保留用于操作系统范围内的分配（Unicode标准将其称为“企业区域”，因为这对Linux不准确，我们称之为“Linux区域”）。选择U+F000作为起点，因为它允许直接映射区域以2的大幂开始（在需要1024或2048字符字体的情况下）。这使得U+E000到U+EFFF成为最终用户区域。


\[v1.2\]: The Unicodes range from U+F000 and up to U+F7FF have been hard-coded to map directly to the loaded font, bypassing the translation table. The user-defined map now defaults to U+F000 to U+F0FF, emulating the previous behaviour. In practice, this range might be shorter; for example, vgacon can only handle 256-character (U+F000..U+F0FF) or 512-character (U+F000..U+F1FF) fonts.

> \[v1.2\]：从U+F000到U+F7FF的Unicode都经过了硬编码，可以绕过翻译表直接映射到加载的字体。用户定义的映射现在默认为U+F000到U+F0FF，模仿以前的行为。在实践中，这个范围可能更短；例如，vgacon只能处理256个字符（U+F000..U+F0FF）或512个字符（U+F000.U+F1FF）的字体。

# Actual characters assigned in the Linux Zone


In addition, the following characters not present in Unicode 1.1.4 have been defined; these are used by the DEC VT graphics map. \[v1.2\] THIS USE IS OBSOLETE AND SHOULD NO LONGER BE USED; PLEASE SEE BELOW.

> 此外，还定义了以下Unicode 1.1.4中不存在的字符；这些被DEC-VT图形图所使用\[v1.2\]此用途已废弃，不应再使用；请参见下文。

  -------- ----------------------------------------
  U+F800   DEC VT GRAPHICS HORIZONTAL LINE SCAN 1
  U+F801   DEC VT GRAPHICS HORIZONTAL LINE SCAN 3
  U+F803   DEC VT GRAPHICS HORIZONTAL LINE SCAN 7
  U+F804   DEC VT GRAPHICS HORIZONTAL LINE SCAN 9
  -------- ----------------------------------------


The DEC VT220 uses a 6x10 character matrix, and these characters form a smooth progression in the DEC VT graphics character set. I have omitted the scan 5 line, since it is also used as a block-graphics character, and hence has been coded as U+2500 FORMS LIGHT HORIZONTAL.

> DEC VT220使用6x10字符矩阵，这些字符在DEC VT图形字符集中形成平滑的进程。我省略了扫描5行，因为它也被用作块图形字符，因此被编码为U+2500 FORMS LIGHT HORIZONTAL。


\[v1.3\]: These characters have been officially added to Unicode 3.2.0; they are added at U+23BA, U+23BB, U+23BC, U+23BD. Linux now uses the new values.

> \[v1.3\]：这些字符已被正式添加到Unicode 3.2.0中；它们在U+23BA、U+23BB、U+23B、U+23DD处添加。Linux现在使用新的值。


\[v1.2\]: The following characters have been added to represent common keyboard symbols that are unlikely to ever be added to Unicode proper since they are horribly vendor-specific. This, of course, is an excellent example of horrible design.

> \[v1.2\]：添加了以下字符来表示通用键盘符号，这些符号不太可能添加到Unicode中，因为它们非常特定于供应商。当然，这是一个糟糕设计的极好例子。

  -------- -------------------------------
  U+F810   KEYBOARD SYMBOL FLYING FLAG
  U+F811   KEYBOARD SYMBOL PULLDOWN MENU
  U+F812   KEYBOARD SYMBOL OPEN APPLE
  U+F813   KEYBOARD SYMBOL SOLID APPLE
  -------- -------------------------------

# Klingon language support


In 1996, Linux was the first operating system in the world to add support for the artificial language Klingon, created by Marc Okrand for the \"Star Trek\" television series. This encoding was later adopted by the ConScript Unicode Registry and proposed (but ultimately rejected) for inclusion in Unicode Plane 1. Thus, it remains as a Linux/CSUR private assignment in the Linux Zone.

> 1996年，Linux成为世界上第一个添加对人工语言克林贡语支持的操作系统，该语言由Marc Okrand为《星际迷航》电视剧创建。这种编码后来被ConScript Unicode注册表采用，并被提议（但最终被拒绝）包含在Unicode平面1中。因此，它仍然是Linux区域中的Linux/CSUR私有分配。


This encoding has been endorsed by the Klingon Language Institute. For more information, contact them at:

> 这种编码方式已经得到了克林贡语言研究所的认可。有关更多信息，请联系他们：

> <http://www.kli.org/>


Since the characters in the beginning of the Linux CZ have been more of the dingbats/symbols/forms type and this is a language, I have located it at the end, on a 16-cell boundary in keeping with standard Unicode practice.

> 由于Linux CZ开始时的字符更多地是dingbats/符号/形式类型，而且这是一种语言，我根据标准Unicode实践将其定位在16单元边界的末尾。

::: note
::: title
Note
:::


This range is now officially managed by the ConScript Unicode Registry. The normative reference is at:

> 此范围现在由ConScript Unicode注册表正式管理。规范性参考文件位于：

> <https://www.evertype.com/standards/csur/klingon.html>
:::


Klingon has an alphabet of 26 characters, a positional numeric writing system with 10 digits, and is written left-to-right, top-to-bottom.

> 克林贡语的字母表有26个字符，位置数字书写系统有10个数字，从左到右、从上到下书写。


Several glyph forms for the Klingon alphabet have been proposed. However, since the set of symbols appear to be consistent throughout, with only the actual shapes being different, in keeping with standard Unicode practice these differences are considered font variants.

> 已经提出了克林贡字母的几种字形形式。然而，由于符号集看起来自始至终都是一致的，只有实际的形状不同，因此根据Unicode的标准做法，这些差异被视为字体变体。

+----------+------------------------------------------------------+
| U+F8D0   | KLINGON LETTER A                                     |
+----------+------------------------------------------------------+
| U+F8D1   | KLINGON LETTER B                                     |
+----------+------------------------------------------------------+
| U+F8D2   | KLINGON LETTER CH                                    |
+----------+------------------------------------------------------+
| U+F8D3   | KLINGON LETTER D                                     |
+----------+------------------------------------------------------+
| U+F8D4   | KLINGON LETTER E                                     |
+----------+------------------------------------------------------+
| U+F8D5   | KLINGON LETTER GH                                    |
+----------+------------------------------------------------------+
| U+F8D6   | KLINGON LETTER H                                     |
+----------+------------------------------------------------------+
| U+F8D7   | KLINGON LETTER I                                     |
+----------+------------------------------------------------------+
| U+F8D8   | KLINGON LETTER J                                     |
+----------+------------------------------------------------------+
| U+F8D9   | KLINGON LETTER L                                     |
+----------+------------------------------------------------------+
| U+F8DA   | KLINGON LETTER M                                     |
+----------+------------------------------------------------------+
| U+F8DB   | KLINGON LETTER N                                     |
+----------+------------------------------------------------------+
| U+F8DC   | KLINGON LETTER NG                                    |
+----------+------------------------------------------------------+
| U+F8DD   | KLINGON LETTER O                                     |
+----------+------------------------------------------------------+
| U+F8DE   | KLINGON LETTER P                                     |
+----------+------------------------------------------------------+
| U+F8DF   | KLINGON LETTER Q                                     |
+----------+------------------------------------------------------+
| > -   Wr | itten \<q\> in standard Okrand Latin transliteration |
+----------+------------------------------------------------------+
| U+F8E0   | KLINGON LETTER QH                                    |
+----------+------------------------------------------------------+
| > -   Wr | itten \<Q\> in standard Okrand Latin transliteration |
+----------+------------------------------------------------------+
| U+F8E1   | KLINGON LETTER R                                     |
+----------+------------------------------------------------------+
| U+F8E2   | KLINGON LETTER S                                     |
+----------+------------------------------------------------------+
| U+F8E3   | KLINGON LETTER T                                     |
+----------+------------------------------------------------------+
| U+F8E4   | KLINGON LETTER TLH                                   |
+----------+------------------------------------------------------+
| U+F8E5   | KLINGON LETTER U                                     |
+----------+------------------------------------------------------+
| U+F8E6   | KLINGON LETTER V                                     |
+----------+------------------------------------------------------+
| U+F8E7   | KLINGON LETTER W                                     |
+----------+------------------------------------------------------+
| U+F8E8   | KLINGON LETTER Y                                     |
+----------+------------------------------------------------------+
| U+F8E9   | KLINGON LETTER GLOTTAL STOP                          |
+----------+------------------------------------------------------+
| U+F8F0   | KLINGON DIGIT ZERO                                   |
+----------+------------------------------------------------------+
| U+F8F1   | KLINGON DIGIT ONE                                    |
+----------+------------------------------------------------------+
| U+F8F2   | KLINGON DIGIT TWO                                    |
+----------+------------------------------------------------------+
| U+F8F3   | KLINGON DIGIT THREE                                  |
+----------+------------------------------------------------------+
| U+F8F4   | KLINGON DIGIT FOUR                                   |
+----------+------------------------------------------------------+
| U+F8F5   | KLINGON DIGIT FIVE                                   |
+----------+------------------------------------------------------+
| U+F8F6   | KLINGON DIGIT SIX                                    |
+----------+------------------------------------------------------+
| U+F8F7   | KLINGON DIGIT SEVEN                                  |
+----------+------------------------------------------------------+
| U+F8F8   | KLINGON DIGIT EIGHT                                  |
+----------+------------------------------------------------------+
| U+F8F9   | KLINGON DIGIT NINE                                   |
+----------+------------------------------------------------------+
| U+F8FD   | KLINGON COMMA                                        |
+----------+------------------------------------------------------+
| U+F8FE   | KLINGON FULL STOP                                    |
+----------+------------------------------------------------------+
| U+F8FF   | KLINGON SYMBOL FOR EMPIRE                            |
+----------+------------------------------------------------------+

# Other Fictional and Artificial Scripts


Since the assignment of the Klingon Linux Unicode block, a registry of fictional and artificial scripts has been established by John Cowan \<<jcowan@reutershealth.com>\> and Michael Everson \<<everson@evertype.com>\>. The ConScript Unicode Registry is accessible at:

> 自从克林贡Linux Unicode块被分配以来，John Cowan已经建立了一个虚构和人工脚本的注册表\<<jcowan@reutershealth.com>\>和Michael Everson\<<everson@evertype.com>\>. ConScript Unicode注册表可在以下位置访问：

> <https://www.evertype.com/standards/csur/>


The ranges used fall at the low end of the End User Zone and can hence not be normatively assigned, but it is recommended that people who wish to encode fictional scripts use these codes, in the interest of interoperability. For Klingon, CSUR has adopted the Linux encoding. The CSUR people are driving adding Tengwar and Cirth into Unicode Plane 1; the addition of Klingon to Unicode Plane 1 has been rejected and so the above encoding remains official.

> 所使用的范围位于最终用户区域的低端，因此无法进行规范分配，但为了互操作性，建议希望对虚构脚本进行编码的人使用这些代码。对于克林贡，CSUR采用了Linux编码。CSUR人正在推动将Tengwar和Circh添加到Unicode平面1中；将克林贡语添加到Unicode平面1已经被拒绝，因此上述编码仍然是官方的。
