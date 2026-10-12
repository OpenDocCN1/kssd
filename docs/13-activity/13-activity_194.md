
# 看雪论坛辞旧迎新 Exploit Me 挑战赛_ 挑战赛发榜

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-760.htm`](https://www.pediy.com/kssd/pediy09/pediy09-760.htm)

> To be the apostrophe which changed "Impossible" into "I'm possible"!

公司事情太多，发榜的时间稍微晚了几天，大家见谅。

这次挑战赛提交贴共 57 个

其中 3 人两道题一起提交：黑色猎鹰，GYGY，netwind

一个测试帖，看雪发

一个超时贴，最后一个

一个水贴！

一个弃权帖。

A 题有效提交共 32 个

B 题有效提交共 24 个

A 题是我在《0day 安全：软件漏洞分析技术》中为了介绍 Metasploit 编程而设计的一个很简单的例子。这种漏洞代表了一大类漏洞利用场景：服务器漏洞。

IIS 解析 HTTP 请求的时候，RPC 服务的函数参数，FTP 服务器解析畸形命令时都有可能发生这类漏洞。应该说服务器类漏洞的影响最严重，能够引起主动入侵。

在出题的时候，和看雪反复商量过关于难度的问题，最后我们把挑战赛定位在让初学者体会漏洞分析的层次上，希望有更多的朋友在读过教程后，能自己动手独立完成一次 POC 的构造。开始的时候曾经想过加点简单的协议进去，让大家 fuzz 一下，增加点漏洞定位的难度，后来干脆把 A 题当作热身的送分题目了。

A 的源代码如下：

```
 /**************************************************************************
      To be the apostrophe which changed "Impossible" into "I'm possible"!
      
POC code of chapter 10.6 in book "Vulnerability Exploit and Analysis Technique"
  
file name  : target_server.cpp
author    : failwest  
date    : 2007.4.4
description  : TCP server which got a stack overflow bug for exploit practice  
Noticed    : Complied with VC 6.0 and build into release version are recommend
version  : 1.0
E-mail    : failwest@gmail.com
  
  Only for educational purposes    enjoy the fun from exploiting :)
**************************************************************************/
#include<iostream.h>
#include<winsock2.h>
#pragma comment(lib, "ws2_32.lib")
void msg_display(char * buf)
{
  char msg[200];
  strcpy(msg,buf);// overflow here, copy 0x200 to 200
  cout<<"********************************"<<endl;
  cout<<"received:"<<endl;
  cout<<msg<<endl;
}
void main()
{
  int sock,msgsock,lenth,receive_len;
  struct sockaddr_in sock_server,sock_client;
  char buf[0x200]; //noticed it is 0x200
  
  WSADATA wsa;
  WSAStartup(MAKEWORD(1,1),&wsa);
  if((sock=socket(AF_INET,SOCK_STREAM,0))<0)
  {
    cout<<sock<<"socket creating error!"<<endl;
    exit(1);
  }
  sock_server.sin_family=AF_INET;
  sock_server.sin_port=htons(7777);
  sock_server.sin_addr.s_addr=htonl(INADDR_ANY);
  if(bind(sock,(struct sockaddr*)&sock_server,sizeof(sock_server)))
  {
    cout<<"binding stream socket error!"<<endl;
  }
  cout<<"***********************************"<<endl;
  cout<<"     exploit target server 1.0     "<<endl;
  cout<<"***********************************"<<endl;
  listen(sock,4);
  lenth=sizeof(struct sockaddr);
  do{
    msgsock=accept(sock,(struct sockaddr*)&sock_client,(int*)&lenth);
    if(msgsock==-1)
    {
      cout<<"accept error!"<<endl;
      break;
    }
    else  
      do
      {
        memset(buf,0,sizeof(buf));
        if((receive_len=recv(msgsock,buf,sizeof(buf),0))<0)
        {
          cout<<"reading stream message erro!"<<endl;
          receive_len=0;  
        }
        msg_display(buf);//trigged the overflow
      }while(receive_len);
      closesocket(msgsock);
  }while(1);
  WSACleanup();
}
```

实在太简单了，不用多说。开了 200 字节，使用了 0x200 字节是问题的根源。最快的提交是在题目公布后 40 分钟左右交上来的。

好几个朋友用了 python 脚本，大多数用了 C 语言写了个简单的 soket 发送程序。没有见到写成 MSF 模块的。

比赛主要评判的标准是分析报告。A 题首先要把缓冲区大小是多少说清楚吧。

稳定性来源于两个方面，定位 shellcode 和 shellcode 本身

大多数朋友使用了我给出的那段动态定位 API 的 shellcode，把显示的名字改了一下
也有朋友使用 Metasploit 生成 shellcode

不管怎样，shellcode 都是比较稳定的。所以在我的环境下没有通过的 exploit 大多数是因为跳板地址选的不够通用。

跳板地址需要根据具体情况来定。针对这个 A 题，即便不用跳板，其实也应该是比较稳定的。

如果选的话，A 又没有版本之分，如果能在 A 自身代码区找个跳板出来，当然万无一失了。

在往后就是用比较通用的 4G 进程空间里的其他跳板。我要说 `user32.dll`，`kernel32.dll`，`ntdll` 都不是好的选择。《网络渗透技术》中曾经提到过跳板选择标准，语言区的往往比较稳定。在提交答卷中，有一些跳板选的就很好。

考虑到大家在自己机器上调通就应该给予肯定，所以运行截图的分数被我调高了，虚拟机验证部分存在一定偶然性，我降低了一些分值。

B 题是 shineast 同学自己挖出来的一个 0day，哪个 0day 大家应该已经知道了。比赛前我和看雪已经正式通知了该公司。

我们在 PE 里做了一些修改，但是忘了 CLSID 这回事，不少朋友在研究的过程中已经发现了网上的 exploit。

我们并不排斥参考网上 exploit 的朋友，因为想到搜索一下本身就值得加分，Google hacking 现在不是很火么：）但我们主要看得是分析报告，是思路。而且这个 ActiveX 中不止一个溢出漏洞，网上给出的只是其中一个的利用。

shineast 在《黑客防线》上曾经写过这个 0day 的分析，但是那篇文章是在网上搜不到的（至少我们没有搜到）。如果你恰巧买了那期杂志，而且精读过练习过，那可就占了点便宜，好像考试的时候最后一道大题恰巧在某本习题集上被撞到一样。

这个因素我们也考虑过。我认为如果你真的很关注这些东西，那么这便宜也是你应得的。

当然如果您是完全凭自己独立完成的 B 题，也不要委屈。因为你刚刚独立的挖掘出了属于自己的 0day！这种喜悦和自豪感才是这个比赛给您的最好的奖品。

虽然我的教程里没有讲到 HeapSpray，但是 B 题里边普遍采用了这种方法，看来大家对浏览器漏洞利用并不陌生。

HeapSpray 在《0day 安全》中有比较详细的介绍。2007 的 black hat 阿姆斯特丹上，一个叫亚历山大的逆向工程师提出了一种叫“heap 风水”的技术，在 heapspray 的基础上又做了很多深入研究。由于 feedback 和火，他在几个月后的 LV black hat 上再次被邀请演讲，当时我和两个趋势的研究员慕名去听，窃窃的发现这个老外和好莱坞一个明星暴像，而且竟然赫然的用中文汉字标注着“风水”两个字，不过是繁体：）。听完之后觉得风水这个词来形容这种技术真的很贴切。

稍后我会把那次演讲的 white paper 和 Slides 发上来，看哪位朋友给大家做点贡献翻一下，大家一起学习，一起进步。

也有不少朋友使用了传统栈溢出的方法。相信大家发现 shellcode 要滤掉一些字节才能完整的塞进去，需要对 shellcode 变形。想到什么了，给 shellcode 加壳与脱壳？呵呵，我说的吧，crack 和 hack 的技术本是一家啊。

详细的评分见附件，如有异议请在一周内发信给我们。另外注明 exploit 验证部分存在一定随机因素，恕不复查。

谢谢大家的积极参与，相信所有提交的朋友都获得了一些东西，不管是鼠标还是挖出 0day 的自豪，这是我们组织这个比赛的初衷。

最后顺便提一下，我的书名做了一些调整，新名字叫《0day 安全：软件漏洞分析技术》，电子工业出版社，出版日期看来得稍微推后几天了，要等到 2 月底 3 月初的样子才能出来，谢谢那些写信给我的朋友对这本书的关注，谢谢。

获奖名单

一等奖（1 名）

**nop**

二等奖（2 名）

**somuch  
toplcj**

三等奖（5 名）

**hahar  
pcasa  
xfmaple  
netwind  
xPLK**

上传的附件

| ![文件类型: rar](img/f4d8045b4cd64e6456be988ac995de11.jpg "文件类型: rar") | VBZH - PHPForce - Joey 修改附件隐藏 | 首届 exploit_me 挑战赛状元榜.rar | /VBZH - PHPForce - Joey 修改附件隐藏 |  |
| --- | --- | --- | --- | --- |

4dd466d0e1e0cc123afbe504da2462ca
