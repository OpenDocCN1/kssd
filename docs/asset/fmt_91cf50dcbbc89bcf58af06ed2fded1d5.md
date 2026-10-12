
# 驱动与应用程序之间的通信

> 原文：[`www.pediy.com/kssd/pediy11/124396.html`](https://www.pediy.com/kssd/pediy11/124396.html)

最近比较忙，其实也就是瞎忙![](img/962324edb48d57effae0c0571850a66b.jpg "酷")，今天抽出一点时间来给大家补补驱动的课，呵呵~~如果您已经会了，就直接跳过啦，真没什么技术含量，让大哥们笑话了![](img/45dd044a2743704cbb333715eae4a5f6.jpg "狂笑")~~但我想对于一些想进军驱动和刚踏入驱动的朋友们，如果能给大家一点小小的帮助我想就足够了！（小弟，也是刚踏入驱动的大门，有误之处，望各位大哥指出![](img/6821a6175013046370981b6035d43ab4.jpg "使眼色")）

看到论坛上有人问起驱动与应用层之间通信的问题，今天我给大家讲讲驱动与应用程序之间是怎么通信的吧~~

首先说说为什么大部分的驱动程序都需要与应用层程序之间进行通信，其实原因很简单，就是为了用应用层软件去控制电脑底层~~不过如果某些特殊功能的驱动，比如病毒中的某个驱动程序进行某种隐蔽的操作（病毒或木马之类的），也可以不用与用户通信，因为它的安装就是为了某项工作，完成就行了~~不过我想大部分的驱动都需要与应用程序进行通信吧~~个人观点！![](img/cda69129882b40377a87144c782680e1.jpg "微笑")

## 通信方法

那么怎么样才能使我们的驱动与应用程序之间进行通信呢？

目前比较流行的两种方法是：

第一种：使用 `WriteFile` 和 `ReadFile` 分别从驱动中读取和写入数据，然后用不同的 `IRP` 来传递信息。

第二种：使用 `DeviceIoControl` 通信

编程工具：`RadASM`

编程语言：`Win32` 汇编

### 使用 WriteFile 和 ReadFile 进行通信

首先用 `RadASM` 建立一个 `Driver` 工程 `Connection`

`Connection.asm` 的代码如下：

```asm
.386

.model flat,stdcall

option casemap:none

include D:\\RadASM\\masm32\\include\\w2k\\ntstatus.inc        ;根据自己的安装路径进行配置

include D:\\RadASM\\masm32\\include\\w2k\\ntddk.inc

include D:\\RadASM\\masm32\\include\\w2k\\ntoskrnl.inc

includelib D:\\RadASM\\masm32\\lib\\w2k\\ntoskrnl.lib

include D:\\RadASM\\masm32\\macros\\Strings.mac

include Connection.inc

.const

CCOUNTED\_UNICODE\_STRING "\\\\Device\\\\Connection",g\_usDeviceName,4

CCOUNTED\_UNICODE\_STRING "\\\\DosDevice\\\\Connection",g\_usSymbolicLinkName,4

.code

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

;DispatchCreateClose

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

DispatchCreateClose proc pDeviceObject:PDEVICE\_OBJECT,pIrp:PIRP

  mov eax,pIrp

  assume eax:PTR \_IRP

  mov [eax].IoStatus.Status,STATUS\_SUCCESS

  and [eax].IoStatus.Information,0

  assume eax:nothing

  fastcall IofCompleteRequest,pIrp,IO\_NO\_INCREMENT

  mov eax,STATUS\_SUCCESS

  ret

DispatchCreateClose endp

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

;DriverUnload

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

DriverUnload proc pDriverObject:PDRIVER\_OBJECT

  ;删除符号符连接

  invoke IoDeleteSymbolicLink,addr g\_usSymbolicLinkName

  mov eax,pDriverObject

  ;删除设备对象

  invoke IoDeleteDevice,(DRIVER\_OBJECT PTR\[eax\]).DeviceObject

  ret

DriverUnload endp

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

;DispatchWrite

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

DispatchWrite proc uses esi edi pDeviceObject:PDEVICE\_OBJECT,pIrp:PIRP

  LOCAL status:NTSTATUS

  mov esi,pIrp

  assume esi:PTR \_IRP

  IoGetCurrentIrpStackLocation esi

  mov edi,eax

  assume edi:PTR IO\_STACK\_LOCATION

  invoke DbgPrint,$CTA0("\[Test\]%d"),[edi].Parameters.Write.\_Length

  invoke DbgPrint,$CTA0("\[Test\]%s"),[esi].AssociatedIrp.SystemBuffer

  push status

  pop  [esi].IoStatus.Status

  push [edi].Parameters.Write.\_Length

  pop  [esi].IoStatus.Information

  assume edi:nothing

  assume esi:nothing

  fastcall IofCompleteRequest,esi,IO\_NO\_INCREMENT

  mov eax,status

  ret

DispatchWrite endp

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

;DispatchRead

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

DispatchRead proc pDeviceObject:PDEVICE\_OBJECT,pIrp:PIRP

  LOCAL status:NTSTATUS

  mov esi,pIrp

  assume esi:PTR \_IRP

  IoGetCurrentIrpStackLocation esi

  mov edi,eax

  assume edi:PTR IO\_STACK\_LOCATION

  invoke DbgPrint,$CTA0("\[Test\]%d"),[edi].Parameters.Read.\_Length

  invoke DbgPrint,$CTA0("\[Test\]%s"),[esi].AssociatedIrp.SystemBuffer

  push status

  pop  [esi].IoStatus.Status

  push [edi].Parameters.Read.\_Length

  pop  [esi].IoStatus.Information

  assume edi:nothing

  assume esi:nothing

  fastcall IofCompleteRequest,esi,IO\_NO\_INCREMENT

  mov eax,status

  ret

DispatchRead endp

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

;DriverEntry

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

DriverEntry proc pDriverObject:PDRIVER\_OBJECT,pusRegistryPath:PUNICODE\_STRING

  LOCAL status:NTSTATUS

  LOCAL pDeviceObject:PDEVICE\_OBJECT

  mov status,STATUS\_DEVICE\_CONFIGURATION\_ERROR

  ;创建设备对象

  invoke IoCreateDevice,pDriverObject,0,addr g\_usDeviceName,FILE\_DEVICE\_UNKNOWN,0,FALSE,addr pDeviceObject

  .if eax == STATUS\_SUCCESS

    ;创建符号连接

    invoke IoCreateSymbolicLink,addr g\_usSymbolicLinkName,addr g\_usDeviceName

    .if eax == STATUS\_SUCCESS

      mov eax,pDriverObject

      assume eax:PTR DRIVER\_OBJECT

      or  [eax].Flags,DO\_BUFFERED\_IO

      mov [eax].DriverUnload,offset DriverUnload

      mov [eax].MajorFunction\[IRP\_MJ\_CREATE\*(sizeof PVOID)\],offset DispatchCreateClose

      mov [eax].MajorFunction\[IRP\_MJ\_CLOSE\*(sizeof PVOID)\],offset DispatchCreateClose

      mov [eax].MajorFunction\[IRP\_MJ\_WRITE\*(sizeof PVOID)\],offset DispatchWrite

      mov [eax].MajorFunction\[IRP\_MJ\_READ\*(sizeof PVOID)\],offset DispatchRead

      assume eax:nothing

      mov status,STATUS\_SUCCESS

    .else

      invoke IoDeleteDevice,pDeviceObject

    .endif

  .endif

  mov eax,status

  ret

DriverEntry endp

end DriverEntry
```

代码很简单，应用层的例子就不用多说了吧，只需要在应用层代码创建一个文件，然后调用 `WriteFile` 和 `ReadFile` 进行文件操作，驱动层就会响应应用层的操作进行相关处理，我这里的代码很简单只是输出了缓冲区中的内容，其实可以在其中加入更加复杂的操作，请读者自行研究~~

### 使用 DeviceIoControl 进行通信

前面我们使用的方法，使得我们每建行一次操作都要分别调用 `ReadFile` 和 `WriteFile` 进行操作，很麻烦，在这里我们就用一个比较通用的方法 `DeviceIoControl` 函数。

`DeviceIoControl` 函数会使操作系统产生一个 `IRP_MJ_DEVICE_CONTROL` 类型的 `IRP`，然后这个 `IRP` 会被分发到相应的派遣例程中

先来看看 `DeviceIoControl` 函数的原型声明：

```c
BOOL DeviceIoControl(
  HANDLE  hDevice,
  DWORD   dwIoControlCode,           //进行的操作代码
  LPVOID    lpInBuffer,
  DWORD   nInBufferSize,
  LPVOID   lpOutBuffer,
  DWORD   nOutBufferSize,
  LPDWORD  lpBytesReturned,
  LPOVERLAPPED  lpOverlapped
  )
```

主要看第二个参数，它是一个 I/O 控制码，即 `IOCTL` 值，是一个 32 位的无符号整型数值。

#### 什么是 IOCTL？

IO 控制指令(IOCTLs)主要用于用户态应用程序和驱动之间的沟通或者设备栈内驱动之间的沟通，这种指令通过 `IRP` 来进行传送。对 `DeviceIoControl` 的调用会促使 I/O 管理器产生一个 `IRP_MJ_DEVICE_CONTROL` 请求并且发送给当前设备栈最顶端的驱动程序。另外，上层驱动程序可以通过产生和发送 `IRP_MJ_DEVICE_CONTROL` 或者 `IRP_MJ_INTERNAL_DEVICE_CONTROL` 请求的方式向下层驱动程序发送 IO 控制指令。驱动程序在 `DispatchDeviceControl` 和 `DispatchInternalDeviceControl` 这两个例程中处理这些请求。

首先我们用 `RadASM` 建立一个名为 `DeviceControl` 的 `Driver` 工程

`DeviceControl.inc` 的代码如下：

```asm
IOCTL_DEVICEIOCONTROL equ CTL_CODE(FILE_DEVICE_UNKNOWN,800h,METHOD_BUFFERED,FILE_READ_ACCESS + FILE_WRITE_ACCESS)
```

`DeviceControl.asm` 的代码如下：

```asm
.386

.model flat,stdcall

option casemap:none

include D:\\RadASM\\masm32\\include\\w2k\\ntstatus.inc        ;根据自己的安装路径进行配置

include D:\\RadASM\\masm32\\include\\w2k\\ntddk.inc

include D:\\RadASM\\masm32\\include\\w2k\\ntoskrnl.inc

includelib D:\\RadASM\\masm32\\lib\\w2k\\ntoskrnl.lib

include D:\\RadASM\\masm32\\macros\\Strings.mac

include DeviceControl.inc

.const

CCOUNTED\_UNICODE\_STRING "\\\\Device\\\\DeviceControl",g\_usDeviceName,4

CCOUNTED\_UNICODE\_STRING "\\\\??\\\\DeviceControl",g\_usSymbolicLinkName,4

.code

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

;DispatchCreateClose

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

DispatchCreateClose proc pDeviceObject:PDEVICE\_OBJECT,pIrp:PIRP

  mov eax,pIrp

  assume eax:PTR \_IRP

  mov [eax].IoStatus.Status,STATUS\_SUCCESS

  and [eax].IoStatus.Information,0

  assume eax:nothing
```
