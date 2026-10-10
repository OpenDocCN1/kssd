

# ASPack 的脱壳

> 原文：[`www.pediy.com/kssd/pediy05/pediy50236.htm`](https://www.pediy.com/kssd/pediy05/pediy50236.htm)

\======================================================================================   
本文也同时发布在 iPB 论坛-\[脱壳专题\], 希望你能够喜欢   
                                    DiKeN/iPB   
\======================================================================================   
1\. 完全解析各个程序部分的功能以及脱壳关键点;   
2\. 指出还原文件的大小的关键数据地址;   
  
其实没有必要写了, ASPack 的壳就那么简单, 没有 SEH, 没有 anti   
  
分析按照程序流程来, 可以顺着顺序看   
\======================================================================================   
01010001> 60            PUSHAD   
01010002  E8 03000000    CALL notepad.0101000A   
01010007  E9            db E9 <========花指令   
01010008  EB 04          JMP SHORT notepad.0101000E   
0101000A  5D            POP EBP   
0101000B  45            INC EBP   
0101000C  55            PUSH EBP   
0101000D  C3            RETN   
  
0101000E  E8 01000000    CALL notepad.01010014   
01010013  EB            db EB <========花指令   
  
01010014            5D            POP EBP   
01010015            BB EDFFFFFF    MOV EBX,-13   
0101001A            03DD          ADD EBX,EBP   
0101001C            81EB 00000100  SUB EBX,10000   
01010022            83BD 22040000 >CMP \[DWORD SS:EBP+422\],0   
01010029            899D 22040000  MOV \[DWORD SS:EBP+422\],EBX<=========保存 ImageBase, 后面会用到的   
0101002F            0F85 65030000  JNZ notepad.0101039A   
  
01010035            8D85 2E040000  LEA EAX,\[DWORD SS:EBP+42E\]   
0101003B            50            PUSH EAX   
0101003C            FF95 4D0F0000  CALL \[DWORD SS:EBP+F4D\]<===GetModuleHandleA('kernel32.dll')   
01010042            8985 26040000  MOV \[DWORD SS:EBP+426\],EAX   
01010048            8BF8          MOV EDI,EAX   
0101004A            8D5D 5E        LEA EBX,\[DWORD SS:EBP+5E\]   
0101004D            53            PUSH EBX   
0101004E            50            PUSH EAX   
0101004F            FF95 490F0000  CALL \[DWORD SS:EBP+F49\]<===GetProcAddress(hKernel,'VirtualAlloc');   
01010055            8985 4D050000  MOV \[DWORD SS:EBP+54D\],EAX   
0101005B            8D5D 6B        LEA EBX,\[DWORD SS:EBP+6B\]   
0101005E            53            PUSH EBX   
0101005F            57            PUSH EDI   
01010060            FF95 490F0000  CALL \[DWORD SS:EBP+F49\]<===GetProcAddress(hKernel,'VirtualFree');   
01010066            8985 51050000  MOV \[DWORD SS:EBP+551\],EAX   
0101006C            8D45 77        LEA EAX,\[DWORD SS:EBP+77\]   
0101006F            FFE0          JMP EAX   
  
0101008A            8B9D 31050000  MOV EBX,\[DWORD SS:EBP+531\]   
01010090            0BDB          OR EBX,EBX   
01010092            74 0A          JE SHORT notepad.0101009E   
01010094            8B03          MOV EAX,\[DWORD DS:EBX\]   
01010096            8785 35050000  XCHG \[DWORD SS:EBP+535\],EAX   
0101009C            8903          MOV \[DWORD DS:EBX\],EAX   
0101009E            8DB5 69050000  LEA ESI,\[DWORD SS:EBP+569\]   
010100A4            833E 00        CMP \[DWORD DS:ESI\],0<=======这个地方是比较重要的数据   
<==========================================================是还原文件源大小的重要数据   
<==========================================================数据格式为:   
<==========================================================RVA (相对虚拟地址)   
<==========================================================Size(解码后的大小, 也就是物理大小)   
<==========================================================这是在还原原大小时可以用到, 否则也没用   
010100A7            0F84 21010000  JE notepad.010101CE   
010100AD            6A 04          PUSH 4   
010100AF            68 00100000    PUSH 1000   
010100B4            68 00180000    PUSH 1800   
010100B9            6A 00          PUSH 0   
010100BB            FF95 4D050000  CALL \[DWORD SS:EBP+54D\]====>分配解码缓冲区   
010100C1            8985 56010000  MOV \[DWORD SS:EBP+156\],EAX   
010100C7            8B46 04        MOV EAX,\[DWORD DS:ESI+4\]   
010100CA            05 0E010000    ADD EAX,10E   
010100CF            6A 04          PUSH 4   
010100D1            68 00100000    PUSH 1000   
010100D6            50            PUSH EAX   
010100D7            6A 00          PUSH 0   
010100D9            FF95 4D050000  CALL \[DWORD SS:EBP+54D\]====>分配输出缓冲区   
010100DF            8985 52010000  MOV \[DWORD SS:EBP+152\],EAX   
010100E5            56            PUSH ESI   
010100E6            8B1E          MOV EBX,\[DWORD DS:ESI\]   
010100E8            039D 22040000  ADD EBX,\[DWORD SS:EBP+422\]   
010100EE            FFB5 56010000  PUSH \[DWORD SS:EBP+156\]   
010100F4            FF76 04        PUSH \[DWORD DS:ESI+4\]   
010100F7            50            PUSH EAX   
010100F8            53            PUSH EBX   
010100F9            E8 6E050000    CALL notepad.0101066C<=====解码数据 DeCode(outBuf,inBuf,size,buf)   
<=============================================================使用的 aPlib 的解码库   
010100FE            B3 00          MOV BL,0   
01010100            80FB 00        CMP BL,0   
01010103            75 5E          JNZ SHORT notepad.01010163<===是否为第一次解码   
01010105            FE85 EC000000  INC \[BYTE SS:EBP+EC\]   
0101010B            8B3E          MOV EDI,\[DWORD DS:ESI\]   
0101010D            03BD 22040000  ADD EDI,\[DWORD SS:EBP+422\]   
01010113            FF37          PUSH \[DWORD DS:EDI\]   
01010115            C607 C3        MOV \[BYTE DS:EDI\],0C3   
01010118            FFD7          CALL EDI   
0101011A            8F07          POP \[DWORD DS:EDI\]   
0101011C            50            PUSH EAX   
0101011D            51            PUSH ECX   
0101011E            56            PUSH ESI   
0101011F            53            PUSH EBX   
01010120            8BC8          MOV ECX,EAX   
01010122            83E9 06        SUB ECX,6   
01010125            8BB5 52010000  MOV ESI,\[DWORD SS:EBP+152\]   
0101012B            33DB          XOR EBX,EBX   
0101012D            0BC9          OR ECX,ECX   
0101012F            74 2E          JE SHORT notepad.0101015F   
01010131            78 2C          JS SHORT notepad.0101015F   
01010133            AC            LODS \[BYTE DS:ESI\]   
01010134            3C E8          CMP AL,0E8   
01010136            74 0A          JE SHORT notepad.01010142   
01010138            EB 00          JMP SHORT notepad.0101013A   
0101013A            3C E9          CMP AL,0E9   
0101013C            74 04          JE SHORT notepad.01010142   
0101013E            43            INC EBX   
0101013F            49            DEC ECX   
01010140          ^EB EB          JMP SHORT notepad.0101012D   
01010142            8B06          MOV EAX,\[DWORD DS:ESI\]   
01010144            EB 00          JMP SHORT notepad.01010146   
01010146            803E 07        CMP \[BYTE DS:ESI\],7   
01010149          ⁷⁵ F3          JNZ SHORT notepad.0101013E   
0101014B            24 00          AND AL,0   
0101014D            C1C0 18        ROL EAX,18   
01010150            2BC3          SUB EAX,EBX   
01010152            8906          MOV \[DWORD DS:ESI\],EAX   
01010154            83C3 05        ADD EBX,5   
01010157            83C6 04        ADD ESI,4   
0101015A            83E9 05        SUB ECX,5   
0101015D          ^EB CE          JMP SHORT notepad.0101012D   
0101015F            5B            POP EBX   
01010160            5E            POP ESI   
01010161            59            POP ECX   
01010162            58            POP EAX   
01010163            EB 08          JMP SHORT notepad.0101016D   
  
0101016D            8BC8          MOV ECX,EAX   
0101016F            8B3E          MOV EDI,\[DWORD DS:ESI\]   
01010171            03BD 22040000  ADD EDI,\[DWORD SS:EBP+422\]   
01010177            8BB5 52010000  MOV ESI,\[DWORD SS:EBP+152\]   
0101017D            C1F9 02        SAR ECX,2   
01010180            F3:A5          REP MOVS \[DWORD ES:EDI\],\[DWORD DS:ESI\]<====将解码后的数据写回   
01010182            8BC8          MOV ECX,EAX   
01010184            83E1 03        AND ECX,3   
01010187            F3:A4          REP MOVS \[BYTE ES:EDI\],\[BYTE DS:ESI\]<====将解码后的数据写回   
01010189            5E            POP ESI   
0101018A            68 00800000    PUSH 8000   
0101018F            6A 00          PUSH 0   
01010191            FFB5 52010000  PUSH \[DWORD SS:EBP+152\]   
01010197            FF95 51050000  CALL \[DWORD SS:EBP+551\]<====释放输出缓冲区   
0101019D            83C6 08        ADD ESI,8   
010101A0            833E 00        CMP \[DWORD DS:ESI\],0<=======ESI 重要数据哟!   
010101A3          ⁰F85 1EFFFFFF  JNZ notepad.010100C7<=======循环解码   
  
010101A9            68 00800000    PUSH 8000   
010101AE            6A 00          PUSH 0   
010101B0            FFB5 56010000  PUSH \[DWORD SS:EBP+156\]   
010101B6            FF95 51050000  CALL \[DWORD SS:EBP+551\]<====释放解码缓冲区   
010101BC            8B9D 31050000  MOV EBX,\[DWORD SS:EBP+531\]   
010101C2            0BDB          OR EBX,EBX   
010101C4            74 08          JE SHORT notepad.010101CE   
010101C6            8B03          MOV EAX,\[DWORD DS:EBX\]   
010101C8            8785 35050000  XCHG \[DWORD SS:EBP+535\],EAX   
010101CE            8B95 22040000  MOV EDX,\[DWORD SS:EBP+422\]   
010101D4            8B85 2D050000  MOV EAX,\[DWORD SS:EBP+52D\]   
010101DA            2BD0          SUB EDX,EAX   
010101DC            74 79          JE SHORT notepad.01010257   
<=======================下面这一段不知道干什么的, 到如今还没执行过=========>   
010101DE            8BC2          MOV EAX,EDX   
010101E0            C1E8 10        SHR EAX,10   
010101E3            33DB          XOR EBX,EBX   
010101E5            8BB5 39050000  MOV ESI,\[DWORD SS:EBP+539\]   
010101EB            03B5 22040000  ADD ESI,\[DWORD SS:EBP+422\]   
010101F1            833E 00        CMP \[DWORD DS:ESI\],0   
010101F4            74 61          JE SHORT notepad.01010257   
010101F6            8B4E 04        MOV ECX,\[DWORD DS:ESI+4\]   
010101F9            83E9 08        SUB ECX,8   
010101FC            D1E9          SHR ECX,1   
010101FE            8B3E          MOV EDI,\[DWORD DS:ESI\]   
01010200            03BD 22040000  ADD EDI,\[DWORD SS:EBP+422\]   
01010206            83C6 08        ADD ESI,8   
01010209            66:8B1E        MOV BX,\[WORD DS:ESI\]   
0101020C            C1EB 0C        SHR EBX,0C   
0101020F            83FB 01        CMP EBX,1   
01010212            74 0C          JE SHORT notepad.01010220   
01010214            83FB 02        CMP EBX,2   
01010217            74 16          JE SHORT notepad.0101022F   
01010219            83FB 03        CMP EBX,3   
0101021C            74 20          JE SHORT notepad.0101023E   
0101021E            EB 2C          JMP SHORT notepad.0101024C   
01010220            66:8B1E        MOV BX,\[WORD DS:ESI\]   
01010223            81E3 FF0F0000  AND EBX,0FFF   
01010229            66:01041F      ADD \[WORD DS:EDI+EBX\],AX   
0101022D            EB 1D          JMP SHORT notepad.0101024C   
0101022F            66:8B1E        MOV BX,\[WORD DS:ESI\]   
01010232            81E3 FF0F0000  AND EBX,0FFF   
01010238            66:01141F      ADD \[WORD DS:EDI+EBX\],DX   
0101023C            EB 0E          JMP SHORT notepad.0101024C   
0101023E            66:8B1E        MOV BX,\[WORD DS:ESI\]   
01010241            81E3 FF0F0000  AND EBX,0FFF   
01010247            01141F        ADD \[DWORD DS:EDI+EBX\],EDX   
0101024A            EB 00          JMP SHORT notepad.0101024C   
0101024C            66:830E FF    OR \[WORD DS:ESI\],0FFFF   
01010250            83C6 02        ADD ESI,2   
01010253          ^E2 B4          LOOPD SHORT notepad.01010209   
01010255          ^EB 9A          JMP SHORT notepad.010101F1   
  
01010257            8B95 22040000  MOV EDX,\[DWORD SS:EBP+422\]   
0101025D            8BB5 41050000  MOV ESI,\[DWORD SS:EBP+541\]   
01010263            0BF6          OR ESI,ESI   
01010265            74 11          JE SHORT notepad.01010278   
01010267            03F2          ADD ESI,EDX   
01010269            AD            LODS \[DWORD DS:ESI\]   
0101026A            0BC0          OR EAX,EAX   
0101026C            74 0A          JE SHORT notepad.01010278   
0101026E            03C2          ADD EAX,EDX   
01010270            8BF8          MOV EDI,EAX   
01010272            66:AD          LODS \[WORD DS:ESI\]   
01010274            66:AB          STOS \[WORD ES:EDI\]   
01010276          ^EB F1          JMP SHORT notepad.01010269   
  
  
01010278            BE 50660000    MOV ESI,6650<===============Import Table   
<========================这个是原始导入表的入口   
<========================在程序入口的这个偏移, 肯定没错   
<========================乘现在导入表还没覆盖 dumper 之   
0101027D            8B95 22040000  MOV EDX,\[DWORD SS:EBP+422\]   
01010283            03F2          ADD ESI,EDX   
01010285            8B46 0C        MOV EAX,\[DWORD DS:ESI+C\]   
01010288            85C0          TEST EAX,EAX   
0101028A            0F84 0A010000  JE notepad.0101039A   
01010290            03C2          ADD EAX,EDX   
01010292            8BD8          MOV EBX,EAX   
01010294            50            PUSH EAX   
01010295            FF95 4D0F0000  CALL \[DWORD SS:EBP+F4D\]   
0101029B            85C0          TEST EAX,EAX   
0101029D            75 07          JNZ SHORT notepad.010102A6   
0101029F            53            PUSH EBX   
010102A0            FF95 510F0000  CALL \[DWORD SS:EBP+F51\]   
010102A6            8985 45050000  MOV \[DWORD SS:EBP+545\],EAX   
010102AC            C785 49050000 >MOV \[DWORD SS:EBP+549\],0   
010102B6            8B95 22040000  MOV EDX,\[DWORD SS:EBP+422\]   
010102BC            8B06          MOV EAX,\[DWORD DS:ESI\]   
010102BE            85C0          TEST EAX,EAX   
010102C0            75 03          JNZ SHORT notepad.010102C5   
010102C2            8B46 10        MOV EAX,\[DWORD DS:ESI+10\]   
010102C5            03C2          ADD EAX,EDX   
010102C7            0385 49050000  ADD EAX,\[DWORD SS:EBP+549\]   
010102CD            8B18          MOV EBX,\[DWORD DS:EAX\]   
010102CF            8B7E 10        MOV EDI,\[DWORD DS:ESI+10\]   
010102D2            03FA          ADD EDI,EDX   
010102D4            03BD 49050000  ADD EDI,\[DWORD SS:EBP+549\]   
010102DA            85DB          TEST EBX,EBX   
010102DC            0F84 A2000000  JE notepad.01010384   
010102E2            F7C3 00000080  TEST EBX,80000000   
010102E8            75 04          JNZ SHORT notepad.010102EE   
010102EA            03DA          ADD EBX,EDX   
010102EC            43            INC EBX   
010102ED            43            INC EBX   
010102EE            53            PUSH EBX   
010102EF            81E3 FFFFFF7F  AND EBX,7FFFFFFF   
010102F5            53            PUSH EBX   
010102F6            FFB5 45050000  PUSH \[DWORD SS:EBP+545\]   
010102FC            FF95 490F0000  CALL \[DWORD SS:EBP+F49\]   
01010302            85C0          TEST EAX,EAX   
01010304            5B            POP EBX   
01010305            75 6F          JNZ SHORT notepad.01010376   
01010307            F7C3 00000080  TEST EBX,80000000   
0101030D            75 19          JNZ SHORT notepad.01010328   
0101030F            57            PUSH EDI   
01010310            8B46 0C        MOV EAX,\[DWORD DS:ESI+C\]   
01010313            0385 22040000  ADD EAX,\[DWORD SS:EBP+422\]   
01010319            50            PUSH EAX   
0101031A            53            PUSH EBX   
0101031B            8D85 75040000  LEA EAX,\[DWORD SS:EBP+475\]   
01010321            50            PUSH EAX   
01010322            57            PUSH EDI   
01010323            E9 98000000    JMP notepad.010103C0   
01010328            81E3 FFFFFF7F  AND EBX,7FFFFFFF   
0101032E            8B85 26040000  MOV EAX,\[DWORD SS:EBP+426\]   
01010334            3985 45050000  CMP \[DWORD SS:EBP+545\],EAX   
0101033A            75 24          JNZ SHORT notepad.01010360   
0101033C            57            PUSH EDI   
0101033D            8BD3          MOV EDX,EBX   
0101033F            4A            DEC EDX   
01010340            C1E2 02        SHL EDX,2   
01010343            8B9D 45050000  MOV EBX,\[DWORD SS:EBP+545\]   
01010349            8B7B 3C        MOV EDI,\[DWORD DS:EBX+3C\]   
0101034C            8B7C3B 78      MOV EDI,\[DWORD DS:EBX+EDI+78\]   
01010350            035C3B 1C      ADD EBX,\[DWORD DS:EBX+EDI+1C\]   
01010354            8B0413        MOV EAX,\[DWORD DS:EBX+EDX\]   
01010357            0385 45050000  ADD EAX,\[DWORD SS:EBP+545\]   
0101035D            5F            POP EDI   
0101035E            EB 16          JMP SHORT notepad.01010376   
01010360            57            PUSH EDI   
01010361            8B46 0C        MOV EAX,\[DWORD DS:ESI+C\]   
01010364            0385 22040000  ADD EAX,\[DWORD SS:EBP+422\]   
0101036A            50            PUSH EAX   
0101036B            53            PUSH EBX   
0101036C            8D85 C6040000  LEA EAX,\[DWORD SS:EBP+4C6\]   
01010372            50            PUSH EAX   
01010373            57            PUSH EDI   
01010374            EB 4A          JMP SHORT notepad.010103C0   
01010376            8907          MOV \[DWORD DS:EDI\],EAX   
01010378            8385 49050000 >ADD \[DWORD SS:EBP+549\],4   
0101037F          ^E9 32FFFFFF    JMP notepad.010102B6   
01010384            8906          MOV \[DWORD DS:ESI\],EAX   
01010386            8946 0C        MOV \[DWORD DS:ESI+C\],EAX   
01010389            8946 10        MOV \[DWORD DS:ESI+10\],EAX   
0101038C            83C6 14        ADD ESI,14   
0101038F            8B95 22040000  MOV EDX,\[DWORD SS:EBP+422\]   
01010395          ^E9 EBFEFFFF    JMP notepad.01010285   
  
0101039A            B8 20640000    MOV EAX,6420   
<========================这个是原始程序的入口, 也就是 OEP 了   
<========================在程序入口的这个偏移, 肯定没错   
<========================好了, 到此你已经没事了, 唯一需要的就是修复导入表入口和 EP 了   
0101039F            50            PUSH EAX   
010103A0            0385 22040000  ADD EAX,\[DWORD SS:EBP+422\]<====修改 OEP 的 RVA 程 VA   
010103A6            59            POP ECX   
010103A7            0BC9          OR ECX,ECX   
010103A9            8985 A8030000  MOV \[DWORD SS:EBP+3A8\],EAX<====+写入   
010103AF            61            POPAD                          +   
010103B0            75 08          JNZ SHORT notepad.010103BA    +   
010103B2            B8 01000000    MOV EAX,1                      +   
010103B7            C2 0C00        RETN 0C                        +   
010103BA            68 00000000    PUSH 0=========================+   
010103BF            C3            RETN<==========================返回原始程序   
\======================================================================================   
Enjoy it:)   
  
                                            DiKeN/iPB   
\======================================================================================   
我相信, 看了这篇文章, 你应该会了 ASPack 的脱壳了.   
关于完全修复, 我就不做赘述, 精通 PE 结构的人可以修复, 新手没有必要修复了   
\======================================================================================   
结束语:   
    标准 ASPack 的壳, 就这样简单. 都是这样, 要还原成原样也没问题   
    tELock 的壳, 也使用了 aPLib 作为其压缩引擎, 不过它有一次加密/解密   
    UPX 也使用了 aPLib 这个压缩引擎.   
    aPLib 引擎, 以前的版本没有了. 可以到 http://apack.cjb.net   
    或者 http://home19.inet.tele.dk/jibz/apack/   
    或者到 iPB 论坛, 工具栏下载. 有兴趣的可以看一下   
\====================================================================================== 

 4dd466d0e1e0cc123afbe504da2462ca