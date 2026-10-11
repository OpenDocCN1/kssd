
# SoftWrap 6.x 之三：IAT 修复之 VB

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-504.htm`](https://www.pediy.com/kssd/pediy09/pediy09-504.htm)

> 【文章标题】: SoftWrap 6.x 之三：IAT 修复之 VB  
> 【文章作者】: wynney  
> 【软件名称】: XP Smoker 4.4  
> 【使用工具】: `OD`、`CodeCaver`  
> \--------------------------------------------------------------------------------  
> 【详细过程】

## 一、前言

`SoftWrap` 的输入表加密很有特色，倒是跟 `ASProtect` 的 `Stolen Code` 有点类似了。我们知道，一般程序有以下几种 IAT 格式

1. `Jmp Dword ptr ds:[API]`      '6 字节
2. `Call Dword ptr ds:[API]`      '6 字节
3. `mov eax, Dword ptr ds:[API]` ->5 字节
4. `mov REG,Dword ptr ds:[API]`  ->6 字节[eax 外的寄存器,且 esp、edx 基本不被使用]

而 `SoftWrap` 加密之后分别被改成这种形式

1. `Call DWORD PTR DS:[addr1]` '6 字节
2. `Call DWORD PTR DS:[addr2]` '6 字节
3. `Call addr3`                '5 字节
4. `Call addr4`                '5 字节+1 个字节的寄存器类型识别指令

上面最难恢复的是第 4 种，因为我们需要分析出他所对应的一个表，例如，在某某情况是要恢复成 `ebx`，另外某个情况需要恢复成 `ecx`。。。。。同时需要注意的是 3 和 4 是分开的情况

然而，`VB` 的程序只有 1 种情况，就是 `Jmp Dword ptr ds:[API]`被改成了 `Call DWORD PTR DS:[addr]`，这样就大大的减小了我们的难度。本文介绍的是如何去恢复 `Jmp Dword ptr ds:[API]`类型，为后面讲解如何恢复这 4 种类型铺个路。

## 二、去 OEP 溜达

1. 设置异常，忽略特权和指定异常外的所有异常

`![名称:  1.PNG> 查看次数: 174> 文件大小:  31.1 KB](img/7cef7468b37448ea084ada1f6b413f50.jpg)`   

4 次 `Shift+F9`，出现试用框，点 `Try Now`，中断后继续 `Shift+F9` 两次，在 `Code` 段下内存访问断点[不可以 `F2`]，`Shift+F9`，两次，即可到达 OEP 了。

………………………………

```
004081F4    FF15 80224E00   call    dword ptr [4E2280]           ; xpsmoker.004E4E39
004081FA    FF15 80224E00   call    dword ptr [4E2280]           ; xpsmoker.004E4E39
00408200    FF15 80224E00   call    dword ptr [4E2280]           ; xpsmoker.004E4E39
00408206    FF15 80224E00   call    dword ptr [4E2280]           ; xpsmoker.004E4E39
0040820C    FF15 80224E00   call    dword ptr [4E2280]           ; xpsmoker.004E4E39
00408212    FF15 80224E00   call    dword ptr [4E2280]           ; xpsmoker.004E4E39
00408218    68 78844000     push    00408478                     ; OEP
0040821D    E8 F0FFFFFF     call    00408212
```

可以看到所有的指针全部改成了 `call dword ptr [4E2280]`

随便找一个指针，Enter 进去吧

```
004E4E39    9C              pushfd
004E4E3A    50              push    eax
004E4E3B    53              push    ebx
004E4E3C    8B5C24 0C       mov     ebx, dword ptr [esp+C]
004E4E40    53              push    ebx
004E4E41    83EB 06         sub     ebx, 6
004E4E44    68 11270000     push    2711
004E4E49    68 00000801     push    1080000                      ;虚拟内存地址
004E4E4E    C3              retn
```

```
01080000    55              push    ebp                          ; API 加密段首，记住①
01080001    56              push    esi
01080002    57              push    edi
01080003    51              push    ecx
01080004    52              push    edx
01080005    50              push    eax
01080006    BD 66274E00     mov     ebp, 4E2766
0108000B    016C24 18       add     dword ptr [esp+18], ebp
0108000F    2B9D 962A0000   sub     ebx, dword ptr [ebp+2A96]
```

## 三、获取输入表

`Ctrl+G`：`VirtualAlloc`，在段尾 `F2`[直接 `bp VirtualAlloc` 会被检测]，`shift+F9`，中断后返回

`Ctrl+B`：`85 C0 0F 84 ?? ?? ?? ?? F6 C3 02 74 21 FF B5 ?? ?? ?? ?? 6A 10 50 FF B5 ?? ?? ?? ?? FF B5 ?? ?? ?? ??`

```
004E463A    85C0            test    eax, eax   ; 找到这里
004E463C    0F84 89000000   je      004E46CB
004E4642    F6C3 02         test    bl, 2      ; F2[he 有时断不下来]，Shft+F9,删除断点，看寄存器
004E4645    74 21           je      short 004E4668
004E4647    FFB5 DF1C0000   push    dword ptr [ebp+1CDF]
004E464D    6A 10           push    10
004E464F    50              push    eax
004E4650    FFB5 DF1C0000   push    dword ptr [ebp+1CDF]
004E4656    FFB5 E31C0000   push    dword ptr [ebp+1CE3]
004E465C    E8 FAF7FFFF      call    004E3E5B
```

```
EAX 6600A019 MSVBVM60.DllFunctionCall
ECX 00000000
EDX CBF65AD9
EBX FC50432D
ESP 0012FF8C ASCII "賈鏊"
EBP 004E2766 xpsmoker.004E2766
ESI 004E5E50 xpsmoker.004E5E50
EDI 004E67EC xpsmoker.004E67EC
```

Eax 里面出现的是 IAT 了，你可以在 `004E4642` 多中断几次，可以看到 eax 里面的 IAT，呵呵

好了，我们现在来写 Patch 代码，找一片空白[不会被使用的]，大小要足够。

我用的是申请的空间[非程序所有的空间、这就意味着后面用 ImportREC 修复的时候需要用高级指令查找]，我申请的是 `3F0000`，`10000` 大小

设置 `3F0000` 为存放基址的位置，`3F0004` 为 IAT 地址指针，`3F0008` 为存放 IAT 的起始地址

`[003F0000]=ECX`       -->第一中断后 ecx 的值，一般是 0  
`[003F0004]=003F0008`  -->IAT 地址指针  

```
003F0000  00 00 00 00 08 00 3F 00  .... .?.
003F0008  00 00 00 00 00 00 00 00  ........
003F0010  00 00 00 00 00 00 00 00  ........
```

```
 004E4642    3A0D 00003F00   cmp     cl, byte ptr [3F0000]
 004E4648    74 0E           je      short 004E4658
 004E464A    FE05 00003F00   inc     byte ptr [3F0000]
 004E4650    66:8305 04003F0>add     word ptr [3F0004], 4
 004E4658    8B1D 04003F00   mov     ebx, dword ptr [3F0004]
 004E465E    8903            mov     dword ptr [ebx], eax
 004E4660    66:8305 04003F0>add     word ptr [3F0004], 4

 3A 0D 00 00 3F 00 74 0E FE 05 00 00 3F 00 66 83 05 04 00 3F 00 04 8B 1D 04 00 3F 00 89 03 66 83 05 04 00 3F 00 04
```

写好代码后，`Shift+F9`，中断在第一次特权异常

`![名称:  2.PNG> 查看次数: 178> 文件大小:  109.6 KB](img/539ba096d9395b6d448f0862d3109508.jpg)`   

所有的 IAT 都存放起来了，呵呵，获取输入表成功。

## 四、修复输入表

手动去 OEP 吧，`Ctrl+G`：`01080000`(API 加密段首)，`F2`，`F9` 中断下来，删除断点

`Ctrl+B`：`8B 04 96 35 ?? ?? ?? ?? 50 8A 00 04 0F`

```
0108007F    8B0496          mov     eax, dword ptr [esi+edx*4] ; 找到这里
01080082    35 72057040     xor     eax, 40700572
01080087    50              push    eax                        ; 从这里写 Patch 代码，还原 IAT
01080088    8A00            mov     al, byte ptr [eax]
0108008A    04 0F           add     al, 0F
```

事先更改设置

`[003F0004]=00401000`     '作为搜索 `Call Dword PTR:[Addr]`的起始地址



```
01080087    B9 08003F00     mov     ecx, 3F0008                   ; 上一步得到的 IAT 表的起始地址
 0108008C    3901            cmp     dword ptr [ecx], eax
 0108008E    74 05           je      short 01080095
 01080090    83C1 04        add     ecx, 4
 01080093  ^ EB F7          jmp     short 0108008C
 01080095    8BD1            mov     edx, ecx
 01080097    81C2 00100000   add     edx, 1000
 0108009D    8902            mov     dword ptr [edx], eax
 0108009F    83C4 1C         add     esp, 1C
 010800A2    3E:8B0424       mov     eax, dword ptr ds:[esp]
 010800A6    83E8 06         sub     eax, 6
 010800A9    66:C700 FF25    mov     word ptr [eax], 25FF          ; 更改成 Jmp 类型
 010800AE    83C0 02         add     eax, 2
 010800B1    8910            mov     dword ptr [eax], edx
 010800B3    90              nop
 010800B4    90              nop
 010800B5    813D 04003F00 1cmp     dword ptr [3F0004], 408218      ; OEP，作为搜索的结束地址
 010800BF    74 2C           je      short 010800ED
 010800C1    8B0D 04003F00   mov     ecx, dword ptr [3F0004]        ; xpsmoker.00401000
 010800C7    66:8139 FF15    cmp     word ptr [ecx], 15FF           ; 对比是否是要修改的 Call
 010800CC    75 17           jnz     short 010800E5
 010800CE    8BC1            mov     eax, ecx                       ; 是则
 010800D0    83C0 02         add     eax, 2
 010800D3    8138 80224E00   cmp     dword ptr [eax], 4E2280        ; 比较 Call 的 Addr 是不是 004E2280
 010800D9    75 0A           jnz     short 010800E5
 010800DB    83C0 04         add     eax, 4                          ; 是则
 010800DE    A3 04003F00     mov     dword ptr [3F0004], eax        ; 赋予真实的 IAT 地址
 010800E3    FFE1            jmp     ecx                             ; 跳回下一个地址继续
 010800E5    FF05 04003F00   inc     dword ptr [3F0004]             ; xpsmoker. 00401000
 010800EB  ^ EB C8          jmp     short 010800B5
 010800ED  - EB FE          jmp     short 010800ED                  ; F2，Shift+F9 后,全部还原
 010800EF    90              nop
 010800F0    90              nop
 010800F1    90              nop
 010800F2    90              nop
```

????????????????????????????

取消断点，在 `OEP` 处新建 `EIP`，看看现在的 `IAT` 表

```
00407D50   $- FF25 58103F00 jmp     dword ptr [3F1058]            ;  MSVBVM60.__vbaChkstk
00407D56   $- FF25 74103F00 jmp     dword ptr [3F1074]            ;  MSVBVM60.__vbaExceptHandler; 异常处理程序
00407D5C   >- FF25 7C103F00 jmp     dword ptr [3F107C]            ;  MSVBVM60.__vbaFPException
00407D62   .- FF25 20123F00 jmp     dword ptr [3F1220]            ;  MSVBVM60._adj_fdiv_m16i
00407D68   $- FF25 24123F00 jmp     dword ptr [3F1224]            ;  MSVBVM60._adj_fdiv_m32
00407D6E   .- FF25 28123F00 jmp     dword ptr [3F1228]            ;  MSVBVM60._adj_fdiv_m32i
00407D74   $- FF25 2C123F00 jmp     dword ptr [3F122C]            ;  MSVBVM60._adj_fdiv_m64
00407D7A   .- FF25 30123F00 jmp     dword ptr [3F1230]            ;  MSVBVM60._adj_fdiv_r
00407D80   .- FF25 34123F00 jmp     dword ptr [3F1234]            ;  MSVBVM60._adj_fdivr_m16i
00407D86   .- FF25 38123F00 jmp     dword ptr [3F1238]            ;  MSVBVM60._adj_fdivr_m32
00407D8C   .- FF25 3C123F00 jmp     dword ptr [3F123C]            ;  MSVBVM60._adj_fdivr_m32i
00407D92   .- FF25 40123F00 jmp     dword ptr [3F1240]            ;  MSVBVM60._adj_fdivr_m64
00407D98   .- FF25 44123F00 jmp     dword ptr [3F1244]            ;  MSVBVM60._adj_fpatan
00407D9E   .- FF25 48123F00 jmp     dword ptr [3F1248]            ;  MSVBVM60._adj_fprem
00407DA4   .- FF25 4C123F00 jmp     dword ptr [3F124C]            ;  MSVBVM60._adj_fprem1
00407DAA   .- FF25 50123F00 jmp     dword ptr [3F1250]            ;  MSVBVM60._adj_fptan
00407DB0   .- FF25 18103F00 jmp     dword ptr [3F1018]            ;  MSVBVM60._CIatan
00407DB6   .- FF25 1C103F00 jmp     dword ptr [3F101C]            ;  MSVBVM60._CIcos
00407DBC   .- FF25 20103F00 jmp     dword ptr [3F1020]            ;  MSVBVM60._CIexp
00407DC2   .- FF25 24103F00 jmp     dword ptr [3F1024]            ;  MSVBVM60._CIlog
00407DC8   .- FF25 28103F00 jmp     dword ptr [3F1028]            ;  MSVBVM60._CIsin
00407DCE   .- FF25 2C103F00 jmp     dword ptr [3F102C]            ;  MSVBVM60._CIsqrt
00407DD4   .- FF25 30103F00 jmp     dword ptr [3F1030]            ;  MSVBVM60._CItan
00407DDA   .- FF25 54123F00 jmp     dword ptr [3F1254]            ;  MSVBVM60._allmul
00407DE0   .- FF25 08103F00 jmp     dword ptr [3F1008]            ;  MSVBVM60.DllFunctionCall
00407DE6   $- FF25 DC113F00 jmp     dword ptr [3F11DC]            ;  MSVBVM60.__vbaVarLateMemSt
00407DEC   $- FF25 3C113F00 jmp     dword ptr [3F113C]            ;  MSVBVM60.__vbaR8Str
00407DF2   $- FF25 88113F00 jmp     dword ptr [3F1188]            ;  MSVBVM60.__vbaStrR8
00407DF8   $- FF25 B4113F00 jmp     dword ptr [3F11B4]            ;  MSVBVM60.__vbaVarCmpGt
00407DFE   $- FF25 30133F00 jmp     dword ptr [3F1330]            ;  MSVBVM60.rtcRound
00407E04   $- FF25 98113F00 jmp     dword ptr [3F1198]            ;  MSVBVM60.__vbaStrVarCopy
00407E0A   $- FF25 04113F00 jmp     dword ptr [3F1104]            ;  MSVBVM60.__vbaLenVar
00407E10   $- FF25 6C103F00 jmp     dword ptr [3F106C]            ;  MSVBVM60.__vbaEnd
00407E16   $- FF25 A4103F00 jmp     dword ptr [3F10A4]            ;  MSVBVM60.__vbaFpI4
00407E1C   $- FF25 38113F00 jmp     dword ptr [3F1138]            ;  MSVBVM60.__vbaR4Var
00407E22   $- FF25 F0103F00 jmp     dword ptr [3F10F0]            ;  MSVBVM60.__vbaLateIdCallSt
00407E28   $- FF25 F8123F00 jmp     dword ptr [3F12F8]            ;  MSVBVM60.rtcInputCharCountVar
00407E2E   $- FF25 B4123F00 jmp     dword ptr [3F12B4]            ;  MSVBVM60.rtcFileLength
00407E34   $- FF25 74113F00 jmp     dword ptr [3F1174]            ;  MSVBVM60.__vbaStrErrVarCopy
00407E3A   $- FF25 7C113F00 jmp     dword ptr [3F117C]            ;  MSVBVM60.__vbaStrI2
00407E40   $- FF25 9C123F00 jmp     dword ptr [3F129C]            ;  MSVBVM60.rtcKillFiles
00407E46   $- FF25 E4103F00 jmp     dword ptr [3F10E4]            ;  MSVBVM60.__vbaInputFile
00407E4C   $- FF25 B8123F00 jmp     dword ptr [3F12B8]            ;  MSVBVM60.rtcEndOfFile
00407E52   $- FF25 84103F00 jmp     dword ptr [3F1084]            ;  MSVBVM60.__vbaFileClose
00407E58   $- FF25 34113F00 jmp     dword ptr [3F1134]            ;  MSVBVM60.__vbaPrintFile
00407E5E   $- FF25 88103F00 jmp     dword ptr [3F1088]            ;  MSVBVM60.__vbaFileOpen
00407E64   $- FF25 4C103F00 jmp     dword ptr [3F104C]            ;  MSVBVM60.__vbaCastObj
00407E6A   $- FF25 C4123F00 jmp     dword ptr [3F12C4]            ;  MSVBVM60.rtcSetFileAttr
00407E70   $- FF25 C0123F00 jmp     dword ptr [3F12C0]            ;  MSVBVM60.rtcGetFileAttr
00407E76   $- FF25 1C133F00 jmp     dword ptr [3F131C]            ;  MSVBVM60.rtcEnvironVar
00407E7C   $- FF25 88123F00 jmp     dword ptr [3F1288]            ;  MSVBVM60.rtcLowerCaseVar
00407E82   $- FF25 04133F00 jmp     dword ptr [3F1304]            ;  MSVBVM60.rtcMidCharVar
00407E88   $- FF25 E4123F00 jmp     dword ptr [3F12E4]            ;  MSVBVM60.rtcVarBstrFromAnsi
00407E8E   $- FF25 8C123F00 jmp     dword ptr [3F128C]            ;  MSVBVM60.rtcTrimVar
00407E94   $- FF25 94123F00 jmp     dword ptr [3F1294]            ;  MSVBVM60.rtcSpaceVar
00407E9A   $- FF25 FC123F00 jmp     dword ptr [3F12FC]            ;  MSVBVM60.rtcGetObject
```



```
  00407EA0   $- FF25 CC103F00 jmp     dword ptr [3F10CC]            ;  MSVBVM60.__vbaI2Var  

  00407EA6   $- FF25 A0103F00 jmp     dword ptr [3F10A0]            ;  MSVBVM60.__vbaFpI2  

  00407EAC   $- FF25 F0113F00 jmp     dword ptr [3F11F0]            ;  MSVBVM60.__vbaVarSetObjAddref  

  00407EB2   $- FF25 24133F00 jmp     dword ptr [3F1324]            ;  MSVBVM60.rtcFormatNumber  

  00407EB8   $- FF25 10133F00 jmp     dword ptr [3F1310]            ;  MSVBVM60.rtcBstrFromFormatVar  

  00407EBE   $- FF25 D8123F00 jmp     dword ptr [3F12D8]            ;  MSVBVM60.rtcShell  

  00407EC4   $- FF25 90123F00 jmp     dword ptr [3F1290]            ;  MSVBVM60.rtcSpaceBstr  

  00407ECA   $- FF25 28113F00 jmp     dword ptr [3F1128]            ;  MSVBVM60.__vbaObjSetAddref  

  00407ED0   $- FF25 C8113F00 jmp     dword ptr [3F11C8]            ;  MSVBVM60.__vbaVarForNext  

  00407ED6   $- FF25 D8103F00 jmp     dword ptr [3F10D8]            ;  MSVBVM60.__vbaI4Var  

  00407EDC   $- FF25 C4113F00 jmp     dword ptr [3F11C4]            ;  MSVBVM60.__vbaVarForInit  

  00407EE2   $- FF25 C0103F00 jmp     dword ptr [3F10C0]            ;  MSVBVM60.__vbaGenerateBoundsError  

  00407EE8   $- FF25 34103F00 jmp     dword ptr [3F1034]            ;  MSVBVM60.__vbaAryConstruct2  

  00407EEE   $- FF25 0C133F00 jmp     dword ptr [3F130C]            ;  MSVBVM60.rtcDir  

  00407EF4   $- FF25 E0123F00 jmp     dword ptr [3F12E0]            ;  MSVBVM60.rtcStringVar  

  00407EFA   $- FF25 9C113F00 jmp     dword ptr [3F119C]            ;  MSVBVM60.__vbaStrVarMove  

  00407F00   $- FF25 EC123F00 jmp     dword ptr [3F12EC]            ;  MSVBVM60.rtcLeftCharVar  

  00407F06   $- FF25 38103F00 jmp     dword ptr [3F1038]            ;  MSVBVM60.__vbaAryDestruct  

  00407F0C   $- FF25 F4103F00 jmp     dword ptr [3F10F4]            ;  MSVBVM60.__vbaLateIdSt  

  00407F12   $- FF25 3C103F00 jmp     dword ptr [3F103C]            ;  MSVBVM60.__vbaAryLock  

  00407F18   $- FF25 68103F00 jmp     dword ptr [3F1068]            ;  MSVBVM60.__vbaDerefAry1  

  00407F1E   $- FF25 58113F00 jmp     dword ptr [3F1158]            ;  MSVBVM60.__vbaRedim  

  00407F24   $- FF25 D4103F00 jmp     dword ptr [3F10D4]            ;  MSVBVM60.__vbaI4Str  

  00407F2A   $- FF25 00113F00 jmp     dword ptr [3F1100]            ;  MSVBVM60.__vbaLenBstr  

  00407F30   $- FF25 BC113F00 jmp     dword ptr [3F11BC]            ;  MSVBVM60.__vbaVarDiv  

  00407F36   $- FF25 C0113F00 jmp     dword ptr [3F11C0]            ;  MSVBVM60.__vbaVarDup  

  00407F3C   $- FF25 14133F00 jmp     dword ptr [3F1314]            ;  MSVBVM60.rtcVarFromFormatVar  

  00407F42   $- FF25 6C113F00 jmp     dword ptr [3F116C]            ;  MSVBVM60.__vbaStrCmp  

  00407F48   $- FF25 34133F00 jmp     dword ptr [3F1334]            ;  MSVBVM60.rtcCreateObject2  

  00407F4E   $- FF25 F4113F00 jmp     dword ptr [3F11F4]            ;  MSVBVM60.__vbaVarSetVar  

  00407F54   $- FF25 F4123F00 jmp     dword ptr [3F12F4]            ;  MSVBVM60.rtcRightCharVar  

  00407F5A   $- FF25 14123F00 jmp     dword ptr [3F1214]            ;  MSVBVM60.__vbaVarTstNe  

  00407F60   $- FF25 E8103F00 jmp     dword ptr [3F10E8]            ;  MSVBVM60.__vbaLateIdCall  

  00407F66   $- FF25 EC103F00 jmp     dword ptr [3F10EC]            ;  MSVBVM60.__vbaLateIdCallLd  

  00407F6C   $- FF25 50103F00 jmp     dword ptr [3F1050]            ;  MSVBVM60.__vbaCastObjVar  

  00407F72   $- FF25 80113F00 jmp     dword ptr [3F1180]            ;  MSVBVM60.__vbaStrI4  

  00407F78   $- FF25 78113F00 jmp     dword ptr [3F1178]            ;  MSVBVM60.__vbaStrFixstr  

  00407F7E   $- FF25 84123F00 jmp     dword ptr [3F1284]            ;  MSVBVM60.rtcAnsiValueBstr  

  00407F84   $- FF25 68113F00 jmp     dword ptr [3F1168]            ;  MSVBVM60.__vbaStrCat  

  00407F8A   $- FF25 4C113F00 jmp     dword ptr [3F114C]            ;  MSVBVM60.__vbaRecDestruct  

  00407F90   $- FF25 50113F00 jmp     dword ptr [3F1150]            ;  MSVBVM60.__vbaRecDestructAnsi  

  00407F96   $- FF25 44113F00 jmp     dword ptr [3F1144]            ;  MSVBVM60.__vbaRecAnsiToUni  

  00407F9C   $- FF25 54113F00 jmp     dword ptr [3F1154]            ;  MSVBVM60.__vbaRecUniToAnsi  

  00407FA2   $- FF25 C8103F00 jmp     dword ptr [3F10C8]            ;  MSVBVM60.__vbaI2I4  

  00407FA8   $- FF25 70113F00 jmp     dword ptr [3F1170]            ;  MSVBVM60.__vbaStrCopy  

  00407FAE   $- FF25 E8123F00 jmp     dword ptr [3F12E8]            ;  MSVBVM60.rtcLeftCharBstr  

  00407FB4   $- FF25 84113F00 jmp     dword ptr [3F1184]            ;  MSVBVM60.__vbaStrMove  

  00407FBA   $- FF25 DC103F00 jmp     dword ptr [3F10DC]            ;  MSVBVM60.__vbaInStr  

  00407FC0   $- FF25 B4103F00 jmp     dword ptr [3F10B4]            ;  MSVBVM60.__vbaFreeStrList  

  00407FC6   $- FF25 94113F00 jmp     dword ptr [3F1194]            ;  MSVBVM60.__vbaStrToUnicode  

  00407FCC   $- FF25 08113F00 jmp     dword ptr [3F1108]            ;  MSVBVM60.__vbaLsetFixstr  

  00407FD2   $- FF25 90113F00 jmp     dword ptr [3F1190]            ;  MSVBVM60.__vbaStrToAnsi  

  00407FD8   $- FF25 8C103F00 jmp     dword ptr [3F108C]            ;  MSVBVM60.__vbaFixstrConstruct  

  00407FDE   $- FF25 78103F00 jmp     dword ptr [3F1078]            ;  MSVBVM60.__vbaExitProc  

  00407FE4   $- FF25 70103F00 jmp     dword ptr [3F1070]            ;  MSVBVM60.__vbaErrorOverflow  

  00407FEA   $- FF25 64113F00 jmp     dword ptr [3F1164]            ;  MSVBVM60.__vbaSetSystemError  

  00407FF0   $- FF25 A8113F00 jmp     dword ptr [3F11A8]            ;  MSVBVM60.__vbaVarAdd  

  00407FF6   $- FF25 E4113F00 jmp     dword ptr [3F11E4]            ;  MSVBVM60.__vbaVarMove  

  00407FFC   $- FF25 B8113F00 jmp     dword ptr [3F11B8]            ;  MSVBVM60.__vbaVarCopy  

  00408002   $- FF25 E0113F00 jmp     dword ptr [3F11E0]            ;  MSVBVM60.__vbaVarMod  

  00408008   $- FF25 0C123F00 jmp     dword ptr [3F120C]            ;  MSVBVM60.__vbaVarTstEq  

  0040800E   $- FF25 AC103F00 jmp     dword ptr [3F10AC]            ;  MSVBVM60.__vbaFreeObjList  

  00408014   $- FF25 40103F00 jmp     dword ptr [3F1040]            ;  MSVBVM60.__vbaAryUnlock  

  0040801A   $- FF25 E8113F00 jmp     dword ptr [3F11E8]            ;  MSVBVM60.__vbaVarNot  

  00408020   $- FF25 AC113F00 jmp     dword ptr [3F11AC]            ;  MSVBVM60.__vbaVarCat  

  00408026   $- FF25 D0123F00 jmp     dword ptr [3F12D0]            ;  MSVBVM60.rtcMsgBox  

  0040802C   $- FF25 C8123F00 jmp     dword ptr [3F12C8]            ;  MSVBVM60.rtcRgb  

  00408032   $- FF25 48103F00 jmp     dword ptr [3F1048]            ;  MSVBVM60.__vbaBoolVarNull  

  00408038   $- FF25 A8103F00 jmp     dword ptr [3F10A8]            ;  MSVBVM60.__vbaFreeObj  

  0040803E   $- FF25 20133F00 jmp     dword ptr [3F1320]            ;  MSVBVM60.rtcErrObj  

  00408044   $- FF25 24113F00 jmp     dword ptr [3F1124]            ;  MSVBVM60.__vbaObjSet  

  0040804A   $- FF25 2C113F00 jmp     dword ptr [3F112C]            ;  MSVBVM60.__vbaObjVar  

  00408050   $- FF25 F8103F00 jmp     dword ptr [3F10F8]            ;  MSVBVM60.__vbaLateMemCall  

  00408056   $- FF25 20113F00 jmp     dword ptr [3F1120]            ;  MSVBVM60.__vbaNextEachVar  

  0040805C   $- FF25 D8113F00 jmp     dword ptr [3F11D8]            ;  MSVBVM60.__vbaVarLateMemCallLdRf  

  00408062   $- FF25 D4113F00 jmp     dword ptr [3F11D4]            ;  MSVBVM60.__vbaVarLateMemCallLd  

  00408068   $- FF25 1C123F00 jmp     dword ptr [3F121C]            ;  MSVBVM60.__vbaVarZero  
```



> 0040806E   $- FF25 9C103F00 jmp     dword ptr `[3F109C]`            ;  `MSVBVM60.__vbaForEachVar`  
> 00408074   $- FF25 EC113F00 jmp     dword ptr `[3F11EC]`            ;  `MSVBVM60.__vbaVarSetObj`  
> 0040807A   $- FF25 B0103F00 jmp     dword ptr `[3F10B0]`            ;  `MSVBVM60.__vbaFreeStr`  
> 00408080   $- FF25 C4103F00 jmp     dword ptr `[3F10C4]`            ;  `MSVBVM60.__vbaHresultCheckObj`  
> 00408086   $- FF25 18123F00 jmp     dword ptr `[3F1218]`            ;  `MSVBVM60.__vbaVarVargNofree`  
> 0040808C   $- FF25 A0113F00 jmp     dword ptr `[3F11A0]`            ;  `MSVBVM60.__vbaStrVarVal`  
> 00408092   $- FF25 10113F00 jmp     dword ptr `[3F1110]`            ;  `MSVBVM60.__vbaNew2`  
> 00408098   $- FF25 BC103F00 jmp     dword ptr `[3F10BC]`            ;  `MSVBVM60.__vbaFreeVarList`  
> 0040809E   $- FF25 18133F00 jmp     dword ptr `[3F1318]`            ;  `MSVBVM60.rtcDateDiff`  
> 004080A4   $- FF25 10123F00 jmp     dword ptr `[3F1210]`            ;  `MSVBVM60.__vbaVarTstGe`  
> 004080AA   $- FF25 D4123F00 jmp     dword ptr `[3F12D4]`            ;  `MSVBVM60.rtcDoEvents`  
> 004080B0   $- FF25 B8103F00 jmp     dword ptr `[3F10B8]`            ;  `MSVBVM60.__vbaFreeVar`  
> 004080B6   $- FF25 A4123F00 jmp     dword ptr `[3F12A4]`            ;  `MSVBVM60.rtcGetPresentDate`  
> 004080BC   $- FF25 64103F00 jmp     dword ptr `[3F1064]`            ;  `MSVBVM60.__vbaDateVar`  
> 004080C2   $- FF25 30113F00 jmp     dword ptr `[3F1130]`            ;  `MSVBVM60.__vbaOnError`  
> 004080C8   $- FF25 F0123F00 jmp     dword ptr `[3F12F0]`            ;  `MSVBVM60.rtcRightCharBstr`  
> 004080CE   $- FF25 08133F00 jmp     dword ptr `[3F1308]`            ;  `MSVBVM60.VarPtr`  
> 004080D4   $- FF25 BC123F00 jmp     dword ptr `[3F12BC]`            ;  `MSVBVM60.rtcHexVarFromVar`  
> 004080DA   $- FF25 DC123F00 jmp     dword ptr `[3F12DC]`            ;  `MSVBVM60.rtcStringBstr`  
> 004080E0   $- FF25 40113F00 jmp     dword ptr `[3F1140]`            ;  `MSVBVM60.__vbaRaiseEvent`  
> 004080E6   $- FF25 80103F00 jmp     dword ptr `[3F1080]`            ;  `MSVBVM60.__vbaFailedFriend`  
> 004080EC   $- FF25 44103F00 jmp     dword ptr `[3F1044]`            ;  `MSVBVM60.__vbaBoolVar`  
> 004080F2   $- FF25 A0123F00 jmp     dword ptr `[3F12A0]`            ;  `MSVBVM60.rtcBstrFromAnsi`  
> 004080F8   $- FF25 98123F00 jmp     dword ptr `[3F1298]`            ;  `MSVBVM60.rtcUpperCaseBstr`  
> 004080FE   $- FF25 00133F00 jmp     dword ptr `[3F1300]`            ;  `MSVBVM60.rtcMidCharBstr`  
> 00408104   $- FF25 98103F00 jmp     dword ptr `[3F1098]`            ;  `MSVBVM60.__vbaForEachCollObj`  
> 0040810A   $- FF25 1C113F00 jmp     dword ptr `[3F111C]`            ;  `MSVBVM60.__vbaNextEachCollObj`  
> 00408110   $- FF25 60113F00 jmp     dword ptr `[3F1160]`            ;  `MSVBVM60.__vbaResume`  
> 00408116   $- FF25 18113F00 jmp     dword ptr `[3F1118]`            ;  `MSVBVM60.__vbaNextEachCollAd`  
> 0040811C   $- FF25 FC103F00 jmp     dword ptr `[3F10FC]`            ;  `MSVBVM60.__vbaLateMemCallLd`  
> 00408122   $- FF25 94103F00 jmp     dword ptr `[3F1094]`            ;  `MSVBVM60.__vbaForEachCollAd`  
> 00408128   $- FF25 FC113F00 jmp     dword ptr `[3F11FC]`            ;  `MSVBVM60.__vbaVarSub`  
> 0040812E   $- FF25 E0103F00 jmp     dword ptr `[3F10E0]`            ;  `MSVBVM60.__vbaInStrVar`  
> 00408134   $- FF25 04123F00 jmp     dword ptr `[3F1204]`            ;  `MSVBVM60.__vbaVarTextTstGt`  
> 0040813A   $- FF25 0C113F00 jmp     dword ptr `[3F110C]`            ;  `MSVBVM60.__vbaNew`  
> 00408140   $- FF25 5C103F00 jmp     dword ptr `[3F105C]`            ;  `MSVBVM60.__vbaCopyBytes`  
> 00408146   $- FF25 48113F00 jmp     dword ptr `[3F1148]`            ;  `MSVBVM60.__vbaRecAssign`  
> 0040814C   $- FF25 08123F00 jmp     dword ptr `[3F1208]`            ;  `MSVBVM60.__vbaVarTextTstNe`  
> 00408152   $- FF25 8C113F00 jmp     dword ptr `[3F118C]`            ;  `MSVBVM60.__vbaStrTextCmp`  
> 00408158   $- FF25 00123F00 jmp     dword ptr `[3F1200]`            ;  `MSVBVM60.__vbaVarTextTstEq`  
> 0040815E   $- FF25 F8113F00 jmp     dword ptr `[3F11F8]`            ;  `MSVBVM60.__vbaVarSetVarAddref`  
> 00408164   $- FF25 54103F00 jmp     dword ptr `[3F1054]`            ;  `MSVBVM60.__vbaCheckTypeVar`  
> 0040816A   $- FF25 B0123F00 jmp     dword ptr `[3F12B0]`            ;  `MSVBVM60.rtcIsObject`  
> 00408170   $- FF25 D0103F00 jmp     dword ptr `[3F10D0]`            ;  `MSVBVM60.__vbaI4ErrVar`  
> 00408176   $- FF25 AC123F00 jmp     dword ptr `[3F12AC]`            ;  `MSVBVM60.rtcIsNumeric`  
> 0040817C   $- FF25 CC123F00 jmp     dword ptr `[3F12CC]`            ;  `MSVBVM60.rtcIsMissing`  
> 00408182   $- FF25 A8123F00 jmp     dword ptr `[3F12A8]`            ;  `MSVBVM60.rtcIsEmpty`  
> 00408188   $- FF25 28133F00 jmp     dword ptr `[3F1328]`            ;  `MSVBVM60.rtcFormatDateTime`  
> 0040818E   $- FF25 D0113F00 jmp     dword ptr `[3F11D0]`            ;  `MSVBVM60.__vbaVarIndexLoadRefLock`  
> 00408194   $- FF25 14113F00 jmp     dword ptr `[3F1114]`            ;  `MSVBVM60.__vbaNextEachAry`  
> 0040819A   $- FF25 CC113F00 jmp     dword ptr `[3F11CC]`            ;  `MSVBVM60.__vbaVarIndexLoad`  
> 004081A0   $- FF25 2C133F00 jmp     dword ptr `[3F132C]`            ;  `MSVBVM60.rtcSplit`  
> 004081A6   $- FF25 90103F00 jmp     dword ptr `[3F1090]`            ;  `MSVBVM60.__vbaForEachAry`  
> 004081AC   $- FF25 5C113F00 jmp     dword ptr `[3F115C]`            ;  `MSVBVM60.__vbaRedimPreserve`  
> 004081B2   $- FF25 A4113F00 jmp     dword ptr `[3F11A4]`            ;  `MSVBVM60.__vbaUbound`  
> 004081B8   $- FF25 B0113F00 jmp     dword ptr `[3F11B0]`            ;  `MSVBVM60.__vbaVarCmpEq`  
> 004081BE   $- FF25 60103F00 jmp     dword ptr `[3F1060]`            ;  `MSVBVM60.__vbaDateR8`  
> 004081C4   .- FF25 10103F00 jmp     dword ptr `[3F1010]`            ;  `MSVBVM60.EVENT_SINK_QueryInterface`  
> 004081CA   .- FF25 0C103F00 jmp     dword ptr `[3F100C]`            ;  `MSVBVM60.EVENT_SINK_AddRef`  
> 004081D0   .- FF25 14103F00 jmp     dword ptr `[3F1014]`            ;  `MSVBVM60.EVENT_SINK_Release`  
> 004081D6   .- FF25 5C123F00 jmp     dword ptr `[3F125C]`            ;  `MSVBVM60.GetMem2`  
> 004081DC   .- FF25 68123F00 jmp     dword ptr `[3F1268]`            ;  `MSVBVM60.PutMem2`  
> 004081E2   .- FF25 64123F00 jmp     dword ptr `[3F1264]`            ;  `MSVBVM60.GetMemObj`  
> 004081E8   .- FF25 70123F00 jmp     dword ptr `[3F1270]`            ;  `MSVBVM60.PutMemObj`  
> 004081EE   .- FF25 74123F00 jmp     dword ptr `[3F1274]`            ;  `MSVBVM60.SetMemObj`  
> 004081F4   .- FF25 60123F00 jmp     dword ptr `[3F1260]`            ;  `MSVBVM60.GetMem4`  
> 004081FA   .- FF25 6C123F00 jmp     dword ptr `[3F126C]`            ;  `MSVBVM60.PutMem4`  
> 00408200   .- FF25 78123F00 jmp     dword ptr `[3F1278]`            ;  `MSVBVM60.GetMemNewObj`  
> 00408206   .- FF25 7C123F00 jmp     dword ptr `[3F127C]`            ;  `MSVBVM60.PutMemNewObj`  
> 0040820C   .- FF25 80123F00 jmp     dword ptr `[3F1280]`            ;  `MSVBVM60.SetMemNewObj`  
> 00408212   $- FF25 58123F00 jmp     dword ptr `[3F1258]`            ;  `MSVBVM60.ThunRTMain`  
> 00408218   .  68 78844000   push    `00408478`                   ;  ASCII `"VB5!6&*"`  
>
> 全部还原了，呵呵，不过大家可能会问为什么 `IAT` 所在的区段全在 `3F0000` 段呢？
>
> 这就是因为我们使用的是申请的空间，这样对我们造成不了任何影响，遇到这种情况我们使用高级指令来获取 `IAT`，如果你使用的地址是程序内的空白地址，那么就不需要使用高级指令来获取 `IAT` 了。
>
> ### 五、脱壳
>
> ![名称: 3.PNG> 查看次数: 177> 文件大小: 117.5 KB](img/285ca6e8a087e944da8d3448914c28a0.jpg)  
>
> ![名称: 4.PNG> 查看次数: 174> 文件大小: 7.3 KB](img/bdd645f57feb75259456aeb79989915f.jpg)  
>
> ![名称: 5.PNG> 查看次数: 169> 文件大小: 1.9 KB](img/afd1f60888e933668d8a1b47e717e9d2.jpg)  
>
> 减掉一个无效指针  
>
> ![名称: 6.PNG> 查看次数: 175> 文件大小: 3.3 KB](img/b62a39963febedd15d13f9b60a07974e.jpg)



**注意**：使用高级指令 `Call` 获取 `IAT` 的话，一定要段上 `Create New IAT`

### 六、总结

1.  最好是选择程序里面的空白地址来放 `IAT` 表，但是，空间得足够大
2.  在设置搜索起始和结束地址的时候，如果你可以确定具体返回的话，尽量设置小一些的范围，这样可以提高处理速度，一般可以设置为 `Code` 段的开始地址 `[CodeBase]` 和 `Code` 段的结束地址 `[CodeBase+CodeSize]`
3.  本文介绍的是如何还原 `Jmp dword ptr ds:[API]`。`Call dword ptr ds:[API]` 类型的处理需要变换下代码。将在下文中介绍
4.  经测试，脱壳文件基本没有跨平台问题

### 七、脚本的使用

需要注意已经打了注释的个地方。

忽略除了特权指令和指定异常外的所有异常

```
/*
Script written by wynney

Date:   2007-04-20
Script: SoftWrap Fixer for VB
Environment : OllyDbg 1.1, ODBGScript 1.52,Winxp Sp2
Debugging options: Ignore all outside of "Invalid or privileged" and "Custom" exception
Thanks :
           kanxue      - author of HideOD        
           hnhuqiong   - author of ODbgScript 1.52
*/

var ImagBase
var ImagSize
var CBase
var CSize
var SrStart
var SrEnd
var IATNew
var temp
var Pointer
var DllBase
var OEPS
var fix

GetBase:
         dbh
         BPHWCALL
         GMI eip, ModuleBase
         cmp $RESULT,0
         je error
         mov ImagBase,$RESULT
         GMI eip,ModuleSize
         cmp $RESULT,0
         je error
         mov ImagSize,$RESULT
         GMI eip,CodeBase     
         cmp $RESULT,0
         je error
         mov CBase,$RESULT
         GMI eip, CodeSize
         cmp $RESULT,0
         je error
         mov CSize,$RESULT
         
WorkFor:
         mov SrStart,CBase
         mov temp,CBase
         add temp,CSize
         mov SrEnd,temp
         Alloc 10000
         mov DllBase,$RESULT
         mov temp,$RESULT
         add temp,4
         mov Pointer,temp
         add temp,4
         mov IATNEW,temp
      

GetTable:
         GPA "VirtualAlloc","kernel32.Dll"
         cmp $RESULT,0
         je error
         find $RESULT,#C21000#
         cmp $RESULT,0
         je error
         mov temp,$RESULT
         bp temp
         esto
         bc temp
         sto
         find eip,#85C00F84????????F6C3027421FFB5????????6A1050FFB5????????FFB5????????#
         cmp $RESULT,0
         je error
         add $RESULT,8
         mov temp,$RESULT
         BP temp,"x"
         esto
         BC temp
         mov [DllBase],ecx
         mov [Pointer],IATNEW  
         mov temp,eip
         mov [temp],#3A0D90909090#
         add temp,2
         mov [temp],DllBase
         add temp,4
         mov [temp],#740E#
         add temp,2
         mov [temp],#FE0590909090#
         add temp,2
         mov [temp],DllBase
         add temp,4
         mov [temp],#6683059090909004#  
         add temp,3
         mov [temp],Pointer
         add temp,5
         mov [temp],#8B1D90909090#
         add temp,2
         mov [temp],Pointer
         add temp,4
         mov [temp],#89036683059090909004#
         add temp,5
         mov [temp],Pointer          

GoToOEP:
         esto
         esto
         esto
         esto
         esto
         esto
         BPRM CBase,CSize
         esto
         esto
         BMC

ASKSt:
         MSGYN "是否到达 OEP？"  
         cmp $RESULT,0
         je Manuly
         jmp JmpAPI    
    
Manuly:
         msg "请手动到达 OEP，再继续！"         //若没到达 OEP，手动到达再点插件的"继续"
         Pause
         jmp JmpAPI

JmpAPI:   
 mov OEPS,eip
 PAUSE
 ask "请输入 Jmp 类型-Call Dword PTR DS:[Addr]中的 Addr"  //输入要修改的地址
 cmp $RESULT,0
 je error
 mov fix,$RESULT
 sti
 sti
 sti
 find eip,#68????????C3#
 cmp $RESULT,0
 je error
 mov temp,$RESULT
 bp temp
 esto
 bc temp
 sti
 sti
         find eip,#8B049635????????508A00040F#
         cmp $RESULT,0
         je error
         bp $RESULT
         esto
         bc $RESULT
         add $RESULT,8
         mov temp,$RESULT
         mov [Pointer],CBase
         mov [temp],#B990909090#
         add temp,1
         mov [temp],IATNEW
         add temp,4
         mov [temp],#3901740583C104EBF78BD181C200100000890283C41C3E8B042483E80666C700FF2583C00289109090#
         add temp,29
         mov [temp],#813D9090909090909090#
         add temp,2
         mov [temp],Pointer
         add temp,4
         mov [temp],SrEnd
         add temp,4
         mov [temp],#742C8B0D90909090#
         add temp,4
         mov [temp],Pointer
         add temp,4
         mov [temp],#668139FF1575178BC183C002813890909090#
         add temp,0E
         mov [temp],fix
         add temp,4
         mov [temp],#750A83C004A390909090FFE1FF0590909090EBC8EBFE90909090#
         add temp,6
         mov [temp],Pointer
         add temp,8
         mov [temp],Pointer
         add temp,6
         bp temp
         esto
         bc temp
         mov eip,OEPS

Done:
         msg "请使用 ImportREC 高级指令修复脱壳文件"
         ret          
  
error:
         msg "非支持版本"
         ret
```

---

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2007 年 04 月 23 日
