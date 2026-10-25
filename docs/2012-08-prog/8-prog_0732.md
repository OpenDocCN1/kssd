
# 深入剖析 `CSocket` 类工作细节原理

> 原文：[`www.pediy.com/kssd/pediy12/130688.html`](https://www.pediy.com/kssd/pediy12/130688.html)

注：本文适合有一定 `Winsock` 编程和 `MFC` 编程基础的朋友阅读。

我是一个对一切事情都爱探究到底的人，前一段由于需要使用 `MFC` 开发一个网络通信程序，于是就顺便研究了一下 `CSocket` 类的工作工程，现将我的一些学习成果公布出来，希望大家多多指正。

先看 `CSocket` 的 `Create` 函数，它调用了基类 `CAsyncSocket::Create` 函数，下面跟进去看到：

```cpp
BOOL CAsyncSocket::Create(UINT nSocketPort, int nSocketType, long lEvent, LPCTSTR lpszSocketAddress)
{
    if (Socket(nSocketType, lEvent))
    {
        if (Bind(nSocketPort, lpszSocketAddress))
            return TRUE;
        int nResult = GetLastError();
        Close();
        WSASetLastError(nResult);
    }
    return FALSE;
}
```

先调用 `Socket` 函数，在此函数中先调用 API 函数 `socket` 创建一个套接字并返回给 `m_hSocket`，如果创建成功就进行 `CAsyncSocket::AttachHandle`，实现套接字句柄与 `CSocket` 类对象的绑定，在绑定函数中创建了一个 `CSocketWnd` 对象，并创建一个窗口但并不显示，我称之为 `socket` 窗口，在创建之前有个判断 `pState->m_pmapSocketHandle->IsEmpty()`，作用是检查 `socket` 的绑定映射是否为空，如果是空就不再创建 `socket` 窗口了，目的是节省资源，也就是保证只有一个 `socket` 窗口。`socket` 窗口用于接收网络消息，并处理消息响应。接下来调用 `CAsyncSocket::AsyncSelect` 函数，其中又调用了 `WSAAsyncSelect` 函数，创建异步套接字，即当一些网络行为发生时向 `socket` 窗口发送 `WM_SOCKET_NOTIFY` 消息，然后消息处理程序在进行分类处理。这是 `CSocket` 类工作的核心。这里注册了这些事件 `FD_READ | FD_WRITE | FD_OOB | FD_ACCEPT | FD_CONNECT | FD_CLOSE`，详细含义大家可参考 MSDN。即这些事件发生时向 `socket` 窗口发送 `WM_SOCKET_NOTIFY` 消息。接下来的 `Bind` 函数无非就是做一些初始化并调用 API 函数 `bind`，如果有一些基础大家都能明白。如果出错就关闭套接字，成功就返回。

接下来按标准过程就该监听调用 `CSocket::Listen` 函数，此函数很简单，就是调用 API 函数 `listen`。

下面我来说核心的另一方面，消息处理过程。`socket` 窗口接到 `WM_SOCKET_NOTIFY` 消息，根据消息映射，调用 `CSocketWnd::OnSocketNotify`，其中调用静态函数 `CSocket::ProcessAuxQueue`，经过一些判断保护，调用了 `CAsyncSocket::DoCallBack`，同样他也是个静态函数。

这个函数最关键，我把代码放上来。详细解释我放在注释里。

```cpp
void PASCAL CAsyncSocket::DoCallBack(WPARAM wParam, LPARAM lParam)
{
    if (wParam == 0 && lParam == 0)
        return;
        
    CAsyncSocket* pSocket = CAsyncSocket::LookupHandle((SOCKET)wParam, TRUE); 
    // 根据 WM_SOCKET_NOTIFY 消息的 wParam 参数寻找对应 CSocket 对象
    // 这是一个保护，如果对象为空直接返回。
    // 因为在异步模式下，每发出一回动作，就会发出 WM_SOCKET_NOTIFY 消息，
    // 如果连续发出读操作，然后关闭套接字。那么当处理多余的消息时套接字已不存在，那么就直接返回。
    
    if (pSocket != NULL)
        return;

    pSocket = CAsyncSocket::LookupHandle((SOCKET)wParam, FALSE);
    if (pSocket == NULL)
    {
        pSocket = CAsyncSocket::LookupHandle(INVALID_SOCKET, FALSE);
        ASSERT(pSocket != NULL);
        if (pSocket == NULL)
            return;
        pSocket->m_hSocket = (SOCKET)wParam;
        CAsyncSocket::DetachHandle(INVALID_SOCKET, FALSE);
        CAsyncSocket::AttachHandle(pSocket->m_hSocket, pSocket, FALSE);
    }
    
    int nErrorCode = WSAGETSELECTERROR(lParam);
    
    // 根据 lParam 的内容分类处理不同事件的操作，并在其中调用程序员自己编写的处理函数如 OnReceive 等。
    switch (WSAGETSELECTEVENT(lParam))
    {
    case FD_READ:
        {
            fd_set fds;
            int nReady;
            timeval timeout;
            timeout.tv_sec = 0;
            timeout.tv_usec = 0;
            FD_ZERO(&fds);
            FD_SET(pSocket->m_hSocket, &fds);
            nReady = select(0, &fds, NULL, NULL, &timeout);
            if (nReady == SOCKET_ERROR)
                nErrorCode = WSAGetLastError();
            if ((nReady == 1) || (nErrorCode != 0))
                pSocket->OnReceive(nErrorCode);
        }
        break;
    case FD_WRITE:
        pSocket->OnSend(nErrorCode);
        break;
    case FD_OOB:
        pSocket->OnOutOfBandData(nErrorCode);
        break;
    case FD_ACCEPT:
        pSocket->OnAccept(nErrorCode);
        break;
    case FD_CONNECT:
        pSocket->OnConnect(nErrorCode);
        break;
    case FD_CLOSE:
        pSocket->OnClose(nErrorCode);
        break;
    }
}
```

在 `CSocket::Accept` 函数即接受连接函数中，就是调用 API 函数 `accept` 这么简单。

下面一点也比较关键，也就是在异步套接字中进行操作 `receive` 等操作时如何实现同步即阻塞。

看代码：

```cpp
int CSocket::Receive(void* lpBuf, int nBufLen, int nFlags)
{
    // m_pbBlocking 是 CSocket 的成员变量，用来标识当前是否正在进行
    // 阻塞操作。但不能同时进行两个阻塞操作。
    if (m_pbBlocking != NULL)
    {
        WSASetLastError(WSAEINPROGRESS);
        return FALSE;
    }
    
    // 完成数据读取
    int nResult;
    while ((nResult = CAsyncSocket::Receive(lpBuf, nBufLen, nFlags)) 
           == SOCKET_ERROR)
    {
        if (GetLastError() == WSAEWOULDBLOCK)
        {
            // 进入消息循环，等待网络事件 FD_READ
            if (!PumpMessages(FD_READ))
                return SOCKET_ERROR;
        }
        else
            return SOCKET_ERROR;
    }
    return nResult;
}
```

`Receive` 函数首先判断当前 `CSocket` 对象是否正在处理一个阻塞操作，如果是，则返回错误 `WSAEINPROGRESS`；否则，开始数据读取的处理。读取数据时，如果基类 `CAsyncSocket` 的 `Receive` 读取到了数据，则返回；否则，如果返回一个错误，而且错误号是 `WSAEWOULDBLOCK`，则表示操作阻塞，于是调用 `PumpMessages` 进入消息循环等待数据到达（网络事件 `FD_READ` 发生）。数据到达之后退出消息循环，再次调用 `CAsyncSocket` 的 `Receive` 读取数据，直到没有数据可读为止。`PumpMessages` 函数中其实就是不断调用 `PeekMessage` 函数，直到取到希望的消息时返回。

当 `CSocket` 对象析构时会自动调用 `Close` 函数关闭套接字或主动关闭。其实其中调用的是 `CAsyncSocket::Close`，又调用 `CAsyncSocket::KillSocket`，其中又调用 `CAsyncSocket::DetachHandle` 将对象与套接字分离，并关闭套接字。

好吧，这些已经将 `CSocket` 类讲的差不多了，其中的几个关键点也讲明白了，希望大家多多关注，谢谢大家的支持，初次写文章，希望大家多多谅解。
