
# [分享][原创]Windows模型——消息异步；

## 客户端为C++语言，服务端为Pascal语言——网络模型之消息异步；（帖源码）

### 1，消息异步的话，顾名思义，是基于消息传递信号，而响应相相对应事件的网络模型。因为我只需要客户端仅仅听从于服务端一个人的命令，所以C++的模型为阻塞模式；

### 2，两端采用类封装，简化代码流程；

### 3，代码封装成流，以便于接收命令码、错误码和数据；以保证数据准确；

## 客户端（C++）

### 头文件：`CSocket.h`

```
//需要包含Winsock库
//#include <WinSock2.h>
//#pragma comment(lib,"ws2\_32.lib")
//利用类简化流程，封装成流，以便于接收命令码和错误码还有数据
class CSocket

{

public:

CSocket();

CSocket(SOCKET s);

~CSocket(void);



public:

int Create();

int Close();

int Connect(char* host, WORD port);

int SoSendStream(DWORD  ucmd, DWORD  uerr, CStream* buf);

int SoRecvStream(DWORD& ucmd, DWORD& uerr, CStream* buf);



private:

 

int SoSendBytes( char* buf, int len);

int SoRecvBytes( char* buf, int len);



int SoRecv(char* buf, int len, int& cb);

int SoSend(char* buf, int len, int& cb);



private:

SOCKET m_s;

};
```

### CPP源码：

```

#include "StdAfx.h"

#include "CSocket.h"



CSocket::CSocket()

{

m_s = INVALID_SOCKET;

}

CSocket::~CSocket()

{

if (int e = Close())

{

printf("%d\r\n",GetLastError());

}



}



int CSocket::Create()

{

m_s = socket(AF_INET, SOCK_STREAM, 0);



if (INVALID_SOCKET == m_s)

{

return WSAGetLastError();

}



return 0;

}



int CSocket::Close()

{



if (INVALID_SOCKET != m_s)

{

if (closesocket(m_s))

{

printf("%d\r\n",GetLastError());  return WSAGetLastError();

}



m_s = INVALID_SOCKET;



}

return 0;



}

int CSocket::Connect(char* host, WORD port)

{

SOCKADDR_IN addrSrv;



addrSrv.sin_addr.S_un.S_addr = inet_addr(host);

addrSrv.sin_family = AF_INET;

addrSrv.sin_port = htons(port);



if (connect(m_s, (SOCKADDR*)&addrSrv, sizeof(SOCKADDR)))

{

return WSAGetLastError();

}



DWORD timeout = 300 * 1000;



if (setsockopt(m_s,SOL_SOCKET,SO_RCVTIMEO,(char*)&timeout,sizeof(timeout)))

{

printf("%d\r\n",GetLastError());

}



if (setsockopt(m_s,SOL_SOCKET,SO_SNDTIMEO,(char*)&timeout,sizeof(timeout)))

{

printf("%d\r\n",GetLastError());

}



return 0;



}



int CSocket::SoRecv(char* buf, int len, int& cb)

{

cb = recv(m_s, buf, len, 0);



if (cb > 0)

{

return 0;

}



if (cb < 0)

{

printf("%d\r\n",GetLastError());//WSAGetLastError();

}

return WSAECONNRESET;

}



int CSocket::SoSend(char* buf, int len, int& cb)

{

cb = send(m_s, buf, len, 0);



if (cb > 0)

{

return 0;

}

if (cb < 0)

{

printf("%d\r\n",GetLastError()); return WSAGetLastError();

}



return WSAECONNRESET;

}



int CSocket::SoSendBytes(char * buf, int len)

{



int off = 0;



while (off < len)

{

int cb = 0;



if (int e = SoSend((char*)buf + off, len - off, cb))

{

return e;

}



off += cb;



}



return 0;



}

int CSocket::SoRecvBytes(char * buf, int len)

{

int off = 0;



while (off < len)

{

int cb = 0;

if (int e = SoRecv((char*)buf + off, len - off, cb))

{

return e;

}



off += cb;



}



return 0;



}



int CSocket::SoSendStream(DWORD ucmd, DWORD uerr, CStream * buf)

{

if (int e = SoSendBytes((char*)&buf->m_size, sizeof(buf->m_size)))

{

return e;

}



if (int e = SoSendBytes((char*)&ucmd, sizeof(ucmd)))

{

return e;

}

if (int e = SoSendBytes((char*)&uerr, sizeof(uerr)))

{

return e;

}

if (int e = SoSendBytes((char*)buf->m_p, buf->m_size))

{

return e;

}



return 0;



}

 

int CSocket::SoRecvStream(DWORD& ucmd, DWORD& uerr, CStream* buf)

{

int size;

if (int e = SoRecvBytes((char*)&size, sizeof(size)))

{

return e;

}

 

DBG_ASSERT((size >= 0) && (size < 1024 * 1024), 0);

DBG_ASSERT((size >= 0) && (size < 1024 * 1024), 0);

DBG_ASSERT((size >= 0) && (size < 1024 * 1024), 0);



if (int e = SoRecvBytes((char*)&ucmd, sizeof(ucmd)))

{

return e;

}



if (int e = SoRecvBytes((char*)&uerr, sizeof(uerr)))

{

return e;

}

if (int e = buf->SetSize(size))

{

printf("%d\r\n",GetLastError()); // return e;

}



if (int e = SoRecvBytes((char*)buf->m_p, buf->m_size))

{

return e;

}



return 0;

}
```

## 服务端（Pascal）：
