
<mark class="encrypted">965K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2$3K9i4u0#2M7%4c8G2N6r3q4D9i4K6u0W2j5$3!0E0i4K6u0r3k6%4g2A6i4K6u0r3k6r3!0E0j5h3W2F1i4K6u0r3j5g2)9J5k6i4j5$3j5X3!0^5i4K6u0W2j5$3&6Q4x3V1k6J5k6h3I4S2N6r3W2G2L8Y4x3`.</mark>

<mark class="encrypted">5baK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2$3K9i4u0#2M7%4c8G2N6r3q4D9i4K6u0W2j5$3!0E0i4K6u0r3k6%4g2A6i4K6u0r3k6r3!0E0j5h3W2F1i4K6u0r3j5#2)9J5k6i4j5$3j5X3!0^5i4K6u0W2j5$3&6Q4x3V1k6J5k6h3I4S2N6r3W2G2L8Y4x3`.</mark>

# Binary114_dll64

## 基本信息

![](upload/attach/202306/917207_A8H8FKPANXTKZR7.png)

## 分析

动态加载 DLL 来调用指定函数后，连接名为 `\RPC Control\E0BD3C9B-15CC-43D5-868C-B75AACFB7F50` 的RPC端口后，创建了两个用户态线程，并关闭了第二个线程返回的句柄。

![](upload/attach/202306/917207_YXDAK9D5EQ27H25.png)

第一个用户态线程，使用RPC端口进行进程间通信，当接受到消息时会执行`sub_180001B50`函数会创建一个用户态子线程来处理接收到的消息；第二个用户态线程会等待一个内核对象并在其被通知后关闭该对象，返回ALPC端口句柄。子线程会创建指定进程的环境块，如环境变量、工作目录等，再通过`NtOpenProcess`、`NtAllocateVirtualMemory`、`NtCreateThreadEx`实现远程注入。

![](upload/attach/202306/917207_VD97KHNY9BJZ32W.png)

最后遍历当前进程的模块列表，来卸载注入的dll，注册一个事件，标识着该 dll已被卸载

# Binary103_dll32

## 基本信息

![](upload/attach/202306/917207_6WJAFNVPM2M7RR8.png)

## 分析

导出函数`Install`，初始化一个安全描述符，并将该安全描述符与创建的名为`{B8592103-AE8C-4D37-807F-F1CB76E62B7C}`的互斥对象相关联。

![](upload/attach/202306/917207_5EG2YA8G4WEP74C.png)

如果互斥对象创建成功，则会检测当前系统上是否运行 `Steam.exe`，如果在运行则远线程注入指定dll。

![](upload/attach/202306/917207_ESDTZZPBZYQNVJ9.png)

导出函数`SetTargetUrl`用于设置链接的目标URL，URL指向导出函数`Address`，在`Address`函数中存在字符串 `\csgo\cfg\gamestate_integration_discordpresence.cfg`、`-LobbyUrl`、`(https.*html)`、`Tslgame.exe`、`ExecPubg.exe`。

`\csgo\cfg\gamestate_integration_discordpresence.cfg` 是一个CS:GO的游戏配置文件，用于与 Discord 集成并显示玩家状态。

`-LobbyUrl` 是一种命令行参数，通常用于指定多人游戏中的游戏大厅链接地址（Lobby URL）

`(https.*html)` 是一个正则表达式，可以匹配包含 "https" 和 ".html" 的任意字符串

导出函数`UnInstall`，用于卸载钩子和释放互斥对象；`SetTargetUrlCString`函数设置CString对象；`SetRedirectUrlStatus`函数重定向URL状态。

`DllMain`函数，dll加载且互斥对象创建成功时，遍历进程来检测`Steam.exe`进程中当前模块是否存在，然后通过`fdwReason` 的值来判断操作系统是在加载还是卸载该DLL，如果是加载事件（`fdwReason = 1`），禁用`DLL_THREAD_ATTACH`和`DLL_THREAD_DETACH`通知，执行`sub_100067F0`函数；如果是卸载事件（`fdwReason = 0`），则执行`sub_10006A80`函数。

![](upload/attach/202306/917207_6V5XWJXU2644TE3.png)

`sub_100067F0`函数：创建一个RPC字符串绑定，关联的GUID为`[Guid("71CE029F-1D04-42EE-8DC8-5012FCC0EED3")]`并Hook `CreateProcessInternalW`函数，然后根据程序计数器判断是否释放互斥对象和终止当前线程，最后创建一个新的定时器对象。HOOK `CreateProcessInternalW`函数来实现对`steam.exe`进程创建的完全控制，在进程创建过程中经 枚举进程快照获得指定进程PID-->`OpenThread`-->`SuspendThread`-->`GetThreadContext`-->获取EIP-->修改EIP-->`SetThreadContext`-->`ResumeThread` 的方式实现线程挂起注入。

![](upload/attach/202306/917207_X3RFXWJSJWZEWJY.png)

`sub_10006A80`函数会判断HOOK或线程挂起注入是否成功，成功则返回0，接着释放内存，销毁堆对象、释放定时器对象等。

# Binary109_sys64

## 基本信息

![](upload/attach/202306/917207_CPAQMYHNZJPQ8ZT.png)

驱动含有一个证书签名

![图片描述](upload/attach/202306/917207_TH7DN8FSYUZNYBX.png)

PDB信息

![](upload/attach/202306/917207_NR2N2P8G76E6F2S.png)

## 分析

加载并注册该驱动，注册驱动前检查当前系统是否开启了内核调试器，接着检测操作系统是否符合要求。符合要求时，注册一个监控进程创建的回调函数，该回调函数还会过调用`PsGetProcessWow64Process`函数获取进程的Wow64进程对象；注册了一个进程镜像加载的回调函数，并创建了一个名为 `\ProcCommPort_{1FF62CDF-9626-41DF-8673-2C2DE973C0F8}` 的通信端口，以便其他进程可以与该端口进行通信，通过`FltStartFiltering`函数启动过滤器，开始对进程的镜像加载进行过滤。

![](upload/attach/202306/917207_SZBX8QU4Q5FW8A7.png)

# Binary107_dll64

## 基本信息

![](upload/attach/202306/917207_RGB7UYEKW2MJ2QQ.png)

## 分析

主要为三个导出函数，`DllRegisterServer`、`DllUnregisterServer`、`NSPStartup`，其中`DllRegisterServer`函数功能为注册`NSPStartup`函数，`DllUnregisterServer` 为卸载`NSPStartup`函数

`NSPStartup`函数中，首先打开名为 `Global\B5437222-9926-4F89-9DA1-F33214770865` 的内存映射文件，之后会初始化和注册NSP、`TCPProxy` 类对象的初始化、创建与网络接口有关的I/O完成端口等等。

![](upload/attach/202306/917207_9D5G8PK83GMDM98.png)

# Binary110_dll32

## 基本信息

![](upload/attach/202306/917207_BM5HV9R8H74TKCW.png)

## 分析

只有获取到环境变量`{FC3BA882-AAED-4E7F-8C6B-1224A40646D3}` 的值为 `{E7539593-2191-4A9D-ABC4-3940BD6E6279}` 时，才会创建一个线程 。

![](upload/attach/202306/917207_F86F682SQ2SRMKM.png)

该线程首先会创建一个`snqu::CsGoPlug`的对象，将其添加到列表中，然后解密以下URL：`http:[//]csapi[.]skinsabc[.]com`、`http:[//]csapi[.]skiin[.]com`、`http:[//]csreport[.]skinsabc[.]com`，将URL传递给`sub_1000CC30`函数并执行，最后释放`snqu::CsGoPlug`对象；在`sub_1000CC30`函数中，创建一个名为 `[Guid("499D40B1-D0D6-4336-9AAF-DD7A5B535923")]` 的全局互斥对象，之后会进行消息循环处理、相关的异常处理等。

![](upload/attach/202306/917207_4P5MH2YCDBR3X3U.png)

`CsGoPlug`类中包含三个函数`sub_1000FFE0`、`sub_10010000`、`sub_10010020`。

在`sub_1000FFE0`函数中，`sub_1000FD6`函数的子函数`sub_101B5D70`会获取桌面窗口的客户区矩形大小，并创建一个新线程来执行`sub_101B5BA0`函数用于检查窗口是否存在且有效，并调用 `MoveWindow` 函数移动窗口位置与另一个窗口的位置重叠，达到HOOK窗口的目的。

![](upload/attach/202306/917207_4YC8YN7P6CPSDVG.png)

子函数`sub_1000F670`会检查`CefCore.dll`是否存在并设置环境变量`{73E58D05-D2A4-4A6E-BFB5-762A53DA1EE0}`的值为`CefCore.dll`的路径，接着从自身RC资源段导出dll，即`Binary110_104_dll64`，然后禁用当前线程中的文件系统重定向，注册导出的dll文件。

子函数`sub_100485F0`和`sub_10048640`相对比较复杂，根据调用的API还有调试信息，如：API：`CreateWindowExA`、`ShowWindow`、`UpdateWindow`、`GetWindowRec`、 `PostMessageA`、`PtInRect`等，字符串：`cs_main_page`、`csgo.png`、`csgo_center.png`、`csgo_banner.png`、`steam_banner.png`、`plug check csgo_show_paster:%s`、`null steam banner`、`csgo_resource_923`等，可以大致推断出功能为HOOK指定窗口，获取指定窗口信息，对指定窗口进行监控以及窗口相关消息处理等等。

![](upload/attach/202306/917207_BKSX4TYZZAH8VJM.png)

`sub_10010000`函数主要用于销毁窗口及其子窗口，释放与窗口相关的资源，释放 OLE 库所占用的资源等。`sub_10010020`调用`PostMessageA`把消息放入消息队列然后返回。

![](upload/attach/202306/917207_NJXTYHVZG358FVN.png)

在 `sub_10045BC` 函数中，存在json字段`nickname`、`steamId`、`channelId`、`mac`、`ip`，而在`sub_100087E0`函数中存在字符串`channel_id={0}&mac={1}&ip={2}&steam_id={3}&type={4}&launcher={5}&version=20220923` ；该dll中包含一个`Crypto++`库，可能是用于在加密通信数据。

# Binary111_dll32

## 基本信息

![](upload/attach/202306/917207_9C23Z3VYPNWSACQ.png)

## 分析

连接名为 `\RPC Control\60C926AD-394B-4FD5-AF74-4BA6163DF10F` 的RPC端口，通过该端口与远程主机建立连接，然后卸载指定模块，注册 ETW 事件并记录相关信息，最后调用`sub_10001800`函数。

![](upload/attach/202306/917207_76ME7XAUHDSGX4U.png)

`sub_10001800`函数主要用于卸载动态HOOK：Inline Hook和远程HOOK，通过检查 P 列表的内存保护属性来判断是否存在 hook，如果存在则通过恢复被修改的代码区域，以及还原被修改的代码区域的访问权限来卸载HOOK。

![](upload/attach/202306/917207_BA94ZTM3KRZ5MAY.png)

# Binary110_103_dll32

位于`Binary110_103`资源段

## 基本信息

![](upload/attach/202306/917207_ZD9CXQ3P6MPMGGF.png)

## 分析

导出函数`CreateInstance`用于创建COM组件的实例对象，其中有`CMbPlugEx`类、`CMbPlug`类，而在`CMbPlugEx`中的`sub_10004280`有一个`EventSink`类。在`CmbPlug`类中的`sub_10004250`中有一个函数`sub_1000E740`，功能很有可能是CEF注入。下面对`sub_1000E740`函数进行分析。

`sub_100012B0`函数会检索当前进程所在操作系统是否为64位，通过改变DLL目录的方式来延迟加载dll，最后动态加载`libcef.dll`中的相关API。

![](upload/attach/202306/917207_WD85EWUV4XG26AA.png)

`sub_10002730`函数用于检测`libcef.dll`文件版本是否为85.0.0。实际上，在`..\steam\bin\cef\cef.win7x64`目录，存在一个`libcef.dll`文件，版本号为85.0.0

![](upload/attach/202306/917207_TSNXJ9E3XH6JQ5Q.png)

`sub_1000ECE0`用于获取`steamwebhelper.exe`进程的可执行文件路径和所有模块句柄

![](upload/attach/202306/917207_W4E7X3D7AMJHMAA.png)

之后根据一些字符信息，如：`initialize_cef_app`、`initialize_client_handler`、`initialize_cef_load_handler`、`cef_shutdown`等等涉及CEF初始化设置、启动、页面加载、关闭等，并基于此`https://bbs.kanxue.com/thread-262546.htm`，推测`BINARY110_103_dll32` 功能很有可能为向steam页面插入广告或者是伪造页面。

# Binary110_104_dll64

位于`Binary110_103`资源段

## 基本信息

![](upload/attach/202306/917207_M7FGADS7APGSWF6.png)

## 分析

获取当前进程的文件名，并判断该进程是否为 `regsvr32.exe`，如果不是，则创建一个子线程。

![](upload/attach/202306/917207_HAFM4V9ZQP644BH.png)

子线程会创建一个窗口，并通过消息循环来处理相关事件，

![](upload/attach/202306/917207_KW2CGUGQ79UPCFS.png)

而`sub_180017EC0`函数功能与`Binary110_103_dll32`中的`sub_1000E740`、`sub_10002730`、`sub_1000ECE0`等函数功能差不多。

包含几个导出函数`DllInstall`、`DllRegisterServer`、`DllUnregisterServer`、`DllCanUnloadNow`、`DllGetClassObject`。

`DllInstall`函数用于安装DLL，`DllRegisterServer` 注册 dll，并返回注册结果。如果注册失败，则会调用 `DllUnregisterServer` 卸载 dll。

![](upload/attach/202306/917207_SM8S7U58SZHFHUU.png)

`DllRegisterServer`函数用于初始化COM对象，将 dll 注册为 COM 对象，并将该 COM 对象关联到的 AppID 和 CLSID。AppID名为`{58942be0-6d19-4812-ac05-fb480c913727}`。

![](upload/attach/202306/917207_BQ9AZ7DDEHKRRTX.png)

`DllUnregisterServer`用于反注册dll；`DllCanUnloadNow` 函数用于检查当前组件是否可以被卸载；`DllGetClassObject` 函数用于获取 COM 组件的类工厂对象，以便后续调用类厂对象的 `CreateInstance` 方法来创建组件的对象实例。

# 某GameThief木马分析

## 样本总体信息
