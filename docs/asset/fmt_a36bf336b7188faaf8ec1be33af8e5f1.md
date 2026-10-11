
# Gif2Swf Ver 2.1

> 原文：[`www.pediy.com/kssd/pediy04/kanxue276.htm`](https://www.pediy.com/kssd/pediy04/kanxue276.htm)

```c
/*Gif2Swf Ver 2.1 KeyGen.c*/

#include "stdio.h"

main()
{
long x=0x163ef7;
char name[80];
long sn=0;
long temp;
int i;

puts("\n\t\tGif2Swf Ver 2.1 KeyGen  --- By WenXinJY --- \n\t\t============================================\n Please input your name:");

scanf("%s",name);

for(i=0;i<0x20&&name[i];i++)
{name[i]-=name[i]>='a'&&name[i]<='z'?'a'-'A':0;sn+=(x/name[i]+i)*name[i];}

printf("\nYour regcode is:%ld",(unsigned long)(sn+0x85d9ed) );

}
```

```asm
;Gif2Swf-kg.asm

.386
.model flat,stdcall
option casemap:none
include hd.h

 _WinMain Proto  :DWORD,:DWORD,:DWORD,:DWORD
 Calculation Proto

.const
 DLG_MAIN  equ     1000
 IDC_UN    equ     1001
 IDC_SN    equ     1002
 ID_GEN    equ     1003
 ID_EXIT   equ     1004

.data
MsgCap         db "Gif2Swf Ver 2.1 注册机                 by WenXinJY",0
MsgName       db "请输入您的用户名！",0
hInstance    dd 0
UserLen      dd 0
szFormat     db "%i",0
UserID       db 80 dup (0)
SN           db 80 dup (0)

.radix 16

.code

start:
invoke GetModuleHandle,NULL
mov hInstance,eax
invoke DialogBoxParam,hInstance,DLG_MAIN,NULL,offset _WinMain,NULL
invoke ExitProcess,NULL

_WinMain  proc    uses ebx edi esi, \
            hWnd:DWORD,wMsg:DWORD,wParam:DWORD,lParam:DWORD
            mov    eax,wMsg
            .if    eax == WM_CLOSE
                invoke    EndDialog,hWnd,NULL
            .elseif eax == WM_INITDIALOG
                    invoke SetWindowText,hWnd,ADDR MsgCap
                    invoke SendDlgItemMessage,hWnd,IDC_SN,EM_SETREADONLY,TRUE,NULL
            .elseif eax == WM_COMMAND
                    mov eax,wParam
                    .IF lParam!=0  
                        .if eax==ID_GEN
                            invoke GetDlgItemText, hWnd, IDC_UN, addr UserID, 80
                            .if  eax!=NULL
                                mov UserLen,eax
                                invoke Calculation
                                invoke SetDlgItemText, hWnd, IDC_SN, addr SN
                            .else
                                invoke MessageBox,NULL,addr MsgName,addr MsgCap,MB_OK or MB_ICONEXCLAMATION
                            .endif
                        .elseif eax==ID_EXIT
                            invoke SendMessage,hWnd,WM_CLOSE,NULL,NULL
                        .endif
                    .ENDIF
            .else
                    mov eax,FALSE
                    ret
            .endif
            mov eax,TRUE
            ret

_WinMain  ENDP       


Calculation Proc
            pushad
            lea esi,UserID
            mov edi,esi

Cvt:
            lodsb
            .if al>='a'
                .if al<='z'
                    sub al,'z'-'Z'
                .endif
            .endif
            stosb
            test al,al
            jnz Cvt
            xor esi,esi
            xor edi,edi
            xor ecx,ecx
            lea ebp,UserID

Next:
            mov eax, 00163EF7
            mov bl, byte ptr [esi+ebp]
            test bl,bl
            jz OK
            movsx edi, bl
            cdq
            idiv edi
            movsx edx, bl
            add eax, esi
            imul eax, edx
            add ecx, eax
            inc esi
            cmp esi,20
            jb Next

OK:
            add ecx, 0085D9ED
            push ecx
            push offset szFormat
            push offset SN
            call wsprintf
            add esp,4*3
            popad
            ret

Calculation Endp

            End start
```

```rc
;gif2swf-kg.rc

#include <\masm32\include\resource.h>

#define DLG_MAIN  1000

#define IDC_UN    1001

#define IDC_SN    1002

#define ID_GEN    1003

#define ID_EXIT   1004

2       icon       key.ico

DLG_MAIN DIALOG 64,53,240,52

STYLE DS_MODALFRAME | WS_POPUP | WS_VISIBLE | WS_CAPTION | WS_SYSMENU

CAPTION "Gif2Swf Ver 2.1 注册机                 by WenXinJY"

FONT 9, "宋体"

{

DEFPUSHBUTTON "注册", ID_GEN, 200, 8, 30, 14

PUSHBUTTON "退出", ID_EXIT, 200, 26, 30, 14

EDITTEXT IDC_UN, 40, 10, 150, 12, WS_BORDER | WS_TABSTOP

EDITTEXT IDC_SN, 40, 29, 150, 13, WS_BORDER | WS_TABSTOP

LTEXT "用户名:", -1, 8, 12, 30, 10

LTEXT "注册码:", -1, 8, 30, 30, 9

}
```
