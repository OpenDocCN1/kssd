
# TLS 隐藏的入口点

> 原文：[`www.pediy.com/kssd/pediy10/63237.html`](https://www.pediy.com/kssd/pediy10/63237.html)

## 1. 什么是 TLS？

`TLS` 是 `Thread Local Storage`（线程局部存储）的简称，是一项解决多线程内部变量使用问题的技术。用于将某些数据和一特定线程关联起来，即，这些数据为关联线程所独有（私有）。在多线程编程中, 同一个变量, 如果要让多个线程共享访问, 那么这个变量可以使用关键字 `volatile` 进行声明; 而如果一个变量不想被多个线程共享访问, 那么就应该使用 `TLS`。

## 2. 如何使用 TLS 编程？

`TLS` 使用非常简单, 只要对变量声明时使用 `__declspec(thread)` 修饰就可以了。例如：

```c
__declspec(thread) int g_nData = 0;
```

## 3. 一个使用 TLS 的例子

```c
//--------------------------------------------------------------------------------------------------------

#include <windows.h> 

#include <stdio.h> 

#define THREADCOUNT 4 

DWORD dwTlsIndex; 

int iNUM_OF_CALL_COMMON=0; 

int iNUM_OF_CALL_THREAD=0; 

VOID ErrorExit(LPSTR); 

**VOID CommonFunc(VOID) **

{ 

   LPVOID lpvData; 

 // Retrieve a data pointer for the current thread. 

 iNUM_OF_CALL_COMMON++; 

   lpvData = TlsGetValue(dwTlsIndex); 

   if ((lpvData == 0) && (GetLastError() != ERROR_SUCCESS)) 

      ErrorExit("TlsGetValue error"); 

// Use the data stored for the current thread. 

    printf("common: thread %d: lpvData=%lx\n", 

      GetCurrentThreadId(), lpvData); 

   Sleep(5000); 

} 

**DWORD WINAPI ThreadFunc(VOID) **

{ 

   LPVOID lpvData; 

// Initialize the TLS index for this thread. 

 iNUM_OF_CALL_THREAD++; 

   lpvData = (LPVOID) LocalAlloc(LPTR, 256); 

   if (! TlsSetValue(dwTlsIndex, lpvData)) 

      ErrorExit("TlsSetValue error"); 

   printf("thread %d: lpvData=%lx\n", GetCurrentThreadId(), lpvData); 

   CommonFunc(); 

// Release the dynamic memory before the thread returns. 

    lpvData = TlsGetValue(dwTlsIndex); 

   if (lpvData != 0) 

      LocalFree((HLOCAL) lpvData); 

   return 0; 

} 

int main(VOID) 

{ 

   DWORD IDThread; 

   HANDLE hThread[THREADCOUNT]; 

   int i; 

// Allocate a TLS index. 

    if ((dwTlsIndex = TlsAlloc()) == TLS_OUT_OF_INDEXES) 

      ErrorExit("TlsAlloc failed"); 

// Create multiple threads. 

    for (i = 0; i < THREADCOUNT; i++) 

   { 

      hThread[i] = CreateThread(NULL,  // default security attributes 

         0,                            // use default stack size 

         (LPTHREAD_START_ROUTINE) ThreadFunc, // thread function 

         NULL,                         // no thread function argument 

         0,                           // use default creation flags 

         &IDThread);                  // returns thread identifier 

   // Check the return value for success. 

      if (hThread[i] == NULL) 

         ErrorExit("CreateThread error\n"); 

   } 

  // printf("All threads were created.\n");

   for (i = 0; i < THREADCOUNT; i++) 

      WaitForSingleObject(hThread[i], INFINITE); 

   TlsFree(dwTlsIndex);

   printf("The nums of thread is: %d\n",iNUM_OF_CALL_THREAD);

   printf("The nums of call is: %d\n",iNUM_OF_CALL_COMMON);

   return 0; 

} 

VOID ErrorExit (LPSTR lpszMessage) 

{   

   fprintf(stderr, "%s\n", lpszMessage); 

   ExitProcess(0); 

}   

//--------------------------------------------------------------------------------------------------------
```

## 4. T L S 的内部数据结构

![](img/6a40b76ab764bfdcc83b558ae4603fa3.jpg)

图 1 用于管理 T L S 的内部数据结构

每个标志均可设置为 `FREE` 或者 `INUSE`，表示 `TLS` 槽( s l o t )是否正在使用。Microsoft 保证至少 `TLS_MINIMUM_AVAILABLE` 位标志是可供使用的。

## 5. 相关 API

`Windows` `TLS` 的 `API`: `TlsAlloc`, `TlsFree`, `TlsSetValue`, `TlsGetValue`。

- `DWORD TlsAlloc();` //(若要使用动态 T L S，首先必须调用 `TlsAlloc` 函数)
    这个函数命令系统对进程中的位标志进行扫描，并找出一个 `FREE` 标志。然后系统将该标志从 `FREE` 改为 `INUSE`，并且 `TlsAlloc` 返回位数组中的标志的索引。`DLL`（或 `APP`）通常将该索引保存在一个全局变量中，因为它的值是每个进程而不是每个线程使用的值。
- `BOOL TlsSetValue( //将一个值放入线程的数组中`
    `DWORD dwTlsIndex,`
    `PVOID pvTlsValue);`
- `PVOID TlsGetValue(DWORD dwTlsIndex); //要从线程的数组中检索一个值`
- `BOOL TlsFree(DWORD dwTlsIndex); //当在所有线程中不再需要保留 TLS 槽时`

参考资料：Jeffrey Richter《《Programming Applications for Microsoft Windows (4th Ed.)》》Chapter 21

## 6. TLS 目录

![](img/8dac4b9fddc88995544ea9773701b30e.jpg)

### TLS Callback Functions

这是线程建立和退出时的回调函数, 包括主线程和其他线程.`AddressOfCallBacks` 是指向函数指针数组的指针, 以 `0` 结束.

```c
typedef struct _TEB {
    NT_TIB Tib;
    PVOID EnvironmentPointer;
    CLIENT_ID Cid;
    PVOID ActiveRpcInfo;
    PVOID ThreadLocalStoragePointer; ; 2ch
    PPEB Peb; ; 30h
    ULONG LastErrorValue; ; 34h
    …}
```

`TLS` 目录 `#define IMAGE_DIRECTORY_ENTRY_TLS 9 (第十个目录)`

### Tls 隐藏的入口点利用

就是利用 `Address of Callbacks` 字段,写入要执行的代码的地址就可以了.

测试对象：`test.exe`

程序类型：`delphi` 编写

`PeEditor` 检测 `Tls` 信息（文件偏移 `59400H` 处）如下：

![](img/60afe7b9980ed97bc701dc7c2a66a4cf.jpg)

```text
TLS 信息

\------------------------------------------

目录表： 0045D000

TLS 数据： 0045D010

索引变量： 004570A0

调用返回地址： 0045E010 （调用返回地址）

\------------------------------------------
```

我们打算让系统在 `0045E010` 处执行我们的代码，就需要在这个地方对应的文件偏移处写入我们的代码。

观察节表信息

`VA=0045E010,按照文件不需要重定位去计算则 RVA=0005E010`

观察节表信息，这个值处于`.rdata` 节内。

`.rdata` 节起始 `RVA=0005E000`，起始 `Offset` 为 `00059400`

则 `callback` 地址对应的 `Offset=（0005E010 - 0005E000）+ 00059400 =00059410`

注意在这里（`00059410H`）应该写入一个函数地址。

打造一个简单的代码:

```asm
；----------------------------------------------------------------

.386

.model flat,stdcall

option casemap:none

include windows.inc 

include user32.inc 

includelib user32.lib 

.code 

start: 

jmp @F 

db 'Run thru Tls entry point.',0 

@@: 

mov eax,$ 

sub eax,26 

call @delta 

@delta: 

pop ebp 

sub ebp,offset @delta 

add eax,ebp 

invoke MessageBox,NULL,eax,NULL,0 

ret 

end start 

；----------------------------------------------------------------
```

编译生成可执行文件，这只是一个简单的弹出对话框的程序。

![](img/3d37cc186e98c912978704e49a27152f.jpg)

打开 16 进制工具，载入 `msgbox.exe`，复制代码：

![](img/f0ceeddb3c487f53a068442f1d8bd8e1.jpg)

注意 `00402000H` 处存放的是 `MessageBox` 的入口地址，我把它固化下来，动态跟踪发现是 `8A05D577H`。（当然在编程时可以动态搜寻我们需要的 `API`，为简单起见这里 `Hard-code` 一下。）

打开 16 进制工具,载入艺术字体 `2.exe`，定位到 `00059410`。

![](img/0af79ea1378d34f1f94edd2a4b1fdb75.jpg)

我们把代码复制在偏移 `00059410` 处，对应的 `VA=0045E030H`，则把 `0045E030` 存入偏移 `00059410`。

还要注意最后的 `MessageBox` 的入口地址我们使用硬编码的方式，我们把这个地址写入在 `00059480` 处了。保存修改，然后运行程序。程序首先弹出两个对话框（一个是 `TLS` 模板，一个是主线程创建），结束程序运行还会弹出一个对话框，不过这个对话框太丑了：

![](img/aa237dd99b6f97b2fd2fe1bca31c4945.jpg)

说明：我在看一本介绍病毒知识的书籍时看到作者提及到这么一句话，大概意思是说他们公司某人发现了这个秘密，一直没有公布，不过后来由于 `rgb` 利用此技术制造了病毒，也就无所谓秘密可言了。由于 `Tls` 入口要比 `OEP` 先执行，所以在加壳与脱壳中都有利用的价值。
