
```assembly
.386
.model flat,stdcall
option casemap:none
include windows.inc
include user32.inc
include kernel32.inc
include rasapi32.inc
include shell32.inc
includelib user32.lib
includelib kernel32.lib
includelib shell32.lib
includelib rasapi32.lib

IdPlay        equ  2000
WM_SHELLNOTIFY equ  WM_USER+5
IDI_TRAY      equ  0
IDM_RESTORE   equ  1000
IDM_EXIT      equ  1010

WinMain PROTO :DWORD,:DWORD,:DWORD,:DWORD
TimProc PROTO :HWND,:UINT,:UINT,:DWORD

.data
RAS_STATS   struc
dwSize             dd ?
dwBytesXmited      dd ?
dwBytesRcved       dd ?
dwFramesXmited     dd ?
dwFramesRcved      dd ?
dwCrcErr           dd ?
dwTimeoutErr       dd ?
dwAlignmentErr     dd ?
dwHardwareOverrunErr dd ?
dwFramingErr       dd ?
dwBufferOverrunErr dd ?
dwCompressionRatioIn dd ?
dwCompressionRatioOut dd ?
dwBps              dd ?
dwConnectDuration  dd ?    ;时间/毫秒
RAS_STATS   ends

rastats RAS_STATS <>
long           dd 19ch
linkN          dd 0
rasconn        RASCONN <>
hStatic        dd 0
miao           dd 0
fen            dd 0
shi            dd 0
pfile          dd 0
Ttime          dd 0
Taddtime       dd 0
Ttimexian      dd 0
Fname          db "c:\Time.DAT",0
Iszero         db 0,0,0,0,1,0
tamp           db "本月累计上网约%d 小时%d 分钟%d 秒",0
XX             dd  0
play           db 0 dup (32)
ClassName      db "WinClass",0
AppName        db "NetLinkTime_Ver1.0 COPY(C)himcrack",0
RestoreString  db "(&R)显示程序",0
ExitString     db "(&X)退出程序",0
ButtonCNm      db "Static",0
buffer         db 0

.data?
mr             dd ?
hInstance      dd ?
note           NOTIFYICONDATA <>
hPopupMenu     dd ?

.code
start:
invoke GetModuleHandle, NULL
mov    hInstance,eax
invoke WinMain, hInstance,NULL,NULL, SW_SHOWDEFAULT

.if linkN!=0
invoke CreateFile,ADDR Fname,GENERIC_READ or GENERIC_WRITE ,\
             FILE_SHARE_READ or FILE_SHARE_WRITE,\
             NULL,OPEN_ALWAYS,FILE_ATTRIBUTE_NORMAL,\
             NULL
mov pfile,eax
lea ebx,Ttime
invoke WriteFile,eax,ebx,4,ADDR mr,0
mov eax,pfile
invoke CloseHandle,eax
.endif
invoke ExitProcess,eax

WinMain proc hInst:HINSTANCE,hPrevInst:HINSTANCE,CmdLine:LPSTR,CmdShow:DWORD
    LOCAL wc:WNDCLASSEX
    LOCAL msg:MSG
    LOCAL hwnd:HWND
    LOCAL LCtime:SYSTEMTIME
    mov   wc.cbSize,SIZEOF WNDCLASSEX
    mov   wc.style, CS_HREDRAW or CS_VREDRAW or CS_DBLCLKS
    mov   wc.lpfnWndProc, OFFSET WndProc
    mov   wc.cbClsExtra,NULL
    mov   wc.cbWndExtra,NULL
    push  hInst
    pop   wc.hInstance
    mov   wc.hbrBackground,COLOR_APPWORKSPACE
    mov   wc.lpszMenuName,NULL
    mov   wc.lpszClassName,OFFSET ClassName
    ;****************************************************************
    invoke GetModuleHandle,NULL
    invoke LoadIcon,eax,1
    ;****************************************************************
    mov   wc.hIcon,eax
    mov   wc.hIconSm,eax
    invoke LoadCursor,NULL,IDC_ARROW
    mov   wc.hCursor,eax
    invoke RegisterClassEx, addr wc
    invoke CreateWindowEx,WS_EX_CLIENTEDGE,ADDR ClassName,ADDR Appname,\
    WS_OVERLAPPED+WS_CAPTION+WS_SYSMENU+WS_MINIMIZEBOX+WS_VISIBLE,CW_USEDEFAULT,\
    CW_USEDEFAULT,400,200,NULL,NULL,\
    hInst,NULL
    mov   hwnd,eax
    ;****************************************************************
    invoke SendMessage,hwnd,WM_SIZE,SIZE_MINIMIZED,NULL
    lea    eax,LCtime
    invoke GetLocalTime,eax
    .if LCtime.wDay==1
    invoke CreateFile,ADDR Fname,GENERIC_READ or GENERIC_WRITE ,\
             FILE_SHARE_READ or FILE_SHARE_WRITE,\
             NULL,OPEN_ALWAYS,FILE_ATTRIBUTE_NORMAL,\
             NULL
    mov pfile,eax
    invoke SetFilePointer,eax,4,0,FILE_BEGIN
    mov eax,pfile
    invoke ReadFile,eax,addr buffer,1,addr mr,NULL
    .if buffer==0
    mov eax,pfile
    invoke SetFilePointer,eax,0,0,FILE_BEGIN
    mov eax,pfile
    lea ebx,Iszero
    invoke WriteFile,eax,ebx,5,ADDR mr,0
    .endif
    mov eax,pfile
    invoke CloseHandle,eax
    .elseif LCtime.wDay==2
    invoke CreateFile,ADDR Fname,GENERIC_READ or GENERIC_WRITE ,\
             FILE_SHARE_READ or FILE_SHARE_WRITE,\
             NULL,OPEN_ALWAYS,FILE_ATTRIBUTE_NORMAL,\
             NULL
    mov pfile,eax
    invoke SetFilePointer,eax,4,0,FILE_BEGIN
    mov eax,pfile
    lea ebx,Iszero
    invoke WriteFile,eax,ebx,1,ADDR mr,0
    mov eax,pfile
    invoke CloseHandle,eax
    .endif
    invoke CreateFile,ADDR Fname,GENERIC_READ or GENERIC_WRITE ,\
             FILE_SHARE_READ or FILE_SHARE_WRITE,\
             NULL,OPEN_ALWAYS,FILE_ATTRIBUTE_NORMAL,\
             NULL
    mov pfile,eax
    invoke ReadFile,eax,addr Ttime,4,addr mr,NULL
    mov eax,pfile
    invoke CloseHandle,eax
    invoke SetTimer,hwnd,1,1000,OFFSET TimProc
    ;****************************************************************
    .while TRUE
    invoke GetMessage, ADDR msg,NULL,0,0
    .BREAK .IF (!eax)
    invoke TranslateMessage, ADDR msg
    invoke DispatchMessage, ADDR msg
    .endw
    mov eax,msg.wParam
    ret
WinMain endp

WndProc proc hWnd:HWND, uMsg:UINT, wParam:WPARAM, lParam:LPARAM
    LOCAL pt:POINT
    .if uMsg==WM_CREATE
    Invoke CreateWindowEx,0,Addr ButtonCNm,addr play,WS_VISIBLE or WS_CHILD or WS_BORDER or BS_PUSHBUTTON,50,20,250,20,hWnd,IdPlay,hInstance,0
    mov hStatic,eax
    invoke CreatePopupMenu
    mov hPopupMenu,eax
    invoke AppendMenu,hPopupMenu,MF_STRING,IDM_RESTORE,addr RestoreString
    invoke AppendMenu,hPopupMenu,MF_STRING,IDM_EXIT,addr ExitString
    .elseif uMsg==WM_DESTROY
    invoke DestroyMenu,hPopupMenu
    invoke PostQuitMessage,NULL
    .elseif uMsg==WM_SIZE
    .if wParam==SIZE_MINIMIZED
    mov note.cbSize,sizeof NOTIFYICONDATA
    push hWnd
    pop note.hwnd
    mov note.uID,IDI_TRAY
    mov note.uFlags,NIF_ICON+NIF_MESSAGE+NIF_TIP
    mov note.uCallbackMessage,WM_SHELLNOTIFY
    ;****************************************************************
    invoke GetModuleHandle,NULL
    invoke LoadIcon,eax,1
    ;****************************************************************
    mov note.hIcon,eax
    invoke lstrcpy,addr note.szTip,addr AppName
    invoke ShowWindow,hWnd,SW_HIDE
    invoke Shell_NotifyIcon,NIM_ADD,addr note
    .endif
    .elseif uMsg==WM_COMMAND
    .if lParam==0
    invoke Shell_NotifyIcon,NIM_DELETE,addr note
    mov eax,wParam
    .if ax==IDM_RESTORE
    invoke ShowWindow,hWnd,SW_RESTORE
    .else
    invoke DestroyWindow,hWnd
    invoke KillTimer,hWnd,1
    .endif
    .endif
    .elseif uMsg==WM_SHELLNOTIFY
    .if wParam==IDI_TRAY
    .if lParam==WM_RBUTTONDOWN
    invoke GetCursorPos,addr pt
    invoke SetForegroundWindow,hWnd
    invoke TrackPopupMenu,hPopupMenu,TPM_RIGHTALIGN,pt.x,pt.y,NULL,hWnd,NULL
    invoke PostMessage,hWnd,WM_NULL,0,0
    .elseif lParam==WM_LBUTTONDBLCLK
    invoke SendMessage,hWnd,WM_COMMAND,IDM_RESTORE,0
    .endif
    .endif
    .else
    invoke DefWindowProc,hWnd,uMsg,wParam,lParam
    ret
    .endif
    xor eax,eax
    ret
WndProc endp

TimProc proc hWnd:HWND,nMsg:UINT,nTimerid:UINT,dwTime:DWORD
;****************************************************************
;以下为写文件的 TIMER 回调函数
;****************************************************************
mov eax,sizeof rasconn
inc eax
mov rasconn.dwSize,eax
invoke RasEnumConnections,addr rasconn,addr long,addr linkN
.if linkN==0
mov eax,Ttimexian
mov Ttime,eax
jmp exit
.endif
mov eax,sizeof rastats
mov rastats.dwSize,eax
invoke RasGetConnectionStatistics,rasconn.hrasconn,addr rastats
mov eax,rastats.dwConnectDuration
mov ecx,1000
cdq
div ecx
mov Taddtime,eax ;本次当前联网秒数
;****************************************************************
mov eax,Ttime
add eax,Taddtime
mov Ttimexian,eax
invoke CreateFile,ADDR Fname,GENERIC_READ or GENERIC_WRITE ,\
             FILE_SHARE_READ or FILE_SHARE_WRITE,\
             NULL,OPEN_ALWAYS,FILE_ATTRIBUTE_NORMAL,\
             NULL
mov pfile,eax
lea ebx,Ttimexian
invoke WriteFile,eax,ebx,4,ADDR mr,0
mov eax,pfile
invoke CloseHandle,eax
mov eax,Ttimexian
mov ecx,60
cdq
div ecx
mov miao,edx
cdq
div ecx
mov fen,edx
mov shi,eax
invoke wsprintf,addr play,addr tamp,shi,fen,miao
invoke SetWindowText,hStatic,addr play

exit: ret
TimProc endp
end start
```
