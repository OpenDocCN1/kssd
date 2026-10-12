
# Reverse Engineering Malware

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-798.htm`](https://www.pediy.com/kssd/pediy08/pediy8-798.htm)

**逆向工程恶意软件**

- 文档编号：`S002-F026`
- 原作者：`Lenny Zeltser`
- 译者：月中人 \[PTG\]
- 审校：
- 发布时间：`2007-01-11`
- 原文：`Reverse Engineering Malware`
- 关键词：逆向工程, 可控环境, 恶意软件, 木马, 保护
- 原文链接：[`www.zeltser.com/reverse-malware-paper/reverse-malware.pdf`](http://www.zeltser.com/reverse-malware-paper/reverse-malware.pdf) May 2001

**摘要**

本文讨论在了解恶意软件如病毒、蠕虫和木马内部工作的时候使用的一些工具和技术。我们描述了一种方法，使用虚拟工作站软件如 `VMware` 建立廉价而且灵活的实验室环境；并示范了使用多种系统监视工具协同反汇编器和调试器一起逆向工程一个木马的过程。本文某些部分是以我们提交给 `GIAC` 的论文为基础，该论文是作为获取 `GCIH` 证书的一个实习作业部分。要了解更多信息请访问[`www.zeltser.com`](http://www.zeltser.com/)

**目录**

1. 第一部分：介绍 ... `2`
   - 1.1 概述 ... `2`
   - 1.2 背景信息 ... `2`
2. 第二部分：方法学 ... `4`
   - 2.1 可控环境 ... `4`
   - 2.2 行为模式 ... `5`
   - 2.3 代码分析 ... `6`
3. 第三部分：特洛伊木马的结构 ... `8`
   - 3.1 本地系统交互 ... `8`
   - 3.2 通信协议 ... `9`
   - 3.3 程序代码 ... `13`
4. 第四部分：保护措施 ... `20`
   - 4.1 传播机制 ... `20`
   - 4.2 特洛伊木马的变种 ... `20`
   - 4.3 特洛伊木马的签名 ... `21`
5. 第五部分：参考书目 ... `23`

# 第一部分 介绍

## 1.1 概述

本论文试图评述逆向工程恶意软件的一种方法。突出该过程本身，而非仅仅关注程序的特点，是有双重原因的。首先，对于在这篇捧场文章中讨论的这个特定的木马(`srvcp.exe`)，仍有许多未解决的问题；如将我们的调查结果定位于全面分析充其量会产生误导。其次，通过开发一种了解恶意软件内部工作的结构化方法，也就是可重复的剖解步骤，可以帮助防卫社区成员。

在分析木马过程中，我们对于攻击者在程序设计上所下的功夫已经留有深刻印象，攻击者乐于费尽心思地创建有弹性的、灵活的恶意软件。因此，逆向工程该木马程序的过程耗费不少时间、但终究还是达到我们的目的了。希望我们的讨论将会鼓励安全专业人士改进本文所建议的方法学，并可能弥补我们对这个特定木马理解上的遗漏之处。

我们要感谢 `Joe Abrams` 对这个木马汇编代码的见解以及对它现实存在环境的阐述。非常感谢 `Slava Frid`，他帮助我们走过程序代码的特别隐秘区域，而且让我们分享他的思想。同时，我们感谢 `Doug Kahler` 与我们分享这个木马还有他的观测记录。最后，感谢 `Jeremy Gaddis` 提醒防卫社区注意这个木马。

## 1.2 背景信息

在 `2000` 年六月 `8` 日，`Jeremy L. Gaddis` 提醒防卫社区注意 `srvcp.exe` 木马的一个变种，即本文讨论的这个。`Jeremy` 在事件邮件列表中发表\[JLG\]，报告发觉入站连接尝试 `TCP` 端口 `113`、企图连接 `Internet` 上某个未知主机，同时未经授权的出站连接尝试一个远程服务器、目的端口是 `TCP` 端口 `6667`。通过调查该事件，`Jeremy` 发现了木马与 `Internet` 多线交谈(`IRC`)网络的联系，在他的消息中描述如下：

> 认出 `6667` 是 `IRC` 服务器经常使用的端口，我启动了一个 `IRC` 客户端并连接到那个主机。它的确是一个 `IRC` 服务器。快速检查显示出 `4` 个在线用户，它们的“Real Name”域被置为“Im trojaned”，其中一个是我的 `IP` 地址。

有的读者可能不熟悉 `IRC`，我们简单说一下，可以把 `IRC` 看作是一个服务器内联的网络，它让使用者可以维持实时在线会话。会话的参与者典型地加入一个热衷于特定主题或兴趣的频道。这是通过把使用者的 `IRC` 客户端连接到一个服务器来实现的，而这个服务器参与所需要的 `IRC` 对等网络。当使用者录入一个消息要让频道参与者看见的时候，这条消息由 `IRC` 服务器转发给参与讨论的客户端，就如评论请求(`RFC`)文档 `1459` 中说明的。\[RFC1459\]

在 `Jeremy` 最初消息发表后数天，在 `2000` 年六月 `10` 日，`Brandon Kittler` 也通过邮件列表转发他关于该木马的体验\[BK\]。`Brandon` 提供了关于该程序可执行文件的位置和相关的注册表项这些细节。基于他的观察和对可执行文件中出现的字符串的检查，他得出结论是该木马能够通过 `IRC` 接受命令，而这些命令具有潜在危险性：

```text
0054C7 PRIVMSG %s :successfully spawned ftp.exe
0054F1 PRIVMSG %s :couldn't spawn ftp.exe
005515 PRIVMSG %s :no more...
00552D PRIVMSG %s :ready and willing...
```

`Brandon` 也指出，该程序在端口 `113` 监听识别请求，而且当它接收到没有遵守识别协议(`Ident protocol`)的请求时该程序崩溃。识别服务的协议在 `RFC 1413` 中定义\[RFC1413\]，在某用户启动一个 `TCP` 连接到一个远程服务器，识别服务允许该服务器确认此用户的登录名字。由于试图控制系统滥用，许多 `IRC` 服务器不接受 `IRC` 连接，除非接入用户的身份能够通过使用识别协议的再确认。

两天后，`2001` 年六月 `12` 日，`Doug Kahler` 报告有关该程序行为的补充信息\[DK\]。`Doug` 提到，在几个月前，他就遭遇一个相似的木马，而且指出当时程序尝试连接 `EFnet` 网络上的一个 `IRC` 服务器、使用键“`soup`”加入一个被命名为“`#mikag`”的频道。`Doug` 还写道：

> 今天刚刚参加了该频道，那里有 `55` 个人，以随机字母和数字记号。我以为他们全部被传染。

`Doug` 稍后会好意地搜索他的文档，并与我们共享他在分析的基础上建立的一个木马拷贝。这对我们的研究是个关键的帮助，因为在这个思路(`thread`)开始时发送到邮件列表的可执行程序被证明是一个良性拷贝，似乎是一个 `Windows` 屏幕保护程序。

就如 `Nick FitzGerald` 在 `2001` 年六月 `13` 日发到事件邮件列表的一个消息中指出，该程序似乎是相对未知的 `Tasmer` 木马的一个变种\[NF\]。`Nick` 证实，在他的体验中该木马加入一个 `IRC` 频道，而且能“通过 `IRC` 上的私人消息进行远程控制”。据 `Nick` 说，有些 `Tasmer` 变种被怀疑有破解分布式口令能力，而且可能有拒绝服务(`DoS`)攻击代理。

关于这个特定木马，杀毒厂商没有提供很多信息，而稍后的病毒数据库的检查发现关于该木马的能力缺乏详细文件证明，这或许归因于它相对低调的姿态。在 `2001` 年六月事件邮件列表上讨论之后，该木马似乎没有引起公众的注意，并且一直保持相对未经调查状态。然而，我们对这个程序的兴趣被 `Pete Schmitt` 最近于 `2001` 年 `3` 月 `10` 日在邮件列表上的发表重新点燃\[PS\]。他对感染他计算机的那个木马的简短描述似乎符合 `8` 个月前在邮件列表上所讨论的那个木马。彼得也观察到，木马尝试使用“`Joe Blow`”这个名字连接 `IRC` 服务器，这一点稍后让我们可以把它联系上该木马的另一个变种。我们的研究方法学和调查结果在后面几页详加评述。

# 第二部分 方法学

## 2.1 可控环境

恶意软件的逆向工程师必须使用灵活的、不张扬的可控实验室环境，以便容易形成有效率、廉价和可靠的研究过程。我们最后选定使用 `VMware` 公司的虚拟工作站软件建立研究环境。`VMware` 藉由提供硬件仿真和虚拟网络服务实现虚拟环境，而且允许我们在一个单一机器上进行操作系统的完全独立安装。使用 `VMware` 可以让多重操作系统同时运行，而且每个虚拟机器“相当于一台 `PC` 机，因为它有一个完全的、未更改的操作系统，一个唯一的网络地址，和一套补充完整的硬件设备。”\[VM1\]

为建立我们的实验室环境，我们使用的一台 `500MHz` 膝上型电脑有 `20GB` 磁盘空间和 `256MB RAM`（随机存储器），操作系统是 `Windows 2000 Professional`，在上面我们安装了 `VMware`。尽管同样定价的桌上型电脑机器会提供更多计算能力，但是为了让我们的实验室尽可能轻便易于携带，我们还是决定使用一台膝上型电脑。因为想为研究提供多种操作系统，我们在 `VMware` 里面创建了三个虚拟机器：红帽子 `Linux 7.0`、`Windows 98` 第二版和 `Windows NT 4.0` 工作站服务包 `6a`。分配给每个客座操作系统 `1.6GB` 磁盘空间和 `64MB RAM`，对于我们的目的来说这足够了。由于 `VMware` 仿真硬件，所以每个客座操作系统的安装必须与典型安装步骤一致。下面图 `2-1` 举例说明我们在主机操作系统里面安装两个同时运行的虚拟机器。

![](img/e06eea4778441d4b41a55e37c79cae55.jpg)

图 `2-1`

每个客座操作系统能够相互独立地操作，`VMware` 允许我们使用一个虚拟网络互连虚拟机器，这是完全封闭在主机机器里面，并没有能力沟通外部世界。当使用 `Windows 2000` 作为主机操作系统的时候，为了实现这个功能，我们必须按照 `VMware` 支持网站的指示\[VM2\]，手工创建一个虚拟的主机专用适配器(连接装置)。适配器被指定在一个私用 `RFC 1918` 范围内的一个 `IP` 地址，以确保它不与我们任何一个现有连接相冲突。\[RFC1918\]

作为主机专用适配器安装的一部份，`VMware` 也安装了一个 `DHCP` 服务器服务，专用于在虚拟网络上配置 `IP` 地址。为了确保虚拟网络真正地与膝上型电脑的物理界面隔离，我们在主机机器上安装了 `ZoneAlarm Pro`，它给我们提供一个舒适的保护层，会在任何数据包试图离开实验室环境时发出警告。不过，这个预防措施不能够保证环境的完全隔离。理想地，`VMware` 主机不应该被连接到生产网络，而且应该使得在那几个虚拟机器看来 `VMware` 主机同样不是必需的。下面图 `2-2` 举例说明安装在我们膝上型电脑里的虚拟网络基础结构。

![](img/563228184e2e9fef10202dc8a416e3b2.jpg)

图 `2-2`

使用 `VMware` 代替物理分离系统的好处之一是能够备份系统并能在大约几分钟内恢复完全系统。每个虚拟机器由使用位于 `VMware` 的 `VMs` 子目录中的一些自包含文件来实现。复制那些被 `VMware` 用来表现虚拟机器的文件就可以备份那个系统。在分析未知的恶意软件时这特别有用，因为此时反复与病毒或木马交互可能感染环境，你需要把系统恢复到一个已知状态。此外，虚拟机器易于复制让工程师们可以维持一个操作系统在不同修正标准上的若干实例。

尽管由 `VMware` 提供高度隔离，在客座操作系统和主机机器之间仍然有一些交互。尤其是，`VMware` 提供了 `VMware` 工具驱动程序，当它们被安装在一个虚拟机器上的时候，将会允许用户在客座操作系统和主机机器之间移动鼠标指针。此外，驱动程序允许虚拟机器共享对主机机器剪贴板的存取。这些功能从虚拟机器内启用，而且主机系统似乎没有禁用它们的能力。这意味着有可能针对一个基于 `VMware` 的实验室用户巧妙地设计一个攻击，它将会从一个虚拟机器内实现在某些层次上访问主机系统。

## 2.2 行为模式

要了解与恶意软件样品相关联的威胁，其方法之一是在一个可控环境里从检查它的行为开始。这典型地包括在实验室里运行该代理，并且研究它与计算机资源如何交互以及如何响应各种不同的诱因。前文所述基于 `VMware` 的实验室环境允许我们使用一个单一屏幕从各个不同系统上监视恶意软件。

我们综合使用进程、磁盘、注册表和网络监视工具来研究在某个特定机器上该木马的活动率，以及它对其他系统的连接尝试。当它尝试与因特网上的系统沟通时，我们能够指引该木马到实验室机器，如此一旦我们完全掌握它与世界如何交互，我们就可以为该木马重复本地环境。

我们发现 `Systernals` 提供的免费软件工具在监听该木马的行为时非常有用。特别地，我们用 `Windows NT/9x` 下的 `Filemon` 观察该木马试图在本地系统上存取哪些文件\[SI1\]。同样，`Windows NT/9x` 下的 `Regmon` 让我们能够监视注册表相关的读写操作\[SI2\]。此外，售价 `$69` 的 `TCPView pro` 实用工具让我们可以获得系统上所有 `TCP` 和 `UDP` 连接的详细列表，并将这些网络端点与建立连接的进程捆绑。\[WI\]

我们使用了 `Winalysis`，这个程序是 `SFullerton.com` 出品\[SF\]，用于检测该木马造成的系统变化。`Winalysis` 让我们可以创建一个原系统基准状态，并用它和木马运行后的系统状态相比较。配合 `Systernals` 的监视工具，使用 `Winalysis` 让我们可以通过检测相对地确定该木马对文件系统的影响。`Windows 98/ME` 下的 `Winalysis` 售价每拷贝 `$25`，`Windows NT/2000` 下的每拷贝 `$35`，能够发现对系统注册表和文件系统的改变。与一个更常用的剖解工具 `Tripwire` 相比\[TR\]，`Winalysis` 提供较少的细节和稳健性，但由于它的低价格和容易使用，所以我们还是把它作为一个有用的附加工具。

恶意软件在运行的时候会创建临时文件并在程序退出前删除它们。在这样的情况下，`Winalysis` 不可能报告即将消失的暂时文件的存在，而 `Filemon` 将会报告文件被建立和删除，但是不会恢复文件的内容。为了应付这种可能性，我们使用 `Undelete` 实用工具，可从 `Executive Software` 公司买到，售价约 `$45` \[ES\]。`Undelete` 工具用一个 `Recovery Bin` 代替本地的 `Windows` 的资源回收站，它能够捕获所有被删除的文件，甚至是那些被非 `GUI` 进程删除的文件。在写作本文的时候 `Undelete` 还只有支持 `Windows NT/2000` 的，真遗憾。

我们依赖 `Snort` 监视实验室网络上的流量，这是个免费分发的轻量级侵入检测系统\[SN\]。尽管 `Snort` 典型地是被用于自动检测基于网络的攻击，我们只是利用它内建的嗅探(`sniffing`)能力来获得有关网络通信的细节：我们使用 `snort -v -d` 命令启动该程序，这个命令告诉它进入冗长模式并且获取数据包的数据净荷。因为 `Snort` 可以支持 `Windows` 和 `UNIX` 操作系统，还有它宽大的配置基础，以及基于文本的日志和控制，所以相比其他的网络捕获工具程序，我们更喜欢使用基于 `Snort` 的嗅探器。

`VMware` 带有一个名为 `vmnetsniffer` 的工具，它能在主机操作系统上运行，而且能显示在虚拟机器之间、主机机器与它的客座之间的网络流量。不过，对于详细的剖解分析来说，该程序所提供的信息范围还不够充分。即便使用 `/e` 开关运行，`vmnetsniffer` 也仅仅适当地记录数据包的大小、源的 `IP` 和 `MAC` 地址、传输协议类型、以及 `ARP` 和 `ICMP` 消息类型。这没有 `Snort` 所提供的信息那么详尽，而且最重要的是缺乏数据包的数据净荷。

出于网络监听的目的，`VMware` 虚拟实验室网络可以被看作是基于集线器的，因为当它处于混杂的网络接口模式时，网络上的每个机器都能够看见所有的网络流量。不过，这个规则的例外是主机操作系统只能看到出入它本身的流量。你只有使用 `vmnetsniffer` 工具才能从主机机器上看见所有的虚拟网络流量。为了获得所需要的数据包细节，我们在虚拟 `Linux` 的实验室机器上运行 `Snort` 监视网络，从这个机器上我们可以看见所有的虚拟网络流量。



## 2.3 代码分析

本文前面部分所描述的行为分析集中在恶意软件的外部方面，即它与环境的交互，而没有让我们对该程序内部的工作有足够的了解。我们联合使用一个调试器和一个反汇编器，尝试逆向工程该木马的可执行文件，因为我们没有办法读到它的源程序码。这个过程依赖反汇编器了解该程序的基本结构，进而使用调试器单步调试它研究木马的工作流、并最终取得它运行时内存数据。

我们使用 `DataRescue` 的 `IDA Pro` 反汇编器把该木马反组合成汇编指令。`IDA Pro` 标准版 `[DR1]` 能够从公司网站按 $299 的售价购买。在购买程序之前，你可利用它有限制的评估版本 `[DR2]`，它可以免费使用而且能证明在分析的早期它是十分有用的。`DataRescue` 也提供 `IDA Pro` 免费软件 `[DR3]`，实际上那是该软件的 `3.85B` 版本、没有基于 `Windows` 的 GUI，而我们发现图形化用户界面是非常有用的，这只有在该程序更新的版本中才有。在我们的分析过程中，由于需要解释我们不熟悉的的汇编指令，我们还要依靠 `英特尔体系结构软件开发者手册`(the Intel Architecture Software Developer's Manual) `[IN]`。

因为汇编是一个低级语言，若不使用 `SoftICE` 单步调试该木马，我们难以理解其代码流；`SoftICE` 是一个强大的调试器 `[NM]`，它是适用于 `Windows 9x/NT/2000` 的 `NuMega SoftICE Driver Suite` 的组成部分，套装售价 $999。我们让这个调试器在实验室系统的后台作业中运行，并且在同一个系统中启动木马。从该木马的反汇编代码和 `Systernals` 截取的系统调用、注册表调用，我们了解了它的总体结构，所以知道怎样在看上去特别重要的代码段中设置 `SoftICE` 断点。断点使得我们可以在特定的工作流分支上自动呼出调试器，不必单步调试该木马程序中的每一条指令。

`SoftICE` 一旦启动，它就在后台作业中运行，直到按下 `Ctrl-D` 组合键把它呼出或者直到某程序触发一个先前设置的断点 `[C4N]`。我们通常通过提供一个 API 函数名或指令地址作为参数，在 `SoftICE` 命令行执行 `bpx` 命令设置断点。我们在 `SoftICE` 中单步调试程序 `[MT]`：按一次 `F10` 键前进一步，这里把执行函数调用作为一个单步；按 `F8` 键则把每一条指令作为一个单独步执行。最后，`d` 命令也是我们的最常使用的 `SoftICE` 命令之一，通过直接指定一个特别的地址，或者储存在某个特定寄存器的值指示一个存储单元，该命令能显示那里的数据内容。

我们在 `Windows NT 4.0` 和 `Windows 98` 实验室机器上安装了 `SoftICE`。最初，我们担心 `SoftICE` 在 `VMware` 环境中运行的稳定性。尤其当 `SoftICE` 被启动的时候，虚拟机器有时会当机而无法启动。不过，我们从这些虚拟机器上移除 `VMware` 视频驱动程序，并安装 `16` 色 `640x480` 模式的 `Windows` 标准的 `VGA` 驱动程序，大部分的问题立刻就不存在了。在 `Windows NT` 下我们使用 `net start ntice` 命令手工启动 `SoftICE`。在 `Windows 98` 下我们试图手工启动 `SoftICE` 时遇到了问题，后来我们让系统在引导后自动启动 `SoftICE`。

我们的装备中另一个有用的工具是 `Strings` 程序，在大多数 `UNIX` 分发版本中可以找到它。这个实用工具能够从可执行文件里面提取文本字符串，这对于评估一个程序的目的和结构常常是有帮助的。恶意软件的字符串快照比起它的反汇编代码完全列表要短得相当多，当然它也就不那么完全。在 `Windows` 上实现相似功能的程序是 `BinText` 程序，它是由 `Foundstone` 免费分发的 `[FS]`。比起它在 `UNIX` 中的另一选择，`BinText` 更灵活一点而且它支持多种高级过滤选项。

最后，为了自动化与分析有关的一些小任务，我们使用 `Perl` 作为脚本引擎。例如，我们能够使用 `Perl` 实现一个灵活的例程根据该木马的汇编代码建造解密片断的模型。大多数 `UNIX` 发布版本内包括了 `Perl`，而且从 `ActiveState` 公司可以免费得到适于 `Windows` 平台的 `Perl` 版本，这个分发版本贴上了 `ActivePerl` 标签。 `[AS]`

# 第三部分 特洛伊木马的结构

## 3.1 本地系统交互

在我们的 `Windows NT 4.0` 实验室机器上启动 `srvcp.exe` 这个木马之前，我们启用所拥有的全部监视工具。我们把 `srvcp.exe` 这个文件放在本地文件系统中任意选择的位置上并且运行该程序。它安静地进入后台作业，而且除了把 `srvcp.exe` 进程加入进程列表之外，肉眼观察不到它有任何行动。我们可以看到通过监视工具程序记录下来的、与之有关的活动，然后我们大约在 `10` 秒钟之后杀死这个 `srvcp.exe` 进程。活动记录的检查显示，其行为与事件邮件列表上“未知木马”这个思路(thread)里讨论的相一致，具体情况我们在‘背景信息’小节中已详述。

在该木马被杀死之后，我们使用 `Winalysis` 扫描系统中文件系统和注册表的变化。该程序标示出涉及正常 `Windows` 活动的几个修改，并明确指出键值 `srvcp.exe` 创建了这个注册表键： `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run\Service Profiler` 。该配置使得每次 `Windows` 启动时系统都将尝试启动该木马。注意，它没有指定该木马的完全路径，这表明木马作者假定 `srvcp.exe` 会在 (环境变量) `path` 的某个路径中。间接说明我们没有掌握该木马的分布机制，或许它已经修改了系统路径，或许它把 `srvcp.exe` 拷贝到 `path` 默认的某个目录里，比如 `C:\WINNT`。我们的案例中，这个可执行文件仍保留在 `C:\Download` 中，而且没有新的文件被创建，这意谓该木马不会在系统引导后启动。`Filemon` 日志和 `Regmon` 日志证实了 `Winalysis` 的调查结果。

下面图 3-1 举例说明 `Regmon` 记录下该木马创建它的注册表键。(为了突出，我们在屏幕截图上手工标明有关的行)。后来实验证实该木马每一次运行时设置这个键，即使它早先已经被创建。注意，`Regmon` 不显示该注册表键的数值–为了获得这个信息，我们必须使用 `regedit.exe` 工具查看注册表或者使用 `Winalysis` 比较系统状态。据 `Regmon` 记录，该木马也查询注册表的 `\\Services\\WinSock2` 键和 `Services\\Tcp` 键，这是可执行文件利用 `TCP/IP` 进行网络通信的典型动作。

![](img/19c935b6218ca8b754f66d66430d5842.jpg)

图 3-1

如下面图 3-2 所示，`Filemon` 报告该木马试图存取 `C:\WINNT\System32\gus.ini` 文件，但在这个系统上没有找到该文件。在 `Windows NT` 下这个请求使用带 “Open” 选项的 “IRP_MJ_CREATE” 调用，而稍后在 `Windows 98` 下 `Filemon` 只是把该请求标示为 “Read”。即使没有找到这个文件，在这两个系统中该木马都会继续运行。本文稍后我们将讨论 `gus.ini` 文件充当什么角色。而且，`Filemon` 捕获到对 `C:\WINNT\System32\crtdll.dll` 的存取尝试，这个 DLL 提供诸如 `fopen`、`fclose` 和 `fscanf` 等函数调用。此外，该木马读取 `msafd.dll`、`wshtcpip.dll` 和 `rnr20.dll`。最后，该木马读主机文件，或许这是我们曾经使用 `Snort` 看到的域名(domain name)解析过程的一部分，见下一节讨论。

![](img/b42bfd2ae803e5d9725d0402eb8d4909.jpg)

图 3-2

## 3.2 通信协议

我们在某一个实验室机器上启动该木马之前，先配置 `Linux` 机器让 `Snort` 以冗长模式以监视网络通信。因为 `VMware` 虚拟网络有类似集线器的特性，所以这个系统能够捕获实验室网络上传送的所有数据包。下面图 3-3 举例说明，由运行 `srvcp.exe` 的 `Windows NT` 机器发出的一个域名服务(Domain Name Service, DNS)请求。我们配置 `Windows NT` 机器，让它把 `VMware` 主机系统 `172.16.198.1` 作为 DNS 服务器，尽管主机系统并没有运行 DNS 软件。发现试图解析 `irc.mcs.net`，这与 `Filemon` 报告的行为相一致，`Filemon` 记录了该木马要读本地主机文件的请求。

| **Domain Name Resolution Request**03/16-06:19:46.414790 172.16.198.131:1046 -> 172.16.198.1:53UDP TTL:128 TOS:0x0 ID:4354 IpLen:20 DgmLen:57Len: 3700 01 01 00 00 01 00 00 00 00 00 00 03 69 72 63 .............irc03 6D 63 73 03 6E 65 74 00 00 01 00 01 .mcs.net..... |
| :--- |

图 3-3

我们不愿在实验室网络上架设一个 DNS 服务器，而是添加一个项目到受感染机器的主机文件，使之解析 `irc.mcs.net` 为 `172.16.198.129`。这个配置步骤的目的是重定向木马到在我们控制下的一个服务器，以便我们可以观察该程序与它正在解析名字的这个系统的连接的性质。这里在实验室环境下操纵 DNS 记录比较麻烦；万一该木马作者硬编码一个 IP 地址放进程序里面，我们必须配置本地的路由选择表和网络参数才能重定向流量到我们的系统。

如下面图 3-4 所示，该木马尝试连接到它认为是 `irc.mcs.net` 的那个系统上 `6667` 端口。这不奇怪，因为端口 `6667` 通常用于 `IRC` 通信，这与 `irc.mcs.net` 的主机名字和性质相一致，它是一个常用的 `EFnet` 网 `IRC` 服务器。注意，因为我们的服务器没有在端口 `6667` 上监听，所以我们的服务器用一个 `RST` 数据包回应该木马请求连接的 `SYN` 数据包。

| **Failed IRC Connection Attempt**03/16-06:44:10.522728 172.16.198.131:1060 -> 172.16.198.129:6667TCP TTL:128 TOS:0x0 ID:47106 IpLen:20 DgmLen:44 DF******S* Seq: 0x2D492 Ack: 0x0 Win: 0x2000 TcpLen: 24TCP Options (1) => MSS: 146003/16-6:44:10.536857 172.16.198.129:6667 -> 172.16.198.131:1060TCP TTL:255 TOS:0x0 ID:3 IpLen:20 DgmLen:40***A*R** Seq: 0x0 Ack: 0x2D493 Win: 0x0 TcpLen: 20 |
| :--- |

图 3-4

为了响应观察到的行为，我们在 `Linux` 机器 `172.16.198.129` 上安装并配置 `ircd-hybrid` 软件 `[IS]` 给该木马提供 `IRC` 服务。再次启动该木马后，我们能够监视它和我们 `IRC` 服务器的会话。如下面图 3-5 所示，该木马试图使用昵称 “mikey” 登录到 `IRC` 频道 “#daFuck”。在这里，“JOIN” 命令前面的句点等于十六进制表示 “0A”，这个字符表示新行。

| **Successful IRC Connection**03/16-07:59:47.630767 172.16.198.131:1030 -> 172.16.198.129:6667TCP TTL:128 TOS:0x0 ID:18433 IpLen:20 DgmLen:102 DF***AP*** Seq: 0x11B0D Ack: 0xB1B0E8B2 Win: 0x2238 TcpLen: 204E 49 43 4B 20 3A 6D 69 6B 65 79 0A 55 53 45 52 NICK :mikey.USER20 55 79 58 63 20 55 79 58 63 20 55 79 58 63 20 UyXc UyXc UyXc.3A 66 69 67 68 74 20 6D 65 2C 20 70 75 73 73 79 :fight me, pussy0A 4A 4F 49 4E 20 23 64 61 46 75 63 6B 0A .JOIN #daFuck. |
| :--- |

图 3-5

该木马连接到 `IRC` 的方式稍微有点不同于早先在本文‘背景信息’小节中所讨论的那些报告。尤其，`Jeremy` 报告，在那个频道上该木马的真实名字是 “Im trojaned”，而在我们这里其真实名字被设置为 “fight me, pussy”。这两个实例中这些名字都暗示滥用，而且有可能我们现在看到的是该程序的一个变异版本。此外，`Doug` 观察到该木马加入 “#mikag” 频道。我们这里发现在 `Doug` 的频道名字 “#mikag” 和我们的昵称 “mikey” 之间可能有某种联系，尽管我们的样品加入一个不同频道。在本文中将做更多讨论，稍后的分析在频道名字和 `gus.ini` 文件的内容之间建立一个联系。

从该木马连接 `IRC` 服务器后的一些数据包当中，我们注意到从服务器向运行该木马的工作站发出的一个识别请求。识别机制典型地被 `UNIX` 系统用来获取初始化 `TCP` 连接的用户名字。在受感染机器上运行的 `TCPView` 报告，该木马正在监听典型地被识别服务使用的 `TCP` 端口 `113`，下面图 3-6 中显示该程序的识别响应。在这个交换之后，该木马登录到 `IRC` 服务器内，在我们的 `IRC` 服务器上运行 `/who #daFuck` 命令得到如下回应：

“#daFuck mikey H@ CaTiRk@172.16.198.131 (fight me, pussy)”

| **Trojan’s Ident Response**03/16-07:59:47.701560 172.16.198.131:113 -> 172.16.198.129:1078TCP TTL:128 TOS:0x0 ID:18945 IpLen:20 DgmLen:78 DF***AP*** Seq: 0x11B21 Ack: 0xB1EAA1B9 Win: 0x222B TcpLen: 2031 30 33 30 20 2C 20 36 36 36 37 20 3A 20 55 53 1030 , 6667 : US45 52 49 44 20 3A 20 55 4E 49 58 20 3A 20 43 61 ERID : UNIX : Ca54 69 52 6B 0D 0A TiRk.. |
| :--- |

图 3-6

该木马内建立识别功能，极有可能是为了提高与 `IRC` 服务器连接的成功可能性，因为许多 `IRC` 服务器不接受那些不能通过识别机制确认身份的连接。一旦该木马回答了识别请求，它停止在 `TCP` 端口 `113` 监听，或许是为了逃避端口扫描器的远程检测，以及简化程序代码流。我们重复启动该木马，看到它以伪随机方式生成用户名经由识别机制提交。例如，生成的其他名字有 “MdSxJy”、“HsSdLhG”、“IyKw”、“FgX” 和 “Ce”。加入需要的频道之后，该木马以大约每 `3` 秒钟的间隔连续地发出 “NICK mikey” 命令。见下面图 3-7 中所示。(为了表示清楚，图中没有包括来自 `IRC` 服务器的对应的 `ACK` 数据包)。

| **Trojan’s Nickname Requests**03/16-09:01:30.651137 172.16.198.131:1030 -> 172.16.198.129:6667TCP TTL:128 TOS:0x0 ID:21761 IpLen:20 DgmLen:51 DF***AP*** Seq: 0x11B79 Ack: 0xB1B0EE89 Win: 0x2238 TcpLen: 204E 49 43 4B 20 6D 69 6B 65 79 0A NICK mikey. 03/16-09:01:33.818259 172.16.198.131:1030 -> 172.16.198.129:6667TCP TTL:128 TOS:0x0 ID:22017 IpLen:20 DgmLen:51 DF***AP*** Seq: 0x11B84 Ack: 0xB1B0EE89 Win: 0x2238 TcpLen: 204E 49 43 4B 20 6D 69 6B 65 79 0A NICK mikey. |
| :--- |

图 3-7

一旦该木马加入 `IRC` 频道，它保持连接，大概是等待它的操作者经由交谈会话发来的命令。这些通信的性质将在本文的‘代码分析’小节中进一步讨论。该程序也参与 `IRC` 协议所定义的周期性 “PING”-“PONG” 消息交换，这用来确定 `IRC` 客户端是活动的。

我们在一个实验中同时启动该木马两个实例，一个在 `Windows NT` 实验室机器上运行，而另一个在 `Windows 98` 系统上运行。连接到 `IRC` 服务器的第一个实例获得昵称 “mikey”。然而，当第二个实例连接的时候，它没有被允许使用相同的名字，因为 `IRC` 协议要求在相同的 `IRC` 网络上昵称是唯一的。如下面图 3-8 示例，这迫使该木马以伪随机方式为自己生成一个不同的昵称。(在这个网络跟踪记录里，“irc.ticklabs.com” 这个名称是我们分配给 `Linux` 实验室机器的主机名)。在这个情形下，该木马的两个实例都继续通过每隔 `3` 秒发出 “NICK mikey” 命令试图获得相同的昵称。



### 3.3 程序代码

#### Nickname Conflict Resolution

`03/16-09:19:48.633843 172.16.198.129:6667 -> 172.16.198.130:1025`
`TCP TTL:64 TOS:0x0 ID:233 IpLen:20 DgmLen:147 DF***AP*** Seq: 0x3FCD1E87 Ack: 0x301A2 Win: 0x7D78 TcpLen: 20`
`3A 69 72 63 2E 74 69 63 6B 6C 61 62 73 2E 63 6F :irc.ticklabs.co`
`6D 20 34 33 33 20 2A 20 6D 69 6B 65 79 20 3A 4E m 433 * mikey :N`
`69 63 6B 6E 61 6D 65 20 69 73 20 61 6C 72 65 61 ickname is alrea`
`64 79 20 69 6E 20 75 73 65 2E 0D 0A 3A 69 72 63 dy in use...:irc`
`2E 74 69 63 6B 6C 61 62 73 2E 63 6F 6D 20 34 35 .ticklabs.com 45`
`31 20 2A 20 4A 4F 49 4E 20 3A 52 65 67 69 73 74 31 * JOIN :Regist`
`65 72 20 66 69 72 73 74 2E 0D 0A er first...`

`03/16-09:19:48.651467 172.16.198.130:1025 -> 172.16.198.129:6667`
`TCP TTL:128 TOS:0x0 ID:7936 IpLen:20 DgmLen:61 DF***AP*** Seq: 0x301A2 Ack: 0x3FCD1EF2 Win: 0x2128 TcpLen: 20`
`4E 49 43 4B 20 59 76 0A 4A 4F 49 4E 20 23 64 61 NICK Yv.JOIN #da`
`46 75 63 6B 0A Fuck.`

图 3-8

该木马似乎有意地确保至少多个实例之一持有“mikey”这个名字。如果持有该名字的那个木马实例从 IRC 服务器断开连接，程序的另一个实例最多会在 3 秒钟内获取这个名字。该木马连接上网的实例越多，它们当中被冠于“mikey”之名的可能性就越大。`Doug Kahler` 在他与我们联系的 e-mail 中建议，该木马的目的之一可能是为它的操作者保留昵称或者阻止有人得到它。可能该程序的作者能够通过 IRC 消息与它通信、命令该木马释放这个名字以便那个人可能获得它。

我们有一个实验针对，检验在一个潜在攻击者和该木马多个实例之间通信的性质。IRC 是用来中心控制一大群分布式攻击代理的一个极好的频道，这是该木马比如 `srvcp.exe` 经常被人猜测具有分布式拒绝服务能力的原因之一。通过在 IRC 频道上发出一个单一命令，攻击者能够与该木马多个实例通信，由该服务器决定转播消息给已连接上网的木马。IRC 通信的这个性质使得很难追溯攻击来源。此外，IRC 提供通过私人消息单独与该木马每个实例沟通的能力。

实现一对多和一对一的 IRC 消息都是使用下层 IRC 协议的“`PRIVMSG`”命令，如下面图 3-9 所示范。(为了表示清楚，我们从跟踪记录里删掉无关数据包)。开始两个数据包携带一个消息有意让所有的频道参与者见到；发送这个消息只要在 IRC 客户端提示符下输入“hi all”。我们的昵称被设置为“attacker”，我们的真实名字是“lzeltser”，而且我们从“`127.0.0.1`”被连接。可以看到该服务器单独发送这个消息给该木马的每个实例，它们藉由不同的目的地 IP 地址被识别。第三个数据包携带一个消息预定只被持有昵称“mikey”的那个木马实例见到；在 IRC 客户端提示符下输入“`/msg mikey just for you`”发送这个信息。对于这两种情况，我们都不能够通过发送 IRC 消息给该木马而引起它的任何响应，稍后对程序代码的分析揭示了，需要以适当的方式加密命令才能让该木马理解它们。

#### Relaying Messages to IRC Clients

`03/16-09:43:15.650781 172.16.198.129:6667 -> 172.16.198.130:1025`
`TCP TTL:64 TOS:0x0 ID:962 IpLen:20 DgmLen:94 DF***AP*** Seq: 0x46C1D0D2 Ack: 0x404F7 Win: 0x7D78 TcpLen: 20`
`3A 61 74 74 61 63 6B 65 72 21 6C 7A 65 6C 74 73 :attacker!lzelts`
`65 72 40 31 32 37 2E 30 2E 30 2E 31 20 50 52 49 er@127.0.0.1 PRI`
`56 4D 53 47 20 23 64 61 46 75 63 6B 20 3A 68 69 VMSG #daFuck :hi`
`20 61 6C 6C 0D 0A all..`

`03/16-09:43:15.651144 172.16.198.129:6667 -> 172.16.198.131:1030`
`TCP TTL:64 TOS:0x0 ID:963 IpLen:20 DgmLen:94 DF***AP*** Seq: 0x3498DF73 Ack: 0xFF37 Win: 0x7D78 TcpLen: 20`
`3A 61 74 74 61 63 6B 65 72 21 6C 7A 65 6C 74 73 :attacker!lzelts`
`65 72 40 31 32 37 2E 30 2E 30 2E 31 20 50 52 49 er@127.0.0.1 PRI`
`56 4D 53 47 20 23 64 61 46 75 63 6B 20 3A 68 69 VMSG #daFuck :hi`
`20 61 6C 6C 0D 0A all..`

`03/16-09:43:49.946018 172.16.198.129:6667 -> 172.16.198.131:1030`
`TCP TTL:64 TOS:0x0 ID:989 IpLen:20 DgmLen:98 DF***AP*** Seq: 0x3498DFE1 Ack: 0xFFB0 Win: 0x7D78 TcpLen: 20`
`3A 61 74 74 61 63 6B 65 72 21 6C 7A 65 6C 74 73 :attacker!lzelts`
`65 72 40 31 32 37 2E 30 2E 30 2E 31 20 50 52 49 er@127.0.0.1 PRI`
`56 4D 53 47 20 6D 69 6B 65 79 20 3A 6A 75 73 74 VMSG mikey :just`
`20 66 6F 72 20 79 6F 75 0D 0A for you..`

图 3-9

本文前面说过，该木马试图在启动后从系统目录读 `gus.ini` 文件。当我们把 `gus.ini` 文件放进系统目录内的时候，`Filemon` 显示该木马成功地打开并读文件。使用正运行在 Linux 实验室机器上的 `Snort`，我们能够观察到该木马尝试连接到 IRC 服务器上的 `TCP` 端口 `6666`，而非先前使用的 `TCP` 端口 `6667`。因为我们的服务器没有在这个端口上监听，所以这个连接尝试失败。如下面图 3-10 所示，我们观察到该木马随后尝试解析一些 IRC 服务器的主机名字。(为了表示清楚，我们从跟踪记录上删掉 `TCP` 重试数据包)。该木马继续尝试解析其他一些似乎与 IRC 有关的主机名直到我们杀死该进程。

#### Resolving IRC Server Names

`03/16-10:29:11.319877 172.16.198.131:1040 -> 172.16.198.1:53`
`UDP TTL:128 TOS:0x0 ID:63756 IpLen:20 DgmLen:61`
`Len: 41`
`00 01 01 00 00 01 00 00 00 00 00 00 05 65 66 6E .............efn`
`65 74 02 63 73 03 68 75 74 02 66 69 00 00 01 00 et.cs.hut.fi....`
`01 .`

`03/16-10:29:44.423230 172.16.198.131:1042 -> 172.16.198.1:53`
`UDP TTL:128 TOS:0x0 ID:1037 IpLen:20 DgmLen:63`
`Len: 43`
`00 03 01 00 00 01 00 00 00 00 00 00 05 65 66 6E .............efn`
`65 74 05 64 65 6D 6F 6E 02 63 6F 02 75 6B 00 00 et.demon.co.uk..`
`01 00 1 ...`

`03/16-10:30:15.667072 172.16.198.131:1044 -> 172.16.198.1:53`
`UDP TTL:128 TOS:0x0 ID:3341 IpLen:20 DgmLen:64`
`Len: 44`
`00 05 01 00 00 01 00 00 00 00 00 00 03 69 72 63 .............irc`
`0A 63 6F 6E 63 65 6E 74 72 69 63 03 6E 65 74 00 .concentric.net.`
`00 01 00 1 ....`

图 3-10

为了响应观察到的行为，我们在我们的服务器上配置 IRC 精灵(daemon)以监听 `TCP` 端口 `6666`。一旦该木马被重新启动，`Snort` 日志显示该程序连接到 IRC 服务器 `TCP` 端口 `6666`。然而，如下面图 3-11 示例，该木马现在使用键“`soup`”加入频道“`#mikag`”。我们还来不及使该木马可用 `gus.ini`，它就使用了一个不同的频道名字，而且没有提供频道键。(在 IRC 上这个键有时被用来限制对一个频道的存取)。事实上，该木马现在的行为与 `Doug` 报告的观察结果相符，这在本文的‘背景信息’小节中先已描述过。

#### Connecting to a Different IRC Channel

`03/16-11:07:12.816149 172.16.198.131:1032 -> 172.16.198.129:6666`
`TCP TTL:128 TOS:0x0 ID:34304 IpLen:20 DgmLen:112 DF***AP*** Seq: 0xF46A Ack: 0x28E76CC2 Win: 0x2238 TcpLen: 20`
`4E 49 43 4B 20 3A 6D 69 6B 65 79 0A 55 53 45 52 NICK :mikey.USER`
`20 49 66 4A 64 43 6E 20 49 66 4A 64 43 6E 20 49 IfJdCn IfJdCn I`
`66 4A 64 43 6E 20 3A 66 69 67 68 74 20 6D 65 2C 66JdCn :fight me,`
`20 70 75 73 73 79 0A 4A 4F 49 4E 20 23 6D 69 6B  pussy.JOIN #mik`
`61 67 20 73 6F 75 70 0A ag soup.`

图 3-11

在环球网上搜索 `srvcp.exe` 这个木马相关信息的时候，我们偶然看到 `Joe Abrams` 的一篇文章，其中他分析了该木马某个变种的几个代码段。`Joe` 指出在该木马可执行文件中嵌入的许多加密字符串而且叙述了一个解密方法，这些字符串是用一个基于异或(`XOR`)的算法加密的\[`JA1`\]。虽然在我们这个木马版本的可执行文件中没有发现该文提到的大部份字符串，但是 `Joe` 解密的字符串中有一个文本值明显地是“`gus.ini`”。此外有个解密后值为“`joeblow`”，这间接说明 `Joe` 的木马版本与本文‘背景信息’中施密特报告提到的那个相似。

为了弄清解密算法，以便可以解密在我们的 `srvcp.exe` 拷贝中嵌入的字符串，我们把这个可执行文件装入 `IDA Pro` 反汇编器。(读者若想查阅该程序的反汇编代码，可在 [`www.zeltser.com`](http://www.zeltser.com/) 上查看我们反汇编这个程序结果的 `Adobe Acrobat` 打印输出)。遵循 `Joe` 的分析，我们搜索字符串“`nhl*pwf`”，据他说这是“`gus.ini`”的加密值。这引领我们来到一个代码段，见下面图 3-12 中所示。在这个段中该程序反复地把一个好象已加密的字符串推入栈，而且调用相同的例程，这暗示“`sub_4012C6`”可能是一个解密例程。`Joe` 也描述了这个过程，尽管他的数据偏移量与我们的不相符，或许这是由于 `srvcp.exe` 可执行文件细微的版本差异。

#### Probable Calls to String Decryption Routine

```
.text:0040141D   push   offset aNhlPwf   ; "nhl*pwf"
.text:00401422   call   sub_4012C6
.text:00401427   push   offset aAhkl     ; "|ahkli"
.text:0040142C   call   sub_4012C6
.text:00401431   push   offset aWtwgr    ; "wtwgr"
.text:00401436   call   sub_4012C6
.text:0040143B   push   offset aCdkk     ; "|cdkk"
.text:00401440   call   sub_4012C6
.text:00401445   push   offset aMfqece  ; "mfqEce"
.text:0040144A   call   sub_4012C6
```

图 3-12

遵循 `Joe` 对解密过程的分析并在 `IDA Pro` 中查看它的汇编码，我们可以创建一个基于 `Perl` 的例程来模仿该木马代码的相应部分。汇编例程在 `IDA Pro` 中被标示为“`sub_4012C6`”，而且在偏移量“`text:004012C6`”处开始。下面图 3-13 给出我们的解密嵌入字符串 `Perl` 例程。该例程取得加密字符串长度，然后从加密字符串右边开始倒着迭代异或每个字符的原始值和它的位置值。为确保我们的例程实现适当地工作，其方法之一是解密在我们和 `Joe` 的可执行文件版本中都出现的“`nhl* pwf`”字符串，而且确保结果值是“`gus.ini`”。我们的 `Perl` 例程使用 “`&decryptEmbeddedString($encryptedString)`” 形式调用并且返回解密后的字符串。

#### Decrypting Embedded Strings

```perl
sub decryptEmbeddedString {
my($encrypted) = @_;
my(@encrypted) = split(//, $encrypted);
my($length) = $#encrypted;
my($plain) = "";
my($counter);
for ($counter=0; $counter<=$length; $counter++) {
$plain .= chr(ord($encrypted[$length-$counter]) ^ ($counter+1));
}
return($plain);
}
```

图 3-13

要获得嵌入字符串的明文译文，其方法之一是调用我们 `Perl` 例程处理每一个需要经过该程序“`sub_4012C6`”例程解密的字符串。此外，为了确保我们没有遗漏任何加密字符串，我们剖析了使用基于 `UNIX` 字符串程序的 `srvcp.exe`，而且使用一个 `Perl` 脚本自动尝试解密每个字符串。然后我们查看生成的字符串列表、手工挑选出看起来象是英语的那些。结合运用两种方法得到的解密字符串结果见下面图 3-14。

#### Embedded Strings Decrypted

| Encrypted Value | Decrypted Value |
| :--- | :--- |
| `nhl*pwf` | `gus.ini` |
| `\|ahkl` | `mikey` |
| `Wtwgr` | `Setpr` |
| `\|cdkk` | `Jiggy` |
| `mfqEce` | `daFuck` |
| `~h\`PmfqEce` | `daFuckWhat` |
| `v}~y{\*%mj&qldkg` | `fight me, pussy` |
| `og&teh\*\`ph` | `irc.mcs.ne` |
| `O_ATU@VDE@` | `AGGRESSIVE` |
| `5|3u1v/k-h+i)j'g%JLQH` | `ISON a b c d e f g h` |

图 3-14



我们解密的嵌入字符串有些已经在我们的分析中见到，本文前面章节已讨论过。特别地，`gus.ini`是文件名，该木马尝试在启动后用这个初始化设置文件改变它某些方面的行为。`mikey`字符串相配在连接到一个 IRC 服务器时使用的昵称。`daFuck`字符串加上前缀`#`，是该木马所参加的 IRC 频道的名字。`fight me, pussy`字符串相配该木马在 IRC 服务器上所具有的 IRC 用户真实名字特性。最后，`irc.mcs.ne`，紧加上`t`字符，就是该木马尝试连接的 IRC 服务器的主机名字；嵌入字符串缺失最后一个字符或许是由于我们的字符串程序不能完全提取它。在我们分析了`gus.ini`文件的内容之后，其他字符串的用途变得更清楚一点。

我们得到的那个`gus.ini`文件拷贝，它使用的加密算法似乎与嵌入`srvcp.exe`的字符串使用的加密算法不一样。基于我们的观察，我们猜测该木马在运行时期间解密这个文件。为了译解这个文件，我们摸索着理解解密过程。在 IDA Pro 中查看该木马的汇编码，我们发现了几个`fopen`函数调用，它典型地是用来存取磁盘上的文件。其中只有一个调用被指定`r`属性，即只读模式打开一个文件，如下面图 3-15 所示。这是程序启动后的第一次`fopen`调用，这个事实结合先前讨论过的 Filemon 日志，暗示我们这是尝试读`gus.ini`文件。在调用`fopen`之前入栈的`arg_0`参数表示将要打开的文件名，在这里它应该是`gus.ini`。

```
Opening gus.ini File
.text:00403662   push   offset aR      ; "r"
.text:00403667   push   [ebp+arg_0]
.text:0040366A   call   fopen
```

图 3-15

为了确保我们对于`gus.ini`文件如何被打开的理解是正确的，我们使用 SoftICE 调查程序运行时间什么时候`fopen`被调用。实现如下，启动 SoftICE，然后按`Ctrl-D`进入 SoftICE 命令行。在命令提示符上我们输入`bpx CRTDLL!fopen`命令，在`fopen`函数被调用处设置断点。然后我们在命令行上输入`x`命令把 SoftICE 置于后台作业，而且启动`srvcp.exe`可执行文件。过 1 秒种左右，SoftICE 拦截了一个对`fopen`的调用而且中断该程序的运行，把我们带回 SoftICE 命令行。然后我们用`F10`键单步调试一些汇编指令，从`fopen`函数返回。下面图 3-16 显示出该木马刚执行它的第一次`fopen`调用不久时 SoftICE 屏幕一部分。我们使用`d *(EBP+08)`命令显示`fopen`第一个参数的内存内容，和我们期望的一样，它是`C:\WINNT\System32\gus.ini`。

![](img/2ee85d29fd9be39290bef82e83362437.jpg)

图 3-16

在 IDA Pro 中查看该木马的汇编码，我们看到唯一的一个对`fscanf`函数的调用带有参数`%[^\\n]\\n`，如下面图 3-17 所示。这是从文件读入一个整行的方式之一，也是该木马逐行读`gus.ini`文件的方法。继续在程序中探索，看到这个可执行文件使用`jnz loc_4036B2`指令跳转到另一个代码段，之后内存被清除而且文件句柄关闭。这暗示文件内容是由位于偏移 4036B2 的代码进行处理。

```
Reading Lines from gus.ini File
.text:00403738   push   eax
.text:00403739   push   offset asc_4083D1    ; "%[^\\n]\\n"
.text:0040373E   push   ebx
.text:0040373F   call   fscanf
.text:00403744   add    esp, 0Ch
.text:00403747   cmp    eax, 0FFFFFFFFh
.text:0040374A   jnz    loc_4036B2
```

图 3-17

该程序在 4036B2 偏移后面几行那里带`%[^=]=%[^`参数调用`sscanf`函数，通常用来解析字符串。此字串模式常常表示典型的.ini 文件格式，即各行遵循`PARAMNAME=value`这种约定。因为加密的`gus.ini`文件没有遵循标准的.ini 文件格式，即没有用等号分开参数名字和数值，所以该程序一定是在调用`sscanf`之前解密文件的每一行。如下面图 3-18 所示，在使用`fscanf`逐行读入并使用`sscanf`解析它之后，唯一被调用的例程是`sub_405366`，它或许是解密例程。

```
Parsing gus.ini Lines
.text:004036B2   lea    eax, [ebp+var_414]
.text:004036B8   push   eax
.text:004036B9   push   esi
.text:004036BA   call   sub_405366
.text:004036BF   mov    edi, eax
.text:004036C1   lea    eax, [ebp+var_9F0]
.text:004036C7   push   eax
.text:004036C8   lea    eax, [ebp+var_14]
.text:004036CB   push   eax
.text:004036CC   push   offset asc_4083C5    ; "%[^=]=%[^"
.text:004036D1   push   edi
.text:004036D2   call   sscanf
```

图 3-18

为了调查在解密例程被调用时该木马的运行时环境，我们重新启动`srvcp.exe`进程，让 SoftICE 保留前面提到的`CRTDLL!fopen`断点。一旦 SoftICE 拦截到调用，我们就在 SoftICE 命令行输入`bpx 4036BA`，即在偏移 4036BA 处增加一个断点，这是`sub_405366`被调用的地方。然后我们使用`x`指令把 SoftICE 置于后台作业，并等待直到调试器在我们的断点上中断该木马。设置第二断点之前等待第一次`fopen`调用，这是为了让 SoftICE 参照该木马运行时栈为`sub_405366`调用的相对偏移计算绝对偏移。

在该程序调用解密例程之前它的状态见下面图 3-19 中所示。在 SoftICE 屏幕这个部分我们观察到这个可执行文件在调用`sub_405366`之前把两个数值推入栈，这跟先前 IDA Pro 中见到的一样。`EAX`寄存器指向的内存单元中的数值是`JexO215WuK60H7HgI.j11vh1`，这实际上是加密的`gus.ini`文件第一行。我们可以通过在 SoftICE 执行`d EAX`命令检视这些存储器内容。用类似的方法，我们查看了`ESI`寄存器所关联的数值，而且确定该值被设置为`EcbJer8\0dx.CJVJsAlmIZ`(注意，中间的 null 字符在这里表示为`\0`) 。第二个字符串的性质和它在解密过程中扮演的角色稍后在我们的分析过程中变得更清楚。

![](img/1e6c6fff2c2803246b4c88f7fd56a7fb.jpg)

图 3-19 我们在 SoftICE 中用`F10`键让该木马执行`sub_405366`调用，之后我们再查看`EAX`寄存器关联的存储器内容。如下面图 3-20 所示，这个内存单元包含`NICK=mikey`，似乎是该木马处理`gus.ini`得出的一行已解密译文。

![](img/1e6c6fff2c2803246b4c88f7fd56a7fb.jpg)

图 3-20

`gus.ini`中的其他行经这个过程处理后，我们证实了在`sub_405366`例程中的确进行文件解密。我们也能够通过监视`EAX`寄存器指向的存储器内容，在调用解密例程之后获得`gus.ini`文件的已解密译文，而不需要知道实际的解密算法。解密后`gus.ini`文件的内容见于下面图 3-21 中。为了更加清楚，我们精简了文件的内容。

```
Decrypted gus.ini File
NICK=mikey
MODE=AGGRESSIVE
SETCOMMAND=setpr
COMMAND=fuckedup
CHANNEL=mikag soup
SOUPCHANNEL=alphasoup ah
SERVER0=irc.mcs.net:6666
SERVER1=efnet.cs.hut.fi:6666
SERVER2=efnet.demon.co.uk:6666
SERVER3=irc.concentric.net:6666
SERVER4=irc.etsmtl.ca:6666
SERVER5=irc.fasti.net:6666
... *cut for brevity *...
```

图 3-21

`gus.ini`的内容似乎要覆盖在编译时间嵌入`srvcp.exe`内的缺省值。这个文件的解密译本除了提供参数数值之外也提供参数名字，这为我们了解以前见过的字符串的用途提供了有关线索。我们已经知道`NICK`参数定义了连接到 IRC 服务器时使用的昵称。虽然我们的`gus.ini`文件把这个名字设置为`mikey`，但是该木马的操作者似乎能够通过操纵`gus.ini`文件的内容把它设置为一个任意值。`CHANNEL`参数定义该木马参加频道的名字。我们的`gus.ini`覆盖该程序内建的频道名字，这与我们先前观察到的行为相一致。这个文件也给该木马提供服务器列表和端口数目，当加入一个 IRC 网络的时候，该程序将会使用它们，这些值覆盖缺省值\----在`irc.mcs.net`上的端口 6667。我们的`gus.ini`文件定义了 34 个这样的服务器，你可以从`www.zeltser.com/`下载这个文件的解密译本。这些主机名字与观察到的 DNS 解析请求相一致，在本文‘通信协议’小节中讨论过。

据参数名字判断，该木马能够加入一个替代频道，这是由`gus.ini`中`SOUPCHANNEL`参数定义了。这个参数的值与`CHANNEL`参数所定义的那个值是通过一个公共子串`soup`发生关系。内建的频道名字`daFuck`和具有`daFuckWhat`值的未知内建参数也是通过一个公共子串产生关联，这暗示`daFuckWhat`是内建的替代频道名字。在写作本文的时候我们还不清楚关于替代频道的使用细节。

在`gus.ini`中`SETCOMMAND`参数定义的值匹配这个可执行文件内嵌入的`setpr`字符串。这似乎是一个关键字\----在 IRC 上认证该木马所使用的基准。这个假说得到 Joe Abrams 分析的支持，尽管在他的木马版本内嵌入的值与我们的不同\[JA2\]。Joe 提出该木马有若干个访问层次，这表明我们这个木马的第二个口令是由`gus.ini`中的`COMMAND`参数设定，其值为`fuckedup`。我们认为对应这个参数的嵌入缺省值是`jiggy`，请注意它们不一样。

`MODE`参数在`gus.ini`文件中被设置为`AGGRESSIVE`，在本文写作的时候它和可执行文件中加密值的用途还是未知的。这可能是一个控制该木马攻击强度的方式，或者操作者是在 IRC 频道上微调它的行为。我们假定这个参数的替代值是`PASSIVE`，因为在可执行文件中这两个值都是用明文形式定义、在栈中又是紧挨着的。该程序比较这些值的明文译本和邻近代码段的一个变量；这可能是比较`MODE`参数的已解密值和某已知值的过程，而且可能会据此行动。

为了试图了解用来混淆`gus.ini`文件各行的那个算法，我们将注意力转向那传递给解密例程的第二个参数。这个字符串被设置为`EcbJer8\0dx.CJVJsAlmIZ`，而且在解密过程中似乎当作一个键使用，但是它没有在可执行文件中作为明文字符串嵌入。我们发现，这个键被存储为一个字符集合，在栈中从位置 4080C8 到 40811F 紧挨着存放。下面图 3-22 中显示了这个数据栈的最后片断，包括结束的 null 字符。这个键不存储为一个单一字符串，可能是不想被轻易检测到。

```
Fragment of Embedded Decryption Key
... *cut for brevity *...
.data:00408110 db 6Dh ; m
.data:00408111 db 0 ;
.data:00408112 db 0 ;
.data:00408113 db 0 ;
.data:00408114 db 49h ; I
.data:00408115 db 0 ;
.data:00408116 db 0 ;
.data:00408117 db 0 ;
.data:00408118 db 5Ah ; Z
.data:00408119 db 0 ;
.data:0040811A db 0 ;
.data:0040811B db 0 ;
.data:0040811C db 0 ;
.data:0040811D db 0 ;
.data:0040811E db 0 ;
.data:0040811F db 0 ;
```

图 3-22

注意，每个有效字符后跟 3 个 null 字符。在启动后，该木马把键组合成一个单一字符串，实施的代码见下面图 3-23 中所示。该程序这个部分逐字符读入并在内存中相邻字节排列它们。

```
Assembling the Decryption Key
.text:00403698   mov  edx, dword_4080C8[edi*4]
.text:0040369F   mov  [esi+edi], dl
.text:004036A2   inc  edi
.text:004036A3   cmp  edi, 2Ch
.text:004036A6   jb   short loc_403698
.text:004036A8   mov  byte ptr [edi+esi+1], 0
```

图 3-23

为了了解它的算法，我们尝试单步调试该木马的解密例程。虽然时间约束阻止我们完成这个过程，不过我们能够了解这个过程的一些潜在原则。作为解密过程的一部份，该程序使用 null 字符作为分隔符把上述的键分成两个字符串。我们跟随该程序的工作流，它变异了这个键的第一部分拷贝`EcbJer8` \----每个字符原始值加 7。实现这个函数的汇编码见下面图 3-24 中所示。这个代码片断的结果是，这个键的最初部份被转换成`LjiQly?Ck`，跟着的一个不可打印字符十六进制数值表示为`7F`。

```
Mutating the Decryption Subkey
.text:00405401 mov eax, edi
.text:00405403 add eax, [ebp+var_C]
.text:00405406 add byte ptr [eax], 7
.text:00405409 inc edi
.text:0040540A mov eax, [ebp+var_C]
.text:0040540D lea ecx, [eax]
.text:0040540F or eax, 0FFFFFFFFh
.text:00405412 inc eax
.text:00405413 cmp byte ptr [ecx+eax], 0
.text:00405417 jnz short loc_405412
.text:00405419 cmp edi, eax
.text:0040541B jb short loc_405401
```

图 3-24

我们还注意到该木马处理了很多嵌入到可执行文件里面的字符值，在内存中该木马从偏移 4086CC 处开始将它们彼此紧靠着排成一行。在变异那个键的前缀之后不久，就在偏移 405435 处调用例程`sub_404E78`来执行这个任务。此时逻辑变得有点难以跟随，而且我们失去程序工作流的踪迹。若有兴趣追踪我们的步骤，可以在引用的偏移处设置一个断点并且单步调试这个代码。(具体实现办法可以是，首先在 SoftICE 中使用`bpx CRTDLL!fopen`命令在`fopen`调用上中断，然后在需要的偏移处用命令制造一新断点，比如`bpx 405435`)。

在建立环境之后（具体如何设置我们还不太清楚），解密过程继续进行，循环处理加密行开头部分的一个字符子集并且从偏移 405473 处调用例程`sub_40517A`。`sub_40517A`例程从加密行取一个字符，并且返回这个字符在某个特定字符串中的位置索引（这个字符串被嵌入到`srvcp.exe`内的偏移 409718 处）。索引从 0 开始，而这个字符串就是`./0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ`。在`sub_40517A`例程返回之后，使用偏移 405489 处的命令把索引值左移（幅度是变化的），并由位于偏移 40548B 处的`OR`指令屏蔽后放入内存。小幅移位过程一直继续到解密例程`sub_405366`结束，这个例程我们前面已讨论过，它返回`gus.ini`文件行的解密译本。

据 Joe Abrams 说，通过 IRC 发送给该木马的命令也被使用相似的加密技术加了密。命令必须被适当地加密，该程序才能解释它们。Joe 观察到在该木马的 IRC 频道上被使用的一些加密命令，通过重放这些命令，他能够以一种有限方式控制他那个版本的木马。这间接表明，用来加密命令和`gus.ini`文件的算法不考虑与该木马实例相关的特有特性。

为了一个适当加密命令能被该木马兑现，在每个通信尝试中，这个命令需要被加上一个适当的口令作为前缀。这意味着，试图控制该木马的操作者需要以`password command`(口令+命令)形式发送消息给 IRC 频道。就如我们前面讨论的，该程序内至少有两个硬编码的口令。这些口令可以使用`gus.ini`文件中的`SETCOMMAND`和`COMMAND`参数覆盖，这或许给操作者提供不同的访问层次。



如 Joe 在他的报告中所述，这个口令必须使用它自己的算法适当地加密。这个算法通过查验试图控制该程序的主机 IP 地址来保护该木马免于受到重放攻击。Joe 在他的报告中描述了口令加密过程，而且指出有些组成加密口令的字符可能是不可打印的。需要适当地加密口令和命令字符串，这暗示存在着某个控制木马程序在外面操作，或者同时配以标准的 IRC 客户端。

`srvcp.exe` 这个木马提供给攻击者什么功能我们还不清楚。就如我们在本文的‘背景信息’小节中提到的，若干 FTP 相关的字符串嵌入到程序里面，这暗示，它的操作者有能力把文件传入和传出受感染系统。如图 3-25 所示，该木马似乎依赖 Windows 内建的 `ftp.exe` 程序来进行基于 FTP 的文件传输。

> **Embedded FTP Messages**
> ```asm
> .data:004084B9 db 'PRIVMSG %s :successfully spawned ftp.exe',0Ah,0
> .data:004084E3 db 'PRIVMSG %s :couldn',27h,'t spawn ftp.exe',0Ah,0
> ```

图 3-25

在 `srvcp.exe` 内嵌入的几个字符串间接表明，该木马能够进行拒绝服务攻击。如下面图 3-26 所示，“sacker”和“jacker”可能是对这些攻击的引用。注意，“sacker”把时间和端口数目作为参数，这可能让攻击者可以在某一个特定时间段内使用网络流量充斥某一个目标。“spawn”命令可能让攻击者可以在受感染机器上启动任意一个程序。

> **Possible Network Flood Commands**
> ```asm
> .data:00408541 db 'PRIVMSG %s :ctcp <nick> PING 848348, help, '
> .data:00408541 db 'getnick <nick>, getnonick, rnick <nick>!!, '
> .data:00408541 db 'sacker time low\_port high\_port addy, jacker'
> .data:00408541 db ' time ip ip ip etc, stopsack, stopjack, spa'
> .data:00408541 db 'wn filename, ftpget EVERYTHING, randnick, c'
> .data:00408541 db 'lone, clonedie',0Ah,0
> ```

图 3-26

# 第四部分 保护措施

## 4.1 传播机制

我们没有证据证明 `srvcp.exe` 不需要一个外部程序帮助就能够传播或复制。如本文‘本地系统交互’小节所提到的，这个可执行文件假定它位于一个被包含在系统路径中的目录里面，比如 `C:\WINNT`，但是它实际上没有在启动后把自己复制或移动到那些目录。这暗示我们遗漏了实际负责把 `srvcp.exe` 放进适当目录内的另外一个程序。受害者可能通过 e-mail 消息收到一个受感染的载体文件，或者从因特网下载了受感染的程序。

从日期为 2000 年六月、有关 `srvcp.exe` 的一个 HackFix 主页上，我们发现这样的信息：该木马一直是用 `DivX_e3.exe` 程序传播的，该程序声称是 `DivX` 视频译码器一个注册版`[HF]`。另外据传，该木马正通过 `serials.2000.v6.0.zip`、`CDRWin3.8.zip` 和 `PSXCopy.v6.0.zip` 这些文件传播。在本文‘背景信息’小节中提到过，Pete Schmitt 最近提交的一份事件报告证实，该木马正在继续传播；然而，在写作本文的时候我们还不知道目前载体文件的名字。一旦机器被感染，攻击者能够通过该木马的文件传输能力在受害者的系统上安装其他恶意软件。

载体程序是否连同 `srvcp.exe` 一起也安装了一个 `gus.ini` 文件拷贝，这还不能肯定。既然该木马没有 `gus.ini` 也能运转，所以这个文件可能是在最初感染之后被创建的。攻击者能够通过 IRC 文件传输和 FTP 能力上传 `gus.ini` 到受感染的机器。注意，这些连接多数是由受感染的机器初始化，因此它们不大可能被防火墙阻止。也可能是攻击者能够通过 IRC 控制频道发送适当命令指示该木马创建 `gus.ini` 文件。

## 4.2 特洛伊木马的变种

浏览杀毒产品厂商的公众数据库，我们发现有关 `srvcp.exe` 木马变种的许多记录。然而，厂商似乎对于该木马的命名有分歧。`Symantec` 软件公司称之为“IRC.SRVCP.木马”，但是除了描述该木马特征为“wild”没有提供任何关于它的信息`[SY]`。`Trend Micro` 提供该木马的简短描述，其行为特性和文件尺寸与我们木马变种相符；他们称之为“TROJ_SRVCP”`[TM1]`。`Computer Associates` 提供了关于该木马的充分信息，这些信息表明他们变种与本文所述的这个可执行文件紧密地相符；他们称该程序为“Tasmer.B”和“Win32.Tasmer.B”`[CA1]`。

`Computer Associates` 还描述了该木马另一个变种，其可执行文件名为 `tskmngr.exe`，而它的注册表键是“Task Manager”；他们称这个程序为“Tasmer.A”、“Win32.Tasmer.A”、“Backdoor.Tasmer”和“Troj/Narnar”，而且说最早在 2000 年 4 月发现它。`Trend Micro` 也描述了该木马的一个 `tskmngr.exe` 变种，但称它为“TROJ_TASMER.B”`[TM2]`。`Sophos` 也命名 `tskmngr.exe` 变种为“Troj/Narnar”；他们指出，该木马连接一个 DALnet 的 IRC 服务器，这是我们这个木马版本所使用的 EFnet 网络的一个替代网络`[SP]`。`Network Associates` 也讨论了这个变种，称之为“IRC/Randy”`[NA]`。

明显地，该木马至少有两个变种：一个在 EFnet 上操作，另一个在 DALnet 上操作。据 Joe Abrams 说，该木马的 EFnet 版本比在 DALnet 上运转的那个年轻许多。在 Joe 的文章中，凭借对 DALnet 频道上受感染用户的数目判断，EFnet 版木马变种不象 DALnet 版的那么流行。Joe 的木马版本加入 DALnet 上的“#KimmiTheB”频道，该频道由该组的成员控制，自成一“派”。

Joe 也指出，他偶然碰到到过该木马的一个新版本，它被使用 `NeoLite` 工具程序加壳/压缩了。`NeoLite` 封装 Windows 可执行文件及其资源以保护它们不被反编译器逆向工程`[NL]`。被 `NeoLite` 压缩以后的可执行文件启动后自解压到内存里面。绕过这种保护的办法是使用 `SoftICE` 在 `NeoLite` 解密例程上设置一个断点，让程序解压缩自己，然后使用 `SoftICE` 研究内存里的已解密程序。

## 4.3 特洛伊木马的签名

如下面图 4-1 所例，诺顿杀毒软件能自动检测我们的 `srvcp.exe` 木马版本。通过检查受感染机器手动检测该木马也相对容易：在文件系统上查找有无 `srvcp.exe` 可执行文件，检查有无注册表键“HKLM\software\Microsoft\Windows\CurrentVersion\Run\Service Profiler”，并检查在机器的系统目录中有无 `gus.ini` 文件。我们没有发现除了 `srvcp.exe` 之外哪个程序有这三个特征之一，但不排除有假阳性的可能。

![](img/ace1b38a172ecb09b598d4990d331dd2.jpg)

图 4-1

该木马甚至不需要存取它的源程序码就可以不费力地变异，这使得对它的检测更加困难。例如，即使该程序的文件名不是“`srvcp.exe`”，它也会继续运行。这个可执行文件也能用一个普通的文件编辑器编辑，让它改用一个不同的注册表键。类似地，也能修改这个可执行文件让它使用“`gus.ini`”之外的资源文件名。

作为一个观念性证明，我们使用一个明文编辑程序修改那个已编译的可执行文件中的“`gus.ini`”文件名：由于这个文件名以加密格式储存，所以需要搜索字符串“nhl*pwf”。然后我们修改加密字符串中的一个字母，把新字符串变成“nhl*pwg”。使用 `IDA Pro` 反汇编修改后的可执行文件，结果的数据栈见下面图 4-2 中所示。运行该木马的修正版，结果它使用“`fus.ini`”这个名字作为它过去一直称为“`gus.ini`”的文件名。最令人不安的是，用这个简单方式修改可执行文件导致诺顿杀毒软件不再把它当作恶意软件。注意，通过逆向嵌入字符串的解密算法（在本文‘程序代码’小节描述过），我们发现这个算法能够给这个文件构造一个任意的文件名。

> **Modifying Name for Initialization File**
> **Before modification:**
> ```asm
> .data:00408034   db 'nhl\*pwf',0
> ```
> **After modification:**
> ```asm
> .data:00408034   db 'nhl\*pwg',0
> ```

图 4-2

发现该木马的方法之一是检测它的网络通信。例如，在受感染的系统上运行“`netstat -a`”命令将会显示系统在 TCP 端口 113 监听识别请求，即使该木马还没有连接到一个 IRC 服务器。扫描网络查找未经授权地运行识别请求的主机是远程检测这种状态木马的一个可能方法。一旦该木马成功连接到一个 IRC 服务器，“`netstat -a`”可能显示一个到某个外部服务器端口 6667 或 6666 的 TCP 连接。如果该组织的工作站不正常地使用 IRC，这种通信可以用一个基于网络的侵入探测系统(Intrusion Detection System, IDS)检测到。

使用基于网络的 IDS 检测该木马，一个更加可靠的方法是扫描数据包内容、查找与该木马操作有关的字符串。通过在网络的流(stream)中查找“NICK mikey”之类字符串可以检测该木马的一般变种。遗憾的是，这个方法不可能比使用杀毒软件更有效，因为该木马的签名很容易改变。有关该木马活动更可靠的指标是每 3 秒左右出现专一一个 IRC 客户端发出昵称改变请求。遗憾的是，实现这种逻辑可能是资源密集的，因为它需要 IDS 维持跨多个数据包之间关于潜在攻击性流量的状态信息。替代办法是把 IDS 调整为扫描用来操作该木马的密码化命令字符串数据包，因为即使是加密的形式，在该木马的多个实例中这些命令还是保持相同，除非加密算法改变。为了获得这些命令字符串，需要对该木马进行一个更全面的分析。我们鼓励读者(他或她)参与这种探索、并和防卫社区共享调查结果。

# 第五部分 参考书目

`[JLG]` Jeremy L. Gaddis. 事件邮件列表文档. “未知木马(附件).” 8 June 2000. URL: [`www.securityfocus.com/archive/75/64241`](http://www.securityfocus.com/archive/75/64241). 21 March 2001.

`[RFC1459]` Jarkko Oikarinen, Darren Reed. RFC 1459. “Internet 多线交谈协议.” May 1993. URL: [`www.faqs.org/rfcs/rfc1459.html`](http://www.faqs.org/rfcs/rfc1459.html). 21 March 2001.

`[BK]` Brandon Kittler. 事件邮件列表文档. “回复：未知木马(附件).” 10 June 2000. URL: [`www.securityfocus.com/archive/75/64380`](http://www.securityfocus.com/archive/75/64380). 21 March 2001.

`[RFC1413]` Michael St. Johns. RFC 1413. “身份认证协议.” February 1993. URL: [`www.faqs.org/rfcs/rfc1413.html`](http://www.faqs.org/rfcs/rfc1413.html). 21 March 2001.

`[DK]` Doug Kahler. 事件邮件列表文档. “回复：未知木马(附件).” 12 June 2000. URL: [`www.securityfocus.com/archive/75/64849`](http://www.securityfocus.com/archive/75/64849). 21 March 2001.

`[NF]` Nick FitzGerald. 事件邮件列表文档. “回复：未知木马(附件).” 13 June 2000. URL: [`www.securityfocus.com/archive/75/64847`](http://www.securityfocus.com/archive/75/64847). 22 March 2001.

`[PS]` Pete Schmitt. 事件邮件列表文档. “新的 (?) windows irc ddos 木马.” 10 March 2001. URL: [`www.securityfocus.com/archive/75/167985`](http://www.securityfocus.com/archive/75/167985). 22 March 2001.

`[VM1]` VMware, Inc. “VMware 工作站常见问题.” URL: [`www.vmware.com/products/desktop/ws_faqs.html`](http://www.vmware.com/products/desktop/ws_faqs.html). 22 March 2001.

`[VM2]` VMware, Inc. “用于 Windows 2000 主机的主机专用网络配置注意事项.” URL: [`www.vmware.com/support/ws2/doc/hostonly_w2k_ws_win.html`](http://www.vmware.com/support/ws2/doc/hostonly_w2k_ws_win.html). 4 April 2001.

`[RFC1918]` RFC 1918.URL: [`www.faqs.org/rfcs/rfc1918.html`](http://www.faqs.org/rfcs/rfc1918.html). 22 March 2001.

`[SI1]` Mark Russinovich, Bryce Cogswell. 用于 Windows NT/9x 的 Filemon. 26 December 2000. URL: [`www.sysinternals.com/ntw2k/source/filemon.shtml`](http://www.sysinternals.com/ntw2k/source/filemon.shtml). 22 March 2001.

`[SI2]` Mark Russinovich, Bryce Cogswell. 用于 Windows NT/9x 的 Regmon. 7 November 2000. URL: [`www.sysinternals.com/ntw2k/source/regmon.shtml`](http://www.sysinternals.com/ntw2k/source/regmon.shtml). 22 March 2001.

`[WI]` Winternals Software. TCPView 专业版. URL: [`www.winternals.com/products/monitoringtools/tcpviewpro.shtml`](http://www.winternals.com/products/monitoringtools/tcpviewpro.shtml). 22 March 2001.

`[SF]` SFullerton.com. Winalysis 2.50 版. 30 December 2000. URL: [`www.sfullerton.com/products.htm`](http://www.sfullerton.com/products.htm). 22 March 2001.

`[TR]` Tripwire 主页. URL: [`www.tripwire.com`](http://www.tripwire.com/). 4 April 2001.

`[ES]` Executive Software. Undelete 2.0. URL: [`www.execsoft.com/undelete/undelete.asp`](http://www.execsoft.com/undelete/undelete.asp). 22 March 2001.

`[SN]` Martin Roesch. Snort – 开放源码网络侵入检测系统. URL: [`www.snort.org`](http://www.snort.org/). 22 March 2000.

`[DR1]` DataRescue. IDA Pro by Ilfak Guilfanov. URL: [`www.datarescue.com/idabase/idaorder.htm`](http://www.datarescue.com/idabase/idaorder.htm). 22 March 2001.

`[DR2]` DataRescue. IDA Pro 评估版下载. URL: [`www.datarescue.com/idabase/ida4down.htm`](http://www.datarescue.com/idabase/ida4down.htm). 22 March 2001.

`[DR3]` DataRescue. IDA Pro 免费软件. URL: [`www.datarescue.be/downloadfreeware.htm. 22 March 2001`](http://www.datarescue.be/downloadfreeware.htm.%2022%20March%202001).  



`[IN]` Intel 公司. Intel 体系结构软件开发者手册, 第 2 册: 指令集参考手册. 1 January 1997. URL: [`developer.intel.com/design/pentium/manuals/243191.htm`](http://developer.intel.com/design/pentium/manuals/243191.htm). 22 March 2001.

`[NM]` Compuware NuMega. “我还可以单独买 SoftICE 吗？” URL: [`www.numega.com/drivercentral/FAQs/dsq29.shtml`](http://www.numega.com/drivercentral/FAQs/dsq29.shtml). 22 March 2001.

`[C4N]` CoRN2. “#Cracking4Newbies SoftIce 教程.” URL: [`whateverhosting.com/krobar/beginner/04.htm`](http://whateverhosting.com/krobar/beginner/04.htm). 22 March 2001.

`[MT]` “Mammon\_’s Tales to His Grandson” URL: [`newdata.box.sk/neworder/cracking/ice.html`](http://newdata.box.sk/neworder/cracking/ice.html). 22 March 2001.

`[FS]` Foundstone. BinText v3.0. URL: [`www.foundstone.com/rdlabs/proddesc/bintext.html`](http://www.foundstone.com/rdlabs/proddesc/bintext.html). 22 March 2001.

`[AS]` ActiveState 公司. ActivePerl. URL: [`www.activestate.com/Products/ActivePerl`](http://www.activestate.com/Products/ActivePerl). 21 March 2001.

`[IS]` IRCD-Hybrid. URL: [`www.ircd-hybrid.net`](http://www.ircd-hybrid.net/). 4 April 2001.

`[JA1]` Joe Abrams. “逆向一个木马.” 1 October 2001. URL: [`freeshell.org/~abrams/troj.txt`](http://freeshell.org/~abrams/troj.txt). 22 March 2001.

`[JA2]` 在寄存器中篡改数据. Joe Abrams. “逆向一个木马.” 19 November 2001. URL: [`www.hackinthebox.org/article.php?sid=1138`](http://www.hackinthebox.org/article.php?sid=1138). 22March 2001.

`[HF]` HackFix. “srvcp.exe” June 2000. URL: [`www.hackfix.org/ircfix/srvcp.shtml`](http://www.hackfix.org/ircfix/srvcp.shtml). 22 March 2001.

`[SY]` Symantec 病毒百科全书. “IRC.SRVCP.Trojan.” URL: [`www.symantec.com/avcenter/cgi-bin/virauto.cgi?vid=18552`](http://www.symantec.com/avcenter/cgi-bin/virauto.cgi?vid=18552). 22 March 2001.

`[TM1]` Trend Micro 病毒百科全书. “TROJ\_SRVCP.” URL: [`antivirus.com/vinfo/virusencyclo/default5.asp?VName=TROJ\_SRVCP&VSect=T`](http://antivirus.com/vinfo/virusencyclo/default5.asp?VName=TROJ_SRVCP&VSect=T). 4 April 2001.

`[CA1]` Computer Associates 病毒百科全书. “Tasmer.B.” URL: [`ca.com/virusinfo/encyclopedia/descriptions/tasmerb.htm`](http://ca.com/virusinfo/encyclopedia/descriptions/tasmerb.htm). 22 March 2001.

`[TM2]` Trend Micro 病毒百科全书. “TROJ\_TASMER.B” URL: [`antivirus.com/vinfo/virusencyclo/default5.asp?VName=TROJ\_TASMER.B&VSect=T`](http://antivirus.com/vinfo/virusencyclo/default5.asp?VName=TROJ_TASMER.B&VSect=T). 22 March 2001.

`[SP]` Sophos 病毒信息. “Troj/Narnar”. URL: [`www.sophos.com/virusinfo/analyses/trojnarnar.html`](http://www.sophos.com/virusinfo/analyses/trojnarnar.html). 22 March 2001.

`[NA]` Network Associates. “IRC/Randy”. 12 April 2000. URL: [`vil.nai.com/villib/dispVirus.asp?virus\_k=98569&EY=y`](http://vil.nai.com/villib/dispVirus.asp?virus_k=98569&EY=y). 22 March 2001.

`[NL]` NeoWorks. “NeoLite: 为开发者程序加密与压缩.” URL: [`www.neoworx.com/products/neolite/default.asp`](http://www.neoworx.com/products/neolite/default.asp). 4 April 2001.

`4dd466d0e1e0cc123afbe504da2462ca`
