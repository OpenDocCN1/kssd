
# Pocket CHM Pro 5.9 另类破解

> 原文：[`www.pediy.com/kssd/pediy10/63980.html`](https://www.pediy.com/kssd/pediy10/63980.html)

## 研究对象
Pocket CHM Pro 5.9

## 下载地址
自己搜索

## 软件介绍
这是用于在 CHM 格式创建帮助，电子读物或者手册的工具。袖珍的 CHM 能够直接打开和编辑 CHM 文件，和它能够反编译成批的 CHM。

## 涉及工具
`OD`, `PEiD`, `stud_pe`, `WinHex`

## 一、前言

最近在学习 CAD 的 VBA，在明经通道上看到有在线的《ActiveX 和 VBA 参考》，想要 CHM 版的还要花 30 大洋，用 OE 把在线版的拉下来一看，乐了：居然正是完整的 HTM 版！自己编译成 CHM 版的不就 OK 了么！

但是正是这个编译过程让我郁闷了好半天。先找了个 Microsoft HTML Help Workshop 汉化版，一编译就出错；再找个国产软件 QuickCHM 试试看，编译出来的文件很多地方有问题，而且打开工程文件巨慢；最后找到这个 PocketCHMPro，用起来感觉不错，但是未注册版只允许编译 10 个 HTM 页面。于是打算 CRACK 之！

## 二、开始 CRACK：开始郁闷

先用 `PEiD` 查一下主程序 `PocketCHMPro.exe`：Microsoft Visual C++ 6.0 [Overlay] 到底还是 2005 年的东西，无壳，心中窃喜；进入输入注册码窗口，不输入任何东西直接点“OK”，弹出“User ID can not be empty”，居然还有错误提示，心中狂喜；直接用 `OD` 载入，查找字符串参考，没有找到任何有用的东西，字符串加密了？有点郁闷了；下断点 `bp MessageBoxA`，没断下来，再下 `MessageBoxExA`，`GetDlgItemTextA`，`GetDlgItemTextExA` 之类的常用断点，还是没断下来，很郁闷了；直接暂停程序，再切换到程序界面，程序还在正常运行，查看一下任务管理器，居然还是双进程？相当郁闷了；任务管理器上显示新进程路径是“c:\Program Files\Pocket CHM Pro\PocketCHM\_Pro.exe”，到目录下找到这个隐藏的主文件，直接运行弹出“程序执行错误”，极其郁闷了居然还是双进程保护！

## 三、双进程调试：继续郁闷

先用 `OD` 分析一下真正的主程序 `PocketCHM\_Pro.exe`，果然不出所料，所有关键字符串都已经加密，几乎没有调用标准的 API，我所知道的常用断点全部失效，静态分析没有找到有用信息；由于是双进程保护，只好开二个 `OD`，一个调试主程序，另一个附加到 `PocketCHM\_Pro.exe` 进行调试，却发现动态调试找不到下手的地方。看着程序总是在 `MFC.dll` 和 `pceeuilib.dll` 中来回转，我却只能一愁莫展，心里这个郁闷呀！

浪费了一二个小时后突然想到：MFC 程序处理字符串时都会用到 `MFC42.#800_CString::~CString`，`MFC42.#858_CString::operator=` 等之类的函数，用这个用断点会不会有用呢？于是查找 `MFC42.#800_CString::~CString` 参考并在 IAT 处的 JMP 直接下断：

```
0046AB10  jmp     dword ptr [<&MFC42.#800_CString::~CString>]       ;  MFC42.#800_CString::~CString
```

输入用户 ID 和注册码后点击“OK”，终于断下来了！

观察堆栈：

```
0012E9EC   0042E26C  返回到 PocketCH.0042E26C 来自 <jmp.&MFC42.#858_CString::operator=>
```

在 `0042E26C` 处下断，再执行完二个 RET 后来到注册码处理过程：

```
代码:
```
