

# Iris v2.00 build 14

> 原文：[`www.pediy.com/kssd/pediy03/forum1091.htm`](https://www.pediy.com/kssd/pediy03/forum1091.htm)

| **写在前面**   
这篇文章仅用于**教学目的** !!  
软件开发者花了许多时间来写他们的程序。他们需要卖软件来养活自己!   
请购买优秀的产品!!   
由 +Tsehp 在 2001 年 3 月发表。  |
| **I. 简介**  |
|     欢迎来到脱壳天地。由于对 ASProtect 太过专注，可能有人都忘了还有其它的加壳保护模式。这篇文章里我们的脱壳对象是一个由 PCGuard-wrapped 保护的软件。当然之后要 K 掉它。  
    这是一个不错的数据包分析程序，只是有点贵，ONLY：1745$。我向作者询问过，他说(引用): "*正确的价格是市场对某产品所愿意支付的价格。Iris 的目标市场是公司用户，它们付的起。如果 IRIS 能帮助它们解决值 2000$的问题(想一想那些昂贵的咨询费用)，那么就值这么多。*"   
    既然我们大家都不开公司，那就要想办法让它变得便宜些....... |
| **II. 关于保护** |
|     由 PCGuard 压缩的标志: 观察被保护 exe 文件的段信息就能发现。它对 import table 的处理与 ASProtect 相同，有基本的反 softice 手段。这是从 PC Guard 网站上的引用:"*Security envelope consist of large number of encryption layers and anti-debugging, anti reverse engineering code and contains all protection functions. Protected application becomes encrypted after protection and is distributed in encrypted state. You can put more then one protection layer around your application for better security. This makes protected application highly secure and safe from tampering.*"说的真好听....但'容易' 推翻。 |
| **III. 反向工程** |
|     我采用如下步骤: **I. 脱壳**  

> 1\. 寻找 OEiP  
> 2\. 解压缩  
> 3\. 重建 Import Table  
> 4\. 另一种脱壳方式

**II. 破解**    开始了!**I. 脱壳  
\*\*\*\*\*\*\*\*\*\*\*\*****1.寻找 OEiP**  
    这是脱壳的第一步：找到原始的 Entrypoint。在我以前的脱壳文章中讲过该怎样作，现在我会更详细的再重复一遍，并采用另一种方法。  
    程序运行方式: 加壳程序先还原所有段的代码，完成后跳到已解压后代码的开始处(=OEiP)。一种模式：  

| push ebp ; 程序开始处  
....解压部分代码....  
....   
ret 或 jmp 到 OEiP; 解压完成转到原程序入口 OEiP: 原程序开始处。 |
| --- |

**第一部分: OEiP**    为了找到 Iris 的 OEiP，运行 IceDump (使 SoftICE 不被基本的反 softice 代码搞乱), 在 GetProcAdress 上下断点。运行程序，在 SoftICE 应该会跳出来，按 3 次 F5(当然你可以多按几次)。从返回处开始追踪，设断点以跳出循环，要有耐心。  
    这就我找 OEiP 的方法，可还有更简单的方式。 IceDump 有一个命令是 **跟踪 X**，语法: /TraceX LowEip HighEip。该能在程序执行到'low eip' 与 'high eip' 之间的指令时，让 SoftICE 跳出来。难点只是如何设置 LowEip 与 HighEip。好了，在 SoftICE 中观察一下：我们是在偏移 560000 处的某个位置上，我打赌原始程序一定是运行于偏移 400000 与 500000 之间。打入： "/Tracex 400000 500000"....要有耐心....SoftICE 在 420000 附近中断，你会发现它又跳回到解压代码，所以此处不是 OEiP。用 SoftICE 跟踪直到你返回到解压代码中。  
    再次键入 "/Tracex 400000 500000" ，**要有耐心**. 这确实需要些时间!  
    当 SoftICE 跳出来时，你会在偏移 44717C 处，指令是 'push ebp'.爽，这就是我们要找的 OEiP! 现在要停下来......**2\. 解压缩**    还停在那里吗? 输入: "/dump 400000 130000 c:\\progra~1\\iris\\dumped.exe" 。 400000 是 Image Base, 130000 是 Image Size，两个值在用 PEEditor 载入 Iris.exe 时得到。现在要修复段信息，使得每一段的 RawSize=VirtualSize，RawOffset=VirtualOffset.在 PEEditor 中，你只要在 "Sections" 列表中对每一規段右击鼠标，选择 "Dumpfixer" ，很快就能作完。  
    完成后，Iris 的图标会显示在 dumped.exe 文件上，再把 OEiP 改为 4717C. 试着运行它 ........没有反应，为什么?**3\. 重建 Import Table**当 Iris 要用到一个 API 时，它要这样工作：  
：472000 call \[4BA48C\]  
4BA48C=CDEC08   
：CDEC08 jmp kernel32!compareStringA   
(在此例中，生成的 importtable 能正确的引用 API)或者：：472000 call \[4BA48C\]  
4BA48C=CDEC08   
：CDEC08 push DFEF04  
：CDEC09 call 18FFE4  
(在此例中，生成的 importtable 调用一个解压子程序，返回是正确的 thunk 值)    在此之前你是找不到原始的 IAT ，因为它事实上不存在，所以你不能仅用 procdump 或者一个 IAT 重建 er 就使得程序工作，这次必须重建一切。我们 \*可以\* 用手工方式，那要用很多时间。现在我们有一个新工具用于重建 \*被破坏的\* importtable。这个东东由 +tsehp 编写，名字是 ReVirgin。我们用它来重建 importtable：    运行 ReVirgin 简单看一下，这个工具不是 "你按按纽我就能搞定全部工作" 的。在用它之前你要清楚是怎么回事，有几个 editboxes 中的值你需要自己填：

| **1\. IAT Start RVA**: importtable 开始处的偏移址  
**2\. IAT Length**: importtable 的长度  
**3\. IT RVA**: {自己看吧}where will the importtable be in memory, when we've inserted it in the dumped.exe |
| --- |

**第二部分: ReVirgin 选项的解释**  
怎么找到目标:**1.**在 UpdateWindow(或其它 API) 上设断点，运行程序，当 SoftICE 中断时，按 F12 返回到 Iris 领空，追进你所看到的每一个 call 直到你进入到跳转表中。跳转表看上去是这样子：**开始部分代码**

```


015F:004469D8  JMP       \[004BAF4C\]
```

      **结束**     现在滚动跳转表，找到最小的跳转偏移 (就是找到 JMP xxxxxxxx ，xxxxxxxx 在整个跳转表中是最小值)。要注意跳转表可能被一些其它的指令所分割，所以你要确认已经找到整个表格。  
    另一种找最小偏移的方法是, 从跳转表随机取一个偏移(比如 4BAF80), 输入： d 4BAF80 ，向上滚动数据窗口，直到没有其它数据而只剩下 ‘00’。   
在 Iris 中， IAT 开始于偏移 4BA48C，在 ReVirgin 中填入 BA48C 。**2.** 对于跳转表的长度，我经常随机取一个值 (当然不能太小) ，比如 2000 什么 的，这也许不正确。在此例中运用滚动数据窗口的方式找到 importtable 的结束位置，长度是 1308 。**3\. **大多数时候，我们要在文件末尾追加一个段用于加入 importtable ，只要填文件结束偏移(image size)就可以，在这里是 130000 。    运行原始的(即压缩过的)Iris.exe, 在 ReVirgin 中刷新进程列表，单击 Iris.exe 进程。它应该告诉你 importtable 不正确。在确认所需要的值都已经填入后，单击 "IAT Resolver" 按纽，完成后单击 "Resolve again" 按纽。 Voila, 现在你能看到新创建的 importtable，单击 "IAT Generator" 把 importtable 保存为 IT.BIN，注意 IT 长度为(在此例中) 194 。你可以在这里找到解析后的 importtable，在这里找到 it.bin。现在离开 ReVirgin ，并从 Iris 中退出。    接下来要作的是用新的 importtable 来改变我们的 dumped.exe：

| 

*   用 PE Editor 打开 dumped.exe 
*   单击 "Sections"
*   在某个 section 上右键单击，选择 "Add section"
*   将其命名为 .idata (什么名字无所谓, 比如可以是 .BlackB)
*   在这个段上右键单击，改变 RawSize 和 virtualSize 为 5000。这个值**a 不能比 importtable 长度小!!**，否则无法正常工作
*   在这个段上右键单击，选择 "Dumpfixer" 
*   离开段编辑器
*   单击 "Directory"
*   改变 importtable RVA 为 130000，长度为 194
*   改变 ImageSize 为 135000 (把新段的长度 5000 加到原来的 ImageSize 130000 上得到)
*   如果还没有改过，把 entrypoint 改为 4717C
*   单击 "Apply changes"
*   从 PE Editor 中退出

 |
| --- |

**第三部分: 准备好解压缩后的 EXE 文件**  
    用一个 **hexeditor** 打开 dumped.exe，光标到末尾，插入 IT.BIN，保存。运行修复后的解压缩文件，一切正常!  
    你还可以使用一个由 SV 开发的名为 IT 的工具，只要改变 OEiP 和 Importtable RVA 以及 dumped.exe 的长度，接下来运行 IT Add，它就为你作好了。  
**4\. 另一种脱壳方法**    对于这种保护还有另一种脱壳方法，非常**非常容易**，也更为简洁。那么**为什么**有简单的不用而采用复杂的方法呢？这是因为对于这种破坏 ImportTable 的保护方式，它在大多数情况下都解决不了。这种方法能对付所有的 PCGuard 保护，但对 ASProtect 就没有办法了。    那么，怎样实现呢？首先要找到 OEiP。当你在 OEiP 处，并且 SoftICE 和 IceDump 已经运行，输入：/pedump 400000 44717C c:\\progra~1\\iris\\iris\_dump.exe。  
    之后，从 SoftICE 退出，用 PeEditor **改变** iris\_dump.exe 的 OEiP 为 4717C ，退出 PeEditor，运行脱壳后的文件，正常。简单，不是吗？;-)    Ok，最后要作的是....K 掉它.....**II. 破戒 Iris  
\*\*\*\*\*\*\*\*\*\*\*\*\*\***     15 天使用限制和 NAG。我试着在 UpdateWindow 和 SendMessageA 上下断点来去除 nag ，结果并不理想，实际上这有用，但我发现了另一种简单的方法。你只需要反汇编脱壳后的文件，寻找任何的 "**0Fh**" , {就是这样写的，估计是 0F85 之类的跳转} 你会看到：**开始部分代码**

```
004308AF loc\_4308AF:                             ; code XREF: sub\_430730+16B

j
```

      **结束**     这里就是检验试用期的。可以给它打补丁，但 nag 不会消失，程序还会显示这是演示版。虽然这是演示版还是可以注册，所以一定存在一个'是否注册的旗标'。如果其值为'已注册'，那么上面代码就**不会**运行到。观察程序在上面(@ 0043089B = 430730 + 16B)跳向的'试用版检查'子程序。**开始部分代码**

```
00430890 loc\_430890:                             ; code XREF: sub\_430730+148

j
```

      **结束** 不管在此处改动什么，我们仍是在试用期检查代码中，所以向前找跳向此处的跳转(@ 00430878= 430730+148)**开始部分代码**

```


00430868 loc\_430868:                             ; code XREF: sub\_430730+134jjz      short loc\_430890

            <- 如果注册失败则跳，否则不跳 mov     dword ptr ds:498A88h, 378Dh

 <- 置注册成功旗标
```

      **结束**     很明显：使用你最喜欢的 hexeditor NOP 掉 jz short loc\_430890。完成后运行 dumped/fixed/cracked 的文件，正常工作，没有 NAG，没有试用期!    使它注册到你的大名和组织，改变键值：HKEY\_CURRENT\_USER\\Software\\eEye Digital Security\\Iris\\User\\Nume (注意了，没有打错)  
HKEY\_CURRENT\_USER\\Software\\eEye Digital Security\\Iris\\User\\Company    这(当然了)用 RegMon 很容易发现。好了，任务完成!  |
| **IV. 结尾** |
|     虽然这文章很短，内容倒是蛮充实的。{同意} 我花了 3 个小时来破解，有些可能记不清了。不管怎样感谢 +tsehp 所开发的 ReVirgin 和使用帮助、 +Splaj 和 Risc 在脱壳和 PE 结构上所作帮助。     发现错误请尽快通知我 @ cracking (at) softhome.net，但不要要求注册码和破解!  
  
**祝贺** The Blackbird aka BlackB  |
|  **Endnote: **  
此文由 The Blackbird 写作，© 1999-2001  
This essay can be freely distributed/ published/ printed etc... as long as no modifications are made in any way. {Plz Forgive 6767 \[BCG\] for translation.^\_^}  | 4dd466d0e1e0cc123afbe504da2462ca