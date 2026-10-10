

# 仙剑 2004.1.6

> 原文：[`www.pediy.com/kssd/pediy06/pediy6945.htm`](https://www.pediy.com/kssd/pediy06/pediy6945.htm)

【前    言】：很好玩的一个新加密壳.辛苦作者.   
  
【软件名称】：仙剑----妻凉雪   
  
【下载页面】：点击此处下载   
  
【软件大小】：19.7 KB (20,259 字节)   
  
【应用平台】：WIN9X/WINNT/WIN2K/WINXP   
  
【软件简介】：一个很有意思的壳.(我个人不希望有最终版)   
  
【软件限制】：融合了很多加密壳的特色,而且还在不段注入新的技术.   
  
【文章作者】：辉仔 Yock   
  
【作者声明】：本人发表这篇文章只是为了学习和研究!!!(在这里向作者以万二分的歉意鞠躬...)   
  
【破解工具】：Ollydbg 1.09b(超级 Cool 的工具)     LordPE(一个超级的 PE 编辑工具,感谢作者)    ImportREC1.42(几乎所有脱壳的 CrackER 都用的工具---Cool)   
  
  
\=================================================================================================   
【过    程】：忽略所有异常(除了 INT 异常外)然后用 OD 载入,停在下面.   
  
00406060 >  55              PUSH    EBP   
//用 OD 载入,先停在这,然后按一下 F9   
00406061    8BEC            MOV     EBP,ESP   
00406063    6A FF           PUSH    -1   
  
  
  
\=================================================================================================   
00406326    8BEF            MOV     EBP,EDI   
//INT 中断在这里,看看堆栈   
00406328    33DB            XOR     EBX,EBX   
0040632A    64:8F03         POP     DWORD PTR FS:\[EBX\]               ; 0012FFE0   
0040632D    83C4 04         ADD     ESP,4   
00406330    3C 04           CMP     AL,4   
00406332    74 19           JE      SHORT 0040634D   
  
  
++++++++++++++++++++++++++++++++++++++++   
//看看堆栈   
0012FF9C   0012FFE0  Pointer to next SEH record   
0012FFA0   00406F1E  SE handler   
//我们对这个地址下内存断点.然后按 F9   
0012FFA4   0012D444   
  
++++++++++++++++++++++++++++++++++++++++   
  
\===============================================================================================   
00406F1E    55              PUSH    EBP   
//停在这里.看下面   
00406F1F    8BEC            MOV     EBP,ESP   
00406F21    57              PUSH    EDI   
00406F22    8B45 10         MOV     EAX,\[EBP+10\]   
00406F25    8BB8 9C000000   MOV     EDI,\[EAX+9C\]   
00406F2B    FFB7 A32F4000   PUSH    DWORD PTR \[EDI+402FA3\]   
00406F31    8F80 B8000000   POP     DWORD PTR \[EAX+B8\]               ; ntdll.77FB172E   
//注意这个指令,\[EAX+B8\]里面的地址是 406326   
//我们对 406326 下内存访问断点.然后 F9   
00406F37    89B8 B4000000   MOV     \[EAX+B4\],EDI   
00406F3D    C780 B0000000 0>MOV     DWORD PTR \[EAX+B0\],4   
00406F47    B8 00000000     MOV     EAX,0   
00406F4C    5F              POP     EDI                              ; ntdll.77FB172E   
00406F4D    C9              LEAVE   
00406F4E    C3              RETN   
//迷失在系统 DLL 里面了   
  
  
\==================================================================================================   
00406326    8BEF            MOV     EBP,EDI   
//停在这里  ^\_^   
//然后取消所有断点.   
//接下来要慢慢走一小段路,很快就可以看见黑名单了.   
00406328    33DB            XOR     EBX,EBX   
0040632A    64:8F03         POP     DWORD PTR FS:\[EBX\]               ; 0012FFE0   
0040632D    83C4 04         ADD     ESP,4   
00406330    3C 04           CMP     AL,4   
00406332    74 19           JE      SHORT 0040634D   
00406334    66:9C           PUSHFW   
00406336    72 08           JB      SHORT 00406340   
00406338    EB 01           JMP     SHORT 0040633B   
  
......   
省略...   
......   
00406491    8985 CA304000   MOV     \[EBP+4030CA\],EAX                 ; XPAL.004064DE   
00406497    8D85 79234000   LEA     EAX,\[EBP+402379\]   
0040649D    50              PUSH    EAX                              ; XPAL.004064DE   
0040649E    C3              RETN   
//这个地址返回 004064DE   
  
\=====================================================================================================   
//来到这里了,下面是黑名单检查站!   
  
004064DE    56              PUSH    ESI                              ; ntdll.77F50000   
004064DF    8BB5 CA304000   MOV     ESI,\[EBP+4030CA\]                 ; ntdll.ZwQueryInformationProcess   
004064E5    85F6            TEST    ESI,ESI                          ; ntdll.77F50000   
004064E7    74 2A           JE      SHORT 00406513   
//我把这里直接改成 JMP 了   
004064E9    6A 00           PUSH    0   
004064EB    8BC4            MOV     EAX,ESP   
004064ED    6A 00           PUSH    0   
004064EF    6A 04           PUSH    4   
004064F1    50              PUSH    EAX                              ; XPAL.004064DE   
004064F2    6A 07           PUSH    7   
004064F4    FF95 9E304000   CALL    \[EBP+40309E\]                     ; kernel32.GetCurrentProcess   
004064FA    50              PUSH    EAX                              ; XPAL.004064DE   
004064FB    FFD6            CALL    ESI                              ; ntdll.77F50000   
004064FD    0BC0            OR      EAX,EAX                          ; XPAL.004064DE   
004064FF    75 0F           JNZ     SHORT 00406510   
00406501    58              POP     EAX                              ; 0012D444   
00406502    0BC0            OR      EAX,EAX                          ; XPAL.004064DE   
00406504    74 0D           JE      SHORT 00406513   
00406506    6A 00           PUSH    0   
00406508    FF95 E12F4000   CALL    \[EBP+402FE1\]                     ; kernel32.ExitProcess   
//哈哈,自杀.   
0040650E    EB 03           JMP     SHORT 00406513   
00406510    83C4 04         ADD     ESP,4   
00406513    5E              POP     ESI                              ; 0012D444   
00406514    F785 232E4000 0>TEST    DWORD PTR \[EBP+402E23\],1   
0040651E    74 35           JE      SHORT 00406555   
//这里我也改成 JMP 了   
00406520    56              PUSH    ESI                              ; ntdll.77F50000   
00406521    8DB5 CE304000   LEA     ESI,\[EBP+4030CE\]   
00406527    EB 26           JMP     SHORT 0040654F   
00406529    6A 00           PUSH    0   
0040652B    56              PUSH    ESI                              ; ntdll.77F50000   
0040652C    FF95 88304000   CALL    \[EBP+403088\]                     ; kernel32.\_lopen   
00406532    83F8 FF         CMP     EAX,-1   
00406535    74 0F           JE      SHORT 00406546   
00406537    50              PUSH    EAX                              ; XPAL.004064DE   
00406538    FF95 67304000   CALL    \[EBP+403067\]                     ; kernel32.CloseHandle   
0040653E    6A 00           PUSH    0   
00406540    FF95 E12F4000   CALL    \[EBP+402FE1\]                     ; kernel32.ExitProcess   
//哈哈,自杀.   
00406546    EB 01           JMP     SHORT 00406549   
00406548    46              INC     ESI                              ; ntdll.77F50000   
00406549    803E 00         CMP     BYTE PTR \[ESI\],0   
0040654C  ^ 75 FA           JNZ     SHORT 00406548   
0040654E    46              INC     ESI                              ; ntdll.77F50000   
0040654F    803E 00         CMP     BYTE PTR \[ESI\],0   
00406552  ^ 75 D5           JNZ     SHORT 00406529   
00406554    5E              POP     ESI                              ; 0012D444   
00406555    F785 232E4000 1>TEST    DWORD PTR \[EBP+402E23\],10   
//来到这里之后就之一往下拉...   
//一直拉到 00406754 这个地址.   
0040655F    74 37           JE      SHORT 00406598   
00406561    64:FF35 3000000>PUSH    DWORD PTR FS:\[30\]   
00406568    58              POP     EAX                              ; 0012D444   
00406569    85C0            TEST    EAX,EAX                          ; XPAL.004064DE   
0040656B    79 1E           JNS     SHORT 0040658B   
0040656D    6A 00           PUSH    0   
0040656F    FF95 D12F4000   CALL    \[EBP+402FD1\]                     ; kernel32.GetModuleHandleA   
00406575    85D2            TEST    EDX,EDX                          ; ntdll.77FC1774   
........................   
........................   
省略...................   
........................   
........................   
0040673A    C1C0 C8         ROL     EAX,0C8                          ; Shift constant out of range 1..31   
0040673D    D8EB            FSUBR   ST,ST(3)   
0040673F    01E8            ADD     EAX,EBP   
00406741    AA              STOS    BYTE PTR ES:\[EDI\]   
00406742  ^ E2 CC           LOOPD   SHORT 00406710   
00406744    C3              RETN   
00406745    61              POPAD   
00406746    83C6 28         ADD     ESI,28   
00406749    42              INC     EDX   
0040674A    66:3B57 06      CMP     DX,\[EDI+6\]   
0040674E  ^ 0F85 4AFFFFFF   JNZ     0040669E   
00406754    C3              RETN   
//一直啦到这里.啦啦啦...   
//然后在这个地方直接按 F4 跳下来      ^\_^   
//这个返回 00406687   
  
  
00406687    8D85 F0254000   LEA     EAX,\[EBP+4025F0\]   
0040668D    50              PUSH    EAX                              ; XPAL.00400000   
0040668E    C3              RETN   
//返回 00406755   
  
  
  
00406755    66:9C           PUSHFW   
//当返回 00406755 后就要注意咯,这里是幻影的花指令,其中就只有几条是重要的   
  
00406757    72 08           JB      SHORT 00406761   
00406759    EB 01           JMP     SHORT 0040675C   
0040675B    63E8            ARPL    EAX,EBP   
..............   
...............   
0040676C    50              PUSH    EAX                              ; XPAL.00406755   
0040676D    8B9D 1B2E4000   MOV     EBX,\[EBP+402E1B\]                 ; XPAL.00400000   
//看看 EBX 是什么!?   
00406773    66:9C           PUSHFW   
00406775    6A 10           PUSH    10   
.................   
.................   
004067A4    8B85 1F2E4000   MOV     EAX,\[EBP+402E1F\]   
//看看 EAX 的值!   
//她+400000 就是 OEP=0040351A 了!   
//赶快在 OEP 下内存断点吧,之后按一下 F9   
004067AA    66:9C           PUSHFW   
004067AC    6A 10           PUSH    10   
004067AE    73 0B           JNB     SHORT 004067BB   
004067B0    EB 02           JMP     SHORT 004067B4   
004067B2    C151 E8 06      RCL     DWORD PTR \[ECX\-18\],6   
\=================================================================================   
//按了 F9 之后会发现没有停在 OEP=0040351A 处   
//而是停在 00406D3A 这里.   
  
00406D3A    803B 68               CMP     BYTE PTR \[EBX\],68   
//这个时候 EBX=OEP   
00406D3D    75 3E                 JNZ     SHORT 00406D7D   
//改成 JMP 试试   
00406D3F    83A8 C4000000 04      SUB     DWORD PTR \[EAX+C4\],4   
00406D46    8BB8 C4000000         MOV     EDI,\[EAX+C4\]   
00406D4C    FF73 01               PUSH    DWORD PTR \[EBX+1\]   
00406D4F    8F07                  POP     DWORD PTR \[EDI\]   
00406D51    C703 00000000         MOV     DWORD PTR \[EBX\],0   
//看看这个是什么?!灭迹!   
00406D57    66:9C                 PUSHFW   
00406D59    72 08                 JB      SHORT 00406D63   
00406D5B    EB 01                 JMP     SHORT 00406D5E   
00406D5D    63E8                  ARPL    EAX,EBP   
00406D5F    0300                  ADD     EAX,\[EAX\]   
00406D61    0000                  ADD     \[EAX\],AL   
00406D63  ^ 72 F6                 JB      SHORT 00406D5B   
00406D65    8383 C404669D EB      ADD     DWORD PTR \[EBX+9D6604C4\],-15   
00406D6C    0175 83               ADD     \[EBP\-7D\],ESI   
00406D6F    C3                    RETN   
00406D70    04 C6                 ADD     AL,0C6   
00406D72    0300                  ADD     EAX,\[EAX\]   
00406D74    8380 B8000000 05      ADD     DWORD PTR \[EAX+B8\],5   
00406D7B    EB 7E                 JMP     SHORT 00406DFB   
00406D7D    803B 6A               CMP     BYTE PTR \[EBX\],6A   
00406D80    75 41                 JNZ     SHORT 00406DC3   
//改成 JMP 试试   
00406D82    83A8 C4000000 04      SUB     DWORD PTR \[EAX+C4\],4   
00406D89    8BB8 C4000000         MOV     EDI,\[EAX+C4\]   
00406D8F    FF73 01               PUSH    DWORD PTR \[EBX+1\]   
00406D92    66:C74424 01 0000     MOV     WORD PTR \[ESP+1\],0   
00406D99    C64424 03 00          MOV     BYTE PTR \[ESP+3\],0   
00406D9E    66:C703 0000          MOV     WORD PTR \[EBX\],0   
//看看这里是什么?!灭迹!   
00406DA3    66:9C                 PUSHFW   
00406DA5    72 08                 JB      SHORT 00406DAF   
00406DA7    EB 01                 JMP     SHORT 00406DAA   
00406DA9    63E8                  ARPL    EAX,EBP   
00406DAB    0300                  ADD     EAX,\[EAX\]   
00406DAD    0000                  ADD     \[EAX\],AL   
00406DAF  ^ 72 F6                 JB      SHORT 00406DA7   
00406DB1    8383 C404669D EB      ADD     DWORD PTR \[EBX+9D6604C4\],-15   
00406DB8    0175 83               ADD     \[EBP\-7D\],ESI   
00406DBB    80B8 00000002 EB      CMP     BYTE PTR \[EAX+2000000\],0EB   
00406DC2    90                    NOP   
00406DC3    803B 55               CMP     BYTE PTR \[EBX\],55   
00406DC6    75 33                 JNZ     SHORT 00406DFB   
//改成 JMP   
00406DC8    83A8 C4000000 04      SUB     DWORD PTR \[EAX+C4\],4   
00406DCF    FFB0 B4000000         PUSH    DWORD PTR \[EAX+B4\]   
00406DD5    8F80 C4000000         POP     DWORD PTR \[EAX+C4\]   
00406DDB    C603 00               MOV     BYTE PTR \[EBX\],0   
//看看这里,灭迹!   
00406DDE    66:9C                 PUSHFW   
00406DE0    72 08                 JB      SHORT 00406DEA   
00406DE2    EB 01                 JMP     SHORT 00406DE5   
00406DE4    90                    NOP   
00406DE5    E8 03000000           CALL    00406DED   
00406DEA  ^ 72 F6                 JB      SHORT 00406DE2   
00406DEC    90                    NOP   
00406DED    83C4 04               ADD     ESP,4   
00406DF0    66:9D                 POPFW   
00406DF2    EB 01                 JMP     SHORT 00406DF5   
00406DF4    90                    NOP   
00406DF5    FF80 B8000000         INC     DWORD PTR \[EAX+B8\]   
00406DFB    B8 00000000           MOV     EAX,0   
//来到这里!   
//然后按一下 F9,很快就停在 OEP=0040351A    
//当然,在这里按 F9 一定要确定之前没有取消 OEP 的内存访问断点   
  
00406E00    5F                    POP     EDI   
00406E01    C9                    LEAVE   
00406E02    C3                    RETN   
//这里迷失在系统领空   
  
\======================================================================================   
//    ^\_^    赶快用 LordPE 把她 DUMP 下来.   
  
0040351A    6A 00                 PUSH    0   
0040351C    E8 3B000000           CALL    0040355C   
00403521    A3 00104000           MOV     \[401000\],EAX   
00403526    6A 00                 PUSH    0   
00403528    68 D0324000           PUSH    4032D0   
0040352D    6A 00                 PUSH    0   
0040352F    6A 64                 PUSH    64   
00403531    50                    PUSH    EAX   
00403532    E8 67000000           CALL    0040359E   
00403537    6A 00                 PUSH    0   
00403539    E8 12000000           CALL    00403550   
0040353E  - FF25 70144000         JMP     \[401470\]   
00403544  - FF25 74144000         JMP     \[401474\]   
...........................   
省略.................   
...........................   
004035CE  - FF25 DC144000         JMP     \[4014DC\]   
004035D4  - FF25 E0144000         JMP     \[4014E0\]   
004035DA  - FF25 E4144000         JMP     \[4014E4\]   
004035E0  - FF25 EC144000         JMP     \[4014EC\]   
//注意这个指针地址!   
  
  
  
\=======================================================================================   
【总    结】：   
  
用 LOrdPE 把她 DUMO 出来之后,用 ImportREC 的"一层查找就可以找到全部指针了!   
  
最后就是   
004035E0  - FF25 EC144000         JMP     \[4014EC\]   
这个地方了,这个指针我用工具怎么也修复不了,所以就自己动手了!   
跟踪没有脱壳的程序得知这个指针是\[ImageRvaToSection\],为其他程序加壳的时候才用到它---重要的指针.     
那么我们用 LordPE 打开修复好的程序-->目录-->导入表右边的那个\[..\]这个按钮,在最下面仔细找找就可以知道\[ImageRvaToSection\]这个指针的地址是多少了!我电脑里的地址是\[807C\]   
  
然后用 OD 载入修复好的主程序,来到   
004035E0  - FF25 EC144000         JMP     \[4014EC\]   
这个地址!把她改成   
JMP     \[40807C\]   
然后点右键的复制-->保存文件!   
  
哈哈,这样就完全修复了,应该是可以跨平台 D   
  
最后在这里真心感谢你花了那么多时间看这篇文章!谢谢了...   
  
by 辉仔 Yock   
2004.1.16  
 4dd466d0e1e0cc123afbe504da2462ca