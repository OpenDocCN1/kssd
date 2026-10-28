
```c
IoCopyCurrentIrpStackLocationToNext( Irp );

IoSetCompletionRoutine( Irp,
                        SfCreateCompletion,
                        &waitEvent,
                        TRUE,
                        TRUE,
                        TRUE );

status = IoCallDriver( ((PSFILTER_DEVICE_EXTENSION) DeviceObject->DeviceExtension)->AttachedToDeviceObject, Irp );

if (STATUS_PENDING == status)//若状态是 挂起
{
    NTSTATUS localStatus = KeWaitForSingleObject(&waitEvent, Executive, KernelMode, FALSE, NULL);
    ASSERT(STATUS_SUCCESS == localStatus);
}

ASSERT(KeReadStateEvent(&waitEvent) || !NT_SUCCESS(Irp->IoStatus.Status));

if (FlagOn(SfDebug, (SFDEBUG_GET_CREATE_NAMES|SFDEBUG_DISPLAY_CREATE_NAMES)))
{
    SfDisplayCreateFileName( Irp );
}

status = Irp->IoStatus.Status;
IoCompleteRequest( Irp, IO_NO_INCREMENT );
return status;
```

}

NTSTATUS  SfCreateCompletion( IN PDEVICE_OBJECT DeviceObject, IN PIRP Irp, IN PVOID Context )
{
    PKEVENT event = Context;
    UNREFERENCED_PARAMETER( DeviceObject );
    UNREFERENCED_PARAMETER( Irp );
    ASSERT(IS_MY_DEVICE_OBJECT( DeviceObject ));
    KeSetEvent(event, IO_NO_INCREMENT, FALSE);
    return STATUS_MORE_PROCESSING_REQUIRED;
}

NTSTATUS  SfCleanupClose( IN PDEVICE_OBJECT DeviceObject, IN PIRP Irp )
{
    PAGED_CODE();
    ASSERT(!IS_MY_CONTROL_DEVICE_OBJECT( DeviceObject ));
    ASSERT(IS_MY_DEVICE_OBJECT( DeviceObject ));
    IoSkipCurrentIrpStackLocation( Irp );
    return IoCallDriver( ((PSFILTER_DEVICE_EXTENSION) DeviceObject->DeviceExtension)->AttachedToDeviceObject, Irp );
}

NTSTATUS  SfFsControl( IN PDEVICE_OBJECT DeviceObject, IN PIRP Irp )
{
    /*
        参数说明:
        DeviceObject:    我们创建的设备对象。它是被绑定到文件系统控制设备对象栈上。
    */
    PIO_STACK_LOCATION irpSp = IoGetCurrentIrpStackLocation( Irp );
    PAGED_CODE();
    ASSERT(!IS_MY_CONTROL_DEVICE_OBJECT( DeviceObject ));
    ASSERT(IS_MY_DEVICE_OBJECT( DeviceObject ));
    switch (irpSp->MinorFunction) {
        case IRP_MN_MOUNT_VOLUME:
            return SfFsControlMountVolume( DeviceObject, Irp );
        case IRP_MN_LOAD_FILE_SYSTEM:
            return SfFsControlLoadFileSystem( DeviceObject, Irp );
        case IRP_MN_USER_FS_REQUEST:
        {
            switch (irpSp->Parameters.FileSystemControl.FsControlCode) {
                case FSCTL_DISMOUNT_VOLUME:
                {
                    PSFILTER_DEVICE_EXTENSION devExt = DeviceObject->DeviceExtension;
                    SF_LOG_PRINT( SFDEBUG_DISPLAY_ATTACHMENT_NAMES,
                                  ("SFilter!SfFsControl:                         Dismounting volume         %p \"%wZ\"\n",
                                   devExt->AttachedToDeviceObject,
                                   &devExt->DeviceName) );
                    break;
                }
            }
            break;
        }
    }
    IoSkipCurrentIrpStackLocation( Irp );
    return IoCallDriver( ((PSFILTER_DEVICE_EXTENSION)DeviceObject->DeviceExtension)->AttachedToDeviceObject, Irp );
}

NTSTATUS  SfFsControlCompletion( IN PDEVICE_OBJECT DeviceObject, IN PIRP Irp, IN PVOID Context )
{
    UNREFERENCED_PARAMETER( DeviceObject );
    UNREFERENCED_PARAMETER( Irp );
    ASSERT(IS_MY_DEVICE_OBJECT( DeviceObject ));
    ASSERT(Context != NULL);
#if WINVER >= 0x0501
    if (IS_WINDOWSXP_OR_LATER())
    {
        KeSetEvent((PKEVENT)Context, IO_NO_INCREMENT, FALSE);
    }
    else
    {
#endif
        if (KeGetCurrentIrql() > PASSIVE_LEVEL)
        {
            ExQueueWorkItem( (PWORK_QUEUE_ITEM) Context, DelayedWorkQueue );
        }
        else
        {
            PWORK_QUEUE_ITEM workItem = Context;
            (workItem->WorkerRoutine)(workItem->Parameter);
        }
#if WINVER >= 0x0501
    }
#endif
    return STATUS_MORE_PROCESSING_REQUIRED;
}

NTSTATUS  SfFsControlMountVolume( IN PDEVICE_OBJECT DeviceObject, IN PIRP Irp )
{
    /*
        参数说明:
        DeviceObject:    它是我们创建的设备对象。它被绑定到文件系统CDO的设备栈上。
        Irp:             它是发送给文件系统CDO的挂载请求。它是一个新卷的挂载请求。
    */
    PSFILTER_DEVICE_EXTENSION devExt = DeviceObject->DeviceExtension;
    PIO_STACK_LOCATION irpSp = IoGetCurrentIrpStackLocation( Irp );
    PDEVICE_OBJECT storageStackDeviceObject;
    // newDeviceObject是将要绑定到文件系统的卷设备对象上。或者说，这个newDeviceObject要被绑定到新挂载卷的设备卷上。
    PDEVICE_OBJECT newDeviceObject;
    PSFILTER_DEVICE_EXTENSION newDevExt;
    NTSTATUS status;
    BOOLEAN isShadowCopyVolume;
    PFSCTRL_COMPLETION_CONTEXT completionContext;
    PAGED_CODE();
    ASSERT(IS_MY_DEVICE_OBJECT( DeviceObject ));
    ASSERT(IS_DESIRED_DEVICE_TYPE(DeviceObject->DeviceType));
    /*
        在把IRP发送到文件系统之前，当挂载请求到来时，Vpb->RealDevice保存的是：将要被挂载的磁盘设备对象。
        storageStackDeviceObject事先保存了VPB的值，这是因为：当IRP下发给底层驱动后，可能会改变。
    */
    storageStackDeviceObject = irpSp->Parameters.MountVolume.Vpb->RealDevice;
    status = SfIsShadowCopyVolume ( storageStackDeviceObject, &isShadowCopyVolume );
    if (NT_SUCCESS(status) &&
        isShadowCopyVolume &&
        !FlagOn(SfDebug,SFDEBUG_ATTACH_TO_SHADOW_COPIES))
    {
        UNICODE_STRING shadowDeviceName;
        WCHAR shadowNameBuffer[MAX_DEVNAME_LENGTH];
        RtlInitEmptyUnicodeString( &shadowDeviceName, shadowNameBuffer, sizeof(shadowNameBuffer) );
        SfGetObjectName( storageStackDeviceObject, &shadowDeviceName );
        SF_LOG_PRINT( SFDEBUG_DISPLAY_ATTACHMENT_NAMES,
                      ("SFilter!SfFsControlMountVolume               Not attaching to Volume    %p \"%wZ\", shadow copy volume\n",
                       storageStackDeviceObject,
                       &shadowDeviceName) );
        //如果不打算绑定卷影则跳到下一层驱动
        IoSkipCurrentIrpStackLocation( Irp );
        return IoCallDriver( devExt->AttachedToDeviceObject, Irp );
    }

    status = IoCreateDevice( gSFilterDriverObject,
                             sizeof( SFILTER_DEVICE_EXTENSION ),
                             NULL,
                             DeviceObject->DeviceType,
                             0,
                             FALSE,
                             &newDeviceObject );
    /*如果不把IRP发送到文件系统中，那么文件系统就不会收到这个卷的挂载请求。*/
    if (!NT_SUCCESS( status ))
    {
        KdPrint(( "SFilter!SfFsControlMountVolume: Error creating volume device object, status=%08x\n", status ));
        Irp->IoStatus.Information = 0;
        Irp->IoStatus.Status = status;
        IoCompleteRequest( Irp, IO_NO_INCREMENT );
        return status;
    }

    //填写设备扩展，这样目的是：可以让完成函数更容易到storageStackDeviceObject
    newDevExt = newDeviceObject->DeviceExtension;
    newDevExt->StorageStackDeviceObject = storageStackDeviceObject;
    RtlInitEmptyUnicodeString( &newDevExt->DeviceName, newDevExt->DeviceNameBuffer, sizeof(newDevExt->DeviceNameBuffer) );
    SfGetObjectName( storageStackDeviceObject, &newDevExt->DeviceName );

#if WINVER >= 0x0501
    if (IS_WINDOWSXP_OR_LATER())
    {
        //在这里设置了事件对象，把它用在完成例程中。这样做的目的是：通知当前例程，文件系统已经完成了当前卷的挂载。
        KEVENT waitEvent;
        KeInitializeEvent( &waitEvent, NotificationEvent, FALSE );
        IoCopyCurrentIrpStackLocationToNext ( Irp );
```
