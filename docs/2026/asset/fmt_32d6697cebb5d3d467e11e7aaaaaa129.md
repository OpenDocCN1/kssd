
# [原创]TTD调试与ttd-bindings逆向工程实践

## 前言

在上篇文章《对一个apk协议的继续分析—libsgmain反混淆与逆向[^1]》中的调试trace一节，我提到了微软的TTD，即Time Travel Debugging，还放了沈沉舟[^2]和krash[^3]两位前辈的文章。

什么是TTD呢，按官网的介绍[^4]来说，TTD是一款用户级进程的trace录制工具，录制完成后可以在调试器中向前向后重放，不需要再重新运行程序就能让你的调试器状态回退，并且还能分享你的trace文件给别人，从方方面面帮助你更快更轻松找到bug。

当时我只是通过各种文章了解到TTD的相关介绍，但一直没有什么契机和需求，就还没有实操过，甚至Windbg也没怎么摸过。直到沈沉舟前辈前段时间在微博上发了一个关于Windows UWP Calculator的puzzle[^5]:

> Win10 Calculator是UWP程序，找出UI界面上乘法运算对应的汇编指令所在。要求计算器必须是UWP，且版本不低于11.2210.0.0。非UWP计算器不考虑，更低版本UWP不考虑，可以是Win11上的UWP计算器，只要版本不低于11.2210.0.0。

当时一看到这个puzzle，便想这是个绝佳的学习上手TTD的机会啊，随后便跟着沈沉舟前辈的TTD入门教程[^6]快速上手了新版的Windbg和TTD调试功能，没花多少时间便把这个puzzle解决了。

回答之后沈沉舟前辈又提出了一个升级版的问题[^7]：

> 假设已有run文件，如何在几分钟内定位四则运算的汇编指令所在。这类问题的答案最终都将演变成生产力工具。

因为我对沈沉舟前辈一些文章十分熟悉了，一看到这个升级版的问题，便知道解答方向正是《TTD调试进阶之ttd-bindings[^8]》一文所讲的ttd-bindings。

然后就快速上手学习了ttd-bindings的使用，做了各种尝试，踩了不少坑，解答了升级版的问题，可以做到一分钟左右定位四则运算的汇编指令所在。

整个过程还是十分宝贵有意义的，这也是我记录此文的主要原因。

一方面我体会到了TTD调试的神奇，当用TTD调试解决了第一个问题后，调试体验让我回味无穷，我很激动地跟学弟xxr0ss分享说“逆向调试体验飞升了一个档次”。在刚开始沉浸在使用TTD反向调试带来的新鲜感中时候，我甚至想起了《百年孤独》中看冰的桥段:)。

不过事后我对TTD进行了一番互联网考古才发现，TTD属于反向调试世界中的一员，而反向调试有着丰富的历史，发展至今已经比较成熟了，所以我这算是少见多怪了。

另一方面这种通过好的问题与挑战来学习的方式让我获益匪浅。虽然我很早就知道有TTD这种东西了，但是一直没有动手去尝试下，而沈沉舟前辈的puzzles却让我很快行动起来，短时间内便能熟悉TTD调试功能以及使用ttd-bindings去编程解决问题，可以看出激励作用是十分巨大的。

这种方式也是非常有利于学习的，沈沉舟前辈在《scz's puzzles[^9]》一文中就提到说：“一个好问题可以产生很多意料之外的成果”，而我也是在解决问题过程中产生了十分多意料之外的成果，除了学习到TTD调试与ttd-bindings的使用，还了解到反向调试的发展历史和不同反向调试器的差异与一些技术实现原理。

## TTD调试

搜一搜的话，会发现除了官网文档，中文互联网关于TTD调试的内容少得可怜，而关于ttd-bindings使用介绍的更是几乎只有沈沉舟前辈的一篇[^8]（可能因为ttd-bindings是去年刚开源的）。。

沈沉舟前辈主要有这几篇：

* MSDN系列(46)--WinDbg Preview TTD入门[^6]
* TTD调试进阶之ttd-bindings[^8]
* TTD历史回顾[^14]

把这几篇文章给读一遍，基本上就可以知道TTD调试和ttd-bindings的来龙去脉和基本使用了。

TTD官网文档也蛮丰富详细的，先上手简单操作感受一下，再回头仔细读一遍官网上TTD部分，也是十分不错的。

我也根据文档概括性简单说一下TTD吧。

TTD功能要求WinDbg版本1.0.13.0或者更新，进行录制需要管理员权限，可以使用WinDbg客户端录制，也可以使用命令行录制，文档有一节专门说TTD命令行使用的，命令行爱好者可以看一下。

由于TTD录制是侵入式的技术，可能会和别的也使用了侵入式技术的应用有冲突，如跟踪了系统内存调用、阻止内存访问的应用，有反病毒软件和Microsoft Enhanced Mitigation Experience Toolkit等。应用了虚拟化框架electron的录制可能会出现死锁或者崩溃的情况。

录制完成会生成`.run`后缀的trace文件，打开Windbg选择Open trace file，输入文件路径，进入后会自动生成优化对跟踪信息访问的`idx`文件，trace文件和`idx`文件都很大，`idx`文件通常是trace文件的两倍大，所以要保证有充足的存储空间。trace文件生成大小没有上限，文档说WinDbg可以重放几百GB大小的trace文件，影响trace生成大小的因素有执行的指令数、录制时间和应用内存大小。

不用`idx`文件来调试也是可以的，但不推荐，因为`idx`文件能保证从调试进程读取的内存值是最准确的，并且能够提高各种调试操作的效率。可以手动使用`!index -status`命令检查关联trace文件的`idx`文件状态，`!index -force`命令可以重新生成`idx`文件。如果trace文件不能用出现了一些报错，可能就需要重新进行录制了。

文档还提到了一个在调试中可能会碰到的错误信息“Derailment events”[^10]，即执行脱轨事件。这个其实蛮有意思，涉及到了TTD的技术实现原理，TTD录制并没有完全进行trace，那样生成的trace文件会非常大，还有一部分技术是重执行，录制过程出错或是模拟执行引擎出错，都会导致脱轨事件，后面TTD互联网考古部分还会说到这个。

调试的话一般我喜欢把窗口调成这样。

![图片描述](upload/attach/202307/802108_3NA2V2K577GD6DV.png)

左下角是Timelines时间线窗口，最左边是trace的开始，最后就是结束，时间线可以可视化表示执行过程中的各种事件，如断点、内存读写、call/ret调用和异常，官网文档介绍得很详细，可以直接去读一遍官网。

TTD时间点位置表示是用两个十六进制数组成 `Major:Minor` 这样的形式，`Major`是序列编号，对应一个序列事件，`Minor`是步数，大致相当于序列事件后的指令数。使用`!position`命令可以查看当前所处的时间位置，使用`!tt Major:Minor`命令达到相应的时间点位置，`!tt`还可以跟着一个十进制数，表示达到整个trace时间线的百分比位置。

![图片描述](upload/attach/202307/802108_9TMDW3JXCYGUDEN.png)

左上角是四个正向调试操作，四个反向调试操作，第一次使用反向调试操作的话会有十分新奇的体验。

![图片描述](upload/attach/202307/802108_YYQWMBX2HTAS5UT.png)

导航栏有个“Time Travel”，主要是一些命令的图形化按钮。Index Trace对应命令`!index -force`，未自动生成`idx`文件的话可以用这个手动生成。Events可以图形化查看模块加载和异常事件对象的属性如所处的时间点，命令行也可以实现。两个Time travel to start和Time travel to end对应命令`!tt 0`和`!tt 100`，可快速到达开始和结束的地方。Information可以查看trace信息，包括trace文件大小，创建时间和线程数量。Timelines就是打开时间线窗口。

![图片描述](upload/attach/202307/802108_NJRN8AADFKAU476.png)

相比于别的反向调试器，TTD一个更强大的地方可能是，调试器加载trace文件后，会将trace过程中各种属性操作或事件生成对应的TTD数据模型对象，然后可以使用LINQ进行查询，关于LINQ放一段官网文档的介绍[^11]：

> Language-Integrated Query (LINQ) is the name for a set of technologies based on the integration of query capabilities directly into the C# language. Traditionally, queries against data are expressed as simple strings without type checking at compile time or IntelliSense support. Furthermore, you have to learn a different query language for each type of data source: SQL databases, XML documents, various Web services, and so on. With LINQ, a query is a first-class language construct, just like classes, methods, events. You write queries against strongly typed collections of objects by using language keywords and familiar operators. The LINQ family of technologies provides a consistent query experience for objects (LINQ to Objects), relational databases (LINQ to SQL), and XML (LINQ to XML).

这样相当于TTD为我们把trace过程中各种信息事件整理好了，我们只要按需查询就可以了，别的很多反向调试器远远没有做到这么多，只是在普通调试器基础上增加了录制重放、反向调试和反向断点等功能。

可以在官网文档中看到数据模型对象的种类十分丰富。

![图片描述](upload/attach/202307/802108_9BFG6D74VDD5VZC.png)

举个内存对象（Memory objects）结合实践的例子吧，场景就是我们在堆中搜索到了一个值`0x9420fef2`的地址是`0x149808ac440`，想找到这个值计算产生的汇编现场，已经完成了TTD录制过程。

![图片描述](upload/attach/202307/802108_EHTHUYKNPE6DSY7.png)

我们使用LINQ查询写入或读取`0x149808ac440`地址开始的四个字节的内存对象，就是`@$cursession.TTD.Memory(0x149808ac440,0x149808ac444,"rw")`，使用`dx -g`以网格形式显示出来：

![图片描述](upload/attach/202307/802108_NHM76YXAMZ6BTEQ.png)

一共有`0x4a`条，每条都有很多属性，匹配的太多了，我们只需要读写了值`0x9420fef2`的部分，对应的属性是`Value`，我们可以使用`Where`从句过滤一下 `.Where(m=>m.Value==0x9420fef2)`。

![图片描述](upload/attach/202307/802108_TDBTN8ASAWECSEQ.png)

可以看到符合条件的有一条查询，表明在`223E1:B53`时间点位置，有对`0x149808ac440`地址写入四个字节即值`0x9420fef2`的操作，接着我们可以执行`!tt 223E1:B53`跳转到操作时间点位置分析汇编，或者直接点击蓝色的时间点，会执行对应的命令，也可以跳转到操作时间点位置。

是不是十分方便，其实在TTD和别的反向调试器中“数据断点+反向执行”也可以实现同样的效果，但是很耗时，远不如这种TTD整理好后，通过LINQ查询的方式快速与便捷。

TTD官网文档中对各种数据模型对象的属性和查询方式都有详细的说明介绍[^12]，值得仔细阅读遍历一遍。

## ttd-bindings

TTD的基本情况差不多说完了，然后开始说下ttd-bindings吧,引用一下沈沉舟前辈对ttd-bindings的介绍：

> TTD录制的.run文件格式未公开，之前只能在WinDbg Preview中操作.run，很难对之进行脚本操作，比如，想对position进行大小比较，很费劲。

> Bindings for Microsoft WinDBG TTD
> 上文作者对TTDReplay.dll进行逆向工程，逆出一套编程解析处理.run文件的API。没有完整逆向，但基本功能都有，比windbgx的GUI还多出一些功能。

> C++ bindings功能较多，Python bindings功能弱一些，后者无法设置CallRetCallback、MemCallback。此外，pyTTD.pyd用的是Python 3.10，为了在Python 3.9中使用，必须自己编译出pyTTD.pyd。

> 作者提供了.cpp、py的使用示例，对于逆向工程人员来说，浅显易懂，可以照猫画虎实现自定义功能。

上手的话，仔细读一遍《TTD调试进阶之ttd-bindings》[^8]，再下载项目代码[^13]看看几个example的实现，动手编译一下，跟着写一写就能熟悉使用了。

借助ttd-bindings可以通过编程来实现对trace文件的自动化操作处理，比如可以基于trace文件开发出多样实用的分析工具，项目仓库中就给出了四个工具的实现样例：

![图片描述](upload/attach/202307/802108_YJFHTCJGRERVYM2.png)

以及在调试过程中，有时候我们想进行一些重复连续的处理，比如我们想遍历trace每一条指令的执行的同时，对寄存器值进行一些判断，光靠WinDbg的TTD调试就有些困难了，但这时候就是ttd-bindings大显身手的时候了。

不过ttd-bindings毕竟是逆向出来的API，不仅.run文件格式没有公开，TTD内部实现原理和细节公开的资料也不多，所以用起来多多少少会踩一些坑。

总的来说TTD调试本身已经很好用了，ttd-bindings则让TTD更加好用。

## TTD互联网考古

受沈沉舟前辈《TTD历史回顾[^14]》一文启发，我对TTD进行了一番互联网考古。发现在互联网历史中人们更习惯说Reverse Debugging，而非Time Traveling Debugging,Jakob在《A new (and old) Reverse Debugger – Microsoft WinDbg[^15]》一文最后 TTD? Reverse? 一节里写到说目前为止，还是很多工具、项目和公司用的是反向调试。

> It seems that time-traveling debug or time travel debugging is becoming more common as a term (again). Still, most tools and companies and projects still talk about reverse debugging. Undo, RR, gdb, and Simics all use “reverse” to describe the feature. If I look at the history of the field as far as I have been able to trace it, most people call it reverse until the a 2005 paper introduces “time-traveling virtual machines”.

> For more on the history of reverse debuggers, see my three-part series of blogs from 2012 that I have since updated as new products and tools have appeared or been found in the archive of history: history 1, history 2, history 3.

如果在谷歌中搜索Reverse Debugging，相关内容确实要比搜索Time Traveling Debugging多得多。

顺便一提，Jakob关于反向执行调试的一系列博客[^16]，是我这次考古中最大的收获，学习思考到了十分多，这里向大家推荐一下。下面这个是《Reverse History》三篇中第一篇[^17]的开头，就很引人入胜。

![图片描述](upload/attach/202307/802108_4GZTJ5UUB29SE7R.png)

我最初除了从沈沉舟前辈这里了解到TTD调试，还从krash前辈《使用时间无关调试技术(Timeless Debugging)高效分析混淆代码[^3]》一文中了解到TTD，他在文中写到：“我理解的时间无关调试就是记录程序执行过程中的寄存器，和内存变化，使用记录的trace离线调试分析的过程，简而言之记录trace，分析trace。”

![图片描述](upload/attach/202307/802108_BC36TYJXKSAATMG.png)

在我进行一番TTD互联网考古后，才发现不完全如此。

一方面，这种基于录制的Trace-based debugging只是反向调试的一种，主要体现为调试工作都在trace上进行，在《Reverse History Part One[^17]》文中有段有详细写到：
