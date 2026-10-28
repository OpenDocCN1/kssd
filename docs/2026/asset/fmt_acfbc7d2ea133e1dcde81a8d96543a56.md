
```c
#include <windows.h>
#include "resource.h"
LRESULT CALLBACK WndProc(HWND, UINT, WPARAM, LPARAM);
BOOL CALLBACK AboutDlgProc(HWND, UINT, WPARAM, LPARAM);
int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance,
PSTR szCmdLine, int iCmdShow)
{
static TCHAR szAppName[] = TEXT("About1");
static TCHAR szMenuName[] = TEXT("IDR_MENU1");
MSG msg;
HWND hwnd;
WNDCLASS wndclass;
wndclass.style = CS_HREDRAW | CS_VREDRAW;
wndclass.lpfnWndProc = WndProc;
wndclass.cbClsExtra = 0;
wndclass.cbWndExtra = 0;
wndclass.hInstance = hInstance;
wndclass.hIcon = LoadIcon(hInstance, szAppName);
wndclass.hCursor = LoadCursor(NULL, IDC_ARROW);
wndclass.hbrBackground = (HBRUSH)GetStockObject(WHITE_BRUSH);
wndclass.lpszMenuName = szMenuName;
wndclass.lpszClassName = szAppName;
if (!RegisterClass(&wndclass))
{
MessageBox(NULL, TEXT("This program requires Windows NT!"),
szAppName, MB_ICONERROR);
return 0;
}
HMENU hMenu;
hMenu = LoadMenu(hInstance, MAKEINTRESOURCE(IDR_MENU1));
hwnd = CreateWindow(szAppName, TEXT("About Box Demo Program"),
WS_OVERLAPPEDWINDOW,
CW_USEDEFAULT, CW_USEDEFAULT,
CW_USEDEFAULT, CW_USEDEFAULT,
NULL, hMenu, hInstance, NULL);
ShowWindow(hwnd, iCmdShow);
UpdateWindow(hwnd);
while (GetMessage(&msg, NULL, 0, 0))
{
TranslateMessage(&msg);
DispatchMessage(&msg);
}
return msg.wParam;
}
LRESULT CALLBACK WndProc(HWND hwnd, UINT message, WPARAM wParam, LPARAM lParam)
{
static HINSTANCE hInstance;
switch (message)
{
case WM_CREATE:
hInstance = ((LPCREATESTRUCT)lParam)->hInstance;
return 0;
case WM_COMMAND:
switch (LOWORD(wParam))
{
case ID_HELP_ABOUTABOUT1:
DialogBox(hInstance, TEXT("AboutBox"), hwnd, AboutDlgProc);
break;
}
return 0;
case WM_DESTROY:
PostQuitMessage(0);
return 0;
}
return DefWindowProc(hwnd, message, wParam, lParam);
}
BOOL CALLBACK AboutDlgProc(HWND hDlg, UINT message,
WPARAM wParam, LPARAM lParam)
{
switch (message)
{
case WM_INITDIALOG:
return TRUE;
case WM_COMMAND:
switch (LOWORD(wParam))
{
case IDOK:
case IDCANCEL:
EndDialog(hDlg, 0);
return TRUE;
}
break;
}
return FALSE;
}
```

### 11.2.更复杂的对话框

`About2.rc`

```c
// Microsoft Visual C++ generated resource script.
//
#include "resource.h"
#define APSTUDIO_READONLY_SYMBOLS
/////////////////////////////////////////////////////////////////////////////
//
// Generated from the TEXTINCLUDE 2 resource.
//
#include "winres.h"
/////////////////////////////////////////////////////////////////////////////
#undef APSTUDIO_READONLY_SYMBOLS
/////////////////////////////////////////////////////////////////////////////
// 中文(简体，中国) resources
#if !defined(AFX_RESOURCE_DLL) || defined(AFX_TARG_CHS)
LANGUAGE LANG_CHINESE, SUBLANG_CHINESE_SIMPLIFIED
#pragma code_page(936)
#ifdef APSTUDIO_INVOKED
/////////////////////////////////////////////////////////////////////////////
//
// TEXTINCLUDE
//
1 TEXTINCLUDE
BEGIN
"resource.h\0"
END
2 TEXTINCLUDE
BEGIN
"#include ""winres.h""\r\n"
"\0"
END
3 TEXTINCLUDE
BEGIN
"\r\n"
"\0"
END
#endif // APSTUDIO_INVOKED
/////////////////////////////////////////////////////////////////////////////
//
// Icon
//
// Icon with lowest ID value placed first to ensure application icon
// remains consistent on all systems.
ABOUT2 ICON "ABOUT2.ico"
/////////////////////////////////////////////////////////////////////////////
//
// Menu
//
IDR_MENU1 MENU
BEGIN
POPUP "Help"
BEGIN
MENUITEM "About", ID_HELP_ABOUT
END
END
/////////////////////////////////////////////////////////////////////////////
//
// Dialog
//
AboutBox DIALOGEX 0, 0, 359, 219
STYLE DS_SETFONT | DS_MODALFRAME | DS_FIXEDSYS | WS_POPUP | WS_SYSMENU
FONT 8, "MS Shell Dlg", 400, 0, 0x1
BEGIN
DEFPUSHBUTTON "确定",IDOK,77,191,50,14
PUSHBUTTON "取消",IDCANCEL,155,188,50,14
ICON "About2",IDC_STATIC,18,19,21,20
LTEXT "About2",IDC_STATIC,102,15,81,17
LTEXT "About derek Demo Program",IDC_STATIC,67,32,184,10
GROUPBOX "Color",IDC_STATIC,36,52,81,127
RADIOBUTTON "Black",IDC_BLACK,44,63,32,10,WS_TABSTOP
RADIOBUTTON "Blue",IDC_BLUE,44,77,29,10
RADIOBUTTON "Green",IDC_GREEN,44,91,35,10
RADIOBUTTON "Cyan",IDC_CYAN,44,105,33,10
RADIOBUTTON "Red",IDC_RED,44,119,29,10
RADIOBUTTON "Magenta",IDC_MAGENTA,43,133,44,10
RADIOBUTTON "Yellow",IDC_YELLOW,43,147,36,10
RADIOBUTTON "White",IDC_WHITE,44,161,35,10
GROUPBOX "Figure",IDC_STATIC,209,126,83,49
RADIOBUTTON "Rectangle",IDC_RECT,215,140,48,10,WS_TABSTOP
RADIOBUTTON "Elipse",IDC_ELLIPSE,215,158,34,10
LISTBOX IDC_PAINT,191,52,116,63,LBS_SORT | LBS_NOINTEGRALHEIGHT | WS_VSCROLL | WS_TABSTOP
END
/////////////////////////////////////////////////////////////////////////////
//
// DESIGNINFO
//
#ifdef APSTUDIO_INVOKED
GUIDELINES DESIGNINFO
BEGIN
AboutBox, DIALOG
BEGIN
LEFTMARGIN, 7
RIGHTMARGIN, 352
TOPMARGIN, 7
BOTTOMMARGIN, 212
END
END
#endif // APSTUDIO_INVOKED
/////////////////////////////////////////////////////////////////////////////
//
// AFX_DIALOG_LAYOUT
//
AboutBox AFX_DIALOG_LAYOUT
BEGIN
0
END
#endif // 中文(简体，中国) resources
/////////////////////////////////////////////////////////////////////////////
#ifndef APSTUDIO_INVOKED
/////////////////////////////////////////////////////////////////////////////
//
// Generated from the TEXTINCLUDE 3 resource.
//
/////////////////////////////////////////////////////////////////////////////
#endif // not APSTUDIO_INVOKED
```

`resource.h`

```c
//{{NO_DEPENDENCIES}}
// Microsoft Visual C++ 生成的包含文件。
// 供 About2.rc 使用
//
#define IDI_ICON1 101
#define IDR_MENU1 102
#define IDD_DIALOG1 105
#define IDC_BLACK 1001
#define IDC_BLUE 1002
#define IDC_GREEN 1003
#define IDC_CYAN 1004
#define IDC_RED 1005
#define IDC_MAGENTA 1006
#define IDC_YELLOW 1007
#define IDC_WHITE 1010
#define IDC_RECT 1011
#define IDC_ELLIPSE 1012
#define IDC_PAINT 1013
#define ID_HELP_ABOUT 40001
// Next default values for new objects
//
#ifdef APSTUDIO_INVOKED
#ifndef APSTUDIO_READONLY_SYMBOLS
#define _APS_NEXT_RESOURCE_VALUE 106
#define _APS_NEXT_COMMAND_VALUE 40002
#define _APS_NEXT_CONTROL_VALUE 1014
#define _APS_NEXT_SYMED_VALUE 101
#endif
#endif
```

`About2.c`
