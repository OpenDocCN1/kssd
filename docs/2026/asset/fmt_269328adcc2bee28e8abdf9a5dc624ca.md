
```
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
    invoke FpuFLtoA, 0, 15, offset szInputDec, SRC1_FPU ; FpuFLtoA只能到小数点后15位，再长也不会输出了。
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
```
