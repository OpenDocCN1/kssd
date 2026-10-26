
```
00327DD9    83BD F3D24000 0>CMP DWORD PTR SS:[EBP+40D2F3],0

00327DE0    74 33           JE SHORT 00327E15                   ; 强行跳过驱动

00327DE2    60              PUSHAD

00327DE3    CD20 8BC48B64   VxDJump 648BC48B

00327DE9    24 08           AND AL,8

00327DEB    50              PUSH EAX

00327DEC    60              PUSHAD

00327DED    56              PUSH ESI

00327DEE    0F014C24 FE     SIDT FWORD PTR SS:[ESP-2]

00327DF3    5E              POP ESI

00327DF4    66:8B85 0E51400>MOV AX,WORD PTR SS:[EBP+40510E]

00327DFB    66:8B9D 1051400>MOV BX,WORD PTR SS:[EBP+405110]

00327E02    66:8946 18      MOV WORD PTR DS:[ESI+18],AX

00327E06    66:895E 1E      MOV WORD PTR DS:[ESI+1E],BX

00327E0A    61              POPAD

00327E0B    5C              POP ESP

00327E0C    8D85 71514000   LEA EAX,DWORD PTR SS:[EBP+405171]

00327E12    50              PUSH EAX

00327E13    CF              IRETD

00327E14    61              POPAD

00327E15    61              POPAD

00327E16    C3              RETN

...
```

```
0032FB10    60              PUSHAD

0032FB11    8B85 BD890000   MOV EAX,DWORD PTR SS:[EBP+89BD]

0032FB17    0385 C5890000   ADD EAX,DWORD PTR SS:[EBP+89C5]

0032FB1D    E8 00000000     CALL 0032FB22

0032FDFB    50              PUSH EAX                                 ; 17x.00403A3F

0032FDFC    8D78 02         LEA EDI,DWORD PTR DS:[EAX+2]

0032FDFF    EB 02           JMP SHORT 0032FE03

0032FE01    61              POPAD

0032FE02    50              PUSH EAX

0032FE03    C3              RETN                                    ; to OEP

00403A3F    FFD7            CALL EDI                                 ; 17x.00403A41

00403A41    58              POP EAX

00403A42    90              NOP

00403A43    90              NOP

00403A44    E8 FD980000     CALL 17x.0040D346                        ; JMP to comdlg32.GetOpenFileNameA

00403A49    85C0            TEST EAX,EAX
```

前5个字节是 Stolen Code, 看一下 STACK,

```
003275D0   0040F034  17x.0040F034

003275D4   004327F9  RETURN to 17x.004327F9

;
```

```
00403A3F                    PUSH 40F034           ;OK
```

再把 `40E170-40E1A7` 的数据复制到 `40E000-40E037`, Dump

IAT 只要用 LordPE 改一下 RVA = `E120`, Size=`50`

[附件:17x.zip](https://bbs.kanxue.com/bbs.pediy.com/upload/2005/8/files/17x.zip_278.zip)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
