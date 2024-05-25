---
tip: translate by baidu@2024-01-30 21:38:22
title: Reporting issues
---

# The short guide (aka TL;DR)


Are you facing a regression with vanilla kernels from the same stable or longterm series? One still supported? Then search the [LKML](https://lore.kernel.org/lkml/) and the [Linux stable mailing list](https://lore.kernel.org/stable/) archives for matching reports to join. If you don\'t find any, install [the latest release from that series](https://kernel.org/). If it still shows the issue, report it to the stable mailing list (<stable@vger.kernel.org>) and CC the regressions list (<regressions@lists.linux.dev>); ideally also CC the maintainer and the mailing list for the subsystem in question.

> 你是否面临着来自同一稳定或长期系列的香草仁的回归？一个仍然支持？然后搜索[LKML](https://lore.kernel.org/lkml/)和[Linux稳定邮件列表](https://lore.kernel.org/stable/)用于要加入的匹配报表的存档。如果找不到，请安装[该系列的最新版本](https://kernel.org/). 如果它仍然显示问题，请将其报告给稳定的邮件列表(<stable@vger.kernel.org>)并抄送回归列表(<regressions@lists.linux.dev>); 理想情况下，还抄送有问题的子系统的维护人员和邮件列表。


In all other cases try your best guess which kernel part might be causing the issue. Check the `MAINTAINERS <maintainers>`{.interpreted-text role="ref"} file for how its developers expect to be told about problems, which most of the time will be by email with a mailing list in CC. Check the destination\'s archives for matching reports; search the [LKML](https://lore.kernel.org/lkml/) and the web, too. If you don\'t find any to join, install [the latest mainline kernel](https://kernel.org/). If the issue is present there, send a report.

> 在所有其他情况下，请尽量猜测是哪个内核部分导致了问题。查看`MAINTAINERS<MAINTAINERS>`｛.explored text role=“ref”｝文件，了解其开发人员希望如何了解问题，大多数情况下，这些问题将通过电子邮件和CC中的邮件列表进行通知。查看目的地的档案以获取匹配的报告；搜索[LKML](https://lore.kernel.org/lkml/)还有网络。如果找不到要加入的，请安装[最新的主线内核](https://kernel.org/). 如果存在问题，请发送报告。


The issue was fixed there, but you would like to see it resolved in a still supported stable or longterm series as well? Then install its latest release. If it shows the problem, search for the change that fixed it in mainline and check if backporting is in the works or was discarded; if it\'s neither, ask those who handled the change for it.

> 这个问题在那里得到了解决，但你想看到它在一个仍然支持的稳定或长期系列中得到解决吗？然后安装其最新版本。如果它显示了问题，请在主线中搜索修复它的更改，并检查后台移植是否正在进行中或已被丢弃；如果两者都不是，请向负责更改的人员索取。


**General remarks**: When installing and testing a kernel as outlined above, ensure it\'s vanilla (IOW: not patched and not using add-on modules). Also make sure it\'s built and running in a healthy environment and not already tainted before the issue occurs.

> **一般备注**：在安装和测试如上所述的内核时，请确保它是普通的（IOW：未修补，也未使用附加模块）。还要确保它是在一个健康的环境中构建和运行的，并且在问题发生之前没有受到污染。


If you are facing multiple issues with the Linux kernel at once, report each separately. While writing your report, include all information relevant to the issue, like the kernel and the distro used. In case of a regression, CC the regressions mailing list (<regressions@lists.linux.dev>) to your report. Also try to pin-point the culprit with a bisection; if you succeed, include its commit-id and CC everyone in the sign-off-by chain.

> 如果您同时面临Linux内核的多个问题，请分别报告每个问题。在编写报告时，包括与问题相关的所有信息，如所使用的内核和发行版。如果是回归，请抄送回归邮件列表(<regressions@lists.linux.dev>)添加到您的报告中。还要试着用平分线来确定罪魁祸首；如果成功，请将其提交id和CC每个人都包含在签出链中。


Once the report is out, answer any questions that come up and help where you can. That includes keeping the ball rolling by occasionally retesting with newer releases and sending a status update afterwards.

> 报告发布后，回答出现的任何问题，并尽可能提供帮助。这包括通过偶尔重新测试新版本并在之后发送状态更新来保持进展。

# Step-by-step guide how to report issues to the kernel maintainers


The above TL;DR outlines roughly how to report issues to the Linux kernel developers. It might be all that\'s needed for people already familiar with reporting issues to Free/Libre & Open Source Software (FLOSS) projects. For everyone else there is this section. It is more detailed and uses a step-by-step approach. It still tries to be brief for readability and leaves out a lot of details; those are described below the step-by-step guide in a reference section, which explains each of the steps in more detail.

> 上述TL；DR大致概述了如何向Linux内核开发人员报告问题。这可能是那些已经熟悉向自由/自由和开源软件（FLOSS）项目报告问题的人所需要的全部。对于其他所有人，都有这个部分。它更详细，并使用逐步的方法。为了可读性，它仍然尽量简短，并省略了很多细节；以下是参考部分中的分步指南，其中更详细地解释了每个步骤。


Note: this section covers a few more aspects than the TL;DR and does things in a slightly different order. That\'s in your interest, to make sure you notice early if an issue that looks like a Linux kernel problem is actually caused by something else. These steps thus help to ensure the time you invest in this process won\'t feel wasted in the end:

> 注：本节涵盖了比TL更多的几个方面；DR，并以稍微不同的顺序做事。这符合您的兴趣，以确保您尽早注意到一个看起来像Linux内核问题的问题实际上是由其他原因引起的。因此，这些步骤有助于确保你在这个过程中投入的时间最终不会被浪费：

> -   Are you facing an issue with a Linux kernel a hardware or software vendor provided? Then in almost all cases you are better off to stop reading this document and reporting the issue to your vendor instead, unless you are willing to install the latest Linux version yourself. Be aware the latter will often be needed anyway to hunt down and fix issues.
> -   Perform a rough search for existing reports with your favorite internet search engine; additionally, check the archives of the [Linux Kernel Mailing List (LKML)](https://lore.kernel.org/lkml/). If you find matching reports, join the discussion instead of sending a new one.
> -   See if the issue you are dealing with qualifies as regression, security issue, or a really severe problem: those are \'issues of high priority\' that need special handling in some steps that are about to follow.
> -   Make sure it\'s not the kernel\'s surroundings that are causing the issue you face.
> -   Create a fresh backup and put system repair and restore tools at hand.
> -   Ensure your system does not enhance its kernels by building additional kernel modules on-the-fly, which solutions like DKMS might be doing locally without your knowledge.
> -   Check if your kernel was \'tainted\' when the issue occurred, as the event that made the kernel set this flag might be causing the issue you face.
> -   Write down coarsely how to reproduce the issue. If you deal with multiple issues at once, create separate notes for each of them and make sure they work independently on a freshly booted system. That\'s needed, as each issue needs to get reported to the kernel developers separately, unless they are strongly entangled.
> -   If you are facing a regression within a stable or longterm version line (say something broke when updating from 5.10.4 to 5.10.5), scroll down to \'Dealing with regressions within a stable and longterm kernel line\'.
> -   Locate the driver or kernel subsystem that seems to be causing the issue. Find out how and where its developers expect reports. Note: most of the time this won\'t be bugzilla.kernel.org, as issues typically need to be sent by mail to a maintainer and a public mailing list.
> -   Search the archives of the bug tracker or mailing list in question thoroughly for reports that might match your issue. If you find anything, join the discussion instead of sending a new report.

After these preparations you\'ll now enter the main part:

> -   Unless you are already running the latest \'mainline\' Linux kernel, better go and install it for the reporting process. Testing and reporting with the latest \'stable\' Linux can be an acceptable alternative in some situations; during the merge window that actually might be even the best approach, but in that development phase it can be an even better idea to suspend your efforts for a few days anyway. Whatever version you choose, ideally use a \'vanilla\' build. Ignoring these advices will dramatically increase the risk your report will be rejected or ignored.
> -   Ensure the kernel you just installed does not \'taint\' itself when running.
> -   Reproduce the issue with the kernel you just installed. If it doesn\'t show up there, scroll down to the instructions for issues only happening with stable and longterm kernels.
> -   Optimize your notes: try to find and write the most straightforward way to reproduce your issue. Make sure the end result has all the important details, and at the same time is easy to read and understand for others that hear about it for the first time. And if you learned something in this process, consider searching again for existing reports about the issue.
> -   If your failure involves a \'panic\', \'Oops\', \'warning\', or \'BUG\', consider decoding the kernel log to find the line of code that triggered the error.
> -   If your problem is a regression, try to narrow down when the issue was introduced as much as possible.
> -   Start to compile the report by writing a detailed description about the issue. Always mention a few things: the latest kernel version you installed for reproducing, the Linux Distribution used, and your notes on how to reproduce the issue. Ideally, make the kernel\'s build configuration (.config) and the output from `dmesg` available somewhere on the net and link to it. Include or upload all other information that might be relevant, like the output/screenshot of an Oops or the output from `lspci`. Once you wrote this main part, insert a normal length paragraph on top of it outlining the issue and the impact quickly. On top of this add one sentence that briefly describes the problem and gets people to read on. Now give the thing a descriptive title or subject that yet again is shorter. Then you\'re ready to send or file the report like the MAINTAINERS file told you, unless you are dealing with one of those \'issues of high priority\': they need special care which is explained in \'Special handling for high priority issues\' below.
> -   Wait for reactions and keep the thing rolling until you can accept the outcome in one way or the other. Thus react publicly and in a timely manner to any inquiries. Test proposed fixes. Do proactive testing: retest with at least every first release candidate (RC) of a new mainline version and report your results. Send friendly reminders if things stall. And try to help yourself, if you don\'t get any help or if it\'s unsatisfying.

## Reporting regressions within a stable and longterm kernel line


This subsection is for you, if you followed above process and got sent here at the point about regression within a stable or longterm kernel version line. You face one of those if something breaks when updating from 5.10.4 to 5.10.5 (a switch from 5.9.15 to 5.10.5 does not qualify). The developers want to fix such regressions as quickly as possible, hence there is a streamlined process to report them:

> 如果您遵循了上面的过程，并在关于稳定或长期内核版本行中的回归的点上被发送到这里，那么本小节就是为您准备的。如果在从5.10.4更新到5.10.5时出现故障（从5.9.15切换到5.10.5不符合条件），您将面临其中一种情况。开发人员希望尽快修复此类回归，因此有一个简化的报告流程：

> -   Check if the kernel developers still maintain the Linux kernel version line you care about: go to the [front page of kernel.org](https://kernel.org/) and make sure it mentions the latest release of the particular version line without an \'\[EOL\]\' tag.
> -   Check the archives of the [Linux stable mailing list](https://lore.kernel.org/stable/) for existing reports.
> -   Install the latest release from the particular version line as a vanilla kernel. Ensure this kernel is not tainted and still shows the problem, as the issue might have already been fixed there. If you first noticed the problem with a vendor kernel, check a vanilla build of the last version known to work performs fine as well.
> -   Send a short problem report to the Linux stable mailing list (<stable@vger.kernel.org>) and CC the Linux regressions mailing list (<regressions@lists.linux.dev>); if you suspect the cause in a particular subsystem, CC its maintainer and its mailing list. Roughly describe the issue and ideally explain how to reproduce it. Mention the first version that shows the problem and the last version that\'s working fine. Then wait for further instructions.

The reference section below explains each of these steps in more detail.

## Reporting issues only occurring in older kernel version lines


This subsection is for you, if you tried the latest mainline kernel as outlined above, but failed to reproduce your issue there; at the same time you want to see the issue fixed in a still supported stable or longterm series or vendor kernels regularly rebased on those. If that the case, follow these steps:

> 如果您尝试了上面概述的最新主线内核，但未能在那里重现您的问题，则本小节适用于您；同时，您希望在仍受支持的稳定或长期系列中解决该问题，或者供应商内核定期重新基于这些问题。如果是这种情况，请执行以下步骤：

> -   Prepare yourself for the possibility that going through the next few steps might not get the issue solved in older releases: the fix might be too big or risky to get backported there.
> -   Perform the first three steps in the section \"Dealing with regressions within a stable and longterm kernel line\" above.
> -   Search the Linux kernel version control system for the change that fixed the issue in mainline, as its commit message might tell you if the fix is scheduled for backporting already. If you don\'t find anything that way, search the appropriate mailing lists for posts that discuss such an issue or peer-review possible fixes; then check the discussions if the fix was deemed unsuitable for backporting. If backporting was not considered at all, join the newest discussion, asking if it\'s in the cards.
> -   One of the former steps should lead to a solution. If that doesn\'t work out, ask the maintainers for the subsystem that seems to be causing the issue for advice; CC the mailing list for the particular subsystem as well as the stable mailing list.

The reference section below explains each of these steps in more detail.

# Reference section: Reporting issues to the kernel maintainers


The detailed guides above outline all the major steps in brief fashion, which should be enough for most people. But sometimes there are situations where even experienced users might wonder how to actually do one of those steps. That\'s what this section is for, as it will provide a lot more details on each of the above steps. Consider this as reference documentation: it\'s possible to read it from top to bottom. But it\'s mainly meant to skim over and a place to look up details how to actually perform those steps.

> 上面的详细指南简要概述了所有主要步骤，这对大多数人来说应该足够了。但有时，即使是有经验的用户也会想知道如何实际执行其中一个步骤。这就是本节的目的，因为它将提供关于上述每个步骤的更多细节。将其视为参考文档：可以从上到下阅读。但它主要是为了浏览和查找如何实际执行这些步骤的细节。

A few words of general advice before digging into the details:

> -   The Linux kernel developers are well aware this process is complicated and demands more than other FLOSS projects. We\'d love to make it simpler. But that would require work in various places as well as some infrastructure, which would need constant maintenance; nobody has stepped up to do that work, so that\'s just how things are for now.
> -   A warranty or support contract with some vendor doesn\'t entitle you to request fixes from developers in the upstream Linux kernel community: such contracts are completely outside the scope of the Linux kernel, its development community, and this document. That\'s why you can\'t demand anything such a contract guarantees in this context, not even if the developer handling the issue works for the vendor in question. If you want to claim your rights, use the vendor\'s support channel instead. When doing so, you might want to mention you\'d like to see the issue fixed in the upstream Linux kernel; motivate them by saying it\'s the only way to ensure the fix in the end will get incorporated in all Linux distributions.
> -   If you never reported an issue to a FLOSS project before you should consider reading [How to Report Bugs Effectively](https://www.chiark.greenend.org.uk/~sgtatham/bugs.html), [How To Ask Questions The Smart Way](http://www.catb.org/esr/faqs/smart-questions.html), and [How to ask good questions](https://jvns.ca/blog/good-questions/).


With that off the table, find below the details on how to properly report issues to the Linux kernel developers.

> 在这之后，可以在下面找到关于如何向Linux内核开发人员正确报告问题的详细信息。

## Make sure you\'re using the upstream Linux kernel

> *Are you facing an issue with a Linux kernel a hardware or software vendor provided? Then in almost all cases you are better off to stop reading this document and reporting the issue to your vendor instead, unless you are willing to install the latest Linux version yourself. Be aware the latter will often be needed anyway to hunt down and fix issues.*


Like most programmers, Linux kernel developers don\'t like to spend time dealing with reports for issues that don\'t even happen with their current code. It\'s just a waste everybody\'s time, especially yours. Unfortunately such situations easily happen when it comes to the kernel and often leads to frustration on both sides. That\'s because almost all Linux-based kernels pre-installed on devices (Computers, Laptops, Smartphones, Routers, ...) and most shipped by Linux distributors are quite distant from the official Linux kernel as distributed by kernel.org: these kernels from these vendors are often ancient from the point of Linux development or heavily modified, often both.

> 和大多数程序员一样，Linux内核开发人员不喜欢花时间处理当前代码中甚至没有发生的问题的报告。这只是浪费每个人的时间，尤其是你的时间。不幸的是，当涉及到内核时，这种情况很容易发生，并且经常导致双方都感到沮丧。这是因为几乎所有预装在设备（计算机、笔记本电脑、智能手机、路由器等）上的基于Linux的内核，以及大多数由Linux分销商提供的内核，都与kernel.org分发的官方Linux内核相距甚远：这些供应商的这些内核从Linux开发的角度来看往往是古老的，或者经过了大量修改，通常两者都有。


Most of these vendor kernels are quite unsuitable for reporting issues to the Linux kernel developers: an issue you face with one of them might have been fixed by the Linux kernel developers months or years ago already; additionally, the modifications and enhancements by the vendor might be causing the issue you face, even if they look small or totally unrelated. That\'s why you should report issues with these kernels to the vendor. Its developers should look into the report and, in case it turns out to be an upstream issue, fix it directly upstream or forward the report there. In practice that often does not work out or might not what you want. You thus might want to consider circumventing the vendor by installing the very latest Linux kernel core yourself. If that\'s an option for you move ahead in this process, as a later step in this guide will explain how to do that once it rules out other potential causes for your issue.

> 这些供应商内核中的大多数都不适合向Linux内核开发人员报告问题：其中一个内核所面临的问题可能在几个月或几年前就已经由Linux内核开发员解决了；此外，供应商的修改和增强可能会导致您面临的问题，即使它们看起来很小或完全无关。这就是为什么您应该向供应商报告这些内核的问题。它的开发人员应该调查该报告，如果它是上游问题，则直接向上游解决或将报告转发到那里。在实践中，这往往不起作用，或者可能不是你想要的。因此，您可能需要考虑通过自己安装最新的Linux内核内核来绕过供应商。如果这是你在这个过程中前进的一个选项，那么本指南的下一步将解释一旦排除了导致你问题的其他潜在原因，如何做到这一点。


Note, the previous paragraph is starting with the word \'most\', as sometimes developers in fact are willing to handle reports about issues occurring with vendor kernels. If they do in the end highly depends on the developers and the issue in question. Your chances are quite good if the distributor applied only small modifications to a kernel based on a recent Linux version; that for example often holds true for the mainline kernels shipped by Debian GNU/Linux Sid or Fedora Rawhide. Some developers will also accept reports about issues with kernels from distributions shipping the latest stable kernel, as long as its only slightly modified; that for example is often the case for Arch Linux, regular Fedora releases, and openSUSE Tumbleweed. But keep in mind, you better want to use a mainline Linux and avoid using a stable kernel for this process, as outlined in the section \'Install a fresh kernel for testing\' in more detail.

> 请注意，前一段是以单词“st”开头的，因为有时开发人员实际上愿意处理有关供应商内核问题的报告。他们最终是否这样做在很大程度上取决于开发人员和问题所在。如果发行商只对基于最新Linux版本的内核进行了小的修改，那么您的机会非常大；例如，Debian GNU/Linux Sid或Fedora Rawhide提供的主流内核通常也是如此。一些开发人员也会接受来自发布最新稳定内核的发行版的关于内核问题的报告，只要它只是稍微修改过；例如，Arch Linux、常规Fedora发行版和openSUSE汤博乐的情况经常如此。但请记住，您最好希望使用主流Linux，并避免在此过程中使用稳定的内核，如“安装新的内核进行测试”一节中所述。


Obviously you are free to ignore all this advice and report problems with an old or heavily modified vendor kernel to the upstream Linux developers. But note, those often get rejected or ignored, so consider yourself warned. But it\'s still better than not reporting the issue at all: sometimes such reports directly or indirectly will help to get the issue fixed over time.

> 显然，您可以忽略所有这些建议，并向上游Linux开发人员报告旧的或经过大量修改的供应商内核的问题。但请注意，这些往往会被拒绝或忽视，所以要提醒自己。但这总比根本不报告问题要好：有时这种直接或间接的报告会有助于随着时间的推移解决问题。

## Search for existing reports, first run

> *Perform a rough search for existing reports with your favorite internet search engine; additionally, check the archives of the Linux Kernel Mailing List (LKML). If you find matching reports, join the discussion instead of sending a new one.*


Reporting an issue that someone else already brought forward is often a waste of time for everyone involved, especially you as the reporter. So it\'s in your own interest to thoroughly check if somebody reported the issue already. At this step of the process it\'s okay to just perform a rough search: a later step will tell you to perform a more detailed search once you know where your issue needs to be reported to. Nevertheless, do not hurry with this step of the reporting process, it can save you time and trouble.

> 报道别人已经提出的问题对所有相关人员来说往往是浪费时间，尤其是作为记者的你。因此，彻底检查是否有人已经报告了这个问题符合你自己的利益。在这个过程的这一步，可以只进行粗略的搜索：稍后的步骤会告诉你，一旦你知道你的问题需要报告给哪里，就要进行更详细的搜索。不过，不要急于进行报告过程的这个步骤，它可以为你节省时间和麻烦。


Simply search the internet with your favorite search engine first. Afterwards, search the [Linux Kernel Mailing List (LKML) archives](https://lore.kernel.org/lkml/).

> 只需先用你喜欢的搜索引擎在互联网上搜索。之后，搜索[Linux Kernel Mailing List（LKML）archives](https://lore.kernel.org/lkml/).


If you get flooded with results consider telling your search engine to limit search timeframe to the past month or year. And wherever you search, make sure to use good search terms; vary them a few times, too. While doing so try to look at the issue from the perspective of someone else: that will help you to come up with other words to use as search terms. Also make sure not to use too many search terms at once. Remember to search with and without information like the name of the kernel driver or the name of the affected hardware component. But its exact brand name (say \'ASUS Red Devil Radeon RX 5700 XT Gaming OC\') often is not much helpful, as it is too specific. Instead try search terms like the model line (Radeon 5700 or Radeon 5000) and the code name of the main chip (\'Navi\' or \'Navi10\') with and without its manufacturer (\'AMD\').

> 如果你收到了大量的搜索结果，可以考虑告诉你的搜索引擎将搜索时间限制在过去的一个月或一年。无论你在哪里搜索，都要确保使用好的搜索词；也可以将它们更改几次。在这样做的时候，试着从别人的角度来看待这个问题：这将帮助你想出其他单词作为搜索词。还要确保不要同时使用过多的搜索词。记住搜索时要有或没有诸如内核驱动程序名称或受影响硬件组件名称之类的信息。但它的确切品牌名称（比如“SUS Red Devil Radeon RX 5700 XT Gaming OC\”）往往没有多大帮助，因为它太具体了。相反，尝试搜索术语，如型号线（Radeon 5700或Radeon 5000）和主芯片的代码名称（“Navi”或“Navi10”）（有制造商和无制造商）（“AMD”）。


In case you find an existing report about your issue, join the discussion, as you might be able to provide valuable additional information. That can be important even when a fix is prepared or in its final stages already, as developers might look for people that can provide additional information or test a proposed fix. Jump to the section \'Duties after the report went out\' for details on how to get properly involved.

> 如果您发现有关您的问题的现有报告，请加入讨论，因为您可能能够提供有价值的其他信息。即使修复程序已经准备好或处于最后阶段，这一点也很重要，因为开发人员可能会寻找能够提供额外信息或测试拟议修复程序的人员。跳到“报告发布后的实用程序”一节，了解如何正确参与的详细信息。


Note, searching [bugzilla.kernel.org](https://bugzilla.kernel.org/) might also be a good idea, as that might provide valuable insights or turn up matching reports. If you find the latter, just keep in mind: most subsystems expect reports in different places, as described below in the section \"Check where you need to report your issue\". The developers that should take care of the issue thus might not even be aware of the bugzilla ticket. Hence, check the ticket if the issue already got reported as outlined in this document and if not consider doing so.

> 注意，搜索[bugzilla.kernel.org/](https://bugzilla.kernel.org/)这可能也是一个好主意，因为这可能会提供有价值的见解或找到匹配的报告。如果您发现了后者，请记住：大多数子系统都希望在不同的地方进行报告，如下面“检查需要报告问题的地方”一节所述。因此，应该处理这个问题的开发人员可能甚至不知道bugzilla票证。因此，如果问题已经如本文件所述报告，请检查票证，如果没有，请考虑这样做。

## Issue of high priority?

> *See if the issue you are dealing with qualifies as regression, security issue, or a really severe problem: those are \'issues of high priority\' that need special handling in some steps that are about to follow.*


Linus Torvalds and the leading Linux kernel developers want to see some issues fixed as soon as possible, hence there are \'issues of high priority\' that get handled slightly differently in the reporting process. Three type of cases qualify: regressions, security issues, and really severe problems.

> Linus Torvalds和领先的Linux内核开发人员希望尽快解决一些问题，因此在报告过程中，有些“高优先级问题”的处理方式略有不同。有三种情况：倒退、安全问题和真正严重的问题。


You deal with a regression if some application or practical use case running fine with one Linux kernel works worse or not at all with a newer version compiled using a similar configuration. The document Documentation/admin-guide/reporting-regressions.rst explains this in more detail. It also provides a good deal of other information about regressions you might want to be aware of; it for example explains how to add your issue to the list of tracked regressions, to ensure it won\'t fall through the cracks.

> 如果某个应用程序或实际用例在一个Linux内核中运行良好，或者在使用类似配置编译的新版本中运行不好，则需要处理回归。文档Documentation/admin-guide/reporting-requestions.rst对此进行了更详细的解释。它还提供了许多关于回归的其他信息，你可能想了解这些信息；例如，它解释了如何将您的问题添加到跟踪回归列表中，以确保它不会从裂缝中掉出来。


What qualifies as security issue is left to your judgment. Consider reading Documentation/process/security-bugs.rst before proceeding, as it provides additional details how to best handle security issues.

> 什么是安全问题，由你们自己判断。在继续之前，请考虑阅读Documentation/process/security-bugs.rst，因为它提供了如何最好地处理安全问题的更多详细信息。


An issue is a \'really severe problem\' when something totally unacceptably bad happens. That\'s for example the case when a Linux kernel corrupts the data it\'s handling or damages hardware it\'s running on. You\'re also dealing with a severe issue when the kernel suddenly stops working with an error message (\'kernel panic\') or without any farewell note at all. Note: do not confuse a \'panic\' (a fatal error where the kernel stop itself) with a \'Oops\' (a recoverable error), as the kernel remains running after the latter.

> 当一些完全不可接受的糟糕事情发生时，问题就是“非常严重的问题”。例如，当Linux内核破坏了它处理的数据或损坏了它运行的硬件时，就会出现这种情况。当内核突然停止工作并出现错误消息（“kernel panic”）或根本没有任何告别语时，你也会遇到严重的问题。注意：不要将“panic”（内核自行停止的致命错误）与“Oops”（可恢复的错误）混淆，因为内核在后者之后仍在运行。

## Ensure a healthy environment

> *Make sure it\'s not the kernel\'s surroundings that are causing the issue you face.*


Problems that look a lot like a kernel issue are sometimes caused by build or runtime environment. It\'s hard to rule out that problem completely, but you should minimize it:

> 看起来很像内核问题的问题有时是由构建或运行时环境引起的。很难完全排除这个问题，但你应该尽量减少它：

> -   Use proven tools when building your kernel, as bugs in the compiler or the binutils can cause the resulting kernel to misbehave.
> -   Ensure your computer components run within their design specifications; that\'s especially important for the main processor, the main memory, and the motherboard. Therefore, stop undervolting or overclocking when facing a potential kernel issue.
> -   Try to make sure it\'s not faulty hardware that is causing your issue. Bad main memory for example can result in a multitude of issues that will manifest itself in problems looking like kernel issues.
> -   If you\'re dealing with a filesystem issue, you might want to check the file system in question with `fsck`, as it might be damaged in a way that leads to unexpected kernel behavior.
> -   When dealing with a regression, make sure it\'s not something else that changed in parallel to updating the kernel. The problem for example might be caused by other software that was updated at the same time. It can also happen that a hardware component coincidentally just broke when you rebooted into a new kernel for the first time. Updating the systems BIOS or changing something in the BIOS Setup can also lead to problems that on look a lot like a kernel regression.

## Prepare for emergencies

> *Create a fresh backup and put system repair and restore tools at hand.*


Reminder, you are dealing with computers, which sometimes do unexpected things, especially if you fiddle with crucial parts like the kernel of its operating system. That\'s what you are about to do in this process. Thus, make sure to create a fresh backup; also ensure you have all tools at hand to repair or reinstall the operating system as well as everything you need to restore the backup.

> 提醒一下，你要处理的是计算机，它们有时会做一些意想不到的事情，尤其是当你摆弄操作系统内核等关键部分时。这就是你在这个过程中要做的。因此，请确保创建新的备份；还要确保您手头有修复或重新安装操作系统的所有工具，以及恢复备份所需的一切工具。

## Make sure your kernel doesn\'t get enhanced

> *Ensure your system does not enhance its kernels by building additional kernel modules on-the-fly, which solutions like DKMS might be doing locally without your knowledge.*


The risk your issue report gets ignored or rejected dramatically increases if your kernel gets enhanced in any way. That\'s why you should remove or disable mechanisms like akmods and DKMS: those build add-on kernel modules automatically, for example when you install a new Linux kernel or boot it for the first time. Also remove any modules they might have installed. Then reboot before proceeding.

> 如果内核以任何方式得到增强，问题报告被忽略或拒绝的风险都会显著增加。这就是为什么你应该删除或禁用像akmods和DKMS这样的机制：这些机制会自动构建附加内核模块，例如当你安装一个新的Linux内核或第一次启动它时。还要删除他们可能安装的任何模块。然后重新启动，然后继续。


Note, you might not be aware that your system is using one of these solutions: they often get set up silently when you install Nvidia\'s proprietary graphics driver, VirtualBox, or other software that requires a some support from a module not part of the Linux kernel. That why your might need to uninstall the packages with such software to get rid of any 3rd party kernel module.

> 请注意，您可能没有意识到您的系统正在使用其中一种解决方案：当您安装Nvidia的专有图形驱动程序VirtualBox或其他需要非Linux内核模块支持的软件时，它们通常会自动设置。这就是为什么您可能需要卸载带有此类软件的软件包，以摆脱任何第三方内核模块。

## Check \'taint\' flag

> *Check if your kernel was \'tainted\' when the issue occurred, as the event that made the kernel set this flag might be causing the issue you face.*


The kernel marks itself with a \'taint\' flag when something happens that might lead to follow-up errors that look totally unrelated. The issue you face might be such an error if your kernel is tainted. That\'s why it\'s in your interest to rule this out early before investing more time into this process. This is the only reason why this step is here, as this process later will tell you to install the latest mainline kernel; you will need to check the taint flag again then, as that\'s when it matters because it\'s the kernel the report will focus on.

> 当发生可能导致看起来完全无关的后续错误的事情时，内核会用“aint”标志来标记自己。如果您的内核受到污染，那么您所面临的问题可能就是这样一个错误。这就是为什么在投入更多时间之前尽早排除这种情况符合你的利益。这是这个步骤出现的唯一原因，因为这个过程稍后会告诉您安装最新的主线内核；然后，您需要再次检查污点标志，因为这很重要，因为它是报告将关注的内核。


On a running system is easy to check if the kernel tainted itself: if `cat /proc/sys/kernel/tainted` returns \'0\' then the kernel is not tainted and everything is fine. Checking that file is impossible in some situations; that\'s why the kernel also mentions the taint status when it reports an internal problem (a \'kernel bug\'), a recoverable error (a \'kernel Oops\') or a non-recoverable error before halting operation (a \'kernel panic\'). Look near the top of the error messages printed when one of these occurs and search for a line starting with \'CPU:\'. It should end with \'Not tainted\' if the kernel was not tainted when it noticed the problem; it was tainted if you see \'Tainted:\' followed by a few spaces and some letters.

> 在运行的系统上，很容易检查内核是否受到污染：如果“cat/proc/sys/kernel/spoted”返回“0\”，则内核没有受到污染，一切都很好。在某些情况下无法检查该文件；这就是为什么内核在停止操作之前报告内部问题（“内核错误”）、可恢复错误（“内核出错”）或不可恢复错误时（“内核死机”）也会提到污点状态。当出现其中一个错误时，请查看打印的错误消息的顶部附近，并搜索以“PU:\”开头的行。如果内核在注意到问题时没有被污染，那么它应该以“未被污染”结尾；如果你看到“ainted：”后面跟着几个空格和一些字母，它就会被污染。


If your kernel is tainted, study Documentation/admin-guide/tainted-kernels.rst to find out why. Try to eliminate the reason. Often it\'s caused by one these three things:

> 如果您的内核受到污染，请研究Documentation/admin-guide/tainted-kernels.rst以找出原因。尽量消除原因。通常它是由以下三件事之一引起的：

> 1.  A recoverable error (a \'kernel Oops\') occurred and the kernel tainted itself, as the kernel knows it might misbehave in strange ways after that point. In that case check your kernel or system log and look for a section that starts with this:
>
>         Oops: 0000 [#1] SMP
>
>     That\'s the first Oops since boot-up, as the \'#1\' between the brackets shows. Every Oops and any other problem that happens after that point might be a follow-up problem to that first Oops, even if both look totally unrelated. Rule this out by getting rid of the cause for the first Oops and reproducing the issue afterwards. Sometimes simply restarting will be enough, sometimes a change to the configuration followed by a reboot can eliminate the Oops. But don\'t invest too much time into this at this point of the process, as the cause for the Oops might already be fixed in the newer Linux kernel version you are going to install later in this process.
>
> 2.  Your system uses a software that installs its own kernel modules, for example Nvidia\'s proprietary graphics driver or VirtualBox. The kernel taints itself when it loads such module from external sources (even if they are Open Source): they sometimes cause errors in unrelated kernel areas and thus might be causing the issue you face. You therefore have to prevent those modules from loading when you want to report an issue to the Linux kernel developers. Most of the time the easiest way to do that is: temporarily uninstall such software including any modules they might have installed. Afterwards reboot.
>
> 3.  The kernel also taints itself when it\'s loading a module that resides in the staging tree of the Linux kernel source. That\'s a special area for code (mostly drivers) that does not yet fulfill the normal Linux kernel quality standards. When you report an issue with such a module it\'s obviously okay if the kernel is tainted; just make sure the module in question is the only reason for the taint. If the issue happens in an unrelated area reboot and temporarily block the module from being loaded by specifying `foo.blacklist=1` as kernel parameter (replace \'foo\' with the name of the module in question).

## Document how to reproduce issue

> *Write down coarsely how to reproduce the issue. If you deal with multiple issues at once, create separate notes for each of them and make sure they work independently on a freshly booted system. That\'s needed, as each issue needs to get reported to the kernel developers separately, unless they are strongly entangled.*


If you deal with multiple issues at once, you\'ll have to report each of them separately, as they might be handled by different developers. Describing various issues in one report also makes it quite difficult for others to tear it apart. Hence, only combine issues in one report if they are very strongly entangled.

> 如果您同时处理多个问题，则必须分别报告每个问题，因为它们可能由不同的开发人员处理。在一份报告中描述各种问题也使其他人很难将其拆开。因此，只有当问题非常纠缠在一起时，才能将其合并到一份报告中。


Additionally, during the reporting process you will have to test if the issue happens with other kernel versions. Therefore, it will make your work easier if you know exactly how to reproduce an issue quickly on a freshly booted system.

> 此外，在报告过程中，您必须测试其他内核版本是否发生了问题。因此，如果您确切地知道如何在新启动的系统上快速复制问题，这将使您的工作更容易。


Note: it\'s often fruitless to report issues that only happened once, as they might be caused by a bit flip due to cosmic radiation. That\'s why you should try to rule that out by reproducing the issue before going further. Feel free to ignore this advice if you are experienced enough to tell a one-time error due to faulty hardware apart from a kernel issue that rarely happens and thus is hard to reproduce.

> 注意：报告只发生过一次的问题往往是徒劳的，因为它们可能是由宇宙辐射引起的一点翻转引起的。这就是为什么你应该在进一步讨论之前，通过复制这个问题来排除这种可能性。如果您有足够的经验来判断由硬件故障引起的一次性错误，而内核问题很少发生，因此很难再现，请随时忽略此建议。

## Regression in stable or longterm kernel?

> *If you are facing a regression within a stable or longterm version line (say something broke when updating from 5.10.4 to 5.10.5), scroll down to \'Dealing with regressions within a stable and longterm kernel line\'.*


Regression within a stable and longterm kernel version line are something the Linux developers want to fix badly, as such issues are even more unwanted than regression in the main development branch, as they can quickly affect a lot of people. The developers thus want to learn about such issues as quickly as possible, hence there is a streamlined process to report them. Note, regressions with newer kernel version line (say something broke when switching from 5.9.15 to 5.10.5) do not qualify.

> Linux开发人员非常希望解决稳定和长期内核版本线中的回归问题，因为这些问题比主要开发分支中的回归更不受欢迎，因为它们会迅速影响很多人。因此，开发人员希望尽快了解这些问题，因此有一个简化的报告流程。请注意，使用较新内核版本行的回归（比如从5.9.15切换到5.10.5时出现问题）不合格。

## Check where you need to report your issue

> *Locate the driver or kernel subsystem that seems to be causing the issue. Find out how and where its developers expect reports. Note: most of the time this won\'t be bugzilla.kernel.org, as issues typically need to be sent by mail to a maintainer and a public mailing list.*


It\'s crucial to send your report to the right people, as the Linux kernel is a big project and most of its developers are only familiar with a small subset of it. Quite a few programmers for example only care for just one driver, for example one for a WiFi chip; its developer likely will only have small or no knowledge about the internals of remote or unrelated \"subsystems\", like the TCP stack, the PCIe/PCI subsystem, memory management or file systems.

> 将报告发送给合适的人是至关重要的，因为Linux内核是一个大项目，大多数开发人员只熟悉其中的一小部分。例如，相当多的程序员只关心一个驱动程序，例如一个用于WiFi芯片的驱动程序；其开发人员可能对远程或不相关的“子系统”的内部知识知之甚少或一无所知，如TCP堆栈、PCIe/PCI子系统、内存管理或文件系统。


Problem is: the Linux kernel lacks a central bug tracker where you can simply file your issue and make it reach the developers that need to know about it. That\'s why you have to find the right place and way to report issues yourself. You can do that with the help of a script (see below), but it mainly targets kernel developers and experts. For everybody else the MAINTAINERS file is the better place.

> 问题是：Linux内核缺乏一个中央错误跟踪器，你可以简单地将问题提交给需要了解它的开发人员。这就是为什么你必须找到正确的地方和方法来自己报告问题。您可以在脚本的帮助下做到这一点（见下文），但它主要针对内核开发人员和专家。对于其他人来说，MAINTAINERS文件是一个更好的地方。

### How to read the MAINTAINERS file


To illustrate how to use the `MAINTAINERS <maintainers>`{.interpreted-text role="ref"} file, lets assume the WiFi in your Laptop suddenly misbehaves after updating the kernel. In that case it\'s likely an issue in the WiFi driver. Obviously it could also be some code it builds upon, but unless you suspect something like that stick to the driver. If it\'s really something else, the driver\'s developers will get the right people involved.

> 为了说明如何使用`MAINTAINERS<MAINTAINERS>`｛.explored text role=“ref”｝文件，假设您的笔记本电脑中的WiFi在更新内核后突然出现故障。在这种情况下，很可能是WiFi驱动程序的问题。显然，它也可能是它所构建的一些代码，但除非你怀疑有类似的事情，否则请坚持使用驱动程序。如果真的是别的事情，驱动程序的开发人员会让合适的人参与进来。


Sadly, there is no way to check which code is driving a particular hardware component that is both universal and easy.

> 遗憾的是，没有办法检查哪种代码驱动了一个通用且简单的特定硬件组件。


In case of a problem with the WiFi driver you for example might want to look at the output of `lspci -k`, as it lists devices on the PCI/PCIe bus and the kernel module driving it:

> 如果WiFi驱动程序出现问题，例如，您可能需要查看“lspci-k”的输出，因为它列出了PCI/PCIe总线上的设备和驱动它的内核模块：

    [user@something ~]$ lspci -k
    [...]
    3a:00.0 Network controller: Qualcomm Atheros QCA6174 802.11ac Wireless Network Adapter (rev 32)
      Subsystem: Bigfoot Networks, Inc. Device 1535
      Kernel driver in use: ath10k_pci
      Kernel modules: ath10k_pci
    [...]


But this approach won\'t work if your WiFi chip is connected over USB or some other internal bus. In those cases you might want to check your WiFi manager or the output of `ip link`. Look for the name of the problematic network interface, which might be something like \'wlp58s0\'. This name can be used like this to find the module driving it:

> 但如果你的WiFi芯片是通过USB或其他内部总线连接的，这种方法就不起作用了。在这种情况下，您可能需要检查WiFi管理器或“ip链接”的输出。查找有问题的网络接口的名称，它可能类似于“lp58s0”。这个名称可以像这样用来查找驱动它的模块：

    [user@something ~]$ realpath --relative-to=/sys/module/ /sys/class/net/wlp58s0/device/driver/module
    ath10k_pci


In case tricks like these don\'t bring you any further, try to search the internet on how to narrow down the driver or subsystem in question. And if you are unsure which it is: just try your best guess, somebody will help you if you guessed poorly.

> 如果这样的技巧不会让你走得更远，试着在互联网上搜索如何缩小有问题的驱动程序或子系统。如果你不确定是哪一个：试着猜吧，如果你猜得不好，有人会帮你的。


Once you know the driver or subsystem, you want to search for it in the MAINTAINERS file. In the case of \'ath10k_pci\' you won\'t find anything, as the name is too specific. Sometimes you will need to search on the net for help; but before doing so, try a somewhat shorted or modified name when searching the MAINTAINERS file, as then you might find something like this:

> 一旦您知道了驱动程序或子系统，就需要在MAINTAINERS文件中搜索它。在“th10k_pci”的情况下，您将找不到任何内容，因为名称过于具体。有时你需要在网上搜索帮助；但在这样做之前，在搜索MAINTAINERS文件时，请尝试使用一个稍微短一些或修改过的名称，因为这样您可能会发现以下内容：

    QUALCOMM ATHEROS ATH10K WIRELESS DRIVER
    Mail:          A. Some Human <shuman@example.com>
    Mailing list:  ath10k@lists.infradead.org
    Status:        Supported
    Web-page:      https://wireless.wiki.kernel.org/en/users/Drivers/ath10k
    SCM:           git git://git.kernel.org/pub/scm/linux/kernel/git/kvalo/ath.git
    Files:         drivers/net/wireless/ath/ath10k/


Note: the line description will be abbreviations, if you read the plain MAINTAINERS file found in the root of the Linux source tree. \'Mail:\' for example will be \'M:\', \'Mailing list:\' will be \'L\', and \'Status:\' will be \'S:\'. A section near the top of the file explains these and other abbreviations.

> 注意：如果您阅读了Linux源代码树根目录中的普通MAINTAINERS文件，则行描述将是缩写。\'邮件：“例如”将是“M：”，“邮件列表：”将是”“L”“，“状态：”将为”“S：”“。文件顶部附近的一个部分解释了这些缩写和其他缩写。


First look at the line \'Status\'. Ideally it should be \'Supported\' or \'Maintained\'. If it states \'Obsolete\' then you are using some outdated approach that was replaced by a newer solution you need to switch to. Sometimes the code only has someone who provides \'Odd Fixes\' when feeling motivated. And with \'Orphan\' you are totally out of luck, as nobody takes care of the code anymore. That only leaves these options: arrange yourself to live with the issue, fix it yourself, or find a programmer somewhere willing to fix it.

> 首先看“状态”一行。理想情况下，它应该是“支持”或“维护”。如果它说“过时”，那么你使用的是一些过时的方法，而这些方法被你需要切换到的新解决方案所取代。有时，代码中只有在感到有动力时才有人提供“奇怪的修复”。有了“孤儿”，你就完全没有运气了，因为再也没有人照顾代码了。这只剩下这些选择：安排自己处理这个问题，自己解决它，或者找一个愿意解决它的程序员。


After checking the status, look for a line starting with \'bugs:\': it will tell you where to find a subsystem specific bug tracker to file your issue. The example above does not have such a line. That is the case for most sections, as Linux kernel development is completely driven by mail. Very few subsystems use a bug tracker, and only some of those rely on bugzilla.kernel.org.

> 检查状态后，查找以“bugs:\”开头的一行：它将告诉您在哪里可以找到子系统特定的错误跟踪器来提交您的问题。上面的例子没有这样的线条。大多数部分都是这样，因为Linux内核开发完全由邮件驱动。很少有子系统使用bug跟踪器，只有部分子系统依赖bugzilla.kernel.org。


In this and many other cases you thus have to look for lines starting with \'Mail:\' instead. Those mention the name and the email addresses for the maintainers of the particular code. Also look for a line starting with \'Mailing list:\', which tells you the public mailing list where the code is developed. Your report later needs to go by mail to those addresses. Additionally, for all issue reports sent by email, make sure to add the Linux Kernel Mailing List (LKML) \<<linux-kernel@vger.kernel.org>\> to CC. Don\'t omit either of the mailing lists when sending your issue report by mail later! Maintainers are busy people and might leave some work for other developers on the subsystem specific list; and LKML is important to have one place where all issue reports can be found.

> 因此，在这种情况和许多其他情况下，您必须查找以“邮件：”开头的行。其中提到了特定代码维护者的姓名和电子邮件地址。还要查找以“Mailing-list:\”开头的行，该行告诉代码开发的公共邮件列表。您的报告稍后需要通过邮件发送到这些地址。此外，对于所有通过电子邮件发送的问题报告，请确保添加Linux内核邮件列表（LKML）\<<linux-kernel@vger.kernel.org>\>抄送。以后通过邮件发送问题报告时，不要忽略任何一个邮件列表！维护人员是忙碌的人，可能会将一些工作留给子系统特定列表中的其他开发人员；LKML重要的是要有一个可以找到所有问题报告的地方。

### Finding the maintainers with the help of a script


For people that have the Linux sources at hand there is a second option to find the proper place to report: the script \'scripts/get_maintainer.pl\' which tries to find all people to contact. It queries the MAINTAINERS file and needs to be called with a path to the source code in question. For drivers compiled as module if often can be found with a command like this:

> 对于手头有Linux源代码的人来说，还有第二种选择可以找到合适的报告地点：脚本“scripts/get_maintainer.pl”，它试图找到所有要联系的人。它查询MAINTAINERS文件，并且需要使用有问题的源代码的路径进行调用。对于编译为模块的驱动程序，如果经常可以使用以下命令找到：

    $ modinfo ath10k_pci | grep filename | sed 's!/lib/modules/.*/kernel/!!; s!filename:!!; s!\.ko\(\|\.xz\)!!'
    drivers/net/wireless/ath/ath10k/ath10k_pci.ko

Pass parts of this to the script:

    $ ./scripts/get_maintainer.pl -f drivers/net/wireless/ath/ath10k*
    Some Human <shuman@example.com> (supporter:QUALCOMM ATHEROS ATH10K WIRELESS DRIVER)
    Another S. Human <asomehuman@example.com> (maintainer:NETWORKING DRIVERS)
    ath10k@lists.infradead.org (open list:QUALCOMM ATHEROS ATH10K WIRELESS DRIVER)
    linux-wireless@vger.kernel.org (open list:NETWORKING DRIVERS (WIRELESS))
    netdev@vger.kernel.org (open list:NETWORKING DRIVERS)
    linux-kernel@vger.kernel.org (open list)


Don\'t sent your report to all of them. Send it to the maintainers, which the script calls \"supporter:\"; additionally CC the most specific mailing list for the code as well as the Linux Kernel Mailing List (LKML). In this case you thus would need to send the report to \'Some Human \<<shuman@example.com>\>\' with \'<ath10k@lists.infradead.org>\' and \'<linux-kernel@vger.kernel.org>\' in CC.

> 不要把你的报告发给所有人。将其发送给维护人员，脚本称之为“supporter:”；另外抄送代码的最具体的邮件列表以及Linux内核邮件列表（LKML）。在这种情况下，您需要将报告发送给“某个人”\<<shuman@example.com具有ath10k@lists.infradead.org和linux-kernel@vger.kernel.org>\'在CC中。


Note: in case you cloned the Linux sources with git you might want to call `get_maintainer.pl` a second time with `--git`. The script then will look at the commit history to find which people recently worked on the code in question, as they might be able to help. But use these results with care, as it can easily send you in a wrong direction. That for example happens quickly in areas rarely changed (like old or unmaintained drivers): sometimes such code is modified during tree-wide cleanups by developers that do not care about the particular driver at all.

> 注意：如果您使用git克隆了Linux源代码，您可能需要使用“--git”再次调用“get_maintainer.pl”。然后，脚本将查看提交历史记录，以找出哪些人最近处理了有问题的代码，因为他们可能会提供帮助。但是要小心使用这些结果，因为它很容易把你推向错误的方向。例如，在很少更改的领域（如旧的或未维护的驱动程序），这种情况会很快发生：有时这样的代码会在树范围内的清理过程中被根本不关心特定驱动程序的开发人员修改。

## Search for existing reports, second run

> *Search the archives of the bug tracker or mailing list in question thoroughly for reports that might match your issue. If you find anything, join the discussion instead of sending a new report.*


As mentioned earlier already: reporting an issue that someone else already brought forward is often a waste of time for everyone involved, especially you as the reporter. That\'s why you should search for existing report again, now that you know where they need to be reported to. If it\'s mailing list, you will often find its archives on [lore.kernel.org](https://lore.kernel.org/).

> 如前所述：报道别人已经提出的问题对所有相关人员来说往往是浪费时间，尤其是作为记者的你。这就是为什么你应该再次搜索现有的报告，现在你知道了它们需要报告到哪里。如果它是邮件列表，你经常会在[lore.kernel.org]上找到它的档案(https://lore.kernel.org/).


But some list are hosted in different places. That for example is the case for the ath10k WiFi driver used as example in the previous step. But you\'ll often find the archives for these lists easily on the net. Searching for \'archive <ath10k@lists.infradead.org>\' for example will lead you to the [Info page for the ath10k mailing list](https://lists.infradead.org/mailman/listinfo/ath10k), which at the top links to its [list archives](https://lists.infradead.org/pipermail/ath10k/). Sadly this and quite a few other lists miss a way to search the archives. In those cases use a regular internet search engine and add something like \'site:lists.infradead.org/pipermail/ath10k/\' to your search terms, which limits the results to the archives at that URL.

> 但有些列表是在不同的地方托管的。例如，前面步骤中使用的ath10k WiFi驱动程序就是这样。但是你经常可以在网上很容易地找到这些清单的档案。正在搜索“存档”<ath10k@lists.infradead.org>\'例如，将引导您进入[ath10k邮件列表的信息页面](https://lists.infradead.org/mailman/listinfo/ath10k)，在顶部链接到其[列表档案](https://lists.infradead.org/pipermail/ath10k/). 遗憾的是，这一列表和其他相当多的列表都错过了搜索档案的方法。在这种情况下，使用常规的互联网搜索引擎，并在搜索词中添加类似“site:lists.infrardead.org/pipermail/ath10k/\”的内容，这将结果限制在该URL的档案中。


It\'s also wise to check the internet, LKML and maybe bugzilla.kernel.org again at this point. If your report needs to be filed in a bug tracker, you may want to check the mailing list archives for the subsystem as well, as someone might have reported it only there.

> 在这一点上，再次查看互联网、LKML以及bugzilla.kernel.org也是明智的。如果您的报告需要在错误跟踪器中存档，您可能还需要检查子系统的邮件列表档案，因为有人可能只在那里报告了它。


For details how to search and what to do if you find matching reports see \"Search for existing reports, first run\" above.

> 有关如何搜索以及如果找到匹配的报告该怎么办的详细信息，请参阅上面的“搜索现有报告，首先运行”。


Do not hurry with this step of the reporting process: spending 30 to 60 minutes or even more time can save you and others quite a lot of time and trouble.

> 不要急于进行报告过程的这一步：花30到60分钟甚至更长的时间可以为你和其他人节省大量的时间和麻烦。

## Install a fresh kernel for testing

> *Unless you are already running the latest \'mainline\' Linux kernel, better go and install it for the reporting process. Testing and reporting with the latest \'stable\' Linux can be an acceptable alternative in some situations; during the merge window that actually might be even the best approach, but in that development phase it can be an even better idea to suspend your efforts for a few days anyway. Whatever version you choose, ideally use a \'vanilla\' built. Ignoring these advices will dramatically increase the risk your report will be rejected or ignored.*


As mentioned in the detailed explanation for the first step already: Like most programmers, Linux kernel developers don\'t like to spend time dealing with reports for issues that don\'t even happen with the current code. It\'s just a waste everybody\'s time, especially yours. That\'s why it\'s in everybody\'s interest that you confirm the issue still exists with the latest upstream code before reporting it. You are free to ignore this advice, but as outlined earlier: doing so dramatically increases the risk that your issue report might get rejected or simply ignored.

> 正如第一步的详细解释中已经提到的：像大多数程序员一样，Linux内核开发人员不喜欢花时间处理当前代码中甚至没有发生的问题的报告。这只是浪费每个人的时间，尤其是你的时间。这就是为什么在报告之前用最新的上游代码确认问题仍然存在符合每个人的利益。你可以忽略这一建议，但如前所述：这样做会大大增加问题报告被拒绝或被忽视的风险。

In the scope of the kernel \"latest upstream\" normally means:

> -   Install a mainline kernel; the latest stable kernel can be an option, but most of the time is better avoided. Longterm kernels (sometimes called \'LTS kernels\') are unsuitable at this point of the process. The next subsection explains all of this in more detail.
> -   The over next subsection describes way to obtain and install such a kernel. It also outlines that using a pre-compiled kernel are fine, but better are vanilla, which means: it was built using Linux sources taken straight [from kernel.org](https://kernel.org/) and not modified or enhanced in any way.

### Choosing the right version for testing


Head over to [kernel.org](https://kernel.org/) to find out which version you want to use for testing. Ignore the big yellow button that says \'Latest release\' and look a little lower at the table. At its top you\'ll see a line starting with mainline, which most of the time will point to a pre-release with a version number like \'5.8-rc2\'. If that\'s the case, you\'ll want to use this mainline kernel for testing, as that where all fixes have to be applied first. Do not let that \'rc\' scare you, these \'development kernels\' are pretty reliable --- and you made a backup, as you were instructed above, didn\'t you?

> 前往[kernel.org](https://kernel.org/)以找出要用于测试的版本。忽略上面写着“最晚释放”的黄色大按钮，在桌子上看得低一点。在它的顶部，你会看到一条以主线开始的线，大多数时候它会指向版本号为“5.8-rc2”的预发布版本。如果是这样的话，你会想使用这个主线内核进行测试，因为所有修复都必须首先应用。不要让“c”吓到你，这些“开发内核”非常可靠——而且你按照上面的指示做了备份，不是吗？


In about two out of every nine to ten weeks, mainline might point you to a proper release with a version number like \'5.7\'. If that happens, consider suspending the reporting process until the first pre-release of the next version (5.8-rc1) shows up on kernel.org. That\'s because the Linux development cycle then is in its two-week long \'merge window\'. The bulk of the changes and all intrusive ones get merged for the next release during this time. It\'s a bit more risky to use mainline during this period. Kernel developers are also often quite busy then and might have no spare time to deal with issue reports. It\'s also quite possible that one of the many changes applied during the merge window fixes the issue you face; that\'s why you soon would have to retest with a newer kernel version anyway, as outlined below in the section \'Duties after the report went out\'.

> 在大约每九到十周中的两周内，mainline可能会向您指出一个版本号为“5.7”的正确版本。如果发生这种情况，请考虑暂停报告过程，直到下一个版本（5.8-rc1）的第一个预发布出现在kernel.org上。这是因为Linux开发周期处于两周长的“合并窗口”。在这段时间内，大部分更改和所有侵入性更改都会合并到下一个版本中。在这段时间里使用主线有点冒险。内核开发人员通常也很忙，可能没有多余的时间来处理问题报告。在合并窗口中应用的众多更改之一也很可能解决了您面临的问题；这就是为什么您很快就必须使用较新的内核版本重新测试的原因，如下文“报告发布后的实用程序”一节所述。


That\'s why it might make sense to wait till the merge window is over. But don\'t to that if you\'re dealing with something that shouldn\'t wait. In that case consider obtaining the latest mainline kernel via git (see below) or use the latest stable version offered on kernel.org. Using that is also acceptable in case mainline for some reason does currently not work for you. An in general: using it for reproducing the issue is also better than not reporting it issue at all.

> 这就是为什么等待合并窗口结束可能是有意义的。但是，如果你正在处理一些不应该等待的事情，就不要这么做。在这种情况下，可以考虑通过git获得最新的主线内核（见下文）或使用kernel.org上提供的最新稳定版本。如果主线由于某种原因目前不适用，也可以使用它。一般来说：使用它来复制问题也比根本不报告问题要好。


Better avoid using the latest stable kernel outside merge windows, as all fixes must be applied to mainline first. That\'s why checking the latest mainline kernel is so important: any issue you want to see fixed in older version lines needs to be fixed in mainline first before it can get backported, which can take a few days or weeks. Another reason: the fix you hope for might be too hard or risky for backporting; reporting the issue again hence is unlikely to change anything.

> 最好避免在合并窗口之外使用最新的稳定内核，因为所有修复都必须首先应用于主线。这就是为什么检查最新的主线内核如此重要：任何你想在旧版本中解决的问题都需要先在主线中解决，然后才能进行后移植，这可能需要几天或几周的时间。另一个原因是：您所希望的修复对于后移植来说可能太难或风险太大；因此，再次报告该问题不太可能改变任何事情。


These aspects are also why longterm kernels (sometimes called \"LTS kernels\") are unsuitable for this part of the reporting process: they are to distant from the current code. Hence go and test mainline first and follow the process further: if the issue doesn\'t occur with mainline it will guide you how to get it fixed in older version lines, if that\'s in the cards for the fix in question.

> 这些方面也是为什么长期内核（有时称为“LTS内核”）不适合报告过程的这一部分：它们与当前代码相距太远。因此，首先测试主线，并进一步遵循流程：如果主线没有出现问题，它将指导您如何在旧版本行中修复它，如果这是有问题的修复方法的话。

### How to obtain a fresh Linux kernel


**Using a pre-compiled kernel**: This is often the quickest, easiest, and safest way for testing --- especially is you are unfamiliar with the Linux kernel. The problem: most of those shipped by distributors or add-on repositories are build from modified Linux sources. They are thus not vanilla and therefore often unsuitable for testing and issue reporting: the changes might cause the issue you face or influence it somehow.

> **使用预编译的内核**：这通常是最快、最简单、最安全的测试方法——尤其是当您不熟悉Linux内核时。问题是：大多数由分销商或附加存储库提供的都是从修改过的Linux源构建的。因此，它们不是香草，因此通常不适合测试和问题报告：这些变化可能会导致您面临的问题或以某种方式影响它。


But you are in luck if you are using a popular Linux distribution: for quite a few of them you\'ll find repositories on the net that contain packages with the latest mainline or stable Linux built as vanilla kernel. It\'s totally okay to use these, just make sure from the repository\'s description they are vanilla or at least close to it. Additionally ensure the packages contain the latest versions as offered on kernel.org. The packages are likely unsuitable if they are older than a week, as new mainline and stable kernels typically get released at least once a week.

> 但如果你使用的是一个流行的Linux发行版，那你就很幸运了：对于其中相当多的发行版，你会在网上找到包含最新主线或稳定Linux构建为香草内核的软件包的存储库。使用这些是完全可以的，只需从存储库的描述中确保它们是普通的，或者至少接近它。此外，确保包包含kernel.org上提供的最新版本。如果包的版本超过一周，则可能不合适，因为新的主线和稳定内核通常每周至少发布一次。


Please note that you might need to build your own kernel manually later: that\'s sometimes needed for debugging or testing fixes, as described later in this document. Also be aware that pre-compiled kernels might lack debug symbols that are needed to decode messages the kernel prints when a panic, Oops, warning, or BUG occurs; if you plan to decode those, you might be better off compiling a kernel yourself (see the end of this subsection and the section titled \'Decode failure messages\' for details).

> 请注意，您稍后可能需要手动构建自己的内核：这有时是调试或测试修复程序所必需的，如本文档稍后所述。还要注意，预编译的内核可能缺少调试符号，这些符号是在出现死机、错误、警告或BUG时解码内核打印的消息所需的；如果你计划解码这些，你最好自己编译一个内核（有关详细信息，请参阅本小节的末尾和标题为“解码失败消息”的部分）。


**Using git**: Developers and experienced Linux users familiar with git are often best served by obtaining the latest Linux kernel sources straight from the [official development repository on kernel.org](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/). Those are likely a bit ahead of the latest mainline pre-release. Don\'t worry about it: they are as reliable as a proper pre-release, unless the kernel\'s development cycle is currently in the middle of a merge window. But even then they are quite reliable.

> **使用git**：熟悉git的开发人员和经验丰富的Linux用户通常可以直接从[kernel.org上的官方开发库]获得最新的Linux内核源代码，从而获得最佳服务(https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/). 这些可能比最新的主线预发布版本提前了一点。不必担心：除非内核的开发周期目前正处于合并窗口的中间，否则它们与适当的预发行版一样可靠。但即便如此，它们也相当可靠。


**Conventional**: People unfamiliar with git are often best served by downloading the sources as tarball from [kernel.org](https://kernel.org/).

> **常规**：不熟悉git的人通常最好从[kernel.org]下载源代码作为tarball(https://kernel.org/).


How to actually build a kernel is not described here, as many websites explain the necessary steps already. If you are new to it, consider following one of those how-to\'s that suggest to use `make localmodconfig`, as that tries to pick up the configuration of your current kernel and then tries to adjust it somewhat for your system. That does not make the resulting kernel any better, but quicker to compile.

> 这里没有描述如何实际构建内核，因为许多网站已经解释了必要的步骤。如果你是新手，可以考虑遵循其中一个建议使用“makelocalmodconfig”的操作方法，因为这会尝试获取当前内核的配置，然后尝试为你的系统进行一些调整。这并没有使生成的内核变得更好，但编译起来更快。


Note: If you are dealing with a panic, Oops, warning, or BUG from the kernel, please try to enable CONFIG_KALLSYMS when configuring your kernel. Additionally, enable CONFIG_DEBUG_KERNEL and CONFIG_DEBUG_INFO, too; the latter is the relevant one of those two, but can only be reached if you enable the former. Be aware CONFIG_DEBUG_INFO increases the storage space required to build a kernel by quite a bit. But that\'s worth it, as these options will allow you later to pinpoint the exact line of code that triggers your issue. The section \'Decode failure messages\' below explains this in more detail.

> 注意：如果您正在处理来自内核的死机、错误、警告或BUG，请在配置内核时尝试启用CONFIG_KALLSYMS。此外，还应启用CONFIG_DEBUG_KERNEL和CONFIG_DEBUG_INFO；后者是这两者中相关的一个，但只有启用前者才能达到。请注意，CONFIG_DEBUG_INFO将构建内核所需的存储空间增加了不少。但这是值得的，因为这些选项将使您稍后能够精确定位引发问题的代码行。下面的“代码故障消息”一节对此进行了更详细的解释。


But keep in mind: Always keep a record of the issue encountered in case it is hard to reproduce. Sending an undecoded report is better than not reporting the issue at all.

> 但请记住：始终记录遇到的问题，以防难以复制。发送未编码的报告总比根本不报告问题要好。

## Check \'taint\' flag

> *Ensure the kernel you just installed does not \'taint\' itself when running.*


As outlined above in more detail already: the kernel sets a \'taint\' flag when something happens that can lead to follow-up errors that look totally unrelated. That\'s why you need to check if the kernel you just installed does not set this flag. And if it does, you in almost all the cases needs to eliminate the reason for it before you reporting issues that occur with it. See the section above for details how to do that.

> 正如上面已经详细描述的那样：当发生可能导致看起来完全无关的后续错误时，内核会设置“aint”标志。这就是为什么您需要检查刚刚安装的内核是否没有设置此标志。如果是这样，在几乎所有情况下，您都需要在报告发生的问题之前消除原因。有关如何做到这一点的详细信息，请参阅上面的部分。

## Reproduce issue with the fresh kernel

> *Reproduce the issue with the kernel you just installed. If it doesn\'t show up there, scroll down to the instructions for issues only happening with stable and longterm kernels.*


Check if the issue occurs with the fresh Linux kernel version you just installed. If it was fixed there already, consider sticking with this version line and abandoning your plan to report the issue. But keep in mind that other users might still be plagued by it, as long as it\'s not fixed in either stable and longterm version from kernel.org (and thus vendor kernels derived from those). If you prefer to use one of those or just want to help their users, head over to the section \"Details about reporting issues only occurring in older kernel version lines\" below.

> 检查您刚刚安装的新Linux内核版本是否出现问题。如果它已经在那里修复了，可以考虑坚持这个版本行，放弃报告问题的计划。但请记住，其他用户可能仍然受到它的困扰，只要它没有在kernel.org的稳定和长期版本中得到修复（因此也没有从中派生出供应商内核）。如果您更喜欢使用其中一个，或者只是想帮助他们的用户，请转到下面的“有关仅在旧内核版本行中发生的报告问题的详细信息”部分。

## Optimize description to reproduce issue

> *Optimize your notes: try to find and write the most straightforward way to reproduce your issue. Make sure the end result has all the important details, and at the same time is easy to read and understand for others that hear about it for the first time. And if you learned something in this process, consider searching again for existing reports about the issue.*


An unnecessarily complex report will make it hard for others to understand your report. Thus try to find a reproducer that\'s straight forward to describe and thus easy to understand in written form. Include all important details, but at the same time try to keep it as short as possible.

> 一份不必要的复杂报告会让其他人很难理解你的报告。因此，试着找到一个直接描述、易于书面理解的复制品。包括所有重要的细节，但同时尽量简短。


In this in the previous steps you likely have learned a thing or two about the issue you face. Use this knowledge and search again for existing reports instead you can join.

> 在前面的步骤中，你可能已经了解了你所面临的问题。使用这些知识，然后重新搜索现有的报告，您可以加入。

## Decode failure messages

> *If your failure involves a \'panic\', \'Oops\', \'warning\', or \'BUG\', consider decoding the kernel log to find the line of code that triggered the error.*


When the kernel detects an internal problem, it will log some information about the executed code. This makes it possible to pinpoint the exact line in the source code that triggered the issue and shows how it was called. But that only works if you enabled CONFIG_DEBUG_INFO and CONFIG_KALLSYMS when configuring your kernel. If you did so, consider to decode the information from the kernel\'s log. That will make it a lot easier to understand what lead to the \'panic\', \'Oops\', \'warning\', or \'BUG\', which increases the chances that someone can provide a fix.

> 当内核检测到内部问题时，它会记录一些关于执行代码的信息。这使得可以精确定位引发该问题的源代码中的确切行，并显示它是如何调用的。但只有在配置内核时启用CONFIG_DEBUG_INFO和CONFIG_KALLSYMS时，这才有效。如果您这样做了，请考虑解码内核日志中的信息。这将使人们更容易理解是什么导致了“痛苦”、“糟糕”、“学习”或“BUG”，这增加了有人提供解决方案的机会。


Decoding can be done with a script you find in the Linux source tree. If you are running a kernel you compiled yourself earlier, call it like this:

> 解码可以使用您在Linux源代码树中找到的脚本来完成。如果您运行的是之前自己编译的内核，请这样调用它：

    [user@something ~]$ sudo dmesg | ./linux-5.10.5/scripts/decode_stacktrace.sh ./linux-5.10.5/vmlinux


If you are running a packaged vanilla kernel, you will likely have to install the corresponding packages with debug symbols. Then call the script (which you might need to get from the Linux sources if your distro does not package it) like this:

> 如果您运行的是打包的vanilla内核，则可能需要安装带有调试符号的相应包。然后调用脚本（如果您的发行版没有对其进行打包，则可能需要从Linux源代码中获取），如下所示：

    [user@something ~]$ sudo dmesg | ./linux-5.10.5/scripts/decode_stacktrace.sh \
     /usr/lib/debug/lib/modules/5.10.10-4.1.x86_64/vmlinux /usr/src/kernels/5.10.10-4.1.x86_64/


The script will work on log lines like the following, which show the address of the code the kernel was executing when the error occurred:

> 该脚本将在如下日志行上工作，这些日志行显示错误发生时内核正在执行的代码的地址：

    [   68.387301] RIP: 0010:test_module_init+0x5/0xffa [test_module]

Once decoded, these lines will look like this:

    [   68.387301] RIP: 0010:test_module_init (/home/username/linux-5.10.5/test-module/test-module.c:16) test_module


In this case the executed code was built from the file \'\~/linux-5.10.5/test-module/test-module.c\' and the error occurred by the instructions found in line \'16\'.

> 在这种情况下，执行的代码是从文件\'\~/linux-510.5/test-module/test-moodule.c\'构建的，错误发生在第\'16\'行的指令中。


The script will similarly decode the addresses mentioned in the section starting with \'Call trace\', which show the path to the function where the problem occurred. Additionally, the script will show the assembler output for the code section the kernel was executing.

> 脚本将类似地解码以“调用跟踪”开头的部分中提到的地址，这些地址显示了出现问题的函数的路径。此外，该脚本将显示内核正在执行的代码段的汇编程序输出。


Note, if you can\'t get this to work, simply skip this step and mention the reason for it in the report. If you\'re lucky, it might not be needed. And if it is, someone might help you to get things going. Also be aware this is just one of several ways to decode kernel stack traces. Sometimes different steps will be required to retrieve the relevant details. Don\'t worry about that, if that\'s needed in your case, developers will tell you what to do.

> 请注意，如果您无法做到这一点，只需跳过此步骤并在报告中提及原因即可。如果你运气好的话，可能就不需要它了。如果是的话，有人可能会帮你把事情做好。还要注意，这只是解码内核堆栈跟踪的几种方法之一。有时需要采取不同的步骤来检索相关的详细信息。不要担心，如果您的情况需要，开发人员会告诉您该怎么做。

## Special care for regressions

> *If your problem is a regression, try to narrow down when the issue was introduced as much as possible.*


Linux lead developer Linus Torvalds insists that the Linux kernel never worsens, that\'s why he deems regressions as unacceptable and wants to see them fixed quickly. That\'s why changes that introduced a regression are often promptly reverted if the issue they cause can\'t get solved quickly any other way. Reporting a regression is thus a bit like playing a kind of trump card to get something quickly fixed. But for that to happen the change that\'s causing the regression needs to be known. Normally it\'s up to the reporter to track down the culprit, as maintainers often won\'t have the time or setup at hand to reproduce it themselves.

> Linux首席开发人员Linus Torvalds坚持认为Linux内核永远不会恶化，这就是为什么他认为倒退是不可接受的，并希望看到它们迅速修复。这就是为什么引入回归的更改通常会在它们引起的问题无法以任何其他方式快速解决的情况下立即恢复。因此，报告回归有点像打一张王牌来快速解决问题。但要做到这一点，需要知道导致回归的变化。通常情况下，这取决于记者来追踪罪魁祸首，因为维护人员通常没有时间或设置来自己复制它。


To find the change there is a process called \'bisection\' which the document Documentation/admin-guide/bug-bisect.rst describes in detail. That process will often require you to build about ten to twenty kernel images, trying to reproduce the issue with each of them before building the next. Yes, that takes some time, but don\'t worry, it works a lot quicker than most people assume. Thanks to a \'binary search\' this will lead you to the one commit in the source code management system that\'s causing the regression. Once you find it, search the net for the subject of the change, its commit id and the shortened commit id (the first 12 characters of the commit id). This will lead you to existing reports about it, if there are any.

> 要找到更改，有一个名为“执行”的过程，文档Documentation/admin guide/bug-bisect.rst对此进行了详细描述。这个过程通常需要构建大约10到20个内核映像，在构建下一个之前，尝试用它们中的每一个来重现问题。是的，这需要一些时间，但别担心，它比大多数人想象的要快得多。多亏了“二进制搜索”，这将引导您在源代码管理系统中找到导致回归的一个提交。找到它后，在网上搜索更改的主题、提交id和缩短的提交id（提交id的前12个字符）。这将引导您找到有关它的现有报告（如果有的话）。


Note, a bisection needs a bit of know-how, which not everyone has, and quite a bit of effort, which not everyone is willing to invest. Nevertheless, it\'s highly recommended performing a bisection yourself. If you really can\'t or don\'t want to go down that route at least find out which mainline kernel introduced the regression. If something for example breaks when switching from 5.5.15 to 5.8.4, then try at least all the mainline releases in that area (5.6, 5.7 and 5.8) to check when it first showed up. Unless you\'re trying to find a regression in a stable or longterm kernel, avoid testing versions which number has three sections (5.6.12, 5.7.8), as that makes the outcome hard to interpret, which might render your testing useless. Once you found the major version which introduced the regression, feel free to move on in the reporting process. But keep in mind: it depends on the issue at hand if the developers will be able to help without knowing the culprit. Sometimes they might recognize from the report want went wrong and can fix it; other times they will be unable to help unless you perform a bisection.

> 注意，平分需要一点专业知识，而不是每个人都有，也需要相当多的努力，而不是所有人都愿意投资。尽管如此，强烈建议你自己做一个平分。如果你真的不能或不想走这条路，至少要找出是哪个主线内核引入了回归。例如，如果从5.5.15切换到5.8.4时出现故障，则至少尝试该区域（5.6、5.7和5.8）中的所有主线释放，以检查其首次出现的时间。除非你试图在稳定或长期的内核中找到回归，否则避免测试版本，因为版本号有三个部分（5.6.12，5.7.8），因为这会使结果难以解释，这可能会使你的测试变得无用。一旦你找到了引入回归的主要版本，就可以在报告过程中继续前进了。但请记住：这取决于手头的问题，开发人员是否能够在不知道罪魁祸首的情况下提供帮助。有时，他们可能会从报告中认识到want出了问题，并可以修复它；其他时候，除非你进行平分，否则他们将无法提供帮助。


When dealing with regressions make sure the issue you face is really caused by the kernel and not by something else, as outlined above already.

> 在处理回归时，请确保您面临的问题确实是由内核引起的，而不是由其他原因引起的，如上所述。


In the whole process keep in mind: an issue only qualifies as regression if the older and the newer kernel got built with a similar configuration. This can be achieved by using `make olddefconfig`, as explained in more detail by Documentation/admin-guide/reporting-regressions.rst; that document also provides a good deal of other information about regressions you might want to be aware of.

> 在整个过程中，请记住：只有当旧内核和新内核使用类似的配置构建时，问题才能被视为回归。这可以通过使用“makeolddefconfig”来实现，如Documentation/admin-guide/reporting-registrations.rst中更详细的解释；该文档还提供了许多您可能想要了解的有关回归的其他信息。

## Write and send the report

> *Start to compile the report by writing a detailed description about the issue. Always mention a few things: the latest kernel version you installed for reproducing, the Linux Distribution used, and your notes on how to reproduce the issue. Ideally, make the kernel\'s build configuration (.config) and the output from \`\`dmesg\`\` available somewhere on the net and link to it. Include or upload all other information that might be relevant, like the output/screenshot of an Oops or the output from \`\`lspci\`\`. Once you wrote this main part, insert a normal length paragraph on top of it outlining the issue and the impact quickly. On top of this add one sentence that briefly describes the problem and gets people to read on. Now give the thing a descriptive title or subject that yet again is shorter. Then you\'re ready to send or file the report like the MAINTAINERS file told you, unless you are dealing with one of those \'issues of high priority\': they need special care which is explained in \'Special handling for high priority issues\' below.*


Now that you have prepared everything it\'s time to write your report. How to do that is partly explained by the three documents linked to in the preface above. That\'s why this text will only mention a few of the essentials as well as things specific to the Linux kernel.

> 既然你已经准备好了一切，是时候写报告了。如何做到这一点，上文序言中提到的三份文件在一定程度上解释了这一点。这就是为什么本文只提到一些要点以及Linux内核特有的东西。


There is one thing that fits both categories: the most crucial parts of your report are the title/subject, the first sentence, and the first paragraph. Developers often get quite a lot of mail. They thus often just take a few seconds to skim a mail before deciding to move on or look closer. Thus: the better the top section of your report, the higher are the chances that someone will look into it and help you. And that is why you should ignore them for now and write the detailed report first. ;-)

> 有一件事适合这两类：报告中最关键的部分是标题/主题、第一句话和第一段。开发人员经常收到很多邮件。因此，他们通常只需要几秒钟的时间浏览邮件，就可以决定继续前进或仔细查看。因此：你报告的顶部越好，有人调查并帮助你的机会就越高。这就是为什么你现在应该忽略它们，先写下详细的报告

### Things each report should mention


Describe in detail how your issue happens with the fresh vanilla kernel you installed. Try to include the step-by-step instructions you wrote and optimized earlier that outline how you and ideally others can reproduce the issue; in those rare cases where that\'s impossible try to describe what you did to trigger it.

> 详细描述您安装的新鲜香草仁是如何出现问题的。试着包括你之前编写和优化的分步说明，概述你和理想情况下的其他人如何重现问题；在极少数不可能的情况下，试着描述一下你是怎么触发它的。


Also include all the relevant information others might need to understand the issue and its environment. What\'s actually needed depends a lot on the issue, but there are some things you should include always:

> 还包括其他人可能需要了解问题及其环境的所有相关信息。实际需要什么在很大程度上取决于问题，但有一些东西你应该始终包括在内：

> -   the output from `cat /proc/version`, which contains the Linux kernel version number and the compiler it was built with.
> -   the Linux distribution the machine is running (`hostnamectl | grep "Operating System"`)
> -   the architecture of the CPU and the operating system (`uname -mi`)
> -   if you are dealing with a regression and performed a bisection, mention the subject and the commit-id of the change that is causing it.


In a lot of cases it\'s also wise to make two more things available to those that read your report:

> 在很多情况下，明智的做法是让阅读报告的人多了解两件事：

> -   the configuration used for building your Linux kernel (the \'.config\' file)
> -   the kernel\'s messages that you get from `dmesg` written to a file. Make sure that it starts with a line like \'Linux version 5.8-1 (<foobar@example.com>) (gcc (GCC) 10.2.1, GNU ld version 2.34) #1 SMP Mon Aug 3 14:54:37 UTC 2020\' If it\'s missing, then important messages from the first boot phase already got discarded. In this case instead consider using `journalctl -b 0 -k`; alternatively you can also reboot, reproduce the issue and call `dmesg` right afterwards.


These two files are big, that\'s why it\'s a bad idea to put them directly into your report. If you are filing the issue in a bug tracker then attach them to the ticket. If you report the issue by mail do not attach them, as that makes the mail too large; instead do one of these things:

> 这两个文件很大，这就是为什么直接将它们放入报告中是个坏主意。如果您正在错误跟踪器中提交问题，请将它们附加到票证上。如果你通过邮件报告问题，不要附上它们，因为这会使邮件太大；相反，请执行以下操作之一：

> -   Upload the files somewhere public (your website, a public file paste service, a ticket created just for this purpose on [bugzilla.kernel.org](https://bugzilla.kernel.org/), \...) and include a link to them in your report. Ideally use something where the files stay available for years, as they could be useful to someone many years from now; this for example can happen if five or ten years from now a developer works on some code that was changed just to fix your issue.
> -   Put the files aside and mention you will send them later in individual replies to your own mail. Just remember to actually do that once the report went out. ;-)

### Things that might be wise to provide


Depending on the issue you might need to add more background data. Here are a few suggestions what often is good to provide:

> 根据问题的不同，您可能需要添加更多的背景数据。以下是一些通常很好的建议：

> -   If you are dealing with a \'warning\', an \'OOPS\' or a \'panic\' from the kernel, include it. If you can\'t copy\'n\'paste it, try to capture a netconsole trace or at least take a picture of the screen.
> -   If the issue might be related to your computer hardware, mention what kind of system you use. If you for example have problems with your graphics card, mention its manufacturer, the card\'s model, and what chip is uses. If it\'s a laptop mention its name, but try to make sure it\'s meaningful. \'Dell XPS 13\' for example is not, because it might be the one from 2012; that one looks not that different from the one sold today, but apart from that the two have nothing in common. Hence, in such cases add the exact model number, which for example are \'9380\' or \'7390\' for XPS 13 models introduced during 2019. Names like \'Lenovo Thinkpad T590\' are also somewhat ambiguous: there are variants of this laptop with and without a dedicated graphics chip, so try to find the exact model name or specify the main components.
> -   Mention the relevant software in use. If you have problems with loading modules, you want to mention the versions of kmod, systemd, and udev in use. If one of the DRM drivers misbehaves, you want to state the versions of libdrm and Mesa; also specify your Wayland compositor or the X-Server and its driver. If you have a filesystem issue, mention the version of corresponding filesystem utilities (e2fsprogs, btrfs-progs, xfsprogs, \...).
> -   Gather additional information from the kernel that might be of interest. The output from `lspci -nn` will for example help others to identify what hardware you use. If you have a problem with hardware you even might want to make the output from `sudo lspci -vvv` available, as that provides insights how the components were configured. For some issues it might be good to include the contents of files like `/proc/cpuinfo`, `/proc/ioports`, `/proc/iomem`, `/proc/modules`, or `/proc/scsi/scsi`. Some subsystem also offer tools to collect relevant information. One such tool is `alsa-info.sh` [which the audio/sound subsystem developers provide](https://www.alsa-project.org/wiki/AlsaInfo).


Those examples should give your some ideas of what data might be wise to attach, but you have to think yourself what will be helpful for others to know. Don\'t worry too much about forgetting something, as developers will ask for additional details they need. But making everything important available from the start increases the chance someone will take a closer look.

> 这些例子应该让你对哪些数据可能是明智的附加有一些想法，但你必须思考哪些数据对其他人有帮助。不要太担心忘记什么，因为开发人员会询问他们需要的更多细节。但是，从一开始就让所有重要的东西都可用，增加了有人仔细观察的机会。

### The important part: the head of your report


Now that you have the detailed part of the report prepared let\'s get to the most important section: the first few sentences. Thus go to the top, add something like \'The detailed description:\' before the part you just wrote and insert two newlines at the top. Now write one normal length paragraph that describes the issue roughly. Leave out all boring details and focus on the crucial parts readers need to know to understand what this is all about; if you think this bug affects a lot of users, mention this to get people interested.

> 现在你已经准备好了报告的详细部分，让我们进入最重要的部分：前几句。因此，转到顶部，在您刚写的部分之前添加类似“详细描述：”的内容，并在顶部插入两行换行符。现在写一个正常长度的段落，大致描述这个问题。去掉所有无聊的细节，专注于读者需要了解的关键部分，以了解这一切；如果你认为这个bug影响了很多用户，请提及它以引起人们的兴趣。


Once you did that insert two more lines at the top and write a one sentence summary that explains quickly what the report is about. After that you have to get even more abstract and write an even shorter subject/title for the report.

> 一旦你做到了，在顶部再插入两行，并写一句话的总结，快速解释报告的内容。之后，你必须变得更加抽象，并为报告写一个更短的主题/标题。


Now that you have written this part take some time to optimize it, as it is the most important parts of your report: a lot of people will only read this before they decide if reading the rest is time well spent.

> 既然你已经写好了这一部分，需要一些时间来优化它，因为它是你报告中最重要的部分：很多人只有在决定阅读其余部分是否花得好之前才会阅读这一部分。


Now send or file the report like the `MAINTAINERS <maintainers>`{.interpreted-text role="ref"} file told you, unless it\'s one of those \'issues of high priority\' outlined earlier: in that case please read the next subsection first before sending the report on its way.

> 现在，按照`MAINTAINERS<MAINTAINERS>`{.depreted text role=“ref”}文件告诉您的那样发送或归档报告，除非这是前面概述的“高优先级问题”之一：在这种情况下，请在发送报告之前先阅读下一小节。

### Special handling for high priority issues

Reports for high priority issues need special handling.


**Severe issues**: make sure the subject or ticket title as well as the first paragraph makes the severeness obvious.

> **严重问题**：确保主题或标题以及第一段使严重性显而易见。


**Regressions**: make the report\'s subject start with \'\[REGRESSION\]\'.

> **回归**：使报告的主题以\'\[REGRESSION \]\'开头。


In case you performed a successful bisection, use the title of the change that introduced the regression as the second part of your subject. Make the report also mention the commit id of the culprit. In case of an unsuccessful bisection, make your report mention the latest tested version that\'s working fine (say 5.7) and the oldest where the issue occurs (say 5.8-rc1).

> 如果你成功地进行了平分，请使用引入回归的变化的标题作为主题的第二部分。报告中还应提及罪犯的犯罪身份。如果平分不成功，请在报告中提及运行良好的最新测试版本（如5.7）和出现问题的最旧版本（如5.8-rc1）。


When sending the report by mail, CC the Linux regressions mailing list (<regressions@lists.linux.dev>). In case the report needs to be filed to some web tracker, proceed to do so. Once filed, forward the report by mail to the regressions list; CC the maintainer and the mailing list for the subsystem in question. Make sure to inline the forwarded report, hence do not attach it. Also add a short note at the top where you mention the URL to the ticket.

> 通过邮件发送报告时，抄送Linux回归邮件列表(<regressions@lists.linux.dev>). 如果报告需要提交给某个网络跟踪器，请继续进行。提交后，通过邮件将报告转发到回归列表；抄送有问题的子系统的维护人员和邮件列表。确保内联转发的报告，因此不要附加它。还要在顶部添加一个简短的注释，在其中提到票证的URL。


When mailing or forwarding the report, in case of a successful bisection add the author of the culprit to the recipients; also CC everyone in the signed-off-by chain, which you find at the end of its commit message.

> 在邮寄或转发报告时，如果成功平分，则将罪犯的作者添加到收件人中；还抄送签名者链中的每个人，您可以在其提交消息的末尾找到该链。


**Security issues**: for these issues your will have to evaluate if a short-term risk to other users would arise if details were publicly disclosed. If that\'s not the case simply proceed with reporting the issue as described. For issues that bear such a risk you will need to adjust the reporting process slightly:

> **安全问题**：对于这些问题，您必须评估如果细节被公开，是否会对其他用户产生短期风险。如果不是这样，只需按照说明报告问题。对于承担此类风险的问题，您需要稍微调整报告流程：

> -   If the MAINTAINERS file instructed you to report the issue by mail, do not CC any public mailing lists.
> -   If you were supposed to file the issue in a bug tracker make sure to mark the ticket as \'private\' or \'security issue\'. If the bug tracker does not offer a way to keep reports private, forget about it and send your report as a private mail to the maintainers instead.


In both cases make sure to also mail your report to the addresses the MAINTAINERS file lists in the section \'security contact\'. Ideally directly CC them when sending the report by mail. If you filed it in a bug tracker, forward the report\'s text to these addresses; but on top of it put a small note where you mention that you filed it with a link to the ticket.

> 在这两种情况下，请确保也将您的报告邮寄到“安全联系人”部分中MAINTAINERS文件列出的地址。理想情况下，通过邮件发送报告时直接抄送他们。如果您将报告提交到错误跟踪器中，请将报告的文本转发到这些地址；但在上面放一张小纸条，上面你提到你提交了一张带有门票链接的纸条。

See Documentation/process/security-bugs.rst for more information.

## Duties after the report went out

> *Wait for reactions and keep the thing rolling until you can accept the outcome in one way or the other. Thus react publicly and in a timely manner to any inquiries. Test proposed fixes. Do proactive testing: retest with at least every first release candidate (RC) of a new mainline version and report your results. Send friendly reminders if things stall. And try to help yourself, if you don\'t get any help or if it\'s unsatisfying.*


If your report was good and you are really lucky then one of the developers might immediately spot what\'s causing the issue; they then might write a patch to fix it, test it, and send it straight for integration in mainline while tagging it for later backport to stable and longterm kernels that need it. Then all you need to do is reply with a \'Thank you very much\' and switch to a version with the fix once it gets released.

> 如果你的报告很好，而且你真的很幸运，那么其中一位开发人员可能会立即发现问题的原因；然后，他们可能会编写一个补丁来修复它，对它进行测试，并将它直接发送到主线中进行集成，同时标记它，以便稍后将其备份到需要它的稳定和长期内核。然后，你所需要做的就是回复“非常感谢”，并在它发布后切换到具有修复的版本。


But this ideal scenario rarely happens. That\'s why the job is only starting once you got the report out. What you\'ll have to do depends on the situations, but often it will be the things listed below. But before digging into the details, here are a few important things you need to keep in mind for this part of the process.

> 但这种理想的情况很少发生。这就是为什么这份工作只有在你拿到报告后才开始的原因。你必须做什么取决于情况，但通常是下面列出的事情。但在深入了解细节之前，在这个过程的这一部分，你需要记住一些重要的事情。

### General advice for further interactions


**Always reply in public**: When you filed the issue in a bug tracker, always reply there and do not contact any of the developers privately about it. For mailed reports always use the \'Reply-all\' function when replying to any mails you receive. That includes mails with any additional data you might want to add to your report: go to your mail applications \'Sent\' folder and use \'reply-all\' on your mail with the report. This approach will make sure the public mailing list(s) and everyone else that gets involved over time stays in the loop; it also keeps the mail thread intact, which among others is really important for mailing lists to group all related mails together.

> **始终公开回复**：当您在错误跟踪器中提交问题时，始终在那里回复，不要私下联系任何开发人员。对于邮寄报告，在回复您收到的任何邮件时，始终使用“全部回复”功能。这包括包含您可能希望添加到报告中的任何其他数据的邮件：转到邮件应用程序“发送”文件夹，并在包含报告的邮件中使用“全部发送”。这种方法将确保公共邮件列表和随着时间的推移参与的其他所有人都处于循环中；它还保持邮件线程的完整性，这对于邮件列表将所有相关邮件分组在一起非常重要。


There are just two situations where a comment in a bug tracker or a \'Reply-all\' is unsuitable:

> 只有两种情况下，错误跟踪器中的注释或“Reply-all\”不合适：

> -   Someone tells you to send something privately.
> -   You were told to send something, but noticed it contains sensitive information that needs to be kept private. In that case it\'s okay to send it in private to the developer that asked for it. But note in the ticket or a mail that you did that, so everyone else knows you honored the request.


**Do research before asking for clarifications or help**: In this part of the process someone might tell you to do something that requires a skill you might not have mastered yet. For example, you might be asked to use some test tools you never have heard of yet; or you might be asked to apply a patch to the Linux kernel sources to test if it helps. In some cases it will be fine sending a reply asking for instructions how to do that. But before going that route try to find the answer own your own by searching the internet; alternatively consider asking in other places for advice. For example ask a friend or post about it to a chatroom or forum you normally hang out.

> **在寻求澄清或帮助之前进行研究**：在这个过程的这一部分，有人可能会告诉你做一些需要你可能还没有掌握的技能的事情。例如，您可能会被要求使用一些您从未听说过的测试工具；或者您可能会被要求对Linux内核源应用一个补丁来测试它是否有帮助。在某些情况下，可以发送回复，询问如何操作。但在走这条路之前，试着通过搜索互联网找到自己的答案；或者考虑到其他地方寻求建议。例如，询问朋友或将其发布到你通常常去的聊天室或论坛。


**Be patient**: If you are really lucky you might get a reply to your report within a few hours. But most of the time it will take longer, as maintainers are scattered around the globe and thus might be in a different time zone -- one where they already enjoy their night away from keyboard.

> **耐心点**：如果你真的很幸运，你的报告可能会在几个小时内得到回复。但大多数情况下，这将需要更长的时间，因为维护人员分散在全球各地，因此可能处于不同的时区——在那里他们已经享受了远离键盘的夜晚。


In general, kernel developers will take one to five business days to respond to reports. Sometimes it will take longer, as they might be busy with the merge windows, other work, visiting developer conferences, or simply enjoying a long summer holiday.

> 通常，内核开发人员需要一到五个工作日才能对报告做出响应。有时这需要更长的时间，因为他们可能忙于合并窗口、其他工作、参加开发者会议，或者只是享受一个漫长的暑假。


The \'issues of high priority\' (see above for an explanation) are an exception here: maintainers should address them as soon as possible; that\'s why you should wait a week at maximum (or just two days if it\'s something urgent) before sending a friendly reminder.

> “高优先级问题”（见上文的解释）在这里是一个例外：维护人员应该尽快解决这些问题；这就是为什么你应该在发送友好的提醒之前最多等一周（如果是紧急情况，则只等两天）。


Sometimes the maintainer might not be responding in a timely manner; other times there might be disagreements, for example if an issue qualifies as regression or not. In such cases raise your concerns on the mailing list and ask others for public or private replies how to move on. If that fails, it might be appropriate to get a higher authority involved. In case of a WiFi driver that would be the wireless maintainers; if there are no higher level maintainers or all else fails, it might be one of those rare situations where it\'s okay to get Linus Torvalds involved.

> 有时维护人员可能没有及时做出响应；其他时候可能会有分歧，例如，一个问题是否符合回归的条件。在这种情况下，在邮件列表上提出你的担忧，并要求其他人公开或私下回复如何继续。如果没有成功，可能需要更高级别的机构参与。如果WiFi驱动程序将是无线维护程序；如果没有更高级别的维护人员，或者所有其他操作都失败了，那么让Linus Torvalds参与进来可能是一种罕见的情况。


**Proactive testing**: Every time the first pre-release (the \'rc1\') of a new mainline kernel version gets released, go and check if the issue is fixed there or if anything of importance changed. Mention the outcome in the ticket or in a mail you sent as reply to your report (make sure it has all those in the CC that up to that point participated in the discussion). This will show your commitment and that you are willing to help. It also tells developers if the issue persists and makes sure they do not forget about it. A few other occasional retests (for example with rc3, rc5 and the final) are also a good idea, but only report your results if something relevant changed or if you are writing something anyway.

> **主动测试**：每次发布新主线内核版本的第一个预发行版（“c1”）时，都要去检查问题是否得到了解决，或者重要的事情是否发生了变化。在你的报告回复单或邮件中提及结果（确保CC中所有参与讨论的人都参与了讨论）。这将表明你的承诺和你愿意提供帮助。它还告诉开发人员问题是否持续存在，并确保他们不会忘记这一点。其他一些偶尔的重新测试（例如rc3、rc5和最终测试）也是一个好主意，但只有在相关内容发生变化或你正在写东西时才报告结果。


With all these general things off the table let\'s get into the details of how to help to get issues resolved once they were reported.

> 有了所有这些一般性的事情，让我们来了解一下如何在问题被报告后帮助解决这些问题的细节。

### Inquires and testing request

Here are your duties in case you got replies to your report:


**Check who you deal with**: Most of the time it will be the maintainer or a developer of the particular code area that will respond to your report. But as issues are normally reported in public it could be anyone that\'s replying --- including people that want to help, but in the end might guide you totally off track with their questions or requests. That rarely happens, but it\'s one of many reasons why it\'s wise to quickly run an internet search to see who you\'re interacting with. By doing this you also get aware if your report was heard by the right people, as a reminder to the maintainer (see below) might be in order later if discussion fades out without leading to a satisfying solution for the issue.

> **检查你与谁打交道**：大多数时候，特定代码区域的维护人员或开发人员会对你的报告做出回应。但由于问题通常是公开报道的，可能是任何人在回复——包括想帮忙的人，但最终可能会让你完全偏离他们的问题或请求。这种情况很少发生，但这也是为什么快速进行互联网搜索以查看与谁互动是明智的众多原因之一。通过这样做，你也会意识到你的报告是否被合适的人听到了，因为如果讨论逐渐消失，而没有找到令人满意的问题解决方案，那么提醒维护人员（见下文）可能会在以后生效。


**Inquiries for data**: Often you will be asked to test something or provide additional details. Try to provide the requested information soon, as you have the attention of someone that might help and risk losing it the longer you wait; that outcome is even likely if you do not provide the information within a few business days.

> **查询数据**：通常您会被要求测试某些内容或提供其他详细信息。尽量尽快提供所需信息，因为你得到了可能会提供帮助的人的关注，而且等待的时间越长，就有失去信息的风险；如果你不在几个工作日内提供信息，这种结果甚至是可能的。


**Requests for testing**: When you are asked to test a diagnostic patch or a possible fix, try to test it in timely manner, too. But do it properly and make sure to not rush it: mixing things up can happen easily and can lead to a lot of confusion for everyone involved. A common mistake for example is thinking a proposed patch with a fix was applied, but in fact wasn\'t. Things like that happen even to experienced testers occasionally, but they most of the time will notice when the kernel with the fix behaves just as one without it.

> **测试请求**：当您被要求测试诊断补丁或可能的修复程序时，也要尝试及时进行测试。但要做得正确，确保不要仓促行事：把事情搞混很容易发生，会给所有相关人员带来很多困惑。例如，一个常见的错误是认为已经应用了带有修复程序的拟议修补程序，但事实并非如此。这样的事情偶尔也会发生在有经验的测试人员身上，但他们大多数时候都会注意到，当有修复的内核表现得和没有修复的内核一样时。

### What to do when nothing of substance happens


Some reports will not get any reaction from the responsible Linux kernel developers; or a discussion around the issue evolved, but faded out with nothing of substance coming out of it.

> 有些报告不会得到负责任的Linux内核开发人员的任何反应；或者围绕这个问题的讨论逐渐演变，但逐渐消失，没有任何实质内容。


In these cases wait two (better: three) weeks before sending a friendly reminder: maybe the maintainer was just away from keyboard for a while when your report arrived or had something more important to take care of. When writing the reminder, kindly ask if anything else from your side is needed to get the ball running somehow. If the report got out by mail, do that in the first lines of a mail that is a reply to your initial mail (see above) which includes a full quote of the original report below: that\'s on of those few situations where such a \'TOFU\' (Text Over, Fullquote Under) is the right approach, as then all the recipients will have the details at hand immediately in the proper order.

> 在这些情况下，请等待两周（更好：三周）后再发送友好的提醒：当您的报告到达时，维护人员可能只是离开键盘一段时间，或者有更重要的事情需要处理。在写提醒时，请询问是否需要你身边的其他人以某种方式让球跑起来。如果报告是通过邮件发出的，请在邮件的第一行中这样做，该邮件是对您最初邮件的回复（见上文），其中包括以下原始报告的完整引用：这是少数几种情况中的一种，这种“TOFU”（文本上方，完整引用下方）是正确的方法，因为这样所有收件人都会立即按照正确的顺序掌握详细信息。


After the reminder wait three more weeks for replies. If you still don\'t get a proper reaction, you first should reconsider your approach. Did you maybe try to reach out to the wrong people? Was the report maybe offensive or so confusing that people decided to completely stay away from it? The best way to rule out such factors: show the report to one or two people familiar with FLOSS issue reporting and ask for their opinion. Also ask them for their advice how to move forward. That might mean: prepare a better report and make those people review it before you send it out. Such an approach is totally fine; just mention that this is the second and improved report on the issue and include a link to the first report.

> 收到提醒后，再等三周才能收到回复。如果你仍然没有得到适当的反应，你首先应该重新考虑你的方法。你有没有试着去接触错误的人？这份报告可能令人反感，还是令人困惑，以至于人们决定完全远离它？排除这些因素的最佳方法是：将报告展示给一两位熟悉FLOSS问题报告的人，并征求他们的意见。也可以向他们征求如何前进的建议。这可能意味着：准备一份更好的报告，让这些人在你发送之前进行审查。这样的方法是完全可以的；只要提到这是关于这个问题的第二份改进报告，并包括第一份报告的链接。


If the report was proper you can send a second reminder; in it ask for advice why the report did not get any replies. A good moment for this second reminder mail is shortly after the first pre-release (the \'rc1\') of a new Linux kernel version got published, as you should retest and provide a status update at that point anyway (see above).

> 如果报告正确，您可以发送第二次提醒；在信中征求意见，为什么报告没有得到任何答复。第二封提醒邮件的好时机是在新Linux内核版本的第一次预发布（“c1”）发布后不久，因为您无论如何都应该重新测试并提供状态更新（见上文）。


If the second reminder again results in no reaction within a week, try to contact a higher-level maintainer asking for advice: even busy maintainers by then should at least have sent some kind of acknowledgment.

> 如果第二次提醒在一周内没有任何反应，请尝试联系更高级别的维护人员寻求建议：即使是繁忙的维护人员也应该至少发送某种确认。


Remember to prepare yourself for a disappointment: maintainers ideally should react somehow to every issue report, but they are only obliged to fix those \'issues of high priority\' outlined earlier. So don\'t be too devastating if you get a reply along the lines of \'thanks for the report, I have more important issues to deal with currently and won\'t have time to look into this for the foreseeable future\'.

> 记住要为失望做好准备：理想情况下，维护人员应该对每个问题报告做出某种反应，但他们只需要解决前面概述的“高优先级问题”。因此，如果你得到的回复是“谢谢这份报告，我目前有更重要的问题要处理，在可预见的未来没有时间研究”，不要太过破坏性。


It\'s also possible that after some discussion in the bug tracker or on a list nothing happens anymore and reminders don\'t help to motivate anyone to work out a fix. Such situations can be devastating, but is within the cards when it comes to Linux kernel development. This and several other reasons for not getting help are explained in \'Why some issues won\'t get any reaction or remain unfixed after being reported\' near the end of this document.

> 也有可能在bug跟踪器或列表中进行了一些讨论后，什么都没有发生，提醒也无助于激励任何人找到解决方案。这种情况可能是毁灭性的，但在Linux内核开发方面是不可能的。本文档末尾的“为什么某些问题在被报告后不会得到任何反应或仍然没有得到解决”中解释了这一点和其他几个没有得到帮助的原因。


Don\'t get devastated if you don\'t find any help or if the issue in the end does not get solved: the Linux kernel is FLOSS and thus you can still help yourself. You for example could try to find others that are affected and team up with them to get the issue resolved. Such a team could prepare a fresh report together that mentions how many you are and why this is something that in your option should get fixed. Maybe together you can also narrow down the root cause or the change that introduced a regression, which often makes developing a fix easier. And with a bit of luck there might be someone in the team that knows a bit about programming and might be able to write a fix.

> 如果你找不到任何帮助，或者问题最终没有得到解决，不要崩溃：Linux内核是FLOSS，因此你仍然可以自助。例如，您可以尝试找到其他受影响的人，并与他们合作解决问题。这样的团队可以一起准备一份新的报告，其中提到你有多少人，以及为什么这是你应该解决的问题。也许你还可以一起缩小引入回归的根本原因或变化，这通常会使开发修复程序变得更容易。如果运气好的话，团队中可能会有人对编程有所了解，并且可能能够编写修复程序。

## Reference for \"Reporting regressions within a stable and longterm kernel line\"


This subsection provides details for the steps you need to perform if you face a regression within a stable and longterm kernel line.

> 本小节提供了如果您在稳定和长期的内核行中面临回归，则需要执行的步骤的详细信息。

### Make sure the particular version line still gets support

> *Check if the kernel developers still maintain the Linux kernel version line you care about: go to the front page of kernel.org and make sure it mentions the latest release of the particular version line without an \'\[EOL\]\' tag.*


Most kernel version lines only get supported for about three months, as maintaining them longer is quite a lot of work. Hence, only one per year is chosen and gets supported for at least two years (often six). That\'s why you need to check if the kernel developers still support the version line you care for.

> 大多数内核版本行只支持大约三个月，因为维护它们的时间要长得多。因此，每年只选择一个，并获得至少两年（通常为六年）的支持。这就是为什么您需要检查内核开发人员是否仍然支持您所关心的版本行。


Note, if kernel.org lists two stable version lines on the front page, you should consider switching to the newer one and forget about the older one: support for it is likely to be abandoned soon. Then it will get a \"end-of-life\" (EOL) stamp. Version lines that reached that point still get mentioned on the kernel.org front page for a week or two, but are unsuitable for testing and reporting.

> 请注意，如果kernel.org在首页上列出了两行稳定的版本，那么您应该考虑切换到新的版本，而忘记旧的版本：对它的支持可能很快就会被放弃。然后它将得到一个“寿命终止”（EOL）标记。达到这一点的版本行仍然会在kernel.org的首页上被提及一两周，但不适合测试和报告。

### Search stable mailing list

> *Check the archives of the Linux stable mailing list for existing reports.*


Maybe the issue you face is already known and was fixed or is about to. Hence, [search the archives of the Linux stable mailing list](https://lore.kernel.org/stable/) for reports about an issue like yours. If you find any matches, consider joining the discussion, unless the fix is already finished and scheduled to get applied soon.

> 也许你面临的问题是已知的，并且已经解决或即将解决。因此，[搜索Linux稳定邮件列表的档案](https://lore.kernel.org/stable/)对于像你这样的问题的报道。如果找到任何匹配项，请考虑加入讨论，除非修复程序已经完成并计划很快应用。

### Reproduce issue with the newest release

> *Install the latest release from the particular version line as a vanilla kernel. Ensure this kernel is not tainted and still shows the problem, as the issue might have already been fixed there. If you first noticed the problem with a vendor kernel, check a vanilla build of the last version known to work performs fine as well.*


Before investing any more time in this process you want to check if the issue was already fixed in the latest release of version line you\'re interested in. This kernel needs to be vanilla and shouldn\'t be tainted before the issue happens, as detailed outlined already above in the section \"Install a fresh kernel for testing\".

> 在这个过程中投入更多时间之前，你需要检查这个问题是否已经在你感兴趣的最新版本中得到了解决。这个内核需要是普通的，在问题发生之前不应该受到污染，正如上面“安装一个新的内核进行测试”一节中已经详细介绍的那样。


Did you first notice the regression with a vendor kernel? Then changes the vendor applied might be interfering. You need to rule that out by performing a recheck. Say something broke when you updated from 5.10.4-vendor.42 to 5.10.5-vendor.43. Then after testing the latest 5.10 release as outlined in the previous paragraph check if a vanilla build of Linux 5.10.4 works fine as well. If things are broken there, the issue does not qualify as upstream regression and you need switch back to the main step-by-step guide to report the issue.

> 您第一次注意到供应商内核的回归了吗？那么供应商应用的更改可能会造成干扰。你需要通过重新检查来排除这种情况。当您从5.10.4-vendor.42更新到5.10.5-vendor.43时，请说出一些损坏的内容。然后，在测试了上一段中概述的最新5.10版本后，检查Linux 5.10.4的普通版本是否也能正常工作。如果出现问题，则该问题不属于上游回归，您需要切换回主要分步指南来报告该问题。

### Report the regression

> *Send a short problem report to the Linux stable mailing list (stable@vger.kernel.org) and CC the Linux regressions mailing list (regressions@lists.linux.dev); if you suspect the cause in a particular subsystem, CC its maintainer and its mailing list. Roughly describe the issue and ideally explain how to reproduce it. Mention the first version that shows the problem and the last version that\'s working fine. Then wait for further instructions.*


When reporting a regression that happens within a stable or longterm kernel line (say when updating from 5.10.4 to 5.10.5) a brief report is enough for the start to get the issue reported quickly. Hence a rough description to the stable and regressions mailing list is all it takes; but in case you suspect the cause in a particular subsystem, CC its maintainers and its mailing list as well, because that will speed things up.

> 当报告在稳定或长期内核行内发生的回归时（例如，当从5.10.4更新到5.10.5时），一份简短的报告就足够开始快速报告问题了。因此，只需要对稳定和回归邮件列表进行粗略描述；但如果您怀疑某个子系统中的原因，也可以抄送其维护人员和邮件列表，因为这会加快速度。


And note, it helps developers a great deal if you can specify the exact version that introduced the problem. Hence if possible within a reasonable time frame, try to find that version using vanilla kernels. Lets assume something broke when your distributor released a update from Linux kernel 5.10.5 to 5.10.8. Then as instructed above go and check the latest kernel from that version line, say 5.10.9. If it shows the problem, try a vanilla 5.10.5 to ensure that no patches the distributor applied interfere. If the issue doesn\'t manifest itself there, try 5.10.7 and then (depending on the outcome) 5.10.8 or 5.10.6 to find the first version where things broke. Mention it in the report and state that 5.10.9 is still broken.

> 请注意，如果您能够指定引入问题的确切版本，这将对开发人员有很大帮助。因此，如果可能的话，在合理的时间范围内，试着用香草仁找到那个版本。假设您的发行商发布了从Linux内核5.10.5到5.10.8的更新时出现了问题。然后按照上面的指示，从版本行检查最新的内核，比如5.10.9。如果它显示出问题，请尝试香草5.10.5，以确保分配器应用的贴片不会干扰。如果问题没有在那里表现出来，请尝试5.10.7，然后（取决于结果）5.10.8或5.10.6，找到出现问题的第一个版本。在报告中提到它，并说明5.10.9仍然被破坏。


What the previous paragraph outlines is basically a rough manual \'bisection\'. Once your report is out your might get asked to do a proper one, as it allows to pinpoint the exact change that causes the issue (which then can easily get reverted to fix the issue quickly). Hence consider to do a proper bisection right away if time permits. See the section \'Special care for regressions\' and the document Documentation/admin-guide/bug-bisect.rst for details how to perform one. In case of a successful bisection add the author of the culprit to the recipients; also CC everyone in the signed-off-by chain, which you find at the end of its commit message.

> 上一段概述的内容基本上是一个粗略的手册。一旦你的报告出来，你可能会被要求做一个正确的报告，因为它可以准确地找出导致问题的确切变化（然后可以很容易地恢复以快速解决问题）。因此，如果时间允许，可以考虑立即进行适当的平分。有关如何执行回归的详细信息，请参阅“回归的特殊护理”一节和文档Documentation/admin guide/bug-bisect.rst。在成功平分的情况下，将罪犯的作者添加到接受者中；还抄送签名者链中的每个人，您可以在其提交消息的末尾找到该链。

## Reference for \"Reporting issues only occurring in older kernel version lines\"


This section provides details for the steps you need to take if you could not reproduce your issue with a mainline kernel, but want to see it fixed in older version lines (aka stable and longterm kernels).

> 本节详细介绍了如果您不能用主线内核重现问题，但希望在旧版本行（也称为稳定内核和长期内核）中解决问题时需要采取的步骤。

### Some fixes are too complex

> *Prepare yourself for the possibility that going through the next few steps might not get the issue solved in older releases: the fix might be too big or risky to get backported there.*


Even small and seemingly obvious code-changes sometimes introduce new and totally unexpected problems. The maintainers of the stable and longterm kernels are very aware of that and thus only apply changes to these kernels that are within rules outlined in Documentation/process/stable-kernel-rules.rst.

> 即使是看似显而易见的微小代码更改，有时也会带来新的、完全出乎意料的问题。稳定内核和长期内核的维护人员非常清楚这一点，因此只对Documentation/process/stable-kernel-rules.rst中列出的规则中的内核进行更改。


Complex or risky changes for example do not qualify and thus only get applied to mainline. Other fixes are easy to get backported to the newest stable and longterm kernels, but too risky to integrate into older ones. So be aware the fix you are hoping for might be one of those that won\'t be backported to the version line your care about. In that case you\'ll have no other choice then to live with the issue or switch to a newer Linux version, unless you want to patch the fix into your kernels yourself.

> 例如，复杂或有风险的更改不符合条件，因此只能应用于主线。其他修复程序很容易后移植到最新的稳定和长期内核，但集成到旧内核的风险太大。因此，请注意，您所希望的修复程序可能是不会向后移植到您所关心的版本行的修复程序之一。在这种情况下，你将别无选择，要么接受这个问题，要么切换到更新的Linux版本，除非你想自己将修复程序补丁到内核中。

### Common preparations

> *Perform the first three steps in the section \"Reporting issues only occurring in older kernel version lines\" above.*


You need to carry out a few steps already described in another section of this guide. Those steps will let you:

> 您需要执行本指南另一节中已描述的几个步骤。这些步骤将让您：

> -   Check if the kernel developers still maintain the Linux kernel version line you care about.
> -   Search the Linux stable mailing list for exiting reports.
> -   Check with the latest release.

### Check code history and search for existing discussions

> *Search the Linux kernel version control system for the change that fixed the issue in mainline, as its commit message might tell you if the fix is scheduled for backporting already. If you don\'t find anything that way, search the appropriate mailing lists for posts that discuss such an issue or peer-review possible fixes; then check the discussions if the fix was deemed unsuitable for backporting. If backporting was not considered at all, join the newest discussion, asking if it\'s in the cards.*


In a lot of cases the issue you deal with will have happened with mainline, but got fixed there. The commit that fixed it would need to get backported as well to get the issue solved. That\'s why you want to search for it or any discussions abound it.

> 在很多情况下，您处理的问题会发生在主线上，但在那里得到了解决。修复它的提交也需要进行后移植才能解决问题。这就是为什么你想搜索它或任何关于它的讨论。

> -   First try to find the fix in the Git repository that holds the Linux kernel sources. You can do this with the web interfaces [on kernel.org](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/) or its mirror [on GitHub](https://github.com/torvalds/linux); if you have a local clone you alternatively can search on the command line with `git log --grep=<pattern>`.
>
>     If you find the fix, look if the commit message near the end contains a \'stable tag\' that looks like this:
>
>     > Cc: \<<stable@vger.kernel.org>\> \# 5.4+
>
>     If that\'s case the developer marked the fix safe for backporting to version line 5.4 and later. Most of the time it\'s getting applied there within two weeks, but sometimes it takes a bit longer.
>
> -   If the commit doesn\'t tell you anything or if you can\'t find the fix, look again for discussions about the issue. Search the net with your favorite internet search engine as well as the archives for the [Linux kernel developers mailing list](https://lore.kernel.org/lkml/). Also read the section [Locate kernel area that causes the issue]{.title-ref} above and follow the instructions to find the subsystem in question: its bug tracker or mailing list archive might have the answer you are looking for.
>
> -   If you see a proposed fix, search for it in the version control system as outlined above, as the commit might tell you if a backport can be expected.
>
>     -   Check the discussions for any indicators the fix might be too risky to get backported to the version line you care about. If that\'s the case you have to live with the issue or switch to the kernel version line where the fix got applied.
>     -   If the fix doesn\'t contain a stable tag and backporting was not discussed, join the discussion: mention the version where you face the issue and that you would like to see it fixed, if suitable.

### Ask for advice

> *One of the former steps should lead to a solution. If that doesn\'t work out, ask the maintainers for the subsystem that seems to be causing the issue for advice; CC the mailing list for the particular subsystem as well as the stable mailing list.*


If the previous three steps didn\'t get you closer to a solution there is only one option left: ask for advice. Do that in a mail you sent to the maintainers for the subsystem where the issue seems to have its roots; CC the mailing list for the subsystem as well as the stable mailing list (<stable@vger.kernel.org>).

> 如果前三个步骤没有让你更接近解决方案，那么只剩下一个选择：征求建议。在您发送给子系统维护人员的邮件中这样做，问题似乎有其根源；抄送子系统的邮件列表以及稳定的邮件列表(<stable@vger.kernel.org>).

# Why some issues won\'t get any reaction or remain unfixed after being reported


When reporting a problem to the Linux developers, be aware only \'issues of high priority\' (regressions, security issues, severe problems) are definitely going to get resolved. The maintainers or if all else fails Linus Torvalds himself will make sure of that. They and the other kernel developers will fix a lot of other issues as well. But be aware that sometimes they can\'t or won\'t help; and sometimes there isn\'t even anyone to send a report to.

> 当向Linux开发人员报告问题时，请注意只有“高优先级问题”（倒退、安全问题、严重问题）才能得到解决。维护人员，或者如果其他一切都失败了，Linus Torvalds自己会确保这一点。他们和其他内核开发人员也将解决许多其他问题。但要意识到，有时他们不能或不会提供帮助；有时甚至没有人可以发送报告。


This is best explained with kernel developers that contribute to the Linux kernel in their spare time. Quite a few of the drivers in the kernel were written by such programmers, often because they simply wanted to make their hardware usable on their favorite operating system.

> 最好向那些在业余时间为Linux内核做出贡献的内核开发人员解释这一点。内核中相当多的驱动程序都是由这样的程序员编写的，通常是因为他们只是想让自己的硬件在自己喜欢的操作系统上可用。


These programmers most of the time will happily fix problems other people report. But nobody can force them to do, as they are contributing voluntarily.

> 这些程序员大多数时候都会很乐意地解决别人报告的问题。但没有人能强迫他们这样做，因为他们是自愿捐款的。


Then there are situations where such developers really want to fix an issue, but can\'t: sometimes they lack hardware programming documentation to do so. This often happens when the publicly available docs are superficial or the driver was written with the help of reverse engineering.

> 还有一些情况下，这些开发人员真的想解决问题，但却做不到：有时他们缺乏硬件编程文档。当公开的文档很肤浅，或者驱动程序是在逆向工程的帮助下编写的时，就会发生这种情况。


Sooner or later spare time developers will also stop caring for the driver. Maybe their test hardware broke, got replaced by something more fancy, or is so old that it\'s something you don\'t find much outside of computer museums anymore. Sometimes developer stops caring for their code and Linux at all, as something different in their life became way more important. In some cases nobody is willing to take over the job as maintainer -- and nobody can be forced to, as contributing to the Linux kernel is done on a voluntary basis. Abandoned drivers nevertheless remain in the kernel: they are still useful for people and removing would be a regression.

> 业余开发人员迟早也会停止照顾司机。也许他们的测试硬件坏了，被更花哨的东西取代了，或者太旧了，在计算机博物馆之外你再也找不到什么了。有时，开发人员根本不关心他们的代码和Linux，因为他们生活中的一些不同变得更加重要。在某些情况下，没有人愿意承担维护人员的工作，也没有人可以被迫承担，因为对Linux内核的贡献是自愿的。尽管如此，被抛弃的驱动程序仍然存在于内核中：它们对人们仍然有用，删除它们将是一种回归。


The situation is not that different with developers that are paid for their work on the Linux kernel. Those contribute most changes these days. But their employers sooner or later also stop caring for their code or make its programmer focus on other things. Hardware vendors for example earn their money mainly by selling new hardware; quite a few of them hence are not investing much time and energy in maintaining a Linux kernel driver for something they stopped selling years ago. Enterprise Linux distributors often care for a longer time period, but in new versions often leave support for old and rare hardware aside to limit the scope. Often spare time contributors take over once a company orphans some code, but as mentioned above: sooner or later they will leave the code behind, too.

> 对于那些为Linux内核工作而获得报酬的开发人员来说，情况并没有什么不同。如今，这些变化最大。但他们的雇主迟早也会停止关心他们的代码，或者让程序员专注于其他事情。例如，硬件供应商主要通过销售新硬件来赚钱；因此，他们中的相当一部分人没有投入太多时间和精力来维护多年前停止销售的Linux内核驱动程序。企业Linux发行商通常会关心更长的时间段，但在新版本中，通常会将对旧硬件和稀有硬件的支持放在一边，以限制范围。通常，一旦一家公司孤立了一些代码，业余贡献者就会接管，但如上所述：他们迟早也会留下代码。


Priorities are another reason why some issues are not fixed, as maintainers quite often are forced to set those, as time to work on Linux is limited. That\'s true for spare time or the time employers grant their developers to spend on maintenance work on the upstream kernel. Sometimes maintainers also get overwhelmed with reports, even if a driver is working nearly perfectly. To not get completely stuck, the programmer thus might have no other choice than to prioritize issue reports and reject some of them.

> 优先级是一些问题没有得到解决的另一个原因，因为在Linux上工作的时间有限，维护人员经常被迫设置优先级。这对于业余时间或雇主授予开发人员用于上游内核维护工作的时间来说是正确的。有时维护人员也会被报告淹没，即使驾驶员工作得几乎完美。为了避免完全陷入困境，程序员可能别无选择，只能优先考虑问题报告并拒绝其中一些报告。


But don\'t worry too much about all of this, a lot of drivers have active maintainers who are quite interested in fixing as many issues as possible.

> 但不要太担心这一切，很多司机都有积极的维护人员，他们对解决尽可能多的问题很感兴趣。

# Closing words


Compared with other Free/Libre & Open Source Software it\'s hard to report issues to the Linux kernel developers: the length and complexity of this document and the implications between the lines illustrate that. But that\'s how it is for now. The main author of this text hopes documenting the state of the art will lay some groundwork to improve the situation over time.

> 与其他自由/自由和开源软件相比，很难向Linux内核开发人员报告问题：本文档的长度和复杂性以及字里行间的含义说明了这一点。但现在就是这样。本文的主要作者希望记录艺术现状将为随着时间的推移改善这种情况奠定一些基础。
