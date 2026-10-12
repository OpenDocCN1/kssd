
# PECompact2 变形工具

> 原文：[`www.pediy.com/kssd/pediy11/118380.html`](https://www.pediy.com/kssd/pediy11/118380.html)

- **标 题**: 【原创】PECompact2 变形工具
- **作 者**: jgaoabc
- **时 间**: 2010-08-11, 15:35

我采用纯汇编语言做了一个 `PECompact2` 的变形工具，适合 `PECompact2` 压缩后的变形处理。`PECompact2` 是一个比较优秀的压缩软件，适合 DLL 和 EXE 文件的压缩处理，压缩后的文件入口处采用一段经典异常。知道它的原理脱壳是非常方便的。

### 改壳处理
我在它的基础上在第一段代码中增加了 `1k` 字节的空间，在第一个异常中又增加了一段反调试代码，这段反调试代码出自《软件保护壳技术专题 - 反调试器技术》（[http://bbs.pediy.com/showthread.php?t=71113](http://bbs.pediy.com/showthread.php?t=71113)），同时在反调试中又增加了另外一段异常，这段异常代码同样出自《反调试器技术》，编号是 `FV_VMWare_VMX`，大家可以看下说明，注意，在虚拟机里程序是不能执行的。这样就形成了在异常中有反调试，而反调试中又有异常，增加脱壳的难度。

### 实现方法
通过 `CreateFileMappingA` 和 `MapViewOfFile` 函数，把要修改的文件直接映射到内存中进行修改。

### 使用方法
该工具在 DOS 下使用，把要处理的文件放在和工具同一个目录下，命令为：`bian.exe EXE/DLL 文件名称`。`PECompact2` 压缩选项最好采用默认。

同时附上通过该工具处理过的记事本程序，由于水平有限，难免会出现错误，请大家指教。

经测试，该工具在 Windows 07 不能运行，在 XP 的 `SP2`、`SP3` 以及 `2003` 上是可以运行的。

### 上传的附件

| 图片 | 文件 |
| :--- | :--- |
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) | PECompact2 变形工具.rar |
