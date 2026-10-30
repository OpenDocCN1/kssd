
```c
// retrieve early server response if needed
if (net_recv(sockfd, timeout, poll_wait_msecs, &response_buf, &response_buf_size)) goto HANDLE_RESPONSES;
// write the request messages
kliter_t(lms) *it;
messages_sent = 0;
for (it = kl_begin(kl_messages); it != kl_end(kl_messages); it = kl_next(it)) {
    n = net_send(sockfd, timeout, kl_val(it)->mdata, kl_val(it)->msize);
    messages_sent++;
    // Allocate memory to store new accumulated response buffer size
    response_bytes = (u32 *) ck_realloc(response_bytes, messages_sent * sizeof(u32));
    // Jump out if something wrong leading to incomplete message sent
    if (n != kl_val(it)->msize) {
        goto HANDLE_RESPONSES;
    }
    // retrieve server response
    u32 prev_buf_size = response_buf_size;
    if (net_recv(sockfd, timeout, poll_wait_msecs, &response_buf, &response_buf_size)) {
        goto HANDLE_RESPONSES;
    }
    // Update accumulated response buffer size
    response_bytes[messages_sent - 1] = response_buf_size;
    // set likely_buggy flag if AFLNet does not receive any feedback from the server
    // it could be a signal of a potentiall server crash, like the case of CVE-2019-7314
    if (prev_buf_size == response_buf_size) likely_buggy = 1;
    else likely_buggy = 0;
}
```

在看 `HANDLE_RESPONSES` 部分之前先来简单看看 `net_recv` 是怎么写的：

建立一个 1000 字节的缓冲区，然后以缓冲区为单位调用原生的 `recv` 函数进行循环接收，每次接收的字节数 `n` 如果小于 0 则返回 1，说明出现问题，否则就用 `realloc` 函数扩展 `response_buf`，并更新 `response_buf_size` 的值

```c
int net_recv(int sockfd, struct timeval timeout, int poll_w, char **response_buf, unsigned int *len) {
    char temp_buf[1000];
    int n;
    struct pollfd pfd[1];
    pfd[0].fd = sockfd;
    pfd[0].events = POLLIN;
    int rv = poll(pfd, 1, poll_w);
    setsockopt(sockfd, SOL_SOCKET, SO_RCVTIMEO, (char *)&timeout, sizeof(timeout));
    // data received
    if (rv > 0) {
        if (pfd[0].revents & POLLIN) {
            n = recv(sockfd, temp_buf, sizeof(temp_buf), 0);
            if ((n < 0) && (errno != EAGAIN)) {
                return 1;
            }
            while (n > 0) {
                usleep(10);
                *response_buf = (unsigned char *)ck_realloc(*response_buf, *len + n + 1);
                memcpy(&(*response_buf)[*len], temp_buf, n);
                (*response_buf)[(*len) + n] = '\0';
                *len = *len + n;
                n = recv(sockfd, temp_buf, sizeof(temp_buf), 0);
                if ((n < 0) && (errno != EAGAIN)) {
                    return 1;
                }
            }
        }
    } else if (rv < 0) // an error was returned
        return 1;
    // rv == 0 poll timeout or all data pending after poll has been received successfully
    return 0;
}
```

最后来看 `HANDLE_RESPONSES` 部分

首先再进行一次接收，猜测是为了防止出现服务端反应较慢导致一些数据没有被接收到，然后更新最后一个状态的 `buf_size`

然后是一个等待服务端执行完毕的逻辑，但是实现的方法比较有趣，先将 `session_virgin_bits` 初始化为全 1，然后不断执行 `has_new_bits` 函数，因为在上一步初始化为全 1，所以 `has_new_bits` 会随着服务端程序的执行不断的覆盖到新的路径，所以会不停的返回 2，如果返回值不是 2，即有任何新的路径覆盖到，说明服务端本次执行已经结束了。

（这里的逻辑我认为其实还是有一些缺陷的，如果有些服务端有一些逻辑需要一定的时间才能到达的话其实会被框架漏掉，但是这种情况比较少并且如果改成连续多少次返回值不是 2 的话会牺牲掉一定的效率，并不值得。）

关闭 `sockfd`，`kill` 子进程并为了使其能够优雅的退出进行一定的等待，

```c
HANDLE_RESPONSES:
    net_recv(sockfd, timeout, poll_wait_msecs, &response_buf, &response_buf_size);
    if (messages_sent > 0 && response_bytes != NULL) {
        response_bytes[messages_sent - 1] = response_buf_size;
    }
    // wait a bit letting the server to complete its remaining task(s)
    memset(session_virgin_bits, 255, MAP_SIZE);
    while(1) {
        if (has_new_bits(session_virgin_bits) != 2)
            break;
    }
    close(sockfd);
    if (likely_buggy && false_negative_reduction)
        return 0;
    if (terminate_child && (child_pid > 0)) kill(child_pid, SIGTERM);
    // give the server a bit more time to gracefully terminate
    while(1) {
        int status = kill(child_pid, 0);
        if ((status != 0) && (errno == ESRCH))
            break;
    }
    return 0;
}
```

AFLNET 的网络通信部分基本就是这些，后面会继续阅读有关状态图的逻辑，搞清楚是如何做到在状态图中进行转换并以此指导模糊测试数据变异的。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#自动化挖掘](https://bbs.kanxue.com/forum-150-1-155.htm)

[#Fuzz](https://bbs.kanxue.com/forum-150-1-157.htm)

[#Linux](https://bbs.kanxue.com/forum-150-1-161.htm)
