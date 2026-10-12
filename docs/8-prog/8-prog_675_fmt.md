
# `Windows 7` 下 `NtSetSystemInformation` 加载驱动新方法

> 原文：[`www.pediy.com/kssd/pediy10/85816.html`](https://www.pediy.com/kssd/pediy10/85816.html)

`NtSetSystemInformation` 的 `0x68` 号命令 `SystemProcessorMicrocodeUpdateInformation`

此调用可以 `Load` 或 `Unload` 处理器 `Microcode` 的更新驱动程序，当调用此命令时，会执行：

`_ExpSetProcessorMicrocodeUpdateInformation`->`_ExpMicrocodeInformationLoad` 或 `_ExpMicrocodeInformationUnload`

接着系统检查当前进程拥有 `LOAD DRIVER` 权限后，即会使用一个 `WorkItem` 来调用 `_ExpLoadMicrocodeImage`

会加载 `SystemRoot\System32\mcupdate.dll`（实际是个驱动）

接着会调用 `xHalLoadMicroCode(参数为驱动基址）` 完毕后调用 `xHalUnloadMicroCode`，接着会卸载这个 `IMAGE`

在我逆向的版本(`NtBuildNumber` = `7000`)下 `xHalLoadMicroCode` 和 `xHalUnloadMicroCode`

都仅有一行代码:

```
return STATUS_NOT_IMPLEMENTED;
```

所以至少 `7000` 下这个方法仅能加载驱动，不能执行代码

不过可能以后微软实现这两个函数，可以做一些猥琐的事~
