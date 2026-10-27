
而对于 `push` 和 `pop` 指令，我们这次除了从 Trace 中提取出寄存器下标，还提取出读写的寄存器值，方便我们分析伪代码。

VMProtect 2.13.8 相对之前的 Demo 版本，不但虚拟机内部混淆有明显增强，关键是伪代码级别变得更加难复杂了。（这其实才是 VMP 强度的关键）

其实伪代码也是可以利用前面介绍的符号执行方法进行分析，不过需要自行处理伪指令的语义（后面有空的话会专门写东西讲利用符号执行分析伪指令）

这里简单人肉分析一下：

```
// 这个代码块人肉看会比较痛苦
// 我也没有具体看
// 不过根据低版本的伪码应该是对 eflag 中的 trap 位进行检查的代码。
vPopReg4 R9 = 0x0
vPushImm4 0xc34f4e9
vAdd4
vPopReg4 R14 = 0x4046a7
vPopReg4 R4 = 0xc34f72f
vPopReg4 R5 = 0x404558
vPopReg4 R2 = 0x396622ad
vPopReg4 R6 = 0x401015
vPopReg4 R13 = 0x401015
vPopReg4 R7 = 0x307000
vPopReg4 R11 = 0x401015
vPopReg4 R10 = 0x401015
vPopReg4 R12 = 0x401015
vPopReg4 R14 = 0x19ff94
vPopReg4 R3 = 0x6b1da363
vPopReg4 R3 = 0x95a00657
vPushReg4 R5 = 0x404558
vPushImm4 0x2d47169f
vPushImm4 0x2d47150d
vPushImmSx2 0xfffffeff
vPushReg4 R5 = 0x404558
vPushVESP
vReadMemSs4
vNor4
vPopReg4 R8 = 0x203a21fc
vNor4
vPopReg4 R15 = 0x203a21fc
vPopReg4 R1 = 0x100
vPushVESP
vPushImm1 0x4
vPushReg4 R15 = 0x203a21fc
vPushVESP
vReadMemSs4
vNor4
vPopReg4 R8 = 0x203a21fc
vPushImmSx4 0xffffffbf
vNor4
vPopReg4 R5 = 0x203a21fc
vShr4
vPopReg4 R3 = 0x202
vAdd4
vPopReg4 R1 = 0x4046a7
vReadMemSs4
vPopReg4 R5 = 0x2d47169f
vPopReg4 R8 = 0x2d47150d
vPopReg4 R8 = 0x2d47169f
vPushReg4 R5 = 0x2d47169f
vPopReg4 R1 = 0x2d47169f
vPushReg4 R1 = 0x2d47169f
vPushReg4 R1 = 0x2d47169f
vNor4
vPopReg4 R3 = 0x203a21fc
vPushImm4 0xd2f8b4b8
vNor4
vPopReg4 R5 = 0x203a21fc
vPushImm4 0x2d074b47
vPushReg4 R1 = 0x2d47169f
vNor4
vPopReg4 R3 = 0x203a21fc
vNor4
vPopReg4 R3 = 0x203a21fc
vPopReg4 R3 = 0x405dd8
vPushReg4 R10 = 0x401015
vPushReg4 R7 = 0x307000
vPushReg4 R13 = 0x401015
vPushReg4 R10 = 0x401015
vPushReg4 R14 = 0x19ff94
vPushReg4 R5 = 0x203a21fc
vPushReg4 R11 = 0x401015
vPushReg4 R12 = 0x401015
vPushReg4 R2 = 0x396622ad
vPushReg4 R15 = 0x203a21fc
vPushReg4 R7 = 0x307000
vPushReg4 R4 = 0xc34f72f
vPushImm4 0xf3cb0b17
vAdd4
vPopReg4 R5 = 0x4046a7
vPushReg4 R9 = 0x0
vPushReg4 R3 = 0x405dd8 ; 跳转目标
vJmp
// 初始化
vPopReg4 R14 = 0x0
vPushImm4 0xc34f4e9
vAdd4
vPopReg4 R10 = 0x4046a7
vPopReg4 R3 = 0xc34f72f
vPopReg4 R7 = 0x307000
vPopReg4 R8 = 0x203a21fc
vPopReg4 R12 = 0x396622ad
vPopReg4 R2 = 0x401015
vPopReg4 R9 = 0x401015
vPopReg4 R10 = 0x203a21fc
vPushReg4 R10 = 0x203a21fc
vPushReg4 R10 = 0x203a21fc
vNor4
vPopReg4 R6 = 0x203a21fc
vPushImm4 0xd2f8b4b8
vNor4
vPopReg4 R1 = 0x203a21fc
vPushImm4 0x2d074b47
vPushReg4 R10 = 0x203a21fc
vNor4
vPopReg4 R5 = 0x203a21fc
vNor4
vPopReg4 R6 = 0x203a21fc
vPopReg4 R6 = 0xd3d6abb
vPopReg4 R0 = 0x19ff94
vPopReg4 R15 = 0x401015
vPopReg4 R1 = 0x401015
vPopReg4 R13 = 0x307000
vPopReg4 R11 = 0x401015
vPopReg4 R5 = 0x404558
vPushReg4 R5 = 0x404558
vPushReg4 R5 = 0x404558
vNor4
vPopReg4 R8 = 0x203a21fc
vPushImmSx2 0x8ff
vNor4
// mov eax, dword_403000
vPushImm4 0x403000
vReadMem4
vPopReg4 R8 = 0xdeadbeef
vPopReg4 R12 = 0x203a21fc
// add eax, 12345678h
vPushImm4 0x12345678
vPushReg4 R8 = 0xdeadbeef
vAdd4
vPopReg4 R4 = 0x4046a7
vPopReg4 R11 = 0xf0e21567
vPopNULL4
vPushReg4 R0 = 0x19ff94
vPushReg4 R2 = 0x401015
// sub eax, 12345678h
vPushImm4 0x12345678
vPushReg4 R11 = 0xf0e21567
vPushVESP
vReadMemSs4
vNor4
vPopReg4 R8 = 0x203a21fc
vAdd4
vPopReg4 R4 = 0x4046a7
vPushVESP
vReadMemSs4
vNor4
vPopReg4 R12 = 0x203a21fc
vPopReg4 R13 = 0xdeadbeef ; result
// flag
vPushReg4 R4 = 0x4046a7
vPushVESP
vReadMemSs4
vNor4
// mov dword_403000, eax
vPushReg4 R13 = 0xdeadbeef
vPushImm4 0x403000
vWriteMem4
// flag
vPopReg4 R11 = 0x203a21fc
vPushImmSx2 0xfffff7ea
vNor4
vPopReg4 R5 = 0x203a21fc
vPushReg4 R12 = 0x203a21fc
vPushReg4 R12 = 0x203a21fc
vNor4
vPopReg4 R6 = 0x203a21fc
vPushImmSx2 0x815
vNor4
vPopReg4 R11 = 0x203a21fc
vAdd4
vPopReg4 R8 = 0x4046a7
vPopReg4 R8 = 0x203a21ed
// ret
vPushReg4 R15 = 0x401015
vPushReg4 R9 = 0x401015
vPushReg4 R7 = 0x307000
vPushReg4 R1 = 0x401015
vPushReg4 R12 = 0x203a21fc
vPushReg4 R13 = 0xdeadbeef
vPushReg4 R8 = 0x203a21ed
vPushReg4 R15 = 0x401015
vPushReg4 R13 = 0xdeadbeef
vRet
```

# 总结

本篇介绍了利用 Trace 和符号执行分析 VMProtect 2.13.8 的方法。其实只是大概提下思路，想要完全自动化分析还有很长的路要走。

Trace 分析这种方法对混淆代码其实还是不错的，动态运行信息有了，比静态看 IDA 要好很多。但这种方法也是有局限的，首先 Trace 文件可能会非常大，处理起来很麻烦。另一方面 Trace 只记录跑过的指令，没有跑过的代码 Trace 是分析不到的。就像文中所述，这个 VMP 样本使用 Trace 大概只能提取到 20 多个 Handler，因为样本实际只用到了这么多。

符号执行是个新东西。对于学术界可能不新了，但是开始出现好用的实用工具如 `angr`, `miasm`, `triton` 等也是近年的事。这种新方法如果应用到虚拟机保护上也许会有奇效。

（文章涉及的样本、IDB 文件、Trace 文件、部分脚本见附件，密码123456）

Moon

2018/04/08

# 序言

系列[第1篇](https://bbs.pediy.com/thread-225262.htm)对一个极弱的虚拟机 VMProtect 1.81 Demo 版进行了分析，初步展示了一下对虚拟机保护代码的分析方法。

Demo 版因为代码没有混淆处理，因此在 IDA 中可以分析的很清楚了，甚至还可以根据字节码（`VM_DATA`）一点点静态还原虚拟指令。

然而正式版本的 VMProtect 虚拟机是有比较严重的混淆的（实际是一种冗余指令的添加），直接使用 IDA 分析十分困难，许多基本块会截断，也没有第1篇中介绍的明显的解释循环图结构。动态调试也不方便，里面大量的 `CALL` 和 `JMP`，跳来跳去。`ESI` 还有指令的立即数等还有加密，整体复杂度有很大的提升。

对于这种情况如何处理呢？本文以 VMProtect 2.13.8 为例，展示如何在混淆比较严重的情况下找到虚拟机关键结构、快速的分析 Handler，提取出虚拟指令。

本文主要介绍两个关键的部分：

- 通过 Trace 分析快速把握虚拟机结构
- 通过符号执行快速分析 Handler

# VMProtect 2.13.8 样本说明

这里使用的样本仍是第1篇中的代码加 VM 的。

```
sub_401000 proc near

mov     eax, dword_403000

add     eax, 12345678h

sub     eax, 12345678h

mov     dword_403000, eax

retn

sub_401000 endp
```

加密级别为**最大速度**，目前我们只讨论虚拟机本身，其他诸如 IAT 保护、反调试等其他 VMP 保护选项都关闭，让我们专注于虚拟机的分析。

其实即使到了 2.13.8 版本， VMProcect 的整体结构仍是和 VMProtect 1.81 一致的。 因此虽然存在混淆，经验丰富的人还是可以迅速找到关键的指令分发点（Dispatcher）位置，从而找到跳转表，提取所有的 Handler。

如下指令（加*的部分）就是典型的 dispatcher 代码，根据 `0x404cf8` 跳转表，再找到 `ESI` 解密的方式，就可以提取所有 Handler。

```
0x4041ff        mov edx, dword ptr [eax*4+0x404cf8]  ;*****

0x404206        cmc

0x404207        sub edx, 0x1aadee33

0x40420d        mov byte ptr [esp+0x8], 0x28

0x404212        pushad

0x404213        mov dword ptr [esp+0x40], edx        ; ******

0x404217        pushfd

0x404218        mov byte ptr [esp+0x4], ch

0x40421c        push dword ptr [esp+0x8]

0x404220        push dword ptr [esp+0x48]

0x404224        ret 0x4c                             ; ******
```

这是前人总结出来的特征。如果是不了解 VMProtect 的人，分析起来就要困难一些。

如果我们对 VMProtect 并不十分清楚，直接面对 VMProtect 2.13.8 版本的样本该如何处理？

# 虚拟机结构分析

首先仍 IDA 打开样本，跳到 `0x401000` 处，看下前几条指令：

```
.text:00401000                 jmp     loc_405DD8  ; 跳转到 .vmp0 中的虚拟机代码
.vmp0:00405DD8                 pushf
.vmp0:00405DD9                 call    sub_405CF7  ; call并不是函数调用，只是跳转进入虚拟机中
.vmp0:00405CF7                 mov     byte ptr [esp+0], 0A4h
.vmp0:00405CFB                 mov     dword ptr [esp+4], 95A00657h
.vmp0:00405D03                 mov     [esp+0], dl
.vmp0:00405D06                 mov     dword ptr [esp+0], 6B1DA363h
.vmp0:00405D0D                 pusha
.vmp0:00405D0E                 push    dword ptr [esp+8]
.vmp0:00405D12                 pusha
.vmp0:00405D13                 mov     [esp+20], al
.vmp0:00405D17                 lea     esp, [esp+44h] ; 前面压栈的内容弹出来
.vmp0:00405D1B                 jmp     loc_4048F4
.vmp0:004048F4                 jmp     loc_40593D
...
```

其实这些代码并没有什么有用的操作，先向栈中压入一些无用的数据，然后又通过 `lea esp, [esp+44h]` 把栈顶降回来，相当于把压入的数据弹出来，结果就是什么都没有做。同时使用了大量的 `jmp` 和 `call` 将代码切割成若干小块，使 IDA 无法正常分析。

我们还是很喜欢第1篇中的 IDA 生成的控制流图的。通过图中明显的解释循环结构，可以很快的定位哪里是 Dispatcher，哪里是 Handler。但现在 `jmp` 和 `call` 干扰了 IDA 的分析，IDA 没法生成完整的 CFG 图。

为了恢复这个图，我们还是使用 Trace 分析的方法。Trace 是 x86 指令执行的序列，Trace 中所有 `jmp` 和 `call` 会天然的和跳转目标指令连接起来。Trace 分析其实是很强大的功能。单步执行时我们的注意力可能会被寄存器、内存的值所分散，而忽略了程序整体的执行情况。而通过对 Trace 的整体分析，则可以让我们跳出局部，从整体去观察。

虚拟机解释执行的过程是一个循环：首先取指令，解码，然后跳转到 Handler 代码，执行完成后再跳转回来。在 Trace 中我们如何捕捉这个关键的循环？我想到的方法是，由 Trace 构造一个像之前 IDA 显示的 CFG 图类似的图。 通过找图中的中心结点，来确定 dispatcher 的位置。

由 Trace 构造的图，是反映了程序执行过程的基本块图，我们可以称之为**执行流图**。构造图的方法说起来比较费力，直接看图。

![graph](https://bbs.kanxue.com/./upload/attach/201804/657023_VTPTQRT8FKXHYGF.png)

假设ABCD是执行的指令序列，按照每条指令在 Trace 中的先后顺序，就可以构造一个图。相邻结点合并一下，就可以得到最终的执行流图。最终 AB 形成一个块，执行到 CDE 块，再跳转回来，再执行 FG 块，再跳转回来，再执行 HI 块，再跳转回来。整个执行的过程就很清楚了。通过写脚本分析 OD 的 Trace 文件，就可以构造出这个图，编程有基础的同学实现一下并不困难。

我做过一个工具，具备生成这种图的功能。不过为了 Trace 的效率，这个工具没有使用 OD Trace，而是利用 Intel 的插装工具 `Pin` 记录指令序列。（这个工具还在写，目前比较渣，不多介绍了。如果写好了会单独写文章介绍。想尝鲜可以自己下载下来编译源码，传送门在 811K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6D9L8i4V1K6y4K6g2Q4x3V1k6H3K9h3&6$3L8i4l9`. ）

下面是用这个工具分析 VMP 样本生成的执行流图。

![img](https://bbs.kanxue.com/./upload/attach/201804/657023_JV6SR9F5P2WUNMG.png)
