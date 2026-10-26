
```
    0042572E    93              XCHG EAX,EBX

    0042572F    6A 00           PUSH 0

    00425731    8D85 00104000   LEA EAX,DWORD PTR SS:[EBP+401000]

    00425737    50              PUSH EAX

    00425738    56              PUSH ESI

    00425739    53              PUSH EBX

    0042573A    57              PUSH EDI

    0042573B    FF95 FC154000   CALL DWORD PTR SS:[EBP+4015FC]

    00425741    8BC3            MOV EAX,EBX

    00425743    8BCE            MOV ECX,ESI

    00425745    60              PUSHAD

    00425746    E8 3E060000     CALL 00425D89

    0042574B    3985 30164000   CMP DWORD PTR SS:[EBP+401630],EAX

    00425751    74 27           JE SHORT 0042577A

    00425753    90              NOP

    00425754    90              NOP

    00425755    90              NOP

    00425756    90              NOP

    00425757    60              PUSHAD

    00425758    E8 15EAFFFF     CALL <GetEBp>

    0042575D    B8 00010000     MOV EAX,100

    00425762    E8 18EAFFFF     CALL 0042417F

    00425767    8BC8            MOV ECX,EAX

    00425769    8DBD 9A404000   LEA EDI,DWORD PTR SS:[EBP+40409A]

    0042576F    03F8            ADD EDI,EAX

    00425771    E8 1DEAFFFF     CALL 00424193

    00425776    AB              STOS DWORD PTR ES:[EDI]

    00425777  ^ E2 F8           LOOPD SHORT 00425771

    00425779    61              POPAD

    0042577A    61              POPAD

    0042577B    8D85 A42C4000   LEA EAX,DWORD PTR SS:[EBP+402CA4]

    00425781    50              PUSH EAX

    00425782    C3              RETN

    00425783    53              PUSH EBX

    00425784    FF95 B0154000   CALL DWORD PTR SS:[EBP+4015B0]

    0042578A    96              XCHG EAX,ESI

    0042578B    50              PUSH EAX

    0042578C    57              PUSH EDI

    0042578D    FF95 D4154000   CALL DWORD PTR SS:[EBP+4015D4]

    00425793    58              POP EAX

    00425794    60              PUSHAD

    00425795    E8 00000000     CALL 0042579A

    0042579A    5E              POP ESI

    0042579B    83EE 06         SUB ESI,6

    0042579E    B9 C3000000     MOV ECX,0C3

    004257A3    29CE            SUB ESI,ECX

    004257A5    BA 48A6FC64     MOV EDX,64FCA648

    004257AA    C1E9 02         SHR ECX,2

    004257AD    83E9 02         SUB ECX,2

    004257B0    83F9 00         CMP ECX,0

    004257B3    7C 1A           JL SHORT 004257CF

    004257B5    8B048E          MOV EAX,DWORD PTR DS:[ESI+ECX\*4]

    004257B8    8B5C8E 04       MOV EBX,DWORD PTR DS:[ESI+ECX\*4+4]

    004257BC    2BC3            SUB EAX,EBX

    004257BE    C1C0 1D         ROL EAX,1D

    004257C1    03C2            ADD EAX,EDX

    004257C3    81C2 5E6AB05C   ADD EDX,5CB06A5E

    004257C9    89048E          MOV DWORD PTR DS:[ESI+ECX\*4],EAX

    004257CC    49              DEC ECX

    004257CD  ^ EB E1           JMP SHORT 004257B0

    004257CF    61              POPAD

    004257D0    61              POPAD

    004257D1    C3              RETN

sub\_INT 1\_Check\_Debug:

    0042170B >  60              PUSHAD                                     ; sub\_INT 1\_Check\_Debug

    0042170C    4D              DEC EBP

    0042170D    50              PUSH EAX

    0042170E    E8 01000000     CALL 00421714

    00421713  ^ 71 83           JNO SHORT 00421698

    00421715    C40458          LES EAX,FWORD PTR DS:[EAX+EBX\*2]         ; Modification of segment register

    00421718    66:BD D8B7      MOV BP,0B7D8

    0042171C    7C 03           JL SHORT 00421721

    0042171E    7D 01           JGE SHORT 00421721

    00421720  - E9 F850E801     JMP 022A681D

    00421725    0000            ADD BYTE PTR DS:[EAX],AL

    00421727    00EB            ADD BL,CH

    ......

    004218B6    E8 72F6FFFF     CALL <GetEBP>

    004218BB    C685 711E4000 C>MOV BYTE PTR SS:[EBP+401E71],0C3

    004218C2    8CC8            MOV AX,CS

    004218C4    A8 04           TEST AL,4                                ; 如果系统是Winnt的，则通过int 1检测调试器

    004218C6    75 5A           JNZ SHORT <OS is Win9x>

    004218C8    90              NOP

    004218C9    90              NOP

    004218CA    90              NOP

    004218CB    90              NOP

    004218CC    E8 0E000000     CALL 004218DF

    004218D1    8B5C24 0C       MOV EBX,DWORD PTR SS:[ESP+C]

    004218D5    8383 B8000000 0>ADD DWORD PTR DS:[EBX+B8],2

    004218DC    33C0            XOR EAX,EAX

    004218DE    C3              RETN

    004218DF    64:67:FF36 0000 PUSH DWORD PTR FS:[0]

    004218E5    64:67:8926 0000 MOV DWORD PTR FS:[0],ESP

    004218EB    33C0            XOR EAX,EAX

    004218ED    CD 01           INT 1

    004218EF    40              INC EAX

    004218F0    40              INC EAX

    004218F1    0BC0            OR EAX,EAX

    004218F3    75 27           JNZ SHORT 0042191C                       ; 如果没有发现则跳

    004218F5    90              NOP

    004218F6    90              NOP

    004218F7    90              NOP

    004218F8    90              NOP

    004218F9    60              PUSHAD

    004218FA    E8 2EF6FFFF     CALL <GetEBP>

    004218FF    B8 00010000     MOV EAX,100

    00421904    E8 31F6FFFF     CALL 00420F3A

    00421909    8BC8            MOV ECX,EAX

    0042190B    8DBD 9A404000   LEA EDI,DWORD PTR SS:[EBP+40409A]

    00421911    03F8            ADD EDI,EAX

    00421913    E8 36F6FFFF     CALL 00420F4E

    00421918    AB              STOS DWORD PTR ES:[EDI]

    00421919  ^ E2 F8           LOOPD SHORT 00421913

    0042191B    61              POPAD

    0042191C    33C0            XOR EAX,EAX

    0042191E    64:8F00         POP DWORD PTR FS:[EAX]

    00421921    58              POP EAX

    00421922 >  60              PUSHAD                                   ; OS         is Win9x

    00421923    E8 00000000     CALL 00421928

    00421928    5E              POP ESI

    00421929    83EE 06         SUB ESI,6

    0042192C    B9 6C000000     MOV ECX,6C

    00421931    29CE            SUB ESI,ECX

    00421933    BA 2B3C3C6D     MOV EDX,6D3C3C2B

    00421938    C1E9 02         SHR ECX,2

    0042193B    83E9 02         SUB ECX,2

    0042193E    83F9 00         CMP ECX,0

    00421941    7C 1A           JL SHORT 0042195D

    00421943    8B048E          MOV EAX,DWORD PTR DS:[ESI+ECX\*4]

    00421946    8B5C8E 04       MOV EBX,DWORD PTR DS:[ESI+ECX\*4+4]

    0042194A    03C3            ADD EAX,EBX

    0042194C    C1C8 1B         ROR EAX,1B

    0042194F    03C2            ADD EAX,EDX

    00421951    81F2 C6E14F4D   XOR EDX,4D4FE1C6

    00421957    89048E          MOV DWORD PTR DS:[ESI+ECX\*4],EAX

    0042195A    49              DEC ECX

    0042195B  ^ EB E1           JMP SHORT 0042193E

    0042195D    61              POPAD

    0042195E    61              POPAD

    0042195F    C3              RETN

sub\_Anti\_Fake\_Unpack:

    00422027 >  60              PUSHAD                                   ; sub\_Anti\_Fake\_Unpack

    00422028    4E              DEC ESI

    00422029    87EA            XCHG EDX,EBP

    0042202B    46              INC ESI

    0042202C    50              PUSH EAX

    0042202D    E8 01000000     CALL 00422033

    00422032  ^ 77 83           JA SHORT 00421FB7

    00422034    C40458          LES EAX,FWORD PTR DS:[EAX+EBX\*2]         ; Modification of segment register

    00422037    66:8BD1         MOV DX,CX

    ......

    004221D2    E8 56EDFFFF     CALL <GetEBP>

    004221D7    C685 8D274000 C>MOV BYTE PTR SS:[EBP+40278D],0C3

    004221DE    8BB5 28164000   MOV ESI,DWORD PTR SS:[EBP+401628]

    004221E4    66:8B16         MOV DX,WORD PTR DS:[ESI]

    004221E7    66:81FA 4D5A    CMP DX,5A4D

    004221EC    0F85 B1000000   JNZ <Good way>

    004221F2    0FB756 3C       MOVZX EDX,WORD PTR DS:[ESI+3C]

    004221F6    8BFE            MOV EDI,ESI

    004221F8    03FA            ADD EDI,EDX

    004221FA    8B47 28         MOV EAX,DWORD PTR DS:[EDI+28]

    004221FD    3B85 50164000   CMP EAX,DWORD PTR SS:[EBP+401650]
```
