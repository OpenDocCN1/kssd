
# 关于拦截“通过 WMI 读取硬件序列号”的一些心得

> 原文：[`www.pediy.com/kssd/pediy11/127176.html`](https://www.pediy.com/kssd/pediy11/127176.html)

大家都知道，WMI（Windows Management Instrumentation, Windows 管理规范）是一项核心的 Windows 管理技术；用户可以使用 WMI 管理本地和远程计算机，在这里只讨论它的硬件信息管理功能。

所有硬件设备的信息要能被 WMI 收集，其编写的驱动程序必须符合 WMI 规范，即 WMI 相当于这些硬件信息的管理员。通过 `IRPTrace` 这个工具可以很方便看到，当 ring3 有读取硬件序列号的动作，传到 ring0 层是这样：`WMIDataDevice` 内核设备会收到一个名叫 `IRP_MJ_SYSTEM_CONTROL` 的通知，当然该通知还会附加着一个类似于 `IRP_MN_QUERY_ALL_DATA` 的通知。只要有一定驱动编程基础的人都知道，该通知会层层传递下去，得到结果后返回上来。

所以，我的拦截思路很简单，编写一个过滤驱动设备，附载在 `WMIDataDevice` 内核设备上，我的过滤驱动只对 `IRP_MJ_SYSTEM_CONTROL` 进行过滤：

```c
NTSTATUS DriverEntry(  
    IN PDRIVER_OBJECT  DriverObject,  
    IN PUNICODE_STRING RegistryPath  
    )  
{  
   for (i = 0; i < IRP_MJ_MAXIMUM_FUNCTION; i++) {  
       DriverObject->MajorFunction[i] = LS2capDispatchGeneral;  
    }  
    //  
    // Our read function is where we do our real work.  
    //  
    DriverObject->MajorFunction[IRP_MJ_SYSTEM_CONTROL] = LS2DispatchSystemControl;  
    return LS2capInit( DriverObject );  
}
```

然后再设置完成例程，目的是等该通知取到序列号后可以拦截并任意修改序列号。

主要就是编写 `NTSTATUS LS2DispatchSystemControl(IN PDEVICE_OBJECT DeviceObject, IN PIRP Irp)` 例程。

代码如下：

```c
NTSTATUS  LS2DispatchSystemControl(  
    IN PDEVICE_OBJECT DeviceObject,  
    IN PIRP          Irp  
    )  
{  
  //PAGED_CODE();  
    PIO_STACK_LOCATION  currentIrpStack;  
    PIO_STACK_LOCATION  nextIrpStack;  
  NTSTATUS status ;  
    PDEVICE_EXTENSION   devExt;  
    UCHAR minorFunc;  
    PIO_STACK_LOCATION       IrpSp;  
  char teststr2[34]="IRP_MN_QUERY_ALL_DATA";  
  char teststr3[34]="IRP_MN_QUERY_SINGLE_INSTANCE";  
  char teststr0[34]="NOTHING";  
    IrpSp = IoGetCurrentIrpStackLocation( Irp );  
  devExt = (PDEVICE_EXTENSION) DeviceObject->DeviceExtension;  
  status = STATUS_SUCCESS ;  
    minorFunc = IrpSp->MinorFunction;  
  switch(minorFunc)  
  {  
  case IRP_MN_QUERY_ALL_DATA:  
    devExt->controlcode=2;  
    currentIrpStack = IoGetCurrentIrpStackLocation(Irp);  
        nextIrpStack = IoGetNextIrpStackLocation(Irp);    
        *nextIrpStack = *currentIrpStack;  
        IoSetCompletionRoutine( Irp, Ctrl2capWMIComplete,  
                                DeviceObject, TRUE, TRUE, TRUE );  
      WriteFileTest(teststr2,34);  
    break;  
  case IRP_MN_QUERY_SINGLE_INSTANCE:  
    devExt->controlcode=3;  
    currentIrpStack = IoGetCurrentIrpStackLocation(Irp);  
        nextIrpStack = IoGetNextIrpStackLocation(Irp);    
        *nextIrpStack = *currentIrpStack;  
        IoSetCompletionRoutine( Irp, Ctrl2capWMIComplete,  
                                DeviceObject, TRUE, TRUE, TRUE );  
      WriteFileTest(teststr3,34);  
    break;  
  case IRP_MN_REGINFO:  
  case IRP_MN_REGINFO_EX:  
  case IRP_MN_CHANGE_SINGLE_INSTANCE:  
  case IRP_MN_CHANGE_SINGLE_ITEM:  
  case IRP_MN_EXECUTE_METHOD:  
  case IRP_MN_DISABLE_EVENTS:  
  case IRP_MN_ENABLE_COLLECTION:  
  case IRP_MN_DISABLE_COLLECTION:  
  case IRP_MN_ENABLE_EVENTS:  
  default:  
       #if WIN2K  
           IoSkipCurrentIrpStackLocation(Irp);  
       #else // WIN2K  
       //  
       // This is the equivalent of the IoSkipCurrentIrpStackLocation macro,  
       // which doesn't exist in the NT 4 DDK.  
       //  
           Irp->CurrentLocation++;  
           Irp->Tail.Overlay.CurrentStackLocation++;  
       #endif // WIN2K  
       WriteFileTest(teststr0,34);  
           return IoCallDriver(((PDEVICE_EXTENSION) DeviceObject->DeviceExtension)->TopOfStack, Irp);  
       break;  
  }  
  // 完成 IRP  
  /*  
  Irp->IoStatus.Status = status;  
  Irp->IoStatus.Information = 0;                // bytes xfered  
  IoCompleteRequest( Irp, IO_NO_INCREMENT );          //结束 IRP 请求，即不再往下传递  
    return status;  
    */  
  return IoCallDriver( devExt->TopOfStack, Irp );  
}
```

为了方便检测是否过滤成功，可以创建一个文本文件，并往里写入标记。本人已实现对硬盘，主板，MAC 地址等读取拦截。主要思路和代码就是这些，请各位多多指教。
