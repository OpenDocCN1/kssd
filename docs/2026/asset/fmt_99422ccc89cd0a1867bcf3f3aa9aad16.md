
1. `dll`注入是指向运行中的其它进程强制插入特定的`dll`文件。从技术细节来说，`DLL`注入命令其他进程自行调用`LoadLibrary()` API，加载用户指定的`dll`文件。

2. 当`dll`被加载到进程中以后，就拥有了访问进程内存的权限。（用户可以通过这个来修复程序bug或增加功能等）。

3. `dll`被加载到进程后会自动运行`DllMain`函数。

4. 使用`LoadLibrary()` API加载某个DLL时，该DLL中的`DllMain`函数就会被自动执行。DLL注入的工作原理就是从外部促使目标进程调用`LoadLibrary()` API，所以会强制掉用执行DLL的`DllMain()`函数。

# 二：DLL注入的方法

## (一)：创建远程线程(`CreateRemoteThread`)

本示例把`myhack.dll`注入`Notepad.exe`进程，被注入的`myhack.dll`是用来下载`9beK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4&6K9X3u0D9L8$3N6Q4x3X3g2U0L8$3#2Q4c8e0k6Q4z5e0k6Q4z5o6N6Q4c8e0c8Q4b7V1u0Q4b7U0k6Q4c8e0N6Q4z5f1q4Q4z5o6b7`。

### 1. `hackme.dll`源码使用到的函数解析

在看源码时对照此处进行理解

#### (1) `#pragma comment( comment-type [, commentstring] )`

将描述记录安排到目标文件或可执行文件中去。`comment-type`是下面说明的五个预定义标识符中的一个，用来指定描述记录的类型。可选的`commentstring`是一个字符串文字值用于为一些描述类型提供附加的信息。因为`commentstring`是一个字符串文字值，所以它遵从字符串文字值的所有规则，例如换码字符、嵌入的引号（"）和联接。

当`comment-type`是`lib`的时候

**将一个库搜索记录放置到目标文件中去。** 该描述类型必须有包含你要连接程序搜索的库名（和可能的路径）的`commentstring`参数。因为在目标文件中该库名先于默认的库搜索记录，所以连接程序将如同你在命令行输入这些库一样来搜索它们。你可以在一个源文件中放置多个库搜索记录，每个记录将按照它们出现在源文件中的顺序出现在目标文件中。

**换句话说，就是将静态库导入到你的项目中去。**

#### (2) `ThreadProc`

定义：`ThreadProc`是一个应用程序定义的函数的名称的占位符。作为一个线程的起始地址。在调用`CreateThread`函数时，指定该地址。

原型：

```cpp
DWORD WINAPI ThreadProc(LPVOID lpParameter);
```

参数：`lpParameter`

接收线程传递给函数使用的`CreateThread`函数`lpParameter`参数数据。

返回值：函数应该返回一个值，表示线程函数返回退出码，一般使用0作为返回值。线程创建成功,返回非零值,否则为0。（一个进程可以通过调用`GetExitCodeThread`函数获取由`CreateThread`创建的线程的`ThreadProc`函数的返回值）

#### (3) `GetModuleFileName`

获取当前进程已加载模块的文件的完整路径，该模块必须由当前进程加载。

如果想要获取另一个已加载模块的文件路径，可以使用`GetModuleFileNameEx`函数。

`hModule` Long：一个模块的句柄。可以是一个`DLL`模块，或者是一个应用程序的实例句柄。如果该参数为`NULL`，该函数返回该应用程序全路径。

```cpp
GetModuleFileName(
    HMODULE hModule,
    LPTSTR lpFilename,
    DWORD nSize
);
```

`lpFileName` String：指定一个字串缓冲区，要在其中容纳文件的用`NULL`字符中止的路径名，`hModule`模块就是从这个文件装载进来的

`nSize` Long：装载到缓冲区`lpFileName`的最大字符数

返回值：如果返回为成功，将在`lpFileName`的缓冲区当中返回相应模块的路径，如果所设的`nSize`过小，那么返回仅按所设置缓冲区大小返回相应字符串内容。如果函数失败，返回值将为0，利用`GetLastError`可获得异常代码。

#### (4) `_tcsrchr`

查找字符串中某个字符最后一次出现的位置

两个参数

第一个参数：字符串

第二个参数：查找的字符

返回值：指向最后一次在字符串中出现的该字符的指针，如果要查找的字符再串中没有出现，则返回`NULL`。

#### (5) `_tcscpy_s`

字符拷贝函数，使用的如果是`UNICODE`编码，则采用`wcscpy_s()`函数，如果是多字节编码，则采用`strcpy_s()`函数。

功能：字符串拷贝

后缀`_s`表示使用安全的字符串拷贝函数，防止缓冲区不够大而引起错误。

#### (6) `URLDownloadToFile`

`URLDownloadToFile`，指从指定URL地址读取内容并将读取到的内容保存到特定的文件里的实现方法。

**函数原型：**

```cpp
HRESULT URLDownloadToFile(
    LPUNKNOWN pCaller,
    LPCTSTR szURL,
    LPCTSTR szFileName,
    DWORD dwReserved,
    LPBINDSTATUSCALLBACK lpfnCB
);
```

**参数：**

`pCaller` Pointer to the controlling `IUnknown` interface of the calling Microsoft ActiveX component (if the caller is an ActiveX component). //控件的接口,如果不是控件则为0.

`szURL`

Pointer to a string value containing the URL to be downloaded. Cannot be set to `NULL`

//要下载的url地址,不能为空.

`szFileName`

Pointer to a string value containing the name of the file to create for bits that come from the download.

//下载后保存的文件名.

`dwReserved`

Reserved. Must be set to 0.

//保留字段,必需为0

`lpfnCB`

Pointer to the caller's `IBindStatusCallback` interface. `URLDownloadToFile` calls this interface's `IBindStatusCallback::OnProgress` method on a connection activity, including the arrival of data. `IBindStatusCallback::OnDataAvailable` is never called.

//下载进度状态回调

**返回值:**

Returns one of the following values.

`S_OK` : The download started successfully.

`E_OUTOFMEMORY` : The buffer length is invalid, or there is insufficient memory to complete the operation.

`INET_E_DOWNLOAD_FAILURE` : The specified resource or callback interface was invalid.

#### (7) `DLLMain`

函数原型：

```cpp
BOOL WINAPI DllMain(
    HINSTANCE hinstDLL, // 指向自身的句柄
    DWORD fdwReason,    // 调用原因
    LPVOID lpvReserved  // 隐式加载和显式加载
);
```

静态链接时，或动态链接时调用`LoadLibrary`和`FreeLibrary`都会调用`DllMain`函数。`DllMain`的第二个参数`fdwReason`指明了系统调用`Dll`的原因，它可能是:

`DLL_PROCESS_ATTACH`、`DLL_PROCESS_DETACH`、`DLL_THREAD_ATTACH`、`DLL_THREAD_DETACH`。

**进程映射**

`DLL_PROCESS_ATTACH`

大家都知道，一个程序要调用`Dll`里的函数，首先要先把`DLL`文件映射到进程的地址空间。要把一个`DLL`文件映射到进程的地址空间，有两种方法：静态链接和动态链接的`LoadLibrary`或者`LoadLibraryEx`。

当一个`DLL`文件被映射到进程的地址空间时，系统调用该`DLL`的`DllMain`函数，传递的`fdwReason`参数为`DLL_PROCESS_ATTACH`，这种调用只会发生在第一次映射时。如果同一个进程后来为已经映射进来的`DLL`再次调用`LoadLibrary`或者`LoadLibraryEx`，操作系统只会增加`DLL`的使用次数，它不会再用`DLL_PROCESS_ATTACH`调用`DLL`的`DllMain`函数。不同进程用`LoadLibrary`同一个`DLL`时，每个进程的第一次映射都会用`DLL_PROCESS_ATTACH`调用`DLL`的`DllMain`函数。

可参考`DllMainTest`的`DLL_PROCESS_ATTACH_Test`函数。

**进程卸载**

`DLL_PROCESS_DETACH`

当`DLL`被从进程的地址空间解除映射时，系统调用了它的`DllMain`，传递的`fdwReason`值是`DLL_PROCESS_DETACH`。当`DLL`处理该值时，它应该执行进程相关的清理工作。

那么什么时候`DLL`被从进程的地址空间解除映射呢？两种情况：

◆`FreeLibrary`解除`DLL`映射（有几个`LoadLibrary`，就要有几个`FreeLibrary`）

◆进程结束而解除`DLL`映射，在进程结束前还没有解除`DLL`的映射，进程结束后会解除`DLL`映射。（如果进程的终结是因为调用了`TerminateProcess`，系统就不会用`DLL_PROCESS_DETACH`来调用`DLL`的`DllMain`函数。这就意味着`DLL`在进程结束前没有机会执行任何清理工作。）

注意：当用`DLL_PROCESS_ATTACH`调用`DLL`的`DllMain`函数时，如果返回`FALSE`，说明没有初始化成功，系统仍会用`DLL_PROCESS_DETACH`调用`DLL`的`DllMain`函数。因此，必须确保清理那些没有成功初始化的东西。

可参考`DllMainTest`的`DLL_PROCESS_DETACH_Test`函数。

**线程映射**

`DLL_THREAD_ATTACH`

当进程创建一线程时，系统查看当前映射到进程地址空间中的所有`DLL`文件映像，并用值`DLL_THREAD_ATTACH`调用`DLL`的`DllMain`函数。

新创建的线程负责执行这次的`DLL`的`DllMain`函数，只有当所有的`DLL`都处理完这一通知后，系统才允许进程开始执行它的线程函数。
