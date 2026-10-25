
# \[0Day\]WINDOWS 7 RC 7100 GDI 驱动 Win32k.sys 内核 D.O.S 漏洞

> 原文：[`www.pediy.com/kssd/pediy10/88782.html`](https://www.pediy.com/kssd/pediy10/88782.html)

\[0Day\]WINDOWS 7 RC 7100 GDI 驱动 Win32k.sys 内核 D.O.S 漏洞

作者：MJ0011 2009.05.13

感谢：Iceboy 发现此问题并提供 DUMP

漏洞厂商及产品：Microsoft Windows 7 rc 7100 090421

存在漏洞组件：`win32k.sys` Timestamp :`49ee8dc8`

存在漏洞函数：`NtUserGetDc`/`NtUserGetDcEx`

漏洞描述：`Win32k.sys` 是 WINDOWS 的 GDI 驱动程序，包含大量复杂的图形界面处理，由于其中大量代码是从 WIN3.1 中修改过来，因此成了 WINDOWS 漏洞多发之地。

这个漏洞主要是因为 Windows 7 在其 `NtUserGetDc`/`NtUserGetDcEx` 函数中（也许不仅仅是这两个函数）不正确地使用了共享临界锁，导致了任何权限下的 GDI 程序可以引发内核 BSOD，从而进行 DOS 攻击

## 漏洞分析

在 WINDOWS VISTA 中，在这两个函数进入前，会调用 `UserEnterUserCirtSec`，进入临界区，同时会将 `gptiCurrent` 设置为当前线程的 WIN32THREAD

`UserEnterUserCritSec` 的实现如下：

代码:

```
PWIN32THREAD UserEnterUserCritSec()
{

PWIN32THREAD pwin32Thread;

pwin32Thread = ExEnterPriorityRegionAndAcquireResourceExclusive(gpresUser);
gptiCurrent = pwin32Thread;
gbValidateHandleForIL = 1;
return pwin32Thread;
}
```

`ExEnterPriorityRegionAndAcquireResourceShared` 是 `NTOSKRNL` 导出一个提供给 WIN32K 使用的共享资源锁函数，这个函数将共享锁住 `gpresUser` 这个资源，同时返回当前线程的 Win32Thread,即 `KeGetCurrentThread->Win32Thread`

同时内核函数 `ExEnterPriorityRegionAndAcquireResourceExclusive` 也与其类似。

（XP 则是调用 `KeEnterCriticalRegion` 后，用 `ExAcquireResourceExclusiveLite` 锁住 `gpresUser`,然后用 `PsGetCurrentThread->PsGetThreadWin32Thread` 获得当前线程 win32kthread,存放到 `gptiCurrent`）

而在 windows 7 中，这两个函数进入前，改为了调用 `UserEnterSharedCrit`，进入共享临界区

`UserEntrySharedCirt` 的实现很简单

代码:

```
PWIN32THREAD EnterSharedCrit()
{
return ExEnterPriorityRegionAndAcquireResourceShared(gpresUser);
}
```

可以注意到，这里并不设置 `gptiCurrent`，事实上，`NtUserGetDc`/`NtUserGetDcEx` 会将这个函数保存在寄存器中以便使用 `CurrentWin32Thread` 中存放的数据

从关键临界转为共享临界，无疑这样的修改将提升 `NtUserGetDc`/`NtUserGetDcEx` 的效率，减少了锁竞争的可能，但是在这里这样修改是错误的。

因为在共享临界中，没有修改 `gptiCurrent`，那么就很可能遇到 `gptiCurrent` 非法的状态,比如遇到一个没有做 `PsConvertToGuiThread` 的线程导致了 `gptiCurrent` 为空。

WIN32K 中有大量内部例程（经常以 `zzz`,`xxx` 开头），他们都认为在调用自己之前，`gptiCurrent` 是一个有效的状态。这其中就包括了 `xxxDestoryWindow`

一个 BSOD 的例子是：`NtUserGetDc`->`GetWindowDc`->`GetDcEx`->`SpbCheckDce`->`SpbCheckRect`->`SpbCheckRect2`->`FreeSpb`

调用到 `FreeSpb` 时，这里要释放一个 spb 对象了

接着就到

`FreeSpb`->`HMAssigmentUnlock`->`HMUnlockObject`->`HMUnlockObjectInternal`->`HMDestroyUnlockedObject`，最终调用到了 ganti 表中的函数，由于这里是 `GetWindowDc`,所以将调用 `xxxDestroyWindow`，`xxxDestroyWindow` 第一句代码就无检查引用了 `gptiCurrent` 指针中的数据，自然直接导致 BSOD

## 解决方案

等待微软更新官方补丁。

微软可能的更新手段：

牺牲效率将 `NtUserGetDc` 等中的不正确共享锁替换为关键临界锁，或者将所有 `NtUserGetDc` 等函数可能用到的内部函数中的 `gptiCurrent` 引用去掉或换为不引用 `gptiCurrent` 的其他函数
