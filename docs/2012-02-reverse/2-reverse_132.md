
# 使用 phantOm 1.2 插件注意了

> 原文：[`www.pediy.com/kssd/pediy10/81986.html`](https://www.pediy.com/kssd/pediy10/81986.html)

目前该问题只发现在 `1.20` 版本,系统为 `WIN2003` 才有这个问题,`XP` 就没有~~!

问题存在为 `Hook` `GetTickCount` 这个选项:

`phantOm` `Hook` 之后会是如下:

```
7C8219D1  >  $  /EB 07         jmp     short 7C8219DA
7C8219D3   >  |F3:           prefix rep:
7C8219D4   .  |90            nop
7C8219D5   .  |90            nop
7C8219D6   .  |90            nop
7C8219D7   .  |90            nop
7C8219D8   .  |90            nop
7C8219D9   .  |90            nop
7C8219DA   $  \\8BE9          mov     ebp, ecx   ;这里的汇编错误
7C8219DC   ?  90             nop
7C8219DD   ?  E6 85           out     85, al
7C8219DF   .  838B 152003FE>or      dword ptr [ebx+FE032015], 7F
7C8219E6   .  3B0D 2803FE7F  cmp     ecx, dword ptr [7FFE0328]
7C8219EC   .^ 75 E5           jnz     short 7C8219D3
7C8219EE   .  A1 0400FE7F     mov     eax, dword ptr [7FFE0004]
7C8219F3   .  F7E2            mul     edx
7C8219F5   .  C1E1 08          shl     ecx, 8
7C8219F8   .  0FAF0D 0400FE>imul    ecx, dword ptr [7FFE0004]
7C8219FF   .  0FACD0 18       shrd    eax, edx, 18
7C821A03   .  C1EA 18         shr     edx, 18
7C821A06   .  03C1            add     eax, ecx
7C821A08   .  C3              retn
```

//应该为:

```
7C8219D1  >  /EB 07           jmp     short 7C8219DA
7C8219D3   |F3:             prefix rep:
7C8219D4   |90               nop
7C8219D5   |90               nop
7C8219D6   |90               nop
7C8219D7   |90               nop
7C8219D8   |90               nop
7C8219D9   |90               nop
7C8219DA   \\8B0D 2403FE7F   mov     ecx, dword ptr [7FFE0324]
7C8219E0    8B15 2003FE7F   mov     edx, dword ptr [7FFE0320]
7C8219E6    3B0D 2803FE7F   cmp     ecx, dword ptr [7FFE0328]
7C8219EC  ^ 75 E5           jnz     short 7C8219D3
7C8219EE    A1 0400FE7F     mov     eax, dword ptr [7FFE0004]
7C8219F3    F7E2            mul     edx
7C8219F5    C1E1 08          shl     ecx, 8
7C8219F8    0FAF0D 0400FE7F imul    ecx, dword ptr [7FFE0004]
7C8219FF  - E9 6CE6A983     jmp     002C0070
7C821A04    90               nop
7C821A05    90               nop
7C821A06    03C1             add     eax, ecx
7C821A08    C3               retn
```

----------------------------------------------------------------------------------

修改方法为:

`winxp` 就不用管了，`win2003` 的话，需要修正，因为这个是代码上要调整，建议大家到资源下载区

下载 `HideOD` `v0.1.9.6` 这个版本，之前的版本存在和 `phantOm` 一样的问题，`0.1.9.6` 这个版本之后加了版本校验，`win2003` 和 `XP` 都就可以用哪~~！

这样 `Hook` `GetTickCount` 才会真正管用
