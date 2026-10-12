
# 天天 DIY 之 小型 钩子的实现

> 原文：[`www.pediy.com/kssd/pediy10/78007.html`](https://www.pediy.com/kssd/pediy10/78007.html)

曾几何时我遇到了钩子这个神奇的东西，特别是有些人用在挂中，感觉神奇，到底是怎么实现的呢？

本程序实现的是获得鼠标传递消息的窗口的句柄

注意编译成 `dll` ！然后调用就可以了，只是实现了一个思路而已....

代码：

```
.386

.model flat,stdcall

option casemap:none

include C:\\RadASM\\masm32\\include\\windows.inc

include C:\\RadASM\\masm32\\include\\kernel32.inc
include C:\\RadASM\\masm32\\include\\user32.inc

includelib C:\\RadASM\\masm32\\lib\\kernel32.lib
includelib C:\\RadASM\\masm32\\lib\\user32.lib

.const
WM\_MOUSEHOOK equ WM\_USER+6

.data
hInstance dd 0

.data?
hHook dd ?
hWnd dd ?

.code
DllEntry proc hInst:HINSTANCE ,reason:DWORD ,reserved1:DWORD
   
   PUSH hInst
   POP hInstance
     MOV EAX,TRUE
   ret

DllEntry endp


MouseProc proc nCode:DWORD ,wParam:DWORD,lParam:DWORD
   
   invoke CallNextHookEx,hHook,nCode,wParam,lParam
   mov EDX,lParam
   assume EDX:PTR MOUSEHOOKSTRUCT
   invoke WindowFromPoint,\[EDX\].pt.x,\[EDX\].pt.y;获得包含指定点的窗口的句柄
   invoke PostMessage,hWnd,WM\_MOUSEHOOK,EAX,0
   assume EDX:nothing
   XOR EAX,EAX
   
   ret

MouseProc endp


InstallHook proc hwnd:DWORD
   
   PUSH hwnd
   POP hWnd
   invoke SetWindowsHookEx,WH\_MOUSE,addr MouseProc,hInstance,NULL
   Mov hHook,EAX
   ret

InstallHook endp

UninstallHook proc
   
   invoke UnhookWindowsHookEx,hHook
   ret

UninstallHook endp

End DllEntry
```
