

```
004272D2    8D0CC9                lea ecx,dword ptr ds:[ecx+ecx*8]       ; ecx=ecx*8+ecx=17BB
004272D5    8D0C4A                lea ecx,dword ptr ds:[edx+ecx*2]       ; ecx=ecx*2+edx=2FC1
004272D8    8D0C8A                lea ecx,dword ptr ds:[edx+ecx*4]       ; ecx=ecx*4+edx=BF4F
004272DB    8D0C4A                lea ecx,dword ptr ds:[edx+ecx*2]       ; ecx=ecx*2+edx=17EE9
004272DE    2BCE                  sub ecx,esi                            ; ecx=ecx-esi=17EE9
004272E0    03C1                  add eax,ecx                            ; ecx=eax+ecx=1A+17EE9=17F03
004272E2    8D0C52                lea ecx,dword ptr ds:[edx+edx*2]       ; ecx=edx*2+edx=E1    (注意：edx=4B ('K'))
004272E5    8D1489                lea edx,dword ptr ds:[ecx+ecx*4]       ; ecx=ecx*4+ecx=465
004272E8    B9 5B000000           mov ecx,5B                             ; ecx=E1
004272ED    33C2                  xor eax,edx                            ; edx=465，eax=17F03
004272EF    33D2                  xor edx,edx                            ; edx=465
004272F1    F7F1                  div ecx                                ; ecx=5B
004272F3    83FA 30               cmp edx,30                             ; edx=1D
004272F6    7C 05                 jl short dumped_1.004272FD
004272F8    83FA 39               cmp edx,39
004272FB    7E 0A                 jle short dumped_1.00427307
004272FD    83FA 41               cmp edx,41                             ; edx=1D
00427300    7C 12                 jl short dumped_1.00427314
00427302    83FA 5A               cmp edx,5A
00427305    7F 0D                 jg short dumped_1.00427314
00427307    52                    push edx
00427308    8D5424 14             lea edx,dword ptr ss:[esp+14]
0042730C    68 98D14600           push dumped_1.0046D198                 ; ASCII "%c"
00427311    52                    push edx
00427312    EB 0B                 jmp short dumped_1.0042731F
00427314    52                    push edx                               ; edx=1D
00427315    8D4424 14             lea eax,dword ptr ss:[esp+14]          ; eax=42B
00427319    68 94D14600           push dumped_1.0046D194                 ; ASCII "%d"
0042731E    50                    push eax                               ; eax=0012D944
0042731F    E8 EA970100           call dumped_1.00440B0E
00427324    8B4C24 1C             mov ecx,dword ptr ss:[esp+1C]          ; ecx=00974798, (ASCII "29")
00427328    83C4 0C               add esp,0C
0042732B    8D5424 18             lea edx,dword ptr ss:[esp+18]          ; esp=0012D928
0042732F    8B41 F8               mov eax,dword ptr ds:[ecx-8]           ; edx=00974799
00427332    8D8C24 94000000       lea ecx,dword ptr ss:[esp+94]
00427339    50                    push eax                               ; eax=00000002
0042733A    55                    push ebp
0042733B    52                    push edx                               ; edx=0012D94C
0042733C    E8 BD920100           call dumped_1.004405FE
00427341    8B00                  mov eax,dword ptr ds:[eax]
00427343    50                    push eax                               ; eax=00974068, (ASCII "98")
00427344    8B4424 14             mov eax,dword ptr ss:[esp+14]          ; 堆栈 ss:[0012D944]=00974798, (ASCII "29")
                                                                         ; eax=00974068, (ASCII "98")
00427348    50                    push eax                               ; eax=00974798, (ASCII "29")
00427349    E8 F1950000           call dumped_1.0043093F
0042734E    83C4 08               add esp,8                              ; esp=0012D92C
00427351    8D4C24 18             lea ecx,dword ptr ss:[esp+18]          ; ecx=00000019
00427355    85C0                  test eax,eax                           ; eax=FFFFFFFF
00427357    0F95C3                setne bl                               ; 条件为真 TRUE，bl=04
0042735A    E8 14EE0100           call dumped_1.00446173
0042735F    84DB                  test bl,bl                             ; bl=01
00427361    0F85 8D000000         jnz dumped_1.004273F4
00427367    8B4C24 10             mov ecx,dword ptr ss:[esp+10]
0042736B    8D7C24 1C             lea edi,dword ptr ss:[esp+1C]
0042736F    8B41 F8               mov eax,dword ptr ds:[ecx-8]
00427372    83C9 FF               or ecx,FFFFFFFF
00427375    03E8                  add ebp,eax
00427377    33C0                  xor eax,eax
00427379    46                    inc esi
0042737A    F2:AE                 repne scas byte ptr es:[edi]
0042737C    F7D1                  not ecx
0042737E    49                    dec ecx
0042737F    3BF1                  cmp esi,ecx
00427381  ^ 0F82 33FFFFFF         jb dumped_1.004272BA
00427387    8D4C24 10             lea ecx,dword ptr ss:[esp+10]
0042738B    C68424 88000000 02    mov byte ptr ss:[esp+88],2
00427393    E8 DBED0100           call dumped_1.00446173
00427398    8D8C24 90000000       lea ecx,dword ptr ss:[esp+90]
0042739F    C68424 88000000 01    mov byte ptr ss:[esp+88],1
004273A7    E8 C7ED0100           call dumped_1.00446173
004273AC    8D8C24 94000000       lea ecx,dword ptr ss:[esp+94]
004273B3    C68424 88000000 00    mov byte ptr ss:[esp+88],0
004273BB    E8 B3ED0100           call dumped_1.00446173
004273C0    8D8C24 98000000       lea ecx,dword ptr ss:[esp+98]
004273C7    C78424 88000000 FFFFF>mov dword ptr ss:[esp+88],-1
004273D2    E8 9CED0100           call dumped_1.00446173
004273D7    B8 01000000           mov eax,1
004273DC    8B8C24 80000000       mov ecx,dword ptr ss:[esp+80]
004273E3    5F                    pop edi
004273E4    5E                    pop esi
004273E5    5D                    pop ebp
004273E6    5B                    pop ebx
004273E7    64:890D 00000000      mov dword ptr fs:[0],ecx
004273EE    83C4 7C               add esp,7C
004273F1    C2 0C00               retn 0C                               ; 返回程序
........
```

---

## 完美注册验证爆破点

```
004272B4    0F84 CD000000         je dumped_1.00427387                  ; je 改 jnz
```

改为：

```
004272B4    0F85 CD000000         jnz dumped_1.00427387
```

---

## 破解总结

本文适合中等Cracker练手，难点主要是在解除程序校验部分，软件调用十一次自校验，往往不注意就Over了，本文主要是才用了暴力破解，不过还是属于比较完美的爆破，呵呵~~至于算法部分，运算太多了，我懒得总结了，上面我已经写得比较清楚了。有兴趣的朋友可以详细看看！

---

版权所有（C）2005 KuNgBiM[DFCG]         Copyright (C) 2005 KuNgBiM[DFCG]  
Cracked By KuNgBiM[DFCG]  
2005-07-23  
12:09:18 PM  

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

