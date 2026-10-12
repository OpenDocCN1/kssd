
# 禁止调用任务管理器方法

> 原文：[`www.pediy.com/kssd/pediy12/128441.html`](https://www.pediy.com/kssd/pediy12/128441.html)

很久之前就遇到过这个需求，总之方法很多，本文将会列举一些你见过的或者没见过的，想到过的或者没想到过的方法。通常调用任务管理器有 2 个方法：1. `ctrl+alt+del` 2. 任务栏点右键选择任务管理器。其中 1 是由 `winlogon.exe` 来处理的，2 是由 `Explorer.exe` 进程 `CreateProcess` 来创建的。区别蛮大的。

## 方法 1. 修改注册表

`[HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\policies\system]` 下新建一个 `DWORD` 键，名为 `DisableTaskMgr`，设置为 1。如果注册表项不存在就自己创建。

## 方法 2. 使用组策略

在开始菜单“运行”对话框中键入 `gpedit.msc` 打开“组策略”设置。展开“本地计算机策略”→“用户配置”→“管理模板”→“系统”→“Ctrl+Alt+Del 选项”，双击右边的“删除任务管理器”，选择“属性”中的“已启用”即可。设置为“未设置”或“已禁用”即可解禁。

## 方法 3. 伪造 msgina.dll

这个方法网上有很多介绍了，文章都比较早了，而且弄不巧就进不去系统了。这里就不介绍了。这个方法比较麻烦，而且还要考虑怎样禁止 `Explorer` 创建任务管理器。

## 方法 4. dll 注入 & Hook api（未实现）

注入一个 `dll` 到 `winlogon.exe` 进程，hook `msgina! WlxLoggedOnSAS`，因为这个函数就是处理调用 `ctrl+alt+del` 的。同时需要另一个 `dll` 注入到 `Explorer.exe` hook `kernel32!CreateProcess`。阻止创建 `taskmgr.exe` 进程。（注入 `winlogon` 会不会存在权限什么的问题？？？有兴趣的可以试试。俺不知道）

## 方法 5. patch 进程（未实现）

直接通过 `writeprocessmemory` patch `winlogon.exe` `explorer.exe` 进程。这个方法稳定性起码就很差，而且需要比较深厚的逆向功底来附加调试进程，找到关键调用。

## 方法 6. 伪造一个假的任务管理器

随便找一个 `exe` 重名为 `taskmgr.exe` 替换 `system32` 目录和 `system32\dllcache` 下的 `taskmgr.exe`，先备份一下原来的，找不到别怪我。试试咋样？还行吧？美中不足的是会有一个错误提示，说什么 window 不能识别的版本。把原来的 `taskmgr.exe` 拖到 `OllyDbg` 里就在入口点加一句 `ExitProcess`，然后替换，小样 window 不会提示说不识别的版本了吧~！搞定！

`Ps.` 有人说挂 `WH_KEYBOARD_LL` 低级键盘钩子，貌似这个绕不过吧？

题外话：

记得看《疯狂的程序员》的时候，里面有一道题目，写一个函数输入年月日，输出这天是星期几。

绝影给的方法是先把系统时间设置为这一天 然后看看是星期几 然后把时间再改回来，哈哈 先不论他的功能性和可行性 起码这种思维方式 就令人拍案叫绝~~~就像这篇文章的第 6 中方法，如此之简单明了。换一种思路也行就豁然开朗了。。。今天蛋疼，给出一个函数完成绝影的功能，而且功能性还蛮不错的。地址：[`hi.baidu.com/blueapple_c/blog/item/bcd4ef127e6fd2936538db8f.html`](http://hi.baidu.com/blueapple_c/blog/item/bcd4ef127e6fd2936538db8f.html)

看雪：`blueapplez`
2011 年 1 月 21 日 14:46:34
