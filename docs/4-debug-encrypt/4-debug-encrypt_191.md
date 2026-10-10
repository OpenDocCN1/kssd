

# Base64 算法分析＋VC 注册机代码【原创】

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-811.htm`](https://www.pediy.com/kssd/pediy08/pediy8-811.htm)

【破文标题】Base64 算法分析＋VC 注册机代码  
【破文作者】Vcsoft  
【破解工具】OD  
【破解平台】WINXP  
【软件简介】生活小管家 1.0 版,就当是 CrackMe 了,只是感兴趣，没有其他目的。  
  
            失误之处敬请诸位大侠赐教!  
  
  
下断 bp GetWindowTextA  
填入试验码 9876543210654321 点注册 中断在 USER32 领空 CTRL＋F9 返回程序领空  
  
00436E05  |.  8B4D 10       mov     ecx, \[ebp+10\]  
00436E08  |.  6A FF         push    -1  
00436E0A  |.  E8 A295FDFF   call    004103B1  
00436E0F  |. /EB 0B         jmp     short 00436E1C  
00436E11  |> |8B45 10       mov     eax, \[ebp+10\]  
00436E14  |. |FF30          push    dword ptr \[eax\]                  ; /Arg2  
00436E16  |. |56            push    esi                              ; |Arg1  
00436E17  |. |E8 16F6FFFF   call    00436432                         ; \\CrackMe1.00436432  
00436E1C  |> \\5F            pop     edi                              ;  0012ED0C  
00436E1D  |.  5E            pop     esi  
00436E1E  |.  5D            pop     ebp  
00436E1F  \\.  C2 0C00       retn    0C  
\------------------------------------------------------------------------------------  
......省去非关键代码，跟着程序流程走，来到下面  
  
0040EA63   .  8B46 74       mov     eax, \[esi+74\]  
0040EA66   .  8378 F4 10    cmp     dword ptr \[eax-C\], 10            ;  比较试验码是不是 16 位  
0040EA6A   .  8D7E 74       lea     edi, \[esi+74\]  
0040EA6D   .  74 24         je      short 0040EA93                   ;  是则跳往计算注册码  
0040EA6F   .  6A 00         push    0  
0040EA71   .  68 C8F64300   push    0043F6C8  
0040EA76   .  68 FC104400   push    004410FC  
0040EA7B   .  8BCE          mov     ecx, esi  
0040EA7D   .  E8 85120200   call    0042FD07  
0040EA82   .  5F            pop     edi  
0040EA83   .  5E            pop     esi  
0040EA84   .  8B4C24 0C     mov     ecx, \[esp+C\]  
0040EA88   .  64:890D 00000>mov     fs:\[0\], ecx  
0040EA8F   .  83C4 18       add     esp, 18  
0040EA92   .  C3            retn  
0040EA93   >  55            push    ebp  
0040EA94   .  E8 75030200   call    0042EE0E  
0040EA99   .  8B10          mov     edx, \[eax\]  
0040EA9B   .  8BC8          mov     ecx, eax  
0040EA9D   .  FF52 0C       call    \[edx+C\]  
0040EAA0   .  83C0 10       add     eax, 10  
0040EAA3   .  894424 10     mov     \[esp+10\], eax  
0040EAA7   .  8D4424 10     lea     eax, \[esp+10\]  
0040EAAB   .  50            push    eax                              ; /Arg2  
0040EAAC   .  8D6E 70       lea     ebp, \[esi+70\]                    ; |  
0040EAAF   .  55            push    ebp                              ; |Arg1  
0040EAB0   .  8D4C24 1C     lea     ecx, \[esp+1C\]                    ; |  
0040EAB4   .  C74424 28 000>mov     dword ptr \[esp+28\], 0            ; |  
0040EABC   .  E8 0F77FFFF   call    004061D0                         ; \\CrackMe1.004061D0  
\---------------------------------------------------------------------; 此处关键 CALL，跟入  
  
004061D0  /$  6A FF         push    -1  
004061D2  |.  68 B8C64300   push    0043C6B8                         ;  SE 处理程序安装  
004061D7  |.  64:A1 0000000>mov     eax, fs:\[0\]  
004061DD  |.  50            push    eax  
004061DE  |.  64:8925 00000>mov     fs:\[0\], esp  
004061E5  |.  83EC 08       sub     esp, 8  
004061E8  |.  53            push    ebx  
004061E9  |.  8B5C24 1C     mov     ebx, \[esp+1C\]  
004061ED  |.  8B03          mov     eax, \[ebx\]  
004061EF  |.  55            push    ebp  
004061F0  |.  8B68 F4       mov     ebp, \[eax-C\]                     ;  ebp = 机器码长度  
004061F3  |.  85ED          test    ebp, ebp  
004061F5  |.  894C24 0C     mov     \[esp+C\], ecx  
004061F9  |.  7F 15         jg      short 00406210  
004061FB  |.  5D            pop     ebp  
004061FC  |.  32C0          xor     al, al  
004061FE  |.  5B            pop     ebx  
004061FF  |.  8B4C24 08     mov     ecx, \[esp+8\]  
00406203  |.  64:890D 00000>mov     fs:\[0\], ecx  
0040620A  |.  83C4 14       add     esp, 14  
0040620D  |.  C2 0800       retn    8  
00406210  |>  56            push    esi  
00406211  |.  57            push    edi  
00406212  |.  8D7D 01       lea     edi, \[ebp+1\]  
00406215  |.  57            push    edi  
00406216  |.  E8 988B0200   call    0042EDB3  
0040621B  |.  8BF0          mov     esi, eax  
0040621D  |.  8BCF          mov     ecx, edi  
0040621F  |.  8BD1          mov     edx, ecx  
00406221  |.  C1E9 02       shr     ecx, 2  
00406224  |.  33C0          xor     eax, eax  
00406226  |.  8BFE          mov     edi, esi  
00406228  |.  F3:AB         rep     stos dword ptr es:\[edi\]  
0040622A  |.  8BCA          mov     ecx, edx  
0040622C  |.  83E1 03       and     ecx, 3  
0040622F  |.  F3:AA         rep     stos byte ptr es:\[edi\]  
00406231  |.  8B03          mov     eax, \[ebx\]                       ;  eax = 机器码地址  
00406233  |.  55            push    ebp  
00406234  |.  50            push    eax  
00406235  |.  56            push    esi  
00406236  |.  E8 15800100   call    0041E250  
0040623B  |.  83C4 10       add     esp, 10  
0040623E  |.  33C9          xor     ecx, ecx                         ;  cl = 0 循环计数器  
00406240  |.  85ED          test    ebp, ebp  
00406242  |.  7E 1C         jle     short 00406260  
00406244  |>  8A0431        /mov     al, \[ecx+esi\]                   ;  al = 机器码各字符 ASCII 码-----------  
00406247  |.  02C1          |add     al, cl                          ;  al = al + cl                      |  
00406249  |.  8AD1          |mov     dl, cl                          ;  dl = cl                           |  
0040624B  |.  FEC0          |inc     al                              ;  al++                              |  
0040624D  |.  C0E2 02       |shl     dl, 2                           ;  dl 左移 2 位                         |  
00406250  |.  32C2          |xor     al, dl                          ;  al^=dl                            |  
00406252  |.  3C 30         |cmp     al, 30                          ;  al 是否等于 0x30                运算 1  
00406254  |.  75 02         |jnz     short 00406258                  ;  不等则跳                          |  
00406256  |.  B0 65         |mov     al, 65                          ;  al = 0x30 则再加上 0x65            |  
00406258  |>  880431        |mov     \[ecx+esi\], al                   ;  保存结果                          |  
0040625B  |.  41            |inc     ecx                             ;  cl++                              |  
0040625C  |.  3BCD          |cmp     ecx, ebp                        ;  cl < 机器码长                     |  
0040625E  |.^ 7C E4         \\jl      short 00406244                  ;  循环------------------------------|  
00406260  |> \\56            push    esi  
00406261  |.  8D4C24 2C     lea     ecx, \[esp+2C\]  
00406265  |.  E8 96C8FFFF   call    00402B00  
0040626A  |.  8D4424 28     lea     eax, \[esp+28\]  
0040626E  |.  50            push    eax  
0040626F  |.  8D4C24 14     lea     ecx, \[esp+14\]  
00406273  |.  51            push    ecx  
00406274  |.  8B4C24 1C     mov     ecx, \[esp+1C\]  
00406278  |.  C74424 28 000>mov     dword ptr \[esp+28\], 0  
00406280  |.  E8 BBFBFFFF   call    00405E40  
  
00405E40  /$  6A FF         push    -1  
00405E42  |.  68 58C64300   push    0043C658                         ;  SE 处理程序安装  
00405E47  |.  64:A1 0000000>mov     eax, fs:\[0\]  
00405E4D  |.  50            push    eax  
00405E4E  |.  64:8925 00000>mov     fs:\[0\], esp  
00405E55  |.  51            push    ecx  
00405E56  |.  53            push    ebx  
00405E57  |.  C74424 04 000>mov     dword ptr \[esp+4\], 0  
00405E5F  |.  55            push    ebp  
00405E60  |.  8B6C24 20     mov     ebp, \[esp+20\]  
00405E64  |.  8B45 00       mov     eax, \[ebp\]  
00405E67  |.  8B58 F4       mov     ebx, \[eax-C\]  
00405E6A  |.  85DB          test    ebx, ebx  
00405E6C  |.  56            push    esi  
00405E6D  |.  7F 26         jg      short 00405E95  
00405E6F  |.  8B7424 20     mov     esi, \[esp+20\]  
00405E73  |.  68 F8FA4300   push    0043FAF8  
00405E78  |.  8BCE          mov     ecx, esi  
00405E7A  |.  E8 81CCFFFF   call    00402B00  
00405E7F  |.  8BC6          mov     eax, esi  
00405E81  |.  5E            pop     esi  
00405E82  |.  5D            pop     ebp  
00405E83  |.  5B            pop     ebx  
00405E84  |.  8B4C24 04     mov     ecx, \[esp+4\]  
00405E88  |.  64:890D 00000>mov     fs:\[0\], ecx  
00405E8F  |.  83C4 10       add     esp, 10  
00405E92  |.  C2 0800       retn    8  
00405E95  |>  8D0C9D 000000>lea     ecx, \[ebx\*4\]  
00405E9C  |.  B8 56555555   mov     eax, 55555556  
00405EA1  |.  F7E9          imul    ecx  
00405EA3  |.  8BCA          mov     ecx, edx  
00405EA5  |.  C1E9 1F       shr     ecx, 1F  
00405EA8  |.  03CA          add     ecx, edx  
00405EAA  |.  8BC3          mov     eax, ebx  
00405EAC  |.  99            cdq  
00405EAD  |.  BE 03000000   mov     esi, 3  
00405EB2  |.  F7FE          idiv    esi  
00405EB4  |.  BE 4C000000   mov     esi, 4C  
00405EB9  |.  03CA          add     ecx, edx  
00405EBB  |.  8BC1          mov     eax, ecx  
00405EBD  |.  99            cdq  
00405EBE  |.  F7FE          idiv    esi  
00405EC0  |.  85D2          test    edx, edx  
00405EC2  |.  74 18         je      short 00405EDC  
00405EC4  |.  81E2 03000080 and     edx, 80000003  
00405ECA  |.  79 05         jns     short 00405ED1  
00405ECC  |.  4A            dec     edx  
00405ECD  |.  83CA FC       or      edx, FFFFFFFC  
00405ED0  |.  42            inc     edx  
00405ED1  |>  74 09         je      short 00405EDC  
00405ED3  |.  BE 04000000   mov     esi, 4  
00405ED8  |.  2BF2          sub     esi, edx  
00405EDA  |.  03CE          add     ecx, esi  
00405EDC  |>  57            push    edi  
00405EDD  |.  8D7C41 02     lea     edi, \[ecx+eax\*2+2\]  
00405EE1  |.  85FF          test    edi, edi  
00405EE3  |.  897C24 10     mov     \[esp+10\], edi  
00405EE7  |.  0F8E BB000000 jle     00405FA8  
00405EED  |.  57            push    edi  
00405EEE  |.  E8 C08E0200   call    0042EDB3  
00405EF3  |.  8BCF          mov     ecx, edi  
00405EF5  |.  8BD1          mov     edx, ecx  
00405EF7  |.  8BF0          mov     esi, eax  
00405EF9  |.  C1E9 02       shr     ecx, 2  
00405EFC  |.  33C0          xor     eax, eax  
00405EFE  |.  8BFE          mov     edi, esi  
00405F00  |.  F3:AB         rep     stos dword ptr es:\[edi\]  
00405F02  |.  8BCA          mov     ecx, edx  
00405F04  |.  83E1 03       and     ecx, 3  
00405F07  |.  83C4 04       add     esp, 4  
00405F0A  |.  F3:AA         rep     stos byte ptr es:\[edi\]  
00405F0C  |.  E8 FD8E0200   call    0042EE0E  
00405F11  |.  8B10          mov     edx, \[eax\]  
00405F13  |.  8BC8          mov     ecx, eax  
00405F15  |.  FF52 0C       call    \[edx+C\]  
00405F18  |.  83C0 10       add     eax, 10  
00405F1B  |.  894424 28     mov     \[esp+28\], eax  
00405F1F  |.  8B45 00       mov     eax, \[ebp\]  
00405F22  |.  6A 00         push    0  
00405F24  |.  8D4C24 14     lea     ecx, \[esp+14\]  
00405F28  |.  51            push    ecx  
00405F29  |.  56            push    esi  
00405F2A  |.  53            push    ebx  
00405F2B  |.  50            push    eax  
00405F2C  |.  C74424 30 000>mov     dword ptr \[esp+30\], 0  
00405F34  |.  E8 A7FBFFFF   call    00405AE0                         ;  Base64 加密 CALL？  
\------------------------------------------------------------------------跟进去看看就知道了  
  
00405AE0  /$  83EC 10       sub     esp, 10  
00405AE3  |.  53            push    ebx  
00405AE4  |.  56            push    esi  
00405AE5  |.  8B7424 1C     mov     esi, \[esp+1C\]  
00405AE9  |.  85F6          test    esi, esi  
00405AEB  |.  57            push    edi  
00405AEC  |.  0F84 C5010000 je      00405CB7  
00405AF2  |.  8B7C24 28     mov     edi, \[esp+28\]  
00405AF6  |.  85FF          test    edi, edi  
00405AF8  |.  0F84 B9010000 je      00405CB7  
00405AFE  |.  8B5C24 2C     mov     ebx, \[esp+2C\]  
00405B02  |.  85DB          test    ebx, ebx  
00405B04  |.  0F84 AD010000 je      00405CB7  
00405B0A  |.  8B4424 30     mov     eax, \[esp+30\]  
00405B0E  |.  55            push    ebp  
00405B0F  |.  8B6C24 28     mov     ebp, \[esp+28\]  
00405B13  |.  50            push    eax  
00405B14  |.  55            push    ebp  
00405B15  |.  E8 56FFFFFF   call    00405A70  
00405B1A  |.  8B0B          mov     ecx, \[ebx\]  
00405B1C  |.  83C4 08       add     esp, 8  
00405B1F  |.  3BC8          cmp     ecx, eax  
00405B21  |.  7D 0A         jge     short 00405B2D  
00405B23  |.  5D            pop     ebp  
00405B24  |.  5F            pop     edi  
00405B25  |.  5E            pop     esi  
00405B26  |.  33C0          xor     eax, eax  
00405B28  |.  5B            pop     ebx  
00405B29  |.  83C4 10       add     esp, 10  
00405B2C  |.  C3            retn  
00405B2D  |>  8BC5          mov     eax, ebp                         ;  eax = ebp = 机器码长度  
00405B2F  |.  99            cdq  
00405B30  |.  B9 03000000   mov     ecx, 3  
00405B35  |.  F7F9          idiv    ecx                              ;  机器码长度/3  
00405B37  |.  B9 4C000000   mov     ecx, 4C  
00405B3C  |.  33DB          xor     ebx, ebx  
00405B3E  |.  BD 13000000   mov     ebp, 13  
00405B43  |.  C1E0 02       shl     eax, 2  
00405B46  |.  895424 1C     mov     \[esp+1C\], edx                    ;  余下字节数存入\[esp+1c\]  
00405B4A  |.  99            cdq  
00405B4B  |.  F7F9          idiv    ecx  
00405B4D  |.  8BC8          mov     ecx, eax  
00405B4F  |.  33C0          xor     eax, eax  
00405B51  |.  85C9          test    ecx, ecx  
00405B53  |.  895424 10     mov     \[esp+10\], edx  
00405B57  |.  894C24 14     mov     \[esp+14\], ecx  
00405B5B  |.  894424 24     mov     \[esp+24\], eax  
00405B5F  |.  0F8C C2000000 jl      00405C27  
00405B65  |.  8B5424 34     mov     edx, \[esp+34\]  
00405B69  |.  83E2 02       and     edx, 2  
00405B6C  |.  895424 18     mov     \[esp+18\], edx  
00405B70  |.  3BC1          cmp     eax, ecx  
00405B72  |>  75 0F         /jnz     short 00405B83  
00405B74  |.  8B4424 10     |mov     eax, \[esp+10\]  
00405B78  |.  99            |cdq  
00405B79  |.  83E2 03       |and     edx, 3  
00405B7C  |.  03C2          |add     eax, edx  
00405B7E  |.  8BE8          |mov     ebp, eax  
00405B80  |.  C1FD 02       |sar     ebp, 2                          ;  ebp 循环次数=机器码长/3 的商  
00405B83  |>  85ED          |test    ebp, ebp  
00405B85  |.  7E 6B         |jle     short 00405BF2  
00405B87  |.  8BD5          |mov     edx, ebp                        ;  edx＝循环计数器  
00405B89  |.  8DA424 000000>|lea     esp, \[esp\]  
00405B90  |>  0FB60E        |/movzx   ecx, byte ptr \[esi\]            ;  取第 1 个字节  
00405B93  |.  0FB646 01     ||movzx   eax, byte ptr \[esi+1\]          ;  取第 2 个字节  
00405B97  |.  C1E1 08       ||shl     ecx, 8                         ;  ecx 左移 8 位  
00405B9A  |.  0BC8          ||or      ecx, eax                       ;  连接 2 个字节  
00405B9C  |.  46            ||inc     esi                            ;  pSrc++  
00405B9D  |.  0FB646 01     ||movzx   eax, byte ptr \[esi+1\]          ;  取第 3 个字节  
00405BA1  |.  C1E1 08       ||shl     ecx, 8                         ;  ecx 左移 8 位  
00405BA4  |.  0BC8          ||or      ecx, eax                       ;  连接 3 个字节  
00405BA6  |.  46            ||inc     esi                            ;  pSrc++  
00405BA7  |.  C1E1 08       ||shl     ecx, 8                         ;  ecx 左移 8 位 3 个字节编码为 4 个字符  
00405BAA  |.  8BC1          ||mov     eax, ecx                       ;  eax = ecx  
00405BAC  |.  C1E8 1A       ||shr     eax, 1A                        ;  eax 右移 26 位 得到第 1 组 6bit  
00405BAF  |.  8A80 98FA4300 ||mov     al, \[eax+43FA98\]               ;  6bit 的值作为索引查表  
00405BB5  |.  8807          ||mov     \[edi\], al                      ;  保存第 1 个加密后字符  
00405BB7  |.  C1E1 06       ||shl     ecx, 6                         ;  去掉前 6bit  
00405BBA  |.  8BC1          ||mov     eax, ecx                       ;  eax = ecx  
00405BBC  |.  C1E8 1A       ||shr     eax, 1A                        ;  得到第 2 组 6bit  
00405BBF  |.  8A80 98FA4300 ||mov     al, \[eax+43FA98\]               ;  查表  
00405BC5  |.  8847 01       ||mov     \[edi+1\], al                    ;  保存第 2 个加密后字符  
00405BC8  |.  46            ||inc     esi                            ;  pSrc++  
00405BC9  |.  C1E1 06       ||shl     ecx, 6                         ;  再去掉 6bit  
00405BCC  |.  47            ||inc     edi                            ;  pDst++  
00405BCD  |.  8BC1          ||mov     eax, ecx                       ;  eax = ecx  
00405BCF  |.  C1E8 1A       ||shr     eax, 1A                        ;  得到第 3 组 6bit  
00405BD2  |.  8A80 98FA4300 ||mov     al, \[eax+43FA98\]               ;  查表  
00405BD8  |.  C1E9 14       ||shr     ecx, 14                        ;  直接右移 20 位得到第 4 组 6bit  
00405BDB  |.  47            ||inc     edi                            ;  pDst++  
00405BDC  |.  8807          ||mov     \[edi\], al                      ;  保存第 3 个加密后字符  
00405BDE  |.  83E1 3F       ||and     ecx, 3F                        ;  将前第 4 组 6bit 的前 2bit 置 0  
00405BE1  |.  8A89 98FA4300 ||mov     cl, \[ecx+43FA98\]               ;  查表  
00405BE7  |.  47            ||inc     edi                            ;  pDst++  
00405BE8  |.  880F          ||mov     \[edi\], cl                      ;  保存第 4 个加密后字符  
00405BEA  |.  47            ||inc     edi                            ;  pDst++  
00405BEB  |.  4A            ||dec     edx                            ;  循环计数器--  
00405BEC  |.^ 75 A2         |\\jnz     short 00405B90  
00405BEE  |.  8B4C24 14     |mov     ecx, \[esp+14\]  
00405BF2  |>  8B5424 18     |mov     edx, \[esp+18\]  
00405BF6  |.  85D2          |test    edx, edx  
00405BF8  |.  8D1CAB        |lea     ebx, \[ebx+ebp\*4\]  
00405BFB  |.  75 0B         |jnz     short 00405C08  
00405BFD  |.  C607 0D       |mov     byte ptr \[edi\], 0D              ;  输出回车  
00405C00  |.  47            |inc     edi                             ;  pDst++  
00405C01  |.  C607 0A       |mov     byte ptr \[edi\], 0A              ;  输出换行  
00405C04  |.  47            |inc     edi                             ;  pDst++  
00405C05  |.  83C3 02       |add     ebx, 2  
00405C08  |>  8B4424 24     |mov     eax, \[esp+24\]  
00405C0C  |.  40            |inc     eax  
00405C0D  |.  3BC1          |cmp     eax, ecx  
00405C0F  |.  894424 24     |mov     \[esp+24\], eax  
00405C13  |.^ 0F8E 59FFFFFF \\jle     00405B72  
00405C19  |.  85DB          test    ebx, ebx  
00405C1B  |.  74 0A         je      short 00405C27  
00405C1D  |.  85D2          test    edx, edx  
00405C1F  |.  75 06         jnz     short 00405C27  
00405C21  |.  83EF 02       sub     edi, 2  
00405C24  |.  83EB 02       sub     ebx, 2  
00405C27  |>  8B4C24 1C     mov     ecx, \[esp+1C\]                    ;  ecx = 余下字节数  
00405C2B  |.  85C9          test    ecx, ecx  
00405C2D  |.  74 75         je      short 00405CA4  
00405C2F  |.  8D69 01       lea     ebp, \[ecx+1\]  
00405C32  |.  85ED          test    ebp, ebp  
00405C34  |.  74 6E         je      short 00405CA4  
00405C36  |.  33C0          xor     eax, eax  
00405C38  |.  85C9          test    ecx, ecx  
00405C3A  |.  7E 04         jle     short 00405C40  
00405C3C  |.  0FB606        movzx   eax, byte ptr \[esi\]              ;  余下的字节  
00405C3F  |.  46            inc     esi  
00405C40  |>  C1E0 08       shl     eax, 8                           ;  eax 左移 8 位  
00405C43  |.  83F9 01       cmp     ecx, 1                           ;  是否余下 1 个字节  
00405C46  |.  7E 06         jle     short 00405C4E                   ;  不是  
00405C48  |.  0FB616        movzx   edx, byte ptr \[esi\]              ;  再取 1 个字节  
00405C4B  |.  0BC2          or      eax, edx                         ;  连接 2 个字节  
00405C4D  |.  46            inc     esi  
00405C4E  |>  C1E0 08       shl     eax, 8                           ;  eax 左移 8 位  
00405C51  |.  83F9 02       cmp     ecx, 2                           ;  是否余下 2 个字节  
00405C54  |.  7E 05         jle     short 00405C5B                   ;  是  
00405C56  |.  0FB60E        movzx   ecx, byte ptr \[esi\]  
00405C59  |.  0BC1          or      eax, ecx  
00405C5B  |>  C1E0 08       shl     eax, 8                           ;  eax 左移 8 位  
00405C5E  |.  85ED          test    ebp, ebp  
00405C60  |.  7E 16         jle     short 00405C78  
00405C62  |.  8BCD          mov     ecx, ebp  
00405C64  |>  8BD0          /mov     edx, eax  
00405C66  |.  C1EA 1A       |shr     edx, 1A                         ;  edx 右移 26 位 取 6bit  
00405C69  |.  8A92 98FA4300 |mov     dl, \[edx+43FA98\]                ;  查表  
00405C6F  |.  8817          |mov     \[edi\], dl                       ;  保存  
00405C71  |.  47            |inc     edi  
00405C72  |.  C1E0 06       |shl     eax, 6                          ;  eax 左移 6 位  
00405C75  |.  49            |dec     ecx  
00405C76  |.^ 75 EC         \\jnz     short 00405C64  
00405C78  |>  8A4424 34     mov     al, \[esp+34\]  
00405C7C  |.  03DD          add     ebx, ebp                         ;  当前字符串长  
00405C7E  |.  A8 01         test    al, 1  
00405C80  |.  75 22         jnz     short 00405CA4  
00405C82  |.  BA 04000000   mov     edx, 4  
00405C87  |.  2BD5          sub     edx, ebp  
00405C89  |.  85D2          test    edx, edx  
00405C8B  |.  7E 15         jle     short 00405CA2  
00405C8D  |.  8BCA          mov     ecx, edx  
00405C8F  |.  8BF1          mov     esi, ecx  
00405C91  |.  C1E9 02       shr     ecx, 2  
00405C94  |.  B8 3D3D3D3D   mov     eax, 3D3D3D3D  
00405C99  |.  F3:AB         rep     stos dword ptr es:\[edi\]  
00405C9B  |.  8BCE          mov     ecx, esi  
00405C9D  |.  83E1 03       and     ecx, 3  
00405CA0  |.  F3:AA         rep     stos byte ptr es:\[edi\]           ;  不满 4 个字符用"="填充  
00405CA2  |>  03DA          add     ebx, edx  
00405CA4  |>  8B4424 30     mov     eax, \[esp+30\]  
00405CA8  |.  5D            pop     ebp  
00405CA9  |.  5F            pop     edi  
00405CAA  |.  8918          mov     \[eax\], ebx  
00405CAC  |.  5E            pop     esi                              ;  机密后的字符串  
00405CAD  |.  B8 01000000   mov     eax, 1  
00405CB2  |.  5B            pop     ebx  
00405CB3  |.  83C4 10       add     esp, 10  
00405CB6  |.  C3            retn                                     ;  返回  
\---------------------------------------------------------------------;  现在可以确定这是 Base64 加密函数了  
  
00405F39  |.  83C4 14       add     esp, 14  
00405F3C  |.  85C0          test    eax, eax  
00405F3E  |.  74 13         je      short 00405F53  
00405F40  |.  56            push    esi  
00405F41  |.  8D5424 2C     lea     edx, \[esp+2C\]  
00405F45  |.  68 D8FA4300   push    0043FAD8                         ;  ASCII "%s"  
00405F4A  |.  52            push    edx  
00405F4B  |.  E8 30C1FFFF   call    00402080  
00405F50  |.  83C4 0C       add     esp, 0C  
00405F53  |>  56            push    esi  
00405F54  |.  E8 558E0200   call    0042EDAE  
00405F59  |.  8B7424 2C     mov     esi, \[esp+2C\]  
00405F5D  |.  83C6 F0       add     esi, -10  
00405F60  |.  56            push    esi  
00405F61  |.  E8 3AB6FFFF   call    004015A0  
00405F66  |.  8B7C24 2C     mov     edi, \[esp+2C\]  
00405F6A  |.  83C0 10       add     eax, 10  
00405F6D  |.  8907          mov     \[edi\], eax  
00405F6F  |.  83C4 08       add     esp, 8  
00405F72  |.  C74424 1C FFF>mov     dword ptr \[esp+1C\], -1  
00405F7A  |.  8D46 0C       lea     eax, \[esi+C\]  
00405F7D  |.  83C9 FF       or      ecx, FFFFFFFF  
00405F80  |.  F0:0FC108     lock xadd \[eax\], ecx  
00405F84  |.  49            dec     ecx  
00405F85  |.  85C9          test    ecx, ecx  
00405F87  |.  7F 08         jg      short 00405F91  
00405F89  |.  8B0E          mov     ecx, \[esi\]  
00405F8B  |.  8B11          mov     edx, \[ecx\]  
00405F8D  |.  56            push    esi  
00405F8E  |.  FF52 04       call    \[edx+4\]  
00405F91  |>  8BC7          mov     eax, edi  
00405F93  |.  5F            pop     edi  
00405F94  |.  5E            pop     esi  
00405F95  |.  5D            pop     ebp  
00405F96  |.  5B            pop     ebx  
00405F97  |.  8B4C24 04     mov     ecx, \[esp+4\]  
00405F9B  |.  64:890D 00000>mov     fs:\[0\], ecx  
00405FA2  |.  83C4 10       add     esp, 10  
00405FA5  |.  C2 0800       retn    8                                ;  返回  
  
00406285  |.  8B4424 28     mov     eax, \[esp+28\]  
00406289  |.  B3 02         mov     bl, 2  
0040628B  |.  83C0 F0       add     eax, -10  
0040628E  |.  885C24 20     mov     \[esp+20\], bl  
00406292  |.  8D50 0C       lea     edx, \[eax+C\]  
00406295  |.  83C9 FF       or      ecx, FFFFFFFF  
00406298  |.  F0:0FC10A     lock xadd \[edx\], ecx  
0040629C  |.  49            dec     ecx  
0040629D  |.  85C9          test    ecx, ecx  
0040629F  |.  7F 08         jg      short 004062A9  
004062A1  |.  8B08          mov     ecx, \[eax\]  
004062A3  |.  8B11          mov     edx, \[ecx\]  
004062A5  |.  50            push    eax  
004062A6  |.  FF52 04       call    \[edx+4\]  
004062A9  |>  6A 10         push    10  
004062AB  |.  8D4424 18     lea     eax, \[esp+18\]  
004062AF  |.  50            push    eax  
004062B0  |.  8D4C24 18     lea     ecx, \[esp+18\]  
004062B4  |.  E8 17B8FFFF   call    00401AD0                         ;  这个 CALL 跟入  
  
00401AD0  /$  51            push    ecx  
00401AD1  |.  56            push    esi  
00401AD2  |.  57            push    edi  
00401AD3  |.  8B7C24 14     mov     edi, \[esp+14\]  
00401AD7  |.  85FF          test    edi, edi  
00401AD9  |.  C74424 08 000>mov     dword ptr \[esp+8\], 0  
00401AE1  |.  7D 02         jge     short 00401AE5  
00401AE3  |.  33FF          xor     edi, edi  
00401AE5  |>  8B31          mov     esi, \[ecx\]  
00401AE7  |.  3B7E F4       cmp     edi, \[esi-C\]                     ;  如果加密后字符数大于 16 位  
00401AEA  |.  7C 1D         jl      short 00401B09  
00401AEC  |.  8D46 F0       lea     eax, \[esi-10\]  
00401AEF  |.  50            push    eax  
00401AF0  |.  E8 ABFAFFFF   call    004015A0  
00401AF5  |.  83C4 04       add     esp, 4  
00401AF8  |.  8BC8          mov     ecx, eax  
00401AFA  |.  8B4424 10     mov     eax, \[esp+10\]  
00401AFE  |.  83C1 10       add     ecx, 10  
00401B01  |.  5F            pop     edi  
00401B02  |.  8908          mov     \[eax\], ecx  
00401B04  |.  5E            pop     esi  
00401B05  |.  59            pop     ecx  
00401B06  |.  C2 0800       retn    8  
00401B09  |>  8B4E F0       mov     ecx, \[esi-10\]  
00401B0C  |.  8B11          mov     edx, \[ecx\]  
00401B0E  |.  FF52 10       call    \[edx+10\]  
00401B11  |.  50            push    eax  
00401B12  |.  57            push    edi  
00401B13  |.  56            push    esi  
00401B14  |.  8B7424 1C     mov     esi, \[esp+1C\]  
00401B18  |.  8BCE          mov     ecx, esi  
00401B1A  |.  E8 E1F8FFFF   call    00401400                         ;  截取前 16 位  
00401B1F  |.  5F            pop     edi  
00401B20  |.  8BC6          mov     eax, esi  
00401B22  |.  5E            pop     esi  
00401B23  |.  59            pop     ecx  
00401B24  \\.  C2 0800       retn    8                                ;  返回  
  
004062B9  |.  8B4C24 2C     mov     ecx, \[esp+2C\]  
004062BD  |.  50            push    eax  
004062BE  |.  C64424 24 03  mov     byte ptr \[esp+24\], 3  
004062C3  |.  E8 28BBFFFF   call    00401DF0  
004062C8  |.  8B4424 14     mov     eax, \[esp+14\]  
004062CC  |.  83C0 F0       add     eax, -10  
004062CF  |.  885C24 20     mov     \[esp+20\], bl  
004062D3  |.  8D48 0C       lea     ecx, \[eax+C\]  
004062D6  |.  83CA FF       or      edx, FFFFFFFF  
004062D9  |.  F0:0FC111     lock xadd \[ecx\], edx  
004062DD  |.  4A            dec     edx  
004062DE  |.  85D2          test    edx, edx  
004062E0  |.  7F 08         jg      short 004062EA  
004062E2  |.  8B08          mov     ecx, \[eax\]  
004062E4  |.  8B11          mov     edx, \[ecx\]  
004062E6  |.  50            push    eax  
004062E7  |.  FF52 04       call    \[edx+4\]  
004062EA  |>  56            push    esi  
004062EB  |.  E8 BE8A0200   call    0042EDAE  
004062F0  |.  8B4424 14     mov     eax, \[esp+14\]  
004062F4  |.  83C0 F0       add     eax, -10  
004062F7  |.  83C4 04       add     esp, 4  
004062FA  |.  C74424 20 FFF>mov     dword ptr \[esp+20\], -1  
00406302  |.  8D48 0C       lea     ecx, \[eax+C\]  
00406305  |.  83CA FF       or      edx, FFFFFFFF  
00406308  |.  F0:0FC111     lock xadd \[ecx\], edx  
0040630C  |.  4A            dec     edx  
0040630D  |.  85D2          test    edx, edx  
0040630F  |.  5F            pop     edi  
00406310  |.  5E            pop     esi  
00406311  |.  7F 08         jg      short 0040631B  
00406313  |.  8B08          mov     ecx, \[eax\]  
00406315  |.  8B11          mov     edx, \[ecx\]  
00406317  |.  50            push    eax  
00406318  |.  FF52 04       call    \[edx+4\]  
0040631B  |>  8B4C24 10     mov     ecx, \[esp+10\]  
0040631F  |.  5D            pop     ebp  
00406320  |.  B0 01         mov     al, 1  
00406322  |.  5B            pop     ebx  
00406323  |.  64:890D 00000>mov     fs:\[0\], ecx  
0040632A  |.  83C4 14       add     esp, 14  
0040632D  \\.  C2 0800       retn    8                                ;  返回  
  
0040EAC1   .  8B07          mov     eax, \[edi\]  
0040EAC3   .  8B4C24 10     mov     ecx, \[esp+10\]  
0040EAC7   .  50            push    eax  
0040EAC8   .  51            push    ecx  
0040EAC9   .  E8 9CE90000   call    0041D46A                         ;  比较真假注册码的每一位  相同 EAX 置 0  
0040EACE   .  83C4 08       add     esp, 8  
0040EAD1   .  85C0          test    eax, eax  
0040EAD3   .  74 4B         je      short 0040EB20                   ;  eax = 0 跳向成功  
0040EAD5   .  6A 00         push    0  
0040EAD7   .  68 C8F64300   push    0043F6C8  
0040EADC   .  68 FC104400   push    004410FC  
0040EAE1   .  8BCE          mov     ecx, esi  
0040EAE3   .  E8 1F120200   call    0042FD07                         ;  eax !=0   GAME OVER  
  
小结：  
机器码代入运算 1 中输出定义为 T1  
T1 经过 Base64 运算输出定义为 T2  
取 T2 的前 16 位即为注册码  
  
具体的实现请看下面的代码  
  
const char EnBase64Tab\[\] = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";  
   
int EncodeBase64(const unsigned char\* pSrc, char\* pDst, int nSrcLen)  
{  
    unsigned char c1, c2, c3;  
    int nDstLen = 0;  
    int nLineLen = 0;  
    int nDiv = nSrcLen / 3;  
    int nMod = nSrcLen % 3;  
   
    for (int i = 0; i < nDiv; i ++)  
    {  
        c1 = \*pSrc++;  
        c2 = \*pSrc++;  
        c3 = \*pSrc++;  
        \*pDst++ = EnBase64Tab\[c1 >> 2\];  
        \*pDst++ = EnBase64Tab\[((c1 << 4) | (c2 >> 4)) & 0x3f\];  
        \*pDst++ = EnBase64Tab\[((c2 << 2) | (c3 >> 6)) & 0x3f\];  
        \*pDst++ = EnBase64Tab\[c3 & 0x3f\];  
        nLineLen += 4;  
        nDstLen += 4;  
    }  
    if (nMod == 1)  
    {  
        c1 = \*pSrc++;  
        \*pDst++ = EnBase64Tab\[(c1 & 0xfc) >> 2\];  
        \*pDst++ = EnBase64Tab\[((c1 & 0x03) << 4)\];  
        \*pDst++ = '=';  
        \*pDst++ = '=';  
        nLineLen += 4;  
        nDstLen += 4;  
    }  
    else if (nMod == 2)  
    {  
        c1 = \*pSrc++;  
        c2 = \*pSrc++;  
        \*pDst++ = EnBase64Tab\[(c1 & 0xfc) >> 2\];  
        \*pDst++ = EnBase64Tab\[((c1 & 0x03) << 4) | ((c2 & 0xf0) >> 4)\];  
        \*pDst++ = EnBase64Tab\[((c2 & 0x0f) << 2)\];  
        \*pDst++ = '=';  
        nDstLen += 4;  
    }  
    \*pDst = '\\0';  
    return nDstLen;  
}  
  
void CKeyGenDlg::OnButton1()   
{  
  // TODO: Add your control notification handler code here  
  CString in,out;  
  char al,dl,en64\[50\];    
  unsigned char temp\[50\];  
    
  GetDlgItemText(IDC\_EDIT1,in);  
  int len=strlen(in);  
  char \*mac=in.GetBuffer(strlen(in));  
  for(int i=0;i<len;i++)  
  {  
    al=(mac\[i\]+=i);  
    dl=i;  
    al+=1;  
    dl=dl<<2 & 0x000000ff;  
    al^=dl;  
    if(al==0x30) al+=0x65;  
    temp\[i\]=al;  
  }  
  temp\[i\]='\\0';  
  EncodeBase64(temp,en64,i);  
  out=en64;  
  out=out.Left(16);  
  SetDlgItemText(IDC\_EDIT2,out);  
}  

 4dd466d0e1e0cc123afbe504da2462ca