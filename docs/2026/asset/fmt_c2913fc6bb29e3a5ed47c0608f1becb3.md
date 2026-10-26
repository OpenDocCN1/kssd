
```asm
00401CC4  |.  E8 E7FCFFFF   call    004019B0
00401CC9  |.  83C4 18       add     esp, 18
00401CCC  |.  837C24 10 00  cmp     dword ptr [esp+10], 0
00401CD1  |.  0F84 A2000000 je      00401D79
00401CD7  |>  8B9424 D80000>mov     edx, dword ptr [esp+D8]
00401CDE  |.  8B3D 24904000 mov     edi, dword ptr [<&KERNEL32.WriteProcessMe>;  kernel32.WriteProcessMemory
00401CE4  |.  8D4424 30     lea     eax, dword ptr [esp+30]
00401CE8  |.  50            push    eax                                       ; /pBytesWritten
00401CE9  |.  8B4424 1C     mov     eax, dword ptr [esp+1C]                   ; |
00401CED  |.  6A 04         push    4                                         ; |BytesToWrite = 4
00401CEF  |.  8D4C24 18     lea     ecx, dword ptr [esp+18]                   ; |
00401CF3  |.  51            push    ecx                                       ; |Buffer
00401CF4  |.  83C2 08       add     edx, 8                                    ; |
00401CF7  |.  52            push    edx                                       ; |Address
00401CF8  |.  50            push    eax                                       ; |hProcess
00401CF9  |.  FFD7          call    edi                                       ; \WriteProcessMemory
00401CFB  |.  8B4C24 14     mov     ecx, dword ptr [esp+14]
00401CFF  |.  8B51 3C       mov     edx, dword ptr [ecx+3C]
00401D02  |.  8B4424 10     mov     eax, dword ptr [esp+10]
00401D06  |.  6A 00         push    0                                         ; /pBytesWritten = NULL
00401D08  |.  53            push    ebx                                       ; |BytesToWrite
00401D09  |.  89442A 34     mov     dword ptr [edx+ebp+34], eax               ; |
00401D0D  |.  8B4C24 18     mov     ecx, dword ptr [esp+18]                   ; |
00401D11  |.  8B5424 20     mov     edx, dword ptr [esp+20]                   ; |
00401D15  |.  55            push    ebp                                       ; |Buffer
00401D16  |.  51            push    ecx                                       ; |Address
00401D17  |.  52            push    edx                                       ; |hProcess
00401D18  |.  FFD7          call    edi                                       ; \WriteProcessMemory
00401D1A  |.  85C0          test    eax, eax
00401D1C  |.  74 52         je      short 00401D70
00401D1E  |.  8B4424 10     mov     eax, dword ptr [esp+10]
00401D22  |.  3B4424 28     cmp     eax, dword ptr [esp+28]
00401D26  |.  C74424 34 070>mov     dword ptr [esp+34], 10007
00401D2E  |.  75 0F         jnz     short 00401D3F
00401D30  |.  8B46 10       mov     eax, dword ptr [esi+10]
00401D33  |.  0346 1C       add     eax, dword ptr [esi+1C]
00401D36  |.  898424 E40000>mov     dword ptr [esp+E4], eax                   ;  计算 EXE入口地址
00401D3D  |.  EB 0C         jmp     short 00401D4B
00401D3F  |>  8B4E 10       mov     ecx, dword ptr [esi+10]
00401D42  |.  03C8          add     ecx, eax
00401D44  |.  898C24 E40000>mov     dword ptr [esp+E4], ecx
00401D4B  |>  8B4424 1C     mov     eax, dword ptr [esp+1C]
00401D4F  |.  8D5424 34     lea     edx, dword ptr [esp+34]
00401D53  |.  52            push    edx                                       ; /pContext
00401D54  |.  50            push    eax                                       ; |hThread
00401D55  |.  FF15 20904000 call    dword ptr [<&KERNEL32.SetThreadContext>]  ; \SetThreadContext
00401D5B  |.  6A 00         push    0                                         ; /hThread = NULL
00401D5D  |.  FF15 1C904000 call    dword ptr [<&KERNEL32.SuspendThread>]     ; \SuspendThread
00401D63  |.  8B4C24 1C     mov     ecx, dword ptr [esp+1C]
00401D67  |.  51            push    ecx                                       ; /hThread
00401D68  |.  FF15 18904000 call    dword ptr [<&KERNEL32.ResumeThread>]      ; \ResumeThread
00401D6E  |.  EB 16         jmp     short 00401D86
00401D70  |>  8B5424 18     mov     edx, dword ptr [esp+18]
00401D74  |.  6A 00         push    0
00401D76  |.  52            push    edx
00401D77  |.  EB 07         jmp     short 00401D80
00401D79  |>  8B4424 18     mov     eax, dword ptr [esp+18]
00401D7D  |.  6A 00         push    0                                         ; /ExitCode = 0
00401D7F  |.  50            push    eax                                       ; |hProcess
00401D80  |>  FF15 14904000 call    dword ptr [<&KERNEL32.TerminateProcess>]  ; \TerminateProcess
00401D86  |>  5F            pop     edi
00401D87  |.  5B            pop     ebx
00401D88  |>  8B8C24 F80200>mov     ecx, dword ptr [esp+2F8]
00401D8F  |.  5E            pop     esi
00401D90  |.  5D            pop     ebp
00401D91  |.  33CC          xor     ecx, esp
00401D93  |.  E8 CF090000   call    00402767
00401D98  |.  81C4 F4020000 add     esp, 2F4
00401D9E  \.  C3            retn
```

## Part 2 : `RSWBSW.exe\K0123456789`

启动`rsRSWBSW.exe`

```asm
004B270F  |.  FF75 CC       push    dword ptr [ebp-34]
004B2712  |.  68 D8274B00   push    004B27D8                                     ;  rs
004B2717  |.  68 E4274B00   push    004B27E4                                     ;  rswbsw.exe
004B271C  |.  8D45 D0       lea     eax, dword ptr [ebp-30]
004B271F  |.  BA 03000000   mov     edx, 3
004B2724  |.  E8 7322F5FF   call    0040499C
004B2729  |.  8B45 D0       mov     eax, dword ptr [ebp-30]
004B272C  |.  E8 AB23F5FF   call    00404ADC
004B2731  |.  50            push    eax                                          ; |FileName
004B2732  |.  68 F0274B00   push    004B27F0                                     ; |open
004B2737  |.  6A 00         push    0                                            ; |hWnd = NULL
004B2739  |.  E8 56FDF7FF   call    <jmp.&shell32.ShellExecuteA>                 ; \ShellExecuteA
004B273E  |.  A1 10494B00   mov     eax, dword ptr [4B4910]
004B2743  |.  8B00          mov     eax, dword ptr [eax]
004B2745  |.  E8 A6DDFBFF   call    004704F0                                     ; 结束自己
004B274A  |.  33C0          xor     eax, eax
004B274C  |.  5A            pop     edx
004B274D  |.  59            pop     ecx
004B274E  |.  59            pop     ecx
004B274F  |.  64:8910       mov     dword ptr fs:[eax], edx
004B2752  |.  68 6C274B00   push    004B276C
004B2757  |>  8D45 C8       lea     eax, dword ptr [ebp-38]
004B275A  |.  BA 0D000000   mov     edx, 0D
004B275F  |.  E8 DC1EF5FF   call    00404640
004B2764  \.  C3            retn
004B2765   .^ E9 8E18F5FF   jmp     00403FF8
004B276A   .^ EB EB         jmp     short 004B2757
004B276C   .  5F            pop     edi
004B276D   .  5E            pop     esi
004B276E   .  8BE5          mov     esp, ebp
004B2770   .  5D            pop     ebp
004B2771   .  C3            retn
```

`结束自己`:

```asm
004704F0  /$  E8 8FDDF9FF   call    0040E284
004704F5  |.  84C0          test    al, al
004704F7  |.  74 07         je      short 00470500
004704F9  |.  6A 00         push    0                                       ; /ExitCode = 0
004704FB  |.  E8 E070F9FF   call    <jmp.&user32.PostQuitMessage>           ; \PostQuitMessage
00470500  \>  C3            retn
```

## Part 3 : `RSWBSW.dat` 解密后是 exe,加壳

`壳入口`:

```asm
00DE9001 >  60              pushad
00DE9002    E8 03000000     call    00DE900A
00DE9007  - E9 EB045D45     jmp     463B94F7
00DE900C    55              push    ebp
00DE900D    C3              retn
00DE900E    E8 01000000     call    00DE9014
00DE9013    EB 5D           jmp     short 00DE9072
00DE9015    BB EDFFFFFF     mov     ebx, -13
00DE901A    03DD            add     ebx, ebp
00DE901C    81EB 00909E00   sub     ebx, 009E9000
00DE9022    83BD 22040000 0>cmp     dword ptr [ebp+422], 0
```
