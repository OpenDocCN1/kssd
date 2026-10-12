
# HexDiff Version 2.51

> 原文：[`www.pediy.com/kssd/pediy06/pediy6788.htm`](https://www.pediy.com/kssd/pediy06/pediy6788.htm)

`Software`: `HexDiff Version 2.51`

[`www.eftstudio.com`](http://www.eftstudio.com)

文件比较工具。可以快速的在两个或者多个文件间进行比较，找出不同的地方

`Tools`: `TRW 2000`

`Cracker`: `lq7972[bruceyu13@sina.com]`

`Notes`: 学习中～

这个软件的注册无论成功与否，都没有诸如“Thanks”或“Sorry”的信息框；下断点 `bpx hmemcpy` 在跟踪中也不时从 `HexDiff` 领空和 `Kernel32/User32` 领空间跳动，我们可以用 `pmodule` 返回来

>>它的注册信息放在注册表的 `\HKEY_CURRENT_USER\Software\EFT Studio\HexDiff v2.5\Register`，与注册表有关的断点应该好一些，不过我没去试:-)

下面看注册算法

(跟踪很简单，就是利用 `F10` 带过 `Call`，敲 `D Register` 来看它的作用，详细过程本文略)

按 12 次(?)`F12`，敲 `pmodule`，再按 8 次 `F12`，按 `F10` 返回，一直到 `00412235` 按 `F8`，按 `F10` 到 `0040D5F8`，`F8` 跟进，就找到注册算法了～够快吧:)

```
0167:0040D4F0 53             PUSH     EBX
0167:0040D4F1 56             PUSH     ESI
0167:0040D4F2 8B74240C       MOV      ESI,[ESP+0C] ;用户名
0167:0040D4F6 57             PUSH     EDI
0167:0040D4F7 8BFE           MOV      EDI,ESI
0167:0040D4F9 83C9FF         OR       ECX,BYTE -01 ;下面几行算用户名长度
0167:0040D4FC 33C0           XOR      EAX,EAX
0167:0040D4FE 33DB           XOR      EBX,EBX
0167:0040D500 F2AE           REPNE SCASB
0167:0040D502 F7D1           NOT      ECX
0167:0040D504 49             DEC      ECX
0167:0040D505 8BC1           MOV      EAX,ECX ;这几行把用户名后填 0
0167:0040D507 881C30         MOV      [EAX+ESI],BL
0167:0040D50A 885C3001       MOV      [EAX+ESI+01],BL
0167:0040D50E 885C3002       MOV      [EAX+ESI+02],BL
0167:0040D512 885C3003       MOV      [EAX+ESI+03],BL
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
0167:0040D516 99             CDQ      ;哈，要进入注册码的计算了
0167:0040D517 83E203         AND      EDX,BYTE +03
0167:0040D51A 03C2           ADD      EAX,EDX
0167:0040D51C BAE7A9E771     MOV      EDX,71E7A9E7
0167:0040D521 C1F802         SAR      EAX,02
0167:0040D524 40             INC      EAX
0167:0040D525 3BC3           CMP      EAX,EBX
0167:0040D527 7E0A           JNG      0040D533
;====================================================================================
0167:0040D529 8B0E           MOV      ECX,[ESI]
0167:0040D52B 83C604         ADD      ESI,BYTE +04
0167:0040D52E 03D1           ADD      EDX,ECX ;我们要的东东
0167:0040D530 48             DEC      EAX
0167:0040D531 75F6           JNZ      0040D529
;====================================================================================
0167:0040D533 8B7C2414       MOV      EDI,[ESP+14]
```

你搞定了吗？是不是很简单呢？

最后，还是写个注册机吧

## 注册机

```
;KeyGen.asm
/////////////////////////////////////////////////////////////////////////////////////
// HexDiff Version 2.51 KeyGen
// TeH kEygeN bY lq7972, wITh WIn32mAsM
// E-Mail:bruceyu13@sina.com
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
;ICO_MAIN equ 1000H
DLG_MAIN equ 1
EditName equ 10
EditSN equ 11
;****************************************************************************************
.data?
szName db 512 dup (?)
szSN       db 8 dup (?)

.data
hInstance dd 0
szErr db '请输入用户名！',0
szCaption db '错误！',0
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.code
;****************************************************************************************
_RegCodCalc proc
pushad
lea esi,szName
invoke lstrlen,offset szName
cdq
mov edx,071E7A9E7H
sar eax,2
inc eax
@@:
mov ecx,[esi]
add esi,4
add edx,ecx
dec eax
jnz @B
invoke dw2hex,edx,offset szSN ;这是 masm32 的转换函数
popad
ret
_RegCodCalc endp
;****************************************************************************************
_ProcDlgMain proc uses ebx edi esi ebp hWnd,wMsg,wParam,lParam
mov eax,wMsg
.if eax  ==  WM_CLOSE
invoke  EndDialog,hWnd,NULL
.elseif eax  ==  WM_COMMAND
mov eax,wParam
.if eax  ==  IDOK
invoke RtlZeroMemory,offset szName,512
invoke  GetDlgItemText,hWnd,EditName,offset szName,512
.if eax !=  NULL
invoke _RegCodCalc
invoke SetDlgItemText,hWnd,EditSN,offset szSN
mov eax,FALSE
ret
          .else
invoke MessageBox,NULL,offset szErr,offset szCaption,MB_OK
mov eax,FALSE
ret
.endif
.elseif eax  ==  IDCANCEL
invoke  EndDialog,hWnd,NULL
.endif
.else
mov    eax,FALSE
ret
.endif
mov eax,TRUE
ret
_ProcDlgMain endp

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
start:
invoke GetModuleHandle,NULL
mov hInstance,eax
invoke DialogBoxParam,hInstance,DLG_MAIN,NULL,offset _ProcDlgMain,NULL
invoke ExitProcess,NULL
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
end start
```

```
//KeyGen.rc
#include
#define ICO_MAIN 0x1000
#define DLG_MAIN 1
#define EDITName 10
#define EDITSN 11

//ICO_MAIN ICON "01.ico"

DLG_MAIN DIALOG 100,150,250,60
STYLE DS_MODALFRAME | WS_POPUP | WS_VISIBLE | WS_CAPTION | WS_SYSMENU | WS_THICKFRAME
CAPTION "exDiff Version 2.51 注册机"
FONT 9,"宋体"
{
 CONTROL "Name:" , -1, "Static", SS_LEFT, 10, 13, 40, 17
 CONTROL "Code:" , -2, "Static", SS_CENTER, 10, 40, 20, 17
 CONTROL "" , 10, "Edit", ES_LEFT, 30, 13, 150, 10
 CONTROL "" , 11, "Edit", ES_LEFT, 30, 40, 150, 10
 DEFPUSHBUTTON "GENERATE", IDOK, 200, 11, 40, 15
 PUSHBUTTON "EXIT", IDCANCEL, 200, 36, 41, 14
}
```

## makefile

```
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
