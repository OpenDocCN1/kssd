
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
