

# Armadillo 3.01

> 原文：[`www.pediy.com/kssd/pediy06/pediy6322.htm`](https://www.pediy.com/kssd/pediy06/pediy6322.htm)

这两天照猫画虎脱了几个 Armadillo 的壳，有了点体会，   
主要是要找到 mgic jump。   
这个壳是标准方式壳，比 COPYMEM-II 要温和不少。COPYMEM-II 方式参照 Ricardo Narvaj 写的 GetRight 脱壳，我试验的 3.10 版的 arm 和他写的文章步骤差不多，就是多了些花指令，还有 magic jump 第一次运行前是动态解码生成。   
  
大概说一下这个软件的脱壳过程。先找原始输入表位置，如在 4EC11C 处下一个内存写断点，中断 1 次后，下断点 GetModuleHandleA    
注意以下地址可能和你的不太一样，请参照内容。   
  
00D15873    55              PUSH    EBP   
00D15874    8BEC            MOV     EBP, ESP   
00D15876    81EC 00010000   SUB     ESP, 100   
00D1587C    53              PUSH    EBX   
00D1587D    56              PUSH    ESI   
00D1587E    57              PUSH    EDI   
00D1587F    33FF            XOR     EDI, EDI   
00D15881    33DB            XOR     EBX, EBX   
00D15883    66:F745 0E FFFF TEST    WORD PTR SS:\[EBP+E\], 0FFFF   
00D15889    75 03           JNZ     SHORT 00D1588E   
00D1588B    8B5D 0C         MOV     EBX, DWORD PTR SS:\[EBP+C\]   
00D1588E    57              PUSH    EDI   
00D1588F    FF15 A450D300   CALL    DWORD PTR DS:\[D350A4\]            ; kernel32.GetModuleHandleA  <======= 中断到这里   
  
00D15895    3945 08         CMP     DWORD PTR SS:\[EBP+8\], EAX        ; M2.00400000   
00D15898    75 07           JNZ     SHORT 00D158A1   
00D1589A    BE C073D300     MOV     ESI, 0D373C0   
00D1589F    EB 60           JMP     SHORT 00D15901   
00D158A1    393D A879D300   CMP     DWORD PTR DS:\[D379A8\], EDI   
00D158A7    B9 A879D300     MOV     ECX, 0D379A8   
00D158AC    74 3C           JE      SHORT 00D158EA             <======== 这就是那个所谓的 magic jump ,改为 jmp,就能得到正确的输入表   
00D158AE    8B35 80D7D300   MOV     ESI, DWORD PTR DS:\[D3D780\]   
00D158B4    A1 8016D400     MOV     EAX, DWORD PTR DS:\[D41680\]   
00D158B9    F641 08 01      TEST    BYTE PTR DS:\[ECX+8\], 1   
00D158BD    74 0E           JE      SHORT 00D158CD   
00D158BF    8B50 68         MOV     EDX, DWORD PTR DS:\[EAX+68\]   
00D158C2    3350 60         XOR     EDX, DWORD PTR DS:\[EAX+60\]   
00D158C5    3350 30         XOR     EDX, DWORD PTR DS:\[EAX+30\]   
00D158C8    F6C2 80         TEST    DL, 80   
00D158CB    75 13           JNZ     SHORT 00D158E0   
00D158CD    8B50 60         MOV     EDX, DWORD PTR DS:\[EAX+60\]   
00D158D0    3350 5C         XOR     EDX, DWORD PTR DS:\[EAX+5C\]   
00D158D3    3350 2C         XOR     EDX, DWORD PTR DS:\[EAX+2C\]   
00D158D6    3350 0C         XOR     EDX, DWORD PTR DS:\[EAX+C\]   
00D158D9    3316            XOR     EDX, DWORD PTR DS:\[ESI\]   
00D158DB    3955 08         CMP     DWORD PTR SS:\[EBP+8\], EDX   
00D158DE    74 1E           JE      SHORT 00D158FE   
00D158E0    83C1 0C         ADD     ECX, 0C   
00D158E3    83C6 04         ADD     ESI, 4   
00D158E6    3939            CMP     DWORD PTR DS:\[ECX\], EDI   
00D158E8  ^ 75 CF           JNZ     SHORT 00D158B9   
00D158EA    FF75 0C         PUSH    DWORD PTR SS:\[EBP+C\]   
00D158ED    FF75 08         PUSH    DWORD PTR SS:\[EBP+8\]   
00D158F0    E8 35FEFFFF     CALL    00D1572A   
00D158F5    59              POP     ECX   
00D158F6    59              POP     ECX   
00D158F7    5F              POP     EDI   
00D158F8    5E              POP     ESI   
00D158F9    5B              POP     EBX   
00D158FA    C9              LEAVE   
00D158FB    C2 0800         RETN    8   
  
  
然后单步走来到这里   
00D2A7B5    6A 01           PUSH    1   
00D2A7B7    58              POP     EAX   
00D2A7B8    85C0            TEST    EAX, EAX   
00D2A7BA    0F84 7B020000   JE      00D2AA3B   
00D2A7C0    83A5 94F7FFFF 0>AND     DWORD PTR SS:\[EBP\-86C\], 0   
00D2A7C7    8B85 94FCFFFF   MOV     EAX, DWORD PTR SS:\[EBP\-36C\]   
00D2A7CD    0FBE00          MOVSX   EAX, BYTE PTR DS:\[EAX\]   
00D2A7D0    85C0            TEST    EAX, EAX   
00D2A7D2    0F85 0C010000   JNZ     00D2A8E4  <======================= 一个 dll 的输入表处理完了吗，若没有 go on，可以在这下面下个断点，没处理完一个 dll 的输入表后就回到这里了，这里特征很明显，还是很好找的   
00D2A7D8    C785 44F7FFFF 5>MOV     DWORD PTR SS:\[EBP\-8BC\], 0D15F5D   
00D2A7E2    C785 48F7FFFF 8>MOV     DWORD PTR SS:\[EBP\-8B8\], 0D15F83   
00D2A7EC    C785 4CF7FFFF A>MOV     DWORD PTR SS:\[EBP\-8B4\], 0D15FA6   
00D2A7F6    C785 50F7FFFF 1>MOV     DWORD PTR SS:\[EBP\-8B0\], 0D1601B   
00D2A800    C785 54F7FFFF B>MOV     DWORD PTR SS:\[EBP\-8AC\], 0D15FB9   
00D2A80A    C785 58F7FFFF B>MOV     DWORD PTR SS:\[EBP\-8A8\], 0D15FBE   
00D2A814    C785 5CF7FFFF C>MOV     DWORD PTR SS:\[EBP\-8A4\], 0D15FC3   
00D2A81E    C785 60F7FFFF C>MOV     DWORD PTR SS:\[EBP\-8A0\], 0D15FC8   
00D2A828    C785 64F7FFFF E>MOV     DWORD PTR SS:\[EBP\-89C\], 0D15FEE   
00D2A832    C785 68F7FFFF 1>MOV     DWORD PTR SS:\[EBP\-898\], 0D16014   
00D2A83C    C785 6CF7FFFF 5>MOV     DWORD PTR SS:\[EBP\-894\], 0D15F5D   
00D2A846    C785 70F7FFFF 8>MOV     DWORD PTR SS:\[EBP\-890\], 0D15F83   
00D2A850    C785 74F7FFFF A>MOV     DWORD PTR SS:\[EBP\-88C\], 0D15FA6   
00D2A85A    C785 78F7FFFF 1>MOV     DWORD PTR SS:\[EBP\-888\], 0D1601B   
00D2A864    C785 7CF7FFFF 5>MOV     DWORD PTR SS:\[EBP\-884\], 0D16058   
00D2A86E    C785 80F7FFFF 5>MOV     DWORD PTR SS:\[EBP\-880\], 0D1605D   
00D2A878    C785 84F7FFFF 6>MOV     DWORD PTR SS:\[EBP\-87C\], 0D16062   
00D2A882    C785 88F7FFFF C>MOV     DWORD PTR SS:\[EBP\-878\], 0D15FC8   
00D2A88C    C785 8CF7FFFF E>MOV     DWORD PTR SS:\[EBP\-874\], 0D15FEE   
00D2A896    C785 90F7FFFF 6>MOV     DWORD PTR SS:\[EBP\-870\], 0D16067   
00D2A8A0    8D8D 64FCFFFF   LEA     ECX, DWORD PTR SS:\[EBP\-39C\]   
00D2A8A6    E8 9567FEFF     CALL    00D11040   
  
  
等输入表处理完以后单步走，碰到一大堆花指令的时候，就要小心了，光明就在眼前了   
  
00D2CF2A    8366 1C 00      AND     DWORD PTR DS:\[ESI+1C\], 0   
00D2CF2E    FF76 18         PUSH    DWORD PTR DS:\[ESI+18\]   
00D2CF31    E8 73E0FEFF     CALL    00D1AFA9   
00D2CF36    59              POP     ECX   
00D2CF37    E9 90000000     JMP     00D2CFCC   
00D2CF3C    E8 6A6BFEFF     CALL    00D13AAB   
00D2CF41    FF15 EC50D300   CALL    DWORD PTR DS:\[D350EC\]            ; kernel32.GetCurrentThreadId   
00D2CF47    A3 8C16D400     MOV     DWORD PTR DS:\[D4168C\], EAX   
00D2CF4C    E8 5F85FEFF     CALL    00D154B0   
00D2CF51    6A 00           PUSH    0   
00D2CF53    E8 51E0FEFF     CALL    00D1AFA9   
00D2CF58    59              POP     ECX   
00D2CF59    E8 074CFFFF     CALL    00D21B65   
00D2CF5E    8BF8            MOV     EDI, EAX   
00D2CF60    A1 8016D400     MOV     EAX, DWORD PTR DS:\[D41680\]   
00D2CF65    8B48 60         MOV     ECX, DWORD PTR DS:\[EAX+60\]   
00D2CF68    3348 28         XOR     ECX, DWORD PTR DS:\[EAX+28\]   
00D2CF6B    3348 04         XOR     ECX, DWORD PTR DS:\[EAX+4\]   
00D2CF6E    03F9            ADD     EDI, ECX   
00D2CF70    8B0E            MOV     ECX, DWORD PTR DS:\[ESI\]   
00D2CF72    85C9            TEST    ECX, ECX   
00D2CF74    75 2F           JNZ     SHORT 00D2CFA5   
00D2CF76    8B78 60         MOV     EDI, DWORD PTR DS:\[EAX+60\]   
00D2CF79    E8 E74BFFFF     CALL    00D21B65   
00D2CF7E    8B0D 8016D400   MOV     ECX, DWORD PTR DS:\[D41680\]       ; M2.00530238   
00D2CF84    FF76 14         PUSH    DWORD PTR DS:\[ESI+14\]   
00D2CF87    8B51 28         MOV     EDX, DWORD PTR DS:\[ECX+28\]   
00D2CF8A    FF76 10         PUSH    DWORD PTR DS:\[ESI+10\]   
00D2CF8D    3351 04         XOR     EDX, DWORD PTR DS:\[ECX+4\]   
00D2CF90    FF76 0C         PUSH    DWORD PTR DS:\[ESI+C\]   
00D2CF93    33D7            XOR     EDX, EDI   
00D2CF95    03C2            ADD     EAX, EDX   
00D2CF97    8B51 58         MOV     EDX, DWORD PTR DS:\[ECX+58\]   
00D2CF9A    3351 34         XOR     EDX, DWORD PTR DS:\[ECX+34\]   
00D2CF9D    33D7            XOR     EDX, EDI   
00D2CF9F    2BC2            SUB     EAX, EDX   
00D2CFA1    FFD0            CALL    EAX   
00D2CFA3    EB 25           JMP     SHORT 00D2CFCA   
00D2CFA5    83F9 01         CMP     ECX, 1   
00D2CFA8    75 22           JNZ     SHORT 00D2CFCC   
00D2CFAA    FF76 04         PUSH    DWORD PTR DS:\[ESI+4\]   
00D2CFAD    FF76 08         PUSH    DWORD PTR DS:\[ESI+8\]   
00D2CFB0    6A 00           PUSH    0   
00D2CFB2    E8 AE4BFFFF     CALL    00D21B65   
00D2CFB7    50              PUSH    EAX   
00D2CFB8    A1 8016D400     MOV     EAX, DWORD PTR DS:\[D41680\]   
00D2CFBD    8B48 60         MOV     ECX, DWORD PTR DS:\[EAX+60\]   
00D2CFC0    3348 58         XOR     ECX, DWORD PTR DS:\[EAX+58\]   
00D2CFC3    3348 34         XOR     ECX, DWORD PTR DS:\[EAX+34\]   
00D2CFC6    2BF9            SUB     EDI, ECX   
00D2CFC8    FFD7            CALL    EDI                              ; M2.004E71CC  <====  OEP＝4E71CC 奔向光明之颠，按照 fly 的说法，   
  
这里有一个快速些的办法到这里，就是在处理完输入表后下断点 GetCurrentThreadId，断下后，清断点，返回，到 call edi 处，下断，然后就到 OEP 了。   
  
好了现在拿出 LordPE,dump 出来。   
还没完，由于 Armadillo 将输入表各 dll 连接处填充了非 0 值，所以拿出 imprec，将两个 dll 函数连接处未识别出的函数 cut 掉，修复一下 dump.exe，即 OK。   
  
特别感谢 heXer 和 shinegood 的文章，才使我对 Armadillo 有了一点认识。  
  
 4dd466d0e1e0cc123afbe504da2462ca