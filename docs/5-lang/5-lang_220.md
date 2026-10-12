
# 追踪 MFC 的按钮事件学习笔记

> 原文：[`www.pediy.com/kssd/pediy10/91742.html`](https://www.pediy.com/kssd/pediy10/91742.html)

追踪 MFC 的按钮事件学习笔记

阅读了“MFC 程序逆向 消息篇(上)+(下)”一文后，收获颇丰。作者提到对于对话框程序，在这个函数入口处设置断点最好(请记住这个函数：`CCmdTarget::OnCmdMsg()`)。在学习的过程中，对 `OD` 的一些使用方法进一步加深了理解。

首先用 `VC6.0` 建立一个基于对话框的 MFC 程序，分别采用动态链接（`TraceButtonDynamic.exe`）和静态链接（`TraceButtonStatic.exe`）生成。

## 1. 动态链接的程序分析

（1）首先用 `IDA` 找到 `CCmdTarget::OnCmdMsg()` 的地址。在我的计算机中为 `0040157C`。

![](img/075fbedc722aa3b6ab1eff1c3298e1a5.jpg)

（2）用 `OD` 载入，`CTRL+G`，输入 `0040157C`，来到 `0040157C`。

![](img/1cafd9bba39744ee9711e2c4ad06b2a0.jpg)

可以看到有一 `jmp` 语句，`Enter` 进入。这里就是:`OnCmdMsg()` 的入口处。

![](img/4d562a2594de1698c32da4a98da86806.jpg)

向下翻倒 `push dword ptr [eax+14]`, `F2` 下断，运行程序。

![](img/95b04d262391519a9108aaca6e9d29a3.jpg)

单击“hello,world”按钮，可以看到 `00401480`，就是按钮所处理的东西。

![](img/1a7a3bf93b5807ce9f4f2d033e3e29c3.jpg)

`CTRL+G`，输入 `00401480`，来到 `00401480`。OK,搞定。

![](img/59e721fe950331d676ecaf5946c68c52.jpg)

（3）更简单的方法。

重新载入 `TraceButtonDynamic.exe`。`Alt+ E`，载入 DLL 模块，双击 `MFC42`。

![](img/ccf28b3a1a4ea93358aece604157b053.jpg)

来到 `MFC42.dll`

![](img/14dffba51e89227482c1f481c65f8729.jpg)

`CTRL+S`，输入

```
push dword ptr [ebp+14]
push dword ptr [eax+10]
push dword ptr [ebp+10]
push dword ptr [eax+14]
push dword ptr [ebp+C]
push dword ptr [ebp+8]
```

单击查找。

![](img/4325596313a1316379c846cfdd7afe4b.jpg)

![](img/c12f4b684ded77c87cabfdaf91ca54f1.jpg)

在“`push dword ptr [eax+14]`”, `F2` 下断即可

## 2. 静态链接的程序分析

`Od` 载入。`CTRL+S`,输入指令序列。

![](img/c2ca3f3b9e44502a12465a028de667fb.jpg)

单击查找，在“`push dword ptr [eax+14]`”, `F2` 下断即可。

![](img/9a98e5348c858cef010e500f1ea9f16d.jpg)

当然还可以在语句序列下面的 `call` 处 `F7` 进入(本例为 `call 004157F5`)。进入后，直接在“`call dword ptr [ebp+14]`”这条语句出下断。当然可以用搜索的方法。

![](img/e4769a7c7bd42b36001dd3fa9fdae21f.jpg)

以上是本人的一些学习心得,不妥之处，敬请指正。

2009-09-17
By littlewisp

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | | `TraceButton.rar` |
| :--- | :--- | :--- |
