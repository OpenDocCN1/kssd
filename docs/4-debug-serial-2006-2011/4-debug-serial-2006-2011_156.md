
# 词林宝典 8.03 解码记

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-402.htm`](https://www.pediy.com/kssd/pediy08/pediy8-402.htm)

**破文作者**：`lzq1973[PYG][CZG][OCN][DFCG]`  
**作者邮箱**：`lzq9888@126.com`  
**破解工具**：`OD`  
**破解平台**：`WinXP`  
**软件名称**：词林宝典  
**软件大小**：`7575K`  
**原版下载**：`http://www.sharebank.com.cn/soft/SoftView_18601.htm`  
**保护方式**：使用次数、注册码  
**破解声明**：俺是只小小鸟，纯为学习，愿与大家分享！

---

`OD` 载入，查找相关字符串后，在这里下断

```asm
004D096C   $  55             push    ebp
004D096D   .  8BEC           mov     ebp, esp
004D096F   .  B9 0B000000    mov     ecx, 0B
004D0974   >  6A 00          push    0
004D0976   .  6A 00          push    0
004D0978   .  49             dec     ecx
004D0979   .^ 75 F9          jnz     short 004D0974
004D097B   .  51             push    ecx
004D097C   .  53             push    ebx
004D097D   .  56             push    esi
004D097E   .  8BD8           mov     ebx, eax
004D0980   .  33C0          xor     eax, eax
004D0982   .  55             push    ebp
004D0983   .  68 370D4D00    push    004D0D37
004D0988   .  64:FF30        push    dword ptr fs:[eax]
004D098B   .  64:8920        mov     fs:[eax], esp
004D098E   .  B8 500D4D00    mov     eax, 004D0D50                 ;  85016698
004D0993   .  E8 3083F3FF    call    00408CC8
004D0998   .  8945 F8        mov     [ebp-8], eax
004D099B   .  33C0          xor     eax, eax
004D099D   .  8945 F4        mov     [ebp-C], eax
004D09A0   .  8D55 EC        lea     edx, [ebp-14]
004D09A3   .  8B83 04030000  mov     eax, [ebx+304]
004D09A9   .  E8 D20CF7FF    call    00441680
004D09AE   .  8B45 EC        mov     eax, [ebp-14]                 ;  (ASCII "lzq1973")
004D09B1   .  8D55 F0        lea     edx, [ebp-10]
004D09B4   .  E8 BB7FF3FF    call    00408974
004D09B9   .  8B45 F0        mov     eax, [ebp-10]
004D09BC   .  E8 FB3DF3FF    call    004047BC
004D09C1   .  83F8 04        cmp     eax, 4                          ;  用户名长度是否大于 4
004D09C4   .  0F8C 57020000  jl      004D0C21                        ;  小于则失败
004D09CA   .  8D55 E4        lea     edx, [ebp-1C]
004D09CD   .  8B83 0C030000  mov     eax, [ebx+30C]
004D09D3   .  E8 A80CF7FF    call    00441680
004D09D8   .  8B45 E4        mov     eax, [ebp-1C]                   ;  假码
004D09DB   .  8D55 E8        lea     edx, [ebp-18]
004D09DE   .  E8 917FF3FF    call    00408974
004D09E3   .  8B45 E8        mov     eax, [ebp-18]
004D09E6   .  E8 D13DF3FF    call    004047BC
004D09EB   .  83F8 11        cmp     eax, 11                         ;  注册码长度是否为 17
004D09EE   .  0F85 2D020000  jnz     004D0C21                        ;  否则失败
004D09F4   .  8D55 DC        lea     edx, [ebp-24]
004D09F7   .  8B83 0C030000  mov     eax, [ebx+30C]
004D09FD   .  E8 7E0CF7FF    call    00441680
004D0A02   .  8B45 DC        mov     eax, [ebp-24]
004D0A05   .  8D55 E0        lea     edx, [ebp-20]
004D0A08   .  E8 677FF3FF    call    00408974
004D0A0D   .  8B45 E0        mov     eax, [ebp-20]
004D0A10   .  50             push    eax
004D0A11   .  8D55 D0        lea     edx, [ebp-30]
004D0A14   .  8B83 08030000  mov     eax, [ebx+308]
004D0A1A   .  E8 610CF7FF    call    00441680
004D0A1F   .  8B45 D0        mov     eax, [ebp-30]                   ;  (ASCII "651645186G01")
004D0A22   .  8D55 D4        lea     edx, [ebp-2C]
004D0A25   .  E8 4A7FF3FF    call    00408974
004D0A2A   .  8B45 D4        mov     eax, [ebp-2C]
004D0A2D   .  50             push    eax
004D0A2E   .  8D55 C8        lea     edx, [ebp-38]
004D0A31   .  8B83 04030000  mov     eax, [ebx+304]
004D0A37   .  E8 440CF7FF    call    00441680
004D0A3C   .  8B45 C8        mov     eax, [ebp-38]
004D0A3F   .  8D55 CC        lea     edx, [ebp-34]
004D0A42   .  E8 2D7FF3FF    call    00408974
004D0A47   .  8B45 CC        mov     eax, [ebp-34]
004D0A4A   .  8D4D D8        lea     ecx, [ebp-28]
004D0A4D   .  5A             pop     edx
004D0A4E   .  E8 01F9FFFF    call    004D0354                        ;  关键算法
004D0A53   .  8B55 D8        mov     edx, [ebp-28]                   ;  注册码 (ASCII "28F71-99A69-235E5")
004D0A56   .  58             pop     eax
004D0A57   .  E8 A43EF3FF    call    00404900                        ;  可做内存注册器
004D0A5C   .  74 15          je      short 004D0A73
004D0A5E   .  8B83 00030000  mov     eax, [ebx+300]
004D0A64   .  BA 640D4D00    mov     edx, 004D0D64                   ;  提示：\n    注册码不正确！
004D0A69   .  E8 420CF7FF    call    004416B0
004D0A6E   .  E9 1C020000    jmp     004D0C8F
004D0A73   >  8D55 C0        lea     edx, [ebp-40]
004D0A76   .  8B83 0C030000  mov     eax, [ebx+30C]
004D0A7C   .  E8 FF0BF7FF    call    00441680
004D0A81   .  8B45 C0        mov     eax, [ebp-40]
004D0A84   .  8D55 C4        lea     edx, [ebp-3C]
004D0A87   .  E8 E87EF3FF    call    00408974
004D0A8C   .  8B45 C4        mov     eax, [ebp-3C]
004D0A8F   .  50             push    eax
004D0A90   .  8D55 B8        lea     edx, [ebp-48]
004D0A93   .  8B83 04030000  mov     eax, [ebx+304]
004D0A99   .  E8 E20BF7FF    call    00441680
004D0A9E   .  8B45 B8        mov     eax, [ebp-48]
004D0AA1   .  8D55 BC        lea     edx, [ebp-44]
004D0AA4   .  E8 CB7EF3FF    call    00408974
004D0AA9   .  8B55 BC        mov     edx, [ebp-44]
004D0AAC   .  8B83 14030000  mov     eax, [ebx+314]
004D0AB2   .  59             pop     ecx
004D0AB3   .  E8 6881FEFF    call    004B8C20
004D0AB8   .  68 880D4D00    push    004D0D88                         ;  注册成功！\n 用户名：
004D0ABD   .  8BB3 14030000  mov     esi, [ebx+314]
004D0AC3   .  FFB6 AC010000  push    dword ptr [esi+1AC]
004D0AC9   .  68 A40D4D00    push    004D0DA4                         ;  \n
004D0ACE   .  68 B00D4D00    push    004D0DB0                         ;  注册码：
004D0AD3   .  FFB6 BC010000  push    dword ptr [esi+1BC]
004D0AD9   .  68 A40D4D00    push    004D0DA4                         ;  \n
004D0ADE   .  68 A40D4D00    push    004D0DA4                         ;  \n
004D0AE3   .  68 C40D4D00    push    004D0DC4                         ;  请退出注册程序，重新启动主程序。
004D0AE8   .  8D45 B4        lea     eax, [ebp-4C]
004D0AEB   .  BA 08000000    mov     edx, 8
004D0AF0   .  E8 873DF3FF    call    0040487C
004D0AF5   .  8B55 B4        mov     edx, [ebp-4C]
004D0AF8   .  8B83 00030000  mov     eax, [ebx+300]
004D0AFE   .  E8 AD0BF7FF    call    004416B0
004D0B03   .  8B83 F4020000  mov     eax, [ebx+2F4]
004D0B09   .  33D2          xor     edx, edx
004D0B0B   .  E8 900AF7FF    call    004415A0
004D0B10   .  8B83 F8020000  mov     eax, [ebx+2F8]
004D0B16   .  33D2          xor     edx, edx
004D0B18   .  E8 830AF7FF    call    004415A0
004D0B1D   .  B2 01         mov     dl, 1
004D0B1F   .  A1 C8354600    mov     eax, [4635C8]
004D0B24   .  E8 9F2BF9FF    call    004636C8
004D0B29   .  8945 FC        mov     [ebp-4], eax
004D0B2C   .  33C0          xor     eax, eax
004D0B2E   .  55             push    ebp
004D0B2F   .  68 980B4D00    push    004D0B98
004D0B34   .  64:FF30        push    dword ptr fs:[eax]
004D0B37   .  64:8920        mov     fs:[eax], esp
004D0B3A   .  BA 02000080    mov     edx, 80000002
004D0B3F   .  8B45 FC        mov     eax, [ebp-4]
004D0B42   .  E8 212CF9FF    call    00463768
004D0B47   .  B1 01         mov     cl, 1
004D0B49   .  BA F00D4D00    mov     edx, 004D0DF0                    ;  \software\classes\clsid\{ab356385-0605-0606-0611-abcd20060428}\inprocserver32
004D0B4E   .  8B45 FC        mov     eax, [ebp-4]
004D0B51   .  E8 762CF9FF    call    004637CC
004D0B56   .  6A 04          push    4                                 ; /Arg1 = 00000004
```



```asm
004D0B58   .  8D4D F8       lea     ecx, [ebp-8]                    ; |  
004D0B5B   .  BA 480E4D00   mov     edx, 004D0E48                   ; |threadingmodel  
004D0B60   .  8B45 FC       mov     eax, [ebp-4]                    ; |  
004D0B63   .  E8 E82EF9FF   call    00463A50                         ; \CLBD.00463A50  
004D0B68   .  6A 04         push    4                                 ; /Arg1 = 00000004  
004D0B6A   .  8D4D F4       lea     ecx, [ebp-C]                     ; |  
004D0B6D   .  BA 600E4D00   mov     edx, 004D0E60                   ; |threadingmodelnn  
004D0B72   .  8B45 FC       mov     eax, [ebp-4]                    ; |  
004D0B75   .  E8 D62EF9FF   call    00463A50                         ; \CLBD.00463A50  
004D0B7A   .  8B45 FC       mov     eax, [ebp-4]  
004D0B7D   .  E8 B62BF9FF   call    00463738  
004D0B82   .  33C0          xor     eax, eax  
004D0B84   .  5A            pop     edx  
004D0B85   .  59            pop     ecx  
004D0B86   .  59            pop     ecx  
004D0B87   .  64:8910       mov     fs:[eax], edx  
004D0B8A   .  68 9F0B4D00   push    004D0B9F  
004D0B8F   >  8B45 FC       mov     eax, [ebp-4]  
004D0B92   .  E8 7D2BF3FF   call    00403714  
004D0B97   .  C3            retn  
004D0B98   .^ E9 0B33F3FF   jmp     00403EA8  
004D0B9D   .^ EB F0         jmp     short 004D0B8F  
004D0B9F   .  B2 01         mov     dl, 1  
004D0BA1   .  A1 C8354600   mov     eax, [4635C8]  
004D0BA6   .  E8 1D2BF9FF   call    004636C8  
004D0BAB   .  8945 FC       mov     [ebp-4], eax  
004D0BAE   .  33C0          xor     eax, eax  
004D0BB0   .  55            push    ebp  
004D0BB1   .  68 1A0C4D00   push    004D0C1A  
004D0BB6   .  64:FF30       push    dword ptr fs:[eax]  
004D0BB9   .  64:8920       mov     fs:[eax], esp  
004D0BBC   .  BA 02000080   mov     edx, 80000002  
004D0BC1   .  8B45 FC       mov     eax, [ebp-4]  
004D0BC4   .  E8 9F2BF9FF   call    00463768  
004D0BC9   .  B1 01         mov     cl, 1  
004D0BCB   .  BA 7C0E4D00   mov     edx, 004D0E7C                   ;  ASCII "\Software\CLASSES\CLSID\{CD356868-0605-0606-0611-abcd20060428}\InprocServer32"  
004D0BD0   .  8B45 FC       mov     eax, [ebp-4]  
004D0BD3   .  E8 F42BF9FF   call    004637CC  
004D0BD8   .  6A 04         push    4                                 ; /Arg1 = 00000004  
004D0BDA   .  8D4D F8       lea     ecx, [ebp-8]                     ; |  
004D0BDD   .  BA 480E4D00   mov     edx, 004D0E48                   ; |ASCII "ThreadingModel"  
004D0BE2   .  8B45 FC       mov     eax, [ebp-4]                    ; |  
004D0BE5   .  E8 662EF9FF   call    00463A50                         ; \CLBD.00463A50  
004D0BEA   .  6A 04         push    4                                 ; /Arg1 = 00000004  
004D0BEC   .  8D4D F4       lea     ecx, [ebp-C]                     ; |  
004D0BEF   .  BA 600E4D00   mov     edx, 004D0E60                   ; |ASCII "ThreadingModelnn"  
004D0BF4   .  8B45 FC       mov     eax, [ebp-4]                    ; |  
004D0BF7   .  E8 542EF9FF   call    00463A50                         ; \CLBD.00463A50  
004D0BFC   .  8B45 FC       mov     eax, [ebp-4]  
004D0BFF   .  E8 342BF9FF   call    00463738  
004D0C04   .  33C0          xor     eax, eax  
004D0C06   .  5A            pop     edx  
004D0C07   .  59            pop     ecx  
004D0C08   .  59            pop     ecx  
004D0C09   .  64:8910       mov     fs:[eax], edx  
004D0C0C   .  68 8F0C4D00   push    004D0C8F  
004D0C11   >  8B45 FC       mov     eax, [ebp-4]  
004D0C14   .  E8 FB2AF3FF   call    00403714  
004D0C19   .  C3            retn  
004D0C1A   .^ E9 8932F3FF   jmp     00403EA8  
004D0C1F   .^ EB F0         jmp     short 004D0C11  
004D0C21   >  8D55 AC       lea     edx, [ebp-54]  
004D0C24   .  8B83 04030000 mov     eax, [ebx+304]  
004D0C2A   .  E8 510AF7FF   call    00441680  
004D0C2F   .  8B45 AC       mov     eax, [ebp-54]  
004D0C32   .  8D55 B0       lea     edx, [ebp-50]  
004D0C35   .  E8 3A7DF3FF   call    00408974  
004D0C3A   .  8B45 B0       mov     eax, [ebp-50]  
004D0C3D   .  E8 7A3BF3FF   call    004047BC  
004D0C42   .  83F8 04       cmp     eax, 4  
004D0C45   .  7D 12         jge     short 004D0C59  
004D0C47   .  8B83 00030000 mov     eax, [ebx+300]  
004D0C4D   .  BA D40E4D00   mov     edx, 004D0ED4  
004D0C52   .  E8 590AF7FF   call    004416B0  
004D0C57   .  EB 36         jmp     short 004D0C8F  
004D0C59   >  8D55 A4       lea     edx, [ebp-5C]  
004D0C5C   .  8B83 0C030000 mov     eax, [ebx+30C]  
004D0C62   .  E8 190AF7FF   call    00441680  
004D0C67   .  8B45 A4       mov     eax, [ebp-5C]  
004D0C6A   .  8D55 A8       lea     edx, [ebp-58]  
004D0C6D   .  E8 027DF3FF   call    00408974  
004D0C72   .  8B45 A8       mov     eax, [ebp-58]  
004D0C75   .  E8 423BF3FF   call    004047BC  
004D0C7A   .  83F8 11       cmp     eax, 11  
004D0C7D   .  74 10         je      short 004D0C8F  
004D0C7F   .  8B83 00030000 mov     eax, [ebx+300]  
004D0C85   .  BA 100F4D00   mov     edx, 004D0F10  
004D0C8A   .  E8 210AF7FF   call    004416B0  
004D0C8F   >  33C0          xor     eax, eax  
004D0C91   .  5A            pop     edx  
004D0C92   .  59            pop     ecx  
004D0C93   .  59            pop     ecx  
004D0C94   .  64:8910       mov     fs:[eax], edx  
004D0C97   .  68 410D4D00   push    004D0D41  
004D0C9C   >  8D45 A4       lea     eax, [ebp-5C]  
004D0C9F   .  E8 6038F3FF   call    00404504  
004D0CA4   .  8D45 A8       lea     eax, [ebp-58]  
004D0CA7   .  E8 5838F3FF   call    00404504  
004D0CAC   .  8D45 AC       lea     eax, [ebp-54]  
004D0CAF   .  E8 5038F3FF   call    00404504  
004D0CB4   .  8D45 B0       lea     eax, [ebp-50]  
004D0CB7   .  BA 02000000   mov     edx, 2  
004D0CBC   .  E8 6738F3FF   call    00404528  
004D0CC1   .  8D45 B8       lea     eax, [ebp-48]  
004D0CC4   .  E8 3B38F3FF   call    00404504  
004D0CC9   .  8D45 BC       lea     eax, [ebp-44]  
004D0CCC   .  E8 3338F3FF   call    00404504  
004D0CD1   .  8D45 C0       lea     eax, [ebp-40]  
004D0CD4   .  E8 2B38F3FF   call    00404504  
004D0CD9   .  8D45 C4       lea     eax, [ebp-3C]  
004D0CDC   .  E8 2338F3FF   call    00404504  
004D0CE1   .  8D45 C8       lea     eax, [ebp-38]  
004D0CE4   .  E8 1B38F3FF   call    00404504  
004D0CE9   .  8D45 CC       lea     eax, [ebp-34]  
004D0CEC   .  E8 1338F3FF   call    00404504  
004D0CF1   .  8D45 D0       lea     eax, [ebp-30]  
004D0CF4   .  E8 0B38F3FF   call    00404504  
004D0CF9   .  8D45 D4       lea     eax, [ebp-2C]  
004D0CFC   .  BA 02000000   mov     edx, 2  
004D0D01   .  E8 2238F3FF   call    00404528  
004D0D06   .  8D45 DC       lea     eax, [ebp-24]  
004D0D09   .  E8 F637F3FF   call    00404504  
004D0D0E   .  8D45 E0       lea     eax, [ebp-20]  
004D0D11   .  E8 EE37F3FF   call    00404504  
004D0D16   .  8D45 E4       lea     eax, [ebp-1C]  
004D0D19   .  E8 E637F3FF   call    00404504  
004D0D1E   .  8D45 E8       lea     eax, [ebp-18]  
004D0D21   .  E8 DE37F3FF   call    00404504  
004D0D26   .  8D45 EC       lea     eax, [ebp-14]  
004D0D29   .  E8 D637F3FF   call    00404504  
004D0D2E   .  8D45 F0       lea     eax, [ebp-10]  
004D0D31   .  E8 CE37F3FF   call    00404504  
004D0D36   .  C3            retn  
004D0D37   .^ E9 6C31F3FF   jmp     00403EA8  
004D0D3C   .^ E9 5BFFFFFF   jmp     004D0C9C  
004D0D41   .  5E            pop     esi  
004D0D42   .  5B            pop     ebx  
004D0D43   .  8BE5          mov     esp, ebp  
004D0D45   .  5D            pop     ebp  
004D0D46   .  C3            retn  
```

`-------------- 在这里跟进  004D0A4E   .  E8 01F9FFFF   call    004D0354 到这里 ----------------`

```asm
004D0354  /$  55           push    ebp  
004D0355  |.  8BEC         mov     ebp, esp  
```



004D0357 `|. 51` `push ecx`  

004D0358 `|. B9 05000000` `mov ecx, 5`  

004D035D `|> 6A 00` `/push 0`  

004D035F `|. 6A 00` `|push 0`  

004D0361 `|. 49` `|dec ecx`  

004D0362 `|.^ 75 F9` `\jnz short 004D035D`  

004D0364 `|. 874D FC` `xchg [ebp-4], ecx`  

004D0367 `|. 53` `push ebx`  

004D0368 `|. 56` `push esi`  

004D0369 `|. 57` `push edi`  

004D036A `|. 8BF9` `mov edi, ecx`  

004D036C `|. 8955 F8` `mov [ebp-8], edx` ; `(ASCII "651645186G01")`  

004D036F `|. 8945 FC` `mov [ebp-4], eax` ; `用户名`  

004D0372 `|. 8B45 FC` `mov eax, [ebp-4]`  

004D0375 `|. E8 2A46F3FF` `call 004049A4`  

004D037A `|. 8B45 F8` `mov eax, [ebp-8]`  

004D037D `|. E8 2246F3FF` `call 004049A4`  

004D0382 `|. 33C0` `xor eax, eax`  

004D0384 `|. 55` `push ebp`  

004D0385 `|. 68 24054D00` `push 004D0524`  

004D038A `|. 64:FF30` `push dword ptr fs:[eax]`  

004D038D `|. 64:8920` `mov fs:[eax], esp`  

004D0390 `|. 8D45 F0` `lea eax, [ebp-10]`  

004D0393 `|. E8 6C41F3FF` `call 00404504`  

004D0398 `|. 8D55 E8` `lea edx, [ebp-18]`  

004D039B `|. 8B45 F8` `mov eax, [ebp-8]`  

004D039E `|. E8 D185F3FF` `call 00408974`  

004D03A3 `|. FF75 E8` `push dword ptr [ebp-18]`  

004D03A6 `|. 8D55 E4` `lea edx, [ebp-1C]`  

004D03A9 `|. 8B45 FC` `mov eax, [ebp-4]`  

004D03AC `|. E8 C385F3FF` `call 00408974`  

004D03B1 `|. FF75 E4` `push dword ptr [ebp-1C]`  

004D03B4 `|. 68 3C054D00` `push 004D053C` ; `clbd`  

004D03B9 `|. 8D45 EC` `lea eax, [ebp-14]`  

004D03BC `|. BA 03000000` `mov edx, 3`  

004D03C1 `|. E8 B644F3FF` `call 0040487C` ; `拼接字串（机器码＋用户名＋clbd）`  

004D03C6 `|. 8B45 EC` `mov eax, [ebp-14]` ; `(ASCII "651645186G01lzq1973clbd")`  

004D03C9 `|. 8D4D F4` `lea ecx, [ebp-C]`  

004D03CC `|. BA 4C054D00` `mov edx, 004D054C` ; `ourclbd`  

004D03D1 `|. E8 D6F9FFFF` `call 004CFDAC`  

004D03D6 `|. 8D4D E0` `lea ecx, [ebp-20]`  

004D03D9 `|. BA 5C054D00` `mov edx, 004D055C` ; `qwer`  

004D03DE `|. 8B45 F4` `mov eax, [ebp-C]`  

004D03E1 `|. E8 56F2FFFF` `call 004CF63C`  

004D03E6 `|. 8B55 E0` `mov edx, [ebp-20]` ; `(ASCII 26,"ifutxpE}pswnakmrpx}t}iqspz~rzrttxs{ADp")`  

004D03E9 `|. 8D45 F4` `lea eax, [ebp-C]`  

004D03EC `|. E8 AB41F3FF` `call 0040459C`  

004D03F1 `|. 8B45 F4` `mov eax, [ebp-C]` ; `(ASCII 26,"ifutxpE}pswnakmrpx}t}iqspz~rzrttxs{ADp")`  

004D03F4 `|. E8 C343F3FF` `call 004047BC`  

004D03F9 `|. 50` `push eax`  

004D03FA `|. 8B45 F4` `mov eax, [ebp-C]`  

004D03FD `|. E8 B245F3FF` `call 004049B4`  

004D0402 `|. 5A` `pop edx`  

004D0403 `|. E8 18F0FFFF` `call 004CF420` ; `MD5 加密`  

004D0408 `|. 8D45 F4` `lea eax, [ebp-C]`  

004D040B `|. 8B15 E0534D00` `mov edx, [4D53E0]` ; `CLBD.004D78EC`  

004D0411 `|. 8B12` `mov edx, [edx]` ; `(ASCII "28F7199A69235E531A088B90AC63390F")`  

004D0413 `|. E8 8441F3FF` `call 0040459C`  

004D0418 `|. 8D55 DC` `lea edx, [ebp-24]`  

004D041B `|. 8B45 F4` `mov eax, [ebp-C]` ; `(ASCII "28F7199A69235E531A088B90AC63390F")`  

004D041E `|. E8 45E2FFFF` `call 004CE668`  

004D0423 `|. 8B55 DC` `mov edx, [ebp-24]`  

004D0426 `|. 8D45 F4` `lea eax, [ebp-C]`  

004D0429 `|. E8 6E41F3FF` `call 0040459C`  

004D042E `|. 8B45 F4` `mov eax, [ebp-C]`  

004D0431 `|. E8 8643F3FF` `call 004047BC`  

004D0436 `|. 83F8 0F` `cmp eax, 0F`  

004D0439 `|. 7D 0B` `jge short 004D0446`  

004D043B `|. 8D45 F4` `lea eax, [ebp-C]`  

004D043E `|. 8B55 F8` `mov edx, [ebp-8]`  

004D0441 `|. E8 7E43F3FF` `call 004047C4`  

004D0446 `|> 8B45 F4` `mov eax, [ebp-C]`  

004D0449 `|. E8 6E43F3FF` `call 004047BC`  

004D044E `|. 83F8 0F` `cmp eax, 0F`  

004D0451 `|. 7E 16` `jle short 004D0469`  

004D0453 `|. 8D45 F4` `lea eax, [ebp-C]`  

004D0456 `|. 50` `push eax`  

004D0457 `|. B9 0F000000` `mov ecx, 0F` ; `取前 15 位`  

004D045C `|. BA 01000000` `mov edx, 1`  

004D0461 `|. 8B45 F4` `mov eax, [ebp-C]`  

004D0464 `|. E8 AB45F3FF` `call 00404A14`  

004D0469 `|> 8B45 F4` `mov eax, [ebp-C]` ; `(ASCII "28F7199A69235E5")`  

004D046C `|. E8 4B43F3FF` `call 004047BC`  

004D0471 `|. 8BF0` `mov esi, eax`  

004D0473 `|. 85F6` `test esi, esi`  

004D0475 `|. 7E 42` `jle short 004D04B9`  

004D0477 `|. BB 01000000` `mov ebx, 1`  

004D047C `|> 8D45 D8` `/lea eax, [ebp-28]` ; `/注册码每 5 位一组中间用—连接`  

004D047F `|. 50` `|push eax`  

004D0480 `|. B9 01000000` `|mov ecx, 1`  

004D0485 `|. 8BD3` `|mov edx, ebx`  

004D0487 `|. 8B45 F4` `|mov eax, [ebp-C]`  

004D048A `|. E8 8545F3FF` `|call 00404A14`  

004D048F `|. 8B55 D8` `|mov edx, [ebp-28]`  

004D0492 `|. 8D45 F0` `|lea eax, [ebp-10]`  

004D0495 `|. E8 2A43F3FF` `|call 004047C4`  

004D049A `|. 8BC3` `|mov eax, ebx`  

004D049C `|. B9 05000000` `|mov ecx, 5`  

004D04A1 `|. 99` `|cdq`  

004D04A2 `|. F7F9` `|idiv ecx`  

004D04A4 `|. 85D2` `|test edx, edx`  

004D04A6 `|. 75 0D` `|jnz short 004D04B5`  

004D04A8 `|. 8D45 F0` `|lea eax, [ebp-10]`  

004D04AB `|. BA 6C054D00` `|mov edx, 004D056C` ; `-`  

004D04B0 `|. E8 0F43F3FF` `|call 004047C4`  

004D04B5 `|> 43` `|inc ebx`  

004D04B6 `|. 4E` `|dec esi`  

004D04B7 `|.^ 75 C3` `\jnz short 004D047C` ; `\循环`  

004D04B9 `|> 8D45 D4` `lea eax, [ebp-2C]`  

004D04BC `|. 50` `push eax`  

004D04BD `|. 8B45 F0` `mov eax, [ebp-10]` ; `(ASCII "28F71-99A69-235E5-")`  

004D04C0 `|. E8 F742F3FF` `call 004047BC`  

004D04C5 `|. 8BD0` `mov edx, eax`  

004D04C7 `|. B9 01000000` `mov ecx, 1`  

004D04CC `|. 8B45 F0` `mov eax, [ebp-10]`  

004D04CF `|. E8 4045F3FF` `call 00404A14`  

004D04D4 `|. 8B45 D4` `mov eax, [ebp-2C]`  

004D04D7 `|. BA 6C054D00` `mov edx, 004D056C` ; `-`  

004D04DC `|. E8 1F44F3FF` `call 00404900`  

004D04E1 `|. 75 1C` `jnz short 004D04FF`  

004D04E3 `|. 8D45 F0` `lea eax, [ebp-10]`  

004D04E6 `|. 50` `push eax`  

004D04E7 `|. 8B45 F0` `mov eax, [ebp-10]`  

004D04EA `|. E8 CD42F3FF` `call 004047BC`  

004D04EF `|. 8BC8` `mov ecx, eax`  

004D04F1 `|. 49` `dec ecx`  

004D04F2 `|. BA 01000000` `mov edx, 1`  

004D04F7 `|. 8B45 F0` `mov eax, [ebp-10]`  

004D04FA `|. E8 1545F3FF` `call 00404A14`  

004D04FF `|> 8BC7` `mov eax, edi`  

004D0501 `|. 8B55 F0` `mov edx, [ebp-10]` ; `注册码(ASCII "28F71-99A69-235E5")`  

004D0504 `|. E8 4F40F3FF` `call 00404558`  

004D0509 `|. 33C0` `xor eax, eax`  

004D050B `|. 5A` `pop edx`  

004D050C `|. 59` `pop ecx`  

004D050D `|. 59` `pop ecx`  

004D050E `|. 64:8910` `mov fs:[eax], edx`  

004D0511 `|. 68 2B054D00` `push 004D052B`  

004D0516 `|> 8D45 D4` `lea eax, [ebp-2C]`  

004D0519 `|. BA 0B000000` `mov edx, 0B`  

004D051E `|. E8 0540F3FF` `call 00404528`  



### 004D0523

```
004D0523   \\.  C3            retn  

004D0524    .^ E9 7F39F3FF   jmp     00403EA8  

004D0529    .^ EB EB         jmp     short 004D0516  

004D052B    .  5F            pop     edi  

004D052C    .  5E            pop     esi  

004D052D    .  5B            pop     ebx  

004D052E    .  8BE5          mov     esp, ebp  

004D0530    .  5D            pop     ebp  

004D0531    .  C3            retn  
```

\------------------ 下面的要用到上面所说的用户名，这才是真的注册码 ------------------------

### 004D1BF0

```
004D1BF0  /.  55            push    ebp  

004D1BF1  |.  8BEC          mov     ebp, esp  

004D1BF3  |.  81C4 6CFEFFFF add     esp, -194  

004D1BF9  |.  33C9          xor     ecx, ecx  

004D1BFB  |.  898D 78FEFFFF mov     [ebp-188], ecx  

004D1C01  |.  898D 6CFEFFFF mov     [ebp-194], ecx  

004D1C07  |.  898D 74FEFFFF mov     [ebp-18C], ecx  

004D1C0D  |.  898D 70FEFFFF mov     [ebp-190], ecx  

004D1C13  |.  8945 FC       mov     [ebp-4], eax  

004D1C16  |.  33C0          xor     eax, eax  

004D1C18  |.  55            push    ebp  

004D1C19  |.  68 E81F4D00   push    004D1FE8  

004D1C1E  |.  64:FF30       push    dword ptr fs:[eax]  

004D1C21  |.  64:8920       mov     fs:[eax], esp  

004D1C24  |.  BA 6E000000   mov     edx, 6E  

004D1C29  |.  A1 10794D00   mov     eax, [4D7910]  

004D1C2E  |.  E8 09F2F6FF   call    00440E3C  

004D1C33  |.  8B45 FC       mov     eax, [ebp-4]  

004D1C36  |.  8B80 F4020000 mov     eax, [eax+2F4]  

004D1C3C  |.  E8 A721F9FF   call    00463DE8  

004D1C41  |.  B2 01         mov     dl, 1  

004D1C43  |.  A1 C8354600   mov     eax, [4635C8]  

004D1C48  |.  E8 7B1AF9FF   call    004636C8  

004D1C4D  |.  8945 F8       mov     [ebp-8], eax  

004D1C50  |.  33C0          xor     eax, eax  

004D1C52  |.  55            push    ebp  

004D1C53  |.  68 881D4D00   push    004D1D88  

004D1C58  |.  64:FF30       push    dword ptr fs:[eax]  

004D1C5B  |.  64:8920       mov     fs:[eax], esp  

004D1C5E  |.  BA 02000080   mov     edx, 80000002  

004D1C63  |.  8B45 F8       mov     eax, [ebp-8]  

004D1C66  |.  E8 FD1AF9FF   call    00463768  

004D1C6B  |.  B1 01         mov     cl, 1  

004D1C6D  |.  BA FC1F4D00   mov     edx, 004D1FFC                  ;  \\software\\classes\\clsid\\{d0358889-e67b-f753-48bf-135864321dca}\\inprocserver32  

004D1C72  |.  8B45 F8       mov     eax, [ebp-8]  

004D1C75  |.  E8 521BF9FF   call    004637CC  

004D1C7A  |.  BA 54204D00   mov     edx, 004D2054                  ;  threadingmodel  

004D1C7F  |.  8B45 F8       mov     eax, [ebp-8]  

004D1C82  |.  E8 1D1FF9FF   call    00463BA4  

004D1C87  |.  84C0          test    al, al  

004D1C89  |.  0F84 AB000000 je      004D1D3A  

004D1C8F  |.  68 74010000   push    174                             ; /Arg1 = 00000174  

004D1C94  |.  8D8D 7CFEFFFF lea     ecx, [ebp-184]                  ; |  

004D1C9A  |.  BA 54204D00   mov     edx, 004D2054                  ; |threadingmodel  

004D1C9F  |.  8B45 F8       mov     eax, [ebp-8]                    ; |  

004D1CA2  |.  E8 BD1DF9FF   call    00463A64                        ; \\CLBD.00463A64  

004D1CA7  |.  8D85 74FEFFFF lea     eax, [ebp-18C]  

004D1CAD  |.  E8 1AE5FFFF   call    004D01CC  

004D1CB2  |.  8B85 74FEFFFF mov     eax, [ebp-18C]  

004D1CB8  |.  50            push    eax                               ;  (ASCII "65P64F182G01")  

004D1CB9  |.  8D85 70FEFFFF lea     eax, [ebp-190]  

004D1CBF  |.  8D95 7CFEFFFF lea     edx, [ebp-184]                  ;  (ASCII 07,"lzq1973")  

004D1CC5  |.  E8 962AF3FF   call    00404760  

004D1CCA  |.  8B85 70FEFFFF mov     eax, [ebp-190]  

004D1CD0  |.  8D8D 78FEFFFF lea     ecx, [ebp-188]  

004D1CD6  |.  5A            pop     edx  

004D1CD7  |.  E8 78E6FFFF   call    004D0354  

004D1CDC  |.  8B85 78FEFFFF mov     eax, [ebp-188]  

004D1CE2  |.  50            push    eax                               ;  (ASCII "CAF40-06649-CD569")  

004D1CE3  |.  8D85 6CFEFFFF lea     eax, [ebp-194]  

004D1CE9  |.  8D95 D5FEFFFF lea     edx, [ebp-12B]                  ;  (ASCII 11,"28F71-99A69-235E5")  

004D1CEF  |.  E8 6C2AF3FF   call    00404760  

004D1CF4  |.  8B95 6CFEFFFF mov     edx, [ebp-194]  

004D1CFA  |.  58            pop     eax  

004D1CFB  |.  E8 002CF3FF   call    00404900                          ;  这里做内存注册（真正的）EAX  

004D1D00  |.  75 06         jnz     short 004D1D08  

004D1D02  |.  807D EC 00    cmp     byte ptr [ebp-14], 0  

004D1D06  |.  75 62         jnz     short 004D1D6A  

004D1D08  |>  A1 BC534D00   mov     eax, [4D53BC]  

004D1D0D  |.  8338 00       cmp     dword ptr [eax], 0  

004D1D10  |.  75 17         jnz     short 004D1D29  

004D1D12  |.  8B4D FC       mov     ecx, [ebp-4]  

004D1D15  |.  B2 01         mov     dl, 1  

004D1D17  |.  A1 08FA4C00   mov     eax, [4CFA08]  

004D1D1C  |.  E8 B38CF8FF   call    0045A9D4  

004D1D21  |.  8B15 BC534D00 mov     edx, [4D53BC]                    ;  CLBD.004D78FC  

004D1D27  |.  8902          mov     [edx], eax  

004D1D29  |>  A1 BC534D00   mov     eax, [4D53BC]  

004D1D2E  |.  8B00          mov     eax, [eax]  

004D1D30  |.  8B10          mov     edx, [eax]  

004D1D32  |.  FF92 E8000000 call    [edx+E8]  

004D1D38  |.  EB 30         jmp     short 004D1D6A  

004D1D3A  |>  A1 BC534D00   mov     eax, [4D53BC]  

004D1D3F  |.  8338 00       cmp     dword ptr [eax], 0  

004D1D42  |.  75 17         jnz     short 004D1D5B  

004D1D44  |.  8B4D FC       mov     ecx, [ebp-4]  

004D1D47  |.  B2 01         mov     dl, 1  

004D1D49  |.  A1 08FA4C00   mov     eax, [4CFA08]  

004D1D4E  |.  E8 818CF8FF   call    0045A9D4  

004D1D53  |.  8B15 BC534D00 mov     edx, [4D53BC]                    ;  CLBD.004D78FC  

004D1D59  |.  8902          mov     [edx], eax  

004D1D5B  |>  A1 BC534D00   mov     eax, [4D53BC]  

004D1D60  |.  8B00          mov     eax, [eax]  

004D1D62  |.  8B10          mov     edx, [eax]  

004D1D64  |.  FF92 E8000000 call    [edx+E8]  

004D1D6A  |>  8B45 F8       mov     eax, [ebp-8]  

004D1D6D  |.  E8 C619F9FF   call    00463738  

004D1D72  |.  33C0          xor     eax, eax  

004D1D74  |.  5A            pop     edx  

004D1D75  |.  59            pop     ecx  

004D1D76  |.  59            pop     ecx  

004D1D77  |.  64:8910       mov     fs:[eax], edx  

004D1D7A  |.  68 8F1D4D00   push    004D1D8F  

004D1D7F  |>  8B45 F8       mov     eax, [ebp-8]  

004D1D82  |.  E8 8D19F3FF   call    00403714  

004D1D87  \\.  C3            retn  
```

明码比较，寻码还是很简单的。

\------------------------------------------------------------------------

### 【破解总结】

1、字符串 `A`；（机器码＋用户名＋`clbd`）

2、取 `A` 的偶数位组成新的字串 `G`；

3、`A` + `G` 组成字串 `B`；

4、`B` 的各字符与 `ourclbd` 的各字符（不够位循环）进行 `XOR` 运算；

4、`B` 的各字符与 `qwer` 的各字符（不够位循环）进行 `OR` 运算；

5、组成新的字串后再进行 `MD5` 加密（32 位）；

6、取前 15 位，每 5 位字间用—连接，就是注册码。

注：用户名长度 4 位以上，注册码长度 17 位，这里的“＋”为连接符。

### 注册说明：

    运行“词林宝典一次.exe”，填入相关内容注册成功后退出；运行“词林宝典二次.exe”，复制截取到的注册码，退出程序；运行主程序，输入第一次用到的用户名，粘贴第二次截取到的注册码；OK!!!

\------------------------------------------------------------------------

### 【版权声明】

本文纯属技术交流，转载请注明作者信息并保持文章的完整，谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
