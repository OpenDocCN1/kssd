
### NDIS_OID 请求处理

`Filter Driver` 可以从 `FilterOidRequest` 同步和异步完成一个 OID 请求，分别返回 `NDIS_STATUS_SUCCESS` 和 `NDIS_STATUS_PENDING` 即可。`FilterOidRequest` 可以用同步的直接完成一个 OID 请求并返回一个错误状态。

如果 `FilterOidRequest` 返回 `NDIS_STATUS_PENDING`，就必须在 OID 请求完成后调用 `NdisFOidRequestComplete` 来通知上层驱动请求完成。在这种情况下，请求的结果通过 `NdisFOidRequestComplete` 的 `OidRequest` 参数返回给上层驱动，并通过 `Status` 参数返回请求完成的最终状态。

如果 `FilterOidRequest` 返回 `NDIS_STATUS_SUCCESS`，通过 `FilterOidRequest` 的 `OidRequest` 参数返回一个查询结果到上层。这时不调用 `NdisFOidRequestComplete` 例程。

要转发 OID 请求到下层驱动，`Filter Driver` 必须调用 `NdisFOidRequest`。如果一个 OID 请求不能被转发到下层驱动应该当立即返回。要完成一个请求且不转发可以直接返回 `NDIS_STATUS_SUCCESS` 或其它错误状态或返回 `NDIS_STATUS_PENDING` 后调用 `NdisFOidRequestComplete`。

如果 `NdisFOidRequest` 返回 `NDIS_STATUS_PENDING`，`NDIS` 在 OID 请求完成后调用 `FilterOidRequestComplete` 来通知请求完成。在这种情况下，请求的结果通过 `NdisFOidRequestComplete` 的 `OidRequest` 参数返回给上层驱动，并通过 `Status` 参数返回请求完成的最终状态。

如果 `NdisFOidRequest` 返回 `NDIS_STATUS_SUCCESS`，通过 `NdisFOidRequest` 的 `OidRequest` 参数返回一个查询结果到上层。这时不调用 `FilterOidRequestComplete` 例程。

一个 `Filter Driver` 可以调用 `NdisFOidRequest` 引发 OID 请求在 `Restarting`、`Running`、`Pausing` 和 `Paused` 状态。

**注意**：`Filter Driver` 必须跟踪这个请求确保不在 `FilterOidRequestComplete` 中调用 `NdisFOidRequestComplete`（因为请求是自己引发的不能传到上层）。

### 代码示例

以下代码展示了 `FilterOidRequest`、`FilterCancelOidRequest` 和 `FilterOidRequestComplete` 的实现逻辑：

```c
/**
 * FilterOidRequest
 */
NDIS_STATUS
FilterOidRequest(IN  NDIS_HANDLE    FilterModuleContext,
                 IN  PNDIS_OID_REQUEST   Request)
{
    PMS_FILTER              pFilter = (PMS_FILTER)FilterModuleContext;
    NDIS_STATUS             Status;
    PNDIS_OID_REQUEST       ClonedRequest = NULL;
    BOOLEAN                 bSubmitted = FALSE;
    PFILTER_REQUEST_CONTEXT Context;
    BOOLEAN                 bFalse = FALSE;

    DEBUGP(DL_TRACE, ("===>FilterOidRequest: Request %p.\n", Request));

    do
    {
        Status = NdisAllocateCloneOidRequest(pFilter->FilterHandle,
                                              Request,
                                              FILTER_TAG,
                                              &ClonedRequest);
        if (Status != NDIS_STATUS_SUCCESS)
        {
            DEBUGP(DL_WARN, ("FilerOidRequest: Cannot Clone Request\n"));
            break;
        }

        Context = (PFILTER_REQUEST_CONTEXT)(&ClonedRequest->SourceReserved[0]);
        *Context = Request;
        bSubmitted = TRUE;
        ClonedRequest->RequestId = Request->RequestId;
        pFilter->PendingOidRequest = ClonedRequest;

        // Filter Driver 可以调用 NdisFOidRequest 引发一个 OID 查询和设置请求给下层驱动。
        Status = NdisFOidRequest(pFilter->FilterHandle, ClonedRequest);
        if (Status != NDIS_STATUS_PENDING)
        {
            FilterOidRequestComplete(pFilter, ClonedRequest, Status);
            Status = NDIS_STATUS_PENDING;
        }
    } while (bFalse);

    if (bSubmitted == FALSE)
    {
        switch (Request->RequestType)
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
}

/**
 * FilterCancelOidRequest
 *
 * NDIS 调用 FilterCancelOidRequest 来取消一个 OID 请求, 当 NDIS 调用 FilterCancelOidRequest 时,
 * Filter Driver 应该尽可能快的调用 NdisFCancelOidRequest。
 */
VOID
FilterCancelOidRequest(
    IN  NDIS_HANDLE         FilterModuleContext,
    IN  PVOID              RequestId
    )
{
    PMS_FILTER               pFilter = (PMS_FILTER)FilterModuleContext;
    PNDIS_OID_REQUEST        Request = NULL;
    PFILTER_REQUEST_CONTEXT  Context;
    PNDIS_OID_REQUEST        OriginalRequest = NULL;
    BOOLEAN                  bFalse = FALSE;

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

VOID
FilterOidRequestComplete(
        IN  NDIS_HANDLE        FilterModuleContext,
        IN  PNDIS_OID_REQUEST  Request,
        IN  NDIS_STATUS        Status
        )
{
    PMS_FILTER               pFilter = (PMS_FILTER)FilterModuleContext;
    PNDIS_OID_REQUEST        OriginalRequest;
    PFILTER_REQUEST_CONTEXT  Context;
    BOOLEAN                  bFalse = FALSE;

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

    switch (Request->RequestType)
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
}
```

### 说明

如果 `FilterOidRequest` 返回 `NDIS_STATUS_PENDING`，就必须在 OID 请求完成后调用 `NdisFOidRequestComplete` 来通知上层驱动请求完成。在这种情况下，请求的结果通过 `NdisFOidRequestComplete` 的 `OidRequest` 参数返回给上层驱动，并通过 `Status` 参数返回请求完成的最终状态。

要转发 OID 请求到下层驱动，`Filter Driver` 必须调用 `NdisFOidRequest`。

如果一个 OID 请求不能被转发到下层驱动应该当立即返回。
