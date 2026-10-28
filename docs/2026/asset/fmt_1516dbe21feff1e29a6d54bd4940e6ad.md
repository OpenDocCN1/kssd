
For a process to grow beyond 2 GB of address space, the image file must have the `IMAGE_FILE_LARGE_ADDRESS_AWARE` flag set in the image header. Otherwise, Windows reserves the additional address space for that process so that the application won't see virtual addresses greater than `0x7FFFFFFF`. Access to the additional virtual memory is opt-in because some applications have assumed that they'd be given at most 2 GB of the address space. Since the high bit of a pointer referencing an address below 2 GB is always zero, these applications would use the high bit in their pointers as a flag for their own data, clearing it, of course, before referencing the data. If they ran with a 3-GB address space, they would inadvertently truncate pointers that have values greater than 2 GB, causing program errors, including possible data corruption. You set this flag by specifying the linker flag `/LARGEADDRESSAWARE` when building the executable. This flag has no effect when running the application on a system with a 2-GB user address space.

为了让一个进程能够增长到超过 2 GB 的地址空间，在其映像文件的映像头部（译注：通常是 PE 文件头）中必须设置了 `IMAGE_FILE_LARGE_ADDRESS_AWARE` 标志。否则，对于该进程，Windows 将“留住”（reserves）额外的地址空间，以至于超过 `0x7FFFFFFF` 以上的虚拟地址对于该应用程序而言是不可见的。由于某些应用程序假设了它们仅被给予最多 2 GB 的地址空间，因此对额外虚拟内存的访问完全是可选（opt-in）的。既然一个引用低于 2 GB 地址的指针其最高位始终为 0，这些应用程序会将它们指针中的最高位用作一个标记，来标识其自身数据的边界；当然，在引用这些应用数据前，必须将最高位“清位”（clearing it）。（译注：由于一个低于 2 GB 的地址，如 `0x7FFFFFFF`，其二进制形式为 `01111111111111111111111111111111`，因此其最高位始终为 0；而 2 GB 以上的任何一个地址，例如 `0x80000000`，其二进制形式为 `10000000000000000000000000000000`，其最高位始终为 1）

就上述场景而言，如果这些应用数据（和代码）运行在一个 3 GB 的地址空间中，它们可能会无意间截断其值大于 2 GB 的指针，导致程序出错，包括可能的数据损坏。

你可以通过在构建相应可执行文件时，指定 `/LARGEADDRESSAWARE` 链接器标志，配置应用程序使用 3 GB 的地址空间。但是，当在一个配置成使用 2 GB 用户地址空间的系统上运行该应用程序时，此链接器标志将不起作用。（译注：也就是说，必须同时启用 `increaseuserva` BCD 启动选项，以及指定 `/LARGEADDRESSAWARE` 链接器标志；关于启用前者，请参考第一部分译文，这里不再赘述）

Several system images are marked as large address space aware so that they can take advantage of systems running with large process address spaces. These include:

- `Lsass.exe`  The Local Security Authority Subsystem
- `Inetinfo.exe`  Internet Information Server
- `Chkdsk.exe`  The Check Disk utility
- `Smss.exe`  The Session Manager
- `Dllhst3g.exe`  A special version of `Dllhost.exe` (for COM+ applications)
- `Dispdiag.exe`  The display diagnostic dump utility
- `Esentutl.exe`  The Active Directory Database Utility tool

下面这些系统映像被标记为察觉到大地址空间，以便它们能够充分利用以大进程地址空间运行的系统优势：

- `Lsass.exe`  本地安全授权子系统
- `Inetinfo.exe`  英特网信息服务器
- `Chkdsk.exe`  Check Disk 实用工具
- `Smss.exe`  会话管理器
- `Dllhst3g.exe`  `Dllhost.exe` 的特别版本（用于 COM+ 应用程序）
- `Dispdiag.exe`  显示诊断转储实用工具
- `Esentutl.exe`  活动目录数据库实用工具

## EXPERIMENT: Checking If an Application Is Large Address Aware

## 实验：检查应用程序是否启用了大地址空间

You can use the Dumpbin utility from the Windows SDK to check other executables to see if they support large address spaces. Use the `/HEADERS` flag to display the results. Here's a sample output of Dumpbin on the Session Manager:

你可以使用 Windows SDK 中的 Dumpbin 实用工具，检查其它的可执行文件是否支持大地址空间。使用 `/HEADERS` 标记能够显示相关的结果。下面这个输出样本是 Dumpbin 对会话管理器进程（`smss.exe`）的检测结果：

```
C:\Program Files\Microsoft SDKs\Windows\v7.1>dumpbin /headers c:\windows\system32\smss.exe

Microsoft (R) COFF/PE Dumper Version 10.00.40219.01

Copyright (C) Microsoft Corporation. All rights reserved.

Dump of file c:\windows\system32\smss.exe

PE signature found

File Type: EXECUTABLE IMAGE

FILE HEADER VALUES

8664 machine (x64)

5 number of sections

4A5BC116 time date stamp Mon Jul 13 16:19:50 2009

0 file pointer to symbol table

0 number of symbols

F0 size of optional header

22 characteristics

Executable

Application can handle large (>2GB) addresses
```

（译注：由于当前发行的 Windows SDK 中仅包含 64 位的 `dumpbin.exe`，因此若要检测 32 位可执行文件，可以通过 PEView 或其它类似的 PE 文件格式查看工具来完成，下面这张截图是使用 PEView 检查 `smss.exe` 是否支持大地址空间的结果：）

![](https://bbs.kanxue.com/upload/attach/201510/533967_b25e6b51c45829db.png)

Finally, because memory allocations using `VirtualAlloc`, `VirtualAllocEx`, and `VirtualAllocExNuma` start with low virtual addresses and grow higher by default, unless a process allocates a lot of virtual memory or it has a very fragmented virtual address space, it will never get back very high virtual addresses. Therefore, for testing purposes, you can force memory allocations to start from high addresses by using the `MEM_TOP_DOWN` flag or by adding a DWORD registry value, `HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management\AllocationPreference`, and setting it to `0x100000`.

最后，由于使用 `VirtualAlloc()`，`VirtualAllocEx()`，以及 `VirtualAllocExNuma()` 的内存分配操作，默认会从虚拟内存的低址段开始分配，然后往高址段增长，所以除非一个进程分配了大量的虚拟内存，或者它有一个非常零碎（分散）的虚拟地址空间，否则它永远都不会回到非常高的虚拟地址上来。故而，出于测试目的，你可以通过使用 `MEM_TOP_DOWN` 标志，强制内存分配操作从高址段开始并且往低址段增长；或者添加一个名为 `AllocationPreference` 的 DWORD 类型注册表键值，路径为：`HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management`，然后将该键值的数据设置为 `0x100000`。

Figure 10-9 shows two screen shots of the TestLimit utility (shown in previous experiments) leaking memory on a 32-bit Windows machine booted with and without the `increaseuserva` option set to 3 GB.

下图 10-9 中的 2 个屏幕截图展示了在一个 32 位 Windows 机器上分别开启 `increaseuserva`（3 GB 用户空间）和关闭该选项然后启动，使用 TestLimit 实用工具（在前面的实验中用过）执行内存泄露操作的结果。

Note that in the second screen shot, TestLimit was able to leak almost 3 GB, as expected. This is only possible because TestLimit was linked with `/LARGEADDRESSAWARE`. Had it not been, the results would have been essentially the same as on the system booted without `increaseuserva`.

注意第二个屏幕截图，其输出表明 TestLimit 能够泄露 3 GB，正如预期的一样。这还需要 TestLimit 应用程序以 `/LARGEADDRESSAWARE` 链接器标志来构建才行。如果没有指定该标记，那么测试的结果将基本上与关闭 `increaseuserva` 启动选项时的相同。（即第一个截图）

![](https://bbs.kanxue.com/upload/attach/201510/533967_dce94e09bc52bb1e.png)

## x86 System Address Space Layout

## x86 系统地址空间布局

The 32-bit versions of Windows implement a dynamic system address space layout by using a virtual address allocator (we'll describe this functionality later in this section). There are still a few specifically reserved areas, as shown in Figure 10-8. However, many kernel-mode structures use dynamic address space allocation. These structures are therefore not necessarily virtually contiguous with themselves. Each can easily exist in several disjointed pieces in various areas of system address space. The uses of system address space that are allocated in this way include:

- Nonpaged pool
- Special pool
- Paged pool
- System page table entries (PTEs)
- System mapped views
- File system cache
- File system structures (metadata)
- Session space

32 位版本的 Windows 通过使用一个虚拟地址分配器（本节稍后将讨论此功能）实现了一个动态的系统地址空间布局。如图 10-8 所示，系统地址空间中仍然有一些专门的保留区域。然而，许多内核模式数据结构使用动态地址空间来分配。因此这些数据结构自身不一定非得几乎连续相邻。每一个都能够轻易地在系统地址空间的各个区域中，存在一些不连续的碎片。被以这种分配方式来使用系统地址空间的包括：

- 非分页池
- 专用（Special）池（译注：关于专用/特殊池的简介请参考译文的“内核模式堆[系统内存池]”一节；详细讨论请参考本书下册第14章）
- 分页池
- 系统页表条目（PTEs）
- 系统映射视图
- 文件系统缓存
- 文件系统结构（元数据）
- 会话空间

## x86 Session Space

## x86 会话空间

For systems with multiple sessions, the code and data unique to each session are mapped into system address space but shared by the processes in that session. Figure 10-10 shows the general layout of session space.

对于有多个会话的系统而言，每个会话独有的代码与数据被映射到系统地址空间，但是被该会话中的所有进程共享。下图 10-10 展示了会话空间的总体布局：

![](https://bbs.kanxue.com/upload/attach/201510/533967_1c1fa546f7ebed4d.png)

The sizes of the components of session space, just like the rest of kernel system address space, are dynamically configured and resized by the memory manager on demand.

会话空间中各种组件的大小，就如同内核系统地址空间的其余部分一样，由内存管理器按需动态配置和重新调整其大小。

## EXPERIMENT: Viewing Sessions

## 实验：查看会话
