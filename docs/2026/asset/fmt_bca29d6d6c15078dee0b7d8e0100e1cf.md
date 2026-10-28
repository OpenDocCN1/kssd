
对于每一类可设限的系统虚拟地址使用情况，`MiSystemVaTypeCountLimit`数组包含了相应的限制。目前，内存管理器仅允许对某些虚拟地址类型做出限制，而且，它提供在系统运行期间，使用未文档化（未公开）的系统调用来动态地设限的能力。（可以通过注册表来设置这些限制，请参考 [948K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2K6k6r3&6Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8W2)9J5k6s2g2K6i4K6u0r3L8r3W2T1M7X3q4J5P5g2)9J5c8X3u0T1z5o6M7H3z5o6R3H3i4K6t1^5g2W2y4Q4x3X3f1^5y4g2)9J5z5g2)9J5k6h3q4K6M7s2R3`.）](<https://bbs.kanxue.com/elink@3d1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2K6k6r3&6Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8W2)9J5k6s2g2K6i4K6u0r3L8r3W2T1M7X3q4J5P5g2)9J5c8X3u0T1z5o6M7H3z5o6R3H3i4K6t1^5g2W2y4Q4x3X3f1^5y4g2)9J5z5g2)9J5k6h3q4K6M7s2S2Q4c8f1k6Q4b7V1y4Q4z5o6V1`.)>）。表 10-9 标记了能够设限的地址类型。你可以使用 Winsider Seminars & Solutions 的 MemLimit 工具（[fd9K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4N6A6L8Y4y4A6k6r3g2J5M7%4y4Q4x3X3g2U0L8$3#2Q4x3V1k6@1L8$3!0D9M7#2)9J5c8X3#2W2L8h3I4A6L8h3W2@1i4K6u0W2K9s2c8E0L8l9`.`.](<https://bbs.kanxue.com/elink@8ccK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4N6A6L8Y4y4A6k6r3g2J5M7%4y4Q4x3X3g2U0L8$3#2Q4x3V1k6@1L8$3!0D9M7#2)9J5c8X3#2W2L8h3I4A6L8h3W2@1i4K6u0W2K9s2c8E0L8l9`.`.)>）来查询和对这些类型做出不同的限制，还可以查看当前和峰值的虚拟地址空间使用情况。通过该工具的`-q`标志，你可以查询这些信息：

```
[FONT="微软雅黑"][SIZE="4"]C:\ >[B]memlimit.exe -q[/B]

MemLimit v1.00 - Query and set hard limits on system VA space consumption

Copyright (C) 2008 Alex Ionescu

413K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3q4D9k6i4S2Q4x3X3c8A6L8$3&6W2M7$3y4#2i4K6u0W2j5$3!0E0

System Va Consumption:

[B]Type[/B]                               [B]Current[/B]                              [B]Peak[/B]                            [B]Limit[/B]

Non Paged Pool               102400 KB                         0 KB                            0 KB

Paged Pool                      59392 KB                           83968 KB                    0 KB

System Cache                 534528 KB                        536576 KB                   0 KB

System PTEs                   73728 KB                          75776 KB                     0 KB

Session Space                 75776 KB                           90112 KB                    0 KB[/SIZE][/FONT]
```

As an experiment, use the following command to set a limit of 100 MB for paged pool:

作为一次实战，使用下面的命令来设置可分页池的上限为 100 MB：

```
[FONT="微软雅黑"][SIZE="4"]memlimit.exe -p 100M[/SIZE][/FONT]
```

And now try running the `testlimit –h` experiment from Chapter 3 (in Part 1) again, which attempted to create 16 million handles. Instead of reaching the 16 million handle count, the process will fail, because the system will have run out of address space available for paged pool allocations.

现在，试着运行第三章的实验中使用过的`testlimit`工具，添加`-h`选项，它会尝试创建 16000000 （16 million）个句柄。此时`testlimit`进程会输出创建失败信息，而不是真的到达 16000000 个句柄数，因为系统将耗尽可分页池中，可用于分配的地址空间。

## System Virtual Address Space Quotas（系统虚拟地址空间配额）

The system virtual address space limits described in the previous section allow for limiting systemwide virtual address space usage of certain kernel components, but they work only on 32-bit systems when applied to the system as a whole. To address more specific quota requirements that system administrators might have, the memory manager also collaborates with the process manager to enforce either systemwide or user-specific quotas for each process.

前一节讨论的系统虚拟地址空间限制，虽然允许对某些内核组件的系统级虚拟地址空间使用率设限，但是在 32 位系统上，只有此一机制被应用到整个系统时，才能正常工作。为了解决系统管理员更多的具体配额需求，内存管理器与进程管理器联手协作，来强制实施每个进程的“系统级”或“用户相关”配额（限制）。

The `PagedPoolQuota`, `NonPagedPoolQuota`, `PagingFileQuota`, and `WorkingSetPagesQuota` values in the `HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management` key can be configured to specify how much memory of each type a given process can use. This information is read at initialization, and the default system quota block is generated and then assigned to all system processes (user processes will get a copy of the default system quota block unless per-user quotas have been configured as explained next).

注册表键`HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management`下的`PagedPoolQuota`，`NonPagedPoolQuota`，`PagingFileQuota`，以及`WorkingSetPagesQuota`键值，其数据可以被配置，对于一个给定的进程，分别指定它能够使用每一类地址空间中的内存上限。系统在初始化时读取这些信息，并且生成默认的系统配额块（system quota block），然后分配给所有系统进程（除非已经配置了每用户的配额——稍后解释——否则用户进程将得到一份“默认系统配额块”的副本）。

To enable per-user quotas, subkeys under the registry key `HKLM\SYSTEM\CurrentControlSet\Session Manager\Quota System` can be created, each one representing a given user SID. The values mentioned previously can then be created under this specific SID subkey, enforcing the limits only for the processes created by that user. Table 10-10 shows how to configure these values, which can be configured at run time or not, and which privileges are required.

要启用每用户的配额，可以在注册表键`HKLM\SYSTEM\CurrentControlSet\Session Manager\Quota System`下面，创建相应的子键，每个子键代表一个给定用户的 SID 。（译注：系统上的每一个帐户都有一个关联的 SID，例如，系统内置帐户 `NT AUTHORITY\SYSTEM` 的 SID 为 `S-1-5-18`；`NT AUTHORITY\LOCAL SERVICE` 帐户的 SID 为 `S-1-5-19`；而用户创建的帐户，如 `user1` ，其 SID 类似 `S-1-5-21-2263621008-420926174-3264184512-1000` 这种格式；作者的意思就是要用这些 SID 作为子键的名称）。然后，就可以在这个特定的 SID 子键右侧，创建前文提到的四类键值（分别代表相应类型的内核虚拟地址空间），从而仅对由该用户创建的进程（例如，以 `user1` 帐户运行的 `explorer.exe` 资源管理器和它创建的子进程），在这些地址空间的使用上实施限制。下表 10-10 向你展示如何配置这些键值的数据；是否可在运行时配置；以及要使配置生效所需的权限：

![](https://bbs.kanxue.com/upload/attach/201601/533967_7d22430796f6b32f.png)

![](https://bbs.kanxue.com/upload/attach/201601/533967_50cc568fcc8bb0f0.png)

## User Address Space Layout（用户地址空间布局）

Just as address space in the kernel is dynamic, the user address space is also built dynamically—the addresses of the thread stacks, process heaps, and loaded images (such as DLLs and an application's executable) are dynamically computed (if the application and its images support it) through a mechanism known as Address Space Layout Randomization, or ASLR.

正如内核地址空间是动态的那样，用户地址空间也是动态建立的——线程栈地址，进程堆，以及加载的映像（例如 DLL 和一个应用程序的可执行文件），都是通过一个被称为地址空间布局随机化（ASLR）的机制，来动态计算的（假设应用程序及其映像文件支持这种技术）。

At the operating system level, user address space is divided into a few well-defined regions of memory, shown in Figure 10-14. The executable and DLLs themselves are present as memory mapped image files, followed by the heap(s) of the process and the stack(s) of its thread(s). Apart from these regions (and some reserved system structures such as the `TEB`s and `PEB`), all other memory allocations are run-time dependent and generated. ASLR is involved with the location of all these run-timedependent regions and, combined with DEP, provides a mechanism for making remote exploitation of a system through memory manipulation harder to achieve. Since Windows code and data are placed at dynamic locations, an attacker cannot typically hardcode a meaningful offset into either a program or a system-supplied DLL.

在操作系统层级，用户地址空间被划分为数个明确定义好的内存区域，如图 10-14 所示。可执行文件与 DLL 文件，它们自身都作为内存映射映像存在，接着是进程堆及其线程的栈。除了这些区域外（以及某些保留的系统结构，如“`TEB`s”——线程环境块，和“`PEB`s”——进程环境块；因为同样运行在用户模式下的映像加载器和子系统 DLL 需要向 PEB 与 TEB 中写入信息，所以这2个环境块必须位于用户空间，内核空间在用户模式下不可写），所有其它的内存分配都依赖于运行时（动态）生成。ASLR 参与所有这些运行时相关区域的位置（确定），它与 DEP （译注：数据执行保护。请参考第一部分译文）相结合——这样就提供了一种保护机制，使得通过内存操纵来远程利用一个系统（的漏洞）更难实现。（译注：这里指的就是远程执行代码的缓冲区溢出攻击，例如使用 metasploit 通过网络发送 shellcode 来取得一个从受害机器到攻击主机的反弹 shell，而且还具备管理员，或者 root 权限。这是由于，被攻击的有缺陷的应用程序同样是以这些特权级别运行的）。

既然 Windows 代码和数据被放置在动态区域，攻击者通常无法将一个有意义的偏移量硬编码至一个程序或系统提供的 DLL 里。

![](https://bbs.kanxue.com/upload/attach/201601/533967_3da2e04d508308f6.png)

## EXPERIMENT: Analyzing User Virtual Address Space（实验：分析用户虚拟地址空间）

The `VMMap` utility from Sysinternals can show you a detailed view of the virtual memory being utilized by any process on your machine, divided into categories for each type of allocation, summarized as follows:

- **Image** — Displays memory allocations used to map the executable and its dependencies (such as dynamic libraries) and any other memory mapped image (portable executable format) files
- **Private** — Displays memory allocations marked as private, such as internal data structures, other than the stack and heap
