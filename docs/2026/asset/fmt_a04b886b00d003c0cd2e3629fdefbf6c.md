
```
0012F704  4F 17 E4 07 9F 11 C8 1F 87 AC 24 DC 18 BD EC C3  O.........$.....

0012F714  B9 BB 4C 9B ED 88 AC D3 A2 9F 48 D4 3D 8E 80 CB  ..L.......H.=...

0012F724  BE 61 3B FB A6 DC D7 38 1F 67 9B A3 F2 EF 37 70  .a;....8.g....7p

0012F734  50 70 7F A4 6D FE FF 6F 45 77 93 C7 E3 AB 44 FF  Pp..m..oEw....D.

0012F744  FC CC DF 5C 0E 23 E8 2C 5E 53 97 88 33 AD 68 E7  ...\.#.,^S..3.h.

0012F754  50 32 07 04 B3 99 43 FB 4F 55 9C A7 41 76 74 8B  P2....C.OU..Avt.

0012F764  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................

0012F774  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
```

## 验证注册码的正伪

```
0040220C   $ /E9 4FD20500   JMP XQSJ.0045F460

0045F460   > \55            PUSH EBP

0045F461   .  8BEC          MOV EBP,ESP

0045F463   .  83E4 F8       AND ESP,FFFFFFF8

0045F466   .  6A FF         PUSH -1

0045F468   .  68 FB804600   PUSH XQSJ.004680FB                                    ;  SE handler installation

0045F46D   .  64:A1 0000000>MOV EAX,DWORD PTR FS:[0]

0045F473   .  50            PUSH EAX

0045F474   .  64:8925 00000>MOV DWORD PTR FS:[0],ESP

0045F47B   .  81EC A0010000 SUB ESP,1A0

0045F481   .  53            PUSH EBX

0045F482   .  56            PUSH ESI

0045F483   .  8B75 08       MOV ESI,DWORD PTR SS:[EBP+8]

0045F486   .  8BD9          MOV EBX,ECX

0045F488   .  57            PUSH EDI

0045F489   .  8BCE          MOV ECX,ESI

0045F48B   .  FF15 58EC4402 CALL DWORD PTR DS:[<&MFC71.#2902>]                    ; strlen(sn)

0045F491   .  83F8 1E       CMP EAX,1E

0045F494   .  0F8C EF010000 JL XQSJ.0045F689                                      ; len<1E 则 over

0045F49A   .  6A 05         PUSH 5                                                ; // 5

0045F49C   .  8D4424 10     LEA EAX,DWORD PTR SS:[ESP+10]

0045F4A0   .  50            PUSH EAX

0045F4A1   .  8BCE          MOV ECX,ESI

0045F4A3   .  FF15 88EB4402 CALL DWORD PTR DS:[<&MFC71.#3997>]                    ; // 取 sn 前 5 个字符到新字符串 name

0045F4A9   .  894424 10     MOV DWORD PTR SS:[ESP+10],EAX                         ; EAX -> name

0045F4AD   .  BF FCE34700   MOV EDI,XQSJ.0047E3FC                                 ; 47E3FC 开始放黑名单, 目前只有两个(TEST0, TEST1) ^\_^

0045F4B2   >  8B0F          MOV ECX,DWORD PTR DS:[EDI]

0045F4B4   .  51            PUSH ECX                                              ; ECX -> black\_name

0045F4B5   .  8B4C24 14     MOV ECX,DWORD PTR SS:[ESP+14]

0045F4B9   .  FF15 48EC4402 CALL DWORD PTR DS:[<&MFC71.#1482>]                    ; \_mbscmp(name, black\_name);

0045F4BF   .  85C0          TEST EAX,EAX

0045F4C1   .  0F84 B8010000 JE XQSJ.0045F67F                                      ; 相等则跳出循环, 设置 EDI=1 后返回 0045F4D4

0045F4C7   .  83C7 04       ADD EDI,4

0045F4CA   .  81FF 04E44700 CMP EDI,XQSJ.0047E404                                 ; 黑名单比较完了吗?

0045F4D0   .^ 7C E0         JL SHORT XQSJ.0045F4B2

0045F4D2   .  33FF          XOR EDI,EDI                                           ; 不在黑名单上, 则 EDI=0

0045F4D4   >  8D4C24 0C     LEA ECX,DWORD PTR SS:[ESP+C]

0045F4D8   .  FF15 68EC4402 CALL DWORD PTR DS:[<&MFC71.#578>]

0045F4DE   .  33C0          XOR EAX,EAX                                           ; EAX = 0 注册码失败标志                                          ;

0045F4E0   .  85FF          TEST EDI,EDI

0045F4E2   .  0F85 A3010000 JNZ XQSJ.0045F68B                                     ; 是黑名则 over

0045F4E8   .  B9 40000000   MOV ECX,40

0045F4ED   .  8D7C24 18     LEA EDI,DWORD PTR SS:[ESP+18]

0045F4F1   .  F3:AB         REP STOS DWORD PTR ES:[EDI]

0045F4F3   .  8BCE          MOV ECX,ESI

0045F4F5   .  FF15 58EC4402 CALL DWORD PTR DS:[<&MFC71.#2902>]

0045F4FB   .  50            PUSH EAX

0045F4FC   .  8BCE          MOV ECX,ESI

0045F4FE   .  FF15 44EC4402 CALL DWORD PTR DS:[<&MFC71.#2468>]

0045F504   .  8D5424 18     LEA EDX,DWORD PTR SS:[ESP+18]                         ; 12F600

0045F508   >  8A08          MOV CL,BYTE PTR DS:[EAX]

0045F50A   .  40            INC EAX

0045F50B   .  880A          MOV BYTE PTR DS:[EDX],CL

0045F50D   .  42            INC EDX

0045F50E   .  84C9          TEST CL,CL

0045F510   .^ 75 F6         JNZ SHORT XQSJ.0045F508                               ; 12F600 -> sn

0045F512   .  8B5424 18     MOV EDX,DWORD PTR SS:[ESP+18]                         ; 取前 5 个字符( 就是 name)

0045F516   .  8A4C24 1C     MOV CL,BYTE PTR SS:[ESP+1C]

0045F51A   .  8D43 04       LEA EAX,DWORD PTR DS:[EBX+4]                          ; 12F8B0

0045F51D   .  8910          MOV DWORD PTR DS:[EAX],EDX

0045F51F   .  8848 04       MOV BYTE PTR DS:[EAX+4],CL                            ; 12F8B0 -> name

0045F522   .  8BCE          MOV ECX,ESI

0045F524   .  8DBB 04010000 LEA EDI,DWORD PTR DS:[EBX+104]                        ; 12F9B0 buffer

0045F52A   .  FF15 58EC4402 CALL DWORD PTR DS:[<&MFC71.#2902>]                    ; strlen(sn)

0045F530   .  8BC8          MOV ECX,EAX

0045F532   .  83E9 05       SUB ECX,5                                             ; 去掉前 5 个后的长度

0045F535   .  8BD1          MOV EDX,ECX

0045F537   .  C1E9 02       SHR ECX,2

0045F53A   .  8D7424 1D     LEA ESI,DWORD PTR SS:[ESP+1D]

0045F53E   .  F3:A5         REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI]

0045F540   .  8BCA          MOV ECX,EDX

0045F542   .  8D83 04010000 LEA EAX,DWORD PTR DS:[EBX+104]

0045F548   .  83E1 03       AND ECX,3

0045F54B   .  50            PUSH EAX                                              ; 12F9B0 -> sn2 (sn2 就是 sn 去掉前 5 个字符)

0045F54C   .  F3:A4         REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]

0045F54E   .  8DB3 08040000 LEA ESI,DWORD PTR DS:[EBX+408]                        ; esi = 12FCB4

0045F554   .  68 1C194700   PUSH XQSJ.0047191C                                    ;  ASCII "%s"

0045F559   .  56            PUSH ESI

0045F55A   .  FF15 90E74402 CALL DWORD PTR DS:[<&MFC71.#2322>]                    ; sprintf(esi, "%s", sn2)

0045F560   .  83C4 0C       ADD ESP,0C

0045F563   .  6A 10         PUSH 10                                               ; hex

0045F565   .  8DBB A4040000 LEA EDI,DWORD PTR DS:[EBX+4A4]

0045F56B   .  56            PUSH ESI                                              ; esi -> sn2

0045F56C   .  8BCF          MOV ECX,EDI

0045F56E   .  E8 2924FAFF   CALL XQSJ.0040199C                                    ; sn2 转换为大数

0045F573   .  8D83 34050000 LEA EAX,DWORD PTR DS:[EBX+534]                        ; RSA N 大数

0045F579   .  50            PUSH EAX

0045F57A   .  8D8B C4050000 LEA ECX,DWORD PTR DS:[EBX+5C4]                        ; RSA E 大数

0045F580   .  51            PUSH ECX

0045F581   .  8D9424 200100>LEA EDX,DWORD PTR SS:[ESP+120]                        ; 12F700 buffer

0045F588   .  52            PUSH EDX

0045F589   .  8BCF          MOV ECX,EDI                                           ; sn2 大数

0045F58B   .  E8 2F2EFAFF   CALL XQSJ.004023BF                                    ; RSA\_encrypt(N, E, sn2) -> sn3

0045F590   .  8DBB 14040000 LEA EDI,DWORD PTR DS:[EBX+414]                        ; 12FCC0

0045F596   .  50            PUSH EAX                                              ; 12F700 -> sn3

0045F597   .  8BCF          MOV ECX,EDI

0045F599   .  C78424 B80100>MOV DWORD PTR SS:[ESP+1B8],0

0045F5A4   .  E8 3B3BFAFF   CALL XQSJ.004030E4                                    ; sn3 大数复制到 12FCC0

0045F5A9   .  8D8C24 180100>LEA ECX,DWORD PTR SS:[ESP+118]

0045F5B0   .  C78424 B40100>MOV DWORD PTR SS:[ESP+1B4],-1

0045F5BB   .  E8 473BFAFF   CALL XQSJ.00403107

0045F5C0   .  6A 10         PUSH 10                                               ; hex

0045F5C2   .  8DB3 04040000 LEA ESI,DWORD PTR DS:[EBX+404]                        ; 12FCB0 buffer

0045F5C8   .  56            PUSH ESI

0045F5C9   .  8BCF          MOV ECX,EDI                                           ; sn3 大数

0045F5CB   .  E8 162AFAFF   CALL XQSJ.00401FE6                                    ; sn3 大数转换为 str( 应该是由 0-9 组成的 10 进制数)

0045F5D0   .  8BCE          MOV ECX,ESI                                           ; esi -> sn3 str
```
