
## Phase IV & Results

```asm
add [esi].dtB, eax
mov eax, dtc
add [esi].dtC, eax
mov eax, dtd
add [esi].dtD, eax
add edi, 64
sub edx, 64
jnz hashloop

; phase IV & results
mov ecx, 4

R5:
mov eax, dword ptr [esi]
xchg al, ah
rol eax, 16
xchg al, ah
mov dword ptr [esi], eax
add esi, 4
loop R5
ret

MD5HASH_Process endp
```

---

## Decryption Word Process

```asm
; decrypt function encryption word
; change 4 function encryption word user inputted from asc to hex
Decryption_Word_Process proc
    mov cx, 4
    mov esi, offset StringBuffer+16+2
    mov edi, offset Decryption_Word1+1
    call DW_1

    mov esi, offset StringBuffer+16
    mov edi, offset Decryption_Word1
    call DW_1

    mov esi, offset StringBuffer+16+6
    mov edi, offset Decryption_Word1+3
    call DW_1

    mov esi, offset StringBuffer+16+4
    mov edi, offset Decryption_Word1+2
    call DW_1

    mov esi, offset StringBuffer+16+10
    mov edi, offset Decryption_Word1+5
    call DW_1

    mov esi, offset StringBuffer+16+8
    mov edi, offset Decryption_Word1+4
    call DW_1

    mov esi, offset StringBuffer+16+14
    mov edi, offset Decryption_Word1+7
    call DW_1

    mov esi, offset StringBuffer+16+12
    mov edi, offset Decryption_Word1+6
    call DW_1

    mov esi, offset MD5HASH_RESULT+8
    mov edi, offset Decryption_Word1
    mov ax, word ptr [esi]
    xor word ptr [edi], ax
    mov ax, word ptr [edi]
    xchg ah, al
    mov word ptr [edi], ax

    mov edi, offset Decryption_Word2
    mov ax, word ptr [esi+2]
    xor word ptr [edi], ax
    mov ax, word ptr [edi]
    xchg ah, al
    mov word ptr [edi], ax

    mov edi, offset Decryption_Word3
    mov ax, word ptr [esi+4]
    xor word ptr [edi], ax
    mov ax, word ptr [edi]
    xchg ah, al
    mov word ptr [edi], ax

    mov edi, offset Decryption_Word4
    mov ax, word ptr [esi+6]
    xor word ptr [edi], ax
    mov ax, word ptr [edi]
    xchg ah, al
    mov word ptr [edi], ax

    jmp DW_END

DW_1: ; ASCII to HEX : convert 2 bytes ASCII to ax register WORD
    xor bx, bx
    mov al, byte ptr [esi]
    sub al, 30h ; obtain ASCII 0-9
    cmp al, 10h
    jb DW_2
    sub al, 07h ; obtain ASCII A-F

DW_2:
    mov bl, al
    shl bl, cl
    mov al, byte ptr [esi+1]
    sub al, 30h
    cmp al, 10h
    jb DW_3
    sub al, 07h ; obtain ASCII A-F

DW_3:
    or bl, al
    mov byte ptr [edi], bl
    ret

DW_END:

; put 3 decryption word to key for TEA decryption
; 3 decryption is d33f 2212 efa8 ==> Tea_Key
    mov edi, offset Tea_Key
    mov esi, offset Decryption_Word4
    mov ax, word ptr [esi]
    mov word ptr [edi], ax
    mov ax, word ptr [esi-2]
    mov word ptr [edi+2], ax
    mov ax, word ptr [esi-4]
    mov word ptr [edi+4], ax
    ret

Decryption_Word_Process endp
```

---

## Decryption Process

```asm
; Decrypting Process
; Loop decrypt object input buffer length must be a multiple of 8
Decryption_Process Proc IptBuffer:dword, IptBufferLength:dword, Decryption_Word_Address:dword, OptBuffer:dword
    Local DECRYPT_WORD:dword
    MOV ESI, Decryption_Word_Address
    MOV AX, WORD PTR [ESI]
    MOV WORD PTR DECRYPT_WORD, AX
    MOV ESI, IptBuffer
    MOV EDI, OptBuffer

Decryption_P1:
    CLD
    LODSW
    PUSH AX
    LODSW
    MOV BX, AX
    LODSW
    MOV CX, AX
    LODSW
    MOV DX, AX
    POP AX

    ; decrypt codes process
    ; annotated instructions are used for encrypt process
    ; XOR AX, WORD PTR DECRYPT_WORD
    ; XOR BX, WORD PTR DECRYPT_WORD
    ; XOR CX, WORD PTR DECRYPT_WORD
    ; XOR DX, WORD PTR DECRYPT_WORD
    ; XCHG AH, DL
    ; XCHG AL, DH
    ; XCHG BH, CL
    ; XCHG BL, CH

    XCHG AH, DL
    XCHG AL, DH
    XCHG BH, CL
    XCHG BL, CH
    XOR DX, WORD PTR DECRYPT_WORD
    XOR CX, WORD PTR DECRYPT_WORD
    XOR BX, WORD PTR DECRYPT_WORD
    XOR AX, WORD PTR DECRYPT_WORD

    PUSH AX
    MOV AX, DX
    STOSW
    MOV AX, CX
    STOSW
    MOV AX, BX
    STOSW
    POP AX
    STOSW

    DEC IptBufferLength
    JNZ Decryption_P1
    RET

Decryption_Process ENDP
```

---

## Encryption Process

```asm
; Encrypting Process
; Loop encrypt object input buffer length must be a multiple of 8
; this process is a reverse process relative to decryption process
Encryption_Process Proc IptBuffer:dword, IptBufferLength:dword, Encryption_Word_Address:dword, OptBuffer:dword
    Local ENCRYPT_WORD:word ; Encrypt_Word equal Decrypt_Word
    MOV ESI, Encryption_Word_Address
    MOV AX, WORD PTR [ESI]
    MOV WORD PTR ENCRYPT_WORD, AX
    MOV ESI, IptBuffer
    MOV EDI, OptBuffer

Encryption_P1:
    CLD
    LODSW
    PUSH AX
    LODSW
    MOV BX, AX
    LODSW
    MOV CX, AX
    LODSW
    MOV DX, AX
    POP AX

    XOR AX, WORD PTR ENCRYPT_WORD
    XOR BX, WORD PTR ENCRYPT_WORD
    XOR CX, WORD PTR ENCRYPT_WORD
    XOR DX, WORD PTR ENCRYPT_WORD

    XCHG AH, DL
    XCHG AL, DH
    XCHG BH, CL
    XCHG BL, CH

    PUSH AX
    MOV AX, DX
    STOSW
    MOV AX, CX
    STOSW
    MOV AX, BX
    STOSW
    POP AX
    STOSW

    DEC IptBufferLength
    JNZ Encryption_P1
    RET

Encryption_Process ENDP
```
