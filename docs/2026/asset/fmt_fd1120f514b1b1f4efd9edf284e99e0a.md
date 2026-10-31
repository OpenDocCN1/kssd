
# [原创] Kernel PWN 从入门到提升

## 前言

> 介于本人在入门 kernel pwn 的时候觉得当前 trick 种类繁多，前置知识也多得吓人，有点不知所措，且有些大佬的博客经常对一些我个人认为比较重要的点一句话带过，导致缺乏经验的我在学习过程中屡屡碰壁。所以我决定用此文章结合一道不错的例题尽可能详细地讲一下 kernel pwn 从入门过渡到较高难度的部分，供想要学习 kernel pwn 的小伙伴们参考。

> 在开始看这篇文章之前，我希望小伙伴们已经掌握了 kernel pwn 一些最基本的操作，例如装好 kernel pwn 所需要的前置环境。这一部分内容的优秀教程并不少。

> 另外，如果在阅读的过程中发现任何问题，都欢迎来和我交流指正。

# BASIC

## Environment

在学习 kernel pwn 之前，需要搭建好很多前置环境：

- `qemu`
- `busybox`
- 编译 Linux 内核（可选）

至于具体的安装过程并不在本文的讨论范围内，如果还没完成，先自行百度解决。

## 文件系统

kernel 题一般都会给出一个打包好的文件系统，因此需要掌握常用到的打包、解包命令：

```shell
find . | cpio -o --format=newc > ./rootfs.cpio
cpio -idmv < ./rootfs.cpio
```

（有时解包出来很奇怪，可能是原始 `cpio` 文件其实是以 `gz` 格式压缩后的，先用 `gunzip` 解压一遍。）

## cred 结构体

kernel 使用 `cred` 结构体记录了进程的权限，如果能劫持或伪造 `cred` 结构体，就能改变当前进程的权限。

原型如下：

```c
struct cred {
    atomic_t usage;
#ifdef CONFIG_DEBUG_CREDENTIALS
    atomic_t subscribers; /* number of processes subscribed */
    void *put_addr;
    unsigned magic;
#define CRED_MAGIC 0x43736564
#define CRED_MAGIC_DEAD 0x44656144
#endif
    kuid_t uid; /* real UID of the task */
    kgid_t gid; /* real GID of the task */
    kuid_t suid; /* saved UID of the task */
    kgid_t sgid; /* saved GID of the task */
    kuid_t euid; /* effective UID of the task */
    kgid_t egid; /* effective GID of the task */
    kuid_t fsuid; /* UID for VFS ops */
    kgid_t fsgid; /* GID for VFS ops */
    unsigned securebits; /* SUID-less security management */
    kernel_cap_t cap_inheritable; /* caps our children can inherit */
    kernel_cap_t cap_permitted; /* caps we're permitted */
    kernel_cap_t cap_effective; /* caps we can actually use */
    kernel_cap_t cap_bset; /* capability bounding set */
    kernel_cap_t cap_ambient; /* Ambient capability set */
#ifdef CONFIG_KEYS
    unsigned char jit_keyring; /* default keyring to attach requested
                                * keys to */
    struct key __rcu *session_keyring; /* keyring inherited over fork */
    struct key *process_keyring; /* keyring private to this process */
    struct key *thread_keyring; /* keyring private to this thread */
    struct key *request_key_auth; /* assumed request_key authority */
#endif
#ifdef CONFIG_SECURITY
    void *security; /* subjective LSM security */
#endif
    struct user_struct *user; /* real user ID subscription */
    struct user_namespace *user_ns; /* user_ns the caps and keyrings are relative to. */
    struct group_info *group_info; /* supplementary groups for euid/fsgid */
    struct rcu_head rcu; /* RCU deletion hook */
} __randomize_layout;
```

一般而言，我们需要想办法将 `uid` 和 `gid` 设置为 `0`（root 的 `uid` 和 `gid` 均为 `0`）。

如果能劫持到程序流程，执行以下函数也可以达到相同效果：

```c
commit_creds(prepare_kernel_cred(0));
commit_creds(init_cred);
```

## 内核态函数

运行在内核态的函数会和用户态有些许不同：

- `printf` -> `kprintf`
- `memcpy` -> `copy_to_user` / `copy_from_user`

内核的动态分配并不会采用用户态的 `glibc`，它的堆分配器是 `SLAB` 或 `SLUB`。常使用的函数如下：

- `malloc` -> `kmalloc`
- `free` -> `kfree`

为了安全考虑，内核态也只能运行内核态的函数（`smep`），想要运行 `system` 等函数，必须手动切换回用户态。

常用的指令是 `swapgs` 和 `iretq`（或者 `swapgs_restore_regs_and_return_to_usermode` 函数，直接对 `CR3` 寄存器的第 13 位取反来完成切换页表的操作，该函数在 KPTI 开启的版本中依然有效，而 `swapgs` 往往会寄）。

然后需要在栈上存一些上下文：

```c
struct pt_regs {
    /* ...................... */
    /* Return frame for iretq */
    unsigned long ip;
    unsigned long cs;
    unsigned long flags;
    unsigned long sp;
    unsigned long ss;
    /* top of stack page */
};
```

## GDB 远程调试

以 `babydriver` 这题为例，先使用脚本 `extract-vmlinux` 提取出带符号的源码：

```shell
./extract-vmlinux ./bzImage > ./vmlinux
```

（脚本源码：<a href="elink@b52K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6@1L8%4u0$3j5h3I4V1M7#2)9J5c8X3I4A6L8Y4g2^5i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8Y4y4U0M7X3W2H3N6s2y4Q4x3V1k6W2P5s2c8J5j5h3y4@1i4K6u0V1N6X3#2D9K9h3&6#2P5l9`.`."><mark class="encrypted">5ddK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6@1L8%4u0$3j5h3I4V1M7#2)9J5c8X3I4A6L8Y4g2^5i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8Y4y4U0M7X3W2H3N6s2y4Q4x3V1k6W2P5s2c8J5j5h3y4@1i4K6u0V1N6X3#2D9K9h3&6#2P5l9`.`.</mark></a>）

（或者用这个<a href="elink@883K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0j5i4u0A6L8W2)9J5k6r3#2Q4x3V1k6$3L8h3I4A6L8Y4g2^5i4K6u0V1N6r3!0Q4x3X3c8W2L8r3j5`."><mark class="encrypted">390K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0j5i4u0A6L8W2)9J5k6r3#2Q4x3V1k6$3L8h3I4A6L8Y4g2^5i4K6u0V1N6r3!0Q4x3X3c8W2L8r3j5`.</mark></a>）

在 qemu 中找到 `babydriver.ko` 代码段的起始地址。

![](upload/attach/202303/951122_HMKWJGHXCAUUD7A.png)

启动 gdb 过后导入符号表：

```shell
add-symbol-file ./lib/modules/4.4.72/babydriver.ko 0xffffffffc0000000
```

![](upload/attach/202303/951122_W297UAMBZN9375Q.png)

然后在 `boot.sh` 中添加以下参数。

![](upload/attach/202303/951122_DYPZDETZVMJNK9C.png)

（直接 `-s` 也行。）

重新启动 qemu 过后，gdb 远程连接：

```shell
pwndbg> target remote 127.0.0.1:1234
```

![](upload/attach/202303/951122_RNNSCCUGUQSR43P.png)

这里给出我常用的一些打包和调试的脚本。

### `pack.sh`

```sh
#!/bin/zsh
gcc \
./exp.c \
-o exp \
-masm=intel \
--static \
-g
chmod 777 ./exp
find . | cpio -o --format=newc > ./rootfs.cpio
chmod 777 ./rootfs.cpio
```

### `gdbinit`

```sh
file ./vmlinux
target remote 127.0.0.1:1234
c
```

## 远程脚本

为了减小远程 exp 的体积，使用 musl 进行静态编译（~~再 O3 优化一下就更小啦~~）。

```python
import sys
import os
from pwn import *
import string

context.log_level = 'debug'
sla = lambda x, y: p.sendlineafter(x, y)
sa = lambda x, y: p.sendafter(x, y)
ru = lambda x: p.recvuntil(x)

p = remote('127.0.0.1', 1234)

def send_cmd(cmd):
    sla('$ ', cmd)

def upload():
    lg = log.progress('Upload')
    with open('exp', 'rb') as f:
        data = f.read()
    encoded = base64.b64encode(data)
    encoded = str(encoded)[2:-1]
    for i in range(0, len(encoded), 300):
        lg.status('%d / %d' % (i, len(encoded)))
        send_cmd('echo -n "%s" >> benc' % (encoded[i:i + 300]))
    send_cmd('cat benc | base64 -d > bout')
    send_cmd('chmod +x bout')
    lg.success()

os.system('musl-gcc -w -s -static -o3 exp.c -o exp')
upload()
p.interactive()
```

# ATTACK

## Kernel UAF

### babydriver

#### 分析

这是 ciscn 2017 年的一道经典 kernel pwn 入门题。

解压 `rootfs.cpio` 后，在 `/lib/modules/4.4.72` 中找到了 LKM 文件 `babydriver.ko`。

`checksec` 只开了 `nx`，且没有去除符号表，很方便调试和分析。

直接丢 `ida` 分析：

```c
int __fastcall babyrelease(inode *inode, file *filp)
{
    _fentry_(inode, filp);
    kfree(babydev_struct.device_buf);
    printk("device release\n");
    return 0;
}
```

在 `babyrelease` 中 `kfree()` 之后没有将 `babydev_struct.device_buf` 清空，从而导致了 UAF 漏洞。

而且 `babydev_struct` 是一个 `babydevice_t` 类型的公共变量，结构如下：

```c
struct babydevice_t
{
    char *device_buf;
    size_t device_buf_len;
};
```

`device_buf` 是存一个缓冲区的指针，`device_buf_len` 存该缓冲区大小。

其他的函数都很常规。

`babyopen` 在打开一个设备的时候简单设置了一下 `babydev_struct` 的值：

```c
int __fastcall babyopen(inode *inode, file *filp)
{
    _fentry_(inode, filp);
    babydev_struct.device_buf =
        (char *)kmem_cache_alloc_trace(kmalloc_caches[6], 0x24000C0LL, 0x40LL);
    babydev_struct.device_buf_len = 64LL;
    printk("device open\n");
    return 0;
}
```

`babywrite` 和 `babyread` 都只检查了一下 `device_buf` 指针是否为空和是否越界，然后对 `device_buf` 进行常规的读写：

```c
ssize_t __fastcall babywrite(file *filp, const char *buffer,
                             size_t length, loff_t *offset)
{
    size_t v4; // rdx
    ssize_t result; // rax
    ssize_t v6; // rbx

    _fentry_(filp, buffer);
    if (!babydev_struct.device_buf)
        return -1LL;

    result = -2LL;
    if (babydev_struct.device_buf_len > v4)
    {
        v6 = v4;
        copy_from_user();
        result = v6;
    }
    return result;
}
```

```c
ssize_t __fastcall babyread(file *filp, char *buffer,
                            size_t length, loff_t *offset)
{
    size_t v4; // rdx
    ssize_t result; // rax
    ssize_t v6; // rbx

    _fentry_(filp, buffer);
    if (!babydev_struct.device_buf)
        return -1LL;

    result = -2LL;
    if (babydev_struct.device_buf_len > v4)
    {
        v6 = v4;
        copy_to_user(buffer);
        result = v6;
    }
    return result;
}
```

`babyioctl` 比较有意思，当第二个参数 `command` 为 `0x10001` 时，可以重新 `kmalloc` 一块指定大小的 object 到 `babydev_struct.device_buf`，从而修改了 `babydev_struct` 的 `device_buf_len` 为一个新值。
