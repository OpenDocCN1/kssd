
# [原创]NtSocket的稳定实现，Client与Server的简单封装，以及SocketAsyncSelect的一种APC实现

## NtSocket_NtClient_NtServer：

- Using `NtCreateFile` and `NtDeviceIoControlFile` to realize the function of winsock
- 利用`NtCreateFile`和`NtDeviceIoControlFile`实现winsock的功能。
- 部分功能参考ReactOS源码实现。
- 现在已经成功封装出了一个`NtClient`和一个`NtServer`。
- 源码主体实现了以下函数：
- `WSPSocket`（创建socket句柄）
- `WSPCloseSocket`（关闭socket）
- `WSPBind`（Bind一个地址，支持IPv6）
- `WSPConnect`（连接一个地址，大致用法同`Connect`函数，支持IPv6）
- `WSPShutdown`（Shutdown指定操作，可以同`WSAShutdown`使用）
- `WSPListen`（基本同`Listen`函数）
- `WSPRecv`（接收数据，基本同`Recv`）
- `WSPSend`（发送数据，基本同`send`）
- `WSPGetPeerName`（基本同`GetPeerName`，支持IPv6）
- `WSPGetSockName`（基本同`GetSockName`，支持IPv6）
- `WSPEventSelect`（事件选择模型，基本同`WSAEventSelect`）
- `WSPEnumNetworkEvents`（事件选择模型，基本同`WSAEnumNetworkEvents`）
- `WSPAccept`（基本完整实现`accept`功能）
- `WSPProcessAsyncSelect`（本源码最大的亮点，APC异步Select模型，这是winsock没有开放的模型，IOCP模型本质上也依赖这个完成）

## 前言：

- 本程序只能使用于Windows。
- 本程序开发环境：Win11 x64.
- 本程序理论上兼容x64和x86环境，不过具体出现问题还得具体分析。

## 为什么写它？

- 之前看了一篇文章：[NTSockets - Downloading a file via HTTP using the `NtCreateFile` and `NtDeviceIoControlFile` syscalls](<mark class="encrypted">485K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2^5z5o6k6E0j5i4c8@1K9r3g2%4i4K6u0W2j5$3!0E0i4K6u0r3N6X3W2W2N6#2)9#2k6Y4m8G2M7%4c8Q4x3@1k6A6k6q4)9K6c8r3&6@1M7$3!0U0K9$3g2@1M7H3`.`.</mark> "NTSockets - Downloading a file via HTTP using the NtCreateFile and NtDeviceIoControlFile syscalls")
- 我觉得这篇文章非常有意思，**直接利用`NtCreateFile`和Afd驱动建立通信，然后利用`NtDeviceIoControlFile`实现向afd发送socket控制的信息**。
- 不过美中不足的是这个程序支持x86（原因是结构体定义的问题），同时也仅实现了Client，其他部分还没有完善。
- 于是便有了这个程序。

## 正文：

- 最基础的内容，利用`NtCreateFile`创建socket句柄：

```
SOCKET WSPSocket(
int AddressFamily,
int SocketType,
int Protocol) {
/// <summary>
/// 类似于Socket函数，可以创建一个Socket文件句柄
/// </summary>
/// <param name="AddressFamily">Address family(Support IPv6)</param>
/// <param name="SocketType">Socket Type</param>
/// <param name="Protocol">Protocol type</param>
/// <returns>如果失败返回INVALID\_SOCKET，成功返回Socket文件句柄</returns>
if (AddressFamily == AF\_UNSPEC && SocketType == 0 && Protocol == 0) {
return INVALID\_SOCKET;
}
//进行基础数据设置
if (AddressFamily == AF\_UNSPEC) {
AddressFamily = AF\_INET;
}
if (SocketType == 0)
{
switch (Protocol)
{
case IPPROTO\_TCP:
SocketType = SOCK\_STREAM;
break;
case IPPROTO\_UDP:
SocketType = SOCK\_DGRAM;
break;
case IPPROTO\_RAW:
SocketType = SOCK\_RAW;
break;
default:
SocketType = SOCK\_STREAM;
break;
}
}
if (Protocol == 0)
{
switch (SocketType)
{
case SOCK\_STREAM:
Protocol = IPPROTO\_TCP;
break;
case SOCK\_DGRAM:
Protocol = IPPROTO\_UDP;
break;
case SOCK\_RAW:
Protocol = IPPROTO\_RAW;
break;
default:
Protocol = IPPROTO\_TCP;
break;
}
}
byte EaBuffer[] = {0x00, 0x00, 0x00, 0x00, 0x00, 0x0F, 0x1E, 0x00,
0x41, 0x66, 0x64, 0x4F, 0x70, 0x65, 0x6E, 0x50,
0x61, 0x63, 0x6B, 0x65, 0x74, 0x58, 0x58, 0x00,
0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
0x02, 0x00, 0x00, 0x00, 0x01, 0x00, 0x00, 0x00,
0x06, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
0x08, 0x01, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00};
memmove((PVOID)((\_\_int64)EaBuffer + 32), &AddressFamily, 0x4);
memmove((PVOID)((\_\_int64)EaBuffer + 36), &SocketType, 0x4);
memmove((PVOID)((\_\_int64)EaBuffer + 40), &Protocol, 0x4);
if (Protocol == IPPROTO\_UDP)
{
memmove((PVOID)((\_\_int64)EaBuffer + 24), &Protocol, 0x4);
}
//初始化UNICODE\_STRING：
UNICODE\_STRING AfdName;
AfdName.Buffer = L"\\Device\\Afd\\Endpoint";
AfdName.Length = 2 \* wcslen(AfdName.Buffer);
AfdName.MaximumLength = AfdName.Length + 2;
OBJECT\_ATTRIBUTES Object;
IO\_STATUS\_BLOCK IOSB;
//初始化OBJECT\_ATTRIBUTES
InitializeObjectAttributes(&Object,
&AfdName,
OBJ\_CASE\_INSENSITIVE | OBJ\_INHERIT,
0,
0);
HANDLE MySock;
NTSTATUS Status;
//创建AfdSocket：
Status = ((NtCreateFile)MyNtCreateFile)(&MySock,
GENERIC\_READ | GENERIC\_WRITE | SYNCHRONIZE,
&Object,
&IOSB,
NULL,
0,
FILE\_SHARE\_READ | FILE\_SHARE\_WRITE,
FILE\_OPEN\_IF,
0,
EaBuffer,
sizeof(EaBuffer));
if (Status != STATUS\_SUCCESS) {
return INVALID\_SOCKET;
}
else {
return (SOCKET)MySock;
}
}
```

- 从总体上看来，这部分代码是最简单的，用`NtCreateFile`创建`\\Device\\Afd\\Endpoint`的连接，麻烦一点的可能是传递的`EaBuffer`，不过这部分可以直接通过CE拦截的方式获取数据，也可以参考ReactOS的结构体，我选择了一种介于两者之间的方法实现。

- 这部分的兼容性也是最难实现的，因为不知道不同的系统的结构体会不会有差异，不过我测试基本是没有问题的。

- 创建出socket句柄后，剩下的事情就非常简单了，只需要根据不同的`IOCTL_AFD_`写出各个函数即可。

- 最有意思的部分其实是`WSPProcessAsyncSelect`函数。

- 这个需要从`WSPAsyncSelect`函数说起

- 使用IDA看一下`mswsock.WSPAsyncSelect`函数，大概长这个样子：![](upload/attach/202209/962506\_V9P56K88K4S8Z7J.jpg)

- 其中最重要的函数是`SockCheckAndInitAsyncSelectHelper`：![](upload/attach/202209/962506\_YEA8QCJPXEVDZ6G.jpg)

- `SockCheckAndInitAsyncSelectHelper`函数先创建了一个`\\Device\\Afd\\AsyncSelectHlp`的IO（功能未知），**然后用`SockCreateAsyncQueuePort`函数创建了一个完成端口**

- 接着用`NtSetInformationFile`函数将完成端口和`\\Device\\Afd\\AsyncSelectHlp`绑定。

- 在`SockAsyncSelectCompletion`函数中找到`SockProcessAsyncSelect`：![](upload/attach/202209/962506\_WBSBUR8ATDAYCAJ.jpg)

- 在`SockProcessAsyncSelect`中找到AsyncSelect真正实现原理：![](upload/attach/202209/962506\_34K4P9RFST3Z9BG.jpg)

- （别问我怎么找的，参看ReactOS源码就会很清晰）

- 图中的操作应该是轮询实现的，效率比较低。

- 不过别忘了，`NtDeviceIoControlFile`是提供了`APCRoutine`接口的。

- **实际上只需要给`NtDeviceIoControlFile`以`APCRoutine`接口，就可以实现无需轮询的AsyncSelect**。

- 大致实现：

```
NTSTATUS WSPProcessAsyncSelect(
SOCKET Handle,
PVOID ApcRoutine,
ULONG lNetworkEvents,
PVOID UserContext
) {
/// <summary>
/// WSPProcessAsyncSelect是本程序最大的亮点，利用异步化的IOCTL\_AFD\_SELECT
/// </summary>
/// <param name="Handle"></param>
/// <param name="ApcRoutine"></param>
/// <param name="lNetworkEvents"></param>
/// <returns></returns>
AFD\_AsyncData\* AsyncData = (AFD\_AsyncData\*)malloc(sizeof(AFD\_AsyncData));
if (AsyncData == NULL)
{
return -1;
}
memset(AsyncData, 0, sizeof(AFD\_AsyncData));
AsyncData->NowSocket = Handle;
AsyncData->PollInfo.Timeout.HighPart = 0x7FFFFFFF;
AsyncData->PollInfo.Timeout.LowPart = 0xFFFFFFFF;
AsyncData->PollInfo.HandleCount = 1;
AsyncData->PollInfo.Handle = Handle;
AsyncData->PollInfo.Events = lNetworkEvents;
AsyncData->UserContext = UserContext;
NTSTATUS Status;
Status = ((NtDeviceIoControlFile)(MyNtDeviceIoControlFile))((HANDLE)Handle,
NULL,
ApcRoutine,
AsyncData,
&AsyncData->IOSB,
IOCTL\_AFD\_SELECT,
&AsyncData->PollInfo,
sizeof(AFD\_PollInfo),
&AsyncData->PollInfo,
sizeof(AFD\_PollInfo));
return Status;
}
```

- 既然已经做出了基础函数，剩下的就是封装了。

- 我封装了“`WSPClient`”和“`WSPServer`”两个类（一个客户端，一个服务端）

- 虽然功能不是很齐全，但是至少是能用的。

- 主体说一下`WSPServer`的AsyncSelect：

- 核心源码：
