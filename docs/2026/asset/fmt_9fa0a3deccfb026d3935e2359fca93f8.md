
# 闲来无事，压缩一个2000的记事本，写一篇ASPack的脱壳，看了不会让你失望 (18千字)

---

本文也同时发布在iPB论坛-[脱壳专题]，希望你能够喜欢

DiKeN/iPB

---

1. 完全解析各个程序部分的功能以及脱壳关键点；
2. 指出还原文件的大小的关键数据地址；

其实没有必要写了，ASPack的壳就那么简单，没有SEH，没有anti。分析按照程序流程来，可以顺着顺序看。

---

```assembly
01010001> 60            PUSHAD
01010002 E8 03000000    CALL notepad.0101000A
01010007 E9            db E9 <========花指令
01010008 EB 04        JMP SHORT notepad.0101000E
0101000A 5D          POP EBP
0101000B 45        INC EBP
0101000C 55      PUSH EBP
0101000D C3    RETN
0101000E E8 01000000    CALL notepad.01010014
01010013 EB            db EB <========花指令
01010014            5D      POP EBP
01010015 BB EDFFFFFF    MOV EBX,-13
0101001A    03DD          ADD EBX,EBP
0101001C          81EB 00000100  SUB EBX,10000
01010022          83BD 22040000 >CMP [DWORD SS:EBP+422],0
01010029            899D 22040000  MOV [DWORD SS:EBP+422],EBX<=========保存ImageBase, 后面会用到的
0101002F      0F85 65030000  JNZ notepad.0101039A
01010035          8D85 2E040000  LEA EAX,[DWORD SS:EBP+42E]
0101003B            50      PUSH EAX
0101003C  FF95 4D0F0000  CALL [DWORD SS:EBP+F4D]<===GetModuleHandleA('kernel32.dll')
01010042            8985 26040000  MOV [DWORD SS:EBP+426],EAX
01010048 8BF8          MOV EDI,EAX
0101004A        8D5D 5E        LEA EBX,[DWORD SS:EBP+5E]
0101004D            53        PUSH EBX
0101004E    50            PUSH EAX
0101004F          FF95 490F0000  CALL [DWORD SS:EBP+F49]<===GetProcAddress(hKernel,'VirtualAlloc');
01010055            8985 4D050000  MOV [DWORD SS:EBP+54D],EAX
0101005B 8D5D 6B        LEA EBX,[DWORD SS:EBP+6B]
0101005E          53 PUSH EBX
0101005F            57        PUSH EDI
01010060    FF95 490F0000  CALL [DWORD SS:EBP+F49]<===GetProcAddress(hKernel,'VirtualFree');
01010066            8985 51050000  MOV [DWORD SS:EBP+551],EAX
0101006C 8D45 77        LEA EAX,[DWORD SS:EBP+77]
0101006F          FFE0          JMP EAX
0101008A            8B9D 31050000 MOV EBX,[DWORD SS:EBP+531]
01010090 0BDB          OR EBX,EBX
01010092      74 0A          JE SHORT notepad.0101009E
01010094            8B03    MOV EAX,[DWORD DS:EBX]
01010096    8785 35050000  XCHG [DWORD SS:EBP+535],EAX
0101009C          8903          MOV [DWORD DS:EBX],EAX
0101009E            8DB5 69050000  LEA ESI,[DWORD SS:EBP+569]
010100A4    833E 00        CMP [DWORD DS:ESI],0<=======这个地方是比较重要的数据
<==========================================================是还原文件源大小的重要数据
<==========================================================数据格式为:
<==========================================================RVA (相对虚拟地址)
<==========================================================Size(解码后的大小, 也就是物理大小)
<==========================================================这是在还原原大小时可以用到,
否则也没用
010100A7            0F84 21010000 JE notepad.010101CE
010100AD            6A 04          PUSH 4
010100AF      68 00100000    PUSH 1000
010100B4        68 00180000    PUSH 1800
010100B9          6A 00          PUSH 0
010100BB            FF95 4D050000  CALL [DWORD SS:EBP+54D]====>分配解码缓冲区
010100C1  8985 56010000  MOV [DWORD SS:EBP+156],EAX
010100C7        8B46 04        MOV EAX,[DWORD DS:ESI+4]
010100CA            05 0E010000  ADD EAX,10E
010100CF            6A 04        PUSH 4
010100D1  68 00100000    PUSH 1000
010100D6    50            PUSH EAX
010100D7          6A 00          PUSH 0
010100D9            FF95 4D050000  CALL [DWORD SS:EBP+54D]====>分配输出缓冲区
010100DF  8985 52010000  MOV [DWORD SS:EBP+152],EAX
010100E5        56            PUSH ESI
010100E6            8B1E    MOV EBX,[DWORD DS:ESI]
010100E8    039D 22040000  ADD EBX,[DWORD SS:EBP+422]
010100EE          FFB5 56010000  PUSH [DWORD SS:EBP+156]
010100F4            FF76 04  PUSH [DWORD DS:ESI+4]
010100F7 50            PUSH EAX
010100F8        53            PUSH EBX
010100F9            E8 6E050000 CALL notepad.0101066C<=====解码数据DeCode(outBuf,inBuf,size,buf)
<=============================================================使用的aPlib的解码库
010100FE            B3 00    MOV BL,0
01010100 80FB 00        CMP BL,0
01010103      75 5E          JNZ SHORT notepad.01010163<===是否为第一次解码
01010105            FE85 EC000000  INC [BYTE SS:EBP+EC]
0101010B            8B3E        MOV EDI,[DWORD DS:ESI]
0101010D        03BD 22040000  ADD EDI,[DWORD SS:EBP+422]
01010113            FF37  PUSH [DWORD DS:EDI]
01010115 C607 C3        MOV [BYTE DS:EDI],0C3
01010118        FFD7          CALL EDI
0101011A            8F07  POP [DWORD DS:EDI]
0101011C 50            PUSH EAX
0101011D        51            PUSH ECX
0101011E            56      PUSH ESI
0101011F  53            PUSH EBX
01010120          8BC8          MOV ECX,EAX
01010122            83E9 06    SUB ECX,6
01010125 8BB5 52010000  MOV ESI,[DWORD SS:EBP+152]
0101012B      33DB          XOR EBX,EBX
0101012D          0BC9          OR ECX,ECX
0101012F            74 2E    JE SHORT notepad.0101015F
01010131    78 2C          JS SHORT notepad.0101015F
01010133            AC      LODS [BYTE DS:ESI]
01010134    3C E8          CMP AL,0E8
01010136          74 0A          JE SHORT notepad.01010142
01010138 EB 00          JMP SHORT notepad.0101013A
0101013A          3C E9          CMP AL,0E9
0101013C            74 04      JE SHORT notepad.01010142
0101013E      43            INC EBX
0101013F            49    DEC ECX
01010140          ^EB EB        JMP SHORT notepad.0101012D
01010142        8B06          MOV EAX,[DWORD DS:ESI]
01010144            EB 00      JMP SHORT notepad.01010146
01010146      803E 07        CMP [BYTE DS:ESI],7
01010149          ^75 F3 JNZ SHORT notepad.0101013E
0101014B 24 00          AND AL,0
0101014D      C1C0 18        ROL EAX,18
01010150          2BC3          SUB EAX,EBX
01010152            8906      MOV [DWORD DS:ESI],EAX
01010154      83C3 05        ADD EBX,5
01010157          83C6 04        ADD ESI,4
0101015A            83E9 05  SUB ECX,5
0101015D          ^EB CE        JMP SHORT notepad.0101012D
0101015F        5B            POP EBX
01010160            5E      POP ESI
01010161 59            POP ECX
01010162        58            POP EAX
01010163            EB 08    JMP SHORT notepad.0101016D
0101016D      8BC8          MOV ECX,EAX
0101016F          8B3E          MOV EDI,[DWORD DS:ESI]
01010171            03BD 22040000  ADD EDI,[DWORD SS:EBP+422]
01010177    8BB5 52010000  MOV ESI,[DWORD SS:EBP+152]
0101017D          C1F9 02        SAR ECX,2
01010180            F3:A5    REP MOVS [DWORD ES:EDI],[DWORD DS:ESI]<====将解码后的数据写回
01010182          8BC8          MOV ECX,EAX
01010184            83E1 03    AND ECX,3
01010187 F3:A4          REP MOVS [BYTE ES:EDI],[BYTE DS:ESI]<====将解码后的数据写回
01010189            5E      POP ESI
0101018A 68 00800000    PUSH 8000
0101018F  6A 00          PUSH 0
01010191        FFB5 52010000  PUSH [DWORD SS:EBP+152]
01010197          FF95 51050000  CALL [DWORD SS:EBP+551]<====释放输出缓冲区
0101019D            83C6 08  ADD ESI,8
010101A0            833E 00      CMP [DWORD DS:ESI],0<=======ESI重要数据哟!
010101A3        ^0F85 1EFFFFFF  JNZ notepad.010100C7<=======循环解码
010101A9            68 00800000  PUSH 8000
010101AE            6A 00        PUSH 0
010101B0  FFB5 56010000  PUSH [DWORD SS:EBP+156]
010101B6        FF95 51050000  CALL [DWORD SS:EBP+551]<====释放解码缓冲区
010101BC            8B9D 31050000  MOV EBX,[DWORD SS:EBP+531]
010101C2 0BDB          OR EBX,EBX
```
