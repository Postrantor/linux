---
tip: translate by baidu@2024-01-30 21:36:17
---
---
title: Mono(tm) Binary Kernel Support for Linux
---


To configure Linux to automatically execute Mono-based .NET binaries (in the form of .exe files) without the need to use the mono CLR wrapper, you can use the BINFMT_MISC kernel support.

> 将Linux配置为自动执行基于Mono的。NET二进制文件（以.exe文件的形式），无需使用mono CLR包装器，即可使用BINFMT_MISC内核支持。


This will allow you to execute Mono-based .NET binaries just like any other program after you have done the following:

> 这将允许您执行基于Mono的。NET二进制文件，就像完成以下操作后的任何其他程序一样：


1)  You MUST FIRST install the Mono CLR support, either by downloading a binary package, a source tarball or by installing from Git. Binary packages for several distributions can be found at:

> 1） 您必须首先通过下载二进制包、源代码tarball或从Git安装Mono CLR支持。几个发行版的二进制包可以在以下位置找到：

    > <https://www.mono-project.com/download/>

    Instructions for compiling Mono can be found at:

    > <https://www.mono-project.com/docs/compiling-mono/linux/>

    Once the Mono CLR support has been installed, just check that `/usr/bin/mono` (which could be located elsewhere, for example `/usr/local/bin/mono`) is working.


2)  You have to compile BINFMT_MISC either as a module or into the kernel (`CONFIG_BINFMT_MISC`) and set it up properly. If you choose to compile it as a module, you will have to insert it manually with modprobe/insmod, as kmod cannot be easily supported with binfmt_misc. Read the file `binfmt_misc.txt` in this directory to know more about the configuration process.

> 2） 您必须将BINFMT_MISC编译为模块或编译到内核（`CONFIG_BINFMT_MISC`）中，并正确设置它。如果您选择将其编译为模块，则必须使用modprobe/insmod手动插入，因为binfmt_misc无法轻松支持kmod。阅读此目录中的文件“binfmt_misc.txt”以了解有关配置过程的更多信息。


3)  Add the following entries to `/etc/rc.local` or similar script to be run at system startup:

> 3） 将以下条目添加到“/etc/rc.local”或类似脚本中，以便在系统启动时运行：

    ``` sh
    # Insert BINFMT_MISC module into the kernel
    if [ ! -e /proc/sys/fs/binfmt_misc/register ]; then
        /sbin/modprobe binfmt_misc
    # Some distributions, like Fedora Core, perform
    # the following command automatically when the
    # binfmt_misc module is loaded into the kernel
    # or during normal boot up (systemd-based systems).
    # Thus, it is possible that the following line
    # is not needed at all.
    mount -t binfmt_misc none /proc/sys/fs/binfmt_misc
    fi

    # Register support for .NET CLR binaries
    if [ -e /proc/sys/fs/binfmt_misc/register ]; then
    # Replace /usr/bin/mono with the correct pathname to
    # the Mono CLR runtime (usually /usr/local/bin/mono
    # when compiling from sources or CVS).
        echo ':CLR:M::MZ::/usr/bin/mono:' > /proc/sys/fs/binfmt_misc/register
    else
        echo "No binfmt_misc support"
        exit 1
    fi
    ```


4)  Check that `.exe` binaries can be ran without the need of a wrapper script, simply by launching the `.exe` file directly from a command prompt, for example:

> 4） 检查“.exe”二进制文件是否可以在不需要包装脚本的情况下运行，只需从命令提示符直接启动“.exe”文件即可，例如：

        /usr/bin/xsd.exe

    ::: note
    ::: title
    Note
    :::

    If this fails with a permission denied error, check that the `.exe` file has execute permissions.
    :::
