
```
HANDLE WINAPI OpenProcess(

_In_ DWORD dwDesiredAccess, 

_In_ BOOL   bInheritHandle, 

_In_ DWORD dwProcessId

);
```

如果你读过MSDN中相关的文档，那么你应该知道我们需要请求获取一系列特定的访问权限；访问权限的完整列表参阅网址：[01fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7$3c8F1i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8Y4j5W2)9J5c8X3I4A6j5Y4u0S2M7Y4W2Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3k6r3g2K6K9%4c8G2M7q4)9J5c8X3#2K6y4U0R3@1z5o6R3H3i4K6t1^5N6W2)9K6c8s2k6K6i4K6u0W2z5o6g2Q4x3U0W2Q4x3X3g2S2M7%4m8^5](https://bbs.kanxue.com/elink@1f1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7$3c8F1i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8Y4j5W2)9J5c8X3I4A6j5Y4u0S2M7Y4W2Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3k6r3g2K6K9%4c8G2M7q4)9J5c8X3#2K6y4U0R3@1z5o6R3H3i4K6t1^5N6W2)9K6c8s2k6K6i4K6u0W2z5o6g2Q4x3U0W2Q4x3X3g2S2M7%4m8^5)。

这些权限随微软Windows操作系统版本不同而不同；以下调用代码可用于几乎每一种技术之中：

```
HANDLE hProcess = OpenProcess(

PROCESS_QUERY_INFORMATION | 

PROCESS_CREATE_THREAD | 

PROCESS_VM_OPERATION | 

PROCESS_VM_WRITE, 

FALSE, dwProcessId );
```

## 在目标/远程进程空间分配内存

我们调用`VirtualAllocEx()`函数为DLL路径分配内存。正如MSDN中所述，`VirtualAllocEx()`函数“保留，提交或改变指定进程虚拟地址空间中的一块内存区域的状态；函数通过置零来初始化内存。”函数原型如下：

```
LPVOID WINAPI VirtualAllocEx(

  _In_     HANDLE hProcess,

  _In_opt_ LPVOID lpAddress,

  _In_     SIZE_T dwSize,

  _In_     DWORD  flAllocationType,

  _In_     DWORD  flProtect

);
```

基本上，我们进行如下的调用操作：

```
// 计算DLL文件路径名称所需的字节数

DWORD dwSize = (lstrlenW(pszLibFile) + 1) * sizeof(wchar_t);

// 在目标/远程进程中为路径名称分配空间

LPVOID pszLibFileRemote = (PWSTR)VirtualAllocEx(hProcess, NULL, dwSize, MEM_COMMIT, PAGE_READWRITE);
```

或者你可以更聪明一点地调用`GetFullPathName()` API函数；然后，我在整个工程中都没有调用这个API函数，仅仅是出于个人偏好或者是不够聪明。

如果你想要为整个DLL文件分配空间，就必须进行如下操作：

```
hFile = CreateFileW(pszLibFile, GENERIC_READ, 0, NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, NULL);

dwSize, = GetFileSize(hFile, NULL);

PVOID pszLibFileRemote = (PWSTR)VirtualAllocEx(hProcess, NULL, dwSize, MEM_COMMIT, PAGE_READWRITE);
```

## 复制DLL文件路径（或者DLL文件）到目标/远程进程的内存空间中

现在，我们需要调用`WriteProcessMemory()` API函数，来将我们的DLL文件路径，或者整个DLL文件，复制到目标/远程进程中。函数原型如下所示：

```
BOOL WINAPI WriteProcessMemory(

  _In_  HANDLE  hProcess,

  _In_  LPVOID  lpBaseAddress,

  _In_  LPCVOID lpBuffer,

  _In_  SIZE_T  nSize,

  _Out_ SIZE_T  *lpNumberOfBytesWritten

);
```

一般的调用代码如下所示：

```
DWORD n = WriteProcessMemory(hProcess, pszLibFileRemote, (PVOID)pszLibFile, dwSize, NULL);
```

如果我们想要像反射DLL注入技术所做的那样复制整个DLL文件，就需要更多的代码，因为在将其复制到目标/远程进程之前我们需要将其读入内存。具体代码如下所示：

```
lpBuffer = HeapAlloc(GetProcessHeap(), 0, dwLength)

ReadFile(hFile, lpBuffer, dwLength, &dwBytesRead, NULL)

WriteProcessMemory(hProcess, pszLibFileRemote, (PVOID)pszLibFile, dwSize, NULL);
```

正如之前所述，通过使用反射DLL注入技术，以及将DLL文件复制到内存中，进程不会记录该DLL模块。

但这样会有一点复杂，因为当DLL模块加载到内存中时我们需要获取其入口点；反射DLL工程的`LoadRemoteLibraryR()`函数部分为我们完成了这项工作。如有需要请参阅源码。

需要注意的是，我们将注入的DLL文件需要使用适当的包含与选项来进行编译，这样它才能与`ReflectiveDLLInjection`方法相匹配。“InjectAllThings”工程中包含了一个名为`rdll_32.dll/rdll_64.dll`的DLL文件，可用于练习。

## 控制进程来运行DLL文件

### `CreateRemoteThread()`

可以说，`CreateRemoteThread()`是最传统和最流行，以及最多文档资料介绍的DLL注入技术。

它包括以下几个步骤：

1. 使用`OpenProcess()`函数打开目标进程
2. 通过调用`GetProAddress()`函数找到`LoadLibrary()`函数的地址
3. 通过调用`VirtualAllocEx()`函数在目标/远程进程地址空间中为DLL文件路径开辟内存空间
4. 调用`WriteProcessMemory()`函数在之前所分配的内存空间中写入DLL文件路径
5. 调用`CreateRemoteThread()`函数创建一个新的线程，新线程以DLL文件路径名称作为参数来调用`LoadLibrary()`函数

如果你看过MSDN中关于`CreateRemoteThread()`函数的文档，那么你应该知道，我们需要一个指针，“指向将由线程执行的，类型为`LPTHREAD_START_ROUTINE`的应用程序定义函数，并且该指针代表远程进程中线程的起始地址”。

这意味着要运行我们的DLL文件，我们只需要控制进程来做就好（译者注：由下文可知，应该是将`LoadLibrary()`函数作为线程的启动函数，来加载待注入的DLL文件）。很简单。

以下代码即之前所列的全部基本步骤。

```
HANDLE hProcess = OpenProcess(PROCESS_QUERY_INFORMATION | PROCESS_CREATE_THREAD | PROCESS_VM_OPERATION | PROCESS_VM_WRITE, FALSE, dwProcessId);

 

// 在远程进程中为路径名称分配空间

LPVOID pszLibFileRemote = (PWSTR)VirtualAllocEx(hProcess, NULL, dwSize, MEM_COMMIT, PAGE_READWRITE);

 

// 将DLL路径名称复制到远程进程地址空间

DWORD n = WriteProcessMemory(hProcess, pszLibFileRemote, (PVOID)pszLibFile, dwSize, NULL);

 

// 获取Kernel32.dll中的LoadLibraryW函数的真正地址

PTHREAD_START_ROUTINE pfnThreadRtn = (PTHREAD_START_ROUTINE)GetProcAddress(GetModuleHandle(TEXT("Kernel32")), "LoadLibraryW");

 

// 创建远程线程调用LoadLibraryW(DLLPathname)

HANDLE hThread = CreateRemoteThread(hProcess, NULL, 0, pfnThreadRtn, pszLibFileRemote, 0, NULL);
```

完整代码请参阅源文件`t_CreateRemoteThread.cpp`。

### `NtCreateThreadEx()`

另一个选择是使用`NtCreateThreadEx()`函数；这是一个未公开的`ntdll.dll`中的函数，未来可能会消失或改变。这种技术相比而言实现更加复杂，因为我们需要一个结构体（具体如下所示）来向函数传递参数，以及另一个结构体用于从函数接收数据。

```
struct NtCreateThreadExBuffer {

  ULONG Size;

  ULONG Unknown1;

  ULONG Unknown2;

  PULONG Unknown3;

  ULONG Unknown4;

  ULONG Unknown5;

  ULONG Unknown6;

  PULONG Unknown7;

  ULONG Unknown8;

};
```

网址：[a30K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4W2j5%4g2J5K9i4c8&6P5s2m8D9L8$3c8W2k6q4)9J5k6h3y4G2L8g2)9J5c8X3&6@1j5%4u0W2j5i4c8W2N6r3S2J5k6h3q4V1k6i4S2Q4x3X3g2H3K9s2l9`](https://bbs.kanxue.com/elink@d0bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4W2j5%4g2J5K9i4c8&6P5s2m8D9L8$3c8W2k6q4)9J5k6h3y4G2L8g2)9J5c8X3&6@1j5%4u0W2j5i4c8W2N6r3S2J5k6h3q4V1k6i4S2Q4x3X3g2H3K9s2l9`.)处的文章详细介绍了该函数调用。设置部分与`CreateRemoteThread()`非常类似；然而，相较于直接调用`CreateRemoteThread()`函数，我们使用如下代码来调用`NtCreateThreadEx()`。

```
PTHREAD_START_ROUTINE ntCreateThreadExAddr = (PTHREAD_START_ROUTINE)GetProcAddress(GetModuleHandle(TEXT("ntdll.dll")), "NtCreateThreadEx");

 

LPFUN_NtCreateThreadEx funNtCreateThreadEx = (LPFUN_NtCreateThreadEx)ntCreateThreadExAddr;

 

NTSTATUS status = funNtCreateThreadEx(

  &hRemoteThread,

  0x1FFFFF,

  NULL,

  hProcess,

  pfnThreadRtn,

  (LPVOID)pszLibFileRemote,

  FALSE,

  NULL,

  NULL,

  NULL,

  NULL

  );
```

完整代码请参阅源文件`t_NtCreateThreadEx.cpp`。

### `QueueUserAPC()`

除了之前介绍的方法还有一种选择，不用在目标/远程进程中创建一个新的线程，那就是`QueueUserAPC()`函数。

根据MSDN中的文档介绍，该函数“向指定线程的APC队列中添加一个用户态的异步过程调用（APC）对象”。

函数原型与参数说明如下所示。

```
DWORD WINAPI QueueUserAPC(

  _In_ PAPCFUNC  pfnAPC,

  _In_ HANDLE    hThread,

  _In_ ULONG_PTR dwData

);

pfnAPC     [输入参数]

指向应用程序提供的APC函数的指针，该函数在指定线程执行一个可唤醒等待操作的时候被调用。（…）

HThread  [输入参数]

线程句柄。该句柄必须具备THREAD_SET_CONTEXT访问权限。（…）

dwData    [输入参数]

传递给pfnAPC参数所指向的APC函数的一个值
```

因此，如果不想创建我们自己的线程，我们可以调用`QueueUserAPC()`函数来劫持目标/远程进程中一个已存在的线程；即，调用该函数将在指定线程的APC队列中添加一个异步过程调用。

我们可以使用一个真实的APC回调函数，而不使用`LoadLibrary()`。事实上参数可以是指向我们想要注入的DLL文件名称的指针，具体代码如下所示。

```
DWORD dwResult = QueueUserAPC((PAPCFUNC)pfnThreadRtn, hThread, (ULONG_PTR)pszLibFileRemote);
```

如果你想试用这种技术，那么有一点你可能注意到了，即它与微软Windows操作系统执行APC的方式有关。没有能够查看APC队列的调度器，这意味着只有线程设置成可唤醒模式才能够检查队列。
