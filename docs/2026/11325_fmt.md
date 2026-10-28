
# [原创]隐藏驱动，绕过 XueTr 0.39 检测

当系统加载一个驱动时，会为这个驱动建立一个 `_KLDR_DATA_TABLE_ENTRY` 结构体，`DRIVER_OBJECT` 结构体的 `DriverSection` 成员指向这个结构体。以下是 WRK 中 `_KLDR_DATA_TABLE_ENTRY` 结构体的定义：

```
typedef struct _KLDR_DATA_TABLE_ENTRY {
    LIST_ENTRY InLoadOrderLinks;
    PVOID ExceptionTable;
    ULONG ExceptionTableSize;
    // ULONG padding on IA64
    PVOID GpValue;
    PNON_PAGED_DEBUG_INFO NonPagedDebugInfo;
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
    // ULONG padding on IA64
    PVOID LoadedImports;
    PVOID PatchInformation;
} KLDR_DATA_TABLE_ENTRY, *PKLDR_DATA_TABLE_ENTRY;
```

其中 `PVOID DllBase;` 成员指明了驱动的加载基址；`UNICODE_STRING FullDllName;` 指明了驱动 `.sys` 文件的全路径；所有驱动的结构体通过 `InLoadOrderLinks` 成员链接起来，链表头部为 `PsLoadedModuleList`，因此可以通过遍历 `PsLoadedModuleList` 来得到所有加载的驱动。

在测试中发现，如果将某一驱动的 `DllBase` 或 `FullDllName.buffer` 填为 0，那么 `XueTr` 就不会显示这个驱动，以下是将 `DllBase` 填 0 时的验证代码：

```
/*
* 【作者：KiDebug】
* 【空间：b03K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2A6i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6w2K9f1c8W2j5Y4g2Y4i4K6u0r3i4@1f1K6i4K6R3H3i4K6V1I4
*/
#include <ntddk.h>

//偷懒版，完整版定义见WRK的_KLDR_DATA_TABLE_ENTRY
typedef struct _LDR_DATA_TABLE_ENTRY {
    LIST_ENTRY                InLoadOrderLinks;
    LIST_ENTRY                InMemoryOrderLinks;
    LIST_ENTRY                InInitializationOrderLinks;
    PVOID                     DllBase;
    PVOID                     EntryPoint;
    ULONG                     SizeOfImage;
    UNICODE_STRING            FullDllName;
    UNICODE_STRING            BaseDllName;
} LDR_DATA_TABLE_ENTRY, *PLDR_DATA_TABLE_ENTRY;

PVOID pDllBase;

VOID Reinitialize(PDRIVER_OBJECT DriverObject, PVOID Context, ULONG Count)
{
    ((PLDR_DATA_TABLE_ENTRY)DriverObject->DriverSection)->DllBase = NULL; //将DllBase填0
}

void testUnload(IN PDRIVER_OBJECT DriverObject)
{
    ((PLDR_DATA_TABLE_ENTRY)DriverObject->DriverSection)->DllBase = pDllBase; //恢复DllBase，以便驱动能顺利卸载
}

NTSTATUS DriverEntry(IN PDRIVER_OBJECT DriverObject, IN PUNICODE_STRING RegistryPath)
{
    DriverObject->DriverUnload = testUnload;
    pDllBase = ((PLDR_DATA_TABLE_ENTRY)DriverObject->DriverSection)->DllBase;
    IoRegisterDriverReinitialization(DriverObject, Reinitialize, NULL);
    return STATUS_SUCCESS;
}
```

用 `InstDrv.exe` 加载编译后的驱动 `test.sys`，依次点击安装、启动，然后用 `XueTr` 查看“驱动模块”，里面没有 `test.sys`；用 `IceSword.exe` 1.22 查看“内核模块”，可以看到 `test.sys` 的存在。

以下是将 `FullDllName.buffer` 填为 0 时的验证代码：

```
/*
* 【作者：KiDebug】
* 【空间：700K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2A6i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6w2K9f1c8W2j5Y4g2Y4i4K6u0r3i4@1f1K6i4K6R3H3i4K6V1I4
*/
#include <ntddk.h>

//偷懒版，完整版定义见WRK的_KLDR_DATA_TABLE_ENTRY
typedef struct _LDR_DATA_TABLE_ENTRY {
    LIST_ENTRY                InLoadOrderLinks;
    LIST_ENTRY                InMemoryOrderLinks;
    LIST_ENTRY                InInitializationOrderLinks;
    PVOID                     DllBase;
    PVOID                     EntryPoint;
    ULONG                     SizeOfImage;
    UNICODE_STRING            FullDllName;
    UNICODE_STRING            BaseDllName;
} LDR_DATA_TABLE_ENTRY, *PLDR_DATA_TABLE_ENTRY;

void testUnload(IN PDRIVER_OBJECT DriverObject)
{
}

NTSTATUS DriverEntry(IN PDRIVER_OBJECT DriverObject, IN PUNICODE_STRING RegistryPath)
{
    DriverObject->DriverUnload = testUnload;
    ((PLDR_DATA_TABLE_ENTRY)DriverObject->DriverSection)->FullDllName.Buffer = NULL;
    return STATUS_SUCCESS;
}
```

先用 `InstDrv.exe` 加载编译后的驱动 `test.sys`，依次点击安装、启动，然后双击打开 `IceSword.exe` 1.22，提示“程序初始化失败，错误代码1”，然后提示“无法初始化，程序退出”；双击打开 `PowerTool.exe` 3.6.2，蓝屏。

打开 `XueTr.exe` 0.39，查看“驱动模块”，显示有可疑驱动的存在，如果在 `WinDBG` 中手动将 `ntkrnlpa.exe` 对应的 `FullDllName.buffer` 填为 0，`XueTr` 不会显示 `ntkrnlpa.exe` 的存在。

**浓缩版：**

1. `DllBase` 填 0，无可疑驱动
2. `FullDllName.buffer` 填 0，有可疑驱动
3. `XueTr` 0.39
4. 没有建立 `DeviceObject`

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
