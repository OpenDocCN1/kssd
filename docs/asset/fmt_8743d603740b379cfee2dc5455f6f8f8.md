
# Winsock 初探

> 原文：[`www.pediy.com/kssd/pediy10/74810.html`](https://www.pediy.com/kssd/pediy10/74810.html)

在编写 Windows 网络应用程序的时候，最常用的便是 Winsock 接口，注意它不是网络协议，你可以理解它为网络应用 API。在百度百科中是这样介绍的：

> " Windows 下网络编程的规范－Windows Sockets 是 Windows 下得到广泛应用的、开放的、支持多种协议的网络编程接口。从 1991 年的 1.0 版到 1995 年的 2.0.8 版，经过不断完善并在 Intel、Microsoft、Sun、SGI、Informix、Novell 等公司的全力支持下，已成为 Windows 网络编程的事实上的标准。 "

实际上 Windows95 以后的各个 windows 版本都支持 Winsock2.2 版本，但是 Window CE 只支持 1.1 版本。Winsock2.2 提供了更多的功能和 API 函数，而且区分两个版本的函数非常容易，2.2 版本的 API 函数都以 `WSA` 开头的，除了 `WSAStartup`, `WSACleanup`, `WSARecvEx`, 和 `WSAGetLastError` 这几个函数也出现在 1.1 版本中。

使用 Winsock 写应用程序所需要的头文件和类库有：

```c
// 对于 1.1 版本
#include<winsock.h>
#pragma comment(lib, "wsock32.lib")

// 对于 2.2 版本
#include<winsock2.h>
#pragma comment(lib, "ws2_32.lib")
```

现在我们来看看一个通用的 Winsock 应用程序框架：

```c
#include<winsock2.h>  // 头文件
#pragma comment(lib, "ws2_32.lib") // 库文件加载

void main(void)
{
   WSADATA wsaData;  // WSADATA 结构体主要包含了系统所支持的 Winsock 版本信息
   // 初始化 Winsock 2.2。使用 WSAStartup 函数，第一个参数是所要用的 Winsock 版本号
   // 第二个参数就是 WSADATA 结构体的指针。如果初始化成功则返回 0
   // 要注意任何 WinsockAPI 函数都必须在初始化后使用，包括错误检查函数
   // WSAGetLastError （用于查看出错详细信息）
   if( WSAStartup( MAKEWORD(2,2), &wsaData) != 0 )
   {
      printf( "WSAStartup 无法初始化！");
      return;
   }
    // winsock 应用代码
    // 最后应该做一些清除工作
    if( WSACleanup() == SOCKET_ERROR )
        printf( "WSACleanup 出错！");
}
```

这里需要说明的有两点：

1. `WSADATA` 结构体：

```c
typedef struct WSAData
{
    WORD            wVersion;
    WORD            wHighVersion;
    char            szDescription[WSADESCRIPTION_LEN + 1];
    char            szSystemStatus[WSASYS_STATUS_LEN + 1];
    unsigned short  iMaxSockets;
    unsigned short  iMaxUdpDg;
    char FAR *      lpVendorInfo;
} WSADATA, * LPWSADATA;
```

`wVersion` 就是对应你所要的版本，比如 1.2，高位字节对应 2，低位字节对应 1。

`wHighVersion` 是系统所支持的最高版本，如果初始化失败，这个参数仍然返回。

其他成员基本都用不到，所以不必关心。

2. 对应每一个 `WSAStartup` 都要有一个 `WSACleanup` 与之对应

## Winsock 初探 II

前些天写了 Winsock 初探一文，现在继续往下写。本文的目的不是想介绍一个完整的网络应用程序，而只是作为一个初步介绍如何写一个网络程序(基本思路)，正因为是初探，所以代码比较简单，为了使代码清晰起见，我省略了错误检查代码部分，但这不意味着错误检查不重要。

写一个网络应用程序分为两个方面：服务器程序和客户端程序，大家可以在一个 solution 中建立 `srv` 和 `client` 两个 projects。

首先回忆一下程序框架，见 Winsock 初探 一文

```c
#include<winsock2.h> // 头文件
#pragma comment(lib, "ws2_32.lib") // 库文件加载

void main(void)
{
WSADATA wsaData; // WSADATA 结构体主要包含了系统所支持的 Winsock 版本信息
// 初始化 Winsock 2.2。使用 WSAStartup 函数，第一个参数是所要用的 Winsock 版本号
// 第二个参数就是 WSADATA 结构体的指针。如果初始化成功则返回 0
// 要注意任何 WinsockAPI 函数都必须在初始化后使用，包括错误检查函数
// WSAGetLastError （用于查看出错详细信息）
if( WSAStartup( MAKEWORD(2,2), &wsaData) != 0 )
{
printf( "WSAStartup 无法初始化！");
return;
}
// winsock 应用代码
// 最后应该做一些清除工作
if( WSACleanup() == SOCKET_ERROR )
printf( "WSACleanup 出错！");
}
```

现在我们的任务是填写红色的部分，即 winsock 应用代码

### 1. 简单 TCP/IP

思路：

**[服务器程序]**

建立 `socket` -----> 绑定 `bind` ------>监听 `listen` ------>接受 `accept` ------>发送和接收 `send` 和 `recv` ------>关闭 `closesocket`

**[客户端程序]**

建立 `socket` -----> 连接 `connect` ------>发送和接收 `send` 和 `recv` ------>关闭 `closesocket`

代码：

**[服务器程序]**

```c
/*使用 IP 地址家族就必须要用 AF_INET，由于我们要建立可靠的传送，因此我们选择 SOCK_STREAM，基于数据流的传送，最后一个参数表示使用 TCP 协议，实际上这个参数可以设置为 0，让程序自己根据情况判断填写*/
SOCKET sockSrv = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP )

/*绑定的意思是把 socket 绑定到 IP 地址上，所以我们需要一个 SOCKADDR 结构来存放 IP 地址信息，这里为了方便填写可以使用 SOCKADDR_IN 结构，这个结构的大小和 SOCKADDR 一样，可以强制类型转换到 SOCKADDR 类型，填写给 bind 函数的参数。这里要注意的是要把 IP 地址和端口号转化为网络存储顺序，即高位存放在低字节内存中，可以用 htonl 和 htons 函数转化*/
SOCKADDR_IN addrSrv;
addrSrv.sin_family = AF_INET;
addrSrv.sin_addr.s_addr = htonl( INADDR_ANY); // INADR_ANY 表示使用默认 IP 地址
addrSrv.sin_port = htons( 27015 );//尽量不要使用在 1024 以下的端口
bind( sockSrv, (SOCKADDR*)&addrSrv, sizeof(addrSrv) );

/*监听，第二个参数表示等待序列的最大个数，就好比最多只能容纳 5 个人排队，第六个就被拒绝排队了。*/
listen( sockSrv, 5 );

/* 接受, 就好比处理房间中第一个人的事务，那么这个队列就空出一个人的位置，其他人还可以来排队。这里通过 accept 函数可以返回一个表示连接进来的客户的 socket，并且通过传递参数可以知道客户端的 IP 地址信息*/
SOCKADDR_IN addrClient;
int len = sizeof(addrClient);
// 循环接受客户，这里没有写退出条件，实际上退出条件可以来自判断所读入的信息
while(1)
{
SOCKET sockClient = accept( sockSrv, (SOCKADDR*)&adrClient, &len );
// 发送和接收数据
char sendbuf[32] = "Server: Sending Data.";
char recvbuf[32] = "";
// recv 和 send 的参数都是 socket，buffer，buffer 的大小以及一个 flag，具体可参考 MSDN
bytesRecv = recv( sockClient, recvbuf, 32, 0 );
printf( "Recv: %s\n", recvbuf );
bytesSent = send( sockClient, sendbuf, strlen(sendbuf), 0 );
}
// 关闭 socket
closesocket(sockClient);
closesocket(sockSrv);
```

**[客户端程序]**

```c
SOCKET sockClient = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP )
SOCKADDR_IN addrSrv;
addrSrv.sin_family = AF_INET;
addrSrv.sin_addr.s_addr = htonl( INADDR_ANY); // INADR_ANY 表示使用默认 IP 地址
addrSrv.sin_port = htons( 27015 );//尽量不要使用在 1024 以下的端口
// 连接
connect( sockSrv, (SOCKADDR*)&addrSrv, sizeof(addrSrv) );
// 发送和接收数据
char sendbuf[32] = "Client: Sending Data.";
char recvbuf[32] = "";
// recv 和 send 的参数都是 socket，buffer，buffer 的大小以及一个 flag，具体可参考 MSDN
bytesRecv = recv( sockClient, recvbuf, 32, 0 );
printf( "Recv： %s\n", recv);
bytesSent = send( sockClient, sendbuf, strlen(sendbuf), 0 );
// 关闭 socket
closesocket(sockClient);
```

### 2. 简单 UDP/IP

思路：

**[服务器程序]**

建立 `socket` -----> 绑定 `bind` ------>发送和接收 `sendto` 和 `recvfrom` ------>关闭 `closesocket`

**[客户端程序]**

建立 `socket` -----> 发送和接收 `send` 和 `recv` ------>关闭 `closesocket`

UDP 的实现更加简单，服务器只要绑定就可以发送接收信息，而客户端只要有 `socket` 就可以了。具体的实现代码可以参考 MSDN 中的例子。

要注意的是：

*   TCP 中用 `SOCK_STREAM` 建立 `socket`，并且使用 `send` 和 `recv` 函数发送接收信息。
*   UDP 中用 `SOCK_DGRAM` 建立 `socket`，并且使用 `sendto` 和 `recvfrom` 来发送和接收信息。`sendto` 比 `send` 多几个参数，主要是对方的 IP 地址信息，`recvfrom` 也是一样比 `recv` 多几个参数。

以上所讨论的仅仅是为了显示一下编写网络应用程序基本思路，至于具体实际的代码还是要看看具体的源代码，如果要使用图形界面则所要用的函数和类会更加多。
