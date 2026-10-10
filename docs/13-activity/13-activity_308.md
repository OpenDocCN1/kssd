

# 大牛们都发代码了，我也来凑凑热闹吧 - 第二题

> 原文：[`www.pediy.com/kssd/pediy10/79028.html`](https://www.pediy.com/kssd/pediy10/79028.html)

大牛们都发代码了，我也来凑凑热闹吧~  
3，4，5 题没什么意思了，发下第二题的关键部分吧~  
处理 ndisQueryStatisticsOids 细致一点就可以很好的解决其他思路的缺陷了~  

代码:

```
NTSTATUS 
Fake\_ndisQueryStatisticsOids(PVOID argv1, 
  PVOID argv2, 
  ULONG Request\[\], 
  ULONG RequestLen, 
  PVOID MdlAddress, 
  SIZE\_T NumberOfBytes, 
  PIRP AssociatedIrp, 
  PBOOLEAN Success)
{
  NTSTATUS status;
  PVOID Process;
  ULONG i, type;
  PNetResponse Response, gPrev;
  PCHAR pCResponse;

  pCResponse = (PCHAR)MdlAddress;
  Process = (PVOID)PsGetCurrentProcess();

  if (0 == \_strnicmp(MAKE\_OFFSET(Process, Offset\_ImageFileName), "explorer.exe", 11) && NULL != MdlAddress)
  {
    /\*First, we copy a statistics\*/
    if (NULL == pPreviousBuffer)
    {
      status = Real\_ndisQueryStatisticsOids(argv1, argv2, Request, RequestLen, MdlAddress, NumberOfBytes, AssociatedIrp, Success);
      if (!NT\_SUCCESS(status))
        return status;

      pPreviousBuffer = ExAllocatePool(NonPagedPool, NumberOfBytes);
      if (NULL == pPreviousBuffer)
        return STATUS\_INSUFFICIENT\_RESOURCES;
      RtlCopyMemory(pPreviousBuffer, MdlAddress, NumberOfBytes);
    }
    else
    {
      status = Real\_ndisQueryStatisticsOids(argv1, argv2, Request, RequestLen, MdlAddress, NumberOfBytes, AssociatedIrp, Success);
      if (!NT\_SUCCESS(status))
        return status;
      
      /\*We replace some updated data here using previous statistics\*/
      i = 0;
      while (i < RequestLen - 1)
      {
        Response = (PNetResponse)pCResponse;
        type = Response->Type & 0x0FFFFFFF;

        /\*we should replace XMIT & RCV Statistics data\*/
        if (type > 0x00020000 && type < 0x0002020D)
        {
          gPrev = (PNetResponse)((ULONG)pPreviousBuffer + (ULONG)(Response) - (ULONG)MdlAddress);
          RtlCopyMemory(&Response->Data, &gPrev->Data, Response->Length);
        }
        i++;
        pCResponse += Response->Length + 8;

        /\*Some stupid input? \*/
        if (Response->Length > 8)
          break;
      }
    }
    return status;
  }
  else
    return Real\_ndisQueryStatisticsOids(argv1, argv2, Request, RequestLen, MdlAddress, NumberOfBytes, AssociatedIrp, Success);
}

```