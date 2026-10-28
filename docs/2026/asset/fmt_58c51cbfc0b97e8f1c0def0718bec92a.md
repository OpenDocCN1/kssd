
```
typedef struct _NDIS_SG_DMA_BLOCK {
    NDIS_OBJECT_HEADER Header;
    PVOID              Miniport;
    PDMA_ADAPTER       DmaAdapterObject;        // DMA 适配器
    PNPAGED_LOOKASIDE_LIST SGListLookasideList; // SG 列表 Lookaside
    ULONG              ScatterGatherListSize;
    // ...
} NDIS_SG_DMA_BLOCK;
```

通过强转`NDIS_HANDLE`为`PNDIS_MINIPORT_BLOCK_WIN7`，然后访问`MiniportSGDmaBlock`字段，就能拿到`DmaAdapterObject`。这个技巧在 DMA 支持中至关重要。

### `protocol.c/h`——协议绑定与发包管理

这是把 lwIP 的 `pbuf` 数据真正塞进网卡的关键模块。它负责三件事：初始化协议子系统、向 lwIP 注册网络接口、以及实现实际的发包路径。

先说初始化。`ProtocolInitializeSubsystem` 创建了两个 `NPAGED_LOOKASIDE_LIST`。Lookaside 列表是 Windows 内核提供的一种对象缓存机制——它预先分配好一批固定大小的对象放在列表里，分配的时候直接从列表头部取一个，释放的时候插回列表。比起每次都调 `ExAllocatePoolWithTag` 去走通用堆分配，Lookaside 列表要快得多，而且因为对象大小固定，不会产生堆碎片。这两个列表一个用于分配 `KIP_PACKET_DESCRIPTOR`（数据包描述符），一个用于分配 `MAX_ETHER_SIZE`（1514 字节）的数据缓冲。这两个结构在发包路径中被高频使用，所以用 Lookaside 优化是很值得的。

然后是接口注册。`ProtocolRegisterInterface` 调用 `netifapi_netif_add` 向 lwIP 添加网络接口。这里要注意参数传递的关键：`pProtocol` 被作为 `state` 参数传给了 `netif`，这样 `ethernetif.c` 里的 `low_level_output` 就能通过 `netif->state` 访问到 `KIP_NDIS_PROTOCOL`，进而拿到 `PSendBuffer` 回调函数和 NDIS 包池句柄。有了这个双向引用，lwIP 侧的数据包才能找到 NDIS 侧的发送路径。

最后是发包路径。代码支持两种模式：

NDIS 5 路径走的是 Packet API。具体过程是：先从 `PacketPoolHandle` 中分配一个 `NDIS_PACKET_EX` 结构——这个结构在 `NDIS_PACKET` 尾部嵌入了一个 `NDIS_PACKET_RESERVED`，里面预分配了 `MAX_ETHER_SIZE` 大小的 `DataBuffer`，这样就不需要额外分配数据缓冲了。然后用 `CopyPbufToBuffer` 遍历 lwIP 的 `pbuf` 链，把所有 `pbuf` 的 `payload` 按顺序拷贝到 `DataBuffer` 中。接着调用 `NdisChainBufferAtFront` 把 `DataBuffer` 挂到 `Packet` 上，最后通过微型端口的 `SendPacketsHandler`（批量发送）或 `SendHandler`（单包发送）把数据发出去。发送完成后，`MpSendPacketCompleteHandler` 会被回调，释放数据包并递减 `OutstandingSends` 计数。

NDIS 6 路径走的是 NetBufferLists API。过程类似，但使用的是 `NET_BUFFER_LIST` 和 `NET_BUFFER` 结构，最后通过 `NdisSendNetBufferLists` 发送。

无论走哪条路径，如果网卡支持 SG DMA，都会优先尝试 `MpAllocSGList` 走 DMA 路径。DMA 路径下数据包的物理地址由硬件直接管理，不需要 CPU 参与拷贝，性能最好。

### `miniport.c/h`——微型端口事件处理

这个文件实现了微型端口驱动的事件处理函数，最主要的就是 `MpInitialize`——它在微型端口驱动的初始化完成回调中被调用，负责探测网卡的所有硬件参数并配置好 lwIP 的 `netif`。

`MpInitialize` 的第一步是介质类型检查。它用一个 `switch` 语句判断网卡的介质类型是否被支持：802.3 以太网、802.5 令牌环、FDDI、WAN、IrDA、WirelessWAN 等 12 种介质都通过，其余的（比如 `NdisMedium1394`、`NdisMediumInfiniBand`）直接返回 `NDIS_STATUS_NOT_SUPPORTED` 并标记为 `inactive`。这一步很重要——不是所有 NDIS 介质都需要 TCP/IP 协议栈处理，有些是专用总线的数据通道。

第二步是通过 OID 查询获取硬件参数。OID（Object Identifier）是 NDIS 定义的一套标准查询接口，每个 OID 对应一个特定的硬件属性。查询流程有严格的顺序依赖：

先查 `OID_GEN_VENDOR_DESCRIPTION` 拿厂商字串，主要给调试输出用。然后查 `OID_GEN_MAC_OPTIONS`，这个返回值是一个位掩码，包含几个重要的能力标志。比如 `NDIS_MAC_OPTION_TRANSFERS_NOT_PEND` 表示网卡不会暂挂传输——也就是说发送请求要么立即完成，要么失败，不会出现 `NDIS_STATUS_PENDING`。有这个标志的话，后面的接收路径可以做一些简化。`NDIS_MAC_OPTION_COPY_LOOKAHEAD_DATA` 表示网卡会在查看缓冲区中拷贝数据，上层可以直接使用。这些标志影响了后续的数据处理策略。

接着查 MAC 地址。这里有一个细节：不同的介质类型要查询不同的 OID。802.3 以太网查 `OID_802_3_CURRENT_ADDRESS`，WAN 介质查 `OID_WAN_CURRENT_ADDRESS`，如果查到的地址全是零，说明网卡还没初始化完成，需要后续再查。MAC 地址是 lwIP `netif` 注册的必要参数，没有它协议栈无法工作。

然后查 MTU。`OID_GEN_MAXIMUM_FRAME_SIZE` 返回的是网卡支持的帧数据区大小。以太网的标准 MTU 是 1500，但有些网卡支持 jumbo frame（9000 字节）。`MpInitialize` 会把查询结果和 `MAX_802_3_LENGTH`（1500）取较小值，确保 lwIP 的 TCP 段大小不会超过硬件限制。

之后依次查 `OID_GEN_MAXIMUM_TOTAL_SIZE`（含头部在内的最大帧大小）、`OID_GEN_MAXIMUM_SEND_PACKETS`（网卡驱动的发送队列深度）、`OID_GEN_LINK_SPEED`（链路速度，单位是 100bps，所以要乘以 100 转成 bps）、`OID_GEN_CURRENT_PACKET_FILTER`（当前的包过滤掩码，比如是否接收广播包、组播包等）。

所有参数查完后，`MpInitialize` 用 `DbgPrint` 输出一行调试日志，包含介质类型、MAC 地址、MTU、速度和 MAC 选项，方便驱动开发者验证网卡是否被正确识别。

除了 `MpInitialize` 之外，这个文件还实现了另外几个核心函数。

`MpIndicateReceiveNetBufferLists` 是接收路径的入口。它的工作机制是：当微型端口驱动收到一个数据包并调用 `NdisMIndicateReceiveNetBufferLists` 时，我们的挂钩函数会先一步拦截到它。这个函数遍历 `NET_BUFFER_LIST` 链中的每个 `NET_BUFFER`，检查 `DataLength` 是否不小于 14 字节（以太网头部的最小长度），然后调用 `NdisGetDataBuffer` 获取数据指针，用 `pbuf_alloced_custom(PBUF_REF)` 创建一个引用该数据的 `pbuf`，最后通过 `pProto->IIF.input` 递交给 lwIP 协议栈。整个路径没有数据拷贝，指针直接传递，性能开销极低。

`MpHalt` 在网卡被拔出或驱动卸载时调用，负责优雅关闭。它先调用 `netifapi_netif_set_down` 让 lwIP 知道接口下线，然后用两个 `while` 循环等待所有待发送包（`OutstandingSends`）和待处理包（`PendingPackets`）完成——每次检查之间 delay 20 毫秒，避免忙等浪费 CPU。确认所有包都处理完后，调用 `ProtocolUnBindAdapter` 解绑协议，最后 `NDIS_FreeAdapter` 释放适配器资源。这个顺序保证了下线过程不会丢包。

### `dma.c/h`——DMA 支持

对于支持 DMA 的网卡，可以用硬件直接搬移数据。`MpAllocSGList` 实现了三级降级策略：

1. **路径 A（最优）**：用 Lookaside 列表预分配的 SG 缓冲 + `BuildScatterGatherList`
2. **路径 B（次优）**：不用预分配，让系统分配 SG 列表
3. **路径 C（回退）**：分配连续缓冲，拷贝数据后用物理地址连续的 MDL

`MpProcessSGList` 是 DMA 完成后的回调，把 SG 列表存到数据包里，然后调用微型端口的 `SendHandler`。`MpFreeSGList` 负责释放 SG 列表和相关的标记。

## lwIP 核心协议栈文件

下面的文件是 lwIP 协议栈的标准实现，没有被修改。它们通过 `sys_arch.c` 和 `rosmem.c` 提供的 OS 抽象来获取系统服务。

### TCP 实现——`tcp.c`, `tcp_in.c`, `tcp_out.c`

TCP 是 lwIP 里最复杂的部分。`tcp.c` 负责 PCB 管理、连接建立/关闭、回调机制。`tcp_in.c` 实现 TCP 状态机——从 `SYN_SENT` 到 `ESTABLISHED` 再到 `TIME_WAIT`，RFC 793 定义的 11 个状态全覆盖。`tcp_out.c` 负责发送数据、分段、重传和零窗口探测。

lwIP 的 TCP 实现了 Nagle 算法减少小包、选择性确认（通过 `TCP_QUEUE_OOSEQ` 支持乱序队列）、完整的拥塞控制（慢启动、拥塞避免、快速重传/恢复）、以及 TCP 时间戳（PAWS 防止序列号回绕）。

### UDP 实现——`udp.c`

相对简单，但功能完整：创建/绑定/连接/发送/接收/回调，支持广播和 UDP Lite。

### IP 层——`ip.c`, `ip_addr.c`, `ip_frag.c`

`ip_input` 负责校验和检查、选项处理、按协议分发给 TCP/UDP/ICMP。`ip_output` 构造 IP 头并路由。`ip_frag` 处理分片和重组。

### ICMP——`icmp.c`

响应 Echo Request（ping），发送 Destination Unreachable 和 Time Exceeded。

### ARP——`etharp.c`

实现 ARP 缓存、老化、请求/响应。要注意的是 lwIP 的 ARP 缓存是固定大小的数组，满了会覆盖最旧的条目。

### DHCP——`dhcp.c`

完整的 DHCP 客户端状态机：`INIT` → `SELECTING` → `REQUESTING` → `BOUND` → `RENEWING` → `REBINDING`。支持 Discover/Offer/Request/Ack/Nak/Release 全套消息。

### DNS——`dns.c`

DNS 客户端，支持 `gethostbyname` 和缓存，最多配 3 台 DNS 服务器。

### 内存与缓冲管理——`mem.c`, `memp.c`, `pbuf.c`

配了 `MEM_LIBC_MALLOC=1` 后，`mem_malloc` 直接调用 `rosmem.c` 的 `malloc`。`pbuf.c` 是驱动开发中最常用的 API——`pbuf_alloc`、`pbuf_free`、`pbuf_header`（调整载荷指针，用于加/删协议头）、`pbuf_chain`（链接多个 `pbuf`）。`pbuf_alloced_custom` 是实现零拷贝的关键，它创建一个引用外部缓冲的 `pbuf`，不拥有数据。

### 网络接口管理——`netif.c`, `netifapi.c`

`netif_add`/`remove`/`set_up`/`set_down` 是接口的基本操作。`netifapi` 系列是线程安全的封装，通过 `tcpip_mbox` 把操作消息发到 `tcpip_thread` 执行。

### Socket API——`sockets.c`, `netdb.c`, `netbuf.c`

实现了标准的 BSD Socket 接口：`socket`/`bind`/`listen`/`accept`/`connect`/`recv`/`send`/`close`/`select`。底层通过 `netconn` API 与协议栈交互。

### TCP/IP 线程模型——`tcpip.c`

`tcpip_thread` 是 lwIP 的心脏，它从 `tcpip_mbox` 接收消息并分发处理。`tcpip_input` 是网卡接收数据包的入口，`tcpip_callback_with_block` 用于向 `tcpip_thread` 投递回调。

### 其他辅助文件

`api_lib.c` 和 `api_msg.c` 是 `netconn` API 的实现。`err.c` 把 `err_t` 转成字符串。`def.c` 做字节序转换。`inet_chksum.c` 计算 IP/TCP/UDP/ICMP 校验和。`init.c` 初始化所有子系统。`timers.c` 管理 TCP 重传定时器、ARP 老化定时器、DHCP 续约定时器。

## 平台配置与头文件

### `lwipopts.h`——lwIP 配置

这是协议栈的控制面板。`MEM_LIBC_MALLOC=1` 让 lwIP 用外部的 `malloc` 代替自有堆管理器。`LWIP_COMPAT_MUTEX=1` 用二进制信号量代替互斥体。协议方面 ARP、ICMP、DHCP、DNS、UDP、TCP 全开，IPv6 没开。`TCP_MSS=1460`，`TCP_WND=8760`（6×MSS），`TCP_SND_BUF=17520`（2×WND）。Socket 和 `netconn` API 都开了，统计和调试关了。

### `cc.h`——平台类型定义

```
typedef unsigned char      u8_t;
typedef unsigned short     u16_t;
typedef unsigned long      u32_t;

#define BYTE_ORDER LITTLE_ENDIAN
#define LWIP_PLATFORM_DIAG(x)    (DbgPrint x)
#define LWIP_PLATFORM_ASSERT(x)  DbgPrint("ASSERT: %s\n", x); DbgBreakPoint()
```

断言失败时 `DbgBreakPoint` 触发断点，配合 DbgView 调试非常方便。

### `sys_arch.h`——系统抽象结构

定义了信号量（`KEVENT`）、邮箱（链表+自旋锁+事件）、保护级别（`KIRQL`）、线程句柄（`u32_t`）、以及自定义自旋锁。

### `perf.h`, `bpstruct.h`, `epstruct.h`

`perf.h` 是空的性能测量桩。`bpstruct.h`/`epstruct.h` 成对使用，等价于 `#pragma pack(push,1)`/`pack(pop)`，保证网络协议头结构体的内存布局和线缆上的字节流一致。

## 构建与部署

用 EWDK 的 `sources` 文件配置构建。链接 `ndis.lib` 和 `ntoskrnl.lib`，源文件包括移植层（`lwip_driver.c`、`sys_arch.c` 等）和 lwIP 核心（`tcp.c`、`udp.c` 等）。构建命令`build -cZg`输出 `lwip_demo.sys`。

安装用 `sc create` 注册为内核驱动，`sc start` 启动，`sc stop` 停止。调试输出用 DebugView 查看。

注意，这里是针对于windows 7的工具链而言，请不要在现代Windows10/11上运行

请记住是Windows 7 的EWDK，而非普通WDK

编译：`msbuild lwip_demo.vcxproj /p:configuration=Release /p:platform=x64`

## 风险与限制

当然，这个项目也有一些潜在的风险和限制。

首先也是最关键的，Kernel PatchGuard 兼容性问题。在 x64 Windows 10/11 上修改内核代码段会触发 `CRITICAL_STRUCTURE_CORRUPTION` 蓝屏。这个问题在目前的技术框架下无解，只能靠禁用 PatchGuard 或者用其他方式绕。

其次是全局自旋锁瓶颈。`sys_arch.c` 用了一个全局自旋锁保护临界区，多核高负载下可能成为瓶颈。

还有就是 NDIS 内部结构的版本依赖。`ndisx.h` 是逆向 Windows 7 得到的，新版本的 NDIS 结构偏移可能变了，需要重新逆向适配。

最后，单线程 `tcpip_thread` 和无 IPv6 支持也是需要考虑的限制。

## 总结

`lwip_nt_minimal` 展示了怎么把嵌入式 TCP/IP 栈移植到 Windows 内核里，怎么通过 API 挂钩拦截 NDIS 子系统，怎么在内核态做 DMA 数据传输，怎么逆向 Windows 的未文档化结构。这些技术组合在一起，实现了一个完整的、能通过物理网卡收发数据的内核态 TCP/IP 协议栈。

最有意思的是，我通过这个发出了一个ICMP Ping包，而wireshark完全没有反应

对于做驱动开发、安全研究或者内核网络编程的人来说，这个项目的代码是非常好的学习材料。

我还在研究这个项目如何适配windows 10/11，这是一个很难的项目

欢迎提出意见，附件是移植源代码
