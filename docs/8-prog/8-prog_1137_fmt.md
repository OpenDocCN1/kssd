
# 分享源码：自己学习 windows 内核而写的工具 kernel star

> 原文：[`www.pediy.com/kssd/pediy11/119991.html`](https://www.pediy.com/kssd/pediy11/119991.html)

这个是我为了学习 `windows` 内核而写的一个工具，参考了看雪和网上 n 多大大的代码和思路撒，再此谢谢撒![](img/cb6288ad81b189b02921e98a92da6abe.jpg "124.gif")

中间历经无数次的调试蓝屏![](img/110d920e4e703968a2a60c986d363163.jpg "尴尬")，尤其是在虚拟机里面用 `od` 调试 `r3` 的程序。那个痛苦啊，好不容易熬过来了，应用层是纯 `sdk` 写的，繁琐饿。打算以后都用 `mfc` 写程序了，![](img/9b2aec7f3d4bb041b2d419a59832192f.jpg "恐怖!")![](img/45dd044a2743704cbb333715eae4a5f6.jpg "狂笑")

简单介绍下这个工具吧：

有如下 12 个功能（测试通过 `xp sp3` 和 `xp sp2`）

## 一：SSDT 表的 hook 检测和恢复

~！~~~

## 二：IDT 表的 hook 检测和恢复

~~~~~~（`idt` 多处理器的恢复没处理，自己机器是单核的，没得搞，不过多核的列举可以）

## 三：系统加载驱动模块的检测

通过使用一个全局 `hash` 表(以 `DRIVEROBJECT` 为对象)来使用以下的方法来存储得到的结果，最终显示出来

1. 常规的 `ZwQuerySystemInformation` 来列举
2. 通过打开驱动对象目录来列举
3. 搜索内核空间匹配驱动的特征来列举(这个功能里面我自己的主机一运行就死机，别的机器都没事，手动设置热键来蓝屏都不行，没 `dump` 没法分析，哎，郁闷)
4. 从本驱动的 `Modulelist` 开始遍历来列举驱动

## 四：进程的列举和进程所加载的 dll 检测

采用以下方法来列举进程：

1. `ZwQuerySystemInformation` 参数 `SystemProcessesAndThreadsInformation` 来枚举
2. 进程 `EPROCESS` 结构的 `Activelist` 遍历来枚举
3. 通过解析句柄表来枚举进程
4. 通过 `Handletablelisthead` 枚举进程
5. 进程创建时都会向 `csrss` 来注册，从这个进程里面句柄表来枚举进程
6. 通过自身进程的 `HANDLETABLE` 来枚举进程
7. 通过 `EPROCESS` 的 `SessionProcessLinks` 来枚举进程
8. 通过 `EPROCESS ---VM---WorkingSetExpansionLinks` 获取进程
9. 暴力搜索内存 `MmSystemRangeStart` 以上查找 `PROCESS` 对象

**进程操作：**

进程的唤醒和暂停通过获取 `PsSuspendProcess` 和 `PsResumeProcess` 来操作的

进程结束通过进程空间清 0 和插入 `apc`。

采用以下方法查找 `DLL`：

1. 遍历 `VAD` 来查找 `dll`
2. 挂靠到对应的进程查找 `InLoadOrderLinks` 来枚举 `dll`
3. 暴力搜索对应进程空间查找 `pe` 特征来枚举 `dll`

**DLL 的操作：**

`Dll` 的卸载是通过 `MmUnmapViewOfSection` 和 `MmmapViewOfSection`（从 `sdt` 表中相应函数搜索到的）来实现的（本来想直接清 0 `dll` 空间，有时行有时不行）（只要将这个进程的 `ntdll` 卸载了，进程就结束了，一个好的杀进程的办法撒，绿色环保无污染），注入 `dll` 使用的是插入 `apc` 实现的。（注入的 `dll` 必须是 `realse` 版的。`Debug` 版会出现`***`错误，全局 `dll` 注入貌似也是）插入 `apc` 效果不是很好，要有线程有告警状态才执行。

## 五：线程信息的检测

遍历 `ThreadList` 来枚举线程

线程的暂停和唤醒都是通过反汇编获取 `PsResumeThread` 和 `PsSuspendThread` 直接从 `r3` 传来 `ETHREAD` 来操作的，通过插入 `APC` 来结束线程

## 六：shadow sdt 表的 hook 检测与恢复

没有采用 `pdb` 来解决函数名问题，直接写入 `xp` 和 `03` 的 `shandow` 表函数名（主要是自己的网不稳定，连 `windbg` 有时都连不上微软）

## 七：系统所有的过滤驱动的检测

查看各 `device` 下是否挂接有驱动之类的，可直接卸载

## 八：系统常用回调历程的检测和清除

只检查了 `PsSetLoadImageNotifyRoutine` `PsSetCreateThreadNotifyRoutine` `PsSetCreateProcessNotifyRoutine` `CmRegisterCallback` 这几个，至于那个什么 `shutdown` 回调不知道是啥玩意，就没搞了，有知道的顺便告诉我下撒，谢谢

## 九：文件系统 fat 和 ntfs 的分发函数检测

直接反汇编 `fat` 和 `ntfs` 的 `DriverEntry` 得到对应的填充分发的偏移，然后和当前已经运行的文件系统的分发相比是否被 `hook`，并附带恢复

## 十：文件查看功能

自己解析 `ntfs` 和 `fat` 的结构，来实现列举文件和直接写磁盘删除。附带有普通的删除和发生 `IRP` 来删除。不过这里面有点问题，`ntfs` 删除有时把目录给搞坏了，大家凑合着吧，

`Ntfs` 网上删除这些操作的代码不多，就是 `sudami` 大大的利用 `ntfs-3g` 来实现的，看了下，太多了，充满了结构。然后自己对照着系统删除文件时目录的变化来自己实现的。只处理了`$BITMAP` 对应的位清除，父目录的对应文件的索引项的覆盖，删除文件对应的 `filerecord` 清 0.

另外偷懒时间都没处理，呵呵，`y` 的，一个破时间都都搞好几个字节移来移去的。

## 十一：常用内核模块钩子的检测和恢复

这里只检测了主要的内核模块 `nkrnlpa*.exe` 的`.win32k.sys`，`hal.dll`，比对它们的原始文件从而查找 `eat inline hook`，`iat hook` ，和 `inline hook`。`Inline` 是从 `TEXT` 段开始一段位置开始比较的。（有点慢貌似，等待显示扫描完成就好了）

## 十二：应用层进程所加载 dll 的钩子

应用层钩子检测和内核模块钩子检测原理一样，不过为了能读写别的进程的空间，并没有使用 `openprocess` 去打开进程，而是通过 `KiattachProcess` 挂靠到当前进程，然后通过在 `r0` 直接读写进程空间的。

最后付个图撒：

![](img/a0609a19c73119372557526ad3b2c5c4.jpg)

还是有好多的 `bug` 在，没时间在继续搞了，等有时间在完善下吧（要忙着写简历了，找 `job`），欢迎大家测试，要是蓝屏了，可以的话发下 `dump` 撒，谢谢哈(邮箱：`834858875@qq.com`)。调试调累死了，打算这段时间看书，然后在继续 `code`。![](img/45dd044a2743704cbb333715eae4a5f6.jpg "狂笑")![](img/45dd044a2743704cbb333715eae4a5f6.jpg "狂笑")

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `驱动层.rar` |
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `应用层.rar` |
