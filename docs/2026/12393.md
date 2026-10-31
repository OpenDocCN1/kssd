
# 山寨U盘加解密

翻阅硬盘，偶然发现了很久以前的许多代码，当初只是改着玩一玩，也没有往深了写，发出来坐等大牛各种拍砖。大概看了一下，ms创建了一个线程来处理读写请求。改的`diskperf`，实在很挫，难登大雅之堂。![](https://bbs.kanxue.com/view/img/face/50.gif)

```c
VOID
DiskPerfReadWriteThread(
  IN PVOID Context
)
{
  PDEVICE_OBJECT pDevice=(PDEVICE_OBJECT)Context;
  PDEVICE_EXTENSION DevExt=pDevice->DeviceExtension;
  PLIST_ENTRY ReqEntry=NULL;
  NTSTATUS status;
  PIRP Irp=NULL;
  PIO_STACK_LOCATION Irpsp=NULL;
  ULONG length=0;
  LARGE_INTEGER offset={0};
  PUCHAR sysBuf=NULL;
  KeSetPriorityThread(KeGetCurrentThread(), LOW_REALTIME_PRIORITY);
  for (; ; )
  {
    KeWaitForSingleObject(
      &DevExt->ReqEvent,
      Executive,
      KernelMode,
      FALSE,
      NULL);
    if (DevExt->ThreadTermFlag)
    {
      PsTerminateSystemThread(STATUS_SUCCESS);
      return ;
    }
    while (ReqEntry=ExInterlockedRemoveHeadList(&DevExt->ReqList, &DevExt->ReqLock))
    {
      Irp=CONTAINING_RECORD(ReqEntry, IRP, Tail.Overlay.ListEntry);
      Irpsp=IoGetCurrentIrpStackLocation(Irp);
      if (NULL==Irp->MdlAddress)
      {
        sysBuf=(PUCHAR)Irp->UserBuffer;
      }
      else
      {
        sysBuf=(PUCHAR)MmGetSystemAddressForMdlSafe(Irp->MdlAddress, NormalPagePriority);
      }
      /*      if (gPDevice==DevExt->PhysicalDeviceObject || gPDevice==NULL)
      {
        IoSkipCurrentIrpStackLocation(Irp);
        IoCallDriver(DevExt->TargetDeviceObject, Irp);
        continue;
      }*/
      /*      if (DiskPerfIsCVolume(DevExt))
      {
        IoSkipCurrentIrpStackLocation(Irp);
        IoCallDriver(DevExt->TargetDeviceObject, Irp);
        continue;
      }*/
      if (!DevExt->VolumeOnline)
      {
        IoSkipCurrentIrpStackLocation(Irp);
        IoCallDriver(DevExt->TargetDeviceObject, Irp);
        continue;
      }
      if(IRP_MJ_READ==Irpsp->MajorFunction)
      {
        IRP_CONTEXT readContext;
        KEVENT event;
        offset=Irpsp->Parameters.Read.ByteOffset;
        length=Irpsp->Parameters.Read.Length;
        readContext.DataBuf=sysBuf;
        readContext.length=length;
        readContext.offset=offset;
        KeInitializeEvent(&event, NotificationEvent, FALSE);
        readContext.Event=&event;
        IoCopyCurrentIrpStackLocationToNext(Irp);
        IoSetCompletionRoutine(
          Irp,
          DiskPerfReadCompletion,
          &readContext,
          TRUE,
          TRUE,
          TRUE);
        status=IoCallDriver(DevExt->TargetDeviceObject, Irp);
        if (status== STATUS_PENDING)
        {
          KeWaitForSingleObject(
            &event,
            Executive,
            KernelMode,
            FALSE,
            NULL);
          status=Irp->IoStatus.Status;
        }
        Irp->IoStatus.Status=STATUS_SUCCESS;
//        IoCompleteRequest(Irp, IO_DISK_INCREMENT);
        continue;
      }
      else
      {
        int i=0;
        IRP_CONTEXT readContext;
        KEVENT event;
        offset=Irpsp->Parameters.Write.ByteOffset;
        length=Irpsp->Parameters.Write.Length;
        readContext.DataBuf=sysBuf;
        readContext.length=length;
        readContext.offset=offset;
        KeInitializeEvent(&event, NotificationEvent, FALSE);
        readContext.Event=&event;
        for(i=0; i<length; i++)
        {
          sysBuf[i]=sysBuf[i]^0xff;
        }
        IoCopyCurrentIrpStackLocationToNext(Irp);
        IoSetCompletionRoutine(
          Irp,
          DiskPerfWriteCompletion,
          &readContext,
          TRUE,
          TRUE,
          TRUE);
        status=IoCallDriver(DevExt->TargetDeviceObject, Irp);
        if (status== STATUS_PENDING)
        {
          KeWaitForSingleObject(
            &event,
            Executive,
            KernelMode,
            FALSE,
            NULL);
          status=Irp->IoStatus.Status;
        }
        Irp->IoStatus.Status=STATUS_SUCCESS;
//        IoCompleteRequest(Irp, IO_DISK_INCREMENT);
        continue;
      }
    }
  }
}
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [diskperf.rar](javascript:void(0);)
  （14.15kb，59次下载）
