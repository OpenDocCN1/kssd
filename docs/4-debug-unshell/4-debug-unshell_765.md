

# MSLRH v0.31 脱壳分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-506.htm`](https://www.pediy.com/kssd/pediy07/pediy7-506.htm)

MSLRH v.031 脱壳分析  
【目     标】: MSLRH v0.31 主程序 附件:MSLRHv0.31a.rar  
【工     具】:Olydbg1.1(diy 版)、LORDPE、ImportREC1.6F   
【任     务】:分析外壳    
【操作平台】:WinXP sp2   
【作     者】: LOVEBOOM\[DFCG\]\[FCG\]\[US\]  
【相关链接】: 看雪里有的下，自己找找  
【简要说明】： 已经有 N 位兄弟写过了，我也来看下”戏”。这个壳的 RDTSC 真是很多。所以就更想看看有什么特别之处。  
【详细过程】：  
由于壳的”垃圾”太多了，所以老习惯写一点脚本用,这次没有用 ollyscript 来写，因为用它写的去垃圾脚本有时会使程序异常，所以改用去垃圾插件，写上以下代码：  
\[CODE\_ml01\]  
S = 0F31500F31??????????????????????????????????????????????2B0424??????????????????83C404  
R = 90909090909090909090909090909090909090909090909090909090909090909090909090909090909090  
\[CODE\_ml02\]  
S = 3DFF0F0000EB01??EB02????EB01??761BEB01??EB02????EB01??CC66B8FE00??????????????????66E764  
R = 9090909090909090909090909090909090909090909090909090909090909090909090909090909090909090  
  
\[CODE\_ml03\]  
S = E80A000000??EB0C????E8F6FFFFFFE8F2FFFFFF83C408  
R = 9090909090909090909090909090909090909090909090  
  
\[CODE\_ml04\]  
S = 50E802000000????586BC0??E802000000????83C40458  
R = 9090909090909090909090909090909090909090909090  
  
\[CODE\_ml05\]  
S = 74047502????EB01??  
R = 909090909090909090  
  
\[CODE\_ml06\]  
S = EB05??EB0440??EBFA  
R = 909090909090909090  
  
写完后可以用 OD 加载目标了。  
00456000 > $  60            PUSHAD                                   ;  壳入口  
00456001   .  D1CB          ROR EBX,1                                ;  这里很多垃圾的，可以先不管它  
00456003   .  0FCA          BSWAP EDX  
00456005   .  C1CA E0       ROR EDX,0E0                              ;  Shift constant out of range 1..31  
……  
004560FA   > \\E8 0A000000   CALL 00456109                            ;  直接这里 F4  
004560FF   .  E8 EB0C0000   CALL 00456DEF  
00456104   .  E8 F6FFFFFF   CALL 004560FF  
00456109   $  E8 F2FFFFFF   CALL 00456100  
……  
0045615A   > \\0F31          RDTSC                                     ;  到这里后，用脚本“清理”一下,没有”垃圾”的世界真清静呀  
0045615C   ?  50            PUSH EAX  
0045615D   ?  0F31          RDTSC  
……  
00456A98    0F31            RDTSC  
00456A9A    50              PUSH EAX  
00456A9B    E8 00000000     CALL 00456AA0  
00456AA0    810424 6F130000 ADD DWORD PTR SS:\[ESP\],136F  
00456AA7    64:FF35 0000000>PUSH DWORD PTR FS:\[0\]                     ; Install SEH  
00456AAE    64:8925 0000000>MOV DWORD PTR FS:\[0\],ESP                  ; 注意在 457E0F 下断  
……  
0045745C    33C0            XOR EAX,EAX                               ; 这里要发生异常了  
0045745E    0FB600          MOVZX EAX,BYTE PTR DS:\[EAX\]  
00457461    66:B8 FE00      MOV AX,0FE  
00457465    66:E7 64        OUT 64,AX                                 ; I/O command  
……  
异常后 SHIT＋F9 到 457E0F 处,然后继续跟到这里：  
004587B6    8B4424 0C       MOV EAX,DWORD PTR SS:\[ESP+C\]  
004587BA    33C9            XOR ECX,ECX  
004587BC    3348 04         XOR ECX,DWORD PTR DS:\[EAX+4\]              ; 清除相关 DRx 断点  
004587BF    3348 08         XOR ECX,DWORD PTR DS:\[EAX+8\]  
004587C2    3348 0C         XOR ECX,DWORD PTR DS:\[EAX+C\]  
004587C5    3348 10         XOR ECX,DWORD PTR DS:\[EAX+10\]  
004587C8    8B6424 08       MOV ESP,DWORD PTR SS:\[ESP+8\]  
004587CC    64:8F05 0000000>POP DWORD PTR FS:\[0\]  
004587D3    83C4 04         ADD ESP,4  
……  
0045917D    0F31            RDTSC                                    ; 一个异常后，再次用 RDTSC 的方式来反调试  
0045917F    2B0424          SUB EAX,DWORD PTR SS:\[ESP\]  
00459182    83C4 04         ADD ESP,4  
00459185    3D FFFFFF00     CMP EAX,0FFFFFF  
0045918A    76 05           JBE SHORT 00459191                       ; 这里一跳要跳，否则就 over 了  
0045918C    E9 F08E0000     JMP 00462081  
00459191    51              PUSH ECX  
00459192    33C9            XOR ECX,ECX  
00459194    E8 00000000     CALL 00459199  
00459199    5F              POP EDI  
0045919A    81C7 C4090000   ADD EDI,9C4  
004591A0    5A              POP EDX  
004591A1    83C2 15         ADD EDX,15  
004591A4    0FB60439        MOVZX EAX,BYTE PTR DS:\[ECX+EDI\]          ; 把值传到 eax 中(从 459b5d 处开始)  
004591A8    33C2            XOR EAX,EDX                              ; 取出的值 xor 15  
004591AA    880439          MOV BYTE PTR DS:\[ECX+EDI\],AL             ; 解密后的值保存到相关地址中  
004591AD    41              INC ECX  
004591AE    81F9 93000000   CMP ECX,93                               ; 要解密的大小为 93  
004591B4  ^ 72 EE           JB SHORT 004591A4                        ; 如果没有解压完则跳去继续解密  
……  
00459B5D    8B5C24 20       MOV EBX,DWORD PTR SS:\[ESP+20\]            ; 准备取 kernel base  
00459B61    66:BB 0000      MOV BX,0  
00459B65    0FB703          MOVZX EAX,WORD PTR DS:\[EBX\]  
00459B68    2D 4D5A0000     SUB EAX,5A4D  
00459B6D    74 08           JE SHORT 00459B77                        ; 如果找到 ODS 头则跳  
00459B6F    81EB 00000100   SUB EBX,10000                            ; UNICODE "ALLUSERSPROFILE=D:\\Documents and Settings\\All Users"  
00459B75  ^ EB EE           JMP SHORT 00459B65  
00459B77    8BFB            MOV EDI,EBX  
00459B79    037B 3C         ADD EDI,DWORD PTR DS:\[EBX+3C\]            ; 定位 pe 头  
00459B7C    83C7 78         ADD EDI,78  
00459B7F    8B3F            MOV EDI,DWORD PTR DS:\[EDI\]               ; 定位输出表  
00459B81    03FB            ADD EDI,EBX  
00459B83    57              PUSH EDI  
00459B84    83C7 20         ADD EDI,20  
00459B87    8B3F            MOV EDI,DWORD PTR DS:\[EDI\]               ; 取 AddressofNames  
00459B89    03FB            ADD EDI,EBX  
00459B8B    33C0            XOR EAX,EAX  
00459B8D    40              INC EAX  
00459B8E    8B0F            MOV ECX,DWORD PTR DS:\[EDI\]  
00459B90    03CB            ADD ECX,EBX                              ; 定位 api  
00459B92    83C7 04         ADD EDI,4  
00459B95    8139 47657450   CMP DWORD PTR DS:\[ECX\],50746547          ; 判断 API 名字前四位是否为 GetP  
00459B9B  ^ 75 F0           JNZ SHORT 00459B8D                       ; 如果不是则跳  
00459B9D    8179 04 726F634>CMP DWORD PTR DS:\[ECX+4\],41636F72        ; 判断后面是否为 rocA，这里也就是循环找出 GetProcAddress 的地址  
00459BA4  ^ 75 E7           JNZ SHORT 00459B8D                       ; 如果没有找到则继续找  
00459BA6    6BC0 02         IMUL EAX,EAX,2  
00459BA9    5F              POP EDI  
00459BAA    57              PUSH EDI  
00459BAB    83C7 24         ADD EDI,24  
00459BAE    8B3F            MOV EDI,DWORD PTR DS:\[EDI\]  
00459BB0    03FB            ADD EDI,EBX                              ; 定位 AddressofNameOrdinal  
00459BB2    03F8            ADD EDI,EAX  
00459BB4    66:8B07         MOV AX,WORD PTR DS:\[EDI\]  
00459BB7    6BC0 04         IMUL EAX,EAX,4  
00459BBA    5F              POP EDI  
00459BBB    83C7 1C         ADD EDI,1C  
00459BBE    8B3F            MOV EDI,DWORD PTR DS:\[EDI\]               ; 定位 AddressofFunctions  
00459BC0    03FB            ADD EDI,EBX  
00459BC2    03F8            ADD EDI,EAX  
00459BC4    8B7F FC         MOV EDI,DWORD PTR DS:\[EDI-4\]             ; 找到 GetProcAddress 的地址  
00459BC7    03FB            ADD EDI,EBX                              ; 找到的地址保存在 edi 中  
00459BC9    803F CC         CMP BYTE PTR DS:\[EDI\],0CC                ; 如果发现 api 下了 int3 断点就会异常出错  
00459BCC    75 09           JNZ SHORT 00459BD7                       ; 如果没有跟踪就跳  
00459BCE    33C9            XOR ECX,ECX                              ; 不跳就完了:-)  
00459BD0    33FF            XOR EDI,EDI  
00459BD2  ^ E9 C1CEFFFF     JMP 00456A98  
00459BD7    E8 00000000     CALL 00459BDC  
00459BDC    58              POP EAX  
00459BDD    2D EC3A0000     SUB EAX,3AEC  
00459BE2    B0 00           MOV AL,0                                 ; EAX=004560F0 定位壳入口  
00459BE4    05 00200100     ADD EAX,12000  
00459BE9    8BF0            MOV ESI,EAX                              ; EAX=00468000  
00459BEB    891E            MOV DWORD PTR DS:\[ESI\],EBX               ; kernel base 保存在 468000 处  
00459BED    897E 10         MOV DWORD PTR DS:\[ESI+10\],EDI            ; 保存 GetProcAddress 的地址到 468010 处  
00459BF0    33C9            XOR ECX,ECX  
00459BF2    E8 00000000     CALL 00459BF7  
00459BF7    5F              POP EDI  
00459BF8    81C7 C4090000   ADD EDI,9C4                              ; EDI=0045A5BB  
00459BFE    0FB60439        MOVZX EAX,BYTE PTR DS:\[ECX+EDI\]          ; 准备解开开始地址为 0045A5BB,大小为 0c3f 块的代码  
00459C02    83F0 15         XOR EAX,15                               ; 运算方法为 xor 15  
00459C05    880439          MOV BYTE PTR DS:\[ECX+EDI\],AL             ; 解压代码  
00459C08    41              INC ECX  
00459C09    81F9 3F0C0000   CMP ECX,0C3F  
00459C0F  ^ 72 ED           JB SHORT 00459BFE                        ; 没解压完则继续上去解压  
……  
0045A5B8    0F31            RDTSC                                     ; 这里取 API 的地址的方法比较有意思  
0045A5BA    50              PUSH EAX  
0045A5BB    EB 13           JMP SHORT 0045A5D0                        ; 跳去准备取 OutPutDebugStringA 的地址  
0045A5BD    4F              DEC EDI  
0045A5BE    75 74           JNZ SHORT 0045A634  
0045A5C0    70 75           JO SHORT 0045A637  
0045A5C2    74 44           JE SHORT 0045A608  
0045A5C4    65:6275 67      BOUND ESI,QWORD PTR GS:\[EBP+67\]           ; Superfluous prefix  
0045A5C8    53              PUSH EBX  
0045A5C9    74 72           JE SHORT 0045A63D  
0045A5CB    696E 67 4100E80>IMUL EBP,DWORD PTR DS:\[ESI+67\],0E80041  
0045A5D2    0000            ADD BYTE PTR DS:\[EAX\],AL  
0045A5D4    0083 2C2418FF   ADD BYTE PTR DS:\[EBX+FF18242C\],AL  
0045A5DA    36:FF56 10      CALL DWORD PTR SS:\[ESI+10\]  
0045A5DE    8946 14         MOV DWORD PTR DS:\[ESI+14\],EAX             ; 获取到的地址保存到 468014 处  
0045A5E1    EB 01           JMP SHORT 0045A5E4  
0045A5E3    68 EB02CD20     PUSH 20CD02EB  
0045A5E8    EB 01           JMP SHORT 0045A5EB  
0045A5EA    E8 E8100000     CALL 0045B6D7  
0045A5EF    0047 65         ADD BYTE PTR DS:\[EDI+65\],AL  
0045A5F2    74 43           JE SHORT 0045A637  
0045A5F4    6F              OUTS DX,DWORD PTR ES:\[EDI\]                ; I/O command  
0045A5F5    6D              INS DWORD PTR ES:\[EDI\],DX                 ; I/O command  
0045A5F6    6D              INS DWORD PTR ES:\[EDI\],DX                 ; I/O command  
0045A5F7    61              POPAD  
0045A5F8    6E              OUTS DX,BYTE PTR ES:\[EDI\]                 ; I/O command  
0045A5F9    64:4C           DEC ESP                                   ; Superfluous prefix  
0045A5FB    696E 65 4100FF3>IMUL EBP,DWORD PTR DS:\[ESI+65\],36FF0041  
0045A602    FF56 10         CALL DWORD PTR DS:\[ESI+10\]  
0045A605    8946 18         MOV DWORD PTR DS:\[ESI+18\],EAX             ; \[468018\]保存 GetCommandLineA 的地址　  
0045A608    90              NOP  
0045A609    90              NOP  
0045A60A    90              NOP  
0045A60B    90              NOP  
0045A60C    90              NOP  
0045A60D    90              NOP  
0045A60E    90              NOP  
0045A60F    90              NOP  
0045A610    90              NOP  
0045A611    E8 0C000000     CALL 0045A622                             ; 获取 CreateFileA 的地址  
0045A616    43              INC EBX  
0045A617    72 65           JB SHORT 0045A67E  
0045A619    61              POPAD  
0045A61A    74 65           JE SHORT 0045A681  
0045A61C    46              INC ESI  
0045A61D    696C65 41 00FF3>IMUL EBP,DWORD PTR SS:\[EBP+41\],FF36FF00  
0045A625    56              PUSH ESI  
0045A626    1089 461C9090   ADC BYTE PTR DS:\[ECX+90901C46\],CL  
……  
0045A7E6    E8 11000000     CALL 0045A7FC  
0045A7EB    47              INC EDI  
0045A7EC    65:74 4D        JE SHORT 0045A83C                        ; Superfluous prefix  
0045A7EF    6F              OUTS DX,DWORD PTR ES:\[EDI\]               ; I/O command  
0045A7F0    64:75 6C        JNZ SHORT 0045A85F                       ; Superfluous prefix  
0045A7F3    65:48           DEC EAX                                  ; Superfluous prefix  
0045A7F5    61              POPAD  
0045A7F6    6E              OUTS DX,BYTE PTR ES:\[EDI\]                ; I/O command  
0045A7F7    64:6C           INS BYTE PTR ES:\[EDI\],DX                 ; I/O command  
0045A7F9    65:41           INC ECX                                  ; Superfluous prefix  
0045A7FB    00FF            ADD BH,BH  
0045A7FD    36:FF56 10      CALL DWORD PTR SS:\[ESI+10\]  
0045A801    8946 50         MOV DWORD PTR DS:\[ESI+50\],EAX            ; 最后一个 GetModuleHandleA  
0045A804    90              NOP  
0045A805    90              NOP  
0045A806    90              NOP  
0045A807    90              NOP  
0045A808    90              NOP  
0045A809    90              NOP  
0045A80A    90              NOP  
0045A80B    90              NOP  
0045A80C    90              NOP  
0045A80D    90              NOP  
0045A80E    0F31            RDTSC  
0045A810    2B0424          SUB EAX,DWORD PTR SS:\[ESP\]               ; 又一个检测标志  
0045A813    83C4 04         ADD ESP,4  
0045A816    3D FFFFFF00     CMP EAX,0FFFFFF  
0045A81B  ^ 0F87 D0B8FFFF   JA 004560F1                              ; 如果发现跟踪则跳，也就 over 了  
0045A821   .  56            PUSH ESI                                  ;  ESI = 468000  
到这里壳就取完了全部壳要用到的 API，明细如下：  
  
  
  
  
   
……  
0045B1C9    8CC9            MOV CX,CS                                 ; 开始判断系统是否为 wk/xp 之类的  
0045B1CB    32C9            XOR CL,CL  
0045B1CD    83F9 00         CMP ECX,0  
0045B1D0    75 28           JNZ SHORT 0045B1FA                        ; 如果为 win9x 则跳  
0045B1D2    64:FF35 3000000>PUSH DWORD PTR FS:\[30\]  
0045B1D9    58              POP EAX  
0045B1DA    0FB648 02       MOVZX ECX,BYTE PTR DS:\[EAX+2\]             ; 取 TEB 的值,  
0045B1DE    884E 0C         MOV BYTE PTR DS:\[ESI+C\],CL  
0045B1E1    8B40 0C         MOV EAX,DWORD PTR DS:\[EAX+C\]  
0045B1E4    8B40 0C         MOV EAX,DWORD PTR DS:\[EAX+C\]  
0045B1E7    8D58 20         LEA EBX,DWORD PTR DS:\[EAX+20\]  
0045B1EA    8D48 18         LEA ECX,DWORD PTR DS:\[EAX+18\]  
0045B1ED    8103 C8000000   ADD DWORD PTR DS:\[EBX\],0C8          ;破坏 pe header  
0045B1F3    B8 00000000     MOV EAX,0  
0045B1F8    0101            ADD DWORD PTR DS:\[ECX\],EAX  
0045B1FA    33C9            XOR ECX,ECX  
0045B1FC    E8 00000000     CALL 0045B201  
0045B201    5F              POP EDI  
0045B202    81C7 C1090000   ADD EDI,9C1  
0045B208    0FB60439        MOVZX EAX,BYTE PTR DS:\[ECX+EDI\]           ; 从 45BBC2 开始解开下一段  
0045B20C    83F0 11         XOR EAX,11                                ; xor key 11  
0045B20F    880439          MOV BYTE PTR DS:\[ECX+EDI\],AL              ; 还原回去  
0045B212    41              INC ECX  
0045B213    81F9 521D0000   CMP ECX,1D52                              ; 解压代码大小 1D52  
0045B219  ^ 72 ED           JB SHORT 0045B208                         ; 如果没有解压完跳回去继续  
……  
0045C569    0F31            RDTSC                                     ; 又准备异常。  
0045C56B    50              PUSH EAX  
0045C56C    E8 00000000     CALL 0045C571                             ; Install SEH  
0045C571    810424 CA090000 ADD DWORD PTR SS:\[ESP\],9CA  
0045C578    64:FF35 0000000>PUSH DWORD PTR FS:\[0\]  
0045C57F    64:8925 0000000>MOV DWORD PTR FS:\[0\],ESP                  ; 异常地址 45CF3B  
0045C586    33DB            XOR EBX,EBX  
0045C588    8B1B            MOV EBX,DWORD PTR DS:\[EBX\]  
……  
0045D8DF    8B4424 0C       MOV EAX,DWORD PTR SS:\[ESP+C\]  
0045D8E3    33C9            XOR ECX,ECX  
0045D8E5    3348 04         XOR ECX,DWORD PTR DS:\[EAX+4\]              ; 再次清除硬件断点  
0045D8E8    3348 08         XOR ECX,DWORD PTR DS:\[EAX+8\]  
0045D8EB    3348 0C         XOR ECX,DWORD PTR DS:\[EAX+C\]  
0045D8EE    3348 10         XOR ECX,DWORD PTR DS:\[EAX+10\]  
0045D8F1    8B6424 08       MOV ESP,DWORD PTR SS:\[ESP+8\]  
0045D8F5    64:8F05 0000000>POP DWORD PTR FS:\[0\]  
0045D8FC    83C4 04         ADD ESP,4  
0045D8FF    0F31            RDTSC  
0045D901    2B0424          SUB EAX,DWORD PTR SS:\[ESP\]  
0045D904    83C4 04         ADD ESP,4  
0045D907    3D FFFFFF00     CMP EAX,0FFFFFF                           ; 这里一定要跳,又一个时间差来 anit debug  
0045D90C    76 06           JBE SHORT 0045D914  
0045D90E    5E              POP ESI  
0045D90F    C646 0F 01      MOV BYTE PTR DS:\[ESI+F\],1  
0045D913    56              PUSH ESI  
……  
0045DA75    5E              POP ESI  
0045DA76    884E 0D         MOV BYTE PTR DS:\[ESI+D\],CL  
……  
0045E420    E8 05000000     CALL 0045E42A  
0045E425    25 73257300     AND EAX,732573                            ; /Debug String =%s%s  
0045E42A    FF56 14         CALL DWORD PTR DS:\[ESI+14\]                ; \\OutPutStringA   
注：如果你没有 patch　OD 的那个漏洞的话，这里是过不来的  
……  
0045EDD4    FF56 18         CALL DWORD PTR DS:\[ESI+18\]                ; GetCommandLineA 获取命令行  
0045EDD7    40              INC EAX  
0045EDD8    33C9            XOR ECX,ECX  
0045EDDA    41              INC ECX                                   ; 获取命令行长度，值保存在 ECX 中  
0045EDDB    803C01 00       CMP BYTE PTR DS:\[ECX+EAX\],0  
0045EDDF    74 0C           JE SHORT 0045EDED  
0045EDE1    803C01 22       CMP BYTE PTR DS:\[ECX+EAX\],22              ; 如果没到结尾则继续回去取  
0045EDE5  ^ 75 F3           JNZ SHORT 0045EDDA  
0045EDE7    C60401 00       MOV BYTE PTR DS:\[ECX+EAX\],0  
0045EDEB  ^ EB ED           JMP SHORT 0045EDDA  
0045EDED    6A 00           PUSH 0                                    ; /hTemplateFile = NULL  
0045EDEF    6A 00           PUSH 0                                    ; |Attributes = 0  
0045EDF1    6A 03           PUSH 3                                    ; |Mode = OPEN\_EXISTING  
0045EDF3    6A 00           PUSH 0                                    ; |pSecurity = NULL  
0045EDF5    6A 00           PUSH 0                                    ; |ShareMode = 0  
0045EDF7    68 00000080     PUSH 80000000                             ; |Access = GENERIC\_READ  
0045EDFC    50              PUSH EAX                                  ; |FileName = "D:\\\[MSLRH\].exe"  
0045EDFD    FF56 1C         CALL DWORD PTR DS:\[ESI+1C\]                ; \\CreateFileA  
0045EE00    90              NOP  
  
这里用 CreateFileA 使 imp rec 不能打开文件。这里可以 patch 一下  
push eax  
Call CloseHandle  
这样 imp rec 就可以用了  
……  
0045F7A7    837E 40 00      CMP DWORD PTR DS:\[ESI+40\],0               ; 判断获取 ZwQueryInformationProcess 的地址有没有成功  
0045F7AB    74 24           JE SHORT 0045F7D1                         ; 没有成功则跳,所以这里可以直接跳过的  
0045F7AD    FF56 24         CALL DWORD PTR DS:\[ESI+24\]                ; 否则先获取当前进程的 ID GetCurrentProcessID  
0045F7B0    50              PUSH EAX                                  ; /ProcessId  
0045F7B1    6A 00           PUSH 0                                    ; |Inheritable = FALSE  
0045F7B3    68 00040000     PUSH 400                                  ; |Access = QUERY\_INFORMATION  
0045F7B8    FF56 28         CALL DWORD PTR DS:\[ESI+28\]                ; \\OpenProcess 打开自己的进程  
0045F7BB    8BDC            MOV EBX,ESP                               ; ESP =12FFA4  
0045F7BD    83EB 04         SUB EBX,4  
0045F7C0    6A 00           PUSH 0  
0045F7C2    6A 00           PUSH 0                                    ; /pReqsize = NULL  
0045F7C4    6A 04           PUSH 4                                    ; |Bufsize = 4  
0045F7C6    53              PUSH EBX                                  ; |Buffer = 0012FFA0  
0045F7C7    6A 07           PUSH 7                                    ; |InfoClass = 7  
0045F7C9    50              PUSH EAX                                  ; |hProcess  
0045F7CA    FF56 40         CALL DWORD PTR DS:\[ESI+40\]                ; \\ZwQueryInformationProcess  
0045F7CD    58              POP EAX  
0045F7CE    8846 0E         MOV BYTE PTR DS:\[ESI+E\],AL                ; 在\[46800E\]处做个标志，如果执行 ZwQueryInformationProcess 成功，则设置为 FF  
……  
00460178    8CC9            MOV CX,CS  
0046017A    32C9            XOR CL,CL  
0046017C    83F9 00         CMP ECX,0  
0046017F    0F84 A1130000   JE 00461526                               ; 如果系统为 win 2k/xp 的话则跳,我用 xp sp2 调试的，所以当然会跳了  
00460185    8B46 38         MOV EAX,DWORD PTR DS:\[ESI+38\]  
00460188    8078 01 4C      CMP BYTE PTR DS:\[EAX+1\],4C  
0046018C    0F85 94130000   JNZ 00461526  
00460192    E8 00000000     CALL 00460197  
00460197    810424 6E130000 ADD DWORD PTR SS:\[ESP\],136E  
0046019E    59              POP ECX  
0046019F    64:FF35 0000000>PUSH DWORD PTR FS:\[0\]  
004601A6    8B46 38         MOV EAX,DWORD PTR DS:\[ESI+38\]  
004601A9    8B40 0B         MOV EAX,DWORD PTR DS:\[EAX+B\]  
004601AC    8908            MOV DWORD PTR DS:\[EAX\],ECX  
……  
00461ECD    E8 00000000     CALL 00461ED2  
00461ED2    58              POP EAX  
00461ED3    2D E2BD0000     SUB EAX,0BDE2                             ; EAX = 004560F0  
00461ED8    B0 00           MOV AL,0  
00461EDA    05 00200100     ADD EAX,12000                             ; eax = 00468000  
00461EDF    8BF0            MOV ESI,EAX  
00461EE1    807E 0C 00      CMP BYTE PTR DS:\[ESI+C\],0                 ; 这个不知道有什么作用:-(  
00461EE5    74 51           JE SHORT 00461F38                         ; 这里跳  
00461EE7    6A 00           PUSH 0  
00461EE9    FF56 50         CALL DWORD PTR DS:\[ESI+50\]                ; GetModuleHandleA  
00461EEC    50              PUSH EAX  
00461EED    8BD8            MOV EBX,EAX  
00461EEF    8B40 3C         MOV EAX,DWORD PTR DS:\[EAX+3C\]             ; 定位 pe 头  
00461EF2    03C3            ADD EAX,EBX  
00461EF4    8D98 00010000   LEA EBX,DWORD PTR DS:\[EAX+100\]  
00461EFA    8B1B            MOV EBX,DWORD PTR DS:\[EBX\]  
00461EFC    58              POP EAX  
00461EFD    03D8            ADD EBX,EAX  
00461EFF    05 00100000     ADD EAX,1000  
00461F04    8BF8            MOV EDI,EAX  
00461F06    81EB FF000000   SUB EBX,0FF  
00461F0C    B9 10270000     MOV ECX,2710  
00461F11    0F31            RDTSC  
00461F13    C1E8 18         SHR EAX,18  
00461F16    03F8            ADD EDI,EAX  
00461F18    3007            XOR BYTE PTR DS:\[EDI\],AL  
00461F1A    3BFB            CMP EDI,EBX  
00461F1C    7D 03           JGE SHORT 00461F21  
00461F1E    49              DEC ECX  
00461F1F  ^ 75 F0           JNZ SHORT 00461F11  
00461F21    90              NOP  
00461F22    90              NOP  
00461F23    90              NOP  
00461F24    90              NOP  
00461F25    90              NOP  
00461F26    90              NOP  
00461F27    90              NOP  
00461F28    90              NOP  
00461F29    90              NOP  
00461F2A    90              NOP  
00461F2B    90              NOP  
00461F2C    90              NOP  
00461F2D    90              NOP  
00461F2E    90              NOP  
00461F2F    90              NOP  
00461F30    90              NOP  
00461F31    90              NOP  
00461F32    90              NOP  
00461F33    90              NOP  
00461F34    90              NOP  
00461F35    90              NOP  
00461F36    90              NOP  
00461F37    90              NOP  
00461F38    807E 0D 00      CMP BYTE PTR DS:\[ESI+D\],0  
00461F3C  ^ 0F85 AF41FFFF   JNZ 004560F1  
00461F42    90              NOP  
00461F43    90              NOP  
00461F44    90              NOP  
00461F45    90              NOP  
00461F46    90              NOP  
00461F47    90              NOP  
00461F48    90              NOP  
00461F49    90              NOP  
00461F4A    90              NOP  
00461F4B    807E 0E 00      CMP BYTE PTR DS:\[ESI+E\],0  
00461F4F  ^ 0F85 9C41FFFF   JNZ 004560F1  
00461F55    90              NOP  
00461F56    90              NOP  
00461F57    90              NOP  
00461F58    90              NOP  
00461F59    90              NOP  
00461F5A    90              NOP  
00461F5B    90              NOP  
00461F5C    90              NOP  
00461F5D    90              NOP  
00461F5E    90              NOP  
00461F5F    90              NOP  
00461F60    90              NOP  
00461F61    90              NOP  
00461F62    90              NOP  
00461F63    90              NOP  
00461F64    90              NOP  
00461F65    90              NOP  
00461F66    90              NOP  
00461F67    90              NOP  
00461F68    90              NOP  
00461F69    90              NOP  
00461F6A    90              NOP  
00461F6B    90              NOP  
00461F6C    807E 0F 00      CMP BYTE PTR DS:\[ESI+F\],0  
00461F70  ^ 0F85 7B41FFFF   JNZ 004560F1  
……  
00461F8D    E8 00000000     CALL 00461F92                            ; 这里一段开始效验 CRC 值,所以我们现在把前面的代码还原回去  
00461F92    59              POP ECX  
00461F93    90              NOP  
00461F94    90              NOP  
00461F95    90              NOP  
00461F96    90              NOP  
00461F97    90              NOP  
00461F98    90              NOP  
00461F99    90              NOP  
00461F9A    90              NOP  
00461F9B    90              NOP  
00461F9C    90              NOP  
00461F9D    83E9 05         SUB ECX,5  
00461FA0    90              NOP  
00461FA1    90              NOP  
00461FA2    90              NOP  
00461FA3    90              NOP  
00461FA4    90              NOP  
00461FA5    90              NOP  
00461FA6    90              NOP  
00461FA7    90              NOP  
00461FA8    90              NOP  
00461FA9    90              NOP  
00461FAA    33DB            XOR EBX,EBX  
00461FAC    90              NOP  
00461FAD    90              NOP  
00461FAE    90              NOP  
00461FAF    90              NOP  
00461FB0    90              NOP  
00461FB1    90              NOP  
00461FB2    90              NOP  
00461FB3    90              NOP  
00461FB4    90              NOP  
00461FB5    90              NOP  
00461FB6    B8 9CBE0000     MOV EAX,0BE9C  
00461FBB    90              NOP  
00461FBC    90              NOP  
00461FBD    90              NOP  
00461FBE    90              NOP  
00461FBF    90              NOP  
00461FC0    90              NOP  
00461FC1    90              NOP  
00461FC2    90              NOP  
00461FC3    90              NOP  
00461FC4    90              NOP  
00461FC5    8BF9            MOV EDI,ECX  
00461FC7    90              NOP  
00461FC8    90              NOP  
00461FC9    90              NOP  
00461FCA    90              NOP  
00461FCB    90              NOP  
00461FCC    90              NOP  
00461FCD    90              NOP  
00461FCE    90              NOP  
00461FCF    90              NOP  
00461FD0    90              NOP  
00461FD1    2BF8            SUB EDI,EAX  
00461FD3    90              NOP  
00461FD4    90              NOP  
00461FD5    90              NOP  
00461FD6    90              NOP  
00461FD7    90              NOP  
00461FD8    90              NOP  
00461FD9    90              NOP  
00461FDA    90              NOP  
00461FDB    90              NOP  
00461FDC    90              NOP  
00461FDD    0FB607          MOVZX EAX,BYTE PTR DS:\[EDI\]  
00461FE0    90              NOP  
00461FE1    90              NOP  
00461FE2    90              NOP  
00461FE3    90              NOP  
00461FE4    90              NOP  
00461FE5    90              NOP  
00461FE6    90              NOP  
00461FE7    90              NOP  
00461FE8    90              NOP  
00461FE9    90              NOP  
00461FEA    03D8            ADD EBX,EAX  
00461FEC    90              NOP  
00461FED    90              NOP  
00461FEE    90              NOP  
00461FEF    90              NOP  
00461FF0    90              NOP  
00461FF1    90              NOP  
00461FF2    90              NOP  
00461FF3    90              NOP  
00461FF4    90              NOP  
00461FF5    90              NOP  
00461FF6    47              INC EDI  
00461FF7    90              NOP  
00461FF8    90              NOP  
00461FF9    90              NOP  
00461FFA    90              NOP  
00461FFB    90              NOP  
00461FFC    90              NOP  
00461FFD    90              NOP  
00461FFE    90              NOP  
00461FFF    90              NOP  
00462000    90              NOP  
00462001    3BF9            CMP EDI,ECX  
00462003    90              NOP  
00462004    90              NOP  
00462005    90              NOP  
00462006    90              NOP  
00462007    90              NOP  
00462008    90              NOP  
00462009    90              NOP  
0046200A    90              NOP  
0046200B    90              NOP  
0046200C    90              NOP  
0046200D  ^ 72 CE           JB SHORT 00461FDD  
0046200F    BF 00704400     MOV EDI,00447000  
00462014    B9 00BC0000     MOV ECX,0BC00  
00462019    90              NOP  
0046201A    90              NOP  
0046201B    90              NOP  
0046201C    90              NOP  
0046201D    90              NOP  
0046201E    90              NOP  
0046201F    90              NOP  
00462020    90              NOP  
00462021    90              NOP  
00462022    90              NOP  
00462023    0FB607          MOVZX EAX,BYTE PTR DS:\[EDI\]  
00462026    90              NOP  
00462027    90              NOP  
00462028    90              NOP  
00462029    90              NOP  
0046202A    90              NOP  
0046202B    90              NOP  
0046202C    90              NOP  
0046202D    90              NOP  
0046202E    90              NOP  
0046202F    90              NOP  
00462030    02DF            ADD BL,BH  
00462032    32DF            XOR BL,BH  
00462034    32C3            XOR AL,BL  
00462036    90              NOP  
00462037    90              NOP  
00462038    90              NOP  
00462039    90              NOP  
0046203A    90              NOP  
0046203B    90              NOP  
0046203C    90              NOP  
0046203D    90              NOP  
0046203E    90              NOP  
0046203F    90              NOP  
00462040    8807            MOV BYTE PTR DS:\[EDI\],AL  
00462042    90              NOP  
00462043    90              NOP  
00462044    90              NOP  
00462045    90              NOP  
00462046    90              NOP  
00462047    90              NOP  
00462048    90              NOP  
00462049    90              NOP  
0046204A    90              NOP  
0046204B    90              NOP  
0046204C    47              INC EDI  
0046204D    90              NOP  
0046204E    90              NOP  
0046204F    90              NOP  
00462050    90              NOP  
00462051    90              NOP  
00462052    90              NOP  
00462053    90              NOP  
00462054    90              NOP  
00462055    90              NOP  
00462056    90              NOP  
00462057    49              DEC ECX  
00462058    90              NOP  
00462059    90              NOP  
0046205A    90              NOP  
0046205B    90              NOP  
0046205C    90              NOP  
0046205D    90              NOP  
0046205E    90              NOP  
0046205F    90              NOP  
00462060    90              NOP  
00462061    90              NOP  
00462062  ^ 75 B5           JNZ SHORT 00462019  
00462064    E8 00000000     CALL 00462069  
00462069    59              POP ECX  
0046206A    2959 16         SUB DWORD PTR DS:\[ECX+16\],EBX  
0046206D    61              POPAD  
0046206E    60              PUSHAD  
0046206F    BE 00704400     MOV ESI,00447000  
00462074    8DBE 00A0FBFF   LEA EDI,DWORD PTR DS:\[ESI+FFFBA000\]  
0046207A    57              PUSH EDI  
0046207B    83CD FF         OR EBP,FFFFFFFF  
0046207E    68 ADE29F00     PUSH 9FE2AD                              ; 如果 CRC 错误则会乱跳。  
00462083    C3              RETN  
……  
进来后看看像什么?, upx.  
004528C0   /EB 10           JMP SHORT 004528D2  
004528C2   |90              NOP  
004528C3   |90              NOP  
004528C4   |90              NOP  
004528C5   |90              NOP  
004528C6   |90              NOP  
004528C7   |90              NOP  
004528C8   |8A06            MOV AL,BYTE PTR DS:\[ESI\]  
004528CA   |46              INC ESI  
004528CB   |8807            MOV BYTE PTR DS:\[EDI\],AL  
004528CD   |47              INC EDI  
004528CE   |01DB            ADD EBX,EBX  
004528D0   |75 07           JNZ SHORT 004528D9  
004528D2   \\8B1E            MOV EBX,DWORD PTR DS:\[ESI\]  
004528D4    83EE FC         SUB ESI,-4  
004528D7    11DB            ADC EBX,EBX  
004528D9  ^ 72 ED           JB SHORT 004528C8  
004528DB    B8 01000000     MOV EAX,1  
004528E0    01DB            ADD EBX,EBX  
004528E2    75 07           JNZ SHORT 004528EB  
004528E4    8B1E            MOV EBX,DWORD PTR DS:\[ESI\]  
004528E6    83EE FC         SUB ESI,-4  
004528E9    11DB            ADC EBX,EBX  
004528EB    11C0            ADC EAX,EAX  
004528ED    01DB            ADD EBX,EBX  
004528EF    73 0B           JNB SHORT 004528FC  
004528F1    75 19           JNZ SHORT 0045290C  
004528F3    8B1E            MOV EBX,DWORD PTR DS:\[ESI\]  
004528F5    83EE FC         SUB ESI,-4  
004528F8    11DB            ADC EBX,EBX  
004528FA    72 10           JB SHORT 0045290C  
004528FC    48              DEC EAX  
004528FD    01DB            ADD EBX,EBX  
004528FF    75 07           JNZ SHORT 00452908  
00452901    8B1E            MOV EBX,DWORD PTR DS:\[ESI\]  
00452903    83EE FC         SUB ESI,-4  
00452906    11DB            ADC EBX,EBX  
00452908    11C0            ADC EAX,EAX  
0045290A  ^ EB D4           JMP SHORT 004528E0  
0045290C    31C9            XOR ECX,ECX  
0045290E    83E8 03         SUB EAX,3  
00452911    72 11           JB SHORT 00452924  
00452913    C1E0 08         SHL EAX,8  
00452916    8A06            MOV AL,BYTE PTR DS:\[ESI\]  
00452918    46              INC ESI  
00452919    83F0 FF         XOR EAX,FFFFFFFF  
0045291C    74 78           JE SHORT 00452996  
0045291E    D1F8            SAR EAX,1  
00452920    89C5            MOV EBP,EAX  
00452922    EB 0B           JMP SHORT 0045292F  
00452924    01DB            ADD EBX,EBX  
00452926    75 07           JNZ SHORT 0045292F  
00452928    8B1E            MOV EBX,DWORD PTR DS:\[ESI\]  
0045292A    83EE FC         SUB ESI,-4  
0045292D    11DB            ADC EBX,EBX  
0045292F    11C9            ADC ECX,ECX  
00452931    01DB            ADD EBX,EBX  
00452933    75 07           JNZ SHORT 0045293C  
00452935    8B1E            MOV EBX,DWORD PTR DS:\[ESI\]  
00452937    83EE FC         SUB ESI,-4  
0045293A    11DB            ADC EBX,EBX  
0045293C    11C9            ADC ECX,ECX  
0045293E    75 20           JNZ SHORT 00452960  
00452940    41              INC ECX  
00452941    01DB            ADD EBX,EBX  
00452943    75 07           JNZ SHORT 0045294C  
00452945    8B1E            MOV EBX,DWORD PTR DS:\[ESI\]  
00452947    83EE FC         SUB ESI,-4  
0045294A    11DB            ADC EBX,EBX  
0045294C    11C9            ADC ECX,ECX  
0045294E    01DB            ADD EBX,EBX  
00452950  ^ 73 EF           JNB SHORT 00452941  
00452952    75 09           JNZ SHORT 0045295D  
00452954    8B1E            MOV EBX,DWORD PTR DS:\[ESI\]  
00452956    83EE FC         SUB ESI,-4  
00452959    11DB            ADC EBX,EBX  
0045295B  ^ 73 E4           JNB SHORT 00452941  
0045295D    83C1 02         ADD ECX,2  
00452960    81FD 00FBFFFF   CMP EBP,-500  
00452966    83D1 01         ADC ECX,1  
00452969    8D142F          LEA EDX,DWORD PTR DS:\[EDI+EBP\]  
0045296C    83FD FC         CMP EBP,-4  
0045296F    76 0F           JBE SHORT 00452980  
00452971    8A02            MOV AL,BYTE PTR DS:\[EDX\]  
00452973    42              INC EDX  
00452974    8807            MOV BYTE PTR DS:\[EDI\],AL  
00452976    47              INC EDI  
00452977    49              DEC ECX  
00452978  ^ 75 F7           JNZ SHORT 00452971  
0045297A  ^ E9 4FFFFFFF     JMP 004528CE  
0045297F    90              NOP  
00452980    8B02            MOV EAX,DWORD PTR DS:\[EDX\]  
00452982    83C2 04         ADD EDX,4  
00452985    8907            MOV DWORD PTR DS:\[EDI\],EAX  
00452987    83C7 04         ADD EDI,4  
0045298A    83E9 04         SUB ECX,4  
0045298D  ^ 77 F1           JA SHORT 00452980  
0045298F    01CF            ADD EDI,ECX  
00452991  ^ E9 38FFFFFF     JMP 004528CE  
00452996    5E              POP ESI  
00452997    89F7            MOV EDI,ESI  
00452999    B9 D5160000     MOV ECX,16D5  
0045299E    8A07            MOV AL,BYTE PTR DS:\[EDI\]  
004529A0    47              INC EDI  
004529A1    2C E8           SUB AL,0E8  
004529A3    3C 01           CMP AL,1  
004529A5  ^ 77 F7           JA SHORT 0045299E  
004529A7    803F 01         CMP BYTE PTR DS:\[EDI\],1  
004529AA  ^ 75 F2           JNZ SHORT 0045299E  
004529AC    8B07            MOV EAX,DWORD PTR DS:\[EDI\]  
004529AE    8A5F 04         MOV BL,BYTE PTR DS:\[EDI+4\]  
004529B1    66:C1E8 08      SHR AX,8  
004529B5    C1C0 10         ROL EAX,10  
004529B8    86C4            XCHG AH,AL  
004529BA    29F8            SUB EAX,EDI  
004529BC    80EB E8         SUB BL,0E8  
004529BF    01F0            ADD EAX,ESI  
004529C1    8907            MOV DWORD PTR DS:\[EDI\],EAX  
004529C3    83C7 05         ADD EDI,5  
004529C6    89D8            MOV EAX,EBX  
004529C8  ^ E2 D9           LOOPD SHORT 004529A3  
004529CA    8DBE 00000500   LEA EDI,DWORD PTR DS:\[ESI+50000\]  
004529D0    8B07            MOV EAX,DWORD PTR DS:\[EDI\]  
004529D2    09C0            OR EAX,EAX  
004529D4    74 3C           JE SHORT 00452A12  
004529D6    8B5F 04         MOV EBX,DWORD PTR DS:\[EDI+4\]  
004529D9    8D8430 B0490500 LEA EAX,DWORD PTR DS:\[EAX+ESI+549B0\]  
004529E0    01F3            ADD EBX,ESI  
004529E2    50              PUSH EAX  
004529E3    83C7 08         ADD EDI,8  
004529E6    FF96 3C4A0500   CALL DWORD PTR DS:\[ESI+54A3C\]  
004529EC    95              XCHG EAX,EBP  
004529ED    8A07            MOV AL,BYTE PTR DS:\[EDI\]  
004529EF    47              INC EDI  
004529F0    08C0            OR AL,AL  
004529F2  ^ 74 DC           JE SHORT 004529D0  
004529F4    89F9            MOV ECX,EDI  
004529F6    57              PUSH EDI  
004529F7    48              DEC EAX  
004529F8    F2:AE           REPNE SCAS BYTE PTR ES:\[EDI\]  
004529FA    55              PUSH EBP  
004529FB    FF96 404A0500   CALL DWORD PTR DS:\[ESI+54A40\]  
00452A01    09C0            OR EAX,EAX  
00452A03    74 07           JE SHORT 00452A0C  
00452A05    8903            MOV DWORD PTR DS:\[EBX\],EAX  
00452A07    83C3 04         ADD EBX,4  
00452A0A  ^ EB E1           JMP SHORT 004529ED  
00452A0C    FF96 444A0500   CALL DWORD PTR DS:\[ESI+54A44\]  
00452A12    61              POPAD  
00452A13  - E9 3E13FCFF     JMP 00413D56  
到些也就分析完了，失败了 N 次，一个下午才分析完，现在好饿了，回家做饭去.  
说明一下:在第一次去”垃圾”之前自己随便改一个地方，然后改回来，这样后面修复代码就可以直接 alt+<-,感觉壳并非想象中那么难的说:-).简直是烂用 RDTSC  
  
Greetz:  
 Fly.Jingulong,yock,tDasm.David.hexer,hmimys,ahao.UFO(brother).alan(sister).all of my friends and you!  
  
By loveboom\[DFCG\]\[FCG\]\[US\]  
Email:loveboom#163.com  
Date:2/25/2005 8:19:00 PM  
  

哪位大虾说说这段的作用：-有 ANTI 的作用吗？  
00456159    90              NOP  
0045615A    0F31            RDTSC  
0045615C    50              PUSH EAX  
0045615D    0F31            RDTSC  
0045615F    E8 0A000000     CALL \[MSLRH\].0045616E  
00456164    E8 EB0C0000     CALL \[MSLRH\].00456E54  
00456169    E8 F6FFFFFF     CALL \[MSLRH\].00456164  
0045616E    E8 F2FFFFFF     CALL \[MSLRH\].00456165  
00456173    83C4 08         ADD ESP,8  
00456176    2B0424          SUB EAX,DWORD PTR SS:\[ESP\]  
00456179    74 04           JE SHORT \[MSLRH\].0045617F  
0045617B    75 02           JNZ SHORT \[MSLRH\].0045617F  
0045617D    EB 02           JMP SHORT \[MSLRH\].00456181  
0045617F    EB 01           JMP SHORT \[MSLRH\].00456182  
00456181    8183 C404E80A 0>ADD DWORD PTR DS:\[EBX+AE804C4\],E8000000  
0045618B    EB 0C           JMP SHORT \[MSLRH\].00456199  
0045618D    0000            ADD BYTE PTR DS:\[EAX\],AL  
0045618F    E8 F6FFFFFF     CALL \[MSLRH\].0045618A  
00456194    E8 F2FFFFFF     CALL \[MSLRH\].0045618B  
00456199    83C4 08         ADD ESP,8  
0045619C    3D FF0F0000     CMP EAX,0FFF  
004561A1    EB 01           JMP SHORT \[MSLRH\].004561A4  
004561A3    68 EB02CD20     PUSH 20CD02EB  
004561A8    EB 01           JMP SHORT \[MSLRH\].004561AB  
004561AA    E8 761BEB01     CALL 02307D25  
004561AF    68 EB02CD20     PUSH 20CD02EB  
004561B4    EB 01           JMP SHORT \[MSLRH\].004561B7  
004561B6    E8 CC66B8FE     CALL FEFDC887  
004561BB    007404 75       ADD BYTE PTR SS:\[ESP+EAX+75\],DH\[B\] 

RDTSC  
PUSH EAX  
RDTSC  
SUB EAX,DWORD PTR SS:\[ESP\]  
CMP EAX,0FFF  
004561AA    E8 761BEB01     CALL 02307D25  
这里去掉 E8 的花指令就可以看到了 761B,这里不跳就进了 int3  
RDTSC 的说明，自己去看相关资料 

看了 loveboom 文章学到不少东西，谢谢！  
  
有一段原文没记载，我补充一下：  
  
00456AA7    .  64:FF35 00000000     push dword ptr fs:\[0\]  
00456AAE    .  64:8925 00000000     mov dword ptr fs:\[0\],esp  
……  
0045745C    .  33C0                 xor eax,eax  
0045745E    .  0FB600               movzx eax,byte ptr ds:\[eax\]  //读零地址异常  
  
  
异常后来到：  
  
//这段代码是检测 DRx 寄存器的值是否为 0， 即检测是否用了 F4 或 G 等用到调试寄存器的命令。  
  
004587B6    .  8B4424 0C       mov eax,dword ptr ss:\[esp+C\]  
004587BA    .  33C9            xor ecx,ecx                       
004587BC    .  3348 04         xor ecx,dword ptr ds:\[eax+4\]  //如果 DR0 不为 0，则 ecx 也不为 0  
004587BF    .  3348 08         xor ecx,dword ptr ds:\[eax+8\]  //同上  
004587C2    .  3348 0C         xor ecx,dword ptr ds:\[eax+C\]  //同上  
004587C5    .  3348 10         xor ecx,dword ptr ds:\[eax+10\] //同上  
004587C8    .  8B6424 08       mov esp,dword ptr ss:\[esp+8\]  
004587CC    .  64:8F05 0000000>pop dword ptr fs:\[0\]  
004587D3    .  83C4 04         add esp,4  
  
……  
  
00459191    51              PUSH ECX                            //注意 ECX 的值  
00459192    33C9            XOR ECX,ECX  
00459194    E8 00000000     CALL 00459199  
00459199    5F              POP EDI  
0045919A    81C7 C4090000   ADD EDI,9C4  
004591A0    5A              POP EDX                              //也就是说在 004587B6 之前用过调试断点，此处 EDX 就不为 0  
                                                                 //那样 459b5d 处的代码解密就不正确了  
004591A1    83C2 15         ADD EDX,15                           //密钥放到 edx 中，正确的值应为 15  
  
004591A4    0FB60439        MOVZX EAX,BYTE PTR DS:\[ECX+EDI\]          ; 把值传到 eax 中(从 459b5d 处开始)  
004591A8    33C2            XOR EAX,EDX                              ; 取出的值 xor 15  
004591AA    880439          MOV BYTE PTR DS:\[ECX+EDI\],AL               
004591AD    41              INC ECX  
004591AE    81F9 93000000   CMP ECX,93                                 
004591B4  ^ 72 EE           JB SHORT 004591A4 

 4dd466d0e1e0cc123afbe504da2462ca