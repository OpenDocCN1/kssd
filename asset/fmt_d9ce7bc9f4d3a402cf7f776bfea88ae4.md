
# 网际快车(FlashGet) V1.40 国际版

> 原文：[`www.pediy.com/kssd/pediy05/pediy50422.htm`](https://www.pediy.com/kssd/pediy05/pediy50422.htm)

**目标软件**： 网际快车(FlashGet) V1.40 国际版

**软件大小**： 1729 KB

**软件性质**： 共享软件

**应用平台**： Win9x/NT/2000/XP

**软件介绍**：

下载的最大问题是什么——速度，其次是什么——下载后的管理。网际快车 `FlashGet` (`JetCar`) 就是为解决这两个问题所写的，通过把一个文件分成几个部分同时下载可以成倍的提高速度，下载速度可以提高 `100%` 到 `500%`。网际快车可以创建不限数目的类别，每个类别指定单独的文件目录，不同的类别保存到不同的目录中去，强大的管理功能包括支持拖拽，更名，添加描述，查找，文件名重复时可自动重命名等等。而且下载前后均可轻易管理文件。

**使用工具**： `W32dasm` `v8.93` 黄金修正版本、`TRW2000` `v1.22` 娃娃修正版。

**破解过程**：

软件注册时，先将注册信息保存在注册表中的：`HKEY_CURRENT_USER\Software\JetCar\JetCar\General` 子键中，然后在下一次启动时进行验证。

所以使用 `W32Dasm` 反汇编，串式参考：`"RegPass"`，发现有

`:0041C3E1 6864144F00  push 004F1464`

和

`:0041C5B6 6864144F00  push 004F1464`

两处调用，经过观察：`41C3E1` 处的作用是注册时将注册信息压入注册，所以来到 `41C5B6`：

```asm
0167:0041C5B6  PUSH      DWORD 004F1464
0167:0041C5BB  LEA       ECX,  [ESP+1C]
0167:0041C5BF  PUSH      DWORD 004EF1AC
0167:0041C5C4  PUSH      ECX
0167:0041C5C5  MOV       ECX,EBX
0167:0041C5C7  CALL      004B483A
0167:0041C5CC  ADD       EBX,035C
0167:0041C5D2  MOV       EDI,01
0167:0041C5D7  PUSH      EAX
0167:0041C5D8  MOV       ECX,EBX
0167:0041C5DA  MOV       [ESP+3C],EDI
0167:0041C5DE  CALL      0049E6AF
0167:0041C5E3  LEA       ECX,  [ESP+14]
0167:0041C5E7  MOV       [ESP+38],EBP
0167:0041C5EB  CALL      0049E576        /* 获得你输入的邮件地址   */
0167:0041C5F0  MOV       EDX,[ESI]
0167:0041C5F2  MOV       EAX,[EDX-08]     /* 其位数送入 EAX */
0167:0041C5F5  TEST      EAX,EAX
0167:0041C5F7  JZ        NEAR 0041C739    /* 为空则 Game Over ！ */
0167:0041C5FD  MOV       EAX,[EBX]        /* 序列号送入 EAX */
0167:0041C5FF  MOV       ECX,[EAX-08]     /* 将其位数送入 ECX */
0167:0041C602  TEST      ECX,ECX
0167:0041C604  JZ        NEAR 0041C739    /* 为空则 Game Over ！ */
0167:0041C60A  MOV       ECX,ESI
0167:0041C60C  CALL      0049A227
0167:0041C611  MOV       ECX,ESI
0167:0041C613  CALL      0049A1DB
0167:0041C618  MOV       ECX,[ESI]
0167:0041C61A  CMP       DWORD [ECX-08],BYTE +05
0167:0041C61E  JNG       NEAR 0041C739   /* 邮件地址长度小于 5h 就 Game Over ！*/
0167:0041C624  PUSH      DWORD 004F1488
0167:0041C629  MOV       ECX,ESI
0167:0041C62B  CALL      00499E42        /* 测试你输入的邮件地址是否合法 */
0167:0041C630  TEST      EAX,EAX
0167:0041C632  JL        NEAR 0041C739   /* 不合法则 Game Over ！ */
0167:0041C638  PUSH      DWORD 004F1484
0167:0041C63D  MOV       ECX,ESI
0167:0041C63F  CALL      00499E42
0167:0041C644  TEST      EAX,EAX
0167:0041C646  JL        NEAR 0041C739
0167:0041C64C  MOV       ECX,EBX
0167:0041C64E  CALL      0049A227
0167:0041C653  MOV       ECX,EBX
0167:0041C655  CALL      0049A1DB        /* 获得你输入的序列号 */
0167:0041C65A  MOV       EDX,[EBX]       /* 将其送入 EDX */
0167:0041C65C  MOV       EAX,[EDX-08]    /* 长度送入 EAX */
0167:0041C65F  CMP       EAX,BYTE +2C    /* 比较序列号是否等于 2Ch 位 */
0167:0041C662  JNZ       NEAR 0041C739   /* 不等于就 Game Over ！ */
0167:0041C668  PUSH      DWORD 004F147C   /* “fgc-” 入栈  */
0167:0041C66D  MOV       ECX,EBX
0167:0041C66F  CALL      00499E42        /* 测序列号试前 4 为是否为 fgc-  */
0167:0041C674  TEST      EAX,EAX
0167:0041C676  JNZ       0041C67E        /* 不是跳向下一步比较  */
0167:0041C678  MOV       [ESP+10],EDI
0167:0041C67C  JMP       SHORT 0041C696
0167:0041C67E  PUSH      DWORD 004F1474   /* “fgf-” 入栈  */
0167:0041C683  MOV       ECX,EBX
0167:0041C685  CALL      00499E42        /* 测序列号试前 4 为是否为 fgf-  */
0167:0041C68A  TEST      EAX,EAX
0167:0041C68C  JNZ       NEAR 0041C739   /* 不是则 Game Over ！ */
```

从上面我们知道要成功注册应满足：

1. 邮件地址合法，且长度应大于 `5` 位。
2. 序列号长度为 `44` 位，前四位为`"fgc-"` 或 `"fgf-"`。

所以重新输入序列号继续：

```asm
0167:0041C692  MOV       [ESP+10],EAX
0167:0041C696  PUSH      BYTE +2C
0167:0041C698  MOV       ECX,EBX
0167:0041C69A  CALL      0049E9A6        /* 获得你输入的序列号 */
0167:0041C69F  MOV       EBP,EAX
0167:0041C6A1  XOR       ESI,ESI
0167:0041C6A3  ADD       EBP,BYTE +04
0167:0041C6A6  XOR       EDI,EDI
0167:0041C6A8  MOV       EAX,[EBP+00]
0167:0041C6AB  MOV       ECX,EDI
0167:0041C6AD  ADD       EBP,BYTE +04
0167:0041C6B0  SUB       ECX,BYTE +00
0167:0041C6B3  MOV       [ESP+1C],EAX
0167:0041C6B7  JZ        0041C6D5        /* 跳向 41C6D5 */
...
0167:0041C6D5  MOV       CL,[ESP+1E]      /* 序列号第 7 位送入 CL */
0167:0041C6D9  MOV       DL,AH           /* 序列号第 6 位送入 DL */
0167:0041C6DB  XOR       ECX,EDX         /* EDX 与 ECX 做异或运算 */
0167:0041C6DD  MOVSX     EDX,BYTE [ESP+1F]  /* 序列号第 8 位送入 EDX */
0167:0041C6E2  AND       ECX,BYTE +7F    /* ECX 与 7F 做与运算 */
0167:0041C6E5  IMUL      ECX,EDX         /* ECX 乘以 EDX */
0167:0041C6E8  MOVSX     EAX,AL          /* 序列号第 5 位送入 EAX */
0167:0041C6EB  ADD       ECX,EAX         /* ECX 加 EDX */
0167:0041C6ED  MOV       ESI,ECX         /* ECX 送入 ESI */
0167:0041C6EF  MOV       EAX,[ESP+10]
0167:0041C6F3  TEST      EAX,EAX
0167:0041C6F5  JZ        0041C703
0167:0041C6F7  MOVSX     ECX,BYTE [004EF643]
0167:0041C6FE  CMP       EDI,BYTE +02
0167:0041C701  JZ        0041C70A
0167:0041C703  MOVSX     ECX,BYTE [EDI+004EF640]
0167:0041C70A  MOV       EAX,ESI         /* ESI 送入 EAX */
0167:0041C70C  XOR       EDX,EDX         /* EDX 清 0 */
0167:0041C70E  DIV       ECX             /* 除以 ECX */
0167:0041C710  MOV       EAX,EDI
0167:0041C712  SUB       EAX,BYTE +00
0167:0041C715  JZ        0041C721        /* 跳向 41C721 */
0167:0041C717  DEC       EAX
0167:0041C718  JNZ       0041C725
0167:0041C71A  CMP       EDX,BYTE +08
0167:0041C71D  JNZ       0041C730
0167:0041C71F  JMP       SHORT 0041C725
0167:0041C721  TEST      EDX,EDX
0167:0041C723  JNZ       0041C730      /* EDX 为 0 就 Game Over ！*/
0167:0041C725  INC       EDI
0167:0041C726  CMP       EDI,BYTE +02
0167:0041C729  JNL       0041C74E
0167:0041C72B  JMP       0041C6A8      /* 从这里跳走 */
0167:0041C730  PUSH      BYTE -01
0167:0041C732  MOV       ECX,EBX
0167:0041C734  CALL      0049E9F5
0167:0041C739  POP       EDI
0167:0041C73A  POP       ESI
0167:0041C73B  POP       EBP
0167:0041C73C  XOR       EAX,EAX
0167:0041C73E  POP       EBX
0167:0041C73F  MOV       ECX,[ESP+20]
0167:0041C743  MOV       [FS:00],ECX
0167:0041C74A  ADD       ESP,BYTE +2C
0167:0041C74D  RET
```

从上面我们知道：序列号第 `7` 位和 `7C2E00` 加第 `6` 位的和做异或运算的值与 `7F` 做与运算，之后再乘以第 `8` 位，最后再加上序列号第 `5` 位的 `HEX` 值的和除以 `6B`， 余数不为 `0` 就会 `Game Over` ！

即：`((SN[7] XOR (7C2E00+SN[8]) AND 7F) * SN[8] + SN[5]) MOD 6B = 0`

然后重新填入序列号，会来到：

```asm
0167:0041C6BC  MOVSX     ESI,BYTE [ESP+1E]     /* 序列号第 11 位送入 ESI */
0167:0041C6C1  MOVSX     EDX,BYTE [ESP+1F]     /* 序列号第 12 位送入 EDX */
0167:0041C6C6  MOVSX     ECX,AH                /* 序列号第 10 位送入 ECX */
0167:0041C6C9  AND       ESI,ECX               /* ESI 与 ECX 做与运算 */
0167:0041C6CB  IMUL      ESI,EDX               /* 然后乘以 EDX */
0167:0041C6CE  MOVSX     EAX,AL                /* 序列号第 9 位 送入 EAX */
0167:0041C6D1  ADD       ESI,EAX               /* ESI 加上 EAX */
0167:0041C6D3  JMP       SHORT 0041C6EF        /* 跳向 41C6EF */
...
0167:0041C6EF  MOV       EAX,[ESP+10]          /* 将 [ESP+10] 的值送入 EAX */
0167:0041C6F3  TEST      EAX,EAX               /* 测试 EAX */
0167:0041C6F5  JZ        0041C703              /* EAX 为 0 就 Game Over ！*/
0167:0041C6F7  MOVSX     ECX,BYTE [004EF643]
0167:0041C6FE  CMP       EDI,BYTE +02
0167:0041C701  JZ        0041C70A
0167:0041C703  MOVSX     ECX,BYTE [EDI+004EF640]
                            /* 将 [EDI+004EF640] 的值送入 ECX */
0167:0041C70A  MOV       EAX,ESI               /* ESI 送入 EAX */
0167:0041C70C  XOR       EDX,EDX               /* EDX 清 0 */
0167:0041C70E  DIV       ECX                   /* 除以 ECX */
```
