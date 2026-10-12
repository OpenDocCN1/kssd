
# (开源)一个简单的 ssdthook 进程名 保护进程 兼容 2000 以后所有 x86 系统

> 原文：[`www.pediy.com/kssd/pediy11/126077.html`](https://www.pediy.com/kssd/pediy11/126077.html)

大牛已经把这些 玩的烂熟了

小小小菜鸟 把一个 修改了 无数次的代码发出来

希望 大家多多 提出意见 代码是非常的挫（本代码 修改自一个 网络博客的代码只是修改函数和部分通讯 把 `r3` `r0` 通讯 修改为 进程名 保护不需要通讯 直接加载驱动就可以了

虽说不是原原创 但是也是花了很多心血 ）

代码:



```c
/*
  Project Name:  Processes Guard
  Description:  Protection user  specified process(es)
  Date:      2010-5-5
  Version:    1.0
  Author:  Kernone     Alter:     君君寒
  Blog:      http://hi.baidu.com/kernone
  File Name:    ProcGuard.c
  Copyright(c)  Kernone Soft 2010
*/
#include <Ntifs.h>

#pragma pack(1)
typedef struct _SYSTEM_SERVICES_DESCRIPTOR_TABLE
{
  PULONG     *ServiceTableBase;
  PULONG     *ServiceCounterTableBase;  //Used in check builds only
  unsigned int  NumberOfServices;
  PULONG     *ParamTableBase;
}SSDT, *PSSDT;
#pragma pack()

typedef struct _DEVICE_EXTENSION
{
  PDEVICE_OBJECT  pDevObj;
  UNICODE_STRING  uniSymLink;
  PMDLE     pMdl;
  PULONG     pulSSDTMapped;
}DEVICE_EXTENSION, *PDEVICE_EXTENSION;

typedef NTSTATUS (__stdcall *ZWOPENPROCESS)(
                       OUT PHANDLE ProcessHandle,
                       IN ACCESS_MASK DesiredAccess,
                       IN POBJECT_ATTRIBUTES ObjectAttributes,
                       IN PCLIENT_ID PCLIENT_ID OPTIONAL
);

typedef NTSTATUS  (__stdcall *ZWOPENTHREAD) (
                         OUT PHANDLE ProcessHandle,
                         IN ACCESS_MASK DesiredAccess,
                         IN POBJECT_ATTRIBUTES ObjectAttributes,
                         IN PCLIENT_ID PCLIENT_ID OPTIONAL

);

NTSYSAPI
NTSTATUS
NTAPI 
ZwOpenProcess( OUT PHANDLE ProcessHandle,
       IN ACCESS_MASK AccessMask,                                        
       IN POBJECT_ATTRIBUTES ObjectAttributes,
              IN PCLIENT_ID ClientId);

NTSYSAPI
NTSTATUS
NTAPI 
ZwOpenThread( OUT PHANDLE ProcessHandle,
           IN ACCESS_MASK AccessMask,                                        
           IN POBJECT_ATTRIBUTES ObjectAttributes,
           IN PCLIENT_ID ClientId);



    /*Getting system service function address, the index of function locates 1 bytes offset*/
#define  SYSTEM_SERVICE(_Func)  KeServiceDescriptorTable.ServiceTableBase[*[(PULONG)((PUCHAR)_Func + 1)]
#define  SYSTEM_INDEX(_Func)    (*[(PULONG)((PUCHAR)_Func + 1))
//#define  SYSTEM_SERVICEONE(_Func)  KeServiceDescriptorTable.ServiceTableBase[*[(PULONG)((PUCHAR)_Func + 1)]
//#define  SYSTEM_INDEXONE(_Func)    (*[(PULONG)((PUCHAR)_Func + 1))

#define  IOCTL_START_PROTECTION  CTL_CODE(FILE_DEVICE_UNKNOWN, 0x800, METHOD_BUFFERED, FILE_ANY_ACCESS)
#define  C_MAXPROCNUMS     12

//Global variable
//__declspec(dllimport) SSDT  KeServiceDescriptorTable;
__declspec(dllimport) SSDT  KeServiceDescriptorTable;
ZWOPENPROCESS        ZwOpenProcessReal;
ZWOPENTHREAD         ZwOpenThreadReal;

ULONG  ulPIDs[C_MAXPROCNUMS];

DRIVER_UNLOAD  DriverUnload;
DRIVER_DISPATCH  DispatchDevOpen, DispatchDevCtl;
NTSTATUS ZwOpenProcessHook(PHANDLE ProcessHandle, ACCESS_MASK DesiredAccess, POBJECT_ATTRIBUTES ObjectAttributes,PCLIENT_ID ClientId);
NTSTATUS ZwOpenThreadHook(PHANDLE ProcessHandle, ACCESS_MASK DesiredAccess, POBJECT_ATTRIBUTES ObjectAttributes,PCLIENT_ID ClientId);


NTSTATUS DriverEntry(PDRIVER_OBJECT pDriverObj, PUNICODE_STRING pRegistryPath)
{
  PDEVICE_OBJECT    pDevObj;
  PDEVICE_EXTENSION  pDevExt;
  UNICODE_STRING    uniSymLink, uniDevName;
  NTSTATUS     ntStatus;
  PMDL         pMdl;
  PULONG         pulSSDTMapped;
  
  RtlInitUnicodeString(&uniSymLink, L"\\DosDevices\\ProcessesGuard");
  RtlInitUnicodeString(&uniDevName, L"\\Device\\ProcessesGuard");
  pDriverObj->DriverUnload = DriverUnload;
  pDriverObj->MajorFunction[IRP_MJ_CREATE] = 
  pDriverObj->MajorFunction[IRP_MJ_CLOSE] = DispatchDevOpen;
  pDriverObj->MajorFunction[IRP_MJ_DEVICE_CONTROL] = DispatchDevCtl;
  
  ntStatus = IoCreateDevice(pDriverObj, sizeof (DEVICE_EXTENSION), &uniDevName, FILE_DEVICE_UNKNOWN, 
    0, FALSE, &pDevObj);
  if (!NT_SUCCESS(ntStatus))
    return(ntStatus);
    
  IoCreateSymbolicLink(&uniSymLink, &uniDevName);
  pDevObj->Flags |= DO_BUFFERED_IO;
  pDevExt = pDevObj->DeviceExtension;
  pDevExt->pDevObj = pDevObj;
  pDevExt->uniSymLink = uniSymLink;
  
  pMdl = IoAllocateMdl(KeServiceDescriptorTable.ServiceTableBase, KeServiceDescriptorTable.NumberOfServices * 4,
    FALSE, FALSE, NULL);
  if (pMdl == NULL)
  {
    IoDeleteSymbolicLink(&uniSymLink);
    IoDeleteDevice(pDevObj);
    return(STATUS_INSUFFICIENT_RESOURCES);
  }
  
  MmBuildMdlForNonPagedPool(pMdl);
  pMdl->MdlFlags |= MDL_MAPPED_TO_SYSTEM_VA;  //Write SSDT
  
  pulSSDTMapped = (PULONG)MmMapLockedPagesSpecifyCache(pMdl, KernelMode, MmNonCached, NULL, FALSE, NormalPagePriority);
  if (pulSSDTMapped == NULL)
  {
    IoDeleteSymbolicLink(&uniSymLink);
    IoDeleteDevice(pDevObj);
    IoFreeMdl(pMdl);
    return(STATUS_UNSUCCESSFUL);
  }
  
  pDevExt->pMdl = pMdl;
  pDevExt->pulSSDTMapped = pulSSDTMapped;
  
  ZwOpenProcessReal = (ZWOPENPROCESS)SYSTEM_SERVICE(ZwOpenProcess);
  pulSSDTMapped[SYSTEM_INDEX(ZwOpenProcess)] = (PULONG)ZwOpenProcessHook;

  ZwOpenThreadReal = (ZWOPENTHREAD)SYSTEM_SERVICE(ZwOpenThread);
  pulSSDTMapped[SYSTEM_INDEX(ZwOpenThread)] = (PULONG)ZwOpenThreadHook;

  
  return(ntStatus);
}

VOID DriverUnload(PDRIVER_OBJECT pDriverObj)
{
  PDEVICE_OBJECT    pDevObj = pDriverObj->DeviceObject;
  PDEVICE_EXTENSION  pDevExt = pDevObj->DeviceExtension;
  PULONG         pulSSDTMapped = pDevExt->pulSSDTMapped;
  PMDL         pMdl = pDevExt->pMdl;
  
  pulSSDTMapped[SYSTEM_INDEX(ZwOpenProcess)] = (PULONG)ZwOpenProcessReal;
  pulSSDTMapped[SYSTEM_INDEX(ZwOpenThread)] = (PULONG)ZwOpenThreadReal;//先这个顺序
  MmUnmapLockedPages(pulSSDTMapped, pMdl);
  IoFreeMdl(pMdl);
  while (pDevObj)
  {
    pDevExt = pDevObj->DeviceExtension;
    pDevObj = pDevObj->NextDevice;
    
    IoDeleteSymbolicLink(&pDevExt->uniSymLink);
    IoDeleteDevice(pDevExt->pDevObj);
  }
}

NTSTATUS DispatchDevOpen(PDEVICE_OBJECT pDevObj, PIRP pIrp)
{
  NTSTATUS  ntStatus = STATUS_SUCCESS;
  
  pIrp->IoStatus.Status = ntStatus;
  pIrp->IoStatus.Information = 0;
  IoCompleteRequest(pIrp, IO_NO_INCREMENT);
  return(ntStatus);
}

NTSTATUS DispatchDevCtl(PDEVICE_OBJECT pDevObj, PIRP pIrp)
{
  PIO_STACK_LOCATION  pIrpStack = IoGetCurrentIrpStackLocation(pIrp);
  ULONG        ulIoCode, ulBufLength, ulRtn, ulCounts = 0, ulIndex;
  PVOID        pvBuf;
  NTSTATUS     ntStatus;
  
  ulIoCode = pIrpStack->Parameters.DeviceIoControl.IoControlCode;
  switch (ulIoCode)
  {
  case IOCTL_START_PROTECTION:
    ulBufLength = pIrpStack->Parameters.DeviceIoControl.InputBufferLength;
    pvBuf = pIrp->AssociatedIrp.SystemBuffer;
    ulCounts = ulBufLength / sizeof (ULONG);
    KdPrint(("Protection Numbers: %d\n"), ulCounts);
    for (ulIndex = 0; ulIndex < ulCounts && ulIndex < C_MAXPROCNUMS; ulIndex++)
    {
      ulPIDs[ulIndex] = ((PULONG)pvBuf)[ulIndex];
      KdPrint(("Index %d -- PID %d\n"), ulIndex, ulPIDs[ulIndex]);
    }
    ntStatus = STATUS_SUCCESS;
    ulRtn = ulBufLength;
    break;
  default:
    ntStatus = STATUS_INVALID_PARAMETER;
    ulRtn = 0;
    break;
  }
  
  pIrp->IoStatus.Status = ntStatus;
  pIrp->IoStatus.Information = ulRtn;
  IoCompleteRequest(pIrp, IO_NO_INCREMENT);
  return(ntStatus);
}


CHAR *TerminateName = "demo.exe";      //这里就是我们的进程名    
UCHAR *PsGetProcessImageFileName( IN PEPROCESS Process );    
  
BOOLEAN IsProtect(CHAR *temp)          //判断正在结束的进程是否是我们要保护的进程    
{    
    ULONG len = strcmp(TerminateName, temp);    
    if(!len)    
        return TRUE;    
    return FALSE;    
} 
NTSTATUS ZwOpenProcessHook(PHANDLE ProcessHandle,ACCESS_MASK DesiredAccess,POBJECT_ATTRIBUTES ObjectAttributes,PCLIENT_ID ClientId)//我们自己的//NtZwOpenProcess    
{    
  PEPROCESS process;  //接受通过 ProcessHandle 返回的进程    
  NTSTATUS status;     
   CHAR *pName;        //接受进程的进程名 
   HANDLE  hID;
   ULONG dwProcessId = NULL;
  PEPROCESS EProcessToOpen;
        

       status = PsLookupProcessByProcessId(
          ClientId->UniqueProcess,
          &process
           );
        
  if(!NT_SUCCESS(status))     
     return(ZwOpenProcessReal(ProcessHandle,DesiredAccess,ObjectAttributes,ClientId)); 
   

     pName = (CHAR*)PsGetProcessImageFileName(process); //获取进程名   


   if(IsProtect(pName)) //判断是否是我们要保护的进程，是则返回权限不足，否则调用原函数结束进程    
    {    
        if(process != PsGetCurrentProcess())    
        {    
            
      hID = PsGetProcessId(process);//获得进程 id
      KdPrint(("Protection Pid: %d\n"), hID);
    return STATUS_ACCESS_DENIED;    
        }    
   }   
   
        
   return(ZwOpenProcessReal(ProcessHandle,DesiredAccess,ObjectAttributes,ClientId));     
} 


NTSTATUS ZwOpenThreadHook(PHANDLE ProcessHandle,ACCESS_MASK DesiredAccess,POBJECT_ATTRIBUTES ObjectAttributes,PCLIENT_ID ClientId)//我们自己的 NtZwOpenProcess    
{    
    PEPROCESS process;  //接受通过 ProcessHandle 返回的进程    
    NTSTATUS status;     
    CHAR *pName;        //接受进程的进程名 
    HANDLE  hID;
    ULONG dwProcessId = NULL;
  //PEPROCESS EProcessToOpen;
        

       status = PsLookupProcessByProcessId(
            ClientId->UniqueProcess,
            &process
            );
        
    if(!NT_SUCCESS(status))     
        return(ZwOpenThreadReal(ProcessHandle,DesiredAccess,ObjectAttributes,ClientId)); 
   

     pName = (CHAR*)PsGetProcessImageFileName(process); //获取进程名   


    if(IsProtect(pName)) //判断是否是我们要保护的进程，是则返回权限不足，否则调用原函数结束进程    
    {    
        if(process != PsGetCurrentProcess())    
        {    
            
      hID = PsGetProcessId(process);//获得进程 id
      KdPrint(("Protection Pid: %d\n"), hID);
      return STATUS_ACCESS_DENIED;    
        }    
    }   
   
        
   return(ZwOpenThreadReal(ProcessHandle,DesiredAccess,ObjectAttributes,ClientId));     
} 

 
```



代码是完全可以编译的（`wdk 7600`）

具体的就是 `ssdt` hook

希望大家保留 原原创的版权

既然他给大家做了 贡献 就要尊重别人的成果

还有希望下了代码的都留点 痕迹 呵呵

`win7` 系统下稳定运行截图

解压密码 `kanxue` 发错代码跟附件 居然没人说我 羞死了。

`![](img/2f3ac239d6db59702244633ace8ca38f.jpg)`

上传的附件

| `![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg)` || `123.rar` |
