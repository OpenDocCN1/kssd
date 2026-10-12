
```
invoke Block_Encrypt_Process

popad

ret

ENDIF

invoke MD5HASH_Process, ADDR StringBuffer, 64,
        ADDR MD5HASH_RESULT

invoke HEX2ASCII, ADDR MD5HASH_RESULT, 16, ADDR MD5_String

invoke GetDlgItemText,hWnd,IDC_EDIT_REGISTRATION_NUMBER,
        ADDR StringBuffer,33      ;32+1 registration number=32,0 33 bytes

mov esi, offset StringBuffer

mov edi, offset MD5_String

mov eax, dword ptr [esi]

mov ebx, dword ptr [edi]

xor eax,ebx

jnz GetErrorBox

mov eax, dword ptr [esi+4]

mov ebx, dword ptr [edi+4]

xor eax,ebx

jnz GetErrorBox

jmp EndProc

GetErrorBox:

invoke MessageBox,hWnd,ADDR Error_Input_String,
        ADDR Error_Name,MB_OK or MB_TASKMODAL

Reset_Dlg:

invoke SetDlgItemText,hWnd,IDC_EDIT_USER_NAME,NULL

invoke SetDlgItemText,hWnd,IDC_EDIT_EMAIL,NULL

invoke SetDlgItemText,hWnd,IDC_EDIT_REGISTRATION_NUMBER,NULL

invoke GetDlgItem,hWnd,IDC_EDIT_USER_NAME

invoke SetFocus,eax

mov ecx,StringBufferSize

mov esi,offset Dialog_Main_Welcome_String

mov edi,offset StringBuffer

rep movsb

popad

mov eax,1   ; exit process

jmp Finish

EndProc:

invoke Decryption_Word_Process

invoke Judge_Process, hWnd, ADDR Success_Info1, Success_Info_Length, ADDR Decryption_Word1, ADDR Success_Info1

cmp eax,1

jz FinishProc

invoke Judge_Process, hWnd, ADDR Success_Info2, Success_Info_Length, ADDR Decryption_Word1, ADDR Success_Info2

cmp eax,1

jz FinishProc

invoke Judge_Process, hWnd, ADDR Success_Info3, Success_Info_Length, ADDR Decryption_Word1, ADDR Success_Info3

cmp eax,1

jz FinishProc

invoke Judge_Process, hWnd, ADDR Success_Info4, Success_Info_Length, ADDR Decryption_Word1, ADDR Success_Info4

FinishProc:

jmp Reset_Dlg

Finish:

nop

.ELSEIF     ax==IDC_BUTTON_CLEAR

mov ecx,32

mov esi,offset Dialog_Main_Welcome_String

mov edi,offset StringBuffer

rep stosb

invoke SetDlgItemText,hWnd,IDC_EDIT_USER_NAME,NULL

invoke SetDlgItemText,hWnd,IDC_EDIT_EMAIL,NULL

invoke SetDlgItemText,hWnd,IDC_EDIT_REGISTRATION_NUMBER,NULL

.ELSEIF     ax==IDC_BUTTON_EXIT

invoke SendMessage,NULL,WM_COMMAND,
        IDC_BUTTON_EXIT,0

invoke ExitProcess,eax

.ELSEIF     ax==IDC_BUTTON_ABOUT

invoke SendMessage,hWnd,WM_COMMAND,IDC_BUTTON_ABOUT,0

invoke DialogBoxParam,hInstance,IDD_DIALOG_ABOUT,NULL,ADDR Dialog_About_Process,NULL

.ENDIF

.ENDIF

.ENDIF

.ELSE

mov eax,FALSE

ret

.ENDIF

mov eax,TRUE

ret

Dialog_Process endp

;************************************************************************
;* ABOUT DIALOG                                        *
;************************************************************************

Dialog_About_Process proc hWnd :DWORD,uMsg:DWORD,wParam:DWORD,lParam:DWORD

LOCAL hdc:HDC

LOCAL ps:PAINTSTRUCT

LOCAL hfont:HFONT

LOCAL rect:RECT

.IF uMsg==WM_INITDIALOG

invoke GetDlgItem,hWnd,IDC_BUTTON_ABOUT_OK

invoke SetFocus,eax

.ELSEIF uMsg==WM_PAINT

invoke BeginPaint,hWnd,ADDR ps

mov hdc,eax

invoke SelectObject,hdc,eax

mov hfont,eax

RGB 63,118,192

invoke SetTextColor,hdc,eax

RGB 233,233,216

invoke SetBkColor,hdc,eax

invoke SetRect,ADDR rect,75,15,400,300

invoke DrawText,hdc,ADDR Dialog_Main_Welcome_String,-1,ADDR rect,
            DT_LEFT or DT_WORDBREAK

invoke EndPaint,hWnd,ADDR ps

.ELSEIF uMsg==WM_COMMAND

invoke EndDialog,hWnd,NULL

.ENDIF

xor eax,eax

ret

Dialog_About_Process endp

;************************************************************************
; this proccess is a conversion of all hex                            *
; to ascii and display it                              *
; al is needed to get the hex                              *
;                                                       *
; input buffer  hex value                              *
; output buffer hex value ascii                         *
;************************************************************************

HEX2ASCII proc IptBuffer:dword,IptBufferLength:dword,OptBuffer:dword

LOCAL LOOP_COUNTER:DWORD

mov esi,IptBuffer

mov edi,OptBuffer

mov eax,IptBufferLength

mov LOOP_COUNTER,eax

LOOP_HEX2ASC:

mov al,byte ptr [esi]

mov Hexctr,al

mov ecx,04

mov ah,0

shr ax,cl

lea ebx,Xlatab

xlat

mov Charx,al

mov al,byte ptr Hexctr

shl ax,cl

shr al,cl

xlat

mov Charx+1,al

mov ax,word ptr Charx

mov word ptr [edi],ax

add edi,2

add esi,1

mov eax,LOOP_COUNTER

dec eax

jz EXIT_HEX2ASC

mov LOOP_COUNTER,eax

jmp LOOP_HEX2ASC

EXIT_HEX2ASC:

ret

HEX2ASCII endp

;************************************************************************
; procMD5hash : hashes a string using the md5 algorithm

; input :
;
```
