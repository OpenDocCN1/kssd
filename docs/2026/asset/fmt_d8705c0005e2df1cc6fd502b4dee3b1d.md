
# [翻译]Anatomy of a Program in Memory—剖析内存中的一个程序（进程的虚拟存储器映像布局详解）

前言：原文来自于：[cddK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8#2j5i4u0@1k6i4y4Q4x3X3g2G2M7X3N6Q4x3V1k6Y4N6i4y4@1j5i4k6G2i4K6u0r3j5X3I4G2k6#2)9J5c8Y4m8G2M7%4c8Q4x3V1k6S2L8X3q4@1L8$3#2&6i4K6u0V1L8$3k6Q4x3X3c8S2i4K6u0V1M7s2u0G2k6%4u0S2L8g2)9J5k6r3W2F1i4K6u0V1L8h3g2E0L8%4u0&6i4K6u0r3](https://bbs.kanxue.com/elink@850K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8#2j5i4u0@1k6i4y4Q4x3X3g2G2M7X3N6Q4x3V1k6Y4N6i4y4@1j5i4k6G2i4K6u0r3j5X3I4G2k6#2)9J5c8Y4m8G2M7%4c8Q4x3V1k6S2L8X3q4@1L8$3#2&6i4K6u0V1L8$3k6Q4x3X3c8S2i4K6u0V1M7s2u0G2k6%4u0S2L8g2)9J5k6r3W2F1i4K6u0V1L8h3g2E0L8%4u0&6i4K6u0r3)，这里只是对其进行翻译，并且重构了原文中的图片。译注则是我增加的内容，用来解释原文或提出问题；由于个人水平有限，译文和译注中的错误之处还请广大坛友提出指正，不胜感激。

下面采用分段中英对照的方式列出内容：

## 内存管理与进程地址空间

**Memory management is the heart of operating systems; it is crucial for both programming and system administration. In the next few posts I’ll cover memory with an eye towards practical aspects, but without shying away from internals. While the concepts are generic, examples are mostly from Linux and Windows on 32-bit x86. This first post describes how programs are laid out in memory.**

内存管理是操作系统的核心功能之一，对编程与系统管理而言都很关键。在接下来的文章里，我将着眼于讨论实践方面的内存，但是也不回避其内部原理。概念是通用的，例子多数来自运行在32位 X86体系结构处理器上的 Linux 与 Windows操作系统。第一篇文章描述了程序在内存中是如何布局的。

**Each process in a multi-tasking OS runs in its own memory sandbox. This sandbox is the virtual address space, which in 32-bit mode is always a 4GB block of memory addresses. These virtual addresses are mapped to physical memory by page tables, which are maintained by the operating system kernel and consulted by the processor. Each process has its own set of page tables, but there is a catch. Once virtual addresses are enabled, they apply to all software running in the machine, including the kernel itself.

Thus a portion of the virtual address space must be reserved to the kernel:**

一个多任务操作系统的每个进程运行在自身的内存沙箱中。该沙箱是虚拟的地址空间，在32位（译注：保护？）模式下总是4GB大小的内存地址块。这些进程的虚拟地址通过页表被映射至物理内存，

页表由操作系统内核维护，并且可被处理器查阅。每个进程都有它自己的一组页表，但这里有一个问题。一旦启用虚拟地址，它们将应用于所有机器上运行的软件，包括内核自身。

因此，一部分虚拟地址空间需要保留给内核：（下图展示了现代操作系统中的每个进程的一致虚拟内存布局，内核占有其中一部分的地址空间）

![](https://bbs.kanxue.com/upload/attach/201508/533967_emk3wh72r931f0t.png)

**This does not mean the kernel uses that much physical memory, only that it has that portion of address space available to map whatever physical memory it wishes. Kernel space is flagged in the page tables as exclusive to privileged code (ring 2 or lower), hence a page fault is triggered if user-mode programs try to touch it. In Linux, kernel space is constantly present and maps the same physical memory in all processes. Kernel code and data are always addressable, ready to handle interrupts or system calls at any time. By contrast, the mapping for the user-mode portion of the address space changes whenever a process switch happens:**

这并不意味着内核就一定使用这么多的物理内存，只有内核可用的那部分虚拟地址空间（1GB）能够映射至物理内存。内核空间在页表中以其独有的特权码（ring2 或者更低）来标记，因此当一个用户模式程序试图接触内核空间时，会引发页面错误。在 Linux 中，内核空间总是存在于所有进程中，并且映射至相同的物理内存。内核代码和数据总是可寻址的，从而能够准备随时处理中断和系统调用（这些都需要处理器转入内核空间访问其代码和数据结构）。相比之下，每当发生进程（任务）切换时，地址空间中的用户模式部分（用户空间）到物理内存的映射也会变化，这一点可以通过下图加以说明：

![](https://bbs.kanxue.com/upload/attach/201508/533967_02ybf8m9f4rq8ow.png)

**Blue regions represent virtual addresses that are mapped to physical memory, whereas white regions are unmapped. In the example above, Firefox has used far more of its virtual address space due to its legendary memory hunger. The distinct bands in the address space correspond to memory segments like the heap, stack, and so on. Keep in mind these segments are simply a range of memory addresses and have nothing to do with Intel-style segments. Anyway, here is the standard segment layout in a Linux process:**

上图中的蓝色区域代表映射至物理内存的虚拟地址，而白色区域则未映射。在上面的例子中，Firefox 浏览器进程由于其惊人的内存需求，已经使用了远多于自身虚拟地址空间的（相应物理）内存。

地址空间中不同（分隔的）条带区域对应诸如堆，栈等的内存段。需要留意的是，这些内存段只是一个虚拟内存地址范围，它与 Intel 的物理内存分段寻址（译注：数据段，代码段，堆栈段。。。这些段的基址通过相应的段描述符保存在全局描述符表即GDT这个裸机内存/内核级数据结构中，也可以将GDT的基址加载到GDTR即全局描述符表寄存器中，处理器通过查找前者内容实现段式寻址）

没有关系。总之，下图就是一个 Linux 进程的虚拟地址空间/内存映射的标准布局：

![](https://bbs.kanxue.com/upload/attach/201508/533967_710ec37750c81d07.jpg)

## 地址空间随机化

**When computing was happy and safe and cuddly, the starting virtual addresses for the segments shown above were exactly the same for nearly every process in a machine.

This made it easy to exploit security vulnerabilities remotely. An exploit often needs to reference absolute memory locations: an address on the stack, the address for a library function, etc. Remote attackers must choose this location blindly, counting on the fact that address spaces are all the same. When they are, people get pwned. Thus address space randomization has become popular. Linux randomizes the stack, memory mapping segment, and heap by adding offsets to their starting addresses. Unfortunately the 32-bit address space is pretty tight, leaving little room for randomization and hampering its effectiveness.**

如果计算机愉快安全且准确无误的运行程序，那么上图所示的每个内存段的起始虚拟地址，对于计算机上的几乎每一个进程而言都是完全相同的。这使得远程利用安全漏洞（译注：远程执行恶意代码？）变得很容易。一个（漏洞）利用往往需要引用绝对内存位置：一个用户栈中的地址，一个库函数的（起始）地址等等。远程攻击者必须尝试摸索性的选择（译注：猜测？）这个位置，并且指望地址空间均相同成为事实。如果它们真的遇到这种福利（译注：程序每次运行时的某个栈地址或库函数地址都不会变化），人们显然就会被黑客攻击了。由于这个关系，地址空间加载随机化技术（译注：windows 中的 ASLR?）开始变得流行起来。在 Linux 中，随机化的有栈地址，用户共享内存映射段（译注：参考上图），以及通过为堆的起始地址添加偏移量实现随机化。

遗憾的是，32位地址空间显得相当吃紧，使得随机化只能在很小的范围中进行而且成效不彰。

## 栈的工作原理

**The topmost segment in the process address space is the stack, which stores local variables and function parameters in most programming languages. Calling a method or function pushes a new stack frame onto the stack. The stack frame is destroyed when the function returns. This simple design, possible because the data obeys strict LIFO order, means that no complex data structure is needed to track stack contents – a simple pointer to the top of the stack will do. Pushing and popping are thus very fast and deterministic. Also, the constant reuse of stack regions tends to keep active stack memory in the cpu caches, speeding up access. Each thread in a process gets its own stack.**

在进程地址空间顶端的那段内存就是用户栈，它用于存储绝大多数编程语言中的局部变量，以及函数调用时传递的参数。

调用一个方法或函数会把一个新栈帧压到栈上（译注：这里的意思应该是改变栈顶指针即ESP指向更低的内存地址，这样相较原来的栈就形成了一段新栈帧）。该栈帧在函数返回时被销毁。（译注：即ESP回指更高的，调用前的内存地址，从而删除被调函数的栈帧）

栈的这种简单设计可能是因为考虑了数据遵守严格的 LIFO 顺序（后进先出）这一点。这意味着不需要用复杂的数据结构来追踪栈的内容——一个简单的栈顶指针（ESP）就能搞定。

得益于栈顶指针，压（进）栈和弹（出）栈的操作变得非常快捷且明确。再者，栈区的反复使用往往能够使其保持活跃，从而驻留在CPU缓存中，加快访问速度。一个进程中的每个线程都有其自身的栈。

（为了直观理解作者的意思，我制作了一张函数调用时的栈存储变化过程图解，便于参考：）

![](https://bbs.kanxue.com/upload/attach/201508/533967_757c2e9b85a64cff.jpg)

**It is possible to exhaust the area mapping the stack by pushing more data than it can fit. This triggers a page fault that is handled in Linux by expand_stack(), which in turn calls acct_stack_growth() to check whether it’s appropriate to grow the stack. If the stack size is below RLIMIT_STACK (usually 8MB), then normally the stack grows and the program continues merrily, unaware of what just happened. This is the normal mechanism whereby stack size adjusts to demand. However, if the maximum stack size has been reached, we have a stack overflow and the program receives a Segmentation Fault. While the mapped stack area expands to meet demand, it does not shrink back when the stack gets smaller. Like the federal budget, it only expands.**

向栈中压入超过其自身能容纳的数据，就有可能耗尽映射的栈内存区。这将触发一个由 Linux 函数 `expand_stack()` 处理的页面错误，该函数转而调用 `acct_stack_growth()` 函数去检查是否应该适当地增大栈内存，如果被填满的栈内存低于 `RLIMIT_STACK`（通常是8MB），一般而言此时栈内存会增大并且程序继续运行，不会察觉到刚才发生了什么事。这是按需调整栈大小的常规机制。

然而，假设达到了栈内存大小上限，就会产生栈溢出，并且程序接收到一个分段错误信息。

尽管映射的栈内存可以扩展来满足需求，却不能在实际使用需求减少时收缩回来以释放多余的空间。

（译注：相比之下，构成栈内存的一个个函数调用栈帧则可以按需增减，由ESP决定。销毁栈帧不会导致栈收缩；展开栈帧不会导致栈扩展，除非填满当前栈大小，而扩展上限就是8MB）

这就像联邦政府的预算，只会越来越多。

**Dynamic stack growth is the only situation in which access to an unmapped memory region, shown in white above, might be valid. Any other access to unmapped memory triggers a page fault that results in a Segmentation Fault. Some mapped areas are read-only, hence write attempts to these areas also lead to segfaults.**

`Dynamic stack growth` 是唯一一种情况下访问上图所示的白色未映射内存区域可能是合法的。任何其他对未映射内存的访问都会触发页面错误，从而导致分段错误。一些映射区域是只读的，因此对这些区域的写尝试也会导致段错误。
