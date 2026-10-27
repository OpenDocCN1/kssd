
```
PAGE:0049F115 ; int __stdcall KeInitThread(PKTHREAD Thread, PVOID KernelStack, int, PKSTART_ROUTINE StartRoutine, PVOID StartContext, PCONTEXT ContextFrame, PVOID Teb, PEPROCESS Process)
PAGE:0049F115 _KeInitThread@32 proc near              ; CODE XREF: PspCreateThread(x,x,x,x,x,x,x,x,x,x,x)+178↑p
PAGE:0049F115                                         ; KeInitializeThread(x,x,x,x,x,x,x,x)+1D↓p
PAGE:0049F115
PAGE:0049F115 var_20 = dword ptr -20h
PAGE:0049F115 var_19 = byte ptr -19h
PAGE:0049F115 ms_exc = CPPEH_RECORD ptr -18h
PAGE:0049F115 Thread = dword ptr 8
PAGE:0049F115 KernelStack = dword ptr 0Ch
PAGE:0049F115 SystemRoutine = dword ptr 10h
PAGE:0049F115 StartRoutine = dword ptr 14h
PAGE:0049F115 StartContext = dword ptr 18h
PAGE:0049F115 ContextFrame = dword ptr 1Ch
PAGE:0049F115 Teb = dword ptr 20h
PAGE:0049F115 Process = dword ptr 24h
PAGE:0049F115
PAGE:0049F115                 push    10h
PAGE:0049F117                 push    offset stru_415040
PAGE:0049F11C                 call    __SEH_prolog
PAGE:0049F121                 mov     [ebp+var_19], 0
PAGE:0049F125                 mov     esi, [ebp+Thread] ; 将线程对象赋给esi
PAGE:0049F128                 mov     [esi+_KTHREAD.Header.Type], ThreadObject
PAGE:0049F12B                 mov     [esi+_KTHREAD.Header.Size], 70h
PAGE:0049F12F                 lea     eax, [esi+_KTHREAD.Header.WaitListHead]
PAGE:0049F132                 mov     [eax+LIST_ENTRY.Blink], eax
PAGE:0049F135                 mov     [eax+LIST_ENTRY.Flink], eax
PAGE:0049F137                 lea     eax, [esi+_KTHREAD.MutantListHead]
PAGE:0049F13A                 mov     [eax+LIST_ENTRY.Blink], eax
PAGE:0049F13D                 mov     [eax+LIST_ENTRY.Flink], eax
PAGE:0049F13F                 lea     eax, [esi+_KTHREAD.WaitBlock.Thread]
PAGE:0049F142                 push    4
PAGE:0049F144                 pop     ecx
PAGE:0049F145
PAGE:0049F145 loc_49F145:                             ; CODE XREF: KeInitThread(x,x,x,x,x,x,x,x)+36↓j
PAGE:0049F145                 mov     [eax], esi
PAGE:0049F147                 add     eax, 18h
PAGE:0049F14A                 dec     ecx
PAGE:0049F14B                 jnz     short loc_49F145
```

## 初始化系统服务表，APC的域

```
PAGE:0049F14D                 mov     edi, [ebp+Process] ; 将进程对象赋给edi
PAGE:0049F150                 mov     al, [edi+_KPROCESS.AutoAlignment]
PAGE:0049F153                 mov     [esi+_KTHREAD.AutoAlignment], al
PAGE:0049F159                 mov     [esi+_KTHREAD.EnableStackSwap], 1
PAGE:0049F160                 mov     [esi+_KTHREAD.KernelStackResident], 1
PAGE:0049F167                 mov     [esi+_KTHREAD.ServiceTable], offset _KeServiceDescriptorTable ; 为系统服务表赋值
PAGE:0049F171                 lea     eax, [esi+_KTHREAD.ApcState] ; 将Apc地址赋给eax
PAGE:0049F174                 mov     [esi+_KTHREAD.ApcStatePointer], eax ; 将eax地址赋给ApcStatePointer[0]
PAGE:0049F17A                 lea     ecx, [esi+_KTHREAD.SavedApcState] ; 将SaveApcState地址赋给ecx
PAGE:0049F180                 mov     [esi+13Ch], ecx ; 将ecx赋给ApcStatePointer[1]
PAGE:0049F186                 mov     [eax+LIST_ENTRY.Blink], eax
PAGE:0049F189                 mov     [eax+LIST_ENTRY.Flink], eax
PAGE:0049F18B                 lea     eax, [esi+3Ch]  ; 将ApcState第二个链表地址，也就是用户链表地址赋给eax
PAGE:0049F18E                 mov     [eax+LIST_ENTRY.Blink], eax
PAGE:0049F191                 mov     [eax+LIST_ENTRY.Flink], eax
PAGE:0049F193                 mov     [esi+_KTHREAD.ApcState.Process], edi
PAGE:0049F196                 mov     [esi+_KTHREAD.ApcQueueable], 1
```

## 调用`KeInitializeApc`初始化一个内核APC对象

```
PAGE:0049F19D                 xor     ebx, ebx        ; ebx清0
PAGE:0049F19F                 push    ebx             ; NormalContext
PAGE:0049F1A0                 push    ebx             ; ApcMode
PAGE:0049F1A1                 push    offset _KiSuspendThread@12 ; NormalRoutine
PAGE:0049F1A6                 push    offset _KiSuspendRundown@4 ; RundownRoutine
PAGE:0049F1AB                 push    offset _KiSuspendNop@20 ; KernelRoutine
PAGE:0049F1B0                 push    ebx             ; int
PAGE:0049F1B1                 push    esi             ; Thread
PAGE:0049F1B2                 lea     eax, [esi+_KTHREAD.SuspendApc]
PAGE:0049F1B8                 push    eax             ; Apc
PAGE:0049F1B9                 call    _KeInitializeApc@32
```

### 该APC对象的`NormalRoutine`为`KiSuspendThread`，该函数实现如下

```
.text:00423E71 ; void __stdcall KiSuspendThread(PVOID NormalContext, PVOID SystemArgument1, PVOID SystemArgument2)
.text:00423E71 _KiSuspendThread@12 proc near           ; DATA XREF: KeInitThread(x,x,x,x,x,x,x,x)+8C↓o
.text:00423E71
.text:00423E71 NormalContext = dword ptr 4
.text:00423E71 SystemArgument1 = dword ptr 8
.text:00423E71 SystemArgument2 = dword ptr 0Ch
.text:00423E71
.text:00423E71                 mov     eax, large fs:124h
.text:00423E77                 xor     ecx, ecx
.text:00423E79                 push    ecx             ; Timeout
.text:00423E7A                 push    ecx             ; Alertable
.text:00423E7B                 push    ecx             ; WaitMode
.text:00423E7C                 push    5               ; WaitReason
.text:00423E7E                 add     eax, _KTHREAD.SuspendSemaphore
.text:00423E83                 push    eax             ; Object
.text:00423E84                 call    _KeWaitForSingleObject@20 ; KeWaitForSingleObject(x,x,x,x,x)
.text:00423E89                 retn    0Ch
.text:00423E89 _KiSuspendThread@12 endp
```

## 初始化互斥体，定时器，`WaitBlock`，APC锁
