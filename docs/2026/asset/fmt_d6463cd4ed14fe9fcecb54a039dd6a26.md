
CALL    `<JMP.&Vcl50.@System@@LStrA>`

```
0040439C  |.  33C0
         XOR     EAX, EAX
0040439E  |.
 55            PUSH    EBP
0040439F
 |.  68 DF434000   PUSH    SnapYa!.004043DF
004043A4
 |.  64:FF30       PUSH    DWORD PTR FS:[EAX]
004043A7
 |.  64:8920       MOV     DWORD PTR FS:[EAX],
ESP
004043AA  |.  33C9          XOR
  ECX, ECX
004043AC  |.  B8 01000000   MOV
EAX, 1           ;  EAX<==1 指针赋初值
004043B1
 |>  8B55 FC       /MOV     EDX, [LOCAL.1]
;  EAX <==SS:[12F268]=011909BC,(ASCII
"789456123012345678")
004043B4  |.  8A5402 FF
|MOV     DL, BYTE PTR DS:[EDX+EAX->
;  DL <== DS:[011909BC]=37
('7')
004043B8  |.  81E2 FF000000 |AND     EDX, 0FF
004043BE
 |.  66:03CA       |ADD     CX, DX
    ;  CX=CX+DX=0+37=37
004043C1  |.  40
         |INC     EAX
                ;  EAX++
004043C2
 |.  83F8 0E       |CMP     EAX, 0E
       ;  计算的次数=14-1=13
004043C5  |.^ 75
EA         \JNZ     SHORT SnapYa!.004043B1
;  //这段把试验码的前13位的hex值相加==>ECX
004043C7  |.  8BD9
         MOV     EBX, ECX
            ;  EBX<==ECX=2A3
004043C9
 |.  33C0          XOR     EAX, EAX
004043CB
 |.  5A            POP     EDX
004043CC
 |.  59            POP     ECX
004043CD
 |.  59            POP     ECX
004043CE
 |.  64:8910       MOV     DWORD PTR FS:[EAX],
EDX
004043D1  |.  68 E6434000   PUSH    SnapYa!.004043E6
004043D6
 |>  8D45 FC       LEA     EAX, [LOCAL.1]
;  (ASCII "7894561230123456")
的地址
004043D9  |.  E8 0ACDFFFF   CALL    <JMP.&Vcl50.@System@@LStrC>
004043DE
 \.  C3            RETN
```

-----------------------------------------------------------------------------

第四,五位如果不是77也不应该是:

```
40003532
   8BC0            MOV     EAX,
EAX
40003534 >  53              PUSH
   EBX
40003535    56
   PUSH    ESI
40003536    57
       PUSH    EDI
40003537    89C6
           MOV     ESI, EAX
40003539
   50              PUSH    EAX
4000353A
   85C0            TEST    EAX,
EAX
4000353C    74 73           JE
   SHORT Vcl50.400035B1
4000353E    31C0
       XOR     EAX, EAX
40003540    31DB
           XOR     EBX, EBX
40003542
   BF CCCCCC0C     MOV     EDI, 0CCCCCCC
40003547
   8A1E            MOV     BL,
BYTE PTR DS:[ESI]
 ; BL <== DS:[0117AC14]=34 ('4')
40003549
 46              INC     ESI
4000354A
   80FB 20         CMP     BL, 20
4000354D
 ^ 74 F8           JE      SHORT
Vcl50.40003547
4000354F    B5 00
MOV     CH, 0
; 第四位的条件:
40003551    80FB 2D         CMP
    BL, 2D                ;
2D ('-')
40003554    74 69           JE
     SHORT Vcl50.400035BF
40003556    80FB 2B
      CMP     BL, 2B
40003559    74 66
        JE      SHORT Vcl50.400035C1
4000355B
   80FB 24         CMP     BL, 24
4000355E
   74 66           JE      SHORT
Vcl50.400035C6
40003560    80FB 78         CMP
    BL, 78                 ;
78 ('x')
40003563    74 61           JE
     SHORT Vcl50.400035C6
40003565    80FB 58
      CMP     BL, 58
      ; 58 ('X')
40003568    74 5C
      JE      SHORT Vcl50.400035C6
4000356A
 80FB 30         CMP     BL, 30
; <==如果是0就不跳,0就正确
4000356D
   75 13           JNZ     SHORT
Vcl50.40003582
4000356F    8A1E
 MOV     BL, BYTE PTR DS:[ESI]  ; BL =35 ('5')第五位
40003571
   46              INC
ESI
40003572    80FB 78         CMP
BL, 78                 ; 78 ('x')
40003575
   74 4F           JE      SHORT
Vcl50.400035C6
40003577    80FB 58         CMP
    BL, 58                 ;
58 ('X')
4000357A    74 4A           JE
     SHORT Vcl50.400035C6
4000357C    84DB
         TEST    BL, BL
4000357E
 74 20           JE      SHORT Vcl50.400035A0
40003580
   EB 04           JMP     SHORT
Vcl50.40003586
40003582    84DB
 TEST    BL, BL
40003584    74 34
    JE      SHORT Vcl50.400035BA
40003586    80EB
30         SUB     BL, 30
40003589    80FB
09         CMP     BL, 9
4000358C    77
2C           JA      SHORT Vcl50.400035BA
4000358E
   39F8            CMP     EAX,
EDI
40003590    77 28           JA
   SHORT Vcl50.400035BA
40003592    8D0480
     LEA     EAX, DWORD PTR DS:[EAX+EAX\*4]
40003595
   01C0            ADD     EAX,
EAX
40003597    01D8            ADD
    EAX, EBX
40003599    8A1E
   MOV     BL, BYTE PTR DS:[ESI]
; BL <== DS:[0117AC15]=35
('5')
4000359B    46              INC
    ESI
4000359C    84DB
 TEST    BL, BL
4000359E  ^ 75 E6
    JNZ     SHORT Vcl50.40003586
400035A0    FECD
           DEC     CH
400035A2
 74 10           JE      SHORT Vcl50.400035B4
400035A4
   85C0            TEST    EAX,
EAX              ; EAX=2D ('-')
400035A6
   7C 12           JL      SHORT
Vcl50.400035BA
400035A8    59
   POP     ECX
400035A9    31F6
       XOR     ESI, ESI
400035AB    8932
           MOV     DWORD PTR DS:[EDX],
ESI
400035AD    5F              POP
    EDI
400035AE    5E
   POP     ESI
400035AF    5B
       POP     EBX
400035B0    C3
           RETN
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

注册码计算的总结:

条件注册码的长度=18位

1. 注册码的前三位是: SNY
2. 注册码的第六位是: A
3. 注册码的第四五位有如下几种限制情况:
   1). 如果是: 77  <== 就是完全版
   2). 不能是: X 和: x
   3). 如果是: 00 <== 就是30天限制版
4. 注册码的第七到第十四位随便
5. 注册码的最后四位的计算方法:
   1). 把前13位的各位hex相加
   2). 如果不足四位前面加0

一组可以用的注册码: SNY77A789456120321 <== 完全版

SNY00A789456120313  <== 30天限制版

fxyang更正于 2003.3.8-12:00

fxyang

2003.3.7
