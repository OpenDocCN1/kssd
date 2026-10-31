
```
#0  check_mem_access (env=0xffff888004b58000, insn_idx=0x1, regno=0xa, off=0x6, bpf_size=0x18, t=BPF_WRITE, value_regno=<error reading variable: Cannot access memory at address 0x0>, strict_alignment_once=<error reading variable: Cannot access memory at address 0x8>, is_ldsx=<error reading variable: Cannot access memory at address 0x10>) at kernel/bpf/verifier.c:6698
#1  0xffffffff812012a9 in do_check (env=<optimized out>) at kernel/bpf/verifier.c:17179
#2  do_check_common (env=0xffff888004b58000, subprog=0x0) at kernel/bpf/verifier.c:19643
#3  0xffffffff812064ba in do_check_main (env=<optimized out>) at kernel/bpf/verifier.c:19706
#4  bpf_check (prog=0xffff888004b58000, attr=0x1 <fixed_percpu_data+1>, uattr=..., uattr_size=0x18) at kernel/bpf/verifier.c:20333
#5  0xffffffff811df0c2 in bpf_prog_load (attr=0xffffc9000023fe58, uattr=..., uattr_size=0xfffffff0) at kernel/bpf/syscall.c:2743
#6  0xffffffff811e196a in __sys_bpf (cmd=0x5, uattr=..., size=0x0) at kernel/bpf/syscall.c:5465
#7  0xffffffff811e4059 in __do_sys_bpf (size=<optimized out>, uattr=<optimized out>, cmd=<optimized out>) at kernel/bpf/syscall.c:5569
#8  __se_sys_bpf (size=<optimized out>, uattr=<optimized out>, cmd=<optimized out>) at kernel/bpf/syscall.c:5567
#9  __x64_sys_bpf (regs=0xffff888004b58000) at kernel/bpf/syscall.c:5567
#10 0xffffffff81f38d39 in do_syscall_x64 (nr=<optimized out>, regs=<optimized out>) at arch/x86/entry/common.c:51
#11 do_syscall_64 (regs=0xffffc9000023ff58, nr=0x1) at arch/x86/entry/common.c:81
#12 0xffffffff82000134 in entry_SYSCALL_64 () at arch/x86/entry/entry_64.S:121
#13 0x0000000000000000 in ?? ()
```

这里利用了 eBPF 只在 load 的时候，对有内存操作的指令进行检查。这里有一个 eBPF 设计上的细节，就是它的只读权限不是真的只读不可写，而是对于 eBPF 字节码程序不可写，它是由自己的虚拟机进行内存检查，不是依靠操作系统，但是 eBPF 设计了辅助函数这个机制，可以在虚拟机中调用 C 代码，因此如果辅助函数的设计有缺陷，可以去写某些只读区域，而虚拟机字节码如果再次使用了这部分被修改的内存，虚拟机并不会对这个引用这个数据的寄存器进行检查，这是非常危险的。

由于 `bpf_aliyunctf_xor_proto` 辅助函数第三个参数有个标识位为 `MEM_RDONLY`，表示参数传入的地址可以是只读的。但是在函数实现中，这个内存地址是会被写入一个 64 位数据的。（刚开始我本来打算直接通过这个函数来修改内核的全局变量，发现不行诶，后来才知道，这些指针传递给辅助函数的时候也是有限制的，只能是 eBPF 内部的某些内存。）因此这里有一种利用方式是：我们可以利用这个设置来修改只读的 maps，只读权限区域可以帮助我们找到一种控制寄存器绕过内存边界检查的方式，我们可以使用 `bpf_skb_load_bytes()` 函数来破坏堆栈，覆盖函数 `ret` 地址，然后利用 rop 完成攻击。

官方有 [writeup](<https://bbs.kanxue.com/elink@805K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5P5W2)9J5k6h3q4D9K9i4W2#2L8W2)9J5k6h3y4G2L8g2)9J5c8X3&6W2N6%4y4Q4x3V1j5I4y4K6l9J5z5g2)9K6c8Y4c8A6L8h3g2Q4y4h3k6Q4y4h3j5I4x3K6p5I4i4K6y4p5L8U0c8Q4x3U0f1J5b7Y4S2n7c8r3y4p5x3p5c8#2y4@1E0s2d9@1N6Q4x3U0f1K6c8p5E0p5M7$3j5K6P5p5y4I4g2$3!0^5f1V1c8E0M7h3#2w2y4q4t1#2g2@1W2p5i4K6t1$3j5h3#2H3i4K6y4n7N6g2)9#2k6X3q4@1L8$3E0W2L8W2)9K6c8o6t1%4y4U0b7&6j5h3g2V1z5e0R3^5x3X3p5%4j5X3c8S2x3U0l9@1k6U0V1&6x3$3x3I4y4e0W2T1x3h3p5$3i4K6t1$3j5h3#2H3i4K6y4n7N6g2)9#2k6X3q4K6K9h3N6Q4x3@1b7#2y4r3b7^5y4b7>)。官方 writeup 的攻击流程就是通过向 eBPF 申请一个只读的 map，然后通过新添加的漏洞函数将原来的值改掉，然后把这个值取出来当 `bpf_skb_load_bytes()` 的第四个参数，因为数据是只读的，所以 eBPF 不会再去检查它的大小，这样就可以造成栈拷贝溢出，控制 `ret` 地址。这里还有一个细节就是刚刚进入虚拟机的时候寄存器 `R1` 被初始化为指向 `struct __sk_buff` 的指针。

但是用 `gcc exploit.c -o exp -static` 这种方式编出来的文件很大，当我写了个 [脚本](<https://bbs.kanxue.com/elink@efdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6k6h3q4U0K9r3!0Q4x3V1k6%4M7X3W2@1k6i4g2H3i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2K9h3&6Q4x3V1k6S2L8r3W2&6N6h3&6U0N6r3j5J5x3o6t1#2i4K6u0r3k6i4S2@1M7X3q4Q4x3X3g2H3P5b7>) 把数据提取出来后，通过 `echo -e "" > exp` 的方式粘贴进虚拟机的时候，不知道为啥我整个测试系统崩了。后来我自己写了一套 syscall 调用，来精简 `exp`。使用 `gcc exploit.c -o exp -nostdlib -static` 命令来编译，编出来的文件大小不足 1M。

[`exploit.c`](<https://bbs.kanxue.com/elink@091K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6k6h3q4U0K9r3!0Q4x3V1k6%4M7X3W2@1k6i4g2H3i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2K9h3&6Q4x3V1k6S2L8r3W2&6N6h3&6U0N6r3j5J5x3o6t1#2i4K6u0r3k6i4S2H3L8r3!0@1i4K6u0W2j5H3>)。

[\[招生\]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
