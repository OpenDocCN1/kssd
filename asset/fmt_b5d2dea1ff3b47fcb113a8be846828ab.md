
```
pSectionObjectPointer = fileObject->SectionObjectPointer;

pSectionObjectPointer->ImageSectionObject = 0;

pSectionObjectPointer->DataSectionObject = 0;

IoCallDriver(DeviceObject, Irp);//这里会依次进入 NTFS.sys 驱动的 NtfsFsdSetInformation 例程->NtfsSetDispositionInfo（）->MmFlushImageSection(),

//MmFlushImageSection（）这函数是用来检查 FILE_OBJECT 对象的 SECTION_OBJECT_POINTER 结构的变量，检查这个文件

//在内存有没有被映射。也就是有没有执行。如果上面那样设置了，也就是说文件可以删除了。我们也可以 HOOK NTFS.sys 导入表中的

//的 MmFlushImageSection（），来检查这个文件对象是不是我们要删除 的，是的话，返回 TRUE 就行了。

KeWaitForSingleObject(&event1, Executive, KernelMode, TRUE, NULL);

ObDereferenceObject(fileObject);

return TRUE;

}

NTSTATUS DriverEntry(

    IN PDRIVER_OBJECT DriverObject,

    IN PUNICODE_STRING RegistryPath

    )

{

    UNICODE_STRING                  uniDeviceName;

    UNICODE_STRING                  uniSymLink;

    NTSTATUS                        ntStatus;

    PDEVICE_OBJECT                  deviceObject = NULL;

    HANDLE                          hFileHandle;

    RtlInitUnicodeString(&uniDeviceName, NT_DEVICE_NAME);

    RtlInitUnicodeString(&uniSymLink, DOS_DEVICE_NAME);

    ntStatus = IoCreateDevice(

            DriverObject,

            0x100u,

            &uniDeviceName,

            FILE_DEVICE_UNKNOWN,

            FILE_DEVICE_SECURE_OPEN,

            TRUE,

            &deviceObject);

    if (!NT_SUCCESS(ntStatus))

    {

        return ntStatus;

    }

    ntStatus = IoCreateSymbolicLink(&uniSymLink, &uniDeviceName);

    if (!NT_SUCCESS(ntStatus))

    {

        IoDeleteDevice(deviceObject);

        return ntStatus;

    }

    DriverObject->DriverUnload = SKillUnloadDriver;

    //

    // 重点在这

    //

    hFileHandle = SkillIoOpenFile(L"\\\\Device\\\\HarddiskVolume1\\\\test.exe",

            FILE_READ_ATTRIBUTES,

            FILE_SHARE_DELETE);   //得到文件句柄

    if (hFileHandle!=NULL)

    {

        SKillDeleteFile(hFileHandle);

        ZwClose(hFileHandle);

    }

    return STATUS_SUCCESS;

}
```
