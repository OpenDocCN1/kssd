
```asm
00414C74    6A 04           PUSH 4                                         ; /Protect = PAGE\_READWRITE

00414C76    68 00300000     PUSH 3000                                      ; |AllocationType = MEM\_COMMIT|MEM\_RESERVE

00414C7B    51              PUSH ECX                                       ; |Size = 61 (97.)

00414C7C    6A 00           PUSH 0                                         ; |Address = NULL

00414C7E    53              PUSH EBX                                       ; |Return address

00414C7F    FFA5 0E4C4000   JMP DWORD PTR SS:[EBP+404C0E]                  ; \VirtualAlloc

00414C85    90              NOP

00414C86    90              NOP

00414C87    90              NOP

00414C88    8DB5 DF564000   LEA ESI,DWORD PTR SS:[EBP+4056DF]

00414C8E    97              XCHG EAX,EDI

00414C8F    8BDF            MOV EBX,EDI

00414C91    B9 26000000     MOV ECX,26

00414C96    F3:A4           REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]   ; from 4a4ada to: D10000 size:26

……

00414CA4    8BB5 E6574000   MOV ESI,DWORD PTR SS:[EBP+4057E6]

00414CAA    03B5 EB574000   ADD ESI,DWORD PTR SS:[EBP+4057EB]

00414CB0    83C6 28         ADD ESI,28

00414CB3    B9 0A000000     MOV ECX,0A                                     ; size

00414CB8    BA A4919C0B     MOV EDX,0B9C91A4

00414CBD    AD              LODS DWORD PTR DS:[ESI]

00414CBE    4A              DEC EDX

00414CBF    03C2            ADD EAX,EDX

00414CC1    42              INC EDX

00414CC2    90              NOP

00414CC3    90              NOP

00414CC4    90              NOP

00414CC5    90              NOP

00414CC6    90              NOP

00414CC7    90              NOP

00414CC8    90              NOP

00414CC9    90              NOP

00414CCA    90              NOP

00414CCB    90              NOP

00414CCC    90              NOP

00414CCD    90              NOP

00414CCE    33C2            XOR EAX,EDX

00414CD0    4A              DEC EDX

00414CD1    C1CA 08         ROR EDX,8

00414CD4    AB              STOS DWORD PTR ES:[EDI]

00414CD5    49              DEC ECX

00414CD6    9C              PUSHFD

00414CD7    90              NOP

00414CD8    90              NOP

00414CD9    90              NOP

00414CDA    90              NOP

00414CDB    90              NOP

00414CDC    90              NOP

00414CDD    90              NOP

00414CDE    90              NOP

00414CDF    90              NOP

00414CE0    C12C24 06       SHR DWORD PTR SS:[ESP],6

00414CE4    F71424          NOT DWORD PTR SS:[ESP]

00414CE7    832424 01       AND DWORD PTR SS:[ESP],1

00414CEB    50              PUSH EAX

00414CEC    52              PUSH EDX

00414CED    B8 635A9AF0     MOV EAX,F09A5A63

00414CF2    05 46A5650F     ADD EAX,0F65A546

00414CF7    F76424 08       MUL DWORD PTR SS:[ESP+8]

00414CFB    8D8428 19594000 LEA EAX,DWORD PTR DS:[EAX+EBP+405919]

00414D02    894424 08       MOV DWORD PTR SS:[ESP+8],EAX

00414D06    5A              POP EDX

00414D07    58              POP EAX

00414D08    8D6424 04       LEA ESP,DWORD PTR SS:[ESP+4]

00414D0C    FF6424 FC       JMP DWORD PTR SS:[ESP-4]                       ; PESpin.00414D14

……

00414D14    B9 13000000     MOV ECX,13

00414D19    8DB5 05574000   LEA ESI,DWORD PTR SS:[EBP+405705]

00414D1F    F3:A4           REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]   ; From:414B00 to:D1004E size:13

00414D21    93              XCHG EAX,EBX

00414D22    B9 0A000000     MOV ECX,0A

00414D27    8BBD E6574000   MOV EDI,DWORD PTR SS:[EBP+4057E6]

00414D2D    03BD EB574000   ADD EDI,DWORD PTR SS:[EBP+4057EB]

00414D33    83C7 28         ADD EDI,28

00414D36    F3:AB           REP STOS DWORD PTR ES:[EDI]

00414D38    58              POP EAX

00414D39    90              NOP

00414D3A    90              NOP

00414D3B    90              NOP

00414D3C    90              NOP

00414D3D    90              NOP

00414D3E    90              NOP

00414D3F    90              NOP

00414D40    90              NOP

00414D41    90              NOP

00414D42    2D F9FFFFFF     SUB EAX,-7

00414D47    90              NOP

00414D48    90              NOP

00414D49    90              NOP

00414D4A    90              NOP

00414D4B    90              NOP

00414D4C    90              NOP

00414D4D    90              NOP

00414D4E    90              NOP

00414D4F    90              NOP

00414D50    90              NOP

00414D51    90              NOP

00414D52    90              NOP

00414D53    90              NOP

00414D54    90              NOP

00414D55    90              NOP

00414D56    90              NOP

00414D57    90              NOP

00414D58  ^ FFE0            JMP EAX                                        ; PESpin.0041317D

……

004132F6    F685 A15D4000 0>TEST BYTE PTR SS:[EBP+405DA1],1                ; 这里判断是否选择了API重定位，0表示不加密，1表示加密

004132FD    74 51           JE SHORT 00413350

004132FF    90              NOP

00413300    90              NOP

00413301    90              NOP

00413302    90              NOP

00413303    90              NOP

00413304    90              NOP

00413305    90              NOP

00413306    90              NOP

00413307    90              NOP

00413308    90              NOP

00413309    90              NOP

0041330A    90              NOP

0041330B    90              NOP

0041330C    90              NOP

0041330D    90              NOP

0041330E    90              NOP

0041330F    90              NOP

00413310    BB 3C080000     MOV EBX,83C                                    ; 重定位api大小

00413315    0BDB            OR EBX,EBX

00413317    74 37           JE SHORT 00413350                              ; 如果重定位API大小为0就跳

00413319    2BC0                  SUB EAX,EAX

0041331B    2185 D14B4000         AND DWORD PTR SS:[EBP+404BD1],EAX

00413321    E8 01000000           CALL 00413327

00413326    90                    NOP

00413327    59                    POP ECX

00413328    6A 40                 PUSH 40                                        ; /Protect = PAGE\_EXECUTE\_READWRITE

0041332A    68 00300000           PUSH 3000                                      ; |AllocationType = MEM\_COMMIT|MEM\_RESERVE

0041332F    53                    PUSH EBX                                       ; |Size = 83C (2108.)

00413330    50                    PUSH EAX                                       ; |Address = NULL

00413331    8D6424 FC             LEA ESP,DWORD PTR SS:[ESP-4]                   ; |

00413335    81C1 23000000         ADD ECX,23                                     ; |

0041333B    890C24                MOV DWORD PTR SS:[ESP],ECX                     ; |Return Address

0041333E    FFA5 0E4C4000         JMP DWORD PTR SS:[EBP+404C0E]                  ; \VirtualAlloc

00413344    90                    NOP

00413345    85C0                  TEST EAX,EAX

00413347    74 21                 JE SHORT 0041336A

00413349    50                    PUSH EAX

0041334A    8F85 C94B4000         POP DWORD PTR SS:[EBP+404BC9]                  ; [EBP+404BC9]保存hmem(00D20000)

00413350    8D85 4A0D3400         LEA EAX,DWORD PTR SS:[EBP+340D4A]

00413356    8D80 5F320C00         LEA EAX,DWORD PTR DS:[EAX+C325F]

0041335C    48                    DEC EAX

0041335D    FFD0                  CALL EAX                                       ; 004133A3

……

00414F25    6A 04                 PUSH 4                                         ; /Protect = PAGE\_READWRITE

00414F27    68 00300000           PUSH 3000                                      ; |AllocationType = MEM\_COMMIT|MEM\_RESERVE

00414F2C    51                    PUSH ECX                                       ; |Size = 5C (92.)

00414F2D    6A 00                 PUSH 0                                         ; |Address = NULL

00414F2F    53                    PUSH EBX                                       ; |Return address

00414F30    FFA5 0E4C4000         JMP DWORD PTR SS:[EBP+404C0E]                  ; \VirtualAlloc

00414F36    0F01FE                INVLPG DH                                      ; Privileged command
```
