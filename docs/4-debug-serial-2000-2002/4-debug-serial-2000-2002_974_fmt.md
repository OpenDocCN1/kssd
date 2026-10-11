
# 信息技术教育考核注册机

> 原文：[`www.pediy.com/kssd/pediy04/kanxue185.htm`](https://www.pediy.com/kssd/pediy04/kanxue185.htm)

```
;rsrc.rc

#include <c:\masm32\include\resource.h>

#define DLG_MAIN  1000
#define IDC_CAP    1001
#define ID_MOD     1003
#define ID_EXIT    1004

DLG_MAIN DIALOG 64,53,180,50
STYLE DS_MODALFRAME | WS_POPUP | WS_VISIBLE | WS_CAPTION | WS_SYSMENU
CAPTION "信息技术教育考核注册机  by 5,555"
FONT 9, "宋体"
{
DEFPUSHBUTTON "生成注册码", ID_MOD, 50, 30, 50, 14
PUSHBUTTON "退出本程序", ID_EXIT, 120, 30, 50, 14
EDITTEXT IDC_CAP, 52, 10, 120, 12, WS_BORDER | WS_TABSTOP
LTEXT "注册码:", -1, 8, 12, 40, 10
}
```

```
;hd.h

      include \masm32\include\windows.inc
      include \masm32\include\user32.inc
      include \masm32\include\kernel32.inc
      include \masm32\include\gdi32.inc
      include \masm32\include\comctl32.inc
      include \masm32\include\comdlg32.inc
      include \masm32\include\shell32.inc
      include \masm32\include\advapi32.inc
      includelib \masm32\lib\user32.lib
      includelib \masm32\lib\kernel32.lib
      includelib \masm32\lib\gdi32.lib
      includelib \masm32\lib\comctl32.lib
      includelib \masm32\lib\comdlg32.lib
      includelib \masm32\lib\shell32.lib
      includelib \masm32\lib\advapi32.lib
```

```
;cr_jykh.asm

.386
.model flat,stdcall
option casemap:none
include hd.h
  _WinMain Proto  :DWORD,:DWORD,:DWORD,:DWORD
  Reg Proto
  DLG_MAIN  equ     1000
  IDC_SN    equ     1001
  ID_MOD    equ     1003
  ID_EXIT  equ     1004

.data
MsgCap       db "信息技术教育考核注册机  by 5,555",0
hInstance     dd 0
VName        db 256 dup (0)
FSName       db 256 dup (0)
Drive        db "C:\\",0
VID          dd 0
SN           db 20 dup (0)

.code

start:
invoke GetModuleHandle,NULL
mov hInstance,eax
invoke DialogBoxParam,hInstance,DLG_MAIN,NULL,offset _WinMain,NULL
invoke ExitProcess,NULL

_WinMain PROC  hWnd:DWORD,wMsg:DWORD,wParam:DWORD,lParam:DWORD
          mov    eax,wMsg
          .if    eax == WM_CLOSE
              invoke    EndDialog,hWnd,NULL
          .elseif    eax == WM_INITDIALOG
                   invoke SetWindowText,hWnd,ADDR MsgCap
          .elseif    eax == WM_COMMAND
              mov eax,wParam
              .IF lParam!=0
                .if ax==ID_MOD
                  invoke Reg
                  invoke SetDlgItemText,hWnd,IDC_SN,addr SN
                .elseif ax==ID_EXIT
                  invoke SendMessage,hWnd,WM_CLOSE,NULL,NULL
                .endif
              .ENDIF
          .else
              mov eax,FALSE
          ret
          .endif
          mov eax,TRUE
          ret
_WinMain ENDP

Reg Proc
  pushad
  invoke RtlZeroMemory,addr VName,256
  invoke RtlZeroMemory,addr FSName,256
  invoke RtlZeroMemory,addr SN,20
  invoke  GetVolumeInformation, addr Drive, addr VName, 255, addr VID, 0, 0, addr FSName, 255
  mov eax,VID
  xor eax,29704275h
  lea esi,SN
  mov edi,esi
  mov ebx,10

Reg_Lop1:
  cdq
  div ebx
  cmp eax,0
  jne ToAsc
  cmp edx,0
  je ReverseIt

ToAsc:
  add dl,'0'
  mov [esi],dl
  inc esi
  jmp Reg_Lop1

ReverseIt:
  dec esi

Reg_Lop2:
  cmp esi,edi
  jbe Reg_Exit
  mov al,[esi]
  xchg al,[edi]
  xchg al,[esi]
  inc edi
  dec esi
  jmp Reg_Lop2

Reg_Exit:
  popad
  ret

Reg Endp

end start
```
