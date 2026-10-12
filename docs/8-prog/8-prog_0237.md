
# 初步实现系统级拦截应用程序取硬盘物理序列号

> 原文：`www.pediy.com/kssd/pediy08/pediy8-513.htm`

【文章标题】: 初步实现系统级拦截应用程序取硬盘物理序列号  
【文章作者】: `rockhard`  
【作者邮箱】: `wnh1@sohu.com`  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
【附件下载】：点击下载

【详细过程】

以前想模拟某个程序取硬盘系列号，就将一个 `DLL` 注入进去，拦截 `DeviceIoControl` 的返回值，将其修改为目标硬盘的值。

后来看到 `REGMON` 从 `SSDT` 着手拦截注册表操作，可以看到任何程序的读写操作，就仿着改了一下程序。经测试可以欺骗相当一部分程序读硬盘序列号，包括 `ASPROTECT` 及用其加密算法的子孙:)。

我的思路是这样的：

1. 首先用程序取自己的真正的硬盘序列号，假设为 `XXXX`
2. 拦截系统的 `ZwDeviceIoControlFile`，并判断入口参数中的 `IoControlCode`，只有某几个特定的值用来取序列号的，目前在所有的程序中取硬盘序列号的，我只发现两个值,一个是 `0x7c088`，另外一个是什么忘了。如果 `IoControlCode` 为上面的值，读取系统原有的 `ZwDeviceIoControlFile` 返回 `BUFFER`，并用串匹配方法查找这个返回值中是否存在 `XXXX`，如果存在，替换为你要欺骗的值.

代码很简单：

```
UCHAR __DiskSerial[DISK_SERIAL_BUFF_LENGTH]={0};
UCHAR __ChangeTo  [DISK_SERIAL_BUFF_LENGTH]={0};

//一个简单的低率串匹配算法 ，判断一个串 S1 是不是另外一个串 S2 的子串
PUCHAR IsSubString(PUCHAR String, PUCHAR SubString ,ULONG StringLength ,ULONG SubStringLength)
{
    ULONG i,j;
    for(i=0;i<StringLength - SubStringLength +1 ;i++){
        for(j=0;j<SubStringLength;j++){
            if(String[i+j]!=SubString[j])
                break;
        }
        if(j==SubStringLength)  //match a substring
            return String+i;
    }
    return NULL;
}
```

```
//----------------------------------------------------------------------
//
//  Our own routine for ZwDeviceIocontrolFile
//  We change the hard disk serial number value requested by user
//
//----------------------------------------------------------------------
NTSTATUS HookZwDeviceIoControlFile(
                                            IN HANDLE FileHandle,
                                            IN HANDLE Event OPTIONAL,
                                            IN PIO_APC_ROUTINE ApcRoutine OPTIONAL,
                                            IN PVOID ApcContext OPTIONAL,
                                            OUT PIO_STATUS_BLOCK IoStatusBlock,
                                            IN ULONG IoControlCode,
                                            IN PVOID InputBuffer OPTIONAL,
                                            IN ULONG InputBufferLength,
                                            OUT PVOID OutputBuffer OPTIONAL,
                                            IN ULONG OutputBufferLength
                                            )
{
    NTSTATUS rc;
    rc = RealZwDeviceIoControlFile (
        FileHandle,
        Event,
        ApcRoutine,
        ApcContext,
        IoStatusBlock,
        IoControlCode,
        InputBuffer,
        InputBufferLength,
        OutputBuffer,
        OutputBufferLength
    );
    //判断 IoControlcode 是不是取序列号的值
    if((0x7c088 ==IoControlCode) && OutputBufferLength >DISK_SERIAL_BUFF_LENGTH){
        //判断返回值中是否包含当前的硬盘序列号，是的话用假的替换
        PUCHAR Locate = IsSubString(OutputBuffer,__DiskSerial,OutputBufferLength,DISK_SERIAL_BUFF_LENGTH);
        if(Locate){
            UCHAR i;
            for(i=0;i<20;i++){
                Locate[i]= __ChangeTo[i];
            }
        }
    }
    return(rc);
}
```

目前，驱动只处理了简单的几个应用层的消息，包括停止欺骗，开始欺骗，设置新的欺骗值。

```
BOOLEAN  HDHookDeviceControl( IN PFILE_OBJECT FileObject, IN BOOLEAN Wait,
                               IN PVOID InputBuffer, IN ULONG InputBufferLength,
                               OUT PVOID OutputBuffer, IN ULONG OutputBufferLength,
                               IN ULONG IoControlCode, OUT PIO_STATUS_BLOCK IoStatus,
                               IN PDEVICE_OBJECT DeviceObject ) {
    BOOLEAN           retval = FALSE;
    ULONG i;
    // Its a message from our GUI!
    IoStatus->Status      = STATUS_SUCCESS; // Assume success
    IoStatus->Information = 0;             // Assume nothing returned
    switch ( IoControlCode ) {
    //  开始欺骗
    case HDHOOK_HOOK:
        HookStart();
        break;
    // 停止欺骗
    case HDHOOK_UNHOOK:
        HookStop();
        break;
    // 告诉驱动当前自己硬盘的序列号值为多少
    case HDHOOK_SETSELFVALUE:
      if( InputBufferLength < DISK_SERIAL_BUFF_LENGTH || InputBuffer == NULL){
        IoStatus->Status = STATUS_INVALID_PARAMETER;
        break;
      }
      for(i=0; i< DISK_SERIAL_BUFF_LENGTH ;i++)
        __DiskSerial[i] = ((UCHAR *)InputBuffer)[i];
        break;
    // 设置新的欺骗的硬盘序列号
    case HDHOOK_SETEMULABLEVALUE:
      if( InputBufferLength < DISK_SERIAL_BUFF_LENGTH || InputBuffer == NULL){
        IoStatus->Status = STATUS_INVALID_PARAMETER;
        break;
      }
      for(i=0;i< DISK_SERIAL_BUFF_LENGTH ;i++)
        __ChangeTo[i] = ((UCHAR *)InputBuffer)[i];
        break;
      //返回驱动的版本号
      case HDHOOK_VERSION:
        if ( OutputBufferLength < sizeof(ULONG) ||
             OutputBuffer == NULL ) {
              IoStatus->Status = STATUS_INVALID_PARAMETER;
              break;
        }
        *(ULONG *)OutputBuffer = REGMONVERSION;
        IoStatus->Information = sizeof(ULONG);
        break;
      default:
        IoStatus->Status = STATUS_INVALID_DEVICE_REQUEST;
        break;
    }
    return TRUE;
}
```

应用层程序可以通过如下简单代码与驱动进行通信：

```
#define HDHOOK_HOOK            (ULONG) CTL_CODE( FILE_DEVICE_REGMON, 0x00, METHOD_BUFFERED, FILE_ANY_ACCESS )
#define HDHOOK_UNHOOK          (ULONG) CTL_CODE( FILE_DEVICE_REGMON, 0x01, METHOD_BUFFERED, FILE_ANY_ACCESS )
#define HDHOOK_VERSION         (ULONG) CTL_CODE( FILE_DEVICE_REGMON, 0x02, METHOD_BUFFERED, FILE_ANY_ACCESS )
#define HDHOOK_SETSELFVALUE    (ULONG) CTL_CODE( FILE_DEVICE_REGMON, 0x03, METHOD_BUFFERED, FILE_ANY_ACCESS )
#define HDHOOK_SETEMULABLEVALUE (ULONG) CTL_CODE( FILE_DEVICE_REGMON, 0x04, METHOD_BUFFERED, FILE_ANY_ACCESS )

#define DISK_SERIAL_BUFF_LENGTH      20
```

```
//设置新的序列号模拟值
DeviceIoControl(__SysHandle,HDHOOK_SETEMULABLEVALUE,szEmulSerial,DISK_SERIAL_BUFF_LENGTH, NULL, 0, &dwDummy, NULL) ;

//告诉驱动自己的硬盘序列号
DeviceIoControl(__SysHandle,HDHOOK_SETSELFVALUE,szBuffer,DISK_SERIAL_BUFF_LENGTH, NULL, 0, &dwDummy, NULL) ;

//开始拦截
DeviceIoControl(__SysHandle,HDHOOK_HOOK,NULL,0,NULL,0,&dwDummy,NULL) ;

//停止拦截
DeviceIoControl(__SysHandle,HDHOOK_UNHOOK,NULL,0,NULL,0,&dwDummy,NULL) ;
```

其中 `__SysHandle` 是安装驱动的句柄。

这篇文章的思路来自 `regmon`(regmon 版权申明我保留在文件中)，写出来是方便调试那些有正版注册号且以硬盘序列号生成注册码的程序，方便一下大家。

附件中包含我写的一个简单的 `UI`，用来与驱动通信。

如果你觉得有用，希望有人能完成如下功能：

1. 进程过滤功能，只对特定程序拦截。
2. 考虑将上面的那个串匹配算法改得高效些，数据结构教材上有现成的：）
3. 那个返回值中有硬盘厂家，磁道数等信息，完成完全模拟，还有 `IoControlCode` 的完善。

若改好了希望能传一份给我 :)

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!
2006 年 11 月 29 日 12:03:24

对于文中的某些地方做一些解释，希望能给那些想学驱动的朋友有一些帮助。

引用:
2、拦截系统的 `ZwDeviceIoControlFile`，并判断入口参数中的 `IoControlCode`，只有某几个特定的值用来取序列号的，
目前在所有的程序中取硬盘序列号的，我只发现两个值,一个是 `0x7c088`，另外一个是什么忘了。

这里所指的两个数应该是 `0x7C088` 和 `0x4D008`：

第一个数 `0x7C088` 是对应 `SMART drive(ATA)` 硬盘接口，这个数在 `NTDDK` 的 `ntdddisk.h` 中定义：

```
//
// IOCTL support for SMART drive fault prediction.
//

#define SMART_RCV_DRIVE_DATA          CTL_CODE(IOCTL_DISK_BASE, 0x0022, METHOD_BUFFERED, FILE_READ_ACCESS | FILE_WRITE_ACCESS)
```

所以原文中对应的语句改成如下形式比较容易理解：



```
代码:

***

  if((SMART_RCV_DRIVE_DATA ==IoControlCode) && OutputBufferLength >DISK_SERIAL_BUFF_LENGTH){
      PPUCHAR Locate = IsSubString(OutputBuffer,__DiskSerial,OutputBufferLength,DISK_SERIAL_BUFF_LENGTH);
      if(Locate){
          UCHAR i;
          for(i=0;i<20;i++){
              Locate[i]= __ChangeTo[i];
          }
      }
  }

***

```

在 DDK 中对 `SMART_RCV_DRIVE_DATA` 的解释如下:

```
代码:

***

SMART_RCV_DRIVE_DATA
Operation
Returns the ATA-2 identify data, the SMART thresholds, or the SMART attributes for the device. This IOCTL must be handled by drivers that support SMART.

Input
The buffer at Irp->AssociatedIrp.SystemBuffer contains a SENDCMDINPARAMS structure that describes the request being sent to the device. The irDriveRegs.bCommandReg member specifies ID_CMD when identify data is requested and SMART_CMD when SMART data is requested. If SMART data is requested, the irDriveRegs.bFeaturesReg member specifies either READ_ATTRIBUTES or READ_THRESHOLDS.

Parameters.DeviceIoControl.InputBufferLength specifies the size in bytes of the input buffer, which must be >= (sizeof(SENDCMDINPARAMS) – 1).

Parameters.DeviceIoControl.OutputBufferLength specifies the size in bytes of the output buffer, which must be >= (sizeof(SENDCMDOUTPARAMS) – 1 + 512).

Output
The driver returns the SENDCMDOUTPARAMS structure and a 512-byte buffer of drive data to the buffer at Irp->AssociatedIrp.SystemBuffer.

I/O Status Block
The driver sets the Information field to (sizeof(SENDCMDOUTPARAMS) – 1 + 512) when it sets the Status field to STATUS_SUCCESS. Otherwise, the driver sets the Information field to zero and the Status field to possibly STATUS_INVALID_PARAMETER or STATUS_INSUFFICIENT_RESOURCES.

***

```

第二个数 `0x4D008` 对应 SCSI 接口的硬盘，这个数在 NTDDK 的 `ntddscsi.h` 中定义:

```
代码:

***

//
// NtDeviceIoControlFile IoControlCode values for this device.
//
// Warning:  Remember that the low two bits of the code specify how the
//           buffers are passed to the driver!
//

#define IOCTL_SCSI_MINIPORT           CTL_CODE(IOCTL_SCSI_BASE, 0x0402, METHOD_BUFFERED, FILE_READ_ACCESS | FILE_WRITE_ACCESS)

***

```

在 NTDDK 中对 `IOCTL_SCSI_MINIPORT` 的解释如下:

```
代码:

***

IOCTL_SCSI_MINIPORT
Operation
Sends a special control function to an HBA-specific miniport driver. Results vary, depending on the particular miniport driver to which this request is forwarded. If the caller specifies a nonzero Length, either the input or output buffer must be at least (sizeof(SRB_IO_CONTROL) + DataBufferLength)).

Input
The buffer at Irp->AssociatedIrp.SystemBuffer must contain an SRB_IO_CONTROL structure. Parameters.DeviceIoControl.InputBufferLength indicates the size in bytes of the buffer, which must be at least sizeof (SRB_IO_CONTROL), with additional storage for data if the Length field is nonzero.

Output
An updated SRB_IO_CONTROL structure is returned to the buffer at Irp->AssociatedIrp.SystemBuffer.

I/O Status Block
The Information field contains the number of bytes returned in the output buffer. The Status field indicates the results of the operation.

***

```
