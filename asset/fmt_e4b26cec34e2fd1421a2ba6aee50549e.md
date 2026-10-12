
### 检查 NT 头中相关信息，并做相应的标识：

```
004775CC  .  66:83BD 68E4>cmp     word ptr [ebp-1B98], 3
004775D4  .  75 07        jnz     short 004775DD
004775D6  .  C745 FC 1000>mov     dword ptr [ebp-4], 10
004775DD  |>  81BD 44E4FFF>cmp     dword ptr [ebp-1BBC], 1000
004775E7  .  73 0E        jnb     short 004775F7
004775E9  .  833D D8364D0>cmp     dword ptr [4D36D8], 2  ;检查是否 WinNT 平台
004775F0  .  74 05        je      short 004775F7
004775F2  .  BB 03000000  mov     ebx, 3
```

### 最后关闭文件：

```
004775F7  |>  85FF         test    edi, edi
004775F9  .  74 07        je      short 00477602
004775FB  .  57           push    edi
004775FC  .  E8 D7D80200  call    _fclose
00477601  .  59           pop     ecx
```

### 根据标识 `ebx` 对文件做相应的处理

#### `ebx = 1` 时，弹出错误信息，返回 `-1` 并退出：

```
00477602  |>  83FB 01      cmp     ebx, 1              ;  Switch (cases 1..3)
00477605  .  75 1E        jnz     short 00477625
00477607  .  8D85 E0FEFFF>lea     eax, dword ptr [ebp-120]    ;  Case 1 of switch 00477602
0047760D  .  50           push    eax                 ; /Arg2
0047760E  .  8D96 5D07000>lea     edx, dword ptr [esi+75D]    ; |
00477614  .  52           push    edx                 ; |Arg1
00477615  .  E8 02CAFDFF  call    _Error              ; \\_Error
0047761A  .  83C4 08      add     esp, 8
0047761D  .  83C8 FF      or      eax, FFFFFFFF       ;返回-1
00477620  .  E9 62040000  jmp     00477A87            ;跳转到结束处
```

#### `ebx = 2` 时，弹出选择对话框，根据选择做相应的处理：

```
00477625  |>  83FB 02      cmp     ebx, 2
00477628  .  75 4C        jnz     short 00477676
0047762A  .  8D95 E0FEFFF>lea     edx, dword ptr [ebp-120]    ;  Case 2 of switch 00477602
00477630  .  52           push    edx                 ; /Arg3
00477631  .  8D8E 7E07000>lea     ecx, dword ptr [esi+77E]    ; |
00477637  .  51           push    ecx                 ; |Arg2
00477638  .  8D85 54F8FFF>lea     eax, dword ptr [ebp-7AC]    ; |
0047763E  .  50           push    eax                 ; |Arg1
0047763F  .  E8 E8F50200  call    004A6C2C            ; \\_sprintf
00477644  .  83C4 0C      add     esp, 0C
00477647  .  8D96 CD07000>lea     edx, dword ptr [esi+7CD]
0047764D  .  8D8D 54F8FFF>lea     ecx, dword ptr [ebp-7AC]
00477653  .  A1 7C3B4D00  mov     eax, dword ptr [4D3B7C]
00477658  .  68 24210000  push    2124                ; /Style
0047765D  .  52           push    edx                 ; |Title
0047765E  .  51           push    ecx                 ; |Text
0047765F  .  50           push    eax                 ; |hOwner
00477660  .  E8 B17E0300  call    <jmp.&USER32.MessageBoxA>     ; \\MessageBoxA
00477665  .  83F8 06      cmp     eax, 6    ;检查选择是否为 IDYES
00477668  .  0F84 B400000>je      00477722
0047766E  .  83C8 FF      or      eax, FFFFFFFF  ;选择为 IDNO 时，返回-1
00477671  .  E9 11040000  jmp     00477A87  ;跳转到结束处
```

#### `ebx = 3` 时，弹出选择对话框，根据选择做相应的处理：

```
00477676  |>  83FB 03      cmp     ebx, 3
00477679  .  75 48        jnz     short 004776C3
0047767B  .  8D95 E0FEFFF>lea     edx, dword ptr [ebp-120]    ;  Case 3 of switch 00477602
00477681  .  52           push    edx                 ; /Arg3
00477682  .  8D8E E107000>lea     ecx, dword ptr [esi+7E1]    ; |
00477688  .  51           push    ecx                 ; |Arg2
00477689  .  8D85 54F8FFF>lea     eax, dword ptr [ebp-7AC]    ; |
0047768F  .  50           push    eax                 ; |Arg1
00477690  .  E8 97F50200  call    004A6C2C            ; \\_sprintf
00477695  .  83C4 0C      add     esp, 0C
00477698  .  8D96 2E08000>lea     edx, dword ptr [esi+82E]
0047769E  .  8D8D 54F8FFF>lea     ecx, dword ptr [ebp-7AC]
004776A4  .  A1 7C3B4D00  mov     eax, dword ptr [4D3B7C]
004776A9  .  68 24210000  push    2124                ; /Style
004776AE  .  52           push    edx                 ; |Title
004776AF  .  51           push    ecx                 ; |Text
004776B0  .  50           push    eax                 ; |hOwner
004776B1  .  E8 607E0300  call    <jmp.&USER32.MessageBoxA>     ; \\MessageBoxA
004776B6  .  83F8 06      cmp     eax, 6    ;检查选择是否为 IDYES
004776B9  .  74 67        je      short 00477722
004776BB  .  83C8 FF      or      eax, FFFFFFFF  ;选择为 IDNO 时，返回-1
004776BE  .  E9 C4030000  jmp     00477A87  ;跳转到结束处
```

### 默认处理处：弹出选择对话框，根据选择做相应的处理

```
004776C3  |>  8B55 EC      mov     edx, dword ptr [ebp-14]     ;  Default
004776C6  .  F642 13 20    test    byte ptr [edx+13], 20
004776CA  .  74 56        je      short 00477722
004776CC  .  837D 0C FF    cmp     dword ptr [ebp+C], -1
004776D0  .  74 49        je      short 0047771B
004776D2  .  8D8D E0FEFFF>lea     ecx, dword ptr [ebp-120]
004776D8  .  51           push    ecx                 ; /Arg3
004776D9  .  8D86 4108000>lea     eax, dword ptr [esi+841]    ; |
004776DF  .  50           push    eax                 ; |Arg2
004776E0  .  8D95 54F8FFF>lea     edx, dword ptr [ebp-7AC]    ; |
004776E6  .  52           push    edx                 ; |Arg1
004776E7  .  E8 40F50200  call    004A6C2C            ; \\_sprintf
004776EC  .  83C4 0C      add     esp, 0C
004776EF  .  8D8E 9F08000>lea     ecx, dword ptr [esi+89F]
004776F5  .  8D85 54F8FFF>lea     eax, dword ptr [ebp-7AC]
004776FB  .  8B15 7C3B4D0>mov     edx, dword ptr [4D3B7C]
00477701  .  68 24200000  push    2024                ; /Style
00477706  .  51           push    ecx                 ; |Title
00477707  .  50           push    eax                 ; |Text
00477708  .  52           push    edx                 ; |hOwner
00477709  .  E8 087E0300  call    <jmp.&USER32.MessageBoxA>     ; \\MessageBoxA
0047770E  .  83F8 06      cmp     eax, 6    ;检查选择是否为 IDYES
00477711  .  74 08        je      short 0047771B
00477713  .  83C8 FF      or      eax, FFFFFFFF  ;选择为 IDNO 时，返回-1
00477716  .  E9 6C030000  jmp     00477A87  ;跳转到结束处
```

### 设置加载文件为 DLL 标识：

```
0047771B  |>  C745 F8 0100>mov     dword ptr [ebp-8], 1
```

### 检查是否还有进程被加载调试，将其关闭，并检查返回值，失败则返回 `-1` 并退出：

```
00477722  |>  6A 01        push    1               ; /Arg1 = 00000001
00477724  .  E8 7BE1FFFF  call    004758A4         ; \\Ollydbg.004758A4
00477729  .  59           pop     ecx
0047772A  .  85C0         test    eax, eax
0047772C  .  74 08        je      short 00477736
0047772E  .  83C8 FF      or      eax, FFFFFFFF
00477731  .  E9 51030000  jmp     00477A87  ;跳转到结束处
```

### 清除原调试程序中的所有相关信息，检查返回值，成功则跳过错误处理：

```
00477736  |>  E8 D1DCFFFF  call    0047540C
0047773B  .  85C0         test    eax, eax
0047773D  .  74 15        je      short 00477754
```

### 报错、返回 `-1`，并退出：

```
0047773F  .  8D96 B308000>lea     edx, dword ptr [esi+8B3]
00477745  .  52           push    edx                 ; /Arg1
00477746  .  E8 D1C8FDFF  call    _Error              ; \\_Error
0047774B  .  59           pop     ecx
0047774C  .  83C8 FF      or      eax, FFFFFFFF
0047774F  .  E9 33030000  jmp     00477A87  ;跳转到结束处
```

### 判断打开的是否为 Dll 文件，如果不是则跳过检查文件是否存在：
