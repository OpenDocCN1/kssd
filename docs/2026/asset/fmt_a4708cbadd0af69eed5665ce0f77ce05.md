
```
    0058E7F6    FF95 20854100   CALL DWORD PTR SS:[EBP+418520]
    0058E7FC    EB 17           JMP SHORT 0058E815
    0058E7FE    90              NOP
    0058E7FF    90              NOP
    0058E800    90              NOP
    0058E801    B8 E8030000     MOV EAX,3E8                               ; Over
    0058E806    E8 C3D0FFFF     CALL <Get_RND_VALUE>
    0058E80B    8DBD 615D4000   LEA EDI,DWORD PTR SS:[EBP+405D61]
    0058E811    03F8            ADD EDI,EAX
    0058E813    AB              STOS DWORD PTR ES:[EDI]
    0058E814    58              POP EAX
    0058E815    0BC0            OR EAX,EAX
    0058E817    75 3D           JNZ SHORT 0058E856
    0058E819    90              NOP
    0058E81A    90              NOP
    0058E81B    90              NOP
    0058E81C    90              NOP
    0058E81D    53              PUSH EBX
    0058E81E    50              PUSH EAX
    0058E81F    8B85 24854100   MOV EAX,DWORD PTR SS:[EBP+418524]
    0058E825    0FB600          MOVZX EAX,BYTE PTR DS:[EAX]
    0058E828    83E8 33         SUB EAX,33
    0058E82B    3D 99000000     CMP EAX,99
    0058E830    74 10           JE SHORT 0058E842
    0058E832    90              NOP
    0058E833    90              NOP
    0058E834    90              NOP
    0058E835    90              NOP
    0058E836    58              POP EAX
    0058E837    FF95 24854100   CALL DWORD PTR SS:[EBP+418524]            ; LoadLibraryA
    0058E83D    EB 17           JMP SHORT 0058E856
    0058E83F    90              NOP
    0058E840    90              NOP
    0058E841    90              NOP
    0058E842    B8 E8030000     MOV EAX,3E8
    0058E847    E8 82D0FFFF     CALL <Get_RND_VALUE>
    0058E84C    8DBD 615D4000   LEA EDI,DWORD PTR SS:[EBP+405D61]
    0058E852    03F8            ADD EDI,EAX
    0058E854    AB              STOS DWORD PTR ES:[EDI]
    0058E855    58              POP EAX
    0058E856    8BD8            MOV EBX,EAX                               ; dllis loaded
    0058E858    8985 16204000   MOV DWORD PTR SS:[EBP+402016],EAX
    0058E85E    B8 BBFC4000     MOV EAX,0040FCBB
    0058E863    BA 07FD4000     MOV EDX,0040FD07                          ; /EnumWindows
    0058E868    E8 BDD8FFFF     CALL <sub_GetProcAddress>                 ; \GetProcAddressFun
    0058E86D    B8 C7FC4000     MOV EAX,0040FCC7
    0058E872    BA 0BFD4000     MOV EDX,0040FD0B                          ; /GetWindowTextA
    0058E877    E8 AED8FFFF     CALL <sub_GetProcAddress>                 ; \GetProcAddressFun
    0058E87C    B8 D6FC4000     MOV EAX,0040FCD6
    0058E881    BA 0FFD4000     MOV EDX,0040FD0F                          ; /GetClassNameA
    0058E886    E8 9FD8FFFF     CALL <sub_GetProcAddress>                 ; \GetProcAddressFun
    0058E88B    B8 E4FC4000     MOV EAX,0040FCE4
    0058E890    BA 13FD4000     MOV EDX,0040FD13                          ; /PostMessageA
    0058E895    E8 90D8FFFF     CALL <sub_GetProcAddress>                 ; \GetProcAddressFun
    0058E89A    B8 BAFD4000     MOV EAX,0040FDBA
    0058E89F    BA B6FD4000     MOV EDX,0040FDB6                          ; /wsprintfA
    0058E8A4    E8 81D8FFFF     CALL <sub_GetProcAddress>                 ; \GetProcAddressFun
    0058E8A9    B8 C8FD4000     MOV EAX,0040FDC8
    0058E8AE    BA C4FD4000     MOV EDX,0040FDC4                          ; /RegisterHotKey
    0058E8B3    E8 72D8FFFF     CALL <sub_GetProcAddress>                 ; \GetProcAddressFun
    0058E8B8    8DBD 83FB4000   LEA EDI,DWORD PTR SS:[EBP+40FB83]
    0058E8BE    8D8D F1FC4000   LEA ECX,DWORD PTR SS:[EBP+40FCF1]         ; 获取完相关的API后，清除名字
    0058E8C4    2BCF            SUB ECX,EDI                               ; 大小为16e
    0058E8C6    33C0            XOR EAX,EAX
    0058E8C8    F3:AA           REP STOS BYTE PTR ES:[EDI]                ; 清除壳API数据
    0058E8CA    C3              RETN
sub_GetProcAddress:
    0058C12A >  53              PUSH EBX                                 ; sub_GetProcAddress
    0058C12B    50              PUSH EAX
    0058C12C    52              PUSH EDX
    0058C12D    03C5            ADD EAX,EBP
    0058C12F    50              PUSH EAX
    0058C130    53              PUSH EBX
    0058C131    50              PUSH EAX
    0058C132    8B85 1C854100   MOV EAX,DWORD PTR SS:[EBP+41851C]
    0058C138    0FB600          MOVZX EAX,BYTE PTR DS:[EAX]
    0058C13B    83E8 33         SUB EAX,33
    0058C13E    3D 99000000     CMP EAX,99
    0058C143    74 10           JE SHORT <gameOver>
    0058C145    90              NOP
    0058C146    90              NOP
    0058C147    90              NOP
    0058C148    90              NOP
    0058C149    58              POP EAX                                  ; /GlobalAlloc
    0058C14A    FF95 1C854100   CALL DWORD PTR SS:[EBP+41851C]           ; \GetProcAddress
    0058C150    EB 17           JMP SHORT <save address>
    0058C152    90              NOP
    0058C153    90              NOP
    0058C154    90              NOP
    0058C155 >  B8 E8030000     MOV EAX,3E8                              ; GameOver
    0058C15A    E8 6FF7FFFF     CALL <Get_RND_VALUE>
    0058C15F    8DBD 615D4000   LEA EDI,DWORD PTR SS:[EBP+405D61]
    0058C165    03F8            ADD EDI,EAX
    0058C167    AB              STOS DWORD PTR ES:[EDI]
    0058C168    58              POP EAX
    0058C169 >  5A              POP EDX                                  ; save address
    0058C16A    894415 00       MOV DWORD PTR SS:[EBP+EDX],EAX           ; 填入正确的API
    0058C16E    58              POP EAX
    0058C16F    5B              POP EBX
    0058C170    C3              RETN
COPY_Import table:
    00590C48 >  60              PUSHAD                                    ; COPY_Import table
    00590C49    47              INC EDI
    00590C4A    7A 02           JPE SHORT 00590C4E
    00590C4C    85FB            TEST EBX,EDI
    00590C4E    8BC1            MOV EAX,ECX
    00590C50    7E 03           JLE SHORT 00590C55
    00590C52    7F 01           JG SHORT 00590C55
    00590C54  ^ 75 D3           JNZ SHORT 00590C29
    00590C56  ^ E0 E8           LOOPDNE SHORT 00590C40
    00590C58    0100            ADD DWORD PTR DS:[EAX],EAX
    00590C5A    0000            ADD BYTE PTR DS:[EAX],AL
    00590C5C  ^ 71 83           JNO SHORT 00590BE1
    00590C5E    C404FC          LES EAX,FWORD PTR SS:[ESP+EDI*8]          ; Modification of segment register
    00590C61    E8 01000000     CALL 00590C67
    00590C66  ^ 7C 83           JL SHORT 00590BEB
    ......
    00590DF3    E8 08B3FFFF     CALL <sub_GetEBP>
    00590DF8    8DB5 1C854100   LEA ESI,DWORD PTR SS:[EBP+41851C]         ; 壳输入表起始位置
    00590DFE    8DBD 33FD4000   LEA EDI,DWORD PTR SS:[EBP+40FD33]         ; 复制到目标地址的起始位置
    00590E04    B9 05000000     MOV ECX,5
    00590E09    F3:A5           REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS:>; 复制
    00590E0B    8DBD 43FD4000   LEA EDI,DWORD PTR SS:[EBP+40FD43]
    00590E11    8D85 4BFE4000   LEA EAX,DWORD PTR SS:[EBP+40FE4B]
    00590E17    AB              STOS DWORD PTR ES:[EDI]                   ; 把MessageBoxA的地址改为58AE4B
    00590E18    60              PUSHAD                                    ; 加密代码
    00590E19    E8 00000000     CALL 00590E1E
    00590E1E    5E              POP ESI
    00590E1F    83EE 06         SUB ESI,6
    00590E22    B9 25000000     MOV ECX,25
    00590E27    29CE            SUB ESI,ECX
    00590E29    BA AB6D95B2     MOV EDX,B2956DAB
    00590E2E    C1E9 02         SHR ECX,2
    00590E31    83E9 02         SUB ECX,2
    00590E34    83F9 00         /CMP ECX,0
    00590E37    7C 1A           |JL SHORT 00590E53
    00590E39    8B048E          |MOV EAX,DWORD PTR DS:[ESI+ECX*4]
    00590E3C    8B5C8E 04       |MOV EBX,DWORD PTR DS:[ESI+ECX*4+4]
    00590E40    33C3            |XOR EAX,EBX
    00590E42    C1C8 02         |ROR EAX,2
```
