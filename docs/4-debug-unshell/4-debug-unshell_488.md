

# ASPROTECT 2.x 脱壳系列(一)

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-675.htm`](https://www.pediy.com/kssd/pediy07/pediy7-675.htm)

ASPROTECT 2.x 脱壳系列(一)  
  
【目    标】:DVD Fab Gold 2.9.4.2  
【工    具】:Olydbg1.1(diy 版)、LORDPE、ImportREC1.6F  
【任    务】:简单的脱壳  
【操作平台】:Windows XP sp2  
【作    者】: LOVEBOOM\[DFCG\]\[FCG\]\[US\]  
【相关链接】: 自己搜索下  
【简要说明】: 时间在不断的减少,人越靠近死亡的边缘.人在死亡的边缘线上挣扎.寻求人间仅有的希望,骤觉得到了,又已失去了:-(.再虚幻的网络都要回到现实,但愿大家的现实比虚幻过的更充实.  
  ASPROTECT 比起以前的版本可算是进步不少,脱壳难度大了不少,回头看看自己,感觉太失败了,别人进步了这么多,自己却还在原地踏步走:-(  
  
【详细过程】:  
  想脱这个壳的话,建议对他的历史版本有所了解,那样对看起文章来不会那么吃力.asprotect 2.x 惯用的伎俩:2.x 版本 anti-debug 方面没有什么新变化,很多人说这是这个壳的失败之外,我倒觉得说其是失败之处,倒不如说是作者别有用心:-).代码混淆处理方面做了很大功夫,变形处理和 1.23 可以说基本不是一个层次了,新版不再是只抽程序入口代码,程序入口代码那一个段基本上都抽光了,其它地方也是抽的很利害.输入表也处理的很难修复.我这次挑的目标是没有抽入口代码的程序.所以相对简单些.  
用 OD 载入目标,去除调试标志.打开内存异常项,通过 N 个异常后到程序的入口:  
  
00ADAF43    64:8921         MOV     DWORD PTR FS:\[ECX\], ESP  
00ADAF46    C601 A6         MOV     BYTE PTR DS:\[ECX\], 0A6           ; 最后一次异常  
00ADAF49    5B              POP     EBX                              ; 00B4AC90  
.......  
00ADB09B    E8 807AFEFF     CALL    00AC2B20  
00ADB0A0    E8 73CAFFFF     CALL    00AD7B18                         ; 直接在这里下断然后直接这里 f2 断点  
00ADB0A5    83C4 2C         ADD     ESP, 2C  
00ADB0A8    5D              POP     EBP                              ; 00B4AC90  
  
最后一个异常发生时,在 00adb0a0 处下 f2 断点,然后 shift+f9,断下后 f7 进去,进去之后 CTRL+F9 执行到 RETURN  
00D20000    68 1F1C0627     PUSH    27061C1F                         ; 进到这里  
00D20005    66:81C0 96FC    ADD     AX, 0FC96  
.......  
00D2011C   /0F85 0F000000   JNZ     00D20131  
00D20122   |B0 6A           MOV     AL, 6A  
00D20124   |E9 3E000000     JMP     00D20167                         ; f4 运行到这里  
00D20129   |40              INC     EAX  
......  
00D20192    2BC3            SUB     EAX, EBX  
00D20194    5C              POP     ESP                              ; DVDFabGo.00400000  
00D20195  - FFE0            JMP     EAX                              ; 这里跳去 OEP  
00D20197    40              INC     EAX  
  
  
当然如果你只是想直接到  
oep 的话,最后一个异常发生后直接在 CODE 段下 f2 断点,再 shift+f9 就可以了：  
  
0047572E    55              PUSH    EBP                              ; 程序 OEP  
0047572F    8BEC            MOV     EBP, ESP  
00475731    6A FF           PUSH    -1  
00475733    68 80F14700     PUSH    0047F180  
.....  
到了 OEP 后，我们先定位下 IAT，观察后发现相关的 API 函数并没有什么加密处理，IAT:47A000 SIZE:BC4.  
  
  
api 没有什么特别处理，并不表示容易的，在程序里随便找一下就会看到 call api 到壳里去了。  
  
00475832    E8 C9A78C00     CALL    00D40000  
  
搜索一下发现很多地方都改成这样子了，一个一个去改??,我以前是试过一个一个去改。但那样效率太低了，且容易出错。跟过 aspr 2.x 的应该知道壳运行时会改地址的，因此在 475832 处下写入断点,运行中断：  
00AE6265    8B45 F4         MOV     EAX, DWORD PTR SS:\[EBP\-C\]  
00AE6268    8B40 3C         MOV     EAX, DWORD PTR DS:\[EAX+3C\]  
00AE626B    8B55 FC         MOV     EDX, DWORD PTR SS:\[EBP\-4\]        ; 传递正确的 API 到 EDX 中  
00AE626E    E8 D1130000     CALL    00AE7644                         ; 因是壳会检测相关代码，进去这里面后有地方不会被检测的  
－－－－－－－－－－－－－－－－－－－－进入第一层－－－－－－－－－－－－－－－－－－－－－－－－－－－－  
  00AE7644    55              PUSH    EBP  
  00AE7645    8BEC            MOV     EBP, ESP  
  00AE7647    83C4 E4         ADD     ESP, -1C  
  00AE764A    53              PUSH    EBX  
  00AE764B    56              PUSH    ESI  
  00AE764C    57              PUSH    EDI  
  00AE764D    894D F4         MOV     DWORD PTR SS:\[EBP\-C\], ECX  
  00AE7650    8955 F8         MOV     DWORD PTR SS:\[EBP\-8\], EDX        ; kernel32.GetModuleHandleA  
  00AE7653    8945 FC         MOV     DWORD PTR SS:\[EBP\-4\], EAX  
  00AE7656    33C0            XOR     EAX, EAX  
  00AE7658    8945 F0         MOV     DWORD PTR SS:\[EBP\-10\], EAX  
  00AE765B    B8 00070000     MOV     EAX, 700                         ; 这里还有代码检测的，再次进入  
  00AE7660    E8 DFAEFDFF     CALL    00AC2544  
  00AE7665    8945 E4         MOV     DWORD PTR SS:\[EBP\-1C\], EAX  
  ----------------------------------进入第二层------------------------------------------  
    00AC2544    85C0            TEST    EAX, EAX  
    00AC2546    74 0A           JE      SHORT 00AC2552                   ; 经跟踪发现这里的条件判断在这个程序里是不会跳转的  
    00AC2548    FF15 18A0AE00   CALL    DWORD PTR DS:\[AEA018\]            ; 这里的代码也不会被检测到，因此我把这里做为我们的突破点  
    00AC254E    09C0            OR      EAX, EAX  
    00AC2550    74 01           JE      SHORT 00AC2553  
    00AC2552    C3              RETN  
    00AC2553    B0 01           MOV     AL, 1  
  ---------------------------------------------------------------------------------------  
－－－－－－－－－－－－－－－－－－－－END－－－－－－－－－－－－－－－－－－－－－－－－－－－－  
00AE6273    8945 FC         MOV     DWORD PTR SS:\[EBP\-4\], EAX  
00AE6276    8B45 E0         MOV     EAX, DWORD PTR SS:\[EBP\-20\]  
00AE6279    8B00            MOV     EAX, DWORD PTR DS:\[EAX\]  
00AE627B    E8 D0E6FFFF     CALL    00AE4950  
00AE6280    8BD0            MOV     EDX, EAX  
00AE6282    0255 DF         ADD     DL, BYTE PTR SS:\[EBP\-21\]  
00AE6285    8B4D FC         MOV     ECX, DWORD PTR SS:\[EBP\-4\]  
00AE6288    8B45 F4         MOV     EAX, DWORD PTR SS:\[EBP\-C\]  
00AE628B    E8 80040000     CALL    00AE6710  
00AE6290    8945 FC         MOV     DWORD PTR SS:\[EBP\-4\], EAX  
00AE6293    8B45 F4         MOV     EAX, DWORD PTR SS:\[EBP\-C\]  
00AE6296    8B40 24         MOV     EAX, DWORD PTR DS:\[EAX+24\]  
00AE6299    8B55 F4         MOV     EDX, DWORD PTR SS:\[EBP\-C\]  
00AE629C    0382 E0000000   ADD     EAX, DWORD PTR DS:\[EDX+E0\]  
00AE62A2    0145 1C         ADD     DWORD PTR SS:\[EBP+1C\], EAX  
00AE62A5    8B45 FC         MOV     EAX, DWORD PTR SS:\[EBP\-4\]  
00AE62A8    2B45 1C         SUB     EAX, DWORD PTR SS:\[EBP+1C\]       ; DVDFabGo.00475832  
00AE62AB    83E8 05         SUB     EAX, 5  
00AE62AE    8B55 1C         MOV     EDX, DWORD PTR SS:\[EBP+1C\]       ; DVDFabGo.00475832  
00AE62B1    42              INC     EDX                              ; DVDFabGo.00475833  
00AE62B2    8902            MOV     DWORD PTR DS:\[EDX\], EAX          ; 这里断下，写入 call 的地址  
00AE62B4    EB 01           JMP     SHORT 00AE62B7  
00AE62B6    E8 8B45F883     CALL    84A6A846  
  
找到突破点后，再找这段代码的出口，tc command is:POPFD,条件符合后中断:  
  
00D500B0    9D              POPFD                                    ; 条件中断在这里  
00D500B1    5C              POP     ESP  
00D500B2    F3:             PREFIX REP:                              ; Superfluous prefix  
00D500B3    EB 02           JMP     SHORT 00D500B7  
00D500B5    CD 20           INT     20  
00D500B7    FF6424 FC       JMP     DWORD PTR SS:\[ESP\-4\]             ; 这里跳去执行原程序的 call api,注意这里已经处理过了，不再是简单的 call api 了  
00D500BB    F3:             PREFIX REP:                              ; Superfluous prefix  
00D500BC    EB 02           JMP     SHORT 00D500C0  
00D500BE    CD 20           INT     20  
  
再找一个空闲的地方写上修复代码,考虑到代码可能会长一点，选择代码后的空白地址 479bc0 修复代码起始地址。  
总结一下大概为:  
  
OEP:        47572E  
IAT:        47A000   
SIZE:        BC4  
  
Patch 点一:      00AC2548  
Patch 点二:      00D500B7(这个地址每次运行都会改变的)  
Patch 起始地址:      479BC0  
  
现在唯一没有解决的问题就是，都有哪些地方改成了 CALL    00D40000,当然，可以通过比较死的方法，直接去搜索，我觉得那样可能没有那么准确，所以我选择了直接记录的方式。  
重新加载，载入后，两次 GetModuleHandleA 中断后，让壳的代码完全解出来：  
00AF25C2   /75 08           JNZ     SHORT 00AF25CC  
00AF25C4   |B8 01000000     MOV     EAX, 1  
00AF25C9   |C2 0C00         RETN    0C  
00AF25CC   \\68 A08FAE00     PUSH    0AE8FA0                          ; 壳的执行代码执行点  
00AF25D1    C3              RETN  
......  
00AE8FA0    55              PUSH    EBP                              ; 壳的代码  
00AE8FA1    8BEC            MOV     EBP, ESP  
00AE8FA3    83C4 B4         ADD     ESP, -4C  
  
壳的代码完全解出来后，搜索命令  
  
MOV \[EBP\],EAX  
PUSH 0A  
  
找到相关位置后下硬件执行断点。执行后中断:  
  
00AE653F    8945 00         MOV     DWORD PTR SS:\[EBP\], EAX          ; 找到这里,ebp-1 就正好是 call d40000 的地址  
00AE6542    6A 0A           PUSH    0A  
00AE6544    E8 63C4FEFF     CALL    00AD29AC  
00AE6549    8BC8            MOV     ECX, EAX  
  
现在要要找的就是存放这些地址的空间，ASPR 加壳的程序，最后一个段是空的，我们可以利用下，我选择 514100 开始保存相关数据:  
514100 保存将要保存相关 call 地址的实际地址，514108 保存 call \[address\]中的 address,514100 开始保存相关 call 00d40000 的实际地址。好了，现在写上一段代码来保存相关数据:  
00AE653F   /EB 43           JMP     SHORT 00AE6584                   ; 跳去执行我们的代码  
00AE6541   |90              NOP  
00AE6542   |6A 0A           PUSH    0A  
00AE6544   |E8 63C4FEFF     CALL    00AD29AC  
00AE6549   |8BC8            MOV     ECX, EAX  
00AE654B   |038B E4000000   ADD     ECX, DWORD PTR DS:\[EBX+E4\]  
00AE6551   |8BD6            MOV     EDX, ESI  
00AE6553   |8BC3            MOV     EAX, EBX  
00AE6555   |E8 9EE5FFFF     CALL    00AE4AF8  
00AE655A   |FF0C24          DEC     DWORD PTR SS:\[ESP\]  
00AE655D   |03B3 E4000000   ADD     ESI, DWORD PTR DS:\[EBX+E4\]  
00AE6563   |833C24 00       CMP     DWORD PTR SS:\[ESP\], 0  
00AE6567  ^|0F87 55FEFFFF   JA      00AE63C2  
00AE656D   |53              PUSH    EBX                              ; 写好代码后直接在这里 F2 断点  
00AE656E   |E8 5D000000     CALL    00AE65D0  
00AE6573   |0183 EC000000   ADD     DWORD PTR DS:\[EBX+EC\], EAX  
00AE6579   |B0 01           MOV     AL, 1  
00AE657B   |83C4 24         ADD     ESP, 24  
00AE657E   |5D              POP     EBP                              ; DVDFabGo.004033AD  
00AE657F   |5F              POP     EDI  
00AE6580   |5E              POP     ESI  
00AE6581   |5B              POP     EBX  
00AE6582   |C3              RETN  
00AE6583   |90              NOP  
00AE6584   \\53              PUSH    EBX                              ; 保存堆栈  
00AE6585    51              PUSH    ECX  
00AE6586    B9 00415100     MOV     ECX, 514100                      ; 起始地址  
00AE658B    8339 00         CMP     DWORD PTR DS:\[ECX\], 0  
00AE658E    75 06           JNZ     SHORT 00AE6596  
00AE6590    C701 10415100   MOV     DWORD PTR DS:\[ECX\], 514110       ; 如果是第一次则写入相关数据  
00AE6596    8B19            MOV     EBX, DWORD PTR DS:\[ECX\]  
00AE6598    4D              DEC     EBP                              ; DVDFabGo.004033AD  
00AE6599    892B            MOV     DWORD PTR DS:\[EBX\], EBP          ; 保存 call 的地址  
00AE659B    83C3 04         ADD     EBX, 4  
00AE659E    8919            MOV     DWORD PTR DS:\[ECX\], EBX          ; 保存下次保存数据的地址  
00AE65A0    45              INC     EBP                              ; DVDFabGo.004033AD  
00AE65A1    59              POP     ECX  
00AE65A2    5B              POP     EBX  
00AE65A3    8945 00         MOV     DWORD PTR SS:\[EBP\], EAX          ; 执行壳原来的代码  
00AE65A6  ^ EB 9A           JMP     SHORT 00AE6542  
  
写完代码后直接在 00AE656D 下 F2 断点，断下后，还原 patch 代码，514100 处的数据保存起来(主要是方便一次操作不成功，第二次不用再写代码获取相关数据,当然其实完全可以一次操作成功的)。  
  
获取到了相关的数据后，运行到 OEP 处,然后就可以直接写上完整的修复代码了,在前面总结的 479BC0 处写上修复代码:  
  
  
00479BC0   .  60            PUSHAD                                   ;  保护堆栈，直接定位到这里  
00479BC1   .  B9 10415100   MOV     ECX, 00514110                    ;  把 call 00d40000 改成 call ds:\[addr\]  
00479BC6   >  8B19          MOV     EBX, DWORD PTR DS:\[ECX\]          ;  取出相关地址  
00479BC8   .  83FB 00       CMP     EBX, 0                           ;  判断是否处理完了  
00479BCB   .  74 15         JE      SHORT 00479BE2  
00479BCD   .- FFE3          JMP     EBX                              ;  执行原 call  
00479BCF   .  8B15 08415100 MOV     EDX, DWORD PTR DS:\[514108\]       ;  \[514108\]就是保存 call \[address\] 中的 address  
00479BD5   .  66:C703 FF15  MOV     WORD PTR DS:\[EBX\], 15FF          ;  修复成 call ds:\[address\]  
00479BDA   .  8953 02       MOV     DWORD PTR DS:\[EBX+2\], EDX        ;  填上实际的 address  
00479BDD   .  83C1 04       ADD     ECX, 4  
00479BE0   .^ EB E4         JMP     SHORT 00479BC6  
00479BE2   >  33C0          XOR     EAX, EAX                         ;  这里开始修复 call \[addr\]中实际上是 jmp \[addr\]的部分  
00479BE4   .  B0 E8         MOV     AL, 0E8  
00479BE6   .  BF 00104000   MOV     EDI, <ModuleEntryPoint>          ;  代码段的起始地址  
00479BEB   .  B9 B89B0600   MOV     ECX, 69BB8                       ;  大小  
00479BF0   >  F2:AE         REPNE   SCAS BYTE PTR ES:\[EDI\]  
00479BF2   .  83F9 00       CMP     ECX, 0  
00479BF5   .  74 3C         JE      SHORT 00479C33                   ;  如果处理完则结束过程  
00479BF7   .  8B1F          MOV     EBX, DWORD PTR DS:\[EDI\]  
00479BF9   .  8D5C3B 04     LEA     EBX, DWORD PTR DS:\[EBX+EDI+4\]  
00479BFD   .  81FB 00104000 CMP     EBX, <ModuleEntryPoint>  
00479C03   .^ 72 EB         JB      SHORT 00479BF0  
00479C05   .  81FB BA9B4700 CMP     EBX, 00479BBA  
00479C0B   .^ 77 E3         JA      SHORT 00479BF0  
00479C0D   .  66:813B FF15  CMP     WORD PTR DS:\[EBX\], 15FF  
00479C12   .^ 75 DC         JNZ     SHORT 00479BF0  
00479C14   .  817B 02 00A04>CMP     DWORD PTR DS:\[EBX+2\], 0047A000   ;  再次准确的判断是否为真的要修改的代码  
00479C1B   .^ 72 D3         JB      SHORT 00479BF0  
00479C1D   .  817B 02 C0AB4>CMP     DWORD PTR DS:\[EBX+2\], 0047ABC0  
00479C24   .^ 77 CA         JA      SHORT 00479BF0  
00479C26   .  66:C703 FF25  MOV     WORD PTR DS:\[EBX\], 25FF  
00479C2B   .  83C7 04       ADD     EDI, 4  
00479C2E   .  83E9 04       SUB     ECX, 4  
00479C31   .^ EB BD         JMP     SHORT 00479BF0  
00479C33   >  61            POPAD                                    ;  还原现场  
00479C34   .  00            DB      00  
00479C35   .  00            DB      00                               ;  因为我是操作边写的，这里留多点空间方便修改代码  
00479C36   .  00            DB      00  
00479C37   .  00            DB      00  
00479C38   .  00            DB      00  
00479C39   .  00            DB      00  
00479C3A   .  00            DB      00  
00479C3B   .  00            DB      00  
00479C3C   .  00            DB      00  
00479C3D   .  00            DB      00  
00479C3E   .  00            DB      00  
00479C3F   .  00            DB      00  
00479C40   .  00            DB      00  
00479C41   .  00            DB      00  
00479C42   .  00            DB      00  
00479C43   .  00            DB      00  
00479C44   .  00            DB      00  
00479C45   .  00            DB      00  
00479C46   .  00            DB      00  
00479C47   .  00            DB      00  
00479C48   .  00            DB      00  
00479C49   .  00            DB      00  
00479C4A   .  00            DB      00  
00479C4B   .  00            DB      00  
00479C4C   .  00            DB      00  
00479C4D   .  00            DB      00  
00479C4E   .  00            DB      00  
00479C4F   .  00            DB      00  
00479C50   .  00            DB      00  
00479C51   .  00            DB      00  
00479C52   .  00            DB      00  
00479C53   .  00            DB      00  
00479C54   .  00            DB      00  
00479C55   .  00            DB      00  
00479C56   .  00            DB      00  
00479C57   .  00            DB      00  
00479C58   .  00            DB      00  
00479C59   .  00            DB      00  
00479C5A   .  00            DB      00  
00479C5B   .  00            DB      00  
00479C5C   .  00            DB      00  
00479C5D   .  00            DB      00  
00479C5E   .  00            DB      00  
00479C5F   .  00            DB      00  
00479C60   .  00            DB      00  
00479C61   .  00            DB      00  
00479C62   .  00            DB      00  
00479C63   .  00            DB      00  
00479C64   .  00            DB      00  
00479C65   .  00            DB      00  
00479C66   .  00            DB      00  
00479C67   .  00            DB      00  
00479C68   .  00            DB      00  
00479C69   .  00            DB      00  
00479C6A   .  00            DB      00  
00479C6B   .  00            DB      00  
00479C6C   .  00            DB      00  
00479C6D   .  00            DB      00  
00479C6E   .  00            DB      00  
00479C6F   .  00            DB      00  
00479C70      789C4700      DD      DVDFabGo.00479C78  
00479C74      00            DB      00  
00479C75      90            NOP  
00479C76      90            NOP  
00479C77      90            NOP  
00479C78   .  60            PUSHAD  
00479C79   .  8BC2          MOV     EAX, EDX  
00479C7B   .  B9 C80B0000   MOV     ECX, 0BC8                        ;  iat 大小  
00479C80   .  BF 00A04700   MOV     EDI, 0047A000                    ;  iat 起始地址  
00479C85   .  F2:AF         REPNE   SCAS DWORD PTR ES:\[EDI\]  
00479C87   .  83EF 04       SUB     EDI, 4  
00479C8A   .  893D 08415100 MOV     DWORD PTR DS:\[514108\], EDI       ;  保存地址  
00479C90   .  61            POPAD  
00479C91   .  FF15 18A0AE00 CALL    DWORD PTR DS:\[AEA018\]            ;  执行程序的原代码  
00479C97   .  C3            RETN  
00479C98      90            NOP  
  
再把这里的代码改一下：  
00AC2548    FF15 709C4700   CALL    DWORD PTR DS:\[479C70\]            ; DVDFabGo.00479C78  
  
写到这里，完了吗?当然没有了，认真看就会发前的的 jmp ebx 那里是个 call，直接这样操作就回不来了,这也就是为什么前面还用一个 patch2,  
直接到 00D500B7 看看去，看看现在在哪里了。  
00D500E1    9D              POPFD  
00D500E2    5C              POP     ESP                              ; 00A80000  
00D500E3  - FF6424 FC       JMP     DWORD PTR SS:\[ESP\-4\]             ; 这次在这里了  
00D500E7    CC              INT3  
  
直接在 D500E3 处下硬件断点，没错了，我想在这里的时候把返回地址改成跳去 00479BCF，直接修改代码?当然不行了，壳会检测的，找到地方去除检测?我偷懒处理下，用脚本就很简单的搞定了：  
//fixed aspr 2.x  
var addr  
  
  
start:  
  run  
  
l1:  
  cmp eip,00D500E3  
  jne l2  
  mov addr,esp  
  sub addr,4  
  mov \[addr\],479bcf  
  jmp start  
  
l2:  
  ret  
  
写好后，记得在 479c33 处下个 f2 断，再运行，否则运行脚本后会"飞"的。  
  
脚本运行完毕，赶快 dump 完，修复收工吧(做完后要记得把修复代码和保存的数据给清除掉哦，做事要有头有尾才行的).嗯，脱壳完毕,当然这里实际上还是有点小问题的，问题解决方法请参考我的第二篇 ASPR 文章  
  
Greetz:  
 Fly.Jingulong,yock,tDasm.David.hexer,hmimys,ahao.UFO(brother).alan(sister).all of my   
  
friends and you!  
  
          By loveboom\[DFCG\]\[FCG\]\[US\]  
                      http://blog.csdn.net/bmd2chen  
                  
  
                                        Email:loveboom#163.com 

> 引用:
> 
> * * *
> 
> 00AE6265    8B45 F4         MOV     EAX, DWORD PTR SS:\[EBP-C\]  
> 00AE6268    8B40 3C         MOV     EAX, DWORD PTR DS:\[EAX+3C\]  
> 00AE626B    8B55 FC         MOV     EDX, DWORD PTR SS:\[EBP-4\]        ; 传递正确的 API 到 EDX 中
> 
> * * *

  
我用 Aspr2.11 加壳的记事本  
第一个 Call api 应该是调用 GetCommandLineA  
被改为 Call 00D60000  
跟进来到你说的传递正确 API 到 EDX 的地方时，竟然变成了 GetSartupInfo  
怎么也想不通会这样，正在郁闷中啊！  
(已解决)  
原来修改代码前传递正确的 API，随后会传递下一个 Api 准备调用，下一个 Api 就是 GetSartupInfo，我跟到那里去了。 

 4dd466d0e1e0cc123afbe504da2462ca