
# HOOK 笔记

> 原文：[`www.pediy.com/kssd/pediy10/90178.html`](https://www.pediy.com/kssd/pediy10/90178.html)

`SendMessage()` 函数发送的消息并不进入消息对列等待 `GetMessage()` 函数取出，而是直接传给窗口函数。

示例：

```c
LRESULT CALLBACK MouseProc(
  int nCode,      // hook code
  WPARAM wParam,  // message identifier
  LPARAM lParam   // mouse coordinates
)
{
  return 1;       //钩子函数返回非零值，系统不再把消息送到相应应用程序
}

HHOOK g_hKeyboard=NULL;

LRESULT CALLBACK KeyboardProc(
  int code,       // hook code
  WPARAM wParam,  // virtual-key code
  LPARAM lParam   // keystroke-message information
)
{
if(VK_SPACE==wParam)//仅屏蔽掉空格键，可鼠标右击，用 go to definition 查看其他虚拟键
      return 1;
//if(VK_F4==wParam&&(1==(lParam>>29&1)))
  //return 1;   
//屏蔽组合键 Alt+F4，lParam 第 29 位请查 MSDN
  else 
    return ::CallNextHookEx(g_hKeyboard,code,wParam,lParam);
}

BOOL CHookDemoDlg::OnInitDialog()
{
…….
::SetWindowsHookEx(WH_MOUSE,MouseProc,NULL,::GetCurrentThreadId());
  g_hKeyboard=::SetWindowsHookEx(WH_KEYBOARD,KeyboardProc,NULL,::GetCurrentThreadId());
}
```

## 为自己程序留后门

编写这样的钩子函数：

```c
LRESULT CALLBACK KeyboardProc(
  int code,       // hook code
  WPARAM wParam,  // virtual-key code
  LPARAM lParam   // keystroke-message information
)
{
//  if(VK_SPACE==wParam)
/*  if(VK_F4==wParam&&(1==(lParam>>29&1)))
      return 1;
  else 
    return ::CallNextHookEx(g_hKeyboard,code,wParam,lParam);*/
  if(VK_F2==wParam)
  {
    ::SendMessage(g_hWnd,WM_CLOSE,0,0);
    UnhookWindowsHookEx(g_hKeyboard);
    UnhookWindowsHookEx(g_hKeyboard);
  }
  return 1;
}
```

要让钩子过程与所有线程相关，必须把 `SetWindowsHookEx` 第四个参数设为 `0`，第三个为钩子函数所在 DLL。

## 安装关联到所有线程的钩子

将挂钩函数写入 DLL 中，记得声明它为导出函数，在 `.def` 文件中，`.dll` 中

```c
#include<windows.h>

HHOOK g_hMouse;

HHOOK g_hKeyboard=NULL;

HWND g_hWnd;

/*HINSTANCE g_hInst;
BOOL WINAPI DllMain(
  HINSTANCE hinstDLL,  // handle to the DLL module
  DWORD fdwReason,     // reason for calling function
  LPVOID lpvReserved   // reserved
)
{
  g_hInst=hinstDll;
}
*/

LRESULT CALLBACK MouseProc(
  int nCode,      // hook code
  WPARAM wParam,  // message identifier
  LPARAM lParam   // mouse coordinates
)
{
  return 1;
}

LRESULT CALLBACK KeyboardProc(
  int code,       // hook code
  WPARAM wParam,  // virtual-key code
  LPARAM lParam   // keystroke-message information
)
{
  if(VK_F2==wParam)
  {
    ::SendMessage(g_hWnd,WM_CLOSE,0,0);
    UnhookWindowsHookEx(g_hMouse);
    ::UnhookWindowsHookEx(g_hKeyboard);
  }
  return 1;
}

void SetHook(HWND hwnd)
{   
  g_hWnd=hwnd;
  g_hKeyboard=::SetWindowsHookEx(WH_KEYBOARD,KeyboardProc,GetModuleHandle("HOOk"),0);
  g_hMouse=::SetWindowsHookEx(WH_MOUSE,MouseProc,::GetModuleHandle("HOOK"),0);
  //::SetWindowsHookEx(WH_MOUSE,MouseProc,g_hInst,0);
}
```

`.def` 中

```
LIBRARY HOOK         //此句可无
EXPORTS
SetHook
```

在目的模块中，如先声明其为外部导入的函数

```c
_declspec(dllimport)void SetHook(HWND hwnd);

BOOL CHookTestDlg::OnInitDialog()
{
…………..
  int cxScreen,cyScreen;//全屏幕当前窗口，而且在最顶层。
  cxScreen=::GetSystemMetrics(SM_CXSCREEN);
  cyScreen=::GetSystemMetrics(SM_CYSCREEN);
  SetWindowPos(&wndTopMost,0,0,cxScreen,cyScreen,SWP_SHOWWINDOW);
  SetHook(m_hWnd);
}
```

## 查看动态链接库的节

`dumpbin -headers xxx.dll`

## 动态连接库里的全局变量数据共享问题

**方法一** `.cpp` 中

```c
#pragma data_seg("MySec")

HWND g_hWnd=NULL;

#pragma data_seg()                  //定义一个节

#pragma comment(linker,"/section:Mysec,RWS")//设置连接选项
```

**方法二** `.cpp` 中

```c
#pragma data_seg("MySec")

HWND g_hWnd=NULL;

#pragma data_seg()                  //定义一个节
```

`.def` 中

```
SEGMENTS
MySec READ WRITE SHARED
```
