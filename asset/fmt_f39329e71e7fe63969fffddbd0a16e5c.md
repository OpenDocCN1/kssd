
# MD5 and RSA--Magic CHM Merge Home

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-71.htm`](https://www.pediy.com/kssd/pediy09/pediy09-71.htm)

> Magic CHM Merge Home

> UPX 0.89.6 - 1.02 / 1.05 - 1.24 -> Markus & Laszlo

```
0041E9B2    55                  push ebp                  ;OEP
0041E9B3    8BEC               mov ebp,esp
0041E9B5    6A FF              push -1
0041E9B7    68 283F4200        push Magic_CH.00423F28
```

用 `LordPE` 脱壳，`Import.Reconstructor.v1.6.Fanal.Fixed.CHS` 修复输入表。

未注册可以试用 15 次。

`Microsoft Visual C++ 6.0`

重启效验。

安装后，是个英文界面，但是却是国产软件，所以并不准备制作注册机。

本程序不是很大，是由看雪论坛一个 ID 为坚持到底 的发过来的，疯玩了十多天后，上网，发现 OCN 开始整理 ID 了。

以此文激活我在 OCN 的 ID。

其实我大部分时间是待在看雪学院。顺路也贴在这里。

软件使用了 `MD5` 和 `RSA` 算法。

软件启动后，列举进程，如果发现被调试，则杀死调试器，并退出。

以下是杀进程的程序片段：

```
0041834E    90                  nop
0041834F    90                  nop
00418350    6A FF              push -1
00418352    64:A1 00000000     mov eax,dword ptr fs:[0]
00418358    68 D60E4200        push Magic_CH.00420ED6
0041835D    50                  push eax
0041835E    B8 44150000        mov eax,1544
00418363    64:8925 00000000   mov dword ptr fs:[0],esp
0041836A    E8 E1650000        call Magic_CH.0041E950
0041836F    53                  push ebx
00418370    55                  push ebp
00418371    56                  push esi
00418372    57                  push edi
00418373    8D8424 4C030000    lea eax,dword ptr ss:[esp+34C]
0041837A    68 04010000        push 104
0041837F    50                  push eax
00418380    6A 00              push 0
00418382    FF15 1CD15400      call dword ptr ds:[<&kernel32.GetModuleF>; kernel32.GetModuleFileNameA
00418388    6A 00              push 0
0041838A    6A 02              push 2
0041838C    E8 8D680000        call <jmp.&kernel32.CreateToolhelp32Snap>
00418391    8D4C24 1C          lea ecx,dword ptr ss:[esp+1C]
00418395    8BD8               mov ebx,eax
00418397    51                  push ecx
00418398    53                  push ebx
00418399    C74424 24 28010000 mov dword ptr ss:[esp+24],128
004183A1    E8 72680000        call <jmp.&kernel32.Process32First>
004183A6    85C0               test eax,eax
004183A8    0F84 86000000      je Magic_CH.00418434
004183AE    8B3D C0D05400      mov edi,dword ptr ds:[<&kernel32.OpenPro>; kernel32.OpenProcess
004183B4    8B2D 88D75400      mov ebp,dword ptr ds:[<&msvcrt._strcmpi>>; msvcrt._stricmp
004183BA    8B5424 24          mov edx,dword ptr ss:[esp+24]
004183BE    52                  push edx
004183BF    6A 01              push 1
004183C1    68 11040000        push 411
004183C6    FFD7               call edi
004183C8    8BF0               mov esi,eax
004183CA    85F6               test esi,esi
004183CC    74 37              je short Magic_CH.00418405
004183CE    8D4424 14          lea eax,dword ptr ss:[esp+14]
004183D2    8D8C24 54050000    lea ecx,dword ptr ss:[esp+554]
004183D9    50                  push eax
004183DA    68 00100000        push 1000
004183DF    51                  push ecx
004183E0    56                  push esi
004183E1    E8 30070000        call <jmp.&psapi.EnumProcessModules>
004183E6    85C0               test eax,eax
004183E8    74 1B              je short Magic_CH.00418405
004183EA    8B8424 54050000    mov eax,dword ptr ss:[esp+554]
004183F1    8D9424 44010000    lea edx,dword ptr ss:[esp+144]
004183F8    68 04010000        push 104
004183FD    52                  push edx
004183FE    50                  push eax
004183FF    56                  push esi
00418400    E8 0B070000        call <jmp.&psapi.GetModuleFileNameExA>
00418405    56                  push esi
00418406    FF15 28D15400      call dword ptr ds:[<&kernel32.CloseHandl>; kernel32.CloseHandle
0041840C    8D8C24 4C030000    lea ecx,dword ptr ss:[esp+34C]
00418413    8D9424 44010000    lea edx,dword ptr ss:[esp+144]
0041841A    51                  push ecx
0041841B    52                  push edx
0041841C    FFD5               call ebp
0041841E    83C4 08            add esp,8
00418421    85C0               test eax,eax
00418423    74 2A              je short Magic_CH.0041844F
00418425    8D4424 1C          lea eax,dword ptr ss:[esp+1C]
00418429    50                  push eax
0041842A    53                  push ebx
0041842B    E8 E2670000        call <jmp.&kernel32.Process32Next>
00418430    85C0               test eax,eax
00418432  ^ 75 86              jnz short Magic_CH.004183BA
00418434    33C0               xor eax,eax
00418436    8B8C24 54150000    mov ecx,dword ptr ss:[esp+1554]
0041843D    5F                  pop edi
0041843E    5E                  pop esi
0041843F    5D                  pop ebp
00418440    5B                  pop ebx
00418441    64:890D 00000000   mov dword ptr fs:[0],ecx
00418448    81C4 50150000      add esp,1550
0041844E    C3                  retn
0041844F    8B4C24 34          mov ecx,dword ptr ss:[esp+34]
00418453    51                  push ecx
00418454    6A 01              push 1
00418456    68 11040000        push 411
0041845B    FFD7               call edi
0041845D    8BF0               mov esi,eax
0041845F    85F6               test esi,esi
00418461    0F84 B9000000      je Magic_CH.00418520
00418467    8D5424 18          lea edx,dword ptr ss:[esp+18]
0041846B    8D8424 54050000    lea eax,dword ptr ss:[esp+554]
00418472    52                  push edx
00418473    68 00100000        push 1000
00418478    50                  push eax
00418479    56                  push esi
0041847A    E8 97060000        call <jmp.&psapi.EnumProcessModules>
0041847F    85C0               test eax,eax
00418481    0F84 99000000      je Magic_CH.00418520
00418487    8B9424 54050000    mov edx,dword ptr ss:[esp+554]
0041848E    8D8C24 48020000    lea ecx,dword ptr ss:[esp+248]
00418495    68 04010000        push 104
0041849A    51                  push ecx
0041849B    52                  push edx
0041849C    56                  push esi
0041849D    E8 6E060000        call <jmp.&psapi.GetModuleFileNameExA>
004184A2    85C0               test eax,eax
004184A4    74 7A              je short Magic_CH.00418520
004184A6    8D8424 50040000    lea eax,dword ptr ss:[esp+450]
004184AD    68 04010000        push 104
004184B2    50                  push eax
004184B3    FF15 E0D05400      call dword ptr ds:[<&kernel32.GetWindows>; kernel32.GetWindowsDirectoryA
004184B9    8D8C24 50040000    lea ecx,dword ptr ss:[esp+450]
004184C0    51                  push ecx
004184C1    8D4C24 18          lea ecx,dword ptr ss:[esp+18]
004184C5    E8 2E5F0000        call <jmp.&mfc42.#537>
004184CA    68 14C44200        push Magic_CH.0042C414                   ; ASCII "\\explorer.exe"
004184CF    8D5424 14          lea edx,dword ptr ss:[esp+14]
004184D3    50                  push eax
004184D4    52                  push edx
004184D5    C78424 68150000 00>mov dword ptr ss:[esp+1568],0
004184E0    E8 F9600000        call <jmp.&mfc42.#924>
004184E5    8D4C24 14          lea ecx,dword ptr ss:[esp+14]
004184E9    C68424 5C150000 02 mov byte ptr ss:[esp+155C],2
004184F1    E8 D25B0000        call <jmp.&mfc42.#800>
004184F6    8B4C24 10          mov ecx,dword ptr ss:[esp+10]
004184FA    8D8424 48020000    lea eax,dword ptr ss:[esp+248]
00418501    50                  push eax
```
