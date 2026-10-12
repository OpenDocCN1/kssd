
```c
DEBUGP(DL_TRACE, ("===>NDISLWF FilterPause: FilterInstance %p\n", FilterModuleContext));
FILTER_ASSERT(pFilter->State == FilterRunning);
FILTER_ACQUIRE_LOCK(&pFilter->Lock, bFalse);
pFilter->State = FilterPausing;
FILTER_RELEASE_LOCK(&pFilter->Lock, bFalse);
Status = NDIS_STATUS_SUCCESS;
pFilter->State = FilterPaused;
DEBUGP(DL_TRACE, ("<===FilterPause:  Status %x\n", Status));
return Status;
}
```

`FilterRestart` 函数的功能：

在 `Restarting` 状态下，一个 Filter Driver 必须为一个 Filter Module 完成重启发送和接收数据时所需要的所有准备工作。

`Restart is complete`

当 Filter Module 在 `Restarting` 状态下并且完成所有发送和接收所需要的准备工作时，Filter Module 就进入了 `Running` 状态。

要启动一个 `Paused` 状态的 Filter Module，如果有 `FilterSetModuleOptions` 就先调用它，接着调用 `FilterRestart`。

当 Filter Module 在 `Restarting` 状态下，它可以：

(1) 完成任何正常的发送和接收所需要的准备工作。
(2) 可读写 Filter Module 的配置参数。
(3) 可以接收网络数据指示，拷贝和排队数据稍后只是给上层驱动或者丢弃数据。
(4) 不能发起任何新的接收指示。
(5) 应该立即调用 `NdisFSendNetBufferListsComplete` 例程来拒绝 `FilterSendNetBufferLists` 传来的发送的请求。应该设置每一个 `NET_BUFFER_LIST` 的完成状态为 `NDIS_STATUS_PAUSED`
(6) 可以使用 `NdisFIndicateStatus` 例程进行状态指示。
(7) 可以控制 OID 请求操作。
(8) 不能发起任何新的发送请求。
(9) 应该调用 `NdisFReturnNetBufferLists` 返回所有新的接收指示。如果需要的话可以在接收指示返回之前拷贝它们。
(10) 可以制作 OID 请求发送给下层驱动设置或查询配置信息。
(11) 可以在 `FilterStatus` 中控制状态指示。
(12) 返回时指示 `NDIS_STATUS_SUCCESS` 或失败状态，如果不 Filter Module 不能启动返回了失败，而它又是一个 Mandatory 的 Filter Driver 那个 NDIS 将会结束这个驱动栈

在一个 Filter Driver 完成对发送和接收的重启后必须指示完成这个重启操作。

Filter Driver 的重启操作的完成可以是同步也可以是异步的，同步时返回 `NDIS_STATUS_SUCCESS` 异步时返回 `NDIS_STATUS_PENDING`。如果返回的是 `NDIS_STATUS_PENDING` 就必须调用 `NdisFRestartComplete` 例程在重启操作完成后。

在这种情况下，驱动需要传递给 `NdisFRestartComplete` 一个固定状态（标识重启结果的成功或失败状态）。

重启操作完成 Filter Module 就进入了 `Running` 状态，恢得一切正常的发送和接收外理。

在 Filter Driver 的 `FilterRestart` 例程执行的时候 NDIS 不会发起任即插即用操作，如附加、分离、暂停请求等等。Ndis 可以在 Filter Module 进入 `Running` 状态后发起一个暂停请求。

```c
NDIS_STATUS
FilterRestart(
    IN  NDIS_HANDLE                    FilterModuleContext,
    IN  PNDIS_FILTER_RESTART_PARAMETERS RestartParameters
    )
{
    NDIS_STATUS     Status;
    PMS_FILTER      pFilter = (PMS_FILTER)FilterModuleContext; // BUGBUG, the cast may be wrong
    NDIS_HANDLE     ConfigurationHandle = NULL;
    PNDIS_RESTART_GENERAL_ATTRIBUTES NdisGeneralAttributes;
    PNDIS_RESTART_ATTRIBUTES         NdisRestartAttributes;
    NDIS_CONFIGURATION_OBJECT        ConfigObject;
    DEBUGP(DL_TRACE, ("===>FilterRestart:   FilterModuleContext %p\n", FilterModuleContext));
    FILTER_ASSERT(pFilter->State == FilterPaused);
    ConfigObject.Header.Type = NDIS_OBJECT_TYPE_CONFIGURATION_OBJECT;
    ConfigObject.Header.Revision = NDIS_CONFIGURATION_OBJECT_REVISION_1;
    ConfigObject.Header.Size = sizeof(NDIS_CONFIGURATION_OBJECT);
    ConfigObject.NdisHandle = FilterDriverHandle;
    ConfigObject.Flags = 0;
    Status = NdisOpenConfigurationEx(&ConfigObject, &ConfigurationHandle);
    if (Status != NDIS_STATUS_SUCCESS)
    {        
#if 0
        //
        // The code is here just to demonstrate how to call NDIS to write an eventlog. If drivers need to write
        // an event log.
        //
        PWCHAR             ErrorString = L"Ndislwf";
        DEBUGP(DL_WARN, ("FilterRestart: Cannot open configuration.\n"));
        NdisWriteEventLogEntry(FilterDriverObject,
                                EVENT_NDIS_DRIVER_FAILURE,
                                0,
                                1,
                                &ErrorString,
                                sizeof(Status),
                                &Status);
#endif                        
    }
    if (Status == NDIS_STATUS_SUCCESS)
    {
        NdisCloseConfiguration(ConfigurationHandle);
    }
    NdisRestartAttributes = RestartParameters->RestartAttributes;
    if (NdisRestartAttributes != NULL)
    {
        PNDIS_RESTART_ATTRIBUTES   NextAttributes;
        ASSERT(NdisRestartAttributes->Oid == OID_GEN_MINIPORT_RESTART_ATTRIBUTES);
        NdisGeneralAttributes = (PNDIS_RESTART_GENERAL_ATTRIBUTES)NdisRestartAttributes->Data;
        NdisGeneralAttributes->LookaheadSize = 128;
        NextAttributes = NdisRestartAttributes->Next;
        while (NextAttributes != NULL)
        {
            NextAttributes = NextAttributes->Next;
        }
    }
    pFilter->State = FilterRunning;
    Status = NDIS_STATUS_SUCCESS;
    if (Status != NDIS_STATUS_SUCCESS)
    {
        pFilter->State = FilterPaused;
    }
    DEBUGP(DL_TRACE, ("<===FilterRestart:  FilterModuleContext %p, Status %x\n", FilterModuleContext, Status));
    return Status;
}
```

`FilterDetach` 函数的功能：

`Detach` 状态：当 Filter Driver 从一个驱动栈上分离一个 Filter Module 时，将发生该事件。
在驱动栈上分离一个过滤模块时，NDIS 会暂停这个驱动栈。这意味着 NDIS 已经使过滤模块进入了 Parse 状态。即 `FilterPause` 函数先被调用了。
在这个例程中释放和这个过滤模块相关的环境上下文和其它资源。这个过程不能失败。
当 `FilterDetach` 函数返回以后，NDIS 会重新启动被暂停的驱动栈。

参数说明：
`FilterDriverContext` 它由 `NdisFRegisterFilterDriver` 的 `FilterDriverContext` 来指定。

```c
VOID
FilterDetach(
        IN  NDIS_HANDLE     FilterModuleContext
        )
{
    PMS_FILTER            pFilter = (PMS_FILTER)FilterModuleContext;
    BOOLEAN               bFalse = FALSE;
    DEBUGP(DL_TRACE, ("===>FilterDetach:    FilterInstance %p\n", FilterModuleContext));
    FILTER_ASSERT(pFilter->State == FilterPaused);
    if (pFilter->FilterName.Buffer != NULL)
    {
        FILTER_FREE_MEM(pFilter->FilterName.Buffer);
    }
    FILTER_ACQUIRE_LOCK(&FilterListLock, bFalse);
    RemoveEntryList(&pFilter->FilterModuleLink);
    FILTER_RELEASE_LOCK(&FilterListLock, bFalse);
    FILTER_FREE_MEM(pFilter);
    DEBUGP(DL_TRACE, ("<===FilterDetach Successfully\n"));
    return;
}
```

系统只会在调用 `FilterDetach()` 分离了所有和本 Filter Driver 相关的 Filter Module 以后，才会调用 `FilterUnload` 例程。

```c
VOID
FilterUnload(
        IN  PDRIVER_OBJECT    DriverObject
        )
{
#if DBG      
    BOOLEAN         bFalse = FALSE;
#endif
    UNREFERENCED_PARAMETER(DriverObject);
    DEBUGP(DL_TRACE, ("===>FilterUnload\n"));
    FilterDeregisterDevice();
    NdisFDeregisterFilterDriver(FilterDriverHandle);
#if DBG      
    FILTER_ACQUIRE_LOCK(&FilterListLock, bFalse);
    ASSERT(IsListEmpty(&FilterModuleList));
    FILTER_RELEASE_LOCK(&FilterListLock, bFalse);
#endif      
    FILTER_FREE_LOCK(&FilterListLock);
    DEBUGP(DL_TRACE, ("<===FilterUnload\n"));
    return;
}
```

`FilterOidRequest` 函数的功能：

Filter Module 可以在 `Runnig` 状态、`Restarting` 状态、`Paused` 状态和 `Pauseing` 状态进行 OID 的控制和处理。
Filter Driver 可以处理上层驱动引发的 OID 请求，NDIS 调用 Filter Driver 的 `FilterOidRequest` 例程来处理 OID 请求，Filter Driver 需要调用 `NdisFOidRequest` 例程来转发请求给下层驱动。
