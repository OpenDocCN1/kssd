

# ASPROTECT 2.x 脱壳系列(二)

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-676.htm`](https://www.pediy.com/kssd/pediy07/pediy7-676.htm)

DFCGASPROTECT 2.x 脱壳系列(二)  
  
【目    标】:DVD Fab Gold 2.9.3.5  
【工    具】:Olydbg1.1(diy 版)、LORDPE、ImportREC1.6F  
【任    务】:简单的脱壳  
【操作平台】:Windows XP sp2  
【作    者】:LOVEBOOM\[DFCG\]\[FCG\]\[US\]  
【相关链接】:自己搜索下  
【简要说明】:其实这个版本的还是更先脱的，只是整理的时候更后整理出来。看这篇文章之前建议先看我的第一篇文章，否则有些过程可能不好理解。  
  
【详细过程】:  
  话也不多说，直接入主题， 载入目标程序，去除调试标志，下断 bp GetModuleHandleA，运行中断两次，然后执行到完全解压出壳的代码:  
00AF24AC    85C0            TEST    EAX, EAX                         ; 第二次返回到这里  
00AF24AE    75 07           JNZ     SHORT 00AF24B7  
00AF24B0    53              PUSH    EBX  
00AF24B1    FF95 F0314400   CALL    DWORD PTR SS:\[EBP+4431F0\]        ; kernel32.LoadLibraryA  
......  
00AF25BB    8985 112F4400   MOV     DWORD PTR SS:\[EBP+442F11\], EAX  
00AF25C1    61              POPAD                                    ; 直接 f4 执行到这里  
00AF25C2    75 08           JNZ     SHORT 00AF25CC  
00AF25C4    B8 01000000     MOV     EAX, 1  
00AF25C9    C2 0C00         RETN    0C  
00AF25CC    68 A08FAE00     PUSH    0AE8FA0                          ; 解压完跳去壳的代码入口点  
00AF25D1    C3              RETN  
00AF25D2    8B85 DC304400   MOV     EAX, DWORD PTR SS:\[EBP+4430DC\]   ; kernel32.7C800000  
......  
00AE8FA0    55               PUSH    EBP                              ; 壳代码入口点  
00AE8FA1    8BEC             MOV     EBP, ESP  
00AE8FA3    83C4 B4          ADD     ESP, -4C  
00AE8FA6    B8 988DAE00      MOV     EAX, 0AE8D98  
  
到了壳的入口点后直接搜索以下命令:  
  MOV     DWORD PTR SS:\[EBP\], EAX  
  PUSH    0A  
找到一处：  
00AE653F    8945 00          MOV     DWORD PTR SS:\[EBP\], EAX          ; 找到这里  
00AE6542    6A 0A            PUSH    0A  
00AE6544    E8 63C4FEFF      CALL    00AD29AC  
00AE6549    8BC8             MOV     ECX, EAX  
  
找到地址后，在上面的地址处下硬件断点，执行发生中断后取消硬件断点。  
  
再找一下存放相关数据的地址,我选择的是 513100.找到后，写段保存数据代码：  
00AE653F   /EB 43            JMP     SHORT 00AE6584                   ; 找到这里  
00AE6541   |90               NOP  
00AE6542   |6A 0A            PUSH    0A  
00AE6544   |E8 63C4FEFF      CALL    00AD29AC  
00AE6549   |8BC8             MOV     ECX, EAX  
00AE654B   |038B E4000000    ADD     ECX, DWORD PTR DS:\[EBX+E4\]  
00AE6551   |8BD6             MOV     EDX, ESI  
00AE6553   |8BC3             MOV     EAX, EBX  
00AE6555   |E8 9EE5FFFF      CALL    00AE4AF8  
00AE655A   |FF0C24           DEC     DWORD PTR SS:\[ESP\]  
00AE655D   |03B3 E4000000    ADD     ESI, DWORD PTR DS:\[EBX+E4\]  
00AE6563   |833C24 00        CMP     DWORD PTR SS:\[ESP\], 0  
00AE6567  ^|0F87 55FEFFFF    JA      00AE63C2  
00AE656D   |53               PUSH    EBX                              ; 写好保存相关数据代码后，在这里下 F2 断点  
00AE656E   |E8 5D000000      CALL    00AE65D0  
00AE6573   |0183 EC000000    ADD     DWORD PTR DS:\[EBX+EC\], EAX  
00AE6579   |B0 01            MOV     AL, 1  
00AE657B   |83C4 24          ADD     ESP, 24  
00AE657E   |5D               POP     EBP                              ; 00ADBDC8  
00AE657F   |5F               POP     EDI                              ; 00ADBDC8  
00AE6580   |5E               POP     ESI                              ; 00ADBDC8  
00AE6581   |5B               POP     EBX                              ; 00ADBDC8  
00AE6582   |C3               RETN  
00AE6583   |90               NOP  
00AE6584   \\53               PUSH    EBX                              ; 保护堆栈  
00AE6585    51               PUSH    ECX  
00AE6586    B9 00315100      MOV     ECX, 513100                      ; 保存相关数据的起始地址  
00AE658B    8339 00          CMP     DWORD PTR DS:\[ECX\], 0  
00AE658E    75 06            JNZ     SHORT 00AE6596  
00AE6590    C701 10315100    MOV     DWORD PTR DS:\[ECX\], 513110       ; 如果是第次则初始化，从 513110 处开始保存相关数据  
00AE6596    8B19             MOV     EBX, DWORD PTR DS:\[ECX\]          ; wsock32.#1139  
00AE6598    4D               DEC     EBP                              ; DVDFabGo.004EE4A6  
00AE6599    892B             MOV     DWORD PTR DS:\[EBX\], EBP          ; 保存相关地址  
00AE659B    83C3 04          ADD     EBX, 4  
00AE659E    8919             MOV     DWORD PTR DS:\[ECX\], EBX  
00AE65A0    45               INC     EBP                              ; DVDFabGo.004EE4A6  
00AE65A1    59               POP     ECX                              ; 00ADBDC8  
00AE65A2    5B               POP     EBX                              ; 00ADBDC8  
00AE65A3    8945 00          MOV     DWORD PTR SS:\[EBP\], EAX          ; 执行原壳的代码  
00AE65A6  ^ EB 9A            JMP     SHORT 00AE6542  
00AE65A8    C1E3 18          SHL     EBX, 18  
00AE65AB    31D8             XOR     EAX, EBX  
  
写好代码后在 00AE656D 处下 F2 断点，再 F9 运行，运行中断后，把修复代码还原回去，第三次出现机器码时写上注册信息：  
  
\--------------------第一次----------------------  
0012FF1C   0012FF28  Pointer to next SEH record  
0012FF20   00ADC92E  SE handler  
0012FF24   00000002  
0012FF28   0012FF80  Pointer to next SEH record  
0012FF2C   00ADD739  SE handler  
0012FF30   0012FF78  
0012FF34   00AC0000  
0012FF38   00A80000  
0012FF3C   00ADBDC8  
0012FF40   00000000  
0012FF44   00B59E34  ASCII "6uCokAAAJmg="  
0012FF48   00B59E00  ASCII "2A0FFBAB-CB5A"  
0012FF4C   C5FC9640  
0012FF50   41B05FF0  
0012FF54   F81D0E0A  
......  
\--------------------第二次----------------------  
0012FF1C   0012FF28  Pointer to next SEH record  
0012FF20   00ADCAD5  SE handler  
0012FF24   00B3045C  
0012FF28   0012FF80  Pointer to next SEH record  
0012FF2C   00ADD739  SE handler  
0012FF30   0012FF78  
0012FF34   00AC0000  
0012FF38   00A80000  
0012FF3C   00ADBDC8  
0012FF40   00000000  
0012FF44   00B59E34  ASCII "6uCokAAAJmg="  
0012FF48   00B59E00  ASCII "2A0FFBAB-CB5A"  
0012FF4C   C5FC9640  
0012FF18   0012FF28  Pointer to next SEH record  
0012FF1C   00ADCEE7  SE handler  
0012FF20   00B3045C  
0012FF24   00B59E00  ASCII "2A0FFBAB-CB5A"  
0012FF28   0012FF80  Pointer to next SEH record  
0012FF2C   00ADD739  SE handler  
0012FF30   0012FF78  
0012FF34   00AC0000  
0012FF38   00A80000  
0012FF3C   00ADBDC8  
0012FF40   00000000  
0012FF44   00B59E34  ASCII "6uCokAAAJmg="  
0012FF48   00B59E00  ASCII "2A0FFBAB-CB5A"  
0012FF4C   C5FC9640  
......  
\--------------------第三次----------------------  
0012FF18   0012FF28  Pointer to next SEH record  
0012FF1C   00ADCEE7  SE handler  
0012FF20   00B3045C  
0012FF24   00B59E00  ASCII "2A0FFBAB-CB5A"  
0012FF28   0012FF80  Pointer to next SEH record  
0012FF2C   00ADD739  SE handler  
0012FF30   0012FF78  
0012FF34   00AC0000  
0012FF38   00A80000  
0012FF3C   00ADBDC8  
0012FF40   00000000  
0012FF44   00B59E34  ASCII "6uCokAAAJmg="  
0012FF48   00B59E00  ASCII "2A0FFBAB-CB5A"  
0012FF4C   C5FC9640  
  
第三次出现机器码后，在 code 段访问断点,shift+f9 后中断在这里:  
00424CD7    8B4424 04        MOV     EAX, DWORD PTR SS:\[ESP+4\]        ; 中断在这里,\[esp+4\]保存注册名地址  
00424CDB    81EC 08020000    SUB     ESP, 208  
......  
0012FF1C   00ADD190  RETURN to 00ADD190  
0012FF20   00AC3A29  
0012FF24   00B59E00  ASCII "2A0FFBAB-CB5A"  
0012FF28   0012FF80  Pointer to next SEH record  
修改为:  
0012FF1C   00ADD190  RETURN to 00ADD190  
0012FF20   00479F70  ASCII "loveboom\[DFCG\]\[FCG\]\[US\]"  
0012FF24   00B59E00  ASCII "2A0FFBAB-CB5A"  
0012FF28   0012FF80  Pointer to next SEH record  
  
这样注册块就处理好了，再过几个异常后直接 F2 断到程序 OEP：  
  
0047550E    55               PUSH    EBP                              ; OEP  
0047550F    8BEC             MOV     EBP, ESP  
00475511    6A FF            PUSH    -1  
......  
00AC2544    85C0             TEST    EAX, EAX  
00AC2546    74 0A            JE      SHORT 00AC2552  
00AC2548    FF15 18A0AE00    CALL    DWORD PTR DS:\[AEA018\]  
00AC254E    09C0             OR      EAX, EAX  
00AC2550    74 01            JE      SHORT 00AC2553  
......  
00D500EB    5C               POP     ESP  
00D500EC  - FF6424 FC        JMP     DWORD PTR SS:\[ESP\-4\]    ;patch   
00D500F0    EB 01            JMP     SHORT 00D500F3  
  
  
  
再找一下找到输入表信息:  
IAT 位置:47A000 SIZE:bb0  
  
  
PATCH 1:       00ac2548  
PATCH 2:      00D500EC  
PATCH 3:      4799b0(保存修复代码的起始地址)  
修改跳转地址为:      004799BF  
  
  
写上全部的修复代码:  
  
004799B0   .  60            PUSHAD                                   ;  保存堆栈  
004799B1   .  B9 10315100   MOV     ECX, 00513110                    ;  取出相关的地址  
004799B6   >  8B19          MOV     EBX, DWORD PTR DS:\[ECX\]  
004799B8   .  83FB 00       CMP     EBX, 0  
004799BB   .  74 15         JE      SHORT 004799D2                   ;  如果取完了则跳去修复 jmp \[addr\]  
004799BD   .  FFE3          JMP     EBX                              ;  执行原 CALL  
004799BF   .  8B15 08315100 MOV     EDX, DWORD PTR DS:\[513108\]       ;  取出 call \[addr\]中的 addr  
004799C5   .  66:C703 FF15  MOV     WORD PTR DS:\[EBX\], 15FF          ;  修改成 call \[addr\]  
004799CA   .  8953 02       MOV     DWORD PTR DS:\[EBX+2\], EDX        ;  <&winmm.PlaySound>  
004799CD   .  83C1 04       ADD     ECX, 4  
004799D0   .^ EB E4         JMP     SHORT 004799B6  
004799D2   >  33C0          XOR     EAX, EAX  
004799D4   .  B0 E8         MOV     AL, 0E8                          ;  修复 jmp \[addr\]  
004799D6   .  BF 00104000   MOV     EDI, 00401000                    ;  code 段的起始地址  
004799DB   .  B9 A0990600   MOV     ECX, 699A0                       ;  大小  
004799E0   >  F2:AE         REPNE   SCAS BYTE PTR ES:\[EDI\]  
004799E2   .  83F9 00       CMP     ECX, 0  
004799E5   .  74 3C         JE      SHORT 00479A23                   ;  处理完则跳  
004799E7   .  8B1F          MOV     EBX, DWORD PTR DS:\[EDI\]  
004799E9   .  8D5C3B 04     LEA     EBX, DWORD PTR DS:\[EBX+EDI+4\]  
004799ED   .  81FB 00104000 CMP     EBX, 00401000                    ;  Entry address  
004799F3   .^ 72 EB         JB      SHORT 004799E0  
004799F5   .  81FB 94994700 CMP     EBX, 00479994  
004799FB   .^ 77 E3         JA      SHORT 004799E0  
004799FD   .  66:813B FF15  CMP     WORD PTR DS:\[EBX\], 15FF  
00479A02   .^ 75 DC         JNZ     SHORT 004799E0  
00479A04   .  817B 02 00A04>CMP     DWORD PTR DS:\[EBX+2\], <&advapi32>  
00479A0B   .^ 72 D3         JB      SHORT 004799E0  
00479A0D   .  817B 02 C0AB4>CMP     DWORD PTR DS:\[EBX+2\], 0047ABC0  
00479A14   .^ 77 CA         JA      SHORT 004799E0                   ;  让代码更精确些  
00479A16   .  66:C703 FF25  MOV     WORD PTR DS:\[EBX\], 25FF  
00479A1B   .  83C7 04       ADD     EDI, 4  
00479A1E   .  83E9 04       SUB     ECX, 4  
00479A21   .^ EB BD         JMP     SHORT 004799E0  
00479A23   >  B9 10315100   MOV     ECX, 00513110                    ;  再次精确些处理 jmp \[addr\] \*\*\*\*\*\*\*\*\*\*这里是上篇中没有注意到的  
00479A28   >  8B19          MOV     EBX, DWORD PTR DS:\[ECX\]  
00479A2A   .  83FB 00       CMP     EBX, 0  
00479A2D   .  74 2E         JE      SHORT 00479A5D                   ;  操作完则跳  
00479A2F   .  66:8B03       MOV     AX, WORD PTR DS:\[EBX\]  
00479A32   .  66:3D FF15    CMP     AX, 15FF                         ;  判断是否为 call \[addr\]  
00479A36   .  75 20         JNZ     SHORT 00479A58  
00479A38   .  66:8B43 FA    MOV     AX, WORD PTR DS:\[EBX\-6\]          ;  如果是的话，判断前一句代码是否为 jmp \[addr\]  
00479A3C   .  66:3D FF25    CMP     AX, 25FF  
00479A40   .  75 16         JNZ     SHORT 00479A58  
00479A42   .  8B53 FC       MOV     EDX, DWORD PTR DS:\[EBX\-4\]  
00479A45   .  81FA 00A04700 CMP     EDX, <&advapi32.AdjustTokenPrivi>  
00479A4B   .  72 0B         JB      SHORT 00479A58  
00479A4D   .  81FA C0AB4700 CMP     EDX, 0047ABC0  
00479A53   .  77 03         JA      SHORT 00479A58  
00479A55   .  66:8903       MOV     WORD PTR DS:\[EBX\], AX            ;  如果是的话则认为这当前句也是 jmp \[addr\]  
00479A58   >  83C1 04       ADD     ECX, 4  
00479A5B   .^ EB CB         JMP     SHORT 00479A28  
00479A5D   >  61            POPAD  
00479A5E   .  00            DB      00  
00479A5F   .  00            DB      00  
00479A60      689A4700      DD      dumped\_.00479A68  
00479A64   .  00            DB      00  
00479A65      90            NOP  
00479A66      90            NOP  
00479A67      90            NOP  
00479A68   .  60            PUSHAD  
00479A69   .  8BC2          MOV     EAX, EDX                         ;  保存 call \[addr\]中的 addr 到 513108 处  
00479A6B   .  B9 C80B0000   MOV     ECX, 0BC8  
00479A70   .  BF 00A04700   MOV     EDI, <&advapi32.AdjustTokenPrivi>  
00479A75   .  F2:AF         REPNE   SCAS DWORD PTR ES:\[EDI\]  
00479A77   .  83EF 04       SUB     EDI, 4  
00479A7A   .  893D 08315100 MOV     DWORD PTR DS:\[513108\], EDI       ;  ntdll.7C930738  
00479A80   .  61            POPAD  
00479A81      FF15 18A0AE00 CALL    DWORD PTR DS:\[AEA018\]  
00479A87   .  C3            RETN  
00479A88      90            NOP  
  
再写上修复脚本:  
//fixed aspr 2.x  
var addr  
  
  
start:  
  run  
  
l1:  
  cmp eip,00D500EC  
  jne l2  
  mov addr,esp  
  sub addr,4  
  mov \[addr\],004799BF  
  jmp start  
  
l2:  
  ret  
  
  
写好后在 00D500EC 下硬件执行断点，在 00479A5D 下普通 F2 断点，再执行脚本。脱壳完毕！  
破解就像恋爱，这过程没有什么人能帮到你，什么都得靠自己一步一步的走。愿自己的恋爱能和脱壳一样风雨无阻:-).  
  
  
  
  
  
  
Greetz:  
 Fly.Jingulong,yock,tDasm.David.hexer,hmimys,ahao.UFO(brother).alan(sister).all of my   
  
friends and you!  
  
          By loveboom\[DFCG\]\[FCG\]\[US\]  
                      http://blog.csdn.net/bmd2chen  
                  
  
                                        Email:loveboom#163.com 

 4dd466d0e1e0cc123afbe504da2462ca