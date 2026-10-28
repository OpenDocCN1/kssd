
```c
#include <windows.h>
#include "resource.h"
#define ID_EDIT 1
LRESULT CALLBACK WndProc(HWND, UINT, WPARAM, LPARAM);
TCHAR szAppName[] = TEXT("PopPad2");
TCHAR szMenuName[] = TEXT("IDR_MENU1");
int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance,
PSTR szCmdLine, int iCmdShow)
{
HACCEL hAccel;
HWND hwnd;
MSG msg;
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
hwnd = CreateWindow(szAppName, szAppName,
WS_OVERLAPPEDWINDOW,
GetSystemMetrics(SM_CXSCREEN) / 4,
GetSystemMetrics(SM_CYSCREEN) / 4,
GetSystemMetrics(SM_CXSCREEN) / 2,
GetSystemMetrics(SM_CYSCREEN) / 2,
NULL, hMenu, hInstance, NULL);
ShowWindow(hwnd, iCmdShow);
UpdateWindow(hwnd);
//把加速键表加载到内存并获得它的句柄
hAccel = LoadAccelerators(hInstance, szAppName);
while (GetMessage(&msg, NULL, 0, 0))
{
//处理菜单命令的快捷键。 如果指定快捷键表中有项) ，然后将WM_COMMAND或WM_SYSCOMMAND消息直接发送到
//指定的窗口过程，则该函数会将WM_KEYDOWN或WM_SYSKEYDOWN消息转换为WM_COMMAND或WM_SYSCOMMAND消息
if (!TranslateAccelerator(hwnd, hAccel, &msg))
{
TranslateMessage(&msg);
DispatchMessage(&msg);
}
}
return msg.wParam;
}
AskConfirmation(HWND hwnd)
{
return MessageBox(hwnd, TEXT("Really want to close PopPad2?"),
szAppName, MB_YESNO | MB_ICONQUESTION);
}
LRESULT CALLBACK WndProc(HWND hwnd, UINT message, WPARAM wParam, LPARAM lParam)
{
static HWND hwndEdit;
int iSelect, iEnable;
switch (message)
{
case WM_CREATE:
hwndEdit = CreateWindow(TEXT("edit"), NULL,
WS_CHILD | WS_VISIBLE | WS_HSCROLL | WS_VSCROLL |
WS_BORDER | ES_LEFT | ES_MULTILINE |
ES_AUTOHSCROLL | ES_AUTOVSCROLL,
0, 0, 0, 0, hwnd, (HMENU)ID_EDIT,
((LPCREATESTRUCT)lParam)->hInstance, NULL);
return 0;
case WM_SETFOCUS:
SetFocus(hwndEdit);
return 0;
case WM_SIZE:
MoveWindow(hwndEdit, 0, 0, LOWORD(lParam), HIWORD(lParam), TRUE);
return 0;
case WM_INITMENUPOPUP:
//Edit在菜单中位置索引为1
if (lParam == 1)
{
EnableMenuItem((HMENU)wParam, ID_EIDT_UNDO,
SendMessage(hwndEdit, EM_CANUNDO, 0, 0) ?
MF_ENABLED : MF_GRAYED);
//只有当剪切板当前包好文本时，Paste菜单项才应该被启用
EnableMenuItem((HMENU)wParam, ID_EIDT_PASTE,
IsClipboardFormatAvailable(CF_TEXT) ?
MF_ENABLED : MF_GRAYED);
//只有在编辑控件中有文本被选中时，Cut、Copy和Deete选项才应该被启用
iSelect = SendMessage(hwndEdit, EM_GETSEL, 0, 0);
if (HIWORD(iSelect) == LOWORD(iSelect))
iEnable = MF_GRAYED;
else
iEnable = MF_ENABLED;
EnableMenuItem((HMENU)wParam, ID_EIDT_CUT, iEnable);
EnableMenuItem((HMENU)wParam, ID_EIDT_COPY, iEnable);
EnableMenuItem((HMENU)wParam, ID_EIDT_DELETE, iEnable);
return 0;
}
break;
case WM_COMMAND:
if (lParam)
{
if (LOWORD(lParam) == ID_EDIT &&
(HIWORD(wParam) == EN_ERRSPACE ||
HIWORD(wParam) == EN_MAXTEXT))
MessageBox(hwnd, TEXT("Edit control out of space."),
szAppName, MB_OK | MB_ICONSTOP);
return 0;
}
else switch (LOWORD(wParam))
{
case ID_FILE_NEW:
case ID_FILE_OPEN:
case ID_FILE_SAVE:
case ID_FILE_SAVEAS:
case ID_FILE_PRINT:
MessageBeep(0);
return 0;
case ID_FILE_EXIT:
SendMessage(hwnd, WM_CLOSE, 0, 0);
return 0;
case ID_EIDT_UNDO:
SendMessage(hwndEdit, WM_UNDO, 0, 0);
return 0;
case ID_EIDT_CUT:
SendMessage(hwndEdit, WM_CUT, 0, 0);
return 0;
case ID_EIDT_COPY:
SendMessage(hwndEdit, WM_COPY, 0, 0);
return 0;
case ID_EIDT_PASTE:
SendMessage(hwndEdit, WM_PASTE, 0, 0);
return 0;
case ID_EIDT_DELETE:
SendMessage(hwndEdit, WM_CLEAR, 0, 0);
return 0;
case ID_EIDT_SELETEALL:
SendMessage(hwndEdit, EM_SETSEL, 0, -1);
return 0;
case ID_HELP_HELP:
MessageBox(hwnd, TEXT("Help not yet implemented!"),
szAppName, MB_OK | MB_ICONEXCLAMATION);
return 0;
case ID_HELP_ABOUTPOPPAD2:
MessageBox(hwnd, TEXT("POPPAD2 (c) Charles Petzold, 1998"),
szAppName, MB_OK | MB_ICONINFORMATION);
return 0;
}
break;
case WM_CLOSE:
if (IDYES == AskConfirmation(hwnd))
DestroyWindow(hwnd);
return 0;
case WM_QUERYENDSESSION:
if (IDYES == AskConfirmation(hwnd))
return 1;
else
return 0;
case WM_DESTROY:
PostQuitMessage(0);
return 0;
}
return DefWindowProc(hwnd, message, wParam, lParam);
}
```

## 十一、对话框

### 11.1.模态对话框

创建一个`About1`对话框

`About1.rc`

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
ABOUT1 ICON "About1.ico"
/////////////////////////////////////////////////////////////////////////////
//
// Dialog
//
AboutBox DIALOGEX 32, 32, 211, 127
STYLE DS_SETFONT | DS_MODALFRAME | DS_FIXEDSYS | WS_POPUP | WS_SYSMENU
FONT 8, "MS Shell Dlg", 400, 0, 0x1
BEGIN
DEFPUSHBUTTON "确定",IDOK,79,99,50,14
LTEXT "About1",IDC_STATIC,82,29,42,8
LTEXT "About Derek Demo",IDC_STATIC,70,50,101,15
LTEXT "derek 2022/12/12",IDC_STATIC,69,70,124,13
ICON "About1",IDC_STATIC,17,23,20,20
END
/////////////////////////////////////////////////////////////////////////////
//
// Menu
//
IDR_MENU1 MENU
BEGIN
POPUP "Help"
BEGIN
MENUITEM "About About1...", ID_HELP_ABOUTABOUT1
END
END
/////////////////////////////////////////////////////////////////////////////
//
// DESIGNINFO
//
#ifdef APSTUDIO_INVOKED
GUIDELINES DESIGNINFO
BEGIN
"AboutBox", DIALOG
BEGIN
LEFTMARGIN, 7
RIGHTMARGIN, 204
TOPMARGIN, 7
BOTTOMMARGIN, 120
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
// 供 About1.rc 使用
//
#define IDI_ICON1 101
#define IDR_MENU1 102
#define IDD_DIALOG1 103
#define ID_HELP_ABOUTABOUT1 40001
// Next default values for new objects
//
#ifdef APSTUDIO_INVOKED
#ifndef APSTUDIO_READONLY_SYMBOLS
#define _APS_NEXT_RESOURCE_VALUE 105
#define _APS_NEXT_COMMAND_VALUE 40002
#define _APS_NEXT_CONTROL_VALUE 1001
#define _APS_NEXT_SYMED_VALUE 101
#endif
#endif
```

`About1.c`
