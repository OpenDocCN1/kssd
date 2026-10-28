
attempt to decode the pointer, resulting in a corrupted value and causing the program to crash. The `EncodePointer` and `DecodePointer` APIs provide similar protection but with a per-process cookie (created on demand) instead of a per-system cookie.

API 函数 `EncodeSystemPointer` 与 `DecodeSystemPointer` 用于实现指针编码，它们也可以保存计算出的cookie值。当一个应用程序或DLL对静态指针进行动态调用，存在让恶意代码将指针值覆写成恶意软件控制的代码风险。通过用cookie值编码所有指针然后对其进行解码，当恶意代码设置一个非编码的指针，应用程序将仍旧尝试解码该指针，从而导致一个损坏的值并造成程序崩溃。API 函数 `EncodePointer` 与 `DecodePointer` 提供类似的保护功能，但是使用每进程cookie（按需创建）取代每系统cookie。

**Note**  The system cookie is a combination of the system time at generation, the stack value of the saved system time, the number of page faults, and the current interrupt time.

**注意** 系统cookie结合了`system time at generation`，`the stack value of the saved system time`（译注：这2项不知作何翻译），缺页数，以及当前中断次数。

### 实战：结合进程浏览器与内核调试器查看并验证进程的DEP状态

1。首先在进程浏览器中，查看 Visual Studio 2010 主进程 `devenv.exe` 的 DEP 状态为“永久启用”（permanent），如下所示：

![](https://bbs.kanxue.com/upload/attach/201509/533967_0464f2cea676520a.jpg)

2。启动 `KD.exe`，执行 `!process 0 0 devenv.exe` 命令，列出进程的 `EPROCESS` 结构信息，然后显式设置调试上下文为该进程：

![](https://bbs.kanxue.com/upload/attach/201509/533967_66522de9906620ef.jpg)

3。由于每个进程的 `EPROCESS` 结构（执行体进程块）中的第一个成员就是 `KPROCESS` 结构（内核进程块），而 `KPROCESS` 结构中的 `KEXECUTE_OPTIONS` 结构，存储了该进程的 DEP 信息，因此，我们可以使用 `dt nt!_KPROCESS` 命令，后面直接带 `EPROCESS` 结构的线性地址，然后加上 `-r` 开关，递归遍历其中的所有成员，从而找出 `KEXECUTE_OPTIONS` 结构：

![](https://bbs.kanxue.com/upload/attach/201509/533967_d62eb7369c18a7b3.jpg)

![](https://bbs.kanxue.com/upload/attach/201509/533967_5ef8bd7b1d5cc265.jpg)

### Copy-on-Write

`Copy-on-write` page protection is an optimization the memory manager uses to conserve physical memory. When a process maps a copy-on-write view of a section object that contains read/write pages, instead of making a process private copy at the time the view is mapped, the memory manager defers making a copy of the pages until the page is written to. For example, as shown in Figure 10-3, two processes are sharing three pages, each marked copy-on-write, but neither of the two processes has attempted to modify any data on the pages.

### 写时复制

写时复制页面保护是内存管理器用于节约物理内存的一种优化手段。既然一个进程映射一个包含读/写页节对象的写时复制视图，而不是在当时创建一个进程私有的副本视图来映射，那么内存管理器将推迟创建该页的私有副本，直到该页被写入。如下图10-3所示，在2个进程之间共享3个页面，每个页面都被标记为写时复制，但是这2个进程都没有尝试修改这些页中的任何数据。

![](https://bbs.kanxue.com/upload/attach/201509/533967_9b66489a344aecbd.jpg)

If a thread in either process writes to a page, a memory management fault is generated. The memory manager sees that the write is to a copy-on-write page, so instead of reporting the fault as an access violation, it allocates a new read/write page in physical memory, copies the contents of the original page to the new page, updates the corresponding page-mapping information (explained later in this chapter) in this process to point to the new location, and dismisses the exception, thus

causing the instruction that generated the fault to be reexecuted. This time, the write operation succeeds, but as shown in Figure 10-4, the newly copied page is now private to the process that did the writing and isn’t visible to the other process still sharing the copy-on-write page. Each new process that writes to that same shared page will also get its own private copy.

假设任一进程中的一个线程向其中一个页面写入，生成一个内存管理错误。内存管理器知道写操作针对的是一个写时复制的页，它不会向进/线程报告一个非法访问错误，而是在物理内存中分配一个新的读/写页，然后将原始页的内容拷贝到新的页，并在该进/线程中更新相应的页面映射信息（本章稍后解释）来指向新的物理页位置，内存管理器还会驳回（dismisses）前面生成的异常，因而使得生成错误的指令重新被执行。这次写操作就会成功，如如下图10-4所示，执行写操作的新副本页现在属于该进/线程私有，并且对于其它仍旧共享原始写时复制页的进程不可见。每个写入同一个写时复制共享页的新进程，都将获得其自身的私有副本页来实际写入。

![](https://bbs.kanxue.com/upload/attach/201509/533967_85e50b030ff07eea.jpg)

One application of copy-on-write is to implement breakpoint support in debuggers. For example, by default, code pages start out as execute-only. If a programmer sets a breakpoint while debugging a program, however, the debugger must add a breakpoint instruction to the code. It does this by first changing the protection on the page to `PAGE_EXECUTE_READWRITE` and then changing the instruction stream. Because the code page is part of a mapped section, the memory manager creates a private copy for the process with the breakpoint set, while other processes continue using the unmodified code page.

Copy-on-write is one example of an evaluation technique known as lazy evaluation that the memory manager uses as often as possible. Lazy-evaluation algorithms avoid performing an expensive operation until absolutely required—if the operation is never required, no time is wasted on it.

To examine the rate of copy-on-write faults, see the performance counter Memory: Write Copies/sec.

写时复制的一个应用就是实现调试器支持的断点功能。例如默认情况下，被调试进程的代码页一开始是仅执行的。然而如果一个程序员在调试程序时设置了一个断点，调试器必须加入一个断点指令到代码中。这是通过首先将目标代码页的内存保护选项改为`PAGE_EXECUTE_READWRITE`（译注：请参考表10-2），然后改变指令流做到的。由于代码页是映射节（译注：这应该是通过节视图映射到可执行文件中的类似.text节实现）的一部分，内存管理器会为调试器进程创建一个设置了断点的代码页私有副本（在调试器向代码页写入断点指令时），而其它进程可以继续共享未修改的原始代码页。写时复制是一种内存管理器优先常用的，被称为`lazy evaluation`（暂译为惰性求值或惰性计算）的求值技术例子之一。惰性求值算法试图避免执行开销很大的操作，除非绝对必要——如果不是必须的，就尽可能节省计算时间。要审查写时复制频率，参见性能计数器内存：Write Copies/sec.

### Address Windowing Extensions

Although the 32-bit version of Windows can support up to 64 GB of physical memory (as shown in Table 2-2 in Part 1), each 32-bit user process has by default only a 2-GB virtual address space. (This can be configured up to 3 GB when using the increaseuserva BCD option, described in the upcoming section “User Address Space Layout.”) An application that needs to make more than 2 GB (or 3 GB) of data easily available in a single process could do so via file mapping, remapping a part of its address space into various portions of a large file. However, significant paging would be involved upon each remap.

For higher performance (and also more fine-grained control), Windows provides a set of functions called Address Windowing Extensions (AWE). These functions allow a process to allocate more physical memory than can be represented in its virtual address space. It then can access the physical memory by mapping a portion of its virtual address space into selected portions of the physical memory at various times.

### 地址窗口化扩展

尽管Windows的32位版本能够支持最多64GB的物理内存（如本书上册表2-2所示），每个32位用户模式进程默认情况只有一个2GB的虚拟地址空间。（使用 `increaseuserva` BCD 选项时，可以配置成最多3GB，本章后面的“用户地址空间布局”小节即将对此进行论述。）一个需要使用或产生多于2GB（或3GB）数据的应用程序通过文件映射，即可在单一进程中轻易获得这样的空间分配，重新将自身地址空间的一部分映射到一个大型文件的各个部分。然而，每个重映射过程都将涉及显式分页。为了提高性能（以及更多细粒度的控制），Windows提供了一组叫做地址窗口化扩展（AWE）的函数。这些函数允许分配给一个进程的物理内存超过能够代表进程自身虚拟地址空间的物理内存。（译注：AWE函数的原型声明在 `winbase.h` 头文件中。这些 API 允许程序访问的物理内存总量大于它的线性地址空间对其限定的物理内存量；“线性地址”是英特尔文档中的用语，微软将其称为“虚拟地址”，在处理器启用分段与分页的多数场景下，将虚拟地址进行第一阶段分段，得到线性地址；将线性地址进行第二阶段分页，得到物理地址，这就是地址翻译的过程；在仅启用分页的情况下，一般可以认为虚拟地址就是线性地址。）

接着，在不同时间点上，进程都可以通过将自身虚拟地址空间的一部分映射到选定部分的物理内存，来对其进行访问。

（译注：`increaseuserva` BCD 选项的设置方法如下图所示）

![](https://bbs.kanxue.com/upload/attach/201509/533967_6f4dcec7bbb25ac8.jpg)

Allocating and using memory via the AWE functions is done in three steps:

1. Allocating the physical memory to be used. The application uses the Windows functions `AllocateUserPhysicalPages` or `AllocateUserPhysicalPagesNuma`. (These require the Lock Pages In Memory user right.)

2. Creating one or more regions of virtual address space to act as windows to map views of the physical memory. The application uses the Win32 `VirtualAlloc`, `VirtualAllocEx`, or `VirtualAllocExNuma` function with the `MEM_PHYSICAL` flag.

3. The preceding steps are, generally speaking, initialization steps. To actually use the memory, the application uses `MapUserPhysicalPages` or `MapUserPhysicalPagesScatter` to map a portion of the physical region allocated in step 1 into one of the virtual regions, or windows, allocated in step 2.

通过AWE函数分配和使用内存需要如下3个步骤：

1. 分配要使用的物理内存。应用程序使用Windows函数`AllocateUserPhysicalPages` 或 `AllocateUserPhysicalPagesNuma`（这要求调用方具有内存中锁定页面的用户权限）。

（译注：这是由于后文将提及的：通过AWE分配的物理内存永远不会换出到磁盘，必须锁定在内存中，因此调用AWE函数的应用程序需要获得“锁定内存页”权限）

2. 创建一个或者更多的虚拟地址空间区域，作为要映射物理内存视图的窗口（译注：原文中的windows为小写，因此假定作者原意指窗口，而非Windows操作系统）。应用程序使用Win32 API 函数`VirtualAlloc`，`VirtualAllocEx`，或者 `VirtualAllocExNuma`，并且指定 `MEM_PHYSICAL` 标志。
