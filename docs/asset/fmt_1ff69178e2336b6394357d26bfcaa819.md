
# 利用 `IShellExecuteHook` 接口对程序监控

> 原文：[`www.pediy.com/kssd/pediy11/122023.html`](https://www.pediy.com/kssd/pediy11/122023.html)

`http://blog.csdn.net/laiboy`  

`http://laiboy.cublog.cn`  

`Windows` 的应用程序的调用大部分是利用 `ShellExecute()`和 `ShellExecuteEx()`这两个 `API` 调用，而这两个 `API` 最终调用的是 `CreateProcess()`这个 `API`，它们的作用是获取进程序足够的参数传给 `CreateProcess()`。所以，只要监控这两个 `API` 的调用，就可以实现对程序启动的监视和控制。  

`Windows` 操作系统提供了 `IShellExecuteHook` 这个 `COM` 的接口，这是一个传统的 `Windows` 操作系统的 `HOOK` 模型中的一个接口。  

## 注册 `IShellExecuteHook` 函数  

`IShellExecuteHook` 是一个 `COM` 服务器，这个 `COM` 所注册的地方位于注册表两个地方，分别为：  

`HEKY_CLASSES_ROOT`  

`\\CLSID`  

与  

`HKEY_LOCAL_MACHINE`  

`\\Software`  

`\\Microsoft`  

`\\Windows`  

`\\CurrentVersion`  

`\\Explorer`  

`\\ShellExecuteHooks`  

处理的 `IShellExecuteHook` 的 `handler`（即 `Com` `server`）必须要在这两个地方注册。  

## 实现 `IShellExecuteHook` 的 `Execute` 处理函数  

以下为实现 `IShellExecuteHook` 的 `Execute` 方法须要注意的问题，`Execute` 方法带有一个 `LPSHELLEXECUTEINFO` 的参数，该参数指向一个可执行程序的所有信息，执行动作（打开，显示，链接等）`lpesi->lpVerb`，被执行程序的完整路径名称，`lpsei->lpFile`，调用的目录，程序执行的参数等。  

当返回值为 `S_FALSE` 的时候，则是允许程序按正常方式打开，当返回为 `S_OK` 的事候，则程序是被阻止执行。在实现程序黑名单的时候，当发现程序是包含在黑名单中的，则返回 `S_OK`，实现程序执行拦截。  

## 实现 `IShellExecuteHook` 的源码  

如图，实现一个 `IShellExecuteImple` 类，并实现 `IUnkown` 和 `Add`，`Release` 等 `COM` 接口。  

实现 `IShellExecuteHook` 接口的 `Execute` 方法:  

以下为实现 `Execute` 的方法。  

设计如下：  

 记录可执行程序的名称，启动时间，并记录到 `C:\\ShellHook.txt` 文件中  

 读取程序拦截黑名单(`C:\\ShellHook.ini`)，根据黑名单拦截程序。  

最后修改 `rgs` 文件，注册 `ShellHook.dll` 的时候会在指定键加入 `ShellHook.Dll`  

## 调试 `IShellExecuteHook` 的问题  

每次编译新运行的时候需要载 `ShellHook.dll`,并注销重新注册加载测试。  

当 `Execute` 执行失败时，会导致 `Windows` `Explorer` 无法执行，要在安全模式下重新运行 `regsvr32 /u ShellHook.dll` 才能让 `Windows` `Explorer` 正常运行。  

## 运行 `IShellExecuteHook` 的结果  

执行程序的时间  

被拦截的程序  

## 让 `IShellExecuteHook` 支持 `Windows` `7` 和 `Vista`  

需要将以下开关打开，则 `IShellExecuteHook` 接口可以在 `Windows` `7` 和 `Vista` 下运行。  

`HKEY_LOCAL_MACHINE`  

`\\Software`  

`\\Microsoft`  

`\\Windows`  

`\\CurrentVersion`  

`\\Policies`  

`\\Explorer`  

加入以下键值  

`EnableShellExecuteHooks`  

`REG_DWORD`  

`1`  

则可以让 `IShellExecuteHooks` 在 `Windows` `7` 和 `Vista` 上运行。  

`http://blog.csdn.net/laiboy/archive/2010/07/22/5755324.aspx`  

## 源码  

`http://blogimg.chinaunix.net/blog/upfile2/100722143536.rar`  

上不了图!麻烦一点就看源码吧!  

高手不要拍砖了!!!`[](img/c1376e2bd59b6be48d07a70822c7e47d.jpg "伤心")`![](img/c1376e2bd59b6be48d07a70822c7e47d.jpg "伤心")
