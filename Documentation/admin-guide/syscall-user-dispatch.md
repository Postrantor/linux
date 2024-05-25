---
tip: translate by baidu@2024-01-30 21:40:02
---
---
title: Syscall User Dispatch
---

# Background


Compatibility layers like Wine need a way to efficiently emulate system calls of only a part of their process - the part that has the incompatible code - while being able to execute native syscalls without a high performance penalty on the native part of the process. Seccomp falls short on this task, since it has limited support to efficiently filter syscalls based on memory regions, and it doesn\'t support removing filters. Therefore a new mechanism is necessary.

> 像Wine这样的兼容层需要一种方法来有效地模拟仅对其进程的一部分（具有不兼容代码的部分）的系统调用，同时能够执行本机系统调用，而不会对进程的本机部分造成高性能损失。Seccomp无法完成这项任务，因为它对基于内存区域有效过滤系统调用的支持有限，而且不支持删除过滤器。因此，有必要建立一个新的机制。


Syscall User Dispatch brings the filtering of the syscall dispatcher address back to userspace. The application is in control of a flip switch, indicating the current personality of the process. A multiple-personality application can then flip the switch without invoking the kernel, when crossing the compatibility layer API boundaries, to enable/disable the syscall redirection and execute syscalls directly (disabled) or send them to be emulated in userspace through a SIGSYS.

> 系统调用用户调度将系统调用调度程序地址的过滤带回用户空间。该应用程序控制一个翻转开关，指示流程的当前个性。然后，当跨越兼容层API边界时，多人应用程序可以在不调用内核的情况下切换开关，以启用/禁用系统调用重定向并直接执行系统调用（禁用），或者通过SIGSYS将其发送到用户空间中进行仿真。


The goal of this design is to provide very quick compatibility layer boundary crosses, which is achieved by not executing a syscall to change personality every time the compatibility layer executes. Instead, a userspace memory region exposed to the kernel indicates the current personality, and the application simply modifies that variable to configure the mechanism.

> 此设计的目标是提供非常快速的兼容性层边界跨越，这是通过不执行每次执行兼容性层时更改个性的系统调用来实现的。相反，暴露给内核的用户空间内存区域指示当前个性，应用程序只需修改该变量即可配置机制。


There is a relatively high cost associated with handling signals on most architectures, like x86, but at least for Wine, syscalls issued by native Windows code are currently not known to be a performance problem, since they are quite rare, at least for modern gaming applications.

> 在大多数架构（如x86）上，处理信号的成本相对较高，但至少对Wine来说，由本机Windows代码发出的系统调用目前还不是性能问题，因为它们非常罕见，至少在现代游戏应用程序中是这样。


Since this mechanism is designed to capture syscalls issued by non-native applications, it must function on syscalls whose invocation ABI is completely unexpected to Linux. Syscall User Dispatch, therefore doesn\'t rely on any of the syscall ABI to make the filtering. It uses only the syscall dispatcher address and the userspace key.

> 由于此机制旨在捕获非本机应用程序发出的系统调用，因此它必须在调用ABI完全出乎Linux意料的系统调用上运行。因此，Syscall User Dispatch不依赖任何Syscall ABI来进行筛选。它只使用系统调用调度程序地址和用户空间密钥。


As the ABI of these intercepted syscalls is unknown to Linux, these syscalls are not instrumentable via ptrace or the syscall tracepoints.

> 由于Linux不知道这些截获的系统调用的ABI，因此无法通过ptrace或系统调用跟踪点检测这些系统调用。

# Interface


A thread can setup this mechanism on supported kernels by executing the following prctl:

> 线程可以通过执行以下prctl在支持的内核上设置此机制：

> prctl(PR_SET_SYSCALL_USER_DISPATCH, \<op\>, \<offset\>, \<length\>, \[selector\])


\<op\> is either PR_SYS_DISPATCH_ON or PR_SYS_DISPATCH_OFF, to enable and disable the mechanism globally for that thread. When PR_SYS_DISPATCH_OFF is used, the other fields must be zero.

> \<op\>是PR_SYS_DISPATCH_ON或PR_SYS-DISPATCH_OFF，以全局启用和禁用该线程的机制。当使用PR_SYS_DISPATCH_OFF时，其他字段必须为零。


\[\<offset\>, \<offset\>+\<length\>) delimit a memory region interval from which syscalls are always executed directly, regardless of the userspace selector. This provides a fast path for the C library, which includes the most common syscall dispatchers in the native code applications, and also provides a way for the signal handler to return without triggering a nested SIGSYS on (rt\_)sigreturn. Users of this interface should make sure that at least the signal trampoline code is included in this region. In addition, for syscalls that implement the trampoline code on the vDSO, that trampoline is never intercepted.

> \[\<offset\>，\<offset\>+\<length\>）定义了一个内存区域间隔，无论用户空间选择器如何，系统调用总是从该内存区域间隔直接执行。这为C库提供了一个快速路径，其中包括本机代码应用程序中最常见的系统调用调度器，还为信号处理程序提供了一种返回的方式，而无需触发（rt\_）上的嵌套SIGSYSsigreturn。该界面的用户应确保该区域至少包含信号蹦床代码。此外，对于在vDSO上实现蹦床代码的系统调用，该蹦床永远不会被拦截。


\[selector\] is a pointer to a char-sized region in the process memory region, that provides a quick way to enable disable syscall redirection thread-wide, without the need to invoke the kernel directly. selector can be set to SYSCALL_DISPATCH_FILTER_ALLOW or SYSCALL_DISPATCH_FILTER_BLOCK. Any other value should terminate the program with a SIGSYS.

> \[selector\]是指向进程内存区域中字符大小区域的指针，它提供了一种在线程范围内启用禁用系统调用重定向的快速方法，而无需直接调用内核。选择器可以设置为SYSCALL_DISPATCH_FILTER_ALLOW或SYSCALL_ISPATCH_FILTER_BLOCK。任何其他值都应使用SIGSYS终止程序。


Additionally, a tasks syscall user dispatch configuration can be peeked and poked via the [PTRACE]()(GET\|SET)\_SYSCALL_USER_DISPATCH_CONFIG ptrace requests. This is useful for checkpoint/restart software.

> 此外，任务系统调用用户调度配置可以通过[PTRACE]（）（GET\|SET）\_syscall_user_dispatch_CONFIG PTRACE请求进行窥探和窥探。这对于检查点/重新启动软件非常有用。

# Security Notes


Syscall User Dispatch provides functionality for compatibility layers to quickly capture system calls issued by a non-native part of the application, while not impacting the Linux native regions of the process. It is not a mechanism for sandboxing system calls, and it should not be seen as a security mechanism, since it is trivial for a malicious application to subvert the mechanism by jumping to an allowed dispatcher region prior to executing the syscall, or to discover the address and modify the selector value. If the use case requires any kind of security sandboxing, Seccomp should be used instead.

> Syscall User Dispatch为兼容性层提供了快速捕获由应用程序的非本机部分发出的系统调用的功能，同时不会影响进程的Linux本机区域。它不是沙盒系统调用的机制，也不应被视为一种安全机制，因为恶意应用程序在执行系统调用之前跳到允许的调度程序区域来破坏该机制，或者发现地址并修改选择器值，都是微不足道的。如果用例需要任何类型的安全沙盒，则应使用Seccomp。


Any fork or exec of the existing process resets the mechanism to PR_SYS_DISPATCH_OFF.

> 现有进程的任何分支或exec都会将机制重置为PR_SYS_DISPATCH_OFF。
