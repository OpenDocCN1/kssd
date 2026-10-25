
# MiniSPY V1.0 (小小间谍)

> 原文：[`www.pediy.com/kssd/pediy04/kanxue472.htm`](https://www.pediy.com/kssd/pediy04/kanxue472.htm)
>
> 这里调用的一些 `API` 函数可能对大家有点启发

```asm
;****************************************************************
;*      Author: 一块三毛钱
;*      E_mail: crazy_soft@163.net
;*   Homepage: zhongts.yeah.net
;****************************************************************
;*      Compile: Hutch's Masm32
;* Create Time: 2001.10.20
;*      Modify: 2001.10.20
;* Description: MiniSPY V1.0 (小小间谍)
;*               捕捉桌面上的窗口（或子窗口）各窗口的属性，改
;*               变各窗口的状态
;****************************************************************

.386
.model flat,stdcall
option casemap:none

;****************************************************************

include     d:\masm32\include\windows.inc
include     d:\masm32\include\user32.inc
include     d:\masm32\include\kernel32.inc
include     d:\masm32\include\gdi32.inc
include     d:\masm32\include\winmm.inc

includelib  d:\masm32\lib\user32.lib
includelib  d:\masm32\lib\kernel32.lib
includelib  d:\masm32\lib\gdi32.lib
includelib  d:\masm32\lib\winmm.lib

;****************************************************************

DlgProc  proto :DWORD,:DWORD,:DWORD,:DWORD
SHWnd  proto :DWORD
DrawFocus  proto :DWORD
GetP   proto :DWORD,:DWORD

;****************************************************************

.data
ClassName    db "MiniSPYClass",0
AppName      db "MiniSPY",0
DlgName      db "DLG_MAIN",0        ;资源文件中所定义的对话框模板名称
szCaptureS   db "显示 &>>",0        ;对话框 收拢/展开 时对应的按钮文本
szCaptureH   db "<&< 隐藏",0
szCombo1     db "最大化",0          ;组合框中的项目
szCombo2     db "最小化",0
szCombo3     db "最顶层",0
szCombo4     db "失效",0
szCombo5     db "恢复",0
FORMat1      db "%08X",0
FORMat2      db "(%d,%d)(%d,%d) %d×%d",0
szSysMenu    db "关于(&A)...",0
szAbout      db "编程：一块三毛钱",13,10
             db "主页：http://zhongts.yeah.net",13,10
     db "信箱：crazy_soft@163.net",13,10,13,10
     db "欢迎各位来我的主页溜达溜达",0
ws_Style     dd WS_BORDER,          ;窗口风格常数
WS_CAPTION,
WS_CHILD,
WS_CHILDWINDOW,
WS_CLIPCHILDREN,
WS_CLIPSIBLINGS,
WS_DISABLED,
WS_DLGFRAME,
WS_GROUP,
WS_HSCROLL,
WS_ICONIC,
WS_MAXIMIZE,
WS_MAXIMIZEBOX,
WS_MINIMIZE,
WS_MINIMIZEBOX,
WS_OVERLAPPED,
WS_OVERLAPPEDWINDOW,
WS_POPUP,
WS_POPUPWINDOW,
WS_SIZEBOX,
WS_SYSMENU,
WS_TABSTOP,
WS_THICKFRAME,
WS_TILED,
WS_TILEDWINDOW,
WS_VISIBLE,
WS_VSCROLL
szws_Style   db "WS_BORDER          ",0
     db "WS_CAPTION           ",0
     db "WS_CHILD             ",0
     db "WS_CHILDWINDOW       ",0
     db "WS_CLIPCHILDREN      ",0
     db "WS_CLIPSIBLINGS      ",0
     db "WS_DISABLED          ",0
     db "WS_DLGFRAME          ",0
     db "WS_GROUP             ",0
     db "WS_HSCROLL           ",0
     db "WS_ICONIC            ",0
     db "WS_MAXIMIZE          ",0
     db "WS_MAXIMIZEBOX       ",0
     db "WS_MINIMIZE          ",0
     db "WS_MINIMIZEBOX       ",0
     db "WS_OVERLAPPED        ",0
     db "WS_OVERLAPPEDWINDOW",0
     db "WS_POPUP             ",0
     db "WS_POPUPWINDOW       ",0
     db "WS_SIZEBOX           ",0
     db "WS_SYSMENU           ",0
     db "WS_TABSTOP           ",0
     db "WS_THICKFRAME        ",0
     db "WS_TILED             ",0
     db "WS_TILEDWINDOW       ",0
     db "WS_VISIBLE           ",0
     db "WS_VSCROLL           ",0
ws_ExStyle   dd WS_EX_ACCEPTFILES,   ;窗口扩展风格常数
WS_EX_APPWINDOW,
WS_EX_CLIENTEDGE,
WS_EX_CONTEXTHELP,
WS_EX_CONTROLPARENT,
WS_EX_DLGMODALFRAME,
WS_EX_LEFT,
WS_EX_LEFTSCROLLBAR,
WS_EX_LTRREADING,
WS_EX_MDICHILD,
WS_EX_NOPARENTNOTIFY,
WS_EX_OVERLAPPEDWINDOW,
WS_EX_PALETTEWINDOW,
WS_EX_RIGHT,
WS_EX_RIGHTSCROLLBAR,
WS_EX_RTLREADING,
WS_EX_STATICEDGE,
WS_EX_TOOLWINDOW,
WS_EX_TOPMOST,
WS_EX_TRANSPARENT,
WS_EX_WINDOWEDGE
szws_ExStyle db "WS_EX_ACCEPTFILES     ",0
     db "WS_EX_APPWINDOW        ",0
     db "WS_EX_CLIENTEDGE       ",0
     db "WS_EX_CONTEXTHELP      ",0
     db "WS_EX_CONTROLPARENT    ",0
     db "WS_EX_DLGMODALFRAME    ",0
     db "WS_EX_LEFT             ",0
     db "WS_EX_LEFTSCROLLBAR    ",0
     db "WS_EX_LTRREADING       ",0
     db "WS_EX_MDICHILD         ",0
     db "WS_EX_NOPARENTNOTIFY   ",0
     db "WS_EX_OVERLAPPEDWINDOW",0
     db "WS_EX_PALETTEWINDOW    ",0
     db "WS_EX_RIGHT            ",0
     db "WS_EX_RIGHTSCROLLBAR   ",0
     db "WS_EX_RTLREADING       ",0
     db "WS_EX_STATICEDGE       ",0
     db "WS_EX_TOOLWINDOW       ",0
     db "WS_EX_TOPMOST          ",0
     db "WS_EX_TRANSPARENT      ",0
     db "WS_EX_WINDOWEDGE       ",0

.data?
hInstance     HINSTANCE ?
hSysMenu      dd    ?               ;系统菜单的句柄
hBtnShowHide  dd    ?   ;"显示/隐藏"按钮的句柄
hBtnCatch     dd    ?               ;图象框的句柄
hBtnIcon      dd    ?               ;图象框 2 的句柄
hCombo        dd    ?               ;组合框的句柄
hListStyle    dd    ?               ;窗口风格列表框的句柄
hListExtStyle dd    ?               ;窗口扩展风格列表框的句柄
hWndOver      dd    ?               ;捕捉时当前控件（或窗口）的句柄
hWndPre       dd    ?               ;捕捉时前一控件（或窗口）的句柄
bSHWnd        BOOL  ?               ;是否展开窗口的标志
bMouseDown    BOOL  ?               ;鼠标是否按下
bHideWnd      BOOL  ?               ;捕捉时是否隐藏主窗口
bFillFocus    BOOL  ?               ;是否以块方式捕捉
rectFull      RECT  <?>             ;对话框展开后的大小
rectHalf      RECT  <?>             ;对话框收拢后的大小
rectSH        RECT  <?>             ;"显示/隐藏"按钮的大小
WndStyle      dd    ?               ;窗口风格

.const
IDI_ICON_MAIN            equ      1000
IDI_ICON_CATCH1          equ      1001
IDI_ICON_CATCH2          equ      1002

IDC_CURSOR_CAPTURE       equ      1003

IDS_SOUND                equ      1004

IDC_EDIT_HANDLE          equ      2010
IDC_EDIT_CAPTION         equ      2011
IDC_EDIT_CLASS           equ      2012
IDC_EDIT_RECT            equ      2013
IDC_LIST_STYLE           equ      2014
IDC_LIST_EXTSTYLE        equ      2015
IDC_PICTURE_CATCH        equ      2016
IDC_PICTURE_ICON         equ      2017
IDC_COMBO_SEND           equ      2018
IDC_BUTTON_APPLY         equ      2019
IDC_BUTTON_FLASH         equ      2020
IDC_BUTTON_SHOWHIDE      equ      2021
IDC_CHECK_HIDEWINDOW     equ      2022
IDC_CHECK_BLOCK          equ      2023

IDM_ABOUT                equ      3000
```



# 文档说明

```
;************
start:
invoke GetModuleHandle, NULL
mov hInstance, eax
invoke DialogBoxParam, hInstance, addr DlgName, NULL, addr DlgProc, 0
invoke ExitProcess, eax

;对话框窗口过程****************************
DlgProc proc hWnd:HWND, uMsg:UINT, wParam:WPARAM, lParam:LPARAM
local rectCatch:RECT
local pt:POINT

.if uMsg==WM_INITDIALOG
invoke GetWindowRect, hWnd, addr rectFull ;取得窗口大小
invoke GetDlgItem, hWnd, IDC_BUTTON_SHOWHIDE ;取得"显示/隐藏"按钮的句柄
mov hBtnShowHide, eax ;句柄 => hBtnShowHide
invoke GetWindowRect, hBtnShowHide, addr rectSH ;取得"显示/隐藏"按钮的大小

push rectFull.left ;为 rectHalf 赋值
push rectFull.top
push rectFull.right
push rectSH.bottom
pop rectHalf.bottom
pop rectHalf.right
pop rectHalf.top
pop rectHalf.left
add rectHalf.bottom, 12

mov bSHWnd, FALSE ;窗口第一次显示时呈收拢状态
invoke SHWnd, hWnd ;调用"展开/收拢窗口"子函数
invoke LoadIcon, hInstance, IDI_ICON_MAIN ;加载图标
invoke SendMessage, hWnd, WM_SETICON, ICON_SMALL, eax ;给对话框窗口设置图标

invoke GetSystemMenu, hWnd, FALSE ;取得系统菜单的句柄
mov hSysMenu, eax ;句柄 => hSysMenu
invoke AppendMenu, hSysMenu, MF_SEPARATOR, NULL, NULL ;插入分隔线
invoke AppendMenu, hSysMenu, MF_STRING, IDM_ABOUT, addr szSysMenu ;插入"关于..."菜单

invoke GetDlgItem, hWnd, IDC_CHECK_HIDEWINDOW
invoke SendMessage, eax, BM_SETCHECK, BST_CHECKED, 0 ;捕捉时隐藏主窗口
mov bHideWnd, TRUE
invoke GetDlgItem, hWnd, IDC_CHECK_BLOCK
invoke SendMessage, eax, BM_SETCHECK, BST_UNCHECKED, 0 ;以边框方式捕捉
mov bFillFocus, FALSE

invoke GetDlgItem, hWnd, IDC_PICTURE_ICON ;取得图象框 2 的句柄
mov hBtnIcon, eax ;句柄 => hBtnIcon
invoke GetDlgItem, hWnd, IDC_PICTURE_CATCH ;取得图象框的句柄
mov hBtnCatch, eax ;句柄 => hBtnCatch

invoke GetDlgItem, hWnd, IDC_COMBO_SEND ;取得组合框的句柄
mov hCombo, eax ;句柄 => hCombo
invoke SendMessage, hCombo, CB_ADDSTRING, 0, addr szCombo1 ;初始化组合框
invoke SendMessage, hCombo, CB_ADDSTRING, 0, addr szCombo2
invoke SendMessage, hCombo, CB_ADDSTRING, 0, addr szCombo3
invoke SendMessage, hCombo, CB_ADDSTRING, 0, addr szCombo4
invoke SendMessage, hCombo, CB_ADDSTRING, 0, addr szCombo5

invoke GetDlgItem, hWnd, IDC_LIST_STYLE ;取得窗口风格列表框的句柄
mov hListStyle, eax ;句柄 => hListStyle
invoke GetDlgItem, hWnd, IDC_LIST_EXTSTYLE ;取得窗口扩展风格列表框的句柄
mov hListExtStyle, eax ;句柄 => hListExtStyle

mov bMouseDown, FALSE ;鼠标未按下

.elseif uMsg==WM_LBUTTONDOWN
invoke GetCursorPos, addr pt ;取得鼠标位置
invoke GetWindowRect, hBtnCatch, addr rectCatch ;取得图象框的大小
invoke PtInRect, addr rectCatch, pt.x, pt.y ;判断鼠标是否在图象框内
.if eax ;如果在
.if bHideWnd ;是否需要隐藏主窗口
invoke ShowWindow, hWnd, SW_HIDE
.endif
mov hWndOver, NULL
mov hWndPre, NULL
invoke SetCapture, hWnd ;锁定鼠标
invoke LoadCursor, hInstance, IDC_CURSOR_CAPTURE ;加载光标
invoke SetCursor, eax ;设置鼠标形状

invoke LoadIcon, hInstance, IDI_ICON_CATCH1 ;加载图标
invoke SendMessage, hBtnCatch, STM_SETICON, eax, 0 ;替换图象框中的图标

mov bMouseDown, TRUE ;鼠标已按下
.endif

.elseif uMsg==WM_MOUSEMOVE
.if bMouseDown ;如果鼠标已按下
invoke GetCursorPos, addr pt ;取得鼠标位置
invoke WindowFromPoint, pt.x, pt.y ;取得鼠标位置处的窗口句柄
mov hWndOver, eax ;句柄 => hWndOver
.if eax != hWndPre && eax != hBtnCatch ;如果窗口改变了并且不是图象框时
invoke GetP, hWnd, hWndOver ;取得鼠标位置处的窗口的属性
invoke DrawFocus, hWndPre ;绘制焦点矩形
invoke DrawFocus, hWndOver
push hWndOver ;hWndOver => hWndPre
pop hWndPre
.endif
.endif

.elseif uMsg==WM_LBUTTONUP
.if bMouseDown ;如果鼠标已按下
invoke ReleaseCapture ;释放鼠标
invoke PlaySound, IDS_SOUND, hInstance, SND_RESOURCE or SND_ASYNC ;播放音效
invoke LoadCursor, NULL, IDC_ARROW ;恢复鼠标形状
invoke SetCursor, eax

invoke LoadIcon, hInstance, IDI_ICON_CATCH2 ;加载图标
invoke SendMessage, hBtnCatch, STM_SETICON, eax, 0 ;替换图象框中的图标

invoke DrawFocus, hWndOver
.if bHideWnd ;如果主窗口已被隐藏，则显示主窗口
invoke ShowWindow, hWnd, SW_SHOW
.endif

invoke SendMessage, hWndOver, WM_GETICON, ICON_BIG, 1 ;取得图标
invoke SendMessage, hBtnIcon, STM_SETICON, eax, 0 ;把图标显示在图象框 2 内

mov bMouseDown, FALSE ;鼠标未按下
.endif

.elseif uMsg==WM_COMMAND
mov eax, wParam
and eax, 0ffffh
.if eax==IDC_BUTTON_SHOWHIDE ;"显示/隐藏"按钮
invoke SHWnd, hWnd
.elseif eax==IDC_BUTTON_APPLY ;"应用"按钮
invoke SendMessage, hCombo, CB_GETCURSEL, 0, 0
.if eax!=CB_ERR
;根据组合框中被选中项的不同，分别设置状态
.if eax==0
invoke ShowWindow, hWndOver, SW_MAXIMIZE
.elseif eax==1
invoke ShowWindow, hWndOver, SW_MINIMIZE
.elseif eax==2
invoke SetWindowPos, hWndOver, HWND_TOPMOST, 0, 0, 0, 0, SWP_NOSIZE or SWP_NOMOVE
.elseif eax==3
invoke EnableWindow, hWndOver, FALSE
.elseif eax==4
invoke ShowWindow, hWndOver, SW_RESTORE
invoke EnableWindow, hWndOver, TRUE
.endif
.endif
.elseif eax==IDC_BUTTON_FLASH ;"闪烁"按钮
.if hWndOver
push bFillFocus
mov bFillFocus, TRUE
invoke DrawFocus, hWndOver ;反色重画 4 次实现闪烁效果
invoke Sleep, 100
invoke DrawFocus, hWndOver
invoke Sleep, 100
invoke DrawFocus, hWndOver
invoke Sleep, 100
invoke DrawFocus, hWndOver
invoke Sleep, 100
invoke DrawFocus, hWndOver
invoke Sleep, 100
invoke DrawFocus, hWndOver
invoke Sleep, 100
invoke DrawFocus, hWndOver
invoke Sleep, 100
invoke DrawFocus, hWndOver
pop bFillFocus
.endif

.elseif eax==IDC_CHECK_HIDEWINDOW ;捕捉时隐藏主窗口
invoke SendMessage, lParam, BM_GETCHECK, 0, 0
.if eax
mov bHideWnd, TRUE
.else
mov bHideWnd, FALSE
.endif
.elseif eax==IDC_CHECK_BLOCK ;块方式捕捉
invoke SendMessage, lParam, BM_GETCHECK, 0, 0
.if eax
mov bFillFocus, TRUE
.else
mov bFillFocus, FALSE
.endif
.endif

.elseif uMsg==WM_SYSCOMMAND
mov eax, wParam
movzx eax, ax
.if eax==IDM_ABOUT ;系统菜单中的"关于..."菜单
invoke MessageBox, hWnd, addr szAbout, addr AppName, MB_OK+MB_ICONINFORMATION
.else
invoke DefWindowProc, hWnd, uMsg, wParam, lParam
.endif

.elseif uMsg==WM_CLOSE
invoke EndDialog, hWnd, NULL
.else
mov eax, FALSE
ret
.endif
mov eax, TRUE
ret
DlgProc endp

;展开/收拢窗口子函数****************************
SHWnd proc hWnd:HWND
```



```
local  rect:RECT

.if      bSHWnd  ;展开窗口
invoke  SetWindowText, hBtnShowHide, addr szCaptureH  ;设置按钮文本为"<&< 隐藏"
mov     bSHWnd, FALSE

push    rectFull.left
pop     rect.left
push    rectFull.right
pop     rect.right
push    rectFull.top
pop     rect.top
push    rectFull.bottom
pop     rect.bottom
.else   ;收拢窗口
invoke  SetWindowText, hBtnShowHide, addr szCaptureS  ;设置按钮文本为"显示 &>>"
mov     bSHWnd, TRUE

push    rectHalf.left
pop     rect.left
push    rectHalf.right
pop     rect.right
push    rectHalf.top
pop     rect.top
push    rectHalf.bottom
pop     rect.bottom
.endif

mov     eax, rect.right                            ;计算窗口的宽度
sub     eax, rect.left
mov     ebx, eax
mov     eax, rect.bottom                             ;计算窗口的高度
sub     eax, rect.top
invoke  SetWindowPos, hWnd, NULL, 0, 0, ebx, eax, SWP_NOZORDER or SWP_NOMOVE  ;重置窗口大小

ret
SHWnd endp

;绘制焦点矩形子函数************************************************************
DrawFocus proc hWnd:HWND
local  hdc:HDC
local  rect:RECT

.if      hWnd
invoke  GetDC, NULL                               ;取得 dc
mov     hdc, eax                               ;dc => hdc
invoke  GetWindowRect, hWnd, addr rect           ;取得窗口大小
invoke  SetROP2, hdc, R2_NOT                     ;绘图方式为反色

.if      bFillFocus  ;块方式捕捉
invoke  InvertRect, hdc, addr rect        ;将矩形区域反色
.else
invoke  MoveToEx, hdc, rect.left, rect.top, NULL       ;根据窗口大小绘制矩形
invoke  LineTo, hdc, rect.right, rect.top
invoke  MoveToEx, hdc, rect.right, rect.top, NULL
invoke  LineTo, hdc, rect.right, rect.bottom
invoke  MoveToEx, hdc, rect.right, rect.bottom, NULL
invoke  LineTo, hdc, rect.left, rect.bottom
invoke  MoveToEx, hdc, rect.left, rect.bottom, NULL
invoke  LineTo, hdc, rect.left, rect.top

invoke  OffsetRect, addr rect, -1, -1             ;根据窗口位置向左，向上分别移动一个象素
invoke  MoveToEx, hdc, rect.left, rect.top, NULL       ;再绘制一个矩形
invoke  LineTo, hdc, rect.right, rect.top
invoke  MoveToEx, hdc, rect.right, rect.top, NULL
invoke  LineTo, hdc, rect.right, rect.bottom
invoke  MoveToEx, hdc, rect.right, rect.bottom, NULL
invoke  LineTo, hdc, rect.left, rect.bottom
invoke  MoveToEx, hdc, rect.left, rect.bottom, NULL
invoke  LineTo, hdc, rect.left, rect.top
.endif
.endif

ret
DrawFocus endp

;取得窗口属性子函数************************************************************
GetP  proc  hWndOwner:HWND, hWnd:HWND
local  rect:RECT
local  buffer[128]:byte

invoke  wsprintf, addr buffer, addr FORMat1, hWnd          ;取得句柄
invoke  SetDlgItemText, hWndOwner, IDC_EDIT_HANDLE, addr buffer

invoke  GetWindowText, hWnd, addr buffer, 128               ;取得标题
invoke  SetDlgItemText, hWndOwner, IDC_EDIT_CAPTION, addr buffer

invoke  GetClassName, hWnd, addr buffer, 128                ;取得类名
invoke  SetDlgItemText, hWndOwner, IDC_EDIT_CLASS, addr buffer

invoke  GetWindowRect, hWnd, addr rect                      ;取得区域大小
mov     eax, rect.right
sub     eax, rect.left
mov     ebx, eax
mov     eax, rect.bottom
sub     eax, rect.top
invoke  wsprintf, addr buffer, addr FORMat2, rect.left, rect.top, rect.right, rect.bottom, ebx, eax
invoke  SetDlgItemText, hWndOwner, IDC_EDIT_RECT, addr buffer

invoke  GetWindowLong, hWnd, GWL_STYLE                       ;取得窗口风格
mov     WndStyle, eax                                   ;风格 => WndStyle
invoke  SendMessage, hListStyle, LB_RESETCONTENT, 0, 0      ;清空列表框
mov     cx, 1bh                ;设置循环次数
mov     bx, 0                  ;bx=0
mov     si, 0                  ;si=0
loop1:
mov     eax, ws_Style[bx]        ;取窗口风格常数
and     eax, WndStyle           ;将它同 WndStyle 与，结果放在 eax 中
cmp     eax, ws_Style[bx]        ;再同原来取出的常数比较
jne     break1                 ;如果不等，则说明没有该风格，跳到下一个继续比较
invoke  SendMessage, hListStyle, LB_ADDSTRING, 0, addr szws_Style[si]  ;填充列表框
break1:
add     bx, 4                  ;修改 bx，使它指向下一个常数
add     si, 20                 ;修改 si，使它指向下一个字符串
loop    loop1

invoke  GetWindowLong, hWnd, GWL_EXSTYLE                      ;取得窗口扩展风格
mov     WndStyle, eax
invoke  SendMessage, hListExtStyle, LB_RESETCONTENT, 0, 0
mov     cx, 15h
mov     bx, 0
mov     si, 0
loop2:
mov     eax, ws_ExStyle[bx]
and     eax, WndStyle
cmp     eax, ws_ExStyle[bx]
jne     break2
invoke  SendMessage, hListExtStyle, LB_ADDSTRING, 0, addr szws_ExStyle[si]
break2:
add     bx, 4
add     si, 23
loop    loop2

ret
GetP  endp

end start
```
`4dd466d0e1e0cc123afbe504da2462ca`
