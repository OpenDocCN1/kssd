
```
004136FE    C1C2 04                  ROL EDX,4  

00413701    75 01                    JNZ SHORT PESpin.00413704  

00413704    43                       INC EBX                          ; PESpin.0040C4C8  

00413705    FF6424 FC                JMP DWORD PTR SS:[ESP-4]          ; 004136F7  

00413709    93                       XCHG EAX,EBX                      

0041370A    8B56 10                 MOV EDX,DWORD PTR DS:[ESI+10]      ; First Thunk  

0041370D    0395 DF4E4000           ADD EDX,DWORD PTR SS:[EBP+404EDF]  ; 00400000  

00413713    830A 00                 OR DWORD PTR DS:[EDX],0  

00413716    0F84 59010000           JE PESpin.00413875                 ; 一个 DLL 处理完了, 出口  

00413729    8B02                    MOV EAX,DWORD PTR DS:[EDX]  

0041372B    A9 00000080             TEST EAX,80000000  

00413730    74 0A                   JE SHORT PESpin.0041373C          ; 跳, Name 导入, 否则 Oridnal 导入  

00413732    25 FFFFFF7F             AND EAX,7FFFFFFF  

00413737    2BFF                    SUB EDI,EDI  

00413739   /EB 09                   JMP SHORT PESpin.00413744  

0041373C    40                      INC EAX                           ; NameHash 前保留了 API Name 首字母  

0041373D    0385 DF4E4000           ADD EAX,DWORD PTR SS:[EBP+404EDF] ; PESpin.00400000  

00413743    97                      XCHG EAX,EDI                     ; EDI 指向 NameHash  

00413744    68 5DFDD0F9            PUSH F9D0FD5D  

00413749    012C24                 ADD DWORD PTR SS:[ESP],EBP  

0041374C    810424 B4466F06        ADD DWORD PTR SS:[ESP],66F46B4     ; 下面 Call 的返回地址 00413765  

00413753    68 A17D630F            PUSH 0F637DA1  

00413758    812C24 9643230F        SUB DWORD PTR SS:[ESP],0F234396  

0041375F    012C24                 ADD DWORD PTR SS:[ESP],EBP  

00413762    C3                      RETN                               ; Call 00412D5F, 返回 API 地址  

; =====================================================================================================================  

; 接下来的一段子程序根据 NameHash 获的 API Address  

; 根据 NameHash 求 API,   EDI 指向 NameHash  

; 根据 Oridnal 求 API,     EDI =0,  EAX = Oridnal  

00412D5F    60                      PUSHAD  

00412D69    0BFF                   OR EDI,EDI                          

00412D6B    75 19                   JNZ SHORT PESpin.00412D86           ; 00412D92  

00412D6D    8B9D 063A4000          MOV EBX,DWORD PTR SS:[EBP+403A06]   ; AddressOfFunctions  

00412D73    2D 01000000            SUB EAX,1                             ; Oridnal Base  

00412D78    8B0483                 MOV EAX,DWORD PTR DS:[EBX+EAX*4]  

00412D7B    0385 FE394000          ADD EAX,DWORD PTR SS:[EBP+4039FE]   ; 根据 Oridnal 求地址 API 地址 OK  

00412D81   /E9 B1000000            JMP PESpin.00412E37  

00412D92    8B9D 023A4000          MOV EBX,DWORD PTR SS:[EBP+403A02]   ; AddressOfNames  

00412D98    8A47 FF                MOV AL,BYTE PTR DS:[EDI-1]  

00412D9B    24 7F                  AND AL,7F                            ; NameHash 前一字母, 第 8 位是标志(是否重定向)  

00412D9D    8885 803A4000          MOV BYTE PTR SS:[EBP+403A80],AL      ; [00412DD4]  

00412DB4    FF37                   PUSH DWORD PTR DS:[EDI]              ; DllName Hash  

00412DB6    8F85 AD3A4000          POP DWORD PTR SS:[EBP+403AAD]       ; [00412E01]  

00412DBC    2BC9                   SUB ECX,ECX  

00412DBE    8327 00                AND DWORD PTR DS:[EDI],0             ; 破坏  

00412DCA    8B3B                   MOV EDI,DWORD PTR DS:[EBX]  

00412DCC    03BD FE394000          ADD EDI,DWORD PTR SS:[EBP+4039FE]   ; 从 Export 中取出名字  

00412DD2    803F 47                CMP BYTE PTR DS:[EDI],47             ; 比较首字母  

00412DD5    75 50                  JNZ SHORT PESpin.00412E27  

00412DD7    E8 03000000            CALL PESpin.00412DDF                ; 首字母对上了, 再比较 NameHash  

00412DDF    58                     POP EAX  

00412DE0    8D6424 FC              LEA ESP,DWORD PTR SS:[ESP-4]  

00412DE4    05 24000000            ADD EAX,24  

00412DE9    870424                 XCHG DWORD PTR SS:[ESP],EAX         ; 00412EE0 下面 CALL 返回地址  

00412DEC    8D85 415C028F          LEA EAX,DWORD PTR SS:[EBP+8F025C41]  

00412DF2    2D BF0AC28E            SUB EAX,8EC20ABF  

00412DF7    50                     PUSH EAX  

00412DF8    C3                     RETN                                ; 004144D6 ( Hash 函数)  

00412E00    3D 3A4E99D2            CMP EAX,D2994E3A                    ; 比较 Hash  

00412E05    75 20                  JNZ SHORT PESpin.00412E27  

00412E07    8B85 FA394000          MOV EAX,DWORD PTR SS:[EBP+4039FA]  ; AddressOfNameOrdinals  

00412E0D    D1E1                   SHL ECX,1                           ; 第几个  

00412E0F    03C1                   ADD EAX,ECX  

00412E11    0FB700                 MOVZX EAX,WORD PTR DS:[EAX]  

00412E14    C1E0 02                SHL EAX,2  

00412E17    0385 063A4000          ADD EAX,DWORD PTR SS:[EBP+403A06]  ; AddressOfFunctions  

00412E1D    8B00                   MOV EAX,DWORD PTR DS:[EAX]  

00412E1F    0385 FE394000          ADD EAX,DWORD PTR SS:[EBP+4039FE]  ; Dll Base  

00412E25    EB 10                  JMP SHORT PESpin.00412E37  

00412E27    83C3 04                ADD EBX,4                           ; Export 中下一个 API  

00412E2A    41                     INC ECX  

00412E2B    81F9 3D030000          CMP ECX,33D  

00412E31  ^ 75 97                  JNZ SHORT PESpin.00412DCA  

00412E33    33C0                   XOR EAX,EAX                         ; 找不到, 则 API address = 0  

00412E35    EB 3F                  JMP SHORT PESpin.00412E76  

00412E37    8BBD F2394000          MOV EDI,DWORD PTR SS:[EBP+4039F2]   ; 是否是指向另一 DLL  

00412E3D    3BC7                   CMP EAX,EDI                         

00412E3F    76 35                  JBE SHORT PESpin.00412E76  

00412E41    03BD F6394000          ADD EDI,DWORD PTR SS:[EBP+4039F6]  

00412E47    3BF8                   CMP EDI,EAX  

00412E49    76 2B                  JBE SHORT PESpin.00412E76  

00412E4B    8DBD AC2C4000          LEA EDI,DWORD PTR SS:[EBP+402CAC]   ; 指向另一 DLL 的另一 API  

00412E51    96                     XCHG EAX,ESI  

00412E52    33C9                  XOR ECX,ECX  

00412E54    8A0431                 MOV AL,BYTE PTR DS:[ECX+ESI]  

00412E57    3C 2E                  CMP AL,2E                           ; "."  

00412E59    74 04                  JE SHORT PESpin.00412E5F  

00412E5B    41                     INC ECX  

00412E5C    AA                     STOS BYTE PTR ES:[EDI]  

00412E5D  ^ EB F5                  JMP SHORT PESpin.00412E54  

00412E5F    41                     INC ECX  

00412E60    03F1                   ADD ESI,ECX  

00412E62    56                     PUSH ESI  

00412E63    2C 2E                  SUB AL,2E  

00412E65    AA                     STOS BYTE PTR ES:[EDI]  

00412E66    2BF9                   SUB EDI,ECX  

00412E68    57                     PUSH EDI  

00412E69    FF95 A24F4000          CALL DWORD PTR SS:[EBP+404FA2]      ; LoadLibraryA  

00412E6F    50                     PUSH EAX  

00412E70    FF95 A74F4000          CALL DWORD PTR SS:[EBP+404FA7]      ; GetProcAddress  

00412E76   /EB 01                  JMP SHORT PESpin.00412E79  

00412E79    894424 1C              MOV DWORD PTR SS:[ESP+1C],EAX       ; KERNEL32.GetCommandLineA  

00412E7D    61                     POPAD  

00412E7E    FF0424                 INC DWORD PTR SS:[ESP]              ; 返回地址 00413766  

00412EB3    0BC0                   OR EAX,EAX                          ; KERNEL32.GetCommandLineA
```
