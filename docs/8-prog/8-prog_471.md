
# Astroid.asm

> 原文：[`www.pediy.com/kssd/pediy12/131670.html`](https://www.pediy.com/kssd/pediy12/131670.html)

```asm
;Astroid.asm  
;星形线极坐标方程如下：  
;x=acos³t  
;y=asin³t  
;不足之处，敬请指导。  
;QQ:112426112  
;Email:leguanyuan at 126 dot com  
;Homtpage:http://correy.webs.com  

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

.data?  
hInstance dd ?  
hWinMain dd ?  
stMsg MSG <>  

.code  
szClassName db "correy",0  
szCaptionMain db "made by correy",0  
pwndclassex dd 48,3,offset liuchunli,0,0,0,0,0,6,0,offset szClassName,0  
liuchunli proc uses ebx edi esi,hWnd,uMsg,wParam,lParam  
LOCAL ps:PAINTSTRUCT  
LOCAL hdc:HDC  
LOCAL rect:RECT  
local Count:DWORD, Linelength:DWORD, base:DWORD  
local x_offset:dword,y_offset:dword,temp:dword  

.if uMsg == WM_CLOSE  
  invoke DestroyWindow,hWinMain  
  invoke PostQuitMessage,0   
.elseif uMsg == WM_PAINT  
  invoke BeginPaint, hWnd, addr ps  
  mov hdc, eax  
  finit  
  mov Count, 0  
  mov x_offset,128h  
  mov y_offset,123h  
  mov eax, y_offset  
  mov Linelength, eax  
  mov esi,y_offset  
  shl esi, 1  
  .while Count < esi  
    mov eax, Count  
    mov temp, eax  
    shl temp, 1  
    fild temp  
    fimul Linelength  
    fild Count  
    fmul st(0),st(0)  
    fsubp st(1),st(0)    
    fsqrt        
    fistp base  
    mov eax, Count  
    invoke MoveToEx,hdc,x_offset,eax,0  
    mov eax, base  
    add eax, x_offset  
    invoke LineTo,hdc,eax,y_offset  
    mov eax, Count  
    invoke MoveToEx, hdc,x_offset,eax, 0  
    mov eax, x_offset  
    sub eax, base  
    invoke LineTo, hdc,eax,y_offset  
    mov eax, y_offset  
    shl eax, 1  
    sub eax, Count  
    invoke MoveToEx, hdc,x_offset,eax,0  
    mov eax, x_offset  
    add eax, base  
    invoke LineTo,hdc,eax,y_offset  
    mov eax,y_offset  
    shl eax,1  
    sub eax,Count  
    invoke MoveToEx,hdc,x_offset,eax,0  
    mov eax,x_offset  
    sub eax,base  
    invoke LineTo,hdc,eax,y_offset  
    add Count, 4  
    .endw  
  invoke EndPaint,hWnd,addr ps  
.else  
  invoke DefWindowProc,hWnd,uMsg,wParam,lParam  
  ret  
.endif  

xor eax,eax  
ret  

liuchunli endp  
start:invoke GetModuleHandle,0  
mov hInstance,eax  
mov pwndclassex+20,eax  
invoke LoadIcon,hInstance,1;加载图标，图标定义在资源文件中。  
mov pwndclassex+24,eax  
invoke LoadCursor,0,32512;加载箭头鼠标  
mov pwndclassex+28,eax  
invoke RegisterClassEx,addr pwndclassex  
invoke CreateWindowEx,200h,offset szClassName,offset szCaptionMain,0Ca0000h,80000000h,80000000h,620,620,0,0,hInstance,0  
mov hWinMain,eax  
invoke ShowWindow,hWinMain,1  
invoke UpdateWindow,hWinMain  
again:invoke GetMessage,addr stMsg,0,0,0  
  cmp eax,0  
  je exit  
  invoke TranslateMessage,addr stMsg  
  invoke DispatchMessage,addr stMsg  
jmp again  
exit:invoke ExitProcess,0  
end start  

;made at 2010.03.26  
;代码如诗，那是需要咬文嚼字的，更需要有品味的人欣赏。
```

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) | | `8.rar` |
