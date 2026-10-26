
``` 
00427D9F   .  51            push    ecx                              ; |hWnd

00427DA0   .  FF15 08E54500 call    dword ptr [<&USER32.PostMessageW>; \PostMessageW

00427DA6   >  8B96 A8000000 mov     edx, dword ptr [esi+A8]

00427DAC   .  52            push    edx                              ; /TimerID

00427DAD   .  6A 00         push    0                                ; |hWnd = NULL

00427DAF   .  FF15 04E54500 call    dword ptr [<&USER32.KillTimer>]  ; \KillTimer

00427DB5   .  5E            pop     esi

00427DB6   .  C2 1000       retn    10
```

虽然函数`00435EC0`和`serialNo_check2`的功能一致，但是他们属于不同的函数，所以仅仅爆破`serialNo_check2`是不行的。

### 6. 更进一步,寻找试用时间的相关信息的存储位置

寻找读取注册表的操作，发现该程序读取了一个`HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced\ShellExStateN`的注册表项。

通过查找`ShellExStateN`字符串的方法可以到来到一下代码处

``` 
004114D8   .  50            push    eax                              ; /pHandle

004114D9   .  6A 01         push    1                                ; |Access = KEY_QUERY_VALUE

004114DB   .  6A 00         push    0                                ; |Reserved = 0

004114DD   .  51            push    ecx                              ; |Subkey = "SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced"

004114DE   .  68 01000080   push    80000001                         ; |hKey = HKEY_CURRENT_USER

004114E3   .  FF15 04E04500 call    dword ptr [<&ADVAPI32.RegOpenKey>; \RegOpenKeyExW

004114E9   .  85C0          test    eax, eax

004114EB   .  75 34         jnz     short 00411521

004114ED   .  8D5424 14     lea     edx, dword ptr [esp+14]

004114F1   .  8D4424 2C     lea     eax, dword ptr [esp+2C]

004114F5   .  52            push    edx                              ; /pBufSize

004114F6   .  8B5424 14     mov     edx, dword ptr [esp+14]          ; |

004114FA   .  8D4C24 1C     lea     ecx, dword ptr [esp+1C]          ; |

004114FE   .  50            push    eax                              ; |Buffer            ;在Buffer地址处下内存访问断点

004114FF   .  51            push    ecx                              ; |pValueType

00411500   .  6A 00         push    0                                ; |Reserved = NULL

00411502   .  68 840A4600   push    00460A84                         ; |shellexstaten

00411507   .  52            push    edx                              ; |hKey

00411508   .  C74424 2C 3C0>mov     dword ptr [esp+2C], 3C           ; |

00411510   .  FF15 00E04500 call    dword ptr [<&ADVAPI32.RegQueryVa>; \RegQueryValueExW
```

下内存断点后，F9运行，然后停在下面的函数里面。

``` 
0040FC20  /$  8B4424 0C     mov     eax, dword ptr [esp+C]

0040FC24  |.  53            push    ebx

0040FC25  |.  55            push    ebp

0040FC26  |.  56            push    esi

0040FC27  |.  8B7424 10     mov     esi, dword ptr [esp+10]          ;参数1保存着注册表shellexstaten的键值

0040FC2B  |.  33C9          xor     ecx, ecx

0040FC2D  |.  8908          mov     dword ptr [eax], ecx

0040FC2F  |.  57            push    edi

0040FC30  |.  8B46 0C       mov     eax, dword ptr [esi+C]

0040FC33  |.  8B56 38       mov     edx, dword ptr [esi+38]

0040FC36  |.  33D0          xor     edx, eax

0040FC38  |.  81F2 C00E48AD xor     edx, AD480EC0

0040FC3E  |.  81FA 1A325684 cmp     edx, 8456321A

0040FC44  |.  0F85 42010000 jnz     0040FD8C

0040FC4A  |.  8B7E 04       mov     edi, dword ptr [esi+4]

0040FC4D  |.  8B16          mov     edx, dword ptr [esi]             ;内存访问断点断下位置

0040FC4F  |.  8B4E 10       mov     ecx, dword ptr [esi+10]

0040FC52  |.  8BDA          mov     ebx, edx

0040FC54  |.  8BEF          mov     ebp, edi

0040FC56  |.  81F3 C00E48AD xor     ebx, AD480EC0

0040FC5C  |.  81F5 C00E48AD xor     ebp, AD480EC0

0040FC62  |.  35 C00E48AD   xor     eax, AD480EC0

0040FC67  |.  03DD          add     ebx, ebp

0040FC69  |.  8BE9          mov     ebp, ecx

0040FC6B  |.  81F5 C00E48AD xor     ebp, AD480EC0

0040FC71  |.  03DD          add     ebx, ebp

0040FC73  |.  03D8          add     ebx, eax

0040FC75  |.  8B46 08       mov     eax, dword ptr [esi+8]

0040FC78  |.  35 C00E48AD   xor     eax, AD480EC0

0040FC7D  |.  03D8          add     ebx, eax

0040FC7F  |.  8B46 18       mov     eax, dword ptr [esi+18]

0040FC82  |.  F7D0          not     eax

0040FC84  |.  3BD8          cmp     ebx, eax

0040FC86  |.  0F85 F9000000 jnz     0040FD85

0040FC8C  |.  8B5E 24       mov     ebx, dword ptr [esi+24]

0040FC8F  |.  8B46 20       mov     eax, dword ptr [esi+20]

0040FC92  |.  8BEB          mov     ebp, ebx

0040FC94  |.  35 C18EF0A8   xor     eax, A8F08EC1

0040FC99  |.  81F5 C18EF0A8 xor     ebp, A8F08EC1

0040FC9F  |.  0FAFC5        imul    eax, ebp

0040FCA2  |.  8B6E 30       mov     ebp, dword ptr [esi+30]

0040FCA5  |.  F7D5          not     ebp

0040FCA7  |.  3BC5          cmp     eax, ebp

0040FCA9  |.  0F85 D6000000 jnz     0040FD85

0040FCAF  |.  8B46 14       mov     eax, dword ptr [esi+14]

0040FCB2  |.  33C1          xor     eax, ecx

0040FCB4  |.  35 C00E48AD   xor     eax, AD480EC0

0040FCB9  |.  3D 5F1E1AE6   cmp     eax, E61A1E5F

0040FCBE  |.  75 59         jnz     short 0040FD19

0040FCC0  |.  8B4E 1C       mov     ecx, dword ptr [esi+1C]

0040FCC3  |.  33CA          xor     ecx, edx

0040FCC5  |.  81F1 C00E48AD xor     ecx, AD480EC0

0040FCCB  |.  81F9 AC351F59 cmp     ecx, 591F35AC

0040FCD1  |.  75 46         jnz     short 0040FD19

0040FCD3  |.  8B56 28       mov     edx, dword ptr [esi+28]

0040FCD6  |.  33D7          xor     edx, edi

0040FCD8  |.  81F2 C00E48AD xor     edx, AD480EC0

0040FCDE  |.  81FA C4FE8426 cmp     edx, 2684FEC4

0040FCE4  |.  75 33         jnz     short 0040FD19

0040FCE6  |.  8B4E 2C       mov     ecx, dword ptr [esi+2C]

0040FCE9  |.  33CB          xor     ecx, ebx

0040FCEB  |.  81F1 C18EF0A8 xor     ecx, A8F08EC1

0040FCF1  |.  81F9 12E6F8A4 cmp     ecx, A4F8E612

0040FCF7  |.  75 20         jnz     short 0040FD19

0040FCF9  |.  8B56 34       mov     edx, dword ptr [esi+34]

0040FCFC  |.  8B5E 20       mov     ebx, dword ptr [esi+20]

0040FCFF  |.  33D3          xor     edx, ebx

0040FD01  |.  81F2 C18EF0A8 xor     edx, A8F08EC1

0040FD07  |.  81FA E8ECA531 cmp     edx, 31A5ECE8

0040FD0D  |.  75 0A         jnz     short 0040FD19

0040FD0F  |.  5F            pop     edi

0040FD10  |.  5E            pop     esi

0040FD11  |.  5D            pop     ebp

0040FD12  |.  B8 01000000   mov     eax, 1

0040FD17  |.  5B            pop     ebx

0040FD18  |.  C3            retn

0040FD19  |>  3D 118E5FA4   cmp     eax, A45F8E11

0040FD1E  |.  75 65         jnz     short 0040FD85

0040FD20  |.  33FF          xor     edi, edi

0040FD22  |.  57            push    edi

0040FD23  |.  E8 66830200   call    0043808E                         ;  time(0)

0040FD28  |.  894424 18     mov     dword ptr [esp+18], eax

0040FD2C  |.  8D4424 18     lea     eax, dword ptr [esp+18]

0040FD30  |.  50            push    eax

0040FD31  |.  E8 26860200   call    0043835C                         ;  localtime

0040FD36  |.  50            push    eax

0040FD37  |.  8978 08       mov     dword ptr [eax+8], edi

0040FD3A  |.  8978 04       mov     dword ptr [eax+4], edi

0040FD3D  |.  8938          mov     dword ptr [eax], edi

0040FD3F  |.  E8 26840200   call    0043816A                         ;  mktime

0040FD44  |.  8B4E 1C       mov     ecx, dword ptr [esi+1C]

0040FD47  |.  8B5E 24       mov     ebx, dword ptr [esi+24]

0040FD4A  |.  33CB          xor     ecx, ebx

0040FD4C  |.  83C4 0C       add     esp, 0C

0040FD4F  |.  81F1 C18EF0A8 xor     ecx, A8F08EC1                    ;  ecx = arg1[7] ^ arg1[9] ^ 0xA8F08EC1;表示上次程序运行的时间

0040FD55  |.  3BC1          cmp     eax, ecx                         ;  eax此时存储的为当前时间值；运算后为与上次运行时间的差

0040FD57  |.  7C 2C         jl      short 0040FD85

0040FD59  |.  2BC1          sub     eax, ecx                         ;  计算与程序上次运行时间的差(以秒为单位)

0040FD5B  |.  8B7E 08       mov     edi, dword ptr [esi+8]

0040FD5E  |.  8BC8          mov     ecx, eax

0040FD60  |.  B8 07452EC2   mov     eax, C22E4507

0040FD65  |.  F7E9          imul    ecx
```
