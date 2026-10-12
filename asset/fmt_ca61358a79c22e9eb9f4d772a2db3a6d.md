
# 如何阻击“ESP 定律”？

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-498.htm`](https://www.pediy.com/kssd/pediy07/pediy7-498.htm)

如何阻击“ESP 定律”？

作者：`jney2`

喜欢 Crack，也关注加密手段。

首先，感谢 `kanxue` 终于推出了我苦苦等待的看雪学院论坛精华 6，惊喜中居然还是五周年纪念收藏版。粗略了一下，精华 6 是分量足，精华多呀！

个人感觉 Crack 界变化较大的有三：
1、Cracker 队伍不断壮大，高手不断涌现；
2、脱壳日益成为 Cracker 的必修功课；
3、`OllyDbg` 日益成为 Cracker 的首选调试工具。

`OllyDbg`，我一直用它！`OllyDbg` 我选择，我喜欢！

其次，我来说一下“ESP 定律”。

在五周年纪念收藏版中搜索“ESP 定律”即可找到 25 篇文章，要了解“ESP 定律”有 3 篇文章必看：

1、`fly` 的（论坛里 `fly` 的印象给我最深了）《ESP 定律 + 壹锅端——另类 `ACProtect` `V1.20` 壳的简便脱壳 And 修复方法》（2004 年 2 月 19 日 03:46）。这篇文章首提“ESP 定律”，相信老鸟一看就懂。我看了是有一种豁然开朗的感觉，我怎么没想到在我心爱的 `OllyDbg` 中可以这样下断呢？

2、`Lenus` 的《寻找真正的入口（OEP）--广义 ESP 定律》（不知由哪位仁兄转帖 2004-09-14,15:43）。这篇文章具有释疑、解惑之功效，我等菜鸟是一看就懂，也更加明白什么是“堆栈平衡”原理。

3、`cater` 的《ESP 脱壳 个人小节》（2004-08-31,09:09）。这篇文章简直就是 Cracker 的众多加密软件的“ESP 定律”应用宝典。在这里，无论 Cracker 还是软件加密者都可以看到“ESP 定律”的存在已是众多加密软件的致命伤之一。

第三、如何阻击“ESP 定律”？

“ESP 定律”之所以存在，就是因为“堆栈平衡”原理。要是违反了这一原理，程序就无法正常运行。然尔学过汇编的人都应当知道以下几点的：

1、堆栈是可以改变的；
2、堆栈指针是可以改变的；
3、堆栈内容是可以复制的；
4、堆栈内容也是可以加密的；
5、操作堆栈更是极其危险的。

你也许会说，堆栈内容复制还要访问原始堆栈地址，还是要断下的，但我可在保护现场之后马上进行，然后再解壳，再改变堆栈指针。并且刚开始的堆栈内容不多，可以连续复制若干份，恢复时随机取一份，防止 Cracker 在新的位置下断。对堆栈内容加密也是可取的。这样“ESP 定律”即使应用了也收效不大，有效的防止了快速脱壳。

还有，我做过试验，有些壳不恢复现场也还能正常运行，可见刚开始的堆栈平衡并不十分重要。当然，这样的危险系数还是蛮大的。

第四、我的实例简单实现。

1、 `UPX` 加壳记事本，这是大家最熟悉的壳了。

原始壳：

```
01014FE0 >  60               PUSHAD
01014FE1   BE 00100101      MOV ESI,notepad.01011000
01014FE6   8DBE 0000FFFF    LEA EDI,DWORD PTR DS:[ESI+FFFF0000]
01014FEC   57               PUSH EDI
01014FED   83CD FF          OR EBP,FFFFFFFF
01014FF0   EB 10            JMP SHORT notepad.01015002
01014FF2   90               NOP
01014FF3   90               NOP
01014FF4   90               NOP
01014FF5   90               NOP
01014FF6   90               NOP
01014FF7   90               NOP
```

刚好这里有 6 个 `90` 字节，可加入如下调用指令：

```
01014FF0   E8 75010000      CALL notepad.0101516A
01014FF5   90               NOP
01014FF6   EB 0A            JMP SHORT notepad.01015002
```

`0101516a` 处是连续的 `0` 字节：

```
01015168   0000             ADD BYTE PTR DS:[EAX],AL       //用来保持复制后的指针位置；
```

以下复制堆栈内容到新址；

```
0101516A   60               PUSHAD
0101516B   8BFC             MOV EDI,ESP
0101516D   81EF 00000100    SUB EDI,10000                   ; UNICODE "=::=::\\"
01015173   8BF4             MOV ESI,ESP
01015175   B9 40000000      MOV ECX,40
0101517A   893D 68510101    MOV DWORD PTR DS:[1015168],EDI
01015180   36:8B06          MOV EAX,DWORD PTR SS:[ESI]
01015183   36:8907          MOV DWORD PTR SS:[EDI],EAX
01015186   83C7 04          ADD EDI,4
01015189   83C6 04          ADD ESI,4
0101518C  ^ E2 F2           LOOPD SHORT notepad.01015180
0101518E   61               POPAD
0101518F   C3               RETN
```

原壳转到 OEP 处的代码：

```
0101512E   61               POPAD
0101512F  - E9 AC19FFFF     JMP notepad.01006AE0
01015134   0000             ADD BYTE PTR DS:[EAX],AL
01015136   0000             ADD BYTE PTR DS:[EAX],AL
01015138   0000             ADD BYTE PTR DS:[EAX],AL
```

修改为：

```
0101512E   8BF4             MOV ESI,ESP
01015130   81EE 00010000    SUB ESI,100
01015136   8B3D 68510101    MOV EDI,DWORD PTR DS:[1015168]
0101513C   B9 40000000      MOV ECX,40
01015141   36:8B07          MOV EAX,DWORD PTR SS:[EDI]
01015144   36:8906          MOV DWORD PTR SS:[ESI],EAX
01015147   83C6 04          ADD ESI,4
0101514A   83C7 04          ADD EDI,4
0101514D  ^ E2 F2           LOOPD SHORT notepad.01015141
0101514F   8BD4             MOV EDX,ESP
01015151   81EA 00010000    SUB EDX,100
01015157   83C2 28          ADD EDX,28
0101515A   8BE2             MOV ESP,EDX
0101515C   61               POPAD
0101515D   68 E06A0001      PUSH notepad1.01006AE0
01015162   C3               RETN
```

以上代码完成堆栈内容转移，指针调整，转到 OEP 处的功能。

第五、完成如上修改只是对我的构想最简单的实现，其实还有很多需要改进和注意的地方，在此只不过抛砖引玉，希望高手也来谈谈自己的感想和经验。

`dll` 有三个参数要保护堆栈，`exe` 怎么破坏只要不搞得溢出了就没必要保留堆栈，干掉最后的 `POPAD` 都行。

`EXE` 的加壳必须注意目标执行文件的退出方式。如果是 `ExitProcess` 退出的话，堆栈就有可以破坏的前提；如果是靠执行文件堆栈入口退出的话，必须要将该地址在壳最后进入 OEP 前压栈。

`4dd466d0e1e0cc123afbe504da2462ca`
