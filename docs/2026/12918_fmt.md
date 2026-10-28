
# 如何编写Loader[翻译]

感谢大家支持.有翻译一篇.有些地方翻的不好.以前看雪精华里面也有相关文章,不过感觉这个说的更浅显易懂一些.

如何编写Loader(How to code a loader)

作者：Detten    Detten@tiscali.be  
来源：<http://biw.rult.at/>  
翻译：nbw    [www.vxer.com](http://www.vxer.com)  
实例下载：[loader.zip](https://www.kanxue.com/pediy6570/loader.zip)（或鼠标右键另存为）

## 1、什么是Loader，为什么需要它？

所谓的Loader是一个用来加载其他程序的小程序。当然，只有被加载的内存的程序需要改动的时候我们才采用Loader。（内存补丁）

Loader常用于让游戏玩家修改游戏。

有很多原因导致我们选择Loader而不是一般的补丁程序。我们或许需要在程序CRC校验以后再进行修改，或者开始的时候修改内存数据，然后在程序中再恢复原来的数据.....

我肯定你还可以找到其他一些用途。

## 2、Loader是怎么工作的？

OK，找到你的`Win32.hlp`然后坐下来:)

首先，Loader必须创建一个进程启动目标程序。我们将用`CreateProcess`函数做这个（很明显嘛）。当目标程序被加载到内存，我们需要中断该进程，以便进行我们的修改。

让我们查看一下`win32.hlp`对这个API函数的讲解：

```c
BOOL CreateProcess(
    LPCTSTR lpApplicationName,    // 可执行模块名称指针
    LPTSTR lpCommandLine,    // 命令行字符串指针
    LPSECURITY_ATTRIBUTES lpProcessAttributes,  // 进程安全属性指针
    LPSECURITY_ATTRIBUTES lpThreadAttributes,  // 线程安全属性指针
    BOOL bInheritHandles,    // 句柄继承标记
    DWORD dwCreationFlags,    // 创建标记
    LPVOID lpEnvironment,    // 新环境块指针
    LPCTSTR lpCurrentDirectory,    // 当前路径指针
    LPSTARTUPINFO lpStartupInfo,  // STARTUPINFO指针
    LPPROCESS_INFORMATION lpProcessInformation   // PROCESS_INFORMATION指针
);
```

这里涉及到的API函数请查看`win32.hlp`以了解详细内容，因为这里我只讲解重要的一些关键的API。

`lpApplicationName` 使目标程序的路径+名称.（例如 `c:\somedir\crackme.exe`）

`lpCommandLine` 可以用来指定命令行参数，如果需要的话。

`dwCreationFlags` 也很重要，因为我们需要随时中断被加载的进程，所以这里设置为`CREATE_SUSPENDED`。

`lpStartupInfo` 指向一个代表启动信息的结构（查看`win32.hlp`看详细信息）。

`lpProcessInformation` 指向一个空结构，该结构在进程被加载的时候载内存中被填充。结构中包含有进程句柄，线程句柄和进程/线程ID。

注意：这里建议采用进程句柄而不是线程句柄，因为如果采用进程句柄，你对整个进程体拥有`PROCESS_ALL_ACCESS`的操作权限。就是说你对整个进程拥有读写权限，但是如果采用线程ID，就需要再设置写权限。

好了，现在目标程序被加载了。我们可以利用下面的API函数来运行或者停止进程：

```c
DWORD ResumeThread(
    HANDLE hThread   // identifies thread to restart
);
```
恢复进程

```c
DWORD SuspendThread(
    HANDLE hThread   // handle to the thread
);
```
挂起进程

`hThread` 可以从`LPPROCESS_INFORMATION`结构获得。

最后，可以利用下面的函数读写进程：

```c
BOOL WriteProcessMemory(
    HANDLE hProcess,  // 需要修改的进程的句柄
    LPVOID lpBaseAddress,  // 开始写入的地址
    LPVOID lpBuffer,  // 指向被写入的数据
    DWORD nSize,  // 写入字节数目
    LPDWORD lpNumberOfBytesWritten   // 返回写入的数据长度
);
```

这是一个典型的信息自我返回（self-explanatory）。`hProcess`可以从`LPPROCESS_INFORMATION`结构获取。

从进程读取数据：

```c
BOOL ReadProcessMemory(
    HANDLE hProcess,  // handle of the process whose memory is read
    LPCVOID lpBaseAddress,  // address to start reading
    LPVOID lpBuffer,  // address of buffer to place read data
    DWORD nSize,  // number of bytes to read
    LPDWORD lpNumberOfBytesRead   // address of number of bytes read
);
```

看明白上面的信息，就可以看下面的内容了。

## 3、Loader举例

下面的事例我将启动一个Crackme（译者：我也没有）并且把窗口标题改成“Detten's Caption”。我将把进程启动5秒钟，然后挂起之。

这里我们修补的字符串，当然用这种方法也可以修补字节或者字:)。作为事情准备，我们需要指导字符串在进程中的地址。所以，开启你喜欢的反汇编软件，找到地址为：`004050FCh`。

```asm
<-------------Code Snippet----------------->
.386
.model flat,stdcall
option casemap:none
include \masm32\include\windows.inc
include \masm32\include\user32.inc
include \masm32\include\kernel32.inc
includelib \masm32\lib\user32.lib
includelib \masm32\lib\kernel32.lib

.data
FileName db "C:\somedir\crackme.exe",0
notloaded db "It did not work :-(",0
Letsgo db "The process is started",13,10,
    "Let's change smthg and run it now :-)",0
NewText db "Dettens Caption",0

Startup STARTUPINFO <>
processinfo PROCESS_INFORMATION <>

.data?
hInstance HINSTANCE ?
byteswritten dd ?
uExitCode dd ?

.code
start:
  invoke GetModuleHandleA, NULL
  mov    hInstance,eax

  ; 创建新进程，载入crackme，并且迅速挂起该线程
  invoke CreateProcess, ADDR FileName, NULL, NULL, NULL, NULL, CREATE_SUSPENDED,
            NULL, NULL, ADDR Startup, ADDR processinfo

  .IF eax == NULL ; 进程创建失败?
    invoke MessageBox, NULL, ADDR notloaded, NULL, MB_ICONEXCLAMATION
  .ELSE
          invoke MessageBox, NULL, ADDR Letsgo, NULL, MB_OK ; Display Message

    ; 修改字符串（004050FCh）
                    invoke WriteProcessMemory, processinfo.hProcess, 004050FCh, ADDR NewText,
                             13, byteswritten

          ; 恢复进程 ;)
          invoke ResumeThread, processinfo.hThread

          ; 让进程运行5秒，然后杀掉它
          invoke Sleep, 5000
          invoke TerminateProcess, processinfo.hProcess, uExitCode
  .ENDIF

  invoke ExitProcess,eax
end start
<-------------End Code Snippet------------->
```

这就是写Loader的整个步骤。

再下一篇中我将讨论更多的Loader技术。比如读取和修改进程环境（所有的寄存器和标记）
