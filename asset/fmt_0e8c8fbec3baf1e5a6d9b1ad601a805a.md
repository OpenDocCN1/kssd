
```
00373B38    8D85 64244000   LEA     EAX, DWORD PTR SS:[EBP+402464]  
00373B3E    50             PUSH    EAX  
00373B3F    8B85 12444000  MOV     EAX, DWORD PTR SS:[EBP+404412]                          ; <aplib_Unpack>  
00373B45    FFE0           JMP     EAX                                                    ; 跳去 aplib 解压代码  
00373B48    8B0C2B         MOV     ECX, DWORD PTR DS:[EBX+EBP]                             ; 解压大小 59000  
00373B4B    56             PUSH    ESI  
00373B4C    51             PUSH    ECX  
00373B4D    C1E9 02        SHR     ECX, 2  
00373B50    F3:A5          REP     MOVS DWORD PTR ES:[EDI], DWORD PTR DS:[ESI]              ; 解压代码到相关位置  
00373B52    59             POP     ECX                                                    ; 0012FFE0  
00373B53    83E1 03        AND     ECX, 3  
00373B56    F3:A4          REP     MOVS BYTE PTR ES:[EDI], BYTE PTR DS:[ESI]  
00373B58    5E             POP     ESI                                                    ; 0012FFE0  
00373B59    53             PUSH    EBX  
00373B5A    68 00800000    PUSH    8000  
00373B5F    6A 00          PUSH    0  
00373B61    56             PUSH    ESI  
00373B62    8D85 91244000  LEA     EAX, DWORD PTR SS:[EBP+402491]  
00373B68    50             PUSH    EAX  
00373B69    8B85 AF444000  MOV     EAX, DWORD PTR SS:[EBP+<APIVirtualFree>]               ; kernel32.VirtualFree  
00373B6F    E9 660E0000    JMP     <proc_Run_FUN>  
00373B74    90             NOP  
00373B75    5B             POP     EBX                                                    ; 0012FFE0  
00373B76    83C3 0C        ADD     EBX, 0C  
00373B79  ^ E9 7BFEFFFF    JMP     <Loop_De_Code>                                         ; 循环解压出相关代码  
;══════════════════════还原 CALL 和相关跳转═════════════════════════  
00373B7E >  8BB5 DD484000  MOV     ESI, DWORD PTR SS:[EBP+4048DD]                          ; Unpacked_Section_DONE  
00373B84    03B5 B2434000  ADD     ESI, DWORD PTR SS:[EBP+<IMGBASE>]                       ; 00400000  
00373B8A    8B8D E1484000  MOV     ECX, DWORD PTR SS:[EBP+4048E1]  
00373B90    83E9 05        SUB     ECX, 5  
00373B93    EB 5B          JMP     SHORT 00373BF0  
00373B95 >  66:8B06        MOV     AX, WORD PTR DS:[ESI]                                  ; De_JMP_CALL  
00373B98    3C E8          CMP     AL, 0E8                                                ; jmp address  
00373B9A    75 16          JNZ     SHORT 00373BB2                                         ; 和远程跳转  
00373B9C    8BC6           MOV     EAX, ESI  
00373B9E    2B85 B2434000  SUB     EAX, DWORD PTR SS:[EBP+<IMGBASE>]                      ; 00400000  
00373BA4    83C0 05        ADD     EAX, 5  
00373BA7    2946 01        SUB     DWORD PTR DS:[ESI+1], EAX  
00373BAA    83C6 04        ADD     ESI, 4  
00373BAD    83E9 04        SUB     ECX, 4  
00373BB0    EB 3C          JMP     SHORT 00373BEE  
00373BB2    3C E9          CMP     AL, 0E9  
00373BB4    75 16          JNZ     SHORT 00373BCC  
00373BB6    8BC6           MOV     EAX, ESI  
00373BB8    2B85 B2434000  SUB     EAX, DWORD PTR SS:[EBP+<IMGBASE>]                      ; 00400000  
00373BBE    83C0 05        ADD     EAX, 5  
00373BC1    2946 01        SUB     DWORD PTR DS:[ESI+1], EAX  
00373BC4    83C6 04        ADD     ESI, 4  
00373BC7    83E9 04        SUB     ECX, 4  
00373BCA    EB 22          JMP     SHORT 00373BEE  
00373BCC    3C 0F          CMP     AL, 0F  
00373BCE    75 1E          JNZ     SHORT 00373BEE  
00373BD0    80FC 7F        CMP     AH, 7F  
00373BD3    76 19          JBE     SHORT 00373BEE  
00373BD5    80FC 90        CMP     AH, 90  
00373BD8    73 14          JNB     SHORT 00373BEE  
00373BDA    8BC6           MOV     EAX, ESI  
00373BDC    2B85 B2434000  SUB     EAX, DWORD PTR SS:[EBP+<IMGBASE>]                      ; 00400000  
00373BE2    83C0 06        ADD     EAX, 6  
00373BE5    2946 02        SUB     DWORD PTR DS:[ESI+2], EAX  
00373BE8    83C6 05        ADD     ESI, 5  
00373BEB    83E9 05        SUB     ECX, 5  
00373BEE    46             INC     ESI  
00373BEF    49             DEC     ECX  
00373BF0    81F9 00000080  CMP     ECX, 80000000  
00373BF6  ^ 72 9D          JB      SHORT <De_JMP_CALL>  
;══════════════════════END══════════════════════  

......  

00373BF8    8DB5 4E254000  LEA     ESI, DWORD PTR SS:[EBP+<Next_Decode_addr>]             ; 开始解出下一段代码  
00373BFE    87E6           XCHG    ESI, ESP  
00373C00    B9 930B0000    MOV     ECX, 0B93                                              ; 解压大小 0b93  
00373C05    58             POP     EAX                                                    ; 0012FFE0  
00373C06    F6D0           NOT     AL  
00373C08    50             PUSH    EAX  
00373C09    44             INC     ESP  
00373C0A  ^ E2 F9          LOOPD   SHORT 00373C05  
00373C0C    87E6           XCHG    ESI, ESP  
;⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕输入表处理⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕⊕  
```

解压出代码后开始处理输入表部分了，输入表部分是复杂了点．总体是这样子，`simonzh2k` 和 `Window` 已经标的很明白，我直接搬了过来:

加密后的 `IAT` 在内存里如下存放(搬了 `simonzh2k` 的)

```
  ; 1. FF FF FF FF    ----------- 00 00 00 00 表示 所有 DLL 结束  
  ; 2. xx   ----- DLL Name 长度(不算 null)  
  ; 3. DLL 名字, null 结尾 ( 明文 )  
  ; 4. 80 yy yy yy    ---------- yy yy yy 表示 API 个数 , 80 表示需要重定向  
  ; 5. zz           ---------- ZZ<>0 表示 API Name 长度(不算 null), ZZ==0, 后 4 byte 函数序号, 1 byte NULL  
  ; 6. API Name, null 结尾 ( 密文, 解密代码见 12FF68 )  
  ; 7. 重复 5, 6 结束 一个 DLL  
  ; 重复 1,..,7 处理 所有 DLL  

; 经过壳的 iat 处理,形成了下面的一个调用过程, ( 引用 window 的表示)  
;  
; iat 中地址 --> Hook_proc:  
; Hook_proc:  
;             |PUSH DWORD PTR DS:[Hook_proc+1C]  
;             |XOR DWORD PTR SS:[ESP], key  
;             |ret; -> |Stub_proc:  
;                       |api_start_code  
                       |api_some_code  
                       |push api_next_code_addr  
                       |ret  
```

```
00373C0E    6A 04          PUSH    4  
00373C10    68 00100000    PUSH    1000  
00373C15    68 00200000    PUSH    2000  
00373C1A    6A 00          PUSH    0  
00373C1C    FF95 AE434000  CALL    DWORD PTR SS:[EBP+<APIVirtualAlloc>]                 ; kernel32.VirtualAlloc  
00373C22    8985 E9494000  MOV     DWORD PTR SS:[EBP+<hvMEM>], EAX  
00373C28    C785 ED494000 0>MOV     DWORD PTR SS:[EBP+<shellFunAddress>], 0  
00373C32    8B85 DE434000  MOV     EAX, DWORD PTR SS:[EBP+<flgCrypt_Improt>]             ; (initial cpu selection)  
00373C38    0BC0           OR      EAX, EAX  
00373C3A    0F85 BD000000  JNZ     <IAT_isCrypted>                                       ; 如果输入表加密了则跳  
00373C40    8BBD 02444000  MOV     EDI, DWORD PTR SS:[EBP+<IAT_RVA>]                    ; 如果没有加密则这里是输入表的 rva，加密了就不是了  
00373C46    03BD B2434000  ADD     EDI, DWORD PTR SS:[EBP+<IMGBASE>]                    ; 00400000  
00373C4C >  8B77 0C        MOV     ESI, DWORD PTR DS:[EDI+C]                             ; dis_Dlls  
00373C4F    0BF6           OR      ESI, ESI  
00373C51    75 05          JNZ     SHORT <dis_iat>                                       ; 如果没有处理完输入表则跳  
00373C53    E9 A0000000    JMP     <not_crypt_IAT_dis_Done>  
00373C58 >  03B5 B2434000  ADD     ESI, DWORD PTR SS:[EBP+<IMGBASE>]                   ; dis_iat  
00373C5E    56             PUSH    ESI  
00373C5F    8D85 8F254000  LEA     EAX, DWORD PTR SS:[EBP+40258F]  
00373C65    50             PUSH    EAX  
00373C66    8B85 A6434000  MOV     EAX, DWORD PTR SS:[EBP+<GetModhandle>]               ; kernel32.GetModuleHandleA  
00373C6C    E9 690D0000    JMP     <proc_Run_FUN>                                        ; 判断 DLL 是否已经载入  
00373C71    90             NOP  
```
