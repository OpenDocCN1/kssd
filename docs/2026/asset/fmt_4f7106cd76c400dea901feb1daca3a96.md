
# [原创]使用TDI与WinSock进行客户端服务器编程

题目：使用TDI与WinSock进行客户端服务器编程

日期：2010年12月24日

作者：Cryin' ([284K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2A6i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6B7N6i4y4@1k6h3q4J5](https://bbs.kanxue.com/elink@166K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2A6i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6B7N6i4y4@1k6h3q4J5))

## 简介：

在本文中，您将了解到使用传输驱动程序接口TDI与应用层套接字WinSock客户端服务器应用程序内核级编程实现细节。介绍常用的TDI函数并提供编写TDI与WinSock(TCP)应用程序的详细说明。最后还提供了源代码以演示编写程序的步骤。

## 开始之前：

在开始学习本文内容之前，假设读者以具备以下基础：
• Windows 内核级编程的知识和经验
• 网络编程概念及应用层Socket编程经验
• 了解并安装有Microsoft DDK

## WinSock（服务器）：

应用层套接字编程流程比较简单，这里简单介绍：
• 加载套接字库并创建Socket(`socket()`)
• 绑定套接字到指定端口和IP(`bind()`)
• 设置套接字进行监听以等待连接请求(`listen()`)
• 接受连接请求(`accept()`)
• 发送或接收信息(`send()/recv()`)
• 关闭套接字(`closesocket()`)

## TDI（客户端）：

TDI 定义在传输协议栈中较高级别公开的内核模式网络接口，如下图：

![](https://bbs.kanxue.com/upload/attach/201012/371024_mtg1gzj45zdwskk.png)

图：传输协议栈

TDI应用程序可以分为两种类型，与基于Socket的应用程序类似：
• 面向连接(使用TCP协议，通信可靠)
• 面向误连接(使用UDP协议，通信不可靠)

## 常用TDI函数：

这里讨论一些比较常用的TDI,以便实现基于传输驱动程序接口TDI与WinSock客户端服务器编程。

### 打开TDI设备：

```
VOID  InitializeObjectAttributes(
    OUT POBJECT_ATTRIBUTES  InitializedAttributes,
    IN PUNICODE_STRING  ObjectName,
    IN ULONG  Attributes,
    IN HANDLE  RootDirectory,
    IN PSECURITY_DESCRIPTOR  SecurityDescriptor
        );
```
所附带的参数：

`InitializedAttributes`
要初始化的`OBJECT_ATTRIBUTES`结构的指针

`ObjectName`
Unicode 设备名称，对于无连接的情况为 `device\UDP`，对于本文是面向连接的通信，则为 `device\TCP`。此 `ObjectName`必须为 Unicode 字符串。该 Unicode 字符串可以使用 `RtlInitUnicodeString` 库调用进行初始化。

`Attributes`
只需填写为`OBJ_CASE_INSENSITIVE | OBJ_KERNEL_HANDLE`即可

`RootDirectory`
此处为NULL

`SecurityDescriptor`
设置安全描述符。由于笔者总是打开内核句柄，所以很少设置这个参数

接下来需要打开本地传输地址的文件对象，客户端应用程序必须调用`ZwCreateFile`函数并在扩展属性中传递此地址。当`ZwCreateFile`成功调用时，将返回传输文件(表示连接端点)的句柄。客户端必须调用`ObReferenceObjectByHandle`函数，该函数将返回相应的传输文件对象。该传输文件对象将用于向远程服务端发送数据或接收数据。此外客户端必须事件处理例程以接收各种网络事件，如接收数据、断开连接、错误等。

```
NTSTATUS ObReferenceObjectByHandle(
    IN HANDLE  Handle,
    IN ACCESS_MASK  DesiredAccess,
    IN POBJECT_TYPE  ObjectType  OPTIONAL,
    IN KPROCESSOR_MODE  AccessMode,
    OUT PVOID  *Object,
    OUT POBJECT_HANDLE_INFORMATION  HandleInformation  OPTIONAL
        );
```
所附带的参数：

`Handle`
为对象指定一个打开句柄。此句柄由 `ZwCreateFile` 返回。

`DesiredAccess`
指定所请求的对象访问权限类型。此字段的解释取决于对象类型。

`ObjectType`
指向对象类型的指针。可以为 `*IoFileObjectType` 或 `*ExEventObjectType`。如果 `AccessMode` 为 `KernelMode`，此参数可以为 NULL。

`AccessMode`
指定要为访问检查使用的访问模式。必须为 `UserMode` 或 `KernelMode`。对于底层驱动程序，应指定 `KernelMode`。

`Object`
指向接收对象指针的变量。

`HandleInformation`
驱动程序会将此参数设置为 NULL。

```
VOID KeInitializeEvent(
    IN PRKEVENT  Event,
    IN EVENT_TYPE  Type,
    IN BOOLEAN  State
        );
```
所附带的参数：

`Event`
指向事件对象地址的指针

`Type`
事件类型，`NotificationEvent` 或者`SynchronizationEvent`

`State`
事件初始化信号状态

`IoSetCompletionRoutine`函数将向基础驱动程序注册回调函数，以在链中底层驱动程序完成了对 IRP 的请求时进行调用。将在 IRP 完成（成功或失败）和取消 IRP 请求时调用回调例程。

```
VOID IoSetCompletionRoutine(
    IN PIRP  Irp,
    IN PIO_COMPLETION_ROUTINE  CompletionRoutine,
    IN PVOID  Context,
    IN BOOLEAN    InvokeOnSuccess,
    IN BOOLEAN  InvokeOnError,
    IN BOOLEAN  InvokeOnCancel
        );
```
所附带的参数：

`Irp`
指向驱动程序希望跟踪的 IRP 的指针

`CompletionRoutine`
指定驱动程序提供的 IoCompletion 例程（更低层的驱动程序完成数据包时调用）的入口点

`Context`
指向驱动程序确定的上下文的指针，以将该上下文传递给 IoCompletion 例程

`InvokeOnSuccess`
指定如果 IRP 完成，且 IRP 的 `IO_STATUS_BLOCK` 结构中包含成功状态值，是否基于 `NT_SUCCESS` 宏（在 `ntdef.h` 中定义）的结果调用完成例程

`InvokeOnError`
指定如果 IRP 完成，且 IRP 的 `IO_STATUS_BLOCK` 结构中非成功状态值，是否调用完成例程

`InvokeOnCancel`
指定如果驱动程序或内核调用 `IoCancelIrp` 来取消 IRP，是否调用完成例程

`IoAllocateMdl` 函数用于分配 MDL 结构，该结构足够对提供的缓冲区进行映射。驱动程序间传输的数据均采用 MDL 结构的形式。驱动程序应使用此函数分配的 MDL 调用 `MmBuildMdlForNonPagedPool` 以从非分页池建立 MDL。此外，还可以选择将此函数与具有 IRP 的 MDL 关联（如果传递了此 IRP）。

```
PMDL IoAllocateMdl(
    IN PVOID  VirtualAddress,
    IN ULONG  Length,
    IN BOOLEAN  SecondaryBuffer,
    IN BOOLEAN  ChargeQuota,
    IN OUT PIRP  Irp  OPTIONAL );
```
所附带的参数：

`VirtualAddress`
指向 MDL 要描述的缓冲区的虚拟基地址的指针

`Length`
指定 MDL 要描述的缓冲区的长度（以字节为单位）。此值必须小于以下公式的计算值： `PAGE_SIZE * (65535 - sizeof` `(MDL)) / sizeof(ULONG_PTR)`

`SecondaryBuffer`
指示缓冲区是主缓冲区还是次缓冲区。确定 MDL 如何链接到 IRP。IRP 中除 MDL 描述的第一个缓冲区外的所有缓冲区均被视为次缓冲区。如果没有与 MDL 关联的 IRP，此字段必须为 FALSE

`ChargeQuota`
应由中间驱动程序设置为 FALSE。只有特定的高层驱动程序才能将此值设置为 TRUE，应在发出 I/O 请求以分配另一个 IRP 线程的上下文中调用此类驱动程序。

`Irp` 指向与 MDL 关联的 IRP 的指针。如果 Irp 指针非空，所分配的 MDL 将根据 `SecondaryBuffer` 的值与指定的 IRP 的 MDL 列表关联

`IoCallDriver` 会在进行调用时将经过格式化的 IRP 请求传递给基础驱动程序（按照在设备对象指定的方式）。在调用此函数前，调用驱动程序必须为目标驱动程序设置 I/O 堆栈位置。（有关更多信息，请参阅 DDK 帮助中关于将 IRP 传递到驱动程序栈的内容。）调用方必须检查返回值，以确保已由较低层的驱动程序完成了 IRP 请求。如果返回值不是 SUCCESS，则调用方在继续下一个任务之前，必须等待相应的事件。

```
NTSTATUS IoCallDriver(
    IN PDEVICE_OBJECT  DeviceObject,
    IN OUT PIRP  Irp );
```
所附带的参数：

`DeviceObject`
指向设备对象的指针，该对象表示所请求的 I/O 操作的目标设备

`Irp`
指向 IRP 的指针

返回值
`IoCallDriver` 返回较低层驱动程序在给定请求的 IO 状态块中设置的 `NTSTATUS` 值；或者，如果请求在排队进行进一步处理，则返回 `STATUS_PENDING`

```
PIRP TdiBuildInternalDeviceControlIrp(
    IN CCHAR  IrpSubFunction,
    IN PDEVICE_OBJECT  DeviceObject,
    IN PFILE_OBJECT  FileObject,
    IN PKEVENT  Event,
    IN PIO_STATUS_BLOCK  IoStatusBlock
    );
```
所附带的参数：

`IrpSubFunction`
返回与制定参数对应的后面要处理的IRP，参数可以为以下值：
`TDI_ASSOCIATE_ADDRESS`
The caller will pass the returned IRP to `TdiBuildAssociateAddress`.
`TDI_DISASSOCIATE_ADDRESS`
The caller will pass the returned IRP to `TdiBuildDisassociateAddress`.
`TDI_CONNECT`
The caller will pass the returned IRP to `TdiBuildConnect`.
`TDI_LISTEN`
The caller will pass the returned IRP to `TdiBuildListen`.
`TDI_ACCEPT`
The caller will pass the returned IRP to `TdiBuildAccept`.
`TDI_DISCONNECT`
The caller will pass the returned IRP to `TdiBuildDisconnect`.
`TDI_SEND`
The caller will pass the returned IRP to `TdiBuildSend`.
`TDI_RECEIVE`
The caller will pass the returned IRP to `TdiBuildReceive`.
`TDI_SEND_DATAGRAM`
The caller will pass the returned IRP to `TdiBuildSendDatagram`.
`TDI_RECEIVE_DATAGRAM`
The caller will pass the returned IRP to `TdiBuildReceiveDatagram`.
`TDI_SET_EVENT_HANDLER`
The caller will pass the returned IRP to `TdiBuildSetEventHandler`.
`TDI_QUERY_INFORMATION`
The caller will pass the returned IRP to `TdiBuildQueryInformation`.
`TDI_SET_INFORMATION`
The caller will pass the returned IRP to `TdiBuildSetInformation`.
`TDI_ACTION`
The caller will pass the returned IRP to `TdiBuildAction`.

`DeviceObject`
设备对象

`FileObject`
由`ObReferenceObjectByHandle`指定的`FileObject`

`Event`
事件指针

`IoStatusBlock`
指向`IO_STATUS_BLOCK`对象的指针

编写TDI客户端的还需要用到的函数有：
• 建立连接`TdiBuildConnect`
• 发送信息`TdiBuildSend`
• 接收信息`TdiBuildReceive`
• 关闭连接`TdiBuildDisconnect`

有关更多TDI的函数，请参考MSDN。实例截图：

![](https://bbs.kanxue.com/upload/attach/201012/371024_513aj82frwbltrg.png)

图：使用TDI与WinSock进行客户端服务器编程效果图

## 实例代码：

`Winsock`服务端编译环境：`Microsoft Visual C++ 6.0`

`TDI`客户端编译环境：`WDK 7600.16385.0`

测试环境：`Windows XP SP3`

源代码下载：[c3fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4G2k6r3g2Q4x3X3g2Y4L8$3!0Y4L8r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6H3i4K6u0r3N6r3c8A6N6$3W2F1M7$3!0U0K9$3y4G2L8h3#2#2L8X3W2U0j5i4c8A6L8$3&6Q4x3V1j5`.](https://bbs.kanxue.com/elink@c5cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4G2k6r3g2Q4x3X3g2Y4L8$3!0Y4L8r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6H3i4K6u0r3N6r3c8A6N6$3W2F1M7$3!0U0K9$3y4G2L8h3#2#2L8X3W2U0j5i4c8A6L8$3&6Q4x3V1j5`.)

## 参考文献：
