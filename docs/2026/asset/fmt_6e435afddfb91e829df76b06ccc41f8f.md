
```
LOAD:000000000002BB3C                 SUB             SP, SP, #0x1B0
LOAD:000000000002BB40                 STP             X28, X27, [SP,#0x1A0+var_50]
LOAD:000000000002BB44                 STP             X26, X25, [SP,#0x1A0+var_40]
LOAD:000000000002BB48                 STP             X24, X23, [SP,#0x1A0+var_30]
LOAD:000000000002BB4C                 STP             X22, X21, [SP,#0x1A0+var_20]
LOAD:000000000002BB50                 STP             X20, X19, [SP,#0x1A0+var_10]
LOAD:000000000002BB54                 STP             X29, X30, [SP,#0x1A0+var_s0]
LOAD:000000000002BB58                 ADD             X29, SP, #0x1A0
LOAD:000000000002BB5C                 MRS             X26, #3, c13, c0, #2
LOAD:000000000002BB60                 MOV             W23, W1
LOAD:000000000002BB64                 MOV             W24, W0
LOAD:000000000002BB68                 MOV             W10, #0x2710
LOAD:000000000002BB6C                 MOV             W12, #0x64 ; 'd'
LOAD:000000000002BB70                 LDR             X14, [X26,#0x28]
LOAD:000000000002BB74                 MUL             W15, W24, W10
LOAD:000000000002BB78                 MUL             W12, W23, W12
LOAD:000000000002BB7C                 MOV             W22, W2
LOAD:000000000002BB80                 MOV             W13, #0xC600
LOAD:000000000002BB84                 ADD             W10, W15, W12
LOAD:000000000002BB88                 MOV             W8, #0x96
LOAD:000000000002BB8C                 SUB             X9, X29, #-var_70
LOAD:000000000002BB90                 MOVK            W13, #0x53E,LSL#16
LOAD:000000000002BB94                 ADD             W10, W10, W22
LOAD:000000000002BB98                 MOV             X21, X4
LOAD:000000000002BB9C                 ADD             W13, W10, W13
LOAD:000000000002BBA0                 STUR            X14, [X29,#var_58]
LOAD:000000000002BBA4                 STUR            W8, [X29,#var_70]
LOAD:000000000002BBA8                 ADR             X27, loc_2BBAC
LOAD:000000000002BBAC
LOAD:000000000002BBAC                 loc_2BBAC                               ; DATA XREF: sub_2BB3C+6C↑o
LOAD:000000000002BBAC                 LDRSW           X4, =0xFFFFFE62
LOAD:000000000002BBB0                 MOV             X20, #0xAB
LOAD:000000000002BBB4                 EOR             X4, X4, X20
LOAD:000000000002BBB8                 ADD             X4, X4, #0x88
LOAD:000000000002BBBC                 MVN             X4, X4
LOAD:000000000002BBC0                 LDRSW           X10, [X9]
LOAD:000000000002BBC4                 EOR             X4, X4, X10
LOAD:000000000002BBC8                 ADD             X27, X27, X4
LOAD:000000000002BBCC                 MOV             X8, #0x62 ; 'b'
LOAD:000000000002BBD0                 BR               X27
```

代码看起来有点复杂，各种运算，还有间接地址访问，而且每个函数头部都有这种间接跳，且貌似每个表达式都不一样，但实际上这种处理起来也相当简单，分静态和动态两种情况。

动态的情况不用说，单步或`trace`来获取对应的跳转地址即可，对于 Ghidra，也不需要特别处理，它会自动帮你把这些常量表达式计算出来，在进行反编译时会自动使用它计算出的结果进行代入。

比如下面的是 Ghidra 的汇编显示：

```
LAB_0013dd1c                                    XREF[1]: 0013dd00(j)
0013dd1c e8 03 00 90     adrp       x8,0x1b9000
0013dd20 08 01 11 91     add        x8,x8,#0x440
0013dd24 08 09 40 f9     ldr        x8,[x8,#0x10]=>DAT_001b9450                     = 000000000013DC94h
0013dd28 08 71 02 91     add        x8,x8,#0x9c
0013dd2c 00 01 1f d6     br         x8=>LAB_0013dd30
```

而如果你要说就用 ida，不开启动态调试的情况下也简单，用 ida 的脚本使用 unicorn 进行模拟执行一下结果就跑出来了：

![](https://bbs.kanxue.com/upload/attach/202210/719948_MG6999XV8SYE27J.png)

对于那种代码之间被各种跳转包围的间接跳，例如：

ida 中那种函数中间的间接跳

```
LOAD:000000000002BD58                 loc_2BD58                               ; DATA XREF: sub_2BB3C+3B0↓o
LOAD:000000000002BD58                                                                       ; LOAD:off_16FC38↓o
LOAD:000000000002BD58 E9 23 0A A9                 STP             X9, X8, [SP,#0x1A0+var_100]
LOAD:000000000002BD5C 88 36 40 F9                 LDR             X8, [X20,#(off_16FC28 - 0x16FBC0)] ; loc_2BD68
LOAD:000000000002BD60 FF 0F 00 F9                 STR             XZR, [SP,#0x1A0+var_188]
LOAD:000000000002BD64 08 2D 00 91                 ADD             X8, X8, #0xB
LOAD:000000000002BD68
LOAD:000000000002BD68                 loc_2BD68                               ; DATA XREF: sub_2BB3C+220↑o
LOAD:000000000002BD68                                                                       ; LOAD:off_16FC28↓o
LOAD:000000000002BD68 03 00 00 14                 B               loc_2BD74 ; loc_2BD74 ; Keypatch modified this from:
LOAD:000000000002BD68                                                                       ;   BR X8
```

ghidra 函数中部的间接跳：

```
LAB_0013dd1c                                    XREF[1]: 0013dd00(j)
0013dd1c e8 03 00 90     adrp       x8,0x1b9000
0013dd20 08 01 11 91     add        x8,x8,#0x440
0013dd24 08 09 40 f9     ldr        x8,[x8,#0x10]=>DAT_001b9450                     = 000000000013DC94h
0013dd28 08 71 02 91     add        x8,x8,#0x9c
0013dd2c 00 01 1f d6     br         x8=>LAB_0013dd30
```

处理方式其实差不多，不管是 ida 还是 Ghidra，它们即便是能算出来跳转地址，也会给你提示，比如上面的 ida 就提示地址是`loc_2BD68`，但是他们为了确保流程尽可能准确，所以一般不会在反汇编窗口中进行常量化，这时候就需要进行手动`patch`。

如果是硬是要 ida 配合 Unicorn，这时候也不是不可以，但是因为部分常量在其它代码块中，再加上里面经常混杂一些地址中常量的访问，在通用性上面可能会不太好，所以这种时候一般建议是用`trace`的数据，或依赖现有工具反汇编引擎的结果来参考进行`Patch`。

比如，在 Ghidra 中与上面的类似的代码块在反汇编引擎中一般会给你提示出来：

![](https://bbs.kanxue.com/upload/attach/202210/719948_24KBRZX3KPUAA8T.png)

在汇编页面就是如下的这种：

```
LAB_00134638                                    XREF[1]: 00134604(j)
00134638 69 22 40 a9     ldp        x9,x8,[x19]
0013463c a8 03 18 f8     stur       x8,[x29,#local_90]
00134640 08 7d 40 92     and        x8,x8,#0xffffffff
00134644 a9 a7 38 a9     stp        x9,x9,[x29,#local_88]
00134648 a8 c3 17 b8     stur       w8,[x29,#local_94]
0013464c e9 07 00 b4     cbz        x9,LAB_00134748
00134650 c8 07 00 34     cbz        w8,LAB_00134748
00134654 f6 09 00 90     adrp       x22,0x270000
00134658 d6 82 01 91     add        x22,x22,#0x60
0013465c c8 4a 40 f9     ldr        x8,[x22,#0x90]=>DAT_002700f0                    = 00000000001345FDh
00134660 08 ad 01 91     add        x8,x8,#0x6b
00134664 00 01 1f d6     br         x8=>LAB_00134668
```

可以看出，Ghidra 已经通过自己的逻辑算出了常量地址，但是它并没有更新到反汇编结果中，`patch`后效果立马就出来了：

![](https://bbs.kanxue.com/upload/attach/202210/719948_FUCZTMPKJX5GGN9.png)

这种是间接跳，使用了全局变量进行跳转（大致含义就是全局变量`0x270140`这个地址中的内容 + `0xda`计算出的结果就是`call`的地址），Ghidra 这里识别为多种结果，所以这种情况必须得上动态才能知道可能的结果了，不同流程可能结果会不一样。

通过上面类似的反混淆操作后，`0x2bb3c`这个函数大致的伪代码也就可以看了：
