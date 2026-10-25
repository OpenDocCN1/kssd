
# WinDbg 的几个默认工作空间（窗口布局）

> 原文：[`www.pediy.com/kssd/pediy10/100391.html`](https://www.pediy.com/kssd/pediy10/100391.html)

## 一 简介

发现 `windbg` 的工作空间来保持窗口布局很方便和和谐，不在为各窗口的拖来拖去而纠结了，就做了几个默认调试环境的工作空间（主要关注窗口布局）。

现在拿出来分享下，喜欢的朋友下载附近，直接把里面的 `reg` 文件导入注册表就好了，当你以某个模式调试的时候会自动加载！一般只使用命令的大牛飘过~~~~

## 二 效果图

**突出 Command**：

![](img/3acdcf9ed5b53dfe00a46d2357182fd1.jpg)

**OD-Style**：

![](img/67736c7f0c19e4db10d8e1e4e400076b.jpg)

## 三 附件说明

`Kernel.reg`：当进行内核态调试的时候自动加载这个工作空间

`user.reg`：当 `Attach to a Process` 进行调试的时候加载这个

`Expicit.reg`：两个命名空间，通过手动 `File->Open Workspace` 选一个加载使用，当用户态调试的时候可以用他们

`Base.reg`：这个可以不导入，它为基本的工作空间，不管什么模式最先会自动加载这个（可作为用户态下通过 `File->Open Executable` 进行调试的时候，

或者上面的都不导入只导入这个在所有模式下使用），然后在加载相应的默认模式下的工作空间。

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || Windbg WorkSpace.rar |
