
`gdb-peda$ info registers`
```
rax 0x0 0x0
rbx 0x2 0x2
rcx 0x1 0x1
rdx 0x0 0x0
rsi 0xffff8800777bd2c0 0xffff8800777bd2c0
rdi 0x0 0x0
rbp 0xffffc90003ecfde0 0xffffc90003ecfde0
rsp 0xffffc90003ecfdc0 0xffffc90003ecfdc0
r8 0x0 0x0
r9 0x0 0x0
r10 0xffff880064c24db0 0xffff880064c24db0
r11 0xffff880074911110 0xffff880074911110
r12 0xffff8800777bd2c0 0xffff8800777bd2c0
r13 0xffff8800772a2138 0xffff8800772a2138
r14 0xffff8800772a2000 0xffff8800772a2000
r15 0x0 0x0
rip 0xffffffff8182f4a5 0xffffffff8182f4a5 <ip\_mc\_leave\_src+37>
eflags 0x10202 [ IF RF ]
cs 0x10 0x10
ss 0x18 0x18
ds 0x0 0x0
es 0x0 0x0
fs 0x0 0x0
gs 0xb 0xb
```
`gdb-peda$ x/50i ip\_mc\_leave\_src`
```
0xffffffff8182f480 <ip\_mc\_leave\_src>: nop DWORD PTR [rax+rax*1+0x0]
0xffffffff8182f485 <ip\_mc\_leave\_src+5>: push rbp
0xffffffff8182f486 <ip\_mc\_leave\_src+6>: mov rbp,rsp
0xffffffff8182f489 <ip\_mc\_leave\_src+9>: push r14
0xffffffff8182f48b <ip\_mc\_leave\_src+11>: push r13
0xffffffff8182f48d <ip\_mc\_leave\_src+13>: push r12
0xffffffff8182f48f <ip\_mc\_leave\_src+15>: push rbx
0xffffffff8182f490 <ip\_mc\_leave\_src+16>: mov r14,rdi
0xffffffff8182f493 <ip\_mc\_leave\_src+19>: mov rbx,QWORD PTR [rsi+0x18]
0xffffffff8182f497 <ip\_mc\_leave\_src+23>: mov r12,rsi
0xffffffff8182f49a <ip\_mc\_leave\_src+26>: mov rdi,rdx
0xffffffff8182f49d <ip\_mc\_leave\_src+29>: test rbx,rbx
0xffffffff8182f4a0 <ip\_mc\_leave\_src+32>: je 0xffffffff8182f4ef <ip\_mc\_leave\_src+111>
0xffffffff8182f4a2 <ip\_mc\_leave\_src+34>: mov edx,DWORD PTR [rsi+0x14]
=> 0xffffffff8182f4a5 <ip\_mc\_leave\_src+37>: mov ecx,DWORD PTR [rbx+0x4]
0xffffffff8182f4a8 <ip\_mc\_leave\_src+40>: lea rsi,[rsi+0x8]
0xffffffff8182f4ac <ip\_mc\_leave\_src+44>: lea r8,[rbx+0x18]
0xffffffff8182f4b0 <ip\_mc\_leave\_src+48>: xor r9d,r9d
0xffffffff8182f4b3 <ip\_mc\_leave\_src+51>: call 0xffffffff8182f110 <ip\_mc\_del\_src>
0xffffffff8182f4b8 <ip\_mc\_leave\_src+56>: mov QWORD PTR [r12+0x18],0x0
0xffffffff8182f4c1 <ip\_mc\_leave\_src+65>: mov r13d,eax
0xffffffff8182f4c4 <ip\_mc\_leave\_src+68>: mov eax,DWORD PTR [rbx]
0xffffffff8182f4c6 <ip\_mc\_leave\_src+70>: lea eax,[rax*4+0x18]
0xffffffff8182f4cd <ip\_mc\_leave\_src+77>: sub DWORD PTR ds:[r14+0x138],eax
0xffffffff8182f4d5 <ip\_mc\_leave\_src+85>: lea rdi,[rbx+0x8]
0xffffffff8182f4d9 <ip\_mc\_leave\_src+89>: mov esi,0x8
0xffffffff8182f4de <ip\_mc\_leave\_src+94>: call 0xffffffff810f4a40 <kfree\_call\_rcu>
0xffffffff8182f4e3 <ip\_mc\_leave\_src+99>: pop rbx
```

### 调用栈

`ip\_mc\_drop\_socket` -> `ip\_mc\_leave\_src`

```
gdb-peda$ bt
#0 0xffffffff8182f4a5 in ip\_mc\_leave\_src (sk=0xffff880077be8800, iml=0xffff88003d222540, in\_dev=0xffff8800)
at /build/linux-hwe-edge-gyUj63/linux-hwe-edge-4.10.0/net/ipv4/igmp.c:2155
#1 0xffffffff81832f18 in ip\_mc\_drop\_socket (sk=0xffff880077be8800)
at /build/linux-hwe-edge-gyUj63/linux-hwe-edge-4.10.0/net/ipv4/igmp.c:2607
#2 0xffffffff8182c2c0 in inet\_release (sock=0xffff880079749180)
at /build/linux-hwe-edge-gyUj63/linux-hwe-edge-4.10.0/net/ipv4/af\_inet.c:411
#3 0xffffffff8178b7bf in sock\_release (sock=0x0 <irq\_stack\_union>)
at /build/linux-hwe-edge-gyUj63/linux-hwe-edge-4.10.0/net/socket.c:599
#4 0xffffffff8178b832 in sock\_close (inode=<optimized out>, filp=<optimized out>)
at /build/linux-hwe-edge-gyUj63/linux-hwe-edge-4.10.0/net/socket.c:1063
#5 0xffffffff81246937 in \_\_fput (file=0xffff88003d171000)
at /build/linux-hwe-edge-gyUj63/linux-hwe-edge-4.10.0/fs/file\_table.c:209
#6 0xffffffff81246ade in \_\_\_\_fput (work=<optimized out>)
at /build/linux-hwe-edge-gyUj63/linux-hwe-edge-4.10.0/fs/file\_table.c:245
#7 0xffffffff810a706e in task\_work\_run () at /build/linux-hwe-edge-gyUj63/linux-hwe-edge-4.10.0/kernel/task\_work.c:116
#8 0xffffffff810032ba in tracehook\_notify\_resume (regs=<optimized out>)
at /build/linux-hwe-edge-gyUj63/linux-hwe-edge-4.10.0/include/linux/tracehook.h:191
#9 exit\_to\_usermode\_loop (regs=0xffffc900030c7f58, cached\_flags=0x2)
at /build/linux-hwe-edge-gyUj63/linux-hwe-edge-4.10.0/arch/x86/entry/common.c:160
#10 0xffffffff81003b29 in prepare\_exit\_to\_usermode (regs=<optimized out>)
at /build/linux-hwe-edge-gyUj63/linux-hwe-edge-4.10.0/arch/x86/entry/common.c:190
#11 syscall\_return\_slowpath (regs=0xffffc900030c7f58)
at /build/linux-hwe-edge-gyUj63/linux-hwe-edge-4.10.0/arch/x86/entry/common.c:259
#12 0xffffffff818ce948 in entry\_SYSCALL\_64 ()
at /build/linux-hwe-edge-gyUj63/linux-hwe-edge-4.10.0/arch/x86/entry/entry\_64.S:239
#13 0x0000000000000000 in ?? ()
```

最终根据oops信息发现kernel在`ip\_mc\_leave\_src`中出现了空指针引用的错误（位于第二次free的调用链中）。

在第一次mc\_list释放后，这块dirty内存的数据就可能会其它线程使用了。根据源码和反汇编代码分析可知，指针`psf`是`iml->sflist`处的值，即[`mc\_list+0x18`], 地址存在`rbx`中，`rbx`的值为`0x2`。从`mc\_list`中取出`psf`后，`ip\_mc\_leave\_src`中引用了`psf->sl\_count`导致了空指针引用。

`mov ecx,DWORD PTR [rbx+0x4] //arg4 psf->sl\_count`

### 源码

![Alt text](./upload/attach/201901/730610\_RGRS2UJBRTQJ798.png)

### 反汇编代码：

```
gdb-peda$ x/20i ip\_mc\_leave\_src
0xffffffff8182f480 <ip\_mc\_leave\_src>: nop DWORD PTR [rax+rax*1+0x0]
0xffffffff8182f485 <ip\_mc\_leave\_src+5>: push rbp
0xffffffff8182f486 <ip\_mc\_leave\_src+6>: mov rbp,rsp
0xffffffff8182f489 <ip\_mc\_leave\_src+9>: push r14
0xffffffff8182f48b <ip\_mc\_leave\_src+11>: push r13
0xffffffff8182f48d <ip\_mc\_leave\_src+13>: push r12
0xffffffff8182f48f <ip\_mc\_leave\_src+15>: push rbx
0xffffffff8182f490 <ip\_mc\_leave\_src+16>: mov r14,rdi
0xffffffff8182f493 <ip\_mc\_leave\_src+19>: mov rbx,QWORD PTR [rsi+0x18] //psf
0xffffffff8182f497 <ip\_mc\_leave\_src+23>: mov r12,rsi
0xffffffff8182f49a <ip\_mc\_leave\_src+26>: mov rdi,rdx //arg1 in\_dev
0xffffffff8182f49d <ip\_mc\_leave\_src+29>: test rbx,rbx //psf
0xffffffff8182f4a0 <ip\_mc\_leave\_src+32>: je 0xffffffff8182f4ef <ip\_mc\_leave\_src+111>
0xffffffff8182f4a2 <ip\_mc\_leave\_src+34>: mov edx,DWORD PTR [rsi+0x14] //arg3 iml->sfmode
=> 0xffffffff8182f4a5 <ip\_mc\_leave\_src+37>: mov ecx,DWORD PTR [rbx+0x4] //arg4 psf->sl\_count
0xffffffff8182f4a8 <ip\_mc\_leave\_src+40>: lea rsi,[rsi+0x8] //arg2 &iml->multi.imr\_multiaddr.s\_addr
0xffffffff8182f4ac <ip\_mc\_leave\_src+44>: lea r8,[rbx+0x18] //arg5 psf->sl\_addr
0xffffffff8182f4b0 <ip\_mc\_leave\_src+48>: xor r9d,r9d // arg6 0
0xffffffff8182f4b3 <ip\_mc\_leave\_src+51>: call 0xffffffff8182f110 <ip\_mc\_del\_src>
```

如果`psf`即[`mc\_list+0x18`]为`0`，那么就不会引用`psf->sl\_count`，内核就会进入`ip\_mc\_drop\_socket`的`kfree\_rcu()`流程，触发`kfree`的double free。

## REF

1. CVE-2017-8890漏洞分析与利用（Root Android 7.x）

<mark class="encrypted">962K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3k6J5k6h3g2T1N6h3k6Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7Y4c8A6j5$3I4W2M7#2)9J5c8Y4c8W2M7X3#2A6L8X3q4D9i4K6u0r3x3e0j5H3x3o6b7I4i4K6u0W2K9s2c8E0L8l9`.`.</mark>

2. “Phoenix Talon”in Linux Kernel —潜伏长达11年之久的内核漏洞

- <mark class="encrypted">f38K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4F1L8X3g2@1M7$3g2U0i4K6u0W2j5$3!0E0i4K6u0r3x3U0j5I4z5g2)9J5k6h3S2@1L8h3H3`.</mark>

- <mark class="encrypted">8bbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7q4)9J5k6i4N6W2K9i4S2A6L8W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8U0k6z5c8@1S2Q4x3X3c8p5K9K6u0F1i4K6g2X3b7X3E0V1L8p5Z5J5K9W2y4y4g2@1A6c8</mark>

- <mark class="encrypted">f43K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0R3$3x3U0M7#2</mark>

2. 【威胁通告】Linux 多个内核拒绝服务漏洞

<mark class="encrypted">10cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2F1M7$3k6G2j5%4g2K6i4K6u0W2L8X3g2@1i4K6u0r3L8r3W2F1N6i4S2Q4x3X3c8E0N6h3I4@1K9i4m8D9k6g2)9J5k6r3E0W2M7X3&6W2L8q4)9J5k6r3c8W2L8X3W2S2L8q4)9J5k6s2y4W2M7Y4k6A6j5$3g2Q4x3X3c8$3N6h3I4F1k6i4u0S2j5X3W2D9K9i4c8A6k6i4y4Q4x3V1j5`.</mark>

# 2018.08 CVE-2017-8890 漏洞分析 原理篇

## 目录

- 2018.08 CVE-2017-8890 漏洞分析 原理篇
- [正文开始](https://bbs.kanxue.com/#正文开始)
- [漏洞原理](https://bbs.kanxue.com/#漏洞原理)
- [POC](https://bbs.kanxue.com/#poc)
- [开始调试](https://bbs.kanxue.com/#开始调试)
- [动态调试](https://bbs.kanxue.com/#动态调试)
- [mc\_list的创建过程](https://bbs.kanxue.com/#mc_list的创建过程)
- mc\_list 的free过程
- double free之：sk和newsk中共有的mc\_list
- 定位crash点 ip\_mc\_leave\_src
