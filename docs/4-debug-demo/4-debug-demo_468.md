
# 自由门 6.2

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-661.htm`](https://www.pediy.com/kssd/pediy08/pediy8-661.htm)

【文章标题】: 修改注册表中的设置信息，实现程序的自定义启动

【文章作者】: 范晨鹏

【作者邮箱】: `pengs168@gmail.com`

【作者主页】: `http://www.cnblogs.com/diylab`

【作者 QQ 号】: 109427716

【软件名称】: 自由门 6.2

【软件大小】: 334k

【下载地址】: 自己搜索下载

【保护方式】: 什么是保护方式？

【编写语言】: `vc++`

【使用工具】: `OllyICE`, `eXeScope`

【操作平台】: `winXP`

【软件介绍】: 很好的代理软件，方便访问 Google 网页快照，维基等。

【作者声明】: 第一次发，有不合适的地方请先警告一下。谢谢！

---

## 详细过程

自由门(`free Gate`)是我用过的最方便的代理软件。运行程序，想去哪儿，就去哪儿。缺点是这个软件启动时会自动访问一些不太好的网站。还有些“程序退出时是否关闭 IE”等不友好的界面。

所以做了些修改。

主要内容:

1. 程序启动的时候不要让它自动打开那个`**`的网页。
2. 退出程序的时候不要提示“是否关闭 IE”而改成“退出而不关闭 IE”。
3. 去掉那些乱七八糟的链接。

总之，留其精华，去其糟粕。

我从实践中学到了:

- `od`的功能键`f8`, `f9`的用法
- `bpx`命令给注册表设断
- 想象对于破解非常重要，尤其像我这样不懂汇编的人。

文章中用到的`Regmon`、`OllyICE`, `eXeScope`在看雪的“工具下载”栏目均有下载。

代码中`//`后面的注释为我加的。其它的是`od`自动加的。

上面“主要任务”中提到的 1 和 2 功能，程序都在“设置”中提供了相关选项，并把用户设置保存到注册表中。只是默认的设置不是我们想要的。

功能 3 就简单了。在`exescope`中将相关对话框资源的`visiable`属性设为`"false"`就行了。

这个程序只有一个独立的文件，所以它的设置项应该保存在注册表中。用`RegmonNt`监视注册表，在`filter`中`include`中填上`freeGate_unpacked`。运行程序`freeGate_unpacked`，并在设置中点一下“程序启动时不打开浏览器”中的复选框。查看监视信息。

```
1.74390316  freeGate_unpacked:2500  OpenKey  HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Internet Settings  SUCCESS  Access: 0x20019

1.74395931  freeGate_unpacked:2500  QueryValue  HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Internet Settings\~venoeil  SUCCESS  0xD420

1.74402022  freeGate_unpacked:2500  CloseKey  HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Internet Settings  SUCCESS

1.74454105  freeGate_unpacked:2500  CreateKey  HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Internet Settings  SUCCESS  Access: 0x2000000

1.74515951  freeGate_unpacked:2500  SetValue  HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Internet Settings\~venoeil  SUCCESS  0xD430

1.74523580  freeGate_unpacked:2500  CloseKey  HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Internet Settings  SUCCESS
```

由第四行和第五行记录可以看到，设置信息保存在了`HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Internet Settings`下的`~venoeil`键中。运行`regedit`多次观察发现, `~venoeil`的默认值为`0x0000d400`，勾选“程序启动时不打开浏览器”后，值为`0x0000d410`。不勾选“程序启动时不打开浏览器”而选择“代理退出时自动保留所有浏览器”后，值为`0x0000d440`。现在我们已经知道。勾选“程序启动时不打开浏览器”后和选择“代理退出时自动保留所有浏览器”后，值为`0x0000d440`。程序的“设置”项中的所有选项的值通过累加保存到`~venoeil`键值中。

注意！每次载入程序之前，记得把注册表项`HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Internet Settings`下的`~venoeil`键删掉，要不程序就不会有我们想要的写注册表的动作了。

用`fi`查壳。壳名叫做`PEcompact2.02`。工具下载栏目提供的`PEcompact`版本不够，脱不了（忘了在哪儿下的了）。在其它地方找了一个，脱得很干净。再用`fi`看一下，是用`vc++`写的。

用`od`载入。停在程序入口处。

```
0042755A  >/$  55           push    ebp（这个信息没用，看别人这么写，我便也这么写）
```

因为我们想要在程序读、写注册表的时候断下来，所以应该下断。

在`od`的最底下`command`文本框中填`bpx RegQueryvalueExA`（本来想用`RegCreateKey` `RegSetValueExA`但经验不足，断点没成功。），`od`跳出一个`"Intermodular calls"`窗口。我不知道它是干什么用的，先关掉。

上下拖了一下，果然看到了一个断点（就是最左面一列显示为红的那一行。），好了，`F9`运行（相当于`twr2000`中的`f5`）。右面寄存器窗口哗哗地一陈翻动。程序断在了

```
00417151  |.  FF15 00B04200 call    [<&ADVAPI32.RegOpenKeyExA>]      ; \RegOpenKeyExA
00417157  |.  85C0          test    eax, eax
00417159  |.  75 40         jnz     short 0041719B
0041715B  |.  8D4424 0C     lea     eax, [esp+C]
0041715F  |.  8D4C24 00     lea     ecx, [esp]
00417163  |.  50            push    eax                        ; /pBufSize
00417164  |.  8B4424 18     mov     eax, [esp+18]              ; |
00417168  |.  8D5424 08     lea     edx, [esp+8]               ; |
0041716C  |.  51            push    ecx                        ; |Buffer
0041716D  |.  8B4C24 18     mov     ecx, [esp+18]              ; |
00417171  |.  52            push    edx                        ; |pValueType
00417172  |.  6A 00         push    0                          ; |Reserved = NULL
00417174  |.  50            push    eax                        ; |ValueName
00417175  |.  51            push    ecx                        ; |hKey
00417176  |.  C74424 24 040>mov     dword ptr [esp+24], 4     ; |
0041717E  |.  FF15 1CB04200 call    [<&ADVAPI32.RegQueryValueExA>] ; \RegQueryValueExA   //程序断在了这里。
00417184  |.  85C0          test    eax, eax
00417186  |.  74 08         je      short 00417190
00417188  |.  8B5424 18     mov     edx, [esp+18]
0041718C  |.  895424 00     mov     [esp], edx
00417190  |>  8B4424 10     mov     eax, [esp+10]
00417194  |.  50            push    eax                        ; /hKey
00417195  |.  FF15 20B04200 call    [<&ADVAPI32.RegCloseKey>] ; \RegCloseKey
```

观察这段代码－－后面的注释（呵呵，我不懂汇编的）。看到后面的`\RegOpenKeyExA`, `RegQueryValueExA`, `RegCloseKey`，大致可以想象程序打开某一注册表键，然后读它的值－－正是我们想到的一段部分。因为我们只需要键名为“`~venoeil`”的注册表信息，所以我们在`00417151`（`RegOpenKeyExA`处）设断（就是在选中这一行，然后按一下`f2`，看到最这一行左面那一列变成红的就行了，再按`f2`就是取消断点）。以得到`open`的`key`为“`~venoeil`”的信息。

现在，按`ctrl+F2`重新开始程序（在菜单“调试”-->重新开始）。按`f9`运行，程序断在了`00417151`（`RegOpenKeyExA`）处。

按`f8`单步跟踪（就是一行一行走，碰上`call`不跟进去，相当于“超级玛丽”中不钻地下管道）。

十二次`f8`以后，向下走了十一行

```
00417174  |.  50            push    eax                        ; |ValueName = "~venoeil"
```

`~vemoeil`. 正是想要的。

主观想象程序对注册表的操作应该是这样的:查找是否存在某一键，有则读取，无则创建。创建成功后继续运行程序。我们这里属于“查找是否存在某一键”。

无论如何，这是第一次查询，程序还会来调用这里“无则创建”。所以继续按`"f9"`。同时观察注册表什么时候添加了`~venoeil`（记得每次观察注册表时需要`f5`刷新）。我这里，第二次走过`0041717E`（`RegQueryValueExA`）后，`od`不动了。跳出了`freegate`的“运行模式对话框”（可能没有跳出来，可能是其它窗口挡住了，把其它窗口拖走找找。如果程序直接运行了，请把注册表项`HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Internet Settings`下的`~venoeil`键删掉）。点“确定”后，回到`OD`，程序又被断在了`00417151`（`RegOpenKeyExA`）处。继续`f9`，再下一次`f9`时，刷新注册表，已经有了`~venoeil`键。

至此，我们经过`00417151`（`RegOpenKeyExA`）处共四次。在第三次和第四次之间，程序创建了注册表键值`~venoeil`。

因为我们的目的是找到程序写注册表的地方，并把写的数值改一下（想像中是把`0x0000d400`改成`0x0000d450`）。所以需要在第三次`f9`之后一步步地看程序怎样进行了写注册表的操作。

先按`F2`把`0041717E`（`RegQueryValueExA`）处的断点清除掉。这时程序里只有一处断点`00417151`（`RegOpenKeyExA`）。

把注册表中的`HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Internet Settings`下的`~venoeil`键删除。

`ctrl+F2`，重新运行程序，按三次`F9`，出现“运行模式”对话框，点“确定”，回到`od`。这时候程序就准备写注册表了。我们用`F8`单步跟踪，目的是找到写注册表的地方。

```
00417175  |.  51            push    ecx                        ; |hKey
00417176  |.  C74424 24 040>mov     dword ptr [esp+24], 4     ; |
0041717E  |.  FF15 1CB04200 call    [<&ADVAPI32.RegQueryValueExA>] ; \RegQueryValueExA
00417184  |.  85C0          test    eax, eax
00417186  |.  74 08         je      short 00417190
00417188  |.  8B5424 18     mov     edx, [esp+18]
0041718C  |.  895424 00     mov     [esp], edx
00417190  |>  8B4424 10     mov     eax, [esp+10]
00417194  |.  50            push    eax                        ; /hKey
00417195  |.  FF15 20B04200 call    [<&ADVAPI32.RegCloseKey>] ; \RegCloseKey
0041719B  |>  8B4424 00     mov     eax, [esp]
0041719F  |.  83C4 08       add     esp, 8
004171A2  \\.  C3            retn                              //程序从这里返回到 004172AC
004171A3      90            nop
```

然后到了`004172AC`，再次`"retn"`（也就是`ret`, 返回，相当于玛丽从管道中钻出来）

```
004172AC  |.  83C4 10       add     esp, 10
004172AF  \\.  C3            retn                          //又一个 ret
```

到了`00408DCB`

```
00408DC6   .  E8 C5E40000   call    00417290
00408DCB   .  8BF8          mov     edi, eax      //ret 到这里
00408DCD   .  8B86 20010000 mov     eax, [esi+120]
00408DD3   .  83C4 04       add     esp, 4
00408DD6   .  83F8 02       cmp     eax, 2
00408DD9   .  75 0E         jnz     short 00408DE9
00408DDB   .  81E7 FF7FFFFF and     edi, FFFF7FFF
00408DE1   .  81CF 00000100 or      edi, 10000
00408DE7   .  EB 11         jmp     short 00408DFA
00408DE9   >  83F8 01       cmp     eax, 1
00408DEC   .  75 0C         jnz     short 00408DFA
```



```
  00408DEE   .  81E7 FFFFFEFF and     edi, FFFEFFFF  
  00408DF4   .  81CF 00800000 or      edi, 8000  
  00408DFA   >  8B86 80000000 mov     eax, [esi+80]  
  00408E00   .  6A 00         push    0                              ; /lParam = 0  
  00408E02   .  6A 00         push    0                              ; |wParam = 0  
  00408E04   .  68 F0000000   push    0F0                            ; |Message = BM_GETCHECK  
  00408E09   .  50           push    eax                            ; |hWnd  
  00408E0A   .  FF15 24B74200 call    [<&USER32.SendMessageA>]       ; \SendMessageA  
  00408E10   .  83F8 01       cmp     eax, 1  
  00408E13   .  75 18         jnz     short 00408E2D  
  00408E15   .  81CF 00400000 or      edi, 4000  
  00408E1B   .  57           push    edi  
  00408E1C   .  E8 8FE40000  call    004172B0    //注册表中有~venoeil 键了！  
  00408E21   .  83C4 04       add     esp, 4  
```

上面这些代码我看不懂。但`F8`跟踪到`00408E21`时，注册表中有了键`~venoeil`，键值为`0x0000d400`。所以，可以知道`00408E1C`处的这个`call`是对写注册表子函数的调用，并且完成了记录“启动时打开主页”复选框的值。

Pardon me，我本来想跟踪写注册表的过程的，但汇编方面，就记住几条“九筒就胡”“霸死便霸无”什么的。像`esp`什么的真不知道是干什么用。进去看了看，吓晕了。赶紧退出来在外面想办法。上下瞅瞅，只看到有两个眼熟的。

```
  00408DF4   .  81CF 00800000 or      edi, 8000
  ...
  00408E15   .  81CF 00400000 or      edi, 4000
```

就是认识后面那两个数字:`8000`和`4000`。

没熟人也得办事。想想前面的分析，注册表写入操作应该是这样的:每个选项都要调用一下写注册表的过程，`~venoeil`的值记录了选项控件的不同状态。比如这里，如果“启动时打开主页”复选框为选中状态，则把`~venoeil`键值增加`0X10`。否则，加`0`。

这样想的话，在这里把这个值改掉。不跟进去也是可以的？试试。

在`00408E15`处`F2`下断（因为再运行时要在这儿附近改代码）`ctrl+F2`重新运行，操作照前，直到程序运行到`00408E15`。

在`00408E15`行右键，选择“汇编”，把`4000`改成`4050`（默认值(`0x0000d400`)+`0x10`(启动时打开主页)+`0x40`(代理退出时自动保留所有浏览器))。

```
  00408E13   . /75 18         jnz     short 00408E2D
  00408E15     |81CF 40400000 or      edi, 4050
  00408E1B   . |57           push    edi
  00408E1C   . |E8 8FE40000  call    004172B0          ;  写注册表子过程
  00408E21   . |83C4 04       add     esp, 4
  00408E24   . |8BCE         mov     ecx, esi
```

两次`f8`，到了`00408E21`，这时写注册表的子过程已经执行完毕。

看看注册表，呵呵，真的变了，变成了我们想要的值`0x0000d450`。

就这样算大“工”告成。剩下的“小工”一是把修改好的文件保存下来。二是用`exescope`把不需要的资源隐藏起来。

保存文件:在`od`中右键，选“复制到可执行文件”－－>所有更改。另存成“`freegate_2.exe`”就好了。

编辑资源不在此赘述。

--------------------------------------------------------------------------------

### 【经验总结】

糊里糊涂地完成了。心里清楚这样的`crack`真是狗屁不通。整理了一下思路。解决这个软件的其实只需要两步:

1、对写注册表的函数下断。（`bpx RegSetValueExA`）。

2、修改`RegSetValueExA`的参数。

据此我回头再看，破解的核心应该是下面这段写注册表的函数

```
  004171C8  |.  FF15 14B04200 call    [<&ADVAPI32.RegCreateKeyA>]      ; \RegCreateKeyA
  004171CE  |.  85C0          test    eax, eax
  004171D0  |.  75 25         jnz     short 004171F7
  004171D2  |.  8B5424 10     mov     edx, [esp+10]
  004171D6  |.  8D4C24 00     lea     ecx, [esp]
  004171DA  |.  6A 04         push    4                              ; /BufSize = 4
  004171DC  |.  51           push    ecx                            ; |Buffer
  004171DD  |.  6A 04         push    4                              ; |ValueType = REG_DWORD
  004171DF  |.  50           push    eax                            ; |Reserved
  004171E0      8B4424 24     mov     eax, [esp+24]
  004171E4      52           push    edx
  004171E5  |.  50           push    eax                            ; |hKey
  004171E6  |.  FF15 18B04200 call    [<&ADVAPI32.RegSetValueExA>]     ; \RegSetValueExA
  004171EC  |.  8B4C24 14     mov     ecx, [esp+14]
  004171F0  |.  51           push    ecx                            ; /hKey
  004171F1  |.  FF15 20B04200 call    [<&ADVAPI32.RegCloseKey>]        ; \RegCloseKey
```

到写注册表前，我本来想把`004171E0`处`mov`的数值改一下就行了。如下

```
  004171E0      8B4424 24     mov     eax, 1FC  //[esp+24]的原值为 0ac
```

但`F9`运行后会提示“不知如何继续，因为内存地址`00000000`不可读……”

即使`mov eax, 1AC`也是一样出错。

我是菜鸟。请高人解释一下为什么不能这么改。要想实现修改`eax`的内容（就是把`push`进去的第四个参数数值比原来加`50`）

应该怎么实现。我弄明白的话会把这篇破文修改得更明白些。

--------------------------------------------------------------------------------

### 【版权声明】

本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2006 年 11 月 11 日 18:54:05

`4dd466d0e1e0cc123afbe504da2462ca`
