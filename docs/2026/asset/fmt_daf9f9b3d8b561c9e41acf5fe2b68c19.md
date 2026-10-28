
键盘过滤驱动创建一个键盘设备将它附加在键盘类`KbdClass`设备栈上，当有键盘按下IRP消息的时候该设备最先接收到IRP消息，实现IRP HOOK。

创建过滤驱动设备的过程如下：

1. `IoCreateDevice`创建一个`FILE_DEVICE_KEYBOARD`键盘设备并调用`IoCreateSymbolicLink`为该设备创建一个符号链接。

2. 调用`IoGetDeviceObjectPointer`函数获取`KeyboardClass0`驱动设备对象

3. 通过`IoAttachDeviceToDeviceStack`将创建的键盘设备附加到`KeyboardClass0`设备对象所在的设备栈顶

```c
NTSTATUS AttachKdbClass(PDEVICE_OBJECT pDevObj)
{
DbgPrint("Enter HookKdbClass\n");
NTSTATUS status = STATUS_SUCCESS;
UNICODE_STRING ustrObjectName;
PFILE_OBJECT pFileObj = NULL;
PDEVICE_OBJECT pKeyboardClassDeviceObject = NULL, pAttachDevObj = NULL;
RtlInitUnicodeString(&ustrObjectName, L"\\Device\\KeyboardClass0");
// 获取键盘设备对象的指针
status = IoGetDeviceObjectPointer(&ustrObjectName, GENERIC_READ | GENERIC_WRITE, &pFileObj, &pKeyboardClassDeviceObject);
if (!NT_SUCCESS(status))
{
DbgPrint("IoGetDeviceObjectPointer Error[0x%X]\n", status);
return status;
}
// 减少引用
ObReferenceObject(pFileObj);
// 将当前设备附加到键盘设备的设备栈顶上, 返回的是原来设备栈上的栈顶设备, 即当前设栈上附加设备的下一个设备
pAttachDevObj = IoAttachDeviceToDeviceStack(pDevObj, pKeyboardClassDeviceObject);
if (NULL == pAttachDevObj)
{
DbgPrint("IoAttachDeviceToDeviceStack Error\n");
return STATUS_UNSUCCESSFUL;
}
// 设置此设备的标志，要与附加到的设备栈上的设备标志保持一致
pDevObj->Flags = pDevObj->Flags | DO_BUFFERED_IO | DO_POWER_PAGABLE;
pDevObj->ActiveThreadCount = pDevObj->ActiveThreadCount & (~DO_DEVICE_INITIALIZING);
// 保存下一设备到DeviceExtension
((PDEVICE_EXTENSION)pDevObj->DeviceExtension)->pAttachDevObj = pAttachDevObj;
((PDEVICE_EXTENSION)pDevObj->DeviceExtension)->ulIrpInQuene = 0;
DbgPrint("Leave HookKdbClass\n");
return status;
}
```

自定义键盘驱动捕获到IRP请求再把该IRP请求发送给操作系统

IRP读请求中处理流程如下：

1. 调用`IoCopyCurrentIrpStackLocationToNext`将当前设备中的IRP复制到下一个设备中

2. 调用`IoSetCompletionRoutine`函数设置完成回调函数，将在下一层驱动完成由IRP指定的操作请求时调用这个函数

3. 调用`IoCallDriver`函数将IRP发送到下一个设备

```c
NTSTATUS DriverRead(PDEVICE_OBJECT pDevObj, PIRP pIrp)
{
DbgPrint("\nEnter DriverRead\n");
NTSTATUS status = STATUS_SUCCESS;
// 复制 pIrp 的 IO_STACK_LOCATION 到下一设备
IoCopyCurrentIrpStackLocationToNext(pIrp);
// 设置完成实例
IoSetCompletionRoutine(pIrp, ReadCompleteRoutine, pDevObj, TRUE, TRUE, TRUE);
// 记录IRP数量
((PDEVICE_EXTENSION)pDevObj->DeviceExtension)->ulIrpInQuene++;
// 发送IRP到下一设备上
status = IoCallDriver(((PDEVICE_EXTENSION)pDevObj->DeviceExtension)->pAttachDevObj, pIrp);
DbgPrint("Leave DriverRead\n");
return status;
}
```

完成回调函数的具体代码如下

```c
NTSTATUS ReadCompleteRoutine(PDEVICE_OBJECT pDevObj, PIRP pIrp, PVOID pContext)
{
NTSTATUS status = pIrp->IoStatus.Status;
PKEYBOARD_INPUT_DATA pKeyboardInputData = NULL;
ULONG ulKeyCount = 0, i = 0;
if (NT_SUCCESS(status))
{
pKeyboardInputData = (PKEYBOARD_INPUT_DATA)pIrp->AssociatedIrp.SystemBuffer;
ulKeyCount = (ULONG)pIrp->IoStatus.Information / sizeof(KEYBOARD_INPUT_DATA);
// 获取按键数据
for (i = 0; i < ulKeyCount; i++)
{
// Key Press
if (KEY_MAKE == pKeyboardInputData[i].Flags)
{
// 按键扫描码
DbgPrint("[Down][0x%X]\n", pKeyboardInputData[i].MakeCode);
}
// Key Release
else if (KEY_BREAK == pKeyboardInputData[i].Flags)
{
// 按键扫描码
DbgPrint("[Up][0x%X]\n", pKeyboardInputData[i].MakeCode);
}
//可以添加上这这一句，然后按键全部被改为了 按下 A
// pKeyboardInputData[i].MakeCode = 0x1e;
}
}
if (pIrp->PendingReturned)
{
IoMarkIrpPending(pIrp);
}
// 减少IRP在队列的数量
((PDEVICE_EXTENSION)pDevObj->DeviceExtension)->ulIrpInQuene--;
status = pIrp->IoStatus.Status;
return status;
}
```

任何时候设备栈底都会有一个键盘的`IRP_MJ_READ`请求处于挂起状态。一般的方式卸载键盘过滤驱动基本都有`IRP_MJ_READ`请求处于挂起未确定状态。这时若执行卸载驱动操作那么在以后按键需要这个IRP时却找不到对应驱动，从而导致计算机蓝屏。所以必须处理好这个挂起状态的IRP。

实现动态卸载过滤的方法是在设备增加一个`DeviceExtension`,并添加一个`u1IrpInQuene`变量来记录。卸载驱动的时候先调用`IoDetachDevice`,取消附加键盘设备。一直循环判断`u1IrpInQuene`是否为0，若不为0则一直循环等待。直到用户手动按键就会在IRP中返回。由于不附加键盘设备栈，所以也不会有新的`IRP_MJ_READ`产生，此时的`u1IrpInQuene`的值是0，可以正确卸载。

卸载程序：

```c
VOID DriverUnload(PDRIVER_OBJECT pDriverObject)
{
DbgPrint("Enter DriverUnload\n");
PDEVICE_OBJECT pDevObj = pDriverObject->DeviceObject;
LARGE_INTEGER liDelay = { 0 };
if (NULL == pDevObj)
{
return;
}
if (NULL == pDevObj->DeviceExtension)
{
return;
}
// 先把过滤驱动设备和键盘设备分离, 以免产生新的IRP_MJ_READ
IoDetachDevice(((PDEVICE_EXTENSION)pDevObj->DeviceExtension)->pAttachDevObj);
// 对于为完成的IRP，因为只前通过IoSetCompletionRoutine已经设置IO完成例程
// 那么对于未完成的IRP ，在完成之后会调用 该层设备的函数
// 需要手动按键, 是pending状态的IRP完成返回
liDelay.QuadPart = -1000000;
while (0 < ((PDEVICE_EXTENSION)pDevObj->DeviceExtension)->ulIrpInQuene)
{
KdPrint(("剩余挂起IRP:%d\n", ((PDEVICE_EXTENSION)pDevObj->DeviceExtension)->ulIrpInQuene));
KeDelayExecutionThread(KernelMode, FALSE, &liDelay);
/*
// 该方法在Win10上不适用
if (1 == ((PDEVICE_EXTENSION)pDevObj->DeviceExtension)->ulIrpInQuene)
{
//取消掉最后一个IRP
IoCancelIrp(((PDEVICE_EXTENSION)pDevObj->DeviceExtension)->pLastIrp);
}
*/
}
UNICODE_STRING ustrSymName;
RtlInitUnicodeString(&ustrSymName, SYM_NAME);
IoDeleteSymbolicLink(&ustrSymName);
if (pDriverObject->DeviceObject)
{
IoDeleteDevice(pDriverObject->DeviceObject);
}
DbgPrint("Leave DriverUnload\n");
}
```

参考资料：

- 看雪论坛：通俗解析IRP和I/O设备栈在内核程序中的作用 `https://bbs.pediy.com/thread-111559.htm`
- 《黑客免杀攻防》
- 《黑客编程技术详解》

SSDT全称为System Service Descriptor Table,即系统服务描述符表。SSDT的作用就是把Ring3的WIN32 API函数和Ring0的内核API联系起来。R3下的一些API最终会对应ntdll.dll中的一个`Nt`开头的函数。将系统服务号放入`EAX`然后调用`KiSystemService`进入内核根据系统服务号得到对应系统服务的内核地址。HIPS，杀毒软件往往会通过SSDT HOOK实现监控系统的目的。

32位系统中SSDT是由内核`Ntoskrnl.exe`导出的一张表，导出符号为`KeServiceDescriptorTable`。在64位系统上是不导出的。

以下程序实现将`ntdll.dll`文件映射到内核空间并从导出表中获取导出函数地址，然后获取SSDT函数索引号。

具体操作流程如下：

1. 调用`InitializeObjectAttributes`宏初始化文件对象，它主要初始化文件路径，内核下的文件路径需要在路径前面加上`\??\`

2. 调用`ZwCreateSection`函数生成一个内存映射对象得到句柄`hSection`,该对象建立在由`ZwOpenFile`函数创建的文件对象基础上。该内存映射对象对视图读写具有`SECTION_MAP_READ`和`SECTION_MAP_WRITE`访问权限，同时设置内存映射方式为`SEC_IMAGE`，以PE中的`SectionAlignment`大小对齐映射文件

3. 调用`ZwMapViewOfSection`函数把文件映射到内存得到`lpMemory`。该指针直接读写内存

4. 调用`ZwUnmapViewOfSection`解除文件映射，进程句柄，`lpMemory`作为参数。

5. 接下来根据PE文件结构获取导出函数地址并从导出函数中获取SSDT函数索引号，根据`IMAGE_DOS_HEADER`和`IMAGE_NT_HEADERS`计算出`OptionalHeader`,获取数据目录中导出表的`RVA`地址查找匹配的函数名称从`AddressOfNamesOrdinal`中获取索引值，32位系统下导出函数偏移1字节后的四字节就是SSDT函数索引号，64位系统下需要偏移4字节

|  |  |
| --- | --- |
