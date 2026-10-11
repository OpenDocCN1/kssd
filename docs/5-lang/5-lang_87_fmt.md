
# NET 程序的破解（二）

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-291.htm`](https://www.pediy.com/kssd/pediy08/pediy8-291.htm)

## NET 程序的破解（二）

---------动态分析之一 `oep` 2006.4.25

**序：**

`Tankaiha` 在他的“**用 `PEBrowse` 对 `.Net` 程序进行动态调试**”中曾经提到过 3 种方法可以破解 .NET 程序，我下面介绍的方法，刚好是他说的第一种，但又不完全是。就是用 .NET SDK 中自带的调试程序 `\Microsoft.NET\SDK\v2.0\GuiDebug` 对 .NET 的可执行文件直接进行调试。

因为 .NET 版本的不同，安装目录有所差别，都可以在 1.1 和 2.0 版本的目录里找到 `GuiDebug` 这个目录，里面的 `DbgCLR.EXE` 就是我今天要说的动态调试器。`DbgCLR.EXE` 程序是我在破解 .NET 程序过程中，被逼无耐而到处狂翻，无意中找到的工具，而且经过自己的使用，知道了一些皮毛的用法，如果您是老手，还不要取消我 J

**另：** 在上篇文章中，忘记告诉大家 `opcode` 类查找的地址，可找到 `opcode` 指令对应的 16 进制字节，这样可以方便的修改程序：[`msdn.microsoft.com/library/chs/default.asp?url=/library/CHS/cpref/html/frlrfSystemReflectionEmitOpCodesClassLdargTopic.asp`](http://msdn.microsoft.com/library/chs/default.asp?url=/library/CHS/cpref/html/frlrfSystemReflectionEmitOpCodesClassLdargTopic.asp)

## 一、 破解目标

`crackeme.exe`（为某些原因隐藏程序的名称），我会把附件上传到网络空间，给出地址。程序有注册号的限制，而且采用了代码混淆和反跟踪的手段。

## 二、 破解工具

`DbgCLR.EXE`, `Reflector` 在 4.2.36.0

## 三、 DbgCLR.EXE 的一些简单设置和使用

启动 `DbgCLR.EXE`，是中文版本的，对我们而言很容易就能看懂，我只把几个关键地方提一下。如下图所示。

1、设置调试配置如下： 点菜单中的“工具”——“选项”进行设置。**如图一**

![](img/440a24e8e003ab813c8d2cab4e5c3678.jpg)

图一

点选项后出现下面的选择，选中下面的“如果源不可用，则显示汇编”。然后“确定”（下面图二是 2.0 版的显示）

![](img/54a470e737300fc79ceb4b2d75c55e76.jpg)

图二

或者在 1.1 版中如下图三所显示

![](img/6d54b06a7b5bc0e50f24e084dac4ebed.jpg)

图三

注：把实时里的项目都选中。

2、加载程序：从菜单中选择“调试”\---“要调试的程序”，从“程序”按钮中选择我们的需要调试的 `crackme.exe`，然后“确定”就可以了。如下图四所示：

![](img/197b74ad6fd33e6e5c0530b8dea7619c.jpg)

![](img/73e23c7f1b4f8a3fcb20c7f008b54df1.jpg)

图四

选择并确定后，程序不会立刻运行，也不会象在 `OD` 中那样立刻分析代码，而是等待我们的下一部命令的输入，而且“调试”菜单中已经多了很多功能。如图五所示

![](img/47331f3f43eca97d5902b5646190462c.jpg)

图五

3、设置“异常”选项：可以根据程序的情况来设置，因为正在讨论的这个 `crackeme.exe` 采用了很多手段反跟踪等，所以我把异常里所有的选项都选择上了，并且激活。点图五菜单中的“异常”，

![](img/31e4b106f8ecef7f8dfb4e6655b4327e.jpg)

![](img/cbf6c3468e1a223e75fde512d5162d3a.jpg)

图六

调试器环境的设置都差不多了，可以进行下一部的跟踪了。

## 四、 调试分析

### 1、分析

用 `PEID` 查看，为 Microsoft Visual C# / Basic .NET，所以用 `Reflector` 打开 `crackme.exe` 进行分析，发现 `Reflector` 不支持，`crackme` 已经对代码进行了混淆（我把此 `crackme.exe` 发送给了 `Reflector` 的作者，所以 `Reflector` 在 4.2.36.0 版本以后开始支持此 `crackme` 混淆代码的处理，这也就是我最初破解采用动态跟踪这个程序的原因）

改用《**NET 程序的破解（一）**》中的 `Xenocode Fox 2006 Evaluation` 打开 `crackme.exe`，发现 `crackme` 中很多代码都显示不出来，或者反编译出来的信息对我们基本没任何作用 L，即使看到了里面很多 `reg` 信息有关的东西，还是没有任何的成型的代码让我们来分析，你的心凉了吗？最初在没发现 `DbgCLR` 这个工具的时候，我彻底陷入了悲哀中，用 `OD` 和 `SOFTICE` 调试 .NET 的程序，无疑就是一种炼狱生活。到现在我也很感叹对这个 `crackme` 进行混淆的人的水平，或者是我刚接触 .NET，对 .NET 了解的太少了，记得 `Reflector` 的作者曾经问过我一句" What tool was this file obfuscated with?"，我告诉他我也很想知道。

### 2、动态调试

在图五中点“启动”或者“逐语句”“逐过程”都可以启动 `crackme`，`crackme` 启动中看来没有对注册信息的验证，点 `crackme` 中的“开始”按钮，才提示让你注册，出现注册框，如下图：

![](img/203f5e522ba90774eb1f14afc34a0171.jpg)

图七

在出现的对话框里，我们输入假的注册号”miaomiao”，点“确定”后，`crackme` 会提示你重新启动验证，然后重新启动。

第 2 次启动 `crackeme`，然后点“开始”后，`DbgCLR` 异常中断，出现提示框，问我们是否中断，选择“中断”后，程序立刻被断下来（多次调试后发现程序已经把我们注册过的信息写入了系统里，点“开始”后就对我们的输入的注册号进行了动态检测比较，并且采用了多处异常的方式来进行）。

此时可以打开几个窗口，从“调试”—“窗口”中选择自己所需要的窗口，我习惯用下面的几个：

![](img/62ae6a8bb22b52615e5b6ea16d489978.jpg)

![](img/afc0bee0758330416a6df7e7296c6aec.jpg)

![](img/7089c668f879be24965af5d3f12897f8.jpg)

图八

`Crackme` 被中断后，看到图八中的“调用堆栈”里的数据，嘿嘿，你看到什么了？秘密就在这里，放大如图九：

![](img/26c417c3a22faae8f5d030b54b1f4f1f.jpg)

图九

在“调用堆栈”中任意双击其中的一个函数，对应的函数代码立刻在“反汇编”窗口里显示出来，显示的效果，跟 `Reflector` 里显示的结构很相似，也让我们很顺利的了解了程序的执行结构。哈，这正是我们所需要的；绿色箭头所指的，正是子程序要返回的位置，在我们需要的代码上点鼠标右键，选择“插入断点”，这样程序执行完现在的子程序返回后，就可以中断在这里了。

![](img/a91fd5d53b77e2ebf43ce421ad01e793.jpg)

图十

经过多次的跟踪分析，发现做如下的修改，就可以让程序正常运行而不再提示注册；如果要研究这个软件的注册算法，你就要自己去跟踪分析了 J：

```
NQQTools.CheckReg.Check

00000093 0F B6 F0         movzx     esi,al          ;修改 mov al,01
00000096 8B C6            mov       eax,esi         ;修改为 mov esi,eax
00000098 25 FF 00 00 00   and       eax,0FFh 
0000009d 89 45 E0         mov       dword ptr [ebp-20h],eax 
000000a0 90               nop                    
000000a1 EB 0E            jmp       000000B1 
000000a3 C7 45 E0 00 00 00 00 mov  dword ptr [ebp-20h],0 ;修改为 mov [ebp-20],1
000000aa E8 4B 6B 20 78   call      78206BFA 
000000af EB 00            jmp       000000B1 

NQQTools.Form1.button2_Click

00000031 0F B6 F8         movzx     edi,al 
00000034 85 FF            test      edi,edi 
00000036 75 1E            jne       00000056 ;改为 jmp 56
000000a0 8B CF            mov       ecx,edi 
000000a2 E8 89 04 A3 78   call      78A30530 
000000a7 0F B6 F8         movzx     edi,al 
000000aa 85 FF            test      edi,edi 
000000ac 74 0A            je        000000B8 ;修改为跳
```

(注：在代码窗口里可以直接选取，然后从菜单“编辑”—“复制”里把代码复制出来)

## 五、 总结

程序的注册过程到现在我也没时间去研究了，对我来说注册不注册都不重要了，我写到这里的目的，也同样是告诉你们如何来动态跟踪 .NET 程序，除了 `Tankaiha` 所说的 `PEBrowseDbg` 动态跟踪方法外，这应该也算是一个不错的方法了，只要让 .NET 程序变成我们所熟悉的 ASM 代码结构，一切都显的那么自然。

`DbgCLR.EXE` 在调试程序的时候，并不一定非要到异常才能断下来的，这是我需要说明的一点，这是我对抗这个 `crackeme` 的策略而已。如果在调试其他程序的时候，断点的方法就要靠你们自己去发掘去了，记得有好的经验就告诉我 J，我也在摸索中前进。

我对这个混淆 .NET 代码的人真的很佩服，我曾经用 `winhex` 对代码的内存进行动态修改，结果发现内存中的代码类似于动态解码状态，就是函数执行完毕后，代码就没有了，需要执行加密函数时，代码又恢复出来了 L,连做内存补丁的机会都不给我。哎，让我想起了论坛里 **B**i11's **S**imple **P**acker for some .net executables 的压缩壳。

先写到这里吧，再写多了反而有灌水的嫌疑 J，下章节………………………………………………………………

**谨以此文献给所有的 cracker**

转载请保持信息的完整性，谢谢！

 `Oep` 2005.4.26 凌晨

 `4dd466d0e1e0cc123afbe504da2462ca`
