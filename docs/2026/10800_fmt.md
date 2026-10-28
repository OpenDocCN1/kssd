
# [原创]扫盲之Api Hook 细析（一）

//Author:Alex(Yock.W)

//转载请署名出处

//针对一些初学者扫盲，牛人就不用看了。希望别笑话……

## 前言 基础知识

本系列文章会对常用的几种API HOOK方法进行全面的分析。

### Hook是什么？

钩子(Hook)，是Windows消息处理机制的一个平台，应用程序可以在上面设置子程以监视指定窗口的某种消息，而且所监视的窗口可以是其他进程所创建的。当消息到达后，在目标窗口处理函数之前处理它。钩子机制允许应用程序截获处理window消息或特定事件。

钩子实际上是一个处理消息的程序段，通过系统调用，把它挂入系统。每当特定的消息发出，在没有到达目的窗口前，钩子程序就先捕获该消息，亦即钩子函数先得到控制权。这时钩子函数即可以加工处理（改变）该消息，也可以不作处理而继续传递该消息，还可以强制结束消息的传递。

### Hook原理

每个Hook都有一个关联的链表，由系统维护，链表指针指向被Hook子程调用的回调函数：

``` 
LRESULT WINAPI HookCallBack(

int nCode,

WPARAM wParam,

LPARAM lParam);
```
`nCode` 事件代码
`wParam` 消息的类型
`lParam` 包含的消息

当被Hook的类型关联的消息发生时，系统会把这个消息传递到链表指针指向的Hook子程，由子程内部对消息进行处理，。

### 钩子安装与卸载

使用Windows提供的API `SetWindowsHookEx`来将Hook子程指针安装到Hook链表，安装的Hook子程始终在Hook链表头部。

``` 
HHOOK SetWindowsHookEx(

int idHook,

HOOKPROC lpfn,

HINSTANCE hMod,

DWORD dwThreadId);
```
`idHook` 安装的钩子类型
`lpfn` 子程函数指针
`hMod` 应用程序实例的句柄
`dwThreadId` Hook关联的线程标识符

当被Hook的消息类型发生时，系统会调用与这个Hook关联的链表头的Hook子程。由子程序决定是否要把这个事件传递给下个子程。Hook子程传递事件给下一个Hook子程需要调用使用Windows提供的API `CallNextHookEx`。

``` 
LRESULT CallNextHookEx(

HHOOK hhk,

int nCode,

WPARAM wParam,

LPARAM lParam);
```
`hhk` 当前钩子的句柄，由`SetWindowsHookEx`返回
`nCode` 事件代码
`wParam` 消息的类型
`lParam` 包含的消息

钩子在使用完，需调用`UnHookWindowsHookEx`来卸载。

``` 
UnHookWindowsHookEx(

HHOOK hhk);
```

## Api Hook 的几种方法

### 一.修改函数过程映射的地址空间

Api Hook:

我们都知道，在调用函数的时候，会先`Call 0xFFFFFF`，来到函数映射在内存的地址空间里，这时候，传入的参数已入栈，那么，我们只需要修改函数头部，跳转到我们自定义的过程，这样，我们就达到了对API函数监视的目的。那么，我们需要多大的空间来写入代码让其跳转到我们自定义的过程里呢？

聪明的你肯定已经想到了。没错，只需5字节即可。Why? 看一下代码：

`jmp 0xFFFFFFFF`
`jmp` 指令占1字节，地址是4字节，所以，只需5字节即可。

有的看官可能会问了，为什么不用`Call`呢？

因为`Call`指令在实现跳转以后会通过`ret`返回到`Call`下面的指令继续执行，而`Jmp`指令实现跳转以后则全有子程来控制了。

我们知道了通过以上方法来修改函数头部数据来达到我们的目的，那么，我们先要得到指向函数头部的地址，通过`LoadLibrary`和`GetProcAddress`即可获取到我们要监视的函数在内存中映射的地址指针。我们以`MessageBoxA`为例：

``` 
typedef int (___stdcall *TMessageBoxA)(HWND hWnd,LPSTR lpText,LPSTR lpCaption,int uType);

extern "C" TMessageBoxA ___fastcall GetPFuncAddr(LPSTR DllName,const char* FullName);

TMessageBoxA ___fastcall GetPFuncAddr(LPSTR DllName,const char* FullName)

{

void *DllModule;

DllModule = LoadLibrary(DllName);

TMessageBoxA PAddr = (TMessageBoxA)GetProcAddress(DllModule,FullName);

return PAddr;

}
```

在得到了指向内存映射里函数头部的指针以后，我们下一步，来读出函数头部的5字节。为什么要读出函数头部的5字节呢？呵呵，当然是留作恢复时写会数据了。`ReadProcessMemory`：

``` 
BOOL ReadProcessMemory(

HANDLE hProcess,

LPCVOID lpBaseAddress,

LPVOID lpBuffer,

DWORD nSize,

LPDWORD lpNumberOfBytesRead);
```
`hProcess` 进程句柄
`lpBaseAddress` 读内存开始地址
`lpBuffer` 保存数据的指针
`nSize` 数据大小
`lpNumberOfBytesRead` //address of number of bytes read

``` 
BYTE *pFuncData = new BYTE[5];

LPDWORD iRead;

ReadProcessMemory((void *)GetCurrentProcess(),PMessageBoxA,pFuncData,5,iRead);
```

保存原函数入口点的数据以后，嘿嘿，当然是写入指向我们自己的函数地址了。

``` 
BYTE AsmCode[5];

AsmCode[0] = 0xE9;

__asm

{

lea eax, MyMessageBoxA

mov ebx,PMessageBoxA

sub eax,ebx

sub eax,5

mov dword ptr[AsmCode+1],eax

}
```
`WriteProcessMemory((void *)GetCurrentProcess(),PMessageBoxA,AsmCode,5,iWrite);`

`MyMessageBoxA`是我们自己构造的函数：

``` 
int WINAPI MyMessageBoxA(HWND hWnd,LPSTR lpText,LPSTR lpCaption,int uType)

{

//我们来弹出一个消息框，改变一下传入的标题参数值

//那么首要做的，就是先恢复MessageBoxA

WriteProcessMemory((void *)GetCurrentProcess(),PMessageBoxA,pFuncData,5,iWrite);

int Result = MessageBoxA(hWnd,lpText,"ApiHook Test",uType);

WriteProcessMemory((void *)GetCurrentProcess(),PMessageBoxA,AsmCode,5,iWrite);

return Result;

}
```

恢复API的原入口处数据：

`WriteProcessMemory((void *)GetCurrentProcess(),PMessageBoxA,pFuncData,5,iWrite);`

防止修改函数入口数据来进行API HOOK的方法：

在程序启动的时候保存关键API入口的5字节，定期检查一次，发现被修改了，改回即可。

----------------------------------------------------------------

下一篇中，会针对SEH技术实现API HOOK和如何防止SEH APIHOOK进行详解。

扫盲到此结束。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
