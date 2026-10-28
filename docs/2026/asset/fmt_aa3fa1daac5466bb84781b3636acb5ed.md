
如果 `Filter Driver` 设置了 `NdisFIndicateReceiveNetBufferLists` 的状态为 `NDIS_STATUS_SUCCESS`, `NDIS` 通过驱动的 `FilterReturnNetBufferLists` 返回指示数据。在这种情况下 `Filter Driver` 失去了对 `NET_BUFFER_LIST` 的所有权,直到 `FilterReturnNetBufferLists` 被调用。

`Filter Driver` 调用 `NdisFIndicateNetBufferLists` 传递接收指示给驱动栈上的上层驱动,如果上层驱动保留了对缓冲区 (`NET_BUFFER_LIST`) 的所有权,`NDIS` 会调用 `Filter Driver` 的 `FilterReturnNetBufferLists` 例程。

在 `FilterReturnNetBufferLists` 中应该撤消在接收路径上 (如在 `FilterReciveNetBufferLists` 中做的一些处理) 的操作。当最底层的 `Filter Module` 完成对缓冲区 (`NET_BUFFER_LIST`) 的处理后,`NDIS` 把缓冲区返回给微端口驱动。如果 `FilterReceiveNetBufferLists` 的 `ReceiveFlags` 没有设置 `NDIS_RECEIVE_FLAGS_RESOURCES` 标记, `FilterDriver` 调用 `NdisFReturnNetBufferList` 返回这个缓冲区数据,如果设置了 `FilterReceiveNetBufferLists` 直接返回时就把缓冲区返还给了下层微端口驱动。

### `FilterReturnNetBufferLists`

```
VOID
FilterReturnNetBufferLists(
        IN  NDIS_HANDLE         FilterModuleContext,
        IN  PNET_BUFFER_LIST    NetBufferLists,
        IN  ULONG               ReturnFlags
        )
{
    PMS_FILTER          pFilter = (PMS_FILTER)FilterModuleContext;
    PNET_BUFFER_LIST    CurrNbl = NULL;
    UINT                NumOfNetBufferLists = 0;
    BOOLEAN             DispatchLevel;
    ULONG               Ref;
    DEBUGP(DL_TRACE, ("===>ReturnNetBufferLists, NetBufferLists is %p.\n", NetBufferLists));
    if (pFilter->TrackReceives)
    {
        while (CurrNbl)
        {
            NumOfNetBufferLists ++;
            CurrNbl = NET_BUFFER_LIST_NEXT_NBL(CurrNbl);
        }
    }
    NdisFReturnNetBufferLists(pFilter->FilterHandle, NetBufferLists, ReturnFlags);
    if (pFilter->TrackReceives)
    {
        DispatchLevel = NDIS_TEST_RETURN_AT_DISPATCH_LEVEL(ReturnFlags);
        FILTER_ACQUIRE_LOCK(&pFilter->Lock, DispatchLevel);
        pFilter->OutstandingRcvs -= NumOfNetBufferLists;
        Ref = pFilter->OutstandingRcvs;
        FILTER_LOG_RCV_REF(3, pFilter, NetBufferLists, Ref);
        FILTER_RELEASE_LOCK(&pFilter->Lock, DispatchLevel);
    }
    DEBUGP(DL_TRACE, ("<===ReturnNetBufferLists.\n"));
}
```

### `FilterReceiveNetBufferLists` 函数的功能

`Filter Driver` 调用 `NdisFIndicateReceiveNetBufferLists` 来指示发送数据。这个函数通过 `NET_BUFFER_LIST` 结构给上层驱动指示数据。`Filter Driver` 可以从池中分配这个结构。如果 `Filter Driver` 设置了 `NdisFIndicateReceiveNetBufferLists` 的状态为 `NDIS_STATUS_SUCCESS`, `NDIS` 通过驱动的 `FilterReturnNetBufferLists` 返回指示数据。在这种情况下 `Filter Driver` 失去了对 `NET_BUFFER_LIST` 的所有权直到 `FilterReturnNetBufferLists` 被调用。如果 `Filter Driver` 在调用 `NdisFIndicateReceiveNetBufferLists` 时设置 `ReceiveFlags` 为 `NDIS_RECEIVE_FLAGS_RESOURCES`, 在函数返回后 `Filter Driver` 会立即恢复对 `NET_BUFFER_LIST` 的所有权,这时 `Filter Driver` 必须立即处理这个 `NET_BUFFER_LIST` 的返回,因为 `NDIS` 在这种情况下是不会调用 `FilterReturnNetBufferLists` 返回 `NET_BUFFER_LIST` 结构的。

注意: 一个 `Filter Driver` 应该跟踪自己引发的接收指示确保它在 `FilterReturnNetBufferLists` 中不调用 `NdisFReturnNetBufferLists`。

### `FilterReceiveNetBufferLists`

```
VOID
FilterReceiveNetBufferLists(
        IN  NDIS_HANDLE         FilterModuleContext,
        IN  PNET_BUFFER_LIST    NetBufferLists,
        IN  NDIS_PORT_NUMBER    PortNumber,
        IN  ULONG               NumberOfNetBufferLists,
        IN  ULONG               ReceiveFlags
         )
{
    PMS_FILTER          pFilter = (PMS_FILTER)FilterModuleContext;
    BOOLEAN             DispatchLevel;
    ULONG               Ref;
    BOOLEAN               bFalse = FALSE;
#if DBG
    ULONG               ReturnFlags;
#endif
    DEBUGP(DL_TRACE, ("===>ReceiveNetBufferList: NetBufferLists = %p.\n", NetBufferLists));
    do
    {
        DispatchLevel = NDIS_TEST_RECEIVE_AT_DISPATCH_LEVEL(ReceiveFlags);
#if DBG
        FILTER_ACQUIRE_LOCK(&pFilter->Lock, DispatchLevel);
        if (pFilter->State != FilterRunning)
        {
            FILTER_RELEASE_LOCK(&pFilter->Lock, DispatchLevel);
            if (NDIS_TEST_RECEIVE_CAN_PEND(ReceiveFlags))
            {
                ReturnFlags = 0;
                if (NDIS_TEST_RECEIVE_AT_DISPATCH_LEVEL(ReceiveFlags))
                {
                    NDIS_SET_RETURN_FLAG(ReturnFlags, NDIS_RETURN_FLAGS_DISPATCH_LEVEL);
                }
                NdisFReturnNetBufferLists(pFilter->FilterHandle, NetBufferLists, ReturnFlags);
            }
            break;
        }
        FILTER_RELEASE_LOCK(&pFilter->Lock, DispatchLevel);
#endif
        ASSERT(NumberOfNetBufferLists >= 1);
/*--------------------------------------------------------------------------------------*/
                  //在这里添加我们的代码
/*---------------------------------------------------------------------------------------*/
        if (pFilter->TrackReceives)
        {
            FILTER_ACQUIRE_LOCK(&pFilter->Lock, DispatchLevel);
            pFilter->OutstandingRcvs += NumberOfNetBufferLists;
            Ref = pFilter->OutstandingRcvs;
            FILTER_LOG_RCV_REF(1, pFilter, NetBufferLists, Ref);
            FILTER_RELEASE_LOCK(&pFilter->Lock, DispatchLevel);
        }
/*****************************************************************************************
调用 NdisFIndicateReceiveNetBufferLists 来指示发送数据。
如果 Filter Driver 设置了 NdisFIndicateReceiveNetBufferLists 的状态为 NDIS_STATUS_SUCCESS, NDIS 通过驱动的 FilterReturnNetBufferLists 返回指示数据。
如果 Filter Driver 设置了 NdisFIndicateReceiveNetBufferLists 的 ReceiveFlags 值为
NDIS_RECEIVE_FLAGS_RESOURCES,那么在函数返回后 Filter Driver 会立即恢复对
NET_BUFFER_LIST 的所有权,这时 Filter Driver 必须立即处理这个 NET_BUFFER_LIST 的返回。
在这种情况下是不会调用 FilterReturnNetBufferLists 返回 NET_BUFFER_LIST 结构的。
*****************************************************************************************/
        NdisFIndicateReceiveNetBufferLists(
                   pFilter->FilterHandle,
                   NetBufferLists,
                   PortNumber,
                   NumberOfNetBufferLists,
                   ReceiveFlags);
        if (NDIS_TEST_RECEIVE_CANNOT_PEND(ReceiveFlags) && pFilter->TrackReceives)
        {
            FILTER_ACQUIRE_LOCK(&pFilter->Lock, DispatchLevel);
            pFilter->OutstandingRcvs -= NumberOfNetBufferLists;
            Ref = pFilter->OutstandingRcvs;
            FILTER_LOG_RCV_REF(2, pFilter, NetBufferLists, Ref);
            FILTER_RELEASE_LOCK(&pFilter->Lock, DispatchLevel);
        }
    } while (bFalse);
    DEBUGP(DL_TRACE, ("<===ReceiveNetBufferList: Flags = %8x.\n", ReceiveFlags));
}
```

### `FilterCancelSendNetBufferLists` 函数的功能

过滤驱动调用 `NDIS_SET_NET_BUFFER_LIST_CANCEL_ID` 宏为每一个 `NET_BUFFER_LIST` 标记一个取消 Id。

在为网络数据分配取消 ID 之前,必须先调用 `NdisGenratePartialCanceId` 获得取消 ID 的高字节。

这是为了确保不是驱动不会把一个取消 ID 分配给两个驱动驱动通常在 `DriverEntry` 调用 `NdisGenratePartialCanceld`,但是驱动可以在不同的时间多次调用它来获得多个取消 ID。

要取消被标记过取消 ID 且正在传输的数据,驱动可以调用 `NdisFCancelSendNetBufferLists` 例程来完成。要获得取消 ID 可以用 `NDIS_GET_NET_BUFFER_LIST_CANCEL_ID` 宏来完成。

如果一个 `Filter Driver` 对所有发送的 `NET_BUFFER_LIST` 标记了相同的取消 ID 那它可以用一个 `NdisFCancelSendNetBufferLists` 来取消所有的发送请求。如果把一部发送请求的 `NET_BUFFER_LIST` 标记相同的取消 ID 那么就可以调用一次 `NdisFCancelSendNetBufferLists` 来取消这部分发送请求。

在实现这个功能时 `NDIS` 会调用下层驱动的取消发送功能。中断正在执行的发送任务后,下层驱动会调用发送完成全程 (如: `NdisMSendNetBufferListComplete`) 返回指定的 `NET_BUFFER_LIST` 结构并指定返回状态为 `NDIS_STATUS_CANCELLED`, `NDIS` 依次调用 `Filter Driver` 的 `FilterSendNetBufferListsComplete` 例程。在 `FilterSendNetBufferListsComplete` 中要用 `NDIS_SET_NET_BUFFER_LIST_CANCEL_ID` 设置取消的 `NET_BUFFER_LIST` 的取消 ID 为 `NULL`,这样是为了防止这个 ID,在 `NET_BUFFER_LIST` 被再次分配时使用。

上层驱动在取消一个未完成的发送请求时也必须对这个发送请求的 `NET_BUFFER_LIST` 结构设定取消 ID。

`NDIS` 会传递给 `Filter Driver` 的 `FilterCancelSendNetBufferLists` 一个取消 ID 来取消发送请求的 `NET_BUFFER_LIST` 发送。`FilterCanCelSendNetBufferLists` 下执行下列操作。
