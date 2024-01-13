---
tip: translate by baidu@2024-01-11 22:57:37
title: Completions - \"wait for completion\" barrier APIs
---

# Introduction:

If you have one or more threads that must wait for some kernel activity to have reached a point or a specific state, completions can provide a race-free solution to this problem. Semantically they are somewhat like a pthread_barrier() and have similar use-cases.

> 如果您有一个或多个线程必须等待某些内核活动达到某个点或特定状态，则完成可以为这个问题提供无竞争的解决方案。从语义上讲，它们有点像 pthread_barrier()，并且有类似的用例。

Completions are a code synchronization mechanism which is preferable to any misuse of locks/semaphores and busy-loops. Any time you think of using yield() or some quirky msleep(1) loop to allow something else to proceed, you probably want to look into using one of the `wait_for_completion()` calls and `complete()` instead.

> 完成是一种代码同步机制，它比滥用锁/信号量和繁忙循环更可取。每当你想到使用 yield()或一些古怪的 msleep(1)循环来允许其他事情继续进行时，你可能想考虑使用其中一个`wait_for_completion()`调用和`complete()`。

The advantage of using completions is that they have a well defined, focused purpose which makes it very easy to see the intent of the code, but they also result in more efficient code as all threads can continue execution until the result is actually needed, and both the waiting and the signalling is highly efficient using low level scheduler sleep/wakeup facilities.

> 使用完成的优点是，它们有一个明确的、有重点的目的，这使得很容易看到代码的意图，但它们也会产生更高效的代码，因为所有线程都可以继续执行，直到实际需要结果，并且使用低级别调度程序睡眠/唤醒设施，等待和信令都是高效的。

Completions are built on top of the waitqueue and wakeup infrastructure of the Linux scheduler. The event the threads on the waitqueue are waiting for is reduced to a simple flag in \'struct completion\', appropriately called \"done\".

> 完成构建在 Linux 调度器的等待队列和唤醒基础设施之上。等待队列上的线程正在等待的事件在`结构完成`中被简化为一个简单的标志，适当地称为`完成`。

As completions are scheduling related, the code can be found in "kernel/sched/completion.c".

> 由于完成与调度相关，代码可以在`kernel/sched/completion.c`中找到。

# Usage:

There are three main parts to using completions:

- the initialization of the \'struct completion\' synchronization object
- the waiting part through a call to one of the variants of `wait_for_completion()`,
- the signaling side through a call to `complete()` or complete_all().

There are also some helper functions for checking the state of completions. Note that while initialization must happen first, the waiting and signaling part can happen in any order. I.e. it\'s entirely normal for a thread to have marked a completion as \'done\' before another thread checks whether it has to wait for it.

> 还有一些帮助函数用于检查完成的状态。请注意，虽然初始化必须首先进行，但等待和信令部分可以按任何顺序进行。也就是说，在另一个线程检查是否必须等待完成之前，一个线程将完成标记为`完成`是完全正常的。

To use completions you need to `#include <linux/completion.h>` and create a static or dynamic variable of type \'struct completion\', which has only two fields:

> 要使用 completion，您需要`#include<linux/completion.h>`并创建一个类型为`struct-completion`的静态或动态变量，该变量只有两个字段：

```c
struct completion {
    unsigned int done;
    wait_queue_head_t wait;
};
```

This provides the `->wait` waitqueue to place tasks on for waiting (if any), and the `->done` completion flag for indicating whether it\'s completed or not.

> 这提供了`->wait`等待队列，用于放置等待任务(如果有)，以及`->done`完成标志，用于指示任务是否完成。

Completions should be named to refer to the event that is being synchronized on. A good example is:

> 应命名完成以引用正在同步的事件。一个很好的例子是：

```c
wait_for_completion(&early_console_added);

complete(&early_console_added);
```

Good, intuitive naming (as always) helps code readability. Naming a completion \'complete\' is not helpful unless the purpose is super obvious\...

> 良好、直观的命名(一如既往)有助于代码的可读性。将完成命名为`完成`是没有帮助的，除非目的非常明显。。。

# Initializing completions:

Dynamically allocated completion objects should preferably be embedded in data structures that are assured to be alive for the life-time of the function/driver, to prevent races with asynchronous `complete()` calls from occurring.

> 动态分配的完成对象最好嵌入到数据结构中，这些数据结构保证在函数/驱动程序的整个生命周期内都是活动的，以防止异步`complete()`调用的竞争发生。

Particular care should be taken when using the `timeout()` or `killable()`/`interruptible()` variants of wait_for_completion(), as it must be assured that memory de-allocation does not happen until all related activities (`complete()` or `reinit_completion()`) have taken place, even if these wait functions return prematurely due to a timeout or a signal triggering.

> 使用 wait_for_completion()的`timeout()`或`killable()`/`interruptible()`变体时应特别小心，因为必须确保在所有相关活动(`complete()`和`reinit_completion`)发生之前不会发生内存减配，即使这些等待函数由于超时或信号触发而提前返回。

Initializing of dynamically allocated completion objects is done via a call to init_completion():

> 动态分配的完成对象的初始化是通过调用 init_completion()完成的：

```c
init_completion(&dynamic_object->done);
```

In this call we initialize the waitqueue and set `->done` to 0, i.e. \"not completed\" or \"not done\".

> 在这个调用中，我们初始化等待队列，并将`->done`设置为 0，即\`not completed\`或\`not done\`。

The re-initialization function, `reinit_completion()`, simply resets the `->done` field to 0 (\"not done\"), without touching the waitqueue. Callers of this function must make sure that there are no racy `wait_for_completion()` calls going on in parallel.

> 重新初始化函数`reinit_completion()`只需将`->done`字段重置为 0(\`not done`)，而不必接触等待队列。此函数的调用方必须确保没有 racy`wait_for_completion()`调用并行进行。

Calling init_completion() on the same completion object twice is most likely a bug as it re-initializes the queue to an empty queue and enqueued tasks could get \"lost\" - use `reinit_completion()` in that case, but be aware of other races.

> 对同一个完成对象调用 init_completion()两次很可能是一个错误，因为它将队列重新初始化为空队列，排队的任务可能会`丢失`-在这种情况下，请使用`reinit_completion`，但要注意其他种族。

For static declaration and initialization, macros are available.

For static (or global) declarations in file scope you can use DECLARE_COMPLETION():

> 对于文件作用域中的静态(或全局)声明，可以使用 DECLARE_COCOMPLETION()：

```c
    static DECLARE_COMPLETION(setup_done);
    DECLARE_COMPLETION(setup_done);
```

Note that in this case the completion is boot time (or module load time) initialized to \'not done\' and doesn\'t require an `init_completion()` call.

> 请注意，在这种情况下，完成是初始化为`未完成`的启动时间(或模块加载时间)，不需要`init_completion()`调用。

When a completion is declared as a local variable within a function, then the initialization should always use `DECLARE_COMPLETION_ONSTACK()` explicitly, not just to make lockdep happy, but also to make it clear that limited scope had been considered and is intentional:

> 当完成被声明为函数中的局部变量时，初始化应该始终显式地使用`DECLARE_OCOMPLEON_ONSTACK()`，这不仅是为了让 lockdep 满意，而且是为了清楚地表明已经考虑了有限的范围，并且是有意的：

```c
DECLARE_COMPLETION_ONSTACK(setup_done)
```

Note that when using completion objects as local variables you must be acutely aware of the short life time of the function stack: the function must not return to a calling context until all activities (such as waiting threads) have ceased and the completion object is completely unused.

> 请注意，当使用完成对象作为局部变量时，您必须敏锐地意识到函数堆栈的使用寿命很短：在所有活动(如等待线程)停止且完成对象完全未使用之前，函数不得返回到调用上下文。

To emphasise this again: in particular when using some of the waiting API variants with more complex outcomes, such as the timeout or signalling (`timeout()`, `killable()` and `interruptible()`) variants, the wait might complete prematurely while the object might still be in use by another thread - and a return from the `wait_on_completion()` caller function will deallocate the function stack and cause subtle data corruption if a `complete()` is done in some other thread. Simple testing might not trigger these kinds of races.

> 再次强调这一点：特别是当使用一些具有更复杂结果的等待 API 变体时，例如超时或信令(`timeout()`、`killable()`和`interruptible()`)变体，等待可能会提前完成，而对象可能仍在被另一个线程使用-如果在其他线程中执行了`complete()`，则`wait_on_completion()`调用方函数的返回将解除分配函数堆栈并导致微妙的数据损坏。简单的测试可能不会引发这类比赛。

If unsure, use dynamically allocated completion objects, preferably embedded in some other long lived object that has a boringly long life time which exceeds the life time of any helper threads using the completion object, or has a lock or other synchronization mechanism to make sure `complete()` is not called on a freed object.

> 如果不确定，请使用动态分配的完成对象，最好嵌入其他一些长寿命对象中，该对象的生存时间长得令人厌烦，超过了使用完成对象的任何辅助线程的生存时间，或者具有锁或其他同步机制，以确保不会对释放的对象调用`complete()`。

A naive `DECLARE_COMPLETION()` on the stack triggers a lockdep warning.

# Waiting for completions:

For a thread to wait for some concurrent activity to finish, it calls `wait_for_completion()` on the initialized completion structure:

> 对于等待某些并发活动完成的线程，它会对初始化的完成结构调用`wait_For_completion()`：

```c
void wait_for_completion(struct completion *done)
```

A typical usage scenario is:

```c
CPU#1                   CPU#2

struct completion setup_done;

init_completion(&setup_done);
initialize_work(...,&setup_done,...);

/* run non-dependent code */        /* do setup */

wait_for_completion(&setup_done);   complete(&setup_done);
```

This is not implying any particular order between wait_for_completion() and the call to complete() - if the call to `complete()` happened before the call to wait_for_completion() then the waiting side simply will continue immediately as all dependencies are satisfied; if not, it will block until completion is signaled by `complete()`.

> 这并不意味着在 wait_for_completion()和对 complete()的调用之间有任何特定的顺序——如果对`complete(；否则，它将阻塞，直到`complete()`发出完成信号。

Note that `wait_for_completion()` is calling spin_lock_irq()/spin_unlock_irq(), so it can only be called safely when you know that interrupts are enabled. Calling it from IRQs-off atomic contexts will result in hard-to-detect spurious enabling of interrupts.

> 请注意，`wait_for_completion()`正在调用 spin_lock_irq()/spin_unlock_irq()，因此只有当您知道已启用中断时，才能安全地调用它。从原子上下文之外的 IRQ 调用它将导致很难检测到虚假的中断启用。

The default behavior is to wait without a timeout and to mark the task as uninterruptible. `wait_for_completion()` and its variants are only safe in process context (as they can sleep) but not in atomic context, interrupt context, with disabled IRQs, or preemption is disabled - see also try\_`wait_for_completion()` below for handling completion in atomic/interrupt context.

> 默认行为是等待而不超时，并将任务标记为不间断`wait_for_completion()`及其变体仅在进程上下文中是安全的(因为它们可以休眠)，但在原子上下文、中断上下文、禁用 IRQ 或抢占被禁用的情况下是不安全的-有关在原子/中断上下文中处理完成的信息，请参阅下面的 try\_`wait_for-completion(()`。

As all variants of `wait_for_completion()` can (obviously) block for a long time depending on the nature of the activity they are waiting for, so in most cases you probably don\'t want to call this with held mutexes.

> 由于`wait_for_completion()`的所有变体都可能(显然)阻塞很长一段时间，这取决于它们所等待的活动的性质，所以在大多数情况下，您可能不想用持有的互斥体来调用它。

# wait_for_completion\*() variants available:

The below variants all return status and this status should be checked in most(/all) cases - in cases where the status is deliberately not checked you probably want to make a note explaining this (e.g. see "arch/arm/kernel/smp.c:\_\_cpu_up()").

> 以下变体都返回状态，并且在大多数(/all)情况下都应该检查此状态-在故意不检查状态的情况下，您可能需要对此进行说明(例如，请参阅`arch/arm/kernel/smp.c:\_\_cpu_up()`)。

A common problem that occurs is to have unclean assignment of return types, so take care to assign return-values to variables of the proper type.

> 发生的一个常见问题是返回类型的赋值不干净，所以要注意将返回值分配给正确类型的变量。

Checking for the specific meaning of return values also has been found to be quite inaccurate, e.g. constructs like:

> 检查返回值的具体含义也被发现是非常不准确的，例如结构，如：

```c
if (!wait_for_completion_interruptible_timeout(...))
```

... would execute the same code path for successful completion and for the interrupted case - which is probably not what you want:

> …将执行相同的代码路径以成功完成和中断情况——这可能不是您想要的：

```c
int wait_for_completion_interruptible(struct completion *done)
```

This function marks the task `TASK_INTERRUPTIBLE` while it is waiting. If a signal was received while waiting it will return `-ERESTARTSYS`; 0 otherwise:

> 此函数在任务等待时标记任务`task_INTERRUPTIBLE`。如果在等待时接收到一个信号，它将返回`-ERESTARTSYS`；0 否则：

```c
unsigned long wait_for_completion_timeout(struct completion *done, unsigned long timeout)
```

The task is marked as TASK_UNINTERRUPTIBLE and will wait at most \'timeout\' jiffies. If a timeout occurs it returns 0, else the remaining time in jiffies (but at least 1).

> 该任务标记为 task_UNINTERRUPTIBLE，最多等待`超时`jiffies。如果发生超时，则返回 0，否则剩余时间以 jiffies 为单位(但至少为 1)。

Timeouts are preferably calculated with `msecs_to_jiffies()` or `usecs_to_jiffies()`, to make the code largely HZ-invariant.

> 超时最好用`msecs_to_jiffies()`或`usecs_to_jiffies`()来计算，以使代码在很大程度上保持 HZ 不变。

If the returned timeout value is deliberately ignored a comment should probably explain why (e.g. see drivers/mfd/wm8350-core.c `wm8350_read_auxadc()`):

> 如果故意忽略返回的超时值，则注释可能会解释原因(例如，请参见 drivers/mfd/wm8350 core.c `wm8350_read_auxadc()`)：

```c
long wait_for_completion_interruptible_timeout(struct completion *done, unsigned long timeout)
```

This function passes a timeout in jiffies and marks the task as TASK_INTERRUPTIBLE. If a signal was received it will return -ERESTARTSYS; otherwise it returns 0 if the completion timed out, or the remaining time in jiffies if completion occurred.

> 此函数以 jiffies 形式传递超时，并将任务标记为 task_INTERRUPTIBLE。如果收到信号，它将返回-ERESTARTSYS；否则，如果完成超时，则返回 0；如果完成发生，则返回 jiffies 中的剩余时间。

Further variants include `killable` which uses TASK_KILLABLE as the designated tasks state and will return -ERESTARTSYS if it is interrupted, or 0 if completion was achieved. There is a `timeout` variant as well:

> 进一步的变体包括`kilable`，它使用 TASK_killable 作为指定的任务状态，如果中断，将返回-ERESTARTSYS，如果完成，则返回 0。还有一种`超时`变体：

```c
long wait_for_completion_killable(struct completion *done)
long wait_for_completion_killable_timeout(struct completion *done, unsigned long timeout)
```

The `io` variants wait_for_completion_io() behave the same as the `non-io` variants, except for accounting waiting time as \'waiting on IO\', which has an impact on how the task is accounted in scheduling/IO stats:

> `io`变体 wait_for_completion_io()的行为与`非 io`变体相同，只是将等待时间计算为`等待 io`，这会影响任务在计划/io 统计中的计算方式：

```c
    void wait_for_completion_io(struct completion *done)
    unsigned long wait_for_completion_io_timeout(struct completion *done, unsigned long timeout)
```

# Signaling completions:

A thread that wants to signal that the conditions for continuation have been achieved calls `complete()` to signal exactly one of the waiters that it can continue:

> 想要发出信号表示已达到继续条件的线程调用`complete()`来向其中一个等待者发出信号，表明它可以继续：

```c
void complete(struct completion *done)
```

\... or calls complete_all() to signal all current and future waiters:

```c
void complete_all(struct completion *done)
```

The signaling will work as expected even if completions are signaled before a thread starts waiting. This is achieved by the waiter \"consuming\" (decrementing) the done field of \'struct completion\'. Waiting threads wakeup order is the same in which they were enqueued (FIFO order).

> 即使在线程开始等待之前发出了完成信号，该信号也将按预期工作。这是通过服务员`消耗`(递减)`结构完成`的 done 字段来实现的。等待线程唤醒的顺序与它们入队的顺序相同(FIFO 顺序)。

If `complete()` is called multiple times then this will allow for that number of waiters to continue - each call to `complete()` will simply increment the done field. Calling complete_all() multiple times is a bug though. Both `complete()` and complete_all() can be called in IRQ/atomic context safely.

> 如果多次调用`complete()`，那么这将允许该数量的等待程序继续进行——每次调用`complete()`都只会增加 done 字段。不过，多次调用 complete_all()是一个错误。`complete()`和 complete_all()都可以在 IRQ/atomic 上下文中安全地调用。

There can only be one thread calling `complete()` or complete_all() on a particular \'struct completion\' at any time - serialized through the wait queue spinlock. Any such concurrent calls to `complete()` or complete_all() probably are a design bug.

> 任何时候都只能有一个线程在特定的`结构完成`上调用`complete()`或 complete_all()-通过等待队列自旋锁序列化。对`complete()`或 complete_all()的任何此类并发调用都可能是设计错误。

Signaling completion from IRQ context is fine as it will appropriately lock with spin_lock_irqsave()/spin_unlock_irqrestore() and it will never sleep.

> 来自 IRQ 上下文的信号完成是很好的，因为它将适当地锁定 spin_lock_irqsave()/spin_unlock_irqrestore()，并且永远不会休眠。

# try_wait_for_completion()/completion_done():

The try_wait_for_completion() function will not put the thread on the wait queue but rather returns false if it would need to enqueue (block) the thread, else it consumes one posted completion and returns true:

> try_wait_for_completion()函数不会将线程放入等待队列，而是在需要将线程排队(阻止)时返回 false，否则它将消耗一个已发布的完成并返回 true：

```c
bool try_wait_for_completion(struct completion *done)
```

Finally, to check the state of a completion without changing it in any way, call `completion_done()`, which returns false if there are no posted completions that were not yet consumed by waiters (implying that there are waiters) and true otherwise:

> 最后，要在不以任何方式更改完成的情况下检查完成的状态，请调用`completion_done()`，如果没有发布的完成尚未被服务员使用(意味着有服务员)，则返回 false，否则返回 true：

```c
bool completion_done(struct completion *done)
```

Both `try_wait_for_completion()` and `completion_done()` are safe to be called in IRQ or atomic context.

> 在 IRQ 或原子上下文中调用`try_wait_for_completion()`和`completion_done()`都是安全的。
