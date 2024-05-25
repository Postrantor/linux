---
tip: translate by baidu@2024-01-16 22:46:24
title: CFS Scheduler
---

# 1. OVERVIEW

CFS stands for "Completely Fair Scheduler," and is the new "desktop" process scheduler implemented by Ingo Molnar and merged in Linux 2.6.23. It is the replacement for the previous vanilla scheduler\'s SCHED_OTHER interactivity code.

> CFS 代表“完全公平的调度器”，是由 Ingo Molnar 实现并在 Linux 2.6.23 中合并的新的“桌面”进程调度器。它取代了以前的普通调度器的 SCHED_OTHER 交互代码。

80% of CFS\'s design can be summed up in a single sentence: CFS basically models an "ideal, precise multi-tasking CPU" on real hardware.

> CFS 80%的设计可以用一句话来概括：CFS 基本上是在真实硬件上为“理想、精确的多任务 CPU”建模。

"Ideal multi-tasking CPU" is a `(non-existent :-))` CPU that has 100% physical power and which can run each task at precise equal speed, in parallel, each at 1/nr_running speed. For example: if there are 2 tasks running, then it runs each at 50% physical power --- i.e., actually in parallel.

> “理想的多任务 CPU”是一种“(不存在：-)”CPU，它具有 100%的物理能力，可以以精确相等的速度并行运行每个任务，每个任务的运行速度为 1/nr_running。例如：如果有两个任务在运行，那么它每个任务都以 50%的物理功率运行——也就是说，实际上是并行的。

On real hardware, we can run only a single task at once, so we have to introduce the concept of "virtual runtime." The virtual runtime of a task specifies when its next timeslice would start execution on the ideal multi-tasking CPU described above. In practice, the virtual runtime of a task is its actual runtime normalized to the total number of running tasks.

> 在实际硬件上，我们一次只能运行一个任务，因此我们必须引入“虚拟运行时”的概念。任务的虚拟运行时指定其下一个时间片何时在上述理想的多任务 CPU 上开始执行。在实践中，任务的虚拟运行时是将其实际运行时标准化为正在运行的任务总数。

# 2. FEW IMPLEMENTATION DETAILS

In CFS the virtual runtime is expressed and tracked via the per-task `p->se.vruntime` (nanosec-unit) value. This way, it\'s possible to accurately timestamp and measure the "expected CPU time" a task should have gotten.

> 在 CFS 中，虚拟运行时通过每个任务的“p->se.vruntime”(纳米秒单位)值来表示和跟踪。这样，就可以准确地对任务应该得到的“预期 CPU 时间”进行时间戳和测量。

Small detail: on "ideal" hardware, at any time all tasks would have the same `p->se.vruntime` value --- i.e., tasks would execute simultaneously and no task would ever get "out of balance" from the "ideal" share of CPU time.

> 小细节：在“理想”硬件上，任何时候所有任务都将具有相同的`p->se.vruntime`值，即任务将同时执行，任何任务都不会从“理想”的 CPU 时间份额中“失去平衡”。

CFS\'s task picking logic is based on this `p->se.vruntime` value and it is thus very simple: it always tries to run the task with the smallest `p->se.vruntime` value (i.e., the task which executed least so far). CFS always tries to split up CPU time between runnable tasks as close to "ideal multitasking hardware" as possible.

> CFS 的任务选择逻辑基于这个`p->se.vruntime`值，因此它非常简单：它总是试图运行具有最小`p->se.vruntime`值的任务(即迄今为止执行最少的任务)。CFS 总是试图在尽可能接近“理想的多任务硬件”的可运行任务之间分配 CPU 时间。

Most of the rest of CFS\'s design just falls out of this really simple concept, with a few add-on embellishments like nice levels, multiprocessing and various algorithm variants to recognize sleepers.

> CFS 的其余大部分设计都脱离了这个非常简单的概念，添加了一些附加的装饰，如漂亮的级别、多处理和各种算法变体来识别睡眠者。

# 3. THE RBTREE

CFS\'s design is quite radical: it does not use the old data structures for the runqueues, but it uses a time-ordered rbtree to build a "timeline" of future task execution, and thus has no "array switch" artifacts (by which both the previous vanilla scheduler and RSDL/SD are affected).

> CFS 的设计相当激进：它不使用旧的运行队列数据结构，而是使用时间排序的 rbtree 来构建未来任务执行的“时间表”，因此没有“阵列切换”工件(之前的香草调度器和 RSDL/SD 都会受到影响)。

CFS also maintains the `rq->cfs.min_vruntime` value, which is a monotonic increasing value tracking the smallest vruntime among all tasks in the runqueue. The total amount of work done by the system is tracked using min_vruntime; that value is used to place newly activated entities on the left side of the tree as much as possible.

> CFS 还维护`rq->cfs.min_vruntime`值，这是一个单调递增的值，用于跟踪运行队列中所有任务中最小的 vruntime。使用 min_vruntime 跟踪系统完成的总工作量；该值用于将新激活的实体尽可能多地放置在树的左侧。

The total number of running tasks in the runqueue is accounted through the `rq->cfs.load` value, which is the sum of the weights of the tasks queued on the runqueue.

> 运行队列中正在运行的任务总数通过“rq->cfs.load”值来计算，该值是运行队列中排队任务的权重之和。

CFS maintains a time-ordered rbtree, where all runnable tasks are sorted by the `p->se.vruntime` key. CFS picks the "leftmost" task from this tree and sticks to it. As the system progresses forwards, the executed tasks are put into the tree more and more to the right --- slowly but surely giving a chance for every task to become the "leftmost task" and thus get on the CPU within a deterministic amount of time.

> CFS 维护一个按时间排序的 rbtree，其中所有可运行的任务都按`p->se.vruntime`键排序。CFS 从这棵树中选择“最左边”的任务并坚持执行。随着系统的前进，执行的任务越来越向右放入树中——缓慢但肯定地给每个任务一个机会成为“最左边的任务”，从而在确定的时间内进入 CPU。

Summing up, CFS works like this: it runs a task a bit, and when the task schedules (or a scheduler tick happens) the task\'s CPU usage is "accounted for": the (small) time it just spent using the physical CPU is added to `p->se.vruntime`. Once `p->se.vruntime` gets high enough so that another task becomes the "leftmost task" of the time-ordered rbtree it maintains (plus a small amount of "granularity" distance relative to the leftmost task so that we do not over-schedule tasks and trash the cache), then the new leftmost task is picked and the current task is preempted.

> 总之，CFS 是这样工作的：它稍微运行一个任务，当任务调度(或发生调度勾选)时，任务的 CPU 使用率会被“考虑”：它刚刚使用物理 CPU 的(小)时间会被添加到`p->se.vruntime`中。一旦`p->se.vruntime`变得足够高，另一个任务就会成为它所维护的时间排序 rbtree 中“最左边的任务”(加上少量的“粒度”相对于最左边任务的距离，这样我们就不会过度调度任务并丢弃缓存)，然后选择新的最左边任务并抢占当前任务。

# 4. SOME FEATURES OF CFS

CFS uses nanosecond granularity accounting and does not rely on any jiffies or other HZ detail. Thus the CFS scheduler has no notion of "timeslices" in the way the previous scheduler had, and has no heuristics whatsoever. There is only one central tunable (you have to switch on CONFIG_SCHED_DEBUG):

> CFS 使用纳秒粒度的核算，不依赖于任何 jiffies 或其他 HZ 细节。因此，CFS 调度器不像以前的调度器那样有“时间片”的概念，也没有任何启发式方法。只有一个中央可调(您必须打开 CONFIG_SCHED_DEBUG)：

> /sys/kernel/debug/sched/base_slice_ns

which can be used to tune the scheduler from "desktop" (i.e., low latencies) to "server" (i.e., good batching) workloads. It defaults to a setting suitable for desktop workloads. `SCHED_BATCH` is handled by the CFS scheduler module too.

> 它可以用于将调度器从“桌面”(即低延迟)调整为“服务器”(即良好的批处理)工作负载。它默认为适用于桌面工作负载的设置`SCHED_BATCH`也由 CFS 调度程序模块处理。

Due to its design, the CFS scheduler is not prone to any of the "attacks" that exist today against the heuristics of the stock scheduler: "fiftyp.c, thud.c, chew.c, ring-test.c, massive_intr.c" all work fine and do not impact interactivity and produce the expected behavior.

> 由于其设计，CFS 调度器不容易受到当今存在的任何针对股票调度器启发式的“攻击”：“fiftyp.c、thud.c、chew.c、ring test.c、massive_intr.c”都工作良好，不会影响交互并产生预期行为。

The CFS scheduler has a much stronger handling of nice levels and `SCHED_BATCH` than the previous vanilla scheduler: both types of workloads are isolated much more aggressively.

> 与以前的普通调度器相比，CFS 调度器对良好级别和“SCHED_BATCH”的处理能力更强：这两种类型的工作负载都被隔离得更积极。

SMP load-balancing has been reworked/sanitized: the runqueue-walking assumptions are gone from the load-balancing code now, and iterators of the scheduling modules are used. The balancing code got quite a bit simpler as a result.

> SMP 负载平衡已经被重新设计/净化：运行队列遍历假设现在已经从负载平衡代码中删除，并且使用了调度模块的迭代器。因此，平衡代码变得简单多了。

# 5. Scheduling policies

CFS implements three scheduling policies:

- `SCHED_NORMAL` (traditionally called SCHED_OTHER): The scheduling policy that is used for regular tasks.

> -“SCHED_NORAL”(传统上称为 SCHED_OTHER)：用于常规任务的计划策略。

- `SCHED_BATCH`: Does not preempt nearly as often as regular tasks would, thereby allowing tasks to run longer and make better use of caches but at the cost of interactivity. This is well suited for batch jobs.

> -“SCHED_BATCH”：不会像常规任务那样频繁地抢占，从而允许任务运行更长时间，更好地利用缓存，但会以交互为代价。这非常适合批量作业。

- `SCHED_IDLE`: This is even weaker than nice 19, but its not a true idle timer scheduler in order to avoid to get into priority inversion problems which would deadlock the machine.

> -“SCHED_IDLE”：这甚至比 nice 19 弱，但它不是一个真正的空闲定时器调度器，以避免出现优先级反转问题，从而使机器死锁。

`SCHED_FIFO/RR` are implemented in "sched/rt.c" and are as specified by POSIX.

The command `chrt` from util-linux-ng 2.13.1.1 can set all of these except `SCHED_IDLE`.

> util linux ng 2.13.1.1 中的命令“chrt”可以设置除“SCHED_IDLE”之外的所有这些。

# 6. SCHEDULING CLASSES

The new CFS scheduler has been designed in such a way to introduce "Scheduling Classes," an extensible hierarchy of scheduler modules. These modules encapsulate scheduling policy details and are handled by the scheduler core without the core code assuming too much about them.

> 新的 CFS 调度器的设计方式是引入“调度类”，这是一种可扩展的调度器模块层次结构。这些模块封装了调度策略细节，并由调度程序核心处理，而核心代码不会对它们进行过多假设。

sched/fair.c implements the CFS scheduler described above.

sched/rt.c implements SCHED_FIFO and SCHED_RR semantics, in a simpler way than the previous vanilla scheduler did. It uses 100 runqueues (for all 100 RT priority levels, instead of 140 in the previous scheduler) and it needs no expired array.

> sched/rt.c 以比以前的 vanilla 调度器更简单的方式实现了 sched_FIFO 和 sched_RR 语义。它使用 100 个运行队列(用于所有 100 个 RT 优先级，而不是以前调度程序中的 140 个)，并且不需要过期的数组。

Scheduling classes are implemented through the sched_class structure, which contains hooks to functions that must be called whenever an interesting event occurs.

> 调度类是通过 sched_class 结构实现的，该结构包含到函数的挂钩，每当发生感兴趣的事件时都必须调用这些挂钩。

This is the (partial) list of the hooks:

- enqueue_task(\...)

Called when a task enters a runnable state. It puts the scheduling entity (task) into the red-black tree and increments the nr_running variable.

> 当任务进入可运行状态时调用。它将调度实体(任务)放入红黑树中，并递增 nr_running 变量。

- dequeue_task(\...)

When a task is no longer runnable, this function is called to keep the corresponding scheduling entity out of the red-black tree. It decrements the nr_running variable.

> 当任务不再可运行时，会调用此函数以将相应的调度实体排除在红黑树之外。它递减 nr_running 变量。

- yield_task(\...)

This function is basically just a dequeue followed by an enqueue, unless the compat_yield sysctl is turned on; in that case, it places the scheduling entity at the right-most end of the red-black tree.

> 这个函数基本上只是一个出队列，后面跟着一个入队，除非 compat_yield sysctl 打开；在这种情况下，它将调度实体放置在红黑树的最右端。

- check_preempt_curr(\...)

This function checks if a task that entered the runnable state should preempt the currently running task.

> 此函数检查进入可运行状态的任务是否应抢占当前正在运行的任务。

- pick_next_task(\...)

This function chooses the most appropriate task eligible to run next.

- set_curr_task(\...)

This function is called when a task changes its scheduling class or changes its task group.

> 当任务更改其调度类或更改其任务组时，会调用此函数。

- task_tick(\...)

This function is mostly called from time tick functions; it might lead to process switch. This drives the running preemption.

> 此函数主要由时间刻度函数调用；这可能会导致进程切换。这驱动了正在运行的抢占。

# 7. GROUP SCHEDULER EXTENSIONS TO CFS

Normally, the scheduler operates on individual tasks and strives to provide fair CPU time to each task. Sometimes, it may be desirable to group tasks and provide fair CPU time to each such task group. For example, it may be desirable to first provide fair CPU time to each user on the system and then to each task belonging to a user.

> 通常情况下，调度器对单个任务进行操作，并努力为每个任务提供公平的 CPU 时间。有时，可能希望对任务进行分组，并为每个这样的任务组提供公平的 CPU 时间。例如，可能希望首先向系统上的每个用户提供公平的 CPU 时间，然后向属于用户的每个任务提供公平的 CPU-时间。

`CONFIG_CGROUP_SCHED` strives to achieve exactly that. It lets tasks to be grouped and divides CPU time fairly among such groups.

`CONFIG_RT_GROUP_SCHED` permits to group real-time (i.e., SCHED_FIFO and SCHED_RR) tasks.

`CONFIG_FAIR_GROUP_SCHED` permits to group CFS (i.e., `SCHED_NORMAL` and `SCHED_BATCH`) tasks.

> These options need CONFIG_CGROUPS to be defined, and let the administrator create arbitrary groups of tasks, using the "cgroup" pseudo filesystem. See Documentation/admin-guide/cgroup-v1/cgroups.rst for more information about this filesystem.

When CONFIG_FAIR_GROUP_SCHED is defined, a "cpu.shares" file is created for each group created using the pseudo filesystem. See example steps below to create task groups and modify their CPU share using the "cgroups" pseudo filesystem:

> 定义 CONFIG_FAIR_GROUP_SCHED 后，将为使用伪文件系统创建的每个组创建一个“cpu.shares”文件。请参阅以下示例步骤，以使用“cgroups”伪文件系统创建任务组并修改其 CPU 共享：

```sh
    # mount -t tmpfs cgroup_root /sys/fs/cgroup
    # mkdir /sys/fs/cgroup/cpu
    # mount -t cgroup -ocpu none /sys/fs/cgroup/cpu
    # cd /sys/fs/cgroup/cpu

    # mkdir multimedia  # create "multimedia" group of tasks
    # mkdir browser     # create "browser" group of tasks

    # #Configure the multimedia group to receive twice the CPU bandwidth
    # #that of browser group

    # echo 2048 > multimedia/cpu.shares
    # echo 1024 > browser/cpu.shares

    # firefox & # Launch firefox and move it to "browser" group
    # echo <firefox_pid> > browser/tasks

    # #Launch gmplayer (or your favourite movie player)
    # echo <movie_player_pid> > multimedia/tasks
```
