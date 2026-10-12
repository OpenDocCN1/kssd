
# UltraProtect 1.x 的脱壳与修复

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-665.htm`](https://www.pediy.com/kssd/pediy07/pediy7-665.htm)

**软件名称**：雪狐提醒簿 V3.0

**实例下载**：`http://www.onlinedown.net/soft/2469.htm`

**软件大小**：4644KB

**软件语言**：简体中文

**软件类别**：国产软件/免费版/闹铃时钟

**运行环境**：`Win9x`/`Me`/`NT`/`2000`/`XP`

**调试环境**：`Win2000`, `Ollydbg` 1.10, `LordPE`, `ImportREC` 1.6.2, `PEid` v0.93

**作者**: `blackeyes`

**作者声明**：初学 Crack，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

## 1. Dump 文件, 寻找 OEP

首先用 `PEiD` v0.93：

```text
UltraProtect 1.x -> RISCO Software Inc.
```

打开 `OllyDbg`，载入 `RemindBook.exe`。

```text
009A6000 >  60                 PUSHAD                       ; 壳入口
009A6001    EB 01              JMP SHORT RemindBo.009A6004
009A6003    7B 66              JPO SHORT RemindBo.009A606B
009A6005    BD A98966C1        MOV EBP,C16689A9
```

按 `F9` 运行，有 `INT3`，停在 `009B799F`：

```text
009B799B    64:8920            MOV DWORD PTR FS:[EAX],ESP
009B799E    CC                 INT3
009B799F    90                 NOP                          ; 停在这
009B79A0    64:67:8F06 0000    POP DWORD PTR FS:[0]
009B79A6    83C4 04            ADD ESP,4
```

按 1 次 `Shift+F9`，程序运行。

`Ctrl+F2` 重来，`CODE` 段下 `f2` 断点，`F9` 运行，`Shift+F9` 通过 `INT3`，停在 `OEP`：

```text
00729280 >  55                 PUSH EBP                     ; 程序 OEP
00729281    8BEC                MOV EBP,ESP
00729283    B9 05000000        MOV ECX,5
00729288    6A 00              PUSH 0
0072928A    6A 00              PUSH 0
```

`F7` 单步跟一下，很快找到 `IAT`：

```text
IAT:  0074F26C-0074FD6B
size: 0B00
```

这时打开 `ImportREC`，选 `Process RemindBook.exe`。

填上下列信息，

```text
    OEP:  329280
    RVA:  0034F26C, Size:0B00
```

按 `Get Imports`，很多无效的，直接用 `ImportREC` 修复不了，

随便选一个，右键==>`Disassemble/HexView`。

```text
009A6010    68 8DF3FB5A        PUSH 5AFBF38D
009A6015    813424 D8A5032D    XOR DWORD PTR SS:[ESP],2D03A5D8
009A601C    C3                 RETN
```

这几行等于 `jmp far 77F85655`， // `5AFBF38D XOR 2D03A5D8 = 77F85655`

变形的跳转.

从 `009A6010` 到 `009A67FB`，全是这样，`ImportREC` 不能处理，

要将它们变成 `jmp xxxx`，

`Ctrl+F2` 重来，到 `OEP` 后，在 `CODE` 段找一段空白地方，写上一段代码修复：

```text
00729F00    60                 PUSHAD
00729F01    BF 10609A00        MOV EDI,RemindBo.009A6010           ; 起始地址
00729F06    B9 EC070000        MOV ECX,7EC                         ; Size
00729F0B    33C0               XOR EAX,EAX
00729F0D    B0 68              MOV AL,68                           ; 先找 68
00729F0F    F2:AE             REPNE SCAS BYTE PTR ES:[EDI]
00729F11    83F9 00            CMP ECX,0
00729F14    74 2C              JE SHORT RemindBo.00729F42
00729F16    8B1F               MOV EBX,DWORD PTR DS:[EDI]          ; push XXXXX
00729F18    8B47 04            MOV EAX,DWORD PTR DS:[EDI+4]
00729F1B    25 FFFFFF00        AND EAX,0FFFFFF
00729F20    3D 81342400        CMP EAX,243481                      ; 再找 813424
00729F25   ^ 75 E4             JNZ SHORT RemindBo.00729F0B
00729F27    8B57 07            MOV EDX,DWORD PTR DS:[EDI+7]        ; xor [esp], YYYY
00729F2A    33DA               XOR EBX,EDX
00729F2C    8A47 0B            MOV AL,BYTE PTR DS:[EDI+B]
00729F2F    3C C3              CMP AL,0C3                          ; 是 C3 吗?, RETN
00729F31   ^ 75 D8             JNZ SHORT RemindBo.00729F0B
00729F33    C647 FF E9         MOV BYTE PTR DS:[EDI-1],0E9         ; 修改成 JMP FAR ZZZZ
00729F37    83EB 04            SUB EBX,4
00729F3A    2BDF               SUB EBX,EDI
00729F3C    891F               MOV DWORD PTR DS:[EDI],EBX
00729F3E   ^ EB CB             JMP SHORT RemindBo.00729F0B
00729F40    90                 NOP
00729F41    90                 NOP
00729F42    61                 POPAD
00729F43    90                 NOP
```

上面代码的 Binary Format，在 `OLLYDBG` 中，可在 CPU 的数据窗直接用 `Binary Copy/Binary Paste`，挺方便的.

```text
60 BF 10 60 9A 00 B9 EC 07 00 00 33 C0 B0 68 F2 AE 83 F9 00 74 2C 8B 1F 8B 47 04 25 FF FF FF 00 

3D 81 34 24 00 75 E4 8B 57 07 33 DA 8A 47 0B 3C C3 75 D8 C6 47 FF E9 83 EB 04 2B DF 89 1F EB CB 

90 90 61 90 00 00 00 00 00 00 00 00 00 00 00 00
```

写完代码后，让这段代码运行一遍，再回到 `OEP`，用 `OllyDump` 插件 `Dump`。

这时打开 `ImportREC`，再进行 `IAT` 修复，用 `Level 1` 后，只有 3 个不能修复，并且指向壳中：

```text
0  0034F330  ?  0000  009A6280
0  0034F844  ?  0000  009B4E31
0  0034F878  ?  0000  009B4E4B
```

## 2. Debug, 修复 IAT

`Ctrl+F2` 重来，到 `OEP` 后，直接转到 `009A6280`/`009B4E31`，然后 `F7` 单步跟踪。

先跟 `009B4E31`：

```text
009B4E31    837C24 04 FF             CMP DWORD PTR SS:[ESP+4],-1
009B4E36    74 13                    JE SHORT RemindBo.009B4E4B
009B4E38    90                       NOP
009B4E39    90                       NOP
009B4E3A    90                       NOP
009B4E3B    90                       NOP
009B4E3C    55                       PUSH EBP
009B4E3D    E8 BE120000              CALL RemindBo.009B6100             ; EBP = 005A5000
009B4E42    8BC5                     MOV EAX,EBP
009B4E44    5D                       POP EBP
009B4E45   - FFA0 C4FD4000           JMP DWORD PTR DS:[EAX+40FDC4]     ; user32.RegisterHotKey
```

所以 `009B4E31` 实际上就是 `user32.RegisterHotKey`，

再跟 `009A6280`，实际上与 `009B4E4B` 一样了：

```text
009A6280   /E9 C6EB0000              JMP RemindBo.009B4E4B
```

从现在起，下面有很多的花指令，以及很多的垃圾指令，来干扰跟踪，而且其中最主要的 `CODE` 还是

加密的，总是先解密，然后执行，接着又加密回去，下面只将有用的指令列出:

```text
009B4E4B    60                       PUSHAD
009B4E4C    FC                       CLD
009B4E4D    78 03                    JS SHORT RemindBo.009B4E52
009B4E4F    79 01                    JNS SHORT RemindBo.009B4E52
...
009B4F13    68 F64F9B00              PUSH RemindBo.009B4FF6             ; 解密的起始地址
009B4F1F    5B                       POP EBX                            ; RemindBo.009B4FF6
009B4F31    BF 535ED47F              MOV EDI,7FD45E53                   ; 解密的 Key
009B4F40    B9 691549E2              MOV ECX,E2491569
009B4F45    03EF                     ADD EBP,EDI
009B4F47    81C1 D6EAB61D            ADD ECX,1DB6EAD6                   ; ECX = 3F, 解密的长度, DWORD
009B4F56    8B03                     MOV EAX,DWORD PTR DS:[EBX]         ; 读出加密的 CODE, 一个 DWORD
009B4F5D    33C7                     XOR EAX,EDI                        ; 变换
009B4F6D    C1C0 11                  ROL EAX,11                         ; 变换
009B4F76    2B43 04                  SUB EAX,DWORD PTR DS:[EBX+4]       ; 变换
009B4F91    8903                     MOV DWORD PTR DS:[EBX],EAX         ; 写回解密后的 CODE, 一个 DWORD
009B4FA5    81C7 DBF7E8FD            ADD EDI,FDE8F7DB                   ; Key 变换
009B4FDA    83E9 01                  SUB ECX,1
009B4FDD   ^ 0F85 73FFFFFF            JNZ RemindBo.009B4F56              ; 循环
...
```

下面是解密后的 `CODE`，没有花指令，没有垃圾指令，看起来舒服多了：

```text
009B4FF6    61                       POPAD
009B4FF7    55                       PUSH EBP
009B4FF8    E8 03110000              CALL RemindBo.009B6100
009B4FFD    8BC5                     MOV EAX,EBP                        ; EAX = RemindBo.005A5000
009B4FFF    5D                       POP EBP
009B5000    837C24 04 FF             CMP DWORD PTR SS:[ESP+4],-1
009B5005    74 25                    JE SHORT RemindBo.009B502C
009B5007   |90                      NOP
009B5008   |90                      NOP
009B5009   |90                      NOP
009B500A   |90                      NOP
009B500B   |8B98 2C854100           MOV EBX,DWORD PTR DS:[EAX+41852C]    ; user32.MessageBoxA
```

到这已经得到了 `API`，可以修复 `IAT` 了。

再跟一跟这段代码：

```text
009B5011    803B CC                  CMP BYTE PTR DS:[EBX],0CC          ; MessageBoxA 上有断点吗?
009B5014    0F84 DE000000            JE RemindBo.009B50F8               ; 有就跳, 最后会到 005A5000
009B501A    807B 01 CC               CMP BYTE PTR DS:[EBX+1],0CC        ; MessageBoxA+1 上有断点吗?
009B501E    0F84 D4000000            JE RemindBo.009B50F8               ; 有就跳, 最后会到 005A5000
009B5024    8BC3                     MOV EAX,EBX                        ; EAX <== API Address
009B5026    60                       PUSHAD
```
