
# [原创]借`windump`初窥网络底层

最近在学ndis以及网络方面的知识,今天借论坛宝地整理一些笔记出来,与大家共同分享下

<太过基础的原理就不做说明了,大家可以去网上搜索>

首先是各种包的格式定义:

以太头:

```
// MAC地址长度

#define MAC_ADDRESS_LEN 6



// 常用的以太网帧类型

#define ETHERNET_FRAME_IP		0x0800				// IP帧 

#define ETHERNET_FRAME_ARP		0x0806				// ARP帧 

#define ETHERNET_FRAME_RARP		0x8035				// RARP帧 

#define ETHERNET_FRAME_IPv6		0x86DD				// IP6帧 



typedef struct _EthernetHeader		// 以太头

{

	UCHAR		DstMacAddr[MAC_ADDRESS_LEN];		// 目标MAC

	UCHAR		SrcMacAddr[MAC_ADDRESS_LEN];		// 源MAC

	USHORT		FrmType;							// 类型

}EthernetHeader, *PEthernetHeader;
```

ARP,RARP头

```
// 定义ARP操作类型

#define ARP_QUERY				0x0001				// ARP请求 

#define ARP_REPLY				0x0002				// ARP应答

#define RARP_QUERY				0x0003				// RARP请求

#define RARP_REPLY				0x0004				// RARP应答



// 定义ARP硬件类型

#define ARP_ETHER				0x0001				// 以太网格式

#define ARP_IEEE802				0x0006				// 令牌环网

#define ARP_FRELAY				0x000f				// 帧中继硬件格式



typedef struct _ArpHeader			// ARP头<RARP>

{

	USHORT		HrdType;							// 硬件类型

	USHORT		ProType;							// 协议类型

	UCHAR		HrdAddrLen;							// 硬件地址长度

	UCHAR		ProAddrLen;							// 协议地址长度

	USHORT		Opr;								// 操作字段 <是ARP请求或者应答 异或RARP请求或者应答>

	UCHAR		SrcMacAddr[MAC_ADDRESS_LEN];		// 发送端的MAC地址

	ULONG		SrcIp;								// 发送端的IP地址

	UCHAR		DstMacAddr[MAC_ADDRESS_LEN];		// 目的MAC地址

	ULONG		DstIp;								// 目的IP地址

}ArpHeader, *PArpHeader;



typedef struct _ARPPACKET			// ARP封包

{

	EthernetHeader	Ehhdr;

	ArpHeader		Arphdr;

}ARPPACKET, *PARPPACKET;
```

IP头

```
 0                            15                              31

-----------------------------------------------------------------

|  ver  |  HL   |     TOS       |        Total len              |

-----------------------------------------------------------------

|            Packet id          |0|D|M|   Fragment Offset       |

-----------------------------------------------------------------

|     TTL       |    protocol   |          checksum             |

-----------------------------------------------------------------

|                       source ip address                       |

-----------------------------------------------------------------

|                       dest ip address                         |

-----------------------------------------------------------------

|                           Optional                            |

-----------------------------------------------------------------

|                              Data                             |

-----------------------------------------------------------------



// 定义IP数据报中的协议类型

#define ICMP_PROTOCOL			0x0001				// ICMP 报文

#define IGMP_PROTOCOL			0x0002				// IGMP 报文

#define TCP_PROTOCOL			0x0006				// TCP 报文

#define UDP_PROTOCOL			0x0011				// UDP 报文 17



// IP标志

#define IP_DF_MASK				0x4000				// 如果置位,表示不能被分割

#define IP_MF_MASK				0x2000				// 为1表示还有更多的数据报分片 为0表示已经是最后一个分片



typedef struct _IPHeader			// IP头 20 BYTES

{

	UCHAR		Ver:4;								// 版本信息

	UCHAR		HdrLen:4;							// IP数据报首部的双字个数,一般为5即20字节长

	UCHAR		TOS;								// 服务类型<一般忽略>

	USHORT		TotalLen;							// IP数据报总长度<字节单位>,包括了IP头,这个长度减去HdrLen就是数据实际长度

	USHORT		PktID;								// 封包ID 每发送一个包这里加一

	USHORT		FlagSeg;							// 标致字段和片偏移 3位标志 13位偏移

	UCHAR		TTL;								// 存活周期

	UCHAR		Protocol;							// 协议类型

	USHORT		Checksum;							// 效验和

	ULONG		SrcIp;								// 源IP

	ULONG		DstIp;								// 目的IP

}IPHeader, *PIPHeader;



typedef struct _IPPACKET			// IP封包

{

	EthernetHeader	Ehhdr;

	IPHeader		Iphdr;

}IPPACKET, *PIPPACKET;
```

TCP头

```
 0                            15                              31

-----------------------------------------------------------------

|          source port          |       destination port        |

-----------------------------------------------------------------

|                        sequence number                        |

-----------------------------------------------------------------

|                     acknowledgment number                     |

-----------------------------------------------------------------

|  HL   | rsvd  |C|E|U|A|P|R|S|F|        window size            |

-----------------------------------------------------------------

|         TCP checksum          |       urgent pointer          |

-----------------------------------------------------------------

|                           Optional                            |

-----------------------------------------------------------------

|                              Data                             |

-----------------------------------------------------------------



// TCP头标志位掩码 OffFlag

#define TCP_HDR_LEN				0xF000				// TCP头长度<双字的个数>转化为字节数需要x4		

#define TCP_URG_FLAG			0x0020				// URG标志

#define TCP_ACK_FLAG			0x0010				// ACK标志 应答1或请求0

#define TCP_PSH_FLAG			0x0008				// psh标志 以最快速度传输数据

#define TCP_RST_FLAG			0x0004				// RST标志 先断开连接,再重建连接

#define TCP_SYN_FLAG			0x0002				// SYN标志 用来建立连接

#define TCP_FIN_FLAG			0x0001				// FIN标志 发送方完成数据发送



typedef struct _TcpHeader			// TCP头 20 BYTES

{

	USHORT		SrcPort;							// 源端口

	USHORT		DstPort;							// 目的端口

	ULONG		SeqNum;								// 序号

	ULONG		AckNum;								// 确认号

	USHORT		OffFlag;							// 首部长度<首部双字的个数>+保留位+标志位

	USHORT		WndSize;							// 窗口大小<实现流量控制>

	USHORT		Checksum;							// 效验和

	USHORT		UrgPointer;							// 紧急指针

}TcpHeader, *PTcpHeader;



// OffFlag : 4位首部长度 + 6位保留位 + URG+ACK+PSH+RST+SYN+FIN



typedef struct _TCPPACKET			// TCP封包 <当ip中没有附加数据时>

{

	EthernetHeader	Ehhdr;

	IPHeader		Iphdr;

	TcpHeader		Tcphdr;

}TCPPACKET, *PTCPPACKET;
```

UDP头:

```
 0                            15                              31

-----------------------------------------------------------------

|          source port          |       destination port        |

-----------------------------------------------------------------

|          udp len              |       udp checksum            |

-----------------------------------------------------------------

|                             data                              |

-----------------------------------------------------------------

typedef struct _UdpHeader

{

	USHORT		SrcPort;							// 源端口

	USHORT		DstPort;							// 目的端口

	USHORT		UdpLen;								// 长度 头+数据字节长 最小8字节

	USHORT		Checksum;							// 效验和

}UdpHeader, *PUdpHeader;



typedef struct _UDPPACKET			// UDP封包 <当ip中没有附加数据时>

{

	EthernetHeader	Ehhdr;

	IPHeader		Iphdr;

	UdpHeader		Udphdr;

}UDPPACKET, *PUDPPACKET;
```

上面这些头在传输时都是网络字节序列,在访问时需要注意,而且不会进行双字对其.

所以最好使用对其标志将结构包起来:

```
#pragma pack(push)

#pragma pack(1)

..

..

#pragma pack(pop)
```

介绍完结构我们就抓几个包来实践一下,抓包工具我使用了`windump`,不过这个是基于控制台的程序,

大家看着不方便也可以使用其他的抓包工具,比如`wireshark`

`windump`我会在最后的附加中上传,使用它需要先安装winpcap

打开`CMD`命令提示符,定位到`windump`的目录然后输入`windump`,就能让它运行起来了:

![](https://bbs.kanxue.com/upload/attach/201203/440388_yayin60mupy4zjo.jpg)

<可以使用Ctrl+C停止`windump`>

如果你的电脑上有多个网卡,可以用`windump -D`指令首先列出可来监听的接口列表,

然后在启动`windump`的时候用`-i`指令来指定使用哪个接口来监听, 比如 : `windump -i3`

如果没有附加条件,`windump`默认将dump所有经过网卡的封包,下面我会介绍几个过滤条件,

更多的信息可以在附件中附带的帮助文档中找到.也可以在网上搜索到详细介绍

首先我们看一下网络中的ARP封包,可以通过 `windump -n arp` 命令来过滤

其中`-n`表示不要将主机的ip地址显示为主机名称;`arp`表示只过滤arp封包

<我的测试环境是局域网,路由ip:192.168.1.1, 我的ip:192.168.1.23, OS:win7 sp1>

监听一会就能捕获一些局域网上的ARP封包:

![](https://bbs.kanxue.com/upload/attach/201203/440388_e0gbdev80vckp58.jpg)

```
arp who-has 192.168.1.1 (00:25:86:99:2c:d6) tell 192.168.1.23
```

可以理解为我的主机在主动判断网关地址是否存在

```
reply 192.168.1.1 is-at 00:25:86:99:2c:d6
```

则是路由对我的主机的回复

我们现在使用命令 `windump -n -XX arp` 来打印出封包的具体内容

`-XX`表示使用16进制和ASCii码的形式打印封包内容.`-X`与之相仿,不过不会打印以太头:

![](https://bbs.kanxue.com/upload/attach/201203/440388_cx12fpneyz8y15y.jpg)

前14个字节描述了以太头的相关信息,`windump`完全按照字节序列进行打印,

以太头最后的 `0806` 代表了后面帧的类型,代表了ARP帧<参见前面的定义>

接下来是ARP头,我们来看第一个封包的内容,

```
0x000E: 0001 表示硬件类型是以太网类型

0x0010: 0800 表示协议类型是IP协议

0x0012: 0604 表示硬件地址<MAC>长为6字节,协议地址<IP>长为4字节

0x0014: 0001 表示操作类型为ARP请求 ARP_QUERY
```

后面就依次是源MAC,源IP,目标MAC,目标IP了

第二个封包描述了路由器对ARP请求的应答,不过末尾多了一部分数据,我找了一些资料也没找到相关说明,

如果哪位朋友知道可以告诉我一下,先行谢过了~~

最后的部分:
