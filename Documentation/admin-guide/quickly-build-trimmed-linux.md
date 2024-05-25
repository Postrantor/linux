---
tip: translate by baidu@2024-01-30 21:37:05
---
---
title: How to quickly build a trimmed Linux kernel
---


This guide explains how to swiftly build Linux kernels that are ideal for testing purposes, but perfectly fine for day-to-day use, too.

> 本指南解释了如何快速构建Linux内核，这些内核非常适合测试，但也非常适合日常使用。

# The essence of the process (aka \'TL;DR\')


*\[If you are new to compiling Linux, ignore this TLDR and head over to the next section below: it contains a step-by-step guide, which is more detailed, but still brief and easy to follow; that guide and its accompanying reference section also mention alternatives, pitfalls, and additional aspects, all of which might be relevant for you.\]*

> *\[如果您是编译Linux的新手，请忽略此TLDR并转到下面的下一节：它包含一个循序渐进的指南，该指南更详细，但仍然简短且易于遵循；该指南及其附带的参考部分还提到了替代方案、陷阱和其他方面，所有这些都可能与您相关。\]*


If your system uses techniques like Secure Boot, prepare it to permit starting self-compiled Linux kernels; install compilers and everything else needed for building Linux; make sure to have 12 Gigabyte free space in your home directory. Now run the following commands to download fresh Linux mainline sources, which you then use to configure, build and install your own kernel:

> 如果您的系统使用安全引导等技术，请准备好允许启动自编译的Linux内核；安装编译器和构建Linux所需的一切；确保您的主目录中有12 GB的可用空间。现在运行以下命令下载新的Linux主线源代码，然后使用这些源代码配置、构建和安装自己的内核：

    git clone --depth 1 -b master \
      https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git ~/linux/
    cd ~/linux/
    # Hint: if you want to apply patches, do it at this point. See below for details.
    # Hint: it's recommended to tag your build at this point. See below for details.
    yes "" | make localmodconfig
    # Hint: at this point you might want to adjust the build configuration; you'll
    #   have to, if you are running Debian. See below for details.
    make -j $(nproc --all)
    # Note: on many commodity distributions the next command suffices, but on Arch
    #   Linux, its derivatives, and some others it does not. See below for details.
    command -v installkernel && sudo make modules_install install
    reboot

If you later want to build a newer mainline snapshot, use these commands:

    cd ~/linux/
    git fetch --depth 1 origin
    # Note: the next command will discard any changes you did to the code:
    git checkout --force --detach origin/master
    # Reminder: if you want to (re)apply patches, do it at this point.
    # Reminder: you might want to add or modify a build tag at this point.
    make olddefconfig
    make -j $(nproc --all)
    # Reminder: the next command on some distributions does not suffice.
    command -v installkernel && sudo make modules_install install
    reboot

# Step-by-step guide


Compiling your own Linux kernel is easy in principle. There are various ways to do it. Which of them actually work and is the best depends on the circumstances.

> 原则上，编译自己的Linux内核很容易。有多种方法可以做到这一点。其中哪种方法真正有效，最好取决于具体情况。


This guide describes a way perfectly suited for those who want to quickly install Linux from sources without being bothered by complicated details; the goal is to cover everything typically needed on mainstream Linux distributions running on commodity PC or server hardware.

> 本指南介绍了一种非常适合那些希望从源代码快速安装Linux而不受复杂细节困扰的人的方法；目标是覆盖在商品PC或服务器硬件上运行的主流Linux发行版上通常需要的一切。


The described approach is great for testing purposes, for example to try a proposed fix or to check if a problem was already fixed in the latest codebase. Nonetheless, kernels built this way are also totally fine for day-to-day use while at the same time being easy to keep up to date.

> 所描述的方法非常适合测试目的，例如尝试所提出的修复或检查问题是否已经在最新的代码库中修复。尽管如此，以这种方式构建的内核对于日常使用来说也是完全可以的，同时也很容易跟上最新的步伐。


The following steps describe the important aspects of the process; a comprehensive reference section later explains each of them in more detail. It sometimes also describes alternative approaches, pitfalls, as well as errors that might occur at a particular point \-- and how to then get things rolling again.

> 以下步骤描述了该过程的重要方面；后面的综合参考部分将更详细地解释其中的每一个。它有时还描述了替代方法、陷阱以及在特定时刻可能发生的错误，以及如何重新开始。

::: {#backup_sbs}
> -   Create a fresh backup and put system repair and restore tools at hand, just to be prepared for the unlikely case of something going sideways.
>
>     \[`details<backup>`{.interpreted-text role="ref"}\]
:::

::: {#secureboot_sbs}
> -   On platforms with \'Secure Boot\' or similar techniques, prepare everything to ensure the system will permit your self-compiled kernel to boot later. The quickest and easiest way to achieve this on commodity x86 systems is to disable such techniques in the BIOS setup utility; alternatively, remove their restrictions through a process initiated by `mokutil --disable-validation`.
>
>     \[`details<secureboot>`{.interpreted-text role="ref"}\]
:::

::: {#buildrequires_sbs}
> -   Install all software required to build a Linux kernel. Often you will need: \'bc\', \'binutils\' (\'ld\' et al.), \'bison\', \'flex\', \'gcc\', \'git\', \'openssl\', \'pahole\', \'perl\', and the development headers for \'libelf\' and \'openssl\'. The reference section shows how to quickly install those on various popular Linux distributions.
>
>     \[`details<buildrequires>`{.interpreted-text role="ref"}\]
:::

::: {#diskspace_sbs}
> -   Ensure to have enough free space for building and installing Linux. For the latter 150 Megabyte in /lib/ and 100 in /boot/ are a safe bet. For storing sources and build artifacts 12 Gigabyte in your home directory should typically suffice. If you have less available, be sure to check the reference section for the step that explains adjusting your kernels build configuration: it mentions a trick that reduce the amount of required space in /home/ to around 4 Gigabyte.
>
>     \[`details<diskspace>`{.interpreted-text role="ref"}\]
:::

::: {#sources_sbs}
> -   Retrieve the sources of the Linux version you intend to build; then change into the directory holding them, as all further commands in this guide are meant to be executed from there.
>
>     *\[Note: the following paragraphs describe how to retrieve the sources by partially cloning the Linux stable git repository. This is called a shallow clone. The reference section explains two alternatives:* `packaged
>     archives<sources_archive>`{.interpreted-text role="ref"} *and* `a full git clone<sources_full>`{.interpreted-text role="ref"} *; prefer the latter, if downloading a lot of data does not bother you, as that will avoid some* `peculiar characteristics of shallow clones the
>     reference section explains<sources_shallow>`{.interpreted-text role="ref"} *.\]*
>
>     First, execute the following command to retrieve a fresh mainline codebase:
>
>         git clone --no-checkout --depth 1 -b master \
>           https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git ~/linux/
>         cd ~/linux/
>
>     If you want to access recent mainline releases and pre-releases, deepen you clone\'s history to the oldest mainline version you are interested in:
>
>         git fetch --shallow-exclude=v6.0 origin
>
>     In case you want to access a stable/longterm release (say v6.1.5), simply add the branch holding that series; afterwards fetch the history at least up to the mainline version that started the series (v6.1):
>
>         git remote set-branches --add origin linux-6.1.y
>         git fetch --shallow-exclude=v6.0 origin
>
>     Now checkout the code you are interested in. If you just performed the initial clone, you will be able to check out a fresh mainline codebase, which is ideal for checking whether developers already fixed an issue:
>
>         git checkout --detach origin/master
>
>     If you deepened your clone, you instead of `origin/master` can specify the version you deepened to (`v6.0` above); later releases like `v6.1` and pre-release like `v6.2-rc1` will work, too. Stable or longterm versions like `v6.1.5` work just the same, if you added the appropriate stable/longterm branch as described.
>
>     \[`details<sources>`{.interpreted-text role="ref"}\]
:::

::: {#patching_sbs}
> -   In case you want to apply a kernel patch, do so now. Often a command like this will do the trick:
>
>         patch -p1 < ../proposed-fix.patch
>
>     If the `-p1` is actually needed, depends on how the patch was created; in case it does not apply thus try without it.
>
>     If you cloned the sources with git and anything goes sideways, run `git reset --hard` to undo any changes to the sources.
>
>     \[`details<patching>`{.interpreted-text role="ref"}\]
:::

::: {#tagging_sbs}
> -   If you patched your kernel or have one of the same version installed already, better add a unique tag to the one you are about to build:
>
>         echo "-proposed_fix" > localversion
>
>     Running `uname -r` under your kernel later will then print something like \'6.1-rc4-proposed_fix\'.
>
>     \[`details<tagging>`{.interpreted-text role="ref"}\]
>
> ::: {#configuration_sbs}
> -   Create the build configuration for your kernel based on an existing configuration.
>
>     If you already prepared such a \'.config\' file yourself, copy it to \~/linux/ and run `make olddefconfig`.
>
>     Use the same command, if your distribution or somebody else already tailored your running kernel to your or your hardware\'s needs: the make target \'olddefconfig\' will then try to use that kernel\'s .config as base.
>
>     Using this make target is fine for everybody else, too \-- but you often can save a lot of time by using this command instead:
>
>         yes "" | make localmodconfig
>
>     This will try to pick your distribution\'s kernel as base, but then disable modules for any features apparently superfluous for your setup. This will reduce the compile time enormously, especially if you are running an universal kernel from a commodity Linux distribution.
>
>     There is a catch: \'localmodconfig\' is likely to disable kernel features you did not use since you booted your Linux \-- like drivers for currently disconnected peripherals or a virtualization software not haven\'t used yet. You can reduce or nearly eliminate that risk with tricks the reference section outlines; but when building a kernel just for quick testing purposes it is often negligible if such features are missing. But you should keep that aspect in mind when using a kernel built with this make target, as it might be the reason why something you only use occasionally stopped working.
>
>     \[`details<configuration>`{.interpreted-text role="ref"}\]
> :::
:::

::: {#configmods_sbs}
> -   Check if you might want to or have to adjust some kernel configuration options:
>
> > -   Evaluate how you want to handle debug symbols. Enable them, if you later might need to decode a stack trace found for example in a \'panic\', \'Oops\', \'warning\', or \'BUG\'; on the other hand disable them, if you are short on storage space or prefer a smaller kernel binary. See the reference section for details on how to do either. If neither applies, it will likely be fine to simply not bother with this. \[`details<configmods_debugsymbols>`{.interpreted-text role="ref"}\]
> > -   Are you running Debian? Then to avoid known problems by performing additional adjustments explained in the reference section. \[`details<configmods_distros>`{.interpreted-text role="ref"}\].
> > -   If you want to influence the other aspects of the configuration, do so now by using make targets like \'menuconfig\' or \'xconfig\'. \[`details<configmods_individual>`{.interpreted-text role="ref"}\].
:::

::: {#build_sbs}
> -   Build the image and the modules of your kernel:
>
>         make -j $(nproc --all)
>
>     If you want your kernel packaged up as deb, rpm, or tar file, see the reference section for alternatives.
>
>     \[`details<build>`{.interpreted-text role="ref"}\]
:::

::: {#install_sbs}
> -   Now install your kernel:
>
>         command -v installkernel && sudo make modules_install install
>
>     Often all left for you to do afterwards is a `reboot`, as many commodity Linux distributions will then create an initramfs (also known as initrd) and an entry for your kernel in your bootloader\'s configuration; but on some distributions you have to take care of these two steps manually for reasons the reference section explains.
>
>     On a few distributions like Arch Linux and its derivatives the above command does nothing at all; in that case you have to manually install your kernel, as outlined in the reference section.
>
>     If you are running a immutable Linux distribution, check its documentation and the web to find out how to install your own kernel there.
>
>     \[`details<install>`{.interpreted-text role="ref"}\]
:::

::: {#another_sbs}
> -   To later build another kernel you need similar steps, but sometimes slightly different commands.
>
>     First, switch back into the sources tree:
>
>         cd ~/linux/
>
>     In case you want to build a version from a stable/longterm series you have not used yet (say 6.2.y), tell git to track it:
>
>         git remote set-branches --add origin linux-6.2.y
>
>     Now fetch the latest upstream changes; you again need to specify the earliest version you care about, as git otherwise might retrieve the entire commit history:
>
>         git fetch --shallow-exclude=v6.0 origin
>
>     Now switch to the version you are interested in \-- but be aware the command used here will discard any modifications you performed, as they would conflict with the sources you want to checkout:
>
>         git checkout --force --detach origin/master
>
>     At this point you might want to patch the sources again or set/modify a build tag, as explained earlier. Afterwards adjust the build configuration to the new codebase using olddefconfig, which will now adjust the configuration file you prepared earlier using localmodconfig (\~/linux/.config) for your next kernel:
>
>         # reminder: if you want to apply patches, do it at this point
>         # reminder: you might want to update your build tag at this point
>         make olddefconfig
>
>     Now build your kernel:
>
>         make -j $(nproc --all)
>
>     Afterwards install the kernel as outlined above:
>
>         command -v installkernel && sudo make modules_install install
>
>     \[`details<another>`{.interpreted-text role="ref"}\]
:::

::: {#uninstall_sbs}
> -   Your kernel is easy to remove later, as its parts are only stored in two places and clearly identifiable by the kernel\'s release name. Just ensure to not delete the kernel you are running, as that might render your system unbootable.
>
>     Start by deleting the directory holding your kernel\'s modules, which is named after its release name \-- \'6.0.1-foobar\' in the following example:
>
>         sudo rm -rf /lib/modules/6.0.1-foobar
>
>     Now try the following command, which on some distributions will delete all other kernel files installed while also removing the kernel\'s entry from the bootloader configuration:
>
>         command -v kernel-install && sudo kernel-install -v remove 6.0.1-foobar
>
>     If that command does not output anything or fails, see the reference section; do the same if any files named \'*6.0.1-foobar*\' remain in /boot/.
>
>     \[`details<uninstall>`{.interpreted-text role="ref"}\]
:::

::: {#submit_improvements}

Did you run into trouble following any of the above steps that is not cleared up by the reference section below? Or do you have ideas how to improve the text? Then please take a moment of your time and let the maintainer of this document know by email (Thorsten Leemhuis \<<linux@leemhuis.info>\>), ideally while CCing the Linux docs mailing list (<linux-doc@vger.kernel.org>). Such feedback is vital to improve this document further, which is in everybody\'s interest, as it will enable more people to master the task described here.

> 您是否在执行上述任何步骤时遇到了以下参考部分未解决的问题？或者你有如何改进文本的想法吗？然后请花点时间，通过电子邮件让本文档的维护者知道（Thorsten Leemhuis\<<linux@leemhuis.info>\>)，最好是在CCing Linux文档邮件列表时(<linux-doc@vger.kernel.org>). 这种反馈对于进一步改进本文档至关重要，这符合每个人的利益，因为它将使更多的人能够掌握此处描述的任务。
:::

# Reference section for the step-by-step guide


This section holds additional information for each of the steps in the above guide.

> 本节包含上述指南中每个步骤的附加信息。

## Prepare for emergencies {#backup}

> *Create a fresh backup and put system repair and restore tools at hand* \[`... <backup_sbs>`{.interpreted-text role="ref"}\]


Remember, you are dealing with computers, which sometimes do unexpected things \-- especially if you fiddle with crucial parts like the kernel of an operating system. That\'s what you are about to do in this process. Hence, better prepare for something going sideways, even if that should not happen.

> 记住，你所面对的是计算机，它们有时会做一些意想不到的事情，尤其是当你摆弄操作系统内核等关键部分时。这就是你在这个过程中要做的。因此，最好为横盘做好准备，即使这种情况不应该发生。


\[`back to step-by-step guide <backup_sbs>`{.interpreted-text role="ref"}\]

> \[`返回分步指南<backup_sbs>`｛.explored text role=“ref”｝\]

## Dealing with techniques like Secure Boot {#secureboot}

> *On platforms with \'Secure Boot\' or similar techniques, prepare everything to ensure the system will permit your self-compiled kernel to boot later.* \[`... <secureboot_sbs>`{.interpreted-text role="ref"}\]


Many modern systems allow only certain operating systems to start; they thus by default will reject booting self-compiled kernels.

> 许多现代系统只允许某些操作系统启动；因此，默认情况下，它们将拒绝引导自编译内核。


You ideally deal with this by making your platform trust your self-built kernels with the help of a certificate and signing. How to do that is not described here, as it requires various steps that would take the text too far away from its purpose; \'Documentation/admin-guide/module-signing.rst\' and various web sides already explain this in more detail.

> 理想情况下，您可以通过证书和签名的帮助，让您的平台信任您自己构建的内核来处理这一问题。这里没有描述如何做到这一点，因为它需要采取各种步骤，使文本偏离其目的太远文档/管理指南/模块签名.rst\'和各种网页已经对此进行了更详细的解释。


Temporarily disabling solutions like Secure Boot is another way to make your own Linux boot. On commodity x86 systems it is possible to do this in the BIOS Setup utility; the steps to do so are not described here, as they greatly vary between machines.

> 暂时禁用安全引导等解决方案是另一种让您自己启动Linux的方法。在商品x86系统上，可以在BIOS设置实用程序中执行此操作；这里没有描述这样做的步骤，因为它们在不同的机器之间有很大的差异。


On mainstream x86 Linux distributions there is a third and universal option: disable all Secure Boot restrictions for your Linux environment. You can initiate this process by running `mokutil --disable-validation`; this will tell you to create a one-time password, which is safe to write down. Now restart; right after your BIOS performed all self-tests the bootloader Shim will show a blue box with a message \'Press any key to perform MOK management\'. Hit some key before the countdown exposes. This will open a menu and choose \'Change Secure Boot state\' there. Shim\'s \'MokManager\' will now ask you to enter three randomly chosen characters from the one-time password specified earlier. Once you provided them, confirm that you really want to disable the validation. Afterwards, permit MokManager to reboot the machine.

> 在主流x86 Linux发行版上，有第三个通用选项：禁用Linux环境的所有安全引导限制。您可以通过运行“mokutil--禁用验证”来启动此过程；这将告诉你创建一个一次性密码，这样写下来是安全的。现在重新启动；BIOS执行完所有自检后，引导加载程序Shim将显示一个蓝色框，并显示消息“按任意键执行MOK管理”。在倒计时曝光之前按下一些键。这将打开一个菜单，并在其中选择“更改安全启动状态”。Shim\的“MokManager\”现在将要求您从前面指定的一次性密码中随机选择三个字符。一旦您提供了它们，请确认您确实要禁用验证。之后，允许MokManager重新启动机器。


\[`back to step-by-step guide <secureboot_sbs>`{.interpreted-text role="ref"}\]

> \[`返回分步指南<secureboot_sbs>`｛.explored text role=“ref”｝\]

## Install build requirements {#buildrequires}

> *Install all software required to build a Linux kernel.* \[`...<buildrequires_sbs>`{.interpreted-text role="ref"}\]


The kernel is pretty stand-alone, but besides tools like the compiler you will sometimes need a few libraries to build one. How to install everything needed depends on your Linux distribution and the configuration of the kernel you are about to build.

> 内核是非常独立的，但除了编译器等工具之外，有时还需要一些库来构建一个。如何安装所需的一切取决于您的Linux发行版和即将构建的内核的配置。


Here are a few examples what you typically need on some mainstream distributions:

> 以下是一些主流发行版通常需要的示例：

> -   Debian, Ubuntu, and derivatives:
>
>         sudo apt install bc binutils bison dwarves flex gcc git make openssl \
>           pahole perl-base libssl-dev libelf-dev
>
> -   Fedora and derivatives:
>
>         sudo dnf install binutils /usr/include/{libelf.h,openssl/pkcs7.h} \
>           /usr/bin/{bc,bison,flex,gcc,git,openssl,make,perl,pahole}
>
> -   openSUSE and derivatives:
>
>         sudo zypper install bc binutils bison dwarves flex gcc git make perl-base \
>           openssl openssl-devel libelf-dev


In case you wonder why these lists include openssl and its development headers: they are needed for the Secure Boot support, which many distributions enable in their kernel configuration for x86 machines.

> 如果你想知道为什么这些列表包括openssl及其开发头：它们是安全引导支持所必需的，许多发行版在x86机器的内核配置中都支持安全引导。


Sometimes you will need tools for compression formats like bzip2, gzip, lz4, lzma, lzo, xz, or zstd as well.

> 有时您还需要用于压缩格式的工具，如bzip2、gzip、lz4、lzma、lzo、xz或zstd。


You might need additional libraries and their development headers in case you perform tasks not covered in this guide. For example, zlib will be needed when building kernel tools from the tools/ directory; adjusting the build configuration with make targets like \'menuconfig\' or \'xconfig\' will require development headers for ncurses or Qt5.

> 您可能需要额外的库及其开发头，以防执行本指南中未涵盖的任务。例如，从tools/目录构建内核工具时需要zlib；使用诸如“enuconfig”或“xconfig”之类的make目标调整构建配置将需要ncurses或Qt5的开发头。


\[`back to step-by-step guide <buildrequires_sbs>`{.interpreted-text role="ref"}\]

> \[`返回分步指南<buildrequires_sbs>`｛.explored text role=“ref”｝\]

## Space requirements {#diskspace}

> *Ensure to have enough free space for building and installing Linux.* \[`... <diskspace_sbs>`{.interpreted-text role="ref"}\]


The numbers mentioned are rough estimates with a big extra charge to be on the safe side, so often you will need less.

> 提到的数字是粗略的估计，为了安全起见，需要额外支付大量费用，所以你通常需要更少的费用。


If you have space constraints, remember to read the reference section when you reach the `section about configuration adjustments' <configmods>`{.interpreted-text role="ref"}, as ensuring debug symbols are disabled will reduce the consumed disk space by quite a few gigabytes.

> 如果您有空间限制，请记得在到达“关于配置调整的部分”<configmods>`{.depreced text role=“ref”}时阅读参考部分，因为确保禁用调试符号将使消耗的磁盘空间减少相当多的GB。


\[`back to step-by-step guide <diskspace_sbs>`{.interpreted-text role="ref"}\]

> \[`返回分步指南<diskspace_sbs>`｛.explored text role=“ref”｝\]

## Download the sources {#sources}

> *Retrieve the sources of the Linux version you intend to build.* \[`...<sources_sbs>`{.interpreted-text role="ref"}\]


The step-by-step guide outlines how to retrieve Linux\' sources using a shallow git clone. There is `more to tell about this method<sources_shallow>`{.interpreted-text role="ref"} and two alternate ways worth describing: `packaged archives<sources_archive>`{.interpreted-text role="ref"} and `a full git clone<sources_full>`{.interpreted-text role="ref"}. And the aspects \'`wouldn't it

> 该分步指南概述了如何使用浅git克隆检索Linux的源代码。关于这种方法<sources_shall>`｛.explored text role=“ref”｝有更多内容可供介绍，还有两种替代方法值得描述：`packaged archives<sources_archive>`{.explered text rol=“ref”}和`a full git clone<sources_full>`{。还有那些方面，不是吗
be wiser to use a proper pre-release than the latest mainline code
<sources_snapshot>`{.interpreted-text role="ref"}\' and \'`how to get an even fresher mainline codebase
<sources_fresher>`{.interpreted-text role="ref"}\' need elaboration, too.


Note, to keep things simple the commands used in this guide store the build artifacts in the source tree. If you prefer to separate them, simply add something like `O=~/linux-builddir/` to all make calls; also adjust the path in all commands that add files or modify any generated (like your \'.config\').

> 注意，为了简单起见，本指南中使用的命令将构建工件存储在源树中。如果您喜欢将它们分开，只需在所有make调用中添加类似“O=~/linux-builddir/”的内容；还可以调整所有添加文件或修改生成文件的命令中的路径（如“.config”）。


\[`back to step-by-step guide <sources_sbs>`{.interpreted-text role="ref"}\]

> \[`返回分步指南<sources_sbs>`｛.explored text role=“ref”｝\]

### Noteworthy characteristics of shallow clones {#sources_shallow}


The step-by-step guide uses a shallow clone, as it is the best solution for most of this document\'s target audience. There are a few aspects of this approach worth mentioning:

> 分步指南使用浅层克隆，因为它是本文档大多数目标受众的最佳解决方案。这种方法有几个方面值得一提：

> -   This document in most places uses `git fetch` with `--shallow-exclude=` to specify the earliest version you care about (or to be precise: its git tag). You alternatively can use the parameter `--shallow-since=` to specify an absolute (say `'2023-07-15'`) or relative (`'12 months'`) date to define the depth of the history you want to download. As a second alternative, you can also specify a certain depth explicitly with a parameter like `--depth=1`, unless you add branches for stable/longterm kernels.
>
> -   When running `git fetch`, remember to always specify the oldest version, the time you care about, or an explicit depth as shown in the step-by-step guide. Otherwise you will risk downloading nearly the entire git history, which will consume quite a bit of time and bandwidth while also stressing the servers.
>
>     Note, you do not have to use the same version or date all the time. But when you change it over time, git will deepen or flatten the history to the specified point. That allows you to retrieve versions you initially thought you did not need \-- or it will discard the sources of older versions, for example in case you want to free up some disk space. The latter will happen automatically when using `--shallow-since=` or `--depth=`.
>
> -   Be warned, when deepening your clone you might encounter an error like \'fatal: error in object: unshallow cafecaca0c0dacafecaca0c0dacafecaca0c0da\'. In that case run `git repack -d` and try again\`\`
>
> -   In case you want to revert changes from a certain version (say Linux 6.3) or perform a bisection (v6.2..v6.3), better tell `git fetch` to retrieve objects up to three versions earlier (e.g. 6.0): `git describe` will then be able to describe most commits just like it would in a full git clone.


\[`back to step-by-step guide <sources_sbs>`{.interpreted-text role="ref"}\] \[`back to section intro <sources>`{.interpreted-text role="ref"}\]

> \[`返回分步指南<sources_sbs>`｛.explored text role=“ref”｝\]\[`返回介绍<sources>`{.explered text rol=“ref”}\]

### Downloading the sources using a packages archive {#sources_archive}


People new to compiling Linux often assume downloading an archive via the front-page of <https://kernel.org> is the best approach to retrieve Linux\' sources. It actually can be, if you are certain to build just one particular kernel version without changing any code. Thing is: you might be sure this will be the case, but in practice it often will turn out to be a wrong assumption.

> 刚开始编译Linux的人通常认为通过的首页下载存档<https://kernel.org>是检索Linux源代码的最佳方法。事实上，如果您确信只构建一个特定的内核版本而不更改任何代码，则可以这样做。问题是：你可能确信会是这样，但在实践中，这往往是一个错误的假设。


That\'s because when reporting or debugging an issue developers will often ask to give another version a try. They also might suggest temporarily undoing a commit with `git revert` or might provide various patches to try. Sometimes reporters will also be asked to use `git bisect` to find the change causing a problem. These things rely on git or are a lot easier and quicker to handle with it.

> 这是因为当报告或调试一个问题时，开发人员经常会要求尝试另一个版本。他们还可能建议使用“git-revert”临时撤消提交，或者可能提供各种补丁进行尝试。有时记者也会被要求使用“git平分线”来找出导致问题的变化。这些东西依赖于git，或者使用它更容易、更快。


A shallow clone also does not add any significant overhead. For example, when you use `git clone --depth=1` to create a shallow clone of the latest mainline codebase git will only retrieve a little more data than downloading the latest mainline pre-release (aka \'rc\') via the front-page of kernel.org would.

> 浅克隆也不会增加任何显著的开销。例如，当您使用“git clone--depth=1”创建最新主线代码库的浅层克隆时，git只会检索到比通过kernel.org首页下载最新主线预发行版（又名“rc”）多一点的数据。


A shallow clone therefore is often the better choice. If you nevertheless want to use a packaged source archive, download one via kernel.org; afterwards extract its content to some directory and change to the subdirectory created during extraction. The rest of the step-by-step guide will work just fine, apart from things that rely on git \-- but this mainly concerns the section on successive builds of other versions.

> 因此，浅层克隆往往是更好的选择。如果您仍然想使用打包的源归档文件，请通过kernel.org下载一个；然后将其内容提取到某个目录，并更改为提取过程中创建的子目录。除了依赖git的部分外，分步指南的其余部分也可以正常工作，但这主要涉及其他版本的后续构建部分。


\[`back to step-by-step guide <sources_sbs>`{.interpreted-text role="ref"}\] \[`back to section intro <sources>`{.interpreted-text role="ref"}\]

> \[`返回分步指南<sources_sbs>`｛.explored text role=“ref”｝\]\[`返回介绍<sources>`{.explered text rol=“ref”}\]

### Downloading the sources using a full git clone {#sources_full}


If downloading and storing a lot of data (\~4,4 Gigabyte as of early 2023) is nothing that bothers you, instead of a shallow clone perform a full git clone instead. You then will avoid the specialties mentioned above and will have all versions and individual commits at hand at any time:

> 如果下载和存储大量数据（截至2023年初，约4，4 GB）并没有什么困扰你的，那就执行一个完整的git克隆，而不是浅克隆。然后，您将避免上述特殊情况，并随时准备好所有版本和单个提交：

    curl -L \
      https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/clone.bundle \
      -o linux-stable.git.bundle
    git clone linux-stable.git.bundle ~/linux/
    rm linux-stable.git.bundle
    cd ~/linux/
    git remote set-url origin \
      https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git
    git fetch origin
    git checkout --detach origin/master


\[`back to step-by-step guide <sources_sbs>`{.interpreted-text role="ref"}\] \[`back to section intro <sources>`{.interpreted-text role="ref"}\]

> \[`返回分步指南<sources_sbs>`｛.explored text role=“ref”｝\]\[`返回介绍<sources>`{.explered text rol=“ref”}\]

### Proper pre-releases (RCs) vs. latest mainline {#sources_snapshot}


When cloning the sources using git and checking out origin/master, you often will retrieve a codebase that is somewhere between the latest and the next release or pre-release. This almost always is the code you want when giving mainline a shot: pre-releases like v6.1-rc5 are in no way special, as they do not get any significant extra testing before being published.

> 当使用git克隆源代码并检查origin/master时，您通常会检索到介于最新版本和下一版本或预发布版本之间的代码库。这几乎总是你在尝试主线时想要的代码：像v6.1-rc5这样的预发行版一点也不特别，因为它们在发布之前没有得到任何重要的额外测试。


There is one exception: you might want to stick to the latest mainline release (say v6.1) before its successor\'s first pre-release (v6.2-rc1) is out. That is because compiler errors and other problems are more likely to occur during this time, as mainline then is in its \'merge window\': a usually two week long phase, in which the bulk of the changes for the next release is merged.

> 有一个例外：在其继任者的第一个预发布版本（v6.2-rc1）发布之前，您可能希望坚持使用最新的主线版本（比如v6.1）。这是因为在这段时间内，编译器错误和其他问题更有可能发生，因为主线正处于“合并窗口”：这是一个通常为期两周的阶段，在这个阶段，下一个版本的大部分更改都会合并。


\[`back to step-by-step guide <sources_sbs>`{.interpreted-text role="ref"}\] \[`back to section intro <sources>`{.interpreted-text role="ref"}\]

> \[`返回分步指南<sources_sbs>`｛.explored text role=“ref”｝\]\[`返回介绍<sources>`{.explered text rol=“ref”}\]

### Avoiding the mainline lag {#sources_fresher}


The explanations for both the shallow clone and the full clone both retrieve the code from the Linux stable git repository. That makes things simpler for this document\'s audience, as it allows easy access to both mainline and stable/longterm releases. This approach has just one downside:

> 浅克隆和完整克隆的解释都从Linux稳定的git存储库中检索代码。这使得本文档的受众可以更简单地访问主线版本和稳定/长期版本。这种方法只有一个缺点：


Changes merged into the mainline repository are only synced to the master branch of the Linux stable repository every few hours. This lag most of the time is not something to worry about; but in case you really need the latest code, just add the mainline repo as additional remote and checkout the code from there:

> 合并到主线存储库中的更改仅每隔几个小时同步到Linux稳定存储库的主分支。这种滞后在大多数情况下是不需要担心的；但如果你真的需要最新的代码，只需添加主线回购作为额外的远程代码，然后从那里签出代码：

    git remote add mainline \
      https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
    git fetch mainline
    git checkout --detach mainline/master


When doing this with a shallow clone, remember to call `git fetch` with one of the parameters described earlier to limit the depth.

> 当使用浅克隆执行此操作时，请记住使用前面描述的参数之一调用“git fetch”以限制深度。


\[`back to step-by-step guide <sources_sbs>`{.interpreted-text role="ref"}\] \[`back to section intro <sources>`{.interpreted-text role="ref"}\]

> \[`返回分步指南<sources_sbs>`｛.explored text role=“ref”｝\]\[`返回介绍<sources>`{.explered text rol=“ref”}\]

## Patch the sources (optional) {#patching}

> *In case you want to apply a kernel patch, do so now.* \[`...<patching_sbs>`{.interpreted-text role="ref"}\]


This is the point where you might want to patch your kernel \-- for example when a developer proposed a fix and asked you to check if it helps. The step-by-step guide already explains everything crucial here.

> 这就是您可能想要修补内核的地方——例如，当开发人员提出一个修复方案并要求您检查它是否有帮助时。分步指南已经解释了这里的所有关键内容。


\[`back to step-by-step guide <patching_sbs>`{.interpreted-text role="ref"}\]

> \[`返回分步指南<patching_sbs>`｛.explored text role=“ref”｝\]

## Tagging this kernel build (optional, often wise) {#tagging}

> *If you patched your kernel or already have that kernel version installed, better tag your kernel by extending its release name:* \[`...<tagging_sbs>`{.interpreted-text role="ref"}\]


Tagging your kernel will help avoid confusion later, especially when you patched your kernel. Adding an individual tag will also ensure the kernel\'s image and its modules are installed in parallel to any existing kernels.

> 标记内核将有助于避免以后出现混淆，尤其是在修补内核时。添加单独的标记还可以确保内核的映像及其模块与任何现有内核并行安装。


There are various ways to add such a tag. The step-by-step guide realizes one by creating a \'localversion\' file in your build directory from which the kernel build scripts will automatically pick up the tag. You can later change that file to use a different tag in subsequent builds or simply remove that file to dump the tag.

> 有多种方法可以添加这样的标签。分步指南通过在构建目录中创建一个“localversion”文件来实现这一点，内核构建脚本将从该文件中自动获取标记。您可以稍后更改该文件以在后续生成中使用不同的标记，也可以简单地删除该文件以转储标记。


\[`back to step-by-step guide <tagging_sbs>`{.interpreted-text role="ref"}\]

> \[`返回分步指南<tagging_sbs>`｛.explored text role=“ref”｝\]

## Define the build configuration for your kernel {#configuration}

> *Create the build configuration for your kernel based on an existing configuration.* \[`... <configuration_sbs>`{.interpreted-text role="ref"}\]


There are various aspects for this steps that require a more careful explanation:

> 此步骤有多个方面需要更仔细的解释：

### Pitfalls when using another configuration file as base


Make targets like localmodconfig and olddefconfig share a few common snares you want to be aware of:

> 让localmodconfig和olddefconfig这样的目标共享一些你想知道的常见陷阱：

> -   These targets will reuse a kernel build configuration in your build directory (e.g. \'\~/linux/.config\'), if one exists. In case you want to start from scratch you thus need to delete it.
> -   The make targets try to find the configuration for your running kernel automatically, but might choose poorly. A line like \'# using defaults found in /boot/config-6.0.7-250.fc36.x86_64\' or \'using config: \'/boot/config-6.0.7-250.fc36.x86_64\' tells you which file they picked. If that is not the intended one, simply store it as \'\~/linux/.config\' before using these make targets.
> -   Unexpected things might happen if you try to use a config file prepared for one kernel (say v6.0) on an older generation (say v5.15). In that case you might want to use a configuration as base which your distribution utilized when they used that or an slightly older kernel version.

### Influencing the configuration


The make target olddefconfig and the `yes "" |` used when utilizing localmodconfig will set any undefined build options to their default value. This among others will disable many kernel features that were introduced after your base kernel was released.

> 使用localmodconfig时使用的make-target-olddefconfig和“yes”“|`会将任何未定义的构建选项设置为默认值。这将禁用在基本内核发布后引入的许多内核功能。


If you want to set these configurations options manually, use `oldconfig` instead of `olddefconfig` or omit the `yes "" |` when utilizing localmodconfig. Then for each undefined configuration option you will be asked how to proceed. In case you are unsure what to answer, simply hit \'enter\' to apply the default value.

> 如果要手动设置这些配置选项，请使用“oldconfig”而不是“olddefconfig”，或者在使用localmodconfig时省略“yes”“|”。然后，对于每个未定义的配置选项，您将被问及如何继续。如果您不确定该回答什么，只需点击“输入”即可应用默认值。

### Big pitfall when using localmodconfig


As explained briefly in the step-by-step guide already: with localmodconfig it can easily happen that your self-built kernel will lack modules for tasks you did not perform before utilizing this make target. That\'s because those tasks require kernel modules that are normally autoloaded when you perform that task for the first time; if you didn\'t perform that task at least once before using localmodonfig, the latter will thus assume these modules are superfluous and disable them.

> 正如在逐步指南中简要解释的那样：使用localmodconfig，很容易发生的情况是，您自己构建的内核将缺少用于在使用此make目标之前没有执行的任务的模块。这是因为当你第一次执行任务时，这些任务需要通常自动加载的内核模块；如果您在使用localmodonfig之前没有执行该任务至少一次，那么后者会认为这些模块是多余的，并禁用它们。


You can try to avoid this by performing typical tasks that often will autoload additional kernel modules: start a VM, establish VPN connections, loop-mount a CD/DVD ISO, mount network shares (CIFS, NFS, \...), and connect all external devices (2FA keys, headsets, webcams, \...) as well as storage devices with file systems you otherwise do not utilize (btrfs, ext4, FAT, NTFS, XFS, \...). But it is hard to think of everything that might be needed \-- even kernel developers often forget one thing or another at this point.

> 您可以尝试通过执行通常会自动加载其他内核模块的典型任务来避免这种情况：启动虚拟机、建立VPN连接、循环装载CD/DVD ISO、装载网络共享（CIFS、NFS、\…），并将所有外部设备（2FA密钥、耳机、网络摄像头等）以及存储设备与您不使用的文件系统（btrfs、ext4、FAT、NTFS、XFS等）连接起来。但很难想到可能需要的一切——即使是内核开发人员也经常在这一点上忘记这样或那样的事情。


Do not let that risk bother you, especially when compiling a kernel only for testing purposes: everything typically crucial will be there. And if you forget something important you can turn on a missing feature later and quickly run the commands to compile and install a better kernel.

> 不要让这种风险困扰你，尤其是当编译内核只是为了测试目的时：通常至关重要的一切都会存在。如果你忘记了一些重要的东西，你可以稍后打开一个缺失的功能，并快速运行命令来编译和安装一个更好的内核。


But if you plan to build and use self-built kernels regularly, you might want to reduce the risk by recording which modules your system loads over the course of a few weeks. You can automate this with [modprobed-db](https://github.com/graysky2/modprobed-db). Afterwards use `LSMOD=<path>` to point localmodconfig to the list of modules modprobed-db noticed being used:

> 但是，如果您计划定期构建和使用自建内核，您可能希望通过记录几周内系统加载的模块来降低风险。您可以使用[modprobeddb]自动执行此操作(https://github.com/graysky2/modprobed-db). 然后使用“LSMOD=＜path＞”将localmodconfig指向modprobed db注意到正在使用的模块列表：

    yes "" | make LSMOD="${HOME}"/.config/modprobed.db localmodconfig

### Remote building with localmodconfig


If you want to use localmodconfig to build a kernel for another machine, run `lsmod > lsmod_foo-machine` on it and transfer that file to your build host. Now point the build scripts to the file like this: `yes "" | make LSMOD=~/lsmod_foo-machine localmodconfig`. Note, in this case you likely want to copy a base kernel configuration from the other machine over as well and place it as .config in your build directory.

> 如果您想使用localmodconfig为另一台机器构建内核，请在其上运行“lsmod>lsmod_foo-machine”，并将该文件传输到您的构建主机。现在将构建脚本指向如下文件：“yes”“|make LSMOD=~/LSMOD_foo-machine localmodconfig`。注意，在这种情况下，您可能也想从另一台机器上复制一个基本内核配置，并将其作为.config放在构建目录中。


\[`back to step-by-step guide <configuration_sbs>`{.interpreted-text role="ref"}\]

> \[`返回分步指南<configuration_sb>`｛.explored text role=“ref”｝\]

## Adjust build configuration {#configmods}

> *Check if you might want to or have to adjust some kernel configuration options:*


Depending on your needs you at this point might want or have to adjust some kernel configuration options.

> 根据您的需要，您现在可能想要或必须调整一些内核配置选项。

### Debug symbols {#configmods_debugsymbols}

> *Evaluate how you want to handle debug symbols.* \[`...<configmods_sbs>`{.interpreted-text role="ref"}\]


Most users do not need to care about this, it\'s often fine to leave everything as it is; but you should take a closer look at this, if you might need to decode a stack trace or want to reduce space consumption.

> 大多数用户不需要关心这一点，保持一切原样通常是可以的；但是，如果您可能需要解码堆栈跟踪或希望减少空间消耗，则应该仔细研究一下这一点。


Having debug symbols available can be important when your kernel throws a \'panic\', \'Oops\', \'warning\', or \'BUG\' later when running, as then you will be able to find the exact place where the problem occurred in the code. But collecting and embedding the needed debug information takes time and consumes quite a bit of space: in late 2022 the build artifacts for a typical x86 kernel configured with localmodconfig consumed around 5 Gigabyte of space with debug symbols, but less than 1 when they were disabled. The resulting kernel image and the modules are bigger as well, which increases load times.

> 当内核稍后在运行时抛出“panic”、“Oops”、“warning”或“BUG”时，具有可用的调试符号可能很重要，因为这样您就可以在代码中找到问题发生的确切位置。但是，收集和嵌入所需的调试信息需要时间并消耗相当大的空间：在2022年末，使用localmodconfig配置的典型x86内核的构建工件消耗了大约5 GB的调试符号空间，但当它们被禁用时，还不到1 GB。由此产生的内核映像和模块也更大，这增加了加载时间。


Hence, if you want a small kernel and are unlikely to decode a stack trace later, you might want to disable debug symbols to avoid above downsides:

> 因此，如果您想要一个小内核，并且以后不太可能解码堆栈跟踪，则可能需要禁用调试符号以避免上述缺点：

    ./scripts/config --file .config -d DEBUG_INFO \
      -d DEBUG_INFO_DWARF_TOOLCHAIN_DEFAULT -d DEBUG_INFO_DWARF4 \
      -d DEBUG_INFO_DWARF5 -e CONFIG_DEBUG_INFO_NONE
    make olddefconfig


You on the other hand definitely want to enable them, if there is a decent chance that you need to decode a stack trace later (as explained by \'Decode failure messages\' in Documentation/admin-guide/tainted-kernels.rst in more detail):

> 另一方面，如果您稍后需要解码堆栈跟踪，您肯定希望启用它们（如Documentation/admin guide/tainted-kernels.rst中的“恢复失败消息”所详细解释的）：

    ./scripts/config --file .config -d DEBUG_INFO_NONE -e DEBUG_KERNEL
      -e DEBUG_INFO -e DEBUG_INFO_DWARF_TOOLCHAIN_DEFAULT -e KALLSYMS -e KALLSYMS_ALL
    make olddefconfig


Note, many mainstream distributions enable debug symbols in their kernel configurations \-- make targets like localmodconfig and olddefconfig thus will often pick that setting up.

> 注意，许多主流发行版在其内核配置中启用调试符号\-使localmodconfig和olddefconfig等目标成为可能，因此通常会选择这种设置。


\[`back to step-by-step guide <configmods_sbs>`{.interpreted-text role="ref"}\]

> \[`返回分步指南<configmods_sbs>`｛.explored text role=“ref”｝\]

### Distro specific adjustments {#configmods_distros}

> *Are you running* \[`... <configmods_sbs>`{.interpreted-text role="ref"}\]


The following sections help you to avoid build problems that are known to occur when following this guide on a few commodity distributions.

> 以下部分可帮助您避免在遵循本指南了解一些商品发行版时出现的构建问题。

**Debian:**

> -   Remove a stale reference to a certificate file that would cause your build to fail:
>
>         ./scripts/config --file .config --set-str SYSTEM_TRUSTED_KEYS ''
>
>     Alternatively, download the needed certificate and make that configuration option point to it, as [the Debian handbook explains in more detail](https://debian-handbook.info/browse/stable/sect.kernel-compilation.html) \-- or generate your own, as explained in Documentation/admin-guide/module-signing.rst.


\[`back to step-by-step guide <configmods_sbs>`{.interpreted-text role="ref"}\]

> \[`返回分步指南<configmods_sbs>`｛.explored text role=“ref”｝\]

### Individual adjustments {#configmods_individual}

> *If you want to influence the other aspects of the configuration, do so now* \[`... <configmods_sbs>`{.interpreted-text role="ref"}\]


You at this point can use a command like `make menuconfig` to enable or disable certain features using a text-based user interface; to use a graphical configuration utilize, use the make target `xconfig` or `gconfig` instead. All of them require development libraries from toolkits they are based on (ncurses, Qt5, Gtk2); an error message will tell you if something required is missing.

> 此时，您可以使用类似“makemenuconfig”的命令，使用基于文本的用户界面启用或禁用某些功能；要使用图形配置，请使用make-target“xconfig”或“gconfig”。所有这些都需要来自它们所基于的工具包（ncurses，Qt5，Gtk2）的开发库；如果缺少所需的内容，则会显示一条错误消息。


\[`back to step-by-step guide <configmods_sbs>`{.interpreted-text role="ref"}\]

> \[`返回分步指南<configmods_sbs>`｛.explored text role=“ref”｝\]

## Build your kernel {#build}

> *Build the image and the modules of your kernel* \[`... <build_sbs>`{.interpreted-text role="ref"}\]


A lot can go wrong at this stage, but the instructions below will help you help yourself. Another subsection explains how to directly package your kernel up as deb, rpm or tar file.

> 在这个阶段可能会出现很多问题，但下面的说明将帮助您自助。另一小节解释了如何将内核直接打包为deb、rpm或tar文件。

### Dealing with build errors


When a build error occurs, it might be caused by some aspect of your machine\'s setup that often can be fixed quickly; other times though the problem lies in the code and can only be fixed by a developer. A close examination of the failure messages coupled with some research on the internet will often tell you which of the two it is. To perform such a investigation, restart the build process like this:

> 当生成错误发生时，它可能是由机器设置的某些方面引起的，这些方面通常可以快速修复；但有时问题在于代码，只能由开发人员解决。仔细检查故障消息，再加上互联网上的一些研究，通常会告诉你这是两者中的哪一个。要进行这样的调查，请重新启动构建过程，如下所示：

    make V=1


The `V=1` activates verbose output, which might be needed to see the actual error. To make it easier to spot, this command also omits the `-j $(nproc --all)` used earlier to utilize every CPU core in the system for the job \-- but this parallelism also results in some clutter when failures occur.

> “V=1”激活详细输出，这可能是查看实际错误所必需的。为了更容易发现，此命令还省略了先前用于将系统中的每个CPU核心用于作业的“-j$（nproc-all）”，但这种并行性在发生故障时也会导致一些混乱。


After a few seconds the build process should run into the error again. Now try to find the most crucial line describing the problem. Then search the internet for the most important and non-generic section of that line (say 4 to 8 words); avoid or remove anything that looks remotely system-specific, like your username or local path names like `/home/username/linux/`. First try your regular internet search engine with that string, afterwards search Linux kernel mailing lists via [lore.kernel.org/all/](https://lore.kernel.org/all/).

> 几秒钟后，构建过程应该会再次出现错误。现在试着找出描述问题的最关键的一行。然后在互联网上搜索该行中最重要和非通用的部分（比如4到8个单词）；避免或删除任何看起来远程系统特定的内容，如用户名或本地路径名，如“/home/username/linux/”。首先尝试使用该字符串的常规互联网搜索引擎，然后通过[lore.kernel.org/all/]搜索Linux内核邮件列表(https://lore.kernel.org/all/).


This most of the time will find something that will explain what is wrong; quite often one of the hits will provide a solution for your problem, too. If you do not find anything that matches your problem, try again from a different angle by modifying your search terms or using another line from the error messages.

> 这在大多数时候都会找到一些东西来解释问题所在；通常，其中一个命中也会为您的问题提供解决方案。如果找不到与您的问题相匹配的内容，请通过修改搜索词或使用错误消息中的另一行，从其他角度重试。


In the end, most trouble you are to run into has likely been encountered and reported by others already. That includes issues where the cause is not your system, but lies the code. If you run into one of those, you might thus find a solution (e.g. a patch) or workaround for your problem, too.

> 最后，你遇到的大多数麻烦很可能已经被其他人遇到并报告了。这包括问题，其中的原因不是您的系统，而是代码。如果你遇到其中一个，你可能也会找到解决问题的方法（例如补丁）或变通方法。

### Package your kernel up


The step-by-step guide uses the default make targets (e.g. \'bzImage\' and \'modules\' on x86) to build the image and the modules of your kernel, which later steps of the guide then install. You instead can also directly build everything and directly package it up by using one of the following targets:

> 分步指南使用默认的make目标（例如x86上的\'bzImage\'和\'modules\'）来构建内核的映像和模块，然后在指南的后面步骤中安装这些映像和模块。相反，您也可以使用以下目标之一直接构建所有内容并直接打包：

> -   `make -j $(nproc --all) bindeb-pkg` to generate a deb package
> -   `make -j $(nproc --all) binrpm-pkg` to generate a rpm package
> -   `make -j $(nproc --all) tarbz2-pkg` to generate a bz2 compressed tarball


This is just a selection of available make targets for this purpose, see `make help` for others. You can also use these targets after running `make -j $(nproc --all)`, as they will pick up everything already built.

> 这只是为此目的选择的可用make目标，请参阅“make help”for others。您也可以在运行“make-j$（nproc-all）”后使用这些目标，因为它们将获取所有已构建的内容。


If you employ the targets to generate deb or rpm packages, ignore the step-by-step guide\'s instructions on installing and removing your kernel; instead install and remove the packages using the package utility for the format (e.g. dpkg and rpm) or a package management utility build on top of them (apt, aptitude, dnf/yum, zypper, \...). Be aware that the packages generated using these two make targets are designed to work on various distributions utilizing those formats, they thus will sometimes behave differently than your distribution\'s kernel packages.

> 如果您使用目标来生成deb或rpm包，请忽略分步指南中关于安装和删除内核的说明；相反，使用格式的包实用程序（例如dpkg和rpm）或在其上构建的包管理实用程序（apt、aptitude、dnf/yum、zypper、\…）来安装和删除包。请注意，使用这两个make目标生成的包设计用于使用这些格式的各种发行版，因此，它们有时的行为与您的发行版的内核包不同。


\[`back to step-by-step guide <build_sbs>`{.interpreted-text role="ref"}\]

> \[`返回分步指南<build_sbs>`｛.explored text role=“ref”｝\]

## Install your kernel {#install}

> *Now install your kernel* \[`... <install_sbs>`{.interpreted-text role="ref"}\]


What you need to do after executing the command in the step-by-step guide depends on the existence and the implementation of an `installkernel` executable. Many commodity Linux distributions ship such a kernel installer in `/sbin/` that does everything needed, hence there is nothing left for you except rebooting. But some distributions contain an installkernel that does only part of the job \-- and a few lack it completely and leave all the work to you.

> 执行分步指南中的命令后需要做什么取决于“installkernel”可执行文件的存在和实现。许多商品Linux发行版都在“/sbin/”中提供了这样一个内核安装程序，它可以完成所需的一切，因此除了重新启动之外，什么都没有了。但有些发行版包含的installkernel只完成了部分工作，而有些发行版则完全没有它，将所有工作留给您。


If `installkernel` is found, the kernel\'s build system will delegate the actual installation of your kernel\'s image and related files to this executable. On almost all Linux distributions it will store the image as \'/boot/vmlinuz-\<your kernel\'s release name\>\' and put a \'System.map-\<your kernel\'s release name\>\' alongside it. Your kernel will thus be installed in parallel to any existing ones, unless you already have one with exactly the same release name.

> 如果找到“installkernel”，内核的构建系统将把内核映像和相关文件的实际安装委托给此可执行文件。在几乎所有的Linux发行版上，它都会将映像存储为\'/boot/vmlinuz-\<yourkernel\'sreleasename\>\'，并在旁边放一个\'System.map-\<your kernel\'s releasename\>\'。因此，您的内核将与任何现有的内核并行安装，除非您已经有一个具有完全相同版本名称的内核。


Installkernel on many distributions will afterwards generate an \'initramfs\' (often also called \'initrd\'), which commodity distributions rely on for booting; hence be sure to keep the order of the two make targets used in the step-by-step guide, as things will go sideways if you install your kernel\'s image before its modules. Often installkernel will then add your kernel to the bootloader configuration, too. You have to take care of one or both of these tasks yourself, if your distributions installkernel doesn\'t handle them.

> 许多发行版上的Installkernel随后会生成一个“itramfs”（通常也称为“itrd”），商品发行版依赖于它进行引导；因此，一定要保持分步指南中使用的两个make目标的顺序，因为如果在模块之前安装内核的映像，事情会发生问题。通常installkernel也会将您的内核添加到引导加载程序配置中。如果您的发行版installkernel无法处理这些任务，您必须自己处理其中一个或两个任务。


A few distributions like Arch Linux and its derivatives totally lack an installkernel executable. On those just install the modules using the kernel\'s build system and then install the image and the System.map file manually:

> 像Arch Linux及其衍生产品这样的少数发行版完全缺乏installkernel可执行文件。在这些模块上，只需使用内核的构建系统安装模块，然后手动安装映像和system.map文件：

    sudo make modules_install
    sudo install -m 0600 $(make -s image_name) /boot/vmlinuz-$(make -s kernelrelease)
    sudo install -m 0600 System.map /boot/System.map-$(make -s kernelrelease)


If your distribution boots with the help of an initramfs, now generate one for your kernel using the tools your distribution provides for this process. Afterwards add your kernel to your bootloader configuration and reboot.

> 如果您的发行版是在initramfs的帮助下启动的，那么现在使用发行版为该过程提供的工具为内核生成一个。然后将内核添加到引导程序配置中，然后重新启动。


\[`back to step-by-step guide <install_sbs>`{.interpreted-text role="ref"}\]

> \[`返回分步指南<install_sbs>`｛.explored text role=“ref”｝\]

## Another round later {#another}

> *To later build another kernel you need similar, but sometimes slightly different commands* \[`... <another_sbs>`{.interpreted-text role="ref"}\]


The process to build later kernels is similar, but at some points slightly different. You for example do not want to use \'localmodconfig\' for succeeding kernel builds, as you already created a trimmed down configuration you want to use from now on. Hence instead just use `oldconfig` or `olddefconfig` to adjust your build configurations to the needs of the kernel version you are about to build.

> 构建后续内核的过程是相似的，但在某些方面略有不同。例如，您不想在后续的内核构建中使用“localmodconfig”，因为您已经创建了一个从现在起要使用的精简配置。因此，只需使用“oldconfig”或“olddefconfig”来根据即将构建的内核版本的需要调整构建配置。

If you created a shallow-clone with git, remember what the `section that

explained the setup described in more detail <sources>`{.interpreted-text role="ref"}: you need to use a slightly different `git fetch` command and when switching to another series need to add an additional remote branch.

> 更详细地解释了<sources>`{.expreted text role=“ref”}中描述的设置：您需要使用稍微不同的`git fetch`命令，当切换到另一个系列时，需要添加一个额外的远程分支。


\[`back to step-by-step guide <another_sbs>`{.interpreted-text role="ref"}\]

> \[`返回分步指南<another_sbs>`｛.explored text role=“ref”｝\]

## Uninstall the kernel later {#uninstall}

> *All parts of your installed kernel are identifiable by its release name and thus easy to remove later.* \[`... <uninstall_sbs>`{.interpreted-text role="ref"}\]


Do not worry installing your kernel manually and thus bypassing your distribution\'s packaging system will totally mess up your machine: all parts of your kernel are easy to remove later, as files are stored in two places only and normally identifiable by the kernel\'s release name.

> 不要担心手动安装内核，从而绕过发行版的打包系统会完全打乱你的机器：内核的所有部分稍后都很容易删除，因为文件只存储在两个地方，通常可以通过内核的发布名称进行识别。


One of the two places is a directory in /lib/modules/, which holds the modules for each installed kernel. This directory is named after the kernel\'s release name; hence, to remove all modules for one of your kernels, simply remove its modules directory in /lib/modules/.

> 这两个位置之一是/lib/modules/中的一个目录，其中包含每个已安装内核的模块。该目录以内核的发布名称命名；因此，要删除其中一个内核的所有模块，只需删除其在/lib/modules/中的模块目录即可。


The other place is /boot/, where typically one to five files will be placed during installation of a kernel. All of them usually contain the release name in their file name, but how many files and their name depends somewhat on your distribution\'s installkernel executable (`see above <install>`{.interpreted-text role="ref"}) and its initramfs generator. On some distributions the `kernel-install` command mentioned in the step-by-step guide will remove all of these files for you \--and the entry for your kernel in the bootloader configuration at the same time, too. On others you have to take care of these steps yourself. The following command should interactively remove the two main files of a kernel with the release name \'6.0.1-foobar\':

> 另一个地方是/boot/，在安装内核的过程中，通常会放置一到五个文件。所有这些文件的文件名中通常都包含发行版名称，但文件的数量和名称在一定程度上取决于您的发行版的installkernel可执行文件（参见上面的＜install＞）及其initramfs生成器。在某些发行版上，分步指南中提到的“kernel install”命令将为您删除所有这些文件\-同时在引导加载程序配置中删除内核的条目。在其他人身上，你必须自己照顾好这些步骤。以下命令应以交互方式删除发布名为“6.0.1-foobar”的内核的两个主文件：

    rm -i /boot/{System.map,vmlinuz}-6.0.1-foobar


Now remove the belonging initramfs, which often will be called something like `/boot/initramfs-6.0.1-foobar.img` or `/boot/initrd.img-6.0.1-foobar`. Afterwards check for other files in /boot/ that have \'6.0.1-foobar\' in their name and delete them as well. Now remove the kernel from your bootloader\'s configuration.

> 现在删除所属的initramfs，这些initramfs通常被称为“/boot/initramfs-6.0.1-foobar.img”或“/boot/initrd.img-60.0.1-foobar”。然后检查/boot/中名称中有“6.0.1-foobar\”的其他文件，并将其删除。现在从引导加载程序的配置中删除内核。


Note, be very careful with wildcards like \'\*\' when deleting files or directories for kernels manually: you might accidentally remove files of a 6.0.11 kernel when all you want is to remove 6.0 or 6.0.1.

> 请注意，手动删除内核的文件或目录时，要非常小心使用“\*\”等通配符：当您只想删除6.0或6.0.1时，您可能会意外删除6.0.11内核的文件。


\[`back to step-by-step guide <uninstall_sbs>`{.interpreted-text role="ref"}\]

> \[`返回分步指南<uninstall_sbs>`｛.explored text role=“ref”｝\]

# FAQ

## Why does this \'how-to\' not work on my system?


As initially stated, this guide is \'designed to cover everything typically needed \[to build a kernel\] on mainstream Linux distributions running on commodity PC or server hardware\'. The outlined approach despite this should work on many other setups as well. But trying to cover every possible use-case in one guide would defeat its purpose, as without such a focus you would need dozens or hundreds of constructs along the lines of \'in case you are having \<insert machine or distro\>, you at this point have to do \<this and that\> \<instead\|additionally\>\'. Each of which would make the text longer, more complicated, and harder to follow.

> 正如最初所说，本指南“旨在涵盖在商品PC或服务器硬件上运行的主流Linux发行版上构建内核所需的一切”。尽管如此，概述的方法也应该适用于许多其他设置。但是，试图在一个指南中涵盖每一个可能的用例将无法达到其目的，因为如果没有这样的重点，你将需要几十个或数百个结构，就像“如果你有插入机器或发行版，那么在这一点上，你必须做这个和那个”，而不是“额外地”。每一个都会使文本更长、更复杂、更难理解。


That being said: this of course is a balancing act. Hence, if you think an additional use-case is worth describing, suggest it to the maintainers of this document, as `described above <submit_improvements>`{.interpreted-text role="ref"}.

> 话虽如此：这当然是一种平衡行为。因此，如果您认为有一个额外的用例值得描述，请向本文档的维护人员建议，如“上文<submit_improvements>`｛.explored text role=“ref”｝所述”。
