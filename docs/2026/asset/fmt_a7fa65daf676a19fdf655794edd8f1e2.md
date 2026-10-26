
```
    0058B024    8BC3            MOV EAX,EBX

    0058B026    60              PUSHAD

    0058B027    E9 CC000000     JMP <Case else>

    0058B02C    60              PUSHAD

    0058B02D    E8 CE100000     CALL <sub\_GetEBP >

    0058B032    8B7C24 28       MOV EDI,DWORD PTR SS:[ESP+28]

    0058B036    8B4424 30       MOV EAX,DWORD PTR SS:[ESP+30]

    0058B03A    0BC0            OR EAX,EAX                                ;         当EAX＝4的时候加密代码，eax=5的时候解密代码

    0058B03C    74 3F           JE SHORT <Case EAX==0>

    0058B03E    90              NOP

    0058B03F    90              NOP

    0058B040    90              NOP

    0058B041    90              NOP

    0058B042    48              DEC EAX

    0058B043    0BC0            OR EAX,EAX

    0058B045    74 65           JE SHORT <Case EAX==1>

    0058B047    90              NOP

    0058B048    90              NOP

    0058B049    90              NOP

    0058B04A    90              NOP

    0058B04B    48              DEC EAX

    0058B04C    0BC0            OR EAX,EAX

    0058B04E    74 68           JE SHORT <Case EAX==2>

    0058B050    90              NOP

    0058B051    90              NOP

    0058B052    90              NOP

    0058B053    90              NOP

    0058B054    48              DEC EAX

    0058B055    0BC0            OR EAX,EAX

    0058B057    74 75           JE SHORT <Case EAX==3>

    0058B059    90              NOP

    0058B05A    90              NOP

    0058B05B    90              NOP

    0058B05C    90              NOP

    0058B05D    48              DEC EAX

    0058B05E    0BC0            OR EAX,EAX

    0058B060    74 76           JE SHORT <Case EAX==4>

    0058B062    90              NOP

    0058B063    90              NOP

    0058B064    90              NOP

    0058B065    90              NOP

    0058B066    48              DEC EAX

    0058B067    0BC0            OR EAX,EAX

    0058B069    74 77           JE SHORT <Case EAX==5>

    0058B06B    90              NOP

    0058B06C    90              NOP

    0058B06D    90              NOP

    0058B06E    90              NOP

    0058B06F    48              DEC EAX

    0058B070    0BC0            OR EAX,EAX

    0058B072    74 78           JE SHORT <Case EAX==6>

    0058B074    90              NOP

    0058B075    90              NOP

    0058B076    90              NOP

    0058B077    90              NOP

    0058B078    EB 7E           JMP SHORT <Case else>

    0058B07A    90              NOP

    0058B07B    90              NOP

    0058B07C    90              NOP

    0058B07D >  8DB5 2CF54000   LEA ESI,DWORD PTR SS:[EBP+40F52C]         ;         当EAX＝0时的处理

    0058B083    B9 08000000     MOV ECX,8

    0058B088    F3:A5           REP MOVS DWORD PTR ES:[EDI],DWORD PTR     DS:>

    0058B08A    8DB5 50F54000   LEA ESI,DWORD PTR SS:[EBP+40F550]

    0058B090    B9 07000000     MOV ECX,7

    0058B095    F3:A5           REP MOVS DWORD PTR ES:[EDI],DWORD PTR     DS:>

    0058B097    4F              DEC EDI

    0058B098    803F 20         CMP BYTE PTR DS:[EDI],20

    0058B09B    75 06           JNZ SHORT 0058B0A3

    0058B09D    90              NOP

    0058B09E    90              NOP

    0058B09F    90              NOP

    0058B0A0    90              NOP

    0058B0A1  ^ EB F4           JMP SHORT 0058B097

    0058B0A3    C647 01 00      MOV BYTE PTR DS:[EDI+1],0

    0058B0A7    EB 4F           JMP SHORT <Case else>

    0058B0A9    90              NOP

    0058B0AA    90              NOP

    0058B0AB    90              NOP

    0058B0AC >  8A85 6DF74000   MOV AL,BYTE PTR SS:[EBP+40F76D]           ;         当eax==1时的处理

    0058B0B2    AA              STOS BYTE PTR ES:[EDI]

    0058B0B3    EB 43           JMP SHORT <Case else>

    0058B0B5    90              NOP

    0058B0B6    90              NOP

    0058B0B7    90              NOP

    0058B0B8 >  50              PUSH EAX                                  ;         当EAX＝＝2时的处理

    0058B0B9    8B4424 24       MOV EAX,DWORD PTR SS:[ESP+24]

    0058B0BD    8985 28F54000   MOV DWORD PTR SS:[EBP+40F528],EAX

    0058B0C3    58              POP EAX

    0058B0C4    E8 02380000     CALL <Reg\_Info>

    0058B0C9    EB 2D           JMP SHORT <Case else>

    0058B0CB    90              NOP

    0058B0CC    90              NOP

    0058B0CD    90              NOP

    0058B0CE >  E8 72000000     CALL 0058B145                             ; 当        eax=3时的处理

    0058B0D3    EB 23           JMP SHORT <Case else>

    0058B0D5    90              NOP

    0058B0D6    90              NOP

    0058B0D7    90              NOP

    0058B0D8 >  E8 9C020000     CALL <sub\_Restore\_Crypted\_Code>           ; 当        eax==4时的处理

    0058B0DD    EB 19           JMP SHORT <Case else>

    0058B0DF    90              NOP

    0058B0E0    90              NOP

    0058B0E1    90              NOP

    0058B0E2 >  E8 CA040000     CALL <sub\_Decrypt\_Code>                   ; 当        eax==5时的处理

    0058B0E7    EB 0F           JMP SHORT <Case else>

    0058B0E9    90              NOP

    0058B0EA    90              NOP

    0058B0EB    90              NOP

    0058B0EC >  8B85 5D814100   MOV EAX,DWORD PTR     SS:[EBP+41815D]         ; 当EAX==6时的处理

    0058B0F2    AB              STOS DWORD PTR ES:[EDI]

    0058B0F3    EB 03           JMP SHORT <Case else>

    0058B0F5    90              NOP

    0058B0F6    90              NOP

    0058B0F7    90              NOP

    0058B0F8 >  90              NOP                                       ;         Case else

    0058B0F9    90              NOP

    0058B0FA    90              NOP

    ......

    0058B133    90              NOP

    0058B134    61              POPAD

    0058B135    837C24 04 FF    CMP DWORD PTR SS:[ESP+4],-1

    0058B13A    74 06           JE SHORT 0058B142

    0058B13C    90              NOP

    0058B13D    90              NOP

    0058B13E    90              NOP

    0058B13F    90              NOP

    0058B140    FFE0            JMP EAX

    0058B142    C2 1000         RETN 10

sub\_CRC:

    004246F9 >  60              PUSHAD                                   ; sub\_CRC

    004246FA    E8 73FAFFFF     CALL <GetEBp>

    004246FF    C685 1A1C4000 C>MOV BYTE PTR SS:[EBP+401C1A],0C3

    00424706    E8 1B0E0000     CALL 00425526                 ;这个Call 进去　

    0042470B    61              POPAD

    0042470C    C3              RETN

    进来后:

    004256D1    E8 9CEAFFFF     CALL <GetEBp>                 ;这里和上篇里的是一样的,所以我也不多注释了:-)

    004256D6    68 20030000     PUSH 320

    004256DB    8DBD 00104000   LEA EDI,DWORD PTR SS:[EBP+401000]

    004256E1    57              PUSH EDI

    004256E2    6A 00           PUSH 0

    004256E4    FF95 20164000   CALL DWORD PTR SS:[EBP+401620]            ;GetModuleFileNameA

    004256EA    6A 00           PUSH 0

    004256EC    68 80000000     PUSH 80

    004256F1    6A 03           PUSH 3

    004256F3    6A 00           PUSH 0

    004256F5    6A 01           PUSH 1

    004256F7    68 00000080     PUSH 80000000

    004256FC    57              PUSH EDI

    004256FD    FF95 D8154000   CALL DWORD PTR SS:[EBP+4015D8]

    00425703    40              INC EAX

    00425704    0F84 8A000000   JE 00425794

    0042570A    48              DEC EAX

    0042570B    8BF8            MOV EDI,EAX

    0042570D    6A 00           PUSH 0

    0042570F    57              PUSH EDI

    00425710    FF95 24164000   CALL DWORD PTR SS:[EBP+401624]

    00425716    2B85 2C164000   SUB EAX,DWORD PTR SS:[EBP+40162C]

    0042571C    96              XCHG EAX,ESI

    0042571D    56              PUSH ESI

    0042571E    6A 40           PUSH 40

    00425720    FF95 AC154000   CALL DWORD PTR SS:[EBP+4015AC]

    00425726    85C0            TEST EAX,EAX

    00425728    74 61           JE SHORT 0042578B

    0042572A    90              NOP

    0042572B    90              NOP

    0042572C    90              NOP

    0042572D    90              NOP
```
