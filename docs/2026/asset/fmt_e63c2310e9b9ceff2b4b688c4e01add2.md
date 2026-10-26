
```asm
00AF5078    5A              POP     EDX                              ; Christma.00400000

00AF5079    81FB 54EBFFFF   CMP     EBX,-14AC

00AF507F  ^ 0F85 00FFFFFF   JNZ     00AF4F85

//注意这里是跳回去循环的!再下面设断点!之后按断点!

00AF5085    E9 0E000000     JMP     00AF5098

//在这里设断!断下后清除之再按F7

===============================================

00AF5098   /0F8F 04000000   JG      00AF50A2

//来到这里!

//下面是一下新型的花指令,很有意思,要有耐心慢慢跟!

00AF509E   |66:B9 2579      MOV     CX,7925

00AF50A2   66:B8 A182      MOV     AX,82A1

00AF50A6    5B              POP     EBX                              ; Christma.00400000

00AF50A7    58              POP     EAX                              ; Christma.00400000

00AF50A8    05 1E9EDFB8     ADD     EAX,B8DF9E1E

00AF50AD    5C              POP     ESP                              ; Christma.00400000

00AF50AE    EB 44           JMP     SHORT 00AF50F4

00AF50B0    EB 01           JMP     SHORT 00AF50B3

00AF50B2    9A 51579CFC BF0>CALL    FAR 00BF:FC9C5751                ; 远距呼叫

00AF50B9    0000            ADD     [EAX],AL

00AF50BB    00B9 00000000   ADD     [ECX],BH

00AF50C1    F3:AA           REP     STOS BYTE PTR ES:[EDI]

00AF50C3    9D              POPFD

00AF50C4    5F              POP     EDI                              ; Christma.00400000

00AF50C5    59              POP     ECX                              ; Christma.00400000

00AF50C6    C3              RETN

00AF50C7    55              PUSH    EBP

00AF50C8    8BEC            MOV     EBP,ESP

00AF50CA    53              PUSH    EBX

00AF50CB    56              PUSH    ESI

00AF50CC    8B75 0C         MOV     ESI,[EBP+C]

00AF50CF    8B5D 08         MOV     EBX,[EBP+8]                      ; Christma.004703C2

00AF50D2    EB 11           JMP     SHORT 00AF50E5

00AF50D4    0FB703          MOVZX   EAX,WORD PTR [EBX]

00AF50D7    03C6            ADD     EAX,ESI

00AF50D9    83C3 02         ADD     EBX,2

00AF50DC    8BD0            MOV     EDX,EAX

00AF50DE    8BC6            MOV     EAX,ESI

00AF50E0    E8 0C000000     CALL    00AF50F1

00AF50E5    66:833B 00      CMP     WORD PTR [EBX],0

00AF50E9  ^ 75 E9           JNZ     SHORT 00AF50D4

00AF50EB    5E              POP     ESI                              ; Christma.00400000

00AF50EC    5B              POP     EBX                              ; Christma.00400000

00AF50ED    5D              POP     EBP                              ; Christma.00400000

00AF50EE    C2 0800         RETN    8

00AF50F1    0102            ADD     [EDX],EAX

00AF50F3    C3              RETN

00AF50F4    03C3            ADD     EAX,EBX

00AF50F6    BB 36030000     MOV     EBX,336

00AF50FB    0BDB            OR      EBX,EBX

00AF50FD    75 07           JNZ     SHORT 00AF5106

00AF50FF    894424 1C       MOV     [ESP+1C],EAX

00AF5103    61              POPAD

00AF5104    50              PUSH    EAX

00AF5105    C3              RETN

00AF5106    E8 00000000     CALL    00AF510B

00AF510B    5D              POP     EBP                              ; Christma.00400000

00AF510C    81ED 4DE14B00   SUB     EBP,4BE14D

00AF5112    8D85 F2E04B00   LEA     EAX,[EBP+4BE0F2]

00AF5118    8D8D 94E14B00   LEA     ECX,[EBP+4BE194]

00AF511E    03CB            ADD     ECX,EBX

00AF5120    8941 01         MOV     [ECX+1],EAX

00AF5123    8D85 36E14B00   LEA     EAX,[EBP+4BE136]

00AF5129    8D8D FAE04B00   LEA     ECX,[EBP+4BE0FA]

00AF512F    8901            MOV     [ECX],EAX

00AF5131    B8 5E140000     MOV     EAX,145E

00AF5136    8D8D FFE04B00   LEA     ECX,[EBP+4BE0FF]

00AF513C    8901            MOV     [ECX],EAX

00AF513E    8D8D 94E14B00   LEA     ECX,[EBP+4BE194]

00AF5144    8D85 94F34B00   LEA     EAX,[EBP+4BF394]

00AF514A    51              PUSH    ECX

00AF514B    50              PUSH    EAX

00AF514C    E8 76FFFFFF     CALL    00AF50C7

00AF5151    61              POPAD

00AF5152    F2:             PREFIX REPNE:                            ; 多余的前缀

//看见这里没有!嘿嘿!有你受的!

00AF5153    EB 01           JMP     SHORT 00AF5156

00AF5155    9A F3EB02CD 205>CALL    FAR 5320:CD02EBF3                ; 远距呼叫

00AF515C    36:EB 02        JMP     SHORT 00AF5161                   ; 多余的前缀

00AF515F    CD20 0BDFEB02   VxDCall 2EBDF0B

00AF5165    CD20 C1C308EB   VxDCall EB08C3C1

00AF516B    02CD            ADD     CL,CH

00AF516D    20F2            AND     DL,DH

00AF516F    EB 01           JMP     SHORT 00AF5172

00AF5171    698D 1CADFD4D F>IMUL    ECX,[EBP+4DFDAD1C],EBF2B1F4

00AF517B    010F            ADD     [EDI],ECX

00AF517D    65:EB 02        JMP     SHORT 00AF5182                   ; 多余的前缀

00AF5180    CD20 55F3EB02   VxDCall 2EBF355

00AF5186    CD20 C7442400   VxDCall 2444C7

00AF518C    2C 00           SUB     AL,0

00AF518E    0000            ADD     [EAX],AL

00AF5190    5B              POP     EBX                              ; Christma.00400000

00AF5191    FF53 1B         CALL    [EBX+1B]

00AF5194    699A 0F9A0F4E 0>IMUL    EBX,[EDX+4E0F9A0F],E8000000

00AF519E    F3:             PREFIX REP:                              ; 多余的前缀

00AF519F    9A 5B26EB01 9A8>CALL    FAR 819A:01EB265B                ; 远距呼叫

00AF51A6    D3EF            SHR     EDI,CL

00AF51A8    FA              CLI

00AF51A9    96              XCHG    EAX,ESI

00AF51AA    42              INC     EDX

00AF51AB    F2:             PREFIX REPNE:                            ; 多余的前缀

00AF51AC    EB 01           JMP     SHORT 00AF51AF

00AF51AE    C7              ???                                      ; 未知命令

00AF51AF    F3:             PREFIX REP:                              ; 多余的前缀

00AF51B0    EB 02           JMP     SHORT 00AF51B4

00AF51B2    CD20 0FA3CBF2   VxDCall F2CBA30F

00AF51B8    EB 01           JMP     SHORT 00AF51BB

00AF51BA    E8 2BDF5B6A     CALL    6B0B30EA

00AF51BF    60              PUSHAD

00AF51C0    68 A0444300     PUSH    4344A0

00AF51C5    F3:             PREFIX REP:                              ; 多余的前缀

00AF51C6    EB 02           JMP     SHORT 00AF51CA

00AF51C8    CD20 66812D82   VxDCall 822D8166

00AF51CE    0000            ADD     [EAX],AL

00AF51D0    0085 47F3704A   ADD     [EBP+4A70F347],AL

00AF51D6    CD20 53EB087F   VxDCall 7F08EB53

00AF51DC  ^ 7D F4           JGE     SHORT 00AF51D2

00AF51DE    FC              CLD

00AF51DF    7F 7D           JG      SHORT 00AF525E

00AF51E1    F4              HLT                                      ; 特权级命令

00AF51E2    FC              CLD

00AF51E3    F3:             PREFIX REP:                              ; 多余的前缀

00AF51E4    EB 02           JMP     SHORT 00AF51E8

00AF51E6    CD20 8D9E8900   VxDCall 899E8D

00AF51EC    0000            ADD     [EAX],AL

00AF51EE    2BDE            SUB     EBX,ESI

00AF51F0    F2:             PREFIX REPNE:                            ; 多余的前缀

00AF51F1    EB 01           JMP     SHORT 00AF51F4

00AF51F3    F0:83EC 08      LOCK SUB ESP,8                           ; 锁定前缀是不允许的

00AF51F7    EB 02           JMP     SHORT 00AF51FB

00AF51F9    CD20 8D640404   VxDCall 404648D

00AF51FF    2BE0            SUB     ESP,EAX

00AF5201    F3:             PREFIX REP:                              ; 多余的前缀

00AF5202    EB 02           JMP     SHORT 00AF5206

00AF5204    CD20 50EB02CD   VxDCall CD02EB50

00AF520A    208F 442400F3   AND     [EDI+F3002444],CL

00AF5210    EB 02           JMP     SHORT 00AF5214

00AF5212    CD20 8B430026   VxDCall 2600438B

00AF5218    EB 01           JMP     SHORT 00AF521B

00AF521A    0F57EB          XORPS   XMM5,XMM3

00AF521D    02CD            ADD     CL,CH

00AF521F    208D 6424F583   AND     [EBP+83F52464],CL

00AF5225    C458 8D         LES     EBX,[EAX-73]                     ; 修正的段位寄存器

00AF5228    64:24 AF        AND     AL,0AF                           ; 多余的前缀

00AF522B    EB 01           JMP     SHORT 00AF522E

00AF522D    9A 53F3EB02 CD2>CALL    FAR 20CD:02EBF353                ; 远距呼叫

00AF5234    8F4424 00       POP     DWORD PTR [ESP]                  ; Christma.00400000

00AF5238    F3:             PREFIX REP:                              ; 多余的前缀
```
