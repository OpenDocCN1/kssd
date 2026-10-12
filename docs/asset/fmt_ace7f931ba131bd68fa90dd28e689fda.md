
```
invoke SetDlgItemText, hWnd, DLG_WINDOW_EDIT_CLASS, addr @szWinBuff3
invoke GetWindowLong, hWinWindow, GWL_STYLE ;获取窗口风格
invoke _Wsprintf, addr @szWinBuff3, sizeof @szWinBuff3, eax
invoke SetDlgItemText, hWnd, DLG_WINDOW_EDIT_SYTLE, addr @szWinBuff3
invoke GetWindowLong, hWinWindow, GWL_ID ;获取窗口 ID
invoke _Wsprintf, addr @szWinBuff3, sizeof @szWinBuff3, eax
invoke SetDlgItemText, hWnd, DLG_WINDOW_EDIT_ID, addr @szWinBuff3
invoke GetWindowLong, hWinWindow, GWL_HINSTANCE ;获取窗口模块
invoke _Wsprintf, addr @szWinBuff3, sizeof @szWinBuff3, eax
invoke SetDlgItemText, hWnd, DLG_WINDOW_EDIT_HINSTANCE, addr @szWinBuff3
invoke GetWindowLong, hWinWindow, GWL_WNDPROC ;获取窗口回调函数
invoke _Wsprintf, addr @szWinBuff3, sizeof @szWinBuff3, eax
invoke SetDlgItemText, hWnd, DLG_EDIT_WNDFUNCTION, addr @szWinBuff3
invoke GetWindowText, hWinWindow, addr @szWinBuff2, sizeof @szWinBuff2
invoke SetDlgItemText, hWnd, DLG_WINDOW_NAME_EDIT, addr @szWinBuff2
.endif
.elseif uMsg==WM_CLOSE ;关闭程序
invoke KillTimer, hWnd, DLG_TIMER_ID
invoke EndDialog, hWnd, FALSE
.else
mov eax, FALSE
ret
.endif
mov eax, TRUE
ret
DlgProc endp

;<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
;关于对话框回调函数
AboutDlgProc proc uses edi esi hWnd, uMsg, wParam, lParam
LOCAL @hFontOut: DWORD
LOCAL @st_hFont: LOGFONT
.if uMsg==WM_INITDIALOG
invoke LoadIcon, hInstance, 100
invoke SendMessage, hWnd, WM_SETICON, ICON_SMALL, eax
mov edi, 1024
.while edi <= 1028 ;子类化超级链接
invoke GetDlgItem, hWnd, edi
mov esi, eax
invoke SetWindowLong, esi, GWL_WNDPROC, offset UrlLinkProc
invoke SetWindowLong, esi, GWL_USERDATA, eax
inc edi
.endw
.elseif uMsg==WM_CTLCOLORSTATIC ;重画超级链接文字颜色及背景颜色
invoke GetDlgCtrlID, lParam
.if (eax>=1024) && (eax<=1028)
invoke SendMessage, lParam, WM_GETFONT, NULL, NULL
mov @hFontOut, eax
invoke GetObject, @hFontOut, sizeof LOGFONT, addr @st_hFont
mov @st_hFont.lfUnderline, NULL
invoke CreateFontIndirect, addr @st_hFont
mov @hFontOut, eax
invoke SelectObject, wParam, eax
invoke SetTextColor, wParam, 0FF0000h
invoke GetSysColor, 15
invoke SetBkColor, wParam, eax
invoke DeleteObject, @hFontOut
invoke GetStockObject, 5
.else
xor eax, eax
ret
.endif
ret
.elseif uMsg==WM_COMMAND
mov eax, wParam
.if ax==ABOUT_OK
invoke EndDialog, hWnd, FALSE
.endif
.elseif uMsg==WM_CLOSE
invoke EndDialog, hWnd, FALSE
.else
xor eax, eax
ret
.endif
mov eax, TRUE
ret
AboutDlgProc endp

;<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
;Label 控件回调函数子类化函数
UrlLinkProc proc hWnd, uMsg, wParam, lParam
.if uMsg==WM_LBUTTONUP
invoke GetDlgCtrlID, hWnd
.if eax==ABOUT_LINK_WWW_PEDIY
mov eax, offset szLinkWwwPediy
.elseif eax==ABOUT_LINK_BBS_PEDIY
mov eax, offset szLinkBbsPediy
.elseif eax==ABOUT_LINK_DFCG
mov eax, offset szLinkDfcg
.elseif eax==ABOUT_LINK_RCT
mov eax, offset szLinkRct
.elseif eax==ABOUT_LINK_CHINADFORCE
mov eax, offset szLinkChinadforce
.endif
invoke ShellExecute, NULL, NULL, eax, NULL, NULL, SW_SHOW
.elseif uMsg==WM_NCHITTEST
mov eax, TRUE
ret
.elseif uMsg==WM_SETCURSOR
invoke SetCursor, hLinkCur
.else
invoke GetWindowLong, hWnd, GWL_USERDATA
invoke CallWindowProc, eax, hWnd, uMsg, wParam, lParam
ret
.endif
xor eax, eax
ret
UrlLinkProc endp
END START
```

4dd466d0e1e0cc123afbe504da2462ca
