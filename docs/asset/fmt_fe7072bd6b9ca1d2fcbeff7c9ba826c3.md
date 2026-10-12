
# 发个程序权当保存在论坛里

> 原文：[`www.pediy.com/kssd/pediy11/123315.html`](https://www.pediy.com/kssd/pediy11/123315.html)

一直对网络编程不太熟悉，结果吃大亏了。第一次应聘被默拒了。

定下心来写了这个个程序。

一个猜数游戏，C/S 模型。这是其中一个题，我当时没做出来，就因为不熟悉网络编程。

他当时不然参考资料，更别说 MSDN 了。

服务器：

```c
#include <winsock2.h>
#pragma comment(lib,"ws2_32")
#include <stdio.h>
#include <stdlib.h>
#include <windows.h>
#include <time.h>

int main(int argc, char * argv[])
{
  srand(time(NULL));
  WSADATA wsaData;
  WORD sockVersion = MAKEWORD(2,2);

  //加载 WinSock 库
  if(WSAStartup(sockVersion,&wsaData) != 0)
  {
    printf("加载 WinSock 库失败");
    return 0;
  }

  //创建套接字
  SOCKET sListen = socket(AF_INET,SOCK_STREAM,IPPROTO_TCP);
  if(sListen == INVALID_SOCKET)
  {
    printf("socket error");
    return 0;
  }

  //在 sockaddr_in 结构中装入地址信息
  sockaddr_in sin;
  sin.sin_family = AF_INET;
  sin.sin_port = htons(4500);
  sin.sin_addr.S_un.S_addr = INADDR_ANY;

  //使用套接字和本地地址绑定
  if(bind(sListen,(LPSOCKADDR)&sin,sizeof(sin)) == SOCKET_ERROR)
  {
    printf("bind error");
    closesocket(sListen);
    return 0;
  }

  //设置套接字进入监听模式
  if(listen(sListen,1) == SOCKET_ERROR)
  {
    printf("listen error\n");
    closesocket(sListen);
    return 0;
  }

  //接受客户请求///暂定一个客户
  sockaddr_in remoteAddr;
  SOCKET sClient;
  int nAddrLen = sizeof(remoteAddr);

  //接受一个连接
  sClient = accept(sListen,(SOCKADDR*)&remoteAddr,&nAddrLen);
  if(sClient == INVALID_SOCKET)
  {
    printf("accept error\n");
    return 0;
  }

  BYTE key = rand()%26 + 'a';
  BYTE ans;
  BYTE flag = 1;//状态:0 成功;1 就位;2 附件;3 太远
  int ret;
  WORD rev;//两个字节
  WORD sen;
  char test[6] = {0};
  char revData[6];

  printf("***********游戏开始***********\n");

  while(flag)////成功否
  {
    //发送问题
    memset(test,0,6);
    sen = MAKEWORD(key,flag);
    wsprintf(test,"%d",sen);
    ret = send(sClient,test,6,0);  //发问题或者反馈回答情况
    Sleep(400);
    memset(revData,0,6);
    ret = recv(sClient,revData,6,0);
    rev = atoi(revData);
    flag = HIBYTE(rev);
    ans = LOBYTE(rev);

    if(flag == 1)
    {
      if(ans == key)
      {
        flag = 0;
        sen = MAKEWORD(key,flag);
        memset(test,0,6);
        wsprintf(test,"%d",sen);
        ret = send(sClient,test,6,0);
        printf("!!!!!!!!果然很强大!!!!!!\n\n");
      }
      else if(ans < key + 3 || ans > key - 3)
      {
        flag = 2;
        printf("!!!!!!!貌似很强大!!!!!!\n\n");
      }
      else
      {
        flag = 3;
        printf("革命尚未成功，同志仍需努力!\n");
      }
    }
  }

  printf("就此为止!\n");
  getchar();
  closesocket(sClient);
  WSACleanup();
  return 0;
}
```

客户端：

```c
#include <winsock2.h>
#pragma comment(lib,"ws2_32")
#include <stdio.h>
#include <stdlib.h>
#include <windows.h>
#include <time.h>

int main()
{
  srand(time(NULL));
  WSADATA wsaData;
  WORD sockVersion = MAKEWORD(2,2);

  //加载 Winsock 库
  if(WSAStartup(sockVersion,&wsaData) != 0)
  {
    printf("加载 WinSock 库失败!\n");
    return 0;
  }

  //创建套接字
  SOCKET sClient = socket(AF_INET,SOCK_STREAM,IPPROTO_TCP);
  if(sClient == INVALID_SOCKET)
  {
    printf("socket error\n");
    return 0;
  }

  //在 sockaddr_in 结构中装入服务器地址信息
  sockaddr_in servAddr;
  servAddr.sin_family = AF_INET;
  servAddr.sin_port = htons(4500);
  servAddr.sin_addr.S_un.S_addr = inet_addr("127.0.0.1");

  if(connect(sClient,(sockaddr*)&servAddr,sizeof(servAddr)) == SOCKET_ERROR)
  {
    printf("connect error\n");
    closesocket(sClient);
    return 0;
  }

  WORD rev;
  char *buff = "hello server!\n";
  char revData[6];
  char test[6];
  BYTE flag = 1;
  BYTE key;
  WORD sen;

  printf("^^^^^^^^^^^^^^^^游戏开始^^^^^^^^^^^^^^^^^^^\n\n");

  while(flag)
  {
    memset(revData,0,6);
    int ret = recv(sClient,revData,6,0);
    rev = atoi(revData);
    flag = HIBYTE(rev);
    key = LOBYTE(rev);

    if(flag == 0)
    {
      printf("\n 恭喜你，回答正确![](img/cda69129882b40377a87144c782680e1.jpg "微笑")):)!\n");
    }
    else if(flag == 1)
    {
      //key 已就位，开始猜测
      printf("开始了！:)\n");
      key = rand()%26 + 'a';
    }
    else if(flag == 2)
    {
      printf("\n 嗯，差一点就对了:)!\n");
      key = key + (rand() > 0?-1:1) * (rand()%3);
    }
    else if(flag == 3)
    {
      printf("\n 嗯，继续努力!: (\n");
      key = rand()%26 + 'a';
    }
    else
    {
      printf("\n 出现异常\n");
    }

    sen = MAKEWORD(key,1);
    memset(test,0,6);
    wsprintf(test,"%d",sen);
    ret = send(sClient,test,6,0);
    Sleep(400);
  }

  printf("\n........游戏结束!......\n");
  getchar();
  closesocket(sClient);
  WSACleanup();
  return 0;
}
```

![](img/be210d06d9dcb2f784a29683554f2aba.jpg)
