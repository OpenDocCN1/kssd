
# 【原创】密码查看器&&窗口激活专家

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-697.htm`](https://www.pediy.com/kssd/pediy08/pediy8-697.htm)

代码如下：

```asm
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;  Programmed by nohacks, kker.cn@163.com
;  Website: http://hacker0058.ys168.com
;             Win32 ASM is Masm
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;  版本信息
;  密码查看器   - 可以查看密码编辑框中的密码
;     &&窗口激活专家 - 可以激活灰色按纽
;     V1.0 ------  2006 年 7 月 25 日
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
    .386
    .model flat, stdcall
    option casemap :none

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;  Include 数据
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
include    windows.inc
include    user32.inc
include    kernel32.inc
includelib  user32.lib
includelib  kernel32.lib

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;  Equ 数据
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
ICO_1           equ             1
DLG_MAIN  equ    1000
ID_PWD    equ    1001
IDC_ENABLE      equ             1002
IDC_HANDLE      equ             1003
IDC_CurPos      equ             1004
IDC_CurHand     equ             1005

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;  数据段
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
.data
template db '%lX',0
posBuffer db 10 dup (0)
D       db  ",",0

.data?
hInstance  dd  ?
szBuffer  db  256 dup  (?)
phwnd           dd ?
X              db 10 dup(?)
Y              db 10 dup(?)

.code
EnableAll proc hWnd:HWND,lParam:LPARAM
  invoke  IsWindowEnabled,hWnd
  cmp  eax,TRUE
  je  getNext
  invoke  EnableWindow,hWnd,TRUE
  getNext:
  mov  eax,TRUE
  ret
EnableAll endp

;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
_ProcDlgMain  proc  uses ebx edi esi,
          hWnd:DWORD,wMsg:DWORD,wParam:DWORD,lParam:DWORD
        local  @stPoint:POINT
        local  @hWindow:dword
  local  @STYLE
        local   wc:WNDCLASSEX
  mov  eax,wMsg
  cmp  eax,WM_TIMER
  je  getPoint
  cmp  eax,WM_INITDIALOG
  je  boxStart
  cmp  eax,WM_CLOSE
  je  boxClose
retFalse:
  mov  eax,FALSE
  ret
boxClose:
        invoke  EndDialog,hWnd,NULL
        invoke  KillTimer,hWnd,1
        jmp  retTrue
boxStart:
  invoke LoadIcon,hInstance,ICO_1
  invoke  SendMessage,hWnd,WM_SETICON,ICON_BIG,eax    ;设置窗口图标
  invoke  SetWindowPos,hWnd,HWND_TOPMOST,0,0,0,0, SWP_NOMOVE or SWP_NOSIZE ;窗口置顶
  invoke  SetTimer,hWnd,1,200,NULL
  jmp  retTrue
getPoint:
        invoke IsDlgButtonChecked,hWnd,IDC_ENABLE
  cmp eax,BST_CHECKED
        jne Getpass
        invoke GetForegroundWindow
  push eax
        invoke wsprintf,offset posBuffer,offset template,eax
        invoke SetDlgItemText,hWnd,IDC_HANDLE,offset posBuffer
        pop eax
  invoke EnumChildWindows,eax,offset EnableAll,NULL
Getpass:
        invoke  GetCursorPos,addr  @stPoint          ;取得鼠标处的坐标位置
        invoke  wsprintf,offset X,offset template,@stPoint.x
  invoke  wsprintf,offset Y,offset template,@stPoint.y
        invoke lstrcat,offset  X, offset D
  invoke lstrcat,offset  X, offset Y
        invoke SetDlgItemText,hWnd,IDC_CurPos,   ADDR X     ;输出坐标位置
  invoke  WindowFromPoint,@stPoint.x,@stPoint.y      ;取得屏幕指定位置的句柄
        mov @hWindow,eax
        push eax
        invoke  wsprintf,offset posBuffer,offset template,eax
  invoke  SetDlgItemText,hWnd,IDC_CurHand,offset posBuffer
  pop eax
  cmp eax,0
  je  retTrue
  cmp eax ,phwnd
  je  retTrue
  mov phwnd,eax
  invoke  SendMessage ,@hWindow,EM_GETPASSWORDCHAR, 0, 0   ;取得编辑框遮盖符代码
        mov @STYLE,eax
  cmp eax,0    ;如果为 0,就不是密码框
  je  retTrue
  invoke  PostMessage,@hWindow, EM_SETPASSWORDCHAR, 0, 0  ;关键一句,去掉密码遮盖符
  invoke  SendMessage,@hWindow,WM_GETTEXT,256,offset szBuffer  ;取得编辑框文本→Pass
        invoke  PostMessage,@hWindow, EM_SETPASSWORDCHAR, @STYLE,0  ;恢复编辑框密码遮盖符
        invoke  SetDlgItemText,hWnd,ID_PWD,offset szBuffer ;输出密码明文到编辑框
  invoke ShowWindow,hWnd,9           ;发现密码后自动弹出
retTrue:
  mov  eax,TRUE
  ret
_ProcDlgMain  endp

start:
    invoke GetModuleHandle,NULL
    mov  hInstance,eax
    invoke DialogBoxParam,eax,DLG_MAIN,NULL,offset _ProcDlgMain,0
    invoke  ExitProcess,NULL
end start
```

程序心得：

记得原来刚接触电脑黑客时最感兴趣的就属破解网吧限制免费上网，当时有一款工具叫“Windows 突破专家”什么的，我用的最多，也梦想着有一天我也能写这样一个工具就好了，可是，当时我以为要会英语才能学编程，只好放弃，直到一次偶然的机会我知道了全中文内核的易语言，于是便迷上了编程，开始知道什么叫 `API`，怎么用 `API`，同时发现各种编程语言调用 `API` 的代码都是差不多的，我开始学着看 `VB`、`VC` 等程序的代码，不明白的地方就问百度问 `GOOGLE`，竟然也能看个大概，2 个星期前我想学习破解又开始学习汇编，发现真的很简单，只要知道了它的语法，剩下的就是 `API` 的调用了，在此推荐大家学习！可以在看学论坛置顶的学习资料处下载到汇编的入门教程，我喜欢 `Masm`，它的教程也多点，如果你讨厌在 `DOS` 下编译程序，可以和我一样用----`RadMAS`（一款汇编集成开发工具），可以高亮显示代码等非常好用，这些都可以在看雪工具处下载到，祝你好运！

程序界面：

![](img/fb5401353bb9f9307dea4789d616b80f.jpg)

`PS`：不知道你们感觉怎么样，我觉得用 `CMP` 比用宏命令 `.IF` 写的程序条理更清楚一点，特别是判断分支多的时候，那些 `.IF` 和 `.ENDIF` 看得我头都大了，所以就直接用 `CMP` 了。

`RadMAS` 工程源码：`getpassword060725.zip`

`4dd466d0e1e0cc123afbe504da2462ca`
