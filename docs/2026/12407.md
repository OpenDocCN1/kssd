
# 举例实现对Win9x注册表内的键值实现打开、创建、设置、读取...

```asm
;首先感谢AoGo论坛上的AoGo、Fox、aboil三位兄弟的帮助，在此一并谢过。
;本例实现对Win9x注册表内的键值实现打开、创建、设置、读取、删除操作
;通过对"HKEY_CURRENT_USERSoftwareMicrosoftWindowsCurrentVersionAppletsHearts"的设置实现
;红心大战中按 Ctrl+Alt+Shift+F12 显示所有人的牌
;通过对"HKEY_CURRENT_USERSoftwareMicrosoftWindowsCurrentVersionPoliciesExplorer"的设置
;实现隐藏"开始菜单"内的"收藏夹、文档、注销"选项

      .386
      .model flat, stdcall
      option casemap :none

include      windows.inc
include      user32.inc
includelib    user32.lib
include      kernel32.inc
includelib    kernel32.lib
include      Advapi32.inc
includelib    Advapi32.lib

DLG_MAIN    equ  1
IDC_Favorites    equ  1001
IDC_Recent    equ  1002
IDC_LogOff    equ  1003
IDC_Hearts    equ  1004
ICO_MAIN    equ  1000h

      .data?
hInstance    dd  ?
LpValue      dd  ?
LpType      dd  ?
DecValue    dd  ?

      .const
SzHearts    db  'SoftwareMicrosoftWindowsCurrentVersionAppletsHearts',0
SzFavoritesMenu    db  'SoftwareMicrosoftWindowsCurrentVersionPoliciesExplorer',0
SzZbName    db  'zb',0
SzFavoritesMenuName  db  'NoFavoritesMenu',0
SzNoRecentDocsMenu  db  'NoRecentDocsMenu',0
SzNoLogOff    db  'NoLogOff',0
SzHeartsValue    db  '42',0
SzFavoritesMenuValue  db  '01',0
SzNoRecentDocsValue  db  '01',0
SzNoLogOffValue    db  '01',0
SzRegSz      db  'REG_SZ',0
SzRegDword    db  'REG_DWORD',0

      .code
_ProcDlgMain  proc  uses ebx edi esi hWnd,wMsg,wParam,lParam
    local  @hKey,@SzValue[256]:byte

    mov  eax,wMsg
    .if  eax == WM_CLOSE
      invoke  EndDialog,hWnd,NULL
    .elseif  eax == WM_INITDIALOG
      invoke  LoadIcon,hInstance,ICO_MAIN
      invoke  SendMessage,hWnd,WM_SETICON,ICON_BIG,eax
      invoke  RegOpenKeyEx,HKEY_CURRENT_USER,addr SzHearts,NULL,KEY_QUERY_VALUE,addr @hKey
      .if  eax ==  ERROR_SUCCESS
        mov  LpValue,4
        invoke  RegQueryValueEx,@hKey,addr SzZbName,NULL,addr SzRegSz,addr @SzValue,addr LpValue
        .if  eax == ERROR_SUCCESS
          invoke  lstrcmp,addr @SzValue,addr SzHeartsValue
          .if  eax == ERROR_SUCCESS
            invoke  CheckDlgButton,hWnd,IDC_Hearts,BST_CHECKED
          .endif
        .endif
      .endif
      invoke  RegOpenKeyEx,HKEY_CURRENT_USER,addr SzFavoritesMenu,NULL,KEY_QUERY_VALUE,addr @hKey
      .if  eax ==  ERROR_SUCCESS
        mov  LpValue,4
        invoke  RegQueryValueEx,@hKey,addr SzFavoritesMenuName,NULL,addr LpType,addr DecValue,addr LpValue
        .if  eax == ERROR_SUCCESS
          invoke  lstrcmp,addr DecValue,addr SzFavoritesMenuValue
          .if  eax == ERROR_SUCCESS
            invoke  CheckDlgButton,hWnd,IDC_Favorites,BST_CHECKED
          .endif
        .endif
      .endif
      invoke  RegOpenKeyEx,HKEY_CURRENT_USER,addr SzFavoritesMenu,NULL,KEY_QUERY_VALUE,addr @hKey
      .if  eax ==  ERROR_SUCCESS
        mov  LpValue,4
        invoke  RegQueryValueEx,@hKey,addr SzNoRecentDocsMenu,NULL,addr LpType,addr DecValue,addr LpValue
        .if  eax == ERROR_SUCCESS
          invoke  lstrcmp,addr DecValue,addr SzNoRecentDocsValue
          .if  eax == ERROR_SUCCESS
            invoke  CheckDlgButton,hWnd,IDC_Recent,BST_CHECKED
          .endif
        .endif
      .endif
      invoke  RegOpenKeyEx,HKEY_CURRENT_USER,addr SzFavoritesMenu,NULL,KEY_QUERY_VALUE,addr @hKey
      .if  eax ==  ERROR_SUCCESS
        mov  LpValue,4
        invoke  RegQueryValueEx,@hKey,addr SzNoLogOff,NULL,addr LpType,addr DecValue,addr LpValue
        .if  eax == ERROR_SUCCESS
          invoke  lstrcmp,addr DecValue,addr SzNoLogOffValue
          .if  eax == ERROR_SUCCESS
            invoke  CheckDlgButton,hWnd,IDC_LogOff,BST_CHECKED
          .endif
        .endif
      .endif
    .elseif  eax == WM_COMMAND
      mov  eax,wParam
      .if  ax == IDCANCEL
          invoke  EndDialog,hWnd,NULL
      .elseif  ax == IDOK
        invoke  IsDlgButtonChecked,hWnd,IDC_Hearts
        .if  eax == BST_CHECKED
          invoke  RegCreateKey,HKEY_CURRENT_USER,addr SzHearts,addr @hKey
          .if  eax ==  ERROR_SUCCESS
            invoke  lstrlen,addr SzHeartsValue
            inc  eax
            invoke  RegSetValueEx,@hKey,addr SzZbName,NULL,
              REG_SZ, addr SzHeartsValue,eax
            invoke  RegCloseKey,@hKey
          .endif
        .else
          invoke  RegOpenKeyEx,HKEY_CURRENT_USER,addr SzHearts,NULL,KEY_WRITE,addr @hKey
          .if  eax == ERROR_SUCCESS
            invoke  RegDeleteValue,@hKey,addr SzZbName
            invoke  RegCloseKey,@hKey
          .endif
        .endif
        invoke  IsDlgButtonChecked,hWnd,IDC_Favorites
        .if  eax == BST_CHECKED
          invoke  RegCreateKey,HKEY_CURRENT_USER,addr SzFavoritesMenu,addr @hKey
          .if  eax==  ERROR_SUCCESS
            invoke  RegSetValueEx,@hKey,addr SzFavoritesMenuName,NULL,
              REG_DWORD, addr SzFavoritesMenuValue,4
            invoke  RegCloseKey,@hKey
          .endif
        .else
          invoke  RegOpenKeyEx,HKEY_CURRENT_USER,addr SzFavoritesMenu,NULL,KEY_WRITE,addr @hKey
          .if  eax == ERROR_SUCCESS
            invoke  RegDeleteValue,@hKey,addr SzFavoritesMenuName
            invoke  RegCloseKey,@hKey
          .endif
        .endif
        invoke  IsDlgButtonChecked,hWnd,IDC_Recent
        .if  eax == BST_CHECKED
          invoke  RegCreateKey,HKEY_CURRENT_USER,addr SzFavoritesMenu,addr @hKey
          .if  eax==  ERROR_SUCCESS
            invoke  RegSetValueEx,@hKey,addr SzNoRecentDocsMenu,NULL,
              REG_DWORD, addr SzNoRecentDocsValue,4
            invoke  RegCloseKey,@hKey
          .endif
        .else
          invoke  RegOpenKeyEx,HKEY_CURRENT_USER,addr SzFavoritesMenu,NULL,KEY_WRITE,addr @hKey
          .if  eax == ERROR_SUCCESS
            invoke  RegDeleteValue,@hKey,addr SzNoRecentDocsMenu
            invoke  RegCloseKey,@hKey
          .endif
        .endif
        invoke  IsDlgButtonChecked,hWnd,IDC_LogOff
        .if  eax == BST_CHECKED
          invoke  RegCreateKey,HKEY_CURRENT_USER,addr SzFavoritesMenu,addr @hKey
          .if  eax==  ERROR_SUCCESS
            invoke  RegSetValueEx,@hKey,addr SzNoLogOff,NULL,
              REG_DWORD, addr SzNoLogOffValue,4
            invoke  RegCloseKey,@hKey
          .endif
        .else
          invoke  RegOpenKeyEx,HKEY_CURRENT_USER,addr SzFavoritesMenu,NULL,KEY_WRITE,addr @hKey
          .if  eax == ERROR_SUCCESS
            invoke  RegDeleteValue,@hKey,addr SzNoLogOff
            invoke  RegCloseKey,@hKey
          .endif
        .endif
      .endif
    .else
      mov  eax,FALSE
      ret
    .endif
    mov  eax,TRUE
    ret
_ProcDlgMain  endp

start:
    invoke  GetModuleHandle,NULL
    mov  hInstance,eax
    invoke  DialogBoxParam,hInstance,DLG_MAIN,NULL,offset _ProcDlgMain,NULL
    invoke  ExitProcess,NULL
    end  start
```

资源文件：

```cpp
#include <resource.h>
#define    DLG_MAIN  1
#define    IDC_Favorites  1001
#define    IDC_Recent  1002
#define    IDC_LogOff  1003
#define    IDC_Hearts  1004
#define    ICO_MAIN  0x1000

ICO_MAIN  ICON    "Main.ico"

DLG_MAIN DIALOGEX 0, 0, 170, 100
STYLE DS_MODALFRAME | WS_POPUP | WS_CAPTION | WS_SYSMENU
CAPTION "注册表优化 For Windows9X"
FONT 9, "宋体"
{
    DEFPUSHBUTTON   "设置",IDOK,114,48,39,14,0,WS_EX_STATICEDGE
    PUSHBUTTON      "退出",IDCANCEL,114,66,39,14,0,WS_EX_STATICEDGE
    GROUPBOX        "开始菜单不显示的项目",IDC_STATIC,7,7,100,73
    CHECKBOX      "收藏夹",IDC_Favorites,16,20,78,10,BS_AUTOCHECKBOX | WS_TABSTOP
    CHECKBOX        "文档",IDC_Recent,16,34,78,10,BS_AUTOCHECKBOX | WS_TABSTOP
    CHECKBOX        "注销",IDC_LogOff,16,48,78,10,BS_AUTOCHECKBOX | WS_TABSTOP
    CHECKBOX        "红心大战中看所有人的牌",IDC_Hearts,16,80,78,10,BS_AUTOCHECKBOX | WS_TABSTOP
}
```

```text
;本程序在Win98Se+Masm32V8 下编译通过
;            呱叽呱叽/lovemelovemydogs[BCG][YCG]
;              2003.10.10
```
