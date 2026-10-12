
# IceSword 是如何结束线程的？

> 原文：`www.pediy.com/kssd/pediy10/80064.html`

`img/3e9367e5826886294bccbb08463f3e34.jpg` 冰刃的强力结束线程貌似很牛，`img/e71d9210a215d5b66755d038631e0c83.jpg` 在网上找了几种驱动代码资料，自己整理编译后，貌似都不如冰刃的强，貌似调用的不是 `PspTerminateThreadByPointer`

`img/3e9367e5826886294bccbb08463f3e34.jpg` 哪位大牛扔点代码或者指点一下小弟

山寨版的 `IceSword` 也不如它

冰刃有两种结束线程的方式 `Terminate` 和 `Kill(Force)`

其中 `Terminate` 功能是使用 `ObOpenObjectByPointer` 打开找到的线程对象，然后调用找到的原始的 `NtTerminateThread` 函数结束之

`Kill(Force)` 功能，使用 `PsLookupThreadByThreadId` 等函数获得线程对象，然后使用 `KeInsertQueueApc` 插入 Kernel routine apc

在 APC 例程里，先调用 `PsTerminateSystemThread` 试图去结束系统线程，如果这里不是一个系统线程，就会失败，那么会接着调用之前找到的原始的 `NtTerminateThread` 函数来结束这个线程。此处的传入线程句柄是使用的 `PsGetCurrentThread`，即-1，自结束
