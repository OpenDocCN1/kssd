
```asm
00401847  |.  74 04         JE SHORT Samples.0040184D

00401849  |.  3C 20         CMP AL,20

0040184B  |.^ 76 F2         JBE SHORT Samples.0040183F

0040184D  |>  895D D0       MOV DWORD PTR SS:[EBP-30],EBX

00401850  |.  8D45 A4       LEA EAX,DWORD PTR SS:[EBP-5C]

00401853  |.  50            PUSH EAX

00401854  |.  FF15 04204000 CALL DWORD PTR DS:[<&KERNEL32.GetStartupInfoA>]

0040185A  |.  F645 D0 01    TEST BYTE PTR SS:[EBP-30],1

0040185E  |.  74 11         JE SHORT Samples.00401871

00401860  |.  0FB745 D4     MOVZX EAX,WORD PTR SS:[EBP-2C]

00401864  |.  EB 0E         JMP SHORT Samples.00401874

00401866  |>  803E 20       /CMP BYTE PTR DS:[ESI],20

00401869  |.^ 76 D8         |JBE SHORT Samples.00401843

0040186B  |.  46            |INC ESI

0040186C  |.  8975 8C       |MOV DWORD PTR SS:[EBP-74],ESI

0040186F  |.^ EB F5         \JMP SHORT Samples.00401866

00401871  |>  6A 0A         PUSH 0A

00401873  |.  58            POP EAX

00401874  |>  50            PUSH EAX

00401875  |.  56            PUSH ESI

00401876  |.  53            PUSH EBX

00401877  |.  53            PUSH EBX

00401878  |.  FF15 00204000 CALL DWORD PTR DS:[<&KERNEL32.GetModuleHandleA>]

0040187E  |.  50            PUSH EAX

0040187F  |.  E8 5E000000   CALL Samples.004018E2

00401884  |.  8945 98       MOV DWORD PTR SS:[EBP-68],EAX

00401887  |.  50            PUSH EAX

00401888  |.  FF15 78214000 CALL DWORD PTR DS:[<&MSVCRT.exit>]         ; "exit"函数

0040188E  |.  8B45 EC       MOV EAX,DWORD PTR SS:[EBP-14]

00401891  |.  8B08          MOV ECX,DWORD PTR DS:[EAX]

00401893  |.  8B09          MOV ECX,DWORD PTR DS:[ECX]

00401895  |.  894D 88       MOV DWORD PTR SS:[EBP-78],ECX

00401898  |.  50            PUSH EAX

00401899  |.  51            PUSH ECX

0040189A  |.  E8 15000000   CALL <JMP.&MSVCRT.\_XcptFilter>

0040189F  |.  59            POP ECX

004018A0  |.  59            POP ECX

004018A1  \.  C3            RETN

              .

              .

              .

77D2F383 >  8BFF            MOV EDI,EDI                                ; (01)[F9]后来到这里，我们先按[F2]取消掉此处的断点，再按[Alt + F9]返回到用户代码领空。

77D2F385    55              PUSH EBP

77D2F386    8BEC            MOV EBP,ESP

77D2F388    56              PUSH ESI

77D2F389    8B75 0C         MOV ESI,DWORD PTR SS:[EBP+C]

77D2F38C    F7C6 0000FEFF   TEST ESI,FFFE0000

77D2F392    0F85 981D0100   JNZ USER32.77D41130

77D2F398    8B4D 08         MOV ECX,DWORD PTR SS:[EBP+8]

77D2F39B    83F9 FF         CMP ECX,-1

77D2F39E    0F84 82F10000   JE USER32.77D3E526

77D2F3A4    81F9 FFFF0000   CMP ECX,0FFFF

77D2F3AA    0F84 76F10000   JE USER32.77D3E526

77D2F3B0    E8 1B91FEFF     CALL USER32.77D184D0

77D2F3B5    85C0            TEST EAX,EAX

77D2F3B7    0F84 62F10000   JE USER32.77D3E51F

77D2F3BD    6A 01           PUSH 1

77D2F3BF    FF75 14         PUSH DWORD PTR SS:[EBP+14]

77D2F3C2    FF75 10         PUSH DWORD PTR SS:[EBP+10]

77D2F3C5    56              PUSH ESI

77D2F3C6    50              PUSH EAX

77D2F3C7    E8 07C4FEFF     CALL USER32.77D1B7D3

77D2F3CC    5E              POP ESI

77D2F3CD    5D              POP EBP

77D2F3CE    C2 1000         RETN 10

              .

              .

              .

00401280   .  64:A1 0000000>MOV EAX,DWORD PTR FS:[0]                   ;  (02)这里就是“OnInitDialog()”函数的入口处。

00401286   .  6A FF         PUSH -1

00401288   .  68 90194000   PUSH Samples.00401990

0040128D   .  50            PUSH EAX

0040128E   .  64:8925 00000>MOV DWORD PTR FS:[0],ESP

00401295   .  83EC 08       SUB ESP,8

00401298   .  56            PUSH ESI

00401299   .  57            PUSH EDI

0040129A   .  8BF1          MOV ESI,ECX

0040129C   .  E8 8B030000   CALL <JMP.&MFC42.#4710\_?OnInitDialog@CDialog>

004012A1   .  8B46 20       MOV EAX,DWORD PTR DS:[ESI+20]              ; (03)[Alt + F9]后程序返回到这里。向下找两处连续的“SendMessageA”函数调用。

004012A4   .  6A 00         PUSH 0                                     ; /Revert = FALSE

004012A6   .  50            PUSH EAX                                   ; |hWnd

004012A7   .  FF15 C0214000 CALL DWORD PTR DS:[<&USER32.GetSystemMenu>]; \GetSystemMenu

004012AD   .  50            PUSH EAX

004012AE   .  E8 39040000   CALL <JMP.&MFC42.#2863\_?FromHandle@CMenu>

004012B3   .  8BF8          MOV EDI,EAX

004012B5   .  85FF          TEST EDI,EDI

004012B7   .  74 5E         JE SHORT Samples.00401317

004012B9   .  8D4C24 08     LEA ECX,DWORD PTR SS:[ESP+8]

004012BD   .  E8 24040000   CALL <JMP.&MFC42.#540\_?CString>

004012C2   .  6A 65         PUSH 65

004012C4   .  8D4C24 0C     LEA ECX,DWORD PTR SS:[ESP+C]

004012C8   .  C74424 1C 000>MOV DWORD PTR SS:[ESP+1C],0

004012D0   .  E8 0B040000   CALL <JMP.&MFC42.#4160\_?LoadStringA@CString>

004012D5   .  8B4C24 08     MOV ECX,DWORD PTR SS:[ESP+8]

004012D9   .  8B41 F8       MOV EAX,DWORD PTR DS:[ECX-8]

004012DC   .  85C0          TEST EAX,EAX

004012DE   .  74 26         JE SHORT Samples.00401306

004012E0   .  8B57 04       MOV EDX,DWORD PTR DS:[EDI+4]

004012E3   .  53            PUSH EBX

004012E4   .  8B1D BC214000 MOV EBX,DWORD PTR DS:[<&USER32.AppendMenuA>]

004012EA   .  6A 00         PUSH 0                                     ; /pItem = NULL

004012EC   .  6A 00         PUSH 0                                     ; |ItemID = 0

004012EE   .  68 00080000   PUSH 800                                   ; |Flags = MF\_BYCOMMAND|MF\_SEPARATOR|MF\_ENABLED|MF\_STRING

004012F3   .  52            PUSH EDX                                   ; |hMenu

004012F4   .  FFD3          CALL EBX                                   ; \AppendMenuA

004012F6   .  8B4424 0C     MOV EAX,DWORD PTR SS:[ESP+C]

004012FA   .  8B4F 04       MOV ECX,DWORD PTR DS:[EDI+4]

004012FD   .  50            PUSH EAX                                   ; /pItem

004012FE   .  6A 10         PUSH 10                                    ; |ItemID = 10 (16.)

00401300   .  6A 00         PUSH 0                                     ; |Flags = MF\_BYCOMMAND|MF\_ENABLED|MF\_STRING

00401302   .  51            PUSH ECX                                   ; |hMenu

00401303   .  FFD3          CALL EBX                                   ; \AppendMenuA

00401305   .  5B            POP EBX

00401306   >  8D4C24 08     LEA ECX,DWORD PTR SS:[ESP+8]

0040130A   .  C74424 18 FFF>MOV DWORD PTR SS:[ESP+18],-1

00401312   .  E8 D3020000   CALL <JMP.&MFC42.#800\_?CString>

00401317   >  8B46 60       MOV EAX,DWORD PTR DS:[ESI+60]

0040131A   .  8B56 20       MOV EDX,DWORD PTR DS:[ESI+20]

0040131D   .  8B3D DC214000 MOV EDI,DWORD PTR DS:[<&USER32.SendMessageA>];  USER32.SendMessageA

00401323   .  50            PUSH EAX                                    ; /lParam

00401324   .  6A 01         PUSH 1                                      ; |wParam = 1

00401326   .  68 80000000   PUSH 80                                     ; |Message = WM\_SETICON

0040132B   .  52            PUSH EDX                                    ; |hWnd

0040132C   .  FFD7          CALL EDI                                    ; \SendMessageA  //(04)第一处“SendMessageA”函数调用。

0040132E   .  8B46 60       MOV EAX,DWORD PTR DS:[ESI+60]

00401331   .  50            PUSH EAX                                    ; /lParam

00401332   .  8B46 20       MOV EAX,DWORD PTR DS:[ESI+20]               ; |

00401335   .  6A 00         PUSH 0                                      ; |wParam = 0

00401337   .  68 80000000   PUSH 80                                     ; |Message = WM\_SETICON

0040133C   .  50            PUSH EAX                                    ; |hWnd

0040133D   .  FFD7          CALL EDI                                    ; \SendMessageA  //(05)第二处“SendMessageA”函数调用。

0040133F   .  68 5C304000   PUSH Samples.0040305C                       ;  (06)第二处初始化位置：OnInitDialog()函数中的用户代码开头！

00401344   .  8D4C24 10     LEA ECX,DWORD PTR SS:[ESP+10]

00401348   .  E8 BB020000   CALL <JMP.&MFC42.#537\_?CString>

0040134D   .  8B4C24 0C     MOV ECX,DWORD PTR SS:[ESP+C]

00401351   .  6A 00         PUSH 0

00401353   .  6A 00         PUSH 0
```
