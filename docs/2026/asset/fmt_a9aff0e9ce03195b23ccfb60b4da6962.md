
# [原创]Phelios Super Sprites 1.61简单算法分析-菜鸟篇

Phelios Super Sprites 1.61简单算法分析-菜鸟篇

【文章作者】: tzl

【作者邮箱】: 无

【软件名称】: Phelios Super Sprites 1.61

【软件大小】: 499KB

【下载地址】: c68K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3&6W2N6$3S2#2j5g2)9J5k6h3y4G2L8g2)9J5c8Y4y4G2k6Y4c8Q4x3V1j5J5x3o6b7#2x3W2)9J5k6h3S2@1L8b7`.`.

【加壳方式】: 无

【保护方式】: 注册码

【编写语言】: CAN (Crunched ANsi) file

【使用工具】: OD PEID

【操作平台】: XP SP2

【软件介绍】: 制作及优化 Tile（动画小图标）的工具软件。

前几天工作很忙，今天放松一下，找了个体积最小的软件来练手，很幸运算法很简单，适合我这只小菜鸟，这里与大家分享一下，菜鸟共同进步。

## 一、查壳，无。

## 二、根据字符串相关信息

我们可以在这里下断开始分析，第一位用户名：`tigerisme`；第二位用户名：`tzl`；试练码：`123456789`

```
004152C0  /$  55            push    ebp
004152C1  |.  31C0          xor     eax, eax
004152C3  |.  89E5          mov     ebp, esp
004152C5  |.  53            push    ebx
004152C6  |.  56            push    esi
004152C7  |.  57            push    edi
004152C8  |.  BF 89444200   mov     edi, 00424489      ;  引入"supersprites"放到edi中，算注册码时用到，记做codeA
004152CD  |.  81EC 08060000 sub     esp, 608
004152D3  |.  83C9 FF       or      ecx, FFFFFFFF
004152D6  |.  F2:AE         repne   scas byte ptr es:[>
004152D8  |.  8B7D 10       mov     edi, [ebp+10]
004152DB  |.  C785 ECFAFFFF>mov     dword ptr [ebp-514>
004152E5  |.  298D ECFAFFFF sub     [ebp-514], ecx
004152EB  |.  8385 ECFAFFFF>add     dword ptr [ebp-514>
004152F2  |.  83C9 FF       or      ecx, FFFFFFFF
004152F5  |.  F2:AE         repne   scas byte ptr es:[>
004152F7  |.  BF FEFFFFFF   mov     edi, -2
004152FC  |.  29CF          sub     edi, ecx
004152FE  |.  74 09         je      short 00415309
00415300  |.  FF75 10       push    dword ptr [ebp+10]
00415303  |.  E8 F8620000   call    0041B600
00415308  |.  59            pop     ecx
00415309  |>  B9 40000000   mov     ecx, 40
0041530E  |.  8DBD ECF9FFFF lea     edi, [ebp-614]
00415314  |.  31C0          xor     eax, eax
00415316  |.  8D9D F4FDFFFF lea     ebx, [ebp-20C]
0041531C  |.  F3:AB         rep     stos dword ptr es:>
0041531E  |.  B9 80000000   mov     ecx, 80
00415323  |.  8DBD F4FDFFFF lea     edi, [ebp-20C]
00415329  |.  F3:AB         rep     stos dword ptr es:>
0041532B  |.  FF75 0C       push    dword ptr [ebp+C]
0041532E  |.  FF75 08       push    dword ptr [ebp+8]
00415331  |.  68 C0484200   push    004248C0           ;  ASCII "%s%s"
00415336  |.  53            push    ebx
00415337  |.  E8 A42E0000   call    004181E0           ;  将两部分注册名合起来
0041533C  |.  31C0          xor     eax, eax           ;  ebx=tzltigerisme，eax=C
0041533E  |.  83C4 10       add     esp, 10
00415341  |.  83C9 FF       or      ecx, FFFFFFFF
00415344  |.  8DBD F4FDFFFF lea     edi, [ebp-20C]
0041534A  |.  F2:AE         repne   scas byte ptr es:[>
0041534C  |.  BE FEFFFFFF   mov     esi, -2
00415351  |.  29CE          sub     esi, ecx
00415353  |.  83FE 08       cmp     esi, 8
00415356  |.  7F 22         jg      short 0041537A     ;  两部分合成的位数须大于8，若不大于8则通过下面的计算自动将合成的位数
放大一倍
00415358  |.  8D8D F4FDFFFF lea     ecx, [ebp-20C]
0041535E  |.  FF75 08       push    dword ptr [ebp+8]
00415361  |.  51            push    ecx
00415362  |.  E8 292F0000   call    00418290
00415367  |.  8D85 F4FDFFFF lea     eax, [ebp-20C]
0041536D  |.  59            pop     ecx
0041536E  |.  59            pop     ecx
0041536F  |.  FF75 0C       push    dword ptr [ebp+C]
00415372  |.  50            push    eax
00415373  |.  E8 182F0000   call    00418290
00415378  |.  59            pop     ecx
00415379  |.  59            pop     ecx
0041537A  |>  31C0          xor     eax, eax
0041537C  |.  8DBD F4FDFFFF lea     edi, [ebp-20C]
00415382  |.  83C9 FF       or      ecx, FFFFFFFF
00415385  |.  F2:AE         repne   scas byte ptr es:[>
00415387  |.  C785 F0FDFFFF>mov     dword ptr [ebp-210>
00415391  |.  298D F0FDFFFF sub     [ebp-210], ecx
00415397  |.  83BD F0FDFFFF>cmp     dword ptr [ebp-210>
0041539E  |.  7F 20         jg      short 004153C0
004153A0  |.  6A 00         push    0                  ; /Style = MB\_OK|MB\_APPLMODAL
004153A2  |.  68 C8484200   push    004248C8           ; |Title = "Operation Failed !"
004153A7  |.  68 DC484200   push    004248DC           ; |invalid code.
004153AC  |.  6A 00         push    0                  ; |hOwner = NULL
004153AE  |.  FF15 58254900 call    [<&USER32.MessageB>; \MessageBoxA
004153B4  |.  8D65 F4       lea     esp, [ebp-C]
004153B7  |.  30C0          xor     al, al
004153B9  |.  5F            pop     edi
004153BA  |.  5E            pop     esi
004153BB  |.  5B            pop     ebx
004153BC  |.  5D            pop     ebp
004153BD  |.  C3            retn
004153BE  |   89C0          mov     eax, eax
004153C0  |>  8D95 F4FDFFFF lea     edx, [ebp-20C]     ;  edx=tzltigerisme
004153C6  |.  52            push    edx
004153C7  |.  E8 34620000   call    0041B600           ;  将小写专成大写TZLTIGERISME，记做codeB
004153CC  |.  31F6          xor     esi, esi
004153CE  |.  83BD F0FDFFFF>cmp     dword ptr [ebp-210>
004153D5  |.  59            pop     ecx
004153D6  |.  7E 53         jle     short 0041542B     ;  进入循环计算
004153D8  |>  0FBFDE        /movsx   ebx, si           ;  si=0，1，2……
004153DB  |.  0FBFFE        |movsx   edi, si
004153DE  |.  46            |inc     esi               ;  esi+1
004153DF  |.  89D8          |mov     eax, ebx          ;  eax置0
004153E1  |.  99            |cdq
004153E2  |.  F7BD ECFAFFFF |idiv    dword ptr [ebp-51>
004153E8  |.  0FBE843D F4FD>|movsx   eax, byte ptr [eb>;  codeB逐位送eax
004153F0  |.  89D3          |mov     ebx, edx
004153F2  |.  0FBE8B 894442>|movsx   ecx, byte ptr [eb>;  codeA逐位送ecx
004153F9  |.  0FAFC8        |imul    ecx, eax          ;  codeA与codeB逐位ascii码相乘,结果放在ecx中，记做codeC（25BC，2922，2140……）
004153FC  |.  B8 89888888   |mov     eax, 88888889
00415401  |.  89CA          |mov     edx, ecx          ;  codeC送edx
00415403  |.  89D3          |mov     ebx, edx          ;  codeC送ebx
00415405  |.  F7EA          |imul    edx               ;  edx与eax内的值88888889相乘，结果的余数放edx，记做codeD
00415407  |.  01DA          |add     edx, ebx          ;  codeC+codeD，结果放edx中，记做codeE （1420，15F0，11BB……）
00415409  |.  C1EB 1F       |shr     ebx, 1F           ;  codeC逻辑右移右移1F，ebx置零
0041540C  |.  C1FA 03       |sar     edx, 3            ;  codeE算术右移3，记做codeF，结果在edx中
0041540F  |.  01D3          |add     ebx, edx          ;  codeF+0，结果放ebx中
00415411  |.  6BDB 0F       |imul    ebx, ebx, 0F      ;  codeF*0F，结果分别为25BC，2922，2139，结果放ebx中
00415414  |.  0FBFD6        |movsx   edx, si           ;  si=1，2，3……，送edx
00415417  |.  29D9          |sub     ecx, ebx          ;  逐位运算，即codeC-25BC，codeC-2922，codeC-2139……结果放cl中
00415419  |.  80C1 46       |add     cl, 46            ;  cl+46
0041541C  |.  3B95 F0FDFFFF |cmp     edx, [ebp-210]    ;  比较codeB的位数，小于则继续循环
00415422  |.  888C3D ECF9FF>|mov     [ebp+edi-614], cl ;  cl送ebp+edi-614，ascii码转成字符，即真注册码逐位出现（F，F，M……）
00415429  |.^ 7C AD         \jl      short 004153D8
0041542B  |>  8D85 ECF9FFFF lea     eax, [ebp-614]     ;  真码出现 "FFMORKIIFSMF"
00415431  |.  6A 09         push    9
00415433  |.  FF75 10       push    dword ptr [ebp+10]
00415436  |.  50            push    eax
00415437  |.  E8 C42E0000   call    00418300           ;  这里可以做内存注册机 ,可以进去简单看一下
0041543C  |.  83C4 0C       add     esp, 0C
0041543F  |.  85C0          test    eax, eax
00415441  |.  74 1E         je      short 00415461       关键跳转，也是爆破点
00415443  |.  6A 00         push    0                  ; /Style = MB\_OK|MB\_APPLMODAL
00415445  |.  68 C8484200   push    004248C8           ; |operation failed !
0041544A  |.  68 DC484200   push    004248DC           ; |invalid code.
0041544F  |.  6A 00         push    0                  ; |hOwner = NULL
00415451  |.  FF15 58254900 call    [<&USER32.MessageB>; \MessageBoxA
00415457  |.  8D65 F4       lea     esp, [ebp-C]
0041545A  |.  30C0          xor     al, al
0041545C  |.  5F            pop     edi
0041545D  |.  5E            pop     esi
```
