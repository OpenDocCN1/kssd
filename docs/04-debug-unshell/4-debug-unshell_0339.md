
# 如何跟踪 ASProtect

> 原文：[`www.pediy.com/kssd/pediy03/forum819.htm`](https://www.pediy.com/kssd/pediy03/forum819.htm)

## 声明

我写文章以交流为主，希望大家在转载时能保持文章的完整性。

## 前言

由于 `ASProtect` 外壳加密过的程序中运用了很多反跟踪的技术，所以说起来实在是件费事的事情。我实在担心自己水平有限，无法把它们说得明白。不过想想我又不是权威，错误在所难免。既然我办的是"技术交流站"，不一定要保证每句话都是正确的，也不是写出来就要做什么教程之类的。既是"交流"，那么应该大家都在"交流"中提高，而不是我象个传教士一样永远给大家念着一百年不变的经文。所以我希望不光我说，大家也说。怎么说都行，每个人都可以有每个人的风格。:-) 希望我的想法不是梦想。

作者：`ljtt`

写作日期：`2001-04-10`

## 开始

你跟踪过 `ASProtect` 外壳加密过的程序吗？如果你说跟踪过，是否有过这样的经历？

```
015F:012803D9 E3 33 C0 5A 59 59 64 89-10 68 F7 03 28 01 8D 45  .3.ZYYd..h..(..E
015F:012803E9 FC E8 A1 2D FF FF C3 E9-F3 29 FF FF EB F0 5F 5E  ...-.....)....\_^
015F:012803F9 5B 59 5D C3 00 00 00 FF-FF FF FF 02 00 00 00 0D  \[Y\].............
015F:01280409 0A 00 00 55 8B EC 53 56-8B DA 8B F0 8B 45 08 8B  ...U..SV.....E..
```

哪哪腒 `ERNEL32!FindClose+0386` 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪腜 `ROT32`

```
015F:BFF768C6  FF5518                  CALL      [EBP+18]
015F:BFF768C9  83C410                  ADD       ESP,10
015F:BFF768CC  648F0500000000          POP       DWORD PTR FS:[00000000]
015F:BFF768D3  C3                      LEAVE
015F:BFF768D4  C3                      RET
```

你时常会被带到如上图所示的 `FindClose` 的领空，然后在系统内核里转悠了大半个世纪，才好不容易回到程序的领空。

呵呵，我就曾有过这样的经历。正当我暗自窃喜时，却发现跟踪没多长时间又一次来到了老地方，昏倒.....。

看来这其中有蹊跷，我们不妨从当前的堆栈入手。看看有什么发现？

我们下指令：`D ESP`

显示如下结果：

哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪 byte 哪哪哪哪哪哪哪 PROT 哪?0)哪

```
0167:0081FB10 D8 FB 81 00 D0 FC 81 00-F4 FB 81 00 B0 FB 81 00  ................
              ^^^^^^^^^^^ ^^^^^^^^^^^ ^^^^^^^^^^^ ^^^^^^^^^^^
```

我们一一显示 `0081FBD8`、`0081FCD0`、`0081FBF4`、`0081FBB0` 的内容。终于你会发现一点线索。

我们下指令：`D 81FBF4`

显示如下结果：

哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪 byte 哪哪哪哪哪哪哪 PROT 哪?0)哪

```
0167:0081FBF4 1F 00 01 00 00 00 00 00-00 00 00 00 00 00 00 00  ................
0167:0081FC04 00 00 00 00 00 00 00 00-55 01 00 00 7F 02 FF FF  ........U...
0167:0081FC14 00 00 FF FF FF FF FF FF-00 00 00 00 00 00 00 00  ................
0167:0081FC24 00 00 00 00 00 00 FF FF-00 00 00 00 00 00 00 00  ................
```

呵呵，正好我学习过一点 `SEH` 的知识，了解 `CONTEXT` 结构，看到这......不再犹豫..... `81FBF4` 可能是 `_CONTEXT` 结构的指针。

我们先来了解一下 `_CONTEXT` 结构的定义：

```
_CONTEXT               STRUC

cx_ContextFlags        DD ?

;CONTEXT_DEBUG_REGISTERS
cx_Dr0                 DD ?       ;04
cx_Dr1                 DD ?       ;08
cx_Dr2                 DD ?       ;0C
cx_Dr3                 DD ?       ;10
cx_Dr6                 DD ?       ;14
cx_Dr7                 DD ?       ;18

;CONTEXT_FLOATING_POINT
cx_ControlWord          DD ?
cx_StatusWord           DD ?
cx_TagWord              DD ?
cx_ErrorOffset          DD ?
cx_ErrorSelector        DD ?
cx_DataOffset           DD ?
cx_DataSelector         DD ?

SIZE_OF_80387_REGISTERS     EQU 80
cx_RegisterArea         DB SIZE_OF_80387_REGISTERS DUP (?)
cx_Cr0NpxState          DD ?

;CONTEXT_SEGMENTS
cx_SegGs                DD ?       ;8C
cx_SegFs                DD ?       ;90
cx_SegEs                DD ?       ;94
cx_SegDs                DD ?       ;98

;CONTEXT_INTEGER
cx_Edi                  DD ?       ;9C
cx_Esi                  DD ?       ;A0
cx_Ebx                  DD ?       ;A4
cx_Edx                  DD ?       ;A8
cx_Ecx                  DD ?       ;AC
cx_Eax                  DD ?       ;B0

;CONTEXT_CONTROL
cx_Ebp                  DD ?       ;B4
cx_Eip                  DD ?       ;B8
cx_SegCs                DD ?       ;BC
cx_EFlags               DD ?       ;C0
cx_Esp                  DD ?       ;C4
cx_SegSs                DD ?       ;C8

_CONTEXT               ENDS
;size of CONTEXT is 0CCH bytes
```

从上面的定义中我们可以知道 `_CONTEXT` 结构偏移 `B8` 处保存的是 `EIP` 的值。我们来看一下 `81FBF4 + B8` 的值：

我们下指令：`D 81FBF4+B8`

显示如下结果：

哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪 byte 哪哪哪哪哪哪哪 PROT 哪?0)哪

```
0167:0081FCAC 4A FF 27 01 5F 01 00 00-46 02 00 00 D0 FC 81 00  J.'._...F.......
              ^^^^^^^^^^^
```

我们设下断点：`BPX 127FF4A`

然后，按 `F5` 键，果然程序在我们设下的断点处停了下来。:-)

显示如下结果：

```
015F:0127FF4A  648F0500000000         POP       DWORD PTR FS:[00000000]          <---停在此处
015F:0127FF51  58                     POP       EAX
015F:0127FF52  E84127FFFF             CALL      01272698
015F:0127FF57  A154352801             MOV       EAX,[01283554]
015F:0127FF5C  83E80D                 SUB       EAX,0D
015F:0127FF5F  8B1554352801           MOV       EDX,[01283554]
```

看来我们已经掌握了它的一点规律。但我们不能只满足于此，我们来看一下程序是如何做的？

我取了程序即将设下这个小圈套的一段代码。由于其中使用了花指令，所以我稍微做了点改动，使 `SoftICE` 能正确显示汇编代码。

```
015F:0127FEFB  90                     NOP
015F:0127FEFC  64FF30                 PUSH      DWORD PTR FS:[EAX]
015F:0127FEFF  EB01                   JMP       0127FF02
015F:0127FF01  90                     NOP
015F:0127FF02  648920                 MOV       FS:[EAX],ESP                 <---建立 SEH 链
015F:0127FF05  3100                   XOR       [EAX],EAX
015F:0127FF07  EB01                   JMP       0127FF0A
015F:0127FF09  90                     NOP
015F:0127FF0A  648F0500000000         POP       DWORD PTR FS:[00000000]
015F:0127FF11  EB02                   JMP       0127FF15
015F:0127FF13  90                     NOP
015F:0127FF14  90                     NOP
015F:0127FF15  58                     POP       EAX
015F:0127FF16  E825000000             CALL      0127FF40
015F:0127FF1B  8B44240C               MOV       EAX,[ESP+0C]                 <---EAX 此时即为 CONTEXT 的指针
015F:0127FF1F  8380B800000002         ADD       DWORD PTR [EAX+000000B8],02  <---修改 CONTEXT.EIP
015F:0127FF26  51                     PUSH      ECX
015F:0127FF27  31C9                   XOR       ECX,ECX                      <---ECX = 0
015F:0127FF29  894804                 MOV       [EAX+04],ECX                <---修改 CONTEXT.Dr0 = 0
015F:0127FF2C  894808                 MOV       [EAX+08],ECX                <---修改 CONTEXT.Dr1 = 0
015F:0127FF2F  89480C                 MOV       [EAX+0C],ECX                <---修改 CONTEXT.Dr2 = 0
015F:0127FF32  894810                 MOV       [EAX+10],ECX                <---修改 CONTEXT.Dr3 = 0
015F:0127FF35  C7401855010000         MOV       DWORD PTR [EAX+18],00000155  <---修改 CONTEXT.Dr7 = 155
015F:0127FF3C  59                     POP       ECX
015F:0127FF3D  31C0                   XOR       EAX,EAX
015F:0127FF3F  C3                     RET                                    <---在此处就将进入 FindClose 的领空了
```

原来程序在反跟踪时，不但用 `SEH` 的方式使得不知其所以的人跟踪时陷入了设下的圈套；而且，还使得我们以前设下的断点失效。

（`Dr0`、`Dr1`、`Dr2`、`Dr3` 与断点有关，`Dr7` 与断点的控制有关，至于这方面的内容如果你不清楚，那么你可以选择看一些比较专业的书籍来理解，我无法把它说得清楚。）

从这里我们更加可以肯定我们先前的判断了。以后我们再跟踪时，就不会再在这个小圈套里出不来了。如果你再总结一下规律，也许会发现更多，这就留给大家了。:-) 。

比如，我发现其实程序每次使用这种方法时，真正要跳到的地方就在 `RET` 指令后面不远处的指令。所以每当我遇到花指令开始的地方，就会明白程序又要给我开开玩笑了，我会在 `RET` 指令处停下来，然后在后面不远处的指令设下新断点，然后按 `F5` 键，呵呵，平稳落地。

## 后记

由于本站的主题为"技术研习"，所以我想改变以前只写"如何做"的方式，而尽量以我所知加上以"为何如此做"的方式写。这样我将选择性地在某些难点上费些口水，而不再写如何完整的脱某个程序的壳了。当然另外的原因就是现在的手动脱壳越来越繁锁，虽然外壳加密程序本身可能只是添加了一些新的反跟踪/反破解的手法而已，但只写"如何做"的方式是无法使人明白这些变化的。
