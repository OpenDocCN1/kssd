
# N 种内核注入 DLL 的思路及实现

> 原文：[`www.pediy.com/kssd/pediy10/75887.html`](https://www.pediy.com/kssd/pediy10/75887.html)

**《N 种内核注入 DLL 的思路及实现》**

Author : `sudami` [`sudami@163.com`]

Time : 01-11-2008

Links : [`hi.baidu.com/sudami`](http://hi.baidu.com/sudami)

内核注入，技术古老但很实用。现在部分 RK 趋向无进程，玩的是 **SYS+DLL**，有的无文件，全部存在于内存中。可能有部分人会说：“都进内核了.什么不能干？”。是啊，要是内核中可以做包括 R3 上所有能做的事，软件开发商们也没必要做应用程序了。有时，我们确实需要 R3 程序去干驱动做起来很困难或者没必要驱动中去做的事，进程 / DLL 是不错的选择，但进程目标太大，所以更多的同学趋向于注 DLL。

若要开发安全软件、小型工具，可借鉴其思路，Anti Rootkits 时，在某些极端情况下，可使用同样的技术发现、清除 RK，保证用户电脑的正常使用。在此，我将探讨几种内核注入 DLL 的思路及实现原理。

### (1) APC 技术

给一个 Alertbale 的用户态线程插 APC，让其执行其中的 ShellCode，来执行我们的代码。这个方法简单易行，但是不够稳定，兼容性不好。测试中发现经常出现 `Explorer.exe` 等插崩溃的情况，而且有杀软在的情况下，插入有时会被拦截，起不到应有的效果。（可参考我以前逆过的一个驱动：[逆向 fuck.sys--编译通过--源码](http://hi.baidu.com/sudami/blog/item/23eac8b18471d151092302f9.html)）

### (2) 内核 Patch \\KnownDLLs\\Kernel32.dll CreateThread

`\\KnownDLLs` 是系统加载时对象管理器加载最新磁盘 DLL 到内存的，当其他进程想调用某个 DLL 时，就不用重复从磁盘加载了，而会从这里映射一份到自己的进程空间中去。这样给我们做全局 Patch 提供了一个很好的机会：

`ZwOpenSection` 打开 `\\KnownDlls\\kernel32.dll`，调用 `ZwMapViewOfSection` 映射一份到自己进程空间，然后寻找 `kernel32.dll` 在内存中代码节的空隙，选择这里作为我们 fake 函数的存储 Buffer。修改 `CreateThread` 函数的开头 5 字节跳转到这个间隙，当系统任何一个线程创建时，会走到 `CreateThread` 函数，然后执行空隙中的 ShellCode，其负责调用 `LoadLibrary` 加载我们的 DLL。DLL 一经加载，会发 IOCTL 通知本驱动，让驱动卸载 HOOK。这样就完成了内核注 DLL 的过程。测试时发现 `Svchost.exe` 进程调用 `CreateThread` 函数很频繁，所以触发也会很快，基本 1 秒不到就能将 DLL 加载进去，而我们的 HOOK 也卸掉了。所以稳定性提高不少。示意图如下：

![](img/72475662051b77cdb0d3683a80c188eb.jpg)

![](img/29976a47beadabcc1721c1c23ec8a8a9.jpg)

### (3) 内核 HOOK ZwMapViewOfSection

有部分模块加载时会调用 `ZwMapViewOfSection`，比如进程创建时映射 N 份 DLL 到自己的虚拟空间中去。我们替换 SSDT 中的这个函数，过滤出是加载 `Kernel32.dll` 的情况，从参数中取得其基址，Inline Hook 其 EAT 中的 `CreateThread` 函数，跳转到在这个进程虚拟地址空间中申请的 Buffer，在其中完成 DLL 的加载过程。

关键 API：

`ZwAllocateVirtualMemory` ---- 在此进程空间中分配内存，存放 Shellcode

`ZwProtectVirtualMemory` ---- 使当前内存块具有可读可写属性

`IoAllocateMdl` ---- 创建 MDL

关键 Code 如下：

![](img/3fa0cd5bb08278094dc1d4ca825358bb.jpg)

![](img/48682caa494e54b4a1bc3949e15ef1de.jpg)

同方法 2 相比，原理类似。但修改时机不同，效果差不多，只是注入 DLL 的时间会慢一些。至于 Shellcode 的编写，就大同小异了。萝卜白菜各有所爱，主要看个人发挥。要是闲写 shellcode 麻烦，请到看雪学院去查资料，模板很多，在这里就不 YY 了。

*   [【看雪读书月】学习 ShellCode 编写](http://bbs.pediy.com/showthread.php?t=69754&highlight=shellcode)
*   [[note]一个简单的 Shellcode](http://hi.baidu.com/sudami/blog/item/ac76e84befff84f383025c41.html)
*   [shellcode 之小小琢磨](http://bbs.pediy.com/showthread.php?t=60338&highlight=shellcode)
*   [Add_Section](http://bbs.pediy.com/showthread.php?t=36497&highlight=shellcode)

### (4) 内核 HOOK NtCreateThread

跟踪进程创建的流程，会很明晰的发现有多点可以 patch 来实现 DLL 的注入。

进程创建完时是一个**空水壶**，里面没有沸腾的热水（threads），于是系统调用 `NtCreateThread` 创建其主线程（给空水壶注水 **凉水**），在这个暂停的线程里面折腾了一阵后完事了也厌倦了，于是系统跳了出来，回到进程空间中，调用 `Kernel32.dll` 去通知 `CSRSS.EXE`，对它说：“这里有一个新进程出生了，你在你的表里标记一下”。然后就开始加载 DLL 啦，把系统 `KnownDLLs` 中的自己需要的 DLL 都 Map 一份到这个大水壶中。接着 `KiThreadStartup` 加热水壶中的凉水，于是水就开始**沸腾**了，此时主线程开始工作。。。

拦截 `NtCreateThread`，取得**当前线程上下文**，保存它要返回的地址（会回到空水壶中去），劫持为我们自己分配的地址，在其中填充 ShellCode 来加载目的 DLL。至于选择 Buffer，思路很多。这里可简单的 Attach 到当前进程，在充足的虚拟 2GB 进程地址空间中分配属于你自己的一块小内存，够放 ShellCode 足矣。示意图如下：

![](img/882e68cb672d1186a5506a7a39df3137.jpg)

### (5) 内核感染常用模块，让感染模块帮我们 Load DLL

这个方法就有点绕远了，开始了最本质最原始的感染，可增加新节，可插空隙，总之，让别人的模块 Load 进内存时顺路的帮我们加载下 DLL，DLL 一旦加载就可以恢复感染，清除痕迹。至于感染代码，网上一堆。只要不是驱动感染驱动（多了个校验和），其他性质都一样，看自己发挥啦。

### (6) 拦截 NtCreateUserProcess、NtCreateSymbolicLinkObject

前者在 Vista 下才有。拦截后通过 `PsLookupProcessThreadByCid` 得到 ETHREAD / EPROCESS，判断是否是 `CSRSS.EXE` 引起的，若是则在此进程空间内分配一块内存，调用 `NtGetContextThread` 得到当前的线程上下文，调用 `ZwWriteVirtualMemory` 填充 Shellcode 区域，取得 `LdrUnloadDll`、`LdrGetDllHandle` 等函数地址，通过他们加载 DLL。然后调用 `NtSetContextThread` 恢复原始的 Context。关于这种方法，可参考 DriverDevelop 上某人发的 BIN。

[内核实现 DLL 注入.可以完美绕过 KAV 瑞星等杀毒软件](http://bbs.driverdevelop.com/htm_data/16/0807/112499.html)

### (7) 内核拦截 NtResumeThread

### (8) NtUserSetWindowsHookEx 注入

顺便提下 R3 上 DLL 的注入：

1.  `CreateRemoteThread` (or `NtCreateThreadEx` (Used in Vista))
2.  `SetThreadContext` (change the EIP)
3.  `NtQueueAPCThread`
4.  `RtlCreateUserThread`
5.  `SetWindowHookEx`

### 小结

纵观进程启动的全过程，可 patch 的地方很多，只要保证进程、线程上下文不被破坏，注入的手法可多种多样。只要保证我们的 DLL 注入时间足够短，稳定性足够高即可。当我们迫不得已要从内核注入 DLL 到用户进程去时，系统已经中毒很深，此时运用类似上面提到的技术来加载 DLL，让 DLL 做我们驱动无法完成的任务，是可以接受的。

上面提到的思路是我暂时想到并且已经实现了的，详细过程可参见代码。欢迎积极探讨更好更稳定而且**不邪恶**的方法。

---

参考资料：

1.  [Windows Internals---Processes,Threads,Jobs](http://hi.baidu.com/sudami/blog/item/f7262627f62e1700908f9d5c.html)
2.  [漫谈兼容内核系列:WINDOWS 进程的用户空间](http://www.google.com/search?q=%E6%BC%AB%E8%B0%88%E5%85%BC%E5%AE%B9%E5%86%85%E6%A0%B8%E4%B9%8B%E4%BA%8C%E5%8D%81%E4%B8%80+WINDOWS%E8%BF%9B%E7%A8%8B%E7%9A%84%E7%94%A8%E6%88%B7%E7%A9%BA%E9%97%B4&hl=zh-CN&sourceid=gd&rlz=1Q1GGLD_zh-CNCN299CN299&aq=t)
3.  WRK 1.2, windbg
