
# [原创]CVE-2022-34718 TCP/IP 远程命令执行漏洞分析

**原文链接：<mark class="encrypted">6beK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7q4)9J5k6i4N6W2K9i4S2A6L8W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8U0g2X3z5p5N6I4f1e0R3&6x3W2A6%4j5W2N6x3k6r3!0%4h3q4S2h3e0$3N6Q4x3V1q4Q4x3V1p5`.</mark>

## 1. 漏洞概述

| 项目 | 详情 |
| -- | -- |
| **名称** | Windows TCP/IP Remote Code Execution Vulnerability |
| **简介** | `tcpip.sys`在对ipv6数据包进行重组时，`NextHeaderOffset`可能超出缓冲区范围，导致越界写的发生，利用该漏洞可以实现远程命令执行 |
| **影响版本** | 该漏洞影响所有受支持的 Windows 操作系统 |
| **编号** | CVE-2022-34718 |

## 2. 基础知识

### 2.1 IPv6 fragment

如果发送的数据包大小超过了最大传输单元（MTU），IPv6 源节点会将数据包分割成多个分片，在前面添加 fragment header ，作为单独的数据包进行传送，然后由目的节点对数据包进行重组。

Fragment header 结构如下：

```
0 1 2 3
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Next Header | Reserved | Fragment Offset |Res|M|
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Identification |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

- Next header: 1 字节，未分割之前原始数据包的包头类型；
- Reserved: 1 字节，初始化为 0；
- Fragment Offset: 13 比特，头部后方数据相对于原始可分割单位的偏移，以 8 字节为单位；
- Res: 2 比特，初始化为 0；
- M flag: 1 = 后面还有fragments; 0 = 这是最后一个fragment;
- Identification: 4 字节，由发送源点生成，用于和最近发送的，且源地址和目的地址相同的分片数据包做区分。

在分割时，数据包会被分为 不可分片部分 和 可分片部分：
- 不可分片部分：数据包中需要被中间节点处理的部分，例如IPv6头、Hop-by-Hop Options头、中间节点的Destination Options头、Routing头等
- 可分片部分：数据包中只需要被目的节点处理的部分，例如目的节点的Destination Options头、上层（ICMPv6、TCP、UDP）头以及上层的数据

假设原始数据包组成如下：

```
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| IPv6 header | Hop-by-Hop options | ICMPv6 header | ICMPv6 paylaod |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

根据数据包的大小，它会被源节点分成多个分片:

```
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| IPv6 header | Hop-by-Hop options | Fragment header | First fragment |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| IPv6 header | Hop-by-Hop options | Fragment header | Second fragment |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-
| IPv6 header | Hop-by-Hop options | Fragment header | Last fragment |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-
```

当上面的数据包都到达目的节点后，会重组成为原始数据包：
- 从第一个分片数据包获取不可分片部分；
- 将每个数据包中 Fragment header 后面的数据提取出来，组成可分片部分。

除此之外需要修改两个位置的数值：
- 不可分片部分最后一个头部中的 next header 字段，数值从第一个分片的 fragment header 中获取；
- 重组后数据包的 Payload length 字段需要重新进行计算。

### 2.2 IPSec

Internet Protocal Security(IPsec) 是一个安全网络协议组，可以对数据包进行验证和数据加密，从而为位于 IP 网络中的两台计算机提供安全的加密通信。IPsec 用于 virtual private networks(VPNs)。

IPsec 使用以下的协议实现不同的功能：
- Authentication Header(AH): 为IP 数据包提供无连接数据完整性和数据源认证，为重放攻击提供保护；
- Encapsulating Security Payloads(ESP): 提供机密性、无连接数据完整性、数据源认证、反重放服务（通过部分序列完整性的方式）以及有效的通信流机密性；
- Internet Security Association and Key Management Protocol(ISAKMP): 提供了一个用于身份认证和密钥交换的框架，通过手动配置预共享密钥、Internet Key Exchange(IKE和IKEv2)、Kerverized Internet Negotiation of Keys(KINK) 或者 IPSECKEY DNS 记录来提供真正经过验证的密钥材料。该协议的目的是使用 AH 或 ESP 操作所需的各种算法和参数生成 security associations(SA)。

其中 ESP 结构如下：

```
0 1 2 3
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Security Parameters Index (SPI) |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Sequence Number |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Payload data |
+ +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| | Padding |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ -+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| | Pad Length | Next Header |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Integrity Check Value (ICV) |
+ ...... +
| ...... |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

- Security Parameters Index (32 bits)

随机值，和目标 IP 一起用于识别接收方的 security association
- Sequence Number (32 bits)

递增计数器，防止重放攻击
- Payload data (variable)

保护的原始 IP 包内容，还包括用于保护数据的其他参数（例如用于加密算法的初始化变量）。保护的数据类型由下面的 Next Header 字段说明。
- Padding (0-255 octets)

填充数据，用于和加密算法的 cipher block size 以及下一个字段对齐
- Pad Length (8 bits)

填充数据的长度，以八位字节为单位
- Next Header (8 bits)

下一个头部的类型，数值选自 <a href="elink@a35K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6x3K9i4y4@1i4K6g2X3L8$3k6Q4y4h3k6u0f1q4)9#2k6Y4m8J5L8%4c8G2j5$3!0D9i4K6g2X3L8Y4g2E0j5X3g2J5M7H3`.`.">List of IP protocol numbers</a>
- Integrity Check Value (multiple of 32 bits)

可变长度的检查值，可能包含填充数据，用于和 IPv6 的 8 字节边界或者 IPv4 的 4 字节边界进行对齐

## 3. 漏洞分析

漏洞发生时的函数调用栈如下：

```
0: kd> kb
# RetAddr : Args to Child : Call Site
00 fffff80a`1f1842ea : ffffb00e`0cbef000 ffffb00e`0d333270 ffffb00e`0f5b4890 ffffb00e`00000050 : tcpip!Ipv6pReassembleDatagram+0x1e3
01 fffff80a`1f184442 : ffffb00e`00000000 fffff80a`1f1c61e8 00000000`00000000 ffffb00e`0d3138f0 : tcpip!Ipv6pReceiveFragment+0xae2
02 fffff80a`1f03bfdf : ffffb00e`0cbef000 ffffb00e`0cbef000 00000000`00000001 00000000`0000002b : tcpip!Ipv6pReceiveFragmentList+0x42
03 fffff80a`1f03de45 : fffff80a`1f1c1240 ffffb00e`0c8ec940 00000000`00000001 ffffb00e`0cbef000 : tcpip!IppReceiveHeaderBatch+0x3ef
04 fffff80a`1f044131 : ffffb00e`0d31eb90 ffffb00e`0d23e9c0 fffff803`94d7b001 00000000`00000000 : tcpip!IppFlcReceivePacketsCore+0x315
05 fffff80a`1f043e12 : 00000000`00000017 fffff803`00000001 fffff80a`1f001bf0 ffffb00e`0d23e901 : tcpip!FlpReceiveNonPreValidatedNetBufferListChain+0x271
06 fffff803`92ebce75 : 00000000`00000002 fffff803`931d1980 fffff80a`1f043d50 fffff803`94d7b1f0 : tcpip!FlReceiveNetBufferListChainCalloutRoutine+0xc2
07 fffff80a`1f0023a6 : ffffb00e`0cc1b7b0 00000000`00000000 ffffb00e`0c7ff870 ffffb00e`0d23e900 : nt!KeExpandKernelStackAndCalloutInternal+0x85
08 fffff80a`1e10392e : 00000000`00000000 fffff803`94d7b2c0 00000000`00000001 fffff80a`1de5f20d : tcpip!FlReceiveNetBufferListChain+0xb6
09 fffff80a`1e1028cc : fffff80a`1f522801 fffff803`94d7dd86 0000000e`00000000 fffff80a`00000001 : NDIS!ndisMIndicateNetBufferListsToOpen+0x11e
0a fffff80a`1eca6156 : 00000000`00000000 00000000`00000000 ffffb00e`0d0f4e40 ffffb00e`0d23e930 : NDIS!NdisMIndicateReceiveNetBufferLists+0x31c
0b fffff80a`1eca73e3 : ffffb00e`0d23e900 00000000`00000001 ffffb00e`0d0f4e40 fffff803`94d7b588 : e1i63x64!RECEIVE::RxIndicateNBLs+0x132
0c fffff80a`1ecae315 : ffffb00e`0c483830 ffffb00e`0d0f4000 ffffb00e`0d0f4001 fffff803`00000000 : e1i63x64!RECEIVE::RxProcessInterrupts+0x253
0d fffff80a`1ecae623 : ffffb00e`0c4226b0 00000001`00000000 00000001`00000000 00000000`00000000 : e1i63x64!INTERRUPT::MsgIntDpcTxRxProcessing+0x121
0e fffff80a`1ecaddb8 : ffffb00e`0e66f7d8 00000000`00000000 40200342`00000000 00000000`00000000 : e1i63x64!INTERRUPT::MsgIntMessageInterruptDPC+0x10f
0f fffff80a`1e104e69 : ffffda01`9d7c9010 ffffda01`9d7c9320 00000000`00000000 00000000`00000000 : e1i63x64!INTERRUPT::MiniportMessageInterruptDPC+0x28
10 fffff803`92e7f785 : ffffb00e`0d200000 fffff803`93155180 ffffb00e`0cc8e5d0 ffffb00e`0cc8e5d0 : NDIS!ndisInterruptDpc+0x1c9
11 fffff803`92e7ed10 : fffff80a`20dc79a8 00000000`002e593c 00000000`00140001 00000000`00000000 : nt!KiExecuteAllDpcs+0x335
12 fffff803`92f7500a : 00000000`00000000 fffff803`93155180 fffff803`931d1980 ffffb00e`0f6cc080 : nt!KiRetireDpcList+0x910
13 00000000`00000000 : fffff803`94d7c000 fffff803`94d76000 00000000`00000000 00000000`00000000 : nt!KiIdleLoop+0x5a
```

根据以上信息可以确定，`tcpip.sys` 使用 ```IppReceiveHeaderBatch``` 函数处理接收到的数据包，对 next header 的数值进行判断并调用相应函数处理对应头部，漏洞就发生在处理 fragment header 的时候。
`tcpip.sys` 会调用 ```Ipv6pReceiveFragmentList``` 函数对 fragment header 进行处理，在对分片进行重组的时候：
