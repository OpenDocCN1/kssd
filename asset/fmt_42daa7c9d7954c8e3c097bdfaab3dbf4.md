
```
34501000    8B5424 04             MOV     EDX,DWORD PTR SS:\[ESP+4\]
34501004    83C8 FF               OR      EAX,FFFFFFFF
34501007    8A0A                  MOV     CL,BYTE PTR DS:\[EDX\]
34501009    84C9                  TEST    CL,CL
3450100B    74 27                 JE      SHORT 34501034
3450100D    81E1 FF000000         AND     ECX,0FF
34501013    33C1                  XOR     EAX,ECX
34501015    B9 08000000           MOV     ECX,8
3450101A    A8 01                 TEST    AL,1
3450101C    74 09                 JE      SHORT 34501027
3450101E    D1E8                  SHR     EAX,1
34501020    35 2083B8ED           XOR     EAX,EDB88320
34501025    EB 02                 JMP     SHORT 34501029
34501027    D1E8                  SHR     EAX,1
34501029    49                    DEC     ECX
3450102A  ^ 75 EE                 JNZ     SHORT 3450101A
3450102C    8A4A 01               MOV     CL,BYTE PTR DS:\[EDX+1\]
3450102F    42                    INC     EDX
34501030    84C9                  TEST    CL,CL
34501032  ^ 75 D9                 JNZ     SHORT 3450100D
34501034    C3                    RETN
34501035    90                    NOP
34501036    90                    NOP
34501037    90                    NOP
34501038    90                    NOP
34501039    90                    NOP
3450103A    90                    NOP
3450103B    90                    NOP
3450103C    90                    NOP
3450103D    90                    NOP
3450103E    90                    NOP
3450103F    90                    NOP
34501040    83EC 14               SUB     ESP,14
34501043    8D4424 00             LEA     EAX,DWORD PTR SS:\[ESP\]
34501047    53                    PUSH    EBX
34501048    55                    PUSH    EBP
34501049    56                    PUSH    ESI
3450104A    57                    PUSH    EDI
3450104B    50                    PUSH    EAX
3450104C    6A 40                 PUSH    40
3450104E    68 00301B00           PUSH    1B3000
34501053    68 00104000           PUSH    401000
34501058    FF15 00505034         CALL    DWORD PTR DS:\[34505000\]          ; kernel32.VirtualProtect
3450105E    8B6C24 2C             MOV     EBP,DWORD PTR SS:\[ESP+2C\]
34501062    BE 01104000           MOV     ESI,401001
34501067    B3 8B                 MOV     BL,8B
34501069    66:817E FF 8DC0       CMP     WORD PTR DS:\[ESI-1\],0C08D
3450106F    0F85 BB030000         JNZ     34501430
34501075    8D4E FF               LEA     ECX,DWORD PTR DS:\[ESI-1\]
34501078    8D5424 14             LEA     EDX,DWORD PTR SS:\[ESP+14\]
3450107C    51                    PUSH    ECX
3450107D    68 30605034           PUSH    34506030                         ; ASCII "%08X"
34501082    52                    PUSH    EDX
34501083    FF15 B4505034         CALL    DWORD PTR DS:\[345050B4\]          ; USER32.wsprintfA
34501089    8D4424 20             LEA     EAX,DWORD PTR SS:\[ESP+20\]
3450108D    50                    PUSH    EAX
3450108E    E8 6DFFFFFF           CALL    34501000
34501093    8BF8                  MOV     EDI,EAX
34501095    B8 A38B2EBA           MOV     EAX,BA2E8BA3
3450109A    F7E5                  MUL     EBP
3450109C    83C4 10               ADD     ESP,10
3450109F    33C9                  XOR     ECX,ECX
345010A1    C1EA 03               SHR     EDX,3
345010A4    0F84 86030000         JE      34501430
345010AA    8B4424 28             MOV     EAX,DWORD PTR SS:\[ESP+28\]
345010AE    3B38                  CMP     EDI,DWORD PTR DS:\[EAX\]
345010B0    74 0D                 JE      SHORT 345010BF
345010B2    41                    INC     ECX
345010B3    83C0 0B               ADD     EAX,0B
345010B6    3BCA                  CMP     ECX,EDX
345010B8  ^ 72 F4                 JB      SHORT 345010AE
345010BA    E9 71030000           JMP     34501430
345010BF    83F9 FF               CMP     ECX,-1
345010C2    0F84 68030000         JE      34501430
345010C8    8B4424 28             MOV     EAX,DWORD PTR SS:\[ESP+28\]
345010CC    8D1489                LEA     EDX,DWORD PTR DS:\[ECX+ECX\*4\]
345010CF    03C1                  ADD     EAX,ECX
345010D1    8A4C50 08             MOV     CL,BYTE PTR DS:\[EAX+EDX\*2+8\]
345010D5    84C9                  TEST    CL,CL
345010D7    8D0450                LEA     EAX,DWORD PTR DS:\[EAX+EDX\*2\]
345010DA    0F85 CB000000         JNZ     345011AB
345010E0    8A48 09               MOV     CL,BYTE PTR DS:\[EAX+9\]
345010E3    83E1 07               AND     ECX,7
345010E6    83F9 04               CMP     ECX,4
345010E9    75 64                 JNZ     SHORT 3450114F
345010EB    8A48 0A               MOV     CL,BYTE PTR DS:\[EAX+A\]
345010EE    80F9 03               CMP     CL,3
345010F1    75 14                 JNZ     SHORT 34501107
345010F3    885E FF               MOV     BYTE PTR DS:\[ESI-1\],BL
345010F6    8A48 09               MOV     CL,BYTE PTR DS:\[EAX+9\]
345010F9    80E1 3F               AND     CL,3F
345010FC    C646 01 24            MOV     BYTE PTR DS:\[ESI+1\],24
34501100    880E                  MOV     BYTE PTR DS:\[ESI\],CL
34501102    E9 29030000           JMP     34501430
34501107    80F9 04               CMP     CL,4
3450110A    75 1D                 JNZ     SHORT 34501129
3450110C    885E FF               MOV     BYTE PTR DS:\[ESI-1\],BL
3450110F    8A50 09               MOV     DL,BYTE PTR DS:\[EAX+9\]
34501112    80E2 3F               AND     DL,3F
34501115    C646 01 24            MOV     BYTE PTR DS:\[ESI+1\],24
34501119    80CA 40               OR      DL,40
3450111C    8816                  MOV     BYTE PTR DS:\[ESI\],DL
3450111E    8A40 04               MOV     AL,BYTE PTR DS:\[EAX+4\]
34501121    8846 02               MOV     BYTE PTR DS:\[ESI+2\],AL
34501124    E9 07030000           JMP     34501430
34501129    80F9 07               CMP     CL,7
3450112C    0F85 FE020000         JNZ     34501430
34501132    885E FF               MOV     BYTE PTR DS:\[ESI-1\],BL
34501135    8A48 09               MOV     CL,BYTE PTR DS:\[EAX+9\]
34501138    80E1 3F               AND     CL,3F
3450113B    C646 01 24            MOV     BYTE PTR DS:\[ESI+1\],24
3450113F    80C9 80               OR      CL,80
34501142    880E                  MOV     BYTE PTR DS:\[ESI\],CL
34501144    8B50 04               MOV     EDX,DWORD PTR DS:\[EAX+4\]
34501147    8956 02               MOV     DWORD PTR DS:\[ESI+2\],EDX
3450114A    E9 E1020000           JMP     34501430
3450114F    83F9 05               CMP     ECX,5
34501152    8A48 0A               MOV     CL,BYTE PTR DS:\[EAX+A\]
34501155    75 40                 JNZ     SHORT 34501197
34501157    80F9 03               CMP     CL,3
3450115A    75 19                 JNZ     SHORT 34501175
3450115C    885E FF               MOV     BYTE PTR DS:\[ESI-1\],BL
3450115F    8A48 09               MOV     CL,BYTE PTR DS:\[EAX+9\]
34501162    80E1 3F               AND     CL,3F
34501165    80C9 40               OR      CL,40
34501168    880E                  MOV     BYTE PTR DS:\[ESI\],CL
3450116A    8A50 04               MOV     DL,BYTE PTR DS:\[EAX+4\]
3450116D    8856 01               MOV     BYTE PTR DS:\[ESI+1\],DL
34501170    E9 BB020000           JMP     34501430
34501175    80F9 06               CMP     CL,6
34501178    0F85 B2020000         JNZ     34501430
3450117E    885E FF               MOV     BYTE PTR DS:\[ESI-1\],BL
34501181    8A48 09               MOV     CL,BYTE PTR DS:\[EAX+9\]
34501184    80E1 3F               AND     CL,3F
34501187    80C9 80               OR      CL,80
3450118A    880E                  MOV     BYTE PTR DS:\[ESI\],CL
3450118C    8B50 04               MOV     EDX,DWORD PTR DS:\[EAX+4\]
3450118F    8956 01               MOV     DWORD PTR DS:\[ESI+1\],EDX
34501192    E9 99020000           JMP     34501430
34501197    80F9 02               CMP     CL,2
3450119A  ^ 75 BB                 JNZ     SHORT 34501157
3450119C    885E FF               MOV     BYTE PTR DS:\[ESI-1\],BL
3450119F    8A40 09               MOV     AL,BYTE PTR DS:\[EAX+9\]
345011A2    24 3F                 AND     AL,3F
345011A4    8806                  MOV     BYTE PTR DS:\[ESI\],AL
345011A6    E9 85020000           JMP     34501430
345011AB    80F9 01               CMP     CL,1
345011AE    0F85 B3000000         JNZ     34501267
345011B4    33C9                  XOR     ECX,ECX
345011B6    8A48 09               MOV     CL,BYTE PTR DS:\[EAX+9\]
345011B9    8BD1                  MOV     EDX,ECX
345011BB    C1E9 04               SHR     ECX,4
345011BE    83E2 07               AND     EDX,7
345011C1    83E1 07               AND     ECX,7
345011C4    8078 0A 02            CMP     BYTE PTR DS:\[EAX+A\],2
345011C8    0F85 62020000         JNZ     34501430
345011CE    8B40 04               MOV     EAX,DWORD PTR DS:\[EAX+4\]
345011D1    83F8 06               CMP     EAX,6
345011D4    0F83 56020000         JNB     34501430
345011DA    85C0                  TEST    EAX,EAX
345011DC    75 13                 JNZ     SHORT 345011F1
345011DE    80C9 F8               OR      CL,0F8
345011E1    C646 FF 09            MOV     BYTE PTR DS:\[ESI-1\],9
345011E5    C0E1 03               SHL     CL,3
345011E8    0ACA                  OR      CL,DL
345011EA    880E                  MOV     BYTE PTR DS:\[ESI\],CL
345011EC    E9 3F020000           JMP     34501430
345011F1    83F8 01               CMP     EAX,1
345011F4    75 13                 JNZ     SHORT 34501209
345011F6    80C9 F8               OR      CL,0F8
345011F9    C646 FF 21            MOV     BYTE PTR DS:\[ESI-1\],21
345011FD    C0E1 03               SHL     CL,3
34501200    0ACA                  OR      CL,DL
34501202    880E                  MOV     BYTE PTR DS:\[ESI\],CL
34501204    E9 27020000           JMP     34501430
34501209    83F8 02               CMP     EAX,2
3450120C    75 13                 JNZ     SHORT 34501221
3450120E    80C9 F8               OR      CL,0F8
34501211    C646 FF 33            MOV     BYTE PTR DS:\[ESI-1\],33
34501215    C0E1 03               SHL     CL,3
34501218    0ACA                  OR      CL,DL
3450121A    880E                  MOV     BYTE PTR DS:\[ESI\],CL
3450121C    E9 0F020000           JMP     34501430
34501221    83F8 03               CMP     EAX,3
34501224    75 12                 JNZ     SHORT 34501238
34501226    80C9 F8               OR      CL,0F8
34501229    8846 FF               MOV     BYTE PTR DS:\[ESI-1\],AL
3450122C    C0E1 03               SHL     CL,3
3450122F    0ACA                  OR      CL,DL
34501231    880E                  MOV     BYTE PTR DS:\[ESI\],CL
34501233    E9 F8010000           JMP     34501430
34501238    83F8 04               CMP     EAX,4
3450123B    75 13                 JNZ     SHORT 34501250
3450123D    80C9 F8               OR      CL,0F8
34501240    C646 FF 2B            MOV     BYTE PTR DS:\[ESI-1\],2B
34501244    C0E1 03               SHL     CL,3
34501247    0ACA                  OR      CL,DL
34501249    880E                  MOV     BYTE PTR DS:\[ESI\],CL
3450124B    E9 E0010000           JMP     34501430
34501250    83F8 05               CMP     EAX,5
34501253    75 03                 JNZ     SHORT 34501258
34501255    885E FF               MOV     BYTE PTR DS:\[ESI-1\],BL
34501258    80C9 F8               OR      CL,0F8
3450125B    C0E1 03               SHL     CL,3
3450125E    0ACA                  OR      CL,DL
34501260    880E                  MOV     BYTE PTR DS:\[ESI\],CL
34501262    E9 C9010000           JMP     34501430
34501267    80F9 02               CMP     CL,2
3450126A    75 6B                 JNZ     SHORT 345012D7
3450126C    8A48 09               MOV     CL,BYTE PTR DS:\[EAX+9\]
3450126F    83E1 07               AND     ECX,7
34501272    83F9 04               CMP     ECX,4
34501275    75 20                 JNZ     SHORT 34501297
34501277    8078 0A 07            CMP     BYTE PTR DS:\[EAX+A\],7
3450127B    0F85 AF010000         JNZ     34501430
34501281    C646 FF C7            MOV     BYTE PTR DS:\[ESI-1\],0C7
34501285    C606 04               MOV     BYTE PTR DS:\[ESI\],4
34501288    C646 01 24            MOV     BYTE PTR DS:\[ESI+1\],24
3450128C    8B40 04               MOV     EAX,DWORD PTR DS:\[EAX+4\]
3450128F    8946 02               MOV     DWORD PTR DS:\[ESI+2\],EAX
34501292    E9 99010000           JMP     34501430
34501297    83F9 05               CMP     ECX,5
3450129A    75 20                 JNZ     SHORT 345012BC
3450129C    8078 0A 07            CMP     BYTE PTR DS:\[EAX+A\],7
345012A0    0F85 8A010000         JNZ     34501430
345012A6    C646 FF C7            MOV     BYTE PTR DS:\[ESI-1\],0C7
345012AA    C606 45               MOV     BYTE PTR DS:\[ESI\],45
345012AD    C646 01 00            MOV     BYTE PTR DS:\[ESI+1\],0
345012B1    8B48 04               MOV     ECX,DWORD PTR DS:\[EAX+4\]
345012B4    894E 02               MOV     DWORD PTR DS:\[ESI+2\],ECX
345012B7    E9 74010000           JMP     34501430
345012BC    8078 0A 06            CMP     BYTE PTR DS:\[EAX+A\],6
345012C0    0F85 6A010000         JNZ     34501430
345012C6    C646 FF C7            MOV     BYTE PTR DS:\[ESI-1\],0C7
345012CA    880E                  MOV     BYTE PTR DS:\[ESI\],CL
345012CC    8B50 04               MOV     EDX,DWORD PTR DS:\[EAX+4\]
345012CF    8956 01               MOV     DWORD PTR DS:\[ESI+1\],EDX
345012D2    E9 59010000           JMP     34501430
345012D7    80F9 03               CMP     CL,3
345012DA    75 5B                 JNZ     SHORT 34501337
345012DC    8A48 0A               MOV     CL,BYTE PTR DS:\[EAX+A\]
345012DF    80F9 02               CMP     CL,2
345012E2    75 23                 JNZ     SHORT 34501307
345012E4    8A48 09               MOV     CL,BYTE PTR DS:\[EAX+9\]
345012E7    84C9                  TEST    CL,CL
345012E9    75 0E                 JNZ     SHORT 345012F9
345012EB    C646 FF 74            MOV     BYTE PTR DS:\[ESI-1\],74
345012EF    8A40 04               MOV     AL,BYTE PTR DS:\[EAX+4\]
345012F2    8806                  MOV     BYTE PTR DS:\[ESI\],AL
345012F4    E9 37010000           JMP     34501430
345012F9    C646 FF 75            MOV     BYTE PTR DS:\[ESI-1\],75
345012FD    8A48 04               MOV     CL,BYTE PTR DS:\[EAX+4\]
34501300    880E                  MOV     BYTE PTR DS:\[ESI\],CL
34501302    E9 29010000           JMP     34501430
34501307    80F9 06               CMP     CL,6
3450130A    0F85 20010000         JNZ     34501430
34501310    8A48 09               MOV     CL,BYTE PTR DS:\[EAX+9\]
34501313    C646 FF 0F            MOV     BYTE PTR DS:\[ESI-1\],0F
34501317    84C9                  TEST    CL,CL
34501319    75 0E                 JNZ     SHORT 34501329
3450131B    C606 84               MOV     BYTE PTR DS:\[ESI\],84
3450131E    8B50 04               MOV     EDX,DWORD PTR DS:\[EAX+4\]
34501321    8956 01               MOV     DWORD PTR DS:\[ESI+1\],EDX
34501324    E9 07010000           JMP     34501430
34501329    C606 85               MOV     BYTE PTR DS:\[ESI\],85
3450132C    8B40 04               MOV     EAX,DWORD PTR DS:\[EAX+4\]
3450132F    8946 01               MOV     DWORD PTR DS:\[ESI+1\],EAX
34501332    E9 F9000000           JMP     34501430
34501337    80F9 04               CMP     CL,4
3450133A    75 5B                 JNZ     SHORT 34501397
3450133C    8A48 0A               MOV     CL,BYTE PTR DS:\[EAX+A\]
3450133F    80F9 02               CMP     CL,2
34501342    75 22                 JNZ     SHORT 34501366
34501344    8078 09 05            CMP     BYTE PTR DS:\[EAX+9\],5
34501348    75 0E                 JNZ     SHORT 34501358
3450134A    C646 FF 72            MOV     BYTE PTR DS:\[ESI-1\],72
3450134E    8A48 04               MOV     CL,BYTE PTR DS:\[EAX+4\]
34501351    880E                  MOV     BYTE PTR DS:\[ESI\],CL
34501353    E9 D8000000           JMP     34501430
34501358    C646 FF 73            MOV     BYTE PTR DS:\[ESI-1\],73
3450135C    8A50 04               MOV     DL,BYTE PTR DS:\[EAX+4\]
3450135F    8816                  MOV     BYTE PTR DS:\[ESI\],DL
34501361    E9 CA000000           JMP     34501430
34501366    80F9 06               CMP     CL,6
34501369    0F85 C1000000         JNZ     34501430
3450136F    8A48 09               MOV     CL,BYTE PTR DS:\[EAX+9\]
34501372    C646 FF 0F            MOV     BYTE PTR DS:\[ESI-1\],0F
34501376    80F9 05               CMP     CL,5
34501379    75 0E                 JNZ     SHORT 34501389
3450137B    C606 82               MOV     BYTE PTR DS:\[ESI\],82
3450137E    8B40 04               MOV     EAX,DWORD PTR DS:\[EAX+4\]
34501381    8946 01               MOV     DWORD PTR DS:\[ESI+1\],EAX
34501384    E9 A7000000           JMP     34501430
34501389    C606 83               MOV     BYTE PTR DS:\[ESI\],83
3450138C    8B48 04               MOV     ECX,DWORD PTR DS:\[EAX+4\]
3450138F    894E 01               MOV     DWORD PTR DS:\[ESI+1\],ECX
34501392    E9 99000000           JMP     34501430
34501397    80F9 05               CMP     CL,5
3450139A    75 27                 JNZ     SHORT 345013C3
3450139C    C646 FF FF            MOV     BYTE PTR DS:\[ESI-1\],0FF
345013A0    C606 24               MOV     BYTE PTR DS:\[ESI\],24
345013A3    8A48 09               MOV     CL,BYTE PTR DS:\[EAX+9\]
345013A6    8AD1                  MOV     DL,CL
345013A8    80E1 F1               AND     CL,0F1
345013AB    80E2 07               AND     DL,7
345013AE    D0E2                  SHL     DL,1
345013B0    0AD1                  OR      DL,CL
345013B2    C0E2 02               SHL     DL,2
345013B5    80CA 05               OR      DL,5
345013B8    8856 01               MOV     BYTE PTR DS:\[ESI+1\],DL
345013BB    8B40 04               MOV     EAX,DWORD PTR DS:\[EAX+4\]
345013BE    8946 02               MOV     DWORD PTR DS:\[ESI+2\],EAX
345013C1    EB 6D                 JMP     SHORT 34501430
345013C3    80F9 06               CMP     CL,6
345013C6    75 4B                 JNZ     SHORT 34501413
345013C8    8A48 0A               MOV     CL,BYTE PTR DS:\[EAX+A\]
345013CB    80F9 02               CMP     CL,2
345013CE    75 1C                 JNZ     SHORT 345013EC
345013D0    8078 09 07            CMP     BYTE PTR DS:\[EAX+9\],7
345013D4    75 0B                 JNZ     SHORT 345013E1
345013D6    C646 FF 76            MOV     BYTE PTR DS:\[ESI-1\],76
345013DA    8A48 04               MOV     CL,BYTE PTR DS:\[EAX+4\]
345013DD    880E                  MOV     BYTE PTR DS:\[ESI\],CL
345013DF    EB 4F                 JMP     SHORT 34501430
345013E1    C646 FF 77            MOV     BYTE PTR DS:\[ESI-1\],77
345013E5    8A50 04               MOV     DL,BYTE PTR DS:\[EAX+4\]
345013E8    8816                  MOV     BYTE PTR DS:\[ESI\],DL
345013EA    EB 44                 JMP     SHORT 34501430
345013EC    80F9 06               CMP     CL,6
345013EF    75 3F                 JNZ     SHORT 34501430
345013F1    8A48 09               MOV     CL,BYTE PTR DS:\[EAX+9\]
345013F4    C646 FF 0F            MOV     BYTE PTR DS:\[ESI-1\],0F
345013F8    80F9 07               CMP     CL,7
345013FB    75 0B                 JNZ     SHORT 34501408
345013FD    C606 86               MOV     BYTE PTR DS:\[ESI\],86
34501400    8B40 04               MOV     EAX,DWORD PTR DS:\[EAX+4\]
34501403    8946 01               MOV     DWORD PTR DS:\[ESI+1\],EAX
34501406    EB 28                 JMP     SHORT 34501430
34501408    C606 87               MOV     BYTE PTR DS:\[ESI\],87
3450140B    8B48 04               MOV     ECX,DWORD PTR DS:\[EAX+4\]
3450140E    894E 01               MOV     DWORD PTR DS:\[ESI+1\],ECX
34501411    EB 1D                 JMP     SHORT 34501430
34501413    80F9 07               CMP     CL,7
34501416    75 18                 JNZ     SHORT 34501430
34501418    8A50 0A               MOV     DL,BYTE PTR DS:\[EAX+A\]
3450141B    33C9                  XOR     ECX,ECX
3450141D    84D2                  TEST    DL,DL
3450141F    76 0F                 JBE     SHORT 34501430
34501421    33D2                  XOR     EDX,EDX
34501423    C6440E FF 90          MOV     BYTE PTR DS:\[ESI+ECX-1\],90
34501428    8A50 0A               MOV     DL,BYTE PTR DS:\[EAX+A\]
3450142B    41                    INC     ECX
3450142C    3BCA                  CMP     ECX,EDX
3450142E  ^ 7C F1                 JL      SHORT 34501421
34501430    46                    INC     ESI
34501431    8D46 FF               LEA     EAX,DWORD PTR DS:\[ESI-1\]
34501434    3D 00405B00           CMP     EAX,5B4000
34501439  ^ 0F82 2AFCFFFF         JB      34501069
3450143F    8B5424 10             MOV     EDX,DWORD PTR SS:\[ESP+10\]
34501443    8D4C24 10             LEA     ECX,DWORD PTR SS:\[ESP+10\]
34501447    51                    PUSH    ECX
34501448    52                    PUSH    EDX
34501449    68 00301B00           PUSH    1B3000
3450144E    68 00104000           PUSH    401000
34501453    FF15 00505034         CALL    DWORD PTR DS:\[34505000\]          ; kernel32.VirtualProtect
34501459    5F                    POP     EDI
3450145A    5E                    POP     ESI
3450145B    5D                    POP     EBP
3450145C    33C0                  XOR     EAX,EAX
3450145E    5B                    POP     EBX
3450145F    83C4 14               ADD     ESP,14
34501462    C3                    RETN
34501463    90                    NOP
34501464    90                    NOP
34501465    90                    NOP
34501466    90                    NOP
34501467    90                    NOP
34501468    90                    NOP
34501469    90                    NOP
3450146A    90                    NOP
3450146B    90                    NOP
3450146C    90                    NOP
3450146D    90                    NOP
3450146E    90                    NOP
3450146F    90                    NOP
34501470    68 B48A0000           PUSH    8AB4
34501475    68 898C6500           PUSH    658C89
3450147A    E8 C1FBFFFF           CALL    34501040
3450147F    83C4 08               ADD     ESP,8
34501482    C3                    RETN
```
