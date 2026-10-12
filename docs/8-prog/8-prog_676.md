
# WDM 驱动程序使用 Buffer I/O,Direct,Neither 模式传递数据(修正版)

> 原文：[`www.pediy.com/kssd/pediy10/58251.html`](https://www.pediy.com/kssd/pediy10/58251.html)

这次应该算是 20%个自己的 Demo 了![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!") （别说我是抄袭的），将上次那个 demo 进行了简化，删除了电源管理部分代码，改写 PnP 处理例程为默认向下传递，添加注释和对 remove lock 的解释。

驱动层：

```
// 解释一下 Remove Locks(移除锁:来自于 MSDN)

// remove lock routines 提供一种方法来跟踪设备上

// 未完成的 I / O 操作的数量和确定可以安全地分离并删除一个驱动程序的设备对象.

// 系统提供了这些例程以驱动程序编写器作为替代实现他们自己跟踪机制。

// 驱动程序可以使用此机制用于两个目的：

// 1. 以确保驱动程序 DispatchPnP 例程将无法完成一个 IRP_MN_REMOVE_DEVICE 请求当锁正在使用

//(例如,另一个驱动程序例程访问设备时)。

// 2. 计算驱动程序不应删除其设备对象原因的数目，并当计数为 0 时，设置事件

// 用于初始化删除锁，驱动程序应该分配 IO_REMOVE_LOCK 结构在设备扩展中

// 然后调用 IoInitializeRemoveLock. 驱动程序通常调用 IoInitializeRemoveLock 在其 AddDevice 例程中，

// 您的驱动程序必须调用 IoAcquireRemoveLock 每次它启动 I / O 操作。

// 驱动程序必须调用 IoReleaseRemoveLock 每次完成一个 I / O 操作。

// 驱动程序可以获得锁不止一次。 删除锁定例程维护未完成的收购的锁的计数。

// 每次调用 IoAcquireRemoveLock 增加计数，和 IoReleaseRemoveLock 递减计数。

#include <wdm.h>

#include "DrvMain.h"

UNICODE_STRING Global_sz_Drv_RegInfo;

UNICODE_STRING Global_sz_DeviceName;

NTSTATUS
  DriverEntry(
    IN PDRIVER_OBJECT  DriverObject,
    IN PUNICODE_STRING  RegistryPath
    )
{
    RtlInitUnicodeString(
        &Global_sz_Drv_RegInfo,
        RegistryPath->Buffer);
    // Initialize function pointers
    DriverObject->DriverUnload = DriverUnload;
    DriverObject->DriverExtension->AddDevice = AddDevice;
    DriverObject->MajorFunction[IRP_MJ_CREATE] = PsdoDispatchCreate;
    DriverObject->MajorFunction[IRP_MJ_CLOSE] = PsdoDispatchClose;
    DriverObject->MajorFunction[IRP_MJ_READ] =  PsdoDispatchRead;
    DriverObject->MajorFunction[IRP_MJ_WRITE] =  PsdoDispatchWrite;
    DriverObject->MajorFunction[IRP_MJ_PNP] = PsdoDispatchPnP;
    return STATUS_SUCCESS;
}

NTSTATUS
  AddDevice(
    IN PDRIVER_OBJECT  DriverObject,
    IN PDEVICE_OBJECT  PhysicalDeviceObject
    )
{
    ULONG DeviceExtensionSize;
    PDEVICE_EXTENSION p_DVCEXT;
    PDEVICE_OBJECT ptr_PDO;
    NTSTATUS status;
    ULONG IdxPwrState;
    RtlInitUnicodeString(
        &Global_sz_DeviceName,
        L"\\DosDevices\\PSDOBUFDVC");
    // Get DEVICE_EXTENSION required memory space
    DeviceExtensionSize = sizeof(DEVICE_EXTENSION);
    status = IoCreateDevice(
        DriverObject,
        DeviceExtensionSize,
        &Global_sz_DeviceName,
        FILE_DEVICE_UNKNOWN,
        FILE_DEVICE_SECURE_OPEN,
        FALSE,
        &ptr_PDO
    );
    if (NT_SUCCESS(status)) {
        ptr_PDO->Flags &= ~DO_DEVICE_INITIALIZING;
        // 注意在这里设置使用 Buffer I/O 模式
        ptr_PDO->Flags |= DO_BUFFERED_IO;
        p_DVCEXT = ptr_PDO->DeviceExtension;
        p_DVCEXT->DeviceObject = ptr_PDO;
        RtlInitUnicodeString(
            &p_DVCEXT->Device_Description,
            L"This is a Buffered I/O Driver for Pseudo Device\r\n"
            L"Created by mjtsai 2003/8/1\r\n");
        IoInitializeRemoveLock(
            &p_DVCEXT->RemoveLock,
            'KCOL',
            0,
            0
        );
        // 分配缓冲区
        p_DVCEXT->DataBuffer=ExAllocatePool(NonPagedPool, 1024);
        RtlZeroMemory(
        p_DVCEXT->DataBuffer,
        1024);
        p_DVCEXT->DeviceBufferSzie=0;
        // Initialize driver power state
        p_DVCEXT->SysPwrState = PowerSystemWorking;
        p_DVCEXT->DevPwrState = PowerDeviceD0;
        // Store next-layered device object
        // Attach device object to device stack
        p_DVCEXT->NextDeviceObject =
            IoAttachDeviceToDeviceStack(ptr_PDO, PhysicalDeviceObject);
    }
    return status;
}

VOID
  DriverUnload(
    IN PDRIVER_OBJECT  DriverObject
    )
{
    PDEVICE_EXTENSION p_DVCEXT;
    DbgPrint("In DriverUnload : Begin\r\n");
    p_DVCEXT = DriverObject->DeviceObject->DeviceExtension;
    ExFreePool(p_DVCEXT->DataBuffer);
    RtlFreeUnicodeString(
        &p_DVCEXT->Device_Description);
    IoDetachDevice(
        p_DVCEXT->DeviceObject);
    IoDeleteDevice(
        p_DVCEXT->NextDeviceObject);
    DbgPrint("In DriverUnload : End\r\n");
    return;
}

NTSTATUS
  PsdoDispatchCreate(
    IN PDEVICE_OBJECT  DeviceObject,
    IN PIRP  Irp
    )
{
    PIO_STACK_LOCATION p_IO_STK;
    PDEVICE_EXTENSION p_DVCEXT;
    NTSTATUS status;
    p_IO_STK = IoGetCurrentIrpStackLocation(Irp);
    p_DVCEXT = DeviceObject->DeviceExtension;
    status = IoAcquireRemoveLock(&p_DVCEXT->RemoveLock, p_IO_STK->FileObject);
    if (NT_SUCCESS(status)) {
        CompleteRequest(Irp, STATUS_SUCCESS, 0);
        return STATUS_SUCCESS;
    } else {
        IoReleaseRemoveLock(&p_DVCEXT->RemoveLock, p_IO_STK->FileObject);
        CompleteRequest(Irp, status, 0);
        return status;
    }
}

NTSTATUS
  PsdoDispatchClose(
    IN PDEVICE_OBJECT  DeviceObject,
    IN PIRP  Irp
    )
{
    PIO_STACK_LOCATION p_IO_STK;
    PDEVICE_EXTENSION p_DVCEXT;
    p_IO_STK = IoGetCurrentIrpStackLocation(Irp);
    p_DVCEXT = DeviceObject->DeviceExtension;
    IoReleaseRemoveLock(&p_DVCEXT->RemoveLock,
        p_IO_STK->FileObject);
    CompleteRequest(Irp, STATUS_SUCCESS, 0);
    return STATUS_SUCCESS;
}

NTSTATUS
  PsdoDispatchRead(
    IN PDEVICE_OBJECT  DeviceObject,
    IN PIRP  Irp
    )
{
    PVOID Buf;        //Buffer provided by user program
    ULONG BufLen;    //Buffer length for user provided buffer
    LONGLONG Offset;//Buffer Offset
    PVOID DataBuf;  //Buffer provided by Driver
    ULONG DataLen;  //Buffer length for Driver Data Buffer
    ULONG ByteTransferred;
    PIO_STACK_LOCATION p_IO_STK;
    PDEVICE_EXTENSION p_DVCEXT;
    DbgPrint("IRP_MJ_READ : Begin\r\n");
    // Get I/o Stack Location & Device Extension
    // 堆栈中包含用户缓冲区的信息
    p_IO_STK = IoGetCurrentIrpStackLocation(Irp);
    p_DVCEXT = DeviceObject->DeviceExtension;
    // Get User Output Buffer & Length
    // 获得用户输出缓冲区以及长度
    BufLen = p_IO_STK->Parameters.Read.Length;
    Buf = (PUCHAR)(Irp->AssociatedIrp.SystemBuffer);
    // Get Driver Data Buffer & Length
    // 获得驱动程序的缓冲区
    DataBuf = p_DVCEXT->DataBuffer;
    if (DataBuf == NULL)
        DataLen = 0;
    else
        DataLen = p_DVCEXT->DeviceBufferSzie;
    IoAcquireRemoveLock(&p_DVCEXT->RemoveLock, Irp);
    DbgPrint("Output Buffer Length : %d\r\n", BufLen);
    DbgPrint("Driver Data Length : %d\r\n", DataLen);
    //
    if (BufLen <= DataLen) {
        ByteTransferred = BufLen;
    } else {
        ByteTransferred = DataLen;
    }
    // 把数据复制到用户缓冲区
    RtlCopyMemory(
        Buf, DataBuf,
        ByteTransferred);
    IoReleaseRemoveLock(&p_DVCEXT->RemoveLock, Irp);
    // 完成对 IRP 的处理
    CompleteRequest(Irp, STATUS_SUCCESS, ByteTransferred);
    DbgPrint("IRP_MJ_READ : End\r\n");
    return STATUS_SUCCESS;
}

NTSTATUS
  PsdoDispatchWrite(
    IN PDEVICE_OBJECT  DeviceObject,
    IN PIRP  Irp
    )
{
    PVOID Buf;        //Buffer provided by user program
    ULONG BufLen;    //Buffer length for user provided buffer
    LONGLONG Offset;//Buffer Offset
    PVOID DataBuf;  //Buffer provided by Driver
    ULONG DataLen;  //Buffer length for Driver Data Buffer
    ULONG ByteTransferred;
    PIO_STACK_LOCATION p_IO_STK;
    PDEVICE_EXTENSION p_DVCEXT;
    NTSTATUS status;
    DbgPrint("IRP_MJ_WRITE : Begin\r\n");
    // Get I/o Stack Location & Device Extension
    // 获得 I/O 堆栈，设备扩展
```



`p_IO_STK = IoGetCurrentIrpStackLocation(Irp);`

`p_DVCEXT = DeviceObject->DeviceExtension;`

// Get User Input Buffer & Length  
// 获得用户输入缓冲区

`BufLen = p_IO_STK->Parameters.Write.Length;`

`Buf = (PUCHAR)(Irp->AssociatedIrp.SystemBuffer);`

// Get Driver Data Buffer & Length  
// 驱动程序缓冲区

`DataBuf = p_DVCEXT->DataBuffer;`

`DataLen = 1024;`

`IoAcquireRemoveLock(&p_DVCEXT->RemoveLock, Irp);`

`DbgPrint("Input Buffer Length : %d\r\n", BufLen);`

`DbgPrint("Driver Data Length : %d\r\n", DataLen);`

`if (BufLen <= DataLen) {`
  `ByteTransferred = BufLen;`
`} else {`
  `ByteTransferred = DataLen;`
`}`

`RtlZeroMemory(`
  `p_DVCEXT->DataBuffer,`
  `1024);`

// 储存实际传输大小

`p_DVCEXT->DeviceBufferSzie = ByteTransferred;`

`DbgPrint("Real Data Size:%d\r\n", ByteTransferred);`

// 复制

`RtlCopyMemory(`
  `DataBuf,`
  `Buf,`
  `ByteTransferred);`

`IoReleaseRemoveLock(&p_DVCEXT->RemoveLock, Irp);`

`CompleteRequest(Irp, STATUS_SUCCESS, ByteTransferred);`

`DbgPrint("IRP_MJ_WRITE : End\r\n");`

`return STATUS_SUCCESS;`

`}`

`NTSTATUS CompleteRequest(`
  `IN PIRP Irp,`
  `IN NTSTATUS status,`
  `IN ULONG_PTR info)`
`{`
  `Irp->IoStatus.Status = status;`
  `Irp->IoStatus.Information = info;`
  // 完成 I/O 操作
  `IoCompleteRequest(Irp, IO_NO_INCREMENT);`
  `return status;`
`}`

`NTSTATUS`
  `PsdoDispatchPnP(`
    `IN PDEVICE_OBJECT DeviceObject,`
    `IN PIRP Irp`
    `)`
`{`
  `PIO_STACK_LOCATION p_IO_STK;`
  `PDEVICE_EXTENSION p_DVCEXT;`
  `NTSTATUS status;`
  `DbgPrint("IRP_MJ_PNP:begin\r\n" );`
  `p_IO_STK = IoGetCurrentIrpStackLocation(Irp);`
  `p_DVCEXT = DeviceObject->DeviceExtension;`
  // 没有任何处理，直接向下传递 IRP
  `IoSkipCurrentIrpStackLocation(Irp);`
  `status = IoCallDriver(p_DVCEXT->NextDeviceObject, Irp);`
  `return status;`
`}`

### 用户层

`#include "stdafx.h"`

`#include <winioctl.h>`

`int main(int argc, char* argv[])`
`{`
  `HANDLE hdevice = CreateFile("\\\\.\\PSDOBUFDVC", GENERIC_READ | GENERIC_WRITE, 0, NULL, OPEN_EXISTING, 0, NULL);`
  `if (hdevice == INVALID_HANDLE_VALUE)`
  `{`
    `printf("Unable to open PSEUDODEVICE device - error %d\n", GetLastError());`
    `return 1;`
  `}`
  `char buffer[1024] = {'\0'};`
  `DWORD junk;`
  `sprintf(buffer, "%s", "this is the first time I write data into Driver\r\n");`
  `WriteFile(hdevice, buffer, strlen(buffer), &junk, NULL);`
  `ZeroMemory(buffer, 1024);`
  `if (ReadFile(hdevice, buffer, 1024, &junk, NULL) != 0)`
  `{`
    `buffer[junk] = 0;`
    `printf("%s", buffer);`
  `}`
  `else`
    `printf("Error %d in call to ReadFile\n", GetLastError());`
  `sprintf(buffer, "%s", "this is the second time I write data into Driver\r\n");`
  `if(!WriteFile(hdevice, buffer, strlen(buffer), &junk, NULL))`
  `{`
    `printf("Error %d in call to WriteFile\n", GetLastError());`
  `}`
  `ZeroMemory(buffer, 1024);`
  `if (ReadFile(hdevice, buffer, 1024, &junk, NULL) != 0)`
  `{`
    `buffer[junk] = 0;`
    `printf("%s", buffer);`
  `}`
  `else`
    `printf("Error %d in call to ReadFile\n", GetLastError());`
  `CloseHandle(hdevice);`
  `return 0;`
`}`

直接使用 `build` 命令即可，然后使用控制面板添加新硬件。

### Direct 方式

首先了解一下 `Irp` 结构中的 `MdlAddress` 域：

`MdlAddress(PMDL)` 域指向一个内存描述符表（MDL），该表描述了一个与该请求关联的用户模式缓冲区。如果顶级设备对象的 `Flags` 域为 `DO_DIRECT_IO`，则 I/O 管理器为 `IRP_MJ_READ` 或 `IRP_MJ_WRITE` 请求创建这个 MDL。如果一个 `IRP_MJ_DEVICE_CONTROL` 请求的控制代码指定 `METHOD_IN_DIRECT` 或 `METHOD_OUT_DIRECT` 操作方式，则 I/O 管理器为该请求使用的输出缓冲区创建一个 MDL。MDL 本身用于描述用户模式虚拟缓冲区，但它同时也含有该缓冲区锁定内存页的物理地址。为了访问用户模式缓冲区，驱动程序必须做一点额外工作。

再看下下面的图：

![](img/991698ced560dab0b800236f20eec5a6.jpg)

MDL 的结构如下：

```
typedef struct _MDL { 
  struct  _MDL  *Next; 
  CSHORT  Size; 
  CSHORT  MdlFlags; 
  struct  _EPROCESS  *Process; 
  PVOID  MappedSystemVa; 
  PVOID  StartVa; 
  ULONG  ByteCount; 
  ULONG  ByteOffset; 
} MDL, *PMDL;
```

要用到的几个宏和函数：

*   `MmGetMdlByteCount` 取缓冲区字节大小
*   `MmGetMdlByteOffset` 取缓冲区在第一个内存页中的偏移
*   `MmGetSystemAddressForMdl` 创建映射到同一内存位置的内核模式虚拟地址
*   `MmGetSystemAddressForMdlSafe` 与 `MmGetSystemAddressForMdl` 相同，但 `Windows 2000` 首选
*   `MmUnlockPages` 为该 MDL 解锁内存页
*   `MmProbeAndLockPages` 地址有效性校验后锁定内存页

简单将上面代码进行修改：

```
NTSTATUS
  AddDevice(
    IN PDRIVER_OBJECT  DriverObject,
    IN PDEVICE_OBJECT  PhysicalDeviceObject 
    )
{
     ......
     ptr_PDO->Flags |= DO_DIRECT_IO;
     .....
}
```

```
NTSTATUS
  PsdoDispatchWrite(
    IN PDEVICE_OBJECT  DeviceObject,
    IN PIRP  Irp
    )
{
    ......
    // 为 MDL 描述的缓冲区创建一个非分页的系统空间虚拟地址
    // 相当于用户的输入缓冲区
    Buf = MmGetSystemAddressForMdlSafe(
    Irp->MdlAddress, HighPagePriority);
    // 获取缓冲区长度
    BufLen = MmGetMdlByteCount(Irp->MdlAddress);
    RtlCopyMemory(
    DataBuf,
    Buf, 
    ByteTransferred);
    ......
}
```

```
NTSTATUS
  PsdoDispatchRead(
    IN PDEVICE_OBJECT  DeviceObject,
    IN PIRP  Irp
    )
{
    ......
    // 为 MDL 描述的缓冲区创建一个非分页的系统空间虚拟地址
    // 相当于用户的输出缓冲区
    Buf = MmGetSystemAddressForMdlSafe(
    Irp->MdlAddress, HighPagePriority);
    BufLen = MmGetMdlByteCount(Irp->MdlAddress);
    RtlCopyMemory(
    Buf, 
    DataBuf, 
    ByteTransferred);
    ......
}
```

### Neither 模式

如果你在设备对象中同时忽略了 `DO_DIRECT_IO` 和 `DO_BUFFERED_IO` 标志设置，你将得到默认的 neither 方式。对于这种方式，I/O 管理器将简单地把用户模式虚拟地址和字节计数交给你，其余的工作由你去做。由于驱动程序不能随意使用用户虚拟地址空间，不能单纯使用复制的方式。

一般使用 `ptr_PDO->Flags |= 0;` 进行设置。

```
NTSTATUS
  PsdoDispatchWrite(
    IN PDEVICE_OBJECT  DeviceObject,
    IN PIRP  Irp
    )
{
  PVOID Buf;    //Buffer provided by user program
  ULONG BufLen;  //Buffer length for user provided buffer
  LONGLONG Offset; //Buffer Offset
  PVOID DataBuf;  //Buffer provided by Driver
  ULONG DataLen;  //Buffer length for Driver Data Buffer
  PMDL mdl;
  PULONG pVer;
  ULONG ByteTransferred;
  PIO_STACK_LOCATION p_IO_STK;
  PDEVICE_EXTENSION p_DVCEXT;
  NTSTATUS status;

  DbgPrint("IRP_MJ_WRITE : Begin\r\n");

  //Get I/o Stack Location & Device Extension
  // 获得 I/O 堆栈，设备扩展
  p_IO_STK = IoGetCurrentIrpStackLocation(Irp);
  p_DVCEXT = DeviceObject->DeviceExtension;

  //Get User Input Buffer & Length 
  // 获得用户输入缓冲区
  BufLen = p_IO_STK->Parameters.Write.Length;
  Buf = Irp->UserBuffer;

  //Get Driver Data Buffer & Length
  // 驱动程序缓冲区
  DataBuf = p_DVCEXT->DataBuffer;
  DataLen = 1024;
  if (Irp->RequestorMode != KernelMode)
  {
    _try{
      mdl = IoAllocateMdl(Buf, BufLen, FALSE, TRUE, Irp);
      if (mdl == NULL)
      {
        DbgPrint("MDL is NULL");
      }
      // 向设备写，就要有读缓冲区的权利
      MmProbeAndLockPages(mdl, UserMode, IoReadAccess);
      }
    _except(EXCEPTION_CONTINUE_EXECUTION){
      return CompleteRequest(Irp, GetExceptionCode(), 0);
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
  // 设置实际字符串长
  p_DVCEXT->DeviceBufferSzie = ByteTransferred;
  // 数据复制
  p_DVCEXT->DeviceBufferSzie = ByteTransferred;
  RtlCopyMemory(
    DataBuf, Buf,
    ByteTransferred);
  DbgPrint(DataBuf);
  CompleteRequest(Irp, STATUS_SUCCESS, ByteTransferred);
  DbgPrint("IRP_MJ_WRITE : End\r\n");
  return STATUS_SUCCESS;
}
```



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
