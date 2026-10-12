
## 反汇编代码分析

```
0177:015C3037  XOR      CL,BL
0177:015C3039  XOR      CL,8D
0177:015C303C  MOV      BL,CL
0177:015C303E  MOV      [ESP+10],BL
0177:015C3042  MOV      EDX,[ESP+10]
0177:015C3046  PUSH     EDX
0177:015C3047  CALL     015C2F40
0177:015C304C  MOV      ECX,[ESP+40]
0177:015C3050  MOV      [ESP+ESI+18],AL
0177:015C3054  SUB      ECX,EDI
0177:015C3056  MOV      DL,[ECX+EBP-01]
0177:015C305A  XOR      DL,AL
0177:015C305C  XOR      DL,D8
0177:015C305F  PUSH     EDX
0177:015C3060  CALL     015C2F40
0177:015C3065  MOV      ECX,[ESP+44]
0177:015C3069  ADD      ESP,BYTE +08
0177:015C306C  MOV      [ESP+ESI+15],AL
0177:015C3070  ADD      ESI,BYTE +02
0177:015C3073  JMP      SHORT 015C3002
0177:015C3075  MOV      ESI,[ESP+40]
0177:015c3079  lea eax, dword ptr [esp+14]
```

经过上面的跳转已经来到这里，假注册出现。这里存放了真的注册码，`d esp+14`。

**总结：** 跟了一遍，发现与风兄所写的有很大出入，不知是否我的 `Winamp` 版本问题？

我的注册名和注册码：

`CNCG`
`rApCpHu9`

谨以此文献给我的好朋友风兄！

## 源码文件：dwakg.asm

去掉了 `.data` 段的 6 个定义，加上了自动生成注册码功能，去除了错误提示。其余两个文件照旧。

```asm
;=====dwakg.asm=====

.386

.model flat,stdcall

option casemap:none

include hd.h

_WinMain Proto :DWORD,:DWORD,:DWORD,:DWORD
Calculation Proto

.const

DLG_MAIN  equ    1000
IDC_UN    equ    1001
IDC_SN    equ    1002
ID_GEN    equ    1003
ID_EXIT   equ    1004

.data

MsgCap      db "Dance Revlution for winamp 注册机      by WenXinJY",0
hInstance   dd 0
UserLen     dd 0
szFormat    db "%i",0
UserID      db 80 dup (0)
SN          db 80 dup (0)

.radix 16

.code

start:

invoke GetModuleHandle,NULL
mov hInstance,eax
invoke DialogBoxParam,hInstance,DLG_MAIN,NULL,offset _WinMain,NULL
invoke ExitProcess,NULL

_WinMain  proc    uses ebx edi esi, \
               hWnd:DWORD,wMsg:DWORD,wParam:DWORD,lParam:DWORD
           mov     eax,wMsg
           .if     eax == WM_CLOSE
                invoke    EndDialog,hWnd,NULL
           .elseif eax == WM_INITDIALOG
                invoke SetWindowText,hWnd,ADDR MsgCap
                invoke SendDlgItemMessage,hWnd,IDC_SN,EM_SETREADONLY,TRUE,NULL
           .elseif eax == WM_COMMAND
                mov eax,wParam
                AND EAX,0FFFFH
                CMP EAX,IDC_UN
                JZ gENkEY
                .IF lParam!=0
                    .if eax==ID_GEN

gENkEY:
                invoke RtlZeroMemory,addr UserID,80
                invoke RtlZeroMemory,addr SN,80
                invoke GetDlgItemText, hWnd, IDC_UN, addr UserID, 80
                .if  eax!=NULL
                    mov UserLen,eax
                    invoke Calculation
                    invoke SetDlgItemText, hWnd, IDC_SN, addr SN
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
           xor ebx,ebx
           mov ecx,UserLen
           mov ebp,ecx
           shl ebp,1

Cal_Lop_1:
           lodsb
           xor bl,al      ;don't change bl
           loop Cal_Lop_1
           xor esi,esi

Cal_Next:
           cmp esi,8
           jl Cal_Lab_0
           cmp esi,ebp
           jge Cal_Exit

Cal_Lab_0:
           mov eax,esi
           test eax,eax
           jz Cal_Lab_1
           dec eax

Cal_Lab_1:
           mov edi,esi
           shr edi,1    ;save
           mov cl,SN[eax]
           xor cl,bl
           xor cl,8dh
           mov bl,cl      ;save
           movzx eax,cl
           mov ecx,3eh
           cdq
           idiv ecx
           mov al,3Dh
           sub al,dl
           cmp al,9
           jbe Cal_OK_11
           cmp al,0ah+1ah  ;10 个数字+26 个大写字母
           jb Cal_OK_12
           add al,'a'-(0a+1ah)      ;转化为小写字母
           jmp Cal_Lab_3

Cal_OK_11:

           add al,'0'                ;转化为数字
           jmp Cal_Lab_3

Cal_OK_12:
           add al,'A'-0ah            ;转化为大写字母

Cal_Lab_3:
           mov SN[esi],al
           mov ecx,UserLen
           sub ecx,edi
           mov dl,UserID[ecx-1]
           xor dl,al
           xor dl,0d8h
           movzx eax,dl
           mov ecx,3eh
           cdq
           idiv ecx
           mov al,3dh
           sub al,dl
           cmp al,9
           jbe Cal_OK_21
           cmp al,0ah+1ah  ;10 个数字+26 个大写字母
           jb Cal_OK_22
           add al,'a'-(0a+1ah)      ;转化为小写字母
           jmp Cal_Lab_4

Cal_OK_21:

           add al,'0'                ;转化为数字
           jmp Cal_Lab_4

Cal_OK_22:
           add al,'A'-0ah            ;转化为大写字母

Cal_Lab_4:
           inc esi
           mov SN[esi],al
           inc esi
           jmp Cal_Next

Cal_Exit:
           popad
           ret

Calculation Endp

           End start
```

## 资源文件：dfwakg.rc

```rc
;======dfwakg.rc=========

#include <\masm32\include\resource.h>

#define DLG_MAIN  1000
#define IDC_UN    1001
#define IDC_SN    1002
#define ID_GEN    1003
#define ID_EXIT   1004

2       icon      key.ico

DLG_MAIN DIALOG 64,53,240,52

STYLE DS_MODALFRAME | WS_POPUP | WS_VISIBLE | WS_CAPTION | WS_SYSMENU

CAPTION "Dance Revlution for winamp 注册机      by WenXinJY"

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

## 头文件：hd.h

```c
;========hd.h==========

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

`4dd466d0e1e0cc123afbe504da2462ca`
