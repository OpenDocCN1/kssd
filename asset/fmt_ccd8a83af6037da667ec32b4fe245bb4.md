
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
