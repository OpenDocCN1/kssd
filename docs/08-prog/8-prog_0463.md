
# Fern.asm

> 原文：[`www.pediy.com/kssd/pediy12/131673.html`](https://www.pediy.com/kssd/pediy12/131673.html)

```asm
;数学之美！
;用函数表示一个人体，精确到一根发丝。
;用微分方程描述宇宙若干年后的状态，精确到一个微尘，甚至夸克等基本粒子。
;以前看过一个递归画三角形的程序，那这个和它比比。
;这是一个画点形成树叶的程序。
;不足之处，敬请指导。
;QQ:112426112
;Email:leguanyuan at 126 dot com
;Homepage:http://correy.webs.com

.386
.model flat,stdcall
option casemap:none

include windows.inc
include kernel32.inc
include user32.inc
includelib user32.lib
includelib kernel32.lib
include gdi32.inc
includelib gdi32.lib

.code

hInstance dd ?
hWinMain dd ?
stMsg MSG <>
szClassName db "correy",0
szCaptionMain db "made by correy",0
pwndclassex dd 48,3,offset liuchunli,0,0,0,0,0,6,0,offset szClassName,0

cxClient DD ?
cyClient DD ?
colour DD ?
p DD 328,2621,4915
limit DD 250000
j DD ?
scaleX DD ?
scaleY DD ?
x_offset DD ?
y_offset DD ?
xx DD ?
yy DD ?
random_init DD 0ae37a9bh;随机数的种子。

a  real4 0.0,0.20,-0.15,0.85
b  real4 0.0,-0.26,0.28,0.04
cc real4 0.0,0.23,0.26,-0.04
d  real4 0.16,0.22,0.24,0.85
e  real4 0.0,0.0,0.0,0.0
f  real4 0.0,1.60,0.44,1.6
x  real4 0.0
y  real4 0.0

liuchunli proc hWnd:HWND,uMsg:UINT,wParam:WPARAM,lParam:LPARAM
LOCAL hdc:HDC, ps:PAINTSTRUCT

.IF uMsg==WM_SIZE
  mov eax,lParam
  and eax,0FFFFh
  mov cxClient,eax
  shr eax,1
  mov x_offset,eax
  mov ecx,15
  mov eax,cxClient
  mov edx,0
  div ecx
  mov scaleX,eax
  mov ebx,lParam
  shr ebx,16
  mov cyClient,ebx
  shr ebx,3
  mov y_offset,ebx
  mov ecx,12
  mov eax,cyClient
  mov edx,0
  div ecx
  mov scaleY,eax
  mov colour,0ff00h
.ELSEIF uMsg==WM_PAINT
  invoke BeginPaint,hWnd, ADDR ps
  mov    hdc,eax
  finit
  mov esi,0
  .WHILE esi < limit
    mov al,16
    mov cl,al ;生成随机数
    xor eax,eax
    mov bl,byte ptr random_init
    and bl,1
    EVEN
    Gen_bit:shl eax,1
    mov edx,random_init
    shr edx,9
    xor bl,dl
    shr edx,5
    xor bl,dl
    bt ebx,1
    rcr random_init,1
    setc bl
    or al,bl
    dec cl
    jnz Gen_bit
    .IF eax < p[0]
      mov ebx,0
    .ELSEIF eax < p[4]
      mov ebx,1
    .ELSEIF eax < p[8]
      mov ebx,2
    .ELSE
      mov ebx,3
    .ENDIF
    shl ebx,2
    fld dword ptr a[ebx]
    fmul x
    fld dword ptr b[ebx]
    fmul y
    fadd
    fadd e[ebx]
    fld dword ptr cc[ebx]
    fmul x
    fld dword ptr d[ebx]
    fmul y
    fadd
    fadd dword ptr f[ebx]
    fstp y
    fst x
    fimul scaleX
    fiadd x_offset
    fistp xx
    mov ecx,xx
    fld y
    fimul scaleY
    fiadd y_offset
    fistp yy
    mov edx,yy
    .IF (ecx >= 0 && ecx < cxClient && edx >= 0 && edx < cyClient)
      invoke SetPixel,hdc,xx,yy,colour
    .ENDIF
    inc esi
  .ENDW
  invoke EndPaint,hWnd,ADDR ps
.ELSEIF uMsg==WM_DESTROY
  invoke PostQuitMessage,0
.ELSE
  invoke DefWindowProc,hWnd,uMsg,wParam,lParam
ret
.ENDIF
xor eax,eax
ret

liuchunli endp

start:invoke GetModuleHandle,0
mov hInstance,eax
mov pwndclassex+20,eax
invoke LoadIcon,hInstance,1;其实这四行也可以去掉。
mov pwndclassex+24,eax
invoke LoadCursor,0,32512
mov pwndclassex+28,eax
invoke RegisterClassEx,addr pwndclassex
invoke CreateWindowEx,200h,offset szClassName,offset szCaptionMain,00cf0000h,80000000h,80000000h,1028,768,0,0,hInstance,0;0cf0000h ca0000h
mov hWinMain,eax
invoke ShowWindow,hWinMain,1;不想显示这一两行也可以不要。
invoke UpdateWindow,hWinMain
again:invoke GetMessage,addr stMsg,0,0,0
  cmp eax,0
  je exit
  invoke TranslateMessage,addr stMsg;这一行也可以去掉，特别是不处理字符信息。
  invoke DispatchMessage,addr stMsg
jmp again
exit:invoke ExitProcess,0
end start
;made at 2010.03.31
```

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || 6.rar |
| :--- | :--- |
