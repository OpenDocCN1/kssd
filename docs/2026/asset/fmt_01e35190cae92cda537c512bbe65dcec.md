
# [原创]再见病毒一枚

之前论坛上已有发大牛分析了，抱着学习目的练习下![](https://bbs.kanxue.com/view/img/face/50.gif)![](https://bbs.kanxue.com/view/img/face/50.gif)

- 病毒名称：xxmb
- 壳信息：yoda's Protector v1.02 (.dll,.ocx) -> Ashkbiz Danehkar (h)
- 可能受到威胁的系统：windows
- 相关漏洞：无

## 文件系统变化

生成如下文件：

- `C:\DOCUME~1\jack\LOCALS~1\Temp\kb712959.sve`（Kb后面的数值名称是随机生成的）
- `C:\Program Files\Common Files\System\kb712959.dla`（由`kb712959.sve`拷贝得来的）
- `C:\WINDOWS\system32\dsound.dll`
- `C:\WINDOWS\system32\dsound.dll.YUCH`
- `C:\WINDOWS\system32\DllCache\dsound.dll`
- `C:\WINDOWS\system32\DllCache\dsound.dll.YUCH`

## 详细分析／功能介绍

### 1.提升本进程权限，查看`CSOLauncher.exe`、`cstrike-online.exe`连个进程是否存在

首先PEID查壳：发现入口RVA:1000 `.text`段。

但是显示是 yoda's Protector v1.02 (.dll,.ocx) -> Ashkbiz Danehkar 壳，具体的看看，果然是在加壳之后，又改了入口。

```asm
00401000 >/$  B8 D507D0B0   mov eax,0xB0D007D5
00401005  |?  B8 6FA04000   mov eax,server.0040A06F
0040100A  |?  8BC0          mov eax,eax
0040100C  |?  8BD2          mov edx,edx                       ;  ntdll.KiFastSystemCallRet
0040100E  |?  55            push ebp
0040100F  |?  8BE9          mov ebp,ecx
00401011  |.  5D            pop ebp
00401012  |?  50            push eax
00401013  |?  51            push ecx
00401014  |?  8BC8          mov ecx,eax
00401016  |.  59            pop ecx
00401017  |.  C3            retn                              ;  将0040A06F压栈 使用retn指令 返回到40A06F处执行外壳程序
```

说以现在我们可以更改OEP为`A06F`，然后再去看看可不可以脱壳。

可以看到外壳，使用esp定律就可以脱了。

```asm
0040A06F >  60              pushad
0040A070    83EC 38         sub esp,0x38
0040A073    33C0            xor eax,eax
0040A075    C745 D8 4765745>mov dword ptr ss:[ebp-0x28],0x50746547
0040A07C    C745 DC 726F634>mov dword ptr ss:[ebp-0x24],0x41636F72
0040A083    C745 E0 6464726>mov dword ptr ss:[ebp-0x20],0x65726464
0040A08A    C745 E4 7373000>mov dword ptr ss:[ebp-0x1C],0x7373
```

脱壳之后，可以再用PEID查看，是Microsoft Visual C++ 6.0写的。

```asm
004048B4 >/$  55            push ebp
004048B5  |.  8BEC          mov ebp,esp
004048B7  |.  6A FF         push -0x1
004048B9  |.  68 E8504000   push Cracker.004050E8
004048BE  |.  68 20484000   push <jmp.&MSVCRT.\_except\_handler3>        ;  SE 处理程序安装
004048C3  |.  64:A1 0000000>mov eax,dword ptr fs:[0]
004048C9  |.  50            push eax
```

接下来进入正题了：

单步到`0040382F`，`call 00402B29`跟进。

```asm
00402BF8  |.  C645 E9 65    mov byte ptr ss:[ebp-0x17],0x65
00402BFC  |.  C645 EA 67    mov byte ptr ss:[ebp-0x16],0x67
00402C00  |.  C645 EB 65    mov byte ptr ss:[ebp-0x15],0x65          ;  SeDebugPrivileg, cstrike-online.exe, CSOLauncher
00402C04  |.  E8 26E8FFFF   call Cracker.0040142F                    ;  提升本进程权限
00402C09  |.  8D45 F0       lea eax,[local.4]
00402C0C  |.  50            push eax                                 ;  CSOLauncher.exe
00402C0D  |.  E8 73E6FFFF   call Cracker.00401285
00402C12  |.  8D45 C8       lea eax,[local.14]
00402C15  |.  50            push eax                                 ;   "cstrike-online.exe"
00402C16  |.  E8 6AE6FFFF   call Cracker.00401285
```

跟进`40142F`看下。

```asm
0040146F  |.  C645 FA 6C    mov byte ptr ss:[ebp-0x6],0x6C           ; |
00401473  |.  C645 FB 6C    mov byte ptr ss:[ebp-0x5],0x6C           ; |Advapi.dll
00401477  |.  885D FC       mov byte ptr ss:[ebp-0x4],bl             ; |
0040147A  |.  FFD6          call esi                                 ; \LoadLibraryA
00401558  |.  C645 A8 6C    mov byte ptr ss:[ebp-0x58],0x6C
0040155C  |.  C645 A9 65    mov byte ptr ss:[ebp-0x57],0x65
00401560  |.  C645 AA 67    mov byte ptr ss:[ebp-0x56],0x67
00401564  |.  C645 AB 65    mov byte ptr ss:[ebp-0x55],0x65
00401568  |.  8D45 DC       lea eax,[local.9]
0040156B  |.  C645 AC 73    mov byte ptr ss:[ebp-0x54],0x73          ;  OpenProcessToken， LookupPrivilegeVauleA, AdjustTokenPrivilege
0040156F  |.  50            push eax      ;“FuncName”
00401570  |.  57            push edi       ;hModuleDll
00401571  |.  885D AD       mov byte ptr ss:[ebp-0x53],bl
00401574  |.  E8 A7030000   call Cracker.00401920
```

大家可以跟进`401920`看看，使用模块句柄，与函数名查找，导出表地址。

然后在`kernel32.dll`中得到`CloseHandle`、`GetCurrentProcess`两个进程。

`GetCurrentProcess`---->`OpenProcessToken`----->`LookupPrivilegeValueA`---->`AdjustTokenPrivileges`---->`CloseHandle`

```asm
00401610  |.  FFD0          call eax                                 ;  GetCurrentProcess
00401612  |.  50            push eax
00401613  |.  FF55 88       call [local.30]                          ;  OpenProcessToken
00401616  |.  85C0          test eax,eax
00401618  |.  74 5C         je XCracker.00401676
0040161A  |.  8D85 7CFFFFFF lea eax,[local.33]
00401620  |.  50            push eax
00401621  |.  FF75 08       push [arg.1]
00401624  |.  53            push ebx
00401625  |.  FF55 8C       call [local.29]                          ;  LookupprivilegeValueA
00401628  |.  85C0          test eax,eax
0040165F  |.  53            push ebx
00401660  |.  50            push eax
00401661  |.  53            push ebx
00401662  |.  89B5 6CFFFFFF mov [local.37],esi
00401668  |.  FF75 94       push [local.27]
0040166B  |.  FF55 84       call [local.31]                          ;  AdjustTokenPrivileges
```

跟进`call Cracker.00401285`。

调用`LoadLibraryA`加载`kernel32.dll`，然后得到`CreateToolhelp32Snapshot`、`Process32First`、`Process32Next`，然后查找`CSOLauncher.exe`、`cstrike-online.exe`进程，如果找到结束该进程。

对应代码：

```asm
004013C7  |.  FFD3          call ebx                                 ;  CreateToolhelp32Snapshot
004013C9  |.  8BD8          mov ebx,eax
004013CB  |.  EB 03         jmp XCracker.004013D0
004013CD  |>  8B5D 08       mov ebx,[arg.1]
004013D0  |>  83FB FF       cmp ebx,-0x1
004013D3  |.  75 04         jnz XCracker.004013D9
004013D5  |.  33C0          xor eax,eax
004013D7  |.  EB 51         jmp XCracker.0040142A
004013D9  |>  8D85 94FEFFFF lea eax,[local.91]
004013DF  |.  C785 94FEFFFF>mov [local.91],0x128
004013E9  |.  50            push eax
004013EA  |.  53            push ebx
004013EB  |.  FF55 BC       call [local.17]                          ;  Process32First
004013EE  |.  85C0          test eax,eax
004013F0  |.  74 33         je XCracker.00401425
004013F2  |>  8D85 B8FEFFFF /lea eax,[local.82]
004013F8  |.  50            |push eax
004013F9  |.  FF75 08       |push [arg.1]                            ;  进程名
004013FC  |.  E8 CA020000   |call Cracker.004016CB                   ;  相当于strcmp
00401401  |.  59            |pop ecx
00401402  |.  85C0          |test eax,eax
00401404  |.  59            |pop ecx
00401405  |.  75 0C         |jnz XCracker.00401413
00401407  |.  FFB5 9CFEFFFF |push [local.89]                         ;  进程PID
0040140D  |.  E8 8DFDFFFF   |call Cracker.0040119F                   ;  OpenProcess TerminateProcess
00401412  |.  59            |pop ecx                                 ;  ntdll.7C92F641
00401413  |>  8D85 94FEFFFF |lea eax,[local.91]
00401419  |.  50            |push eax
0040141A  |.  53            |push ebx
0040141B  |.  FF55 C0       |call [local.16]                         ;  Process32Next
0040141E  |.  85C0          |test eax,eax
00401420  |.^ 75 D0         \jnz XCracker.004013F2
```

### 2.将资源写入临时文件，拷贝变型文件并设置隐藏属性

将资源写入临时文件`C:\DOCUME~1\jack\LOCALS~1\Temp\kb\*\*\*\*\*\*.sve`（`\*\*\*\*\*\*`是一串随机数），拷贝变型了的临时文件到`C:\Program Files\Common Files\System\kd\*\*\*\*\*\*.dla`，并将文件属性设置为隐藏。

```asm
00403840  |.  8BF8          mov edi,eax
00403842  |.  56            push esi                                 ; /n
00403843  |.  6A 00         push 0x0                                 ; |c = 00
00403845  |.  57            push edi                                 ; |s
00403846  |.  E8 E70F0000   call <jmp.&MSVCRT.memset>                ; \memset
0040384B  |.  6A 00         push 0x0
0040384D  |.  57            push edi
0040384E  |.  6A 06         push 0x6
00403850  |.  E8 F7F3FFFF   call Cracker.00402C4C
```

跟进关键`call 402C4C`。

```asm
00402D35  |.  C645 88 53    mov byte ptr ss:[ebp-0x78],0x53          ;  copyfile
00402D39  |.  AA            stos byte ptr es:[edi]
00402D3A  |.  C645 89 4F    mov byte ptr ss:[ebp-0x77],0x4F
00402D3E  |.  C645 8A 46    mov byte ptr ss:[ebp-0x76],0x46
```

