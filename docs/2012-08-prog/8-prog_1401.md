
# NDIS6 防火墙的 BUG 通告

> 原文：[`www.pediy.com/kssd/pediy12/143945.html`](https://www.pediy.com/kssd/pediy12/143945.html)

原帖见：

[`bbs.pediy.com/showthread.php?t=143756`](http://bbs.pediy.com/showthread.php?t=143756)

朋友报了一个 `miniDMP`，崩溃堆栈如下:

```
badb0d00 8757f9a8 00000000 nt!KiTrap0E+0x2cf (FPO: [0,0] TrapFrame @ a794e278)
875e9448 00000000 00000000 vwififlt!FilterSendNetBufferListsComplete+0x66 (FPO: [3,0,4])
874450e0 8757f9a8 00000000 ndis!NdisMSendNetBufferListsComplete+0xa4 (FPO: [3,1,4])
874450e0 8757f9a8 00000000 XFireWall!HookSendNetBufferLists+0x9b (FPO: [Non-Fpo]) (CONV: stdcall)
87f17798 8757f9a8 00000000 ndis!NdisSendNetBufferLists+0x162 (FPO: [4,3,4])
87f17cf8 00000000 00000000 tcpip!FlSendPackets+0x416 (FPO: [3,10,4])
8a56ada0 00000000 00000000 tcpip!IppFragmentPackets+0x2e2 (FPO: [2,13,4])
8a56ada0 870a2544 870a25e0 tcpip!IppDispatchSendPacketHelper+0x266 (FPO: [2,7,4])
000a2544 00000000 8672be40 tcpip!IppPacketizeDatagrams+0x8d6 (FPO: [1,33,4])
00000000 00000004 8a56ada0 tcpip!IppSendDatagramsCommon+0x652 (FPO: [5,26,4])
870af8f8 a794e614 00000000 tcpip!IpNlpSendDatagrams+0x4b (FPO: [2,0,4])
00000000 00000000 866fc970 tcpip!UdpSendMessagesOnPathCreation+0x7c0 (FPO: [15,112,4])
870af970 0094e940 00000000 tcpip!UdpSendMessages+0x595 (FPO: [2,45,4])
a794e90c b915b771 00000000 tcpip!UdpTlProviderSendMessagesCalloutRoutine+0x13 (FPO: [1,0,4])
8a4e5882 a794e90c 00000000 nt!KeExpandKernelStackAndCalloutEx+0x132
866fc970 a794e940 8672a4c0 tcpip!UdpTlProviderSendMessages+0x67 (FPO: [2,6,4])
867d8628 8672a400 864e7050 tdx!TdxSendDatagramTransportAddress+0x204 (FPO: [3,12,4])
87275b58 8672a4c0 87151dc8 tdx!TdxTdiDispatchInternalDeviceControl+0x5c (FPO: [2,0,0])
00000000 a794e9d8 8a60dd98 nt!IofCallDriver+0x63
WARNING: Stack unwind information not available. Following frames may be wrong.
87275b58 8672a4c0 871d1000 kmodurl+0x14b5
```

`IDA` 反汇编 `vwififlt!FilterSendNetBufferListsComplete+0x66`：

```c
v6 = v4->ProtocolReserved[0];
v5 = v4->ProtocolReserved[1];
v14 = v4->_union_1._struc_2.Next;
if ( !v6 || !v5 )                 // !!!!ERROR!!!!!!!!!!!!! <----这里错误
  TraceAssert("pFilterMpCtx && OriginalNbl", "d:\\w7rtm\\net\\vwifi\\filter\\filter.c", 2619);
```

原因如下：

原因 `vwififlt!FilterSendNetBufferListsComplete+0x66` 在其 `FilterSendNetBufferLists` 未被调用的情况下执行了 `vwififlt!FilterSendNetBufferListsComplete`，使得该驱动认为数据包非法。

这个 `flt` 驱动居然不 pass 掉，直接 `BSOD`，靠！！！

下面是我测试的 `flt` 驱动的 `FilterSendNetBufferLists` 命中堆栈：

```
862356c8 868d47e8 00000000 ndislwf!FilterSendNetBufferLists+0x39
868d47e8 868d47e8 00000000 ndis!ndisFilterSendNetBufferLists+0x87
8683e820 868d47e8 00000000 ndis!NdisFSendNetBufferLists+0x38
86845430 868d47e8 00000000 pacer!PcFilterSendNetBufferLists+0x256
8629a0e0 868d47e8 00000000 ndis!ndisSendNBLToFilter+0xf2  《-----关键！！！！
85479598 868d47e8 00000000 ndis!NdisSendNetBufferLists+0x162  《-----这里开始
854bdcf8 00000000 00000000 tcpip!FlSendPackets+0x416
88cf6fa8 00000000 00000000 tcpip!IppFragmentPackets+0x2e2
88cf6fa8 860b9d54 860b9df0 tcpip!IppDispatchSendPacketHelper+0x266
000b9d54 86c65128 86322318 tcpip!IppPacketizeDatagrams+0x8d6
00000000 8cc2fa00 88cf6fa8 tcpip!IppSendDatagramsCommon+0x652
88cf6fa8 8cc2fa84 86c651ec tcpip!IppSendDatagrams+0x2a
86c65128 86df48d0 00000001 tcpip!IppProcessMulticastDiscoveryTimeoutEvents+0x345
8608fa28 86c65128 853cb020 tcpip!IppMulticastWorkerRoutine+0x43
86df14e8 00000000 853cb020 nt!IopProcessWorkItem+0x23
00000001 59c38d03 00000000 nt!ExpWorkerThread+0x10d
83c9c076 00000001 00000000 nt!PspSystemThreadStartup+0x9e
00000000 00000000 00000000 nt!KiThreadStartup+0x19
```

协议驱动最终会调用 `ndis!NdisFSendNetBufferLists` 发送数据包。

直接 hook `ndis!NdisFSendNetBufferLists` ？？？该函数导出！！！

```c
int __stdcall NdisFSendNetBufferLists(_NDIS_FILTER_BLOCK *NdisFilterHandle, _NET_BUFFER_LIST *nbl, int a3, int a4)
{
  _NET_BUFFER_LIST *v4; // eax@1
  int v5; // ecx@1
  v5 = NdisFilterHandle->NextSendNetBufferListsObject;
  v4 = nbl;
  if ( ndisTrackNblOwner )
  {
    while ( v4 )
    {
      v4->NetBufferListInfo[18] = v5;
      v4 = (_NET_BUFFER_LIST *)v4->_union_1._struc_2.Next;
    }
  }
  return ((int (__stdcall *)(_DWORD, _DWORD, _DWORD, _DWORD))NdisFilterHandle->FilterSendNetBufferListsHandler)(
           NdisFilterHandle,
           nbl,
           a3,
           a4);
}
```

该指针指向函数 `ndis!ndisFilterSendNetBufferLists`，会继续发往下一个！！比较麻烦。

最终调用 `ndis!ndisMSendNBLToMiniport` 发送给 `miniport`。

```c
void *__stdcall ndisMSendNBLToMiniport(_NDIS_MINIPORT_BLOCK *a1, _NET_BUFFER_LIST *nbl, int a3, int a4)
{
  void *result; // eax@1
  _NDIS_MINIPORT_BLOCK *v5; // edi@1
  int v6; // eax@1
  int v7; // ecx@5
  int v8; // ecx@12
  int v9; // ecx@16
  int v10; // eax@18
  int v11; // esi@18
  int v12; // edx@18
  int v13; // ecx@21
  char v14; // zf@22
  int v15; // ecx@24
  int v16; // ebx@24
  __int64 v17; // qax@24
  int v18; // [sp+8h] [bp-10h]@1
  int v19; // [sp+Ch] [bp-Ch]@1
  _NET_BUFFER_LIST *v20; // [sp+10h] [bp-8h]@1
  char v21; // [sp+17h] [bp-1h]@1
  int v22; // [sp+2Ch] [bp+14h]@22
  v5 = a1;
  v6 = a1->DriverHandle->_union_15.MiniportDriverCharacteristics.SendNetBufferListsHandler;
```

因此在这里做一个替换即可 `MiniportDriverCharacteristics.SendNetBufferListsHandler`！！！

### 解决办法：

1、hook 点下移到 `MiniportDriverCharacteristics.SendNetBufferListsHandler`

2、直接调用 `ProtoSendNBLComplete` 完成得了！！！该指针可以直接从 `_ndis_open_block` 处获取，不过好像不通用

**深入研究点这里**：

[`www.opscn.com/index.php?actio...rum=1&topic=29`](http://www.opscn.com/index.php?action=vthread&forum=1&topic=29)
