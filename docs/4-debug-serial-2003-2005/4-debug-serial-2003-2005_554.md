
# 争分夺秒背单词

> 原文：[`www.pediy.com/kssd/pediy05/pediy50216.htm`](https://www.pediy.com/kssd/pediy05/pediy50216.htm)

```
;从 hume 的例子中改的：)

;********************************crkslm.asm

.386

.model flat,stdcall

option casemap:none

include hd.h

_ProcDlg proto :DWORD,:DWORD,:DWORD,:DWORD

wsprintfA PROTO C :DWORD,:VARARG

BUFF_SIZE       equ    50

;->>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>data seg

       .data

esp_10          db     0  ;保存 ecx+5

esp_04          db     0  ;保存高 4 位

esp_08          db     0  ;保存低 4 位

flg             db     0

rand_init       db     0

rand_out         db     0

MsgTitle         db     "A KeyMaker by Fpc", 0

MsgContend       db     "XXXXXXXXXXXX", 0dh, 0ah, 0dh, 0ah,

                          "Hompage: www.abc.com", 0

Rc1             db      BUFF_SIZE dup(0)

Rc2             db      BUFF_SIZE dup(0)

Serial           db      BUFF_SIZE dup(0)

format           db      "%lu", 0

       .data?

hInstance  HANDLE ?

       .const

ID_GEN            equ    1003

ID_EXIT           equ    1002

ID_ABOUT          equ    1004

IDC_OUT           equ    1001

IDC_RC1           equ    1010

IDC_RC2           equ    1011

DLG_MAIN          equ    1000

  ;-->>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>code seg

       .code

start:

   invoke    GetModuleHandle,NULL

   mov       hInstance,eax

   invoke    DialogBoxParam,hInstance,DLG_MAIN,NULL,offset _ProcDlg,0

   invoke    ExitProcess,NULL

_ProcDlg    proc    uses ebx edi esi, \

       hWnd:DWORD,wMsg:DWORD,wParam:DWORD,lParam:DWORD

       mov    eax,wMsg

       .if    eax == WM_CLOSE

          invoke    EndDialog,hWnd,NULL

       .elseif    eax == WM_COMMAND

          mov eax,wParam

          .IF lParam!=0

            .if ax==ID_GEN

            ;invoke RtlZeroMemory, addr Serial, BUFF_SIZE

            invoke GetDlgItemText, hWnd, IDC_RC1, addr Rc1, BUFF_SIZE

            invoke GetDlgItemText, hWnd, IDC_RC2, addr Rc2, BUFF_SIZE

            call Cal

            invoke SetDlgItemText,hWnd,IDC_OUT,addr Serial

            .elseif ax==ID_ABOUT

            invoke MessageBox, hWnd, Addr MsgContend, Addr MsgTitle, MB_OK

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

_ProcDlg    ENDP

include  cal.asm

end start

;************************************cal.asm

Cal    proc

       PUSHAD

       lea edi, Rc2

       or ecx, 0ffffffffh

       xor eax, eax

       repnz scasb

       not ecx

       dec ecx

       test ecx, ecx

       jz no_input

       push ecx

       ;

       lea edi, Rc1

       or ecx, 0ffffffffh

       repnz scasb

       not ecx

       dec ecx

       test ecx, ecx

       jnz next_1

       pop ecx

       jmp no_input

next_1:

       lea edi, Rc1

       add edi, ecx

       lea esi, Rc2

       pop ecx

       rep movsb

       ;

       ;main proc

       ;

       xor eax, eax

       lea esi, Rc1

       mov ebx, eax

       inc ebx

main_loop:

       lodsb

       test al, al

       jz next_2

       imul ebx, ebx, 022d6h

       add ebx, eax

       jmp main_loop

next_2:

       mov eax, ebx

       cdq

       xor eax, edx

       sub eax, edx    ;done

       invoke wsprintfA, addr Serial, addr format, eax

no_input:

       POPAD

       ret

Cal    endp

;************************************rsrc.rc

#include <d:\\masm32\\include\\resource.h>

#define ID_GEN          1003

#define ID_ABOUT       1004

#define ID_EXIT          1002

#define IDC_OUT          1001

#define IDC_RC1          1010

#define IDC_RC2          1011

#define DLG_MAIN      1000

DLG_MAIN DIALOG 37, 61, 282, 59

STYLE DS_MODALFRAME | WS_POPUP | WS_VISIBLE | WS_CAPTION | WS_SYSMENU

CAPTION "Asm Keygen temple, By Fpc, thx to Hume"

FONT 8, "MS Sans Serif"

{

DEFPUSHBUTTON "Generate", ID_GEN, 209, 4, 50, 14

PUSHBUTTON "Exit", ID_EXIT, 209, 40, 50, 14

PUSHBUTTON "About", ID_ABOUT, 209, 22, 50, 14

LTEXT "Name     :", -1, 15, 8, 36, 8

LTEXT "Email    :", -1, 15, 25, 39, 8

LTEXT "Code    :", -1, 15, 43, 49, 10

EDITTEXT IDC_RC1, 67, 6, 48, 13, WS_BORDER | WS_TABSTOP,

EDITTEXT IDC_RC2, 67, 23, 71, 13, WS_BORDER | WS_TABSTOP,

EDITTEXT IDC_OUT, 67, 42, 130, 12, ES_READONLY | WS_BORDER | WS_TABSTOP

} 4dd466d0e1e0cc123afbe504da2462ca
```
