

# WinKawaks 1.45

> 原文：[`www.pediy.com/kssd/pediy05/pediy50237.htm`](https://www.pediy.com/kssd/pediy05/pediy50237.htm)

WinKawaks 1.45 脱壳笔记   
  
  
最近有朋友问 UPX + cryptor 的壳怎么脱,我还真以为是 UPX + cryptor 的壳呢,结果下载了一看 fi2.5 显示为 UPX + cryptor,但是我要说这次 FI 错了,因为我从来没有遇到 UPX + cryptor 里面有 int3 的.   
后来有朋友说是老王的 NC?(什么东东?)加的壳?,我如在云里雾里.不管它,先脱着试试.   
  
0187:00732060 50              PUSH    EAX        //载入后停在这里   
0187:00732061 51              PUSH    ECX   
0187:00732062 52              PUSH    EDX   
0187:00732063 53              PUSH    EBX   
0187:00732064 54              PUSH    ESP   
0187:00732065 55              PUSH    EBP   
0187:00732066 56              PUSH    ESI   
0187:00732067 57              PUSH    EDI   
0187:00732068 E800000000      CALL    0073206D    //这里进去,然后就一路 F10   
0187:0073206D 5D              POP      EBP   
0187:0073206E 81ED1E1C4000    SUB      EBP,00401C1E   
0187:00732074 B97B090000      MOV      ECX,097B   
0187:00732079 8DBD661C4000    LEA      EDI,\[EBP+00401C66\]   
0187:0073207F 8BF7            MOV      ESI,EDI   
0187:00732081 AC              LODSB    
......   
0187:007320B3 E2CC            LOOP    00732081        //这里 g 007320B5   
0187:007320B5 8B6901          MOV      EBP,\[ECX+01\]   
0187:007320B8 FFA37D9888F6    JMP      NEAR \[EBX+F688987D\]   
0187:007320BE 1BFA            SBB      EDI,EDX   
0187:007320C0 E195            LOOPE    00732057   
0187:007320C2 94              XCHG    EAX,ESP   
0187:007320C3 15D494AA74      ADC      EAX,74AA94D4   
0187:007320C8 0FB65F0F        MOVZX    EBX,BYTE \[EDI+0F\]   
0187:007320CC 18F1            SBB      CL,DH   
0187:007320CE D20E            ROR      BYTE \[ESI\],CL   
0187:007320D0 CDA7            INT      A7   
0187:007320D2 B0A0            MOV      AL,A0   
......   
0187:00732113 CC              INT3                    //在这里下断 bpx 0073277C   
0187:00732114 8BEF            MOV      EBP,EDI               
0187:00732116 33DB            XOR      EBX,EBX   
0187:00732118 648F03          POP      DWORD \[FS:EBX\]   
0187:0073211B 83C404          ADD      ESP,BYTE +04   
0187:0073211E 3C04            CMP      AL,04   
0187:00732120 7405            JZ      00732127   
0187:00732122 EB01            JMP      SHORT 00732125   
0187:00732124 E961C38B85      JMP      85FEE48A   
0187:00732129 8F              DB      8F   
0187:0073212A 234000          AND      EAX,\[EAX+00\]   
0187:0073212D 03403C          ADD      EAX,\[EAX+3C\]   
0187:00732130 0580000000      ADD      EAX,80   
0187:00732135 8B08            MOV      ECX,\[EAX\]   
0187:00732137 038D8F234000    ADD      ECX,\[EBP+0040238F\]   
......   
0187:0073277C 55              PUSH    EBP            //ok,断下了,再 F10 吧!   
0187:0073277D 8BEC            MOV      EBP,ESP   
0187:0073277F 57              PUSH    EDI   
0187:00732780 8B4510          MOV      EAX,\[EBP+10\]   
0187:00732783 8BB89C000000    MOV      EDI,\[EAX+9C\]   
0187:00732789 FFB717254000    PUSH    DWORD \[EDI+00402517\]   
0187:0073278F 8F80B8000000    POP      DWORD \[EAX+B8\]   
0187:00732795 89B8B4000000    MOV      \[EAX+B4\],EDI   
0187:0073279B C780B00000000400+MOV      DWORD \[EAX+B0\],04   
0187:007327A5 B800000000      MOV      EAX,00   
0187:007327AA 5F              POP      EDI   
0187:007327AB C9              LEAVE    
0187:007327AC C3              RET                    //最后到 00732114    
0187:007327AD 55              PUSH    EBP   
......   
0187:00732112 FFCC            DEC      ESP   
0187:00732114 8BEF            MOV      EBP,EDI            //停在这里(简单的办法,载入后输入 i3here on;g,就可以直接停在这里)   
0187:00732116 33DB            XOR      EBX,EBX   
0187:00732118 648F03          POP      DWORD \[FS:EBX\]   
0187:0073211B 83C404          ADD      ESP,BYTE +04   
0187:0073211E 3C04            CMP      AL,04   
0187:00732120 7405            JZ      00732127   
0187:00732122 EB01            JMP      SHORT 00732125   
0187:00732124 E961C38B85      JMP      85FEE48A   
0187:00732129 8F              DB      8F   
0187:0073212A 234000          AND      EAX,\[EAX+00\]   
0187:0073212D 03403C          ADD      EAX,\[EAX+3C\]   
0187:00732130 0580000000      ADD      EAX,80   
0187:00732135 8B08            MOV      ECX,\[EAX\]   
0187:00732137 038D8F234000    ADD      ECX,\[EBP+0040238F\]   
......   
0187:0073261A 0F85B4FEFFFF    JNZ      NEAR 007324D4            (JUMP)    //g 00732620   
0187:00732620 33C0            XOR      EAX,EAX   
0187:00732622 40              INC      EAX   
0187:00732623 83F801          CMP      EAX,BYTE +01   
0187:00732626 7402            JZ      0073262A   
0187:00732628 61              POPA      
0187:00732629 C3              RET      
0187:0073262A F785972340000200+TEST    DWORD \[EBP+00402397\],02   
0187:00732634 7418            JZ      0073264E   
0187:00732636 8BBD8F234000    MOV      EDI,\[EBP+0040238F\]   
0187:0073263C 037F3C          ADD      EDI,\[EDI+3C\]   
0187:0073263F 8B4F54          MOV      ECX,\[EDI+54\]   
0187:00732642 8BB58F234000    MOV      ESI,\[EBP+0040238F\]   
......   
0187:00732676 8DBD42224000    LEA      EDI,\[EBP+00402242\]   
0187:0073267C 8BF7            MOV      ESI,EDI   
0187:0073267E B9DF000000      MOV      ECX,DF   
0187:00732683 33DB            XOR      EBX,EBX   
0187:00732685 AC              LODSB    
0187:00732686 3479            XOR      AL,79   
0187:00732688 2AC3            SUB      AL,BL   
0187:0073268A C0C002          ROL      AL,02   
0187:0073268D AA              STOSB    
0187:0073268E 43              INC      EBX   
0187:0073268F E2F4            LOOP    00732685        //g 00732691   
0187:00732691 8D1B            LEA      EBX,\[EBX\]   
0187:00732693 8C              DB      8C   
0187:00732694 356D7C637F      XOR      EAX,7F637C6D   
0187:00732699 0C6C            OR      AL,6C   
......   
0187:00732752 AA              STOSB    
0187:00732753 E2FD            LOOP    00732752        //g 00732755   
0187:00732755 8DBD21234000    LEA      EDI,\[EBP+00402321\]   
0187:0073275B B9C0020000      MOV      ECX,02C0   
0187:00732760 AA              STOSB    
0187:00732761 E2FD            LOOP    00732760        //g 00732763   
0187:00732763 61              POPA      
0187:00732764 50              PUSH    EAX   
0187:00732765 33C0            XOR      EAX,EAX   
0187:00732767 64FF30          PUSH    DWORD \[FS:EAX\]   
0187:0073276A 648920          MOV      \[FS:EAX\],ESP   
......   
0187:0072FAC0 7507            JNZ      0072FAC9   
0187:0072FAC2 8B1E            MOV      EBX,\[ESI\]   
0187:0072FAC4 83EEFC          SUB      ESI,BYTE -04   
0187:0072FAC7 11DB            ADC      EBX,EBX   
0187:0072FAC9 72ED            JC      0072FAB8            (JUMP)    //g 0072FACB   
0187:0072FACB B801000000      MOV      EAX,01   
0187:0072FAD0 01DB            ADD      EBX,EBX   
0187:0072FAD2 7507            JNZ      0072FADB   
0187:0072FAD4 8B1E            MOV      EBX,\[ESI\]   
0187:0072FAD6 83EEFC          SUB      ESI,BYTE -04   
0187:0072FAD9 11DB            ADC      EBX,EBX   
0187:0072FADB 11C0            ADC      EAX,EAX   
0187:0072FADD 01DB            ADD      EBX,EBX   
0187:0072FADF 730B            JNC      0072FAEC   
0187:0072FAE1 7519            JNZ      0072FAFC   
......   
0187:0072FB68 75F7            JNZ      0072FB61            (JUMP)    //g  0072FB6A   
0187:0072FB6A E94FFFFFFF      JMP      0072FABE   
0187:0072FB6F 90              NOP      
0187:0072FB70 8B02            MOV      EAX,\[EDX\]   
......   
0187:0072FAC9 72ED            JC      0072FAB8            (JUMP)    //g 0072FACB   
0187:0072FACB B801000000      MOV      EAX,01   
0187:0072FAD0 01DB            ADD      EBX,EBX   
0187:0072FAD2 7507            JNZ      0072FADB   
0187:0072FAD4 8B1E            MOV      EBX,\[ESI\]   
0187:0072FAD6 83EEFC          SUB      ESI,BYTE -04   
0187:0072FAD9 11DB            ADC      EBX,EBX   
0187:0072FADB 11C0            ADC      EAX,EAX   
0187:0072FADD 01DB            ADD      EBX,EBX   
......   
0187:0072FB81 E938FFFFFF      JMP      0072FABE        //g 0072FB86   
0187:0072FB86 5E              POP      ESI   
0187:0072FB87 89F7            MOV      EDI,ESI   
0187:0072FB89 B939200000      MOV      ECX,2039   
0187:0072FB8E 8A07            MOV      AL,\[EDI\]   
0187:0072FB90 47              INC      EDI   
0187:0072FB91 2CE8            SUB      AL,E8   
0187:0072FB93 3C01            CMP      AL,01   
0187:0072FB95 77F7            JA      0072FB8E   
......   
0187:0072FB97 803F15          CMP      BYTE \[EDI\],15   
0187:0072FB9A 75F2            JNZ      0072FB8E        //g 0072FB9C   
0187:0072FB9C 8B07            MOV      EAX,\[EDI\]   
0187:0072FB9E 8A5F04          MOV      BL,\[EDI+04\]   
0187:0072FBA1 66C1E808        SHR      AX,08   
0187:0072FBA5 C1C010          ROL      EAX,10   
0187:0072FBA8 86C4            XCHG    AL,AH   
0187:0072FBAA 29F8            SUB      EAX,EDI   
0187:0072FBAC 80EBE8          SUB      BL,E8   
......   
0187:0072FBB8 E2D9            LOOP    0072FB93        //g 0072FBBA   
0187:0072FBBA 8DBE00D03200    LEA      EDI,\[ESI+0032D000\]   
0187:0072FBC0 8B07            MOV      EAX,\[EDI\]   
0187:0072FBC2 09C0            OR      EAX,EAX   
0187:0072FBC4 7445            JZ      0072FC0B   
0187:0072FBC6 8B5F04          MOV      EBX,\[EDI+04\]   
0187:0072FBC9 8D84308C0A3300  LEA      EAX,\[EAX+ESI+00330A8C\]   
0187:0072FBD0 01F3            ADD      EBX,ESI   
0187:0072FBD2 50              PUSH    EAX   
0187:0072FBD3 83C708          ADD      EDI,BYTE +08   
0187:0072FBD6 FF967C0B3300    CALL    NEAR \[ESI+00330B7C\]   
......   
0187:0072FBEF 57              PUSH    EDI           
0187:0072FBF0 48              DEC      EAX   
0187:0072FBF1 F2AE            REPNE SCASB   
0187:0072FBF3 55              PUSH    EBP   
0187:0072FBF4 FF96800B3300    CALL    NEAR \[ESI+00330B80\]   
0187:0072FBFA 09C0            OR      EAX,EAX   
0187:0072FBFC 7407            JZ      0072FC05   
0187:0072FBFE 8903            MOV      \[EBX\],EAX   
0187:0072FC00 83C304          ADD      EBX,BYTE +04   
0187:0072FC03 EBD8            JMP      SHORT 0072FBDD   
0187:0072FC05 FF96840B3300    CALL    NEAR \[ESI+00330B84\]   
0187:0072FC0B 61              POPA                    //看上去还真象是 UPX 加壳的.   
0187:0072FC0C E903F6DBFF      JMP      004EF214        //看到这里吗,跳 oep 了哦!    
0187:0072FC11 0000            ADD      \[EAX\],AL   
0187:0072FC13 0000            ADD      \[EAX\],AL   
  
  
后记:我所有写的脱壳文章里,这次是我 U 得最多的,以往几次就搞定了,这次却用了 20 余次,这个壳里面太复杂,稍不小心就不知道跳到什么地方去了,所以写的详细一点.应该可以节约不少时间.   
  
听说是加的 2 层壳,我不知道加的是哪 2 层,没有仔细分析过,估计是在 73xxxx 是一层,然后跳 72xxxx 又是一层(这一层应该是 UPX).   
  
  
  
flyfancy   
http://flyfancy.126.com 

第二层肯定是 UPX 的，按标记应该是 UPX1.22   
最外面的一层嘛！我也不清楚是什么。不过他装入导入表如下:   
0073252C        0BC9          OR ECX,ECX   
0073252E        75 03          JNZ SHORT WinKawak.00732533   
00732530        8B4E 04        MOV ECX,\[DWORD DS:ESI+4\]   
00732533        038D 8F234000  ADD ECX,\[DWORD SS:EBP+40238F\]   
00732539        8B56 04        MOV EDX,\[DWORD DS:ESI+4\]   
0073253C        0395 8F234000  ADD EDX,\[DWORD SS:EBP+40238F\]   
00732542        E9 C3000000    JMP WinKawak.0073260A   
00732547        F701 00000080  TEST \[DWORD DS:ECX\],80000000   
0073254D        75 4B          JNZ SHORT WinKawak.0073259A   
0073254F        8B01          MOV EAX,\[DWORD DS:ECX\]   
00732551        83C0 02        ADD EAX,2   
00732554        0385 8F234000  ADD EAX,\[DWORD SS:EBP+40238F\]   
0073255A        50            PUSH EAX   
0073255B        E8 8BFFFFFF    CALL WinKawak.007324EB   
<=========这儿边解码，边进行 IAT 修改   
00732560        58            POP EAX   
00732561        8BF8          MOV EDI,EAX                        ; WinKawak.00731C5A   
00732563        52            PUSH EDX   
00732564        51            PUSH ECX   
00732565        50            PUSH EAX   
00732566        53            PUSH EBX   
00732567        FF95 1F254000  CALL \[DWORD SS:EBP+40251F\]   
0073256D        0BC0          OR EAX,EAX   
0073256F        75 07          JNZ SHORT WinKawak.00732578   
00732571        59            POP ECX   
  
0073257A        60            PUSHAD   
0073257B        F785 97234000 >TEST \[DWORD SS:EBP+402397\],4   
00732585        74 0E          JE SHORT WinKawak.00732595   
00732587        8D85 46214000  LEA EAX,\[DWORD SS:EBP+402146\]   
0073258D        50            PUSH EAX   
0073258E        8BC7          MOV EAX,EDI   
00732590        E9 DB010000    JMP WinKawak.00732770   
00732595        61            POPAD   
00732596        8902          MOV \[DWORD DS:EDX\],EAX   
别让他写入，免得 IAT 被他破坏，先把它的这层皮拔掉   
00732598        EB 19          JMP SHORT WinKawak.007325B3 

00731B6E                                            58 1C                X   
00731B7E  33 00 66 1C 33 00 76 1C 33 00 00 00 00 00 84 1C  3.f3.v3.....?0   
00731B8E  33 00 00 00 00 00 9C 1C 33 00 00 00 00 00 AE 1C  3.....?3.....? 0   
00731B9E  33 00 00 00 00 00 01 00 00 80 00 00 00 00 C2 1C  3.......€....?0   
00731BAE  33 00 00 00 00 00 28 06 00 80 00 00 00 00 CC 1C  3.....(.€....?0   
00731BBE  33 00 00 00 00 00 E4 1C 33 00 00 00 00 00 EA 1C  3.....?3.....? 0   
00731BCE  33 00 00 00 00 00 FE 1C 33 00 00 00 00 00 4B 45  3.....?3.....KE0   
00731BDE  52 4E 45 4C 33 32 2E 44 4C 4C 00 43 4F 4D 43 54  RNEL32.DLL.COMCT   
00731BEE  4C 33 32 2E 64 6C 6C 00 44 44 52 41 57 2E 64 6C  L32.dll.DDRAW.dl   
00731BFE  6C 00 44 49 4E 50 55 54 2E 64 6C 6C 00 44 53 4F  l.DINPUT.dll.DSO   
00731C0E  55 4E 44 2E 64 6C 6C 00 74 44 94 33 23 E2 46 C6  UND.dll.tD?#釬   
......   
00731C4E                                      4C 6F 61 64              ad   
00731C5E  4C 69 62 72 61 72 79 41 00 00 47 65 74 50 72 6F  LibraryA..GetPro   
00731C6E  63 41 64 64 72 65 73 73 00 00 45 78 69 74 50 72  cAddress..ExitPr   
00731C7E  6F 63 65 73 73 00 00 00 49 6D 61 67 65 4C 69 73  ocess...ImageLis   
00731C8E  74 5F 52 65 70 6C 61 63 65 49 63 6F 6E 00 00 00  t\_ReplaceIcon...   
00731C9E  44 69 72 65 63 74 44 72 61 77 43 72 65 61 74 65  DirectDrawCreate   
00731CAE  00 00 44 69 72 65 63 74 49 6E 70 75 74 43 72 65  ..DirectInputCre   
00731CBE  61 74 65 41 00 00 44 65 6C 65 74 65 44 43 00 00  ateA..DeleteDC..   
00731CCE  3F 3F 30 5F 57 69 6E 69 74 40 73 74 64 40 40 51  ??0\_Winit@std@@Q   
00731CDE  41 45 40 58 5A 00 00 00 72 61 6E 64 00 00 53 48  AE@XZ...rand..SH   
00731CEE  42 72 6F 77 73 65 46 6F 72 46 6F 6C 64 65 72 41  BrowseForFolderA   
00731CFE  00 00 53 65 74 4D 65 6E 75 00 00 00 00 00 00    ..SetMenu......   
  
(函数名的解码是高低为互换，DLL 名的解码我没注意^\_^)  4dd466d0e1e0cc123afbe504da2462ca