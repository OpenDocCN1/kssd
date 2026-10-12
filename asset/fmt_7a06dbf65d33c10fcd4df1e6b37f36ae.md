
```
00373C72    90               NOP  

00373C73    0BC0             OR      EAX, EAX  

00373C75    75 1E            JNZ     SHORT <dll\_isLoaded>  

00373C77    56               PUSH    ESI  

00373C78    8D85 A8254000    LEA     EAX, DWORD PTR SS:\[EBP+4025A8\]  

00373C7E    50               PUSH    EAX  

00373C7F    8B85 AA434000    MOV     EAX, DWORD PTR SS:\[EBP+<APILoadLib>\]                     ; kernel32.LoadLibraryA  

00373C85    E9 500D0000      JMP     <proc\_Run\_FUN>  

00373C8A    90               NOP  

00373C8B    90               NOP  

00373C8C    0BC0             OR      EAX, EAX  

00373C8E    75 05            JNZ     SHORT <dll\_isLoaded>  

00373C90    E9 5A0F0000      JMP     <proc\_Loaddll\_failed>                                 ; 载入 DLL 失败显示失败信息  

00373C95 >  8BF0             MOV     ESI, EAX                                                ; dll\_isLoaded  

00373C97    8B17             MOV     EDX, DWORD PTR DS:\[EDI\]  

00373C99    0BD2             OR      EDX, EDX  

00373C9B    75 03            JNZ     SHORT 00373CA0  

00373C9D    8B57 10          MOV     EDX, DWORD PTR DS:\[EDI+10\]                            ; 004480AC  

00373CA0    0395 B2434000    ADD     EDX, DWORD PTR SS:\[EBP+<IMGBASE>\]                     ; 00400000  

00373CA6    8B5F 10          MOV     EBX, DWORD PTR DS:\[EDI+10\]                            ; 004480AC  

00373CA9    039D B2434000    ADD     EBX, DWORD PTR SS:\[EBP+<IMGBASE>\]                     ; 00400000  

00373CAF >  8B02             MOV     EAX, DWORD PTR DS:\[EDX\]                               ; dis\_current\_DLLs\_api  

00373CB1    0BC0             OR      EAX, EAX  

00373CB3    75 02            JNZ     SHORT 00373CB7  

00373CB5    EB 39            JMP     SHORT 00373CF0  

00373CB7    53               PUSH    EBX  

00373CB8    52               PUSH    EDX  

00373CB9    99               CDQ  

00373CBA    0BD2             OR      EDX, EDX  

00373CBC    75 0B            JNZ     SHORT <is\_number1>                                      ; 是序号还是 API 名字  

00373CBE    83C0 02          ADD     EAX, 2  

00373CC1    0385 B2434000    ADD     EAX, DWORD PTR SS:\[EBP+<IMGBASE>\]                     ; 00400000  

00373CC7    EB 05            JMP     SHORT 00373CCE  

00373CC9 >  25 FFFFFFFF      AND     EAX, 7FFFFFFF                                          ; is\_number1  

00373CCE    50               PUSH    EAX  

00373CCF    56               PUSH    ESI  

00373CD0    8D85 00264000    LEA     EAX, DWORD PTR SS:\[EBP+402600\]  

00373CD6    50               PUSH    EAX  

00373CD7    8B85 9E434000    MOV     EAX, DWORD PTR SS:\[EBP+<MyGetProcAddress>\]  

00373CDD    E9 F80C0000      JMP     <proc\_Run\_FUN>  

00373CE2    90               NOP  

00373CE3    90               NOP  

00373CE4    8903             MOV     DWORD PTR DS:\[EBX\], EAX                               ; 填充输入表  

00373CE6    5A               POP     EDX                                                     ; 0012FFE0  

00373CE7    5B               POP     EBX                                                     ; 0012FFE0  

00373CE8    83C2 04          ADD     EDX, 4  

00373CEB    83C3 04          ADD     EBX, 4  

00373CEE   ^ EB BF           JMP     SHORT <dis\_current\_DLLs\_api>  

00373CF0    83C7 14          ADD     EDI, 14  

00373CF3   ^ E9 54FFFFFF      JMP     <dis\_Dlls>                                             ; 循环填充输入表  

00373CF8 >  E9 C6050000      JMP     <Disposal\_IAT\_Done>                                    ; not\_crypt\_IAT\_dis\_Done  

00373CFD >  8D95 A01A4000    LEA     EDX, DWORD PTR SS:\[EBP+<Crc\_Start\_addr>\]             ; IAT\_isCrypted  

00373D03    0395 02444000    ADD     EDX, DWORD PTR SS:\[EBP+<IAT\_RVA>\]  

00373D09 >  8B3A             MOV     EDI, DWORD PTR DS:\[EDX\]                               ; loop\_De\_Crypted\_iat  

00373D0B    0BFF             OR      EDI, EDI  

00373D0D    75 05            JNZ     SHORT <DIS\_NEXT\_1>                                     ; 如果没有处理完 IAT 则跳  

00373D0F    E9 AF050000      JMP     <Disposal\_IAT\_Done>  

00373D14 >  03BD B2434000    ADD     EDI, DWORD PTR SS:\[EBP+<IMGBASE>\]                     ; DIS\_NEXT\_1  

00373D1A    83C2 05          ADD     EDX, 5  

00373D1D    8BF2             MOV     ESI, EDX  

00373D1F    56               PUSH    ESI  

00373D20    8D85 50264000    LEA     EAX, DWORD PTR SS:\[EBP+402650\]  

00373D26    50               PUSH    EAX  

00373D27    8B85 A6434000    MOV     EAX, DWORD PTR SS:\[EBP+<GetModhandle>\]                ; kernel32.GetModuleHandleA  

00373D2D    E9 A80C0000      JMP     <proc\_Run\_FUN>  

00373D32    90               NOP  

00373D33    90               NOP  

00373D34    0BC0             OR      EAX, EAX  

00373D36    75 1E            JNZ     SHORT 00373D56  

00373D38    56               PUSH    ESI  

00373D39    8D85 69264000    LEA     EAX, DWORD PTR SS:\[EBP+402669\]  

00373D3F    50               PUSH    EAX  

00373D40    8B85 AA434000    MOV     EAX, DWORD PTR SS:\[EBP+<APILoadLib>\]                   ; kernel32.LoadLibraryA  

00373D46    E9 8F0C0000      JMP     <proc\_Run\_FUN>  

00373D4B    90               NOP  

00373D4C    90               NOP  

00373D4D    0BC0             OR      EAX, EAX  

00373D4F    75 05            JNZ     SHORT 00373D56  

00373D51    E9 990E0000      JMP     <proc\_Loaddll\_failed>  

00373D56    0FB64E FF        MOVZX   ECX, BYTE PTR DS:\[ESI\-1\]                              ; MODULE NAME 长度  

00373D5A    03F1             ADD     ESI, ECX  

00373D5C    8BD6             MOV     EDX, ESI  

00373D5E    8BF0             MOV     ESI, EAX  

00373D60    42               INC     EDX  

00373D61    8B0A             MOV     ECX, DWORD PTR DS:\[EDX\]                               ; 本 DLL 需引入函数的数  

00373D63    81E1 00000080    AND     ECX, 80000000  

00373D69    0BC9             OR      ECX, ECX  

00373D6B    0F85 87000000   JNZ     <Reloc\_FUN>                                              ; 判断 DLL 中的 API 是否需要重定位处理,如果需要则跳  

00373D71    8B0A             MOV     ECX, DWORD PTR DS:\[EDX\]                               ; 不需要特殊处理的 API 则跳这里  

00373D73    83C2 04          ADD     EDX, 4  

00373D76 >  51               PUSH    ECX                                                     ; loop\_not\_relocs\_api  

00373D77    0FB602           MOVZX   EAX, BYTE PTR DS:\[EDX\]  

00373D7A    0BC0             OR      EAX, EAX  

00373D7C    75 27            JNZ     SHORT <not\_reloc\_Ord\_by\_name>  

00373D7E    42               INC     EDX                                                     ; 以序号方式填充  

00373D7F    52               PUSH    EDX  

00373D80    8B02             MOV     EAX, DWORD PTR DS:\[EDX\]  

00373D82    50               PUSH    EAX  

00373D83    56               PUSH    ESI  

00373D84    8D85 B4264000    LEA     EAX, DWORD PTR SS:\[EBP+4026B4\]  

00373D8A    50               PUSH    EAX  

00373D8B    8B85 9E434000    MOV     EAX, DWORD PTR SS:\[EBP+<MyGetProcAddress>\]  

00373D91    E9 440C0000      JMP     <proc\_Run\_FUN>  

00373D96    90               NOP  

00373D97    36:E8 A10E0000   CALL    <proc\_check\_CC>                                          ; Superfluous prefix  

00373D9D    8907             MOV     DWORD PTR DS:\[EDI\], EAX                               ; 填充 IAT  

00373D9F    5A               POP     EDX                                                     ; 0012FFE0  

00373DA0    83C2 04          ADD     EDX, 4  

00373DA3    EB 47            JMP     SHORT 00373DEC  

00373DA5 >  42               INC     EDX                                                     ; not\_reloc\_Ord\_by\_name  

00373DA6    52               PUSH    EDX  

00373DA7    60               PUSHAD  

00373DA8    8BF2             MOV     ESI, EDX  

00373DAA    8DBD 74484000    LEA     EDI, DWORD PTR SS:\[EBP+<strAPIName>\]                   ; 循环解压出 API 名称  
```
