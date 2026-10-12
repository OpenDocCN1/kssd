
# 虚拟桌面技术的初步探讨

> 原文：[`www.pediy.com/kssd/pediy10/82537.html`](https://www.pediy.com/kssd/pediy10/82537.html)

【文章标题】: 虚拟桌面技术的初步探讨

【文章作者】: newjueqi

【作者邮箱】: zengjiansheng1@126.com

【作者 QQ 号】: 190678908

【编写语言】: VC++6.0

【操作平台】: XP-SP2

【作者声明】: 这几天研究了虚拟桌面技术，感觉到这是一种非常有意思的技术，这篇就当成是学习笔记吧！本人只是感兴趣，没有其它目的，失误之处敬请给位大侠原谅！

## 什么是桌面？

每一个运行着 `Window NT` 的系统中都有一个 `Window` 工作站对象，这个对象是安全对象的第一层，是所有用户安全对象的继承之源，每一个 `Window` 工作站对象可以拥有一些桌面对象，每一个桌面都拥有一个窗口链。窗口链里存放着显示在所属桌面的各种窗口。`Window NT` 用了两个桌面窗口对象，一个是用来处理登陆界面、屏蔽、锁住工作站等，一个是我们登陆之后进来操作的窗口了。

`Window NT` 通过 `"explorer.exe"` 进程来管理这个桌面对象。这就是为什么我们在任务管理器里杀掉 `"explorer.exe"`，我们的桌面就会消失的原因。

## 什么是虚拟桌面？

虚拟桌面是一种可以在电脑原来桌面基础上再创造一个新的桌面出来，在新的桌面上可以进行日常的操作。

## 虚拟桌面的用途？

1. 本人觉得这门技术最重要的用途就是可以把任何有 UI 界面的软件变成一个后台软件（即看不到任何界面，包括启动界面）
2. 可以工作时是一个桌面，娱乐时是一个桌面（大家可以去下载网站上搜索一下这类软件的用途，上面的功能描述非常有意思）

## 虚拟桌面的实现方法

在 `windows` 中，要创建新的桌面可用到 API：`CreateDesktop()`，函数声明如下：

```c
HDESK CreateDesktop(
  LPCTSTR lpszDesktop,         // 新桌面的名称
  LPCTSTR lpszDevice,         // 为 NULL
  LPDEVMODE pDevmode,         // 为 NULL
  DWORD dwFlags,              // 指定应用程序在桌面的兼容方式
  ACCESS_MASK dwDesiredAccess, // 指定新桌面的权限
  LPSECURITY_ATTRIBUTES lpsa   // 指定句柄是否能被继承
);
```

返回值是新创建的桌面的句柄。

那么新建了一个桌面后，怎么在这个新的桌面上运行程序呢？先不要着急，我们先来回顾一下创建进程的函数 `CreateProcess()`，在这个函数的参数中 `StartupInfo` 中有 `lpDesktop` 这么一个属性，如果这个属性为 `NULL` 则在当前的桌面创建线程，如果指定了桌面的名称，则进程将会在指定的桌面上启动，所以想在创建的新桌面里初始化一些程序，只要把 `lpDesktop` 参数指定为新桌面的名称即可。

另外也有一个简单的方法可把新的线程挂在新创建的桌面下，就是使用 API 函数 `SetThreadDesktop()`，声明如下：

```c
BOOL SetThreadDesktop(
  HDESK hDesktop  // 指向指定的桌面句柄
);
```

但使用这个函数要注意一点，根据 MSDN 的说法：The `SetThreadDesktop` function will fail if the calling thread has any windows or hooks on its current desktop (unless the `hDesktop` parameter is a handle to the current desktop) 意思就是除非要指定的桌面句柄是当前的桌面，不然的话这个函数的调用会失败如果当前线程拥有任何的窗口（即 UI 界面）。

### 怎么实现不同桌面之间的切换呢？

要在不同的桌面之间切换，可用 API 函数 `SwitchDesktop`，声明如下：

```c
BOOL SwitchDesktop(
  HDESK hDesktop  // 桌面的句柄
);
```

另外也可通过点击“切换”按钮实现桌面的切换。

但又引申出一个新的问题，必须要知道各个桌面的句柄，获取桌面的句柄可通过 API 函数 `GetThreadDesktop`，函数的声明如下：

```c
HDESK GetThreadDesktop(
  DWORD dwThreadId  // 线程的 ID
);
```

返回值就是指定线程所在的桌面了。

而且我们必须要认清的是创建新桌面的线程启动是在旧的桌面上，所以可以用下面的语句轻松获得当前桌面的句柄：

```c
GetThreadDesktop(GetCurrentThreadId());
```

返回值就是旧的桌面句柄。

### 怎么关闭新创建的桌面？

这个问题其实也不用我们担心，微软已经替我们想好了^-^，用 `CloseDesktop` 函数可轻松实现这个功能，函数声明如下：

```c
BOOL CloseDesktop(
  HDESK hDesktop  // 指定要关闭的桌面的句柄
);
```

下面贴一段代码的例子，是在新创建的桌面上运行计算器（`calc.exe`）实现计算器的后台运行：

```c
#include <windows.h>

HINSTANCE hInst;  //当前的实例句柄

HWND hWnd;        //窗口句柄

HDESK hvirtualDesk;    //新创建的虚拟桌面句柄

PROCESS_INFORMATION pi; //计算器进程信息

//消息循环
LRESULT CALLBACK WinProc(
      HWND hwnd,      // handle to window
      UINT uMsg,      // message identifier
      WPARAM wParam,  // first message parameter
      LPARAM lParam   // second message parameter
)
{
  switch(uMsg)
  {
  case WM_CLOSE:
      TerminateProcess( pi.hProcess, 1 );
      CloseDesktop( hvirtualDesk );
      DestroyWindow(hwnd);
      PostQuitMessage(0);
      break;
  case WM_DESTROY:
      CloseDesktop( hvirtualDesk );
      PostQuitMessage(0);
      break;
  case WM_HOTKEY:
    if ( 0x0001 == wParam ) //为退出桌面热键 Alt+Q
    {
        SendMessage(hwnd,WM_CLOSE,0,0);
    }
    break;
  default:
    return DefWindowProc(hwnd,uMsg,wParam,lParam);
  }
  return 0;
}

//创建虚拟桌面
void CrateVirtualDesk()
{
  //把新创建的虚拟桌面句柄存放在 hvirtualDesk
  hvirtualDesk=CreateDesktop( "newdesk",
          NULL,
          NULL,
          DF_ALLOWOTHERACCOUNTHOOK,
          GENERIC_ALL,
          NULL);
}

//在虚拟桌面上运行一个计算器的实例
void RunCalc()
{
  STARTUPINFO si;
  ZeroMemory( &si, sizeof(si) );
  si.cb = sizeof(si);
    si.lpDesktop = "newdesk";
  ZeroMemory( &pi, sizeof(pi) );
  if( !CreateProcess( NULL,
    "calc",
    NULL,
    NULL,
    FALSE,
    0,
    NULL,
    NULL,
    &si,
    &pi ) )
  {
    MessageBox(NULL,"运行计算器失败","Error",0);
    ExitProcess(1);
  }
}

int WINAPI WinMain(
    HINSTANCE hInstance,      // handle to current instance
    HINSTANCE hPrevInstance,  // handle to previous instance
    LPSTR lpCmdLine,          // command line
     int nCmdShow             // show state
)
{
  WNDCLASS wndcls;
  MSG msg;
  hInst=hInstance;
  ZeroMemory( &wndcls,sizeof(wndcls) );
  wndcls.cbClsExtra=0;
  wndcls.cbWndExtra=0;
  wndcls.hbrBackground=(HBRUSH)GetStockObject(BLACK_BRUSH);
  wndcls.hInstance=hInstance;
  wndcls.lpfnWndProc=WinProc;
  wndcls.lpszClassName="hello";
  wndcls.lpszMenuName=NULL;
  wndcls.style=CS_HREDRAW | CS_VREDRAW;
  RegisterClass(&wndcls);
  hWnd=CreateWindow( "hello","hello",WS_OVERLAPPEDWINDOW,
    300,300,100,100,NULL,NULL,hInstance,NULL );
  //注册所需的热键，Alt+Q 为退出创建的虚拟桌面
  if( !RegisterHotKey( hWnd,0x0001,MOD_ALT ,'Q' ) )
    {
        //处理退出进程
        return TRUE;
    }
  //创建虚拟桌面
  CrateVirtualDesk();
  //在虚拟桌面上运行一个计算器的实例
  RunCalc();
  ShowWindow( hWnd,SW_SHOWNORMAL );
  UpdateWindow( hWnd );
  while( GetMessage( &msg,NULL,0,0 ) )
  {
    TranslateMessage(&msg);
    DispatchMessage(&msg);
  }
  return msg.wParam;
}
```

程序运行后如下图：

![](img/243fae648dee74f4ac83f117164ab0e1.jpg)

红框的 `virtualDesktop.exe` 就是代码创建的程序，`calc.exe` 就是计算器的进程，大家可以看一下任务栏，是没有计算器的影踪的，证明是运行在另一个桌面上，从另一个角度来看，就是运行在后台（只要不切换到另一桌面）。

## 上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || 虚拟桌面技术的初步探讨.rar |
| :--- | :--- | :--- |

