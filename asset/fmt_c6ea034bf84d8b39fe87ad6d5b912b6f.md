
# NDIS 中间层驱动开发在 Win7 系统下和 Windows XP 系统下的区别

> 原文：[`www.pediy.com/kssd/pediy12/137545.html`](https://www.pediy.com/kssd/pediy12/137545.html)

NDIS 中间层驱动的开发在 Win7 系统上和 Windows XP 系统上有差别。

我把 NDIS 中间层的讨论分成 2 块。windows 7 系统和 Windows XP 系统。

## (一) 在 Windows XP 系统上进行开发

平时很多朋友包括我在内，我们都在 XP 系统上使用 NDIS5.1 的框架来进行程序开发。我们都使用 Microsoft WDK 提供的 NDIS 的 Passthru 例子，在这个例子上做进一步的修改，来达到我们的目地。

在 Passthru 工程的 `DriverEntry` 函数里面，我们都看见如下的代码：

```c
NTSTATUS  DriverEntry(IN PDRIVER_OBJECT  DriverObject,  IN  PUNICODE_STRING  RegistryPath)

{
          NDIS_STATUS    Status;
  NDIS_PROTOCOL_CHARACTERISTICS  PChars;
  NDIS_MINIPORT_CHARACTERISTICS  MChars;
  PNDIS_CONFIGURATION_PARAMETER  Param;
  NDIS_STRING      Name;
  NDIS_HANDLE      WrapperHandle;
  UNICODE_STRING nameString, linkString;
  UINT FuncIndex;
  PDEVICE_OBJECT MyDeviceObject;
  PDRIVER_DISPATCH MajorFunction[IRP_MJ_MAXIMUM_FUNCTION + 1];
  Status = NDIS_STATUS_SUCCESS;
  //申请自旋锁 以到达资源共享的同步访问
  NdisAllocateSpinLock(&GlobalLock);
  //调用 NdisMInitializeWrapper 函数来保存在 NdisWrapperHandle 返回的句柄
  NdisMInitializeWrapper(&NdisWrapperHandle, DriverObject, RegistryPath, NULL);
  do
  {
//调用 NdisMInitializeWrapper 函数来保存在 WrapperHandle 返回的句柄
    NdisMInitializeWrapper(&WrapperHandle, DriverObject, RegistryPath, NULL);

//对于 MiniportCharacteristics 组件,如果驱动程序不用导出 MiniportXxx 这样的函数,则必须赋值为 NULL。

//如果要导出任何新版本的 V4.0 或 V5.0 的 MiniportXxx 函数,那么中间层驱动程序的主版本必须是 V4.0,并且提供 4.0 或 5.0 版本的 MiniportCharacteristics 组件.
  NdisZeroMemory(&MChars, sizeof(NDIS_MINIPORT_CHARACTERISTICS));
  MChars.MajorNdisVersion = PASSTHRU_MAJOR_NDIS_VERSION;
  MChars.MinorNdisVersion = PASSTHRU_MINOR_NDIS_VERSION;

   /******************************

      下面开始注册中间层驱动程序的 MiniportXxx 函数

    ****************************/
  MChars.HaltHandler           =   MPHalt;
  MChars.InitializeHandler     =   MPInitialize;
  MChars.QueryInformationHandler =   MPQueryInformation;
  MChars.SetInformationHandler   =   MPSetInformation;
  MChars.ResetHandler           =   MPReset;
  MChars.SendHandler           =   NULL;
  MChars.SendPacketsHandler      =   MPSendPackets;
  MChars.TransferDataHandler     =   MPTransferData;
  MChars.ReturnPacketHandler     =   MPReturnPacket;
  MChars.CheckForHangHandler     =   NULL;

#ifdef NDIS51_MINIPORT
  MChars.CancelSendPacketsHandler =  MPCancelSendPackets;
  MChars.PnPEventNotifyHandler    =  MPDevicePnPEvent;
  MChars.AdapterShutdownHandler   =  MPAdapterShutdown;
#endif // NDIS51_MINIPORT

/*
传递上一步保存的句柄,并调用 NdisIMRegisterLayeredMiniport 函数来注册驱动程序的 MiniportXxx 系列函数。其中句柄 NdisWrapperHandle 是由先前的 NdisMInitializeWrapper 函数返回的。

当驱动程序调用 NdisIMInitializeDeviceInstance 函数,请求中间层驱动程序的 MiniportInitialize 函数对虚拟 NIC 进行初始化时,需要把句柄 NdisWrapperHandle 传入 NDIS

当中间层驱动程序成功地绑定到一个或者多个 NIC 驱动程序上的时候, 或者是绑定到一个非 NIC 驱动程序上的时候也会调用 NdisIMInitializeDeviceInstance 函数。这样做使得中间层驱动程序可以初始化 Miniport 组件来接受虚拟 NIC 上的 I/O 请求
*/
           Status = NdisIMRegisterLayeredMiniport(NdisWrapperHandle, &MChars, sizeof(MChars), &DriverHandle);
            if (Status != NDIS_STATUS_SUCCESS)
       {
       break;
             }

#ifndef WIN9X
  NdisMRegisterUnloadHandler(NdisWrapperHandle, PtUnload);
#endif
  //中间层驱动程序要做的事情:
  //初始化 NDIS_PROTOCOL_CHARACTERISTICS 类型的结构.  NDIS 不再支持 3.0 版本的协议驱动.
  //能够使用 4.0 或 5.0 版本的 ProtocolCharacteristic 结构体,协议驱动必须支持 PNP 即插即用功能.
  NdisZeroMemory(&PChars, sizeof(NDIS_PROTOCOL_CHARACTERISTICS));
  PChars.MajorNdisVersion = PASSTHRU_PROT_MAJOR_NDIS_VERSION;
  PChars.MinorNdisVersion = PASSTHRU_PROT_MINOR_NDIS_VERSION;
  NdisInitUnicodeString(&Name, L"Passthru");

  PChars.Name = Name;

  /******************************

         下面开始注册中间层驱动程序的  ProtocolXxx 函数

    这里主要是注册---下边界面向无连接的中间层驱动程序的 ProtocolXxx 函数

        ****************************/
  PChars.OpenAdapterCompleteHandler = PtOpenAdapterComplete;
  PChars.CloseAdapterCompleteHandler = PtCloseAdapterComplete;
  PChars.SendCompleteHandler = PtSendComplete;
  PChars.TransferDataCompleteHandler = PtTransferDataComplete;
  PChars.ResetCompleteHandler = PtResetComplete;
  PChars.RequestCompleteHandler = PtRequestComplete;
  PChars.ReceiveHandler = PtReceive;
  PChars.ReceivePacketHandler = PtReceivePacket;
  PChars.ReceiveCompleteHandler = PtReceiveComplete;
  PChars.StatusHandler = PtStatus;
  PChars.StatusCompleteHandler = PtStatusComplete;
  PChars.BindAdapterHandler = PtBindAdapter;
  PChars.UnbindAdapterHandler = PtUnbindAdapter;
  PChars.UnloadHandler = PtUnloadProtocol;
  PChars.PnPEventHandler= PtPNPHandler;

  /*
  如果驱动程序随后要绑定到底层的 NDIS 驱动程序上,则调用 NdisRegisterProtocol 函数来注册驱动程序的 ProtocolXxx 函数。

  全局变量 ProtHandle 是在 NDIS 协议驱动里面的 NdisRegisterProtocol 函数里面初始化的,然后中间层驱动必须保存 protHandle 的值,并在将来 NDIS 中间层驱动程序的协议部分的函数调用中作为输入参数来传递.

  */
  NdisRegisterProtocol(&Status,  &ProtHandle,&PChars,
       sizeof(NDIS_PROTOCOL_CHARACTERISTICS));
  if (Status != NDIS_STATUS_SUCCESS)
  {
    NdisIMDeregisterLayeredMiniport(DriverHandle);
                break;
  }

//如果驱动程序导出了 MiniportXxx 和 ProtocolXxx 这一些列的函数,那么就调用 NdisIMAssociateMiniport 函数向 NDIS 通告有关驱动程序的微端口低边界和协议高边界信息
    NdisIMAssociateMiniport(DriverHandle, ProtHandle);

  }
  while (FALSE);

//----------------- 创建设备对象与符号连接----------------------------
  RtlInitUnicodeString(&nameString, L"\\\\Device\\\\MyPassthru" );
  RtlInitUnicodeString(&linkString, L"\\\\??\\\\MyPassthru");
  for(FuncIndex = 0;
            FuncIndex <=IRP_MJ_MAXIMUM_FUNCTION;
            FuncIndex++)
  {
    MajorFunction[FuncIndex] = NULL;
  }
  MajorFunction[IRP_MJ_CREATE]           =  MydrvDispatch;
  MajorFunction[IRP_MJ_CLOSE]            =  MydrvDispatch;
  MajorFunction[IRP_MJ_DEVICE_CONTROL]  =  MydrvDispatchIoctl;
  Status = NdisMRegisterDevice(
            WrapperHandle,
            &nameString,
            &linkString,
            MajorFunction,
            &MyDeviceObject,
            &NdisDeviceHandle
            );
  if(Status != STATUS_SUCCESS)
  {
    DbgPrint("NdisMRegisterDevice failed!\n");
  }
  return(Status);
}
```

然后，我们就开始实现相应的回调函数。然而为了收、发网络数据包，对它们进行控制，我们基本上只会去修改以下的回调函数：

下面我把相关的函数的功能给罗列出来。

```c
/**** MPSend 函数的功能是: 发送单个数据包。
    若中间层驱动不支持 MiniportSendPackets,那么 MPSend 函数必须实现。
    参数说明:
      MiniportAdapterContext    它是适配器。
      PacketArray               它是包描述符指针。
      Flags                     它未被使用。
    *****/
NDIS_STATUS
MPSend(
    IN NDIS_HANDLE        MiniportAdapterContext,
    IN PNDIS_PACKET       Packet,
    IN  UINT              Flags
    )

/**** MPSendPackets 函数的功能是:
  用于指定网络上传输数据包的包描述符指针数组。该函数可以用来发送多个数据包。而不是只发送单个数据包。除非中间层驱动程序绑定到低层 WAN NIC 驱动程序上,并提供 MiniportWanSend 函数,否则驱动程序应提供对 MPSendPackets 函数的支持, 而不是对 MPSend 函数的支持。
****/
```
