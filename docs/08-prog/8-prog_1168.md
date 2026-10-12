
# 内核无 HOOK 文件防删除，可以过冰刃，xuetr，easydelete

> 原文：[`www.pediy.com/kssd/pediy11/112593.html`](https://www.pediy.com/kssd/pediy11/112593.html)

本来是打算写暴力删除文件的程序的，结果意外发现，只需要在内核发送 irp 打开一个文件，之后不关闭 Object，就拒绝其他程序访问了，easydelete 这个工具还是比较强的，不过也不能删除，关键是不挂钩任何函数，不修该系统内核，看属性的时候只能看到一个常规，其它的什么都看不到，当然了 360（7.0）版本的文件粉碎机也是搞不定的,其它工具没试 ，大家可以自己试试玩玩，打开文件使用炉子的代码：

```c
NTSTATUS
IoCompletionRoutine(
     IN PDEVICE_OBJECT DeviceObject,
     IN PIRP Irp,
     IN PVOID Context)
{
*Irp->UserIosb = Irp->IoStatus;
if (Irp->UserEvent)
   KeSetEvent(Irp->UserEvent, IO_NO_INCREMENT, 0);
if (Irp->MdlAddress)
{
   IoFreeMdl(Irp->MdlAddress);
   Irp->MdlAddress = NULL;
}
IoFreeIrp(Irp);
return STATUS_MORE_PROCESSING_REQUIRED;
}

NTSTATUS
IrpCreateFile(
     OUT PFILE_OBJECT *FileObject,
     IN ACCESS_MASK DesiredAccess,
     IN PUNICODE_STRING FilePath,
     OUT PIO_STATUS_BLOCK IoStatusBlock,
     IN PLARGE_INTEGER AllocationSize OPTIONAL,
     IN ULONG FileAttributes,
     IN ULONG ShareAccess,
     IN ULONG CreateDisposition,
     IN ULONG CreateOptions,
     IN PVOID EaBuffer OPTIONAL,
     IN ULONG EaLength)
{
NTSTATUS ntStatus;

HANDLE hFile;
PFILE_OBJECT pFile, _FileObject;
UNICODE_STRING UniDeviceNameString;
OBJECT_ATTRIBUTES ObjectAttributes;
PDEVICE_OBJECT DeviceObject, RealDevice;

PIRP Irp;
KEVENT kEvent;
PIO_STACK_LOCATION IrpSp;
ACCESS_STATE AccessState;
AUX_ACCESS_DATA AuxData;
IO_SECURITY_CONTEXT SecurityContext;

WCHAR DeviceNameString[]=L"\\DosDevices\\*:\\";

if(FilePath->Length < 6)
   return STATUS_INVALID_PARAMETER;
// \\??\c:\xxxx

DeviceNameString[12]=FilePath->Buffer[0];

RtlInitUnicodeString( &UniDeviceNameString, DeviceNameString);
InitializeObjectAttributes(&ObjectAttributes, &UniDeviceNameString, OBJ_KERNEL_HANDLE, NULL, NULL);

ntStatus = IoCreateFile(&hFile,
   GENERIC_READ|SYNCHRONIZE,
   &ObjectAttributes,
   IoStatusBlock,
   NULL,
   FILE_ATTRIBUTE_NORMAL,
   FILE_SHARE_READ|FILE_SHARE_WRITE|FILE_SHARE_DELETE,
   FILE_OPEN,
   FILE_SYNCHRONOUS_IO_NONALERT,
   NULL,
   0,
   CreateFileTypeNone,
   NULL,
   IO_NO_PARAMETER_CHECKING);

if(!NT_SUCCESS(ntStatus))
{
   KdPrint(("IrpCreateFile: IoCreateFile 0x%X.\n",ntStatus));
   return ntStatus;
}
       RecoverOneKernelRoutine("ObReferenceObjectByHandle");
ntStatus = ObReferenceObjectByHandle(hFile,
   FILE_READ_ACCESS, // ACCESS_MASK
   *IoFileObjectType,
   KernelMode,
   (PVOID*)&pFile,
   0);

NtClose(hFile);

if(!NT_SUCCESS(ntStatus))
{
   KdPrint(("IrpCreateFile: ObReferenceObjectByHandle 0x%X.\n",ntStatus));
   return ntStatus;
}

DeviceObject = pFile->Vpb->DeviceObject;
RealDevice = pFile->Vpb->RealDevice;
ObDereferenceObject(pFile);

InitializeObjectAttributes(&ObjectAttributes, NULL, OBJ_CASE_INSENSITIVE, 0, NULL);

ntStatus = ObCreateObject(KernelMode,
   *IoFileObjectType,
   &ObjectAttributes,
   KernelMode,
   NULL,
   sizeof(FILE_OBJECT),
   0,
   0,
   (PVOID*)&_FileObject);

if(!NT_SUCCESS(ntStatus))
{
   KdPrint(("IrpCreateFile: ObCreateObject 0x%X.\n",ntStatus));
   return ntStatus;
}

Irp = IoAllocateIrp(DeviceObject->StackSize, FALSE);
if(Irp == NULL)
{
   KdPrint(("IrpCreateFile: IoAllocateIrp 0x%X.\n",ntStatus));
   ObDereferenceObject(_FileObject);
   return STATUS_INSUFFICIENT_RESOURCES;
}

KeInitializeEvent(&kEvent, SynchronizationEvent, FALSE);

RtlZeroMemory(_FileObject, sizeof(FILE_OBJECT));
_FileObject->Type = IO_TYPE_FILE;
_FileObject->Size = sizeof(FILE_OBJECT);
_FileObject->DeviceObject = RealDevice;
_FileObject->Flags = FO_SYNCHRONOUS_IO;
////////\\??\c:\xxxxx
RtlInitUnicodeString( &_FileObject->FileName, &FilePath->Buffer[2]);
KdPrint(("准备打开文件：%ws\n",_FileObject->FileName.Buffer));
KeInitializeEvent(&_FileObject->Lock, SynchronizationEvent, FALSE);
KeInitializeEvent(&_FileObject->Event, NotificationEvent, FALSE);

RtlZeroMemory(&AuxData, sizeof(AUX_ACCESS_DATA));
ntStatus = SeCreateAccessState( &AccessState,
   &AuxData,
   DesiredAccess,
   IoGetFileObjectGenericMapping());

if (!NT_SUCCESS(ntStatus))
{
   KdPrint((" IrpCreateFile: SeCreateAccessState 0x%X.\n",ntStatus));
   IoFreeIrp(Irp);
   ObDereferenceObject(_FileObject);
   return ntStatus;
}

SecurityContext.SecurityQos = NULL;
SecurityContext.AccessState = &AccessState;
SecurityContext.DesiredAccess = DesiredAccess;
SecurityContext.FullCreateOptions = 0;

Irp->MdlAddress = NULL;
Irp->AssociatedIrp.SystemBuffer = EaBuffer;
Irp->Flags = IRP_CREATE_OPERATION|IRP_SYNCHRONOUS_API;
Irp->RequestorMode = KernelMode;
Irp->UserIosb = IoStatusBlock;
Irp->UserEvent = &kEvent;
Irp->PendingReturned = FALSE;
Irp->Cancel = FALSE;
Irp->CancelRoutine = NULL;
Irp->Tail.Overlay.Thread = PsGetCurrentThread();
Irp->Tail.Overlay.AuxiliaryBuffer = NULL;
Irp->Tail.Overlay.OriginalFileObject = _FileObject;

IrpSp = IoGetNextIrpStackLocation(Irp);
IrpSp->MajorFunction = IRP_MJ_CREATE;
IrpSp->DeviceObject = DeviceObject;
IrpSp->FileObject = _FileObject;
IrpSp->Parameters.Create.SecurityContext = &SecurityContext;
IrpSp->Parameters.Create.Options = (CreateDisposition << 24) | CreateOptions;
IrpSp->Parameters.Create.FileAttributes = (USHORT)FileAttributes;
IrpSp->Parameters.Create.ShareAccess = (USHORT)ShareAccess;
IrpSp->Parameters.Create.EaLength = EaLength;

IoSetCompletionRoutine(Irp, IoCompletionRoutine, 0, TRUE, TRUE, TRUE);
//add 
RecoverIopfCompleteRequest();
ntStatus = IofCallDriverEx(DeviceObject, Irp);
if(ntStatus == STATUS_PENDING)
   KeWaitForSingleObject(&kEvent, Executive, KernelMode, TRUE, 0);

ntStatus = IoStatusBlock->Status;


if(!NT_SUCCESS(ntStatus))
{
   KdPrint(("IrpCreateFile: IoCallDriver 0x%X.\n",ntStatus));
   _FileObject->DeviceObject = NULL;
   ObDereferenceObject(_FileObject);
}
else
{
   InterlockedIncrement((volatile LONG *)&_FileObject->DeviceObject->ReferenceCount);
   if (_FileObject->Vpb)
    InterlockedIncrement((volatile LONG *)&_FileObject->Vpb->ReferenceCount);
   *FileObject = _FileObject;
}

return ntStatus;
}
//UNICODE_STRING Name;
//IO_STATUS_BLOCK IoBlock;
//RtlInitUnicodeString(&Name,L"C:\\test1.exe");
//Status=IrpCreateFile(&FileObj,GENERIC_READ|DELETE,&Name,&IoBlock,0,FILE_ATTRIBUTE_NORMAL,FILE_SHARE_READ|FILE_SHARE_WRITE|FILE_SHARE_DELETE,FILE_OPEN,0,0,0);
```

上面是测试代码，打开之后不`ObDereferenceObject`，直接返回，之后文件就被保护起来了
