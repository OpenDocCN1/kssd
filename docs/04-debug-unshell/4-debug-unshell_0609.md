
# EncryptPE 保护 Notepad 脱壳

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-637.htm`](https://www.pediy.com/kssd/pediy07/pediy7-637.htm)

`EncryptPE` `20050314` 保护 `Notepad` 脱壳

这个壳久闻大名，没有碰过。把 `Win98` 的 `Notepad` 拿来试试。没用 `SDK`，加壳时只用了缺省选项，省点事。另外，我只用跟了少量代码，所以很多推断可能完全是错的。不对的地方请告诉我 `;-)`

看看加壳后的 `Notepad`，没什么东西。只是用 `exe` 中的数据改写 `V22005314.EPE`，然后 `LoadLibrary`，获取 `EncryptPE_Init` 地址后 `call`。对比一下 `EPE` 主程序和加壳后 `Notepad` 释放出的 `V22005314`，是一样的，即这个文件中没有特定保护程序的信息，`LoadLibrary` 后的初始化动作是固定的。区别在调用 `EncryptPE_Init` 的时候：

```
EPE1:0040D2A4             call    dword ptr [eax] ; GetProcAddress 取
EPE1:0040D2A4                                      ; EncryptPE_Init
EPE1:0040D2A6             cmp     eax, 0
EPE1:0040D2A9             jz      short loc_40D2C3
EPE1:0040D2AB             mov     ebx, eax
EPE1:0040D2AD             mov     eax, ebp
EPE1:0040D2AF             add     eax, 171h
EPE1:0040D2B4             mov     ecx, [eax]
EPE1:0040D2B6             mov     edx, [eax+4]
EPE1:0040D2B9             add     eax, 9
EPE1:0040D2BC             add     eax, edx
EPE1:0040D2BE             add     eax, ecx
EPE1:0040D2C0             push    eax             ; 477ecc
EPE1:0040D2C1             call    ebx
```

`40D2C0` 处 `push` 的参数指向 `Notepad` 的加密数据。

`V22005314.EPE` 是个加壳的 `dll`，用 `ESP` 定律即可轻松脱掉。开始想用脱壳的 `dll` 代替原始文件，关闭 `Notepad` 内重写 `dll` 的代码，这样也许可以调试。后来发现 `dll` 内自校验太多，难以清除干净，只好作罢。不过脱壳的 `V22005314.EPE` 可以拿来反汇编，帮助理解。

程序运行的时候，`OD` 进程被杀掉（开始在 `re-pair` 修改过的 `OD` 下可以跑，后来不知为什么又不行了。不过得到个有用的信息，`dll` 没有使用 `SEH`，可以放心设置硬件断点）。我用 `OD` 的 `LoadDll` 跟了部分初始化代码。调试用 `DriverStudio` `3.2`/`Ice` `Ext` `0.67`，在 `WinXP` 下脱壳。

## 1. Inside `V22005314.EPE`

当 `Notepad` 调用 `LoadLibrary`，这个 `dll` 就全面插手了 `;-)`。如果是初次运行，会安装一个 `WH_CALLWNDPROC` 类型的钩子。

```
EPE0:711AB9E2 loc_711AB9E2:                       ; CODE XREF: EPE0:711AB9D5
EPE0:711AB9E2                 push    offset dwCriticalSection
EPE0:711AB9E7                 call    LeaveCriticalSection_0
EPE0:711AB9EC                 cmp     ds:bFileMappingObjCreated, 0
EPE0:711AB9F3                 jz      loc_711ABA87
EPE0:711AB9F9                 push    0           ; system-wide hook
EPE0:711AB9F9                                         ; 装了个全局钩子
EPE0:711AB9FB                 mov     eax, ds:hInstance ; 71120000,v22005314.epe 的 handle
EPE0:711ABA00                 push    eax
EPE0:711ABA01                 mov     eax, offset f_hook_proc
EPE0:711ABA06                 push    eax
EPE0:711ABA07                 push    4           ; WH_CALLWNDPROC
EPE0:711ABA07                                         ; Installs a hook procedure that monitors
EPE0:711ABA07                                         ; messages before the system sends them to
EPE0:711ABA07                                         ; the destination window procedure
EPE0:711ABA09                 call    SetWindowsHookExA
EPE0:711ABA0E                 mov     ds:hKernelObject, eax
EPE0:711ABA13                 cmp     ds:hKernelObject, 0
```

调用 `SetWindowsHookEx` 的 `dwThreadId` 参数为 `0`，意味着这是个 `system-wide` 钩子，所有的线程都会被拦截。钩子为 `WH_CALLWNDPROC` 类型，即当 `thread` 调用 `SendMessage` 时，安装的 `Hook` 函数将被调用。对于 `GUI` 程序，`SendMessage` 调用无疑是很频繁的。所以，在 `call SetWindowsHookEx` 执行后，`Hook` 函数立即就得到机会执行了。

通过设置钩子，`V22005314.EPE` 被注入到几乎所有的进程。可以运行几个程序试试，用 `LoadPE` 的 `region dump` 可以看到，`dll` 被映射到了 `71120000` 地址，即 `V22005314.EPE` 的预设 `Imagebase`（如果是大程序，可能被重定位到其他地址）。

注意，不是所有的进程。如果程序不调用 `SendMessage`，如果是个 `console` 程序，就可以保持清白 `;-)`。进程隐藏也是用 `Hook` 实现的（对任务管理器网开一面，允许列出进程）。写个 `console` 程序，直接调用 `PSAPI` 中的函数，可以列出隐藏的 `Notepad` 进程，`OpenProcess`，将其 `dump` 出来。

`DLL` 通过查找 `Desktop` 窗口来得到 `Explorer` 的 `PID`，与 `GetCurrentProcessId` 结果比较，如果相同还要检测文件名是否为 `"EXPLORER.EXE"`，以判断是否运行在 `Explorer` 进程内。只有在 `Explorer` 内，才负责对 `Notepad` 解码，创建子进程。即加壳 `Notepad` 的运行牵涉到 3 个进程，`Notepad` 父进程，`Explorer`（真正的父进程，`WaitForDebugEvent` 循环在这里）和 `Notepad` 子进程。由于调试子进程的代码寄生在 `Explorer` 内，用 `OD` 来调试不方便。

对于一般的 `gui` 进程，`dll` 只是简单地 `Hook` 一些敏感 `api`，以达到隐藏进程的目的。进程隐藏是如何实现的？ 这个壳并没有进入 `Ring0`。

```
EPE0:711AE154                 mov     esi, edi
EPE0:711AE156                 push    esi         ; esi=77F75E55 (ntdll.ZwOpenProcess)
EPE0:711AE157                 call    GetCurrentProcess
EPE0:711AE15C                 push    eax         ; hProcess
EPE0:711AE15D                 call    ReadProcessMemory
EPE0:711AE162                 lea     eax, [ebp+NumberOfBytesWritten]
EPE0:711AE165                 push    eax         ; lpflOldProtect
EPE0:711AE166                 push    40h         ; flNewProtect
EPE0:711AE168                 push    5           ; dwSize
EPE0:711AE16A                 push    esi         ; lpAddress
EPE0:711AE16B                 call    GetCurrentProcess
EPE0:711AE170                 push    eax         ; hProcess
EPE0:711AE171                 call    VirtualProtectEx ; 0006F794   FFFFFFFF  |hProcess = FFFFFFFF
EPE0:711AE171                                       ; 0006F798   77F75E55  |Address = ntdll.ZwOpenProcess
EPE0:711AE171                                       ; 0006F79C   00000005  |Size = 5
EPE0:711AE171                                       ; 0006F7A0   00000040  |NewProtect = PAGE_EXECUTE_READWRITE
EPE0:711AE171                                       ; 0006F7A4   0006F7D8  \pOldProtect = 0006F7D8
EPE0:711AE171                                       ;
EPE0:711AE176                 lea     eax, [ebp+NumberOfBytesWritten]
EPE0:711AE179                 push    eax         ; lpNumberOfBytesWritten
EPE0:711AE17A                 push    5           ; nSize
EPE0:711AE17C                 lea     eax, [ebp+Buffer]
EPE0:711AE17F                 push    eax         ; lpBuffer
EPE0:711AE180                 push    esi         ; lpBaseAddress
EPE0:711AE181                 call    GetCurrentProcess
EPE0:711AE186                 push    eax         ; hProcess
EPE0:711AE187                 call    WriteProcessMemory ; 0006F794   FFFFFFFF  |hProcess = FFFFFFFF
EPE0:711AE187                                       ; 0006F798   77F75E55  |Address = 77F75E55
EPE0:711AE187                                       ; 0006F79C   0006F7DF  |Buffer = 0006F7DF
EPE0:711AE187                                       ; 0006F7A0   00000005  |BytesToWrite = 5
EPE0:711AE187                                       ; 0006F7A4   0006F7D8  \pBytesWritten = 0006F7D8
EPE0:711AE187                                       ;
EPE0:711AE187                                       ; 0006F7DF  - E9 6A8323F9     jmp F92A7B4E
EPE0:711AE187                                       ;
EPE0:711AE18C                 test    bl, bl      ; 写入后的代码:
EPE0:711AE18C                                       ;
EPE0:711AE18C                                       ; 77F75E55 -> E9 6A8323F9     jmp V2200531.711AE1C4
EPE0:711AE18C                                       ; 77F75E5A    BA 0003FE7F     mov edx,7FFE0300
EPE0:711AE18C                                       ; 77F75E5F    FFD2            call edx
EPE0:711AE18C                                       ; 77F75E61    C2 1000         retn 10
EPE0:711AE18C                                       ;
EPE0:711AE18C                                       ; 把当前进程中的 NtOpenProcess 替换了
EPE0:711AE18C                                       ; 以后,OpenProcess 不会被拦下
EPE0:711AE18E                 jz      short loc_711AE19A
EPE0:711AE190                 push    offset dwCriticalSection
EPE0:711AE195                 call    LeaveCriticalSection_0
```

这段代码演示了对 `NtOpenProcess` 的 `Hook` 过程。直接用 `GetProcAddress` 取到 `api` 地址，用 `VirtualProtectEx` 修改内存属性后写入 5 `bytes`，跳到壳代码。当然，对于 `console` 程序是不会执行的。

## 2. `oep` 寻找，`dump` 与 `iat` 修复

应该拦截的 `api` 很明显，`CreateProcess`，`WriteProcessMemory`，`SetThreadContext`。过多地与 `dll` 纠缠会使问题复杂化。设置断点时用硬件断点，`bpx` 通不过自校验。



如果想定位感兴趣的代码，可以在 `SoftIce` 这样设断：

```
:bpmb CreateProcessA x
```

创建 `child process` 后 `F12` 到 `V22005314.EPE` 空间。

```
:proc
```

得到刚创建的 `Notepad` 子进程的 `KPEB`，如 `81594DA8`。

```
:addr 81594D18
```

现在可以对感兴趣的地址（如 `IAT`）下断。

要 `dump` `Notepad`，设置以下断点：

```
:bpmb WriteProcessMemory x if *(esp+10)==5 do "eb esp+10 0;x"
```

拦截 `WriteProcessMemory`，当写 `5 bytes` 时为 Hook `api`，改为 `0 bytes`。

```
:bpmb 1B:711E37C5 x do "r edx *(ebp-4);x"
```

在 `edx` 中保留正确 `api` 地址，避开 `IAT` 加密。

```
:bpmb SetThreadContext x if *(*(esp+8) + B8)<500000
```

拦截 `SetThreadContext`，当 `context` 中 `eip<0x500000` 时断下，因为 `Notepad` 的 `oep` 为 `4010CC`，开卷考试;)

当 `SetThreadContext` 断下后，`F12` 到 `V22005314.EPE` 空间：

```
EPE0:711B07B1                    push    eax 
EPE0:711B07B2                    call    SetThreadContext ; 这里 
EPE0:711B07B7 

EPE0:711B07B7 loc_711B07B7:                            ; CODE XREF: EPE0:711AFD30 

EPE0:711B07B7                                        ; EPE0:711AFD65 

EPE0:711B07B7                                        ; EPE0:711AFDAD 

EPE0:711B07B7                    mov     eax, [ebp-0E0h] 

EPE0:711B07BD                    xor     edx, edx 

EPE0:711B07BF                    push    edx 

EPE0:711B07C0                    push    eax 

EPE0:711B07C1                    lea     eax, [ebp-204h] 

EPE0:711B07C7                    call    unknown_libname_49 ; Borland Visual Component Library & Packages 

EPE0:711B07CC                    lea     eax, [ebp-204h] 

EPE0:711B07D2                    push    eax 

EPE0:711B07D3                    lea     ecx, [ebp-208h] 

EPE0:711B07D9                    mov     edx, 1 

EPE0:711B07DE                    mov     eax, 26h 

EPE0:711B07E3                    call    sub_711AC764 

EPE0:711B07E8                    mov     edx, [ebp-208h] 

EPE0:711B07EE                    pop     eax 

EPE0:711B07EF                    call    System::__linkproc__ LStrCat(void) 

EPE0:711B07F4                    mov     edx, [ebp-204h] 

EPE0:711B07FA                    xor     ecx, ecx 

EPE0:711B07FC                    mov     eax, ebx 

EPE0:711B07FE                    call    Classes::TStrings::SetValue(System::AnsiString,System::AnsiString) 

EPE0:711B0803                    jmp     short loc_711B0877 ; default 

    F10 走到 711B0803 处 jmp。

EPE0:711B0877                    push    esi                 ; default 

EPE0:711B0878                    mov     eax, [ebp-0E0h] 

EPE0:711B087E                    push    eax 

EPE0:711B087F                    mov     eax, [ebp-0E4h] 

EPE0:711B0885                    push    eax 

EPE0:711B0886                    call    ContinueDebugEvent 

EPE0:711B088B 

EPE0:711B088B l_wait_for_next_event:                  ; CODE XREF: EPE0:711AFBBF 

EPE0:711B088B                    push    0FFFFFFFFh        ; INFINITE 

EPE0:711B088D                    lea     eax, [ebp-0E8h] 

EPE0:711B0893                    push    eax 

EPE0:711B0894                    call    WaitForDebugEvent 

EPE0:711B0899                    test    eax, eax 

EPE0:711B089B                    jnz     l_debug_event_handler
```

在 `711B0887` 处改 `eip` 为 `711B088B`，跳过对 `ContinueDebugEvent` 的调用。

然后 `F5` 退出 `SoftIce`。现在 `Explorer` 在无限等待 `child process`，`child process` 被挂在 `oep` 处了。

用 `LoadPE` `dump`（已经避开了 `api` hook:)，`ImportRec` 修复 `iat`。没有 `stolen code`，前面 `SetThreadContext` 断下时 `context` 中的 `eip` 就是 `4010CC`。（也许实战会有;)

3. 修复 `replaced code`

反汇编 `dumped_.exe`，可以看到 `replaced code` 有 3 类：

  type 1  对 `call [iat]` 的变形

  `.text:004010D3 FF 15 E4 63 40 00  call    ds:GetCommandLineA`

  被变形为：

  `EPE0:004010D3 90                  nop`

  `EPE0:004010D4 E8 2F 91 5B 00      call    near ptr 9BA208h`

  `        009BA208 FF 25 E4 63 40 00     jmp     [4063E4]`

  type 2. 对 `mov` 指令变形

  `.text:004017AD 89 35 BC 57 40 00  mov     stru_4057B0.hDevNames, esi`

  变为：

  `EPE0:004017AD 90                  nop`

  `EPE0:004017AE E9 E9 93 5D 00      jmp     near ptr 9DAB9Ch`

  `        009DAB9C 89 35 BC 57 40 00     mov     [4057BC],esi`

  type 3. 对 `jmp [iat]` 变形

  `00404FAA FF 25 1C 65 40 00        jmp     ds:__imp_CommDlgExtendedError  `

  变为：

  `EPE0:00404FAA E9 25 F9 5A 00       jmp     near ptr 9B48D4h`

  `      009B48D4 FF 25 1C 65 40 00     jmp     [40651C]`

  这种代码变形有 2 种，原来的 6 字节 `FF25xxxxxx` 可能变为：

  `E9 xx xx xx xx 00` 或

  `90 E9 xx xx xx xx`

对于 `Notepad` 就是这些。我不想费事再去跟代码了，直接写个程序从挂起的子进程读出代码，修复后写回，再次 `dump`，`fix iat`。收工。

本文只是抛砖引玉，捏 `Notepad` 这个软柿子，实战肯定要麻烦得多了。

4dd466d0e1e0cc123afbe504da2462ca
