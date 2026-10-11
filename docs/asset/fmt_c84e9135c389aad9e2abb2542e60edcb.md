
```
0037406F    75 0B             JNZ     SHORT 0037407C

00374071    8D85 5F3A4000    LEA     EAX, DWORD PTR SS:[EBP+<SDKGetCMDline>]

00374077    E9 29020000      JMP     <Fill_IAT_RELOC_2>

0037407C    52               PUSH    EDX

0037407D    52               PUSH    EDX

0037407E    8D85 41474000    LEA     EAX, DWORD PTR SS:[EBP+<strLockRes>]

00374084    50               PUSH    EAX

00374085    8D85 B4294000    LEA     EAX, DWORD PTR SS:[EBP+4029B4]

0037408B    50               PUSH    EAX

0037408C    8B85 E3444000    MOV     EAX, DWORD PTR SS:[EBP+<APIlstrcmpA>]                 ; kernel32.lstrcmpA

00374092    E9 43090000      JMP     <proc_Run_FUN>

00374097    90               NOP

00374098    5A               POP     EDX                                                 ; 0012FFE0

00374099    85C0             TEST    EAX, EAX

0037409B    75 0B            JNZ     SHORT 003740A8

0037409D    8D85 023B4000    LEA     EAX, DWORD PTR SS:[EBP+<SDKLockRes>]

003740A3    E9 FD010000      JMP     <Fill_IAT_RELOC_2>

003740A8    52               PUSH    EDX

003740A9    52               PUSH    EDX

003740AA    8D85 4E474000    LEA     EAX, DWORD PTR SS:[EBP+<strFreeRes>]

003740B0    50               PUSH    EAX

003740B1    8D85 E0294000    LEA     EAX, DWORD PTR SS:[EBP+4029E0]

003740B7    50               PUSH    EAX

003740B8    8B85 E3444000    MOV     EAX, DWORD PTR SS:[EBP+<APIlstrcmpA>]                 ; kernel32.lstrcmpA

003740BE    E9 17090000      JMP     <proc_Run_FUN>

003740C3    90               NOP

003740C4    5A               POP     EDX                                                 ; 0012FFE0

003740C5    85C0             TEST    EAX, EAX

003740C7    75 0B            JNZ     SHORT 003740D4

003740C9    8D85 023B4000    LEA     EAX, DWORD PTR SS:[EBP+<SDKLockRes>]

003740CF    E9 D1010000      JMP     <Fill_IAT_RELOC_2>

003740D4    52               PUSH    EDX

003740D5    52               PUSH    EDX

003740D6    8D85 28474000    LEA     EAX, DWORD PTR SS:[EBP+<strExitProc>]

003740DC    50               PUSH    EAX

003740DD    8D85 0C2A4000    LEA     EAX, DWORD PTR SS:[EBP+402A0C]

003740E3    50               PUSH    EAX

003740E4    8B85 E3444000    MOV     EAX, DWORD PTR SS:[EBP+<APIlstrcmpA>]                 ; kernel32.lstrcmpA

003740EA    E9 EB080000      JMP     <proc_Run_FUN>

003740EF    90               NOP

003740F0    5A               POP     EDX                                                 ; 0012FFE0

003740F1    85C0             TEST    EAX, EAX

003740F3    75 0B            JNZ     SHORT 00374100

003740F5    8D85 7C3A4000    LEA     EAX, DWORD PTR SS:[EBP+<SDKExitProc>]

003740FB    E9 A5010000      JMP     <Fill_IAT_RELOC_2>

00374100    52               PUSH    EDX

00374101    52               PUSH    EDX

00374102    8D85 5B474000    LEA     EAX, DWORD PTR SS:[EBP+<strDLGBoxParamA>]

00374108    50               PUSH    EAX

00374109    8D85 852A4000    LEA     EAX, DWORD PTR SS:[EBP+402A85]

0037410F    50               PUSH    EAX

00374110    8B85 E3444000    MOV     EAX, DWORD PTR SS:[EBP+<APIlstrcmpA>]                 ; kernel32.lstrcmpA

00374116    E9 BF080000      JMP     <proc_Run_FUN>

0037411B    90               NOP

0037411C    8BC5             MOV     EAX, EBP

0037411E    8DB5 014A4000    LEA     ESI, DWORD PTR SS:[EBP+404A01]                       ; 过完全部的异常后把最后异常的那些地址再加密回去

00374124    2946 04          SUB     DWORD PTR DS:[ESI+4], EAX

00374127    2946 08          SUB     DWORD PTR DS:[ESI+8], EAX

0037412A    83C6 20          ADD     ESI, 20

0037412D    2946 04          SUB     DWORD PTR DS:[ESI+4], EAX

00374130    83C6 20          ADD     ESI, 20

00374133    2946 04          SUB     DWORD PTR DS:[ESI+4], EAX

00374136    2946 08          SUB     DWORD PTR DS:[ESI+8], EAX

00374139    83C6 20          ADD     ESI, 20

0037413C    2946 04          SUB     DWORD PTR DS:[ESI+4], EAX

0037413F    83C6 20          ADD     ESI, 20

00374142    2946 04          SUB     DWORD PTR DS:[ESI+4], EAX

00374145    83C6 20          ADD     ESI, 20

00374148    2946 04          SUB     DWORD PTR DS:[ESI+4], EAX

0037414B    83C6 20          ADD     ESI, 20

0037414E    2946 04          SUB     DWORD PTR DS:[ESI+4], EAX

00374151    83C6 20          ADD     ESI, 20

00374154    2946 04          SUB     DWORD PTR DS:[ESI+4], EAX

00374157    8DB5 FD494000    LEA     ESI, DWORD PTR SS:[EBP+4049FD]

0037415D    B8 014A4000      MOV     EAX, 404A01

00374162    8906             MOV     DWORD PTR DS:[ESI], EAX

00374164    E9 F5F8FFFF      JMP     00373A5E                                            ; 跳去处理 OEP 的代码

00374169    5A               POP     EDX                                                 ; 0012FFE0

0037416A    85C0             TEST    EAX, EAX

0037416C    75 0B            JNZ     SHORT 00374179                                      ; 如果不是特殊函数则跳

0037416E    8D85 8B3A4000    LEA     EAX, DWORD PTR SS:[EBP+<SDKDLGBoxParamA>]

00374174    E9 2C010000      JMP     <Fill_IAT_RELOC_2>

00374179    52               PUSH    EDX

0037417A    52               PUSH    EDX

0037417B    8D85 6B474000    LEA     EAX, DWORD PTR SS:[EBP+<strCreateDLGParamA>]

00374181    50               PUSH    EAX

00374182    8D85 B12A4000    LEA     EAX, DWORD PTR SS:[EBP+402AB1]

00374188    50               PUSH    EAX

00374189    8B85 E3444000    MOV     EAX, DWORD PTR SS:[EBP+<APIlstrcmpA>]                 ; kernel32.lstrcmpA

0037418F    E9 46080000      JMP     <proc_Run_FUN>

00374194    90               NOP

00374195    5A               POP     EDX                                                 ; 0012FFE0

00374196    85C0             TEST    EAX, EAX

00374198    75 0B            JNZ     SHORT 003741A5

0037419A    8D85 C83A4000    LEA     EAX, DWORD PTR SS:[EBP+<SDKCreateDLGParamA>]

003741A0    E9 00010000      JMP     <Fill_IAT_RELOC_2>

003741A5    52               PUSH    EDX

003741A6    52               PUSH    EDX

003741A7    8D85 34474000    LEA     EAX, DWORD PTR SS:[EBP+<strSndMsg>]

003741AD    50               PUSH    EAX

003741AE    8D85 DD2A4000    LEA     EAX, DWORD PTR SS:[EBP+402ADD]

003741B4    50               PUSH    EAX

003741B5    8B85 E3444000    MOV     EAX, DWORD PTR SS:[EBP+<APIlstrcmpA>]                 ; kernel32.lstrcmpA

003741BB    E9 1A080000      JMP     <proc_Run_FUN>

003741C0    90               NOP

003741C1    5A               POP     EDX                                                 ; 0012FFE0

003741C2    85C0             TEST    EAX, EAX

003741C4    75 0B            JNZ     SHORT 003741D1

003741C6    8D85 2E3B4000    LEA     EAX, DWORD PTR SS:[EBP+<SDKSndMsg>]

003741CC    E9 D4000000      JMP     <Fill_IAT_RELOC_2>

003741D1    52               PUSH    EDX

003741D2    52               PUSH    EDX

003741D3    8D85 7E474000    LEA     EAX, DWORD PTR SS:[EBP+<strsend>]

003741D9    50               PUSH    EAX

003741DA    8D85 092B4000    LEA     EAX, DWORD PTR SS:[EBP+402B09]

003741E0    50               PUSH    EAX

003741E1    8B85 E3444000    MOV     EAX, DWORD PTR SS:[EBP+<APIlstrcmpA>]                 ; kernel32.lstrcmpA

003741E7    E9 EE070000      JMP     <proc_Run_FUN>

003741EC    90               NOP

003741ED    5A               POP     EDX                                                 ; 0012FFE0

003741EE    85C0             TEST    EAX, EAX

003741F0    75 0B            JNZ     SHORT 003741FD

003741F2    8D85 323B4000    LEA     EAX, DWORD PTR SS:[EBP+<SDKsend>]

003741F8    E9 A8000000      JMP     <Fill_IAT_RELOC_2>

003741FD    52               PUSH    EDX
```
