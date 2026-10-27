
# 通过驱动接触win内核

## windows I/O 模型 主要概念

整个结构会很复杂，比如处理`IRP`部分：

![2girpeg](upload/attach/202212/856851\_EEFBYH6CGMS4S26.png)

模型中我们最需要关注这三个概念：

![pimpmypid\_winkernel\_device\_objects\_model](upload/attach/202212/856851\_FF8EV6WCXSSF8W4.png)

1. I/O 请求使用 `IRP` 从用户空间发送到驱动程序

2. I/O 管理器 `I/O manager` 为所有内核模式驱动程序提供一致的接口。

3. 此 I/O 管理器为每个已安装和加载的驱动程序创建一个驱动程序对象 `driver object`。

`DRIVER_OBJECT` 包含许多驱动程序标准例程的入口点的存储。同样重要的是要注意这一点：当 I/O 管理器处理 `IRP` 时，它会将当前驱动程序的 `DRIVER_OBJECT` 内存地址提供给名为 `DriverEntry` 的主函数。更详细的描述参考链接1。

## 内核中需要了解的驱动结构和组件

了解一些重要的结构能帮助我们快速构建需要的 `payload`。

### IRP structure

它是 `Input/Output Request Packet` 的简称，在 `WDM.H` 定义了标准的NT结构。如下：

```
typedef struct _IRP {
CSHORT Type;
USHORT Size;
PMDL MdlAddress;
ULONG Flags;
union {
struct _IRP *MasterIrp;
__volatile LONG IrpCount;
PVOID SystemBuffer;
} AssociatedIrp;
LIST_ENTRY ThreadListEntry;
IO_STATUS_BLOCK IoStatus;
KPROCESSOR_MODE RequestorMode;
BOOLEAN PendingReturned;
CHAR StackCount;
CHAR CurrentLocation;
BOOLEAN Cancel;
KIRQL CancelIrql;
CCHAR ApcEnvironment;
UCHAR AllocationFlags;
union {
PIO_STATUS_BLOCK UserIosb;
PVOID IoRingContext;
};
PKEVENT UserEvent;
union {
struct {
union {
PIO_APC_ROUTINE UserApcRoutine;
PVOID IssuingProcess;
};
union {
PVOID UserApcContext;
#if ...
_IORING_OBJECT *IoRing;
#else
struct _IORING_OBJECT *IoRing;
#endif
};
} AsynchronousParameters;
LARGE_INTEGER AllocationSize;
} Overlay;
__volatile PDRIVER_CANCEL CancelRoutine;
PVOID UserBuffer;
union {
struct {
union {
KDEVICE_QUEUE_ENTRY DeviceQueueEntry;
struct {
PVOID DriverContext[4];
};
};
PETHREAD Thread;
PCHAR AuxiliaryBuffer;
struct {
LIST_ENTRY ListEntry;
union {
struct _IO_STACK_LOCATION *CurrentStackLocation;
ULONG PacketType;
};
};
PFILE_OBJECT OriginalFileObject;
} Overlay;
KAPC Apc;
PVOID CompletionKey;
} Tail;
} IRP;
```

我们重点关注两个元素：

1. `UserBuffer` 字段 ：这是从驱动程序返回的数据将被传输到客户端的地方

2. `CurrentStackLocation` : 这是驱动程序将访问从客户端发送的数据的位置。

3.

![upload/attach/202212/856851\_J28VGJC56KYXJNJ.png](upload/attach/202212/856851\_J28VGJC56KYXJNJ.png)

### IO_STACK_LOCATION 结构

该结构体有上面提到的 `CurrentStackLocation` 指定，其结构可以从这查看`<mark class="encrypted">344K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8X3g2F1i4K6u0V1N6i4y4Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0V1K9r3q4J5k6s2N6S2M7X3g2Q4x3V1k6V1M7X3W2$3k6i4u0K6i4K6u0r3k6r3c8A6i4K6u0r3N6$3c8E0i4K6u0r3L8Y4y4Q4x3X3c8%4k6r3#2Q4x3X3c8Q4y4h3k6A6L8#2)9#2k6Y4y4@1j5h3y4C8i4K6g2X3L8r3!0U0j5i4c8A6L8$3&6Q4y4U0m8Q4c8e0y4Q4z5o6m8Q4z5o6t1`.</mark>

我们需要关注的两个结构体是：

1. `MajorFunction` : 它是要执行的 I/O 操作的类型。在后面的示例中，将使用 `DeviceIoControl` API，它通过调用 `IRP_MJ_DEVICE_CONTROL` 主要函数来访问驱动程序。

2. `Parameters` : 实际就是一堆 win api，用来检索用户输入或者数据输入的所在位置。

![upload/attach/202212/856851\_AYW4X24PSPC8HGK.png](upload/attach/202212/856851\_AYW4X24PSPC8HGK.png)

### DEVICE_OBJECT 结构

该结构体位于 [IO_STACK_LOCATION结构](#io\_stack\_location-结构) 中，标识驱动程序处理 I/O 请求的逻辑、虚拟或物理设备。

1. `DeviceType` :标识设备类型。生成 IOCTL 控制代码时，此信息非常重要

2. `DriverObject`: 指向 `DRIVER_OBJECT` 的指针，该指针表示输入到 `DriverEntry` 例程的驱动程序的加载图像。此成员由 I/O 管理器在成功调用 `IoCreateDevice` API 时设置。

![upload/attach/202212/856851\_8NUZZPXUM47XJ6Y.png](upload/attach/202212/856851\_8NUZZPXUM47XJ6Y.png)

### DRIVER_OBJECT

把上面的结构体组好到一起，就是 `DRIVER_OBJECT`

```
typedef struct _DRIVER_OBJECT {
CSHORT Type;
CSHORT Size;
PDEVICE_OBJECT DeviceObject;
ULONG Flags;
PVOID DriverStart;
ULONG DriverSize;
PVOID DriverSection;
PDRIVER_EXTENSION DriverExtension;
UNICODE_STRING DriverName;
PUNICODE_STRING HardwareDatabase;
PFAST_IO_DISPATCH FastIoDispatch;
PDRIVER_INITIALIZE DriverInit;
PDRIVER_STARTIO DriverStartIo;
PDRIVER_UNLOAD DriverUnload;
PDRIVER_DISPATCH MajorFunction[IRP_MJ_MAXIMUM_FUNCTION + 1];
} DRIVER_OBJECT, *PDRIVER_OBJECT;
```

1. `DeviceObject` : 由 `IoCreateDevice` 创建

2. `DriverStart` :包含内核中的驱动程序内存位置

3. `DriverName` : 字面意思

4. `MajorFunction` : 调度表，由驱动程序的 `DispatchXxx` 例程的入口点数组组成

每个驱动程序都包含主要功能代码 `major function codes`，这些代码告诉驱动程序应执行哪些操作来满足 I/O 请求。所有驱动程序必须至少支持：

* `IRP_MJ_CREATE`
* `IRP_MJ_CLOSE`
* `IRP_MJ_DEVICE_CONTROL`

![upload/attach/202212/856851\_FNY8ZAF9VPJQJ8J.png](upload/attach/202212/856851\_FNY8ZAF9VPJQJ8J.png)

## 实例

下面将给一个实际的例子，直接上demo。

核心功能将被放置在 `vikingdrv2DeviceControl` 例程中，该例程有两个参数：

* `DEVICE_OBJECT` 指针
* `IRP` 指针

首先把 `IO_STACK_LOCATION` 指针存储到栈上，该结构用来接受用户发来的信息：

1. 我们首先验证IOCTL编号

2. 如果IOCTL是已知值，我们继续

3. 最终我们将有效负载存储在数据变量中

大致结构如下：

```
NTSTATUS vikingdrv2DeviceControl(PDEVICE_OBJECT, PIRP Irp) {
auto stack = IoGetCurrentIrpStackLocation(Irp);
switch (stack->Parameters.DeviceIoControl.IoControlCode) {
case IOCTL_number1: {
auto data = (ThreadData*)stack->Parameters.DeviceIoControl.Type3InputBuffer;
}
...
}
}
```

然后我们定义名为 `DriverEntry` 的主函数。如前所述，此函数获取指向 `DRIVER_OBJECT` 结构的指针。为了知道在提供 `IRP_MJ` 代码时必须执行哪个函数，驱动程序必须定义调度例程 `dispatch routine`。这里最重要的是使其能够处理 `IRP_MJ_DEVICE_CONTROL` 消息：让它指向 `vikingdrv2DeviceControl` 函数。

```
NTSTATUS DriverEntry(_In_ PDRIVER_OBJECT DriverObject, _In_ PUNICODE_STRING RegistryPath) {
//调度例程 dispatch routine
DriverObject->MajorFunction[IRP_MJ_CREATE] = vikingdrv2CreateClose;
DriverObject->MajorFunction[IRP_MJ_CLOSE] = vikingdrv2CreateClose;
DriverObject->MajorFunction[IRP_MJ_DEVICE_CONTROL] = vikingdrv2DeviceControl; # 这
}
```

这就是一个简单基础的驱动实例~，确实很基础，但是能用。

## 写一个driver

参考自 `<mark class="encrypted">9fbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4F1M7s2g2T1i4K6u0W2j5$3!0E0i4K6u0r3N6$3W2F1k6r3!0%4M7$3E0W2M7X3&6W2L8s2m8J5L8$3N6J5j5h3#2E0K9h3&6Y4i4K6j5H3i4K6u0o6i4@1f1#2i4K6R3$3i4K6V1&6i4@1f1@1i4@1t1^5i4K6R3H3i4@1f1@1i4@1t1^5i4@1q4m8i4@1f1%4i4@1q4q4i4K6R3H3i4@1f1#2i4K6S2p5i4K6V1#2i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1&6i4@1p5&6i4@1t1I4i4@1f1#2i4K6S2m8i4@1p5^5i4@1f1%4i4@1p5^5i4K6S2n7i4@1f1#2i4@1u0m8i4K6S2r3i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1&6i4K6W2o6i4K6R3H3i4@1f1^5i4@1p5$3i4K6R3I4i4@1f1#2i4@1q4q4i4K6S2o6i4@1f1$3i4K6R3^5i4K6V1H3i4@1f1@1i4@1t1^5i4K6R3&6i4@1f1@1i4@1t1^5i4@1q4m8i4@1f1#2i4K6S2m8i4K6W2r3i4@1f1^5i4K6R3K6i4@1u0p5i4@1g2r3i4@1u0o6i4K6W2m8</mark>

1. 等待 IRP

2. 当 IRP有一个完整且正确的 IOCTL 结构的时候，展示windows版本信息。

3. 更改给定线程的优先级 `<=` 用户进程

### 内核方面

#### 第一步，准备驱动和外界交互部分

驱动程序客户端和驱动程序本身必须具有“通用”说话方式。 `DeviceIoControl` 函数将控制代码直接发送到指定的设备驱动程序，使设备执行相应的操作。这个函数有三个重要的部分：

* 控制代码 `在本例中，我们选择 IOCTL 0x800`
* 包含我们数据的输入缓冲区 `对本例中是 ThreadData，线程信息`
* 输出缓存

```
#define IOCTL_PRIORITY_BOOSTER_SET_PRIORITY CTL_CODE(VIKINGDRV2_DEVICE, \
0x800, METHOD_NEITHER, FILE_ANY_ACCESS)
struct ThreadData {
ULONG ThreadId;
int Priority;
};
```

#### 第二步，准备内核代码处理驱动客户端发来请求

创建入口点：

`DriverEntry(_In_ PDRIVER_OBJECT DriverObject, _In_ PUNICODE_STRING RegistryPath)`

设置调度例程来处理 `IRP_MJ_DEVICE_CONTROL`/驱动程序对象：我们将函数命名为 `vikingdrv2DeviceControl`。

`DriverObject->MajorFunction[IRP_MJ_DEVICE_CONTROL] = vikingdrv2DeviceControl;`

提供 `device name` 和 `symlink name` 名称，然后创建设备对象，以便客户端可以访问驱动程序并打开文件系统句柄。

```
UNICODE_STRING devName = RTL_CONSTANT_STRING(L"\\Device\\vikingdrv2"); // 驱动名
UNICODE_STRING symLink = RTL_CONSTANT_STRING(L"\\??\\vikingdrv2"); // symlink
PDEVICE_OBJECT DeviceObject;
NTSTATUS status = IoCreateDevice(
DriverObject, // 自定义的驱动对象
0,
& devName, // device name,
FILE_DEVICE_UNKNOWN, // device type,
0, // characteristics flags,
FALSE,
& DeviceObject // 输出指针位置
);
if (!NT_SUCCESS(status)) {
KdPrint(("Failed to create device object (0x%08X)\n", status));
return status;
}
```

现在我们有一个指向设备对象的指针，通过提供符号链接使用户模式调用方可以访问它。

```
status = IoCreateSymbolicLink(&symLink, &devName);
if (!NT_SUCCESS(status)) {
KdPrint(("Failed to create symbolic link (0x%08X)\n", status));
IoDeleteDevice(DeviceObject);
return status;
}
```

#### 第三步 驱动核心功能

前面是准备处理请求部分，下面是如何处理它。

首先，我们必须找到我们的堆栈位置 `从驱动程序的角度来看`，并确认我们的客户端/用户给了我们一个我们能够处理的 IOCTL。

```
NTSTATUS vikingdrv2DeviceControl(PDEVICE_OBJECT, PIRP Irp) {
auto stack = IoGetCurrentIrpStackLocation(Irp); // IO_STACK_LOCATION*
switch (stack->Parameters.DeviceIoControl.IoControlCode) {
case IOCTL_PRIORITY_BOOSTER_SET_PRIORITY: {
```

然后我们处理缓冲区以检索客户端准备的 `ThreadData` 结构
