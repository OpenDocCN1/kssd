
```c
//快速 IO 函数例程数组 
FAST_IO_DISPATCH g_fastIoDispatch = 
{ 
sizeof(FAST_IO_DISPATCH), 
FsFilterFastIoCheckIfPossible, 
FsFilterFastIoRead, 
FsFilterFastIoWrite, 
FsFilterFastIoQueryBasicInfo, 
FsFilterFastIoQueryStandardInfo, 
FsFilterFastIoLock, 
FsFilterFastIoUnlockSingle, 
FsFilterFastIoUnlockAll, 
FsFilterFastIoUnlockAllByKey, 
FsFilterFastIoDeviceControl, 
NULL, 
NULL, 
FsFilterFastIoDetachDevice, 
FsFilterFastIoQueryNetworkOpenInfo, 
NULL, 
FsFilterFastIoMdlRead, 
FsFilterFastIoMdlReadComplete, 
FsFilterFastIoPrepareMdlWrite, 
FsFilterFastIoMdlWriteComplete, 
FsFilterFastIoReadCompressed, 
FsFilterFastIoWriteCompressed, 
FsFilterFastIoMdlReadCompleteCompressed, 
FsFilterFastIoMdlWriteCompleteCompressed, 
FsFilterFastIoQueryOpen, 
NULL, 
NULL, 
NULL, 
};
```

快速 IO 例程实现基本都是一样的，直接向下层转发，并返回 `FALSE`。

代码:

```c
BOOLEAN FsFilterFastIoCheckIfPossible( 
IN PFILE_OBJECT FileObject, 
IN PLARGE_INTEGER FileOffset, 
IN ULONG Length, 
IN BOOLEAN Wait, 
IN ULONG LockKey, 
IN BOOLEAN CheckForReadOperation, 
OUT PIO_STATUS_BLOCK IoStatus, 
IN PDEVICE_OBJECT DeviceObject 
) 
{ 
PDEVICE_OBJECT nextDeviceObject = ((PFSFILTER_DEVICE_EXTENSION)DeviceObject->DeviceExtension)->AttachedToDeviceObject; 
PFAST_IO_DISPATCH fastIoDispatch = nextDeviceObject->DriverObject->FastIoDispatch; 

if (VALID_FAST_IO_DISPATCH_HANDLER(fastIoDispatch, FastIoCheckIfPossible)) 
{ 
return (fastIoDispatch->FastIoCheckIfPossible)( 
FileObject, 
FileOffset, 
Length, 
Wait, 
LockKey, 
CheckForReadOperation, 
IoStatus, 
nextDeviceObject); 
} 

return FALSE; 
}
```

到此，整个 DEMO 驱动就完成了。可见去掉了过滤驱动中的旁枝末节，整个过滤驱动框架是简单明了的。

![](img/2853eb364d985717c6216d1447d8a2fb.jpg)

您正在看的文章来自赏金论坛 `http://www.sgoldcn.com`,原文地址:`http://www.sgoldcn.com/read.php?tid=1417`

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) | `Code_FileSystemFilterDemo.rar` |
