
# 密码截取(getpassword)2.8破解全面分析 (3千字)

作者 风飘雪

http://fpx.yeah.net

## 1. 黑名单

黑名单：`guodong`、`ttian`、`fpx`

```
* Reference To: MFC42.Ordinal:021D, Ord:021Dh
            |
:00405EEA E837210000              Call 00408026
* Possible StringData Ref from Data Obj ->"guodong"
            |
:00405EEF 6858E74000              push 0040E758
:00405EF4 8D4DB8
lea ecx, dword ptr [ebp-48]
:00405EF7 FF75C0
push [ebp-40]
:00405EFA C645FC02                mov
[ebp-04], 02
* Reference To: MFC42.Ordinal:16E5, Ord:16E5h
            |
:00405EFE E81D210000              Call 00408020
* Possible StringData Ref from Data Obj ->"ttian"
            |
:00405F03 6850E74000              push 0040E750
:00405F08 8D4DB8
lea ecx, dword ptr [ebp-48]
:00405F0B FF75C0
push [ebp-40]
* Reference To: MFC42.Ordinal:16E5, Ord:16E5h
            |
:00405F0E E80D210000              Call 00408020
* Possible StringData Ref from Data Obj ->"fpx"
            |
:00405F13 684CE74000              push 0040E74C
:00405F18 8D4DB8
lea ecx, dword ptr [ebp-48]
:00405F1B FF75C0
push [ebp-40]
```

## 2. 暗桩

注册码最后两位不可以是 `00`。

```
* Reference To: MFC42.Ordinal:1021, Ord:1021h
            |
:00405F6D E8F21E0000              Call 00407E64
:00405F72 8B00
  mov eax, dword ptr [eax]
* Reference To: MSVCRT.\_mbscmp, Ord:0159h
            |
:00405F74 8B3510A44000            mov esi, dword
ptr [0040A410]
* Possible StringData Ref from Data Obj ->"00"
            |
:00405F7A BBA0E24000              mov ebx,
0040E2A0
:00405F7F C645FC04                mov
[ebp-04], 04
:00405F83 53
  push ebx
:00405F84 50
  push eax
:00405F85 FFD6
  call esi
:00405F87 59
  pop ecx
:00405F88 85C0
  test eax, eax
:00405F8A 59
  pop ecx
:00405F8B 7453
  je 00405FE0
:00405F8D 8D45D4
lea eax, dword ptr [ebp-2C]
* Possible Reference to String Resource ID=00001: "Option.ini"
            |
:00405F90 6A01
  push 00000001
:00405F92 50
  push eax
:00405F93 8D4DF0
lea ecx, dword ptr [ebp-10]
```

## 3. 注册算法

`注册码最后2位 xor 注册码前面的=(姓名)ascii码之和`

```
:0040601C 0FBE3408                movsx
esi, byte ptr [eax+ecx]
:00406020 0175DC
add dword ptr [ebp-24], esi
:00406023 40
  inc eax
:00406024 3BC2
  cmp eax, edx
:00406026 7CF4
  jl 0040601C
```

## 4. 关键比较

```
:00406070 E8B11C0000              Call 00407D26
:00406075 33DF
  xor ebx, edi
:00406077 395DDC
cmp dword ptr [ebp-24], ebx
:0040607A 0F855B010000            jne 004061DB
:00406080 6888EB4000              push 0040EB88
```

密码截取(getpassword)2.8 注册名 `fpxfpx` 注册码 `66901`

2001.12.11
