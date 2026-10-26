
# 反汇编分析

``` 
0063D503     64:FF30             push dword ptr fs:[eax]
0063D506     64:8920             mov dword ptr fs:[eax],esp
0063D509     8B45 D8             mov eax,dword ptr ss:[ebp-28]            ; 试炼码长度-2计算结果
0063D50C     8B55 D4             mov edx,dword ptr ss:[ebp-2C]            ; 机器码计算结果
0063D50F <>  E8 0C7BDCFF         call bdiary_e.00405020                   ; ->System.@LStrCmp;
0063D514     75 17               jnz short bdiary_e.0063D52D              ; 爆破1
0063D516     8B45 D0             mov eax,dword ptr ss:[ebp-30]            ; 试炼码第1位
0063D519     8B55 CC             mov edx,dword ptr ss:[ebp-34]            ; 试炼码第5位
0063D51C <>  E8 FF7ADCFF         call bdiary_e.00405020                   ; ->System.@LStrCmp;
0063D521     75 0A               jnz short bdiary_e.0063D52D              ; 爆破2
0063D523     A1 B8C16800         mov eax,dword ptr ds:[68C1B8]
0063D528     C600 01             mov byte ptr ds:[eax],1                  ; 置注册成功标志
0063D52B     EB 08               jmp short bdiary_e.0063D535
```

-------------------------------------------------------------------------------------------------------------------

在 `0063CB05     E8 B2D6FEFF         call bdiary_e.0062A1BC` 处跟进:

-------------------------------------------------------------------------------------------------------------------

```
0062A1BC     55                  push ebp
0062A1BD     8BEC                mov ebp,esp
0062A1BF     33C9                xor ecx,ecx
0062A1C1     51                  push ecx
0062A1C2     51                  push ecx
0062A1C3     51                  push ecx
0062A1C4     51                  push ecx
0062A1C5     51                  push ecx
0062A1C6     53                  push ebx
0062A1C7     56                  push esi
0062A1C8     8BF2                mov esi,edx
0062A1CA     8945 FC             mov dword ptr ss:[ebp-4],eax
0062A1CD     8B45 FC             mov eax,dword ptr ss:[ebp-4]
0062A1D0     E8 EFAEDDFF         call bdiary_e.004050C4
0062A1D5     33C0                xor eax,eax
0062A1D7     55                  push ebp
0062A1D8     68 75A26200         push bdiary_e.0062A275
0062A1DD     64:FF30             push dword ptr fs:[eax]
0062A1E0     64:8920             mov dword ptr fs:[eax],esp
0062A1E3     BB 01000000         mov ebx,1                                ; EBX=1
0062A1E8     8D45 F8             lea eax,dword ptr ss:[ebp-8]             ; 常数字符串1
0062A1EB     BA 8CA26200         mov edx,bdiary_e.0062A28C                ; ASCII
"YqhNrd8K1JOsbfZVUatRgB3McDiExTSu5GjvPekL2w7AXWzlIyHmQn94Co6Fp0#S_"
0062A1F0     E8 C7AADDFF         call bdiary_e.00404CBC
0062A1F5     8D45 F4             lea eax,dword ptr ss:[ebp-C]             ; 常数字符串2
0062A1F8     BA D8A26200         mov edx,bdiary_e.0062A2D8                ; ASCII
"_$#abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
0062A1FD     E8 BAAADDFF         call bdiary_e.00404CBC
0062A202     8D45 F0             lea eax,dword ptr ss:[ebp-10]            ; 保存常数字符串1的首字符
0062A205     50                  push eax
0062A206     B9 01000000         mov ecx,1                                ; 1
0062A20B     8BD3                mov edx,ebx                              ; 1
0062A20D     8B45 F8             mov eax,dword ptr ss:[ebp-8]             ; 常数字符串1'YqhNr……'
0062A210     E8 1FAFDDFF         call bdiary_e.00405134                   ; System.@LStrCopy;
0062A215     8D45 F8             lea eax,dword ptr ss:[ebp-8]
0062A218     8B55 F8             mov edx,dword ptr ss:[ebp-8]
0062A21B     E8 C4ACDDFF         call bdiary_e.00404EE4                   ; System.@LStrCat;
0062A220     8D45 F8             lea eax,dword ptr ss:[ebp-8]             ; 常数字符串1自相加
0062A223     50                  push eax
0062A224     B9 41000000         mov ecx,41                               ; 41
0062A229     8BD3                mov edx,ebx                              ; 1
0062A22B     8B45 F8             mov eax,dword ptr ss:[ebp-8]
0062A22E     E8 01AFDDFF         call bdiary_e.00405134                   ; System.@LStrCopy;
0062A233     8D45 EC             lea eax,dword ptr ss:[ebp-14]
0062A236     50                  push eax
0062A237     8B4D F4             mov ecx,dword ptr ss:[ebp-C]             ; 常数字符串2
0062A23A     8B55 F8             mov edx,dword ptr ss:[ebp-8]             ; 常数字符串1
0062A23D     8B45 FC             mov eax,dword ptr ss:[ebp-4]             ; 待编码字符串
0062A240     E8 53020000         call bdiary_e.0062A498                   ; 重要函数，需要分析(字符串编码?)
0062A245     8B55 EC             mov edx,dword ptr ss:[ebp-14]            ; 编码后字符串
0062A248     8D45 F0             lea eax,dword ptr ss:[ebp-10]            ; 常数字符串1的首字符
0062A24B     E8 94ACDDFF         call bdiary_e.00404EE4                   ; System.@LStrCat;
0062A250     8BC6                mov eax,esi
0062A252     8B55 F0             mov edx,dword ptr ss:[ebp-10]            ; 连接结果
0062A255     E8 1EAADDFF         call bdiary_e.00404C78
0062A25A     33C0                xor eax,eax
0062A25C     5A                  pop edx
0062A25D     59                  pop ecx
0062A25E     59                  pop ecx
0062A25F     64:8910             mov dword ptr fs:[eax],edx
0062A262     68 7CA26200         push bdiary_e.0062A27C
0062A267     8D45 EC             lea eax,dword ptr ss:[ebp-14]
0062A26A     BA 05000000         mov edx,5
0062A26F     E8 D4A9DDFF         call bdiary_e.00404C48
0062A274     C3                  retn
0062A275   ^ E9 4EA2DDFF         jmp bdiary_e.004044C8
0062A27A   ^ EB EB               jmp short bdiary_e.0062A267
0062A27C     5E                  pop esi
0062A27D     5B                  pop ebx
0062A27E     8BE5                mov esp,ebp
0062A280     5D                  pop ebp
0062A281     C3                  retn
```

-------------------------------------------------------------------------------------------------------------------

在 `0062A240     E8 53020000         call bdiary_e.0062A498   ; 重要函数，需要分析(字符串编码?)` 处继续跟进：

-------------------------------------------------------------------------------------------------------------------

```
0062A498     55                  push ebp
0062A499     8BEC                mov ebp,esp
0062A49B     83C4 F0             add esp,-10
0062A49E     53                  push ebx
0062A49F     56                  push esi
0062A4A0     57                  push edi
0062A4A1     33DB                xor ebx,ebx
0062A4A3     895D F0             mov dword ptr ss:[ebp-10],ebx
0062A4A6     894D F4             mov dword ptr ss:[ebp-C],ecx             ; 常数字符串2
0062A4A9     8955 F8             mov dword ptr ss:[ebp-8],edx             ; 常数字符串1
0062A4AC     8945 FC             mov dword ptr ss:[ebp-4],eax             ; 待处理字符串,Str
0062A4AF     8B45 FC             mov eax,dword ptr ss:[ebp-4]
0062A4B2     E8 0DACDDFF         call bdiary_e.004050C4
0062A4B7     8B45 F8             mov eax,dword ptr ss:[ebp-8]
0062A4BA     E8 05ACDDFF         call bdiary_e.004050C4
0062A4BF     8B45 F4             mov eax,dword ptr ss:[ebp-C]
0062A4C2     E8 FDABDDFF         call bdiary_e.004050C4
0062A4C7     33C0                xor eax,eax
0062A4C9     55                  push ebp
0062A4CA     68 49A56200         push bdiary_e.0062A549
0062A4CF     64:FF30             push dword ptr fs:[eax]
0062A4D2     64:8920             mov dword ptr fs:[eax],esp
0062A4D5     8B45 08             mov eax,dword ptr ss:[ebp+8]
0062A4D8     8B55 FC             mov edx,dword ptr ss:[ebp-4]             ; 待处理字符串
0062A4DB     E8 98A7DDFF         call bdiary_e.00404C78
0062A4E0     8B45 08             mov eax,dword ptr ss:[ebp+8]
0062A4E3     8B00                mov eax,dword ptr ds:[eax]               ; 同上
0062A4E5     E8 F2A9DDFF         call bdiary_e.00404EDC                   ; Length
0062A4EA     8BF8                mov edi,eax
0062A4EC     85FF                test edi,edi
```
