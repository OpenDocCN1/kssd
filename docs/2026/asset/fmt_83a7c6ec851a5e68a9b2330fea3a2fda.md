
```
100098A2   |.  75 49           jnz short 100098ED

100098A4   |.  68 5C300210     push 1002305C                                 ;  winhlp32.exe

100098A9   |.  56              push esi

100098AA   |.  E8 919E0000     call <比较cmp>

100098AF   |.  59              pop ecx

100098B0   |.  59              pop ecx

100098B1   |.  85C0            test eax, eax

100098B3   |.  74 2E           je short 100098E3

100098B5   |.  57              push edi

100098B6   |.  57              push edi

100098B7   |.  57              push edi

100098B8   |.  68 476A0010     push 10006A47                                 ;  线程4

100098BD   |.  57              push edi

100098BE   |.  57              push edi

100098BF   |.  FF15 5CC70210   call dword ptr [1002C75C]                     ;  kernel32.CreateThread

100098C5   |.  50              push eax

100098C6   |.  FF15 8C100310   call dword ptr [1003108C]                     ;  kernel32.CloseHandle

100098CC   |.  68 F4010000     push 1F4

100098D1   |.  FF15 7CDD0210   call dword ptr [1002DD7C]                     ;  kernel32.Sleep

100098D7   |.  C705 5C170310 0>mov dword ptr [1003175C], 0B

100098E1   |.  EB 0A           jmp short 100098ED

100098E3   |>  E8 038CFFFF     call 100024EB

100098E8   |.  E8 54FDFFFF     call 10009641

100098ED   |>  8B4D FC         mov ecx, [local.1]

100098F0   |.  5F              pop edi

100098F1   |.  5E              pop esi

100098F2   |.  33CD            xor ecx, ebp

100098F4   |.  5B              pop ebx

100098F5   |.  E8 509F0000     call 1001384A

100098FA   |.  C9              leave

100098FB   \.  C3              retn
```

## 关键call 1：动态获取调用API函数

```
100013F8     /$  55              push ebp

100013F9     |.  8BEC            mov ebp, esp

100013FB     |.  83EC 18         sub esp, 18

100013FE     |.  A1 60B00210     mov eax, dword ptr [1002B060]

10001403     |.  33C5            xor eax, ebp

10001405     |.  8945 FC         mov [local.1], eax

10001408     |.  53              push ebx

10001409     |.  56              push esi                                         //动态计算

1000140A     |.  8B35 38200210   mov esi, dword ptr [10022038]                ;  kernel32.LoadLibraryA

10001410     |.  57              push edi

10001411     |.  68 48250210     push 10022548                                ; /TextOutA

10001416     |.  68 40250210     push 10022540                                ; |/GDI32

1000141B     |.  FFD6            call esi                                     ; |\LoadLibraryA

1000141D     |.  8B3D 34200210   mov edi, dword ptr [10022034]                ; |kernel32.GetProcAddress

10001423     |.  50              push eax                                     ; |hModule

10001424     |.  FFD7            call edi                                     ; \GetProcAddress

10001426     |.  68 30250210     push 10022530                                ; /CoInitialize

1000142B     |.  BB 28250210     mov ebx, 10022528                            ; |ole32

10001430     |.  53              push ebx                                     ; |/FileName => "ole32"

10001431     |.  A3 F0140310     mov dword ptr [100314F0], eax                ; ||

10001436     |.  FFD6            call esi                                     ; |\LoadLibraryA

10001438     |.  50              push eax                                     ; |hModule

10001439     |.  FFD7            call edi                                     ; \GetProcAddress

1000143B     |.  68 18250210     push 10022518                                ; /CoUninitialize

10001440     |.  53              push ebx                                     ; |/FileName => "ole32"

10001441     |.  A3 B8110310     mov dword ptr [100311B8], eax                ; ||

10001446     |.  FFD6            call esi                                     ; |\LoadLibraryA

10001448     |.  50              push eax                                     ; |hModule

10001449     |.  FFD7            call edi                                     ; \GetProcAddress

1000144B     |.  68 04250210     push 10022504                                ; /CoCreateInstance

10001450     |.  53              push ebx                                     ; |/FileName => "ole32"

10001451     |.  A3 0C160310     mov dword ptr [1003160C], eax                ; ||

10001456     |.  FFD6            call esi                                     ; |\LoadLibraryA

10001458     |.  50              push eax                                     ; |hModule

10001459     |.  FFD7            call edi                                     ; \GetProcAddress

1000145B     |.  68 F0240210     push 100224F0                                ; /GetForegroundWindowCoCreateInstance

10001460     |.  68 E8240210     push 100224E8                                ; |/user32

10001465     |.  A3 80100310     mov dword ptr [10031080], eax                ; ||

1000146A     |.  FFD6            call esi                                     ; |\LoadLibraryA

1000146C     |.  50              push eax                                     ; |hModule

1000146D     |.  FFD7            call edi                                     ; \GetProcAddress

1000146F     |.  6A 14           push 14

10001471     |.  A3 B0110310     mov dword ptr [100311B0], eax

10001476     |.  8D45 E8         lea eax, [local.6]

10001479     |.  6A 00           push 0

1000147B     |.  50              push eax

1000147C     |.  E8 AF080100     call 10011D30

10001481     |.  68 E0240210     push 100224E0                                ;  rnel3

10001486     |.  8D45 E8         lea eax, [local.6]

10001489     |.  68 D8240210     push 100224D8                                ;  ke%s2

1000148E     |.  50              push eax

1000148F     |.  E8 32230100     call <字符处理函数>

10001494     |.  83C4 18         add esp, 18

10001497     |.  68 C4240210     push 100224C4                                ;  RemoveDirectoryA

1000149C     |.  8D45 E8         lea eax, [local.6]

1000149F     |.  50              push eax                                     ;  //动态调用LoadLibraryA函数

100014A0     |.  FFD6            call esi                                     ;  kernel32.LoadLibraryA

100014A2     |.  50              push eax                                        //保存函数地址

100014A3     |.  FFD7            call edi                                     ;  kernel32.GetProcAddress

100014A5     |.  68 B0240210     push 100224B0                                ;  VirtualProtectEx

100014AA     |.  BB A4240210     mov ebx, 100224A4                            ;  kernel32

100014AF     |.  53              push ebx

100014B0     |.  A3 BC100310     mov dword ptr [100310BC], eax

100014B5     |.  FFD6            call esi                                     ;  kernel32.LoadLibraryA

100014B7     |.  50              push eax

100014B8     |.  FFD7            call edi

100014BA     |.  68 98240210     push 10022498                                ;  WriteFile

100014BF     |.  53              push ebx

100014C0     |.  A3 8C020310     mov dword ptr [1003028C], eax

100014C5     |.  FFD6            call esi

100014C7     |.  50              push eax

100014C8     |.  FFD7            call edi

100014CA     |.  68 8C240210     push 1002248C                                ;  GetFileSizeWriteFile

100014CF     |.  53              push ebx

100014D0     |.  A3 E8C20210     mov dword ptr [1002C2E8], eax

100014D5     |.  FFD6            call esi

100014D7     |.  50              push eax

100014D8     |.  FFD7            call edi

100014DA     |.  68 78240210     push 10022478                                ;  GetShortPathNameA

100014DF     |.  53              push ebx                                     ;  //获取文件短路径

100014E0     |.  A3 E8140310     mov dword ptr [100314E8], eax

100014E5     |.  FFD6            call esi

100014E7     |.  50              push eax

100014E8     |.  FFD7            call edi
```
