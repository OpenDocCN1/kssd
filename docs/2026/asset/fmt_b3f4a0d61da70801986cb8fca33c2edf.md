
```asm
00406D22  |.  75 7C         JNZ SHORT fixed.00406DA0                          ; 是否选中了 MS\_DOS header 优化

00406DD2  |.  50            PUSH EAX

00406DD3  |.  6A 2E         PUSH 2E                                           ; /ButtonID = 2E (46.)

00406DD5  |.  FF35 38D04000 PUSH DWORD PTR DS:[40D038]                        ; |hWnd = 00020264 (class='#32770',parent=00070234)

00406DDB  |.  E8 A8330000   CALL <JMP.&user32.IsDlgButtonChecked>             ; \IsDlgButtonChecked

00406DE0  |.  48            DEC EAX

00406DE1  |.  58            POP EAX

00406DE2  |.  75 0C         JNZ SHORT fixed.00406DF0                          ; 是否选中了 Code redirection, 是的话, 头部需要增加大小

00406E93  |.  8B43 78       MOV EAX,DWORD PTR DS:[EBX+78]                     ; 导出表 RVA

00406E96  |.  0BC0          OR EAX,EAX

00406E98  |.  74 0B         JE SHORT fixed.00406EA5

00406E9A  |.  50            PUSH EAX

00406E9B  |.  68 3ED54000   PUSH fixed.0040D53E                               ;  ASCII "Export  : %.8lXh"

00406EA0  |.  E8 1FB3FFFF   CALL fixed.004021C4

00406EA5  |>  8B83 80000000 MOV EAX,DWORD PTR DS:[EBX+80]                     ; 导入表 RVA

00406EAB  |.  50            PUSH EAX

00406EAC  |.  68 2DD54000   PUSH fixed.0040D52D                               ;  ASCII "Import  : %.8lXh"

00406EB1  |.  E8 0EB3FFFF   CALL fixed.004021C4

00406EB6  |.  8B83 88000000 MOV EAX,DWORD PTR DS:[EBX+88]                     ; 资源 RVA

00406EBC  |.  0BC0          OR EAX,EAX

00406EBE  |.  74 0B         JE SHORT fixed.00406ECB

00406FFD  |.  6A 0D         PUSH 0D                                           ; /ButtonID = D (13.)

00406FFF  |.  FF35 38D04000 PUSH DWORD PTR DS:[40D038]                        ; |hWnd = 00020264 (class='#32770',parent=00070234)

00407005  |.  E8 7E310000   CALL <JMP.&user32.IsDlgButtonChecked>             ; \IsDlgButtonChecked

0040700A  |.  48            DEC EAX

0040700B  |.  75 0B         JNZ SHORT fixed.00407018                          ; 是否选中了 API redirection

0040716B  |.  6A 28         PUSH 28                                           ; /ButtonID = 28 (40.)

0040716D  |.  FF35 38D04000 PUSH DWORD PTR DS:[40D038]                        ; |hWnd = 00020264 (class='#32770',parent=00070234)

00407173  |.  E8 10300000   CALL <JMP.&user32.IsDlgButtonChecked>             ; \IsDlgButtonChecked

00407178  |.  48            DEC EAX

00407179  |.  74 73         JE SHORT fixed.004071EE                           ; 是否选中了 Strip overlay

00407250  |> \6A 29         PUSH 29                                           ; /ButtonID = 29 (41.)

00407252  |.  FF35 38D04000 PUSH DWORD PTR DS:[40D038]                        ; |hWnd = 00020264 (class='#32770',parent=00070234)

00407258  |.  E8 2B2F0000   CALL <JMP.&user32.IsDlgButtonChecked>             ; \IsDlgButtonChecked

0040725D  |.  48            DEC EAX

0040725E  |.  0F85 C9000000 JNZ fixed.0040732D                                ; 是否选中了 Strip .reloc section

00407341  |.  6A 0A         PUSH 0A                                           ; /ButtonID = A (10.)

00407343  |.  FF35 38D04000 PUSH DWORD PTR DS:[40D038]                        ; |hWnd = 00020264 (class='#32770',parent=00070234)

00407349  |.  E8 3A2E0000   CALL <JMP.&user32.IsDlgButtonChecked>             ; \IsDlgButtonChecked

0040734E  |.  48            DEC EAX

0040734F  |.  75 6A         JNZ SHORT fixed.004073BB                          ; 是否选中了 Add Debug Dection

004073BB

// 暗桩4, 去掉这个暗桩后, 加壳的程序不提示, 直接退出了, 看来还有问题, 继续

004073D2  |.  BF 2B8F4002   MOV EDI,2408F2B

004073D7  |.  81EF 91EDFF01 SUB EDI,1FFED91                                   ; 40A19A

004073DD  |.  B0 3D         MOV AL,3D

004073DF  |.  34 D4         XOR AL,0D4                                        ; E9

004073E1  |.  3807          CMP BYTE PTR DS:[EDI],AL                          ; 40A19A 必须是 E9

004073E3  |.  74 07         JE SHORT fixed.004073EC                           ; 改成 JMP

004073E5  |.  8305 30804000>ADD DWORD PTR DS:[408030],20                      ; 影响下面 0040802F 校验和的计算

004073EC  |>  C705 6AE54000>MOV DWORD PTR DS:[40E56A],0

00407622  |.  6A 27         PUSH 27                                           ; /ButtonID = 27 (39.)

00407624  |.  FF35 38D04000 PUSH DWORD PTR DS:[40D038]                        ; |hWnd = 00020264 (class='#32770',parent=00070234)

0040762A  |.  E8 592B0000   CALL <JMP.&user32.IsDlgButtonChecked>             ; \IsDlgButtonChecked

0040762F  |.  48            DEC EAX

00407630  |.  0F85 DF000000 JNZ fixed.00407715                                ; 是否选中了 Remove OEP

00407793  |.  51            PUSH ECX

00407794  |.  6A 2E         PUSH 2E                                           ; /ButtonID = 2E (46.)

00407796  |.  FF35 38D04000 PUSH DWORD PTR DS:[40D038]                        ; |hWnd = 00020264 (class='#32770',parent=00070234)

0040779C  |.  E8 E7290000   CALL <JMP.&user32.IsDlgButtonChecked>             ; \IsDlgButtonChecked

004077A1  |.  59            POP ECX

004077A2  |.  48            DEC EAX

004077A3  |.  0F85 90000000 JNZ fixed.00407839                                ; 是否选中了 Code Redirection

00407B22  |> \6A 0F         PUSH 0F                                           ; /ButtonID = F (15.)

00407B24  |.  FF35 38D04000 PUSH DWORD PTR DS:[40D038]                        ; |hWnd = 00020264 (class='#32770',parent=00070234)

00407B2A  |.  E8 59260000   CALL <JMP.&user32.IsDlgButtonChecked>             ; \IsDlgButtonChecked

00407B2F  |.  48            DEC EAX

00407B30  |.  75 0D         JNZ SHORT fixed.00407B3F                          ; 是否选中了 AntiDump

00407C2C  |.  6A 2A         PUSH 2A                                           ; /ButtonID = 2A (42.)

00407C2E  |.  FF35 38D04000 PUSH DWORD PTR DS:[40D038]                        ; |hWnd = 00020264 (class='#32770',parent=00070234)

00407C34  |.  E8 4F250000   CALL <JMP.&user32.IsDlgButtonChecked>             ; \IsDlgButtonChecked

00407C39  |.  A3 F5E04000   MOV DWORD PTR DS:[40E0F5],EAX                     ; 是否选中了 Compress resource

00407C88  |.  51            PUSH ECX

00407C89  |.  53            PUSH EBX

00407C8A  |.  50            PUSH EAX

00407C8B  |.  E8 F0A0FFFF   CALL fixed.00401D80                               ; 返回最大一块压缩区段大小, 这里有问题, F7 进去看看

00407C90  |.  8BD0          MOV EDX,EAX

00407D63  |> /833D F5E04000>/CMP DWORD PTR DS:[40E0F5],0                      ; 压缩区段

00407D6A  |. |0F84 B4000000 |JE fixed.00407E24

00407E5E  |.  E8 3D3A0000   |CALL fixed.0040B8A0                              ; 更新进度条

00407EEC  |.  8305 49E14000>|ADD DWORD PTR DS:[40E149],28

00407EF3  |.  8305 45E14000>|ADD DWORD PTR DS:[40E145],28

00407EFA  |.  FF05 41E14000 |INC DWORD PTR DS:[40E141]

00407F00  |.  59            |POP ECX

00407F01  |.  49            |DEC ECX

00407F02  |.^ 0F85 5BFEFFFF \JNZ fixed.00407D63

00407F7E   .  FF73 08       PUSH DWORD PTR DS:[EBX+8]                         ; /hObject

00407F81   .  E8 06210000   CALL <JMP.&kernel32.CloseHandle>                  ; \CloseHandle

0040802F   .  B9 20000000   MOV ECX,20                                        ; 计算 20h 的校验和(401000-401020),

00408034   .  8B43 14       MOV EAX,DWORD PTR DS:[EBX+14]                     ; 400 Raw Offset

00408037   .  03F8          ADD EDI,EAX

00408039   .  E8 C6D7FFFF   CALL fixed.00405804

0040833B   > \6A 11         PUSH 11                                           ; /ButtonID = 11 (17.)

0040833D   .  FF35 38D04000 PUSH DWORD PTR DS:[40D038]                        ; |hWnd = 00100174 (class='#32770',parent=0007025C)

00408343   .  E8 401E0000   CALL <JMP.&user32.IsDlgButtonChecked>             ; \IsDlgButtonChecked

00408348   .  48            DEC EAX

00408349   .  75 24         JNZ SHORT fixed.0040836F                          ; 是否选中了 Close Program After x minute

00408376   > \6A 08         PUSH 8                                            ; /ButtonID = 8

00408378   .  FF35 38D04000 PUSH DWORD PTR DS:[40D038]                        ; |hWnd = 00100174 (class='#32770',parent=0007025C)
```
