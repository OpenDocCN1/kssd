
```
    0058EFAA    56              PUSH ESI                                 ;         /LibraryA Name

    0058EFAB    FF95 24854100   CALL DWORD PTR SS:[EBP+418524]           ;         \LoadLibraryA

    0058EFB1    8985 3EF84000   MOV DWORD PTR SS:[EBP+40F83E],EAX        ;         hModule

    0058EFB7    8DB5 5D814100   LEA ESI,DWORD PTR SS:[EBP+41815D]

    0058EFBD    56              PUSH ESI                                 ;         /ProcNameOrOrdinal = "Randimize"

    0058EFBE    FFB5 3EF84000   PUSH DWORD PTR SS:[EBP+40F83E]           ;         |hModule = NULL

    0058EFC4    FF95 1C854100   CALL DWORD PTR SS:[EBP+41851C]           ;         \GetProcAddress

    0058EFCA    FFB5 67814100   PUSH DWORD PTR SS:[EBP+418167]

    0058EFD0    FFD0            CALL EAX                                 ;         获取该API的地址后，执行该函数

    0058EFD2    8985 5D814100   MOV DWORD PTR SS:[EBP+41815D],EAX

    0058EFD8    C785 61814100 0>MOV DWORD PTR SS:[EBP+418161],0

    0058EFE2    FFB5 3EF84000   PUSH DWORD PTR SS:[EBP+40F83E]           ;         /hModule

    0058EFE8    FF95 57FD4000   CALL DWORD PTR SS:[EBP+40FD57]           ;         \FreeLibraryA

    0058EFEE    61              POPAD

    0058EFEF    60              PUSHAD

    0058EFF0    6A 00           PUSH 0

    0058EFF2    FF95 20854100   CALL DWORD PTR SS:[EBP+418520]           ;         GetModuleHandleA

    0058EFF8    8DB5 8BF14000   LEA ESI,DWORD PTR SS:[EBP+40F18B]

    0058EFFE    56              PUSH ESI

    0058EFFF    50              PUSH EAX

    0058F000    AD              LODS DWORD PTR DS:[ESI]

    0058F001    93              XCHG EAX,EBX

    0058F002    AD              LODS DWORD PTR DS:[ESI]

    0058F003    3BC3            CMP EAX,EBX

    0058F005    75 09           JNZ SHORT 0058F010

    0058F007    90              NOP

    0058F008    90              NOP

    0058F009    90              NOP

    0058F00A    90              NOP

    0058F00B    EB 20           JMP SHORT 0058F02D

    0058F00D    90              NOP

    0058F00E    90              NOP

    0058F00F    90              NOP

    0058F010    8DB5 8BF14000   LEA ESI,DWORD PTR SS:[EBP+40F18B]

    0058F016    FF95 1C854100   CALL DWORD PTR SS:[EBP+41851C]

    0058F01C    8985 8BF14000   MOV DWORD PTR SS:[EBP+40F18B],EAX

    0058F022    8985 8FF14000   MOV DWORD PTR SS:[EBP+40F18F],EAX

    0058F028    EB 05           JMP SHORT 0058F02F

    0058F02A    90              NOP

    0058F02B    90              NOP

    0058F02C    90              NOP

    0058F02D    58              POP EAX

    0058F02E    5E              POP ESI

    0058F02F    8B85 8BF14000   MOV EAX,DWORD PTR SS:[EBP+40F18B]

    0058F035    0BC0            OR EAX,EAX

    0058F037    74 1A           JE SHORT 0058F053

    0058F039    90              NOP

    0058F03A    90              NOP

    0058F03B    90              NOP

    0058F03C    90              NOP

    0058F03D    FF95 8BF14000   CALL DWORD PTR SS:[EBP+40F18B]

    0058F043    8985 5D814100   MOV DWORD PTR SS:[EBP+41815D],EAX

    0058F049    C785 61814100 0>MOV DWORD PTR SS:[EBP+418161],0

    0058F053    61              POPAD

    0058F054    8DB5 B7F44000   LEA ESI,DWORD PTR SS:[EBP+40F4B7]        ;         生成%tmp%\perplex.dll

    0058F05A    B9 0B000000     MOV ECX,0B

    0058F05F    F3:A4           REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[>

    0058F061    8DB5 98F34000   LEA ESI,DWORD PTR SS:[EBP+40F398]

    0058F067    6A 00           PUSH 0

    0058F069    6A 20           PUSH 20

    0058F06B    6A 02           PUSH 2

    0058F06D    6A 00           PUSH 0

    0058F06F    6A 03           PUSH 3

    0058F071    68 000000C0     PUSH C0000000

    0058F076    56              PUSH ESI

    0058F077    FF95 2BFD4000   CALL DWORD PTR SS:[EBP+40FD2B]           ;         CreateFileA

    0058F07D    0BC0            OR EAX,EAX

    0058F07F    74 75           JE SHORT 0058F0F6

    0058F081    90              NOP

    0058F082    90              NOP

    0058F083    90              NOP

    0058F084    90              NOP

    0058F085    50              PUSH EAX

    0058F086    6A 00           PUSH 0

    0058F088    8DB5 B3F44000   LEA ESI,DWORD PTR SS:[EBP+40F4B3]

    0058F08E    56              PUSH ESI

    0058F08F    FFB5 53814100   PUSH DWORD PTR SS:[EBP+418153]

    0058F095    8B9D 4F814100   MOV EBX,DWORD PTR SS:[EBP+41814F]

    0058F09B    039D 46F84000   ADD EBX,DWORD PTR SS:[EBP+40F846]

    0058F0A1    2B9D 53814100   SUB EBX,DWORD PTR SS:[EBP+418153]

    0058F0A7    53              PUSH EBX

    0058F0A8    50              PUSH EAX

    0058F0A9    FF95 53FD4000   CALL DWORD PTR SS:[EBP+40FD53]

    0058F0AF    FF95 27FD4000   CALL DWORD PTR SS:[EBP+40FD27]

    0058F0B5    8DB5 98F34000   LEA ESI,DWORD PTR SS:[EBP+40F398]

    0058F0BB    56              PUSH ESI                                 ;         /library name

    0058F0BC    FF95 24854100   CALL DWORD PTR SS:[EBP+418524]           ;         \LoadLibraryA

    0058F0C2    0BC0            OR EAX,EAX

    0058F0C4    74 30           JE SHORT 0058F0F6                        ; 如        果载入失败则忽略并跳去执行后面的代码

    0058F0C6    90              NOP

    0058F0C7    90              NOP

    0058F0C8    90              NOP

    0058F0C9    90              NOP

    0058F0CA    8985 3EF84000   MOV DWORD PTR SS:[EBP+40F83E],EAX

    0058F0D0    8DB5 C4F44000   LEA ESI,DWORD PTR SS:[EBP+40F4C4]        ;         载入成功则获取zcf\_decrypt的地址

    0058F0D6    56              PUSH ESI                                 ;         /name

    0058F0D7    FFB5 3EF84000   PUSH DWORD PTR SS:[EBP+40F83E]           ;         |hmodule

    0058F0DD    FF95 1C854100   CALL DWORD PTR SS:[EBP+41851C]           ;         \GetProcAddress

    0058F0E3    8985 C4F44000   MOV DWORD PTR SS:[EBP+40F4C4],EAX

    0058F0E9    0BC0            OR EAX,EAX

    0058F0EB    74 09           JE SHORT 0058F0F6

    0058F0ED    90              NOP

    0058F0EE    90              NOP

    0058F0EF    90              NOP

    0058F0F0    90              NOP

    0058F0F1    E8 D5F7FFFF     CALL <Reg\_Info>

    0058F0F6    60              PUSHAD                                   ; 把        前面的代码加密回去

    0058F0F7    E8 00000000     CALL 0058F0FC

    0058F0FC    5E              POP ESI

    0058F0FD    83EE 06         SUB ESI,6

    0058F100    B9 12020000     MOV ECX,212

    0058F105    29CE            SUB ESI,ECX

    0058F107    BA 9A465152     MOV EDX,5251469A

    0058F10C    C1E9 02         SHR ECX,2

    0058F10F    83E9 02         SUB ECX,2

    0058F112    83F9 00         CMP ECX,0

    0058F115    7C 1A           JL SHORT 0058F131

    0058F117    8B048E          MOV EAX,DWORD PTR DS:[ESI+ECX\*4]

    0058F11A    8B5C8E 04       MOV EBX,DWORD PTR DS:[ESI+ECX\*4+4]

    0058F11E    03C3            ADD EAX,EBX

    0058F120    C1C8 1A         ROR EAX,1A

    0058F123    2BC2            SUB EAX,EDX

    0058F125    81C2 4F6B073C   ADD EDX,3C076B4F

    0058F12B    89048E          MOV DWORD PTR DS:[ESI+ECX\*4],EAX

    0058F12E    49              DEC ECX

    0058F12F  ^ EB E1           JMP SHORT 0058F112

    0058F131    61              POPAD

    ......

    0058F2DC    E8 1FCEFFFF     CALL <sub\_GetEBP >                       ; 开始执行判断是否用了注册提示,是否显示机器码信息框之类的

    0058F2E1    66:81BD 2CF5400>CMP WORD PTR SS:[EBP+40F52C],0FF00

    0058F2EA    0F84 AC000000   JE 0058F39C

    0058F2F0    66:81BD 2CF5400>CMP WORD PTR SS:[EBP+40F52C],0FF01       ; 这几个判断没有认真去看

    0058F2F9    74 39           JE SHORT 0058F334

    0058F2FB    90              NOP

    0058F2FC    90              NOP

    0058F2FD    90              NOP

    0058F2FE    90              NOP

    0058F2FF    66:81BD 2CF5400>CMP WORD PTR SS:[EBP+40F52C],0FF02

    0058F308    74 53           JE SHORT 0058F35D

    ......

    0058F4E0    60              PUSHAD                                   ; 加密回去

    0058F4E1    E8 00000000     CALL 0058F4E6

    0058F4E6    5E              POP ESI
```
