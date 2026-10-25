
# Windows Internals---Processes,Threads,Jobs

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-340.htm`](https://www.pediy.com/kssd/pediy09/pediy09-340.htm)
>
> 译文作者：sudami
>
> 信息来源：邪恶八进制信息安全团队（[`www.eviloctal.com/`](http://www.eviloctal.com/)）
>
> 文章备注：此文不仅仅是翻译,其中有部分为译者自己的总结,写的很烂,共勉之
>
> +++++++++++++++++++++++++++++
>
> +   sudami  \[xiao\_rui\_119@163.com\] +
>
> +   2007.12.07                     +
>
> +   英文翻译&学习笔记         +
>
> +++++++++++++++++++++++++++++
>
> 这是 Windows Internals 第六章的内容。发现描述的还算详细。读了总比不读强。于是纪录之，老鸟飘过![恐怖!](img/5d638e741426994b728ef5ba26021d82.jpg "恐怖!")  ----

## <一> 进程相关

`EPROCESS` 算是进程的代表，它还保留着与之相关的其他信息。进程是死的，其中的线程才是活的。进程就好比一个容器，装着很多活跃的线程而已\[没有线程的进程注定要死亡，因为它没有活力\]。对了，还有 `PEB`、`TEB`\[这 2 个结构存在于用户进程空间中，其他的都在系统的高 2GB 地址范围里\] 。见图：

![名称:  Snap1.gif> 查看次数: 368> 文件大小:  14.1 KB](img/ce7aa2ab6d2d9294166a9b675e803041.jpg)

### ① 明确 KPCR、KPRCB、ETHREAD、KTHREAD、EPROCESS、KPROCESS、TEB、PEB

`KPCR`（Kernel's Processor Control Region,**内核进程控制区域**）是一个不会随 WINDOWS 版本变动而改变的固定结构体，在它的末尾\[偏移`0x120`\]指向`KPRCB`结构。

`nt!\_KPCR`

```
  +0x000 NtTib            : _NT_TIB
  +0x01c SelfPcr          : Ptr32 _KPCR
  +0x020 Prcb             : Ptr32 _KPRCB
  +0x024 Irql             : UChar
  +0x028 IRR              : Uint4B
  +0x02c IrrActive        : Uint4B
  +0x030 IDR              : Uint4B
  +0x034 KdVersionBlock   : Ptr32 Void
  +0x038 IDT              : Ptr32 _KIDTENTRY
  +0x03c GDT              : Ptr32 _KGDTENTRY
  +0x040 TSS              : Ptr32 _KTSS
  ...// 省略
  +0x120 PrcbData         : _KPRCB
```

`KPRCB`同样是一个不会随 WINDOWS 版本变动而改变的固定结构体。它包含有指向当前`KTHREAD`的指针，偏移值`0x004`。其实也就是知道了当前的`ETHREAD`基地址。\[因为 ETHREAD 的第一项便是 KTHREAD，ETHREAD 在后面讨论，现在讨论进程相关\] \[通过 `KeGetCurrentPrcb()` 函数即可得到 PKPRCB，具体参见 WRK\]

![名称:  1.gif> 查看次数: 366> 文件大小:  4.2 KB](img/e4ee395e11fa93ef924adf85c99f0f55.jpg)

展开 KTHREAD，其中的`_KAPC_STATE` 结构中包含当前 KPROCESS 的地址

`nt!\_KTHREAD`

```
  +0x000 Header           : _DISPATCHER_HEADER
  ...

+0x034 ApcState         : _KAPC_STATE
+0x034 ApcState         : struct _KAPC_STATE, 5 elements, 0x18 bytes
      +0x000 ApcListHead     : [2] struct _LIST_ENTRY, 2 elements, 0x8 bytes

+0x010 Process
          : Ptr32 to struct
_KPROCESS
, 29 elements, 0x6c bytes
      +0x014 KernelApcInProgress : UChar
      +0x015 KernelApcPending : UChar
      +0x016 UserApcPending   : UChar
```

而 EPROCESS 的第一项正是 KPROCESS。联想我们熟悉的断 EPROCESS 链表隐藏进程的手法。通过`PsGetCurrentProcess`得到的其实是当前 KPROCESS 的地址，而 KPROCESS 就是 EPROCESS 结构体的第一项，这样就得到了当前的 EPROCESS。然后遍历整个链表。。。

---->>大致流程：

`PsGetCurrentProcess()函数--->_PsGetCurrentProcess()宏----->KeGetCurrentThread()函数`

---->>具体细节：

```
#define _PsGetCurrentProcess() (CONTAINING_RECORD(((KeGetCurrentThread())->ApcState.Process),EPROCESS,Pcb))
// 很明显，KeGetCurrentThread()得到 KTHREAD 结构体，KTHREAD 偏移 0x034 处的
// ApcState 中 process 即为 EPROCESS 的第一项 KPROCESS 的地址。CONTAINING_RECORD 宏
// 将此地址减去它在 EPROCESS 中的偏移值，得到当前 EPROCESS 的实际地址
FORCEINLINE
struct _KTHREAD *
NTAPI KeGetCurrentThread (VOID)
{
#if (_MSC_FULL_VER >= 13012035)
    return (struct _KTHREAD *) (ULONG_PTR) __readfsdword (FIELD_OFFSET (KPCR, PrcbData.CurrentThread));
#else
    __asm { mov eax, fs:[0] KPCR.PrcbData.CurrentThread }
#endif
}
// fs 在用户模式下指向 TEB 结构，在内核模式下指向 KPCR[前面第一个描述的结构体]
```

呵呵，偶画个图更直观些，也更方便记忆

![名称:  Snap3.gif> 查看次数: 364> 文件大小:  15.2 KB](img/7077ae5b81767a39567b4b56416fa707.jpg)

关于`KPROCESS`。里面保存了一些有用的信息，我们来简单的瞅下。

`nt!\_KPROCESS`

```
  +0x000 Header           : _DISPATCHER_HEADER
  +0x010 ProfileListHead : _LIST_ENTRY
  +0x018
DirectoryTableBase
 : [2] Uint4B

// 进程的页目录 PDT [涉及内存管理知识]

  +0x020

LdtDescriptor
  : _KGDTENTRY

// GDT 的入口

  +0x028

Int21Descriptor
 : _KIDTENTRY

// IDT 的入口
  +0x030 IopmOffset       : Uint2B
  +0x032 Iopl             : UChar
  +0x033 Unused           : UChar
  +0x034 ActiveProcessors : Uint4B
  +0x038 KernelTime       : Uint4B
  +0x03c UserTime         : Uint4B
  +0x040 ReadyListHead    : _LIST_ENTRY
  +0x048 SwapListEntry    : _SINGLE_LIST_ENTRY
  +0x04c VdmTrapcHandler : Ptr32 Void
  +0x050

ThreadListHead
  : _LIST_ENTRY
// 指向 KTHREAD 链
  +0x058 ProcessLock      : Uint4B
  +0x05c Affinity         : Uint4B
  +0x060 StackCount       : Uint2B
  +0x062 BasePriority     : Char
  +0x063 ThreadQuantum    : Char
  +0x064 AutoAlignment    : UChar
  +0x065 State            : UChar
  +0x066 ThreadSeed       : UChar
  +0x067 DisableBoost     : UChar
  +0x068 PowerState       : UChar
  +0x069 DisableQuantum   : UChar
  +0x06a IdealNode        : UChar
  +0x06b Flags            : _KEXECUTE_OPTIONS
  +0x06b ExecuteOptions   : UChar
```

`PEB`是很有用的东西，写 shellcode、定位 EPROCESS 等都可以用到它。PEB 在 EPROCESS 偏移 0x1b0 处

`nt!\_EPROCESS`

```
  +0x000 Pcb              : _KPROCESS
  ...
  +0x084 UniqueProcessId : Ptr32 Void
  +0x088 ActiveProcessLinks : _LIST_ENTRY
  ...
  +0x160 PhysicalVadList : _LIST_ENTRY
  +0x168 PageDirectoryPte : _HARDWARE_PTE
  ...

+0x1b0 Peb              : Ptr32 _PEB
  ...
```

---->>获得 PEB 的地址是非常简单的。可以通过 EPROCESS 的偏移，也可以用硬编码实现\[不同进程的 PEB 高位都是一样的\]

```
xor esi, esi                ;   FS 寄存器 -> TEB 结构，TEB+0x30 -> PEB 结构
mov esi, fs:[esi + 30H]   ;   而 PEB 中包含有_PEB_LDR_DATA。通过一系列的
mov eax, esi               ;   偏移可以定位到 Kernel32.dll 基地址。。。
ret                       ;   呵呵，参看 gz1X 大虾的文章：
```

[

WIN 下获取 kernel 基址的 shellcode 探讨

](http://hi.baidu.com/gz1x/blog/item/c3c8f8f8b028040cd9f9fd90.html)

当然可以直接用 Windbg 来查看当前的 PEB 的结构 `lkd>dt _peb`

`lkd> !peb`

```
PEB at 7ffd
c000

//高位都是 7ffd

   InheritedAddressSpace:    No
   ReadImageFileExecOptions: No
   BeingDebugged:           No
   ImageBaseAddress:        01000000
   Ldr                     00191e90
   Ldr.Initialized:        Yes
   Ldr.InInitializationOrderModuleList: 00191f28 . 00193330
   Ldr.InLoadOrderModuleList:         00191ec0 . 00193320
   Ldr.InMemoryOrderModuleList:       00191ec8 . 00193328
   ...// 省略
```

### ② 与进程相关的一些内核变量、计数、函数

![名称:  Snap5.gif> 查看次数: 358> 文件大小:  28.3 KB](img/2497a78333f1691c78b6149383fe4187.jpg)

关于 PspCidTable 参见 gz1X 大虾的：[基于 pspCidTable 的进程检测技术](http://hi.baidu.com/gz1x/blog/item/d99aeefa4d1c92ddb48f31b9.html)

这些变量的申明保存在 WRK 的`Psinit.c`中。先看下`PspCreateProcess`的参数

```
PspCreateProcess(
    OUT PHANDLE ProcessHandle,
    IN ACCESS_MASK DesiredAccess,
    IN POBJECT_ATTRIBUTES ObjectAttributes OPTIONAL,
    IN HANDLE ParentProcess OPTIONAL,
//如果没有指定，表明此进程没有父进程。是系统进程
    IN ULONG Flags,
    IN HANDLE SectionHandle OPTIONAL,
    IN HANDLE DebugPort OPTIONAL,
    IN HANDLE ExceptionPort OPTIONAL,
    IN ULONG JobMemberLevel
    )
```

系统启动时,`PspInitPhase0()` 函数会做很多事情，这里只是例举与上图全局变量相关的细节：

初始化 Queue header：`InitializeListHead(&PsActiveProcessHead);`

初始化 PsIdleProcess\[这个是系统进程的 EPROCESS\]

![名称:  Snap10.gif> 查看次数: 351> 文件大小:  6.4 KB](img/b5b8e7b3e2538c9a080158b7ba7968d6.jpg)

创建系统进程，并且把系统进程的 EPROCESS 保存在`PsInitialSystemProcess`中。见代码：

```
InitializeObjectAttributes (&ObjectAttributes,
                            NULL,
                            0,
```



```
NULL,

NULL);

if (!NT_SUCCESS (PspCreateProcess (&PspInitialSystemProcessHandle,

PROCESS_ALL_ACCESS,

&ObjectAttributes,


NULL, // 注意这里。没有指定，所以创建的是系统进程


0,

NULL,

NULL,

NULL,

0))) {
return FALSE;
}
if (!NT_SUCCESS (ObReferenceObjectByHandle (PspInitialSystemProcessHandle,
0L,
PsProcessType,
KernelMode,


&PsInitialSystemProcess
, // 系统的 EPROCESS 保存于此
NULL))) {
return FALSE;
}
strcpy((char *) &PsIdleProcess->ImageFileName[0], "Idle");
strcpy((char *) &PsInitialSystemProcess->ImageFileName[0], "System");
// EPROCESS OFFEST+0x1f4 SeAuditProcessCreationInfo : _SE_AUDIT_PROCESS_CREATION_INFO
PsInitialSystemProcess->SeAuditProcessCreationInfo.ImageFileName =
ExAllocatePoolWithTag (PagedPool,
sizeof(OBJECT_NAME_INFORMATION),
'aPeS');
if (PsInitialSystemProcess->SeAuditProcessCreationInfo.ImageFileName != NULL) {
RtlZeroMemory (PsInitialSystemProcess->SeAuditProcessCreationInfo.ImageFileName,
sizeof (OBJECT_NAME_INFORMATION));
} else {
return FALSE;
}
```

下面几个全局变量定义在 `WRK` 的 `Psp.h` 文件中：

```
ULONG PspCreateProcessNotifyRoutineCount;

EX_CALLBACK PspCreateProcessNotifyRoutine[8];

ULONG PspLoadImageNotifyRoutineCount;

EX_CALLBACK PspLoadImageNotifyRoutine[8];

extern PHANDLE_TABLE PspCidTable;
```

![名称: aa.gif> 查看次数: 260> 文件大小: 20.7 KB](img/137fe5a111d590a5c752973076f56a50.jpg)

![名称: bb.gif> 查看次数: 262> 文件大小: 16.2 KB](img/eca8cec46dd687e9ef6494f36a4b4e0b.jpg)

相关的函数就更多了，具体请参见原版 `WINDOWS` `INTERNALS` `6`[附件里有]

### ③ 一个进程的诞生，`CreateProcess` 的步骤 ----

用户程序可以调用 `CreateProcess`、`CreateProcessAsUser`、`CreateProcessWithTokenWor`、`CreateProcessWithLogonW` 创建进程。而进程的创建主要由 3 部分参合进来完成的：`Kernel32.dll`、`the` `Windows` `executive`、子系统进程 `CSRSS.exe`。

`CreateProcess` 创建进程的大致步骤：

1. 打开文件[.exe]。
2. 创建进程内核对象
3. 在进程中创建线程[堆栈、线程上下文、线程内核对象]
4. 通知 `WINDOWS` 子系统已经创建了一个新进程，便于它进一步的初始化
5. 如果标致不是 `CREATE_SUSPENDED`，那么就开始执行进程中的线程
6. 在进程和线程的 context 里，完成地址空间的初始化[eg.加载需要的 DLL]，开始执行程序

![名称: Snap7.gif> 查看次数: 352> 文件大小: 22.0 KB](img/c4dfd79fdc9d60e7ee10e3f50cdedca2.jpg)

`CreateProcess` 在打开文件运行之前，会检查参数中的 flags，决定如何设置新进程的优先级。

\---->> stage 1: 打开并运行文件 <<----

如果是 `PE` 文件，直接运行；如果不是 `PE` 格式，系统根据相应的格式选择相应的加载措施：

*   `POSIX` 的程序 -- 调用 `Posix.exe`
*   `OS/2` `1.x` 的程序 -- 调用 `Os2.exe`
*   `.bat`/.cmd 的程序 -- 调用 `Cmd.exe`
*   16 位/`MS-DOS` `.exe`/.com/.pif -- 调用 `Ntvdm.exe`

如果还是不能加载，`CreateProcess` 就会失败。

![名称: Snap8.gif> 查看次数: 348> 文件大小: 11.1 KB](img/8726bc15b4b916f932407adb842d2520.jpg)

与 `Ntvdm.exe` 相关的注册表在 `HKLM\SYSTEM\CurrentControlSet\Control\WOW` 下

![名称: Snap9.gif> 查看次数: 348> 文件大小: 5.4 KB](img/b338a136f035fcaecc3a3aec251eb4d6.jpg)

到此，`CreateProcess` 已经成功的打开了一个有效的文件，并且为它创建了一个 section object。但并没有映射到内存中，仅仅是打开了而已。a section object 被成功的创建了并不能说明此文件是一个有效的 Windows image，因为文件可能是 `DLL` 或者 `POSIX`。

接着检查注册表 `IFEO`，系统如果发现某个程序文件在 `IFEO` 列表中，它就会首先来读取 `Debugger` 参数，如果该参数不为空，系统则会把 `Debugger` 参数里指定的程序文件名作为用户试图启动的程序执行请求来处理，而仅仅把用户试图启动的程序作为 `Debugger` 参数里指定的程序文件名的参数发送过去。 联想到了很久的映像劫持病毒。

\---->> stage 2: 创建进程内核对象 <<----

`NtCreateProcess`-->`NtCreateProcessEx`-->;`PspCreateProcess` 来创建进程内核对象，如下：

1. 设置 `EPROCESS`
2. 创建进程地址空间
3. 初始化 `KPROCESS` // 偶在源码里面没有找到耶，太多了，反正没找到
4. 完成进程地址空间的设置 // 原文中写着此时 `Ntdll.dll` 被映射到进程地址空间中
5. 设置 `PEB`
6. 完成进程内核对象的创建，审核收尾工作

#### \----- stage2.1 设置 EPROCESS-----

1. 创建 `EPROCESS`

//系统在启动时在 `PspInitPhase0()`中初始化进程/线程对象的对象类型[Object types]

`// Create Object types for Thread and Process Objects.`

```
RtlInitUnicodeString (&NameString, L"Process");
ObjectTypeInitializer.DefaultPagedPoolCharge = PSP_PROCESS_PAGED_CHARGE;
ObjectTypeInitializer.DefaultNonPagedPoolCharge = PSP_PROCESS_NONPAGED_CHARGE;
ObjectTypeInitializer.DeleteProcedure = PspProcessDelete;
ObjectTypeInitializer.ValidAccessMask = PROCESS_ALL_ACCESS;
ObjectTypeInitializer.GenericMapping = PspProcessMapping;
if (!NT_SUCCESS (**ObCreateObjectType** (&NameString,
&ObjectTypeInitializer,
(PSECURITY_DESCRIPTOR) NULL,
&PsProcessType))) {
return FALSE;

// PsProcessType 是在 Psinit.c 中定义的全局变量 POBJECT_TYPE PsProcessType;
}

// 用它来保存对象类型的信息

// 函数 PspCreateProcess 中创建 EPROCESS
// Create the process object
Status =** ObCreateObject** (PreviousMode,

// PreviousMode = KeGetPreviousModeByThread(&CurrentThread->Tcb);
PsProcessType,

// 就是上面描述的那个全局变量
ObjectAttributes,

// 函数的参数
PreviousMode,
NULL,
sizeof (EPROCESS),
0,
0,
&Process);

// 创建后的 EPROCESS 保存在此
RtlZeroMemory (Process, sizeof(EPROCESS));
ExInitializeRundownProtection (&Process->RundownProtect);
PspInitializeProcessLock (Process);
InitializeListHead (&Process->ThreadListHead);
```

2. 设置进程 Working set size 的最值[20-45]

```
WorkingSetMinimum = PsMinimumWorkingSet; // 20
WorkingSetMaximum = PsMaximumWorkingSet; // 45
```

3. 从父进程那里继承一些属性[略过]

#### \----- stage2.2 创建进程地址空间-----

1. page directory 即是 PDT 相关。 嘿嘿，想必大家都看过操作系统之类的书，上面介绍的有关内存管理的东西必须好好的看
   前置知识是必须充分了解 `JT`、`MBT`、`PMT`、`PMTR`、`PTE`、`PDE` 等其中页表(PTE 映射在系统高 2GB 中--0xC0000000~0xC03FFFFF 的 4MB 空间，页目录映射在 0xC0300000~...)

PS: 进程的地址空间分为 4 种：

1. `Boot Process`--Address space is initialized during `MmInit`. Parent is not specified
2. `System Process`--系统地址空间，看看前面介绍的这个全局变量 `PspInitialSystemProcess`，熟悉了吧
3. `User Process (Cloned Address Space)` 从指定的进程复制的
4. `User Process (New Image Address Space)` 新的
\------------------------------------------------------

```
if (SectionHandle != NULL) {
// User Process (New Image Address Space).
Status = **MmInitializeProcessAddressSpace ** (Process,
NULL,
SectionObject,
&Flags,
&(Process->SeAuditProcessCreationInfo.ImageFileName));
...//省略
} else if (Parent != NULL) {

// 如果存在父进程
if (Parent != PsInitialSystemProcess) {

// 如果父进程不是系统进程，说明是一个普通的父进程创建的
Process->SectionBaseAddress = Parent->SectionBaseAddress;
// User Process ( Cloned Address Space )

Status = MmInitializeProcessAddressSpace (Process,
```



```c
            Parent,  
            NULL,  
            &Flags,  
            NULL);  
    // A cloned process isn't started from an image file, so we give it the name  
    // of the process of which it is a clone, provided the original has a name.  
    // 暂时不重要，省略掉  
       } else {  
    // System Process  
    Flags &= ~PROCESS_CREATE_FLAGS_ALL_LARGE_PAGE_FLAGS;  
    Status = MmInitializeProcessAddressSpace (Process,  
            NULL,  
            NULL,  
            &Flags,  
            NULL);  
   }  

```

2. Hyperspace page  
3. Working set list  

--- stage2.5 设置 PEB----
```c
if (Parent && CreatePeb) {  
       RtlZeroMemory (&InitialPeb, FIELD_OFFSET(INITIAL_PEB, Mutant));  
        
// PspCreateProcess 中申明的局部变量 INITIAL_PEB InitialPeb;  

       InitialPeb.Mutant = (HANDLE)(-1);  
       InitialPeb.ImageUsesLargePages = (BOOLEAN) UseLargePages;  
            
       if (SectionHandle != NULL) {  
// 如果要创建新的进程.PEB 也得从新创建  

       Status =  
MmCreatePeb  
(Process, &InitialPeb, &Process->;Peb);  
       Peb = Process->;Peb;  
       } else {  
// 如果是 CLONE 进程,PEB 只要从父进程里面拷贝一份就可以了  

       SIZE_T BytesCopied;  
       InitialPeb.InheritedAddressSpace = TRUE;  
        
Process->;Peb = Parent->;Peb;  
        
MmCopyVirtualMemory  
(CurrentProcess,  
                &InitialPeb,  
                Process,  
                Process->;Peb,  
                sizeof (INITIAL_PEB),  
                KernelMode,  
                &BytesCopied);  
       }  
   }  
   Peb = Process->;Peb;  

--- stage2.6 完成进程内核对象的创建-----
1. Audit the process creation  
```c
if (SeDetailedAuditingWithToken (NULL)) {  
       SeAuditProcessCreation (Process); // Process 为当前的 EPROCESS  
   }  
```
2. See if the parent has a job. If so reference the job and add the process in.  
3. 将 `EPROCESS` 插入的系统的 active process 链表中 [联想的断链和复制链]  
```c
   PspLockProcessList (CurrentThread);  
   InsertTailList (&PsActiveProcessHead, &Process->ActiveProcessLinks);  
   PspUnlockProcessList (CurrentThread);  
```
4. 线程创建的时间被设定，然后当线程的句柄可用的时候，返回到最初的调用者 `CreateProcess` 中 [`Kernel32.dll`]  

--- stage3 在进程中创建线程-----  
此时，进程对象的有关内容已经全部就绪。它就像一个空的茶壶，里面还没有茶水，所以什么都不能做。终于，`CreateProcess` [房主]在用 `PspCreateProcess` 创建了进程后 [茶壶]，继续调用 `NtCreateProcess` 创建线程 [往茶壶里倒茶水] 不过此时的茶水不凉的，它需要加热才会沸腾，才会有活力。即此时的线程创建后一直处于暂停状态，直到整个进程被完全的初始化完毕。  

`CreateProcess-->NtCreateProcess-->NtCreateProcessEx-->PspCreateProcess`, `PspCreateProcess`,它要做的事:  
*   进程对象中的线程计数增加  
*   `ETHREAD` 被创建并初始化  
*   线程 ID 被创建  
*   用户进程空间中的 `TEB` 被设置  
*   调用 `KeInitThread` 函数来设置 `KTHREAD` 结构  
*   创建线程时会检查权限.对于远程创建的线程,如果没有调试权限,则创建失败  
*   最终,线程完整,等待它的新生  

--- stage4 通知 WINDOWS 子系统已经创建了一个新进程-----  
这时候所有的进程和线程对象都已经继续. `Kernel32.dll` 发送消息到 WINDOWS 子系统,以便让它设置新的进程和线程.消息如下:  
*   进程和线程句柄  
*   `Creation Flags` 的入口  
*   父进程的 ID  
*   `Flag` 标志,表明此程序是否有窗体,以便 `CSRSS` 决定是否创建漏斗状的鼠标,表明有程序在后台运行  
*   The `Csrss thread block` is allocated and initialized. [没怎么看明白,`CSRSS` 咋还有新的线程块呢?]  
*   `CreateProcess` 在进程的线程表中插入新的线程  
*   进程的关闭等级设定为 `0x280`  
*   The new process block is inserted into the list of Windows subsystemwide processes. The per-process data structure used by the kernel-mode part of the Windows subsystem (`W32PROCESS` structure) is allocated and initialized. [没完全理解]  

--- stage6 在进程和线程的 context 里，完成地址空间的初始化[eg.加载需要的 DLL]，开始执行程序-----  
线程经过漫长的等待后终于开始了它的生命之旅,这要拜 `KiThreadStartup` 函数所赐.  
`KiThreadStartup`降低`IRQL`[从`DPC`到`APC`],然后调用`PspUserThreadStartup`函数.它再降低 `IRQL` 到`PASSIVE`.然后...  

最后, `PspUserThreadStartup` queues a user-mode `APC` to run the image loader initialization routine (`LdrInitializeThunk` in `Ntdll.dll`). The `APC` will be delivered when the thread attempts to return to user mode.  
```c
       // Queue the initial APC to the thread  
       KeRaiseIrql (APC_LEVEL, &OldIrql);  
       KiInitializeUserApc (PspGetBaseExceptionFrame (Thread),  
                  PspGetBaseTrapFrame (Thread),  
                  PspSystemDll.LoaderInitRoutine,  
                  NULL,  
                  PspSystemDll.DllBase,  
                  NULL);  
       KeLowerIrql (PASSIVE_LEVEL);  
```
当 `PspUserThreadStartup` 函数返回到 `KiThreadStartup` 中时,`APC` 已经被传送出去了,`LdrInitializeThunk` 也被调用 [All threads start with an `APC` at `LdrInitializeThunk`,它负责初始化 loader, heap manager, NLS 表, `TLS` (线程局部存储) 数组, critialsection 结构,然后加载任何需要的 `DLL`,并调用 `DLL` 中的代码]  

最终, 当 the loader initialization 返回到 user mode `APC` dispatcher 时,用户模式下的可执行文件开始真正的执行...  

WOW,终于把进程相关的内容大致的写完了。偶写的这些不单单是 `WINDWOS INTERNALS` 上的内容，还参考了前辈们的一些文章，当然还有 `WRK`、`OSREACT`！  

## 二> 线程相关  

线程的代表便是`ETHREAD`，它保存在系统地址空间中. `TEB` (线程环境块) 在用户进程空间中. `CSRSS` 为每个进程的所有线程保留一份类似的结构 (这么多进程,那得多少线程啊,不晓得这些备份的结构究竟是如何存储在 `CSRSS` 中的)；`Win32k.sys` [CSRSS 和内核交互的接口] 保留一份 `ETHREAD` 指向的 per-thread 数据结构 [`W32THREAD`]  

关键词：`ETHREAD`、`KTHREAD`、`TEB` [TLS/GDI/OpenGL]  
形象化：  

```text
             0x044  
  ETHREAD  ---------→EPROCESS  
 0x000↓                 ↓0x000  
  KTHREAD             KPROCESS     kernel mode  
 ---------------------------------------------------  
  0x020↓                |        user   mode  
        TEB                      |  
  0x30  ↓         0x1b0        |  
        PEB ←------------------------|  
```

`\_ETHREAD` 的有用信息很多，简单了解下  
```c
nt!\_ETHREAD  
  +0x000 Tcb             : \_KTHREAD // 利于系统调度和同步线程的结构体  
  ...  
  +0x1ec Cid             : \_CLIENT_ID  // 指向进程 ID  
  ...  
  +0x210 IrpList        : \_LIST_ENTRY  // I/O 信息 IRPS  
  +0x218 TopLevelIrp      : Uint4B  
  +0x21c DeviceToVerify   : Ptr32 \_DEVICE_OBJECT  
  +0x220 ThreadsProcess   : Ptr32 \_EPROCESS  // 指向进程的 EPROCESS  
  +0x224 StartAddress     : Ptr32 Void      // 线程的起始地址  
  +0x228 Win32StartAddress : Ptr32 Void   
  ...  
  +0x238 ThreadLock       : \_EX_PUSH_LOCK  
  ...  
  +0x248 SystemThread     : Pos 4, 1 Bit  
  ...  
```

① 与线程相关的函数  
![名称: Snap4.gif> 查看次数: 311> 文件大小:  28.7 KB](img/ebc16028f853e66954a7abcdbb909369.jpg)  

`CreateThread-->NtCreateThread-->PspCreateThread` [和创建进程同出一辙.这里不详细说明了.如果您有耐心看 `WRK` 的源码的话,相信会理解函数内部的很多东西]  

宏观上由 `CreateThread` [`Kernel32.dll`] 完成线程的创建：  
*   在进程的地址空间创建一个用户堆栈  
*   初始化线程的 hardware context  
```c
struct \_CONTEXT, 25 elements, 0x2cc bytes  
  +0x000 ContextFlags     : Uint4B  
  +0x004 Dr0              : Uint4B  
  +0x008 Dr1              : Uint4B  
  +0x00c Dr2              : Uint4B  
  +0x010 Dr3              : Uint4B  
  +0x014 Dr6              : Uint4B  
  +0x018 Dr7              : Uint4B  
  +0x01c FloatSave        : struct \_FLOATING_SAVE_AREA, 9 elements, 0x70 bytes  
  +0x08c SegGs            : Uint4B  
  +0x090 SegFs            : Uint4B  
  +0x094 SegEs            : Uint4B  
  +0x098 SegDs            : Uint4B  
  +0x09c Edi              : Uint4B  
  +0x0a0 Esi              : Uint4B  
  +0x0a4 Ebx              : Uint4B  
  +0x0a8 Edx              : Uint4B  
```



> +0x0ac Ecx : Uint4B
> +0x0b0 Eax : Uint4B
> +0x0b4 Ebp : Uint4B
> +0x0b8 Eip : Uint4B
> +0x0bc SegCs : Uint4B
> +0x0c0 EFlags : Uint4B
> +0x0c4 Esp : Uint4B
> +0x0c8 SegSs : Uint4B
> +0x0cc ExtendedRegisters : [512] UChar

> ■ 创建线程对象，通知 `CSRSS` 让它进一步设置
> ■ 返回线程句柄、ID 到调用者

> ②

> ---->抢占式的环境**量子时间**[`quantum`]

> 后面讲的都是关于线程优先级、线程调度和系统的算法问题，这些基本都是些原理性的东西，没必要翻译。而且 N 多关于操作系统的书上对这些问题都有非常详细的描述。偶就不扯淡了~~~

> ---->中断等级[`IRQL`] VS 线程优先级

> ![名称: Snap2.gif> 查看次数: 316> 文件大小: 10.4 KB](img/dc8aa6090c062160452e2aa1ee3b462a.jpg)

### <三> `Jobs`

`JOB` 内核对象有名字、可共享得到，用来控制一个或多个进程。把他们组合形成一个 `group`。它的主要功能是保证多个进程被统一管理操作。一个进程只能属于一个 `job` 对象。

> ![名称: Snap6.gif> 查看次数: 312> 文件大小: 20.6 KB](img/b6dc26b2e258345c6937af568c7ee728.jpg)

> 偶觉得 `job` 实在没啥好讲的,略过!

> ---------------------------<over><写的很仓促,不正确的地方还请指出>-----------------------

> [`hi.baidu.com/sudami/blog/item/f7262627f62e1700908f9d5c.html`](http://hi.baidu.com/sudami/blog/item/f7262627f62e1700908f9d5c.html)

`4dd466d0e1e0cc123afbe504da2462ca`
