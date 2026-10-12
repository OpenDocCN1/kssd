
```text
00433C1A   .  50           PUSH EAX                        ; |hWnd
00433C1B   .  FFD3        CALL EBX                         ; \GetWindowTextA
00433C1D   .  6A 64       PUSH 64                           ; /Count = 64 (100.)
00433C1F   .  8D8424 C800000>LEA EAX,DWORD PTR SS:[ESP+C8]    ; |
00433C26   .  50          PUSH EAX                          ; |Buffer
00433C27   .  68 50040000  PUSH 450                         ; |/ControlID = 450 (1104.)
00433C2C   .  57          PUSH EDI                          ; ||hWnd
00433C2D   .  FFD6        CALL ESI                          ; |\GetDlgItem
00433C2F   .  50          PUSH EAX                          ; |hWnd
00433C30   .  FFD3        CALL EBX                          ; \GetWindowTextA
00433C32   .  8D8C24 C400000>LEA ECX,DWORD PTR SS:[ESP+C4]    ;  //注册码
00433C39   .  51          PUSH ECX
00433C3A   .  8D5424 64   LEA EDX,DWORD PTR SS:[ESP+64]    ;  //用户名
00433C3E   .  52          PUSH EDX
00433C3F   .  E8 FCFBFFFF  CALL movgear.00433840            ;  //关键 CALL
00433C44   .  83C4 08     ADD ESP,8
00433C47   .  85C0        TEST EAX,EAX
00433C49   .  0F84 B6000000  JE movgear.00433D05           ;  //关键跳转
00433C4F   .  8D4424 10   LEA EAX,DWORD PTR SS:[ESP+10]
00433C53   .  50          PUSH EAX                          ; /pDisposition
00433C54   .  8D4C24 10   LEA ECX,DWORD PTR SS:[ESP+10]    ; |
00433C58   .  51          PUSH ECX                          ; |pHandle
00433C59   .  6A 00       PUSH 0                            ; |pSecurity = NULL
00433C5B   .  68 3F000F00  PUSH 0F003F                     ; |Access = KEY\_ALL\_ACCESS
00433C60   .  6A 00       PUSH 0                            ; |Options = REG\_OPTION\_NON\_VOLATILE
00433C62   .  68 85F64700  PUSH movgear.0047F685            ; |Class = ""
00433C67   .  6A 00       PUSH 0                            ; |Reserved = 0
00433C69   .  68 84E44800  PUSH movgear.0048E484            ; |software\\gamani\\gifmoviegear\\2.0
00433C6E   .  68 02000080  PUSH 80000002                    ; |hKey = HKEY\_LOCAL\_MACHINE
00433C73   .  FF15 0CF04700  CALL DWORD PTR DS:[<&ADVAPI32.RegCreateK>;  \RegCreateKeyExA
00433C79   .  8D4424 60   LEA EAX,DWORD PTR SS:[ESP+60]
00433C7D   .  8D50 01     LEA EDX,DWORD PTR DS:[EAX+1]
00433C80   >  8A08        MOV CL,BYTE PTR DS:[EAX]
00433C82   .  40          INC EAX
00433C83   .  84C9        TEST CL,CL
00433C85   .^ 75 F9       JNZ SHORT movgear.00433C80
00433C87   .  8B35 00F04700  MOV ESI,DWORD PTR DS:[<&ADVAPI32.RegSetV>;  ADVAPI32.RegSetValueExA
00433C8D   .  2BC2        SUB EAX,EDX
00433C8F   .  40          INC EAX
00433C90   .  50          PUSH EAX                          ; /BufSize
00433C91   .  8B4424 10   MOV EAX,DWORD PTR SS:[ESP+10]    ; |
00433C95   .  8D5424 64   LEA EDX,DWORD PTR SS:[ESP+64]    ; |
00433C99   .  52          PUSH EDX                          ; |Buffer
00433C9A   .  6A 01       PUSH 1                            ; |ValueType = REG\_SZ
00433C9C   .  6A 00       PUSH 0                            ; |Reserved = 0
00433C9E   .  68 C8F34800  PUSH movgear.0048F3C8            ; |regname3
00433CA3   .  50          PUSH EAX                          ; |hKey
00433CA4   .  FFD6        CALL ESI                          ; \RegSetValueExA
00433CA6   .  8D8424 C400000>LEA EAX,DWORD PTR SS:[ESP+C4]
00433CAD   .  8D48 01     LEA ECX,DWORD PTR DS:[EAX+1]
00433CB0   >  8A10        MOV DL,BYTE PTR DS:[EAX]
00433CB2   .  40          INC EAX
00433CB3   .  84D2        TEST DL,DL
00433CB5   .^ 75 F9       JNZ SHORT movgear.00433CB0
00433CB7   .  8B5424 0C   MOV EDX,DWORD PTR SS:[ESP+C]
00433CBB   .  2BC1        SUB EAX,ECX
00433CBD   .  40          INC EAX
00433CBE   .  50          PUSH EAX
00433CBF   .  8D8C24 C800000>LEA ECX,DWORD PTR SS:[ESP+C8]
00433CC6   .  51          PUSH ECX
00433CC7   .  6A 01       PUSH 1
00433CC9   .  6A 00       PUSH 0
00433CCB   .  68 D4F34800  PUSH movgear.0048F3D4            ;  regcode3
00433CD0   .  52          PUSH EDX
00433CD1   .  FFD6        CALL ESI
00433CD3   .  8B4424 0C   MOV EAX,DWORD PTR SS:[ESP+C]
00433CD7   .  50          PUSH EAX                          ; /hKey
00433CD8   .  FF15 18F04700  CALL DWORD PTR DS:[<&ADVAPI32.RegCloseKe>;  \RegCloseKey
00433CDE   .  68 E0F34800  PUSH movgear.0048F3E0            ; /software\\loani\\mg4
00433CE3   .  68 02000080  PUSH 80000002                    ; |hKey = HKEY\_LOCAL\_MACHINE
00433CE8   .  FF15 14F04700  CALL DWORD PTR DS:[<&ADVAPI32.RegDeleteK>;  \RegDeleteKeyA
00433CEE   .  6A 01       PUSH 1                            ; /Result = 1
00433CF0   .  57          PUSH EDI                          ; |hWnd
00433CF1   .  FF15 A4F34700  CALL DWORD PTR DS:[<&USER32.EndDialog>\]  ; \EndDialog
00433CF7   .  5F          POP EDI
00433CF8   .  5E          POP ESI
00433CF9   .  33C0        XOR EAX,EAX
00433CFB   .  5B          POP EBX
00433CFC   .  81C4 1C010000  ADD ESP,11C
00433D02   .  C2 1000     RETN 10
00433D05   >  6A 30        PUSH 30
00433D07   .  68 159D0000  PUSH 9D15
00433D0C   .  68 149D0000  PUSH 9D14
00433D11   .  57          PUSH EDI
00433D12   .  E8 C9D9FDFF  CALL movgear.004116E0            ;  //错误提示
00433D17   .  83C4 10     ADD ESP,10
00433D1A   .  68 4F040000  PUSH 44F
00433D1F   .  57          PUSH EDI
00433D20   .  FFD6        CALL ESI
00433D22   .  50          PUSH EAX                          ; /hWnd
00433D23   .  FF15 A4F44700  CALL DWORD PTR DS:[<&USER32.SetFocus>\]   ; \SetFocus
00433D29   .  5F          POP EDI
00433D2A   .  5E          POP ESI
00433D2B   .  33C0        XOR EAX,EAX
00433D2D   .  5B          POP EBX
00433D2E   .  81C4 1C010000  ADD ESP,11C
00433D34   .  C2 1000     RETN 10
00433D37   >  2D 35010000  SUB EAX,135
00433D3C   .  74 43        JE SHORT movgear.00433D81
00433D3E   .  83E8 03     SUB EAX,3
00433D41   .  74 0E        JE SHORT movgear.00433D51
00433D43   >  5F          POP EDI                            ;  Default case of switch 00433BD0
00433D44   .  5E          POP ESI
00433D45   .  33C0        XOR EAX,EAX
00433D47   .  5B          POP EBX
00433D48   .  81C4 1C010000  ADD ESP,11C
00433D4E   .  C2 1000     RETN 10
00433D51   >  8BB424 340100>MOV ESI,DWORD PTR SS:[ESP+134]      ;  Case 138 (WM\_CTLCOLORSTATIC) of switch 00433A4A
00433D58   .  68 FFFFFF00  PUSH 0FFFFFF                       ; /Color = <WHITE>
00433D5D   .  56          PUSH ESI                          ; |hDC
00433D5E   .  FF15 CCF04700  CALL DWORD PTR DS:[<&GDI32.SetBkColor>\]  ; \SetBkColor
00433D64   .  6A 00       PUSH 0                            ; /Color = <BLACK>
00433D66   .  56          PUSH ESI                          ; |hDC
00433D67   .  FF15 C8F04700  CALL DWORD PTR DS:[<&GDI32.SetTextColor>];  \SetTextColor
00433D6D   .  6A 00       PUSH 0                            ; /ObjType = WHITE\_BRUSH
00433D6F   .  FF15 D0F04700  CALL DWORD PTR DS:[<&GDI32.GetStockObjec>;  \GetStockObject
00433D75   .  5F          POP EDI
00433D76   .  5E          POP ESI
00433D77   .  5B          POP EBX
00433D78   .  81C4 1C010000  ADD ESP,11C
00433D7E   .  C2 1000     RETN 10
00433D81   >  6A 05        PUSH 5                            ; /ObjType = NULL\_BRUSH; Case 135 (WM\_CTLCOLORBTN) of

switch 00433A4A
00433D83   .  FF15 D0F04700  CALL DWORD PTR DS:[<&GDI32.GetStockObjec>;  \GetStockObject
00433D89   .  5F          POP EDI
00433D8A   .  5E          POP ESI
00433D8B   .  5B          POP EBX
00433D8C   .  81C4 1C010000  ADD ESP,11C
00433D92   .  C2 1000     RETN 10
\==============================================================
00433840  /$  53          PUSH EBX
00433841  |.  55          PUSH EBP
00433842  |.  8B6C24 10   MOV EBP,DWORD PTR SS:[ESP+10]
00433846  |.  807D 00 6D  CMP BYTE PTR SS:[EBP],6D             ;  //注册码第 1 个字母与 6D 即"m"比较
0043384A  |.  56          PUSH ESI
0043384B  |.  57          PUSH EDI
0043384C  |.  0F85 AD000000  JNZ movgear.004338FF              ;  //不等则跳
00433852  |.  807D 01 67  CMP BYTE PTR SS:[EBP+1],67           ;  //注册码第 2 个字母与 67 即"g"比较
00433856  |.  0F85 A3000000  JNZ movgear.004338FF              ;  //不等则跳
0043385C  |.  807D 02 33  CMP BYTE PTR SS:[EBP+2],33           ;  //注册码第 3 个字母与 33 即"3"比较
00433860  |.  0F85 99000000  JNZ movgear.004338FF              ;  //不等则跳
00433866  |.  807D 03 37  CMP BYTE PTR SS:[EBP+3],37           ;  //注册码第 4 个字母与 37 即"7"比较
0043386A  |.  0F85 8F000000  JNZ movgear.004338FF              ;  //不等则跳
00433870  |.  33DB        XOR EBX,EBX                          ;  //EBX=0
00433872  |>  8BBB F8F34800 /MOV EDI,DWORD PTR DS:[EBX+48F3F8]    ;  //"mvg21951736"
00433878  |.  8BC7         |MOV EAX,EDI
0043387A  |.  8D50 01      |LEA EDX,DWORD PTR DS:[EAX+1]
0043387D  |.  8D49 00      |LEA ECX,DWORD PTR DS:[ECX]
00433880  |>  8A08         |/MOV CL,BYTE PTR DS:[EAX]
00433882  |.  40          ||INC EAX
00433883  |.  84C9         ||TEST CL,CL
00433885  |.^ 75 F9        |\JNZ SHORT movgear.00433880
00433887  |.  2BC2         |SUB EAX,EDX                         ;  //EAX=EAX-EDX
00433889  |.  8BC8         |MOV ECX,EAX
0043388B  |.  8BF5         |MOV ESI,EBP
0043388D  |.  33C0         |XOR EAX,EAX                         ;  //EAX=0
0043388F  |.  F3:A6        |REPE CMPS BYTE PTR ES:[EDI],BYTE PTR DS>
00433891  |.  74 65        |JE SHORT movgear.004338F8
00433893  |.  83C3 04      |ADD EBX,4
00433896  |.  81FB 80000000 |CMP EBX,80
0043389C  |.^ 72 D4        \JB SHORT movgear.00433872           ;  //注册码是否在黑名单
0043389E  |.  807D 04 73  CMP BYTE PTR SS:[EBP+4],73           ;  //注册码第 5 个字母与 73 即"s"比较
004338A2  |.  75 01        JNZ SHORT movgear.004338A5          ;  //不等则跳
004338A4  |.  45          INC EBP
004338A5  |>  8D4D 07      LEA ECX,DWORD PTR SS:[EBP+7]
004338A8  |.  51          PUSH ECX
004338A9  |.  E8 26BD0300  CALL movgear.0046F5D4              ;  //将注册码第 8 位以后的数字转 16 进制送入 EAX(如果第 5 个
004338AE  |.  8B5C24 18   MOV EBX,DWORD PTR SS:[ESP+18]       ;  //用户名
004338B2  |.  8A13        MOV DL,BYTE PTR DS:[EBX]             ;  //用户名第 1 个字母 ASC 值
004338B4  |.  83C4 04     ADD ESP,4
004338B7  |.  33C9        XOR ECX,ECX                          ;  //ECX=0
004338B9  |.  84D2        TEST DL,DL
004338BB  |.  8BFB        MOV EDI,EBX                          ;  (初始 cpu 选择)
004338BD  |.  BE DF0B0000  MOV ESI,0BDF                       ;  //ESI=0BDF
004338C2  |.  74 26        JE SHORT movgear.004338EA
004338C4  |>  0FBED2       /MOVSX EDX,DL
004338C7  |.  41          |INC ECX                              ;  //ECX=ECX+1
004338C8  |.  0FAFD1      |IMUL EDX,ECX                        ;  //EDX=EDX\*ECX
004338CB  |.  03F2        |ADD ESI,EDX                         ;  //ESI=ESI+EDX
004338CD  |.  81FE BE170000 |CMP ESI,17BE                      ;  //ESI 与 17BE 比较
004338D3  |.  7E 06       |JLE SHORT movgear.004338DB          ;  //小于等于则跳
004338D5  |.  81EE BE170000 |SUB ESI,17BE                      ;  //ESI=ESI-17BE
004338DB  |>  83F9 0A      |CMP ECX,0A                          ;  //ECX 与 0A 比较
004338DE  |.  7E 02       |JLE SHORT movgear.004338E2          ;  //小于等于则跳
004338E0  |.  33C9        |XOR ECX,ECX                         ;  //ECX=0
004338E2  |>  8A57 01      |MOV DL,BYTE PTR DS:[EDI+1]          ;  //依次取用户名 ASC 值
004338E5  |.  47          |INC EDI                             ;  //EDI=EDI+1
004338E6  |.  84D2        |TEST DL,DL
004338E8  |.^ 75 DA       \JNZ SHORT movgear.004338C4          ;  //循环
004338EA  |>  3BF0        CMP ESI,EAX                          ;  //比较 ESI 与 EAX
004338EC  |.  75 15        JNZ SHORT movgear.00433903          ;  //不等则跳,爆破点
004338EE  |.  5F          POP EDI
004338EF  |.  5E          POP ESI
004338F0  |.  5D          POP EBP
004338F1  |.  B8 01000000  MOV EAX,1                          ;  //标志位赋值
004338F6  |.  5B          POP EBX
004338F7  |.  C3          RETN
004338F8  |>  5F          POP EDI
004338F9  |.  5E          POP ESI
004338FA  |.  5D          POP EBP
004338FB  |.  33C0        XOR EAX,EAX
004338FD  |.  5B          POP EBX
004338FE  |.  C3          RETN
004338FF  |>  8B5C24 14   MOV EBX,DWORD PTR SS:[ESP+14]
00433903  |>  55          PUSH EBP
00433904  |.  53          PUSH EBX
00433905  |.  E8 16FCFFFF  CALL movgear.00433520
0043390A  |.  83C4 08     ADD ESP,8
0043390D  |.  5F          POP EDI
0043390E  |.  5E          POP ESI
0043390F  |.  5D          POP EBP
00433910  |.  5B          POP EBX
00433911  \\.  C3         RETN

```
