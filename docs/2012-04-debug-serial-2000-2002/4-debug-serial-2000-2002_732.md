
# 易经八卦占卜

> 原文：[`www.pediy.com/kssd/pediy03/forum1278.htm`](https://www.pediy.com/kssd/pediy03/forum1278.htm)

下面的是易经八卦占卜程序的注册码生成过程，请告诉我详细的解释好吗？

可能的话写个注册机。

下载地址：`http://www.csdn.net/soft/openfile.asp?kind=1&id=13032`

```asm
0167:00401BCD  MOV      ECX,[EBP-38]
0167:00401BD0  MOV      EAX,[EBP-40]
0167:00401BD3  MOV      DL,[ECX+EAX]
0167:00401BD6  MOV      [EBP-45],DL
0167:00401BD9  MOVSX    ECX,BYTE [EBP-45]
0167:00401BDD  MOVSX    EAX,BYTE [EBP-45]
0167:00401BE1  IMUL     ECX,EAX
0167:00401BE4  MOVSX    EDX,BYTE [EBP-45]
0167:00401BE8  IMUL     ECX,EDX
0167:00401BEB  MOV      EAX,[EBP-40]
0167:00401BEE  INC      EAX
0167:00401BEF  MOV      EDX,[EBP-40]
0167:00401BF2  INC      EDX
0167:00401BF3  IMUL     EDX
0167:00401BF5  SUB      ECX,EAX
0167:00401BF7  MOV      EAX,[EBP-40]
0167:00401BFA  INC      EAX
0167:00401BFB  MOVSX    EDX,BYTE [EBP-45]
0167:00401BFF  IMUL     EDX
0167:00401C01  SUB      ECX,EAX
0167:00401C03  PUSH     ECX
0167:00401C04  CALL     00401CCC
0167:00401C09  POP      ECX
0167:00401C0A  MOV      ECX,4B
0167:00401C0F  CDQ
0167:00401C10  IDIV     ECX
0167:00401C12  ADD      DL,30
0167:00401C15  MOV      [EBP-46],DL
0167:00401C18  MOV      EAX,[EBP-38]
0167:00401C1B  MOV      EDX,[EBP-40]
0167:00401C1E  MOV      CL,[EBP-46]
0167:00401C21  MOV      [EAX+EDX],CL
0167:00401C24  INC      DWORD [EBP-40]
0167:00401C27  MOV      EAX,[EBP-40]
0167:00401C2A  MOV      EDX,[EBP-44]
0167:00401C2D  CMP      EAX,EDX
0167:00401C0A  MOV      ECX,4B
0167:00401C0F  CDQ       ;al=eax=result
0167:00401C10  IDIV     ECX ;divide by 4Bh,and get the remainder in dl then add it with
0167:00401C12  ADD      DL,30 ;30h to convert it to ASCII
```

Strongly recommend you study the basic knowledge carefully, Cracking is in fact a challenge to one, think and study more, then you will get more.

自己编译一下罢，如果要编译后的版本，可 qq 发给你（在线时），`BCG` 内部也有。

这个算法很简单，只不过演示了如何用汇编来写注册机，大部分是模板，`cal` 是算法部分，写其它注册机时可少加改动即可重用大部分代码，界面设计是简单的对话框，虽然简单，优点是速度快（体现不出来），体积小 `3.0k`（未压缩），当然我不提倡用汇编写注册机，这只不过是我的业余爱好而已。

```bat
;===================make.bat compiled with MASM32

cls

c:\masm32\bin\ml /c /coff /Cp cr_yj.asm

rc /v cr_yj.rc

c:\masm32\bin\link /SUBSYSTEM:WINDOWS /LIBPATH:c:\masm32\lib cr_yj.obj cr_yj.res
```

```rc
;===================cr_yj.rc

#include <c:\masm32\include\resource.h>

#define DLG_MAIN       1000
#define IDC_UN         1001
#define IDC_REG        1002
#define ID_GEN         1003
#define ID_EXIT        1004

DLG_MAIN DIALOG 54, 53, 244, 52
STYLE DS_MODALFRAME | WS_POPUP | WS_VISIBLE | WS_CAPTION | WS_SYSMENU
CAPTION "易经八卦彩票占卜程序.V6.3/7.0 Keygen by Hume/BCG"
FONT 8, "MS Sans Serif"
{
    DEFPUSHBUTTON "GENERATE", ID_GEN, 181, 6, 50, 14
    PUSHBUTTON "EXIT", ID_EXIT, 181, 24, 50, 14
    CONTROL "Hume", IDC_UN, "EDIT", WS_BORDER | WS_TABSTOP, 62, 10, 98, 12
    EDITTEXT IDC_REG, 62, 29, 109, 13, ES_READONLY | WS_BORDER | WS_TABSTOP
    LTEXT "USERNAME:", -1, 18, 12, 40, 10
    LTEXT "REG  CODE:", -1, 18, 30, 41, 9
}
```

```asm
;===================cr_yj.asm

.386
.model flat,stdcall
option casemap:none
include hd.h

_ProcDlg proto :DWORD, :DWORD, :DWORD, :DWORD

;->>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>data seg

    .data
User       db 80 dup(0)
Serial     db 80 dup(0)
Count      dd 0
adsn dd    0
MsgMesaage1 db "Must input Username  in order to generate Reg Code! ",0
MsgCap     db "By (C)hume,July,2001",0

    .data?
hInstance  HANDLE ?

    .const
DLG_MAIN equ    1000
IDC_UN     equ    1001
IDC_REG    equ    1002
ID_GEN     equ    1003
ID_EXIT    equ    1004

 ;-->>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>code seg

    .code
start:

    invoke    GetModuleHandle, NULL
    mov       hInstance, eax
    invoke    DialogBoxParam, hInstance, DLG_MAIN, NULL, offset _ProcDlg, 0
    invoke    ExitProcess, NULL

_ProcDlg    proc    uses ebx edi esi, \
    hWnd:DWORD, wMsg:DWORD, wParam:DWORD, lParam:DWORD
    mov       eax, wMsg
    .if       eax == WM_CLOSE
        invoke    EndDialog, hWnd, NULL
    .elseif    eax == WM_INITDIALOG
        invoke    SetWindowPos, hWnd, HWND_TOPMOST, 0, 0, 0, 0, \
            SWP_NOMOVE or SWP_NOSIZE or SWP_DRAWFRAME
    .elseif    eax == WM_COMMAND
        mov eax, wParam
        .IF lParam!=0
            .if ax==ID_GEN
            invoke RtlZeroMemory, addr User, 80
            invoke RtlZeroMemory, addr Serial, 80
            invoke GetDlgItemText, hWnd, IDC_UN, addr User, 80
            .if eax!=NULL
            mov Count, eax
            call Cal
            invoke SetDlgItemText, hWnd, IDC_REG, addr Serial
            .else
            invoke MessageBox, NULL, addr MsgMesaage1, addr MsgCap, MB_OK or MB_ICONEXCLAMATION
            .endif

            .elseif ax==ID_EXIT
            invoke SendMessage, hWnd, WM_CLOSE, NULL, NULL
            .endif
        .ENDIF
    .else
    mov eax, FALSE
    ret
    .endif
    mov eax, TRUE
    ret
_ProcDlg    ENDP

Cal       PROC    uses eax ebx
    xor       edx, edx
    mov       ecx, 0
    lea       ebx, Serial
    mov       adsn, ebx
lopcal: LEA       EDI, [ECX+01]
    lea       eax, User
    add       eax, ecx
    MOV       BL, [EAX]
    MOVSX     ESI, BL
    MOV       EAX, ESI
    MOV       EBX, 4Bh
    IMUL      ESI
    IMUL      ESI
    LEA       EDX, [ECX+01]
    IMUL      EDX, EDI
    SUB       EAX, EDX
    LEA       EDX, [ECX+01]
    IMUL      EDX, ESI
    SUB       EAX, EDX
    INC       ECX
    CDQ
    XOR       EAX, EDX
    SUB       EAX, EDX
    CDQ
    IDIV      EBX
    MOV       EAX, EDX
    ADD       AL, 30h
    mov       ebx, adsn
    add       ebx, ecx
    dec       ebx
    MOV       [ebx], AL
    inc       ebx
    inc       eax
    CMP       ECX, Count
    JL        lopcal    ;//比较循环是否结束循环,计算
ToEit:      ret
Cal         endp

;-->>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>end all

    end start
```

```c
#include <stdio.h>
#include <string.h>

main()
{
    char name[80];
    register int i;
    long int tmp;
    printf("***********************************************************\n");
    printf("*YiJIng Lottery Program Key_Gen by hume*\n");
    printf("***********************************************************\n");
    printf("Please input your register name:");
    gets(name);
    printf("\nYour register Code Is:");
    for (i=0;i<strlen(name);i++ )
    {
        tmp=name[i];
        tmp=((tmp*tmp*tmp-(i+1)*(i+1)-(i+1)*tmp)%0x4b+0x30);
        printf("%c",(char)tmp);
    }
}
```
`4dd466d0e1e0cc123afbe504da2462ca`
