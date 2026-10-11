
# 逆向工程，怎么样和哪里开始？

> 原文：`www.pediy.com/kssd/pediy6969.htm`

大家好，我刚译了一文：“逆向工程，怎么样和哪里开始？”由于我不是程序员，很多专业词不懂，请多谅解。

声声慢

原文：`Pegasus[REN]`

摘要：

经过二十二年多的编程，破解，逆向工程，我看见过不少人来人去。很多人称自己为 `cracker`，但真正出色的 `cracker` 并不是太多。"scene" 是从 80 年代开始发展起来的，大量的破解团体成立了，许多信息的被分享了。以后，它们都成为了地下组织，各破解团体之间的战争也展开了。没人真正知道为什么这场战争开始，但这时几乎是 "scene" 的结尾了。这是一次对于信息的，关于怎样更好地破解的，和新技术的战斗。时光飞逝，许多 "老学校(old school)" 的 `cracker` 退休了，一些去世了，只剩一少部分人仍然活跃。他们变老了，成家立业了，或许更加灰色和更加明智了，他们重返破解领域，再次参与收集与分享信息，他们创造了一个新的 "scene"，现在是大多是合法的，不再需要是地下组织。它就象一个大家庭，如果您愿意，你可以自由加入并成为它的一分子。

## 1 什么是逆向工程?

有很多种不同定义，但共通点是：竞争、乐趣和吸取知识。举个例，逆向工程就象把建造房子的步骤倒过来：您看见一个漂亮的房子，你问自己：这个房子是怎样建造的？用一下你的脑子，倒过来想：拿走柱子，拿走墙壁，然后再掘开地基，继续下去……直到你找到人们创造这个房子的主要想法那一刻停止...

另外，从信息技术观点描述逆向工程，看一看以下例子：

有个软件公司开发了以“`name/serial`”方式保护的注册工具。它很旧，但仍然能运行，在欧洲百多个软件上使用。几年前，这个工具开发商加入一个新公司。每天新的客户购买这个软件，注册他们的资料，收注册的钥匙为了打开软件。一天，开发公司的服务器出毛病了，管理员使用存盘恢复了系统，但注册工具和注册原始码未被存放！现在是时候了，尽可能赶快行动避免不良影响。您可能有五十名顾客说这是优秀软件，但只要你有一名客户说你这个公司有个小问题，您就失败了...

逆向工程开始前，要尽量收集关于这个程序的信息。它是使用哪个编程语言编的？文件被压缩或加壳了吗？它需要使用其它文件（其它的程序或系统）的支持进行工作吗？有 `Anti-debugging` 或 `Antidisassembling`（这个中文怎么说呢？）收集了足够的信息后，可以进行逆向工程。

在我们这个“`name/serial`”例子里，要扭转过来，因为我们是从最终的测试“`name/serial`”的程序开吃的，从这里，我们找上去，直到我们输入 `name/serial` 的地方。你记住那座房子，柱子，墙壁是它的骨骼。

## 2 我需要什么工具?

首先，您将需要一个非常重要工具。没人卖它，它也不可能被下载，这就是你自己的脑子！有许多工具象调试器，监测软件，反汇编程序，资源编辑等等。但不使用您自己的主要工具（你自己的脑子），您将不能了解怎么做。并且您必须结合所有工具的力量。这里有一张被使用于反向工程的工具的名单（参见表 5）。

有太多的工具。如果我想要把它们全部列名，这篇文章永远不完成...

许多商业软件是可以免费测试的。如果您认为它是必要的，请购买它和保持合法。

并且避免问他人“什么是最佳的调试器？”或“什么是最佳的反汇编程序？”。这是一种个人感觉。选择使用 `OllyDbg` 还是 `SoftIce` 去解决问题不要紧，最终的成功才是重要的，测试工具和选择你忠爱的。

## 3 我必须懂编程吗？ 如果是的，哪种语言呢？

这个问题的第一部分是可以自已回答的。使用您的主要工具（aka "脑子"），你要能结合如果您想要逆向工程，您必须能创造软件。并且软件由编程开。不管你用哪个语言来创造软件，为反向工程您需要懂得 "特别" 的那个：`assmbler` 和这方面的知识。当你要 reverse 一个软件，你的工具给你重新造出 `assembler` 的源码，你需要明白它的意思。什么是编程？存在各种的观点。

今天，几乎所有能创造一份可执行文件的，都称程序员。大多数只需“点击”工具就能产生程序，例如使用 `Visual.basic`。但它能真正地编程吗？这位“程序员”了解他的应用程序是怎么运行的吗？很多人不能，计算机快速发展，内存不花费多少钱，许多人都不会在意的。一种简单信息显示框 "hello world"，用 `VB` 那样的高级语言来写，包括这种语言指定的动态连接库，几乎需要二个兆字节运行。同样的应用显示框，如用 `assembler` 来写，使用系统动态连接库，只需要一个千字节的空间。除此，`assembler` 的程序运行起来将是比较快的，但你需要更多的时间和知识为了编程。可是为了逆向工程你需要它。如果现在您不通晓编程，开始用以下组合：`C++` 和 `assembler`，应用程序编程接口（aka `API`，和帮助文件 `Win32.Hlp`）。这样能够同时学会两种语言！您不相信我？好，看这：

`C++` source:

```
MessageBox(NULL,"myText","myCaption",MB_OK);
```

`Masm` source:

```
...
.data
myText db "myText",0
myCaption db "myCaption",0
...
.code
...
invoke MessageBox,NULL,offset myText,
offset myCaption,MB_OK
```

想一下，你会看出它们的区别。很简单，是吗？

以上举的例子只是一个侧面，如果我们把它转过来，我们会发现另一面更具意义。

用 `Visual Basic`，`Delphi` `Visual C++`，微软的 `MFC` 或者其它的高级语言写的程序，比起专用 `API` 的程序写的拆卸后难懂。并且许多人说：“它们很难反译”。但为什么他们这么说呢？因为他们开始了他们“逆向工程”时没有 `assembler` 的知识。在这的情况下，大多数在他们真正开始之前以经放弃了。不要犯同样错误。给自己一个公平的机会。

随着时间过去，您开始掌握 `assembler` 语言，和反译简单的程序。现在您必须再学。您必须学会区别。您不需要在所有的语言上都是位好程序员，您必须了解它是怎么运行的，在哪里获得解决问题的信息。很多时候，知道用哪个供能将做什么，以及哪个参量被使用就足够了。

为了学会它没有最佳的方式或“准则”，最佳的方式是开始第一步。很多前人都学会了，你也能办到！

## 4 我可以在哪里得到信息、讲解和帮助?

通过互联网你将找到几乎所有信息。当然，如果您不知道在哪里寻找，您将得不到需要的信息，有成千上万网站。这里列表的所有站点都包含许多信息，并且与其它站点的链接。大家能自由地加入的各种论坛，只是注意读规则，不要违反它们。

如果你认为，你有足够好的在逆向工程经验，您能在这里证明你的知识：

- `Reverse Engineering Academy`
  - `http://reverser-course.de`
- `CrackMes`
  - `http://www.crackmes.de`
- `ReverseMes`
  - `http://www.reversemes.de`
- `Contests`
  - `http://contests.anticrack.de`

`Disassembler/Debugger` :

- `Ida Pro` `http://www.datarescue.com`
- `SoftIce Driver Suite` `http://www.numega.com`
- `OllyDbg` `http://home.t-online.de/home/OllyDbg`
- `W32Dasm 8.93` `http://www/expage.com/page/w32dasm`
- `PE Explorer` `http://www.heaventools.com`
- `Turbo Debugger 5.5` `ftp://ftp.inprise.com/bcppbuilder/`
- `HIEW` `http://www.serje.net/sen/`

`File information` :

- `Registry Monitor` `http://www.sysinternals.com`
- `File Monitor` `http://www.sysinternals.com`
- `Stud PE` `http://itimer.home.ro/studpe.html`

`Resource editors` :

- `Resource Hacker` `www.users.on.net/johnson/resourcehacker/`
- `Borland Resource Workshop` `http://www.google.com`

`Programming` :

- `Win32 Help` `ftp://ftp.inprise.com/pub/delphi/techpubs/`
- `Microsoft Developer Network` `http://msdn.Microsoft.com`
- `Masm` `http://www.masm32.com`
- `RadASM IDE` `http://radasm.visualassembler.com/`
- `Chrome` `http://perso.wanadoo.fr/franck.charlet`
- `Borland C++ 5.5` `http://www.borland.com`
- `Watcom C++` `http://www.openwatcom.org`
- `Microsoft Visual Studio 6` `http://www.microsoft.com`
- `Digital Mars Compiler` `http://www.digitalmars.com`

`Editors` :

- `Ultraedit` `http://www.ultraedit.com`
- `Hex WorkShop` `http://www.bpsoft.com`

`TAB. 1 ━ Outils`

`Assembly language` :

- `Iczelion’s Masm site` `http://win32asm.cjb.net`
- `MasmForum` `http://www.masmforum.com`
- `Test-Department` `http://www.crahkob.com`
- `Deinmeister` `http://www.deinmeister.de`

`Forums` :

- `AntiCrack Deutschland` `http://board.anticrack.de`
- `New2Cracking` `http://n2c.anbcs.com`
- `Assembler board` `http://board.win32asmcommunity.net`
- `MasmForum` `http://www.masmforum.com`

`Tutorials` :

- `AntiCrack Deutschland` `http://www.anticrack.de`
- `Krobar` `http://zor.org/krobar/`

`IRC` :

`#rea #rce #cracking4newbies #new2cracking`

最后一句... 现在到你了！您需要为了起步的的所有信息都有了，记住我所说的：

最佳学逆向工程的方式是迈出第一步...

`Pegasus`

`4dd466d0e1e0cc123afbe504da2462ca`
