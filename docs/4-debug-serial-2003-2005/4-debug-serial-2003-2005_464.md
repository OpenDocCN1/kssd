
# 轻松提取资源 v1.82 注册机 (MASM32)

> 原文：[`www.pediy.com/kssd/pediy06/pediy6785.htm`](https://www.pediy.com/kssd/pediy06/pediy6785.htm)

SoftWare：轻松提取资源 v1.82 注册机 (MASM32)

Cracker：lq7972 [bruceyu13@sina.com]

Notes：学习。今天 PJ 完这个软件后，发现 simonyan 分析过它的老版本(1.45)(精华 V)；新旧之间没有很大的区别，只是对注册码的加密做了一点变动，同时在计算注册码前加入了几个黑名单，如“ttdown”“crsky”等。这个软件很适合新手练习比如我啦

### KeyGen.asm 文件

```asm
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
; The KeyGen by lq7972,with MASM32 V8
; E-mail:bruceyu13@sina.com
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.386
.model flat,stdcall
option casemap:none

;Include 文件定义
include windows.inc
include user32.inc
includelib user32.lib
include kernel32.inc
includelib kernel32.lib
include gdi32.inc
includelib gdi32.lib
include comdlg32.inc
includelib comdlg32.lib
include masm32.inc
includelib masm32.lib

;Equ 等值定义
ICO_MAIN equ 1000H
DLG_MAIN equ 1
EditName equ 10
EditSN equ 11

;********************************************************************

.data?
szSN              db ?
szName db 20 dup (?)
Temp db ?

.data
hInstance dd 0
szErr db '请输入用户名！',0
szCaption db '错误！',0
szString db 'huydong',0

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.code

;********************************************************************

_RegCodCalc proc
pushad
invoke lstrlen,offset szName
mov esi,eax
sub ecx,ecx

@@:
lea eax,szName
mov ebp,7H
mov bl,BYTE ptr [eax+ecx]
mov eax,ecx
cdq
idiv ebp
movsx eax,bl
mov ebx,ecx
movsx edx,BYTE ptr [szString+edx]
add ebx,edx
add eax,ebx
mov ebx,9H
add eax,esi
cdq
idiv ebx
add dl,30H
inc ecx
mov BYTE ptr [Temp+ecx-1],dl
cmp ecx,esi
jl @B
invoke revstr,addr Temp,addr szSN ;这是 MASM32.lib 中的翻转字符串函数

;最后一位注册码
lea eax,DWORD ptr [esi+4DH]
mov ecx,9
cdq
idiv ecx
add dl,30H
mov BYTE ptr [szSN+esi],dl

popad
ret

_RegCodCalc endp

;********************************************************************

_ProcDlgMain proc uses ebx edi esi ecx edx,hWnd,wMsg,wParam,lParam

mov eax,wMsg

.if eax==WM_CLOSE
invoke EndDialog,hWnd,NULL
.elseif eax==WM_COMMAND
mov eax,wParam
.if eax == IDOK
invoke RtlZeroMemory,addr szSN,100
invoke GetDlgItemText,hWnd,EditName,offset szName,20
.if eax != NULL
invoke _RegCodCalc
invoke SetDlgItemText,hWnd,EditSN,offset szSN
mov eax,FALSE
ret
         .else
invoke MessageBox,NULL,offset szErr,offset szCaption,MB_OK
mov eax,FALSE
ret
.endif

.elseif eax == IDCANCEL
invoke EndDialog,hWnd,NULL
.endif

        .else
                mov    eax,FALSE
                ret
        .endif

mov eax,TRUE
ret

_ProcDlgMain endp

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

start:
invoke GetModuleHandle,NULL
mov hInstance,eax
invoke DialogBoxParam,hInstance,DLG_MAIN,NULL,offset _ProcDlgMain,0
invoke ExitProcess,NULL

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

end start
```

### KeyGen.rc 文件

```rc
#include
#define ICO_MAIN 0x1000
#define DLG_MAIN 1
#define EDITName 10
#define EDITSN 11

//ICO_MAIN ICON "01.ico"

DLG_MAIN DIALOG 100,150,250,60
STYLE DS_MODALFRAME | WS_POPUP | WS_VISIBLE | WS_CAPTION | WS_SYSMENU | WS_THICKFRAME
CAPTION "轻松提取资源 v1.82 注册机"
FONT 9,"宋体"
{
 CONTROL "Name:" ,-1,"Static",SS_LEFT,10,13,40,17
 CONTROL "Code:" ,-2,"Static",SS_CENTER,10,40,20,17
 CONTROL "" ,10,"Edit",ES_LEFT,30,13,150,10
 CONTROL "" ,11,"Edit",ES_LEFT,30,40,150,10
 DEFPUSHBUTTON "GENERATE",IDOK,200,11,40,15
 PUSHBUTTON "EXIT",IDCANCEL,200,36,41,14
}
```

### makefile 文件

```makefile
NAME = KeyGen
OBJS = $(NAME).obj
RES = $(NAME).res
LINK_FLAG = /subsystem:windows
ML_FLAG = /c /coff

$(NAME).exe: $(OBJS) $(RES)
Link $(LINK_FLAG) $(OBJS) $(RES)

.asm.obj:
ml $(ML_FLAG) $<

.rc.res:
rc $<

clean:
del *.obj
del *.res
```
