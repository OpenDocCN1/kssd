
```
00402FC4  |.  8B45 CC       MOV EAX,DWORD PTR SS:[EBP-34]             ;   
00402FC7  |.  50           PUSH EAX                                    ;   
00402FC8  |.  68 459B4B00  PUSH 《真正的.004B9B45                      ;   
00402FCD  |.  51           PUSH ECX                                    ;   
00402FCE  |.  E8 21460900  CALL 《真正的.004975F4                      ;   
00402FD3  |.  83C4 0C      ADD ESP,0C

00402FD6  |.  8D86 E5050000 LEA EAX,DWORD PTR DS:[ESI+5E5]            ;   
00402FDC  |.  56           PUSH ESI

00402FDD  |.  57           PUSH EDI

00402FDE  |.  8BF8         MOV EDI,EAX                                 ;   
00402FE0  |.  33C0         XOR EAX,EAX

00402FE2  |.  83C9 FF      OR ECX,FFFFFFFF                             ;   
00402FE5  |.  8DB5 A8FDFFFF LEA ESI,DWORD PTR SS:[EBP-258]

00402FEB  |.  F2:AE        REPNE SCAS BYTE PTR ES:[EDI]

00402FED  |.  F7D1         NOT ECX                                     ;   
00402FEF  |.  2BF9         SUB EDI,ECX                                 ;   
00402FF1  |.  8BD1         MOV EDX,ECX                                 ;   
00402FF3  |.  87F7         XCHG EDI,ESI                                ;   
00402FF5  |.  C1E9 02      SHR ECX,2                                   ;   
00402FF8  |.  8BC7         MOV EAX,EDI                                 ;   
00402FFA  |.  F3:A5        REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS>;   
00402FFC  |.  8BCA         MOV ECX,EDX                                 ;   
00402FFE  |.  8D85 A8FDFFFF LEA EAX,DWORD PTR SS:[EBP-258]            ;   
00403004  |.  83E1 03      AND ECX,3                                   ;   
00403007  |.  F3:A4        REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[>

00403009  |.  5F           POP EDI

0040300A  |.  5E           POP ESI

0040300B  |.  50           PUSH EAX                                    ;   
0040300C  |.  56           PUSH ESI                                    ;   
0040300D  |.  E8 06B90100  CALL 《真正的.0041E918                      ;   
00403012  |.  83C4 08      ADD ESP,8                                   ;   
00403015  |.  8D95 A8FEFFFF LEA EDX,DWORD PTR SS:[EBP-158]            ;   
0040301B  |.  52           PUSH EDX                                    ;   
0040301C  |.  8D8D A8FDFFFF LEA ECX,DWORD PTR SS:[EBP-258]            ;   
00403022  |.  51           PUSH ECX                                    ;   
00403023  |.  E8 000F0900  CALL 《真正的.00493F28                      ;  这个 CALL 将机器码前 20 位同[EBP-34]中的十进制值连接起来存储在 EAX 中，数值为(56780123453362080553105342)
00403028  |.  83C4 08      ADD ESP,8                                   ;   
0040302B  |.  33C9         XOR ECX,ECX                                 ;   
0040302D  |.  8D9D A8FDFFFF LEA EBX,DWORD PTR SS:[EBP-258]            ;  [EBP-258]里存储的值为(56780123453362080553105342)

00403033  |>  8B45 CC       /MOV EAX,DWORD PTR SS:[EBP-34]            ;   
00403036  |.  51           |PUSH ECX

00403037  |.  99           |CDQ

00403038  |.  B9 64000000  |MOV ECX,64

0040303D  |.  F7F9         |IDIV ECX                                   ;   
0040303F  |.  59           |POP ECX                                     ;   
00403040  |.  8D51 64      |LEA EDX,DWORD PTR DS:[ECX+64]            ;   
00403043  |.  41           |INC ECX                                     ;   
00403044  |.  F7EA         |IMUL EDX                                     ;   
00403046  |.  0FBE13       |MOVSX EDX,BYTE PTR DS:[EBX]              ;   
00403049  |.  0345 C4      |ADD EAX,DWORD PTR SS:[EBP-3C]            ; 
0040304C  |.  43           |INC EBX

0040304D  |.  03C2         |ADD EAX,EDX                                 ; 
0040304F  |.  83F9 14      |CMP ECX,14

00403052  |.  8945 C4      |MOV DWORD PTR SS:[EBP-3C],EAX            ;   
00403055  |.^ 7C DC        \JL SHORT 《真正的.00403033

00403057  |.  8D85 94FDFFFF LEA EAX,DWORD PTR SS:[EBP-26C]            ;   
0040305D  |.  50           PUSH EAX                                    ; 
0040305E  |.  8D8E E5030000 LEA ECX,DWORD PTR DS:[ESI+3E5]            ; 
00403064  |.  51           PUSH ECX                                    ; 
00403065  |.  E8 EAFCFFFF  CALL 《真正的.00402D54                      ;  用 MD5 循环加密去掉倒数 8 位的假码
0040306A  |.  83C4 08      ADD ESP,8

0040306D  |.  68 00010000  PUSH 100                                    ; 
00403072  |.  E8 09060900  CALL 《真正的.00493680                      ; 
00403077  |.  59           POP ECX

00403078  |.  8945 C8      MOV DWORD PTR SS:[EBP-38],EAX

0040307B  |.  8B45 C8      MOV EAX,DWORD PTR SS:[EBP-38]              ;   
0040307E  |.  56           PUSH ESI

0040307F  |.  8BF0         MOV ESI,EAX                                 ;   
00403081  |.  57           PUSH EDI

00403082  |.  33C0         XOR EAX,EAX                                 ;   
00403084  |.  8DBD A8FDFFFF LEA EDI,DWORD PTR SS:[EBP-258]            ;   
0040308A  |.  83C9 FF      OR ECX,FFFFFFFF

0040308D  |.  F2:AE        REPNE SCAS BYTE PTR ES:[EDI]

0040308F  |.  F7D1         NOT ECX                                     ;   
00403091  |.  2BF9         SUB EDI,ECX                                 ;   
00403093  |.  8BD1         MOV EDX,ECX                                 ;   
00403095  |.  87F7         XCHG EDI,ESI                                ;   
00403097  |.  C1E9 02      SHR ECX,2

0040309A  |.  8BC7         MOV EAX,EDI

0040309C  |.  F3:A5        REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS>

0040309E  |.  8BCA         MOV ECX,EDX

004030A0  |.  8D95 A8FDFFFF LEA EDX,DWORD PTR SS:[EBP-258]

004030A6  |.  83E1 03      AND ECX,3

004030A9  |.  F3:A4        REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[>

004030AB  |.  5F           POP EDI

004030AC  |.  5E           POP ESI

004030AD  |.  8B45 C8      MOV EAX,DWORD PTR SS:[EBP-38]              ;   
004030B0  |.  8986 10190000 MOV DWORD PTR DS:[ESI+1910],EAX

004030B6  |.  52           PUSH EDX

004030B7  |.  E8 FC0E0900  CALL 《真正的.00493FB8                      ;  计算上面数字长度为 1A
004030BC  |.  59           POP ECX

004030BD  |.  8945 D0      MOV DWORD PTR SS:[EBP-30],EAX              ;  [EBP-30]=1A
004030C0  |.  8D85 A8FDFFFF LEA EAX,DWORD PTR SS:[EBP-258]            ;   
004030C6  |.  33C9         XOR ECX,ECX                                 ;   
004030C8  |.  8945 AC      MOV DWORD PTR SS:[EBP-54],EAX              ;   
004030CB  |.  33DB         XOR EBX,EBX                                 ;   
004030CD  |.  3B4D D0      CMP ECX,DWORD PTR SS:[EBP-30]              ;   
004030D0  |.  7D 17        JGE SHORT 《真正的.004030E9

004030D2  |>  8B55 AC      /MOV EDX,DWORD PTR SS:[EBP-54]            ;  [EBP-54]里存储的值为(56780123453362080553105342)
004030D5  |.  33C0         |XOR EAX,EAX                                 ;  0
004030D7  |.  8A02         |MOV AL,BYTE PTR DS:[EDX]                  ;   
004030D9  |.  8D51 01      |LEA EDX,DWORD PTR DS:[ECX+1]              ;   
004030DC  |.  F7EA         |IMUL EDX                                     ;   
004030DE  |.  03D8         |ADD EBX,EAX                                 ;   
004030E0  |.  41           |INC ECX                                     ;   
004030E1  |.  FF45 AC      |INC DWORD PTR SS:[EBP-54]

004030E4  |.  3B4D D0      |CMP ECX,DWORD PTR SS:[EBP-30]             ;  循环 1A 次
004030E7  |.^ 7C E9        \JL SHORT 《真正的.004030D2                 ;  这个循环将[EBP-54]里的 ASCII 值逐一取出乘以自身所在位置数，并将结果累加起来放入 EBX 中,这里结果值为 4624
004030E9  |>  33C9         XOR ECX,ECX                                 ;   
004030EB  |>  8B86 A4030000 /MOV EAX,DWORD PTR DS:[ESI+3A4]          ;   
004030F1  |.  F7E9         |IMUL ECX

004030F3  |.  03F8         |ADD EDI,EAX

004030F5  |.  41           |INC ECX

004030F6  |.  83F9 14      |CMP ECX,14

004030F9  |.^ 7C F0        \JL SHORT 《真正的.004030EB                 ;   
004030FB  |.  8B96 A4030000 MOV EDX,DWORD PTR DS:[ESI+3A4]            ;   
```
