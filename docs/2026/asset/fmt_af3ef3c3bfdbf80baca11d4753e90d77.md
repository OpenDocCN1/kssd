
`FilterReceiveNetBufferLists`

上述四个例程也定义在`NDIS_FILTER_PARTIAL_CHARACTERISTICS`中，这个结构中指定的例程可以在运行时的`FilterSetModuleOptions`例程中调用`NdisSetOptionalHandles`来改变。如果 Filter Drivers 要在例程中改自身的一个特性，那么就必须提供`FilterSetModuleOpitons`例程。部分特性每一个不同的 Filter Module 可以各不相同。

如果`NdisFRegisterFilterDriver`调用成功，NDIS 会填充`NdisFilterDriverHandle`变量返回一个 Filter Driver 句柄。Filter Driver 应该保存这个句柄，后面调用 NDIS 例程时会使这个句柄作为输入参数，例如`NdisFDeregisterFilterDriver`例程就用它做输入参数。在驱动被卸载时会调用驱动的卸载例程，必须在卸载例程中调用`NdisFDeregisterFilterDriver`来释放这个句柄，同时也要释放所有已申请分配的资源。

`FilterSetOptions`返回后，Filter Module 处于`Detached`状态。NDIS 可以在`FilterSetPtions`返回后的任何时间调用`FilterAttach`例程。Filter Module 在`FilterAttach`中完成和它相关的特定初始化。

Filter Drivers 也可以在`DriverEntry`中选择一些其它（如全局的）初始化。但一定要在`FilterDriverUnload`例程中释放所有初始化时或运行时申请分配的资源。下面是 DDK 中所带例子的`DriverEntry`例程的源码。

```
NDIS_STATUS
DriverEntry(
IN PDRIVER_OBJECT   DriverObject,
IN PUNICODE_STRING  RegistryPath
)
{
NDIS_STATUS               Status;
NDIS_FILTER_DRIVER_CHARACTERISTICS   FChars;
NDIS_STRING               ServiceName;
NDIS_STRING               UniqueName;
NDIS_STRING               FriendlyName;

DEBUGP(DL_TRACE,("===>DriverEntry...\n"));

RtlInitUnicodeString(&ServiceName, FILTER_SERVICE_NAME);
RtlInitUnicodeString(&FriendlyName, FILTER_FRIENDLY_NAME);
RtlInitUnicodeString(&UniqueName, FILTER_UNIQUE_NAME);
FilterDriverObject = DriverObject;

do
{
NdisZeroMemory(&FChars, sizeof(NDIS_FILTER_DRIVER_CHARACTERISTICS));
FChars.Header.Type = NDIS_OBJECT_TYPE_FILTER_DRIVER_CHARACTERISTICS;
FChars.Header.Size = sizeof(NDIS_FILTER_DRIVER_CHARACTERISTICS);
FChars.Header.Revision = NDIS_FILTER_CHARACTERISTICS_REVISION_1;
FChars.MajorNdisVersion = FILTER_MAJOR_NDIS_VERSION;
FChars.MinorNdisVersion = FILTER_MINOR_NDIS_VERSION;
FChars.MajorDriverVersion = 1;
FChars.MinorDriverVersion = 0;
FChars.Flags = 0;

FChars.FriendlyName = FriendlyName;
FChars.UniqueName = UniqueName;
FChars.ServiceName = ServiceName;

//
// for the time being, there is no additional options to register
// but let's have this handler anyway
//
FChars.SetOptionsHandler = FilterRegisterOptions;
FChars.AttachHandler = FilterAttach;
FChars.DetachHandler = FilterDetach;
FChars.RestartHandler = FilterRestart;
FChars.PauseHandler = FilterPause;
FChars.SetFilterModuleOptionsHandler = FilterSetModuleOptions;
FChars.OidRequestHandler = FilterOidRequest;
FChars.OidRequestCompleteHandler = FilterOidRequestComplete;
FChars.CancelOidRequestHandler = FilterCancelOidRequest;

FChars.SendNetBufferListsHandler = FilterSendNetBufferLists;
FChars.ReturnNetBufferListsHandler = FilterReturnNetBufferLists;
FChars.SendNetBufferListsCompleteHandler =
FilterSendNetBufferListsComplete;
FChars.ReceiveNetBufferListsHandler = FilterReceiveNetBufferLists;
FChars.DevicePnPEventNotifyHandler = FilterDevicePnPEventNotify;
FChars.NetPnPEventHandler = FilterNetPnPEvent;
FChars.StatusHandler = FilterStatus;
FChars.CancelSendNetBufferListsHandler = FilterCancelSendNetBufferLists;

DriverObject->DriverUnload = FilterUnload;
FilterDriverHandle = NULL;

FILTER_INIT_LOCK(&FilterListLock);

InitializeListHead(&FilterModuleList);

Status = NdisFRegisterFilterDriver(DriverObject,
(NDIS_HANDLE)FilterDriverObject,
&FChars,
&FilterDriverHandle);
if (Status != NDIS_STATUS_SUCCESS)
{
DEBUGP(DL_WARN, ("MSFilter: Register filter driver failed.\n"));
break;
}
//
// Initilize spin locks
//

Status = FilterRegisterDevice();

if (Status != NDIS_STATUS_SUCCESS)
{
NdisFDeregisterFilterDriver(FilterDriverHandle);
FILTER_FREE_LOCK(&FilterListLock);
DEBUGP(DL_WARN, ("MSFilter: Register device for the filter driver
failed.\n"));
break;
}

}
while(FALSE);

DEBUGP(DL_TRACE, ("<===DriverEntry, Status = %8x\n", Status));
return Status;

}
```

## 2.2 卸载 Filter Drivers

一个与 Filter Drivers 相关联的驱动对象联结一个 Unload 例程，系统会在这个 Filter Driver 在所有微端口适配器（Miniport Adapter）上的过滤服务被移除后调用这个 Unload 例程。Filter Drivers 的 Unload 例程被命名为`FilterDriverUnload`。

`FilterDriverUnload`应该释放与自己驱动相关的资源。任何 Filter Driver 创建的驱动对象都要被销毁。当`FilterDriverUnload`返回后系统就可以完成 Filter Driver 的卸载操作了。

卸载例程的功能是和驱动相关的，一般的规则是，卸载例程应该撤销初始化时执行的所有操作。一个 Filter Driver 例程必须在`FilterDriverUnload`例程中调用`NdisFDeregisterFilterDriver`来撤销`NdisFRegisterFilterDriver`的注册。

注意：系统只会在调用`FilterDetach`分离了所有和本 Filter Driver 相关的 Filter Module 后才会调用`FilterDriverUnload`例程。下面贴出 DDK 中例子 filter 的`FilterDriverUnload`源码。

```
VOID
FilterUnload(
IN PDRIVER_OBJECT   DriverObject
)
{
DEBUGP(DL_TRACE, ("===>FilterUnload\n"));
//
// Should free the filter context list
//
FilterDeregisterDevice();
NdisFDeregisterFilterDriver(FilterDriverHandle);

#if DBG
FILTER_ACQUIRE_LOCK(&FilterListLock, FALSE);
ASSERT(IsListEmpty(&FilterModuleList));

FILTER_RELEASE_LOCK(&FilterListLock, FALSE);

#endif

FILTER_FREE_LOCK(&FilterListLock);

DEBUGP(DL_TRACE, ("<===FilterUnload\n"));

return;

}
```

## 2.3 Filter Module 状态和操作

一个 Filter Driver 的 Filter Module（Filter Driver 的一个实例）必须支持下列操作状态。

1） `Detached`

Detached 状态是每一个 Filter Module 的初始状态。当一个 Filter Driver 停留在这个状态时，它可以附加一个 Filter Module 到一个驱动栈上。

2） `Attaching`

这个状态是标识一个 Filter Driver 正准备附加一个 Filter Module 到一个驱动栈上。

3） `Paused`

在这个状态时，Filter Driver 不能执行接收和发送操作。

4） `Restarting`

在这个状态时，一个 Filter Driver 必须为一个 Filter Module 完成重启发送和接收数据时所需要的所有准备工作。

5） `Running`

在这个状态时，一个 Filter Driver 执行 Filter Module 正常的发送和接收处理。

6） `Pausing`

在这个状态时，一个 Filter Driver 要为一个 Filter Module 完成停止发送和接收处理所需要的所有准备工作。

在下面表格中，标题列出了 Filter Module 的所有状态，第一列是主要的事件。其余的条目是要进入的下一个状态，它是事件发生时的内部状态。空白条目表示无效状态/事件的组合。

![](https://bbs.kanxue.com/upload/attach/201402/203861_7gu93ut3br00xvv.png)

![](https://bbs.kanxue.com/upload/attach/201402/203861_dnig1trl9w83zs2.png)

主要的过滤驱动事件定义如下：

1） `FilterAttach`

当 NDIS 调用`FilterAttach`例程向一个驱动栈附加一个 Filter Module 时发生这个事件。

2） Attach is complete

当 Filter Module 在`Attaching`状态时且 Filter Driver 初始化了 Filter Module 所需要的所有资源时，Filter Module 进入`Paused`状态。

3） `FilterDetach`

当 NDIS 调用 Filter Driver 的`FilterDetach`从一个驱动栈上分离一个 Filter Module 时发生这个事件。

4） `FilterRestart`

当 NDIS 调用 Filter Driver 的`FilterRestart`把一个暂停的 Filter Module 重新启动时发生这个事件。

5） Restart is complete

当 Filter Module 在`Restarting`状态时且完成所有发送和接收所需要的准备工作时，Filter Module 就进入了`Running`状态。

6） `FilterPause`

当 NDIS 调用 Filter Driver 的`FilterPause`暂停一个 Filter Module 时发生此事件。

7） Pause is complete

当 Filter Driver 为 Filter Module 完成了所有发送和接收操作的停止工作时，暂停完成同时也进入了`Paused`状态。

8） Attach fail

当 NDIS 调用`FilterAttach`操作失败时（比如，Filter Module 所有需要的资源申请分配失败等等），这时就进入了`Detached`状态。

9） Restart fail

当 NDIS 调用`FilterRestart`操作失败时，Filter Module 就进入了`Paused`状态。

10）Send and Receive Operations

Filter Module 可以在`Pausing`状态和`Running`状态进行发送和接收操作的处理。

11）OID Requests

Filter Module 可以在`Running`状态、`Restarting`状态、`Paused`状态和`Pausing`状态进行 OID 的控制和处理。

## 2.4 附加 Filter Module

在向一个驱动栈插入一个 Filter Module 时，NDIS 会调用 Filter Driver 的`FilterAttach`例程。在进入执行`FilterAttach`时，Filter Module 就进入了`Attaching`状态。下面列出`FilterAttach`在 DDK 中的声明。

```
NDIS_STATUS
FilterAttach(

IN NDIS_HANDLE  NdisFilterHandle,
IN NDIS_HANDLE  FilterDriverContext,
IN PNDIS_FILTER_ATTACH_PARAMETERS  AttachParameters
);
```

NDIS 通过`NdisFilterHandle`传过来一个句柄，这个句柄将用于所有 Filter Driver 中对`NdisXXX`类例程的调用时引用指示这个 Filter Module，这些例程有状态指示、发送请求、接收指示、OID 请求等等。当一个 Filter Module 进入`Attaching`状态时驱动可以：

1） 创建一个环境上下文区域并且初始化一个缓冲区池以及其 Filter Module 特定的资源

2） 用 NDIS 传给`FilterAttach`的`NdisFilterHandle`作为输入来调用`NdisFSetAttributes`例程，其中`NdisFSetAttributes`的`FilterModuleContext`参数的作用是为 Filter Module 指定环境上下文用的（注意：这里要与`NdisFRegisterFilterDriver`中的`FilterDriverContext`参数区别开来）。NDIS 会把这个环境上下文传递给 Filter Drivers 注册的`FilterXXX`例程（注意：`FilterAttach`中的`FilterDriverContext`是由`NdisFRegisterFilterDriver`的`FilterDriverContext`指定的），下面列出`NdisFSetAttributes`例程在 DDK 中的声明：

```
NDIS_STATUS
NdisFSetAttributes(

IN NDIS_HANDLE  NdisFilterHandle,
IN NDIS_HANDLE  FilterModuleContext,
IN PNDIS_FILTER_ATTRIBUTES  FilterAttributes
);
```

3） 这步是可选的，Filter Module 可以从注册表中读取一些参数。

4） 如果上述的操作完成成功，那么 Filter Module 就进入了`Paused`状态。

5） 如果上述的操作完成失败，那么要释放已经成功申请分配的资源，返回后进入`Detached`状态。
