
```
00408472  |.  8B5424 34           MOV EDX,DWORD PTR SS:[ESP+34]              ;  "5" 第5位字符

00408476  |.  8D3480              LEA ESI,DWORD PTR DS:[EAX+EAX\*4]           ;  ESI = 5\*EAX  其中EAX为对应的第(Arg2-30+1)位数字

00408479  |.  51                  PUSH ECX                                   ;  Arg2

0040847A  |.  52                  PUSH EDX                                   ;  Arg1

0040847B  |.  D1E6                SHL ESI,1                                  ;  加倍 ESI = A\*EAX

0040847D  |.  E8 8EFEFFFF         CALL FocusMag.00408310

00408482  |.  83C4 10             ADD ESP,10

00408485  |.  03F0                ADD ESI,EAX                                ;  ESI = ESI + EAX

00408487  |.  8D4424 18           LEA EAX,DWORD PTR SS:[ESP+18]

0040848B  |.  50                  PUSH EAX                                   ; /pSystemTime

0040848C  |.  FF15 A0B04200       CALL DWORD PTR DS:[<&KERNEL32.GetSystemTim>; \GetSystemTime

00408492  |.  8B4424 18           MOV EAX,DWORD PTR SS:[ESP+18]              ;  年份/月份

00408496  |.  B9 0A000000         MOV ECX,0A

0040849B  |.  25 FFFF0000         AND EAX,0FFFF                              ;  年份

004084A0  |.  99                  CDQ                                        ;  EDX 清零

004084A1  |.  F7F9                IDIV ECX                                   ;  EAX = EAX \ A ; EDX = EAX | A

004084A3  |.  8B4424 1A           MOV EAX,DWORD PTR SS:[ESP+1A]              ;  月份/星期

004084A7  |.  33C9                XOR ECX,ECX

004084A9  |.  25 FFFF0000         AND EAX,0FFFF                              ;  月份

004084AE  |.  3BD7                CMP EDX,EDI                                ;  （年份/A）的余数 <> EDI ？

004084B0  |.  75 09               JNZ SHORT FocusMag.004084BB

004084B2  |.  3BC6                CMP EAX,ESI                                ;  月份 <> ESI ？

004084B4  |.  75 05               JNZ SHORT FocusMag.004084BB

004084B6  |.  B9 01000000         MOV ECX,1                                  ;  上面条件相等时置ECX=1

004084BB  |>  66:837C24 1E 0E     CMP WORD PTR SS:[ESP+1E],0E                ;  日 > E ？

004084C1  |.  77 1B               JA SHORT FocusMag.004084DE

004084C3  |.  46                  INC ESI

004084C4  |.  83FE 0D             CMP ESI,0D                                 ;  ESI <> D ？

004084C7  |.  75 0D               JNZ SHORT FocusMag.004084D6

004084C9  |.  47                  INC EDI

004084CA  |.  83FF 0A             CMP EDI,0A                                 ;  EDI <> A ？

004084CD  |.  75 02               JNZ SHORT FocusMag.004084D1

004084CF  |.  33FF                XOR EDI,EDI

004084D1  |>  BE 01000000         MOV ESI,1

004084D6  |>  3BD7                CMP EDX,EDI

004084D8  |.  75 04               JNZ SHORT FocusMag.004084DE

004084DA  |.  3BC6                CMP EAX,ESI                                ;  EAX = ESI ？

004084DC  |.  74 04               JE SHORT FocusMag.004084E2                 ;  若ECX=0则要跳，否则可不跳

004084DE  |>  85C9                TEST ECX,ECX

004084E0  |.  74 5B               JE SHORT FocusMag.0040853D                 ;  不能跳，ECX <> 0

004084E2  |>  83FB 05             CMP EBX,5                                  ;  EBX <= 5 ？

004084E5  |.  7E 52               JLE SHORT FocusMag.00408539                ;  最好跳

004084E7  |.  BE 05000000         MOV ESI,5

004084EC  |>  8DBC24 C0000000     /LEA EDI,DWORD PTR SS:[ESP+C0]

004084F3  |.  83C9 FF             |OR ECX,FFFFFFFF

004084F6  |.  33C0                |XOR EAX,EAX

004084F8  |.  8D56 FC             |LEA EDX,DWORD PTR DS:[ESI-4]

004084FB  |.  F2:AE               |REPNE SCAS BYTE PTR ES:[EDI]

004084FD  |.  F7D1                |NOT ECX

004084FF  |.  49                  |DEC ECX

00408500  |.  3BCA                |CMP ECX,EDX

00408502  |.  7C 39               |JL SHORT FocusMag.0040853D

00408504  |.  8A8434 BB000000     |MOV AL,BYTE PTR SS:[ESP+ESI+BB]

0040850B  |.  3C 60               |CMP AL,60

0040850D  |.  7E 02               |JLE SHORT FocusMag.00408511

0040850F  |.  04 E0               |ADD AL,0E0

00408511  |>  3C 41               |CMP AL,41

00408513  |.  7C 17               |JL SHORT FocusMag.0040852C

00408515  |.  3C 5A               |CMP AL,5A

00408517  |.  7F 13               |JG SHORT FocusMag.0040852C

00408519  |.  0FBEC0              |MOVSX EAX,AL

0040851C  |.  83E8 40             |SUB EAX,40

0040851F  |.  B9 0A000000         |MOV ECX,0A

00408524  |.  99                  |CDQ

00408525  |.  F7F9                |IDIV ECX

00408527  |.  80C2 30             |ADD DL,30

0040852A  |.  EB 02               |JMP SHORT FocusMag.0040852E

0040852C  |>  B2 30               |MOV DL,30

0040852E  |>  385434 28           |CMP BYTE PTR SS:[ESP+ESI+28],DL

00408532  |.  75 09               |JNZ SHORT FocusMag.0040853D

00408534  |.  46                  |INC ESI

00408535  |.  3BF3                |CMP ESI,EBX

00408537  |.^ 7C B3               \JL SHORT FocusMag.004084EC

00408539  |>  85ED                TEST EBP,EBP                               ;  EBP <> 0 ？正确值应为：EBP = 1

0040853B  |.  75 27               JNZ SHORT FocusMag.00408564                ;  一定要跳

0040853D  |>  8BAC24 28010000     MOV EBP,DWORD PTR SS:[ESP+128]

00408544  |>  6A 30               PUSH 30                                    ; /Style = MB\_OK|MB\_ICONEXCLAMATION|MB\_APPLMODAL

00408546  |.  68 CCEE4200         PUSH FocusMag.0042EECC                     ; |Title = "Focus Magic"

0040854B  |.  68 68F54200         PUSH FocusMag.0042F568                     ; |Text = "Invalid Registration Code !

                                                                             ; |        If you have lost your registration code, or it

                                                                             ; |        does not work, then you can get your registration

                                                                             ; |        details emailed to you by going to :-

                                                                             ; |        <http://www.focusmagic.com/newcode>"

00408550  |.  55                  PUSH EBP                                   ; |hOwner

00408551  |.  FF15 94B14200       CALL DWORD PTR DS:[<&USER32.MessageBoxA>]  ; \MessageBoxA

00408557  |.  5F                  POP EDI

00408558  |.  5E                  POP ESI

00408559  |.  5D                  POP EBP

0040855A  |.  32C0                XOR AL,AL

0040855C  |.  5B                  POP EBX

0040855D  |.  81C4 14010000       ADD ESP,114

00408563  |.  C3                  RETN

00408564  |>  83FD 01             CMP EBP,1                                  ;  EBP <> 1 ？

00408567  |.  0F85 F6000000       JNZ FocusMag.00408663                      ;  这儿不能跳 EBP = 1 若EBP<>1、3，则显示"专业版"？并非成功注册。

0040856D  |.  8D5424 14           LEA EDX,DWORD PTR SS:[ESP+14]              ;  只要前面能跳至这儿就可以顺利完成自动注册了。

00408571  |.  52                  PUSH EDX                                   ; /pHandle

00408572  |.  68 8CE04200         PUSH FocusMag.0042E08C                     ; |Subkey = "SOFTWARE\Acclaim Software Ltd\Focus Magic"

00408577  |.  68 02000080         PUSH 80000002                              ; |hKey = HKEY\_LOCAL\_MACHINE

0040857C  |.  FF15 04B04200       CALL DWORD PTR DS:[<&ADVAPI32.RegOpenKeyA>>; \RegOpenKeyA

00408582  |.  85C0                TEST EAX,EAX

00408584  |.  74 27               JE SHORT FocusMag.004085AD                 ;  这儿要跳

00408586  |.  8B8424 28010000     MOV EAX,DWORD PTR SS:[ESP+128]

0040858D  |.  6A 30               PUSH 30                                    ; /Style = MB\_OK|MB\_ICONEXCLAMATION|MB\_APPLMODAL

0040858F  |.  68 4CF54200         PUSH FocusMag.0042F54C                     ; |Title = "Registry Entries Missing"

00408594  |.  68 0CF54200         PUSH FocusMag.0042F50C                     ; |Text = "Focus Magic is not installed properly.  Please re-install it"

00408599  |.  50                  PUSH EAX                                   ; |hOwner
```
