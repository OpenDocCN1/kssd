
# IEEE 浮点数工具

> 原文：[`www.pediy.com/kssd/pediy06/pediy6938.htm`](https://www.pediy.com/kssd/pediy06/pediy6938.htm)

前不久学习了浮点指令，试着自己写了个转换程序，现贴出来让大家看看，写得不好请不要见笑。

界面见附件图片。

```
;********************************************************************************
; 程序功能：显示 IEEE 浮点数在内存中的存放以十六进制数或二进制方式显示
; 程序设计：Greatboy
; 程序建立时间：2004.11
;********************************************************************************

.386

.model flat, stdcall

option casemap :none

include windows.inc
include user32.inc
includelib user32.lib
include kernel32.inc
includelib kernel32.lib
include gdi32.inc
includelib gdi32.lib
include fpu.inc
includeLib fpu.lib

IDD_FLOATCONDLG EQU 1000
IDD_DOUBLECONDLG EQU 1001
IDC_ICO EQU 1104
IDC_COPYBUTTON EQU 1105
IDC_PASTEBUTTON EQU 1106
IDC_DECBUTTON EQU 1
IDC_EXITBUTTON EQU 2
IDC_HEXBUTTON EQU 3
IDC_BINBUTTON EQU 4
IDC_DOUBLEBUTTON EQU 1002
IDC_FLOATBUTTON EQU 1003
IDC_DDECEDIT EQU 1004
IDC_DHEXEDIT EQU 1005
IDC_DBINEDIT63 EQU 1006
IDC_DBINEDIT62 EQU 1007
IDC_DBINEDIT61 EQU 1008
IDC_DBINEDIT60 EQU 1009
IDC_DBINEDIT59 EQU 1010
IDC_DBINEDIT58 EQU 1011
IDC_DBINEDIT57 EQU 1012
IDC_DBINEDIT56 EQU 1013
IDC_DBINEDIT55 EQU 1014
IDC_DBINEDIT54 EQU 1015
IDC_DBINEDIT53 EQU 1016
IDC_DBINEDIT52 EQU 1017
IDC_DBINEDIT51 EQU 1018
IDC_DBINEDIT50 EQU 1019
IDC_DBINEDIT49 EQU 1020
IDC_DBINEDIT48 EQU 1021
IDC_DBINEDIT47 EQU 1022
IDC_DBINEDIT46 EQU 1023
IDC_DBINEDIT45 EQU 1024
IDC_DBINEDIT44 EQU 1025
IDC_DBINEDIT43 EQU 1026
IDC_DBINEDIT42 EQU 1027
IDC_DBINEDIT41 EQU 1028
IDC_DBINEDIT40 EQU 1029
IDC_DBINEDIT39 EQU 1030
IDC_DBINEDIT38 EQU 1031
IDC_DBINEDIT37 EQU 1032
IDC_DBINEDIT36 EQU 1033
IDC_DBINEDIT35 EQU 1034
IDC_DBINEDIT34 EQU 1035
IDC_DBINEDIT33 EQU 1036
IDC_DBINEDIT32 EQU 1037
IDC_DBINEDIT31 EQU 1038
IDC_DBINEDIT30 EQU 1039
IDC_DBINEDIT29 EQU 1040
IDC_DBINEDIT28 EQU 1041
IDC_DBINEDIT27 EQU 1042
IDC_DBINEDIT26 EQU 1043
IDC_DBINEDIT25 EQU 1044
IDC_DBINEDIT24 EQU 1045
IDC_DBINEDIT23 EQU 1046
IDC_DBINEDIT22 EQU 1047
IDC_DBINEDIT21 EQU 1048
IDC_DBINEDIT20 EQU 1049
IDC_DBINEDIT19 EQU 1050
IDC_DBINEDIT18 EQU 1051
IDC_DBINEDIT17 EQU 1052
IDC_DBINEDIT16 EQU 1053
IDC_DBINEDIT15 EQU 1054
IDC_DBINEDIT14 EQU 1055
IDC_DBINEDIT13 EQU 1056
IDC_DBINEDIT12 EQU 1057
IDC_DBINEDIT11 EQU 1058
IDC_DBINEDIT10 EQU 1059
IDC_DBINEDIT09 EQU 1060
IDC_DBINEDIT08 EQU 1061
IDC_DBINEDIT07 EQU 1062
IDC_DBINEDIT06 EQU 1063
IDC_DBINEDIT05 EQU 1064
IDC_DBINEDIT04 EQU 1065
IDC_DBINEDIT03 EQU 1066
IDC_DBINEDIT02 EQU 1067
IDC_DBINEDIT01 EQU 1068
IDC_DBINEDIT00 EQU 1069
IDC_FDECEDIT EQU 1070
IDC_FHEXEDIT EQU 1071
IDC_FBINEDIT31 EQU 1072
IDC_FBINEDIT30 EQU 1073
IDC_FBINEDIT29 EQU 1074
IDC_FBINEDIT28 EQU 1075
IDC_FBINEDIT27 EQU 1076
IDC_FBINEDIT26 EQU 1077
IDC_FBINEDIT25 EQU 1078
IDC_FBINEDIT24 EQU 1079
IDC_FBINEDIT23 EQU 1080
IDC_FBINEDIT22 EQU 1081
IDC_FBINEDIT21 EQU 1082
IDC_FBINEDIT20 EQU 1083
IDC_FBINEDIT19 EQU 1084
IDC_FBINEDIT18 EQU 1085
IDC_FBINEDIT17 EQU 1086
IDC_FBINEDIT16 EQU 1087
IDC_FBINEDIT15 EQU 1088
IDC_FBINEDIT14 EQU 1089
IDC_FBINEDIT13 EQU 1090
IDC_FBINEDIT12 EQU 1091
IDC_FBINEDIT11 EQU 1092
IDC_FBINEDIT10 EQU 1093
IDC_FBINEDIT09 EQU 1094
IDC_FBINEDIT08 EQU 1095
IDC_FBINEDIT07 EQU 1096
IDC_FBINEDIT06 EQU 1097
IDC_FBINEDIT05 EQU 1098
IDC_FBINEDIT04 EQU 1099
IDC_FBINEDIT03 EQU 1100
IDC_FBINEDIT02 EQU 1101
IDC_FBINEDIT01 EQU 1102
IDC_FBINEDIT00 EQU 1103

ProcFloatDlgMain PROTO :DWORD, :DWORD, :DWORD, :DWORD
ProcDoubleDlgMain PROTO :DWORD, :DWORD, :DWORD, :DWORD
FloatToHexText PROTO :DWORD, :DWORD, :DWORD
HexToBinText PROTO :DWORD, :DWORD, :DWORD
HexTextToDec PROTO :DWORD, :DWORD, :DWORD
BinTextToHex PROTO :DWORD, :DWORD, :DWORD
ProcDecEdit PROTO :DWORD, :DWORD, :DWORD, :DWORD
ProcHexEdit PROTO :DWORD, :DWORD, :DWORD, :DWORD

.data?

hInstance dd ?
hGlobal dd ?
lpOldProcDecEdit dd ?
lpOldProcHexEdit dd ?

.data

SeleDispDlg dd 1
szInputDec db 30 dup(0)
szInputHex db 17 dup(0)
szInputBin db 65 dup(0)
r4Data dd 0
r8Data dq 0
szChar db 0,0

.const

szAppName db '浮点数转换', 0
szErrPrompt1 db '十进制栏没有输入数值！',0
szErrPrompt2 db '输入的十进制数太长或错误！',0
szErrPrompt3 db '输入的十六进制数错误或长度不够！',0
szErrPrompt4 db '输入的二进制数错误（非 0 或 1）！',0
szErrPrompt5 db '剪贴板格式无效！',0
szAllowDecChar db '0123456789.', 08h
szAllowHexChar db '0123456789ABCDEFabcdef', 08h

.code

start:
 invoke GetModuleHandle, NULL
 mov hInstance, eax
 .while TRUE
 .if SeleDispDlg == 1
 invoke DialogBoxParam, hInstance, IDD_FLOATCONDLG,NULL, offset ProcFloatDlgMain, NULL
 .elseif SeleDispDlg == 2
 invoke DialogBoxParam, hInstance, IDD_DOUBLECONDLG,NULL, offset ProcDoubleDlgMain, NULL
 .else
 invoke ExitProcess, NULL
 .endif
 .endw

ProcFloatDlgMain proc uses ebx edx edi esi hWnd, wMsg, wParam, lParam
 mov eax,wMsg
 .if eax == WM_CLOSE
 mov SeleDispDlg, 0
 invoke EndDialog, hWnd, NULL
 .elseif eax == WM_INITDIALOG
 invoke LoadIcon, hInstance, IDC_ICO
 invoke SendMessage, hWnd,WM_SETICON, ICON_BIG, eax
 invoke SendDlgItemMessage, hWnd, IDC_FDECEDIT, EM_LIMITTEXT, 29, 0
 invoke SendDlgItemMessage, hWnd, IDC_FHEXEDIT, EM_LIMITTEXT, 8, 0
 ;invoke GetDlgItem, hWnd, IDC_FDECEDIT
 ;invoke SetWindowLong, eax, GWL_WNDPROC, addr ProcDecEdit
 ;mov lpOldProcDecEdit, eax
 invoke GetDlgItem, hWnd, IDC_FHEXEDIT
 invoke SetWindowLong, eax, GWL_WNDPROC, addr ProcHexEdit
 mov lpOldProcHexEdit, eax
 .elseif eax == WM_COMMAND
 mov eax,wParam
 .if ax == IDCANCEL
 mov SeleDispDlg, 0
 invoke EndDialog,hWnd,NULL
 .elseif ax == IDC_DOUBLEBUTTON
 mov SeleDispDlg, 2
 invoke EndDialog,hWnd,NULL
 .elseif ax == IDC_DECBUTTON
 invoke GetDlgItem, hWnd, IDC_FDECEDIT
```



```
invoke GetWindowText, eax, offset szInputDec, 30    ; 千万注意最后一个参数的长度要包括空字符在内！

.if eax == 0
    invoke MessageBox, hWnd, offset szErrPrompt1, offset szAppName, MB_OK or MB_ICONWARNING
    mov eax,TRUE
    ret
.endif

invoke FpuAtoFL, offset szInputDec, 0, DEST_FPU
fstp r4Data   ; mem32
.if eax == 0
    invoke MessageBox, hWnd, offset szErrPrompt2, offset szAppName, MB_OK or MB_ICONERROR
    mov eax,TRUE
    ret
.endif

invoke FloatToHexText, offset r4Data, offset szInputHex, 8
invoke GetDlgItem, hWnd, IDC_FHEXEDIT
invoke SetWindowText, eax, offset szInputHex
invoke HexToBinText, offset szInputHex, offset szInputBin, 8
lea esi,szInputBin
lea edi,szChar
mov ebx,IDC_FBINEDIT31
mov ecx,32
NextEdit:
    mov al,[esi]
    inc esi
    mov byte ptr[edi],al
    push ecx
    invoke GetDlgItem, hWnd, ebx
    inc ebx
    invoke SetWindowText, eax, offset szChar
    pop ecx
loop NextEdit
.elseif ax == IDC_HEXBUTTON
    invoke GetDlgItem, hWnd, IDC_FHEXEDIT
    invoke GetWindowText, eax, offset szInputHex, 9     ; 千万注意最后一个参数的长度要包括空字符在内！
    invoke HexTextToDec, offset szInputHex, offset r4Data, 8
.if eax == 0
    invoke MessageBox, hWnd, offset szErrPrompt3, offset szAppName, MB_OK or MB_ICONERROR
    mov eax,TRUE
    ret
.endif
    finit
    fld r4Data
    invoke FpuFLtoA, 0, 7, offset szInputDec, SRC1_FPU  ; 对于单精度浮点数最多精确到小数后 7 位
    invoke GetDlgItem, hWnd, IDC_FDECEDIT
    invoke SetWindowText, eax, offset szInputDec
    invoke HexToBinText, offset szInputHex, offset szInputBin, 8
    lea esi,szInputBin
    lea edi,szChar
    mov ebx,IDC_FBINEDIT31
    mov ecx,32
NextEdit1:
    mov al,[esi]
    inc esi
    mov byte ptr[edi],al
    push ecx
    invoke GetDlgItem, hWnd, ebx
    inc ebx
    invoke SetWindowText, eax, offset szChar
    pop ecx
loop NextEdit1
.elseif ax == IDC_BINBUTTON
    lea edi,szInputBin
    lea esi,szChar
    cld
    mov ebx,IDC_FBINEDIT31
    mov ecx,32
NextEdit2:
    push ecx
    invoke GetDlgItem, hWnd, ebx
    inc ebx
    invoke GetWindowText, eax, offset szChar, 2     ; 千万注意最后一个参数的长度要包括空字符在内！
    mov al,[esi]                                  ; store szChar to al
    stosb
    pop ecx
loop NextEdit2
    mov al,0
    mov byte ptr[edi],al                             ; end of string
    invoke BinTextToHex, offset szInputBin, offset szInputHex, 8
.if eax == 0
    invoke MessageBox, hWnd, offset szErrPrompt4, offset szAppName, MB_OK or MB_ICONERROR
    mov eax,TRUE
    ret
.endif
    invoke GetDlgItem, hWnd, IDC_FHEXEDIT
    invoke SetWindowText, eax, offset szInputHex
    invoke HexTextToDec, offset szInputHex, offset r4Data, 8
    finit
    fld r4Data
    invoke FpuFLtoA, 0, 7, offset szInputDec, SRC1_FPU
    invoke GetDlgItem, hWnd, IDC_FDECEDIT
    invoke SetWindowText, eax, offset szInputDec
.elseif ax == IDC_COPYBUTTON
    invoke GlobalAlloc, GHND or GMEM_SHARE, 33
    mov hGlobal,eax
    invoke GlobalLock, hGlobal
    mov edi,eax
    lea esi,szInputBin
    mov ecx,33
    cld
    rep movsb
    invoke GlobalUnlock, hGlobal
    invoke OpenClipboard, hWnd
    invoke EmptyClipboard
    invoke SetClipboardData, CF_TEXT, hGlobal
    invoke CloseClipboard
    ;invoke GlobalFree, hGlobal
.elseif ax == IDC_PASTEBUTTON
    invoke IsClipboardFormatAvailable, CF_TEXT
.if eax == 0
    invoke MessageBox, hWnd, offset szErrPrompt5, offset szAppName, MB_OK or MB_ICONWARNING
    mov eax,TRUE
    ret
.endif
    invoke OpenClipboard, hWnd
    invoke GetClipboardData, CF_TEXT
    mov hGlobal,eax
    invoke GlobalLock, hGlobal
    mov esi,eax
    lea edi,szInputBin
    mov ecx,33
    cld
    rep movsb
    invoke GlobalUnlock, hGlobal
    invoke CloseClipboard
    lea esi,szInputBin
    lea edi,szChar
    mov ebx,IDC_FBINEDIT31
    mov ecx,32
NextEdit3:
    mov al,[esi]
    inc esi
    mov byte ptr[edi],al
    push ecx
    invoke GetDlgItem, hWnd, ebx
    inc ebx
    invoke SetWindowText, eax, offset szChar
    pop ecx
loop NextEdit3
.endif

; .elseif eax == WM_PAINT ;没有找到更好的处理按钮允许的消息，暂用 WM_PAINT 消息
; invoke IsClipboardFormatAvailable, CF_TEXT
; .if eax == 0
; invoke GetDlgItem, hWnd, IDC_PASTEBUTTON
; invoke EnableWindow, eax, FALSE
; .else
; invoke GetDlgItem, hWnd, IDC_PASTEBUTTON
; invoke EnableWindow, eax, TRUE
; .endif
; mov eax,FALSE       ; 让默认消息处理程序再处理，不然会没完没了地发 WM_PAINT 消息，使窗口反应变慢
; ret
.else
    mov eax,FALSE
    ret
.endif

mov eax,TRUE
ret

ProcFloatDlgMain endp

ProcDoubleDlgMain proc uses ebx edi esi hWnd, wMsg, wParam, lParam
    mov eax,wMsg
.if eax == WM_CLOSE
    mov SeleDispDlg, 0
    invoke EndDialog, hWnd, NULL
.elseif eax == WM_INITDIALOG
    invoke LoadIcon, hInstance, IDC_ICO
    invoke SendMessage, hWnd,WM_SETICON, ICON_BIG, eax
    invoke SendDlgItemMessage, hWnd, IDC_DDECEDIT, EM_LIMITTEXT, 29, 0
    invoke SendDlgItemMessage, hWnd, IDC_DHEXEDIT, EM_LIMITTEXT, 16, 0
    ;invoke GetDlgItem, hWnd, IDC_DDECEDIT
    ;invoke SetWindowLong, eax, GWL_WNDPROC, addr ProcDecEdit
    ;mov lpOldProcDecEdit, eax
    invoke GetDlgItem, hWnd, IDC_DHEXEDIT
```



```
            invoke  SetWindowLong, eax, GWL_WNDPROC, addr ProcHexEdit

            mov     lpOldProcHexEdit, eax

        .elseif eax == WM_COMMAND

            mov eax,wParam

            .if     ax == IDCANCEL

                mov     SeleDispDlg, 0

                invoke  EndDialog,hWnd,NULL

            .elseif ax == IDC_FLOATBUTTON

                mov     SeleDispDlg, 1

                invoke  EndDialog,hWnd,NULL

            .elseif ax == IDC_DECBUTTON

                invoke GetDlgItem, hWnd, IDC_DDECEDIT

                invoke GetWindowText, eax, offset szInputDec, 30

                .if eax == 0

                    invoke MessageBox, hWnd, offset szErrPrompt1, offset szAppName, MB_OK or MB_ICONWARNING

                    mov    eax,TRUE

                    ret

                .endif

                invoke FpuAtoFL, offset szInputDec, 0, DEST_FPU

                fstp   r8Data   ; mem64

                .if eax == 0

                    invoke MessageBox, hWnd, offset szErrPrompt2, offset szAppName, MB_OK or MB_ICONERROR

                    mov    eax,TRUE

                    ret

                .endif

                invoke FloatToHexText, offset r8Data, offset szInputHex, 16

                invoke GetDlgItem, hWnd, IDC_DHEXEDIT

                invoke SetWindowText, eax, offset szInputHex

                invoke HexToBinText, offset szInputHex, offset szInputBin, 16

                lea    esi,szInputBin

                lea    edi,szChar

                mov    ebx,IDC_DBINEDIT63

                mov    ecx,64

                NextEdit:

                    mov    al,[esi]

                    inc    esi

                    mov    byte ptr[edi],al

                    push   ecx

                    invoke GetDlgItem, hWnd, ebx

                    inc    ebx

                    invoke SetWindowText, eax, offset szChar

                    pop    ecx

                loop   NextEdit

            .elseif ax == IDC_HEXBUTTON

                invoke GetDlgItem, hWnd, IDC_DHEXEDIT

                invoke GetWindowText, eax, offset szInputHex, 17

                invoke HexTextToDec, offset szInputHex, offset r8Data, 16

                .if    eax == 0

                    invoke MessageBox, hWnd, offset szErrPrompt3, offset szAppName, MB_OK or MB_ICONERROR

                    mov    eax,TRUE

                    ret

                .endif

                finit

                fld    r8Data

                invoke FpuFLtoA, 0, 15, offset szInputDec, SRC1_FPU  ; FpuFLtoA 只能到小数点后 15 位，再长也不会输出了。

                invoke GetDlgItem, hWnd, IDC_DDECEDIT

                invoke SetWindowText, eax, offset szInputDec

                invoke HexToBinText, offset szInputHex, offset szInputBin, 16

                lea    esi,szInputBin

                lea    edi,szChar

                mov    ebx,IDC_DBINEDIT63

                mov    ecx,64

                NextEdit1:

                    mov    al,[esi]

                    inc    esi

                    mov    byte ptr[edi],al

                    push   ecx

                    invoke GetDlgItem, hWnd, ebx

                    inc    ebx

                    invoke SetWindowText, eax, offset szChar

                    pop    ecx

                loop   NextEdit1

            .elseif ax == IDC_BINBUTTON

                lea    edi,szInputBin

                lea    esi,szChar

                cld

                mov    ebx,IDC_DBINEDIT63

                mov    ecx,64

                NextEdit2:

                    push   ecx

                    invoke GetDlgItem, hWnd, ebx

                    inc    ebx

                    invoke GetWindowText, eax, offset szChar, 2     ; 千万注意最后一个参数的长度要包括空字符在内！

                    mov    al,[esi]                                 ; store szChar to al

                    stosb

                    pop    ecx

                loop    NextEdit2

                mov    al,0

                mov    byte ptr[edi],al                             ; end of string

                invoke BinTextToHex, offset szInputBin, offset szInputHex, 16

                .if    eax == 0

                    invoke MessageBox, hWnd, offset szErrPrompt4, offset szAppName, MB_OK or MB_ICONERROR

                    mov    eax,TRUE

                    ret

                .endif

                invoke GetDlgItem, hWnd, IDC_DHEXEDIT

                invoke SetWindowText, eax, offset szInputHex

                invoke HexTextToDec, offset szInputHex, offset r8Data, 16

                finit

                fld     r8Data

                invoke FpuFLtoA, 0, 15, offset szInputDec, SRC1_FPU

                invoke GetDlgItem, hWnd, IDC_DDECEDIT

                invoke SetWindowText, eax, offset szInputDec

            .elseif ax == IDC_COPYBUTTON

                invoke  GlobalAlloc, GHND or GMEM_SHARE, 65

                mov     hGlobal,eax

                invoke GlobalLock, hGlobal

                mov    edi,eax

                lea    esi,szInputBin

                mov    ecx,65

                cld

                rep    movsb

                invoke GlobalUnlock, hGlobal

                invoke OpenClipboard, hWnd

                invoke EmptyClipboard

                invoke SetClipboardData, CF_TEXT, hGlobal

                invoke CloseClipboard

                ;invoke  GlobalFree, hGlobal

            .elseif ax == IDC_PASTEBUTTON

                invoke IsClipboardFormatAvailable, CF_TEXT

                .if eax == 0

                    invoke MessageBox, hWnd, offset szErrPrompt5, offset szAppName, MB_OK or MB_ICONWARNING

                    mov    eax,TRUE

                    ret

                .endif

                invoke OpenClipboard, hWnd

                invoke GetClipboardData, CF_TEXT

                mov    hGlobal,eax

                invoke GlobalLock, hGlobal

                mov    esi,eax

                lea    edi,szInputBin

                mov    ecx,65

                cld

                rep    movsb

                invoke GlobalUnlock, hGlobal

                invoke CloseClipboard

                lea    esi,szInputBin

                lea    edi,szChar

                mov    ebx,IDC_DBINEDIT63

                mov    ecx,64

                NextEdit3:

                    mov    al,[esi]

                    inc    esi

                    mov    byte ptr[edi],al

                    push   ecx

                    invoke GetDlgItem, hWnd, ebx

                    inc    ebx

                    invoke SetWindowText, eax, offset szChar

                    pop    ecx

                loop   NextEdit3

            .endif

        .else

            mov eax,FALSE

            ret

        .endif

        mov eax,TRUE

        ret

ProcDoubleDlgMain   endp

FloatToHexText  proc    uses ebx ecx edx edi esi lpSrc, lpDest, lBitNum

        LOCAL   @SrcSetCount:DWORD

        LOCAL   @DestSetCount:DWORD

        LOCAL   @TempStr[16]:BYTE

        mov     @SrcSetCount,0

        mov     @DestSetCount,0

        mov     esi,lpSrc

        mov     edi,lpDest

        xor     eax,eax

        xor     ebx,ebx

        mov     ecx,lBitNum                 ; lBitNum = 8 or 16
        shl     ecx,1                       ; div 2

        NextByte:
            push    ecx
            mov     ecx,@SrcSetCount
            mov     al,[esi+ecx]
            pop     ecx
            push    ecx
            mov     ecx,2                   ; 2 char by 1 byte
            NextBCD:
```



# 汇编代码

```asm
xor     bl,bl  

sar     al,1  

jnc     @F  

add     bl,1  

@@:  

sar     al,1  

jnc     @F  

add     bl,2  

@@:  

sar     al,1  

jnc     @F  

add     bl,4  

@@:  

sar     al,1  

jnc     @F  

add     bl,8  

@@:  

cmp     bl,10  

jae     @F  

add     bl,48               ; 0-9 Adjust to ASCII  

jmp     lp0  

@@:  

add     bl,55               ; A-F Adjust to ASCII  

lp0:  

push    ecx  

mov     ecx,@DestSetCount  

mov     [edi+ecx],bl        ; store to set 1  

pop     ecx  

inc     @DestSetCount  

loop    NextBCD  

pop     ecx          

inc     @SrcSetCount  

loop    NextByte  

mov     ecx,@DestSetCount        

; swap string set  

mov     esi,lpDest  

lea     edi,@TempStr  

mov     ecx,lBitNum  

mov     ebx,lBitNum  

dec     ebx                         ; Source stream set  

xor     edx,edx  

TempNextChar:  

    mov     al,[esi+ebx]  

    mov     [edi+edx],al  

    dec     ebx  

    inc     edx  

loop    TempNextChar  

lea     esi,@TempStr  

mov     edi,lpDest  

mov     ecx,lBitNum  

cld  

rep     movsb  

mov     al,0  

mov     [edi],al                    ; end of string  

ret      

FloatToHexText  endp  

HexToBinText    proc    uses ecx edi esi lpSrc, lpDest, lBitNum  

    mov     esi,lpSrc  

    mov     edi,lpDest      

    mov     ecx,lBitNum             ; lBitNum = 8 or 16  

NextChar:  

    mov     al,[esi]  

    inc     esi  

    cmp     al,39h  

    jbe     @F  

    sub     al,7  

@@:  

    sub     al,30h  

    sal     al,4  

    push    ecx  

    mov     ecx,4  

NextBit:  

    sal     al,1  

    jc      @F  

    mov     byte ptr[edi],30h  

    jmp     lp0  

@@:  

    mov     byte ptr[edi],31h  

    lp0:  

    inc     edi  

loop    NextBit  

    pop     ecx  

loop    NextChar  

    mov     byte ptr[edi],0                 ; end of string  

    ret  

HexToBinText    endp  

HexTextToDec    proc    uses ecx edi esi lpSrc, lpDest, lBitNum  

    mov     esi,lpSrc      

    mov     ecx,lBitNum      

NextCharChk:              

    mov     al,[esi]         

    .if     (al >= 'a') && (al <= 'f')  

        sub     al,20h  

    .endif  

    .if     al < 'A'  

        .if     (al < '0') || (al > '9')  

            jmp     HexErr  

        .endif  

    .else  

        .if     al > 'F'  

            jmp     HexErr  

        .endif  

    .endif  

    mov     byte ptr[esi],al  

    inc     esi          

loop    NextCharChk      

    mov     esi,lpSrc  

    mov     edi,lpDest  

    add     esi,lBitNum             ; lBitNum = 8 or 16  

    dec     esi  

    xor     eax,eax  

    mov     ecx,lBitNum  

    sar     ecx,1                   ; ecx / 2  

NextByte:  

    mov     al,[esi]  

    dec     esi  

    cmp     al,'9'  

    jbe     @F                  

    sub     al,7  

@@:          

    sub     al,30h  

    mov     ah,al  

    mov     al,[esi]  

    dec     esi  

    cmp     al,'9'  

    jbe     @F  

    sub     al,7  

@@:  

    sub     al,30h  

    sal     al,4  

    add     al,ah  

    mov     byte ptr[edi],al  

    inc     edi  

loop    NextByte  

    mov     eax,1  

    ret  

    HexErr:  

    xor     eax,eax  

    ret  

HexTextToDec endp  

BinTextToHex    proc    uses ecx edi esi lpSrc, lpDest, lBitNum  

    mov     esi,lpSrc  

    mov     edi,lpDest  

    xor     eax,eax  

    mov     ecx,lBitNum             ; lBitNum = 8 or 16  

    cld  

NextHexChar:  

    lodsb  

    cmp     al,'0'  

    jz      @F  

    cmp     al,'1'  

    jnz     BinErr  

    add     ah,8  

@@:  

    lodsb  

    cmp     al,'0'  

    jz      @F  

    cmp     al,'1'  

    jnz     BinErr  

    add     ah,4  

@@:  

    lodsb  

    cmp     al,'0'  

    jz      @F  

    cmp     al,'1'  

    jnz     BinErr  

    add     ah,2  

@@:  

    lodsb  

    cmp     al,'0'  

    jz      @F  

    cmp     al,'1'  

    jnz     BinErr  

    add     ah,1  

@@:  

    cmp     ah,9  

    jbe     @F  

    add     ah,7  

@@:  

    add     ah,30h  

    mov     byte ptr[edi],ah  

    inc     edi  

    xor     ah,ah  

loop    NextHexChar  

    mov     ah,0  

    mov     byte ptr[edi],ah        ; end of string  

    mov     eax,1                   ; return TRUE  

    ret  

    BinErr:  

    xor     eax,eax  

    ret  

BinTextToHex endp  

ProcDecEdit     proc    uses ebx edi esi hWnd, wMsg, wParam, lParam  

    mov     eax, wMsg  

    .if     wMsg == WM_CHAR  

            mov     eax, wParam  

            lea     edi,szAllowDecChar  

            mov     ecx, sizeof szAllowDecChar  

            repnz   scasb  

            .if     ZERO?   ; 只有允许的字符才发送消息给默认对话框处理过程  

                invoke  CallWindowProc, lpOldProcDecEdit, hWnd, wMsg, eax, lParam  

                ret  

            .endif  

    .else  

            invoke  CallWindowProc, lpOldProcDecEdit, hWnd, wMsg, wParam, lParam  

            ret  

    .endif  

    xor     eax,eax         ; 未处理消息  

    ret  

ProcDecEdit endp  

ProcHexEdit     proc    uses ebx edi esi hWnd, wMsg, wParam, lParam  

    mov     eax, wMsg  

    .if     wMsg == WM_CHAR  

            mov     eax, wParam  

            lea     edi,szAllowHexChar  

            mov     ecx, sizeof szAllowHexChar  

            repnz   scasb  

            .if     ZERO?   ; 只有允许的字符才发送消息给默认对话框处理过程  

                .if     al > '9'  

                    and     al, not 20h     ; 将 a-f 转换为大写  

                .endif  

                invoke  CallWindowProc, lpOldProcHexEdit, hWnd, wMsg, eax, lParam  

                ret  

            .endif  

    .else  

            invoke  CallWindowProc, lpOldProcHexEdit, hWnd, wMsg, wParam, lParam  

            ret  

    .endif  

    xor     eax,eax         ; 未处理消息  

    ret  

ProcHexEdit endp  

end start  

```

## 资料文件

```c
#include        <resource.h>

//***************************************************************************

#define IDD_FLOATCONDLG                 1000  

#define IDD_DOUBLECONDLG                1001
```



``` 
#define IDC_ICO 1104

//-----------------------------------------------------------------------------
#define IDC_COPYBUTTON 1105
#define IDC_PASTEBUTTON 1106
#define IDC_DECBUTTON 1
#define IDC_EXITBUTTON 2
#define IDC_HEXBUTTON 3
#define IDC_BINBUTTON 4
#define IDC_DOUBLEBUTTON 1002
#define IDC_FLOATBUTTON 1003
//-----------------------------------------------------------------------------

#define IDC_DDECEDIT 1004
#define IDC_DHEXEDIT 1005
#define IDC_DBINEDIT63 1006
#define IDC_DBINEDIT62 1007
#define IDC_DBINEDIT61 1008
#define IDC_DBINEDIT60 1009
#define IDC_DBINEDIT59 1010
#define IDC_DBINEDIT58 1011
#define IDC_DBINEDIT57 1012
#define IDC_DBINEDIT56 1013
#define IDC_DBINEDIT55 1014
#define IDC_DBINEDIT54 1015
#define IDC_DBINEDIT53 1016
#define IDC_DBINEDIT52 1017
#define IDC_DBINEDIT51 1018
#define IDC_DBINEDIT50 1019
#define IDC_DBINEDIT49 1020
#define IDC_DBINEDIT48 1021
#define IDC_DBINEDIT47 1022
#define IDC_DBINEDIT46 1023
#define IDC_DBINEDIT45 1024
#define IDC_DBINEDIT44 1025
#define IDC_DBINEDIT43 1026
#define IDC_DBINEDIT42 1027
#define IDC_DBINEDIT41 1028
#define IDC_DBINEDIT40 1029
#define IDC_DBINEDIT39 1030
#define IDC_DBINEDIT38 1031
#define IDC_DBINEDIT37 1032
#define IDC_DBINEDIT36 1033
#define IDC_DBINEDIT35 1034
#define IDC_DBINEDIT34 1035
#define IDC_DBINEDIT33 1036
#define IDC_DBINEDIT32 1037
#define IDC_DBINEDIT31 1038
#define IDC_DBINEDIT30 1039
#define IDC_DBINEDIT29 1040
#define IDC_DBINEDIT28 1041
#define IDC_DBINEDIT27 1042
#define IDC_DBINEDIT26 1043
#define IDC_DBINEDIT25 1044
#define IDC_DBINEDIT24 1045
#define IDC_DBINEDIT23 1046
#define IDC_DBINEDIT22 1047
#define IDC_DBINEDIT21 1048
#define IDC_DBINEDIT20 1049
#define IDC_DBINEDIT19 1050
#define IDC_DBINEDIT18 1051
#define IDC_DBINEDIT17 1052
#define IDC_DBINEDIT16 1053
#define IDC_DBINEDIT15 1054
#define IDC_DBINEDIT14 1055
#define IDC_DBINEDIT13 1056
#define IDC_DBINEDIT12 1057
#define IDC_DBINEDIT11 1058
#define IDC_DBINEDIT10 1059
#define IDC_DBINEDIT09 1060
#define IDC_DBINEDIT08 1061
#define IDC_DBINEDIT07 1062
#define IDC_DBINEDIT06 1063
#define IDC_DBINEDIT05 1064
#define IDC_DBINEDIT04 1065
#define IDC_DBINEDIT03 1066
#define IDC_DBINEDIT02 1067
#define IDC_DBINEDIT01 1068
#define IDC_DBINEDIT00 1069
//-----------------------------------------------------------------------------

#define IDC_FDECEDIT 1070
#define IDC_FHEXEDIT 1071
#define IDC_FBINEDIT31 1072
#define IDC_FBINEDIT30 1073
#define IDC_FBINEDIT29 1074
#define IDC_FBINEDIT28 1075
#define IDC_FBINEDIT27 1076
#define IDC_FBINEDIT26 1077
#define IDC_FBINEDIT25 1078
#define IDC_FBINEDIT24 1079
#define IDC_FBINEDIT23 1080
#define IDC_FBINEDIT22 1081
#define IDC_FBINEDIT21 1082
#define IDC_FBINEDIT20 1083
#define IDC_FBINEDIT19 1084
#define IDC_FBINEDIT18 1085
#define IDC_FBINEDIT17 1086
#define IDC_FBINEDIT16 1087
#define IDC_FBINEDIT15 1088
#define IDC_FBINEDIT14 1089
#define IDC_FBINEDIT13 1090
#define IDC_FBINEDIT12 1091
#define IDC_FBINEDIT11 1092
#define IDC_FBINEDIT10 1093
#define IDC_FBINEDIT09 1094
#define IDC_FBINEDIT08 1095
#define IDC_FBINEDIT07 1096
#define IDC_FBINEDIT06 1097
#define IDC_FBINEDIT05 1098
#define IDC_FBINEDIT04 1099
#define IDC_FBINEDIT03 1100
#define IDC_FBINEDIT02 1101
#define IDC_FBINEDIT01 1102
#define IDC_FBINEDIT00 1103

********************************************************************************
IDC_ICO ICON "Main.ico"
1 24 "FloatConvert.manifest"
********************************************************************************

IDD_FLOATCONDLG DIALOGEX 120, 180, 409, 146
STYLE DS_SETFONT | DS_MODALFRAME | WS_MINIMIZEBOX | WS_POPUP | WS_CAPTION | WS_SYSMENU
CAPTION "单精度浮点数转换"
FONT 9, "宋体", 400, 0, 0x86
BEGIN
    EDITTEXT IDC_FDECEDIT,98,57,73,12,ES_AUTOHSCROLL
    EDITTEXT IDC_FHEXEDIT,98,75,73,12,ES_AUTOHSCROLL
    EDITTEXT IDC_FBINEDIT31,12,105,9,12,ES_CENTER | ES_AUTOHSCROLL
    EDITTEXT IDC_FBINEDIT30,33,105,9,12,ES_CENTER | ES_AUTOHSCROLL
    EDITTEXT IDC_FBINEDIT29,45,105,9,12,ES_CENTER | ES_AUTOHSCROLL
    EDITTEXT IDC_FBINEDIT28,57,105,9,12,ES_CENTER | ES_AUTOHSCROLL
    EDITTEXT IDC_FBINEDIT27,69,105,9,12,ES_CENTER | ES_AUTOHSCROLL
    EDITTEXT IDC_FBINEDIT26,81,105,9,12,ES_CENTER | ES_AUTOHSCROLL
    EDITTEXT IDC_FBINEDIT25,93,105,9,12,ES_CENTER | ES_AUTOHSCROLL
    EDITTEXT IDC_FBINEDIT24,105,105,9,12,ES_CENTER | ES_AUTOHSCROLL
    EDITTEXT IDC_FBINEDIT23,117,105,9,12,ES_CENTER | ES_AUTOHSCROLL
    EDITTEXT IDC_FBINEDIT22,129,105,9,12,ES_CENTER | ES_AUTOHSCROLL
    EDITTEXT IDC_FBINEDIT21,141,105,9,12,ES_CENTER | ES_AUTOHSCROLL
    EDITTEXT IDC_FBINEDIT20,153,105,9,12,ES_CENTER | ES_AUTOHSCROLL
    EDITTEXT IDC_FBINEDIT19,165,105,9,12,ES_CENTER | ES_AUTOHSCROLL
    EDITTEXT IDC_FBINEDIT18,177,105,9,12,ES_CENTER | ES_AUTOHSCROLL
    EDITTEXT IDC_FBINEDIT17,189,105,9,12,ES_CENTER | ES_AUTOHSCROLL
    EDITTEXT IDC_FBINEDIT16,201,105,9,12,ES_CENTER | ES_AUTOHSCROLL
    EDITTEXT IDC_FBINEDIT15,213,105,9,12,ES_CENTER | ES_AUTOHSCROLL
    EDITTEXT IDC_FBINEDIT14,225,105,9,12,ES_CENTER | ES_AUTOHSCROLL
    EDITTEXT IDC_FBINEDIT13,237,105,9,12,ES_CENTER | ES_AUTOHSCROLL
    EDITTEXT IDC_FBINEDIT12,249,105,9,12,ES_CENTER | ES_AUTOHSCROLL
```



```
    EDITTEXT        IDC\_FBINEDIT11,261,105,9,12,ES\_CENTER  |  ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_FBINEDIT10,273,105,9,12,ES\_CENTER  |  ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_FBINEDIT09,285,105,9,12,ES\_CENTER  |  ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_FBINEDIT08,297,105,9,12,ES\_CENTER  |  ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_FBINEDIT07,309,105,9,12,ES\_CENTER  |  ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_FBINEDIT06,321,105,9,12,ES\_CENTER  |  ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_FBINEDIT05,333,105,9,12,ES\_CENTER  |  ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_FBINEDIT04,345,105,9,12,ES\_CENTER  |  ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_FBINEDIT03,357,105,9,12,ES\_CENTER  |  ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_FBINEDIT02,369,105,9,12,ES\_CENTER  |  ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_FBINEDIT01,381,105,9,12,ES\_CENTER  |  ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_FBINEDIT00,393,105,9,12,ES\_CENTER  |  ES\_AUTOHSCROLL  

    DEFPUSHBUTTON   "由十进制转换(&D)",IDC\_DECBUTTON,183,18,78,18  

    PUSHBUTTON   "由十六进制转换(&H)",IDC\_HEXBUTTON,183,44,78,18  

    PUSHBUTTON   "由二进制转换(&B)",IDC\_BINBUTTON,183,69,78,18  

    PUSHBUTTON      "复制到剪贴板(&C)",IDC\_COPYBUTTON,297,18,78,18  

    PUSHBUTTON      "从剪贴板粘贴(&V)",IDC\_PASTEBUTTON,297,44,78,18  

    PUSHBUTTON      "退出(&X)",IDC\_EXITBUTTON,297,69,78,18  

    PUSHBUTTON   "双精度浮点数转换→(&S)",IDC\_DOUBLEBUTTON,18,18,126,27  

    LTEXT           "Greatboy. All Rights Reserved",IDC\_STATIC,290,137,120,8  

    LTEXT           "十进制数",IDC\_STATIC,62,59,33,8  

    LTEXT           "符号位",IDC\_STATIC,4,93,24,8  

    LTEXT           "指数位",IDC\_STATIC,68,93,24,8  

    LTEXT           "尾数位",IDC\_STATIC,254,93,24,8  

    LTEXT           "十六进制数",IDC\_STATIC,54,77,42,8  

    LTEXT           "二进制数",IDC\_STATIC,3,78,36,8  

    CONTROL         "",IDC\_STATIC,"Static",SS\_ETCHEDHORZ,33,96,36,1  

    CONTROL         "",IDC\_STATIC,"Static",SS\_ETCHEDHORZ,90,96,39,1  

    LTEXT           "31   30 29 28 27 26 25 24 23 22 21 20 19 18 17 16 15 14 13 12 11 10 09 08 07 06 05 04 03 02 01 00",  
                    IDC\_STATIC,14,123,390,8  

    CONTROL         "",IDC\_STATIC,"Static",SS\_ETCHEDHORZ,129,96,123,1  

    CONTROL         "",IDC\_STATIC,"Static",SS\_ETCHEDHORZ,279,96,123,1  

    CONTROL         "",IDC\_STATIC,"Static",SS\_ETCHEDVERT,128,91,1,12  

    CONTROL         "",IDC\_STATIC,"Static",SS\_ETCHEDVERT,402,92,1,11  

    CONTROL         "",IDC\_STATIC,"Static",SS\_ETCHEDVERT,33,92,1,11  

END  

//-----------------------------------------------------------------------------  

IDD\_DOUBLECONDLG DIALOGEX 120, 180, 409, 186  

STYLE DS\_SETFONT | DS\_MODALFRAME | WS\_MINIMIZEBOX | WS\_POPUP | WS\_CAPTION | WS\_SYSMENU  

CAPTION "双精度浮点数转换"  

FONT 9, "宋体", 400, 0, 0x86  

BEGIN  

    EDITTEXT        IDC\_DDECEDIT,98,57,73,12,ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DHEXEDIT,98,75,73,12,ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT63,12,105,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT62,33,105,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT61,45,105,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT60,57,105,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT59,69,105,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT58,81,105,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT57,93,105,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT56,105,105,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT55,117,105,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT54,129,105,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT53,141,105,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT52,153,105,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT51,165,105,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT50,177,105,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT49,189,105,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT48,201,105,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT47,213,105,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT46,225,105,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT45,237,105,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT44,249,105,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT43,261,105,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT42,273,105,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT41,285,105,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT40,297,105,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT39,309,105,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT38,321,105,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT37,333,105,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT36,345,105,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT35,357,105,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT34,369,105,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT33,381,105,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT32,393,105,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT31,21,147,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT30,33,147,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT29,45,147,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT28,57,147,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT27,69,147,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT26,81,147,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT25,93,147,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT24,105,147,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT23,117,147,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT22,129,147,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT21,141,147,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT20,153,147,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT19,165,147,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT18,177,147,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT17,189,147,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT16,201,147,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT15,213,147,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT14,225,147,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT13,237,147,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT12,249,147,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT11,261,147,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT10,273,147,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT09,285,147,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT08,297,147,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT07,309,147,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT06,321,147,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT05,333,147,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT04,345,147,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT03,357,147,9,12,ES\_CENTER | ES\_AUTOHSCROLL  

    EDITTEXT        IDC\_DBINEDIT02,369,147,9,12,ES\_CENTER | ES\_AUTOHSCROLL  
```



```
EDITTEXT        IDC_DBINEDIT01,381,147,9,12,ES_CENTER | ES_AUTOHSCROLL
EDITTEXT        IDC_DBINEDIT00,393,147,9,12,ES_CENTER | ES_AUTOHSCROLL
DEFPUSHBUTTON   "由十进制转换(&D)",IDC_DECBUTTON,183,18,78,18
PUSHBUTTON      "由十六进制转换(&H)",IDC_HEXBUTTON,183,44,78,18
PUSHBUTTON      "由二进制转换(&B)",IDC_BINBUTTON,183,69,78,18
PUSHBUTTON      "复制到剪贴板(&C)",IDC_COPYBUTTON,297,18,78,18
PUSHBUTTON      "从剪贴板粘贴(&V)",IDC_PASTEBUTTON,297,44,78,18
PUSHBUTTON      "退出(&X)",IDC_EXITBUTTON,297,69,78,18
PUSHBUTTON      "单精度浮点数转换→(&S)",IDC_FLOATBUTTON,18,18,126,27
LTEXT           "Greatboy. All Rights Reserved",IDC_STATIC,290,177,120,8
LTEXT           "十进制数",IDC_STATIC,62,59,33,8
LTEXT           "符号位",IDC_STATIC,4,93,24,8
LTEXT           "指数位",IDC_STATIC,86,93,24,8
LTEXT           "尾数位",IDC_STATIC,272,93,24,8
LTEXT           "十六进制数",IDC_STATIC,54,77,42,8
LTEXT           "二进制数",IDC_STATIC,3,78,36,8
CONTROL         "",IDC_STATIC,"Static",SS_ETCHEDHORZ,33,96,51,1
LTEXT           "63 62 61 60 59 58 57 56 55 54 53 52 51 50 49 48 47 46 45 44 43 42 41 40 39 38 37 36 35 34 33 32",IDC_STATIC,14,123,390,8
CONTROL         "",IDC_STATIC,"Static",SS_ETCHEDHORZ,111,96,159,1
CONTROL         "",IDC_STATIC,"Static",SS_ETCHEDHORZ,297,96,105,1
CONTROL         "",IDC_STATIC,"Static",SS_ETCHEDVERT,163,91,1,12
CONTROL         "",IDC_STATIC,"Static",SS_ETCHEDVERT,402,92,1,11
CONTROL         "",IDC_STATIC,"Static",SS_ETCHEDVERT,33,92,1,11
LTEXT           "尾数位",IDC_STATIC,200,135,24,8
LTEXT           "31 30 29 28 27 26 25 24 23 22 21 20 19 18 17 16 15 14 13 12 11 10 09 08 07 06 05 04 03 02 01 00",IDC_STATIC,23,164,382,8
CONTROL         "",IDC_STATIC,"Static",SS_ETCHEDHORZ,21,137,177,1
CONTROL         "",IDC_STATIC,"Static",SS_ETCHEDHORZ,225,137,177,1
CONTROL         "",IDC_STATIC,"Static",SS_ETCHEDVERT,402,133,1,10
CONTROL         "",IDC_STATIC,"Static",SS_ETCHEDVERT,21,133,1,10
END
4dd466d0e1e0cc123afbe504da2462ca
```
