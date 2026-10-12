
# 反 StrongOD 的原理和实例

> 原文：[`www.pediy.com/kssd/pediy12/139657.html`](https://www.pediy.com/kssd/pediy12/139657.html)

反 `StrongOD.dll` 的原理和实例：

还是采用 `FD_DebugObject_NtQueryObject` 反调试，不过换个写法就可以过 `StrongOD`。

`FD_DebugObject_NtQueryObject` 原理是这样的：当调试会话创建，将产生一个调试对象，我们通过 `ntdll` 中的 `NtQueryObject` 函数参看调试对象的个数是否不为零，来确定调试器的存在。《【原创】软件保护壳技术专题 - 反调试器技术》（http://bbs.pediy.com/showthread.php?t=71113）中的 `FD_DebugObject_NtQueryObject` 通过循环遍历 `ObjectTypeInformation` 对比类型的名字,如有类型名为 `DebugObject`，则检测 `TotalNumberofHandles` 与 `TotalNumberofObjects`，如果不为 `0` 则存在调试器。

`StrongOD.dll` 里的 `fengyue.sys` 驱动 `HOOK` 了 `ntdll` 中的 `ZwQueryObject`，在 `XP` 系统里他只是简单的把 `DebugObject` 对象类型隐藏了，通过搜索 `DebugObject` 字符，如果不存在就说明有调试器。如果存在并且 `TotalNumberofHandles` 不为 `0` 就说明也存在调试器。在 `WINDOWS 7` 中 `StrongOD.dll` 好象没把 `DebugObject` 对象类型隐藏。

当你用 `OD` 打开一个程序，同时运行 `ceshi.exe`，会提示“debug?”.或者当你在 `OD` 里运行 `ceshi.exe`，也会提示“debug?”，可以过最新的 `StrongOD.dll`。当双击 `ceshi.exe`，在没有 `OD` 打开的情况下，会出现“fun!”。

本程序是在 `OD` 里用 `shellcode` 格式写的。在 `XP` 中的 `SP2`、`SP3` 和 `WINDOWS 7` 中测试通过。欢迎讨论。

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || ceshi.rar  

 |
