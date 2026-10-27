
``` 
#pragma PAGEDCODE

NTSTATUS ReadAndWriteSectors(PDEVICE_OBJECT pDevObj,LARGE_INTEGER lDiskOffset,PVOID pbuf,BOOLEAN iSreadorWrite)

{

 NTSTATUS status=STATUS_SUCCESS;

// CHAR    *sBuf;

 KEVENT    Event;

    SIZE_T    size = 512;                             //每次读写的大小

 PIRP    pIrp = NULL;       //创建读写的IRP

 IO_STATUS_BLOCK  ioStatus;  

// DWORD               i;

 

 KeInitializeEvent(&Event, NotificationEvent, FALSE);        //创建一个通知事件来进行U盘扇区读写完成

// DbgPrint("ReadAndWriteSectors pDevObj :%lX\n",pDevObj);

// DbgPrint("ReadAndWriteSectors QuaPart :%lX \n",lDiskOffset->QuadPart);

 DbgPrint("ReadAndWriteSectors QuaPart:%X\n",lDiskOffset.QuadPart);

 lDiskOffset.QuadPart=lDiskOffset.QuadPart*512;

// sBuf =(CHAR *) ExAllocatePool(NonPagedPool, size);

// memset(sBuf, '0', size);

 if(iSreadorWrite==TRUE)

 {

  pIrp = IoBuildSynchronousFsdRequest(IRP_MJ_READ, pDevObj, pbuf, size, &lDiskOffset, &Event, &ioStatus);

  DbgPrint("TRUE");

 }

 else

 {

  pIrp = IoBuildSynchronousFsdRequest(IRP_MJ_WRITE, pDevObj, pbuf, size, &lDiskOffset, &Event, &ioStatus);

  DbgPrint("FALSE");

 }

 

 

 if (!pIrp) 

 {

  

  //ExFreePool(sBuf);

  KdPrint(("Creat IRP faile\n"));

  return STATUS_INSUFFICIENT_RESOURCES;

 }

 

 status = IoCallDriver(pDevObj, pIrp);

 

 if (status == STATUS_PENDING) 

 {

  KeWaitForSingleObject(&Event, Executive, KernelMode, FALSE, NULL);

  status = ioStatus.Status;

 }

 

/* 
 for (i=0;i<size;i++)
 {
  KdPrint(("%X \n",pbuf[i]&0XFF));
  if ((i % 16) == 15)
  {
   KdPrint(("\n"));
  }
 }
*/ 

// ExFreePool(sBuf);

 

 return status;

}
```

这样基本功能就实现了，一个简单的驱动就做出来了，然后把参数什么的都填好，把`IOCTRL`和应用程序的通信假设好，一个完整的工程就OK了，在写上驱动安装，卸载函数，做成`DLL`也可以供其他语言的程序来使用了，这些我都完善好了。但是总是感觉这不是最好的办法。如果大家有别的好的办法，请告知，谢谢，本人QQ:`253229030`欢迎一起交流！

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [RWUDisk.rar](javascript:void(0);)
  （131.25kb，205次下载）
