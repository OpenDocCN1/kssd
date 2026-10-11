
# 汇编代码分析

## 主程序逻辑段

```
:004BCB3B 64FF30                 push dword ptr fs:[eax]
:004BCB3E 648920                 mov dword ptr fs:[eax], esp
:004BCB41 C645FF00               mov [ebp-01], 00
:004BCB45 B201                   mov dl, 01
:004BCB47 A180304500             mov eax, dword ptr [00453080]
:004BCB4C E89B66F9FF             call 004531EC
:004BCB51 8945F4                 mov dword ptr [ebp-0C], eax
:004BCB54 33C0                   xor eax, eax
:004BCB56 55                     push ebp
:004BCB57 68DCCB4B00             push 004BCBDC
:004BCB5C 64FF30                 push dword ptr fs:[eax]
:004BCB5F 648920                 mov dword ptr fs:[eax], esp
:004BCB62 BA02000080             mov edx, 80000002
:004BCB67 8B45F4                 mov eax, dword ptr [ebp-0C]
:004BCB6A E85967F9FF             call 004532C8
:004BCB6F B101                   mov cl, 01
```

* Possible StringData Ref from Code Obj ->`"\software\Rockboy\eBookmark"`
  > 取注册信息

```
:004BCB71 BA10CC4B00             mov edx, 004BCC10
:004BCB76 8B45F4                 mov eax, dword ptr [ebp-0C]
:004BCB79 E88E68F9FF             call 0045340C
:004BCB7E 84C0                   test al, al
:004BCB80 7444                   je 004BCBC6
```
  > 跳则 OVER！

```
:004BCB82 8D4DF8                 lea ecx, dword ptr [ebp-08]
```

* Possible StringData Ref from Code Obj ->`"UserName"`

```
:004BCB85 BA34CC4B00             mov edx, 004BCC34
:004BCB8A 8B45F4                 mov eax, dword ptr [ebp-0C]
:004BCB8D E8F26DF9FF             call 00453984
:004BCB92 6A11                   push 00000011
:004BCB94 8D4DE3                 lea ecx, dword ptr [ebp-1D]
```

* Possible StringData Ref from Code Obj ->`"UserData"`

```
:004BCB97 BA48CC4B00             mov edx, 004BCC48
:004BCB9C 8B45F4                 mov eax, dword ptr [ebp-0C]
:004BCB9F E8386FF9FF             call 00453ADC
:004BCBA4 8D55D2                 lea edx, dword ptr [ebp-2E]
:004BCBA7 8B45F8                 mov eax, dword ptr [ebp-08]
```
  > `EAX=fly01` 用户名

```
:004BCBAA E84D86FFFF             call 004B51FC
```
  > 算法 CALL！进入！

```
:004BCBAF 84C0                   test al, al
:004BCBB1 7413                   je 004BCBC6
```
  > 跳则 OVER！

```
:004BCBB3 8D55E3                 lea edx, dword ptr [ebp-1D]
```
  > `EDX=1234567890ABCDEF` 试炼码!

```
:004BCBB6 8D45D2                 lea eax, dword ptr [ebp-2E]
```
  > `EAX=22cM19ZWIWWCDDcZ` 注册码!

```
:004BCBB9 E826C5F4FF             call 004090E4
```
  > 比较 CALL！

```
:004BCBBE 85C0                   test eax, eax
:004BCBC0 7504                   jne 004BCBC6
```
  > 跳则 OVER！

```
:004BCBC2 C645FF01               mov [ebp-01], 01
```
  > 置 1 则 OK！

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses: `:004BCB80(C), :004BCBB1(C), :004BCBC0(C)`

```
:004BCBC6 33C0                   xor eax, eax
```
  > 清 0 则 OVER！

```
:004BCBC8 5A                     pop edx
:004BCBC9 59                     pop ecx
:004BCBCA 59                     pop ecx
:004BCBCB 648910                 mov dword ptr fs:[eax], edx
:004BCBCE 68E3CB4B00             push 004BCBE3
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:004BCBE1(U)`

```
:004BCBD3 8B45F4                 mov eax, dword ptr [ebp-0C]
:004BCBD6 E8C162F4FF             call 00402E9C
:004BCBDB C3                     ret
:004BCBDC E91B6AF4FF             jmp 004035FC
:004BCBE1 EBF0                   jmp 004BCBD3
:004BCBE3 33C0                   xor eax, eax
:004BCBE5 5A                     pop edx
:004BCBE6 59                     pop ecx
:004BCBE7 59                     pop ecx
:004BCBE8 648910                 mov dword ptr fs:[eax], edx
:004BCBEB 6800CC4B00             push 004BCC00
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:004BCBFE(U)`

```
:004BCBF0 8D45F8                 lea eax, dword ptr [ebp-08]
:004BCBF3 E8F06FF4FF             call 00403BE8
:004BCBF8 C3                     ret
:004BCBF9 E9FE69F4FF             jmp 004035FC
:004BCBFE EBF0                   jmp 004BCBF0
:004BCC00 8A45FF                 mov al, byte ptr [ebp-01]
```
  > 注册标志位的值 入 `AL`！爆破点！这次我选这儿完美爆破！

```
:004BCC03 8BE5                   mov esp, ebp
:004BCC05 5D                     pop ebp
:004BCC06 C3                     ret
```

---

## 算法 CALL 分析

进入算法 CALL：`:004BCBAA call 004B51FC`

* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:004B52C6(C)`

```
:004B52CD 8A443EFF               mov al, byte ptr [esi+edi-01]
```
  1. `AL=66` 依次取用户名 `fly01` 字符的 `HEX` 值
  …… ……省 略…… ……共 16 次。用户名不够 16 位则循环取数。

```
:004B52D1 8801                   mov byte ptr [ecx], al
```
  > `[ecx]=AL`

```
:004B52D3 47                     inc edi
```

* Possible StringData Ref from Code Obj ->`"k($j3dAd18L;0gfj"`

```
:004B52D4 B868534B00             mov eax, 004B5368
```
  > `EAX=k($j3dAd18L;0gfj`

```
:004B52D9 8A4418FF               mov al, byte ptr [eax+ebx-01]
```
  1. `AL=6B` 依次取 `k($j3dAd18L;0gfj` 字符的 `HEX` 值
  …… ……省 略…… ……共 16 次

```
:004B52DD 3001                   xor byte ptr [ecx], al
```
  > 依次进行 异或 运算！
  1. `[ecx]=66 XOR 6B=0D`
  2. `[ecx]=6C XOR 28=44` 即：字符 `D`
  3. `[ecx]=79 XOR 24=5D`
  4. `[ecx]=30 XOR 6A=5A` 即：字符 `Z`
  5. `[ecx]=31 XOR 33=02`
  6. `[ecx]=66 XOR 64=02`
  7. `[ecx]=6C XOR 41=2D`
  8. `[ecx]=79 XOR 64=1D`
  9. `[ecx]=30 XOR 31=01`
  10. `[ecx]=31 XOR 38=09`
  11. `[ecx]=66 XOR 4C=2A`
  12. `[ecx]=6C XOR 3B=57` 即：字符 `W`
  13. `[ecx]=79 XOR 30=49` 即：字符 `I`
  14. `[ecx]=30 XOR 67=57` 即：字符 `W`
  15. `[ecx]=31 XOR 66=57` 即：字符 `W`
  16. `[ecx]=66 XOR 6A=0C`

```
:004B52DF 80397A                 cmp byte ptr [ecx], 7A
```
  > 结果 和 `7A` 比较。小于 `7A` 就跳下去。否则就进行下面的运算！

```
:004B52E2 7611                   jbe 004B52F5
:004B52E4 33C0                   xor eax, eax
:004B52E6 8A01                   mov al, byte ptr [ecx]
:004B52E8 51                     push ecx
:004B52E9 B97A000000             mov ecx, 0000007A
:004B52EE 33D2                   xor edx, edx
:004B52F0 F7F1                   div ecx
:004B52F2 59                     pop ecx
:004B52F3 8811                   mov byte ptr [ecx], dl
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:004B52E2(C)`

```
:004B52F5 803930                 cmp byte ptr [ecx], 30
```
  > 结果 和 `30` 比较。大于 `30` 就跳下去。否则就进行下面的运算！

```
:004B52F8 7303                   jnb 004B52FD
:004B52FA 800130                 add byte ptr [ecx], 30
```
