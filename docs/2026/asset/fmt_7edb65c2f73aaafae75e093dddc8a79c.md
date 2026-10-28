
# [原创]强制关闭进程的两种方法

## 一.前言

实验平台是`WIN XP sp3 x86`。

要使用两种不同的方法来强制关闭进程。

## 二.通过PspTerminateProcess来关闭进程

在关闭进程的时候，`PspTerminateProcess`是一个比较底层的`API`。通过调用它来关闭进程可以绕过一些保护，该函数的定义如下

```
NTSTATUS 

PspTerminateProcess(

    EPROCESS pEprocess,

    NTSTATUS ExitCode);
```

|  |  |

| --- | --- |

| **参数** | **含义** |

| pEprocess | 要关闭的进程的`EPROCESS` |

| ExitCode | 进程的退出码 |

由于这个函数是未导出的，所以不能直接调用，要先在内存中找到这个函数才可以调用。这里选择找到这个函数的办法是通过内核模块遍历的办法来查找这个函数。

我们知道每一个内核模块都有一个`DRIVER_OBJECT`，该结构体的定义如下

```
typedef struct _DRIVER_OBJECT {

    CSHORT Type;

    CSHORT Size;

    PDEVICE_OBJECT DeviceObject;

    ULONG Flags;

    PVOID DriverStart;

    ULONG DriverSize;

    PVOID DriverSection;

    PDRIVER_EXTENSION DriverExtension;

    UNICODE_STRING DriverName;

    PUNICODE_STRING HardwareDatabase;

    PFAST_IO_DISPATCH FastIoDispatch;

    PDRIVER_INITIALIZE DriverInit;

    PDRIVER_STARTIO DriverStartIo;

    PDRIVER_UNLOAD DriverUnload;

    PDRIVER_DISPATCH MajorFunction[IRP_MJ_MAXIMUM_FUNCTION + 1];

} DRIVER_OBJECT;

typedef struct _DRIVER_OBJECT *PDRIVER_OBJECT;
```

其中的第7个成员`DriverSection`，该成员指向的是一个`LDR_DATA_TABLE_ENTRY`，该结构体的定义如下

```
kd> dt _LDR_DATA_TABLE_ENTRY

nt!_LDR_DATA_TABLE_ENTRY

   +0x000 InLoadOrderLinks : _LIST_ENTRY

   +0x008 InMemoryOrderLinks : _LIST_ENTRY

   +0x010 InInitializationOrderLinks : _LIST_ENTRY

   +0x018 DllBase          : Ptr32 Void

   +0x01c EntryPoint       : Ptr32 Void

   +0x020 SizeOfImage      : Uint4B

   +0x024 FullDllName      : _UNICODE_STRING

   +0x02c BaseDllName      : _UNICODE_STRING

   +0x034 Flags            : Uint4B

   +0x038 LoadCount        : Uint2B

   +0x03a TlsIndex         : Uint2B

   +0x03c HashLinks        : _LIST_ENTRY

   +0x03c SectionPointer   : Ptr32 Void

   +0x040 CheckSum         : Uint4B

   +0x044 TimeDateStamp    : Uint4B

   +0x044 LoadedImports    : Ptr32 Void

   +0x048 EntryPointActivationContext : Ptr32 Void

   +0x04c PatchInformation : Ptr32 Void
```

本实验要用到的几个关键成员

|  |  |

| --- | --- |

| **成员** | **含义** |

| InLoadOrderLinks | 双向链表，指向上一个或者下一个`LDR_DATA_TABLE_ENTRY`结构 |

| DllBase | 该模块加载的基地址 |

| SizeOfImage | 模块的大小 |

由于这个双向链表的存在，就可以找到内存中所有内核模块，在从这些内核模块中一一通过特征码匹配的方式就可以找到`PspTerminateProcess`函数地址。接下来就要从`PspTerminateProcess`中找到特征码了，首先查看`PspTerminateProcess`函数的内容

```
kd> u PspTerminateProcess L30

nt!PspTerminateProcess:

805d3482 8bff            mov     edi,edi

805d3484 55              push    ebp

805d3485 8bec            mov     ebp,esp

805d3487 56              push    esi

805d3488 64a124010000    mov     eax,dword ptr fs:[00000124h]

805d348e 8b7508          mov     esi,dword ptr [ebp+8]

805d3491 3b7044          cmp     esi,dword ptr [eax+44h]

805d3494 7507            jne     nt!PspTerminateProcess+0x1b (805d349d)

805d3496 b80d0000c0      mov     eax,0C000000Dh

805d349b eb5a            jmp     nt!PspTerminateProcess+0x75 (805d34f7)

805d349d 57              push    edi

805d349e 8dbe48020000    lea     edi,[esi+248h]

805d34a4 f6470120        test    byte ptr [edi+1],20h

805d34a8 7412            je      nt!PspTerminateProcess+0x3a (805d34bc)

805d34aa 8d8674010000    lea     eax,[esi+174h]

805d34b0 50              push    eax

805d34b1 56              push    esi

805d34b2 6854345d80      push    offset nt!NtTerminateProcess+0x14c (805d3454)

805d34b7 e8f2eeffff      call    nt!PspCatchCriticalBreak (805d23ae)

805d34bc 6a08            push    8

805d34be 58              pop     eax

805d34bf f00907          lock or dword ptr [edi],eax

805d34c2 6a00            push    0

805d34c4 56              push    esi

805d34c5 e8ca560000      call    nt!PsGetNextProcessThread (805d8b94)

805d34ca 8bf8            mov     edi,eax

805d34cc 85ff            test    edi,edi

805d34ce 741e            je      nt!PspTerminateProcess+0x6c (805d34ee)

805d34d0 ff750c          push    dword ptr [ebp+0Ch]

805d34d3 57              push    edi

805d34d4 e807fdffff      call    nt!PspTerminateThreadByPointer (805d31e0)

805d34d9 57              push    edi

805d34da 56              push    esi

805d34db e8b4560000      call    nt!PsGetNextProcessThread (805d8b94)

805d34e0 8bf8            mov     edi,eax

805d34e2 85ff            test    edi,edi

805d34e4 75ea            jne     nt!PspTerminateProcess+0x4e (805d34d0)

805d34e6 3986bc000000    cmp     dword ptr [esi+0BCh],eax

805d34ec 7406            je      nt!PspTerminateProcess+0x72 (805d34f4)

805d34ee 56              push    esi

805d34ef e86e05ffff      call    nt!ObClearProcessHandleTable (805c3a62)

805d34f4 33c0            xor     eax,eax

805d34f6 5f              pop     edi

805d34f7 5e              pop     esi

805d34f8 5d              pop     ebp

805d34f9 c20800          ret     8

805d34fc cc              int     3

805d34fd cc              int     3
```

从中挑选如下代码

```
805d3487 56              push    esi

805d3488 64a124010000    mov     eax,dword ptr fs:[00000124h]

805d348e 8b7508          mov     esi,dword ptr [ebp+8]

805d3491 3b7044          cmp     esi,dword ptr [eax+44h]

805d3494 7507            jne     nt!PspTerminateProcess+0x1b (805d349d)

805d3496 b80d0000c0      mov     eax,0C000000Dh

805d349b eb5a            jmp     nt!PspTerminateProcess+0x75 (805d34f7)

805d349d 57              push    edi
```

就可以提取出相应的特征码如下

```
UCHAR szSpecialCode[] = {0x56, 0x64, 0xA1, 0x24, 0x01, 0x00, 0x00, 0x8B, 

		     0x75, 0x08, 0x3B, 0x70, 0x44, 0x75, 0x07, 0xB8, 

			 0x0D, 0x00, 0x00, 0xC0, 0xEB, 0x5A, 0x57 };
```

接下来要做的就是遍历内核模块在用这个特征码找到相应的函数地址，随后在调用该函数来关闭进程
