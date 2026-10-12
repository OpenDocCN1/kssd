
# Win7 下的段和门 (2)

> 原文：[`www.pediy.com/kssd/pediy12/143871.html`](https://www.pediy.com/kssd/pediy12/143871.html)

原贴最初发表在 QQ 空间：

[Win7 下的段和门 (2) ](http://user.qzone.qq.com/31731705/blog/1322121566)，[`user.qzone.qq.com/31731705/blog/1322121566`](http://user.qzone.qq.com/31731705/blog/1322121566)

代码和相关的分析在这：

[Win7 下的段和门 (2) （附录）](http://user.qzone.qq.com/31731705/blog/1322535584)，[`user.qzone.qq.com/31731705/blog/1322535584`](http://user.qzone.qq.com/31731705/blog/1322535584)

不久前写过一篇 [Win7 下的段和门](http://user.qzone.qq.com/31731705/blog/1313733365)（[`user.qzone.qq.com/31731705/blog/1313733365`](http://user.qzone.qq.com/31731705/blog/1313733365)），分析了 Win7 平台上保护模式中使用到的数据结构，主要就是 `GDT` 和 `IDT` 的数据分析。实际上，在 Win7 下，`GDT` 和 `IDT` 都和 CPU 有关系，我的电脑是双核处理器，测试结果如下，

```
0: kd> !sysinfo cpuinfo

[CPU Information]

~MHz = REG_DWORD 2527

Component Information = REG_BINARY 0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0

Configuration Data = REG_FULL_RESOURCE_DESCRIPTOR ff,ff,ff,ff,ff,ff,ff,ff,0,0,0,0,0,0,0,0

Identifier = REG_SZ x86 Family 6 Model 23 Stepping 10

ProcessorNameString = REG_SZ Intel(R) Core(TM)2 Duo CPU     P8700  @ 2.53GHz

Update Signature = REG_BINARY 0,0,0,0,7,a,0,0

Update Status = REG_DWORD 6

VendorIdentifier = REG_SZ GenuineIntel

MSR8B = REG_QWORD a0700000000

0: kd> r gdtr, gdtl, idtr, idtl

gdtr=80b95000 gdtl=000003ff idtr=80b95400 idtl=000007ff

1: kd> r gdtr, gdtl, idtr, idtl

gdtr=807d2c20 gdtl=000003ff idtr=807d3020 idtl=000007ff
```

可见，针对不同的 CPU，`IDT` 和 `GDT` 的地址是不一样的，那么，它们中的内容是否是一样的呢？因为 Win7 是对称式多处理系统，理论上即使使用了多个表，它们中的内容应该是相同的。为了研究这个问题，也顺便写了个脚本，测试一下我的想法。

```
0: kd> $$>a< ${sp}\\cmp_gdt.wds

This script is to compare 2 CPU information.

First argument (0)...

0x1 is to compare GDT.

0x2 is to compare IDT.

====================================================================================
```

这个脚本带 1 个参数，用来指出是比较哪张表，`GDT` 还是 `IDT`。

看下脚本运行的结果，给上参数 3，就是 1+2 的意思，同时比较 `GDT` 和 `IDT`。

```
0: kd> $$>a< ${sp}\\cmp_gdt.wds 3

GDT base address: 80b95000, 807d2c20, Number of items: 0n128, 0n128

Index Selector    Address                      1st value                      2nd value

0005 0028/0028 (80b95028, 807d2c48) (80008b1e, d00020ab) (80008b7c, d75020ab)

0006 0030/0030 (80b95030, 807d2c50) (84409396, 9c003748) (8040937c, a0003748)

0007 003B/003B (80b95038, 807d2c58) (7f40f3fd, e0000fff) (7f40f3fd, e0004000)

000A 0050/0050 (80b95050, 807d2c70) (84008996, 70000068) (8000897c, fac00068)

000B 0058/0058 (80b95058, 807d2c78) (84008996, 70680068) (8000897c, fb300068)

0014 00A0/00A0 (80b950a0, 807d2cc0) (870089bb, 91c00068) (870089ab, 8f980068)

There are total 0n6 items different.

IDT base address: 80b95400, 807d3020, Number of items: 0n256, 0n256

Index Address                       1st value                      2nd value

0052 (80b95690, 807d32b0) (8aa78e00, 000812d8) (8aa68e00, 0008c558)

0062 (80b95710, 807d3330) (8aa78e00, 000817d8) (8aa68e00, 0008ca58)

0071 (80b95788, 807d33a8) (8c6b8e00, 00087058) (8c6a8e00, 000892d8)

0072 (80b95790, 807d33b0) (887f8e00, 00088a58) (887f8e00, 00089a58)

0081 (80b95808, 807d3428) (8c6b8e00, 000872d8) (8c6a8e00, 00089558)

0082 (80b95810, 807d3430) (887f8e00, 00088058) (887f8e00, 00089058)

0092 (80b95890, 807d34b0) (8aa78e00, 00081cd8) (8aa68e00, 0008ccd8)

00A0 (80b95900, 807d3520) (8afb8e00, 0008fcd8) (8aa68e00, 0008c058)

00A2 (80b95910, 807d3530) (887f8e00, 00088558) (887f8e00, 00089558)

00B0 (80b95980, 807d35a0) (8aa78e00, 00081a58) (84878e00, 0008cbc0)

00B1 (80b95988, 807d35a8) (887f8e00, 00088cd8) (887f8e00, 00089cd8)

00B3 (80b95998, 807d35b8) (8aa78e00, 00081558) (8aa68e00, 0008c7d8)

00D1 (80b95a88, 807d36a8) (84808e00, 0008a634) (84808e00, 0008b2d8)

There are total 0n13 items different.

====================================================================================
```

乍一看不同项挺多的，实际上总共 `GDT` 有 128 项，`IDT` 有 256 项，不同的项占的比例并不多。那么，这些不同的项到底是些什么内容呢？先看 `GDT`，有 6 项，结合第一篇 [Win7 下的段和门](http://user.qzone.qq.com/31731705/blog/1313733365)（[`user.qzone.qq.com/31731705/blog/1313733365`](http://user.qzone.qq.com/31731705/blog/1313733365)）中描述的，`30`/`3B` 分别是内核态和用户态的 `FS` 段，它们确实是 CPU 相关的，因此不一样。`28` 呢？它是当前任务的 `TSS` 段，与当前的任务相关，也是 CPU 相关的。

```
1: kd> r @tr

tr=00000028
```

剩下的是什么？分析下就知道，实际上是三个 `TSS`，分别对应着 `IDT` 中的三个任务门，用来处理异常 `NMI (0x2)`, `#DF (0x8)`, `#MC (0x12)`

```
1: kd> !idt 2; !idt 8; !idt 12

Dumping IDT:

02: Task Selector = 0x0058

Dumping IDT:

08: Task Selector = 0x0050

Dumping IDT:

12: Task Selector = 0x00A0
```

关于 `TSS` 和 `FS` 段的作用，参见 [http://user.qzone.qq.com/31731705/blog/1304389170](http://user.qzone.qq.com/31731705/blog/1304389170)。

接下来继续分析 `IDT`，有 13 个，都是中断门。(关于门格式，参见 [INTEL-X86 保护模式下的内存管理](http://user.qzone.qq.com/31731705/blog/1307847866)，[`user.qzone.qq.com/31731705/blog/1307847866`](http://user.qzone.qq.com/31731705/blog/1307847866)))。先引用一段 Windows Internals (Fifth Edition)上的话，了解些基本概念。

> Each processor has a separate IDT so that different processors can run different ISRs, ifappropriate. For example, in a multiprocessor system, each processor receives the clock interrupt,but only one processor updates the system clock in response to this interrupt. All the processors, however, use the interrupt to measure thread quantum and to initiate rescheduling when a thread’s quantum ends. Similarly, some system configurations might require that a particular processor handle certain device interrupts.

在我的系统上，`D1` 是时钟中断，2 个处理器各自有不同的处理函数。

```
1: kd> ~0s; !idt d1; ~1s; !idt d1

Dumping IDT:

d1: 8480a634 hal!HalpHpetClockInterrupt

Dumping IDT:

d1: 8480b2d8 hal!HalpClockInterruptPn
```

`B0` 也很奇怪，

```
1: kd> ~0s;!idt b0;~1s;!idt b0;

Dumping IDT:

b0: 8aa71a58 ndis!ndisMiniportMessageIsr (KINTERRUPT 8aa71a00)

Dumping IDT:

b0: 8487cbc0 nt!KiUnexpectedInterrupt128
```

这是不是意味着这个中断只能由 0 号 CPU 来处理？困惑中。

其它的是设备中断，每个处理器分配和使用自己的中断对象，因此比较出来会有差异。以 `0x82` 为例，上面的输出如下，

`0082 (80b95810, 807d3430) (887f8e00, 00082058) (887f8e00, 00083058)`

```
1: kd> ~0s; !idt 82; ~1s; !idt 82

Dumping IDT:

82: 887f2058 ataport!IdePortInterrupt (KINTERRUPT 887f2000)

          ataport!IdePortInterrupt (KINTERRUPT 887f2780)

          sdbus!SdbusInterrupt (KINTERRUPT 8c5c2a00)

          rimmptsk+0x7674 (KINTERRUPT 8c5c2780)

Dumping IDT:

82: 887f3058 ataport!IdePortInterrupt (KINTERRUPT 887f3000)

          ataport!IdePortInterrupt (KINTERRUPT 887f3780)

          sdbus!SdbusInterrupt (KINTERRUPT 8c5c4c80)

          rimmptsk+0x7674 (KINTERRUPT 8c5c4a00)
```

中断对象不一样，但 `ISR` 是一样的，中断的处理流程是一样的。
