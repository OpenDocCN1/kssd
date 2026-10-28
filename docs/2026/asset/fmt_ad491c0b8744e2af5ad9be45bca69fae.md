
# [原创]基于PMI实现对读写行为检测

这几天分析某驱动样本发现几种检测读写的方法，就有意思的一种和大家分享。

## PMI

使用CPU提供的性能监视（PM）功能，

对内存读写时的关键挂靠函数（`KeStackAttachProcess`, `KeAttachProcess`）实现了一种类似hook的技术。

通过监视调用情况，来判断进程内存是否被读取。

关于处理器的性能监视功能，不同的处理器架构都有差异，可以参考白皮书。

或者这个翻译的帖子：<mark class="encrypted">7d7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8$3c8W2L8X3!0F1k6#2)9J5k6h3y4G2L8g2)9J5c8X3y4K6x3e0l9$3y4o6M7#2x3o6l9&6i4K6u0r3</mark>

大致原理：

设置`ICACHE_MISS`类型的监视事件开启性能监视功能。

当L1指令高速缓存未命中次数达到溢出值时，触发PM中断，执行样本注册的PMI回调函数。

在回调函数中判断中断的位置是否处于监视函数代码的序言部分（这样才能获取到正确的调用参数），

如果处于那么通过调用参数判断挂靠的目标进程是否为被保护的进程，进而记录下调用信息。

不处于则使用`clflush`或`clflushopt`指令使监视函数代码从缓存中失效，以提高下次被中断到的概率。

下面是详细的流程：

### 判断处理器对性能监视功能的支持性

通过`cpuid`的0号功能来判断处理器是不是intel（看代码样本尚未支持amd）。

![](upload/attach/202210/963828_BCQ6RQVV2G3T837.png)

`Cupid`的a号功能判断处理器是否支持版本2以上的性能监视功能

![](upload/attach/202210/963828_NEKRT5X6B2G8DGF.png)

单个核心是否支持4个以上的计数器

![](upload/attach/202210/963828_9XDJ67PRU8ZR8TG.png)

是否支持`clflush`指令，并且处理器是否支持`ICACHE_MISS`类型的监视事件

![](upload/attach/202210/963828_PBRRR4PSDSZKUJW.png)

### 获取监视函数（KeStackAttachProcess, KeAttachProcess）

解析`nt`模块的导出表得到两个监视函数地址，并解析异常表得到两个监视函数的序言部分的大小（`SizeOfProlog`），应该是为了兼容不同系统的做法。

![](upload/attach/202210/963828_8N568QF3W6M7YF8.png)

### 配置并开启性能监视

#### 1、注册PMI回调函数

获取`nt`模块导出的`HalDispatchTable`地址，这是`hal`提供的一张函数地址表。

调用其中的`hal!HalpSetSystemInformation`并指定`HalProfileSourceInterruptHandler`来注册自己的PMI回调函数。

其中的`foo`就是注册的回调函数地址。

![](upload/attach/202210/963828_2TJCVJ6Z7W3UZ6E.png)

样本使用Windows提供的接口来注册回调函数，当然也可以通过配置`IA32_X2APIC_LVT_PMI`寄存器来指定PMI发生时的中断向量号。

Windows系统中配置的向量号是`0xfe`。

![](upload/attach/202210/963828_CWP6WYFNCS8MQDQ.png)

中断历程实际是`_KINTERRUPT.ServiceRoutine`字段指向的函数：`hal!HalpPerfInterrupt`

而这个函数很简单就是逐个调用`hal!HalpPerfInterruptHandler`中注册的PMI回调函数。

![](upload/attach/202210/963828_AUUJSHZ2CWE4K9D.png)

所以`hal!HalpSetSystemInformation`的作用很简单就是指定的`foo`参数设置到`HalpPerfInterruptHandler`中。

#### 2、开启每个处理器核心中的计数器

开启参数中`CounterMode_0`指明计数器的类型（固定用途、通用），当然样本只用到了通用的性能监视计数器。

`CtrxIndex_4`是每个处理器核心中开启的计数器的索引。

`EVESEL_8`是选择的性能监视事件，也就是在一开始提到的`ICACHE_MISS`类型。

`CtrCount_C`是计数值，这个参数直接决定了PMI发生的频率，计数值越低，频率越高，当然中断到监视函数的概率也越高。

![](upload/attach/202210/963828_EQ4EANCXTBAW8MX.png)

![](upload/attach/202210/963828_53FKHCBCJB465TT.png)

开启前保证计数器处于停止状态

![](upload/attach/202210/963828_HSC5TUFR69AEFTG.png)

重置指定计数器的计数值

![](upload/attach/202210/963828_74N2X5BW8377VT3.png)

选择监视事件

![](upload/attach/202210/963828_4793PN48SZ62TED.png)

开启性能监视

![](upload/attach/202210/963828_6M8UX7YBUGY7RAV.png)

### 回调函数处理

样本注册的回调函数为`PMICallBackMode0_140058d40`，唯一的参数是`_KTRAP_FRAME`，记录了发生中断的寄存器环境

![](upload/attach/202210/963828_P5MUWAX2Y8NP6ZW.png)

通过`_KTRAP_FRAME`，可以拿到发生中断的地址，用来判断是否命中监视函数的序言部分。

![](upload/attach/202210/963828_W9UF4U7KXJKD96F.png)

如果命中，进而判断是否挂靠指定进程。

![](upload/attach/202210/963828_EKAFXNNPGYCQAH7.png)

如果未命中，将定时使用`clflush`或`clflushopt`指令让监视函数代码从缓存中失效

![](upload/attach/202210/963828_6GQYNW8RTWGNNJG.png)

![](upload/attach/202210/963828_66ENHRSDGMRZEQT.png)

最后，不管是否命中，都需要清除溢出标志，重置计数器以允许下一次PMI处理程序调用。

![](upload/attach/202210/963828_5EC7G8KDNMFQCQB.png)

![](upload/attach/202210/963828_NU5DV3NBPCHA93R.png)

![](upload/attach/202210/963828_K58D6VEZHT6EGWA.png)

## 总结

我将样本实现PMI的关键代码拷贝出来编译后在虚拟机中运行，并测试XT扫描进程钩子时读写内存的情况，确实可以命中！

（测试时保证处理器支持PM并且虚拟机处理器开启虚拟化CPU性能计数器）。

![](upload/attach/202210/963828_M2E8733WMNX97JU.png)

这种中断实现的监视调用的方法应用场景有限，即使使用`clflsuh`指令提高了对监视函数的命中概率，

实际测试效果也一般，考虑运行效率的情况下不能将计数值设置的足够小，就不能保证每次调用监视函数时都能命中。

但在游戏安全方面，用来监视高频率的读写行为确实很有效果，并且其不依赖系统函数的隐蔽性和中断的随机性意味着更难被作弊者发现。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2022-10-1 07:49

被晚安玛卡巴卡编辑，原因：

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

上传的附件：

- [sample_code.zip](javascript:void(0);)

（15.85kb，207次下载）
