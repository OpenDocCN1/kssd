
# Ring3注入总结及编程实现【有码】

在科锐一年有余，最近研究了下Ring3下面的各种注入姿势和防御姿势，特地整理了一下，为了练习一下开发所以就边学注入边实验，每学一种注入都用MFC或者汇编开发一个小工具加强印象，感觉可能一些新手朋友可能会用到，水平有限，如有错误，望大家多多赐教。

# Ring3注入总结

### 导入表注入

静态注入的方法。修改PE文件，添加一个新节，修改导入表添加一个新的DLL实现注入。

### 挂起线程注入

`OpenThread`--> `SuspendThread`--> `GetThreadContext`--> 获取EIP--> 修改EIP--> `SetThreadContext`--> `ResumeThread`

### 挂起进程注入

`CreateProcess`注入方法之一，`CREATE_SUSPENDED`以挂起的方式打开进程，写入ShellCode注入。

### 调试器注入

`CreateProcess`注入方法之二，`DEBUG_ONLY_THIS_PROCESS`以调试的方法打开进程，利用`CREATE_PROCESS_DEBUG_EVENT`，向目标程序中写入我们的ShellCode完成相应功能，并且我们的ShellCode中写入以`CC`断点，代码执行指令时触发`EXCEPTION_DEBUG_EVENT`事件，在`EXCEPTION_DEBUG_EVENT`的处理函数中回到原来的执行流程。

### 注册表注入

全局的注入方式：

修改：

`HKEY_LOCAL_MACHINE/Software/Microsoft/WindowsNT/CurrentVersion/Windows/AppInit_DLLs`的键值为我们DLL的路径，只要使用了`user32.dll`的程序都会加载这个目录下的DLL。

### 钩子注入

利用`SetWindowsHookEx`拦截消息进行注入。

### APC注入

APC注入的原理是利用当线程被唤醒时APC中的注册函数会被执行的机制，并以此去执行我们的DLL加载代码，进而完成DLL注入的目的。利用`QueueUserAPC()`可以向APC队列投入`Loadlibrary`函数指针完成注入，其实这种方法配合`CreateProcess`使用注入最为简单，先挂起打开线程，再`QueueUserAPC()`，再恢复线程，完成注入。

### 远程线程注入

老的套路，主要是用到`CreateRemoteThread`这个API，通过它可以打开目标进程中的远程线程，然后跑我们的代码完成注入。

### 输入法注入

利用输入法在工作时需要向进程中加载Ime文件（其实就是个Dll）,我们构造自己的Ime文件，在Ime文件注入对方进程的时候加载我们自己的DLL完成注入

### DLL劫持

简单来说就是自己实现应用程序的某些DLL，完成导出函数，替换DLL实现注入。

不会排版，详细编程思路见文档

# 关于Ring3下的反注入思路

反注入的方法大牛应该都是在Ring0下面玩，我只了解一些三环下的反注入思路，这里只聊3环下的反注入思路。

### 导入表注入

毕竟是静态修改文件的方法注入DLL，可以对自身PE文件求校验值判断是否被修改。

### 钩子注入

(1) `HOOK`自身进程的`LoadLibraryExW`这个函数，判断调用是否来自`user32.dll`，因为钩子注入时`LoadLibraryExW`的调用者为`user32.dll`，`HOOK`关键代码如下

```
HMODULE WINAPI newLoadLibraryExW(LPCWSTR lpLibFileName,HANDLE hFile,DWORD dwFlags)
{
  //get the return address
  DWORD dwCaller;          //ebp+4返回上层调用者的地址
  __asm push dword ptr [ebp+4]
  __asm pop dword ptr [dwCaller]
  if(dwCaller > m_dwUser32Start && dwCaller < m_dwUser32End)
  {
        return FALSE;
  }
        return rawLoadLibraryExW(lpLibFileName,hFile,dwFlags);
}
```

(2) 安装`WH_DEBUG`消息钩子，在`WH_DEBUG`钩子的消息回到中屏蔽消息钩子，回调

```
LRESULT CALLBACK DebugProc(
      int nCode,
      WPARAM wParam,
      LPARAM lParam
)
```

其中第三个参数 `lParam` 指向`DEBUGHOOKINFO`结构体如下:

```
typedef struct
{
    DWORD idThread;  //安装WH_DEBUG钩子的线程ID。
    DWORD idThreadInstaller;  //当前即将被调用的钩子所在的线程ID。
    LPARAM lParam;
    WPARAM wParam;
    int code;
} DEBUGHOOKINFO, *PDEBUGHOOKINFO;
```

判断这两个`idThread`与`idThreadInstaller`是否相等即可判断是自己进程的钩子

### 远程线程注入

当程序被远程线程被注入时候，线程的入口点一定为`LoadLibraryA`或者`LoadLibraryW`，我们用判断线程的入口点是不是`LoadLibraryA`或者`LoadLibraryW`，是就干掉。拿到线程入口点的方法是通过为公开的API --`GetThreadStartAddress`

关键代码如下:

```
typedef enum _THREADINFOCLASS {
     ThreadBasicInformation,
     ThreadTimes,
     ThreadPriority,
     ThreadBasePriority,
     ThreadAffinityMask,
     ThreadImpersonationToken,
     ThreadDescriptorTableEntry,
     ThreadEnableAlignmentFaultFixup,
     ThreadEventPair_Reusable,
     ThreadQuerySetWin32StartAddress,
     ThreadZeroTlsCell,
     ThreadPerformanceCount,
     ThreadAmILastThread,
     ThreadIdealProcessor,
     ThreadPriorityBoost,
     ThreadSetTlsArrayAddress,
     ThreadIsIoPending,
     ThreadHideFromDebugger,
     ThreadBreakOnTermination,
     MaxThreadInfoClass
} THREADINFOCLASS;

typedef LONG (WINAPI *NtQueryInformationThreadProc)(
     _In_ HANDLE ThreadHandle,
     _In_ THREADINFOCLASS ThreadInformationClass,
     _Inout_ PVOID ThreadInformation,
     _In_ ULONG ThreadInformationLength,
     _Out_opt_ PULONG ReturnLength
);

NtQueryInformationThreadProc NtQueryInformationThread = NULL;
hNtdll = GetModuleHandleW(L"ntdll.dll");
NtQueryInformationThread = (NtQueryInformationThreadProc)GetProcAddress(hNtdll, "NtQueryInformationThread");

HANDLE hThread = NULL;
PVOID pvStart = NULL;
hThread = OpenThread(THREAD_QUERY_INFORMATION | THREAD_TERMINATE, FALSE, te32.th32ThreadID);
NtQueryInformationThread(hThread, ThreadQuerySetWin32StartAddress, &pvStart, sizeof(pvStart), NULL);
```

### DLL劫持

DLL的防御姿势我能想到就是对自己进程每个要加载的DLL求一个校验值，程序运行过程中进行校验

其他的防御姿势坐等各位指教。。。。。。

献上学习时候写的一个注入小工具，VS2013编译的，32位

上传的附件：

- `SRC.zip`
- `Ring3注入总结及编程实现.docx`
- `Ring3注入总结及编程实现.pdf`
