
# 玩玩 360ExitWindowsEx 大法

> 原文：[`www.pediy.com/kssd/pediy10/97539.html`](https://www.pediy.com/kssd/pediy10/97539.html)

现在流行 `R3`，对 `R0` 里的东西大伙都不太感兴趣了，俺来放个 `R3` 暴力结束进程的代码，看雪里貌似有个类似的 `bin`，不管它，玩玩而已-_-

先来看下 Windows XP 的关机流程：

1. 当 Windows XP 用户发起关机指令以后，发起关机指令的执行程序会调用系统函数库 `user32.dll` 中的 `ExitWindowsEx` 函数，此函数向 XP 系统进程 `Csrss.exe` 发出关机信息，`Csrss.exe` 立即再把信息传递给隐含的 `Winlogon.exe` 窗口。

2. `Winlogon.exe` 接到前面 `Csrss.exe` 传来的信息后，`Winlogon.exe` 开始检查请求者的权限，预先做好准备，并给 `ExitWindowsEx` 发回准备就绪信号。`Csrss.exe` 收到 `Winlogon.EXE` 的通知以后，会依次查询拥有顶层窗口的用户进程，让这些用户退出进程。如果某一个用户进程在一个默认的延时时间 5000 毫秒内没有退出的话，Windows XP 会显示一个结束任务的对话框用于询问用户是否结束这个任务。默认情况下将显示这个对话框并一直保持而不会自动关闭。

3. 此时 `Winlogon.exe` 将再次调用 `ExitWindowsEx` 函数来关闭系统进程。（这些系统进程包括 `SMSS.EXE`、`Winlogon.EXE`、`Lsass.EXE` 等）。Windows 在终止系统进程的时候并不像终止用户进程那样：进程无法在规定时间内终止，则提示用户。而是跳过这个进程，去执行下一个系统进程的终止操作。在这个时间段里面，Windows XP 会执行子系统来完成最后的关机操作。

4. 当准备工作全部完成后，`Smss.exe` 命令释放所有系统资源，最后 `Smss.exe` 调用 `NtShutdownSystem` 函数，等除了电源管理以后的全部子系统完成退出以后，电源管理完成最后的操作:重启或关机。

了解了 Windows XP 的关机流程以后，偶们很容易利用 Windows 窗口消息机制，实现 `ExitWindowsEx` 伪关机操作，结束顽固窗口进程。代码完成后，初略试验了一下，`V5.2` 版 360 和保险箱是无声无息的消失了^-^..微点、卡巴、金山、瑞星之类的杀软窗口进程也可以结束掉，主防成了睁眼瞎，加载驱动，不再有摭拦，很好玩啊。呵呵。。。。

关于 `WM_QUERYENDSESSION`，MSDN 上有明确的讲解，摘录如下，

```
The WM_QUERYENDSESSION message is sent when the user chooses to end the session or when an application calls the ExitWindows function. If any application returns zero, the session is not ended. The system stops sending WM_QUERYENDSESSION messages as soon as one application returns zero.

After processing this message, the system sends the WM_ENDSESSION message with the wParam parameter set to the results of the WM_QUERYENDSESSION message.

WM_QUERYENDSESSION

nSource = (UINT) wParam;    // source of end-session request

fLogOff = lParam            // logoff flag

Parameters

nSource

Reserved for future use.

fLogOff

Value of lParam. Indicates whether the user is logging off or shutting down the system. Supported values include: ENDSESSION_LOGOFF.

Return Values

If an application can terminate conveniently, it should return TRUE; otherwise, it should return FALSE.

Remarks

By default, the DefWindowProc function returns TRUE for this message
```

代码放后面了，大家共同交流学习。祝大家节日快乐，每天都有毒杀-_-

## 上传的附件

| 图片 | 文件 |
| :--- | :--- |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `Kill360 Bin.rar` |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `kill360 Src.rar` |
