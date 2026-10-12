
# 3 个脱壳相关的重要函数介绍

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-561.htm`](https://www.pediy.com/kssd/pediy08/pediy8-561.htm)

脱壳相关的重要函数介绍

【QduWg】翻译

声明：本人翻译此类文章，原来是为了自己阅读方便，想到其他人可能也需要，不过 E 文可能比较难理解，故本人贴出来供各位菜鸟学习吧。错误之处请指出。祝学习进步！^\_\*

## 1. `GetModuleHandle` 函数

如果文件已经被映射进了调用进程的地址空间，则此函数返回指定模块的句柄，

```
HMODULE GetModuleHandle(
  LPCTSTR lpModuleName   // 需要返回句柄的模块名的地址
);
```

**参数：**

`lpModuleName` 指向一个包含模块名字的以 0 结尾的串（或者是 `a.DLL` 或者 `.EXE` 文件）。如果函数扩展名省略掉，默认为库文件扩展名 `.DLL`。文件名串可以尾部可以是一个点(.)，指明模块没有扩展名。该串不必指定路径。当指定路径时务必用反斜杠(`\`), 而不是正斜杠(`/`). 名字与映射进进程地址空间的模块比较。如果此参数为空 `NULL`, `GetModuleHandle` 返回创建调用进程的文件句柄。

**返回值：**

如果函数成功调用，返回值是指定模块的句柄。如果函数失败，返回 `NULL`。要得到更进一步的错误信息，调用 `GetLastError` 函数。

**备注：**

返回的句柄不是全局的，可继承的或者可复制的，它不可以被另外的进程使用。

由 `GetModuleHandle` 和 `LoadLibrary` 返回的句柄可以在同一个函数内使用（例如 `GetProcAddress`, `FreeLibrary`, 或者 `LoadResource`）吗？两个函数之间的区别在于引用次数（reference count）。`LoadLibrary` 把模块映射进调用进程的地址空间内，映射进地址空间后，如果必要，则增加模块的引用次数。 然而 `GetModuleHandle` 却是在不增加引用次数的情况下返回已映射模块的句柄。

注意：引用次数被 `FreeLibrary` 用来决定是否把一个函数从进程地址空间释放掉。由于这个原因，当把由 `GetModuleHandle` 返回的句柄用于 `FreeLibrary` 时，必须小心，因为如此以来可能导致一个 DLL 模块被提前释放掉。

在多线程应用里，要小心使用这个函数。因为在一个函数返回句柄和另一个函数使用句柄期间，无法保证模块句柄保持有效。比如一个线程可能通过 `GetModuleHandle` 提取模块句柄。在线程于某个函数内使用句柄之前，第二个线程可能释放了句柄，系统可能加载了另一个模块，而且给它赋予和刚释放的模块相同的句柄值。 第一个线程则被留下一个引用了不同模块的句柄，此模块并非它想要的那个模块。

## 2. `GetProcAddress` 函数

`GetProcAddress` 函数返回指定的输出动态链接库内的函数地址。

```
FARPROC GetProcAddress(
  HMODULE hModule,    // DLL 模块句柄
  LPCSTR lpProcName   // 函数名
);
```

**参数：**

(1) `hModule`

`Handle` 是包含函数的 DLL 模块的句柄。该句柄由 `LoadLibrary` 或 `GetModuleHandle` 返回。

(2) `lpProcName`

指向以 0 结尾的函数名字符串，或者指定函数的序号值。如果参数是序号值，必须出现在低字部分，高字部分置 0。

**返回值：**

如果函数成功，返回 DLL 输出函数的地址。如果失败，返回 `NULL`。得到更多错误信息，使用 `GetLastError` 函数.

**备注：**

`GetProcAddress` 函数用来提取在 DLL 内的输出函数的地址。由 `lpProcName` 指向的函数名的拼写和大小写必须与 DLL 模块定义文件(`.DEF`) 的 `EXPORTS` 声明的函数一致。Win32 API 函数的输出名也许与你代码内调用的函数名不一样。这些差异被 SDK 头文件内使用的宏隐藏起来了。

`lpProcName` 参数可以通过指定一个函数序号（在输出声明中关联到一个函数上）来表明一个 DLL 函数。`GetProcAddress` 校验指定的序号是否在 `.DEF` 文件声明的从 1 和最高序号值之间。 然后函数使用序号作为索引从一个函数表内读取函数地址。如果 `.DEF` 文件没有对函数连续从 1 到 N 编号的话，错误产生了，该函数返回一个无效的非 0 地址，即使没有函数使用指定的序号。在函数不操作的情况下，函数必须以名字指出而非序号。

## 3. `LoadLibrary` 函数

`LoadLibrary` 函数把指定的可执行模块映射进调用进程的地址空间。

```
HINSTANCE LoadLibrary(
  LPCTSTR lpLibFileName   //可执行模块的文件名
);
```

**参数：**

`lpLibFileName` 是指向以 0 结尾的字符串命名的可执行模块。如 `a.DLL` 或 `.EXE`。指定的名字是模块的文件名，与保存在库模块文件的由 `.DEF` 文件内的 `LIBRARY` 关键字指定的名字无关。

如果文件不在指定的目录上，函数失败。当指定路径时，必须使用反斜杠(`\`),而非正斜杠(`/`).

如果没有指定路径，函数就使用标准搜索策略寻找函数。

**返回值：**

如果函数成功，返回值为模块句柄。如果函数失败，则返回 `NULL`。得到更多错误信息，使用 `GetLastError` 函数.

**备注：**

`LoadLibrary` 可以用来把 DLL 模块映射进内存并返回一个可以被 `GetProcAddress` 函数使用的句柄，以得到 DLL 函数的地址。`LoadLibrary` 也可用来映射其他可执行模块，例如可以得到一个 `.EXE` 文件句柄，该句柄用于 `FindResource` 或者 `LoadResource` 函数。不要使用 `LoadLibrary` 来“运行”一个 `.EXE` 文件。

如果模块是一个没有映射进调用进程内存空间的 DLL，系统用 `DLL_PROCESS_ATTACH` 来调用 DLL 的 `DllMain` 函数。如果 DLL 的入口点函数没有返回 `TRUE`, `LoadLibrary` 失败并返回 `NULL`. 从 `DllMain` 调用 `LoadLibrary` 是不安全的。

模块句柄不是全局的和可继承的。一个进程对 `LoadLibrary` 的调用可以产生被另一个进程使用的句柄吗？不可以。另外一个进程在调用 `GetProcAddress` 之前，必须调用自己的 `LoadLibrary` 得到句柄。

如果在 `lpLibFileName` 指定的字符串内没有扩展名，则把默认的库扩展名 DLL 追加上。文件名串可以用点(.)结尾，指明该模块没有扩展名。当没有路径指定时，该函数搜索已调入模块，而且其主名与即将调入的模块主名匹配。如果名字匹配，加载成功，否则，函数按照如下顺序搜索。

⑴应用程序加载的目录；

⑵当前目录；

⑶Windows 95 and Windows 98 系统目录，用 `GetSystemDirectory` 函数得到目录路径；

⑷Windows NT 系统目录，用 `GetSystemDirectory` 函数得到目录路径，目录名为 `SYSTEM32`；

Windows NT: 16 位系统目录，没有函数得到该目录。但它被搜索，目录名是 `SYSTEM`.

Windows 目录：该目录是在环境变量内用 `PATH` 列出的目录。被搜索的第一个目录是含有用于创建调用进程的映象文件的目录。这样做允许私有 DLL 文件与已经发现的进程关联，而不必添加进程的已安装目录到环境变量的 `PAHT` 里。

Visual C++ 编译器提供允许声明线程局部变量的语法：`_declspec(thread)`。如果在 DLL 内使用这个语法，你将不能够显式地用 `LoadLibrary` 或者 `LoadLibraryEx` 函数了。如果你的 DLL 显式地加载，你必须使用线程局部存储函数，而非 `_declspec(thread)`.

Windows 95: 如果用 `LoadLibrary` 加载一个包含 ID 大于 `0x7FFF` 资源的模块将失败。

Windows CE: 两个不同的模块不能够使用相同文件名。例如，试图用 `LoadLibrary` 加载 `"Sample.cpl"`, 操作系统不会加载 `Sample.cpl`, 而是加载的 `Sample.dll`。一个类似的限制存在于使用相同名字但位于不同目录的情况，如：`LoadLibrary` 加载 `"\\Windows\Sample.dll"`，`LoadLibrary` 加载 `"\\MyDir\Sample.dll"`，`"\\Windows\Sample.dll"` 将被重新加载

可执行模块的搜索路径不可以指定，除非给出模块的完全路径。

**另外参考：**

Dynamic-Link Libraries Overview, Dynamic-Link Library Functions, `DllMain`, `FindResource`, `FreeLibrary`, `GetProcAddress`, `GetSystemDirectory`, `GetWindowsDirectory`, `LoadLibraryEx`, `LoadResource`

翻译：Qduwg

2006 年 1 月 9 日
