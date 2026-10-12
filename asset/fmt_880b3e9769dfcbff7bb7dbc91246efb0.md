
```assembly
:00411F4B 81E1FFFF0000             and ecx, 0000FFFF

:00411F51 25FFFF0000               and eax, 0000FFFF

:00411F56 890D98716000             mov dword ptr [00607198], ecx

:00411F5C 8B0DA4716000             mov ecx, dword ptr [006071A4]

:00411F62 83C404                   add esp, 00000004

:00411F65 3BC1                     cmp eax, ecx

:00411F67 A39C716000               mov dword ptr [0060719C], eax

:00411F6C 752E                     jne 00411F9C      ***比较运算结果是否正确，此处要跳！

:00411F6E 5F                       pop edi

:00411F6F 5E                       pop esi

:00411F70 5D                       pop ebp

:00411F71 83C8FF                   or eax, FFFFFFFF

:00411F74 5B                       pop ebx

:00411F75 81C4F8020000             add esp, 000002F8

:00411F7B C3                       ret
```

*****

**Referenced by a CALL at Address:** `:0041155E`

```assembly
:00412210 81EC14030000             sub esp, 00000314

:00412216 53                       push ebx

:00412217 55                       push ebp

:00412218 56                       push esi

:00412219 57                       push edi

:0041221A 6A04                     push 00000004

:0041221C 6800100000               push 00001000

:00412221 68C0450400               push 000445C0

:00412226 6A00                     push 00000000
```

**Reference To:** `KERNEL32.VirtualAlloc`, Ord:`02BBh`

```assembly
:00412228 FF15E4125C00             Call dword ptr [005C12E4]

:0041222E 85C0                     test eax, eax

:00412230 A3E4876000               mov dword ptr [006087E4], eax

:00412235 7522                     jne 00412259

:00412237 50                       push eax
```

**Possible StringData Ref from Data Obj ->** `"致命错误,内存不足!"`

```assembly
:00412238 A10C035F00               mov eax, dword ptr [005F030C]

:0041223D 6894316000               push 00603194

:00412242 50                       push eax

:00412243 6A00                     push 00000000
```

**Reference To:** `USER32.MessageBoxA`, Ord:`01BEh`

```assembly
:00412245 FF15F0165C00             Call dword ptr [005C16F0]

:0041224B 5F                       pop edi

:0041224C 5E                       pop esi

:0041224D 5D                       pop ebp

:0041224E 83C8FF                   or eax, FFFFFFFF

:00412251 5B                       pop ebx

:00412252 81C414030000             add esp, 00000314

:00412258 C3                       ret
```

**Referenced by a (U)nconditional or (C)onditional Jump at Address:** `:00412235(C)`

```assembly
:00412259 68E8886000               push 006088E8

:0041225E 8D4C2460                 lea ecx, dword ptr [esp+60]
```

**Possible StringData Ref from Data Obj ->** `"%s\font\slhz.lib"`

```assembly
:00412262 6868185F00               push 005F1868

:00412267 51                       push ecx

:00412268 E8BE5B1600               call 00577E2B

:0041226D 8D542468                 lea edx, dword ptr [esp+68]
```

**Possible Reference to Dialog:**

```assembly
:00412271 6830185F00               push 005F1830

:00412276 52                       push edx

:00412277 E8D2611600               call 0057844E

:0041227C 83C414                   add esp, 00000014

:0041227F A3E8876000               mov dword ptr [006087E8], eax

:00412284 85C0                     test eax, eax

:00412286 7520                     jne 004122A8

:00412288 50                       push eax
```

**Possible Reference to Dialog:**

```assembly
:00412289 6894316000               push 00603194
```

**Possible StringData Ref from Data Obj ->** `"无法打开矢量字库文件"`

```assembly
:0041228E 6850185F00               push 005F1850

:00412293 50                       push eax
```

**Reference To:** `USER32.MessageBoxA`, Ord:`01BEh`

```assembly
:00412294 FF15F0165C00             Call dword ptr [005C16F0]
```

**Referenced by a (U)nconditional or (C)onditional Jump at Address:** `:0041234A(C)`

```assembly
:0041229A 5F                       pop edi

:0041229B 5E                       pop esi

:0041229C 5D                       pop ebp

:0041229D 83C8FF                   or eax, FFFFFFFF

:004122A0 5B                       pop ebx

:004122A1 81C414030000             add esp, 00000314

:004122A7 C3                       ret
```

**Referenced by a (U)nconditional or (C)onditional Jump at Address:** `:00412286(C)`

```assembly
:004122A8 66A19C716000             mov ax, word ptr [0060719C]

:004122AE 668B0D98716000           mov cx, word ptr [00607198]

:004122B5 8D542414                 lea edx, dword ptr [esp+14]

:004122B9 6689442422               mov word ptr [esp+22], ax

:004122BE 52                       push edx

:004122BF 66894C2422               mov word ptr [esp+22], cx

:004122C4 E8079EFFFF               call 0040C0D0          **随机数 1,2 变换运算结果逆变换

:004122C9 8B442426                 mov eax, dword ptr [esp+26]

:004122CD 8B4C2422                 mov ecx, dword ptr [esp+22]

:004122D1 8B1DE4876000             mov ebx, dword ptr [006087E4]

:004122D7 83C404                   add esp, 00000004

:004122DA 25FFFF0000               and eax, 0000FFFF

:004122DF 81E1FFFF0000             and ecx, 0000FFFF

:004122E5 A39C716000               mov dword ptr [0060719C], eax

:004122EA 890D98716000             mov dword ptr [00607198], ecx

:004122F0 C74424100B000000         mov [esp+10], 0000000B

:004122F8 EB05                     jmp 004122FF
```

**Referenced by a (U)nconditional or (C)onditional Jump at Address:** `:00412362(C)`

```assembly
:004122FA A19C716000               mov eax, dword ptr [0060719C]
```

**Referenced by a (U)nconditional or (C)onditional Jump at Address:** `:004122F8(U)`

```assembly
:004122FF 3B05A4716000             cmp eax, dword ptr [006071A4]

:00412305 755D                     jne 00412364

:00412307 33ED                     xor ebp, ebp
```

**Referenced by a (U)nconditional or (C)onditional Jump at Address:** `:00412354(C)`

```assembly
:00412309 8B15E8876000             mov edx, dword ptr [006087E8]

:0041230F 8D842424010000           lea eax, dword ptr [esp+00000124]

:00412316 52                       push edx

:00412317 6A01                     push 00000001

:00412319 6800020000               push 00000200

:0041231E 50                       push eax

:0041231F E885641600               call 005787A9
```

**Possible Reference to Menu:** `MenuID_0080`

**Possible Reference to String Resource ID=`00128`:** `"eda-嘓"`

```assembly
:00412324 B980000000               mov ecx, 00000080

:00412329 8DB42434010000           lea esi, dword ptr [esp+00000134]

:00412330 8BFB                     mov edi, ebx

:00412332 83C410                   add esp, 00000010

:00412335 F3                       repz

:00412336 A5                       movsd

:00412337 8B0D98716000             mov ecx, dword ptr [00607198]

:0041233D A1A0716000               mov eax, dword ptr [006071A0]

:00412342 81C300020000             add ebx, 00000200

:00412348 3BC8                     cmp ecx, eax

:0041234A 0F854AFFFFFF             jne 0041229A          **比较逆变换结果是否正确，此处不能跳！

:00412350 45                       inc ebp

:00412351 83FD20                   cmp ebp, 00000020

:00412354 7CB3                     jl 00412309

:00412356 8B442410                 mov eax, dword ptr [esp+10]

:0041235A 40                       inc eax

:0041235B 83F81C                   cmp eax, 0000001C

:0041235E 89442410                 mov dword ptr [esp+10], eax

:00412362 7C96                     jl 004122FA
```
