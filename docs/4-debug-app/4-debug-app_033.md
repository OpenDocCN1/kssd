
# 分析 DNF 双开外挂

> 原文：[`www.pediy.com/kssd/pediy12/128841.html`](https://www.pediy.com/kssd/pediy12/128841.html)

**分析 DNF 双开外挂**

朋友下了一个 NDF 双开的外挂，但使用的时候杀软报了警，不知道是否是病毒，所以拿来给我帮忙看看。

这款外挂没加壳，是用 VC6 写的，大小才 36KB，所以表面上看确实很可疑。

那么接下来就让我们一起确认这到底是木马还是外挂吧，如果是外挂，还可以顺便看看它是怎么实现 DNF 双开的。

载入 `OD`，先下个 `bpx CreateFileA`，`OD` 会自动把该程序所调用的函数显示给我们。

很显然，这个程序是用 MFC 写的，让我们找找里面有什么关键函数，让我们下断点吧。

因为程序不大，所以调用的函数也不多，看到里面有多服务操作的函数，我估计该程序可能创建了一个服务程序或驱动程序，来完成后面的功能。在这些函数上面下好断点，让我们来看看，它具体创建了一个什么服务。

首先它先用互斥函数查看外挂是否已经开启，防止外挂启动多次。

```
004010EF   .  50            push    eax                          ; /MutexName
004010F0   .  6A 00         push    0                              ; |Inheritable = FALSE
004010F2   .  68 01001F00   push    1F0001                         ; |Access = 1F0001
004010F7   .  FF15 44304000 call    dword ptr [<&KERNEL32.OpenMutexA>]; \OpenMutexA
004010FD   .  85C0          test    eax, eax
004010FF   .  75 7C         jnz     short 0040117D
00401101   .  8B4C24 04     mov     ecx, dword ptr [esp+4]
00401105   .  57            push    edi
00401106   .  51            push    ecx                          ; /MutexName
00401107   .  6A 01         push    1                              ; |InitialOwner = TRUE
00401109   .  50            push    eax                          ; |pSecurity
0040110A   .  FF15 38304000 call    dword ptr [<&KERNEL32.CreateMute>; \CreateMutexA
```

接下来提升自己进程的权限：

```
004023D7  |.  50            push    eax                          ; /phToken
004023D8  |.  6A 28         push    28                           ; |DesiredAccess = TOKEN_QUERY|TOKEN_ADJUST_PRIVILEGES
004023DA  |.  FF15 68304000 call    dword ptr [<&KERNEL32.GetCurrent>; |\[GetCurrentProcess
004023E0  |.  50            push    eax                          ; |hProcess
004023E1  |.  FF15 24304000 call    dword ptr [<&ADVAPI32.OpenProces>; \OpenProcessToken
004023E7  |.  85C0          test    eax, eax
004023E9  |.  75 04         jnz     short 004023EF
004023EB  |.  83C4 1C       add     esp, 1C
004023EE  |.  C3            retn
004023EF  |>  8D4C24 04     lea     ecx, dword ptr [esp+4]
004023F3  |.  51            push    ecx                          ; /pLocalId
004023F4  |.  68 0C414000   push    0040410C                 ; |Privilege = "SeDebugPrivilege"
004023F9  |.  6A 00         push    0                              ; |SystemName = NULL
004023FB  |.  FF15 00304000 call    dword ptr [<&ADVAPI32.LookupPriv>; \LookupPrivilegeValueA
00402401  |.  85C0          test    eax, eax
00402403  |.  75 11         jnz     short 00402416
00402405  |.  8B5424 00     mov     edx, dword ptr [esp]
00402409  |.  52            push    edx                          ; /hObject
0040240A  |.  FF15 48304000 call    dword ptr [<&KERNEL32.CloseHandl>; \CloseHandle
```

之后显示出了外挂的界面：

我们单击启动，`OD` 断在了 `OpenSCManagerA` 上了。

```
00401B4C  |.  68 3F000F00   push    0F003F
00401B51  |.  BB 01000000   mov     ebx, 1
00401B56  |.  6A 00         push    0
00401B58  |.  6A 00         push    0
00401B5A  |.  885C24 44     mov     byte ptr [esp+44], bl
00401B5E  |.  FF15 18304000 call    dword ptr [<&ADVAPI32.OpenSCManagerA>]    ;  ADVAPI32.OpenSCManagerA
```

接下来判断服务是否已经存在：

```
00401C15  |.  68 FF010F00   push    0F01FF
00401C1A  |.  51            push    ecx
00401C1B  |.  57            push    edi
00401C1C  |.  FF15 10304000 call    dword ptr [<&ADVAPI32.OpenServiceA>]      ;  ADVAPI32.OpenServiceA
00401C22  |.  8BF0          mov     esi, eax
00401C24  |.  85F6          test    esi, esi
00401C26  |.  75 04         jnz     short 00401C2C
00401C28  |.  32DB          xor     bl, bl
00401C2A  |.  EB 25         jmp     short 00401C51
00401C2C  |>  8D5424 14     lea     edx, dword ptr [esp+14]
00401C30  |.  52            push    edx
00401C31  |.  53            push    ebx
00401C32  |.  56            push    esi
00401C33  |.  FF15 0C304000 call    dword ptr [<&ADVAPI32.ControlService>]    ;  ADVAPI32.ControlService
00401C39  |.  85C0          test    eax, eax
00401C3B  |.  74 08         je      short 00401C45
00401C3D  |.  FF15 6C304000 call    dword ptr [<&KERNEL32.GetLastError>]      ; [GetLastError
00401C43  |.  32DB          xor     bl, bl
00401C45  |>  57            push    edi
00401C46  |.  8B3D 20304000 mov     edi, dword ptr [<&ADVAPI32.CloseServiceHa>;  ADVAPI32.CloseServiceHandle
00401C4C  |.  FFD7          call    edi                               ;  <&ADVAPI32.CloseServiceHandle>
```

不存在就在当前路径下面释放一个名字为 `G-a-m-e-D-K.dat` 文件：

```
0040161B  |.  6A 00         push    0                               ; /hTemplateFile = NULL
0040161D  |.  6A 00         push    0                               ; |Attributes = 0
0040161F  |.  6A 02         push    2                               ; |Mode = CREATE_ALWAYS
00401621  |.  6A 00         push    0                               ; |pSecurity = NULL
00401623  |.  6A 00         push    0                               ; |ShareMode = 0
00401625  |.  68 00000040   push    40000000                        ; |Access = GENERIC_WRITE
0040162A  |.  50            push    eax                           ; |FileName
0040162B  |.  FF15 50304000 call    dword ptr [<&KERNEL32.CreateFileA>]       ; \CreateFileA
00401631  |.  8BF0          mov     esi, eax
00401633  |.  83FE FF       cmp     esi, -1
00401636  |.  74 42         je      short 0040167A
00401638  |.  8B5424 20     mov     edx, dword ptr [esp+20]
0040163C  |.  8B4424 1C     mov     eax, dword ptr [esp+1C]
00401640  |.  8D4C24 04     lea     ecx, dword ptr [esp+4]
00401644  |.  6A 00         push    0                               ; /pOverlapped = NULL
00401646  |.  51            push    ecx                           ; |pBytesWritten
00401647  |.  52            push    edx                           ; |nBytesToWrite
00401648  |.  50            push    eax                           ; |Buffer
00401649  |.  56            push    esi                           ; |hFile
0040164A  |.  FF15 4C304000 call    dword ptr [<&KERNEL32.WriteFile>]         ; \WriteFile
00401650  |.  56            push    esi                           ; /hObject
00401651  |.  FF15 48304000 call    dword ptr [<&KERNEL32.CloseHandle>]       ; \CloseHandle
```

我们先复制一份该文件，以做备份，估计该程序的主要功能都在这里面了，我们先继续分析主程序：

该文件释放完成后，主程序就创建了一个以该文件为主的内核驱动程序，并启动：

```
004018D6  |.  6A 00         push    0                               ; |ServiceStartName = NULL
004018D8  |.  6A 00         push    0                               ; |pDependencies = NULL
004018DA  |.  6A 00         push    0                               ; |pTagId = NULL
004018DC  |.  6A 00         push    0                               ; |LoadOrderGroup = NULL
004018DE  |.  52            push    edx                           ; |BinaryPathName
004018DF  |.  6A 01         push    1                               ; |ErrorControl = SERVICE_ERROR_NORMAL
004018E1  |.  6A 03         push    3                               ; |StartType = SERVICE_DEMAND_START
004018E3  |.  6A 01         push    1                               ; |ServiceType = SERVICE_KERNEL_DRIVER
004018E5  |.  68 FF010F00   push    0F01FF                        ; |DesiredAccess = SERVICE_ALL_ACCESS
004018EA  |.  50            push    eax                           ; |DisplayName
004018EB  |.  50            push    eax                           ; |ServiceName
004018EC  |.  57            push    edi                           ; |hManager
```



```
004018ED  |.  FF15 1C304000 call    dword ptr [<ADVAPI32.CreateServiceA>]    ; \CreateServiceA

00401A97  |> \57           push    edi

00401A98  |.  6A 00         push    0

00401A9A  |.  6A 00         push    0

00401A9C  |.  56           push    esi

00401A9D  |.  FF15 14304000 call    dword ptr [<ADVAPI32.StartServiceA>]     ;  ADVAPI32.StartServiceA
```

到这里，该外挂主程序的功能就分析完成了，点退出后，该外挂会删除掉该程序创建的内核服务。

从目前的行为上来说，该程序应该是快外挂而不是木马。那么接下来我们来分析下`G-a-m-e-D-K.dat`，看看它被加载后主要都做了些什么。

我们把它载入`IDA`，前面的`DriverEntry`并没做什么，都是些正常流程，如创建设备对象，和设备名称等，这些就不看了，我们直接来到关键函数处：

```
.text:000105C8 sub_105C8       proc near                 ; CODE XREF: sub_106F8:loc_10779 p

.text:000105C8               mov     eax, ds:KeServiceDescriptorTable ;取得 SSDT 表的地址

.text:000105CD               push    esi

.text:000105CE               mov     esi, [eax]        ; 把 SSDT 表中的函数地址给 esi

.text:000105D0               add     esi, 0ACh         ; 找到序号为 0xAC 的函数

.text:000105D6               mov     eax, [esi]        ; 取得该函数地址并放到 eax 中

.text:000105D8               mov     dword_1090C, eax  ; 把函数地址保存到变量中

.text:000105DD               call    sub_104EA         ; 该函数是通过 Cr0 寄存器修改内存读写属性

.text:000105E2               call    ds:KeRaiseIrqlToDpcLevel ; 提升 Irql 等级

.text:000105E8               mov     cl, al            ; NewIrql

.text:000105EA               mov     dword ptr [esi], offset sub_10574 ; Hook SSDT,把自己的函数替换 SSDT 表中的函数

.text:000105F0               call    ds:KfLowerIrql

.text:000105F6               sti                     ; 通过 Cr0 寄存器恢复内存读写属性

.text:000105F7               push    eax

.text:000105F8               mov     eax, dword_10908

.text:000105FD               mov     cr0, eax

.text:00010600               pop     eax

.text:00010601               xor     eax, eax

.text:00010603               inc     eax

.text:00010604               pop     esi

.text:00010605               retn

.text:00010605 sub_105C8       endp
```

通过对以上代码的分析，可以得出，该外挂`Hook`了`SSDT`，至于`Hook`了哪个内核函数，由于操作系统的版本不同，`SSDT`表中内核函数的序号也不同，所以我找了个干净的虚拟机，启动外挂，然后用工具查看，原来该外挂是`hook`了`NtCreateMutant`。

至于该外挂用于替换`NtCreateMutant`的函数做了什么，其实也很简单，我这就不分析了，有兴趣的朋友可以自己看看。

## 上传的附件

`![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg)`

`桌面.rar`

