
```
0058EEF6    E8 AA130000      CALL <Sub_Disposal IAT>

0058EEFB >   66:C785 2CF5400>MOV WORD PTR SS:[EBP+40F52C],0

0058EF04    83BD 4F814100 0>CMP DWORD PTR SS:[EBP+41814F],0          ;   这里判断是否用了 ACPROTECT 定义的机器码函数那个选项,是否用 RSA1024

0058EF0B    0F84 E5010000    JE 0058F0F6                               ; 没有用则跳

0058EF11    8DB5 98F34000    LEA ESI,DWORD PTR SS:[EBP+40F398]         ;   如果有则先强行生成一个 CID.dll 文件

0058EF17    56               PUSH ESI                                  ; /Buffer

0058EF18    68 FF000000      PUSH 0FF                                  ; |size

0058EF1D    FF95 5BFD4000    CALL DWORD PTR SS:[EBP+40FD5B]            ; \\GetTempPathA

0058EF23    B9 FFFF0000      MOV ECX,0FFFF

0058EF28    8DBD 98F34000    LEA EDI,DWORD PTR SS:[EBP+40F398]

0058EF2E    33C0             XOR EAX,EAX

0058EF30    F2:AE            REPNE SCAS BYTE PTR ES:[EDI]              ; 获取长度

0058EF32    4F               DEC EDI

0058EF33    60               PUSHAD

0058EF34    83BD 59814100 0>CMP DWORD PTR SS:[EBP+418159],0

0058EF3B    0F84 AD000000    JE 0058EFEE

0058EF41    C707 4349442E    MOV DWORD PTR DS:[EDI],2E444943

0058EF47    C747 04 646C6C6>MOV DWORD PTR DS:[EDI+4],6C6C6C64

0058EF4E    C647 07 00       MOV BYTE PTR DS:[EDI+7],0

0058EF52    8DB5 98F34000    LEA ESI,DWORD PTR SS:[EBP+40F398]

0058EF58    6A 00            PUSH 0                                    ;   /hTemplateFile = NULL

0058EF5A    6A 20            PUSH 20                                   ;   |Attributes = ARCHIVE

0058EF5C    6A 02            PUSH 2                                    ;   |Mode = CREATE_ALWAYS

0058EF5E    6A 00            PUSH 0                                    ;   |pSecurity = NULL

0058EF60    6A 03            PUSH 3                                    ;   |ShareMode = FILE_SHARE_READ|FILE_SHARE_WRITE

0058EF62    68 000000C0      PUSH C0000000                             ;   |Access = GENERIC_READ|GENERIC_WRITE

0058EF67    56               PUSH ESI                                  ;   |FileName = "C:\\DOCUME~1\\ADMINI~1\\LOCALS~1\\Temp\\CID.dll"

0058EF68    FF95 2BFD4000    CALL DWORD PTR SS:[EBP+40FD2B]            ;   \\CreateFileA

0058EF6E    50               PUSH EAX                                  ; /push hFile

0058EF6F    6A 00            PUSH 0                                    ; |/pOverlapped = NULL

0058EF71    8DB5 B3F44000    LEA ESI,DWORD PTR SS:[EBP+40F4B3]         ; ||

0058EF77    56               PUSH ESI                                  ; ||pBytesWritten = MainCon.0058A4B3

0058EF78    FFB5 59814100    PUSH DWORD PTR SS:[EBP+418159]            ; ||nBytesToWrite = 0

0058EF7E    8B9D 4F814100    MOV EBX,DWORD PTR SS:[EBP+41814F]        ; ||

0058EF84    039D 46F84000    ADD EBX,DWORD PTR SS:[EBP+40F846]         ; ||

0058EF8A    2B9D 53814100    SUB EBX,DWORD PTR SS:[EBP+418153]         ; ||

0058EF90    2B9D 59814100    SUB EBX,DWORD PTR SS:[EBP+418159]         ; ||

0058EF96    53               PUSH EBX                                  ; ||Buffer = MainCon.00400000

0058EF97    50               PUSH EAX                                  ; ||hFile = 00000054 (window)

0058EF98    FF95 53FD4000    CALL DWORD PTR SS:[EBP+40FD53]            ; |\\WriteFile

0058EF9E    FF95 27FD4000    CALL DWORD PTR SS:[EBP+40FD27]            ; \\CloseHandle

0058EFA4    8DB5 98F34000    LEA ESI,DWORD PTR SS:[EBP+40F398]

0058EFAA    56               PUSH ESI                                  ;   /LibraryA Name

0058EFAB    FF95 24854100    CALL DWORD PTR SS:[EBP+418524]            ;   \\LoadLibraryA

0058EFB1    8985 3EF84000    MOV DWORD PTR SS:[EBP+40F83E],EAX         ;   hModule

0058EFB7    8DB5 5D814100    LEA ESI,DWORD PTR SS:[EBP+41815D]

0058EFBD    56               PUSH ESI                                  ;   /ProcNameOrOrdinal = "Randimize"

0058EFBE    FFB5 3EF84000    PUSH DWORD PTR SS:[EBP+40F83E]            ;   |hModule = NULL

0058EFC4    FF95 1C854100    CALL DWORD PTR SS:[EBP+41851C]            ;   \\GetProcAddress

0058EFCA    FFB5 67814100    PUSH DWORD PTR SS:[EBP+418167]

0058EFD0    FFD0             CALL EAX                                  ;   获取该 API 的地址后，执行该函数

0058EFD2    8985 5D814100    MOV DWORD PTR SS:[EBP+41815D],EAX

0058EFD8    C785 61814100 0>MOV DWORD PTR SS:[EBP+418161],0

0058EFE2    FFB5 3EF84000    PUSH DWORD PTR SS:[EBP+40F83E]            ;   /hModule

0058EFE8    FF95 57FD4000    CALL DWORD PTR SS:[EBP+40FD57]            ;   \\FreeLibraryA

0058EFEE    61               POPAD

0058EFEF    60               PUSHAD

0058EFF0    6A 00            PUSH 0

0058EFF2    FF95 20854100    CALL DWORD PTR SS:[EBP+418520]            ;   GetModuleHandleA

0058EFF8    8DB5 8BF14000    LEA ESI,DWORD PTR SS:[EBP+40F18B]

0058EFFE    56               PUSH ESI

0058EFFF    50               PUSH EAX

0058F000    AD               LODS DWORD PTR DS:[ESI]

0058F001    93               XCHG EAX,EBX

0058F002    AD               LODS DWORD PTR DS:[ESI]

0058F003    3BC3             CMP EAX,EBX

0058F005    75 09            JNZ SHORT 0058F010

0058F007    90               NOP

0058F008    90               NOP

0058F009    90               NOP

0058F00A    90               NOP

0058F00B    EB 20            JMP SHORT 0058F02D

0058F00D    90               NOP

0058F00E    90               NOP

0058F00F    90               NOP

0058F010    8DB5 8BF14000    LEA ESI,DWORD PTR SS:[EBP+40F18B]

0058F016    FF95 1C854100    CALL DWORD PTR SS:[EBP+41851C]

0058F01C    8985 8BF14000    MOV DWORD PTR SS:[EBP+40F18B],EAX

0058F022    8985 8FF14000    MOV DWORD PTR SS:[EBP+40F18F],EAX

0058F028    EB 05            JMP SHORT 0058F02F

0058F02A    90               NOP

0058F02B    90               NOP

0058F02C    90               NOP

0058F02D    58               POP EAX

0058F02E    5E               POP ESI

0058F02F    8B85 8BF14000    MOV EAX,DWORD PTR SS:[EBP+40F18B]

0058F035    0BC0             OR EAX,EAX

0058F037    74 1A            JE SHORT 0058F053

0058F039    90               NOP

0058F03A    90               NOP

0058F03B    90               NOP

0058F03C    90               NOP

0058F03D    FF95 8BF14000    CALL DWORD PTR SS:[EBP+40F18B]

0058F043    8985 5D814100    MOV DWORD PTR SS:[EBP+41815D],EAX

0058F049    C785 61814100 0>MOV DWORD PTR SS:[EBP+418161],0

0058F053    61               POPAD

0058F054    8DB5 B7F44000    LEA ESI,DWORD PTR SS:[EBP+40F4B7]         ;   生成%tmp%\\perplex.dll

0058F05A    B9 0B000000      MOV ECX,0B

0058F05F    F3:A4            REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[>

0058F061    8DB5 98F34000    LEA ESI,DWORD PTR SS:[EBP+40F398]

0058F067    6A 00            PUSH 0

0058F069    6A 20            PUSH 20

0058F06B    6A 02            PUSH 2

0058F06D    6A 00            PUSH 0

0058F06F    6A 03            PUSH 3

0058F071    68 000000C0      PUSH C0000000

0058F076    56               PUSH ESI

0058F077    FF95 2BFD4000    CALL DWORD PTR SS:[EBP+40FD2B]            ;   CreateFileA

0058F07D    0BC0             OR EAX,EAX

0058F07F    74 75            JE SHORT 0058F0F6

0058F081    90               NOP

0058F082    90               NOP

0058F083    90               NOP

0058F084    90               NOP

0058F085    50               PUSH EAX

0058F086    6A 00            PUSH 0

0058F088    8DB5 B3F44000    LEA ESI,DWORD PTR SS:[EBP+40F4B3]
```
