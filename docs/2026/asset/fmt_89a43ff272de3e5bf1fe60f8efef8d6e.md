
```c
//{{NO_DEPENDENCIES}}
// Microsoft Visual C++ 生成的包含文件。
// 供 PopPad2.rc 使用
//
#define IDR_MENU1 101
//#define IDR_ACCELERATOR1 102
#define ID_FILE_NEW 40001
#define ID_FILE_OPEN 40002
#define ID_FILE_SAVE 40003
#define ID_FILE_SAVEAS 40004
#define ID_FILE_PRINT 40005
#define ID_FILE_EXIT 40006
#define ID_EIDT_UNDO 40007
#define ID_EIDT_CUT 40008
#define ID_EIDT_COPY 40009
#define ID_EIDT_PASTE 40010
#define ID_Menu 40011
#define ID_EIDT_DELETE 40012
#define ID_EIDT_SELETEALL 40013
#define ID_HELP_HELP 40014
#define ID_HELP_ABOUTPOPPAD2 40015
// Next default values for new objects
//
#ifdef APSTUDIO_INVOKED
#ifndef APSTUDIO_READONLY_SYMBOLS
#define _APS_NEXT_RESOURCE_VALUE 103
#define _APS_NEXT_COMMAND_VALUE 40027
#define _APS_NEXT_CONTROL_VALUE 1001
#define _APS_NEXT_SYMED_VALUE 101
#endif
#endif
```

`PopPad2.rc`

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
// Menu
//
IDR_MENU1 MENU
BEGIN
POPUP "&File"
BEGIN
MENUITEM "&New", ID_FILE_NEW
MENUITEM "&Open", ID_FILE_OPEN
MENUITEM "&Save", ID_FILE_SAVE
MENUITEM "Save &As...", ID_FILE_SAVEAS
MENUITEM SEPARATOR
MENUITEM "&Print", ID_FILE_PRINT
MENUITEM SEPARATOR
MENUITEM "E&xit", ID_FILE_EXIT
END
POPUP "&Eidt"
BEGIN
MENUITEM "&Undo\tCtrl+Z", ID_EIDT_UNDO
MENUITEM SEPARATOR
MENUITEM "Cu&t\tCtrl+X", ID_EIDT_CUT
MENUITEM "&Copy\tCtrl+C", ID_EIDT_COPY
MENUITEM "&Paste\tCtrl+V", ID_EIDT_PASTE
MENUITEM "De&lete\tCtrl+Del", ID_EIDT_DELETE
MENUITEM SEPARATOR
MENUITEM "&Selete All", ID_EIDT_SELETEALL
END
POPUP "&Help"
BEGIN
MENUITEM "&Help", ID_HELP_HELP
MENUITEM "&About PopPad2...", ID_HELP_ABOUTPOPPAD2
END
END
/////////////////////////////////////////////////////////////////////////////
//
// Accelerator
//
POPPAD2 ACCELERATORS
BEGIN
VK_BACK, ID_EIDT_UNDO, VIRTKEY, ALT, NOINVERT
VK_DELETE, ID_EIDT_DELETE, VIRTKEY, NOINVERT
VK_DELETE, ID_EIDT_CUT, VIRTKEY, SHIFT, NOINVERT
VK_F1, ID_HELP_HELP, VIRTKEY, NOINVERT
VK_INSERT, ID_EIDT_COPY, VIRTKEY, CONTROL, NOINVERT
VK_INSERT, ID_EIDT_PASTE, VIRTKEY, SHIFT, NOINVERT
"^C", ID_EIDT_COPY, ASCII, NOINVERT
"^V", ID_EIDT_PASTE, ASCII, NOINVERT
"^X", ID_EIDT_CUT, ASCII, NOINVERT
"^Z", ID_EIDT_UNDO, ASCII, NOINVERT
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

`PopPad2.c`
