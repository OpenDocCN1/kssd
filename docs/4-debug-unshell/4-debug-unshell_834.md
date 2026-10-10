

# Hying's PeLock v0.7x 外壳分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-664.htm`](https://www.pediy.com/kssd/pediy07/pediy7-664.htm)

前言：以前版分析的一个版本，通过对旧版的了解可以很快的掌握新版的变化的，新版刚看完，谢谢 hnhuqiong 的帮助，脚本太多了，我自己看不下去:-(,  
我还是选择了自己的方法，走了一程。走了后感觉新版变化也不是很大，虽然多  
乱序 IAT 和特殊代码加密转换了一种比较方式，不过去乱序 iat 感觉就像鸡肋，  
特殊代码处理虽然有一定难度，但很可惜，花指令太死了，如果有足够的时间的话，可以完全不用去掉花指令跟踪到特殊代码，也许是 Magic esp 的烂用吧:-),  
没有任何的杀伤力可言。  
当然这个壳的精彩代码是非常不错的，也因此非常感谢 hexer。这段代码我现在还专门记录下来了:-).这篇文章把 Simxx 和 hexer 部分没有分析清楚的，自己补了一下。新版的总体流程还是差不多的。  
  
                    Hying's PeLock v0.7x 外壳分析  
  
【目    标】:Delphi  
【工    具】:Olydbg1.1(diy 版)  
【任    务】:分析外壳 loader  
【操作平台】:Windows XP sp2  
【作    者】:LOVEBOOM\[DFCG\]\[FCG\]\[US\]  
【相关链接】:NONE  
【简要说明】:Hying 的外壳上次没有完全分析完下篇，这回来个全面的，外壳有几个地方很有"意思",新版本的花指令也比较有"意思"的.也随便把 Hexer 和 simonzh2000  
文章中没有分析明白的，自己分析了下．文章我大部分地方用了标签来注明，也因此很少额外写注释的.应该直接看下就知道什么意思了，看完后，事理一下可以做出  
一个和原版一样的 loader 来的:-)．  
  
【详细过程】:  
  因为全面分析的文章，所以我也懒得多打几个字．边分析边写吧:-).  
用 OD 载入，下断 Bp VirtualAlloc 然后执行到返回，之所以这样是因为前面的解压代码很是无聊，也没有什么价值，直接入主题更好:-).  
来到这里:  
  
0045A11A    6A 04           PUSH    4  
0045A11C    68 00100000     PUSH    1000  
0045A121    FF75 10         PUSH    DWORD PTR SS:\[EBP+10\]  
0045A124    6A 00           PUSH    0  
0045A126    FF55 2C         CALL    DWORD PTR SS:\[EBP+2C\]                                       ; VirtualAlloc  
0045A129    50              PUSH    EAX                                                         ; 返回这里  
0045A12A    8945 0C         MOV     DWORD PTR SS:\[EBP+C\], EAX  
0045A12D    8B5D 08         MOV     EBX, DWORD PTR SS:\[EBP+8\]  
0045A130    03DD            ADD     EBX, EBP  
0045A132    50              PUSH    EAX  
0045A133    53              PUSH    EBX  
0045A134    E8 18000000     CALL    <aplib\_Unpack>                                              ; 解压代码  
0045A139    5A              POP     EDX                                                         ; 0012FFE0  
0045A13A    52              PUSH    EDX  
0045A13B    55              PUSH    EBP  
0045A13C    8D85 DE000000   LEA     EAX, DWORD PTR SS:\[EBP+DE\]  
0045A142    C600 EB         MOV     BYTE PTR DS:\[EAX\], 0EB                                      ; 对 dll 的再次重入进行处理  
0045A145    C640 01 10      MOV     BYTE PTR DS:\[EAX+1\], 10  
0045A149    8B45 30         MOV     EAX, DWORD PTR SS:\[EBP+30\]  
0045A14C    8945 74         MOV     DWORD PTR SS:\[EBP+74\], EAX  
0045A14F  - FFE2            JMP     EDX                                                         ; 这里跳去壳的部分  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
aplib\_Unpack:  
  
0045A151 >  60              PUSHAD                                                              ; aplib\_Unpack  
0045A152    8B7424 24       MOV     ESI, DWORD PTR SS:\[ESP+24\]                                  ; aplib 解压代码，后面壳里还会用到一次  
0045A156    8B7C24 28       MOV     EDI, DWORD PTR SS:\[ESP+28\]                                  ; 因此直接抓下来  
0045A15A    FC              CLD  
0045A15B    B2 80           MOV     DL, 80  
0045A15D    33DB            XOR     EBX, EBX  
0045A15F    A4              MOVS    BYTE PTR ES:\[EDI\], BYTE PTR DS:\[ESI\]  
0045A160    B3 02           MOV     BL, 2  
0045A162    E8 6D000000     CALL    0045A1D4  
0045A167  ^ 73 F6           JNB     SHORT 0045A15F  
0045A169    33C9            XOR     ECX, ECX  
0045A16B    E8 64000000     CALL    0045A1D4  
0045A170    73 1C           JNB     SHORT 0045A18E  
0045A172    33C0            XOR     EAX, EAX  
0045A174    E8 5B000000     CALL    0045A1D4  
0045A179    73 23           JNB     SHORT 0045A19E  
0045A17B    B3 02           MOV     BL, 2  
0045A17D    41              INC     ECX  
0045A17E    B0 10           MOV     AL, 10  
0045A180    E8 4F000000     CALL    0045A1D4  
0045A185    12C0            ADC     AL, AL  
0045A187  ^ 73 F7           JNB     SHORT 0045A180  
0045A189    75 3F           JNZ     SHORT 0045A1CA  
0045A18B    AA              STOS    BYTE PTR ES:\[EDI\]  
0045A18C  ^ EB D4           JMP     SHORT 0045A162  
0045A18E    E8 4D000000     CALL    0045A1E0  
0045A193    2BCB            SUB     ECX, EBX  
0045A195    75 10           JNZ     SHORT 0045A1A7  
0045A197    E8 42000000     CALL    0045A1DE  
0045A19C    EB 28           JMP     SHORT 0045A1C6  
0045A19E    AC              LODS    BYTE PTR DS:\[ESI\]  
0045A19F    D1E8            SHR     EAX, 1  
0045A1A1    74 4D           JE      SHORT 0045A1F0  
0045A1A3    13C9            ADC     ECX, ECX  
0045A1A5    EB 1C           JMP     SHORT 0045A1C3  
0045A1A7    91              XCHG    EAX, ECX  
0045A1A8    48              DEC     EAX  
0045A1A9    C1E0 08         SHL     EAX, 8  
0045A1AC    AC              LODS    BYTE PTR DS:\[ESI\]  
0045A1AD    E8 2C000000     CALL    0045A1DE  
0045A1B2    3D 007D0000     CMP     EAX, 7D00  
0045A1B7    73 0A           JNB     SHORT 0045A1C3  
0045A1B9    80FC 05         CMP     AH, 5  
0045A1BC    73 06           JNB     SHORT 0045A1C4  
0045A1BE    83F8 7F         CMP     EAX, 7F  
0045A1C1    77 02           JA      SHORT 0045A1C5  
0045A1C3    41              INC     ECX  
0045A1C4    41              INC     ECX  
0045A1C5    95              XCHG    EAX, EBP  
0045A1C6    8BC5            MOV     EAX, EBP  
0045A1C8    B3 01           MOV     BL, 1  
0045A1CA    56              PUSH    ESI  
0045A1CB    8BF7            MOV     ESI, EDI  
0045A1CD    2BF0            SUB     ESI, EAX  
0045A1CF    F3:A4           REP     MOVS BYTE PTR ES:\[EDI\], BYTE PTR DS:\[ESI\]  
0045A1D1    5E              POP     ESI                                                         ; 0012FFE0  
0045A1D2  ^ EB 8E           JMP     SHORT 0045A162  
0045A1D4    02D2            ADD     DL, DL  
0045A1D6    75 05           JNZ     SHORT 0045A1DD  
0045A1D8    8A16            MOV     DL, BYTE PTR DS:\[ESI\]  
0045A1DA    46              INC     ESI  
0045A1DB    12D2            ADC     DL, DL  
0045A1DD    C3              RETN  
0045A1DE    33C9            XOR     ECX, ECX  
0045A1E0    41              INC     ECX  
0045A1E1    E8 EEFFFFFF     CALL    0045A1D4  
0045A1E6    13C9            ADC     ECX, ECX  
0045A1E8    E8 E7FFFFFF     CALL    0045A1D4  
0045A1ED  ^ 72 F2           JB      SHORT 0045A1E1  
0045A1EF    C3              RETN  
0045A1F0    2B7C24 28       SUB     EDI, DWORD PTR SS:\[ESP+28\]                                  ; kernel32.7C816D4F  
0045A1F4    897C24 1C       MOV     DWORD PTR SS:\[ESP+1C\], EDI  
0045A1F8    61              POPAD  
0045A1F9    C2 0800         RETN    8  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
jmp edx,来到这里:  
00370000    E8 24000000     CALL    00370029                                 ; 这里有很多无意义的异常,跳过这一段没用的垃圾代码的说明  
00370005    8B4424 04       MOV     EAX, DWORD PTR SS:\[ESP+4\]  
00370009    8B00            MOV     EAX, DWORD PTR DS:\[EAX\]  
0037000B    3D 04000080     CMP     EAX, 80000004  
00370010    75 08           JNZ     SHORT 0037001A  
00370012    8B6424 08       MOV     ESP, DWORD PTR SS:\[ESP+8\]          
.......  
经过 N 个漫长的 SEH，飞到下面来:  
003731DE    8B06            MOV     EAX, DWORD PTR DS:\[ESI\]                                     ; 循环把 API 搬到壳中,取两次  
003731E0    90              NOP  
003731E1    90              NOP  
003731E2    90              NOP  
003731E3    90              NOP  
003731E4    90              NOP  
003731E5    90              NOP  
003731E6    90              NOP  
003731E7    8907            MOV     DWORD PTR DS:\[EDI\], EAX                                     ; 从 00375A8A 开始  
003731E9    90              NOP  
003731EA    90              NOP  
003731EB    90              NOP  
003731EC    90              NOP  
003731ED    90              NOP  
003731EE    90              NOP  
003731EF    90              NOP  
003731F0    90              NOP  
003731F1    90              NOP  
003731F2    90              NOP  
003731F3    90              NOP  
003731F4    83C6 04         ADD     ESI, 4  
003731F7    90              NOP  
003731F8    90              NOP  
003731F9    90              NOP  
003731FA    90              NOP  
003731FB    90              NOP  
003731FC    83C7 04         ADD     EDI, 4  
003731FF    90              NOP  
00373200    90              NOP  
00373201    90              NOP  
00373202    90              NOP  
00373203  ^ E2 D9           LOOPD   SHORT 003731DE  
00373205    90              NOP  
00373206    90              NOP  
00373207    90              NOP  
00373208    90              NOP  
00373209    90              NOP  
0037320A    8B45 04         MOV     EAX, DWORD PTR SS:\[EBP+4\]  
0037320D    90              NOP  
0037320E    90              NOP  
0037320F    90              NOP  
00373210    90              NOP  
00373211    90              NOP  
00373212    90              NOP  
00373213    90              NOP  
00373214    90              NOP  
00373215    90              NOP  
00373216    90              NOP  
00373217    90              NOP  
00373218    8982 B2434000   MOV     DWORD PTR DS:\[EDX+<IMGBASE>\], EAX  
0037321E    90              NOP  
0037321F    90              NOP  
00373220    90              NOP  
00373221    90              NOP  
00373222    90              NOP  
00373223    90              NOP  
00373224    90              NOP  
00373225    8D85 51010000   LEA     EAX, DWORD PTR SS:\[EBP+151\]  
0037322B    90              NOP  
0037322C    90              NOP  
0037322D    90              NOP  
0037322E    90              NOP  
0037322F    90              NOP  
00373230    8982 12444000   MOV     DWORD PTR DS:\[EDX+404412\], EAX  
00373236    90              NOP  
00373237    90              NOP  
00373238    90              NOP  
00373239    90              NOP  
0037323A    90              NOP  
0037323B    8B45 2C         MOV     EAX, DWORD PTR SS:\[EBP+2C\]  
0037323E    90              NOP  
0037323F    90              NOP  
00373240    90              NOP  
00373241    90              NOP  
00373242    90              NOP  
00373243    90              NOP  
00373244    90              NOP  
00373245    90              NOP  
00373246    90              NOP  
00373247    90              NOP  
00373248    90              NOP  
00373249    90              NOP  
0037324A    90              NOP  
0037324B    90              NOP  
0037324C    90              NOP  
0037324D    90              NOP  
0037324E    90              NOP  
0037324F    90              NOP  
00373250    90              NOP  
00373251    90              NOP  
00373252    8982 AE434000   MOV     DWORD PTR DS:\[EDX+<APIVirtualAlloc>\], EAX                 ; VirtualAlloc  
00373258    90              NOP  
00373259    90              NOP  
......  
003732F9    6A 00           PUSH    0                                                           ; 获取 ImageBase  
003732FB    FF95 A6434000   CALL    DWORD PTR SS:\[EBP+<GetModhandle>\]                         ; kernel32.GetModuleHandleA  
00373301    90              NOP  
00373302    90              NOP  
00373303    90              NOP  
00373304    68 C2100000     PUSH    10C2  
00373309    E8 01000000     CALL    0037330F  
0037330E    90              NOP  
0037330F    68 24080E68     PUSH    680E0824  
00373314    68 90908344     PUSH    44839090  
00373319  - FFE4            JMP     ESP                                                         ; 呵呵，有个性的花指令这里指令没有起什么作用只是跳去下一步而已  
0037331B    90              NOP  
0037331C    8985 B2434000   MOV     DWORD PTR SS:\[EBP+<IMGBASE>\], EAX  
......  
00373339    6A 04           PUSH    4  
0037333B    68 00100000     PUSH    1000  
00373340    68 00100000     PUSH    1000  
00373345    6A 00           PUSH    0  
00373347    FF95 AE434000   CALL    DWORD PTR SS:\[EBP+<APIVirtualAlloc>\]                      ; kernel32.VirtualAlloc  
0037334D    8985 E5494000   MOV     DWORD PTR SS:\[EBP+<hMEM334d>\], EAX                        ; 申请空间  
00373353    5E              POP     ESI                                                         ; 下面就准备开始壳中最为精彩的代码了  
00373354    E8 28010000     CALL    <Important\_SEH>                                             ; 这里开始精彩代码  
下面开始引用 HEXER 那篇文章里讲的，这个目标也是差不多，只是代码不是一开始就解压出来了，每层都是动态解压出来的，我是开两个来记录的．  
嗯看精彩代码：  
；\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
；=====注意看精彩片段开始了==================================================================================   
TELOCK 中有一段曾经被称为非常精彩的代码，下面这一段是青处于蓝而胜于蓝，精彩倍增：   
00373359    E8 04000000     CALL    <SEH\_Disposal>  
0037335E    90              NOP  
0037335F    90              NOP  
00373360    90              NOP  
00373361    90              NOP  
00373362 >  5A              POP     EDX                                                         ; SEH\_Disposal  
00373363    8B4424 04       MOV     EAX, DWORD PTR SS:\[ESP+4\]  
00373367    8B00            MOV     EAX, DWORD PTR DS:\[EAX\]                                     ; EXCEPTION REASON  
00373369    8B4C24 0C       MOV     ECX, DWORD PTR SS:\[ESP+C\]                                   ; pContext  
0037336D    C701 17000100   MOV     DWORD PTR DS:\[ECX\], 10017  
00373373    FF81 B8000000   INC     DWORD PTR DS:\[ECX+B8\]                                       ; regEIP+1  
00373379    3D 03000080     CMP     EAX, 80000003                                               ; 判断是否为断点异常 int 3  
0037337E    75 51           JNZ     SHORT <SINGLE STEP BREAKPIONT >  
00373380    8B81 B4000000   MOV     EAX, DWORD PTR DS:\[ECX+B4\]                                  ; 第一次断点异常后，壳开始设置硬件断点实现 DEBUG 他自己  
00373386    8D80 F41D4000   LEA     EAX, DWORD PTR DS:\[EAX+<Dr0:003734D8>\]  
0037338C    8941 04         MOV     DWORD PTR DS:\[ECX+4\], EAX                                   ; 设置 Dr0  
0037338F    8B81 B4000000   MOV     EAX, DWORD PTR DS:\[ECX+B4\]  
00373395    8D80 221E4000   LEA     EAX, DWORD PTR DS:\[EAX+<Dr1:00373506>\]  
0037339B    8941 08         MOV     DWORD PTR DS:\[ECX+8\], EAX                                   ; 设置 Dr1  
0037339E    8B81 B4000000   MOV     EAX, DWORD PTR DS:\[ECX+B4\]  
003733A4    8D80 4F1E4000   LEA     EAX, DWORD PTR DS:\[EAX+<Dr2:00373533>\]  
003733AA    8941 0C         MOV     DWORD PTR DS:\[ECX+C\], EAX                                   ; 设置 Dr2  
003733AD    8B81 B4000000   MOV     EAX, DWORD PTR DS:\[ECX+B4\]  
003733B3    8D80 821E4000   LEA     EAX, DWORD PTR DS:\[EAX+<Dr3:00373566>\]  
003733B9    8941 10         MOV     DWORD PTR DS:\[ECX+10\], EAX                                  ; 设置 Dr3  
003733BC    33C0            XOR     EAX, EAX  
003733BE    8161 14 F00FFFF>AND     DWORD PTR DS:\[ECX+14\], FFFF0FF0                             ; 设置 Dr6,dr6 所有是 1 的位都是保留为 1 的，初始化调试状态寄存器  
003733C5    C741 18 5501000>MOV     DWORD PTR DS:\[ECX+18\], 155                                  ; 设置 DR7，设置调试控制寄存器，允许 4 个当前任务的执行断点  
003733CC    E9 AF000000     JMP     <Return\_System>  
003733D1 >  3D 04000080     CMP     EAX, 80000004                                               ; SINGLE STEP BREAKPIONT  
003733D6    75 64           JNZ     SHORT <DIVIDE BY ZERO >                                     ; 这里是解密整个壳的关键  
003733D8    FF02            INC     DWORD PTR DS:\[EDX\]                                          ; 因为前面设置了四个硬件断点，因此，正常运行的话，肯定会要经过这里的  
003733DA    8B02            MOV     EAX, DWORD PTR DS:\[EDX\]  
003733DC    83F8 01         CMP     EAX, 1                                                      ; Dr0:003734D8 异常  
003733DF    75 08           JNZ     SHORT 003733E9  
003733E1    F791 B0000000   NOT     DWORD PTR DS:\[ECX+B0\]                                       ; 第一次异常发生在 3734d8 处时,执行操作 not regEAX  
003733E7    EB 4F           JMP     SHORT 00373438  
003733E9    83F8 02         CMP     EAX, 2                                                      ; Dr1:00373506 异常  
003733EC    75 11           JNZ     SHORT 003733FF  
003733EE    8B81 B0000000   MOV     EAX, DWORD PTR DS:\[ECX+B0\]                                  ; 第二次异常发生在 00373506 处时,ROL regEAX,13h  
003733F4    C1C0 13         ROL     EAX, 13  
003733F7    8981 B0000000   MOV     DWORD PTR DS:\[ECX+B0\], EAX                                  ; 写回 regEAX  
003733FD    EB 39           JMP     SHORT 00373438  
003733FF    83F8 03         CMP     EAX, 3                                                      ; Dr2:00373533 异常  
00373402    75 2B           JNZ     SHORT <Exception Dr3\>  
00373404    53              PUSH    EBX  
00373405    8181 B0000000 2>ADD     DWORD PTR DS:\[ECX+B0\], 4B23526                              ; 第三次单步异常发生在 373533 处时,执行 add regEAX,4b23526  
0037340F    8B81 B0000000   MOV     EAX, DWORD PTR DS:\[ECX+B0\]                                  ; regEAX  
00373415    8B99 A4000000   MOV     EBX, DWORD PTR DS:\[ECX+A4\]                                  ; regEBX  
0037341B    66:93           XCHG    AX, BX                                                      ; 相加后 ax 和 bx 交换再加上 bx  
0037341D    66:03C3         ADD     AX, BX  
00373420    8981 B0000000   MOV     DWORD PTR DS:\[ECX+B0\], EAX                                  ; 写回 eax  
00373426    8999 A4000000   MOV     DWORD PTR DS:\[ECX+A4\], EBX                                  ; 写回 regEBX  
0037342C    5B              POP     EBX                                                         ; 0012FFE0  
0037342D    EB 09           JMP     SHORT 00373438  
0037342F >  8B81 A0000000   MOV     EAX, DWORD PTR DS:\[ECX+A0\]  
00373435    8030 55         XOR     BYTE PTR DS:\[EAX\], 55                                       ; Dr3:00373566 异常  
00373438    33C0            XOR     EAX, EAX                                                    ; 第四次异常发生时,xor \[regESI\],55h  
0037343A    EB 44           JMP     SHORT <Return\_System>  
0037343C >  3D 940000C0     CMP     EAX, C0000094                                               ; DIVIDE BY ZERO  
00373441    75 3A           JNZ     SHORT 0037347D  
00373443    C702 00000000   MOV     DWORD PTR DS:\[EDX\], 0  
00373449    FF81 B8000000   INC     DWORD PTR DS:\[ECX+B8\]                                       ; inc regEIP  
0037344F    33C0            XOR     EAX, EAX                                                    ; 最后一次除 0 异常后，写入值到 Drx 中, 后面要参与计算关键值的  
00373451    C741 04 2301FF0>MOV     DWORD PTR DS:\[ECX+4\], 0FFF0123                              ; 设置 Dr0  
00373458    C741 08 6745FF0>MOV     DWORD PTR DS:\[ECX+8\], 0FFF4567                              ; 设置 Dr1  
0037345F    C741 0C AB89FF0>MOV     DWORD PTR DS:\[ECX+C\], 0FFF89AB                              ; 设置 Dr2  
00373466    C741 10 EFCDFF0>MOV     DWORD PTR DS:\[ECX+10\], 0FFFCDEF                             ; 设置 Dr3  
0037346D    8161 14 F00FFFF>AND     DWORD PTR DS:\[ECX+14\], FFFF0FF0                             ; 设置 Dr6  
00373474    C741 18 5501000>MOV     DWORD PTR DS:\[ECX+18\], 155                                  ; 设置 Dr7  
0037347B    EB 03           JMP     SHORT <Return\_System>  
0037347D    33C0            XOR     EAX, EAX  
0037347F    40              INC     EAX  
00373480 >  C3              RETN                                                                ; Return\_System  
00373481 >  56              PUSH    ESI                                                         ; Important\_SEH  
00373482    8DB5 951C4000   LEA     ESI, DWORD PTR SS:\[EBP+401C95\]                              ; 进入 int3 异常之前解压出处理 int3 的代码  
00373488    8BFE            MOV     EDI, ESI  
0037348A    B9 55000000     MOV     ECX, 55  
0037348F    AC              LODS    BYTE PTR DS:\[ESI\]  
00373490    32C1            XOR     AL, CL  
00373492    C0C0 04         ROL     AL, 4  
00373495    AA              STOS    BYTE PTR ES:\[EDI\]  
00373496  ^ E2 F7           LOOPD   SHORT 0037348F  
00373498    5E              POP     ESI                                                         ; 一层一层的解开，真是"浪费"时间:-)  
00373499    33C0            XOR     EAX, EAX  
0037349B    64:FF35 0000000>PUSH    DWORD PTR FS:\[0\]  
003734A2    64:8925 0000000>MOV     DWORD PTR FS:\[0\], ESP  
003734A9    CC              INT3  
003734AA    90              NOP  
003734AB    56              PUSH    ESI  
003734AC    8DB5 ED1C4000   LEA     ESI, DWORD PTR SS:\[EBP+401CED\]                              ; 循环解压出处理四个单步异常的代码  
003734B2    8BFE            MOV     EDI, ESI  
003734B4    B9 6B000000     MOV     ECX, 6B  
003734B9    AC              LODS    BYTE PTR DS:\[ESI\]  
003734BA    32C1            XOR     AL, CL  
003734BC    04 4D           ADD     AL, 4D  
003734BE    C0C0 03         ROL     AL, 3  
003734C1    AA              STOS    BYTE PTR ES:\[EDI\]  
003734C2  ^ E2 F5           LOOPD   SHORT 003734B9  
003734C4    5E              POP     ESI                                                         ; 0012FFE0  
003734C5    8D8D A01A4000   LEA     ECX, DWORD PTR SS:\[EBP+<Crc\_Start\_addr>\]                  ; 解压出处理四个单步异常的代码后，开始检测是代码是否被修改过  
003734CB    2BCE            SUB     ECX, ESI                                                    ; 这一招有够利害，直接下 CC 断点，crc 检测通不过  
003734CD    33DB            XOR     EBX, EBX                                                    ; 下硬件断点的话，原来的硬件断点就破坏了．  
003734CF    33C0            XOR     EAX, EAX  
003734D1    AC              LODS    BYTE PTR DS:\[ESI\]  
003734D2    03D8            ADD     EBX, EAX                                                    ; 计算校验值  
003734D4  ^ E2 FB           LOOPD   SHORT 003734D1  
003734D6    8BC3            MOV     EAX, EBX  
003734D8    F8              CLC                                                                 ; 第一次异常处,not eax  
003734D9    90              NOP  
003734DA    8DB5 0D1E4000   LEA     ESI, DWORD PTR SS:\[EBP+<De1>\]                          ; 异常后开始解压出下面将要执行的代码  
003734E0    B9 91250000     MOV     ECX, 2591  
003734E5    F7E1            MUL     ECX  
003734E7    D3C8            ROR     EAX, CL  
003734E9    3006            XOR     BYTE PTR DS:\[ESI\], AL  
003734EB    46              INC     ESI  
003734EC    40              INC     EAX  
003734ED    D40A            AAM  
003734EF  ^ E2 F4           LOOPD   SHORT 003734E5  
003734F1    B9 D5010000     MOV     ECX, 1D5                                                    ; De1  
003734F6    8DB5 A01A4000   LEA     ESI, DWORD PTR SS:\[EBP+<Crc\_Start\_addr>\]                  ; 刚解压出来代码再次检测内存代码，把计算出的值作为解密 KEY  
003734FC    33C0            XOR     EAX, EAX  
003734FE    3206            XOR     AL, BYTE PTR DS:\[ESI\]  
00373500    C1C8 08         ROR     EAX, 8  
00373503    46              INC     ESI  
00373504  ^ E2 F8           LOOPD   SHORT 003734FE  
00373506    FC              CLD                                                                 ; 第二次异常处  
00373507    90              NOP  
00373508    B9 64250000     MOV     ECX, 2564                                                   ; rol eax,13  
0037350D    8DB5 3A1E4000   LEA     ESI, DWORD PTR SS:\[EBP+401E3A\]  
00373513    8D4481 43       LEA     EAX, DWORD PTR DS:\[ECX+EAX\*4+43\]  
00373517    3006            XOR     BYTE PTR DS:\[ESI\], AL  
00373519    D40A            AAM  
0037351B    46              INC     ESI  
0037351C  ^ E2 F5           LOOPD   SHORT 00373513                                              ; 再次把通过异常中断处理来计算 eax 的值  
0037351E    B9 B2000000     MOV     ECX, 0B2                                                    ; 通过 eax 的值来解密出 37351e 处的代码  
00373523    C1E9 02         SHR     ECX, 2  
00373526    8DB5 9D1D4000   LEA     ESI, DWORD PTR SS:\[EBP+401D9D\]                              ; 这里从开始解密处的代码处开始计算检测值  
0037352C    33DB            XOR     EBX, EBX  
0037352E    AD              LODS    DWORD PTR DS:\[ESI\]  
0037352F    33D8            XOR     EBX, EAX  
00373531  ^ E2 FB           LOOPD   SHORT 0037352E                                              ; 这样在这段范围内的代码又不可以被修改又不可以被下断点  
00373533    F9              STC                                                                 ; 第三次异常处  
00373534    90              NOP                                                                 ; 执行 add regEAX,4b23526  
00373535    B9 2A250000     MOV     ECX, 252A                                                   ; xchg ax,bx  
0037353A    C1E9 02         SHR     ECX, 2                                                      ; add ax,bx  
0037353D    8DB5 741E4000   LEA     ESI, DWORD PTR SS:\[EBP+401E74\]  
00373543    33D2            XOR     EDX, EDX  
00373545    F7E3            MUL     EBX  
00373547    81C2 2635B204   ADD     EDX, 4B23526  
0037354D    3116            XOR     DWORD PTR DS:\[ESI\], EDX  
0037354F    8BC3            MOV     EAX, EBX  
00373551    8BDA            MOV     EBX, EDX  
00373553    83C6 04         ADD     ESI, 4  
00373556  ^ E2 EB           LOOPD   SHORT 00373543                                              ; 通过 SEH 来计算出 EAX 和 ebx 的值,用于解出 00373558 处的代码  
00373558    8DB5 8E1E4000   LEA     ESI, DWORD PTR SS:\[EBP+401E8E\]  
0037355E    B9 C0060000     MOV     ECX, 6C0  
00373563    C1E9 02         SHR     ECX, 2  
00373566    90              NOP                                                                 ; 这里第四次异常，其实也就是把 Dr3 夹在里面  
00373567    90              NOP                                                                 ; 执行 xor \[esi\],55  
00373568    802E 13         SUB     BYTE PTR DS:\[ESI\], 13  
0037356B    F616            NOT     BYTE PTR DS:\[ESI\]  
0037356D    83C6 04         ADD     ESI, 4  
00373570  ^ E2 F4           LOOPD   SHORT 00373566                                              ; 通过 SEH 处理来换算出 373570 处的代码  
00373572    8DB5 581D4000   LEA     ESI, DWORD PTR SS:\[EBP+401D58\]                              ; 到了这里后开始解压出处理除０异常的代码  
00373578    8BFE            MOV     EDI, ESI  
0037357A    B9 41000000     MOV     ECX, 41  
0037357F    AC              LODS    BYTE PTR DS:\[ESI\]  
00373580    32C1            XOR     AL, CL  
00373582    04 63           ADD     AL, 63  
00373584    AA              STOS    BYTE PTR ES:\[EDI\]  
00373585  ^ E2 F8           LOOPD   SHORT 0037357F  
00373587    B8 00010000     MOV     EAX, 100  
0037358C    33D2            XOR     EDX, EDX  
0037358E    33DB            XOR     EBX, EBX  
00373590    F7F3            DIV     EBX                                                         ; DIVIDE BY ZERO，最后一次 SEH，恢复一些原始状态  
00373592    90              NOP  
00373593    64:8F05 0000000>POP     DWORD PTR FS:\[0\]                                            ; 恢复 SEH  
0037359A    58              POP     EAX                                                         ; 0012FFE0  
  
；=====此阶段精彩片段结束===================================================================================   
；\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
  
嗯这里挺过来了就好办，上面这段代码设计的非常不错，操作不当很容易出错的，当然动下脑还是很容易跳过这段代码，我的做法是写个脚本，脚本大概的做法就是把壳原有的  
异常都让我的脚本来操作，脚本先把 seh 时那几个 Drx 保存起来，然后脚本通知 OD 去解开代码就成了:-).  
继续向下分析:  
  
0037359B    8BFC            MOV     EDI, ESP                                                    ; 用 esp 来转换解压，有个性的:-)  
0037359D    8DA5 9D434000   LEA     ESP, DWORD PTR SS:\[EBP+40439D\]                              ; 从 375a81 处开始倒着向上解压  
003735A3    B9 B3240000     MOV     ECX, 24B3  
003735A8    8B85 16444000   MOV     EAX, DWORD PTR SS:\[EBP+404416\]  
003735AE    BB BDD89800     MOV     EBX, 98D8BD  
003735B3    BE D5260000     MOV     ESI, 26D5  
003735B8 >  33D2            XOR     EDX, EDX                                                    ; Loop\_Decode\_3735cf  
003735BA    F7E6            MUL     ESI                                                         ; 循环解出 3735cf 处的代码  
003735BC    05 78563412     ADD     EAX, 12345678  
003735C1    83D2 00         ADC     EDX, 0  
003735C4    F7F3            DIV     EBX  
003735C6    58              POP     EAX                                                         ; 0012FFE0  
003735C7    32C2            XOR     AL, DL  
003735C9    50              PUSH    EAX  
003735CA    4C              DEC     ESP  
003735CB    8BC2            MOV     EAX, EDX  
003735CD  ^ E2 E9           LOOPD   SHORT <Loop\_Decode\_3735cf>  
上面精彩的过了，这回来了个变本加厉的，下面还有一段关键代码，不过感觉也太浪费时间做这些工作吧:-),解密代码:  
  
003735CF    8BE7            MOV     ESP, EDI  
003735D1    E8 51000000     CALL    <Fuck\_Int3\_3627>                                            ; 这里还有好玩的东西  
003735D6    8B4C24 0C       MOV     ECX, DWORD PTR SS:\[ESP+C\]  
003735DA    8B81 B0000000   MOV     EAX, DWORD PTR DS:\[ECX+B0\]                                  ; regEAX  
003735E0    8B51 04         MOV     EDX, DWORD PTR DS:\[ECX+4\]  
003735E3    F6D0            NOT     AL  
003735E5    32C2            XOR     AL, DL  
003735E7    66:25 FF00      AND     AX, 0FF  
003735EB    66:03D0         ADD     DX, AX  
003735EE    66:C1CA 03      ROR     DX, 3  
003735F2    66:8951 04      MOV     WORD PTR DS:\[ECX+4\], DX                                     ; 运算后改变 Dr0 的值  
003735F6    66:3151 08      XOR     WORD PTR DS:\[ECX+8\], DX  
003735FA    66:8B51 08      MOV     DX, WORD PTR DS:\[ECX+8\]                                     ; 再把值和 Dr1 进行运算  
003735FE    66:C1CA 02      ROR     DX, 2  
00373602    66:0151 0C      ADD     WORD PTR DS:\[ECX+C\], DX                                     ; 和 Dr2 进行运算  
00373606    66:8B51 0C      MOV     DX, WORD PTR DS:\[ECX+C\]  
0037360A    66:F7D2         NOT     DX  
0037360D    66:2B51 10      SUB     DX, WORD PTR DS:\[ECX+10\]                                    ; 和 Dr3 进行运算  
00373611    66:D1CA         ROR     DX, 1  
00373614    66:3151 04      XOR     WORD PTR DS:\[ECX+4\], DX                                     ; 运算后的值再保存回 Dr0  
00373618    8981 B0000000   MOV     DWORD PTR DS:\[ECX+B0\], EAX                                  ; 写回 regEAX  
0037361E    FF81 B8000000   INC     DWORD PTR DS:\[ECX+B8\]                                       ; inc regEIP  
00373624    33C0            XOR     EAX, EAX  
00373626    C3              RETN  
00373627 >  64:FF35 0000000>PUSH    DWORD PTR FS:\[0\]                                            ; Fuck\_Int3\_3627  
0037362E    64:8925 0000000>MOV     DWORD PTR FS:\[0\], ESP  
00373635    8DB5 33204000   LEA     ESI, DWORD PTR SS:\[EBP+402033\]                              ; 准备解开 373717 处的代码  
0037363B    B9 C6000000     MOV     ECX, 0C6                                                    ; 解压大小 0c6  
00373640 >  8A0431          MOV     AL, BYTE PTR DS:\[ECX+ESI\]                                   ; loop\_Decode\_373717  
00373643    CC              INT3  
00373644    90              NOP  
00373645    880431          MOV     BYTE PTR DS:\[ECX+ESI\], AL                                   ; 同样是倒序解压  
00373648  ^ E2 F6           LOOPD   SHORT <loop\_Decode\_373717>                                  ; 可以看出 Drx 的值是多么重要了吧:-)  
0037364A    64:8F05 0000000>POP     DWORD PTR FS:\[0\]                                            ; 恢复现场  
00373651    58              POP     EAX                                                         ; 0012FFE0  
00373652    8D05 6B424000   LEA     EAX, DWORD PTR DS:\[40426B\]  
00373658    03C5            ADD     EAX, EBP                                                    ; 计算出 MyGetProcAddress 函数的地址  
0037365A    8985 9E434000   MOV     DWORD PTR SS:\[EBP+<MyGetProcAddress>\], EAX  
  
  
;\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_  
;  
;  下面开始壳获取壳所需的相关 API 函数,先判断 DLL 有没有载入，如果没有则 Load dll,载入后循环的解出 API 的名字，再用 MyGetProcAddress  
;  获取相关 API 的名字，获取到再检测 API 第一个字节有没有下 CC 断点  
  
  
00373660    8DB5 A1444000   LEA     ESI, DWORD PTR SS:\[EBP+<strDllKernel32>\]  
00373666    56              PUSH    ESI  
00373667    8D85 961F4000   LEA     EAX, DWORD PTR SS:\[EBP+401F96\]  
0037366D    50              PUSH    EAX                                                         ; 获取 Kernel32.dll 的句柄  
0037366E    8B85 A6434000   MOV     EAX, DWORD PTR SS:\[EBP+<GetModhandle>\]                    ; kernel32.GetModuleHandleA  
00373674    E9 61130000     JMP     <proc\_Run\_FUN>  
00373679    90              NOP  
0037367A    8BF0            MOV     ESI, EAX  
0037367C    8DBD AF444000   LEA     EDI, DWORD PTR SS:\[EBP+<APIVirtualFree>\]  
00373682    B9 1B000000     MOV     ECX, 1B  
00373687 >  57              PUSH    EDI                                                         ; Fill\_Packer\_API  
00373688    8A07            MOV     AL, BYTE PTR DS:\[EDI\]  
0037368A    EB 05           JMP     SHORT 00373691  
0037368C    F6D0            NOT     AL  
0037368E    AA              STOS    BYTE PTR ES:\[EDI\]                                           ; 解压出 API 名称  
0037368F    8A07            MOV     AL, BYTE PTR DS:\[EDI\]  
00373691    0AC0            OR      AL, AL  
00373693  ^ 75 F7           JNZ     SHORT 0037368C  
00373695    5F              POP     EDI                                                         ; 0012FFE0  
00373696    51              PUSH    ECX  
00373697    57              PUSH    EDI  
00373698    56              PUSH    ESI  
00373699    8D85 C81F4000   LEA     EAX, DWORD PTR SS:\[EBP+401FC8\]  
0037369F    50              PUSH    EAX  
003736A0    8B85 9E434000   MOV     EAX, DWORD PTR SS:\[EBP+<MyGetProcAddress>\]                ; 获取 VirtualFree 的地址  
003736A6    E9 2F130000     JMP     <proc\_Run\_FUN>  
003736AB    90              NOP  
003736AC    E8 8D150000     CALL    <proc\_check\_CC>                                             ; 检测是否下了 CC 断点  
003736B1    50              PUSH    EAX  
003736B2    57              PUSH    EDI  
003736B3    56              PUSH    ESI  
003736B4    8D85 E31F4000   LEA     EAX, DWORD PTR SS:\[EBP+401FE3\]  
003736BA    50              PUSH    EAX  
003736BB    8B85 9E434000   MOV     EAX, DWORD PTR SS:\[EBP+<MyGetProcAddress>\]  
003736C1    E9 14130000     JMP     <proc\_Run\_FUN>  
003736C6    90              NOP  
003736C7    59              POP     ECX                                                         ; 0012FFE0  
003736C8    2BC8            SUB     ECX, EAX  
003736CA    2BC1            SUB     EAX, ECX  
003736CC    0FB64F FF       MOVZX   ECX, BYTE PTR DS:\[EDI\-1\]  
003736D0    8907            MOV     DWORD PTR DS:\[EDI\], EAX  
003736D2    03F9            ADD     EDI, ECX  
003736D4    47              INC     EDI  
003736D5    59              POP     ECX                                                         ; 0012FFE0  
003736D6  ^ E2 AF           LOOPD   SHORT <Fill\_Packer\_API>                                     ; 跳回去循环获取 KERNEL32.DLL 中的相关函数  
003736D8    8DB5 1E444000   LEA     ESI, DWORD PTR SS:\[EBP+<strDllUsr32>\]  
003736DE    56              PUSH    ESI  
003736DF    8D85 0E204000   LEA     EAX, DWORD PTR SS:\[EBP+40200E\]  
003736E5    50              PUSH    EAX  
003736E6    8B85 A6434000   MOV     EAX, DWORD PTR SS:\[EBP+<GetModhandle>\]                    ; kernel32.GetModuleHandleA  
003736EC    E9 E9120000     JMP     <proc\_Run\_FUN>  
003736F1    90              NOP  
003736F2    0BC0            OR      EAX, EAX  
003736F4    75 15           JNZ     SHORT 0037370B  
003736F6    56              PUSH    ESI  
003736F7    8D85 27204000   LEA     EAX, DWORD PTR SS:\[EBP+<strDllUsr32>\]  
003736FD    50              PUSH    EAX  
003736FE    8B85 AA434000   MOV     EAX, DWORD PTR SS:\[EBP+<APILoadLib>\]                     ; kernel32.LoadLibraryA  
00373704    E9 D1120000     JMP     <proc\_Run\_FUN>  
00373709    90              NOP  
0037370A    90              NOP  
0037370B    8BF0            MOV     ESI, EAX  
0037370D    8DBD 2A444000   LEA     EDI, DWORD PTR SS:\[EBP+<APIwsPrintfA>\]  
00373713    B9 07000000     MOV     ECX, 7  
00373718 >  57              PUSH    EDI                                                         ; De\_dllusr32\_api  
00373719    8A07            MOV     AL, BYTE PTR DS:\[EDI\]  
0037371B    EB 05           JMP     SHORT 00373722  
0037371D    F6D0            NOT     AL  
0037371F    AA              STOS    BYTE PTR ES:\[EDI\]                                           ; 解压出 api name  
00373720    8A07            MOV     AL, BYTE PTR DS:\[EDI\]  
00373722    0AC0            OR      AL, AL  
00373724  ^ 75 F7           JNZ     SHORT 0037371D  
00373726    5F              POP     EDI                                                         ; 0012FFE0  
00373727    51              PUSH    ECX  
00373728    57              PUSH    EDI  
00373729    56              PUSH    ESI  
0037372A    8D85 5A204000   LEA     EAX, DWORD PTR SS:\[EBP+40205A\]  
00373730    50              PUSH    EAX  
00373731    8B85 9E434000   MOV     EAX, DWORD PTR SS:\[EBP+<MyGetProcAddress>\]  
00373737    E9 9E120000     JMP     <proc\_Run\_FUN>  
0037373C    90              NOP  
0037373D    90              NOP  
0037373E    E8 FB140000     CALL    <proc\_check\_CC>  
00373743    50              PUSH    EAX  
00373744    57              PUSH    EDI  
00373745    56              PUSH    ESI  
00373746    8D85 75204000   LEA     EAX, DWORD PTR SS:\[EBP+402075\]  
0037374C    50              PUSH    EAX  
0037374D    8B85 9E434000   MOV     EAX, DWORD PTR SS:\[EBP+<MyGetProcAddress>\]  
00373753    E9 82120000     JMP     <proc\_Run\_FUN>  
00373758    90              NOP  
00373759    59              POP     ECX                                                         ; 0012FFE0  
0037375A    2BC8            SUB     ECX, EAX  
0037375C    2BC1            SUB     EAX, ECX  
0037375E    0FB64F FF       MOVZX   ECX, BYTE PTR DS:\[EDI\-1\]  
00373762    8907            MOV     DWORD PTR DS:\[EDI\], EAX  
00373764    03F9            ADD     EDI, ECX  
00373766    47              INC     EDI  
00373767    59              POP     ECX                                                         ; 0012FFE0  
00373768  ^ E2 AE           LOOPD   SHORT <De\_dllusr32\_api>                                     ; 循环获取 user32.dll 的相关函数  
0037376A    8DB5 58464000   LEA     ESI, DWORD PTR SS:\[EBP+<strDllws32\_32>\]  
00373770    56              PUSH    ESI  
00373771    8D85 A0204000   LEA     EAX, DWORD PTR SS:\[EBP+4020A0\]  
00373777    50              PUSH    EAX                                                         ; 载入 dll  
00373778    8B85 A6434000   MOV     EAX, DWORD PTR SS:\[EBP+<GetModhandle>\]                    ; kernel32.GetModuleHandleA  
0037377E    E9 57120000     JMP     <proc\_Run\_FUN>  
00373783    90              NOP  
00373784    0BC0            OR      EAX, EAX  
00373786    75 15           JNZ     SHORT 0037379D  
00373788    56              PUSH    ESI  
00373789    8D85 B9204000   LEA     EAX, DWORD PTR SS:\[EBP+4020B9\]  
0037378F    50              PUSH    EAX  
00373790    8B85 AA434000   MOV     EAX, DWORD PTR SS:\[EBP+<APILoadLib>\]                      ; kernel32.LoadLibraryA  
00373796    E9 3F120000     JMP     <proc\_Run\_FUN>  
0037379B    90              NOP  
0037379C    90              NOP  
0037379D    8BF0            MOV     ESI, EAX  
0037379F    8DBD 64464000   LEA     EDI, DWORD PTR SS:\[EBP+<APIwsASend>\]  
003737A5    B9 04000000     MOV     ECX, 4  
003737AA >  57              PUSH    EDI                                                         ; De\_ws2\_32\_apis  
003737AB    8A07            MOV     AL, BYTE PTR DS:\[EDI\]  
003737AD    EB 05           JMP     SHORT 003737B4  
003737AF    F6D0            NOT     AL  
003737B1    AA              STOS    BYTE PTR ES:\[EDI\]  
003737B2    8A07            MOV     AL, BYTE PTR DS:\[EDI\]  
003737B4    0AC0            OR      AL, AL  
003737B6  ^ 75 F7           JNZ     SHORT 003737AF  
003737B8    5F              POP     EDI                                                         ; 0012FFE0  
003737B9    51              PUSH    ECX  
003737BA    57              PUSH    EDI  
003737BB    56              PUSH    ESI  
003737BC    8D85 EC204000   LEA     EAX, DWORD PTR SS:\[EBP+4020EC\]  
003737C2    50              PUSH    EAX  
003737C3    8B85 9E434000   MOV     EAX, DWORD PTR SS:\[EBP+<MyGetProcAddress>\]  
003737C9    E9 0C120000     JMP     <proc\_Run\_FUN>  
003737CE    90              NOP  
003737CF    90              NOP  
003737D0    E8 69140000     CALL    <proc\_check\_CC>  
003737D5    50              PUSH    EAX  
003737D6    57              PUSH    EDI  
003737D7    56              PUSH    ESI  
003737D8    8D85 07214000   LEA     EAX, DWORD PTR SS:\[EBP+402107\]  
003737DE    50              PUSH    EAX  
003737DF    8B85 9E434000   MOV     EAX, DWORD PTR SS:\[EBP+<MyGetProcAddress>\]  
003737E5    E9 F0110000     JMP     <proc\_Run\_FUN>  
003737EA    90              NOP  
003737EB    59              POP     ECX                                                         ; 0012FFE0  
003737EC    2BC8            SUB     ECX, EAX  
003737EE    2BC1            SUB     EAX, ECX  
003737F0    0FB64F FF       MOVZX   ECX, BYTE PTR DS:\[EDI\-1\]  
003737F4    8907            MOV     DWORD PTR DS:\[EDI\], EAX  
003737F6    03F9            ADD     EDI, ECX  
003737F8    47              INC     EDI  
003737F9    59              POP     ECX                                                         ; 0012FFE0  
003737FA  ^ E2 AE           LOOPD   SHORT <De\_ws2\_32\_apis>                                      ; 循环处理 ws2\_32 的相关函数  
  
;        获取完毕  
;\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_  
  
  
获取完壳的相关 API 后，壳先对文件进行一次检测，判断文件大小是否被改变了,如果改变了则 OVER．  
  
003737FC    8B85 B2434000   MOV     EAX, DWORD PTR SS:\[EBP+<IMGBASE>\]                         ; 00400000  
00373802    0340 3C         ADD     EAX, DWORD PTR DS:\[EAX+3C\]                                  ; pe header  
00373805    8B40 50         MOV     EAX, DWORD PTR DS:\[EAX+50\]                                  ; 取出 SizofImage  
00373808    3385 C6434000   XOR     EAX, DWORD PTR SS:\[EBP+<xorsizeimg\_Key>\]                  ; 取出的 SizeofImage 和 key01a334f8 异或  
0037380E    8B8D A2434000   MOV     ECX, DWORD PTR SS:\[EBP+<xorkeyimag>\]  
00373814    3BC1            CMP     EAX, ECX                                                    ; 很简单的判断,sizeimg xor 01a334f8=01A6CC88 就 ok 了  
00373816    0F85 222A0000   JNZ     <Game\_Over>                                                 ; 如果文件大小改变了就 over 了  
  
  
  
;★★★★★★★★★★★★★★★★★★★★★★★★★★检测调试器★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★  
  
利用 CreateFileA 进行 ANTI 检测 \\\\.\\NTICE，\\\\.\\SICE，\\\\.\\TWX2002 \\\\.\\filemon，\\\\.\\regmon，\\\\.\\FILEVXD \\\\.\\REGVXD，\\\\.\\ICEDUMP，\\\\.\\BW2K  
这些检测对 OD 没有作用的:-)  
  
0037381C    8DB5 9C474000   LEA     ESI, DWORD PTR SS:\[EBP+<Anti\_str>\]  
00373822    46              INC     ESI  
00373823    B9 09000000     MOV     ECX, 9  
00373828    EB 58           JMP     SHORT 00373882  
0037382A >  51              PUSH    ECX                                                         ; Loop\_Check\_DBG  
0037382B    56              PUSH    ESI  
0037382C    AC              LODS    BYTE PTR DS:\[ESI\]  
0037382D    EB 06           JMP     SHORT 00373835  
0037382F >  F6D0            NOT     AL                                                          ; De\_Str  
00373831    8846 FF         MOV     BYTE PTR DS:\[ESI\-1\], AL                                     ; 解密出\\\\.\\xxxx  
00373834    AC              LODS    BYTE PTR DS:\[ESI\]  
00373835    0AC0            OR      AL, AL  
00373837  ^ 75 F6           JNZ     SHORT <De\_Str>  
00373839    5E              POP     ESI                                                         ; 0012FFE0  
0037383A    6A 00           PUSH    0  
0037383C    68 80000000     PUSH    80  
00373841    6A 03           PUSH    3  
00373843    6A 00           PUSH    0  
00373845    6A 03           PUSH    3  
00373847    68 000000C0     PUSH    C0000000  
0037384C    56              PUSH    ESI  
0037384D    8D85 7C214000   LEA     EAX, DWORD PTR SS:\[EBP+40217C\]  
00373853    50              PUSH    EAX  
00373854    8B85 BC444000   MOV     EAX, DWORD PTR SS:\[EBP+<APICreateFileA>\]                  ; kernel32.CreateFileA  
0037385A    E9 7B110000     JMP     <proc\_Run\_FUN>  
0037385F    90              NOP  
00373860    83F8 FF         CMP     EAX, -1  
00373863    74 05           JE      SHORT <No\_DBG>  
00373865    E9 D4290000     JMP     <Game\_Over>                                                 ; 检测到调试器就 over 了  
0037386A >  56              PUSH    ESI                                                         ; No\_DBG  
0037386B    AC              LODS    BYTE PTR DS:\[ESI\]  
0037386C    EB 06           JMP     SHORT 00373874  
0037386E >  F6D0            NOT     AL                                                          ; Crypt\_Str  
00373870    8846 FF         MOV     BYTE PTR DS:\[ESI\-1\], AL  
00373873    AC              LODS    BYTE PTR DS:\[ESI\]  
00373874    0AC0            OR      AL, AL  
00373876  ^ 75 F6           JNZ     SHORT <Crypt\_Str>  
00373878    5E              POP     ESI                                                         ; 0012FFE0  
00373879    59              POP     ECX                                                         ; 0012FFE0  
0037387A    0FB646 FF       MOVZX   EAX, BYTE PTR DS:\[ESI\-1\]  
0037387E    03F0            ADD     ESI, EAX  
00373880    46              INC     ESI  
00373881    49              DEC     ECX  
00373882    0BC9            OR      ECX, ECX  
00373884  ^ 75 A4           JNZ     SHORT <Loop\_Check\_DBG>  
  
;★★★★★★★★★★★★★★★★★★★★★★★★★★结束检测★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★  
  
  
00373886    83BD EA434000 0>CMP     DWORD PTR SS:\[EBP+<flg\_Hook9xDump>\], 1                    ; 判断是否选择了 Hook\_Win9x\_DUMP  
0037388D    0F85 60010000   JNZ     <no\_Hook\_DUMP>  
00373893    8CC9            MOV     CX, CS  
00373895    32C9            XOR     CL, CL  
00373897    0BC9            OR      ECX, ECX                                                    ; 检测 OS 版本  
00373899    0F84 A4000000   JE      <isWinNT>                                                   ; 如果是 WinNT 则跳  
0037389F    6A 40           PUSH    40                                                          ; 这里一段不明代码:-(  
003738A1    68 00100008     PUSH    8001000                                                     ; 因为我没有 Win9x,所以这里没有办法测试:-(  
003738A6    68 81000000     PUSH    81  
003738AB    6A 00           PUSH    0  
003738AD    FF95 AE434000   CALL    DWORD PTR SS:\[EBP+<APIVirtualAlloc>\]                      ; kernel32.VirtualAlloc  
003738B3    8985 F9494000   MOV     DWORD PTR SS:\[EBP+4049F9\], EAX  
003738B9    8BF8            MOV     EDI, EAX  
003738BB    8DB5 8E224000   LEA     ESI, DWORD PTR SS:\[EBP+40228E\]  
003738C1    B9 81000000     MOV     ECX, 81  
003738C6    F3:A4           REP     MOVS BYTE PTR ES:\[EDI\], BYTE PTR DS:\[ESI\]  
003738C8    8BD8            MOV     EBX, EAX  
003738CA    55              PUSH    EBP  
003738CB    8F83 79000000   POP     DWORD PTR DS:\[EBX+79\]                                       ; 0012FFE0  
003738D1    FFB5 D6444000   PUSH    DWORD PTR SS:\[EBP+4044D6\]                                   ; kernel32.ExitProcess  
003738D7    8F83 7D000000   POP     DWORD PTR DS:\[EBX+7D\]                                       ; 0012FFE0  
003738DD    8B85 46464000   MOV     EAX, DWORD PTR SS:\[EBP+404646\]                              ; kernel32.ReadProcessMemory  
003738E3    83C0 05         ADD     EAX, 5  
003738E6    8983 6B000000   MOV     DWORD PTR DS:\[EBX+6B\], EAX  
003738EC    8DBB 65000000   LEA     EDI, DWORD PTR DS:\[EBX+65\]  
003738F2    8BB5 46464000   MOV     ESI, DWORD PTR SS:\[EBP+404646\]                              ; kernel32.ReadProcessMemory  
003738F8    803E E9         CMP     BYTE PTR DS:\[ESI\], 0E9  
003738FB    74 09           JE      SHORT 00373906  
003738FD    B9 05000000     MOV     ECX, 5  
00373902    F3:A4           REP     MOVS BYTE PTR ES:\[EDI\], BYTE PTR DS:\[ESI\]  
00373904    EB 0D           JMP     SHORT 00373913  
00373906    8B46 01         MOV     EAX, DWORD PTR DS:\[ESI+1\]  
00373909    03C6            ADD     EAX, ESI  
0037390B    2BC7            SUB     EAX, EDI  
0037390D    8947 01         MOV     DWORD PTR DS:\[EDI+1\], EAX  
00373910    C607 E9         MOV     BYTE PTR DS:\[EDI\], 0E9  
00373913    50              PUSH    EAX  
00373914    0F014C24 FE     SIDT    FWORD PTR SS:\[ESP\-2\]  
00373919    5F              POP     EDI                                                         ; 0012FFE0  
0037391A    83C7 20         ADD     EDI, 20  
0037391D    8B4F 04         MOV     ECX, DWORD PTR DS:\[EDI+4\]  
00373920    66:8B0F         MOV     CX, WORD PTR DS:\[EDI\]  
00373923    FA              CLI  
00373924    8DB5 64224000   LEA     ESI, DWORD PTR SS:\[EBP+402264\]  
0037392A    66:8937         MOV     WORD PTR DS:\[EDI\], SI  
0037392D    C1EE 10         SHR     ESI, 10  
00373930    66:8977 06      MOV     WORD PTR DS:\[EDI+6\], SI  
00373934    FB              STI  
00373935    CD 04           INT     4  
00373937    FA              CLI  
00373938    66:890F         MOV     WORD PTR DS:\[EDI\], CX  
0037393B    C1E9 10         SHR     ECX, 10  
0037393E    66:894F 06      MOV     WORD PTR DS:\[EDI+6\], CX  
00373942    FB              STI  
00373943 >  E9 AB000000     JMP     <no\_Hook\_DUMP>                                              ; isWinNT  
00373948    60              PUSHAD  
00373949    E8 00000000     CALL    0037394E  
0037394E    5D              POP     EBP                                                         ; 0012FFE0  
0037394F    81ED 6A224000   SUB     EBP, 40226A  
00373955    8B85 F9494000   MOV     EAX, DWORD PTR SS:\[EBP+4049F9\]  
0037395B    2B85 46464000   SUB     EAX, DWORD PTR SS:\[EBP+404646\]                              ; kernel32.ReadProcessMemory  
00373961    83E8 05         SUB     EAX, 5  
00373964    8B8D 46464000   MOV     ECX, DWORD PTR SS:\[EBP+404646\]                              ; kernel32.ReadProcessMemory  
0037396A    C601 E9         MOV     BYTE PTR DS:\[ECX\], 0E9  
0037396D    8941 01         MOV     DWORD PTR DS:\[ECX+1\], EAX  
00373970    61              POPAD  
00373971    CF              IRETD  
00373972    9C              PUSHFD  
00373973    60              PUSHAD  
00373974    E8 00000000     CALL    00373979  
00373979    5D              POP     EBP                                                         ; 0012FFE0  
0037397A    81ED 95224000   SUB     EBP, 402295  
00373980    8B7424 28       MOV     ESI, DWORD PTR SS:\[ESP+28\]                                  ; kernel32.7C816D4F  
00373984    8D85 03234000   LEA     EAX, DWORD PTR SS:\[EBP+402303\]  
0037398A    50              PUSH    EAX  
0037398B    6A 04           PUSH    4  
0037398D    8D85 FF224000   LEA     EAX, DWORD PTR SS:\[EBP+4022FF\]  
00373993    50              PUSH    EAX  
00373994    B8 57484000     MOV     EAX, 404857  
00373999    0385 07234000   ADD     EAX, DWORD PTR SS:\[EBP+402307\]  
0037399F    50              PUSH    EAX  
003739A0    56              PUSH    ESI  
003739A1    E8 31000000     CALL    003739D7  
003739A6    0BC0            OR      EAX, EAX  
003739A8    74 2B           JE      SHORT 003739D5  
003739AA    B8 2635B204     MOV     EAX, 4B23526  
003739AF    3985 FF224000   CMP     DWORD PTR SS:\[EBP+4022FF\], EAX  
003739B5    75 1E           JNZ     SHORT 003739D5  
003739B7    8B7C24 30       MOV     EDI, DWORD PTR SS:\[ESP+30\]  
003739BB    8B4C24 34       MOV     ECX, DWORD PTR SS:\[ESP+34\]  
003739BF    33C0            XOR     EAX, EAX  
003739C1    F3:AA           REP     STOS BYTE PTR ES:\[EDI\]  
003739C3    8B4424 34       MOV     EAX, DWORD PTR SS:\[ESP+34\]  
003739C7    8B4C24 38       MOV     ECX, DWORD PTR SS:\[ESP+38\]  
003739CB    8901            MOV     DWORD PTR DS:\[ECX\], EAX  
003739CD    61              POPAD  
003739CE    9D              POPFD  
003739CF    33C0            XOR     EAX, EAX  
003739D1    40              INC     EAX  
003739D2    C2 1400         RETN    14  
003739D5    61              POPAD  
003739D6    9D              POPFD  
003739D7    55              PUSH    EBP  
003739D8    8BEC            MOV     EBP, ESP  
003739DA    56              PUSH    ESI  
003739DB    57              PUSH    EDI  
003739DC    B8 00000000     MOV     EAX, 0  
003739E1    FFE0            JMP     EAX  
003739E3    0000            ADD     BYTE PTR DS:\[EAX\], AL  
003739E5    0000            ADD     BYTE PTR DS:\[EAX\], AL  
003739E7    0000            ADD     BYTE PTR DS:\[EAX\], AL  
003739E9    0000            ADD     BYTE PTR DS:\[EAX\], AL  
003739EB    0000            ADD     BYTE PTR DS:\[EAX\], AL  
003739ED    0000            ADD     BYTE PTR DS:\[EAX\], AL  
003739EF    0000            ADD     BYTE PTR DS:\[EAX\], AL  
003739F1    0000            ADD     BYTE PTR DS:\[EAX\], AL  
  
跳过上面一段对 WIN9X\_hook\_anti\_dump 的处理后来到这里,准备对文件进行解密操作.  
  
003739F3 >  8D1D 65494000   LEA     EBX, DWORD PTR DS:\[404965\]                                  ; no\_Hook\_DUMP  
003739F9 >  833C2B 00       CMP     DWORD PTR DS:\[EBX+EBP\], 0                                   ; Loop\_De\_Code  
003739FD    0F84 7B010000   JE      <Unpacked\_Section\_DONE>                                     ; 判断节的大小是否为 0,是否解压完相关的段  
00373A03    8D042B          LEA     EAX, DWORD PTR DS:\[EBX+EBP\]  
00373A06    8B48 08         MOV     ECX, DWORD PTR DS:\[EAX+8\]  
00373A09    8B70 04         MOV     ESI, DWORD PTR DS:\[EAX+4\]  
00373A0C    03B5 B2434000   ADD     ESI, DWORD PTR SS:\[EBP+<IMGBASE>\]                         ; 00400000  
00373A12    8BFE            MOV     EDI, ESI  
00373A14    BA 2635B204     MOV     EDX, 4B23526  
00373A19    EB 1F           JMP     SHORT 00373A3A  
00373A1B    AC              LODS    BYTE PTR DS:\[ESI\]                                           ; 比较简单的第一次解压出代码  
00373A1C    D2C8            ROR     AL, CL  
00373A1E    32C1            XOR     AL, CL  
00373A20    04 66           ADD     AL, 66  
00373A22    32C5            XOR     AL, CH  
00373A24    02C6            ADD     AL, DH  
00373A26    2AC2            SUB     AL, DL  
00373A28    02C1            ADD     AL, CL  
00373A2A    2AC5            SUB     AL, CH  
00373A2C    32C2            XOR     AL, DL  
00373A2E    04 23           ADD     AL, 23  
00373A30    32C6            XOR     AL, DH  
00373A32    F6D0            NOT     AL  
00373A34    D2C8            ROR     AL, CL  
00373A36    D3CA            ROR     EDX, CL  
00373A38    AA              STOS    BYTE PTR ES:\[EDI\]  
00373A39    49              DEC     ECX  
00373A3A    0BC9            OR      ECX, ECX  
00373A3C  ^ 75 DD           JNZ     SHORT 00373A1B  
00373A3E    53              PUSH    EBX  
00373A3F    6A 04           PUSH    4  
00373A41    68 00100000     PUSH    1000  
00373A46    FF342B          PUSH    DWORD PTR DS:\[EBX+EBP\]  
00373A49    6A 00           PUSH    0  
00373A4B    8D85 42244000   LEA     EAX, DWORD PTR SS:\[EBP+402442\]  
00373A51    50              PUSH    EAX  
00373A52    8B85 AE434000   MOV     EAX, DWORD PTR SS:\[EBP+<APIVirtualAlloc>\]                 ; kernel32.VirtualAlloc  
00373A58    E9 7D0F0000     JMP     <proc\_Run\_FUN>  
00373A5D    90              NOP  
  
这里有有一个远的返回地址了，不再向前面的那样返回地址就在 3 行代码之内．  
  
;☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆跳去 OEP 的代码☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆  
  
  我是直接按壳的代码一直 COPY，然后再做标记的，那样对全面分析可能会更好看些吧,当然这样看了也就会有点乱，  
  
00373A5E    8B85 0E444000   MOV     EAX, DWORD PTR SS:\[EBP+<OEP(RVA)>\]                        ; 处理完输入表和加密块后经过８个异常后来到这里  
00373A64    0385 B2434000   ADD     EAX, DWORD PTR SS:\[EBP+<IMGBASE>\]                         ; 准备跳去 oep 的处理  
00373A6A    90              NOP  
......  
00373A71    894424 EC       MOV     DWORD PTR SS:\[ESP\-14\], EAX  
00373A75    90              NOP  
00373A76    90              NOP  
00373A77    90              NOP  
00373A78    90              NOP  
00373A79    90              NOP  
00373A7A    90              NOP  
00373A7B    90              NOP  
00373A7C    90              NOP  
00373A7D    90              NOP  
00373A7E    90              NOP  
00373A7F    90              NOP  
00373A80    896C24 E8       MOV     DWORD PTR SS:\[ESP\-18\], EBP  
00373A84    90              NOP  
00373A85    90              NOP  
00373A86    90              NOP  
00373A87    90              NOP  
00373A88    FF85 C2434000   INC     DWORD PTR SS:\[EBP+4043C2\]  
00373A8E    90              NOP  
00373A8F    90              NOP  
00373A90    90              NOP  
00373A91    90              NOP  
00373A92    90              NOP  
00373A93    90              NOP  
00373A94    90              NOP  
00373A95    90              NOP  
00373A96    90              NOP  
00373A97    90              NOP  
00373A98    90              NOP  
00373A99    8B9D EE434000   MOV     EBX, DWORD PTR SS:\[EBP+4043EE\]  
00373A9F    90              NOP  
00373AA0    90              NOP  
00373AA1    90              NOP  
00373AA2    90              NOP  
00373AA3    90              NOP  
00373AA4    83FB 01         CMP     EBX, 1                                                      ; EBX=1 表示是 push ebp, mov ebp,esp 的方式  
00373AA7    75 33           JNZ     SHORT <isVBOEP\_MODE?>  
00373AA9    61              POPAD                                                               ; is Push ebp mode  
00373AAA    90              NOP  
00373AAB    90              NOP  
00373AAC    90              NOP  
00373AAD    90              NOP  
00373AAE    90              NOP  
00373AAF    8B4424 CC       MOV     EAX, DWORD PTR SS:\[ESP\-34\]  
00373AB3    90              NOP  
00373AB4    90              NOP  
00373AB5    90              NOP  
00373AB6    90              NOP  
00373AB7    90              NOP  
00373AB8    90              NOP  
00373AB9    90              NOP  
00373ABA    90              NOP  
00373ABB    90              NOP  
00373ABC    90              NOP  
00373ABD    90              NOP  
00373ABE    8D78 02         LEA     EDI, DWORD PTR DS:\[EAX+2\]  
00373AC1    90              NOP  
00373AC2    90              NOP  
00373AC3    90              NOP  
00373AC4    90              NOP  
00373AC5    90              NOP  
00373AC6    55              PUSH    EBP  
00373AC7    90              NOP  
00373AC8    90              NOP  
00373AC9    90              NOP  
00373ACA    90              NOP  
00373ACB    90              NOP  
00373ACC    8BEC            MOV     EBP, ESP  
00373ACE    90              NOP  
00373ACF    90              NOP  
00373AD0    90              NOP  
00373AD1    90              NOP  
00373AD2    90              NOP  
00373AD3    90              NOP  
00373AD4    90              NOP  
00373AD5    90              NOP  
00373AD6    90              NOP  
00373AD7    90              NOP  
00373AD8    90              NOP  
00373AD9    50              PUSH    EAX  
00373ADA    EB 45           JMP     SHORT 00373B21  
00373ADC >  83FB 02         CMP     EBX, 2                                                      ; isVBOEP\_MODE?  
00373ADF    75 2E           JNZ     SHORT 00373B0F                                              ; EBX==2 表示是 VB 的程序 push address  
00373AE1    61              POPAD  
00373AE2    90              NOP  
00373AE3    90              NOP  
00373AE4    90              NOP  
00373AE5    90              NOP  
00373AE6    90              NOP  
00373AE7    90              NOP  
00373AE8    90              NOP  
00373AE9    8B4424 C8       MOV     EAX, DWORD PTR SS:\[ESP\-38\]  
00373AED    90              NOP  
00373AEE    90              NOP  
00373AEF    90              NOP  
00373AF0    90              NOP  
00373AF1    FFB0 F2434000   PUSH    DWORD PTR DS:\[EAX+4043F2\]  
00373AF7    90              NOP  
00373AF8    90              NOP  
00373AF9    90              NOP  
00373AFA    90              NOP  
00373AFB    90              NOP  
00373AFC    8B4424 D0       MOV     EAX, DWORD PTR SS:\[ESP\-30\]                                  ; ntdll.7C930551  
00373B00    90              NOP  
00373B01    90              NOP  
00373B02    90              NOP  
00373B03    90              NOP  
00373B04    90              NOP  
00373B05    50              PUSH    EAX  
00373B06    90              NOP  
00373B07    90              NOP  
00373B08    90              NOP  
00373B09    90              NOP  
00373B0A    8D78 02         LEA     EDI, DWORD PTR DS:\[EAX+2\]  
00373B0D    EB 12           JMP     SHORT 00373B21  
00373B0F    61              POPAD                                                               ; 如果不是 push ebp mov ebp,esp 的方式  
00373B10    90              NOP                                                                 ; 也不是 push address call address 的方式则直接跳去 oep  
00373B11    90              NOP  
00373B12    90              NOP  
00373B13    90              NOP  
00373B14    90              NOP  
00373B15    90              NOP  
00373B16    90              NOP  
00373B17    8B4424 CC       MOV     EAX, DWORD PTR SS:\[ESP\-34\]  
00373B1B    90              NOP  
00373B1C    90              NOP  
00373B1D    90              NOP  
00373B1E    90              NOP  
00373B1F    90              NOP  
00373B20    50              PUSH    EAX  
00373B21    90              NOP  
00373B22    90              NOP  
00373B23    90              NOP  
00373B24    90              NOP  
00373B25    C3              RETN                                                                ; 返回到程序 OEP  
  
;☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆跳去 OEP 代码结束☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆  
  
上面申请空间后直接返回到这里，继续分析下去:  
  
00373B26    5B              POP     EBX                                                         ; 0012FFE0  
00373B27    8BF0            MOV     ESI, EAX  
00373B29    8BC3            MOV     EAX, EBX  
00373B2B    03C5            ADD     EAX, EBP  
00373B2D    8B78 04         MOV     EDI, DWORD PTR DS:\[EAX+4\]  
00373B30    03BD B2434000   ADD     EDI, DWORD PTR SS:\[EBP+<IMGBASE>\]                         ; 00400000  
00373B36    56              PUSH    ESI  
00373B37    57              PUSH    EDI  
00373B38    8D85 64244000   LEA     EAX, DWORD PTR SS:\[EBP+402464\]  
00373B3E    50              PUSH    EAX  
00373B3F    8B85 12444000   MOV     EAX, DWORD PTR SS:\[EBP+404412\]                              ; <aplib\_Unpack>  
00373B45    FFE0            JMP     EAX                                                         ; 跳去 aplib 解压代码  
00373B48    8B0C2B          MOV     ECX, DWORD PTR DS:\[EBX+EBP\]                                 ; 解压大小 59000  
00373B4B    56              PUSH    ESI  
00373B4C    51              PUSH    ECX  
00373B4D    C1E9 02         SHR     ECX, 2  
00373B50    F3:A5           REP     MOVS DWORD PTR ES:\[EDI\], DWORD PTR DS:\[ESI\]                 ; 解压代码到相关位置  
00373B52    59              POP     ECX                                                         ; 0012FFE0  
00373B53    83E1 03         AND     ECX, 3  
00373B56    F3:A4           REP     MOVS BYTE PTR ES:\[EDI\], BYTE PTR DS:\[ESI\]  
00373B58    5E              POP     ESI                                                         ; 0012FFE0  
00373B59    53              PUSH    EBX  
00373B5A    68 00800000     PUSH    8000  
00373B5F    6A 00           PUSH    0  
00373B61    56              PUSH    ESI  
00373B62    8D85 91244000   LEA     EAX, DWORD PTR SS:\[EBP+402491\]  
00373B68    50              PUSH    EAX  
00373B69    8B85 AF444000   MOV     EAX, DWORD PTR SS:\[EBP+<APIVirtualFree>\]                  ; kernel32.VirtualFree  
00373B6F    E9 660E0000     JMP     <proc\_Run\_FUN>  
00373B74    90              NOP  
00373B75    5B              POP     EBX                                                         ; 0012FFE0  
00373B76    83C3 0C         ADD     EBX, 0C  
00373B79  ^ E9 7BFEFFFF     JMP     <Loop\_De\_Code>                                              ; 循环解压出相关代码  
  
  
;══════════════════════还原 CALL 和相关跳转═════════════════════════  
  
00373B7E >  8BB5 DD484000   MOV     ESI, DWORD PTR SS:\[EBP+4048DD\]                              ; Unpacked\_Section\_DONE  
00373B84    03B5 B2434000   ADD     ESI, DWORD PTR SS:\[EBP+<IMGBASE>\]                         ; 00400000  
00373B8A    8B8D E1484000   MOV     ECX, DWORD PTR SS:\[EBP+4048E1\]  
00373B90    83E9 05         SUB     ECX, 5  
00373B93    EB 5B           JMP     SHORT 00373BF0  
00373B95 >  66:8B06         MOV     AX, WORD PTR DS:\[ESI\]                                       ; De\_JMP\_CALL  
00373B98    3C E8           CMP     AL, 0E8                                                     ; jmp address  
00373B9A    75 16           JNZ     SHORT 00373BB2                                              ; 和远程跳转  
00373B9C    8BC6            MOV     EAX, ESI  
00373B9E    2B85 B2434000   SUB     EAX, DWORD PTR SS:\[EBP+<IMGBASE>\]                         ; 00400000  
00373BA4    83C0 05         ADD     EAX, 5  
00373BA7    2946 01         SUB     DWORD PTR DS:\[ESI+1\], EAX  
00373BAA    83C6 04         ADD     ESI, 4  
00373BAD    83E9 04         SUB     ECX, 4  
00373BB0    EB 3C           JMP     SHORT 00373BEE  
00373BB2    3C E9           CMP     AL, 0E9  
00373BB4    75 16           JNZ     SHORT 00373BCC  
00373BB6    8BC6            MOV     EAX, ESI  
00373BB8    2B85 B2434000   SUB     EAX, DWORD PTR SS:\[EBP+<IMGBASE>\]                         ; 00400000  
00373BBE    83C0 05         ADD     EAX, 5  
00373BC1    2946 01         SUB     DWORD PTR DS:\[ESI+1\], EAX  
00373BC4    83C6 04         ADD     ESI, 4  
00373BC7    83E9 04         SUB     ECX, 4  
00373BCA    EB 22           JMP     SHORT 00373BEE  
00373BCC    3C 0F           CMP     AL, 0F  
00373BCE    75 1E           JNZ     SHORT 00373BEE  
00373BD0    80FC 7F         CMP     AH, 7F  
00373BD3    76 19           JBE     SHORT 00373BEE  
00373BD5    80FC 90         CMP     AH, 90  
00373BD8    73 14           JNB     SHORT 00373BEE  
00373BDA    8BC6            MOV     EAX, ESI  
00373BDC    2B85 B2434000   SUB     EAX, DWORD PTR SS:\[EBP+<IMGBASE>\]                         ; 00400000  
00373BE2    83C0 06         ADD     EAX, 6  
00373BE5    2946 02         SUB     DWORD PTR DS:\[ESI+2\], EAX  
00373BE8    83C6 05         ADD     ESI, 5  
00373BEB    83E9 05         SUB     ECX, 5  
00373BEE    46              INC     ESI  
00373BEF    49              DEC     ECX  
00373BF0    81F9 00000080   CMP     ECX, 80000000  
00373BF6  ^ 72 9D           JB      SHORT <De\_JMP\_CALL>  
  
  
;══════════════════════END══════════════════════ 

......  
00373BF8    8DB5 4E254000   LEA     ESI, DWORD PTR SS:\[EBP+<Next\_Decode\_addr>\]                ; 开始解出下一段代码  
00373BFE    87E6            XCHG    ESI, ESP  
00373C00    B9 930B0000     MOV     ECX, 0B93                                                   ; 解压大小 0b93  
00373C05    58              POP     EAX                                                         ; 0012FFE0  
00373C06    F6D0            NOT     AL  
00373C08    50              PUSH    EAX  
00373C09    44              INC     ESP  
00373C0A  ^ E2 F9           LOOPD   SHORT 00373C05  
00373C0C    87E6            XCHG    ESI, ESP  
  
  
  
;⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕输入表处理⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕  
  
  
解压出代码后开始处理输入表部分了，输入表部分是复杂了点．总体是这样子,simonzh2k 和 Window 已经标的很明白，我直接搬了过来:  
  
加密后的 IAT 在内存里如下存放(搬了 simonzh2k 的)  
  ; 1. FF FF FF FF    ----------- 00 00 00 00 表示 所有 DLL 结束  
  ; 2. xx   ----- DLL Name　长度(不算 null)  
  ; 3. DLL 名字, null 结尾 ( 明文 )  
  ; 4. 80 yy yy yy    ---------- yy yy yy 表示　API 个数 ,  80 表示需要重定向  
  ; 5. zz             ---------- ZZ<>0 表示 API Name　长度(不算 null),   ZZ==0, 后 4 byte 函数序号, 1 byte NULL  
  ; 6. API Name, null 结尾 ( 密文, 解密代码见 12FF68 )  
  ; 7. 重复 5,  6 结束 一个 DLL  
  ; 重复 1,..,7 处理 所有 DLL  
  
  
  
; 经过壳的 iat 处理,形成了下面的一个调用过程, ( 引用 window 的表示)  
;  
; iat 中地址 --> Hook\_proc:  
; Hook\_proc:   
;            |PUSH DWORD PTR DS:\[Hook\_proc+1C\]  
;            |XOR DWORD PTR SS:\[ESP\], key  
;            |ret; -> |Stub\_proc:  
;                         |api\_start\_code  
                          |api\_some\_code  
                          |push api\_next\_code\_addr  
                          |ret  
  
  
  
00373C0E    6A 04           PUSH    4  
00373C10    68 00100000     PUSH    1000  
00373C15    68 00200000     PUSH    2000  
00373C1A    6A 00           PUSH    0  
00373C1C    FF95 AE434000   CALL    DWORD PTR SS:\[EBP+<APIVirtualAlloc>\]                      ; kernel32.VirtualAlloc  
00373C22    8985 E9494000   MOV     DWORD PTR SS:\[EBP+<hvMEM>\], EAX  
00373C28    C785 ED494000 0>MOV     DWORD PTR SS:\[EBP+<shellFunAddress>\], 0  
00373C32    8B85 DE434000   MOV     EAX, DWORD PTR SS:\[EBP+<flgCrypt\_Improt>\]                 ; (initial cpu selection)  
00373C38    0BC0            OR      EAX, EAX  
00373C3A    0F85 BD000000   JNZ     <IAT\_isCrypted>                                             ; 如果输入表加密了则跳  
00373C40    8BBD 02444000   MOV     EDI, DWORD PTR SS:\[EBP+<IAT\_RVA>\]                         ; 如果没有加密则这里是输入表的 rva，加密了就不是了  
00373C46    03BD B2434000   ADD     EDI, DWORD PTR SS:\[EBP+<IMGBASE>\]                         ; 00400000  
00373C4C >  8B77 0C         MOV     ESI, DWORD PTR DS:\[EDI+C\]                                   ; dis\_Dlls  
00373C4F    0BF6            OR      ESI, ESI  
00373C51    75 05           JNZ     SHORT <dis\_iat>                                             ; 如果没有处理完输入表则跳  
00373C53    E9 A0000000     JMP     <not\_crypt\_IAT\_dis\_Done>  
00373C58 >  03B5 B2434000   ADD     ESI, DWORD PTR SS:\[EBP+<IMGBASE>\]                         ; dis\_iat  
00373C5E    56              PUSH    ESI  
00373C5F    8D85 8F254000   LEA     EAX, DWORD PTR SS:\[EBP+40258F\]  
00373C65    50              PUSH    EAX  
00373C66    8B85 A6434000   MOV     EAX, DWORD PTR SS:\[EBP+<GetModhandle>\]                    ; kernel32.GetModuleHandleA  
00373C6C    E9 690D0000     JMP     <proc\_Run\_FUN>                                              ; 判断 DLL 是否已经载入  
00373C71    90              NOP  
00373C72    90              NOP  
00373C73    0BC0            OR      EAX, EAX  
00373C75    75 1E           JNZ     SHORT <dll\_isLoaded>  
00373C77    56              PUSH    ESI  
00373C78    8D85 A8254000   LEA     EAX, DWORD PTR SS:\[EBP+4025A8\]  
00373C7E    50              PUSH    EAX  
00373C7F    8B85 AA434000   MOV     EAX, DWORD PTR SS:\[EBP+<APILoadLib>\]                      ; kernel32.LoadLibraryA  
00373C85    E9 500D0000     JMP     <proc\_Run\_FUN>  
00373C8A    90              NOP  
00373C8B    90              NOP  
00373C8C    0BC0            OR      EAX, EAX  
00373C8E    75 05           JNZ     SHORT <dll\_isLoaded>  
00373C90    E9 5A0F0000     JMP     <proc\_Loaddll\_failed>                                       ; 载入 DLL 失败显示失败信息  
00373C95 >  8BF0            MOV     ESI, EAX                                                    ; dll\_isLoaded  
00373C97    8B17            MOV     EDX, DWORD PTR DS:\[EDI\]  
00373C99    0BD2            OR      EDX, EDX  
00373C9B    75 03           JNZ     SHORT 00373CA0  
00373C9D    8B57 10         MOV     EDX, DWORD PTR DS:\[EDI+10\]                                  ; 004480AC  
00373CA0    0395 B2434000   ADD     EDX, DWORD PTR SS:\[EBP+<IMGBASE>\]                         ; 00400000  
00373CA6    8B5F 10         MOV     EBX, DWORD PTR DS:\[EDI+10\]                                  ; 004480AC  
00373CA9    039D B2434000   ADD     EBX, DWORD PTR SS:\[EBP+<IMGBASE>\]                         ; 00400000  
00373CAF >  8B02            MOV     EAX, DWORD PTR DS:\[EDX\]                                     ; dis\_current\_DLLs\_api  
00373CB1    0BC0            OR      EAX, EAX  
00373CB3    75 02           JNZ     SHORT 00373CB7  
00373CB5    EB 39           JMP     SHORT 00373CF0  
00373CB7    53              PUSH    EBX  
00373CB8    52              PUSH    EDX  
00373CB9    99              CDQ  
00373CBA    0BD2            OR      EDX, EDX  
00373CBC    75 0B           JNZ     SHORT <is\_number1>                                          ; 是序号还是 API 名字  
00373CBE    83C0 02         ADD     EAX, 2  
00373CC1    0385 B2434000   ADD     EAX, DWORD PTR SS:\[EBP+<IMGBASE>\]                         ; 00400000  
00373CC7    EB 05           JMP     SHORT 00373CCE  
00373CC9 >  25 FFFFFF7F     AND     EAX, 7FFFFFFF                                               ; is\_number1  
00373CCE    50              PUSH    EAX  
00373CCF    56              PUSH    ESI  
00373CD0    8D85 00264000   LEA     EAX, DWORD PTR SS:\[EBP+402600\]  
00373CD6    50              PUSH    EAX  
00373CD7    8B85 9E434000   MOV     EAX, DWORD PTR SS:\[EBP+<MyGetProcAddress>\]  
00373CDD    E9 F80C0000     JMP     <proc\_Run\_FUN>  
00373CE2    90              NOP  
00373CE3    90              NOP  
00373CE4    8903            MOV     DWORD PTR DS:\[EBX\], EAX                                     ; 填充输入表  
00373CE6    5A              POP     EDX                                                         ; 0012FFE0  
00373CE7    5B              POP     EBX                                                         ; 0012FFE0  
00373CE8    83C2 04         ADD     EDX, 4  
00373CEB    83C3 04         ADD     EBX, 4  
00373CEE  ^ EB BF           JMP     SHORT <dis\_current\_DLLs\_api>  
00373CF0    83C7 14         ADD     EDI, 14  
00373CF3  ^ E9 54FFFFFF     JMP     <dis\_Dlls>                                                  ; 循环填充输入表  
00373CF8 >  E9 C6050000     JMP     <Disposal\_IAT\_Done>                                         ; not\_crypt\_IAT\_dis\_Done  
00373CFD >  8D95 A01A4000   LEA     EDX, DWORD PTR SS:\[EBP+<Crc\_Start\_addr>\]                  ; IAT\_isCrypted  
00373D03    0395 02444000   ADD     EDX, DWORD PTR SS:\[EBP+<IAT\_RVA>\]  
00373D09 >  8B3A            MOV     EDI, DWORD PTR DS:\[EDX\]                                     ; loop\_De\_Crypted\_iat  
00373D0B    0BFF            OR      EDI, EDI  
00373D0D    75 05           JNZ     SHORT <DIS\_NEXT\_1>                                          ; 如果没有处理完 IAT 则跳  
00373D0F    E9 AF050000     JMP     <Disposal\_IAT\_Done>  
00373D14 >  03BD B2434000   ADD     EDI, DWORD PTR SS:\[EBP+<IMGBASE>\]                         ; DIS\_NEXT\_1  
00373D1A    83C2 05         ADD     EDX, 5  
00373D1D    8BF2            MOV     ESI, EDX  
00373D1F    56              PUSH    ESI  
00373D20    8D85 50264000   LEA     EAX, DWORD PTR SS:\[EBP+402650\]  
00373D26    50              PUSH    EAX  
00373D27    8B85 A6434000   MOV     EAX, DWORD PTR SS:\[EBP+<GetModhandle>\]                    ; kernel32.GetModuleHandleA  
00373D2D    E9 A80C0000     JMP     <proc\_Run\_FUN>  
00373D32    90              NOP  
00373D33    90              NOP  
00373D34    0BC0            OR      EAX, EAX  
00373D36    75 1E           JNZ     SHORT 00373D56  
00373D38    56              PUSH    ESI  
00373D39    8D85 69264000   LEA     EAX, DWORD PTR SS:\[EBP+402669\]  
00373D3F    50              PUSH    EAX  
00373D40    8B85 AA434000   MOV     EAX, DWORD PTR SS:\[EBP+<APILoadLib>\]                      ; kernel32.LoadLibraryA  
00373D46    E9 8F0C0000     JMP     <proc\_Run\_FUN>  
00373D4B    90              NOP  
00373D4C    90              NOP  
00373D4D    0BC0            OR      EAX, EAX  
00373D4F    75 05           JNZ     SHORT 00373D56  
00373D51    E9 990E0000     JMP     <proc\_Loaddll\_failed>  
00373D56    0FB64E FF       MOVZX   ECX, BYTE PTR DS:\[ESI\-1\]                                    ; MODULE NAME 长度  
00373D5A    03F1            ADD     ESI, ECX  
00373D5C    8BD6            MOV     EDX, ESI  
00373D5E    8BF0            MOV     ESI, EAX  
00373D60    42              INC     EDX  
00373D61    8B0A            MOV     ECX, DWORD PTR DS:\[EDX\]                                     ; 本 DLL 需引入函数的数  
00373D63    81E1 00000080   AND     ECX, 80000000  
00373D69    0BC9            OR      ECX, ECX  
00373D6B    0F85 87000000   JNZ     <Reloc\_FUN>                                                 ; 判断 DLL 中的 API 是否需要重定位处理,如果需要则跳  
00373D71    8B0A            MOV     ECX, DWORD PTR DS:\[EDX\]                                     ; 不需要特殊处理的 API 则跳这里  
00373D73    83C2 04         ADD     EDX, 4  
00373D76 >  51              PUSH    ECX                                                         ; loop\_not\_relocs\_api  
00373D77    0FB602          MOVZX   EAX, BYTE PTR DS:\[EDX\]  
00373D7A    0BC0            OR      EAX, EAX  
00373D7C    75 27           JNZ     SHORT <not\_reloc\_Ord\_by\_name>  
00373D7E    42              INC     EDX                                                         ; 以序号方式填充  
00373D7F    52              PUSH    EDX  
00373D80    8B02            MOV     EAX, DWORD PTR DS:\[EDX\]  
00373D82    50              PUSH    EAX  
00373D83    56              PUSH    ESI  
00373D84    8D85 B4264000   LEA     EAX, DWORD PTR SS:\[EBP+4026B4\]  
00373D8A    50              PUSH    EAX  
00373D8B    8B85 9E434000   MOV     EAX, DWORD PTR SS:\[EBP+<MyGetProcAddress>\]  
00373D91    E9 440C0000     JMP     <proc\_Run\_FUN>  
00373D96    90              NOP  
00373D97    36:E8 A10E0000  CALL    <proc\_check\_CC>                                             ; Superfluous prefix  
00373D9D    8907            MOV     DWORD PTR DS:\[EDI\], EAX                                     ; 填充 IAT  
00373D9F    5A              POP     EDX                                                         ; 0012FFE0  
00373DA0    83C2 04         ADD     EDX, 4  
00373DA3    EB 47           JMP     SHORT 00373DEC  
00373DA5 >  42              INC     EDX                                                         ; not\_reloc\_Ord\_by\_name  
00373DA6    52              PUSH    EDX  
00373DA7    60              PUSHAD  
00373DA8    8BF2            MOV     ESI, EDX  
00373DAA    8DBD 74484000   LEA     EDI, DWORD PTR SS:\[EBP+<strAPIName>\]                      ; 循环解压出 API 名称  
00373DB0    33C0            XOR     EAX, EAX  
00373DB2    AC              LODS    BYTE PTR DS:\[ESI\]  
00373DB3    EB 07           JMP     SHORT 00373DBC  
00373DB5    C0C0 03         ROL     AL, 3  
00373DB8    F6D0            NOT     AL  
00373DBA    AA              STOS    BYTE PTR ES:\[EDI\]  
00373DBB    AC              LODS    BYTE PTR DS:\[ESI\]  
00373DBC    0BC0            OR      EAX, EAX  
00373DBE  ^ 75 F5           JNZ     SHORT 00373DB5  
00373DC0    AA              STOS    BYTE PTR ES:\[EDI\]  
00373DC1    61              POPAD  
00373DC2    8D95 74484000   LEA     EDX, DWORD PTR SS:\[EBP+<strAPIName>\]  
00373DC8    52              PUSH    EDX  
00373DC9    56              PUSH    ESI  
00373DCA    8D85 FA264000   LEA     EAX, DWORD PTR SS:\[EBP+4026FA\]  
00373DD0    50              PUSH    EAX  
00373DD1    8B85 9E434000   MOV     EAX, DWORD PTR SS:\[EBP+<MyGetProcAddress>\]  
00373DD7    E9 FE0B0000     JMP     <proc\_Run\_FUN>  
00373DDC    90              NOP  
00373DDD    90              NOP  
00373DDE    E8 5B0E0000     CALL    <proc\_check\_CC>  
00373DE3    8907            MOV     DWORD PTR DS:\[EDI\], EAX                                     ; 填充 IAT  
00373DE5    5A              POP     EDX                                                         ; 0012FFE0  
00373DE6    0FB642 FF       MOVZX   EAX, BYTE PTR DS:\[EDX\-1\]  
00373DEA    03D0            ADD     EDX, EAX  
00373DEC    42              INC     EDX  
00373DED    83C7 04         ADD     EDI, 4  
00373DF0    59              POP     ECX                                                         ; 0012FFE0  
00373DF1  ^ E2 83           LOOPD   SHORT <loop\_not\_relocs\_api>  
00373DF3    E9 C6040000     JMP     <jmp\_loop\_de\_iat>  
00373DF8 >  8B0A            MOV     ECX, DWORD PTR DS:\[EDX\]                                     ; Reloc\_FUN  
00373DFA    81E1 FFFFFF7F   AND     ECX, 7FFFFFFF  
00373E00    51              PUSH    ECX  
00373E01    52              PUSH    EDX  
00373E02    C1E1 05         SHL     ECX, 5  
00373E05    6A 04           PUSH    4  
00373E07    68 00100000     PUSH    1000  
00373E0C    51              PUSH    ECX  
00373E0D    6A 00           PUSH    0  
00373E0F    8D85 3E274000   LEA     EAX, DWORD PTR SS:\[EBP+40273E\]  
00373E15    50              PUSH    EAX  
00373E16    8B85 AE434000   MOV     EAX, DWORD PTR SS:\[EBP+<APIVirtualAlloc>\]                 ; kernel32.VirtualAlloc  
00373E1C    E9 B90B0000     JMP     <proc\_Run\_FUN>  
00373E21    90              NOP  
00373E22    8985 FE434000   MOV     DWORD PTR SS:\[EBP+<hMEM\_IAT\_RELOC\_1>\], EAX  
00373E28    5A              POP     EDX                                                         ; 0012FFE0  
00373E29    59              POP     ECX                                                         ; 0012FFE0  
00373E2A    50              PUSH    EAX  
00373E2B    51              PUSH    ECX  
00373E2C    2BBD B2434000   SUB     EDI, DWORD PTR SS:\[EBP+<IMGBASE>\]                         ; 00400000  
00373E32    83FF FF         CMP     EDI, -1  
00373E35    74 15           JE      SHORT 00373E4C  
00373E37    03BD B2434000   ADD     EDI, DWORD PTR SS:\[EBP+<IMGBASE>\]                         ; 00400000  
00373E3D    EB 09           JMP     SHORT 00373E48  
00373E3F    8907            MOV     DWORD PTR DS:\[EDI\], EAX                                     ; 循环填充第一层加密地址  
00373E41    83C0 20         ADD     EAX, 20                                                     ; 每次+20  
00373E44    83C7 04         ADD     EDI, 4  
00373E47    49              DEC     ECX  
00373E48    0BC9            OR      ECX, ECX  
00373E4A  ^ 75 F3           JNZ     SHORT 00373E3F  
00373E4C    59              POP     ECX                                                         ; 0012FFE0  
00373E4D    58              POP     EAX                                                         ; 0012FFE0  
00373E4E    8BF8            MOV     EDI, EAX  
00373E50    57              PUSH    EDI  
00373E51    51              PUSH    ECX  
00373E52    EB 2D           JMP     SHORT 00373E81  
00373E54 >  8D47 1C         LEA     EAX, DWORD PTR DS:\[EDI+1C\]                                  ; Fill\_1\_address  
00373E57    66:C707 FF35    MOV     WORD PTR DS:\[EDI\], 35FF                                     ; 填充为 push \[addr\]  
00373E5C    C747 06 8134240>MOV     DWORD PTR DS:\[EDI+6\], 243481                                ; xor \[esp\],rndkey  
00373E63    8947 02         MOV     DWORD PTR DS:\[EDI+2\], EAX                                   ; ret  
00373E66    C647 0D C3      MOV     BYTE PTR DS:\[EDI+D\], 0C3  
00373E6A    52              PUSH    EDX  
00373E6B    0F31            RDTSC  
00373E6D    32E0            XOR     AH, AL  
00373E6F    C1C8 08         ROR     EAX, 8  
00373E72    02E0            ADD     AH, AL  
00373E74    C1C8 08         ROR     EAX, 8  
00373E77    32E0            XOR     AH, AL  
00373E79    8947 09         MOV     DWORD PTR DS:\[EDI+9\], EAX  
00373E7C    5A              POP     EDX                                                         ; 0012FFE0  
00373E7D    83C7 20         ADD     EDI, 20  
00373E80    49              DEC     ECX  
00373E81    0BC9            OR      ECX, ECX  
00373E83  ^ 75 CF           JNZ     SHORT <Fill\_1\_address>  
00373E85    59              POP     ECX                                                         ; 0012FFE0  
00373E86    5F              POP     EDI                                                         ; 0012FFE0  
00373E87    83C2 04         ADD     EDX, 4  
00373E8A >  51              PUSH    ECX                                                         ; loop\_Current\_DLL  
00373E8B    0FB602          MOVZX   EAX, BYTE PTR DS:\[EDX\]  
00373E8E    0BC0            OR      EAX, EAX  
00373E90    0F85 85000000   JNZ     <By\_Name>                                                   ; 判断是名称方式还是序号方式  
00373E96    42              INC     EDX                                                         ; API 是序号方式则这里开始处理  
00373E97    52              PUSH    EDX  
00373E98    8B02            MOV     EAX, DWORD PTR DS:\[EDX\]  
00373E9A    50              PUSH    EAX  
00373E9B    56              PUSH    ESI  
00373E9C    8D85 CB274000   LEA     EAX, DWORD PTR SS:\[EBP+4027CB\]  
00373EA2    50              PUSH    EAX  
00373EA3    8B85 9E434000   MOV     EAX, DWORD PTR SS:\[EBP+<MyGetProcAddress>\]  
00373EA9    E9 2C0B0000     JMP     <proc\_Run\_FUN>  
00373EAE    90              NOP  
00373EAF    8B9D E9494000   MOV     EBX, DWORD PTR SS:\[EBP+<hvMEM>\]  
00373EB5    039D ED494000   ADD     EBX, DWORD PTR SS:\[EBP+<shellFunAddress>\]  
00373EBB    53              PUSH    EBX  
00373EBC    50              PUSH    EAX  
00373EBD    53              PUSH    EBX  
00373EBE    E8 2C0B0000     CALL    <steal code>  
00373EC3    2B85 E9494000   SUB     EAX, DWORD PTR SS:\[EBP+<hvMEM>\]  
00373EC9    8985 ED494000   MOV     DWORD PTR SS:\[EBP+<shellFunAddress>\], EAX  
00373ECF    60              PUSHAD  
00373ED0    3D C01F0000     CMP     EAX, 1FC0  
00373ED5    76 31           JBE     SHORT 00373F08                                              ; 判断是否够空间  
00373ED7    6A 04           PUSH    4                                                           ; 存放空间不够则再申请空间  
00373ED9    68 00100000     PUSH    1000  
00373EDE    68 00200000     PUSH    2000  
00373EE3    6A 00           PUSH    0  
00373EE5    8D85 14284000   LEA     EAX, DWORD PTR SS:\[EBP+402814\]  
00373EEB    50              PUSH    EAX  
00373EEC    8B85 AE434000   MOV     EAX, DWORD PTR SS:\[EBP+<APIVirtualAlloc>\]                 ; kernel32.VirtualAlloc  
00373EF2    E9 E30A0000     JMP     <proc\_Run\_FUN>  
00373EF7    90              NOP  
00373EF8    8985 E9494000   MOV     DWORD PTR SS:\[EBP+<hvMEM>\], EAX  
00373EFE    C785 ED494000 0>MOV     DWORD PTR SS:\[EBP+<shellFunAddress>\], 0  
00373F08    61              POPAD  
00373F09    5B              POP     EBX                                                         ; 0012FFE0  
00373F0A    8BC3            MOV     EAX, EBX  
00373F0C    3347 09         XOR     EAX, DWORD PTR DS:\[EDI+9\]  
00373F0F    8947 1C         MOV     DWORD PTR DS:\[EDI+1C\], EAX  
00373F12    5A              POP     EDX                                                         ; 0012FFE0  
00373F13    83C2 04         ADD     EDX, 4  
00373F16    E9 97030000     JMP     003742B2  
00373F1B >  42              INC     EDX                                                         ; By\_Name  
00373F1C    52              PUSH    EDX  
00373F1D >  60              PUSHAD                                                              ; Decrypt\_API\_name  
00373F1E    8BF2            MOV     ESI, EDX  
00373F20    8DBD 74484000   LEA     EDI, DWORD PTR SS:\[EBP+<strAPIName>\]  
00373F26    33C0            XOR     EAX, EAX  
00373F28    0FB64E FF       MOVZX   ECX, BYTE PTR DS:\[ESI\-1\]  
00373F2C    EB 0E           JMP     SHORT 00373F3C  
00373F2E    AC              LODS    BYTE PTR DS:\[ESI\]  
00373F2F    34 79           XOR     AL, 79  
00373F31    2C 55           SUB     AL, 55  
00373F33    C0C0 03         ROL     AL, 3  
00373F36    F6D0            NOT     AL  
00373F38    AA              STOS    BYTE PTR ES:\[EDI\]  
00373F39    49              DEC     ECX  
00373F3A    33C0            XOR     EAX, EAX  
00373F3C    0BC9            OR      ECX, ECX  
00373F3E  ^ 75 EE           JNZ     SHORT 00373F2E  
00373F40    AA              STOS    BYTE PTR ES:\[EDI\]  
00373F41    61              POPAD  
00373F42    8D95 74484000   LEA     EDX, DWORD PTR SS:\[EBP+<strAPIName>\]  
00373F48    52              PUSH    EDX  
00373F49    52              PUSH    EDX  
00373F4A    8D85 C9464000   LEA     EAX, DWORD PTR SS:\[EBP+<strLoadLib>\]  
00373F50    50              PUSH    EAX  
00373F51    8D85 80284000   LEA     EAX, DWORD PTR SS:\[EBP+402880\]  
00373F57    50              PUSH    EAX  
00373F58    8B85 E3444000   MOV     EAX, DWORD PTR SS:\[EBP+<APIlstrcmpA>\]                     ; kernel32.lstrcmpA  
00373F5E    E9 770A0000     JMP     <proc\_Run\_FUN>                                              ; 判断是否为特殊处理的 API  
00373F63    90              NOP  
00373F64    5A              POP     EDX                                                         ; 0012FFE0  
00373F65    85C0            TEST    EAX, EAX  
00373F67    75 0B           JNZ     SHORT 00373F74  
00373F69    8D85 89394000   LEA     EAX, DWORD PTR SS:\[EBP+<SDKLoadLib>\]  
00373F6F    E9 31030000     JMP     <Fill\_IAT\_RELOC\_2>  
00373F74    52              PUSH    EDX  
00373F75    52              PUSH    EDX  
00373F76    8D85 BA464000   LEA     EAX, DWORD PTR SS:\[EBP+<strGetProcaddress>\]  
00373F7C    50              PUSH    EAX  
00373F7D    8D85 AC284000   LEA     EAX, DWORD PTR SS:\[EBP+4028AC\]  
00373F83    50              PUSH    EAX  
00373F84    8B85 E3444000   MOV     EAX, DWORD PTR SS:\[EBP+<APIlstrcmpA>\]                     ; kernel32.lstrcmpA  
00373F8A    E9 4B0A0000     JMP     <proc\_Run\_FUN>  
00373F8F    90              NOP  
00373F90    5A              POP     EDX                                                         ; 0012FFE0  
00373F91    85C0            TEST    EAX, EAX  
00373F93    75 0B           JNZ     SHORT 00373FA0  
00373F95    8D85 9A394000   LEA     EAX, DWORD PTR SS:\[EBP+<SDKGetProcAddr>\]  
00373F9B    E9 05030000     JMP     <Fill\_IAT\_RELOC\_2>  
00373FA0    52              PUSH    EDX  
00373FA1    52              PUSH    EDX  
00373FA2    8D85 D6464000   LEA     EAX, DWORD PTR SS:\[EBP+<strGetVersion>\]  
00373FA8    50              PUSH    EAX  
00373FA9    8D85 D8284000   LEA     EAX, DWORD PTR SS:\[EBP+4028D8\]  
00373FAF    50              PUSH    EAX  
00373FB0    8B85 E3444000   MOV     EAX, DWORD PTR SS:\[EBP+<APIlstrcmpA>\]                     ; kernel32.lstrcmpA  
00373FB6    E9 1F0A0000     JMP     <proc\_Run\_FUN>    
00373FBB    90              NOP  
00373FBC    5A              POP     EDX                                                         ; 0012FFE0  
00373FBD    85C0            TEST    EAX, EAX  
00373FBF    75 0B           JNZ     SHORT 00373FCC  
00373FC1    8D85 AF394000   LEA     EAX, DWORD PTR SS:\[EBP+<SDKGetVersion>\]  
00373FC7    E9 D9020000     JMP     <Fill\_IAT\_RELOC\_2>  
00373FCC    52              PUSH    EDX  
00373FCD    52              PUSH    EDX  
00373FCE    8D85 E1464000   LEA     EAX, DWORD PTR SS:\[EBP+<strGetModlehnd>\]  
00373FD4    50              PUSH    EAX  
00373FD5    8D85 04294000   LEA     EAX, DWORD PTR SS:\[EBP+402904\]  
00373FDB    50              PUSH    EAX  
00373FDC    8B85 E3444000   MOV     EAX, DWORD PTR SS:\[EBP+<APIlstrcmpA>\]                     ; kernel32.lstrcmpA  
00373FE2    E9 F3090000     JMP     <proc\_Run\_FUN>  
00373FE7    90              NOP  
00373FE8    5A              POP     EDX                                                         ; 0012FFE0  
00373FE9    85C0            TEST    EAX, EAX  
00373FEB    75 0B           JNZ     SHORT 00373FF8  
00373FED    8D85 E4394000   LEA     EAX, DWORD PTR SS:\[EBP+<SDKGetModlehnd>\]  
00373FF3    E9 AD020000     JMP     <Fill\_IAT\_RELOC\_2>  
00373FF8    52              PUSH    EDX  
00373FF9    52              PUSH    EDX  
00373FFA    8D85 F2464000   LEA     EAX, DWORD PTR SS:\[EBP+<strGetCurrProcess>\]  
00374000    50              PUSH    EAX  
00374001    8D85 30294000   LEA     EAX, DWORD PTR SS:\[EBP+402930\]  
00374007    50              PUSH    EAX  
00374008    8B85 E3444000   MOV     EAX, DWORD PTR SS:\[EBP+<APIlstrcmpA>\]                     ; kernel32.lstrcmpA  
0037400E    E9 C7090000     JMP     <proc\_Run\_FUN>  
00374013    90              NOP  
00374014    5A              POP     EDX                                                         ; 0012FFE0  
00374015    85C0            TEST    EAX, EAX  
00374017    75 0B           JNZ     SHORT 00374024  
00374019    8D85 F5394000   LEA     EAX, DWORD PTR SS:\[EBP+<SDKGetCurProcess>\]  
0037401F    E9 81020000     JMP     <Fill\_IAT\_RELOC\_2>  
00374024    52              PUSH    EDX  
00374025    52              PUSH    EDX  
00374026    8D85 04474000   LEA     EAX, DWORD PTR SS:\[EBP+<strGetCurprocID>\]  
0037402C    50              PUSH    EAX  
0037402D    8D85 5C294000   LEA     EAX, DWORD PTR SS:\[EBP+40295C\]  
00374033    50              PUSH    EAX  
00374034    8B85 E3444000   MOV     EAX, DWORD PTR SS:\[EBP+<APIlstrcmpA>\]                     ; kernel32.lstrcmpA  
0037403A    E9 9B090000     JMP     <proc\_Run\_FUN>  
0037403F    90              NOP  
00374040    5A              POP     EDX                                                         ; 0012FFE0  
00374041    85C0            TEST    EAX, EAX  
00374043    75 0B           JNZ     SHORT 00374050  
00374045    8D85 323A4000   LEA     EAX, DWORD PTR SS:\[EBP+<SDKGetcurProcID>\]  
0037404B    E9 55020000     JMP     <Fill\_IAT\_RELOC\_2>  
00374050    52              PUSH    EDX  
00374051    52              PUSH    EDX  
00374052    8D85 18474000   LEA     EAX, DWORD PTR SS:\[EBP+<strGetcmdline>\]  
00374058    50              PUSH    EAX  
00374059    8D85 88294000   LEA     EAX, DWORD PTR SS:\[EBP+402988\]  
0037405F    50              PUSH    EAX  
00374060    8B85 E3444000   MOV     EAX, DWORD PTR SS:\[EBP+<APIlstrcmpA>\]                     ; kernel32.lstrcmpA  
00374066    E9 6F090000     JMP     <proc\_Run\_FUN>  
0037406B    90              NOP  
0037406C    5A              POP     EDX                                                         ; 0012FFE0  
0037406D    85C0            TEST    EAX, EAX  
0037406F    75 0B           JNZ     SHORT 0037407C  
00374071    8D85 5F3A4000   LEA     EAX, DWORD PTR SS:\[EBP+<SDKGetCMDline>\]  
00374077    E9 29020000     JMP     <Fill\_IAT\_RELOC\_2>  
0037407C    52              PUSH    EDX  
0037407D    52              PUSH    EDX  
0037407E    8D85 41474000   LEA     EAX, DWORD PTR SS:\[EBP+<strLockRes>\]  
00374084    50              PUSH    EAX  
00374085    8D85 B4294000   LEA     EAX, DWORD PTR SS:\[EBP+4029B4\]  
0037408B    50              PUSH    EAX  
0037408C    8B85 E3444000   MOV     EAX, DWORD PTR SS:\[EBP+<APIlstrcmpA>\]                     ; kernel32.lstrcmpA  
00374092    E9 43090000     JMP     <proc\_Run\_FUN>  
00374097    90              NOP  
00374098    5A              POP     EDX                                                         ; 0012FFE0  
00374099    85C0            TEST    EAX, EAX  
0037409B    75 0B           JNZ     SHORT 003740A8  
0037409D    8D85 023B4000   LEA     EAX, DWORD PTR SS:\[EBP+<SDKLockRes>\]  
003740A3    E9 FD010000     JMP     <Fill\_IAT\_RELOC\_2>  
003740A8    52              PUSH    EDX  
003740A9    52              PUSH    EDX  
003740AA    8D85 4E474000   LEA     EAX, DWORD PTR SS:\[EBP+<strFreeRes>\]  
003740B0    50              PUSH    EAX  
003740B1    8D85 E0294000   LEA     EAX, DWORD PTR SS:\[EBP+4029E0\]  
003740B7    50              PUSH    EAX  
003740B8    8B85 E3444000   MOV     EAX, DWORD PTR SS:\[EBP+<APIlstrcmpA>\]                     ; kernel32.lstrcmpA  
003740BE    E9 17090000     JMP     <proc\_Run\_FUN>  
003740C3    90              NOP  
003740C4    5A              POP     EDX                                                         ; 0012FFE0  
003740C5    85C0            TEST    EAX, EAX  
003740C7    75 0B           JNZ     SHORT 003740D4  
003740C9    8D85 023B4000   LEA     EAX, DWORD PTR SS:\[EBP+<SDKLockRes>\]  
003740CF    E9 D1010000     JMP     <Fill\_IAT\_RELOC\_2>  
003740D4    52              PUSH    EDX  
003740D5    52              PUSH    EDX  
003740D6    8D85 28474000   LEA     EAX, DWORD PTR SS:\[EBP+<strExitProc>\]  
003740DC    50              PUSH    EAX  
003740DD    8D85 0C2A4000   LEA     EAX, DWORD PTR SS:\[EBP+402A0C\]  
003740E3    50              PUSH    EAX  
003740E4    8B85 E3444000   MOV     EAX, DWORD PTR SS:\[EBP+<APIlstrcmpA>\]                     ; kernel32.lstrcmpA  
003740EA    E9 EB080000     JMP     <proc\_Run\_FUN>  
003740EF    90              NOP  
003740F0    5A              POP     EDX                                                         ; 0012FFE0  
003740F1    85C0            TEST    EAX, EAX  
003740F3    75 0B           JNZ     SHORT 00374100  
003740F5    8D85 7C3A4000   LEA     EAX, DWORD PTR SS:\[EBP+<SDKExitProc>\]  
003740FB    E9 A5010000     JMP     <Fill\_IAT\_RELOC\_2>  
00374100    52              PUSH    EDX  
00374101    52              PUSH    EDX  
00374102    8D85 5B474000   LEA     EAX, DWORD PTR SS:\[EBP+<strDLGBoxParamA>\]  
00374108    50              PUSH    EAX  
00374109    8D85 852A4000   LEA     EAX, DWORD PTR SS:\[EBP+402A85\]  
0037410F    50              PUSH    EAX  
00374110    8B85 E3444000   MOV     EAX, DWORD PTR SS:\[EBP+<APIlstrcmpA>\]                     ; kernel32.lstrcmpA  
00374116    E9 BF080000     JMP     <proc\_Run\_FUN>  
0037411B    90              NOP  
0037411C    8BC5            MOV     EAX, EBP  
0037411E    8DB5 014A4000   LEA     ESI, DWORD PTR SS:\[EBP+404A01\]                              ; 过完全部的异常后把最后异常的那些地址再加密回去  
00374124    2946 04         SUB     DWORD PTR DS:\[ESI+4\], EAX  
00374127    2946 08         SUB     DWORD PTR DS:\[ESI+8\], EAX  
0037412A    83C6 20         ADD     ESI, 20  
0037412D    2946 04         SUB     DWORD PTR DS:\[ESI+4\], EAX  
00374130    83C6 20         ADD     ESI, 20  
00374133    2946 04         SUB     DWORD PTR DS:\[ESI+4\], EAX  
00374136    2946 08         SUB     DWORD PTR DS:\[ESI+8\], EAX  
00374139    83C6 20         ADD     ESI, 20  
0037413C    2946 04         SUB     DWORD PTR DS:\[ESI+4\], EAX  
0037413F    83C6 20         ADD     ESI, 20  
00374142    2946 04         SUB     DWORD PTR DS:\[ESI+4\], EAX  
00374145    83C6 20         ADD     ESI, 20  
00374148    2946 04         SUB     DWORD PTR DS:\[ESI+4\], EAX  
0037414B    83C6 20         ADD     ESI, 20  
0037414E    2946 04         SUB     DWORD PTR DS:\[ESI+4\], EAX  
00374151    83C6 20         ADD     ESI, 20  
00374154    2946 04         SUB     DWORD PTR DS:\[ESI+4\], EAX  
00374157    8DB5 FD494000   LEA     ESI, DWORD PTR SS:\[EBP+4049FD\]  
0037415D    B8 014A4000     MOV     EAX, 404A01  
00374162    8906            MOV     DWORD PTR DS:\[ESI\], EAX  
00374164  ^ E9 F5F8FFFF     JMP     00373A5E                                                    ; 跳去处理 OEP 的代码  
00374169    5A              POP     EDX                                                         ; 0012FFE0  
0037416A    85C0            TEST    EAX, EAX  
0037416C    75 0B           JNZ     SHORT 00374179                                              ; 如果不是特殊函数则跳  
0037416E    8D85 8B3A4000   LEA     EAX, DWORD PTR SS:\[EBP+<SDKDLGBoxParamA>\]  
00374174    E9 2C010000     JMP     <Fill\_IAT\_RELOC\_2>  
00374179    52              PUSH    EDX  
0037417A    52              PUSH    EDX  
0037417B    8D85 6B474000   LEA     EAX, DWORD PTR SS:\[EBP+<strCreateDLGParamA>\]  
00374181    50              PUSH    EAX  
00374182    8D85 B12A4000   LEA     EAX, DWORD PTR SS:\[EBP+402AB1\]  
00374188    50              PUSH    EAX  
00374189    8B85 E3444000   MOV     EAX, DWORD PTR SS:\[EBP+<APIlstrcmpA>\]                     ; kernel32.lstrcmpA  
0037418F    E9 46080000     JMP     <proc\_Run\_FUN>  
00374194    90              NOP  
00374195    5A              POP     EDX                                                         ; 0012FFE0  
00374196    85C0            TEST    EAX, EAX  
00374198    75 0B           JNZ     SHORT 003741A5  
0037419A    8D85 C83A4000   LEA     EAX, DWORD PTR SS:\[EBP+<SDKCreateDLGParamA>\]  
003741A0    E9 00010000     JMP     <Fill\_IAT\_RELOC\_2>  
003741A5    52              PUSH    EDX  
003741A6    52              PUSH    EDX  
003741A7    8D85 34474000   LEA     EAX, DWORD PTR SS:\[EBP+<strSndMsg>\]  
003741AD    50              PUSH    EAX  
003741AE    8D85 DD2A4000   LEA     EAX, DWORD PTR SS:\[EBP+402ADD\]  
003741B4    50              PUSH    EAX  
003741B5    8B85 E3444000   MOV     EAX, DWORD PTR SS:\[EBP+<APIlstrcmpA>\]                     ; kernel32.lstrcmpA  
003741BB    E9 1A080000     JMP     <proc\_Run\_FUN>  
003741C0    90              NOP  
003741C1    5A              POP     EDX                                                         ; 0012FFE0  
003741C2    85C0            TEST    EAX, EAX  
003741C4    75 0B           JNZ     SHORT 003741D1  
003741C6    8D85 2E3B4000   LEA     EAX, DWORD PTR SS:\[EBP+<SDKSndMsg>\]  
003741CC    E9 D4000000     JMP     <Fill\_IAT\_RELOC\_2>  
003741D1    52              PUSH    EDX  
003741D2    52              PUSH    EDX  
003741D3    8D85 7E474000   LEA     EAX, DWORD PTR SS:\[EBP+<strsend>\]  
003741D9    50              PUSH    EAX  
003741DA    8D85 092B4000   LEA     EAX, DWORD PTR SS:\[EBP+402B09\]  
003741E0    50              PUSH    EAX  
003741E1    8B85 E3444000   MOV     EAX, DWORD PTR SS:\[EBP+<APIlstrcmpA>\]                     ; kernel32.lstrcmpA  
003741E7    E9 EE070000     JMP     <proc\_Run\_FUN>  
003741EC    90              NOP  
003741ED    5A              POP     EDX                                                         ; 0012FFE0  
003741EE    85C0            TEST    EAX, EAX  
003741F0    75 0B           JNZ     SHORT 003741FD  
003741F2    8D85 323B4000   LEA     EAX, DWORD PTR SS:\[EBP+<SDKsend>\]  
003741F8    E9 A8000000     JMP     <Fill\_IAT\_RELOC\_2>  
003741FD    52              PUSH    EDX  
003741FE    52              PUSH    EDX  
003741FF    8D85 83474000   LEA     EAX, DWORD PTR SS:\[EBP+<strrecv>\]  
00374205    50              PUSH    EAX  
00374206    8D85 352B4000   LEA     EAX, DWORD PTR SS:\[EBP+402B35\]  
0037420C    50              PUSH    EAX  
0037420D    8B85 E3444000   MOV     EAX, DWORD PTR SS:\[EBP+<APIlstrcmpA>\]                     ; kernel32.lstrcmpA  
00374213    E9 C2070000     JMP     <proc\_Run\_FUN>  
00374218    90              NOP  
00374219    5A              POP     EDX                                                         ; 0012FFE0  
0037421A    85C0            TEST    EAX, EAX  
0037421C    75 08           JNZ     SHORT 00374226  
0037421E    8D85 363B4000   LEA     EAX, DWORD PTR SS:\[EBP+<SDKrecv>\]  
00374224    EB 7F           JMP     SHORT <Fill\_IAT\_RELOC\_2>  
00374226    52              PUSH    EDX  
00374227    56              PUSH    ESI  
00374228    8D85 572B4000   LEA     EAX, DWORD PTR SS:\[EBP+402B57\]  
0037422E    50              PUSH    EAX  
0037422F    8B85 9E434000   MOV     EAX, DWORD PTR SS:\[EBP+<MyGetProcAddress>\]                ; GetProcAddress 获取 API 的地址  
00374235    E9 A0070000     JMP     <proc\_Run\_FUN>  
0037423A    90              NOP  
0037423B    8B9D E9494000   MOV     EBX, DWORD PTR SS:\[EBP+<hvMEM>\]  
00374241    039D ED494000   ADD     EBX, DWORD PTR SS:\[EBP+<shellFunAddress>\]  
00374247    53              PUSH    EBX  
00374248    50              PUSH    EAX  
00374249    53              PUSH    EBX  
0037424A    E8 A0070000     CALL    <steal code>  
0037424F    2B85 E9494000   SUB     EAX, DWORD PTR SS:\[EBP+<hvMEM>\]  
00374255    8985 ED494000   MOV     DWORD PTR SS:\[EBP+<shellFunAddress>\], EAX  
0037425B    60              PUSHAD  
0037425C    3D C01F0000     CMP     EAX, 1FC0                                                   ; 判断是否够空间  
00374261    76 3E           JBE     SHORT 003742A1                                              ; 如果空间够用则跳  
00374263    6A 04           PUSH    4  
00374265    68 00100000     PUSH    1000  
0037426A    68 00200000     PUSH    2000  
0037426F    6A 00           PUSH    0  
00374271    8D85 AD2B4000   LEA     EAX, DWORD PTR SS:\[EBP+402BAD\]  
00374277    50              PUSH    EAX                                                         ; 空间不够用则跳  
00374278    8B85 AE434000   MOV     EAX, DWORD PTR SS:\[EBP+<APIVirtualAlloc>\]                 ; kernel32.VirtualAlloc  
0037427E    E9 57070000     JMP     <proc\_Run\_FUN>  
......  
00374284    64:8F05 0000000>POP     DWORD PTR FS:\[0\]                                            ; 最后一个恢复 SEH 到这里  
0037428B    58              POP     EAX                                                         ; 0012FFE0  
0037428C  ^ E9 8BFEFFFF     JMP     0037411C  
00374291    8985 E9494000   MOV     DWORD PTR SS:\[EBP+<hvMEM>\], EAX  
00374297    C785 ED494000 0>MOV     DWORD PTR SS:\[EBP+<shellFunAddress>\], 0  
003742A1    61              POPAD  
003742A2    5B              POP     EBX                                                         ; 0012FFE0  
003742A3    8BC3            MOV     EAX, EBX  
003742A5 >  3347 09         XOR     EAX, DWORD PTR DS:\[EDI+9\]                                   ; Fill\_IAT\_RELOC\_2  
003742A8    8947 1C         MOV     DWORD PTR DS:\[EDI+1C\], EAX                                  ; 填充地址  
003742AB    5A              POP     EDX                                                         ; 0012FFE0  
003742AC    0FB642 FF       MOVZX   EAX, BYTE PTR DS:\[EDX\-1\]  
003742B0    03D0            ADD     EDX, EAX  
003742B2    42              INC     EDX  
003742B3    83C7 20         ADD     EDI, 20  
003742B6    59              POP     ECX                                                         ; 0012FFE0  
003742B7    49              DEC     ECX  
003742B8  ^ 0F85 CCFBFFFF   JNZ     <loop\_Current\_DLL>  
003742BE >^ E9 46FAFFFF     JMP     <loop\_De\_Crypted\_iat>                                       ; jmp\_loop\_de\_iat  
003742C3 >  B9 00010000     MOV     ECX, 100                                                    ; Disposal\_IAT\_Done  
  
看起来都有够复杂了,还好脱的时候不会这么复杂  
  
;⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕END⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕  
  
  
  
  
;♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀文件 CRC 检测♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀  
  
  
  
003742C8    2BE1            SUB     ESP, ECX  
003742CA    8BF4            MOV     ESI, ESP  
003742CC    8BFC            MOV     EDI, ESP  
003742CE    C1E9 02         SHR     ECX, 2  
003742D1    33C0            XOR     EAX, EAX  
003742D3    F3:AB           REP     STOS DWORD PTR ES:\[EDI\]  
003742D5    68 00010000     PUSH    100  
003742DA    56              PUSH    ESI  
003742DB    8B85 B2434000   MOV     EAX, DWORD PTR SS:\[EBP+<IMGBASE>\]                         ; 00400000  
003742E1    50              PUSH    EAX  
003742E2    8D85 112C4000   LEA     EAX, DWORD PTR SS:\[EBP+402C11\]  
003742E8    50              PUSH    EAX  
003742E9    8B85 0B454000   MOV     EAX, DWORD PTR SS:\[EBP+<APIGetModuleFileName>\]            ; kernel32.GetModuleFileNameA  
003742EF    E9 E6060000     JMP     <proc\_Run\_FUN>                                              ; 获取模块名  
003742F4    90              NOP  
003742F5    6A 00           PUSH    0  
003742F7    68 80000000     PUSH    80  
003742FC    6A 03           PUSH    3  
003742FE    6A 00           PUSH    0  
00374300    6A 03           PUSH    3  
00374302    68 00000080     PUSH    80000000  
00374307    56              PUSH    ESI  
00374308    8D85 3F2C4000   LEA     EAX, DWORD PTR SS:\[EBP+402C3F\]  
0037430E    50              PUSH    EAX  
0037430F    8B85 BC444000   MOV     EAX, DWORD PTR SS:\[EBP+<APICreateFileA>\]                  ; kernel32.CreateFileA  
00374315    E9 C0060000     JMP     <proc\_Run\_FUN>  
0037431A    90              NOP  
0037431B    6285 F1494000   BOUND   EAX, QWORD PTR SS:\[EBP+4049F1\]                              ; 最后一次异常跳这里  
00374321  ^ EB F8           JMP     SHORT 0037431B  
00374323    8BD8            MOV     EBX, EAX  
00374325    81C4 00010000   ADD     ESP, 100  
0037432B    6A 00           PUSH    0  
0037432D    53              PUSH    EBX  
0037432E    8D85 5D2C4000   LEA     EAX, DWORD PTR SS:\[EBP+402C5D\]  
00374334    50              PUSH    EAX  
00374335    8B85 C9444000   MOV     EAX, DWORD PTR SS:\[EBP+<APIGetFileSize>\]                  ; kernel32.GetFileSize  
0037433B    E9 9A060000     JMP     <proc\_Run\_FUN>  
00374340    90              NOP  
00374341    8985 B6434000   MOV     DWORD PTR SS:\[EBP+<\_dwFileSize>\], EAX  
00374347    6A 00           PUSH    0  
00374349    FFB5 B6434000   PUSH    DWORD PTR SS:\[EBP+<\_dwFileSize>\]  
0037434F    6A 00           PUSH    0  
00374351    6A 02           PUSH    2  
00374353    6A 00           PUSH    0  
00374355    53              PUSH    EBX  
00374356    8D85 852C4000   LEA     EAX, DWORD PTR SS:\[EBP+402C85\]  
0037435C    50              PUSH    EAX  
0037435D    8B85 75454000   MOV     EAX, DWORD PTR SS:\[EBP+<CreateFileMapA>\]                  ; kernel32.CreateFileMappingA  
00374363    E9 72060000     JMP     <proc\_Run\_FUN>  
00374368    90              NOP  
00374369    8985 BA434000   MOV     DWORD PTR SS:\[EBP+<hMap>\], EAX  
0037436F    6A 00           PUSH    0  
00374371    6A 00           PUSH    0  
00374373    6A 00           PUSH    0  
00374375    6A 04           PUSH    4  
00374377    FFB5 BA434000   PUSH    DWORD PTR SS:\[EBP+<hMap>\]  
0037437D    8D85 B32C4000   LEA     EAX, DWORD PTR SS:\[EBP+402CB3\]  
00374383    50              PUSH    EAX  
00374384    8B85 89454000   MOV     EAX, DWORD PTR SS:\[EBP+<APIMapViewofFile>\]                ; kernel32.MapViewOfFile  
0037438A    E9 4B060000     JMP     <proc\_Run\_FUN>  
0037438F    90              NOP  
00374390    90              NOP  
00374391    40              INC     EAX  
00374392    D1C8            ROR     EAX, 1  
00374394    CE              INTO  
00374395  ^ EB FA           JMP     SHORT 00374391  
00374397    8985 BE434000   MOV     DWORD PTR SS:\[EBP+<hvmapmem>\], EAX  
0037439D    53              PUSH    EBX  
0037439E    8B40 3C         MOV     EAX, DWORD PTR DS:\[EAX+3C\]  
003743A1    8B8D B6434000   MOV     ECX, DWORD PTR SS:\[EBP+<\_dwFileSize>\]  
003743A7    2BC8            SUB     ECX, EAX  
003743A9    8BB5 BE434000   MOV     ESI, DWORD PTR SS:\[EBP+<hvmapmem>\]  
003743AF    03F0            ADD     ESI, EAX  
003743B1    E8 A5080000     CALL    <Calculate\_CRC>                                             ; 计算 CRC 值  
003743B6    5B              POP     EBX                                                         ; 0012FFE0  
003743B7    3385 C6434000   XOR     EAX, DWORD PTR SS:\[EBP+<xorsizeimg\_Key>\]  
003743BD    C1C8 03         ROR     EAX, 3  
003743C0    8BF0            MOV     ESI, EAX  
003743C2    8B85 BE434000   MOV     EAX, DWORD PTR SS:\[EBP+<hvmapmem>\]  
003743C8    0340 3C         ADD     EAX, DWORD PTR DS:\[EAX+3C\]  
003743CB    8B78 FC         MOV     EDI, DWORD PTR DS:\[EAX\-4\]                                   ; 取出文件的 CRC 值  
003743CE    FFB5 BE434000   PUSH    DWORD PTR SS:\[EBP+<hvmapmem>\]  
003743D4    8D85 032D4000   LEA     EAX, DWORD PTR SS:\[EBP+402D03\]  
003743DA    50              PUSH    EAX  
003743DB    8B85 98454000   MOV     EAX, DWORD PTR SS:\[EBP+<APIUnmapviewofFile>\]              ; kernel32.UnmapViewOfFile  
003743E1    E9 F4050000     JMP     <proc\_Run\_FUN>  
003743E6    90              NOP  
003743E7    FFB5 BA434000   PUSH    DWORD PTR SS:\[EBP+<hMap>\]  
003743ED    8D85 1C2D4000   LEA     EAX, DWORD PTR SS:\[EBP+402D1C\]  
003743F3    50              PUSH    EAX  
003743F4    8B85 A9454000   MOV     EAX, DWORD PTR SS:\[EBP+<APICloaseHandel>\]                 ; kernel32.CloseHandle  
003743FA    E9 DB050000     JMP     <proc\_Run\_FUN>  
003743FF    90              NOP  
00374400    53              PUSH    EBX  
00374401    8D85 302D4000   LEA     EAX, DWORD PTR SS:\[EBP+402D30\]  
00374407    50              PUSH    EAX  
00374408    8B85 A9454000   MOV     EAX, DWORD PTR SS:\[EBP+<APICloaseHandel>\]                 ; kernel32.CloseHandle  
0037440E    E9 C7050000     JMP     <proc\_Run\_FUN>  
00374413    90              NOP  
00374414    8B85 E6434000   MOV     EAX, DWORD PTR SS:\[EBP+<flg\_CRC\_Check>\]  
0037441A    83F8 01         CMP     EAX, 1  
0037441D    75 08           JNZ     SHORT <not\_Check\_crc>                                       ; 判断是否需要进行 CRC 效验  
0037441F    3BF7            CMP     ESI, EDI                                                    ; 如果要检测，不相等的话就 OVER 了  
00374421    0F85 171E0000   JNZ     <Game\_Over>  
  
  
;♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀END♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀♀  
  
  
  
00374427 >  8D85 5F2D4000   LEA     EAX, DWORD PTR SS:\[EBP+402D5F\]                              ; not\_Check\_crc  
0037442D    50              PUSH    EAX  
0037442E    8B85 ED444000   MOV     EAX, DWORD PTR SS:\[EBP+<APIGetVersion>\]                   ; kernel32.GetVersion  
00374434    E9 A1050000     JMP     <proc\_Run\_FUN>    
00374439    90              NOP  
0037443A    33C0            XOR     EAX, EAX  
0037443C    F7F0            DIV     EAX                                                         ; 除 0 异常  
0037443E    E9 FB1D0000     JMP     <Game\_Over>  
00374443    8985 88474000   MOV     DWORD PTR SS:\[EBP+<save\_VerInfo>\], EAX  
00374449    8D85 782D4000   LEA     EAX, DWORD PTR SS:\[EBP+402D78\]  
0037444F    50              PUSH    EAX  
00374450    8B85 1F454000   MOV     EAX, DWORD PTR SS:\[EBP+<APIGetCurProcess>\]                ; kernel32.GetCurrentProcess  
00374456    E9 7F050000     JMP     <proc\_Run\_FUN>  
0037445B    90              NOP  
0037445C    8985 90474000   MOV     DWORD PTR SS:\[EBP+<\_dwCurProc>\], EAX  
00374462    8D85 912D4000   LEA     EAX, DWORD PTR SS:\[EBP+402D91\]  
00374468    50              PUSH    EAX  
00374469    8B85 32454000   MOV     EAX, DWORD PTR SS:\[EBP+<APIGetCurProcId>\]                 ; kernel32.GetCurrentProcessId  
0037446F    E9 66050000     JMP     <proc\_Run\_FUN>  
00374474    90              NOP  
00374475    8985 94474000   MOV     DWORD PTR SS:\[EBP+<\_dwCurProcId>\], EAX  
0037447B    8D85 B52D4000   LEA     EAX, DWORD PTR SS:\[EBP+402DB5\]  
00374481    50              PUSH    EAX  
00374482    8B85 47454000   MOV     EAX, DWORD PTR SS:\[EBP+<APIGetCmdLine>\]                   ; kernel32.GetCommandLineA  
00374488    E9 4D050000     JMP     <proc\_Run\_FUN>  
0037448D    90              NOP  
0037448E    9C              PUSHFD  
0037448F    9C              PUSHFD  
00374490    58              POP     EAX                                                         ; 0012FFE0  
00374491    80CC 01         OR      AH, 1                                                       ; 这里也是最后八个异常里的  
00374494    50              PUSH    EAX  
00374495    9D              POPFD  
00374496    9D              POPFD  
00374497  ^ EB F5           JMP     SHORT 0037448E  
00374499    8985 98474000   MOV     DWORD PTR SS:\[EBP+<ptrGetCmdLine>\], EAX  
0037449F    6A 00           PUSH    0  
003744A1    8D85 D02D4000   LEA     EAX, DWORD PTR SS:\[EBP+402DD0\]  
003744A7    50              PUSH    EAX  
003744A8    8B85 F9444000   MOV     EAX, DWORD PTR SS:\[EBP+<APIGetModulehndA>\]                ; kernel32.GetModuleHandleA  
003744AE    E9 27050000     JMP     <proc\_Run\_FUN>  
003744B3    90              NOP  
003744B4    8985 8C474000   MOV     DWORD PTR SS:\[EBP+<\_dwHandle>\], EAX  
003744BA    FFB5 64464000   PUSH    DWORD PTR SS:\[EBP+<APIwsASend>\]                           ; 对 WSASend 特别处理  
003744C0    8D85 E5484000   LEA     EAX, DWORD PTR SS:\[EBP+4048E5\]  
003744C6    50              PUSH    EAX  
003744C7    E8 23050000     CALL    <steal code>  
003744CC    FFB5 6D464000   PUSH    DWORD PTR SS:\[EBP+<APIWSARecv>\]                           ; WS2\_32.WSARecv  
003744D2    8D85 25494000   LEA     EAX, DWORD PTR SS:\[EBP+404925\]  
003744D8    50              PUSH    EAX  
003744D9    E8 11050000     CALL    <steal code>  
003744DE    8D85 AC484000   LEA     EAX, DWORD PTR SS:\[EBP+<strShellTmpMap>\]  
003744E4    50              PUSH    EAX  
003744E5    68 00010000     PUSH    100  
003744EA    6A 00           PUSH    0  
003744EC    6A 04           PUSH    4  
003744EE    6A 00           PUSH    0  
003744F0    6A FF           PUSH    -1  
003744F2    8D85 212E4000   LEA     EAX, DWORD PTR SS:\[EBP+402E21\]  
003744F8    50              PUSH    EAX  
003744F9    8B85 75454000   MOV     EAX, DWORD PTR SS:\[EBP+<CreateFileMapA>\]                  ; kernel32.CreateFileMappingA  
003744FF    E9 D6040000     JMP     <proc\_Run\_FUN>  
00374504    90              NOP  
00374505    83F8 00         CMP     EAX, 0  
00374508    0F84 301D0000   JE      <Game\_Over>  
0037450E    8985 B8484000   MOV     DWORD PTR SS:\[EBP+<hMAP1>\], EAX  
00374514    68 00010000     PUSH    100  
00374519    6A 00           PUSH    0  
0037451B    6A 00           PUSH    0  
0037451D    6A 06           PUSH    6  
0037451F    50              PUSH    EAX  
00374520    8D85 4F2E4000   LEA     EAX, DWORD PTR SS:\[EBP+402E4F\]  
00374526    50              PUSH    EAX  
00374527    8B85 89454000   MOV     EAX, DWORD PTR SS:\[EBP+<APIMapViewofFile>\]                ; kernel32.MapViewOfFile  
0037452D    E9 A8040000     JMP     <proc\_Run\_FUN>  
00374532    90              NOP  
00374533    8985 BC484000   MOV     DWORD PTR SS:\[EBP+<hMapview1>\], EAX  
00374539    8BF8            MOV     EDI, EAX  
0037453B    8DB5 C0484000   LEA     ESI, DWORD PTR SS:\[EBP+4048C0\]  
00374541    B9 0A000000     MOV     ECX, 0A  
00374546    F3:A4           REP     MOVS BYTE PTR ES:\[EDI\], BYTE PTR DS:\[ESI\]                   ; 把 ShellMap 字符串复制到 990000 处  
00374548    8B85 88474000   MOV     EAX, DWORD PTR SS:\[EBP+<save\_VerInfo>\]  
0037454E    3D 00000080     CMP     EAX, 80000000  
00374553    73 16           JNB     SHORT <OSisWin9x>                                           ; 判断系统是否为 WinNT 或以上系统  
00374555    64:FF35 3000000>PUSH    DWORD PTR FS:\[30\]                                           ; 如果是 NT 系统则检测 IsDebuggerPresent  
0037455C    58              POP     EAX                                                         ; 检测 Ring3 级调试器  
0037455D    0FB658 02       MOVZX   EBX, BYTE PTR DS:\[EAX+2\]  
00374561    0ADB            OR      BL, BL  
00374563    0F85 D51C0000   JNZ     <Game\_Over>  
00374569    EB 2A           JMP     SHORT 00374595  
0037456B >  50              PUSH    EAX                                                         ; OSisWin9x  
0037456C    0F014C24 FE     SIDT    FWORD PTR SS:\[ESP\-2\]  
00374571    5B              POP     EBX                                                         ; 0012FFE0  
00374572    83C3 18         ADD     EBX, 18  
00374575    8B4B 04         MOV     ECX, DWORD PTR DS:\[EBX+4\]  
00374578    66:8B0B         MOV     CX, WORD PTR DS:\[EBX\]  
0037457B    8B53 0C         MOV     EDX, DWORD PTR DS:\[EBX+C\]  
0037457E    66:8B53 08      MOV     DX, WORD PTR DS:\[EBX+8\]  
00374582    8B43 14         MOV     EAX, DWORD PTR DS:\[EBX+14\]  
00374585    66:8B43 10      MOV     AX, WORD PTR DS:\[EBX+10\]  
00374589    2BC2            SUB     EAX, EDX  
0037458B    2BD1            SUB     EDX, ECX  
0037458D    2BC2            SUB     EAX, EDX  
0037458F    0F85 A91C0000   JNZ     <Game\_Over>  
  
;◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎重定位表处理◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎  
  
  如果是 DLL 的话，这里填充重定位表，修复重定位表时要注意一点，如果加密时选择了加密输入和特殊代码加密的话，是不能直接通过  
修改这里来获取到全部的重定位表．  
  
  
00374595    8BB5 D6434000   MOV     ESI, DWORD PTR SS:\[EBP+<Reloc\_RVA(DLL)>\]                  ; 判断是否有重定位表，一般的 EXE 这里为 0  
0037459B    0BF6            OR      ESI, ESI  
0037459D    74 4C           JE      SHORT <no\_Reloc\_Tab>                                        ; 如果没有重定位表则跳  
0037459F    03B5 B2434000   ADD     ESI, DWORD PTR SS:\[EBP+<IMGBASE>\]                         ; 00400000  
003745A5    8BBD B2434000   MOV     EDI, DWORD PTR SS:\[EBP+<IMGBASE>\]                         ; 00400000  
003745AB    8BDF            MOV     EBX, EDI  
003745AD    2BBD D2434000   SUB     EDI, DWORD PTR SS:\[EBP+<Reloc\_BASE>\]                      ; 重定位后的实际基址  
003745B3    0FB606          MOVZX   EAX, BYTE PTR DS:\[ESI\]  
003745B6    EB 2F           JMP     SHORT 003745E7  
003745B8 >  3C 01           CMP     AL, 1                                                       ; Loop\_Fill\_Reloc\_Tab  
003745BA    75 15           JNZ     SHORT 003745D1  
003745BC    46              INC     ESI  
003745BD    0FB606          MOVZX   EAX, BYTE PTR DS:\[ESI\]  
003745C0    3C 02           CMP     AL, 2  
003745C2    75 08           JNZ     SHORT 003745CC  
003745C4    46              INC     ESI  
003745C5    031E            ADD     EBX, DWORD PTR DS:\[ESI\]  
003745C7    83C6 04         ADD     ESI, 4  
003745CA    EB 18           JMP     SHORT 003745E4  
003745CC    46              INC     ESI  
003745CD    03D8            ADD     EBX, EAX  
003745CF    EB 13           JMP     SHORT 003745E4  
003745D1    3C 02           CMP     AL, 2  
003745D3    75 0A           JNZ     SHORT 003745DF  
003745D5    46              INC     ESI  
003745D6    031E            ADD     EBX, DWORD PTR DS:\[ESI\]  
003745D8    013B            ADD     DWORD PTR DS:\[EBX\], EDI                                     ; 填充重定位表  
003745DA    83C6 04         ADD     ESI, 4  
003745DD    EB 05           JMP     SHORT 003745E4  
003745DF    46              INC     ESI  
003745E0    03D8            ADD     EBX, EAX  
003745E2    013B            ADD     DWORD PTR DS:\[EBX\], EDI                                     ; 填充重定位表  
003745E4    0FB606          MOVZX   EAX, BYTE PTR DS:\[ESI\]  
003745E7    0AC0            OR      AL, AL  
003745E9  ^ 75 CD           JNZ     SHORT <Loop\_Fill\_Reloc\_Tab>  
  
;◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎重定位表处理完毕◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎  
  
  
003745EB >  8CC9            MOV     CX, CS                                                      ; no\_Reloc\_Tab  
003745ED    32C9            XOR     CL, CL  
003745EF    0BC9            OR      ECX, ECX                                                    ; 判断操作系统  
003745F1    74 32           JE      SHORT <Os\_isWinNT>  
003745F3    50              PUSH    EAX  
003745F4    0F014C24 FE     SIDT    FWORD PTR SS:\[ESP\-2\]  
003745F9    5F              POP     EDI                                                         ; 0012FFE0  
003745FA    83C7 20         ADD     EDI, 20  
003745FD    8B4F 04         MOV     ECX, DWORD PTR DS:\[EDI+4\]  
00374600    66:8B0F         MOV     CX, WORD PTR DS:\[EDI\]  
00374603    FA              CLI  
00374604    8DB5 434B4000   LEA     ESI, DWORD PTR SS:\[EBP+404B43\]  
0037460A    66:8937         MOV     WORD PTR DS:\[EDI\], SI  
0037460D    C1EE 10         SHR     ESI, 10  
00374610    66:8977 06      MOV     WORD PTR DS:\[EDI+6\], SI  
00374614    FB              STI  
00374615    CD 04           INT     4  
00374617    FA              CLI  
00374618    66:890F         MOV     WORD PTR DS:\[EDI\], CX  
0037461B    C1E9 10         SHR     ECX, 10  
0037461E    66:894F 06      MOV     WORD PTR DS:\[EDI+6\], CX  
00374622    FB              STI  
00374623    EB 37           JMP     SHORT 0037465C  
00374625 >  E8 0E000000     CALL    <Check\_Debug>                                               ; Os\_isWinNT  
0037462A    8B4C24 0C       MOV     ECX, DWORD PTR SS:\[ESP+C\]  
0037462E    8381 B8000000 0>ADD     DWORD PTR DS:\[ECX+B8\], 2                                    ; 异常地址+2  
00374635    33C0            XOR     EAX, EAX  
00374637    C3              RETN  
00374638 >  64:FF35 0000000>PUSH    DWORD PTR FS:\[0\]                                            ; Check\_Debug  
0037463F    64:8925 0000000>MOV     DWORD PTR FS:\[0\], ESP  
00374646    33C0            XOR     EAX, EAX  
00374648    CD 01           INT     1  
0037464A    40              INC     EAX  
0037464B    40              INC     EAX  
0037464C    0BC0            OR      EAX, EAX  
0037464E    64:8F05 0000000>POP     DWORD PTR FS:\[0\]                                            ; 0012FFE0  
00374655    58              POP     EAX                                                         ; 0012FFE0  
00374656    0F84 E21B0000   JE      <Game\_Over>                                                 ; 如果是 sice 这里就要处理了  
0037465C    8BB5 FA434000   MOV     ESI, DWORD PTR SS:\[EBP+4043FA\]                              ; 修正 JMP IAT 到 HOOK　table  
00374662    0BF6            OR      ESI, ESI                                                    ; 可惜我这个程序没有  
00374664    74 27           JE      SHORT 0037468D  
00374666    03B5 B2434000   ADD     ESI, DWORD PTR SS:\[EBP+<IMGBASE>\]                         ; 00400000  
0037466C    EB 18           JMP     SHORT 00374686  
0037466E    8B46 02         MOV     EAX, DWORD PTR DS:\[ESI+2\]  
00374671    C1E0 05         SHL     EAX, 5  
00374674    0385 FE434000   ADD     EAX, DWORD PTR SS:\[EBP+<hMEM\_IAT\_RELOC\_1>\]  
0037467A    2BC6            SUB     EAX, ESI  
0037467C    48              DEC     EAX  
0037467D    83E8 05         SUB     EAX, 5  
00374680    8946 02         MOV     DWORD PTR DS:\[ESI+2\], EAX  
00374683    83C6 06         ADD     ESI, 6  
00374686    66:813E 90E9    CMP     WORD PTR DS:\[ESI\], 0E990  
0037468B  ^ 74 E1           JE      SHORT 0037466E  
  
;++++++++++++++++++++++++++++++++++++++++++++++++++++对 DELPHI 程序的特别处理++++++++++++++++++++++++++++++++++++++++++++++++++++  
  
  如果是 delphi 的程序，加密时选择了 DELPHI＋＋选项，壳就会把 mainform 的部分数据搬到壳里去了，脱壳的话就得把它找回来．  
  
0037468D    8B85 0A444000   MOV     EAX, DWORD PTR SS:\[EBP+<flgDelphi++>\]                     ; 这里是对 DELPHI 的 MAINFORM 的特别处理  
00374693    0BC0            OR      EAX, EAX  
00374695    74 3F           JE      SHORT 003746D6                                              ; 如果不是 delphi 的程序或没有选择 Delphi++选项就会跳过这里了:-)  
00374697    8DB5 A01A4000   LEA     ESI, DWORD PTR SS:\[EBP+<Crc\_Start\_addr>\]  
0037469D    03F0            ADD     ESI, EAX  
0037469F    8B1E            MOV     EBX, DWORD PTR DS:\[ESI\]                                     ; MAINFORM 的原始参考 RVA  
003746A1    039D B2434000   ADD     EBX, DWORD PTR SS:\[EBP+<IMGBASE>\]                         ; 00400000  
003746A7    C706 00000000   MOV     DWORD PTR DS:\[ESI\], 0  
003746AD    83C6 04         ADD     ESI, 4  
003746B0    8933            MOV     DWORD PTR DS:\[EBX\], ESI                                     ; \[esi\]就是抽取 mainform 的数据保存处  
003746B2    0FB70E          MOVZX   ECX, WORD PTR DS:\[ESI\]                                      ; ebx 中查找和\[esi\]对应的第一个字节空处就是了  
003746B5    83C6 02         ADD     ESI, 2  
003746B8    8B9D B2434000   MOV     EBX, DWORD PTR SS:\[EBP+<IMGBASE>\]                         ; 00400000  
003746BE    8B95 D2434000   MOV     EDX, DWORD PTR SS:\[EBP+<Reloc\_BASE>\]                      ; 00400000  
003746C4    EB 0C           JMP     SHORT 003746D2  
003746C6    2956 02         SUB     DWORD PTR DS:\[ESI+2\], EDX                                   ; 对重定位的处理  
003746C9    015E 02         ADD     DWORD PTR DS:\[ESI+2\], EBX                                   ; exe 文件一般不用去管的  
003746CC    0FB706          MOVZX   EAX, WORD PTR DS:\[ESI\]  
003746CF    03F0            ADD     ESI, EAX  
003746D1    49              DEC     ECX  
003746D2    0BC9            OR      ECX, ECX  
003746D4  ^ 75 F0           JNZ     SHORT 003746C6  
  
;++++++++++++++++++++++++++++++++++++++++++++++++++++特殊处理结束++++++++++++++++++++++++++++++++++++++++++++++++++++ 

;█████████████████████████ Anti Dump █████████████████████████   
  
003746D6    6A 04           PUSH    4  
003746D8    68 00100000     PUSH    1000  
003746DD    68 00100000     PUSH    1000  
003746E2    6A 00           PUSH    0  
003746E4    8D85 13304000   LEA     EAX, DWORD PTR SS:\[EBP+403013\]  
003746EA    50              PUSH    EAX  
003746EB    8B85 AE434000   MOV     EAX, DWORD PTR SS:\[EBP+<APIVirtualAlloc>\]                 ; kernel32.VirtualAlloc  
003746F1    E9 E4020000     JMP     <proc\_Run\_FUN>  
003746F7    8985 1A444000   MOV     DWORD PTR SS:\[EBP+<hMEM46f7>\], EAX  
003746FD    8185 1A444000 0>ADD     DWORD PTR SS:\[EBP+<hMEM46f7>\], 1000                       ; 修改 VirtualSize??  
00374707    64:FF35 3000000>PUSH    DWORD PTR FS:\[30\]  
0037470E    58              POP     EAX                                                         ; 0012FFE0  
0037470F    85C0            TEST    EAX, EAX  
00374711    78 0F           JS      SHORT 00374722                                              ; ??检测操作系统??,判断是否可以 Anti\_dump  
00374713    8B40 0C         MOV     EAX, DWORD PTR DS:\[EAX+C\]  
00374716    8B40 0C         MOV     EAX, DWORD PTR DS:\[EAX+C\]  
00374719    C740 20 0010000>MOV     DWORD PTR DS:\[EAX+20\], 1000                                 ; anti\_dump  
00374720    EB 39           JMP     SHORT 0037475B  
00374722    6A 00           PUSH    0  
00374724    8D85 53304000   LEA     EAX, DWORD PTR SS:\[EBP+403053\]  
0037472A    50              PUSH    EAX  
0037472B    8B85 A6434000   MOV     EAX, DWORD PTR SS:\[EBP+<GetModhandle>\]                    ; kernel32.GetModuleHandleA  
00374731    E9 A4020000     JMP     <proc\_Run\_FUN>  
00374736    90              NOP  
00374737    85D2            TEST    EDX, EDX  
00374739    79 20           JNS     SHORT 0037475B  
0037473B    837A 08 FF      CMP     DWORD PTR DS:\[EDX+8\], -1  
0037473F    75 1A           JNZ     SHORT 0037475B  
00374741    8B52 04         MOV     EDX, DWORD PTR DS:\[EDX+4\]  
00374744    C742 50 0010000>MOV     DWORD PTR DS:\[EDX+50\], 1000  
0037474B    64:FF35 2000000>PUSH    DWORD PTR FS:\[20\]  
00374752    58              POP     EAX                                                         ; 0012FFE0  
00374753    85C0            TEST    EAX, EAX  
00374755    0F85 E31A0000   JNZ     <Game\_Over>  
0037475B    50              PUSH    EAX  
0037475C    8BC4            MOV     EAX, ESP  
0037475E    50              PUSH    EAX  
0037475F    6A 04           PUSH    4  
00374761    68 00100000     PUSH    1000  
00374766    FFB5 B2434000   PUSH    DWORD PTR SS:\[EBP+<IMGBASE>\]                              ; 00400000  
0037476C    8D85 9B304000   LEA     EAX, DWORD PTR SS:\[EBP+40309B\]  
00374772    50              PUSH    EAX  
00374773    8B85 28464000   MOV     EAX, DWORD PTR SS:\[EBP+<APIVirtualProtect>\]               ; kernel32.VirtualProtect  
00374779    E9 5C020000     JMP     <proc\_Run\_FUN>  
0037477E    90              NOP  
0037477F    83C4 04         ADD     ESP, 4  
00374782    0BC0            OR      EAX, EAX  
00374784    74 0F           JE      SHORT 00374795                                              ; 修改 PE 文件头为可写  
00374786    8B95 B2434000   MOV     EDX, DWORD PTR SS:\[EBP+<IMGBASE>\]                         ; 00400000  
0037478C    0352 3C         ADD     EDX, DWORD PTR DS:\[EDX+3C\]  
0037478F    8B42 30         MOV     EAX, DWORD PTR DS:\[EDX+30\]  
00374792    8942 2C         MOV     DWORD PTR DS:\[EDX+2C\], EAX                                  ; 修改 BaseOfCode 为 1000  
  
;█████████████████████████ End █████████████████████████   
  
  
00374795    8DB5 07484000   LEA     ESI, DWORD PTR SS:\[EBP+<Author's TIP>\]  
0037479B    8BFE            MOV     EDI, ESI  
0037479D    B9 4F000000     MOV     ECX, 4F  
003747A2    EB 05           JMP     SHORT 003747A9                                              ; 显示 I am xxxx  
003747A4    AC              LODS    BYTE PTR DS:\[ESI\]  
003747A5    2C 80           SUB     AL, 80  
003747A7    AA              STOS    BYTE PTR ES:\[EDI\]  
003747A8    49              DEC     ECX  
003747A9    0BC9            OR      ECX, ECX  
003747AB  ^ 75 F7           JNZ     SHORT 003747A4  
003747AD    8DB5 07484000   LEA     ESI, DWORD PTR SS:\[EBP+<Author's TIP>\]  
003747B3    8BFE            MOV     EDI, ESI  
003747B5    B9 4F000000     MOV     ECX, 4F  
003747BA    EB 05           JMP     SHORT 003747C1  
003747BC    AC              LODS    BYTE PTR DS:\[ESI\]                                           ; 显示完再清除掉  
003747BD    04 80           ADD     AL, 80  
003747BF    AA              STOS    BYTE PTR ES:\[EDI\]  
003747C0    49              DEC     ECX  
003747C1    0BC9            OR      ECX, ECX  
003747C3  ^ 75 F7           JNZ     SHORT 003747BC  
  
  
;◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇Calculate UnLock Key◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇  
  
    解出关键 KEY，这个非常重要，KEY 由内存代码效验值和 Drx 的值计算得来，这个程序最终 KEY 为：299A8442.  
  
  
003747C5    8B85 0E444000   MOV     EAX, DWORD PTR SS:\[EBP+<OEP(RVA)>\]  
003747CB    0385 B2434000   ADD     EAX, DWORD PTR SS:\[EBP+<IMGBASE>\]                         ; 计算出 OEP 的 VA  
003747D1    894424 EC       MOV     DWORD PTR SS:\[ESP\-14\], EAX                                  ; OEP 保存到 ESP-14 中  
003747D5    896C24 E8       MOV     DWORD PTR SS:\[ESP\-18\], EBP                                  ; 保存 EBP  
003747D9    C785 F6434000 0>MOV     DWORD PTR SS:\[EBP+<UnLock\_Important\_Key>\], 0                ; 初始化关键 KEY，这里的 KEY 是整个壳的关键  
003747E3    33C0            XOR     EAX, EAX                                                    ; 没有 KEY 后面就会出错的  
003747E5    8DB5 A01A4000   LEA     ESI, DWORD PTR SS:\[EBP+<Crc\_Start\_addr>\]                  ; 从内存 00373184 处开始计算出关键 KEY  
003747EB    B9 FE280000     MOV     ECX, 28FE                                                   ; 内存代码检测大小 28fe  
003747F0    C1E9 02         SHR     ECX, 2  
003747F3    EB 08           JMP     SHORT 003747FD  
003747F5    AD              LODS    DWORD PTR DS:\[ESI\]                                          ; 如果内存代码修改过，这个 KEY 就肯定会不正确  
003747F6    3185 F6434000   XOR     DWORD PTR SS:\[EBP+<UnLock\_Important\_Key>\], EAX    ;这里关键一定要记下正确的值否则后面解码会出错  
003747FC    49              DEC     ECX  
003747FD    0BC9            OR      ECX, ECX  
003747FF  ^ 75 F4           JNZ     SHORT 003747F5  
00374801    8B4424 EC       MOV     EAX, DWORD PTR SS:\[ESP\-14\]  
00374805    2B85 B2434000   SUB     EAX, DWORD PTR SS:\[EBP+<IMGBASE>\]                         ; 00400000  
0037480B    8985 0E444000   MOV     DWORD PTR SS:\[EBP+<OEP(RVA)>\], EAX  
00374811    8B6C24 E8       MOV     EBP, DWORD PTR SS:\[ESP\-18\]  
00374815    8B85 F6434000   MOV     EAX, DWORD PTR SS:\[EBP+<UnLock\_Important\_Key>\]  
0037481B    E8 3F000000     CALL    <Fuck\_Int3>  
00374820    8B4C24 0C       MOV     ECX, DWORD PTR SS:\[ESP+C\]  
00374824    FF81 B8000000   INC     DWORD PTR DS:\[ECX+B8\]                                       ; 异常地址+1  
0037482A    33C0            XOR     EAX, EAX  
0037482C    3341 04         XOR     EAX, DWORD PTR DS:\[ECX+4\]                                   ; 取出 Dr0 参与运算  
0037482F    0341 08         ADD     EAX, DWORD PTR DS:\[ECX+8\]                                   ; 取出 Dr1 参与运算  
00374832    3341 0C         XOR     EAX, DWORD PTR DS:\[ECX+C\]                                   ; 取出 Dr2 参与运算  
00374835    0341 10         ADD     EAX, DWORD PTR DS:\[ECX+10\]                                  ; 取出 Dr3 参与运算  
00374838    0181 B0000000   ADD     DWORD PTR DS:\[ECX+B0\], EAX                                  ; 算出的值保存回 regEAX,壳的关键陷阱  
0037483E    60              PUSHAD                                                              ; 如果 Dr0 被我们跟踪时破坏了则后面肯定出错  
0037483F    8D71 04         LEA     ESI, DWORD PTR DS:\[ECX+4\]  
00374842    8BA9 B4000000   MOV     EBP, DWORD PTR DS:\[ECX+B4\]  
00374848    8DBD 014A4000   LEA     EDI, DWORD PTR SS:\[EBP+404A01\]  
0037484E    81C7 E8000000   ADD     EDI, 0E8  
00374854    B9 06000000     MOV     ECX, 6  
00374859    F3:A5           REP     MOVS DWORD PTR ES:\[EDI\], DWORD PTR DS:\[ESI\]  
0037485B    61              POPAD  
0037485C    33C0            XOR     EAX, EAX  
0037485E    C3              RETN  
0037485F >  64:FF35 0000000>PUSH    DWORD PTR FS:\[0\]                                            ; Fuck\_Int3  
00374866    64:8925 0000000>MOV     DWORD PTR FS:\[0\], ESP  
0037486D    CC              INT3  
0037486E    90              NOP  
0037486F    64:8F05 0000000>POP     DWORD PTR FS:\[0\]                                            ; 0012FFE0  
00374876    83C4 04         ADD     ESP, 4  
00374879    8985 F6434000   MOV     DWORD PTR SS:\[EBP+<UnLock\_Important\_Key>\], EAX              ; 看到了吧，如果 Drx 被破坏或内存代码被修改过  
0037487F    33C0            XOR     EAX, EAX                                                    ; 那个关键 kEy 就肯定不对了,那样程序就会异常退出  
  
;◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇END◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇◇  
  
  
:\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_  
  
          最后一个部分特殊代码加密  
  这部分也是关键的，如果上面的 KEY 不正确这里处理的时候就会产生异常，特殊代码加密也就是把原程序中的 call \[address\]和 jmp \[address\]  
改成:  
  NOP  
  CALL  HOOKED\_ADDRESS  
  或者  
  NOP  
  JMP   HOOK\_ADDRESS  
  
  
00374881    8B8D E2434000   MOV     ECX, DWORD PTR SS:\[EBP+<flg\_specific\_Code\_Encrypt>\]       ; 特殊代码加密标志  
00374887    83F9 01         CMP     ECX, 1  
0037488A    0F85 AE000000   JNZ     <Disposal\_Hook\_code\_done>                                   ; 如果没有选择特殊代码加密这里会跳过  
00374890    8DBD A01A4000   LEA     EDI, DWORD PTR SS:\[EBP+<Crc\_Start\_addr>\]  
00374896    03BD 06444000   ADD     EDI, DWORD PTR SS:\[EBP+404406\]  
0037489C    8DB5 5E344000   LEA     ESI, DWORD PTR SS:\[EBP+40345E\]  
003748A2 >  8B0F            MOV     ECX, DWORD PTR DS:\[EDI\]                                     ; Loop\_Hook\_Encrypt\_code  
003748A4    0BC9            OR      ECX, ECX  
003748A6    75 05           JNZ     SHORT 003748AD  
003748A8    E9 91000000     JMP     <Disposal\_Hook\_code\_done>  
003748AD    83F8 01         CMP     EAX, 1  
003748B0    75 21           JNZ     SHORT 003748D3  
003748B2    81E1 FFFFFF7F   AND     ECX, 7FFFFFFF  
003748B8    038D B2434000   ADD     ECX, DWORD PTR SS:\[EBP+<IMGBASE>\]                         ; 00400000  
003748BE    2B8D D2434000   SUB     ECX, DWORD PTR SS:\[EBP+<Reloc\_BASE>\]                      ; 00400000  
003748C4    8BDE            MOV     EBX, ESI                                                    ; 这里不会直接计算出地址，还要用 call 运行时解压出来  
003748C6    2BD9            SUB     EBX, ECX  
003748C8    8959 FC         MOV     DWORD PTR DS:\[ECX\-4\], EBX                                   ; 填充 hook 后的地址  
003748CB    66:C741 FA 90E8 MOV     WORD PTR DS:\[ECX\-6\], 0E890                                  ; 填充为 call hookadd  
003748D1    EB 60           JMP     SHORT 00374933  
003748D3    8BD1            MOV     EDX, ECX  
003748D5    81E1 FFFFFF7F   AND     ECX, 7FFFFFFF  
003748DB    038D B2434000   ADD     ECX, DWORD PTR SS:\[EBP+<IMGBASE>\]                         ; 00400000  
003748E1    2B8D D2434000   SUB     ECX, DWORD PTR SS:\[EBP+<Reloc\_BASE>\]                      ; 00400000  
003748E7    81E2 00000080   AND     EDX, 80000000                                               ; 如果是 call address,则值为 80xxxxxx  
003748ED    0BD2            OR      EDX, EDX                                                    ; 如果 edx=0 表示是 jmp addr  
003748EF    75 08           JNZ     SHORT <is\_long\_jmp>                                         ; 不是 jmp address 就是 25xxxxxx  
003748F1    66:C741 FA 90E8 MOV     WORD PTR DS:\[ECX\-6\], 0E890                                  ; 如果是非 0 则 call address  
003748F7    EB 06           JMP     SHORT 003748FF  
003748F9 >  66:C741 FA 90E9 MOV     WORD PTR DS:\[ECX\-6\], 0E990                                  ; is\_long\_jmp  
003748FF    8B57 04         MOV     EDX, DWORD PTR DS:\[EDI+4\]  
00374902    0395 F6434000   ADD     EDX, DWORD PTR SS:\[EBP+<UnLock\_Important\_Key>\]              ; 这里也是阴险之处，如果关键 KEY 不正确这里就会异常  
00374908    50              PUSH    EAX  
00374909    8B07            MOV     EAX, DWORD PTR DS:\[EDI\]  
0037490B    25 FFFFFF7F     AND     EAX, 7FFFFFFF  
00374910    2BD0            SUB     EDX, EAX  
00374912    F7D2            NOT     EDX  
00374914    C1C2 10         ROL     EDX, 10  
00374917    0395 B2434000   ADD     EDX, DWORD PTR SS:\[EBP+<IMGBASE>\]                         ; 00400000  
0037491D    2B95 D2434000   SUB     EDX, DWORD PTR SS:\[EBP+<Reloc\_BASE>\]                      ; 这里计算出正确 jmp \[address\]中的 address,sub 后 edx=address  
00374923    8B12            MOV     EDX, DWORD PTR DS:\[EDX\]  
00374925    2BD1            SUB     EDX, ECX  
00374927    8951 FC         MOV     DWORD PTR DS:\[ECX\-4\], EDX                                   ; 写入加密后的地址  
0037492A    33C0            XOR     EAX, EAX  
0037492C    48              DEC     EAX  
0037492D    8907            MOV     DWORD PTR DS:\[EDI\], EAX                                     ; 一填充完就把相关地址填-1  
0037492F    8947 04         MOV     DWORD PTR DS:\[EDI+4\], EAX                                   ; 地址+4 处也填-1  
00374932    58              POP     EAX                                                         ; 0012FFE0  
00374933    83C7 08         ADD     EDI, 8  
00374936    83F0 01         XOR     EAX, 1  
00374939  ^ E9 64FFFFFF     JMP     <Loop\_Hook\_Encrypt\_code>  
  
  
:\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_  
  
  
0037493E >  8B85 C2434000   MOV     EAX, DWORD PTR SS:\[EBP+4043C2\]                              ; Disposal\_Hook\_code\_done  
00374944    0BC0            OR      EAX, EAX  
00374946    75 14           JNZ     SHORT 0037495C  
00374948    8B85 C9484000   MOV     EAX, DWORD PTR SS:\[EBP+4048C9\]  
0037494E    0BC0            OR      EAX, EAX  
00374950    74 0A           JE      SHORT 0037495C  
00374952    0385 B2434000   ADD     EAX, DWORD PTR SS:\[EBP+<IMGBASE>\]                         ; 00400000  
00374958    60              PUSHAD  
00374959    FFD0            CALL    EAX  
0037495B    61              POPAD  
0037495C    8BB5 DD484000   MOV     ESI, DWORD PTR SS:\[EBP+4048DD\]                              ; 准备从 401000 处开始计算内存中原程序的 CRC 值  
00374962    03B5 B2434000   ADD     ESI, DWORD PTR SS:\[EBP+<IMGBASE>\]                         ; 00400000  
00374968    8B8D E1484000   MOV     ECX, DWORD PTR SS:\[EBP+4048E1\]                              ; 计算大小 48000  
0037496E    E8 E8020000     CALL    <Calculate\_CRC>  
00374973    8985 CA434000   MOV     DWORD PTR SS:\[EBP+<save\_Mem\_CRC\_Key>\], EAX                ; 保存计算后的 crc 值，不知道有什么用:-(  
00374979    8BC5            MOV     EAX, EBP  
0037497B    8DB5 014A4000   LEA     ESI, DWORD PTR SS:\[EBP+404A01\]  
00374981    0146 04         ADD     DWORD PTR DS:\[ESI+4\], EAX                                   ; 这里准备进入八个异常了.  
00374984    0146 08         ADD     DWORD PTR DS:\[ESI+8\], EAX  
00374987    83C6 20         ADD     ESI, 20  
0037498A    0146 04         ADD     DWORD PTR DS:\[ESI+4\], EAX  
0037498D    83C6 20         ADD     ESI, 20  
00374990    0146 04         ADD     DWORD PTR DS:\[ESI+4\], EAX  
00374993    0146 08         ADD     DWORD PTR DS:\[ESI+8\], EAX  
00374996    83C6 20         ADD     ESI, 20  
00374999    0146 04         ADD     DWORD PTR DS:\[ESI+4\], EAX  
0037499C    83C6 20         ADD     ESI, 20  
0037499F    0146 04         ADD     DWORD PTR DS:\[ESI+4\], EAX  
003749A2    83C6 20         ADD     ESI, 20  
003749A5    0146 04         ADD     DWORD PTR DS:\[ESI+4\], EAX  
003749A8    83C6 20         ADD     ESI, 20  
003749AB    0146 04         ADD     DWORD PTR DS:\[ESI+4\], EAX  
003749AE    83C6 20         ADD     ESI, 20  
003749B1    0146 04         ADD     DWORD PTR DS:\[ESI+4\], EAX  
003749B4    8DB5 FD494000   LEA     ESI, DWORD PTR SS:\[EBP+4049FD\]  
003749BA    0106            ADD     DWORD PTR DS:\[ESI\], EAX  
003749BC    8D85 014B4000   LEA     EAX, DWORD PTR SS:\[EBP+<Last\_SEHS\_Disposal>\]  
003749C2    50              PUSH    EAX  
003749C3    64:FF35 0000000>PUSH    DWORD PTR FS:\[0\]  
003749CA    64:8925 0000000>MOV     DWORD PTR FS:\[0\], ESP  
003749D1    33C0            XOR     EAX, EAX  
003749D3    8B00            MOV     EAX, DWORD PTR DS:\[EAX\]  
003749D5    90              NOP  
003749D6    90              NOP  
003749D7    CC              INT3  
003749D8  ^ EB FB           JMP     SHORT 003749D5                                              ; 到这里看到这里也就预告即将到入口了  
  
到了这里，因为后面也没有什么重要的东西，我是直接在 00373A5E 处下断，然后过两个异常直接到 OEP 处了．  
  
全部分析完后得到两个重要的信息:  
  
Dr 的全部值  
  
DR0 0FFF90CA  
DR1 0FFFCF7F  
DR2 0FFF73B0  
DR3 0FFFCDEF  
DR6 FFFF0FF0  
DR7 00000555  
  
关键 KEY:299A8442  
  
当然其实有了关键 KEY 的话，就可以不用管 Drx 了.  
  
  
  
  
;&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&以下是各模块代码:&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&  
  
proc\_Run\_FUN:  
  
            003749DA >  50              PUSH    EAX                                                         ; proc\_Run\_FUN  
            003749DB    8B85 E5494000   MOV     EAX, DWORD PTR SS:\[EBP+<hMEM334d>\]                      　　; 因为这后面是各个过程来的  
            003749E1    50              PUSH    EAX  
            003749E2    E8 08000000     CALL    <steal code>  
            003749E7    8B85 E5494000   MOV     EAX, DWORD PTR SS:\[EBP+<hMEM334d>\]  
            003749ED    FFE0            JMP     EAX  
            003749EF >  60              PUSHAD                                                              ; steal code  
            003749F0    8B7C24 24       MOV     EDI, DWORD PTR SS:\[ESP+24\]                                  ; 0045F0A1  
            003749F4    8B7424 28       MOV     ESI, DWORD PTR SS:\[ESP+28\]                                  ; ESI=FUNCTION  
            003749F8 >  66:8B06         MOV     AX, WORD PTR DS:\[ESI\]                                       ; Loop\_chek\_code  
            003749FB    3C 50           CMP     AL, 50                                                      ; 判断是否在为 push eax push edi  
            003749FD    72 0A           JB      SHORT 00374A09  
            003749FF    3C 57           CMP     AL, 57  
            00374A01    77 06           JA      SHORT 00374A09  
            00374A03    8807            MOV     BYTE PTR DS:\[EDI\], AL                                       ; 如果是则直接抽取一字节  
            00374A05    46              INC     ESI  
            00374A06    47              INC     EDI  
            00374A07  ^ EB EF           JMP     SHORT <Loop\_chek\_code>  
            00374A09    3C 6A           CMP     AL, 6A                                                      ; 如果是 push 0 的方式则直接获取 2 个字节  
            00374A0B    75 09           JNZ     SHORT 00374A16  
            00374A0D    66:8907         MOV     WORD PTR DS:\[EDI\], AX  
            00374A10    46              INC     ESI  
            00374A11    46              INC     ESI  
            00374A12    47              INC     EDI  
            00374A13    47              INC     EDI  
            00374A14  ^ EB E2           JMP     SHORT <Loop\_chek\_code>  
            00374A16    3C 68           CMP     AL, 68                                                      ; 判断是否为 push address 的方式  
            00374A18    75 09           JNZ     SHORT 00374A23  
            00374A1A    B9 05000000     MOV     ECX, 5  
            00374A1F    F3:A4           REP     MOVS BYTE PTR ES:\[EDI\], BYTE PTR DS:\[ESI\]                   ; 如果是则抽取 5 个字节  
            00374A21  ^ EB D5           JMP     SHORT <Loop\_chek\_code>  
            00374A23    3C A1           CMP     AL, 0A1                                                     ; 判断是否为 Mov eax,\[address\]  
            00374A25    75 09           JNZ     SHORT 00374A30  
            00374A27    B9 05000000     MOV     ECX, 5  
            00374A2C    F3:A4           REP     MOVS BYTE PTR ES:\[EDI\], BYTE PTR DS:\[ESI\]                   ; 如果是则抽取 5 个字节  
            00374A2E  ^ EB C8           JMP     SHORT <Loop\_chek\_code>  
            00374A30    66:3D 2BD2      CMP     AX, 0D22B                                                   ; 判断是否为 sub edx,edx  
            00374A34    75 2D           JNZ     SHORT 00374A63  
            00374A36    66:8907         MOV     WORD PTR DS:\[EDI\], AX                                       ; 如果是则抽取两个字节  
            00374A39    46              INC     ESI  
            00374A3A    46              INC     ESI  
            00374A3B    47              INC     EDI  
            00374A3C    47              INC     EDI  
            00374A3D    8BDE            MOV     EBX, ESI  
            00374A3F    AC              LODS    BYTE PTR DS:\[ESI\]  
            00374A40    EB 01           JMP     SHORT 00374A43  
            00374A42    AC              LODS    BYTE PTR DS:\[ESI\]  
            00374A43    3C C3           CMP     AL, 0C3  
            00374A45  ^ 75 FB           JNZ     SHORT 00374A42                                              ; 循环找到 ret 处  
            00374A47    4E              DEC     ESI  
            00374A48    C607 68         MOV     BYTE PTR DS:\[EDI\], 68                                       ; 改变成 push address  
            00374A4B    8D47 0B         LEA     EAX, DWORD PTR DS:\[EDI+B\]                                   ; ret  
            00374A4E    8947 01         MOV     DWORD PTR DS:\[EDI+1\], EAX  
            00374A51    C647 05 68      MOV     BYTE PTR DS:\[EDI+5\], 68  
            00374A55    8977 06         MOV     DWORD PTR DS:\[EDI+6\], ESI  
            00374A58    C647 0A C3      MOV     BYTE PTR DS:\[EDI+A\], 0C3  
            00374A5C    83C7 0B         ADD     EDI, 0B  
            00374A5F    8BF3            MOV     ESI, EBX  
            00374A61  ^ EB 95           JMP     SHORT <Loop\_chek\_code>  
            00374A63    66:3D FF74      CMP     AX, 74FF                                                    ; 判断是否为 push dword \[reg\]  
            00374A67    75 09           JNZ     SHORT 00374A72  
            00374A69    B9 04000000     MOV     ECX, 4                                                      ; 如果是则抽取 4 个字节  
            00374A6E    F3:A4           REP     MOVS BYTE PTR ES:\[EDI\], BYTE PTR DS:\[ESI\]  
            00374A70  ^ EB 86           JMP     SHORT <Loop\_chek\_code>  
            00374A72    66:3D 8BEC      CMP     AX, 0EC8B                                                   ; 判断是否为 mov ebp,esp  
            00374A76    75 0C           JNZ     SHORT 00374A84  
            00374A78    B9 02000000     MOV     ECX, 2                                                      ; 如果是抽取 2 个字节  
            00374A7D    F3:A4           REP     MOVS BYTE PTR ES:\[EDI\], BYTE PTR DS:\[ESI\]  
            00374A7F  ^ E9 74FFFFFF     JMP     <Loop\_chek\_code>  
            00374A84    3C E8           CMP     AL, 0E8                                                     ; 判断是否为 call address  
            00374A86    75 25           JNZ     SHORT 00374AAD  
            00374A88    8D47 0B         LEA     EAX, DWORD PTR DS:\[EDI+B\]  
            00374A8B    C607 68         MOV     BYTE PTR DS:\[EDI\], 68                                       ; 如果是则改变为 push address  
            00374A8E    8947 01         MOV     DWORD PTR DS:\[EDI+1\], EAX                                   ; ret  
            00374A91    8D46 05         LEA     EAX, DWORD PTR DS:\[ESI+5\]  
            00374A94    0346 01         ADD     EAX, DWORD PTR DS:\[ESI+1\]  
            00374A97    C647 05 68      MOV     BYTE PTR DS:\[EDI+5\], 68  
            00374A9B    8947 06         MOV     DWORD PTR DS:\[EDI+6\], EAX  
            00374A9E    C647 0A C3      MOV     BYTE PTR DS:\[EDI+A\], 0C3  
            00374AA2    83C6 05         ADD     ESI, 5  
            00374AA5    83C7 0B         ADD     EDI, 0B  
            00374AA8  ^ E9 4BFFFFFF     JMP     <Loop\_chek\_code>  
            00374AAD    66:3D 64FF      CMP     AX, 0FF64  
            00374AB1    75 25           JNZ     SHORT 00374AD8  
            00374AB3    807E 02 32      CMP     BYTE PTR DS:\[ESI+2\], 32                                     ; 判断是否为 push \[edx\]  
            00374AB7    75 09           JNZ     SHORT 00374AC2  
            00374AB9    B9 03000000     MOV     ECX, 3                                                      ; 如果是则抽取 3 字节  
            00374ABE    F3:A4           REP     MOVS BYTE PTR ES:\[EDI\], BYTE PTR DS:\[ESI\]  
            00374AC0    EB 11           JMP     SHORT 00374AD3  
            00374AC2    807E 02 35      CMP     BYTE PTR DS:\[ESI+2\], 35                                     ; 判断是否为 puhs \[address\],带前缀的　  
            00374AC6    75 09           JNZ     SHORT 00374AD1  
            00374AC8    B9 07000000     MOV     ECX, 7                                                      ; 如果是则抽取 7 字节  
            00374ACD    F3:A4           REP     MOVS BYTE PTR ES:\[EDI\], BYTE PTR DS:\[ESI\]  
            00374ACF    EB 02           JMP     SHORT 00374AD3  
            00374AD1    EB 4B           JMP     SHORT 00374B1E  
            00374AD3  ^ E9 20FFFFFF     JMP     <Loop\_chek\_code>  
            00374AD8    66:3D 6489      CMP     AX, 8964  
            00374ADC    75 25           JNZ     SHORT 00374B03  
            00374ADE    807E 02 22      CMP     BYTE PTR DS:\[ESI+2\], 22                                     ; 判断是否为 mov \[reg\],reg  
            00374AE2    75 09           JNZ     SHORT 00374AED  
            00374AE4    B9 03000000     MOV     ECX, 3                                                      ; 如果是则抽取前三位,带前缀  
            00374AE9    F3:A4           REP     MOVS BYTE PTR ES:\[EDI\], BYTE PTR DS:\[ESI\]  
            00374AEB    EB 11           JMP     SHORT 00374AFE  
            00374AED    807E 02 25      CMP     BYTE PTR DS:\[ESI+2\], 25                                     ; 判断是否为 mov \[addr\],reg  
            00374AF1    75 09           JNZ     SHORT 00374AFC  
            00374AF3    B9 07000000     MOV     ECX, 7                                                      ; 如果是则抽取七位  
            00374AF8    F3:A4           REP     MOVS BYTE PTR ES:\[EDI\], BYTE PTR DS:\[ESI\]  
            00374AFA    EB 02           JMP     SHORT 00374AFE  
            00374AFC    EB 20           JMP     SHORT 00374B1E  
            00374AFE  ^ E9 F5FEFFFF     JMP     <Loop\_chek\_code>  
            00374B03    66:3D 83EC      CMP     AX, 0EC83                                                   ; 判断是否为 sub esp,val  
            00374B07    75 0C           JNZ     SHORT 00374B15  
            00374B09    B9 03000000     MOV     ECX, 3                                                      ; 如果是则抽取 3 字节  
            00374B0E    F3:A4           REP     MOVS BYTE PTR ES:\[EDI\], BYTE PTR DS:\[ESI\]  
            00374B10  ^ E9 E3FEFFFF     JMP     <Loop\_chek\_code>  
            00374B15    3C CC           CMP     AL, 0CC  
            00374B17    75 05           JNZ     SHORT 00374B1E                                              ; 判断指令的第一个字节是否为 cc,如果是则 over 了  
            00374B19    E9 20170000     JMP     <Game\_Over>  
            00374B1E    66:3D CD03      CMP     AX, 3CD  
            00374B22    75 05           JNZ     SHORT 00374B29                                              ; 同样判断是否为 int 3(CD 03)  
            00374B24    E9 15170000     JMP     <Game\_Over>  
            00374B29    C607 68         MOV     BYTE PTR DS:\[EDI\], 68                                       ; 如果都不是的话改变为 push address  
            00374B2C    8977 01         MOV     DWORD PTR DS:\[EDI+1\], ESI                                   ; ret  
            00374B2F    C647 05 C3      MOV     BYTE PTR DS:\[EDI+5\], 0C3  
            00374B33    83C7 06         ADD     EDI, 6  
            00374B36    897C24 FC       MOV     DWORD PTR SS:\[ESP\-4\], EDI  
            00374B3A    61              POPAD  
            00374B3B    8B4424 DC       MOV     EAX, DWORD PTR SS:\[ESP\-24\]                                  ; ntdll.RtlFreeHeap  
            00374B3F    C2 0800         RETN    8  
  
  
            00374B42    50              PUSH    EAX                                                         ; HookJmp  
            00374B43    60              PUSHAD  
            00374B44    E8 00000000     CALL    00374B49  
            00374B49    5D              POP     EBP                                                         ; 0012FFE0  
            00374B4A    81ED 65344000   SUB     EBP, 403465                                                 ; 计算出 EBP 的值  
            00374B50    8B7C24 24       MOV     EDI, DWORD PTR SS:\[ESP+24\]                                  ; 取出 call 的来源+5  
            00374B54    8DB5 A01A4000   LEA     ESI, DWORD PTR SS:\[EBP+<Crc\_Start\_addr>\]  
            00374B5A    03B5 06444000   ADD     ESI, DWORD PTR SS:\[EBP+404406\]  
            00374B60    8B06            MOV     EAX, DWORD PTR DS:\[ESI\]  
            00374B62    33D2            XOR     EDX, EDX  
            00374B64    B9 02000000     MOV     ECX, 2  
            00374B69    F7E1            MUL     ECX  
            00374B6B    D1E8            SHR     EAX, 1  
            00374B6D    0385 B2434000   ADD     EAX, DWORD PTR SS:\[EBP+<IMGBASE>\]                            ; 00400000  
            00374B73    2B85 D2434000   SUB     EAX, DWORD PTR SS:\[EBP+<Reloc\_BASE>\]                         ; 00400000  
            00374B79    3BF8            CMP     EDI, EAX  
            00374B7B    75 0A           JNZ     SHORT 00374B87  
            00374B7D    0AD2            OR      DL, DL  
            00374B7F    75 04           JNZ     SHORT 00374B85  
            00374B81    EB 09           JMP     SHORT 00374B8C  
            00374B83    EB 02           JMP     SHORT 00374B87  
            00374B85    EB 35           JMP     SHORT 00374BBC  
            00374B87    83C6 08         ADD     ESI, 8  
            00374B8A  ^ EB D4           JMP     SHORT 00374B60  
            00374B8C    8B46 04         MOV     EAX, DWORD PTR DS:\[ESI+4\]                                   ; 这里对 call \[address\]的处理  
            00374B8F    0385 F6434000   ADD     EAX, DWORD PTR SS:\[EBP+<UnLock\_Important\_Key>\]  
            00374B95    03BD D2434000   ADD     EDI, DWORD PTR SS:\[EBP+<Reloc\_BASE>\]                    　　; 00400000  
            00374B9B    2BBD B2434000   SUB     EDI, DWORD PTR SS:\[EBP+<IMGBASE>\]                       　　; 00400000  
            00374BA1    2BC7            SUB     EAX, EDI  
            00374BA3    F7D0            NOT     EAX  
            00374BA5    C1C0 10         ROL     EAX, 10  
            00374BA8    0385 B2434000   ADD     EAX, DWORD PTR SS:\[EBP+<IMGBASE>\]                       　　; 00400000  
            00374BAE    2B85 D2434000   SUB     EAX, DWORD PTR SS:\[EBP+<Reloc\_BASE>\]                    　　; 相减之后 eax 就是原 iat 的地址  
            00374BB4    8B00            MOV     EAX, DWORD PTR DS:\[EAX\]                                     ; 取出 IAT 中第一层的加密地址  
            00374BB6    894424 20       MOV     DWORD PTR SS:\[ESP+20\], EAX  
            00374BBA    61              POPAD  
            00374BBB    C3              RETN  
            00374BBC    8B46 04         MOV     EAX, DWORD PTR DS:\[ESI+4\]                                   ; 这里对 jmp \[address\]的处理  
            00374BBF    0385 F6434000   ADD     EAX, DWORD PTR SS:\[EBP+<UnLock\_Important\_Key>\]  
            00374BC5    03BD D2434000   ADD     EDI, DWORD PTR SS:\[EBP+<Reloc\_BASE>\]                   　　 ; 00400000  
            00374BCB    2BBD B2434000   SUB     EDI, DWORD PTR SS:\[EBP+<IMGBASE>\]                      　　 ; 00400000  
            00374BD1    2BC7            SUB     EAX, EDI  
            00374BD3    F7D0            NOT     EAX  
            00374BD5    C1C0 10         ROL     EAX, 10  
            00374BD8    0385 B2434000   ADD     EAX, DWORD PTR SS:\[EBP+<IMGBASE>\]                   　　    ; 00400000  
            00374BDE    2B85 D2434000   SUB     EAX, DWORD PTR SS:\[EBP+<Reloc\_BASE>\]         　　           ; 减了之后算出 jmp \[address\]中 address 的地址  
            00374BE4    8B00            MOV     EAX, DWORD PTR DS:\[EAX\]                                     ; 取出 IAT 中第一层的加密地址  
            00374BE6    894424 24       MOV     DWORD PTR SS:\[ESP+24\], EAX  
            00374BEA    61              POPAD  
            00374BEB    83C4 04         ADD     ESP, 4                                                      ; 因为是 jmp \[address\]所以这里要 add esp,4  
            00374BEE    C3              RETN  
  
  
  
proc\_Loaddll\_failed:  
  
    00374BEF >  56              PUSH    ESI                                                         ; proc\_Loaddll\_failed  
    00374BF0    8D85 5B484000   LEA     EAX, DWORD PTR SS:\[EBP+40485B\]                              ; ASCII "can not found %s"  
    00374BF6    50              PUSH    EAX  
    00374BF7    8D85 74484000   LEA     EAX, DWORD PTR SS:\[EBP+<strAPIName>\]            　　        ; ASCII "RtlSetLastWin32Error"  
    00374BFD    50              PUSH    EAX  
    00374BFE    8D85 2D354000   LEA     EAX, DWORD PTR SS:\[EBP+40352D\]  
    00374C04    50              PUSH    EAX  
    00374C05    8B85 2A444000   MOV     EAX, DWORD PTR SS:\[EBP+<APIwsPrintfA>\]         　　         ; USER32.wsprintfA  
    00374C0B  ^ E9 CAFDFFFF     JMP     <proc\_Run\_FUN>  
    00374C10    90              NOP  
    00374C11    83C4 0C         ADD     ESP, 0C  
    00374C14    6A 00           PUSH    0  
    00374C16    8D85 A4484000   LEA     EAX, DWORD PTR SS:\[EBP+4048A4\]                              ; ASCII "warning"  
    00374C1C    50              PUSH    EAX  
    00374C1D    8D85 74484000   LEA     EAX, DWORD PTR SS:\[EBP+<strAPIName>\]  
    00374C23    50              PUSH    EAX  
    00374C24    6A 00           PUSH    0  
    00374C26    8D85 55354000   LEA     EAX, DWORD PTR SS:\[EBP+403555\]  
    00374C2C    50              PUSH    EAX  
    00374C2D    8B85 35444000   MOV     EAX, DWORD PTR SS:\[EBP+<APIMsgBox>\]             　　        ; USER32.MessageBoxA  
    00374C33  ^ E9 A2FDFFFF     JMP     <proc\_Run\_FUN>　  
    00374C38    90              NOP  
    00374C39    E9 00160000     JMP     <Game\_Over>  
  
  
proc\_check\_CC:  
  
    00374C3E >  56              PUSH    ESI                                                         ; proc\_check\_CC  
    00374C3F    51              PUSH    ECX                                                         ; 检测 API 是否下了 cc 断点  
    00374C40    50              PUSH    EAX  
    00374C41    8BF0            MOV     ESI, EAX  
    00374C43    B9 01000000     MOV     ECX, 1  
    00374C48    AC              LODS    BYTE PTR DS:\[ESI\]  
    00374C49    3C CC           CMP     AL, 0CC  
    00374C4B    75 08           JNZ     SHORT 00374C55  
    00374C4D    58              POP     EAX                                                         ; 0012FFE0  
    00374C4E    59              POP     ECX                                                         ; 0012FFE0  
    00374C4F    5E              POP     ESI                                                         ; 0012FFE0  
    00374C50    E9 E9150000     JMP     <Game\_Over>  
    00374C55  ^ E2 F1           LOOPD   SHORT 00374C48  
    00374C57    58              POP     EAX                                                         ; 0012FFE0  
    00374C58    59              POP     ECX                                                         ; 0012FFE0  
    00374C59    5E              POP     ESI                                                         ; 0012FFE0  
    00374C5A    C3              RETN  
  
Calculate\_CRC:  
  
    00374C5B >  83CA FF         OR      EDX, FFFFFFFF                                               ; Calculate\_CRC  
    00374C5E    51              PUSH    ECX  
    00374C5F    AC              LODS    BYTE PTR DS:\[ESI\]  
    00374C60    32C2            XOR     AL, DL  
    00374C62    6A 08           PUSH    8  
    00374C64    59              POP     ECX                                                         ; 0012FFE0  
    00374C65    0FB6D8          MOVZX   EBX, AL  
    00374C68    D1EB            SHR     EBX, 1  
    00374C6A    73 06           JNB     SHORT 00374C72  
    00374C6C    81F3 2083B8ED   XOR     EBX, EDB88320  
    00374C72  ^ E2 F4           LOOPD   SHORT 00374C68  
    00374C74    C1EA 08         SHR     EDX, 8  
    00374C77    33D3            XOR     EDX, EBX  
    00374C79    59              POP     ECX                                                         ; 0012FFE0  
    00374C7A  ^ E2 E2           LOOPD   SHORT 00374C5E  
    00374C7C    F7D2            NOT     EDX  
    00374C7E    92              XCHG    EAX, EDX  
    00374C7F    C3              RETN  
  
Game\_Over:  
    0037623E    8B85 CE434000   MOV     EAX, DWORD PTR SS:\[EBP+4043CE\]                              ; Game\_Over  
    00376244    85C0            TEST    EAX, EAX  
    00376246    74 07           JE      SHORT 0037624F  
    0376248    61              POPAD  
    00376249    B8 00000000     MOV     EAX, 0  
    0037624E    C3              RETN  
    0037624F    6A 00           PUSH    0  
    00376251    6A 00           PUSH    0  
    00376253    FFB5 D6444000   PUSH    DWORD PTR SS:\[EBP+<727.APIExitProcess>\]                     ; kernel32.ExitProcess  
    00376259    8D8D 834B4000   LEA     ECX, DWORD PTR SS:\[EBP+404B83\]  
    0037625F    8DBD A01A4000   LEA     EDI, DWORD PTR SS:\[EBP+<727.Crc\_Start\_addr>\]  
    00376265    2BCF            SUB     ECX, EDI  
    00376267    33C0            XOR     EAX, EAX  
    00376269    F3:AA           REP     STOS BYTE PTR ES:\[EDI\]  
    0037626B    AB              STOS    DWORD PTR ES:\[EDI\]  
    0037626C    C3              RETN  
  
  
  
  
Greetz:  
 Fly.Jingulong,yock,tDasm.David.hexer,hmimys,ahao.UFO(brother).alan(sister).all of my   
  
friends and you!  
  
   
谨此献给我爱的文，love you every day!   
          By loveboom\[DFCG\]\[FCG\]\[US\]  
                      http://blog.csdn.net/bmd2chen  
                  
  
                                        Email:loveboom#163.com 

 4dd466d0e1e0cc123afbe504da2462ca