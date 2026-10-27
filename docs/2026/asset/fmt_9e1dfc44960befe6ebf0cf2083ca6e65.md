
```asm
00498CF8    81B5 93280000 4>XOR DWORD PTR SS:[EBP+2893],96DF4B4A    ; [49A964]

00498D02    CD 68           INT 68                                  ; Access[FFFFFFFF] 异常

00498D10    8D85 540C0000   LEA EAX,DWORD PTR SS:[EBP+C54]          ; 498D25, SEH handler

00498D16    894424 04       MOV DWORD PTR SS:[ESP+4],EAX

00498D1A    64:8925 0000000>MOV DWORD PTR FS:[0],ESP                ; 修复 SEH

// 第15个异常

00498D21    EB 1F           JMP SHORT xPEP.00498D45

00498D25    8B6424 08       MOV ESP,DWORD PTR SS:[ESP+8]            ; 下断, shift+F9 到这里

00498D29    8B6C24 08       MOV EBP,DWORD PTR SS:[ESP+8]

00498D2D    8D85 66150000   LEA EAX,DWORD PTR SS:[EBP+1566]         ; 499637

00498D33    50              PUSH EAX

00498D34    EB 01           JMP SHORT xPEP.00498D37

00498D37    81AD 93280000 4>SUB DWORD PTR SS:[EBP+2893],3F139C44    ; [49A964]

00498D41    C3              RETN

00498D45    33DB            XOR EBX,EBX

00498D47    8BC3            MOV EAX,EBX

00498D49    66:BE 4040      MOV SI,4040

00498D4D    66:BF 0D0A      MOV DI,0A0D

00498D51    66:03FE         ADD DI,SI

00498D54    66:81C6 0706    ADD SI,607

00498D59    CC              INT3                                    ; INT 3 异常

00498D5A    90              NOP

00499637    64:8F05 0000000>POP DWORD PTR FS:[0]                     ; 0012FFE0

0049963E    83C4 04         ADD ESP,4

00499641    5D              POP EBP

// 计算壳所用 API 的函数

00499642    EB 00           JMP SHORT xPEP.00499644

00499644    8BDD            MOV EBX,EBP                              ; xPEP.004980D1

00499646    81ED D1104000   SUB EBP,xPEP.004010D1                    ; 97000

0049964C    9C              PUSHFD

0049964D    60              PUSHAD

0049964E    8D85 801E4000   LEA EAX,DWORD PTR SS:[EBP+401E80]        ; 498E80

00499654    FFD0            CALL EAX                                 ; 计算壳所用 API 的函数, F8, 见  最后 NOTE 3

// GetVersionEx

        typedef struct _OSVERSIONINFO{
                    DWORD dwOSVersionInfoSize;
                    DWORD dwMajorVersion;
                    DWORD dwMinorVersion;
                    DWORD dwBuildNumber;
                    DWORD dwPlatformId;
                    TCHAR szCSDVersion[ 128 ];
        } OSVERSIONINFO

00499656    68 94000000     PUSH 94

0049965B    5B              POP EBX

0049965C    2BE3            SUB ESP,EBX

0049965E    8BF4            MOV ESI,ESP

00499660    891E            MOV DWORD PTR DS:[ESI],EBX

00499662    56              PUSH ESI

00499663    FF95 641F4000   CALL DWORD PTR SS:[EBP+401F64]           ; kernel32.GetVersionExA

00499669    03E3            ADD ESP,EBX

0049966B    837E 10 02      CMP DWORD PTR DS:[ESI+10],2              ; PlatformID=2 表示 Win32_NT

0049966F   /75 0C           JNZ SHORT xPEP.0049967D

00499671   |C785 E8204000 0>MOV DWORD PTR SS:[EBP+4020E8],1          ; [4990E8] 是否 NT 的标志

0049967B   |EB 0A           JMP SHORT xPEP.00499687

0049967D    C785 E8204000 0>MOV DWORD PTR SS:[EBP+4020E8],0

00499687    83BD E8204000 0>CMP DWORD PTR SS:[EBP+4020E8],0

0049968E    0F84 A4000000   JE xPEP.00499738                         ; 不是 NT 跳走, 可以省很多事哦.

// 利用 Event 来进行同步 ???

00499694    E8 0D000000     CALL xPEP.004996A6                       ; F7

004996A6    5F              POP EDI                                  ; xPEP.00499699 "UnpackinSaga"

004996A7    57              PUSH EDI

004996A8    6A 00           PUSH 0

004996AA    68 03001F00     PUSH 1F0003

004996AF    FF95 AD1F4000   CALL DWORD PTR SS:[EBP+401FAD]           ; kernel32.OpenEventA

        0012FF70   004996B5  ?I.   /CALL to OpenEventA from xPEP.004996AF

        0012FF74   001F0003  ..  |Access = 1F0003

        0012FF78   00000000  ....  |Inheritable = FALSE

        0012FF7C   00499699  ?I.   \EventName = "UnpackinSaga"      ; US ??????

004996B5    0BC0            OR EAX,EAX

004996B7    75 1B           JNZ SHORT xPEP.004996D4                  ; 如 Open 成功, 将一直在这里循环

004996B9    EB 0D           JMP SHORT xPEP.004996C8

004996BB    57              PUSH EDI

004996BC    6A 00           PUSH 0

004996BE    6A 00           PUSH 0

004996C0    6A 00           PUSH 0

004996C2    FF95 9C1F4000   CALL DWORD PTR SS:[EBP+401F9C]           ; kernel32.CreateEventA

004996C8    0BC0            OR EAX,EAX

004996CA  ^ 74 EF           JE SHORT xPEP.004996BB

004996CC    8985 EC204000   MOV DWORD PTR SS:[EBP+4020EC],EAX        ; [4990EC]

004996D2    EB 14           JMP SHORT xPEP.004996E8                  ; 如 Create 成功, 跳出循环

004996D4    50              PUSH EAX

004996D5    FF95 9F154000   CALL DWORD PTR SS:[EBP+40159F]           ; kernel32.CloseHandle

004996DB    68 F4010000     PUSH 1F4

004996E0    FF95 CC1F4000   CALL DWORD PTR SS:[EBP+401FCC]           ; kernel32.Sleep

004996E6  ^ EB BF           JMP SHORT xPEP.004996A7

// 利用驱动 AntiTrace

004996E8    E8 10FEFFFF     CALL xPEP.004994FD                       ; F8,  生成 C:\WINNT\system32\Saga.sys, 见最后 NOTE 2

004996ED    E8 89FEFFFF     CALL xPEP.0049957B                       ; 加载驱动,见最后Note1.  跳过这句, 修改 EIP 到 499738 , 否则以后不能下普通断点.

004996F2    CD20 8BC48B64   VxDJump 648BC48B                         ; 这句其实是 CD20 (INT 20), 调整后如下

004996F2    CD20            INT 20                                   ; 下面几句都和驱动有关, 等我学会了再来解释吧, 哈.

004996F4    8BC4            MOV EAX,ESP

004996F6    8B6424 08       MOV ESP,DWORD PTR SS:[ESP+8]

004996FA    50              PUSH EAX

004996FB    60              PUSHAD

004996FC    56              PUSH ESI

004996FD    0F014C24 FE     SIDT FWORD PTR SS:[ESP-2]

00499702    5E              POP ESI

00499703    66:8B46 18      MOV AX,WORD PTR DS:[ESI+18]

00499707    66:8B5E 1E      MOV BX,WORD PTR DS:[ESI+1E]

0049970B    66:8985 3027400>MOV WORD PTR SS:[EBP+402730],AX

00499712    66:899D 3227400>MOV WORD PTR SS:[EBP+402732],BX

00499719    33C0            XOR EAX,EAX

0049971B    66:8946 18      MOV WORD PTR DS:[ESI+18],AX

0049971F    C1E8 10         SHR EAX,10

00499722    66:8946 1E      MOV WORD PTR DS:[ESI+1E],AX

00499726    61              POPAD

00499727    5C              POP ESP

00499728    8D85 38274000   LEA EAX,DWORD PTR SS:[EBP+402738]

0049972E    50              PUSH EAX

0049972F    CF              IRETD

00499730    0000            ADD BYTE PTR DS:[EAX],AL

00499732    0000            ADD BYTE PTR DS:[EAX],AL

00499734    0000            ADD BYTE PTR DS:[EAX],AL

00499736    0000            ADD BYTE PTR DS:[EAX],AL

// 跳过驱动, 这里继续

00499738    FFB5 EC204000   PUSH DWORD PTR SS:[EBP+4020EC]           ; handle of EVENT

0049973E    FF95 9F154000   CALL DWORD PTR SS:[EBP+40159F]           ; kernel32.CloseHandle

00499744    61              POPAD

00499745    9D              POPFD

// 检测 API 是否有 CC 断点

00499746    8DBD 7B154000   LEA EDI,DWORD PTR SS:[EBP+40157B]        ; 49857B

0049974C    8BB5 C2384000   MOV ESI,DWORD PTR SS:[EBP+4038C2]        ; user32.MessageBoxA

00499752    AC              LODS BYTE PTR DS:[ESI]

00499753    04 34           ADD AL,34                                ; CC+34=00

00499755    0F84 310A0000   JE xPEP.0049A18C

0049975B    AC              LODS BYTE PTR DS:[ESI]

0049975C    04 34           ADD AL,34

0049975E    0F84 280A0000   JE xPEP.0049A18C

00499764    AC              LODS BYTE PTR DS:[ESI]

00499765    04 34           ADD AL,34

00499767    0F84 1F0A0000   JE xPEP.0049A18C

0049976D    AC              LODS BYTE PTR DS:[ESI]

0049976E    04 34           ADD AL,34

00499770    0F84 160A0000   JE xPEP.0049A18C

00499776    6A 0D           PUSH 0D                                  ; 共 14 个 API

00499778    59              POP ECX

00499779    8B37            MOV ESI,DWORD PTR DS:[EDI]

0049977B    83C7 04         ADD EDI,4

0049977E    AC              LODS BYTE PTR DS:[ESI]

0049977F    04 34           ADD AL,34

00499781    0F84 D8060000   JE xPEP.00499E5F

00499787    AC              LODS BYTE PTR DS:[ESI]

00499788    04 34           ADD AL,34

0049978A    0F84 CF060000   JE xPEP.00499E5F
```
