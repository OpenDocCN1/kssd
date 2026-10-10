

# 1st Security Agent V6.4 

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-328-1.htm`](https://www.pediy.com/kssd/pediy08/pediy8-328-1.htm)

【文章标题】: 1st Security Agent V6.4 算法分析 (容易)  
【文章作者】: \[DCG\]PeerLessSoul  
【作者邮箱】: PeerLessSoul@Gmail.com  
【作者主页】: http://114989149.qzone.qq.com  
【作者 QQ 号】: 114989149  
【软件名称】: 1st Security Agent   
【软件大小】: 1.08M  
【下载地址】: www.download.com  
【保护方式】: 注册码验证  
【使用工具】: Peid,Ollice  
【作者声明】: 本人小菜，请大侠们不要见笑……  
最近闲逛 Download 发现一款用来保护电脑安全的软件 1st Security Agent ，主要功能和作用就让一些 windows 自带的程序运行的时候需要密码否则就不能使用！安装好程序……使用 Peid 查看为 Borland Delphi 6.0 - 7.0.就直接使用 ollice 了，载入程序！下断 user32.MessageBoxA,OK! F9 走！输入伪码 1234567890 点 reg!程序被断下在这里！  
\-----------------------------------------------------------------------------------------------------------------------------  
77D504EC  /.  55            push    ebp                                          ；我们被断在这里！F8 走！  
77D504ED  |.  8BEC          mov     ebp, esp  
77D504EF  |.  833D BC04D777>cmp     dword ptr \[77D704BC\], 0  
77D504F6  |.  74 24         je      short 77D5051C  
77D504F8  |.  64:A1 1800000>mov     eax, fs:\[18\]  
77D504FE  |.  6A 00         push    0  
77D50500  |.  FF70 24       push    dword ptr \[eax+24\]  
77D50503  |.  68 240BD777   push    77D70B24  
77D50508  |.  FF15 C812D177 call    \[<&KERNEL32.InterlockedCompareEx>;  kernel32.InterlockedCompareExchange  
77D5050E  |.  85C0          test    eax, eax  
77D50510  |.  75 0A         jnz     short 77D5051C  
77D50512  |.  C705 200BD777>mov     dword ptr \[77D70B20\], 1  
77D5051C  |>  6A 00         push    0                                ; /LanguageID = 0 (LANG\_NEUTRAL)  
77D5051E  |.  FF75 14       push    dword ptr \[ebp+14\]               ; |Style  
77D50521  |.  FF75 10       push    dword ptr \[ebp+10\]               ; |Title  
77D50524  |.  FF75 0C       push    dword ptr \[ebp+C\]                ; |Text  
77D50527  |.  FF75 08       push    dword ptr \[ebp+8\]                ; |hOwner  
77D5052A  |.  E8 2D000000   call    MessageBoxExA                    ; \\MessageBoxExA 这个就是出错的提示地方！  
77D5052F  |.  5D            pop     ebp                              ;  0012F3E8  
77D50530  \\.  C2 1000       retn    10  
\-----------------------------------------------------------------------------------------------------------------------------  
从上面的 retn 退出到这里！  
\-----------------------------------------------------------------------------------------------------------------------------  
0048A66E  |.  E8 95CEF7FF   call    <jmp.&user32.MessageBoxA>        ; 这里是叫出错提示！  
0048A673  |.  8945 F8       mov     \[ebp-8\], eax                     ；从上面 retn 处来到这里！  
0048A676  |.  33C0          xor     eax, eax  
\----------------------------------------------------------------------------------------------------------------------------往上下看看……发现上面几个 Je 不不能跳过这个 MessageBoxA,那么就意味着，这一小段已经是注册错误时候必然经过的程序断！我们就想，可能再外面吧！外面有个判断注册的正确与否，然后错误就走我们现在在的这段程序段！我们就往下走～找到 retn 跳出了段程序……我们来到！  
\----------------------------------------------------------------------------------------------------------------------------004995E4  /$  55      push    ebp                          ;我们在这里下断！！！  
004995E5  |.  8BEC          mov     ebp, esp  
004995E7  |.  83C4 F8       add     esp, -8  
004995EA  |.  33D2          xor     edx, edx  
004995EC  |.  8955 F8       mov     \[ebp-8\], edx  
004995EF  |.  8945 FC       mov     \[ebp-4\], eax  
004995F2  |.  8B45 FC       mov     eax, \[ebp-4\]  
004995F5  |.  E8 66B6F6FF   call    00404C60  
004995FA  |.  33C0          xor     eax, eax  
004995FC  |.  55            push    ebp  
004995FD  |.  68 52964900   push    00499652  
00499602  |.  64:FF30       push    dword ptr fs:\[eax\]  
00499605  |.  64:8920       mov     fs:\[eax\], esp  
00499608  |.  6A 40         push    40  
0049960A  |.  8D55 F8       lea     edx, \[ebp-8\]  
0049960D  |.  B8 68964900   mov     eax, 00499668                ;  ASCII "L\_INFORMATION"  
00499612  |.  E8 51F0FFFF   call    00498668  
00499617  |.  8B45 F8       mov     eax, \[ebp-8\]  
0049961A  |.  E8 51B6F6FF   call    00404C70  
0049961F  |.  50            push    eax  
00499620  |.  8B45 FC       mov     eax, \[ebp-4\]  
00499623  |.  E8 48B6F6FF   call    00404C70  
00499628  |.  8BD0          mov     edx, eax  
0049962A  |.  A1 DCCF4C00   mov     eax, \[4CCFDC\]  
0049962F  |.  8B00          mov     eax, \[eax\]  
00499631  |.  59            pop     ecx  
00499632  |.  E8 510FFFFF   call    0048A588         ;  这个就是 Call 就是 CAll 我们注册错误的那段代码  
00499637  |.  33C0          xor     eax, eax                ;  我们往上一个 retn 出来到这里！  
00499639  |.  5A            pop     edx  
0049963A  |.  59            pop     ecx  
0049963B  |.  59            pop     ecx  
0049963C  |.  64:8910       mov     fs:\[eax\], edx  
0049963F  |.  68 59964900   push    00499659  
00499644  |>  8D45 F8       lea     eax, \[ebp-8\]  
00499647  |.  BA 02000000   mov     edx, 2  
0049964C  |.  E8 83B1F6FF   call    004047D4  
00499651  \\.  C3            retn  
\----------------------------------------------------------------------------------------------------------------------------  
我们取消先前的断点，而在 004995E4 下断！！！也就是这段代码的开头地方……再一次运行程序！注册～这下就被断在了 004995E4 处～我们 F8 慢慢来……注意看右边的那些寄存器值.就在被断下的一瞬间，我们的寄存器值如下  
EAX 012902D4 ASCII "This registration code is invalid"  
ECX FFFFFFFE  
EDX 004A8348 ASCII "M\_BADCODE"  
就看这几个了……你看，已经出现了 This registration code is invalid,这个又有什么意思？这个表示……这段代码还是为注册出错以后提示做准备，注册验证的代码段还在外面……我们在走到 00499651 处 retn 出来！到这里！  
\----------------------------------------------------------------------------------------------------------------------------  
004A821E  |.  50            push    eax  
004A821F  |.  8D55 D8       lea     edx, \[ebp-28\]  
004A8222  |.  B8 CC824A00   mov     eax, 004A82CC                ;  ASCII "BF8A83B980"  
004A8227  |.  E8 304DFEFF   call    0048CF5C  
004A822C  |.  8B45 D8       mov     eax, \[ebp-28\]  
004A822F  |.  50            push    eax  
004A8230  |.  8D45 D4       lea     eax, \[ebp-2C\]  
004A8233  |.  50            push    eax  
004A8234  |.  B8 E0824A00   mov     eax, 004A82E0                ;  ASCII"BE828B999A8C9F88B1A0848E9F829E828B99B1A9BFA0BFBEB5"  
004A8239  |.  5A            pop     edx  
004A823A  |.  E8 1D4DFEFF   call    0048CF5C  
004A823F  |.  8B55 D4       mov     edx, \[ebp-2C\]  
004A8242  |.  A1 54ED4C00   mov     eax, \[4CED54\]  
004A8247  |.  59            pop     ecx  
004A8248  |.  E8 8781FEFF   call    004903D4  
004A824D  |>  8D55 D0       lea     edx, \[ebp-30\]  
004A8250  |.  B8 34834A00   mov     eax, 004A8334                ;  ASCII "M\_THANKS"  
004A8255  |.  E8 0E04FFFF   call    00498668  
004A825A  |.  8B45 D0       mov     eax, \[ebp-30\]  
004A825D  |.  E8 8213FFFF   call    004995E4  
004A8262  |.  EB 1F         jmp     short 004A8283               ;跳过了注册失败提示  
004A8264  |>  B8 4CED4C00   mov     eax, 004CED4C  
004A8269  |.  E8 42C5F5FF   call    004047B0  
004A826E  |.  8D55 CC       lea     edx, \[ebp-34\]  
004A8271  |.  B8 48834A00   mov     eax, 004A8348                ;  ASCII "M\_BADCODE"  
004A8276  |.  E8 ED03FFFF   call    00498668  
004A827B  |.  8B45 CC       mov     eax, \[ebp-34\]  
004A827E  |.  E8 6113FFFF   call    004995E4                      ;注册失败提示框  
004A8283  |>  33C0          xor     eax, eax　　　　　　　　　　　 ;  又出来到这里！　　  
004A8285  |.  5A            pop     edx  
004A8286  |.  59            pop     ecx  
004A8287  |.  59            pop     ecx  
004A8288  |.  64:8910       mov     fs:\[eax\], edx  
004A828B  |.  68 BA824A00   push    004A82BA  
004A8290  |>  8D45 CC       lea     eax, \[ebp-34\]  
004A8293  |.  BA 0A000000   mov     edx, 0A  
004A8298  |.  E8 37C5F5FF   call    004047D4  
004A829D  |.  8D45 F4       lea     eax, \[ebp-C\]  
004A82A0  |.  E8 0BC5F5FF   call    004047B0  
004A82A5  |.  8D45 F8       lea     eax, \[ebp-8\]  
004A82A8  |.  BA 02000000   mov     edx, 2  
004A82AD  |.  E8 22C5F5FF   call    004047D4  
004A82B2  \\.  C3            retn  
\----------------------------------------------------------------------------------------------------------------------------  
我们从上面的 retn 出来到 04A8283,这里又是新的一个程序段……我们往上看……那么出错提示框就是在 004A827E 这个 Call,上面有个 Jmp 跳过了注册失败提示！但是在看看……004A8264  |>  这里有个接受的跳转……说明如果上面有个跳转，如果哪个跳转不实现，那么就有可能经过 Jmp 从而跳过了注册失败的提示！我们就右击 004A8264  选择转到……我们转到这里……  
\-----------------------------------------------------------------------------------------------------------------------------  
004A8134  /$  55            push    ebp  
004A8135  |.  8BEC          mov     ebp, esp  
004A8137  |.  B9 06000000   mov     ecx, 6  
004A813C  |>  6A 00         /push    0  
004A813E  |.  6A 00         |push    0  
004A8140  |.  49            |dec     ecx  
004A8141  |.^ 75 F9         \\jnz     short 004A813C  
004A8143  |.  51            push    ecx  
004A8144  |.  53            push    ebx  
004A8145  |.  56            push    esi  
004A8146  |.  8BF0          mov     esi, eax  
004A8148  |.  33C0          xor     eax, eax  
004A814A  |.  55            push    ebp  
004A814B  |.  68 B3824A00   push    004A82B3  
004A8150  |.  64:FF30       push    dword ptr fs:\[eax\]  
004A8153  |.  64:8920       mov     fs:\[eax\], esp  
004A8156  |.  8D55 F4       lea     edx, \[ebp-C\]  
004A8159  |.  8B86 48030000 mov     eax, \[esi+348\]  
004A815F  |.  E8 5011FCFF   call    004692B4  
004A8164  |.  8B45 F4       mov     eax, \[ebp-C\]                 ;  取出伪码  
004A8167  |.  8D55 F8       lea     edx, \[ebp-8\]  
004A816A  |.  E8 6139FEFF   call    0048BAD0  
004A816F  |.  8B55 F8       mov     edx, \[ebp-8\]  
004A8172  |.  B8 4CED4C00   mov     eax, 004CED4C  
004A8177  |.  E8 88C6F5FF   call    00404804  
004A817C  |.  E8 FBFDFFFF   call    004A7F7C                     ;这里就是关键 CAll  
004A8181  |.  8BD8          mov     ebx, eax  
004A8183  |.  84DB          test    bl, bl  
004A8185  |.  0F84 D9000000 je      004A8264                     ;这个 je 关键  
\-----------------------------------------------------------------------------------------------------------------------------我们取消先前的全部断点……在 004A814A 下断……F8 走～一下……发现关键 Call 跟进……到这里……  
\-----------------------------------------------------------------------------------------------------------------------------  
004A7F7C  /$  53            push    ebx  
004A7F7D  |.  56            push    esi  
004A7F7E  |.  57            push    edi  
004A7F7F  |.  BF 4CED4C00   mov     edi, 004CED4C  
004A7F84  |.  33F6          xor     esi, esi  
004A7F86  |.  33DB          xor     ebx, ebx  
004A7F88  |.  8B07          mov     eax, \[edi\]  
004A7F8A  |.  E8 E1CAF5FF   call    00404A70  
004A7F8F  |.  83F8 0E       cmp     eax, 0E                       ;注册码长度与 14 比较，不是 14 不行！  
004A7F92  |.  75 67         jnz     short 004A7FFB  
004A7F94  |.  8B07          mov     eax, \[edi\]  
004A7F96  |.  8038 36       cmp     byte ptr \[eax\], 36           ;  第 1 位与 6 比  
004A7F99  |.  0F94C0        sete    al                           ；如果相同就为 true!以下类似  
004A7F9C  |.  83E0 7F       and     eax, 7F  
004A7F9F  |.  03F0          add     esi, eax  
004A7FA1  |.  8B07          mov     eax, \[edi\]  
004A7FA3  |.  8078 02 32    cmp     byte ptr \[eax+2\], 32         ;  第 3 位与字符 2 比  
004A7FA7  |.  0F94C0        sete    al  
004A7FAA  |.  83E0 7F       and     eax, 7F  
004A7FAD  |.  03F0          add     esi, eax  
004A7FAF  |.  8B07          mov     eax, \[edi\]  
004A7FB1  |.  8078 03 31    cmp     byte ptr \[eax+3\], 31         ;  第 4 位与字符 1 比  
004A7FB5  |.  0F94C0        sete    al  
004A7FB8  |.  83E0 7F       and     eax, 7F  
004A7FBB  |.  03F0          add     esi, eax  
004A7FBD  |.  8B07          mov     eax, \[edi\]  
004A7FBF  |.  8078 04 39    cmp     byte ptr \[eax+4\], 39         ;  第 5 位与字符 9 比  
004A7FC3  |.  0F94C0        sete    al  
004A7FC6  |.  83E0 7F       and     eax, 7F  
004A7FC9  |.  03F0          add     esi, eax  
004A7FCB  |.  8B07          mov     eax, \[edi\]  
004A7FCD  |.  8078 07 30    cmp     byte ptr \[eax+7\], 30         ;  第 8 位与字符 0 比  
004A7FD1  |.  0F94C0        sete    al  
004A7FD4  |.  83E0 7F       and     eax, 7F  
004A7FD7  |.  03F0          add     esi, eax  
004A7FD9  |.  8B07          mov     eax, \[edi\]  
004A7FDB  |.  8078 08 35    cmp     byte ptr \[eax+8\], 35         ;  第 9 位与字符 5 比  
004A7FDF  |.  0F94C0        sete    al  
004A7FE2  |.  83E0 7F       and     eax, 7F  
004A7FE5  |.  03F0          add     esi, eax  
004A7FE7  |.  8B07          mov     eax, \[edi\]  
004A7FE9  |.  8078 0A 33    cmp     byte ptr \[eax+A\], 33         ;  第 11 位与字符 3 比  
004A7FED  |.  0F94C0        sete    al  
004A7FF0  |.  83E0 7F       and     eax, 7F  
004A7FF3  |.  03F0          add     esi, eax  
004A7FF5  |.  83FE 07       cmp     esi, 7  
004A7FF8  |.  0F94C3        sete    bl  
004A7FFB  |>  8BC3          mov     eax, ebx  
004A7FFD  |.  5F            pop     edi  
004A7FFE  |.  5E            pop     esi  
004A7FFF  |.  5B            pop     ebx  
004A8000  \\.  C3            retn  
\----------------------------------------------------------------------------------------------------------------------------  
当七个比较都吻合时…004A8185 不实现……注册成功！   
注册算法总结：  
1.注册码长度为 14  
2.注册码格式为 6?219??05?3???  
  
  
还有他们公司其他软件,也大同小异.就顺便走一下算法了!  
  
我写了个注册机…其中包括  
  
 "1st Security Agent V6.4"  
  
 "Access Controller V3.1"  
  
"Best Network Security  V1.6"  
  
"Hide Files and Folders V2.5"  
  
 "Pc Security Tweaker V6.5"  
  
 "Bulletproof Public PC V3.3"  
  
"Outlook Express Privacy  V1.7"  
  
大家可以试试看看…… 

 4dd466d0e1e0cc123afbe504da2462ca