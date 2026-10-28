
## 1. `ShowWindow`函数

```c
ShowWindow(
    hWnd: HWND; {要显示的窗口的句柄}
    nCmdShow: Integer {选项, 参见下表}
): BOOL;
//nCmdShow 参数可选值:
SW_HIDE = 0; {隐藏, 并且任务栏也没有最小化图标}
SW_SHOWNORMAL = 1; {用最近的大小和位置显示, 激活}
SW_NORMAL = 1; {同 SW_SHOWNORMAL}
SW_SHOWMINIMIZED = 2; {最小化, 激活}
SW_SHOWMAXIMIZED = 3; {最大化, 激活}
SW_MAXIMIZE = 3; {同 SW_SHOWMAXIMIZED}
SW_SHOWNOACTIVATE = 4; {用最近的大小和位置显示, 不激活}
SW_SHOW = 5; {同 SW_SHOWNORMAL}
SW_MINIMIZE = 6; {最小化, 不激活}
SW_SHOWMINNOACTIVE = 7; {同 SW_MINIMIZE}
SW_SHOWNA = 8; {同 SW_SHOWNOACTIVATE}
SW_RESTORE = 9; {同 SW_SHOWNORMAL}
SW_SHOWDEFAULT = 10; {同 SW_SHOWNORMAL}
SW_MAX = 10; {同 SW_SHOWNORMAL}
```

## 2. `STARTUPINFO`

```c
WORD cbReserved2; //保留。必须被初始化为0
PBYTE lpReserved2; //保留。必须被初始化为NULL
HANDLE hStdInput; //用于设定供控制台输入和输出用的缓存的句柄。按照默认设置，hStdInput用于标识键盘缓存，hStdOutput和hStdError用于标识控制台窗口的缓存
HANDLE hStdOutput;
HANDLE hStdError;
```

当Windows创建新进程时，它将使用该结构的有关成员。大多数应用程序将要求生成的应用程序仅仅使用默认值。至少应该将该结构中的所有成员初始化为零，然后将`cb`成员设置为该结构的大小：

```c
STARTUPINFO si = { sizeof(si) };
CreateProcess(..., &si, ...);
```

**表4-7 `dwFlags` 使用标志及含义**

| 标志 | 含义 |
| --- | --- |
| `STARTF_USESIZE` | 使用`dwXSize`和`dwYSize`成员 |
| `STARTF_USESHOWWINDOW` | 使用`wShowWindow`成员 |
| `STARTF_USEPOSITION` | 使用`dwX`和`dwY`成员 |
| `STARTF_USECOUNTCHARS` | 使用`dwXCountChars`和`dwYCountChars`成员 |
| `STARTF_USEFILLATTRIBUTE` | 使用`dwFillAttribute`成员 |
| `STARTF_USESTDHANDLES` | 使用`hStdInput`、`hStdOutput`和`hStdError`成员 |
| `STARTF_RUN_FULLSCREEN` | 强制在x86计算机上运行的控制台应用程序以全屏幕方式启动运行 |

另外还有两个标志，即`STARTF_FORCEONFEEDBACK`和`STARTF_FORCEOFFFEEDBACK`，当启动一个新进程时，它们可以用来控制鼠标的光标。由于Windows支持真正的多任务抢占式运行方式，因此可以启动一个应用程序，然后在进程初始化时，使用另一个程序。为了向用户提供直观的反馈信息，`CreateProcess`能够临时将系统的箭头光标改为一个新光标，即沙漏箭头光标：

该光标表示可以等待出现某种情况，也可以继续使用系统。当启动另一个进程时，`CreateProcess`函数使你能够更好地控制光标。当设定`STARTF_FORCEONFEEDBACK`标志时，`CreateProcess`并不将光标改为沙漏。

`STARTF_FORCEONFEEDBACK`可使`CreateProcess`能够监控新进程的初始化，并可根据结果来改变光标。当使用该标志来调用`CreateProcess`时，光标改为沙漏。过2秒后，如果新进程没有调用GUI，`CreateProcess`将光标恢复为箭头。

如果该进程在2秒内调用了GUI，`CreateProcess`将等待该应用程序显示一个窗口。这必须在进程调用GUI后5秒内发生。如果没有显示窗口，`CreateProcess`就会恢复原来的光标。如果显示了一个窗口，`CreateProcess`将使沙漏光标继续保留5秒。如果某个时候该应用程序调用了`GetMessage`函数，指明它完成了初始化，那么`CreateProcess`就会立即恢复原来的光标，并且停止监控新进程。

在结束这一节内容的介绍之前，我想讲一讲`STARTUPINFO`的`wShowWindow`成员。你将该成员初始化为传递给`(w)WinMain`的最后一个参数`nCmdShow`的值。该成员显示你想要传递给新进程的`(w)WinMain`函数的最后一个参数`nCmdShow`的值。它是可以传递给`ShowWindow`函数的标识符之一。通常，`nCmdShow`的值既可以是`SW_SHOWNORMAL`，也可以是`SW_SHOWMINNOACTIVE`。但是，它有时可以是`SW_SHOWDEFAULT`。

当在`Explorer`中启动一个应用程序时，该应用程序的`(w)WinMain`函数被调用，而`SW_SHOWNORMAL`则作为`nCmdShow`参数来传递。如果为该应用程序创建了一个快捷方式，可以使用快捷方式的属性页来告诉系统，应用程序的窗口最初应该如何显示。图4-3显示了运行`Notepad`的快捷方式的属性页。注意，使用`Run`选项的组合框，就能够设定如何显示`Notepad`的窗口。

当使用`Explorer`来启动该快捷方式时，`Explorer`会正确地准备`STARTUPINFO`结构并调用`CreateProcess`。这时`Notepad`开始运行，并且为`nCmdShow`参数将`SW_SHOWMINNOACTIVE`传递给它的`(w)WinMain`函数。

运用这样的方法，用户能够很容易地启动一个应用程序，其主窗口可以用正常状态、最小或最大状态进行显示。

最后，应用程序可以调用下面的函数，以便获取由父进程初始化的`STARTUPINFO`结构的拷贝。子进程可以查看该结构，并根据该结构的成员的值来改变它的行为特性。

```c
VOID GetStartupInfo(LPSTARTUPINFO pStartupInfo);
```

注意虽然Windows文档没有明确地说明，但是在调用`GetStartupInfo`函数之前，必须像下面这样对该结构的`cb`成员进行初始化：

```c
STARTUPINFO si = { sizeof(si) };
GetStartupInfo(&si);
```

## 3. `GetModuleFileName`

获取当前进程已加载模块的文件的完整路径，该模块必须由当前进程加载。

如果想要获取另一个已加载模块的文件路径，可以使用`GetModuleFileNameEx`函数。

```c
GetModuleFileName(
    HMODULE hModule,
    LPTSTR lpFilename,
    DWORD nSize
);
```

- `hModule`（`Long`）：一个模块的句柄。可以是一个`DLL`模块，或者是一个应用程序的实例句柄。如果该参数为`NULL`，该函数返回该应用程序全路径。
- `lpFilename`（`String`）：指定一个字串缓冲区，要在其中容纳文件的用`NULL`字符中止的路径名，`hModule`模块就是从这个文件装载进来的。
- `nSize`（`Long`）：装载到缓冲区`lpFilename`的最大字符数。
- 返回值：如果返回为成功，将在`lpFilename`的缓冲区当中返回相应模块的路径，如果所设的`nSize`过小，那么返回仅按所设置缓冲区大小返回相应字符串内容。如果函数失败，返回值将为0，利用`GetLastError`可获得异常代码。

## 4. `_tcsrchr` 和 `_tcsicmp`

`_tcsrchr`：查找字符串中某个字符最后一次出现的位置。

两个参数：

- 第一个参数：字符串
- 第二个参数：查找的字符
- 返回值：指向最后一次在字符串中出现的该字符的指针，如果要查找的字符在串中没有出现，则返回`NULL`。

`_tcsicmp`：不区分大小写比较。

附常用函数功能简介表：

```text
_tcsclen 大小
_tcscpy 拷贝
_tcscat 加尾
lstrcpyn 带大小拷贝
_tcsicmp stricmp 不区分大小写比较
_tcscmp 区分大小写比较
_tcsnccmp strncmp wcsncmp 区分大小写比较
_ttoi atoi 字母转数字int
_ttol atol 字母转数字long
_tstof atof 字母转数字double
_ltot ltoa long转换成字符
_tcstol wcstol strtol
_stprintf 格式化
_sntprintf _snprintf 格式化
_tprintf printf wprintf 输出
_tcsupr _strupr 大写化
_tcslwr strlwr 小写化
_tcsrchr strrchr 查找最后的字符
_stscanf sscanf 格式化获得数值
```

## 5. `PROCESS_INFORMATION`

在创建进程时相关的数据结构之一，该结构返回有关新进程及其主线程的信息。

```c
typedef struct _PROCESS_INFORMATION {
    HANDLE hProcess;
    HANDLE hThread;
    DWORD dwProcessId;
    DWORD dwThreadId;
} PROCESS_INFORMATION;
```

**该结构体成员含义：**

- `hProcess`：返回新进程的句柄。
- `hThread`：返回主线程的句柄。
- `dwProcessId`：返回一个全局进程标识符。该标识符用于标识一个进程。从进程被创建到终止，该值始终有效。
- `dwThreadId`：返回一个全局线程标识符。该标识符用于标识一个线程。从线程被创建到终止，该值始终有效。

## 6. `CreateProcess`

用来创建一个新的进程和它的主线程，这个新进程运行指定的可执行文件。

```c
//函数原型:
BOOL CreateProcess(
    LPCTSTR lpApplicationName,
    LPTSTR lpCommandLine,
    LPSECURITY_ATTRIBUTES lpProcessAttributes,
    LPSECURITY_ATTRIBUTES lpThreadAttributes,
    BOOL bInheritHandles,
    DWORD dwCreationFlags,
    LPVOID lpEnvironment,
    LPCTSTR lpCurrentDirectory,
    LPSTARTUPINFO lpStartupInfo,
    LPPROCESS_INFORMATION lpProcessInformation
);
```

### 参数：

- `lpApplicationName`：指向一个`NULL`结尾的、用来指定可执行模块的字符串。
