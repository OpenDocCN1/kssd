
# 一句话让 XueTr 卸载不了我们的驱动（不知道这会不会是火星或抄袭了…）

> 原文：[`www.pediy.com/kssd/pediy12/134090.html`](https://www.pediy.com/kssd/pediy12/134090.html)

一句话让 `XueTr` 卸载不了我们的驱动（不知道这会不会是火星或抄袭了…）

`MJ` 语录"`inline` hook.,非标准链式 `hook`（各自管理各自的）不存在绝对的安全 有很大几率蓝屏，所以唯一正确的办法，就是不要卸载 "

`http://www.debugman.com/discussion/1542/`

来自：第三届 360 软件大赛--第二阶段题目的解答

驱动代码如下：

```c
/*
 * 作者：KiDebug
 * 空间：http://hi.baidu.com/KiDebug/
 */
#include <ntddk.h>

void testUnload(IN PDRIVER_OBJECT DriverObject)
{
}

NTSTATUS testDefaultHandler(IN PDEVICE_OBJECT DeviceObject, IN PIRP Irp)
{
  Irp->IoStatus.Status = STATUS_NOT_SUPPORTED;
  Irp->IoStatus.Information = 0;
  IoCompleteRequest(Irp, IO_NO_INCREMENT);
  return Irp->IoStatus.Status;
}

NTSTATUS DriverEntry(IN PDRIVER_OBJECT DriverObject, IN PUNICODE_STRING RegistryPath)
{
  ULONG i;
  for (i = 0; i <= IRP_MJ_MAXIMUM_FUNCTION; i++)
    DriverObject->MajorFunction[i] = testDefaultHandler;
  DriverObject->DriverUnload = testUnload;
  return STATUS_SUCCESS;
}
```

驱动卸载时，函数调用如下：

```
kd> kp

ChildEBP RetAddr   
ee5deb30 805b1bde nt!IopDeleteDriver
ee5deb4c 80523bf1 nt!ObpRemoveObjectRoutine+0xe0
ee5deb70 804f5778 nt!ObfDereferenceObject+0x5f
ee5dec14 8057a83d nt!IopUnloadDriver+0x28a
ee5dec24 8053e6d8 nt!NtUnloadDriver+0xf
ee5dec24 80500231 nt!KiFastCallEntry+0xf8
ee5deca0 804f55df nt!ZwUnloadDriver+0x11
ee5ded48 8057a83d nt!IopUnloadDriver+0xf1
ee5ded58 8053e6d8 nt!NtUnloadDriver+0xf
```

在 `nt!IopDeleteDriver` 中，有如下的判断代码（`WRK`，`/base/ntos/io/iomgr/objsup.c` 787 行）：

```c
    if (driverObject->DriverSection != NULL) {
        //
        // Make sure any DPC's that may be running inside the driver have completed
        //
        KeFlushQueuedDpcs ();
        MmUnloadSystemImage( driverObject->DriverSection );
        PpDriverObjectDereferenceComplete(driverObject);
    }
```

如果 `driverObject->DriverSection` 不为空的话，就会调用 `MmUnloadSystemImage` 把驱动映象从内核中卸掉

如果 `driverObject->DriverSection` 为空的话呢？

那当然就不会把把驱动映象从内核中卸掉了，驱动仍然在内核中，该干嘛干嘛

所以我们只要在驱动的 `DriverUnload` 函数里面添加一句代码就行：

```c
/*
 * 作者：KiDebug
 * 空间：http://hi.baidu.com/KiDebug/
 */
#include <ntddk.h>

void testUnload(IN PDRIVER_OBJECT DriverObject)
{
  DriverObject->DriverSection=NULL;
}

NTSTATUS testDefaultHandler(IN PDEVICE_OBJECT DeviceObject, IN PIRP Irp)
{
  Irp->IoStatus.Status = STATUS_NOT_SUPPORTED;
  Irp->IoStatus.Information = 0;
  IoCompleteRequest(Irp, IO_NO_INCREMENT);
  return Irp->IoStatus.Status;
}

NTSTATUS DriverEntry(IN PDRIVER_OBJECT DriverObject, IN PUNICODE_STRING RegistryPath)
{
  ULONG i;
  for (i = 0; i <= IRP_MJ_MAXIMUM_FUNCTION; i++)
    DriverObject->MajorFunction[i] = testDefaultHandler;
  DriverObject->DriverUnload = testUnload;
  return STATUS_SUCCESS;
}
```

用 `InstDrv.exe` 加载编译后的驱动，依次点击安装、启动、停止、卸载，然后用 `XueTr` 测试一下，发现虽然能显示 `test.sys` 的存在，但菜单里面“卸载驱动(危险)”已经变灰，无法点击了。

虽然是自己在做题时根据 `MJ` 的语录翻的 `WRK`，不知道上面这文章会不会是火星或抄袭了…还请大家指正。。
