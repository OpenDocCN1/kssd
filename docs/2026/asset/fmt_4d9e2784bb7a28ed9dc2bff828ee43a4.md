
# IPC 进程间通信

## System V 消息队列操作

```c
#include <sys/msg.h>
int msgsnd(int msqid, const void *msgp, size_t msgsz, int msgflg);
ssize_t msgrcv(int msqid, void *msgp, size_t msgsz, long msgtyp, int msgflg);
```

* <mark class="encrypted">758K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0j5h3^5%4i4K6u0W2L8%4u0Y4i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6E0j5h3&6Q4y4f1y4Q4x3X3c8H3j5h3N6W2M7#2)9J5c8X3#2S2L8U0u0Q4x3V1k6E0M7$3N6K6L8X3c8Q4x3X3f1J5i4K6u0W2K9s2c8E0L8l9`.`.</mark>

## 3、`msgrcv` sys v消息队列操作

```c
#include <sys/msg.h>
int msgsnd(int msqid, const void *msgp, size_t msgsz, int msgflg);
ssize_t msgrcv(int msqid, void *msgp, size_t msgsz, long msgtyp, int msgflg);
```

* <a href="elink@6feK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0j5h3^5%4i4K6u0W2L8%4u0Y4i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6E0j5h3&6Q4y4f1y4Q4x3X3c8H3j5h3N6W2M7#2)9J5c8X3#2S2L8U0u0Q4x3V1k6E0M7$3N6J5j5%4k6Q4x3X3f1J5i4K6u0W2K9s2c8E0L8l9`.`."><mark class="encrypted">562K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0j5h3^5%4i4K6u0W2L8%4u0Y4i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6E0j5h3&6Q4y4f1y4Q4x3X3c8H3j5h3N6W2M7#2)9J5c8X3#2S2L8U0u0Q4x3V1k6E0M7$3N6J5j5%4k6Q4x3X3f1J5i4K6u0W2K9s2c8E0L8l9`.`.</mark></a>

在 Go 代码实现之前，需要了解进程间的通信 IPC（Inter-Process Communication）。

* 这里的解释来自于企鹅大佬的公众号文章，我在这只是把一些精简的复制过来了，完整的可点击链接<mark class="encrypted">4f1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7q4)9J5k6i4N6W2K9i4S2A6L8W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8W2N6Y4h3X3q4e0y4i4M7#2d9g2S2S2x3@1W2n7c8#2u0K6f1p5E0@1j5W2p5`.</mark>

![0321134b5ec59da94edf00ee16d97d52.png](upload/attach/202204/809626_HNEAC9PDY578KRW.jpg)

其中，最初 Unix IPC 包括：管道、FIFO、信号；System V IPC 包括：System V 消息队列、System V 信号灯、System V 共享内存区；Posix IPC 包括：Posix 消息队列、Posix 信号灯、Posix 共享内存区。简单说明一下，现有大部分 Unix 和流行版本都是遵循 POSIX 标准的，而 Linux 从一开始就遵循 POSIX 标准。

## 进程间的七大通信方式

### `signal`

#### 信号（Signal）的本质

信号是进程间通信机制中唯一的异步通信机制，可以看作是异步通知，通知接收信号的进程有哪些发生事情了。

#### 信号（Signal）的来源

信号事件的发生有两个来源：

#### 进程对信号（Signal）响应的三种方式

1. 忽略信号，不作任何处理，不过其中有两个不能忽略的：`SIGKILL`和`SIGSTOP`
2. 捕捉信号，定义信号处理函数，当信号发生时，执行相应的处理函数
3. 执行缺省操作，Linux 对每种信号都规定了默认操作

#### 信号（Signal）发送的主要函数

* `kill()`
* `raise()`
* `siguqeue()`
* `alarm()`
* `setitimer()`
* `abort()`

#### 信号（Signal）的处理

如果进程要处理某一信号，那么就要在进程中安装该信号。**安装信号主要用来确定信号值及进程针对该信号值的动作之间的映射关系，即进程将要处理哪个信号；该信号被传递给进程时，将执行何种操作。**

### `file`

在 Linux 中，每打开一个文件，就会产生一个文件控制块，而文件控制块与文件描述符是一一对应的，因此可以通过文件描述符的操作进而对文件进行操作。

文件描述符的分配原则：

* 使用的 API：`read()`和`write()`
* 通过`pause()`等待对方发起一个信号，已确认可以开始执行下一次读写操作；`pause()`只要接受到任何的信号，立马就可以往下执行
* 通过`kill()`方法向对方发出明确的信号：可以开始下一步执行（读，写）

缺点：

* 文件通信没有访问规则
* 访问速度慢

### `pipe`

**管道（Pipe）及有名管道（named pipe）**

* （1）管道是半双工的，数据只能向一个方向流动；需要双方通信时，需要建立起两个管道；
* （2）只能用于父子进程或者兄弟进程之间（具有亲缘关系的进程）；
* （3）单独构成一种独立的文件系统：管道对于管道两端的进程而言，就是一个文件，但它不是普通的文件，它不属于某种文件系统，而是自立门户，单独构成一种文件系统，并且只存在与内存中。
* （4）数据的读出和写入：一个进程向管道中写的内容被管道另一端的进程读出。写入的内容每次都添加在管道缓冲区的末尾，并且每次都是从缓冲区的头部读出数据。

### `shm`

#### 共享内存（`shm`）概念

使得多个进程可以访问同一块内存空间。

#### 进程对于共享内存的操作与管理主要是：

* （1）申请创建一个共享内存区域（操作系统内核是不可能主动为进程创建共享内存），操作系统内核得到申请然后创建。
* （2）申请使用一个已存在的共享内存区域。
* （3）申请释放共享内存区域（操作系统内核也是不可能主动释放共享内存区域），操作系统内核得到申请然后释放。

共享内存允许两个或多个进程共享一给定的存储区，因为数据不需要来回复制，所以是最快的一种进程间通信机制。

### `sem`

#### 信号量（`semaphore`）的概念

主要作为进程间以及同一进程不同线程之间的同步手段。

#### 存在的问题

进程在访问共享资源的时候存在冲突的问题，必须有一种强制手段说明这些共享资源的访问规则。

#### `sem`

表示的是一种共享资源的个数，对共享资源的访问规则。

#### 访问规则

* （1）用一种数量单位去标识某一种共享资源的个数。
* （2）当有进程需要访问对应的共享资源的时候，则需要先查看申请，根据当前资源对应的可用数量进行申请。
* （3）资源的管理者（也就是操作系统内核）就使用当前的资源个数减去要申请的资源的个数。如果结果 `>= 0` 表示有可用资源，允许该进程的继续访问；否则表示资源不可用，通知进程（暂停或立即返回）。
* （4）资源数量的变化就表示资源的占用和释放。占用：使得可用资源减少；释放：使得可用资源增加。

### `msg`

#### 消息队列（message）的概念

消息队列就是一个消息的链表。可以把消息看作一个记录，具有特定的格式以及特定的优先级。

对消息队列有写权限的进程可以向中按照一定的规则添加新消息；对消息队列有读权限的进程则可以从消息队列中读走消息。

![78cff2e860d3c98f34062ddad4b6c8a2.png](upload/attach/202204/809626_ZQW3V965C7E95VB.jpg)

### `socket`

#### 套接字（`socket`）

一个套接口可以看作是进程间通信的端点（endpoint），每个套接口的名字都是唯一的（唯一的含义是不言而喻的），其他进程可以发现、连接并且与之通信。

## IPC 常用命令

```bash
# 查看
ipcs
# 创建
ipcmk
# 删除
ipcrm
```

## Go 的实现

```go
package main

import (
    "log"
    "os"
    "os/exec"
    "syscall"
)

func main() {
    // fork出新的进程。并执行sh命令
    cmd := exec.Command("sh")
    // 设置Cloneflags。 clone新的IPC命名空间
    cmd.SysProcAttr = &syscall.SysProcAttr{
        Cloneflags: syscall.CLONE_NEWIPC,
    }
    // 输入输出
    cmd.Stdin = os.Stdin
    cmd.Stderr = os.Stderr
    cmd.Stdout = os.Stdout
    if err := cmd.Run(); err != nil {
        log.Fatal(err)
    }
}
```

在 `go build` 后运行程序，查看 `ipcs`。

![a328485fd68a4f3e75933b88196b7f29.png](upload/attach/202204/809626_6NFP6SXYJ569N7S.jpg)

新建一个消息队列

![0e2602cc5bcdc7d29a8587074678f6e6.png](upload/attach/202204/809626_9DF9BYDXJVCZHZK.jpg)

会宿主机中无法看到创建的消息队列

![2ff73fd55912af42409ed3d5a842a315.png](upload/attach/202204/809626_PRW7QA3YMCCFWEQ.jpg)

## PID NameSpaces

用于隔离进程 ID 号，换句话说，不同的 PID NameSces 中的进程可以有一模一样的 PID，PID 命名空间的主要好处之一就是容器可以在主机之间迁移，同时为容器内的进程保持相同的进程 PID，PID NameSpaces 还允许每个容器又用自己的 `init`（PID 1），也就是 Linux 中最大的父进程，管理各种系统初始化任务并在他们终止前 `kill` 掉其他的子进程。

当我们创建一个新的 PID NameSpace 的时候，第一个进程的 PID 会被赋值为 1，进程退出时，内核就会 `kill` NameSpace 内的其他进程。

```go
package main

import (
    "log"
    "os"
    "os/exec"
    "syscall"
)

func main() {
    // fork出新的进程。并执行sh命令
    cmd := exec.Command("sh")
    // 设置Cloneflags。 clone新的PID命名空间
    cmd.SysProcAttr = &syscall.SysProcAttr{
        Cloneflags: syscall.CLONE_NEWPID, // 注释这一行前后，对比一下PID的隔离情况
    }
    // 输入输出
    cmd.Stdin = os.Stdin
    cmd.Stderr = os.Stderr
    cmd.Stdout = os.Stdout
    if err := cmd.Run(); err != nil {
        log.Fatal(err)
    }
}
```

![0379e0a28d161905a64166014c282d93.png](upload/attach/202204/809626_J8R9YKXDKDNMJN9.jpg)

在 `go build` 后我们输出当前进程的 PID 可以发现为 1。

## NetWork NameSpaces

提供与网络相关的系统资源隔离，因此每个 NetWork NameSpaces 都有自己的网络设备，IP 地址，IP 路由表，`/proc/net` 目录，端口号等。

那么从网络的角度来看，NetWork NameSpaces 使每个容器都可以拥有自己的（虚拟）网络设备和绑定到每个命名空间端口号空间的应用程序，主机系统中合适的路由规则可以将网络数据包定向到与特定容器相关的网络设备，例如：可以在同一个主机系统上拥有多个容器化的 Web 服务器，每个服务器都绑定到每个容器的 NetWork NameSpaces 中的 80 端口。

![29574aa56e84bdf0df3535eb21ee3c75.png](upload/attach/202204/809626_AH6WHYUZV74E7NK.jpg)

### Go 的实现

```go
package main

import (
    "log"
    "os"
    "os/exec"
    "syscall"
)

func main() {
    // fork出新的进程。并执行sh命令
    cmd := exec.Command("sh")
    // 设置Cloneflags。 clone新的network命名空间
    cmd.SysProcAttr = &syscall.SysProcAttr{
        Cloneflags: syscall.CLONE_NEWNET,
    }
    // 输入输出
    cmd.Stdin = os.Stdin
    cmd.Stderr = os.Stderr
    cmd.Stdout = os.Stdout
    if err := cmd.Run(); err != nil {
        log.Fatal(err)
    }
}
```

![95dcc61998522eaa8af9c56c4e0aad80.png](upload/attach/202204/809626_6SBHXR4UR627E56.jpg)

可以看到隔离之后，只有一个 `link/loopback` 本地回环。

## User NameSpaces

用于隔离用户和组 ID 号的空间，换句话说：进程的用户和组 ID 在 User NameSpaces 内部和外部可以不同，这里最有意思的是：一个进程可以在 User NameSpaces 之外拥有一个普通的非特权用户 ID，同时在命名空间内拥有一个用户 ID 为 0，这意味着该进程对 User NameSpaces 内的操作具有完全的 root 权限，但对命名空间外的操作没有特权。

从 Linux 3.8 开始，非特权进程可以创建 User NameSpaces，这给应用程序开启了许多有意思的可能性：由于原本非特权进程可以在 User NameSpaces 内拥有 root 权限，因此非特权应用程序现在可以访问以前仅限于根路径，`Eric Biederman` 为使 User NameSpaces 实现安全和正确付出了很多努力。然而，这项工作带来的变化是微妙而广泛的。因此，User NameSpaces 可能会出现一些尚待发现和修复的未知安全问题。

> ps: 这里我理解的意思是，一个普通权限的用户，可以创建一个 User NameSpaces，在这个 User NameSpaces 外面，它只是一个普通权限的用户，但是在 User NameSpaces 中拥有绝对的 root 权限

### Go 的实现

```go
package main

import (
    "log"
    "os"
    "os/exec"
    "syscall"
)

func main() {
    // fork出新的进程。并执行sh命令
    cmd := exec.Command("sh")
    // 设置Cloneflags。 clone新的user命名空间
    cmd.SysProcAttr = &syscall.SysProcAttr{
        Cloneflags: syscall.CLONE_NEWUSER,
        UidMappings: []syscall.SysProcIDMap{
            {
                ContainerID: 0,
                HostID:      0,
                Size:        1,
            },
        },
        GidMappings: []syscall.SysProcIDMap{
            {
                ContainerID: 0,
                HostID:      0,
                Size:        1,
            },
        },
    }
    // 输入输出
    cmd.Stdin = os.Stdin
    cmd.Stderr = os.Stderr
    cmd.Stdout = os.Stdout
    if err := cmd.Run(); err != nil {
        log.Fatal(err)
    }
}
```

上面的代码中有一个 `GidMappings` 和 `UidMappings`。

![df346365badd787b87860bc4db4fd263.png](upload/attach/202204/809626_BSSMNY6Z5Y65DDY.jpg)

从这个结构体的解释中我们可以发现，一个是用来映射用户 ID 的，一个是用来映射用户组的。

可以运行后查看用户的隔离性。

![dd00f737baf7262fd3a5686485261437.png](upload/attach/202204/809626_3QRE6GS27KGV28J.jpg)

## Mount namespaces
