
# [原创]Google CTF 2021 Linux内核部分题目解析

## Google CTF 2021 / Linux Kernel Exploiting

在本次比赛中，我主要做了 Linux Kernel 方向的漏洞利用，也看了一些别的题，以下是我对题目的一些总结：

- 题目 **fullchain** 是由 **chrome+mojo+kernel 三部分** 构成了一个完整的链条，我个人完成了 kernel 部分，chrome 部分由队里的 2019 师傅完成，这种出题方式我认为很新颖。
- 题目 **COMPRESSION** 是一道涉及压缩的用户态 PWN 题。
- 题目 **ATHERIS** 是一道 Fuzz 类型的题目，**需要利用 Google 最新的 python-fuzz——<a href="elink@0e5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Y4L8$3!0Y4L8r3g2Q4x3V1k6S2N6r3S2W2M7X3W2K6i4K6u0r3">atheris</a> 进行解题**。
- 题目 **MEMSAFETY** 是一道 **Rust+Sandbox** 逃逸。（不过我这边没接触过 rust，所以没有看这道题）
- 题目 **ABC ARM AND AMD** 是一道 shellcode 编写的题目，要求构造一个可见字符 shellcode，**同时可以在 aarch64 和 x86-64 下达成 orw 的效果**。

本次比赛我认为题目质量很高，都是较为新颖的题目形式/考点，或者偏 realword 一点的题目。

[TOC]

## Fullchain(kernel part)

### Overview

本题的文件系统不能直接调试，需要做一定修改之后才能正常启动。

（感谢 @Lime 当时跟我凌晨还在讨论怎么调试这个文件系统）

题目给出了一个 **ext4** 的文件系统的 .img。

```text
rootfs.img: Linux rev 1.0 ext4 filesystem data, UUID=fa0aa1fe-4545-4f4f-9d0e-aec298407b0c (needs journal recovery) (extents) (64bit) (large files) (huge files)
```

这种 `rootfs.img` 不能够直接使用 `cpio` 工具进行解压，而应该用 `mount` + `umount` 进行 unpack-->change-->pack。

具体如下：

```shell
mkdir rootfs
mount -o loop ./rootfs.img ./rootfs # mount到./rootfs目录
```

在 mount 到 `./rootfs` 之后，你可以进入 `./rootfs` 对文件进行修改，而我们的修改可以同步到文件系统中。

```shell
root@ubuntu:~/google-CTF/fullchain/fullchain/rootfs# ls
bin ctf.ko etc exp.c init lib64 media opt root run_chromium.py srv tmp var
boot dev exp home lib lost+found mnt proc run sbin sys usr
```

修改结束后一定要执行：

```sh
umount ./rootfs
```

`umount` 文件系统，然后再用 qemu 启动才可以正常调试。如果不进行 `umount`，无法正常地启动内核。

### 利用思路

本题给了一个裸的 UAF，没有任何保护，利用非常简单，思路较为清晰。

思路：

1. 首先构造一个 UAF 的 slab，在内核堆上喷射 `tty_struct`，然后利用喷上来的 `tty_struct + 0x18` 的指针 leak kernel base。
2. 利用 `tty_struct + 0x50` 处的值 leak slab addr。
3. 然后将 fake ops 放到当前的 slab 的 `tty_struct` 之后。
4. 劫持 `ptmx` 对应的 `ioctl` 函数为 `work_for_cpu_fn`。
5. 利用 `work_for_cpu_fn` 调用 `commit_creds(prepare_kernel_cred(0))` 提权即可。

### 相关文件

我个人的 exp 以及题目的 .c 文件如下：

`exp.c`：<mark class="encrypted">16fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5i4y4@1k6g2)9J5k6i4g2T1N6h3&6@1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8Q4x3V1k6&6k6W2m8r3f1h3R3J5f1e0N6%4i4K6u0r3</mark>
`ctf.c`：<mark class="encrypted">2ceK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5i4y4@1k6g2)9J5k6i4g2T1N6h3&6@1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8Q4x3V1k6X3z5s2q4B7b7Y4A6@1x3@1S2y4i4K6u0r3</mark>

## eBPF

本题偏 **realword**，主要考察了 **Linux 内核 eBPF 虚拟机漏洞利用**，当时做了半天没做出来，最后由 @2019 师傅出了这道题，我跟着师傅的思路再做复盘。

传送门：<a href="elink@b32K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0k6h3@1J5x3o6p5&6i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1k6B7k6h3E0&6L8r3I4Q4x3V1k6#2M7r3c8S2N6r3g2Q4x3V1j5J5x3o6t1I4i4K6u0r3x3o6N6Q4x3V1j5I4z5g2)9J5c8V1N6o6g2p5j5J5x3o6t1I4i4K6u0V1k6f1u0b7c8W2)9J5k6h3S2@1L8h3H3`.">Google CTF 2021 eBPF</a>

首先本题由于是需要基于 qemu 下进行爆破，所以我基于 python 编写了一个爆破脚本。

<a href="elink@8fdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5i4y4@1k6g2)9J5k6i4g2T1N6h3&6@1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8Q4x3V1j5%4z5f1D9&6M7X3S2d9K9X3N6v1i4K6u0r3">script</a>

**<u>Somthing useful in `verifier.c`'s comments：</u>**

**eBPF 寄存器**

```c
/*
 * All registers are 64-bit.
 * R0 - return register
 * R1-R5 argument passing registers
 * R6-R9 callee saved registers
 * R10 - frame pointer read-only
 * At the start of BPF program the register R1 contains a pointer to bpf_context and has type PTR_TO_CTX.
 */
```

在 eBPF 程序的一开始 R1 类型为 `PTR_TO_CTX`。

**eBPF 寄存器类型**

Most of the time the registers have `SCALAR_VALUE` type, which means the register has some value, but it's not a valid pointer.（like pointer plus pointer becomes `SCALAR_VALUE` type）

大多数时间 `reg` 是标量类型而不是指针类型。

`PTR_TO_MAP_VALUE` means that this register is pointing to 'map element value'，and the **range of [`ptr`, `ptr + map's value_size`) is accessible**。

更多有用的 comments 的可以看：<mark class="encrypted">078K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5i4y4@1k6g2)9J5k6i4g2T1N6h3&6@1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8Q4x3V1k6V1d9s2m8v1M7U0y4$3M7o6b7$3i4K6u0r3</mark>

### 题目内容

#### `bpf_verifier.h.patch`

```c
--- linux-5.12.2/include/linux/bpf_verifier.h 2021-05-07 03:53:26.000000000 -0700
+++ linux-5.12.2-modified/include/linux/bpf_verifier.h 2021-06-15 20:06:53.019787853 -0700
@@ -156,6 +156,7 @@
 enum bpf_reg_liveness live;
 /* if (!precise && SCALAR_VALUE) min/max/tnum don't affect safety */
 bool precise;
+ bool auth_map;
 };
 enum bpf_stack_slot_type {
```

主要是在 `struct bpf_reg_state` eBPF 寄存器结构体中加了一个 `bool auth_map`。

<mark class="encrypted">069K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8r3W2^5K9i4u0Q4x3X3g2T1L8$3!0@1L8r3W2F1i4K6u0W2j5$3!0E0i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6$3y4g2)9J5k6e0p5J5i4K6u0W2x3W2)9J5c8Y4y4G2N6i4u0U0k6g2)9J5c8X3W2F1j5$3I4#2k6r3g2Q4x3V1k6D9K9h3&6#2P5q4)9J5c8X3u0H3k6W2)9#2k6Y4k6W2M7X3W2X3K9h3g2J5i4K6u0W2K9q4)9J5x3@1H3I4y4U0p5`.</mark>

#### `verifier.c.patch`

```c
--- linux-5.12.2/kernel/bpf/verifier.c 2021-05-07 03:53:26.000000000 -0700
+++ linux-5.12.2-modified/kernel/bpf/verifier.c 2021-06-15 20:06:54.495796355 -0700
@@ -2923,6 +2924,7 @@
 int off, int size, u32 mem_size,
 bool zero_size_allowed)
 {
+
 struct bpf_verifier_state *vstate = env->cur_state;
 struct bpf_func_state *state = vstate->frame[vstate->curframe];
 struct bpf_reg_state *reg = &state->regs[regno];
@@ -6326,13 +6330,19 @@
 memset(&dst_reg->raw, 0, sizeof(dst_reg->raw));
 }
 break;
- case BPF_AND:
- case BPF_OR:
 case BPF_XOR:
- /* bitwise ops on pointers are troublesome, prohibit. */
- verbose(env, "R%d bitwise operator %s on pointer prohibited\n",
- dst, bpf_alu_string[opcode >> 4]);
- return -EACCES;
+ // As long as we downgrade the result to scalar it is safe.
+ if (dst_reg->type == PTR_TO_MAP_VALUE) {
+     dst_reg->type = SCALAR_VALUE;
+     dst_reg->auth_map = true;
+     break;
+ }
+ case BPF_AND:
+ case BPF_OR:
+ /* bitwise ops on pointers are troublesome, prohibit. */
+ verbose(env, "R%d bitwise operator %s on pointer prohibited\n",
+ dst, bpf_alu_string[opcode >> 4]);
+ return -EACCES;
 default:
 /* other operators (e.g. MUL,LSH) produce non-pointer results */
 verbose(env, "R%d pointer arithmetic with %s operator prohibited\n",
@@ -7037,6 +7047,13 @@
 scalar_min_max_or(dst_reg, &src_reg);
 break;
 case BPF_XOR:
+ /* Restore the pointer type. */
+ if (dst_reg->auth_map) {
+     dst_reg->auth_map = false;
+     dst_reg->type = PTR_TO_MAP_VALUE;
+     break;
+ }
+
 dst_reg->var_off = tnum_xor(dst_reg->var_off, src_reg.var_off);
 scalar32_min_max_xor(dst_reg, &src_reg);
 scalar_min_max_xor(dst_reg, &src_reg);
```

在 `adjust_ptr_min_max_vals()` 里打了 patch。

<mark class="encrypted">9a5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8r3W2^5K9i4u0Q4x3X3g2T1L8$3!0@1L8r3W2F1i4K6u0W2j5$3!0E0i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6$3y4g2)9J5k6e0p5J5i4K6u0W2x3W2)9J5c8Y4y4G2N6i4u0U0k6g2)9J5c8X3E0W2M7X3&6W2L8q4)9J5c8X3u0H3k6W2)9J5c8Y4k6W2M7X3W2X3K9h3g2J5i4K6u0W2j5#2)9J5x3@1H3$3x3e0x3I4</mark>

```c
adjust_reg_min_max_vals()
-> adjust_ptr_min_max_vals()
```

被 patch 的函数主要用于处理：对于 **指针与标量** 的算术运算，并计算新的最大最小的 `var_off`。

在 **正常的** 函数中不允许指针与标量做位运算：

```c
case BPF_XOR:
/* bitwise ops on pointers are troublesome, prohibit. */
verbose(env, "R%d bitwise operator %s on pointer prohibited\n",
dst, bpf_alu_string[opcode >> 4]);
return -EACCES;
```

这里会直接 `return -EACCES;`。并且，在 `adjust_reg_min_max_vals()` 中还指出了：两个指针之间只能做减法运算。

#### patch流程

主要是在 eBPF 寄存器中加了一个 `auth_map` 变量，在处理 `adjust_ptr_min_max_vals()` 时：

`case BPF_XOR:`

- 如果目标 `dst_reg->type == PTR_TO_MAP_VALUE`，直接设置 `dst_reg->type = SCALAR_VALUE` 为标量，然后 `break`。
- 如果寄存器之前设置了 `auth_map`，那么置为 `false`，恢复 `dst_reg->type = PTR_TO_MAP_VALUE`。

#### 漏洞点

对于 XOR 来说，假如我们是：

```c
XOR dst_reg, src_reg
```
