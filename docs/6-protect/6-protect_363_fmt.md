
# 一个奇怪的带壳 crackme 的简单分析（Anti、Validate）

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-667.htm`](https://www.pediy.com/kssd/pediy08/pediy8-667.htm)

【文章标题】: 一个奇怪的带壳 crackme 的简单分析  
【文章作者】: hawking  
【作者邮箱】: rich_hawking@hotmail.com  
【软件名称】: Bustme4  
【软件大小】: 21.5k  
【下载地址】: bm4.zip  
【加壳方式】: 不详  
【保护方式】: 未知  
【使用工具】: PEiD OLlyICE  
【操作平台】: win2k  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

## 【详细过程】

首先声明，本人菜菜，分析水平有限，本文只是自己破解过程的一个记录，供和我一样的菜鸟参考，也备自己日后遇到类似问题时查找。侠之大者可以直接漂过。如果肯指点一二，感激不尽。

近来坛子里加壳与脱壳版里一个奇怪的带壳 crackme 讨论的人很多，不少大牛都信手给出了 key，成了创意比拼。我等小菜，载入 OD 里还没开始跑就 GameOver 了。实在是心有不甘。

发扬一下 cracker 的风格，自己动手搞定它。感谢 skylly，我一开始茫无头绪时，是你的提示给了我前进的方向。

### （一）Anti （Validate 部分参见 11 楼）

首先用 PDiD 查一下壳，显示 Nothing found * 点击 EP Section 后面的按钮可以看出有 6 个段，段名都是$ 看来好像是加壳了。

OD 载入

[PRE0]

`F8`,来到 `0040E50D`,OD 提示不知如何单步，晕，才第一条指令啊。

1 tlscallback+CC 保护入口

我们再次在 PEiD 中打开，点一下 Subsystem 后面的按钮，看看 PE Details

可以看到

Directory **Information**

RVA SIZE

TLSTable: 00009000 00000024

点一下后面的按钮打开 TLS Viewer 可以看到

CallbackTableVA: 00409060

在数据窗口里可以看到 `00409060` 处的数值是 `00409040`。

具体 TLS 是什么我也不清楚，如果有大侠路过，请指点一下。只是大概了解这里的 Callback Function 会在系统加载程序的时候执行（程序第一条代码执行之前），我们可以看到这里的 Callback Function 地址是 `00409040`，我们在这里下断。然后将 OD 调试选项中事件选项卡设置第一次暂停于系统断点（默认情况下会暂停于 WinMain）。

Ctrl+F2 重新开始。

[PRE1]

`F9`

[PRE2]

原来 OD 会自动在模块入口点下 Int3 断点，如果我们 Alt+B 打开断点窗口就会看到

地址 模块 激活 反汇编

[PRE3]

我们修改 `jnz` 为 `jmp`，始终让它跳就可以避免这里的检测了。

-----------------------------------------------------------------------------------------

CC 反调试

注意后面通过的 `00750000` 段代码也存在 Int3 反调试，有些地址下 F2 断点将会导致程序异常。

[PRE4]

另外 `00401000` 代码段也存在 Int3 反调试。

[PRE5]

如果想在上述地址下断点的话，就要修改一下 `je` 处的代码，直接 `nop` 掉就可以了。

-----------------------------------------------------------------------------------------

由于程序老是出现意外，我们不得不频繁重新开始，这种重复性的工作太没有意思了。怎么办，交给脚本来处理好了，这里我们用 OllyScript 写脚本通过检测。

脚本其实很简单的，脚本命令总共也就那么几条，看一下 OllyScript 附带的说明文档就搞定了，也可以参考 hnhuqiong 大侠的 ODbgScript 入门系列。

Script:
```
asm 00409047,"jmp 00409053"
esto
```

脚本很简单，由于水平不高，地址都是硬编码的，有些地址可能会因为机器的不同而存在差异，请自行修改。

2 进程入口参数校验

来到模块入口点

[PRE6]

[PRE7]

这里我们一路 `F8`，可以看到由于程序没有带命令行参数启动，导致上面的字符串比较没能通过，程序将"`Teeth_In_The_Grass`"作为 CommandLine 的值，重新创建了一个新的进程，然后就退出了。

要通过这里的检测，我们必须将"`Teeth_In_The_Grass`"作为参数创建进程并调试。

Script:
```
asm 0040918D,"mov byte ptr[4094E1],0"
go 40919a
mov eax,00409452
go 4091B0
```

3 `SetUnhandledExceptionFilter`

然后就是一路 `jmp` 来 `jmp` 去，来到

[PRE8]

看看堆栈

[PRE9]

观察堆栈可以知道，如果程序出现了 UnhandledException，正常运行时将会由 `00405325` 处的代码进行处理。由于我们正在调试程序，这时如果 `F9`，OD 会提示程序不知如何继续。

程序从 `004091B0` 处开始就一直跳来跳去，也没干什么正事。要想过这里的检测，我们改一下 `004091B0` 处的代码

Script:
```
asm 004091B0,"jmp 00405325"
```

4 CreateThead 线程保护

........

[PRE10]

........

[PRE11]

........

[PRE12]

........

[PRE13]

........

[PRE14]

........

[PRE15]

堆栈：

[PRE16]

这儿程序复制了一个句柄，源句柄为 `FFFFFFFFFE(-2)`,这里按 `F8` 将会出现引用的内存不能为"written"的错误。

SourceProcess 应该是不存在 `FFFFFFFFFE` 的句柄的，这里不知道有没有什么好的方法处理一下。我们采用最笨的方法，直接跳过它，注意堆栈平衡就可以了。

Script:
```
asm 00405953,"jmp 00405A09"
```

........

[PRE17]

........

[PRE18]

堆栈：

[PRE19]

这里程序创建了一个新线程，线程函数入口地址 `00406727`，这个线程除了 `SuspendThread` `VirtualQuery` `ResumeThread` 外没做什么特别的事情。其实并没有像有些大侠说的那样是什么线程保护。

........

[PRE20]

........

[PRE21]

........

[PRE22]

直接在 `00405BDF` 处下断 `F9` 就可以通过以上代码了。

5 `GetTickCount` 时间校验

........

[PRE23]

........

[PRE24]

........

[PRE25]

........

[PRE26]

........

[PRE27]

........

[PRE28]

6 `RaiseException`

........

[PRE29]

........

[PRE30]

堆栈：

[PRE31]

很明显程序在这里人为地制造了一处错误。如果要想程序正常运行，必须要跳到 SE 处理程序入口处 `004060BC`。

我们在 `004060BC` 处下断，不断地 `Shift+F9`,可最终程序也没有停在我们想断下来的 `004060BC` 处，而是弹出一个窗口告诉我们某某地址内存不能为"read"，然后就 Game Over 了。

这里不知道有没有什么好的办法处理通过，我用的是将 ExceptionCode 改为 `C0123333`，然后通过。

Script:
```
asm 00405E55,"mov dword ptr[4052F9],1"
asm 00405F9B,"push 0C0123333"
```

........

[PRE32]

........

[PRE33]

........

[PRE34]

堆栈：

[PRE35]

按上面相同的方法处理通过

Script:
```
asm 00406286,"mov dword ptr [4052F9],1"
asm 004063E7,"push 0C0123333"
```

[PRE36]

[PRE37]

[PRE38]

7 `GetWindowThreadProcessId` 检测调试器

.......

[PRE39]

........

[PRE40]

........

[PRE41]

堆栈：

[PRE42]

这里取得创建当前窗口的进程 ID，并保存在 `004052ED` 处。由于我们正在用 OD 调试程序，所以这里取得是 OD 的进程 ID。

[PRE43]

[PRE44]

[PRE45]

[PRE46]

[PRE47]

[PRE48]

这里程序设置了一个暗桩，如果检测到被调试，则最终 `004052EC` 值为 1，否则 `004052EC` 值为 2。检测后并不立即异常，而是在后面有对 `004052EC` 寄存的值检测的代码，如果检测到是 1 则会使程序运行不正常而退出。

要跳过这里程序的检测，可以按照通用的方法在调用 `GetWindowThreadProcessID` 后立即将 `004052ED` 处的值改为 Explorer.exe 的进程 ID,

也可以修改上面的 `jnz` 代码为 `jmp`。

Script:
```
asm 00751778,"jmp 007517A9"
```

8 OutputDebugStringA 检测调试器

[PRE49]

这里还有一处检测，由于我用的是 OllyICE，己经针对 OutputDebugStringA 作了修改了，此处顺利通过。

9 Anti_Dump

[PRE50]

堆栈:

[PRE51]

这里将 `00405000` 段代码保护方式设置成了 `PAGE_NOACCESS`，这样在您 DUMP 程序的时候就会出错。如果您想 Dump 程序，就要想办法跳过这段代码或者将 NewProtect 的值设置成 `PAGE_READWRITE` 之类的。

之后程序就没有什么 Anti 了，程序又 `VirtualAlloc` 若干段内存，并对 `00401000` 段进行解码，解码完之后会来到 `00401014`，然后进入 `0076000` 调用 `DialogBoxParamA` 载入程序界面，并设定 DlgProc 地址在 `00401040`。

给个跑到 `00401014` 处的完整 Script：

```
asm 00409047,"jmp 00409053"
esto
asm 0040918D,"mov byte ptr[4094E1],0"
go 40919a
mov eax,00409452
asm 004091B0,"jmp 00405325"
asm 00405953,"jmp 00405A09"
go 00405A09
asm 00405E55,"mov dword ptr[4052F9],1"
asm 00405F9B,"push 0C0123333"
esto
asm 00406286,"mov dword ptr [4052F9],1"
asm 004063E7,"push 0C0123333"
esto
asm 00751778,"jmp 007517A9"
go 00401014
ret
```

注意您有可能需要修改脚本中的相应地址，并且在运行脚本之前请清除所有的断点，调试选项中选择忽略所有异常。

--------------------------------------------------------------------------------

## 【经验总结】

这个壳用

1 tlscallback+CC 保护入口

2 进程入口参数校验

3 `SetUnhandledExceptionFilter`

4 CreateThead 线程保护 （个人感觉好像这里 CreateThread 没什么 Anti 的作用，不知道分析的对不对，请高手指正 ）

5 `GetTickCount` 时间校验

6 `RaiseException` （程序正常运行的时候不知道它是怎么通过这里的，请高手说说）

7 `GetWindowThreadProcessId` 检测调试器

8 OutputDebugStringA 检测调试器

9 Anti_Dump 设置某些代码段为 PAGE_NOACCESS 防止 Dump

--------------------------------------------------------------------------------

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2006 年 12 月 24 日 11:28:51

### （二）Validate

抛了块砖砖，咋就不见大侠们的玉来呢？![](img/45777b5dbd6b901752ab7df42fce6892.jpg) ![](img/45777b5dbd6b901752ab7df42fce6892.jpg) ![](img/45777b5dbd6b901752ab7df42fce6892.jpg) ![](img/45777b5dbd6b901752ab7df42fce6892.jpg) ![](img/45777b5dbd6b901752ab7df42fce6892.jpg) ![](img/45777b5dbd6b901752ab7df42fce6892.jpg)

再接再励，不信大牛们能做到的事我们菜菜努力之后会做不到。再说了有大侠们的讨论在前，多少我们也会得到一些启发吧。

用上面的脚本跑到 `00401014` 处，我们再用一个脚本去掉一些花花，这样看来会舒服一些。

Script:
```
fill 00401027,1,90
fill 0040102F,1,90
fill 004010C7,1,90
fill 004010D1,1,90
fill 00401156,1,90
fill 0040115D,1,90
fill 0040117A,1,90
fill 004011A5,1,90
fill 004011AB,1,90
ret
```

其实这里的代码少得可怜，正好适合我等菜菜分析。![](img/77a5f7cfd3ec840c98be3aecc80df838.jpg) 看见大段大段的代码就有些胆怯。

[PRE52]

........

[PRE53]

[PRE54]

[PRE55]

[PRE56]

[PRE57]

[PRE58]

[PRE59]

[PRE60]

[PRE61]

[PRE62]

我们在 `0040109D` 处下断，然后在输入框内输入试炼码 hawking，点 Validate 按钮被断下来，`F7` 来到

[PRE63]

........

[PRE64]

通过我们的分析，程序会将我们输入的字符串两两一组转换成 1 个字节，但是不管我们输入的字符串长度是多少，都不能同时满足 `len(hkey)>=5` and `len(hkey) <=A` and `len(hkey) = 1C` 这个条件的，看来这个 CrackMe 是不可能 Validate 成功的。除非我们改变程序的执行流程。

而我们执行的情况也确实如此，不管我们输入什么样的字符串，`0040118C` retn 到 `004010A2` 之后 `eax` 值总是-64,没戏。

但是大侠们给出的 key 确实是能弹出成功的对话框框出来的，而且有的弹出的对话框还很彪捍，而我们又没有修改程序的执行代码呀，这是怎么回事呢？

输入 aalloverred 大侠给出的 key

[PRE65]

运行并跟踪看一下。没什么出奇，程序和我们先前看到的流程都是一样的，比较到长度是否大于 A 时就 return -64 了。看来是不会成功的了，但是 `F8` 几下之后竟然弹出了成功的对话框，狂晕！

再仔细跟着看了看，原来这里 retn 并没有 retn 到我们先前的 `004010A2` 处进行 `eax` 值是否大于等于 0 的比较，而是来到了

[PRE66]

然后程序就直接跳过了返回值的检验，成功了。



我们来看看关键代码![](img/f5d9879e4703d3dcc963d0743af268d2.jpg) 秘密就在这里了。

```
[PRE67]
[PRE68]
[PRE69]
```

我们看看 `enter` 到底何方神圣

`ENTER`—Make Stack Frame for Procedure Parameters

Creates a stack frame for a procedure. The first operand (`size` operand) specifies the size of the stack frame (that is, the number of bytes of dynamic storage allocated on the stack for the procedure). The second operand (`nesting level` operand) gives the lexical nesting level (0 to 31) of the procedure.

这里 `enter 100 , 0` 在栈里动态划分一块 100 个字节的空间

之前堆栈：

```
[PRE70]
```

寄存器：

```
ESP 0006FC9C
EBP 0006FCAC
```

之后堆栈：

```
[PRE71]
```

寄存器：

```
ESP 0006FB98
EBP 0006FC98
```

如果我们输入的字符串长度大于 100 的话，`lstrcpy` 操作将会超出 `enter` 分配的 100 大小的空间，copy 到程序正常的堆栈空间里来。这就是所谓的溢出吧。

再看看 `LEAVE`—High Level Procedure Exit

Releases the stack frame set up by an earlier `ENTER` instruction.

```
ESP ← EBP;
EBP ← Pop();
```

之前堆栈：

```
[PRE72]
```

寄存器：

```
ESP 0006FB98
EBP 0006FC98
```

之后堆栈：

```
[PRE73]
```

寄存器：

```
ESP 0006FC9C
EBP 0006FCAC
```

然后的 `retn` 语句就会返回到 `ESP` 所指向的 004010A2。到这里已经很明显了，只要 `lstrcpy` 的时候能够覆盖掉 006FC9C 处的值，那么程序执行到后来的 `retn` 时，就会返回到我们修改后想让它到达的地方。来个实验，随便输入 0x208 位字符，然后再加上 A9104000 应该也能验证成功。

由于程序可以 `retn` 到你想要 `retn` 到的任意地址，因此您完全可以随心所欲地作许多操作，这就要看您的创意和编程功底了。

shoooo 有个 CrackMe 和这个有着异曲同工之妙。有意者可以在 `CrackMe & ReverseMe` 版块搜索下载试玩之。

搞了半天，终于站在各位大侠的肩膀上将这个 CrackMe 分析完了。如果没有大侠们的 key 作参考，我们菜菜还真的难以想到这种通过堆栈溢出动态改变程序执行流程的方法（好像也叫什么 ShellCode 吧），改天好好看看《The Shellcoder's handbook》，学习学习。

看别的大侠写文章往往举重若轻，信手拈来。自己写来却是别有一番滋味在心头。很多东东知其然，不知其所以然，平时不写的时候感觉不到，真正想表达，特别是想清晳地阐述的时候往往失语，可能这就是高手与菜菜的区别吧。![](img/70b1b1b49da6fb968b36803b1ec24864.jpg) ![](img/70b1b1b49da6fb968b36803b1ec24864.jpg) ![](img/70b1b1b49da6fb968b36803b1ec24864.jpg)

贴一组可用的 key

> ```
> [PRE74]
> ```
>

大家都来贴

看谁的有创意

> ```
> [PRE75]
> ```
>

```
[PRE76]
```
