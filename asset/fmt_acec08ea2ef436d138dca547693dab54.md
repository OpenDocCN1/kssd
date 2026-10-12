
你可能想知道我为什么会选择监视 `LoadLibraryExW` 函数。有一个很好的理由可以解释其中的缘由。因为 `LoadLibrary` 函数带一个字符串参数，它同时有 ANSI 和 UNICODE 这两个版本。`LoadLibrary` 最常用的形式是 `LoadLibraryA`。但实际上，`LoadLibraryA` 只是对 `LoadLibraryExA` 进行了简单的封装，而 `LoadLibraryExA` 只是对 `LoadLibraryExW` 进行了简单的封装。同样，`LoadLibraryW` 也只是对 `LoadLibraryExW` 进行了简单的封装。这样，所有调用 DLL 的方法最终都归结为调用 `LoadLibraryExW` 函数。只需要在这个 API 上设置一个断点，我就可以捕获所有对 `LoadLibrary` 函数及其变种的调用。

如果想实验一下 `VectoredExcBP`，确保你使用的是 Windows XP Beta 2 或者更新版本的操作系统，然后运行 `TestVE` 程序。`TestVE` 自身只是通过调用 `LoadLibrary` 来加载两个 DLL（`MFC42.DLL` 和 `WININET.DLL`）。但是这两个 DLL 内部也要加载其它 DLL，因此你会看到其它一些附加的对 `LoadLibrary` 的调用。如果一切正常，你会看到类似下面的输出结果：

```
LoadLibrary called on: MFC42
LoadLibrary called on: MSVCRT.DLL
LoadLibrary called on: G:\WINDOWS\System32\MFC42LOC.DLL
LoadLibrary called on: WININET
LoadLibrary called on: kernel32.dll
LoadLibrary called on: advapi32.dll
LoadLibrary called on: kernel32.dll
```

### 向量化异常处理的实现

Windows XP Beta 2 中的向量化异常处理在实现上相当简单。虽然表面上 `AddVectoredExceptionHandler` 这个 API 是在 `KERNEL32.DLL` 中，但是它实际上转发到了 `NTDLL` 中的 `RtlAddVectoredExceptionHandler` 函数上。图 4 是我为这个函数写的伪代码。

向量化异常处理程序链表是一个循环链表（译者注）。每个已注册的异常处理程序用一个 12 字节的结点表示，这个结点所占用的内存在进程堆上分配。同时用一个临界区对象来保护将异常处理程序结点插入到链表的头部或尾部的这部分代码。如果 `FirstHandler` 参数不为 0，则新的异常处理程序结点被插入到链表的头部；否则它被插入到链表的尾部。就这么简单！它并没有检查这是不是以前曾经注册过的某个结点进行的重复注册，这样某个异常处理程序可以注册（因此也会被调用）多次。

在向量化异常处理的实现中另一个值得注意的地方是这些向量化异常处理程序被调用的方式。正如在我那篇关于 SEH 的文章中讲到的那样，`KiUserExceptionDispatcher`（在 `NTDLL` 中）调用了 `RtlDispatchException`。图 5 说明了有关向量化异常处理的代码是如何被添加到 `NTDLL` 中的 `RtlDispatchException` 函数中的。如果你把它和我以前的那篇文章中的伪代码比较一下，会发现它仅在 `RtlDispatchException` 函数开头添加了一个函数调用（调用 `RtlCallVectorExceptionHandlers` 函数）。这充分证明了向量化异常处理程序是在结构化异常处理程序之前被调用的。

图 6 是 `RtlCallVectorExceptionHandlers` 函数的伪代码。同样，这段代码也十分简单。它使用一个临界区对象来保护一个 while 循环。在这个循环中，它遍历异常处理程序链表并调用相应的异常处理程序。如果异常处理程序返回 `EXCEPTION_CONTINUE_EXECUTION`，这个循环就直接退出而不再调用后续的处理程序。它负责返回一个值来指示 `RtlDispatchException` 是否应该去调用结构化异常处理程序。

正如你所料，我认为向量化异常处理是 Windows XP 中添加的非常重要的功能。我已经期盼这个功能被添加到 Win32 中很长时间了。在本文中，我已经向你演示了使用向量化异常处理的极大优点，希望这个功能在以后会有新的应用。

## 附录

图 2 `VectoredExcBP`

```cpp
//===========================================================================
// VectoredExcBP   Matt Pietrek 2001
// MSDN Magazine, September 2001
//
// !^!^!^! 警告 警告 警告 !^!^!^!
//   这段代码只能运行在 Windows XP 及其后继操作系统上。
//   要想正确地编译它，你必须做到：
//     A) 拥有一个定义了 AddVectoredExceptionHandler 函数的 WINBASE.H 文件
//     B) 确保编译器最先找到的 WINBASE.H 文件是前面提到的那个而不是其它的。
//        如果你不能确定如何做，请参考编译器文档。
//     C) 定义 _WIN32_WINNT=0x0500（或更高）
//
// 这段代码在 Windows XP Beta 2 下用 Visual C++ 6.0 可以通过编译并正常运行。
// 在写这段代码时，Windows XP 还处于 beta 测试阶段，因此某些内容可能改变，
// 其中包括 API 的行为等等。不能保证它可以在将来的 Windows 系统上正常运行。
//===========================================================================

#include "stdafx.h"

#ifndef _M_IX86
#error "This code only runs on an x86 architecture CPU"
#endif

LONG NTAPI LoadLibraryBreakpointHandler(PEXCEPTION_POINTERS pExceptionInfo);
void BreakpointCallback( PVOID pCodeAddr, PVOID pStackAddr );
void SetupLoadLibraryExWCallback(void);
BYTE SetBreakpoint( PVOID pAddr );
void RemoveBreakpoint( PVOID pAddr, BYTE bOriginalOpcode );

// 全局变量
PVOID g_pfnLoadLibraryAddress = 0;
BYTE g_originalCodeByte;

/*+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*/

BOOL APIENTRY DllMain( HANDLE hModule,
                       DWORD  ul_reason_for_call,
                       LPVOID lpReserved
                     )
{
  // 我们不需要接收有关线程启动/停止时的通知，因此禁用它们
  DisableThreadLibraryCalls( (HINSTANCE)hModule );

  // 在进程启动时设置断点，进程退出时移除断点
  if ( DLL_PROCESS_ATTACH == ul_reason_for_call )
       SetupLoadLibraryExWCallback();
  else if ( DLL_PROCESS_DETACH == ul_reason_for_call )
       RemoveBreakpoint( g_pfnLoadLibraryAddress, g_originalCodeByte );

  return TRUE;
}

/*+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*/

void SetupLoadLibraryExWCallback(void)
{
  // 获取 LoadLibraryExW 函数的地址。
  // 所有对 LoadLibraryA/W/ExA 的调用最终都转到了 LoadLibraryExW
  g_pfnLoadLibraryAddress=(PVOID)GetProcAddress(GetModuleHandle("KERNEL32"),
                                                "LoadLibraryExW" );

  // 为我们的断点添加一个向量化异常处理程序
  AddVectoredExceptionHandler( 1, LoadLibraryBreakpointHandler );

  // 在 LoadLibraryExW 上设置断点
  g_originalCodeByte = SetBreakpoint( g_pfnLoadLibraryAddress );
}

/*+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*/
// 这是 LoadLibraryExW 断点的处理程序。当断点被触发时，调用相应的回调
// 函数（BreakpointCallback）。然后单步执行原来的指令并继续执行下去。
// 实际上这需要处理两个异常，正如代码中所处理的那样。
/*+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*/

LONG NTAPI LoadLibraryBreakpointHandler(PEXCEPTION_POINTERS
                                       pExceptionInfo )
{
  // printf( "In LoadLibraryBreakpointHandler: EIP=%p\n",
  //         pExceptionInfo->ExceptionRecord->ExceptionAddress );

  LONG exceptionCode = pExceptionInfo->ExceptionRecord->ExceptionCode;

  if ( exceptionCode == STATUS_BREAKPOINT)
  {
       // 检查它是否是我们设置的断点。如果不是，将它传递给其它处理程序
       if ( pExceptionInfo->ExceptionRecord->ExceptionAddress
           != g_pfnLoadLibraryAddress )
       {
           return EXCEPTION_CONTINUE_SEARCH;
       }

       // 我们需要单步执行原来的指令，因此必须临时移去断点
       RemoveBreakpoint( g_pfnLoadLibraryAddress, g_originalCodeByte );

       // 调用我们的代码以执行所需的任务
       BreakpointCallback( pExceptionInfo->ExceptionRecord->ExceptionAddress,
                          (PVOID)pExceptionInfo->ContextRecord->Esp );

       // 设置 EFlags 寄存器中的跟踪标志，以便我们在捕获
       // STATUS_SINGLE_STEP 异常之前只执行一条指令（看下面）
       pExceptionInfo->ContextRecord->EFlags |= 0x00000100;

       return EXCEPTION_CONTINUE_EXECUTION;    // 重新执行出错指令
  }
  else if ( exceptionCode == STATUS_SINGLE_STEP )
  {
       // 检查异常地址是否是上面产生的那个异常的地址的下一个地址
       // 如果不是，将它传递给其它异常处理程序
  // 译者注：由于 Windows XP SP2 增强了系统的安全性，实际上许多
  // 系统 API 的 prolog 代码已经发生了变化。通常情况下系统 API 的第
  // 一条指令为 PUSH EBP，它的机器码为 0x55，占一个字节，因此作者
  // 才说要检查异常地址是否是上面产生的那个异常的地址的下一个地址
  // （+1）。而在 Windows XP SP2 中，LoadLibraryExW 的第一条指令变成
  // 了 PUSH 34h，它的机器码为 0x6A34，占两个字节，因此我们需要将
  // 其改为+2 才能正确执行
       if ( pExceptionInfo->ExceptionRecord->ExceptionAddress
           != (PVOID)((DWORD_PTR)g_pfnLoadLibraryAddress+2) )
       {
           return EXCEPTION_CONTINUE_SEARCH;
       }

       // printf( "In STATUS_SINGLE_STEP handler\n" );

       // 我们已经单步执行完了原来的指令，因此重新设置断点
       SetBreakpoint( g_pfnLoadLibraryAddress );

       // 清除前面设置的跟踪标志
       pExceptionInfo->ContextRecord->EFlags &= ~0x00000100;

       return EXCEPTION_CONTINUE_EXECUTION;    // 继续运行！
  }
  else    // 不是断点异常或单步异常。它不是我们处理的目标！
  {
       return EXCEPTION_CONTINUE_SEARCH;
  }
}

/*+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*/
// 当 LoadLibraryExW 被调用时就调用此函数。它的参数为断点的地址和堆栈指针。
// 堆栈指针可以用来获取堆栈上的参数值。此时我们想获取的是表示将要加载的
// DLL 的名称的字符串（UNICODE 格式）。
/*+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*/

void BreakpointCallback( PVOID pCodeAddr, PVOID pStackAddr )
{
  DWORD nBytes;

  LPWSTR pwszDllName;

  // pStackAddr+0 == 返回地址
  // pStackAddr+4 == 第一个参数
  ReadProcessMemory(  GetCurrentProcess(),
                      (PVOID)((DWORD_PTR)pStackAddr+4),
                      &pwszDllName, sizeof(pwszDllName),
                      &nBytes );
```
