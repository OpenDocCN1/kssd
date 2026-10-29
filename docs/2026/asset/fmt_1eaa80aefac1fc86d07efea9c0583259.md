
# 深入解析池污染（第一部分）：检测与初始分析

```
kd> .cxr 0xfffff88004763560
rax=4f4f4f4f4f4f4f4f rbx=fffff80092690460 rcx=fffff800926fbc60
rdx=0000000000000000 rsi=0000000000001000 rdi=0000000000000000
rip=fffff8009267244a rsp=fffff88004763f60 rbp=fffff8009268fb40
r8=fffffa8001a1b820 r9=0000000000000001 r10=fffff800926fbc60
r11=0000000000000011 r12=0000000000000000 r13=fffff8009268fb48
r14=0000000000000012 r15=000000006374504d
iopl=0 nv up ei pl nz na po nc
cs=0010 ss=0018 ds=002b es=002b fs=0053 gs=002b efl=00010206
nt!ExAllocatePoolWithTag+0x442:
fffff800`9267244a 4c8b4808 mov r9,qword ptr [rax+8] ds:002b:4f4f4f4f`4f4f4f4f57=????????????????
```

从上述输出可以看出，崩溃发生在`ExAllocatePoolWithTag`中，这很好地表明崩溃与池污染有关。通常，工程师分析转储文件时会在此处停下，并断定崩溃由污染引起，但我们可以进一步深入。

## 指令分析与根因初探

导致失败的指令是对`rax+8`的解引用。`rax`寄存器中的值为`0x4f4f4f4f4f4f4f4f`，这与x64系统上指针所需的规范形式不符。这说明系统崩溃的原因是`rax`中的数据本应是一个指针，但实际并非如此。

要确定`rax`为何未包含预期数据，我们必须检查故障发生前的指令。

```
kd> ub .
nt!KzAcquireQueuedSpinLock [inlined in nt!ExAllocatePoolWithTag+0x421]:
fffff800`92672429 488d542440 lea rdx,[rsp+40h]
fffff800`9267242e 49875500 xchg rdx,qword ptr [r13]
fffff800`92672432 4885d2 test rdx,rdx
fffff800`92672435 0f85c3030000 jne nt!ExAllocatePoolWithTag+0x7ec (fffff800`926727fe)
fffff800`9267243b 48391b cmp qword ptr [rbx],rbx
fffff800`9267243e 0f8464060000 je nt!ExAllocatePoolWithTag+0xa94 (fffff800`92672aa8)
fffff800`92672444 4c8b03 mov r8,qword ptr [rbx]
fffff800`92672447 498b00 mov rax,qword ptr [r8]
```

汇编代码显示，`rax`源自`r8`指向的数据。此前运行的`.cxr`命令显示`r8`的值为`fffffa8001a1b820`。检查该地址处的数据时，发现它与`rax`的内容一致，从而确认该内存是`rax`中意外数据的来源。

```
kd> dq fffffa8001a1b820 l1
fffffa80`01a1b820 4f4f4f4f`4f4f4f4f
```

为确定该意外数据是否由池污染引起，可使用`!pool`命令。

```
kd> !pool fffffa8001a1b820
Pool page fffffa8001a1b820 region is Nonpaged pool
fffffa8001a1b000 size: 810 previous size: 0 (Allocated) None
fffffa8001a1b810 doesn't look like a valid small pool allocation, checking to see
if the entire page is actually part of a large page allocation...
fffffa8001a1b810 is not a valid large pool allocation, checking large session pool...
fffffa8001a1b810 is freed (or corrupt) pool
Bad previous allocation size @fffffa8001a1b810, last size was 81
\*\*\*
\*\*\* An error (or corruption) in the pool was detected;
\*\*\* Attempting to diagnose the problem.
\*\*\*
\*\*\* Use !poolval fffffa8001a1b000 for more details.
Pool page [ fffffa8001a1b000 ] is \_\_INVALID.
Analyzing linked list...
[ fffffa8001a1b000 --> fffffa8001a1b010 (size = 0x10 bytes)]: Corrupt region
Scanning for single bit errors...
None found
```

上述输出与之前的`!pool`结果不同。它展示了池头的污染，这阻止了命令遍历分配链。进一步查看`fffffa8001a1b000`处的大小为`810`的分配，其头部本应是一个池头，实际却被`4f4f4f4f\`4f4f4f4f`模式覆盖。

```
kd> dq fffffa8001a1b000 + 810
fffffa80`01a1b810 4f4f4f4f`4f4f4f4f 4f4f4f4f`4f4f4f4f
fffffa80`01a1b820 4f4f4f4f`4f4f4f4f 4f4f4f4f`4f4f4f4f
fffffa80`01a1b830 4f4f4f4f`4f4f4f4f 00574f4c`46524556
fffffa80`01a1b840 00000000`00000000 00000000`00000000
fffffa80`01a1b850 00000000`00000000 00000000`00000000
fffffa80`01a1b860 00000000`00000000 00000000`00000000
fffffa80`01a1b870 00000000`00000000 00000000`00000000
fffffa80`01a1b880 00000000`00000000 00000000`00000000
```

至此，我们可以确认系统崩溃的原因是池污染。

由于污染发生在过去，而转储文件只是当前系统状态的快照，没有具体证据能指出内存是如何被污染的。可能是某个驱动在分配池块前写入了错误位置，从而导致污染。该池块的标签为"None"，我们可以在内存中搜索该标签，以确定哪个驱动使用了它。

```
kd> !for\_each\_module s -a @#Base @#End "None"
fffff800`92411bc2 4e 6f 6e 65 e9 45 04 26-00 90 90 90 90 90 90 90 None.E.&........
kd> u fffff800`92411bc2-1
nt!ExAllocatePool+0x1:
fffff800`92411bc1 b84e6f6e65 mov eax,656E6F4Eh
fffff800`92411bc6 e945042600 jmp nt!ExAllocatePoolWithTag (fffff800`92672010)
fffff800`92411bcb 90 nop
```

文件`Pooltag.txt`列出了Windows支持的内核模式组件和驱动所使用的池分配标签、关联的文件或组件（如果已知）以及组件的名称。`Pooltag.txt`内置在Windows调试工具中（位于`triage`目录），也在Windows WDK中（位于`\tools\other\*platform*\poolmon`）。`Pooltag.txt`显示该标签的信息为：

`None` - `<unknown>` - 调用`ExAllocatePool`

不幸的是，此标签用于驱动调用`ExAllocatePool`且未指定标签的情况，这使得我们无法确定是哪个驱动分配了污染前的块。即使能将标签与驱动关联，也未必足以断定使用该标签的驱动就是污染内存的元凶。

下一步应启用特殊池（Special Pool），以期能当场捕获污染源。我们将在下一篇文章中讨论特殊池。

# 理解池污染（第二部分）：特殊池下的缓冲区溢出

在[上一篇文章](elink@ba5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6K6i4K6u0W2L8i4y4V1L8W2)9J5k6h3y4G2L8g2)9J5c8X3u0Q4x3V1k6F1N6r3c8W2j5Y4g2Y4k6$3W2F1k6#2)9J5c8X3q4J5j5$3S2A6N6X3g2Q4x3V1j5J5x3o6p5K6i4K6u0r3x3o6k6Q4x3V1j5I4y4q4)9J5c8Y4g2F1k6r3g2J5M7%4c8S2L8X3c8A6L8X3N6Q4x3X3c8H3L8$3!0D9i4K6u0V1j5$3!0J5M7Y4g2H3N6r3W2G2L8W2)9J5k6s2m8S2M7Y4c8Q4x3X3b7I4i4K6u0V1j5Y4g2X3k6X3g2J5i4K6u0V1L8%4k6W2M7X3k6D9L8%4N6K6i4K6u0W2j5i4y4H3P5l9`.`.")中，我们讨论了驱动在缓冲区中写入过多数据时引发的池污染。在本文中，我们将讨论特殊池如何帮助识别写出越界的驱动程序。
