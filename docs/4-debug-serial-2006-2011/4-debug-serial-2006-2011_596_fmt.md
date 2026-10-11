
# 某虚拟机 7.1.3 注册码算法分析+注册机

> 原文：[`www.pediy.com/kssd/pediy12/130419.html`](https://www.pediy.com/kssd/pediy12/130419.html)

【文章标题】: 某虚拟机 7.1.3 注册码算法分析+注册机  
【文章作者】: Daniel  
【作者邮箱】: `vvcracker@gmail.com`  
【操作系统】: `Windows 7`  
【生产日期】: 2010  
【软件名称】: 某虚拟机 7.1.3  
【软件介绍】: 一个虚拟机系统平台  
【加壳方式】: 无  
【保护方式】: 注册码  
【编写语言】: `Microsoft Visual C++ 10.0`  
【使用工具】: `OD` + `PEID` + `IDA Pro`  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请各位大牛小牛原谅!

--------------------------------------------------------------------------------

## 讨论说明

本文只讨论核心验证算法，至于如何跟踪到核心算法，不作说明。

## 软件加密详细过程

验证过程：由于此验证过程比较多，我会一层一层地说明

`112341E0` 为主注册算法入口，还要处理返回的日期限制数组

```
112341E0  /$  55            push    ebp
112341E1  |.  8BEC          mov     ebp, esp
112341E3  |.  83EC 4C       sub     esp, 4C
112341E6  |.  53            push    ebx
112341E7  |.  33C0          xor     eax, eax
112341E9  |.  57            push    edi
112341EA  |.  50            push    eax                       ; /Arg4 => 00000000
112341EB  |.  8945 F0       mov     dword ptr ss:[ebp-10], eax      ; |
112341EE  |.  8945 E4       mov     dword ptr ss:[ebp-1C], eax      ; |
112341F1  |.  8B86 E8010000 mov     eax, dword ptr ds:[esi+1E8]     ; |
112341F7  |.  68 38B13D11   push    113DB138                      ; |Arg3 = 113DB138 ASCII "Hash"
112341FC  |.  50            push    eax                       ; |Arg2
112341FD  |.  8D5E 08       lea     ebx, dword ptr ds:[esi+8]     ; |
11234200  |.  53            push    ebx                       ; |Arg1
11234201  |.  E8 2A290000   call    11236B30                      ; \vmwareba.11236B30
11234206  |.  8BF8          mov     edi, eax                      ;  获取这次循环中，sha 散列固定的字符数组
11234208  |.  83C4 10       add     esp, 10
1123420B  |.  85FF          test    edi, edi
1123420D  |.  0F84 C1000000 je      112342D4
11234213  |.  68 84003C11   push    113C0084                      ; /Arg2 = 113C0084 ASCII "Option"
11234218  |.  56            push    esi                       ; |Arg1
11234219  |.  E8 12FAFFFF   call    11233C30                      ; \vmwareba.11233C30
1123421E  |.  83C4 08       add     esp, 8
11234221  |.  A8 02         test    al, 2
11234223  |.  8945 EC       mov     dword ptr ss:[ebp-14], eax
11234226  |.  74 1F         je      short 11234247
11234228  |.  8B45 08       mov     eax, dword ptr ss:[ebp+8]
1123422B  |.  8D50 01       lea     edx, dword ptr ds:[eax+1]
1123422E  |.  8BFF          mov     edi, edi
11234230  |>  8A08          /mov     cl, byte ptr ds:[eax]
11234232  |.  83C0 01       |add     eax, 1
11234235  |.  84C9          |test    cl, cl
11234237  |.^ 75 F7         \jnz     short 11234230
11234239  |.  2BC2          sub     eax, edx
1123423B  |.  83F8 1D       cmp     eax, 1D
1123423E  |.  0F85 90000000 jnz     112342D4
11234244  |.  8B45 EC       mov     eax, dword ptr ss:[ebp-14]
11234247  |>  A8 08         test    al, 8
11234249  |.  74 1A         je      short 11234265
1123424B  |.  8B8E E8010000 mov     ecx, dword ptr ds:[esi+1E8]
11234251  |.  6A 01         push    1                           ; /Arg4 = 00000001
11234253  |.  68 F0BB3D11   push    113DBBF0                      ; |Arg3 = 113DBBF0 ASCII "Hash2"
11234258  |.  51            push    ecx                       ; |Arg2
11234259  |.  53            push    ebx                       ; |Arg1
1123425A  |.  E8 D1280000   call    11236B30                      ; \vmwareba.11236B30
1123425F  |.  83C4 10       add     esp, 10
11234262  |.  8945 F0       mov     dword ptr ss:[ebp-10], eax
11234265  |>  8D55 F8       lea     edx, dword ptr ss:[ebp-8]
11234268  |.  52            push    edx                       ; /Arg7
11234269  |.  8D9E EC010000 lea     ebx, dword ptr ds:[esi+1EC]     ; |
1123426F  |.  53            push    ebx                       ; |Arg6
11234270  |.  8D45 E8       lea     eax, dword ptr ss:[ebp-18]      ; |
11234273  |.  50            push    eax                       ; |edx 为日期数组，1123a320 处理后写入数据
11234274  |.  8B45 08       mov     eax, dword ptr ss:[ebp+8]       ; |
11234277  |.  8D4D F4       lea     ecx, dword ptr ss:[ebp-C]       ; |
1123427A  |.  51            push    ecx                       ; |Arg4
1123427B  |.  8D55 FC       lea     edx, dword ptr ss:[ebp-4]       ; |
1123427E  |.  52            push    edx                       ; |Arg3
1123427F  |.  57            push    edi                       ; |Arg2
11234280  |.  50            push    eax                       ; |Arg1
11234281  |.  E8 9A600000   call    1123A320                      ; \1123a320 注册算法入口
11234286  |.  83C4 1C       add     esp, 1C
11234289  |.  85C0          test    eax, eax
1123428B  |.  74 13         je      short 112342A0
1123428D  |>  6A 00         push    0                           ; /Arg1 = 00000000
1123428F  |.  E8 ECA70900   call    #510                        ; \#510
11234294  |.  83C4 04       add     esp, 4
11234297  |.  5F            pop     edi
11234298  |.  83C8 FF       or      eax, FFFFFFFF
1123429B  |.  5B            pop     ebx
1123429C  |.  8BE5          mov     esp, ebp
1123429E  |.  5D            pop     ebp
1123429F  |.  C3            retn
112342A0  |>  837D F8 00    cmp     dword ptr ss:[ebp-8], 0
112342A4  |.  75 36         jnz     short 112342DC
112342A6  |.  8B45 F0       mov     eax, dword ptr ss:[ebp-10]
112342A9  |.  85C0          test    eax, eax
112342AB  |.  74 27         je      short 112342D4
112342AD  |.  8D4D F8       lea     ecx, dword ptr ss:[ebp-8]
112342B0  |.  51            push    ecx                       ; /Arg7
112342B1  |.  53            push    ebx                       ; |Arg6
112342B2  |.  8D55 E8       lea     edx, dword ptr ss:[ebp-18]      ; |
112342B5  |.  52            push    edx                       ; |
112342B6  |.  8D4D F4       lea     ecx, dword ptr ss:[ebp-C]       ; |
112342B9  |.  51            push    ecx                       ; |Arg4
112342BA  |.  8D55 FC       lea     edx, dword ptr ss:[ebp-4]       ; |
112342BD  |.  52            push    edx                       ; |Arg3
112342BE  |.  50            push    eax                       ; |Arg2
112342BF  |.  8B45 08       mov     eax, dword ptr ss:[ebp+8]       ; |
112342C2  |.  50            push    eax                       ; |Arg1
112342C3  |.  E8 58600000   call    1123A320                      ; \1123a320 一系列注册算法
112342C8  |.  83C4 1C       add     esp, 1C
112342CB  |.  85C0          test    eax, eax
112342CD  |.^ 75 BE         jnz     short 1123428D
112342CF  |.  3945 F8       cmp     dword ptr ss:[ebp-8], eax
112342D2  |.  75 08         jnz     short 112342DC
112342D4  |>  5F            pop     edi
112342D5  |.  33C0          xor     eax, eax
112342D7  |.  5B            pop     ebx
112342D8  |.  8BE5          mov     esp, ebp
112342DA  |.  5D            pop     ebp
112342DB  |.  C3            retn
112342DC  |>  8B45 F4       mov     eax, dword ptr ss:[ebp-C]
112342DF  |.  84C0          test    al, al
112342E1  |.^ 74 F1         je      short 112342D4
112342E3  |.  8B4D 10       mov     ecx, dword ptr ss:[ebp+10]
112342E6  |.  8B55 0C       mov     edx, dword ptr ss:[ebp+C]
112342E9  |.  51            push    ecx                       ; /Arg7
112342EA  |.  52            push    edx                       ; |Arg6
112342EB  |.  8B55 EC       mov     edx, dword ptr ss:[ebp-14]      ; |
112342EE  |.  8D4D E4       lea     ecx, dword ptr ss:[ebp-1C]      ; |
```



1.  **代码段**

```
112342F1  |.  51            push    ecx                             ; |Arg5  
112342F2  |.  52            push    edx                             ; |Arg4  
112342F3  |.  50            push    eax                             ; |Arg3  
112342F4  |.  57            push    edi                             ; |Arg2  
112342F5  |.  56            push    esi                             ; |Arg1  
112342F6  |.  8D45 B4       lea     eax, dword ptr ss:[ebp-4C]       ; |  
112342F9  |.  8D4D CC       lea     ecx, dword ptr ss:[ebp-34]       ; |  
112342FC  |.  E8 7FFAFFFF   call    11233D80                         ; \\判断日期数组，处理日期限制数组  
11234301  |.  83C4 1C       add     esp, 1C  
11234304  |.  83F8 01       cmp     eax, 1  
11234307  |.  0F85 AC000000 jnz     112343B9  
1123430D  |.  8B55 14       mov     edx, dword ptr ss:[ebp+14]  
11234310  |.  85D2          test    edx, edx  
11234312  |.  0F84 9C000000 je      112343B4  
11234318  |.  66:8B4D FC    mov     cx, word ptr ss:[ebp-4]  
1123431C  |.  66:898A A2000>mov     word ptr ds:[edx+A2], cx        ;  以下写入日期信息  
11234323  |.  8B4D E8       mov     ecx, dword ptr ss:[ebp-18]  
11234326  |.  898A A4000000 mov     dword ptr ds:[edx+A4], ecx  
1123432C  |.  8B4D CC       mov     ecx, dword ptr ss:[ebp-34]  
1123432F  |.  894A 24       mov     dword ptr ds:[edx+24], ecx  
11234332  |.  8B4D D0       mov     ecx, dword ptr ss:[ebp-30]  
11234335  |.  894A 28       mov     dword ptr ds:[edx+28], ecx  
11234338  |.  8B4D D4       mov     ecx, dword ptr ss:[ebp-2C]  
1123433B  |.  894A 2C       mov     dword ptr ds:[edx+2C], ecx  
1123433E  |.  8B4D D8       mov     ecx, dword ptr ss:[ebp-28]  
11234341  |.  8A45 F4       mov     al, byte ptr ss:[ebp-C]  
11234344  |.  3C 07         cmp     al, 7  
11234346  |.  894A 30       mov     dword ptr ds:[edx+30], ecx  
11234349  |.  8B4D DC       mov     ecx, dword ptr ss:[ebp-24]  
1123434C  |.  894A 34       mov     dword ptr ds:[edx+34], ecx  
1123434F  |.  8B4D E0       mov     ecx, dword ptr ss:[ebp-20]  
11234352  |.  894A 38       mov     dword ptr ds:[edx+38], ecx  
11234355  |.  8B4D B4       mov     ecx, dword ptr ss:[ebp-4C]  
11234358  |.  894A 40       mov     dword ptr ds:[edx+40], ecx  
1123435B  |.  8B4D B8       mov     ecx, dword ptr ss:[ebp-48]  
1123435E  |.  894A 44       mov     dword ptr ds:[edx+44], ecx  
11234361  |.  8B4D BC       mov     ecx, dword ptr ss:[ebp-44]  
11234364  |.  894A 48       mov     dword ptr ds:[edx+48], ecx  
11234367  |.  8B4D C0       mov     ecx, dword ptr ss:[ebp-40]  
1123436A  |.  894A 4C       mov     dword ptr ds:[edx+4C], ecx  
1123436D  |.  8B4D C4       mov     ecx, dword ptr ss:[ebp-3C]  
11234370  |.  894A 50       mov     dword ptr ds:[edx+50], ecx  
11234373  |.  8B4D C8       mov     ecx, dword ptr ss:[ebp-38]  
11234376  |.  8882 A0000000 mov     byte ptr ds:[edx+A0], al  
1123437C  |.  894A 54       mov     dword ptr ds:[edx+54], ecx  
1123437F  |.  76 2D         jbe     short 112343AE  
11234381  |.  33C0          xor     eax, eax  
11234383  |.  8D8E 28020000 lea     ecx, dword ptr ds:[esi+228]  
11234389  |.  8DA424 000000>lea    esp, dword ptr ss:[esp]  
11234390  |>  8079 FC 00    /cmp    byte ptr ds:[ecx-4], 0  
11234394  |.  74 0D         |je     short 112343A3  
11234396  |.  8339 00       |cmp    dword ptr ds:[ecx], 0  
11234399  |.  74 08         |je     short 112343A3  
1123439B  |.  C68410 AE0000>|mov    byte ptr ds:[eax+edx+AE], 1  
112343A3  |>  83C0 01       |add    eax, 1  
112343A6  |.  83C1 08       |add    ecx, 8  
112343A9  |.  83F8 05       |cmp    eax, 5  
112343AC  |.^ 7C E2         \\jl     short 11234390  
112343AE  |>  8B45 E4       mov     eax, dword ptr ss:[ebp-1C]  
112343B1  |.  8942 7C       mov     dword ptr ds:[edx+7C], eax  
112343B4  |>  B8 01000000   mov     eax, 1  
112343B9  |>  5F            pop     edi  
112343BA  |.  5B            pop     ebx  
112343BB  |.  8BE5          mov     esp, ebp  
112343BD  |.  5D            pop     ebp  
112343BE  \\.  C3            retn  
```

2.  **说明**

按 `F7` 进入 `1123a320` 算法处理函数，如下。在这个函数中，有两个关键处理：`112396A0` 为注册码转换为 16 位数组 `A`，`1123A190` 对数组 `A` 进行第二次转换处理，以及对数组位作检测，作判断。

```
1123A320  /$  55            push    ebp  
1123A321  |.  8BEC          mov     ebp, esp  
1123A323  |.  56            push    esi  
1123A324  |.  57            push    edi  
1123A325  |.  8B7D 08       mov     edi, dword ptr ss:[ebp+8]  
1123A328  |.  8BC7          mov     eax, edi  
1123A32A  |.  E8 41F2FFFF   call    11239570  
1123A32F  |.  8BF0          mov     esi, eax  
1123A331  |.  85F6          test    esi, esi  
1123A333  |.  74 2F         je      short 1123A364  
1123A335  |.  8B4D 20       mov     ecx, dword ptr ss:[ebp+20]  
1123A338  |.  8B55 0C       mov     edx, dword ptr ss:[ebp+C]  
1123A33B  |.  8B45 14       mov     eax, dword ptr ss:[ebp+14]  
1123A33E  |.  51            push    ecx                             ; /Arg3  
1123A33F  |.  8B4D 10       mov     ecx, dword ptr ss:[ebp+10]      ; |  
1123A342  |.  52            push    edx                             ; |Arg2  
1123A343  |.  C600 00       mov     byte ptr ds:[eax], 0            ; |  
1123A346  |.  8B45 18       mov     eax, dword ptr ss:[ebp+18]      ; |  
1123A349  |.  56            push    esi                             ; |Arg1  
1123A34A  |.  E8 01F4FFFF   call    11239750                        ; \\vmwareba.11239750  
1123A34F  |.  83C4 0C       add     esp, 0C  
1123A352  |.  56            push    esi                             ; /block  
1123A353  |.  8BF8          mov     edi, eax                        ; |  
1123A355  |.  FF15 BC253911 call    dword ptr ds:[<&MSVCR80.free>]  ; \\free  
1123A35B  |.  83C4 04       add     esp, 4  
1123A35E  |.  8BC7          mov     eax, edi  
1123A360  |.  5F            pop     edi  
1123A361  |.  5E            pop     esi  
1123A362  |.  5D            pop     ebp  
1123A363  |.  C3            retn  
1123A364  |>  6A 00         push    0                               ; /Arg1 = 00000000  
1123A366  |.  E8 15470900   call    #510                            ; \\#510  
1123A36B  |.  83C4 04       add     esp, 4  
1123A36E  |.  8BC7          mov     eax, edi  
1123A370  |.  E8 8BF2FFFF   call    11239600  
1123A375  |.  8BF0          mov     esi, eax                        ; |  
1123A377  |.  85F6          test    esi, esi                        ; |  
1123A379  |.  74 3E         je      short 1123A3B9                  ; |  
1123A37B  |.  8B7D 1C       mov     edi, dword ptr ss:[ebp+1C]      ; |  
1123A37E  |.  6A 60         push    60                              ; |/n = 60 (96.)  
1123A380  |.  6A 00         push    0                               ; ||c = 00  
1123A382  |.  57            push    edi                             ; ||s  
1123A383  |.  E8 F8A00F00   call    <jmp.&MSVCR80.memset>           ; |\\memset  
1123A388  |.  8B45 20       mov     eax, dword ptr ss:[ebp+20]      ; |  
1123A38B  |.  8B4D 18       mov     ecx, dword ptr ss:[ebp+18]      ; |  
1123A38E  |.  8B55 14       mov     edx, dword ptr ss:[ebp+14]      ; |  
1123A391  |.  50            push    eax                             ; |Arg6  
1123A392  |.  8B45 10       mov     eax, dword ptr ss:[ebp+10]      ; |  
1123A395  |.  51            push    ecx                             ; |Arg5  
1123A396  |.  8B4D 0C       mov     ecx, dword ptr ss:[ebp+C]       ; |  
1123A399  |.  52            push    edx                             ; |Arg4  
1123A39A  |.  50            push    eax                             ; |Arg3  
1123A39B  |.  51            push    ecx                             ; |Arg2  
1123A39C  |.  56            push    esi                             ; |Arg1  
```



1123A39D  |.  8BC7           mov     eax, edi                             ; |  

1123A39F  |.  E8 6CF6FFFF   call    11239A10                             ; \vmwareba.11239A10  

1123A3A4  |.  83C4 24       add     esp, 24  

1123A3A7  |.  56            push    esi                                ; /block  

1123A3A8  |.  8BF8          mov     edi, eax                             ; |  

1123A3AA  |.  FF15 BC253911 call    dword ptr ds:[<&MSVCR80.free>]    ; \free  

1123A3B0  |.  83C4 04       add     esp, 4  

1123A3B3  |.  8BC7          mov     eax, edi  

1123A3B5  |.  5F            pop     edi  

1123A3B6  |.  5E            pop     esi  

1123A3B7  |.  5D            pop     ebp  

1123A3B8  |.  C3            retn  

1123A3B9  |>  6A 00         push    0                                    ; /Arg1 = 00000000  

1123A3BB  |.  E8 C0460900   call    #510                                 ; \#510  

1123A3C0  |.  83C4 04       add     esp, 4  

1123A3C3  |.  8BC7          mov     eax, edi                             ;  eax 为我们输入的注册码  

1123A3C5  |.  E8 D6F2FFFF   call    112396A0  

1123A3CA  |.  8BF0          mov     esi, eax                             ;  eax 由注册值转换的 16 位数组值  

1123A3CC  |.  85F6         test    esi, esi                             ; |  

1123A3CE  |.  74 3C         je      short 1123A40C                       ; |  

1123A3D0  |.  8B7D 1C       mov     edi, dword ptr ss:[ebp+1C]          ; |  

1123A3D3  |.  6A 60        push    60                                   ; |/n = 60 (96.)  

1123A3D5  |.  6A 00        push    0                                    ; ||c = 00  

1123A3D7  |.  57            push    edi                                ; ||s  

1123A3D8  |.  E8 A3A0F00   call    <jmp.&MSVCR80.memset>              ; |\memset  

1123A3DD  |.  8B55 20       mov     edx, dword ptr ss:[ebp+20]          ; |  

1123A3E0  |.  8B45 18       mov     eax, dword ptr ss:[ebp+18]          ; |  

1123A3E3  |.  8B4D 10       mov     ecx, dword ptr ss:[ebp+10]          ; |  

1123A3E6  |.  52            push    edx                                ; |Arg6  

1123A3E7  |.  8B55 0C       mov     edx, dword ptr ss:[ebp+C]           ; |  

1123A3EA  |.  57            push    edi                                ; |Arg5  

1123A3EB  |.  50            push    eax                                ; |Arg4  

1123A3EC  |.  8B45 14       mov     eax, dword ptr ss:[ebp+14]          ; |  

1123A3EF  |.  51            push    ecx                                ; |Arg3  

1123A3F0  |.  52            push    edx                                ; |edx 为每次循环提供的 hash 值  

1123A3F1  |.  56            push    esi                                ; |Arg1  

1123A3F2  |.  E8 99FDFFFF   call    1123A190                             ; \vmwareba.1123A190  

1123A3F7  |.  83C4 24       add     esp, 24                              ;   

1123A3FA  |.  56            push    esi                                ; /block  

1123A3FB  |.  8BF8          mov     edi, eax                             ; |  

1123A3FD  |.  FF15 BC253911 call    dword ptr ds:[<&MSVCR80.free>]    ; \free  

1123A403  |.  83C4 04       add     esp, 4  

1123A406  |.  8BC7          mov     eax, edi  

1123A408  |.  5F            pop     edi  

1123A409  |.  5E            pop     esi  

1123A40A  |.  5D            pop     ebp  

1123A40B  |.  C3            retn  

1123A40C  |>  56            push    esi                                ; /block  

1123A40D  |.  BF 01000000   mov     edi, 1                               ; |  

1123A412  |.  FF15 BC253911 call    dword ptr ds:[<&MSVCR80.free>]    ; \free  

1123A418  |.  83C4 04       add     esp, 4  

1123A41B  |.  8BC7          mov     eax, edi  

1123A41D  |.  5F            pop     edi  

1123A41E  |.  5E            pop     esi  

1123A41F  |.  5D            pop     ebp  

1123A420  \\.  C3            retn  

先进入 `112396A0`，看到真正的注册码处理转换在 `11239400` 中

1123973C  |>  \68 68C73D11  push    113DC768                             ; /Arg2 = 113DC768  

11239741  |.  50            push    eax                                ; |Arg1  

11239742  |.  B8 19000000   mov     eax, 19                              ; |注册码长度  

11239747  |.  E8 B4FCFFFF   call    11239400                             ; \  

1123974C  |.  83C4 08       add     esp, 8  

1123974F  \\.  C3            retn  

进入 `11239400`，注册码转换为 16 位数组，算法描述如下：

11239400  /$  55            push    ebp  

11239401  |.  8BEC          mov     ebp, esp  

11239403  |.  51            push    ecx                                ;   

11239404  |.  56            push    esi  

11239405  |.  8BF0          mov     esi, eax  

11239407  |.  8D04B6        lea     eax, dword ptr ds:[esi+esi*4]  

1123940A  |.  57            push    edi  

1123940B  |.  50            push    eax                                ; /Arg1  

1123940C  |.  E8 3F36EDFF   call    #201                                 ; \#201  

11239411  |.  8BF8          mov     edi, eax  

11239413  |.  83C4 04       add     esp, 4  

11239416  |.  85FF          test    edi, edi  

11239418  |.  75 06         jnz     short 11239420  

1123941A  |.  5F            pop     edi  

1123941B  |.  5E            pop     esi  

1123941C  |.  8BE5          mov     esp, ebp  

1123941E  |.  5D            pop     ebp  

1123941F  |.  C3            retn  

11239420  |>  53            push    ebx  

11239421  |.  8D5E FF       lea     ebx, dword ptr ds:[esi-1]          ;  循环 19 次，从最后一位位开始  

11239424  |.  85DB          test    ebx, ebx  

11239426  |.  895D FC       mov     dword ptr ss:[ebp-4], ebx  

11239429  |.  0F8C 0C010000 jl      1123953B  

1123942F  |.  8D749B 02     lea     esi, dword ptr ds:[ebx+ebx*4+2]    ;  赋值 esi，位置计算从 7a 开始  

11239433  |>  8B4D 0C       /mov    ecx, dword ptr ss:[ebp+C]          ecx 为索引数组  

11239436  |.  8B1499        |mov    edx, dword ptr ds:[ecx+ebx*4]    ;  edx 获取索引值  

11239439  |.  8B45 08       |mov    eax, dword ptr ss:[ebp+8]  

1123943C  |.  0FBE0C02      |movsx  ecx, byte ptr ds:[edx+eax]      ;  获取索引的注册码值  

11239440  |.  51            |push    ecx                            ; /c  

11239441  |.  68 30C63D11   |push    113DC630                       ; |s = "0123456789ACDEFGHJKLMNPQRTUVWXYZ"  

11239446  |.  FF15 38263911 |call    dword ptr ds:[<&MSVCR80.strchr>>; \strchr  

1123944C  |.  83C4 08       |add     esp, 8                         ;  eax 计算注册码在 113DC630 数组的位置  

1123944F  |.  85C0          |test    eax, eax  

11239451  |.  0F84 ED000000 |je      11239544  

11239457  |.  BA 30C63D11   |mov     edx, 113DC630                   ;  ASCII "0123456789ACDEFGHJKLMNPQRTUVWXYZ"  

1123945C  |.  2AC2          |sub     al, dl                         ;  al 为索引值  

1123945E  |.  8D4E FE       |lea     ecx, dword ptr ds:[esi-2]      ;  处理 esi-2 的位置值  

11239461  |.  8BD1          |mov     edx, ecx  

11239463  |.  83E1 1F       |and     ecx, 1F                        ;  ecx 为 32 位中相对的位置  

11239466  |.  BB 01000000   |mov     ebx, 1                         ;  ebx 赋值为 1  

1123946B  |.  C1FA 05       |sar     edx, 5                         ;  edx 为转换 16 位数组中的索引  

1123946E  |.  D3E3          |shl     ebx, cl                        ;  ebx 为对 32 位中的 cl 的位置置 1  

11239470  |.  A8 01         |test    al, 1                          ;  测试 al 是否为 1  

11239472  |.  8D5497 08     |lea     edx, dword ptr ds:[edi+edx*4+8] ;  计算转换数组的位置  

11239476  |.  74 04         |je      short 1123947C  

11239478  |.  091A          |or      dword ptr ds:[edx], ebx        ;  对转换数组中的位置赋值 1  

1123947A  |.  EB 04         |jmp     short 11239480  

1123947C  |>  F7D3          |not     ebx  

1123947E  |.  211A          |and     dword ptr ds:[edx], ebx        ;  对转换数组中的位置赋值 0  

11239480  |>  8D4E FF       |lea     ecx, dword ptr ds:[esi-1]      ;  处理 esi-1 的位置值  

11239483  |.  8BD1          |mov     edx, ecx  

11239485  |.  83E1 1F       |and     ecx, 1F                        ;  ecx 为 32 位中相对的位置  

11239488  |.  BB 01000000   |mov     ebx, 1                         ;  ebx 赋值为 1  



```
1123948D  |.  D0E8           |shr      al, 1                        ;  al 为索引值，右移一位  
1123948F  |.  C1FA 05        |sar      edx, 5                       ;  edx 为转换 16 位数组中的索引  
11239492  |.  D3E3           |shl      ebx, cl                      ;  ebx 为对 32 位中的 cl 的位置置 1  
11239494  |.  A8 01          |test     al, 1                        ;  测试 al 是否为 1  
11239496  |.  8D5497 08      |lea      edx, dword ptr ds:[edi+edx*4+8] ;  计算转换数组的位置  
1123949A  |.  74 04          |je       short 112394A0  
1123949C  |.  091A           |or       dword ptr ds:[edx], ebx      ;  对转换数组中的位置赋值 1  
1123949E  |.  EB 04          |jmp      short 112394A4  
112394A0  |>  F7D3           |not      ebx  
112394A2  |.  211A          |and      dword ptr ds:[edx], ebx      ;  对转换数组中的位置赋值 0  
112394A4  |>  D0E8          |shr      al, 1                        ;  以下也有 3 步类似处理，同上  
112394A6  |.  BB 01000000    |mov      ebx, 1  
112394AB  |.  84C3          |test     bl, al  
112394AD  |.  8BCE          |mov      ecx, esi  
112394AF  |.  74 12         |je       short 112394C3  
112394B1  |.  C1F9 05       |sar      ecx, 5  
112394B4  |.  8D548F 08      |lea      edx, dword ptr ds:[edi+ecx*4+8]  
112394B8  |.  8BCE          |mov      ecx, esi  
112394BA  |.  83E1 1F       |and      ecx, 1F  
112394BD  |.  D3E3          |shl      ebx, cl  
112394BF  |.  091A          |or       dword ptr ds:[edx], ebx  
112394C1  |.  EB 12         |jmp      short 112394D5  
112394C3  |>  83E1 1F       |and      ecx, 1F  
112394C6  |.  D3E3          |shl      ebx, cl  
112394C8  |.  8BD6          |mov      edx, esi  
112394CA  |.  C1FA 05       |sar      edx, 5  
112394CD  |.  8D5497 08      |lea      edx, dword ptr ds:[edi+edx*4+8]  
112394D1  |.  F7D3          |not      ebx  
112394D3  |.  211A          |and      dword ptr ds:[edx], ebx  
112394D5  |>  8D4E 01       |lea      ecx, dword ptr ds:[esi+1]  
112394D8  |.  8BD1          |mov      edx, ecx  
112394DA  |.  83E1 1F       |and      ecx, 1F  
112394DD  |.  BB 01000000    |mov      ebx, 1  
112394E2  |.  D0E8          |shr      al, 1  
112394E4  |.  C1FA 05       |sar      edx, 5  
112394E7  |.  D3E3          |shl      ebx, cl  
112394E9  |.  A8 01         |test     al, 1  
112394EB  |.  8D5497 08      |lea      edx, dword ptr ds:[edi+edx*4+8]  
112394EF  |.  74 04         |je       short 112394F5  
112394F1  |.  091A          |or       dword ptr ds:[edx], ebx  
112394F3  |.  EB 04         |jmp      short 112394F9  
112394F5  |>  F7D3          |not      ebx  
112394F7  |.  211A          |and      dword ptr ds:[edx], ebx  
112394F9  |>  8D4E 02       |lea      ecx, dword ptr ds:[esi+2]  
112394FC  |.  A8 02         |test     al, 2  
112394FE  |.  BA 01000000    |mov      edx, 1  
11239503  |.  8BC1          |mov      eax, ecx  
11239505  |.  74 10         |je       short 11239517  
11239507  |.  83E1 1F       |and      ecx, 1F  
1123950A  |.  C1F8 05       |sar      eax, 5  
1123950D  |.  D3E2          |shl      edx, cl  
1123950F  |.  8D4487 08      |lea      eax, dword ptr ds:[edi+eax*4+8]  
11239513  |.  0910          |or       dword ptr ds:[eax], edx  
11239515  |.  EB 10         |jmp      short 11239527  
11239517  |>  83E1 1F       |and      ecx, 1F  
1123951A  |.  D3E2          |shl      edx, cl  
1123951C  |.  C1F8 05       |sar      eax, 5  
1123951F  |.  8D4487 08      |lea      eax, dword ptr ds:[edi+eax*4+8]  
11239523  |.  F7D2          |not      edx  
11239525  |.  2110          |and      dword ptr ds:[eax], edx  
11239527  |>  8B5D FC       |mov      ebx, dword ptr ss:[ebp-4]  
1123952A  |.  83EB 01       |sub      ebx, 1                         ;  循环值减一  
1123952D  |.  83EE 05       |sub      esi, 5                         ;  esi 减 5 位  
11239530  |.  85DB          |test     ebx, ebx  
11239532  |.  895D FC       |mov      dword ptr ss:[ebp-4], ebx  
11239535  |.^ 0F8D F8FEFFFF  |jge     11239433  
1123953B  |>  5B           pop      ebx  
1123953C  |.  8BC7          mov      eax, edi  
1123953E  |.  5F           pop      edi  
1123953F  |.  5E           pop      esi  
11239540  |.  8BE5          mov      esp, ebp  
11239542  |.  5D           pop      ebp  
11239543  |.  C3           retn  
11239544  |>  8B45 0C       mov      eax, dword ptr ss:[ebp+C]  
11239547  |.  8B0C98        mov      ecx, dword ptr ds:[eax+ebx*4]  
1123954A  |.  83C1 01       add      ecx, 1  
1123954D  |.  51           push     ecx  
1123954E  |.  68 38C83D11    push     113DC838                         ;   
11239553  |.  E8 08550900    call     112CEA60  
11239558  |.  57           push     edi                         ; /block  
11239559  |.  FF15 BC253911  call     dword ptr ds:[<&MSVCR80.free>]   ; \free  
1123955F  |.  83C4 0C       add      esp, 0C  
11239562  |.  5B           pop      ebx  
11239563  |.  5F           pop      edi  
11239564  |.  33C0          xor      eax, eax  
11239566  |.  5E           pop      esi  
11239567  |.  8BE5          mov      esp, ebp  
11239569  |.  5D           pop      ebp  
1123956A  \.  C3           retn  
```

观看索引数组，循环获取索引值，索引从 `0x18` 开始到 `0` 结束

```
113DC768  19 00 00 00 15 00 00 00 01 00 00 00 10 00 00 00  ............  
113DC778  13 00 00 00 00 00 00 00 14 00 00 00 08 00 00 00  .............  
113DC788  16 00 00 00 18 00 00 00 1B 00 00 00 02 00 00 00  ............  
113DC798  09 00 00 00 0A 00 00 00 06 00 00 00 0E 00 00 00  ..............  
113DC7A8  1A 00 00 00 1C 00 00 00 03 00 00 00 04 00 00 00  ............  
113DC7B8  0C 00 00 00 0D 00 00 00 07 00 00 00 0F 00 00 00  ..............  
113DC7C8  12 00 00 00 02 00 00 00 01 00 00 00 00 00 00 00  .............
```

## IDA 分析 `11239400` 算法

转换为 C，算法说明文章结尾看软件加密思路 1，算法如下

```c
v6 = v5 - 1;  
v23 = v5 - 1;  
if ( v5 - 1 < 0 )  
{  
LABEL_22:  
  result = v4;  
}  
else  
{  
  v7 = 5 * v6 + 2;  
  while ( 1 )                    
  {  
    v8 = strchr("0123456789ACDEFGHJKLMNPQRTUVWXYZ", *(_BYTE *)(*(_DWORD *)(a3 + 4 * v6) + a2));  
    if ( !v8 )  
      break;  
    v9 = (_BYTE)v8 - (unsigned int)"0123456789ACDEFGHJKLMNPQRTUVWXYZ";  
    v11 = 1 << ((v7 - 2) & 0x1F);  
    v10 = (int)(v4 + 4 * ((v7 - 2) >> 5) + 8);  
    if ( v9 & 1 )  
      *(_DWORD *)v10 |= v11;  
    else  
      *(_DWORD *)v10 &= ~v11;  
    v12 = v9 >> 1;  
    v14 = 1 << ((v7 - 1) & 0x1F);  
    v13 = (int)(v4 + 4 * ((v7 - 1) >> 5) + 8);  
    if ( v12 & 1 )  
      *(_DWORD *)v13 |= v14;  
    else  
      *(_DWORD *)v13 &= ~v14;  
    v15 = v12 >> 1;  
    if ( v15 & 1 )  
      *((_DWORD *)v4 + (v7 >> 5) + 2) |= 1 << (v7 & 0x1F);  
    else  
      *((_DWORD *)v4 + (v7 >> 5) + 2) &= ~(1 << (v7 & 0x1F));  
    v16 = v15 >> 1;  
    v18 = 1 << ((v7 + 1) & 0x1F);  
    v17 = (int)(v4 + 4 * ((v7 + 1) >> 5) + 8);  
    if ( v16 & 1 )  
      *(_DWORD *)v17 |= v18;  
    else  
      *(_DWORD *)v17 &= ~v18;  
    v20 = v7 + 2;  
    v21 = (v16 & 2) == 0;  
    v19 = v7 + 2;  
    if ( v21 )  
      *((_DWORD *)v4 + (v19 >> 5) + 2) &= ~(1 << (v20 & 0x1F));  
    else  
      *((_DWORD *)v4 + (v19 >> 5) + 2) |= 1 << (v20 & 0x1F);  
    v6 = v23 - 1;  
    v7 -= 5;  
    v22 = v23-- - 1 < 0;  
    if ( v22 )  
      goto LABEL_22;  
  }  
  sub_112CEA60(  
    "@&!*@*@(msg.serial.full.alphabet)The character number %u of the serial number is invalid.\n",  
    *(_DWORD *)(a3 + 4 * v6) + 1);  
  free(v4);  
  result = 0;  
}  
```

至此，第一次转换注册码完成。

## 进入 `1123A190`，看第二个注册验证过程

又是一个漫长的验证过程，注意，注释的地方都属于处理过程

```
1123A190  /$  55           push     ebp  
1123A191  |.  8BEC          mov      ebp, esp  
1123A193  |.  83EC 08       sub      esp, 8  
1123A196  |.  53           push     ebx  
```



```
1123A197   |.   56          push    esi  

1123A198   |.   6A 7D       push    7D                     ; /Arg1 = 0000007D  

1123A19A   |.   8BD8       mov     ebx, eax               ; |  

1123A19C   |.   E8 AF28EDFF   call    #201               ; \\#201  

1123A1A1   |.   8BF0       mov     esi, eax  

1123A1A3   |.   83C4 04    add     esp, 4  

1123A1A6   |.   85F6       test    esi, esi  

1123A1A8   |.   75 09      jnz     short 1123A1B3  

1123A1AA   |.   8D46 01    lea     eax, dword ptr ds:\[esi+1\]  

1123A1AD   |.   5E         pop     esi  

1123A1AE   |.   5B         pop     ebx  

1123A1AF   |.   8BE5       mov     esp, ebp  

1123A1B1   |.   5D         pop     ebp  

1123A1B2   |.   C3         retn  

1123A1B3   |>   8B45 08    mov     eax, dword ptr ss:\[ebp+8\]  

1123A1B6   |.   57         push    edi  

1123A1B7   |.   6A 01      push    1  

1123A1B9   |.   50         push    eax                  ;  eax 为第一次转换的 16 位数组  

1123A1BA   |.   8BFE       mov     edi, esi               ;  edi 为经过 11239100 处理的第二次转换 16 位数组  

1123A1BC   |.   E8 3FEFFFFF   call    11239100  

1123A1C1   |.   8B45 14    mov     eax, dword ptr ss:\[ebp+14\]       ;  

1123A1C4   |.   8B55 10    mov     edx, dword ptr ss:\[ebp+10\]       ;  

1123A1C7   |.   53         push    ebx                  ;  

1123A1C8   |.   E8 03FBFFFF   call    11239CD0             ; 根据第二次转换 16 位数组的值，获取第 14 位的值，判断是否在合理值范围内，并赋值到 ebx 的地址  

1123A1CD   |.   83C4 0C    add     esp, 0C  

1123A1D0   |.   803B 06    cmp     byte ptr ds:\[ebx\], 6         ; 判断 ebx 是否在合理值范围内  

1123A1D3   |.   72 0A      jb      short 1123A1DF  

1123A1D5   |.   8B45 18    mov     eax, dword ptr ss:\[ebp+18\]       ; eax 为时间数组  

1123A1D8   |.   8BD6       mov     edx, esi               ; edx 为第二次转换 16 位数组地址  

1123A1DA   |.   E8 11FDFFFF   call    11239EF0             ; 11239EF0 根据 edx 数组赋值到 eax  

1123A1DF   |>   803B 08    cmp     byte ptr ds:\[ebx\], 8         ; 判断 ebx 是否在合理值范围内  

1123A1E2   |.   72 0C      jb      short 1123A1F0  

1123A1E4   |.   8B55 18    mov     edx, dword ptr ss:\[ebp+18\]  

1123A1E7   |.   56         push    esi                  ;  

1123A1E8   |.   E8 53FFFFFF   call    1123A140             ;  

1123A1ED   |.   83C4 04    add     esp, 4  

1123A1F0   |>   33DB       xor     ebx, ebx  

1123A1F2   |.   33FF       xor     edi, edi  

1123A1F4   |.   B8 25000000   mov     eax, 25  

1123A1F9   |.   8DA424 000000>lea     esp, dword ptr ss:\[esp\]          ;  以下处理是分离出 16 位数组前 5 位  

1123A200   |>   0FA4DF 01    /shld    edi, ebx, 1             ;  前 4 位，后 1 位分别赋给 ebx，edi  

1123A204   |.   8D50 02      |lea     edx, dword ptr ds:\[eax+2\]       ;  此过程有点迷惑人的成分，过程很复杂  

1123A207   |.   897D FC      |mov     dword ptr ss:\[ebp-4\], edi       ;  开头跟踪一番后，才发觉自己被骗了  

1123A20A   |.   8BCA         |mov     ecx, edx               ;   

1123A20C   |.   83E1 1F      |and     ecx, 1F  

1123A20F   |.   BF 01000000   |mov     edi, 1  

1123A214   |.   D3E7         |shl     edi, cl  

1123A216   |.   8B4D FC      |mov     ecx, dword ptr ss:\[ebp-4\]  

1123A219   |.   C1FA 05      |sar     edx, 5  

1123A21C   |.   03DB         |add     ebx, ebx  

1123A21E   |.   857C96 08    |test    dword ptr ds:\[esi+edx*4+8\], edi  

1123A222   |.   74 06        |je      short 1123A22A  

1123A224   |.   83C3 01      |add     ebx, 1  

1123A227   |.   83D1 00      |adc     ecx, 0  

1123A22A   |>   0FA4D9 01    |shld    ecx, ebx, 1  

1123A22E   |.   894D FC      |mov     dword ptr ss:\[ebp-4\], ecx  

1123A231   |.   8D50 01      |lea     edx, dword ptr ds:\[eax+1\]  

1123A234   |.   8BCA         |mov     ecx, edx  

1123A236   |.   83E1 1F      |and     ecx, 1F  

1123A239   |.   BF 01000000   |mov     edi, 1  

1123A23E   |.   D3E7         |shl     edi, cl  

1123A240   |.   C1FA 05      |sar     edx, 5  

1123A243   |.   03DB         |add     ebx, ebx  

1123A245   |.   857C96 08    |test    dword ptr ds:\[esi+edx*4+8\], edi  

1123A249   |.   8B55 FC      |mov     edx, dword ptr ss:\[ebp-4\]  

1123A24C   |.   74 06        |je      short 1123A254  

1123A24E   |.   83C3 01      |add     ebx, 1  

1123A251   |.   83D2 00      |adc     edx, 0  

1123A254   |>   8BC8         |mov     ecx, eax  

1123A256   |.   83E1 1F      |and     ecx, 1F  

1123A259   |.   BF 01000000   |mov     edi, 1  

1123A25E   |.   D3E7         |shl     edi, cl  

1123A260   |.   8BC8         |mov     ecx, eax  

1123A262   |.   0FA4DA 01    |shld    edx, ebx, 1  

1123A266   |.   C1F9 05      |sar     ecx, 5  

1123A269   |.   03DB         |add     ebx, ebx  

1123A26B   |.   857C8E 08    |test    dword ptr ds:\[esi+ecx*4+8\], edi  

1123A26F   |.   74 06        |je      short 1123A277  

1123A271   |.   83C3 01      |add     ebx, 1  

1123A274   |.   83D2 00      |adc     edx, 0  

1123A277   |>   0FA4DA 01    |shld    edx, ebx, 1  

1123A27B   |.   8955 FC      |mov     dword ptr ss:\[ebp-4\], edx  

1123A27E   |.   8D50 FF      |lea     edx, dword ptr ds:\[eax-1\]  

1123A281   |.   8BCA         |mov     ecx, edx  

1123A283   |.   83E1 1F      |and     ecx, 1F  

1123A286   |.   BF 01000000   |mov     edi, 1  

1123A28B   |.   D3E7         |shl     edi, cl  

1123A28D   |.   8B4D FC      |mov     ecx, dword ptr ss:\[ebp-4\]  

1123A290   |.   C1FA 05      |sar     edx, 5  

1123A293   |.   03DB         |add     ebx, ebx  

1123A295   |.   857C96 08    |test    dword ptr ds:\[esi+edx*4+8\], edi  

1123A299   |.   74 06        |je      short 1123A2A1  

1123A29B   |.   83C3 01      |add     ebx, 1  

1123A29E   |.   83D1 00      |adc     ecx, 0  

1123A2A1   |>   0FA4D9 01    |shld    ecx, ebx, 1  

1123A2A5   |.   894D FC      |mov     dword ptr ss:\[ebp-4\], ecx  

1123A2A8   |.   8D50 FE      |lea     edx, dword ptr ds:\[eax-2\]  

1123A2AB   |.   8BCA         |mov     ecx, edx  

1123A2AD   |.   83E1 1F      |and     ecx, 1F  

1123A2B0   |.   BF 01000000   |mov     edi, 1  

1123A2B5   |.   D3E7         |shl     edi, cl  

1123A2B7   |.   C1FA 05      |sar     edx, 5  

1123A2BA   |.   03DB         |add     ebx, ebx  

1123A2BC   |.   857C96 08    |test    dword ptr ds:\[esi+edx*4+8\], edi  

1123A2C0   |.   8B7D FC      |mov     edi, dword ptr ss:\[ebp-4\]  

1123A2C3   |.   74 06        |je      short 1123A2CB  

1123A2C5   |.   83C3 01      |add     ebx, 1  

1123A2C8   |.   83D7 00      |adc     edi, 0  

1123A2CB   |>   83E8 05      |sub     eax, 5  

1123A2CE   |.   8D50 02      |lea     edx, dword ptr ds:\[eax+2\]  

1123A2D1   |.   85D2         |test    edx, edx  

1123A2D3   |.^ 0F8D 27FFFFFF  \jge     1123A200  

1123A2D9   |.   8B4D 0C      mov     ecx, dword ptr ss:\[ebp+C\]  

1123A2DC   |.   8D45 F8      lea     eax, dword ptr ss:\[ebp-8\]  

1123A2DF   |.   50         push    eax                  ; 处理结果保存到 eax 中  

1123A2E0   |.   51         push    ecx                  ; ecx 上层提供的固定字符串数组  

1123A2E1   |.   56         push    esi                  ; esi 第二次转换 16 位数组  

1123A2E2   |.   E8 49E8FFFF   call    11238B30             ; 对第二次还原 16 位后 11 位，进行处理  

1123A2E7   |.   83C4 0C      add     esp, 0C                ;   

1123A2EA   |.   3B5D F8      cmp     ebx, dword ptr ss:\[ebp-8\]        ;  对比 ebx，edi 的值是否相同  

1123A2ED   |.   75 0C      jnz     short 1123A2FB  

1123A2EF   |.   3B7D FC      cmp     edi, dword ptr ss:\[ebp-4\]  

1123A2F2   |.   75 07      jnz     short 1123A2FB  

1123A2F4   |.   B8 01000000   mov     eax, 1  

1123A2F9   |.   EB 02      jmp     short 1123A2FD  

1123A2FB   |>   33C0       xor     eax, eax  

1123A2FD   |>   8B55 1C      mov     edx, dword ptr ss:\[ebp+1C\]       ;  1zhi  

1123A300   |.   56         push    esi                  ; /block  

1123A301   |.   8902       mov     dword ptr ds:\[edx\], eax        ; |  

1123A303   |.   FF15 BC253911  call    dword ptr ds:\[<&MSVCR80.free>\]   ; \\free
```



```
1123A309  |.  83C4 04       add     esp, 4  
1123A30C  |.  5F            pop     edi  
1123A30D  |.  5E            pop     esi  
1123A30E  |.  33C0          xor     eax, eax  
1123A310  |.  5B            pop     ebx  
1123A311  |.  8BE5          mov     esp, ebp  
1123A313  |.  5D            pop     ebp  
1123A314  \.  C3            retn  
```

先看 `11239100` 处理过程，算法描述如下：

```
11239100  /$  55            push    ebp  
11239101  |.  8BEC          mov     ebp, esp  
11239103  |.  83EC 10       sub     esp, 10  
11239106  |.  53            push    ebx  
11239107  |.  56            push    esi  
11239108  |.  33F6          xor     esi, esi  
1123910A  |.  33C0          xor     eax, eax                      ;  以下为要保存计算的 3 个开始的位数值  
1123910C      C745 FC 50000>mov     dword ptr ss:\[ebp-4\], 50        ;  保存 80  
11239113      C745 F8 28000>mov     dword ptr ss:\[ebp-8\], 28        ;  保存 40  
1123911A  |.  8975 F4       mov     dword ptr ss:\[ebp-C\], esi       ;  保存 0  
1123911D  |.  33DB          xor     ebx, ebx                      ;  ebx 作为内循环求余处理的索引值  
1123911F  |.  8945 F0       mov     dword ptr ss:\[ebp-10\], eax      ;  eax 是从 0 开始的索引值  
11239122  |.  EB 0F         jmp     short 11239133                ;  以下为两个循环作为处理  
11239124  |>  8B75 F4       /mov    esi, dword ptr ss:\[ebp-C\]      ;  内循环的索引值主要用来求余  
11239127  |.  33DB          |xor    ebx, ebx                      ;  外循环的索引值从 0 到 105 位之间进行处理  
11239129  |.  8945 F0       |mov    dword ptr ss:\[ebp-10\], eax     ;  ebx 内循环索引置 0  
1123912C  |.  EB 05         |jmp    short 11239133  

1123912E  |   8BFF          |mov     edi, edi  
11239130  |>  8B75 F4       |mov     esi, dword ptr ss:\[ebp-C\]  

11239133  |>  837D 0C 00    cmp     dword ptr ss:\[ebp+C\], 0  
11239137  |.  0F84 EE000000 |je      1123922B  

1123913D  |.  8BC8          |mov     ecx, eax  
1123913F  |.  83E1 1F       |and     ecx, 1F                       ;  这里主要根据索引获取数组的值，并判断是否为 1  
11239142  |.  BE 01000000   |mov     esi, 1                        ;  同时队内循环索引求余  
11239147  |.  D3E6          |shl     esi, cl  
11239149  |.  8B4D 08       |mov     ecx, dword ptr ss:\[ebp+8\]  
1123914C  |.  C1F8 05       |sar     eax, 5  
1123914F  |.  237481 08     |and     esi, dword ptr ds:\[ecx+eax\*4+8\]  
11239153  |.  B8 ABAAAAAA   |mov     eax, AAAAAAB  
11239158  |.  F7DE          |neg     esi  
1123915A  |.  1BF6          |sbb     esi, esi  
1123915C  |.  F7E3          |mul     ebx  
1123915E  |.  D1EA          |shr     edx, 1  
11239160  |.  8D1452        |lea     edx, dword ptr ds:\[edx+edx\*2\]  
11239163  |.  8BC3          |mov     eax, ebx  
11239165  |.  F7DE          |neg     esi  
11239167  |.  2BC2          |sub     eax, edx                      ;  eax 为内循环索引值求余的结果  
11239169  |.  0F84 8A000000 |je      112391F9                      ;  以下分别对求余结果 0,1,2 进行处理  
1123916F  |.  83FB 14       |cmp     ebx, 14  
11239172  |.  0F84 81000000 |je      112391F9  
11239178  |.  F6C3 01       |test    bl, 1  
1123917B  |.  75 4A         |jnz     short 112391C7  
1123917D  |.  83FB 10       |cmp     ebx, 10  
11239180  |.  74 45         |je      short 112391C7  
11239182  |.  85F6          |test    esi, esi                      ;  处理余数为 1 的情况  
11239184  |.  8B4D FC       |mov     ecx, dword ptr ss:\[ebp-4\]     ;  获取要处理的位数  
11239187  |.  74 1E         |je      short 112391A7  
11239189  |.  8BD1          |mov     edx, ecx  
1123918B  |.  C1FA 05       |sar     edx, 5                        ;  edx 为目标 16 位数组中的索引  
1123918E  |.  8D4497 08     |lea     eax, dword ptr ds:\[edi+edx\*4+8\] ;  计算目标数组的位置  
11239192  |.  83E1 1F       |and     ecx, 1F                       ;  ecx 为 32 位中相对的位置  
11239195  |.  BA 01000000   |mov     edx, 1  
1123919A  |.  D3E2          |shl     edx, cl                       ;  edx 为对 32 位中的 cl 的位置置 1  
1123919C  |.  0910          |or      dword ptr ds:\[eax\], edx      ;  目标数组的位置置 1  
1123919E  |.  8345 FC 01    |add     dword ptr ss:\[ebp-4\], 1      ;  对位数加 1  
112391A2  |.  E9 4B010000   |jmp     112392F2  
112391A7  |>  8BC1          |mov     eax, ecx  
112391A9  |.  83E1 1F       |and     ecx, 1F                       ;  ecx 为 32 位中相对的位置  
112391AC  |.  BA 01000000   |mov     edx, 1  
112391B1  |.  D3E2          |shl     edx, cl                       ;  edx 为对 32 位中的 cl 的位置置 1  
112391B3  |.  C1F8 05       |sar     eax, 5                        ;  edx 为目标 16 位数组中的索引  
112391B6  |.  8D4487 08     |lea     eax, dword ptr ds:\[edi+eax\*4+8\] ;  计算目标数组的位置  
112391BA  |.  F7D2          |not     edx  
112391BC  |.  2110          |and     dword ptr ds:\[eax\], edx      ;  目标数组的位置置 0  
112391BE  |.  8345 FC 01    |add     dword ptr ss:\[ebp-4\], 1      ;  对位数加 1  
112391C2  |.  E9 2B010000   |jmp     112392F2  
112391C7  |>  8B4D F8       |mov     ecx, dword ptr ss:\[ebp-8\]  
112391CA  |.  8BC1          |mov     eax, ecx                      ;  以下求余为 0,2 的处理跟求余为 1 相似  
112391CC  |.  83E1 1F       |and     ecx, 1F  
112391CF  |.  BA 01000000   |mov     edx, 1  
112391D4  |.  C1F8 05       |sar     eax, 5  
112391D7  |.  D3E2          |shl     edx, cl  
112391D9  |.  85F6          |test    esi, esi  
112391DB  |.  8D4487 08     |lea     eax, dword ptr ds:\[edi+eax\*4+8\]  
112391DF  |.  74 0B         |je      short 112391EC  
112391E1  |.  0910          |or      dword ptr ds:\[eax\], edx  
112391E3  |.  8345 F8 01    |add     dword ptr ss:\[ebp-8\], 1  
112391E7  |.  E9 06010000   |jmp     112392F2  
112391EC  |>  F7D2          |not     edx  
112391EE  |.  2110          |and     dword ptr ds:\[eax\], edx  
112391F0  |.  8345 F8 01    |add     dword ptr ss:\[ebp-8\], 1  
112391F4  |.  E9 F9000000   |jmp     112392F2  
112391F9  |>  8B4D F4       |mov     ecx, dword ptr ss:\[ebp-C\]  
112391FC  |.  8BC1          |mov     eax, ecx  
112391FE  |.  83E1 1F       |and     ecx, 1F  
11239201  |.  BA 01000000   |mov     edx, 1  
11239206  |.  C1F8 05       |sar     eax, 5  
11239209  |.  D3E2          |shl     edx, cl  
1123920B  |.  85F6          |test    esi, esi  
1123920D  |.  8D4487 08     |lea     eax, dword ptr ds:\[edi+eax\*4+8\]  
11239211  |.  74 0B         |je      short 1123921E  
11239213  |.  0910          |or      dword ptr ds:\[eax\], edx  
11239215  |.  8345 F4 01    |add     dword ptr ss:\[ebp-C\], 1  
11239219  |.  E9 D4000000   |jmp     112392F2  
1123921E  |>  F7D2          |not     edx  
11239220  |.  2110          |and     dword ptr ds:\[eax\], edx  
11239222  |.  8345 F4 01    |add     dword ptr ss:\[ebp-C\], 1  
11239226  |.  E9 C7000000   |jmp     112392F2  
1123922B  |>  B8 ABAAAAAA   |mov     eax, AAAAAAB  
11239230  |.  F7E3          |mul     ebx  
11239232  |.  D1EA          |shr     edx, 1  
11239234  |.  8D0452        |lea     eax, dword ptr ds:\[edx+edx\*2\]  
11239237  |.  8BCB          |mov     ecx, ebx  
11239239  |.  2BC8          |sub     ecx, eax  
1123923B  |.  74 63         |je      short 112392A0  
1123923D  |.  83FB 14       |cmp     ebx, 14  
11239240  |.  74 5E         |je      short 112392A0  
11239242  |.  F6C3 01       |test    bl, 1  
11239245  |.  75 2F         |jnz     short 11239276  
11239247  |.  83FB 10       |cmp     ebx, 10  
1123924A  |.  74 2A         |je      short 11239276  
1123924C  |.  8B4D FC       |mov     ecx, dword ptr ss:\[ebp-4\]  
1123924F  |.  8B75 08       |mov     esi, dword ptr ss:\[ebp+8\]  
11239252  |.  83E1 1F       |and     ecx, 1F  
11239255  |.  B8 01000000   |mov     eax, 1  
1123925A  |.  D3E0          |shl     eax, cl  
1123925C  |.  8B4D FC       |mov     ecx, dword ptr ss:\[ebp-4\]  
1123925F  |.  8BD1          |mov     edx, ecx  
11239261  |.  C1FA 05       |sar     edx, 5  
11239264  |.  234496 08     |and     eax, dword ptr ds:\[esi+edx\*4+8\]  
11239268  |.  F7D8          |neg     eax  
1123926A  |.  1BC0          |sbb     eax, eax  
```



1123926C  |.  F7D8          |neg     eax  

1123926E  |.  83C1 01       |add     ecx, 1  

11239271  |.  894D FC       |mov     dword ptr ss:\[ebp-4\], ecx  

11239274  |.  EB 4E         |jmp     short 112392C4  

11239276  |>  8B4D F8       |mov     ecx, dword ptr ss:\[ebp-8\]  

11239279  |.  8B75 08       |mov     esi, dword ptr ss:\[ebp+8\]  

1123927C  |.  83E1 1F       |and     ecx, 1F  

1123927F  |.  B8 01000000   |mov     eax, 1  

11239284  |.  D3E0          |shl     eax, cl  

11239286  |.  8B4D F8       |mov     ecx, dword ptr ss:\[ebp-8\]  

11239289  |.  8BD1          |mov     edx, ecx  

1123928B  |.  C1FA 05       |sar     edx, 5  

1123928E  |.  234496 08     |and     eax, dword ptr ds:\[esi+edx\*4+8\]  

11239292  |.  F7D8          |neg     eax  

11239294  |.  1BC0          |sbb     eax, eax  

11239296  |.  F7D8          |neg     eax  

11239298  |.  83C1 01       |add     ecx, 1  

1123929B  |.  894D F8       |mov     dword ptr ss:\[ebp-8\], ecx  

1123929E  |.  EB 24         |jmp     short 112392C4  

112392A0  |>  8B55 08       |mov     edx, dword ptr ss:\[ebp+8\]  

112392A3  |.  8BCE          |mov     ecx, esi  

112392A5  |.  83E1 1F       |and     ecx, 1F  

112392A8  |.  B8 01000000   |mov     eax, 1  

112392AD  |.  D3E0          |shl     eax, cl  

112392AF  |.  8BCE          |mov     ecx, esi  

112392B1  |.  C1F9 05       |sar     ecx, 5  

112392B4  |.  23448A 08     |and     eax, dword ptr ds:\[edx+ecx\*4+8\]  

112392B8  |.  F7D8          |neg     eax  

112392BA  |.  1BC0          |sbb     eax, eax  

112392BC  |.  F7D8          |neg     eax  

112392BE  |.  83C6 01       |add     esi, 1  

112392C1  |.  8975 F4       |mov     dword ptr ss:\[ebp-C\], esi  

112392C4  |>  8B4D F0       |mov     ecx, dword ptr ss:\[ebp-10\]  

112392C7  |.  85C0          |test    eax, eax  

112392C9  |.  BA 01000000   |mov     edx, 1  

112392CE  |.  8BC1          |mov     eax, ecx  

112392D0  |.  74 10         |je      short 112392E2  

112392D2  |.  83E1 1F       |and     ecx, 1F  

112392D5  |.  C1F8 05       |sar     eax, 5  

112392D8  |.  D3E2          |shl     edx, cl  

112392DA  |.  8D4487 08     |lea     eax, dword ptr ds:\[edi+eax\*4+8\]  

112392DE  |.  0910          |or      dword ptr ds:\[eax\], edx  

112392E0  |.  EB 10         |jmp     short 112392F2  

112392E2  |>  83E1 1F       |and     ecx, 1F  

112392E5  |.  D3E2          |shl     edx, cl  

112392E7  |.  C1F8 05       |sar     eax, 5  

112392EA  |.  8D4487 08     |lea     eax, dword ptr ds:\[edi+eax\*4+8\]  

112392EE  |.  F7D2          |not     edx  

112392F0  |.  2110          |and     dword ptr ds:\[eax\], edx  

112392F2  |>  8345 F0 01    |add     dword ptr ss:\[ebp-10\], 1  

112392F6  |.  8B45 F0       |mov     eax, dword ptr ss:\[ebp-10\]      ;  增加外循环索引  

112392F9  |.  83C3 01       |add     ebx, 1                            ;  基于上一个 key，从 1 到 69 位，求余，分别基于 0,1,2  

112392FC  |.  83FB 15       |cmp     ebx, 15  

112392FF  |.^ 0F8C 2BFEFFFF |jl      11239130  

11239305  |.  83F8 69       |cmp     eax, 69  

11239308  |.^ 0F8C 16FEFFFF \|jl      11239124  

1123930E  |.  BE 6B000000   mov     esi, 6B                            ;  从 107 位开始到 128 位直接复制  

11239313  |>  8B5D 08       /mov     ebx, dword ptr ss:\[ebp+8\]  

11239316  |.  8D46 FE       |lea     eax, dword ptr ds:\[esi-2\]  

11239319  |.  8BC8          |mov     ecx, eax  

1123931B  |.  83E1 1F       |and     ecx, 1F  

1123931E  |.  BA 01000000   |mov     edx, 1  

11239323  |.  D3E2          |shl     edx, cl  

11239325  |.  C1F8 05       |sar     eax, 5  

11239328  |.  8D0485 080000>|lea     eax, dword ptr ds:\[eax\*4+8\]  

1123932F  |.  851418        |test    dword ptr ds:\[eax+ebx\], edx  

11239332  |.  74 05         |je      short 11239339  

11239334  |.  091438        |or      dword ptr ds:\[eax+edi\], edx  

11239337  |.  EB 05         |jmp     short 1123933E  

11239339  |>  F7D2          |not     edx  

1123933B  |.  211438        |and     dword ptr ds:\[eax+edi\], edx  

1123933E  |>  8D46 FF       |lea     eax, dword ptr ds:\[esi-1\]  

11239341  |.  8BC8          |mov     ecx, eax  

11239343  |.  83E1 1F       |and     ecx, 1F  

11239346  |.  BA 01000000   |mov     edx, 1  

1123934B  |.  D3E2          |shl     edx, cl  

1123934D  |.  C1F8 05       |sar     eax, 5  

11239350  |.  8D0485 080000>|lea     eax, dword ptr ds:\[eax\*4+8\]  

11239357  |.  851418        |test    dword ptr ds:\[eax+ebx\], edx  

1123935A  |.  74 05         |je      short 11239361  

1123935C  |.  091438        |or      dword ptr ds:\[eax+edi\], edx  

1123935F  |.  EB 05         |jmp     short 11239366  

11239361  |>  F7D2          |not     edx  

11239363  |.  211438        |and     dword ptr ds:\[eax+edi\], edx  

11239366  |>  8BCE          |mov     ecx, esi  

11239368  |.  83E1 1F       |and     ecx, 1F  

1123936B  |.  B8 01000000   |mov     eax, 1  

11239370  |.  D3E0          |shl     eax, cl  

11239372  |.  8BCE          |mov     ecx, esi  

11239374  |.  C1F9 05       |sar     ecx, 5  

11239377  |.  8D0C8D 080000>|lea     ecx, dword ptr ds:\[ecx\*4+8\]  

1123937E  |.  850419        |test    dword ptr ds:\[ecx+ebx\], eax  

11239381  |.  74 05         |je      short 11239388  

11239383  |.  090439        |or      dword ptr ds:\[ecx+edi\], eax  

11239386  |.  EB 05         |jmp     short 1123938D  

11239388  |>  F7D0          |not     eax  

1123938A  |.  210439        |and     dword ptr ds:\[ecx+edi\], eax  

1123938D  |>  8D46 01       |lea     eax, dword ptr ds:\[esi+1\]  

11239390  |.  8BC8          |mov     ecx, eax  

11239392  |.  83E1 1F       |and     ecx, 1F  

11239395  |.  BA 01000000   |mov     edx, 1  

1123939A  |.  D3E2          |shl     edx, cl  

1123939C  |.  C1F8 05       |sar     eax, 5  

1123939F  |.  8D0485 080000>|lea     eax, dword ptr ds:\[eax\*4+8\]  

112393A6  |.  851418        |test    dword ptr ds:\[eax+ebx\], edx  

112393A9  |.  74 05         |je      short 112393B0  

112393AB  |.  091438        |or      dword ptr ds:\[eax+edi\], edx  

112393AE  |.  EB 05         |jmp     short 112393B5  

112393B0  |>  F7D2          |not     edx  

112393B2  |.  211438        |and     dword ptr ds:\[eax+edi\], edx  

112393B5  |>  8D46 02       |lea     eax, dword ptr ds:\[esi+2\]  

112393B8  |.  8BC8          |mov     ecx, eax  

112393BA  |.  83E1 1F       |and     ecx, 1F  

112393BD  |.  BA 01000000   |mov     edx, 1  

112393C2  |.  D3E2          |shl     edx, cl  

112393C4  |.  C1F8 05       |sar     eax, 5  

112393C7  |.  8D0485 080000>|lea     eax, dword ptr ds:\[eax\*4+8\]  

112393CE  |.  851418        |test    dword ptr ds:\[eax+ebx\], edx  

112393D1  |.  74 05         |je      short 112393D8  

112393D3  |.  091438        |or      dword ptr ds:\[eax+edi\], edx  

112393D6  |.  EB 05         |jmp     short 112393DD  

112393D8  |>  F7D2          |not     edx  

112393DA  |.  211438        |and     dword ptr ds:\[eax+edi\], edx  

112393DD  |>  83C6 05       |add     esi, 5  

112393E0  |.  8D56 FE       |lea     edx, dword ptr ds:\[esi-2\]  

112393E3  |.  83FA 7D       |cmp     edx, 7D  

112393E6  |.^ 0F8C 27FFFFFF \|jl      11239313  

112393EC  |.  5E            pop     esi  

112393ED  |.  5B            pop     ebx  

112393EE  |.  8BE5          mov     esp, ebp  

112393F0  |.  5D            pop     ebp  

112393F1  |\.  C3            retn  

使用 `IDA` 分析 `11239100` 算法，转换为 C 语言。算法说明见文章结尾“软件加密思路 2”，算法代码如下：

```c
while ( 1 )
{
    while ( 1 )
    {
        if ( a3 )
        {
            v6 = (*(\_DWORD \*)(a2 + 4 * (v3 >> 5) + 8) & (1 << (v3 & 0x1F))) != 0;
            if ( v4 % 3 && v4 != 20 )
            {
                if ( v4 & 1 || v4 == 16 )
                {
                    v8 = 1 << (v25 & 0x1F);
                    v7 = a1 + 4 * (v25 >> 5) + 8;
                    if ( v6 )
                    {
                        *(\_DWORD \*)v7 |= v8;
                        ++v25;
                    }
                    else
                    {
```



```c
              *( _DWORD *)v7 &= ~v8;

              ++v25;

            }

          }

          else

          {

            if ( ( *(_DWORD *)(a2 + 4 * (v3 >> 5) + 8) & (1 << (v3 & 0x1F))) != 0 )

            {

              *( _DWORD *)(a1 + 4 * (v26 >> 5) + 8) |= 1 << (v26 & 0x1F);

              ++v26;

            }

            else

            {

              *( _DWORD *)(a1 + 4 * (v26 >> 5) + 8) &= ~(1 << (v26 & 0x1F));

              ++v26;

            }

          }

        }

        else

        {

          v10 = 1 << (v24 & 0x1F);

          v9 = a1 + 4 * (v24 >> 5) + 8;

          if ( v6 )

          {

            *( _DWORD *)v9 |= v10;

            ++v24;

          }

          else

          {

            *( _DWORD *)v9 &= ~v10;

            ++v24;

          }

        }

      }

      else //这个 else 逻辑处理地方没用到，不用看这部分

      {

        if ( v4 % 3 && v4 != 20 )

        {

          if ( v4 & 1 || v4 == 16 )

          {

            v11 = ( *(_DWORD *)(a2 + 4 * (v25 >> 5) + 8) & (1 << (v25 & 0x1F))) != 0;

            ++v25;

          }

          else

          {

            v11 = ( *(_DWORD *)(a2 + 4 * (v26 >> 5) + 8) & (1 << (v26 & 0x1F))) != 0;

            ++v26;

          }

        }

        else

        {

          v11 = ( *(_DWORD *)(a2 + 4 * (v5 >> 5) + 8) & (1 << (v5 & 0x1F))) != 0;

          v24 = v5 + 1;

        }

        if ( v11 )

          *( _DWORD *)(a1 + 4 * (v23 >> 5) + 8) |= 1 << (v23 & 0x1F);

        else

          *( _DWORD *)(a1 + 4 * (v23 >> 5) + 8) &= ~(1 << (v23 & 0x1F));

      }

      ++v23;

      v3 = v23;

      ++v4;

      if ( (signed int)v4 >= 21 )

        break;

      v5 = v24;

    }

    if ( v23 >= 105 )

      break;

    v5 = v24;

    v4 = 0;

  }

  v12 = 107;

  do

  {

    v14 = 1 << ((v12 - 2) & 0x1F);

    v13 = 4 * ((v12 - 2) >> 5) + 8;

    if ( v14 & *(_DWORD *)(v13 + a2) )

      *( _DWORD *)(v13 + a1) |= v14;

    else

      *( _DWORD *)(v13 + a1) &= ~v14;

    v16 = 1 << ((v12 - 1) & 0x1F);

    v15 = 4 * ((v12 - 1) >> 5) + 8;

    if ( v16 & *(_DWORD *)(v15 + a2) )

      *( _DWORD *)(v15 + a1) |= v16;

    else

      *( _DWORD *)(v15 + a1) &= ~v16;

    v17 = 1 << (v12 & 0x1F);

    v18 = 4 * (v12 >> 5) + 8;

    if ( v17 & *(_DWORD *)(v18 + a2) )

      *( _DWORD *)(v18 + a1) |= v17;

    else

      *( _DWORD *)(v18 + a1) &= ~v17;

    v20 = 1 << ((v12 + 1) & 0x1F);

    v19 = 4 * ((v12 + 1) >> 5) + 8;

    if ( v20 & *(_DWORD *)(v19 + a2) )

      *( _DWORD *)(v19 + a1) |= v20;

    else

      *( _DWORD *)(v19 + a1) &= ~v20;

    v22 = 1 << ((v12 + 2) & 0x1F);

    result = 4 * ((v12 + 2) >> 5) + 8;

    if ( v22 & *(_DWORD *)(result + a2) )

      *( _DWORD *)(result + a1) |= v22;

    else

      *( _DWORD *)(result + a1) &= ~v22;

    v12 += 5;

  }

  while ( v12 - 2 < 125 );
```

毫无疑问地说，`11239100`，`11239400` 这两个地方逻辑是很繁琐很麻烦的，我当初也花了很多时间理解这两个过程，但是各位读者想做出注册机，必须逆向理解这两个地方的算法，并加以调试分析。

![124.gif](img/afde35a5f3b3a4675abb3221cf07c6d1.jpg "124.gif")

回 `1123A190` 再进入 `11239CD0`，很多干扰代码，经过我分析，只需看 `11239DB1` 到 `11239E10` 的代码，主要将处理结果返回给参数

```asm
11239DB1  |.  8B45 08       mov     eax, dword ptr ss:[ebp+8]
11239DB4  |.  C600 00       mov     byte ptr ds:[eax], 0
11239DB7  |.  F746 14 00200>test    dword ptr ds:[esi+14], 2000      ;  以下检测第二次还原数组第 13 位
11239DBE  |.  74 03         je      short 11239DC3                   ;  是否符合条件
11239DC0  |.  C600 01       mov     byte ptr ds:[eax], 1             ;  加一
11239DC3  |>  D020          shl     byte ptr ds:[eax], 1             ;  左移一位
11239DC5  |.  F746 14 00100>test    dword ptr ds:[esi+14], 1000      ;  以下几步处理过程大致相同
11239DCC  |.  8A08          mov     cl, byte ptr ds:[eax]
11239DCE  |.  74 05         je      short 11239DD5
11239DD0  |.  80C1 01       add     cl, 1
11239DD3  |.  8808          mov     byte ptr ds:[eax], cl
11239DD5  |>  D020          shl     byte ptr ds:[eax], 1
11239DD7  |.  F746 14 00080>test    dword ptr ds:[esi+14], 800
11239DDE  |.  8A08          mov     cl, byte ptr ds:[eax]
11239DE0  |.  74 05         je      short 11239DE7
11239DE2  |.  80C1 01       add     cl, 1
11239DE5  |.  8808          mov     byte ptr ds:[eax], cl
11239DE7  |>  D020          shl     byte ptr ds:[eax], 1
11239DE9  |.  F746 14 00040>test    dword ptr ds:[esi+14], 400
11239DF0  |.  8A08          mov     cl, byte ptr ds:[eax]
11239DF2  |.  74 05         je      short 11239DF9
11239DF4  |.  80C1 01       add     cl, 1
11239DF7  |.  8808          mov     byte ptr ds:[eax], cl
11239DF9  |>  D020          shl     byte ptr ds:[eax], 1
11239DFB  |.  F746 14 00020>test    dword ptr ds:[esi+14], 200
11239E02  |.  8A08          mov     cl, byte ptr ds:[eax]
11239E04  |.  74 05         je      short 11239E0B
11239E06  |.  80C1 01       add     cl, 1
11239E09  |.  8808          mov     byte ptr ds:[eax], cl
11239E0B  |>  66:C702 0000  mov     word ptr ds:[edx], 0
11239E10  |.  B8 7B000000   mov     eax, 7B
11239E15  |.  C745 08 03000>mov     dword ptr ss:[ebp+8], 3
11239E1C  |.  8D6424 00     lea     esp, dword ptr ss:[esp]
```

回 `1123A190` 再进入 `11239EF0`，与 `11239CD0` 相似很多干扰代码，只需看 `11239FB0` 到 `1123A030` 的代码，主要对压入的数组进行计算赋值

```asm
11239FB0  |.  33F6          xor     esi, esi
11239FB2  |.  8858 08       mov     byte ptr ds:[eax+8], bl
11239FB5  |.  8970 34       mov     dword ptr ds:[eax+34], esi
11239FB8  |.  F742 0C 00000>test    dword ptr ds:[edx+C], 40000000
11239FBF  |.  74 03         je      short 11239FC4
11239FC1  |.  8958 34       mov     dword ptr ds:[eax+34], ebx
11239FC4  |>  D160 34       shl     dword ptr ds:[eax+34], 1
11239FC7  |.  F742 0C 00000>test    dword ptr ds:[edx+C], 20000000
11239FCE  |.  8B48 34       mov     ecx, dword ptr ds:[eax+34]
11239FD1  |.  74 05         je      short 11239FD8
11239FD3  |.  03CB          add     ecx, ebx
11239FD5  |.  8948 34       mov     dword ptr ds:[eax+34], ecx
11239FD8  |>  8858 30       mov     byte ptr ds:[eax+30], bl
11239FDB  |.  8970 2C       mov     dword ptr ds:[eax+2C], esi
11239FDE  |.  F642 10 02    test    byte ptr ds:[edx+10], 2
11239FE2  |.  74 03         je      short 11239FE7
11239FE4  |.  8958 2C       mov     dword ptr ds:[eax+2C], ebx
11239FE7  |>  D160 2C       shl     dword ptr ds:[eax+2C], 1         ;  16 位数组第 9 位的值直接影响到注册码的时间限制
11239FEA  |.  845A 10       test    byte ptr ds:[edx+10], bl        ;  测试第 9 位的值
11239FED  |.  8B48 2C       mov     ecx, dword ptr ds:[eax+2C]
11239FF0  |.  74 05         je      short 11239FF7
11239FF2  |.  03CB          add     ecx, ebx
11239FF4  |.  8948 2C       mov     dword ptr ds:[eax+2C], ecx
11239FF7  |>  D160 2C       shl     dword ptr ds:[eax+2C], 1
11239FFA  |.  F742 0C 00000>test    dword ptr ds:[edx+C], 80000000
1123A001  |.  8B48 2C       mov     ecx, dword ptr ds:[eax+2C]
1123A004  |.  74 05         je      short 1123A00B
1123A006  |.  03CB          add     ecx, ebx
1123A008  |.  8948 2C       mov     dword ptr ds:[eax+2C], ecx
1123A00B  |>  8858 28       mov     byte ptr ds:[eax+28], bl
1123A00E  |.  8970 04       mov     dword ptr ds:[eax+4], esi
1123A011  |.  F642 10 08    test    byte ptr ds:[edx+10], 8
1123A015  |.  74 03         je      short 1123A01A
1123A017  |.  8958 04       mov     dword ptr ds:[eax+4], ebx
1123A01A  |>  D160 04       shl     dword ptr ds:[eax+4], 1
1123A01D  |.  F642 10 04    test    byte ptr ds:[edx+10], 4
```



## 汇编代码段

```asm
1123A021  |.  8B48 04       mov     ecx, dword ptr ds:[eax+4]  

1123A024  |.  74 05         je      short 1123A02B  

1123A026  |.  03CB        add     ecx, ebx  

1123A028  |.  8948 04     mov     dword ptr ds:[eax+4], ecx        ;  赋给时间数组，程序会判断该地址  

1123A02B  |>  8970 24     mov     dword ptr ds:[eax+24], esi  

1123A02E  |.  8818        mov     byte ptr ds:[eax], bl  

1123A030  |.  BE 4E000000   mov     esi, 4E  

1123A035  |.  C745 FC 02000>mov     dword ptr ss:[ebp-4], 2  

1123A03C  |.  8D6424 00     lea     esp, dword ptr ss:[esp]  
```

### 跳转至 `1123A190` 然后进入 `11238B30`

```asm
11238B30  /$  55          push    ebp  

11238B31  |.  8BEC        mov     ebp, esp  

11238B33  |.  83EC 20     sub     esp, 20  

11238B36  |.  B0 C0       mov     al, 0C0  

11238B38  |.  8845 EE     mov     byte ptr ss:[ebp-12], al  

11238B3B  |.  8845 EF     mov     byte ptr ss:[ebp-11], al  

11238B3E  |.  8B45 10     mov     eax, dword ptr ss:[ebp+10]  

11238B41  |.  50          push    eax                              ; /Arg6  

11238B42  |.  8B45 08     mov     eax, dword ptr ss:[ebp+8]        ; |  

11238B45  |.  B1 BF       mov     cl, 0BF                          ; |  

11238B47  |.  B2 C7       mov     dl, 0C7                          ; |  

11238B49  |.  6A 20       push    20                               ; |Arg5 = 00000020  

11238B4B  |.  884D EC     mov     byte ptr ss:[ebp-14], cl         ; |  

11238B4E  |.  884D FF     mov     byte ptr ss:[ebp-1], cl          ; |  

11238B51  |.  8D4D E0     lea     ecx, dword ptr ss:[ebp-20]       ; |  

11238B54  |.  51          push    ecx                              ; |Arg4  

11238B55  |.  8855 E1     mov     byte ptr ss:[ebp-1F], dl         ; |  

11238B58  |.  8855 F8     mov     byte ptr ss:[ebp-8], dl         ; |  

11238B5B  |.  8B55 0C     mov     edx, dword ptr ss:[ebp+C]        ; |  

11238B5E  |.  52          push    edx                              ; |Arg3  

11238B5F  |.  6A 55       push    55                               ; |Arg2 = 00000055  

11238B61  |.  50          push    eax                              ; |Arg1  

11238B62  |.  C645 E0 7D  mov     byte ptr ss:[ebp-20], 7D         ; |从[ebp-20]开始是固定给 112389A0 处理的数组  

11238B66  |.  C645 E2 89  mov     byte ptr ss:[ebp-1E], 89         ; |  

11238B6A  |.  C645 E3 F9  mov     byte ptr ss:[ebp-1D], 0F9        ; |  

11238B6E  |.  C645 E4 69  mov     byte ptr ss:[ebp-1C], 69         ; |  

11238B72  |.  C645 E5 5D  mov     byte ptr ss:[ebp-1B], 5D         ; |  

11238B76  |.  C645 E6 71  mov     byte ptr ss:[ebp-1A], 71         ; |  

11238B7A  |.  C645 E7 90  mov     byte ptr ss:[ebp-19], 90         ; |  

11238B7E  |.  C645 E8 FA  mov     byte ptr ss:[ebp-18], 0FA       ; |  

11238B82  |.  C645 E9 84  mov     byte ptr ss:[ebp-17], 84         ; |  

11238B86  |.  C645 EA 22  mov     byte ptr ss:[ebp-16], 22         ; |  

11238B8A  |.  C645 EB CA  mov     byte ptr ss:[ebp-15], 0CA       ; |  

11238B8E  |.  C645 ED 1D  mov     byte ptr ss:[ebp-13], 1D        ; |  

11238B92  |.  C645 F0 88  mov     byte ptr ss:[ebp-10], 88        ; |  

11238B96  |.  C645 F1 42  mov     byte ptr ss:[ebp-F], 42         ; |  

11238B9A  |.  C645 F2 37  mov     byte ptr ss:[ebp-E], 37         ; |  

11238B9E  |.  C645 F3 D5  mov     byte ptr ss:[ebp-D], 0D5        ; |  

11238BA2  |.  C645 F4 70  mov     byte ptr ss:[ebp-C], 70         ; |  

11238BA6  |.  C645 F5 80  mov     byte ptr ss:[ebp-B], 80         ; |  

11238BAA  |.  C645 F6 2E  mov     byte ptr ss:[ebp-A], 2E         ; |  

11238BAE  |.  C645 F7 83  mov     byte ptr ss:[ebp-9], 83         ; |  

11238BB2  |.  C645 F9 92  mov     byte ptr ss:[ebp-7], 92         ; |  

11238BB6  |.  C645 FA A3  mov     byte ptr ss:[ebp-6], 0A3        ; |  

11238BBA  |.  C645 FB 59  mov     byte ptr ss:[ebp-5], 59         ; |  

11238BBE  |.  C645 FC C6  mov     byte ptr ss:[ebp-4], 0C6        ; |  

11238BC2  |.  C645 FD 95  mov     byte ptr ss:[ebp-3], 95         ; |  

11238BC6  |.  C645 FE 24  mov     byte ptr ss:[ebp-2], 24         ; |  

11238BCA  |.  E8 D1FDFFFF   call    112389A0                        ; \vmwareba.112389A0  

11238BCF  |.  83C4 18     add     esp, 18  

11238BD2  |.  8BE5        mov     esp, ebp  

11238BD4  |.  5D          pop     ebp  

11238BD5  \.  C3          retn  
```

### 固定要 SHA 的数组

长度 `0x20`，如下：

```text
0529F4E8  7D C7 89 F9 69 5D 71 90 FA 84 22 CA BF 1D C0 C0  }?]q?士览  

0529F4F8  88 42 37 D5 70 80 2E 83 C7 92 A3 59 C6 95 24 BF  7.Y$?  
```

### 进入 `112389A0`

`sha` 初始化处理，`sha update` 处理，`sha finalize` 处理不作说明，可到看雪论坛查阅，算法处理如下，

```asm
112389A0  /$  55          push    ebp  

112389A1  |.  8BEC        mov     ebp, esp  

112389A3  |.  81EC 80000000  sub     esp, 80  

112389A9  |.  56          push    esi  

112389AA  |.  8B75 0C     mov     esi, dword ptr ss:[ebp+C]  

112389AD  |.  83C6 07     add     esi, 7  

112389B0  |.  C1EE 03     shr     esi, 3  

112389B3  |.  57          push    edi  

112389B4  |.  56          push    esi                              ; /size  

112389B5  |.  8975 0C     mov     dword ptr ss:[ebp+C], esi        ; |  

112389B8  |.  FF15 8C263911  call    dword ptr ds:[<&MSVCR80.malloc>]  ; \malloc  

112389BE  |.  8BF8        mov     edi, eax  

112389C0  |.  83C4 04     add     esp, 4  

112389C3  |.  85FF        test    edi, edi  

112389C5  |.  0F84 4B010000  je      11238B16  

112389CB  |.  33C0        xor     eax, eax  

112389CD  |.  85F6        test    esi, esi  

112389CF  |.  53          push    ebx  

112389D0  |.  0F86 D3000000  jbe     11238AA9  

112389D6  |.  C745 F8 2E000>mov     dword ptr ss:[ebp-8], 2E  

112389DD  |.  8D49 00     lea     ecx, dword ptr ds:[ecx]  

112389E0  |>  8B75 F8     /mov     esi, dword ptr ss:[ebp-8]       ;  以下是循环分离出第二次还原数据最后 11 位  

112389E3  |.  C60438 00     |mov     byte ptr ds:[eax+edi], 0        ;  跟 1123A200 的代码相似，存在迷惑解密者的嫌疑  

112389E7  |.  C745 F4 02000>|mov     dword ptr ss:[ebp-C], 2  

112389EE  |.  8BFF        |mov     edi, edi  

112389F0  |>  D02438        |/shl     byte ptr ds:[eax+edi], 1  

112389F3  |.  8D56 01       ||lea     edx, dword ptr ds:[esi+1]  

112389F6  |.  8BCA          ||mov     ecx, edx  

112389F8  |.  83E1 1F       ||and     ecx, 1F  

112389FB  |.  BB 01000000   ||mov     ebx, 1  

11238A00  |.  D3E3          ||shl     ebx, cl  

11238A02  |.  C1FA 05       ||sar     edx, 5  

11238A05  |.  8BCB          ||mov     ecx, ebx  

11238A07  |.  8B5D 08       ||mov     ebx, dword ptr ss:[ebp+8]  

11238A0A  |.  854C93 08     ||test    dword ptr ds:[ebx+edx*4+8], ec>  

11238A0E  |.  74 09         ||je      short 11238A19  

11238A10  |.  8A0C38        ||mov     cl, byte ptr ds:[eax+edi]  

11238A13  |.  80C1 01       ||add     cl, 1  

11238A16  |.  880C38        ||mov     byte ptr ds:[eax+edi], cl  

11238A19  |>  D02438        ||shl     byte ptr ds:[eax+edi], 1  

11238A1C  |.  8BCE          ||mov     ecx, esi  

11238A1E  |.  83E1 1F       ||and     ecx, 1F  

11238A21  |.  BA 01000000   ||mov     edx, 1  

11238A26  |.  D3E2          ||shl     edx, cl  

11238A28  |.  8BCE          ||mov     ecx, esi  

11238A2A  |.  C1F9 05       ||sar     ecx, 5  

11238A2D  |.  85548B 08     ||test    dword ptr ds:[ebx+ecx*4+8], ed>  

11238A31  |.  74 09         ||je      short 11238A3C  

11238A33  |.  8A0C38        ||mov     cl, byte ptr ds:[eax+edi]  

11238A36  |.  80C1 01       ||add     cl, 1  

11238A39  |.  880C38        ||mov     byte ptr ds:[eax+edi], cl  

11238A3C  |>  D02438        ||shl     byte ptr ds:[eax+edi], 1  

11238A3F  |.  8D56 FF       ||lea     edx, dword ptr ds:[esi-1]  

11238A42  |.  8BCA          ||mov     ecx, edx  

11238A44  |.  83E1 1F       ||and     ecx, 1F  

11238A47  |.  BB 01000000   ||mov     ebx, 1  
```



```assembly
11238A4C  |.  D3E3          ||shl      ebx, cl
11238A4E  |.  8B4D 08       ||mov      ecx, dword ptr ss:[ebp+8]
11238A51  |.  C1FA 05       ||sar      edx, 5
11238A54  |.  855C91 08     ||test     dword ptr ds:[ecx+edx*4+8], eb>
11238A58  |.  74 09         ||je       short 11238A63
11238A5A  |.  8A0C38        ||mov      cl, byte ptr ds:[eax+edi]
11238A5D  |.  80C1 01       ||add      cl, 1
11238A60  |.  880C38        ||mov      byte ptr ds:[eax+edi], cl
11238A63  |>  D02438        ||shl      byte ptr ds:[eax+edi], 1
11238A66  |.  8D56 FE       ||lea      edx, dword ptr ds:[esi-2]
11238A69  |.  8BCA          ||mov      ecx, edx
11238A6B  |.  83E1 1F       ||and      ecx, 1F
11238A6E  |.  BB 01000000   ||mov      ebx, 1
11238A73  |.  D3E3          ||shl      ebx, cl
11238A75  |.  8B4D 08       ||mov      ecx, dword ptr ss:[ebp+8]
11238A78  |.  C1FA 05       ||sar      edx, 5
11238A7B  |.  855C91 08     ||test     dword ptr ds:[ecx+edx*4+8], eb>
11238A7F  |.  74 09         ||je       short 11238A8A
11238A81  |.  8A0C38        ||mov      cl, byte ptr ds:[eax+edi]
11238A84  |.  80C1 01       ||add      cl, 1
11238A87  |.  880C38        ||mov      byte ptr ds:[eax+edi], cl
11238A8A  |>  83EE 04       ||sub      esi, 4
11238A8D  |.  836D F4 01    ||sub      dword ptr ss:[ebp-C], 1
11238A91  |.^ 0F85 59FFFFFF |\jnz      112389F0
11238A97  |.  8B75 0C       |mov      esi, dword ptr ss:[ebp+C]
11238A9A  |.  8345 F8 08    |add      dword ptr ss:[ebp-8], 8
11238A9E  |.  83C0 01       |add      eax, 1
11238AA1  |.  3BC6          |cmp      eax, esi
11238AA3  |.^ 0F82 37FFFFFF \jb       112389E0
11238AA9  |>  8D55 80       lea      edx, dword ptr ss:[ebp-80]
11238AAC  |.  52            push     edx                        ;  edx 为 sha 的数据结构
11238AAD  |.  E8 6E93EDFF   call     #207                       ;  sha 初始化
11238AB2  |.  56            push     esi                        ; |esi 为 edi 的长度
11238AB3  |.  8D45 80       lea      eax, dword ptr ss:[ebp-80]     ; |
11238AB6  |.  57            push     edi                        ; |edi 为分离出的 11 位数组
11238AB7  |.  50            push     eax                        ; |edx 为 sha 的数据结构
11238AB8  |.  E8 A393EDFF   call     #206                       ; \sha update 11 位值
11238ABD  |.  8B75 10       mov      esi, dword ptr ss:[ebp+10]
11238AC0  |.  8BC6          mov      eax, esi
11238AC2  |.  83C4 10       add      esp, 10
11238AC5  |.  8D50 01       lea      edx, dword ptr ds:[eax+1]
11238AC8  |>  8A08          /mov     cl, byte ptr ds:[eax]
11238ACA  |.  83C0 01       |add     eax, 1
11238ACD  |.  84C9          |test    cl, cl
11238ACF  |.^ 75 F7         \jnz     short 11238AC8
11238AD1  |.  2BC2          sub      eax, edx
11238AD3  |.  50            push     eax                        ;  eax 为 esi 的长度
11238AD4  |.  8D4D 80       lea      ecx, dword ptr ss:[ebp-80]
11238AD7  |.  56            push     esi                        ;  esi 为 11234201 代码处返回的数组，11234201 位于注册算法开始的地方
11238AD8  |.  51            push     ecx                        ;  ecx 为 sha 的数据结构
11238AD9  |.  E8 8293EDFF   call     #206                       ;  sha update 循环提供的常量数组
11238ADE  |.  8B75 18       mov      esi, dword ptr ss:[ebp+18]
11238AE1  |.  8B5D 14       mov      ebx, dword ptr ss:[ebp+14]
11238AE4  |.  56            push     esi                        ;  esi 为 ebx 的长度
11238AE5  |.  8D55 80       lea      edx, dword ptr ss:[ebp-80]
11238AE8  |.  53            push     ebx                        ;  ebx 为上层 11238B62 代码处提供的数组
11238AE9  |.  52            push     edx                        ;  ecx 为 sha 的数据结构
11238AEA  |.  E8 7193EDFF   call     #206                       ;  sha update 固定数组
11238AEF  |.  8D45 80       lea      eax, dword ptr ss:[ebp-80]
11238AF2  |.  50            push     eax
11238AF3  |.  8D4D DC       lea      ecx, dword ptr ss:[ebp-24]
11238AF6  |.  51            push     ecx                        ;  ecx 为 hash 后返回数组地址
11238AF7  |.  E8 243CF1FF   call     #205                       ;  sha final 处理
11238AFC  |.  56            push     esi                        ; |/n = 20 (32.)
11238AFD  |.  6A 00         push     0                           ; ||c = 00
11238AFF  |.  53            push     ebx                        ; ||s
11238B00  |.  E8 7BB90F00   call     <jmp.&MSVCR80.memset>      ; |\memset
11238B05  |.  8B75 1C       mov      esi, dword ptr ss:[ebp+1C]     ; |
11238B08  |.  6A 28         push     28                         ; |Arg1 = 00000028
11238B0A  |.  8D4D DC       lea      ecx, dword ptr ss:[ebp-24]     ; |
11238B0D  |.  E8 EEFDFFFF   call     11238900                     ; \这个函数比较简单，对 hash 后的 20 位值
11238B12  |.  83C4 30       add      esp, 30                      ;  通过处理，异或转变为 5 位数值存入到 esi 的地址
11238B15  |.  5B            pop      ebx
11238B16  |>  57            push     edi                        ; /block
11238B17  |.  FF15 BC253911 call     dword ptr ds:[<&MSVCR80.free>]  ; \free
11238B1D  |.  83C4 04       add      esp, 4
11238B20  |.  5F            pop      edi
11238B21  |.  5E            pop      esi
11238B22  |.  8BE5          mov      esp, ebp
11238B24  |.  5D            pop      ebp
11238B25  \.  C3            retn
```

进入 `11238900`，通过 `ida` 分析的代码如下:

```c
v4 = (unsigned __int8)(*(__int8 *)(a1 + 4) ^ *(__int8 *)(a1 + 9) ^ *(__int8 *)(a1 + 14) ^ *(__int8 *)(a1 + 19));
HIDWORD(v5) = (unsigned __int64)(v4 << 8) >> 32;
LODWORD(v5) = (unsigned __int8)(*(__int8 *)(a1 + 3) ^ *(__int8 *)(a1 + 8) ^ *(__int8 *)(a1 + 13) ^ *(__int8 *)(a1 + 18)) | ((__DWORD)v4 << 8);
HIDWORD(v5) = (unsigned __int64)(v5 << 8) >> 32;
LODWORD(v5) = (unsigned __int8)(*(__int8 *)(a1 + 2) ^ *(__int8 *)(a1 + 7) ^ *(__int8 *)(a1 + 12) ^ *(__int8 *)(a1 + 17)) | ((__DWORD)v5 << 8);
HIDWORD(v5) = (unsigned __int64)(v5 << 8) >> 32;
LODWORD(v5) = (unsigned __int8)(*(__int8 *)(a1 + 1) ^ *(__int8 *)(a1 + 6) ^ *(__int8 *)(a1 + 11) ^ *(__int8 *)(a1 + 16)) | ((__DWORD)v5 << 8);
result = (unsigned __int8)(*(__int8 *)a1 ^ *(__int8 *)(a1 + 5) ^ *(__int8 *)(a1 + 10) ^ *(__int8 *)(a1 + 15));
*(__DWORD *)a2 = result | ((__DWORD)v5 << 8);
*(__DWORD *)(a2 + 4) = (unsigned __int64)(v5 << 8) >> 32;
```

简单来说，就是 `sha` 后的 20 位值分别进行异或，产生 5 位的值，对压入 `11238b30` 第三个地址赋值，至此返回到 `1123A2E7`，判断对比 `ebx`，`edi` 的值是否相同。

然后程序再返回到 `11234286`，再到 `11233D80` 来判断时间数组，进入 `11233D80`。

```assembly
11233E4A  |.  8B93 10020000 mov      edx, dword ptr ds:[ebx+210]
11233E50  |.  52            push     edx                        ; /Arg2
11233E51  |.  56            push     esi                        ; |Arg1
11233E52  |.  E8 19EEEDFF   call     11112C70                   ; \vmwareba.11112C70
11233E57  |.  83C4 08       add      esp, 8
11233E5A  |>  80BB EC010000>cmp     byte ptr ds:[ebx+1EC], 0          ;  前 2 4 字节随便为 0
11233E61  |.  0F84 56030000 je       112341BD
11233E67  |.  83BB F0010000>cmp     dword ptr ds:[ebx+1F0], 0        ;  判断该位是否为 0,不为 0 有时间限制，0 没时间限制
11233E6E  |.  0F86 49030000 jbe      112341BD
```

`11233D80` 其他的代码不多说，这里只说 `11233E67`，`[ebx+1F0]` 主要是 `11239EF0` 作处理的，可参考之前 `11239EF0` 的说明。

### 【软件加密思路】

1. 在一个循环中，分别获取注册码，在索引数组中获取索引值，根据索引值获取注册码的值，计算注册码在固定数组中的位置为 `Index`。
   转换数组 `A` 循环从 `0x7C` 位开始到 `0` 位结束，每一循环处理 5 位，每一次判断 `Index` 低位为 1 时，对其数组 `A` 置 1，否则不作处理，然后 `Index` 右移一位。
2. 根据步骤 1 数组 `A` 进行处理，循环控制从 `0` 到 `105` 位之间的处理，分别获取当前位数的值 `C`，内循环控制 `0` 到 `20` 位,分别对 3 求余，然后根据求余的值，去到新数组 `B`，从 `0`，`40`，`80` 位开始的索引范围内，然后根据值 `C` 判断是否对数组 `B` 当前位置 1，然后索引加一，最后从 `107` 位开始到 `128` 位，直接原值到 `B` 数组。
3. 经过上述两步后，还原出数组 `B`，然后经过 `0x11239CD0` 的处理，获取第 14 位的值，判断是否在合理值范围内。
4. 根据数组 `B`，然后经过 `0x11239CD0` 的处理，判断第 9 位的值，是否有时间限制，产生时间数组 `C`。



### 5：分离前 5 位，后 11 位
对 `(11 位字节 + 注册算法循环提供的固定字节数组 + 算法内 sha 固定字节数组)` 进行 `sha` 散列。`sha` 后的 20 位值分别进行 `call 11238900` 异或处理，产生 5 位的新值，然后与之前的前 5 位比较，是否相同。

### 6：对时间数组 C 进行计算
计算限制日期，至此，完成注册过程。

## 【逆向解密思路】

1. 随机产生 11 位字节，第 4 位为日期验证值，要求为 0，第 9 位为范围验证值，异或 `0x1F`，第 11 位为 0 到 `0x1f`。
2. 对 `(11 位字节 + 注册算法循环提供的固定字节数组 + 算法内 sha 固定字节数组)` 进行 `sha` 散列。
3. `sha` 后的 20 位值分别进行 `call 11238900` 异或处理，产生 5 位的值，然后跟 11 位字节组合，产生 16 位字节。
4. 第一次还原操作，还原 3 步骤后的 16 位数组，两个循环，第一个循环控制 0 到 20 位，分别对 3 求余，然后根据求余的值，分别判断其从 0，40，80 位开始的值是否为 1，然后写入第二个循环的 0 到 105 位的新数组范围内，最后，从 107 位开始到 128 位，直接复制值。
5. 还原到原始 `key` 值，根据 4 步骤后的数组进行操作，用一个循环，循环从 `0x7C` 开始到 0 结束，每一次获取 5 位，转换为其 `key` 值，用 `key` 值根据其数组获取数字值，在循环中获取数组索引，然后写入数字值到相应数组位置中。

## 【经验总结】

发觉软件的加密者擅长用位操作，给解密者带来一定困难，还有验证地方也不少，需要很大的耐性和恒心，不过做出注册机还是一件很兴奋的事！

![使眼色](img/f3773294bf3a914c9a8c0c242528a003.jpg)

## 【版权声明】

本文原创于看雪技术论坛，转载请注明作者并保持文章的完整，谢谢！

2011 年 3 月 6 日 by `Daniel`

### 上传的附件

![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) `VMware713 keymaker.rar`
