
# 利用 Thunk 技术改写窗口类回调函数例子

> 原文：[`www.pediy.com/kssd/pediy13/133178.html`](https://www.pediy.com/kssd/pediy13/133178.html)

## 什么是 Thunk

Thunk 技术就是将一段机器码对应的字节保存在一个连续内存结构里，然后把其指针强制转换成函数，即用作函数来执行。

## 什么是 DEP

说道 Thunk，顺便介绍下 DEP。数据执行保护 (DEP) 是一套软硬件技术，能够在内存上执行额外检查以帮助防止在系统上运行恶意代码。在 `Microsoft Windows XP Service Pack 2`、`Microsoft Windows Server 2003 Service Pack 1`、`Microsoft Windows XP Tablet PC Edition 2005` 和 `Microsoft Windows Vista` 中，由硬件和软件一起强制实施 DEP。DEP 的主要优点是可以帮助防止数据页执行代码。通常情况下，不从默认堆和堆栈执行代码。硬件实施 DEP 检测从这些位置运行的代码，并在发现执行情况时引发异常。软件实施 DEP 可帮助阻止恶意代码利用 Windows 中的异常处理机制进行破坏。硬件实施 DEP 是某些 DEP 兼容处理器的功能，可以防止在已标记为数据存储区的内存区域中执行代码。此功能也称为非执行和执行保护。`Windows XP SP2` 还包括软件实施 DEP，其目的在于减少利用 Windows 中的例外处理机制的情况。

## 在哪里查看 DEP 设置呢?

我的电脑-> 属性-> 高级-> 性能-> 设置-> 数据执行保护。

在 `windows xp sp2` 上默认是“仅为基本 Windows 程序和服务启用 DEP”；在 `Microsoft Windows Server 2003 Service Pack 2` 上默认是“为除下列选定程序之外的所有程序和服务启用 DEP”。

## WTL 是怎样解决 DEP 问题

WTL 也用了 Thunk 技术，如果在 `Microsoft Windows Server 2003 Service Pack 2` 上把 DEP 设置成回默认的，然后运行一下 WTL 的 `Dialog`，可以跑起来，同样是用 Thunk，这是什么原因呢？看 `ATL` 代码，可以发现如下代码：

```
thunkPage = (PATL_THUNK_ENTRY)VirtualAlloc(NULL, 
               PAGE_SIZE, 
                MEM_COMMIT, 
                PAGE_EXECUTE_READWRITE); 
```

原来 WTL 是用 `VirtualAlloc[PAGE_EXECUTE_READWRITE]` 来分配 Thunk 代码的内存。所以要是你的代码中使用的 Thunk 技术，请使用 `VirtualAlloc` 为 Thunk 分配内存。

下面给出一个使用 Thunk 改写窗口类回调函数为类成员函数的例子：

注意：

1. 这个例子从网上[利用 thunk 技术封装窗口类](http://www.cnblogs.com/charm/archive/2011/04/19/2021031.html)改写过来，但是他的程序有些 bug，下载下来直接编译会出错。
2. 利用 thunk 技术封装窗口类中使用的是静态变量传递窗口对象和窗口句柄，需要定义额外的变量，代码中使用[RingSDK](http://blog.csdn.net/ringphone/archive/2008/09/11/2911244.aspx)中的技术，在 `CreateWindow` 中将对象指针作为参数传递。

代码:

```cpp
// TestTrunk.cpp : 定义应用程序的入口点。
//

#include "stdafx.h"
#include "TestTrunk.h"

#define MAX_LOADSTRING 100


#pragma pack(push,1)
typedef struct _StdCallThunk
{
  DWORD   m_mov;          // = 0x042444C7
  DWORD   m_this;         // = this
  BYTE    m_jmp;          // = 0xe9
  DWORD   m_relproc;      // = relative distance
} StdCallThunk;
#pragma pack(pop)
class CMyWindow
{
public:
  CMyWindow():_hwnd(NULL){}
  ~CMyWindow(){VirtualFree(_pStdthunk, sizeof(StdCallThunk), MEM_RELEASE);}
  bool Create();


protected:
  LRESULT CALLBACK WndProc(UINT message, WPARAM wParam, LPARAM lParam);
  MSG _msg;
  HWND _hwnd;
  StdCallThunk **_pStdthunk;
  static LRESULT CALLBACK TempWndProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam);
  static LRESULT CALLBACK StaticWndProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam);

};

bool CMyWindow::Create()
{
  WNDCLASSEX wcex;
  LPCTSTR lpszClassName = _T("ClassName");
  wcex.cbSize = sizeof(WNDCLASSEX);
  wcex.style      = CS_HREDRAW | CS_VREDRAW;
  wcex.lpfnWndProc  = CMyWindow::TempWndProc;
  wcex.cbClsExtra    = 0;
  wcex.cbWndExtra    = 0;
  wcex.hInstance    = GetModuleHandle(NULL);
  wcex.hIcon      = LoadIcon(GetModuleHandle(NULL), MAKEINTRESOURCE(IDI_TESTTRUNK));
  wcex.hCursor    = LoadCursor(NULL, IDC_ARROW);
  wcex.hbrBackground  = (HBRUSH)(COLOR_WINDOW+1);
  wcex.lpszMenuName  = MAKEINTRESOURCE(IDC_TESTTRUNK);
  wcex.lpszClassName  = lpszClassName;
  wcex.hIconSm    = LoadIcon(wcex.hInstance, MAKEINTRESOURCE(IDI_SMALL));

  RegisterClassEx(&wcex);

  _pStdthunk = (StdCallThunk *)VirtualAlloc(NULL, sizeof(StdCallThunk), MEM_RESERVE | MEM_COMMIT, PAGE_EXECUTE_READWRITE);
  _pStdthunk->m_mov = 0x042444c7;
  _pStdthunk->m_jmp = 0xe9;

  //CreateWindow 的最后一个参数为 this 指针
  CreateWindow(lpszClassName, NULL, WS_OVERLAPPEDWINDOW,
    CW_USEDEFAULT, 0, CW_USEDEFAULT, 0, NULL, NULL, GetModuleHandle(NULL),this);

  if (_hwnd == NULL)
  {
    MessageBox(NULL,TEXT("Error"),NULL,NULL);
    return FALSE;
  }

  ShowWindow(_hwnd, SW_SHOW);
  UpdateWindow(_hwnd);

  return TRUE;
}

LRESULT CALLBACK CMyWindow::TempWndProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam)
{
  if (message == WM_CREATE)//此处使用 WM_NCCREATE 消息也可以
  {
    //提取出对象指针
    CMyWindow *w = NULL;
    w = (CMyWindow*)((LPCREATESTRUCT)lParam)->lpCreateParams;
    w->_hwnd = hWnd;
    WNDPROC pWndProc = (WNDPROC)w->_pStdthunk;
    w->_pStdthunk->m_this = (DWORD)w;

    //计算跳转位置
    w->_pStdthunk->m_relproc = (DWORD)&CMyWindow::StaticWndProc - ((DWORD)w->_pStdthunk + sizeof(StdCallThunk));

    SetWindowLong(hWnd, GWL_WNDPROC, (LONG)pWndProc);
    return pWndProc( hWnd,  message, wParam, lParam);
  }
  return DefWindowProc(hWnd, message, wParam, lParam);
}

LRESULT CALLBACK CMyWindow::StaticWndProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam)
{
  return ((CMyWindow *)hWnd)->WndProc(message, wParam,  lParam);
}

LRESULT CALLBACK CMyWindow::WndProc(UINT message, WPARAM wParam, LPARAM lParam)
{
  int wmId, wmEvent;
  PAINTSTRUCT ps;
  HDC hdc;
  switch (message)
  {

  case WM_COMMAND:
    wmId    = LOWORD(wParam);
    wmEvent = HIWORD(wParam);
    // 分析菜单选择:
    switch (wmId)
    {
    case IDM_ABOUT:
      MessageBox(NULL,TEXT("AboutDlg"),TEXT("关于"),NULL);
      break;
    case IDM_EXIT:
      DestroyWindow(_hwnd);
      break;
    }
    break;
  case WM_PAINT:
    hdc = BeginPaint(_hwnd, &ps);
    // TODO: 在此添加任意绘图代码...
    EndPaint(_hwnd, &ps);
    break;
  case WM_DESTROY:
    PostQuitMessage(0);
    break;

  default:
    return DefWindowProc(_hwnd, message, wParam, lParam);
    break;
  }
  return TRUE;
}

int APIENTRY _tWinMain(HINSTANCE hInstance,
                      HINSTANCE hPrevInstance,
                      LPTSTR    lpCmdLine,
                      int       nCmdShow)
{
  UNREFERENCED_PARAMETER(hPrevInstance);
  UNREFERENCED_PARAMETER(lpCmdLine);

  CMyWindow pMyWindow;
  pMyWindow.Create();
  MSG msg;
  HACCEL hAccelTable =  LoadAccelerators(GetModuleHandle(NULL), MAKEINTRESOURCE(IDC_TESTTRUNK));
  while (GetMessage(&msg, NULL, 0, 0))
  {
    if (!TranslateAccelerator(msg.hwnd, hAccelTable, &msg))
    {
      TranslateMessage(&msg);
      DispatchMessage(&msg);
    }
  }
  return 0;
}
```

最后，我也是参考网上的东西，发这份代码主要是想把我的学习心得和大家一起分享。要是有什么错误和不足，请大家多多包涵。

上传的附件

| ![](img/620251b33676a8e4b98ca0ece915ee8d.jpg) || `TestTrunk.zip` |
|---|---|

1. 由于建立项目时疏忽，把 `TestThunk` 建成了 `TestTrunk` 给大家带来阅读不便，请谅解。
2. 由于工作后很少研究底层东西了，除了在 `COM` 里面看到一些关于 `stub`。在网上看到 Thunk，姑且当做 thunk 吧。

看到论坛里帖子[`bbs.pediy.com/showthread.php?t=76213&highlight=`](http://bbs.pediy.com/showthread.php?t=76213&highlight=)发现可以只用一个静态函数实现窗口类回调函数变为非静态成员函数。主要是在

代码:

```cpp
LRESULT CALLBACK CMyWindow::TempWndProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam)
{
  if (message == WM_CREATE)//此处使用 WM_NCCREATE 消息也可以
  {
    //提取出对象指针
    CMyWindow *w = NULL;
    w = (CMyWindow*)((LPCREATESTRUCT)lParam)->lpCreateParams;
    w->_hwnd = hWnd;
   //在此直接使用非静态成员函数，this 指针通过 ecx 传递。
    void* pProc = w->m_PorcA.Thiscall(w, Thunk::GetMemberFxnAddr(&CMyWindow::WndProc));
    SetWindowLong(hWnd, GWL_WNDPROC, (LONG)pProc);
     
  }
  return DefWindowProc(hWnd, message, wParam, lParam);
}
```

做以上更改。

在此感谢 `zhangludu`，这份代码中使用了他写的的 `thunk` 类，真的很好用。以前一直在想着怎样使用 thunk 实现非静态成员函数作为线程函数，没想到他早就实现了。希望大家继续多提意见，![](img/c1d8523a7842d7b67917c210ac6dcf18.jpg "微笑")。

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || `TestTrunk2.rar` |
|---|---|
