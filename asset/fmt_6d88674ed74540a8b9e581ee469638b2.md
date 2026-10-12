
# 屏蔽 WINDOWS 热键

> 原文：[`www.pediy.com/kssd/pediy12/131423.html`](https://www.pediy.com/kssd/pediy12/131423.html)

最近帮一个朋友实现这个功能，和大家分享一下，参考了一部分网上代码。

在屏蔽 `ctrl+alt+del` 用了 `sas windos` 这个东东，貌似 `vista` 以后的系统失效了啦![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")

测试环境为 `win2k`, `winxp sp2`, `winxp sp3`

```cpp
#define _WIN32_WINNT 0x0500 //Use WH_KEYBOARD_LL

#include "stdafx.h"

#include <stdio.h>
#include <string>
#include <stdlib.h>
#include <winuser.h>

//SAS window 句柄
HWND hSASWnd = NULL;

//原有 SAS window 回调函数地址
FARPROC FOldProc = NULL;

//起屏蔽作用的新 SAS window 回调函数
LRESULT CALLBACK SASWindowProc(HWND hwnd,UINT uMsg,WPARAM wParam,LPARAM lParam);

//枚举所有窗体句柄的回调函数
BOOL CALLBACK EnumWindowsProc(HWND hwnd,LPARAM lParam);

//Dll 所创建线程的句柄
HANDLE hThread = NULL;

//Dll 所创建线程的 ID
DWORD dwThreadId = 0;

//Dll 所创建线程的线程函数
DWORD WINAPI ThreadFunc();

//_H 钩子句柄
HHOOK hHook = NULL;

//_H 低级键盘钩子回调函数
LRESULT CALLBACK KeyboardProc(int,WPARAM,LPARAM);

//对外输出字符串
char szOutput[36];

BOOL APIENTRY DllMain(HANDLE hMoudle, DWORD dwReason, LPVOID lpReserved)
{
    switch(dwReason)
    {
    case DLL_PROCESS_ATTACH:
        wsprintf(szOutput,"Dll 成功加载于 %d 号进程 !\n",GetCurrentProcessId());
        OutputDebugString(szOutput);
        //创建更替 SAS window 回调函数的线程
        if(FOldProc == NULL)
            hThread = CreateThread(NULL,0,(LPTHREAD_START_ROUTINE)ThreadFunc,NULL,0,&dwThreadId);
        break;
    case DLL_PROCESS_DETACH:
        wsprintf(szOutput,"Dll 成功卸载 !\n",GetCurrentProcessId());
        //MessageBox(NULL, szOutput, "ZZ", MB_ICONINFORMATION | MB_OK);
        OutputDebugString(szOutput);
        //恢复原有 SAS window 的回调函数
        if(FOldProc != NULL)
            SetWindowLong(hSASWnd,GWL_WNDPROC,long(FOldProc));
        //_H 卸载低级键盘钩子
        if(hHook != NULL)
        {
            if(!UnhookWindowsHookEx(hHook))
            {
                OutputDebugString("键盘钩子卸载失败 !\n");
                //__leave;
                break;
            }
            OutputDebugString("键盘钩子卸载成功 !\n");
        }
        TerminateThread(hThread,1);
        CloseHandle(hThread);
        break;
    case DLL_THREAD_ATTACH:
        break;
    case DLL_THREAD_DETACH:
        break;
    }
    return TRUE;
}

//Dll 所创建线程的线程函数
DWORD WINAPI ThreadFunc()
{
    //打开 Winlogon 桌面
    HDESK hDesk = OpenDesktop("Winlogon",0,FALSE,MAXIMUM_ALLOWED);
    //枚举桌面所有窗体
    EnumDesktopWindows(hDesk,(WNDENUMPROC)EnumWindowsProc,0);
    //修改 SAS window 的回调函数
    if(hSASWnd != NULL)
    {
        FOldProc = (FARPROC)SetWindowLong(hSASWnd,GWL_WNDPROC,long(SASWindowProc));
    }
    CloseHandle(hDesk);
    //_H 同一桌面上进程之间只能发送窗口消息。无法跨进程与其他桌面发送它们。
    //_H 同样，Windows 消息是限制应用程序定义挂钩。
    //_H 特定桌面中运行的进程挂钩过程将〈〈只获得针对同一桌面上创建窗口消息。〉〉
    //_H 所以，这里必须设置钩子所在线程的桌面为 Default 桌面
    //_H 才能使得钩子所在线程能接收到 Default 桌面的消息
    hDesk = OpenDesktop("Default",0,FALSE,MAXIMUM_ALLOWED);
    SetThreadDesktop(hDesk);
    CloseHandle(hDesk);
    //_H 设置低级键盘钩子，屏蔽非 SAS window 的热键
    //_H 需要#define _WIN32_WINNT 0x0500
    hHook = SetWindowsHookEx(WH_KEYBOARD_LL,KeyboardProc,GetModuleHandle(NULL),0);
    if (hHook == NULL)
    {
        OutputDebugString("键盘钩子设置失败 !\n");
        //__leave;
        return 1;
    }
    OutputDebugString("键盘钩子设置成功 !\n");
    //_H 在非 GUI 线程中使用消息钩子必须主动接收并分发收到的消息
    MSG msg;
    while(GetMessage(&msg, NULL, 0, 0))
    {
        TranslateMessage(&msg);
        DispatchMessage(&msg);
    }
    return 1;
}

//枚举所有窗体句柄的回调函数
BOOL CALLBACK EnumWindowsProc(HWND hwnd,LPARAM lParam)
{
    char ClassBuf[128];
    //获得当前窗体的显示文本
    GetWindowText(hwnd,ClassBuf,sizeof(ClassBuf));
    //在"Winlogon"桌面中查询窗口"SAS window"。
    if(strstr(ClassBuf,"SAS window")!=NULL)
    {
        //返回 SAS window 句柄
        hSASWnd = hwnd;
        return FALSE;
    }
    return TRUE;
}

//起屏蔽作用的新 SAS window 回调函数
LRESULT CALLBACK SASWindowProc(HWND hwnd,UINT uMsg,WPARAM wParam,LPARAM lParam)
{
    if(uMsg == WM_HOTKEY)
    {
        //屏蔽所有 WM_HOTKEY 消息
        OutputDebugString("All SAS window's hotkeys are disabled\n");
        return 1;
        WORD wKey = HIWORD(lParam);
        WORD wModifier = LOWORD(lParam);
        bool IsCtrlDown = ((wModifier & VK_CONTROL) != 0);
        bool IsAltDown = ((wModifier & VK_MENU) != 0);
        bool IsShiftDown = ((wModifier & VK_SHIFT) != 0);
        //Ctrl + Alt + Del 组合键
        if(IsCtrlDown && IsAltDown && wKey == VK_DELETE)
        {
            return 1; //屏蔽
        }
        //Ctrl + Shift + Esc 组合键，这个组合键将显示任务管理器，可根据需要是否屏蔽。
        else if(IsCtrlDown && IsShiftDown && wKey == VK_ESCAPE)
        {
            return 1;
            // Do nothing
        }
    }
    return CallWindowProc((WNDPROC)FOldProc,hwnd,uMsg,wParam,lParam);
}

//_H 低级键盘钩子回调函数
LRESULT CALLBACK KeyboardProc(int nCode,WPARAM wParam,LPARAM lParam)
{
    if (nCode == HC_ACTION)
    {
        switch (wParam)
        {
        case WM_KEYDOWN:  case WM_SYSKEYDOWN:
            //case WM_KEYUP:    case WM_SYSKEYUP:
            PKBDLLHOOKSTRUCT p = (PKBDLLHOOKSTRUCT) lParam;
            if (p->vkCode == VK_F12)
            {
                //实现模拟按键代码
                MessageBox(GetForegroundWindow(),"O(∩_∩)O is by Angel XHZ","Test",MB_OK);
            }
            //屏蔽 ALT+TAB
            else if ((p->vkCode == VK_TAB) && ((p->flags & LLKHF_ALTDOWN) != 0))
            {
                OutputDebugString("ALT+TAB is disabled\n");
                return 1;
            }
            //屏蔽 ALT+ESC
            else if ((p->vkCode == VK_ESCAPE) && ((p->flags & LLKHF_ALTDOWN) != 0))
            {
                OutputDebugString("ALT+ESC is disabled\n");
                return 1;
            }
            //屏蔽 CTRL+ESC
            else if ((p->vkCode == VK_ESCAPE) && ((GetKeyState(VK_CONTROL) & 0x8000) != 0))
            {
                OutputDebugString("CTRL+ESC is disabled\n");
                return 1;
            }
            //屏蔽 CTRL+SHIFT+ESC，（SAS window 中也已屏蔽）
            else if ((p->vkCode == VK_ESCAPE) &&
                ((GetKeyState(VK_CONTROL) & 0x8000) != 0) &&
                ((GetKeyState(VK_SHIFT) & 0x8000) != 0))
            {
                OutputDebugString("CTRL+SHIFT+ESC is disabled\n");
                return 1;
            }
            //屏蔽左右 windows 键
            else if (p->vkCode == VK_LWIN || p->vkCode == VK_RWIN)
            {
                OutputDebugString("windows key is disabled\n");
                return 1;
            }
            //此处无法屏蔽 CTRL+ALT+DEL，已在 SAS window 中屏蔽
            else if ((p->vkCode == VK_DELETE) &&
                ((GetKeyState(VK_CONTROL) & 0x8000) != 0) &&
                ((GetKeyState(VK_MENU) & 0x8000) != 0 ))
            {
                return 1;
            }
            break;
        }
    }
    return CallNextHookEx(hHook,nCode,wParam,lParam);
}
```

`DLL` 部分源代码，附件为完整源代码。

另外请教各位大牛，在 `vista` 以后的系统如何实现屏蔽 `ctrl+alt+del` 键，除了驱动之外因为这东东是给其他项目调用，偶试了很多法子都失败告终![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")，有法子的一定联系我哦`!~!![](img/afde35a5f3b3a4675abb3221cf07c6d1.jpg "124.gif")![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")`

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) | InjectionDLL.rar |
| --- | --- |
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) | winlogdll_vc6.rar |

