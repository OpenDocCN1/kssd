
# 反汇编代码分析

## 1. 模块尾部代码段

```
:10014A64 5E  pop esi
:10014A65 5D  pop ebp
:10014A66 5B  pop ebx
:10014A67 81C4A4010000  add esp, 000001A4
:10014A6D C3  ret
```

> Referenced by a (U)nconditional or (C)onditional Jump at Address:
> `:100149E3(C)`

```
:10014A6E 5F  pop edi
:10014A6F 5E  pop esi
:10014A70 5D  pop ebp
:10014A71 6633C0  xor ax, ax
:10014A74 5B  pop ebx
:10014A75 81C4A4010000  add esp, 000001A4
:10014A7B C3  ret

:10014A7C 90  nop
:10014A7D 90  nop
:10014A7E 90  nop
:10014A7F 90  nop
```

## 2. 主消息循环与导入表调用

> \* Referenced by a CALL at Address:
> `:1000E81E`

```
:10014A80 83EC1C  sub esp, 0000001C
:10014A83 66833D0825031000  cmp word ptr [10032508], 0000
:10014A8B 53  push ebx
:10014A8C 56  push esi
:10014A8D 57  push edi
:10014A8E 745D  je 10014AED
:10014A90 A110250310  mov eax, dword ptr [10032510]
:10014A95 50  push eax
```

> \* Reference To: `USER32.DestroyWindow`, Ord:`008Ah`

```
:10014A96 FF1594710210  Call dword ptr [10027194]
```

> \* Reference To: `USER32.PeekMessageA`, Ord:`01AFh`

```
:10014A9C 8B35E0710210  mov esi, dword ptr [100271E0]
:10014AA2 6A01  push 00000001
:10014AA4 6A00  push 00000000
:10014AA6 6A00  push 00000000
:10014AA8 8D4C2418  lea ecx, dword ptr [esp+18]
:10014AAC 6A00  push 00000000
:10014AAE 51  push ecx
:10014AAF FFD6  call esi
:10014AB1 85C0  test eax, eax
:10014AB3 7459  je 10014B0E
```

> \* Reference To: `USER32.TranslateMessage`, Ord:`0245h`

```
:10014AB5 8B3DE4710210  mov edi, dword ptr [100271E4]
```

> \* Reference To: `USER32.DispatchMessageA`, Ord:`0090h`

```
:10014ABB 8B1DE8710210  mov ebx, dword ptr [100271E8]
```

> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
> `:10014AE0(C)`

```
:10014AC1 8D54240C  lea edx, dword ptr [esp+0C]
:10014AC5 52  push edx
:10014AC6 FFD7  call edi
:10014AC8 8D44240C  lea eax, dword ptr [esp+0C]
:10014ACC 50  push eax
:10014ACD FFD3  call ebx
:10014ACF 6A01  push 00000001
:10014AD1 6A00  push 00000000
:10014AD3 6A00  push 00000000
:10014AD5 8D4C2418  lea ecx, dword ptr [esp+18]
:10014AD9 6A00  push 00000000
:10014ADB 51  push ecx
:10014ADC FFD6  call esi
:10014ADE 85C0  test eax, eax
:10014AE0 75DF  jne 10014AC1
:10014AE2 66B80100  mov ax, 0001
:10014AE6 5F  pop edi
:10014AE7 5E  pop esi
:10014AE8 5B  pop ebx
:10014AE9 83C41C  add esp, 0000001C
:10014AEC C3  ret
```

----------------------------------------------------------------------------------------------------------------------------------

## 3. 脱壳后 `.stxt744` 节代码分析

以下是脱壳过后的 `.stxt744` section 的代码，跳转(`jmp`)切入 `Kernel32.dll` 和 `User32.dll` 要用到这里的代码，这里就是 `IMPORT_TABLE`

重构部分，小弟完全是用手工给重构的，真太笨了。哪个大哥能说说这样的东西该如何用技巧解决？

----------------------------------------------------------------------------------------------------------------------------------

```
:0041B000 4B  dec ebx
:0041B001 0300  add eax, dword ptr [eax]
:0041B003 00DB  add bl, bl
:0041B005 630D0153E800  arpl dword ptr [00E85301], ecx
:0041B00B 000000  BYTE  3 DUP(0)

:0041B00E 870424  xchg dword ptr [esp], eax
:0041B011 9C  pushfd
:0041B012 05F2FFFFFF  add eax, FFFFFFF2
:0041B017 8B18  mov ebx, dword ptr [eax]
:0041B019 6BDB00  imul ebx, 00000000
:0041B01C 035804  add ebx, dword ptr [eax+04]
:0041B01F 9D  popfd
:0041B020 58  pop eax
:0041B021 871C24  xchg dword ptr [esp], ebx
:0041B024 C3  ret
```

> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
> `:0040C488(U)`

```
:0041B025 53  push ebx
:0041B026 E800000000  call 0041B02B
```

> \* Referenced by a CALL at Address:
> `:0041B026`

```
:0041B02B 870424  xchg dword ptr [esp], eax
:0041B02E 9C  pushfd
:0041B02F 05D5FFFFFF  add eax, FFFFFFD5
:0041B034 8B18  mov ebx, dword ptr [eax]
:0041B036 6BDB01  imul ebx, 00000001
:0041B039 035804  add ebx, dword ptr [eax+04]
:0041B03C 9D  popfd
:0041B03D 58  pop eax
:0041B03E 871C24  xchg dword ptr [esp], ebx
:0041B041 C3  ret
```

> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
> `:0040C90A(U)`

```
:0041B042 53  push ebx
:0041B043 E800000000  call 0041B048
```

> \* Referenced by a CALL at Address:
> `:0041B043`

```
:0041B048 870424  xchg dword ptr [esp], eax
:0041B04B 9C  pushfd
:0041B04C 05B8FFFFFF  add eax, FFFFFFB8
:0041B051 8B18  mov ebx, dword ptr [eax]
:0041B053 6BDB02  imul ebx, 00000002
:0041B056 035804  add ebx, dword ptr [eax+04]
:0041B059 9D  popfd
:0041B05A 58  pop eax
:0041B05B 871C24  xchg dword ptr [esp], ebx
:0041B05E C3  ret
```

> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
> `:00409D7D(U)`

```
:0041B05F 53  push ebx
:0041B060 E800000000  call 0041B065
```

> \* Referenced by a CALL at Address:
> `:0041B060`

```
:0041B065 870424  xchg dword ptr [esp], eax
:0041B068 9C  pushfd
:0041B069 059BFFFFFF  add eax, FFFFFF9B
:0041B06E 8B18  mov ebx, dword ptr [eax]
:0041B070 6BDB03  imul ebx, 00000003
:0041B073 035804  add ebx, dword ptr [eax+04]
:0041B076 9D  popfd
:0041B077 58  pop eax
:0041B078 871C24  xchg dword ptr [esp], ebx
:0041B07B C3  ret
```

> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
> `:0040D48A(U)`
