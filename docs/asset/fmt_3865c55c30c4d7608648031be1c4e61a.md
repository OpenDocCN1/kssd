
# 超级菜鸟组装别人的遍历进程，根据 PID ,结束进程

> 原文：[`www.pediy.com/kssd/pediy10/92084.html`](https://www.pediy.com/kssd/pediy10/92084.html)

这些代码没一个是我自己写的，全是组装的别人的！真有点惭愧！`![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")`  

一部分抄的，一部分问的！  

还有好多方法，我慢慢找！组装好了和大家分享！`![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")`  

感谢看雪的大牛！  

代码:

```c
#include "Driver.h"
#pragma comment(lib,"ntdll.lib")
extern "C" 
NTSYSAPI
NTSTATUS
NTAPI
ZwOpenProcess(
        OUT PHANDLE ProcessHandle,
        IN ACCESS_MASK DesiredAccess,
        IN POBJECT_ATTRIBUTES ObjectAttributes,
        IN PCLIENT_ID ClientId OPTIONAL
        );

extern "C"
NTSYSAPI
NTSTATUS
NTAPI
ZwTerminateProcess(
        IN HANDLE ProcessHandle OPTIONAL,
        IN NTSTATUS ExitStatus
        ); 

bool TerminateProcess(IN ULONG);

VOID GetAllProcess(ULONG);
#pragma INITCODE
VOID ProcessTest()
{
  NTSTATUS status = STATUS_SUCCESS;
  ULONG Address;
  ULONG oldAddress;

  //得到当前进程的地址
  Address = (ULONG)PsGetCurrentProcess();
  //得到 EPROCESS 中链表的偏移
  Address += 0x88;
  //用 oldAddress 保存当前进程链表的地址
  oldAddress = Address;

  //遍历进程链表
  do
  {
    GetAllProcess(Address);
    //让 Address 指向当前链表的下一个进程链表的地址(注意是 Flink)
    Address = *(ULONG*)Address;

  }while( oldAddress!=Address );//当得到的链表地址与保存的地址相等说明遍历完整个线程链表，返回
}

VOID GetAllProcess(ULONG Address)
{
  //得到对应的 EPROCESS 结构
  NTSTATUS status1; 
  Address -= 0x88;
 // ULONG *pid;
  //pid =(ULONG*)Address+0x084;
  //=EPROCESS 偏移 0x174 为 ImageFileName(进程名)
  DbgPrint("PID %8d\\n",*(int *)(Address+0x84));
  if(*(int *)(Address+0x84)==1234){   //进程 pid
        TerminateProcess(1234);     
  }
  DbgPrint("ProcessName %s \\n",(char*)(Address+0x174));
  //EPROCESS 偏移 0x18 为页目录物理地址
  DbgPrint("Process Context %d \\n",*(ULONG*)(Address+0x18)); 

}
bool TerminateProcess(IN ULONG pid)
{
  bool ret = false;
  HANDLE ProcessHandle;
  CLIENT_ID ClientId;
  OBJECT_ATTRIBUTES ObjectAttributes;
  ClientId.UniqueProcess = (HANDLE)pid;
  ClientId.UniqueThread = (HANDLE)0;
  ObjectAttributes.Attributes = 0;
  ObjectAttributes.Length = sizeof(OBJECT_ATTRIBUTES);
  ObjectAttributes.ObjectName = NULL;
  ObjectAttributes.RootDirectory = NULL;
  ObjectAttributes.SecurityDescriptor = NULL;
  ObjectAttributes.SecurityQualityOfService = NULL;
  NTSTATUS ntstatus = ZwOpenProcess(&ProcessHandle, PROCESS_ALL_ACCESS, &ObjectAttributes, &ClientId);
  if (NT_SUCCESS(ntstatus))
  {
    DbgPrint("OpenProcess Successed!\\n");
    ntstatus = ZwTerminateProcess(ProcessHandle, 0);
    if (NT_SUCCESS(ntstatus))
    {
      DbgPrint("TerminateProcess Successed!\\n");
      ret = true;
    }
    ZwClose(ProcessHandle);
  }
  return ret;
} 

#pragma INITCODE
extern "C" NTSTATUS DriverEntry (
      IN PDRIVER_OBJECT pDriverObject,
      IN PUNICODE_STRING pRegistryPath  ) 
{
  NTSTATUS status;
  KdPrint(("Enter DriverEntry\\n"));

  //注册其他驱动调用函数入口
  pDriverObject->DriverUnload = HelloDDKUnload;
  pDriverObject->MajorFunction[IRP_MJ_CREATE] = HelloDDKDispatchRoutine;
  pDriverObject->MajorFunction[IRP_MJ_CLOSE] = HelloDDKDispatchRoutine;
  pDriverObject->MajorFunction[IRP_MJ_WRITE] = HelloDDKDispatchRoutine;
  pDriverObject->MajorFunction[IRP_MJ_READ] = HelloDDKDispatchRoutine;
  
  //创建驱动设备对象
  status = CreateDevice(pDriverObject);

  ProcessTest();
  KdPrint(("DriverEntry end\\n"));
  return status;
}

/************************************************************************
* 函数名称:CreateDevice
* 功能描述:初始化设备对象
* 参数列表:
      pDriverObject:从 I/O 管理器中传进来的驱动对象
* 返回 值:返回初始化状态
************************************************************************/
#pragma INITCODE
NTSTATUS CreateDevice (
    IN PDRIVER_OBJECT  pDriverObject) 
{
  NTSTATUS status;
  PDEVICE_OBJECT pDevObj;
  PDEVICE_EXTENSION pDevExt;
  
  //创建设备名称
  UNICODE_STRING devName;
  RtlInitUnicodeString(&devName,L"\\\\Device\\\\MyDDKDevice");
  
  //创建设备
  status = IoCreateDevice( pDriverObject,
            sizeof(DEVICE_EXTENSION),
            &(UNICODE_STRING)devName,
            FILE_DEVICE_UNKNOWN,
            0, TRUE,
            &pDevObj );
  if (!NT_SUCCESS(status))
    return status;

  pDevObj->Flags |= DO_BUFFERED_IO;
  pDevExt = (PDEVICE_EXTENSION)pDevObj->DeviceExtension;
  pDevExt->pDevice = pDevObj;
  pDevExt->ustrDeviceName = devName;
  //创建符号链接
  UNICODE_STRING symLinkName;
  RtlInitUnicodeString(&symLinkName,L"\\\\??\\\\HelloDDK");
  pDevExt->ustrSymLinkName = symLinkName;
  status = IoCreateSymbolicLink( &symLinkName,&devName );
  if (!NT_SUCCESS(status)) 
  {
    IoDeleteDevice( pDevObj );
    return status;
  }

  return STATUS_SUCCESS;
}

/************************************************************************
* 函数名称:HelloDDKUnload
* 功能描述:负责驱动程序的卸载操作
* 参数列表:
      pDriverObject:驱动对象
* 返回 值:返回状态
************************************************************************/
#pragma PAGEDCODE
VOID HelloDDKUnload (IN PDRIVER_OBJECT pDriverObject) 
{
  PDEVICE_OBJECT  pNextObj;
  KdPrint(("Enter DriverUnload\\n"));
  pNextObj = pDriverObject->DeviceObject;
  while (pNextObj != NULL) 
  {
    PDEVICE_EXTENSION pDevExt = (PDEVICE_EXTENSION)
      pNextObj->DeviceExtension;

    //删除符号链接
    UNICODE_STRING pLinkName = pDevExt->ustrSymLinkName;
    IoDeleteSymbolicLink(&pLinkName);
    pNextObj = pNextObj->NextDevice;
    IoDeleteDevice( pDevExt->pDevice );
  }
}

/************************************************************************
* 函数名称:HelloDDKDispatchRoutine
* 功能描述:对读 IRP 进行处理
* 参数列表:
      pDevObj:功能设备对象
      pIrp:从 IO 请求包
* 返回 值:返回状态
************************************************************************/
#pragma PAGEDCODE
NTSTATUS HelloDDKDispatchRoutine(IN PDEVICE_OBJECT pDevObj,
            IN PIRP pIrp) 
{
  KdPrint(("Enter HelloDDKDispatchRoutine\\n"));
  NTSTATUS status = STATUS_SUCCESS;
  // 完成 IRP
  pIrp->IoStatus.Status = status;
  pIrp->IoStatus.Information = 0;  // bytes xfered
  IoCompleteRequest( pIrp, IO_NO_INCREMENT );
  KdPrint(("Leave HelloDDKDispatchRoutine\\n"));
  return status;
}

```
