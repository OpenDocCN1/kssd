
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
