
# Beyond compare 1.9e 注册机! (3千字)

由于工作的关系很久没来这了, 利用回家休假, 破了一套(勉强一点说还包括`cuteftp4.24`) 特贴上来与大家共享

```c
// Key Generator for Beyond compare 1.9e(latest version), an excellent file
compare program by
// www.scootersoftware.com
// Cracked by BPX

#include <windows.h>
#include <tchar.h>
#include "resource.h"

// Return value must be of BOOL type
BOOL CALLBACK DlgProc(HWND, UINT, WPARAM, LPARAM);
BOOL CALLBACK AboutDlgProc(HWND, UINT, WPARAM, LPARAM);

int WINAPI _tWinMain(HINSTANCE hInstance, HINSTANCE, LPTSTR, int)
{
    DialogBoxParam(hInstance, MAKEINTRESOURCE(IDD_DIALOG1), NULL,
DlgProc, (LPARAM)hInstance);
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
        SendMessage(GetDlgItem(hWnd, IDC_NAME),
EM_LIMITTEXT, 120, 0L);
        SendMessage(GetDlgItem(hWnd, IDC_ORGANIZATION),
EM_LIMITTEXT, 120, 0L);
        hSysMenu = GetSystemMenu(hWnd, FALSE);
        if (hSysMenu != NULL) {
            AppendMenu(hSysMenu,
MF_SEPARATOR, NULL, NULL);
            // IDM_ABOUT must be
in the system command range.
            // IDM_ABOUT &
0xFFF0) == IDM_ABOUTBOX
            // IDM_ABOUT < 0xF000
            AppendMenu(hSysMenu,
MF_STRING, IDM_ABOUT, TEXT("关于..."));
        }
        // tell the default procedure to set focus
to the first non-disabled
        // control that have a tab-stop.
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
            DialogBoxParam(GetModuleHandle(NULL),
MAKEINTRESOURCE(IDD_ABOUT), NULL, AboutDlgProc, 0);
            return TRUE;
        }
    }
    return FALSE;
}

void CalcKey(HWND hWnd)
{
    TCHAR buf[260] = "Beyond Compare 1";
    TCHAR name[120], organization[120], key[9];
    unsigned char data[1024];
    int *p, esi, ecx, eax, edx;
    for (esi = 0, p = (int*)&data[0]; esi < 256; esi++)
{
        for (ecx = 0, eax = esi << 24; ecx
< 8; ecx++) {
            edx = 0;
            if ((eax & 0x8000000)
!= 0)
                edx
= 0x4c11db7;
            eax += eax;
            edx ^= eax;
            eax = edx;
        }
        *p++ = eax;
    }
    GetDlgItemText(hWnd, IDC_NAME, name, sizeof(name));
    GetDlgItemText(hWnd, IDC_ORGANIZATION, organization, sizeof(organization));
    _tcsncat(buf, name, _tcslen(name));
    _tcsncat(buf, organization, _tcslen(organization));
    _tcsupr(buf);
    int len = _tcslen(buf);
    for (int i = 0; i < len; i++) {
        if ((unsigned char)(buf[i] - 'A') <
26)
            buf[i] = buf[i] - 'A'
+ 10;
        else if ((unsigned char)(buf[i] - '0')
< 10)
            buf[i] = buf[i] - '0';
        else
            _tcsncpy(&buf[i],
&buf[i+1], len - i - 1), len--, i--;
    }
    for (eax = edx = 0, ecx = len, p = (int*)data; ecx > 0; ecx--,
edx++) {
        esi = buf[edx];
        esi ^= eax >> 24;
        esi &= 0xff;
        esi = p[esi];
        eax <<= 8;
        esi ^= eax;
        eax = esi;
    }
    wsprintf(key, "%08x", eax);
    SetDlgItemText(hWnd, IDC_KEY, key);
}
```
