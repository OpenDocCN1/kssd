
# ProcessExplorer 原理分析之句柄处理

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-24.htm`](https://www.pediy.com/kssd/pediy09/pediy09-24.htm)

> ProcessExplorer 原理分析之句柄处理 by sucsor/RCT

### 1. 如何获得各进程的句柄

使用 `NtQuerySystemInformation` 函数的 `SystemHandleInformation=16` 号功能。

其相关结构定义如下：

```
typedef struct _SYSTEM_HANDLE_TABLE_ENTRY_INFO{
    USHORT UniqueProcessId;
    USHORT CreatorBackTraceIndex;
    UCHAR ObjectTypeIndex;
    UCHAR HandleAttributes;
    USHORT HandleValue;
    PVOID Object;
    ULONG GrantedAccess;
} SYSTEM_HANDLE_TABLE_ENTRY_INFO, *PSYSTEM_HANDLE_TABLE_ENTRY_INFO;

typedef struct _SYSTEM_HANDLE_INFORMATION{
    ULONG NumberOfHandles;
    SYSTEM_HANDLE_TABLE_ENTRY_INFO Handles[1];
} SYSTEM_HANDLE_INFORMATION, *PSYSTEM_HANDLE_INFORMATION;
```

该功能号获取系统内所有进程的句柄放在 `Handles` 里，个数由 `NumberOfHandles` 标识，每个句柄由 `UniqueProcessId` 来区分属于那个不同的进程。

### 2. 如何得到句柄的信息

首先 `ProcessExplorer` 要打开该进程(`OpenProcess`)，然后根据使用 `DuplicateHandle`，将目标进程的句柄和要关闭的句柄(这不是唯一的办法，不过 `PE` 是这样做的) 做为参加传入，得到该句柄执行体对象的在本进程内的句柄。然后通过 `DeviceIoControl` 将该句柄传到 `ProcessExplorer` 的驱动中，通过使用 `PsLookupProcessByProcessId` 得到进程有内核对象，然后使用 `KeAttachProcess` 函数切换到进程的上下文中，再通过使用 `ObReferenceObjectByHandle` 得到对象，再通过 `ObQueryNameString` 得到对象的名称信息。根据对象的结构，还可以得到其他的相关信息，比如打开的句柄数和引用计数及一些访问控制信息。

### 3. 如何关闭某进程中的句柄

如同二中提到的通过一系统的函数可以切换到进程的上下文，在该进程上下文中，即可调用 `ZwClose` 来关闭本进程的句柄，需要的参数只是句柄，这个句柄在第一步中已经得到。

### 4. 其他说明

`ObQueryNameString` 并不是对于所有的对象都能得到名称，对于文件对象的名称可能需要进一步使用文件系统提供的相关函数来获取名称信息。

### 5. 结语

有误之外请各位指正，谢谢。

`ProcessExplorer` 逆向中，不过总体来说只要代码思路有了，相信大家一看便知。

如果可能的话，会将代码按部分贴出来。只是可能...:)

`4dd466d0e1e0cc123afbe504da2462ca`
