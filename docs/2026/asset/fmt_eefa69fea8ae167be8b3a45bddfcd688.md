
在Windows下，Socket是以DLL的形式实现的。在DLL内部维持着一个计数器，只有第一次调用`WSAStartup`才真正装载DLL，以后的调用只是简单的增加计数器，而`WSACleanup`函数的功能则刚好相反，每调用一次使计数器减1，当计数器减到0时，DLL就从内存中被卸载！因此，你调用了多少次`WSAStartup`，就应相应的调用多少次的`WSACleanup`。

### 2、`inet_addr()`

`inet_addr()`将一个点分十进制的IP转换成一个长整数型数（`u_long`）

### 3、`socket()`

`socket()`函数对应于普通文件的打开操作。普通文件的打开操作返回一个文件描述字，而`socket()`用于创建一个socket描述符（socket descriptor），它唯一标识一个socket。这个socket描述字跟文件描述字一样，后续的操作都有用到它，把它作为参数，通过它来进行一些读写操作。

正如可以给`fopen`的传入不同参数值，以打开不同的文件。创建socket的时候，也可以指定不同的参数创建不同的socket描述符，`socket()`函数的三个参数分别为：

- `domain`：即协议域，又称为协议族（family）。常用的协议族有，`AF_INET`、`AF_INET6`、`AF_LOCAL`（或称`AF_UNIX`，Unix域socket）、`AF_ROUTE`等等。协议族决定了socket的地址类型，在通信中必须采用对应的地址，如`AF_INET`决定了要用ipv4地址（32位的）与端口号（16位的）的组合、`AF_UNIX`决定了要用一个绝对路径名作为地址。

| Af | Meaning |
| --- | --- |
| `AF_UNSPEC` 0 | The address family is unspecified. |
| `AF_INET` 2 | The Internet Protocol version 4 (IPv4) address family. |
| `AF_IPX` 6 | The IPX/SPX address family. This address family is only supported if the NWLink IPX/SPX NetBIOS Compatible Transport protocol is installed. This address family is not supported on Windows Vista and later. |
| `AF_APPLETALK` 16 | The AppleTalk address family. This address family is only supported if the AppleTalk protocol is installed. This address family is not supported on Windows Vista and later. |
| `AF_NETBIOS` 17 | The NetBIOS address family. This address family is only supported if the Windows Sockets provider for NetBIOS is installed. The Windows Sockets provider for NetBIOS is supported on 32-bit versions of Windows. This provider is installed by default on 32-bit versions of Windows. The Windows Sockets provider for NetBIOS is not supported on 64-bit versions of windows including Windows 7, Windows Server 2008, Windows Vista, Windows Server 2003, or Windows XP. The Windows Sockets provider for NetBIOS only supports sockets where the `type` parameter is set to `SOCK_DGRAM`. The Windows Sockets provider for NetBIOS is not directly related to the [NetBIOS](https://bbs.kanxue.com/elink@b1dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4m8J5k6i4k6A6L8%4g2K6i4K6u0V1N6X3g2J5M7$3W2G2L8Y4y4Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3k6r3g2K6K9%4c8G2M7q4)9J5c8X3&6W2N6r3u0A6L8%4y4Q4x3V1k6H3L8%4u0@1j5h3H3`.) programming interface. The NetBIOS programming interface is not supported on Windows Vista, Windows Server 2008, and later. |
| `AF_INET6` 23 | The Internet Protocol version 6 (IPv6) address family. |
| `AF_IRDA` 26 | The Infrared Data Association (IrDA) address family. This address family is only supported if the computer has an infrared port and driver installed. |
| `AF_BTH` 32 | The Bluetooth address family. This address family is supported on Windows XP with SP2 or later if the computer has a Bluetooth adapter and driver installed. |

- `type`：指定socket类型。常用的socket类型有，`SOCK_STREAM`、`SOCK_DGRAM`、`SOCK_RAW`、`SOCK_PACKET`、`SOCK_SEQPACKET`等等。

| Type | Meaning |
| --- | --- |
| `SOCK_STREAM` 1 | A socket type that provides sequenced, reliable, two-way, connection-based byte streams with an OOB data transmission mechanism. This socket type uses the Transmission Control Protocol (TCP) for the Internet address family (AF_INET or AF_INET6). |
| `SOCK_DGRAM` 2 | A socket type that supports datagrams, which are connectionless, unreliable buffers of a fixed (typically small) maximum length. This socket type uses the User Datagram Protocol (UDP) for the Internet address family (AF_INET or AF_INET6). |
| `SOCK_RAW` 3 | A socket type that provides a raw socket that allows an application to manipulate the next upper-layer protocol header. To manipulate the IPv4 header, the [`IP_HDRINCL`](https://bbs.kanxue.com/elink@afdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6V1k6i4y4C8N6r3!0H3i4K6u0r3g2$3W2F1f1$3!0U0K9#2)9J5c8X3W2H3M7s2u0G2N6r3!0Q4x3X3c8A6M7q4)9J5k6s2y4G2j5$3E0W2N6q4)9J5k6r3!0H3N6r3W2G2L8Y4x3`.) socket option must be set on the socket. To manipulate the IPv6 header, the [`IPV6_HDRINCL`](https://bbs.kanxue.com/elink@2b7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6V1k6i4y4C8N6r3!0H3i4K6u0r3g2$3W2F1f1$3!0U0K9#2)9J5c8X3W2H3M7s2u0G2N6r3!0Q4x3X3c8A6M7s2j5$3i4K6u0V1M7$3!0U0K9$3g2@1i4K6u0V1L8%4m8@1K9h3!0F1M7H3`.`.) socket option must be set on the socket. |
| `SOCK_RDM` 4 | A socket type that provides a reliable message datagram. An example of this type is the Pragmatic General Multicast (PGM) multicast protocol implementation in Windows, often referred to as [reliable multicast programming](https://bbs.kanxue.com/elink@6a1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6V1k6i4y4C8N6r3!0H3i4K6u0r3g2$3W2F1f1$3!0U0K9#2)9J5c8Y4u0W2L8r3W2S2j5X3I4W2i4K6u0V1L8i4g2D9N6r3W2U0j5i4y4@1i4K6u0V1M7s2u0G2k6%4u0S2L8h3#2A6L8X3N6Q4x3X3c8Q4x3X3c8H3k6$3#2Q4x3X3b7`.). This `type` value is only supported if the Reliable Multicast Protocol is installed. |
| `SOCK_SEQPACKET` 5 | A socket type that provides a pseudo-stream packet based on datagrams. |

- `protocol`：故名思意，就是指定协议。常用的协议有，`IPPROTO_TCP`、`IPPTOTO_UDP`、`IPPROTO_SCTP`、`IPPROTO_TIPC`等，它们分别对应TCP传输协议、UDP传输协议、STCP传输协议、TIPC传输协议。

| protocol | Meaning |
| --- | --- |
| `IPPROTO_ICMP` 1 | The Internet Control Message Protocol (ICMP). This is a possible value when the `af` parameter is `AF_UNSPEC`, `AF_INET`, or `AF_INET6` and the `type` parameter is `SOCK_RAW` or unspecified. This `protocol` value is supported on Windows XP and later. |
| `IPPROTO_IGMP` 2 | The Internet Group Management Protocol (IGMP). This is a possible value when the `af` parameter is `AF_UNSPEC`, `AF_INET`, or `AF_INET6` and the `type` parameter is `SOCK_RAW` or unspecified. This `protocol` value is supported on Windows XP and later. |
| `BTHPROTO_RFCOMM` 3 | The Bluetooth Radio Frequency Communications (Bluetooth RFCOMM) protocol. This is a possible value when the `af` parameter is `AF_BTH` and the `type` parameter is `SOCK_STREAM`. This `protocol` value is supported on Windows XP with SP2 or later. |
| `IPPROTO_TCP` 6 | The Transmission Control Protocol (TCP). This is a possible value when the `af` parameter is `AF_INET` or `AF_INET6` and the `type` parameter is `SOCK_STREAM`. |
| `IPPROTO_UDP` 17 | The User Datagram Protocol (UDP). This is a possible value when the `af` parameter is `AF_INET` or `AF_INET6` and the `type` parameter is `SOCK_DGRAM`. |
| `IPPROTO_ICMPV6` 58 | The Internet Control Message Protocol Version 6 (ICMPv6). This is a possible value when the `af` parameter is `AF_UNSPEC`, `AF_INET`, or `AF_INET6` and the `type` parameter is `SOCK_RAW` or unspecified. This `protocol` value is supported on Windows XP and later. |
| `IPPROTO_RM` 113 | The PGM protocol for reliable multicast. This is a possible value when the `af` parameter is `AF_INET` and the `type` parameter is `SOCK_RDM`. On the Windows SDK released for Windows Vista and later, this protocol is also called `IPPROTO_PGM`. This `protocol` value is only supported if the Reliable Multicast Protocol is installed. |

注意：并不是上面的`type`和`protocol`可以随意组合的，如`SOCK_STREAM`不可以跟`IPPROTO_UDP`组合。当`protocol`为0时，会自动选择`type`类型对应的默认协议。

当我们调用`socket()`创建一个socket时，返回的socket描述字它存在于协议族（address family，`AF_XXX`）空间中，但没有一个具体的地址。如果想要给它赋值一个地址，就必须调用`bind()`函数，否则就当调用`connect()`、`listen()`时系统会自动随机分配一个端口。
