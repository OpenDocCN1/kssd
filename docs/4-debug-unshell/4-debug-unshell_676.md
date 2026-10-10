

# BmJanpanese 3.0

> 原文：[`www.pediy.com/kssd/pediy06/pediy6368.htm`](https://www.pediy.com/kssd/pediy06/pediy6368.htm)

BmJanpanese 3.0 手记   
kongfoo/2003.12.8   
　　fly 发表了一篇 EXEStealth 2.7 的脱壳文章，所用目标是大嘴   
日语 1.0 版，并给出了下载地址。由于想学习一下日文，到时打到   
日本时不会不知道鬼子们说什么。所以去下载地址看看，已经是   
3.0 版了。但 PEiD 查还是 EXEStealth 2.72-2.73 ->WebToolMaster   
壳。好，打开 fly 的文章跟着做：   
(载入后准备工作：isdebug->hide,options->只留 memory access   
violatoin)载入后停在下面，和文章中大嘴日语的代码不同，但   
和第四部分‘ExeStealth V2.72 主程序 ExeStealth2.exe 脱壳’相   
同，就跟着这部分做。（哪么说壳是 ASProtect 的壳了）   
00ED0060 > EB 00            JMP SHORT BmJapane.00ED0062   
00ED0062   EB 2F            JMP SHORT BmJapane.00ED0093   
00ED0064   53               PUSH EBX   
00ED0065   68 61726577      PUSH 77657261   
00ED006A   61               POPAD   
00ED006B   72 65            JB SHORT BmJapane.00ED00D2   
00ED006D   202D 20457865    AND BYTE PTR DS:\[65784520\],CH   
00ED0073   53               PUSH EBX   
00ED0074   74 65            JE SHORT BmJapane.00ED00DB   
00ED0076   61               POPAD   
00ED0077   6C               INS BYTE PTR ES:\[EDI\],DX                 ; I/O command   
00ED0078   74 68            JE SHORT BmJapane.00ED00E2   
00ED007A   00EB             ADD BL,CH   
00ED007C   16               PUSH SS   
00ED007D   77 77            JA SHORT BmJapane.00ED00F6   
00ED007F   77 2E            JA SHORT BmJapane.00ED00AF   
00ED0081   77 65            JA SHORT BmJapane.00ED00E8   
00ED0083   62746F 6F        BOUND ESI,QWORD PTR DS:\[EDI+EBP\*2+6F\]   
00ED0087   6C               INS BYTE PTR ES:\[EDI\],DX                 ; I/O command   
00ED0088   6D               INS DWORD PTR ES:\[EDI\],DX                ; I/O command   
00ED0089   61               POPAD   
00ED008A   73 74            JNB SHORT BmJapane.00ED0100   
00ED008C   65:72 2E         JB SHORT BmJapane.00ED00BD               ; Superfluous prefix   
00ED008F   636F 6D          ARPL DWORD PTR DS:\[EDI+6D\],EBP   
00ED0092   0060 90          ADD BYTE PTR DS:\[EAX\-70\],AH   
00ED0095   E8 00000000      CALL BmJapane.00ED009A   
　　F9 之后：   
00ED0766   CD 68            INT 68   
　　shift+f9：   
00ED07F2   0000             ADD BYTE PTR DS:\[EAX\],AL   
　　在 7f2 处 shift+f9 之后软件就运行了!   
　　restart，在 ed07f2 停下来后，堆栈中有 SEH 结构，之前玩   
一个 unpackme 就有对 SEH 的跟踪，现在刚好来复习一下。堆栈   
内容：   
0012FFBC   0012FFE0  Pointer to next SEH record   
0012FFC0   00ED0795  SE handler   
0012FFC4   77E5EB69  RETURN to kernel32.77E5EB69   
　　去 ed0795 处下断，shift+f9 去到 ed0795 处。去掉断点，由于   
试过 tc eip<00ed0000 不行，所以要 f7 一步步跟了：   
00ED0795   55               PUSH EBP   
00ED0796   8BEC             MOV EBP,ESP   
00ED0798   57               PUSH EDI   
00ED0799   8B45 10          MOV EAX,DWORD PTR SS:\[EBP+10\]   
00ED079C   8BB8 C4000000    MOV EDI,DWORD PTR DS:\[EAX+C4\]   
00ED07A2   FF37             PUSH DWORD PTR DS:\[EDI\]   
00ED07A4   33FF             XOR EDI,EDI   
00ED07A6   64:8F07          POP DWORD PTR FS:\[EDI\]   
00ED07A9   8380 C4000000 08 ADD DWORD PTR DS:\[EAX+C4\],8   
00ED07B0   8BB8 A4000000    MOV EDI,DWORD PTR DS:\[EAX+A4\]   
00ED07B6   C1C7 07          ROL EDI,7                     ==edi\=00ecd001   
00ED07B9   89B8 B8000000    MOV DWORD PTR DS:\[EAX+B8\],EDI   
00ED07BF   B8 00000000      MOV EAX,0   
00ED07C4   5F               POP EDI   
00ED07C5   C9               LEAVE   
00ED07C6   C3               RETN   
　　RETN 要返回到 ntdll 中：   
0012FBF0   77F833A0  RETURN to ntdll.77F833A0   
　　想想 fly 的文章‘模拟跟踪 之 PELock 1.06 脱壳——fly 杀手 unpackme ’   
里面的内容好相似!对 SEH 的跟踪及 RETN 到 ntdll 的方式，都一一对位了。   
而 lzqgj 的回复：   
\========   
实际只要小心 F7 走十几步，就到了这里    
0037478A  ADD DWORD PTR DS:\[EAX+B8\],2    //B8!!!明显是修改 ConText 结构，程序将从这里继续执行。     
原来值为 374986，加 2 后为 374988。    
\========   
　　上面 ed07a9 的代码也对位了。上面 0ed07f2+8=0ed07fa，不过，   
从 ed07f2 开始的一直到尾都是 00! 跟到 ed07b6 时 edi=00ecd001，不   
管 3721，去下个断,F9。   
　　运气不错，断下了：(到第二层壳了,ASPack 壳）   
00ECD001   60               PUSHAD   
00ECD002   E8 03000000      CALL 00ECD00A   
00ECD007  -E9 EB045D45      JMP 4649D4F7   
00ECD00C   55               PUSH EBP   
00ECD00D   C3               RETN   
　　跟着 fly 的文章‘ExeStealth 常用脱壳方法 + ExeStealth V2.72 主程   
序脱壳’，下断 BP GetModuleHandleA(这可是宝贵的经验，不是我的，是   
fly 的，可能 fly 并不觉得宝贵，但我就觉得宝贵了，因为这对于快速脱壳很   
重要，对于特别的壳用特别的断:))   
77E59F93 > 837C24 04 00     CMP DWORD PTR SS:\[ESP+4\],0  ==fly 的原文：//断在这！取消断点 Ctrl+F9 执行到返回    
77E59F98   0F84 23060000    JE kernel32.77E5A5C1        ==ctrl+f9 和 shift+f9 可大不相同，我刚看到这句时就眼花了   
77E59F9E   FF7424 04        PUSH DWORD PTR SS:\[ESP+4\]   ==看成 shift+f9 了 :(   
77E59FA2   E8 55080000      CALL kernel32.77E5A7FC   
77E59FA7   85C0             TEST EAX,EAX   
77E59FA9   74 08            JE SHORT kernel32.77E59FB3   
77E59FAB   FF70 04          PUSH DWORD PTR DS:\[EAX+4\]   
77E59FAE   E8 B0060000      CALL kernel32.GetModuleHandleW   
77E59FB3   C2 0400          RETN 4   ==返回到 ecd042   
　　看看地址，和 ecd001 非常近!　   
　　但和文章里说的不同，指令并不是一个 test eax,eax。只好   
F7 下去了...   
00ECD042   8985 26040000    MOV DWORD PTR SS:\[EBP+426\],EAX           ; kernel32.77E40000   
00ECD048   8BF8             MOV EDI,EAX   
00ECD04A   8D5D 5E          LEA EBX,DWORD PTR SS:\[EBP+5E\]   =='VirtualAlloc',要分配内存   
00ECD04D   53               PUSH EBX   
00ECD04E   50               PUSH EAX   
00ECD04F   FF95 490F0000    CALL DWORD PTR SS:\[EBP+F49\]   ==取 VirtualAlloc 地址到 EAX   
00ECD055   8985 4D050000    MOV DWORD PTR SS:\[EBP+54D\],EAX   
00ECD05B   8D5D 6B          LEA EBX,DWORD PTR SS:\[EBP+6B\]   =='VirtualFree'   
00ECD05E   53               PUSH EBX   
00ECD05F   57               PUSH EDI   
00ECD060   FF95 490F0000    CALL DWORD PTR SS:\[EBP+F49\]   
00ECD066   8985 51050000    MOV DWORD PTR SS:\[EBP+551\],EAX   
00ECD06C   8D45 77          LEA EAX,DWORD PTR SS:\[EBP+77\]  ==00ecd08a   
00ECD06F   FFE0             JMP EAX   
跳过来：   
00ECD08A   8B9D 31050000    MOV EBX,DWORD PTR SS:\[EBP+531\]   
00ECD090   0BDB             OR EBX,EBX   
00ECD092   74 0A            JE SHORT 00ECD09E  ==跳了   
00ECD094   8B03             MOV EAX,DWORD PTR DS:\[EBX\]   
00ECD096   8785 35050000    XCHG DWORD PTR SS:\[EBP+535\],EAX   
00ECD09C   8903             MOV DWORD PTR DS:\[EBX\],EAX   
00ECD09E   8DB5 69050000    LEA ESI,DWORD PTR SS:\[EBP+569\]   
00ECD0A4   833E 00          CMP DWORD PTR DS:\[ESI\],0   
00ECD0A7   0F84 21010000    JE 00ECD1CE           
00ECD0AD   6A 04            PUSH 4   
00ECD0AF   68 00100000      PUSH 1000   
00ECD0B4   68 00180000      PUSH 1800   
00ECD0B9   6A 00            PUSH 0   
00ECD0BB   FF95 4D050000    CALL DWORD PTR SS:\[EBP+54D\]    ==VirtualAlloc   
00ECD0C1   8985 56010000    MOV DWORD PTR SS:\[EBP+156\],EAX   
00ECD0C7   8B46 04          MOV EAX,DWORD PTR DS:\[ESI+4\]   ==0040a000!   
00ECD0CA   05 0E010000      ADD EAX,10E   
00ECD0CF   6A 04            PUSH 4   
00ECD0D1   68 00100000      PUSH 1000   
00ECD0D6   50               PUSH EAX   
00ECD0D7   6A 00            PUSH 0   
00ECD0D9   FF95 4D050000    CALL DWORD PTR SS:\[EBP+54D\]    ==VirtualAlloc   
00ECD0DF   8985 52010000    MOV DWORD PTR SS:\[EBP+152\],EAX   
00ECD0E5   56               PUSH ESI   
00ECD0E6   8B1E             MOV EBX,DWORD PTR DS:\[ESI\]   
00ECD0E8   039D 22040000    ADD EBX,DWORD PTR SS:\[EBP+422\]   
00ECD0EE   FFB5 56010000    PUSH DWORD PTR SS:\[EBP+156\]   
00ECD0F4   FF76 04          PUSH DWORD PTR DS:\[ESI+4\]      ==0040a000   
00ECD0F7   50               PUSH EAX   
00ECD0F8   53               PUSH EBX   
00ECD0F9   E8 6E050000      CALL 00ECD66C    ==F8   
00ECD0FE   B3 00            MOV BL,0   
00ECD100   80FB 00          CMP BL,0   
00ECD103   75 5E            JNZ SHORT 00ECD163     
00ECD105   FE85 EC000000    INC BYTE PTR SS:\[EBP+EC\]   
00ECD10B   8B3E             MOV EDI,DWORD PTR DS:\[ESI\]   
00ECD10D   03BD 22040000    ADD EDI,DWORD PTR SS:\[EBP+422\]   
00ECD113   FF37             PUSH DWORD PTR DS:\[EDI\]   
00ECD115   C607 C3          MOV BYTE PTR DS:\[EDI\],0C3   
00ECD118   FFD7             CALL EDI                  ==call 007e1000 ,f8   
00ECD11A   8F07             POP DWORD PTR DS:\[EDI\]   
00ECD11C   50               PUSH EAX   
00ECD11D   51               PUSH ECX   
00ECD11E   56               PUSH ESI   
00ECD11F   53               PUSH EBX   
00ECD120   8BC8             MOV ECX,EAX   
00ECD122   83E9 06          SUB ECX,6   
00ECD125   8BB5 52010000    MOV ESI,DWORD PTR SS:\[EBP+152\]   
00ECD12B   33DB             XOR EBX,EBX   
00ECD12D   0BC9             OR ECX,ECX   
00ECD12F   74 2E            JE SHORT 00ECD15F      
00ECD131   78 2C            JS SHORT 00ECD15F      
00ECD133   AC               LODS BYTE PTR DS:\[ESI\]   
00ECD134   3C E8            CMP AL,0E8   
00ECD136   74 0A            JE SHORT 00ECD142      
00ECD138   EB 00            JMP SHORT 00ECD13A   
00ECD13A   3C E9            CMP AL,0E9   
00ECD13C   74 04            JE SHORT 00ECD142   
00ECD13E   43               INC EBX   
00ECD13F   49               DEC ECX   
00ECD140  ^EB EB            JMP SHORT 00ECD12D   ==循环   
00ECD142   8B06             MOV EAX,DWORD PTR DS:\[ESI\]  ==来这里 F4 跳出循环   
00ECD144   EB 0A            JMP SHORT 00ECD150   
00ECD146   803E 00          CMP BYTE PTR DS:\[ESI\],0   
00ECD149  ⁷⁵ F3            JNZ SHORT 00ECD13E   
00ECD14B   24 00            AND AL,0   
00ECD14D   C1C0 18          ROL EAX,18   
00ECD150   2BC3             SUB EAX,EBX   
00ECD152   8906             MOV DWORD PTR DS:\[ESI\],EAX   
00ECD154   83C3 05          ADD EBX,5   
00ECD157   83C6 04          ADD ESI,4   
00ECD15A   83E9 05          SUB ECX,5   
00ECD15D  ^EB CE            JMP SHORT 00ECD12D    ==又来循环   
00ECD15F   5B               POP EBX  ==F4 007e1000   
00ECD160   5E               POP ESI   
00ECD161   59               POP ECX   
00ECD162   58               POP EAX  ==0040a000   
00ECD163   EB 08            JMP SHORT 00ECD16D   
跳过来：   
00ECD16D   8BC8             MOV ECX,EAX   
00ECD16F   8B3E             MOV EDI,DWORD PTR DS:\[ESI\]   
00ECD171   03BD 22040000    ADD EDI,DWORD PTR SS:\[EBP+422\]   
00ECD177   8BB5 52010000    MOV ESI,DWORD PTR SS:\[EBP+152\]   
00ECD17D   C1F9 02          SAR ECX,2   
00ECD180   F3:A5            REP MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS\>   
00ECD182   8BC8             MOV ECX,EAX   
00ECD184   83E1 03          AND ECX,3   
00ECD187   F3:A4            REP MOVS BYTE PTR ES:\[EDI\],BYTE PTR DS:\[>   
00ECD189   5E               POP ESI   
00ECD18A   68 00800000      PUSH 8000   
00ECD18F   6A 00            PUSH 0   
00ECD191   FFB5 52010000    PUSH DWORD PTR SS:\[EBP+152\]   
00ECD197   FF95 51050000    CALL DWORD PTR SS:\[EBP+551\]   ==VirtualFree   
00ECD19D   83C6 08          ADD ESI,8   
00ECD1A0   833E 00          CMP DWORD PTR DS:\[ESI\],0   
00ECD1A3  ⁰F85 1EFFFFFF    JNZ 00ECD0C7    ==循环回去了   
00ECD1A9   68 00800000      PUSH 8000       ==这里 f4   
00ECD1AE   6A 00            PUSH 0   
00ECD1B0   FFB5 56010000    PUSH DWORD PTR SS:\[EBP+156\]   
00ECD1B6   FF95 51050000    CALL DWORD PTR SS:\[EBP+551\]   ==VirtualFree 解压完毕   
00ECD1BC   8B9D 31050000    MOV EBX,DWORD PTR SS:\[EBP+531\]   
00ECD1C2   0BDB             OR EBX,EBX   
00ECD1C4   74 08            JE SHORT 00ECD1CE  ==跳了   
00ECD1C6   8B03             MOV EAX,DWORD PTR DS:\[EBX\]   
00ECD1C8   8785 35050000    XCHG DWORD PTR SS:\[EBP+535\],EAX   
00ECD1CE   8B95 22040000    MOV EDX,DWORD PTR SS:\[EBP+422\]  ==00400000   
00ECD1D4   8B85 2D050000    MOV EAX,DWORD PTR SS:\[EBP+52D\]  ==00400000   
00ECD1DA   2BD0             SUB EDX,EAX   
00ECD1DC   74 79            JE SHORT 00ECD257  ==解压 OK?跳了   
  
00ECD257   8B95 22040000    MOV EDX,DWORD PTR SS:\[EBP+422\]           ; BmJapane.00400000   
00ECD25D   8BB5 41050000    MOV ESI,DWORD PTR SS:\[EBP+541\]   ==00400000   
00ECD263   0BF6             OR ESI,ESI   
00ECD265   74 11            JE SHORT 00ECD278  ==跳   
  
00ECD278   BE 28EF7E00      MOV ESI,7EEF28   
00ECD27D   8B95 22040000    MOV EDX,DWORD PTR SS:\[EBP+422\]   
00ECD283   03F2             ADD ESI,EDX   
00ECD285   8B46 0C          MOV EAX,DWORD PTR DS:\[ESI+C\]     
00ECD288   85C0             TEST EAX,EAX   
00ECD28A   0F84 0A010000    JE 00ECD39A     
00ECD290   03C2             ADD EAX,EDX   
00ECD292   8BD8             MOV EBX,EAX   
00ECD294   50               PUSH EAX   
00ECD295   FF95 4D0F0000    CALL DWORD PTR SS:\[EBP+F4D\]   
00ECD29B   85C0             TEST EAX,EAX   
00ECD29D   75 07            JNZ SHORT 00ECD2A6  ==跳了（上面一段功能是得到 kernel32.dll 的地址）   
  
00ECD2A6   8985 45050000    MOV DWORD PTR SS:\[EBP+545\],EAX           ; kernel32.77E40000   
00ECD2AC   C785 49050000 00>MOV DWORD PTR SS:\[EBP+549\],0   
00ECD2B6   8B95 22040000    MOV EDX,DWORD PTR SS:\[EBP+422\]   
00ECD2BC   8B06             MOV EAX,DWORD PTR DS:\[ESI\]   
00ECD2BE   85C0             TEST EAX,EAX   
00ECD2C0   75 03            JNZ SHORT 00ECD2C5     
00ECD2C2   8B46 10          MOV EAX,DWORD PTR DS:\[ESI+10\]   
00ECD2C5   03C2             ADD EAX,EDX   
00ECD2C7   0385 49050000    ADD EAX,DWORD PTR SS:\[EBP+549\]   
00ECD2CD   8B18             MOV EBX,DWORD PTR DS:\[EAX\]   
00ECD2CF   8B7E 10          MOV EDI,DWORD PTR DS:\[ESI+10\]   
00ECD2D2   03FA             ADD EDI,EDX   
00ECD2D4   03BD 49050000    ADD EDI,DWORD PTR SS:\[EBP+549\]   
00ECD2DA   85DB             TEST EBX,EBX   
00ECD2DC   0F84 A2000000    JE 00ECD384     
00ECD2E2   F7C3 00000080    TEST EBX,80000000   
00ECD2E8   75 04            JNZ SHORT 00ECD2EE     
00ECD2EA   03DA             ADD EBX,EDX   
00ECD2EC   43               INC EBX   
00ECD2ED   43               INC EBX   
00ECD2EE   53               PUSH EBX   =='LoadLibraryA'   
00ECD2EF   81E3 FFFFFF7F    AND EBX,7FFFFFFF   
00ECD2F5   53               PUSH EBX   
00ECD2F6   FFB5 45050000    PUSH DWORD PTR SS:\[EBP+545\]   
00ECD2FC   FF95 490F0000    CALL DWORD PTR SS:\[EBP+F49\]  ==取 LoadLibraryA 地址   
00ECD302   85C0             TEST EAX,EAX                             ; kernel32.LoadLibraryA   
00ECD304   5B               POP EBX   
00ECD305   75 6F            JNZ SHORT 00ECD376  ==跳了   
  
00ECD376   8907             MOV DWORD PTR DS:\[EDI\],EAX               ; kernel32.LoadLibraryA   
00ECD378   8385 49050000 04 ADD DWORD PTR SS:\[EBP+549\],4   
00ECD37F  ^E9 32FFFFFF      JMP 00ECD2B6    ==又要往回跳，去下一行 F4   
00ECD384   8906             MOV DWORD PTR DS:\[ESI\],EAX   
00ECD386   8946 0C          MOV DWORD PTR DS:\[ESI+C\],EAX   
00ECD389   8946 10          MOV DWORD PTR DS:\[ESI+10\],EAX   
00ECD38C   83C6 14          ADD ESI,14   
00ECD38F   8B95 22040000    MOV EDX,DWORD PTR SS:\[EBP+422\]   
00ECD395  ^E9 EBFEFFFF      JMP 00ECD285   ==下一行 F4   
00ECD39A   B8 90AE7E00      MOV EAX,7EAE90   
00ECD39F   50               PUSH EAX   
00ECD3A0   0385 22040000    ADD EAX,DWORD PTR SS:\[EBP+422\]  ==00beae90   
00ECD3A6   59               POP ECX   
00ECD3A7   0BC9             OR ECX,ECX   
00ECD3A9   8985 A8030000    MOV DWORD PTR SS:\[EBP+3A8\],EAX  ==ebp\=ecd013+3a8=ecd3bb,动态改代码!   
00ECD3AF   61               POPAD           ==这个指令可是等了好久的:)   
00ECD3B0   75 08            JNZ SHORT 00ECD3BA  ==跳了   
00ECD3B2   B8 01000000      MOV EAX,1   
00ECD3B7   C2 0C00          RETN 0C   
00ECD3BA   68 00000000      PUSH 0   
\==改后：   
00ECD3BA   68 90AEBE00      PUSH 0BEAE90     
00ECD3BF   C3               RETN   
去到 beae90：(又一个壳？昏倒，典型 UPX 壳)   
00BEAE90   60               PUSHAD   
00BEAE91   BE 00107E00      MOV ESI,7E1000   
00BEAE96   8DBE 0000C2FF    LEA EDI,DWORD PTR DS:\[ESI+FFC20000\]   ==4010000   
00BEAE9C   57               PUSH EDI   
00BEAE9D   83CD FF          OR EBP,FFFFFFFF   
00BEAEA0   EB 10            JMP SHORT 00BEAEB2   
　　对于 UPX 壳，可以直接去 PUSHAD+11c(此处是 beae90+11c=   
beafac)的地方找 POPAD，然后用 F4 来到 POPAD 行。   
00BEAFAC   7E 00            JLE SHORT 00BEAFAE   
00BEAFAE   09C0             OR EAX,EAX   
00BEAFB0   74 07            JE SHORT 00BEAFB9   
00BEAFB2   8903             MOV DWORD PTR DS:\[EBX\],EAX   
00BEAFB4   83C3 04          ADD EBX,4   
00BEAFB7  ^EB E1            JMP SHORT 00BEAF9A   
00BEAFB9   FF96 5CE07E00    CALL DWORD PTR DS:\[ESI+7EE05C\]   
00BEAFBF   61               POPAD   
00BEAFC0  ^E9 3CE0D1FF      JMP 00909001  ==跟   
去 909001,又是壳？口吐泡沫 ing...   
00909001   60               PUSHAD   
00909002   E8 03000000      CALL 0090900A   
00909007  -E9 EB045D45      JMP 45ED94F7   
0090900C   55               PUSH EBP   
0090900D   C3               RETN   
　　由于太烦了，直接了当，去后面找 POPAD，找到下面的   
地方，是不是和刚才第二层壳的尾部非常相似?!   
　　对啦，连 PUSHAD 和 POPAD 的地址都一样：前面的是 ecd001   
\-ecd3af，这里是 909001-9093af，所以，和 UPX 的壳一样，非   
常好找啊:) 到了 PUSHAD 之后直接加上偏移量就行了。(UPX 是   
11c 附近，ASPack 是 3ae)   
009093AF   61               POPAD   
009093B0   75 08            JNZ SHORT 009093BA   
009093B2   B8 01000000      MOV EAX,1   
009093B7   C2 0C00          RETN 0C   
009093BA   68 C4B25F00      PUSH 5FB2C4   
009093BF   C3               RETN   
　　在 9093af F4。F7 到 RETN 后来到 5fb2c4：   
005FB2C4   55               PUSH EBP   
005FB2C5   8BEC             MOV EBP,ESP   
005FB2C7   83C4 F0          ADD ESP,-10   
005FB2CA   53               PUSH EBX   
005FB2CB   B8 B4AE5F00      MOV EAX,5FAEB4   
005FB2D0   E8 8FBCE0FF      CALL 00406F64   
005FB2D5   8B1D 3C696000    MOV EBX,DWORD PTR DS:\[60693C\]   
005FB2DB   8B0B             MOV ECX,DWORD PTR DS:\[EBX\]   
005FB2DD   B2 01            MOV DL,1   
005FB2DF   A1 F4A05500      MOV EAX,DWORD PTR DS:\[55A0F4\]   
005FB2E4   E8 CB4FE8FF      CALL 004802B4   
......   
　　看样子应该是 OEP 吧...试下 dump 出来...   
　　直接用 OllyDump(Dump 了好几分钟，以为死机了:(，看看网页先:) )，   
再来 ImportREC。oep->1fb2c4,get imports,fix dump. 10M 多.运行   
正常。   
　　作者也算用心良苦了，EXEStealth 2.72-2.73 / ASPack / UPX / ASPack   
加了 4 次壳。   
\=====================================================================   
 4dd466d0e1e0cc123afbe504da2462ca