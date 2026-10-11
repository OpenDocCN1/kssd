
# 向非 `ATTACH` 的 `DEVICE` 转发 `IRP` (`VIRTUALDISK`)

> 原文：[`www.pediy.com/kssd/pediy10/80484.html`](https://www.pediy.com/kssd/pediy10/80484.html)

我的 NO.2 个贴子,发现很多人都关心驱动了，`HOOK` 这个内核什么的其实没什么意思，用 `DBG` 调试可以找到很多 `HOOK` 点，还有那些照抄 `WINDOWS` 的源码帖子，没意思。创新点都没有，我是很少发帖子，但要发就发精华，全球没第二家。

```c
NTSTATUS IoCompletion(
    IN PDEVICE_OBJECT DeviceObject,
    IN PIRP Irp,
    IN PVOID Context
)
{
    PKEVENT hEvent = (PKEVENT)Context;
    if( Irp->Flags & SL_PENDING_RETURNED)
    {
        IoMarkIrpPending( Irp );
    };
    KeSetEvent( hEvent, IO_NO_INCREMENT, FALSE );
    /*自己释放 IRP*/
    return STATUS_MORE_PROCESSING_REQUIRED;
}

VirtualDiskCommonControl(
    IN PDEVICE_OBJECT DeviceObject,
    IN PIRP Irp
)
{
    PIO_STACK_LOCATION iostack,newiostack;
    IO_STATUS_BLOCK iostatus;
    PIRP newirp;
    NTSTATUS ok;
    IRP backIrp;
    KEVENT CompleteEvent;
    DEVICE_OBJECT lodev;
    DbgPrint( "Irp------------\n");
    EnumPrintIrpStack( Irp );
    memcpy( &lodev, LowerDevice, sizeof( DEVICE_OBJECT ) );
    iostack = IoGetCurrentIrpStackLocation( Irp );
    newirp = IoAllocateIrp( LowerDevice->StackSize, TRUE );
    DbgPrint( "NewIrp------------\n");
    EnumPrintIrpStack( newirp );
    memcpy( &backIrp, newirp, sizeof(IRP) );
    /*主要复制 USER 相关事件与回调*/
    memcpy( newirp, Irp, sizeof(IRP) );
    newirp->StackCount = backIrp.StackCount;
    newirp->CurrentLocation = backIrp.CurrentLocation; /*很关键点*/
    /*NO THIS will to Write Device(IoCallDriver) struct to 0x0*/
    /*SIZE=IRP+TAIL+STACK*/
    newirp->Size = backIrp.Size; /*很关键点*/
    /*开始这里没加,发现 IRP 释放后会冲掉 LowerDevice 的对象几个子节*/
    newirp->Tail = backIrp.Tail; /*很关键点*/
    newiostack = IoGetNextIrpStackLocation( newirp );
    memcpy( newiostack, iostack, sizeof(IO_STACK_LOCATION) );
    newiostack->DeviceObject = LowerDevice/*文件驱动*/;
    newiostack->FileObject = iostack->FileObject;
    KeInitializeEvent( &CompleteEvent, NotificationEvent, FALSE );
    IoSetCompletionRoutine( newirp, IoCompletion, &CompleteEvent, TRUE, TRUE, TRUE );
    ok = IoCallDriver( LowerDevice, newirp );
    KeWaitForSingleObject( &CompleteEvent, Executive, KernelMode, TRUE, NULL );
    {
        Irp->IoStatus.Status = newirp->IoStatus.Status;
        Irp->IoStatus.Information = newirp->IoStatus.Information;
        Irp->IoStatus.Pointer = newirp->IoStatus.Pointer;
        DbgPrint( "Irp:%08x M:%08x n:%08x\n", newirp->IoStatus.Status,iostack->MajorFunction, iostack->MinorFunction);
        DbgPrint( "Commpltetw New Irp------------\n");
        EnumPrintIrpStack( newirp );
        /*因为 IOFREEIRP 有线程操作,这样才能释放*/
        newirp->ThreadListEntry = backIrp.ThreadListEntry; /*很关键点*/
        IoFreeIrp( newirp );
        IoCompleteRequest( Irp, IO_NO_INCREMENT );/*完成原来 IRP*/
    };
    // memcpy( LowerDevice, &lodev, sizeof( DEVICE_OBJECT ) );
    return ok;
}

NTSTATUS DriverEntry(
    IN PDRIVER_OBJECT DriverObject,
    IN PUNICODE_STRING Registry
)
{
    UNICODE_STRING DiskDeviceName;
    UNICODE_STRING DiskName;
    NTSTATUS opRet;
    HANDLE hFile;
    OBJECT_ATTRIBUTES ob;
    IO_STATUS_BLOCK IoStatus;
    OBJECT_HANDLE_INFORMATION ohi;
    PDEVICE_OBJECT FileSystemDevice;
    PDEVICE_OBJECT DiskDevice;
    UNICODE_STRING cname;
    PFILE_OBJECT fobj;
    DriverObject->MajorFunction[IRP_MJ_CREATE] = VirtualDiskCommonControl;
    DriverObject->MajorFunction[IRP_MJ_READ] = VirtualDiskCommonControl;
    DriverObject->MajorFunction[IRP_MJ_WRITE] = VirtualDiskCommonControl;
    DriverObject->MajorFunction[IRP_MJ_DEVICE_CONTROL] = VirtualDiskCommonControl;
    DriverObject->MajorFunction[IRP_MJ_SYSTEM_CONTROL] = VirtualDiskCommonControl;
    DriverObject->MajorFunction[IRP_MJ_CLEANUP] = VirtualDiskCommonControl;
    DriverObject->MajorFunction[IRP_MJ_FILE_SYSTEM_CONTROL] = VirtualDiskCommonControl;
    DriverObject->MajorFunction[IRP_MJ_CLOSE] = VirtualDiskCommonControl;
    DriverObject->MajorFunction[IRP_MJ_FLUSH_BUFFERS] = VirtualDiskCommonControl;
    DriverObject->MajorFunction[IRP_MJ_PNP] = VirtualDiskCommonControl;
    DriverObject->MajorFunction[IRP_MJ_POWER] = VirtualDiskDispatchPower;
    DriverObject->MajorFunction[IRP_MJ_CREATE_NAMED_PIPE] = VirtualDiskCommonControl;
    DriverObject->MajorFunction[IRP_MJ_CREATE_MAILSLOT] = VirtualDiskCommonControl;
    DriverObject->DriverExtension->AddDevice = VirtualDiskAddDevice;
    DriverObject->DriverUnload = VirtualDiskUnload;
    RtlInitUnicodeString( &DiskDeviceName, L"\\Device\\VirtualDisk");
    RtlInitUnicodeString( &DiskName, L"\\DosDevices\\w:");//用 CMD 去测试吧很爽
    RtlInitUnicodeString( &cname, L"\\DosDevices\\c:\\");
    IoCreateSymbolicLink( &DiskName, &DiskDeviceName );
    InitializeObjectAttributes(
        &ob,
        &cname,
        OBJ_KERNEL_HANDLE,
        NULL,
        NULL
    );
    opRet = ZwCreateFile(
        &hFile,
        SYNCHRONIZE|FILE_ANY_ACCESS,
        &ob,
        &IoStatus,
        0,
        0,
        FILE_SHARE_READ,
        FILE_OPEN,
        FILE_SYNCHRONOUS_IO_NONALERT|FILE_DIRECTORY_FILE,
        NULL,
        0
    );
    ERR_EXIT( opRet );
    opRet = ObReferenceObjectByHandle(
        hFile,
        FILE_READ_DATA,
        NULL,
        KernelMode,
        &fobj,
        &ohi
    );
    ERR_EXIT( opRet );
    FileSystemDevice = IoGetRelatedDeviceObject( fobj );
    opRet = IoGetDiskDeviceObject( FileSystemDevice, &DiskDevice );
    // ERR_EXIT( opRet );
    opRet = IoCreateDevice(
        DriverObject,
        0,
        &DiskDeviceName,
        FileSystemDevice->DeviceType,
        0,
        0,
        &VirtualDiskDevice
    );
    //ERR_EXIT( opRet );
    LowerDevice = FileSystemDevice;
    return STATUS_SUCCESS;
}
```

核心点：

保留原 `IRP` 的 `THREAD` 环境。

一些道理要调试你才知道，用 `DBG` 多看`_IRP` 结构。

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `VirtualDisk.rar` |
