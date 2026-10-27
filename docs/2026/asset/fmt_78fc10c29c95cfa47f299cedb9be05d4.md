
```c
int WSASend(

  __in   SOCKET s,

  __in   LPWSABUF lpBuffers,

  __in   DWORD dwBufferCount,

  __out  LPDWORD lpNumberOfBytesSent,

  __in   DWORD dwFlags,

  __in   LPWSAOVERLAPPED lpOverlapped,

  __in   LPWSAOVERLAPPED_COMPLETION_ROUTINE lpCompletionRoutine);
```

| 参数 | 含义 |
| --- | --- |
| `s` | 套接字句柄，传递具有重叠I/O属性的套接字句柄时，以重叠I/O模型输出 |
| `lpBuffers` | `WSABUF`结构体变量数组的地址，`WSABUF`中存有待传输数据 |
| `dwBufferCount` | `lpBuffers`数组的长度 |
| `lpNumberOfBytesSent` | 用于保存实际发送字节数的变量地址 |
| `dwFlags` | 用于更改数据传输特性，如传递`MSG_OOB`时发送OOB模式的数据 |
| `lpOverlapped` | `WSAOVERLAPPED`结构体变量的地址值，使用事件对象，用于确认完成数据传输 |
| `lpCompletionRoutine` | 传入`Completion Routine`函数的入口地址值，可以通过该函数确认是否完成数据传输 |

其中`WSABUF`结构体定义如下：

```c
typedef struct _WSABUF {

    ULONG len;     /* the length of the buffer */

    __field_bcount(len) CHAR FAR *buf; /* the pointer to the buffer */

} WSABUF, FAR * LPWSABUF;
```

| 成员 | 含义 |
| --- | --- |
| `len` | 待传输数据长度 |
| `buf` | 缓冲区地址值 |

该参数保存的就是需要传输的数据信息，而`WSAOVERLAPPED`结构体的定义如下：

```c
typedef struct _OVERLAPPED {

    ULONG_PTR Internal;

    ULONG_PTR InternalHigh;

    union {

        struct {

            DWORD Offset;

            DWORD OffsetHigh;

        } DUMMYSTRUCTNAME;

        PVOID Pointer;

    } DUMMYUNIONNAME;

    HANDLE  hEvent;

} OVERLAPPED, *LPOVERLAPPED;
```

这里最关键的是`hEvent`句柄，该句柄用来保证数据的完整发送。

> 为了进行重叠I/O，`WSASend`函数的`lpOverlapped`参数中应该传递有效的结构体变量地址值，而不是`NULL`。如果传递的是`NULL`，那么`WSASend`函数的第一个参数中的句柄所指的套接字将以阻塞模式工作

`WSASend`函数调用以后，如果返回值为`SOCKET_ERROR`并且`WSAGetLastError`的值为`WSA_IO_PENDING`就说明此时数据并没有发送完。而要等待数据发送完几句要用到`OVERLAPPED`结构体的`hEvent`，通过调用`WSAWaitFirMultipleEvents`函数并将其事件传入就可以等待发送数据完成，该函数定义如下：

```c
DWORD WSAWaitForMultipleEvents(

  __in  DWORD cEvents,

  __in  const WSAEVENT* lphEvents,

  __in  BOOL fWaitAll,

  __in  DWORD dwTimeout,

  __in  BOOL fAlertable);
```

| 参数 | 含义 |
| --- | --- |
| `cEvents` | `lphEvents`指向的数组中的事件对象句柄数。事件对象句柄的最大数量是`WSA_MAXIMUM_WAIT_EVENTS`事件。必须指定一个或多个事件。 |
| `lpEvents` | 指向事件对象句柄数组的指针。数组可以包含不同类型对象的句柄。如果`fWaitAll`参数设置为`TRUE`，则它可能不包含同一句柄的多个副本。如果其中一个句柄在等待仍处于挂起状态时关闭，则`WSAWaitForMultipleEvents`的行为未定义。句柄必须具有同步访问权限。有关更多信息，请参阅标准访问权限。 |
| `fWaitAll` | 指定等待类型的值。如果为`TRUE`，则当`lphEvents`数组中所有对象的状态都被通知时，函数返回。如果为`FALSE`，则当任何事件对象发出信号时，函数返回。在后一种情况下，返回值减去`WSA_WAIT_EVENT_0`表示其状态导致函数返回的事件对象的索引。如果在调用过程中有多个事件对象发出信号，则这是信号事件对象的数组索引，其索引值在所有信号事件对象中最小。 |
| `dwTimeout` | 超时间隔，以毫秒为单位。如果超时时间间隔过期，即使不满足`fWaitAll`参数指定的条件，`WSAWaitForMultipleEvents`也会返回。如果`dwTimeout`参数为零，`WSAWaitForMultipleEvents`将测试指定事件对象的状态并立即返回。如果`dwTimeout`是`WSA_INFINITE`，那么`WSAWaitForMultipleEvents`将永远等待；也就是说，超时间隔永远不会过期 |
| `fAlertable` | 指定线程是否处于可警报的等待状态，以便系统可以执行I/O完成例程。如果为`TRUE`，线程将处于可变等待状态，并且当系统执行I/O完成例程时，`WSAWaitForMultipleEvents`可以返回。在这种情况下，将返回`WSA_WAIT_IO_COMPLETION`，并且尚未通知正在等待的事件。应用程序必须再次调用`WSAWaitForMultipleEvents`函数。如果为`FALSE`，则线程不会处于可变等待状态，并且不会执行I/O完成例程 |

当该函数成功返回，此时代表数据发送完毕，可以使用`WSAGetOverlappedResult`函数来获取状态，该函数定义如下：

```c
BOOL WSAAPI WSAGetOverlappedResult(

  __in   SOCKET s,

  __in   LPWSAOVERLAPPED lpOverlapped,

  __out  LPDWORD lpcbTransfer,

  __in   BOOL fWait,

  __out  LPDWORD lpdwFlags);
```

| 参数 | 含义 |
| --- | --- |
| `s` | 进行重叠I/O的套接字 |
| `lpOverlapped` | 进行重叠I/O时传递的`WSAOVERLAPPED`结构体变量的地址 |
| `lpcbTransfer` | 用于保存实际传输的字节数的变量地址值 |
| `fWait` | 如果调用该函数时仍在进行I/O，`fWait`为`TRUE`时等待I/O完成，`fWait`为`FALSE`时将返回`FALSE`并跳出函数 |
| `lpdwFlags` | 调用`WSARecv`函数时，用于获取附加信息（例如OOB消息）。如果不需要，可以传递`NULL` |

在重叠I/O模型中，接收数据使用`WSARecv`函数，该函数定义如下：

```c
int WSARecv(

  __in     SOCKET s,

  __inout  LPWSABUF lpBuffers,

  __in     DWORD dwBufferCount,

  __out    LPDWORD lpNumberOfBytesRecvd,

  __inout  LPDWORD lpFlags,

  __in     LPWSAOVERLAPPED lpOverlapped,

  __in     LPWSAOVERLAPPED_COMPLETION_ROUTINE lpCompletionRoutine);
```

| 参数 | 含义 |
| --- | --- |
| `s` | 赋予重叠I/O属性的套接字句柄 |
| `lpBuffers` | 用于保存接收数据的`WSABUF`结构体数组地址值 |
| `dwBufferCount` | 第二个参数传递的数组长度 |
| `lpNumberOfBytesRecvd` | 保存接收数据大小信息的变量地址值 |
| `lpFlags` | 用于设置或读取传输特性信息 |
| `lpOverlapped` | `WSAOVERLAPPED`结构体变量地址值 |
| `lpCompletionRoutine` | `Completion Routine`函数地址 |

参数含义和`WSASend`是一样的所以不重复，到这里就可以实现基于事件的重叠I/O模型，具体代码如下：

服务端：

```cpp
// Server.cpp : 此文件包含 "main" 函数。程序执行将在此处开始并结束。

//

#include <cstdio>

#include <WinSock2.h>

#include <Windows.h>

#pragma comment(lib, "ws2_32.lib")



#define BUF_SIZE 30



VOID ShowError(PCHAR msg);



int main()

{

	WSADATA wsaData;

	SOCKET hServerSocket = INVALID_SOCKET, hClientSocket = INVALID_SOCKET;

	SOCKADDR_IN servAddr, clientAddr;

	int iClientAddrSize = 0;

	

	WSABUF dataBuf;

	WSAEVENT evObj;

	WSAOVERLAPPED overlapped;

	CHAR szBuf[BUF_SIZE] = { 0 };

	DWORD recvBytes = 0, flags = 0;



	// 初始化Winsock库

	if (WSAStartup(MAKEWORD(2, 2), &wsaData) != 0)

	{

		ShowError("WSAStartup");

		goto fail;

	}

	

	// 创建套接字

	hServerSocket = WSASocket(PF_INET, SOCK_STREAM, IPPROTO_TCP, NULL, 0, WSA_FLAG_OVERLAPPED);

	if (hServerSocket == INVALID_SOCKET)

	{

		ShowError("socket");

		goto exit;

	}



	// 绑定端口

	memset(&servAddr, 0, sizeof(servAddr));

	servAddr.sin_family = AF_INET;

	servAddr.sin_addr.S_un.S_addr = htonl(INADDR_ANY);

	servAddr.sin_port = htons(1900);

	if (bind(hServerSocket, (SOCKADDR *)&servAddr, sizeof(servAddr)) == SOCKET_ERROR)

	{

		ShowError("bind");

		goto exit;

	}



	// 监听端口

	if (listen(hServerSocket, SOMAXCONN) == SOCKET_ERROR)

	{

		ShowError("listen");

		goto exit;

	}



	printf("服务端已开启...\n");



	iClientAddrSize = sizeof(clientAddr);

	hClientSocket = accept(hServerSocket, (SOCKADDR *)&clientAddr, &iClientAddrSize);

	printf("新的连接 IP:%s\n", inet_ntoa(clientAddr.sin_addr));



	// 创建等待事件

	evObj = WSACreateEvent();

	memset(&overlapped, 0, sizeof(overlapped));

	overlapped.hEvent = evObj;

	// 初始化数据接收

	dataBuf.len = BUF_SIZE;

	dataBuf.buf = szBuf;



	if (WSARecv(hClientSocket, &dataBuf, 1, &recvBytes, &flags, &overlapped, NULL) == SOCKET_ERROR)

	{

		// 说明此时正在接收数据

		if (WSAGetLastError() == WSA_IO_PENDING)

		{

			// 等待数据接收完毕

			WSAWaitForMultipleEvents(1, &evObj, TRUE, WSA_INFINITE, FALSE);

			// 获取接收状态

			WSAGetOverlappedResult(hClientSocket, &overlapped, &recvBytes, FALSE, NULL);

		}

		else

		{

			ShowError("WSARecv");

		}

	}



	printf("receive message: %s\n", szBuf);

	WSACloseEvent(evObj);



exit:

	if (hServerSocket != INVALID_SOCKET)

	{

		closesocket(hServerSocket);

		hServerSocket = INVALID_SOCKET;

	}

	if (hClientSocket != INVALID_SOCKET)

	{

		closesocket(hClientSocket);

		hClientSocket = INVALID_SOCKET;

	}

	WSACleanup();

fail:

	system("pause");

	return 0;

}



VOID ShowError(PCHAR msg)

{

	printf("%s Error %d\n", msg, WSAGetLastError());

}
```

客户端：
