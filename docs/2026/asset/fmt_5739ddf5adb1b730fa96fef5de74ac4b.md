
在“调用者/调用者的调用者”一列中，感叹号前面的就是驱动程序文件名，其省略了`.sys`文件名后缀，这些驱动程序的二进制文件，绝大多数可以在`C:\Windows\System32\drivers`路径下找到；感叹号右侧的则是该驱动中，请求`MiObtainSystemVa()`分配系统 PTE 的例程（包括直接或间接递归调用），以及偏移位置。从这些例程的名称可以看出，它们请求在系统 PTE 区域中分配内存的目的都是与映射视图，MDLs（内存描述符列表），适配器内存映射，驱动程序映像，内核栈，I/O 映射等相关的。例如，`MiFindContiguousMemory()` -> `MmMapIoSpace()` -> `MiInsertIoSpaceMap` -> `ExAllocatePoolWithTag()` -> `MiAllocatePoolPages()` -> `MiAllocatePagedPoolPages()` -> `MiObtainSystemVa()`

由此可知，无论是直接还是间接调用`MiObtainSystemVa()`，来在系统 PTE 区域分配内存，都会被跟踪记录下来（启用系统 PTE 分配者跟踪后）。

再如`hal!HalpAllocateAdapterEx`，从其名称就可以推测出是与适配器内存映射相关的例程。我们可以直接使用调试器命令`u`后接完整的例程偏移位置，来反汇编涉及分配操作地址处的机器代码，下面是在真实机器上的输出，注意，Caller/CallersCaller 一列被“Callstack”取代。另外，毕竟是用 Livekd 单机模拟实时调试，下图结果不够准确或者容易让人迷惑，例如无法定位确切的 PTE 分配代码。因此，建议有物理双机环境的童鞋，可以自行验证看看）

![](https://bbs.kanxue.com/upload/attach/201510/533967_dc6da4651ee67c8a.png)

## 64-Bit Address Space Layouts

## 64 位地址空间布局

The theoretical 64-bit virtual address space is 16 exabytes (18,446,744,073,709,551,616 bytes, or approximately 18.44 billion billion bytes). Unlike on x86 systems, where the default address space is divided in two parts (half for a process and half for the system), the 64-bit address is divided into a number of different size regions whose components match conceptually the

portions of user, system, and session space. The various sizes of these regions, listed in Table 10-8, represent current implementation limits that could easily be extended in future releases. Clearly, 64 bits provides a tremendous leap in terms of address space sizes.

理论上的 64 位虚拟地址空间大小为 16 EB（18,446,744,073,709,551,616 字节，或者大约 18.44 十亿十亿字节；1 EB 等于 10 的 18 次方字节）。与 x86 系统上的地址空间布局不一样（其默认地址空间被划分为两部分，一半用于一个进程；另一半用于系统）的是，64 位地址空间被划分成多个不同大小的区域，其组成与概念上的用户，系统，以及会话空间等部分相匹配。表 10-8 列出了这些区域的各种大小；需要指出，其中的数值仅仅代表当前实现中的限制，在后续的发布版中，这些值能够轻易地被扩大（译注：想象一下，16 EB 可以“容纳”多少个 GB，TB 以及 PB 单位）。很显然，“64 位”提供了

地址空间尺寸方面的一个巨大飞跃。（译注：以各位读到这里的水平而言，下表应该能轻松看懂，就不翻译了）

![](https://bbs.kanxue.com/upload/attach/201510/533967_f03ee8a700168cfb.png)

Also, on 64-bit Windows, another useful feature of having an image that is large address space aware is that while running on 64-bit Windows (under Wow64), such an image will actually receive all 4 GB of user address space available—after all, if the image can support 3-GB pointers, 4-GB pointers should not be any different, because unlike the switch from 2 GB to 3 GB,

there are no additional bits involved. Figure 10-11 shows TestLimit, running as a 32-bit application, reserving address space on a 64-bit Windows machine, followed by the 64-bit version of TestLimit leaking memory on the same machine.

此外，在 64 位 Windows 上，另一个有用的功能就是，假设一个标记了意识（aware）到大地址空间的映像文件在 64 位 Windows 上运行时（在 `Wow64` 下），那么该映像将真正获得全部可用的 4 GB 用户地址空间——毕竟，如果该映像能够支持 3 GB 的指针，那么 4 GB 的指针应该不会有任何差异，这是由于，与从 2 GB 切换到 3 GB 不同， 从 3 GB 切换到 4 GB 不会涉及额外的位。图 10-11 展示了作为一个 32 位应用程序运行的 `TestLimit` 程序在一个 64 位 Windows 机器上保留的地址空间大小；该图的下半部分则是 `TestLimit` 程序的 64 位版本在相同机器上泄露（leaking）的内存大小。

（译注：从下半部分输出可以看到，64位 `TestLimit` 程序能够泄露 8388548 MB ，即大约 8388 GB 的内存，这与表 10-8 中的 x64 系统上的进程地址空间限制——8192 GB 非常近似；而上半部分——32 位 `TestLimit` 在 64 位机器上至多能够泄露 4031 MB ，即 4 GB 内存——也与原文中的描述相符）

![](https://bbs.kanxue.com/upload/attach/201510/533967_d67685d9e5782880.png)

Note that these results depend on the two versions of TestLimit having been linked with the `/LARGEADDRESSAWARE` option. Had they not been, the results would have been about 2 GB for each. 64-bit applications linked without `/LARGEADDRESSAWARE` are constrained to the first 2 GB of the process virtual address space, just like 32-bit applications.

注意，这些输出的结果依赖于 `TestLimit` 程序的两个版本（32 位与 64 位）已经使用 `/LARGEADDRESSAWARE` 选项进行了链接。如果不是这样，每个版本的输出结果将约为 2 GB。没有使用 `/LARGEADDRESSAWARE` 链接器选项构建的 64 位应用程序被限制为仅能使用该进程虚拟地址空间的首个 2 GB 范围，这就与 32 位应用程序一样了。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
