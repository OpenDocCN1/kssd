
# 网络游戏解密

> 原文：[`www.pediy.com/kssd/pediy05/pediy50474.htm`](https://www.pediy.com/kssd/pediy05/pediy50474.htm)

`WPE` 我用过，前段时间还给网金也疯狂做了一个破解：－）

但说实话，它的使用和 `CRACK` 真是有区别

## 1. `WPE` 的作用：举个例子

我在网络游戏中，捡到 10 个钱币，又捡到 10000 个钱币，

然后，对比两次传输数据的不同。

再用 `WPE` 对网络游戏发出的封包进行拦截，找到 10 个钱币的数据，自动改为 10000 个钱币的数据，发送给网络服务器，来欺骗它。

## 2. `CRACK` 就完全不是这样了

因为很多加密代码段，是在服务器上的，所以，有些网络游戏没办法破解

我也知道不是很多，高手莫笑！

启动 `wpe`，双击 `select game`，选择你所要截获 `ip` 包的程序

先找到 `MIR.DAT`

`trace` 选项是用来截获封包的，具体方法就是双击 `trace`，按开始录制的三角，停止按红方。

`sent` `recv` 显示的接收和发送封包的进度！在设定的长度到达后，自动停止，默认值 100 行！

比如以前有人说的……找到……你要复制的封包复制金条的过程，希望大家不要去试，当然你想被删除自己的 `ID`（双击 `filter list` 里任意一个 `filter`，进入编辑模式！`data` 对话框里，`search` 是写入可能截获的封包的内容，`modify` 里写入的是，每当出现 `search` 里的内容的时候，用 `modify` 里写入的替换掉！也就是说假如我截获一个“金条”的封包，我写入 `search`，再截获一个“金条”的封包，那么每当过关卡的时候都将出现一块金条！）

下面这些是发送的方法……截获封包后，`s` 框后面的数字是发送封包的内容，`r` 框后面的数字是接收封包的内容，以 16 进制数和 `acsII` 两种方式体现！整个程序的上面有储存、清除，查找 3 个选项。

`filter` 的功能实现起来比较复杂，原理是每当有一个发送的封包中含有指定的字节，那么发送另一个由你编写伪封包出去！打开 `fliter` 以后选项有：打开、保存、全部选择写封包、开始 `fliter`、清除已编写的封包、清除所有选择的封包、关闭！

`send packets` 是发送封包的功能，`packets list` 里有 5 个选项，不再重复介绍。`send` 对话框 `time` 可以设置发送封包的次数，`new socket` 是指定 `ip` 和端口发送，`open socket` 是利用 `select game` 指定的程序发送，端口可以双击截获的封包前 `s` 框获得！

双击任意 `packets` 可以进入编辑界面，用 `size` 来设置封包最大长度！编辑好以后，按箭头加入右面发送区（最大 100 个），设置好发送次数，点击开始的三角，即可发送！

`options` 里面是一些关于 `wpe` 的设置！

`winsock fuctions` 用来选择截获封包的的种类，比如，只截获发送封包，就在 `send,send to` 前面选择对号！

`fliter options` 里面是设置 `fliter` 功能里最大封包的长度！

`packets trace options` 里设置的是截获封包的最大长度，和发送封包。

看那个也没用，可能只有早期的石器时代可以简单的用 `wpe` 改了。

潜入到游戏进程里面，获得 `socket`，从 `sendto` 这类函数入手，`hook` 或者别的什么方法。

我给你提供一些资料关于 `winsock` 函数简介

## 1. `WSAStartup` 函数

用于初始化 `Winsock`

**[声明]**

```c
int WSAStarup(WORD wVersionRequested, LPWSADATA lpWSAData);
```

**[参数]**

`wVersionRequested` - 要求使用 `Winsock` 的最低版本号

`lpWSAData` - `Winsock` 的详细资料

**[返回值]**

当函数成功调用时返回 0

失败时返回非 0 的值

## 2. `socket` 函数

用于生成 `socket` (`Socket Descriptor`)

**[声明]**

```c
SOCKET socket(int af, int type, int protocol);
```

**[参数]**

`af` - 地址家族 (通常使用: `AF_INET`)

`type` - `socket` 的种类

`SOCK_STREAM` : 用于 `TCP` 协议

`SOCK_DGRAM` : 用于 `UDP` 协议

`protocol` - 所使用的协议

**[返回值]**

当函数成功调用时返回一个新的 `SOCKET` (`Socket Descriptor`)

失败时返回 `INVALID_SOCKET`.

## 3. `inet_addr` 函数

把好象 `"xxx.xxx.xxx.xxx"` 的 10 进制的 `IP` 地址转换为 32 位整数表示方法

**[声明]**

```c
unsigned long inet_addr(const char FAR *cp);
```

**[参数]**

`cp` - 指向用 `"xxx.xxx.xxx.xxx"` 的 10 进制来表示的 `IP` 地址字符串的指针

**[返回值]**

当函数成功调用时返回用 32 位整数表示的 `IP` 地址 (按网络字节排列顺序)

失败时返回 `INADDR_NONE`.

## 4. `gethostbyname` 函数

可以从主机名获取主机资料.

**[声明]**

```c
struct hostent FAR *gethostbyname(const char FAR *name);
```

**[参数]**

`name` - 指向主机名字符串的指针

**[返回值]**

当函数成功调用时返回主机信息

失败时返回 `NULL` (空值)

## 5. `Bind` 函数

指定本地 `IP` 地址所使用的端口号时候使用

**[声明]**

```c
int bind(SOCKET s, const struct sockaddr FAR *addr, int namelen);
```

**[参数]**

`s` - 指向用 `Socket` 函数生成的 `Socket Descriptor`

`addr` - 指向 `Socket` 地址的指针

`namelen` - 该地址的长度.

**[返回值]**

当函数成功调用时返回 0

调用失败时返回 `SOCKET_ERROR`

## 6. `connect` 函数

用于与服务器建立连接,发出连接请求,必须在参数中指定服务器的 `IP` 地址和端口号

**[声明]**

```c
int connect(SOCKET s, const struct sockaddr FAR *name, int namelen);
```

**[参数]**

`s` - 指向用 `Socket` 函数生成的 `Socket Descriptor`

`name` - 指向服务器地址的指针

`namelen` - 该地址的长度.

**[返回值]**

当函数成功调用时返回 0

调用失败时返回 `SOCKET_ERROR`

## 7. `select` 函数

可以用于调查一个或多个 `SOCKET` 的状态.

**[声明]**

```c
int select(int nfds, fd_set FAR *readfds, fd_set FAR *writefds, fd_set FAR *exceptfds, const struct timeval FAR *timeout);
```

**[参数]**

`nfds` - 在 `WINDOWS SOCKET API` 中该参数可以忽略,通常赋予 `NILL` 值

`readfds` - 由于接受的 `SOCKET` 设备的指针

`writefds` - 用于发送数据的 `SOCKET` 设备的指针

`exceptfds` - 检查错误的状态

`timeout` - 超时设定

**[返回值]**

返回大于 0 的值时,表示与条件相符的 `SOCKET` 数

返回 0 表示超时

失败时返回 `SOCKET_ERROR`

## 8. `recv` 函数

利用 `Socket` 进行接受数据.

**[声明]**

```c
int recv(SOCKET s, char FAR *buf, int len, int flags);
```

**[参数]**

`s` - 指向用 `Socket` 函数生成的 `Socket Descriptor`

`buf` - 接受数据的缓冲区(数组)的指针

`len` - 缓冲区的大小

`flag` - 调用方式 (`MSG_PEEK` 或 `MSG_OOB`)

**[返回值]**

成功时返回收到的字节数.

如果连接被中断则返回 0

失败时返回 `SOCKET_ERROR`

## 9. `sendto` 函数

利用 `Socket` 进行发送数据.

**[声明]**

```c
int sendto(SOCKET s, const char FAR *buf, int len, int flags, const struct sockaddr FAR *to, int token);
```

**[参数]**

`s` - 指向用 `Socket` 函数生成的 `Socket Descriptor`

`buf` - 接受数据的缓冲区(数组)的指针

`len` - 缓冲区的大小

`flag` - 调用方式 (`MSG_DONTROUTE`, `MSG_OOB`)

`to` - 指向发送方 `SOCKET` 地址的指针

`token` - 发送方 `SOCKET` 地址的大小

**[返回值]**

成功时返回已经发送的字节数.

失败时返回 `SOCKET_ERROR`

你可以用 `softICE` 当一下看看

今天反编译了传奇的客户端和服务器端，发现的只是程序编写上的漏洞，但这些是违法的，所以不便讲解。

这里我只讲一些运算上的原理；

### A. 服务器所要做的工作只是效验计算值而已

客户端也要作相关的计算（预算），但此计算结果并无太多用处，`it` 只是给予客户端的玩家一种错觉，你可以看到类似的例子，比如我们经常遇到，人明明走了过去但是又退回来了，走过去的地方插了一个人，这个就是客户端计算后发现可能走过去，发给服务器数据，问她能否走么？服务器考虑了一下，回答，兄弟好像不行诶，然后客户端骂道 `NND`,退回去，就退回来来了。：））

### B. 数据如果放在客户端，是没有任何意义的

谁都能改。网络游戏所有计算都应该放在服务器端。客户端只是用来显示，不做任何逻辑判断。

比如：

1. `client` 发送战斗指令（比如普通统计，魔法攻击) -> `server`
2. `server` 计算每一次伤害的数值 -> `client`
3. `client` 显示。

这只是简单模型。如果要完善服务器需要记录很多变量，比如为了防止快砍功能，服务器需要判断每两次攻击的时间间隔；如果要防止瞬移，服务器要记录响应坐标信息等等。修改在现阶段是没有办法完全防止的。但是基本原则，如果 `softICE` 只是修改客户端显示方式等，无所谓。但是如果可能对游戏关键数据进行修改的话，则此游戏数据应该放在服务器端计算。

1. 客户端发送玩家的信息到服务端并显示相应的动画
2. 服务端收到动作信息后，读取玩家的静态数据并进行计算
3. 服务端将计算中心结果传给客户端
4. 客户端收到服务端的计算结果后，以图形或其他方式表示出来。其实你可以看看现在开放源代码的网络游戏。比如:`UO` (非官方),人在江湖等。

`wpe` 无用

自己分析游戏 自己写钩子 `HOOK` 去监视游戏软件

截取封包

`4dd466d0e1e0cc123afbe504da2462ca`
