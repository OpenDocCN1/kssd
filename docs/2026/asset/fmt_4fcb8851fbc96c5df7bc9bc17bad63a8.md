
## 反汇编代码片段

```
0041C845  |.  C74424 24 000>mov     dword ptr [esp+24], 42480000

0041C84D  |.  C74424 28 000>mov     dword ptr [esp+28], 41C80000

0041C855  |.  C74424 2C 000>mov     dword ptr [esp+2C], 42600000

0041C85D  |.  C74424 30 000>mov     dword ptr [esp+30], 41F00000

0041C865  |.  C74424 34 000>mov     dword ptr [esp+34], 42100000

0041C86D  |.  C74424 38 000>mov     dword ptr [esp+38], 42280000

0041C875  |.  C74424 3C 000>mov     dword ptr [esp+3C], 423C0000

0041C87D  |.  33F6          xor     esi, esi

0041C87F  |.  8D7C24 10     lea     edi, [esp+10]

0041C883  |>  D907          /fld     dword ptr [edi]                    ;  装入实数到st(0)

0041C885  |.  DC0D B0274A00 |fmul    qword ptr [4A27B0]                 ;  乘上一个实数

0041C88B  |.  83FE 0B       |cmp     esi, 0B

0041C88E  |.  D9FE          |fsin                                       ;  正弦函数sin

0041C890  |.  DFAD 16060000 |fild    qword ptr [ebp+616]                ;  装入整数到st(0)

0041C896  |.  DEC9          |fmulp   st(1), st                          ;  乘上一个实数

0041C898  |.  7D 0D         |jge     short 0041C8A7

0041C89A  |.  E8 31950700   |call    <jmp.&MSVCRT.\_ftol>                ;  浮点数转换成长整型数

0041C89F  |.  99            |cdq                                        ;  eax为返回值

0041C8A0  |.  B9 0A000000   |mov     ecx, 0A

0041C8A5  |.  EB 0B         |jmp     short 0041C8B2

0041C8A7  |>  E8 24950700   |call    <jmp.&MSVCRT.\_ftol>

0041C8AC  |.  99            |cdq

0041C8AD  |.  B9 1A000000   |mov     ecx, 1A

0041C8B2  |>  F7F9          |idiv    ecx

0041C8B4  |.  46            |inc     esi

0041C8B5  |.  83C7 04       |add     edi, 4

0041C8B8  |.  83FE 0C       |cmp     esi, 0C

0041C8BB  |.  88541E FF     |mov     [esi+ebx-1], dl                    ;  保存余数

0041C8BF  |.^ 7C C2         \jl      short 0041C883

0041C8C1  |.  5F            pop     edi

0041C8C2  |.  5E            pop     esi

0041C8C3  |.  5D            pop     ebp

0041C8C4  |.  B8 01000000   mov     eax, 1

0041C8C9  |.  5B            pop     ebx

0041C8CA  |.  83C4 30       add     esp, 30

0041C8CD  \.  C2 0800       retn    8
}}}
```

```
0041C8FC  |.  85C0          test    eax, eax                            ;  循环结束后的计算结果为

0041C8FE  |.  75 07         jnz     short 0041C907                      ;  0012F294  04 02 08 07 07 08 06 01 01 08 05 00

0041C900  |.  5E            pop     esi

0041C901  |.  83C4 28       add     esp, 28

0041C904  |.  C2 0800       retn    8

0041C907  |> \DD05 50294A00 fld     qword ptr [4A2950]                  ;  装入实数到st(0)

0041C90D  |.  D9FE          fsin                                        ;  正弦函数sin

0041C90F  |.  8B5424 25     mov     edx, [esp+25]

0041C913  |.  8B4424 26     mov     eax, [esp+26]

0041C917  |.  81E2 FF000000 and     edx, 0FF                            ;  edx=8

0041C91D  |.  25 FF000000   and     eax, 0FF                            ;  eax=5

0041C922  |.  895424 04     mov     [esp+4], edx

0041C926  |.  8B4C24 21     mov     ecx, [esp+21]

0041C92A  |.  81E1 FF000000 and     ecx, 0FF                            ;  ecx=8

0041C930  |.  57            push    edi

0041C931  |.  DD5C24 18     fstp    qword ptr [esp+18]                  ;  保存实数st(0)到[esp+18]，然后出栈

0041C935  |.  DD05 48294A00 fld     qword ptr [4A2948]                  ;  装入实数到st(0)

0041C93B  |.  D9FE          fsin                                        ;  正弦函数sin

0041C93D  |.  DD5C24 10     fstp    qword ptr [esp+10]                  ;  保存实数st(0)到[esp+10]，然后出栈

0041C941  |.  DD05 40294A00 fld     qword ptr [4A2940]                  ;  装入实数到st(0)

0041C947  |.  D9FE          fsin                                        ;  正弦函数sin

0041C949  |.  DB4424 08     fild    dword ptr [esp+8]                   ;  装入整数到st(0)

0041C94D  |.  894424 08     mov     [esp+8], eax                        ;  eax=5

0041C951  |.  DEC9          fmulp   st(1), st                           ;  st(1) <- st(0) \* st(1)，然后出栈

0041C953  |.  DB4424 08     fild    dword ptr [esp+8]                   ;  装入整数到st(0)

0041C957  |.  894C24 08     mov     [esp+8], ecx                        ;  ecx=8

0041C95B  |.  DC4C24 10     fmul    qword ptr [esp+10]                  ;  乘上实数[esp+10]

0041C95F  |.  DEC1          faddp   st(1), st                           ;  st(1) <- st(1) + st(0)，然后出栈

0041C961  |.  DB4424 08     fild    dword ptr [esp+8]                   ;  装入整数到st(0)

0041C965  |.  DC4C24 18     fmul    qword ptr [esp+18]                  ;  乘上实数[esp+18]

0041C969  |.  DEC1          faddp   st(1), st                           ;  st(1) <- st(1) + st(0)，然后出栈

0041C96B  |.  E8 60940700   call    <jmp.&MSVCRT.\_ftol>                 ;  浮点数转换成长整型数

0041C970  |.  99            cdq                                         ;  eax为返回值

0041C971  |.  B9 0A000000   mov     ecx, 0A

0041C976  |.  F7F9          idiv    ecx

0041C978  |.  8BCE          mov     ecx, esi

0041C97A  |.  52            push    edx

0041C97B  |.  E8 D0010000   call    0041CB50                            ;  根据余数查表得计算结果

0041C980  |.  DD05 38294A00 fld     qword ptr [4A2938]                  ;  下面运算部分跟上面的类似，不再做详细分析了

0041C986  |.  D9FE          fsin                                        ;  注意eax,ecx,edx的取值就可以了

0041C988  |.  8B5424 25     mov     edx, [esp+25]

0041C98C  |.  8B7C24 34     mov     edi, [esp+34]

0041C990  |.  81E2 FF000000 and     edx, 0FF                            ;  edx=8

0041C996  |.  8B4C24 22     mov     ecx, [esp+22]

0041C99A  |.  895424 34     mov     [esp+34], edx

0041C99E  |.  8847 02       mov     [edi+2], al                         ;  保存计算结果[12F2B0]

0041C9A1  |.  8B4424 26     mov     eax, [esp+26]

0041C9A5  |.  81E1 FF000000 and     ecx, 0FF                            ;  ecx=8

0041C9AB  |.  25 FF000000   and     eax, 0FF                            ;  eax=6

0041C9B0  |.  DD5C24 08     fstp    qword ptr [esp+8]

0041C9B4  |.  DD05 30294A00 fld     qword ptr [4A2930]

0041C9BA  |.  D9FE          fsin

0041C9BC  |.  DB4424 34     fild    dword ptr [esp+34]

0041C9C0  |.  894424 34     mov     [esp+34], eax

0041C9C4  |.  DEC9          fmulp   st(1), st

0041C9C6  |.  DD05 28294A00 fld     qword ptr [4A2928]

0041C9CC  |.  D9FE          fsin

0041C9CE  |.  DB4424 34     fild    dword ptr [esp+34]

0041C9D2  |.  894C24 34     mov     [esp+34], ecx

0041C9D6  |.  DEC9          fmulp   st(1), st

0041C9D8  |.  DEC1          faddp   st(1), st

0041C9DA  |.  DB4424 34     fild    dword ptr [esp+34]

0041C9DE  |.  DC4C24 08     fmul    qword ptr [esp+8]

0041C9E2  |.  DEC1          faddp   st(1), st

0041C9E4  |.  E8 E7930700   call    <jmp.&MSVCRT.\_ftol>

0041C9E9  |.  99            cdq

0041C9EA  |.  B9 0A000000   mov     ecx, 0A

0041C9EF  |.  F7F9          idiv    ecx

0041C9F1  |.  8BCE          mov     ecx, esi

0041C9F3  |.  52            push    edx

0041C9F4  |.  E8 57010000   call    0041CB50
......（略）
```

### 第二部分验证分析过程

运行程序输入注册码`"8301480573"`，提示注册成功后退出程序，把注册码写入注册表中。

Ctrl+F2重新载入程序，F9运行，程序被断下。

```
0041CAED  |.  6A 0A         push    0A                                ; /Arg2 = 0000000A

0041CAEF  |.  894424 09     mov     [esp+9], eax                      ; |

0041CAF3  |.  52            push    edx                               ; |Arg1

0041CAF4  |.  C64424 08 00  mov     byte ptr [esp+8], 0               ; |

0041CAF9  |.  884424 11     mov     [esp+11], al                      ; |

0041CAFD  |.  E8 CEFDFFFF   call    0041C8D0                          ; \\*\*\*算法call\*\*\* 断在此处

0041CB02  |.  85C0          test    eax, eax
```

这里是第一部分的验证过程，Ctrl+F9返回上一级调用到`0043DB66`处，下断bp `0043DB21`，Ctrl+F2重新载入程序。

F9运行，程序断在`0043DB21`处，F8到`0043DB27`，此时`edx`为假码存放地址`003F4FC8`，在此地址下硬件访问断点，

F9运行程序

```
0043DB10  /$  83EC 0C       sub     esp, 0C

0043DB13  |.  53            push    ebx

0043DB14  |.  56            push    esi

0043DB15  |.  8BF1          mov     esi, ecx

0043DB17  |.  33C0          xor     eax, eax

0043DB19  |.  8B4C24 18     mov     ecx, [esp+18]

0043DB1D  |.  894424 09     mov     [esp+9], eax
```
