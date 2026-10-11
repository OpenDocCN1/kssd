
## 代码段分析

```
004A82AD   |.   E8 22C5F5FF    call    004047D4  
004A82B2   \.   C3            retn  
```

我们从上面的 `retn` 出来到 `004A8283`，这里又是新的一个程序段……我们往上看……那么出错提示框就是在 `004A827E` 这个 `Call`，上面有个 `Jmp` 跳过了注册失败提示！但是在看看……`004A8264` 这里有个接受的跳转……说明如果上面有个跳转，如果哪个跳转不实现，那么就有可能经过 `Jmp` 从而跳过了注册失败的提示！我们就右击 `004A8264` 选择转到……我们转到这里……

```
004A8134   /$   55            push    ebp  
004A8135   |.   8BEC         mov     ebp, esp  
004A8137   |.   B9 06000000  mov     ecx, 6  
004A813C   |>   6A 00        /push    0  
004A813E   |.   6A 00        |push    0  
004A8140   |.   49          |dec     ecx  
004A8141   |.^  75 F9        \jnz     short 004A813C  
004A8143   |.   51          push    ecx  
004A8144   |.   53          push    ebx  
004A8145   |.   56          push    esi  
004A8146   |.   8BF0        mov     esi, eax  
004A8148   |.   33C0        xor     eax, eax  
004A814A   |.   55          push    ebp  
004A814B   |.   68 B3824A00 push    004A82B3  
004A8150   |.   64:FF30     push    dword ptr fs:[eax]  
004A8153   |.   64:8920     mov     fs:[eax], esp  
004A8156   |.   8D55 F4     lea     edx, [ebp-C]  
004A8159   |.   8B86 48030000 mov   eax, [esi+348]  
004A815F   |.   E8 5011FCFF call    004692B4  
004A8164   |.   8B45 F4     mov     eax, [ebp-C]                ;  取出伪码  
004A8167   |.   8D55 F8     lea     edx, [ebp-8]  
004A816A   |.   E8 6139FEFF call    0048BAD0  
004A816F   |.   8B55 F8     mov     edx, [ebp-8]  
004A8172   |.   B8 4CED4C00 mov     eax, 004CED4C  
004A8177   |.   E8 88C6F5FF call    00404804  
004A817C   |.   E8 FBFDFFFF call    004A7F7C                  ;这里就是关键 CAll  
004A8181   |.   8BD8        mov     ebx, eax  
004A8183   |.   84DB        test    bl, bl  
004A8185   |.   0F84 D9000000 je    004A8264                  ;这个 je 关键  
```

我们取消先前的全部断点……在 `004A814A` 下断……`F8` 走～一下……发现关键 `Call` 跟进……到这里……

## 关键函数分析

```
004A7F7C   /$   53          push    ebx  
004A7F7D   |.   56          push    esi  
004A7F7E   |.   57          push    edi  
004A7F7F   |.   BF 4CED4C00 mov     edi, 004CED4C  
004A7F84   |.   33F6        xor     esi, esi  
004A7F86   |.   33DB        xor     ebx, ebx  
004A7F88   |.   8B07        mov     eax, [edi]  
004A7F8A   |.   E8 E1CAF5FF call    00404A70  
004A7F8F   |.   83F8 0E     cmp     eax, 0E                   ;注册码长度与 14 比较，不是 14 不行！  
004A7F92   |.   75 67       jnz     short 004A7FFB  
004A7F94   |.   8B07        mov     eax, [edi]  
004A7F96   |.   8038 36     cmp     byte ptr [eax], 36        ;  第 1 位与 6 比  
004A7F99   |.   0F94C0      sete    al                        ；如果相同就为 true!以下类似  
004A7F9C   |.   83E0 7F     and     eax, 7F  
004A7F9F   |.   03F0        add     esi, eax  
004A7FA1   |.   8B07        mov     eax, [edi]  
004A7FA3   |.   8078 02 32  cmp     byte ptr [eax+2], 32      ;  第 3 位与字符 2 比  
004A7FA7   |.   0F94C0      sete    al  
004A7FAA   |.   83E0 7F     and     eax, 7F  
004A7FAD   |.   03F0        add     esi, eax  
004A7FAF   |.   8B07        mov     eax, [edi]  
004A7FB1   |.   8078 03 31  cmp     byte ptr [eax+3], 31      ;  第 4 位与字符 1 比  
004A7FB5   |.   0F94C0      sete    al  
004A7FB8   |.   83E0 7F     and     eax, 7F  
004A7FBB   |.   03F0        add     esi, eax  
004A7FBD   |.   8B07        mov     eax, [edi]  
004A7FBF   |.   8078 04 39  cmp     byte ptr [eax+4], 39      ;  第 5 位与字符 9 比  
004A7FC3   |.   0F94C0      sete    al  
004A7FC6   |.   83E0 7F     and     eax, 7F  
004A7FC9   |.   03F0        add     esi, eax  
004A7FCB   |.   8B07        mov     eax, [edi]  
004A7FCD   |.   8078 07 30  cmp     byte ptr [eax+7], 30      ;  第 8 位与字符 0 比  
004A7FD1   |.   0F94C0      sete    al  
004A7FD4   |.   83E0 7F     and     eax, 7F  
004A7FD7   |.   03F0        add     esi, eax  
004A7FD9   |.   8B07        mov     eax, [edi]  
004A7FDB   |.   8078 08 35  cmp     byte ptr [eax+8], 35      ;  第 9 位与字符 5 比  
004A7FDF   |.   0F94C0      sete    al  
004A7FE2   |.   83E0 7F     and     eax, 7F  
004A7FE5   |.   03F0        add     esi, eax  
004A7FE7   |.   8B07        mov     eax, [edi]  
004A7FE9   |.   8078 0A 33  cmp     byte ptr [eax+A], 33      ;  第 11 位与字符 3 比  
004A7FED   |.   0F94C0      sete    al  
004A7FF0   |.   83E0 7F     and     eax, 7F  
004A7FF3   |.   03F0        add     esi, eax  
004A7FF5   |.   83FE 07     cmp     esi, 7  
004A7FF8   |.   0F94C3      sete    bl  
004A7FFB   |>   8BC3        mov     eax, ebx  
004A7FFD   |.   5F          pop     edi  
004A7FFE   |.   5E          pop     esi  
004A7FFF   |.   5B          pop     ebx  
004A8000   \.   C3          retn  
```

当七个比较都吻合时…`004A8185` 不实现……注册成功！

### 注册算法总结

1. 注册码长度为 `14`
2. 注册码格式为 `6?219??05?3???`

还有他们公司其他软件，也大同小异。就顺便走一下算法了！

我写了个注册机…其中包括：

```
"1st Security Agent V6.4"
"Access Controller V3.1"
"Best Network Security  V1.6"
"Hide Files and Folders V2.5"
"Pc Security Tweaker V6.5"
"Bulletproof Public PC V3.3"
"Outlook Express Privacy  V1.7"
```

大家可以试试看看……

`4dd466d0e1e0cc123afbe504da2462ca`
