
# 共享段攻击 Shared Sections' Attacking

> 原文：[`www.pediy.com/kssd/pediy12/139116.html`](https://www.pediy.com/kssd/pediy12/139116.html)

一般常见的某些软件喜欢使用`HHOOK`类钩子注入`DLL`，由于他们多数都抄了流行的那个典范代码中一些东西，比如共享段，对于共享段而言，丫是全宇宙可读可写啊（属性是`0xD0000040`）

于是一种新型的针对某些软件的某些部件的攻击方式诞生鸟~

这里以金山`KSAFE`的`ksfmon.dll`举个例子

![](img/c5705d05f82d6cb0ad883b138b8097a4.jpg)

`IDA`中打开的样子

![](img/9552f49a0efafbd81d1f09e6c6fe2629.jpg)

我震惊的发现`HHOOK`

想到什么了？

对如果我们直接对丫进行`XX`

测试代码：

```
DWORD ptr=*(DWORD *)(GetModuleHandleA("ksfmon.dll")+0x139000+0x0C);
UnhookWindowsHookEx((HHOOK)ptr);
```

同类软件也存在类似的问题，比某那啥，某那啥~这里就不一一列举了~这种攻击有时候会造成杀毒软件直接崩溃退出（某`XX`的共享段清 0 后主进程自动退出），有的则造成监控状态异常~
