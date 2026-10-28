
newDevExt = `newDeviceObject`->`DeviceExtension`;

`newDevExt`->`StorageStackDeviceObject` = `storageStackDeviceObject`;

RtlInitEmptyUnicodeString( &`newDevExt`->`DeviceName`, `newDevExt`->`DeviceNameBuffer`,sizeof(`newDevExt`->`DeviceNameBuffer`) );

`SfGetObjectName`( `storageStackDeviceObject`, &`newDevExt`->`DeviceName` );

/*

 在绑定最后的时候，再测试下，该设备是否被绑定过。这里加了一个锁。如果没被绑定，则执行下面的绑定过程，否则，直接返回。

 */

ExAcquireFastMutex( &`gSfilterAttachLock` );

if (!`SfIsAttachedToDevice`( `devList[i]`, NULL ))

{

        `status` = `SfAttachToMountedDevice`( `devList[i]`, `newDeviceObject` );

        if (!`NT_SUCCESS`( `status` ))

        {

                `SfCleanupMountedDevice`( `newDeviceObject` );

                `IoDeleteDevice`( `newDeviceObject` );

        }

}

else

{

        `SfCleanupMountedDevice`( `newDeviceObject` );

        `IoDeleteDevice`( `newDeviceObject` );

}

ExReleaseFastMutex( &`gSfilterAttachLock` );

}

finally {

 /*

 减少设备对象的计数，这个计数是由函数`IoGetDiskDeviceObject`增加的。成功绑定后，就减少该设备对象的计数。

 一旦成功绑定到`devList[i]`，I/O管理器会确定设备栈的下层设备，会一直存在，一直到这个文件系统栈被卸掉。

 */

        if (`storageStackDeviceObject` != NULL)

        {

                `ObDereferenceObject`( `storageStackDeviceObject` );

        }

        //减少设备对象的计数，这个计数是由函数`IoEnumerateDeviceObjectList`增加的。

        `ObDereferenceObject`( `devList[i]` );

}

}

`status` = `STATUS_SUCCESS`;

ExFreePool( `devList` );

}

return `status`;

/}

#endif

```

NTSTATUS `SfAttachToMountedDevice`( IN `PDEVICE_OBJECT` `DeviceObject`, IN `PDEVICE_OBJECT` `SFilterDeviceObject` )

{

/*

`SfAttachToMountedDevice`函数的功能: 完成绑定一个文件系统卷设备的操作。

参数说明:

`SFilterDeviceObject`:  它是我们使用`IoCreateDevice`函数来创建的设备对象。

*/

`PSFILTER_DEVICE_EXTENSION` `newDevExt` = `SFilterDeviceObject`->`DeviceExtension`;

`NTSTATUS` `status`;

`ULONG` `i`;

`PAGED_CODE`();

`ASSERT`(`IS_MY_DEVICE_OBJECT`(`SFilterDeviceObject`));

#if `WINVER` >= 0x0501

`ASSERT`(!`SfIsAttachedToDevice` ( `DeviceObject`, NULL ));

#endif

if (FlagOn( `DeviceObject`->`Flags`, `DO_BUFFERED_IO` ))

{

        `SetFlag`( `SFilterDeviceObject`->`Flags`, `DO_BUFFERED_IO` );

}

if (FlagOn( `DeviceObject`->`Flags`, `DO_DIRECT_IO` ))

{

        `SetFlag`( `SFilterDeviceObject`->`Flags`, `DO_DIRECT_IO` );

}

for (`i`=0; `i` < 8; `i++`)

{

        `LARGE_INTEGER` `interval`;

        //调用`SfAttachDeviceToDeviceStack`函数进行  卷的绑定

        `status` = `SfAttachDeviceToDeviceStack`( `SFilterDeviceObject`, `DeviceObject`, &`newDevExt`->`AttachedToDeviceObject` );

        if (`NT_SUCCESS`(`status`))

        {

                `ClearFlag`( `SFilterDeviceObject`->`Flags`, `DO_DEVICE_INITIALIZING` );

                `SF_LOG_PRINT`( `SFDEBUG_DISPLAY_ATTACHMENT_NAMES`,

                  ("`SFilter`!`SfAttachToMountedDevice`:             Attaching to volume        %p \"%wZ\"\n",

                   `newDevExt`->`AttachedToDeviceObject`,

                   &`newDevExt`->`DeviceName`) );

                return `STATUS_SUCCESS`;

        }

        `interval`.QuadPart = (500 * `DELAY_ONE_MILLISECOND`);      //delay 1/2 second

        `KeDelayExecutionThread`( `KernelMode`, `FALSE`, &`interval` );

}

return `status`;

}

```

VOID `SfCleanupMountedDevice`( IN `PDEVICE_OBJECT` `DeviceObject` )

{

UNREFERENCED_PARAMETER( `DeviceObject` );

`ASSERT`(`IS_MY_DEVICE_OBJECT`(`DeviceObject`));

}

```

VOID `SfGetObjectName`( IN `PVOID` `Object`, IN OUT `PUNICODE_STRING` `Name` )

{

`NTSTATUS` `status`;

CHAR `nibuf`[512];                                                                                                                        //接收返回的对象名

`POBJECT_NAME_INFORMATION` `nameInfo` = (`POBJECT_NAME_INFORMATION`)`nibuf`;                //nameInfo存放的是指向`UNICODE_STRING`结构变量的指针

`ULONG` `retLength`;                                                                                                                        //接收"实际返回的对象名长度"

        `status` = `ObQueryNameString`( `Object`, `nameInfo`, sizeof(`nibuf`), &`retLength`);        //返回对象名

        `Name`->`Length` = 0;

        if (`NT_SUCCESS`( `status` ))

        {

                `RtlCopyUnicodeString`( `Name`, &`nameInfo`->`Name` );

        }

}

```

#if `WINVER` >= 0x0501

VOID `SfGetBaseDeviceObjectName`( IN `PDEVICE_OBJECT` `DeviceObject`, IN OUT `PUNICODE_STRING` `Name` )

{

`ASSERT`( NULL != `gSfDynamicFunctions`.GetDeviceAttachmentBaseRef );

`DeviceObject` = (`gSfDynamicFunctions`.GetDeviceAttachmentBaseRef)( `DeviceObject` );

`SfGetObjectName`( `DeviceObject`, `Name` );

`ObDereferenceObject`( `DeviceObject` );

}

#endif

```

`PUNICODE_STRING` `SfGetFileName`( IN `PFILE_OBJECT` `FileObject`, IN `NTSTATUS` `CreateStatus`, IN OUT `PGET_NAME_CONTROL` `NameControl` )

{

`POBJECT_NAME_INFORMATION` `nameInfo`;

`NTSTATUS` `status`;

`ULONG` `size`;

`ULONG` `bufferSize`;

        `NameControl`->`allocatedBuffer` = NULL;

        `nameInfo` = (`POBJECT_NAME_INFORMATION`)`NameControl`->`smallBuffer`;

        `bufferSize` = sizeof(`NameControl`->`smallBuffer`);

        `status` = `ObQueryNameString`( (`NT_SUCCESS`( `CreateStatus` ) ? (`PVOID`)`FileObject` :        (`PVOID`)`FileObject`->`DeviceObject`), `nameInfo`, `bufferSize`, &`size` );

        if (`status` == `STATUS_BUFFER_OVERFLOW`)

        {

                `bufferSize` = `size` + sizeof(WCHAR);

                `NameControl`->`allocatedBuffer` = `ExAllocatePoolWithTag`( `NonPagedPool`, `bufferSize`, `SFLT_POOL_TAG` );

                if (NULL == `NameControl`->`allocatedBuffer`)

                {

                        `RtlInitEmptyUnicodeString`(

                (`PUNICODE_STRING`)&`NameControl`->`smallBuffer`,

                (PWCHAR)(`NameControl`->`smallBuffer` + sizeof(`UNICODE_STRING`)),

                (USHORT)(sizeof(`NameControl`->`smallBuffer`) - sizeof(`UNICODE_STRING`)) );

            return (`PUNICODE_STRING`)&`NameControl`->`smallBuffer`;

        }

                `nameInfo` = (`POBJECT_NAME_INFORMATION`)`NameControl`->`allocatedBuffer`;

                `status` = `ObQueryNameString`(

                      `FileObject`,

                      `nameInfo`,

                      `bufferSize`,

                      &`size` );

    }

        if (`NT_SUCCESS`( `status` ) && !`NT_SUCCESS`( `CreateStatus` ))

        {

                `ULONG` `newSize`;

        PCHAR `newBuffer`;

        `POBJECT_NAME_INFORMATION` `newNameInfo`;

                `newSize` = `size` + `FileObject`->`FileName`.Length;

                if (NULL != `FileObject`->`RelatedFileObject`)

                {

                        `newSize` += `FileObject`->`RelatedFileObject`->`FileName`.Length + sizeof(WCHAR);

                }

                if (`newSize` > `bufferSize`)

                {

                        `newBuffer` = `ExAllocatePoolWithTag`( `NonPagedPool`, `newSize`, `SFLT_POOL_TAG` );

                        if (NULL == `newBuffer`)

                        {

                                `RtlInitEmptyUnicodeString`(

                    (`PUNICODE_STRING`)&`NameControl`->`smallBuffer`,

                    (PWCHAR)(`NameControl`->`smallBuffer` + sizeof(`UNICODE_STRING`)),

                    (USHORT)(sizeof(`NameControl`->`smallBuffer`) - sizeof(`UNICODE_STRING`)) );

                return (`PUNICODE_STRING`)&`NameControl`->`smallBuffer`;

        }

                `nameInfo` = (`POBJECT_NAME_INFORMATION`)`newBuffer`;

                `status` = `ObQueryNameString`(

                      (PVOID)`FileObject`,

                      `nameInfo`,

                      `newSize`,

                      &`size` );

                }

                // 原文未完

