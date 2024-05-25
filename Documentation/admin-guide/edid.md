---
tip: translate by baidu@2024-01-30 21:33:36
---
---
title: EDID
---


In the good old days when graphics parameters were configured explicitly in a file called xorg.conf, even broken hardware could be managed.

> 在过去的好日子里，图形参数是在一个名为xorg.conf的文件中显式配置的，即使是损坏的硬件也可以管理。


Today, with the advent of Kernel Mode Setting, a graphics board is either correctly working because all components follow the standards -or the computer is unusable, because the screen remains dark after booting or it displays the wrong area. Cases when this happens are:

> 今天，随着内核模式设置的出现，图形板要么正常工作，因为所有组件都符合标准，要么计算机无法使用，因为启动后屏幕仍然黑暗，或者显示错误的区域。发生这种情况的情况有：

-   The graphics board does not recognize the monitor.
-   The graphics board is unable to detect any EDID data.
-   The graphics board incorrectly forwards EDID data to the driver.
-   The monitor sends no or bogus EDID data.
-   A KVM sends its own EDID data instead of querying the connected monitor.


Adding the kernel parameter \"nomodeset\" helps in most cases, but causes restrictions later on.

> 在大多数情况下，添加内核参数“nomodeset”会有所帮助，但稍后会造成限制。


As a remedy for such situations, the kernel configuration item CONFIG_DRM_LOAD_EDID_FIRMWARE was introduced. It allows to provide an individually prepared or corrected EDID data set in the /lib/firmware directory from where it is loaded via the firmware interface. The code (see drivers/gpu/drm/drm_edid_load.c) contains built-in data sets for commonly used screen resolutions (800x600, 1024x768, 1280x1024, 1600x1200, 1680x1050, 1920x1080) as binary blobs, but the kernel source tree does not contain code to create these data. In order to elucidate the origin of the built-in binary EDID blobs and to facilitate the creation of individual data for a specific misbehaving monitor, commented sources and a Makefile environment are given here.

> 作为对这种情况的补救措施，引入了内核配置项CONFIG_DRM_LOAD_EDID_FIRMWARE。它允许在/lib/firmware目录中提供单独准备或校正的EDID数据集，从该目录通过固件接口加载EDID数据。该代码（请参阅drivers/gpu/drm/drm_edid_load.c）包含作为二进制Blob的常用屏幕分辨率（800x600、1024x768、1280x1024、1600x1200、1680x1050、1920x1080）的内置数据集，但内核源代码树不包含创建这些数据的代码。为了阐明内置二进制EDID Blob的起源，并促进为特定行为不端的监视器创建单独的数据，这里给出了注释源和Makefile环境。


To create binary EDID and C source code files from the existing data material, simply type \"make\" in tools/edid/.

> 要从现有的数据材料中创建二进制EDID和C源代码文件，只需在tools/EDID/中键入“make”即可。


If you want to create your own EDID file, copy the file 1024x768.S, replace the settings with your own data and add a new target to the Makefile. Please note that the EDID data structure expects the timing values in a different way as compared to the standard X11 format.

> 如果您想创建自己的EDID文件，请复制文件1024x768.S，用您自己的数据替换设置，并将新目标添加到Makefile中。请注意，与标准X11格式相比，EDID数据结构以不同的方式期望定时值。

X11:

:   

    HTimings:

    :   hdisp hsyncstart hsyncend htotal

    VTimings:

    :   vdisp vsyncstart vsyncend vtotal

EDID:

    #define XPIX hdisp
    #define XBLANK htotal-hdisp
    #define XOFFSET hsyncstart-hdisp
    #define XPULSE hsyncend-hsyncstart

    #define YPIX vdisp
    #define YBLANK vtotal-vdisp
    #define YOFFSET vsyncstart-vdisp
    #define YPULSE vsyncend-vsyncstart
