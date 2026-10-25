
# tElock 脱壳笔记

> 原文：[`www.pediy.com/kssd/pediy10/104251.html`](https://www.pediy.com/kssd/pediy10/104251.html)

## tElock 介绍

`tElock` 是一款免费的加密保护程序。压缩引擎使用 `UPX` 内核，可压缩保护 32 位可执行文件 `EXE`、`DLL` 和 `OCX`。它使用了各种 `Anti-Debug` 技术、`SMC` 技术、重整覆盖、重整 `Reloc`、自建输入表和产生任意区块名等保护技术。

## OEP 查找

1. 工具查找：`peid` 的插件 `Generic OEP Finder`，`AsprLoader`，`oepfinder`
2. 手动查找：二次内存断点法，最后一次异常法，单步跟踪

## IAT 修复

### (1) 解密被外壳加密了的 IAT

`IAT` 修复可以使用 `Import_REC` 插件、`OD` 脚本，或者手动修复。

对于 `tElock 0.98` 加壳的程序，通过跟踪代码，很容易发现：真正的 `API` 地址在一个 `DWORD` 数组里面，在 `OD` 的数据窗口中使用二进制复制这个 `DWORD` 数组（注意要复制最后一个为 `0` 的 `DWORD`），再二进制粘贴到 `IAT` 相应的位置。

对于 `tElock 0.99` 加壳的程序，虽然也可以跟踪代码，找到 `API` 真正的地址，但是其加密方式不再像 `tElock 0.98` 那样简单。如果有很多地址需要修复，最好使用插件或者脚本。（参考老外的一个视频）

### (2) 跳过外壳对 IAT 的加密

直接在 `VirtualAlloc` 的末尾 `F2` 下 `int 3` 断点（不要在函数的开始部分下断点，也不要下硬件断点），或者在过了一段利用 `S.E.H` 清除硬件断点的代码后，再对 `VirtualAlloc` 下硬件执行断点（过了这段代码后就可以下硬件断点了）。

断下后，执行到返回。将条件跳转指令修改为无条件跳转，这样就跳过了 `IAT` 加密。

### (3) 跳过外壳对原输入表的处理

外壳先将各个区段解压还原（此时内存中有完整的输入表），再加密 `IAT`，同时破坏 `IID` 数组。`IAT` 的加密方式是，将 `IAT` 中的指针指向外壳代码空间（这段空间是由外壳动态申请的，抓取内存映像时不会被保存下来），由外壳调用相应的 `API`。

由于 `tElock 0.98` 没有破坏原输入表，如果跳过外壳处理原输入表的代码，在到达 `OEP` 时抓取内存映像，那么只要修改 `PE` 头的几个数据就可以了，不需要再使用 `Import_REC` 重建输入表。

具体步骤：

1. 将 `OD` 的异常选项设置为“忽略所有异常”。对 `GetProcAddress` `F2` 下断，断下来后取消 `int 3` 断点。
2. 返回程序领空，向下查找指令 `add esi, 14`。在下面的 `JMP` 指令上按 `ENTER` 键。
3. 外壳在 `0042F21E` 处，判断原程序是否有输入表。如果有输入表，则 `ESI` 中放的是 `IID` 数组的 `RVA`，否则，`ESI = 0`，外壳就不填充输入表。因此，在 `0042F21E` 处强行跳转，就可以得到包含原程序完整输入表的映像文件。
4. 并且，同版本的 `tElock` 外壳此处的地址与外壳入口点的差值是定值。`tElock 0.98` 是 `0x9B8`。
   `0002FBD6` `9B8 + 00401000 = 0042F21E`
5. 据此，可以得到如下脱壳方法：
   - 计算出关键跳转的 `VA = 0042F21E`。
   - `OD` 设置为忽略除了除零异常外的全部异常。`Shift + F9` 运行，到达除零异常。
   - 在下一条指令上 `F2` 下断。`Shift + F9` 运行。停在 `0042E0A9` 处。`F2` 取消 `int 3` 断点。设置 `OD` 异常选项为忽略所有异常。`Ctrl+G` 来到 `0042F21E`，下硬件执行断点。`Shift + F9` 运行，断在 `0042F21E` 处，取消硬件执行断点。记下此时 `ESI` 的值 `00028000`，然后在寄存器面板中双击 `ZF` 使其为 `1`。`Ctrl+G` 来到 `OEP=00403B70`，下硬件执行断点，或者在内存映像中对代码段 `F2` 下断。`Shift + F9` 运行。程序就会停在 `OEP` 了。此时抓取内存映像为 `dumpd.exe`（注意 `correct image size`）。使用 `LordPE` 修改 `dumpd.exe` 的入口点为 `3B70`，输入表为 `28000`。

## tElock 0.99 与 0.98 的区别

1. `tElock 0.99` 破坏了原输入表，所以无法采用方法 3 完美脱壳。
2. `tElock 0.99` 增加了通过枚举窗口来反调试，而 `0.98` 只有代码自校验、检测一些 `API` 是否被下 `CC` 断点等反调试手段（`OD` 会被关闭 `SendMessage WM_CLOSE`）。

下面是我脱吾爱破解脱壳练习第 6 期 `tElock 0.99` 的步骤（使用原版 `OD`，不加载任何插件）。

1. `OD` 设置为忽略除了除零异常外的全部异常。`Shift + F9` 运行，到达除零异常。
2. 在下一条指令处 `F2` 下断，`Shift + F9` 运行，停在 `004681D2` 处。`F2` 取消断点。
3. 设置 `OD` 为忽略所有异常。单步跟踪程序（注意变形 `JMP`，不要跑飞了）。
4. 直到如下地址，`EDI=00468613` 指向一个结构体，其中第一个 `DWORD` 存放的是缓冲区指针。

相关 `API` 的资料：

```
BOOL EnumWindows(
    WNDENUMPROC lpEnumFunc,
    LPARAM lParam
);

Parameters

lpEnumFunc
[in] Pointer to an application-defined callback function.

lParam
[in] Specifies an application-defined value to be passed to the callback function.

Return Value

If the function succeeds, the return value is nonzero.

If the function fails, the return value is zero. To get extended error information, call GetLastError.

If EnumWindowsProc returns zero, the return value is also zero. In this case, the callback function should call SetLastError to obtain a meaningful error code to be returned to the caller of EnumWindows.

BOOL CALLBACK EnumWindowsProc(
    HWND hwnd,
    LPARAM lParam
);

Parameters

hwnd
[in] Handle to a top-level window.

lParam
[in] Specifies the application-defined value given in EnumWindows or EnumDesktopWindows.

Return Value

To continue enumeration, the callback function must return TRUE; to stop enumeration, it must return FALSE.
```

这段代码的功能是：枚举系统中所有的窗口，如果发现某个窗口的类名为 `OLLY`、`OWL_`、`FileMOnC` 中的一个，则向该窗口发送 `WM_CLOSE` 消息。

过这段代码的方法：在 `0046865E` 处 `F2` 下断，`Shift + F9` 运行。`F2` 取消断点，在 `00468671` 处新建 `EIP`（右键---> `new origin here`）。这样 `OD` 就不会被关闭了。

（使用修改版的 `OD` 或者使用插件隐藏 `OD`，`OD` 不会被关闭）

过了这段代码，`Ctrl+G` 来到 `VirtualAlloc`，下硬件执行断点。`Shift + F9` 若干次，执行到返回，取消硬件断点。将 `00469817` 处的指令用 `NOP` 填充。打开内存映像，对代码段 `F2` 下断。`Shift + F9` 运行，直接到达 `OEP`。并且 `IAT` 没有被加密。然后，用 `Lord_PE` 抓取内存映像（注意 `correct image size`），用 `Import_REC` 重建输入表。

## 参考

- `<<加密与解密 第二版>>`
- [`bbs.pediy.com/showthread.php?t=97661`](http://bbs.pediy.com/showthread.php?t=97661) 【转帖】Unpacking `tElock` 1.0 by `HellSpider`
- [`bbs.pediy.com/showthread.php?t=63158`](http://bbs.pediy.com/showthread.php?t=63158) 【原创】单步跟踪 `tElock` v 0.98
- [`bbs.pediy.com/showthread.php?t=86036`](http://bbs.pediy.com/showthread.php?t=86036) 【原创】手脱 `tElock` 0.98
- [`bbs.chinapyg.com/viewthread.php?tid=50769`](http://bbs.chinapyg.com/viewthread.php?tid=50769) 图文并茂\_ 手脱 `tElock`

## 说明

1. 这里文章没有图。大家可以看附件里面的 `Word` 文档。
2. 我是一个菜鸟。笔记中如有什么不妥的地方，希望各位高手指出。![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")
3. 参考了看雪很多文章，没有全部列出。![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")

by `asdfslw` [`www.pediy.com`]

2009.12.31

【版权声明】本文由 `asdfslw` 原创于看雪软件安全论坛，转载请注明出处，并保留版权声明。
