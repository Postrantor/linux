---
tip: translate by baidu@2024-01-30 21:34:12
---
---
subtitle: File system Error Reporting
title: File system Monitoring with fanotify
---


Fanotify supports the FAN_FS_ERROR event type for file system-wide error reporting. It is meant to be used by file system health monitoring daemons, which listen for these events and take actions (notify sysadmin, start recovery) when a file system problem is detected.

> Fanotify支持用于文件系统范围错误报告的FAN_FS_ERROR事件类型。它旨在由文件系统运行状况监视守护进程使用，这些守护进程侦听这些事件，并在检测到文件系统问题时采取行动（通知sysadmin，启动恢复）。


By design, a FAN_FS_ERROR notification exposes sufficient information for a monitoring tool to know a problem in the file system has happened. It doesn\'t necessarily provide a user space application with semantics to verify an IO operation was successfully executed. That is out of scope for this feature. Instead, it is only meant as a framework for early file system problem detection and reporting recovery tools.

> 根据设计，FAN_FS_ERROR通知为监视工具提供了足够的信息，以便知道文件系统中发生了问题。它不一定为用户空间应用程序提供语义来验证IO操作是否成功执行。这超出了此功能的范围。相反，它只是作为早期文件系统问题检测和报告恢复工具的框架。


When a file system operation fails, it is common for dozens of kernel errors to cascade after the initial failure, hiding the original failure log, which is usually the most useful debug data to troubleshoot the problem. For this reason, FAN_FS_ERROR tries to report only the first error that occurred for a file system since the last notification, and it simply counts additional errors. This ensures that the most important pieces of information are never lost.

> 当文件系统操作失败时，通常会在初始失败后级联数十个内核错误，从而隐藏原始故障日志，这通常是解决问题最有用的调试数据。由于这个原因，FAN_FS_ERROR尝试只报告自上次通知以来文件系统发生的第一个错误，而只计算其他错误。这样可以确保最重要的信息永远不会丢失。


FAN_FS_ERROR requires the fanotify group to be setup with the FAN_REPORT_FID flag.

> FAN_FS_ERROR要求使用FAN_REPORT_FID标志设置fanotify组。


At the time of this writing, the only file system that emits FAN_FS_ERROR notifications is Ext4.

> 在撰写本文时，唯一发出FAN_FS_ERROR通知的文件系统是Ext4。

A FAN_FS_ERROR Notification has the following format:

    ::

       [ Notification Metadata (Mandatory) ]
       [ Generic Error Record  (Mandatory) ]
       [ FID record            (Mandatory) ]


The order of records is not guaranteed, and new records might be added in the future. Therefore, applications must not rely on the order and must be prepared to skip over unknown records. Please refer to `samples/fanotify/fs-monitor.c` for an example parser.

> 记录的顺序无法保证，将来可能会添加新的记录。因此，应用程序不能依赖于订单，必须准备跳过未知记录。有关语法分析器的示例，请参阅“samples/fanotify/fs-monitor.c”。

# Generic error record


The generic error record provides enough information for a file system agnostic tool to learn about a problem in the file system, without providing any additional details about the problem. This record is identified by `struct fanotify_event_info_header.info_type` being set to FAN_EVENT_INFO_TYPE_ERROR.

> 通用错误记录为文件系统无关工具提供了足够的信息来了解文件系统中的问题，而无需提供有关该问题的任何其他详细信息。此记录由设置为FAN_event_info_type_ERROR的“struct fanotify_event_info_header.info_type”标识。

>     struct fanotify_event_info_error {
>          struct fanotify_event_info_header hdr;
>         __s32 error;
>         __u32 error_count;
>     };


The [error]{.title-ref} field identifies the type of error using errno values. [error_count]{.title-ref} tracks the number of errors that occurred and were suppressed to preserve the original error information, since the last notification.

> [error]｛.title-ref｝字段使用errno值标识错误类型。[error_count]｛.title-ref｝跟踪自上次通知以来发生并被抑制以保留原始错误信息的错误数。

# FID record


The FID record can be used to uniquely identify the inode that triggered the error through the combination of fsid and file handle. A file system specific application can use that information to attempt a recovery procedure. Errors that are not related to an inode are reported with an empty file handle of type FILEID_INVALID.

> FID记录可用于通过fsid和文件句柄的组合唯一标识触发错误的索引节点。特定于文件系统的应用程序可以使用该信息尝试恢复过程。与索引节点无关的错误报告为FILEID_INVALID类型的空文件句柄。
