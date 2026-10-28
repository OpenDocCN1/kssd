
1. 遍历 Filter Driver 的发送队列，用 `NDIS_GET_NET_BUFFER_LIST_CANCEL_ID` 获得队列中 `NET_BUFFER_LIST` 的取消 ID 与 `FilterCancelSendBufferLists` 的取消 ID 比较。

2. 移除队列中取消 ID 和 `FilterCancelSentBufferLists` 中取消 ID 相同的元素。

3. 调用 `NdisFSendNetBufferListsComplete` 来完成这些 `NET_BUFFER_LIST` 并设定返回状态为 `NDIS_STATUS_CANCELLED`。

4. 调用 `NdisFCancelSendNetBufferLists` 传递取消发送请求给下层驱动。传递取消 ID 给下层驱动，就像 Filter Driver 取消自己引发的发送请求一样。

```c
VOID
FilterCancelSendNetBufferLists(
    IN  NDIS_HANDLE             FilterModuleContext,
    IN  PVOID                   CancelId
    )
{
    PMS_FILTER  pFilter = (PMS_FILTER)FilterModuleContext;
    NdisFCancelSendNetBufferLists(pFilter->FilterHandle,CancelId);
}
```

`FilterSetModuleOptions` 函数的功能：

必须要在初始化时为驱动注册 `FilterSetModuleOptions` 例程，驱动可以在这个例程中初始化 `NDIS_FILTER_PARTIAL_CHARACTERISTICS` 结构来调用 `NdisSetOptionalHandlers` 来完成改变。这个例程如果存在，那么在调用 Filter Driver 的 `FilterRestart` 例程之前调用它。

```c
NDIS_STATUS
FilterSetModuleOptions(
    IN  NDIS_HANDLE             FilterModuleContext
    )
{
   PMS_FILTER                   pFilter = (PMS_FILTER)FilterModuleContext;
   NDIS_FILTER_PARTIAL_CHARACTERISTICS      OptionalHandlers;
   NDIS_STATUS                Status = NDIS_STATUS_SUCCESS;
   BOOLEAN               bFalse = FALSE;

   if (bFalse)
   {
       UINT    i;
       pFilter->CallsRestart++;
       i = pFilter->CallsRestart % 8;
       pFilter->TrackReceives = TRUE;
       pFilter->TrackSends = TRUE;

       NdisMoveMemory(&OptionalHandlers, &DefaultChars, sizeof(OptionalHandlers));
       OptionalHandlers.Header.Type = NDIS_OBJECT_TYPE_FILTER_PARTIAL_CHARACTERISTICS;
       OptionalHandlers.Header.Size = sizeof(OptionalHandlers);

       switch (i)
       {
            case 0:
                OptionalHandlers.ReceiveNetBufferListsHandler = NULL;
                pFilter->TrackReceives = FALSE;
                break;

            case 1:
                OptionalHandlers.ReturnNetBufferListsHandler = NULL;
                pFilter->TrackReceives = FALSE;
                break;

            case 2:
                OptionalHandlers.SendNetBufferListsHandler = NULL;
                pFilter->TrackSends = FALSE;
                break;

            case 3:
                OptionalHandlers.SendNetBufferListsCompleteHandler = NULL;
                pFilter->TrackSends = FALSE;
                break;

            case 4:
                OptionalHandlers.ReceiveNetBufferListsHandler = NULL;
                OptionalHandlers.ReturnNetBufferListsHandler = NULL;
                break;

            case 5:
                OptionalHandlers.SendNetBufferListsHandler = NULL;
                OptionalHandlers.SendNetBufferListsCompleteHandler = NULL;
                break;

            case 6:
                OptionalHandlers.ReceiveNetBufferListsHandler = NULL;
                OptionalHandlers.ReturnNetBufferListsHandler = NULL;
                OptionalHandlers.SendNetBufferListsHandler = NULL;
                OptionalHandlers.SendNetBufferListsCompleteHandler = NULL;
                break;

            case 7:
                break;
       }

       Status = NdisSetOptionalHandlers(pFilter->FilterHandle, (PNDIS_DRIVER_OPTIONAL_HANDLERS)&OptionalHandlers );
   }

   return Status;
}
```

```c
NDIS_STATUS
filterDoInternalRequest(
    IN PMS_FILTER                   FilterModuleContext,
    IN NDIS_REQUEST_TYPE            RequestType,
    IN NDIS_OID                     Oid,
    IN PVOID                        InformationBuffer,
    IN ULONG                        InformationBufferLength,
    IN ULONG                        OutputBufferLength, OPTIONAL
    IN ULONG                        MethodId, OPTIONAL
    OUT PULONG                      pBytesProcessed
    )
{
    FILTER_REQUEST              FilterRequest;
    PNDIS_OID_REQUEST           NdisRequest = &FilterRequest.Request;
    NDIS_STATUS                 Status;
    BOOLEAN               bFalse;

    bFalse = FALSE;

    NdisZeroMemory(NdisRequest, sizeof(NDIS_OID_REQUEST));
    NdisInitializeEvent(&FilterRequest.ReqEvent);

    NdisRequest->Header.Type = NDIS_OBJECT_TYPE_OID_REQUEST;
    NdisRequest->Header.Revision = NDIS_OID_REQUEST_REVISION_1;
    NdisRequest->Header.Size = sizeof(NDIS_OID_REQUEST);
    NdisRequest->RequestType = RequestType;

    switch (RequestType)
    {
        case NdisRequestQueryInformation:
             NdisRequest->DATA.QUERY_INFORMATION.Oid = Oid;
             NdisRequest->DATA.QUERY_INFORMATION.InformationBuffer =InformationBuffer;
             NdisRequest->DATA.QUERY_INFORMATION.InformationBufferLength = InformationBufferLength;
            break;

        case NdisRequestSetInformation:
             NdisRequest->DATA.SET_INFORMATION.Oid = Oid;
             NdisRequest->DATA.SET_INFORMATION.InformationBuffer =InformationBuffer;
             NdisRequest->DATA.SET_INFORMATION.InformationBufferLength =InformationBufferLength;
            break;

        case NdisRequestMethod:
             NdisRequest->DATA.METHOD_INFORMATION.Oid = Oid;
             NdisRequest->DATA.METHOD_INFORMATION.MethodId = MethodId;
             NdisRequest->DATA.METHOD_INFORMATION.InformationBuffer = InformationBuffer;
             NdisRequest->DATA.METHOD_INFORMATION.InputBufferLength = InformationBufferLength;
             NdisRequest->DATA.METHOD_INFORMATION.OutputBufferLength = OutputBufferLength;
             break;

        default:
            FILTER_ASSERT(bFalse);
            break;
    }

    NdisRequest->RequestId = (PVOID)FILTER_REQUEST_ID;

    Status = NdisFOidRequest(FilterModuleContext->FilterHandle,NdisRequest);

    if (Status == NDIS_STATUS_PENDING)
    {
        NdisWaitEvent(&FilterRequest.ReqEvent, 0);
        Status = FilterRequest.Status;
    }

    if (Status == NDIS_STATUS_SUCCESS)
    {
        if (RequestType == NdisRequestSetInformation)
        {
            *pBytesProcessed = NdisRequest->DATA.SET_INFORMATION.BytesRead;
        }

        if (RequestType == NdisRequestQueryInformation)
        {
            *pBytesProcessed = NdisRequest->DATA.QUERY_INFORMATION.BytesWritten;
        }

        if (RequestType == NdisRequestMethod)
        {
            *pBytesProcessed = NdisRequest->DATA.METHOD_INFORMATION.BytesWritten;
        }

        if (RequestType == NdisRequestMethod)
        {
            if (*pBytesProcessed > OutputBufferLength)
            {
                *pBytesProcessed = OutputBufferLength;
            }
        }

        else
        {
            if (*pBytesProcessed > InformationBufferLength)
            {
                *pBytesProcessed = InformationBufferLength;
            }
        }
    }

    return (Status);
}

VOID
filterInternalRequestComplete(
    IN NDIS_HANDLE                  FilterModuleContext,
    IN PNDIS_OID_REQUEST            NdisRequest,
    IN NDIS_STATUS                  Status
    )
{
    PFILTER_REQUEST              FilterRequest;

    UNREFERENCED_PARAMETER(FilterModuleContext);

    FilterRequest = CONTAINING_RECORD(NdisRequest, FILTER_REQUEST, Request);

    FilterRequest->Status = Status;

    NdisSetEvent(&FilterRequest->ReqEvent);
}
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
