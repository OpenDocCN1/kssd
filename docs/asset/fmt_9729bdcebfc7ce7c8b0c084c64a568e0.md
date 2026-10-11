
### `Blowfish_Enc(Yl,Yr)` 结果

`Blowfish_Enc(Yl,Yr)` 返回 `Ml` 和 `Mr`。

还需要 `key`。

```
:004015FF 3BC8                cmp ecx, eax         ; 两次比较
:00401601 751C                jne 0040161F
:00401603 6A30                push 00000030
```

### 分析 `BF_Dec` 后的 `BF_Enc` 过程

刚刚分析了 `BF_Dec` 过程，再来分析一个 `BF_Enc` 过程。

其实 `BF_Enc` 过程与 `BF_Dec` 完全一样，只是使用 `P-Box` 顺序倒过来了。

```
:00401070 8B442408            mov eax, dword ptr [esp+08]
:00401074 8B4C240C            mov ecx, dword ptr [esp+0C]
:00401078 53                  push ebx
:00401079 55                  push ebp
:0040107A 8B00                mov eax, dword ptr [eax]
:0040107C 56                  push esi
:0040107D 8B31                mov esi, dword ptr [ecx]
:0040107F 57                  push edi
:00401080 8B7C2414            mov edi, dword ptr [esp+14]
:00401084 C744241410000000    mov [esp+14], 00000010
:0040108C 8BDF                mov ebx, edi
```

*Referenced by a (U)nconditional or (C)onditional Jump at Address:* `004010AC` (C)

```
:0040108E 3303                xor eax, dword ptr [ebx]
:00401090 50                  push eax
:00401091 57                  push edi
:00401092 8BE8                mov ebp, eax
:00401094 E867FFFFFF          call 00401000        ; 函数 F(xl)，参见上面的分析
:00401099 8B4C241C            mov ecx, dword ptr [esp+1C]
:0040109D 83C408              add esp, 00000008
:004010A0 33C6                xor eax, esi
:004010A2 83C304              add ebx, 00000004
:004010A5 49                  dec ecx
:004010A6 8BF5                mov esi, ebp
:004010A8 894C2414            mov dword ptr [esp+14], ecx
:004010AC 75E0                jne 0040108E
:004010AE 8B4F40              mov ecx, dword ptr [edi+40]
:004010B1 8B5744              mov edx, dword ptr [edi+44]
:004010B4 33C8                xor ecx, eax
:004010B6 8B442418            mov eax, dword ptr [esp+18]
:004010BA 33D6                xor edx, esi
:004010BC 5F                  pop edi
:004010BD 8910                mov dword ptr [eax], edx
:004010BF 8B542418            mov edx, dword ptr [esp+18]
:004010C3 5E                  pop esi
:004010C4 5D                  pop ebp
:004010C5 890A                mov dword ptr [edx], ecx
:004010C7 5B                  pop ebx
:004010C8 C3                  ret
```

`BF_Enc` 分析完毕。

### `Init_Key` 过程分析

最后再来一个 `Init_Key` 的过程分析。

```
:00401130 51                  push ecx
:00401131 53                  push ebx
:00401132 55                  push ebp
:00401133 56                  push esi
:00401134 8B742414            mov esi, dword ptr [esp+14]
:00401138 57                  push edi
:00401139 B898614000          mov eax, 00406198
:0040113E 8D4E48              lea ecx, dword ptr [esi+48]
```

*Referenced by a (U)nconditional or (C)onditional Jump at Address:* `00401158` (C)

```
:00401141 BA00010000          mov edx, 00000100
```

*Referenced by a (U)nconditional or (C)onditional Jump at Address:* `00401151` (C)

```
:00401146 8B38                mov edi, dword ptr [eax]      ; S-Box
:00401148 83C004              add eax, 0000004
:0040114B 8939                mov dword ptr [ecx], edi
:0040114D 83C104              add ecx, 0000004
:00401150 4A                  dec edx
:00401151 75F3                jne 00401146
:00401153 3D98714000          cmp eax, 00407198
:00401158 7CE7                jl 00401141
:0040115A 8B6C2420            mov ebp, dword ptr [esp+20]
:0040115E 8B54241C            mov edx, dword ptr [esp+1C]
:00401162 BF50614000          mov edi, 00406150
:00401167 33C0                xor eax, eax
:00401169 2BFE                sub edi, esi
:0040116B C744241012000000    mov [esp+10], 00000012
```

*Referenced by a (U)nconditional or (C)onditional Jump at Address:* `004011AD` (C)

```
:00401173 33C9                xor ecx, ecx
:00401175 C744242004000000    mov [esp+20], 00000004
```

*Referenced by a (U)nconditional or (C)onditional Jump at Address:* `00401197` (C)

```
:0040117D 33DB                xor ebx, ebx
:0040117F 8A1C10              mov bl, byte ptr [eax+edx]
:00401182 C1E108              shl ecx, 08
:00401185 0BCB                or ecx, ebx
:00401187 40                  inc eax
:00401188 3BC5                cmp eax, ebp
:0040118A 7C02                jl 0040118E
:0040118C 33C0                xor eax, eax
```

*Referenced by a (U)nconditional or (C)onditional Jump at Address:* `0040118A` (C)

```
:0040118E 8B5C2420            mov ebx, dword ptr [esp+20]
:00401192 4B                  dec ebx
:00401193 895C2420            mov dword ptr [esp+20], ebx
:00401197 75E4                jne 0040117D
:00401199 8B1C37              mov ebx, dword ptr [edi+esi]
:0040119C 83C604              add esi, 00000004
:0040119F 33D9                xor ebx, ecx
:004011A1 8B4C2410            mov ecx, dword ptr [esp+10]
:004011A5 895EFC              mov dword ptr [esi-04], ebx
:004011A8 49                  dec ecx
:004011A9 894C2410            mov dword ptr [esp+10], ecx
:004011AD 75C4                jne 00401173
:004011AF 8B5C2418            mov ebx, dword ptr [esp+18]
:004011B3 33C0                xor eax, eax
:004011B5 89442420            mov dword ptr [esp+20], eax
:004011B9 8944241C            mov dword ptr [esp+1C], eax
:004011BD 8BF3                mov esi, ebx
:004011BF BF09000000          mov edi, 00000009
```

*Referenced by a (U)nconditional or (C)onditional Jump at Address:* `004011E8` (C)

```
:004011C4 8D44241C            lea eax, dword ptr [esp+1C]
:004011C8 8D4C2420            lea ecx, dword ptr [esp+20]
:004011CC 50                  push eax
:004011CD 51                  push ecx
:004011CE 53                  push ebx
:004011CF E89CFEFFFF          call 00401070               ; BF_Enc(0,0,key)
:004011D4 8B54242C            mov edx, dword ptr [esp+2C]
:004011D8 8B442428            mov eax, dword ptr [esp+28]
:004011DC 8916                mov dword ptr [esi], edx
:004011DE 894604              mov dword ptr [esi+04], eax
:004011E1 83C40C              add esp, 0000000C
:004011E4 83C608              add esi, 00000008
:004011E7 4F                  dec edi
:004011E8 75DA                jne 004011C4
:004011EA 8D734C              lea esi, dword ptr [ebx+4C]
:004011ED BD04000000          mov ebp, 00000004
```

*Referenced by a (U)nconditional or (C)onditional Jump at Address:* `0040121E` (C)

```
:004011F2 BF80000000          mov edi, 00000080
```

*Referenced by a (U)nconditional or (C)onditional Jump at Address:* `0040121B` (C)
