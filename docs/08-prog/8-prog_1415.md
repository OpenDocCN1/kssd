
# 利用 PsSetCreateProcessNotifyRoutine 监控进程启动的方法

> 原文：[`www.pediy.com/kssd/pediy10/102411.html`](https://www.pediy.com/kssd/pediy10/102411.html)

想监控进程，有好几种方式，有 Hook `NTCreatProcess`，但是有很多进程并不是通过 `NTCreatProcess` 创建，还有 hook `NTCreatSection` 等。但是我想介绍的就是利用到的函数 `PsSetCreateProcessNotifyRoutine`，这是一个回调函数，当进程创建和销毁时，就会告诉这个函数。这里就不多介绍其函数原型。但是它的参数会传入父进程的 `ID`，进程的 `ID` 和是否创建。

所以我设置一个结构用来保存以上参数的值，并且将该结构传给应用层显示出来。

```c
typedef struct _CallbackInfo
{
    HANDLE  hParentId;
    HANDLE  hProcessId;
    CHAR    ProcFullPath[256];         //进程路径
    BOOLEAN bCreate;
}CALLBACK_INFO, *PCALLBACK_INFO;
```

其中 `ProcFullPath` 用来保存进程的完整路径。还需要定义一个设备扩展结构如下：

```c
struct _DEVICE_EXTENSION
{
    HANDLE             hProcessHandle;        // 事件对象句柄
    PKEVENT            ProcessEvent;          // 用户和内核通信的事件对象指针
    HANDLE             hParentId;             // 在回调函数中保存进程信息
    HANDLE             hProcessId;
    PCHAR              ProcFullPath;          //进程路径
    BOOLEAN            bCreate;
}DEVICE_EXTENSION, *PDEVICE_EXTENSION;
```

这是驱动设备留给用户自己可以定义的结构，这个结构用来保存回调函数的信息。主要代码如下：

```c
// 驱动入口
NTSTATUS  DriverEntry(IN PDRIVER_OBJECT DriverObject,IN PUNICODE_STRING RegistryPath )
{
    UNICODE_STRING  nameString, linkString,ProcessEventString;
  PDEVICE_OBJECT      deviceObject;
    PDEVICE_EXTENSION  deviceExtension;
  NTSTATUS            status;
  IO_STATUS_BLOCK     ioStatus;
  PIRP        pIrp;
    int               i;
  status=STATUS_SUCCESS;
    DriverObject->MajorFunction[IRP_MJ_CREATE]=DispatchCreate;    
    DriverObject->MajorFunction[IRP_MJ_CLOSE]=DispatchClose;
  DriverObject->MajorFunction[IRP_MJ_DEVICE_CONTROL]=DispatchIoctl;    
    DriverObject->DriverUnload=DriverUnload;   
    //建立设备
    RtlInitUnicodeString(&nameString,L"\\Device\\WssProcMon");
    // 创建设备对象
  status=IoCreateDevice(DriverObject,
                        sizeof(DEVICE_EXTENSION), // 为设备扩展结构申请空间 
                        &nameString,
                        FILE_DEVICE_UNKNOWN,
                        0,
                        FALSE,
                        &deviceObject);
    if (!NT_SUCCESS(status))
  { return status;
  }
    deviceExtension=(PDEVICE_EXTENSION)deviceObject->DeviceExtension;
    RtlInitUnicodeString(&linkString,L"\\DosDevices\\WssProcMon");
    status = IoCreateSymbolicLink (&linkString, &nameString);
    if (!NT_SUCCESS(status))
    {
        //删除上面的设备对象
    IoDeleteDevice(deviceObject);
        return status;
    }
   //保存到设备对象的指针，下面在进程回调函数中还要使用
    g_pDriverObject=deviceObject;
    RtlInitUnicodeString(&ProcessEventString,EVENT_NAME);   
  //这里是创建事件对象用来与应用层交流
  deviceExtension->ProcessEvent=IoCreateNotificationEvent(&ProcessEventString,&deviceExtension->hProcessHandle);
    这里需要说明一下事件的作用，是为了达到两个对象同步，其实达到互斥还可以利用信号灯、互斥体，首先设置为非受信状态，这里注意事件对象只有两种状态：非受信和受信状态，非受信状态指的是没激活，受信状态指的是激活，也就是说建立事件对象后，先不激活，当有进程创建时，就激活，激活的话，由于应用层一直打开这个事件对象，并且用 WaitForSingleObject 一直等待，直到有事件激活，它在返回给内核，然后内核有设置为非受信状态。
  //设置为非受信状态
  KeClearEvent(deviceExtension->ProcessEvent);
    //设置回调例程
    status = PsSetCreateProcessNotifyRoutine(ProcessCreateMon,FALSE);
    if (!NT_SUCCESS(status))
    {
        DbgPrint("PsSetCreateProcessNotifyRoutine()\n");
        return status;
    }     
   return status;
} 

//处理设备对象操作
NTSTATUS DispatchCreate(IN PDEVICE_OBJECT DeviceObject,IN PIRP   Irp)
{
  DbgPrint("IRP_MJ_Create\n");
  Irp->IoStatus.Information=0;
  Irp->IoStatus.Status=STATUS_SUCCESS;
  IoCompleteRequest(Irp,IO_NO_INCREMENT);
  return STATUS_SUCCESS;
}
```

这里省略（.......）

```c
// I/O 控制派遣例程
NTSTATUS DispatchIoctl(IN PDEVICE_OBJECT DeviceObject,IN PIRP  Irp) 
{
  PDEVICE_EXTENSION  deviceExtension;
  ULONG IoControlCode,InBufLength,outBufLength; 
  PCALLBACK_INFO pCallbackInfo;
  NTSTATUS status=STATUS_INVALID_DEVICE_REQUEST;
  // 取得此 IRP（pIrp）的 I/O 堆栈指针
   PIO_STACK_LOCATION  irpStack = IoGetCurrentIrpStackLocation(Irp);
  // 取得设备扩展结构指针
  deviceExtension=(PDEVICE_EXTENSION)DeviceObject->DeviceExtension;
  // 取得 I/O 控制代码
  IoControlCode = irpStack->Parameters.DeviceIoControl.IoControlCode;
  pCallbackInfo=(PCALLBACK_INFO)Irp->AssociatedIrp.SystemBuffer;
  InBufLength  =irpStack->Parameters.DeviceIoControl.InputBufferLength;
  outBufLength =irpStack->Parameters.DeviceIoControl.OutputBufferLength;
  switch(IoControlCode)
  {
   case IOCTL_NTPROCDRV_GET_PROCINFO:
     {
         KdPrint(("[ProcMon] IOCTL: 0x%X", IoControlCode));
       if(outBufLength>=sizeof(CALLBACK_INFO))
     {
        pCallbackInfo->hParentId   = deviceExtension->hParentId;
        pCallbackInfo->hProcessId  = deviceExtension->hProcessId;
    strcpy(pCallbackInfo->ProcFullPath,deviceExtension->ProcFullPath);
    pCallbackInfo->bCreate     = deviceExtension->bCreate;
    status=STATUS_SUCCESS;
     }
     }
     break;
  }
  /*这里如果直接将 Irp->IoStatus.Information=0，那么 ring3 就读不出 CallbackInfo 结构体的值，也就是如果收到 IOCTL_NTPROCDRV_GET_PROCINFO
  必须设置 Irp->IoStatus.Information=outBufLength 的大小*/
  if(status==STATUS_SUCCESS)
  Irp->IoStatus.Information=outBufLength; 
   else
    Irp->IoStatus.Information=0;
  //必须完成 i/0 回复
  Irp->IoStatus.Status=status;
  IoCompleteRequest(Irp,IO_NO_INCREMENT);
  return status;
}

VOID ProcessCreateMon(IN HANDLE hParentId,IN HANDLE PId,IN BOOLEAN bCreate)
{
    PEPROCESS    EProcess;
    ULONG        ulCurrentProcessId;
    LPTSTR       lpCurProc; //进程名
    NTSTATUS     status;
  PDEVICE_EXTENSION  deviceExtension;
  HANDLE hProcess=NULL;
    ANSI_STRING pImageName;
    PCHAR outbuf;
  ULONG outlen;
    status = PsLookupProcessByProcessId((ULONG)PId, &EProcess);
    if (!NT_SUCCESS( status ))
    {
        DbgPrint("PsLookupProcessByProcessId()\n");
        return;
    }
    if ( bCreate )
    {
          lpCurProc = (LPTSTR)EProcess;
          lpCurProc = lpCurProc + ProcessNameOffset;
      RtlInitAnsiString(&pImageName,"test");  
    GetProcPath(PId,&pImageName);
    outbuf=(PCHAR)pImageName.Buffer;
      outlen=pImageName.Length+1;
  // 得到设备扩展结构的指针
    deviceExtension=(PDEVICE_EXTENSION)g_pDriverObject->DeviceExtension;
  // 安排当前值到设备扩展结构
    // 用户模式应用程序将使用 DeviceIoControl 调用把它取出
    deviceExtension->hParentId=hParentId;
    deviceExtension->hProcessId=PId;
    deviceExtension->ProcFullPath=(PCHAR)outbuf;
  deviceExtension->bCreate=bCreate;     
  //触发事件，通知应用程序
  // 用户模式下的应用程序不能重置 KM 事件，所以我们要在这里触发它
  KeSetEvent(deviceExtension->ProcessEvent,0,FALSE);
    KeClearEvent(deviceExtension->ProcessEvent);
  DbgPrint( "CREATE PROCESS = PROCESS NAME: %s , PROCESS PARENTID: %d, PROCESS ID: %d, PROCESS ADDRESS %x:\n", 
                      deviceExtension->ProcFullPath,
                      deviceExtension->hParentId,
                      deviceExtension->hProcessId,
                      EProcess);
  //DbgPrint("[ProcMon] IOCTL: 0x%X", IOCTL_NTPROCDRV_GET_PROCINFO);
  }
    else
    {
        DbgPrint( "TERMINATED == PROCESS ID: %d\n", PId);
    }
}
```

先运行加载驱动，在运行应用程序

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `ProcMon.rar`

\[注意该附件请到论坛下载：[`bbs.pediy.com/showthread.php?t=102411`](http://bbs.pediy.com/showthread.php?t=102411)\]

|
