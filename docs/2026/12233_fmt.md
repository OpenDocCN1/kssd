
# [原创]利用任意物理读写驱动来加载自己的驱动

早期许多漏洞驱动未对`MmMapIoSpace`施加严格的用户态访问限制，导致攻击者可借此读写物理内存（如利用 PFN 页表可直接将物理内存转换为虚拟内存）。为此，Windows 后续封堵了这一利用路径，将 PFN 页表设为不可映射，强行访问会直接触发蓝屏（BSOD）。

使用VDM。

[VDM介绍](https://bbs.kanxue.com/elink@c85K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5h3y4C8i4K6u0W2k6h3&6Y4K9h3&6W2k6i4u0A6L8X3N6Q4x3V1k6T1L8r3!0Y4i4K6u0r3x3o6q4Q4x3V1j5I4x3g2)9J5c8U0t1H3x3U0m8Q4x3V1j5)

其本质为利用物理内存读写驱动来找到一个用户态可以调用传给内核态的函数 修改写入一段shellcode，来实现任意内核代码执行。但是由于其本质还是类似.data ptr hook (容易被检测，并且效率不高),而且作者最后利用它写的[physmeme](https://bbs.kanxue.com/elink@229K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6m8e0p5g2t1b7f1y4w2M7%4m8Q4x3V1k6H3K9s2W2K6L8h3g2E0k6b7`.`.)只是一个无模块驱动加载器 我认为其被检测的可能性还是很大的。

## 2.直接修改驱动校验函数

即为我想要说的方法，如果我的目的只是加载我自己的驱动，为什么要这么复杂呢？

这里介绍`SystemSuperfetchInformation` 原帖：[SuperFetch Query 超能力 - vegvisir](https://bbs.kanxue.com/elink@445K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6$3x3h3D9I4L8X3N6X3M7W2)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3M7%4g2H3k6i4u0X3k6i4c8U0K9s2q4#2k6i4u0&6i4K6u0V1M7%4g2H3k6i4u0H3L8%4N6W2M7W2)9J5c8R3`.`.)

**作者**：Viking | **发布日期**：2024年8月25日 | **阅读时间**：约 40 分钟

**标签**：`Windows` `漏洞利用开发 (Exploit Dev)` `提权 (Privesc)` `恶意软件开发 (Maldev)`

在之前的博客文章《修复（Windows内部）`Meminfo.exe`》中，我们深入探讨了 Windows 内部结构手册中基于“`FileInfo`请求”的`Meminfo.exe`工具。本文我建议你看看另一种被称为 “`SuperFetchQuery`” 的请求类型，它在诸如红队行动/提权、渗透测试、漏洞利用开发或恶意软件开发（Maldev）等特定场景中非常有用。让我们一起来看看！

**Superfetch 查询** 和 **fileInfo 请求** 是一种可以让你获取许多有趣的 Windows 系统信息的替代方法，以下是一些具体的使用场景：

**恶意软件开发 (Maldev)**：虚拟机沙箱检测技巧，获取内存布局和与内存映射页面相关的文件名信息。

**红队 / 提权 (Red Team / Privesc)**：将虚拟地址（VA）转换为物理地址（PA），这在 BYOVD（自带漏洞驱动）场景中利用物理内存读/写原语时非常有用。

**渗透测试 (Pentest)**：规避某些检测区域，例如在枚举正在运行的进程列表时。

**漏洞利用开发 (Exploit Dev)**：在需要绕过 KASLR 的场景中获取内核地址泄露。

*(注意：以下内容是在 Microsoft Windows 版本 10.0.19045.4291 上测试的)*

我发现研究 Superfetch 查询最好的工具是 Alex Ionescu 和 Pavel Yosifovich 编写的 Windows Internals 项目中的[MemInfo](https://bbs.kanxue.com/elink@954K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6*7L8$3c8A6j5h3y4G2L8W2)9J5c8W2N6A6L8X3c8G2N6%4y4u0L8Y4c8W2M7X3&6S2L8s2y4Q4x3V1k6@1M7X3g2W2i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3e0h3g2E0d9h3&6X3L8H3`.`.)工具。以下是该工具能实现的功能：

`MemInfo.cpp` 主函数概述（仅标注对本文有用的部分）：

为了检索信息，每个“示例”都分为两个步骤：

1. 构建查询
2. 将查询发送给内核

**步骤 1 - 构建查询**

`PfiBuildSuperfetchInfo()` 函数使用 4 个参数来构建 superfetch 查询：

`SuperfetchInfo`：存储你要发送给内核的请求结构。

`Buffer`：存储从内核接收到的结果。

`Length`：`Buffer` 的大小。

`InfoClass`：你请求的信息类型。

**步骤 2 - 发送查询**

Windows API `NtQuerySystemInformation()` 通过 4 个参数向内核发送 superfetch 查询：

`SystemInformationClass`：表示要检索的系统信息类型（这里设置为 `SystemSuperfetchInformation`）。

`SystemInformation`：指向一个缓冲区，用于接收请求的信息（设置为步骤 1 中准备好的 `SuperfetchInfo`）。

`Length`：`SystemInformation` 的大小。

`ResultLength`：实际需要的信息大小。

*(注意：调用 `NtQuerySystemInformation()` 需要中等完整性级别的进程。)*

现在你已经熟悉了准备和发送 Superfetch 查询的方法。接下来，我们会回顾一些用于“态势感知”等目的的传统方法，并向你展示基于 Superfetch 查询的替代方案。

一个有趣的沙箱检测技巧使用了内存范围，具体可参考 Graham Sutherland 的《虚拟机检测技巧》一文。简而言之，该技巧如下：

读取以下 Windows 注册表路径：

比较不同物理主机和虚拟机的内存资源映射。例如：VirtualBox 或 Hyper-V 都有特定的物理内存分配范围。

使用 Superfetch 替代方案不会触及上述注册表键值，因此避开了某些安全软件的检测区域。我们可以使用超级查询来获取 Windows 当前检测到的**有效物理内存范围**。

修改 Meminfo 项目中的相关代码：

输出示例：

如你所见，“物理内存范围”是`00001000 – 0009f000`，这通常对应 VirtualBox 上的 Win10 虚拟机。

在利用 Windows 内核漏洞时，如果利用原语是物理读写，你需要一种将虚拟地址 (VA) 转换为物理地址 (PA) 的方法。

最古老的技术是读取内核的页表，但微软从 Windows 10 开始已经修补了这一点。

另一种方法是通过读取物理内存 0-0x20000 区域的 DOS“low stub”来找到 CR3 寄存器。但这目前看来有触发各种蓝屏 (BSOD) 的风险（例如 `DRIVER_IRQL_NOT_LESS_OR_EQUAL`, `MEMORY_MANAGEMENT` 等）。

第三种方法是扫描非分页池中名为`Proc`的池标签（用于存储`EPROCESS`）。局限性在于你需要内存泄漏，具备内核任意读能力，且通常需要指针跳跃。

如 Cedric Van Bockhaven 所述，利用 Superfetch API 进行漏洞开发非常稳定且安全。

**优势**：更加稳定（使用官方 Windows API），触发蓝屏死机的风险极低（保持在用户态运行）。

通过调用`SuperfetchPfnQuery()`，我们可以检索`PF_PFN_PRIO_REQUEST`数据结构，其中包含了页帧号 (PFN) 数据库的副本。

我们可以修改`PfiDumpProcessPfnEntry()`函数来进行 VA 到 PA 的转换：

输出示例 (转换 `notepad.exe` 的地址)：

*(可选)*：您可以使用 WinDbg 通过`!vtop`命令或`!db`命令来验证这个物理地址转换的正确性。

枚举运行中进程或查找特定进程 PID 的方法有很多（例如通过读取 LSASS PID 的标准 API）。

在此示例中，你使用 Superfetch 查询来获取当前 Windows 上正在运行的进程信息。通过查询类型`SuperfetchPrivSourceQuery`，你可以提取`PF_PRIVSOURCE_INFO`数据结构。

修改 Meminfo 项目以打印进程名称和 PID：

输出示例：

编写内核提权漏洞时，首先必须在内核中找到进程的`_EPROCESS`结构位置。常用的方法包括：

`PsInitialSystemProcess()`

`PsReferencePrimaryToken()`

`PsLookupProcessByProcessId()`

窃取 Token 的 shellcode

[回复或点赞可查看完整内容](https://bbs.kanxue.com/#quick_reply_form)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2026-4-19 19:46

被Peyriat编辑

，原因：

[#系统内核](https://bbs.kanxue.com/forum-41-1-131.htm)

[#开源分享](https://bbs.kanxue.com/forum-41-1-134.htm)

[#工具脚本](https://bbs.kanxue.com/forum-41-1-137.htm)

[#驱动开发](https://bbs.kanxue.com/forum-41-1-132.htm)
