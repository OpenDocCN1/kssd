
# NDIS 驱动学习笔记

> 原文：[`www.pediy.com/kssd/pediy10/65053.html`](https://www.pediy.com/kssd/pediy10/65053.html)

**作者：** `ufphpc`

小弟看了大牛们众多文章，而自己还没写过什么东西，并不是不想写，实在是和大牛们差距太大，今天小弟把学习 NDIS 驱动贴出来，希望大牛们给指正。

## Ndis Intermediate Miniport driver 参考 passthru

```c
NTSTATUS DriverEntry(
    IN  PDRIVER_OBJECT     DriverObject,
    IN  PUNICODE_STRING    RegistryPath
)
```

程序入口没什么好说的，而 NDIS 不符合 WDM 往下到了：

```c
NdisMInitializeWrapper(&NdisWrapperHandle, DriverObject, RegistryPath, NULL);
```

在 `DriverEntry` 里调用 `NdisMInitializeWrapper` 函数初始包裹来使得微端口驱动和 NDIS 相联系。它返回一个 `NdisWrapperHandle` 句柄，它是代表这个微端口驱动程序结构的句柄。

然而 `NdisMInitializeWrapper` 到底作了什么呢？我们看 REACTOS 里这个函数：

```c
EXPORT NdisInitializeWrapper(
    OUT PNDIS_HANDLE    NdisWrapperHandle,
    IN  PVOID           SystemSpecific1,
    IN  PVOID           SystemSpecific2,
    IN  PVOID           SystemSpecific3)

/*
 * FUNCTION: Notifies the NDIS library that a new miniport is initializing
 * ARGUMENTS:
 *     NdisWrapperHandle = Address of buffer to place NDIS wrapper handle
 *     SystemSpecific1   = Pointer to the driver's driver object
 *     SystemSpecific2   = Pointer to the driver's registry path
 *     SystemSpecific3   = Always NULL
 * NOTES:
 *     - SystemSpecific2 goes invalid so we copy it
 */
{
  PNDIS_M_DRIVER_BLOCK Miniport;
  PUNICODE_STRING RegistryPath;
  WCHAR *RegistryBuffer;

  NDIS_DbgPrint(MAX_TRACE, ("Called.\n"));
  ASSERT(NdisWrapperHandle);
  *NdisWrapperHandle = NULL;

#if BREAK_ON_MINIPORT_INIT
  __asm__ ("int $3\n");
#endif

  Miniport = ExAllocatePool(NonPagedPool, sizeof(NDIS_M_DRIVER_BLOCK));
  if (!Miniport)
    {
      NDIS_DbgPrint(MIN_TRACE, ("Insufficient resources.\n"));
      return;
    }

  RtlZeroMemory(Miniport, sizeof(NDIS_M_DRIVER_BLOCK));
  KeInitializeSpinLock(&Miniport->Lock);
  Miniport->DriverObject = (PDRIVER_OBJECT)SystemSpecific1;

  /* set the miniport's driver registry path */
  RegistryPath = ExAllocatePool(PagedPool, sizeof(UNICODE_STRING));
  if(!RegistryPath)
    {
      NDIS_DbgPrint(MIN_TRACE, ("Insufficient resources.\n"));
      return;
    }

  RegistryPath->Length = ((PUNICODE_STRING)SystemSpecific2)->Length;
  RegistryPath->MaximumLength = RegistryPath->Length + sizeof(WCHAR);  /* room for 0-term */
  RegistryBuffer = ExAllocatePool(PagedPool, RegistryPath->MaximumLength);
  if(!RegistryBuffer)
    {
      NDIS_DbgPrint(MIN_TRACE, ("Insufficient resources.\n"));
      return;
    }

  RtlCopyMemory(RegistryBuffer, ((PUNICODE_STRING)SystemSpecific2)->Buffer, RegistryPath->Length);
  RegistryBuffer[RegistryPath->Length/sizeof(WCHAR)] = 0;
  RegistryPath->Buffer = RegistryBuffer;
  Miniport->RegistryPath = RegistryPath;

  InitializeListHead(&Miniport->DeviceList);

  /* Put miniport in global miniport list */
  ExInterlockedInsertTailList(&MiniportListHead, &Miniport->ListEntry, &MiniportListLock);

  *NdisWrapperHandle = Miniport;
}
```

它仅初始化了一个 `NDIS_M_DRIVER_BLOCK` 结构 `Miniport`，并把它插入链表并返回这个结构。让我们看看它的结构：

```c
typedef struct _NDIS_M_DRIVER_BLOCK {
    LIST_ENTRY                 ListEntry;            /* Entry on global list */
    KSPIN_LOCK                 Lock;                 /* Protecting spin lock */
    NDIS_MINIPORT_CHARACTERISTICS  MiniportCharacteristics;  /* Miniport characteristics */
    WORK_QUEUE_ITEM            WorkItem;             /* Work item */
    PDRIVER_OBJECT             DriverObject;         /* Driver object of miniport */
    LIST_ENTRY                 DeviceList;           /* Adapters created by miniport */
    PUNICODE_STRING            RegistryPath;         /* SCM Registry key */
} NDIS_M_DRIVER_BLOCK, *PNDIS_M_DRIVER_BLOCK;
```

注意有一个 `NDIS_MINIPORT_CHARACTERISTICS` 结构，我们下面必须初始化这一结构。`NDIS_MINIPORT_CHARACTERISTICS` 结构指定了与微端口兼容的 NDIS 版本以及由微端口提供的可选上层函数（`MINIPORTXXX`）。然后 `DriverEntry` 通过一个指向 `NDIS_MINIPORT_CHARACTERISTICS` 的指针调用 `NdisIMRegisterLayeredMiniport` 或 `NdisMRegisterMiniport`：

```c
NdisZeroMemory(&MChars, sizeof(NDIS_MINIPORT_CHARACTERISTICS));
MChars.MajorNdisVersion = PASSTHRU_MAJOR_NDIS_VERSION;
MChars.MinorNdisVersion = PASSTHRU_MINOR_NDIS_VERSION;
MChars.InitializeHandler = MPInitialize;
MChars.QueryInformationHandler = MPQueryInformation;
MChars.SetInformationHandler = MPSetInformation;
MChars.ResetHandler = MPReset;
MChars.TransferDataHandler = MPTransferData;
MChars.HaltHandler = MPHalt;

#ifdef NDIS51_MINIPORT
    MChars.CancelSendPacketsHandler = MPCancelSendPackets;
    MChars.PnPEventNotifyHandler = MPDevicePnPEvent;
    MChars.AdapterShutdownHandler = MPAdapterShutdown;
#endif // NDIS51_MINIPORT

//
// We will disable the check for hang timeout so we do not
// need a check for hang handler!
//
MChars.CheckForHangHandler = NULL;
MChars.ReturnPacketHandler = MPReturnPacket;

//
// Either the Send or the SendPackets handler should be specified.
// If SendPackets handler is specified, SendHandler is ignored
//
MChars.SendHandler = NULL;  // MPSend;
MChars.SendPacketsHandler = MPSendPackets;

Status = NdisIMRegisterLayeredMiniport(NdisWrapperHandle,
                                        &MChars,
                                        sizeof(MChars),
                                        &DriverHandle);
```

我们来看看 `NdisMRegisterMiniport` 做了什么：

```c
NDIS_STATUS
EXPORT
NdisMRegisterMiniport(
    IN  NDIS_HANDLE                     NdisWrapperHandle,
    IN  PNDIS_MINIPORT_CHARACTERISTICS  MiniportCharacteristics,
    IN  UINT                            CharacteristicsLength)

/*
 * FUNCTION: Registers a miniport's MiniportXxx entry points with the NDIS library
 * ARGUMENTS:
 *     NdisWrapperHandle       = Pointer to handle returned by NdisMInitializeWrapper
 *     MiniportCharacteristics = Pointer to a buffer with miniport characteristics
 *     CharacteristicsLength   = Number of bytes in characteristics buffer
 * RETURNS:
 *     Status of operation
 */
{
  UINT MinSize;
  PNDIS_M_DRIVER_BLOCK Miniport = GET_MINIPORT_DRIVER(NdisWrapperHandle);
  PNDIS_M_DRIVER_BLOCK *MiniportPtr;
  NTSTATUS Status;

  NDIS_DbgPrint(MAX_TRACE, ("Called.\n"));

  switch (MiniportCharacteristics->MajorNdisVersion)
    {
      case 0x03:
        MinSize = sizeof(NDIS30_MINIPORT_CHARACTERISTICS);
        break;
      case 0x04:
        MinSize = sizeof(NDIS40_MINIPORT_CHARACTERISTICS);
        break;
      case 0x05:
        MinSize = sizeof(NDIS50_MINIPORT_CHARACTERISTICS);
        break;
      default:
        NDIS_DbgPrint(DEBUG_MINIPORT, ("Bad miniport characteristics version.\n"));
        return NDIS_STATUS_BAD_VERSION;
    }

  if (CharacteristicsLength < MinSize)
    {
        NDIS_DbgPrint(DEBUG_MINIPORT, ("Bad miniport characteristics.\n"));
        return NDIS_STATUS_BAD_CHARACTERISTICS;
    }

  /* Check if mandatory MiniportXxx functions are specified */
  if ((!MiniportCharacteristics->HaltHandler) ||
      (!MiniportCharacteristics->InitializeHandler)||
      (!MiniportCharacteristics->QueryInformationHandler) ||
      (!MiniportCharacteristics->ResetHandler) ||
      (!MiniportCharacteristics->SetInformationHandler))
    {
      NDIS_DbgPrint(DEBUG_MINIPORT, ("Bad miniport characteristics.\n"));
      return NDIS_STATUS_BAD_CHARACTERISTICS;
    }

  if (MiniportCharacteristics->MajorNdisVersion == 0x03)
    {
      if (!MiniportCharacteristics->SendHandler)
        {



```c
            NDIS_DbgPrint(DEBUG_MINIPORT, ("Bad miniport characteristics.\n"));  
            return NDIS_STATUS_BAD_CHARACTERISTICS;  
        }  
    }  
  else if (MiniportCharacteristics->MajorNdisVersion >= 0x04)  
    {  
      /* NDIS 4.0+ */  
      if ((!MiniportCharacteristics->SendHandler) &&  
          (!MiniportCharacteristics->SendPacketsHandler))  
        {  
          NDIS_DbgPrint(DEBUG_MINIPORT, ("Bad miniport characteristics.\n"));  
          return NDIS_STATUS_BAD_CHARACTERISTICS;  
        }  
    }  
  /* TODO: verify NDIS5 and NDIS5.1 */  
  RtlCopyMemory(&Miniport->MiniportCharacteristics, MiniportCharacteristics, MinSize);  
  /*  
   * NOTE: This is VERY unoptimal! Should we store the NDIS_M_DRIVER_BLOCK  
   * structure in the driver extension or what?  
   */  
  Status = IoAllocateDriverObjectExtension(Miniport->DriverObject, (PVOID)TAG('D','I','M','N'),  
                                           sizeof(PNDIS_M_DRIVER_BLOCK), (PVOID*)&MiniportPtr);  
  if (!NT_SUCCESS(Status))  
    {  
      NDIS_DbgPrint(DEBUG_MINIPORT, ("Can't allocate driver object extension.\n"));  
      return NDIS_STATUS_RESOURCES;  
    }  
  *MiniportPtr = Miniport;  
  Miniport->DriverObject->MajorFunction[IRP_MJ_PNP] = NdisIDispatchPnp;  
  Miniport->DriverObject->DriverExtension->AddDevice = NdisIAddDevice;  
  return NDIS_STATUS_SUCCESS;  
}  
```

首先检查 `NDIS_MINIPORT_CHARACTERISTICS` 结构的有效性，然后调用了 `IoAllocateDriverObjectExtension` 分配了 `DriverObjectExtension` 然后就是设置 PNP 例程和 `AddDevice` 例程，当有网卡插入设备时创建设备，让我们看看 `NdisIAddDevice`

```c
NTSTATUS  
NTAPI  
NdisIAddDevice(  
    IN PDRIVER_OBJECT DriverObject,  
    IN PDEVICE_OBJECT PhysicalDeviceObject)  
/*  
 * FUNCTION: Create a device for an adapter found using PnP  
 * ARGUMENTS:  
 *     DriverObject         = Pointer to the miniport driver object  
 *     PhysicalDeviceObject = Pointer to the PDO for our adapter  
 */  
{  
  static const WCHAR ClassKeyName[] = {'C','l','a','s','s','\\'};  
  static const WCHAR LinkageKeyName[] = {'\\','L','i','n','k','a','g','e',0};  
  PNDIS_M_DRIVER_BLOCK Miniport;  
  PNDIS_M_DRIVER_BLOCK *MiniportPtr;  
  WCHAR *LinkageKeyBuffer;  
  ULONG DriverKeyLength;  
  RTL_QUERY_REGISTRY_TABLE QueryTable[2];  
  UNICODE_STRING ExportName;  
  PDEVICE_OBJECT DeviceObject;  
  PLOGICAL_ADAPTER Adapter;  
  NTSTATUS Status;  
  /*  
   * Gain the access to the miniport data structure first.  
   */  
  MiniportPtr = IoGetDriverObjectExtension(DriverObject, (PVOID)TAG('D','I','M','N'));  
  if (MiniportPtr == NULL)  
    {  
      NDIS_DbgPrint(DEBUG_MINIPORT, ("Can't get driver object extension.\n"));  
      return STATUS_UNSUCCESSFUL;  
    }  
  Miniport = *MiniportPtr;  
  /*  
   * Get name of the Linkage registry key for our adapter. It's located under  
   * the driver key for our driver and so we have basicly two ways to do it.  
   * Either we can use IoOpenDriverRegistryKey or compose it using information  
   * gathered by IoGetDeviceProperty. I choosed the second because  
   * IoOpenDriverRegistryKey wasn't implemented at the time of writing.  
   */  
  Status = IoGetDeviceProperty(PhysicalDeviceObject, DevicePropertyDriverKeyName,  
                               0, NULL, &DriverKeyLength);  
  if (Status != STATUS_BUFFER_TOO_SMALL)  
    {  
      NDIS_DbgPrint(DEBUG_MINIPORT, ("Can't get miniport driver key length.\n"));  
      return Status;  
    }  
  LinkageKeyBuffer = ExAllocatePool(PagedPool, DriverKeyLength +  
                                    sizeof(ClassKeyName) + sizeof(LinkageKeyName));  
  if (LinkageKeyBuffer == NULL)  
    {  
      NDIS_DbgPrint(DEBUG_MINIPORT, ("Can't allocate memory for driver key name.\n"));  
      return STATUS_INSUFFICIENT_RESOURCES;  
    }  
  Status = IoGetDeviceProperty(PhysicalDeviceObject, DevicePropertyDriverKeyName,  
                               DriverKeyLength, LinkageKeyBuffer +  
                               (sizeof(ClassKeyName) / sizeof(WCHAR)),  
                               &DriverKeyLength);  
  if (!NT_SUCCESS(Status))  
    {  
      NDIS_DbgPrint(DEBUG_MINIPORT, ("Can't get miniport driver key.\n"));  
      ExFreePool(LinkageKeyBuffer);  
      return Status;  
    }  
  /* Compose the linkage key name. */  
  RtlCopyMemory(LinkageKeyBuffer, ClassKeyName, sizeof(ClassKeyName));  
  RtlCopyMemory(LinkageKeyBuffer + ((sizeof(ClassKeyName) + DriverKeyLength) /  
                sizeof(WCHAR)) - 1, LinkageKeyName, sizeof(LinkageKeyName));  
  NDIS_DbgPrint(DEBUG_MINIPORT, ("LinkageKey: %S.\n", LinkageKeyBuffer));  
  /*  
   * Now open the linkage key and read the "Export" and "RootDevice" values  
   * which contains device name and root service respectively.  
   */  
  RtlZeroMemory(QueryTable, sizeof(QueryTable));  
  RtlInitUnicodeString(&ExportName, NULL);  
  QueryTable[0].Flags = RTL_QUERY_REGISTRY_REQUIRED | RTL_QUERY_REGISTRY_DIRECT;  
  QueryTable[0].Name = L"Export";  
  QueryTable[0].EntryContext = &ExportName;  
  Status = RtlQueryRegistryValues(RTL_REGISTRY_CONTROL, LinkageKeyBuffer,  
                                  QueryTable, NULL, NULL);  
  ExFreePool(LinkageKeyBuffer);  
  if (!NT_SUCCESS(Status))  
    {  
      NDIS_DbgPrint(DEBUG_MINIPORT, ("Can't get miniport device name. (%x)\n", Status));  
      return Status;  
    }  
  /*  
   * Create the device object.  
   */  
  NDIS_DbgPrint(MAX_TRACE, ("creating device %wZ\n", &ExportName));  
  Status = IoCreateDevice(Miniport->DriverObject, sizeof(LOGICAL_ADAPTER),  
    &ExportName, FILE_DEVICE_PHYSICAL_NETCARD,  
    0, FALSE, &DeviceObject);  
  if (!NT_SUCCESS(Status))  
    {  
      NDIS_DbgPrint(MIN_TRACE, ("Could not create device object.\n"));  
      RtlFreeUnicodeString(&ExportName);  
      return Status;  
    }  
  /*  
   * Initialize the adapter structure.  
   */  
  Adapter = (PLOGICAL_ADAPTER)DeviceObject->DeviceExtension;  
  KeInitializeSpinLock(&Adapter->NdisMiniportBlock.Lock);  
  InitializeListHead(&Adapter->ProtocolListHead);  
  Adapter->NdisMiniportBlock.DriverHandle = Miniport;  
  Adapter->NdisMiniportBlock.MiniportName = ExportName;  
  Adapter->NdisMiniportBlock.DeviceObject = DeviceObject;  
  Adapter->NdisMiniportBlock.PhysicalDeviceObject = PhysicalDeviceObject;  
  Adapter->NdisMiniportBlock.NextDeviceObject =  
    IoAttachDeviceToDeviceStack(Adapter->NdisMiniportBlock.DeviceObject,  
                                PhysicalDeviceObject);  
  Adapter->NdisMiniportBlock.OldPnPDeviceState = 0;  
  Adapter->NdisMiniportBlock.PnPDeviceState = NdisPnPDeviceAdded;  
  KeInitializeDpc(&Adapter->NdisMiniportBlock.DeferredDpc, MiniportDpc, (PVOID)Adapter);  
  DeviceObject->Flags &= ~DO_DEVICE_INITIALIZING;  
  return STATUS_SUCCESS;  
}  
```

可以看到首先通过注册表得到设备名，然后创建设备对象，然后初始化了一个 adapter structure，让我们看看这个结构

```c
typedef struct _LOGICAL_ADAPTER  
{  
    NDIS_MINIPORT_BLOCK         NdisMiniportBlock;      /* NDIS defined fields */  
    BOOLEAN                   MiniportBusy;             /* A MiniportXxx routine is executing */  
    PNDIS_MINIPORT_WORK_ITEM    WorkQueueHead;           /* Head of work queue */  
    PNDIS_MINIPORT_WORK_ITEM    WorkQueueTail;           /* Tail of work queue */  
    LIST_ENTRY                ListEntry;                /* Entry on global list */  
    LIST_ENTRY                MiniportListEntry;        /* Entry on miniport driver list */  
    LIST_ENTRY                ProtocolListHead;         /* List of bound protocols */  
    ULONG                     MediumHeaderSize;         /* Size of medium header */  
    HARDWARE_ADDRESS          Address;                  /* Hardware address of adapter */  
```



### `LOGICAL_ADAPTER` 结构体定义

```c
    ULONG                       AddressLength;         /* Length of hardware address */
    
    PUCHAR                      LookaheadBuffer;        /* Pointer to lookahead buffer */
    
    ULONG                       LookaheadLength;        /* Length of lookahead buffer */
    
    PMINIPORT_BUGCHECK_CONTEXT  BugcheckContext;        /* Adapter's shutdown handler */
    
} LOGICAL_ADAPTER, *PLOGICAL_ADAPTER
```

第一个是 `NDIS_MINIPORT_BLOCK` 结构，它包括的东西很多很全这里就不贴了，可以到 `DDK` 去看 哈哈。

### 协议注册流程

好了按照 `passthru` 下面它应该调用 `NdisRegisterProtocol` 注册协议了，它有一个 `NDIS_PROTOCOL_CHARACTERISTICS` 结构必须先初始化，和上面那个差不多吧 哈哈

```c
    // Now register the protocol.
    //
    NdisZeroMemory(&PChars, sizeof(NDIS_PROTOCOL_CHARACTERISTICS));
    
    PChars.MajorNdisVersion = PASSTHRU_PROT_MAJOR_NDIS_VERSION;
    PChars.MinorNdisVersion = PASSTHRU_PROT_MINOR_NDIS_VERSION;
    
    //
    // Make sure the protocol-name matches the service-name
    // (from the INF) under which this protocol is installed.
    // This is needed to ensure that NDIS can correctly determine
    // the binding and call us to bind to miniports below.
    //
    NdisInitUnicodeString(&Name, L"Passthru");  // Protocol name
    PChars.Name = Name;
    
    PChars.OpenAdapterCompleteHandler = PtOpenAdapterComplete;
    PChars.CloseAdapterCompleteHandler = PtCloseAdapterComplete;
    PChars.SendCompleteHandler = PtSendComplete;
    PChars.TransferDataCompleteHandler = PtTransferDataComplete;
    PChars.ResetCompleteHandler = PtResetComplete;
    PChars.RequestCompleteHandler = PtRequestComplete;
    PChars.ReceiveHandler = PtReceive;
    PChars.ReceiveCompleteHandler = PtReceiveComplete;
    PChars.StatusHandler = PtStatus;
    PChars.StatusCompleteHandler = PtStatusComplete;
    PChars.BindAdapterHandler = PtBindAdapter;
    PChars.UnbindAdapterHandler = PtUnbindAdapter;
    PChars.UnloadHandler = PtUnloadProtocol;
    PChars.ReceivePacketHandler = PtReceivePacket;
    PChars.PnPEventHandler= PtPNPHandler;
    
    NdisRegisterProtocol(&Status,
               &ProtHandle,
               &PChars,
               sizeof(NDIS_PROTOCOL_CHARACTERISTICS));
```

可以看到在 `NDIS_PROTOCOL_CHARACTERISTICS` 中除了指定兼容版本以外还保存了强制的和非强制的 `ProtocolXxx` 函数地址。下面我们看一看 `NdisRegisterProtocol` 函数

```c
VOID
EXPORT
NdisRegisterProtocol(
    OUT PNDIS_STATUS                 Status,
    OUT PNDIS_HANDLE                 NdisProtocolHandle,
    IN  PNDIS_PROTOCOL_CHARACTERISTICS ProtocolCharacteristics,
    IN  UINT                        CharacteristicsLength)
/*
 * FUNCTION: Registers an NDIS driver's ProtocolXxx entry points
 * ARGUMENTS:
 *     Status                  = Address of buffer for status information
 *     NdisProtocolHandle      = Address of buffer for handle used to identify the driver
 *     ProtocolCharacteristics = Pointer to NDIS_PROTOCOL_CHARACTERISTICS structure
 *     CharacteristicsLength   = Size of structure which ProtocolCharacteristics targets
 * NOTES:
 *     - you *must* set NdisProtocolHandle before doing anything that could wind up
 *       getting BindAdapterHandler, as it will probably call OpenAdapter with this handle
 *     - the above implies that the initialization of the protocol block must be complete
 *       by then
 * TODO:
 *     - break this function up - probably do a 'ndisRefreshProtocolBindings' function
 *     - make this thing able to handle >1 protocol
 */
{
  PPROTOCOL_BINDING Protocol;
  NTSTATUS NtStatus;
  UINT MinSize;
  HANDLE DriverKeyHandle = NULL;
  PKEY_VALUE_PARTIAL_INFORMATION KeyInformation = NULL;
  WCHAR *DataPtr;
  
  NDIS_DbgPrint(MAX_TRACE, ("Called.\n"));
  
  /* first validate the PROTOCOL_CHARACTERISTICS */
  switch (ProtocolCharacteristics->MajorNdisVersion)
    {
    case 0x03:
      /* we don't really want to support ndis3 drivers - so we complain for now */
      NDIS_DbgPrint(MID_TRACE, ("NDIS 3 protocol attempting to register\n"));
      MinSize = sizeof(NDIS30_PROTOCOL_CHARACTERISTICS);
      break;
    case 0x04:
      MinSize = sizeof(NDIS40_PROTOCOL_CHARACTERISTICS);
      break;
    case 0x05:
      MinSize = sizeof(NDIS50_PROTOCOL_CHARACTERISTICS);
      break;
    default:
      *Status = NDIS_STATUS_BAD_VERSION;
      NDIS_DbgPrint(MIN_TRACE, ("Incorrect characteristics size\n"));
      return;
    }
  
  if (CharacteristicsLength < MinSize)
    {
      NDIS_DbgPrint(DEBUG_PROTOCOL, ("Bad protocol characteristics.\n"));
      *Status = NDIS_STATUS_BAD_CHARACTERISTICS;
      return;
    }
  
  /* set up the protocol block */
  Protocol = ExAllocatePool(NonPagedPool, sizeof(PROTOCOL_BINDING));
  if (!Protocol)
    {
      NDIS_DbgPrint(MIN_TRACE, ("Insufficient resources.\n"));
      *Status = NDIS_STATUS_RESOURCES;
      return;
    }
  
  RtlZeroMemory(Protocol, sizeof(PROTOCOL_BINDING));
  RtlCopyMemory(&Protocol->Chars, ProtocolCharacteristics, MinSize);
  NtStatus = RtlUpcaseUnicodeString(&Protocol->Chars.Name, &ProtocolCharacteristics->Name, TRUE);
  if (!NT_SUCCESS(NtStatus))
    {
      NDIS_DbgPrint(MIN_TRACE, ("Insufficient resources.\n"));
      ExFreePool(Protocol);
      *Status = NDIS_STATUS_RESOURCES;
      return;
    }
  
  KeInitializeSpinLock(&Protocol->Lock);
  InitializeListHead(&Protocol->AdapterListHead);
  /*
   * bind the protocol to all of its miniports
   *
   * open registry path
   * get list of devices from Bind key
   * call BindAdapterHandler for each
   */
  {
    OBJECT_ATTRIBUTES ObjectAttributes;
    UNICODE_STRING RegistryPath;
    WCHAR *RegistryPathStr;
    
    RegistryPathStr = ExAllocatePoolWithTag(PagedPool, sizeof(SERVICES_KEY) + ProtocolCharacteristics->Name.Length + sizeof(LINKAGE_KEY), NDIS_TAG + __LINE__);
    if(!RegistryPathStr)
      {
        NDIS_DbgPrint(MIN_TRACE, ("Insufficient resources.\n"));
        ExFreePool(Protocol);
        *Status = NDIS_STATUS_RESOURCES;
        return;
      }
    
    wcscpy(RegistryPathStr, SERVICES_KEY);
    wcsncat(RegistryPathStr, ((WCHAR *)ProtocolCharacteristics->Name.Buffer), ProtocolCharacteristics->Name.Length / sizeof(WCHAR));
    RegistryPathStr[wcslen(SERVICES_KEY)+ProtocolCharacteristics->Name.Length/sizeof(WCHAR)] = 0;
    wcscat(RegistryPathStr, LINKAGE_KEY);
    RtlInitUnicodeString(&RegistryPath, RegistryPathStr);
    NDIS_DbgPrint(MAX_TRACE, ("Opening configuration key: %wZ\n", &RegistryPath));
    InitializeObjectAttributes(&ObjectAttributes, &RegistryPath, OBJ_CASE_INSENSITIVE, NULL, NULL);
    NtStatus = ZwOpenKey(&DriverKeyHandle, KEY_READ, &ObjectAttributes);
    ExFreePool(RegistryPathStr);
    
    if(!NT_SUCCESS(NtStatus))
      {
        NDIS_DbgPrint(MID_TRACE, ("Unable to open protocol configuration\n"));
        ExFreePool(Protocol);
        *Status = NDIS_STATUS_FAILURE;
        return;
      }
  }
  
  NDIS_DbgPrint(MAX_TRACE, ("Successfully opened the registry configuration\n"));
  
  {
    UNICODE_STRING ValueName;
    ULONG ResultLength;
    RtlInitUnicodeString(&ValueName, L"Bind");
    NtStatus = ZwQueryValueKey(DriverKeyHandle, &ValueName, KeyValuePartialInformation, NULL, 0, &ResultLength);
    
    if(NtStatus != STATUS_BUFFER_OVERFLOW && NtStatus != STATUS_BUFFER_TOO_SMALL)
      {
        NDIS_DbgPrint(MID_TRACE, ("Unable to query the Bind value for size\n"));
        ZwClose(DriverKeyHandle);
        ExFreePool(Protocol);
        *Status = NDIS_STATUS_FAILURE;
        return;
      }
```



```
    KeyInformation = ExAllocatePoolWithTag(PagedPool, sizeof(KEY\_VALUE\_PARTIAL\_INFORMATION) + ResultLength, NDIS\_TAG + \_\_LINE\_\_);

    if(!KeyInformation)

      {

        NDIS\_DbgPrint(MIN\_TRACE, ("Insufficient resources.\\n"));

        ZwClose(DriverKeyHandle);

        ExFreePool(Protocol);

        \*Status = NDIS\_STATUS\_FAILURE;

        return;

      }

    NtStatus = ZwQueryValueKey(DriverKeyHandle, &ValueName, KeyValuePartialInformation, KeyInformation,

        sizeof(KEY\_VALUE\_PARTIAL\_INFORMATION) + ResultLength, &ResultLength);

    if(!NT\_SUCCESS(NtStatus))

      {

        NDIS\_DbgPrint(MIN\_TRACE, ("Unable to query the Bind value\\n"));

        ZwClose(DriverKeyHandle);

        ExFreePool(KeyInformation);

        ExFreePool(Protocol);

        \*Status = NDIS\_STATUS\_FAILURE;

        return;

      }

  }

  for (DataPtr = (WCHAR \*)KeyInformation->Data;

       \*DataPtr != 0;

       DataPtr += wcslen(DataPtr) + 1)

    {

      /\* BindContext is for tracking pending binding operations \*/

      VOID \*BindContext = 0;

      NDIS\_STRING DeviceName;

      NDIS\_STRING RegistryPath;

      WCHAR \*RegistryPathStr = NULL;

      ULONG PathLength = 0;

      RtlInitUnicodeString(&DeviceName, DataPtr);  /\* we know this is 0-term \*/

      /\*

       \* RegistryPath should be:

       \*     \\Registry\\Machine\\System\\CurrentControlSet\\Services\\Nic1\\Parameters\\Tcpip

       \*

       \*  This is constructed as follows:

       \*     SERVICES\_KEY + extracted device name + Protocol name from characteristics

       \*/

      PathLength = sizeof(SERVICES\_KEY) +                          /\* \\Registry\\Machine\\System\\CurrentControlSet\\Services\\ \*/

          wcslen( DataPtr + 8 ) \* sizeof(WCHAR) + /\* Adapter1  (extracted from \\Device\\Adapter1)          \*/

          sizeof(PARAMETERS\_KEY) +                       /\* \\Parameters\\                                  \*/

          ProtocolCharacteristics->Name.Length + sizeof(WCHAR);                 /\* Tcpip                      \*/

      RegistryPathStr = ExAllocatePool(PagedPool, PathLength);

      if(!RegistryPathStr)

        {

          NDIS\_DbgPrint(MIN\_TRACE, ("insufficient resources.\\n"));

          ExFreePool(KeyInformation);

          ExFreePool(Protocol);

          \*Status = NDIS\_STATUS\_RESOURCES;

          return;

        }

      wcscpy(RegistryPathStr, SERVICES\_KEY);

      wcscat(RegistryPathStr, DataPtr + 8 );

      wcscat(RegistryPathStr, PARAMETERS\_KEY);

      wcsncat(RegistryPathStr, ProtocolCharacteristics->Name.Buffer, ProtocolCharacteristics->Name.Length / sizeof(WCHAR) );

      RegistryPathStr\[PathLength/sizeof(WCHAR) - 1\] = 0;

      RtlInitUnicodeString(&RegistryPath, RegistryPathStr);

      NDIS\_DbgPrint(MAX\_TRACE, ("Calling protocol's BindAdapter handler with DeviceName %wZ and RegistryPath %wZ\\n",

          &DeviceName, &RegistryPath));

      /\* XXX SD must do something with bind context \*/

      \*NdisProtocolHandle = Protocol;

        {

          BIND\_HANDLER BindHandler = ProtocolCharacteristics->BindAdapterHandler;

          if(BindHandler)

            BindHandler(Status, BindContext, &DeviceName, &RegistryPath, 0);

          else

            NDIS\_DbgPrint(MID\_TRACE, ("No protocol bind handler specified\\n"));

        }

      /\*

      (\*(Protocol->Chars.BindAdapterHandler))(Status, BindContext, &DeviceName, &RegistryPath, 0);

      \*/

      if(\*Status == NDIS\_STATUS\_SUCCESS)

        {

          /\* Put protocol binding struct on global list \*/

          ExInterlockedInsertTailList(&ProtocolListHead, &Protocol->ListEntry, &ProtocolListLock);

        }

      /\*

      else if(\*Status != NDIS\_STATUS\_PENDING)

        {

          // what to do here?

        }

       \*/

    }

  \*Status           = NDIS\_STATUS\_SUCCESS;

}
```

它首先判断 `NDIS_PROTOCOL_CHARACTERISTICS` 的有效性，然后初始化了一个 `PPROTOCOL_BINDING Protocol` 结构，然后从 `Bind` key 获得设备链表，获得设备名，并创建了这样 `\\Registry\\Machine\\System\\CurrentControlSet\\Services\\Nic1\\Parameters\\Tcpip` 的注册表键，然后调用我们传进去的 `BindAdapterHandler` 进行绑定设备。并把 `Protocol` 加如链表。从 `*NdisProtocolHandle = Protocol` 我们知道 `NdisProtocolHandle` 指向 `Protocol`，那让我们看看这个 `Protocol` 是什么，

```
typedef struct _PROTOCOL\_BINDING {

    LIST\_ENTRY                 ListEntry;      /\* Entry on global list \*/

    KSPIN\_LOCK                 Lock;           /\* Protecting spin lock \*/

    NDIS\_PROTOCOL\_CHARACTERISTICS Chars;      /\* Characteristics \*/

    WORK\_QUEUE\_ITEM            WorkItem;       /\* Work item \*/

    LIST\_ENTRY                 AdapterListHead; /\* List of adapter bindings \*/

} PROTOCOL\_BINDING, \*PPROTOCOL\_BINDING;
```

这里面有个指向 `adapter bindings` 的链表，来让我们看看 `adapter binding`

```
typedef struct _ADAPTER\_BINDING {

    NDIS\_OPEN\_BLOCK NdisOpenBlock;                  /\* NDIS defined fields \*/

    LIST\_ENTRY         ListEntry;             /\* Entry on global list \*/

    LIST\_ENTRY         ProtocolListEntry;     /\* Entry on protocol binding adapter list \*/

    LIST\_ENTRY         AdapterListEntry;      /\* Entry on logical adapter list \*/

    KSPIN\_LOCK         Lock;                  /\* Protecting spin lock \*/

    PPROTOCOL\_BINDING ProtocolBinding;        /\* Protocol that opened adapter \*/

    PLOGICAL\_ADAPTER  Adapter;                /\* Adapter opened by protocol \*/

} ADAPTER\_BINDING, \*PADAPTER\_BINDING;
```

我们看到第一项是一个 `NdisOpenBlock` 结构，现在很多的 NDIS 放火墙就是通过注册协议根据返回的 `NdisProtocolHandle` 来找到 `NdisOpenBlock` 结构的，因为 `NdisOpenBlock` 结构是用来描述和该 `NDIS protocol` 有联系的所有 `NDIS miniport` 和该 `NDIS protocol` 绑定的状态每个绑定的 `send(packets)handler` 和 `receive(packet)handler` 都在这个 `NDIS open block` 里面。

```
struct _NDIS\_OPEN\_BLOCK

{

#ifdef \_\_cplusplus

　　NDIS\_COMMON\_OPEN\_BLOCK NdisCommonOpenBlock;

#else

　　NDIS\_COMMON\_OPEN\_BLOCK;

#endif

#if defined(NDIS\_WRAPPER)

　　//

　　// The stuff below is for CO drivers/protocols. This part is not allocated for CL drivers.

　　//

　　struct _NDIS\_OPEN\_CO

　　{

　　....

　　};

#endif

};

typedef struct _NDIS\_COMMON\_OPEN\_BLOCK

{

　　PVOID                    MacHandle;           // needed for backward compatibility

　　NDIS\_HANDLE              BindingHandle;       // Miniport's open context

　　PNDIS\_MINIPORT\_BLOCK     MiniportHandle;      // pointer to the miniport

　　PNDIS\_PROTOCOL\_BLOCK     ProtocolHandle;      // pointer to our protocol

　　NDIS\_HANDLE              ProtocolBindingContext;// context when calling ProtXX funcs

　　PNDIS\_OPEN\_BLOCK         MiniportNextOpen;    // used by adapter's OpenQueue

　　PNDIS\_OPEN\_BLOCK         ProtocolNextOpen;    // used by protocol's OpenQueue

　　NDIS\_HANDLE              MiniportAdapterContext; // context for miniport

　　BOOLEAN                    Reserved1;

　　BOOLEAN                    Reserved2;

　　BOOLEAN                    Reserved3;

　　BOOLEAN                    Reserved4;

　　PNDIS\_STRING              BindDeviceName;

　　KSPIN\_LOCK                Reserved5;

　　PNDIS\_STRING              RootDeviceName;

　　//

　　// These are referenced by the macros used by protocols to call.

　　// All of the ones referenced by the macros are internal NDIS handlers for the miniports

　　//

　　union

　　{

　　　　SEND\_HANDLER            SendHandler;

　　　　WAN\_SEND\_HANDLER       WanSendHandler;

　　};

　　TRANSFER\_DATA\_HANDLER     TransferDataHandler;

　　//

　　// These are referenced internally by NDIS

　　//

　　SEND\_COMPLETE\_HANDLER     SendCompleteHandler;

　　TRANSFER\_DATA\_COMPLETE\_HANDLER TransferDataCompleteHandler;

　　RECEIVE\_HANDLER            ReceiveHandler;

　　RECEIVE\_COMPLETE\_HANDLER  ReceiveCompleteHandler;
```



WAN_RECEIVE_HANDLER WanReceiveHandler;  

REQUEST_COMPLETE_HANDLER RequestCompleteHandler;  

```
//
// NDIS 4.0 extensions
//
```

RECEIVE_PACKET_HANDLER ReceivePacketHandler;  

SEND_PACKETS_HANDLER SendPacketsHandler;  

```
//
// More Cached Handlers
//
```

RESET_HANDLER ResetHandler;  

REQUEST_HANDLER RequestHandler;  

RESET_COMPLETE_HANDLER ResetCompleteHandler;  

STATUS_HANDLER StatusHandler;  

STATUS_COMPLETE_HANDLER StatusCompleteHandler;  

```
#if defined(NDIS_WRAPPER)
....
#endif
```

`NDIS_COMMON_OPEN_BLOCK`

需要处理的，是`ndis open block`里面的`SendHandler`、`ReceiveHandler`、`WanReceiveHandler`、`ReceivePacketHandler`和`SendPacketsHandler`。

好了在`passthru`里注册好协议以后调用`NdisIMAssociateMiniport(DriverHandle, ProtHandle)`就可以了，因为`passthru`是中间层驱动程序，下面就应该分开说`miniport`和`PROTOCOL`，象`MPInitialize`，还有上面我们调用了`BindAdapterHandler`，我们应该说我们的`BindAdapterHandler`例程了，还有接包发包，和注册的各种例程，但现在就到这吧，要说下去不知什么时候能说完，这仅仅是一个开始，下面复杂的东西还很多，我这个小菜鸟就不现丑了。

哈哈

有什么问题可以讨论 `qq:498793761`
