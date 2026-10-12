
# 学习内核写的 hook 库，支持内核中常用的几种 hook

> 原文：[`www.pediy.com/kssd/pediy12/132418.html`](https://www.pediy.com/kssd/pediy12/132418.html)

学习内核写的 `hook` 库，共享出来，方便大家使用。

支持内核中常用的几种 `hook`。

```cpp
#include "HookEngine.h"

//inlinehook 相关

VOID InLineRotine()
{
    KdPrint(("你好\\r\\n"));
}

VOID DriverUnload(IN PDRIVER_OBJECT pDriverObj);

NTSTATUS DriverEntry(IN PDRIVER_OBJECT pDriverObj, IN PUNICODE_STRING pRegistryString)
{
    NTSTATUS status = STATUS_SUCCESS;
    UNICODE_STRING ustrLinkName;
    UNICODE_STRING ustrDevName;
    PDEVICE_OBJECT pDevObj;

    pDriverObj->DriverUnload = DriverUnload;

    HookEngine_Init();

    //Inlinehook 方式
    {
        HookEngine_Hook(HT_INLINE_JMP_HOOK, SSDTHook_GetFuncAddr(122), (ULONG)InLineRotine);
    }

    return STATUS_SUCCESS;
}

VOID DriverUnload(IN PDRIVER_OBJECT pDriverObj)
{
    HookEngine_Destroy();
    return;
}
```

特意做了个 `chm` 帮助文件![](img/081868d20f20fe7a9b4b88ab0d0d33c6.jpg "狂笑")

## 上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) | | `HookEngine.rar` |
| :--- | :--- | :--- |
