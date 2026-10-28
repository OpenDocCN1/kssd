
# [原创] 优雅的动态卸载键盘过滤驱动

在参考《寒江独钓 Windows内核安全编程》的键盘过滤驱动时，其键盘过滤驱动的卸载方式是通过调用`KeDelayExecutionThread`函数实现卸载函数所属的线程等待，在这期间通过按键处理掉键盘处于挂起状态的 `IRP_MJ_READ` 请求，从而顺利卸载键盘过滤驱动。

但是这种方法不够优雅，在了解到Windows I/O机制的移除锁时，想到通过该锁实现在卸载键盘过滤驱动时根据`irp`动态阻塞。这样的话，在某个时刻发生键盘按键事件时便可动态卸载。

1. `IoReleaseRemoveLockAndWait` 例程释放了驱动程序在之前调用 `IoAcquireRemoveLock` 中获取的删除锁，并等到释放锁的所有获取为止。

2. 由于 `IoReleaseRemoveLockAndWait` 工作在 `PASSIVE_LEVEL`，因此只能使用系统线程或工作项。

```

IoAcquireRemoveLock(&io_remove_lock, irp);

IoCopyCurrentIrpStackLocationToNext(irp);

IoSetCompletionRoutine(irp, c2pReadComplete, device_object_physics, TRUE, TRUE, TRUE);    // 此处设置了回调函数

//IoSkipCurrentIrpStackLocation(irp);

status = IoCallDriver(device_object_physics, irp);

IoReleaseRemoveLock(&io_remove_lock, irp);    // 移除锁的释放可在此处，也可以在回调函数中实现

return status;

/\* 此处为键盘过滤驱动的卸载函数 \*/

PIO_WORKITEM io_workitem = IoAllocateWorkItem(driver->DeviceObject);

IoQueueWorkItem(io_workitem, IoWorkitemRoutine, DelayedWorkQueue, driver);

IoFreeWorkItem(io_workitem);

/\* 工作项的回调例程 \*/

void IoWorkitemRoutine(

PDEVICE_OBJECT DeviceObject,

PVOID Context

)

{

PDRIVER_OBJECT driver = (PDRIVER_OBJECT)Context;

KIRQL irql = KeGetCurrentIrql();    // irql为PASSIVE_LEVEL

DbgPrintEx(DPFLTR_IHVDRIVER_ID, DPFLTR_ERROR_LEVEL, "keyboard driver filter unload...\n");

IoDetachDevice(device_object_low);

IoAcquireRemoveLock(&io_remove_lock, "quit");

IoReleaseRemoveLockAndWait(&io_remove_lock, "quit");

        IoDeleteDevice(driver->DeviceObject);

DbgPrintEx(DPFLTR_IHVDRIVER_ID, DPFLTR_ERROR_LEVEL, "[+] 工作项回调函数执行. irql:%d\n", irql);

}
```

![](https://bbs.kanxue.com/upload/attach/202305/832354_NYWCN96VZWQNHMY.jpg)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
