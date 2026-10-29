
因为特殊池把自身的每个池分配都存储在自己的4K页上，它会引起内存使用上的增加。当特殊池被启用时，内存管理器会配置一个系统上可被分配的特殊池的上限，当上限值达到时，就会使用普通的池取而代之。这一限制尤其在32位系统上声明，它比64位系统的内核空间小太多了。

Now that we have explained how special pool works, we should use it.

现在我们解释过了特殊池是如何工作的，我们现在使用它。

There are two methods to enable special pool. `Driver verifier` allows special pool to be enabled on specific drivers. The `PoolTag` registry value described in [KB188831](https://bbs.kanxue.com/elink@d3eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4#2M7s2m8G2M7Y4c8Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6C8j5W2)9J5c8U0p5^5z5o6R3K6x3b7`.`.) allows special pool to be enabled for a particular pool tag. Starting in Windows Vista and Windows Server 2008, `driver verifier` captures additional information for special pool allocations so this is typically the recommended method.

有两种方法来启用特殊池。`Driver verifier`可以在特定驱动上启用特殊池。在 [KB188831](https://bbs.kanxue.com/elink@459K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4#2M7s2m8G2M7Y4c8Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6C8j5W2)9J5c8U0p5^5z5o6R3K6x3b7`.`.)中描述的`PoolTag`注册的值允许特殊池以一个特定的池标签来启用。从Windows Vista和Server 2008开始，`driver verifier`还会捕获特殊池分配额外的信息，所以这种方法极为推荐。

To enable special pool using `driver verifier` use the following command line, or choose the option from the `verifier` GUI. Use the `/driver` flag to specify drivers you want to verify, this is the place to list drivers you suspect as the cause of the problem. You may want to verify drivers you have written and want to test or drivers you have recently updated on the system. In the command line below I am only verifying `myfault.sys`. A reboot is required to enable special pool.

使用`driver verifier`启用特殊池要用下面的命令行，或者用`verifier` GUI配置选项。使用`/driver`来指定驱动，它用于列举你怀疑可能引起问题的驱动程序。你可能想要核查你写过的驱动或者系统上最近更新过的驱动。在下面的命令行中我仅仅核查`myfault.sys`。激活特殊池需要重启。

```
verifier /flags 1 /driver myfault.sys
```

After enabling `verifier` and rebooting the system, repeat the activity that causes the crash. For some problems the activity may just be to wait for a period of time. For our demonstration we are running `NotMyFault` (see Part 1 for details).

激活并重启后，重复上文中引起崩溃的操作。由于一些原因，我们需要等待一段时间。为了演示我们运行`NotMyFault`(更多细节参考第一部分)。

The crash resulting from a buffer overflow in special pool will be a stop `0xD6`, `DRIVER_PAGE_FAULT_BEYOND_END_OF_ALLOCATION`.

在特殊池中由于缓冲区溢出而崩溃，停止码为`0xD6` `DRIVER_PAGE_FAULT_BEYOND_END_OF_ALLOCATION`。

```
kd> !analyze -v

*******************************************************************************
*                                                                             *
*                        Bugcheck Analysis                                    *
*                                                                             *
*******************************************************************************

DRIVER_PAGE_FAULT_BEYOND_END_OF_ALLOCATION (d6)
N bytes of memory was allocated and more than N bytes are being referenced.
This cannot be protected by try-except.
When possible, the guilty driver's name (Unicode string) is printed on
the bugcheck screen and saved in KiBugCheckDriver.
Arguments:
Arg1: fffff9800b5ff000, memory referenced
Arg2: 0000000000000001, value 0 = read operation, 1 = write operation
Arg3: fffff88004f834eb, if non-zero, the address which referenced memory.
Arg4: 0000000000000000, (reserved)
```

We can debug this crash and determine that `notmyfault.sys` wrote beyond its pool buffer.

我们可以调试这一崩溃，判断`notmyfault.sys`执行了越界写。

The call stack shows that `myfault.sys` accessed invalid memory and this generated a page fault.

栈回溯显示了`myfault.sys`访问了非法的内存，导致了一个页错误。

```
kd> k
Child-SP          RetAddr           Call Site
fffff880`04822658 fffff803`721333f1 nt!KeBugCheckEx
fffff880`04822660 fffff803`720acacb nt! ?? ::FNODOBFM::`string'+0x33c2b
fffff880`04822700 fffff803`7206feee nt!MmAccessFault+0x55b
fffff880`04822840 fffff880`04f834eb nt!KiPageFault+0x16e
fffff880`048229d0 fffff880`04f83727 myfault+0x14eb
fffff880`04822b20 fffff803`72658a4a myfault+0x1727
fffff880`04822b80 fffff803`724476c7 nt!IovCallDriver+0xba
fffff880`04822bd0 fffff803`7245c8a6 nt!IopXxxControlFile+0x7e5
fffff880`04822d60 fffff803`72071453 nt!NtDeviceIoControlFile+0x56
fffff880`04822dd0 000007fc`4fe22c5a nt!KiSystemServiceCopyEnd+0x13
00000000`004debb8 00000000`00000000 0x000007fc`4fe22c5a
```

The `!pool` command shows that the address being referenced by `myfault.sys` is special pool.

`!pool`命令显示了`myfault.sys`引用的内存是特殊池。

```
kd> !pool fffff9800b5ff000
Pool page fffff9800b5ff000 region is Special pool
fffff9800b5ff000: Unable to get contents of special pool block
```

The page table entry shows that the address is not valid. This is the guard page used by special pool to catch overruns.

页表项显示了这一地址是非法的。这是特殊池用于捕捉溢出的守护页。

```
kd> !pte fffff9800b5ff000
                                           VA fffff9800b5ff000
PXE at FFFFF6FB7DBEDF98    PPE at FFFFF6FB7DBF3000    PDE at FFFFF6FB7E6002D0    PTE at FFFFF6FCC005AFF8
contains 0000000001B8F863  contains 000000000138E863  contains 000000001A6A1863  contains 0000000000000000
pfn 1b8f      ---DA--KWEV  pfn 138e      ---DA--KWEV  pfn 1a6a1     ---DA--KWEV  not valid
```

The allocation prior to this memory is an `800` byte block of non paged pool tagged as `"Wrap"`. `"Wrap"` is the tag used by `verifier` when pool is allocated without a tag, it is the equivalent to the `"None"` tag we saw in Part 1.

该段内存前的`800`字节非分页内存池块被标记为`"Wrap"`。`"Wrap"`是`verifier`在分配未使用标签时，使用的标签，它和第一部分看到的`"None"`标签是等价的。

```
kd> !pool fffff9800b5ff000-1000
Pool page fffff9800b5fe000 region is Special pool
*fffff9800b5fe000 size:  800 data: fffff9800b5fe800 (NonPaged) *Wrap
            Owning component : Unknown (update pooltag.txt)
```

Special pool is an effective mechanism to track down buffer overflow pool corruption. It can also be used to catch other types of pool corruption which we will discuss in future articles.

特殊池是一个追溯缓冲区溢出污染的有效机制。他也可以用于捕捉其他类型的池污染，我们会在下文中说明。

## 理解池污染第三部分：特殊池下的 Double Free

In [Part 1](https://bbs.kanxue.com/elink@5a0K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6K6i4K6u0W2L8i4y4V1L8W2)9J5k6h3y4G2L8g2)9J5c8X3u0Q4x3V1k6F1N6r3c8W2j5Y4g2Y4k6$3W2F1k6#2)9J5c8X3q4J5j5$3S2A6N6X3g2Q4x3V1j5J5x3o6p5K6i4K6u0r3x3o6k6Q4x3V1j5I4y4q4)9J5c8Y4g2F1k6r3g2J5M7%4c8S2L8X3c8A6L8X3N6Q4x3X3c8H3L8$3!0D9i4K6u0V1j5$3!0J5M7Y4g2H3N6r3W2G2L8W2)9J5k6s2m8S2M7Y4c8Q4x3X3b7I4i4K6u0V1j5Y4g2X3k6X3g2J5i4K6u0V1L8%4k6W2M7X3k6D9L8%4N6K6i4K6u0W2j5i4y4H3P5l9`.`.) and [Part 2](https://bbs.kanxue.com/elink@851K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6K6i4K6u0W2L8i4y4V1L8W2)9J5k6h3y4G2L8g2)9J5c8X3u0Q4x3V1k6F1N6r3c8W2j5Y4g2Y4k6$3W2F1k6#2)9J5c8X3q4J5j5$3S2A6N6X3g2Q4x3V1j5J5x3o6p5K6i4K6u0r3x3o6S2Q4x3V1j5J5x3W2)9J5c8Y4g2F1k6r3g2J5M7%4c8S2L8X3c8A6L8X3N6Q4x3X3c8H3L8$3!0D9i4K6u0V1j5$3!0J5M7Y4g2H3N6r3W2G2L8W2)9J5k6s2m8S2M7Y4c8Q4x3X3b7J5i4K6u0V1M7%4m8W2j5$3W2S2L8q4)9J5k6s2m8G2L8$3I4Q4x3X3c8X3L8%4u0Q4x3X3c8T1N6h3k6X3k6i4u0Q4x3X3c8G2N6X3g2J5M7Y4g2F1M7#2)9J5k6h3q4K6M7s2R3`.) of this series we discussed pool corruption and how special pool can be used to identify the cause of such corruption. In today’s article we will use special pool to catch a double free of pool memory.

在第一和第二部分我们讨论了池污染以及如何使用特殊池来侦测池污染的真凶。今天我们将使用特殊池来捕捉池内存的二次释放。

A double free of pool will cause a system to blue screen, however the resulting crash may vary. In the most obvious scenario a driver that frees a pool allocation twice will cause the system to immediately crash with a stop code of `C2` `BAD_POOL_CALLER`, and the first parameter will be `7` to indicate “Attempt to free pool which was already freed”. If you experience such a crash, enabling special pool should be high on your list of troubleshooting steps.

池的二次释放会引起一个系统蓝屏，然而导致崩溃的结果多种多样。最明显的就是一个驱动释放了同一个池块两次，此时立即引起系统崩溃，停止码为`0xC2` `BAD_POOL_CALLER`，第一个参数是`7`，表示“试图释放已经释放的池块”。如果你遇到过这样的崩溃，启用特殊池应该是定位问题的不二手段。
