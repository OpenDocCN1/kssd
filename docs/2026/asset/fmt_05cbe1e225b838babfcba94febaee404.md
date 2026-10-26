
```asm
00F806B1    90              NOP
00F806B2    90              NOP
00F806B3    FF35 04874500   PUSH DWORD PTR DS:[458704]
00F806B9    034C24 38       ADD ECX,DWORD PTR SS:[ESP+38]            ; 垃圾代码
00F806BD    034C24 18       ADD ECX,DWORD PTR SS:[ESP+18]
00F806C1    59              POP ECX                                  ;
☆☆☆☆☆ 这里也就是MOV ECX,[458704]
00F806C2    8908            MOV DWORD PTR DS:[EAX],ECX            ;
☆☆☆☆☆
00F806C4    FF15 5C644300   CALL DWORD PTR DS:[43645C]               ;
☆☆☆☆☆ msvcrt.\_\_p\_\_commode
00F806CA    83E9 33         SUB ECX,33
00F806CD    83D9 8F         SBB ECX,-71
00F806D0    90              NOP
00F806D1    90              NOP
00F806D2    90              NOP
00F806D3    90              NOP
00F806D4    90              NOP
00F806D5    FF35 00874500   PUSH DWORD PTR DS:[458700]
00F806DB    334C24 28       XOR ECX,DWORD PTR SS:[ESP+28]
00F806DF    83D9 89         SBB ECX,-77
00F806E2    59              POP ECX                                  ;
☆☆☆☆☆ mov ecx,[458700]
00F806E3    8908            MOV DWORD PTR DS:[EAX],ECX            ;
☆☆☆☆☆
00F806E5    64:90           NOP                                      ; Superfluous prefix
00F806E7    90              NOP
00F806E8    90              NOP
00F806E9    90              NOP
00F806EA    8D440B 36       LEA EAX,DWORD PTR DS:[EBX+ECX+36]
00F806EE    FF35 50644300   PUSH DWORD PTR DS:[436450]               ; msvcrt.\_adjust\_fdiv
00F806F4    81C8 FA2D991F   OR EAX,1F992DFA
00F806FA    58              POP EAX                                  ;
☆☆☆☆☆ MOV EAX,[436450]
00F806FB  ^ E9 88FEFFFF     JMP 00F80588
00F80588    8B00            MOV EAX,DWORD PTR DS:[EAX]               ; ☆☆☆☆☆
00F8058A    A3 0C874500     MOV DWORD PTR DS:[45870C],EAX            ; ☆☆☆☆☆
00F8058F    68 9D08F800     PUSH 0F8089D                             ; 这里就是第一个call的代码了
00F80594    E8 67FA1000     CALL 01090000                            ; 跟进
```

进来看看先：

进来后就有更多垃圾代码了，不过不用怕，实际好对付的。

```asm
01090000    64:90           NOP                                      ; 跟进后到这里
01090002    90              NOP
01090003    90              NOP
01090004    90              NOP
01090005    51              PUSH ECX
01090006    9C              PUSHFD
01090007    90              NOP
01090008    90              NOP
01090009    90              NOP
0109000A    90              NOP
0109000B    81C1 9F3A8AB8   ADD ECX,B88A3A9F
01090011    64:90           NOP                                      ; Superfluous prefix
```

……

一直向下找到一个 `call` 寄存器。

```asm
0109010D    0BD1            OR EDX,ECX
0109010F    C1D9 23         RCR ECX,23                               ; Shift constant out of range 1..31
01090112    034C24 38       ADD ECX,DWORD PTR SS:[ESP+38]
01090116    68 A429D600     PUSH 0D629A4
0109011B    81E1 D82E9C40   AND ECX,409C2ED8
01090121    59              POP ECX
01090122    FFD1            CALL ECX                                 ; 向下找到这里,这里继续跟进
01090124    FF7424 10       PUSH DWORD PTR SS:[ESP+10]
01090128    8D4C0B 06       LEA ECX,DWORD PTR DS:[EBX+ECX+6]
```

再次到来到这里：

```asm
00D629A4    55              PUSH EBP
00D629A5    8BEC            MOV EBP,ESP
00D629A7    83C4 F8         ADD ESP,-8
00D629AA    53              PUSH EBX
00D629AB    56              PUSH ESI
00D629AC    57              PUSH EDI
00D629AD    8B5D 08         MOV EBX,DWORD PTR SS:[EBP+8]
```

这里还是很多垃圾代码。

……

```asm
00D62A12    8B45 14         MOV EAX,DWORD PTR SS:[EBP+14]
00D62A15    50              PUSH EAX
00D62A16    E8 CDF3FFFF     CALL 00D61DE8
00D62A1B    50              PUSH EAX
00D62A1C    8BCE            MOV ECX,ESI
00D62A1E    8B55 18         MOV EDX,DWORD PTR SS:[EBP+18]
00D62A21    8BC3            MOV EAX,EBX
00D62A23    E8 8CFDFFFF     CALL 00D627B4                            ; 到这里继续跟进
00D62A28    4F              DEC EDI
00D62A29    0373 70         ADD ESI,DWORD PTR DS:[EBX+70]
00D62A2C    85FF            TEST EDI,EDI
00D62A2E  ^ 77 CB           JA SHORT 00D629FB
```

再跟进来看看

```asm
00D627B4    55              PUSH EBP                                 ; 进到这里
00D627B5    8BEC            MOV EBP,ESP
00D627B7    83C4 F0         ADD ESP,-10
00D627BA    53              PUSH EBX
00D627BB    56              PUSH ESI
00D627BC    57              PUSH EDI
```

……

进来后一直向下找到 `JMP DWORD PTR DS:[EAX+24]`。找到这里：

```asm
00D62959    FF75 0C         PUSH DWORD PTR SS:[EBP+C]
00D6295C    8B45 F8         MOV EAX,DWORD PTR SS:[EBP-8]
00D6295F    FF60 24         JMP DWORD PTR DS:[EAX+24]                ; 找到这里后，直接F4到这里.再按F8一次终于到了我们想到的地方:

010A0000    64:90           NOP                                      ; 进到这里直接到POPFD
010A0002    90              NOP
010A0003    90              NOP
010A0004    90              NOP
010A0005    90              NOP
010A0006    90              NOP
010A0007    90              NOP
010A0008    C1D9 D7         RCR ECX,0D7                              ; Shift constant out of range 1..31
010A000B    36:90           NOP                                      ; Superfluous prefix
010A000D    90              NOP
010A000E    90              NOP
010A000F    8B4C24 10       MOV ECX,DWORD PTR SS:[ESP+10]
010A0013    59              POP ECX
010A0014    035424 18       ADD EDX,DWORD PTR SS:[ESP+18]
```

到这里后，直接CTRL＋S找：

`POPFD`

`POP ESP`

`JMP [ESP-4]`

找到这里：

```asm
010A00BA    59              POP ECX
010A00BB    9D              POPFD
010A00BC    5C              POP ESP
010A00BD  - FF6424 FC       JMP DWORD PTR SS:[ESP-4]                 ; 99mu.00431BDD
```

找到后直接在`010a00bd`处下一个断，下次就不用再重复上面的工作了(当然这个地址，每次载入后不一定相同)。执行到`010a00bd`处后，看到跳去的值就是原程序处，所以我们这里的代码就是：

☆☆☆☆☆ `CALL 431BDD`

继续返回到壳代码中，继续跟出代码来：

```asm
00F8089D    391D 781C4400   CMP DWORD PTR DS:[441C78],EBX            ; ☆☆☆☆☆返回到这里继续
00F808A3    0F85 61000000   JNZ 00F8090A                             ; ☆☆☆☆☆这里的跳转不能直接写，我们要记下先，这里会跳下去。
00F808A9    64:FF35 0000000>PUSH DWORD PTR FS:[0]
00F808B0    90              NOP
00F808B1    90              NOP
00F808B2    90              NOP
00F808B3    66:9C           PUSHFW
00F808B5    57              PUSH EDI
00F808B6    1BFB            SBB EDI,EBX
00F808B8    90              NOP
00F808B9    90              NOP
00F808BA    90              NOP
00F808BB    90              NOP
00F808BC    90              NOP
00F808BD    90              NOP
00F808BE    90              NOP
00F808BF    C1C7 8D         ROL EDI,8D                               ; Shift constant out of range 1..31
00F808C2    90              NOP
00F808C3    90              NOP
00F808C4    90              NOP
00F808C5    90              NOP
00F808C6    83E7 40         AND EDI,40
00F808C9    90              NOP
00F808CA    90              NOP
00F808CB    90              NOP
00F808CC    90              NOP
00F808CD    2BFD            SUB EDI,EBP
00F808CF    8D7C0C 36       LEA EDI,DWORD PTR SS:[ESP+ECX+36]
00F808D3    2BF9            SUB EDI,ECX
00F808D5    8D7C2F CA       LEA EDI,DWORD PTR DS:[EDI+EBP-36]
00F808D9    90              NOP
00F808DA    90              NOP
00F808DB    90              NOP
00F808DC    90              NOP
00F808DD    2BFD            SUB EDI,EBP
00F808DF    36:90           NOP                                      ; Superfluous prefix
00F808E1    90              NOP
00F808E2    90              NOP
00F808E3    8D7C27 06       LEA EDI,DWORD PTR DS:[EDI+6]
00F808E7    55              PUSH EBP
00F808E8    C1C5 97         ROL EBP,97                               ; Shift constant out of range 1..31
00F808EB    C1C5 B3         ROL EBP,0B3                              ; Shift constant out of range 1..31
00F808EE    8DAC26 DA1B4300 LEA EBP,DWORD PTR DS:[ESI+431BDA]
00F808F5    90              NOP
00F808F6    90              NOP
00F808F7    90              NOP
00F808F8    90              NOP
00F808F9    90              NOP
00F808FA    2BEE            SUB EBP,ESI
00F808FC    55              PUSH EBP
00F808FD    8F07            POP DWORD PTR DS:[EDI]                   ; ☆☆☆☆☆ PUSH 431BDA
00F808FF    5D              POP EBP
```
