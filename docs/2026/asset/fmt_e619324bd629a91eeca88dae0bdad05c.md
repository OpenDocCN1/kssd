
# 反汇编代码片段

## 第一段

```
00405733    |.  A1 8C924800         mov eax,dword ptr ds:[48928C]
00405738    |.  33C4                xor eax,esp
0040573A    |.  894424 18           mov dword ptr ss:[esp+18],eax
0040573E    |.  33C0                xor eax,eax
00405740    |.  53                  push ebx
00405741    |.  8B19                mov ebx,dword ptr ds:[ecx]
00405743    |.  894424 10           mov dword ptr ss:[esp+10],eax
00405747    |.  894424 14           mov dword ptr ss:[esp+14],eax
0040574B    |.  884424 18           mov byte ptr ss:[esp+18],al
0040574F    |.  3943 14             cmp dword ptr ds:[ebx+14],eax
00405752    |.  74 05               je short ****.00405759
00405754    |.  E8 07A00000         call ****.0040F760
00405759    |>  837B 18 FF          cmp dword ptr ds:[ebx+18],-1
0040575D    |.  74 20               je short ****.0040577F
0040575F    |.  8B43 18             mov eax,dword ptr ds:[ebx+18]
00405762    |.  8B4C83 04           mov ecx,dword ptr ds:[ebx+eax*4+4]
00405766    |.  8B11                mov edx,dword ptr ds:[ecx]
00405768    |.  8B52 0C             mov edx,dword ptr ds:[edx+C]
0040576B    |.  8D4424 10           lea eax,dword ptr ss:[esp+10]
0040576F    |.  50                  push eax
00405770    |.  6A 08               push 8
00405772    |.  6A 1C               push 1C
00405774    |.  68 E2850000         push 85E2
00405779    |.  FFD2                call edx
0040577B    |.  85C0                test eax,eax
0040577D    |.  75 15               jnz short ****.00405794
0040577F    |>  B8 01000000         mov eax,1
00405784    |.  5B                  pop ebx
00405785    |.  8B4C24 18           mov ecx,dword ptr ss:[esp+18]
00405789    |.  33CC                xor ecx,esp
0040578B    |.  E8 8ACC0500         call ****.0046241A
00405790    |.  83C4 1C             add esp,1C
00405793    |.  C3                  retn
00405794    |>  33C0                xor eax,eax
00405796    |.  894424 04           mov dword ptr ss:[esp+4],eax
0040579A    |.  894424 08           mov dword ptr ss:[esp+8],eax
0040579E    |.  884424 0C           mov byte ptr ss:[esp+C],al
004057A2    |.  8D4424 04           lea eax,dword ptr ss:[esp+4]
004057A6    |.  50                  push eax
004057A7    |.  B8 70C64800         mov eax,****.0048C670
004057AC    |.  8D4C24 14           lea ecx,dword ptr ss:[esp+14]
004057B0    |.  E8 EBAD0000         call ****.004105A0
004057B5    |.  8B4424 08           mov eax,dword ptr ss:[esp+8]
004057B9    |.  8D88 AC200000       lea ecx,dword ptr ds:[eax+20AC]
004057BF    |.  F7D1                not ecx
004057C1    |.  83C4 04             add esp,4
004057C4    |.  3B4C24 08           cmp ecx,dword ptr ss:[esp+8]
004057C8    |.^ 75 B5               jnz short ****.0040577F                   ;  nop掉
004057CA    |.  8B4C24 1C           mov ecx,dword ptr ss:[esp+1C]
004057CE    |.  5B                  pop ebx
004057CF    |.  8906                mov dword ptr ds:[esi],eax
004057D1    |.  33CC                xor ecx,esp
004057D3    |.  33C0                xor eax,eax
004057D5    |.  E8 40CC0500         call ****.0046241A
004057DA    |.  83C4 1C             add esp,1C
004057DD    \.  C3                  retn
```

## 第二段

```
00402E3B     .  85C0                test eax,eax
00402E3D     .  74 07               je short ****.00402E46                    ;  错误跳转7
00402E3F     .  8B7F 2C             mov edi,dword ptr ds:[edi+2C]
00402E42     .  3BFB                cmp edi,ebx
00402E44     .  EB 61               jmp short ****.00402EA7
00402E46     >  8B4424 14           mov eax,dword ptr ss:[esp+14]
00402E4A     .  8D48 FF             lea ecx,dword ptr ds:[eax-1]
00402E4D     .  81F9 0E0E0000       cmp ecx,0E0E
00402E53     .  77 19               ja short ****.00402E6E                    ;  是否过期判断
00402E55     .  8B7F 28             mov edi,dword ptr ds:[edi+28]
00402E58     .  3BFB                cmp edi,ebx
00402E5A     .  74 3A               je short ****.00402E96
00402E5C     .  50                  push eax
00402E5D     .  FFD7                call edi
00402E5F     .  83C4 04             add esp,4
00402E62     .  5B                  pop ebx
00402E63     .  5F                  pop edi
00402E64     .  5E                  pop esi
00402E65     .  B8 01000000         mov eax,1
00402E6A     .  5D                  pop ebp
00402E6B     .  C2 0800             retn 8
00402E6E     >  3BC3                cmp eax,ebx
00402E70     .  75 24               jnz short ****.00402E96
00402E72     .  8B47 30             mov eax,dword ptr ds:[edi+30]
00402E75     .  3BC3                cmp eax,ebx
00402E77     .  74 02               je short ****.00402E7B
00402E79     .  FFD0                call eax
00402E7B     >  8B17                mov edx,dword ptr ds:[edi]
00402E7D     .  8B42 18             mov eax,dword ptr ds:[edx+18]
00402E80     .  8BCF                mov ecx,edi
00402E82     .  FFD0                call eax
00402E84     .  8B4F 0C             mov ecx,dword ptr ds:[edi+C]
00402E87     .  E8 C4250000         call ****.00405450
00402E8C     .  85C0                test eax,eax
00402E8E     .  75 1B               jnz short ****.00402EAB
00402E90     .  895F 14             mov dword ptr ds:[edi+14],ebx
00402E93     .  895F 10             mov dword ptr ds:[edi+10],ebx
00402E96     >  5B                  pop ebx
00402E97     .  5F                  pop edi
00402E98     .  5E                  pop esi
00402E99     .  B8 01000000         mov eax,1
00402E9E     .  5D                  pop ebp
00402E9F     .  C2 0800             retn 8
00402EA2     >  8B7F 2C             mov edi,dword ptr ds:[edi+2C]
00402EA5     .  85FF                test edi,edi
00402EA7     >  74 02               je short ****.00402EAB
00402EA9     .  FFD7                call edi
00402EAB     >  5B                  pop ebx
00402EAC     .  5F                  pop edi
00402EAD     .  5E                  pop esi
00402EAE     .  33C0                xor eax,eax
00402EB0     .  5D                  pop ebp
00402EB1     .  C2 0800             retn 8
```

总结可以发现软件打狗主要需要针对`RYC_OPEN`,`RYC_READ`函数调用进行爆破，具体爆破需要根据软件流程进行爆破，很难三言两语进行交待，我等菜鸟凑合看吧！

高手飘过。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#软件保护](https://bbs.kanxue.com/forum-4-1-3.htm)
