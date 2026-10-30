
# AFLNET 源码分析：网络通信部分

（这里的逻辑我认为其实还是有一些缺陷的，如果有些服务端有一些逻辑需要一定的时间才能到达的话其实会被框架漏掉，但是这种情况比较少并且如果改成连续多少次返回值不是2的话会牺牲掉一定的效率，并不值得。）

关闭`sockfd`，`kill`子进程并为了使其能够优雅的退出进行一定的等待，

```c
HANDLE_RESPONSES:
  net_recv(sockfd, timeout, poll_wait_msecs, &response_buf, &response_buf_size);
  if (messages_sent > 0 && response_bytes != NULL) {
    response_bytes[messages_sent - 1] = response_buf_size;
  }
  //wait a bit letting the server to complete its remaining task(s)
  memset(session_virgin_bits, 255, MAP_SIZE);
  while(1) {
    if (has_new_bits(session_virgin_bits) != 2) break;
  }
  close(sockfd);
  if (likely_buggy && false_negative_reduction) return 0;
  if (terminate_child && (child_pid > 0)) kill(child_pid, SIGTERM);
  //give the server a bit more time to gracefully terminate
  while(1) {
    int status = kill(child_pid, 0);
    if ((status != 0) && (errno == ESRCH)) break;
  }
  return 0;
}
```

AFLNET的网络通信部分基本就是这些，后面会继续阅读有关状态图的逻辑，搞清楚是如何做到在状态图中进行转换并以此指导模糊测试数据变异的。

**AFLNET**源自一篇论文：

[论文下载](<https://bbs.kanxue.com/elink@f85K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9K9h3&6C8i4K6u0W2P5X3S2A6K9s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6Q4x3@1k6@1j5i4u0Y4k6i4c8Q4x3@1c8Z5N6s2c8H3M7#2)9J5y4e0y4m8i4K6u0r3i4K6u0r3L8h3u0G2k6h3S2E0k6g2)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3M7r3q4H3k6i4u0Q4x3V1k6u0b7#2y4f1x3U0m8Q4x3X3g2m8c8V1I4z5k6i4c8Q4x3X3g2H3k6r3j5>).

它是一种针对**网络协议**的灰盒模糊测试器，通过状态反馈来指导模糊测试，在模糊测试的过程中根据协议的反馈不断完善有限状态自动机并据此指导模糊测试框架的数据变异和状态选择。

以往针对协议的黑盒模糊测试框架`boofuzz`由于缺少代码插桩和覆盖率引导，所以测试数据往往以字典为主，无法主动变异出字典以外的数据。AFLNET以经典框架AFL为基础，改进了测试端和被测端的通信方式，使其能够适配网络协议，接下来这个系列将对AFLNET的源码进行分析。

## AFLNET 与 AFL 的通信差异

首先分析的是AFLNET如何实现的测试框架和被测服务端的通信，我们知道在AFL中，框架首先用`execv()`来初始化forkserver，后续的程序启动则通过管道向forkserver发送消息不断`fork()`子进程来进行，输入多为执行参数中直接指定某个文件作为输入，但是对于网络协议来说输入需要以数据包的形式发送到server端，来看看AFLNET是如何实现。

首先对于forkserver的初始化阶段AFL与AFLNET没有任何区别，区别主要体现在forkserver初始化以后的`fork()`操作中，首先关注直接涉及到运行程序的函数`run_target()`。

### run_target 对比

**AFL：**

```c
if (dumb_mode == 1 || no_forkserver) {
  if (waitpid(child_pid, &status, 0) <= 0) PFATAL("waitpid() failed");
} else {
  s32 res;
  if ((res = read(fsrv_st_fd, &status, 4)) != 4) {
    if (stop_soon) return 0;
    RPFATAL(res, "Unable to communicate with fork server (OOM?)");
  }
}
```

**AFLNET：**

```c
if (dumb_mode == 1 || no_forkserver) {
  if (use_net) send_over_network();
  if (waitpid(child_pid, &status, 0) <= 0) PFATAL("waitpid() failed");
} else {
  if (use_net) send_over_network();
  s32 res;
  if ((res = read(fsrv_st_fd, &status, 4)) != 4) {
    if (stop_soon) return 0;
    RPFATAL(res, "Unable to communicate with fork server (OOM?)");
  }
}
```

二者的主要区别在于AFLNET多了一个`send_over_network()`函数。

在forkserver已经初始化并且是灰盒模式的情况下，AFL会直接读取状态管道内容，即等待程序执行完毕。而AFLNET会先调用`send_over_network()`，然后再等待程序执行。这个函数起到了测试端向服务端喂数据的功能，无论是种子数据或者经过变异后的数据，都经过这个函数发送给server进行后续代码执行。

## send_over_network 的具体实现

首先是定义一些变量以及执行一些初始化操作：

- `likely_buggy` 用来标志服务端是否可能出现crash
- `serv_addr` 服务端地址套接字地址
- `local_serv_addr` 本地地址套接字地址

判断是否存在清除脚本，如果存在则执行，一般用于清除执行服务端所产生的痕迹。
`sleep()`一定的时间用来等待服务端初始化
清除缓冲区并重置缓冲区大小
创建TCP/UDP套接字

```c
int n;
u8 likely_buggy = 0;
struct sockaddr_in serv_addr;
struct sockaddr_in local_serv_addr;
// Clean up the server if needed
if (cleanup_script) system(cleanup_script);
// Wait a bit for the server initialization
usleep(server_wait_usecs);
// Clear the response buffer and reset the response buffer size
if (response_buf) {
  ck_free(response_buf);
  response_buf = NULL;
  response_buf_size = 0;
}
if (response_bytes) {
  ck_free(response_bytes);
  response_bytes = NULL;
}
int sockfd = -1;
if (net_protocol == PRO_TCP)
  sockfd = socket(AF_INET, SOCK_STREAM, 0);
else if (net_protocol == PRO_UDP)
  sockfd = socket(AF_INET, SOCK_DGRAM, 0);
if (sockfd < 0) {
  PFATAL("Cannot create a socket");
}
```

### 设置超时与服务端套接字

接下来是设置超时时间以及建立服务端套接字

```c
// Set timeout for socket data sending/receiving -- otherwise it causes a big delay
// if the server is still alive after processing all the requests
struct timeval timeout;
timeout.tv_sec = 0;
timeout.tv_usec = socket_timeout_usecs;
setsockopt(sockfd, SOL_SOCKET, SO_SNDTIMEO, (char *)&timeout, sizeof(timeout));
memset(&serv_addr, '0', sizeof(serv_addr));
serv_addr.sin_family = AF_INET;
serv_addr.sin_port = htons(net_port);
serv_addr.sin_addr.s_addr = inet_addr(net_ip);
```

### 绑定本地端口

然后是检查`local_port`是否大于0，这里的`local_port`是由`-l`参数指定的，从注释中也能理解到，有一些协议的响应包只会发给特定的端口，所以框架提供了`-l`参数指定接收端口，并将`sockfd`用`bind()`函数绑定到指定端口上，如果没有指定则无需`bind()`

```c
// This piece of code is only used for targets that send responses to a specific port number
// The Kamailio SIP server is an example. After running this code, the intialized sockfd
// will be bound to the given local port
if (local_port > 0) {
  local_serv_addr.sin_family = AF_INET;
  local_serv_addr.sin_addr.s_addr = INADDR_ANY;
  local_serv_addr.sin_port = htons(local_port);
  local_serv_addr.sin_addr.s_addr = inet_addr("127.0.0.1");
  if (bind(sockfd, (struct sockaddr *) &local_serv_addr, sizeof(struct sockaddr_in)))  {
    FATAL("Unable to bind socket on local source port");
  }
}
```

### 连接服务端

连接服务端，如果没有连上则等待一定时间之后继续尝试

```c
if (connect(sockfd, (struct sockaddr *)&serv_addr, sizeof(serv_addr)) < 0) {
  // If it cannot connect to the server under test
  // try it again as the server initial startup time is varied
  for (n=0; n < 1000; n++) {
    if (connect(sockfd, (struct sockaddr *)&serv_addr, sizeof(serv_addr)) == 0)
      break;
    usleep(1000);
  }
  if (n == 1000) {
    close(sockfd);
    return 1;
  }
}
```

### 发送数据逻辑

接下来就是正式发送数据的逻辑：

首先进行一次接收，这里的`net_recv()`后面说，这里只需要知道如果正常接收的话会返回0即可，第一次接收是为了处理刚连接上服务端的时候服务端发来的一些先行消息。如果接收出现问题则会提前跳转到`HANDLE_RESPONSES`部分

然后初始化`messages_sent`，这个变量表示已发送了几次消息，然后`kl_messages`链表，发送每个状态对应的数据并进行接收

`net_recv()`函数每次执行都会更新`response_buf`和`response_buf_size`，每发送完一个状态的数据后就将当前`buf_size`记录到`prev_buf_size`中，然后接收本次发送的数据所获得的响应数据。

更新`response_bytes`，它是一个全局变量，用于记录协议走到每个状态的接收缓冲区大小

如果`buf_size`在执行完`net_recv()`后没有任何变化说明可能出现了crash，所以将`likely_buggy`置1
