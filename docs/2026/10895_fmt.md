
# [原创] win7 x64 `IDT 0E` `KiPageFault` 分析

最近研究`IDT hook`，有很多地方不明白，所以就逆向了一遍`0E`。本人刚入门不久，可能分析的有些地方是错的。

首先看下异常堆栈结构：

![](https://bbs.kanxue.com/upload/attach/201904/802719_GKGHEM66H35XT3X.jpg)
![](https://bbs.kanxue.com/upload/attach/201904/802719_YFA2DX36MXYFRBF.jpg)

刚进异常的时候，系统会自动往堆栈里面从`18c`一直压入到`168`的`R3`环境，便于`iretq`返回`R3`。但是`0E`比较特殊，系统会多压入一个`ErrorCode`。

所以刚进入`0E`的时候，`RSP`所在的位置就在结构体`+160`的地方。

![](https://bbs.kanxue.com/upload/attach/201904/802719_CS29F6WJ8ZX4ZAN.jpg)

开始，他会保持环境，判断先前模式，然后做不同的处理。

是`R3`他就会`swapgs`，然后读取`msr102`，我估计应该是校验`gs`的准确性。

然后保存调试寄存器状态。

![](https://bbs.kanxue.com/upload/attach/201904/802719_SZWCN55EJGS2YRM.jpg)
![](https://bbs.kanxue.com/upload/attach/201904/802719_T3A64XTAFZKRTQF.jpg)

这里没什么特别的操作，就是保存下环境，各种读取`msr`，不知道有啥用。

![](https://bbs.kanxue.com/upload/attach/201904/802719_HA5TRZ4J6EZG9WF.jpg)

然后这里是一些常规的操作，保存寄存器之类的，就是不知道`ums`是个什么东西。

![](https://bbs.kanxue.com/upload/attach/201904/802719_UWCQG26V9VD34JW.jpg)

这里出现了两个函数：`MmAccessFault`和`KiPreProcessKernelAccessFault`。这两个函数就是关键处理页异常的函数了。

在参考`wrk`和他传入的参数来看，他处理失败会返回错误代码，然后成功会返回`0`。`R3`直接由`MmAccessFault`处理，`R0`则需要多调用一个`KiPreProcessKernelAccessFault`。这俩函数实现太多了，就没分析了，不然真想看看他怎么处理异常的。

![](https://bbs.kanxue.com/upload/attach/201904/802719_C29PB5K8BWXKZKN.jpg)
![](https://bbs.kanxue.com/upload/attach/201904/802719_3QMA8WP8845FG72.jpg)

下面就是紧接着判断异常代码，然后分发异常，没人处理就直接蓝屏了。

![](https://bbs.kanxue.com/upload/attach/201904/802719_J4GR2RWXPJG4HE2.jpg)
![](https://bbs.kanxue.com/upload/attach/201904/802719_QST6GUVUZ2Q7WXT.jpg)

最后就是还原环境，准备退出了。实力有限，只能分析出这么多东西。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2019-4-26 01:55 被不吃麻婆豆腐编辑，原因：
