
(1) `ITIMER_REAL`. 定时真实时间, 与 `alarm` 类型相同. 对应信号为 `SIGALRM`.

(2) `ITIMER_VIRT`. 定时进程在用户态下的实际执行时间. 对应信号为 `SIGVTALRM`.

(3) `ITIMER_PROF`. 定时进程在用户态和核心态下的实际执行时间. 对应信号为 `SIGPROF`.

在一个 UNIX 进程中, 不能同时使用 `alarm` 和 `ITIMER_REAL` 类定时器.

结构 `itimerval` 描述了定时器的组成:

```
struct itimerval{
      struct timeval it_interval;
      struct timeval it_value;
}
```

结构成员 `it_value` 指定首次定时的时间, 结构成员 `it_interval` 指定下次定时的时间. 定时器工作时, 先将 `it_value` 的时间值减到 0, 发送一个信号, 再将 `it_value` 赋值为 `it_interval` 的值, 重新开始定时, 如此反复. 如果 `it_value` 值被设置为 0, 则定时器停止定时.

结构 `timeval` 秒数了一个精确到微秒的时间:

```
struct timeval{
      long tv_sec;
      long tv_usec;
}
```

下面设计了一个精确定时器的例子, 进程每 1.5 秒发送定时信号 `SIGPROF`, 用户可键入 `Ctrl+C` 或 `Delete` 结束程序.

```
[bill@billstone Unix_study]$ cat time3.c
#include <sys/select.h>
#include <sys/time.h>
#include <stdio.h>
#include <unistd.h>
#include <signal.h>

int n = 0;

void timefunc(int sig){
        fprintf(stderr, "ITIMER_PROF[%d]\n", n++);
        signal(SIGPROF, timefunc);
}

int main()
{
        struct itimerval value;

        value.it_value.tv_sec = 1;
        value.it_value.tv_usec = 500000;
        value.it_interval.tv_sec = 1;
        value.it_interval.tv_usec = 500000;
        signal(SIGPROF, timefunc);
        setitimer(ITIMER_PROF, &value, NULL);
        while(1);

        return 0;
}
[bill@billstone Unix_study]$ make time3
cc      time3.c   -o time3
[bill@billstone Unix_study]$ ./time3
ITIMER_PROF[0]
ITIMER_PROF[1]
ITIMER_PROF[2]

[bill@billstone Unix_study]$
```

欢迎转载，请保留作者信息

bill@华中科技大学

http://billstone.cublog.cn

### 全局跳转

UNIX 下的 C 语言中,有一对特殊的调用:跳转函数, 原型如下:

```
#include <setjmp.h>
int setjmp(jmp_buf env);
void longjump(jmp_buf env, int val);
```

函数 `setjmp` 存储当前的堆栈环境(包括程序的当前执行位置)到参数 `env` 中,当函数正常调用成功时返回 0. 函数 `longjmp` 恢复保存在 `env` 中堆栈信息, 并使程序转移到 `env` 中保存的位置处重新执行. 这两个函数联合使用, 可以实现程序的重复执行.

函数 `longjmp` 调用成功后, 程序转移到函数 `setjmp` 处执行, 函数 `setjmp` 返回 `val`. 如果参数 `val` 的取值为 0, 为了与上次正常调用 `setjmp` 相区别,函数 `setjmp` 将自动返回 1.

下面是一个使用了跳转语句的例子, 它跳转两次后退出.

```
[bill@billstone Unix_study]$ cat jmp1.c
#include <setjmp.h>

int j = 0;
jmp_buf env;

int main()
{
        auto int i, k = 0;

        i = setjmp(env);
        printf("setjmp = [%d], j = [%d], k = [%d]\n", i, j++, k++);
        if(j > 2)
                exit(0);
        sleep(1);
        longjmp(env, 1);

        return 0;
}
[bill@billstone Unix_study]$ make jmp1
cc      jmp1.c   -o jmp1
[bill@billstone Unix_study]$ ./jmp1
setjmp = [0], j = [0], k = [0]
setjmp = [1], j = [1], k = [1]
setjmp = [1], j = [2], k = [2]
[bill@billstone Unix_study]$
```

其中, `j` 记录了程序的执行次数. 按理说, `k` 的值应该保持不变, 因为当返回到 `setjmp` 重新执行时, 保存的堆栈中 `k` 应该保持 0 不变, 但实际上却变化了. 请高手指点, 是不是 `setjmp` 本身实现的问题(我用的环境是 Red Hat 9)?

### 单线程 I/O 超时处理

UNIX 下的 I/O 超时处理是一个很常见的问题, 它的通常做法是接收输入(或发送输出)后立刻返回, 如果无输入(或输出)则 n 秒后定时返回.

一般情况下, 处理 UNIX 中 I/O 超时的方式有终端方式, 信号跳转方式和多路复用方式等三种. 本节设计一个定时 I/O 的例子, 它从文件描述符 0 中读取一个字符, 当有输入时继续, 或者 3 秒钟后超时退出,并打印超时信息.

#### (1) 终端 I/O 超时方式

利用 `ioctl` 函数, 设置文件描述符对应的标准输入文件属性为”接收输入后立刻返回, 如无输入则 3 秒后定时返回.

```
[bill@billstone Unix_study]$ cat timeout1.c
#include <unistd.h>
#include <termio.h>
#include <fcntl.h>

int main()
{
        struct termio old, new;
        char c = 0;

        ioctl(0, TCGETA, &old);
        new = old;
        new.c_lflag &= ~ICANON;
        new.c_cc[VMIN] = 0;
        new.c_cc[VTIME] = 30;         // 设置文件的超时时间为 3 秒
        ioctl(0, TCSETA, &new);
        if((read(0, &c, 1)) != 1)
                printf("timeout\n");
        else
                printf("\n%d\n", c);
        ioctl(0, TCSETA, &old);

        return 0;
}
[bill@billstone Unix_study]$ make timeout1
cc      timeout1.c   -o timeout1
[bill@billstone Unix_study]$ ./timeout1
x
120
[bill@billstone Unix_study]$ ./timeout1
timeout
[bill@billstone Unix_study]$
```

#### (2) 信号与跳转 I/O 超时方式

在 `read` 函数前调用 `setjmp` 保存堆栈数据并使用 `alarm` 设定 3 秒定时.

```
[bill@billstone Unix_study]$ cat timeout2.c
#include <setjmp.h>
#include <stdio.h>
#include <unistd.h>
#include <signal.h>

int timeout = 0;
jmp_buf env;

void timefunc(int sig){
        timeout = 1;
        longjmp(env, 1);
}

int main()
{
        char c;

        signal(SIGALRM, timefunc);
        setjmp(env);
        if(timeout == 0){
                alarm(3);
                read(0, &c, 1);
                alarm(0);
                printf("%d\n", c);
        }
        else
                printf("timeout\n");

        return 0;
}
[bill@billstone Unix_study]$ make timeout2
cc      timeout2.c   -o timeout2
[bill@billstone Unix_study]$ ./timeout2
v                     // 需要按 Enter 健激活输入
118
[bill@billstone Unix_study]$ ./timeout2
timeout
[bill@billstone Unix_study]$
```

#### (3) 多路复用 I/O 超时方式

一个进程可能同时打开多个文件, UNIX 中函数 `select` 可以同时监控多个文件描述符的输入输出, 进程将一直阻塞, 直到超时或产生 I/O 为止, 此时函数返回, 通知进程读取或发送数据.

函数 `select` 的原型如下:

```
#include <sys/types.h>
#include <sys/times.h>
#include <sys/select.h>
int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
FD_CLR(int fd, fd_set *fdset);    // 从 fdset 中删去文件描述符 fd
FD_ISSET(int fd, fd_set *fdset);   // 查询文件描述符是否在 fdset 中
FD_SET(int fd, fd_set *fdset);     // 在 fdset 中插入文件描述符 fd
FD_ZERO(fd_set *fdset);        // 清空 fdset
```

参数 `nfds` 是 `select` 监控的文件描述符的时间, 一般为监控的最大描述符编号加 1.

类型 `fd_set` 是文件描述符集合, 其元素为监控的文件描述符.

参数 `timeout` 是描述精确时间的 `timeval` 结构,它确定了函数的超时时间,有三种取值情况:

a) `NULL`. 函数永远等待, 直到文件描述符就绪.

b) `0`. 函数不等待, 检查文件描述符状态后立即返回.

c) 其他值. 函数等待文件描述符就绪, 或者定时完成时返回.

函数 `select` 将返回文件描述符集合中已准备好的文件总个数. 函数 `select` 返回就绪文件描述符数量后, 必须执行 `read` 等函数, 否则函数继续返回就绪文件数.

```
[bill@billstone Unix_study]$ cat timeout3.c
#include <stdio.h>
#include <sys/types.h>
#include <sys/times.h>
#include <sys/select.h>

int main()
{
        struct timeval timeout;
        fd_set readfds;
        int i;
        char c;

        timeout.tv_sec = 3;
        timeout.tv_usec = 0;
        FD_ZERO(&readfds);
        FD_SET(0, &readfds);
        i = select (1, &readfds, NULL, NULL, &timeout);
        if(i > 0){
                read(0, &c, 1);
                printf("%d\n", c);
        }
        else if(i == 0)
                printf("timeout\n");
        else
                printf("error\n");

        return 0;
}
[bill@billstone Unix_study]$ make timeout3
cc      timeout3.c   -o timeout3
[bill@billstone Unix_study]$ ./timeout3
x
120
[bill@billstone Unix_study]$
[bill@billstone Unix_study]$ ./timeout3
timeout
[bill@billstone Unix_study]$
```

回答“全局跳转”要点中的问题<红色标记>:

我重新阅读了 APUE 中关于 `setjmp` 的章节，截出关于该问题的解答：

"what are the states of the automatic variables and register variables in the main function?" When main is returned to by the `longjmp`, do these variables have values corresponding to when the `setjmp` was previously called, or are their values left alone so that their values left alone so that their values are whatever they were when `do_line` was called? Unfortunately, the answer is **"it depends."**

下面以一个实例看下具体情况：

