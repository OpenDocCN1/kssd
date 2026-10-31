
# [原创]ret2gets 的原理与利用方法

## 前言

`ret2gets` 是一种利用 glibc 优化特性（高版本编译器）的漏洞利用技术，核心是通过 `gets` 函数配合 `printf`/`puts` 实现 libc 地址泄露。该技术适用于：

1. 存在栈溢出漏洞
2. 程序包含 `gets` 函数
3. 缺乏直接控制 `rdi` 寄存器的 gadget（如 `pop rdi; ret`）

> 技术原型参考: <a href="elink@c81K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6j5i4y4Z5j5h3y4@1k6W2)9J5k6h3N6A6N6r3u0G2L8$3E0Q4x3X3g2A6L8#2)9J5c8Y4m8%4L8W2)9J5k6r3&6G2N6r3g2K6i4K6u0r3M7s2N6F1i4K6u0r3M7X3!0H3i4K6u0V1x3W2)9J5k6e0x3@1i4K6u0n7i4K6u0r3M7X3g2@1x3X3N6W2N6s2y4Q4x3U0y4K6K9h3c8W2L8X3!0@1k6g2)9J5k6r3!0F1i4K6u0V1k6X3W2F1k6r3W2F1k6#2)9J5k6r3I4G2j5$3E0A6L8X3N6Q4x3X3c8X3N6h3&6U0N6r3W2G2L8Y4x3`.">ret2gets | pwn-notes</a>
> 演示程序: `ret2gets_demo`: <mark class="encrypted">72dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4M7X3j5^5d9V1g2A6x3i4y4s2b7W2A6V1e0g2)9J5k6p5#2^5M7r3&6B7e0g2c8Y4i4K6y4r3M7s2N6V1i4K6y4p5P5r3W2V1M7l9`.`.</mark> 提取码: xidp

## 程序中 `pop rdi; ret` 的来源

```c
// gcc demo.c -o demo -no-pie -fno-stack-protector
#include <stdio.h>
int main() {
char buf[0x20];
puts("ROP me if you can!");
gets(buf);
}
```

我们正常会采用的方法很简单就是 `ret2libc`

我们会利用 `gets` 的溢出，使用程序里面的 gadget 来构造 `puts(func_got_addr)` 来泄露某一函数在 libc 中的地址，从而获取 libc 基地址，再控制程序返回，再来一次溢出使用 gadget 构造 `system('/bin/sh')` 以此来获得远程的 shell

但是对于这个程序编译之后我们会遇到一个问题，下面我们使用 `ROPgadget` 来查看一下我们可用的 gadget

```text
$ ROPgadget --binary demo
Gadgets information
============================================================
0x00000000004010ab : add bh, bh ; loopne 0x401115 ; nop ; ret
0x0000000000401037 : add byte ptr [rax], al ; add byte ptr [rax], al ; jmp 0x401020
0x000000000040115f : add byte ptr [rax], al ; add byte ptr [rax], al ; leave ; ret
0x0000000000401078 : add byte ptr [rax], al ; add byte ptr [rax], al ; nop dword ptr [rax] ; ret
0x0000000000401160 : add byte ptr [rax], al ; add cl, cl ; ret
0x000000000040111a : add byte ptr [rax], al ; add dword ptr [rbp - 0x3d], ebx ; nop ; ret
0x0000000000401039 : add byte ptr [rax], al ; jmp 0x401020
0x0000000000401161 : add byte ptr [rax], al ; leave ; ret
0x000000000040107a : add byte ptr [rax], al ; nop dword ptr [rax] ; ret
0x0000000000401034 : add byte ptr [rax], al ; push 0 ; jmp 0x401020
0x0000000000401044 : add byte ptr [rax], al ; push 1 ; jmp 0x401020
0x0000000000401009 : add byte ptr [rax], al ; test rax, rax ; je 0x401012 ; call rax
0x000000000040111b : add byte ptr [rcx], al ; pop rbp ; ret
0x0000000000401162 : add cl, cl ; ret
0x00000000004010aa : add dil, dil ; loopne 0x401115 ; nop ; ret
0x0000000000401047 : add dword ptr [rax], eax ; add byte ptr [rax], al ; jmp 0x401020
0x000000000040111c : add dword ptr [rbp - 0x3d], ebx ; nop ; ret
0x0000000000401117 : add eax, 0x2f03 ; add dword ptr [rbp - 0x3d], ebx ; nop ; ret
0x0000000000401118 : add ebp, dword ptr [rdi] ; add byte ptr [rax], al ; add dword ptr [rbp - 0x3d], ebx ; nop ; ret
0x0000000000401013 : add esp, 8 ; ret
0x0000000000401012 : add rsp, 8 ; ret
0x00000000004010a8 : and byte ptr [rax + 0x40], al ; add bh, bh ; loopne 0x401115 ; nop ; ret
0x0000000000401010 : call rax
0x0000000000401133 : cli ; jmp 0x4010c0
0x0000000000401130 : endbr64 ; jmp 0x4010c0
0x000000000040100e : je 0x401012 ; call rax
0x00000000004010a5 : je 0x4010b0 ; mov edi, 0x404020 ; jmp rax
0x00000000004010e7 : je 0x4010f0 ; mov edi, 0x404020 ; jmp rax
0x000000000040103b : jmp 0x401020
0x0000000000401134 : jmp 0x4010c0
0x00000000004010ac : jmp rax
0x0000000000401163 : leave ; ret
0x00000000004010ad : loopne 0x401115 ; nop ; ret
0x0000000000401116 : mov byte ptr [rip + 0x2f03], 1 ; pop rbp ; ret
0x000000000040115e : mov eax, 0 ; leave ; ret
0x00000000004010a7 : mov edi, 0x404020 ; jmp rax
0x00000000004010af : nop ; ret
0x000000000040112c : nop dword ptr [rax] ; endbr64 ; jmp 0x4010c0
0x000000000040107c : nop dword ptr [rax] ; ret
0x00000000004010a6 : or dword ptr [rdi + 0x404020], edi ; jmp rax
0x000000000040111d : pop rbp ; ret
0x0000000000401036 : push 0 ; jmp 0x401020
0x0000000000401046 : push 1 ; jmp 0x401020
0x0000000000401016 : ret
0x0000000000401042 : ret 0x2f
0x0000000000401022 : retf 0x2f
0x000000000040100d : sal byte ptr [rdx + rax - 1], 0xd0 ; add rsp, 8 ; ret
0x0000000000401169 : sub esp, 8 ; add rsp, 8 ; ret
0x0000000000401168 : sub rsp, 8 ; add rsp, 8 ; ret
0x000000000040100c : test eax, eax ; je 0x401012 ; call rax
0x00000000004010a3 : test eax, eax ; je 0x4010b0 ; mov edi, 0x404020 ; jmp rax
0x00000000004010e5 : test eax, eax ; je 0x4010f0 ; mov edi, 0x404020 ; jmp rax
0x000000000040100b : test rax, rax ; je 0x401012 ; call rax
Unique gadgets found: 53
```

在以往我们想要构造函数调用，不论是 `puts(func_got_addr)` 还是 `system('/bin/sh')`，我们首先需要的一点就是控制 `rdi` 寄存器

而我们往往是使用 `pop rdi; ret` 这个 gadget 来控制 `rdi` 寄存器的，但是显然，上面的程序是没有的

如果我们再仔细观察，其实会发现下面这三个我们常用的 gadget 都没有了

```text
pop rdi; ret
pop rsi; pop r15; ret
pop rbp; pop r12; pop r13; pop r14; ret
```

这是由于程序中的 `pop rdi; ret` 它大概的位置在 `__libc_csu_init+99`: `pop rdi`

也就是说它存在于 `__libc_csu_init` 这个函数中

```asm
__libc_csu_init 的汇编
0x0000000000400670 <+0>: push r15
0x0000000000400672 <+2>: push r14
0x0000000000400674 <+4>: mov r15d,edi
0x0000000000400677 <+7>: push r13
0x0000000000400679 <+9>: push r12
0x000000000040067b <+11>: lea r12,[rip+0x20078e] # 0x600e10
0x0000000000400682 <+18>: push rbp
0x0000000000400683 <+19>: lea rbp,[rip+0x20078e] # 0x600e18
0x000000000040068a <+26>: push rbx
0x000000000040068b <+27>: mov r14,rsi
0x000000000040068e <+30>: mov r13,rdx
0x0000000000400691 <+33>: sub rbp,r12
0x0000000000400694 <+36>: sub rsp,0x8
0x0000000000400698 <+40>: sar rbp,0x3
0x000000000040069c <+44>: call 0x4004b0 <_init>
0x00000000004006a1 <+49>: test rbp,rbp
0x00000000004006a4 <+52>: je 0x4006c6 <__libc_csu_init+86>
0x00000000004006a6 <+54>: xor ebx,ebx
0x00000000004006a8 <+56>: nop DWORD PTR [rax+rax*1+0x0]
0x00000000004006b0 <+64>: mov rdx,r13
0x00000000004006b3 <+67>: mov rsi,r14
0x00000000004006b6 <+70>: mov edi,r15d
0x00000000004006b9 <+73>: call QWORD PTR [r12+rbx*8]
0x00000000004006bd <+77>: add rbx,0x1
0x00000000004006c1 <+81>: cmp rbx,rbp
0x00000000004006c4 <+84>: jne 0x4006b0 <__libc_csu_init+64>
0x00000000004006c6 <+86>: add rsp,0x8
0x00000000004006ca <+90>: pop rbx
0x00000000004006cb <+91>: pop rbp
0x00000000004006cc <+92>: pop r12
0x00000000004006ce <+94>: pop r13
0x00000000004006d0 <+96>: pop r14
0x00000000004006d2 <+98>: pop r15
0x00000000004006d4 <+100>: ret
```

我们观察它的汇编代码其实不难发现，它其中并未含有 `pop rdi; ret` 这个 gadget

其实它来自于 `pop r15;` 的一部分

对比下面字节码我们就可以知道，`pop rdi; ret` 的字节码和 `pop r15; ret` 后半部分相同，所以把 `pop r15; ret` 截下来一半就是 `pop rdi; ret`

```text
pop r15 ; ret = 41 5f c3
pop rdi ; ret = 5f c3
```

而在 glibc 2.34 中 `pop rdi; ret` 消失的根本原因是<a href="elink@9ecK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4g2J5j5$3g2%4j5i4u0W2i4K6u0W2L8%4u0Y4i4K6u0r3M7r3W2H3k6i4u0E0j5h3W2D9i4K6u0r3L8r3W2T1j5#2)9J5k6r3q4D9M7r3S2S2i4K6u0r3x3U0l9J5x3g2)9J5k6p5k6W2j5Y4u0#2j5i4u0&6i4K6u0r3x3e0t1J5y4K6V1@1i4K6u0W2K9s2c8E0L8l9`.`.">一个补丁</a>移除了 `__libc_csu_init` 的二进制生成。

该补丁旨在删除 `ret2csu` 的有用 ROP 小工具，并具有删除针对 glibc 2.34+ 编译的二进制文件中的 `pop rdi; ret` 的效果。

这将导致一些问题，例如 `__libc_start_main`，它将 `__libc_csu_init` 作为参数。现在它不存在，它仍然接受参数，但对它没有任何作用，所以它在 2.34 中被版本优化了，因为它现在有不同的行为。这意味着我们不能够在较旧的 glibc 版本上运行为 2.34+ 编译的二进制文件，否则你会得到非常烦人的错误，如下

```text
/lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.34' not found
```

现在我们知道在程序中由于 `__libc_csu_init` 函数被优化了，所以我们已经没办法在编译后的程序中找到 `pop rdi; ret` 了

但是没有关系，`__libc_csu_init` 并非是 `pop rdi; ret` 的唯一来源，显然按照我们上面的解释，哪里有 `pop r15` 那么哪里就会有 `pop rdi`

虽然程序中没有 `pop rdi`

但是 glibc 自身含大量使用 `r15` 的函数，必存 `pop r15; ret`，泄露 libc 基址后，即可定位 libc 中的 `pop rdi; ret` 偏移

所以我们最大的问题还是需要溢出

## `ret2gets` 原理和利用方法

让我们来调试一下我们的 demo（glibc-2.35）

通过下图可以看到，在调用 `gets` 函数之前我们的 `rdi` 寄存器是指向了栈地址

![图片描述](upload/attach/202505/1005607_R2JK9M9C2RTB3NW.webp)

我们使用 `n` 步过 `call gets`，然后我们就会观察发现 `rdi` 寄存器变成了 `_IO_stdfile_0_lock`

也就是下图的 `*RDI 0x7ffff7e1ba80 (_IO_stdfile_0_lock) ◂— 0`

![图片描述](upload/attach/202505/1005607_E3GNE3ZC4777URM.webp)

而 `_IO_stdfile_0_lock` 其实是一个“锁”，用来锁住 `FILE`
