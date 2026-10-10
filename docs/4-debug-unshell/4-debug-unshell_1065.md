

# Telock 修改版之初尝驱动

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-481.htm`](https://www.pediy.com/kssd/pediy07/pediy7-481.htm)

【脱文作者】 simonzh2000\[US\]  
【使用工具】 Ollydbg1.10, ImportREC  
【破解平台】 Win2000SP4 English  
【软件名称】 xIkUg's PE Protector 0.31  
【软件简介】 xIkUg 兄弟的壳, 一开始以为是用自己加的壳,  
  
最后才发现是用我们 US 成员 heXer, forgot 修改的 TELock.   
这个壳虽然比较老了,但还是有点难度,  两位兄弟又加了驱动防止 INT3 断点.  
最近我正好在学驱动, 就仔细看了看, 有了这篇文章.   
其实到最后, 我也没搞懂 Saga.sys 倒底干了些什么, 只是找到了一个避开驱动的方法.  
希望我下次能自如对付驱动.  
  
  
【加壳方式】 TELock heXer, forgot 修改版  
  
【作者声明】 本笔记只用于学习交流, 初学 Crack，只是感兴趣技术，没有其他目的, 如有不妥之处, 请 xIkUg 兄弟谅解.  
            期待兄弟新的更强的壳.  
  
压缩包内有全文.  
  
  
OD  不忽略异常.  
  
0049A862 >^\\E9 99D7FFFF     JMP xPEP.00498000    
  
...  
  
// 第一个异常  
0049A68C    E8 06000000      CALL xPEP.0049A697                 ; F4 到这里, 跳过花指令, F7  
  
0049A691    8B6424 08        MOV ESP,DWORD PTR SS:\[ESP+8\]       ; SEH handler, 下断, SHIFT+F9  
0049A695    EB 0D            JMP SHORT xPEP.0049A6A4  
  
0049A697    33FF             XOR EDI,EDI  
0049A699    64:FF37          PUSH DWORD PTR FS:\[EDI\]  
0049A69C    64:8927          MOV DWORD PTR FS:\[EDI\],ESP  
0049A69F    F1               INT1                               ; F9 到这里是单步异常  
0049A6A0    F7F7             DIV EDI                            ; F7 到这里是除零异常, 效果一样  
  
0049A6AB    BB 00000000      MOV EBX,0                          ; 恢复 SEH  
0049A6B0    64:8F03          POP DWORD PTR FS:\[EBX\]  
0049A6B3    5B               POP EBX  
  
  
// 第二个异常  
0049A6BC    60               PUSHAD  
0049A6BD    E8 06000000      CALL xPEP.0049A6C8                 ; 下面几句 F9 , 不能 F7  
  
0049A6C2    8B6424 08        MOV ESP,DWORD PTR SS:\[ESP+8\]       ; SEH handler, 下断, SHIFT+F9  
0049A6C6    EB 1A            JMP SHORT xPEP.0049A6E2  
  
0049A6C8    64:67:FF36 0000  PUSH DWORD PTR FS:\[0\]  
0049A6CE    64:67:8926 0000  MOV DWORD PTR FS:\[0\],ESP  
0049A6D4    9C               PUSHFD  
0049A6D5    810C24 00010000  OR DWORD PTR SS:\[ESP\],100          ; 设置单步异常  
0049A6DC    9D               POPFD  
0049A6DD    F8               CLC  
0049A6DE  ^ 73 DC            JNB SHORT xPEP.0049A6BC            ; 这里单步异常  
  
0049A6E2    64:67:8F06 0000  POP DWORD PTR FS:\[0\]               ; 恢复 SEH  
0049A6E8    58               POP EAX  
0049A6E9    61               POPAD  
  
...  
  
// 第三个异常  
004980CC    E8 00000000      CALL xPEP.004980D1  
004980D1    5D               POP EBP                          ; EBP=4980D1  
004980D2    8D85 1E010000    LEA EAX,DWORD PTR SS:\[EBP+11E\]   ; 4981EF  
004980D8    50               PUSH EAX  
004980D9    33C0             XOR EAX,EAX  
004980DB    64:FF30          PUSH DWORD PTR FS:\[EAX\]  
004980DE    64:8920          MOV DWORD PTR FS:\[EAX\],ESP  
004980E1    CC               INT3                             ; Int3 异常  
004980E2    90               NOP                              ; 设置了 4 个硬件断点后, 从这里继续, F9 走  
  
  
// 经过一些花指令 和 4 个断点.  断点处 F9 走是单步异常, 如果 F7 走, 没有异常  
// 注意看 \[498242\], 单步异常的次数  
  
00498112    F9              STC                               ; 第 4 个异常  
00498113    90              NOP  
  
00498141    F8              CLC                               ; 第 5 个异常  
00498142    90              NOP  
  
00498170    FD              STD                               ; 第 6 个异常  
00498171    90              NOP  
  
0049819F    FC              CLD                               ; 第 7 个异常                                
004981A0    90              NOP  
  
  
  
// 来到第 8 个异常  
004981CE    33DB            XOR EBX,EBX                         
004981D0    F7F3            DIV EBX                           ; 除 0 异常  
004981D2    64:8F05 0000000>POP DWORD PTR FS:\[0\]              ; 清除断点后, 这里继续  
004981D9    83C4 04         ADD ESP,4  
  
004981DC    66:BE 4040      MOV SI,4040  
004981E0    66:BF 0D0A      MOV DI,0A0D  
004981E4    8A85 71010000   MOV AL,BYTE PTR SS:\[EBP+171\]      ; \[498242\] 单步异常的次数=4  
004981EA    E9 A4000000     JMP xPEP.00498293  
  
  
// SEH Handler 4981EF  
004981EF    8B4424 04        MOV EAX,DWORD PTR SS:\[ESP+4\]     ; pEXCEPTION\_RECORD  
004981F3    8B4C24 0C        MOV ECX,DWORD PTR SS:\[ESP+C\]     ; pCONTEXT  
004981F7    FF81 B8000000    INC DWORD PTR DS:\[ECX+B8\]        ; regEIP+1  
004981FD    8B00             MOV EAX,DWORD PTR DS:\[EAX\]       ; ExceptionCode  
004981FF    3D 940000C0      CMP EAX,C0000094                 ; 非法除 0?  
00498204    75 24            JNZ SHORT xPEP.0049822A  
   
00498206    FF81 B8000000    INC DWORD PTR DS:\[ECX+B8\]        ; 非法除 0, regEIP 再加 1        
0049820C    33C0             XOR EAX,EAX  
0049820E    2141 04          AND DWORD PTR DS:\[ECX+4\],EAX     ; 清断点  
00498211    2141 08          AND DWORD PTR DS:\[ECX+8\],EAX  
00498214    2141 0C          AND DWORD PTR DS:\[ECX+C\],EAX  
00498217    2141 10          AND DWORD PTR DS:\[ECX+10\],EAX  
0049821A    8161 14 F00FFFFF AND DWORD PTR DS:\[ECX+14\],FFFF0FF0 ;Dr6  
00498221    8161 18 00DC0000 AND DWORD PTR DS:\[ECX+18\],0DC00    ;Dr7  
00498228    EB 68            JMP SHORT xPEP.00498292          ; 异常处理完毕  
  
0049822A    3D 04000080      CMP EAX,80000004                 ; 单步调试状态  
0049822F    74 0C            JE SHORT xPEP.0049823D  
  
00498231    3D 03000080      CMP EAX,80000003                 ; Int3 异常  
00498236    74 12            JE SHORT xPEP.0049824A  
  
00498238    6A 01            PUSH 1                           ; 其他异常  
0049823A    58               POP EAX                          ; EAX=1  
0049823B    EB 55            JMP SHORT xPEP.00498292          ; 不处理  
  
0049823D    E8 01000000      CALL xPEP.00498243  
00498242    00               00                               ; 单步异常发生次数  
00498243    58               POP EAX                          ; EAX=498242                                    
00498244    FE00             INC BYTE PTR DS:\[EAX\]            ; 计数器加 1  
00498246    2BC0             SUB EAX,EAX  
00498248    EB 48            JMP SHORT xPEP.00498292          ; 异常处理完毕  
  
0049824A    8B81 B4000000    MOV EAX,DWORD PTR DS:\[ECX+B4\]    ; Int3 异常, regEBP  
00498250    05 CE000000      ADD EAX,0CE  
00498255    8941 04          MOV DWORD PTR DS:\[ECX+4\],EAX     ; Dr0=49819F  
  
00498258    8B81 B4000000    MOV EAX,DWORD PTR DS:\[ECX+B4\]  
0049825E    05 9F000000      ADD EAX,9F  
00498263    8941 08          MOV DWORD PTR DS:\[ECX+8\],EAX     ; Dr1=498170  
  
00498266    8B81 B4000000    MOV EAX,DWORD PTR DS:\[ECX+B4\]  
0049826C    05 70000000      ADD EAX,70  
00498271    8941 0C          MOV DWORD PTR DS:\[ECX+C\],EAX     ; Dr2=498141  
  
00498274    8B81 B4000000    MOV EAX,DWORD PTR DS:\[ECX+B4\]  
0049827A    05 41000000      ADD EAX,41  
0049827F    8941 10          MOV DWORD PTR DS:\[ECX+10\],EAX    ; Dr3=498112  
  
00498282    33C0             XOR EAX,EAX  
00498284    8161 14 F00FFFFF AND DWORD PTR DS:\[ECX+14\],FFFF0FF0 ; Dr6  
0049828B    C741 18 55010000 MOV DWORD PTR DS:\[ECX+18\],155      ; Dr7  
00498292    C3               RETN  
  
  
  
// 单步异常次数 \[498242\]=0, 校验代码时用  
00498293    2C 04           SUB AL,4  
00498295    66:03FE         ADD DI,SI  
00498298    66:81C6 0706    ADD SI,607  
0049829D    8885 71010000   MOV BYTE PTR SS:\[EBP+171\],AL        ; \[498242\] 单步异常次数=0  
  
  
// 求 13 个 API address  
004982A3    8B95 E9270000   MOV EDX,DWORD PTR SS:\[EBP+27E9\]     ; kernel32.GetModuleHandleA, 仅有的两个 API 之一  
004982A9    81E2 0000FFFF   AND EDX,FFFF0000  
  
004982AF    8BC4            MOV EAX,ESP                         ; 防 ESP 定律?  
004982B1    33E4            XOR ESP,ESP  
004982B3    8BE0            MOV ESP,EAX  
  
004982B5    66:813A 4D5A    CMP WORD PTR DS:\[EDX\],5A4D          ; "MZ"  
004982BA   /74 08           JE SHORT xPEP.004982C4              ;  EDX = Kernel32.dll 的 base  
004982BC    81C2 0000FFFF   ADD EDX,FFFF0000                    ;  -1000h  
004982C2  ^ EB F1           JMP SHORT xPEP.004982B5  
  
004982C4    8B42 3C         MOV EAX,DWORD PTR DS:\[EDX+3C\]       ; PE 头  
004982C7    3D 00020000     CMP EAX,200  
004982CC  ^ 77 E7           JA SHORT xPEP.004982B5              ; >200h 不对  
  
004982CE    813C10 50450000 CMP DWORD PTR DS:\[EAX+EDX\],4550     ; "PE"  
004982D5  ^ 75 DE           JNZ SHORT xPEP.004982B5  
  
004982D7    8B4410 78       MOV EAX,DWORD PTR DS:\[EAX+EDX+78\]   ; Export Table  
  
004982DB    8B5C10 1C       MOV EBX,DWORD PTR DS:\[EAX+EDX+1C\]   ; AddressOfFunctions  
004982DF    53              PUSH EBX  
004982E0    8B5C10 24       MOV EBX,DWORD PTR DS:\[EAX+EDX+24\]   ; AddressOfNameOrdinals  
004982E4    8B7410 20       MOV ESI,DWORD PTR DS:\[EAX+EDX+20\]   ; AddressOfNames  
004982E8    8B4C10 18       MOV ECX,DWORD PTR DS:\[EAX+EDX+18\]   ; NumberOfNames  
004982EC    8B4410 0C       MOV EAX,DWORD PTR DS:\[EAX+EDX+C\]    ; Name  
  
004982F0    03C2            ADD EAX,EDX  
004982F2    8B00            MOV EAX,DWORD PTR DS:\[EAX\]          ; "KERN"  
004982F4    25 5F5F5F5F     AND EAX,5F5F5F5F  
004982F9    3D 4B45524E     CMP EAX,4E52454B  
004982FE    74 07           JE SHORT xPEP.00498307  
  
00498300    68 EBFE0000     PUSH 0FEEB                          ; DLL 名字不对, 出错  
00498305    FFE4            JMP ESP  
  
00498307    8D7416 FC       LEA ESI,DWORD PTR DS:\[ESI+EDX-4\]  
0049830B    8D5C13 FE       LEA EBX,DWORD PTR DS:\[EBX+EDX-2\]  
0049830F    6A 0D           PUSH 0D                             ; 一共有 0Dh 个函数   
00498311    5F              POP EDI  
  
00498312    83C6 04         ADD ESI,4                           ; AddressOfNames 每项 4 字节  
00498315    43              INC EBX  
00498316    43              INC EBX                             ; AddressOfNameOrdinals 每项 2 字节  
00498317    49              DEC ECX                     
00498318  ^ 7C E6           JL SHORT xPEP.00498300              ; 所有函数都找遍了,  出错  
  
0049831A    8B06            MOV EAX,DWORD PTR DS:\[ESI\]  
0049831C    03C2            ADD EAX,EDX  
0049831E    8138 4C6F6164   CMP DWORD PTR DS:\[EAX\],64616F4C     ; LoadLibraryA  
00498324    75 3A           JNZ SHORT xPEP.00498360  
00498326    8178 04 4C69627>CMP DWORD PTR DS:\[EAX+4\],7262694C  
0049832D    75 31           JNZ SHORT xPEP.00498360  
0049832F    8178 08 6172794>CMP DWORD PTR DS:\[EAX+8\],41797261  
00498336    75 28           JNZ SHORT xPEP.00498360  
00498338    68 AE040000     PUSH 4AE                            ; 壳表示函数的代号  
  
0049833D    58              POP EAX  
0049833E    58              POP EAX                             ; AddressOfFunctions   
0049833F    50              PUSH EAX  
00498340    83EC 04         SUB ESP,4  
00498343    53              PUSH EBX                            ; AddressOfNameOrdinals  
00498344    03C2            ADD EAX,EDX                         ; 7C570000+AddressOfFunctions  
00498346    0FB71B          MOVZX EBX,WORD PTR DS:\[EBX\]  
00498349    8B1C98          MOV EBX,DWORD PTR DS:\[EAX+EBX\*4\]  
0049834C    03DA            ADD EBX,EDX                         ; EBX = API Address  
0049834E    8B4424 04       MOV EAX,DWORD PTR SS:\[ESP+4\]        ; 壳表示函数的代号  
00498352    891C28          MOV DWORD PTR DS:\[EAX+EBP\],EBX      ; EBP=4980D1, \[49857B\] 区域  
00498355    5B              POP EBX  
00498356    58              POP EAX  
00498357    4F              DEC EDI  
00498358  ^ 75 B8           JNZ SHORT xPEP.00498312             ; 还有, 继续下一个函数  
  
0049835A   /0F84 E5010000   JE xPEP.00498545                    ; OK, 13 个找齐了   
  
  
00498360    8138 45786974   CMP DWORD PTR DS:\[EAX\],74697845     ; ExitProcess  
00498366    75 19           JNZ SHORT xPEP.00498381  
00498368    8178 04 50726F6>CMP DWORD PTR DS:\[EAX+4\],636F7250  
0049836F    75 10           JNZ SHORT xPEP.00498381  
00498371    8178 08 6573730>CMP DWORD PTR DS:\[EAX+8\],737365  
00498378    75 07           JNZ SHORT xPEP.00498381  
0049837A    68 B2040000     PUSH 4B2  
0049837F  ^ EB BC           JMP SHORT xPEP.0049833D  
  
00498381    8138 56697274   CMP DWORD PTR DS:\[EAX\],74726956     ; VirtualAlloc  
00498387    75 1F           JNZ SHORT xPEP.004983A8  
00498389    8178 04 75616C4>CMP DWORD PTR DS:\[EAX+4\],416C6175  
00498390    75 16           JNZ SHORT xPEP.004983A8  
00498392    8178 08 6C6C6F6>CMP DWORD PTR DS:\[EAX+8\],636F6C6C  
00498399    75 0D           JNZ SHORT xPEP.004983A8  
0049839B    8078 0C 00      CMP BYTE PTR DS:\[EAX+C\],0  
0049839F    75 07           JNZ SHORT xPEP.004983A8  
004983A1    68 B6040000     PUSH 4B6  
004983A6  ^ EB 95           JMP SHORT xPEP.0049833D  
  
004983A8    8138 56697274   CMP DWORD PTR DS:\[EAX\],74726956     ; VirtualFree  
004983AE    75 1C           JNZ SHORT xPEP.004983CC  
004983B0    8178 04 75616C4>CMP DWORD PTR DS:\[EAX+4\],466C6175  
004983B7    75 13           JNZ SHORT xPEP.004983CC  
004983B9    8178 08 7265650>CMP DWORD PTR DS:\[EAX+8\],656572  
004983C0    75 0A           JNZ SHORT xPEP.004983CC  
004983C2    68 BA040000     PUSH 4BA  
004983C7  ^ E9 71FFFFFF     JMP xPEP.0049833D  
  
004983CC    8138 43726561   CMP DWORD PTR DS:\[EAX\],61657243      ; CreateMutexA  
004983D2    75 1C           JNZ SHORT xPEP.004983F0  
004983D4    8178 04 74654D7>CMP DWORD PTR DS:\[EAX+4\],754D6574  
004983DB    75 13           JNZ SHORT xPEP.004983F0  
004983DD    8178 08 7465784>CMP DWORD PTR DS:\[EAX+8\],41786574  
004983E4    75 0A           JNZ SHORT xPEP.004983F0  
004983E6    68 BE040000     PUSH 4BE  
004983EB  ^ E9 4DFFFFFF     JMP xPEP.0049833D  
  
004983F0    8138 4F70656E   CMP DWORD PTR DS:\[EAX\],6E65704F     ; OpenProcess  
004983F6    75 1C           JNZ SHORT xPEP.00498414  
004983F8    8178 04 50726F6>CMP DWORD PTR DS:\[EAX+4\],636F7250  
004983FF    75 13           JNZ SHORT xPEP.00498414  
00498401    8178 08 6573730>CMP DWORD PTR DS:\[EAX+8\],737365  
00498408    75 0A           JNZ SHORT xPEP.00498414  
0049840A    68 C6040000     PUSH 4C6  
0049840F  ^ E9 29FFFFFF     JMP xPEP.0049833D  
  
00498414    8138 47657443   CMP DWORD PTR DS:\[EAX\],43746547     ; GetCurrentProcessId  
0049841A    75 2E           JNZ SHORT xPEP.0049844A  
0049841C    8178 04 7572726>CMP DWORD PTR DS:\[EAX+4\],65727275  
00498423    75 25           JNZ SHORT xPEP.0049844A  
00498425    8178 08 6E74507>CMP DWORD PTR DS:\[EAX+8\],7250746E  
0049842C    75 1C           JNZ SHORT xPEP.0049844A  
0049842E    8178 0C 6F63657>CMP DWORD PTR DS:\[EAX+C\],7365636F  
00498435    75 13           JNZ SHORT xPEP.0049844A  
00498437    8178 10 7349640>CMP DWORD PTR DS:\[EAX+10\],644973  
0049843E    75 0A           JNZ SHORT xPEP.0049844A  
00498440    68 C2040000     PUSH 4C2  
00498445  ^ E9 F3FEFFFF     JMP xPEP.0049833D  
  
0049844A    8138 56697274   CMP DWORD PTR DS:\[EAX\],74726956    ; VirtualProtectEx  
00498450    75 25           JNZ SHORT xPEP.00498477  
00498452    8178 04 75616C5>CMP DWORD PTR DS:\[EAX+4\],506C6175  
00498459    75 1C           JNZ SHORT xPEP.00498477  
0049845B    8178 08 726F746>CMP DWORD PTR DS:\[EAX+8\],65746F72  
00498462    75 13           JNZ SHORT xPEP.00498477  
00498464    8178 0C 6374457>CMP DWORD PTR DS:\[EAX+C\],78457463  
0049846B    75 0A           JNZ SHORT xPEP.00498477  
0049846D    68 CA040000     PUSH 4CA  
00498472  ^ E9 C6FEFFFF     JMP xPEP.0049833D  
  
00498477    8138 43726561   CMP DWORD PTR DS:\[EAX\],61657243      ; CreateFileA  
0049847D    75 1C           JNZ SHORT xPEP.0049849B  
0049847F    8178 04 7465466>CMP DWORD PTR DS:\[EAX+4\],69466574  
00498486    75 13           JNZ SHORT xPEP.0049849B  
00498488    8178 08 6C65410>CMP DWORD PTR DS:\[EAX+8\],xPEP.0041656C  
0049848F    75 0A           JNZ SHORT xPEP.0049849B  
00498491    68 D6040000     PUSH 4D6  
00498496  ^ E9 A2FEFFFF     JMP xPEP.0049833D  
  
0049849B    8138 436C6F73   CMP DWORD PTR DS:\[EAX\],736F6C43      ; CloseHandle  
004984A1    75 1C           JNZ SHORT xPEP.004984BF  
004984A3    8178 04 6548616>CMP DWORD PTR DS:\[EAX+4\],6E614865  
004984AA    75 13           JNZ SHORT xPEP.004984BF  
004984AC    8178 08 646C650>CMP DWORD PTR DS:\[EAX+8\],656C64  
004984B3    75 0A           JNZ SHORT xPEP.004984BF  
004984B5    68 CE040000     PUSH 4CE  
004984BA  ^ E9 7EFEFFFF     JMP xPEP.0049833D  
  
004984BF    8138 52656164   CMP DWORD PTR DS:\[EAX\],64616552      ; ReadFile  
004984C5    75 19           JNZ SHORT xPEP.004984E0  
004984C7    8178 04 46696C6>CMP DWORD PTR DS:\[EAX+4\],656C6946  
004984CE    75 10           JNZ SHORT xPEP.004984E0  
004984D0    8078 08 00      CMP BYTE PTR DS:\[EAX+8\],0  
004984D4    75 0A           JNZ SHORT xPEP.004984E0  
004984D6    68 D2040000     PUSH 4D2  
004984DB  ^ E9 5DFEFFFF     JMP xPEP.0049833D  
  
004984E0    8138 4765744D   CMP DWORD PTR DS:\[EAX\],4D746547      ; GetModuleFileNameA  
004984E6    75 2D           JNZ SHORT xPEP.00498515  
004984E8    8178 04 6F64756>CMP DWORD PTR DS:\[EAX+4\],6C75646F  
004984EF    75 24           JNZ SHORT xPEP.00498515  
004984F1    8178 08 6546696>CMP DWORD PTR DS:\[EAX+8\],6C694665  
004984F8    75 1B           JNZ SHORT xPEP.00498515  
004984FA    8178 0C 654E616>CMP DWORD PTR DS:\[EAX+C\],6D614E65  
00498501    75 12           JNZ SHORT xPEP.00498515  
00498503    66:8178 10 6541 CMP WORD PTR DS:\[EAX+10\],4165  
00498509    75 0A           JNZ SHORT xPEP.00498515  
0049850B    68 DA040000     PUSH 4DA  
00498510  ^ E9 28FEFFFF     JMP xPEP.0049833D  
  
00498515    8138 47657450   CMP DWORD PTR DS:\[EAX\],50746547     ; GetProcAddress  
0049851B  ^ 0F85 F1FDFFFF   JNZ xPEP.00498312  
00498521    8178 04 726F634>CMP DWORD PTR DS:\[EAX+4\],41636F72  
00498528  ^ 0F85 E4FDFFFF   JNZ xPEP.00498312  
0049852E    8178 08 6464726>CMP DWORD PTR DS:\[EAX+8\],65726464  
00498535  ^ 0F85 D7FDFFFF   JNZ xPEP.00498312  
0049853B    68 AA040000     PUSH 4AA  
00498540  ^ E9 F8FDFFFF     JMP xPEP.0049833D  
  
  
  
  
  
// 第 9 个异常  
004987C1    60              PUSHAD  
004987C2    E8 09000000     CALL xPEP.004987D0  
  
004987C7    8B6424 08       MOV ESP,DWORD PTR SS:\[ESP+8\]    ; SEH Handler  
004987CB    EB 01           JMP SHORT xPEP.004987CE  
004987CE   /EB 1B           JMP SHORT xPEP.004987EB  
  
004987D0    2BC9            SUB ECX,ECX  
004987D6    64:FF31         PUSH DWORD PTR FS:\[ECX\]  
  
004987DD    64:8921         MOV DWORD PTR FS:\[ECX\],ESP  
004987E0    41              INC ECX  
004987E1    49              DEC ECX  
004987E2    74 01           JE SHORT xPEP.004987E5  
004987E5    8DC0            LEA EAX,EAX                     ; Illegal use of register 异常  
  
  
004987EB    64:8F05 0000000>POP DWORD PTR FS:\[0\]            ; 恢复 SEH  
004987F6    59              POP ECX                         
004987F7    61              POPAD  
  
  
  
// 计算字节校验和前先清除 13 个 API 数据, 保存到 Stack  
00498923    B9 0D000000          MOV ECX,0D                                ; 13 个 API  
00498928    8DB5 AA040000        LEA ESI,DWORD PTR SS:\[EBP+4AA\]            ; 49857B  
0049892E    8D7C8E FC            LEA EDI,DWORD PTR DS:\[ESI+ECX\*4-4\]        ; 4985AB  
00498932    AD                   LODS DWORD PTR DS:\[ESI\]  
00498933    50                   PUSH EAX                                  ; API Address 入栈  
00498934    8366 FC 00           AND DWORD PTR DS:\[ESI-4\],0                ; 清除 API 数据  
00498938  ^ E2 F8                LOOPD SHORT xPEP.00498932                 ; 看看 Stack  
  
  0012FF70   7C590A10  kernel32.GetModuleFileNameA  
  0012FF74   7C58C1AB  kernel32.CreateFileA  
  0012FF78   7C586168  kernel32.ReadFile  
  0012FF7C   7C577E6D  kernel32.CloseHandle  
  0012FF80   7C58E970  kernel32.VirtualProtectEx  
  0012FF84   7C596915  kernel32.OpenProcess  
  0012FF88   7C5978C1  kernel32.GetCurrentProcessId  
  0012FF8C   7C599D35  kernel32.CreateMutexA  
  0012FF90   7C58E85F  kernel32.VirtualFree  
  0012FF94   7C58E845  kernel32.VirtualAlloc  
  0012FF98   7C596972  kernel32.ExitProcess  
  0012FF9C   7C5901D5  kernel32.LoadLibraryA  
  0012FFA0   7C590C5F  kernel32.GetProcAddress  
  
// 计算 498000 开始的 2618 字节校验和, CRC32?   
0049893A    8DB5 2FFFFFFF        LEA ESI,DWORD PTR SS:\[EBP-D1\]            ; 498000  
00498940    BB 18260000          MOV EBX,2618  
00498945    8D51 FF              LEA EDX,DWORD PTR DS:\[ECX-1\]             ; ECX=0, EDX=FFFFFFFF  
  
00498948    33C0                 XOR EAX,EAX  
0049894A    AC                   LODS BYTE PTR DS:\[ESI\]  
0049894B    32C2                 XOR AL,DL  
0049894D    D1E8                 SHR EAX,1                                ; 右移位  
0049894F    73 08                JNB SHORT xPEP.00498959  
  
00498954    35 E195C7CD          XOR EAX,CDC795E1                         ; 移出的位 1 则 Xor  
  
00498959    41                   INC ECX  
0049895A    80E1 07              AND CL,7  
0049895D  ^ 75 EE                JNZ SHORT xPEP.0049894D                  ; 8 次右移位  
  
0049895F    C1EA 08              SHR EDX,8  
00498962    33D0                 XOR EDX,EAX                                
00498964    4B                   DEC EBX  
00498965  ^ 7F E1                JG SHORT xPEP.00498948  
00498967   /74 02                JE SHORT xPEP.0049896B                   ; EDX 结果=7636A915  
  
  
// 计算字节校验和后恢复 13 个 API 数据   
0049896B    B9 0D000000          MOV ECX,0D  
00498970    58                   POP EAX  
00498971    AB                   STOS DWORD PTR ES:\[EDI\]  
00498972    83EF 08              SUB EDI,8  
00498975  ^ E2 F9                LOOPD SHORT xPEP.00498970  
  
// 保存校验和信息  
00498977    66:3395 8C270000     XOR DX,WORD PTR SS:\[EBP+278C\]           ; \[49A85D\]=0  
0049897E    8995 8F280000        MOV DWORD PTR SS:\[EBP+288F\],EDX         ; \[49A960\] 校验和信息保存在这里  
00498984    F9                   STC  
00498985    72 01                JB SHORT xPEP.00498988  
  
  
//  取 EXE 文件名, 读 400h 字节  
00498A7D    8BC5                 MOV EAX,EBP                         ; 4980D1  
00498A7F    2B85 77280000        SUB EAX,DWORD PTR SS:\[EBP+2877\]     ; \[49A948\]=98000  
00498A85    2D D1000000          SUB EAX,0D1  
00498A8A    8985 7B280000        MOV DWORD PTR SS:\[EBP+287B\],EAX     ; \[49A94C\]=400000  
00498A90    8DBD 2FFFFFFF        LEA EDI,DWORD PTR SS:\[EBP-D1\]       ; 498000  
00498A96    57                   PUSH EDI  
00498A97    BB 00040000          MOV EBX,400  
00498A9C    8D77 08              LEA ESI,DWORD PTR DS:\[EDI+8\]        ; 498008  
  
00498A9F    68 04010000          PUSH 104                            ; BufferSize  
00498AA4    57                   PUSH EDI                            ; Buffer=498000  
00498AA5    50                   PUSH EAX                            ; hModule  
00498AA6    FF95 DA040000        CALL DWORD PTR SS:\[EBP+4DA\]         ; kernel32.GetModuleFileNameA  
  
00498AAC    56                   PUSH ESI                            ; xPEP.00498008  
00498AAD    33C0                 XOR EAX,EAX  
00498AAF    8D48 FF              LEA ECX,DWORD PTR DS:\[EAX-1\]  
00498AB2    FC                   CLD                                 ; 左到右  
00498AB3    F2:AE                REPNE SCAS BYTE PTR ES:\[EDI\]          
00498AB5    F7D1                 NOT ECX                             ; StrLen+1  
  
  
00498AB7    8D51 FF              LEA EDX,DWORD PTR DS:\[ECX-1\]  
00498ABA    FD                   STD                                 ; 右到左  
00498ABB    4F                   DEC EDI                             ; 指向最后的 0  
00498ABC    B0 5C                MOV AL,5C                           ; ''  
00498ABE    F2:AE                REPNE SCAS BYTE PTR ES:\[EDI\]  
00498AC0    FC                   CLD  
00498AC1    47                   INC EDI  
00498AC2    85C9                 TEST ECX,ECX                        ; 第几个字符是最后一个''  
00498AC4   /74 02                JE SHORT xPEP.00498AC8  
  
00498AC6    41                   INC ECX  
00498AC7    47                   INC EDI  
00498AC8    2BD1                 SUB EDX,ECX                         ; 文件名长度(不带路径)  
00498ACA    8BCA                 MOV ECX,EDX  
00498ACC    83E1 1F              AND ECX,1F                          ; 最大 1Fh  
00498ACF    8BF7                 MOV ESI,EDI  
00498AD1    8DBD 77290000        LEA EDI,DWORD PTR SS:\[EBP+2977\]     ; 49AA48 本来 "Error"  
00498AD7    F3:A4                REP MOVS BYTE PTR ES:\[EDI\],BYTE PTR DS:\[ESI\]     ; 保存到这里  
00498AD9    5E                   POP ESI  
00498ADA    5F                   POP EDI  
  
00498ADB    6A 00                PUSH 0  
00498ADD    68 80000000          PUSH 80  
00498AE2    6A 03                PUSH 3  
00498AE4    6A 00                PUSH 0  
00498AE6    6A 01                PUSH 1  
00498AE8    68 00000080          PUSH 80000000  
00498AED    57                   PUSH EDI  
00498AEE    FF95 D6040000        CALL DWORD PTR SS:\[EBP+4D6\]         ; kernel32.CreateFileA  
  
  0012FF84   00498AF4  /CALL to CreateFileA from xPEP.00498AEE  
  0012FF88   00498000  |FileName = "E:\\xikug\\xPEP.exe"  
  0012FF8C   80000000  |Access = GENERIC\_READ  
  0012FF90   00000001  |ShareMode = FILE\_SHARE\_READ  
  0012FF94   00000000  |pSecurity = NULL  
  0012FF98   00000003  |Mode = OPEN\_EXISTING  
  0012FF9C   00000080  |Attributes = NORMAL  
  0012FFA0   00000000  \\hTemplateFile = NULL  
  
00498AF4    50                   PUSH EAX  
00498AF5    6A 00                PUSH 0  
00498AF7    57                   PUSH EDI  
00498AF8    53                   PUSH EBX  
00498AF9    56                   PUSH ESI  
00498AFA    50                   PUSH EAX  
00498AFB    FF95 D2040000        CALL DWORD PTR SS:\[EBP+4D2\]          ; kernel32.ReadFile  
  
  0012FF88   00498B01  /CALL to ReadFile from xPEP.00498AFB  
  0012FF8C   00000044  |hFile = 00000044 (window)  
  0012FF90   00498008  |Buffer = xPEP.00498008  
  0012FF94   00000400  |BytesToRead = 400 (1024.)  
  0012FF98   00498000  |pBytesRead = xPEP.00498000  
  0012FF9C   00000000  \\pOverlapped = NULL  
  
00498B01    FF95 CE040000        CALL DWORD PTR SS:\[EBP+4CE\]          ; kernel32.CloseHandle  
  
  
// 计算 PE 头 CRC32 ?  
00498B07    8B46 3C         MOV EAX,DWORD PTR DS:\[ESI+3C\]             ; PE 头            
00498B0A    33C9            XOR ECX,ECX  
00498B0C    03C6            ADD EAX,ESI                               ; ESI=Buffer  
00498B0E    2148 58         AND DWORD PTR DS:\[EAX+58\],ECX             ; PE 头 CheckSum 清零              
00498B11    8D41 FF         LEA EAX,DWORD PTR DS:\[ECX-1\]  
00498B14    BF 2083B8ED     MOV EDI,EDB88320  
  
00498B19    33D2            XOR EDX,EDX  
00498B1B    8A16            MOV DL,BYTE PTR DS:\[ESI\]  
00498B1D    32D0            XOR DL,AL  
  
00498B1F    D1EA            SHR EDX,1                                 ; 右移一位  
00498B21    73 02           JNB SHORT xPEP.00498B25  
00498B23    33D7            XOR EDX,EDI                               ; 移出的是 1 则 XOR  
  
00498B25    41              INC ECX  
00498B26    80E1 07         AND CL,7  
00498B29  ^ 75 F4           JNZ SHORT xPEP.00498B1F                   ; 移 8 次  
  
00498B2B    C1E8 08         SHR EAX,8  
00498B2E    33C2            XOR EAX,EDX  
00498B30    46              INC ESI  
00498B31    4B              DEC EBX  
00498B32  ^ 7F E5           JG SHORT xPEP.00498B19  
00498B34    F7D0            NOT EAX                                   ; 结果 EAX = C47BB43A  
00498B36    3185 99270000   XOR DWORD PTR SS:\[EBP+2799\],EAX           ; \[49A86A\]  
  
  
// 检查 API 是否有效  
00498B3C    6A 0C           PUSH 0C                                   ; 13 个 API 中 后 12 个  
00498B3E    59              POP ECX  
00498B3F    8DB5 AE040000   LEA ESI,DWORD PTR SS:\[EBP+4AE\]            ; 49857F  
00498B45    FF76 04         PUSH DWORD PTR DS:\[ESI+4\]                 ; ExitProcess  
  
00498B48    AD              LODS DWORD PTR DS:\[ESI\]  
00498B49    85C0            TEST EAX,EAX  
00498B4B    74 05           JE SHORT xPEP.00498B52                    ; 12 个 API 任何一个为 0 则 ExitProcess  
00498B4D  ^ E2 F9           LOOPD SHORT xPEP.00498B48  
00498B4F    F9              STC  
00498B50    72 08           JB SHORT xPEP.00498B5A  
  
00498B52    58              POP EAX  
00498B53    61              POPAD  
00498B54    6A 00           PUSH 0  
00498B56    FF5424 E0       CALL DWORD PTR SS:\[ESP-20\]                ; ExitProcess  
  
// 检查 CreateMutex 是否下断  
00498B5A    58              POP EAX                                  ; kernel32.ExitProcess  
00498B5B    8DB5 BE040000   LEA ESI,DWORD PTR SS:\[EBP+4BE\]           ; 49858F  
00498B61    8B36            MOV ESI,DWORD PTR DS:\[ESI\]               ; kernel32.CreateMutexA  
  
00498B63    AC              LODS BYTE PTR DS:\[ESI\]  
00498B64    04 34           ADD AL,34                                ; CC+34=00  
00498B66    75 0C           JNZ SHORT xPEP.00498B74  
  
00498B68    81ED D1104000   SUB EBP,xPEP.004010D1                    ; 检查出断点, 出错  
00498B6E    0F85 18160000   JNZ xPEP.0049A18C  
  
  
  
// 第 10 个异常  
00498BBE    E8 01000000     CALL xPEP.00498BC4  
00498BC4    8B85 8D270000   MOV EAX,DWORD PTR SS:\[EBP+278D\]         ; \[49A85E\]=0  
00498BCA    8985 850B0000   MOV DWORD PTR SS:\[EBP+B85\],EAX          ; \[498C56\]  
00498BD0    892C24          MOV DWORD PTR SS:\[ESP\],EBP              ; 4980D1  
00498BD3    90              NOP  
00498BD4    8D85 170B0000   LEA EAX,DWORD PTR SS:\[EBP+B17\]          ; 498BE8, SEH Handler  
00498BDA    66:2BDB         SUB BX,BX  
00498BDD    50              PUSH EAX  
00498BDE    33C0            XOR EAX,EAX  
00498BE0    64:FF30         PUSH DWORD PTR FS:\[EAX\]  
00498BE3    64:8920         MOV DWORD PTR FS:\[EAX\],ESP  
00498BE6    EB 11           JMP SHORT xPEP.00498BF9  
  
00498BE8    8B6424 08       MOV ESP,DWORD PTR SS:\[ESP+8\]            ; 下断, shift+F9 到这里  
00498BEC    8B6C24 08       MOV EBP,DWORD PTR SS:\[ESP+8\]  
00498BF0    8D85 370B0000   LEA EAX,DWORD PTR SS:\[EBP+B37\]          ; 498C08  
00498BF6    50              PUSH EAX  
00498BF7    C3              RETN  
  
00498BF9    66:F7F3         DIV BX                                  ; 除零异常  
  
00498C08    8D85 4A0B0000   LEA EAX,DWORD PTR SS:\[EBP+B4A\]          ; 498C1B, SEH handler  
00498C0E    894424 04       MOV DWORD PTR SS:\[ESP+4\],EAX              
00498C12    64:8925 0000000>MOV DWORD PTR FS:\[0\],ESP                ; 修复 SEH  
  
  
  
// 第 11 个异常  
00498C19    EB 12           JMP SHORT xPEP.00498C2D  
  
00498C1B    8B6424 08       MOV ESP,DWORD PTR SS:\[ESP+8\]            ; 下断, shift+F9 到这里  
00498C1F    8B6C24 08       MOV EBP,DWORD PTR SS:\[ESP+8\]  
00498C23    8D85 730B0000   LEA EAX,DWORD PTR SS:\[EBP+B73\]          ; 498C44  
00498C29    50              PUSH EAX  
00498C2A    C3              RETN  
  
00498C2D    9C              PUSHFD  
00498C2E    810C24 00010000 OR DWORD PTR SS:\[ESP\],100               ; 准备一个单步异常  
00498C35    75 02           JNZ SHORT xPEP.00498C39  
00498C39    9D              POPFD  
  
00498C3A   /EB 01           JMP SHORT xPEP.00498C3D  
00498C3D    F8              CLC                                     ; 单步异常  
  
00498C44    8D85 8C0B0000   LEA EAX,DWORD PTR SS:\[EBP+B8C\]          ; 498C5D, SEH handler  
00498C4A    894424 04       MOV DWORD PTR SS:\[ESP+4\],EAX  
00498C4E    64:8925 0000000>MOV DWORD PTR FS:\[0\],ESP                ; 修复 SEH  
  
  
// 第 12 个异常  
00498C55    E9 00000000     JMP xPEP.00498C7B  
  
00498C5D    8B6424 08       MOV ESP,DWORD PTR SS:\[ESP+8\]            ; 下断, shift+F9 到这里  
00498C61    8B6C24 08       MOV EBP,DWORD PTR SS:\[ESP+8\]  
00498C65    8D85 B70B0000   LEA EAX,DWORD PTR SS:\[EBP+BB7\]          ; 498C88  
00498C6B    50              PUSH EAX  
00498C6C    EB 02           JMP SHORT xPEP.00498C70  
00498C70    81B5 93280000 6>XOR DWORD PTR SS:\[EBP+2893\],FACDDE63    ; \[49A964\]  
00498C7A    C3              RETN  
  
00498C7B    2BC0            SUB EAX,EAX                                
00498C7D    74 02           JE SHORT xPEP.00498C81  
00498C81    CC              INT3                                    ; INT3 异常  
00498C82    90              NOP  
  
  
00498C88    8D85 CB0B0000   LEA EAX,DWORD PTR SS:\[EBP+BCB\]          ; 498C9C, SEH handler  
00498C8E    894424 04       MOV DWORD PTR SS:\[ESP+4\],EAX  
00498C92    64:8925 0000000>MOV DWORD PTR FS:\[0\],ESP                ; 修复 SEH  
  
  
// 第 13 个异常  
00498C99    EB 1E           JMP SHORT xPEP.00498CB9  
  
00498C9C    8B6424 08       MOV ESP,DWORD PTR SS:\[ESP+8\]            ; 下断, shift+F9 到这里  
00498CA0    8B6C24 08       MOV EBP,DWORD PTR SS:\[ESP+8\]  
00498CA4    8D85 FB0B0000   LEA EAX,DWORD PTR SS:\[EBP+BFB\]          ; 498CCC  
00498CAA    50              PUSH EAX  
00498CAB    EB 01           JMP SHORT xPEP.00498CAE  
00498CAE    8185 93280000 4>ADD DWORD PTR SS:\[EBP+2893\],E5D91E43    ; \[49A964\]  
00498CB8    C3              RETN  
  
00498CB9    2BDB            SUB EBX,EBX  
00498CBB    8BD3            MOV EDX,EBX  
00498CBD    B8 72472388     MOV EAX,88234772  
00498CC2    F7F3            DIV EBX                                 ; 除零异常  
  
  
00498CCC    8D85 0F0C0000   LEA EAX,DWORD PTR SS:\[EBP+C0F\]          ; 498CE0, SEH handler  
00498CD2    894424 04       MOV DWORD PTR SS:\[ESP+4\],EAX  
00498CD6    64:8925 0000000>MOV DWORD PTR FS:\[0\],ESP                ; 修复 SEH  
  
  
  
// 第 14 个异常  
00498CDD    EB 11           JMP SHORT xPEP.00498CF0  
  
00498CE0    8B6424 08       MOV ESP,DWORD PTR SS:\[ESP+8\]             ; 下断, shift+F9 到这里  
00498CE4    8B6C24 08       MOV EBP,DWORD PTR SS:\[ESP+8\]  
00498CE8    8D85 3F0C0000   LEA EAX,DWORD PTR SS:\[EBP+C3F\]           ; 498D10  
00498CEE    50              PUSH EAX  
00498CEF    C3              RETN  
  
  
00498CF0    66:B8 0043      MOV AX,4300  
00498CF4    EB 02           JMP SHORT xPEP.00498CF8  
00498CF8    81B5 93280000 4>XOR DWORD PTR SS:\[EBP+2893\],96DF4B4A    ; \[49A964\]  
00498D02    CD 68           INT 68                                  ; Access\[FFFFFFFF\] 异常  
  
  
00498D10    8D85 540C0000   LEA EAX,DWORD PTR SS:\[EBP+C54\]          ; 498D25, SEH handler  
00498D16    894424 04       MOV DWORD PTR SS:\[ESP+4\],EAX  
00498D1A    64:8925 0000000>MOV DWORD PTR FS:\[0\],ESP                ; 修复 SEH  
  
  
// 第 15 个异常  
00498D21    EB 1F           JMP SHORT xPEP.00498D45  
  
00498D25    8B6424 08       MOV ESP,DWORD PTR SS:\[ESP+8\]            ; 下断, shift+F9 到这里  
00498D29    8B6C24 08       MOV EBP,DWORD PTR SS:\[ESP+8\]  
00498D2D    8D85 66150000   LEA EAX,DWORD PTR SS:\[EBP+1566\]         ; 499637  
00498D33    50              PUSH EAX  
00498D34    EB 01           JMP SHORT xPEP.00498D37  
00498D37    81AD 93280000 4>SUB DWORD PTR SS:\[EBP+2893\],3F139C44    ; \[49A964\]  
00498D41    C3              RETN  
  
  
00498D45    33DB            XOR EBX,EBX  
00498D47    8BC3            MOV EAX,EBX  
00498D49    66:BE 4040      MOV SI,4040  
00498D4D    66:BF 0D0A      MOV DI,0A0D  
00498D51    66:03FE         ADD DI,SI  
00498D54    66:81C6 0706    ADD SI,607  
00498D59    CC              INT3                                    ; INT 3 异常  
00498D5A    90              NOP  
  
  
00499637    64:8F05 0000000>POP DWORD PTR FS:\[0\]                     ; 0012FFE0  
0049963E    83C4 04         ADD ESP,4  
00499641    5D              POP EBP  
  
  
  
// 计算壳所用 API 的函数  
00499642    EB 00           JMP SHORT xPEP.00499644  
00499644    8BDD            MOV EBX,EBP                              ; xPEP.004980D1  
00499646    81ED D1104000   SUB EBP,xPEP.004010D1                    ; 97000  
  
0049964C    9C              PUSHFD  
0049964D    60              PUSHAD  
0049964E    8D85 801E4000   LEA EAX,DWORD PTR SS:\[EBP+401E80\]        ; 498E80  
00499654    FFD0            CALL EAX                                 ; 计算壳所用 API 的函数, F8, 见  最后 NOTE 3  
  
  
// GetVersionEx  
  typedef struct \_OSVERSIONINFO{   
        DWORD dwOSVersionInfoSize;   
        DWORD dwMajorVersion;   
        DWORD dwMinorVersion;   
        DWORD dwBuildNumber;   
        DWORD dwPlatformId;   
        TCHAR szCSDVersion\[ 128 \];   
  } OSVERSIONINFO  
  
00499656    68 94000000     PUSH 94  
0049965B    5B              POP EBX  
0049965C    2BE3            SUB ESP,EBX  
0049965E    8BF4            MOV ESI,ESP  
00499660    891E            MOV DWORD PTR DS:\[ESI\],EBX  
00499662    56              PUSH ESI  
00499663    FF95 641F4000   CALL DWORD PTR SS:\[EBP+401F64\]           ; kernel32.GetVersionExA  
00499669    03E3            ADD ESP,EBX  
0049966B    837E 10 02      CMP DWORD PTR DS:\[ESI+10\],2              ; PlatformID=2 表示 Win32\_NT  
0049966F   /75 0C           JNZ SHORT xPEP.0049967D  
  
00499671   |C785 E8204000 0>MOV DWORD PTR SS:\[EBP+4020E8\],1          ; \[4990E8\] 是否 NT 的标志  
0049967B   |EB 0A           JMP SHORT xPEP.00499687  
  
0049967D    C785 E8204000 0>MOV DWORD PTR SS:\[EBP+4020E8\],0  
  
00499687    83BD E8204000 0>CMP DWORD PTR SS:\[EBP+4020E8\],0  
0049968E    0F84 A4000000   JE xPEP.00499738                         ; 不是 NT 跳走, 可以省很多事哦.  
  
  
// 利用 Event 来进行同步 ???  
  
00499694    E8 0D000000     CALL xPEP.004996A6                       ; F7  
  
004996A6    5F              POP EDI                                  ; xPEP.00499699 "UnpackinSaga"  
  
004996A7    57              PUSH EDI  
004996A8    6A 00           PUSH 0  
004996AA    68 03001F00     PUSH 1F0003  
004996AF    FF95 AD1F4000   CALL DWORD PTR SS:\[EBP+401FAD\]           ; kernel32.OpenEventA  
  
  0012FF70   004996B5  禆 I.   /CALL to OpenEventA from xPEP.004996AF  
  0012FF74   001F0003  ..  |Access = 1F0003  
  0012FF78   00000000  ....  |Inheritable = FALSE  
  0012FF7C   00499699  櫀 I.   \\EventName = "UnpackinSaga"      ; US ??????   
  
  
004996B5    0BC0            OR EAX,EAX  
004996B7    75 1B           JNZ SHORT xPEP.004996D4                  ; 如 Open 成功, 将一直在这里循环  
  
004996B9    EB 0D           JMP SHORT xPEP.004996C8  
  
004996BB    57              PUSH EDI  
004996BC    6A 00           PUSH 0  
004996BE    6A 00           PUSH 0  
004996C0    6A 00           PUSH 0  
004996C2    FF95 9C1F4000   CALL DWORD PTR SS:\[EBP+401F9C\]           ; kernel32.CreateEventA  
  
004996C8    0BC0            OR EAX,EAX  
004996CA  ^ 74 EF           JE SHORT xPEP.004996BB  
  
004996CC    8985 EC204000   MOV DWORD PTR SS:\[EBP+4020EC\],EAX        ; \[4990EC\]  
004996D2    EB 14           JMP SHORT xPEP.004996E8                  ; 如 Create 成功, 跳出循环  
  
004996D4    50              PUSH EAX  
004996D5    FF95 9F154000   CALL DWORD PTR SS:\[EBP+40159F\]           ; kernel32.CloseHandle  
004996DB    68 F4010000     PUSH 1F4  
004996E0    FF95 CC1F4000   CALL DWORD PTR SS:\[EBP+401FCC\]           ; kernel32.Sleep  
004996E6  ^ EB BF           JMP SHORT xPEP.004996A7  
  
  
// 利用驱动 AntiTrace  
  
004996E8    E8 10FEFFFF     CALL xPEP.004994FD                       ; F8,  生成 C:\\WINNT\\system32\\Saga.sys, 见最后 NOTE 2  
  
004996ED    E8 89FEFFFF     CALL xPEP.0049957B                       ; 加载驱动,见最后 Note1.  跳过这句, 修改 EIP 到 499738 , 否则以后不能下普通断点.  
  
004996F2    CD20 8BC48B64   VxDJump 648BC48B                         ; 这句其实是 CD20 (INT 20), 调整后如下  
  
  
004996F2    CD20            INT 20                                   ; 下面几句都和驱动有关, 等我学会了再来解释吧, 哈.  
004996F4    8BC4            MOV EAX,ESP  
004996F6    8B6424 08       MOV ESP,DWORD PTR SS:\[ESP+8\]  
004996FA    50              PUSH EAX  
004996FB    60              PUSHAD  
004996FC    56              PUSH ESI  
004996FD    0F014C24 FE     SIDT FWORD PTR SS:\[ESP-2\]  
00499702    5E              POP ESI  
00499703    66:8B46 18      MOV AX,WORD PTR DS:\[ESI+18\]  
00499707    66:8B5E 1E      MOV BX,WORD PTR DS:\[ESI+1E\]  
0049970B    66:8985 3027400>MOV WORD PTR SS:\[EBP+402730\],AX  
00499712    66:899D 3227400>MOV WORD PTR SS:\[EBP+402732\],BX  
00499719    33C0            XOR EAX,EAX  
0049971B    66:8946 18      MOV WORD PTR DS:\[ESI+18\],AX  
0049971F    C1E8 10         SHR EAX,10  
00499722    66:8946 1E      MOV WORD PTR DS:\[ESI+1E\],AX  
00499726    61              POPAD  
00499727    5C              POP ESP  
00499728    8D85 38274000   LEA EAX,DWORD PTR SS:\[EBP+402738\]  
0049972E    50              PUSH EAX  
0049972F    CF              IRETD  
00499730    0000            ADD BYTE PTR DS:\[EAX\],AL  
00499732    0000            ADD BYTE PTR DS:\[EAX\],AL  
00499734    0000            ADD BYTE PTR DS:\[EAX\],AL  
00499736    0000            ADD BYTE PTR DS:\[EAX\],AL  
  
  
// 跳过驱动, 这里继续  
00499738    FFB5 EC204000   PUSH DWORD PTR SS:\[EBP+4020EC\]           ; handle of EVENT  
0049973E    FF95 9F154000   CALL DWORD PTR SS:\[EBP+40159F\]           ; kernel32.CloseHandle  
00499744    61              POPAD  
00499745    9D              POPFD  
  
  
// 检测 API 是否有 CC 断点  
00499746    8DBD 7B154000   LEA EDI,DWORD PTR SS:\[EBP+40157B\]        ; 49857B  
0049974C    8BB5 C2384000   MOV ESI,DWORD PTR SS:\[EBP+4038C2\]        ; user32.MessageBoxA  
00499752    AC              LODS BYTE PTR DS:\[ESI\]  
00499753    04 34           ADD AL,34                                ; CC+34=00  
00499755    0F84 310A0000   JE xPEP.0049A18C  
0049975B    AC              LODS BYTE PTR DS:\[ESI\]  
0049975C    04 34           ADD AL,34  
0049975E    0F84 280A0000   JE xPEP.0049A18C  
00499764    AC              LODS BYTE PTR DS:\[ESI\]  
00499765    04 34           ADD AL,34  
00499767    0F84 1F0A0000   JE xPEP.0049A18C  
0049976D    AC              LODS BYTE PTR DS:\[ESI\]  
0049976E    04 34           ADD AL,34  
00499770    0F84 160A0000   JE xPEP.0049A18C  
  
  
00499776    6A 0D           PUSH 0D                                  ; 共 14 个 API  
00499778    59              POP ECX  
  
00499779    8B37            MOV ESI,DWORD PTR DS:\[EDI\]                 
0049977B    83C7 04         ADD EDI,4  
0049977E    AC              LODS BYTE PTR DS:\[ESI\]  
0049977F    04 34           ADD AL,34  
00499781    0F84 D8060000   JE xPEP.00499E5F  
00499787    AC              LODS BYTE PTR DS:\[ESI\]  
00499788    04 34           ADD AL,34  
0049978A    0F84 CF060000   JE xPEP.00499E5F  
00499790    AC              LODS BYTE PTR DS:\[ESI\]  
00499791    04 34           ADD AL,34  
00499793    0F84 C6060000   JE xPEP.00499E5F  
00499799    AC              LODS BYTE PTR DS:\[ESI\]  
0049979A    04 34           ADD AL,34  
0049979C    0F84 BD060000   JE xPEP.00499E5F  
004997A2  ^\\E2 D5           LOOPD SHORT xPEP.00499779               ; 下一个  
  
  
// PE 头 CRC32 确认   
004997A4    8B85 6A384000   MOV EAX,DWORD PTR SS:\[EBP+40386A\]       ; \[49A86A\],  00498B36 计算的 PE 头 CRC32  
004997AA    50              PUSH EAX  
004997AB    35 1DFBE8F9     XOR EAX,F9E8FB1D  
004997B0    2D 21E59B98     SUB EAX,989BE521                        ; 这步后, EAX 应该为 0   
004997B5    5B              POP EBX  
  
004997B6    6A 01           PUSH 1  
004997B8    58              POP EAX  
  
004997B9    6A 08           PUSH 8  
004997BB    59              POP ECX  
  
004997BC    0F85 3D060000   JNZ xPEP.00499DFF                       ; 这里不能跳  
004997C2   /74 19           JE SHORT xPEP.004997DD                  ; 这里跳  
  
  
// ZwSetInformationThread  AntiDebug  
004997DD    60              PUSHAD  
004997DE    8BBD A7204000   MOV EDI,DWORD PTR SS:\[EBP+4020A7\]       ; ntdll.ZwSetInformationThread  
004997E4    85FF            TEST EDI,EDI  
004997E6    74 0F           JE SHORT xPEP.004997F7                  ; 强制跳  
  
004997E8    FF95 191F4000   CALL DWORD PTR SS:\[EBP+401F19\]           ; kernel32.GetCurrentThread  
004997EE    6A 00           PUSH 0  
004997F0    6A 00           PUSH 0  
004997F2    6A 11           PUSH 11  
004997F4    50              PUSH EAX  
004997F5    FFD7            CALL EDI  
004997F7    61              POPAD  
  
  
// 解密 8 个字节, Mutex Name, CreateMutex  
004997F8    8DB5 C4274000   LEA ESI,DWORD PTR SS:\[EBP+4027C4\]       ; 4997C4 "xPkWZ8Hau3EdYMn9"  
004997FE    8D7E 10         LEA EDI,DWORD PTR DS:\[ESI+10\]           ; 4997D4  
00499801    57              PUSH EDI                                ; xPEP.004997D4,    
                                                                    ; EBX = PE 头 CRC32, ECX=8, EAX=1  
  
00499802    8AC3            MOV AL,BL                                 
00499804    24 0F           AND AL,0F  
00499806    8A0406          MOV AL,BYTE PTR DS:\[ESI+EAX\]  
00499809    AA              STOS BYTE PTR ES:\[EDI\]  
0049980A    C1EB 04         SHR EBX,4  
0049980D  ^ E2 F3           LOOPD SHORT xPEP.00499802  
  
   \[4997D4\]="YWnPWaPH"  
  
0049980F    58              POP EAX  
00499810    50              PUSH EAX                                 ; \[4997D4\]  
00499811    6A 01           PUSH 1  
00499813    51              PUSH ECX  
00499814    FF95 8F154000   CALL DWORD PTR SS:\[EBP+40158F\]           ; kernel32.CreateMutexA  
0049981A    8D00            LEA EAX,DWORD PTR DS:\[EAX\]               ; handle  
0049981C    85E4            TEST ESP,ESP  
0049981E    EB 03           JMP SHORT xPEP.00499823  
  
  
// 第 16 个异常  
00499823    C1E8 DB         SHR EAX,0DB  
00499826    E8 0D010000     CALL xPEP.00499938  
  
00499938   /EB 02           JMP SHORT xPEP.0049993C  
  
  
0049993C    2BC5            SUB EAX,EBP  
0049993E    60              PUSHAD  
0049993F    E8 06000000     CALL xPEP.0049994A  
  
00499944    8B6424 08       MOV ESP,DWORD PTR SS:\[ESP+8\]              ; SEH handler, 下断  
00499948    EB 0D           JMP SHORT xPEP.00499957  
  
  
0049994A    2BD2            SUB EDX,EDX  
0049994C    64:FF32         PUSH DWORD PTR FS:\[EDX\]  
0049994F    64:8922         MOV DWORD PTR FS:\[EDX\],ESP  
00499952    F1              INT1                                      ; 单步异常                                    
00499953    F7F2            DIV EDX                                     
  
  
00499957    F8              CLC  
00499958    73 02           JNB SHORT xPEP.0049995C  
  
  
0049995C    2D D501C301     SUB EAX,1C301D5                             
00499961    2BF6            SUB ESI,ESI  
00499963    64:8F06         POP DWORD PTR FS:\[ESI\]                    ; 恢复 SEH  
00499966    5E              POP ESI  
00499967   /EB 02           JMP SHORT xPEP.0049996B  
  
// 第 17 个异常  
0049996B    F5              CMC  
0049996C    8BC1            MOV EAX,ECX  
0049996E    60              PUSHAD  
0049996F    E8 06000000     CALL xPEP.0049997A  
  
00499974    8B6424 08       MOV ESP,DWORD PTR SS:\[ESP+8\]             ; SEH Handler, 下断  
00499978    EB 1A           JMP SHORT xPEP.00499994  
  
0049997A    64:67:FF36 0000 PUSH DWORD PTR FS:\[0\]  
00499980    64:67:8926 0000 MOV DWORD PTR FS:\[0\],ESP  
00499986    9C              PUSHFD  
00499987    810C24 00010000 OR DWORD PTR SS:\[ESP\],100                ; 准备一个单步异常  
0049998E    9D              POPFD  
0049998F    F8              CLC  
00499990  ^ 73 DC           JNB SHORT xPEP.0049996E                  ; 这里单步异常  
  
  
00499994    64:67:8F06 0000 POP DWORD PTR FS:\[0\]                     ; 恢复 SEH  
0049999A    58              POP EAX  
0049999B    61              POPAD  
0049999C    EB 02           JMP SHORT xPEP.004999A0  
  
  
  
  
// 真正开始解压缩被加壳的程序, 乱又长的代码, 不细跟了  
// 可能的出口是 00499DFF, 00499CEE, 都下断, F9, 断在 499CEE, 哈  
  
0049982B    8BB5 68394000   MOV ESI,DWORD PTR SS:\[EBP+403968\]  
00499831    8BFD            MOV EDI,EBP  
00499833    8D85 1A284000   LEA EAX,DWORD PTR SS:\[EBP+40281A\]  
00499839    E8 00000000     CALL xPEP.0049983E  
0049983E    5B              POP EBX  
0049983F    81C3 09000000   ADD EBX,9  
00499845    8918            MOV DWORD PTR DS:\[EAX\],EBX  
  
00499847    8B9D 4C394000   MOV EBX,DWORD PTR SS:\[EBP+40394C\]  
0049984D    8B87 70394000   MOV EAX,DWORD PTR DS:\[EDI+403970\]  
00499853    03D8            ADD EBX,EAX  
00499855    8B8F 74394000   MOV ECX,DWORD PTR DS:\[EDI+403974\]  
0049985B    81E1 FFFFFF7F   AND ECX,7FFFFFFF  
...  
  
00499BCB    50              PUSH EAX  
00499BCC    51              PUSH ECX  
00499BCD    E8 1A000000     CALL xPEP.00499BEC                     ; F8 过这个 CALL(内有 VirtualAlloc, VirtualFree)  
00499BD2    83F8 FF         CMP EAX,-1  
00499BD5    0F84 24020000   JE xPEP.00499DFF                       ; 这个象循环出口  
00499BDB    83C7 0C         ADD EDI,0C  
00499BDE    4E              DEC ESI  
00499BDF    7E 06           JLE SHORT xPEP.00499BE7  
00499BE1  ^ FFA5 1A284000   JMP DWORD PTR SS:\[EBP+40281A\]          ; xPEP.00499847  
00499BE7    E9 02010000     JMP xPEP.00499CEE                      ; 这个也象循环出口  
  
  
  
  
// 是否要重定位 ?  
00499CEE    8BB5 40394000   MOV ESI,DWORD PTR SS:\[EBP+403940\]  
00499CF4    85F6            TEST ESI,ESI  
00499CF6    0F84 8B000000   JE xPEP.00499D87  
00499CFC    8B95 4C394000   MOV EDX,DWORD PTR SS:\[EBP+40394C\]  
00499D02    03F2            ADD ESI,EDX  
00499D04    2B95 50394000   SUB EDX,DWORD PTR SS:\[EBP+403950\]  
00499D0A    74 7B           JE SHORT xPEP.00499D87                 ; 不需要, 跳  
  
  
  
// 终于处理 IAT 了  
00499D87    8B95 4C394000   MOV EDX,DWORD PTR SS:\[EBP+40394C\]  
00499D8D    8BB5 3C394000   MOV ESI,DWORD PTR SS:\[EBP+40393C\]     ; ESI=71000, IAT 在这里  
00499D93    0BF6            OR ESI,ESI  
00499D95    0F84 3B040000   JE xPEP.0049A1D6                      ; 没有 IAT 则结束  
  
  
00499D9B    03F2            ADD ESI,EDX  
00499D9D    83A5 3C3A4000 0>AND DWORD PTR SS:\[EBP+403A3C\],0  
00499DA4    33C0            XOR EAX,EAX  
00499DA6    8746 0C         XCHG DWORD PTR DS:\[ESI+C\],EAX         ; EAX = pDLLName  
00499DA9    0BC0            OR EAX,EAX  
00499DAB    0F84 25040000   JE xPEP.0049A1D6                      ; IAT 处理结束的出口  
  
00499DB1    35 78563412     XOR EAX,12345678  
00499DB6    E8 DC000000     CALL xPEP.00499E97                    ; F8 解密  
  
00499DBB    03C2            ADD EAX,EDX                           ; RVA->VA  
00499DBD    8BD8            MOV EBX,EAX  
00499DBF    50              PUSH EAX  
00499DC0    FF95 BA384000   CALL DWORD PTR SS:\[EBP+4038BA\]        ; kernel32.GetModuleHandleA  
  
00499DC6    85C0            TEST EAX,EAX  
00499DC8    0F85 E6000000   JNZ xPEP.00499EB4                     ; DLL, OK, 去判断是否需要加密  
00499DCE    53              PUSH EBX  
00499DCF    FF95 7F154000   CALL DWORD PTR SS:\[EBP+40157F\]        ; kernel32.LoadLibraryA  
  
  
00499DD5    85C0            TEST EAX,EAX  
00499DD7    0F85 D7000000   JNZ xPEP.00499EB4                     ; DLL, OK, 去判断是否需要加密  
  
  
  
  
//  判断 是不是需要加密的 4 个 DLL  
  
00499EB4    8985 34394000   MOV DWORD PTR SS:\[EBP+403934\],EAX  
00499EBA    8D85 7C304000   LEA EAX,DWORD PTR SS:\[EBP+40307C\]      ; 只加密这几个 DLL 的 API  
  
  0049A07C  47 44 49 33 32 2E 44 4C 4C 55 53 45 52 33 32 2E  GDI32.DLLUSER32.  
  0049A08C  44 4C 4C 53 48 45 4C 4C 33 32 2E 44 4C 4C 4B 45  DLLSHELL32.DLLKE  
  0049A09C  52 4E 45 4C 33 32 2E 44 4C 4C 00                 RNEL32.DLL.  
  
00499EC0    60              PUSHAD  
00499EC1    33C9            XOR ECX,ECX  
00499EC3    2AF6            SUB DH,DH  
00499EC5    8A13            MOV DL,BYTE PTR DS:\[EBX\]  
00499EC7    F6C2 40         TEST DL,40  
00499ECA    74 03           JE SHORT xPEP.00499ECF  
00499ECC    80E2 5F         AND DL,5F  
00499ECF    0AD2            OR DL,DL  
00499ED1    74 1E           JE SHORT xPEP.00499EF1  
00499ED3    43              INC EBX  
00499ED4    FEC6            INC DH  
00499ED6    41              INC ECX  
00499ED7    3A5408 FF       CMP DL,BYTE PTR DS:\[EAX+ECX-1\]  
00499EDB  ^ 74 E8           JE SHORT xPEP.00499EC5  
00499EDD    3A5408 08       CMP DL,BYTE PTR DS:\[EAX+ECX+8\]  
00499EE1  ^ 74 E2           JE SHORT xPEP.00499EC5  
00499EE3    3A5408 12       CMP DL,BYTE PTR DS:\[EAX+ECX+12\]  
00499EE7  ^ 74 DC           JE SHORT xPEP.00499EC5  
00499EE9    3A5408 1D       CMP DL,BYTE PTR DS:\[EAX+ECX+1D\]  
00499EED  ^ 74 D6           JE SHORT xPEP.00499EC5  
00499EEF  ^ EB D0           JMP SHORT xPEP.00499EC1  
  
00499EF1    0AF6            OR DH,DH  
00499EF3    895424 1C       MOV DWORD PTR SS:\[ESP+1C\],EDX  
00499EF7    61              POPAD  
00499EF8    C685 E1314000 0>MOV BYTE PTR SS:\[EBP+4031E1\],0  
00499EFF    74 24           JE SHORT xPEP.00499F25                 ; 这个改成 JMP, 就可以避开 IAT 加密了, 就不多说了.  
                                                                   ; Fly 等大侠都写过无数次了.  
  
  
00499F35    8B95 4C394000   MOV EDX,DWORD PTR SS:\[EBP+40394C\]  
00499F3B    8B06            MOV EAX,DWORD PTR DS:\[ESI\]  
00499F3D    85C0            TEST EAX,EAX  
00499F3F    75 0C           JNZ SHORT xPEP.00499F4D  
00499F41    FF76 10         PUSH DWORD PTR DS:\[ESI+10\]  
00499F44    58              POP EAX  
00499F45    85C0            TEST EAX,EAX  
00499F47  ^ 0F84 23FFFFFF   JE xPEP.00499E70  
  
  
...  
  
0049A063    53              PUSH EBX  
0049A064    FFB5 34394000   PUSH DWORD PTR SS:\[EBP+403934\]  
0049A06A    FF95 7B154000   CALL DWORD PTR SS:\[EBP+40157B\]         ; GetProcAddress  
  
  0012FF94   0049A070  /CALL to GetProcAddress from xPEP.0049A06A  
  0012FF98   010F0000  |hModule = 010F0000  
  0012FF9C   00000002  \\ProcNameOrOrdinal = #2  
  
0049A070    40              INC EAX  
0049A071    48              DEC EAX  
0049A072    75 33           JNZ SHORT xPEP.0049A0A7  
  
  
0049A170    8385 38394000 0>ADD DWORD PTR SS:\[EBP+403938\],4  
0049A177  ^ E9 B9FDFFFF     JMP xPEP.00499F35  
  
  
  
  
// IAT 处理结束后, 再次调用驱动  
0049A23A    FF95 93154000   CALL DWORD PTR SS:\[EBP+401593\]           ; kernel32.GetCurrentProcessId  
0049A240    8BD8            MOV EBX,EAX  
0049A242    50              PUSH EAX  
0049A243    6A 00           PUSH 0  
0049A245    68 FF0F1F00     PUSH 1F0FFF  
  
0049A24A    FF95 97154000   CALL DWORD PTR SS:\[EBP+401597\]           ; kernel32.OpenProcess  
  
  
0049A250    40              INC EAX  
0049A251    48              DEC EAX  
0049A252    74 2B           JE SHORT xPEP.0049A27F  
0049A254    6A 00           PUSH 0  
0049A256    54              PUSH ESP  
0049A257    6A 04           PUSH 4  
0049A259    68 00100000     PUSH 1000  
0049A25E    FFB5 4C394000   PUSH DWORD PTR SS:\[EBP+40394C\]  
0049A264    50              PUSH EAX  
0049A265    FF95 9B154000   CALL DWORD PTR SS:\[EBP+40159B\]           ; kernel32.VirtualProtectEx  
  
...  
  
004995FA    83BD E8204000 0>CMP DWORD PTR SS:\[EBP+4020E8\],0          ; \[4990E8\]=1, NT 的标志  
00499601    74 33           JE SHORT xPEP.00499636                   ; 改成 JMP  
  
00499603    60              PUSHAD                                   ; 这里又和驱动有关, 跳过  
00499604    CD20 8BC48B64   VxDJump 648BC48B  
0049960A    24 08           AND AL,8  
0049960C    50              PUSH EAX  
0049960D    60              PUSHAD  
0049960E    56              PUSH ESI  
0049960F    0F014C24 FE     SIDT FWORD PTR SS:\[ESP-2\]  
00499614    5E              POP ESI  
00499615    66:8B85 3027400>MOV AX,WORD PTR SS:\[EBP+402730\]  
0049961C    66:8B9D 3227400>MOV BX,WORD PTR SS:\[EBP+402732\]  
00499623    66:8946 18      MOV WORD PTR DS:\[ESI+18\],AX  
00499627    66:895E 1E      MOV WORD PTR DS:\[ESI+1E\],BX  
0049962B    61              POPAD  
0049962C    5C              POP ESP  
0049962D    8D85 35264000   LEA EAX,DWORD PTR SS:\[EBP+402635\]  
00499633    50              PUSH EAX  
00499634    CF              IRETD  
00499635    61              POPAD  
00499636    C3              RETN  
  
  
// 最后 3 个异常后到 OEP  
  
0049A319    55              PUSH EBP  
0049A31A    E8 30000000     CALL xPEP.0049A34F  
  
0049A336    8DC0            LEA EAX,EAX                            ; 非法指令                             ;   
  
  
0049A44E    FF07            INC DWORD PTR DS:\[EDI\]                 ; 单步异常  
  
  
0049A48A  ^\\73 DC           JNB SHORT xPEP.0049A468                ; 单步异常  
  
0049A46E    8B6424 08       MOV ESP,DWORD PTR SS:\[ESP+8\]           ; SEH Handler, 下断,  
0049A472    EB 1A           JMP SHORT xPEP.0049A48E  
  
0049A48E    64:67:8F06 0000 POP DWORD PTR FS:\[0\]                   ; 修复 SEH  
0049A494    58              POP EAX  
0049A495    61              POPAD  
0049A496    EB 01           JMP SHORT xPEP.0049A499  
...  
  
0049A407    61              POPAD  
0049A408    E8 00000000     CALL xPEP.0049A40D  
0049A40D    5A              POP EDX  
0049A40E    81EA 0E000000   SUB EDX,0E  
0049A414    57              PUSH EDI  
0049A415    51              PUSH ECX  
0049A416    33C0            XOR EAX,EAX  
0049A418    8D3A            LEA EDI,DWORD PTR DS:\[EDX\]  
0049A41A    B9 20000000     MOV ECX,20  
0049A41F    F3:AA           REP STOS BYTE PTR ES:\[EDI\]  
0049A421    66:AB           STOS WORD PTR ES:\[EDI\]  
0049A423    59              POP ECX  
0049A424    5F              POP EDI  
0049A425    C3              RETN                                   ; 到 OEP  
  
  
  
  
Dump, ImportREC 修复  
  
0040123C   /EB 10           JMP SHORT xPEP.0040124E  
0040123E   |66:623A         BOUND DI,DWORD PTR DS:\[EDX\]  
00401241   |43              INC EBX  
00401242   |2B2B            SUB EBP,DWORD PTR DS:\[EBX\]  
00401244   |48              DEC EAX  
00401245   |4F              DEC EDI  
00401246   |4F              DEC EDI  
00401247   |4B              DEC EBX  
00401248   |90              NOP  
00401249  -|E9 98004600     JMP 008612E6  
0040124E   \\A1 8B004600     MOV EAX,DWORD PTR DS:\[46008B\]  
00401253    C1E0 02         SHL EAX,2  
00401256    A3 8F004600     MOV DWORD PTR DS:\[46008F\],EAX  
0040125B    52              PUSH EDX  
0040125C    6A 00           PUSH 0  
0040125E    E8 69DD0500     CALL xPEP.0045EFCC                       ; JMP to kernel32.GetModuleHandleA  
  
  
RVA=71000, Size=D00,  Delete 无效的, RVA=72000, Size=1XXX, No ADD new.  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
Note 1  
  
// 加载驱动  
0049957B    9C              PUSHFD  
0049957C    60              PUSHAD  
0049957D    68 0F000F00     PUSH 0F000F                              ; dwDesiredAccess  
00499582    6A 00           PUSH 0  
00499584    6A 00           PUSH 0  
00499586    FF95 34204000   CALL DWORD PTR SS:\[EBP+402034\]           ; ADVAPI32.OpenSCManagerA  
0049958C    8BF8            MOV EDI,EAX                              ; SC\_handle  
0049958E    6A 00           PUSH 0  
00499590    6A 00           PUSH 0  
00499592    6A 00           PUSH 0  
00499594    6A 00           PUSH 0  
00499596    6A 00           PUSH 0  
00499598    FFB5 F0204000   PUSH DWORD PTR SS:\[EBP+4020F0\]           ; "C:\\WINNT\\system32\\Saga.sys"  
0049959E    6A 00           PUSH 0  
004995A0    6A 01           PUSH 1  
004995A2    6A 01           PUSH 1  
004995A4    68 30000100     PUSH 10030  
004995A9    E8 0F000000     CALL xPEP.004995BD                      ; F7  
004995BD    E8 0C000000     CALL xPEP.004995CE                      ; F7  
004995CE    50              PUSH EAX                                ; SC\_handle  
004995CF    FF95 47204000   CALL DWORD PTR SS:\[EBP+402047\]          ; ADVAPI32.CreateServiceA  
  
  0012FF20   004995D5  /CALL to CreateServiceA from xPEP.004995CF  
  0012FF24   001361E0  |hManager = 001361E0  
  0012FF28   004995C2  |ServiceName = "SagaService"  
  0012FF2C   004995AE  |DisplayName = "Unpacking Saga"  
  0012FF30   00010030  |DesiredAccess = SERVICE\_START|SERVICE\_STOP|10000  
  0012FF34   00000001  |ServiceType = SERVICE\_KERNEL\_DRIVER  
  0012FF38   00000001  |StartType = SERVICE\_SYSTEM\_START  
  0012FF3C   00000000  |ErrorControl = SERVICE\_ERROR\_IGNORE  
  0012FF40   00133888  |BinaryPathName = "C:\\WINNT\\system32\\Saga.sys"  
  0012FF44   00000000  |LoadOrderGroup = NULL  
  0012FF48   00000000  |pTagId = NULL  
  0012FF4C   00000000  |pDependencies = NULL  
  0012FF50   00000000  |ServiceStartName = NULL  
  0012FF54   00000000  \\Password = NULL  
  
  
004995D5    8BF0            MOV ESI,EAX                             ; handle to service  
004995D7    6A 00           PUSH 0                                  ; lpServiceArgVectors  
004995D9    6A 00           PUSH 0                                  ; dwNumServiceArgs  
004995DB    50              PUSH EAX                                ; handle  
004995DC    FF95 5A204000   CALL DWORD PTR SS:\[EBP+40205A\]          ; ADVAPI32.StartServiceA  
004995E2    56              PUSH ESI  
004995E3    FF95 6C204000   CALL DWORD PTR SS:\[EBP+@@Fumain@Finalize\]; ADVAPI32.DeleteService,    delete in 注册表, 但 service  不 stop  
004995E9    56              PUSH ESI  
004995EA    FF95 7E204000   CALL DWORD PTR SS:\[EBP+40207E\]          ; ADVAPI32.CloseServiceHandle  
004995F0    57              PUSH EDI  
004995F1    FF95 7E204000   CALL DWORD PTR SS:\[EBP+40207E\]          ; ADVAPI32.CloseServiceHandle  
004995F7    61              POPAD  
004995F8    9D              POPFD  
004995F9    C3              RETN  
  
  
  
  
  
  
  
  
Note 2  
  
  
// 生成 C:\\WINNT\\system32\\Saga.sys  
004994FD    9C              PUSHFD  
004994FE    60              PUSHAD  
004994FF    68 04010000     PUSH 104                                 ; size=104  
00499504    6A 40           PUSH 40                                  ; Flags=GPTR  
00499506    FF95 BC1F4000   CALL DWORD PTR SS:\[EBP+401FBC\]           ; kernel32.GlobalAlloc  
  
  
0049950C    8985 F0204000   MOV DWORD PTR SS:\[EBP+4020F0\],EAX        ; \[4990F0\]  
00499512    8BF8            MOV EDI,EAX  
00499514    68 04010000     PUSH 104                                 ; buffersize  
00499519    50              PUSH EAX                                 ; buffer  
0049951A    FF95 761F4000   CALL DWORD PTR SS:\[EBP+401F76\]           ; kernel32.GetSystemDirectoryA  
00499520    57              PUSH EDI  
00499521    FC              CLD  
00499522    E8 A7F9FFFF     CALL xPEP.00498ECE                       ; EDI 开始找名字结束的地方  
00499527    4F              DEC EDI  
00499528    B0 5C           MOV AL,5C                                ; ''  
0049952A    AA              STOS BYTE PTR ES:\[EDI\]                   ; 最后加一 ''  
  
0049952B    B8 53616761     MOV EAX,61676153  
00499530    AB              STOS DWORD PTR ES:\[EDI\]  
00499531    B8 2E737973     MOV EAX,7379732E  
00499536    AB              STOS DWORD PTR ES:\[EDI\]  
00499537    33C0            XOR EAX,EAX  
00499539    AA              STOS BYTE PTR ES:\[EDI\]                  ; 再加 "Saga.sys"  
  
0049953A    5F              POP EDI                                 ; "C:\\WINNT\\system32\\Saga.sys"  
0049953B    6A 00           PUSH 0  
0049953D    6A 20           PUSH 20  
0049953F    6A 04           PUSH 4  
00499541    6A 00           PUSH 0  
00499543    6A 02           PUSH 2  
00499545    68 00000040     PUSH 40000000  
0049954A    57              PUSH EDI  
0049954B    FF95 A7154000   CALL DWORD PTR SS:\[EBP+4015A7\]          ; kernel32.CreateFileA  
  
  0012FF38   00499551  Q 旾.   /CALL to CreateFileA from xPEP.0049954B  
  0012FF3C   00133888  ?.   |FileName = "C:\\WINNT\\system32\\Saga.sys"  
  0012FF40   40000000  ...@  |Access = GENERIC\_WRITE  
  0012FF44   00000002  ...  |ShareMode = FILE\_SHARE\_WRITE  
  0012FF48   00000000  ....  |pSecurity = NULL  
  0012FF4C   00000004  ...  |Mode = OPEN\_ALWAYS  
  0012FF50   00000020   ...  |Attributes = ARCHIVE  
  0012FF54   00000000  ....  \\hTemplateFile = NULL  
  
  
00499551    40              INC EAX  
00499552    74 24           JE SHORT xPEP.00499578  
00499554    48              DEC EAX  
00499555    8BF8            MOV EDI,EAX                           ; hFile  
00499557    8D9D F4204000   LEA EBX,DWORD PTR SS:\[EBP+4020F4\]     ; 4990F4  
0049955D    B9 09040000     MOV ECX,409  
00499562    6A 00           PUSH 0  
  
00499564    6A 00           PUSH 0  
00499566    54              PUSH ESP  
00499567    51              PUSH ECX  
00499568    53              PUSH EBX  
00499569    50              PUSH EAX  
0049956A    FF95 8E1F4000   CALL DWORD PTR SS:\[EBP+401F8E\]           ; kernel32.WriteFile  
  
  0012FF3C   00499570  p 旾.   /CALL to WriteFile from xPEP.0049956A  
  0012FF40   0000004C  L...  |hFile = 0000004C (window)  
  0012FF44   004990F4  魫 I.   |Buffer = xPEP.004990F4  
  0012FF48   00000409  ...  |nBytesToWrite = 409 (1033.)  
  0012FF4C   0012FF50  P�.  |pBytesWritten = 0012FF50  
  0012FF50   00000000  ....  \\pOverlapped = NULL  
  
  
00499570    58              POP EAX  
00499571    57              PUSH EDI                              ; hFile  
00499572    FF95 9F154000   CALL DWORD PTR SS:\[EBP+40159F\]        ; kernel32.CloseHandle  
  
00499578    61              POPAD  
00499579    9D              POPFD  
0049957A    C3              RETN  
  
  
  
  
  
  
  
  
  
Note 3  
  
// 计算壳所用 API 的函数  
00498E80    60              PUSHAD  
00498E81    FC              CLD  
00498E82    8DB5 D81E4000   LEA ESI,DWORD PTR SS:\[EBP+401ED8\]     ; 498ED8  
00498E88    AD              LODS DWORD PTR DS:\[ESI\]  
00498E89    EB 3D           JMP SHORT xPEP.00498EC8  
  
00498E8B    60              PUSHAD  
00498E8C    03C5            ADD EAX,EBP                          ; 类似 RVA->VA  
00498E8E    8BF8            MOV EDI,EAX  
00498E90    50              PUSH EAX  
00498E91    FF95 BA384000   CALL DWORD PTR SS:\[EBP+4038BA\]       ; kernel32.GetModuleHandleA  
00498E97    0BC0            OR EAX,EAX  
00498E99    75 07           JNZ SHORT xPEP.00498EA2  
  
00498E9B    57              PUSH EDI                             ; GetModuleHandleA 不成功则  
00498E9C    FF95 7F154000   CALL DWORD PTR SS:\[EBP+40157F\]       ; LoadLibraryA  
  
00498EA2    85C0            TEST EAX,EAX  
00498EA4    74 20           JE SHORT xPEP.00498EC6               ; Load 不成功                                 
  
00498EA6    8BF0            MOV ESI,EAX                          ; hModule  
  
00498EA8    E8 21000000     CALL xPEP.00498ECE                   ; EDI 开始找名字结束的地方  
  
00498EAD    8BD7            MOV EDX,EDI  
00498EAF    83C7 04         ADD EDI,4                            ; 4 个 0 后下一个 API  
00498EB2    32C0            XOR AL,AL  
00498EB4    3807            CMP BYTE PTR DS:\[EDI\],AL             ; EDI 指向 API 名字  
00498EB6    74 0E           JE SHORT xPEP.00498EC6               ; 当前 DLL 的 API 处理完了, 跳  
00498EB8    52              PUSH EDX  
00498EB9    57              PUSH EDI                             ; API Name  
00498EBA    56              PUSH ESI                             ; hModule  
00498EBB    FF95 7B154000   CALL DWORD PTR SS:\[EBP+40157B\]       ; kernel32.GetProcAddress  
00498EC1    5A              POP EDX  
00498EC2    8902            MOV DWORD PTR DS:\[EDX\],EAX           ; 保存 Address  
00498EC4  ^ EB E2           JMP SHORT xPEP.00498EA8  
  
00498EC6    61              POPAD  
00498EC7    AD              LODS DWORD PTR DS:\[ESI\]              ; 下一个 DLL  
  
00498EC8    0BC0            OR EAX,EAX  
00498ECA  ^ 75 BF           JNZ SHORT xPEP.00498E8B  
  
00498ECC    EB 08           JMP SHORT xPEP.00498ED6              ; 所有 DLL 都处理完了  
  
00498ECE    50              PUSH EAX                          ; 一个小函数, EDI 开始找名字结束的地方  
00498ECF    32C0            XOR AL,AL  
00498ED1    AE              SCAS BYTE PTR ES:\[EDI\]  
00498ED2  ^ 75 FD           JNZ SHORT xPEP.00498ED1           ; EDI 指向名字结尾的 Null  
00498ED4    58              POP EAX  
00498ED5    C3              RETN  
  
00498ED6    61              POPAD  
00498ED7    C3              RETN  
  
  
  
  
// 4 个 DLL  
00498ED8  EC 1E 40 00 DE 1F 40 00 9D 20 40 00 27 20 40 00  ?@.?@.?@.' @.  
  
00498EE8  00 00 00 00 4B 65 72 6E 65 6C 33 32 2E 44 6C 6C  ....Kernel32.Dll  
00498EF8  00 00 00 00 00 43 72 65 61 74 65 54 68 72 65 61  .....CreateThrea  
00498F08  64 00 00 00 00 00 45 78 69 74 54 68 72 65 61 64  d.....ExitThread  
00498F18  00 00 00 00 00 47 65 74 43 75 72 72 65 6E 74 54  .....GetCurrentT  
00498F28  68 72 65 61 64 00 00 00 00 00 47 65 74 43 75 72  hread.....GetCur  
00498F38  72 65 6E 74 50 72 6F 63 65 73 73 00 00 00 00 00  rentProcess.....  
00498F48  47 65 74 54 69 63 6B 43 6F 75 6E 74 00 00 00 00  GetTickCount....  
00498F58  00 47 65 74 56 65 72 73 69 6F 6E 00 00 00 00 00  .GetVersion.....  
00498F68  47 65 74 56 65 72 73 69 6F 6E 45 78 41 00 00 00  GetVersionExA...  
00498F78  00 00 47 65 74 53 79 73 74 65 6D 44 69 72 65 63  ..GetSystemDirec  
00498F88  74 6F 72 79 41 00 00 00 00 00 57 72 69 74 65 46  toryA.....WriteF  
00498F98  69 6C 65 00 00 00 00 00 43 72 65 61 74 65 45 76  ile.....CreateEv  
00498FA8  65 6E 74 41 00 00 00 00 00 4F 70 65 6E 45 76 65  entA.....OpenEve  
00498FB8  6E 74 41 00 00 00 00 00 47 6C 6F 62 61 6C 41 6C  ntA.....GlobalAl  
00498FC8  6C 6F 63 00 00 00 00 00 53 6C 65 65 70 00 00 00  loc.....Sleep...  
00498FD8  00 00 00 00 00 00 55 73 65 72 33 32 2E 44 6C 6C  ......User32.Dll  
00498FE8  00 00 00 00 00 45 6E 61 62 6C 65 57 69 6E 64 6F  .....EnableWindo  
00498FF8  77 00 00 00 00 00 47 65 74 46 6F 72 65 67 72 6F  w.....GetForegro  
00499008  75 6E 64 57 69 6E 64 6F 77 00 00 00 00 00 53 65  undWindow.....Se  
00499018  74 54 69 6D 65 72 00 00 00 00 00 00 00 00 00 41  tTimer.........A  
00499028  64 76 61 70 69 33 32 2E 44 6C 6C 00 00 00 00 00  dvapi32.Dll.....  
00499038  4F 70 65 6E 53 43 4D 61 6E 61 67 65 72 41 00 00  OpenSCManagerA..  
00499048  00 00 00 43 72 65 61 74 65 53 65 72 76 69 63 65  ...CreateService  
00499058  41 00 00 00 00 00 53 74 61 72 74 53 65 72 76 69  A.....StartServi  
00499068  63 65 41 00 00 00 00 00 44 65 6C 65 74 65 53 65  ceA.....DeleteSe  
00499078  72 76 69 63 65 00 00 00 00 00 43 6C 6F 73 65 53  rvice.....CloseS  
00499088  65 72 76 69 63 65 48 61 6E 64 6C 65 00 00 00 00  erviceHandle....  
00499098  00 00 00 00 00 4E 74 64 6C 6C 2E 44 6C 6C 00 00  .....Ntdll.Dll..  
004990A8  00 00 00 5A 77 53 65 74 49 6E 66 6F 72 6D 61 74  ...ZwSetInformat  
004990B8  69 6F 6E 54 68 72 65 61 64 00 00 00 00 00 5A 77  ionThread.....Zw  
004990C8  51 75 65 72 79 49 6E 66 6F 72 6D 61 74 69 6F 6E  QueryInformation  
004990D8  50 72 6F 63 65 73 73 00 00 00 00 00 00 00 00 00  Process.........  
  
  
附件：Telock.rar

 4dd466d0e1e0cc123afbe504da2462ca