
# 检测 OllyDbg 的另类方法

> 原文：[`www.pediy.com/kssd/pediy06/pediy6866.htm`](https://www.pediy.com/kssd/pediy06/pediy6866.htm)

原文链接：[`bbs.pediy.com/showthread.php?s=&threadid=4013`](http://bbs.pediy.com/showthread.php?s=&threadid=4013)

这是 linhanshi 兄放在他的网站上的，我觉得不错就翻译了下，水平很烂，多多包涵！

The other ways to detect OllyDbg 检测 OllyDbg 的另类方法

Pumqara 作/RoBa[TT]译

## 前言

现在是 2004 年了，`RING-3` 级调试器被越来越多地使用，因为它们有图形界面并且比 `RING-0` 级调试器（比如 `SOFTICE`）更加方便。在这篇文章中我将讲述如何检测最好的 `RING-3` 调试器之一——`OllyDbg`。许多人都听说过 `IsDebugerPresent` 和 `fs:[20]` 检测手段，但是有没有其他的新方法呢？下面我向你介绍我自己的一些检测手段。我会给出详细的解释，因此你可以用你自己的想象力来完善它们。

### 方法一：`FindWindow`

这个方法是基于 `FindWindow` 函数。像所有的对话框一样，`OllyDbg` 的主对话框（窗口？）也有它的标题和类名。使用这个 API 函数我们可以判断 `OllyDbg` 的主窗口是否打开。Microsoft 这样写道：

---

`FindWindow` 函数能够获得窗口类名或标题为特定字符串的顶层窗口的句柄。该函数不搜索子窗口。

```
HWND FindWindow(
LPCTSTR lpClassName, //窗口类名的地址
LPCTSTR lpWindowName //窗口标题的地址
);
```

#### 参数说明

`lpClassName`

指向一个以 NULL 结尾的表示窗口类名的字符串的指针，或者是一个标识窗口类名字符串的 atom。如果该参数是一个 atom，它必须是被函数 `GlobalAddAtom` 预先建立的一个全局 atom。这个 16 位的 atom 必须放在 `lpClassName` 的低 8 位，`lpClassName` 的高 8 位必须为 0。

`lpWindowName`

指向一个以 NULL 结尾的表示窗口名称（即标题）的字符串的指针。如果这个参数为 NULL，所有的窗口都被认为是符合条件的。

#### 返回值

如果搜索成功，返回找到的符合条件的窗口句柄。

如果搜索失败，返回值为 NULL。要得到详细的错误信息可以调用 `GetLastError`。

---

我的程序片断：

```
.data
strOllyClsName db "OLLYDBG",0

.code
invoke FindWindow, ADDR strOllyClsName, NULL
cmp eax, 00000000h
jne Olly_Detected
```

### 方法二：`CreateToolhelp32Snapshot`, `Process32First`/`Process32Next`

这是一个有趣的方法。它基于四个 API 函数 (`CreateToolhelp32Snapshot`, `Process32First`, `Process32Next`, `GetCurrentProcessId`) 和一个结构 (`PROCESSENTRY32`)。先看看 MSDN 怎么说：

---

#### `[CreateToolhelp32Snapshot]`

`CreateToolhelp32Snapshot` 函数为指定的进程（可以包括进程使用的堆 `[HEAP]`，模块 `[MODULE]` 和线程 `[THREAD]`）建立一个快照 `[snapshot]`。

```
HANDLE WINAPI CreateToolhelp32Snapshot(
DWORD dwFlags,
DWORD th32ProcessID
);
```

**参数说明：**

`dwFlags`

`[输入]` `TH32CS_INHERIT` - 声明快照句柄是可继承的。

`TH32CS_SNAPALL` - 在快照中包含系统中所有的进程和线程，还包括在 `th32ProcessID` 中指定的进程的堆和模块 `TH32CS_SNAPHEAPLIST` - 在快照中包含在 `th32ProcessID` 中指定的进程的所有的堆。要列举出进程的堆，查看 `Heap32ListFirst`。

`TH32CS_SNAPMODULE` - 在快照中包含在 `th32ProcessID` 中指定的进程的所有的模块。要列举出进程的模块，查看 `Module32ListFirst`。

`TH32CS_SNAPPROCESS` - 在快照中包含系统中所有的进程。要列举出所有进程，查看 `Process32First`.

`TH32CS_SNAPTHREAD` - 在快照中包含系统中所有的线程。要列举出所有线程，查看 `Thread32First`.

`th32ProcessID`

`[输入]` 指定将要抓取的进程 ID。如果该参数为 0 表示抓取当前进程。该参数只有在设置了 `TH32CS_SNAPHEAPLIST`, `TH32CS_SNAPMOUDLE` 或 `TH32CS_SNAPALL` 后才有效，在其他情况下该参数被忽略，所有的进程都会被抓取。

**Return Values**

如果调用成功，返回快照的句柄。

如果调用失败，返回 `INVAID_HANDLE_VALUE`。

**Remarks**

The snapshot taken by this function is examined by the other tool help functions to provide their results. Access to the snapshot is read only. The snapshot handle acts like an object handle and is subject to the same rules regarding which processes and threads it is valid in.

要列举出所有进程的堆和模块状态，指定 `TH32CS_SNAPALL` 并且把 `th32ProcessID` 设为 0。然后，对于快照中每个新增的进程，再次调用 `CreateToolhelp32Snapshot`，设置要抓取的新进程 ID 和 `TH32CS_SNAPHEAPLIST` 或 `TH32CS_SNAPMOULE`.

要删除快照，使用 `CloseHandle` 函数。

---

#### `[Process32First]`

`Process32First` 得到一个系统快照里第一个进程的信息。

```
BOOL WINAPI Process32First(
HANDLE hSnapshot,
LPPROCESSENTRY32 lppe
);
```

**参数说明**

`hSnapshot`

`[输入]` 先前调用 `CreateToolhelp32Snapshot` 函数时返回的指向系统快照的句柄

`lppe`

`[输入，输出]` 指向一个 `PROCESSENTRY32` 结构的指针

**返回值**

如果进程列表的第一个入口已经被复制到缓冲区中则返回 `TRUE`，否则返回 `FALSE`。如果快照中不包含进程信息，通过 `GetLastError` 会返回一个 `ERROR_NO_MORE_FILES` 错误。

**备注**

调用该函数的程序必须把 `PROCESSENTRY32` 中的成员 `dwSize` 设定为该结构的大小（用字节数表示）。`Process32First` 会把 `dwSize` 改为写入该结构的字节数。这个值永远不会比 `dwSize` 的初始值更大，但是有可能更小。如果这个值变小了，任何偏移量大于这个值的成员都是不可靠的。

要得到同一快照中其它进程的信息，使用 `Process32Next` 函数。

---

#### `[Process32Next]`

`Process32Next` 可以得到在一个系统快照中下一个进程的信息。

```
BOOL WINAPI Process32Next(
HANDLE hSnapshot,
LPPROCESSENTRY32 lppe
);
```

**参数说明**

`hSnapshot`

`[输入]` 先前调用 `CreateToolhelp32Snapshot` 函数时返回的指向系统快照的句柄

`lppe`

`[输出]` 指向一个 `PROCESSENTRY32` 结构的指针

**返回值**

如果进程列表的下一个入口已经被复制到缓冲区中则返回 `TRUE`，否则返回 `FALSE`。如果快照中不包含进程信息，通过 `GetLastError` 会返回一个 `ERROR_NO_MORE_FILES` 错误。

**备注**

要得到快照中第一个进程的信息，使用 `Process32First` 函数。

---

#### `[PROCESSENTRY32]`

当一个快照建立后，`PROCESSENTRY32` 描述了在系统地址空间中一系列进程中的一条。

```
typedef struct tagPROCESSENTRY32 {
DWORD dwSize;
DWORD cntUsage;
DWORD th32ProcessID;
ULONG_PTR th32DefaultHeapID;
DWORD th32ModuleID;
DWORD cntThreads;
DWORD th32ParentProcessID;
LONG pcPriClassBase;
DWORD dwFlags;
TCHAR szExeFile[MAX_PATH];
} PROCESSENTRY32, *PPROCESSENTRY32;
```

**成员变量**

`dwSize`

用字节数表示的结构大小。在调用 `Process32First` 前，把这个成员设为 `sizeof(PROCESSENTRY32)`。如果你不对 `dwSize` 进行初始化，`Process32First` 会调用失败。

`cntUsage`

该进程被引用的次数。只有一个进程的引用次数不为 0 时这个进程才存在。一旦它的引用次数为 0，进程就终止了。

`th32ProcessID`

该进程的标识。(ID)

`th32DefaultHeapID`

Identifier of the default heap for the process. The contents of this member has meaning only to the tool help functions. It is not a handle, nor is it usable by functions other than the ToolHelp functions.

`th32ModuleID`

Module identifier of the process. The contents of this member has meaning only to the tool help functions. It is not a handle, nor is it usable by functions other than the ToolHelp functions.

`cntThreads`

该进程启动的线程数目。

`th32ParentProcessID`

建立该进程的父进程的标识。

`pcPriClassBase`

Base priority of any threads created by this process.

`dwFlags`

未使用，保留

`szExeFile`

指向一个以 NULL 结尾的字符串的指针，该字符串说明了进程所属的可执行文件。

在 `Windows Me`/`98`/`95` 中，这个文件名包含路径。

---

#### `[GetCurrentProcessId]`

`GetCurrentProcessId` 返回调用该函数的进程的标识。

```
DWORD GetCurrentProcessId(VOID)
```

**参数说明：**

该函数没有参数。

**返回值**

返回值为调用该函数的进程的标识。

**备注**

这个进程标识（ID）在系统中确定唯一的进程，直到进程终止。

---

我们的目标是检测我们的程序的父进程是不是 `OllyDbg`.

**计划：**

1.) 用 `GetCurrentProcessId` 得到我们程序的进程 ID。

2.) 用 `CreateToolhelp32Snapshot` 和 `Process32First`/`Process32Next` 依次比较每一个 `PROCESSENTRY32` 结构中的 `th32ProcessID` 成员是不是我们的进程 ID，直到找出为止。

3.) 得到 `PROCESSENTRY32` 结构中的 `th32ParentProcessID`。（译注：即父进程 ID）

4.) 用 `CreateToolhelp32Snapshot` 开始一轮新的比较，但这次比较的是每个 `th32ProcessID` 成员是不是我们在第 3 步得到的父进程的 ID，直到找出为止。

5.) 得到 `PROCESSENTRY32` 中的 `szExeFile` 成员，看是不是"Ollydbg.exe"

6.) 如果是的话就知道我们的程序正在 `OllyDbg` 控制下运行了。

示例代码：
