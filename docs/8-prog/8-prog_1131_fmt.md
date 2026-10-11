
# 动态链接库笔记

> 原文：`www.pediy.com/kssd/pediy10/90177.html`

这两天看了下动态链接库，一点笔记，高手莫笑。

Windows 平台下文件名大小写无关。

用 `dumpbin` 工具查看动态链接库导出函数。

Dos 下进入动态链接库当前目录：

```
dumpbin exports dll.dll
```

要在希望导出的函数前加上 `__declspec(dllexport)`。

例：

```
__declspec(dllexport) int add(int a,int b)

{
  return a+b;
}
```

外部定义的函数用 `extern` 声明。

例：

```
extern int add(int a,int b);
```

也可用：

```
__declspec(dllimport) int add(int a,int b)；
```

告诉编译器函数从 `lib` 文件中引出。

为了方便，弄一个头文件 `dll.h`。

```
#ifdef MYDLL_EXPORTS

#define MYDLL_API __declspec(dllexport)

#else

#define MYDLL_API __declspec(dllimport)

#endif

MYDLL_API int add(int a,int b)；
```

输入库 `.lib` 不包含实际代码，仅为编译器提供信息，以便 `exe` 文件动态链接时建立重定位表。

查看可执行文件输入信息，要用到哪些动态链接库。方法如下：

Dos 下进入可执行文件当前目录：

```
dumpbin imports dll.dll
```

也可以用 `Depend` 可视化工具查看，在 `vc6.0` 目录下 `Tools` 中。

导出一个类：

```
Class __declspec(dllexport) Point         //将导出整个类，一般用宏 MYDLL_API 代替__declspec(dllexport)

{
Void output(){};
};
```

如果仅导出成员函数，就仅在成员函数声明前加 `MYDLL_API`，类名前不加。但访问没有不同，虽然 `Point` 没被导出，仍可在目的工程中构造其对象，并利用对象调用其成员函数(`public`)。

`GetForegroundWindow()`得到调用者进程当前窗口句柄。

`C++`改变函数名，可能导致其他平台访问函数失败。

```
#define MYDLL_API “extern C” __declspec(dllimport)

#define MYDLL_API “extern C” __declspec(dllexport)
```

这样仅针对全局函数，成员函数无效。

如果函数调用方式改为非`_cdecl`，以上方法无效。

解决这个问题，用模块定义文件。如下：

建立文本文件，明名为 `xx.def`。

```
EXPORTS

Add

funName
```

表示导出哪些函数。这样不必用`__declspec(dllexport)`。

编译器此时不会改变函数名字。即使改变调用方式。

注意此时调用模块中函数调用方式应作相应改变。

动态加载函数 `LoadLibrary()`；也用它加载可执行模块(为利用其资源)。

`GetProcAddress()`；得到要到处函数地址。

动态加载只需`.dll` 文件。

将整数转化为指向常字符串指针变量，用宏 `MAKEINTRESOURCE`。
