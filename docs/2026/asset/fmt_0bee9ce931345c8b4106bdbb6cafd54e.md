
## 汇编代码片段

```
00467590   |.  8BD5 mov edx,ebp
00467592   |.  E8 7>call unpacked.00407308
00467597   |.  8BD6 mov edx,esi
00467599   |.  8D46>lea eax,dword ptr ds:[esi+18]
0046759C   |.  E8 4>call unpacked.00466DF0           ;  关键进入
004675A1   |.  EB 0>jmp short unpacked.004675B1
004675A3   |>  8BD6 /mov edx,esi
004675A5   |.  8D44>|lea eax,dword ptr ss:[ebp+ebx]
004675A9   |.  E8 4>|call unpacked.00466DF0
004675AE   |.  83C3>|add ebx,40
004675B1   |>  8D43> lea eax,dword ptr ds:[ebx+3F]
004675B4   |.  3BF8 |cmp edi,eax
004675B6   |.^ 77 E>\ja short unpacked.004675A3
004675B8   |.  33C0 xor eax,eax
004675BA   |.  EB 0>jmp short unpacked.004675BE
```

进入`0046759C`   `|.  E8 4>call unpacked.00466DF0` 关键进入 这里是最后关键的 `CALL` 了

```
00466E02   |.  8D74>lea esi,dword ptr ss:[esp+C]
00466E06   |.  8D7C>lea edi,dword ptr ss:[esp+10]
00466E0A   |.  8D6C>lea ebp,dword ptr ss:[esp+14]
00466E0E   |.  8D54>lea edx,dword ptr ss:[esp+18]
00466E12   |.  B9 4>mov ecx,40
00466E17   |.  8B04>mov eax,dword ptr ss:[esp]
00466E1A   |.  E8 5>call unpacked.00466D7C           ;  得到机器码表的地方
00466E1F   |.  8B44>mov eax,dword ptr ss:[esp+4]
00466E23   |.  8B00 mov eax,dword ptr ds:[eax]       ;  得到初始化的的 67452301
00466E25   |.  8903 mov dword ptr ds:[ebx],eax       ;  存储在堆栈第一组
00466E27   |.  8B44>mov eax,dword ptr ss:[esp+4]
00466E2B   |.  8B40>mov eax,dword ptr ds:[eax+4]     ;  固定 EFCDAB89
00466E2E   |.  8906 mov dword ptr ds:[esi],eax       ;  存储在堆栈第2组
00466E30   |.  8B44>mov eax,dword ptr ss:[esp+4]
00466E34   |.  8B40>mov eax,dword ptr ds:[eax+8]     ;  固定 98BADCFE
00466E37   |.  8907 mov dword ptr ds:[edi],eax       ;  存储在堆栈第3组
00466E39   |.  8B44>mov eax,dword ptr ss:[esp+4]
00466E3D   |.  8B40>mov eax,dword ptr ds:[eax+C]     ;  固定 10325476
00466E40   |.  8945>mov dword ptr ss:[ebp],eax       ;  存储在堆栈最后一组
00466E43   |.  8B45>mov eax,dword ptr ss:[ebp]       ;  得到最后一个固定值
00466E46   |.  50   push eax                         ; /压入固定值表第4组数据
00466E47   |.  8B44>mov eax,dword ptr ss:[esp+1C]    ; |
00466E4B   |.  50   push eax                         ; |压入机器码表的第1组数据
00466E4C   |.  6A 0>push 7                           ; |固定值07
00466E4E   |.  68 7>push D76AA478                    ; |固定值  D76AA478
00466E53   |.  8BC3 mov eax,ebx                      ; |1准备修改的地址
00466E55   |.  8B0F mov ecx,dword ptr ds:[edi]       ; |固定值表第3组数据
00466E57   |.  8B16 mov edx,dword ptr ds:[esi]       ; |固定值表第2组数据
00466E59   |.  E8 4>call unpacked.00466CAC           ; \unpacked.00466CAC
00466E5E   |.  8B07 mov eax,dword ptr ds:[edi]       ;  得到第3组数据
00466E60   |.  50   push eax                         ; /固定值表循环（以准备修改开始计算）的第4组
数据
00466E61   |.  8B44>mov eax,dword ptr ss:[esp+20]    ; |
00466E65   |.  50   push eax                         ; |压入机器码表的第2组数据
00466E66   |.  6A 0>push 0C                          ; |固定值0C
00466E68   |.  68 5>push E8C7B756                    ; |固定值E8C7B756
00466E6D   |.  8BC5 mov eax,ebp                      ; |2准备修改的固定值数据地址
00466E6F   |.  8B0E mov ecx,dword ptr ds:[esi]       ; |固定值表循环（以准备修改开始计算）的第3组
数据
00466E71   |.  8B13 mov edx,dword ptr ds:[ebx]       ; |固定值表循环（以准备修改开始计算）的第2组
数据
00466E73   |.  E8 3>call unpacked.00466CAC           ; \unpacked.00466CAC
00466E78   |.  8B06 mov eax,dword ptr ds:[esi]       ;  固定值表循环（以准备修改开始计算）的第4组
数据
00466E7A   |.  50   push eax                         ; /Arg4
00466E7B   |.  8B44>mov eax,dword ptr ss:[esp+24]    ; |
00466E7F   |.  50   push eax                         ; |Arg3
00466E80   |.  6A 1>push 11                          ; |固定值11
00466E82   |.  68 D>push 242070DB                    ; |固定值242070DB
00466E87   |.  8BC7 mov eax,edi                      ; |3准备修改的固定值数据地址
00466E89   |.  8B0B mov ecx,dword ptr ds:[ebx]       ; |
00466E8B   |.  8B55>mov edx,dword ptr ss:[ebp]       ; |
00466E8E   |.  E8 1>call unpacked.00466CAC           ; \unpacked.00466CAC
00466E93   |.  8B03 mov eax,dword ptr ds:[ebx]
00466E95   |.  50   push eax                         ; /Arg4
00466E96   |.  8B44>mov eax,dword ptr ss:[esp+28]    ; |
00466E9A   |.  50   push eax                         ; |Arg3
00466E9B   |.  6A 1>push 16                          ; |Arg2 = 00000016
00466E9D   |.  68 E>push C1BDCEEE                    ; |Arg1 = C1BDCEEE
00466EA2   |.  8BC6 mov eax,esi                      ; |4准备修改的固定值数据地址
00466EA4   |.  8B4D>mov ecx,dword ptr ss:[ebp]       ; |
00466EA7   |.  8B17 mov edx,dword ptr ds:[edi]       ; |
00466EA9   |.  E8 F>call unpacked.00466CAC           ; \unpacked.00466CAC
00466EAE   |.  8B45>mov eax,dword ptr ss:[ebp]
00466EB1   |.  50   push eax                         ; /Arg4
00466EB2   |.  8B44>mov eax,dword ptr ss:[esp+2C]    ; |
00466EB6   |.  50   push eax                         ; |Arg3
00466EB7   |.  6A 0>push 7                           ; |Arg2 = 00000007
00466EB9   |.  68 A>push F57C0FAF                    ; |Arg1 = F57C0FAF
00466EBE   |.  8BC3 mov eax,ebx                      ; |5准备修改的固定值数据地址
00466EC0   |.  8B0F mov ecx,dword ptr ds:[edi]       ; |
00466EC2   |.  8B16 mov edx,dword ptr ds:[esi]       ; |
00466EC4   |.  E8 E>call unpacked.00466CAC           ; \unpacked.00466CAC
00466EC9   |.  8B07 mov eax,dword ptr ds:[edi]
00466ECB   |.  50   push eax                         ; /Arg4
00466ECC   |.  8B44>mov eax,dword ptr ss:[esp+30]    ; |
00466ED0   |.  50   push eax                         ; |Arg3
00466ED1   |.  6A 0>push 0C                          ; |Arg2 = 0000000C
00466ED3   |.  68 2>push 4787C62A                    ; |Arg1 = 4787C62A
00466ED8   |.  8BC5 mov eax,ebp                      ; |6准备修改的固定值数据地址
00466EDA   |.  8B0E mov ecx,dword ptr ds:[esi]       ; |
00466EDC   |.  8B13 mov edx,dword ptr ds:[ebx]       ; |
00466EDE   |.  E8 C>call unpacked.00466CAC           ; \unpacked.00466CAC
00466EE3   |.  8B06 mov eax,dword ptr ds:[esi]
00466EE5   |.  50   push eax                         ; /Arg4
00466EE6   |.  8B44>mov eax,dword ptr ss:[esp+34]    ; |
00466EEA   |.  50   push eax                         ; |Arg3
00466EEB   |.  6A 1>push 11                          ; |Arg2 = 00000011
00466EED   |.  68 1>push A8304613                    ; |Arg1 = A8304613
00466EF2   |.  8BC7 mov eax,edi                      ; |7准备修改的固定值数据地址
00466EF4   |.  8B0B mov ecx,dword ptr ds:[ebx]       ; |
00466EF6   |.  8B55>mov edx,dword ptr ss:[ebp]       ; |
00466EF9   |.  E8 A>call unpacked.00466CAC           ; \unpacked.00466CAC
00466EFE   |.  8B03 mov eax,dword ptr ds:[ebx]
00466F00   |.  50   push eax                         ; /Arg4
00466F01   |.  8B44>mov eax,dword ptr ss:[esp+38]    ; |
00466F05   |.  50   push eax                         ; |Arg3
00466F06   |.  6A 1>push 16                          ; |Arg2 = 00000016
00466F08   |.  68 0>push FD469501                    ; |Arg1 = FD469501
00466F0D   |.  8BC6 mov eax,esi                      ; |8准备修改的固定值数据地址
00466F0F   |.  8B4D>mov ecx,dword ptr ss:[ebp]       ; |
00466F12   |.  8B17 mov edx,dword ptr ds:[edi]       ; |
00466F14   |.  E8 9>call unpacked.00466CAC           ; \unpacked.00466CAC
00466F19   |.  8B45>mov eax,dword ptr ss:[ebp]
00466F1C   |.  50   push eax                         ; /Arg4
00466F1D   |.  8B44>mov eax,dword ptr ss:[esp+3C]    ; |
00466F21   |.  50   push eax                         ; |Arg3
00466F22   |.  6A 0>push 7                           ; |Arg2 = 00000007
00466F24   |.  68 D>push 698098D8                    ; |Arg1 = 698098D8
00466F29   |.  8BC3 mov eax,ebx                      ; |9准备修改的固定值数据地址
00466F2B   |.  8B0F mov ecx,dword ptr ds:[edi]       ; |
00466F2D   |.  8B16 mov edx,dword ptr ds:[esi]       ; |
00466F2F   |.  E8 7>call unpacked.00466CAC           ; \unpacked.00466CAC
00466F34   |.  8B07 mov eax,dword ptr ds:[edi]
00466F36   |.  50   push eax                         ; /Arg4
00466F37   |.  8B44>mov eax,dword ptr ss:[esp+40]    ; |
```
