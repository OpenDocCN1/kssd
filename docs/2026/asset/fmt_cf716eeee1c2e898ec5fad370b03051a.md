
\* <a href="elink@bc0K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4U0M7#2)9J5k6h3&6W2N6g2)9J5k6h3g2V1N6g2)9J5c8X3S2G2L8h3g2Q4x3V1k6H3K9X3c8Q4x3V1k6U0M7$3M7I4x3e0u0Q4x3X3c8X3x3o6S2Q4x3V1k6d9k6h3q4V1K9h3&6Y4i4K6g2X3j5i4y4K6K9h3N6F1L8h3g2F1N6q4)9#2k6X3c8#2k6g2)9#2k6U0p5I4i4K6u0V1x3e0m8Q4y4h3j5H3x3g2)9J5c8U0j5K6z5e0M7J5y4o6u0Q4x3X3g2H3k6r3j5`.">Virtualization System Including a Virtual Machine Monitor For A Computer With A Segmented Architecture</a>

\* <a href="elink@9fcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6%4K9i4y4Z5M7%4c8#2k6r3W2G2i4K6u0r3k6X3I4A6L8Y4g2^5">Foreign LINUX - Run unmodified Linux applications inside Windows</a>

\* <a href="elink@fd2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6H3K9i4W2#2M7#2)9J5c8X3u0@1K9$3g2J5L8X3g2D9">Fast Binary Translator for the Kernel</a>

\* <a href="elink@6a6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6e0x3V1g2Q4x3V1k6D9K9h3u0U0M7s2f1`.">x86 Dynamic Binary Translator Library</a>

原文链接：<a href="elink@ddeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6j5h3k6W2M7Y4N6S2L8r3I4Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3!0Y4i4K6u0r3N6X3W2J5N6s2g2S2L8r3W2\*7j5i4c8A6L8$3&6Q4x3X3c8A6L8Y4c8W2M7X3&6S2L8s2y4Q4x3X3c8H3j5i4u0@1i4K6u0V1x3W2)9J5k6s2k6E0N6$3q4J5k6g2)9J5k6r3q4F1k6q4)9J5k6s2k6A6M7Y4c8#2j5h3I4A6P5X3q4@1K9h3!0F1i4K6u0V1N6i4y4A6L8X3N6Q4x3X3c8T1K9h3&6S2M7Y4W2Q4x3X3c8@1M7X3q4F1M7$3I4S2N6r3W2G2L8R3`.`."><mark class="encrypted">5f9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6j5h3k6W2M7Y4N6S2L8r3I4Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3!0Y4i4K6u0r3N6X3W2J5N6s2g2S2L8r3W2\*7j5i4c8A6L8$3&6Q4x3X3c8A6L8Y4c8W2M7X3&6S2L8s2y4Q4x3X3c8H3j5i4u0@1i4K6u0V1x3W2)9J5k6s2k6E0N6$3q4J5k6g2)9J5k6r3q4F1k6q4)9J5k6s2k6A6M7Y4c8#2j5h3I4A6P5X3q4@1K9h3!0F1i4K6u0V1N6i4y4A6L8X3N6Q4x3X3c8T1K9h3&6S2M7Y4W2Q4x3X3c8@1M7X3q4F1M7$3I4S2N6r3W2G2L8R3`.`.</mark></a>

编译 看雪翻译小组

翻译： [sudozhange](https://bbs.pediy.com/user-703263.htm)

校对： [StrokMitream](https://bbs.pediy.com/user-747320.htm)

# 深入解析虚拟化（二）——VMWare和使用二进制翻译的虚拟化

> 译者注：由于本片文章涉及较多的操作系统以及编译原理相关知识，译者才疏学浅，难免有翻译的不准确或错误的地方，希望大家多多包涵，欢迎评论区批评指正。同时要感谢校对人员认真的校对。

在前面的章节中，我们介绍了一些关于管理程序的基本概念，并简要介绍了x86虚拟化的不同技术：使用二进制翻译的全虚拟化，超虚拟化和硬件虚拟化。今天，我们回深入研究全虚拟化，特别是早期版本的VMWare Workstation如何成功将虚拟化带回到x86中，不管缺少虚拟化及时支持和架构的深度复杂性。

- [深入解析虚拟化（一）—— 虚拟化简介](https://bbs.kanxue.com/elink@3f8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6j5h3k6W2M7Y4N6S2L8r3I4Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3!0Y4i4K6u0r3N6X3W2J5N6s2g2S2L8r3W2*7j5i4c8A6L8$3&6Q4x3X3c8A6L8Y4c8W2M7X3&6S2L8s2y4Q4x3X3c8H3j5i4u0@1i4K6u0V1x3g2)9J5k6r3W2F1N6s2u0G2i4K6u0V1N6r3!0Q4x3X3c8$3K9i4u0@1N6h3q4D9K9i4A6S2N6r3W2G2L8R3`.`.) [[*译文*](https://bbs.pediy.com/thread-228395.htm)]

- **深入解析虚拟化（二）——VMWare和使用二进制翻译的全虚拟化**

- 深入解析虚拟化（三）——Xen和超虚拟化

在我们进一步讨论前，我想强调，本章将讨论的内容是专门设计用于在引入64位扩展或硬件支持虚拟化（ [VT-x和AMD-v](https://bbs.kanxue.com/elink@a06K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6j5z5o6k6Q4y4h3k6$3K9i4u0@1N6h3q4D9K9i4A6S2N6r3W2G2L8W2)9J5x3@1W2F1N6r3g2D9i4K6g2X3N6X3W2J5N6s2g2S2L8r3W2*7j5i4c8A6L8$3&6Q4y4h3k6Q4x3U0S2h3g2q4)9J5k6s2R3`.) )[2006] 之前的虚拟化x86架构。VMware当前的市面上虚拟机管理程序（VMM）与原始设计明显不同。不过，你将学的知识将扩展您对虚拟化和底层概念的理解。

## 关于VMWare的一些话

VMWare从两种管理程序解决方案开始：**Workstation** 和 **ESX**。VMWare Workstation的第一个发布版可追溯到1999年（[发布版本历史](https://bbs.kanxue.com/elink@f03K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4k6A6M7Y4c8W2L8W2)9J5k6h3&6W2N6q4)9J5c8Y4k6E0N6$3q4J5k6g2)9J5c8X3g2K6P5r3W2Q4x3X3c8J5k6h3I4W2j5i4y4W2i4K6u0V1j5Y4g2A6L8r3c8Q4x3X3c8F1N6h3#2T1k6i4u0Q4x3X3c8Z5K9i4y4@1L8%4u0&6i4K6u0r3)）。ESX在2001年出现（[发布版本历史](https://bbs.kanxue.com/elink@ce1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4k6A6M7Y4c8W2L8W2)9J5k6h3&6W2N6q4)9J5c8Y4k6E0N6$3q4J5k6g2)9J5c8X3g2K6P5r3W2Q4x3X3c8J5k6h3I4W2j5i4y4W2i4K6u0V1j5Y4g2A6L8r3c8Q4x3X3c8F1N6h3#2T1k6i4u0Q4x3X3c8Z5K9i4y4@1L8%4u0&6i4K6u0r3)）。Workstation被认为是宿主架构(类型1)，然而ESX是运行在裸金属架构（类型2）。在这篇文章中，我们将专注于**VMWare Workstation**。

![图片描述](https://bbs.kanxue.com/upload/attach/201807/703263_E5NFNTCFR4QE4PV.png)

如果你想查看VMM（虚拟机管理程序），从[此处](https://bbs.kanxue.com/elink@6deK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3I4A6L8Y4g2^5x3W2)9J5k6h3#2S2N6r3S2W2L8h3q4@1K9h3E0Q4x3X3g2@1N6g2)9J5k6r3c8S2M7X3#2K6N6r3q4V1N6q4)9J5k6h3c8W2i4K6u0r3M7s2g2T1i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6E0K9i4u0J5L8%4u0K6i4K6u0r3L8h3W2K6j5#2)9J5c8X3!0D9k6q4)9J5c8Y4k6E0N6$3q4J5k6g2)9J5c8W2k6y4N6$3q4J5k6g2)9J5k6o6q4Q4x3X3f1H3i4K6u0W2x3g2)9J5k6o6x3%4x3W2)9J5k6h3g2^5k6b7`.`.)下载安装程序，将其安装到Windows XP VM中，安装后，在**ProgramFiles**目录中找到 `vmware.exe`，使用PE资源编辑器（如[`CFF Explorer`](https://bbs.kanxue.com/elink@d6bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3&6@1j5$3!0J5k6g2)9J5k6h3y4G2L8g2)9J5c8X3g2^5M7%4g2A6N6r3g2Q4x3X3g2H3K9s2l9`.)）打开它，并转储二进制文件，VMM是一个**ELF**文件。

## VMWare Workstation的宿主架构

正如我们在第一篇文章中看到的，宿主架构允许将虚拟化插入到现有的操作系统中。VMWare打包为一个正常的应用程序，其中包含一系列的驱动和可执行/dll文件。作为正常应用程序运行有很多好处。首先，VMWare依靠主机图形用户界面，以至于每个虚拟姐屏幕的内容可以自然的出现在一个特别的窗口中，这将是很好的用户体验。另一方面，每个虚拟机实例以进程（**`vmware-vmx.exe`**）的形式运行在主机操作系统，可以独立启动，监控，终止。该进程在本章中会被标记为**VMX**。

![图片描述](https://bbs.kanxue.com/upload/attach/201807/703263_GHKVWS2HGJPGEGA.png)

除此之外，在主机OS上运行有助于 I/O 设备模拟。由于操作系统可以使用自己的设备驱动和 I/O 设备通讯，因此VMWare支持通过标准系统调用到主机操作系统来模拟设备。例如，它会读写宿主机文件系统来模拟虚拟磁盘设备，或者在宿主机的桌面窗口中绘画来模拟显卡。只要宿主机操作系统中有合适的驱动程序，VMWare就可以在其上运行虚拟机。

然而，正常的应用程序没有VMM的必要API或工具来复用CPU和内存资源。因此，VMWare似乎只运行在当前操作系统的顶端，实际上，它的VMM可以在系统级运行，完全控制硬件。事实上，主机操作系统恰当地假设它一直在控制硬件资源。但是，VMM实际上在一段有限的时间内控制硬件，在这段时间内主机操作系统被从虚拟和线性内存中短暂移除。

![](https://bbs.kanxue.com/upload/attach/201807/703263_C3PRDTGH4SQP4JV.png)

从上图中可以看出，在任何时候，每个CPU可以位于：

- 操作系统完全控制的宿主机OS上下文，或者；

- VMM完全控制的VMM上下文

VMM和宿主机操作系统的上下文切换又称为**世界切换（world switch）**。每个上下文有自己的地址空间，中断描述符表，堆栈，执行上下文。驻留在宿主机的**VMM驱动**程序实现了一系列操作，包括锁定物理内存页，转发中断以及调用世界切换原语。就主机操纵系统而言，设备驱动是标准的可加载的内核模块。但不是驱动某些硬件设备，而是驱动VMM并将其从宿主机操作系统完全隐藏。

当一个设备产生中断时，CPU可能在主机上下文和VMM上下文中运行。在第一种情况下，CPU通过**中断描述符表**（Interrupt Descriptor Table ,IDT）将控制权交给主机操作系统。在任何VMM上下文中发生中断的第二种情况下，涉及步骤（i）-（v）：

- i：VMM被CPU中断，并触发VMM外部中断处理程序执行。

- ii：中断处理程序立刻触发世界切换回主机操作系统上下文，**`idtr`**恢复为指向主机操作系统中断表。译者注：`idtr`，内存管理寄存器之一。

- iii：内核驻留 驱动将控制权交给由主机操作系统指定的中断处理程序。

- iv： 这只需通过发出一个 `int <vector>`指令实现，其中**`<vector>`**指令对应于原始的外部中断。主机操作系统的中断处理程序随后正常运行，就像在VMX进程中，当VMM驱动正在处理**`ioctl`**时发生了外部I/O中断一样。

- v:VMM驱动程序随后将控制权返还给用户级的VMX进程，从而主机操作系统有机会决定优先调度。

处理物理中断的一部分，这个插图展示出VMWare如何依靠VMs发出I/O请求，所有这些虚拟I/O请求都是使用VMM和VMX进程间的RPC调用来执行的，VMX进程随后执行正常对主机操作OS的系统调用。为了允许虚拟机和它自己挂起的I/O请求重叠执行，VMX进程运行不同的线程：

- **模拟器**线程（Emulator thread）处理执行VM指令的主循环，并模拟设备前端，作为RPC调用处理的一部分。

- 其他线程**异步IO**（Asychrounous IO，AIO）负责所有可能阻塞操作的执行。

现在返回到世界切换，它与之前可能遇到的传统上下文切换非常相似（如在内核空间和用户空间之间，或者在调试器和调试对象之间），提供了加载并执行虚拟机上下文的低级VMM机制，以及恢复主机操作系统上下文的反向机制。

![](https://bbs.kanxue.com/upload/attach/201807/703263_SWHGG6GH5YCM8TP.png)

上图指明了世界切换程序如何将主机切换到VMM上下文，反之亦然。VMM将离开前4MB空间。**交叉页面**（cross page）是单页内存，以非常特殊的方式使用，是世界切换的重点。交叉页面由内核驻留驱动分配到主机操作系统的内核地址空间。由于驱动程序使用标准API进行分配，因此主机操作系统决定了交叉页面的地址。

紧接着，在每次世界切换前后，交叉页面也映射到VMM地址空间。交叉页面包含世界切换的代码和数据结构。下面是双向执行的指令反汇编：

![图片描述](https://bbs.kanxue.com/upload/attach/201807/703263_D64WGME2UHN84XG.png)

**VMX**进程表示主机上的虚拟机。它的作用是分配，锁定和最终释放所有内存资源。此外，它以文件映射（映射到自己的地址空间）的方式管理VM的物理内存（Linux使用**`mmap`**或者Windows使用**文件映射**（file mapping）API）。虚拟设备[`DMA`](https://bbs.kanxue.com/elink@b5fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6p5K9i4u0W2j5%4c8Q4y4h3k6E0k6h3#2G2M7Y4W2Q4y4h3k6S2j5$3y4W2M7%4x3`.)的模拟是通过VMX对映射文件对应部分的简单**bcopy**，**read**或**write**操作实现的。VMX和驻留在内核的驱动程序协同工作来为**客户机物理地址**（Guest Physical Address，`gPA`）锁定的页面提供**机器物理地址**（Machine Physical Address，`mPA`）。

在Windows上显示锁定页面的屏幕截图。

## 虚拟机监视器

既然我们对VMWare的整体托管架构有了一个概念，现在让我们转到VMM本身以及它的运作方式。我们之前已经看到，VMM的主要功能是虚拟化CPU和内存。我们也讨论了虚拟机通常使用称为**陷阱和模拟**（trap-and-emulate）的方法运行。在陷阱和模拟方式的VMM中，客户机代码直接运行在CPU上，但**减少了权限**(reduced privilege)。当客户机尝试读或修改特权状态时，处理器会生成一个将控制权交给VMM的陷阱。VMM随后使用解释器模拟指令并在下一条指令恢复客户机代码的直接执行。我们说过x86不能使用陷阱和模拟，因为许多如**敏感非特权指令**(sensitive non-privileged instructions)的阻碍。那么如何继续？
