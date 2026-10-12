
# 逆向 rainsafe.sys

> 原文：[`www.pediy.com/kssd/pediy12/128306.html`](https://www.pediy.com/kssd/pediy12/128306.html)

【文章标题】: 逆向 `rainsafe.sys`  

【文章作者】: `majinxin`  

【作者邮箱】: `majinxin2003@126.com`  

【下载地址】: 附件  

【作者声明】: 我知道有源码，纯属为了学习  

禹盾 hips 是一款绿色的主机入侵防御系统,它从驱动层保护系统关键位置,拦截进程,注册表,驱动 等危险操作,从而达到保护电脑的目标。  

从网上下来这个软件后，把它的驱动 A 掉，发现它 hook 了四个函数分别是 `ZwCreateProcessEx`、`ZwLoadDriver`、`ZwSetValueKey`、`ZwSetSystemTime`。然后把这驱动，进程，注册表信息通过 `DeviceControl` 传递到用户层，但是后来在网上看到，原来早已经开源了，![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")不过权当是学习了吧![](img/f3773294bf3a914c9a8c0c242528a003.jpg "使眼色")，有些函数因为完成的是类似的工作就没逆出来，同时，我也上传上它的源码和我分析的 IDB，有兴趣的朋友可以参考下。  

首先非常感谢禹盾的作者大方地发布出源码！！![](img/081868d20f20fe7a9b4b88ab0d0d33c6.jpg "狂笑")![](img/081868d20f20fe7a9b4b88ab0d0d33c6.jpg "狂笑")![](img/081868d20f20fe7a9b4b88ab0d0d33c6.jpg "狂笑")  

在最新发布的源码中可以看到 hook 了更多的函数：

```
IoVolumeDeviceToDosName=0;

ZwCreateProcess;

ZwCreateProcessEx;

NtCreateUserProcess;

ZwSetValueKey;

ZwLoadDriver;

ZwSetSystemTime;

NtSetSystemInformation;

NtSystemDebugControl;

NtOpenProcess;

NtOpenThread;

NtWriteFile;

NtUserSetInformationThread;
```

以下是我 A 出来的代码：![](img/906f120cfe711ccbc8463fd6eb86eee9.jpg "尴尬")

```c
#include <ntddk.h>

ULONG g_ProcessOffset = 0;

PEPROCESS g_SelfPEProcess;

BOOL IsCreateProcessHooked;

BOOL IsSetValueKeyHooked;

BOOL IsLoadDricerHooked;

BOOL IsSetSystemTimeHooked;

ULONG g_NtProcessAddr;

ULONG g_NtSetValueKeyAddr;

ULONG g_NtLoadDriverAddr;

ULONG g_NtSetSystemTimeAddr;

ULONG g_CreateProcessID;

ULONG g_SetValueKeyID;

ULONG g_LoadDriverID;

ULONG g_SetSystemTimeID;

KEVENT Event;

typedef struct _SERVICE_DESCRIPTOR_TABLE

{

  PVOID   ServiceTableBase;

  PULONG  ServiceCounterTableBase;

  ULONG   NumberOfService;

  ULONG   ParamTableBase;

}SERVICE_DESCRIPTOR_TABLE,*PSERVICE_DESCRIPTOR_TABLE;

extern PSERVICE_DESCRIPTOR_TABLE  KeServiceDescriptorTable;

VOID WPOFF()

{

  __asm{

    cli

      push eax

      mov  eax,cr0

      and  eax,not 10000h

      mov  cr0,eax

      pop eax

  }

}

VOID WPON()

{

  __asm{

    push eax

      mov  eax,cr0

      or   eax,10000h

      mov  cr0,eax

      pop eax

      sti          

  }

}

int __cdecl GetProcessOffset()

{

  //get the offset from EPROCESS to ProcessName

  PEPROCESS pp = IoGetCurrentProcess();

  int i;

  for(i = pp; i < 3000h; i++)

  {

    if(strncmp(i, "System", 6) == 0)

      g_ProcessOffset = i - pp;

  }

}

int __stdcall ZeroMemory(void *addr, unsigned int length)

{

  memset(addr, 0, length);

}

int __stdcall GetProcessAddr(PCSZ FunctionName, LSA_UNICODE_STRING *NtdllPath)

{

  //Get the function address from the export table of the ntdll.dll

  PHANDLE FileHandle;

  ACCESS_MASK DesiredAccess = 0x10020;

  POBJECT_ATTRIBUTES  ObjectAttributes;

  PIO_STATUS_BLOCK  IoStatusBlock;

  ULONG  ShareAccess = 1;

  ULONG  OpenOptions = 0x20h;

  ObjectAttributes.ObjectName = NtdllPath;

  ObjectAttributes.Length = 0x18;

  ObjectAttributes.RootDirectory = 0;

  ObjectAttributes.Attributes = 0x40;

  PHANDLE  SectionHandle;

  PLARGE_INTEGER  MaximumSize;

  ULONG  SectionPageProtection;

  ULONG  AllocationAttributes;

  PHANDLE SectionHandle;

  HANDLE  ProcessHandle = -1;

  PVOID  *BaseAddress;

  ULONG*  ZeroBits = 0;

  SIZE_T  CommitSize = 0x3E8;

  SECTION_INHERIT  InheritDisposition = 1;

  PSIZE_T  ViewSize;

  ULONG  AllocationType = 0x100000;

  ULONG  Win32Protect = 4;

  PLARGE_INTEGER  SectionOffset = 0;

  int i = 0;

  ULONG* pPeNtHeader;

  IMAGE_EXPORT_DIRECTORY *Exp;

  PSTRING pFunctionNameString;

  ULONG* pNameAddr;

  char* Name;

  PSTRING pFindFunctionString;

  ULONG Oridinal;

  ULONG ExportID;

  ULONG funcAddr;

  ZwOpenFile(FileHandle, DesiredAccess, ObjectAttributes, IoStatusBlock, ShareAccess, OpenOptions);

  AllocationAttributes = 0x1000000;

  SectionPageProtection = 0x10;

  MaximumSize = 0;

  ObjectAttributes.ObjectName = 0;

  DesiredAccess = 0xF001F;

  ZwCreateSection(SectionHandle, DesiredAccess, ObjectAttributes, MaximumSize, SectionPageProtection, AllocationAttributes, FileHandle); 

  ZwMapViewOfSection(SectionHandle, ProcessHandle, *BaseAddress, ZeroBits, CommitSize, SectionOffset, ViewSize, InheritDisposition,

    AllocationType, Win32Protect);

  ZwClose(FileHandle);

  pPeNtHeader = *(BaseAddress + 0x3c);

  Exp = *((IMAGE_EXPORT_DIRECTORY *)(BaseAddress + *(pPeNtHeader + 0x78)));

  RtlInitString(pFindFunctionString, FunctionName);

  pNameAddr = BaseAddress + Exp->AddressOfNames;

  for(i  = 0; i < Exp->NumberOfFunctions; i++)

  {

    Name = (char*)(BaseAddress + *(pNameAddr + i * 4));

    RtlInitString(pFunctionNameString, Name);

    if(!RtlCompareString(pFindFunctionString, pFunctionNameString, 1))

    {

      Oridinal = *(ULONG*)((BaseAddress + Exp->AddressOfNameOrdinals) + i * 2)

      ExportID = Exp->Base + Oridinal;

      funcAddr = BaseAddress + *(ULONG*)(Exp->AddressOfFunctions + (ExportID-1) * 4);

      ZwClose(SectionHandle);

      return funcAddr;

    }

  }

  ZwClose(SectionHandle);

  return 0;

}

NTSTATUS __stdcall GetSectionFilePath(HANDLE SectionHandle, int OutputBuffer)

{

  HANDLE  Handle = SectionHandle;

  ACCESS_MASK  DesiredAccess = 0;

  POBJECT_TYPE  ObjectType = 0;

  KPROCESSOR_MODE  AccessMode = 0;

  PFILE_OBJECT  FileObject;

  POBJECT_HANDLE_INFORMATION  HandleInformation = 0;

  NTSTATUS ntSatus;

  POOL_TYPE  PoolType = 1;

  SIZE_T  NumberOfBytes = 0x200;

  ULONG Tag = 0x206B6444;

  PUNICODE_STRING pString1;

  PUNICODE_STRING  DosName;

  PANSI_STRING pString2;

  PFILE_OBJECT FilePointer;

  ntSatus = ObReferenceObjectByHandle(Handle, DesiredAccess, ObjectType, AccessMode, &FileObject, HandleInformation);

  if(ntSatus != 0) return ntSatus;

  pString1->Buffer = ExAllocatePoolWithTag(PoolType, NumberOfBytes, Tag);

  pString1->MaximumLength = 0x200;

  FilePointer = *(*(FileObject->SectionObjectPointer->DataSectionObject) +0x24)

  ObReferenceObjectByPointer(FilePointer, 0, 0, 0);

  RtlVolumeDeviceToDosName(FilePointer->DeviceObject, DosName);

  RtlCopyUnicodeString(pString1, DosName);

  RtlAppendUnicodeStringToString(pString1, FilePointer->FileName);

  ObfDereferenceObject(FilePointer);

  RtlUnicodeStringToAnsiString(pString1, pString2);

  RtlCopyMemory(OutputBuffer, pString2->Buffer, pString2->Length);

  RtlFreeAnsiString(pString2);

  ExFreePoolWithTag(pString1->Buffer, Tag);

  return 1;

}

char __stdcall sub_107D6(HANDLE Object, int a2)

{

}

int __stdcall ToAnsiString(PCWSTR ProcessPath, ULONG OutBuff)

{

  PUNICODE_STRING ProcessPathString;

  PANSI_STRING OutSting;

  RtlInitUnicodeString(&ProcessPathString, ProcessPath);

  OutSting->Length = 0;

  OutSting->MaximumLength = 256;

  OutSting->Buffer = OutBuff;

  RtlUnicodeStringToAnsiString((PANSI_STRING)&OutSting, &ProcessPathString, 0);

  return 0;

}

signed int __stdcall GetProcessPath(int Buff)

{

  PEPROCESS pProcess = IoGetCurrentProcess();

  ToAnsiString(pProcess->Peb->ProcessParameters->ImagePathName.Buff, Buff);

}

int __stdcall CombineAndCopyString(char *PProcessPathA, const char *DriveNameString)

{

}

signed int __stdcall MyCreateProcess(int ProcessHandle, int DesiredAccess, int ObjectAttributes, int ParentProcess, int InheritObjectTable, HANDLE SectionHandle, int DebugPort, int ExceptionPort, int a9)

{

  char *SectionFilePath;

  char *ProcessPath;

  if (g_SelfPEProcess == IoGetCurrentProcess())
```



```c
    return g_NtProcessAddr(ProcessHandle, DesiredAccess, ObjectAttributes, ParentProcess, InheritObjectTable, SectionHandle, DebugPort, ExceptionPort, a9);

  ZeroMemory(ProcessPath, 0x100u);

  GetSectionFilePath(SectionHandle, SectionFilePath);

  GetProcessPath(ProcessPath);

  CombineAndCopyString(ProcessPath, SectionFilePath);

  return g_NtProcessAddr(ProcessHandle, DesiredAccess, ObjectAttributes, ParentProcess, InheritObjectTable, SectionHandle, DebugPort, ExceptionPort, a9);

}

signed int __stdcall MySetValueKey(HANDLE KeyHandle, PCUNICODE_STRING ValueName, int TitleIndex, int Type, int Data, int DataSize)

{

  //类似 MyCreateProcess，取得信息传递到全局变量地址，随后传递给用户层

}

int __stdcall MyLoaddriver(PCUNICODE_STRING DriverName)

{

  //类似 MyCreateProcess，取得信息传递到全局变量地址，随后传递给用户层

}

struct PEPROCESS __cdecl HookCreateProcess()

{

  ULONG Address;

  if(IsCreateProcessHooked == FALSE)

  {

    g_SelfPEProcess = IoGetCurrentProcess();

    ULONG Address = (ULONG)KeServiceDescriptorTable->ServiceTableBase + g_CreateProcessID * 4;

    WPOFF();

    *((ULONG*)Address) = (ULONG)MyCreateProcess;

    WPON();

    IsCreateProcessHooked = TRUE;

  }

  return g_SelfPEProcess;

}

unsigned int __cdecl UnhookCreateProcess()

{

  if (IsCreateProcessHooked == TRUE)

  {

    ULONG Address = (ULONG)KeServiceDescriptorTable->ServiceTableBase + g_CreateProcessID * 4;

    WPOFF();

    *((ULONG*)Address) = (ULONG)g_NtProcessAddr;

    WPON();

  }

  IsCreateProcessHooked = FALSE;

  return 0;

}

struct PEPROCESS __cdecl HookSetValueKey()

{

  ULONG Address;

  if(IsSetValueKeyHooked == FALSE)

  {

    g_SelfPEProcess = IoGetCurrentProcess();

    ULONG Address = (ULONG)KeServiceDescriptorTable->ServiceTableBase + g_SetValueKeyID * 4;

    WPOFF();

    *((ULONG*)Address) = (ULONG)MySetValueKey;

    WPON();

    IsCreateProcessHooked = TRUE;

  }

  return g_SelfPEProcess;

}

unsigned int __cdecl UnhookSetValueKey()

{

  if (IsSetValueKeyHooked == TRUE)

  {

    ULONG Address = (ULONG)KeServiceDescriptorTable->ServiceTableBase + g_SetValueKeyID * 4;

    WPOFF();

    *((ULONG*)Address) = (ULONG)g_NtSetValueKeyAddr;

    WPON();

  }

  IsCreateProcessHooked = FALSE;

  return 0;

}

struct PEPROCESS __cdecl HookLoadDriver()

{

  ULONG Address;

  if(IsLoadDricerHooked == FALSE)

  {

    g_SelfPEProcess = IoGetCurrentProcess();

    ULONG Address = (ULONG)KeServiceDescriptorTable->ServiceTableBase + g_LoadDriverID * 4;

    WPOFF();

    *((ULONG*)Address) = (ULONG)MyLoaddriver;

    WPON();

    IsLoadDricerHooked = TRUE;

  }

  return g_SelfPEProcess;

}

unsigned int __cdecl UnhookLoadDriver()

{

  if (IsLoadDricerHooked == TRUE)

  {

    ULONG Address = (ULONG)KeServiceDescriptorTable->ServiceTableBase + g_LoadDriverID * 4;

    WPOFF();

    *((ULONG*)Address) = (ULONG)g_NtLoadDriverAddr;

    WPON();

  }

  IsLoadDricerHooked = FALSE;

  return 0;

}

struct PEPROCESS __cdecl HookSetSystemTime()

{

  ULONG Address;

  if(IsSetSystemTimeHooked == FALSE)

  {

    g_SelfPEProcess = IoGetCurrentProcess();

    ULONG Address = (ULONG)KeServiceDescriptorTable->ServiceTableBase + g_SetSystemTimeID * 4;

    WPOFF();

    *((ULONG*)Address) = (ULONG)sub_10CD2;

    WPON();

    IsSetSystemTimeHooked = TRUE;

  }

  return g_SelfPEProcess;

}

unsigned int __cdecl UnhookSetSystemTime()

{

  if (IsSetSystemTimeHooked == TRUE)

  {

    ULONG Address = (ULONG)KeServiceDescriptorTable->ServiceTableBase + g_SetSystemTimeID * 4;

    WPOFF();

    *((ULONG*)Address) = (ULONG)g_NtSetSystemTimeAddr;

    WPON();

  }

  IsSetSystemTimeHooked = FALSE;

  return 0;

}

int __stdcall DispatchCreate(PDEVICE_OBJECT DeviceObject, PIRP Irp)

{

  UNICODE_STRING DestinationString;

  Irp->IoStatus.Information = 0;

  Irp->IoStatus.Status = 0;

  IofCompleteRequest(Irp, 0);

  KeInitializeEvent(&Event, SynchronizationEvent, 1);

  RtlInitUnicodeString(&DestinationString, L"\\Device\\HarddiskVolume1\\Windows\\System32\\ntdll.dll");

  g_CreateProcessID = GetProcessAddr("ZwCreateProcessEx", &DestinationString) + 1;

  g_NtProcessAddr = KeServiceDescriptorTable->ServiceTableBase + g_CreateProcessID * 4;

  g_SetValueKeyID = *((char*)&ZwSetValueKey + 1);

  g_NtSetValueKeyAddr = KeServiceDescriptorTable->ServiceTableBase + g_SetValueKeyID * 4;

  g_LoadDriverID = GetProcessAddr("ZwLoadDriver", &DestinationString) + 1;

  g_NtLoadDriverAddr = KeServiceDescriptorTable->ServiceTableBase + g_LoadDriverID * 4;

  g_SetSystemTimeID = GetProcessAddr("ZwSetSystemTime", &DestinationString) + 1;

  g_NtSetSystemTimeAddr = KeServiceDescriptorTable->ServiceTableBase + g_SetSystemTimeID * 4;

  return 0;

}

int __stdcall DispatchClose(PDEVICE_OBJECT DeviceObject, PIRP Irp)

{

  Irp->IoStatus.Information = 0;

  Irp->IoStatus.Status = 0;

  IofCompleteRequest(Irp, 0);

  return 0;

}

int __stdcall DispatchDeviceControl(PDEVICE_OBJECT DeviceObject, PVOID Irp)

{

  //与用户层交互信息，略

}

void __stdcall DriverUnload(PDRIVER_OBJECT driver)

{

  UNICODE_STRING DestinationString;

  UnhookCreateProcess();

  UnhookSetValueKey();

  UnhookLoadDriver();

  UnhookSetSystemTime();

  RtlInitUnicodeString(&DestinationString, L"\\DosDevices\\rainsafe");

  IoDeleteSymbolicLink(&DestinationString);

  IoDeleteDevice(driver->DeviceObject);

}

NTSTATUS __stdcall DriverEntry(PDRIVER_OBJECT DriverObject, PUNICODE_STRING RegistryPath)

{

  NTSTATUS ntStatus;

  PDEVICE_OBJECT DeviceObject = NULL;

  UNICODE_STRING ntDeviceName, SymbolicLinkName;

  BOOLEAN  fSymbolicLink = FALSE;

  RtlInitUnicodeString(&ntDeviceName, L"\\Device\\rainsafe");

  ntStatus = IoCreateDevice(DriverObject, 4, &ntDeviceName, 0x8000u, 0, 1u, &DeviceObject);

  if (!NT_SUCCESS(ntStatus))

  {

    goto __failed;

  }

  RtlInitUnicodeString(&SymbolicLinkName, L"\\DosDevices\\rainsafe")

  ntStatus = IoCreateSymbolicLink(&SymbolicLinkName, &ntDeviceName);

  if (!NT_SUCCESS(ntStatus))

  {

    goto __failed;

  }

  fSymbolicLink = TRUE;

  DriverObject->MajorFunction[IRP_MJ_CREATE]          = DispatchCreate;

  DriverObject->MajorFunction[IRP_MJ_CLOSE]           = DispatchClose;

  DriverObject->MajorFunction[IRP_MJ_DEVICE_CONTROL]  = DispatchDeviceControl;

  DriverObject->DriverUnload                          = DriverUnload;

  GetProcessOffset();

  if (!NT_SUCCESS(ntStatus))
    goto __failed;

  return ntStatus;

  __failed:

    if (fSymbolicLink)
      IoDeleteSymbolicLink(&dosDeviceName);
    if (DeviceObject)
      IoDeleteDevice(DeviceObject);

  return ntStatus;

}
```

### 附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) | `rasafesys.rar` |
| :--- | :--- |
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) | `rainsafeidb.rar` |
