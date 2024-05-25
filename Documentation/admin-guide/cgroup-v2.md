---
tip: translate by baidu@2024-01-30 21:30:26
---
---
author:
  - Tejun Heo \<<tj@kernel.org>\>
date: October, 2015
title: Control Group v2
---


This is the authoritative documentation on the design, interface and conventions of cgroup v2. It describes all userland-visible aspects of cgroup including core and specific controller behaviors. All future changes must be reflected in this document. Documentation for v1 is available under `Documentation/admin-guide/cgroup-v1/index.rst <cgroup-v1>`{.interpreted-text role="ref"}.

> 这是关于cgroupv2的设计、接口和约定的权威文档。它描述了cgroup的所有用户和可见方面，包括核心和特定控制器的行为。所有未来的变化都必须反映在本文件中。v1的文档可在`Documentation/admin guide/cgroup-v1/index.rst<cgroup-v1>`{.depreted text role=“ref”}下获得。

1\. Introduction

: 1-1. Terminology 1-2. What is cgroup?

2\. Basic Operations


: 2-1. Mounting 2-2. Organizing Processes and Threads 2-2-1. Processes 2-2-2. Threads 2-3. \[Un\]populated Notification 2-4. Controlling Controllers 2-4-1. Enabling and Disabling 2-4-2. Top-down Constraint 2-4-3. No Internal Process Constraint 2-5. Delegation 2-5-1. Model of Delegation 2-5-2. Delegation Containment 2-6. Guidelines 2-6-1. Organize Once and Control 2-6-2. Avoid Name Collisions

> : 2-1. 安装2-2。组织过程和线程2-2-1。过程2-2-2。螺纹2-3\未填充通知2-4。控制控制器2-4-1。启用和禁用2-4-2。自上而下约束2-4-3。无内部流程约束2-5。代表团2-5-1。代表团模式2-5-2。授权包含2-6。指南2-6-1。组织一次并控制2-6-2。避免名称冲突

3\. Resource Distribution Models

: 3-1. Weights 3-2. Limits 3-3. Protections 3-4. Allocations

4\. Interface Files

: 4-1. Format 4-2. Conventions 4-3. Core Interface Files

5\. Controllers

:

5-1. CPU

:   5-1-1. CPU Interface Files

5-2. Memory


:   5-2-1. Memory Interface Files 5-2-2. Usage Guidelines 5-2-3. Memory Ownership

> :   5-2-1. 内存接口文件5-2-2。使用指南5-2-3。内存所有权

5-3. IO


:   5-3-1. IO Interface Files 5-3-2. Writeback 5-3-3. IO Latency 5-3-3-1. How IO Latency Throttling Works 5-3-3-2. IO Latency Interface Files 5-3-4. IO Priority

> :   5-3-1. IO接口文件5-3-2。写回5-3-3。IO延迟5-3-3-1。IO延迟限制的工作原理5-3-3-2。IO延迟接口文件5-3-4。IO优先级

5-4. PID

:   5-4-1. PID Interface Files

5-5. Cpuset

:   5.5-1. Cpuset Interface Files


5-6. Device 5-7. RDMA 5-7-1. RDMA Interface Files 5-8. HugeTLB 5.8-1. HugeTLB Interface Files 5-9. Misc 5.9-1 Miscellaneous cgroup Interface Files 5.9-2 Migration and Ownership 5-10. Others 5-10-1. perf_event 5-N. Non-normative information 5-N-1. CPU controller root cgroup process behaviour 5-N-2. IO controller root cgroup process behaviour

> 5-6.装置5-7。RDMA 5-7-1。RDMA接口文件5-8。HugeTLB 5.8-1。HugeTLB接口文件5-9。其他5.9-1其他cgroup接口文件5.9-2迁移和所有权5-10。其他5-10-1。perf_event 5-N。非规范性信息5-N-1。CPU控制器根组进程行为5-N-2。IO控制器根进程行为

6\. Namespace


: 6-1. Basics 6-2. The Root and Views 6-3. Migration and setns(2) 6-4. Interaction with Other Namespaces

> : 6-1. 基础6-2。根和观点6-3。迁移和setns（2）6-4。与其他命名空间的交互

P. Information on Kernel Programming

: P-1. Filesystem Support for Writeback

D. Deprecated v1 Core Features

R. Issues with v1 and Rationales for v2


: R-1. Multiple Hierarchies R-2. Thread Granularity R-3. Competition Between Inner Nodes and Threads R-4. Other Interface Issues R-5. Controller Issues and Remedies R-5-1. Memory

> ：R-1。多层次结构R-2。螺纹粒度R-3。内部节点和线程之间的竞争R-4。其他接口问题R-5。控制器问题和补救措施R-5-1。记忆力

# Introduction

## Terminology


\"cgroup\" stands for \"control group\" and is never capitalized. The singular form is used to designate the whole feature and also as a qualifier as in \"cgroup controllers\". When explicitly referring to multiple individual control groups, the plural form \"cgroups\" is used.

> \“cgroup”代表“对照组”，从不大写。单数形式用于指定整个功能，也用作\“cgroup controllers”中的限定符。当明确提及多个单独的对照组时，使用复数形式“cgroups”。

## What is cgroup?


cgroup is a mechanism to organize processes hierarchically and distribute system resources along the hierarchy in a controlled and configurable manner.

> cgroup是一种按层次结构组织流程并以可控和可配置的方式沿层次结构分配系统资源的机制。


cgroup is largely composed of two parts - the core and controllers. cgroup core is primarily responsible for hierarchically organizing processes. A cgroup controller is usually responsible for distributing a specific type of system resource along the hierarchy although there are utility controllers which serve purposes other than resource distribution.

> cgroup主要由核心和控制器两部分组成。cgroupcore主要负责按层次组织流程。cgroup控制器通常负责沿着层次结构分配特定类型的系统资源，尽管存在用于资源分配以外的目的的实用程序控制器。


cgroups form a tree structure and every process in the system belongs to one and only one cgroup. All threads of a process belong to the same cgroup. On creation, all processes are put in the cgroup that the parent process belongs to at the time. A process can be migrated to another cgroup. Migration of a process doesn\'t affect already existing descendant processes.

> cgroups形成一个树结构，系统中的每个进程都属于一个且仅属于一个cgroup。进程的所有线程都属于同一个cgroup。创建时，所有进程都被放入父进程当时所属的cgroup中。一个进程可以迁移到另一个cgroup。进程的迁移不会影响现有的子进程。


Following certain structural constraints, controllers may be enabled or disabled selectively on a cgroup. All controller behaviors are hierarchical - if a controller is enabled on a cgroup, it affects all processes which belong to the cgroups consisting the inclusive sub-hierarchy of the cgroup. When a controller is enabled on a nested cgroup, it always restricts the resource distribution further. The restrictions set closer to the root in the hierarchy can not be overridden from further away.

> 根据某些结构约束，可以在cgroup上选择性地启用或禁用控制器。所有控制器行为都是分层的——如果在一个cgroup上启用了控制器，它会影响属于cgroup的包含子层次结构的cgroups的所有进程。当在嵌套的cgroup上启用控制器时，它总是进一步限制资源分布。在层次结构中靠近根设置的限制不能从远处覆盖。

# Basic Operations

## Mounting


Unlike v1, cgroup v2 has only single hierarchy. The cgroup v2 hierarchy can be mounted with the following mount command:

> 与v1不同，cgroupv2只有一个层次结构。cgroup v2层次结构可以使用以下mount命令进行挂载：

# mount -t cgroup2 none $MOUNT_POINT


cgroup2 filesystem has the magic number 0x63677270 (\"cgrp\"). All controllers which support v2 and are not bound to a v1 hierarchy are automatically bound to the v2 hierarchy and show up at the root. Controllers which are not in active use in the v2 hierarchy can be bound to other hierarchies. This allows mixing v2 hierarchy with the legacy v1 multiple hierarchies in a fully backward compatible way.

> cgroup2文件系统具有神奇的数字0x63677270（\“cgrp\”）。所有支持v2且未绑定到v1层次结构的控制器都会自动绑定到v2层次结构并显示在根目录中。在v2层次结构中未处于活动使用中的控制器可以绑定到其他层次结构。这允许以完全向后兼容的方式将v2层次结构与传统的v1多个层次结构混合。


A controller can be moved across hierarchies only after the controller is no longer referenced in its current hierarchy. Because per-cgroup controller states are destroyed asynchronously and controllers may have lingering references, a controller may not show up immediately on the v2 hierarchy after the final umount of the previous hierarchy. Similarly, a controller should be fully disabled to be moved out of the unified hierarchy and it may take some time for the disabled controller to become available for other hierarchies; furthermore, due to inter-controller dependencies, other controllers may need to be disabled too.

> 只有在控制器在其当前层次结构中不再被引用后，才能在层次结构之间移动控制器。由于每个cgroup的控制器状态是异步销毁的，并且控制器可能具有延迟引用，因此控制器可能不会在前一层次结构的最终umount之后立即显示在v2层次结构上。类似地，控制器应该被完全禁用以移出统一层次结构，并且被禁用的控制器可能需要一些时间才能变为可用于其他层次结构；此外，由于控制器间的依赖性，可能也需要禁用其他控制器。


While useful for development and manual configurations, moving controllers dynamically between the v2 and other hierarchies is strongly discouraged for production use. It is recommended to decide the hierarchies and controller associations before starting using the controllers after system boot.

> 虽然对开发和手动配置很有用，但强烈反对在v2和其他层次结构之间动态移动控制器以供生产使用。建议在系统引导后开始使用控制器之前，先确定层次结构和控制器关联。


During transition to v2, system management software might still automount the v1 cgroup filesystem and so hijack all controllers during boot, before manual intervention is possible. To make testing and experimenting easier, the kernel parameter cgroup_no_v1= allows disabling controllers in v1 and make them always available in v2.

> 在转换到v2的过程中，系统管理软件可能仍然会自动安装v1 cgroup文件系统，因此在手动干预之前，会在引导期间劫持所有控制器。为了使测试和实验更容易，内核参数cgroup_no_v1=允许禁用v1中的控制器，并使它们在v2中始终可用。

cgroup v2 currently supports the following mount options.

nsdelegate


: Consider cgroup namespaces as delegation boundaries. This option is system wide and can only be set on mount or modified through remount from the init namespace. The mount option is ignored on non-init namespace mounts. Please refer to the Delegation section for details.

> ：将cgroup命名空间视为委派边界。此选项是系统范围的，只能在装载时设置，也只能通过从init命名空间重新装载进行修改。在非init命名空间装载上，将忽略装载选项。有关详细信息，请参阅“委派”部分。

favordynmods


: Reduce the latencies of dynamic cgroup modifications such as task migrations and controller on/offs at the cost of making hot path operations such as forks and exits more expensive. The static usage pattern of creating a cgroup, enabling controllers, and then seeding it with CLONE_INTO_CGROUP is not affected by this option.

> ：减少动态cgroup修改（如任务迁移和控制器打开/关闭）的延迟，代价是使分叉和出口等热路径操作更加昂贵。创建一个cgroup，启用控制器，然后用CLONE_INTO_cgroup为其设定种子的静态使用模式不受此选项的影响。

memory_localevents


: Only populate memory.events with data for the current cgroup, and not any subtrees. This is legacy behaviour, the default behaviour without this option is to include subtree counts. This option is system wide and can only be set on mount or modified through remount from the init namespace. The mount option is ignored on non-init namespace mounts.

> ：只使用当前cgroup的数据填充memory.events，而不使用任何子树。这是遗留行为，没有此选项的默认行为是包括子树计数。此选项是系统范围的，只能在装载时设置，也只能通过从init命名空间重新装载进行修改。在非init命名空间装载上，将忽略装载选项。

memory_recursiveprot


: Recursively apply memory.min and memory.low protection to entire subtrees, without requiring explicit downward propagation into leaf cgroups. This allows protecting entire subtrees from one another, while retaining free competition within those subtrees. This should have been the default behavior but is a mount-option to avoid regressing setups relying on the original semantics (e.g. specifying bogusly high \'bypass\' protection values at higher tree levels).

> ：递归地将memory.min和memory.low保护应用于整个子树，而不需要显式向下传播到叶cgroups中。这允许保护整个子树彼此不受影响，同时保留这些子树内的自由竞争。这本应是默认行为，但却是一个装载选项，以避免依赖于原始语义的设置倒退（例如，在更高的树级别上指定高得离谱的“旁路”保护值）。

memory_hugetlb_accounting


: Count HugeTLB memory usage towards the cgroup\'s overall memory usage for the memory controller (for the purpose of statistics reporting and memory protetion). This is a new behavior that could regress existing setups, so it must be explicitly opted in with this mount option.

> ：将HugeTLB内存使用量计入内存控制器的cgroup的总体内存使用量（用于统计报告和内存保护）。这是一种新的行为，可能会倒退现有的设置，因此必须使用此装载选项明确选择它。

A few caveats to keep in mind:


-   There is no HugeTLB pool management involved in the memory controller. The pre-allocated pool does not belong to anyone. Specifically, when a new HugeTLB folio is allocated to the pool, it is not accounted for from the perspective of the memory controller. It is only charged to a cgroup when it is actually used (for e.g at page fault time). Host memory overcommit management has to consider this when configuring hard limits. In general, HugeTLB pool management should be done via other mechanisms (such as the HugeTLB controller).

> -内存控制器中不涉及HugeTLB池管理。预先分配的池不属于任何人。具体地说，当一个新的HugeTLB文件夹被分配到池中时，它不会从内存控制器的角度来考虑。它只在实际使用时（例如，在页面故障时）向cgroup收费。主机内存过度使用管理在配置硬限制时必须考虑这一点。通常，HugeTLB池管理应通过其他机制（如HugeTLB控制器）进行。

-   Failure to charge a HugeTLB folio to the memory controller results in SIGBUS. This could happen even if the HugeTLB pool still has pages available (but the cgroup limit is hit and reclaim attempt fails).

> -未能将HugeTLB文件夹充电到内存控制器会导致SIGBUS。即使HugeTLB池仍然有可用的页面（但达到了cgroup限制并且回收尝试失败），也可能发生这种情况。

-   Charging HugeTLB memory towards the memory controller affects memory protection and reclaim dynamics. Any userspace tuning (of low, min limits for e.g) needs to take this into account.

> -向内存控制器充电HugeTLB内存会影响内存保护和回收动态。任何用户空间调整（例如，低、最小限制）都需要考虑到这一点。

-   HugeTLB pages utilized while this option is not selected will not be tracked by the memory controller (even if cgroup v2 is remounted later on).

> -未选择此选项时使用的HugeTLB页面将不会被内存控制器跟踪（即使稍后重新装入cgroup v2）。

## Organizing Processes and Threads

### Processes


Initially, only the root cgroup exists to which all processes belong. A child cgroup can be created by creating a sub-directory:

> 最初，只存在所有进程所属的根cgroup。可以通过创建子目录来创建子cgroup：

# mkdir $CGROUP_NAME


A given cgroup may have multiple child cgroups forming a tree structure. Each cgroup has a read-writable interface file \"cgroup.procs\". When read, it lists the PIDs of all processes which belong to the cgroup one-per-line. The PIDs are not ordered and the same PID may show up more than once if the process got moved to another cgroup and then back or the PID got recycled while reading.

> 给定的cgroup可以具有多个子cgroup，这些子cgroup形成树结构。每个cgroup都有一个可读写的接口文件“cgroup.procs\”。读取时，它每行列出一个属于该cgroup的所有进程的PID。PID没有排序，如果进程被移到另一个组，然后返回，或者PID在读取时被回收，则同一PID可能会多次显示。


A process can be migrated into a cgroup by writing its PID to the target cgroup\'s \"cgroup.procs\" file. Only one process can be migrated on a single write(2) call. If a process is composed of multiple threads, writing the PID of any thread migrates all threads of the process.

> 进程可以通过将其PID写入目标cgroup的“cgroup.procs”文件来迁移到cgroup中。一次写入（2）调用只能迁移一个进程。如果一个进程由多个线程组成，那么写入任何线程的PID都会迁移该进程的所有线程。


When a process forks a child process, the new process is born into the cgroup that the forking process belongs to at the time of the operation. After exit, a process stays associated with the cgroup that it belonged to at the time of exit until it\'s reaped; however, a zombie process does not appear in \"cgroup.procs\" and thus can\'t be moved to another cgroup.

> 当一个进程分叉一个子进程时，新进程将生成到分叉进程在操作时所属的cgroup中。退出后，进程将与退出时所属的cgroup保持关联，直到它被收割；但是，僵尸进程不会出现在“cgroup.procs”中，因此无法移动到另一个cgroup。


A cgroup which doesn\'t have any children or live processes can be destroyed by removing the directory. Note that a cgroup which doesn\'t have any children and is associated only with zombie processes is considered empty and can be removed:

> 没有任何子级或活动进程的cgroup可以通过删除目录来销毁。请注意，没有任何子级且仅与僵尸进程关联的cgroup被视为空，可以删除：

# rmdir $CGROUP_NAME


\"/proc/\$PID/cgroup\" lists a process\'s cgroup membership. If legacy cgroup is in use in the system, this file may contain multiple lines, one for each hierarchy. The entry for cgroup v2 is always in the format \"0::\$PATH\":

> \“/proc/\$PID/cgroup”列出进程的cgroup成员身份。如果系统中正在使用旧式cgroup，则此文件可能包含多行，每个层次结构一行。cgroup v2的条目的格式始终为“0:：\$PATH\”：

# cat /proc/842/cgroup
...
0::/test-cgroup/test-cgroup-nested


If the process becomes a zombie and the cgroup it was associated with is removed subsequently, \" (deleted)\" is appended to the path:

> 如果进程变成僵尸，并且与之关联的cgroup随后被删除，则“（已删除）”将附加到路径：

# cat /proc/842/cgroup
...
0::/test-cgroup/test-cgroup-nested (deleted)

### Threads


cgroup v2 supports thread granularity for a subset of controllers to support use cases requiring hierarchical resource distribution across the threads of a group of processes. By default, all threads of a process belong to the same cgroup, which also serves as the resource domain to host resource consumptions which are not specific to a process or thread. The thread mode allows threads to be spread across a subtree while still maintaining the common resource domain for them.

> cgroupv2支持控制器子集的线程粒度，以支持需要跨一组进程的线程进行分层资源分配的用例。默认情况下，进程的所有线程都属于同一个cgroup，该cgroup还充当资源域，以承载并非特定于进程或线程的资源消耗。线程模式允许线程分布在子树中，同时仍然为它们维护公共资源域。


Controllers which support thread mode are called threaded controllers. The ones which don\'t are called domain controllers.

> 支持线程模式的控制器称为线程控制器。没有的称为域控制器。


Marking a cgroup threaded makes it join the resource domain of its parent as a threaded cgroup. The parent may be another threaded cgroup whose resource domain is further up in the hierarchy. The root of a threaded subtree, that is, the nearest ancestor which is not threaded, is called threaded domain or thread root interchangeably and serves as the resource domain for the entire subtree.

> 将cgroup标记为线程化使其作为线程化cgroup加入其父级的资源域。父级可以是另一个线程cgroup，其资源域在层次结构中更靠上。带线程子树的根，即未带线程的最近祖先，可互换地称为线程域或线程根，并作为整个子树的资源域。


Inside a threaded subtree, threads of a process can be put in different cgroups and are not subject to the no internal process constraint - threaded controllers can be enabled on non-leaf cgroups whether they have threads in them or not.

> 在线程子树中，进程的线程可以放在不同的cgroup中，并且不受无内部进程约束的约束-线程控制器可以在非叶cgroup上启用，无论它们中是否有线程。


As the threaded domain cgroup hosts all the domain resource consumptions of the subtree, it is considered to have internal resource consumptions whether there are processes in it or not and can\'t have populated child cgroups which aren\'t threaded. Because the root cgroup is not subject to no internal process constraint, it can serve both as a threaded domain and a parent to domain cgroups.

> 由于线程域cgroup承载子树的所有域资源消耗，因此无论其中是否有进程，它都被认为具有内部资源消耗，并且不能填充未线程化的子cgroup。因为根cgroup不受任何内部进程约束，所以它既可以作为线程域，也可以作为域cgroup的父级。


The current operation mode or type of the cgroup is shown in the \"cgroup.type\" file which indicates whether the cgroup is a normal domain, a domain which is serving as the domain of a threaded subtree, or a threaded cgroup.

> cgroup的当前操作模式或类型显示在“cgroup.type”文件中，该文件指示cgroup是普通域、用作线程子树域的域还是线程cgroup。


On creation, a cgroup is always a domain cgroup and can be made threaded by writing \"threaded\" to the \"cgroup.type\" file. The operation is single direction:

> 在创建时，cgroup始终是域cgroup，可以通过将“threeded”写入“cgroup.type”文件来实现线程化。操作是单向的：

# echo threaded cgroup.type


Once threaded, the cgroup can\'t be made a domain again. To enable the thread mode, the following conditions must be met.

> 一旦线程化，cgroup就不能再次成为域。要启用线程模式，必须满足以下条件。


- As the cgroup will join the parent\'s resource domain. The parent must either be a valid (threaded) domain or a threaded cgroup.

> -因为cgroup将加入父级的资源域。父级必须是有效的（线程化的）域或线程化的cgroup。

- When the parent is an unthreaded domain, it must not have any domain controllers enabled or populated domain children. The root is exempt from this requirement.

> -当父域是无线程域时，它不能启用任何域控制器或填充域子域。根不受此要求的约束。


Topology-wise, a cgroup can be in an invalid state. Please consider the following topology:

> 从拓扑角度来看，cgroup可能处于无效状态。请考虑以下拓扑结构：

A (threaded domain) - B (threaded) - C (domain, just created)


C is created as a domain but isn\'t connected to a parent which can host child domains. C can\'t be used until it is turned into a threaded cgroup. \"cgroup.type\" file will report \"domain (invalid)\" in these cases. Operations which fail due to invalid topology use EOPNOTSUPP as the errno.

> C被创建为域，但没有连接到可以承载子域的父级。只有将C转换成一个线程化的cgroup，才能使用它\在这些情况下，“cgroup.type”文件将报告“域（无效）”。由于拓扑无效而失败的操作使用EOPNOTSUPP作为错误号。


A domain cgroup is turned into a threaded domain when one of its child cgroup becomes threaded or threaded controllers are enabled in the \"cgroup.subtree_control\" file while there are processes in the cgroup. A threaded domain reverts to a normal domain when the conditions clear.

> 当域cgroup的一个子cgroup变为线程化或在“cgroup.subtree_control”文件中启用线程控制器时，该cgroup将变为线程域，而cgroup中有进程。当条件清除时，线程域将恢复为正常域。


When read, \"cgroup.threads\" contains the list of the thread IDs of all threads in the cgroup. Except that the operations are per-thread instead of per-process, \"cgroup.threads\" has the same format and behaves the same way as \"cgroup.procs\". While \"cgroup.threads\" can be written to in any cgroup, as it can only move threads inside the same threaded domain, its operations are confined inside each threaded subtree.

> 读取时，“cgroup.threads”包含cgroup中所有线程的线程ID列表。除了每个线程而不是每个进程的操作之外，“cgroup.threads”具有与“cgroup.procs”相同的格式和行为方式。虽然“cgroup.threads“可以在任何cgroup中写入，因为它只能在同一线程域内移动线程，但它的操作被限制在每个线程子树内。


The threaded domain cgroup serves as the resource domain for the whole subtree, and, while the threads can be scattered across the subtree, all the processes are considered to be in the threaded domain cgroup. \"cgroup.procs\" in a threaded domain cgroup contains the PIDs of all processes in the subtree and is not readable in the subtree proper. However, \"cgroup.procs\" can be written to from anywhere in the subtree to migrate all threads of the matching process to the cgroup.

> 线程域cgroup充当整个子树的资源域，虽然线程可以分散在子树上，但所有进程都被认为在线程域cggroup中\线程域中的“cgroup.procs\”cgroup包含子树中所有进程的PID，在子树中是不可读的。但是，可以从子树中的任何位置将“cgroup.procs”写入，以将匹配进程的所有线程迁移到cgroup。


Only threaded controllers can be enabled in a threaded subtree. When a threaded controller is enabled inside a threaded subtree, it only accounts for and controls resource consumptions associated with the threads in the cgroup and its descendants. All consumptions which aren\'t tied to a specific thread belong to the threaded domain cgroup.

> 在线程子树中只能启用线程控制器。当在线程子树内启用线程控制器时，它只考虑并控制与cgroup及其子代中的线程相关联的资源消耗。所有未绑定到特定线程的消费都属于线程域cgroup。


Because a threaded subtree is exempt from no internal process constraint, a threaded controller must be able to handle competition between threads in a non-leaf cgroup and its child cgroups. Each threaded controller defines how such competitions are handled.

> 因为线程子树不受任何内部进程约束，所以线程控制器必须能够处理非叶cgroup中的线程与其子cgroup之间的竞争。每个线程控制器都定义了如何处理此类竞争。


Currently, the following controllers are threaded and can be enabled in a threaded cgroup:

> 目前，以下控制器是线程化的，可以在线程化的cgroup中启用：

- cpu
- cpuset
- perf_event
- pids

## \[Un\]populated Notification


Each non-root cgroup has a \"cgroup.events\" file which contains \"populated\" field indicating whether the cgroup\'s sub-hierarchy has live processes in it. Its value is 0 if there is no live process in the cgroup and its descendants; otherwise, 1. poll and \[id\]notify events are triggered when the value changes. This can be used, for example, to start a clean-up operation after all processes of a given sub-hierarchy have exited. The populated state updates and notifications are recursive. Consider the following sub-hierarchy where the numbers in the parentheses represent the numbers of processes in each cgroup:

> 每个非根cgroup都有一个“cgroup.events”文件，其中包含“已填充”字段，指示cgroup的子层次结构中是否有活动进程。如果cgroup及其子代中没有活动进程，则其值为0；否则，1。当值更改时，将触发poll和\[id\]notify事件。例如，这可以用于在给定子层次结构的所有进程退出后启动清理操作。填充的状态更新和通知是递归的。考虑以下子层次结构，其中括号中的数字表示每个cgroup中的进程数：

A(4) - B(0) - C(1)
            \ D(0)


A, B and C\'s \"populated\" fields would be 1 while D\'s 0. After the one process in C exits, B and C\'s \"populated\" fields would flip to \"0\" and file modified events will be generated on the \"cgroup.events\" files of both cgroups.

> A、 B和C的“填充”字段为1，而D为0。在C中的一个进程退出后，B和C的“已填充”字段将翻转到“0\”，并且在两个cgroup的“cgroup.events”文件上将生成文件修改事件。

## Controlling Controllers

### Enabling and Disabling


Each cgroup has a \"cgroup.controllers\" file which lists all controllers available for the cgroup to enable:

> 每个cgroup都有一个“cgroup.controllers”文件，其中列出了该cgroup可启用的所有控制器：

# cat cgroup.controllers
cpu io memory


No controller is enabled by default. Controllers can be enabled and disabled by writing to the \"cgroup.subtree_control\" file:

> 默认情况下未启用任何控制器。可以通过写入“cgroup.subtree_control”文件来启用和禁用控制器：

# echo "+cpu +memory -io" cgroup.subtree_control


Only controllers which are listed in \"cgroup.controllers\" can be enabled. When multiple operations are specified as above, either they all succeed or fail. If multiple operations on the same controller are specified, the last one is effective.

> 只能启用“cgroup.controllers”中列出的控制器。当如上所述指定多个操作时，它们要么全部成功，要么全部失败。如果在同一控制器上指定了多个操作，则最后一个操作有效。


Enabling a controller in a cgroup indicates that the distribution of the target resource across its immediate children will be controlled. Consider the following sub-hierarchy. The enabled controllers are listed in parentheses:

> 启用cgroup中的控制器表示将控制目标资源在其直属子级之间的分布。考虑以下子层次结构。括号中列出了启用的控制器：

A(cpu,memory) - B(memory) - C()
                          \ D()


As A has \"cpu\" and \"memory\" enabled, A will control the distribution of CPU cycles and memory to its children, in this case, B. As B has \"memory\" enabled but not \"CPU\", C and D will compete freely on CPU cycles but their division of memory available to B will be controlled.

> 由于A启用了“cpu”和“内存”，A将控制cpu周期和内存分配给其子级，在这种情况下为B。由于B启用了“内存”但没有启用“cpu”，C和D将在cpu周期上自由竞争，但它们对B可用内存的分配将受到控制。


As a controller regulates the distribution of the target resource to the cgroup\'s children, enabling it creates the controller\'s interface files in the child cgroups. In the above example, enabling \"cpu\" on B would create the \"cpu.\" prefixed controller interface files in C and D. Likewise, disabling \"memory\" from B would remove the \"memory.\" prefixed controller interface files from C and D. This means that the controller interface files - anything which doesn\'t start with \"cgroup.\" are owned by the parent rather than the cgroup itself.

> 当控制器调节目标资源到cgroup的子级的分配时，启用它会在子cgroups中创建控制器的接口文件。在上面的例子中，在B上启用“cpu”将在C和D中创建前缀为“cpu.”的控制器接口文件。同样，从B中禁用“memory”将从C和D删除前缀为“memory.”的控制器界面文件。这意味着控制器接口文件-任何不以“cgroup.”开头的文件都由父级而非cgroup本身所有。

### Top-down Constraint


Resources are distributed top-down and a cgroup can further distribute a resource only if the resource has been distributed to it from the parent. This means that all non-root \"cgroup.subtree_control\" files can only contain controllers which are enabled in the parent\'s \"cgroup.subtree_control\" file. A controller can be enabled only if the parent has the controller enabled and a controller can\'t be disabled if one or more children have it enabled.

> 资源是自上而下分布的，只有当资源已经从父级分配给cgroup时，cgroup才能进一步分配资源。这意味着所有非根的“cgroup.subtree_control”文件只能包含在父级的“cggroup.subtree/control”中启用的控制器。只有当父控制器已启用时，才能启用控制器；如果一个或多个子控制器已启用，则不能禁用控制器。

### No Internal Process Constraint


Non-root cgroups can distribute domain resources to their children only when they don\'t have any processes of their own. In other words, only domain cgroups which don\'t contain any processes can have domain controllers enabled in their \"cgroup.subtree_control\" files.

> 只有当非根cgroups没有自己的进程时，它们才能将域资源分发给它们的子级。换句话说，只有不包含任何进程的域cgroups才能在其“cgroup.subtree_control”文件中启用域控制器。


This guarantees that, when a domain controller is looking at the part of the hierarchy which has it enabled, processes are always only on the leaves. This rules out situations where child cgroups compete against internal processes of the parent.

> 这保证了，当域控制器查看层次结构中启用它的部分时，进程总是只在叶子上。这排除了子cgroup与父cgroup的内部进程竞争的情况。


The root cgroup is exempt from this restriction. Root contains processes and anonymous resource consumption which can\'t be associated with any other cgroups and requires special treatment from most controllers. How resource consumption in the root cgroup is governed is up to each controller (for more information on this topic please refer to the Non-normative information section in the Controllers chapter).

> 根cgroup不受此限制。根包含进程和匿名资源消耗，它们不能与任何其他cgroup关联，并且需要大多数控制器进行特殊处理。如何管理根cgroup中的资源消耗取决于每个控制器（有关此主题的更多信息，请参阅控制器一章中的非规范性信息部分）。


Note that the restriction doesn\'t get in the way if there is no enabled controller in the cgroup\'s \"cgroup.subtree_control\". This is important as otherwise it wouldn\'t be possible to create children of a populated cgroup. To control resource distribution of a cgroup, the cgroup must create children and transfer all its processes to the children before enabling controllers in its \"cgroup.subtree_control\" file.

> 请注意，如果cgroup的“cgroup.subtree_control”中没有启用的控制器，则该限制不会成为障碍。这一点很重要，因为否则将无法创建已填充的cggroup的子级。要控制cgroup的资源分布，cgroup必须创建子级并将其所有进程传输到子级，然后才能在其“cgroup.subtree_control”文件中启用控制器。

## Delegation

### Model of Delegation


A cgroup can be delegated in two ways. First, to a less privileged user by granting write access of the directory and its \"cgroup.procs\", \"cgroup.threads\" and \"cgroup.subtree_control\" files to the user. Second, if the \"nsdelegate\" mount option is set, automatically to a cgroup namespace on namespace creation.

> 一个cgroup可以通过两种方式进行委派。首先，通过将目录及其“cgroup.procs\”、“cgroup.threads\”和“cgroup.subtree_control”文件的写入权限授予特权较低的用户。其次，如果设置了“nsdelegate”mount选项，则在创建命名空间时自动将其设置为cgroup命名空间。


Because the resource control interface files in a given directory control the distribution of the parent\'s resources, the delegatee shouldn\'t be allowed to write to them. For the first method, this is achieved by not granting access to these files. For the second, the kernel rejects writes to all files other than \"cgroup.procs\" and \"cgroup.subtree_control\" on a namespace root from inside the namespace.

> 由于给定目录中的资源控制接口文件控制父级资源的分布，因此不应允许被委派者写入这些文件。对于第一种方法，这是通过不授予对这些文件的访问权限来实现的。第二种情况是，内核拒绝从命名空间内部向命名空间根上的“cgroup.procs\”和“cgroup.subtree_control\”以外的所有文件写入。


The end results are equivalent for both delegation types. Once delegated, the user can build sub-hierarchy under the directory, organize processes inside it as it sees fit and further distribute the resources it received from the parent. The limits and other settings of all resource controllers are hierarchical and regardless of what happens in the delegated sub-hierarchy, nothing can escape the resource restrictions imposed by the parent.

> 两种委托类型的最终结果都是等效的。一旦被委派，用户就可以在目录下构建子层次结构，根据自己的意愿组织其中的流程，并进一步分配从父目录接收的资源。所有资源控制器的限制和其他设置都是分层的，无论委托的子层次结构中发生了什么，都无法逃脱父级施加的资源限制。


Currently, cgroup doesn\'t impose any restrictions on the number of cgroups in or nesting depth of a delegated sub-hierarchy; however, this may be limited explicitly in the future.

> 目前，cgroup没有对委托子层次结构中的cgroup数量或嵌套深度施加任何限制；然而，这在未来可能会受到明确的限制。

### Delegation Containment


A delegated sub-hierarchy is contained in the sense that processes can\'t be moved into or out of the sub-hierarchy by the delegatee.

> 被委派的子层次结构包含在被委派者不能将进程移入或移出子层次结构的意义上。


For delegations to a less privileged user, this is achieved by requiring the following conditions for a process with a non-root euid to migrate a target process into a cgroup by writing its PID to the \"cgroup.procs\" file.

> 对于授权给特权较低的用户，这是通过要求具有非根euid的进程满足以下条件来实现的，以便通过将目标进程的PID写入“cgroup.procs”文件将其迁移到cgroup中。

- The writer must have write access to the \"cgroup.procs\" file.

- The writer must have write access to the \"cgroup.procs\" file of the common ancestor of the source and destination cgroups.

> -编写器必须具有对源和目标cgroups的公共祖先的“cgroup.procs”文件的写访问权限。


The above two constraints ensure that while a delegatee may migrate processes around freely in the delegated sub-hierarchy it can\'t pull in from or push out to outside the sub-hierarchy.

> 以上两个约束确保了当被委派者可以在被委派的子层次结构中自由迁移进程时，它不能从子层次结构外部拉入或推出。


For an example, let\'s assume cgroups C0 and C1 have been delegated to user U0 who created C00, C01 under C0 and C10 under C1 as follows and all processes under C0 and C1 belong to U0:

> 例如，假设cgroups C0和C1已被委派给用户U0，用户U0在C0下创建了C00、C01，在C1下创建了C10，如下所示，C0和C2下的所有进程都属于U0：

~~~~~~~~~~~~~ - C0 - C00
~ cgroup    ~      \ C01
~ hierarchy ~
~~~~~~~~~~~~~ - C1 - C10


Let\'s also say U0 wants to write the PID of a process which is currently in C10 into \"C00/cgroup.procs\". U0 has write access to the file; however, the common ancestor of the source cgroup C10 and the destination cgroup C00 is above the points of delegation and U0 would not have write access to its \"cgroup.procs\" files and thus the write will be denied with -EACCES.

> 还假设U0想将当前位于C10中的进程的PID写入“C00/cgroup.procs\”。U0有权写入该文件；然而，源cgroup C10和目标cgroup C00的共同祖先在委托点之上，并且U0将无法对其“cgroup.procs”文件进行写访问，因此使用-EACCES将拒绝写入。


For delegations to namespaces, containment is achieved by requiring that both the source and destination cgroups are reachable from the namespace of the process which is attempting the migration. If either is not reachable, the migration is rejected with -ENOENT.

> 对于命名空间的委派，通过要求源和目标cgroup都可以从尝试迁移的进程的命名空间访问来实现包含。如果其中一个不可访问，则使用-ENOENT拒绝迁移。

## Guidelines

### Organize Once and Control


Migrating a process across cgroups is a relatively expensive operation and stateful resources such as memory are not moved together with the process. This is an explicit design decision as there often exist inherent trade-offs between migration and various hot paths in terms of synchronization cost.

> 跨cgroups迁移进程是一项相对昂贵的操作，而且内存等有状态资源不会与进程一起移动。这是一个明确的设计决策，因为在同步成本方面，迁移和各种热路径之间往往存在固有的权衡。


As such, migrating processes across cgroups frequently as a means to apply different resource restrictions is discouraged. A workload should be assigned to a cgroup according to the system\'s logical and resource structure once on start-up. Dynamic adjustments to resource distribution can be made by changing controller configuration through the interface files.

> 因此，不鼓励将频繁跨cgroup迁移进程作为应用不同资源限制的手段。一旦启动，就应该根据系统的逻辑和资源结构将工作负载分配给一个cgroup。可以通过接口文件更改控制器配置来对资源分布进行动态调整。

### Avoid Name Collisions


Interface files for a cgroup and its children cgroups occupy the same directory and it is possible to create children cgroups which collide with interface files.

> cgroup及其子cgroup的接口文件占用相同的目录，并且可以创建与接口文件冲突的子cgroups。


All cgroup core interface files are prefixed with \"cgroup.\" and each controller\'s interface files are prefixed with the controller name and a dot. A controller\'s name is composed of lower case alphabets and \'\_\'s but never begins with an \'\_\' so it can be used as the prefix character for collision avoidance. Also, interface file names won\'t start or end with terms which are often used in categorizing workloads such as job, service, slice, unit or workload.

> 所有cgroup核心接口文件都以“cgroup.\”为前缀，每个控制器的接口文件以控制器名称和一个点为前缀。控制器的名称由小写字母和“”组成，但从不以“”开头，因此可以用作避免冲突的前缀字符。此外，接口文件名不会以常用于对工作负载（如作业、服务、切片、单元或工作负载）进行分类的术语开头或结尾。


cgroup doesn\'t do anything to prevent name collisions and it\'s the user\'s responsibility to avoid them.

> cgroup不会采取任何措施来防止名称冲突，避免这些冲突是用户的责任。

# Resource Distribution Models


cgroup controllers implement several resource distribution schemes depending on the resource type and expected use cases. This section describes major schemes in use along with their expected behaviors.

> cgroup控制器根据资源类型和预期用例实现几种资源分配方案。本节介绍了正在使用的主要方案及其预期行为。

## Weights


A parent\'s resource is distributed by adding up the weights of all active children and giving each the fraction matching the ratio of its weight against the sum. As only children which can make use of the resource at the moment participate in the distribution, this is work-conserving. Due to the dynamic nature, this model is usually used for stateless resources.

> 通过将所有活动子项的权重相加，并给出与其权重与总和之比相匹配的分数，来分配父项的资源。由于目前只有能够利用资源的儿童才能参与分配，这是节约工作。由于其动态特性，该模型通常用于无状态资源。


All weights are in the range \[1, 10000\] with the default at 100. This allows symmetric multiplicative biases in both directions at fine enough granularity while staying in the intuitive range.

> 所有权重都在\[110000\]的范围内，默认值为100。这允许在两个方向上以足够精细的粒度进行对称乘法偏差，同时保持在直观的范围内。


As long as the weight is in range, all configuration combinations are valid and there is no reason to reject configuration changes or process migrations.

> 只要权重在范围内，所有配置组合都是有效的，没有理由拒绝配置更改或处理迁移。


\"cpu.weight\" proportionally distributes CPU cycles to active children and is an example of this type.

> \“cpu.weight”按比例将cpu周期分配给活动的子级，就是这种类型的一个例子。

## Limits {#cgroupv2-limits-distributor}


A child can only consume up to the configured amount of the resource. Limits can be over-committed - the sum of the limits of children can exceed the amount of resource available to the parent.

> 子级最多只能消耗配置数量的资源。限制可能被过度承诺——子项限制的总和可能超过父项可用的资源量。


Limits are in the range \[0, max\] and defaults to \"max\", which is noop.

> 限制在\[0，max\]范围内，默认为“max\”，即noop。


As limits can be over-committed, all configuration combinations are valid and there is no reason to reject configuration changes or process migrations.

> 由于限制可能被过度提交，所有配置组合都是有效的，没有理由拒绝配置更改或处理迁移。


\"io.max\" limits the maximum BPS and/or IOPS that a cgroup can consume on an IO device and is an example of this type.

> \“io.max\”限制了一个cgroup在io设备上可以消耗的最大BPS和/或IOPS，这就是一个例子。

## Protections {#cgroupv2-protections-distributor}


A cgroup is protected up to the configured amount of the resource as long as the usages of all its ancestors are under their protected levels. Protections can be hard guarantees or best effort soft boundaries. Protections can also be over-committed in which case only up to the amount available to the parent is protected among children.

> 只要一个cgroup的所有祖先的使用都在其受保护级别之下，它就会受到资源配置量的保护。保护可以是硬保证，也可以是尽最大努力的软边界。保护也可能被过度承诺，在这种情况下，只有父母可获得的金额才能在孩子之间得到保护。

Protections are in the range \[0, max\] and defaults to 0, which is noop.


As protections can be over-committed, all configuration combinations are valid and there is no reason to reject configuration changes or process migrations.

> 由于保护可能被过度提交，所有配置组合都是有效的，没有理由拒绝配置更改或处理迁移。


\"memory.low\" implements best-effort memory protection and is an example of this type.

> \“memory.low”实现了尽力而为的内存保护，就是这种类型的一个例子。

## Allocations


A cgroup is exclusively allocated a certain amount of a finite resource. Allocations can\'t be over-committed - the sum of the allocations of children can not exceed the amount of resource available to the parent.

> 一个cgroup被独占地分配了一定数量的有限资源。分配不能过多-子级分配的总和不能超过父级可用的资源量。


Allocations are in the range \[0, max\] and defaults to 0, which is no resource.

> 分配在\[0，max\]范围内，默认为0，这不是资源。


As allocations can\'t be over-committed, some configuration combinations are invalid and should be rejected. Also, if the resource is mandatory for execution of processes, process migrations may be rejected.

> 由于分配不能被过度提交，一些配置组合是无效的，应该被拒绝。此外，如果资源对于进程的执行是强制性的，则进程迁移可能会被拒绝。


\"cpu.rt.max\" hard-allocates realtime slices and is an example of this type.

> \“cpu.rt.max\”硬分配实时切片，就是这种类型的一个例子。

# Interface Files

## Format


All interface files should be in one of the following formats whenever possible:

> 所有接口文件应尽可能采用以下格式之一：

New-line separated values
(when only one value can be written at once)

  VAL0\n
  VAL1\n
  ...

Space separated values
(when read-only or multiple values can be written at once)

  VAL0 VAL1 ...\n

Flat keyed

  KEY0 VAL0\n
  KEY1 VAL1\n
  ...

Nested keyed

  KEY0 SUB_KEY0=VAL00 SUB_KEY1=VAL01...
  KEY1 SUB_KEY0=VAL10 SUB_KEY1=VAL11...
  ...


For a writable file, the format for writing should generally match reading; however, controllers may allow omitting later fields or implement restricted shortcuts for most common use cases.

> 对于可写文件，写入的格式通常应与读取相匹配；然而，对于大多数常见的使用情况，控制器可以允许省略后面的字段或实现受限的快捷方式。


For both flat and nested keyed files, only the values for a single key can be written at a time. For nested keyed files, the sub key pairs may be specified in any order and not all pairs have to be specified.

> 对于平面和嵌套键控文件，一次只能写入单个键的值。对于嵌套键控文件，可以按任何顺序指定子键对，而不必指定所有对。

## Conventions

- Settings for a single feature should be contained in a single file.


- The root cgroup should be exempt from resource control and thus shouldn\'t have resource control interface files.

> -根cgroup应该不受资源控制，因此不应该有资源控制接口文件。


- The default time unit is microseconds. If a different unit is ever used, an explicit unit suffix must be present.

> -默认的时间单位是微秒。如果使用不同的单位，则必须存在明确的单位后缀。


- A parts-per quantity should use a percentage decimal with at least two digit fractional part - e.g. 13.40.

> -每个数量的零件应使用百分比小数，小数部分至少为两位数，例如13.40。


- If a controller implements weight based resource distribution, its interface file should be named \"weight\" and have the range \[1, 10000\] with 100 as the default. The values are chosen to allow enough and symmetric bias in both directions while keeping it intuitive (the default is 100%).

> -如果控制器实现基于权重的资源分配，则其接口文件应命名为“weight”，范围为\[110000\]，默认值为100。选择这些值以允许在两个方向上有足够的对称偏差，同时保持直观（默认值为100%）。


- If a controller implements an absolute resource guarantee and/or limit, the interface files should be named \"min\" and \"max\" respectively. If a controller implements best effort resource guarantee and/or limit, the interface files should be named \"low\" and \"high\" respectively.

> -如果控制器实现了绝对的资源保证和/或限制，则接口文件应分别命名为“min\”和“max\”。如果控制器实现尽最大努力的资源保证和/或限制，则接口文件应分别命名为“低”和“高”。


  In the above four control files, the special token \"max\" should be used to represent upward infinity for both reading and writing.

> 在上述四个控制文件中，应使用特殊标记“max\”来表示向上无穷大，用于读取和写入。


- If a setting has a configurable default value and keyed specific overrides, the default entry should be keyed with \"default\" and appear as the first entry in the file.

> -如果设置具有可配置的默认值和键控的特定覆盖，则默认条目应使用“default\”键控，并显示为文件中的第一个条目。


  The default value can be updated by writing either \"default \$VAL\" or \"\$VAL\".

> 可以通过写入\“default\$VAL\”或\“\$VAL”来更新默认值。


  When writing to update a specific override, \"default\" can be used as the value to indicate removal of the override. Override entries with \"default\" as the value must not appear when read.

> 当写入以更新特定的覆盖时，\“default\”可以用作指示删除覆盖的值。使用“default\”覆盖条目，因为读取时不得出现值。


  For example, a setting which is keyed by major:minor device numbers with integer values may look like the following:

> 例如，由具有整数值的主要：次要设备编号键入的设置可能如下所示：

  # cat cgroup-example-interface-file
  default 150
  8:0 300

  The default value can be updated by:

  # echo 125 cgroup-example-interface-file

  or:

  # echo "default 125" cgroup-example-interface-file

  An override can be set by:

  # echo "8:16 170" cgroup-example-interface-file

  and cleared by:

  # echo "8:0 default" cgroup-example-interface-file
  # cat cgroup-example-interface-file
  default 125
  8:16 170


- For events which are not very high frequency, an interface file \"events\" should be created which lists event key value pairs. Whenever a notifiable event happens, file modified event should be generated on the file.

> -对于频率不高的事件，应创建一个接口文件“events”，列出事件键值对。每当发生可通知事件时，都应在文件上生成文件修改事件。

## Core Interface Files

All cgroup core files are prefixed with \"cgroup.\"

cgroup.type

: A read-write single value file which exists on non-root cgroups.


When read, it indicates the current type of the cgroup, which can be one of the following values.

> 读取时，它指示cgroup的当前类型，可以是以下值之一。

-   \"domain\" : A normal valid domain cgroup.

-

    \"domain threaded\" : A threaded domain cgroup which is

    :   serving as the root of a threaded subtree.


-   \"domain invalid\" : A cgroup which is in an invalid state. It can\'t be populated or have controllers enabled. It may be allowed to become a threaded cgroup.

> -“域无效”：处于无效状态的cgroup。无法填充或启用控制器。它可以被允许成为一个线程cgroup。

-

    \"threaded\" : A threaded cgroup which is a member of a

    :   threaded subtree.


A cgroup can be turned into a threaded cgroup by writing \"threaded\" to this file.

> 通过将\“threeded”写入该文件，可以将cgroup转换为线程cgroup。

cgroup.procs


: A read-write new-line separated values file which exists on all cgroups.

> ：一个存在于所有cgroup上的读写新行分隔值文件。


When read, it lists the PIDs of all processes which belong to the cgroup one-per-line. The PIDs are not ordered and the same PID may show up more than once if the process got moved to another cgroup and then back or the PID got recycled while reading.

> 读取时，它每行列出一个属于cgroup的所有进程的PID。PID没有排序，如果进程被移到另一个组，然后返回，或者PID在读取时被回收，则同一PID可能会多次显示。


A PID can be written to migrate the process associated with the PID to the cgroup. The writer should match all of the following conditions.

> 可以编写PID以将与PID相关联的进程迁移到cgroup。编写器应符合以下所有条件。

-   It must have write access to the \"cgroup.procs\" file.

-   It must have write access to the \"cgroup.procs\" file of the common ancestor of the source and destination cgroups.

> -它必须具有对源和目标cgroups的公共祖先的“cgroup.procs”文件的写访问权限。


When delegating a sub-hierarchy, write access to this file should be granted along with the containing directory.

> 在委派子层次结构时，应授予对此文件的写入访问权限以及包含的目录。


In a threaded cgroup, reading this file fails with EOPNOTSUPP as all the processes belong to the thread root. Writing is supported and moves every thread of the process to the cgroup.

> 在线程cgroup中，由于所有进程都属于线程根，因此使用EOPNOTSUPP读取此文件失败。支持写入，并将进程的每个线程移动到cgroup。

cgroup.threads


: A read-write new-line separated values file which exists on all cgroups.

> ：一个存在于所有cgroup上的读写新行分隔值文件。


When read, it lists the TIDs of all threads which belong to the cgroup one-per-line. The TIDs are not ordered and the same TID may show up more than once if the thread got moved to another cgroup and then back or the TID got recycled while reading.

> 读取时，它每行列出一个属于cgroup的所有线程的TID。TID没有排序，如果线程被移到另一个cgroup然后返回，或者TID在读取时被回收，那么同一个TID可能会出现多次。


A TID can be written to migrate the thread associated with the TID to the cgroup. The writer should match all of the following conditions.

> 可以编写一个TID，将与该TID相关联的线程迁移到cgroup。编写器应符合以下所有条件。

-   It must have write access to the \"cgroup.threads\" file.

-

    The cgroup that the thread is currently in must be in the

    :   same resource domain as the destination cgroup.


-   It must have write access to the \"cgroup.procs\" file of the common ancestor of the source and destination cgroups.

> -它必须具有对源和目标cgroups的公共祖先的“cgroup.procs”文件的写访问权限。


When delegating a sub-hierarchy, write access to this file should be granted along with the containing directory.

> 在委派子层次结构时，应授予对此文件的写入访问权限以及包含的目录。

cgroup.controllers

: A read-only space separated values file which exists on all cgroups.


It shows space separated list of all controllers available to the cgroup. The controllers are not ordered.

> 它显示了cgroup可用的所有控制器的空格分隔列表。控制器未排序。

cgroup.subtree_control


: A read-write space separated values file which exists on all cgroups. Starts out empty.

> ：一个读写空间分隔的值文件，存在于所有cgroup上。一开始是空的。


When read, it shows space separated list of the controllers which are enabled to control resource distribution from the cgroup to its children.

> 读取时，它显示控制器的空格分隔列表，这些控制器可以控制从cgroup到其子组的资源分配。


Space separated list of controllers prefixed with \'+\' or \'-\' can be written to enable or disable controllers. A controller name prefixed with \'+\' enables the controller and \'-\' disables. If a controller appears more than once on the list, the last one is effective. When multiple enable and disable operations are specified, either all succeed or all fail.

> 以“+”或“-”为前缀的控制器的空格分隔列表可以用来启用或禁用控制器。前缀为“+”的控制器名称启用控制器，“-”禁用控制器。如果一个控制器在列表上出现多次，则最后一个控制器有效。当指定了多个启用和禁用操作时，要么全部成功，要么全部失败。

cgroup.events


: A read-only flat-keyed file which exists on non-root cgroups. The following entries are defined. Unless specified otherwise, a value change in this file generates a file modified event.

> ：存在于非根cgroups上的只读平面键控文件。定义了以下条目。除非另有规定，否则此文件中的值更改将生成文件修改事件。

    populated

    :   1 if the cgroup or its descendants contains any live processes; otherwise, 0.

    frozen

    :   1 if the cgroup is frozen; otherwise, 0.

cgroup.max.descendants

: A read-write single value files. The default is \"max\".


Maximum allowed number of descent cgroups. If the actual number of descendants is equal or larger, an attempt to create a new cgroup in the hierarchy will fail.

> 下降cgroups的最大允许数量。如果子体的实际数量等于或大于，则尝试在层次结构中创建新的cgroup将失败。

cgroup.max.depth

: A read-write single value files. The default is \"max\".


Maximum allowed descent depth below the current cgroup. If the actual descent depth is equal or larger, an attempt to create a new child cgroup will fail.

> 当前组下方允许的最大下降深度。如果实际下降深度等于或大于，则尝试创建新的子cgroup将失败。

cgroup.stat

: A read-only flat-keyed file with the following entries:

    nr_descendants

    :   Total number of visible descendant cgroups.

    nr_dying_descendants

    :   Total number of dying descendant cgroups. A cgroup becomes dying after being deleted by a user. The cgroup will remain in dying state for some time undefined time (which can depend on system load) before being completely destroyed.

        A process can\'t enter a dying cgroup under any circumstances, a dying cgroup can\'t revive.

        A dying cgroup can consume system resources not exceeding limits, which were active at the moment of cgroup deletion.

cgroup.freeze


: A read-write single value file which exists on non-root cgroups. Allowed values are \"0\" and \"1\". The default is \"0\".

> ：存在于非根cgroups上的读写单值文件。允许的值为“0\”和“1\”。默认值为“0\”。


Writing \"1\" to the file causes freezing of the cgroup and all descendant cgroups. This means that all belonging processes will be stopped and will not run until the cgroup will be explicitly unfrozen. Freezing of the cgroup may take some time; when this action is completed, the \"frozen\" value in the cgroup.events control file will be updated to \"1\" and the corresponding notification will be issued.

> 将“1”写入文件会导致cgroup和所有子代cgroup冻结。这意味着所有属于的进程都将停止，并且在cgroup明确解除冻结之前不会运行。冻结cgroup可能需要一些时间；此操作完成后，cgroup.events控制文件中的“冻结”值将更新为“1”，并发出相应的通知。


A cgroup can be frozen either by its own settings, or by settings of any ancestor cgroups. If any of ancestor cgroups is frozen, the cgroup will remain frozen.

> 一个cgroup可以通过它自己的设置冻结，也可以通过任何祖先cgroup的设置冻结。如果任何祖先cgroups被冻结，则该cgroup将保持冻结状态。


Processes in the frozen cgroup can be killed by a fatal signal. They also can enter and leave a frozen cgroup: either by an explicit move by a user, or if freezing of the cgroup races with fork(). If a process is moved to a frozen cgroup, it stops. If a process is moved out of a frozen cgroup, it becomes running.

> 冻结的cgroup中的进程可能会被致命信号杀死。他们也可以进入和离开冻结的cgroup：通过用户的显式移动，或者如果cgroup的冻结与fork（）竞争。如果进程被移动到冻结的cgroup，它将停止。如果一个进程从冻结的cgroup中移出，它将变为正在运行。


Frozen status of a cgroup doesn\'t affect any cgroup tree operations: it\'s possible to delete a frozen (and empty) cgroup, as well as create new sub-cgroups.

> cgroup的冻结状态不会影响任何cgroup树操作：可以删除冻结的（空的）cgroup，也可以创建新的子cgroup。

cgroup.kill


: A write-only single value file which exists in non-root cgroups. The only allowed value is \"1\".

> ：存在于非根cgroups中的仅写单值文件。唯一允许的值是“1”。


Writing \"1\" to the file causes the cgroup and all descendant cgroups to be killed. This means that all processes located in the affected cgroup tree will be killed via SIGKILL.

> 将“1”写入文件会导致cgroup和所有子代cgroup被终止。这意味着位于受影响的cgroup树中的所有进程都将通过SIGKILL终止。


Killing a cgroup tree will deal with concurrent forks appropriately and is protected against migrations.

> 杀死一个cgroup树将适当地处理并发分叉，并防止迁移。


In a threaded cgroup, writing this file fails with EOPNOTSUPP as killing cgroups is a process directed operation, i.e. it affects the whole thread-group.

> 在线程cgroup中，使用EOPNOTSUPP写入此文件失败，因为杀死cgroups是一个进程导向的操作，即它会影响整个线程组。

cgroup.pressure


: A read-write single value file that allowed values are \"0\" and \"1\". The default is \"1\".

> ：允许值为“0\”和“1\”的读写单值文件。默认值为“1”。


Writing \"0\" to the file will disable the cgroup PSI accounting. Writing \"1\" to the file will re-enable the cgroup PSI accounting.

> 将“0\”写入文件将禁用cgroup PSI记帐。将“1”写入文件将重新启用cgroup PSI记帐。


This control attribute is not hierarchical, so disable or enable PSI accounting in a cgroup does not affect PSI accounting in descendants and doesn\'t need pass enablement via ancestors from root.

> 此控制属性不是分层的，因此在cgroup中禁用或启用PSI记帐不会影响后代中的PSI记帐，也不需要从根通过祖先传递启用。


The reason this control attribute exists is that PSI accounts stalls for each cgroup separately and aggregates it at each level of the hierarchy. This may cause non-negligible overhead for some workloads when under deep level of the hierarchy, in which case this control attribute can be used to disable PSI accounting in the non-leaf cgroups.

> 存在此控制属性的原因是PSI分别为每个cgroup帐户暂停，并在层次结构的每个级别聚合它。当处于层次结构的深层时，这可能会对某些工作负载造成不可忽略的开销，在这种情况下，此控制属性可用于禁用非叶cgroups中的PSI记帐。

irq.pressure

: A read-write nested-keyed file.


Shows pressure stall information for IRQ/SOFTIRQ. See `Documentation/accounting/psi.rst <psi>`{.interpreted-text role="ref"} for details.

> 显示IRQ/SOFTIRQ的压力失速信息。有关详细信息，请参阅`Documentation/accounting/pis.rst<psi>`｛.explored text role=“ref”｝。

# Controllers

## CPU {#cgroup-v2-cpu}


The \"cpu\" controllers regulates distribution of CPU cycles. This controller implements weight and absolute bandwidth limit models for normal scheduling policy and absolute bandwidth allocation model for realtime scheduling policy.

> “cpu”控制器调节cpu周期的分布。该控制器实现了正常调度策略的权重和绝对带宽限制模型以及实时调度策略的绝对带宽分配模型。


In all the above models, cycles distribution is defined only on a temporal base and it does not account for the frequency at which tasks are executed. The (optional) utilization clamping support allows to hint the schedutil cpufreq governor about the minimum desired frequency which should always be provided by a CPU, as well as the maximum desired frequency, which should not be exceeded by a CPU.

> 在所有上述模型中，周期分布仅在时间基础上定义，并且它不考虑执行任务的频率。（可选）利用率箝位支持允许向schedutil cpufreq调速器提示CPU应始终提供的最小期望频率，以及CPU不应超过的最大期望频率。


WARNING: cgroup2 doesn\'t yet support control of realtime processes and the cpu controller can only be enabled when all RT processes are in the root cgroup. Be aware that system management software may already have placed RT processes into nonroot cgroups during the system boot process, and these processes may need to be moved to the root cgroup before the cpu controller can be enabled.

> 警告：cgroup2还不支持实时进程的控制，并且只有当所有RT进程都在根cgroup中时，才能启用cpu控制器。请注意，在系统引导过程中，系统管理软件可能已经将RT进程放置到非根cgroup中，并且在启用cpu控制器之前，这些进程可能需要移动到根cgroup。

### CPU Interface Files

All time durations are in microseconds.

cpu.stat


: A read-only flat-keyed file. This file exists whether the controller is enabled or not.

> ：只读平面键控文件。无论控制器是否启用，此文件都存在。

It always reports the following three stats:

-   usage_usec
-   user_usec
-   system_usec

and the following five when the controller is enabled:

-   nr_periods
-   nr_throttled
-   throttled_usec
-   nr_bursts
-   burst_usec

cpu.weight


: A read-write single value file which exists on non-root cgroups. The default is \"100\".

> ：存在于非根cgroups上的读写单值文件。默认值为“100”。

The weight in the range \[1, 10000\].

cpu.weight.nice


: A read-write single value file which exists on non-root cgroups. The default is \"0\".

> ：存在于非根cgroups上的读写单值文件。默认值为“0\”。

The nice value is in the range \[-20, 19\].


This interface file is an alternative interface for \"cpu.weight\" and allows reading and setting weight using the same values used by nice(2). Because the range is smaller and granularity is coarser for the nice values, the read value is the closest approximation of the current weight.

> 该接口文件是“cpu.weight”的替代接口，允许使用nice（2）使用的相同值读取和设置权重。因为漂亮值的范围较小，粒度较粗，所以读取值是当前权重的最接近值。

cpu.max


: A read-write two value file which exists on non-root cgroups. The default is \"max 100000\".

> ：存在于非根cgroups上的读写双值文件。默认值为“最大100000”。

The maximum bandwidth limit. It\'s in the following format:

    $MAX $PERIOD


which indicates that the group may consume up to \$MAX in each \$PERIOD duration. \"max\" for \$MAX indicates no limit. If only one number is written, \$MAX is updated.

> 这表明该组在每个\$PERIOD持续时间内可以消耗高达\$MAX\\$max的“max\”表示没有限制。如果只写入一个数字，则更新\$MAX。

cpu.max.burst


: A read-write single value file which exists on non-root cgroups. The default is \"0\".

> ：存在于非根cgroups上的读写单值文件。默认值为“0\”。

The burst in the range \[0, \$MAX\].

cpu.pressure

: A read-write nested-keyed file.


Shows pressure stall information for CPU. See `Documentation/accounting/psi.rst <psi>`{.interpreted-text role="ref"} for details.

> 显示CPU的压力失速信息。有关详细信息，请参阅`Documentation/accounting/pis.rst<psi>`｛.explored text role=“ref”｝。

cpu.uclamp.min


: A read-write single value file which exists on non-root cgroups. The default is \"0\", i.e. no utilization boosting.

> ：存在于非根cgroups上的读写单值文件。默认值为“0\”，即没有提高利用率。


The requested minimum utilization (protection) as a percentage rational number, e.g. 12.34 for 12.34%.

> 要求的最低利用率（保护）是一个合理的百分比，例如12.34%为12.34。


This interface allows reading and setting minimum utilization clamp values similar to the sched_setattr(2). This minimum utilization value is used to clamp the task specific minimum utilization clamp.

> 该接口允许读取和设置与sched_setattr（2）类似的最小利用率箝位值。此最小利用率值用于钳制任务特定的最小利用率钳制。


The requested minimum utilization (protection) is always capped by the current value for the maximum utilization (limit), i.e. [cpu.uclamp.max]{.title-ref}.

> 请求的最小利用率（保护）总是由最大利用率（限制）的当前值来限制，即[cpu.uclamp.max]{.title-ref}。

cpu.uclamp.max


: A read-write single value file which exists on non-root cgroups. The default is \"max\". i.e. no utilization capping

> ：存在于非根cgroups上的读写单值文件。默认值为“最大值”。即没有使用上限


The requested maximum utilization (limit) as a percentage rational number, e.g. 98.76 for 98.76%.

> 请求的最大利用率（限制），以百分比有理数表示，例如98.76表示98.76%。


This interface allows reading and setting maximum utilization clamp values similar to the sched_setattr(2). This maximum utilization value is used to clamp the task specific maximum utilization clamp.

> 该接口允许读取和设置与sched_setattr（2）类似的最大利用率箝位值。此最大利用率值用于钳制任务特定的最大利用率钳制。

## Memory


The \"memory\" controller regulates distribution of memory. Memory is stateful and implements both limit and protection models. Due to the intertwining between memory usage and reclaim pressure and the stateful nature of memory, the distribution model is relatively complex.

> “内存”控制器调节内存的分配。内存是有状态的，同时实现了限制和保护模型。由于内存使用和回收压力以及内存的状态性之间的交织，分布模型相对复杂。


While not completely water-tight, all major memory usages by a given cgroup are tracked so that the total memory consumption can be accounted and controlled to a reasonable extent. Currently, the following types of memory usages are tracked.

> 虽然不是完全防水的，但会跟踪给定cgroup的所有主要内存使用情况，以便在合理的范围内计算和控制总内存消耗。目前，跟踪以下类型的内存使用情况。

- Userland memory - page cache and anonymous memory.
- Kernel data structures such as dentries and inodes.
- TCP socket buffers.

The above list may expand in the future for better coverage.

### Memory Interface Files


All memory amounts are in bytes. If a value which is not aligned to PAGE_SIZE is written, the value may be rounded up to the closest PAGE_SIZE multiple when read back.

> 所有内存量均以字节为单位。如果写入了与PAGE_SIZE不对齐的值，则在读回时，该值可以四舍五入到最接近的PAGE_SIZE倍数。

memory.current

: A read-only single value file which exists on non-root cgroups.


The total amount of memory currently being used by the cgroup and its descendants.

> cgroup及其子代当前正在使用的内存总量。

memory.min


: A read-write single value file which exists on non-root cgroups. The default is \"0\".

> ：存在于非根cgroups上的读写单值文件。默认值为“0\”。


Hard memory protection. If the memory usage of a cgroup is within its effective min boundary, the cgroup\'s memory won\'t be reclaimed under any conditions. If there is no unprotected reclaimable memory available, OOM killer is invoked. Above the effective min boundary (or effective low boundary if it is higher), pages are reclaimed proportionally to the overage, reducing reclaim pressure for smaller overages.

> 硬内存保护。如果一个cgroup的内存使用量在其有效最小边界内，那么在任何情况下都不会回收该cggroup的内存。如果没有不受保护的可回收内存可用，就会调用OOM杀手。在有效最小边界以上（如果更高，则为有效低边界），页面将按比例回收，从而降低较小超额的回收压力。


Effective min boundary is limited by memory.min values of all ancestor cgroups. If there is memory.min overcommitment (child cgroup or cgroups are requiring more protected memory than parent will allow), then each child cgroup will get the part of parent\'s protection proportional to its actual memory usage below memory.min.

> 有效最小边界受所有祖先cgroups的memory.min值的限制。如果存在memory.min过度使用（子cgroup或cgroups需要比父cgroup允许的更多的受保护内存），则每个子cgroups将获得与低于memory.min的实际内存使用量成比例的父保护部分。


Putting more memory than generally available under this protection is discouraged and may lead to constant OOMs.

> 不鼓励将比通常可用的内存更多的内存置于这种保护之下，并可能导致持续的OOM。


If a memory cgroup is not populated with processes, its memory.min is ignored.

> 如果内存cgroup未填充进程，则会忽略其memory.min。

memory.low


: A read-write single value file which exists on non-root cgroups. The default is \"0\".

> ：存在于非根cgroups上的读写单值文件。默认值为“0\”。


Best-effort memory protection. If the memory usage of a cgroup is within its effective low boundary, the cgroup\'s memory won\'t be reclaimed unless there is no reclaimable memory available in unprotected cgroups. Above the effective low boundary (or effective min boundary if it is higher), pages are reclaimed proportionally to the overage, reducing reclaim pressure for smaller overages.

> 尽力保护记忆。如果一个cgroup的内存使用率在其有效的低边界内，则除非在未受保护的cgroups中没有可用的可回收内存，否则不会回收该cggroup的内存。在有效的低边界（或有效的最小边界，如果它更高）以上，页面将按比例回收，以减少较小超额的回收压力。


Effective low boundary is limited by memory.low values of all ancestor cgroups. If there is memory.low overcommitment (child cgroup or cgroups are requiring more protected memory than parent will allow), then each child cgroup will get the part of parent\'s protection proportional to its actual memory usage below memory.low.

> 有效的低边界受到内存的限制。所有祖先cgroups的值都很低。如果内存不足（子组或子组所需的受保护内存比父组所允许的多），则每个子组将获得与内存不足的实际内存使用量成比例的父组保护部分。


Putting more memory than generally available under this protection is discouraged.

> 不鼓励将比通常可用的内存更多的内存置于这种保护之下。

memory.high


: A read-write single value file which exists on non-root cgroups. The default is \"max\".

> ：存在于非根cgroups上的读写单值文件。默认值为“最大值”。


Memory usage throttle limit. If a cgroup\'s usage goes over the high boundary, the processes of the cgroup are throttled and put under heavy reclaim pressure.

> 内存使用限制。如果一个cgroup的使用超过了上限，则该cgroup中的进程将受到抑制，并承受巨大的回收压力。


Going over the high limit never invokes the OOM killer and under extreme conditions the limit may be breached. The high limit should be used in scenarios where an external process monitors the limited cgroup to alleviate heavy reclaim pressure.

> 超过上限永远不会引发OOM杀手，在极端条件下，可能会突破上限。高限值应用于外部流程监控有限的cgroup以缓解沉重回收压力的情况。

memory.max


: A read-write single value file which exists on non-root cgroups. The default is \"max\".

> ：存在于非根cgroups上的读写单值文件。默认值为“最大值”。


Memory usage hard limit. This is the main mechanism to limit memory usage of a cgroup. If a cgroup\'s memory usage reaches this limit and can\'t be reduced, the OOM killer is invoked in the cgroup. Under certain circumstances, the usage may go over the limit temporarily.

> 内存使用硬限制。这是限制cgroup内存使用的主要机制。如果一个cgroup的内存使用量达到了这个极限并且无法减少，那么就会在cgroup中调用OOM杀手。在某些情况下，使用量可能会暂时超过限制。


In default configuration regular 0-order allocations always succeed unless OOM killer chooses current task as a victim.

> 在默认配置中，常规的0阶分配总是成功的，除非OOM杀手选择当前任务作为牺牲品。


Some kinds of allocations don\'t invoke the OOM killer. Caller could retry them differently, return into userspace as -ENOMEM or silently ignore in cases like disk readahead.

> 某些类型的分配不会调用OOM杀手。调用方可以以不同的方式重试，以-ENOMM的形式返回到用户空间，或者在磁盘预读等情况下静默忽略。

memory.reclaim

: A write-only nested-keyed file which exists for all cgroups.


This is a simple interface to trigger memory reclaim in the target cgroup.

> 这是一个在目标cgroup中触发内存回收的简单接口。


This file accepts a single key, the number of bytes to reclaim. No nested keys are currently supported.

> 此文件接受一个密钥，即要回收的字节数。当前不支持嵌套键。

Example:

    echo "1G" memory.reclaim


The interface can be later extended with nested keys to configure the reclaim behavior. For example, specify the type of memory to reclaim from (anon, file, ..).

> 该接口稍后可以使用嵌套键进行扩展，以配置回收行为。例如，指定要回收的内存类型（anon、file、..）。


Please note that the kernel can over or under reclaim from the target cgroup. If less bytes are reclaimed than the specified amount, -EAGAIN is returned.

> 请注意，内核可以从目标cgroup中回收或回收不足。如果回收的字节数少于指定的数量，则返回-EAGAIN。


Please note that the proactive reclaim (triggered by this interface) is not meant to indicate memory pressure on the memory cgroup. Therefore socket memory balancing triggered by the memory reclaim normally is not exercised in this case. This means that the networking layer will not adapt based on reclaim induced by memory.reclaim.

> 请注意，主动回收（由该接口触发）并不意味着指示内存cgroup上的内存压力。因此，在这种情况下，通常不会执行由内存回收触发的套接字内存平衡。这意味着网络层将不会根据memory.reclaim引起的回收进行调整。

memory.peak

: A read-only single value file which exists on non-root cgroups.


The max memory usage recorded for the cgroup and its descendants since the creation of the cgroup.

> 自创建cgroup以来，为cgroup及其子代记录的最大内存使用量。

memory.oom.group


: A read-write single value file which exists on non-root cgroups. The default value is \"0\".

> ：存在于非根cgroups上的读写单值文件。默认值为“0\”。


Determines whether the cgroup should be treated as an indivisible workload by the OOM killer. If set, all tasks belonging to the cgroup or to its descendants (if the memory cgroup is not a leaf cgroup) are killed together or not at all. This can be used to avoid partial kills to guarantee workload integrity.

> 确定cgroup是否应被OOM杀手视为不可分割的工作负载。如果设置，则属于cgroup或其子代（如果内存cgroup不是叶cgroup）的所有任务都会一起终止或根本不终止。这可以用来避免部分终止，以保证工作负载的完整性。


Tasks with the OOM protection (oom_score_adj set to -1000) are treated as an exception and are never killed.

> 具有OOM保护（OOM_score_adj设置为-1000）的任务被视为异常，并且永远不会被终止。


If the OOM killer is invoked in a cgroup, it\'s not going to kill any tasks outside of this cgroup, regardless memory.oom.group values of ancestor cgroups.

> 如果OOM杀手在一个cgroup中被调用，它不会杀死这个cgroup之外的任何任务，无论祖先cgroups的memory.OOM.group值如何。

memory.events


: A read-only flat-keyed file which exists on non-root cgroups. The following entries are defined. Unless specified otherwise, a value change in this file generates a file modified event.

> ：存在于非根cgroups上的只读平面键控文件。定义了以下条目。除非另有规定，否则此文件中的值更改将生成文件修改事件。


Note that all fields in this file are hierarchical and the file modified event can be generated due to an event down the hierarchy. For the local events at the cgroup level see memory.events.local.

> 请注意，此文件中的所有字段都是分层的，文件修改事件可能是由于层次结构中的事件而生成的。有关cgroup级别的本地事件，请参阅memory.events.local。

    low

    :   The number of times the cgroup is reclaimed due to high memory pressure even though its usage is under the low boundary. This usually indicates that the low boundary is over-committed.

    high

    :   The number of times processes of the cgroup are throttled and routed to perform direct memory reclaim because the high memory boundary was exceeded. For a cgroup whose memory usage is capped by the high limit rather than global memory pressure, this event\'s occurrences are expected.

    max

    :   The number of times the cgroup\'s memory usage was about to go over the max boundary. If direct reclaim fails to bring it down, the cgroup goes to OOM state.

    oom

    :   The number of time the cgroup\'s memory usage was reached the limit and allocation was about to fail.

        This event is not raised if the OOM killer is not considered as an option, e.g. for failed high-order allocations or if caller asked to not retry attempts.

    oom_kill

    :   The number of processes belonging to this cgroup killed by any kind of OOM killer.

        oom_group_kill
        >
        :   The number of times a group OOM has occurred.

memory.events.local


: Similar to memory.events but the fields in the file are local to the cgroup i.e. not hierarchical. The file modified event generated on this file reflects only the local events.

> ：类似于memory.events，但文件中的字段是cgroup的本地字段，即不具有层次结构。在此文件上生成的文件修改事件仅反映本地事件。

memory.stat

: A read-only flat-keyed file which exists on non-root cgroups.


This breaks down the cgroup\'s memory footprint into different types of memory, type-specific details, and other information on the state and past events of the memory management system.

> 这将cgroup的内存占用分解为不同类型的内存、特定类型的详细信息以及关于内存管理系统的状态和过去事件的其他信息。

All memory amounts are in bytes.


The entries are ordered to be human readable, and new entries can show up in the middle. Don\'t rely on items remaining in a fixed position; use the keys to look up specific values!

> 条目被排序为人类可读，新条目可以显示在在中间。不要依赖于固定位置的物品；使用键查找特定值！


If the entry has no per-node counter (or not show in the memory.numa_stat). We use \'npn\' (non-per-node) as the tag to indicate that it will not show in the memory.numa_stat.

> 如果条目没有每个节点计数器（或没有显示在内存.numa_stat中）。我们使用\'npn\'（non-per-node）作为标记来指示它不会显示在内存.numa_stat。

    anon

    :   Amount of memory used in anonymous mappings such as brk(), sbrk(), and mmap(MAP_ANONYMOUS)

    file

    :   Amount of memory used to cache filesystem data, including tmpfs and shared memory.

    kernel (npn)

    :   Amount of total kernel memory, including (kernel_stack, pagetables, percpu, vmalloc, slab) in addition to other kernel memory use cases.

    kernel_stack

    :   Amount of memory allocated to kernel stacks.

    pagetables

    :   Amount of memory allocated for page tables.

    sec_pagetables

    :   Amount of memory allocated for secondary page tables, this currently includes KVM mmu allocations on x86 and arm64.

    percpu (npn)

    :   Amount of memory used for storing per-cpu kernel data structures.

    sock (npn)

    :   Amount of memory used in network transmission buffers

    vmalloc (npn)

    :   Amount of memory used for vmap backed memory.

    shmem

    :   Amount of cached filesystem data that is swap-backed, such as tmpfs, shm segments, shared anonymous mmap()s

    zswap

    :   Amount of memory consumed by the zswap compression backend.

    zswapped

    :   Amount of application memory swapped out to zswap.

    file_mapped

    :   Amount of cached filesystem data mapped with mmap()

    file_dirty

    :   Amount of cached filesystem data that was modified but not yet written back to disk

    file_writeback

    :   Amount of cached filesystem data that was modified and is currently being written back to disk

    swapcached

    :   Amount of swap cached in memory. The swapcache is accounted against both memory and swap usage.

    anon_thp

    :   Amount of memory used in anonymous mappings backed by transparent hugepages

    file_thp

    :   Amount of cached filesystem data backed by transparent hugepages

    shmem_thp

    :   Amount of shm, tmpfs, shared anonymous mmap()s backed by transparent hugepages

    inactive_anon, active_anon, inactive_file, active_file, unevictable

    :   Amount of memory, swap-backed and filesystem-backed, on the internal memory management lists used by the page reclaim algorithm.

        As these represent internal list state (eg. shmem pages are on anon memory management lists), inactive_foo + active_foo may not be equal to the value for the foo counter, since the foo counter is type-based, not list-based.

    slab_reclaimable

    :   Part of \"slab\" that might be reclaimed, such as dentries and inodes.

    slab_unreclaimable

    :   Part of \"slab\" that cannot be reclaimed on memory pressure.

    slab (npn)

    :   Amount of memory used for storing in-kernel data structures.

    workingset_refault_anon

    :   Number of refaults of previously evicted anonymous pages.

    workingset_refault_file

    :   Number of refaults of previously evicted file pages.

    workingset_activate_anon

    :   Number of refaulted anonymous pages that were immediately activated.

    workingset_activate_file

    :   Number of refaulted file pages that were immediately activated.

    workingset_restore_anon

    :   Number of restored anonymous pages which have been detected as an active workingset before they got reclaimed.

    workingset_restore_file

    :   Number of restored file pages which have been detected as an active workingset before they got reclaimed.

    workingset_nodereclaim

    :   Number of times a shadow node has been reclaimed

    pgscan (npn)

    :   Amount of scanned pages (in an inactive LRU list)

    pgsteal (npn)

    :   Amount of reclaimed pages

    pgscan_kswapd (npn)

    :   Amount of scanned pages by kswapd (in an inactive LRU list)

    pgscan_direct (npn)

    :   Amount of scanned pages directly (in an inactive LRU list)

    pgscan_khugepaged (npn)

    :   Amount of scanned pages by khugepaged (in an inactive LRU list)

    pgsteal_kswapd (npn)

    :   Amount of reclaimed pages by kswapd

    pgsteal_direct (npn)

    :   Amount of reclaimed pages directly

    pgsteal_khugepaged (npn)

    :   Amount of reclaimed pages by khugepaged

    pgfault (npn)

    :   Total number of page faults incurred

    pgmajfault (npn)

    :   Number of major page faults incurred

    pgrefill (npn)

    :   Amount of scanned pages (in an active LRU list)

    pgactivate (npn)

    :   Amount of pages moved to the active LRU list

    pgdeactivate (npn)

    :   Amount of pages moved to the inactive LRU list

    pglazyfree (npn)

    :   Amount of pages postponed to be freed under memory pressure

    pglazyfreed (npn)

    :   Amount of reclaimed lazyfree pages

    thp_fault_alloc (npn)

    :   Number of transparent hugepages which were allocated to satisfy a page fault. This counter is not present when CONFIG_TRANSPARENT_HUGEPAGE is not set.

    thp_collapse_alloc (npn)

    :   Number of transparent hugepages which were allocated to allow collapsing an existing range of pages. This counter is not present when CONFIG_TRANSPARENT_HUGEPAGE is not set.

    thp_swpout (npn)

    :   Number of transparent hugepages which are swapout in one piece without splitting.

    thp_swpout_fallback (npn)

    :   Number of transparent hugepages which were split before swapout. Usually because failed to allocate some continuous swap space for the huge page.

memory.numa_stat

: A read-only nested-keyed file which exists on non-root cgroups.


This breaks down the cgroup\'s memory footprint into different types of memory, type-specific details, and other information per node on the state of the memory management system.

> 这将cgroup的内存占用分解为不同类型的内存、类型特定的详细信息以及关于内存管理系统状态的每个节点的其他信息。


This is useful for providing visibility into the NUMA locality information within an memcg since the pages are allowed to be allocated from any physical node. One of the use case is evaluating application performance by combining this information with the application\'s CPU allocation.

> 这对于提供对memcg内的NUMA位置信息的可见性是有用的，因为允许从任何物理节点分配页面。其中一个用例是通过将这些信息与应用程序的CPU分配相结合来评估应用程序性能。

All memory amounts are in bytes.

The output format of memory.numa_stat is:

    type N0=<bytes in node 0N1=<bytes in node 1...


The entries are ordered to be human readable, and new entries can show up in the middle. Don\'t rely on items remaining in a fixed position; use the keys to look up specific values!

> 条目被排序为人类可读，新条目可以显示在在中间。不要依赖于固定位置的物品；使用键查找特定值！

The entries can refer to the memory.stat.

memory.swap.current

: A read-only single value file which exists on non-root cgroups.


The total amount of swap currently being used by the cgroup and its descendants.

> cgroup及其子代当前正在使用的交换总量。

memory.swap.high


: A read-write single value file which exists on non-root cgroups. The default is \"max\".

> ：存在于非根cgroups上的读写单值文件。默认值为“最大值”。


Swap usage throttle limit. If a cgroup\'s swap usage exceeds this limit, all its further allocations will be throttled to allow userspace to implement custom out-of-memory procedures.

> 交换使用限制。如果一个cgroup的交换使用量超过了这个限制，它的所有进一步分配都将被限制，以允许用户空间实现自定义的内存不足过程。


This limit marks a point of no return for the cgroup. It is NOT designed to manage the amount of swapping a workload does during regular operation. Compare to memory.swap.max, which prohibits swapping past a set amount, but lets the cgroup continue unimpeded as long as other memory can be reclaimed.

> 这个限制标志着cgroup的一个不归路点。它并不是为了管理工作负载在常规操作期间的交换量而设计的。与memory.swap.max相比，后者禁止交换超过设定的数量，但只要其他内存可以回收，cgroup就可以不受阻碍地继续。

Healthy workloads are not expected to reach this limit.

memory.swap.peak

: A read-only single value file which exists on non-root cgroups.


The max swap usage recorded for the cgroup and its descendants since the creation of the cgroup.

> 自创建cgroup以来记录的cgroup及其子代的最大交换使用量。

memory.swap.max


: A read-write single value file which exists on non-root cgroups. The default is \"max\".

> ：存在于非根cgroups上的读写单值文件。默认值为“最大值”。


Swap usage hard limit. If a cgroup\'s swap usage reaches this limit, anonymous memory of the cgroup will not be swapped out.

> 交换使用硬限制。如果cgroup的交换使用量达到此限制，则不会交换出该cgroup中的匿名内存。

memory.swap.events


: A read-only flat-keyed file which exists on non-root cgroups. The following entries are defined. Unless specified otherwise, a value change in this file generates a file modified event.

> ：存在于非根cgroups上的只读平面键控文件。定义了以下条目。除非另有规定，否则此文件中的值更改将生成文件修改事件。

    high

    :   The number of times the cgroup\'s swap usage was over the high threshold.

    max

    :   The number of times the cgroup\'s swap usage was about to go over the max boundary and swap allocation failed.

    fail

    :   The number of times swap allocation failed either because of running out of swap system-wide or max limit.


When reduced under the current usage, the existing swap entries are reclaimed gradually and the swap usage may stay higher than the limit for an extended period of time. This reduces the impact on the workload and memory management.

> 当在当前使用量下减少时，现有交换条目会逐渐回收，并且交换使用量可能会在较长的一段时间内保持高于限制。这减少了对工作负载和内存管理的影响。

memory.zswap.current

: A read-only single value file which exists on non-root cgroups.

The total amount of memory consumed by the zswap compression backend.

memory.zswap.max


: A read-write single value file which exists on non-root cgroups. The default is \"max\".

> ：存在于非根cgroups上的读写单值文件。默认值为“最大值”。


Zswap usage hard limit. If a cgroup\'s zswap pool reaches this limit, it will refuse to take any more stores before existing entries fault back in or are written out to disk.

> Zswap使用硬限制。如果cgroup的zswap池达到此限制，它将拒绝在现有条目故障返回或写入磁盘之前获取更多存储。

memory.pressure

: A read-only nested-keyed file.


Shows pressure stall information for memory. See `Documentation/accounting/psi.rst <psi>`{.interpreted-text role="ref"} for details.

> 显示内存的压力失速信息。有关详细信息，请参阅`Documentation/accounting/pis.rst<psi>`｛.explored text role=“ref”｝。

### Usage Guidelines


\"memory.high\" is the main mechanism to control memory usage. Over-committing on high limit (sum of high limits \available memory) and letting global memory pressure to distribute memory according to usage is a viable strategy.

> \“memory.high”是控制内存使用的主要机制。对高限制（高限制\可用内存的总和）过度承诺，并让全局内存压力根据使用情况分配内存是一种可行的策略。


Because breach of the high limit doesn\'t trigger the OOM killer but throttles the offending cgroup, a management agent has ample opportunities to monitor and take appropriate actions such as granting more memory or terminating the workload.

> 因为违反高限制不会触发OOM杀手，但会抑制有问题的cgroup，所以管理代理有足够的机会进行监控并采取适当的操作，如授予更多内存或终止工作负载。


Determining whether a cgroup has enough memory is not trivial as memory usage doesn\'t indicate whether the workload can benefit from more memory. For example, a workload which writes data received from network to a file can use all available memory but can also operate as performant with a small amount of memory. A measure of memory pressure - how much the workload is being impacted due to lack of memory - is necessary to determine whether a workload needs more memory; unfortunately, memory pressure monitoring mechanism isn\'t implemented yet.

> 确定一个cgroup是否有足够的内存并不是一件小事，因为内存使用并不能表明工作负载是否可以从更多的内存中受益。例如，将从网络接收到的数据写入文件的工作负载可以使用所有可用的内存，但也可以使用少量内存进行性能操作。内存压力的衡量标准——工作负载由于内存不足而受到的影响有多大——对于确定工作负载是否需要更多内存是必要的；不幸的是，内存压力监测机制尚未实现。

### Memory Ownership


A memory area is charged to the cgroup which instantiated it and stays charged to the cgroup until the area is released. Migrating a process to a different cgroup doesn\'t move the memory usages that it instantiated while in the previous cgroup to the new cgroup.

> 内存区域被充电到实例化它的cgroup，并保持充电到cgroup直到该区域被释放。将进程迁移到另一个cgroup不会将其在前一个cggroup中实例化的内存使用情况移动到新的cgroup。


A memory area may be used by processes belonging to different cgroups. To which cgroup the area will be charged is in-deterministic; however, over time, the memory area is likely to end up in a cgroup which has enough memory allowance to avoid high reclaim pressure.

> 属于不同cgroup的进程可以使用存储器区域。该区域将被充电到哪个cgroup是确定的；然而，随着时间的推移，内存区域很可能最终会进入一个具有足够内存余量的cgroup，以避免高回收压力。


If a cgroup sweeps a considerable amount of memory which is expected to be accessed repeatedly by other cgroups, it may make sense to use POSIX_FADV_DONTNEED to relinquish the ownership of memory areas belonging to the affected files to ensure correct memory ownership.

> 如果一个cgroup扫描了预计会被其他cgroup重复访问的大量内存，那么使用POSIX_FADV_ONTNEED放弃属于受影响文件的内存区域的所有权以确保正确的内存所有权可能是有意义的。

## IO


The \"io\" controller regulates the distribution of IO resources. This controller implements both weight based and absolute bandwidth or IOPS limit distribution; however, weight based distribution is available only if cfq-iosched is in use and neither scheme is available for blk-mq devices.

> “io”控制器调节io资源的分配。该控制器实现了基于权重和绝对带宽或IOPS极限分布；但是，只有在使用cfq-iosched并且两种方案都不适用于blk-mq设备的情况下，基于权重的分发才可用。

### IO Interface Files

io.stat

: A read-only nested-keyed file.


Lines are keyed by \$MAJ:\$MIN device numbers and not ordered. The following nested keys are defined.

> 行由\$MAJ:\$MIN设备编号键控，不排序。定义了以下嵌套键。

      -------- -----------------------
      rbytes   Bytes read
      wbytes   Bytes written
      rios     Number of read IOs
      wios     Number of write IOs
      dbytes   Bytes discarded
      dios     Number of discard IOs
      -------- -----------------------

An example read output follows:

    8:16 rbytes=1459200 wbytes=314773504 rios=192 wios=353 dbytes=0 dios=0
    8:0 rbytes=90430464 wbytes=299008000 rios=8950 wios=1252 dbytes=50331648 dios=3021

io.cost.qos

: A read-write nested-keyed file which exists only on the root cgroup.


This file configures the Quality of Service of the IO cost model based controller (CONFIG_BLK_CGROUP_IOCOST) which currently implements \"io.weight\" proportional control. Lines are keyed by \$MAJ:\$MIN device numbers and not ordered. The line for a given device is populated on the first write for the device on \"io.cost.qos\" or \"io.cost.model\". The following nested keys are defined.

> 该文件配置当前实现“IO.weight”比例控制的基于IO成本模型的控制器（CONFIG_BLK_CGROUP_IOCOST）的服务质量。行由\$MAJ:\$MIN设备编号键控，不排序。给定设备的行是在设备的第一次写入时填充在\“io.cost.qos \”或\“io.cost.model\”上的。定义了以下嵌套键。

      -------- -----------------------------------------
      enable   Weight-based control enable
      ctrl     \"auto\" or \"user\"
      rpct     Read latency percentile \[0, 100\]
      rlat     Read latency threshold
      wpct     Write latency percentile \[0, 100\]
      wlat     Write latency threshold
      min      Minimum scaling percentage \[1, 10000\]
      max      Maximum scaling percentage \[1, 10000\]
      -------- -----------------------------------------


The controller is disabled by default and can be enabled by setting \"enable\" to 1. \"rpct\" and \"wpct\" parameters default to zero and the controller uses internal device saturation state to adjust the overall IO rate between \"min\" and \"max\".

> 默认情况下，控制器处于禁用状态，可以通过将“enable”设置为1来启用控制器\“rpct\”和“wpct\”参数默认为零，控制器使用内部设备饱和状态将总IO速率调整在“最小”和“最大”之间。


When a better control quality is needed, latency QoS parameters can be configured. For example:

> 当需要更好的控制质量时，可以配置延迟QoS参数。例如

    8:16 enable=1 ctrl=auto rpct=95.00 rlat=75000 wpct=95.00 wlat=150000 min=50.00 max=150.0


shows that on sdb, the controller is enabled, will consider the device saturated if the 95th percentile of read completion latencies is above 75ms or write 150ms, and adjust the overall IO issue rate between 50% and 150% accordingly.

> 显示在sdb上，如果读取完成延迟的第95个百分位高于75ms或写入150ms，则控制器将认为设备饱和，并相应地将总IO发布率调整在50%和150%之间。


The lower the saturation point, the better the latency QoS at the cost of aggregate bandwidth. The narrower the allowed adjustment range between \"min\" and \"max\", the more conformant to the cost model the IO behavior. Note that the IO issue base rate may be far off from 100% and setting \"min\" and \"max\" blindly can lead to a significant loss of device capacity or control quality. \"min\" and \"max\" are useful for regulating devices which show wide temporary behavior changes - e.g. a ssd which accepts writes at the line speed for a while and then completely stalls for multiple seconds.

> 饱和点越低，以聚合带宽为代价的延迟QoS越好。“最小值”和“最大值”之间允许的调整范围越窄，IO行为就越符合成本模型。请注意，IO问题基本速率可能远未达到100%，盲目设置“最小”和“最大”可能会导致设备容量或控制质量的显著损失\“min\”和“max\”可用于调节显示大范围临时行为变化的设备，例如ssd，它接受以线速度写入一段时间，然后完全停滞数秒。


When \"ctrl\" is \"auto\", the parameters are controlled by the kernel and may change automatically. Setting \"ctrl\" to \"user\" or setting any of the percentile and latency parameters puts it into \"user\" mode and disables the automatic changes. The automatic mode can be restored by setting \"ctrl\" to \"auto\".

> 当“ctrl”为“auto”时，参数由内核控制，并可能自动更改。将“ctrl\”设置为“user\”或设置任何百分位数和延迟参数将其置于“user”模式并禁用自动更改。可以通过将“ctrl\”设置为“auto\”来恢复自动模式。

io.cost.model

: A read-write nested-keyed file which exists only on the root cgroup.


This file configures the cost model of the IO cost model based controller (CONFIG_BLK_CGROUP_IOCOST) which currently implements \"io.weight\" proportional control. Lines are keyed by \$MAJ:\$MIN device numbers and not ordered. The line for a given device is populated on the first write for the device on \"io.cost.qos\" or \"io.cost.model\". The following nested keys are defined.

> 该文件配置当前实现“IO.weight”比例控制的基于IO成本模型的控制器（CONFIG_BLK_CGROUP_IOCOST）的成本模型。行由\$MAJ:\$MIN设备编号键控，不排序。给定设备的行是在设备的第一次写入时填充在\“io.cost.qos \”或\“io.cost.model\”上的。定义了以下嵌套键。

      ------- ------------------------------------
      ctrl    \"auto\" or \"user\"
      model   The cost model in use - \"linear\"
      ------- ------------------------------------


When \"ctrl\" is \"auto\", the kernel may change all parameters dynamically. When \"ctrl\" is set to \"user\" or any other parameters are written to, \"ctrl\" become \"user\" and the automatic changes are disabled.

> 当“ctrl”为“auto”时，内核可以动态更改所有参数。当“ctrl\”设置为“user\”或写入任何其他参数时，“ctrl\\”将变为“user\\”，自动更改将被禁用。

When \"model\" is \"linear\", the following model parameters are defined.

      ------------------ ------------------------------------------
      \[r\|w\]bps The    maximum sequential IO throughput
      \[r\|w\]seqiops    The maximum 4k sequential IOs per second
      \[r\|w\]randiops   The maximum 4k random IOs per second
      ------------------ ------------------------------------------


From the above, the builtin linear model determines the base costs of a sequential and random IO and the cost coefficient for the IO size. While simple, this model can cover most common device classes acceptably.

> 根据以上内容，内置线性模型确定了顺序和随机IO的基本成本以及IO大小的成本系数。虽然简单，但该模型可以覆盖大多数常见的设备类别。


The IO cost model isn\'t expected to be accurate in absolute sense and is scaled to the device behavior dynamically.

> IO成本模型预计不会在绝对意义上准确，而是根据设备行为动态缩放。


If needed, tools/cgroup/iocost_coef_gen.py can be used to generate device-specific coefficients.

> 如果需要，可以使用工具/cgroup/icost_coff_gen.py生成设备特定系数。

io.weight


: A read-write flat-keyed file which exists on non-root cgroups. The default is \"default 100\".

> ：存在于非根cgroups上的读写平面键控文件。默认值为“默认值100”。


The first line is the default weight applied to devices without specific override. The rest are overrides keyed by \$MAJ:\$MIN device numbers and not ordered. The weights are in the range \[1, 10000\] and specifies the relative amount IO time the cgroup can use in relation to its siblings.

> 第一行是应用于没有特定覆盖的设备的默认权重。其余的是由\$MAJ:\$MIN设备编号键入的覆盖，而不是排序的。权重在\[110000\]范围内，并指定cgroup相对于其同级可以使用的相对IO时间量。


The default weight can be updated by writing either \"default \$WEIGHT\" or simply \"\$WEIGHT\". Overrides can be set by writing \"\$MAJ:\$MIN \$WEIGHT\" and unset by writing \"\$MAJ:\$MIN default\".

> 可以通过写入“default\$weight”或简单地写入“\$weight”来更新默认权重。可通过写入“\$MAJ:\$MIN\$WEIGHT”设置覆盖，并通过写入“\$MAJ:\$MIN default\”取消设置覆盖。

An example read output follows:

    default 100
    8:16 200
    8:0 50

io.max

: A read-write nested-keyed file which exists on non-root cgroups.


BPS and IOPS based IO limit. Lines are keyed by \$MAJ:\$MIN device numbers and not ordered. The following nested keys are defined.

> 基于BPS和IOPS的IO限制。行由\$MAJ:\$MIN设备编号键控，不排序。定义了以下嵌套键。

      ------- ------------------------------------
      rbps    Max read bytes per second
      wbps    Max write bytes per second
      riops   Max read IO operations per second
      wiops   Max write IO operations per second
      ------- ------------------------------------


When writing, any number of nested key-value pairs can be specified in any order. \"max\" can be specified as the value to remove a specific limit. If the same key is specified multiple times, the outcome is undefined.

> 写入时，可以按任何顺序指定任意数量的嵌套键值对\“max\”可以指定为删除特定限制的值。如果多次指定同一个键，则结果未定义。


BPS and IOPS are measured in each IO direction and IOs are delayed if limit is reached. Temporary bursts are allowed.

> 在每个IO方向上测量BPS和IOPS，如果达到限制，IO将延迟。允许临时爆发。

Setting read limit at 2M BPS and write at 120 IOPS for 8:16:

    echo "8:16 rbps=2097152 wiops=120" io.max

Reading returns the following:

    8:16 rbps=2097152 wbps=max riops=max wiops=120

Write IOPS limit can be removed by writing the following:

    echo "8:16 wiops=max" io.max

Reading now returns the following:

    8:16 rbps=2097152 wbps=max riops=max wiops=max

io.pressure

: A read-only nested-keyed file.


Shows pressure stall information for IO. See `Documentation/accounting/psi.rst <psi>`{.interpreted-text role="ref"} for details.

> 显示IO的压力失速信息。有关详细信息，请参阅`Documentation/accounting/pis.rst<psi>`{.depredicted text role=“ref”}。

### Writeback


Page cache is dirtied through buffered writes and shared mmaps and written asynchronously to the backing filesystem by the writeback mechanism. Writeback sits between the memory and IO domains and regulates the proportion of dirty memory by balancing dirtying and write IOs.

> 页面缓存通过缓冲写入和共享mmap被弄脏，并通过写回机制异步写入到备份文件系统。写回位于内存和IO域之间，通过平衡脏IO和写IO来调节脏内存的比例。


The io controller, in conjunction with the memory controller, implements control of page cache writeback IOs. The memory controller defines the memory domain that dirty memory ratio is calculated and maintained for and the io controller defines the io domain which writes out dirty pages for the memory domain. Both system-wide and per-cgroup dirty memory states are examined and the more restrictive of the two is enforced.

> io控制器与内存控制器一起实现对页面缓存写回io的控制。存储器控制器定义计算和维护脏存储器比率的存储器域，并且io控制器定义为存储器域写出脏页的io域。检查系统范围和每个组的脏内存状态，并强制执行这两者中更严格的状态。


cgroup writeback requires explicit support from the underlying filesystem. Currently, cgroup writeback is implemented on ext2, ext4, btrfs, f2fs, and xfs. On other filesystems, all writeback IOs are attributed to the root cgroup.

> cgroup写回需要底层文件系统的明确支持。目前，cgroup写回是在ext2、ext4、btrfs、f2fs和xfs上实现的。在其他文件系统上，所有写回IO都归属于根cgroup。


There are inherent differences in memory and writeback management which affects how cgroup ownership is tracked. Memory is tracked per page while writeback per inode. For the purpose of writeback, an inode is assigned to a cgroup and all IO requests to write dirty pages from the inode are attributed to that cgroup.

> 内存和写回管理存在固有差异，这会影响如何跟踪cgroup所有权。内存是按页面跟踪的，而写回是按索引节点跟踪的。为了进行写回，将一个inode分配给一个cgroup，并且所有从该inode写入脏页的IO请求都归因于该cgroup。


As cgroup ownership for memory is tracked per page, there can be pages which are associated with different cgroups than the one the inode is associated with. These are called foreign pages. The writeback constantly keeps track of foreign pages and, if a particular foreign cgroup becomes the majority over a certain period of time, switches the ownership of the inode to that cgroup.

> 由于内存的cgroup所有权是按页面跟踪的，因此可能有一些页面与不同于inode所关联的cgroups相关联。这些被称为外国页面。写回不断跟踪外来页面，如果某个特定的外来cgroup在某段时间内成为多数，则会将索引节点的所有权切换到该cgroup。


While this model is enough for most use cases where a given inode is mostly dirtied by a single cgroup even when the main writing cgroup changes over time, use cases where multiple cgroups write to a single inode simultaneously are not supported well. In such circumstances, a significant portion of IOs are likely to be attributed incorrectly. As memory controller assigns page ownership on the first use and doesn\'t update it until the page is released, even if writeback strictly follows page ownership, multiple cgroups dirtying overlapping areas wouldn\'t work as expected. It\'s recommended to avoid such usage patterns.

> 虽然这个模型对于大多数用例来说已经足够了，在大多数用例中，给定的inode大多被单个cgroup弄脏，即使主写入cgroup随着时间的推移而变化，但不太支持多个cgroup同时写入单个inode的用例。在这种情况下，很大一部分IO可能被错误地归因。由于内存控制器在第一次使用时分配页面所有权，并且在页面发布之前不会更新它，即使写回严格遵循页面所有权，多个cgroups对重叠区域进行目录化也不会像预期的那样工作。建议避免这种使用模式。


The sysctl knobs which affect writeback behavior are applied to cgroup writeback as follows.

> 影响写回行为的sysctl旋钮应用于cgroup写回，如下所示。

vm.dirty_background_ratio, vm.dirty_ratio


: These ratios apply the same to cgroup writeback with the amount of available memory capped by limits imposed by the memory controller and system-wide clean memory.

> ：这些比率同样适用于cgroup写回，可用内存量受内存控制器和系统范围内干净内存的限制。

vm.dirty_background_bytes, vm.dirty_bytes


: For cgroup writeback, this is calculated into ratio against total available memory and applied the same way as vm.dirty\[\_background\]\_ratio.

> ：对于cgroup写回，它被计算为与总可用内存的比率，并以与vm.sdirty\[\_background\]\_ratio相同的方式应用。

### IO Latency


This is a cgroup v2 controller for IO workload protection. You provide a group with a latency target, and if the average latency exceeds that target the controller will throttle any peers that have a lower latency target than the protected workload.

> 这是一个用于IO工作负载保护的cgroup v2控制器。您为一个组提供了一个延迟目标，如果平均延迟超过该目标，控制器将对延迟目标低于受保护工作负载的任何对等方进行节流。


The limits are only applied at the peer level in the hierarchy. This means that in the diagram below, only groups A, B, and C will influence each other, and groups D and F will influence each other. Group G will influence nobody:

> 这些限制仅应用于层次结构中的对等级别。这意味着在下图中，只有A、B和C组会相互影响，而D和F组会互相影响。G组不会影响任何人：

[root]
/      |        \
A      B        C
/  \        |
D    F       G


So the ideal way to configure this is to set io.latency in groups A, B, and C. Generally you do not want to set a value lower than the latency your device supports. Experiment to find the value that works best for your workload. Start at higher than the expected latency for your device and watch the avg_lat value in io.stat for your workload group to get an idea of the latency you see during normal operation. Use the avg_lat value as a basis for your real setting, setting at 10-15% higher than the value in io.stat.

> 因此，配置此功能的理想方法是在组A、组B和组C中设置io.latency。通常，您不希望设置低于设备支持的延迟的值。尝试找出最适合您工作负载的值。以高于设备预期延迟的速度启动，并观察工作负载组io.stat中的avg_lat值，以了解正常操作期间的延迟。使用avg_lat值作为实际设置的基础，设置为比io.stat中的值高10-15%。

### How IO Latency Throttling Works


io.latency is work conserving; so as long as everybody is meeting their latency target the controller doesn\'t do anything. Once a group starts missing its target it begins throttling any peer group that has a higher target than itself. This throttling takes 2 forms:

> 专利就是节省工作；因此，只要每个人都达到了他们的延迟目标，控制器就不会做任何事情。一旦一个组开始错过其目标，它就会开始限制任何目标高于自身的对等组。这种节流有两种形式：


- Queue depth throttling. This is the number of outstanding IO\'s a group is allowed to have. We will clamp down relatively quickly, starting at no limit and going all the way down to 1 IO at a time.

> -队列深度限制。这是一个组允许拥有的未完成IO的数量。我们将相对快速地取缔，从没有限制的开始，一次减少到1个IO。

- Artificial delay induction. There are certain types of IO that cannot be throttled without possibly adversely affecting higher priority groups. This includes swapping and metadata IO. These types of IO are allowed to occur normally, however they are \"charged\" to the originating group. If the originating group is being throttled you will see the use_delay and delay fields in io.stat increase. The delay value is how many microseconds that are being added to any process that runs in this group. Because this number can grow quite large if there is a lot of swapping or metadata IO occurring we limit the individual delay events to 1 second at a time.

> -人工延迟诱导。某些类型的IO无法在不可能对更高优先级组产生不利影响的情况下进行抑制。这包括交换和元数据IO。这些类型的IO可以正常发生，但它们“收费”给发起组。如果原始组被抑制，您将看到io.stat中的use_delay和delay字段增加。延迟值是添加到此组中运行的任何进程的微秒数。因为如果发生大量交换或元数据IO，这个数字可能会增长得很大，所以我们将单个延迟事件限制为一次1秒。


Once the victimized group starts meeting its latency target again it will start unthrottling any peer groups that were throttled previously. If the victimized group simply stops doing IO the global counter will unthrottle appropriately.

> 一旦受害组再次开始满足其延迟目标，它将开始取消限制以前被限制的任何对等组。如果受害组简单地停止执行IO，则全局计数器将适当地解除限制。

### IO Latency Interface Files

io.latency

: This takes a similar format as the other controllers.

    \"MAJOR:MINOR target=\<target time in microseconds\>\"

io.stat


: If the controller is enabled you will see extra stats in io.stat in addition to the normal ones.

> ：如果启用了控制器，除了正常的统计数据外，您还会在io.stat中看到额外的统计数据。

    depth

    :   This is the current queue depth for the group.

    avg_lat

    :   This is an exponential moving average with a decay rate of 1/exp bound by the sampling interval. The decay rate interval can be calculated by multiplying the win value in io.stat by the corresponding number of samples based on the win value.

    win

    :   The sampling window size in milliseconds. This is the minimum duration of time between evaluation events. Windows only elapse with IO activity. Idle periods extend the most recent window.

### IO Priority


A single attribute controls the behavior of the I/O priority cgroup policy, namely the io.prio.class attribute. The following values are accepted for that attribute:

> 一个属性控制I/O优先级cgroup策略的行为，即io.prio.class属性。该属性接受以下值：

no-change

: Do not modify the I/O priority class.

promote-to-rt


: For requests that have a non-RT I/O priority class, change it into RT. Also change the priority level of these requests to 4. Do not modify the I/O priority of requests that have priority class RT.

> ：对于具有非RT I/O优先级类别的请求，请将其更改为RT。同时将这些请求的优先级级别更改为4。不要修改优先级为RT的请求的I/O优先级。

restrict-to-be


: For requests that do not have an I/O priority class or that have I/O priority class RT, change it into BE. Also change the priority level of these requests to 0. Do not modify the I/O priority class of requests that have priority class IDLE.

> ：对于没有I/O优先级类别或具有I/O优先级类别RT的请求，请将其更改为BE。同时将这些请求的优先级级别更改为0。不要修改优先级为IDLE的请求的I/O优先级。

idle


: Change the I/O priority class of all requests into IDLE, the lowest I/O priority class.

> ：将所有请求的I/O优先级更改为IDLE，即最低的I/O优先级。

none-to-rt

: Deprecated. Just an alias for promote-to-rt.


The following numerical values are associated with the I/O priority policies:

> 以下数值与I/O优先级策略相关联：

---

no-change 0

promote-to-rt 1

restrict-to-be 2

idle 3

---


The numerical value that corresponds to each I/O priority class is as follows:

> 与每个I/O优先级类别相对应的数值如下：

---

IOPRIO_CLASS_NONE 0

IOPRIO_CLASS_RT (real-time) 1

IOPRIO_CLASS_BE (best effort) 2

IOPRIO_CLASS_IDLE 3

---

The algorithm to set the I/O priority class for a request is as follows:


- If I/O priority class policy is promote-to-rt, change the request I/O priority class to IOPRIO_CLASS_RT and change the request I/O priority level to 4.

> -如果I/O优先级类策略提升为rt，请将请求I/O优先级类更改为IOPRIO_class_rt，并将请求I/O优先级别更改为4。

- If I/O priority class policy is not promote-to-rt, translate the I/O priority class policy into a number, then change the request I/O priority class into the maximum of the I/O priority class policy number and the numerical I/O priority class.

> -如果I/O优先级类策略未提升为rt，请将I/O优先级类政策转换为一个数字，然后将请求I/O优先级类更改为I/O优先级类战略数字和数字I/O优先级类的最大值。

## PID


The process number controller is used to allow a cgroup to stop any new tasks from being fork()\'d or clone()\'d after a specified limit is reached.

> 进程号控制器用于允许cgroup在达到指定限制后停止任何新任务的分叉（）或克隆（）。


The number of tasks in a cgroup can be exhausted in ways which other controllers cannot prevent, thus warranting its own controller. For example, a fork bomb is likely to exhaust the number of tasks before hitting memory restrictions.

> 一个cgroup中的任务数量可能会以其他控制器无法阻止的方式耗尽，从而保证其自己的控制器。例如，叉子炸弹可能会在达到内存限制之前耗尽任务数量。


Note that PIDs used in this controller refer to TIDs, process IDs as used by the kernel.

> 请注意，此控制器中使用的PID指的是内核使用的进程ID TID。

### PID Interface Files

pids.max


: A read-write single value file which exists on non-root cgroups. The default is \"max\".

> ：存在于非根cgroups上的读写单值文件。默认值为“最大值”。

Hard limit of number of processes.

pids.current

: A read-only single value file which exists on all cgroups.

The number of processes currently in the cgroup and its descendants.


Organisational operations are not blocked by cgroup policies, so it is possible to have pids.current \pids.max. This can be done by either setting the limit to be smaller than pids.current, or attaching enough processes to the cgroup such that pids.current is larger than pids.max. However, it is not possible to violate a cgroup PID policy through fork() or clone(). These will return -EAGAIN if the creation of a new process would cause a cgroup policy to be violated.

> 组织操作不受cgroup策略的阻止，因此可以使用pids.current\pids.max。这可以通过将限制设置为小于pids.ccurrent来实现，也可以将足够多的进程附加到cgroup以使pids-current大于pids.max。但是，不能通过fork（）或clone（）违反cgroup PID策略。如果创建新进程会导致违反cgroup策略，则这些将返回-EAGAIN。

## Cpuset


The \"cpuset\" controller provides a mechanism for constraining the CPU and memory node placement of tasks to only the resources specified in the cpuset interface files in a task\'s current cgroup. This is especially valuable on large NUMA systems where placing jobs on properly sized subsets of the systems with careful processor and memory placement to reduce cross-node memory access and contention can improve overall system performance.

> “cpuset”控制器提供了一种机制，用于将任务的CPU和内存节点位置限制为仅在任务的当前cgroup中的cpuset接口文件中指定的资源。这在大型NUMA系统上尤其有价值，在这些系统中，将作业放置在适当大小的系统子集上，并仔细放置处理器和内存，以减少跨节点内存访问和争用，可以提高整体系统性能。


The \"cpuset\" controller is hierarchical. That means the controller cannot use CPUs or memory nodes not allowed in its parent.

> “cpuset”控制器是分层的。这意味着控制器不能使用其父级中不允许使用的CPU或内存节点。

### Cpuset Interface Files

cpuset.cpus


: A read-write multiple values file which exists on non-root cpuset-enabled cgroups.

> ：一个读写多值文件，存在于启用了非根cpuset的cgroups上。


It lists the requested CPUs to be used by tasks within this cgroup. The actual list of CPUs to be granted, however, is subjected to constraints imposed by its parent and can differ from the requested CPUs.

> 它列出了该cgroup中的任务要使用的请求CPU。然而，要授予的CPU的实际列表受到其父CPU施加的约束，并且可能与请求的CPU不同。

The CPU numbers are comma-separated numbers or ranges. For example:

    # cat cpuset.cpus
    0-4,6,8-10


An empty value indicates that the cgroup is using the same setting as the nearest cgroup ancestor with a non-empty \"cpuset.cpus\" or all the available CPUs if none is found.

> 空值表示该cgroup使用与最近的cgroup祖先相同的设置，具有非空的“cpuset.cpus\”或所有可用的CPU（如果未找到）。


The value of \"cpuset.cpus\" stays constant until the next update and won\'t be affected by any CPU hotplug events.

> “cpuset.cpus\”的值在下一次更新之前保持不变，并且不会受到任何CPU热插拔事件的影响。

cpuset.cpus.effective


: A read-only multiple values file which exists on all cpuset-enabled cgroups.

> ：只读多值文件，存在于所有启用cpuset的cgroups上。


It lists the onlined CPUs that are actually granted to this cgroup by its parent. These CPUs are allowed to be used by tasks within the current cgroup.

> 它列出了父组实际授予该cgroup的联机CPU。允许当前cgroup中的任务使用这些CPU。


If \"cpuset.cpus\" is empty, the \"cpuset.cpus.effective\" file shows all the CPUs from the parent cgroup that can be available to be used by this cgroup. Otherwise, it should be a subset of \"cpuset.cpus\" unless none of the CPUs listed in \"cpuset.cpus\" can be granted. In this case, it will be treated just like an empty \"cpuset.cpus\".

> 如果“cpuset.cpus\”为空，则“cpuset.cpus.effect”文件将显示父计算机组中可供该计算机组使用的所有CPU。否则，它应该是“cpuset.cpus\”的一个子集，除非“cpuset.cpus\“中列出的CPU都不能被授予。在这种情况下，它将被视为一个空的“cpuset.cpus\”。

Its value will be affected by CPU hotplug events.

cpuset.mems


: A read-write multiple values file which exists on non-root cpuset-enabled cgroups.

> ：一个读写多值文件，存在于启用了非根cpuset的cgroups上。


It lists the requested memory nodes to be used by tasks within this cgroup. The actual list of memory nodes granted, however, is subjected to constraints imposed by its parent and can differ from the requested memory nodes.

> 它列出了此cgroup中的任务要使用的请求内存节点。然而，授予的内存节点的实际列表受到其父节点施加的约束，并且可能与请求的内存节点不同。


The memory node numbers are comma-separated numbers or ranges. For example:

> 内存节点编号是逗号分隔的数字或范围。例如

    # cat cpuset.mems
    0-1,3


An empty value indicates that the cgroup is using the same setting as the nearest cgroup ancestor with a non-empty \"cpuset.mems\" or all the available memory nodes if none is found.

> 空值表示该cgroup使用与最近的cgroup祖先相同的设置，该设置具有非空的“cpuset.mems”或所有可用的内存节点（如果未找到）。


The value of \"cpuset.mems\" stays constant until the next update and won\'t be affected by any memory nodes hotplug events.

> “cpuset.mems”的值在下一次更新之前保持不变，并且不会受到任何内存节点热插拔事件的影响。


Setting a non-empty value to \"cpuset.mems\" causes memory of tasks within the cgroup to be migrated to the designated nodes if they are currently using memory outside of the designated nodes.

> 将非空值设置为“cpuset.mems”会导致cgroup内任务的内存迁移到指定节点，如果它们当前正在使用指定节点之外的内存。


There is a cost for this memory migration. The migration may not be complete and some memory pages may be left behind. So it is recommended that \"cpuset.mems\" should be set properly before spawning new tasks into the cpuset. Even if there is a need to change \"cpuset.mems\" with active tasks, it shouldn\'t be done frequently.

> 这种内存迁移是有代价的。迁移可能未完成，可能会留下一些内存页。因此，建议在向cpuset中生成新任务之前，应正确设置“cpuset.mems”。即使需要用活动任务更改“cpuset.mems”，也不应该频繁进行。

cpuset.mems.effective


: A read-only multiple values file which exists on all cpuset-enabled cgroups.

> ：只读多值文件，存在于所有启用cpuset的cgroups上。


It lists the onlined memory nodes that are actually granted to this cgroup by its parent. These memory nodes are allowed to be used by tasks within the current cgroup.

> 它列出了父组实际授予此cgroup的联机内存节点。允许当前cgroup中的任务使用这些内存节点。


If \"cpuset.mems\" is empty, it shows all the memory nodes from the parent cgroup that will be available to be used by this cgroup. Otherwise, it should be a subset of \"cpuset.mems\" unless none of the memory nodes listed in \"cpuset.mems\" can be granted. In this case, it will be treated just like an empty \"cpuset.mems\".

> 如果“cpuset.mems”为空，它将显示父cgroup中可供该cgroup使用的所有内存节点。否则，它应该是“cpuset.mems”的一个子集，除非“cpuset.mems”中列出的任何内存节点都不能被授予。在这种情况下，它将被视为一个空的“cpuset.mems”。

Its value will be affected by memory nodes hotplug events.

cpuset.cpus.exclusive


: A read-write multiple values file which exists on non-root cpuset-enabled cgroups.

> ：一个读写多值文件，存在于启用了非根cpuset的cgroups上。


It lists all the exclusive CPUs that are allowed to be used to create a new cpuset partition. Its value is not used unless the cgroup becomes a valid partition root. See the \"cpuset.cpus.partition\" section below for a description of what a cpuset partition is.

> 它列出了允许用于创建新cpuset分区的所有独占CPU。除非cgroup成为有效的分区根，否则不会使用其值。有关cpuset分区的描述，请参阅下面的“cpuset.cpu.partition”一节。


When the cgroup becomes a partition root, the actual exclusive CPUs that are allocated to that partition are listed in \"cpuset.cpus.exclusive.effective\" which may be different from \"cpuset.cpus.exclusive\". If \"cpuset.cpus.exclusive\" has previously been set, \"cpuset.cpus.exclusive.effective\" is always a subset of it.

> 当cgroup成为分区根目录时，分配给该分区的实际独占CPU列在“cpuset.CPUs.exclusive.effect\”中，这可能与“cpuset.CPUs.eexclusive\”不同。如果之前设置了“cpuset-CPUs.e排他\”，“cpusett.CPUs.exclusive.eeffective\”始终是它的子集。


Users can manually set it to a value that is different from \"cpuset.cpus\". The only constraint in setting it is that the list of CPUs must be exclusive with respect to its sibling.

> 用户可以手动将其设置为不同于“cpuset.cpus\”的值。设置它的唯一限制是CPU列表必须相对于其同级具有独占性。


For a parent cgroup, any one of its exclusive CPUs can only be distributed to at most one of its child cgroups. Having an exclusive CPU appearing in two or more of its child cgroups is not allowed (the exclusivity rule). A value that violates the exclusivity rule will be rejected with a write error.

> 对于父cgroup，它的任何一个独占CPU最多只能分配给它的一个子cgroup。不允许独占CPU出现在其两个或多个子cgroup中（独占规则）。违反独占性规则的值将被拒绝，并出现写入错误。


The root cgroup is a partition root and all its available CPUs are in its exclusive CPU set.

> 根cgroup是一个分区根，它的所有可用CPU都在它的独占CPU集中。

cpuset.cpus.exclusive.effective


: A read-only multiple values file which exists on all non-root cpuset-enabled cgroups.

> ：只读多值文件，存在于所有启用了cpuset的非根cgroups上。


This file shows the effective set of exclusive CPUs that can be used to create a partition root. The content of this file will always be a subset of \"cpuset.cpus\" and its parent\'s \"cpuset.cpus.exclusive.effective\" if its parent is not the root cgroup. It will also be a subset of \"cpuset.cpus.exclusive\" if it is set. If \"cpuset.cpus.exclusive\" is not set, it is treated to have an implicit value of \"cpuset.cpus\" in the formation of local partition.

> 此文件显示可用于创建分区根的一组有效的独占CPU。如果其父项不是根cgroup，则此文件的内容将始终是\“cpuset.cpus\”及其父项的\“cpuser.cpus.exclusion.effective”的子集。如果设置了它，它也将是“cpuset.cpus.exclusive”的子集。如果未设置“cpuset.cpus.exclusive”，则在形成局部分区时会将其视为具有“cpuset.cpus\”的隐式值。

cpuset.cpus.partition


: A read-write single value file which exists on non-root cpuset-enabled cgroups. This flag is owned by the parent cgroup and is not delegatable.

> ：存在于启用了非根cpuset的cgroups上的读写单值文件。此标志由父cgroup所有，不可删除。

It accepts only the following input values when written to.

      ----------------- ---------------------------------------
      \"member\" Non-   root member of a partition
      \"root\" Part     ition root
      \"isolated\"      Partition root without load balancing
      ----------------- ---------------------------------------


A cpuset partition is a collection of cpuset-enabled cgroups with a partition root at the top of the hierarchy and its descendants except those that are separate partition roots themselves and their descendants. A partition has exclusive access to the set of exclusive CPUs allocated to it. Other cgroups outside of that partition cannot use any CPUs in that set.

> cpuset分区是启用了cpuset的cgroups的集合，其分区根位于层次结构的顶部及其子代，但作为独立分区根本身及其子代的除外。分区对分配给它的独占CPU集具有独占访问权限。该分区之外的其他cgroup不能使用该集中的任何CPU。


There are two types of partitions - local and remote. A local partition is one whose parent cgroup is also a valid partition root. A remote partition is one whose parent cgroup is not a valid partition root itself. Writing to \"cpuset.cpus.exclusive\" is optional for the creation of a local partition as its \"cpuset.cpus.exclusive\" file will assume an implicit value that is the same as \"cpuset.cpus\" if it is not set. Writing the proper \"cpuset.cpus.exclusive\" values down the cgroup hierarchy before the target partition root is mandatory for the creation of a remote partition.

> 有两种类型的分区——本地分区和远程分区。本地分区是其父cgroup也是有效分区根的分区。远程分区的父cgroup本身不是有效的分区根。写入“cpuset.cpus.exclusive”对于创建本地分区是可选的，因为如果未设置其“cpuset.cpus.eexclusive”文件，则会采用与“cpuset.cpus”相同的隐式值。在目标分区根之前，在cgroup层次结构中写入正确的“cpuset.cpus.exclusive”值是创建远程分区所必需的。


Currently, a remote partition cannot be created under a local partition. All the ancestors of a remote partition root except the root cgroup cannot be a partition root.

> 目前，无法在本地分区下创建远程分区。除了根cgroup之外，远程分区根的所有祖先都不能是分区根。


The root cgroup is always a partition root and its state cannot be changed. All other non-root cgroups start out as \"member\".

> 根cgroup始终是分区根，其状态无法更改。所有其他非根cgroup都以“member”开头。


When set to \"root\", the current cgroup is the root of a new partition or scheduling domain. The set of exclusive CPUs is determined by the value of its \"cpuset.cpus.exclusive.effective\".

> 当设置为“根”时，当前cgroup是新分区或计划域的根。独占CPU的集合由其“cpuset.CPUs.exclusive.effect\”的值决定。


When set to \"isolated\", the CPUs in that partition will be in an isolated state without any load balancing from the scheduler. Tasks placed in such a partition with multiple CPUs should be carefully distributed and bound to each of the individual CPUs for optimal performance.

> 当设置为“isolated”时，该分区中的CPU将处于隔离状态，而没有来自调度程序的任何负载平衡。放置在具有多个CPU的分区中的任务应该小心地分配并绑定到每个单独的CPU，以获得最佳性能。


A partition root (\"root\" or \"isolated\") can be in one of the two possible states - valid or invalid. An invalid partition root is in a degraded state where some state information may be retained, but behaves more like a \"member\".

> 分区根目录（\“root\”或\“isolated\”）可以处于两种可能的状态之一——有效或无效。无效的分区根处于降级状态，某些状态信息可能会保留，但其行为更像“成员”。


All possible state transitions among \"member\", \"root\" and \"isolated\" are allowed.

> 允许在“成员”、“根”和“隔离”之间进行所有可能的状态转换。


On read, the \"cpuset.cpus.partition\" file can show the following values.

> 读取时，“cpuset.cpus.dartition”文件可以显示以下值。

      ------------------------------------ ---------------------------------
      \"member\" Non-root mem              ber of a partition
      \"root\" Partition ro                ot
      \"isolated\" Partitio                n root without load balancing
      \"root invalid (\<reason\>)\" Inva   lid partition root
      \"isolated invalid (\<reason\>)\"    Invalid isolated partition root
      ------------------------------------ ---------------------------------


In the case of an invalid partition root, a descriptive string on why the partition is invalid is included within parentheses.

> 在分区根无效的情况下，括号中会包含一个关于分区无效原因的描述性字符串。


For a local partition root to be valid, the following conditions must be met.

> 要使本地分区根目录有效，必须满足以下条件。

1)  The parent cgroup is a valid partition root.

2)  The \"cpuset.cpus.exclusive.effective\" file cannot be empty, though it may contain offline CPUs.

> 2） “cpuset.cpus.exclusive.effect\”文件不能为空，尽管它可能包含脱机CPU。

3)  The \"cpuset.cpus.effective\" cannot be empty unless there is no task associated with this partition.

> 3） “cpuset.cpus.effect”不能为空，除非没有与此分区关联的任务。


For a remote partition root to be valid, all the above conditions except the first one must be met.

> 要使远程分区根有效，必须满足除第一个条件外的所有上述条件。


External events like hotplug or changes to \"cpuset.cpus\" or \"cpuset.cpus.exclusive\" can cause a valid partition root to become invalid and vice versa. Note that a task cannot be moved to a cgroup with empty \"cpuset.cpus.effective\".

> 外部事件，如热插拔或对“cpuset.cpus\”或“cpuseet.cpus.exclusive”的更改，可能会导致有效的分区根无效，反之亦然。请注意，不能将任务移动到“cpuset.cpus.effect\”为空的cgroup中。


A valid non-root parent partition may distribute out all its CPUs to its child local partitions when there is no task associated with it.

> 当没有任务与其关联时，有效的非根父分区可以将其所有CPU分配给其子本地分区。


Care must be taken to change a valid partition root to \"member\" as all its child local partitions, if present, will become invalid causing disruption to tasks running in those child partitions. These inactivated partitions could be recovered if their parent is switched back to a partition root with a proper value in \"cpuset.cpus\" or \"cpuset.cpus.exclusive\".

> 必须小心将有效的分区根目录更改为“member\”，因为它的所有子本地分区（如果存在）都将无效，从而导致在这些子分区中运行的任务中断。如果将这些未激活的分区的父分区切换回“cpuset.cpus\”或“cpuset.cpus.exclusive”中具有正确值的分区根，则可以恢复这些分区。


Poll and inotify events are triggered whenever the state of \"cpuset.cpus.partition\" changes. That includes changes caused by write to \"cpuset.cpus.partition\", cpu hotplug or other changes that modify the validity status of the partition. This will allow user space agents to monitor unexpected changes to \"cpuset.cpus.partition\" without the need to do continuous polling.

> 每当“cpuset.cpus.dartition”的状态发生变化时，就会触发轮询和inotify事件。这包括写入“cpuset.cpus.dartition”、cpu热插拔或其他修改分区有效性状态的更改所引起的更改。这将允许用户空间代理监视对“cpuset.cpu.partition\”的意外更改，而无需进行连续轮询。


A user can pre-configure certain CPUs to an isolated state with load balancing disabled at boot time with the \"isolcpus\" kernel boot command line option. If those CPUs are to be put into a partition, they have to be used in an isolated partition.

> 用户可以使用“isolcpus”内核启动命令行选项将某些CPU预先配置为隔离状态，并在启动时禁用负载平衡。如果要将这些CPU放入一个分区中，就必须在一个独立的分区中使用它们。

## Device controller


Device controller manages access to device files. It includes both creation of new device files (using mknod), and access to the existing device files.

> 设备控制器管理对设备文件的访问。它包括创建新设备文件（使用mknod）和访问现有设备文件。


Cgroup v2 device controller has no interface files and is implemented on top of cgroup BPF. To control access to device files, a user may create bpf programs of type BPF_PROG_TYPE_CGROUP_DEVICE and attach them to cgroups with BPF_CGROUP_DEVICE flag. On an attempt to access a device file, corresponding BPF programs will be executed, and depending on the return value the attempt will succeed or fail with -EPERM.

> Cgroup v2设备控制器没有接口文件，是在Cgroup BPF之上实现的。为了控制对设备文件的访问，用户可以创建bpf_PROG_type_CGROUP_device类型的bpf程序，并将它们附加到带有bpf_CGROUP-device标志的CGROUP。在尝试访问设备文件时，将执行相应的BPF程序，根据返回值，使用-EPERM尝试将成功或失败。


A BPF_PROG_TYPE_CGROUP_DEVICE program takes a pointer to the bpf_cgroup_dev_ctx structure, which describes the device access attempt: access type (mknod/read/write) and device (type, major and minor numbers). If the program returns 0, the attempt fails with -EPERM, otherwise it succeeds.

> BPF_PROG_TYPE_CGROUP_DEVICE程序将指针指向BPF_CGROUP_dev_ctx结构，该结构描述设备访问尝试：访问类型（mknod/read/write）和设备（类型、主要和次要编号）。如果程序返回0，则使用-EPERM尝试失败，否则尝试成功。


An example of BPF_PROG_TYPE_CGROUP_DEVICE program may be found in tools/testing/selftests/bpf/progs/dev_cgroup.c in the kernel source tree.

> BPF_PROG_TYPE_CGROUP_DEVICE程序的示例可以在内核源代码树的tools/testing/selftests/BPF/progs/dev_CGROUP.c中找到。

## RDMA


The \"rdma\" controller regulates the distribution and accounting of RDMA resources.

> “rdma”控制器管理rdma资源的分配和核算。

### RDMA Interface Files

rdma.max


: A readwrite nested-keyed file that exists for all the cgroups except root that describes current configured resource limit for a RDMA/IB device.

> ：一个读写嵌套键控文件，存在于除根以外的所有cgroup，用于描述RDMA/IB设备的当前配置资源限制。


Lines are keyed by device name and are not ordered. Each line contains space separated resource name and its configured limit that can be distributed.

> 行按设备名称键入，不按顺序排列。每一行都包含以空格分隔的资源名称及其可分配的配置限制。

The following nested keys are defined.

      ------------ -------------------------------
      hca_handle   Maximum number of HCA Handles
      hca_object   Maximum number of HCA Objects
      ------------ -------------------------------

An example for mlx4 and ocrdma device follows:

    mlx4_0 hca_handle=2 hca_object=2000
    ocrdma1 hca_handle=3 hca_object=max

rdma.current


: A read-only file that describes current resource usage. It exists for all the cgroup except root.

> ：描述当前资源使用情况的只读文件。它存在于除根以外的所有cgroup。

An example for mlx4 and ocrdma device follows:

    mlx4_0 hca_handle=1 hca_object=20
    ocrdma1 hca_handle=1 hca_object=23

## HugeTLB


The HugeTLB controller allows to limit the HugeTLB usage per control group and enforces the controller limit during page fault.

> HugeTLB控制器允许限制每个控制组的HugeTLB使用，并在页面故障期间强制执行控制器限制。

### HugeTLB Interface Files

hugetlb.\<hugepagesize\>.current


: Show current usage for \"hugepagesize\" hugetlb. It exists for all the cgroup except root.

> ：显示\“hugepagesize”hugetlb的当前用法。它存在于除根以外的所有cgroup。

hugetlb.\<hugepagesize\>.max


: Set/show the hard limit of \"hugepagesize\" hugetlb usage. The default value is \"max\". It exists for all the cgroup except root.

> ：设置/显示“hugepadgesize”hugetlb用法的硬限制。默认值为“最大值”。它存在于除根以外的所有cgroup。

hugetlb.\<hugepagesize\>.events

: A read-only flat-keyed file which exists on non-root cgroups.

    max

    :   The number of allocation failure due to HugeTLB limit

hugetlb.\<hugepagesize\>.events.local


: Similar to hugetlb.\<hugepagesize\>.events but the fields in the file are local to the cgroup i.e. not hierarchical. The file modified event generated on this file reflects only the local events.

> ：类似于hugetlb\<hugepageize\>.events，但文件中的字段是cgroup的本地字段，即不具有层次结构。在此文件上生成的文件修改事件仅反映本地事件。

hugetlb.\<hugepagesize\>.numa_stat

:

Similar to memory.numa_stat, it shows the numa information of the


:   hugetlb pages of \<hugepagesize\in this cgroup. Only active in use hugetlb pages are included. The per-node values are in bytes.

> ：该组中的\<hugepadgesize\的hugetlb页。只包括使用中的活动hugetlb页面。每个节点的值以字节为单位。

## Misc


The Miscellaneous cgroup provides the resource limiting and tracking mechanism for the scalar resources which cannot be abstracted like the other cgroup resources. Controller is enabled by the CONFIG_CGROUP_MISC config option.

> Miscellaneous cgroup为标量资源提供了资源限制和跟踪机制，标量资源不能像其他cgroup资源那样抽象。控制器由CONFIG_CGROUP_MISC配置选项启用。


A resource can be added to the controller via enum misc_res_type{} in the include/linux/misc_cgroup.h file and the corresponding name via misc_res_name\[\] in the kernel/cgroup/misc.c file. Provider of the resource must set its capacity prior to using the resource by calling misc_cg_set_capacity().

> 可以通过include/linux/misc_group.h文件中的enum misc_restype｛｝将资源添加到控制器，并通过kernel/cgroup/misc.c文件中的misc_restname\[\]将相应的名称添加到控制器。在使用资源之前，资源的提供程序必须通过调用misc_cg_set_capacity（）来设置其容量。


Once a capacity is set then the resource usage can be updated using charge and uncharge APIs. All of the APIs to interact with misc controller are in include/linux/misc_cgroup.h.

> 一旦设置了容量，就可以使用收费和不收费API更新资源使用情况。所有与misc控制器交互的API都在include/linux/misc_group.h中。

### Misc Interface Files


Miscellaneous controller provides 3 interface files. If two misc resources (res_a and res_b) are registered then:

> 杂项控制器提供3个接口文件。如果注册了两个其他资源（res_a和res_b），则：

misc.capacity


: A read-only flat-keyed file shown only in the root cgroup. It shows miscellaneous scalar resources available on the platform along with their quantities:

> ：只读平面键控文件，仅显示在根cgroup中。它显示了平台上可用的各种标量资源及其数量：

    $ cat misc.capacity

res_a 50 res_b 10

misc.current


: A read-only flat-keyed file shown in the all cgroups. It shows the current usage of the resources in the cgroup and its children.:

> ：显示在所有cgroup中的只读平面键控文件。它显示了cgroup及其子组中资源的当前使用情况

    $ cat misc.current

res_a 3 res_b 0

misc.max


: A read-write flat-keyed file shown in the non root cgroups. Allowed maximum usage of the resources in the cgroup and its children.:

> ：非根cgroups中显示的读写平面键控文件。允许最大限度地使用cgroup及其子组中的资源。：

    $ cat misc.max

res_a max res_b 4

Limit can be set by:

    # echo res_a 1 misc.max

Limit can be set to max by:

    # echo res_a max misc.max


Limits can be set higher than the capacity value in the misc.capacity file.

> 可以将限制设置为高于misc.capacity文件中的容量值。

misc.events


: A read-only flat-keyed file which exists on non-root cgroups. The following entries are defined. Unless specified otherwise, a value change in this file generates a file modified event. All fields in this file are hierarchical.

> ：存在于非根cgroups上的只读平面键控文件。定义了以下条目。除非另有规定，否则此文件中的值更改将生成文件修改事件。此文件中的所有字段都是分层的。

    max

    :   The number of times the cgroup\'s resource usage was about to go over the max boundary.

### Migration and Ownership


A miscellaneous scalar resource is charged to the cgroup in which it is used first, and stays charged to that cgroup until that resource is freed. Migrating a process to a different cgroup does not move the charge to the destination cgroup where the process has moved.

> 杂项标量资源被计入首先使用它的cgroup，并一直被计入该cgroup直到该资源被释放。将流程迁移到不同的cgroup不会将费用移动到流程移动的目标cgroup。

## Others

### perf_event


perf_event controller, if not mounted on a legacy hierarchy, is automatically enabled on the v2 hierarchy so that perf events can always be filtered by cgroup v2 path. The controller can still be moved to a legacy hierarchy after v2 hierarchy is populated.

> 如果perf_event控制器未安装在遗留层次结构上，则会在v2层次结构上自动启用，因此perf事件始终可以通过cgroup v2路径进行筛选。在填充v2层次结构之后，控制器仍然可以移动到遗留层次结构。

## Non-normative information


This section contains information that isn\'t considered to be a part of the stable kernel API and so is subject to change.

> 本节包含的信息不被视为稳定内核API的一部分，因此可能会发生更改。

### CPU controller root cgroup process behaviour


When distributing CPU cycles in the root cgroup each thread in this cgroup is treated as if it was hosted in a separate child cgroup of the root cgroup. This child cgroup weight is dependent on its thread nice level.

> 当在根cgroup中分配CPU周期时，该cgroup的每个线程都被视为托管在根cggroup的一个单独的子cggroup中。这个子cgroup的权重取决于它的线程漂亮程度。


For details of this mapping see sched_prio_to_weight array in kernel/sched/core.c file (values from this array should be scaled appropriately so the neutral - nice 0 - value is 100 instead of 1024).

> 有关此映射的详细信息，请参阅kernel/sched/core.c文件中的sched_prio_to_weight数组（此数组中的值应适当缩放，使中性值nice 0为100而不是1024）。

### IO controller root cgroup process behaviour


Root cgroup processes are hosted in an implicit leaf child node. When distributing IO resources this implicit child node is taken into account as if it was a normal child cgroup of the root cgroup with a weight value of 200.

> 根cgroup进程托管在隐式叶子子节点中。当分配IO资源时，将考虑该隐式子节点，就好像它是权重值为200的根cgroup的正常子cgroup一样。

# Namespace

## Basics


cgroup namespace provides a mechanism to virtualize the view of the \"/proc/\$PID/cgroup\" file and cgroup mounts. The CLONE_NEWCGROUP clone flag can be used with clone(2) and unshare(2) to create a new cgroup namespace. The process running inside the cgroup namespace will have its \"/proc/\$PID/cgroup\" output restricted to cgroupns root. The cgroupns root is the cgroup of the process at the time of creation of the cgroup namespace.

> cgroup命名空间提供了一种机制来虚拟化\“/proc/\$PID/cgroup\”文件和cgroup装载的视图。CLONE_NEWCGROUP克隆标志可以与CLONE（2）和unshare（2）一起使用，以创建新的cgroup命名空间。在cgroup命名空间内运行的进程将其\“/proc/\$PID/cgroup\”输出限制为cgroupns根。cgroupns根是创建cgroup命名空间时进程的cgroup。


Without cgroup namespace, the \"/proc/\$PID/cgroup\" file shows the complete path of the cgroup of a process. In a container setup where a set of cgroups and namespaces are intended to isolate processes the \"/proc/\$PID/cgroup\" file may leak potential system level information to the isolated processes. For example:

> 如果没有cgroup命名空间，“/proc/\$PID/cgroup\”文件将显示进程的cgroup的完整路径。在容器设置中，一组cgroup和命名空间用于隔离进程，“/proc/\$PID/cgroup\”文件可能会将潜在的系统级信息泄露给隔离的进程。例如

# cat /proc/self/cgroup
0::/batchjobs/container_id1


The path \'/batchjobs/container_id1\' can be considered as system-data and undesirable to expose to the isolated processes. cgroup namespace can be used to restrict visibility of this path. For example, before creating a cgroup namespace, one would see:

> 路径“/batchjobs/container_id1\”可以被视为系统数据，不希望暴露给隔离的进程。cgroup命名空间可用于限制此路径的可见性。例如，在创建cgroup名称空间之前，可以看到：

# ls -l /proc/self/ns/cgroup

lrwxrwxrwx 1 root root 0 2014-07-15 10:37 /proc/self/ns/cgroup -cgroup:[4026531835]

> lrwxrwxrwx 1 root 0 2014-07-15 10:37/proc/self/ns/cgroup-cgroup：[4026531835]
# cat /proc/self/cgroup
0::/batchjobs/container_id1

After unsharing a new namespace, the view changes:

# ls -l /proc/self/ns/cgroup

lrwxrwxrwx 1 root root 0 2014-07-15 10:35 /proc/self/ns/cgroup -cgroup:[4026532183]

> lrwxrwxrwx 1根根0 2014-07-15 10:35/proc/self/ns/cgroup-cgroup：[4026532183]
# cat /proc/self/cgroup
0::/


When some thread from a multi-threaded process unshares its cgroup namespace, the new cgroupns gets applied to the entire process (all the threads). This is natural for the v2 hierarchy; however, for the legacy hierarchies, this may be unexpected.

> 当多线程进程中的某个线程取消共享其cgroup命名空间时，新的cgroupn将应用于整个进程（所有线程）。这对于v2层次结构来说是很自然的；然而，对于遗留层次结构来说，这可能是出乎意料的。


A cgroup namespace is alive as long as there are processes inside or mounts pinning it. When the last usage goes away, the cgroup namespace is destroyed. The cgroupns root and the actual cgroups remain.

> 只要cgroup命名空间中有进程或挂载固定它，它就处于活动状态。当最后一次使用消失时，cgroup名称空间就会被销毁。保留cgroupns根和实际的cgroups。

## The Root and Views


The \'cgroupns root\' for a cgroup namespace is the cgroup in which the process calling unshare(2) is running. For example, if a process in /batchjobs/container_id1 cgroup calls unshare, cgroup /batchjobs/container_id1 becomes the cgroupns root. For the init_cgroup_ns, this is the real root (\'/\') cgroup.

> cgroup命名空间的“groupns根”是运行调用unshare（2）的进程的cgroup。例如，如果/batchjobs/container_id1 cgroup中的进程调用unshare，则cgroup/batchjobs/container_id1将成为cgroupns的根。对于init_cgroup_ns，这是真正的根（\'/\'）cgroup。


The cgroupns root cgroup does not change even if the namespace creator process later moves to a different cgroup:

> cgroupns根cgroup不会更改，即使命名空间创建者进程稍后移动到不同的cgroup：

# ~/unshare -c # unshare cgroupns in some cgroup
# cat /proc/self/cgroup
0::/
# mkdir sub_cgrp_1
# echo 0 sub_cgrp_1/cgroup.procs
# cat /proc/self/cgroup
0::/sub_cgrp_1

Each process gets its namespace-specific view of \"/proc/\$PID/cgroup\"


Processes running inside the cgroup namespace will be able to see cgroup paths (in /proc/self/cgroup) only inside their root cgroup. From within an unshared cgroupns:

> 在cgroup命名空间内运行的进程只能在其根cgroup内看到cgroup路径（在/proc/self/cgroup中）。从非共享的cgroupns中：

# sleep 100000 &
[1] 7353
# echo 7353 sub_cgrp_1/cgroup.procs
# cat /proc/7353/cgroup
0::/sub_cgrp_1

From the initial cgroup namespace, the real cgroup path will be visible:

$ cat /proc/7353/cgroup
0::/batchjobs/container_id1/sub_cgrp_1


From a sibling cgroup namespace (that is, a namespace rooted at a different cgroup), the cgroup path relative to its own cgroup namespace root will be shown. For instance, if PID 7353\'s cgroup namespace root is at \'/batchjobs/container_id2\', then it will see:

> 从同级cgroup命名空间（即根位于不同cgroup的命名空间），将显示相对于其自身cgroup名称空间根的cgroup路径。例如，如果PID 7353\的cgroup命名空间根位于\'/batchjobs/container_id2\'，则它将看到：

# cat /proc/7353/cgroup
0::/../container_id2/sub_cgrp_1


Note that the relative path always starts with \'/\' to indicate that its relative to the cgroup namespace root of the caller.

> 请注意，相对路径总是以\'/\'开头，表示其相对于调用方的cgroup命名空间根的相对路径。

## Migration and setns(2)


Processes inside a cgroup namespace can move into and out of the namespace root if they have proper access to external cgroups. For example, from inside a namespace with cgroupns root at /batchjobs/container_id1, and assuming that the global hierarchy is still accessible inside cgroupns:

> 如果cgroup命名空间内的进程能够正确访问外部cgroup，则它们可以移入和移出命名空间根。例如，从一个cgroupns根位于/batchjobs/container_id1的命名空间内部，并假设全局层次结构在cgroupns内部仍然可以访问：

# cat /proc/7353/cgroup
0::/sub_cgrp_1
# echo 7353 batchjobs/container_id2/cgroup.procs
# cat /proc/7353/cgroup
0::/../container_id2


Note that this kind of setup is not encouraged. A task inside cgroup namespace should only be exposed to its own cgroupns hierarchy.

> 请注意，不鼓励进行这种设置。cgroup命名空间内的任务应仅向其自己的cgroupns层次结构公开。

setns(2) to another cgroup namespace is allowed when:

(a) the process has CAP_SYS_ADMIN against its current user namespace

(b) the process has CAP_SYS_ADMIN against the target cgroup namespace\'s userns

> （b） 进程具有针对目标cgroup命名空间的用户的CAP_SYS_ADMIN


No implicit cgroup changes happen with attaching to another cgroup namespace. It is expected that the someone moves the attaching process under the target cgroup namespace root.

> 附加到另一个cgroup命名空间时不会发生任何隐式的cgroup更改。预计someone会将附加进程移动到目标cgroup命名空间根目录下。

## Interaction with Other Namespaces


Namespace specific cgroup hierarchy can be mounted by a process running inside a non-init cgroup namespace:

> 命名空间特定的cgroup层次结构可以由在非init cgroup命名空间内运行的进程挂载：

# mount -t cgroup2 none $MOUNT_POINT


This will mount the unified cgroup hierarchy with cgroupns root as the filesystem root. The process needs CAP_SYS_ADMIN against its user and mount namespaces.

> 这将挂载统一的cgroup层次结构，将cgroupns根作为文件系统根。进程需要针对其用户和装载命名空间的CAP_SYS_ADMIN。


The virtualization of /proc/self/cgroup file combined with restricting the view of cgroup hierarchy by namespace-private cgroupfs mount provides a properly isolated cgroup view inside the container.

> /proc/self/cgroup文件的虚拟化，加上通过命名空间私有cgroupfs装载限制cgroup层次结构的视图，在容器内提供了一个适当隔离的cgroup视图。

# Information on Kernel Programming


This section contains kernel programming information in the areas where interacting with cgroup is necessary. cgroup core and controllers are not covered.

> 本节包含需要与cgroup交互的领域中的内核编程信息。不包括cgroup核心和控制器。

## Filesystem Support for Writeback


A filesystem can support cgroup writeback by updating address_space_operations-\>writepage\[s\]() to annotate bio\'s using the following two functions.

> 文件系统可以通过使用以下两个函数更新address_space_operations-\>writepage\[s\]（）来注释bio来支持cgroup写回。

wbc_init_bio(@wbc, \@bio)


: Should be called for each bio carrying writeback data and associates the bio with the inode\'s owner cgroup and the corresponding request queue. This must be called after a queue (device) has been associated with the bio and before submission.

> ：应该为每个携带写回数据的bio调用，并将bio与inode的所有者cgroup和相应的请求队列相关联。这必须在队列（设备）与生物关联之后和提交之前调用。

wbc_account_cgroup_owner(@wbc, \@page, \@bytes)


: Should be called for each data segment being written out. While this function doesn\'t care exactly when it\'s called during the writeback session, it\'s the easiest and most natural to call it as data segments are added to a bio.

> ：应该为正在写入的每个数据段调用。虽然这个函数并不关心在写回会话期间何时调用它，但当数据段被添加到bio时，调用它是最简单、最自然的。


With writeback bio\'s annotated, cgroup support can be enabled per super_block by setting SB_I_CGROUPWB in -\>s_iflags. This allows for selective disabling of cgroup writeback support which is helpful when certain filesystem features, e.g. journaled data mode, are incompatible.

> 有了写回bio的注释，可以通过在-\>s_iflags中设置SB_I_CGROUPBB来为每个super_block启用cgroup支持。这允许选择性地禁用cgroup写回支持，这在某些文件系统功能（例如日志数据模式）不兼容时很有帮助。


wbc_init_bio() binds the specified bio to its cgroup. Depending on the configuration, the bio may be executed at a lower priority and if the writeback session is holding shared resources, e.g. a journal entry, may lead to priority inversion. There is no one easy solution for the problem. Filesystems can try to work around specific problem cases by skipping wbc_init_bio() and using bio_associate_blkg() directly.

> wbc_init_bio（）将指定的bio绑定到其cgroup。根据配置，生物可以以较低的优先级执行，并且如果写回会话持有共享资源，例如日志条目，则可能导致优先级反转。这个问题没有一个简单的解决办法。文件系统可以跳过wbc_init_bio（）并直接使用bio_associate_blk（）来解决特定的问题。

# Deprecated v1 Core Features

- Multiple hierarchies including named ones are not supported.
- All v1 mount options are not supported.
- The \"tasks\" file is removed and \"cgroup.procs\" is not sorted.
- \"cgroup.clone_children\" is removed.
- /proc/cgroups is meaningless for v2. Use \"cgroup.controllers\" file at the root instead.

# Issues with v1 and Rationales for v2

## Multiple Hierarchies


cgroup v1 allowed an arbitrary number of hierarchies and each hierarchy could host any number of controllers. While this seemed to provide a high level of flexibility, it wasn\'t useful in practice.

> cgroupv1允许任意数量的层次结构，并且每个层次结构可以承载任意数量的控制器。虽然这似乎提供了很高的灵活性，但在实践中却毫无用处。


For example, as there is only one instance of each controller, utility type controllers such as freezer which can be useful in all hierarchies could only be used in one. The issue is exacerbated by the fact that controllers couldn\'t be moved to another hierarchy once hierarchies were populated. Another issue was that all controllers bound to a hierarchy were forced to have exactly the same view of the hierarchy. It wasn\'t possible to vary the granularity depending on the specific controller.

> 例如，由于每个控制器只有一个实例，因此在所有层次结构中都有用的实用类型控制器（如冷冻机）只能在一个层次结构中使用。一旦填充了层次结构，控制器就无法移动到另一个层次结构，这一事实加剧了这个问题。另一个问题是，绑定到层次结构的所有控制器都被迫具有完全相同的层次结构视图。不可能根据特定的控制器来改变粒度。


In practice, these issues heavily limited which controllers could be put on the same hierarchy and most configurations resorted to putting each controller on its own hierarchy. Only closely related ones, such as the cpu and cpuacct controllers, made sense to be put on the same hierarchy. This often meant that userland ended up managing multiple similar hierarchies repeating the same steps on each hierarchy whenever a hierarchy management operation was necessary.

> 在实践中，这些问题严重限制了哪些控制器可以放在同一层次结构上，大多数配置都将每个控制器放在自己的层次结构上。只有密切相关的控制器，如cpu和cpuacct控制器，才有意义放在同一层次结构上。这通常意味着，每当需要进行层次管理操作时，userland最终会管理多个类似的层次，在每个层次上重复相同的步骤。


Furthermore, support for multiple hierarchies came at a steep cost. It greatly complicated cgroup core implementation but more importantly the support for multiple hierarchies restricted how cgroup could be used in general and what controllers was able to do.

> 此外，对多层次结构的支持也付出了高昂的代价。它使cgroup核心实现非常复杂，但更重要的是，对多个层次结构的支持限制了cgroup的一般使用方式以及控制器的功能。


There was no limit on how many hierarchies there might be, which meant that a thread\'s cgroup membership couldn\'t be described in finite length. The key might contain any number of entries and was unlimited in length, which made it highly awkward to manipulate and led to addition of controllers which existed only to identify membership, which in turn exacerbated the original problem of proliferating number of hierarchies.

> 层次结构的数量没有限制，这意味着线程的cgroup成员身份不能用有限的长度来描述。密钥可能包含任何数量的条目，并且长度不受限制，这使得操作起来非常困难，并导致添加了仅用于识别成员身份的控制器，这反过来又加剧了层次结构数量激增的原始问题。


Also, as a controller couldn\'t have any expectation regarding the topologies of hierarchies other controllers might be on, each controller had to assume that all other controllers were attached to completely orthogonal hierarchies. This made it impossible, or at least very cumbersome, for controllers to cooperate with each other.

> 此外，由于控制器对其他控制器可能所在的层次结构的拓扑结构没有任何期望，每个控制器都必须假设所有其他控制器都连接到完全正交的层次结构。这使得控制器之间无法相互协作，或者至少非常麻烦。


In most use cases, putting controllers on hierarchies which are completely orthogonal to each other isn\'t necessary. What usually is called for is the ability to have differing levels of granularity depending on the specific controller. In other words, hierarchy may be collapsed from leaf towards root when viewed from specific controllers. For example, a given configuration might not care about how memory is distributed beyond a certain level while still wanting to control how CPU cycles are distributed.

> 在大多数用例中，将控制器放在彼此完全正交的层次结构上是不必要的。通常需要的是根据特定控制器具有不同粒度级别的能力。换句话说，当从特定控制器观看时，层次结构可以从叶向根折叠。例如，给定的配置可能不关心内存在某个级别之外的分布方式，同时仍然希望控制CPU周期的分布方式。

## Thread Granularity


cgroup v1 allowed threads of a process to belong to different cgroups. This didn\'t make sense for some controllers and those controllers ended up implementing different ways to ignore such situations but much more importantly it blurred the line between API exposed to individual applications and system management interface.

> cgroupv1允许进程的线程属于不同的cgroup。这对一些控制器来说没有意义，这些控制器最终实现了不同的方法来忽略这些情况，但更重要的是，它模糊了暴露于单个应用程序的API和系统管理界面之间的界限。


Generally, in-process knowledge is available only to the process itself; thus, unlike service-level organization of processes, categorizing threads of a process requires active participation from the application which owns the target process.

> 一般来说，过程中的知识只能用于过程本身；因此，与进程的服务级别组织不同，对进程的线程进行分类需要拥有目标进程的应用程序的积极参与。


cgroup v1 had an ambiguously defined delegation model which got abused in combination with thread granularity. cgroups were delegated to individual applications so that they can create and manage their own sub-hierarchies and control resource distributions along them. This effectively raised cgroup to the status of a syscall-like API exposed to lay programs.

> cgroupv1有一个定义模糊的委托模型，结合线程粒度被滥用。cgroups被委托给各个应用程序，这样它们就可以创建和管理自己的子层次结构，并控制资源分布。这有效地将cgroup提升到了向lay程序公开的类似系统调用的API的状态。


First of all, cgroup has a fundamentally inadequate interface to be exposed this way. For a process to access its own knobs, it has to extract the path on the target hierarchy from /proc/self/cgroup, construct the path by appending the name of the knob to the path, open and then read and/or write to it. This is not only extremely clunky and unusual but also inherently racy. There is no conventional way to define transaction across the required steps and nothing can guarantee that the process would actually be operating on its own sub-hierarchy.

> 首先，cgroup有一个根本不适合以这种方式公开的接口。为了让进程访问自己的knob，它必须从/proc/self/cgroup中提取目标层次结构上的路径，通过将knob的名称附加到路径上来构建路径，打开然后对其进行读取和/或写入。这不仅非常笨拙和不寻常，而且本质上也很活泼。没有传统的方法来定义跨所需步骤的事务，也没有什么可以保证流程实际上在其自己的子层次结构上运行。


cgroup controllers implemented a number of knobs which would never be accepted as public APIs because they were just adding control knobs to system-management pseudo filesystem. cgroup ended up with interface knobs which were not properly abstracted or refined and directly revealed kernel internal details. These knobs got exposed to individual applications through the ill-defined delegation mechanism effectively abusing cgroup as a shortcut to implementing public APIs without going through the required scrutiny.

> cgroup控制器实现了许多旋钮，这些旋钮永远不会被接受为公共API，因为它们只是将控制旋钮添加到系统管理伪文件系统中。cgroup最终出现了接口旋钮，这些旋钮没有经过适当的抽象或精炼，直接揭示了内核的内部细节。这些旋钮通过定义不清的委托机制暴露在单个应用程序中，有效地滥用了cgroup作为实现公共API的快捷方式，而没有经过必要的审查。


This was painful for both userland and kernel. Userland ended up with misbehaving and poorly abstracted interfaces and kernel exposing and locked into constructs inadvertently.

> 这对userland和内核来说都是痛苦的。Userland最终出现了行为不端、抽象性差的接口，以及无意中暴露和锁定在结构中的内核。

## Competition Between Inner Nodes and Threads


cgroup v1 allowed threads to be in any cgroups which created an interesting problem where threads belonging to a parent cgroup and its children cgroups competed for resources. This was nasty as two different types of entities competed and there was no obvious way to settle it. Different controllers did different things.

> cgroupv1允许线程在任何cgroup中，这就产生了一个有趣的问题，即属于父cgroup及其子cgroup的线程争夺资源。这很糟糕，因为两种不同类型的实体在竞争，而且没有明显的解决方法。不同的控制者做了不同的事情。


The cpu controller considered threads and cgroups as equivalents and mapped nice levels to cgroup weights. This worked for some cases but fell flat when children wanted to be allocated specific ratios of CPU cycles and the number of internal threads fluctuated - the ratios constantly changed as the number of competing entities fluctuated. There also were other issues. The mapping from nice level to weight wasn\'t obvious or universal, and there were various other knobs which simply weren\'t available for threads.

> cpu控制器将线程和cgroup视为等价物，并将良好的级别映射到cgroup权重。这在某些情况下有效，但当孩子们希望被分配特定的CPU周期比率，并且内部线程的数量波动时，这种情况就没有了——随着竞争实体的数量波动，这种比率不断变化。还有其他问题。从好的级别到权重的映射并不明显或普遍，还有各种其他的旋钮根本不适用于线程。


The io controller implicitly created a hidden leaf node for each cgroup to host the threads. The hidden leaf had its own copies of all the knobs with `leaf_` prefixed. While this allowed equivalent control over internal threads, it was with serious drawbacks. It always added an extra layer of nesting which wouldn\'t be necessary otherwise, made the interface messy and significantly complicated the implementation.

> io控制器隐式地为每个cgroup创建了一个隐藏的叶节点来承载线程。隐藏的叶子有自己的所有旋钮的副本，前缀是“叶子_”。虽然这允许对内部线程进行同等的控制，但也有严重的缺点。它总是添加一层额外的嵌套，否则这是不必要的，这会使接口变得混乱，并使实现变得非常复杂。


The memory controller didn\'t have a way to control what happened between internal tasks and child cgroups and the behavior was not clearly defined. There were attempts to add ad-hoc behaviors and knobs to tailor the behavior to specific workloads which would have led to problems extremely difficult to resolve in the long term.

> 内存控制器无法控制内部任务和子cgroup之间发生的事情，并且行为没有明确定义。有人试图添加特别行为和旋钮，以根据特定的工作负载调整行为，这将导致长期难以解决的问题。


Multiple controllers struggled with internal tasks and came up with different ways to deal with it; unfortunately, all the approaches were severely flawed and, furthermore, the widely different behaviors made cgroup as a whole highly inconsistent.

> 多个控制器在内部任务中挣扎，并想出了不同的方法来处理它；不幸的是，所有的方法都存在严重的缺陷，而且，广泛不同的行为使cgroup作为一个整体高度不一致。


This clearly is a problem which needs to be addressed from cgroup core in a uniform way.

> 这显然是一个需要从cgroup核心以统一的方式解决的问题。

## Other Interface Issues


cgroup v1 grew without oversight and developed a large number of idiosyncrasies and inconsistencies. One issue on the cgroup core side was how an empty cgroup was notified - a userland helper binary was forked and executed for each event. The event delivery wasn\'t recursive or delegatable. The limitations of the mechanism also led to in-kernel event delivery filtering mechanism further complicating the interface.

> cgroupv1在没有监督的情况下成长，并发展出大量的特质和不一致性。cgroup核心方面的一个问题是如何通知空的cgroup——为每个事件派生并执行一个userland辅助二进制文件。事件传递不是递归的或不可删除的。该机制的局限性还导致内核内事件传递过滤机制进一步使接口复杂化。


Controller interfaces were problematic too. An extreme example is controllers completely ignoring hierarchical organization and treating all cgroups as if they were all located directly under the root cgroup. Some controllers exposed a large amount of inconsistent implementation details to userland.

> 控制器接口也有问题。一个极端的例子是，控制器完全忽略层次组织，并将所有cgroup视为它们都直接位于根cgroup之下。一些控制器向userland暴露了大量不一致的实现细节。


There also was no consistency across controllers. When a new cgroup was created, some controllers defaulted to not imposing extra restrictions while others disallowed any resource usage until explicitly configured. Configuration knobs for the same type of control used widely differing naming schemes and formats. Statistics and information knobs were named arbitrarily and used different formats and units even in the same controller.

> 控制器之间也没有一致性。当创建新的cgroup时，一些控制器默认不施加额外的限制，而其他控制器则在明确配置之前不允许使用任何资源。同一类型控件的配置旋钮使用了广泛不同的命名方案和格式。统计和信息旋钮是任意命名的，即使在同一控制器中也使用不同的格式和单位。


cgroup v2 establishes common conventions where appropriate and updates controllers so that they expose minimal and consistent interfaces.

> cgroupv2在适当的情况下建立通用约定，并更新控制器，以便它们公开最小且一致的接口。

## Controller Issues and Remedies

### Memory


The original lower boundary, the soft limit, is defined as a limit that is per default unset. As a result, the set of cgroups that global reclaim prefers is opt-in, rather than opt-out. The costs for optimizing these mostly negative lookups are so high that the implementation, despite its enormous size, does not even provide the basic desirable behavior. First off, the soft limit has no hierarchical meaning. All configured groups are organized in a global rbtree and treated like equal peers, regardless where they are located in the hierarchy. This makes subtree delegation impossible. Second, the soft limit reclaim pass is so aggressive that it not just introduces high allocation latencies into the system, but also impacts system performance due to overreclaim, to the point where the feature becomes self-defeating.

> 原始的下限，即软限制，被定义为默认情况下未设置的限制。因此，全局回收更喜欢的一组cgroups是选择加入，而不是选择退出。优化这些大多是负查找的成本非常高，以至于尽管实现规模巨大，但它甚至不能提供基本的理想行为。首先，软限制没有等级意义。所有配置的组都组织在全局rbtree中，并被视为平等的对等体，无论它们在层次结构中的位置如何。这使得子树委派成为不可能。其次，软限制回收过程过于激进，不仅给系统带来了高分配延迟，而且由于过度声明而影响了系统性能，以至于该功能变得弄巧成拙。


The memory.low boundary on the other hand is a top-down allocated reserve. A cgroup enjoys reclaim protection when it\'s within its effective low, which makes delegation of subtrees possible. It also enjoys having reclaim pressure proportional to its overage when above its effective low.

> 另一方面，内存的低边界是自上而下分配的储备。当一个cgroup处于其有效低位时，它可以获得回收保护，这使得子树的委派成为可能。当超过其有效低点时，它还喜欢回收压力与其过大成比例。


The original high boundary, the hard limit, is defined as a strict limit that can not budge, even if the OOM killer has to be called. But this generally goes against the goal of making the most out of the available memory. The memory consumption of workloads varies during runtime, and that requires users to overcommit. But doing that with a strict upper limit requires either a fairly accurate prediction of the working set size or adding slack to the limit. Since working set size estimation is hard and error prone, and getting it wrong results in OOM kills, most users tend to err on the side of a looser limit and end up wasting precious resources.

> 最初的高边界，即硬极限，被定义为一个不能移动的严格极限，即使必须调用OOM杀手。但这通常违背了充分利用可用内存的目标。工作负载的内存消耗在运行时会有所不同，这需要用户过度投入。但要在严格的上限下做到这一点，要么需要对工作集大小进行相当准确的预测，要么在极限上增加松弛度。由于工作集大小估计很难而且容易出错，错误的估计会导致OOM死亡，因此大多数用户倾向于选择更宽松的限制，最终浪费宝贵的资源。


The memory.high boundary on the other hand can be set much more conservatively. When hit, it throttles allocations by forcing them into direct reclaim to work off the excess, but it never invokes the OOM killer. As a result, a high boundary that is chosen too aggressively will not terminate the processes, but instead it will lead to gradual performance degradation. The user can monitor this and make corrections until the minimal memory footprint that still gives acceptable performance is found.

> 另一方面，内存的高边界可以设置得保守得多。当被击中时，它会通过迫使分配直接回收来消除多余的部分来抑制分配，但它从不调用OOM杀手。因此，过于激进地选择高边界不会终止过程，反而会导致性能逐渐下降。用户可以对此进行监控并进行更正，直到找到仍能提供可接受性能的最小内存占用。


In extreme cases, with many concurrent allocations and a complete breakdown of reclaim progress within the group, the high boundary can be exceeded. But even then it\'s mostly better to satisfy the allocation from the slack available in other groups or the rest of the system than killing the group. Otherwise, memory.max is there to limit this type of spillover and ultimately contain buggy or even malicious applications.

> 在极端情况下，由于许多并发分配和组内回收进度的完全分解，可能会超出上限。但即便如此，从其他组或系统其他部分的空闲中满足分配也比杀死组要好得多。否则，memory.max会限制这种类型的溢出，并最终包含有缺陷甚至恶意的应用程序。


Setting the original memory.limit_in_bytes below the current usage was subject to a race condition, where concurrent charges could cause the limit setting to fail. memory.max on the other hand will first set the limit to prevent new charges, and then reclaim and OOM kill until the new limit is met - or the task writing to memory.max is killed.

> 将原始内存.limit_in_bytes设置为低于当前使用量会受到竞争条件的影响，同时收费可能会导致限制设置失败。另一方面，memory.max将首先设置限制以防止新的电荷，然后回收并OOM杀死，直到达到新的限制，或者写入memory.max的任务被杀死。


The combined memory+swap accounting and limiting is replaced by real control over swap space.

> 存储器+交换记账和限制的组合被对交换空间的实际控制所取代。


The main argument for a combined memory+swap facility in the original cgroup design was that global or parental pressure would always be able to swap all anonymous memory of a child group, regardless of the child\'s own (possibly untrusted) configuration. However, untrusted groups can sabotage swapping by other means - such as referencing its anonymous memory in a tight loop - and an admin can not assume full swappability when overcommitting untrusted jobs.

> 在最初的cgroup设计中，组合内存+交换功能的主要论点是，全局或父母的压力总是能够交换子组的所有匿名内存，而不管子组自己（可能不受信任）的配置如何。然而，不受信任的组可以通过其他方式破坏交换，例如在紧密循环中引用其匿名内存，并且管理员在过度提交不受信任作业时不能假设完全的可交换性。


For trusted jobs, on the other hand, a combined counter is not an intuitive userspace interface, and it flies in the face of the idea that cgroup controllers should account and limit specific physical resources. Swap space is a resource like all others in the system, and that\'s why unified hierarchy allows distributing it separately.

> 另一方面，对于受信任的作业，组合计数器不是一个直观的用户空间界面，它违背了cgroup控制器应该考虑和限制特定物理资源的想法。交换空间和系统中的其他资源一样是一种资源，这就是为什么统一的层次结构允许单独分配它。
