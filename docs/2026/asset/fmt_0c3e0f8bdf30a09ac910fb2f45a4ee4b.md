
### OID 请求处理

```c
Context = (PFILTER_REQUEST_CONTEXT)(&ClonedRequest->SourceReserved[0]);

*Context = Request;

bSubmitted = TRUE;

ClonedRequest->RequestId = Request->RequestId;

pFilter->PendingOidRequest = ClonedRequest;

//Filter Driver可以调用NdisFOidRequest引发一个OID查询和设置请求给下层驱动。

Status = NdisFOidRequest(pFilter->FilterHandle, ClonedRequest);

if (Status != NDIS_STATUS_PENDING)
{
    FilterOidRequestComplete(pFilter, ClonedRequest, Status);
    Status = NDIS_STATUS_PENDING;
}
```while (bFalse);

if (bSubmitted == FALSE)
{
    switch(Request->RequestType)
    {
        case NdisRequestMethod:
            Request->DATA.METHOD_INFORMATION.BytesRead = 0;
            Request->DATA.METHOD_INFORMATION.BytesNeeded = 0;
            Request->DATA.METHOD_INFORMATION.BytesWritten = 0;
            break;
        case NdisRequestSetInformation:
            Request->DATA.SET_INFORMATION.BytesRead = 0;
            Request->DATA.SET_INFORMATION.BytesNeeded = 0;
            break;
        case NdisRequestQueryInformation:
        case NdisRequestQueryStatistics:
        default:
            Request->DATA.QUERY_INFORMATION.BytesWritten = 0;
            Request->DATA.QUERY_INFORMATION.BytesNeeded = 0;
            break;
    }
}

DEBUGP(DL_TRACE, ("<===FilterOidRequest: Status %8x.\n", Status));
return Status;
```

/*********************************************************************
FilterCancelOidRequest函数的功能:
  NDIS调用FilterCancelOidRequest来取消一个OID请求,当NDIS调用FilterCancelOidRequest时,
  Filter Driver应该尽可能快的调用NdisFCancelOidRequest。
*********************************************************************/

### FilterCancelOidRequest

```c
VOID
FilterCancelOidRequest(
    IN  NDIS_HANDLE             FilterModuleContext,
    IN  PVOID                   RequestId
    )
{
    PMS_FILTER                          pFilter = (PMS_FILTER)FilterModuleContext;
    PNDIS_OID_REQUEST                   Request = NULL;
    PFILTER_REQUEST_CONTEXT             Context;
    PNDIS_OID_REQUEST                   OriginalRequest = NULL;
    BOOLEAN               bFalse = FALSE;
    FILTER_ACQUIRE_LOCK(&pFilter->Lock, bFalse);
    Request = pFilter->PendingOidRequest;
    if (Request != NULL)
    {
        Context = (PFILTER_REQUEST_CONTEXT)(&Request->SourceReserved[0]);
        OriginalRequest = (*Context);
    }
    if ((OriginalRequest != NULL) && (OriginalRequest->RequestId == RequestId))
    {
        FILTER_RELEASE_LOCK(&pFilter->Lock, bFalse);
        NdisFCancelOidRequest(pFilter->FilterHandle, RequestId);
    }
    else
    {
        FILTER_RELEASE_LOCK(&pFilter->Lock, bFalse);
    }
}
```

### FilterOidRequestComplete

```c
VOID
FilterOidRequestComplete(
        IN  NDIS_HANDLE         FilterModuleContext,
        IN  PNDIS_OID_REQUEST   Request,
        IN  NDIS_STATUS         Status
        )
{
    PMS_FILTER                          pFilter = (PMS_FILTER)FilterModuleContext;
    PNDIS_OID_REQUEST                   OriginalRequest;
    PFILTER_REQUEST_CONTEXT             Context;
    BOOLEAN               bFalse = FALSE;
    DEBUGP(DL_TRACE, ("===>FilterOidRequestComplete, Request %p.\n", Request));
    Context = (PFILTER_REQUEST_CONTEXT)(&Request->SourceReserved[0]);
    OriginalRequest = (*Context);
    if (OriginalRequest == NULL)
    {
        filterInternalRequestComplete(pFilter, Request, Status);
        return;
    }
    FILTER_ACQUIRE_LOCK(&pFilter->Lock, bFalse);
    ASSERT(pFilter->PendingOidRequest == Request);
    pFilter->PendingOidRequest = NULL;
    FILTER_RELEASE_LOCK(&pFilter->Lock, bFalse);
    switch(Request->RequestType)
    {
        case NdisRequestMethod:
            OriginalRequest->DATA.METHOD_INFORMATION.OutputBufferLength =  Request->DATA.METHOD_INFORMATION.OutputBufferLength;
            OriginalRequest->DATA.METHOD_INFORMATION.BytesRead = Request->DATA.METHOD_INFORMATION.BytesRead;
            OriginalRequest->DATA.METHOD_INFORMATION.BytesNeeded = Request->DATA.METHOD_INFORMATION.BytesNeeded;
            OriginalRequest->DATA.METHOD_INFORMATION.BytesWritten = Request->DATA.METHOD_INFORMATION.BytesWritten;
            break;
        case NdisRequestSetInformation:
            OriginalRequest->DATA.SET_INFORMATION.BytesRead = Request->DATA.SET_INFORMATION.BytesRead;
            OriginalRequest->DATA.SET_INFORMATION.BytesNeeded = Request->DATA.SET_INFORMATION.BytesNeeded;
            break;
        case NdisRequestQueryInformation:
        case NdisRequestQueryStatistics:
        default:
            OriginalRequest->DATA.QUERY_INFORMATION.BytesWritten = Request->DATA.QUERY_INFORMATION.BytesWritten;
            OriginalRequest->DATA.QUERY_INFORMATION.BytesNeeded = Request->DATA.QUERY_INFORMATION.BytesNeeded;
            break;
    }
    (*Context) = NULL;
    NdisFreeCloneOidRequest(pFilter->FilterHandle, Request);
```

如果FilterOidRequest返回`NDIS_STATUS_PENDING`,就必须在OID请求完成后调用`NdisFOidRequestComplete` 来通知上层驱动求请求完成。在这种情况下,请求的结果通过`NdisFOidRequestComplete`的OidRequest参数返回给上层驱动,并通过Status参数返回请求完成的最终状态。

要转发OID请求到下层驱动,Filter Driver必须调用`NdisFOidRequest`。

如果一个OID请求不能被转发到下层驱动应该当立即返回。

要完成一个请求且不转发可以直接返回`NDIS_STATUS_SUCCESS`或其它错误状态或返回`NDIS_STATUS_PENDING`后调用`NdisFOidRequestComplete`。

```c
    NdisFOidRequestComplete(pFilter->FilterHandle, OriginalRequest, Status);
    DEBUGP(DL_TRACE, ("<===FilterOidRequestComplete.\n"));
}
```

/*********************************************************************
  FilterStatus函数的功能:
    当下层驱动报告状态的时候 NDIS会调用它。此外,Filter Driver还可以自己引发一个状态指示。

当下层驱动调用一个状态指示例程时(NdisMIndicateStatusEx或NdisFIndicateStats),NDIS
会调用Filter Driver的FilterStatus例程。

Filter Driver在FilterStatus中调用NdisFIndicateStatus传递一个状态指示给上层驱动。此外,还可以过滤状态指示(不用调用 NdisFIndicateStatus)或在调用 NdisFIndicateStatus之前修改状态信息。

Filter Driver要自己引发一个状态报告,可以在NDIS未调用 FilterStatus的情况下调用NdisFIndicateStatus。在这种情况下,Filter Driver要设置 SourceHandle 成员为 FilteAttech 参数提供的NdisFilterHandle句柄。

如果一个状态指示是一个OID请求相关的(下层请求一个 OID 下层要做相应的状态指示),那么状态的DestinationHandle和RequestId成员要设置成上层的OID请求包携带的数据。

Filter Driver调用NdisFIndicateStatus后NDIS会调用相邻上层的状态指示函数(ProtocolStatusEx或FilterStatus)。
*********************************************************************/

### FilterStatus

```c
VOID
FilterStatus(
        IN  NDIS_HANDLE             FilterModuleContext,
        IN  PNDIS_STATUS_INDICATION StatusIndication
        )
{
    PMS_FILTER              pFilter = (PMS_FILTER)FilterModuleContext;
#if DBG
    BOOLEAN                  bFalse = FALSE;
#endif
    DEBUGP(DL_TRACE, ("===>FilterStaus, IndicateStatus = %8x.\n", StatusIndication->StatusCode));
#if DBG
    FILTER_ACQUIRE_LOCK(&pFilter->Lock, bFalse);
    ASSERT(pFilter->bIndicating == FALSE);
    pFilter->bIndicating = TRUE;
    FILTER_RELEASE_LOCK(&pFilter->Lock, bFalse);
#endif
    NdisFIndicateStatus(pFilter->FilterHandle, StatusIndication);
#if DBG
    FILTER_ACQUIRE_LOCK(&pFilter->Lock, bFalse);
    ASSERT(pFilter->bIndicating == TRUE);
    pFilter->bIndicating = FALSE;
    FILTER_RELEASE_LOCK(&pFilter->Lock, bFalse);
#endif
    DEBUGP(DL_TRACE, ("<===FilterStaus.\n"));
}
```

/*
Filter Driver提供FilterPnpEventNotify来接收NDIS传递的PnP和电源管理事件
*/

### FilterDevicePnPEventNotify

```c
VOID
FilterDevicePnPEventNotify(
        IN  NDIS_HANDLE             FilterModuleContext,
        IN  PNET_DEVICE_PNP_EVENT   NetDevicePnPEvent
        )
{
    PMS_FILTER          pFilter = (PMS_FILTER)FilterModuleContext;
    NDIS_DEVICE_PNP_EVENT   DevicePnPEvent = NetDevicePnPEvent->DevicePnPEvent;
#if DBG
    BOOLEAN             bFalse = FALSE;
#endif
```
