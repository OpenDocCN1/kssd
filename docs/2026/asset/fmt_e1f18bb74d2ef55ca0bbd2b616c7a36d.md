
如果不是系统进程或者先前模式是内核模式，则会调用 `ObCreateObject` 来创建线程内核对象

```
PAGE: 0052DAB9 loc_52DAB9:                             ; CODE XREF: PspCreateThread(x,x,x,x,x,x,x,x,x,x,x) + F320↑j
PAGE: 0052DAB9                 mov     esi, STATUS_INVALID_HANDLE
PAGE: 0052DABE                 jmp     short loc_52DAC2
```

将 `ETHREAD` 结构地址清 0

```
PAGE: 0049EDAE loc_49EDAE:                             ; CODE XREF: PspCreateThread(x,x,x,x,x,x,x,x,x,x,x) + F31A↓j
PAGE: 0049EDAE                 lea     eax, [ebp + var_Thread]
PAGE: 0049EDB1                 push    eax             ; Object
PAGE: 0049EDB2                 push    esi             ; NonPagedPoolCharge
PAGE: 0049EDB3                 push    esi             ; PagePoolCharge
PAGE: 0049EDB4                 push    260h            ; ObjectBodySize
PAGE: 0049EDB9                 push    esi             ; ParentContext
PAGE: 0049EDBA                 push    dword ptr [ebp + var_PreviousMode] ; OwnershipMode
PAGE: 0049EDBD                 push    [ebp + ObjectAttributes] ; ObjectAttributes
PAGE: 0049EDC0                 push    _PsThreadType   ; ObjectType
PAGE: 0049EDC6                 push    dword ptr [ebp + var_PreviousMode] ; PreviousMode
PAGE: 0049EDC9                 call    _ObCreateObject@36   ; ObCreateObject(x,x,x,x,x,x,x,x,x)
PAGE: 0049EDCE                 cmp     eax, esi
PAGE: 0049EDD0                 jl      loc_52DAC0
```

将 `ETHREAD` 结构地址清 0

```
PAGE: 0049EDD6                 mov     ecx, 98h
PAGE: 0049EDDB                 xor     eax, eax
PAGE: 0049EDDD                 mov     esi, [ebp + var_Thread] ; 将ETHREAD赋给esi
PAGE: 0049EDE0                 mov     edi, esi
PAGE: 0049EDE2                 rep stosd
```

为线程对象的一些成员赋值

```
PAGE: 0049EDE4                 and     [esi + _ETHREAD.RundownProtect.___u0.Ptr], eax ; 将RundownProtect赋值为 0
PAGE: 0049EDEA                 mov     [esi + _ETHREAD.ThreadsProcess], ebx ; 为ThreadProcess赋值
PAGE: 0049EDF0                 lea     edi, [esi + _ETHREAD.Cid] ; 将Cid的地址赋给edi
PAGE: 0049EDF6                 mov     eax, [ebx + _EPROCESS.UniqueProcessId] ; 将EPROCESS中的UniqueProcessId赋给eax
PAGE: 0049EDFC                 mov     [edi], eax      ; 将PID赋值edi保存的地址中
```

调用 `ExCreateHandle` 在全局句柄表中插入线程对象

```
PAGE: 0049EDFE                 mov     [ebp + var_CidEntry], esi
PAGE: 0049EE01                 and     [ebp + var_CidEntryPlus4], 0
PAGE: 0049EE05                 lea     eax, [ebp + var_CidEntry]
PAGE: 0049EE08                 push    eax
PAGE: 0049EE09                 push    _PspCidTable
PAGE: 0049EE0F                 call    _ExCreateHandle@8   ; ExCreateHandle(x,x)
PAGE: 0049EE14                 mov     [esi + _ETHREAD.Cid.UniqueThread], eax ; 将得到的句柄值也就是线程 ID 赋给ETHRAED -> Cid.UniqueThread
PAGE: 0049EE1A                 test    eax, eax
PAGE: 0049EE1C                 jz      loc_52DAD0
```

继续为线程 `ETHREAD` 对象赋值

```
PAGE: 0049EE22                 mov     eax, _MmReadClusterSize
PAGE: 0049EE27                 mov     [esi + _ETHREAD.ReadClusterSize], eax ; 为ReadClusterSize赋值
PAGE: 0049EE2D                 push    1               ; Limit
PAGE: 0049EE2F                 push    0               ; Count
PAGE: 0049EE31                 lea     eax, [esi + 1F4h] ; 取出LpcReplySemaphore地址
PAGE: 0049EE37                 push    eax             ; Semaphore
PAGE: 0049EE38                 call    _KeInitializeSemaphore@12   ; KeInitializeSemaphore(x,x,x)
PAGE: 0049EE3D                 lea     eax, [esi + 1C8h] ; 取出LpcReplyChain地址
PAGE: 0049EE43                 mov     [eax + LIST_ENTRY.Blink], eax
PAGE: 0049EE46                 mov     [eax + LIST_ENTRY.Flink], eax
PAGE: 0049EE48                 lea     eax, [esi + _ETHREAD.IrpList] ; 取出IrpList地址
PAGE: 0049EE4E                 mov     [eax + LIST_ENTRY.Blink], eax
PAGE: 0049EE51                 mov     [eax + LIST_ENTRY.Flink], eax
PAGE: 0049EE53                 lea     eax, [esi + _ETHREAD.PostBlockList] ; 取出PostBlockList地址
PAGE: 0049EE59                 mov     [eax + LIST_ENTRY.Blink], eax
PAGE: 0049EE5C                 mov     [eax + LIST_ENTRY.Flink], eax
PAGE: 0049EE5E                 and     dword ptr [esi + _ETHREAD.ThreadLock.___u0], 0   ; 将ThreadLock清 0
PAGE: 0049EE65                 lea     eax, [esi + _ETHREAD.ActiveTimerListLock] ; 取出ActiveTimerListLock地址
PAGE: 0049EE6B                 push    eax             ; SpinLock
PAGE: 0049EE6C                 call    _KeInitializeSpinLock@4   ; KeInitializeSpinLock(x)
PAGE: 0049EE71                 lea     eax, [esi + _ETHREAD.ActiveTimerListHead] ; 取出ActiveTimerListHead地址
PAGE: 0049EE77                 mov     [eax + LIST_ENTRY.Blink], eax
PAGE: 0049EE7A                 mov     [eax + LIST_ENTRY.Flink], eax
PAGE: 0049EE7A                 and     [esi + 234h], eax ; 将RundownProtect赋值为 0
```

获得进程的 `RundownProtect` 锁，以免再创建过程中进程被停掉

```
PAGE: 0049EE82                 mov     [ebp + var_RundownProtectAddress], ecx
PAGE: 0049EE85                 call    @ExAcquireRundownProtection@4   ; ExAcquireRundownProtection(x)
PAGE: 0049EE8A                 test    al, al
PAGE: 0049EE8C                 jz      loc_52DADA
```

判断传入的 `ThreadContext` 是否为 `NULL`

```
PAGE: 0049EE92                 cmp     [ebp + ThraedContext], 0
PAGE: 0049EE96                 jnz     loc_4ADFA9
```

如果不为 `NULL`，说明此次创建的是用户模式的进程，接下来就会调用 `MmCreateTeb` 来创建一个 `TEB` 并用 `InitialTeb` 来对其进行初始化

```
PAGE: 004ADFA9 loc_4ADFA9:                             ; CODE XREF: PspCreateThread(x,x,x,x,x,x,x,x,x,x,x) + 14A↑j
PAGE: 004ADFA9                 lea     eax, [ebp + var_Teb]
PAGE: 004ADFAC                 push    eax             ; int
PAGE: 004ADFAD                 push    edi             ; int
PAGE: 004ADFAE                 push    [ebp + InitialTeb] ; int
PAGE: 004ADFB1                 push    ebx             ; BugCheckParameter1
PAGE: 004ADFB2                 call    _MmCreateTeb@16   ; MmCreateTeb(x,x,x,x)
PAGE: 004ADFB7                 mov     edi, eax
PAGE: 004ADFB9                 test    edi, edi
PAGE: 004ADFBB                 jl      loc_52DAE4
```

用 `ThreadContext` 中的 `Eip` 和 `eax` 为线程对象赋值

```
PAGE: 004ADFC5                 mov     eax, [ebp + ThreadContext]
PAGE: 004ADFC8                 mov     ecx, [eax + CONTEXT._Eip]
PAGE: 004ADFCE                 mov     [esi + _ETHREAD.StartAddress], ecx
PAGE: 004ADFD4                 mov     eax, [eax + CONTEXT._Eax]
PAGE: 004ADFDA                 mov     [esi + _ETHREAD.___u17.Win32StartAddress], eax
```

调用 `KeInitThread` 来初始化线程对象，此时第三个参数是 `PspUserThreadStartup` 函数的地址，第六个参数是 `ThreadContext`
