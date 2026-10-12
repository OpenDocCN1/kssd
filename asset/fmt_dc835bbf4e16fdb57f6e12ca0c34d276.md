
```
00374B64    B9 02000000      MOV     ECX, 2  
00374B69    F7E1             MUL     ECX  
00374B6B    D1E8             SHR     EAX, 1  
00374B6D    0385 B2434000    ADD     EAX, DWORD PTR SS:[EBP+<IMGBASE>]                     ; 00400000  
00374B73    2B85 D2434000    SUB     EAX, DWORD PTR SS:[EBP+<Reloc_BASE>]                  ; 00400000  
00374B79    3BF8             CMP     EDI, EAX  
00374B7B    75 0A            JNZ     SHORT 00374B87  
00374B7D    0AD2             OR      DL, DL  
00374B7F    75 04            JNZ     SHORT 00374B85  
00374B81    EB 09            JMP     SHORT 00374B8C  
00374B83    EB 02            JMP     SHORT 00374B87  
00374B85    EB 35            JMP     SHORT 00374BBC  
00374B87    83C6 08          ADD     ESI, 8  
00374B8A   ^ EB D4          JMP     SHORT 00374B60  
00374B8C    8B46 04          MOV     EAX, DWORD PTR DS:[ESI+4]                             ; 这里对 call [address]的处理  
00374B8F    0385 F6434000    ADD     EAX, DWORD PTR SS:[EBP+<UnLock_Important_Key>]  
00374B95    03BD D2434000    ADD     EDI, DWORD PTR SS:[EBP+<Reloc_BASE>]                  ; 00400000  
00374B9B    2BBD B2434000    SUB     EDI, DWORD PTR SS:[EBP+<IMGBASE>]                     ; 00400000  
00374BA1    2BC7             SUB     EAX, EDI  
00374BA3    F7D0             NOT     EAX  
00374BA5    C1C0 10          ROL     EAX, 10  
00374BA8    0385 B2434000    ADD     EAX, DWORD PTR SS:[EBP+<IMGBASE>]                     ; 00400000  
00374BAE    2B85 D2434000    SUB     EAX, DWORD PTR SS:[EBP+<Reloc_BASE>]                  ; 相减之后 eax 就是原 iat 的地址  
00374BB4    8B00             MOV     EAX, DWORD PTR DS:[EAX]                               ; 取出 IAT 中第一层的加密地址  
00374BB6    894424 20        MOV     DWORD PTR SS:[ESP+20], EAX  
00374BBA    61               POPAD  
00374BBB    C3               RETN  
00374BBC    8B46 04          MOV     EAX, DWORD PTR DS:[ESI+4]                             ; 这里对 jmp [address]的处理  
00374BBF    0385 F6434000    ADD     EAX, DWORD PTR SS:[EBP+<UnLock_Important_Key>]  
00374BC5    03BD D2434000    ADD     EDI, DWORD PTR SS:[EBP+<Reloc_BASE>]                  ; 00400000  
00374BCB    2BBD B2434000    SUB     EDI, DWORD PTR SS:[EBP+<IMGBASE>]                     ; 00400000  
00374BD1    2BC7             SUB     EAX, EDI  
00374BD3    F7D0             NOT     EAX  
00374BD5    C1C0 10          ROL     EAX, 10  
00374BD8    0385 B2434000    ADD     EAX, DWORD PTR SS:[EBP+<IMGBASE>]                     ; 00400000  
00374BDE    2B85 D2434000    SUB     EAX, DWORD PTR SS:[EBP+<Reloc_BASE>]                  ; 减了之后算出 jmp [address]中 address 的地址  
00374BE4    8B00             MOV     EAX, DWORD PTR DS:[EAX]                               ; 取出 IAT 中第一层的加密地址  
00374BE6    894424 24        MOV     DWORD PTR SS:[ESP+24], EAX  
00374BEA    61               POPAD  
00374BEB    83C4 04          ADD     ESP, 4                                               ; 因为是 jmp [address]所以这里要 add esp,4  
00374BEE    C3               RETN  

proc_Loaddll_failed:  
    00374BEF >  56               PUSH    ESI                                                  ; proc_Loaddll_failed  
    00374BF0    8D85 5B484000    LEA     EAX, DWORD PTR SS:[EBP+40485B]                       ; ASCII "can not found %s"  
    00374BF6    50               PUSH    EAX  
    00374BF7    8D85 74484000    LEA     EAX, DWORD PTR SS:[EBP+<strAPIName>]                 ; ASCII "RtlSetLastWin32Error"  
    00374BFD    50               PUSH    EAX  
    00374BFE    8D85 2D354000    LEA     EAX, DWORD PTR SS:[EBP+40352D]  
    00374C04    50               PUSH    EAX  
    00374C05    8B85 2A444000    MOV     EAX, DWORD PTR SS:[EBP+<APIwsPrintfA>]              ; USER32.wsprintfA  
    00374C0B   ^ E9 CAFDFFFF     JMP     <proc_Run_FUN>  
    00374C10    90               NOP  
    00374C11    83C4 0C          ADD     ESP, 0C  
    00374C14    6A 00            PUSH    0  
    00374C16    8D85 A4484000    LEA     EAX, DWORD PTR SS:[EBP+4048A4]                       ; ASCII "warning"  
    00374C1C    50               PUSH    EAX  
    00374C1D    8D85 74484000    LEA     EAX, DWORD PTR SS:[EBP+<strAPIName>]  
    00374C23    50               PUSH    EAX  
    00374C24    6A 00            PUSH    0  
    00374C26    8D85 55354000    LEA     EAX, DWORD PTR SS:[EBP+403555]  
    00374C2C    50               PUSH    EAX  
    00374C2D    8B85 35444000    MOV     EAX, DWORD PTR SS:[EBP+<APIMsgBox>]                 ; USER32.MessageBoxA  
    00374C33   ^ E9 A2FDFFFF     JMP     <proc_Run_FUN>  
    00374C38    90               NOP  
    00374C39    E9 00160000      JMP     <Game_Over>  

proc_check_CC:  
    00374C3E >  56               PUSH    ESI                                                  ; proc_check_CC  
    00374C3F    51               PUSH    ECX                                                  ; 检测 API 是否下了 cc 断点  
    00374C40    50               PUSH    EAX  
    00374C41    8BF0             MOV     ESI, EAX  
    00374C43    B9 01000000      MOV     ECX, 1  
    00374C48    AC               LODS    BYTE PTR DS:[ESI]  
    00374C49    3C CC            CMP     AL, 0CC  
    00374C4B    75 08            JNZ     SHORT 00374C55  
    00374C4D    58               POP     EAX                                                  ; 0012FFE0  
    00374C4E    59               POP     ECX                                                  ; 0012FFE0  
    00374C4F    5E               POP     ESI                                                  ; 0012FFE0  
    00374C50    E9 E9150000      JMP     <Game_Over>  
    00374C55   ^ E2 F1            LOOPD   SHORT 00374C48  
    00374C57    58               POP     EAX                                                  ; 0012FFE0  
    00374C58    59               POP     ECX                                                  ; 0012FFE0  
    00374C59    5E               POP     ESI                                                  ; 0012FFE0  
    00374C5A    C3               RETN  

Calculate_CRC:  
    00374C5B >  83CA FF          OR      EDX, FFFFFFFF                                      ; Calculate_CRC  
    00374C5E    51               PUSH    ECX  
    00374C5F    AC               LODS    BYTE PTR DS:[ESI]  
    00374C60    32C2             XOR     AL, DL  
    00374C62    6A 08            PUSH    8  
    00374C64    59               POP     ECX                                                  ; 0012FFE0  
    00374C65    0FB6D8           MOVZX   EBX, AL  
    00374C68    D1EB             SHR     EBX, 1  
    00374C6A    73 06            JNB     SHORT 00374C72  
    00374C6C    81F3 2083B8ED    XOR     EBX, EDB88320  
    00374C72   ^ E2 F4            LOOPD   SHORT 00374C68  
    00374C74    C1EA 08          SHR     EDX, 8  
    00374C77    33D3             XOR     EDX, EBX  
    00374C79    59               POP     ECX                                                  ; 0012FFE0  
    00374C7A   ^ E2 E2            LOOPD   SHORT 00374C5E  
    00374C7C    F7D2             NOT     EDX  
    00374C7E    92               XCHG    EAX, EDX  
    00374C7F    C3               RETN  

Game_Over:  
```
