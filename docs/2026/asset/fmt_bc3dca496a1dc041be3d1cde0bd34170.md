
```c
DriverObject->MajorFunction[IRP\_MJ\_CLEANUP] = SfCleanupClose;

DriverObject->MajorFunction[IRP\_MJ\_CLOSE] = SfCleanupClose;

    fastIoDispatch = ExAllocatePoolWithTag( NonPagedPool,                                //从非分页池中分配

                                                                            sizeof( FAST\_IO\_DISPATCH ),        //要分配的字节数

                                                                            SFLT\_POOL\_TAG );                        //指定一个4字节的标签(前面已宏定义:'tlFS')

    if (!fastIoDispatch)                //内存分配失败

    {

            IoDeleteDevice( gSFilterControlDeviceObject );                                                //删除上面创建的CDO

            return STATUS\_INSUFFICIENT\_RESOURCES;                                                                //返回一个错误status码(资源不足)

    }

RtlZeroMemory( fastIoDispatch, sizeof( FAST\_IO\_DISPATCH ) );

DriverObject->FastIoDispatch = fastIoDispatch;                                                        //将FastIo分派表保存到驱动对象的FastIoDispatch域

fastIoDispatch->SizeOfFastIoDispatch = sizeof( FAST\_IO\_DISPATCH );                //设置FastIo分派表的长度域

fastIoDispatch->FastIoCheckIfPossible = SfFastIoCheckIfPossible;                //设置FastIo分派函数,共21个

fastIoDispatch->FastIoRead = SfFastIoRead;

fastIoDispatch->FastIoWrite = SfFastIoWrite;

fastIoDispatch->FastIoQueryBasicInfo = SfFastIoQueryBasicInfo;

fastIoDispatch->FastIoQueryStandardInfo = SfFastIoQueryStandardInfo;

fastIoDispatch->FastIoLock = SfFastIoLock;

fastIoDispatch->FastIoUnlockSingle = SfFastIoUnlockSingle;

fastIoDispatch->FastIoUnlockAll = SfFastIoUnlockAll;

fastIoDispatch->FastIoUnlockAllByKey = SfFastIoUnlockAllByKey;

fastIoDispatch->FastIoDeviceControl = SfFastIoDeviceControl;

fastIoDispatch->FastIoDetachDevice = SfFastIoDetachDevice;

fastIoDispatch->FastIoQueryNetworkOpenInfo = SfFastIoQueryNetworkOpenInfo;

fastIoDispatch->MdlRead = SfFastIoMdlRead;

fastIoDispatch->MdlReadComplete = SfFastIoMdlReadComplete;

fastIoDispatch->PrepareMdlWrite = SfFastIoPrepareMdlWrite;

fastIoDispatch->MdlWriteComplete = SfFastIoMdlWriteComplete;

fastIoDispatch->FastIoReadCompressed = SfFastIoReadCompressed;

fastIoDispatch->FastIoWriteCompressed = SfFastIoWriteCompressed;

fastIoDispatch->MdlReadCompleteCompressed = SfFastIoMdlReadCompleteCompressed;

fastIoDispatch->MdlWriteCompleteCompressed = SfFastIoMdlWriteCompleteCompressed;

fastIoDispatch->FastIoQueryOpen = SfFastIoQueryOpen;

//--------------------------------注册fsFilter回调函数-------------------------------

#if WINVER >= 0x0501        //如果OS版本为WinXP以上,编译这段代码,否则不编译

    {

            FS\_FILTER\_CALLBACKS fsFilterCallbacks;

            if (NULL != gSfDynamicFunctions.RegisterFileSystemFilterCallbacks)

            {

                    fsFilterCallbacks.SizeOfFsFilterCallbacks = sizeof( FS\_FILTER\_CALLBACKS );

        fsFilterCallbacks.PreAcquireForSectionSynchronization = SfPreFsFilterPassThrough;

        fsFilterCallbacks.PostAcquireForSectionSynchronization = SfPostFsFilterPassThrough;

        fsFilterCallbacks.PreReleaseForSectionSynchronization = SfPreFsFilterPassThrough;

        fsFilterCallbacks.PostReleaseForSectionSynchronization = SfPostFsFilterPassThrough;

        fsFilterCallbacks.PreAcquireForCcFlush = SfPreFsFilterPassThrough;

        fsFilterCallbacks.PostAcquireForCcFlush = SfPostFsFilterPassThrough;

        fsFilterCallbacks.PreReleaseForCcFlush = SfPreFsFilterPassThrough;

        fsFilterCallbacks.PostReleaseForCcFlush = SfPostFsFilterPassThrough;

        fsFilterCallbacks.PreAcquireForModifiedPageWriter = SfPreFsFilterPassThrough;

        fsFilterCallbacks.PostAcquireForModifiedPageWriter = SfPostFsFilterPassThrough;

        fsFilterCallbacks.PreReleaseForModifiedPageWriter = SfPreFsFilterPassThrough;

        fsFilterCallbacks.PostReleaseForModifiedPageWriter = SfPostFsFilterPassThrough;

        status = (gSfDynamicFunctions.RegisterFileSystemFilterCallbacks)( DriverObject, &fsFilterCallbacks );

        if (!NT\_SUCCESS( status ))

                {

                        DriverObject->FastIoDispatch = NULL;

            ExFreePool( fastIoDispatch );

            IoDeleteDevice( gSFilterControlDeviceObject );

            return status;

                }

        }

    }

#endif

status = IoRegisterFsRegistrationChange( DriverObject, SfFsNotification );

if (!NT\_SUCCESS( status ))

    {

            KdPrint(( "SFilter!DriverEntry: Error registering FS change notification, status=%08x\n", status ));

            DriverObject->FastIoDispatch = NULL;                                        //注销指向fastIo函数组的指针为NULL

            ExFreePoolWithTag( fastIoDispatch, SFLT\_POOL\_TAG );                //释放分配给fastIo函数组的内存

            IoDeleteDevice( gSFilterControlDeviceObject );                        //删除上面创建的CDO

            return status;                                                                                        //错误返回

    }

{

            PDEVICE\_OBJECT rawDeviceObject;

            PFILE\_OBJECT fileObject;

            RtlInitUnicodeString( &nameString, L"\\Device\\RawDisk" );

            /\*

            IoGetDeviceObjectPointer函数的功能是:

            它从下层的设备对象名称来获得下层设备指针。该函数造成了对下层设备对象以及下层设备对象所对应的文件对象的引用。

            如果本层驱动在卸载之前对下层的设备对象的引用还没有消除，则下层驱动的卸载会被停止。因此必须要消除对下层设备对象的引用。

            但是程序一般不会直接对下层设备对象的引用减少。因此只要减少对文件对象的引用就可以减少文件对象和设备对象两个对象的引用。

            事实上，IoGetDeviceObjectPointer返回的并不是下层设备对象的指针，而是该设备堆栈中顶层的设备对象的指针。

            IoGetDeviceObjectPointer函数的调用必须在 IRQL=PASSIVE\_LEVEL的级别上运行。

            \*/

    status = IoGetDeviceObjectPointer( &nameString, FILE\_READ\_ATTRIBUTES, &fileObject, &rawDeviceObject );

            if (NT\_SUCCESS( status ))

            {

                    SfFsNotification( rawDeviceObject, TRUE );

                    ObDereferenceObject( fileObject ); //这里减少对文件对象的引用

            }

            RtlInitUnicodeString( &nameString, L"\\Device\\RawCdRom" );

    status = IoGetDeviceObjectPointer( &nameString, FILE\_READ\_ATTRIBUTES, &fileObject, &rawDeviceObject );

            if (NT\_SUCCESS( status ))

            {

                    SfFsNotification( rawDeviceObject, TRUE );

                    ObDereferenceObject( fileObject );//这里减少对文件对象的引用

            }

    }

    ClearFlag( gSFilterControlDeviceObject->Flags, DO\_DEVICE\_INITIALIZING );

    return STATUS\_SUCCESS;

}

// 驱动卸载函数

#if DBG && WINVER >= 0x0501

VOID  DriverUnload( IN PDRIVER\_OBJECT DriverObject )

{

        PSFILTER\_DEVICE\_EXTENSION devExt;

        PFAST\_IO\_DISPATCH fastIoDispatch;

        NTSTATUS status;

        ULONG numDevices;

        ULONG i;

        LARGE\_INTEGER interval;

        PDEVICE\_OBJECT devList[DEVOBJ\_LIST\_SIZE];

        ASSERT(DriverObject == gSFilterDriverObject);

        SF\_LOG\_PRINT( SFDEBUG\_DISPLAY\_ATTACHMENT\_NAMES, ("SFilter!DriverUnload:                        Unloading driver (%p)\n", DriverObject) );

        IoUnregisterFsRegistrationChange( DriverObject, SfFsNotification );

        for (;;)

        {

                ASSERT( NULL != gSfDynamicFunctions.EnumerateDeviceObjectList );

        status = (gSfDynamicFunctions.EnumerateDeviceObjectList)( DriverObject, devList, sizeof(devList), &numDevices);

                if (numDevices <= 0)

                {

                        break;

                }

                numDevices = min( numDevices, DEVOBJ\_LIST\_SIZE );

                for (i=0; i < numDevices; i++)

                {

                        devExt = devList[i]->DeviceExtension;

                        if (NULL != devExt)
```
