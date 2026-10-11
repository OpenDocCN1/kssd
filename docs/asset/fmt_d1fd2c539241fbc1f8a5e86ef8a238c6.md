
# TDI Object hook

> 原文：[`www.pediy.com/kssd/pediy11/115248.html`](https://www.pediy.com/kssd/pediy11/115248.html)

申请邀请码，文章一

在防火墙或者某些特殊场合，需要拦截 `udp` 或者 `tcp`，可能是按进程来拦截，可能是按 `ip` 或者端口来拦截。

这里发一段 `TDI` 层 hook `udp` 的代码，是 `object hook`，非 `dispatch hook`，`tcp` 或者 `udp` 的 `dispatch hook` 通常是通过 `InterlockedExchange` 交换出 `tcp` 或者 `udp` 的 `dispatch` 函数，网上资料容易找到，这里不做探讨。

这篇文章没有包括全部代码，但是整个思路已经说清，再说看雪的兄弟们都这么聪明，我偷点懒你们也能整明白。有些函数和变量的命名，纯属娱乐。

`object hook`，每种 `IRP` 都要处理，这是跟 `dispatch hook` 不同的地方，当然重点是放在 `IRP_MJ_INTERNAL_DEVICE_CONTROL` 的处理上。

## 1、`DriverEntry` 中，设置 `IRP_MJ_INTERNAL_DEVICE_CONTROL` 处理例程入口:

代码:

```
//pDriverObject 是入口传入的驱动对象
pDriverObject->MajorFunction[IRP_MJ_INTERNAL_DEVICE_CONTROL] = TDI_DispatchInternalDeviceControl;
pDriverObject->MajorFunction[IRP_MJ_CREATE] = TDI_DispatchCreate;//如果需要处理本地地址时，IRP_MJ_CREATE 的处理是个关键的地方，本文不做细究
```

另外在 `DriverEntry` 的适当位置添加：

代码:

```
status = HookUdpDevice(pDriverObject);
if (!NT_SUCCESS(status))
{
//KdPrint(("monitor.sys: Failed to hook udp device\n"));
}
```

## 2、`HookUdpDevice` 的实现如下：

代码:

```
#define SYSTEM_DEVICE_UDP_NAME            L"\\Device\\Udp"
#define AVKILLER_DEVICE_UDP_NAME          L"\\Device\\UdpMonitor"

NTSTATUS HookUdpDevice(IN PDRIVER_OBJECT DriverObject)
{
    NTSTATUS nsStatus;
    UNICODE_STRING  uString;
    PFILE_OBJECT    pUdpFileObject = NULL;
    PDEVICE_OBJECT  pUdpDeviceObject = NULL;
    PDEVICE_OBJECT  pAVKillerUdpDeviceObject = NULL;
    PDEVICE_OBJECT  pTargetDeviceObject = NULL;
    PAVKILLER_DEVICE_EXTENSION    pDeviceExtention = NULL;
    int nRetCode = 0;

    //取 udp 对象
    RtlInitUnicodeString(&uString, SYSTEM_DEVICE_UDP_NAME);
    nsStatus = IoGetDeviceObjectPointer(&uString, FILE_ALL_ACCESS, 
        &pUdpFileObject, &pUdpDeviceObject);
    if (!NT_SUCCESS(nsStatus))
    {
        //KdPrint(("HookUdpDevice: Failed to get udp device\n"));
        goto Exit0;
    }
    
    //创建 udp hook 对象
    RtlInitUnicodeString(&uString, AVKILLER_DEVICE_UDP_NAME);
    nsStatus = IoCreateDevice(
        DriverObject, sizeof(AVKILLER_DEVICE_EXTENSION),
        &uString, pUdpDeviceObject->DeviceType,
        pUdpDeviceObject->Characteristics, FALSE, &pAVKillerUdpDeviceObject);
    if (!NT_SUCCESS(nsStatus))
    {
        //KdPrint(("HookUdpDevice: Failed to create hook udp device\n"));
        goto Exit1;
    }
    
    //设备扩展处理，自个发挥
    pDeviceExtention = (PAVKILLER_DEVICE_EXTENSION)(pAVKillerUdpDeviceObject->DeviceExtension);
    nRetCode = InitDeviceExtension(pDeviceExtention, pAVKillerUdpDeviceObject);     
    
    //将 udp hook 对象“挂载”到 udp 对象之上
    pTargetDeviceObject = IoAttachDeviceToDeviceStack(pAVKillerUdpDeviceObject, pUdpDeviceObject);
    if (pTargetDeviceObject != NULL)
    {
        //设备扩展，自个发挥
        pDeviceExtention->pTdiDeviceObject    = pUdpDeviceObject;
        pDeviceExtention->pTdiFileObject      = pUdpFileObject;
        pDeviceExtention->pTargetDeviceObject = pTargetDeviceObject;
        pDeviceExtention->bTdiAttached        = TRUE;
        
        pAVKillerUdpDeviceObject->Flags |= pUdpDeviceObject->Flags & (DO_BUFFERED_IO | DO_DIRECT_IO);
        
        gAvkillerUdpDeviceObject = pAVKillerUdpDeviceObject;
        goto Exit0;
    }
    
    IoDeleteDevice(pAVKillerUdpDeviceObject);
    pAVKillerUdpDeviceObject = NULL;
    
Exit1:
    
    ObDereferenceObject(pUdpFileObject);
    
Exit0:
    
    return nsStatus;
}
```

## 3. 关键的拦截部分

代码:

```
NTSTATUS TDI_DispatchInternalDeviceControl(IN PDEVICE_OBJECT DeviceObject, IN PIRP Irp)
{
    NTSTATUS nsStatus = STATUS_SUCCESS;
    HANDLE hProcessID = 0;
    UCHAR* procName = NULL;     
    PIO_STACK_LOCATION pCurrentStackLocation = IoGetCurrentIrpStackLocation(Irp);
    UCHAR uMinorFunction = pCurrentStackLocation->MinorFunction;
    PFILE_OBJECT pFileObject = pCurrentStackLocation->FileObject;
    ULONG uContext = (ULONG)(pFileObject->FsContext2);
    PAVKILLER_DEVICE_EXTENSION pDeviceExtension = (PAVKILLER_DEVICE_EXTENSION)(DeviceObject->DeviceExtension);

    if (DeviceObject != gAvkillerUdpDeviceObject)
    {           
        Irp->IoStatus.Status = nsStatus;
        Irp->IoStatus.Information = 0;
        IoCompleteRequest(Irp, IO_NO_INCREMENT);
        goto Exit0;
    }

    //看你根据 ip 还是 port 拦截，怎么取这些拦截参数，这里只举处理 TDI_SEND_DATAGRAM 时的情况，其它自行查 ddk 文档
    switch (uMinorFunction)
    {              
    case TDI_SEND_DATAGRAM:
         nsStatus = TDISendDatagramXXX(pDeviceExtension, Irp, pCurrentStackLocation);
         break;     
    case TDI_RECEIVE_DATAGRAM:
         nsStatus = TDIReceiveDatagramXXX(pDeviceExtension, Irp, pCurrentStackLocation);
         break;                     
    case TDI_SET_EVENT_HANDLER:
         nsStatus = TDISetEventHandlerXXX(pDeviceExtension, Irp, pCurrentStackLocation);
         break;     
    case TDI_QUERY_INFORMATION:
         nsStatus = TDIQueryInformationXXX(pDeviceExtension, Irp, pCurrentStackLocation);
         break;     
    default:
         nsStatus = AllowCallLowerObject(pDeviceExtension, Irp);
         break;
    }

    /*如果要拦截某个进程:
   1. 获取当前进程
   1.1 获取当前进程名：PsGetProcessImageFileName(PsGetCurrentProcess());
   1.2 获取当前进程 id: PsGetCurrentProcessId();
   1.3 其实还有很多方法获取进程，看雪论坛总结得更好得文章，这里不再举例
   2. 直接如下面这样处理
   */
    if (拒绝连接)
    {
        nsStatus = DenyCallLowerObject(Irp);
    }
    if (允许连接)
    {
        nsStatus = AllowCallLowerObject(pDeviceExtension, Irp);
    }

    return nsStatus;
}

NTSTATUS TDISendDatagramXXX(IN PDEVICE_EXTENSION DeviceExtension, IN PIRP Irp, IN PIO_STACK_LOCATION IoStackLocation)
{
    NTSTATUS nsStatus = STATUS_SUCCESS;
    PTDI_REQUEST_KERNEL_SENDDG pTDI_Request_Kernel_Senddg = (PTDI_REQUEST_KERNEL_SENDDG)(&(IoStackLocation->Parameters.DeviceIoControl));
    ULONG uSendLength = pTDI_Request_Kernel_Senddg->SendLength;//发送长度
    PTDI_CONNECTION_INFORMATION pTDI_Connection_Information = pTDI_Request_Kernel_Senddg->SendDatagramInformation;
    PVOID pRemoteAddress = pTDI_Connection_Information->RemoteAddress;//发送地址
    ULONG uRemoteAddress;
    USHORT uRemotePort;
         
    /*这是取远端地址参数，假如取本地地址参数，比较麻烦，光从 TDISendDatagramXXX 函数里面是取不到的，需要在处理 IRP_MJ_CREATE 时，从 Irp 包中的 Irp->AssociatedIrp.SystemBuffer 中分析出来， Irp->AssociatedIrp.SystemBuffer 指向 FILE_FULL_EA_INFORMATION 结构，可以从 FILE_FULL_EA_INFORMATION 中解析本地地址参数，具体可参考 ddk。
    */   
    uRemoteAddress = GetIP(pRemoteAddress);//查 ddk 说明，解析出 ip
    uRemotePort = GetPort(pRemoteAddress);//查 ddk 说明，解析出 port

    if (拒绝连接)
    {
        nsStatus = DenyCallLowerObject(Irp);
    }
    if (允许连接)
    {
        nsStatus = AllowCallLowerObject(pDeviceExtension, Irp);
    }
}

//允许连接
NTSTATUS AllowCallLowerObject(IN PAVKILLER_DEVICE_EXTENSION DeviceExtension, IN PIRP Irp)
{
    NTSTATUS nsStatus = STATUS_SUCCESS;
     
    IoSkipCurrentIrpStackLocation(Irp);
    //IoCopyCurrentIrpStackLocationToNext(Irp);
     
    nsStatus = IoCallDriver(DeviceExtension->pTargetDeviceObject, Irp);
     
    return nsStatus;
}

//拒绝连接
NTSTATUS DenyCallLowerObject(PIRP Irp)
{
    NTSTATUS nsStatus = STATUS_ACCESS_DENIED;
     
    Irp->IoStatus.Status = nsStatus;
    Irp->IoStatus.Information = 0;
     
    IoCompleteRequest(Irp, IO_NO_INCREMENT);
     
    return nsStatus;
}
```
