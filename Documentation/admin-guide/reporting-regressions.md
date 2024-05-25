---
tip: translate by baidu@2024-01-30 21:39:22
---
---
title: Reporting regressions
---


\"*We don\'t cause regressions*\" is the first rule of Linux kernel development; Linux founder and lead developer Linus Torvalds established it himself and ensures it\'s obeyed.

> \“*我们不会造成倒退”是Linux内核开发的第一条规则；Linux创始人兼首席开发人员Linus Torvalds自己建立了它，并确保它得到遵守。


This document describes what the rule means for users and how the Linux kernel\'s development model ensures to address all reported regressions; aspects relevant for kernel developers are left to Documentation/process/handling-regressions.rst.

> 本文档描述了该规则对用户意味着什么，以及Linux内核的开发模型如何确保解决所有报告的回归；与内核开发人员相关的方面留给Documentation/process/handling-regression.rst。

# The important bits (aka \"TL;DR\")


1.  It\'s a regression if something running fine with one Linux kernel works worse or not at all with a newer version. Note, the newer kernel has to be compiled using a similar configuration; the detailed explanations below describes this and other fine print in more detail.

> 1.如果一个Linux内核运行良好的东西在新版本中运行得更差或根本不好，这是一种回归。注意，更新的内核必须使用类似的配置进行编译；下面的详细解释更详细地描述了这一点和其他细节。


2.  Report your issue as outlined in Documentation/admin-guide/reporting-issues.rst, it already covers all aspects important for regressions and repeated below for convenience. Two of them are important: start your report\'s subject with \"\[REGRESSION\]\" and CC or forward it to [the regression mailing list](https://lore.kernel.org/regressions/) (<regressions@lists.linux.dev>).

> 2.按照Documentation/admin guide/reporting-issues中的概述报告您的问题。首先，它已经涵盖了回归的所有重要方面，为了方便起见，在下面重复。其中两个很重要：以“\[REGRESSION \]\”开头并抄送报告主题，或将其转发到[回归邮件列表](https://lore.kernel.org/regressions/) (<regressions@lists.linux.dev>).


3.  Optional, but recommended: when sending or forwarding your report, make the Linux kernel regression tracking bot \"regzbot\" track the issue by specifying when the regression started like this:

> 3.可选，但建议：在发送或转发报告时，通过指定回归何时开始，使Linux内核回归跟踪机器人“regzbot”跟踪问题，如下所示：

        #regzbot introduced v5.13..v5.14-rc1

# All the details on Linux kernel regressions relevant for users

## The important basics

### What is a \"regression\" and what is the \"no regressions rule\"?


It\'s a regression if some application or practical use case running fine with one Linux kernel works worse or not at all with a newer version compiled using a similar configuration. The \"no regressions rule\" forbids this to take place; if it happens by accident, developers that caused it are expected to quickly fix the issue.

> 如果某个应用程序或实际用例在一个Linux内核上运行良好，或者在使用类似配置编译的新版本上运行较差，那么这就是回归。“无回归规则”禁止这种情况发生；如果它是偶然发生的，那么引起它的开发人员有望迅速解决这个问题。


It thus is a regression when a WiFi driver from Linux 5.13 works fine, but with 5.14 doesn\'t work at all, works significantly slower, or misbehaves somehow. It\'s also a regression if a perfectly working application suddenly shows erratic behavior with a newer kernel version; such issues can be caused by changes in procfs, sysfs, or one of the many other interfaces Linux provides to userland software. But keep in mind, as mentioned earlier: 5.14 in this example needs to be built from a configuration similar to the one from 5.13. This can be achieved using `make olddefconfig`, as explained in more detail below.

> 因此，当Linux 5.13中的WiFi驱动程序运行良好，但使用5.14时根本不起作用，运行速度明显较慢，或行为不端时，这是一种回归。如果一个运行良好的应用程序在更新的内核版本中突然显示出不稳定的行为，这也是一种回归；这样的问题可能是由procfs、sysfs或Linux提供给userland软件的许多其他接口之一的更改引起的。但请记住，如前所述：本例中的5.14需要根据与5.13中的配置类似的配置构建。这可以使用“make-olddefconfig”来实现，下面将对此进行更详细的解释。


Note the \"practical use case\" in the first sentence of this section: developers despite the \"no regressions\" rule are free to change any aspect of the kernel and even APIs or ABIs to userland, as long as no existing application or use case breaks.

> 请注意本节第一句中的“实际用例”：尽管有“无回归”规则，但开发人员可以自由地将内核的任何方面，甚至API或ABI更改为userland，只要没有现有的应用程序或用例中断。


Also be aware the \"no regressions\" rule covers only interfaces the kernel provides to the userland. It thus does not apply to kernel-internal interfaces like the module API, which some externally developed drivers use to hook into the kernel.

> 还要注意，“无回归”规则只涵盖内核提供给用户的接口。因此，它不适用于内核内部接口，如模块API，一些外部开发的驱动程序使用该模块挂接内核。

### How do I report a regression?


Just report the issue as outlined in Documentation/admin-guide/reporting-issues.rst, it already describes the important points. The following aspects outlined there are especially relevant for regressions:

> 只需按照Documentation/admin guide/reporting-issues中的概述报告问题即可。首先，它已经描述了要点。其中概述的以下方面与回归特别相关：

> -   When checking for existing reports to join, also search the [archives of the Linux regressions mailing list](https://lore.kernel.org/regressions/) and [regzbot\'s web-interface](https://linux-regtracking.leemhuis.info/regzbot/).
>
> -   Start your report\'s subject with \"\[REGRESSION\]\".
>
> -   In your report, clearly mention the last kernel version that worked fine and the first broken one. Ideally try to find the exact change causing the regression using a bisection, as explained below in more detail.
>
> -   Remember to let the Linux regressions mailing list (<regressions@lists.linux.dev>) know about your report:
>
>     -   If you report the regression by mail, CC the regressions list.
>     -   If you report your regression to some bug tracker, forward the submitted report by mail to the regressions list while CCing the maintainer and the mailing list for the subsystem in question.
>
>     If it\'s a regression within a stable or longterm series (e.g. v5.15.3..v5.15.5), remember to CC the [Linux stable mailing list](https://lore.kernel.org/stable/) (<stable@vger.kernel.org>).
>
> > In case you performed a successful bisection, add everyone to the CC the culprit\'s commit message mentions in lines starting with \"Signed-off-by:\".


When CCing for forwarding your report to the list, consider directly telling the aforementioned Linux kernel regression tracking bot about your report. To do that, include a paragraph like this in your mail:

> 当请求将您的报告转发到列表时，请考虑直接将报告告知上述Linux内核回归跟踪机器人。要做到这一点，请在邮件中包含这样一段内容：

    #regzbot introduced: v5.13..v5.14-rc1


Regzbot will then consider your mail a report for a regression introduced in the specified version range. In above case Linux v5.13 still worked fine and Linux v5.14-rc1 was the first version where you encountered the issue. If you performed a bisection to find the commit that caused the regression, specify the culprit\'s commit-id instead:

> 然后，Regzbot会将您的邮件视为在指定版本范围内引入的回归的报告。在上面的例子中，Linux v5.13仍然运行良好，Linux v5.14-rc1是您遇到该问题的第一个版本。如果您执行了平分以查找导致回归的提交，请指定罪魁祸首的提交id：

    #regzbot introduced: 1f2e3d4c5d


Placing such a \"regzbot command\" is in your interest, as it will ensure the report won\'t fall through the cracks unnoticed. If you omit this, the Linux kernel\'s regressions tracker will take care of telling regzbot about your regression, as long as you send a copy to the regressions mailing lists. But the regression tracker is just one human which sometimes has to rest or occasionally might even enjoy some time away from computers (as crazy as that might sound). Relying on this person thus will result in an unnecessary delay before the regressions becomes mentioned [on the list of tracked and unresolved Linux kernel regressions](https://linux-regtracking.leemhuis.info/regzbot/) and the weekly regression reports sent by regzbot. Such delays can result in Linus Torvalds being unaware of important regressions when deciding between \"continue development or call this finished and release the final?\".

> 放置这样一个“regzbot命令”符合您的利益，因为它将确保报告不会在未被注意的情况下从裂缝中掉出来。如果您忽略了这一点，Linux内核的回归跟踪器将负责告诉regzbot您的回归，只要您向回归邮件列表发送一份副本。但回归跟踪器只是一个人，有时不得不休息，有时甚至可能享受离开电脑的一段时间（听起来很疯狂）。因此，在提到回归之前（在已跟踪和未解决的Linux内核回归列表上），依赖此人将导致不必要的延迟(https://linux-regtracking.leemhuis.info/regzbot/)以及regzbot发送的每周回归报告。这种延迟可能导致Linus Torvalds在决定“继续开发还是称之为完成并发布最终版本？”时没有意识到重要的倒退。

### Are really all regressions fixed?


Nearly all of them are, as long as the change causing the regression (the \"culprit commit\") is reliably identified. Some regressions can be fixed without this, but often it\'s required.

> 只要可靠地识别出导致回归的变化（“罪魁祸首提交”），几乎所有这些都是。一些回归可以在没有这一点的情况下修复，但通常是必需的。

### Who needs to find the root cause of a regression?


Developers of the affected code area should try to locate the culprit on their own. But for them that\'s often impossible to do with reasonable effort, as quite a lot of issues only occur in a particular environment outside the developer\'s reach \-- for example, a specific hardware platform, firmware, Linux distro, system\'s configuration, or application. That\'s why in the end it\'s often up to the reporter to locate the culprit commit; sometimes users might even need to run additional tests afterwards to pinpoint the exact root cause. Developers should offer advice and reasonably help where they can, to make this process relatively easy and achievable for typical users.

> 受影响代码区域的开发人员应该尝试自己找到罪魁祸首。但对他们来说，这往往是不可能通过合理的努力来做到的，因为相当多的问题只发生在开发人员无法触及的特定环境中——例如，特定的硬件平台、固件、Linux发行版、系统配置或应用程序。这就是为什么最终往往由记者来找到罪犯的原因；有时，用户甚至可能需要在之后运行额外的测试来确定确切的根本原因。开发人员应该尽可能提供建议和合理的帮助，使这一过程对典型用户来说相对容易和可实现。

### How can I find the culprit?


Perform a bisection, as roughly outlined in Documentation/admin-guide/reporting-issues.rst and described in more detail by Documentation/admin-guide/bug-bisect.rst. It might sound like a lot of work, but in many cases finds the culprit relatively quickly. If it\'s hard or time-consuming to reliably reproduce the issue, consider teaming up with other affected users to narrow down the search range together.

> 执行平分，大致如Documentation/admin-guide/reporting-issues.rst中所述，并由Documentation/admin-guide/bug-bisect.rst进行更详细的描述。这听起来可能需要做很多工作，但在许多情况下，相对较快地找到罪魁祸首。如果要可靠地再现问题很难或很耗时，可以考虑与其他受影响的用户合作，共同缩小搜索范围。

### Who can I ask for advice when it comes to regressions?


Send a mail to the regressions mailing list (<regressions@lists.linux.dev>) while CCing the Linux kernel\'s regression tracker (<regressions@leemhuis.info>); if the issue might better be dealt with in private, feel free to omit the list.

> 向回归邮件列表发送邮件(<regressions@lists.linux.dev>)在CCing Linux内核的回归跟踪器时(<regressions@leemhuis.info>); 如果这个问题最好私下处理，请随意省略列表。

## Additional details about regressions

### What is the goal of the \"no regressions rule\"?


Users should feel safe when updating kernel versions and not have to worry something might break. This is in the interest of the kernel developers to make updating attractive: they don\'t want users to stay on stable or longterm Linux series that are either abandoned or more than one and a half years old. That\'s in everybody\'s interest, as [those series might have known bugs, security issues, or other problematic aspects already fixed in later versions](http://www.kroah.com/log/blog/2018/08/24/what-stable-kernel-should-i-use/). Additionally, the kernel developers want to make it simple and appealing for users to test the latest pre-release or regular release. That\'s also in everybody\'s interest, as it\'s a lot easier to track down and fix problems, if they are reported shortly after being introduced.

> 用户在更新内核版本时应该感到安全，不必担心某些东西可能会损坏。这符合内核开发人员的利益，使更新具有吸引力：他们不希望用户停留在被放弃或使用超过一年半的稳定或长期的Linux系列上。这符合每个人的利益，因为[这些系列可能有已知的错误、安全问题或其他问题，已经在后续版本中修复](http://www.kroah.com/log/blog/2018/08/24/what-stable-kernel-should-i-use/). 此外，内核开发人员希望让用户测试最新的预发行版或常规发行版变得简单而有吸引力。这也符合每个人的利益，因为如果问题在引入后不久被报告，那么追踪和解决问题会容易得多。

### Is the \"no regressions\" rule really adhered in practice?


It\'s taken really seriously, as can be seen by many mailing list posts from Linux creator and lead developer Linus Torvalds, some of which are quoted in Documentation/process/handling-regressions.rst.

> 正如Linux创建者和首席开发人员Linus Torvalds的许多邮件列表帖子所示，它被认真对待，其中一些帖子被引用在Documentation/process/handling-registrations.rst中。


Exceptions to this rule are extremely rare; in the past developers almost always turned out to be wrong when they assumed a particular situation was warranting an exception.

> 这一规则的例外情况极为罕见；在过去，当开发人员认为某个特定情况需要一个例外时，他们几乎总是错的。

### Who ensures the \"no regressions\" is actually followed?


The subsystem maintainers should take care of that, which are watched and supported by the tree maintainers \-- e.g. Linus Torvalds for mainline and Greg Kroah-Hartman et al. for various stable/longterm series.

> 子系统维护人员应注意这一点，并得到树维护人员的关注和支持，例如主线的Linus Torvalds和各种稳定/长期系列的Greg Kroah-Hartman等人。


All of them are helped by people trying to ensure no regression report falls through the cracks. One of them is Thorsten Leemhuis, who\'s currently acting as the Linux kernel\'s \"regressions tracker\"; to facilitate this work he relies on regzbot, the Linux kernel regression tracking bot. That\'s why you want to bring your report on the radar of these people by CCing or forwarding each report to the regressions mailing list, ideally with a \"regzbot command\" in your mail to get it tracked immediately.

> 所有这些都得到了人们的帮助，他们试图确保没有回归报告漏洞百出。其中之一是Thorsten Leemhuis，他目前担任Linux内核的“回归跟踪器”；为了促进这项工作，他依赖于regzbot，Linux内核回归跟踪机器人。这就是为什么你想通过CCing或将每个报告转发到回归邮件列表来让这些人注意到你的报告，最好在你的邮件中使用“regzbot命令”来立即跟踪它。

### How quickly are regressions normally fixed?


Developers should fix any reported regression as quickly as possible, to provide affected users with a solution in a timely manner and prevent more users from running into the issue; nevertheless developers need to take enough time and care to ensure regression fixes do not cause additional damage.

> 开发人员应尽快修复任何报告的回归，及时为受影响的用户提供解决方案，防止更多用户遇到问题；然而，开发人员需要花足够的时间和精力来确保回归修复不会造成额外的损坏。


The answer thus depends on various factors like the impact of a regression, its age, or the Linux series in which it occurs. In the end though, most regressions should be fixed within two weeks.

> 因此，答案取决于各种因素，如回归的影响、年龄或发生回归的Linux系列。然而，最终，大多数倒退应该在两周内得到解决。

### Is it a regression, if the issue can be avoided by updating some software?


Almost always: yes. If a developer tells you otherwise, ask the regression tracker for advice as outlined above.

> 几乎总是：是的。如果开发人员告诉您其他情况，请向回归跟踪器咨询上述建议。

### Is it a regression, if a newer kernel works slower or consumes more energy?


Yes, but the difference has to be significant. A five percent slow-down in a micro-benchmark thus is unlikely to qualify as regression, unless it also influences the results of a broad benchmark by more than one percent. If in doubt, ask for advice.

> 是的，但差异必须是显著的。因此，微观基准中5%的减速不太可能被视为回归，除非它也会对广义基准的结果产生超过1%的影响。如果有疑问，请咨询。

### Is it a regression, if an external kernel module breaks when updating Linux?


No, as the \"no regression\" rule is about interfaces and services the Linux kernel provides to the userland. It thus does not cover building or running externally developed kernel modules, as they run in kernel-space and hook into the kernel using internal interfaces occasionally changed.

> 不，因为“无回归”规则是关于Linux内核向用户提供的接口和服务。因此，它不包括构建或运行外部开发的内核模块，因为它们在内核空间中运行，并使用偶尔更改的内部接口连接到内核。

### How are regressions handled that are caused by security fixes?


In extremely rare situations security issues can\'t be fixed without causing regressions; those fixes are given way, as they are the lesser evil in the end. Luckily this middling almost always can be avoided, as key developers for the affected area and often Linus Torvalds himself try very hard to fix security issues without causing regressions.

> 在极少数情况下，安全问题无法在不造成倒退的情况下得到解决；这些解决方案被放弃了，因为它们最终是较小的邪恶。幸运的是，这种中等程度的情况几乎总是可以避免的，因为受影响地区的关键开发人员——通常是Linus Torvalds本人——都会非常努力地解决安全问题，而不会造成倒退。


If you nevertheless face such a case, check the mailing list archives if people tried their best to avoid the regression. If not, report it; if in doubt, ask for advice as outlined above.

> 如果你仍然面临这样的情况，如果人们尽力避免回归，请查看邮件列表档案。如果没有，请报告；如有疑问，请咨询上述建议。

### What happens if fixing a regression is impossible without causing another?


Sadly these things happen, but luckily not very often; if they occur, expert developers of the affected code area should look into the issue to find a fix that avoids regressions or at least their impact. If you run into such a situation, do what was outlined already for regressions caused by security fixes: check earlier discussions if people already tried their best and ask for advice if in doubt.

> 不幸的是，这些事情发生了，但幸运的是不经常发生；如果出现这种情况，受影响代码领域的专家开发人员应该调查这个问题，找到一个避免回归或至少避免其影响的解决方案。如果你遇到这种情况，请按照安全修复导致的倒退所述的方法进行：如果人们已经尽了最大努力，请查看之前的讨论，如果有疑问，请寻求建议。


A quick note while at it: these situations could be avoided, if people would regularly give mainline pre-releases (say v5.15-rc1 or -rc3) from each development cycle a test run. This is best explained by imagining a change integrated between Linux v5.14 and v5.15-rc1 which causes a regression, but at the same time is a hard requirement for some other improvement applied for 5.15-rc1. All these changes often can simply be reverted and the regression thus solved, if someone finds and reports it before 5.15 is released. A few days or weeks later this solution can become impossible, as some software might have started to rely on aspects introduced by one of the follow-up changes: reverting all changes would then cause a regression for users of said software and thus is out of the question.

> 在此简要说明：如果人们定期对每个开发周期的主线预发行版（比如v5.15-rc1或-rc3）进行测试运行，这些情况是可以避免的。这可以通过想象Linux v5.14和v.15-rc1之间集成的变化来最好地解释，这会导致回归，但同时也是对5.15-rc1应用的一些其他改进的硬性要求。如果有人在5.15发布之前发现并报告了所有这些变化，通常可以简单地恢复并解决回归问题。几天或几周后，这种解决方案可能变得不可能，因为一些软件可能已经开始依赖于其中一个后续更改所引入的方面：恢复所有更改将导致所述软件的用户回归，因此是不可能的。

### Is it a regression, if some feature I relied on was removed months ago?


It is, but often it\'s hard to fix such regressions due to the aspects outlined in the previous section. It hence needs to be dealt with on a case-by-case basis. This is another reason why it\'s in everybody\'s interest to regularly test mainline pre-releases.

> 是的，但由于上一节中概述的方面，通常很难修复这种回归。因此，需要根据具体情况加以处理。这也是为什么定期测试主线预发布符合每个人利益的另一个原因。

### Does the \"no regression\" rule apply if I seem to be the only affected person?


It does, but only for practical usage: the Linux developers want to be free to remove support for hardware only to be found in attics and museums anymore.

> 确实如此，但仅限于实际用途：Linux开发人员希望可以自由删除对硬件的支持，只在阁楼和博物馆中找到。


Note, sometimes regressions can\'t be avoided to make progress \-- and the latter is needed to prevent Linux from stagnation. Hence, if only very few users seem to be affected by a regression, it for the greater good might be in their and everyone else\'s interest to lettings things pass. Especially if there is an easy way to circumvent the regression somehow, for example by updating some software or using a kernel parameter created just for this purpose.

> 请注意，有时为了取得进展，不可避免地会出现倒退，而后者是防止Linux停滞不前所必需的。因此，如果只有极少数用户似乎受到回归的影响，那么为了更大的利益，让事情过去可能符合他们和其他人的利益。特别是如果有一种简单的方法可以以某种方式绕过回归，例如更新一些软件或使用仅为此目的创建的内核参数。

### Does the regression rule apply for code in the staging tree as well?


Not according to the [help text for the configuration option covering all staging code](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/drivers/staging/Kconfig), which since its early days states:

> 不符合[包含所有暂存代码的配置选项的帮助文本](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/drivers/staging/Kconfig)，自其早期以来，声明：

    Please note that these drivers are under heavy development, may or
    may not work, and may contain userspace interfaces that most likely
    will be changed in the near future.


The staging developers nevertheless often adhere to the \"no regressions\" rule, but sometimes bend it to make progress. That\'s for example why some users had to deal with (often negligible) regressions when a WiFi driver from the staging tree was replaced by a totally different one written from scratch.

> 尽管如此，临时开发人员经常坚持“不倒退”规则，但有时会为了取得进展而改变规则。例如，当阶段树中的WiFi驱动程序被从头开始编写的完全不同的驱动程序取代时，一些用户不得不处理（通常可以忽略不计）倒退。

### Why do later versions have to be \"compiled with a similar configuration\"?


Because the Linux kernel developers sometimes integrate changes known to cause regressions, but make them optional and disable them in the kernel\'s default configuration. This trick allows progress, as the \"no regressions\" rule otherwise would lead to stagnation.

> 因为Linux内核开发人员有时会集成已知会导致倒退的更改，但会使其成为可选的，并在内核的默认配置中禁用它们。这个技巧允许进步，因为“不倒退”规则否则会导致停滞。


Consider for example a new security feature blocking access to some kernel interfaces often abused by malware, which at the same time are required to run a few rarely used applications. The outlined approach makes both camps happy: people using these applications can leave the new security feature off, while everyone else can enable it without running into trouble.

> 例如，考虑一个新的安全功能来阻止对一些经常被恶意软件滥用的内核接口的访问，同时运行一些很少使用的应用程序需要恶意软件。概述的方法让双方都很高兴：使用这些应用程序的人可以关闭新的安全功能，而其他人可以启用它而不会遇到麻烦。

### How to create a configuration similar to the one of an older kernel?


Start your machine with a known-good kernel and configure the newer Linux version with `make olddefconfig`. This makes the kernel\'s build scripts pick up the configuration file (the \".config\" file) from the running kernel as base for the new one you are about to compile; afterwards they set all new configuration options to their default value, which should disable new features that might cause regressions.

> 使用已知的好内核启动机器，并使用“makeolddefconfig”配置较新的Linux版本。这使得内核的构建脚本从正在运行的内核中获取配置文件（“.config”文件），作为即将编译的新内核的基础；之后，他们将所有新的配置选项设置为默认值，这将禁用可能导致倒退的新功能。

### Can I report a regression I found with pre-compiled vanilla kernels?


You need to ensure the newer kernel was compiled with a similar configuration file as the older one (see above), as those that built them might have enabled some known-to-be incompatible feature for the newer kernel. If in doubt, report the matter to the kernel\'s provider and ask for advice.

> 您需要确保新内核是使用与旧内核类似的配置文件编译的（请参阅上文），因为构建这些文件的人可能为新内核启用了一些已知的不兼容功能。如果有疑问，请向内核提供商报告并征求建议。

## More about regression tracking with \"regzbot\"

### What is regression tracking and why should I care about it?


Rules like \"no regressions\" need someone to ensure they are followed, otherwise they are broken either accidentally or on purpose. History has shown this to be true for Linux kernel development as well. That\'s why Thorsten Leemhuis, the Linux Kernel\'s regression tracker, and some people try to ensure all regression are fixed by keeping an eye on them until they are resolved. Neither of them are paid for this, that\'s why the work is done on a best effort basis.

> 像“不倒退”这样的规则需要有人确保它们得到遵守，否则它们会被意外或故意打破。历史已经证明，Linux内核开发也是如此。这就是为什么Linux内核的回归跟踪器Thorsten Leemhuis和一些人试图通过密切关注它们直到它们得到解决来确保所有回归都得到修复。他们两个都没有得到报酬，这就是为什么这项工作是在尽最大努力的基础上完成的。

### Why and how are Linux kernel regressions tracked using a bot?


Tracking regressions completely manually has proven to be quite hard due to the distributed and loosely structured nature of Linux kernel development process. That\'s why the Linux kernel\'s regression tracker developed regzbot to facilitate the work, with the long term goal to automate regression tracking as much as possible for everyone involved.

> 由于Linux内核开发过程的分布式和松散结构，完全手动跟踪回归已经被证明是相当困难的。这就是为什么Linux内核的回归跟踪器开发了regzbot来促进这项工作，其长期目标是为所有参与者尽可能自动化回归跟踪。


Regzbot works by watching for replies to reports of tracked regressions. Additionally, it\'s looking out for posted or committed patches referencing such reports with \"Link:\" tags; replies to such patch postings are tracked as well. Combined this data provides good insights into the current state of the fixing process.

> Regzbot的工作原理是观察对追踪回归报告的回复。此外，它正在寻找引用带有“Link:\”标签的此类报告的已发布或已提交的补丁；对此类补丁发布的回复也会被跟踪。结合这些数据可以很好地了解固定过程的当前状态。

### How to see which regressions regzbot tracks currently?


Check out [regzbot\'s web-interface](https://linux-regtracking.leemhuis.info/regzbot/).

> 查看[regzbot的web界面](https://linux-regtracking.leemhuis.info/regzbot/).

### What kind of issues are supposed to be tracked by regzbot?


The bot is meant to track regressions, hence please don\'t involve regzbot for regular issues. But it\'s okay for the Linux kernel\'s regression tracker if you involve regzbot to track severe issues, like reports about hangs, corrupted data, or internal errors (Panic, Oops, BUG(), warning, \...).

> 该机器人旨在跟踪回归，因此请不要涉及regzbot的常规问题。但是，如果您使用regzbot来跟踪严重的问题，比如关于挂起、损坏的数据或内部错误的报告（Panic、Oops、BUG（）、warning…），那么Linux内核的回归跟踪器也可以。

### How to change aspects of a tracked regression?


By using a \'regzbot command\' in a direct or indirect reply to the mail with the report. The easiest way to do that: find the report in your \"Sent\" folder or the mailing list archive and reply to it using your mailer\'s \"Reply-all\" function. In that mail, use one of the following commands in a stand-alone paragraph (IOW: use blank lines to separate one or multiple of these commands from the rest of the mail\'s text).

> 通过在直接或间接回复带有报告的邮件中使用\'Rzbot命令\'。最简单的方法是：在“已发送”文件夹或邮件列表档案中找到报告，并使用邮件收发程序的“全部回复”功能进行回复。在该邮件中，在独立段落中使用以下命令之一（IOW：使用空行将其中一个或多个命令与邮件的其余文本分隔开）。

> -   Update when the regression started to happen, for example after performing a bisection:
>
>         #regzbot introduced: 1f2e3d4c5d
>
> -   Set or update the title:
>
>         #regzbot title: foo
>
> -   Monitor a discussion or bugzilla.kernel.org ticket where additions aspects of the issue or a fix are discussed::
>
>         #regzbot monitor: https://lore.kernel.org/r/30th.anniversary.repost@klaava.Helsinki.FI/
>         #regzbot monitor: https://bugzilla.kernel.org/show_bug.cgi?id=123456789
>
> -   Point to a place with further details of interest, like a mailing list post or a ticket in a bug tracker that are slightly related, but about a different topic:
>
>         #regzbot link: https://bugzilla.kernel.org/show_bug.cgi?id=123456789
>
> -   Mark a regression as invalid:
>
>         #regzbot invalid: wasn't a regression, problem has always existed


Regzbot supports a few other commands primarily used by developers or people tracking regressions. They and more details about the aforementioned regzbot commands can be found in the [getting started guide](https://gitlab.com/knurd42/regzbot/-/blob/main/docs/getting_started.md) and the [reference documentation](https://gitlab.com/knurd42/regzbot/-/blob/main/docs/reference.md) for regzbot.

> Regzbot支持其他一些主要由开发人员或跟踪回归的人员使用的命令。它们和有关上述regzbot命令的更多详细信息可以在[入门指南]中找到(https://gitlab.com/knurd42/regzbot/-/blob/main/docs/getting_started.md)和[参考文件](https://gitlab.com/knurd42/regzbot/-/blob/main/docs/reference.md)雷兹伯特。
