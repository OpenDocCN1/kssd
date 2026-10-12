
# Acprotect v1.41 奇幻旅程(终结篇)

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-575.htm`](https://www.pediy.com/kssd/pediy07/pediy7-575.htm)

`ACPROTECT` `v1.41` 奇幻旅程--未了的情缘(终结篇)

【工具】: `Olydbg1.1`(diy 版)、`LORDPE`、`ImportREC1.6F`

【任务】:去除 `Acprotect` 用到的 SDK 保护 `Embedd Protect`,去除 `Replace Code`

【操作平台】: `Windows 2003 server`

【作者】:`LOVEBOOM[DFCG][FCG][US]`

【简要说明】:继上篇分析篇，这次是来终结的了:-),这个旅程比上次的分析可算辛苦很多，一不小心就会飞了的。

不管这么多了，累就累下吧，既然进已经开始了这个旅程，不到最后放弃的话，不是太可惜了。

## 详细过程

### 去除 `Acprotect` 用到的 SDK 保护 `Embedd Protect`

按照任务的顺序一个一个来，我们先解决最大的拌脚石，`ACProtect` 的 `Embedd Protect` 应该这个壳里的最大亮点之一。

加上了对程序的保护加强了不少。下面看看我们就去搬走这块石头。

由上篇的文章里的分析，我写个了脚本方便很快就去关键地方，脚本如下：

```
/*
    Set break point at Embedd protect start address
*/
var bpaddr
start:
  bprm 401000,b6000             //在 Code section 处下断点
  esto
  bpmc
  bp 591079                   //断下后，在处理 SDK 的那个关键地方下个断
lbl1:
  eob lbl2
  run
lbl2:
  cmp esi,0
  je lbl3
  cob
  mov bpaddr,esi
  add bpaddr,400000             //取出 sdk 的具体地址，然后在相关地址处下断
  mov addrval,[bpaddr]
  bp bpaddr
  jmp lbl1
lbl3:
  cob
  bc 591079                   //如果处理完了就结束这个脚本,清除上面的关键断点
  ret
```

写好后，设置 `OD` 忽略全部异常。运行脚本后最后结束在这里：

```
00591079    0BF6            OR ESI,ESI                            ; 如果没有用 SDK 或 SDK 处理部分已经操作完成则跳
0059107B    74 31           JE SHORT <finished>
0059107D    90              NOP
0059107E    90              NOP
0059107F    90              NOP
```

断下后，`alt+B` 打开断点窗口:把非 `pushad` 处设置的断点取消：

```
Breakpoints
Address    Module     Active                          Disassembly                                   Comment
0040CC05   MainCon    Always                          PUSHAD
0040FE71   MainCon    Always                          SUB BYTE PTR DS:[ECX],2D                      ;Clear it
004132D7   MainCon    Always                          PUSHAD
004166D1   MainCon    Always                          PUSHAD
00419A9B   MainCon    Always                          PUSHAD
0041CCD6   MainCon    Always                          PUSHAD
0042089A   MainCon    Always                          CMP AL,0A3                                    ;Clear it
00423ADF   MainCon    Always                          CMC                                         ;Clear it
00427422   MainCon    Always                          PUSHAD
0042ABD1   MainCon    Always                          PUSHAD
0042DEA1   MainCon    Always                          PUSHAD
004317A2   MainCon    Always                          PUSHAD
00436067   MainCon    Always                          PUSHAD
0043930F   MainCon    Always                          PUSHAD
0043C77D   MainCon    Always                          PUSHAD
0043FA3D   MainCon    Always                          PUSHAD
00442DBD   MainCon    Always                          PUSHAD
0044607D   MainCon    Always                          PUSHAD
0044933D   MainCon    Always                          PUSHAD
0044C600   MainCon    Always                          PUSHAD
0044F8DD   MainCon    Always                          PUSHAD
00452C0F   MainCon    Always                          PUSHAD
00455EF0   MainCon    Always                          PUSHAD
00459389   MainCon    Always                          PUSHAD
0045C95D   MainCon    Always                          PUSHAD
0045FC38   MainCon    Always                          PUSHAD
00462EC8   MainCon    Always                          PUSHAD
0046618B   MainCon    Always                          PUSHAD
004696FF   MainCon    Always                          PUSHAD
0046C932   MainCon    Always                          INS BYTE PTR ES:[EDI],DX                    ;Clear it
004708D9   MainCon    Always                          PUSHAD
00474389   MainCon    Always                          PUSHAD
004775C3   MainCon    Always                          OR DH,BYTE PTR DS:[EAX+6F]                   ;Clear it
0047A80F   MainCon    Always                          MOV AL,BYTE PTR DS:[C095E1F]                 ;Clear it
0047DE02   MainCon    Always                          PUSHAD
00481126   MainCon    Always                          PUSHAD
00484396   MainCon    Always                          INT1                                        ;Clear it
```

取消相关的断点后，`F9` 运行后，中断下来：

```
0042089A    60              PUSHAD                            ; 运行后中断在这里
0042089B    F8              CLC
0042089C    66:2BCD        SUB CX,BP
0042089F    87C1            XCHG ECX,EAX
    ......
00420A62    0000            ADD BYTE PTR DS:[EAX],AL
00420A64    47              INC EDI
00420A65    4F              DEC EDI
00420A66    83E8 01         SUB EAX,1
00420A69  ^ 0F85 81FFFFFF   JNZ 004209F0                       ; 循环解压代码后面有很多的东西
00420A6F    7A 03           JPE SHORT 00420A74
    ......
00420D64   /E9 08000000    JMP 00420D71
00420D69   |79 03          JNS SHORT 00420D6E
00420D6B   |C1EE 47        SHR ESI,47                           ;解压了几个回合到一个远程 jmp 处，不要以为这里就差不多到头哦,后面还有很长的路，
00420D6E   |66:8BCE        MOV CX,SI
00420D71   \E9 E91D0000    JMP 00422B5F
    ......
00422B5F    E8 8FFDFFFF    CALL <sub_Copy code>                 ; 这里进去复制代码
00422B64    33C2            XOR EAX,EDX
00422B66    EB 01           JMP SHORT 00422B69
00422B68    73 40           JNB SHORT 00422BAA
    ......
00422C1F    03FD            ADD EDI,EBP
00422C21    E8 01000000    CALL 00422C27
00422C26    90              NOP
00422C27    83C4 04         ADD ESP,4
00422C2A    E8 37E5FFFF    CALL <Check Debugger>
00422C2F    66:C1C6 B1      ROL SI,0B1                           ; Shift constant out of range 1..31
00422C33    46              INC ESI
00422C34    C1CF 1E         ROR EDI,1E
00422C37    7C 03           JL SHORT 00422C3C
00422C39    7D 01           JGE SHORT 00422C3C
00422C3B  ^ 73 E9           JNB SHORT 00422C26
    ......
00422C6C    E8 01000000    CALL 00422C72
00422C71    90              NOP
00422C72    83C4 04         ADD ESP,4
00422C75    E8 3AE8FFFF    CALL <Crc File>
00422C7A    66:C1D6 21      RCL SI,21                            ; Shift constant out of range 1..31
00422C7E    81C5 19AA764B   ADD EBP,4B76AA19
    ......
00422E90    830424 06       ADD DWORD PTR SS:[ESP],6
00422E94    C3              RETN
00422E95    E8 8DF1FFFF    CALL <sub_Anti_Fake_Unpack>          ; 检测是否被脱壳了，
00422E9A    8BD5            MOV EDX,EBP
00422E9C    81EF 6D259939   SUB EDI,3999256D
00422EA2    50              PUSH EAX
00422EA3    E8 01000000    CALL 00422EA9
00422EA8    EA 83C40458 66C>JMP FAR C166:5804C483                ; Far jump
00422EAF    FA              CLI
00422EB0    F1              INT1
00422EB1    83C1 04         ADD ECX,4
00422EB4    E8 01000000    CALL 00422EBA
00422EB9  ^ 7D 83           JGE SHORT 00422E3E
00422EBB    C404BA         LES EAX,FWORD PTR DS:[EDX+EDI*4]     ; Modification of segment register
00422EBE    41              INC ECX
00422EBF    0D AE870F87     OR EAX,870F87AE
00422EC4    0100            ADD DWORD PTR DS:[EAX],EAX
00422EC6    0000            ADD BYTE PTR DS:[EAX],AL
00422EC8    43              INC EBX
00422EC9    83C5 FF         ADD EBP,-1
00422ECC  ^ 0F85 6AFFFFFF   JNZ 00422E3C                         ; 循环解压代码
00422ED2    E8 01000000    CALL 00422ED8
00422ED7    9A 83C40487 D86>CALL FAR 66D8:8704C483               ; Far call
00422EDE    81C0 56FFE800   ADD EAX,0E8FF56
    ......
0042327E    83C4 04         ADD ESP,4                            ; 过了一段漫长的路，到这里
00423281    58              POP EAX
```
