
# 学习强制删除正在运行的文件

> 原文：[`www.pediy.com/kssd/pediy10/87345.html`](https://www.pediy.com/kssd/pediy10/87345.html)

首先说明一下的是，代码是从网上找来的，自己加了一些理解，供和我一样菜的人学习。

![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")

`http://hi.baidu.com/%C3%F7%ED%F8%B5%C4%D0%C4`

强制删除文件，简单说来，其实就是自己构造`IRP`，然后向`NTFS.sys`发送`IRP`，先设置文件的属性，再删除文件。删除文件时会先到`NTFS.sys`的分派例程，依次进入`NtfsSetDispositionInfo-》MmFlushImageSection`。`MmFlushImageSection()`函数会检查这个文件对象的`SECTION_OBJECT_POINTER`结构，看里面是不是为空，也就是检查这个文件在没在运行，没有的话，直接返回`TRUE`。所以我们如果想要删除正在运行的文件，一种方法是，我们可以设置`SECTION_OBJECT_POINTER`结构里的变量都为`0`。这样`MmFlushImageSection()`为返回`TRUE`，表示能删除。另一种方法是`HOOK NTFS.sys`的导入表中的`MmFlushImageSection()`函数，在`HOOK`函数中检查是不是我们要删除的文件，是的话直接返回`TRUE`也行。下面是完整代码

```c
#include <ntddk.h>

#define NT_DEVICE_NAME             L"\\Device\\SuperKill"

#define DOS_DEVICE_NAME            L"\\DosDevices\\SuperKill"

VOID

SKillUnloadDriver(
      IN PDRIVER_OBJECT    DriverObject

      )

{

      PDEVICE_OBJECT    deviceObject = DriverObject->DeviceObject;

      UNICODE_STRING    uniSymLink;

      RtlInitUnicodeString(&uniSymLink, DOS_DEVICE_NAME);

      IoDeleteSymbolicLink(&uniSymLink);

      IoDeleteDevice(deviceObject);

}

HANDLE

SkillIoOpenFile(

      IN PCWSTR FileName,

      IN ACCESS_MASK DesiredAccess,

      IN ULONG ShareAccess

      )

{

      NTSTATUS               ntStatus;

      UNICODE_STRING         uniFileName;

      OBJECT_ATTRIBUTES      objectAttributes;

      HANDLE                 ntFileHandle;

      IO_STATUS_BLOCK        ioStatus;

      if (KeGetCurrentIrql() > PASSIVE_LEVEL)

      {

          return 0;

      }

      RtlInitUnicodeString(&uniFileName, FileName);

      InitializeObjectAttributes(&objectAttributes, &uniFileName,

          OBJ_KERNEL_HANDLE | OBJ_CASE_INSENSITIVE, NULL, NULL);

      ntStatus = IoCreateFile(&ntFileHandle,

          DesiredAccess,

          &objectAttributes,

          &ioStatus,

          0,

          FILE_ATTRIBUTE_NORMAL,

          ShareAccess,

          FILE_OPEN,

          0,

          NULL,

          0,

          0,

          NULL,

          IO_NO_PARAMETER_CHECKING);

      if (!NT_SUCCESS(ntStatus))

      {

          return 0;

      }

      return ntFileHandle;

}

NTSTATUS

SkillSetFileCompletion(

      IN PDEVICE_OBJECT DeviceObject,

      IN PIRP Irp,

      IN PVOID Context

      )

{

      Irp->UserIosb->Status = Irp->IoStatus.Status;

      Irp->UserIosb->Information = Irp->IoStatus.Information;

      KeSetEvent(Irp->UserEvent, IO_NO_INCREMENT, FALSE);

      IoFreeIrp(Irp);

      return STATUS_MORE_PROCESSING_REQUIRED;

}

BOOLEAN

SKillStripFileAttributes(

      IN HANDLE    FileHandle

      )

{

      NTSTATUS         ntStatus = STATUS_SUCCESS;

      PFILE_OBJECT     fileObject;

      PDEVICE_OBJECT   DeviceObject;

      PIRP             Irp;

      KEVENT           event1;

      FILE_BASIC_INFORMATION    FileInformation;

      IO_STATUS_BLOCK ioStatus;

      PIO_STACK_LOCATION irpSp;

      ntStatus = ObReferenceObjectByHandle(FileHandle,

          DELETE,

          *IoFileObjectType,

          KernelMode,

          &fileObject,

          NULL);//我想知道的是这个文件句柄是在哪个进程的句柄表中

      if (!NT_SUCCESS(ntStatus))

      {

          return FALSE;

      }

      DeviceObject = IoGetRelatedDeviceObject(fileObject);

      Irp = IoAllocateIrp(DeviceObject->StackSize, TRUE);

      if (Irp == NULL)

      {

          ObDereferenceObject(fileObject);

          return FALSE;

      }

      KeInitializeEvent(&event1, SynchronizationEvent, FALSE);

      memset(&FileInformation,0,0x28);

      FileInformation.FileAttributes = FILE_ATTRIBUTE_NORMAL;

      Irp->AssociatedIrp.SystemBuffer = &FileInformation;

      Irp->UserEvent = &event1;

      Irp->UserIosb = &ioStatus;

      Irp->Tail.Overlay.OriginalFileObject = fileObject;

      Irp->Tail.Overlay.Thread = (PETHREAD)KeGetCurrentThread();

      Irp->RequestorMode = KernelMode;

      irpSp = IoGetNextIrpStackLocation(Irp);

      irpSp->MajorFunction = IRP_MJ_SET_INFORMATION;

      irpSp->DeviceObject = DeviceObject;

      irpSp->FileObject = fileObject;

      irpSp->Parameters.SetFile.Length = sizeof(FILE_BASIC_INFORMATION);

      irpSp->Parameters.SetFile.FileInformationClass = FileBasicInformation;

      irpSp->Parameters.SetFile.FileObject = fileObject;

      IoSetCompletionRoutine(

              Irp,

              SkillSetFileCompletion,

              &event1,

              TRUE,

              TRUE,

              TRUE);

      IoCallDriver(DeviceObject, Irp);//调用这个设备对象的驱动对象，并且ＩＯ＿ＳｔＡＣＫ＿ＬＯＣＡｔｉｏｎ会指向下一个，也就是刚刚设置的

                                        //如果没有文件系统驱动建立的设备对象没有 Attacked 的话，就调用文件系统驱动的 IRP_MJ_SET_INFORMATION 分派例程

                                        //会调用 NTFS.sys 驱动的 NtfsFsdSetInformation 例程，再会进入 NtfsSetBasicInfo（）函数，最后它会设置代表此文件的 FCB（文件

                                        //控制块结构的一些信息，用来设置代表此文件的属性。最后不知道在哪里会调用 IoCompleteRequest,它会依次调用先前设置的回调函数

                                        //回调函数会释放刚分配的 IRP 和设置事件对象为受信状态。

      KeWaitForSingleObject(&event1, Executive, KernelMode, TRUE, NULL);//一等到事件对象变成受信状态就会继续向下执行。

      ObDereferenceObject(fileObject);

      return TRUE;

}

BOOLEAN

SKillDeleteFile(

      IN HANDLE    FileHandle

      )

{

      NTSTATUS         ntStatus = STATUS_SUCCESS;

      PFILE_OBJECT     fileObject;

      PDEVICE_OBJECT   DeviceObject;

      PIRP             Irp;

      KEVENT           event1;

      FILE_DISPOSITION_INFORMATION    FileInformation;

      IO_STATUS_BLOCK ioStatus;

      PIO_STACK_LOCATION irpSp;

      PSECTION_OBJECT_POINTERS pSectionObjectPointer;     //////////////////////

      SKillStripFileAttributes( FileHandle);          //去掉只读属性，才能删除只读文件

      ntStatus = ObReferenceObjectByHandle(FileHandle,

          DELETE,

          *IoFileObjectType,

          KernelMode,

          &fileObject,

          NULL);

      if (!NT_SUCCESS(ntStatus))

      {

          return FALSE;

      }

      DeviceObject = IoGetRelatedDeviceObject(fileObject);//如果 NTFS.sys 驱动建立的设备对象上没有附加的设备对象的话，就返回 NTFS.sys 建立的设备对象

                                                       //否则返回的是这个设备对象的 highest level 设备对象。

      Irp = IoAllocateIrp(DeviceObject->StackSize, TRUE);//如果没有附加，StackSize 为 7

      if (Irp == NULL)

      {

          ObDereferenceObject(fileObject);

          return FALSE;

      }

      KeInitializeEvent(&event1, SynchronizationEvent, FALSE);

      FileInformation.DeleteFile = TRUE;

      Irp->AssociatedIrp.SystemBuffer = &FileInformation;

      Irp->UserEvent = &event1;

      Irp->UserIosb = &ioStatus;

      Irp->Tail.Overlay.OriginalFileObject = fileObject;

      Irp->Tail.Overlay.Thread = (PETHREAD)KeGetCurrentThread();

      Irp->RequestorMode = KernelMode;

      irpSp = IoGetNextIrpStackLocation(Irp);            //得到文件系统 NTFS.sys 驱动的设备 IO_STACK_LOCATION

      irpSp->MajorFunction = IRP_MJ_SET_INFORMATION;

      irpSp->DeviceObject = DeviceObject;

      irpSp->FileObject = fileObject;

      irpSp->Parameters.SetFile.Length = sizeof(FILE_DISPOSITION_INFORMATION);

      irpSp->Parameters.SetFile.FileInformationClass = FileDispositionInformation;

      irpSp->Parameters.SetFile.FileObject = fileObject;

      IoSetCompletionRoutine(

              Irp,

              SkillSetFileCompletion,

              &event1,

              TRUE,

              TRUE,

              TRUE);

//再加上下面这三行代码 ，MmFlushImageSection    函数通过这个结构来检查是否可以删除文件。
```



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
