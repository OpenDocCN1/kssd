
# PESpin v1.1 外壳分析(全篇)

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-528.htm`](https://www.pediy.com/kssd/pediy07/pediy7-528.htm)

【目标】:`PESpin` `v1.1` 主程序  

【工具】:`Olydbg1.1` (diy 版)、`LORDPE`、`ImportREC1.6F`  

【任务】:分析外壳  

【操作平台】:`WinXP` `sp2`  

【作者】: `LOVEBOOM` `[DFCG]` `[FCG]` `[US]`  

【相关链接】: 自己去上网搜搜  

【简要说明】： 这篇文章算是给 `yock` 的一份礼物吧，前一段时间我答应他看看这个版本的壳，拖了这么久真不好意思 `J`,上次看过一下，发现这个版本比上一版本增强了不少。要 `patch` 的代码也多很多的,壳新增了一个非常有用的东西 `SDK`,用上 `SDK` 去加程序增强不少，不过壳的 `PE Header` 抽代码显得有点鸡肋的感觉 `J`。  

【详细过程】:  

`PESpin` `v0.7` 开始就从头到尾看了一下，这个版本同样也看看，主要是看看有没有什么改进的地方，不过结果比较遗憾，在 `Loader` 里没有什么新的变化,到现在壳还不 `anti-OllyDbg`,不知道是不是作者用意的放水.`J`。

分两步进行：分析，脱壳。

## 第一步:分析

`OD` 载入目标程序，慢慢的分析,细细的品味 `^\_^`。

```
00412087  >  /EB 01            JMP SHORT 0041208A                     ; EP
00412089   |90               NOP
0041208A   \\60               PUSHAD
0041208B    E8 00000000       CALL 00412090
00412090    8B1C24            MOV EBX,DWORD PTR SS:[ESP]              ; SMC
00412093    83C3 12           ADD EBX,12
00412096    812B E8B10600     SUB DWORD PTR DS:[EBX],6B1E8
0041209C    FE4B FD           DEC BYTE PTR DS:[EBX-3]
0041209F    822C24 7D         SUB BYTE PTR SS:[ESP],7D
004120A3    DE46 00           FIADD WORD PTR DS:[ESI]
004120A6    0BE4             OR ESP,ESP
004120A8  ^ 74 9E            JE SHORT 00412048
...
004120F1    8B95 C34B4000     MOV EDX,DWORD PTR SS:[EBP+404BC3]      ; [EBP+404BC3]=hModule(400000)
004120F7    8B42 3C           MOV EAX,DWORD PTR DS:[EDX+3C]
004120FA    03C2              ADD EAX,EDX
004120FC    8985 CD4B4000     MOV DWORD PTR SS:[EBP+404BCD],EAX      ; [EBP+404BCD]保存 peHeader(4000D0)
...
00412134    41               INC ECX
00412135    C1E1 07          SHL ECX,7
00412138    8B0C01           MOV ECX,DWORD PTR DS:[ECX+EAX]          ; 定位输入表 RVA(12000)
0041213B    03CA             ADD ECX,EDX                             ; 转为 VA
...
0041214E    8B59 10          MOV EBX,DWORD PTR DS:[ECX+10]          ; 定位 OriginalFirstThunk
00412151    03DA             ADD EBX,EDX
00412153    8B1B             MOV EBX,DWORD PTR DS:[EBX]             ; 取出 MessageBoxA 的地址
00412155    899D E14B4000     MOV DWORD PTR SS:[EBP+404BE1],EBX      ; 结果保存到[EBP+404BE1]处
0041215B    53               PUSH EBX
0041215C    8F85 D7494000     POP DWORD PTR SS:[EBP+4049D7]         ; 地址同时保存在[EBP+4049D7]中
00412162    BB CC000000       MOV EBX,0CC
00412167    B9 FE110000       MOV ECX,11FE
0041216C    8DBD 714C4000     LEA EDI,DWORD PTR SS:[EBP+404C71]
00412172    4F               DEC EDI
...
0041217F    301C39            XOR BYTE PTR DS:[ECX+EDI],BL
00412182    FECB              DEC BL
00412184    49               DEC ECX
00412185    9C               PUSHFD
00412186    C12C24 06        SHR DWORD PTR SS:[ESP],6
0041218A    F71424           NOT DWORD PTR SS:[ESP]
0041218D    832424 01        AND DWORD PTR SS:[ESP],1
00412191    50               PUSH EAX
00412192    52               PUSH EDX
00412193    B8 83B2DC12       MOV EAX,12DCB283
00412198    05 444D23ED       ADD EAX,ED234D44
0041219D    F76424 08        MUL DWORD PTR SS:[ESP+8]
004121A1    8D8428 BD2D4000  LEA EAX,DWORD PTR DS:[EAX+EBP+402DBD]
004121A8    894424 08        MOV DWORD PTR SS:[ESP+8],EAX
004121AC    5A               POP EDX
004121AD    58               POP EAX
004121AE    8D6424 04        LEA ESP,DWORD PTR SS:[ESP+4]
004121B2    FF6424 FC        JMP DWORD PTR SS:[ESP-4]              ; 从 415269 处开始向前解压代码， size 为 11FE
...
004121CE    8170 03 E89868EA XOR DWORD PTR DS:[EAX+3],EA6898E8      ; SMC
004121D5    83C0 21          ADD EAX,21
...
004121E3    68 CB000000       PUSH 0CB
004121E8    59               POP ECX                               ; 解码大小 0CB
004121E9    8DBD A35D4000     LEA EDI,DWORD PTR SS:[EBP+405DA3]     ; [EBP+405DA3]=[41519E]
...
004121E3    68 CB000000       PUSH 0CB
004121E8    59               POP ECX                               ; 解码大小 0CB
004121E9    8DBD A35D4000     LEA EDI,DWORD PTR SS:[EBP+405DA3]     ; [EBP+405DA3]=[41519E]
004121EF    90               NOP
004121F0    90               NOP
004121F1    90               NOP
004121F2    90               NOP
004121F3    90               NOP
004121F4    90               NOP
004121F5    90               NOP
004121F6    90               NOP
004121F7    90               NOP
004121F8    90               NOP
004121F9    90               NOP
004121FA    90               NOP
004121FB    90               NOP
004121FC    90               NOP
004121FD    90               NOP
004121FE    90               NOP
004121FF    90               NOP
00412200    C00C39 02        ROR BYTE PTR DS:[ECX+EDI],2          ; KEY=2
00412204    49               DEC ECX
...
00412205    9C               PUSHFD
00412206    C12C24 06        SHR DWORD PTR SS:[ESP],6
0041220A    F71424           NOT DWORD PTR SS:[ESP]
0041220D    832424 01        AND DWORD PTR SS:[ESP],1
00412211    50               PUSH EAX
00412212    52               PUSH EDX
00412213    B8 72B2DC12       MOV EAX,12DCB272
00412218    05 444D23ED       ADD EAX,ED234D44
0041221D    F76424 08        MUL DWORD PTR SS:[ESP+8]
00412221    8D8428 3E2E4000  LEA EAX,DWORD PTR DS:[EAX+EBP+402E3E]
00412228 >  894424 08        MOV DWORD PTR SS:[ESP+8],EAX          ; PESpin.00412239
0041222C    5A               POP EDX
0041222D    58               POP EAX
0041222E    8D6424 04        LEA ESP,DWORD PTR SS:[ESP+4]
00412232    FF6424 FC        JMP DWORD PTR SS:[ESP-4]              ; 循环解压从 415269 处开始向上解压，解压大小为 0CB
...
00413F09    8B7C24 20       MOV EDI,DWORD PTR SS:[ESP+20]         ; 获取 KERNELBASE
00413F0D    81E7 0000FFFF   AND EDI,FFFF0000
...
00413F23    90             NOP
00413F24    BA 246BDE21     MOV EDX,21DE6B24
00413F29    81F2 6931DE21   XOR EDX,21DE3169                    ; EDX=PE sig(5A4D)
00413F2F    66:3917         CMP WORD PTR DS:[EDI],DX
00413F32    75 17          JNZ SHORT 00413F4B                  ; 判断是否定位到 DOS header
00413F34    81C2 EFA5FFFF   ADD EDX,FFFFA5EF
00413F3A    0FB7143A        MOVZX EDX,WORD PTR DS:[EDX+EDI]
00413F3E    66:F7C2 00F8    TEST DX,0F800
00413F43    75 06          JNZ SHORT 00413F4B
00413F45    3B7C3A 34       CMP EDI,DWORD PTR DS:[EDX+EDI+34]
00413F49    74 08          JE SHORT 00413F53
00413F4B    81EF 00000100   SUB EDI,10000                     ; UNICODE "ALLUSERSPROFILE=D:\Documents and Settings\All Users"
00413F51  ^ EB C0          JMP SHORT 00413F13                  ; 减 10000 继续回去
00413F53    97             XCHG EAX,EDI                        ; 获取出来的 KERNELBASE 保存到 EAX
...
00413F65    68 F44B4000     PUSH 00404BF4
00413F6A    50             PUSH EAX                            ; push kerbase(7c800000)
00413F6B    8785 E54B4000   XCHG DWORD PTR SS:[EBP+404BE5],EAX  ; 保存 KERNELBASE 到[EBP+404BE5]=(413FE0)
00413F71    016C24 04       ADD DWORD PTR SS:[ESP+4],EBP
00413F75    8D85 ECA183EB   LEA EAX,DWORD PTR SS:[EBP+EB83A1EC]
00413F7B    8D80 BDAABC14   LEA EAX,DWORD PTR DS:[EAX+14BCAABD]
...
00413F8A    FFD0            CALL EAX                          ; EAX=4140A4 这里面就是获取相关 API 的地址

进去看看：

004140A4    59             POP ECX
004140A5    58             POP EAX
004140A6    5F             POP EDI                             ; EDI=413FEF
004140A7    90             NOP
004140A8    90             NOP
004140A9    90             NOP
```



004140AA 90 NOP  
004140AB 90 NOP  
004140AC 90 NOP  
004140AD 90 NOP  
004140AE 90 NOP  
004140AF 90 NOP  
004140B0 41 INC `ECX`  
004140B1 41 INC `ECX`  
004140B2 51 PUSH `ECX` ; `ECX=413F8E`  
004140B3 8BF0 MOV `ESI, EAX`  
004140B5 0340 3C ADD `EAX, DWORD PTR DS:[EAX+3C]` ; 定位 `PE header`  
004140B8 8B40 78 MOV `EAX, DWORD PTR DS:[EAX+78]` ; 定位输出表  
004140BB 03C6 ADD `EAX, ESI`  
004140BD FF70 20 PUSH `DWORD PTR DS:[EAX+20]` ; `AddressofNames`  
004140C0 5B POP `EBX`  
004140C1 03DE ADD `EBX, ESI`  
004140C3 FF70 18 PUSH `DWORD PTR DS:[EAX+18]` ; `NumberofNames`  
004140C6 8F85 674D4000 POP `DWORD PTR SS:[EBP+404D67]` ; `[EBP+404D67]` 保存 `NumberofNames`  
004140CC FF70 24 PUSH `DWORD PTR DS:[EAX+24]` ; `AddressofNamesOrdnials`  
004140CF 5A POP `EDX`  
004140D0 03D6 ADD `EDX, ESI`  
004140D2 FF70 1C PUSH `DWORD PTR DS:[EAX+1C]` ; `AddressofFunctions`  
004140D5 59 POP `ECX`  
004140D6 03CE ADD `ECX, ESI`  
004140D8 898D 574D4000 MOV `DWORD PTR SS:[EBP+404D57], ECX` ; `[EBP+404D57]` 保存 `AddressofFunctions`  
004140DE 83EF 05 SUB `EDI, 5`  
004140E1 83C7 05 ADD `EDI, 5`  
004140E4 833F 00 CMP `DWORD PTR DS:[EDI], 0`  
004140E7 0F84 9D000000 JE `0041418A`  

004140ED 8A07 MOV `AL, BYTE PTR DS:[EDI]`  
004140EF 8885 1B4D4000 MOV `BYTE PTR SS:[EBP+404D1B], AL`  
004140F5 FF77 01 PUSH `DWORD PTR DS:[EDI+1]`  
004140F8 8F85 474D4000 POP `DWORD PTR SS:[EBP+404D47]`  
004140FE 53 PUSH `EBX`  
004140FF 52 PUSH `EDX`  
00414100 57 PUSH `EDI`  
00414101 2BC9 SUB `ECX, ECX`  
00414103 90 NOP  
00414104 90 NOP  
00414105 90 NOP  
00414106 90 NOP  
00414107 90 NOP  
00414108 90 NOP  
00414109 90 NOP  
0041410A 90 NOP  
0041410B 90 NOP  
0041410C 90 NOP  
0041410D 90 NOP  
0041410E 90 NOP  
0041410F 8B3B MOV `EDI, DWORD PTR DS:[EBX]`  
00414111 03FE ADD `EDI, ESI`  
00414113 807F 02 61 CMP `BYTE PTR DS:[EDI+2], 61` ; 获取 `LoadLibraryA` 的地址  
00414117 75 43 JNZ SHORT `0041415C`  
00414119 E8 02000000 CALL `00414120`  
0041411E 90 NOP  
0041411F 90 NOP  
00414120 58 POP `EAX`  
00414121 8D6424 FC LEA `ESP, DWORD PTR SS:[ESP-4]`  
00414125 05 23000000 ADD `EAX, 23`  
0041412A 890424 MOV `DWORD PTR SS:[ESP], EAX`  
0041412D 8D85 CA8A94ED LEA `EAX, DWORD PTR SS:[EBP+ED948ACA]`  
00414133 2D 353D54ED SUB `EAX, ED543D35`  
00414138 50 PUSH `EAX`  
00414139 C3 RETN  
0041413A 3BC3 CMP `EAX, EBX`  
0041413C 74 35 JE SHORT `00414173`  
0041413E 2BC2 SUB `EAX, EDX`  
00414140 9A 3D72423E C07 CALL FAR `75C0:3E42723D` ; Far call  
00414147 14 8D ADC `AL, 8D`  
00414149 04 4A ADD `AL, 4A`  
0041414B 0FB700 MOVZX `EAX, WORD PTR DS:[EAX]`  
0041414E C1E0 02 SHL `EAX, 2`  
00414151 05 5426807C ADD `EAX, 7C802654`  
00414156 8B00 MOV `EAX, DWORD PTR DS:[EAX]`  
00414158 03C6 ADD `EAX, ESI`  
0041415A EB 0E JMP SHORT `0041416A`  
0041415C 83C3 04 ADD `EBX, 4`  
0041415F 41 INC `ECX`  
00414160 81F9 B5030000 CMP `ECX, 3B5`  
00414166 ^ 75 A7 JNZ SHORT `0041410F`  
00414168 33C0 XOR `EAX, EAX`  
0041416A 5F POP `EDI`  
0041416B 5A POP `EDX`  
0041416C 5B POP `EBX`  
0041416D 0BC0 OR `EAX, EAX`  
0041416F 74 1B JE SHORT `0041418C`  
00414171 90 NOP  
00414172 90 NOP  
00414173 90 NOP  
00414174 90 NOP  
00414175 90 NOP  
00414176 90 NOP  
00414177 90 NOP  
00414178 90 NOP  
00414179 90 NOP  
0041417A 8038 CC CMP `BYTE PTR DS:[EAX], 0CC` ; 判断有没有下断点  
0041417D 75 03 JNZ SHORT `00414182`  
0041417F 8028 00 SUB `BYTE PTR DS:[EAX], 0`  
00414182 8947 01 MOV `DWORD PTR DS:[EDI+1], EAX`  
00414185 ^ E9 57FFFFFF JMP `004140E1`  
0041418A 0BC0 OR `EAX, EAX`  
0041418C EB 01 JMP SHORT `0041418F`  
0041418E 90 NOP  
0041418F C3 RETN  

获取了下面几个 API:

`LoadLibraryA`

`ExitProcess`

`GetProcAddress`

`VirtualProtect`

`CloseHandle`

`VirtualAlloc`

`VirtualFree`

`CreateFileA`

`ReadFile`

`GetTickCount`

`GetModuleHandleA`

`CreateThread`

`Sleep`

`GetCurrentProcessID`

`OpenProcess`

`TerminateProcess`

`GetFileSize`

`GetModuleFileNameA`

……

00412267 B8 944380EF MOV `EAX, EF804394`  
0041226C 2BC9 SUB `ECX, ECX`  
0041226E 83C9 15 OR `ECX, 15`  
00412271 0FA3C8 BT `EAX, ECX`  
00412274 0F83 81000000 JNB `004122FB` ; 如果没有设置保护密码这里就跳，因此如果是要输入密码的程序，强行跳过是没有用的  
0041227A 8DB40D D44B4000 LEA `ESI, DWORD PTR SS:[EBP+ECX+404BD4]`  
00412281 8BD6 MOV `EDX, ESI`  
00412283 B9 10000000 MOV `ECX, 10`  
00412288 AC LODS `BYTE PTR DS:[ESI]`  
00412289 84C0 TEST `AL, AL`  
0041228B 74 06 JE SHORT `00412293`  
0041228D C04E FF 03 ROR `BYTE PTR DS:[ESI-1], 3`  
00412291 ^ E2 F5 LOOPD SHORT `00412288`  
00412293 E8 00000000 CALL `00412298`  
00412298 59 POP `ECX`  
00412299 81C1 1D000000 ADD `ECX, 1D`  
0041229F 52 PUSH `EDX`  
004122A0 51 PUSH `ECX`  
004122A1 C1E9 05 SHR `ECX, 5`  
004122A4 23D1 AND `EDX, ECX`  
004122A6 FFA5 F54B4000 JMP `DWORD PTR SS:[EBP+404BF5]`  
004122AC 0BC0 OR `EAX, EAX`  
004122AE 0F85 3F0A0000 JNZ `00412CF3`  
004122B4 A3 8D8D534C MOV `DWORD PTR DS:[4C538D8D], EAX`  
004122B9 40 INC `EAX`  
004122BA 0051 50 ADD `BYTE PTR DS:[ECX+50], DL`  
004122BD 8D85 19F54500 LEA `EAX, DWORD PTR SS:[EBP+45F519]`  
004122C3 2D 70A80500 SUB `EAX, 5A870`  
004122C8 FFD0 CALL `EAX`  
004122CA 0BC0 OR `EAX, EAX`  
004122CC 0F84 D41B0000 JE `00413EA6`  
004122D2 8DBD AB454000 LEA `EDI, DWORD PTR SS:[EBP+4045AB]`  
004122D8 2BC9 SUB `ECX, ECX`  
004122DA 2BC0 SUB `EAX, EAX`  
004122DC B0 23 MOV `AL, 23`  
004122DE 41 INC `ECX`  
004122DF 32C1 XOR `AL, CL`  
004122E1 48 DEC `EAX`  
004122E2 284439 FF SUB `BYTE PTR DS:[ECX+EDI-1], AL`  
004122E6 81F9 F4030000 CMP `ECX, 3F4`  
004122EC ^ 75 F0 JNZ SHORT `004122DE`  
004122EE 8D85 6A894000 LEA `EAX, DWORD PTR SS:[EBP+40896A]`  
004122F4 05 5EBDFFFF ADD `EAX, FFFFBD5E`  
004122F9 FFD0 CALL `EAX` ; 这里进去就是显示密码框的代码，注意，壳不会直接比较密码的  
004122FB EB 01 JMP SHORT `004122FE`  

……

00414776 68 A0050000 PUSH `5A0`  
0041477B 59 POP `ECX` ; push size `5a0`  
0041477C 8DBD 8B304000 LEA `EDI, DWORD PTR SS:[EBP+40308B]`  
00414782 81EF 2A010000 SUB `EDI, 12A`  
00414788 D1EB SHR `EBX, 1`  



```
0041478A    73 06            JNB SHORT 00414792  

0041478C    81F3 3488328C    XOR EBX,8C328834  

00414792    301F             XOR BYTE PTR DS:[EDI],BL                 ; 从 41235c 开始向下解压,SIZE:5A0  

00414794    47               INC EDI  

00414795    49               DEC ECX  

00414796    9C               PUSHFD  

00414797    C12C24 06        SHR DWORD PTR SS:[ESP],6  

0041479B    F71424           NOT DWORD PTR SS:[ESP]  

0041479E    832424 01        AND DWORD PTR SS:[ESP],1  

004147A2    50               PUSH EAX  

004147A3    52               PUSH EDX  

004147A4    B8 77B2DC10      MOV EAX,10DCB277  

004147A9    05 444D23EF      ADD EAX,EF234D44  

004147AE    F76424 08        MUL DWORD PTR SS:[ESP+8]  

004147B2    8D8428 D2534000 LEA EAX,DWORD PTR DS:[EAX+EBP+4053D2]  

004147B9    894424 08        MOV DWORD PTR SS:[ESP+8],EAX           ; PESpin.004147CD  

004147BD    5A               POP EDX  

004147BE    58               POP EAX  

004147BF    8D6424 04        LEA ESP,DWORD PTR SS:[ESP+4]  

004147C3    FF6424 FC        JMP DWORD PTR SS:[ESP-4]  

……  

004123D9    68 FF000000      PUSH 0FF                               ; /BufSize = FF (255.)  

004123DE    56               PUSH ESI                                 ; |PathBuffer = PESpin.00412000  

004123DF    6A 00            PUSH 0                                   ; |hModule = NULL  

004123E1    53               PUSH EBX                                 ; |Return address  

004123E2    FFA5 4A4C4000    JMP DWORD PTR SS:[EBP+404C4A]            ; \GetModuleFileNameA  

……  

004123F6    6A 00            PUSH 0                                   ; /hTemplateFile = NULL  

004123F8    68 80000000      PUSH 80                                  ; |Attributes = NORMAL  

004123FD    6A 03            PUSH 3                                   ; |Mode = OPEN_EXISTING  

004123FF    6A 00            PUSH 0                                   ; |pSecurity = NULL  

00412401    6A 01            PUSH 1                                   ; |ShareMode = FILE_SHARE_READ  

00412403    68 00000080      PUSH 80000000                            ; |Access = GENERIC_READ  

00412408    56               PUSH ESI                                 ; |FileName  

00412409    53               PUSH EBX                                 ; |Return address  

0041240A    FFA5 184C4000    JMP DWORD PTR SS:[EBP+404C18]            ; \CreateFileA  

……  

00412413    E8 01000000      CALL 00412419  

00412418    90               NOP  

00412419    5A               POP EDX  

0041241A    81C2 1A000000    ADD EDX,1A  

00412420    8985 8F5E4000    MOV DWORD PTR SS:[EBP+405E8F],EAX  

00412426    93               XCHG EAX,EBX  

00412427    6A 00            PUSH 0                                   ; /pFileSizeHigh = NULL  

00412429    53               PUSH EBX                                 ; |hFile = 00000040 (window)  

0041242A    52               PUSH EDX                                 ; |Return Address  

0041242B    FFA5 454C4000    JMP DWORD PTR SS:[EBP+404C45]            ; \GetFileSize  

00412431    90               NOP  

00412432    E8 01000000      CALL 00412438  

00412437    90               NOP  

00412438    5A               POP EDX  

00412439    81C2 24000000    ADD EDX,24  

0041243F    8BD8             MOV EBX,EAX  

00412441    53               PUSH EBX  

00412442    8F85 9B5E4000    POP DWORD PTR SS:[EBP+405E9B]  

00412448    6A 04            PUSH 4                                   ; /Protect = PAGE_READWRITE  

0041244A    68 00300000      PUSH 3000                                ; |AllocationType = MEM_COMMIT|MEM_RESERVE  

0041244F    50               PUSH EAX                                 ; |Size = D400 (54272.)  

00412450    6A 00            PUSH 0                                   ; |Address = NULL  

00412452    52               PUSH EDX                                 ; |Return address  

00412453    FFA5 0E4C4000    JMP DWORD PTR SS:[EBP+404C0E]            ; \VirtualAlloc  

00412459    90               NOP  

0041245A    90               NOP  

0041245B    50               PUSH EAX  

0041245C    8F85 C94B4000    POP DWORD PTR SS:[EBP+404BC9]           ; [EBP+404BC9]=[413FC4]保存 hmem  

00412462    8D8D 9B5E4000    LEA ECX,DWORD PTR SS:[EBP+405E9B]  

00412468    E8 01000000      CALL 0041246E  

0041246D    90               NOP  

0041246E    5A               POP EDX  

0041246F    81C2 1E000000    ADD EDX,1E  

00412475    6A 00            PUSH 0                                   ; /pOverlapped = NULL  

00412477    51               PUSH ECX                                 ; |pBytesRead = PESpin.00415296  

00412478    53               PUSH EBX                                 ; |BytesToRead = D400 (54272.)  

00412479    50               PUSH EAX                                 ; |Buffer = 003D0000  

0041247A    FFB5 8F5E4000    PUSH DWORD PTR SS:[EBP+405E8F]          ; |hFile = 00000040 (window)  

00412480    52               PUSH EDX                                 ; |Return Address  

00412481    FFA5 1D4C4000    JMP DWORD PTR SS:[EBP+404C1D]            ; \ReadFile  

00412487    90               NOP  

00412488    90               NOP  

00412489    90               NOP  

0041248A    90               NOP  

0041248B    E8 01000000      CALL 00412491  

00412490    90               NOP  

00412491    5A               POP EDX  

00412492    81C2 17000000    ADD EDX,17  

00412498    FFB5 8F5E4000    PUSH DWORD PTR SS:[EBP+405E8F]          ; /hObject = 00000040 (window)  

0041249E    52               PUSH EDX                                 ; |Return address  

0041249F    FFA5 094C4000    JMP DWORD PTR SS:[EBP+404C09]            ; \CloseHandle  

004124A5    90               NOP  

004124A6    90               NOP  

……  

004124E4    FFD0             CALL EAX                                 ; 计算 CRC 的值  

004124E6    2985 A35E4000    SUB DWORD PTR SS:[EBP+405EA3],EAX       ; [EBP+405EA3]=[0041529E]  

004124EC    E8 01000000      CALL 004124F2  

004124F1    90               NOP  

004124F2    5A               POP EDX  

004124F3    81C2 1E000000    ADD EDX,1E  

004124F9    68 00800000      PUSH 8000                                ; /FreeType = MEM_RELEASE  

004124FE    6A 00            PUSH 0                                   ; |Size = 0  

00412500    FFB5 C94B4000    PUSH DWORD PTR SS:[EBP+404BC9]          ; |Address = 003D0000  

00412506    52               PUSH EDX                                 ; |Return address  

00412507    FFA5 134C4000    JMP DWORD PTR SS:[EBP+404C13]            ; \VirtualFree  

……  

004125BF    0FB78D C74B4000 MOVZX ECX,WORD PTR SS:[EBP+404BC7]  

004125C6    8B95 CD4B4000    MOV EDX,DWORD PTR SS:[EBP+404BCD]  

004125CC    81C2 F8000000    ADD EDX,0F8  

004125D2    8B9D 935E4000    MOV EBX,DWORD PTR SS:[EBP+405E93]  

004125D8    33C0             XOR EAX,EAX  

004125DA    90               NOP  

004125DB    90               NOP  

004125DC    90               NOP  

004125DD    90               NOP  

004125DE    90               NOP  

004125DF    90               NOP  

004125E0    90               NOP  

004125E1    90               NOP  

004125E2    90               NOP  

004125E3    90               NOP  

004125E4    90               NOP  

004125E5    90               NOP  

004125E6    90               NOP  

004125E7    90               NOP  

004125E8    90               NOP  

004125E9    90               NOP  

004125EA    90               NOP  

004125EB    51               PUSH ECX  

004125EC    0FA3C3           BT EBX,EAX  

004125EF    73 67            JNB SHORT 00412658  

004125F1    52               PUSH EDX  

004125F2    90               NOP  

004125F3    90               NOP  

004125F4    90               NOP  

004125F5    90               NOP  

004125F6    90               NOP  

004125F7    90               NOP  

004125F8    90               NOP  

004125F9    90               NOP  

004125FA    90               NOP  

004125FB    90               NOP  
```



```
004125FC    90              NOP  

004125FD    90              NOP  

004125FE    90              NOP  

004125FF    90              NOP  

00412600    90              NOP  

00412601    90              NOP  

00412602    90              NOP  

00412603    8B7A 0C         MOV EDI,DWORD PTR DS:[EDX+C]  

00412606    03BD C34B4000   ADD EDI,DWORD PTR SS:[EBP+404BC3]  

0041260C    8B4A 10         MOV ECX,DWORD PTR DS:[EDX+10]  

0041260F    8B95 A35E4000   MOV EDX,DWORD PTR SS:[EBP+405EA3]  

00412615    D1EA            SHR EDX,1  

00412617    72 06           JB SHORT 0041261F  

00412619    81F2 31AF43ED   XOR EDX,ED43AF31  

0041261F    3017            XOR BYTE PTR DS:[EDI],DL          ; 循环还原各区段  

00412621    47              INC EDI  

00412622    90              NOP  

00412623    90              NOP  

00412624    90              NOP  

00412625    90              NOP  

00412626    90              NOP  

00412627    90              NOP  

00412628    90              NOP  

00412629    90              NOP  

0041262A    90              NOP  

0041262B    90              NOP  

0041262C    90              NOP  

0041262D    90              NOP  

0041262E    90              NOP  

0041262F    90              NOP  

00412630    90              NOP  

00412631    90              NOP  

00412632    90              NOP  

00412633    90              NOP  

00412634    90              NOP  

00412635    90              NOP  

00412636    90              NOP  

00412637    90              NOP  

00412638    90              NOP  

00412639    90              NOP  

0041263A    90              NOP  

0041263B    90              NOP  

0041263C    90              NOP  

0041263D    90              NOP  

0041263E    90              NOP  

0041263F    90              NOP  

00412640    90              NOP  

00412641    90              NOP  

00412642    90              NOP  

00412643    90              NOP  

00412644    90              NOP  

00412645    90              NOP  

00412646    90              NOP  

00412647    90              NOP  

00412648    90              NOP  

00412649    90              NOP  

0041264A    90              NOP  

0041264B    90              NOP  

0041264C    90              NOP  

0041264D    90              NOP  

0041264E    90              NOP  

0041264F    90              NOP  

00412650    90              NOP  

00412651    90              NOP  

00412652    90              NOP  

00412653    90              NOP  

00412654    49              DEC ECX  

00412655  ^ 75 BE           JNZ SHORT 00412615  

00412657    5A              POP EDX  

00412658    40              INC EAX  

00412659    83C2 28         ADD EDX,28  

0041265C    59              POP ECX  

0041265D    90              NOP  

0041265E    90              NOP  

0041265F    90              NOP  

00412660    90              NOP  

00412661    90              NOP  

00412662    90              NOP  

00412663    90              NOP  

00412664    90              NOP  

00412665    90              NOP  

00412666    90              NOP  

00412667    90              NOP  

00412668    90              NOP  

00412669    90              NOP  

0041266A    90              NOP  

0041266B    90              NOP  

0041266C    90              NOP  

0041266D    90              NOP  

0041266E    49              DEC ECX  

0041266F    9C              PUSHFD  

00412670    C12C24 06       SHR DWORD PTR SS:[ESP],6  

00412674    F71424          NOT DWORD PTR SS:[ESP]  

00412677    832424 01       AND DWORD PTR SS:[ESP],1  

0041267B    50              PUSH EAX  

0041267C    52              PUSH EDX  

0041267D    B8 04B2DC12     MOV EAX,12DCB204  

00412682    05 444D23ED     ADD EAX,ED234D44  

00412687    F76424 08       MUL DWORD PTR SS:[ESP+8]  

0041268B    8D8428 A8324000 LEA EAX,DWORD PTR DS:[EAX+EBP+4032A8]  

00412692    894424 08       MOV DWORD PTR SS:[ESP+8],EAX  

00412696    5A              POP EDX  

00412697    58              POP EAX  

00412698    8D6424 04       LEA ESP,DWORD PTR SS:[ESP+4]  

0041269C    FF6424 FC       JMP DWORD PTR SS:[ESP-4]         ; 没有解压完则继续跳回去  

……  

004126B4    838D 9D5D4000 0>OR DWORD PTR SS:[EBP+405D9D],0    ; 测试是否 anti-debug  

004126BB    74 0D           JE SHORT 004126CA                ; 如果没有选择 anti-degub 则跳下一步,主程序没有设置 anti debug  

004126BD    8D85 C8554000   LEA EAX,DWORD PTR SS:[EBP+4055C8] ; CreateFileA 方式测试 sice  

004126C3    2D D1030000     SUB EAX,3D1  

004126C8    FFD0            CALL EAX  

004126CA    68 80010000     PUSH 180  

004126CF    59              POP ECX  

……  

00412703    E8 01000000     CALL 00412709  

00412708    90              NOP  

00412709    D1EA            SHR EDX,1  

0041270B    73 06           JNB SHORT 00412713  

0041270D    81F2 32AF43ED   XOR EDX,ED43AF32  

00412713    3017            XOR BYTE PTR DS:[EDI],DL  

00412715    47              INC EDI  

00412716    49              DEC ECX  

00412717    9C              PUSHFD  

00412718    C12C24 06       SHR DWORD PTR SS:[ESP],6  

0041271C    F71424          NOT DWORD PTR SS:[ESP]  

0041271F    832424 01       AND DWORD PTR SS:[ESP],1  

00412723    50              PUSH EAX  

00412724    52              PUSH EDX  

00412725    B8 CEBFABF2     MOV EAX,F2ABBFCE  

0041272A    05 EB3F540D     ADD EAX,0D543FEB  

0041272F    F76424 08       MUL DWORD PTR SS:[ESP+8]  

00412733    8D8428 4F334000 LEA EAX,DWORD PTR DS:[EAX+EBP+40334F]  

0041273A    894424 08       MOV DWORD PTR SS:[ESP+8],EAX  

0041273E    5A              POP EDX  

0041273F    58              POP EAX  

00412740    8D6424 04       LEA ESP,DWORD PTR SS:[ESP+4]  

00412744    FF6424 FC       JMP DWORD PTR SS:[ESP-4]         ; 从 41495a 处开始向下解压,大小为 180  

……  

00412757    2BC3            SUB EAX,EBX  

00412759    50              PUSH EAX                         ; 解压完去执行解压后的代码  

0041275A    C3              RETN  

……  

0041495A   /EB 01          JMP SHORT 0041495D  

0041495C   |90             NOP  

0041495D   \8DBD 60334000  LEA EDI,DWORD PTR SS:[EBP+403360] ; 0041275B  

00414963    B9 A1010000     MOV ECX,1A1                      ; 从 41275b 处开始向下解压代码，大小为 1A1  

00414968    90              NOP  

00414969    90              NOP  

0041496A    90              NOP  

0041496B    90              NOP  

0041496C    90              NOP  

0041496D    90              NOP  

0041496E    90              NOP  

0041496F    90              NOP  

00414970    90              NOP  

00414971    8A07            MOV AL,BYTE PTR DS:[EDI]  

00414973    02C1            ADD AL,CL  

00414975    C0C8 1E         ROR AL,1E  

00414978    F9              STC  

00414979    90              NOP  

0041497A    F9              STC  

0041497B    02C1            ADD AL,CL  

0041497D    EB 01           JMP SHORT 00414980  

0041497F    90              NOP  

00414980    02C1            ADD AL,CL  

00414982    C0C0 93         ROL AL,93                        ; Shift constant out of range 1..31  

00414985    EB 01           JMP SHORT 00414988  

00414987    90              NOP  

00414988    EB 01           JMP SHORT 0041498B  

0041498A    90              NOP  

0041498B    EB 01           JMP SHORT 0041498E  

0041498D    90              NOP  

0041498E    EB 01           JMP SHORT 00414991  

00414990    90              NOP  

00414991    32C1            XOR AL,CL  

00414993    2C 57           SUB AL,57  

00414995    02C1            ADD AL,CL  

00414997    AA              STOS BYTE PTR ES:[EDI]  

00414998    49              DEC ECX  

00414999    9C              PUSHFD  

0041499A    C12C24 06       SHR DWORD PTR SS:[ESP],6  

0041499E    F71424          NOT DWORD PTR SS:[ESP]  

004149A1    832424 01       AND DWORD PTR SS:[ESP],1  

004149A5    50              PUSH EAX  

004149A6    52              PUSH EDX
```



```
004149A7    B8 5EBFDC32     MOV EAX,32DCBF5E  
004149AC    05 444023CD     ADD EAX,CD234044  
004149B1    F76424 08       MUL DWORD PTR SS:\[ESP+8\]  
004149B5    8D8428 D4554000 LEA EAX,DWORD PTR DS:\[EAX+EBP+4055D4\]  
004149BC >   894424 08       MOV DWORD PTR SS:\[ESP+8\],EAX            ; PESpin.004149CF  
004149C0    5A              POP EDX  
004149C1    58              POP EAX  
004149C2    8D6424 04       LEA ESP,DWORD PTR SS:\[ESP+4\]  
004149C6    FF6424 FC       JMP DWORD PTR SS:\[ESP\-4\]  
……  
004149CF    55              PUSH EBP  
004149D0    9C              PUSHFD  
004149D1    E8 77000000     CALL 00414A4D                          ; 这里进去就是 SEH 异常  
……  
004149D7    8B5424 08       MOV EDX,DWORD PTR SS:\[ESP+8\]  
004149DB    8B4424 0C       MOV EAX,DWORD PTR SS:\[ESP+C\]  
004149DF    8142 04 3500000>ADD DWORD PTR DS:\[EDX+4\],35  
004149E6    81CA 29242123   OR EDX,23212429  
004149EC    2BC9            SUB ECX,ECX  
004149EE    2148 04         AND DWORD PTR DS:\[EAX+4\],ECX            ; 清除硬件断点  
004149F1    2148 08         AND DWORD PTR DS:\[EAX+8\],ECX  
004149F4    2148 0C         AND DWORD PTR DS:\[EAX+C\],ECX  
004149F7    2148 10         AND DWORD PTR DS:\[EAX+10\],ECX  
004149FA    8160 14 F00FFFF>AND DWORD PTR DS:\[EAX+14\],FFFF0FF0  
00414A01    C740 18 5501000>MOV DWORD PTR DS:\[EAX+18\],155  
00414A08    33C0            XOR EAX,EAX  
00414A0A    C3              RETN  
……  
00414A65    8DBD 01354000   LEA EDI,DWORD PTR SS:\[EBP+403501\]      ; 从 004128FC 开始解压代码,大小为 108f  
00414A6B    B9 8F100000     MOV ECX,108F  
00414A70    90              NOP  
00414A71    90              NOP  
00414A72    90              NOP  
00414A73    90              NOP  
00414A74    90              NOP  
00414A75    90              NOP  
00414A76    90              NOP  
00414A77    90              NOP  
00414A78    90              NOP  
00414A79    8A07            MOV AL,BYTE PTR DS:\[EDI\]  
00414A7B    02C1            ADD AL,CL  
00414A7D    C0C0 43         ROL AL,43                               ; Shift constant out of range 1..31  
00414A80    FEC8            DEC AL  
00414A82    04 40           ADD AL,40  
00414A84    2C 39           SUB AL,39  
00414A86    EB 01           JMP SHORT 00414A89  
00414A88    90              NOP  
00414A89    34 BB           XOR AL,0BB  
00414A8B    0AC0            OR AL,AL  
00414A8D    04 85           ADD AL,85  
00414A8F    EB 01           JMP SHORT 00414A92  
00414A91    90              NOP  
00414A92    02C1            ADD AL,CL  
00414A94    90              NOP  
00414A95    F9              STC  
00414A96    C0C8 53         ROR AL,53                               ; Shift constant out of range 1..31  
00414A99    0AC0            OR AL,AL  
00414A9B    04 C2           ADD AL,0C2  
00414A9D    2AC1            SUB AL,CL  
00414A9F    AA              STOS BYTE PTR ES:\[EDI\]  
00414AA0    49              DEC ECX  
00414AA1    9C              PUSHFD  
00414AA2    C12C24 06       SHR DWORD PTR SS:\[ESP\],6  
00414AA6    F71424          NOT DWORD PTR SS:\[ESP\]  
00414AA9    832424 01       AND DWORD PTR SS:\[ESP\],1  
00414AAD    50              PUSH EAX  
00414AAE    52              PUSH EDX  
00414AAF    B8 61B2DC12     MOV EAX,12DCB261  
00414AB4    05 444D23ED     ADD EAX,ED234D44  
00414AB9    F76424 08       MUL DWORD PTR SS:\[ESP+8\]  
00414ABD    8D8428 D9564000 LEA EAX,DWORD PTR DS:\[EAX+EBP+4056D9\]  
00414AC4    894424 08       MOV DWORD PTR SS:\[ESP+8\],EAX            ; PESpin.00414AD4  
00414AC8    5A              POP EDX  
00414AC9    58              POP EAX  
00414ACA    8D6424 04       LEA ESP,DWORD PTR SS:\[ESP+4\]  
00414ACE    FF6424 FC       JMP DWORD PTR SS:\[ESP\-4\]             ; 如果没有解压完则继续  
……  
00412777    68 07000000     PUSH 7  
0041277C    5B              POP EBX  
0041277D    25 25382C37     AND EAX,372C3825  
00412782    50              PUSH EAX  
00412783    8D6424 04       LEA ESP,DWORD PTR SS:\[ESP+4\]  
00412787    F7D0            NOT EAX  
00412789    234424 FC       AND EAX,DWORD PTR SS:\[ESP\-4\]  
0041278D    51              PUSH ECX                                 ; 从这里开始解密各段  
0041278E    90              NOP  
0041278F    90              NOP  
00412790    90              NOP  
00412791    90              NOP  
00412792    90              NOP  
00412793    90              NOP  
00412794    90              NOP  
00412795    90              NOP  
00412796    90              NOP  
00412797    90              NOP  
00412798    90              NOP  
00412799    90              NOP  
0041279A    0FA3C3          BT EBX,EAX  
0041279D    73 79           JNB SHORT 00412818                      ; 如果该段解压完则跳去解压下一段  
0041279F    90              NOP  
004127A0    90              NOP  
004127A1    90              NOP  
004127A2    90              NOP  
004127A3    90              NOP  
004127A4    90              NOP  
004127A5    90              NOP  
004127A6    90              NOP  
004127A7    90              NOP  
004127A8    90              NOP  
004127A9    90              NOP  
004127AA    90              NOP  
004127AB    90              NOP  
004127AC    90              NOP  
004127AD    90              NOP  
004127AE    90              NOP  
004127AF    90              NOP  
004127B0    90              NOP  
004127B1    90              NOP  
004127B2    90              NOP  
004127B3    90              NOP  
004127B4    90              NOP  
004127B5    90              NOP  
004127B6    90              NOP  
004127B7    90              NOP  
004127B8    90              NOP  
004127B9    90              NOP  
004127BA    90              NOP  
004127BB    90              NOP  
004127BC    90              NOP  
004127BD    90              NOP  
004127BE    90              NOP  
004127BF    90              NOP  
004127C0    90              NOP  
004127C1    90              NOP  
004127C2    90              NOP  
004127C3    90              NOP  
004127C4    90              NOP  
004127C5    90              NOP  
004127C6    90              NOP  
004127C7    90              NOP  
004127C8    90              NOP  
004127C9    90              NOP  
004127CA    90              NOP  
004127CB    90              NOP  
004127CC    90              NOP  
004127CD    8B7A 0C         MOV EDI,DWORD PTR DS:\[EDX+C\]  
004127D0    03BD C34B4000   ADD EDI,DWORD PTR SS:\[EBP+404BC3\]  
004127D6    8B4A 10         MOV ECX,DWORD PTR DS:\[EDX+10\]          ; RSIZE = 6000  
004127D9    50              PUSH EAX  
004127DA    8A07            MOV AL,BYTE PTR DS:\[EDI\]              ; 第一次 从 401000 处开始解密代码，size:6000  
004127DC    2C 61           SUB AL,61  
004127DE    F8              CLC  
004127DF    F8              CLC  
004127E0    C0C0 B1         ROL AL,0B1                               ; Shift constant out of range 1..31  
004127E3    34 AF           XOR AL,0AF  
004127E5    04 70           ADD AL,70  
004127E7    FEC8            DEC AL  
004127E9    EB 01           JMP SHORT 004127EC  
004127EB    90              NOP  
004127EC    F8              CLC  
004127ED    32C1            XOR AL,CL  
004127EF    C0C0 42         ROL AL,42                               ; Shift constant out of range 1..31  
004127F2    EB 01           JMP SHORT 004127F5  
004127F4    90              NOP  
004127F5    02C1            ADD AL,CL  
004127F7    2AC1            SUB AL,CL  
004127F9    34 04           XOR AL,4  
004127FB    C0C0 9B         ROL AL,9B                               ; Shift constant out of range 1..31  
004127FE    FEC8            DEC AL  
00412800    AA              STOS BYTE PTR ES:\[EDI\]  
00412801    49              DEC ECX  
00412802    90              NOP  
00412803    90              NOP  
00412804    90              NOP  
00412805    90              NOP  
00412806    90              NOP  
```



```
00412807    90              NOP
00412808    90              NOP
00412809    90              NOP
0041280A    90              NOP
0041280B    90              NOP
0041280C    90              NOP
0041280D    90              NOP
0041280E    90              NOP
0041280F    90              NOP
00412810    90              NOP
00412811    90              NOP
00412812    90              NOP
00412813    0BC9            OR ECX,ECX
00412815  ^ 75 C3          JNZ SHORT 004127DA            ; 该段没解压完该段则继续上去解密
00412817    58              POP EAX
00412818    40              INC EAX
00412819    83C2 28         ADD EDX,28
0041281C    90              NOP
0041281D    90              NOP
0041281E    90              NOP
0041281F    90              NOP
00412820    90              NOP
00412821    90              NOP
00412822    90              NOP
00412823    90              NOP
00412824    90              NOP
00412825    59              POP ECX
00412826    49              DEC ECX
00412827    9C              PUSHFD
00412828    C12C24 06       SHR DWORD PTR SS:[ESP],6
0041282C    F71424          NOT DWORD PTR SS:[ESP]
0041282F    832424 01       AND DWORD PTR SS:[ESP],1
00412833    50              PUSH EAX
00412834    52              PUSH EDX
00412835    B8 E979A6F5     MOV EAX,F5A679E9
0041283A    05 4985590A     ADD EAX,0A598549
0041283F    F76424 08       MUL DWORD PTR SS:[ESP+8]
00412843    8D8428 60344000 LEA EAX,DWORD PTR DS:[EAX+EBP+403460]
0041284A    894424 08       MOV DWORD PTR SS:[ESP+8],EAX
0041284E    5A              POP EDX
0041284F    58              POP EAX
00412850    8D6424 04       LEA ESP,DWORD PTR SS:[ESP+4]
00412854    FF6424 FC       JMP DWORD PTR SS:[ESP-4]           ; 没有解压完则继续回去解密

……

0041286B    E8 BA1C0000     CALL 0041452A                       ; 这个 CALL 实际就是一个异常 CALL

……

00415062    6A 04           PUSH 4                              ; /Protect = PAGE_READWRITE
00415064    68 00300000     PUSH 3000                           ; |AllocationType = MEM_COMMIT|MEM_RESERVE
00415069    51              PUSH ECX                            ; |Size = 3166 (12646.)
0041506A    6A 00           PUSH 0                              ; |Address = NULL
0041506C    FF95 0E4C4000   CALL DWORD PTR SS:[EBP+404C0E]          ; \\VirtualAlloc
00415072    96              XCHG EAX,ESI                      ; hmem==003D0000
00415073    5A              POP EDX
00415074    BF 50F40000     MOV EDI,0F450
00415079    81C7 00004000   ADD EDI,00400000
0041507F    56              PUSH ESI                            ; /存放地址 == 003D0000
00415080    57              PUSH EDI                            ; |解压地址 == 40f450
00415081    E8 1CDEFFFF     CALL 00412EA2                       ; \\aplib_depack
00415086    91              XCHG EAX,ECX
00415087    F3:A4           REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]
00415089    5F              POP EDI
0041508A    5E              POP ESI
0041508B    EB 01           JMP SHORT 0041508E
0041508D    90              NOP
0041508E    68 00400000     PUSH 4000                           ; /FreeType = MEM_DECOMMIT
00415093    52              PUSH EDX                            ; |Size = 3166 (12646.)
00415094    56              PUSH ESI                            ; |Address = 003D0000
00415095    FF95 134C4000   CALL DWORD PTR SS:[EBP+404C13]          ; \\VirtualFree

……

004150A7    8D85 ED5C4000   LEA EAX,DWORD PTR SS:[EBP+405CED]
004150AD    8338 00         CMP DWORD PTR DS:[EAX],0
004150B0    0F84 CB000000   JE 00415181
004150B6    B9 80B60000     MOV ECX,0B680
004150BB    6A 04           PUSH 4                              ; /Protect = PAGE_READWRITE
004150BD    68 00300000     PUSH 3000                           ; |AllocationType = MEM_COMMIT|MEM_RESERVE
004150C2    51              PUSH ECX                            ; |Size = B680 (46720.)
004150C3    6A 00           PUSH 0                              ; |Address = NULL
004150C5    FF95 0E4C4000   CALL DWORD PTR SS:[EBP+404C0E]          ; \\VirtualAlloc
004150CB    8985 0E5D4000   MOV DWORD PTR SS:[EBP+405D0E],EAX      ; [EBP+405D0E]==[00415109]
004150D1    EB 01           JMP SHORT 004150D4
004150D3    90              NOP
004150D4    0FB78D C74B4000 MOVZX ECX,WORD PTR SS:[EBP+404BC7]      ; ecx==4
004150DB    8B95 CD4B4000   MOV EDX,DWORD PTR SS:[EBP+404BCD]
004150E1    81C2 F8000000   ADD EDX,0F8
004150E7    BB 07000000     MOV EBX,7
004150EC    2BC0            SUB EAX,EAX
004150EE    51              PUSH ECX
004150EF    90              NOP
004150F0    90              NOP
004150F1    90              NOP
004150F2    90              NOP
004150F3    90              NOP
004150F4    90              NOP
004150F5    90              NOP
004150F6    90              NOP
004150F7    90              NOP
004150F8    0FA3C3          BT EBX,EAX
004150FB    73 27           JNB SHORT 00415124                  ; 如果解压完该段则跳
004150FD    50              PUSH EAX
004150FE    53              PUSH EBX                            ; 铺张浪费^_^
004150FF    8B7A 0C         MOV EDI,DWORD PTR DS:[EDX+C]
00415102    03BD C34B4000   ADD EDI,DWORD PTR SS:[EBP+404BC3]      ; code 起始地址 401000
00415108    BE 00003F00     MOV ESI,3F0000
0041510D    56              PUSH ESI                            ; /临时存放位置 ==003F0000
0041510E    57              PUSH EDI                            ; |要解压的地址 == 401000
0041510F    E8 8EDDFFFF     CALL 00412EA2                       ; \\aplib_dePack
00415114    91              XCHG EAX,ECX
00415115    90              NOP
00415116    90              NOP
00415117    90              NOP
00415118    90              NOP
00415119    90              NOP
0041511A    90              NOP
0041511B    90              NOP
0041511C    90              NOP
0041511D    90              NOP
0041511E    F3:A4           REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]
00415120    5F              POP EDI
00415121    5E              POP ESI
00415122    5B              POP EBX
00415123    58              POP EAX
00415124    40              INC EAX
00415125    83C2 28         ADD EDX,28
00415128    59              POP ECX
00415129    49              DEC ECX
0041512A    9C              PUSHFD
0041512B    C12C24 06       SHR DWORD PTR SS:[ESP],6
0041512F    F71424          NOT DWORD PTR SS:[ESP]
00415132    832424 01       AND DWORD PTR SS:[ESP],1
00415136    50              PUSH EAX
00415137    52              PUSH EDX
00415138    B8 49B2DC12     MOV EAX,12DCB249
0041513D    05 444D23ED     ADD EAX,ED234D44
00415142    F76424 08       MUL DWORD PTR SS:[ESP+8]
00415146    8D8428 665D4000 LEA EAX,DWORD PTR DS:[EAX+EBP+405D66]
0041514D    894424 08       MOV DWORD PTR SS:[ESP+8],EAX
00415151    5A              POP EDX
00415152    58              POP EAX
00415153    8D6424 04       LEA ESP,DWORD PTR SS:[ESP+4]
00415157    FF6424 FC       JMP DWORD PTR SS:[ESP-4]            ; 循环 aplib 解压代码

……

00415164    8B8D BC5C4000   MOV ECX,DWORD PTR SS:[EBP+405CBC]      ; [EBP+405CBC]=[4150B7]=B680
0041516A    8B85 0E5D4000   MOV EAX,DWORD PTR SS:[EBP+405D0E]      ; [EBP+405D0E]=[415109]=3F0000
00415170    0BC0            OR EAX,EAX
00415172    74 0D           JE SHORT 00415181                   ; 如果已经释放了空间或申请空间失败则跳
```



## 反汇编代码片段

```asm
00415174    68 00400000     PUSH 4000                           ; /FreeType = MEM_DECOMMIT  
00415179    51             PUSH ECX                             ; |Size = B680 (46720.)  
0041517A    56             PUSH ESI                             ; |Address = 003F0000  
0041517B    FF95 134C4000  CALL DWORD PTR SS:[EBP+404C13]      ; \\VirtualFree  
00415181    EB 01          JMP SHORT 00415184  
```

这个壳比较会省，只申请一次空间通过擦除的方法循环解压各段

……

```asm
0041441D    51             PUSH ECX  
0041441E    8D85 8B5E4000  LEA EAX,DWORD PTR SS:[EBP+405E8B]  
00414424    50             PUSH EAX                             ; /pOldProtect = PESpin.00415286  
00414425    6A 40          PUSH 40                              ; |NewProtect = PAGE_EXECUTE_READWRITE  
00414427    51             PUSH ECX                             ; |Size = 25C (604.  
00414428    57             PUSH EDI                             ; |Address = PESpin.004001C8  
00414429    8DB5 F44B4000  LEA ESI,DWORD PTR SS:[EBP+404BF4]     ; |  
0041442F    FF56 10        CALL DWORD PTR DS:[ESI+10]           ; \\VirtualProtect  
00414432    59             POP ECX  
00414433    B0 FF          MOV AL,0FF  
```

……

```asm
004143F2    8D85 9C504000  LEA EAX,DWORD PTR SS:[EBP+40509C]  
004143F8    8785 7E504000  XCHG DWORD PTR SS:[EBP+40507E],EAX  
004143FE    8BBD C34B4000  MOV EDI,DWORD PTR SS:[EBP+404BC3]  
00414404    037F 3C        ADD EDI,DWORD PTR DS:[EDI+3C]  
00414407    89BD A8504000  MOV DWORD PTR SS:[EBP+4050A8],EDI  
0041440D    03F8           ADD EDI,EAX  
0041440F    B9 5C020000    MOV ECX,25C  
00414414    90             NOP  
00414415    90             NOP  
00414416    90             NOP  
00414417    90             NOP  
00414418    90             NOP  
00414419    90             NOP  
0041441A    90             NOP  
0041441B    90             NOP  
0041441C    90             NOP  
0041441D    51             PUSH ECX  
0041441E    8D85 8B5E4000  LEA EAX,DWORD PTR SS:[EBP+405E8B]  
00414424    50             PUSH EAX                             ; /pOldProtect = PESpin.00415286  
00414425    6A 40          PUSH 40                              ; |NewProtect = PAGE_EXECUTE_READWRITE  
00414427    51             PUSH ECX                             ; |Size = 25C (604.  
00414428    57             PUSH EDI                             ; |Address = PESpin.004001C8  
00414429    8DB5 F44B4000  LEA ESI,DWORD PTR SS:[EBP+404BF4]     ; |  
0041442F    FF56 10        CALL DWORD PTR DS:[ESI+10]           ; \\VirtualProtect  
00414432    59             POP ECX  
00414433    B0 FF          MOV AL,0FF  
00414435    90             NOP  
00414436    90             NOP  
00414437    90             NOP  
00414438    90             NOP  
00414439    90             NOP  
0041443A    90             NOP  
0041443B    90             NOP  
0041443C    90             NOP  
0041443D    90             NOP  
0041443E    90             NOP  
0041443F    90             NOP  
00414440    90             NOP  
00414441    8BF7           MOV ESI,EDI  
00414443    83C6 07        ADD ESI,7  
00414446    C607 BE        MOV BYTE PTR DS:[EDI],0BE            ; 开始修改 PE 头  
00414449    8977 01        MOV DWORD PTR DS:[EDI+1],ESI  
0041444C    C747 05 8F06000>MOV DWORD PTR DS:[EDI+5],68F  
00414453    83E9 03        SUB ECX,3  
00414456    8D1C0F         LEA EBX,DWORD PTR DS:[EDI+ECX]  
00414459    66:C703 33D2   MOV WORD PTR DS:[EBX],0D233  
0041445E    C643 02 C3     MOV BYTE PTR DS:[EBX+2],0C3  
00414462    53             PUSH EBX  
00414463    8F85 DD4B4000  POP DWORD PTR SS:[EBP+404BDD]  
00414469    2BDB           SUB EBX,EBX  
0041446B    90             NOP  
0041446C    90             NOP  
0041446D    90             NOP  
0041446E    90             NOP  
0041446F    90             NOP  
00414470    90             NOP  
00414471    90             NOP  
00414472    90             NOP  
00414473    90             NOP  
00414474    E8 04000000    CALL 0041447D  
00414479    97             XCHG EAX,EDI  
0041447A    44             INC ESP  
0041447B    41             INC ECX  
0041447C    90             NOP                                  ; ***这里不能看成垃圾指令而 nop 掉  
0041447D    5A             POP EDX                              ; 注意这上面一句不能 nop，否则 seh 就出问题了  
0041447E    8B12           MOV EDX,DWORD PTR DS:[EDX]  
00414480    55             PUSH EBP  
00414481    52             PUSH EDX  
00414482    64:FF33        PUSH DWORD PTR FS:[EBX]  
00414485    64:8923        MOV DWORD PTR FS:[EBX],ESP           ; install SEH  
00414488    68 F3AA9090    PUSH 9090AAF3  
0041448D    FFE7           JMP EDI                                ; 这里 jmp 去破坏 pe 头  
0041448F    64:8F02        POP DWORD PTR FS:[EDX]  
00414492    83C4 08        ADD ESP,8  
00414495    C3             RETN  
```

看看破坏方式：

```asm
004001C8    BE CF014000    MOV ESI,004001CF                       ; 把 pe 头部从 4001c8 开始全部填充成 FF，大小为 259,  
004001CD    8F06           POP DWORD PTR DS:[ESI]  
004001CF    F3:AA          REP STOS BYTE PTR ES:[EDI]  
004001D1    90             NOP  
004001D2    90             NOP  
```

解决方法就是在破坏 PE 头之前把 PE 头给 dump 下来.

……

```asm
004144CA    8D85 F44B4000  LEA EAX,DWORD PTR SS:[EBP+404BF4]  
004144D0    B9 2E000000    MOV ECX,2E  
004144D5    FF1401         CALL DWORD PTR DS:[ECX+EAX]             ; GetTickCount  
004144D8    8BD8           MOV EBX,EAX  
004144DA    F7D3           NOT EBX  
004144DC    33D8           XOR EBX,EAX  
004144DE    43             INC EBX  
004144DF    68 87000000    PUSH 87  
004144E4    59             POP ECX  
004144E5    66:35 4C50     XOR AX,504C  
004144E9    66:05 8911     ADD AX,1189  
004144ED    AA             STOS BYTE PTR ES:[EDI]                   ; 循环把 412000 处的代码给抹掉  
004144EE    EB 01          JMP SHORT 004144F1  
004144F0    90             NOP  
004144F1    49             DEC ECX  
004144F2    9C             PUSHFD  
004144F3    C12C24 06      SHR DWORD PTR SS:[ESP],6  
004144F7    F71424         NOT DWORD PTR SS:[ESP]  
004144FA    832424 01      AND DWORD PTR SS:[ESP],1  
004144FE    50             PUSH EAX  
004144FF    52             PUSH EDX  
00414500    B8 6FB2DC12    MOV EAX,12DCB26F  
00414505    05 4E4D23ED    ADD EAX,ED234D4E  
0041450A    F76424 08      MUL DWORD PTR SS:[ESP+8]  
0041450E    8D8428 2D514000 LEA EAX,DWORD PTR DS:[EAX+EBP+40512D]  
00414515    894424 08      MOV DWORD PTR SS:[ESP+8],EAX  
00414519    5A             POP EDX  
0041451A    58             POP EAX  
0041451B    8D6424 04      LEA ESP,DWORD PTR SS:[ESP+4]  
0041451F    FF6424 FC      JMP DWORD PTR SS:[ESP-4]  
```

……

```asm
00414BBC    6A 04          PUSH 4                                   ; /Protect = PAGE_READWRITE  
00414BBE    68 00300000    PUSH 3000                                ; |AllocationType = MEM_COMMIT|MEM_RESERVE  
00414BC3    51             PUSH ECX                                 ; |Size = 62 (98.)  
00414BC4    6A 00          PUSH 0                                   ; |Address = NULL  
00414BC6    53             PUSH EBX                                 ; |Return address  
00414BC7    FFA5 0E4C4000  JMP DWORD PTR SS:[EBP+404C0E]            ; \\VirtualAlloc  
00414BCD    90             NOP  
00414BCE    90             NOP  
00414BCF    90             NOP  
00414BD0    8DB5 19574000  LEA ESI,DWORD PTR SS:[EBP+405719]  
00414BD6    97             XCHG EAX,EDI  
00414BD7    8BDF           MOV EBX,EDI  
```



```
00414BD9    B9 2A000000     MOV ECX,2A  

00414BDE    F3:A4           REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]   ; 把从 414b14 开始的代码搬到刚申请的地址空间里,大小为 2a  

00414BE0    BE 759FE9D4     MOV ESI,D4E99F75  

00414BE5    BA B1B5572B     MOV EDX,2B57B5B1  

00414BEA    03F2            ADD ESI,EDX  

00414BEC    B9 0A000000     MOV ECX,0A                                                  ; 大小 0a  

00414BF1    BA 13E40E80     MOV EDX,800EE413  

00414BF6    AD              LODS DWORD PTR DS:[ESI]  

00414BF7    4A              DEC EDX  

00414BF8    03C2            ADD EAX,EDX  

00414BFA    42              INC EDX  

00414BFB    33C2            XOR EAX,EDX  

00414BFD    4A              DEC EDX  

00414BFE    C1CA 08         ROR EDX,8  

00414C01    AB              STOS DWORD PTR ES:[EDI]  

00414C02    49              DEC ECX  

00414C03    9C              PUSHFD  

00414C04    C12C24 06       SHR DWORD PTR SS:[ESP],6  

00414C08    F71424          NOT DWORD PTR SS:[ESP]  

00414C0B    832424 01       AND DWORD PTR SS:[ESP],1  

00414C0F    50              PUSH EAX  

00414C10    52              PUSH EDX  

00414C11    B8 817A6FF2     MOV EAX,F26F7A81  

00414C16    05 4085900D     ADD EAX,0D908540  

00414C1B    F76424 08       MUL DWORD PTR SS:[ESP+8]  

00414C1F    8D8428 3A584000 LEA EAX,DWORD PTR DS:[EAX+EBP+40583A]  

00414C26    894424 08       MOV DWORD PTR SS:[ESP+8],EAX  

00414C2A    5A              POP EDX  

00414C2B    58              POP EAX  

00414C2C    8D6424 04       LEA ESP,DWORD PTR SS:[ESP+4]  

00414C30    FF6424 FC       JMP DWORD PTR SS:[ESP-4]                                      ; Loop  

……  

00414C35    B9 10000000     MOV ECX,10  

00414C3A    8DB5 43574000   LEA ESI,DWORD PTR SS:[EBP+405743]  

00414C40    F3:A4           REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]   ; From: 414b3e to: d00052 size: 10  

00414C42    90              NOP  

00414C43    90              NOP  

00414C44    90              NOP  

00414C45    90              NOP  

00414C46    90              NOP  

00414C47    90              NOP  

00414C48    90              NOP  

00414C49    90              NOP  

00414C4A    90              NOP  

00414C4B    90              NOP  

00414C4C    90              NOP  

00414C4D    90              NOP  

00414C4E    93              XCHG EAX,EBX  

00414C4F    B9 0A000000     MOV ECX,0A                                                  ; size  

00414C54    8BBD E6574000   MOV EDI,DWORD PTR SS:[EBP+4057E6]  

00414C5A    03BD EB574000   ADD EDI,DWORD PTR SS:[EBP+4057EB]  

00414C60    F3:AB           REP STOS DWORD PTR ES:[EDI]                                 ; 填充刚申请的地址 d00000  

00414C62    E8 01000000     CALL 00414C68  

00414C67    90              NOP  

00414C68    5B              POP EBX  

00414C69    81C3 21000000   ADD EBX,21  

00414C6F    B9 61000000     MOV ECX,61  

00414C74    6A 04           PUSH 4                                                      ; /Protect = PAGE_READWRITE  

00414C76    68 00300000     PUSH 3000                                                   ; |AllocationType = MEM_COMMIT|MEM_RESERVE  

00414C7B    51              PUSH ECX                                                    ; |Size = 61 (97.)  

00414C7C    6A 00           PUSH 0                                                      ; |Address = NULL  

00414C7E    53              PUSH EBX                                                    ; |Return address  

00414C7F    FFA5 0E4C4000   JMP DWORD PTR SS:[EBP+404C0E]                               ; \VirtualAlloc  

00414C85    90              NOP  

00414C86    90              NOP  

00414C87    90              NOP  

00414C88    8DB5 DF564000   LEA ESI,DWORD PTR SS:[EBP+4056DF]  

00414C8E    97              XCHG EAX,EDI  

00414C8F    8BDF            MOV EBX,EDI  

00414C91    B9 26000000     MOV ECX,26  

00414C96    F3:A4           REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]   ; from 4a4ada to: D10000 size:26  

……  

00414CA4    8BB5 E6574000   MOV ESI,DWORD PTR SS:[EBP+4057E6]  

00414CAA    03B5 EB574000   ADD ESI,DWORD PTR SS:[EBP+4057EB]  

00414CB0    83C6 28         ADD ESI,28  

00414CB3    B9 0A000000     MOV ECX,0A                                                  ; size  

00414CB8    BA A4919C0B     MOV EDX,0B9C91A4  

00414CBD    AD              LODS DWORD PTR DS:[ESI]  

00414CBE    4A              DEC EDX  

00414CBF    03C2            ADD EAX,EDX  

00414CC1    42              INC EDX  

00414CC2    90              NOP  

00414CC3    90              NOP  

00414CC4    90              NOP  

00414CC5    90              NOP  

00414CC6    90              NOP  

00414CC7    90              NOP  

00414CC8    90              NOP  

00414CC9    90              NOP  

00414CCA    90              NOP  

00414CCB    90              NOP  

00414CCC    90              NOP  

00414CCD    90              NOP  

00414CCE    33C2            XOR EAX,EDX  

00414CD0    4A              DEC EDX  

00414CD1    C1CA 08         ROR EDX,8  

00414CD4    AB              STOS DWORD PTR ES:[EDI]  

00414CD5    49              DEC ECX  

00414CD6    9C              PUSHFD  

00414CD7    90              NOP  

00414CD8    90              NOP  

00414CD9    90              NOP  

00414CDA    90              NOP  

00414CDB    90              NOP  

00414CDC    90              NOP  

00414CDD    90              NOP  

00414CDE    90              NOP  

00414CDF    90              NOP  

00414CE0    C12C24 06       SHR DWORD PTR SS:[ESP],6  

00414CE4    F71424          NOT DWORD PTR SS:[ESP]  

00414CE7    832424 01       AND DWORD PTR SS:[ESP],1  

00414CEB    50              PUSH EAX  

00414CEC    52              PUSH EDX  

00414CED    B8 635A9AF0     MOV EAX,F09A5A63  

00414CF2    05 46A5650F     ADD EAX,0F65A546  

00414CF7    F76424 08       MUL DWORD PTR SS:[ESP+8]  

00414CFB    8D8428 19594000 LEA EAX,DWORD PTR DS:[EAX+EBP+405919]  

00414D02    894424 08       MOV DWORD PTR SS:[ESP+8],EAX  

00414D06    5A              POP EDX  

00414D07    58              POP EAX  

00414D08    8D6424 04       LEA ESP,DWORD PTR SS:[ESP+4]  

00414D0C    FF6424 FC       JMP DWORD PTR SS:[ESP-4]                                      ; PESpin.00414D14  

……  

00414D14    B9 13000000     MOV ECX,13  

00414D19    8DB5 05574000   LEA ESI,DWORD PTR SS:[EBP+405705]  

00414D1F    F3:A4           REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]   ; From:414B00 to:D1004E size:13  

00414D21    93              XCHG EAX,EBX  

00414D22    B9 0A000000     MOV ECX,0A  

00414D27    8BBD E6574000   MOV EDI,DWORD PTR SS:[EBP+4057E6]  

00414D2D    03BD EB574000   ADD EDI,DWORD PTR SS:[EBP+4057EB]  

00414D33    83C7 28         ADD EDI,28  

00414D36    F3:AB           REP STOS DWORD PTR ES:[EDI]  

00414D38    58              POP EAX  

00414D39    90              NOP  

00414D3A    90              NOP  

00414D3B    90              NOP  

00414D3C    90              NOP  

00414D3D    90              NOP  

00414D3E    90              NOP  

00414D3F    90              NOP  

00414D40    90              NOP  

00414D41    90              NOP  

00414D42    2D F9FFFFFF     SUB EAX,-7  

00414D47    90              NOP  

00414D48    90              NOP  

00414D49    90              NOP  

00414D4A    90              NOP  

00414D4B    90              NOP  

00414D4C    90              NOP  

00414D4D    90              NOP  

00414D4E    90              NOP  

00414D4F    90              NOP  

00414D50    90              NOP  

00414D51    90              NOP  

00414D52    90              NOP  

00414D53    90              NOP  

00414D54    90              NOP  

00414D55    90              NOP  

00414D56    90              NOP  

00414D57    90              NOP  

00414D58    90              NOP  

00414D58    FFE0            JMP EAX                                                       ; PESpin.0041317D  

……  

004132F6    F685 A15D4000 0 TEST BYTE PTR SS:[EBP+405DA1],1                               ; 这里判断是否选择了 API 重定位，0 表示不加密，1 表示加密  

004132FD    74 51           JE SHORT 00413350  

004132FF    90              NOP  
```



```
00413300    90                  NOP  

00413301    90                  NOP  

00413302    90                  NOP  

00413303    90                  NOP  

00413304    90                  NOP  

00413305    90                  NOP  

00413306    90                  NOP  

00413307    90                  NOP  

00413308    90                  NOP  

00413309    90                  NOP  

0041330A    90                  NOP  

0041330B    90                  NOP  

0041330C    90                  NOP  

0041330D    90                  NOP  

0041330E    90                  NOP  

0041330F    90                  NOP  

00413310    BB 3C080000         MOV EBX,83C                             ; 重定位 api 大小  

00413315    0BDB                OR EBX,EBX  

00413317    74 37               JE SHORT 00413350                       ; 如果重定位 API 大小为 0 就跳  

00413319    2BC0                SUB EAX,EAX  

0041331B    2185 D14B4000       AND DWORD PTR SS:[EBP+404BD1],EAX  

00413321    E8 01000000         CALL 00413327  

00413326    90                  NOP  

00413327    59                  POP ECX  

00413328    6A 40               PUSH 40                                 ; /Protect = PAGE\_EXECUTE\_READWRITE  

0041332A    68 00300000         PUSH 3000                               ; |AllocationType = MEM\_COMMIT|MEM\_RESERVE  

0041332F    53                  PUSH EBX                                  ; |Size = 83C (2108.)  

00413330    50                  PUSH EAX                                  ; |Address = NULL  

00413331    8D6424 FC           LEA ESP,DWORD PTR SS:[ESP-4]                ; |  

00413335    81C1 23000000       ADD ECX,23                                ; |  

0041333B    890C24              MOV DWORD PTR SS:[ESP],ECX                ; |Return Address  

0041333E    FFA5 0E4C4000       JMP DWORD PTR SS:[EBP+404C0E]             ; \\VirtualAlloc  

00413344    90                  NOP  

00413345    85C0                TEST EAX,EAX  

00413347    74 21               JE SHORT 0041336A  

00413349    50                  PUSH EAX  

0041334A    8F85 C94B4000       POP DWORD PTR SS:[EBP+404BC9]             ; [EBP+404BC9]保存 hmem(00D20000)  

00413350    8D85 4A0D3400       LEA EAX,DWORD PTR SS:[EBP+340D4A]  

00413356    8D80 5F320C00       LEA EAX,DWORD PTR DS:[EAX+C325F]  

0041335C    48                  DEC EAX  

0041335D    FFD0                CALL EAX                                  ; 004133A3  

……  

00414F25    6A 04               PUSH 4                                  ; /Protect = PAGE\_READWRITE  

00414F27    68 00300000         PUSH 3000                               ; |AllocationType = MEM\_COMMIT|MEM\_RESERVE  

00414F2C    51                  PUSH ECX                                  ; |Size = 5C (92.)  

00414F2D    6A 00               PUSH 0                                    ; |Address = NULL  

00414F2F    53                  PUSH EBX                                  ; |Return address  

00414F30    FFA5 0E4C4000       JMP DWORD PTR SS:[EBP+404C0E]             ; \\VirtualAlloc  

00414F36    0F01FE              INVLPG DH                                 ; Privileged command  

00414F39    8DB5 AA5A4000       LEA ESI,DWORD PTR SS:[EBP+405AAA]  

00414F3F    97                  XCHG EAX,EDI  

00414F40    8BDF                MOV EBX,EDI  

00414F42    B9 22000000         MOV ECX,22  

00414F47    F3:A4               REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]   ; From:414ea5 to:D30000 szie:22  

……  

0041340C    3BB5 C34B4000       CMP ESI,DWORD PTR SS:[EBP+404BC3]           ; ESI 保存输入表的起始地址 0040C160  

……  

00413468    8B5E 0C             MOV EBX,DWORD PTR DS:[ESI+C]  

0041346B    039D C34B4000       ADD EBX,DWORD PTR SS:[EBP+404BC3]  

00413471    8BFB                MOV EDI,EBX                               ; 第一个 API 的 Name 地址  

……  

00413473    E8 4C120000         CALL 004146C4                             ; 进去就是还原 DLL 名  
```

进去看看:

```
004146C4    57                  PUSH EDI  

004146C5    800F 00             OR BYTE PTR DS:[EDI],0                    ; 如果获取完全部的 DLL 就直接返回，否则 not 还原出正确的 DLL 名  

004146C8    74 16               JE SHORT 004146E0  

004146CA    90                  NOP  

004146CB    90                  NOP  

004146CC    90                  NOP  

004146CD    90                  NOP  

004146CE    90                  NOP  

004146CF    90                  NOP  

004146D0    90                  NOP  

004146D1    90                  NOP  

004146D2    90                  NOP  

004146D3    90                  NOP  

004146D4    90                  NOP  

004146D5    90                  NOP  

004146D6    90                  NOP  

004146D7    90                  NOP  

004146D8    90                  NOP  

004146D9    90                  NOP  

004146DA    90                  NOP  

004146DB    F617                NOT BYTE PTR DS:[EDI]  

004146DD    47                  INC EDI  

004146DE    EB E5               JMP SHORT 004146C5  

004146E0    5F                  POP EDI                                   ; PESpin.0040C4C8  

004146E1    C3                  RETN  

……  

0041347F    53                  PUSH EBX                                  ; /FileName = "KERNEL32.DLL  

00413480    50                  PUSH EAX                                  ; |  

00413481    FFB5 F54B4000       PUSH DWORD PTR SS:[EBP+404BF5]             ; \\LoadLibraryA  

00413487    814424 04 14000000  ADD DWORD PTR SS:[ESP+4],14  

……  

00413491    85C0                TEST EAX,EAX  

00413493    0F84 3F090000       JE 00413DD8                               ; 如果载入失败则 OVER  

00413499    E8 01000000         CALL 0041349F  

0041349E    90                  NOP  

0041349F    59                  POP ECX  

004134A0    50                  PUSH EAX  

004134A1    51                  PUSH ECX  

004134A2    55                  PUSH EBP  

004134A3    810424 12374000     ADD DWORD PTR SS:[ESP],00403712  

004134AA    814424 04 22000000  ADD DWORD PTR SS:[ESP+4],22  

004134B2    C3                  RETN                      ;这里进去相当于 GetModuleHandleA 获取 DLL 的句柄  

……  

004134C1    2BD2                SUB EDX,EDX                               ; 获取到句柄后把原有 DLL 的函数名给清 0  

……  

004134F0    800B 00             OR BYTE PTR DS:[EBX],0  

004134F3    74 0D               JE SHORT 00413502                       ; 如果全部清除完毕则跳  

004134F5    8813                MOV BYTE PTR DS:[EBX],DL                ; DLL 名清 0  

004134F7    C1C2 04             ROL EDX,4  

004134FA    90                  NOP  

004134FB    90                  NOP  

004134FC    90                  NOP  

004134FD    43                  INC EBX  

004134FE    FF6424 FC           JMP DWORD PTR SS:[ESP-4]  

00413502    93                  XCHG EAX,EBX  

00413503    8B56 10             MOV EDX,DWORD PTR DS:[ESI+10]  

00413506    0395 C34B4000       ADD EDX,DWORD PTR SS:[EBP+404BC3]         ; 定位 ThunkValue  

0041350C    830A 00             OR DWORD PTR DS:[EDX],0  

0041350F    0F84 59010000       JE 0041366E                               ; 如果该 DLL 的 API 处理完则跳去下一步  

00413515    90                  NOP  

00413516    90                  NOP  

00413517    90                  NOP  

00413518    90                  NOP  

00413519    90                  NOP  

0041351A    90                  NOP  

0041351B    90                  NOP  

0041351C    90                  NOP  

0041351D    90                  NOP  

0041351E    75 02               JNZ SHORT 00413522  

00413520    90                  NOP  

00413521    90                  NOP  

00413522    8B02                MOV EAX,DWORD PTR DS:[EDX]
```



## 汇编代码片段

```
00413524    A9 00000080           TEST EAX,80000000  
00413529    74 0A               JE SHORT 00413535  
0041352B    25 FFFFFF7F         AND EAX,7FFFFFFF  
00413530    2BFF                SUB EDI,EDI  
00413532    EB 09               JMP SHORT 0041353D  
00413534    90                  NOP  
00413535    40                  INC EAX  
00413536    0385 C34B4000       ADD EAX,DWORD PTR SS:[EBP+404BC3]  
0041353C    97                  XCHG EAX,EDI  
0041353D    68 AFFAD0F9         PUSH F9D0FAAF  
00413542    012C24              ADD DWORD PTR SS:[ESP],EBP  
00413545    810424 B4466F06     ADD DWORD PTR SS:[ESP],66F46B4  
0041354C    68 4D7B630F         PUSH 0F637B4D  
00413551    812C24 9643230F     SUB DWORD PTR SS:[ESP],0F234396  
00413558    012C24              ADD DWORD PTR SS:[ESP],EBP  
0041355B    C3                  RETN                      ; 这里返回 API 处理部分
```

跟进看看：

……

```
00412C70    8B00                MOV EAX,DWORD PTR DS:[EAX]  
00412C72    0385 AA374000       ADD EAX,DWORD PTR SS:[EBP+4037AA]          ; 获取到的 API 放到 eax 中  
00412C78    EB 10               JMP SHORT 00412C8A  
00412C7A    83C3 04             ADD EBX,4  
00412C7D    41                  INC ECX  
00412C7E    81F9 B5030000       CMP ECX,3B5  
00412C84  ^ 75 97               JNZ SHORT 00412C1D  
00412C86    33C0                XOR EAX,EAX  
00412C88    EB 3F               JMP SHORT 00412CC9  
00412C8A    8BBD 9E374000       MOV EDI,DWORD PTR SS:[EBP+40379E]  
00412C90    3BC7                CMP EAX,EDI                                 ; 判断是否要加密  
00412C92    76 35               JBE SHORT 00412CC9                          ; 如果小于或等于 7c80262c 则不加密直接填充  
00412C94    03BD A2374000       ADD EDI,DWORD PTR SS:[EBP+4037A2]  
00412C9A    3BF8                CMP EDI,EAX  
00412C9C    76 2B               JBE SHORT 00412CC9  
00412C9E    8DBD 052C4000       LEA EDI,DWORD PTR SS:[EBP+402C05]  
00412CA4    96                  XCHG EAX,ESI  
00412CA5    33C9                XOR ECX,ECX  
00412CA7    8A0431              MOV AL,BYTE PTR DS:[ECX+ESI]  
00412CAA    3C 2E               CMP AL,2E  
00412CAC    74 04               JE SHORT 00412CB2  
00412CAE    41                  INC ECX  
00412CAF    AA                  STOS BYTE PTR ES:[EDI]  
00412CB0  ^ EB F5               JMP SHORT 00412CA7  
00412CB2    41                  INC ECX  
00412CB3    03F1                ADD ESI,ECX  
00412CB5    56                  PUSH ESI  
00412CB6    2C 2E               SUB AL,2E  
00412CB8    AA                  STOS BYTE PTR ES:[EDI]  
00412CB9    2BF9                SUB EDI,ECX  
00412CBB    57                  PUSH EDI  
00412CBC    FF95 F54B4000       CALL DWORD PTR SS:[EBP+404BF5]  
00412CC2    50                  PUSH EAX  
00412CC3    FF95 FF4B4000       CALL DWORD PTR SS:[EBP+404BFF]  
00412CC9    EB 01               JMP SHORT 00412CCC  
00412CCB    90                  NOP  
00412CCC    894424 1C           MOV DWORD PTR SS:[ESP+1C],EAX             ; 填充 API  
00412CD0    61                  POPAD  
00412CD1    FF0424              INC DWORD PTR SS:[ESP]
```

……

```
0041355F   /0F84 36080000       JE 00413D9B                                   ; 如果获取 API 失败则 over
```

……

```
004135A2    0FBA67 FF 07    BT DWORD PTR DS:[EDI-1],7        ; 获取[EDI-1]的第 7 位位传送给 CF,如果 cf 为 1 刚加密 api
```

所以这里可以直接 patch 成 `clc`

```
004135A7    EB 01            JMP SHORT 004135AA  
004135A9    90               NOP  
004135AA    9C               PUSHFD  
004135AB    F71424           NOT DWORD PTR SS:[ESP]  
004135AE    832424 01        AND DWORD PTR SS:[ESP],1  
004135B2    50               PUSH EAX  
004135B3    52               PUSH EDX  
004135B4    B8 2E306BF9      MOV EAX,F96B302E  
004135B9    05 31D09406      ADD EAX,694D031  
004135BE    F76424 08        MUL DWORD PTR SS:[ESP+8]  
004135C2    8D8428 E9414000 LEA EAX,DWORD PTR DS:[EAX+EBP+4041E9]  
004135C9    894424 08        MOV DWORD PTR SS:[ESP+8],EAX  
004135CD    5A               POP EDX  
004135CE    58               POP EAX  
004135CF    90               NOP  
004135D0    90               NOP  
004135D1    90               NOP  
004135D2    90               NOP  
004135D3    90               NOP  
004135D4    90               NOP  
004135D5    90               NOP  
004135D6    90               NOP  
004135D7    90               NOP  
004135D8    90               NOP  
004135D9    90               NOP  
004135DA    90               NOP  
004135DB    8D6424 04        LEA ESP,DWORD PTR SS:[ESP+4]  
004135DF    FF6424 FC        JMP DWORD PTR SS:[ESP-4]      ; CF 为 1 则加密 API，加密就跳去 eip+5 处
```

……

```
00413614    E8 03000000      CALL 0041361C  
00413619    A0 9AFF5B81      MOV AL,BYTE PTR DS:[815BFF9A]  
0041361E    C3               RETN  
0041361F    1900             SBB DWORD PTR DS:[EAX],EAX  
00413621    0000             ADD BYTE PTR DS:[EAX],AL  
00413623    53               PUSH EBX  
00413624    8D9D C050288E    LEA EBX,DWORD PTR SS:[EBP+8E2850C0]  
0041362A    81EB BC1AE88D    SUB EBX,8DE81ABC  
00413630    FFE3             JMP EBX                        ; 这里跳去加密 api
```

……

当 `CF` 为 0 时跳到这里:

```
00413643    E8 C4F6FFFF      CALL 00412D0C                   ; 不用加密则处理 jmp 表
```

进来看看：

```
00412D18    57               PUSH EDI                         ; 这段代码和 1.0 没有什么变化  
00412D19    EB 01            JMP SHORT 00412D1C  
00412D1B    90               NOP  
00412D1C    51               PUSH ECX  
00412D1D    90               NOP  
00412D1E    90               NOP  
00412D1F    90               NOP  
00412D20    90               NOP  
00412D21    90               NOP  
00412D22    90               NOP  
00412D23    90               NOP  
00412D24    90               NOP  
00412D25    90               NOP  
00412D26    BF DA9A4000      MOV EDI,00409ADA  
00412D2B    EB 01            JMP SHORT 00412D2E  
00412D2D    90               NOP  
00412D2E    B9 8C010000      MOV ECX,18C  
00412D33    90               NOP  
00412D34    90               NOP  
00412D35    90               NOP  
00412D36    90               NOP  
00412D37    90               NOP  
00412D38    90               NOP  
00412D39    90               NOP  
00412D3A    90               NOP  
00412D3B    90               NOP  
00412D3C    90               NOP  
00412D3D    90               NOP  
00412D3E    90               NOP  
00412D3F    90               NOP  
00412D40    90               NOP  
00412D41    90               NOP  
00412D42    90               NOP  
00412D43    90               NOP  
00412D44    3917             CMP DWORD PTR DS:[EDI],EDX      ; 判断是否找到了该地址  
00412D46    90               NOP  
00412D47    90               NOP  
00412D48    90               NOP  
00412D49    90               NOP  
00412D4A    90               NOP  
00412D4B    90               NOP  
00412D4C    90               NOP  
00412D4D    90               NOP  
00412D4E    90               NOP  
00412D4F    90               NOP  
00412D50    90               NOP  
00412D51    90               NOP  
00412D52    0F84 90000000    JE 00412DE8                     ; 如果找到则跳  
00412D58    47               INC EDI  
00412D59    EB 01            JMP SHORT 00412D5C  
00412D5B    90               NOP  
00412D5C    49               DEC ECX  
00412D5D    9C               PUSHFD  
00412D5E    C12C24 06        SHR DWORD PTR SS:[ESP],6  
00412D62    F71424           NOT DWORD PTR SS:[ESP]  
00412D65    832424 01        AND DWORD PTR SS:[ESP],1  
00412D69    50               PUSH EAX  
00412D6A    52               PUSH EDX  
00412D6B    B8 6592DC52      MOV EAX,52DC9265  
00412D70    05 446D23AD      ADD EAX,AD236D44  
00412D75    F76424 08        MUL DWORD PTR SS:[ESP+8]  
00412D79    90               NOP
```



```
00412D7A    90                      NOP  

00412D7B    90                      NOP  

00412D7C    90                      NOP  

00412D7D    90                      NOP  

00412D7E    90                      NOP  

00412D7F    90                      NOP  

00412D80    90                      NOP  

00412D81    90                      NOP  

00412D82    8D8428 A0394000 LEA EAX,DWORD PTR DS:[EAX+EBP+4039A0]  

00412D89    894424 08          MOV DWORD PTR SS:[ESP+8],EAX  

00412D8D    5A                  POP EDX  

00412D8E    58                  POP EAX  

00412D8F    8D6424 04          LEA ESP,DWORD PTR SS:[ESP+4]  

00412D93    FF6424 FC          JMP DWORD PTR SS:[ESP-4]                ; 循环回去找到该地址  

……  

00412DCF    90                      NOP  

00412DD0    8902                MOV DWORD PTR DS:[EDX],EAX              ; 没有找到则直接填充  

00412DD2    90                      NOP  

00412DD3    90                      NOP  

00412DD4    90                      NOP  

00412DD5    90                      NOP  

00412DD6    90                      NOP  

00412DD7    90                      NOP  

00412DD8    90                      NOP  

00412DD9    90                      NOP  

00412DDA    90                      NOP  

00412DDB    90                      NOP  

00412DDC    90                      NOP  

00412DDD    90                      NOP  

00412DDE    90                      NOP  

00412DDF    90                      NOP  

00412DE0    90                      NOP  

00412DE1    90                      NOP  

00412DE2    90                      NOP  

00412DE3    E9 B2000000      JMP 00412E9A                            ; 填充完跳去返回 处  

00412DE8    90                      NOP  

00412DE9    90                      NOP  

00412DEA    90                      NOP  

00412DEB    90                      NOP  

00412DEC    90                      NOP  

00412DED    90                      NOP  

00412DEE    90                      NOP  

00412DEF    90                      NOP  

00412DF0    90                      NOP  

00412DF1    807F FF 00       CMP BYTE PTR DS:[EDI-1],0               ; 如果地址前一位为空则直接填充 API  

00412DF5    74 60           JE SHORT 00412E57  

……  

00412E08    807F FF EA       CMP BYTE PTR DS:[EDI-1],0EA              ; 如果 EDI-1 位为 EA 的情况  

00412E0C  ^ 75 90           JNZ SHORT 00412D9E  

00412E0E    90                      NOP  

00412E0F    90                      NOP  

00412E10    90                      NOP  

00412E11    90                      NOP  

00412E12    90                      NOP  

00412E13    90                      NOP  

00412E14    90                      NOP  

00412E15    90                      NOP  

00412E16    90                      NOP  

00412E17    FE4F FF          DEC BYTE PTR DS:[EDI-1]                 ; 当为 EA 时改成 e9 远程跳去壳存放 API 的地方  

00412E1A    83C7 04          ADD EDI,4  

00412E1D    2BC7             SUB EAX,EDI  

00412E1F    8947 FC          MOV DWORD PTR DS:[EDI-4],EAX  

……  

patch 一下:  

00412E0E    66:C747 FF FF25 MOV WORD PTR DS:[EDI-1],25FF  

00412E14    8957 01          MOV DWORD PTR DS:[EDI+1],EDX  

00412E17    8902             MOV DWORD PTR DS:[EDX],EAX  

……  

00412E97   /EB 01           JMP SHORT 00412E9A  

00412E99   |90                NOP  

00412E9A   \59                POP ECX  

00412E9B    EB 01           JMP SHORT 00412E9E  

00412E9D    90                      NOP  

00412E9E    5F                  POP EDI  

00412E9F    C3                  RETN                            ; 返回  

……  

00413689  ^\E9 A1FDFFFF     JMP 0041342F                        ; 如果没有处理完全部的 API 则跳回去继续  

……  

00413773    F3:                  PREFIX REP:                   ; Superfluous prefix  

00413774    0F31             RDTSC                            ; 处理完全部的 api 就到这里来了,壳用 rdtsc 时间来反调试器  

00413776    50                  PUSH EAX  

00413777    F3:                  PREFIX REP:                   ; Superfluous prefix  

00413778    0F31             RDTSC  

0041377A    EB 01           JMP SHORT 0041377D  

把这两个 RDTSC 给 nop 掉就行了  

……  

004137B7    8D6424 04          LEA ESP,DWORD PTR SS:[ESP+4]  

004137BB    FF6424 FC          JMP DWORD PTR SS:[ESP-4]                ; 如果让上面的 RDTSC 执行的话，这里就会跳去错误 的地址  

……  

00413834    BB BDAED669     MOV EBX,69D6AEBD  

00413839    2BC3             SUB EAX,EBX  

0041383B    3D 99E925A9     CMP EAX,A925E999                  ; 这里判断加壳时有没有选择 code redirection  

00413840    90                      NOP  

00413841    90                      NOP  

00413842    90                      NOP  

00413843    90                      NOP  

00413844    90                      NOP  

00413845    90                      NOP  

00413846    90                      NOP  

00413847    90                      NOP  

00413848    90                      NOP  

00413849    74 79           JE SHORT 004138C4                 ; 如果没有选择 code 重定位则跳  

0041384B    BE A2524100     MOV ESI,004152A2                  ; 从 4152a2 处开始处理重定位代码  

00413850    B9 5C020000     MOV ECX,25C  

00413855    51                  PUSH ECX  

00413856    B0 05           MOV AL,5  

00413858    304431 FF       XOR BYTE PTR DS:[ECX+ESI-1],AL    ; 结束地址为 4154fd 计算方法为 xor 5  

0041385C    90                      NOP  

0041385D    90                      NOP  

0041385E    90                      NOP  

0041385F    90                      NOP  

00413860    90                      NOP  

00413861    90                      NOP  

00413862    90                      NOP  

00413863    90                      NOP  

00413864    90                      NOP  

00413865    90                      NOP  

00413866    90                      NOP  

00413867    90                      NOP  

00413868    004C31 FF       ADD BYTE PTR DS:[ECX+ESI-1],CL    ; 然后 xor cl 的值  

0041386C    49                  DEC ECX  

0041386D    9C                  PUSHFD  

0041386E    C12C24 06       SHR DWORD PTR SS:[ESP],6  

00413872    F71424          NOT DWORD PTR SS:[ESP]  

00413875    832424 01       AND DWORD PTR SS:[ESP],1  

00413879    50                  PUSH EAX  

0041387A    52                  PUSH EDX  

0041387B    B8 72B2DC12     MOV EAX,12DCB272  

00413880    05 444D23ED     ADD EAX,ED234D44  

00413885    F76424 08       MUL DWORD PTR SS:[ESP+8]  

00413889    8D8428 A7444000 LEA EAX,DWORD PTR DS:[EAX+EBP+4044A7]  

00413890    894424 08       MOV DWORD PTR SS:[ESP+8],EAX  

00413894    5A                  POP EDX  

00413895    58                  POP EAX  

00413896    8D6424 04       LEA ESP,DWORD PTR SS:[ESP+4]  

0041389A  ^ FF6424 FC       JMP DWORD PTR SS:[ESP-4]                ; 如果没有解压完则跳回去继续  

0041384B    BE A2524100     MOV ESI,004152A2                  ; 从 4152a2 处开始处理重定位代码  

00413850    B9 5C020000     MOV ECX,25C  

00413855    51                  PUSH ECX  

00413856    B0 05           MOV AL,5  

00413858    304431 FF       XOR BYTE PTR DS:[ECX+ESI-1],AL    ; 结束地址为 4154fd 计算方法为 xor 5  

0041385C    90                      NOP  

0041385D    90                      NOP  

0041385E    90                      NOP  

0041385F    90                      NOP  

00413860    90                      NOP  

00413861    90                      NOP  

00413862    90                      NOP  

00413863    90                      NOP  

00413864    90                      NOP  

00413865    90                      NOP  

00413866    90                      NOP  

00413867    90                      NOP  

00413868    004C31 FF       ADD BYTE PTR DS:[ECX+ESI-1],CL    ; 然后 xor cl 的值  

0041386C    49                  DEC ECX  

0041386D    9C                  PUSHFD  

0041386E    C12C24 06       SHR DWORD PTR SS:[ESP],6  

00413872    F71424          NOT DWORD PTR SS:[ESP]  

00413875    832424 01       AND DWORD PTR SS:[ESP],1  

00413879    50                  PUSH EAX  

0041387A    52                  PUSH EDX  

0041387B    B8 72B2DC12     MOV EAX,12DCB272  

00413880    05 444D23ED     ADD EAX,ED234D44  

00413885    F76424 08       MUL DWORD PTR SS:[ESP+8]  

00413889    8D8428 A7444000 LEA EAX,DWORD PTR DS:[EAX+EBP+4044A7]  

00413890    894424 08       MOV DWORD PTR SS:[ESP+8],EAX  

00413894    5A                  POP EDX  

00413895    58                  POP EAX  

00413896    8D6424 04       LEA ESP,DWORD PTR SS:[ESP+4]  

0041389A  ^ FF6424 FC       JMP DWORD PTR SS:[ESP-4]                ; 如果没有解压完则跳回去继续  

……  

004138A2    59                  POP ECX  

004138A3    90                      NOP  

004138A4    90                      NOP  
```



# 004138A5 - 004138B4: NOP Padding

```
004138A5    90                 NOP  
004138A6    90                 NOP  
004138A7    90                 NOP  
004138A8    90                 NOP  
004138A9    90                 NOP  
004138AA    90                 NOP  
004138AB    90                 NOP  
004138AC    90                 NOP  
004138AD    90                 NOP  
004138AE    90                 NOP  
004138AF    90                 NOP  
004138B0    90                 NOP  
004138B1    90                 NOP  
004138B2    90                 NOP  
004138B3    90                 NOP  
```

# 004138B4 - 004138C0: Setup and Copy

```
004138B4    BF C8014000      MOV EDI,004001C8  
004138B9    90                 NOP  
004138BA    90                 NOP  
004138BB    90                 NOP  
004138BC    90                 NOP  
004138BD    90                 NOP  
004138BE    90                 NOP  
004138BF    90                 NOP  
004138C0    90                 NOP  
```

# 004138C1 - 004138C2: Repetition and End

```
004138C1    90                 NOP  
004138C2    F3:A4              REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]      ; From 4152a2 to:4001c8 size:25c  

……  

004138FB    61                 POPAD                                             ; 到这里就着陆了  
004138FC    BA 0F5C8CCE      MOV EDX,CE8C5C0F                                    ; 程序 OEP 代码  
00413901    EB 01              JMP SHORT 00413904  
00413903    90                 NOP  
00413904    81F2 753DE58A   XOR EDX,8AE53D75  
0041390A    EB 01              JMP SHORT 0041390D  
0041390C    90                 NOP  
0041390D    2BC0               SUB EAX,EAX                                       ; sub eax,eax  
0041390F    EB 01              JMP SHORT 00413912  
00413911    90                 NOP  
00413912    68 1D39ACE7      PUSH E7AC391D  
00413917    810424 63979418 ADD DWORD PTR SS:[ESP],18949763                  ; push 40d080  
0041391E    50                 PUSH EAX                                        ; push eax  
0041391F    EB 01              JMP SHORT 00413922  
00413921    90                 NOP  
00413922    50                 PUSH EAX                                        ; push eax  
00413923    EB 01              JMP SHORT 00413926  
00413925    90                 NOP  
00413926    68 30394100      PUSH 00413930                                     ; call 00409AF2  
0041392B  - E9 C26FFFF      JMP 00409AF2                                        ; JMP to kernel32.CreateMutexA  
00413930    68 3A394100      PUSH 0041393A                                     ; call 00409B1C  
00413935  - E9 E26FFFF      JMP 00409B1C                                        ; JMP to ntdll.RtlGetLastWin32Error  
0041393A    3D B7000000      CMP EAX,0B7                                         ; CMP EAX,0B7  
0041393F    EB 01              JMP SHORT 00413942  
00413941    90                 NOP  
00413942  - E9 025AFFFF      JMP 00409349  
```

## 正确的 STOLEN code

```
00409326   .  BA 0F5C8CCE   MOV EDX,CE8C5C0F  
0040932B   .  81F2 753DE58A XOR EDX,8AE53D75  
00409331   .  2BC0          SUB EAX,EAX  
00409333   .  68 80D04000  PUSH 0040D080                             ; /MutexName = "PE_SPIN_v1.1"  
00409338   .  50            PUSH EAX                                  ; |InitialOwner => FALSE  
00409339   .  50            PUSH EAX                                  ; |pSecurity => NULL  
0040933A   .  E8 B3070000  CALL 00409AF2                             ; \CreateMutexA  
0040933F   .  E8 D8070000  CALL 00409B1C                             ;  JMP to ntdll.RtlGetLastWin32Error  
00409344   .  3D B7000000  CMP EAX,0B7  

……  
```

(上次分析到这里就没有下去了，自己整天忙的像什么一样,不过忙完又不知道忙了些什么 L)

这里开始比较重要是关系到后面的修复。下面分析出哪几个地方要修复：

## 第一种：Pe Header Stolen code

到壳的代码处可以看到很多类似这样的代码：

```
004093E9  - E9 1E6EFFFF        JMP 0040020C        ;*****  
004093EE    FF35 65E04000      PUSH DWORD PTR DS:[40E065]  
004093F4    E8 1E6EFFFF        CALL 00400217  
004093F9    2BC0              SUB EAX,EAX  
004093FB    50                PUSH EAX  
004093FC    E8 1C6EFFFF        CALL 0040021D        ;*****  
00409401    E8 1D6EFFFF        CALL 00400223        ;*****  
```

代码放到 `PE HEADER` 里去了，壳每次抽取 5 个字节主到 `PE` 头里去执行。这里的修复比较简单的 `J`。

## 第二种：SDK Protect

`V1.1` 最大的变化，比较有意思，又分为两种：

### 第一种情况：解码代码

```
00409369    9C                  PUSHFD  
0040936A    60                  PUSHAD  
0040936B    B9 C018265F        MOV ECX,5F2618C0  
00409370    BF BABA78D9        MOV EDI,D978BABA  
00409375    81E9 A318265F      SUB ECX,5F2618A3  
0040937B    B8 33F423AF        MOV EAX,AF23F433  
00409380    05 E7601D51        ADD EAX,511D60E7  
00409385    FF0D 8B934000      DEC DWORD PTR DS:[40938B]  
0040938B    FF10               CALL DWORD PTR DS:[EAX]        ;这里跟进去可以看到算法  
0040938D    61                  POPAD  
0040938E    9D                  POPFD  
```

### 具体解码代码

```
00D30000    90                  NOP  
……  
00D30009    81EF 512738D9       SUB EDI,D9382751  
00D3000F    87D9               XCHG ECX,EBX  
00D30011    B9 24000000        MOV ECX,24  
00D30016    2AC0               SUB AL,AL  
00D30018    FC                 CLD  
00D30019    F3:AA             REP STOS BYTE PTR ES:[EDI]  
00D3001B    87D9               XCHG ECX,EBX  
00D3001D    83C7 02            ADD EDI,2  
00D30020    8A07               MOV AL,BYTE PTR DS:[EDI]  
00D30022    90                  NOP  
00D30023    90                  NOP  
00D30024    90                  NOP  
00D30025    90                  NOP  
00D30026    90                  NOP  
00D30027    90                  NOP  
00D30028    FEC8               DEC AL  
00D3002A    C0C8 D1            ROR AL,0D1                   ; Shift constant out of range 1..31  
00D3002D    C0C8 D7            ROR AL,0D7                   ; Shift constant out of range 1..31  
00D30030    90                  NOP  
00D30031    90                  NOP  
00D30032    90                  NOP  
00D30033    FEC8               DEC AL  
00D30035    04 4E              ADD AL,4E  
00D30037    32C1               XOR AL,CL  
00D30039    C0C8 0F            ROR AL,0F  
00D3003C    FEC8               DEC AL  
00D3003E    90                  NOP  
00D3003F    90                  NOP  
00D30040    90                  NOP  
00D30041    90                  NOP  
00D30042    90                  NOP  
00D30043    90                  NOP  
00D30044    90                  NOP  
00D30045    90                  NOP  
00D30046    90                  NOP  
00D30047    90                  NOP  
00D30048    02C1               ADD AL,CL  
00D3004A    AA                 STOS BYTE PTR ES:[EDI]             ; 还原代码  
00D3004B    49                 DEC ECX  
00D3004C  ^ 75 D2             JNZ SHORT 00D30020  
00D30053    90                  NOP  
……  
00D3005A    C3                  RETN  
```

### 清除代码

```
004093AC   /EB 0B              JMP SHORT 004093B9          
004093AE   |90                 NOP  
004093AF   |81E9 2D08830B      SUB ECX,0B83082D  
004093B5   |40                 INC EAX  
004093B6   |74 10               JE SHORT 004093C8  
004093B8   |90                 NOP  
004093B9   \9C                 PUSHFD  
004093BA    EB 01               JMP SHORT 004093BD  
004093BC    90                 NOP  
004093BD    60                 PUSHAD  
004093BE    F9                 STC  
004093BF    1BC0               SBB EAX,EAX  
004093C1    B9 6E08830B        MOV ECX,0B83086E  
004093C6  ^ EB E7               JMP SHORT 004093AF  
004093C8    BF C8B93096        MOV EDI,9630B9C8  
004093CD    FC                 CLD  
004093CE    81C7 C7D90F6A     ADD EDI,6A0FD9C7  
004093D4    F3:AA              REP STOS BYTE PTR ES:[EDI]  
004093D6    48                 DEC EAX  
004093D7    75 04               JNZ SHORT 004093DD  
004093D9    9D                 POPFD  
004093DA    EB 05               JMP SHORT 004093E1  
004093DC    90                 NOP  
004093DD    61                 POPAD  
004093DE  ^ EB F9               JMP SHORT 004093D9  
```



004093E0    90                    NOP

**第二种情况：**

**解码/加密：**

```
00406348   /75 49                JNZ SHORT 00406393

0040634A  |FF15 3E554100   CALL DWORD PTR DS:[41553E]               ; 这里进去解码

00406350  |026B E8            ADD CH,BYTE PTR DS:[EBX-18]

00406353 |77 CD               JA SHORT 00406322

00406369 |99                  CDQ

……

00406382 |FF15 4E554100   CALL DWORD PTR DS:[41554E]               ; 这里进去把解码后的代码加密回去
```

**解码代码：**

```
……
00D00003    9C                 PUSHFD
00D00004    90                 NOP
00D00005    90                 NOP
00D00006    90                 NOP
00D00007    60                 PUSHAD
00D00008    8B4424 24          MOV EAX,DWORD PTR SS:[ESP+24]
00D0000C    8B08               MOV ECX,DWORD PTR DS:[EAX]
00D0000E    8D78 04            LEA EDI,DWORD PTR DS:[EAX+4]
00D00011    897C24 24          MOV DWORD PTR SS:[ESP+24],EDI
00D00015    81E9 D46AE877      SUB ECX,77E86AD4
00D0001B    FC                 CLD

……

00D00028    8A07               MOV AL,BYTE PTR DS:[EDI]
00D0002A    C0C8 42            ROR AL,42

00D0002D    90                 NOP
00D0002E    90                 NOP
00D0002F    90                 NOP
00D00030    90                 NOP
00D00031    04 D0              ADD AL,0D0
00D00033    02C1               ADD AL,CL
00D00035    FEC8               DEC AL
00D00037    04 09              ADD AL,9
00D00039    FEC8               DEC AL

00D0003B    90                 NOP
00D0003C    90                 NOP
00D0003D    90                 NOP
00D0003E    90                 NOP
00D0003F    90                 NOP
00D00040    90                 NOP
00D00041    34 2C              XOR AL,2C
00D00043    C0C0 DA            ROL AL,0DA

00D00046    90                 NOP
00D00047    90                 NOP
00D00048    90                 NOP
00D00049    90                 NOP
00D0004A    90                 NOP
00D0004B    90                 NOP
00D0004C    90                 NOP
00D0004D    90                 NOP
00D0004E    90                 NOP
00D0004F    FEC8               DEC AL
00D00051    90                 NOP
00D00052    AA                 STOS BYTE PTR ES:[EDI]                 ; 解码
00D00053    49                 DEC ECX
00D00054  ^ 75 D2              JNZ SHORT 00D00028
```

**加密代码：**

```
……
00D10009    9C                 PUSHFD
00D1000A    60                 PUSHAD
00D1000B    8B4424 24          MOV EAX,DWORD PTR SS:[ESP+24]
00D1000F    8B08               MOV ECX,DWORD PTR DS:[EAX]
00D10011    8D78 04            LEA EDI,DWORD PTR DS:[EAX+4]
00D10014    897C24 24          MOV DWORD PTR SS:[ESP+24],EDI
00D10018    8D7F F6            LEA EDI,DWORD PTR DS:[EDI-A]
00D1001B    81E9 F67FBB0E      SUB ECX,0EBB7FF6
00D10021    2BF9               SUB EDI,ECX
00D10023    FC                 CLD
00D10024    8A07               MOV AL,BYTE PTR DS:[EDI]
00D10026    90                 NOP
00D10027    FEC0               INC AL

00D10029    90                 NOP
00D1002A    90                 NOP
00D1002B    90                 NOP
00D1002C    90                 NOP
00D1002D    90                 NOP
00D1002E    90                 NOP
00D1002F    90                 NOP
00D10030    90                 NOP
00D10031    90                 NOP
00D10032    C0C8 DA            ROR AL,0DA

00D10035    34 2C              XOR AL,2C

00D10037    90                 NOP
00D10038    90                 NOP
00D10039    90                 NOP
00D1003A    90                 NOP
00D1003B    90                 NOP
00D1003C    90                 NOP
00D1003D    FEC0               INC AL
00D1003F    2C 09              SUB AL,9
00D10041    FEC0               INC AL
00D10043    2AC1               SUB AL,CL
00D10045    2C D0              SUB AL,0D0

00D10047    90                 NOP
00D10048    90                 NOP
00D10049    90                 NOP
00D1004A    90                 NOP
00D1004B   C0C0 42            ROL AL,42

00D1004E    AA                 STOS BYTE PTR ES:[EDI]
00D1004F    49                 DEC ECX
00D10050  ^ 75 D2              JNZ SHORT 00D10024
```

### 第三种：`Anti Unpack`

这个不知道算不算，但主程序里有一处检测：

```
00409837    B8 ABA44300        MOV EAX,43A4AB                   ; 检测是否被脱壳
0040983C    2D 910A0300        SUB EAX,30A91
00409841    FFD0               CALL EAX                         ; 这里进去就是具体的方法
```

看看具体的：

```
00409A1A    B8 1BBAD5FA        MOV EAX,FAD5BA1B
00409A1F    05 BFE06A05        ADD EAX,56AE0BF
00409A24    BB 4655A308        MOV EBX,8A35546
00409A29    81EB 5D54A308      SUB EBX,8A3545D                  ; EBX=0E9
00409A2F    2A18               SUB BL,BYTE PTR DS:[EAX]          ; 也就是检测 409ada 处是否为 0E9,如果不是后面就会乱跳
00409A31    58                 POP EAX
00409A32    C1C3 16            ROL EBX,16
00409A35    03C3               ADD EAX,EBX
00409A37    FFE0               JMP EAX                          ; 如果正确返回上面执行正确的代码
```

到这里壳已经分析完毕。转第二步了\_^.\_

### 第二步：脱壳

分析完就可以脱壳了，这里要写的修复代码也比较多哦。我用脚本、修改壳代码和自己写代码完成脱壳过程。

过程为：脚本修复 `IAT` 到 `stolen code` -> 申请空间 -> 写代码 -> 改 `EIP` -> 设断 -> 修改代码 -> 脚本清除“垃圾”代码 -> 去除 `anti-unpack`。

先写点脚本修复 `IAT` 并到 `OEP Stolen code` 处：脚本出下：

```
/*
//////////////////////////////////////////////////
    PESpin v1.1 Stolen Code Finder v0.1
    Author:    loveboom
    Email : loveboom#163.com
    OS    : WinXP sp1,Ollydbg 1.1,OllyScript v0.92
    Date  : 2005-3-9
        Action: 修复 IAT，停在 stolen code 处.
    Config: Ignore all exceptions
    Note  : If you have one or more question, email me please,thank you!
//////////////////////////////////////////////////
*/

var addr
var addr1

start:

   Msgyn "Config:Ignore all exceptions,continue?"

   cmp $RESULT,1

   je lbl1

   ret

lbl1:

  gpa "LoadLibraryA","kernel32.dll"        //在 LoadLibraryA+B 处下断
  mov addr,$RESULT
  add addr,B
  bp addr
  esto

lbl2:

  cmp eip,addr
  jne lblabort
  bc addr
  mov addr,esp
  add addr,c
  mov addr,[addr]
  bp addr
  esto
  bc addr

lbl3:

  find eip,#0FBA67FF07#        //find command 'bt [edi-1],7'
  cmp $RESULT,0
  je lblabort
  mov addr,$RESULT
  fill addr,1,F8        //修改为 clc 清除 CF
  inc addr
  mov [addr],90909090

lblnext1:

  find addr,#0F31#        //find command 'RDTSC'
  cmp $RESULT,0
  je lblabort
  find $RESULT,#FF6424FC#        //find command 'JMP DWORD PTR SS:[ESP-4]'
  cmp $RESULT,0
  je lblabort
  mov addr1,$RESULT
  bp addr1

lblfind1:

  find addr,#FF6424FC#        //find command 'JMP DWORD PTR SS:[ESP-4]'
  cmp $RESULT,0
  je lblabort
  go $RESULT
  sto
  sti

lblfind2:

  find eip,#807FFFEA#        //find command'CMP BYTE PTR DS:[EDI-1],0EA'
  cmp $RESULT,0
  je lblabort
  find $RESULT,#FE4FFF83C7042BC78947FC#

/*

find commands:

    FE4F FF         DEC BYTE PTR DS:[EDI-1]
    83C7 04         ADD EDI,4
    2BC7            SUB EAX,EDI
    8947 FC         MOV DWORD PTR DS:[EDI-4],EAX

*/

  cmp $RESULT,0
  je lblabort
  fill $RESULT,b,90
  mov addr,$RESULT
  bp addr

lblloop1:

  run

lblcheck:

  cmp eip,addr
  jne lbl4
  exec                //fix iat
    mov word ptr [edi-1],25FF
    mov [edi+1],edx
    mov [edx],eax
  ende
  jmp lblloop1

lbl4:

  bc addr
  bc addr1
  find eip,#E801000000??83C404#        //find commands:'call $+1 add esp,4'
  cmp $RESULT,0
  je lblerrver
  go $RESULT
  find $RESULT,#61#
  cmp $RESULT,0
  je lblerrver
  go $RESULT
  sto
  cmt eip,"Stolen code."

lblend:

  msg "Script finished,script by loveboom[DFCG][FCG][US].Thank you for using my script!"
```



`ret`

`lblabort:`

`msg "Error,script aborted.Maybe target is not protect by pespin 1.1 or you forgot ignore all exceptions."`

`ret`

`lblerrver:`

`msg "目标程序可能是用 pespin 1.0 或更低版本保护的!"`

`ret`

脚本运行完毕自己申请一点空间(自己手工或用工具都可以)，并写上一点代码，把 `EIP` 改为你的 `patch` 代码起始地址：

`.code`

`start:`
`pushfd`
`pushad`
`mov edi,401000h ;起始地址`
`mov ecx,0B000h ;搜索大小`
`push edi ;保护这两个寄存器方便后面写代码`
`push ecx`
`cld`
`lblpupfd01: ;pushfd 的情况`
`mov al,09Ch`
`lbllp1:`
`repne scas byte ptr [edi] ;查找 PUSHFD`
`jnz lblcallpart ;如果找完则跳`
`cmp byte ptr [edi],60h ;比较是否为 pushfd pushad`
`jne lbllp1`
`cmp word ptr [edi+23h],9D61h ;再次判断是否全要求`
`jne lbllp1`
`dec edi`
`call edi ;通过调用壳代码来还原程序代码`
`jmp lbllp1`
`lblcallpart: ;处理 PeHeader 的 stolen code`
`pop ecx`
`pop edi`
`push edi`
`push ecx`
`mov al,0E8h ;先查找 CALL 的部分`
`lblLoop:`
`repne scas byte ptr [edi]`
`jnz lbljmppart ;如果处理完则跳去处理 JMP 部分`
`mov edx,[edi]`
`lea edx,[edi+edx+4] ;取出绝对地址`
`cmp edx,4001C8h ;判断是否在范围内`
`jb lblLoop ;如果不合要求则跳上去`
`cmp edx,400428h ;这也是在判断是否合要求`
`ja lblLoop`
`cmp byte ptr [edx],0E9h ;判断是否为直接的 jmp address,其实这里可以不要写，我写是为了方便以后改代码:-)`
`jne lblLoop`
`push ecx`
`mov ecx,[edx+1] ;还原代码`
`lea ecx,[edx+ecx+5]`
`sub ecx,edi`
`sub ecx,4`
`mov [edi],ecx`
`pop ecx`
`jmp lblLoop`
`lbljmppart: ;Jmp 方式的处理`
`pop ecx`
`pop edi`
`push edi`
`push ecx`
`mov al,0E9h`
`lblloop01:`
`repne scas byte ptr [edi] ;这里和 CALL 的处理方式一样`
`jnz lblcallpart01`
`mov edx,[edi]`
`lea edx,[edi+edx+4]`
`cmp edx,4001c8h ;判断是否符合要求`
`jb lblloop01`
`cmp edx,400428h`
`ja lblloop01`
`cmp byte ptr [edx+5],0E9h ;判断是否只抽五个字节的代码，这里也是为了方便以后修改代码`
`jne lblloop01`
`push ecx`
`mov cl,byte ptr [edx] ;还原代码`
`mov byte ptr [edi-1],cl`
`mov ecx,[edx+1]`
`mov [edi],ecx`
`pop ecx`
`jmp lblloop01`
`lblcallpart01: ;处理 Call 的加密代码部分`
`pop ecx`
`pop edi`
`@@:`
`mov al,0FFh`
`lblloop02:`
`repne scas byte ptr [edi]`
`jnz lblend ;如果搜索完毕则收工:-)`
`cmp byte ptr [edi],15h ;判断是否为 CALL DS:[ADDRESS]`
`jne lblloop02`
`mov edx,[edi+1]`
`cmp edx,41553eh ;判断 ADDRESS 是否为 41553Eh,`
`je @F`
`cmp edx,41554Eh`
`jne lblloop02`
`push edi ;把壳加密代码的部分 nop 掉`
`dec edi`
`push ecx`
`mov ecx,0ah`
`mov al,90h`
`cld`
`rep stos byte ptr [edi]`
`pop ecx`
`pop edi`
`jmp @B`
`@@:`
`mov edx,edi ;调用壳的代码还原程序代码`
`dec edx`
`call edx`
`jmp lblloop02`
`lblend:`
`popad`
`popfd`
`end start`

我自己的如下：

`01120000    9C            PUSHFD`
`01120001    60            PUSHAD`
`01120002    BF 00104000   MOV EDI,401000`
`01120007    B9 00B00000   MOV ECX,0B000`
`0112000C    57            PUSH EDI`
`0112000D    51            PUSH ECX`
`0112000E    FC            CLD`
`0112000F    B0 9C         MOV AL,9C`
`01120011    F2:AE         REPNE SCAS BYTE PTR ES:[EDI]`
`01120013    75 12         JNZ SHORT 01120027`
`01120015    803F 60       CMP BYTE PTR DS:[EDI],60`
`01120018    ^ 75 F7       JNZ SHORT 01120011`
`0112001A    66:817F 23 619D CMP WORD PTR DS:[EDI+23],9D61`
`01120020    ^ 75 EF       JNZ SHORT 01120011`
`01120022    4F            DEC EDI`
`01120023    FFD7          CALL EDI ; 写完代码后，第一次在这里下个断，然后跟进修改壳代码`
`01120025    ^ EB EA       JMP SHORT 01120011`
`01120027    59            POP ECX`
`01120028    5F            POP EDI`
`01120029    57            PUSH EDI`
`0112002A    51            PUSH ECX`
`0112002B    B0 E8         MOV AL,0E8`
`0112002D    F2:AE         REPNE SCAS BYTE PTR ES:[EDI]`
`0112002F    75 2D         JNZ SHORT 0112005E`
`01120031    8B17          MOV EDX,DWORD PTR DS:[EDI]`
`01120033    8D543A 04     LEA EDX,DWORD PTR DS:[EDX+EDI+4]`
`01120037    81FA C8014000 CMP EDX,4001C8`
`0112003D    ^ 72 EE       JB SHORT 0112002D`
`0112003F    81FA 28044000 CMP EDX,400428`
`01120045    ^ 77 E6       JA SHORT 0112002D`
`01120047    803A E9       CMP BYTE PTR DS:[EDX],0E9`
`0112004A    ^ 75 E1       JNZ SHORT 0112002D`
`0112004C    51            PUSH ECX`
`0112004D    8B4A 01       MOV ECX,DWORD PTR DS:[EDX+1]`
`01120050    8D4C11 05     LEA ECX,DWORD PTR DS:[ECX+EDX+5]`
`01120054    2BCF          SUB ECX,EDI`
`01120056    83E9 04       SUB ECX,4`
`01120059    890F          MOV DWORD PTR DS:[EDI],ECX`
`0112005B    59            POP ECX`
`0112005C    ^ EB CF       JMP SHORT 0112002D`
`0112005E    59            POP ECX`
`0112005F    5F            POP EDI`
`01120060    57            PUSH EDI`
`01120061    51            PUSH ECX`
`01120062    B0 E9         MOV AL,0E9`
`01120064    F2:AE         REPNE SCAS BYTE PTR ES:[EDI]`
`01120066    75 2A         JNZ SHORT 01120092`
`01120068    8B17          MOV EDX,DWORD PTR DS:[EDI]`
`0112006A    8D543A 04     LEA EDX,DWORD PTR DS:[EDX+EDI+4]`
`0112006E    81FA C8014000 CMP EDX,4001C8`
`01120074    ^ 72 EE       JB SHORT 01120064`
`01120076    81FA 28044000 CMP EDX,400428`
`0112007C    ^ 77 E6       JA SHORT 01120064`
`0112007E    807A 05 E9    CMP BYTE PTR DS:[EDX+5],0E9`
`01120082    ^ 75 E0       JNZ SHORT 01120064`
`01120084    51            PUSH ECX`
`01120085    8A0A          MOV CL,BYTE PTR DS:[EDX]`
`01120087    884F FF       MOV BYTE PTR DS:[EDI-1],CL`
`0112008A    8B4A 01       MOV ECX,DWORD PTR DS:[EDX+1]`
`0112008D    890F          MOV DWORD PTR DS:[EDI],ECX`
`0112008F    59            POP ECX`
`01120090    ^ EB D2       JMP SHORT 01120064`
`01120092    59            POP ECX`
`01120093    5F            POP EDI`
`01120094    B0 FF         MOV AL,0FF`
`01120096    F2:AE         REPNE SCAS BYTE PTR ES:[EDI]`
`01120098    75 30         JNZ SHORT 011200CA`
`0112009A    803F 15       CMP BYTE PTR DS:[EDI],15`
`0112009D    ^ 75 F7       JNZ SHORT 01120096`
`0112009F    8B57 01       MOV EDX,DWORD PTR DS:[EDI+1]`
`011200A2    81FA 3E554100 CMP EDX,41553E`
`011200A8    74 19         JE SHORT 011200C3`
`011200AA    81FA 4E554100 CMP EDX,41554E`
`011200B0    ^ 75 E4       JNZ SHORT 01120096`
`011200B2    57            PUSH EDI`
`011200B3    4F            DEC EDI`
`011200B4    51            PUSH ECX`
`011200B5    B9 0A000000   MOV ECX,0A`
`011200BA    B0 90         MOV AL,90`
`011200BC    FC            CLD`
`011200BD    F3:AA         REP STOS BYTE PTR ES:[EDI]`
`011200BF    59            POP ECX`
`011200C0    5F            POP EDI`
`011200C1    ^ EB D1       JMP SHORT 01120094`
`011200C3    8BD7          MOV EDX,EDI`
`011200C5    4A            DEC EDX`
`011200C6    FFD2          CALL EDX ; 第一次这里也下断，然后进去利用壳的代码完成脱壳`
`011200C8    ^ EB CC       JMP SHORT 01120096`
`011200CA    61            POPAD`
`011200CB    9D            POPFD`

第一处中断后的 `patch` 代码：



# 汇编代码片段 1

```asm
00D30009    81EF 512738D9     SUB EDI,D9382751  
00D3000F    87D9              XCHG ECX,EBX  
00D30011    B9 26000000       MOV ECX,26  
00D30016    B0 90             MOV AL,90                  ; 清除原代码  
00D30018    FC                CLD  
00D30019    F3:AA             REP STOS BYTE PTR ES:[EDI]  
00D3001B    87D9              XCHG ECX,EBX  
00D3001D    90                NOP  
00D3001E    90                NOP  
00D3001F    90                NOP  
00D30020    8A07              MOV AL,BYTE PTR DS:[EDI]  
00D30022    F8                CLC  
00D30023    90                NOP  
00D30024    90                NOP  
00D30025    90                NOP  
00D30026    F9                STC  
00D30027    F9                STC  
00D30028    FEC8              DEC AL  
00D3002A    C0C8 D1           ROR AL,0D1                 ; Shift constant out of range 1..31  
00D3002D    C0C8 D7           ROR AL,0D7                 ; Shift constant out of range 1..31  
00D30030    90                NOP  
00D30031    90                NOP  
00D30032    90                NOP  
00D30033    FEC8              DEC AL  
00D30035    04 4E             ADD AL,4E  
00D30037    32C1              XOR AL,CL  
00D30039    C0C8 0F           ROR AL,0F  
00D3003C    FEC8              DEC AL  
00D3003E    F9                STC  
00D3003F    F8                CLC  
00D30040    90                NOP  
00D30041    F9                STC  
00D30042    90                NOP  
00D30043    90                NOP  
00D30044    90                NOP  
00D30045    90                NOP  
00D30046    90                NOP  
00D30047    90                NOP  
00D30048    02C1              ADD AL,CL  
00D3004A    AA                STOS BYTE PTR ES:[EDI]     ; 还原代码  
00D3004B    49                DEC ECX  
00D3004C  ^ 75 D2            JNZ SHORT 00D30020  
00D3004E    83C4 04           ADD ESP,4                  ; 还原完跳去我们自己的 patch 代码  
00D30051    61                POPAD  
00D30052    9D                POPFD  
00D30053    C3                RETN  
```

修改完把第一处的断点取消，然后 `F9` 运行，中断在第二处。

## 第二处中断后的 Patch 代码

```asm
00D00000    90                NOP  
00D00001    90                NOP  
00D00002    90                NOP  
00D00003    9C                PUSHFD  
00D00004    90                NOP  
00D00005    90                NOP  
00D00006    90                NOP  
00D00007    60                PUSHAD  
00D00008    8B4424 24         MOV EAX,DWORD PTR SS:[ESP+24]  
00D0000C    8B08              MOV ECX,DWORD PTR DS:[EAX]  
00D0000E    8D78 04           LEA EDI,DWORD PTR DS:[EAX+4]  
00D00011    897C24 24         MOV DWORD PTR SS:[ESP+24],EDI  
00D00015    81E9 D46AE877     SUB ECX,77E86AD4  
00D0001B    FC                CLD  
00D0001C    90                NOP  
00D0001D    90                NOP  
00D0001E    90                NOP  
00D0001F    90                NOP  
00D00020    90                NOP  
00D00021    90                NOP  
00D00022    90                NOP  
00D00023    90                NOP  
00D00024    90                NOP  
00D00025    90                NOP  
00D00026    90                NOP  
00D00027    90                NOP  
00D00028    8A07              MOV AL,BYTE PTR DS:[EDI]  
00D0002A    C0C8 42           ROR AL,42                  ; Shift constant out of range 1..31  
00D0002D    90                NOP  
00D0002E    90                NOP  
00D0002F    90                NOP  
00D00030    F8                CLC  
00D00031    04 D0             ADD AL,0D0  
00D00033    02C1              ADD AL,CL  
00D00035    FEC8              DEC AL  
00D00037    04 09             ADD AL,9  
00D00039    FEC8              DEC AL  
00D0003B    90                NOP  
00D0003C    90                NOP  
00D0003D    90                NOP  
00D0003E    90                NOP  
00D0003F    90                NOP  
00D00040    90                NOP  
00D00041    34 2C             XOR AL,2C  
00D00043    C0C0 DA           ROL AL,0DA  
00D00046    90                NOP  
00D00047    90                NOP  
00D00048    90                NOP  
00D00049    90                NOP  
00D0004A    90                NOP  
00D0004B    90                NOP  
00D0004C    90                NOP  
00D0004D    F9                STC  
00D0004E    F9                STC  
00D0004F    FEC8              DEC AL  
00D00051    90                NOP  
00D00052    AA                STOS BYTE PTR ES:[EDI]     ; 解码  
00D00053    49                DEC ECX  
00D00054  ^ 75 D2            JNZ SHORT 00D00028  
00D00056    8B7C24 24         MOV EDI,DWORD PTR SS:[ESP+24]   ; 清除原代码  
00D0005A    4F                DEC EDI  
00D0005B    FD                STD  
00D0005C    B0 90             MOV AL,90  
00D0005E    B9 0A000000       MOV ECX,0A  
00D00063    F3:AA             REP STOS BYTE PTR ES:[EDI]  
00D00065    61                POPAD  
00D00066    9D                POPFD  
00D00067    83C4 04           ADD ESP,4  
00D0006A    C3                RETN  
```

修改完取消断点在 `011200CB` 的下一行下断。`F9` 运行就可以。

```asm
011200CB    9D                POPFD  
```

再次断下后代码就已经修复的差不多了，再写一点脚本把上面的清除代码给删除掉：

```idc
//用于清除壳留下的"垃圾代码

var addr  
var endaddr  

start:  
  mov addr,401000        //起始地址  

loop:  
/*
  查找以下内容:

004093AC   /EB 0B                 JMP SHORT 004093B9             //这类代码也清除掉  
004093AE   |90                    NOP  
004093AF   |81E9 2D08830B         SUB ECX,0B83082D  
004093B5   |40                    INC EAX  
004093B6   |74 10                 JE SHORT 004093C8  
004093B8   |90                    NOP  
004093B9   \9C                    PUSHFD  
004093BA    EB 01                 JMP SHORT 004093BD  
004093BC    90                    NOP  
004093BD    60                    PUSHAD  
004093BE    F9                    STC  
004093BF    1BC0                  SBB EAX,EAX  
004093C1    B9 6E08830B           MOV ECX,0B83086E  
004093C6  ^ EB E7                 JMP SHORT 004093AF  
004093C8    BF C8B93096           MOV EDI,9630B9C8  
004093CD    FC                    CLD  
004093CE    81C7 C7D90F6A         ADD EDI,6A0FD9C7  
004093D4    F3:AA                 REP STOS BYTE PTR ES:[EDI]  
004093D6    48                    DEC EAX  
004093D7    75 04                 JNZ SHORT 004093DD  
004093D9    9D                    POPFD  
004093DA    EB 05                 JMP SHORT 004093E1  
004093DC    90                    NOP  
004093DD    61                    POPAD  
004093DE  ^ EB F9                 JMP SHORT 004093D9  
004093E0    90                    NOP  

*/  
  find addr,#EB0B??81E9????????407410??9CEB01??60F91BC0B9????????EBE7BF????????FC81C7????????F3AA4875049DEB05??61EBF9??#  
  cmp $RESULT,0  
  je lblend  
  fill $RESULT,35,90  
  mov addr,$RESULT  
  add addr,35  
  jmp loop  

lblend:  
  ret  
```

到这里代码已经全部修复好了,修复好的代码片段：

```asm
004093E9    68 20C14000       PUSH 0040C120                 ; ASCII "IDD_PE_SPIN"  
004093EE    FF35 65E04000     PUSH DWORD PTR DS:[40E065]  
004093F4    E8 A7070000       CALL 00409BA0                 ; JMP to USER32.DialogBoxParamA  
004093F9    2BC0              SUB EAX,EAX  
004093FB    50                PUSH EAX  
004093FC    E8 03070000       CALL 00409B04                 ; JMP to kernel32.ExitProcess  
00409401    E8 1E080000       CALL 00409C24                 ; JMP to COMCTL32.InitCommonControls  
```

最后把那个 `Anti-Unpack` 给清除掉：

```asm
00409837    B8 ABA44300       MOV EAX,43A4AB               ; 检测是否被脱壳  
0040983C    2D 910A0300       SUB EAX,30A91  
00409841    FFD0              CALL EAX                      ; 这里进去就是具体的方法  
```

修改：

```asm
00409837    90                NOP  
```



```
00409838    90                  NOP  
00409839    90                  NOP  
0040983A    90                  NOP  
0040983B    90                  NOP  
0040983C    90                  NOP  
0040983D    90                  NOP  
0040983E    90                  NOP  
0040983F    90                  NOP  
00409840    90                  NOP  
00409841    90                  NOP  
00409842    90                  NOP  
```

壳要处理的代码全部处理完了，现在 `Dmp` 用 `improtREC` 找回 `IAT` 就行了。

脱壳完毕！垃圾写了堆，从头到脚看是比较浪费时间^\_^。写文章也浪费 N 长的时间:-)。

但愿这篇文章对 `YOCK` 有点用…………….

`Greet`:

`Fly.Jingulong,yock,tDasm.David.hexer,hmimys,ahao.UFO(brother).alan(sister).all of my friends and you!`

`By loveboom[DFCG][FCG][US]`

`Email:loveboom#163.com`

`Date:2005-03-30 11:45`

`WroTe bY LoVeBOoM[DFCG][FCG][US]`

`4dd466d0e1e0cc123afbe504da2462ca`
