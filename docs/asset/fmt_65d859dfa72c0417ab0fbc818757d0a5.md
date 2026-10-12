
# [逆向]Dell 风扇控制驱动源码(已修正并重新上传)

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-325.htm`](https://www.pediy.com/kssd/pediy08/pediy8-325.htm)

附件下载

---

1. `C` 源代码中有一处遗漏，会导致驱动向应用程序返回结果错误，在 `DellFanDriver.c` 中
2. 逆向分析的 `IDB` 文件中，`INIT:00010818` 处有个笔误，这一行中的 `IRP_DEVICE_CONTROL` 应该是 `IRP_MJ_DEVICE_CONTROL`，中间丢了"MJ"
3. `DispatchIoControl` 函数中一处数据结构分析有误，已修正

刚开学天天赋闲，无聊中翻起老帖，无意中看到 `Isaiah` 兄写的控制 `Dell` 风扇的程序，可惜的是没有驱动源码，于是心血来潮，搞了三个小时，终于把 `Dell` 风扇控制驱动的源码搞定了，附件里有全部 `C` 源码和 `IDA` 分析生成的 `IDB` 文件(需要 `IDA4.9` 或者看雪论坛的 `5.0` 版本方可正常打开)，另外工程中附带了 `Isaiah` 兄写的程序(没有修改)供大家参考，在此对 `Isaiah` 兄表示感谢

`Isaiah` 兄的帖子

`http://bbs.pediy.com/showthread.php?s=&threadid=27805&highlight=dell`

限于本人水平，加之驱动没有测试过，程序中错误和疏漏一定不少，明天我会到学校机房找一台 `dell` 机器测试一下，如果有错误还请各位多多指教。

由于懒得写驱动程序中那一堆令人厌烦的例行公事式的代码，程序的框架是用 `DriverWorks` 生成的(没有用 `Driver C++ FrameWork`)。所以源代码中基本上是框架代码，控制风扇的核心在于 `DispatchIoControl` 函数中(`C` 源码中对应的函数名是 `DellFanDriverDeviceIoControlDispatch`，`DriverWorks` 生成的函数名)，大家也只需要看这个函数就可以了，如果想看全部代码请看打开 `IDB` 文件看，那儿的代码要简洁明了很多。

另外还有一点要说明的，驱动源码中更改了设备名称和符号链接名，因为我觉得原来的太丑了

原设备名为 `\\Device\\#!#!`

原符号链接名为 `\\DosDevices\\#!#!`

源代码中设备名为 `\\Device\\DellFanDriver`

符号链接名为 `\\??\\DellFanDriver`

打开驱动的时候，`CreateFile` 指定的文件名也要做相应修改

代码中，直接操作风扇的是这两句

`out 0B2h, al; Interrupt Controller #2, 8259A`

`out 84h, al;`

---

以下就是驱动控制风扇的代码(只列出最核心的函数，其他不是很重要的可以去看附件中的代码和 `IDB` 文件)

两个`//**********`之间的是核心

```c
///////////////////////////////////////////////////////////////////////////////////////////////////
//  DellFanDriverDeviceIoControlDispatch
//     Dispatch routine for IRP_MJ_DEVICE_CONTROL requests.
//
//  Arguments:
//     IN  DeviceObject
//            pointer to the device object for our device
//
//     IN  Irp
//            the device i/o control IRP
//
//  Return Value:
//     NT status code.
//
NTSTATUS DellFanDriverDeviceIoControlDispatch(
  IN  PDEVICE_OBJECT  DeviceObject,
  IN  PIRP            Irp
  )
{
  PIO_STACK_LOCATION                irpStack;
  NTSTATUS                          status;
  PDELLFANDRIVER_DEVICE_EXTENSION   deviceExtension;
  PVOID                             inputBuffer;
  ULONG                             inputLength;
  PVOID                             outputBuffer;
  ULONG                             outputLength;
  ULONG           minBufferLen;
  PVOID           sysBuffer;
  ULONG           var1, var2, var3;
  USHORT          result;


  DellFanDriverDebugPrint(DBG_IO, DBG_TRACE, __FUNCTION__"++. IRP %p", Irp);

  deviceExtension = (PDELLFANDRIVER_DEVICE_EXTENSION)DeviceObject->DeviceExtension;

  // Get our IRP stack location
  irpStack = IoGetCurrentIrpStackLocation(Irp);

  // Get the buffer lengths
  inputLength = irpStack->Parameters.DeviceIoControl.InputBufferLength;
  outputLength = irpStack->Parameters.DeviceIoControl.OutputBufferLength;

  switch (irpStack->Parameters.DeviceIoControl.IoControlCode) 
  {
  case CTL_CODE(0x8001, 0x800, METHOD_BUFFERED, FILE_ANY_ACCESS):

    //minimal buffer size requirement
    minBufferLen = 6;

    //check buffer length
    if (inputLength < minBufferLen || outputLength < minBufferLen)
    {
      DellFanDriverDebugPrint(DBG_IO, DBG_TRACE, __FUNCTION__":I/O Bufferlength %X | %X is too small\n",inputLength, outputLength);

      status = STATUS_BUFFER_TOO_SMALL;
    }
    else
    {
      //bytes transfered
      Irp->IoStatus.Information  = minBufferLen;

      //**********************************************************************************************
      //get buffer
      sysBuffer = Irp->AssociatedIrp.SystemBuffer;

      var1 = ((ULONG)((PUCHAR)sysBuffer)[0] << 8) | 0xA3;

      var2 = (ULONG)((PUCHAR)sysBuffer)[1] << 8;

      var3 = (ULONG)((PUCHAR)sysBuffer)[2];

      __asm 
      {
        pushad;

        mov  ebx, var2;
        mov ecx, var3;
        mov eax, var1;

        out  0B2h, al;Interrupt Controller #2, 8259A
          out  84h, al;

        mov result, ax;

        popad;
      }

      ((PUSHORT)sysBuffer)[4/sizeof(USHORT)] = result;
      status = STATUS_SUCCESS;
      //**********************************************************************************************
      DellFanDriverDebugPrint(DBG_IO, DBG_TRACE, __FUNCTION__"Fan control result:%4X", (ULONG)result);
    }   

    Irp->IoStatus.Status = status;
    IoCompleteRequest (Irp, IO_NO_INCREMENT);
    break;
  default:
    status = STATUS_INVALID_DEVICE_REQUEST;
    Irp->IoStatus.Status = status;
    IoCompleteRequest (Irp, IO_NO_INCREMENT);
    break;
  }

  DellFanDriverDebugPrint(DBG_IO, DBG_TRACE, __FUNCTION__"-. IRP %p STATUS %x", Irp, status);

  return status;
}
```

`4dd466d0e1e0cc123afbe504da2462ca`
