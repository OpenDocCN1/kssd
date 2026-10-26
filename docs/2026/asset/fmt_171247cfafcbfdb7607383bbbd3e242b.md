
`00435D35`  `|.  FF15 0CE34500 call    dword ptr [<&KERNEL32.GetTickCou>; [GetTickCount`

`00435D3B`  `|.  8B4424 28     mov     eax, dword ptr [esp+28]          ;  eax被直接赋值,所以上面3个函数为干扰函数`

`00435D3F`  `|.  8B5424 30     mov     edx, dword ptr [esp+30]`

`00435D43`  `|.  8B8C24 840000>mov     ecx, dword ptr [esp+84]`

`00435D4A`  `|.  33C2          xor     eax, edx                         ;  eax = esp\_28 ^ esp\_30`

`00435D4C`  `|.  81F1 ECA89458 xor     ecx, 5894A8EC                    ;  ecx = esp\_84 ^ 0x5894A8EC`

`00435D52`  `|.  3D EA4F7A89   cmp     eax, 897A4FEA`

`00435D57`  `|.  0F85 99000000 jnz     00435DF6                         ;  分支`

`00435D5D`  `|.  8B4424 70     mov     eax, dword ptr [esp+70]`

`00435D61`  `|.  8B7424 2C     mov     esi, dword ptr [esp+2C]`

`00435D65`  `|.  33C6          xor     eax, esi`

`00435D67`  `|.  3D 544F7A98   cmp     eax, 987A4F54`

`00435D6C`  `|.  74 49         je      short 00435DB7                   ;  if((esp\_70 ^ esp\_2c) == 0x987A4F54) jmp`

`00435D6E`  `|.  8B4424 68     mov     eax, dword ptr [esp+68]`

`00435D72`  `|.  8B7C24 50     mov     edi, dword ptr [esp+50]`

`00435D76`  `|.  8B4C24 6C     mov     ecx, dword ptr [esp+6C]`

`00435D7A`  `|.  8B6C24 60     mov     ebp, dword ptr [esp+60]`

`00435D7E`  `|.  F7D0          not     eax`

`00435D80`  `|.  23C7          and     eax, edi`

`00435D82`  `|.  33D2          xor     edx, edx`

`00435D84`  `|.  F7D1          not     ecx`

`00435D86`  `|.  33C5          xor     eax, ebp`

`00435D88`  `|.  23CA          and     ecx, edx`

`00435D8A`  `|.  F7D0          not     eax`

`00435D8C`  `|.  33CA          xor     ecx, edx`

`00435D8E`  `|.  3D 2802C042   cmp     eax, 42C00228`

`00435D93`  `|.  F7D1          not     ecx`

`00435D95`  `|.  0F85 12010000 jnz     00435EAD`

`00435D9B`  `|.  81F9 8AE96598 cmp     ecx, 9865E98A                    ;  ecx = ^(x & 0 ^ 0)`

`00435DA1`  `|.  0F85 06010000 jnz     00435EAD                         ;  go to failure`

`00435DA7`  `|.  5F            pop     edi                              ;  not enter`

`00435DA8`  `|.  5E            pop     esi`

`00435DA9`  `|.  5D            pop     ebp`

`00435DAA`  `|.  B8 01000000   mov     eax, 1                            ;这是一个比较好的爆破点,可以修改程序的开头直接跳到这个位置`

`00435DAF`  `|.  5B            pop     ebx`

`00435DB0`  `|.  81C4 C8000000 add     esp, 0C8`

`00435DB6`  `|.  C3            retn`

`00435DB7`  `|>  B8 F1F0F0F0   mov     eax, F0F0F0F1`

`00435DBC`  `|.  F7E1          mul     ecx                              ;  ecx = esp\_84 ^ 0x5894A8EC 从前面copy的`

`00435DBE`  `|.  C1EA 04       shr     edx, 4`

`00435DC1`  `|.  B8 4FECC44E   mov     eax, 4EC4EC4F`

`00435DC6`  `|.  8BFA          mov     edi, edx`

`00435DC8`  `|.  F7E1          mul     ecx`

`00435DCA`  `|.  0FAFFE        imul    edi, esi`

`00435DCD`  `|.  8B4424 28     mov     eax, dword ptr [esp+28]`

`00435DD1`  `|.  8B4C24 14     mov     ecx, dword ptr [esp+14]`

`00435DD5`  `|.  C1EA 02       shr     edx, 2`

`00435DD8`  `|.  0FAFF0        imul    esi, eax`

`00435DDB`  `|.  0FAFD0        imul    edx, eax`

`00435DDE`  `|.  03FA          add     edi, edx`

`00435DE0`  `|.  33C0          xor     eax, eax`

`00435DE2`  `|.  03FE          add     edi, esi`

`00435DE4`  `|.  F7D7          not     edi`

`00435DE6`  `|.  3BF9          cmp     edi, ecx                         ;  ~(esp\_28 \* esp\_2c + esp\_28 \* ecx / 13 + esp\_2c \* ecx / 17) == esp\_14;`

`00435DE8`  `|.  5F            pop     edi`

`00435DE9`  `|.  5E            pop     esi`

`00435DEA`  `|.  5D            pop     ebp`

`00435DEB`  `|.  0F94C0        sete    al                               ;  如果上面等式成立,则注册成功`

`00435DEE`  `|.  5B            pop     ebx`

`00435DEF`  `|.  81C4 C8000000 add     esp, 0C8`

`00435DF5`  `|.  C3            retn`

`00435DF6`  `|>  8B4424 40     mov     eax, dword ptr [esp+40]`

`00435DFA`  `|.  8B5424 48     mov     edx, dword ptr [esp+48]`

`00435DFE`  `|.  33C2          xor     eax, edx`

`00435E00`  `|.  8B5424 44     mov     edx, dword ptr [esp+44]`

`00435E04`  `|.  C74424 4C 000>mov     dword ptr [esp+4C], 0            ;  esp\_4c == 0`

`00435E0C`  `|.  335424 4C     xor     edx, dword ptr [esp+4C]          ;  edx = edx`

`00435E10`  `|.  3D E8005A5F   cmp     eax, 5F5A00E8`

`00435E15`  `|.  75 5A         jnz     short 00435E71`

`00435E17`  `|.  81FA E48954A6 cmp     edx, A65489E4`

`00435E1D`  `|.  75 52         jnz     short 00435E71                   ;   (esp\_48 ^ esp\_40) == 0x5F5A00E8 && esp\_44 == 0xA65489E4 如果不满足跳转00435E71`

`00435E1F`  `|.  B8 25499224   mov     eax, 24924925`

`00435E24`  `|.  0FAF7424 78   imul    esi, dword ptr [esp+78]`

`00435E29`  `|.  F7E1          mul     ecx                              ;  ecx = esp\_84 ^ 0x5894A8EC 从前面copy的`

`00435E2B`  `|.  8BC1          mov     eax, ecx`

`00435E2D`  `|.  2BC2          sub     eax, edx`

`00435E2F`  `|.  D1E8          shr     eax, 1`

`00435E31`  `|.  03C2          add     eax, edx`

`00435E33`  `|.  C1E8 02       shr     eax, 2`

`00435E36`  `|.  8BF8          mov     edi, eax`

`00435E38`  `|.  B8 CB6B28AF   mov     eax, AF286BCB`

`00435E3D`  `|.  F7E1          mul     ecx`

`00435E3F`  `|.  0FAF7C24 2C   imul    edi, dword ptr [esp+2C]`

`00435E44`  `|.  2BCA          sub     ecx, edx`

`00435E46`  `|.  33C0          xor     eax, eax`

`00435E48`  `|.  D1E9          shr     ecx, 1`

`00435E4A`  `|.  03CA          add     ecx, edx`

`00435E4C`  `|.  C1E9 04       shr     ecx, 4`

`00435E4F`  `|.  0FAF4C24 28   imul    ecx, dword ptr [esp+28]`

`00435E54`  `|.  03F9          add     edi, ecx`

`00435E56`  `|.  8B8C24 D00000>mov     ecx, dword ptr [esp+D0]`

`00435E5D`  `|.  03FE          add     edi, esi`

`00435E5F`  `|.  F7D7          not     edi`

`00435E61`  `|.  3BF9          cmp     edi, ecx`

`00435E63`  `|.  5F            pop     edi`

`00435E64`  `|.  5E            pop     esi`

`00435E65`  `|.  5D            pop     ebp`

`00435E66`  `|.  0F94C0        sete    al                               ;   return ~(esp\_78 \* esi + esp\_28 \* ecx / 19 + esp\_2c \* ecx /7) == esp\_d0`

`00435E69`  `|.  5B            pop     ebx`

`00435E6A`  `|.  81C4 C8000000 add     esp, 0C8`

`00435E70`  `|.  C3            retn`

`00435E71`  `|>  8B8424 980000>mov     eax, dword ptr [esp+98]`

`00435E78`  `|.  8B8C24 9C0000>mov     ecx, dword ptr [esp+9C]`

`00435E7F`  `|.  8B9424 940000>mov     edx, dword ptr [esp+94]`

`00435E86`  `|.  23C7          and     eax, edi`

`00435E88`  `|.  23CA          and     ecx, edx`

`00435E8A`  `|.  33C3          xor     eax, ebx`

`00435E8C`  `|.  33CD          xor     ecx, ebp`

`00435E8E`  `|.  3D 4F79AE48   cmp     eax, 48AE794F`

`00435E93`  `|.  75 18         jnz     short 00435EAD`

`00435E95`  `|.  81F9 34023784 cmp     ecx, 84370234                    ;  ((esp\_98 & edi) ^ ebx == 0x48AE794F) && (((esp\_9c & esp\_94) ^ ebp) == 0x84370234)`

`00435E9B`  `|.  75 10         jnz     short 00435EAD`

`00435E9D`  `|.  5F            pop     edi`

`00435E9E`  `|.  5E            pop     esi`

`00435E9F`  `|.  5D            pop     ebp`

`00435EA0`  `|.  B8 01000000   mov     eax, 1`

`00435EA5`  `|.  5B            pop     ebx`

`00435EA6`  `|.  81C4 C8000000 add     esp, 0C8`

`00435EAC`  `|.  C3            retn`

`00435EAD`  `|>  5F            pop     edi`

`00435EAE`  `|.  5E            pop     esi`

`00435EAF`  `|.  5D            pop     ebp`

`00435EB0`  `|.  33C0          xor     eax, eax`

`00435EB2`  `|.  5B            pop     ebx`

`00435EB3`  `|.  81C4 C8000000 add     esp, 0C8`

`00435EB9`  `\.  C3            retn`

总结：改注册码验证函数使用了一下几个手段来增加逆向难度。

1. 使用了8个整数的值来做注册码，使穷举变得基本不可能。
2. 使用了与运算这种单向不可逆运算，模糊运算的中间值和注册码本身的关系。
3. 加入无用代码，如`GetTickCount`等3个函数的调用。
4. 增加了无效分支，用随机值来填充中间数据，把逆向者引入歧途。
5. 函数分支众多。

读者可以根据上面描述的计算步骤，自己尝试写出注册算法。

### 5. 拆解时间保护

如果采用爆破的方法,在试用30天后,你会发现程序启动5分钟后,还是会弹出提示过了试用期,确定后退出.显然这里存在个计时器,在启动后再次进行检查.

在所有的`SetTimer`参考上下断点,

参考位于 `HooNetMe:.text` 到 `USER32.SetTimer`

```
0040A1E9   call    dword ptr [<&USER32.SetTimer>]    USER32.SetTimer
0040CB40   call    dword ptr [<&USER32.SetTimer>]    USER32.SetTimer
0040F1FF   call    dword ptr [<&USER32.SetTimer>]    USER32.SetTimer
0041152F   call    dword ptr [<&USER32.SetTimer>]    USER32.SetTimer
004118CD   call    dword ptr [<&USER32.SetTimer>]    USER32.SetTimer
00419ACA   call    dword ptr [<&USER32.SetTimer>]    USER32.SetTimer      这是我们要找的目标
```
