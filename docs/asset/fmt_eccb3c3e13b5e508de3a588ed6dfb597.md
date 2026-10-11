
00374546  F3:A4          REP     MOVS BYTE PTR ES:[EDI], BYTE PTR DS:[ESI]                     ; 把 `ShellMap` 字符串复制到 990000 处

00374548  8B85 88474000  MOV     EAX, DWORD PTR SS:[EBP+<save_VerInfo>]

0037454E  3D 00000080    CMP     EAX, 80000000

00374553  73 16          JNB     SHORT <OSisWin9x>                                             ; 判断系统是否为 `WinNT` 或以上系统

00374555  64:FF35 3000000>PUSH    DWORD PTR FS:[30]                                             ; 如果是 `NT` 系统则检测 `IsDebuggerPresent`

0037455C  58             POP     EAX                                                           ; 检测 `Ring3` 级调试器

0037455D  0FB658 02      MOVZX   EBX, BYTE PTR DS:[EAX+2]

00374561  0ADB           OR      BL, BL

00374563  0F85 D51C0000  JNZ     <Game_Over>

00374569  EB 2A          JMP     SHORT 00374595

0037456B >  50           PUSH    EAX                                                           ; OSisWin9x

0037456C  0F014C24 FE    SIDT    FWORD PTR SS:[ESP-2]

00374571  5B            POP     EBX                                                           ; 0012FFE0

00374572  83C3 18       ADD     EBX, 18

00374575  8B4B 04       MOV     ECX, DWORD PTR DS:[EBX+4]

00374578  66:8B0B       MOV     CX, WORD PTR DS:[EBX]

0037457B  8B53 0C       MOV     EDX, DWORD PTR DS:[EBX+C]

0037457E  66:8B53 08    MOV     DX, WORD PTR DS:[EBX+8]

00374582  8B43 14       MOV     EAX, DWORD PTR DS:[EBX+14]

00374585  66:8B43 10    MOV     AX, WORD PTR DS:[EBX+10]

00374589  2BC2          SUB     EAX, EDX

0037458B  2BD1          SUB     EDX, ECX

0037458D  2BC2          SUB     EAX, EDX

0037458F  0F85 A91C0000  JNZ     <Game_Over>

;◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎重定位表处理◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎

  如果是 `DLL` 的话，这里填充重定位表，修复重定位表时要注意一点，如果加密时选择了加密输入和特殊代码加密的话，是不能直接通过

修改这里来获取到全部的重定位表．

00374595  8BB5 D6434000  MOV     ESI, DWORD PTR SS:[EBP+<Reloc_RVA(DLL)>]                    ; 判断是否有重定位表，一般的 `EXE` 这里为 0

0037459B  0BF6          OR      ESI, ESI

0037459D  74 4C         JE      SHORT <no_Reloc_Tab>                                          ; 如果没有重定位表则跳

0037459F  03B5 B2434000  ADD     ESI, DWORD PTR SS:[EBP+<IMGBASE>]                           ; 00400000

003745A5  8BBD B2434000  MOV     EDI, DWORD PTR SS:[EBP+<IMGBASE>]                           ; 00400000

003745AB  8BDF          MOV     EBX, EDI

003745AD  2BBD D2434000  SUB     EDI, DWORD PTR SS:[EBP+<Reloc_BASE>]                        ; 重定位后的实际基址

003745B3  0FB606        MOVZX   EAX, BYTE PTR DS:[ESI]

003745B6  EB 2F         JMP     SHORT 003745E7

003745B8 >  3C 01       CMP     AL, 1                                                         ; Loop_Fill_Reloc_Tab

003745BA  75 15         JNZ     SHORT 003745D1

003745BC  46            INC     ESI

003745BD  0FB606        MOVZX   EAX, BYTE PTR DS:[ESI]

003745C0  3C 02         CMP     AL, 2

003745C2  75 08         JNZ     SHORT 003745CC

003745C4  46            INC     ESI

003745C5  031E          ADD     EBX, DWORD PTR DS:[ESI]

003745C7  83C6 04       ADD     ESI, 4

003745CA  EB 18         JMP     SHORT 003745E4

003745CC  46            INC     ESI

003745CD  03D8          ADD     EBX, EAX

003745CF  EB 13         JMP     SHORT 003745E4

003745D1  3C 02         CMP     AL, 2

003745D3  75 0A         JNZ     SHORT 003745DF

003745D5  46            INC     ESI

003745D6  031E          ADD     EBX, DWORD PTR DS:[ESI]

003745D8  013B          ADD     DWORD PTR DS:[EBX], EDI                                       ; 填充重定位表

003745DA  83C6 04       ADD     ESI, 4

003745DD  EB 05         JMP     SHORT 003745E4

003745DF  46            INC     ESI

003745E0  03D8          ADD     EBX, EAX

003745E2  013B          ADD     DWORD PTR DS:[EBX], EDI                                       ; 填充重定位表

003745E4  0FB606        MOVZX   EAX, BYTE PTR DS:[ESI]

003745E7  0AC0          OR      AL, AL

003745E9  ^ 75 CD       JNZ     SHORT <Loop_Fill_Reloc_Tab>

;◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎重定位表处理完毕◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎

003745EB >  8CC9        MOV     CX, CS                                                          ; no_Reloc_Tab

003745ED  32C9          XOR     CL, CL

003745EF  0BC9          OR      ECX, ECX                                                        ; 判断操作系统

003745F1  74 32         JE      SHORT <Os_isWinNT>

003745F3  50            PUSH    EAX

003745F4  0F014C24 FE   SIDT    FWORD PTR SS:[ESP-2]

003745F9  5F            POP     EDI                                                           ; 0012FFE0

003745FA  83C7 20       ADD     EDI, 20

003745FD  8B4F 04       MOV     ECX, DWORD PTR DS:[EDI+4]

00374600  66:8B0F       MOV     CX, WORD PTR DS:[EDI]

00374603  FA            CLI

00374604  8DB5 434B4000  LEA     ESI, DWORD PTR SS:[EBP+404B43]

0037460A  66:8937       MOV     WORD PTR DS:[EDI], SI

0037460D  C1EE 10       SHR     ESI, 10

00374610  66:8977 06    MOV     WORD PTR DS:[EDI+6], SI

00374614  FB            STI

00374615  CD 04         INT     4

00374617  FA            CLI

00374618  66:890F       MOV     WORD PTR DS:[EDI], CX

0037461B  C1E9 10       SHR     ECX, 10

0037461E  66:894F 06    MOV     WORD PTR DS:[EDI+6], CX

00374622  FB            STI

00374623  EB 37         JMP     SHORT 0037465C

00374625 >  E8 0E000000  CALL    <Check_Debug>                                                 ; Os_isWinNT

0037462A  8B4C24 0C     MOV     ECX, DWORD PTR SS:[ESP+C]

0037462E  8381 B8000000 0>ADD     DWORD PTR DS:[ECX+B8], 2                                      ; 异常地址+2

00374635  33C0          XOR     EAX, EAX

00374637  C3            RETN

00374638 >  64:FF35 0000000>PUSH    DWORD PTR FS:[0]                                            ; Check_Debug

0037463F  64:8925 0000000>MOV      DWORD PTR FS:[0], ESP

00374646  33C0          XOR     EAX, EAX

00374648  CD 01         INT     1

0037464A  40            INC     EAX

0037464B  40            INC     EAX

0037464C  0BC0          OR      EAX, EAX

0037464E  64:8F05 0000000>POP      DWORD PTR FS:[0]                                            ; 0012FFE0

00374655  58            POP     EAX                                                           ; 0012FFE0

00374656  0F84 E21B0000  JE      <Game_Over>                                                    ; 如果是 sice 这里就要处理了

0037465C  8BB5 FA434000  MOV     ESI, DWORD PTR SS:[EBP+4043FA]                               ; 修正 `JMP IAT` 到 `HOOK` table

00374662  0BF6          OR      ESI, ESI                                                        ; 可惜我这个程序没有

00374664  74 27         JE      SHORT 0037468D

00374666  03B5 B2434000  ADD     ESI, DWORD PTR SS:[EBP+<IMGBASE>]                            ; 00400000

0037466C  EB 18         JMP     SHORT 00374686

0037466E  8B46 02       MOV     EAX, DWORD PTR DS:[ESI+2]

00374671  C1E0 05       SHL     EAX, 5

00374674  0385 FE434000  ADD     EAX, DWORD PTR SS:[EBP+<hMEM_IAT_RELOC_1>]

0037467A  2BC6          SUB     EAX, ESI

0037467C  48            DEC     EAX

0037467D  83E8 05       SUB     EAX, 5

00374680  8946 02       MOV     DWORD PTR DS:[ESI+2], EAX

00374683  83C6 06       ADD     ESI, 6

00374686  66:813E 90E9  CMP     WORD PTR DS:[ESI], 0E990

0037468B ^ 74 E1        JE      SHORT 0037466E

;++++++++++++++++++++++++++++++++++++++++++++++++++++对 `DELPHI` 程序的特别处理++++++++++++++++++++++++++++++++++++++++++++++++++++
