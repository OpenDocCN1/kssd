
```
00405E6D  |.  7F 26         jg      short 00405E95  
00405E6F  |.  8B7424 20     mov     esi, [esp+20]  
00405E73  |.  68 F8FA4300   push    0043FAF8  
00405E78  |.  8BCE          mov     ecx, esi  
00405E7A  |.  E8 81CCFFFF   call    00402B00  
00405E7F  |.  8BC6          mov     eax, esi  
00405E81  |.  5E            pop     esi  
00405E82  |.  5D            pop     ebp  
00405E83  |.  5B            pop     ebx  
00405E84  |.  8B4C24 04     mov     ecx, [esp+4]  
00405E88  |.  64:890D 00000>mov     fs:[0], ecx  
00405E8F  |.  83C4 10       add     esp, 10  
00405E92  |.  C2 0800       retn    8  
00405E95  |>  8D0C9D 000000>lea     ecx, [ebx*4]  
00405E9C  |.  B8 56555555   mov     eax, 55555556  
00405EA1  |.  F7E9          imul    ecx  
00405EA3  |.  8BCA          mov     ecx, edx  
00405EA5  |.  C1E9 1F       shr     ecx, 1F  
00405EA8  |.  03CA          add     ecx, edx  
00405EAA  |.  8BC3          mov     eax, ebx  
00405EAC  |.  99            cdq  
00405EAD  |.  BE 03000000   mov     esi, 3  
00405EB2  |.  F7FE          idiv    esi  
00405EB4  |.  BE 4C000000   mov     esi, 4C  
00405EB9  |.  03CA          add     ecx, edx  
00405EBB  |.  8BC1          mov     eax, ecx  
00405EBD  |.  99            cdq  
00405EBE  |.  F7FE          idiv    esi  
00405EC0  |.  85D2          test    edx, edx  
00405EC2  |.  74 18         je      short 00405EDC  
00405EC4  |.  81E2 03000080 and     edx, 80000003  
00405ECA  |.  79 05         jns     short 00405ED1  
00405ECC  |.  4A            dec     edx  
00405ECD  |.  83CA FC       or      edx, FFFFFFFC  
00405ED0  |.  42            inc     edx  
00405ED1  |>  74 09         je      short 00405EDC  
00405ED3  |.  BE 04000000   mov     esi, 4  
00405ED8  |.  2BF2          sub     esi, edx  
00405EDA  |.  03CE          add     ecx, esi  
00405EDC  |>  57            push    edi  
00405EDD  |.  8D7C41 02     lea     edi, [ecx+eax*2+2]  
00405EE1  |.  85FF          test    edi, edi  
00405EE3  |.  897C24 10     mov     [esp+10], edi  
00405EE7  |.  0F8E BB000000 jle     00405FA8  
00405EED  |.  57            push    edi  
00405EEE  |.  E8 C08E0200   call    0042EDB3  
00405EF3  |.  8BCF          mov     ecx, edi  
00405EF5  |.  8BD1          mov     edx, ecx  
00405EF7  |.  8BF0          mov     esi, eax  
00405EF9  |.  C1E9 02       shr     ecx, 2  
00405EFC  |.  33C0          xor     eax, eax  
00405EFE  |.  8BFE          mov     edi, esi  
00405F00  |.  F3:AB         rep     stos dword ptr es:[edi]  
00405F02  |.  8BCA          mov     ecx, edx  
00405F04  |.  83E1 03       and     ecx, 3  
00405F07  |.  83C4 04       add     esp, 4  
00405F0A  |.  F3:AA         rep     stos byte ptr es:[edi]  
00405F0C  |.  E8 FD8E0200   call    0042EE0E  
00405F11  |.  8B10          mov     edx, [eax]  
00405F13  |.  8BC8          mov     ecx, eax  
00405F15  |.  FF52 0C       call    [edx+C]  
00405F18  |.  83C0 10       add     eax, 10  
00405F1B  |.  894424 28     mov     [esp+28], eax  
00405F1F  |.  8B45 00       mov     eax, [ebp]  
00405F22  |.  6A 00         push    0  
00405F24  |.  8D4C24 14     lea     ecx, [esp+14]  
00405F28  |.  51            push    ecx  
00405F29  |.  56            push    esi  
00405F2A  |.  53            push    ebx  
00405F2B  |.  50            push    eax  
00405F2C  |.  C74424 30 000>mov     dword ptr [esp+30], 0  
00405F34  |.  E8 A7FBFFFF   call    00405AE0                       ;  Base64 加密 CALL？  
\------------------------------------------------------------------------跟进去看看就知道了  
00405AE0  /$  83EC 10       sub     esp, 10  
00405AE3  |.  53            push    ebx  
00405AE4  |.  56            push    esi  
00405AE5  |.  8B7424 1C     mov     esi, [esp+1C]  
00405AE9  |.  85F6          test    esi, esi  
00405AEB  |.  57            push    edi  
00405AEC  |.  0F84 C5010000 je      00405CB7  
00405AF2  |.  8B7C24 28     mov     edi, [esp+28]  
00405AF6  |.  85FF          test    edi, edi  
00405AF8  |.  0F84 B9010000 je      00405CB7  
00405AFE  |.  8B5C24 2C     mov     ebx, [esp+2C]  
00405B02  |.  85DB          test    ebx, ebx  
00405B04  |.  0F84 AD010000 je      00405CB7  
00405B0A  |.  8B4424 30     mov     eax, [esp+30]  
00405B0E  |.  55            push    ebp  
00405B0F  |.  8B6C24 28     mov     ebp, [esp+28]  
00405B13  |.  50            push    eax  
00405B14  |.  55            push    ebp  
00405B15  |.  E8 56FFFFFF   call    00405A70  
00405B1A  |.  8B0B          mov     ecx, [ebx]  
00405B1C  |.  83C4 08       add     esp, 8  
00405B1F  |.  3BC8          cmp     ecx, eax  
00405B21  |.  7D 0A         jge     short 00405B2D  
00405B23  |.  5D            pop     ebp  
00405B24  |.  5F            pop     edi  
00405B25  |.  5E            pop     esi  
00405B26  |.  33C0          xor     eax, eax  
00405B28  |.  5B            pop     ebx  
00405B29  |.  83C4 10       add     esp, 10  
00405B2C  |.  C3            retn  
00405B2D  |>  8BC5          mov     eax, ebp                       ;  eax = ebp = 机器码长度  
00405B2F  |.  99            cdq  
00405B30  |.  B9 03000000   mov     ecx, 3  
00405B35  |.  F7F9          idiv    ecx                            ;  机器码长度/3  
00405B37  |.  B9 4C000000   mov     ecx, 4C  
00405B3C  |.  33DB          xor     ebx, ebx  
00405B3E  |.  BD 13000000   mov     ebp, 13  
00405B43  |.  C1E0 02       shl     eax, 2  
00405B46  |.  895424 1C     mov     [esp+1C], edx                  ;  余下字节数存入[esp+1c]  
00405B4A  |.  99            cdq  
00405B4B  |.  F7F9          idiv    ecx  
00405B4D  |.  8BC8          mov     ecx, eax  
00405B4F  |.  33C0          xor     eax, eax  
00405B51  |.  85C9          test    ecx, ecx  
00405B53  |.  895424 10     mov     [esp+10], edx  
00405B57  |.  894C24 14     mov     [esp+14], ecx  
00405B5B  |.  894424 24     mov     [esp+24], eax  
00405B5F  |.  0F8C C2000000 jl      00405C27  
00405B65  |.  8B5424 34     mov     edx, [esp+34]  
00405B69  |.  83E2 02       and     edx, 2  
00405B6C  |.  895424 18     mov     [esp+18], edx  
00405B70  |.  3BC1          cmp     eax, ecx  
00405B72  |>  75 0F         /jnz     short 00405B83  
00405B74  |.  8B4424 10     |mov     eax, [esp+10]  
00405B78  |.  99            |cdq  
00405B79  |.  83E2 03       |and     edx, 3  
00405B7C  |.  03C2          |add     eax, edx  
00405B7E  |.  8BE8          |mov     ebp, eax  
00405B80  |.  C1FD 02       |sar     ebp, 2                         ;  ebp 循环次数=机器码长/3 的商  
00405B83  |>  85ED          |test    ebp, ebp  
00405B85  |.  7E 6B         |jle     short 00405BF2  
00405B87  |.  8BD5          |mov     edx, ebp                       ;  edx＝循环计数器  
00405B89  |.  8DA424 000000>|lea     esp, [esp]  
00405B90  |>  0FB60E        |/movzx   ecx, byte ptr [esi]            ;  取第 1 个字节  
00405B93  |.  0FB646 01     ||movzx   eax, byte ptr [esi+1]          ;  取第 2 个字节  
00405B97  |.  C1E1 08       ||shl     ecx, 8                         ;  ecx 左移 8 位  
00405B9A  |.  0BC8          ||or      ecx, eax                      ;  连接 2 个字节  
00405B9C  |.  46            ||inc     esi                            ;  pSrc++  
00405B9D  |.  0FB646 01     ||movzx   eax, byte ptr [esi+1]          ;  取第 3 个字节  
00405BA1  |.  C1E1 08       ||shl     ecx, 8                         ;  ecx 左移 8 位  
00405BA4  |.  0BC8          ||or      ecx, eax                      ;  连接 3 个字节  
00405BA6  |.  46            ||inc     esi                            ;  pSrc++  
00405BA7  |.  C1E1 08       ||shl     ecx, 8                         ;  ecx 左移 8 位 3 个字节编码为 4 个字符  
00405BAA  |.  8BC1          ||mov     eax, ecx                      ;  eax = ecx  
00405BAC  |.  C1E8 1A       ||shr     eax, 1A                        ;  eax 右移 26 位 得到第 1 组 6bit  
00405BAF  |.  8A80 98FA4300 ||mov     al, [eax+43FA98]              ;  6bit 的值作为索引查表  
```
