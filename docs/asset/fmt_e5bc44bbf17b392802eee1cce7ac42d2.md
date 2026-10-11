
```
:00456EF2 53                    push ebx
:00456EF3 53                    push ebx
:00456EF4 8945D8                mov dword ptr [ebp-28], eax
:00456EF7 FF5228                call [edx+28]
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:00456F86(U)
```

```
:00456EFA 83FB01                cmp ebx, 00000001
:00456EFD 0F8FB2000000          jg 00456FB5
:00456F03 33FF                  xor edi, edi
:00456F05 EB03                  jmp 00456F0A
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:00456F58(C)
```

```
:00456F07 8B75EC                mov esi, dword ptr [ebp-14]
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:00456F05(U)
```

```
:00456F0A 8B449DE4              mov eax, dword ptr [ebp+4*ebx-1C]
:00456F0E 3DA00F0000            cmp eax, 00000FA0
:00456F13 7205                  jb 00456F1A
:00456F15 B8A00F0000            mov eax, 00000FA0
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:00456F13(C)
```

```
:00456F1A 8B9604090000          mov edx, dword ptr [esi+00000904]
:00456F20 8D8E04090000          lea ecx, dword ptr [esi+00000904]
:00456F26 50                    push eax
:00456F27 8D8530F0FFFF          lea eax, dword ptr [ebp+FFFFF030]
:00456F2D 50                    push eax
:00456F2E FF5234                call [edx+34]
:00456F31 8BF0                  mov esi, eax
:00456F33 85F6                  test esi, esi
:00456F35 7E23                  jle 00456F5A
:00456F37 57                    push edi
:00456F38 8D8D30F0FFFF          lea ecx, dword ptr [ebp+FFFFF030]
:00456F3E 56                    push esi
:00456F3F 51                    push ecx
:00456F40 E8EB270800            call 004D9730
:00456F45 8B4C9DE4              mov ecx, dword ptr [ebp+4*ebx-1C]
:00456F49 8BF8                  mov edi, eax
:00456F4B 2BCE                  sub ecx, esi
:00456F4D 83C40C                add esp, 0000000C
:00456F50 8BC1                  mov eax, ecx
:00456F52 894C9DE4              mov dword ptr [ebp+4*ebx-1C], ecx
:00456F56 85C0                  test eax, eax
:00456F58 7FAD                  jg 00456F07
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:00456F35(C)
```

```
:00456F5A 85DB                  test ebx, ebx
:00456F5C 7516                  jne 00456F74
:00456F5E 8B55EC                mov edx, dword ptr [ebp-14]
:00456F61 6A01                  push 00000001
```

\* Possible Reference to Dialog: `DialogID_0067`, `CONTROL_ID:0008`, "??e 湍(&E)"

```
:00456F63 6A08                  push 00000008
:00456F65 8B8204090000          mov eax, dword ptr [edx+00000904]
:00456F6B 8D8A04090000          lea ecx, dword ptr [edx+00000904]
:00456F71 FF5028                call [eax+28]
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:00456F5C(C)
```

```
:00456F74 3B7C9DD4              cmp edi, dword ptr [ebp+4*ebx-2C]    ;比较校验码
:00456F78 7511                  jne 00456F8B
:00456F7A 8B449DE4              mov eax, dword ptr [ebp+4*ebx-1C]
:00456F7E 85C0                  test eax, eax
:00456F80 7509                  jne 00456F8B
:00456F82 8B75EC                mov esi, dword ptr [ebp-14]
:00456F85 43                    inc ebx
:00456F86 E96FFFFFFF            jmp 00456EFA
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

```
:00456F78(C), :00456F80(C)
```

```
:00456F8B E800000000            call 00456F90
```

\* Referenced by a CALL at Address:

```
:00456F8B
```

```
:00456F90 83042406              add dword ptr [esp], 00000006
:00456F94 C3                    ret
```

在 `456F74` 中记录下两次程序计算的校验码和程序源校验码，修改源程序中预置校验码。至此破解完成。

`4dd466d0e1e0cc123afbe504da2462ca`
