
```
Xlatab          db 30h,31h,32h,33h,34h,35h,36h,37h,38h,39h
                db 41h,42h,43h,44h,45h,46h
                db "What"
Charx           db 0,0
                db "^^^"
Current_Time SYSTEMTIME <>  ; Current   "
                db "Time"

; Tiny Encryption Algorithm data area
; keep 093eah and replace other 3 word with
; 3 decryption word 4-2  deef 2212 efa8
;
Decryption_Block_Password equ Tea_Data
Decryption_Block_Cipher   equ Tea_Key
;correct values : 0d33fh, 02212h, 0efa8h, 007afh... address:00404a64
Tea_Key       dw 04efdh, 03216h, 09adfh, 007afh
Tea_Data      dw 00e54h, 02acch, 032afh, 049cdh     ;4 words changeless values

IF INITIALIZE_ENCRYPTION_INFORMATION
; original calues:  0D33FH, 02212H, 0EFA8H, 007AFH
;                   00E54H, 02ACCH, 032AFH, 049CDH
;
; initial encrypted values:
;                  0d33fh, 02212h, 0efa8h, 007afh,
;           089DEh, 0A91Ah, 00939h, 0F3A9h
Tea_Encrypt_Key  dw 0d33fh,02212h,0efa8h,007afh
Tea_Encrypt_Data dw 00e54h,02acch,032afh,049cdh
ENDIF

           dw 0acbeh                ;chaos codes
           dw 043a9h,0cfeah            ;chaos codes
           db 013h,0a6h,0b3h,08eh          ;chaos codes

sum     equ     eax
y       equ     ebx
z       equ     ecx
delta   equ     edx
rounds  equ     di
t       equ     ebp
v0      equ     dword ptr [edi]
v1      equ     dword ptr [edi+4]
k0      equ     dword ptr [esi]
k1      equ     dword ptr [esi+4]
k2      equ     dword ptr [esi+8]
k3      equ     dword ptr [esi+12]

;*************************************************************************
; Uninitialized data                                                     *
;*************************************************************************
.data?

hInstance      HINSTANCE ?
CommandLine    LPSTR ?
szCharCount    db ?
MD5HASH_RESULT dd ?,?,?,?
               db ?,?,?,?

;*************************************************************************
.code

start:
  invoke GetModuleHandle, NULL
  mov hInstance,eax
  invoke DialogBoxParam,hInstance,IDD_DIALOG_MAIN,NULL,ADDR Dialog_Process,NULL
  invoke ExitProcess,eax

;*************************************************************************
;* MAIN DIALOG                                                          *
;*************************************************************************
Dialog_Process proc hWnd:HWND, uMsg:UINT, wParam:WPARAM, lParam:LPARAM
      LOCAL hdc:HDC
      LOCAL ps:PAINTSTRUCT
      LOCAL hfont:HFONT
    .IF uMsg==WM_INITDIALOG
            invoke GetDlgItem,hWnd,IDC_EDIT_USER_NAME
            invoke SetFocus,eax
            invoke GetMenu,hWnd
      .ELSEIF uMsg==WM_PAINT
            invoke BeginPaint,hWnd,ADDR ps
            mov hdc,eax
            invoke CreateFont,14,9,0,0,100,0,0,0,OEM_CHARSET,
                   OUT_DEFAULT_PRECIS,CLIP_DEFAULT_PRECIS,PROOF_QUALITY,
                   VARIABLE_PITCH OR FF_ROMAN,ADDR FontNameSmall
            invoke SelectObject,hdc,eax
            mov hfont,eax
;            RGB 4,50,232
;            invoke SetTextColor,hdc,eax
;            RGB 233,233,216
;            invoke SetBkColor,hdc,eax
;            invoke TextOut,hdc,135,136,ADDR TitleText,SIZEOF TitleText
            invoke SelectObject,hdc,hfont
      .ELSEIF uMsg==WM_CLOSE
    invoke SendMessage,NULL,WM_COMMAND,IDC_BUTTON_EXIT,0
    invoke ExitProcess,eax
      .ELSEIF uMsg==WM_COMMAND
    mov eax,wParam
    .IF lParam==0
    .ELSE
             mov edx,wParam
       shr edx,16
            Check:
      .IF dx==BN_CLICKED
                        .IF ax==IDC_BUTTON_SUBMIT
                              pushad
                              mov ecx,StringBufferSize
                              mov esi,offset Dialog_Main_Welcome_String
                              mov edi,offset StringBuffer
                              rep movsb
                      invoke GetDlgItemText,hWnd,IDC_EDIT_USER_NAME,
                                     ADDR StringBuffer,32
                              cmp eax,sizeOf Hello_String
                              mov szCharCount,al
                              jb  GetErrorBox
                              mov edi,offset StringBuffer
                              add edi,eax
                              mov ecx,32
                              sub ecx,eax
                              mov al,0f6h
                              rep stosb
                     StartProc:
                     ;initialize once original info with default encryption word
                     IF INITIALIZE_ENCRYPTION_INFORMATION
                              invoke Encryption_Process, ADDR Show_Info1, Success_Info_Length, ADDR FUNCTION_1_ENCRYPT_VALUE_1_ADDRESS, ADDR Show_Info1
                              invoke Encryption_Process, ADDR Show_Info2, Success_Info_Length, ADDR FUNCTION_1_ENCRYPT_VALUE_2_ADDRESS, ADDR Show_Info2
                              invoke Encryption_Process, ADDR Show_Info3, Success_Info_Length, ADDR FUNCTION_1_ENCRYPT_VALUE_3_ADDRESS, ADDR Show_Info3
                              invoke Encryption_Process, ADDR Show_Info4, Success_Info_Length, ADDR FUNCTION_1_ENCRYPT_VALUE_4_ADDRESS, ADDR Show_Info4
                              invoke Tea_Encrypt_Process
                              invoke Block_Encrypt_Process
                              popad
                              ret
                     ENDIF
                              invoke MD5HASH_Process,ADDR StringBuffer,64,
                                     ADDR MD5HASH_RESULT
                              invoke HEX2ASCII, ADDR MD5HASH_RESULT, 16, ADDR MD5_String
                      invoke GetDlgItemText,hWnd,IDC_EDIT_REGISTRATION_NUMBER,
                                     ADDR StringBuffer,33       ;32+1 registration number=32,0 33 bytes
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
                              mov eax,1    ; exit process
                              jmp Finish
                      EndProc:
                              invoke Decryption_Word_Process
```
