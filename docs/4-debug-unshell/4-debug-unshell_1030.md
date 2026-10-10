

# PC Guard 和 SVKP1.32 两层变态壳

> 原文：[`www.pediy.com/kssd/pediy06/pediy6943.htm`](https://www.pediy.com/kssd/pediy06/pediy6943.htm)

两点兄找到的,据说他搞了一天,毫无进展,我可是搞了两天.   
  
Btw:pcguard 和 svkp1.32 都是很猛的壳,要完成它们的组合壳,你必须有两个单个壳的脱壳经验.   
  
OD 载入软件,隐藏 OD,异常设置除了忽略内存异常外,全部打勾.   
  
004143E6    8913            MOV DWORD PTR DS:\[EBX\],EDX  开始载入后程序已经异常.   
004143E8    FC              CLD   
004143E9    EB 01           JMP SHORT unpackme.004143EC   
004143EB    0AEB            OR CH,BL   
004143ED    2D 038B9AF6     SUB EAX,F69A8B03   
004143F2    C11F 75         RCR DWORD PTR DS:\[EDI\],75                ; 移动常数超出 1..31 的范围   
004143F5    028B F2AC8D9D   ADD CL,BYTE PTR DS:\[EBX+9D8DACF2\]   
.............................................................................   
  
  
异常 2   
  
00414639    8903            MOV DWORD PTR DS:\[EBX\],EAX   
0041463B    FC              CLD   
0041463C    EB 01           JMP SHORT unpackme.0041463F   
0041463E    0AEB            OR CH,BL   
00414640    3103            XOR DWORD PTR DS:\[EBX\],EAX   
  
确定几个入口警告.   
  
异常 3   
  
04590492    8900            MOV DWORD PTR DS:\[EAX\],EAX   
04590494    E8 01000000     CALL 0459049A   
04590499    E8 E8020000     CALL 04590786   
0459049E    00CD            ADD CH,CL   
045904A0    2083 04240B83   AND BYTE PTR DS:\[EBX+830B2404\],AL   
045904A6    44              INC ESP   
045904A7    24 04           AND AL,4   
  
异常 4   
  
045F03E1    8900            MOV DWORD PTR DS:\[EAX\],EAX   
045F03E3    60              PUSHAD   
045F03E4    E8 03000000     CALL 045F03EC   
045F03E9    D2EB            SHR BL,CL   
045F03EB    0A58 EB         OR BL,BYTE PTR DS:\[EAX-15\]   
045F03EE    0148 40         ADD DWORD PTR DS:\[EAX+40\],ECX   
045F03F1    EB 01           JMP SHORT 045F03F4   
  
异常 5   
  
045F137F    6285 0E0B0000   BOUND EAX,QWORD PTR SS:\[EBP+B0E\]   
045F1385    EB 02           JMP SHORT 045F1389   
045F1387    0FE88B D1EB02CD PSUBSB MM1,QWORD PTR DS:\[EBX+CD02EBD1\]   
045F138E    208B C2EB02CD   AND BYTE PTR DS:\[EBX+CD02EBC2\],CL   
045F1394    208B 8A4F0800   AND BYTE PTR DS:\[EBX+84F8A\],CL   
045F139A    007C03 EB       ADD BYTE PTR DS:\[EBX+EAX-15\],BH   
045F139E    0369 74         ADD EBP,DWORD PTR DS:\[ECX+74\]   
045F13A1    FB              STI   
  
异常 6   
  
045FC028    CD 01           INT 1   
045FC02A    E8 01000000     CALL 045FC030   
045FC02F  - E9 83C4047C     JMP 806484B7   
045FC034    03EB            ADD EBP,EBX   
045FC036    039A 74FB648F   ADD EBX,DWORD PTR DS:\[EDX+8F64FB74\]   
045FC03C    05 00000000     ADD EAX,0   
045FC041    E8 02000000     CALL 045FC048   
045FC046    CD 20           INT 20   
045FC048    830424 08       ADD DWORD PTR SS:\[ESP\],8   
045FC04C    C3              RETN   
  
停,Svkp1.32 典型异常最后一次.参考   
  
http://www.jxlb.com/bbs/dispbbs.asp?boardID=25&ID=1580   
  
看堆栈地址.   
  
0012FF74   0012FF84  指针到下一个 SEH 记录   
0012FF78   045F046F  SE 句柄  直接去哪里吧.   
0012FF7C   045F000C   
  
  
045F046F   /EB 0B           JMP SHORT 045F047C  真是熟悉.   
045F0471   |0000            ADD BYTE PTR DS:\[EAX\],AL   
045F0473   |FFCE            DEC ESI   
045F0475   |0000            ADD BYTE PTR DS:\[EAX\],AL   
045F0477   |0000            ADD BYTE PTR DS:\[EAX\],AL   
045F0479   |0000            ADD BYTE PTR DS:\[EAX\],AL   
045F047B   |00EB            ADD BL,CH   
045F047D    03C7            ADD EAX,EDI   
045F047F    84E8            TEST AL,CH   
  
到这里凭我的能力已经无法继续单步跟踪,可是我又要脱这个壳.用我的程咬金三斧头砍出 Oep   
  
Alt+M 打开内存镜像窗口.   
  
内存镜像   
地址       大小       Owner      Section    Contains      类型   访问      初始访问  映射为   
00400000   00001000   unpackme              PE header     Imag   RW        RWE   
00401000   00021000                                       Imag   RW        RWE   
00430000   00005000                                       Map    R E       R E   
004F0000   00002000                                       Map    R E       R E   
00500000   00103000                                       Map    R         R   
00610000   0007A000                                       Map    R E       R E   
04590000   00045000                                       Priv   RW        RW   
045F0000   00043000                                       Priv   RW        RW   
04640000   00002000                                       Priv   RW        RW   
62C20000   00001000   LPK                   PE header     Imag   R         RWE   
62C21000   00004000   LPK        .text      code,imports  Imag   R         RWE   
62C25000   00001000   LPK        .data      data          Imag   R         RWE   
62C26000   00001000   LPK        .rsrc      resources     Imag   R         RWE   
62C27000   00001000   LPK        .reloc     relocations   Imag   R         RWE   
72F10000   00001000   USP10                 PE header     Imag   R         RWE   
72F11000   0003A000   USP10      .text      code,imports  Imag   R         RWE   
72F4B000   00009000   USP10      .data      data          Imag   R         RWE   
72F54000   00002000   USP10      Shared                   Imag   R         RWE   
72F56000   00012000   USP10      .rsrc      resources     Imag   R         RWE   
72F68000   00002000   USP10      .reloc     relocations   Imag   R         RWE   
  
注意 401000 就是 Pe header 下面一句,平时我们看到的加密壳,包括 Asprotect 在 401000 处的 Contains 都是 code 类型.它也不是没有,只不过藏的很深,继续跟踪我迷路了.但是不管它玩什么 Anti 最终还是要访问 401000 处 Code,   
这一瞬间就是找 Oep 的时机会.你如果简单在 401000 处下内存访问断点,你会陷入死循环.   
  
呵呵,经过两天研究,双击 401000 这句,打开转存窗口.   
  
见附件图.   
  
  
在 401000 处下内存断点,F9 运行一直到 Oep 处.   
  
0461B6B1    8A06            MOV AL,BYTE PTR DS:\[ESI\]  断   
0461B6B3    46              INC ESI   
0461B6B4    47              INC EDI   
0461B6B5    8843 0F         MOV BYTE PTR DS:\[EBX+F\],AL   
0461B6B8    8A46 FF         MOV AL,BYTE PTR DS:\[ESI-1\]   
0461B6BB    55              PUSH EBP   
0461B6BC    E8 00000000     CALL 0461B6C1   
.............................................................   
  
04615F10    66:813E 4A43    CMP WORD PTR DS:\[ESI\],434A   
04615F15    0F85 23010000   JNZ 0461603E   
04615F1B    83C6 0A         ADD ESI,0A   
04615F1E    C745 FC 0800000>MOV DWORD PTR SS:\[EBP-4\],8   
04615F25    33DB            XOR EBX,EBX   
04615F27    BA 00000080     MOV EDX,80000000   
.............................................................   
  
046160DD    F3:A5           REP MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS>  这个东西很眼熟.   
046160DF    03CA            ADD ECX,EDX   
046160E1    F3:A4           REP MOVS BYTE PTR ES:\[EDI\],BYTE PTR DS:\[>   
046160E3    5A              POP EDX   
046160E4    81C2 00040000   ADD EDX,400   
046160EA    8BC8            MOV ECX,EAX   
046160EC    EB 02           JMP SHORT 046160F0   
046160EE    0FE850 52       PSUBSB MM2,QWORD PTR DS:\[EAX+52\]   
046160F2    EB 02           JMP SHORT 046160F6   
  
00401000    6A 00           PUSH 0    呵呵 Oep,这是汇编语言特征入口,应该无法伪装,有人伪装 C++入口,由于技术愿因,也只能在入口处,这里已经过了核心区,插件直接脱壳吧.   
00401002    E8 5F000000     CALL 00401066   
00401007    A3 50344000     MOV DWORD PTR DS:\[403450\],EAX   
0040100C    68 00040000     PUSH 400   
00401011    68 26304000     PUSH 403026                              ; ASCII "C:MY DOCUMENTSUNPACKME.EXE   
  
I am packed with XXXXXXX, unpack me "   
00401016    FF35 50344000   PUSH DWORD PTR DS:\[403450\]               ; unpackme.00400000   
0040101C    E8 3F000000     CALL 00401060   
00401021    B9 27000000     MOV ECX,27   
00401026    8D35 26344000   LEA ESI,DWORD PTR DS:\[403426\]   
0040102C    8D3D 26304000   LEA EDI,DWORD PTR DS:\[403026\]   
00401032    03F8            ADD EDI,EAX   
00401034    66:B8 0A0D      MOV AX,0D0A   
00401038    66:AB           STOS WORD PTR ES:\[EDI\]   
0040103A    66:AB           STOS WORD PTR ES:\[EDI\]   
0040103C    F3:A4           REP MOVS BYTE PTR ES:\[EDI\],BYTE PTR DS:\[>   
0040103E    6A 40           PUSH 40   
00401040    68 00304000     PUSH 403000                              ; ASCII "Please Unpack Me:)"   
00401045    68 26304000     PUSH 403026                              ; ASCII "C:MY DOCUMENTSUNPACKME.EXE   
  
I am packed with XXXXXXX, unpack me "   
  
.................................................................................   
  
关闭 Od,用单独线程修复.   
  
修复方法,你如果读了我的手动脱壳进阶第八篇 Skvp1.32,应该很清楚.   
  
好，现在重新修复指针吧，先用等级 1 修复，还剩两个指针,先用插件修复 00002008 处的指针为 GetModuleHandleA,另外一个不用说肯定是 KERNEL32.ExitProcess,然后手动填入 00002000 处的 KERNEL32.ExitProcess 指针，修复脱壳文件,正常运行。   
  
  
btw:如果脱壳也像破解软件一样,哪我的脱文绝对属于爆破型,我的风格.   
 4dd466d0e1e0cc123afbe504da2462ca