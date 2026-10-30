
```
typedef struct {

    ULONG                           Version;

    pHalQuerySystemInformation      HalQuerySystemInformation;

    pHalSetSystemInformation        HalSetSystemInformation;

    pHalQueryBusSlots               HalQueryBusSlots;

    ULONG                           Spare1;

    pHalExamineMBR                  HalExamineMBR;

    pHalIoReadPartitionTable        HalIoReadPartitionTable;

    pHalIoSetPartitionInformation   HalIoSetPartitionInformation;

    pHalIoWritePartitionTable       HalIoWritePartitionTable;



    pHalHandlerForBus               HalReferenceHandlerForBus;

    pHalReferenceBusHandler         HalReferenceBusHandler;

    pHalReferenceBusHandler         HalDereferenceBusHandler;



    pHalInitPnpDriver               HalInitPnpDriver;

    pHalInitPowerManagement         HalInitPowerManagement;



    pHalGetDmaAdapter               HalGetDmaAdapter;

    pHalGetInterruptTranslator      HalGetInterruptTranslator;



    pHalStartMirroring              HalStartMirroring;

    pHalEndMirroring                HalEndMirroring;

    pHalMirrorPhysicalMemory        HalMirrorPhysicalMemory;

    pHalEndOfBoot                   HalEndOfBoot;

    pHalMirrorVerify                HalMirrorVerify;



    pHalGetAcpiTable                HalGetCachedAcpiTable;

    pHalSetPciErrorHandlerCallback  HalSetPciErrorHandlerCallback;



#if defined(_IA64_)

    pHalGetErrorCapList             HalGetErrorCapList;

    pHalInjectError                 HalInjectError;

#endif



} HAL_DISPATCH, *PHAL_DISPATCH;
```

根据该结构体的定义，可以看到`HalQuerySystemInformation`函数就保存在`HAL_DISPATCH`结构体偏移为0x04的地址。而`HalDispatchTable`是从内核模块中导出的，所以就需要找到内核模块在内存中的基地址，再根据偏移得到`HalDispatchTable`的VA。而要找到内核模块的基地址就需要用到`ZwQuerySystemInformation`这个API。该函数的定义如下：

```
NTSTATUS WINAPI ZwQuerySystemInformation(

  __in       SYSTEM_INFORMATION_CLASS SystemInformationClass,

  __inout    PVOID SystemInformation,

  __in       ULONG SystemInformationLength,

  __out_opt  PULONG ReturnLength);
```

| 参数 | 含义 |
| --- | --- |
| `SystemInformationClass` | 要检索的类型。是一个`SYSTEM_INFORMATION_CLASS`的联合体 |
| `SystemInformation` | 指向缓冲区的指针，用于接收请求信息。该信息的大小和结构取决于`SystemInformationClass` |
| `SystemInformationLength` | `SystemInformation`参数指向的缓冲区的大小 |
| `ReturnLength` | 指向函数写入所请求信息的实际大小的位置的可选指针。如果该大小小于或等于`SystemInformationLength`参数，则函数将信息复制到`SystemInformation`缓冲区中；否则，它将返回`NTSTATUS`错误代码，并以`ReturnLength`的形式返回接收请求信息所需的缓冲区大小。 |

而`SYSTEM_INFORMATION_CLASS`，在文档中的部分内容如下：

```
typedef enum _SYSTEM_INFORMATION_CLASS {

	SystemInformationClassMin = 0,

	SystemBasicInformation = 0,

	SystemProcessInformation = 5,

	SystemProcessesAndThreadsInformation = 5,

	SystemModuleInformation = 11,

	SystemExceptionInformation = 33,

	SystemKernelDebuggerInformation = 35,

} SYSTEM_INFORMATION_CLASS;
```

当该值指定的是`SystemModuleInformation`(11)的时候，`SystemInformation`返回的就是`SYSTEM_MODULE_INFORMATION`的指针。该结构体的定义如下：

```
typedef struct _SYSTEM_MODULE_INFORMATION_ENTRY {

	ULONG	 Unknown1;

	ULONG	 Unknown2;

	PVOID  Base;

	ULONG  Size;

	ULONG  Flags;

	USHORT  Index;

  /* Length of module name not including the path, this

     field contains valid value only for NTOSKRNL module */

	USHORT	NameLength;

	USHORT  LoadCount;

	USHORT  PathLength;

	CHAR  ImageName[256];

} SYSTEM_MODULE_INFORMATION_ENTRY, *PSYSTEM_MODULE_INFORMATION_ENTRY;



typedef struct _SYSTEM_MODULE_INFORMATION {

  ULONG  Count;

  SYSTEM_MODULE_INFORMATION_ENTRY Module[1];

} SYSTEM_MODULE_INFORMATION, *PSYSTEM_MODULE_INFORMATION;
```

可以看到`SYSTEM_MODULE_INFORMATION`中保存了`SYSTEM_MODULE_INFORMATION_ENTRY`的数组。数组中的每一个元素都保存了一个模块的信息，其中的`Base`保存模块的加载地址，`ImageName`保存了模块的名称。而本次查询一共有多少个`SYSTEM_MODULE_INFORMATION`的数组则保存在`Count`中。

由于一开始并不知道要申请多大的内存才可以保存这些数据，所以需要调用两次函数，第一次调用的目的就是通过将第三个参数传入0来获得需要使用的内存大小。

查询到的内核模块中的第一个就是要使用的内核模块。这样就可以获得这个内核模块的名称，接下来就需要使用`LdrLoadDll`来将内核模块导入，该函数的定义如下：

```
#define IMP_SYSCALL __declspec(dllimport) NTSTATUS __stdcall



IMP_SYSCALL

 LdrLoadDll(IN PWSTR DllPath OPTIONAL,

 	     IN PULONG DllCharacteristics OPTIONAL,

 	     IN PUNICODE_STRING DllName,

        OUT PVOID *DllHandle);
```

| 参数 | 含义 |
| --- | --- |
| `DllPath` | 可选，指定要加载的DLL的路径 |
| `DllCharacteristics` | 可选，指向要加载的DLL的属性 |
| `DllName` | 指向要加载的DLL的名称 |
| `DllHandle` | 用来接收得到的DLL的句柄 |

通过这个函数就可以将模块加载到内存中算出`HalDispatchTable`的偏移，再使用上面得到的`ImageBase`就可以计算出`HalDispatchTable`在内核中的具体位置，接下来就通过IO控制码发送消息给驱动完成修改。

## 3.调用`HalQuerySystemInformation`函数

要调用这个函数，只需要在调用`NtQueryIntervalProfile`函数的时候，它的第一个参数传入的值不等于`ProfileTime`和`ProfileAlignmentFixup`就好。

# 四.ShellCode的提权原理

首先要知道0xFFDFF000这个地址保存的是`_KPCR`，该结构体的定义如下：

```
kd> dt _KPCR

nt!_KPCR

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

   +0x044 MajorVersion     : Uint2B

   +0x046 MinorVersion     : Uint2B

   +0x048 SetMember        : Uint4B

   +0x04c StallScaleFactor : Uint4B

   +0x050 DebugActive      : UChar

   +0x051 Number           : UChar

   +0x052 Spare0           : UChar

   +0x053 SecondLevelCacheAssociativity : UChar

   +0x054 VdmAlert         : Uint4B

   +0x058 KernelReserved   : [14] Uint4B

   +0x090 SecondLevelCacheSize : Uint4B

   +0x094 HalReserved      : [16] Uint4B

   +0x0d4 InterruptMode    : Uint4B

   +0x0d8 Spare1           : UChar

   +0x0dc KernelReserved2  : [17] Uint4B

   +0x120 PrcbData         : _KPRCB
```

该结构体偏移0x120处保存的是`_KPRCB`结构体，该结构体的部分定义如下：

```
kd> dt _KPRCB

ntdll!_KPRCB

   +0x000 MinorVersion     : Uint2B

   +0x002 MajorVersion     : Uint2B

   +0x004 CurrentThread    : Ptr32 _KTHREAD

   +0x008 NextThread       : Ptr32 _KTHREAD

   +0x00c IdleThread       : Ptr32 _KTHREAD

   +0x010 Number           : Char
```

据此可以知道0xFFDFF124保存的是当前线程的`_KTHREAD`，而`_KTHREAD`又是`_ETHREAD`结构体中的第一个成员，所以0xFFDFF124保存的其实是当前`_ETHREAD`结构体。`_ETHREAD`结构体中保存的内容如下：

```
kd> dt _ETHREAD

ntdll!_ETHREAD

   +0x000 Tcb              : _KTHREAD

   +0x1c0 CreateTime       : _LARGE_INTEGER

   +0x1c0 NestedFaultCount : Pos 0, 2 Bits

   +0x1c0 ApcNeeded        : Pos 2, 1 Bit

   +0x1c8 ExitTime         : _LARGE_INTEGER

   +0x1c8 LpcReplyChain    : _LIST_ENTRY

   +0x1c8 KeyedWaitChain   : _LIST_ENTRY

   +0x1d0 ExitStatus       : Int4B

   +0x1d0 OfsChain         : Ptr32 Void

   +0x1d4 PostBlockList    : _LIST_ENTRY

   +0x1dc TerminationPort  : Ptr32 _TERMINATION_PORT

   +0x1dc ReaperLink       : Ptr32 _ETHREAD

   +0x1dc KeyedWaitValue   : Ptr32 Void

   +0x1e0 ActiveTimerListLock : Uint4B

   +0x1e4 ActiveTimerListHead : _LIST_ENTRY

   +0x1ec Cid              : _CLIENT_ID

   +0x1f4 LpcReplySemaphore : _KSEMAPHORE

   +0x1f4 KeyedWaitSemaphore : _KSEMAPHORE

   +0x208 LpcReplyMessage  : Ptr32 Void

   +0x208 LpcWaitingOnPort : Ptr32 Void

   +0x20c ImpersonationInfo : Ptr32 _PS_IMPERSONATION_INFORMATION

   +0x210 IrpList          : _LIST_ENTRY

   +0x218 TopLevelIrp      : Uint4B

   +0x21c DeviceToVerify   : Ptr32 _DEVICE_OBJECT

   +0x220 ThreadsProcess   : Ptr32 _EPROCESS

   +0x224 StartAddress     : Ptr32 Void

   +0x228 Win32StartAddress : Ptr32 Void
```

可以看到`_ETHREAD`偏移0x220的地址，保存的是线程对应的`_EPROCESS`结构体，该结构体的部分成员如下
