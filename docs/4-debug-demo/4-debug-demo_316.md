
# 解除“天 x 结构 7.1”时间限制的过程

> 原文：[`www.pediy.com/kssd/pediy10/82321.html`](https://www.pediy.com/kssd/pediy10/82321.html)

## 软件的简单介绍

“天 x 结构 7.1”是土木工程专业同学进行结构设计的一款较有名的软件，它是 `AutoCAD` 二次开发的产物。

## 为什么要破解，破解什么

选择“试用版”模式安装完成，使用软件时我们的系统时间得小于 `2010-01-01`，否则直接弹出如下对话框：

![](img/29d98541b07c9f493bfd5ce5979b44ae.jpg)

第一次使用时必须进行注册，此时使用“天 x 结构 7.1”安装压缩包中的注册机即可。

（成功安装后，就会有左边的工具栏，以后称此为 `T` 工具栏）

因此我们以后如果要使用，必须把时间调到 `2010-01-01` 之前。

我们要把它“变”成以后即使不调整时间也可使用：）

## 破解过程

### 阶段一：

它是 `AutoCAD` 二次开发的产物，那我们得看看 `AutoCAD` 的帮助，了解下有哪些方式开发。

1. `ActiveX Automation`
2. `AutoCAD VBA`
3. `AutoLISP` 和 `Visual LISP`
4. `ObjectARX`

那是哪一种方式呢？我们看看它安装目录下的文件，可以发现有许多 `.arx` 扩张名的文件，还有一些 `.lsp` 的文件。应此我们认定它应该是 3 方式与 4 方式的结合。

我们要解决的矛盾是：时间判断在哪个地方呢？

3 方式中？它的代码是以源代码的形式存在的，应该不会在这里面；而 4 方式开发的 `.arx` 其实是 `dll` 文件，应该这。所以我首先想到的是：对 `GetSystemTime` 函数下断点。

用 `OllyDbg` 打开软件，`AutoCAD` 都还没加载完就出现错误了，不用说加载“天 x”模块了，显然使用了反调试技术。既然这条大路行不通，那我得找其它的路了。

### 阶段二：

那我们看看加载了哪些模块，再看看 `AutoCAD` 帮助，可以知道：用“`arx`”命令可查看已加载的 `arx` 模块。可以发现“`tch_initstart.arx` - 支持 MDI”，这是“天 x”加载的第一个模块，从字面也可以看出。所以主要研究这个文件。

可是 `tch_initstart.arx` 文件在好几个文件夹下都有啊！难道每个都要分析？那可不是一件轻松的事！抱着试一试的心情再重新打开软件，突然在状态栏下看到它的加载路径。这下让我松了口气。它加载的是 `sys16` 文件夹下的文件。所以应该就只研究它一个啦，哈哈……

用 `PEID` 查了下：`Microsoft Visual C++ 7.0 DLL Method 3`

没加壳！哈哈……马上用 `IDA` 打开，查找 `GetSystemTime` 函数，果然有调用。这下让我高兴了，马上用 `IDA` 的“交叉参考到” `GetSystemTime` 功能，可看到好几个地方调用。那就配合 `HIEW` 一个个改咯，然而后面的努力……让我很失望，居然一点效果都没有！一气之下把它的 `OEP` 入口改成 `int 3`，打开“天 x”软件，然而在加载 `tch_initstart.arx` 的时候，它认为 `tch_initstart.arx` 是无效 `ARX` 文件了。

### 阶段三：

这时我想到了 `OllyDbg` 的附加（attactch to process)功能。因为“天 x”是 `AutoCAD` 的二次开发产品，所以我想可以自己手动加载：

首先，把系统时间调到 `2010-01-01` 以后

其次：打开 `AutoCAD`，用 `arx` 把 `tch_initstart.arx` 加载

再次：它应该会弹出“软件已经过期！请……”对话框。

要是成功的话，我就在可以用 `OllyDbg` 的附加功能调试 `tch_initstart.arx`（后面介绍）。

然而，并没有弹出“软件已经过期！请……”对话框，这肯定是之前还有东西要处理。

再次看看 `AutoCAD` 帮助，了解了它的启动过程：“如果打开文件目录下有 `AutoCAD` 三种用户可自定义的文件：`acad.lsp`、`acaddoc.lsp` 以及用于支持当前菜单的 `MNL` 文件，那么它会自动加载。”

我发现 `sys16` 目录下的确有 `acad.lsp` 文件，和菜单文件（非 `MNL` 文件）。

用“`appload`”命令直接加载 `acad.lsp`，然而并没有正常的打开软件，命令行提示：`TAsd.lsp` 加载失败；我在 `common` 文件夹下发现 `TAsd.lsp` 文件，再加载之。我再用“`menuload`”命令加载了 `sys16` 文件夹下的 `TCH.mnc` 文件。这时，我再用“`arx`”命令加载 `tch_initstart.arx` 成功的弹出了“软件已经过期！请……”对话框。这下好办了！

### 阶段四：

在阶段三中，我们知道了自己手动加载的方法！我们在自己加载 `tch_initstart.arx` 之前，一定要在用了“`arx`”命令的加载参数 `L` 弹出通用对话框时（这很重要），才能打开 `OllyDbg` 然后附加到 `acad.exe` 进程。我们要对 `user32!MesssageBoxA(W)` 入口下断点。至于怎么弄我不多说。

这时按 `F9` 运行，选择 `tch_initstart.arx` 加载！这时停在 `USER32!MessageBoxA` 入口处。

这时我们就要看堆栈了。此时堆栈是这样子的：

```
0271EED4   7C1CB311  /CALL 到  MessageBoxA 来自  MFC70.7C1CB30B
0271EED8   000F0118  |hOwner = 000F0118 ('AutoCAD 2004 - [Drawing1.dwg]',class='Afx:00400000:8:00010011:00000...')
0271EEDC   1C04B9FC  |Text = "软件已?,AD,"过期!请登?,BD,"天正网站:WWW.Tangent.com.cn 下载",D7,"钚率杂?,B0,"?"
0271EEE0   011D9FD0  |Title = "AutoCAD"
0271EEE4   00000030  \Style = MB_OK|MB_ICONEXCLAMATION|MB_APPLMODAL
…………
0271F024   012A5330
0271F028   1C022F64  返回到  tch_init.1C022F64 来自  <jmp.&MFC70.#1014>
0271F02C   1C04B9FC  tch_init.1C04B9FC
0271F030   00000000
0271F034   00000000
0271F038   012A5120  ASCII "Class"
0271F03C   012A5330
```

往回看，直到找到第一个返回 `tch_initstart` 模块的地址。可以找到：

```
0271F028   1C022F64  返回到  tch_init.1C022F64 来自  <jmp.&MFC70.#1014>
```

到 `tch_init.1C022F64` 看下：

```
1C022F0E    FF15 FC8B041C   call    dword ptr [<&tch_kernal.XAString::XAString>]   ; tch_kern.XAString::XAString
1C022F14    8D4C24 0C       lea     ecx, dword ptr [esp+C]
1C022F18    C68424 D0010000>mov     byte ptr [esp+1D0], 2
1C022F20    FF15 148C041C   call    dword ptr [<&tch_kernal.XAString::operator cha>; tch_kern.XAString::operator char *
1C022F26    50             push    eax
1C022F27    FF15 888E041C   call    dword ptr [<&tch_kernal.GetDate>]              ; tch_kern.GetDate
1C022F2D    83C4 04        add     esp, 4
1C022F30    8D4C24 0C       lea     ecx, dword ptr [esp+C]
1C022F34    FF15 148C041C   call    dword ptr [<&tch_kernal.XAString::operator cha>; tch_kern.XAString::operator char *
1C022F3A    50             push    eax
1C022F3B    FF15 6C87041C   call    dword ptr [<&MSVCR70.atoi>]                    ; MSVCR70.atoi
1C022F41    83C4 04        add     esp, 4
1C022F44    3D 05B43201     cmp     eax, 132B405
1C022F49    7E 30          jle     short 1C022F7B
1C022F4B    A1 A48B041C     mov     eax, dword ptr [<&tch_kernal.g_nProtectCode>]
1C022F50    6A 00          push    0
1C022F52    6A 00          push    0
1C022F54    68 FCB9041C     push    1C04B9FC
1C022F59    C700 00000000   mov     dword ptr [eax], 0
1C022F5F    E8 38050200     call    <jmp.&MFC70.#1014>
1C022F64    8D4C24 0C       lea     ecx, dword ptr [esp+C]
```

往上看,是不是这句最引人注目：

```
1C022F27    FF15 888E041C   call    dword ptr [<&tch_kernal.GetDate>]              ; tch_kern.GetDate
```

原来它是用它自己 `tch_kernal.arx` 模块中的 `GetDate` 函数获得系统时间的啊！

在看看这两句：

```
1C022F44    3D 05B43201     cmp     eax, 132B405
1C022F49    7E 30          jle     short 1C022F7B
```

其中 `132B405` 把它弄到计算器中，转化成十进制是：`20090101`

应此这两句的意思就是如果系统时间小于等于 `2009-01-01` 的话就转到 `1C022F7B` 继续执行，否则弹出：“软件已经过期！请……”对话框。

因此把 `1C022F49` 处把 `7E` 改成 `EB` 即：`jle` 改成 `jmp`

打开软件，顺利打开软件，也有 `T` 工具栏，哈哈……但是，功能却是没用啊！

用 `IDA` 打开 `tch_initstart.arx`，找到 `GetDate` 函数，由“交叉参考”可以看到还有一个地方调用了此函数，双击另一个过程（`ReVerifyUserRight`），来到此处

```
.text:1C023387             call    ds:?GetDate@@YAXPAD@Z ; GetDate(char *)
.text:1C02338D             add     esp, 4
.text:1C023390             lea     ecx, [esp+1B0h+var_1B0]
.text:1C023394             call    ds:??BXAString@@QAEPADXZ ; XAString::operator char *(void)
.text:1C02339A             push    eax         ; Str
.text:1C02339B             call    ds:atoi
.text:1C0233A1             add     esp, 4
.text:1C0233A4             cmp     eax, 132B405h
.text:1C0233A9             jg      short loc_1C023404
.text:1C0233AB             lea     ecx, [esp+1B0h+var_10C]
.text:1C0233B2             push    ecx
.text:1C0233B3             call    sub_1C022530
```

映入眼帘的是这两句：

```
.text:1C0233A4             cmp     eax, 132B405h
.text:1C0233A9             jg      short loc_1C023404
```

它的意思是如果系统时间大于 `2009-01-01`，那么跳到 `1C023404` 处，导致出错。

因此只需把 `1C0233A9` 处机器码都改成 `90` 即可。

打开软件，终于可以使用软件了。
