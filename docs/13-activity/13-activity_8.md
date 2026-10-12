
# 《加密与解密(第三版)》学习指引(第四章)逆向分析技术

> 原文：[`www.pediy.com/kssd/pediy10/73989.html`](https://www.pediy.com/kssd/pediy10/73989.html)

数学书每介绍一条定理，一般会给出证明过程。哲学书上给出一个观点，也给出一堆实例。即使物理或者化学，如果条件许可，也会有相关的实验。相信大家也同意这样的观点，如果只会使用，而不知道如何得出，那么你很难创造/发现新的事物。
高级语言添加了很多特性，不同的语言提供的特性也不尽相同。这些特性方便了软件开发者，却苦了软件分析者。相信大家现在知道 `PEID` 显示软件由什么语言编写的意义了吧。书中仅仅分析了 `VC`，在实际工作中是远远不够的。我们也要关注分析高级语言特性的方法。

引用: 1. 找到你的代码。正如`4.1 启动函数`描述的那样,第一步必须先找到你的代码编译后所在的位置。
2. 从简单做起。`4.2 函数`以及后面的几节，都先是简单写了要了解的代码，然后寻找这些代码与汇编代码的对应关系。

高级语言特性的分析除了让你更好的理解软件之外，也有助于解决实际问题。正好书中缺少例题，推荐两个简单的题目大家练习。
引用: 1. 用 `MFC` 写一个窗体，并为其增加一个按钮。找到定位这个按钮的点击事件的一般方法。

**参考答案**  
逆向角度看 `MFC` 程序消息机制　作　者: szdbg
[`bbs.pediy.com/showthread.php?t=54150`](http://bbs.pediy.com/showthread.php?t=54150)
定位ＭＦＣ程序消息处理函数（虚函数表）　作　者: 北斗之摇光
[`bbs.pediy.com/showthread.php?t=41087`](http://bbs.pediy.com/showthread.php?t=41087)

2. 用 `VB` 写一个窗体，并为其增加一个按钮，找到修改这个按钮的 `Enabled` 属性的一般方法。

**参考答案**  
`VB` 快速逆向法　作　者: nbw
[`bbs.pediy.com/showthread.php?t=12133`](http://bbs.pediy.com/showthread.php?t=12133)

分析高级语言是大工程，拥有分析的能力仅仅是为了应付找不到资料又不得不做的情况。幸运的是前人已经作了大量的分析，你需要做的是阅读并且记忆

引用: ［翻译］Reversing.Microsoft.Visual.C++.Part.CN　作　者: buddhaluo
[`bbs.pediy.com/showthread.php?t=36539`](http://bbs.pediy.com/showthread.php?t=36539)
跟踪调试 `COM` 组件的接口　作　者: kisser1
[`bbs.pediy.com/showthread.php?t=46523`](http://bbs.pediy.com/showthread.php?t=46523)
`Delphi` 中类的逆向工程　作　者: firstrose
[`bbs.pediy.com/showthread.php?t=8209`](http://bbs.pediy.com/showthread.php?t=8209)
`Delphi` 的内部字符串处理函数/过程不完全列表　作　者: firstrose
[`bbs.pediy.com/showthread.php?t=8807`](http://bbs.pediy.com/showthread.php?t=8807)
`VB` P-code　作　者: cyclotron
[`bbs.pediy.com/showthread.php?t=8946`](http://bbs.pediy.com/showthread.php?t=8946)
[`bbs.pediy.com/showthread.php?t=8947`](http://bbs.pediy.com/showthread.php?t=8947)
[`bbs.pediy.com/showthread.php?t=8948`](http://bbs.pediy.com/showthread.php?t=8948)
[`bbs.pediy.com/showthread.php?t=8949`](http://bbs.pediy.com/showthread.php?t=8949)
浅谈 `VB6` 逆向工程　作　者: MengLong
[`bbs.pediy.com/showthread.php?t=8794`](http://bbs.pediy.com/showthread.php?t=8794)
[`bbs.pediy.com/showthread.php?t=8835`](http://bbs.pediy.com/showthread.php?t=8835)
[`bbs.pediy.com/showthread.php?t=8914`](http://bbs.pediy.com/showthread.php?t=8914)
[`bbs.pediy.com/showthread.php?t=8992`](http://bbs.pediy.com/showthread.php?t=8992)
[`bbs.pediy.com/showthread.php?t=8993`](http://bbs.pediy.com/showthread.php?t=8993)
\[翻译\]Reversing .NET　作　者: kkbing
[`bbs.pediy.com/showthread.php?t=26513`](http://bbs.pediy.com/showthread.php?t=26513)
