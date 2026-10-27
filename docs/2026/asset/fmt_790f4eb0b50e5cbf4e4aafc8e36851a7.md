
`ApcState`和`SaveApcState`是`KAPC_STATE`类型的成员，`ApcStatePointer`是一个包含两个元素的指针数组，`ApcStateIndex`则是`KAPC_ENVIRONMENT`的枚举类型索引。当一个线程在它所属的进程中运行时，使用的APC链表是`ApcState`成员，此时`ApcStatePointer[0]`指向的就是`ApcState`，并且`ApcStateIndex`等于0。如果线程挂载（attach）到另一个进程中，那么，尚未交付的APC对象从`ApcState`转移到`SaveApcState`中，并且`ApcStatePointer[0]`指向的就是`SaveApcState`，`ApcStatePointer[1]`指向的就是`ApcState`，`ApcStateIndex`等于1。所以，挂载完成以后，`ApcStateIndex`知名了`ApcState`将包含的当前进程的信息，包括在新进程环境中要执行的APC对象。此后插入的APC对象都将进入到`ApcState`链表中。

等到线程回到它自己的进程(detach)时，`KeDetachProcess`首先让属于当前进程的APC对象，即`ApcState`中的APC对象被交付，然后，将`SaveApcState`中的APC转移到`ApcState`中，并且设置`ApcStateIndex`等于0。这样就恢复了该线程被挂载以前的状态。因此，`ApcState`总包含了要在当前进程环境中执行的APC对象；`ApcStateIndex`总是指向`ApcState`在指针数组`ApcStatePointer`中的下标值。APC对象的插入操作总数以`ApcStateIndex`为下标来访问数组`ApcStatePointer`中的元素。

`KAPC_STATE`和定义如下：

```
kd> dt _KAPC_STATE
nt!_KAPC_STATE
   +0x000 ApcListHead
   +0x010 Process
   +0x014 KernelApcInProgress
   +0x015 KernelApcPending
   +0x016 UserApcPending
```

`KAPC_ENVIRONMENT`枚举类型的定义如下：

```
typedef enum _KAPC_ENVIRONMENT{
    OriginalApcEnvironment,
    AttachedApcEnvironment,
    CurrentApcEnvironment,
    InsertApcEnvironment
}KAPC_ENVIRONMENT;
```

`OriginalApcEnvironment`表示原始的进程环境；`AttachedApcEnvironment`表示挂载以后的新进程环境；`CurrentApcEnvironment`表示在APC对象初始化时线程所属的进程环境；最后的`InsertApcEnvironment`表示在APC对象被插入时线程所属的进程环境。在`APC_STATE`结构中，`Process`域指向了一个进程对象，代表了这些APC所关联的进程，`PsGetCurrentProcess`函数会从该与域获取进程对象的原因就是无论是否是挂载状态，总能获取正确的进程对象

```
.text:0040ED86                 public _PsGetCurrentProcess@0
.text:0040ED86 _PsGetCurrentProcess@0 proc near
.text:0040ED86                 mov     eax, large fs:124h ; IoGetCurrentProcess
.text:0040ED8C                 mov     eax, [eax+_KTHREAD.ApcState.Process]
.text:0040ED8F                 retn
.text:0040ED8F _PsGetCurrentProcess@0 endp
```

`ApcListHead`数组有两个元素，分别代表了内核模式APC对象链表和用户模式APC对象链表。三个布尔变量`KernelApcInProcess`, `KernelApcPending`和`UserApcPending`分别表示：该线程当前正在处理一个内核APC，有内核模式APC对象正在等待被交付，以及有用户模式APC对象正在等待被交付。

## 4.APC机制的实现

### A.系统调用`NtQueueApcThread`

不仅内核可以向一个线程发出APC请求，别的线程乃至目标线程自身也可以发出这样的请求。Win32 API为应用程序提供了一个函数`QueueUserAPC`就是用于此目的，而该函数是通过调用内核的`NtQueueApcThread`来实现的，第三个参数为函数`BaesDispatchAPC`函数地址，所以当从用户层插入APC的时候`NormalRoutine`被指定为`BaseDispatchAPC`，而用户真正要指向的函数则用第三个参数`NormalContext`传递进去

```
.text:7C82C0CF loc_7C82C0CF:                           ; CODE XREF: QueueUserAPC(x,x,x)+18B79↓j
.text:7C82C0CF                 push    eax             ; SystemArgument2
.text:7C82C0D0                 push    [ebp+dwData]    ; SystemArgument1
.text:7C82C0D3                 push    [ebp+pfnAPC]    ; NormalContext
.text:7C82C0D6                 push    offset _BaseDispatchAPC@12 ; NormalRoutine
.text:7C82C0DB                 push    [ebp+hThread]   ; ThreadHandle
.text:7C82C0DE                 call    ds:__imp__NtQueueApcThread@20 ; NtQueueApcThread(x,x,x,x,x)
.text:7C82C0E4                 xor     ecx, ecx
.text:7C82C0E6                 test    eax, eax
.text:7C82C0E8                 setnl   cl
.text:7C82C0EB                 mov     eax, ecx
```

在`NtQueueApcThread`函数中，首先将线程的先前模式保存在局部变量中

```
PAGE:004C2AAA ; NTSTATUS __stdcall NtQueueApcThread(HANDLE ThreadHandle, PKNORMAL_ROUTINE ApcRoutine, PVOID NormalContext, PVOID SystemArgument1, PVOID SystemArgument2)
PAGE:004C2AAA _NtQueueApcThread@20 proc near          ; DATA XREF: .text:0040DAF0↑o
PAGE:004C2AAA
PAGE:004C2AAA AccessMode = byte ptr -4
PAGE:004C2AAA ThreadHandle = dword ptr 8
PAGE:004C2AAA ApcRoutine = dword ptr 0Ch
PAGE:004C2AAA NormalContext = dword ptr 10h
PAGE:004C2AAA SystemArgument1 = dword ptr 14h
PAGE:004C2AAA SystemArgument2 = dword ptr 18h
PAGE:004C2AAA
PAGE:004C2AAA ; FUNCTION CHUNK AT PAGE:0052E6D5 SIZE 00000025 BYTES
PAGE:004C2AAA
PAGE:004C2AAA                 mov     edi, edi
PAGE:004C2AAC                 push    ebp
PAGE:004C2AAD                 mov     ebp, esp
PAGE:004C2AAF                 push    ecx
PAGE:004C2AB0                 push    ebx
PAGE:004C2AB1                 push    esi
PAGE:004C2AB2                 mov     eax, large fs:124h ; 将线程对象赋给eax
PAGE:004C2AB8                 mov     al, [eax+_KTHREAD.PreviousMode]
PAGE:004C2ABE                 mov     [ebp+AccessMode], al ; 为局部变量赋值
```

调用函数来获得线程句柄的线程对象

```
PAGE:004C2AC1                 xor     esi, esi        ; esi清0
PAGE:004C2AC3                 push    esi             ; HandleInformation
PAGE:004C2AC4                 lea     eax, [ebp+Object]
PAGE:004C2AC7                 push    eax             ; Object
PAGE:004C2AC8                 push    dword ptr [ebp+AccessMode] ; AccessMode
PAGE:004C2ACB                 push    _PsThreadType   ; ObjectType
PAGE:004C2AD1                 push    10h             ; DesiredAccess
PAGE:004C2AD3                 push    [ebp+ThreadHandle] ; Handle
PAGE:004C2AD6                 call    _ObReferenceObjectByHandle@24 ; ObReferenceObjectByHandle(x,x,x,x,x,x)
PAGE:004C2ADB                 mov     ebx, eax
PAGE:004C2ADD                 cmp     ebx, esi
PAGE:004C2ADF                 jl      short loc_4C2B42
```

对解析到的线程对象的`CrossThreadFlags`进行检测

```
PAGE:004C2AE1                 mov     eax, [ebp+Object]
PAGE:004C2AE4                 xor     ebx, ebx
PAGE:004C2AE6                 test    byte ptr [eax+_ETHREAD.___u24.CrossThreadFlags], OBJ_PERMANENT
PAGE:004C2AED                 jnz     loc_52E6D5
```

如果通过检测，接下来就调用函数来申请0x30大小的内存用来保存APC对象

```
PAGE:004C2AF3                 push    edi
PAGE:004C2AF4                 push    'pasP'          ; Tag
PAGE:004C2AF9                 push    30h             ; NumberOfBytes
PAGE:004C2AFB                 push    8               ; PoolType
PAGE:004C2AFD                 call    _ExAllocatePoolWithQuotaTag@12 ; ExAllocatePoolWithQuotaTag(x,x,x)
PAGE:004C2B02                 mov     edi, eax        ; 将申请到的内存地址赋给edi
PAGE:004C2B04                 cmp     edi, esi
PAGE:004C2B06                 jz      loc_52E6DF
```

通过函数来初始化APC对象

```
PAGE:004C2B0C                 push    [ebp+NormalContext] ; NormalContext
PAGE:004C2B0F                 push    1               ; ApcMode
PAGE:004C2B11                 push    [ebp+ApcRoutine] ; NormalRoutine
PAGE:004C2B14                 push    esi             ; RundownRoutine
PAGE:004C2B15                 push    offset _IopDeallocateApc@20 ; KernelRoutine
PAGE:004C2B1A                 push    esi             ; int
PAGE:004C2B1B                 push    [ebp+ThreadHandle] ; Thread
PAGE:004C2B1E                 push    edi             ; Apc
PAGE:004C2B1F                 call    _KeInitializeApc@32
```

将初始化完成的APC对象插入到线程中

