
# 围观一下 `tp` 的游戏保护

> 原文：[`www.pediy.com/kssd/pediy12/140344.html`](https://www.pediy.com/kssd/pediy12/140344.html)

<好吧,由于之前没法过帖子致使斑竹受累,原本本文分成了好几篇,现追加到了一起,第二篇在 25 楼,第 3 篇在第 29 楼>

以下言论只当作技术探讨,请勿应用在非法领域

初学驱动,闲来无事探究了一下 `t.p`,版本为 `DXF` 中的最新版本

网上有不少介绍 `t.p` 的相关文章,我也参考了一下,顺便补充其不足

首先来看一下 `t.p` 加载之后对系统内核所作的更改:

众所周知,有:

* `NtReadVirtualMemory`, `NtWriteVirtualMemory` 的入口处 inline hook
* `KiAttachProcess` 的入口处 inline hook
* `PsOpenProcess`<win7> 或 `NtOpenProcess`<xp> 的深层 inline hook
* `PsOpenThread`<win7> 或 `NtOpenThread`<xp> 的深层 inline hook
* `KdSendPacket`, `KdReceivePacket` 的 IAT hook

这几项网上已经给出了比较完整的解决方案,不再赘述

我们来看一下除此之外的一些东西:

<由于害怕 ARK 遭封杀,自己造了几个轮子>

`t.p` 加载之后创建了两条系统线程:

![](img/91256bba31bd794161b679e937ea2a36.jpg)

创建了一个 `IO TIMER`:

![](img/157546c7420d2faa8005e7924c6b2488.jpg)

创建了一个 `DPC TIMER`:

![](img/f3679d8fd4491137d97034f8b756a10f.jpg)

注册了一个 `CreateProcess` 回调:

<这个忘了截图了..![](img/8665dab461df9ff5e20e52adf9adf6de.jpg "伤心")\>

可以看到它们所涉及到的知识点还是比较多的,

想摘掉 `IO TIMER` 就要知道如何遍历 `IO TIMER` 链,

想摘掉 `DPC TIMER` 就要知道如何遍历 `DPC TIMER` 链,

想摘掉 `CreateProcess` 回调就要知道如何遍历 `PspCreateProcessNotifyRoutine` 数组,

要摘掉两条工作线程就要知道如何找到它们...

当然了,我们这里的讨论的目的不是恢复它们之后可以做什么,而是单纯性的通过探究它们来更深刻地理解 `Windows` 系统底层

今天我们首先来看一下最简单的,关于遍历 `PspCreateProcessNotifyRoutine` 数组,`PspLoadImageNotifyRoutine` 也同理

这两个数组保存了两组函数地址,它们将在有进程被创建或销毁,有镜像被装载或映射入内存的时候被依次调用,

当然了这两个符号都是未导出的,而且它们的数量在 `xp` 和 `win7` 下也有所不同,`xp`<2k3> 下最大数组数量为 8,`win7` 下为 64

既然要找到两个未导出符号,我们就在已导出的相关函数中看一下有没有线索,

* 相关函数:`PsSetCreateProcessNotifyRoutine`,注册或注销一个进程创建,销毁的通知回调
* 相关函数:`PsSetLoadImageNotifyRoutine`,注册一个镜像装载,映射的通知回调
* 相关函数:`PsRemoveLoadImageNotifyRoutine`,注销一个镜像装载,映射的通知回调

<PS 一下:不清楚为什么那么多文章说 `PsSetCreateProcessNotifyRoutine`,或 `PsSetLoadImageNotifyRoutine` 注册的例程无法被删除,从而导致相关的驱动无法被卸载,云云..

`WDK` 中已经解释的很清楚了,而且我平时的使用中没出过任何异常,两个回调都可以通过正规函数删掉,当然手动遍历数组删除也没问题>

另外值得一提的是 `PsSetLoadImageNotifyRoutine`,

该函数注册的回调例程会在镜像被装载或映射入内存之后马上被调用,

注意它是先于 `DLLMAIN` 或者 `DRIVER_ENTRY` 被调用的,联想到有的驱动会在被加载之后马上删除源文件,我们可以通过这种回调在驱动自动删掉文件本身之前将它 copy 出去,这种方式比内存 dump 这种马后炮的方式优雅可靠得多

首先来看一下 `xp` 的 `PsSetCreateProcessNotifyRoutine` <2k3 下相仿>:

![](img/e12ad87e3629e6e17c388b7410a7cc87.jpg)

可以看到在入口地址不远处就有 `PspCreateProcessNotifyRoutine` 的出现,我们可以很方便地通过搜索特征码的方式得到.

在看一下 `win7` 下的 `PsSetCreateProcessNotifyRoutine`:

![](img/8bfaf2cc45aa98249e90df6c104ab06e.jpg)

它首先调用了 `PspSetCreateProcessNotifyRoutine` 函数,追踪一下:

![](img/0a3df239a8df5314fe29b7d9c809d979.jpg)

`PspCreateProcessNotifyRoutine` 在入口不远处出现

这样,我们就得到了 `xp`<2k3>和 `win7` 下的 `PspCreateProcessNotifyRoutine` 的地址,

接下来就是遍历一下这个数组看一下每个回调例程都位于哪个模块中了,

找到之后调用 `PsSetCreateProcessNotifyRoutine`,第二个参数设置为 `TRUE` 就能摘掉回调例程了

...什么?你遍历了数组但是没有发现目标模块中的回调?好吧,这其中貌似有些猫腻,我们来看一下:

<图中选中的项是我自己的回调例程,当然地址是我已经计算好了的,`PspCreateProcessNotifyRoutine` 数组中并不会出现此地址>

![](img/89a7bba4df3d08d86dbf25b2fccc5283.jpg)

我们来看一下这个驱动文件在内核地址空间中的范围:

<图中选中项是我自己的回调例程所在模块>

![](img/e33855937c99795eebf2d8655b0a29f5.jpg)

可以计算一下该驱动文件的地址范围是 `0x927F7000` 至 `0x927F7000+7000` <927FE000>

接下来我们用 `livekd` 来看一下,直接找到 `PspCreateProcessNotifyRoutine`:

![](img/3ca7321586db34afa83e8aac41ff3c93.jpg)

可以看到 8 个回调地址中没有 `0x927F7000` - `927FE000` 范围内的地址,直觉上最后一个应该是与我们自己注册的例程有关系的数据,那么我们看一下地址 `0x9c79061f` 中又有什么玄机:

![](img/6f24475c40858a193006b2cea161507d.jpg)

这..貌似没有什么有用的数据,那么我自己注册的例程地址保存到了哪里呢?于是我们自然想到翻看一下 `WRK`,

在 `WRK` 中找到 `PsSetCreateProcessNotifyRoutine`,

然后发现其中一处调用了 `ExAllocateCallBack`,看名称应该是为回调分配内存,我们看一下:

![](img/7f0e1328ba854ba93e50a10fe758b85a.jpg)

这里进行了内存分配,并且将传参 `Function`,也就是我们注册回调时提供的例程地址,写入了一个结构中 `PEX_CALLBACK_ROUTINE_BLOCK`,看一下这个结构有什么成员:

![](img/9f99547b7693282152814e7b9433a5f1.jpg)

我们感兴趣的只有结构中的第二个成员 `Function`.

回想一下,既然系统为回调分配了内存,那么 `PspCreateProcessNotifyRoutine` 中保存的应该是每个结构的地址了,但是回过头我们再看一下 `0x9c79061f` 地址处的数据:

![](img/6f24475c40858a193006b2cea161507d.jpg)

这里还是没有我们要寻找的地址!

而继续在 `PsSetCreateProcessNotifyRoutine` 中寻找线索也没有什么收获

于是我们开始转换了思路,既然我们找不到这个回调的地址,那么系统是如何找到的呢?

嗯,貌似这是一个新的突破口,可是我们不知道系统是在什么时候通过什么方式调用 `PspCreateProcessNotifyRoutine` 中的例程的啊!

额,再想一下,系统要在某个函数中调用 `PspCreateProcessNotifyRoutine` 中的例程,那么应该会对 `PspCreateProcessNotifyRoutine` 进行引用的吧,我们在 `WRK` 中搜索 `PspCreateProcessNotifyRoutine` ,看一看都有哪里对其进行了引用:

![](img/fed36178edcd3077d33ddc5f5033afe4.jpg)

于是我们找到了这里:

![](img/ad5d06135968aeecd130ec478f3f2e83.jpg)

想想也对,因为 `PspCreateProcessNotifyRoutine` 是在进程创建,销毁都进行调用的

跟踪一下 `CallBack = ExReferenceCallBackBlock (&PspCreateProcessNotifyRoutine[i]);` 这行代码,这里应该是取得回调结构的调用

![](img/d1b5989e371db553d0bc998e653dad1f.jpg)

仔细看看该函数的函数头,我们才发现原来 `PspCreateProcessNotifyRoutine` 中保存的是 `PEX_CALLBACK` 结构:

![](img/6c05bdd65cae71e691d266aa9ab9abf1.jpg)

![](img/381d5f5d8d45aa513edd912c3d967325.jpg)

我们看这一行代码

`CallBackBlock = ExFastRefGetObject (CallBack->RoutineBlock);`

`CallBackBlock` 被定义为 `PEX_CALLBACK_ROUTINE_BLOCK` 也就是 `PsSetCreateProcessNotifyRoutine` 分配的内存数据结构

它的意思很明确,就是得到这个结构的地址,看一下 `ExFastRefGetObject`:

![](img/5dad07a12999d0364276fc7899ff243e.jpg)

只是简单地和一个常量进行了与运算

好了,我们理清思路:

* `PspCreateProcessNotifyRoutine` 数组中保存的是 `PEX_CALLBACK` 类型的结构地址数组;
* `EX_CALLBACK` 结构中只有一个成员,就是 `RoutineBlock` ,它是一个 `EX_FAST_REF` 结构;
* `EX_FAST_REF` 是一个联合,我们这里简单地将之视为一个指针;
* 省略掉中间步骤, `PspCreateProcessNotifyRoutine` 保存的就是指针数组;
* 系统将数组中的指针传递给 `ExFastRefGetObject`,就得到了 `PEX_CALLBACK_ROUTINE_BLOCK` 结构地址,这个结构中保存这我们想要的回调例程地址;
* `ExFastRefGetObject` 将传进来的指针进行了与运算得到了 `PEX_CALLBACK_ROUTINE_BLOCK` 结构地址.

好了,回想一下我们自己寻找回调地址的过程,我们没有进行最后的与运算而是直接将 `PspCreateProcessNotifyRoutine` 中的数据当成了结构的地址

最后看一下那个神秘的常量:

![](img/ae3bdffdb793a04a067147ec52333f90.jpg)

`win32` 下被定义为 7,先取反再与运算,也就是低 3 位清零

我们回过头在用 `windbg` 看一下:

![](img/3ca7321586db34afa83e8aac41ff3c93.jpg)

对 `0x9c79061f` 进行运算得到了 `0x9C790618`, 看一下这里的数据:

![](img/08138e1a10e9634c5fe5f29836be16f0.jpg)

成功得到了注册的例程地址,我们在 `PT` 中验证一下:

![](img/6a51e0c3581dab22c23f2e4376e462fb.jpg)

如过想通过 `PsSetCreateProcessNotifyRoutine` 摘掉其他进程的回调,传参必须经过以上运算

当然了,如果你直接将 `PspCreateProcessNotifyRoutine` 数组暴力清空也行,不过既然有相对优雅的方式,何乐而不为呢?

关于 `IO TIMER`, `DPC TIMER` 等等, 下回再和大家一起分析~~

如有疏漏之处,请各位前辈不吝赐教
