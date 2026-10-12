
# 竹影棋友

> 原文：[`www.pediy.com/kssd/pediy03/forum1003.htm`](https://www.pediy.com/kssd/pediy03/forum1003.htm)

四月初在即将离开我的上一家公司--东大阿尔派深圳分公司的时候新来一位同事嚷着要和我下围棋。找来找去发现只有用"竹影棋友围棋程序"可在局域网下棋。当时在"盗亦有道"上下载了破解版，在软件中也可以看到`dyyd`字样。可是当下到 250 手时，软件弹出"共享版，只能下 250 手，然后中断连接"。让我一盘大优的棋没有下完。如此"有道":(

只得祭起`wdasm8.93`，(顺便解决了`wdam`不能正确显示种文字串的一个小`bug`) 结合`SoftIce`发现了竹影棋友的注册算法。考虑到这程序有 2 年没有更新，为此贴出其注册机的完整程序。这也正是当时我离开论坛时答应要做的

```c
#include <windows.h>
#include <tchar.h>
#include "resource.h"

BOOL CALLBACK DlgProc(HWND, UINT, WPARAM, LPARAM);
BOOL CALLBACK AboutDlgProc(HWND, UINT, WPARAM, LPARAM);

int WINAPI _tWinMain(HINSTANCE hInstance, HINSTANCE, LPTSTR, int)
{
    DialogBoxParam(hInstance, MAKEINTRESOURCE(IDD_DIALOG1), NULL, DlgProc, (LPARAM)hInstance);
    return 0;
}

BOOL CALLBACK AboutDlgProc(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam)
{
    switch (msg)
    {
    case WM_COMMAND:
        if (LOWORD(wParam) == IDCANCEL) {
            EndDialog(hWnd, 0);
            return TRUE;
        }
    }
    return FALSE;
}

void CalcKey(HWND);

BOOL CALLBACK DlgProc(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam)
{
    HICON hIcon;
    HMENU hSysMenu;
    switch (msg)
    {
    case WM_INITDIALOG:
        hIcon = LoadIcon((HINSTANCE)lParam, MAKEINTRESOURCE(MAINICON));
        SendMessage(hWnd, WM_SETICON, TRUE, (LPARAM)hIcon);
        SendMessage(hWnd, WM_SETICON, FALSE, (LPARAM)hIcon);
        SendMessage(GetDlgItem(hWnd, IDC_EDIT1), EM_LIMITTEXT, 8, 0L);
        SendMessage(GetDlgItem(hWnd, IDC_EDIT2), EM_LIMITTEXT, 8, 0L);
        hSysMenu = GetSystemMenu(hWnd, FALSE);
        if (hSysMenu != NULL) {
            AppendMenu(hSysMenu, MF_SEPARATOR, NULL, NULL);
            AppendMenu(hSysMenu, MF_STRING, IDM_ABOUT, TEXT("关于..."));
        }
        return TRUE;
    case WM_COMMAND:
        if (LOWORD(wParam) == IDCANCEL) {
            EndDialog(hWnd, 0);
            return TRUE;
        }
        else if (LOWORD(wParam) == IDOK) {
            CalcKey(hWnd);
            return TRUE;
        }
        break;
    case WM_SYSCOMMAND:
        if (LOWORD(wParam) == IDM_ABOUT) {
            DialogBoxParam(GetModuleHandle(NULL), MAKEINTRESOURCE(IDD_ABOUT), NULL, AboutDlgProc, 0);
            return TRUE;
        }
    }
    return FALSE;
}

void CalcKey(HWND hWnd)
{
    TCHAR Buf[9];
    TCHAR* StopString;
    GetDlgItemText(hWnd, IDC_EDIT1, Buf, sizeof(Buf));
    if (_tcslen(Buf) != 8) {
        MessageBox(hWnd, TEXT("标识号应为 8 位数字"), TEXT("竹影棋友注册机"), MB_OK | MB_ICONEXCLAMATION);
        SetFocus(GetDlgItem(hWnd, IDC_EDIT1));
        return;
    }
    int ID = _tcstoul(Buf, &StopString, 16);
    int key;
    *(((BYTE*)&key + 0)) = *(((BYTE*)&ID + 2)) * *(((BYTE*)&ID + 1)) % 0xff;
    *(((BYTE*)&key + 1)) = *(((BYTE*)&ID + 3)) * *(((BYTE*)&ID + 0)) % 0xfc;
    *(((BYTE*)&key + 2)) = *(((BYTE*)&ID + 1)) * *(((BYTE*)&ID + 3)) % 0xfe;
    *(((BYTE*)&key + 3)) = *(((BYTE*)&ID + 0)) * *(((BYTE*)&ID + 2)) % 0xfd;
    wsprintf(Buf, "%08x", key);
    SetDlgItemText(hWnd, IDC_EDIT2, Buf);
}
```

`4dd466d0e1e0cc123afbe504da2462ca`
