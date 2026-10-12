
# 挣扎的菜鸟 - 当 OD 不能装载也不能附加程序时

> 原文：[`www.pediy.com/kssd/pediy12/132199.html`](https://www.pediy.com/kssd/pediy12/132199.html)

## 文章信息

- 【文章标题】: 挣扎的菜鸟 - 当 OD 不能装载也不能附加程序时
- 【文章作者】: justlovemm
- 【作者邮箱】: 无
- 【作者主页】: 无
- 【作者 QQ 号】: 无
- 【下载地址】: 自己搜索下载
- 【保护方式】: `VMP2.07`
- 【使用工具】: `OD`, `LoadPE`, `CE`, `Exception Monitor`
- 【作者声明】: 只是感兴趣，没有其他目的。只是一个菜鸟对几年前大牛们就搞定的问题的一次练习，请大牛们无视本文。

## 正文

前几天拿到一个程序 `X`，用 `PEiD` 查不到任何信息，用 `exeinfope` 说是 `VMP2.07`。于是先用 `OD` 载入，结果 `OD` 立即直接被关闭。开始认为 `TLS` 在作怪，用 `LoadPE`，删除掉 `TLS`，保存。再用 `OD` 载入，还是一样。下来先直接运行程序，然后用 `OD` 附加，结果 `OD` 也是立即直接被关闭。

检查 `OD` 的选项，`first pause` 是在 `system breakpoint` 处(其实自 `TMD` 利用那个浮点漏洞关闭 `OD` 以后，我的 `OD` 一直是启动时停在 `system breakpoint` 的)。打开 `StrongOD` 的选项，`break On TLS` 也是打上了勾的。这时仔细一想，不对啊，现在的设置应该能截获 `TLS` 的运行，这说明不是 `TLS` 的问题。

下来考虑是否 `plugin` 有问题，于是删除了 `plugin` 目录下的所有文件，结果还是一样。

现在菜鸟有点失望了，于是开始在论坛里找关于 `VMP2.07` 的资料，无果。查找 `OD` 被关闭，基本上就 2 条参考意见，一个是删除 `plugin` 试试，另一个是删除掉 `OD` 目录下的 `dbghelp.dll` 文件。于是直接试第二个方法，删除掉了 `OD` 目录下修改时间为 2001 年 11 月 12 日的 `dbghelp.dll` 文件，测试，结果还是一样。于是又把 `system32` 目录下的修改时间为 2005 年 5 月 26 日的 `dbghelp.dll` 拷贝到 `OD` 目录下，其实菜鸟心里也很清楚，这和刚试过的方法没有什么不同的，不过这个时候菜鸟已经有点绝望了，所以这样明知道不会有任何作用的方法菜鸟也要去试试。结果就不说了。

也许是这个时候命不好，菜鸟竟然没有搜索到海风大牛关于 `dbghelp.dll` 文件的那个原帖，如果当时看到那个帖子，直接下载帖子里的 `dbghelp.dll`，就不会有这些问题了。

接着菜鸟在 `UPK` 发了一个帖子求助，热心人很多，不过菜鸟太菜，没能搞定。

下来改用 `CE` 装载，`OK`，可以装载。然后运行，提示发现调试器。菜鸟据此判断，此 `anti` 一定是针对 `OD` 的。不过菜鸟不会用 `CE` 调试程序，至少是不会象用 `OD` 那样去用 `CE` 调试程序。

现在，菜鸟已经绝望了。以前 `TMD` 不能加载的时候，菜鸟只能等待，不过很快，大牛们就给出了解决方法，菜鸟只是照着去做就 `OK` 了。可是现在菜鸟觉得很无助，不知道去哪里寻找帮助，觉得四周一片漆黑。绝望的菜鸟只能一个人在那儿苦苦思索，过了几个小时，菜鸟突然想到一个方法，可以用 `OD` 来调试 `OD`，然后用被调试的 `OD` 去装载程序 `X`，这样应该就能知道 `OD` 是怎么被关闭的了。菜鸟当时心里还有点激动，心想，也许一个新的 `anti OD` 的方法就要让本菜鸟给搞定了，哈哈。

立即动手，用 `OllyICE` 加载一个 `OllyDbg`，用 `OllyDbg` 加载程序 `X`，`OllyDbg` 立即被关闭，去 `OllyICE` 中查看 `OllyDbg` 退出时的栈，没有得到任何有用的线索。

再试，用 `OllyICE` 加载一个 `OllyDbg`，然后在 `CreateProcessA` 和 `ResumeThread` 处下断，用 `OllyDbg` 加载程序 `X`，断在 `CreateProcessA` 处，`F8` 过去，`OllyDbg` 正常，再 `Shift+F9`，断到 `ResumeThread` 处，再 `F8` 过去，然后调出任务管理器，看到进程 `X` 已经有一点 `CPU` 的占用率了，这时 `OllyDbg` 还未被关闭，再 `Shift+F9`，`OllyDbg` 直接被关闭，查看退出的栈，还是没有任何线索。

这时候先推理到，`OllyDbg` 当时是一被调试的进程，除了调试进程以外，其它进程是无法关闭 `OllyDbg` 的，菜鸟猜想是否给 `OllyDbg` 窗口发送了 `WM_QUIT` 或者 `WM_CLOSE` 消息。于是在 `OD` 中查找 `GetMessage`，未发现，查找 `PeekMessage`，找到好几个，根据传入的消息的上下限，找到了 `OllyDbg` 的窗口过程中使用的 `PeekMessage`，在 `PeekMessage` 返回后的一个地方下条件断点，当消息 `ID` 是 `WM_QUIT` 或者 `WM_CLOSE` 时触发断点。再次载入程序 `X`，`OllyDbg` 同样被关掉，并未触发任何条件断点。

这样就排除了程序 `X` 发送消息给 `OllyDbug` 窗口的可能性。这时菜鸟认定 `OllyDbg` 在装入程序 `X` 的时候本身产生了什么不可捕获的异常，导致程序直接被关闭了。问题是怎么才能知道这个异常发生在什么地方呢？

此时的菜鸟又一次陷入了绝望，又觉得四周一片漆黑。

在黑暗中孤独的思索了几个小时后，菜鸟突然想起以前用过的 `Exception Monitor`，是 `M$` 的产品，当时是为了调试服务进程而推出的东东。用这个东西能捕获最后一次异常发生时的线程信息，包括各个寄存器和栈数据。上网搜索，还是命不好，竟然找不到 `MS` 的 `Exception Monitor` 的下载地点。不过，也意外的发现还有好几个类似的产品，一个叫 `EMon.exe` 的还是开源的，于是下载了这个 `EMon.exe`。

打开 `OllyDbg`，用 `EMon.exe` 去 `attach`，在进程列表里没有发现 `OllyDbg`。呵呵，是 `StrongOD` 隐藏了 `OllyDbg` 进程，删除掉 `SOD`，`OK`，用 `EMon.exe` `attach` `OllyDbg`，再用 `OllyDbg` 载入程序 `X`，然后看到 `EMon.exe` 的 `Log` 窗口哗哗哗的闪，计算机不停的叫"叭嗒叭嗒"，过了有 2、3 分钟，停了下来，`OllyDbg` 已经不见踪影了。

`EMon.exe` 最后的 `Log` 是这样的:

```
下午 04:00:09: Process terminated with exit code 3221225477
下午 04:00:09: End of Process: 4076 - OllyDBG.EXE
下午 04:00:09: End of Process: 2380 - X.exe
```

接着往上看，最后出错信息是这样的：

```
下午 04:00:09: ACCESS_VIOLATION at address 7C92EDDD
** The thread tried to read from or write to a virtual address
   for which it does not have the appropriate access.
** Thread ID=00000F34
** Access violation writing address 00030FFC
** First chance
** Register Dump:
    EAX=0000000C
    EBX=7C920000 : 00905A4D 00000003 00000004 0000FFFF
    ECX=7C930833 : 900004C2 FFFFFF90 95C5FEFF 95C6077C
    EDX=000000E0
    EDI=7C9237D8 : 04244C8B 060441F7 B8000000 00000001
    ESI=7C920000 : 00905A4D 00000003 00000004 0000FFFF
    ESP=00031000 : 7C920000 00000000 00000000 00000000
    EBP=00031020 : 00031030 7C93086E 7C920000 00031400
    EIP=7C92EDDD : 458B5756 E86589F8 FC458B50 FFFC45C7
** Stack Dump:
    00031000: 7C920000 : 00905A4D 00000003 00000004 0000FFFF
    00031004: 00000000
    00031008: 00000000
    0003100C: 00000000
    00031010: 00031464 : 00031834 7C9237D8 0012B060 00031520
    00031014: 7C92EE18 : 83EC8B55 565308EC 8BFC5557 458B0C5D
    00031018: 7C9307F5 : 4D8BC033 74C98508 FFF98330 45212B74
    0003101C: 7C930838 : FFFFFFFF 7C95C5FE 7C95C607 90909090
    00031020: 00031030 : 0003104C 7C9579F0 7C920000 00000001
    00031024: 7C93086E : 840FC085 000025BC 18488B66 0BF98166
    00031028: 7C920000 : 00905A4D 00000003 00000004 0000FFFF
    0003102C: 00031400 : C0000034 00000000 00000018 00E8DAA0
    00031030: 0003104C : 00031080 7C95799C 7C920000 0003107C
    00031034: 7C9579F0 : 840FC085 000001D8 85FC4D8B CD840FC9
    00031038: 7C920000 : 00905A4D 00000003 00000004 0000FFFF
    0003103C: 00000001
** Disassembly listing
    7C92EDDD: 56               push            esi
    7C92EDDE: 57               push            edi
    7C92EDDF: 8B45F8           mov             eax, [ebp-0x08]
    7C92EDE2: 8965E8           mov             [ebp-0x18], esp
    7C92EDE5: 50               push            eax
    7C92EDE6: 8B45FC           mov             eax, [ebp-0x04]
    7C92EDE9: C745FCFFFFFFFF   mov             [ebp-0x04], 0xFFFFFFFF
    7C92EDF0: 8945F8           mov             [ebp-0x08], eax
    7C92EDF3: 8D45F0           lea             eax, [ebp-0x10]
    7C92EDF6: 64A300000000     mov             0x00000000, eax
** End of exception report
```

`7C92EDDD` 是在 `ntdll` 中的，菜鸟不知道这个是怎么调用来的。不过在 `OllyICE` 中看了一下 `fs:[8]`, 是 `00031000`，而上面的出错信息的 `ESP` 也是这个值，这说明线程栈已经已经用光了，程序无法再运行了。

再往上看，下来的出错信息好像都一样，都是下面这个样子的：

```
下午 04:00:09: ACCESS_VIOLATION at address 32C21FAE
** The thread tried to read from or write to a virtual address
   for which it does not have the appropriate access.
** Thread ID=00000F34
** Access violation reading address 32C21FAE
** First chance
** Register Dump:
    EAX=00000000
    EBX=00000000
    ECX=32C21FAE
    EDX=7C9237D8 : 04244C8B 060441F7 B8000000 00000001
    EDI=00000000
    ESI=00000000
    ESP=00031450 : 7C9237BF 00031538 0012B060 00031554
    EBP=00031470 : 00031520 7C92378B 00031538 0012B060
    EIP=32C21FAE
** Stack Dump:
    00031450: 7C9237BF : 00258B64 64000000 0000058F E58B0000
    00031454: 00031538 : C0000005 00000010 00000000 32C21FAE
    00031458: 0012B060 : 8525521F 32C21FAE 1D2F676B ABC87504
    0003145C: 00031554 : 0001003F 00000000 00000000 00000000
    00031460: 0003150C : 0012B060 00000000 00130000 00031000
    00031464: 00031834 : 00031C04 7C9237D8 0012B060 000318F0
    00031468: 7C9237D8 : 04244C8B 060441F7 B8000000 00000001
    0003146C: 0012B060 : 8525521F 32C21FAE 1D2F676B ABC87504
    00031470: 00031520 : 00031840 7C92EAFA 0012B060 00031554
    00031474: 7C92378B : C25B5E5F 8B900014 909090FF 8B559090
    00031478: 00031538 : C0000005 00000010 00000000 32C21FAE
    0003147C: 0012B060 : 8525521F 32C21FAE 1D2F676B ABC87504
    00031480: 00031554 : 0001003F 00000000 00000000 00000000
    00031484: 0003150C : 0012B060 00000000 00130000 00031000
    00031488: 32C21FAE
    0003148C: 00000002
** End of exception report
```

每次出错的地址都是 `32C21FAE`，用上面的 `OllyICE` 调试 `OllyDbg` 并断在 `ResumeThread` 之后，查看，这个地址是空的。菜鸟觉得很怪异。

继续向上看，都是 `32C21FAE`。于是直接翻到到最上面的 `Log`，才发现只有第一个不是 `32C21FAE`，第一个出错的地址是 `68D808BF`，信息如下：

```
下午 03:57:58: ACCESS_VIOLATION at address 68D808BF
** The thread tried to read from or write to a virtual address
   for which it does not have the appropriate access.
** Thread ID=00000F34
** Access violation reading address 8C699B73
** First chance
** Register Dump:
    EAX=8C699B73
    EBX=023FBF58 : 00000000 00000000 00000000 0040B987
    ECX=8C291638
    EDX=73A90925
    EDI=023F9160 : 00000000 00000000 00400000 00000000
```



`ESI=023FA090 : 023FAA48 00000000 00400000 00000000`

`ESP=0012A460 : 023F9160 80000000 023FA090 023FC3E4`

`EBP=0012ACAC : A1A8ECCE E2B7A856 9C91972D C09558D3`

`EIP=68D808BF : 0C88088A C9844002 8D38F675 FFFFF7EC`

**Stack Dump:**

```
0012A460: 023F9160 : 00000000 00000000 00400000 00000000  
0012A464: 80000000  
0012A468: 023FA090 : 023FAA48 00000000 00400000 00000000  
0012A46C: 023FC3E4 : 000A0118 00040006 00020008 001A000F  
0012A470: 023FC616 : 0040791E 0040853B 0040855D 00408581  
0012A474: 023FBF58 : 00000000 00000000 00000000 0040B987  
0012A478: 023FBF80 : 0000158D 00001F0C 0000D640 0000D778  
0012A47C: 023FBF58 : 00000000 00000000 00000000 0040B987  
0012A480: 00343158 : 00580058 0070002E 00620064 00000000  
0012A484: 00000124  
0012A488: 02400AF8 : 00000000 00000000 00000000 00000000  
0012A48C: 000000B8  
0012A490: 00000001  
0012A494: 00000001  
0012A498: 7EA7BC00  
0012A49C: F11FC1BF  
```

**Disassembly listing**

```
68D808BF: 8A08             mov         cl, [eax]  
68D808C1: 880C02          mov         [edx+eax], cl  
68D808C4: 40              inc         eax  
68D808C5: 84C9            test        cl, cl  
68D808C7: 75F6            jnz         +0x000000F6  
68D808C9: 388DECF7FFFF    cmp         [ebp-0x00000814]  
68D808CF: 7473            jz          +0x00000073  
68D808D1: F6059804DF6802  test        0x68DF0498, 0x00000002  
68D808D8: 8D85ECF7FFFF    lea         eax, [ebp-0x00000814]  
68D808DE: 8D5001          lea         edx, [eax+0x01]  
```

**End of exception report**

在 `OllyICE` 中查看了一下 `68D808BF`，在 `DbgHelp.dll` 中。

`OK`，重新用 `OllyICE` 装载 `OllyDbg`，在 `68D808BF` 处下断，然后装载程序 `X`，成功断下。在数据窗口跳到 `eax`，`eax` 地址有效，看了一下代码，这个应该是一个字符串拷贝的循环，删除 `68D808BF` 处的断点，在循环结束后的 `68D808C9` 下断，`F9` 正常运行到 `68D808C9`，然后再 `F9`，就跑飞了。

为了捕获异常不让 `OllyDbg` 跑飞，再重新来一次，在 `68D808BF` 断下后，取消断点，跳到 `fs:[0]`，对当前 `SEH` 的 handler `68D90888` 下硬件断点，`F9`，还是跑飞了。跑飞的时候好像还提示了那个不存在的地址 `32C21FAE`，一看 log，是"32C21FAE|访问违规: 正在执行 [32C21FAE]"，很奇怪，为什么在跳到这个不存在的地址之前没有去执行 `SEH` 的 hander `68D90888`。

`无奈`，再来，再次断到 `68D808BF` 处，`F9` 了几次，每次都又断到 `68D808BF`，这时查看了一下后面一行代码中的 `edx+eax` 竟然等于 `12A498`，哈哈字符串拷贝的目标地址在当前栈上，在数据窗口跳到 `EAX(02363102)`，看一下，没有发现 `00`，二进制搜索 `00`，在 `02363D1E`，原来 `EAX` 指向的字符串长达 `0xC1C`，就是 `3100` 字节长，而根据前面的代码

```
68D808B7    8D95 ECF7FFFF   lea     edx, dword ptr [ebp-814]
```

`edx` 在栈上最大的可用空间也只能是 `0x814`，这肯定是把线程的栈数据给覆盖了。重新测试一下，当断到 `68D808BF` 处时，在数据窗口跳去 `fs:[0]`，然后 `F4` 到 `68D808C9`，这时数据窗口的数据已经完全改变了，就是说 `SEH` 的链和 hander 已经被覆盖了，而 hander 就是被修改为那个不存在的 `32C21FAE`。

这时，菜鸟已经明白了，那个字符串拷贝的循环应该就是一个 `inline` 化的 `strcpy`，由于没有限制拷贝长度，造成栈数据被覆盖掉了，而新生成的 Exception Handler 是无效的，结果就又触发了新的异常，进入了一个异常循环，直到栈被用光，`OD` 就结束了，连错误提示都没有，让人以为 `OD` 是被正常关闭了。

搞清楚这个以后，菜鸟又在想，这个分明就是一个异常，为什么用 `OllyICE` 调试 `OllyDbg` 时没有捕获这个异常呢，查看了一下 `OD` 的设置，发现是忽略了非法的内存访问异常。设置成不忽略非法的内存访问，用 `OllyICE` 装载 `OllyDbg`，然后 `OllyDbg` 装载程序 `X`，在 `OllyICE` 中就断到了 `68D808BF`，`OK`，确定了就是这个异常。

分析到这一步，本来应该再分析一下 `DbgHelp.dll`，看看是在拷贝什么信息的时候造成的栈溢出，可是这个时候菜鸟又去搜索了一下 `DbgHelp.dll`，找到了海风大牛的那个帖子，下载了那个 `DbgHelp.dll`，用 `OD` 加载程序 `X`，正常的停到了系统入口。这时菜鸟才发现自己的确成不了大牛，因为菜鸟这个时候已经不愿意再去分析 `DbgHelp.dll` 了。

最后再说一下，菜鸟因为自己在 `OD` 不能装载和附加时，经过思考发现了如何去寻找不能装载和附加的方法，菜鸟自己非常高兴，所以写了此文，为自己庆贺一下！

今天分析了一下 `dbghelp` 溢出时的具体数据，在 11 楼，请大家指正。

接着把今天对引起溢出的具体的数据的分析发出来，请大家指正。

先还是用老办法，利用 `system32` 下的老版本的 `dbghelp.dll`，双 `OD` 调试程序，在 `OllyICE` 中对 `dbghelp.dll` 中的 `68D808BF` 下断，`OllyDbg` 装载程序 `X`，这时断在 `68D808BF`，代码为

```
68D808BF    8A08             mov     cl, byte ptr [eax]
```

我们直奔 `eax`，二进制拷贝头部的 96 字节如下：

```
03 BC A7 7E BF C1 1F F1 32 E9 E2 D7 C4 B5 89 14 78 E6 1D 2C 43 B2 7C 16 B1 5E 64 52 26 EB 1E 49
53 FB 10 11 D6 26 AE 0F FC 5B 85 18 1F 11 5A FF 26 52 9B C9 19 F5 A1 D7 D0 86 7D D6 29 B5 46 3D
53 51 2F A3 16 18 D4 4A 30 F3 90 85 1F 64 B4 F7 FB 21 01 C9 35 52 CA 48 58 CD 83 91 32 47 B0 4B
```

下来，在 `OD` 的目录下拷贝进去海风大牛给的 `dbgHelp.dll`，启动 `OD`，直接装载程序 `X`，这时就停在了系统入口处，打开内存窗口，二进制搜索上面的 96 字节，在 `0080791E` 处找到，且整个内存中只找到这一个，`OK`，肯定就是这个地方了。

这时要分析一下这个地址是做什么用的了，我们打开 `IDA`，查看 `0080791E`，没有直接的引用，由于是在装载时引起的 `dbghelp.dll` 溢出的，所以，估计这个地址应该是从 `PE` 头数据中引出的，也许应该查找 `RVA 0040791E`，在 `OD` 中二进制搜索，在 `008074BA` 处发现如下：

```
008074BA  1E 79 40 00 3B 85 40 00 5D 85 40 00 81 85 40 00   y@.;.\].@.
008074CA  A6 85 40 00 CE 85 40 00 FA 85 40 00 19 86 40 00   @.@.@..
```

后面好像都是 `RVA` 值，在 `IDA` 中设置数据类型为 `RVA`，显示如下：

```
.text2:008074BA ; Export Names Table for X.exe

.text2:008074BA ;

.text2:008074BA off_8074BA        dd rva word_80791E, rva a@Xp26shdocvw_t, rva a@Xp28shdocvw_t

.text2:008074BA                                           ; DATA XREF: .text2:00806E1Co

.text2:008074BA        dd rva a@Xp29shdocvw_t, rva a@Xp32shdocvw_t, rva a@Xp36shdocvw_t ; "@$xp$26Shdocvw_tlb@TCppWebBrowser"

.text2:008074BA        dd rva a@XpYnpqqrp14sy, rva a@XpYnpqqrp14_0, rva a@XpYnpqqrp14_1

.text2:008074BA        dd rva a@XpYnpqqrp14_2, rva a@XpYnpqqrp14_3, rva a@XpYnpqqrp14_4
```

哈哈，`IDA` 已经给分析过了，是输出表的 `Names Table`，而且 `0080791E` 是该表的第一项，向上看名字对应的序号表如下：

```
.text2:00807288 ; Export Orfinals Table for X.exe

.text2:00807288 ;

.text2:00807288 word_807288       dw 118h, 0Ah, 6, 4, 8, 2, 0Fh, 1Ah, 14h, 15h, 1Fh, 18h

.text2:00807288                                           ; DATA XREF: .text2:00806E20o

.text2:00807288        dw 20h, 1Eh, 16h, 13h, 1Ch, 17h, 1Dh, 10h, 19h, 1Bh, 0Dh
```

再向上看到输出表如下：

```
.text2:00806DFC ; Export directory for X.exe

.text2:00806DFC ;

.text2:00806DFC        dd 0                     ; Characteristics

.text2:00806E00        dd 0                     ; TimeDateStamp

.text2:00806E04        dw 0                     ; MajorVersion

.text2:00806E06        dw 0                     ; MinorVersion

.text2:00806E08        dd rva aXx_exe           ; Name

.text2:00806E0C        dd 1                     ; Base

.text2:00806E10        dd 119h                  ; NumberOfFunctions

.text2:00806E14        dd 119h                  ; NumberOfNames

.text2:00806E18        dd rva off_806E24         ; AddressOfFunctions

.text2:00806E1C        dd rva off_8074BA         ; AddressOfNames

.text2:00806E20        dd rva word_807288        ; AddressOfNameOrdinals
```

就是一共有 `0x119` 个输出项，而造成溢出的输出项的序号是 `0x118`，呵呵，是最后一个，这说明 `VMP` 给输出表增加了一项，其序号在最后，而其名字的序号在第一个，只要老版本的 `dbghelp.dll` 提取输出表的名字，立即就会溢出。

搞到这个时候，又去看了一下 `IDA` 目录下是否有 `dbghelp.dll`，结果发现有，但修改日期是 2009 年 7 月 13 日，比海风大牛给的那个 `dll` 还要新一些。我的 `IDA` 的版本是 `5.5`。

下来还有一个问题就是，为什么栈上只需要覆盖多于 `0x814` 几个字节就可以使返回地址无效，可为什么 `VMP` 要把那个名字的长度设定为 `0xC1C`，查看了一下 `68D808BF` 所在的函数，函数头上没有 `SEH` 的代码，在 `OllyICE` 中直奔 `fs:[0]`，为 `0012B060`，而 `ebp-814+0C1C=0012B0B4`，在栈上看一下，`0xC1C` 个字节不但把当前 `SEH` 数据都覆盖掉了，还把下一个 `SEH` 链点的指针也覆盖掉了。不过这里还有的不太理解，其实只要覆盖第一个 `SEH` 数据，整个 `SEH` 链就已经不可用了，为什么还要把第二个 `SEH` 链点也覆盖掉。猜想也许是不同版本有漏洞的 `dbghelp.dll` 的栈空间不一样大吧，可能这个 `0xC1C` 是个满足所有可溢出覆盖掉 `SEH` 链头的 `dbghelp.dll` 的最小值吧。

最后得出结论：`VMP` 利用了老版本 `dbghelp.dll` 在拷贝输出表名称时使用 `inline` 化的 `strcpy` 同时未限制源字符串的长度的漏洞，在加壳时给目标程序增加了一个输出项，该项在输出名字表中排在第一位，该名字长达 `0xC1C` 字节，对应的输出项为最后一个输出项。而当调试分析程序(如 `OD`，`IDA`)调用 `dbghelp.dll` 分析加壳程序的输出表时，就会造成线程栈溢出，同时覆盖了当前 `SEH` 链表头，使得异常处理程序在无效的地址上，从而再次引发异常，进而形成异常循环，耗光了线程栈，引起程序终止，却没有任何提示。
