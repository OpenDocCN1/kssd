
```
00604106    75 5E             JNZ     SHORT TimeHelp.00604166******没有跳!

00604108    FE85 10404400    INC     BYTE PTR SS:[EBP+444010]

0060410E    8B3E             MOV     EDI, DWORD PTR DS:[ESI]

00604110    03BD FC494400    ADD     EDI, DWORD PTR SS:[EBP+4449FC］

00604116    FF37             PUSH    DWORD PTR DS:[EDI]

00604118    C607 C3          MOV     BYTE PTR DS:[EDI], 0C3

0060411B    FFD7             CALL    EDI

00604128    8BB5 F43F4400    MOV     ESI, DWORD PTR SS:[EBP+443FF4]*****F8 到这里，继续往下!

0060412E    33DB             XOR     EBX, EBX

00604130    0BC9             OR      ECX, ECX

00604132    74 2E            JE      SHORT TimeHelp.00604162********平安无事!

00604134    78 2C            JS      SHORT TimeHelp.00604162*******平安无事!继续走!

00604136    AC               LODS    BYTE PTR DS:[ESI]

00604137    3C E8            CMP     AL, 0E8

00604139    74 0A            JE      SHORT TimeHelp.00604145*******平安无事!继续走!

0060413B    EB 00            JMP     SHORT TimeHelp.0060413D********跳!

0060413D    3C E9            CMP     AL, 0E9******************************去!跳到这里!搞什么飞机!

0060413F    74 04            JE      SHORT TimeHelp.00604145*********不跳!

00604141    43               INC     EBX

00604142    49               DEC     ECX

00604143  ^ EB EB            JMP     SHORT TimeHelp.00604130**********不好!要回去了!F4 下去!

00604145    8B06             MOV     EAX, DWORD PTR DS:[ESI]*********在这里 F4 下来!

00604147    EB 00            JMP     SHORT TimeHelp.00604149

00604149    803E 2A          CMP     BYTE PTR DS:[ESI], 2A

0060414C  ^ 75 F3            JNZ     SHORT TimeHelp.00604141*********又要回去？去吧!

0060414E    24 00            AND     AL, 0**************************在这一行 F4 下来!

00604150    C1C0 18          ROL     EAX, 18

00604153    2BC3             SUB     EAX, EBX

00604155    8906             MOV     DWORD PTR DS:[ESI], EAX

00604157    83C3 05          ADD     EBX, 5

0060415A    83C6 04          ADD     ESI, 4

0060415D    83E9 05          SUB     ECX, 5

00604160  ^ EB CE            JMP     SHORT TimeHelp.00604130*********又要回去!下去吧!

00604162    5B               POP     EBX***************************在这一行 F4 下来!

00604163    5E               POP     ESI

00604164    59               POP     ECX

00604165    58               POP     EAX

00604166    8BC8             MOV     ECX, EAX

00604168    8B3E             MOV     EDI, DWORD PTR DS:[ESI]

0060416A    03BD FC494400    ADD     EDI, DWORD PTR SS:[EBP+4449FC]

00604170    8BB5 F43F4400    MOV     ESI, DWORD PTR SS:[EBP+443FF4]

00604176    C1F9 02          SAR     ECX, 2

00604179    F3:A5            REP     MOVS DWORD PTR ES:[EDI], DWORD P>

0060417B    8BC8             MOV     ECX, EAX

0060417D    83E1 03          AND     ECX, 3

00604180    F3:A4            REP     MOVS BYTE PTR ES:[EDI], BYTE PTR>

00604182    5E               POP     ESI

00604183    68 00800000      PUSH    8000

00604188    6A 00            PUSH    0

0060418A    FFB5 F43F4400    PUSH    DWORD PTR SS:[EBP+443FF4]

00604190    FF95 00404400    CALL    DWORD PTR SS:[EBP+444000]

00604196    83C6 08          ADD     ESI, 8

00604199    833E 00          CMP     DWORD PTR DS:[ESI], 0

0060419C  ^ 0F85 26FFFFFF    JNZ     TimeHelp.006040C8********这里又要回去!F4 下去!

006041A2    68 00800000      PUSH    8000**************************在这一行 F4 下来!

006041A7    6A 00            PUSH    0

006041A9    FFB5 F83F4400    PUSH    DWORD PTR SS:[EBP+443FF8]

006041AF    FF95 00404400    CALL    DWORD PTR SS:[EBP+444000]

006041B5    8B9D D83F4400    MOV     EBX, DWORD PTR SS:[EBP+443FD8]

006041BB    0BDB             OR      EBX, EBX

006041BD    74 08            JE      SHORT TimeHelp.006041C7******跳，跳到这里!

006041BF    8B03             MOV     EAX, DWORD PTR DS:[EBX]

006041C1    8785 DC3F4400    XCHG    DWORD PTR SS:[EBP+443FDC], EAX

006041C7    8B95 FC494400    MOV     EDX, DWORD PTR SS:[EBP+4449FC]***跳到这里!F8 往下!

006041CD    8B85 D43F4400    MOV     EAX, DWORD PTR SS:[EBP+443FD4]

006041D3    2BD0             SUB     EDX, EAX

006041D5    74 79            JE      SHORT TimeHelp.00604250******再跳!

00604250    8B95 FC494400    MOV     EDX, DWORD PTR SS:[EBP+4449FC]***跳到这里!

00604256    8BB5 E83F4400    MOV     ESI, DWORD PTR SS:[EBP+443FE8]

0060425C    0BF6             OR      ESI, ESI

0060425E    74 11            JE      SHORT TimeHelp.00604271******再跳!

00604271    8BB5 B1394400    MOV     ESI, DWORD PTR SS:[EBP+4439B1]***跳到这里!

00604271    8BB5 B1394400    MOV     ESI, DWORD PTR SS:[EBP+4439B1]

00604277    8B95 FC494400    MOV     EDX, DWORD PTR SS:[EBP+4449FC]

0060427D    03F2             ADD     ESI, EDX

0060427F    8B46 0C          MOV     EAX, DWORD PTR DS:[ESI+C]

00604282    85C0             TEST    EAX, EAX

00604284    0F84 0A010000    JE      TimeHelp.00604394

00604295    85C0             TEST    EAX, EAX

00604297    75 07            JNZ     SHORT TimeHelp.006042A0******这里跳!

00604299    53               PUSH    EBX

0060429A    FF95 044B4400    CALL    DWORD PTR SS:[EBP+444B04]

006042A0    8985 EC3F4400    MOV     DWORD PTR SS:[EBP+443FEC], EAX ***跳到这里!

006042A6    C785 F03F4400 0>MOV     DWORD PTR SS:[EBP+443FF0], 0

006042B0    8B95 FC494400    MOV     EDX, DWORD PTR SS:[EBP+4449FC]

006042B6    8B06             MOV     EAX, DWORD PTR DS:[ESI]

006042B8    85C0             TEST    EAX, EAX

006042BA    75 03            JNZ     SHORT TimeHelp.006042BF

006042FC    85C0             TEST    EAX, EAX

006042FE    5B               POP     EBX

006042FF    75 6F            JNZ     SHORT TimeHelp.00604370******在这里跳了!往下看!

00604301    F7C3 00000080    TEST    EBX, 80000000

00604307    75 19            JNZ     SHORT TimeHelp.00604322

00604309    57               PUSH    EDI

00604370    8907             MOV     DWORD PTR DS:[EDI], EAX********跳到了这里!往下看看!多美妙!

00604372    8385 F03F4400 0>ADD     DWORD PTR SS:[EBP+443FF0], 4

00604379  ^ E9 32FFFFFF      JMP     TimeHelp.006042B0********回跳!

0060437E    8906             MOV     DWORD PTR DS:[ESI], EAX

00604380    8946 0C          MOV     DWORD PTR DS:[ESI+C], EAX

00604383    8946 10          MOV     DWORD PTR DS:[ESI+10], EAX

00604386    83C6 14          ADD     ESI, 14

00604389    8B95 FC494400    MOV     EDX, DWORD PTR SS:[EBP+4449FC]

0060438F  ^ E9 EBFEFFFF      JMP     TimeHelp.0060427F********回跳!

00604394    8B85 AD394400    MOV     EAX, DWORD PTR SS:[EBP+4439AD]

0060439A    50               PUSH    EAX

0060439B    0385 FC494400    ADD     EAX, DWORD PTR SS:[EBP+4449FC]

006043A1    59               POP     ECX

006043A2    0BC9             OR      ECX, ECX

006043A4    8985 E63C4400    MOV     DWORD PTR SS:[EBP+443CE6], EAX
```
