
`/``/` `/``/` `Sample` `"Hello World"` `driver` `/``/` `/``/` `creates a HelloDev, that expects one IOCTL`

```c
#include <ntddk.h>
```
```c
#define HELLO_DRV_IOCTL CTL_CODE(FILE_DEVICE_UNKNOWN, 0x800, METHOD_NEITHER, FILE_ANY_ACCESS)
```
```c
//#define CTL_CODE(DeviceType, Function, Method, Access) (  ((DeviceType) << 16) | ((Access) << 14) | ((Function) << 2) | (Method))
```
`/``/``这里的CTL CODE是一个宏，是SDK里的头文件提供的。读者要做的是直接利用这个宏来生成一个自己的设备控制请求功能号。CTL_CODE有``4``个参数，其中第一个参数是设备类型。笔者生成的这种控制设备和任何硬件都没有关系，所以直接定义成未知类型(FILE_DEVICE_UNKNOWN)即可。第二个参数是生成这个功能号的核心数字，这个数字直接用来和其他参数“合成”功能号。OxO~Ox7ff已经被微软预留，所以笔者只能使用大于``0x7ff``的数字。同时，这个数字不能大于``0xfmf``。如果要定义超过一个的功能号，那么不同的功能号就靠这个数字进行区分。第三个参数METHOD_BUFFERED是说用缓冲方式``2``。用缓冲方式的话，输入``/``输出缓冲会在用户和内核之间拷贝。这是比较简单和安全的一种方式。最后一个参数是这个操作需要的权限。当笔者需要将数据发送到设备时，相当于往设备上写入数据，所以标志为拥有写数据权限（FILE_WRITE_DATA)。`
```c
#define DOS_DEV_NAME L"\\DosDevices\\HelloDev"
#define DEV_NAME L"\\Device\\HelloDev"
```

### IRP Not Implemented Handler
```c
/<summary>
/</summary>
/</summary>
/<param name="DeviceObject">The pointer to DEVICE_OBJECT</param>
/</summary>
/<param name="Irp">The pointer to IRP</param>
/</summary>
/<returns>NTSTATUS</returns>
```
```c
/*
以下是分发函数
分发函数是一组用来处理发送给设备对象（当然也包括控制设备）的请求的函数。这些数由内核驱动的开发者编写，以便处理这些请求并返回给Windows。分发函数是设置在驱动对象（Driver Object）上的。也就是说，每个驱动都有一组自己的分发函数。Windows的IO管理器在收到请求时，会根据请求发送的目标，也就是一个设备对象，来调用这个设备对象所从的驱动对象上对应的分发函数。
不同的分发函数处理不同的请求
打开（Create'):在试图访问一个设备对象之前，必须先用打开请求“打开”它。只有得到成功的返回，才可以发送其他的请求。
关闭（Close):在结束访问一个设备对象之后，发送关闭请求将它关闭。关闭之后，就必须再次打开才能访问。
设备控制（Device Control):设备控制请求是一种既可以用来输入（从应用到内核),又可以用来输出（从内核到应用）的请求。
NTSTATUS cwkDispatch(IN_PDEVICE_OB3ECT dev,IN PIRp irp)
其中的dev就是请求要发送给的目标对象;irp则是代表请求内容的数据结构的指针。无论如何，分发函数必须首先设置给驱动对象，这个工作一般在 DriverEntry中完成。
*/
```
```c
NTSTATUS IrpNotImplementedHandler(IN PDEVICE_OBJECT DeviceObject, IN PIRP Irp) {
Irp->IoStatus.Information = 0;
Irp->IoStatus.Status = STATUS_NOT_SUPPORTED;

UNREFERENCED_PARAMETER(DeviceObject);
PAGED_CODE();

//
Complete the request
IoCompleteRequest(Irp, IO_NO_INCREMENT);

return STATUS_NOT_SUPPORTED;
}
```

### IRP Create Close Handler
```c
/<summary>
/</summary>
IRP Create Close Handler
/</summary>
/<param name="DeviceObject">The pointer to DEVICE_OBJECT</param>
/</summary>
/<param name="Irp">The pointer to IRP</param>
/</summary>
/<returns>NTSTATUS</returns>
```
```c
NTSTATUS IrpCreateCloseHandler(IN PDEVICE_OBJECT DeviceObject, IN PIRP Irp) {
Irp->IoStatus.Information = 0;
Irp->IoStatus.Status = STATUS_SUCCESS;

UNREFERENCED_PARAMETER(DeviceObject);
PAGED_CODE();

//
Complete the request
IoCompleteRequest(Irp, IO_NO_INCREMENT);

return STATUS_SUCCESS;
}
```

### IRP Unload Handler
```c
/<summary>
/</summary>
IRP Unload Handler
/</summary>
/<param name="DeviceObject">The pointer to DEVICE_OBJECT</param>
/</summary>
/<returns>NTSTATUS</returns>
```
```c
VOID IrpUnloadHandler(IN PDRIVER_OBJECT DriverObject) {
UNICODE_STRING DosDeviceName = {
0 };
PAGED_CODE();
RtlInitUnicodeString(&DosDeviceName, DOS_DEV_NAME);

//
Delete the symbolic link
IoDeleteSymbolicLink(&DosDeviceName);

//
Delete the device
IoDeleteDevice(DriverObject->DeviceObject);

DbgPrint("[!] Hello Driver Unloaded\n");
}
```

### IRP Device IoCtl Handler
```c
/<summary>
/</summary>
IRP Device IoCtl Handler
/</summary>
/<param name="DeviceObject">The pointer to DEVICE_OBJECT</param>
/</summary>
/<param name="Irp">The pointer to IRP</param>
/</summary>
/<returns>NTSTATUS</returns>
```
```c
NTSTATUS IrpDeviceIoCtlHandler(IN PDEVICE_OBJECT DeviceObject, IN PIRP Irp) {
ULONG IoControlCode = 0;
PIO_STACK_LOCATION IrpSp = NULL;
NTSTATUS Status = STATUS_NOT_SUPPORTED;

UNREFERENCED_PARAMETER(DeviceObject);
PAGED_CODE();

IrpSp = IoGetCurrentIrpStackLocation(Irp);
IoControlCode = IrpSp->Parameters.DeviceIoControl.IoControlCode;

//
根据不同的控制码作出不同的处理
if (IrpSp) {
switch (IoControlCode) {
case HELLO_DRV_IOCTL:
DbgPrint("[< HelloDriver >] Hello from the Driver!\n");
break;
default:
DbgPrint("[-] Invalid IOCTL Code: 0x%X\n", IoControlCode);
Status = STATUS_INVALID_DEVICE_REQUEST;
break;
}
}

Irp->IoStatus.Status = Status;/ /3环的GetLastError得到的就是这个值
Irp->IoStatus.Information = 0;/ /返回数据的字节数 没有写0

/
Complete the request
IoCompleteRequest(Irp, IO_NO_INCREMENT);/ /用于结束这个请求

return Status;
}
```
以上都是分发函数

### 驱动入口函数
```c
/
驱动入口函数
NTSTATUS DriverEntry(IN PDRIVER_OBJECT DriverObject, IN PUNICODE_STRING RegistryPath) {
UINT32 i = 0;
PDEVICE_OBJECT DeviceObject = NULL;
NTSTATUS Status = STATUS_UNSUCCESSFUL;
/ /绝大多数内核函数都会有一个返回值，类型为NTSTATUS。该类型本质就是一个LONG
UNICODE_STRING DeviceName, DosDeviceName = {
0 };

UNREFERENCED_PARAMETER(RegistryPath);
PAGED_CODE();

RtlInitUnicodeString(&DeviceName, DEV_NAME);/ /初始化unicode字符串，为其赋值。不会申请内存。
RtlInitUnicodeString(&DosDeviceName, DOS_DEV_NAME);

DbgPrint("[*] In DriverEntry\n");/ /驱动的打印函数

/
1.创建设备，指定设备的名字
Status = IoCreateDevice(DriverObject,
0,
&DeviceName,
FILE_DEVICE_UNKNOWN,
FILE_DEVICE_SECURE_OPEN,
FALSE,
&DeviceObject);
```
```c
/*
NTSTATUS
IoCreateDevice(
IN PDRIVER_OBJECT DriverObject,
IN ULONG DeviceExtensionsize,设备扩展的大小
IN PUNICODE_STRING DeviceName OPTIONAL,设备名，以便应用程序打开
IN DEVICE_TYPE DeviceType, 设备类型
IN ULONG DeviceCharacteristics, 设备属性
IN BOOLEAN Exclusive, 表示是否是一个独占设备
OUT PDEVICE_OBJECT *DeviceObject); 用来返回结果，如果函数执行成功（返回值为STATUS_SUCCESS），
*DeviceObject就是生成的设备对象对的指针
生成设备对象DriverObject，设备对象可以在内核中暴露出来给应用层，应用层可以像操作文件一样操作它。这
*/
```

```c
if (!NT_SUCCESS(Status)) {
if (DeviceObject) {
/
Delete the device
IoDeleteDevice(DeviceObject);
}

DbgPrint("[-] Error Initializing HelloDriver\n");
return Status;
}
```
```c
/*
分发函数必须首先设置给驱动对象
注意，下面的片段中将所有的分发函数（实际上MajorFunction是一个函数指针数组，保存所有分发函数的指针>都设置成同一个函数，这是一种简单的处理方案。
*/
```
```c
/
Assign the IRP handlers
for (i = 0; i <= IRP_MJ_MAXIMUM_FUNCTION; i++) {
/
Disable the Compiler Warning:
28169
#pragma warning(push)
#pragma warning(disable : 28169)
DriverObject->MajorFunction[i] = IrpNotImplementedHandler;
#pragma warning(pop)
}
```
```c
/
5.请求的处理

/
Assign the IRP handlers for Create, Close and Device Control
```
```c
/*
在用户层，我们每次调用CreateFile、OpenFIle、DeleteFile、CloseHandle等API时，都会向0环发送一个消息，这个消息成为IRP数据包。这些API称为设备操作API。如：当调用CreateFile时，会向内核层发送一个名为IRP_MJ_CREATE的打开设备的IRP消息。其他常用IRP类型如下：
CreateFile -> IRP_MJ_CREATE
ReadFile -> IRP_MJ_READ
WriteFile -> IRP_MJ_WRITE
CloseHandle -> IRP_MJ_CLOSE
DeviceControl -> IRP_MJ_DEVICE_CONTROL
/ /此API比上面的API更加灵活方便，因此内核编程中常使用该API进行消息的传递
*/
```

```c
DriverObject->MajorFunction[IRP_MJ_CREATE] = IrpCreateCloseHandler;
DriverObject->MajorFunction[IRP_MJ_CLOSE] = IrpCreateCloseHandler;
DriverObject->MajorFunction[IRP_MJ_DEVICE_CONTROL] = IrpDeviceIoCtlHandler;

//
Assign the driver Unload routine
DriverObject->DriverUnload = IrpUnloadHandler;/ /为驱动指定卸载函数

/
Set the flags
```
```c
/*
设置数据交互方式
pDeviceObj->Flags
/ /缓冲区方式读写（DO_BUFFERED_IO）：将3环缓冲区内的数据复制一份到0环的缓冲区。 方便，但性能不好。
/ /直接方式读写（DO_DIRECT_IO）:首先将3环缓冲区锁住，然后在将对应的物理地址映射一份0环的线性地址。适合大量数据传输。两个线性地址对应同一个物理地址。
/ /其它方式读写（不设置值）：0环直接读取3环的线性地址，不建议。当进程切换,CR3改变，会读取到其他进程的内存数据。
pDeviceObj->Flags &= DO_DEVICE_INITIALIZING;
/ /将DO_DEVICE_INITIALIZING初始化标志位清空，如果不清空这个位，那么3环可能无法打开设备。
*/
```
```c
DeviceObject->Flags |= DO_DIRECT_IO;
DeviceObject->Flags &= ~DO_DEVICE_INITIALIZING;

//
2.创建符号链接
/ /控制设备需要名字，暴露出来，供其他程序打开与之通信
```
```c
/*
设备的名字可以在调用IoCreateDevice或IoCreateDeviceSecure时指定。此外，应用层是无法直接通过设备的名字来打开对象的，为此必须要建立一个暴露给应用层的符号链接。符号链接就是记录一个字符串对应到另一个字符串的一种简单结构。生成符号链接的函数是:
NTSTATUS
IoCreatesymbolicLink(
IN PUNICODE_STRING symbolicLinkName, 符号链接，名
IN PUNICODE_STRING DeviceName 设备名
);
*/
```
```c
/
IoCreateSymbolicLink
3.控制设备的删除，依次删除符号链接和控制设备
Status = IoCreateSymbolicLink(&DosDeviceName, &DeviceName);

//
Show the banner
DbgPrint("[!] HelloDriver Loaded\n");

return Status;
}
```
