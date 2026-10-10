
# 常用 Win32 API 函数简介

> 原文：[`www.pediy.com/kssd/tutorial/chap3-3.htm`](https://www.pediy.com/kssd/tutorial/chap3-3.htm)

## 常用 Win32 API 函数简介

现在很多讲 Windows 程序设计的书都是讲基于 `MFC` 库和 `OWL` 库的 Windows 设计，对 Windows 实现的细节都鲜有讨论，而调试程序都是和系统底层打交道，所以有必要掌握一些 `Win32 API` 函数的知识，这样我们可快捷地找出程序调用错在哪？是哪个参数出了问题。

Windows 程序模块包括 `KERNEL`、`USER` 和 `GDI`，其中 `KERNEL` 完成内存管理、程序的装人与执行和任务调度等功能，它需要调用原 MS-DOS 中的文件管理、磁盘输入输出和程序执行等功能；`USER` 是一个程序库，它用来对声音、时钟、鼠标器及键盘输入等操作进行管理；`GDI` 是一功能十分丰富的子程序库，它提供了图形与文字输出、图象操作和窗口管理等各种与显示和打印有关的功能。上述 `KERNEL`、`USER` 和 `GDI` 模块中的库函数可被应用程序调用，也可被其他程序模块调用。把包含库函数的模块称为输出者（export）。你应明白为什么跟踪软件时经常在 `KERNEL32!.text` 和 `USER32.text` 等系统领空转的问题吧。

这里列出几个经常碰到的 `Win32 API` 函数，它们都是存在 Windows 系统核心文件 `KERNEL32.DLL` 中和视窗管理文件 `USER32.DLL` 中。

Windows 函数是区分字符集的：`A` 表示 `ANSI`，`W` 表示 `Wide`，即 `Unicode` (Wide character-set)，前者就是通常使用的单字节方式，但这种方式处理象中文这样的双字节字符不方便，容易出现半个汉字的情况。而后者是双字节方式，方便处理双字节字符。Windows 的所有与字符有关的函数都提供两种方式的版本。尽管你编程时使用 `GetWindowText`，但实际上编译程序会根据设置自动调用 `GetWindowTextA` 或 `GetWindowTextW`。函数的最后一个字母告诉我们函数是使用单字节还是双字节字符串。

### 1. `Hmemcpy` 函数

```
void hmemcpy(hpvDest, hpvSource, cbCopy)

void _huge* hpvDest; // 目的数据地址
const void _huge* hpvSource; // 源数据地址
long cbCopy; // 数据大小 (Bytes)
```

这个函数在 `KERNEL32.DLL` 中，它很常用，俗称万能断点，但一般的编程书籍上很少提到，原因它是底层的东西，没有特殊需要，一般不直接调用。但的确它是很有用的！有意思的是它执行的操作很简单，只是将内存中的一块数据拷贝到另一个地方。

注意：此函数只在 `Windows 9x` 系统上有效，在 `Win NT/2K` 系统上相关的函数是 `memcpy`，但在 `Win NT/2K` 上不同于 `Windows 9x` 上，很少再调用 `memcpy` 来处理数据了，用此函数设断基本上什么也拦不住。

### 2. `GetWindowText` 函数

此函数在 `USER32.DLL` 用户模块中，它的作用是复制指定窗口中的字符到缓冲区。函数原型：

```
int GetWindowText(
HWND hWnd //欲获取文字的那个窗口的句柄
LPTSTR lpString //预定义的一个缓冲区，至少有 cch+1 个字符大小；随同窗口文字载入
int nMaxCount //lpString 缓冲区的长度
);
```

16 位：`GetWindowText`

32 位：`GetWindowTextA`，`GetWindowTextW`

### 3. `GetDlgItemText`

此函数在 `USER32.DLL` 用户模块中，它的作用是返回对话框中某一个窗口的标题或文字。函数原型：

```
UINT GetDlgItemText(
HWND hDlg, // 对话框句柄
int nIDDlgItem, //控制标识符
LPTSTR lpString, // 预定义的一个字符缓冲区
int nMaxCount // 字符缓冲区的长度
);
```

16 位：`GetDlgItemText`

32 位：`GetDlgItemTextA`，`GetDlgItemTextW`

### 4. `MessageBox` 函数

此函数是在 `USER32.DLL` 用户模块中，它的作用创建、显示和操作信息框。函数原型：

```
int MessageBox(
HWND hWnd, //窗口句柄
LPCTSTR lpText, // 信息框中文字的地址
LPCTSTR lpCaption, // 信息框标题地址
UINT uType // 信息框类型
);
```

16 位：`MessageBox`

32 位：`MessageBoxA`，`MessageBoxW`

`4dd466d0e1e0cc123afbe504da2462ca`
