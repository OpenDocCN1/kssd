
```asm
                004214D4    FF95 C13D4200   call    dword ptr ss:[ebp+423DC1]        ; kernel32.LoadLibraryA
                ; \* 载入"NTDLL.DLL"动态链接库, 我觉得是Bug,应该先GetModuleHandleA才对![](https://www.kanxue.com/images/smilies/smile.gif),有些默认都映射了.
                00421507    8985 FC704000   mov     dword ptr ss:[ebp+4070FC], eax   ; 写入 NTDLL 基地址
                0042153A    83F8 00         cmp     eax, 0                           ; 获得 NTDLL 模块了?
                0042153D    75 46           jnz     short Try.00421585               ; Good...走人
                00421544    C785 00714000 0>mov     dword ptr ss:[ebp+407100], 0     ; 设置失败标记
                00421553    E9 6F020000     jmp     Try.004217C7
                00421585    90              nop
                004215A1    8B33            mov     esi, dword ptr ds:[ebx]          ; Try.00407150
                004215A8    8B7B 04         mov     edi, dword ptr ds:[ebx+4]        ; Try.0040719F
                004215B0    03F5            add     esi, ebp                         ; 指向 dll函数 名表
                004215B7    03FD            add     edi, ebp                         ; 指向 函数地址表
                004215EC    803E 00         cmp     byte ptr ds:[esi], 0
                004215EF    75 58           jnz     short Try.00421649               ; 处理完了吗?
                004215F6    83C3 08         add     ebx, 8
                004215FE    803B 00         cmp     byte ptr ds:[ebx], 0             ; NULL?
                00421601    74 1F           je      short Try.00421622
                0042161A    43              inc     ebx
                00421620    EB DC           jmp     short Try.004215FE
                00421622    90              nop
                0042167B    56              push    esi                              ; 函数名
                0042167C    FFB5 FC704000   push    dword ptr ss:[ebp+4070FC]        ; NTDLL
                00421682    FF95 C53D4200   call    dword ptr ss:[ebp+423DC5]        ; kernel32.GetProcAddress
                0042169F    83F8 00         cmp     eax, 0
                004216A2    75 30           jnz     short Try.004216D4
                004216A9    C785 00714000 0>mov     dword ptr ss:[ebp+407100], 0
                004216CA   /E9 F8000000     jmp     Try.004217C7
                00421701    8907            mov     dword ptr ds:[edi], eax          ; 写入地址
                00421730    83C7 04         add     edi, 4                           ; 下一个地址
                0042173D    803E 00         cmp     byte ptr ds:[esi], 0             ; NULL?
                00421740    74 31           je      short Try.00421773
                00421759    46              inc     esi
                00421771    EB CA           jmp     short Try.0042173D
                0042178A    46              inc     esi                              ; 上边到达NULL,这里跳过NULL
                004217B8    E9 18FEFFFF     jmp     Try.004215D5                     ; 循环处理 -> 004215EC
                ; \* 所有要处理的函数:
                ---------------------------------------------------------------------------
                00421883  52 74 6C 49 6E 69 74 55 6E 69 63 6F 64 65 53 74  RtlInitUnicodeSt
                00421893  72 69 6E 67 00 5A 77 4F 70 65 6E 53 65 63 74 69  ring.ZwOpenSecti
                004218A3  6F 6E 00 5A 77 55 6E 6D 61 70 56 69 65 77 4F 66  on.ZwUnmapViewOf
                004218B3  53 65 63 74 69 6F 6E 00 5A 77 4D 61 70 56 69 65  Section.ZwMapVie
                004218C3  77 4F 66 53 65 63 74 69 6F 6E 00 00 00 00 00 F8  wOfSection.....
                004218D3  41 F9 77 00 00 00 00 00 00 00 00 00 00 00 00 00  Aw.............
                004218E3  00 00 00 52 65 6C 65 61 73 65 4D 75 74 65 78 00  ...ReleaseMutex.
                004218F3  43 72 65 61 74 65 4D 75 74 65 78 41 00 44 65 6C  CreateMutexA.Del
                00421903  65 74 65 46 69 6C 65 41 00 44 65 76 69 63 65 49  eteFileA.DeviceI
                00421913  6F 43 6F 6E 74 72 6F 6C 00 47 65 74 4C 61 73 74  oControl.GetLast
                00421923  45 72 72 6F 72 00 47 65 74 53 79 73 74 65 6D 44  Error.GetSystemD
                00421933  69 72 65 63 74 6F 72 79 41 00 43 72 65 61 74 65  irectoryA.Create
                00421943  46 69 6C 65 41 00 57 72 69 74 65 46 69 6C 65 00  FileA.WriteFile.
                00421953  43 6C 6F 73 65 48 61 6E 64 6C 65 00 00 00 00 00  CloseHandle.....
                00421963  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
                00421973  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
                00421983  00 00 00 00 00 00 00 00 53 74 61 72 74 53 65 72  ........StartSer
                00421993  76 69 63 65 41 00 44 65 6C 65 74 65 53 65 72 76  viceA.DeleteServ
                004219A3  69 63 65 00 4F 70 65 6E 53 43 4D 61 6E 61 67 65  ice.OpenSCManage
                004219B3  72 41 00 43 72 65 61 74 65 53 65 72 76 69 63 65  rA.CreateService
                004219C3  41 00 4F 70 65 6E 53 65 72 76 69 63 65 41 00 43  A.OpenServiceA.C
                004219D3  6F 6E 74 72 6F 6C 53 65 72 76 69 63 65 00 43 6C  ontrolService.Cl
                004219E3  6F 73 65 53 65 72 76 69 63 65 48 61 6E 64 6C 65  oseServiceHandle
                004219F3  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
                00421A03  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
                00421A13  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
                00421A23  00                                               .
                ---------------------------------------------------------------------------
                0042167B    56              push    esi
                0042167C    FFB5 FC704000   push    dword ptr ss:[ebp+4070FC]
                00421682    FF95 C53D4200   call    dword ptr ss:[ebp+423DC5]        ; kernel32.GetProcAddress, 最后一个函数
                0042169F    83F8 00         cmp     eax, 0                           ; 失败?
                004216A2    75 30           jnz     short Try.004216D4
                004216A9    C785 00714000 0>mov     dword ptr ss:[ebp+407100], 0     ; 失败标记
                004216CA   /E9 F8000000     jmp     Try.004217C7
                00421701    8907            mov     dword ptr ds:[edi], eax          ; 写入地址
                00421730    83C7 04         add     edi, 4
                0042173D    803E 00         cmp     byte ptr ds:[esi], 0
                00421740    74 31           je      short Try.00421773
                00421759    46              inc     esi
                00421771  ^\EB CA           jmp     short Try.0042173D               ; 循环到遇到NULL
                0042178A    46              inc     esi                              ; 跳过NULL,指向下一个函数
                ; \* forgot :一大堆取函数地址,不说了
                004217CC    61              popad                                    ; 终于结束了
                004217E4    8B85 00714000   mov     eax, dword ptr ss:[ebp+407100]   ; 设置返回值
                00421801    C3              ret

                }
004205E1    8D85 2E6C4000   lea     eax, dword ptr ss:[ebp+406C2E]               ; 告诉我们要干坏事了;-), 互斥体名称

---------------------------------------------------------------------------
00421361  4C 6F 61 64 69 6E 67 44 42 50 45 00 00 00 00 00  LoadingDBPE.....
00421371  43 64 73 79 73 00 5C 5C 2E 5C 44 62 70 65 44 65  Cdsys.\\.\DbpeDe
00421381  76 69 63 65 30 00 00 00 00 00 00 00 00 00 00 00  vice0...........
00421391  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
004213A1  00 00 00 00 00 00 00 00 00 00                    ..........
---------------------------------------------------------------------------
00420614    50              push    eax
00420615    6A 01           push    1
00420617    6A 00           push    0
```
