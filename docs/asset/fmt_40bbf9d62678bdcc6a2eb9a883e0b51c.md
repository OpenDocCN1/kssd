
# 汇编代码分析

## 1. 第一组逻辑 (004126BB - 004126D4)

Referenced by a (U)nconditional or (C)onditional Jump at Address: `:004126D4(C)`

```assembly
:004126BB 8B31                  mov esi, dword ptr [ecx]
:004126BD 33D2                  xor edx, edx
:004126BF 8A9070035F00          mov dl, byte ptr [eax+005F0370]  **与码表第 3 组数比对
:004126C5 3BD6                  cmp edx, esi
:004126C7 0F85C6FEFFFF          jne 00412593
:004126CD 40                     inc eax
:004126CE 83C104                add ecx, 00000004
:004126D1 83F806                cmp eax, 00000006
:004126D4 7CE5                  jl 004126BB
```

## 2. 第二组逻辑 (004126D6 - 004126F5)

Referenced by a (U)nconditional or (C)onditional Jump at Address: `:00412704(C)`

```assembly
:004126D6 BE23000000            mov esi, 00000023
:004126DB 8D7C2450              lea edi, dword ptr [esp+50]
:004126DF 8D442408              lea eax, dword ptr [esp+08]
:004126E3 668974240A            mov word ptr [esp+0A], si
:004126E8 50                    push eax
:004126E9 E87297FFFF            call 0040BE60
:004126EE 8B4C2412              mov ecx, dword ptr [esp+12]
:004126F2 83C404                add esp, 00000004
:004126F5 81E1FFFF0000          and ecx, 0000FFFF
```

## 3. 第三组逻辑 (004126FB - 00412704)

```assembly
:004126FB 46                    inc esi
:004126FC 890F                  mov dword ptr [edi], ecx
:004126FE 83C704                add edi, 00000004
:00412701 83FE29                cmp esi, 00000029
:00412704 7CD9                  jl 004126DF
```

## 4. 第四组逻辑 (00412706 - 00412718)

Referenced by a (U)nconditional or (C)onditional Jump at Address: `:00412725(C)`

```assembly
:00412706 33C0                  xor eax, eax
:00412708 8D4C2450              lea ecx, dword ptr [esp+50]
:0041270C 8B31                  mov esi, dword ptr [ecx]
:0041270E 33D2                  xor edx, edx
:00412710 8A9078035F00          mov dl, byte ptr [eax+005F0378]  **与码表第 4 组数比对
:00412716 3BD6                  cmp edx, esi
:00412718 0F8575FEFFFF          jne 00412593
```

## 5. 第五组逻辑 (0041271E - 00412725)

```assembly
:0041271E 40                    inc eax
:0041271F 83C104                add ecx, 00000004
:00412722 83F806                cmp eax, 00000006
:00412725 7CE5                  jl 0041270C
```

## 6. 第六组逻辑 (00412727 - 00412746)

Referenced by a (U)nconditional or (C)onditional Jump at Address: `:00412755(C)`

```assembly
:00412727 BE29000000            mov esi, 00000029
:0041272C 8D7C2450              lea edi, dword ptr [esp+50]
:00412730 8D442408              lea eax, dword ptr [esp+08]
:00412734 668974240A            mov word ptr [esp+0A], si
:00412739 50                    push eax
:0041273A E82197FFFF            call 0040BE60
:0041273F 8B4C2412              mov ecx, dword ptr [esp+12]
:00412743 83C404                add esp, 00000004
:00412746 81E1FFFF0000          and ecx, 0000FFFF
```

## 7. 第七组逻辑 (0041274C - 00412755)

```assembly
:0041274C 46                    inc esi
:0041274D 890F                  mov dword ptr [edi], ecx
:0041274F 83C704                add edi, 00000004
:00412752 83FE30                cmp esi, 00000030
:00412755 7CD9                  jl 00412730
```

## 8. 数据处理与计算 (00412757 - 0041277D)

```assembly
:00412757 A1A8886000            mov eax, dword ptr [006088A8]
:0041275C 8B742464              mov esi, dword ptr [esp+64]    ***[ESP+64]=30
:00412760 8B542468              mov edx, dword ptr [esp+68]    ***[ESP+68]=3A
:00412764 8B4C2458              mov ecx, dword ptr [esp+58]    ***[ESP+58]=14
:00412768 2BC6                  sub eax, esi
:0041276A 03C2                  add eax, edx
:0041276C 03C1                  add eax, ecx
:0041276E 8D4C2450              lea ecx, dword ptr [esp+50]
:00412772 8D1480                lea edx, dword ptr [eax+4*eax]
:00412775 D1E2                  shl edx, 1
:00412777 8915A8886000          mov dword ptr [006088A8], edx
:0041277D 33C0                  xor eax, eax
```

## 9. 第八组逻辑 (0041277F - 00412798)

Referenced by a (U)nconditional or (C)onditional Jump at Address: `:00412798(C)`

```assembly
:0041277F 8B31                  mov esi, dword ptr [ecx]
:00412781 33D2                  xor edx, edx
:00412783 8A9080035F00          mov dl, byte ptr [eax+005F0380]  **与码表第 5 组数比对
:00412789 3BD6                  cmp edx, esi
:0041278B 0F8502FEFFFF          jne 00412593
:00412791 40                    inc eax
:00412792 83C104                add ecx, 00000004
:00412795 83F807                cmp eax, 00000007
:00412798 7CE5                  jl 0041277F
```

## 10. 第九组逻辑 (0041279A - 004127B9)

Referenced by a (U)nconditional or (C)onditional Jump at Address: `:004127C8(C)`

```assembly
:0041279A BE30000000            mov esi, 00000030
:0041279F 8D7C2450              lea edi, dword ptr [esp+50]
:004127A3 8D442408              lea eax, dword ptr [esp+08]
:004127A7 668974240A            mov word ptr [esp+0A], si
:004127AC 50                    push eax
:004127AD E8AE96FFFF            call 0040BE60
:004127B2 8B4C2412              mov ecx, dword ptr [esp+12]
:004127B6 83C404                add esp, 00000004
:004127B9 81E1FFFF0000          and ecx, 0000FFFF
```

## 11. 第十组逻辑 (004127BF - 004127C8)

```assembly
:004127BF 46                    inc esi
:004127C0 890F                  mov dword ptr [edi], ecx
:004127C2 83C704                add edi, 00000004
:004127C5 83FE36                cmp esi, 00000036
:004127C8 7CD9                  jl 004127A3
```

## 12. 第十一组逻辑 (004127CA - 004127DC)

Referenced by a (U)nconditional or (C)onditional Jump at Address: `:004127E9(C)`

```assembly
:004127CA 33C0                  xor eax, eax
:004127CC 8D4C2450              lea ecx, dword ptr [esp+50]
:004127D0 8B31                  mov esi, dword ptr [ecx]
:004127D2 33D2                  xor edx, edx
:004127D4 8A9088035F00          mov dl, byte ptr [eax+005F0388]  **与码表第 6 组数比对
:004127DA 3BD6                  cmp edx, esi
:004127DC 0F85B1FDFFFF          jne 00412593
```

## 13. 第十二组逻辑 (004127E2 - 004127E9)

```assembly
:004127E2 40                    inc eax
:004127E3 83C104                add ecx, 00000004
:004127E6 83F806                cmp eax, 00000006
:004127E9 7CE5                  jl 004127D0
```

## 14. 第十三组逻辑 (004127EB - 0041280A)

Referenced by a (U)nconditional or (C)onditional Jump at Address: `:00412819(C)`

```assembly
:004127EB BE36000000            mov esi, 00000036
:004127F0 8D7C2450              lea edi, dword ptr [esp+50]
:004127F4 8D442408              lea eax, dword ptr [esp+08]
:004127F8 668974240A            mov word ptr [esp+0A], si
:004127FD 50                    push eax
:004127FE E85D96FFFF            call 0040BE60
:00412803 8B4C2412              mov ecx, dword ptr [esp+12]
:00412807 83C404                add esp, 00000004
:0041280A 81E1FFFF0000          and ecx, 0000FFFF
```

## 15. 第十四组逻辑 (00412810 - 00412819)

```assembly
:00412810 46                    inc esi
:00412811 890F                  mov dword ptr [edi], ecx
:00412813 83C704                add edi, 00000004
:00412816 83FE3C                cmp esi, 0000003C
:00412819 7CD9                  jl 004127F4
```

## 16. 数据计算与存储 (0041281B - 00412839)

```assembly
:0041281B 8B542460              mov edx, dword ptr [esp+60]      ***[ESP+60]=15
:0041281F 8B442450              mov eax, dword ptr [esp+50]      ***[ESP+50]=1A
:00412823 03C2                  add eax, edx
:00412825 8D0480                lea eax, dword ptr [eax+4*eax]
:00412828 8D0480                lea eax, dword ptr [eax+4*eax]
:0041282B 8D0C80                lea ecx, dword ptr [eax+4*eax]
:0041282E C1E103                shl ecx, 03
:00412831 890DA4886000          mov dword ptr [006088A4], ecx
:00412837 33C0                  xor eax, eax
:00412839 8D4C2450              lea ecx, dword ptr [esp+50]
```

## 17. 十五组逻辑开始 (0041283D)

Referenced by a (U)nconditional or (C)onditional Jump at Address: `:00412856(C)`

```assembly
:0041283D 8B31                  mov esi, dword ptr [ecx]
```
