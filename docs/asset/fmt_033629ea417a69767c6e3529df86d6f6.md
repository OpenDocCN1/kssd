
# 一个用于 C/C++输出编译提示信息的宏

> 原文：[`www.pediy.com/kssd/pediy12/131188.html`](https://www.pediy.com/kssd/pediy12/131188.html)

有感于`LiuTaoTao`的帖子，来一个更实用的宏，而且这个宏更好地诠释了`#`这个宏操作符。

宏定义及使用见下面的代码：

```
// cpptest.cpp : Defines the entry point for the console application.

//

#include "stdafx.h"

//宏定义

#define _REMIND_ME0(a) #a
#define _REMIND_ME1(a) __FILE__ "(" _REMIND_ME0(a) ") : "
#define REMIND_ME(a) message(_REMIND_ME1(__LINE__)a)

//宏用法

#pragma REMIND_ME("这只是一个空函数,有时间实现它!")

int _tmain(int argc, _TCHAR* argv[])
{
  return 0;
}
```

你编译该源文件时，在`VS` `IDE`的`Output`窗口中有如下的输出：

```
1>------ Build started: Project: cpptest, Configuration: Debug Win32 ------

1>   cpptest.cpp
1>   e:\study\tttest\cpptest\cpptest.cpp(11) : 这只是一个空函数,有时间实现它!

1>   cpptest.vcxproj -> e:\Study\tttest\Debug\cpptest.exe

========== Build: 1 succeeded, 0 failed, 0 up-to-date, 0 skipped ==========
```

如果你双击第 3 行，`IDE`可以直接转到源程序中对应的行；如果你的工程是自动联编，则提示信息会出现在你编译报告中，对自己和协作者都是一个提示。

> `最初由` `半道出家`发布 `![查看帖子](img/showthread.php?p=938918#post938918)` 没什么大用，只是如果代码编译时有错或警告，双击编译输出对应的错误或警告条目可以跳转到对应的源代码行，如果没有错误，又想在源代码中做个提示标记，这个宏就有用。我有时有些想法暂时不实现或多部分代码要配合时，用它做一标记，以免时间长了自己忘记了，而且关键是可以快速定位相应的代码位置。

有没有用. 要看写得怎么样.

我就写了个简单的

```
#if !defined(_LWLIB_H)

#define _LWLIB_H

#include <windows.h>

#if defined(_DEBUG)|defined(DEBUG)

#define DBGOUTPUT DebugPrint

#define DBGPRINT(pszName) {int  retValue; \\
  __asm{ mov retValue, eax} \\
  int error_code \= GetLastError(); \\
  DebugPrint("%s", "... start ..."); \\
  DebugPrint("call %s", pszName); \\
  DebugPrint("return value(hex): %0X", retValue); \\
  DebugPrint("error code(hex): %0X", error_code); \\
  DebugPrint("from: %s", __FUNCTION__); \\
  DebugPrint("file: %s", __FILE__); \\
  DebugPrint("line(dec): %d", __LINE__); \\
  DebugPrint("%s", "... end ...");}

#else

#define DBGPRINT

#define DBGOUTPUT

#endif

__inline void WINAPI DebugPrint(LPSTR pszFormat, ...)

{
  CHAR szBuffer[10240];
  ZeroMemory(szBuffer, sizeof(szBuffer));
  va_list arg_list;
  va_start(arg_list, pszFormat);
  vsprintf_s(szBuffer, _countof(szBuffer), pszFormat, arg_list);
  va_end(arg_list);
  OutputDebugStringA(szBuffer);
}

#endif // end of _LWLIB_H
```

用的时候，先把头文件包含进来：

```
1.
CreateFile(...);
DBGPRINT("CreateFile"); // 监视上面函数的返回值
2.
myFunc()
{
   DBGOUTPUT("Enter myFunc"); // 写点注释什么的...
}
```

在脱离`IDE`调试时，可以直接使用`dbgview`查看调试输出.
