
```
// Allocate TAP packet memory
tapPacket = (PTAP_PACKET )NdisAllocateMemoryWithTagPriority(
Adapter->MiniportAdapterHandle,
TAP_PACKET_SIZE (packetLength+addHeaderSize),
TAP_PACKET_TAG,
NormalPoolPriority
);
// 提取Buf数据
packetData = NdisGetDataBuffer(NetBuffer,packetLength,tapPacket->m_Data+addHeaderSize,1,0);
// 拷贝数据到tapPacket+addHeaderSize后的位置，预留出来addHeaderSize
if(packetData != (tapPacket->m_Data+addHeaderSize))
{
// Packet data was contiguous and not yet copied to m_Data.
NdisMoveMemory(tapPacket->m_Data+addHeaderSize,packetData,packetLength);
}
// 填充addHeaderSize大小数据
if(addHeaderSize > 0)
{
// Add an 802.1Q header between the ethernet header and the payload
NdisMoveMemory(tapPacket->m_Data,tapPacket->m_Data+addHeaderSize,ETHERNET_HEADER_SIZE-2);
PETH_HEADER header = (PETH_HEADER)tapPacket->m_Data;
PETH_8021Q_HEADER tag = (PETH_8021Q_HEADER)(header+1);
header->proto = htons(0x8100);
USHORT tagValue = 0;
tagValue |= packetPriority.TagHeader.UserPriority<<13;
tagValue |= packetPriority.TagHeader.VlanId & 0xFFF;
tag->Tag = tagValue;
packetLength += addHeaderSize;
}
// DHCP的处理从数据链路层(ETH_HEADER) 到 网络层(IP_HDR) 到 UDPHDR(传输层) DHCP
......
const ETH_HEADER *eth = (ETH_HEADER *) tapPacket->m_Data;
const IPHDR *ip = (IPHDR *) (tapPacket->m_Data + sizeof (ETH_HEADER));
const UDPHDR *udp = (UDPHDR *) (tapPacket->m_Data + sizeof (ETH_HEADER) + sizeof (IPHDR));
......
else if (packetLength >= sizeof (ETH_HEADER) + sizeof (IPHDR) + sizeof (UDPHDR) + sizeof (DHCP)
&& eth->proto == htons (NDIS_ETH_TYPE_IPV4)
&& ip->version_len == 0x45 // IPv4, 20 byte header
&& ip->protocol == IPPROTO_UDP
&& udp->dest == htons (BOOTPS_PORT)
)
......
// 首先要从Ethernet->proto确认协议
ETH_HEADER *e;
e = (ETH_HEADER *) tapPacket->m_Data;
switch (ntohs (e->proto))
// ARP处理
if (packetLength != sizeof (ARP_PACKET))
{
goto no_queue;
}
ProcessARP (
Adapter,
(PARP_PACKET) tapPacket->m_Data,
Adapter->m_localIP,
Adapter->m_remoteNetwork,
Adapter->m_remoteNetmask,
Adapter->m_TapToUser.dest
);
// ipv4/ipv6处理
case NDIS_ETH_TYPE_IPV4:
// Make sure that packet is large enough to be IPv4.
if (packetLength < (ETHERNET_HEADER_SIZE + IP_HEADER_SIZE))
{
goto no_queue;
}
// Only accept directed packets, not broadcasts.
if (memcmp (e, &Adapter->m_TapToUser, ETHERNET_HEADER_SIZE))
{
goto no_queue;
}
// Packet looks like IPv4, queue it. :-)
tapPacket->m_SizeFlags |= TP_TUN;
break;
case NDIS_ETH_TYPE_IPV6:
// Make sure that packet is large enough to be IPv6.
if (packetLength < (ETHERNET_HEADER_SIZE + IPV6_HEADER_SIZE))
{
goto no_queue;
}
// Broadcasts and multicasts are handled specially
// (to be implemented)
// Neighbor discovery packets to fe80::8 are special
// OpenVPN sets this next-hop to signal "handled by tapdrv"
if ( HandleIPv6NeighborDiscovery(Adapter,tapPacket->m_Data,
packetLength) )
{
goto no_queue;
}
// Packet looks like IPv6, queue it. :-)
tapPacket->m_SizeFlags |= TP_TUN;
}
```

6) ipv6处理时候会检测fe80::xxx，这类的本地单播不处理，有可能是自动配置地址，邻居发现等等。

```
static IPV6ADDR IPV6_NS_TARGET_MCAST =
{ 0xff, 0x02, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
0x00, 0x00, 0x00, 0x01, 0xff, 0x00, 0x00, 0x08 };
static IPV6ADDR IPV6_NS_TARGET_UNICAST =
{ 0xfe, 0x80, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x08 };
if ( memcmp( ipv6->daddr, IPV6_NS_TARGET_MCAST,
sizeof(IPV6ADDR) ) != 0 &&
memcmp( ipv6->daddr, IPV6_NS_TARGET_UNICAST,
sizeof(IPV6ADDR) ) != 0 )
{
return FALSE; // wrong target address
}
// ICMPv6 type+code must be 135/0 for NS
if ( icmpv6_ns->type != ICMPV6_TYPE_NS ||
icmpv6_ns->code != ICMPV6_CODE_0 )
{
return FALSE; // wrong ICMPv6 type
}
// 需要计算和填充checksum
icmpv6_csum = icmpv6_checksum (
(UCHAR*) &(na->icmpv6),
icmpv6_len,
na->ipv6.saddr,
na->ipv6.daddr
);
```

最后调用`tapProcessSendPacketQueue`完成。

7) 应用层拿到数据包以后代理后，代理回包如何处理(`recv`)？这就和`Write`操作有关`dispatchTable[IRP_MJ_WRITE] = TapDeviceWrite;`

```
......
......
// 拿到应用层传递过来的数据包
unsigned char* packetBuffer = (unsigned char *) Irp->AssociatedIrp.SystemBuffer;
ULONG packetLength = irpSp->Parameters.Write.Length;
PVOID packetPriority = 0;
DUMP_PACKET ("IRP_MJ_WRITE ETH",
packetBuffer,
packetLength);
// 8021Q处理
packetPriority = TapStrip8021Q(&packetBuffer, &packetLength);
// 发送包到虚拟网卡
ntStatus = TapSharedSendPacket(
adapter,
Irp,
packetBuffer,
packetLength,
packetPriority,
NULL,
0
);
```

8) `TapSharedSendPacket`

```
// 小于60byte拷贝副本在MDL映射到NetBufferList
// Copy packet data to flat buffer.
......
if(PrefixLength > 0)
{
NdisMoveMemory(allocBuffer, PrefixData, PrefixLength);
}
NdisMoveMemory (allocBuffer + PrefixLength, PacketBuffer, PacketLength);
NdisZeroMemory(allocBuffer + fullLength, paddedLength - fullLength);
......
// 标记注入包
TAP_RX_NBL_FLAGS_CLEAR_ALL(netBufferList);
TAP_RX_NBL_FLAG_SET(netBufferList,TAP_RX_NBL_FLAGS_IS_INJECTED);
// 设置pending
IoMarkIrpPending(Irp);
IoSetCancelRoutine(Irp,NULL);
// Stash IRP pointer in NBL MiniportReserved[0] field.
netBufferList->MiniportReserved[0] = Irp;
netBufferList->MiniportReserved[1] = NULL;
NET_BUFFER_LIST_INFO(netBufferList, Ieee8021QNetBufferListInfo) = PacketPriority;
// Increment in-flight receive NBL count.
nblCount = NdisInterlockedIncrement(&Adapter->ReceiveNblInFlightCount);
ASSERT(nblCount > 0 );
//
// Indicate the packet
// -------------------
// This NBL contains the complete packet including Ethernet header and payload.
//
// 完整的数据调用ReceiveNet完成包
NdisMIndicateReceiveNetBufferLists(
Adapter->MiniportAdapterHandle,
netBufferList,
NDIS_DEFAULT_PORT_NUMBER,
1, // NumberOfNetBufferLists
0 // 清除所有标志
);
// 注意返回标志是Pending
return STATUS_PENDING;
```

9) tap 5.0 Recv NBL实现和6.0有些异同，直接发送：

```
if (!l_Adapter->m_tun && ((l_IrpSp->Parameters.Write.Length) >= ETHERNET_HEADER_SIZE))
{
......
NdisMEthIndicateReceive
(l_Adapter->m_MiniportAdapterHandle,
(NDIS_HANDLE)l_Adapter,
(unsigned char*)p_IRP->AssociatedIrp.SystemBuffer,
ETHERNET_HEADER_SIZE,
(unsigned char*)p_IRP->AssociatedIrp.SystemBuffer + ETHERNET_HEADER_SIZE,
l_IrpSp->Parameters.Write.Length - ETHERNET_HEADER_SIZE,
l_IrpSp->Parameters.Write.Length - ETHERNET_HEADER_SIZE);
NdisMEthIndicateReceiveComplete();
p_IRP->IoStatus.Status = l_Status = STATUS_SUCCESS;
......
}
else if (l_Adapter->m_tun && ((l_IrpSp->Parameters.Write.Length) >= IP_HEADER_SIZE))
{
......
if (IPH_GET_VER(((IPHDR*)p_IRP->AssociatedIrp.SystemBuffer)->version_len) == 6)
{
p_UserToTap = &l_Adapter->m_UserToTap_IPv6;
}
......
NdisMEthIndicateReceive
(l_Adapter->m_MiniportAdapterHandle,
(NDIS_HANDLE)l_Adapter,
(unsigned char*)p_UserToTap,
sizeof(ETH_HEADER),
(unsigned char*)p_IRP->AssociatedIrp.SystemBuffer,
l_IrpSp->Parameters.Write.Length,
l_IrpSp->Parameters.Write.Length);
NdisMEthIndicateReceiveComplete(l_Adapter->m_MiniportAdapterHandle);
p_IRP->IoStatus.Status = l_Status = STATUS_SUCCESS;
......
}
```

`NdisMIndicateReceiveNetBufferLists` NDIS6.0或更高版本才可以使用，过程并不复杂，不过以前自己做NDIS轮子遇到不少麻烦，后来在做类似项目就直接用开源的驱动。

## WireGuard NT:

**这里并不是讲WirGuard NT本身如何做隧道的，而是通过假设代理方案举例：**

1. 初始化WireGuard驱动，注册小端生成虚拟网卡。
2. 应用层设置路由表和虚拟网卡，指定IP路由到虚拟网卡。
3. NDIS捕获数据包，内核`WSK`代理双向连接。

**WireGuard NT梳理基于`SendNetBufferLists`/`ReturnNetBufferLists` `WSK` UDP处理主线**

### 数据包流动实现:

#### 应用层:

1) 两个阶段：

第一个是通过`WireGuardSetConfiguration`-->`DevIceIoControl` `WG_IOCTL_SET` 发送 `WIREGUARD_INTERFACE_HAS_PRIVATE_KEY`,这个不做讨论。

第二个是通过`WireGuardSetAdapterState`-->`DevIceIoControl` `WG_IOCTL_SET_ADAPTER_STATE` 发送 `WIREGUARD_ADAPTER_STATE_UP`。
