
```c
NTSTATUS
  PsdoDispatchRead(
    IN PDEVICE_OBJECT  DeviceObject,
    IN PIRP  Irp
    )
{
  PVOID Buf;    //Buffer provided by user program
  ULONG BufLen;  //Buffer length for user provided buffer
  LONGLONG Offset;//Buffer Offset
  PVOID DataBuf;  //Buffer provided by Driver
  ULONG DataLen;  //Buffer length for Driver Data Buffer
  ULONG ByteTransferred;
  PMDL mdl;
  PULONG pVer;
  PIO_STACK_LOCATION p_IO_STK;
  PDEVICE_EXTENSION p_DVCEXT;

  DbgPrint("IRP_MJ_READ : Begin\r\n");
  //Get I/o Stack Location & Device Extension
  // 堆栈中包含用户缓冲区的信息
  p_IO_STK = IoGetCurrentIrpStackLocation(Irp);
  p_DVCEXT = DeviceObject->DeviceExtension;

  //Get Driver Data Buffer & Length
  //获得驱动程序的缓冲区
  DataBuf = p_DVCEXT->DataBuffer;
  if (DataBuf == NULL)
    DataLen = 0;
  else
    DataLen = p_DVCEXT->DeviceBufferSzie;

  //Get User Output Buffer & Length
  //获得用户输出缓冲区以及长度
  BufLen = p_IO_STK->Parameters.Read.Length;
  Buf = Irp->UserBuffer;
  if (Irp->RequestorMode!= KernelMode)
  {  //异常处理
    _try{
      //为用户缓冲区创建 MDL
      mdl=IoAllocateMdl(Buf,BufLen,FALSE,TRUE,Irp);
      //要读设备，就要有写缓冲区的权利，不知道理解的对不？？
      MmProbeAndLockPages(mdl,UserMode,IoWriteAccess);
    }
    _except(EXCEPTION_CONTINUE_EXECUTION){
      return CompleteRequest(Irp,GetExceptionCode(),0);
    }
  }
  DbgPrint("Output Buffer Length : %d\r\n", BufLen);
  DbgPrint("Driver Data Length : %d\r\n", DataLen);
  //
  if (BufLen <= DataLen) {
    ByteTransferred = BufLen;   
  } else {
    ByteTransferred = DataLen;
  }
  //把数据复制到用户缓冲区
  RtlCopyMemory(
    Buf, DataBuf, ByteTransferred);
  //完成对 IRP 的处理
  CompleteRequest(Irp, STATUS_SUCCESS, ByteTransferred);

  DbgPrint("IRP_MJ_READ : End\r\n");
  return STATUS_SUCCESS;
}
```

以上代码测试成功，一直没搜到别人写的 Neither 部分的代码，所以自己参考资料写了这个

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `BufferedIO_PW.rar` |
|---|---|
