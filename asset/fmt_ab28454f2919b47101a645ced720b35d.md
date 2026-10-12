
# LSP 修复工具(山寨 360 的 LSP 修复工具)

> 原文：[`www.pediy.com/kssd/pediy12/136102.html`](https://www.pediy.com/kssd/pediy12/136102.html)

【文章标题】: LSP 修复工具(山寨 360 的 LSP 修复工具)

【文章作者】: Root

【作者邮箱】: `cppcoffee@gmail.com`

【下载地址】: 附件

【编写语言】: VC++、WTL

【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

## 详细过程

开始先了解下概念:

LSP 即分层服务提供商,`Winsock` 作为应用程序的 `Windows` 的网络套接字工具，可以由称为“分层服务提供商”的机制进行扩展。`Winsock LSP` 可用于非常广泛的实用用途，包括 `Internet` 家长控制 (`parental control`) 和 `Web` 内容筛选。在以前版本的 `Windows XP` 中，删除不正确的（也称为 `buggy`）`LSP` 可能会导致注册表中的 `Winsock` 目录损坏，潜在地导致所有网络连接的丢失。

“浏览器劫持”或者“分层服务提供程序”。某些间谍软件会修改 `Winsock 2` 的设置，进行 `LSP`“浏览器劫持”，所有与网络交换的信息都要通过这些间谍软件，从而使得它们可以监控使用者的信息。著名的如 `New net` 插件或 `WebHancer` 组件，它们是安装一些软件时带来的你不想要的东西。

最近公司的 `Client` 团队在开发自己的 `LSP`,于是开始好奇它,再者见到 `360` 有个 `LSP` 修复的功能,很想知道它的工作原理和实现,所以就有了这篇文章(纯属菜鸟作品，大牛请飘过).![](img/906f120cfe711ccbc8463fd6eb86eee9.jpg "尴尬")

进入正题吧![](img/081868d20f20fe7a9b4b88ab0d0d33c6.jpg "狂笑")

先了解下 `LSP` 在注册表中的位置 `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WinSock2\Parameters` 下的 `Current_NameSpace_Catalog` 描述了当先使用的协议目录，一般为 `Protocol_Catalog9`。

`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WinSock2\Parameters\Protocol_Catalog9` 下的 `Num_Catalog_Entries` 记录了该结构链的节点的总数。

`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WinSock2\Parameters\Protocol_Catalog9\Catalog_Entries` 下面的以序号为键名的内容依次对应了一个服务程序，可以自启动，因此 `SPI` 成了流氓软件和病毒的目标。

程序首先读取 `Current_Protocol_Catalog` 协议目录存放的子键名称,接着读取存放目录子键下的 `Num_Catalog_Entries` 目录总数,接着遍历 `Catalog_Entries` 下的所有存放协议目录的子键,他们的存放结构如下：

```
typedef struct tagPACKED_CATALOG_ITEM
{
    char spi_path[MAX_PATH];    // dll 的路径名称
    WSAPROTOCOL_INFO protocol_info;    // 协议的结构体
} PACKED_CATALOG_ITEM, *PPACKED_CATALOG_ITEM;
```

将注册表存放目录的数据填充到这个结构体里面即可.

**如何判断是否有效的 LSP 呢。**

1、对协议结构体中的 `CLSID` 进行匹配，与以下的 `CLSID`：

   {E70F1AA0-AB8B-11CF-8CA3-00805F48A192}
   {9D60A9E0-337A-11D0-BD88-0000C082E69A}
   {8D5F1830-C273-11CF-95C8-00805F48A192}
   {3972523D-2AF1-11D1-B655-00805F3642CC}

2、根据 `DLL` 存放的路径进行匹配，与以下的文件名进行匹配（必须是 `Windows\System32` 目录下）

   `mswsock.dll`
   `rsvpsp.dll`
   `msafd.dll`
   `ws2_64.dll`

以上操作完成了 `LSP` 的辨别操作。

**接下来是修复的工作：**

1、保存原先的注册表信息，路径：`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WinSock2\Parameters` 保存为 `WinSockLSP.reg`。

2、保存 `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WS2IFSL` 到当前路径下的 `WinSockLSPIFSL.reg`。

3、将在第一步确定为未知的 `LSP` 的 `DLL` 保存到当前路径下的 `LSP` 中，并写入相关的信息到 `LSP.ini` 中，以便后续的恢复操作可执行。

4、对检测为未知的 `DLL` 再次进行检测，是否与以下的 `DLL` 名称相符合。

   `shdocvw.dll`
   `Shell32.dll`
   `Oleaut32.dll`
   `Actxprxy.dll`
   `Mshtml.dll`
   `Urlmon.dll`
   `Msjava.dll`
   `Browseui.dll`

5、如果不符合已知的 `DLL`，那么将这个未知的 `LSP` 的 `DLL` 进行删除处理，并删除这个相关的 `LSP` 协议。

**最后是还原操作：**

1、导入保存的 `WinSockLSP.reg` 和 `WinSockLSPIFSL.reg`

2、读取保存的 `LSP` 文件夹下的 `LSP.ini`，得到 `DLL` 删除前存放的路径。

3、将所有的 `DLL` 拷贝到原先存放的路径中。

在完成这个小工具的期间在反汇编 `360LSP` 修复工具的时候发现一个逻辑上的小 `BUG`：

`360LSP` 修复在写入 `LSP` 相关的 `DLL` 文件到 `ini` 中的时候，只能恢复保存在 `ini` 中的第一个 `DLL` 文件。是由于 `GetPrivateProfileSection` 只能读取到第一个换行的数据.而第二行的数据无法得到.(文件版本号: `7.0.2.1005`)![](img/afde35a5f3b3a4675abb3221cf07c6d1.jpg "124.gif")

贴张界面效果图![](img/906f120cfe711ccbc8463fd6eb86eee9.jpg "尴尬")

![](img/0cf7dfa46615d6550bc92f15eea0b0d5.jpg)

---

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2011 年 06 月 26 日 21:14:05

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || `Fix_Lsp by Root.rar` | |
|---|---|---|
