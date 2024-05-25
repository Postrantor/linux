---
tip: translate by baidu@2024-01-30 21:32:56
---
---
title: CPU load
---


Linux exports various bits of information via `/proc/stat` and `/proc/uptime` that userland tools, such as top(1), use to calculate the average time system spent in a particular state, for example:

> Linux通过“/proc/stat”和“/proc/uptime”导出各种信息，用户界面工具（如top（1））使用这些信息来计算系统在特定状态下花费的平均时间，例如：

    $ iostat
    Linux 2.6.18.3-exp (linmac)     02/20/2007

    avg-cpu:  %user   %nice %system %iowait  %steal   %idle
              10.01    0.00    2.92    5.44    0.00   81.63

    ...


Here the system thinks that over the default sampling period the system spent 10.01% of the time doing work in user space, 2.92% in the kernel, and was overall 81.63% of the time idle.

> 在这里，系统认为在默认采样期内，系统在用户空间中花费了10.01%的时间，在内核中花费了2.92%的时间，总体上空闲了81.63%的时间。


In most cases the `/proc/stat` information reflects the reality quite closely, however due to the nature of how/when the kernel collects this data sometimes it can not be trusted at all.

> 在大多数情况下，“/proc/stat”信息非常密切地反映了现实，然而，由于内核如何/何时收集这些数据的性质，有时它根本不可信。


So how is this information collected? Whenever timer interrupt is signalled the kernel looks what kind of task was running at this moment and increments the counter that corresponds to this tasks kind/state. The problem with this is that the system could have switched between various states multiple times between two timer interrupts yet the counter is incremented only for the last state.

> 那么这些信息是如何收集的呢？每当定时器中断发出信号时，内核就会查看此时运行的任务类型，并递增与此任务类型/状态对应的计数器。这方面的问题是，系统可能在两个定时器中断之间多次在不同状态之间切换，但计数器仅针对最后一个状态递增。

# Example


If we imagine the system with one task that periodically burns cycles in the following manner:

> 如果我们想象一个系统有一个任务，它以以下方式周期性地燃烧循环：

    time line between two timer interrupts
    |--------------------------------------|
    ^                                    ^
    |_ something begins working          |
                                         |_ something goes to sleep
                                        (only to be awaken quite soon)


In the above situation the system will be 0% loaded according to the `/proc/stat` (since the timer interrupt will always happen when the system is executing the idle handler), but in reality the load is closer to 99%.

> 在上述情况下，系统将根据“/proc/stat”加载0%（因为当系统执行空闲处理程序时总是会发生定时器中断），但实际上加载接近99%。


One can imagine many more situations where this behavior of the kernel will lead to quite erratic information inside `/proc/stat`:

> 可以想象，在更多的情况下，内核的这种行为将导致“/proc/stat”内部的信息非常不稳定：

    /* gcc -o hog smallhog.c */
    #include <time.h>
    #include <limits.h>
    #include <signal.h>
    #include <sys/time.h>
    #define HIST 10

    static volatile sig_atomic_t stop;

    static void sighandler(int signr)
    {
        (void) signr;
        stop = 1;
    }

    static unsigned long hog (unsigned long niters)
    {
        stop = 0;
        while (!stop && --niters);
        return niters;
    }

    int main (void)
    {
        int i;
        struct itimerval it = {
            .it_interval = { .tv_sec = 0, .tv_usec = 1 },
            .it_value    = { .tv_sec = 0, .tv_usec = 1 } };
        sigset_t set;
        unsigned long v[HIST];
        double tmp = 0.0;
        unsigned long n;
        signal(SIGALRM, &sighandler);
        setitimer(ITIMER_REAL, &it, NULL);

        hog (ULONG_MAX);
        for (i = 0; i < HIST; ++i) v[i] = ULONG_MAX - hog(ULONG_MAX);
        for (i = 0; i < HIST; ++i) tmp += v[i];
        tmp /= HIST;
        n = tmp - (tmp / 3.0);

        sigemptyset(&set);
        sigaddset(&set, SIGALRM);

        for (;;) {
            hog(n);
            sigwait(&set, &i);
        }
        return 0;
    }

# References

-   <https://lore.kernel.org/r/loom.20070212T063225-663@post.gmane.org>
-   Documentation/filesystems/proc.rst (1.8)

# Thanks

Con Kolivas, Pavel Machek
