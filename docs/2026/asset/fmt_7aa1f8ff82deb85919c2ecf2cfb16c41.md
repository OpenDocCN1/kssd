
So far we have enough evidence to prove that `myfault.sys` was freeing invalid memory, but how to we know this memory is being freed twice? If there was a double free we need to determine if the first or second call to `ExFreePool` was incorrect. To this so we need to determine what code freed the memory first.

到此我们已经有了足够的证据来证实`myfault.sys`在释放一段非法的内存，但是要如何知道这段内存被释放了两次呢？如果是二次释放的话我们需要判断第一次或第二次对`ExFreePool`的调用是有误的。因此我们需要判断该内存第一次释放时的代码。

Driver Verifier special pool keeps track of the last 0x10000 calls to allocate and free pool. You can dump this database with the `!verifier 80` command. To limit the data output you can also pass this command the address of the memory you suspect was double freed.

Driver verifier特殊池保留了最后的0x10000个分配和释放池块调用的回溯。你可以通过`!verifier 80`命令将其转储成数据库。你也可以通过传递该疑似二次释放的内存地址作为参数来限制输出的数据。

Don’t assume the address in the bugcheck code is the address being freed, go get the address from the function that called `VerifierExFreePoolWithTag`.

不要假定bugcheck码就是被释放的地址，从`VerifierExFreePoolWithTag`的调用处获取地址。

In the above call stack the call below `VerifierExFreePoolWithTag` is frame 9 (start counting with 0, or use `kn`).

上面的栈回溯中，`VerifierExFreePoolWithTag`下面的调用是帧9（从0开始数，或者使用`kn`命令）。

```text
kd> .frame /r 9
09 fffff880`041a09d0 fffff880`04c9b727 myfault+0x15d9
rax=0000000000000000 rbx=fffff9800a7fe800 rcx=fffff9800a7fe800
rdx=fffffa8001a37fa0 rsi=fffffa80035975e0 rdi=fffffa8003597610
rip=fffff88004c9b5d9 rsp=fffff880041a09d0 rbp=fffffa80034568d0
r8=fffff9800a7fe801 r9=fffff9800a7fe7f0 r10=fffff9800a7fe800
r11=0000000000000000 r12=0000000000000000 r13=0000000000000000
r14=fffff800600306c7 r15=fffffa8004381b80
iopl=0 nv up ei ng nz na po nc
cs=0010 ss=0018 ds=002b es=002b fs=0053 gs=002b efl=00000286
myfault+0x15d9:
fffff880`04c9b5d9 eb7a jmp myfault+0x1655 (fffff880`04c9b655)
```

On x64 systems the first parameter is passed in `rcx`. The below assembly shows that `rcx` originated from `rbx`.

x64系统上第一个参数由`rcx`传递。下面的汇编显示了`rcx`从`rbx`获取。

```text
kd> ub fffff880`04c9b5d9
myfault+0x15ba:
fffff880`04c9b5ba ff15a80a0000 call qword ptr [myfault+0x2068 (fffff880`04c9c068)]
fffff880`04c9b5c0 33d2 xor edx,edx
fffff880`04c9b5c2 488bc8 mov rcx,rax
fffff880`04c9b5c5 488bd8 mov rbx,rax
fffff880`04c9b5c8 ff154a0a0000 call qword ptr [myfault+0x2018 (fffff880`04c9c018)]
fffff880`04c9b5ce 33d2 xor edx,edx
fffff880`04c9b5d0 488bcb mov rcx,rbx
fffff880`04c9b5d3 ff153f0a0000 call qword ptr [myfault+0x2018 (fffff880`04c9c018)]
```

Run `!verifier 80` using the address from `rbx`:

运行`!verifier 80`，使用`rbx`中的地址。

```text
kd> !verifier 80 fffff9800a7fe800
Log of recent kernel pool Allocate and Free operations:
There are up to 0x10000 entries in the log.
Parsing 0x0000000000010000 log entries, searching for address 0xfffff9800a7fe800.
======================================================================
Pool block fffff9800a7fe800, Size 0000000000000800, Thread fffffa80046ce4c0
fffff80060251a32 nt!VfFreePoolNotification+0x4a
fffff8005fe736c9 nt!ExFreePool+0x595
fffff80060242597 nt!VerifierExFreePoolWithTag+0x47
fffff88004c9b5ce myfault!MyfaultDeviceControl+0x2f2
fffff88004c9b727 myfault!MyfaultDispatch+0xb7
fffff80060241a4a nt!IovCallDriver+0xba
fffff800600306c7 nt!IopXxxControlFile+0x7e5
fffff800600458a6 nt!NtDeviceIoControlFile+0x56
fffff8005fc5a453 nt!KiSystemServiceCopyEnd+0x13
======================================================================
Pool block fffff9800a7fe800, Size 0000000000000800, Thread fffffa80046ce4c0
fffff80060242a5d nt!VeAllocatePoolWithTagPriority+0x2d1
fffff8006024b20e nt!XdvExAllocatePoolInternal+0x12
fffff80060242f69 nt!VerifierExAllocatePool+0x61
fffff88004c9b5c0 myfault!MyfaultDeviceControl+0x2e4
fffff88004c9b727 myfault!MyfaultDispatch+0xb7
fffff80060241a4a nt!IovCallDriver+0xba
fffff800600306c7 nt!IopXxxControlFile+0x7e5
fffff800600458a6 nt!NtDeviceIoControlFile+0x56
fffff8005fc5a453 nt!KiSystemServiceCopyEnd+0x13
```

The above output shows the pool block being allocated by `myfault.sys` and then freed by `myfault.sys`. If we combine this information with the call stack leading up to our bugcheck we can conclude that the pool was freed once in `MyfaultDeviceControl` at offset 0x2f2, then freed again in `MyfaultDeviceControl` at offset 0x2fd.

上面的输出显示`myfault.sys`分配了池块，后来又释放了池块。如果我们组合栈回溯和bugcheck的信息，就可以推断出池块被第一次释放是在`MyfaultDeviceControl`偏移0x2f2处，第二次释放是在`MyfaultDeviceControl`偏移0x2fd处。

Now we know which driver is causing the problem, and if this is our driver we know which area of the code to investigate.

现在我们知道了到底是哪个驱动引起的问题，如果这是我们的驱动我们就知道该去调查哪段代码了。

> msdn上关于池污染的官方说明。
>
> 原文如下:
>
> - Understanding Pool Corruption Part 1 (MSDN) - [here](https://bbs.kanxue.com/elink@b61K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4M7#2)9J5k6h3#2K6k6r3&6Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6F1N6r3c8W2j5Y4g2Y4k6$3W2F1k6#2)9J5c8U0t1H3x3e0y4Q4x3V1j5H3y4W2)9J5c8U0p5@1i4K6u0r3N6h3&6V1k6i4u0K6N6r3q4F1k6r3W2F1k6#2)9J5k6s2m8G2L8$3I4Q4x3X3c8U0L8%4u0J5N6i4m8@1K9h3!0F1i4K6u0V1M7r3q4J5N6q4)9J5k6o6q4Q4x3X3c8T1N6h3k6X3k6i4u0Q4x3X3c8G2N6X3g2J5k6X3I4G2N6%4y4Q4x3V1j5`.)
> - Understanding Pool Corruption Part 2 (MSDN) - [here](https://bbs.kanxue.com/elink@5fdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4M7#2)9J5k6h3#2K6k6r3&6Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6F1N6r3c8W2j5Y4g2Y4k6$3W2F1k6#2)9J5c8U0t1H3x3e0y4Q4x3V1j5H3z5q4)9J5c8U0t1J5i4K6u0r3N6h3&6V1k6i4u0K6N6r3q4F1k6r3W2F1k6#2)9J5k6s2m8G2L8$3I4Q4x3X3c8U0L8%4u0J5N6i4m8@1K9h3!0F1i4K6u0V1M7r3q4J5N6q4)9J5k6o6u0Q4x3X3c8K6M7r3g2U0K9h3q4D9i4K6u0V1M7r3!0G2L8q4)9J5k6r3k6G2M7W2)9J5k6r3u0#2k6X3k6W2M7W2)9J5k6r3!0$3k6i4u0J5N6h3&6K6i4K6u0r3)
> - Understanding Pool Corruption Part 3 (MSDN) - [here](https://bbs.kanxue.com/elink@adbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4M7#2)9J5k6h3#2K6k6r3&6Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6F1N6r3c8W2j5Y4g2Y4k6$3W2F1k6#2)9J5c8U0t1H3x3e0y4Q4x3V1j5I4x3W2)9J5c8U0x3I4i4K6u0r3N6h3&6V1k6i4u0K6N6r3q4F1k6r3W2F1k6#2)9J5k6s2m8G2L8$3I4Q4x3X3c8U0L8%4u0J5N6i4m8@1K9h3!0F1i4K6u0V1M7r3q4J5N6q4)9J5k6o6y4Q4x3X3c8K6M7r3g2U0K9h3q4D9i4K6u0V1M7r3!0G2L8q4)9J5k6r3k6G2M7W2)9J5k6r3c8G2N6h3u0D9k6g2)9J5k6r3k6J5k6h3g2K6i4K6u0r3)
>
> 很早的文章了（13年底），翻译这个主要是因为fuzzySecurity的Windows exp开发系列教程中内核篇的池溢出一文中有提到，同步翻译此文作为参考。
>
> 详细见：[Windows exploit开发系列教程第十六部分：内核利用程序之池溢出](https://bbs.pediy.com/thread-225182.htm)

# 理解池污染三部曲

## Understanding Pool Corruption Part 1 – Buffer Overflows

## 理解池污染第一部分：缓冲区溢出

Before we can discuss pool corruption we must understand what pool is. Pool is kernel mode memory used as a storage space for drivers. Pool is organized in a similar way to how you might use a notepad when taking notes from a lecture or a book. Some notes may be 1 line, others may be many lines. Many different notes are on the same page.

在我们讨论池污染之前必须先理解什么是池。池是内核模式下为驱动程序所需的存储空间而准备的内存。池的组织结构和我们日常使用记事本的方式很像，比如你从一本书或文献中摘取一段内容。某段内容可能只有一行，某段则可能有多行。多段不同的内容可以在同一页上。

Memory is also organized into pages, typically a page of memory is 4KB. The Windows memory manager breaks up this 4KB page into smaller blocks. One block may be as small as 8 bytes or possibly much larger. Each of these blocks exists side by side with other blocks.

内存也被组织成页，典型的页内存大小为4KB。Windows内存管理器把4K的页拆分成较小的块。每个块的大小至少是8字节，也可能更大一些。这些块彼此相互毗邻。

The `!pool` command can be used to see the pool blocks stored in a page.

`!pool`命令可以列出单个页上存储的池块。
