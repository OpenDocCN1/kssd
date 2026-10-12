
# 发个用于监视目标程序的反调试的小工具

> 原文：[`www.pediy.com/kssd/pediy11/125464.html`](https://www.pediy.com/kssd/pediy11/125464.html)

在软件调试或脱壳的时候经常遇到反调试，但是不知道目标程序用了哪种反调试技术，在什么地方做了反调试。使用这个辅助工具可以看到具体的反调试手段与信息。

注意：如果目标程序检测了 `peb` 的 `DebugFlag` 位或者 `NtGlobalFlag` 位，本工具不输出相关信息。

原理很简单，就是一个调试器

1. 在驱动程序中，`hook` `ZwQueryInformationProcess`，检测 `ProcessDebugPort`，`hook` `ZwQueryObject`，检测 `DebugObject`，`hook` `SetInformationThread`，检测 `ThreadHideFromDebugger`，`hook` `GetContextThread`，检测对调试器硬件断点的检测，`hook` `SetContextThread`，检测清除调试器的硬件断点
2. 去掉 `PEB` 中的 `DebugFlag` 位和 `NtGlobalFlag` 位，`heap` `flag` 位等。
3. 对 `KiUserExceptionDispatcher` 和 `ZwContinue` 设置硬件断点，检测异常处理中的反调试
4. 在发生 `CREATE_THREAD_DEBUG_EVENT` 事件时，对新建线程设置 `KiUserExceptionDispatcher` 和 `ZwContinue` 硬件断点，检测其他线程中的反调试
5. 如果发生单步异常或调试异常之类的事件，也输出异常信息和地址
6. 如果目标程序发生了异常（即拦截到了硬件断点 `KiUserExceptionDispatcher`），那么清除 `context` 中的调试寄存器，并在执行下一个 `ZwContinue` 的时候还原。

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `HideDebugger.rar`
|
