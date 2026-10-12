
```asm
0190    SHR     SI,1             ;junk  
0192    MOV     DX,CS            ;*  Get CODE SEG in DX  
0194    CALL    0171             ;junk  
0197    TEST    SI,1410          ;junk  
019B    CLC                      ;junk  
019C    SHL     DI,CL            ;junk  
019E    MOV     DS,DX            ;*  Make DATA SEG = CODE SEG  
01A0    NEG     SI               ;junk  
01A2    CALL    0171             ;junk  
01A5    TEST    CH,[BP+DI+070F]  ;junk  
01A9    MOV     DL,8D            ;*  Set decrypt KEY register  
01AB    MOV     DI,3A30          ;junk  
01AF    JMP     01B9             ;junk  
01B2    JBE     01B5             ;junk  
01B4    INC     DI               ;junk  
01B5    NOT     DI               ;junk  
01B7    CMC                      ;junk  
01B8    RET                      ;junk  
01B9    XOR     CX,DX            ;junk  
01BB    CALL    01B2             ;junk * Decryptor MAIN LOOP  
01BE    TEST    SI,3029          ;junk  
01C2    INC     DI               ;junk  
01C3    SBB     DI,1E19          ;junk  
01C7    MOV     DI,0038          ;junk  
01CB    RCR     DI,CL            ;junk  
01CD    MOV     BP,1809          ;junk  
01D1    NEG     BYTE PTR [BX+4969]       ;*  NEG the byte at [BX + 4969]  
                                   ;   NOTE:  original index B977h plus  
                                   ;   4969h = 102E0h AND FFFFh = 02E0h!  
                                   ;   Which is the start of the  
                                   ;   encrypted code!  
01D5    TEST    BP,2A37          ;junk  
01D9    CMP     CX,2B37          ;junk  
01DD    JMP     01E2             ;junk  
01E0    DEC     DI               ;junk  
01E1    RET                      ;junk  
01E2    MOV     CL,[BX+4969]     ;*  Get the NEGed byte into CL  
01E6    CMC                      ;junk  
01E7    ROR     DI,CL            ;junk  
01E9    INC     BP               ;junk  
01EA    TEST    DI,281E          ;junk  
01EE    JZ      01F3             ;junk  
01F0    TEST    BH,[BX+DI+05]    ;junk  
01F3    MOV     DI,160C          ;junk  
01F7    SUB     BP,BP            ;junk  
01F9    XOR     CX,DX            ;*  XOR byte with the KEY  
01FB    TEST    BL,[BP+DI+3C]    ;junk  
01FE    JNB     0204             ;junk  
0200    ADD     BP,0A13          ;junk  
0204    CMP     [BX+DI],CL       ;junk  
0206    CALL    01E0             ;junk  
0209    CALL    01E0             ;junk  
020C    DEC     DI               ;junk  
020D    AND     DI,073A          ;junk  
0211    DEC     AX               ;*  Decrease the COUNT register  
0212    XOR     DI,2036          ;junk  
0216    NEG     BP               ;junk  
0218    ADC     DI,SP            ;junk  
021A    CMC                      ;junk  
021B    CMP     BL,[BX+SI]       ;junk  
021D    DEC     DX               ;*  Decrease the KEY register  
021E    ADC     BP,1821          ;junk  
0222    SHL     DI,CL            ;junk  
0224    CMP     AX,1816          ;junk  
0228    SHL     DI,1             ;junk  
022A    CMP     AL,[BP+DI+1A]    ;junk  
022D    MOV     SI,1819          ;junk  
0231    ADD     SI,063B          ;junk  
0235    DEC     DX               ;*  Decrease the KEY register  
0236    SUB     BP,0028          ;junk  
023A    AND     BP,1930          ;junk  
023E    CLD                      ;junk  
023F    ADC     BP,2D1D          ;junk  
0243    SAR     DI,CL            ;junk  
0245    XCHG    CX,DX            ;*  Swap CX & DX  
0247    TEST    CX,DX            ;junk  
0249    MOV     SI,CX            ;junk  
024B    XOR     SI,030D          ;junk  
024F    SUB     DI,311C          ;junk  
0253    XCHG    DL,[BX+4969]     ;*  Swap [index] & DL  
                                   ;   NOTE:  This restores the decrypted byte!  
0257    ADD     DI,0E13          ;junk  
025B    CMP     BL,[BP+DI+33]    ;junk  
025E    CLD                      ;junk  
025F    NOT     SI               ;junk  
0261    MOV     SI,3F1C          ;junk  
0265    XCHG    CX,DX            ;*  Swap CX & DX, restoring the KEY in DL  
0267    MOV     SI,221A          ;junk  
026B    OR      BP,0D2C          ;junk  
026F    MOV     DI,231B          ;junk  
0273    ADD     BX,0001          ;*  Increase the INDEX register  
0277    JMP     0288             ;junk  
027A    ADC     BP,AX            ;junk  
027C    TEST    BL,[DI+19]       ;junk  
027F    TEST    DI,0321          ;junk  
0283    NEG     DI               ;junk  
0285    ROL     SI,CL            ;junk  
0287    RET                      ;junk  
0288    SBB     BP,1B0D          ;junk  
028C    XOR     BP,2A23          ;junk  
0290    CMP     DL,3A            ;junk  
0293    TEST    BH,[DI]          ;junk  
0295    AND     AX,AX            ;*  Test if COUNT is zero  
0297    JNZ     02AD             ;*  Jump to 02ADh if not  
0299    CALL    027A             ;junk  
029C    AND     DI,291F          ;junk  
02A0    JA      02A6             ;junk  
02A2    MOV     DI,0514          ;junk  
02A6    ADC     SI,1F2A          ;junk  
02AA    JMP     02BC             ;*  Finished decrypting  
02AD    JMP     02B2             ;junk  
02B0    CLC                      ;junk  
02B1    RET                      ;junk  
02B2    SHL     DI,CL            ;junk  
02B4    CLD                      ;junk  
02B5    ADD     SI,2C1A          ;junk  
02B9    JMP     01BB             ;*  Jump to MAIN LOOP  
02BC    TEST    BH,BL            ;junk  
02BE    MOV     DI,210C          ;junk  
02C2    SUB     SI,1600          ;junk  
02C6    CALL    02B0             ;junk  
02C9    XOR     SI,2F1D          ;junk  
02CD    MOV     BP,0430          ;junk  
02D1    TEST    BH,[DI+362A]     ;junk  
02D5    OR      DI,1C21          ;junk  
02D9    STC                      ;junk  
02DA    CMP     DI,2828          ;junk  
02DE    CLC                      ;junk  
02DF    DEC     BP               ;junk  
02E0    The encrypted code starts here.  
```

`************************** 译码器 2 汇编列表到此结束 **************************`

下面是上述两个译码器的 `HEX` 转储，译码器 1 在左边，译码器 2 在右边。这些转储显示很难从它们当中找出相同的特征，这就是多态变形引擎最主要的目的!因此，为了检测，你必须写一个程序，尝试使用智能的方法找出看起来像多态变形引擎译码器的东西。这很容易产生误报，或者在某些情况下，完全遗漏了译码器！

`HEX DUMP OF ENCRYPTOR 1, 368 bytes`        `HEX DUMP OF ENCRYPTOR 2, 480 bytes`

`―――――――――――――――――`        `―――――――――――――――――`

```
7901FCD3FE81FD0807F97F020BF181F7            F7D6846B0F47FC81D72A137A0481CF2E
213281C5050881E51235D3EEC7C6041B            334684C5E909007A0481FA0919D3DFC3
D3FF81D6062581D7111F1B2E3E0F81FD            4785FDE90D00F7C7240EF7C73A0923FC
1E3F4EF7D781E63D084681DF0301BA91            383AC3C7C5280C84F5F7C5161CD3CD74
17FC720285F01BFC853E1B25840CD1E5            04843703FC848C3534C7C5082E85CFFC
BB7D01F5C7C718127002D1DFF939FD8C            C7C6313881E53E36D3CFF87906D1FE81
C8846F1781DD073147D1DD8ED8033E04            DF083381DF2B36B8DB0781E71E0FE911
3BB0507304C7C63914720213F8E90500            00C7C7312F81F9122281DE142E849C1D
C7C5361BC3D1DEBB2D842BF181CF0F0B            34C380FF19BB77B984852C07F7C70623
C7C53E1ED3D781ED122E033E2A2ED3C6            D1EE8CCAE8DAFFF7C61014F8D3E78EDA
8A8F437E740684BD3B2B3812D1C781DF            F7DEE8CCFF84AB0F07B28DC7C7303AE9
3A264AE8CAFF8BF981D52E282AC8C7C6            0700760147F7D7F5C333CAE8F4FFF7C6
2A37F7C5103AE8B7FF81D61713FC4081            29304781DF191EC7C73800D3DFC7C509
F63D20E90B004FF52B2E243681F60002            18F69F6949F7C5372A81F9372BE90200
C3387C1381EA010081F8170581ED1628            4FC38A8F6949F5D3CF45F7C71E287403
81E6070881EE032ED1CD47D3DE84EE81            847905C7C70C162BED33CA845B3C7304
ED2610888F437E730481F7301BFC81C6            81C5130A3809E8D7FFE8D4FF4F81E73A
383C4381F72C0BE9150081CD181C7E01            074881F73620F7DD13FCF53A184A81D5
4D81D6320E81E72215387A36D1C6C3D1            2118D3E781F81618D1E73A431AC7C619
E7D1EF4A7509840B81D52D01E91C0045            1881C63B064A81ED280081E53019FC81
730481FB2E0E85FE81DE3332B98A01D1            D51D2DD3FF87CA85CA8BF181F60D0381
```
