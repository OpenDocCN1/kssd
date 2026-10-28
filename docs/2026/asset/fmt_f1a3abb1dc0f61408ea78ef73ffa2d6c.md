
```asm
00401F7D  |.  6A 10         push 0x10                                ; |ServiceType = SERVICE\_WIN32\_OWN\_PROCESS
00401F7F  |.  68 FF010F00   push 0xF01FF                             ; |DesiredAccess = SERVICE\_ALL\_ACCESS
00401F84  |.  8B45 0C       mov eax,[arg.2]                          ; |
00401F87  |.  50            push eax                                 ; |DisplayName
00401F88  |.  8B7D 08       mov edi,[arg.1]                          ; |
00401F8B  |.  57            push edi                                 ; |ServiceName
00401F8C  |.  56            push esi                                 ; |hManager
00401F8D  |.  FF15 00A04000 call dword ptr ds:[<&ADVAPI32.CreateServ>; \CreateServiceA   创建一个以aaaaa.exe为启动文件的服务
00401F93  |.  8BD8          mov ebx,eax
00401F95  |.  899D D0FCFFFF mov [local.204],ebx
00401F9B  |.  85DB          test ebx,ebx
00401F9D  |.  75 35         jnz short fuckme.00401FD4
00401F9F  |.  FF15 3CA04000 call dword ptr ds:[<&KERNEL32.GetLastErr>; [GetLastError
00401FA5  |.  3D 31040000   cmp eax,0x431
00401FAA  |.  75 28         jnz short fuckme.00401FD4
00401FAC  |.  68 FF010F00   push 0xF01FF
00401FB1  |.  57            push edi
00401FB2  |.  56            push esi
00401FB3  |.  FF15 2CA04000 call dword ptr ds:[<&ADVAPI32.OpenServic>;  ADVAPI32.OpenServiceA  打开服务
00401FB9  |.  8BD8          mov ebx,eax
00401FBB  |.  899D D0FCFFFF mov [local.204],ebx
00401FC1  |.  85DB          test ebx,ebx
00401FC3  |.  0F84 B5000000 je fuckme.0040207E
00401FC9  |.  6A 00         push 0x0
00401FCB  |.  6A 00         push 0x0
00401FCD  |.  53            push ebx
00401FCE  |.  FF15 04A04000 call dword ptr ds:[<&ADVAPI32.StartServi>;  ADVAPI32.StartServiceA  启动这个服务（自启动类型）
00401FD4  |>  6A 00         push 0x0
00401FD6  |.  6A 00         push 0x0
00401FD8  |.  53            push ebx
00401FD9  |.  FF15 04A04000 call dword ptr ds:[<&ADVAPI32.StartServi>;  ADVAPI32.StartServiceA
00401FDF  |.  85C0          test eax,eax
00401FE1  |.  0F84 97000000 je fuckme.0040207E
00401FE7  |.  8D95 E4FEFFFF lea edx,[local.71]
00401FED  |.  BF F8B24000   mov edi,fuckme.0040B2F8                  ;  ASCII "SYSTEM\CurrentControlSet\Services\"
00401FF2  |.  83C9 FF       or ecx,-0x1
00401FF5  |.  33C0          xor eax,eax
00401FF7  |.  F2:AE         repne scas byte ptr es:[edi]
00401FF9  |.  F7D1          not ecx
00401FFB  |.  2BF9          sub edi,ecx
00401FFD  |.  8BC1          mov eax,ecx
00401FFF  |.  8BF7          mov esi,edi
00402001  |.  8BFA          mov edi,edx
00402003  |.  C1E9 02       shr ecx,0x2
00402006  |.  F3:A5         rep movs dword ptr es:[edi],dword ptr ds>
00402008  |.  8BC8          mov ecx,eax
0040200A  |.  83E1 03       and ecx,0x3
0040200D  |.  F3:A4         rep movs byte ptr es:[edi],byte ptr ds:[>
0040200F  |.  8D95 E4FEFFFF lea edx,[local.71]
00402015  |.  8B7D 08       mov edi,[arg.1]
00402018  |.  83C9 FF       or ecx,-0x1
0040201B  |.  33C0          xor eax,eax
0040201D  |.  F2:AE         repne scas byte ptr es:[edi]
0040201F  |.  F7D1          not ecx
00402021  |.  2BF9          sub edi,ecx
00402023  |.  8BF7          mov esi,edi
00402025  |.  8BFA          mov edi,edx
00402027  |.  8BD1          mov edx,ecx
00402029  |.  83C9 FF       or ecx,-0x1
0040202C  |.  F2:AE         repne scas byte ptr es:[edi]
0040202E  |.  4F            dec edi
0040202F  |.  8BCA          mov ecx,edx
00402031  |.  C1E9 02       shr ecx,0x2
00402034  |.  F3:A5         rep movs dword ptr es:[edi],dword ptr ds>
00402036  |.  8BCA          mov ecx,edx
00402038  |.  83E1 03       and ecx,0x3
0040203B  |.  F3:A4         rep movs byte ptr es:[edi],byte ptr ds:[>
0040203D  |.  8D85 DCFDFFFF lea eax,[local.137]
00402043  |.  50            push eax                                 ; /pHandle
00402044  |.  8D8D E4FEFFFF lea ecx,[local.71]                       ; |
0040204A  |.  51            push ecx                                 ; |Subkey
0040204B  |.  68 02000080   push 0x80000002                          ; |hKey = HKEY\_LOCAL\_MACHINE
00402050  |.  FF15 08A04000 call dword ptr ds:[<&ADVAPI32.RegOpenKey>; \RegOpenKeyA  设置注册表
00402056  |.  8B75 10       mov esi,[arg.3]
00402059  |.  56            push esi                                 ; /String
0040205A  |.  FF15 48A04000 call dword ptr ds:[<&KERNEL32.lstrlenA>] ; \lstrlenA
00402060  |.  50            push eax                                 ; /BufSize
00402061  |.  56            push esi                                 ; |Buffer
00402062  |.  6A 01         push 0x1                                 ; |ValueType = REG\_SZ
00402064  |.  6A 00         push 0x0                                 ; |Reserved = 0
00402066  |.  68 1CB34000   push fuckme.0040B31C                     ; |ValueName = "Description"
0040206B  |.  8B95 DCFDFFFF mov edx,[local.137]                      ; |
00402071  |.  52            push edx                                 ; |hKey
00402072  |.  FF15 0CA04000 call dword ptr ds:[<&ADVAPI32.RegSetValu>; \RegSetValueExA  添加服务的描述信息，为从 WMI HiPerf  提供程序提供性能库信息。
00402078  |.  8BB5 D4FCFFFF mov esi,[local.203]
0040207E  |>  C745 FC FFFFF>mov [local.1],-0x1
00402085  |.  E8 1D000000   call fuckme.004020A7
0040208A  |.  8B4D F0       mov ecx,[local.4]
0040208D  |.  64:890D 00000>mov dword ptr fs:[0],ecx
00402094  |.  5F            pop edi
00402095  |.  5E            pop esi
00402096  |.  5B            pop ebx
00402097  |.  8BE5          mov esp,ebp
00402099  |.  5D            pop ebp
```

以上就是全部过程，这个是当没有被感染的的时候出现这种情况，分析的不好大家不要见怪，分析技术有限。还有一些感觉系统的东西判断该系统是xp还是2000还是win7。还有很多，先分析这些，有时间再分析。

当感染的了以后那个就不会跳，后面的我没怎么分析，大概可能也许会是这样创建了了一个显示名称为`svchost.exe`的进程，其实还是`aaaaa.exe`文件，只不过进程里面现实为`svchost.exe`，然后内存操作很多。没时间了就分析这些，以后在分析。

很多不足的地方请大牛多多指教。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [病毒分析.rar](javascript:void(0);)

  （39.76kb，75次下载）
