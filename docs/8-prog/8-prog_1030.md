
# 金山词霸屏幕取词技术

> 原文：[`www.pediy.com/kssd/pediy04/kanxue201.htm`](https://www.pediy.com/kssd/pediy04/kanxue201.htm)

“金山词霸”屏幕取词技术揭密(讨论稿)

**主题**　屏幕取词技术系列讲座（一）

**作者**　亦东

很多人对这个问题感兴趣。

原因是这项技术让人感觉很神奇，也很有商业价值。

现在词典市场金山词霸占了绝对优势，所以再做字典也没什么前途了。我就是这么认为的，所以我虽然掌握了这项技术，却没去做字典软件。只做了一个和词霸相似的软件自己用，本来想拿出来做共享软件，但我的词库是“偷”来的，而且词汇不多，所以也就算了，词库太小，只能取词有什么用呢？而且词霸有共享版的。

但既然很多人想了解这项技术，我也不会保留。我准备分多次讲述这项技术的所有细节。

大约每周一两次。想知道的人就常常来看看吧！

## 一. 基础知识

首先想编这种程序需要一些基础知识。

* 会用`Vc++`，包括 16/32 位。
* 精通`Windows API`特别是`GDI`、`KERNEL`部分。
* 懂汇编语言，会用`softice`调试程序，因为这种程序最好用`softice`调试。

## 二. 基本原理

在`Window 3.x`时代，`windows`系统提供的字符输出函数只有很少的几个。

* `TextOut`
* `ExtTextOut`
* `DrawText`
* ...

其中`DrawText`最终是用`ExtTextOut`实现的。

所以`Windows`的所有字符输出都是由调用`TextOut`和`ExtTextOut`实现的。因此，如果你可以修改这两个函数的入口，让程序先调用你自己的一个函数再调用系统的字符输出，你就可以得到`Windows`所有输出的字符了。

到了`Windows95`时代，原理基本没变，但是 95 比 3.x 要复杂。开始的时候，一些在`windows3.x`下编写的取词软件仍然可以是使用。但是后来出了个`IE4`，结果很多词典软件就因为不支持`IE4`而被淘汰了，但同时也给一些软件创造了机会，如金山词霸。其实`IE4`的问题并不复杂，只不过它的输出的是`unicode`字符，是用`TextOutW`和`ExtTextOutW`输出的。知道了这一点，只要也截取就可以了。不过实现方法复杂一点，以后会有详细讲解。现在又出了个`IE5`，结果词霸也不好用了，微软真是#^@#$%$\*&^&#@#@..........

我研究后找到了一种解决办法，但还有些问题，有时会取错，正在继续研究，希望大家共同探讨。

另外还有`WindowsNT`，原理也是一样，只是实现方法和 95 下完全不同。

## 三. 技术要点

要实现取词，主要要解决以下技术问题。

1. 截取 API 入口，获得 API 的参数。
2. 安全地潜入 `Windows` 内部，良好地兼容 `Windows` 的各个版本
3. 计算鼠标所在的单词和字母。
4. 如果你在`Window95`下，做 32 位程序，还涉及`Windows32/16`混合编程的技术。

今天先到这里吧！最好准备一份`softice for 95/98`和金山词霸，让我们先来分析一下别人是怎么做的。

欢迎与我联系

`E-Mail:yeedong@163.net`

**Guest**　1999-04-30 16:00:48

请问用`VC`自己的`DEBUGGER`不行吗?为什么要用`SOFTICE`? 我没用过`SOFTICE`,它有什么特别之处吗?

**葫芦**　1999-04-30 19:15:03

本人对这个问题也有兴趣，以前研究过 16 位版本截获`TextOut`和`ExtTextOut`的过程；

但对金山词霸，用`Softice`跟踪，发现`SetWindowsHookEx`在程序装载时就安装了鼠标钩子，暂停取字/恢复取字只是设置的内部变量，但本人发现金山词霸并没有象 16 位版本那样修改`TextOut`和`ExtTextOut`。

**苍蝇** (555021552)　1999-05-02 08:56:57

有哪位大虾愿意先介绍一下`SOFTICE`？

**蟑螂**　1999-05-04 13:58:22

把金山词霸的`cjktl95.dll`用`tdump`分析可以看到它根本用的不是`hook`，而是用了一些`kernel32.dll`中的`Win32 SDK`里没有包含的函数，使用这些函数来替换改写原先的几个`GDI`函数。看来是`Win95`未公开的一些东西。有些人知道，为什么不肯说出来呢？未必见得多么高深！

**葫芦**　1999-05-05 23:28:07

你可能没有研究过它的目标代码，怎么能断言没有使用`hook`呢？ 其实金山词霸一运行就安装了鼠标钩子。

**亦东**　1999-05-07 09:52:42

未必见得多么高深？

你知道有些人是怎么知道这些`Win95`未公开的东西的吗？你不会是以为微软偷偷告诉他们的吧？其实他们大多和我一样是用`softice`自己跟踪`Win95`跟出来的。

还有你从`cjktl95.dll`里`tdump`出的未公开函数和`hook`无关，那是做别的用的。

`setwindowshook`在`user`里

**亦东**　1999-05-10 16:16:14

从`cjktl95.dll`里`tdump`出的未公开函数和 32 位和 16 位之间的互调有关。

**主题**　屏幕取词技术系列讲座（二）

**作者**　亦东

很抱歉让大家久等了！

我看了一些人的回帖，发现很多人对取词的原理还是不太清楚。

首先我来解释一下`hook`问题。词霸中的确用到了`hook`，而且他用了两种`hook`其中一种是`Windows`标准`hook`,`通过 SetWindowHook`安装一个回调函数，它安装了一个鼠标`hook`，是为了可以及时响应鼠标的消息用的和取词没太大关系。

另一种钩子是`API`钩子，这才是取词的核心技术所在。他在`TextOut`等函数的开头写了一个`jmp`语句，跳转到自己的代码里。

你用`softice`看不到这个跳转语句是因为它只在取词的一瞬间才存在，平时是没有的。

你可以在`TextOut`开头设一个读写断点

```
bpm textout
```

再取词，就会找到词霸用来写钩子的代码了。

```
/*..................................................................*/
```

所以我再次强调，想学这种技术一定要懂汇编语言和熟练使用`softice`.

```
/*..................................................................*/
```

至于从`cjktl95`中`dump`出来的未公开函数是和`Windows32/16`混合编程有关的，以后我会提到他们。

我先来讲述取词的过程，

0 判断鼠标是否在一个地方停留了一段时间
1 取得鼠标当前位置
2 以鼠标位置为中心生成一个矩形
3 挂上`API`钩子
4 让这个矩形产生重画消息
5 在钩子里等输出字符
6 计算鼠标在哪个单词上面，把这个单词保存下来
7 如果得到单词则摘掉`API`钩子，在一段时间后，无论是否得到单词都摘掉`API`钩子
8 用单词查词库，显示解释框。

很多步骤实现起来都有一些难度，所以在中国可以做一个完善的取词词典的人屈指可数。

其中 0,1,2,7,8 比较简单就不提了。

先说如何挂钩子：

所谓钩子其实就是在`WindowsAPI`入口写一个`JMP XXXX:XXXX`语句，跳转到自己的代码里。

步骤如下：

1. 取得`Windows API`入口，用`GetProcAddress`实现
2. 保存`API`入口的前五个字节，因为`JMP`是`0xEA`，地址是 4 个字节
3. 写入跳转语句

这步最复杂

`Windows`的代码段本来是不可以写的，但是`Microsoft`给自己留了个后门。

有一个未公开函数是`AllocCsToDsAlias`，

```
UINT WINAPI ALLOCCSTODSALIAS(UINT);
```

你可以取到这个函数的入口，把`API`的代码段的选择符（要是不知道什么是选择符，就先去学学保护模式编程吧）传给他，他会返回一个可写的数据段选择符。这个选择符用完要释放的。用新选择符和`API`入口的偏移量合成一个指针就可以写`windows`的代码段了。

这就是取词技术的最核心的东东，不止取词，连外挂中文平台全屏汉化都是使用的这种技术。现在知道为什么这么简单的几句话却很少知道了吧？因为太多的产品使用他，太多的公司靠他赚钱了。

这些公司和产品有：中文之星，四通利方，南极星，金山词霸，实达铭泰的东方快车，`roboword`，译典通,即时汉化专家等等等等。。。。还有至少 20 多家小公司。他们的具体实现虽然不同，但大致原理是相同的。

我这些都是随手写的，也没有提纲之类的东西，以后如果有机会我会整理一下，大家先凑合着看吧！xixi...

**葫芦**　1999-05-06 14:52:30

你说的这个技术是 16 位的,至少金山词霸 III 没有采用 16 位的`AllocCsToDsAlias`函数,也根本没有修改`TextOut`开始的语句为`JMP XXXXXXXX`。`softice`本人非常精通，是绝对不会错的！

**葫芦**　1999-05-06 15:38:40

谁假冒吾名，坏吾名声？！本人在此郑重宣布，以上贴子非本人所发！

经过跟踪分析，金山词霸 III 确实修改了`TextOut`的入口为转跳到自身代码的`JMP XXXXXXXX`语句，只是修改没有调用`AllocCsToDsAlias`，也不是在取字时才去修改，而是常常修改（估计是在`Timer`中进行的）。

**亦东**　1999-05-06 17:25:23

我说过词霸里用`AllocCsToDsAlias`了吗？

我是说`AllocCsToDsAlias`，的确可以用，但词霸里是用`DPMI`的`Int 31`做的，其实本质是一样的，你用`softice`跟一下就知道了

`bpint 31` 设一个中断断点。

实际上我至少有五种方法写`Windows`代码段

我介绍使用`AllocCsToDsAlias`是因为这种方法最简单，其他方法要麻烦得多。

词霸用`int 31`来做是出于兼容性的考虑，

`AllocCsToDsAlias`毕竟不是公开函数，但`DPMI`却是标准。

词霸在你鼠标在某一点停留超过 200ms 时就会取词，所以他有一个定时器，会经常修改`API`入口。

看来“假”葫芦只是自以为非常精通`softice`.

**mao**　1999-05-06 19:29:28

在微软的`MSDN`中有一个程序，包含了全部的`Source code`, 名字好象叫"`Inject"`或`stealth`什么的，忘了。提供了一个很完善的`hook`任何一个`windows`函数的功能。对`win16`完全适用，在`win95`下也有用，但`NT`下不行。

建议大家去找找这个例子看看会很有帮助。

此外有本清华出的微软的`Advanced Windows`也介绍了具体的方法。

建议亦东干脆把`source code`给`open`出来，让有兴趣的朋友用起来更方便，把取词技术可发展的应用发扬光大!

**亦东**　1999-05-07 09:40:56

我不主张“把取词技术可发展的应用发扬光大”，这也是我掌握了这项技术一年多才公开它的原因。我公开它并不是想让大家都来做字典软件，相反我希望不要再有人做字典了，现在做字典的人已经太多了。看到某种软件有利可图，大家就一哄而上，这种恶性竞争对中国软件业的发展是极其有害的。我公开他的目的是希望提高大家的编程水平。你会发现在研究这项技术的过程中你的编程水平和对`Windows`的理解程度会有质的飞跃，我本人就从中获益匪浅。

`MSDN`中是有这样的代码。

甚至有一个叫`ProcHook.Dll`，提供`SetProcAddress`之类的函数，但是没有源码。源码在 1994 年的`MSJ`上很难弄到的。

源码我会分几次公开，每次会有详细的说明。

想要源码的人，准备一份`VC++1.52`或`Borland C++`，最好还有`softice`，下次我会给出一段代码，教你如何修改`Windows`的代码。

**老冒** (555036)　1999-05-07 11:56:46

如果认为屏幕取词的应用就是做字典，就大错特错了。其实关于拦劫`windows api`的东东早就在 93 年的`Undocument Windows`上公开过了。

其实`Adobe`的`Adobe Type Manager`在`Windows 3.0`的时代就通过这种办法实现了漂亮的字体. (现在有`TTF`不需要`ATM`了)

`MSDN`上的那个东东是有全部`source`和`sample`, 我抓下来编译过。是 1996 年夏天的一张`MSDN Level 2`光碟上的，现在也不知搁哪里了，有兴趣的朋友自己找去吧。

还是`open`完整的`source`好，很多朋友其实只要用这项技术，并不太想知道细节，不是吗.

**亦东**　1999-05-07 13:28:19

这种技术不是做字典全屏汉化就是外挂语言平台，自从王志东使用它以来，就没用来编过其他软件，也许有但我不知道。

原来有那么多书和其他资料上都有这种技术的资料还有例子，到是我孤陋寡闻了，以为大家都不知道，在这里给大家讲一些众所周知的东西。回去我要好好研究一下，看看`Rasir Dex`的词霸是从那里抄的。

我不知道某项技术中细节是不是重要，如果很多人只想用而不想自己编，那么楼下那 50 多个回帖是怎么回是？

**老冒**　1999-05-07 13:54:18

呵呵，你可千万要坚持把讲座做下去，否则那 50 多个回应的哥们企不要把我给痛扁了...:)

俺已两年多不碰底层技术的,这方面很落后啦...俺可应付不了这么多热切的求知朋友，亦东要顶住呵!

欢迎和俺多多交流探导!

`P.S.` 亦东大侠目前何方高就？ 正在忙什么项目？交流交流

**亦东**　1999-05-07 17:46:06

没什么正经事做，到处瞎混呢！

**葫芦**　1999-05-08 21:50:29

我对此持否定观点，不要自作聪明，以为`AllocCsToDsAlias`就是能用的，其实`AllocCsToDsAlias`只是 16 位的`Windows`用的函数，32 位的`Win95`程序不能使用此函数，不信你在`VC 5.0`或`6.0`中可以试试。

另外，`int 31h`也不是说能用就能拿来用的，在`Windows 3.x`下使用是没有问题的（本人还有这方面的文章发表），但在`Win95`下随意使用会产生`GP`错，主要原因是 32 位并不支持`DPMI`直接调用，不知亦东先生对此有没有研究，就在此发表诸多理论!本人就先请问：32 位程序如何调用 16 位函数或动态库你懂不懂？

**葫芦**　1999-05-09 02:39:10

上面的这个帖子并没有攻击谁的意思，只是希望大家探讨问题都要本着认真的态度，不要不懂装懂，至少有一点大家要清楚：32 位的程序根本不能使用`int 31h`，调用 16 位的动态库`Kernel`中的`AllocCsToDsAlias`也并不是件简单的事。

`nn\_zdm` (555031742)　1999-05-09 16:35:35

使用`hook`函数，可用的功能并非只是做字典全屏汉化和外挂语言平台。使用`hook`可以调试程序，就象你们说的`softice`其本身也是使用了`hook`函数。

`nn\_zdm` (555031742)　1999-05-09 16:42:05

另外`hook`函数还可以使用在游戏修改工具中，本人就开发过此类工具。《整人专家》估计也是使用这种方法。当然还有另外两种方法。

**亦东**　1999-05-10 16:10:59

你们说的都有道理。

但`hook`有两种，一种是`Windows`标准钩子，通过`SetWindowshook`挂。

另一种是非标准的，通过在`API`入口写`JMP XXXXXXXX`来实现的。

`softice`的钩子更高级，他都挂到`VXD`上了。

从 32 为代码调用 16 位`DLL`碰巧我会。

打倒米 D 国主义！！！

**瓜果**　1999-05-10 17:07:00

谁知道在哪能搞到`SOFTICE`，我以前从未用过它！

**葫芦**　1999-05-10 21:39:46

愿继续拜读你以后的讲座。

`SOFTICE`吗？光盘上很多，有`for DOS`、`for Windows95`、`for Windows NT`各个版本。

**孙玮** (555031339)　1999-05-11 11:08:35

能否将`si for NT`上传到`10.82.46.33`

(使用`ftp`) `user: haotao`

`pass: haotao123`

**tommy**　1999-05-11 11:34:11

`http://www.swww.com.cn/htm/down/others/main.html` 可以下载

**亦东**　1999-05-11 14:01:19

最近忙于反美，暂时没时间再写了，过些时间才行，下次我会给出源码。

最新消息，美国海军被黑了。

`http://www.nctsw.navy.mil/`

打倒米 D 国主义！！！

**黄金狮子**　1999-05-12 13:19:32

我对各位大虾的讨论深感兴趣。

有几个问题想请教：



1. `AllocCsToDsAlias` 在 32-bit 下调用是否采用 `Thunk`？
2. 32-bit 下是否有类似 `function`？
3. Jeffrey Richter 的 "Advanced Windows" 里 `Remote Thread` 的 `Thread Stack` 来远程注入 `DLL` 函数，因此不需上述 `Function`。
4. 我有 `MSJ 1994-1` 的 `ProcHook.dll` 的 source code，不知用于 `WinNT` 需如何改动。
5. 总而言之，有无 `WinNT` 下 hook `API` 的 source code，请告知。

（我还有 `Softice 3.24 for Win95`，`3.25 for WinNT`。）

--这个主题很好

鼹鼠　1999-05-14 09:41:20　

请大虾给我发一份 `MSJ 1994-1` 的 `ProcHool.dll` 的 source code，我现在急需这方面的资料。谢谢！！

Email Address: `yanshg@263.net`

下面是一个 Australia 人的 `API` hook 软件，它是基于 `VxD` 技术。

`Molten` Home Page:

`http://ourworld.compuserve.com/homepages/molten`

黄金狮子　1999-05-14 14:34:54　

在 `Win95` 和 `NT` 上，可通过 `WriteProcessMemory()` 直接写 `Code Segment`。（原来以为 "advanced Windows" 调 `CreateRemoteThread()`，是因为 `WriteProcessMemory()` 只能写代码段和堆栈段）

阿涛　1999-05-14 19:45:05　

如何亦东老兄要分步公布是 `PROCHOOK` 的代码就不必了，这个程序的代码很容易搞到，只须到 `MSJ` 的站点上查一下就可找到。

亦东　1999-05-14 21:44:39　

大家到 `msj` 的大海里去捞针好了。

在 `95` 你 `WriteProcessMemory` 写 `kernel` `user` `gdi` 试试，一定失败。

调 `CreateRemoteThread` 并不是为了写代码段，有别的用途，是为了在其他进程里分配内存。回去再好好看看 "advanced Windows" 最好用一个程序试试，你就明白了。

其实在 `NT4.0` 调 `CreateRemoteThread` 是没必要的，这是为了兼容 `NT3.51`。

`nn_zdm (555031742)`　1999-05-18 13:57:48　

利用 `CreateRemoteThread()` 函数，在 `WinNT4.0` 中使用很有用，它是在 `winNT` 中闯过进程边界的三种办法之一，在 `WinNT` 中，使用它可以进行远程调试，及修改他人代码。

`nn_zdm (555031742)`　1999-05-18 14:07:17　

在 `winNT` 中用 `WriteProcessMemory()` 写 `code` 代码是可以的，`win95` 没试过，但 `MSDN` 上说是可以的。不过，可能没什么用，因为 `CreateRemoteThread()` 函数只在 `winNT` 中有用。

如:

```
WriteProcessMemory(...,"LoadLibrary(...,"mydll.dll",..).
CreateRemoteThread(...)
```

`nn_zdm (555031742)`　1999-05-18 14:14:19　

上面写漏了，应是

```
WriteProcessMemory(...,"LoadLibrary(...,"mydll.dll",..). ");
CreateRemoteThread(...) ;
```

主题 关于屏幕取词的讨论(三)

作者 亦东

让大家久等，很抱歉，前些时候工作忙硬盘又坏了，太不幸了。

这回来点真格的。

咱们以截取 `TextOut` 为例。

下面是代码：

```
//截取 TextOut
typedef UINT (WINAPI* ALLOCCSTODSALIAS)(UINT);
ALLOCCSTODSALIAS AllocCsToDsAlias;
BYTE NewValue[5];//保存新的入口代码
BYTE OldValue[5];//API 原来的入口代码
unsigned char * Address=NULL;//可写的 API 入口地址
UINT DsSelector=NULL;//指向 API 入口的可写的选择符
WORD OffSetEntry=NULL;//API 的偏移量
BOOL bHookAlready = FALSE; //是否挂钩子的标志

BOOL InitHook()
{
    HMODULE hKernel,hGdi;
    hKernel = GetModuleHandle("Kernel");
    if(hKernel==NULL)
        return FALSE;
    AllocCsToDsAlias = (ALLOCCSTODSALIAS)GetProcAddress(hKernel,"AllocCsToDsAlias");//这是未公开的 API 所以要这样取地址
    if(AllocCsToDsAlias==NULL)
        return FALSE;
    hGdi = GetModuleHandle("Gdi");
    if(hmGdi==NULL)
        return FALSE;
    FARPROC Entry = GetProcAddress(hGdi,"TextOut");
    if(Entry==NULL)
        return FALSE;
    OffSetEntry = (WORD)(FP_OFF(Entry));//取得 API 代码段的选择符
    DsSelector = AllocCsToDsAlias(FP_SEG(Entry));//分配一个等同的可写的选择符
    Address = (unsigned char*)MK_FP(DsSelector,OffSetEntry);//合成地址
    NewValue[0]=0xEA;
    *((DWORD*)(NewValue+1)) = (DWORD)MyTextOut;
    OldValue[0]=Address[0];
    *((DWORD*)(OldValue+1)) = *((DWORD*)(Address+1));
}

BOOL ClearHook()
{
    if(bHookAlready)
        HookOff();
    FreeSelector(DsSelector);
}

BOOL HookOn()
{
    if(!bHookAlready){
        for(int i=0;i<5;i++){
            Address[i]=NewValue[i];
        }
        bHookAlready=TRUE;
    }
}

BOOL HookOff()
{
    if(bHookAlready){
        for(int i=0;i<5;i++){
            Address[i]=OldValue[i];
        }
        bHookAlready=FALSE;
    }
}

//钩子函数，一定要和 API 有相同的参数和声明
BOOL WINAPI MyTextOut(HDC hdc,int nXStart,int nYStart,LPCSTR lpszString,UINT cbString)
{
    BOOL ret;
    HookOff();
    ret = TextOut(hdc,nXStart,nYStart,lpszString,cbString);//调原来的 TextOut
    HookOn();
    return ret;
}
```

上面的代码是一个最简单的挂 `API` 钩子的例子，我要提醒大家的是，这段代码是我凭记忆写的，我以前的代码丢了，我没有编译测试过。

因为我没有 `VC++1.52`，所以代码可能会有错。

建议使用 `Borland c++`，按 `16` 位编译。

如果用 `VC++1.52`，则要改个选项。

在 `VC++1.52` 的 `Option` 里，有个内存模式的设置，选大模式，和 "DS!=SS DS Load on Function entry."，切记，否则会系统崩溃。

有什么不明白的可以给我写信 `yeedong@163.net`

`Guest`　1999-05-21 22:20:47　

你这是 `16` 为的地址存取模式吧。

你看这个 `MK_FP`，`win32` 不用了，

而且 `GetProcAddress(hKernel,"AllocCsToDsAlias")`

这个 `API` 有用吗？

sorry

`Guest`　1999-05-21 22:31:47　

亦东，想请教一个问题，

`win32` 下，每个 `process` 有自己的地址空间，`process A` 得到 `process B` 的一个窗口 `C` 的 `handle`，这个 `handle` 的值等于 `process B` 自己得到的 window `C` 的 `handle` 值吗？

我想应该不相等，但系统是如何转换的呢？（比如 `process A` 向 window `C` 发消息，系统如何知道 `process A` 里的 `handle` 和 `process B` 里的 `handle` 都是指的 window `C`）

. 是不是用 `duplicatehandle()`？

（声明，我是真的不知道 `4dd466d0e1e0cc123afbe504da2462ca`）
