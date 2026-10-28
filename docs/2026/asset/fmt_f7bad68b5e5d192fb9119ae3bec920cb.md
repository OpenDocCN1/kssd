
```
LONG WINAPI RegEnumKeyEx(             // 枚举指定项下方的子项 

    _in          HKEY hKey,           // 一个已打开项的句柄，或者指定一个标准项名 

    _in          DWORD dwIndex,        // 欲获取的子项的索引。第一个子项的索引编号为零 

    _out         LPTSTR lpName,        // 用于装载指定索引处项名的一个缓冲区

    _in_out      LPDWORD lpcName,         // 指定一个变量，用于装载lpName缓冲区的实际长度，含空字符。一旦返回，它会设为实际装载到lpName缓冲区的字符数量 

                 LPDWORD lpReserved,      // 未用，设为零

    _in_out      LPTSTR lpClass,          // 项使用的类名

    _in_out      LPDWORD lpcClass,        // 用于装载lpClass缓冲区长度的一个变量。

    _out         PFILETIME lpftLastWriteTime     // 枚举子项上一次修改的时间

);
```

使用 API `RegEnumValue` 获取键值内容，以及获取 `lpType` 判断类型、`lpData` 获取内容。

```
LONG WINAPI RegEnumValue(         // 读取键值

    _In_ HKEY hKey,          // 一个已打开项的句柄，或者指定一个标准项名

    _In_ DWORD dwIndex,          // 欲获取值的索引。注意第一个值的索引编号为零

    _Out_writes_to_opt_(*lpcchValueName,*lpcchValueName + 1) LPSTR lpValueName,   // 用于装载位于指定索引处值名的一个缓冲区

    _Inout_ LPDWORD lpcchValueName,        // 用于装载lpValueName缓冲区长度的一个变量。一旦返回，它会设为实际载入缓冲区的字符数量

    _Reserved_ LPDWORD lpReserved,        // 未用 设为零

    _Out_opt_ LPDWORD lpType,         // 用于装载值的类型代码的变量

    _Out_writes_bytes_to_opt_(*lpcbData, *lpcbData) __out_data_source(REGISTRY) LPBYTE lpData, // 用于装载值数据的一个缓冲区

    _Inout_opt_ LPDWORD lpcbData        // 用于装载lpData缓冲区长度的一个变量。一旦返回，它会设为实际载入缓冲区的字符数量

    );
```

## 4 服务监控

建立一个连接到服务控制管理器，并打开指定的数据库。

```
SC_HANDLE WINAPI OpenSCManager(          

  __in          LPCTSTR lpMachineName,         // 指向零终止字符串 名为目标计算机

  __in          LPCTSTR lpDatabaseName,        // 指向零终止字符串 名称的服务控制管理数据库

  __in          DWORD dwDesiredAccess           // 指定服务的访问控制管理

);
```

由 `OpenSCManager` 返回的句柄做参数，使用 API `EnumServicesStatus` 枚举系统当前服务信息。

```
BOOL WINAPI EnumServicesStatus(             // 枚举当前系统服务

  _in          SC_HANDLE hSCManager,       // 打开的服务管理的句柄

  _in          DWORD dwServiceType,        // 所要枚举服务的类型

  _in          DWORD dwServiceState,       // 所要枚举服务的状态

  _out         LPENUM_SERVICE_STATUS lpServices,   // 指向转载枚举服务的缓冲区

  _in          DWORD cbBufSize,                    // 缓冲区大小

  _out         LPDWORD pcbBytesNeeded,             // 如果提供的缓冲区太小 那么这里将返回需要的缓冲区大小

  _out         LPDWORD lpServicesReturned,         // 服务的个数 枚举每个服务信息时用到

  _in_out      LPDWORD lpResumeHandle             // 返回枚举是否成功

);
```

通过 API `EnumServicesStatus` 得到的 `lpServicesReturned` 数量、`lpServices` 缓冲区遍历服务，`OpenService` 获取服务句柄。

```
SC_HANDLE OpenService(   // 获取服务句柄

  SC_HANDLE hSCManager,  // 服务控制管理器数据库的句柄

  LPCSTR    lpServiceName, // 要打开的服务的名称

  DWORD     dwDesiredAccess // 访问服务权限

);
```

根据获取到的服务句柄，调用 API `QueryServiceConfig` 获取服务信息。

```
BOOL WINAPI QueryServiceConfig(

  _in          SC_HANDLE hService,                         // 指向要检索的服务

  _out         LPQUERY_SERVICE_CONFIG lpServiceConfig,   // 指向包含服务信息的缓冲区指针

  _in          DWORD cbBufSize,                          // 缓冲区大小

  _out         LPDWORD pcbBytesNeeded                    // 实际需要的缓冲区大小

);
```

## 5 键盘监控

### 键盘钩子

windows系统是建立在事件驱动的机制上，整个系统都是通过消息传递来实现的，而钩子是windows系统中非常重要的系统接口，用它可以截获并处理发送给其他进程的消息来实现诸多功能，钩子种类很多，每种钩子可以截取相应的消息，例如键盘钩子截取键盘消息等等。

全局钩子运行机制，通过系统调用，将钩子挂入系统，每当特定消息发出，在消息没有到达目标窗口之前，钩子就会先行捕获到消息。这时钩子回调函数可以对消息进行操作，然后继续传递该消息，也可结束该消息的传递。每种类型的钩子都会由系统来维护一个钩子链，并且最后安装的钩子在链子的开始，最先安装的在最后。实现win32的系统钩子，必须调用API函数`SetWindowsHookEx`来安装这个函数。

### 安装钩子

```
HHOOK WINAPI SetWindowsHookEx(

__in int idHook,            \\ 钩子类型

__in HOOKPROC lpfn,         \\ 回调函数地址 

__in HINSTANCE hMod,        \\ 实例句柄 (包含钩子函数的模块句柄)

__in DWORD dwThreadId);     \\ 线程ID (指定监视的线程,如果指定确定的线程，即为线程专用钩子；如果指定为空，即为全局钩子。)

```

几点需要说明的地方：

（1）如果对于同一事件（如键盘消息）既安装了线程钩子又安装了系统钩子，系统会优先调用线程钩子，然后调用系统钩子。

（2）对同一事件消息可安装多个钩子处理过程，这些钩子处理过程形成了钩子链。处理顺序是先安装的后处理，后安装的先处理。

（3）钩子特别是系统钩子会消耗消息处理时间，降低系统性能。只有在必要的时候才安装钩子，在使用完毕后要及时卸载。

### 定义钩子回调

```
LRESULT CALLBACK HookProc(int nCode ,WPARAM wParam,LPARAM lParam)
```

我们先在钩子函数中实现自定义的功能，然后调用函数 `CallNextHookEx` 把钩子信息传递给钩子链的下一个钩子函数。

```
LRESULT CallNextHookEx( HHOOK hhk, int nCode, WPARAM wParam, LPARAM lParam )
```

参数 `hhk` 是钩子句柄。`nCode`、`wParam` 和 `lParam` 是钩子函数。

当然也可以通过直接返回 `TRUE` 来丢弃该消息，就阻止了该消息的传递。

当不再使用钩子时，必须及时卸载。简单地调用下面的函数即可。

```
BOOL UnhookWindowsHookEx( HHOOK hhk)
```

值得注意的是，线程钩子和系统钩子的钩子函数的位置有很大的差别。线程钩子一般在当前线程或者当前线程派生的线程内，而系统钩子必须放在独立的动态链接库中。

## 6 窗口监控

`EnumWindows` 可以遍历当前屏幕上所有的父窗口，创建 `lpEnumFunc` 回调函数遍历每一个顶层窗口。

```
 BOOL EnumWindows(              // 枚举桌面上的所有窗口 

    _In_ WNDENUMPROC lpEnumFunc,       // 回调函数，自己定义 当枚举到窗口时系统就会调用这个函数

    _In_ LPARAM lParam                 // 向回调函数传递的一个参数

    );
```

回调函数的返回值必须为 `TRUE` 才能保证系统会依次遍历每一个窗口。如果返回值非 `TRUE`，则在当前窗口后不会进行后续的遍历动作。

```
BOOL CALLBACK EnumWindowsProc(         // 回调函数的定义  

    _In_ HWND hwnd,                   // 系统传递进来的窗口句柄

    _In_ LPARAM lParam                // 传递进来的那个参数

);
```

`GetWindowText`、`GetWindowThreadProcessId` 可以通过遍历到的 `HWND` 得到对应 window 的 `Title`、`PID`。

不过这里有个问题，在我自己写的 demo 调用 dll 枚举窗口时，遍历获取信息没有发生问题，当使用 `rundll32` 调用导出出现了问题。

在某一个窗口句柄调用 `GetWindowText` 时，出现阻塞，无法返回。

经过调试分析发现：

```
调用GetWindowText的进程 == 目标窗口所属于的进程 && 调用GetWindowText的线程 != 目标窗口所属于的线程
```

此时 `GetWindowText` 将发送 `WM_GETTEXT` 消息至目标窗口所在的线程，线程响应此消息，返回窗口标题。如果目标窗口所在的线程刚好此时无法响应消息，则会导致 `GetWindowText` 一直处于阻塞状态，直到目标窗口所在进程响应了消息，才会得到返回。就会出现没有反应或卡死的情况。

解决方案就是调用 `GetWindowText` 时判断目标窗口所在进程和线程ID，使用 `InternalGetWindowText` 替换 `GetWindowText`。

```
当目标窗口所在进程ID == 调用者所在进程ID && 目标进程所在线程ID != 调用者所在线程ID时->InternalGetWindowText替换GetWindowText
```

## 7 桌面监控

```
HDC CreateDC(              // 得到指定设备名的设备描述表

  LPCTSTR lpszDriver,        // driver name     // 设备名

  LPCTSTR lpszDevice,        // device name      // 特殊设备名

  LPCTSTR lpszOutput,        // not used; should be NULL  // 通常设置为NULL  

  CONST DEVMODE* lpInitData  // optional printer data    // 驱动程序的初始化DEVMODE结构指针

);

HDC CreateCompatibleDC(         // 为设备描述表创建兼容的内存设备描述表

  HDC hdc   // handle to DC         // 设备句柄

);

int GetDeviceCaps(                           // 得到指定设备的信息

  HDC hdc,     // handle to DC                  // 设备句柄

  int nIndex   // index of capability          // 指定要得到那个方面的信息

);

HBITMAP CreateCompatibleBitmap(               // 创建一个与设备描述表兼容的位图

  HDC hdc,        // handle to DC                    // 设备描述表

  int nWidth,     // width of bitmap, in pixels          // 位图的宽度

  int nHeight     // height of bitmap, in pixels         // 位图的高度

);

HGDIOBJ SelectObject(            // 把对象选到内存设备描述表中         

  HDC hdc,          // handle to DC              // 设备描述表

  HGDIOBJ hgdiobj   // handle to object           // 要加入的对象

);

BOOL BitBlt(                                      // 对指定的原设备环境区域中的像素进行位块转换  抓图

  HDC hdcDest,  // handle to destination DC         // 设备对象                 

  int nXDest,   // x-coord of destination upper-left corner     // 目标矩型区域的左上角x坐标

  int nYDest,   // y-coord of destination upper-left corner     // 目标矩形区域的左上角y坐标

  int nWidth,  // width of destination rectangle               // 目标巨型区域的逻辑宽度

  int nHeight,  // height of destination rectangle              // 目标巨型区域的逻辑高度

  HDC hdcSrc,   // handle to source DC                          // 源设备句柄

  int nXSrc,    // x-coordinate of source upper-left corner     // 源矩型区域的左上角x坐标

  int nYSrc,    // y-coordinate of source upper-left corner     // 源矩型区域的左上角y坐标

  DWORD dwRop   // raster operation code                        // 光栅操作代码

);

 // 为屏幕创建设备描述表

 hscrdc = CreateDC("display", NULL, NULL, NULL);



 // 为屏幕设备描述表创建兼容的内存设备描述表

 hmemdc = CreateCompatibleDC(hscrdc);

 // 获得屏幕分辨率

 xscrn = GetDeviceCaps(hscrdc, HORZRES);

 yscrn = GetDeviceCaps(hscrdc, VERTRES);



 // 创建一个与屏幕设备描述表兼容的位图

 hbitmap = CreateCompatibleBitmap(hscrdc, nwidth, nheight);



 // 把新位图选到内存设备描述表中

 holdbitmap = (HBITMAP)SelectObject(hmemdc, hbitmap);



 // 把屏幕设备描述表拷贝到内存设备描述表中

 BitBlt(hmemdc, 0, 0, nwidth, nheight,hscrdc, nx, ny, SRCCOPY);



 // 得到屏幕位图的句柄

 hbitmap = (HBITMAP)SelectObject(hmemdc, holdbitmap);
```

## active启动方式

win7 64下

```
 64位程序注册表位置 HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Active Setup\Installed Components

 32位重定位注册表位置 HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Microsoft\Active Setup\Installed Components
```

例如 `{052860C8-3E53-3D0B-9332-48A8B4971352}`。
