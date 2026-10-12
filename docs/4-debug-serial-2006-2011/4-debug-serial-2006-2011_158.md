
# 定时播映系统

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-155.htm`](https://www.pediy.com/kssd/pediy08/pediy8-155.htm)

【破文标题】菜鸟破解之三“定时播映系统的注册算法分析”  
【破文作者】紫色缘`[TFW]``[PCG]`  
【作者邮箱】`Cn_Fish@126.com`  
【作者主页】`www.cniso.org`  
【破解工具】`OD`/`peid`  
【破解平台】`Win9x`/`NT`/`2000`/`XP`  
【软件名称】****播放系统  
【软件大小】2MB  
【原版下载】不提供~  
【保护方式】无  
【软件简介】功能不错的一个播放系统  

------------------------------------------------------------------------  

【破解过程】  

第一先启动主程序，在输入假码后出现错误提示框。  

再使用 `Peid` 查壳后，发现是 `Borland Delphi 6.0 - 7.0` 语言编写，于是用 `OD` 载入查找到字符：  

超级字串参考＋        , 条目 835  

 地址=`00500E39`  

 反汇编=`push 00500FAC`  

 文本字串=`错误 86599`  

双击后来到 `500E39` 代码处，想分析一下注册的算法过程，即向上找到事件触发点~  

--------------------代码如下-----------------------  

```
00500C62   .  55              push    ebp                            ;  在此下断，F8 继续  
00500C63   .  68 B10E5000     push    00500EB1  
00500C68   .  64:FF30         push    dword ptr fs:[eax]  
00500C6B   .  64:8920         mov     fs:[eax], esp  
00500C6E   .  A1 34A35100     mov     eax, [51A334]                   ;  [51A334]送入 EAX  
00500C73   .  C700 8B140000   mov     dword ptr [eax], 148B           ;  148B 送入[EAX]  
00500C79   .  C740 04 0000000>mov     dword ptr [eax+4], 0  
00500C80   .  A1 5CA05100     mov     eax, [51A05C]                   ;  [51A05C]送入 EAX  
00500C85   .  8B00            mov     eax, [eax]                      ;  [EAX]送入 EAX  
00500C87   .  8B80 A4030000   mov     eax, [eax+3A4]  
00500C8D   .  66:BE EBFF      mov     si, 0FFEB                       ;  SI=FFEB  
00500C91   .  E8 8633F0FF     call    0040401C                        ;  关键 call,跟进  
00500C96   .  8D55 F8         lea     edx, [ebp-8]  
00500C99   .  8B45 FC         mov     eax, [ebp-4]  
00500C9C   .  8B80 00030000   mov     eax, [eax+300]  
00500CA2   .  E8 3159F5FF     call    004565D8                        ;  读取假码位数  
00500CA7   .  8B45 F8         mov     eax, [ebp-8]                    ;  假码位数送入 EAX  
00500CAA   .  BA 140F5000     mov     edx, 00500F14                   ;  999999999999  
00500CAF   .  E8 5843F0FF     call    0040500C                        ;  EAX 与 EDX 做比较  
00500CB4   .  0F83 B6010000   jnb     00500E70                        ;  大于则跳  
00500CBA   .  8D55 F4         lea     edx, [ebp-C]  
00500CBD   .  8B45 FC         mov     eax, [ebp-4]  
00500CC0   .  8B80 00030000   mov     eax, [eax+300]  
00500CC6   .  E8 0D59F5FF     call    004565D8                        ;  读取假码位数  
00500CCB   .  8B45 F4         mov     eax, [ebp-C]                    ;  假码位数送入 EAX  
00500CCE   .  BA 2C0F5000     mov     edx, 00500F2C                   ;  0000000000  
00500CD3   .  E8 3443F0FF     call    0040500C                        ;  EAX 与 EDX 做比较  
00500CD8   .  0F86 92010000   jbe     00500E70                        ;  小于或等于则跳转到注册码不正确  
00500CDE   .  8D55 F0         lea     edx, [ebp-10]  
00500CE1   .  8B45 FC         mov     eax, [ebp-4]  
00500CE4   .  8B80 00030000   mov     eax, [eax+300]  
00500CEA   .  E8 E958F5FF     call    004565D8  
00500CEF   .  8B45 F0         mov     eax, [ebp-10]                   ;  假码位数送入 EAX  
00500CF2   .  50              push    eax                              ;  EAX 入栈  
00500CF3   .  A1 34A35100     mov     eax, [51A334]                   ;  [51A334]送入 EAX  
00500CF8   .  FF70 04        push    dword ptr [eax+4]                ; /Arg2  
00500CFB   .  FF30           push    dword ptr [eax]                  ; |此处出现真码 16 进制  
00500CFD   .  8D45 EC         lea     eax, [ebp-14]                    ; |  
00500D00   .  E8 5B8CF0FF     call    00409960                        ; \\转为 10 进制%d  
00500D05   .  8B55 EC         mov     edx, [ebp-14]                   ;  真码入 EDX  
00500D08   .  58              pop     eax                              ;  假码弹出  
00500D09   .  E8 FE42F0FF     call    0040500C                        ;  经典比较/可做内存注册机  
00500D0E   .  0F85 23010000   jnz     00500E37                        ;  此处爆破 nop~  
00500D14   .  A1 34A35100     mov     eax, [51A334]  
00500D19   .  FF70 04        push    dword ptr [eax+4]                ; /Arg2  
00500D1C   .  FF30           push    dword ptr [eax]                  ; |Arg1  
00500D1E   .  8D45 E8         lea     eax, [ebp-18]                    ; |  
00500D21   .  E8 3A8CF0FF     call    00409960                        ; \\dsbyxt.00409960  
00500D26   .  8B55 E8         mov     edx, [ebp-18]  
00500D29   .  A1 80A15100     mov     eax, [51A180]  
00500D2E   .  83C0 04        add     eax, 4  
00500D31   .  E8 2E3FF0FF     call    00404C64  
00500D36   .  A1 5CA05100     mov     eax, [51A05C]  
00500D3B   .  8B00            mov     eax, [eax]  
00500D3D   .  8B80 90030000   mov     eax, [eax+390]  
00500D43   .  66:BE EBFF      mov     si, 0FFEB  
00500D47   .  E8 D032F0FF     call    0040401C  
00500D4C   .  A1 34A35100     mov     eax, [51A334]  
00500D51   .  C700 05000000   mov     dword ptr [eax], 5  
00500D57   .  C740 04 0000000>mov     dword ptr [eax+4], 0  
00500D5E   .  6A 40           push    40                               ; /Style = MB_OK|MB_ICONASTERISK|MB_APPLMODAL  
00500D60   .  68 380F5000     push    00500F38                        ; |系统信息  
00500D65   .  68 440F5000     push    00500F44                        ; |注册成功!\\n\\n 你将获得本软件终身免费升级权。\\n\\n 感谢你我使用**播放软件!  
00500D6A   .  6A 00           push    0                                ; |hOwner = NULL  
00500D6C   .  E8 3F6FF0FF     call    <jmp.&user32.MessageBoxA>        ; \\MessageBoxA  
00500D71   .  A1 5CA05100     mov     eax, [51A05C]  
00500D76   .  8B00            mov     eax, [eax]  
00500D78   .  8B80 5C030000   mov     eax, [eax+35C]  
00500D7E   .  33D2            xor     edx, edx  
00500D80   .  E8 13BBF6FF     call    0046C898  
00500D85   .  B2 01           mov     dl, 1  
00500D87   .  A1 D0854900     mov     eax, [4985D0]  
00500D8C   .  E8 9779F9FF     call    00498728  
00500D91   .  A3 30BE5100     mov     [51BE30], eax  
00500D96   .  8B15 18A35100   mov     edx, [51A318]                   ;  dsbyxt.0051C188  
00500D9C   .  8B12            mov     edx, [edx]  
00500D9E   .  A1 30BE5100     mov     eax, [51BE30]  
00500DA3   .  E8 607AF9FF     call    00498808  
00500DA8   .  8B15 70A35100   mov     edx, [51A370]                   ;  dsbyxt.0051C184  
00500DAE   .  8B12            mov     edx, [edx]  
00500DB0   .  8D45 E4         lea     eax, [ebp-1C]  
00500DB3   .  B9 940F5000     mov     ecx, 00500F94                   ;  markjl  
00500DB8   .  E8 5741F0FF     call    00404F14  
00500DBD   .  8B55 E4         mov     edx, [ebp-1C]  
00500DC0   .  33C9            xor     ecx, ecx  
00500DC2   .  A1 30BE5100     mov     eax, [51BE30]  
00500DC7   .  E8 F07AF9FF     call    004988BC  
00500DCC   .  84C0            test    al, al  
00500DCE   .  74 3D           je      short 00500E0D  
00500DD0   .  BA A40F5000     mov     edx, 00500FA4                   ;  gc_id  
00500DD5   .  A1 30BE5100     mov     eax, [51BE30]  
00500DDA   .  E8 C17DF9FF     call    00498BA0  
00500DDF   .  84C0            test    al, al  
00500DE1   .  74 2A           je      short 00500E0D  
00500DE3   .  BA A40F5000     mov     edx, 00500FA4                   ;  gc_id  
00500DE8   .  A1 30BE5100     mov     eax, [51BE30]  
00500DED   .  E8 867CF9FF     call    00498A78  
00500DF2   .  3D F0000000     cmp     eax, 0F0  
00500DF7   .  7E 14           jle     short 00500E0D  
00500DF9   .  B9 C8000000     mov     ecx, 0C8  
00500DFE   .  BA A40F5000     mov     edx, 00500FA4                   ;  gc_id  
00500E03   .  A1 30BE5100     mov     eax, [51BE30]  
00500E08   .  E8 437CF9FF     call    00498A50  
00500E0D   >  A1 30BE5100     mov     eax, [51BE30]  
00500E12   .  E8 0930F0FF     call    00403E20  
00500E17   .  A1 5CA05100     mov     eax, [51A05C]  
00500E1C   .  8B00            mov     eax, [eax]  
```



### 00500E1E - 00500F0A 处代码

```
00500E1E   .  8B80 04030000   mov     eax, [eax+304]  

00500E24   .  B2 01           mov     dl, 1  

00500E26   .  E8 456FF4FF     call    00447D70  

00500E2B   .  A1 28BE5100     mov     eax, [51BE28]  

00500E30   .  E8 DB9AF7FF     call    0047A910  

00500E35   .  EB 70           jmp     short 00500EA7  

00500E37   >  6A 10           push    10                             ; /Style = MB_OK|MB_ICONHAND|MB_APPLMODAL  

00500E39   .  68 AC0F5000     push    00500FAC                       ; |错误 86599  

00500E3E   .  68 B80F5000     push    00500FB8                       ; |对不起，注册码不正确！  

00500E43   .  6A 00           push    0                              ; |hOwner = NULL  

00500E45   .  E8 666EF0FF     call    <jmp.&user32.MessageBoxA>      ; \\MessageBoxA  

00500E4A   .  8B45 FC         mov     eax, [ebp-4]  

00500E4D   .  8B80 00030000   mov     eax, [eax+300]  

00500E53   .  33D2           xor     edx, edx  

00500E55   .  E8 BE57F5FF     call    00456618  

00500E5A   .  A1 5CA05100     mov     eax, [51A05C]  

00500E5F   .  8B00           mov     eax, [eax]  

00500E61   .  8B80 04030000   mov     eax, [eax+304]  

00500E67   .  33D2           xor     edx, edx  

00500E69   .  E8 026FF4FF     call    00447D70  

00500E6E   .  EB 37           jmp     short 00500EA7  

00500E70   >  6A 10           push    10                             ; /Style = MB_OK|MB_ICONHAND|MB_APPLMODAL  

00500E72   .  68 D00F5000     push    00500FD0                       ; |错误 efgkc  

00500E77   .  68 B80F5000     push    00500FB8                       ; |对不起，注册码不正确！  

00500E7C   .  6A 00           push    0                              ; |hOwner = NULL  

00500E7E   .  E8 2D6EF0FF     call    <jmp.&user32.MessageBoxA>      ; \\MessageBoxA  

00500E83   .  8B45 FC         mov     eax, [ebp-4]  

00500E86   .  8B80 00030000   mov     eax, [eax+300]  

00500E8C   .  33D2           xor     edx, edx  

00500E8E   .  E8 8557F5FF     call    00456618  

00500E93   .  A1 5CA05100     mov     eax, [51A05C]  

00500E98   .  8B00           mov     eax, [eax]  

00500E9A   .  8B80 04030000   mov     eax, [eax+304]  

00500EA0   .  33D2           xor     edx, edx  

00500EA2   .  E8 C96EF4FF     call    00447D70  

00500EA7   >  33C0           xor     eax, eax  

00500EA9   .  5A             pop     edx  

00500EAA   .  59             pop     ecx  

00500EAB   .  59             pop     ecx  

00500EAC   .  64:8910        mov     fs:[eax], edx  

00500EAF   .  EB 14           jmp     short 00500EC5  

00500EB1   .^ E9 4A34F0FF    jmp     00404300  

00500EB6   .  A1 28BE5100     mov     eax, [51BE28]  

00500EBB   .  E8 509AF7FF     call    0047A910  

00500EC0   .  E8 A337F0FF     call    00404668  

00500EC5   >  8B45 FC         mov     eax, [ebp-4]  

00500EC8   .  8B80 08030000   mov     eax, [eax+308]  

00500ECE   .  33D2           xor     edx, edx  

00500ED0   .  E8 4F55F5FF     call    00456424  

00500ED5   .  33C0           xor     eax, eax  

00500ED7   .  5A             pop     edx  

00500ED8   .  59             pop     ecx  

00500ED9   .  59             pop     ecx  

00500EDA   .  64:8910        mov     fs:[eax], edx  

00500EDD   .  68 040F5000    push    00500F04  

00500EE2   >  8D45 E4         lea     eax, [ebp-1C]  

00500EE5   .  BA 03000000     mov     edx, 3  

00500EEA   .  E8 453DF0FF     call    00404C34  

00500EEF   .  8D45 F0         lea     eax, [ebp-10]  

00500EF2   .  BA 03000000     mov     edx, 3  

00500EF7   .  E8 383DF0FF     call    00404C34  

00500EFC   .  C3             retn  

00500EFD   .^ E9 B236F0FF    jmp     004045B4  

00500F02   .^ EB DE         jmp     short 00500EE2  

00500F04   .  5F             pop     edi  

00500F05   .  5E             pop     esi  

00500F06   .  5B             pop     ebx  

00500F07   .  8BE5           mov     esp, ebp  

00500F09   .  5D             pop     ebp  

00500F0A   .  C3             retn  
```

### 跟进 500C91 处的代码如下

```
0040401C   $  50             push    eax  

0040401D   .  51             push    ecx  

0040401E   .  8B00           mov     eax, [eax]  

00404020   .  E8 C7FFFFFF    call    00403FEC                       ;  这里跟进  

00404025   .  59             pop     ecx  

00404026   .  58             pop     eax  

00404027   .  74 02         je      short 0040402B  

00404029   .  FFE6           jmp     esi  

0040402B   >  59             pop     ecx  

0040402C   .^ E9 AFECFFFF   jmp     00402CE0  

00404031   .  C3             retn  
```

### 跟进 404020 处的代码如下

其中这里有 2 个 `call`，虽然说地址跟进是一样，但是有 2 段代码....小弟不明白,请各位大侠指点一二~

```
00403FEC  /$  57             push    edi  

00403FED  |.  96             xchg    eax, esi  

00403FEE  |.  EB 02         jmp     short 00403FF2  

00403FF0  |>  8B36          /mov     esi, [esi]  

00403FF2  |>  8B7E D0       mov     edi, [esi-30]  

00403FF5  |.  85FF          |test    edi, edi  

00403FF7  |.  74 0D         |je      short 00404006  

00403FF9  |.  0FB70F       |movzx   ecx, word ptr [edi]  

00403FFC  |.  51           |push    ecx  

00403FFD  |.  83C7 02      |add     edi, 2  

00404000  |.  F2:66:AF     |repne   scas word ptr es:[edi]  

00404003  |.  74 0A         |je      short 0040400F  

00404005  |.  59           |pop     ecx  

00404006  |>  8B76 DC      |mov     esi, [esi-24]  

00404009  |.  85F6         |test    esi, esi  

0040400B  |.^ 75 E3        \\jnz     short 00403FF0  

0040400D  |.  5F           pop     edi  

0040400E  |.  C3           retn  

0040400F  |>  58           pop     eax  

00404010  |.  01C0         add     eax, eax  

00404012  |.  29C8         sub     eax, ecx  

00404014  |.  8B7447 FC    mov     esi, [edi+eax*2-4]  

00404018  |.  5F           pop     edi  

00404019  \\.  C3          retn  

0040401A      8BC0         mov     eax, eax  

0040401C   $  50             push    eax  

0040401D   .  51             push    ecx  

0040401E   .  8B00           mov     eax, [eax]  

00404020   .  E8 C7FFFFFF    call    00403FEC                       

00404025   .  59             pop     ecx  

00404026   .  58             pop     eax  

00404027   .  74 02         je      short 0040402B  

00404029   . /FFE6           jmp     esi                              ;  *****.0044055C  （注意这里直接跳转）  

0040402B   >  59             pop     ecx  

0040402C   .^ E9 AFECFFFF   jmp     00402CE0  

00404031   .  C3             retn  
```

中间省略 N 行代码~~~

```
0044055C  /$  55             push    ebp  

0044055D  |.  8BEC           mov     ebp, esp  

0044055F  |.  83C4 F8        add     esp, -8  

00440562  |.  8945 FC        mov     [ebp-4], eax  

00440565  |.  8B45 FC        mov     eax, [ebp-4]  

00440568  |.  E8 DB340300    call    00473A48  

0044056D  |.  8945 F8        mov     [ebp-8], eax  

00440570  |.  837D F8 00     cmp     dword ptr [ebp-8], 0  

00440574  |.  74 12         je      short 00440588  

00440576  |.  8B45 FC        mov     eax, [ebp-4]  

00440579  |.  8B80 14020000  mov     eax, [eax+214]  

0044057F  |.  8B55 F8       mov     edx, [ebp-8]  

00440582  |.  8982 4C020000 mov     [edx+24C], eax  

00440588  |>  8B45 FC        mov     eax, [ebp-4]  

0044058B  |.  E8 707D0100    call    00458300                       ;  算法  

00440590  |.  59             pop     ecx  

00440591  |.  59             pop     ecx  

00440592  |.  5D             pop     ebp  

00440593  \\.  C3            retn  
```

### 跟进 44058B 处的代码如下

中间省略 N 行代码~~~

```
00458363  |.  8B40 6C       mov     eax, [eax+6C]  

00458366  |.  8B08          mov     ecx, [eax]  

00458368  |.  FF51 18       call    [ecx+18]  
```



```
0045836B  |.  EB 1F           jmp     short 0045838C  

0045836D  |>  8B45 FC         mov     eax, [ebp-4]  

00458370  |.  66:83B8 2201000>cmp     word ptr [eax+122], 0  

00458378  |.  74 12           je      short 0045838C  

0045837A  |.  8B5D FC         mov     ebx, [ebp-4]  

0045837D  |.  8B55 FC         mov     edx, [ebp-4]  

00458380  |.  8B83 24010000   mov     eax, [ebx+124]  

00458386  |.  FF93 20010000   call    [ebx+120]                          ;  算法 call  

0045838C  |>  5B              pop     ebx  

0045838D  |.  59              pop     ecx  

0045838E  |.  5D              pop     ebp  

0045838F  |\. C3              retn  

------------------------跟进 458386 处的代码如下（算法 call）-----------------------------------------  

00511884  /.  55              push    ebp                            ;  跟到此,F8 继续  

00511885  |.  8BEC            mov     ebp, esp  

00511887  |.  6A 00           push    0  

00511889  |.  6A 00           push    0  

0051188B  |.  6A 00           push    0  

0051188D  |.  53              push    ebx  

0051188E  |.  56              push    esi  

0051188F  |.  57              push    edi  

00511890  |.  33C0            xor     eax, eax  

00511892  |.  55              push    ebp  

00511893  |.  68 DD1A5100     push    00511ADD  

00511898  |.  64:FF30         push    dword ptr fs:[eax]  

0051189B  |.  64:8920         mov     fs:[eax], esp  

0051189E  |.  8D45 FC         lea     eax, [ebp-4]  

005118A1  |.  8B15 C0BF5100   mov     edx, [51BFC0]  

005118A7  |.  E8 FC33EFFF     call    00404CA8  

005118AC  |.  8D45 F8         lea     eax, [ebp-8]  

005118AF  |.  E8 5C33EFFF     call    00404C10  

005118B4  |.  8B45 FC         mov     eax, [ebp-4]                     ;出硬盘序列号送入 EAX  

005118B7  |.  E8 0C36EFFF     call    00404EC8  

005118BC  |.  8BF0            mov     esi, eax                         ;  EAX -->ESI  

005118BE  |.  85F6            test    esi, esi                         ;  测试 ESI 是否小于 E  

005118C0  |.  0F8E 8D010000   jle     00511A53                         ;  小于则跳转  

005118C6  |.  BB 01000000     mov     ebx, 1  

005118CB  |>  8D45 F4         /lea     eax, [ebp-C]  

005118CE  |.  50              |push     eax  

005118CF  |.  B9 01000000     |mov      ecx, 1  

005118D4  |.  8BD3            |mov      edx, ebx  

005118D6  |.  8B45 FC         |mov      eax, [ebp-4]                   ;  硬盘序列号送入 EAX  

005118D9  |.  E8 4238EFFF     |call     00405120  

005118DE  |.  8B45 F4         |mov      eax, [ebp-C]  

005118E1  |.  BA F41A5100     |mov      edx, 00511AF4                  ;  9  

005118E6  |.  E8 2137EFFF     |call     0040500C  

005118EB  |.  0F87 B2000000   |ja       005119A3  

005118F1  |.  8D45 F8         |lea      eax, [ebp-8]  

005118F4  |.  50              |push     eax  

005118F5  |.  B9 01000000     |mov      ecx, 1  

005118FA  |.  8BD3            |mov      edx, ebx  

005118FC  |.  8B45 FC         |mov      eax, [ebp-4]  

005118FF  |.  E8 1C38EFFF     |call     00405120  

00511904  |.  8B45 F8         |mov      eax, [ebp-8]  

00511907  |.  E8 8C80EFFF     |call     00409998  

0051190C  |.  8BF8            |mov      edi, eax  

0051190E  |.  8BC7            |mov      eax, edi  

00511910  |.  F7EB            |imul     ebx  

00511912  |.  71 05          |jno      short 00511919  

00511914  |.  E8 4724EFFF     |call     00403D60  

00511919  |>  83C0 03         |add      eax, 3                         ;  EAX=EAX+3  

0051191C  |.  71 05          |jno      short 00511923  

0051191E  |.  E8 3D24EFFF     |call     00403D60  

00511923  |>  99              |cdq                                 ;  双字扩展  

00511924  |.  33C2            |xor      eax, edx  

00511926  |.  2BC2            |sub      eax, edx                     ;  EAX=EAX-EDX  

00511928  |.  71 05          |jno      short 0051192F  

0051192A  |.  E8 3124EFFF     |call     00403D60  

0051192F  |>  8BF8            |mov      edi, eax                     ;  EAX 送入 EDI  

00511931  |.  833D DCBF5100 0>|cmp      dword ptr [51BFDC], 0        ;  比较是否取完  

00511938  |.  75 0E          |jnz      short 00511948  

0051193A  |.  813D D8BF5100 F>|cmp      dword ptr [51BFD8], 5F5E0FF  

00511944  |.  76 2F          |jbe      short 00511975  

00511946  |.  EB 02          |jmp      short 0051194A  

00511948  |>  7E 2B          |jle      short 00511975  

0051194A  |>  6A 00          |push     0  

0051194C  |.  68 80969800     |push     989680  

00511951  |.  8B05 D8BF5100   |mov      eax, [51BFD8]  

00511957  |.  8B15 DCBF5100   |mov      edx, [51BFDC]  

0051195D  |.  E8 5E44EFFF     |call     00405DC0  

00511962  |.  71 05          |jno      short 00511969  

00511964  |.  E8 F723EFFF     |call     00403D60  

00511969  |>  8905 D8BF5100   |mov      [51BFD8], eax  

0051196F  |.  8915 DCBF5100   |mov      [51BFDC], edx  

00511975  |>  8BC7            |mov      eax, edi  

00511977  |.  99              |cdq  

00511978  |.  52              |push     edx  

00511979  |.  50              |push     eax  

0051197A  |.  8B05 D8BF5100   |mov      eax, [51BFD8]  

00511980  |.  8B15 DCBF5100   |mov      edx, [51BFDC]  

00511986  |.  E8 CD42EFFF     |call     00405C58  

0051198B  |.  71 05          |jno      short 00511992  

0051198D  |.  E8 CE23EFFF     |call     00403D60  

00511992  |>  8905 D8BF5100   |mov      [51BFD8], eax                ;  EAX 送入[51BFD8]  

00511998  |.  8915 DCBF5100   |mov      [51BFDC], edx  

0051199E  |.  E9 A8000000     |jmp      00511A4B  

005119A3  |>  833D DCBF5100 0>|cmp      dword ptr [51BFDC], 0  

005119AA  |.  75 0E          |jnz      short 005119BA  

005119AC  |.  813D D8BF5100 7>|cmp      dword ptr [51BFD8], 98967F  

005119B6  |.  76 2F          |jbe      short 005119E7  

005119B8  |.  EB 02          |jmp      short 005119BC  

005119BA  |>  7E 2B          |jle      short 005119E7  

005119BC  |>  6A 00          |push     0  

005119BE  |.  68 40420F00     |push     0F4240  

005119C3  |.  8B05 D8BF5100   |mov      eax, [51BFD8]  

005119C9  |.  8B15 DCBF5100   |mov      edx, [51BFDC]  

005119CF  |.  E8 EC43EFFF     |call     00405DC0  

005119D4  |.  71 05          |jno      short 005119DB  

005119D6  |.  E8 8523EFFF     |call     00403D60  

005119DB  |>  8905 D8BF5100   |mov      [51BFD8], eax  

005119E1  |.  8915 DCBF5100   |mov      [51BFDC], edx  

005119E7  |>  8B45 FC         |mov      eax, [ebp-4]                 ;  硬盘序列号送入 EAX  

005119EA  |.  4B              |dec      ebx                         ;  减 1  

005119EB  |.  85C0            |test     eax, eax  

005119ED  |.  74 05          |je       short 005119F4  

005119EF  |.  3B58 FC         |cmp      ebx, [eax-4]  

005119F2  |.  72 05          |jb       short 005119F9  

005119F4      E8 5F23EFFF     call     00403D58  

005119F9  |>  43              |inc      ebx                         ;  +1  

005119FA  |.  8A4418 FF       |mov      al, [eax+ebx-1]               ;  开始取硬盘序列号的每一位 ASCII 值  

005119FE  |.  25 FF000000     |and      eax, 0FF                     ;  取的值与 0FF 做 and 运算  

00511A03  |.  33D2            |xor      edx, edx                     ;  EDX 清 0  

00511A05  |.  52              |push     edx                         ;  EDX 入栈  

00511A06  |.  50              |push     eax                         ;  EAX 入栈  

00511A07  |.  8B05 D8BF5100   |mov      eax, [51BFD8]                ;  [51BFD8]送入 EAX  

00511A0D  |.  8B15 DCBF5100   |mov      edx, [51BFDC]                ;  [51BFDC]送入 EDX  

00511A13  |.  E8 4042EFFF     |call     00405C58  

00511A18  |.  71 05          |jno      short 00511A1F  

00511A1A  |.  E8 4123EFFF     |call     00403D60  

00511A1F  |>  83C0 07         |add      eax, 7                       ;  EAX=EAX+7  

00511A22  |.  83D2 00         |adc      edx, 0                       ;  EDX=EDX+0=0  

00511A25  |.  71 05          |jno      short 00511A2C  

00511A27  |.  E8 3423EFFF     |call     00403D60  

00511A2C  |>  85D2            |test     edx, edx  
```



```
00511A2E  |.  7D 0F             |jge     short 00511A3F  

00511A30  |.  33C9             |xor     ecx, ecx  

00511A32  |.  F7D8             |neg     eax  

00511A34  |.  87CA             |xchg    edx, ecx  

00511A36  |.  1BD1             |sbb     edx, ecx  

00511A38  |.  71 05           |jno     short 00511A3F  

00511A3A  |.  E8 2123EFFF     |call    00403D60  

00511A3F  |>  8905 D8BF5100   |mov     [51BFD8], eax          ;  EAX 送入[51BFD8]  

00511A45  |.  8915 DCBF5100   |mov     [51BFDC], edx          ;  EDX 送入[51BFDC]  

00511A4B  |>  43              |inc     ebx                    ;  ebx(计数器)加 1  

00511A4C  |.  4E              |dec     esi                    ;  减 1  

00511A4D  |.^ 0F85 78FEFFFF   \jnz    005118CB                 ;  循环取  

00511A53  |>  8B05 D8BF5100   mov     eax, [51BFD8]  

00511A59  |.  8B15 DCBF5100   mov     edx, [51BFDC]  

00511A5F  |.  85D2            test    edx, edx  

00511A61  |.  7D 0F           jge     short 00511A72  

00511A63  |.  33C9            xor     ecx, ecx  

00511A65  |.  F7D8            neg     eax  

00511A67  |.  87CA            xchg    edx, ecx  

00511A69  |.  1BD1            sbb     edx, ecx  

00511A6B  |.  71 05          jno     short 00511A72  

00511A6D  |.  E8 EE22EFFF    call    00403D60  

00511A72  |>  8905 D8BF5100   mov     [51BFD8], eax           ;  EAX(一系列运算完毕的 16 进制)送入[51BFD8]  

00511A78  |.  8915 DCBF5100   mov     [51BFDC], edx  

00511A7E  |.  833D DCBF5100 0>cmp     dword ptr [51BFDC], 0  

00511A85  |.  75 0E          jnz     short 00511A95  

00511A87  |.  813D D8BF5100 F>cmp     dword ptr [51BFD8], 3B9AC9FF   ;  真码 16 进制与"3B9AC9FF"作对比  

00511A91  |.  76 2F          jbe     short 00511AC2            ;  小于就跳  

00511A93  |.  EB 02          jmp     short 00511A97  

00511A95  |>  7E 2B          jle     short 00511AC2  

00511A97  |>  6A 00          push    0  

00511A99  |.  68 00CA9A3B    push    3B9ACA00  

00511A9E  |.  8B05 D8BF5100  mov     eax, [51BFD8]  

00511AA4  |.  8B15 DCBF5100  mov     edx, [51BFDC]  

00511AAA  |.  E8 1143EFFF    call    00405DC0  

00511AAF  |.  71 05          jno     short 00511AB6  

00511AB1  |.  E8 AA22EFFF    call    00403D60  

00511AB6  |>  8905 D8BF5100   mov     [51BFD8], eax  

00511ABC  |.  8915 DCBF5100   mov     [51BFDC], edx  

00511AC2  |>  33C0            xor     eax, eax  

00511AC4  |.  5A              pop     edx  

00511AC5  |.  59              pop     ecx  

00511AC6  |.  59              pop     ecx  

00511AC7  |.  64:8910         mov     fs:[eax], edx  

00511ACA  |.  68 E41A5100     push    00511AE4  

00511ACF  |>  8D45 F4         lea     eax, [ebp-C]  

00511AD2  |.  BA 03000000     mov     edx, 3  

00511AD7  |.  E8 5831EFFF     call    00404C34  

00511ADC  \\. C3             retn                   ;  返回  

--------------------------------------------------------------------------------
```

### 算法总结

1. 硬盘序列号必须为 `14` 位，可记为`A`
2. 取`A`的每一位`ASCII`值与`0FF`做`and`运算再加上`7`，最后全部累加为`16`进制，可记为`B`
3. 然后把`B`转换为`10`进制，即是注册码

### 后记

1. 其实在破解的时候：触发事件一断下来就给了我们真码的`16`进制.
   （当时没太在意，可一直找不出关键`call`，然后就一层层`call`跟进，最后找到正确的算法核心）
2. 以下出现一系列的`ascii``"999999999999"`及`ascii``"0000000000"`都没有与算法搭上边,当时被这些迷惑住了,呵呵~~~
3. 感谢你看到此处~~很简单的一个算法~ 其他也没啥好说的了.有不当之处,请各位大侠多多指正~~~

---

### 版权声明

本文纯属技术交流，转载请注明作者信息并保持文章的完整，谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
