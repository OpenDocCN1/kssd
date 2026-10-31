
# [旧帖][原创]对PPS的一次调试0.00雪花

昨天晚上朋友说想上PPS看美剧，结果发现欧美剧场没内容了。PPS之前因为版权问题屏蔽了欧美剧场，但是通过补丁还能看到。这次是即使通过补丁找回欧美剧场，里面也没内容。

![](https://bbs.kanxue.com/upload/attach/201110/462151_ek05y8xl7xqku90.jpg)

应朋友要求，简单对PPS进行了一次调试，水平有限，高手勿喷。

查壳，Microsoft Visual Studio .NET 2005 -- 2008 -> Microsoft Corporation。

用`OD`加载调试，首先想到的是下消息断点。在PPS上试验了一下，应该是在`WM_LBUTTONUP`时进行的处理。

让PPS先运行起来，查看窗口。

![](https://bbs.kanxue.com/upload/attach/201110/462151_pe456ypxc1ovuy1.jpg)

PPS的窗口比较多，用工具找了下频道列表窗口（句柄值为`0002049E`）类别为树形控件。对其下消息断点（`WM_LBUTTONUP`）。

点击频道列表中的某项，程序立刻断下来。

```
776CE0E4 >  6A 10           push    10
```

此时处于系统`user32`的领空，`Alt+F9`返回，可以看到程序来到`PowerList.ocx`的领空。看来PPS是由这个控件来处理频道列表的。

回溯一下栈，没发现什么明显的有用的东西（也可以`F8`初略跟一下）。查找一下参考文本字符串，会发现很多与协议有关的东西，这里暂不说明。因此转而重启并`bp recv`。

PPS还没有完全打开就会断下来，`Alt+F9`返回，依然是在`PowerList`领空，可以看到下面的代码。

```
06244179  |.  FF15 ACD93106 call    dword ptr [<&WININET.HttpSendRequestA>]   ;  wininet.HttpSendRequestA
0624417F  |.  85C0          test    eax, eax
06244181  |.  75 09         jnz     short 0624418C
```

程序使用了`HttpSendRequestA` API，在`MSDN`上查一下其原型。

```
BOOL HttpSendRequest(
  __in  HINTERNET hRequest,
  __in  LPCTSTR lpszHeaders,
  __in  DWORD dwHeadersLength,
  __in  LPVOID lpOptional,
  __in  DWORD dwOptionalLength
);
```

注意其第一个参数，Handle returned by `HttpOpenRequest`，再看一下`HttpOpenRequest`的原型。

```
HINTERNET HttpOpenRequest(
  __in  HINTERNET hConnect,
  __in  LPCTSTR lpszVerb,
  __in  LPCTSTR lpszObjectName,
  __in  LPCTSTR lpszVersion,
  __in  LPCTSTR lpszReferer,
  __in  LPCTSTR *lplpszAcceptTypes,
  __in  DWORD dwFlags,
  __in  DWORD_PTR dwContext
);
```

同样的注意到第一个参数，A handle to an HTTP session returned by `InternetConnect`，再看下`InternetConnect`的原型。

```
HINTERNET InternetConnect(
  __in  HINTERNET hInternet,
  __in  LPCTSTR lpszServerName,
  __in  INTERNET_PORT nServerPort,
  __in  LPCTSTR lpszUsername,
  __in  LPCTSTR lpszPassword,
  __in  DWORD dwService,
  __in  DWORD dwFlags,
  __in  DWORD_PTR dwContext
);
```

`OK`，在这里我们能看到`lpszServerName`。大致流程可以猜想到，用`InternetConnect`指定连接的服务器，然后通过`HttpOpenRequest`制造一个`Http`请求，最后`HttpSendRequest`。当然，有请求就会有返回，通过多次`recv`断下来，还可以发现`InternetReadFile`函数的调用。

对`InternetConnect`、`HttpOpenRequest`和`InternetReadFile`这三个函数下断（此时`recv`的断点可以禁用了），接下来`F9`加`Atl+F9`的过程可以看到很多东西。

![](https://bbs.kanxue.com/upload/attach/201110/462151_za8b1hm0iox8sua.jpg)

`InternetConnect`的服务器有`list1.ppstream.com`（获取频道列表）、`stat.ppstream.com`（统计）等。

![](https://bbs.kanxue.com/upload/attach/201110/462151_vv8k46g7qlhtnei.jpg)

`HttpOpenRequest`请求的文件有`/class/50.xml.zip`（最近更新）、`/class/generas.xml.zip`（频道大栏目）等。

仔细跟踪打开一个大栏目的过程还可以发现其获取子栏目列表或者具体视频信息的过程，如`/schs/13034.xml.zip`为高清影院动作片的视频信息。

![](https://bbs.kanxue.com/upload/attach/201110/462151_dgekrm5bjr88unv.jpg)

通过打了补丁的PPS发现，欧美剧场的ID为`192`，因此到`list1.ppstream.com/class/192.xml.zip`下载下来`zip`文件打开发现，其中的`xml`中已经没有内容。因此可以判定，之前PPS只是在`generas.xml.zip`中屏蔽了欧美剧场，这次在其基础上更在欧美剧场的内容列表`192.xml.zip`上进行了屏蔽。

花了近一个小时写这篇文字，希望能求个邀请码。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [1.jpg](javascript:void(0);)
  （4.58kb，219次下载）
- [2.jpg](javascript:void(0);)
  （79.62kb，220次下载）
- [3.jpg](javascript:void(0);)
  （16.39kb，219次下载）
- [4.jpg](javascript:void(0);)
  （20.16kb，220次下载）
- [5.jpg](javascript:void(0);)
  （64.66kb，222次下载）
