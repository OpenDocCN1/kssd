

# Unpacking\_US\_unpackMe\_5

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-532.htm`](https://www.pediy.com/kssd/pediy07/pediy7-532.htm)

By:来自轻院的狼【+Immlep+】  
Site:http://immlep.blogone.net  
US\_unpackMe\_5 加壳程序:  
附件:US\_unpackMe\_5.rar   
  
算算，还是我写的第一篇比较详细的脱文！一个印象就是写脱文比脱壳难，  
  
US\_unpackMe\_5 是 Unpacking Saga（简称 US）出的来验证成员的五个 UnpackMe 中的一个（去年的事，现在这五个东东我想也代表不了什么了,嘿嘿），US\_unpackMe\_5 是 Forgot 以前写的壳加的，集合了其它一些壳的特点，这个壳脱起来不难，只不过输入表的恢复上还是有点难度，但看清了就没什么了，这里我较为"详细"的说一下这个壳(晕，我整理这篇脱文花了我 N 久，看来写破文真是一件吃力不讨好的事)，首先我先描述一下这个壳：  
反调试(说实在的，这个壳我没有一句一句从头到未的详细看，如果有漏掉的不要见怪)：  
FUNCTION CC CHECK(我喜欢这么叫，嘿嘿)  
EnableWindow  
ZwQueryInformationProcess  
ZwSetInformationThread  
  
1.FUNCTION CC CHECK 会检测壳用到的函数和程序用到的函数的第一个字节是不是被下断点，形式是这样的：  
003B0AB6    8038 CC         CMP BYTE PTR DS:\[EAX\], 0CC      ;EAX＝FUNCTION ADDR  
003B0AB9    74 01           JE SHORT 003B0ABC                  
2.EnableWindow  
使用 GetForegroundWindow 获得前景窗口的句柄, 然后调用 EnableWindow, 使其无效，等解码结束后在一次调用 EnableWindow 恢复有效  
  
003B0219    FFD0            CALL GetForegroundWindow                    
003B0236    8985 94394000   MOV DWORD PTR SS:\[EBP+403994\], EAX    
003B023C    6A 00           PUSH 0  
003B023E    FFB5 A4394000   PUSH DWORD PTR SS:\[EBP+4039A4\]  
003B0244    FFD0            CALL USER32.EnableWindow  
003B0246    E8 0A000000     CALL 003B0255  
3.ZwQueryInformationProcess，检测是不是用了调试器，有的话就挂掉了  
4.ZwSetInformationThread，无论如何也不能然这个函数运行的了  
看雪精华里有上面两个函数的详细说明的文章，自己去找找  
  
输入表加密：  
集合了一些壳的特征，有点像 ACP 的，不过比 ACP 的好，有点像 PESPIN 的，不过比 PESPIN 差了些。  
壳在还原之前获取每个函数的地址，并检查了第一个字节是不是被下了断点，  
壳把程序里的函数替换成了壳里的地址(不同函数替换成不同地址的，这里统称为 ADDR1)，ADDR1 里面的指令都是这样的：  
PUSH XXXXXXXX  
XOR DWORD PTR SS:\[ESP\], XXXXXXXX  
RETN     ;返回到 ADDR2            (像 acp)  
再 ADDR2 里是这样的：  
判断函数的第一个字节去的指令有没有 PUSH 和 MOV,如果有的话就把这些指令搬到壳 ADDR2 地址去（像 pespin），然后再跳到指向函数剩下指令的地址。  
来个例子：  
比如调用,本来是：  
CALL \[XXXXX\]  
XXXXX 放的事 MessageBoxA 的地址  
原函数被替换成这样：  
CALL \[XXXXX\]  
XXXXX 里放的地址是 ADDR1：  
PUSH XXXXXXXX  
XOR DWORD PTR SS:\[ESP\], XXXXXXXX  
RETN     ;返回到 ADDR2  
ADDR2：  
jmp MessageBoxA  
  
小花：  
典型的小花  
###########  
;PUSHFD  
;PUSH 63  
;L000:  
;  JNB L007  
;  JMP L004  
;  NOP  
;  NOP  
;L004:  
;  CALL L010  
;  NOP  
;  NOP  
;L007:  
;  JNB L004  
;  NOP  
;  NOP  
;L010:  
;  ADD ESP, 4  
;  JMP L014  
;  NOP  
;  NOP  
;L014:  
;  DEC DWORD PTR SS:\[ESP\]  
;  JNO L017  
;  NOP  
;L017:  
;  JNS L000  
;  JPE L020  
;  NOP  
;L020:  
;  ADD ESP, 4  
;  POPFD  
;  jmp L001  
;  nop   
;L001:  
  
S=9C6A63730BEB02????E806000000????73F7????83C404EB02????FF0C247101??79E07A01??83C4049DEB01??  
R=EB2B90909090909090909090909090909090909090909090909090909090909090909090909090909090909090  
#############  
  
好吧，一些介绍就到此为止，我们还是真枪实弹的来干一次：  
  
忽略异常，od 载入, 去除小花，清爽爽的：  
  
在 GetProcAddress，EnableWindow 下硬件断点（其实你慢慢跟也可以，很快就可以跟到下面的地址，如果你想了解一个壳你就慢慢跟 F7.F7..）  
  
你会在 GetProcAddress 看到壳获取了许多函数的地址，EnableWindow 断下：  
0012FF78   003B0246  /CALL to EnableWindow from 003B0244  
0012FF7C   000B03E6  |hWnd = 000B03E6 ('pediy - unpackMe\_5.exe - \[CPU...',class='hello',wndproc=02663168)  
0012FF80   00000000  \\Enable = FALSE   ;在这里改这个参数的值就是改为 1.使 Enable = TURE，这样 OD 就不会被干掉了！  
  
003B0234    FFD7            CALL NEAR EDI                        ;KERNEL32.GetProcAddress    
003B0236    8985 94394000   MOV DWORD PTR SS:\[EBP+403994\], EAX   ;USER32.EnableWindow  ;GetProcAddres 返回到这里  
003B023C    6A 00           PUSH 0  
003B023E    FFB5 A4394000   PUSH DWORD PTR SS:\[EBP+4039A4\]  
003B0244    FFD0            CALL NEAR EAX                            ; USER32.EnableWindow  
  
  
在 ZwSetInformationThread，ZwQueryInformationProcess 下硬件断点。  
  
这里调用 ZwQueryInformationProcess，我们把 hProcess 的值改了，一般的来说可以不用修改到壳就可以把它干掉还是比较好的方法的。  
  
0012FF68   003B02D2  /CALL to ZwQueryInformationProcess from 003B02D0  
0012FF6C   FFFFFFFE  |hProcess = FFFFFFFF   ;改为非 FFFFFFFF，如 FFFFFFF7，这样就可以防止 ZwQueryInformationProcess 了  
0012FF70   00000007  |InfoClass = 7  
0012FF74   0012FF80  |Buffer = 0012FF80  
0012FF78   00000004  |Bufsize = 4  
0012FF7C   00000000  \\pReqsize = NULL  
  
过一会来到这里，如果你慢跟的会来到这里，可以看到程序原来的 OEP：  
003B0363    60              PUSHAD  
003B0364    8D85 5D304000   LEA EAX, DWORD PTR SS:\[EBP+40305D\]  
003B036A    50              PUSH EAX  
003B036B    33C0            XOR EAX, EAX  
003B036D    64:FF30         PUSH DWORD PTR FS:\[EAX\]  
003B0370    64:8920         MOV DWORD PTR FS:\[EAX\], ESP  
003B0373    8BFE            MOV EDI, ESI                             ; ESI 里的是原来程序的 OEP  
003B0375    B0 E8           MOV AL, 0E8                                
003B0377    F2:AE           REPNE SCAS BYTE PTR ES:\[EDI\]               
003B0379    8BF7            MOV ESI, EDI                               
003B037B    AD              LODS DWORD PTR DS:\[ESI\]                    
003B037C    03F0            ADD ESI, EAX                               
003B037E    66:AD           LODS WORD PTR DS:\[ESI\]                   ; 第二次到这里时异常  
003B0380    66:3D FF25      CMP AX, 25FF                               
003B0384  ^ 75 EF           JNZ SHORT 003B0375  
003B0386    4F              DEC EDI  
003B0387    33C0            XOR EAX, EAX  
003B0389    64:8F00         POP DWORD PTR FS:\[EAX\]                   ; 异常处理后回到这里  
003B038C    83C4 04         ADD ESP, 4  
003B038F    897C24 1C       MOV DWORD PTR SS:\[ESP+1C\], EDI  
003B0393    61              POPAD  
003B0394    C3              RETN  
  
继续会在 ZwSetInformationThread 断下，注意会断两次，第二次不是壳调用的不用去管它！  
  
77F5C288 >  B8 E5000000     MOV EAX, 0E5         ；函数入口  
77F5C28D    BA 0003FE7F     MOV EDX, 7FFE0300  
77F5C292    FFD2            CALL NEAR EDX  
77F5C294    C2 1000         RETN 10  
看看堆栈：  
0012FF70   003B03ED  RETURN to 003B03ED  
  
为了维护堆栈平衡，我们把 ESP 的值加 4，然后跳到 003B03ED 去。  
003B03E4    6A 00           PUSH 0  
003B03E6    6A 00           PUSH 0  
003B03E8    6A 11           PUSH 11  
003B03EA    50              PUSH EAX  
003B03EB    FFD7            CALL NEAR EDI  ;ZwSetInformationThread  
003B03ED    BE 00400400     MOV ESI, 44000;直接转到这里，这样我们就跳过 ZwSetInformationThread 这个函数了  
  
如果慢慢跟的话，会看到：  
  
晕：  
003B0AB6    8038 CC         CMP BYTE PTR DS:\[EAX\], 0CC       EAX： kernel32.LoadLibraryA  
003B0AB9    74 01           JE SHORT 003B0ABC  
  
毁尸灭迹：  
003B0A7F    C603 00         MOV BYTE PTR DS:\[EBX\], 0    ；EBX 0044466D ASCII "KERNEL32.DLL"  
003B0A82    43              INC EBX  
003B0A83    803B 00         CMP BYTE PTR DS:\[EBX\], 0  
003B0A86  ^ 75 F7           JNZ SHORT 003B0A7F  
  
F9 后你会在 GetProcAddress 断下，返回来看看，记得随时清除小花：  
0012FF68   003B06D4  /CALL to GetProcAddress from 003B06D2  
0012FF6C   77E40000  |hModule = 77E40000 (kernel32)  
0012FF70   004446F2  \\ProcNameOrOrdinal = "lstrcpyA"  
  
003B06D2    FFD0            CALL NEAR EAX               ; kernel32.GetProcAddress  
003B06D4    85C0            TEST EAX, EAX  
003B06D6    0F84 85050000   JE 003B0C61  
003B0AB6    8038 CC         CMP BYTE PTR DS:\[EAX\], 0CC  
003B0AB9    74 01           JE SHORT 003B0ABC  
003B0ABB    C3              RETN                  ;003B073C  
003B073C    BB 1E344500     MOV EBX, 45341E  
003B0741    8D8D C92C4000   LEA ECX, DWORD PTR SS:\[EBP+402CC9\]  
003B0747    0319            ADD EBX, DWORD PTR DS:\[ECX\]  
003B0749    8139 3D0E0000   CMP DWORD PTR DS:\[ECX\], 0E3D       ;这个数值知道了吧◎kernel32 的函数输出个数  
003B074F   /0F83 21020000   JNB 003B0976  
003B0755   |53              PUSH EBX  
003B0756   |EB 2B           JMP SHORT 003B0783  
.....(n 个 nop)  
003B0783   |57              PUSH EDI  
003B0784   |EB 2B           JMP SHORT 003B07B1  
.....(n 个 nop)  
003B07B1    51              PUSH ECX  
003B07B2    8A10            MOV DL, BYTE PTR DS:\[EAX\]       ;指向函数的地址  
003B07B4    80FA 50         CMP DL, 50                      ;比较第一个字节是不 PUSH 指令  
003B07B7    72 05           JB SHORT 003B07BE               ;这里改为 JMP SHORT 003B07D2,让它不把函数的字节搬到壳里 ADDR2－－－记下这个地址  
003B07B9    80FA 5F         CMP DL, 5F                      ;拿个指令对照表，你自己看看把  
003B07BC   |76 3A           JBE SHORT 003B07F8                
003B07BE    80FA 6A         CMP DL, 6A  
003B07C1    74 39           JE SHORT 003B07FC  
003B07C3    80FA 68         CMP DL, 68  
003B07C6    74 38           JE SHORT 003B0800  
003B07C8    80FA B0         CMP DL, 0B0  
003B07CB    72 05           JB SHORT 003B07D2                ;003B07D2  
....(n 个 nop+XXXX 指令)  
003B0887    6A 05                   PUSH 5                                     // <－－－－－－－－－－－记下这个地址 要改为 PUSH 4   
003B0889    5A                      POP EDX              ;EDX=5           
003B088A    EB 2B                   JMP SHORT 003B08B7  
....(n 个 nop)  
003B08B7    2BC3                    SUB EAX, EBX                                // <－－－－－－－－－－－记下这个地址  
003B08B9    2BC2                    SUB EAX, EDX                        // 取得函数的相对地址  
003B08BB    C603 E9                 MOV BYTE PTR DS:\[EBX\], 0E9          //jmp，其实合起来就是 jmp \[函数地址\]共五个字节  
003B08BE    8943 01                 MOV DWORD PTR DS:\[EBX+1\], EAX               //等一下这里要做一下手脚  
003B08C1    0111                    ADD DWORD PTR DS:\[ECX\], EDX                    //递增五个字节，我们把 jmp 指令去掉所以 4 个字节， PUSH 4  
003B08C3    EB 2B                   JMP SHORT 003B08F0  
  
向下不很近的地方，把函数的的一些代码偷到 ADDR2 里去：  
003B08F2    60                          PUSHAD  
003B08F3    E8 15000000                 CALL 003B090D  
003B08F8    68 6FD9444A                 PUSH 4A44D96F                           ;这两句是 ADDR1 里指令的样本  
003B08FD    813424 71ED014A             XOR DWORD PTR \[ESP\], 4A01ED71          ;这两句是 ADDR1 里指令的样本  
003B0904    C3                          RETN  
003B0905    0D 009700F3                 OR EAX, F3009700  
003B090A    0F0000                      SLDT WORD PTR \[EAX\]  
003B090D    5E                          POP ESI  
003B090E    50                          PUSH EAX  
003B090F    FF95 8C394000               CALL NEAR DWORD PTR \[EBP+40398C\]         ; KERNEL32.GetTickCount  
003B0915    C1C8 07                     ROR EAX, 7                               ; 取随机数  
003B0918    8985 C8354000               MOV DWORD PTR \[EBP+4035C8\], EAX          ; 把取得的地址付给 XOR 里的数值  
003B091E    5A                          POP EDX  
003B091F    33D0                        XOR EDX, EAX                             ; 和函数跳转地址 XOR 然后再一次付给 push 里的数值  
003B0921    8995 C1354000               MOV DWORD PTR \[EBP+4035C1\], EDX  
003B0927    B9 0D000000                 MOV ECX, 0D                              ; 全部 OD 个字节  
003B092C    60                          PUSHAD  
003B092D    8DBD D1354000               LEA EDI, DWORD PTR \[EBP+4035D1\]  
003B0933    390F                        CMP DWORD PTR \[EDI\], ECX                 ; 看看 ECX 是不是为零  
003B0935    77 1E                       JA SHORT 003B0955                        ;不要跳    <－－－－－－－－－－－记下这个地址  
003B0937    B9 00100000                 MOV ECX, 1000  
003B093C    51                          PUSH ECX  
003B093D    6A 04                       PUSH 4  
003B093F    68 00300000                 PUSH 3000  
003B0944    51                          PUSH ECX  
003B0945    6A 00                       PUSH 0  
003B0947    FF15 12344500               CALL NEAR DWORD PTR \[<&KERNEL32.VirtualAlloc>\]     ; kernel32.VirtualAlloc  
003B094D    59                          POP ECX  
003B094E    890F                        MOV DWORD PTR \[EDI\], ECX  
003B0950    83EF 04                     SUB EDI, 4  
003B0953    8907                        MOV DWORD PTR \[EDI\], EAX  
003B0955    61                          POPAD  
003B0956    8BBD CD354000               MOV EDI, DWORD PTR \[EBP+4035CD\]  
003B095C    57                          PUSH EDI                                            ; 要搬到什么地址（ADDR1）里去  
003B095D    51                          PUSH ECX  
003B095E    9C                          PUSHFD  
003B095F    FC                          CLD  
003B0960    F3:A4                       REP MOVS BYTE PTR ES:\[EDI\], BYTE PTR \[ESI\]          ; 这里开始搬了啊  
003B0962    9D                          POPFD  
003B0963    59                          POP ECX  
003B0964    298D D1354000               SUB DWORD PTR \[EBP+4035D1\], ECX  
003B096A    018D CD354000               ADD DWORD PTR \[EBP+4035CD\], ECX                      ; 搬完成了目的地址加 0D  
003B0970    58                          POP EAX                                     <－－－－－－－－－－－记下这个地址  
003B0971    894424 1C                   MOV DWORD PTR \[ESP+1C\], EAX       ;EAX 里放的是 ADDR1，\[ESP+1C\]放的是 ADDR2 的地址，记下 ADDR2 的首地址，  
                                                      ;也就是还原第一个函数时 ADDR2 的地址  
003B0975    61                          POPAD  
003B0976    3385 E0394000               XOR EAX, DWORD PTR \[EBP+4039E0\]           
003B097C    8907                        MOV DWORD PTR \[EDI\], EAX   
003B097E    8385 E4394000 04            ADD DWORD PTR \[EBP+4039E4\], 4  
003B0985    61                          POPAD  
  
整理一下，第二次载入壳，首先在  
GetProcAddress，EnableWindow，ZwSetInformationThread，ZwQueryInformationProcess 下硬件断点  
躲过反跟踪，处理完 ZwQueryInformationProcess 函数后跳到刚才记下的地址，修改：  
1.003B07B7    72 05           JB SHORT 003B07BE   
2.003B0887    6A 05           PUSH 5     
3.003B08B7    2BC3            SUB EAX, EBX   
4.003B0935    77 1E           JA SHORT 003B0955   
5.003B0971    894424 1C       MOV DWORD PTR \[ESP+1C\], EAX  
修改：  
1.  
003B07B1    51                      PUSH ECX  
003B07B2    8A10                    MOV DL, BYTE PTR DS:\[EAX\] ;取函数的第一个地址  
003B07B4    80FA 50                 CMP DL, 50  
003B07B7    72 05                   JB SHORT 003B07BE  ; 改为 JMP SHORT 003B07D2,让壳不从函数里搬走任何东西  
  
2.  
003B0887    6A 05                   PUSH 5                        ;改为 PUSH 4   
003B0889    5A                      POP EDX                           
003B088A    EB 2B                   JMP SHORT 003B08B7  
3.  
003B08B7    2BC3                    SUB EAX, EBX                                          //  
003B08B9    2BC2                    SUB EAX, EDX                            // 取得函数的先对地址  
003B08BB    C603 E9                 MOV BYTE PTR DS:\[EBX\], 0E9              //jmp，其实合起来就是 jmp \[函数地址\]  
003B08BE    8943 01                 MOV DWORD PTR DS:\[EBX+1\], EAX                       ;EAX 里放的是函数的地址，EBX＝000440F0  
003B08C1    0111                    ADD DWORD PTR DS:\[ECX\], EDX  
改为：  
003B08B7    90                      NOP  
003B08B8    90                      NOP  
003B08B9    90                      NOP  
003B08BA    90                      NOP  
003B08BB    90                      NOP  
003B08BC    90                      NOP  
003B08BD    90                      NOP  
003B08BE    8903                    MOV DWORD PTR DS:\[EBX\], EAX                         ;让壳直接放入函数的地址到 ADDR2  
003B08C0    90                      NOP  
003B08C1    0111                    ADD DWORD PTR DS:\[ECX\], EDX  
  
4.  
003B090E    50              PUSH EAX  
003B090F    FF95 8C394000   CALL NEAR DWORD PTR SS:\[EBP+40398C\]      ; kernel32.GetTickCount  
003B0915    C1C8 07         ROR EAX, 7                             
003B0918    8985 C8354000   MOV DWORD PTR SS:\[EBP+4035C8\], EAX  
003B091E    5A              POP EDX  
003B091F    33D0            XOR EDX, EAX  
003B0921    8995 C1354000   MOV DWORD PTR SS:\[EBP+4035C1\], EDX  
003B0927    B9 0D000000     MOV ECX, 0D  
003B092C    60              PUSHAD  
003B092D    8DBD D1354000   LEA EDI, DWORD PTR SS:\[EBP+4035D1\]  
003B0933    390F            CMP DWORD PTR DS:\[EDI\], ECX  
003B0935    77 1E           JA SHORT 003B0955                       ;这里 nop 掉  
  
5.  
  
003B0970    58                          POP EAX                                       
003B0971    894424 1C                   MOV DWORD PTR \[ESP+1C\], EAX                  ;EAX 里放的是 ADDR1，\[ESP+1C\]放的是 ADDR2 的地址  
003B0975    61                          POPAD  
003B0976    3385 E0394000               XOR EAX, DWORD PTR \[EBP+4039E0\]           
003B097C    8907                        MOV DWORD PTR \[EDI\], EAX   
  
  
改成这样：  
003B0970    58                      POP EAX  
003B0971    61                      POPAD               
003B0972    8B4424 1C               MOV EAX, DWORD PTR SS:\[ESP+1C\]                   ;让壳直接把函数的值放到 ADDR2  
003B0976    90                      NOP  
003B0977    90                      NOP  
003B0978    90                      NOP  
003B0979    90                      NOP  
003B097A    90                      NOP  
003B097B    90                      NOP  
  
F9 再一次在 EnableWindow 断下，返回到壳，F7 不久就到 OEP=00441270  
  
00453057    68 6F124400      PUSH unpackMe.0044126F  
0045305C    EB 01            JMP SHORT unpackMe.0045305F  
0045305F    58               POP EAX  
00453060    40               INC EAX  
00453061    50               PUSH EAX  
00453062    C3               RETN        ;返回到 OEP  
  
RAV 填上;0005341E（ADDR2 的首地址）  
大小：1000  
getimports,无效的 cut 掉,fixdump,ok  
  
  
thanks 你看完全文！  

 4dd466d0e1e0cc123afbe504da2462ca