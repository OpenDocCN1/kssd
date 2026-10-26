
# [转载]介绍PELOCK1.06的保护

标 题：[转载]介绍PELOCK1.06的保护

发信人：liuyilin

时 间：2004年2月22日 09:30

详细信息：

说明：本文是转载，版权属于原作者

前一篇由于篇幅的关系不完整，作废。

这篇是翻译，欢迎大家提出宝贵意见。

LOCK手册集摘

\\*\*\*PELOCK(LOCK手册集摘)\*\*\*///

\*\*\*\*\* \*\*\*\*\* 作者：crUsAdEr \*\*\*\*\* \*\*\*\*

这篇文章是为了介绍PELOCK1.06的保护。许多软件（作者）用此保护以免遭到破解的厄运，我会介绍此保护的特点并最终脱壳。PELOCK是高度依赖操作系统的，由于这篇文章是基于WIN2K系统介绍其特点，所以会与别的系统有些出入。

所需工具：

- IDA4.15
- SOFT ICE（基于WIN2K SP3操作系统）
- LordPE Deluxe
- Superbpm（基于NT/2k操作系统）
- Imprec 1.4.2

目标：PELock 1.06

包括：`PElock.idc`

序言：

PELOCK是与SVKPROTECTOR具有许多相似之处的一种壳，尤其是在IAT REDIRECTION和OEP脱壳方面。同时PELOCK还可以通过ICEDUMP对`KERNEL32.DLL`的漏洞侦测出WIN98操作系统中的ICEDUMP（引用SPLAJ在FRAVIA公告版上的留言）。

尽管我用比较浅显的语言说明，但我想，大家都是拥有一定基础并能熟练运用工具的高手（应该能听的懂）。PELOCK还具有通过检验OPCODES “CC”来侦测BPX的功能，所以请不要下`BPX 函数名`不然的话PELOCK会飘然离你而去。同时它具有反跟踪能力，因此你就得用上SUPERBPM。

（我建议大家看看我的两篇介绍ASPROTECT1.2和ARMADILLO2．6的文章以获得一些基础知识使介绍更加清楚明白）

## 1．玩转LOCK

OK，首先你保证已经下载了PELOCK，这时你发现SOFTICE不能载入。怎么办！PELOCK的核心，许多的壳都是通过载入调试来脱壳，因此我们需要一个可以载入的工具。这下用到了前面介绍的SUPERBPM工具，用它与SOFTICE合作，下“`BPM VERTUALALLOC X`”载入。运行PELOCK，断下来后，你会停在这：

```text
00431682     push 4
00431684     push 3000h
00431689     push 0F000h; loader 大小
0043168E     push ecx
0043168F     mov edx, [ebp+VirtualAlloc]
00431692     cmp byte ptr [edx], 0CCh   ; 检测 opcodes for bpx
00431695     setz cl; 如果发现bpx 那么
00431698     add edx, ecx               ; 改变edx的值并且发生错误
0043169A     call edx
0043169C     lea edx, [ebp+756h]   <===  你应该停在这
004316A2     push eax; 这里的eax是VirtualAlloc的地址
004316A3     push edx; 这儿main PELock loader将会decrypted to
004316A4     push eax
004316A5     pusha
<-----------省略------------->
<--loader 解密（decryption）程序----------->
00431768     mov [esp+1Ch], edi
0043176C     popa
0043176D     retn; 从这跳向PELock loader code
```

正如上面所看到的，这儿就是PELOCK解密的地方。尽管这段程序省略了，但是你可以很容易的通过SOFTICE滚动条找到`RET`命令。我们要在`43176D`处下断点，并且DUMP这个ELOCK LOADER，你可以在堆栈中找到新解密（DECRYPTED）的LOADER的基址。DUMP后我们将进入IDA去分解学习。我相信，你一定会重建PE文件头，剪切粘贴DUMP下来的到`PELOCK.EXE`中去并再次用IDA重新载入。

如果你照做了，你将会发现：

```text
00401000     call near ptr PE\_Lock\_start+24h
00401005
00401005 GetAPIAddress proc near       ; CODE XREF: ...
00401005
00401005 var\_4= byte ptr -4
00401005 arg\_0= dword ptr  8
00401005 arg\_4= dword ptr  0Ch
00401005 arg\_8= dword ptr  10h
00401005 arg\_C= byte ptr  14h
00401005 arg\_104= dword ptr  10Ch
00401005
00401005     push ebp
00401006     mov ebp, esp
00401008     pusha
<---------继续---------->
```

摘录的部分是一些PELOCK LOADER的命令，开始处是一些`GETPROCADDRESS`程序，用来读取系统DLL，PE文件头和从输出表找到API地址。API名称是杂乱的并与表相比较（我们将讨论更多关于API REDIRECTION SECTION）。如果一个API地址被找到，那么将核对首字节“CC”，假使发现BPX，那么执行FLOW的些微改变将会导致PELOCK的最终结束。由于整个LOADER不完全解密（DECRYPTED），许多部分仍然是个“谜”，只有在PELOCK执行他的主程序时才能完全解密，如果你发现IDA DISASSEMBLY与SOFTICE屏幕有差别，那么你不得不重新DUMP这个LOADER并且把DUMP下来的解密部分覆盖在原来的部分以使IDA分析时更准确，方便。

当你开始追踪PELOCK LOADER时，你会发现有许多无用但会阻碍分析的代码包含在LOADER中，使分析变的困难。下面就是这样的例子：

```text
0040159F C1 F7 00              sal edi, 0
004015A2 C1 F2 00              sal edx, 0; 毫无用处
004015A5 EB 02                 jmp short loc\_4015A9; 跳过
004015A5                   ; -----------------------------------------------------------
004015A7 CD                    db 0CDh ; -
004015A8 20                    db  20h ;
004015A9                   ; -----------------------------------------------------------
004015A9
004015A9                   loc\_4015A9:                   ; CODE XREF: ...
004015A9 C1 F0 00              sal eax, 0
004015AC EB 02                 jmp short loc\_4015B0
004015AC                   ; -----------------------------------------------------------
004015AE C9                    db 0C9h ; +
004015AF 22                    db  22h ; "
004015B0                   ; -----------------------------------------------------------
004015B0
004015B0                   loc\_4015B0:                   ; CODE XREF: ...
004015B0 E8 01 00 00 00        call loc\_4015B6; 顺序读出 head
004015B0                   ; -----------------------------------------------------------
004015B5 42                    db  42h ; B
004015B6                   ; -----------------------------------------------------------
004015B6
004015B6                   loc\_4015B6:                   ; CODE XREF: ...
004015B6 8D 64 24 04           lea esp, [esp+4];纠正堆栈
OR THIS
004015F1 E8 01 00 00 00        call loc\_4015F7
004015F1                   ;
-----------------------------------------------------------
004015F6 3B                    db  3Bh ; ;
004015F7                   ; -----------------------------------------------------------
004015F7
004015F7                   loc\_4015F7:                   ; CODE XREF: ...
004015F7 8F 44 24 FC           pop dword ptr [esp-4];纠正堆栈
OR THIS
004015D6 7A 03                 jp  short loc\_4015DB; 不要在意Eflags
004015D8 7B 01                 jnp short loc\_4015DB; 和将会跳到哪里
004015D8                   ;
-----------------------------------------------------------
004015DA 7E                    db  7Eh ; ~
004015DB                   ; -----------------------------------------------------------
004015DB
004015DB                   loc\_4015DB:                   ; CODE XREF: ...
004015DB EB 02                 jmp short loc\_4015DF; 无用代码
004015DB                   ; -----------------------------------------------------------
004015DD 8B                    db  8Bh ; ?
004015DE 8D                    db  8Dh ; ?
004015DF                   ; -----------------------------------------------------------
004015DF
004015DF                   loc\_4015DF:                   ; CODE XREF: ...
004015DF C1 F6 00              sal esi, 0
```

我们有必要把这些无用的，对于分析没有任何价值的代码清除，以便于我们看清整个代码，同时也方便了IDA的分析。你可以用如MOW，IMHOTEP或者自创的工具来修整这些代码。我自己就做了一个简单的IDC SCRIPT，他可以检查并把他们`NOP`掉并单行排列显示。文章中`FURTHER CODE`的摘录就正象无用代码的清除一样。我已把IDC SCRIPT（作者写的工具）帖了上来作为参考，你可以自己做一个比较顺手的。

## 2．SOFTICE 侦测

当你清除了无用代码后，你就可以放心的研究PELOCK了，但首先我们必须使PELOCK运行在SOFTICE下。在WIN2K系统下，PELOCK对SOFTICE有很强的侦测性，如INT1侦测，`UNHANDLEDEXCEPTIONFILTER` BYTE侦测。

即由于SOFTICE载入时，他会把`KERNEL32！UNHANDLEDEXCEPTIONFILTER` API的第一字节“55”用“CC”来代替。

PELOCK就是这样侦测`UNHANDLEDEXCEPTIONFILTER` BYTE的，如下：

```text
004010D4 80 38 CC              cmp byte ptr [eax], 0CCh ; '?
004010D7 75 0F                 jnz short 跳则成功
004010D9 74 01                 jz  short near ptr 跳则完蛋
```

因此，我们使用READ（`R`命令）修改ZERO FLAG的值来避开对`UNHANDLEDEXCEPTIONFILTER`的校验。

`BPM UNHANDLEDEXCEPTIONFILTER  R FL Z`

当WIN2000系统运行在无SOFTICE状态时，登陆IDT的INT01描述符已经是DPL=0。因此，如果RING-3级试图访问INT01 WITH DPL=0时，GP=INT0D的例外就会出现，同时WINDOWS INT0D HANDLER 将会报告COOOOOO5（例外\_过程\_违反）作为“例外”代码，而EIP就作为“例外”产生的地址。当两者同时运行时，登陆IDT的INT01描述符被设定为DPL=3，于是这时INT01便可顺利的进入RING3级。如果INT01冲突，那么下一个命令地址便代替它保存在堆栈中，同时WINDOWS将检验所有的断点，如果不存在，那么WINDOWS将报告80000004（例外\_独立\_进程）错误。简言之，如果SOFTICE运行，当INT01冲突时，则“例外”代码是80000004，而出现“例外”的地址便是EIP+2；反言之，如果SOFTICE不运行，“例外“代码和地址就变成了C0000005和EIP。（更多细节，请参见INTEL MANUAL VOLUME 3）

在此感谢[YATES]的精彩解说！

当SOFTICE运行时，PELOCK根据校验的结果产生不同的错误地址。一个SEH HANDLER便形成了！

```text
004044A3 CD 01                 int 1    ; 如果SOFTICE不运行的例外地址
004044A5 C3                    retn; 如果SOFTICE运行的例外地址
004044A6                   ; -----------------------------------------------------------
004044A6 EB 36                 jmp short near sice\_不\_存在
004044A6                   ; -----------------------------------------------------------
004044A8 90 90 90 90 90 90+    Softice\_侦测到 db 35h dup(90h)
004044DD                   ; -----------------------------------------------------------
004044DD C3                    retn
004044DD                   ; -----------------------------------------------------------
004044DE 90 90 90 90 90 90+sice\_不\_存在 db 40h dup(90h)
<--- Inside SEH handler --->
004040E4 83 A8 B8 00 00 00+    sub [eax+CONTEXT.Eip], 0FFFFFFFDh ; Eip+3
004040E4 FD                                     ; 如果SICE存在, int1 例外
004040E4                                        ; 将向前移2字节
```

因此，如果侦测到SOFTICE，例外FLOW将会改变，同时PELOCK也会结束。为了避免这种情况的发生，我们需要修改登陆IDT的INT01描述符，使DPL=0。在WIN2K下运行IDT；找到INTERRUPT DESCRIPTORS TABLE的地址

`D <ADDRESS OF IDT>`

用IDT修改从EE到8E的总共0DH字节  感谢SINTAX 和NIKOLATESLA 的帮助

具体细节，请参考ART OF ASSEMBLY ABOUT IDT FORMAT

## 3．输入表重建
