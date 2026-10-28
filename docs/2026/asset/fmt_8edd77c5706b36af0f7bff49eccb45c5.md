
静态链接时，或动态链接时调用 `LoadLibrary` 和 `FreeLibrary` 都会调用 `DllMain` 函数。`DllMain` 的第二个参数 `fdwReason` 指明了系统调用 `Dll` 的原因，它可能是:

`DLL_PROCESS_ATTACH`、

`DLL_PROCESS_DETACH`、

`DLL_THREAD_ATTACH`、

`DLL_THREAD_DETACH`。

## 进程映射

`DLL_PROCESS_ATTACH`

大家都知道，一个程序要调用 `Dll` 里的函数，首先要先把 `DLL` 文件映射到进程的 `地址空间`。要把一个 `DLL` 文件映射到进程的地址空间，有两种方法：`静态链接` 和 `动态链接` 的 `LoadLibrary` 或者 `LoadLibraryEx`。

当一个 `DLL` 文件被映射到进程的地址空间时，系统调用该 `DLL` 的 `DllMain` 函数，传递的 `fdwReason` 参数为 `DLL_PROCESS_ATTACH`，这种调用只会发生在第一次映射时。如果同一个进程后来为已经映射进来的 `DLL` 再次调用 `LoadLibrary` 或者 `LoadLibraryEx`，操作系统只会增加 `DLL` 的使用次数，它不会再用 `DLL_PROCESS_ATTACH` 调用 `DLL` 的 `DllMain` 函数。不同进程用 `LoadLibrary` 同一个 `DLL` 时，每个进程的第一次映射都会用 `DLL_PROCESS_ATTACH` 调用 `DLL` 的 `DllMain` 函数。

可参考 `DllMainTest` 的 `DLL_PROCESS_ATTACH_Test` 函数。

## 进程卸载

`DLL_PROCESS_DETACH`

当 `DLL` 被从进程的 `地址空间` 解除映射时，系统调用了它的 `DllMain`，传递的 `fdwReason` 值是 `DLL_PROCESS_DETACH`。当 `DLL` 处理该值时，它应该执行进程相关的清理工作。

那么什么时候 `DLL` 被从进程的地址空间解除映射呢？两种情况：

◆ `FreeLibrary` 解除 `DLL` 映射（有几个 `LoadLibrary`，就要有几个 `FreeLibrary`）

◆ 进程结束而解除 `DLL` 映射，在进程结束前还没有解除 `DLL` 的映射，进程结束后会解除 `DLL` 映射。如果进程的终结是因为调用了 `TerminateProcess`，系统就不会用 `DLL_PROCESS_DETACH` 来调用 `DLL` 的 `DllMain` 函数。这就意味着 `DLL` 在进程结束前没有机会执行任何清理工作。

注意：当用 `DLL_PROCESS_ATTACH` 调用 `DLL` 的 `DllMain` 函数时，如果返回 `FALSE`，说明没有初始化成功，系统仍会用 `DLL_PROCESS_DETACH` 调用 `DLL` 的 `DllMain` 函数。因此，必须确保清理那些没有成功初始化的东西。

可参考 `DllMainTest` 的 `DLL_PROCESS_DETACH_Test` 函数。

## 线程映射

`DLL_THREAD_ATTACH`

当进程创建一 `线程` 时，系统查看当前映射到进程 `地址空间` 中的所有 `DLL` 文件映像，并用值 `DLL_THREAD_ATTACH` 调用 `DLL` 的 `DllMain` 函数。

新创建的线程负责执行这次的 `DLL` 的 `DllMain` 函数，只有当所有的 `DLL` 都处理完这一通知后，系统才允许进程开始执行它的线程函数。

注意跟 `DLL_PROCESS_ATTACH` 的区别，我们在前面说过，第 `n(n>=2)` 次以后地把 `DLL` `映像文件` 映射到进程的地址空间时，是不再用 `DLL_PROCESS_ATTACH` 调用 `DllMain` 的。而 `DLL_THREAD_ATTACH` 不同，进程中的每次建立 `线程`，都会用值 `DLL_THREAD_ATTACH` 调用 `DllMain` 函数，哪怕是线程中建立线程也一样。

## 线程卸载

`DLL_THREAD_DETACH`

如果 `线程` 调用了 `ExitThread` 来结束线程（线程函数返回时，系统也会自动调用 `ExitThread`），系统查看当前映射到进程空间中的所有 `DLL` 文件映像，并用 `DLL_THREAD_DETACH` 来调用 `DllMain` 函数，通知所有的 `DLL` 去执行线程级的清理工作。

注意：如果线程的结束是因为系统中的一个线程调用了 `TerminateThread`，系统就不会用值 `DLL_THREAD_DETACH` 来调用所有 `DLL` 的 `DllMain` 函数。

## 2. STARTUPINFO

`STARTUPINFO` 用于指定新进程的主窗口特性的一个结构

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 | `typedef struct _STARTUPINFO {` `DWORD cb;` `LPTSTR lpReserved;` `LPTSTR lpDesktop;` `LPTSTR lpTitle;` `DWORD dwX;` `DWORD dwY;` `DWORD dwXSize;` `DWORD dwYSize;` `DWORD dwXCountChars;` `DWORD dwYCountChars;` `DWORD dwFillAttribute;` `DWORD dwFlags;` `WORD wShowWindow;` `WORD cbReserved2;` `LPBYTE lpReserved2;` `HANDLE hStdInput;` `HANDLE hStdOutput;` `HANDLE hStdError;` `} STARTUPINFO,` `*LPSTARTUPINFO;` |

**参数说明：**

`STARTUPINFO` 结构 该结构用于指定新进程的主窗口特性

`DWORD cb;` // 包含 `STARTUPINFO` 结构中的字节数.如果 `Microsoft` 将来扩展该结构,它可用作 `版本控制` 手段，应用程序必须将 `cb` 初始化为 `sizeof(STARTUPINFO)`。

`LPSTR lpReserved;` // 保留。必须初始化为 `NULL`。

`LPSTR lpDesktop;` // 用于标识启动应用程序所在的桌面的名字。如果该桌面存在，新进程便与指定的桌面相关联。如果桌面不存在，便创建一个带有默认属性的桌面，并使用为新进程指定的名字。如果 `lpDesktop` 是 `NULL`（这是最常见的情况),那么该进程将与当前桌面相关联。

`LPSTR lpTitle;` // 用于设定控制台窗口的名称。如果 `lpTitle` 是 `NULL`，则 `可执行文件` 的名字将用作窗口名

`DWORD dwX;` // 用于设定应用程序窗口在屏幕上应该放置的位置的 `x` 和 `y` 坐标（以像素为单位）。

`DWORD dwY;` 只有当子进程用 `CW_USEDEFAULT` 作为 `CreateWindow` 的 `x` 参数来创建它的第一个重叠窗口时,才使用这两个坐标。若是创建控制台窗口的应用程序，这些成员用于指明控制台窗口的左上角

`DWORD dwXSize;` // 用于设定应用程序窗口的宽度和长度（以像素为单位）只有 `dwYsize`

`DWORD dwYSize;` 当子进程将 `CW_USEDEFAULT` 用作 `CreateWindow` 的 `nWidth` 参数来创建它的第一个重叠窗口时，才使用这些值。

`DWORD dwXCountChars;` // 用于设定子应用程序的控制台窗口的宽度和高度（以 `字符` 为单位）

`DWORD dwYCountChars;`

`DWORD dwFillAttribute;` // 用于设定子应用程序的控制台窗口使用的文本和背景颜色

`DWORD dwFlags;` // 请参见下一段和表4 - 7 的说明

`WORD wShowWindow;` // 用于设定如果子应用程序初次调用的 `ShowWindow` 将 `SW_SHOWDEFAULT` 作为 `nCmdShow` 参数传递时，该应用程序的第一个重叠窗口应该如何出现。

本成员可以是通常用于 `ShowWindow` 函数的任何一个 `SW_*` 标识符

|  |  |
| --- | --- |
