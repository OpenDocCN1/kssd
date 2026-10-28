
### 3. 一般而言，前面2步为初始化操作。为了实际使用内存，应用程序需要 `MapUserPhysicalPages` 或 `MapUserPhysicalPagesScatter`函数，来将第1步分配的一部分物理区域映射到第2步分配的其中一个虚拟区域或窗口。

Figure 10-5 shows an example. The application has created a 256-MB window in its address space and has allocated 4 GB of physical memory (on a system with more than 4 GB of physical memory).

It can then use `MapUserPhysicalPages` or `MapUserPhysicalPagesScatter` to access any portion of the physical memory by mapping the desired portion of memory into the 256-MB window. The size of the application’s virtual address space window determines the amount of physical memory that the application can access with any given mapping. To access another portion of the allocated RAM, the application can simply remap the area.

图10-5展示了一个例子。应用程序在其自身地址空间中，创建一个256MB大小的窗口并且为该窗口分配4GB的物理内存（在一个超过4GB物理内存的系统上）。应用程序接着可以使用 `MapUserPhysicalPages` 或 `MapUserPhysicalPagesScatter`函数，通过将期望的物理内存（4GB中的一部分）映射到256MB的窗口，来对其进行访问。应用程序虚拟地址空间窗口的大小决定了它能够通过任意给定映射访问的物理内存数量。应用程序可以简单地重映射区域来访问已分配RAM中的其它部分。

![](https://bbs.kanxue.com/upload/attach/201509/533967_fd7526270892c50c.jpg)

The AWE functions exist on all editions of Windows and are usable regardless of how much physical memory a system has. However, AWE is most useful on 32-bit systems with more than 2 GB of physical memory because it provides a way for a 32-bit process to access more RAM than its virtual address space would otherwise allow. Another use is for security purposes: because AWE memory is never paged out, the data in AWE memory can never have a copy in the paging file that someone could examine by rebooting into an alternate operating system. (`VirtualLock` provides the same guarantee for pages in general.)

所有Windows版本中都提供了AWE函数，并且无论系统有多少物理内存，它都可以正常工作。然而，在有多于2GB物理内存的32位系统上，AWE最有帮助，因为它提供一种方式，让32位进程能够访问更多其自身虚拟地址空间原本不允许访问的RAM。另一个安全用途是：由于AWE内存永远不会换出（到磁盘上的页面文件），AWE内存中的数据就不会在页面文件中有一份副本，从而别有用心人士就无法通过重启进入其它操作系统来查看页面文件中的副本。（译注：将安装在其它硬盘上的操作系统加载到内存后，查看目标硬盘分页文件的脱机攻击）（总体上，`VirtualLock`函数为页面提供相同的安全保障）

Finally, there are some restrictions on memory allocated and mapped by the AWE functions:

■ Pages can’t be shared between processes.

■ The same physical page can’t be mapped to more than one virtual address in the same process.

■ Page protection is limited to read/write, read-only, and no access.

最后，在使用AWE函数进行内存分配和映射方面，有一些限制：

■　页面不能在进程间共享；

■　相同的物理页只能被映射到同一个进程中的单个虚拟地址窗口（即多个虚拟地址窗口不能映射相同的物理页，参考图10-5很容易理解）

■　AWE的页面保护选项被限制为：读/写，只读，以及不允许访问。

AWE is less useful on x64 or IA64 Windows systems because these systems support 8 TB or 7 TB (respectively) of virtual address space per process, while allowing a maximum of only 2 TB of RAM.

Therefore, AWE is not necessary to allow an application to use more RAM than it has virtual address space; the amount of RAM on the system will always be smaller than the process virtual address space. AWE remains useful, however, for setting up nonpageable regions of a process address space. It provides finer granularity than the file mapping APIs (the system page size, 4 KB or 8 KB, versus 64 KB).

For a description of the page table data structures used to map memory on systems with more than 4 GB of physical memory, see the section “Physical Address Extension (PAE).”

在 x64 或 IA64 Windows系统上，AWE用处不大，因为此类系统分别支持每进程 8TB 或 7TB 的虚拟地址空间，而仅允许最大 2TB 的RAM。因此，在允许一个应用程序使用超过其自身虚拟地址空间代表的物理内存方面（即 8TB 或 7TB），AWE不是必需的；此类系统上的RAM数量将总是小于进程的虚拟地址空间。即便如此，AWE仍旧有用，例如设置一个进程地址空间中不可分页区域。

在这方面，AWE提供比文件映射 APIs 更细的粒度（AWE提供 4KB 或 8KB 的系统页尺寸分配粒度，而文件映射 APIs 只能基于 64KB 边界对齐的分配粒度）。

参见本章“物理地址扩展”一节，其中讨论了带有4GB以上物理内存的系统用于映射内存的页表数据结构。

**至此，第一部分就结束了，感谢各位能够坚持把这篇陋文读完，第二部分涉及更深入的内容，包括在第一部分多处提到的一些内存管理器组件和服务的具体数据结构实现，算法描述；其利用的硬件特性与 Intel x86/x64 体系结构紧密相关，尤其是对地址翻译，内存保护以及分页相关机制的解析，因此若能够与《Intel 64 与 IA-32 架构软件开发者手册》系列配套阅读能够有助于理解（后者也是英文原版，有1000多页需要翻译。。。）另外，实验解析了各种场景下Windows内核调试器的输出，因此第二部分的翻译将更有难度，期待你们的反馈信息！**

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [web584.png](javascript:void(0);)

  （63.52kb，11次下载）

- [web585.png](javascript:void(0);)

  （73.16kb，7次下载）

- [web587.png](javascript:void(0);)

  （65.32kb，8次下载）

- [web586.png](javascript:void(0);)

  （80.32kb，4次下载）

- [web588.png](javascript:void(0);)

  （68.08kb，7次下载）

- [web589.png](javascript:void(0);)

  （113.37kb，7次下载）
