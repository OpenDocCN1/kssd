
# 自己写的 TFTP 服务端，使用完成端口模型,呵呵,大牛飘过~~

> 原文：[`www.pediy.com/kssd/pediy11/124920.html`](https://www.pediy.com/kssd/pediy11/124920.html)

最近学习了完成端口编程模型，即 `IOCP`,看了书上写的 `TCP` 服务端，就想写个 `TFTP` 服务端练练手,一方面因为 `TFTP` 是比较简单的协议，另一方面对完成端口的熟悉.

完成端口:

完成端口基本上公认为一种在 `windows` 服务平台上比较成熟和高效的 `IO` 方法,用完成端口进行重叠 `I/O` 的技术在 `WindowsNT` 和 `WIndows2000` 上提供了真正的可扩展性。完成端口和 `Windows Socket2.0` 结合可以开发出支持大量连接的网络服务程序。使用起来还是很简单的，调用 `CreateIoCompletionPort` 既可以创建完成端口也可以将套接字与完成端口对象相关联，然后所有在完成端口上关联的套接字上读和写的操作的完成都反应到了完成端口上

`TFTP`:

`TFTP` 类似 `FTP`，不过它比 `FTP` 简单的多了，也少了安全的认证，适用于小型文件的传输,关于它的详细信息可以百度搜索下对应的 `RFC` 文件说明,附件中也有.

| Opcode | Filename | 0 | Mode | 0 |
| :--- | :--- | :--- | :--- | :--- |

这是个简单的 `TFTP` 报文结构，`Mode` 字段是模式，我的服务端只支持 2 进制模式，这也是普遍的格式。程序的基本思想是创建 1 个监听线程和 `N` 个工作线程，监听线程负责启动工作线程，和处理超时重发机制，最后结束工作线程。工作线程负责从完成端口取出数据 ，并根据读和写操作的完成来具体做些处理。对于每个客户端则创建一个套接字并申请一个 `CIOCPContext` 来描述它，对于发送和接收每个数据包 用一个 `CIOCPBuffer` 来描述 ,具体定义见代码

几个感觉有点难度的方面：

1. 在判断一个数据包是属于哪个 `CIOCPContext` 的时候，起初是在 `CIOCPBuffer` 中加入了对应的 `Context` 指针，但后来发现这样的逻辑是有问题的,比如你在客户端的 `socket1,`和 `socket2` 上都投递了读请求，这时候向 `socket2` 发送的数据来了，但这时候完成端口它是分不清的，可能的结果是它满足了 `socket1` 的读请求，这就导致了 `context` 指针不正确，所以我是这样解决

代码:

```
//read 操作完成后的 Context 是不正确的需要修正
memcpy(&addrFrom ,(pBuffer->buff+516) ,sizeof(SOCKADDR));
```

从完成的读请求 `BUFF` 的 516 偏移处读取源地址 。在投递读请求的时候是这样的代码:

```
::WSARecvFrom(m_sListen, &buf, 1, &dwBytes, &dwFlags ,(SOCKADDR*)(pBuffer->buff+516) ,&fromlen ,&pBuffer->ol, NULL)
```

意思是这个请求被完成后会在 `pBuffer->buff` 的偏移 516 字节处得到发送地址，为什么要 516 字节呢 ？ 因为一个 `TFTP` 头最多 512 字节 我空了 4 个字节方便调试![](img/9b2aec7f3d4bb041b2d419a59832192f.jpg "恐怖!")  得到源地址后就好办了，代码:

```
pContext = FindCorrectContext(&addrFrom);
```

通过 `FindCorrectContext` 这个函数在 `m_connectList` 中找到对应的 `Context,`这个函数很简单就不引用代码了

2. 超时处理方面 ,只对于写操作完成的情况,每次一个写操作完成后，都会将 `Buffer` 记录在 `CIOCPContext` 结构中的 `pOutTimeBuffer` 中，然后通知监听线程进行超时处理，也就是说每次监听线程只能处理一个 `Buffer` 的超时工作 ,监听线程会将这个 `Buffer` 重新发送一遍

应该就这 2 点了 其余的都是些小细节了。。另外我把发送错误信息重新写了个函数 `PostError,`因为根据协议发送过错误信息是不需要得到回应的<br>本工程在 `VC2008` 下编译成功 第一次发帖 希望不要被喷的太惨。。另外大四了 卖身找工作啊 白菜价出租 最好南京这边的。。。![](img/cb6288ad81b189b02921e98a92da6abe.jpg "124.gif")<br>

`tftp` 协议.rar

`TftpServer.rar`
