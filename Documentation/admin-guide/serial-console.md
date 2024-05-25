---
tip: translate by baidu@2024-01-30 21:39:47
---
---
title: Linux Serial Console
---


To use a serial port as console you need to compile the support into your kernel - by default it is not compiled in. For PC style serial ports it\'s the config option next to menu option:

> 要使用串行端口作为控制台，您需要将支持编译到内核中-默认情况下，它不会在中编译。对于PC风格的串行端口，它是菜单选项旁边的配置选项：

`Character devices --> Serial drivers --> 8250/16550 and compatible serial support --> Console on 8250/16550 and compatible serial port`{.interpreted-text role="menuselection"}

You must compile serial support into the kernel and not as a module.


It is possible to specify multiple devices for console output. You can define a new kernel command line option to select which device(s) to use for console output.

> 可以为控制台输出指定多个设备。您可以定义一个新的内核命令行选项来选择要用于控制台输出的设备。

The format of this option is:

    console=device,options

    device:     tty0 for the foreground virtual console
            ttyX for any other virtual console
            ttySx for a serial port
            lp0 for the first parallel port
            ttyUSB0 for the first USB serial device

    options:    depend on the driver. For the serial port this
            defines the baudrate/parity/bits/flow control of
            the port, in the format BBBBPNF, where BBBB is the
            speed, P is parity (n/o/e), N is number of bits,
            and F is flow control ('r' for RTS). Default is
            9600n8. The maximum baudrate is 115200.

You can specify multiple console= options on the kernel command line.


The behavior is well defined when each device type is mentioned only once. In this case, the output will appear on all requested consoles. And the last device will be used when you open `/dev/console`. So, for example:

> 当每种设备类型只提到一次时，行为就得到了很好的定义。在这种情况下，输出将出现在所有请求的控制台上。当您打开“/dev/console”时，将使用最后一个设备。因此，例如：

    console=ttyS1,9600 console=tty0


defines that opening `/dev/console` will get you the current foreground virtual console, and kernel messages will appear on both the VGA console and the 2nd serial port (ttyS1 or COM2) at 9600 baud.

> 定义打开“/dev/console”将获得当前的前台虚拟控制台，并且内核消息将以9600波特出现在VGA控制台和第二个串行端口（ttyS1或COM2）上。


The behavior is more complicated when the same device type is defined more times. In this case, there are the following two rules:

> 当同一设备类型被定义更多次时，行为会更加复杂。在这种情况下，有以下两条规则：

1.  The output will appear only on the first device of each defined type.


2.  `/dev/console` will be associated with the first registered device. Where the registration order depends on how kernel initializes various subsystems.

> 2.“/dev/console”将与第一个注册的设备相关联。注册顺序取决于内核如何初始化各个子系统。

    This rule is used also when the last console= parameter is not used for other reasons. For example, because of a typo or because the hardware is not available.


The result might be surprising. For example, the following two command lines have the same result:

> 结果可能令人惊讶。例如，以下两个命令行具有相同的结果：

    console=ttyS1,9600 console=tty0 console=tty1
    console=tty0 console=ttyS1,9600 console=tty1


The kernel messages are printed only on `tty0` and `ttyS1`. And `/dev/console` gets associated with `tty0`. It is because kernel tries to register graphical consoles before serial ones. It does it because of the default behavior when no console device is specified, see below.

> 内核消息仅打印在“tty0”和“ttyS1”上。并且“/dev/console”与“tty0”关联。这是因为内核试图在串行控制台之前注册图形控制台。它这样做是因为在未指定控制台设备时的默认行为，请参见下文。


Note that the last `console=tty1` parameter still makes a difference. The kernel command line is used also by systemd. It would use the last defined `tty1` as the login console.

> 请注意，最后一个“console=tty1”参数仍然会产生影响。systemd也使用内核命令行。它将使用最后定义的“tty1”作为登录控制台。


If no console device is specified, the first device found capable of acting as a system console will be used. At this time, the system first looks for a VGA card and then for a serial port. So if you don\'t have a VGA card in your system the first serial port will automatically become the console.

> 如果未指定控制台设备，则将使用第一个能够充当系统控制台的设备。此时，系统首先寻找VGA卡，然后寻找串行端口。因此，如果您的系统中没有VGA卡，第一个串行端口将自动成为控制台。


You will need to create a new device to use `/dev/console`. The official `/dev/console` is now character device 5,1.

> 您将需要创建一个新设备来使用“/dev/console”。官方的“/dev/console”现在是字符设备5,1。


(You can also use a network device as a console. See `Documentation/networking/netconsole.rst` for information on that.)

> （您也可以将网络设备用作控制台。有关此方面的信息，请参阅“Documentation/networking/netconsole.rst”。）


Here\'s an example that will use `/dev/ttyS1` (COM2) as the console. Replace the sample values as needed.

> 下面是一个使用“/dev/ttyS1”（COM2）作为控制台的示例。根据需要替换采样值。


1.  Create `/dev/console` (real console) and `/dev/tty0` (master virtual console):

> 1.创建“/dev/console”（真实控制台）和“/dev/tty0”（主虚拟控制台）：

        cd /dev
        rm -f console tty0
        mknod -m 622 console c 5 1
        mknod -m 622 tty0 c 4 0


2.  LILO can also take input from a serial device. This is a very useful option. To tell LILO to use the serial port: In lilo.conf (global section):

> 2.LILO还可以从串行设备获取输入。这是一个非常有用的选项。要告诉LILO使用串行端口：在LILO.conf（全局部分）中：

        serial  = 1,9600n8 (ttyS1, 9600 bd, no parity, 8 bits)


3.  Adjust to kernel flags for the new kernel, again in lilo.conf (kernel section):

> 3.再次在lilo.conf（内核部分）中调整到新内核的内核标志：

        append = "console=ttyS1,9600"


4.  Make sure a getty runs on the serial port so that you can login to it once the system is done booting. This is done by adding a line like this to `/etc/inittab` (exact syntax depends on your getty):

> 4.确保getty在串行端口上运行，以便在系统启动后可以登录到它。这是通过在“/etc/inittab”中添加这样一行来完成的（确切的语法取决于getty）：

        S1:23:respawn:/sbin/getty -L ttyS1 9600 vt100

5.  Init and `/etc/ioctl.save`

    Sysvinit remembers its stty settings in a file in `/etc`, called `/etc/ioctl.save`. REMOVE THIS FILE before using the serial console for the first time, because otherwise init will probably set the baudrate to 38400 (baudrate of the virtual console).


6.  `/dev/console` and X Programs that want to do something with the virtual console usually open `/dev/console`. If you have created the new `/dev/console` device, and your console is NOT the virtual console some programs will fail. Those are programs that want to access the VT interface, and use `/dev/console instead of /dev/tty0`. Some of those programs are:

> 6.“/dev/console”和X要使用虚拟控制台执行某些操作的程序通常会打开“/dev/consel”。如果您已经创建了新的“/dev/console”设备，并且您的控制台不是虚拟控制台，则某些程序将失败。这些程序希望访问VT接口，并使用“/dev/console而不是/dev/tty0”。其中一些项目是：

        Xfree86, svgalib, gpm, SVGATextMode

    It should be fixed in modern versions of these programs though.

    Note that if you boot without a `console=` option (or with `console=/dev/tty0`), `/dev/console` is the same as `/dev/tty0`. In that case everything will still work.

7.  Thanks

    Thanks to Geert Uytterhoeven \<<geert@linux-m68k.org>\> for porting the patches from 2.1.4x to 2.1.6x for taking care of the integration of these patches into m68k, ppc and alpha.

Miquel van Smoorenburg \<<miquels@cistron.nl>\>, 11-Jun-2000
