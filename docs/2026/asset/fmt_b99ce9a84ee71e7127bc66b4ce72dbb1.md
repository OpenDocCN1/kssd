
```c
#include <sys/types.h>
#include <sys/xattr.h>
#include <stdio.h>
#include <linux/userfaultfd.h>
#include <pthread.h>
#include <errno.h>
#include <unistd.h>
#include <stdlib.h>
#include <fcntl.h>
#include <signal.h>
#include <poll.h>
#include <string.h>
#include <sys/mman.h>
#include <sys/syscall.h>
#include <sys/ioctl.h>
#include <sys/sem.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <semaphore.h>

static char *page = NULL;
static size_t   page_size;
static pthread_t monitor_thread;

void registerUserFaultFd(void *addr, unsigned long len, void (*handler)(void *));
void errExit(char *msg);
static void *test_thread(void *arg);

int main(int argc, char *argv[])
{
    char *uffd_buf_leak;
    page = malloc(0x1000);
    for(int i = 0; i < 0x20; i++)
    {
        page[i] = "\x62";
    }
    page_size = sysconf(_SC_PAGE_SIZE);

    //申请一块保护内存，我们对该内存进行读写操作就会被userfaultfd给捕获，从而进入到test_thread函数中
    uffd_buf = (char*) mmap(NULL, page_size, PROT_READ | PROT_WRITE, MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
    registerUserFaultFd(uffd_buf, page_size, test_thread);
    printf("\033[1;31;40m[Test:]%016llx\033[0m\n", *uffd_buf);
}

static void *test_thread(void *arg)
{
    struct uffd_msg msg;
    int fault_cnt = 0;
    long uffd;

    struct uffdio_copy uffdio_copy;
    ssize_t nread;

    uffd = (long) arg;

    for (;;)
    {
        struct pollfd pollfd;
        int nready;
        pollfd.fd = uffd;
        pollfd.events = POLLIN;
        nready = poll(&pollfd, 1, -1);

        //自定义代码区域--------------------------
        printf("\033[1;31;40m[Loading handler userfaultfd]%016llx\033[0m\n");
        //-----------------------------------------

        //判断是否正确打开
        if (nready == -1)
            errExit("poll");
        nread = read(uffd, &msg, sizeof(msg));
        if (nread == 0)
            errExit("EOF on userfaultfd!\n");
        if (nread == -1)
            errExit("read");
        if (msg.event != UFFD_EVENT_PAGEFAULT)
            errExit("Unexpected event on userfaultfd\n");

        //数据拷贝区域
        uffdio_copy.src = (unsigned long) page;
        uffdio_copy.dst = (unsigned long) msg.arg.pagefault.address & ~(page_size - 1);
        uffdio_copy.len = page_size;
        uffdio_copy.mode = 0;
        uffdio_copy.copy = 0;
        if (ioctl(uffd, UFFDIO_COPY, &uffdio_copy) == -1)
            errExit("ioctl-UFFDIO_COPY");
        return NULL;
    }
}

void registerUserFaultFd(void *addr, unsigned long len, void (*handler)(void *))
{
    long uffd;
    struct uffdio_api uffdio_api;
    struct uffdio_register uffdio_register;
    int s;

    /* Create and enable userfaultfd object */
    uffd = syscall(__NR_userfaultfd, O_CLOEXEC | O_NONBLOCK);
    if (uffd == -1)
        errExit("userfaultfd");

    uffdio_api.api = UFFD_API;
    uffdio_api.features = 0;
    if (ioctl(uffd, UFFDIO_API, &uffdio_api) == -1)
        errExit("ioctl-UFFDIO_API");

    uffdio_register.range.start = (unsigned long) addr;
    uffdio_register.range.len = len;
    uffdio_register.mode = UFFDIO_REGISTER_MODE_MISSING;
    if (ioctl(uffd, UFFDIO_REGISTER, &uffdio_register) == -1)
        errExit("ioctl-UFFDIO_REGISTER");

    s = pthread_create(&monitor_thread, NULL, handler, (void *) uffd);
    if (s != 0)
        errExit("pthread_create");
}

void errExit(char *msg)
{
    printf("\033[31m\033[1m[x] Error at: \033[0m%s\n", msg);
    exit(EXIT_FAILURE);
}
```

如下效果：

![](https://bbs.kanxue.com/upload/attach/202208/904686_5SW3MV8UZE6ABP2.jpg)

可以看到在试图打印被我们申请读写保护的区域时，会调用到我们的`test_thread`函数中

### (3)配合知识点

通常意义上来讲，UserFaultFD是用在多线程下，用来制造double-free或者UAF的。而在比较严苛的条件下，比如，没法写，或者没办法读的时候，就需要配合以下的两个知识点。

#### `setxattr`

调用链为`SYS_setxattr() -> path_setxattr() -> setxattr()`。

代码如下：

```c
// fs/xattr.c
static long setxattr(struct user_namespace *mnt_userns, struct dentry *d,
                     const char __user *name, const void __user *value, size_t size,
                     int flags)
{
    int error;
    void *kvalue = NULL;
    char kname[XATTR_NAME_MAX + 1];

    if (flags & ~(XATTR_CREATE|XATTR_REPLACE))
        return -EINVAL;

    error = strncpy_from_user(kname, name, sizeof(kname));
    if (error == 0 || error == sizeof(kname))
        error = -ERANGE;
    if (error < 0)
        return error;

    if (size) {
        if (size > XATTR_SIZE_MAX)
            return -E2BIG;
        //申请chunk，基本相当于kmalloc函数，size可控
        kvalue = kvmalloc(size, GFP_KERNEL);
        if (!kvalue)
            return -ENOMEM;
        //从value拷贝内容到kvalue，value可控
        if (copy_from_user(kvalue, value, size)) {
            error = -EFAULT;
            goto out;
        }
        if ((strcmp(kname, XATTR_NAME_POSIX_ACL_ACCESS) == 0) ||
            (strcmp(kname, XATTR_NAME_POSIX_ACL_DEFAULT) == 0))
            posix_acl_fix_xattr_from_user(mnt_userns, kvalue, size);
    }

    error = vfs_setxattr(mnt_userns, d, kname, kvalue, size, flags);
out:
    //释放chunk，基本等于kfree函数
    kvfree(kvalue);

    return error;
}
```

关注点在`kvmalloc`、`copy_from_user`、`kvfree`。

`kvmalloc`中的size可控，`copy_from_user`中的value可控

也就是说当`freelist`中存在我们需要修改的chunk，而该chunk又是我们控制的某个设备内存块时，(通过double-free或者UAF实现)那么我们就可以通过`setxattr`来对该设备内存进行任意写。虽然最后会释放，但是也只会影响内存块中存放下一个chunk地址处的内容0x8个字节，而当我们用不着这个地方的内容时，就不用太关注了。

##### 注：

使用的时候需要注意指定一个当前的exp程序，类似如下，第二个参数字符串任意。

```c
setxattr("/tmp/ufdExp", "PIG-007", &buf, 0x100, 0);
```

#### `msg_msg`

这个下面新创一个点，方便标题跳转

承接上面的知识点

这个在之前也总结过，不过总结得有些错误，也不太完善，这里再好好总结一下

修改时间：2022-05-12

参照：[【NOTES.0x08】Linux Kernel Pwn IV：通用结构体与技巧 - arttnba3's blog](<https://bbs.kanxue.com/elink@ac8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2M7Y4c8@1L8X3u0S2x3#2)9J5k6h3y4F1i4K6u0r3x3U0l9J5x3g2)9J5c8U0p5I4i4K6u0r3x3U0W2Q4x3V1k6z5e0#2c8q4i4K6u0V1x3q4R3H3z5q4)9J5k6p5I4u0e0W2g2j5i4K6u0V1d9@1g2d9e0V1g2x3i4K6u0V1f1q4N6z5i4K6u0V1f1p5q4d9g2q4)9J5k6p5W2h3i4K6u0r3i4K6t1K6i4@1f1#2i4K6R3^5i4K6R3$3i4@1f1&6i4K6R3#2i4K6S2p5i4@1g2r3i4@1u0o6i4K6R3^5c8@1k6b7i4K6u0V1d9@1g2d9e0V1g2x3i4K6u0V1b7f1y4o6e0#2g2z5g2q4!0q4c8W2!0n7b7#2)9^5z5g2!0q4c8W2!0n7b7#2)9&6b7h3#2K6k6%4y4F1k6q4)9J5k6q4!0q4y4#2!0n7x3#2!0n7b7W2!0q4y4#2!0n7b7W2)9&6c8W2!0q4z5q4!0n7x3q4)9^5x3#2!0q4y4#2)9&6y4q4!0m8z5l9>)

[Linux内核中利用msg_msg结构实现任意地址读写 - 安全客，安全资讯平台 (anquanke.com)](<https://bbs.kanxue.com/elink@872K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0t1#2x3U0f1#2z5l9>)
