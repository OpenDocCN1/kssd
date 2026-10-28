
One extremely common use for reserving a large space and committing portions of it as needed is the user-mode stack for each thread. When a thread is created, a stack is created by reserving a contiguous portion of the process address space. (`1 MB` is the default; you can override this size with the `CreateThread` and `CreateRemoteThread` function calls or change it on an imagewide basis by using the `/STACK` linker flag.) By default, the initial page in the stack is committed and the next page is marked as a guard page (which isn’t committed) that traps references beyond the end of the committed portion of the stack and expands it.

对于保留一大段地址空间并且按需提交其中一部分来使用，有一个极其常见的例子——每个线程的用户模式栈。当一个线程被创建时，通过在进程地址空间中预留出一段连续相邻的部分来创建栈。（默认是`1MB`，你可以通过`CreateThread`与`CreateRemoteThread`函数调用覆盖默认值，或者使用`/STACK`链接器标志，在整个映像文件的基础上更改默认值）。默认情况下，栈中的初始页面是提交的，并且接下来的页面被标记为保护页（非提交的），引用超出栈中提交部分结尾的页面时，通过陷阱分发机制执行特定的异常处理程序来扩展当前提交的页面。（译注：本句请参考原文，有更好的译法欢迎提出或指正）

### EXPERIMENT: Reserved vs. Committed Pages

The `TestLimit` utility (which you can download from the Windows Internals book webpage) can be used to allocate large amounts of either reserved or private committed virtual memory, and the difference can be observed via Process Explorer. First, open two Command Prompt windows.

Invoke `TestLimit` in one of them to create a large amount of reserved memory:

### 实验：保留的与提交的页面

`TestLimit`工具（您可以从本书的web页面下载）可以用于分配大量的保留，或者私有提交虚拟内存，这两者的差异可以通过进程浏览器（`Process Explorer`）来观察。首先，打开2个命令行提示符窗口。在其中一个调用`TestLimit`命令创建一段大量的保留内存：
```
[FONT="微软雅黑"][SIZE="4"]C:\temp>testlimit -r 1 -c 800

Testlimit v5.2 - test Windows limits

Copyright (C) 2012 Mark Russinovich

Sysinternals - w1c3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4&6M7$3W2F1N6r3g2J5L8X3q4D9M7#2)9J5k6h3y4G2L8b7`.`.

Process ID: 1544

Reserving private bytes 1 MB at a time ...

Leaked 800 MB of reserved memory (800 MB total leaked). Lasterror: 0

The operation completed successfully.[/SIZE][/FONT]
```

In the other window, create a similar amount of committed memory:

在另一个窗口，创建相同数量的提交内存：
```
[FONT="微软雅黑"][SIZE="4"]C:\temp>testlimit -m 1 -c 800

Testlimit v5.2 - test Windows limits

Copyright (C) 2012 Mark Russinovich

Sysinternals - wbfbK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4&6M7$3W2F1N6r3g2J5L8X3q4D9M7#2)9J5k6h3y4G2L8b7`.`.

Process ID: 2828

Leaking private bytes 1 KB at a time ...

Leaked 800 MB of private memory (800 MB total leaked). Lasterror: 0

The operation completed successfully.[/SIZE][/FONT]
```

Now run Task Manager, go to the Processes tab, and use the Select Columns command on the View menu to include Memory—Commit Size in the display. Find the two instances of TestLimit in the list. They should appear something like the following figure.

现在，打开Windows任务管理器，切换到进程选项卡，在主菜单的“查看”->“选择列”，勾选“提交大小”来显示提交的内存。在进程列表中找到2个`TestLimit`实例，应该如下图所示：

![](https://bbs.kanxue.com/upload/attach/201509/533967_fe455960e13e0f4d.jpg)

Task Manager shows the committed size, but it has no counters that will reveal the reserved memory in the other TestLimit process.

Finally, invoke Process Explorer. Choose View, Select Columns, select the Process Memory tab, and enable the Private Bytes and Virtual Size counters. Find the two TestLimit processes in the main display:

任务管理器显示出提交大小，但是没有计数器揭示关于保留内存的信息。最后，我们调用进程浏览器，在主菜单的“查看”->“选择列”，切换到进程内存选项卡，然后勾选`Private Bytes`与`Virtual Size`计数器的复选框。在主界面中找出2个`TestLimit`进程：

![](https://bbs.kanxue.com/upload/attach/201509/533967_a9eba9eda237dc59.jpg)

Notice that the virtual sizes of the two processes are identical, but only one shows a value for Private Bytes comparable to that for Virtual Size. The large difference in the other TestLimit process (process ID 1544) is due to the reserved memory.

注意，2个`TestLimit`进程的`Virtual Size`完全相同，但是只有其中一个显示出与`Virtual Size`大小接近的`Private Bytes`值。进程ID为1544的`TestLimit`进程的`Private Bytes`与`Virtual Size`差别如此之大，是因为此进程创建保留的内存，因为还没有实际提交使用，因此它不会被`Private Bytes`计数器考虑在内。

（译注：前文提到，“保[预]留内存是相对廉价的操作，因为它只消耗非常少的内存”，也许就是它的`Private Bytes`只有2.8M，而不是822M的原因。而对于`Virtual Size`计数器，不论是保留的还是提交的，都会将其考虑在内，这一点是需要注意的。下面提供在我自己机器上的实验截图，结果基本与上面描述的类似：）

![](https://bbs.kanxue.com/upload/attach/201509/533967_dfc786c60f9909ab.jpg)

![](https://bbs.kanxue.com/upload/attach/201509/533967_62ff46493d80b6a8.jpg)

The same comparison could be made in Performance Monitor by looking at the Process | Virtual Bytes and Process | Private Bytes counters.

可以在性能监视器中，查看 Process | Virtual Bytes and Process | Private Bytes 计数器，作出同样的比较。

### Commit Limit

On Task Manager’s Performance tab, there are two numbers following the legend Commit. The memory manager keeps track of private committed memory usage on a global basis, termed commitment or commit charge; this is the first of the two numbers, which represents the total of all committed virtual memory in the system.

There is a systemwide limit, called the system commit limit or simply the commit limit, This limit corresponds to the current total size of all paging files, plus the amount of RAM that is usable by the operating system.

This is the second of the two numbers displayed as Commit on Task Manager’s Performance tab. The memory manager can increase the commit limit automatically by expanding one or more of the paging files, if they are not already at their configured maximum size.

Commit charge and the system commit limit will be explained in more detail in a later section.

### 提交限制

在任务管理器的性能选项卡，“提交（GB）”后面跟着2个数值。内存管理器在全局基础上，跟踪记录私有提交内存的使用情况，这被称为`commitment`或`commit charge`，即第一个数值，它代表系统中所有提交的虚拟内存总合。还有一个系统级限制，叫做`system commit limit`或者简称`commit limit`，这个限制对应当前的所有分页文件总大小，增加可被操作系统使用的物理内存总量，即第二个数值。内存管理器可以通过扩展一个或多个分页文件，自动增加`commit limit`上限。（如果它们尚未配置最大上限）。本章后面部分将详细解释`Commit charge`与`system commit limit`。

（译注：经过实践发现，能为进程分配的虚拟内存大小受限于“`system commit limit`”的大小，即系统探测到的物理内存大小加上分页文件的大小，例如，在32位系统上，假设识别到的物理内存总量为3GB，并且设置了初始大小为2GB的分页文件，那么`system commit limit`的值为5GB；如果所有当前运行的进程被分配的虚拟内存总合达到这个限制，新的进程将无法运行，windows会给出页面文件太小，无法执行应用程序的错误提示；例如，运行`vmware`至少需要2GB的空闲`system commit`，以前的5GB上限为例，如果当前的`system commit`已经使用了3GB，那么将无法运行`vmware`，即便勉强启动程序，系统的响应速度也会变得异常缓慢。此时可以通过增大页面文件的默认最小值来扩大“交换区”的大小，好让更多当前用不到的物理内存页能够交换到磁盘[因为磁盘的页面文件交换区已经扩大]，从而给`vmware`的进驻内存准备更多的空间。配置的方法：在计算机上右击 ->属性 -> 高级系统设置 ->高级选项卡 -> 单击性能栏目的“设置”，再切换到“高级选项卡”，单击“虚拟内存”栏目的“更改”。虚拟内存的用词可能会产生误导，实际上它就是用来配置磁盘页面文件大小的。

使用`sysinternal`的进程浏览器，单击最上方工具栏右侧的第二个矩形区域，就能显示`system commit`的统计数据。另外还要注意一点：

由于32位`windows`的内核代码限制了系统识别的物理内存上限为4GB，再扣掉为硬件寻址[CPU芯片内的核心显卡共享系统内存，外围I/O设备自带的硬件缓冲区映射系统内存等等]保留的地址空间，实际能使用的仅有3GB左右的物理内存，下图给出为了寻址各种总线，总线控制器，以及充当集成显卡显存，而保留的内存范围例子：

![](https://bbs.kanxue.com/upload/attach/201509/533967_06d19855b18d8140.jpg)

因此无法通过添加物理内存的容量来提高`system commit`上限，只能通过增大页面交换文件的方式，而推荐设置的页面文件大小通常是识别出的物理内存的1.5~2倍；

这意味着32位`windows`的页面文件大小为4.5~6GB，则`system commit limit`的值就为7.5~9GB；相反，64位`windows`内核代码支持CPU的48位地址总线模式，可以寻址16TB地址空间[Intel X64体系结构当前使用48位地址总线，理论上支持256TB地址空间，但是64位windows仅使用其中的44位，因此只能寻址16TB物理内存]，以及通过各种非官方渠道开启物理地址扩展[PAE补丁]的32位`windows`，可以使用36位地址空间，也就是能够识别64GB物理内存，所以在64位`windows`和32位 PAE `windows`上，可以通过添加物理内存的容量来提高`system commit`上限，从而能够“同时”运行更多应用程序；一般而言，增大物理内存会比增大页面交换文件来的有效。）

### Locking Memory

In general, it’s better to let the memory manager decide which pages remain in physical memory.

However, there might be special circumstances where it might be necessary for an application or device driver to lock pages in physical memory. Pages can be locked in memory in two ways:

■ Windows applications can call the `VirtualLock` function to lock pages in their process working set. Pages locked using this mechanism remain in memory until explicitly unlocked or until the process that locked them terminates. The number of pages a process can lock can’t exceed its minimum working set size minus eight pages. Therefore, if a process needs to lock more pages, it can increase its working set minimum with the `SetProcessWorkingSetSizeEx` function (referred to in the section “Working Set Management”).

■ Device drivers can call the kernel-mode functions `MmProbeAndLockPages`, `MmLockPagableCodeSection`, `MmLockPagableDataSection`, or
