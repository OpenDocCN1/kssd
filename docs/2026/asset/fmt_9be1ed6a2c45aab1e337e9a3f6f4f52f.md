
```
BOOL WINAPI CreatePipe(

          __out   PHANDLE hReadPipe,                        // __out 读取句柄

          __out   PHANDLE hWritePipe,                       // __out 写入句柄

          __in    LPSECURITY_ATTRIBUTES lpPipeAttributes,   // __in SECURITY_ATTRIBUTES结构体指针 加测返回的句柄是否能够被子进程继承，为NULL不能继承 匿名管道必须有这个结构体

          __in    DWORD nSize );                            // 缓冲区大小，参数为0时使用默认大小



typedef struct _SECURITY_ATTRIBUTES {

    DWORD nLength;

    LPVOID lpSecurityDescriptor;

    BOOL bInheritHandle;

} SECURITY_ATTRIBUTES, *PSECURITY_ATTRIBUTES, *LPSECURITY_ATTRIBUTES;
```

`lpPipeAttributes`指向一个`SECURITY_ATTRIBUTES`的结构体指针，其检测返回的句柄是否能够被子进程继承，如果参数为`NULL`，表明不能被继承

子进程与父进程之间的通信必须构建一个这样的结构体，并且该结构体的的第三个成员变量参数必须设置为`True`

这样子进程才可以进程父进程所创建的匿名管道句柄。

## 创建子进程

```
BOOL  CreateProcess( 

        LPCWSTR pszImageName,                   // 指向程序名称以NULL结尾的字符串

        LPCWSTR pszCmdLine,                     // 命令行

        LPSECURITY_ATTRIBUTES psaProcess,       // 创建进程对象设置安全性

        LPSECURITY_ATTRIBUTES psaThread,        // 该进程主线程设置安全性

        BOOL fInheritHandles,                   // *指定父进程创建的子进程是否能够继承父进程对象句柄

        DWORD fdwCreate,                        // 指定控件优先级类和进程创建的附加标记

        LPVOID pvEnvironment,                   // 只想环境块的指针

        LPWSTR pszCurDir,                       // 用来指定子进程当前的路径

        LPSTARTUPINFOW psiStartInfo,            // *指向 StartUpInfo 的结构体的指针，用来指定新进程的主窗口如何显示

        LPPROCESS_INFORMATION pProcInfo );      // ROCESS_INFORMATION 结构体的指针，用来接收关于新进程的标识信息



typedef struct _STARTUPINFOA {

    DWORD   cb;

    LPSTR   lpReserved;

    LPSTR   lpDesktop;

    LPSTR   lpTitle;

    DWORD   dwX;

    DWORD   dwY;

    DWORD   dwXSize;

    DWORD   dwYSize;

    DWORD   dwXCountChars;

    DWORD   dwYCountChars;

    DWORD   dwFillAttribute;

    DWORD   dwFlags;

    WORD    wShowWindow;

    WORD    cbReserved2;

    LPBYTE  lpReserved2;

    HANDLE  hStdInput;  // *

    HANDLE  hStdOutput; // *

    HANDLE  hStdError;  // *

} STARTUPINFOA, *LPSTARTUPINFOA;

typedef struct _PROCESS_INFORMATION {

    HANDLE hProcess;

    HANDLE hThread;

    DWORD dwProcessId;

    DWORD dwThreadId;

} PROCESS_INFORMATION, *PPROCESS_INFORMATION, *LPPROCESS_INFORMATION;
```

创建进程时`fInheritHandles`字段我们需要设置为`true`，继承父进程句柄

`LPSTARTUPINFOW psiStartInfo`结构体中进行如下设置：

```
si.wShowWindow = SW_HIDE;                                 //隐藏CMD进程窗口
si.dwFlags = STARTF_USESTDHANDLES | STARTF_USESHOWWINDOW; //使用标准输出和标准错误输出句柄 | 控制CMD窗口隐藏
si.hStdInput  = m_hReadPipeShell;                         // 将管道赋值 设置标准输入句柄
si.hStdOutput = si.hStdError = m_hWritePipeShell;         // 将管道赋值 设置标准输出、标准错误句柄
```

然后通过`PeekNamedPipe`查询是否有新的数据，以及`ReadFile`进行读取管道中的内容进行读操作，`WriteFile`进行写入管道内容进行操作。

一般是使用`while`循环配套`ReadFile`函数。如果控制台程序暂时没有输出并且没有退出，`ReadFile`函数将一直等待，导致死循环。所以在使用`ReadFile`之前，加入`PeekNamedPipe`函数调用。

## 2 进程监控

### 进行进程枚举有很多方法

```
A：CreateToolhelp32Snapshot()、Process32First()和Process32Next()
B：EnumProcesses()、EnumProcessModules()、GetModuleBaseName()
C：Native Api的ZwQuerySystemInformation
D：wtsapi32.dll的WTSOpenServer()、WTSEnumerateProcess()
```

gh0st使用的最常见的方法A，通过建立进程快照进行遍历进程获取信息

```
HANDLE
WINAPI
CreateToolhelp32Snapshot(
    DWORD dwFlags,      // 用来指定快照中需要返回的对象
    DWORD th32ProcessID // 一个进程ID号，为0可获取所有或当前快照
    );
```

通过函数`CreateToolhelp32Snapshot`获取的快照句柄使用`Process32First`、`Process32Next`遍历所有进程的`PROCESSENTRY32`信息，再通过`GetProcessFullPath`获取进程路径等信息。

### 下面的方法可以获取进程内存列表、模块等信息，不过没有加入到项目中：

#### 获取进程模块信息使用到的 API：

```
    HANDLE WINAPI OpenProcess(
  __in          DWORD dwDesiredAccess,      // 打开的标识
  __in          BOOL bInheritHandle,        // 是否继承句柄
  __in          DWORD dwProcessId           // 被打开的进程句柄
);

    //枚举进程里的模块
   BOOL WINAPI EnumProcessModules(
  __in          HANDLE hProcess,            // 进程句柄
  __out         HMODULE* lphModule,         // 返回进程里的模块
  __in          DWORD cb,                   // 模块的个数
  __out         LPDWORD lpcbNeeded          // 存储的模块的空间大小
);

  //得到模块的名字
  DWORD WINAPI GetModuleFileNameEx(
  __in          HANDLE hProcess,            // 进程的句柄
  __in          HMODULE hModule,            // 模块的句柄
  __out         LPTSTR lpFilename,          // 返回模块的名字
  __in          DWORD nSize                 // 缓冲区大小
);
```

#### 获取进程所有内存信息：

```
//枚举指定进程所有内存块
//assert(hProcess != nullptr);
//参数:
//  hProcess:  要枚举的进程,需拥有PROCESS_QUERY_INFORMATION权限
//  memories:  返回枚举到的内存块数组
//返回:
//  成功返回true,失败返回false.
static bool EnumAllMemoryBlocks(HANDLE hProcess, OUT vector<MEMORY_BASIC_INFORMATION>& memories) {
 // 如果 hProcess 为空则结束运行
 assert(hProcess != nullptr);
 // 初始化 vector 容量
 memories.clear();
 memories.reserve(200);
 // 获取 PageSize 和地址粒度
 SYSTEM_INFO sysInfo = { 0 };
 GetSystemInfo(&sysInfo);
 /*
  typedef struct _SYSTEM_INFO {
    union {
   DWORD dwOemId;     // 兼容性保留
   struct {
     WORD wProcessorArchitecture;   // 操作系统处理器体系结构
     WORD wReserved;    // 保留
   } DUMMYSTRUCTNAME;
    } DUMMYUNIONNAME;
    DWORD     dwPageSize;     // 页面大小和页面保护和承诺的粒度
    LPVOID    lpMinimumApplicationAddress;  // 指向应用程序和dll可访问的最低内存地址的指针
    LPVOID    lpMaximumApplicationAddress;  // 指向应用程序和dll可访问的最高内存地址的指针
    DWORD_PTR dwActiveProcessorMask;   // 处理器掩码
    DWORD     dwNumberOfProcessors;   // 当前组中逻辑处理器的数量
    DWORD     dwProcessorType;    // 处理器类型，兼容性保留
    DWORD     dwAllocationGranularity;     // 虚拟内存的起始地址的粒度
    WORD      wProcessorLevel;    // 处理器级别
    WORD      wProcessorRevision;    // 处理器修订
  } SYSTEM_INFO, *LPSYSTEM_INFO;
 */
 //遍历内存
 const char* p = (const char*)sysInfo.lpMinimumApplicationAddress;
 MEMORY_BASIC_INFORMATION  memInfo = { 0 };
 while (p < sysInfo.lpMaximumApplicationAddress) {
  // 获取进程虚拟内存块缓冲区字节数
  size_t size = VirtualQueryEx(
   hProcess,     // 进程句柄
   p,      // 要查询内存块的基地址指针
   &memInfo,     // 接收内存块信息的 MEMORY_BASIC_INFORMATION 对象
   sizeof(MEMORY_BASIC_INFORMATION32)  // 缓冲区大小
  );
  if (size != sizeof(MEMORY_BASIC_INFORMATION32)) { break; }
  // 内存块属性memInfo保存一些内存块信息可以从这里判断获取
  if (memInfo.Protect == PAGE_EXECUTE_READWRITE)
   if (memInfo.State == MEM_COMMIT)
    if (memInfo.Type == MEM_PRIVATE)
     memories.push_back(memInfo); // 将内存块信息追加到 vector 数组尾部
  // 移动指针
  p += memInfo.RegionSize;
 }
 return memories.size() > 0;
}
```

## 3 注册表监控

通过`RegOpenKeyEx`打卡一个注册表项得要打开项的句柄`PHKEY phkResult` 利用这个句柄来获取子项和信息

```
    LONG WINAPI RegOpenKeyEx(
    _In_   HKEY hKey,   // 需要打开的主键的名称
    _In_opt_  LPCSTR lpSubKey,  // 需要打开的子键的名称
    _In_opt_  DWORD ulOptions,  // 保留 设为零
    _In_   REGSAM samDesired, // 安全访问标记 也就是权限
    _Out_   PHKEY phkResult  // 得到的将要打开键的句柄
    );
```

得到`PHKEY`句柄后使用 API `RegQueryInfoKey` 获取该项信息

```
    LONG WINAPI RegQueryInfoKey(       // 获取某项有关的信息 
    _in          HKEY hKey,             // 已打开项的句柄 或指定一个标准项名 
    _out         LPTSTR lpClass,         // 指定一个字串 用于装载这个注册表项的类名 
    _in_out      LPDWORD lpcClass,       // 指定一个变量 用于装载lpClass缓冲区的长度。一旦返回 它会设为实际装载到缓冲区的字节数量 
                 LPDWORD lpReserved,      // 未用 设为零 
    _out         LPDWORD lpcSubKeys,      // 用于装载（保存）这个项的子项数量的一个变量 
    _out         LPDWORD lpcMaxSubKeyLen,     // 指定一个变量 用于装载这个项最长一个子项的长度。注意这个长度不包括空中止字符 
    _out         LPDWORD lpcMaxClassLen,      // 指定一个变量 用于装载这个项之子项的最长一个类名的长度。注意这个长度不包括空中止字符 
    _out         LPDWORD lpcValues,           // 用于装载这个项的设置值数量的一个变量 
    _out         LPDWORD lpcMaxValueNameLen,     // 指定一个变量 用于装载这个项之子项的最长一个值名的长度。注意这个长度不包括空中止字符 
    _out         LPDWORD lpcMaxValueLen,         // 指定一个变量 用于装载容下这个项最长一个值数据所需的缓冲区长度
    _out         LPDWORD lpcbSecurityDescriptor,   // 装载值安全描述符长度的一个变量 
    _out         PFILETIME lpftLastWriteTime         // 指定一个结构 用于容纳该项的上一次修改时间
);
```

通过`RegQueryInfoKey`获取到`lpcSubKeys`子项数量同于`RegEnumKeyEx`的`DWORD dwIndex`，参数进行循环遍历得到索引项名`LPTSTR lpName`
