
# SSDT Hook 示例代码

```c
#include <ntddk.h>
#include <ntstatus.h>
```

## 系统服务表结构

```c
// 系统服务表
typedef struct _KSYSTEM_SERVICE_TABLE {
    PULONG ServiceTableBase;    // 函数地址表（SSDT）
    PULONG ServiceCounterTableBase;    // SSDT 函数被调用的次数
    ULONG NumberOfService;    // 函数个数
    PULONG ParamTableBase;    // 函数参数表（SSPT）
} KSYSTEM_SERVICE_TABLE, *PKSYSTEM_SERVICE_TABLE;

typedef struct _KSERVICE_TABLE_DESCRIPTOR {
    KSYSTEM_SERVICE_TABLE ntoskrnl;    // 内核函数
    KSYSTEM_SERVICE_TABLE win32k;    // win32k.sys 函数
    KSYSTEM_SERVICE_TABLE unUsed1;
    KSYSTEM_SERVICE_TABLE unUsed2;
} KSERVICE_TABLE_DESCRIPTOR, *PKSERVICE_TABLE_DESCRIPTOR;
```

## NTOPENPROCESS 类型定义

```c
// NTOPENPROCESS
typedef NTSTATUS (*NTOPENPROCESS) (PHANDLE ProcessHandle, ACCESS_MASK DesiredAccess, POBJECT_ATTRIBUTES ObjectAttributes, PCLIENT_ID ClientId);
```

## 函数声明

```c
VOID DriverUnload(PDRIVER_OBJECT pDriver);
NTSTATUS DriverEntry(PDRIVER_OBJECT pDriver, PUNICODE_STRING reg_path);
VOID PageProtectOff();
VOID PageProtectOn();
VOID HookNtOpenProcess();
VOID UnHookNtOpenProcess();
NTSTATUS HbgNtOpenProcess(PHANDLE ProcessHandle, ACCESS_MASK DesiredAccess, POBJECT_ATTRIBUTES ObjectAttributes, PCLIENT_ID ClientId);
```

## 全局变量

```c
extern PKSERVICE_TABLE_DESCRIPTOR KeServiceDescriptorTable;    // ntoskrnl.exe 导出的全局变量
ULONG uOldNtOpenProcess;    // 旧的函数地址
```

## 驱动入口

```c
// 驱动入口
NTSTATUS DriverEntry(PDRIVER_OBJECT pDriver, PUNICODE_STRING reg_path)
{
    // HOOK
    HookNtOpenProcess();

    pDriver->DriverUnload = DriverUnload;

    return STATUS_SUCCESS;
}
```

## 卸载驱动

```c
// 卸载驱动
VOID DriverUnload(PDRIVER_OBJECT pDriver)
{
    UnHookNtOpenProcess();
    DbgPrint("Driver unloaded.\n");
}
```

## 页保护控制

```c
// 关闭页保护
VOID PageProtectOff()
{
    __asm
    {
        cli;    // 关闭中断
        mov eax, cr0;
        and eax, not 0x10000;    // WP位置0
        mov cr0, eax;
    }
}

// 开启页保护
VOID PageProtectOn()
{
    __asm
    {
        mov eax, cr0;
        or eax, 0x10000;    // WP位置1
        mov cr0, eax;
        sti;    // 恢复中断
    }
}
```

## Hook NtOpenProcess

```c
// HOOK NtOpenProcess
// 关闭页面保护
// 对未进行SSDT Hook前的SSDT表进行了备份，一个索引号对应一个当前地址。
// SSDT HOOK
// 开启页面保护
VOID HookNtOpenProcess()
{
    PageProtectOff();
    uOldNtOpenProcess = KeServiceDescriptorTable->ntoskrnl.ServiceTableBase[0x7A];
    KeServiceDescriptorTable->ntoskrnl.ServiceTableBase[0x7A] = (ULONG)HbgNtOpenProcess;
    PageProtectOn();
}
```

## UnHook NtOpenProcess

```c
// UnHOOK NtOpenProcess
// 关闭页面保护
// 通过之前保存的地址，恢复SSDT表
// 开启页面保护
VOID UnHookNtOpenProcess()
{
    PageProtectOff();
    KeServiceDescriptorTable->ntoskrnl.ServiceTableBase[0x7A] = uOldNtOpenProcess;
    PageProtectOn();
}
```

## 被修改的 NtOpenProcess 函数

```c
// 被修改的 NtOpenProcess 函数，简单打印参数
NTSTATUS HbgNtOpenProcess(PHANDLE ProcessHandle, ACCESS_MASK DesiredAccess, POBJECT_ATTRIBUTES ObjectAttributes, PCLIENT_ID ClientId)
{
    DbgPrint("%x %x %x %x\n", ProcessHandle, DesiredAccess, ObjectAttributes, ClientId);
    return ((NTOPENPROCESS)uOldNtOpenProcess)(ProcessHandle, DesiredAccess, ObjectAttributes, ClientId);
}
```

只做笔记整理。代码非原创。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2021-9-29 20:52

被pyikaaaa编辑，原因：改标题

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
