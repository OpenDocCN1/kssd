
# 身份证信息查询与校验(IdCard) v1.01

> 原文：`www.pediy.com/kssd/pediy05/pediy50645.htm`

身份证信息查询与校验(IdCard) v1.01

软件名称：身份证信息查询与校验(IdCard) v1.01
软件语言：简体中文
软件类型：共享软件 / 实用工具 / 其他工具
运行环境：`WinXP`, `Win2000`, `NT`, `WinME`, `Win9X`
授权方式：共享软件
软件大小：747 KB
软件等级：
整理时间：2003-4-22 22:33:00
开发商：
下载地址：`ttdown.com/SoftView_12998.htm`

## 软件简介

该软件主要应用于对身份证号码信息的查询与校验，并且具备升级 15 位旧身份证为 18 位新身份证号码格式的能力。尤其适用于对不明身份证号码持有人的信息查询与校验。

功能：
1. 查询身份证号码持有人的住址，生日，性别等信息。
2. 校验身份证号码，检查身份证号码的真实性。
3. 将 15 位的旧身份证号码升级为 18 位的新身份证号码。

使用方法：输入待查询的身份证号码，即可得到这个号码持有人的住址，生日，性别等信息。并能检验该号码的真实性。

## 【作者声明】

本人只是对 Crack 感兴趣，没有其它目的。

## 【破解工具】

`Ollydbg1.09` 中文版 `TRW2000` V1.23

---

## 【过程】

这个软件防 `Ollydbg` 并且加了壳，我没有脱成功，所以就用 `TRW2000` 的万能断点找注册码计算的入口，然后用 `Ollydbg` 的附加功能跟踪程序。

填注册信息，用户名:`fxyang` 试验码:`7894561230123456`

跟踪来到这里:

```
004A120C  PUSH    EBP
004A120D  PUSH    4A128A
004A1212  PUSH    DWORD PTR FS:[EAX]
004A1215  MOV     DWORD PTR FS:[EAX], ESP
004A1218  LEA     EDX, DWORD PTR SS:[EBP-1C]
004A121B  MOV     EAX, DWORD PTR DS:[EBX+2F8]
004A1221  CALL    004388D8                              ; IDCard.004388D8
004A1226  MOV     EAX, DWORD PTR SS:[EBP-1C]            ; EAX<--00E5A784,(ASCII"7894561230123456")
004A1229  LEA     EDX, DWORD PTR SS:[EBP-18]
004A122C  CALL    0049BBAC                              ; <--检查注册码前部分的正确性
```

\====>F8

\--------检查注册码前部分的正确性---------

|

```
0049BBBB  PUSH    ECX
0049BBBC  PUSH    EBX
0049BBBD  PUSH    ESI
0049BBBE  PUSH    EDI
0049BBBF  MOV     DWORD PTR SS:[EBP-8], E>
0049BBC2  MOV     DWORD PTR SS:[EBP-4], E>; EAX<--00E5A784,(ASCII"7894561230123456")
0049BBC5  MOV     EAX, DWORD PTR SS:[EBP]->; EAX<--00E5A784,(ASCII"7894561230123456")
0049BBC8  CALL    00404950                  ; IDCard.00404950
0049BBCD  XOR     EAX, EAX
0049BBCF  PUSH    EBP
0049BBD0  PUSH    49BD33
0049BBD5  PUSH    DWORD PTR FS:[EAX]
0049BBD8  MOV     DWORD PTR FS:[EAX], ESP
0049BBDB  XOR     EAX, EAX
0049BBDD  PUSH    EBP
0049BBDE  PUSH    49BD06
0049BBE3  PUSH    DWORD PTR FS:[EAX]
0049BBE6  MOV     DWORD PTR FS:[EAX], ESP
0049BBE9  PUSH    6FB2
0049BBEE  LEA     EAX, DWORD PTR SS:[EBP]->
0049BBF1  PUSH    EAX
0049BBF2  LEA     EAX, DWORD PTR SS:[EBP]->
0049BBF5  PUSH    EAX
0049BBF6  MOV     ECX, 9                <---取位的长度(9)
0049BBFB  MOV     EDX, 1
0049BC00  MOV     EAX, DWORD PTR SS:[EBP]->; EAX<--00E5A784,(ASCII"7894561230123456")
0049BC03  CALL    004049C0                  ; <--取试验码的前 9 位
0049BC08  MOV     EAX, DWORD PTR SS:[EBP]->; EAX=00D955C4,(ASCII "789456123")
0049BC0B  LEA     EDX, DWORD PTR SS:[EBP]->
0049BC0E  CALL    0049BAF4                ; <--把上面的串每３个一组变换成１６进制
```

\====>F8

\--------把上面的串每３个一组变换成１６进制---------

|

```
0049BAF4    PUSH    EBP
0049BAF5    MOV     EBP, ESP
0049BAF7    PUSH    0
0049BAF9    PUSH    0
0049BAFB    PUSH    0
0049BAFD    PUSH    EBX
0049BAFE    PUSH    ESI
0049BAFF    MOV     ESI, EDX
0049BB01    MOV     DWORD PTR SS:[EBP-4], EAX        ; EAX=00D955C4,(ASCII "789456123")
0049BB04    MOV     EAX, DWORD PTR SS:[EBP-4]        ; EAX=00D955C4,(ASCII "789456123")
0049BB07    CALL    00404950                              ; IDCard.00404950
0049BB0C    XOR     EAX, EAX
0049BB0E    PUSH    EBP
0049BB0F    PUSH    49BB9C
0049BB14    PUSH    DWORD PTR FS:[EAX]
0049BB17    MOV     DWORD PTR FS:[EAX], ESP
0049BB1A    MOV     EBX, 1
0049BB1F    MOV     EAX, ESI
0049BB21    CALL    004044B0                              ; IDCard.004044B0
0049BB26    MOV     EAX, DWORD PTR SS:[EBP-4]        ; EAX=00D955C4,(ASCII "789456123")
0049BB29    CALL    00404768                              ; IDCard.00404768
0049BB2E    MOV     ECX, 3                              ; ECX=3
0049BB33    CDQ
0049BB34    IDIV    ECX
0049BB36    TEST    EDX, EDX
0049BB38    JNZ     SHORT 0049BB81                ; IDCard.0049BB81
0049BB3A    JMP     SHORT 0049BB75                ; IDCard.0049BB75
0049BB3C    LEA     EAX, DWORD PTR SS:[EBP-C]
0049BB3F    PUSH    EAX
0049BB40    MOV     ECX, 3
0049BB45    MOV     EDX, EBX
0049BB47    MOV     EAX, DWORD PTR SS:[EBP-4]        ; EAX=00D955C4,(ASCII "789456123")
0049BB4A    CALL    004049C0                          ; <--取前三位 ("789")
0049BB4F    MOV     EAX, DWORD PTR SS:[EBP-C]        ; EAX<--00E5A730,(ASCII"789")
0049BB52    CALL    00408A84                          ; <--把"789"变成 16 进制值"315"
```

\=====>F8

\-----把"789"变成 16 进制值"315"------

|

```
004030C2    SUB     BL, 30
004030C5    CMP     BL, 9
004030C8    JA      SHORT 004030EF                ; IDCard.004030EF
004030CA    CMP     EAX, EDI
004030CC    JA      SHORT 004030EF                ; IDCard.004030EF
004030CE    LEA     EAX, DWORD PTR DS:[EAX+EAX*4]
004030D1    ADD     EAX, EAX
004030D3    ADD     EAX, EBX
004030D5    MOV     BL, BYTE PTR DS:[ESI]
004030D7    INC     ESI
004030D8    TEST    BL, BL
004030DA    JNZ     SHORT 004030C2
```

<--这段循环把"789"变成 16 进制值"315"

\-----------------------------------------

继续:

|

```
0049BB57    MOV     EDX, EAX                              ; EDX=EAX=315
0049BB59    LEA     EAX, DWORD PTR SS:[EBP-8]
0049BB5C    CALL    00404690                          ; IDCard.00404690
0049BB61    MOV     EDX, DWORD PTR SS:[EBP-8]
0049BB64    MOV     EAX, ESI
0049BB66    CALL    00404770                          ; IDCard.00404770
0049BB6B    ADD     EBX, 3
0049BB6E    JNO     SHORT 0049BB75                ; IDCard.0049BB75
0049BB70    CALL    00403684                          ; IDCard.00403684
0049BB75    MOV     EAX, DWORD PTR SS:[EBP-4]        ; EAX=00D955C4,(ASCII "789456123")
0049BB78    CALL    00404768                          ; IDCard.00404768
0049BB7D    CMP     EBX, EAX
0049BB7F    JL      SHORT 0049BB3C                ; IDCard.0049BB3C
0049BB81    XOR     EAX, EAX
0049BB83    POP     EDX
0049BB84    POP     ECX
0049BB85    POP     ECX
0049BB86    MOV     DWORD PTR FS:[EAX], EDX
0049BB89    PUSH    49BBA3
0049BB8E    LEA     EAX, DWORD PTR SS:[EBP-C]
0049BB91    MOV     EDX, 3
0049BB96    CALL    004044D4                          ; IDCard.004044D4
0049BB9B    RETN
```

\-----------------------------------------

继续:

|

```
0049BC13  MOV     EAX, DWORD PTR SS:[EBP]->
0049BC16  MOV     ECX, 0C891                ; ECX=0C891
0049BC1B  MOV     EDX, 3D0                ; EDX=3D0
0049BC20  CALL    0049BA54                ; <--用上面的值计算出新值
```

\===>F8

\-------用上面的值计算出新值--------

|

```
0049BA54  PUSH    EBP
0049BA55  MOV     EBP, ESP
0049BA57  ADD     ESP, -0C
0049BA5A  PUSH    EBX
0049BA5B  PUSH    ESI
0049BA5C  PUSH    EDI
0049BA5D  XOR     EBX, EBX
0049BA5F  MOV     DWORD PTR SS:[EBP-C], EBX
0049BA62  MOV     DWORD PTR SS:[EBP-4], ECX
0049BA65  MOV     ESI, EDX                          ; EDX=3D0=ESI
0049BA67  MOV     EDI, EAX
0049BA69  XOR     EAX, EAX
0049BA6B  PUSH    EBP
0049BA6C  PUSH    49BAE2
0049BA71  PUSH    DWORD PTR FS:[EAX]
0049BA74  MOV     DWORD PTR FS:[EAX], ESP
0049BA77  MOV     EAX, DWORD PTR SS:[EBP+8]
0049BA7A  CALL    004044B0                          ; IDCard.004044B0
0049BA7F  MOV     EAX, EDI
0049BA81  CALL    00404768                          ; IDCard.00404768
0049BA86  TEST    AL, AL
0049BA88  JBE     SHORT 0049BACC                ; IDCard.0049BACC
0049BA8A  MOV     BYTE PTR SS:[EBP-5], AL          ; AL=3
0049BA8D  MOV     BL, 1                           ; BL=1
0049BA8F  LEA     EAX, DWORD PTR SS:[EBP-C]
0049BA92  XOR     EDX, EDX
0049BA94  MOV     DL, BL                           ; DL=BL=1
```



```
0049BA96  MOV     DL, BYTE PTR DS:[EDI+EDX-1]      ; DL=DS:[EDI+EDX-1]=15 (315)|=C8 (1C8)|=7B (7B)  
0049BA9A  MOV     ECX, ESI                          ; ECX=ESI=3D0|=030D8C67|=50FA1F51  
0049BA9C  SHR     ECX, 8                            ; ECX=3|=30D8C=50FA1F  
0049BA9F  XOR     DL, CL                            ; DL=15 XOR 03=16|=C8 XOR 8C=44|=7B XOR 1F=64  
0049BAA1  CALL    00404690                          ; IDCard.00404690  
0049BAA6  MOV     EDX, DWORD PTR SS:[EBP-C]         ; EDX=00D9AFD8<--("16")  
0049BAA9  MOV     EAX, DWORD PTR SS:[EBP+8]  
0049BAAC  CALL    00404770                          ; IDCard.00404770  
0049BAB1  MOV     EAX, DWORD PTR SS:[EBP+8]  
0049BAB4  XOR     EAX, EAX  
0049BAB6  MOV     AL, BL                            ; BL=1  
0049BAB8  MOVZX   EAX, BYTE PTR DS:[EDI+EAX-1]     
; EAX=DS:[EDI+EDX-1]=15 (315)|=C8 (1C8)|=7B (7B)
0049BABD  ADD     ESI, EAX                          
; ESI=3D0+15=3E5|=030D8C67+C8=030D8D2F|=50FA1F51+7B=50FA1FCC
0049BABF  IMUL    ESI, DWORD PTR SS:[EBP-4]         
; ESI=3E5*C891=030D1CB5|=030D8D2F*C891=50F9AF9F|=50FA1FCC*C891=4683628C
0049BAC3  ADD     ESI, DWORD PTR SS:[EBP+C]         
; ESI=030D1CB5+6FB2=030D8C67|=50F9AF9F+6FB2=50FA1F51|=4683628C+6FB2=4683D23E
0049BAC6  INC     EBX                                 ; EBX=1++  
0049BAC7  DEC     BYTE PTR SS:[EBP-5]               ; SS:[00`12F847]=03--  
0049BACA  JNZ     SHORT 0049BA8F                    ; IDCard.0049BA8F  
0049BACC  XOR     EAX, EAX  
0049BACE  POP     EDX  
0049BACF  POP     ECX  
0049BAD0  POP     ECX  
0049BAD1  MOV     DWORD PTR FS:[EAX], EDX  
0049BAD4  PUSH    49BAE9  
0049BAD9  LEA     EAX, DWORD PTR SS:[EBP-C]  
0049BADC  CALL    004044B0                          ; IDCard.004044B0  
0049BAE1  RETN 
-----------------------------------------  
```

继续:

```
|  
0049BC25  MOV     EAX, DWORD PTR SS:[EBP-8]         ; EAX=00D9A910,(ASCII "867")  
0049BC28  CALL    00408A84                          ; <--比较计算值的正确性  
====>F8  
------比较计算值的正确性--------  
|  
00403070  PUSH    EBX  
00403071  PUSH    ESI  
00403072  PUSH    EDI  
00403073  MOV     ESI, EAX                           ; EAX<--00E5A730,(ASCII"789")  
00403075  PUSH    EAX  
00403076  TEST    EAX, EAX  
00403078  JE      SHORT 004030E6                    ; IDCard.004030E6  
0040307A  XOR     EAX, EAX  
0040307C  XOR     EBX, EBX  
0040307E  MOV     EDI, 0CCCCCCC  
00403083  MOV     BL, BYTE PTR DS:[ESI]              ; BL=DS:[ESI]=37 ||||=16(1)  
00403085  INC     ESI  
00403086  CMP     BL, 20  
00403089  JE      SHORT 00403083                    ; IDCard.00403083  
0040308B  MOV     CH, 0  
0040308D  CMP     BL, 2D  
00403090  JE      SHORT 004030F4                    ; IDCard.004030F4  
00403092  CMP     BL, 2B  
00403095  JE      SHORT 004030F6                    ; IDCard.004030F6  
00403097  CMP     BL, 24  
0040309A  JE      SHORT 004030FB                    ; IDCard.004030FB  
0040309C  CMP     BL, 78  
0040309F  JE      SHORT 004030FB                    ; IDCard.004030FB  
004030A1  CMP     BL, 58  
004030A4  JE      SHORT 004030FB                    ; IDCard.004030FB  
004030A6  CMP     BL, 30  
004030A9  JNZ     SHORT 004030BE                    ; IDCard.004030BE  
004030AB  MOV     BL, BYTE PTR DS:[ESI]  
004030AD  INC     ESI  
004030AE  CMP     BL, 78  
004030B1  JE      SHORT 004030FB                    ; IDCard.004030FB  
004030B3  CMP     BL, 58  
004030B6  JE      SHORT 004030FB                    ; IDCard.004030FB  
004030B8  TEST    BL, BL  
004030BA  JE      SHORT 004030DC                    ; IDCard.004030DC  
004030BC  JMP     SHORT 004030C2                    ; IDCard.004030C2  
004030BE  TEST    BL, BL  
004030C0  JE      SHORT 004030EF                    \  
004030C2  SUB     BL, 30                            |  
004030C5  CMP     BL, 9                             |<--正确性效验  
004030C8  JA      SHORT 004030EF                    |  
004030CA  CMP     EAX, EDI                          |  
004030CC  JA      SHORT 004030EF                    /
```

说明：效验的方法是比较上面计算的值是不是数字

```
------------------------------------  
|  
0049BC2D  MOV     EBX, EAX            ; EBX=EAX=363  
0049BC2F  PUSH    6FB2  
0049BC34  LEA     EAX, DWORD PTR SS:[EBP-C]  
0049BC37  PUSH    EAX  
0049BC38  LEA     EAX, DWORD PTR SS:[EBP-8]  
0049BC3B  PUSH    EAX  
0049BC3C  MOV     ECX, 0F             ; <---取位的长度(15),改试验码位 30 位继续  
0049BC41  MOV     EDX, 0A  
0049BC46  MOV     EAX, DWORD PTR SS:[EBP-4] ; EAX<--00E49794,(ASCII"315359390147258")  
0049BC49  CALL    004049C0            ; <--取余下的试验码的前 15 位"315359390315359"  
0049BC4E  MOV     EAX, DWORD PTR SS:[EBP-C] ; EAX<--01C19570,(ASCII "315359390315359")  
0049BC51  LEA     EDX, DWORD PTR SS:[EBP-8]  
0049BC54  CALL    0049BAF4            <--把上面的串每３个一组变换成１６进制  
0049BC59  MOV     EAX, DWORD PTR SS:[EBP-4]  
0049BC5C  MOV     ECX, 0C891  
0049BC61  MOV     EDX, 3D0  
0049BC66  CALL    0049BA54            ; <--用上面的值计算出新值  
0049BC6B  MOV     EAX, DWORD PTR SS:[EBP-8] ; EAX=00D9A910,(ASCII "86725")  
0049BC6E  CALL    00408A84            <--比较计算值的正确性

；说明：计算和比较的方法同上

0049BC73  MOV     ESI, EAX  
0049BC75  PUSH    6FB2  
0049BC7A  LEA     EAX, DWORD PTR SS:[EBP-C]  
0049BC7D  PUSH    EAX  
0049BC7E  LEA     EAX, DWORD PTR SS:[EBP-8]  
0049BC81  PUSH    EAX  
0049BC82  MOV     ECX, 0F             ; <---取位的长度(15),改试验码位 40 位继续  
0049BC87  MOV     EDX, 19  
0049BC8C  MOV     EAX, DWORD PTR SS:[EBP-4]  
0049BC8F  CALL    004049C0            ; IDCard.004049C0  
0049BC94  MOV     EAX, DWORD PTR SS:[EBP-C]  
0049BC97  LEA     EDX, DWORD PTR SS:[EBP-8]  
0049BC9A  CALL    0049BAF4            <--把上面的串每３个一组变换成１６进制  
0049BC9F  MOV     EAX, DWORD PTR SS:[EBP-4]  
0049BCA2  MOV     ECX, 0C891  
0049BCA7  MOV     EDX, 3D0  
0049BCAC  CALL    0049BA54            ; <--用上面的值计算出新值  
0049BCB1  MOV     EAX, DWORD PTR SS:[EBP-8] ; EAX=00D9A910,(ASCII "86725")  
0049BCB4  CALL    00408A84            <--比较计算值的正确性

；说明：计算和比较的方法同上

0049BCB9  MOV     EDI, EAX            ; EAX=152C5=EDI  
0049BCBB  PUSH    EDI  
0049BCBC  MOV     EAX, DWORD PTR SS:[EBP-C]  
0049BCBF  PUSH    EAX  
0049BCC0  LEA     EAX, DWORD PTR SS:[EBP-8]  
0049BCC3  PUSH    EAX  
0049BCC4  MOV     EAX, DWORD PTR SS:[EBP-4]

; EAX<--01C13B94  ASCII "315359390315359390438360315359390438360123456789"

0049BCC7  CALL    00404768            ; IDCard.00404768  
0049BCCC  MOV     ECX, EAX            ; ECX=30  
0049BCCE  SUB     ECX, 27  
0049BCD1  JNO     SHORT 0049BCD8      ; IDCard.0049BCD8  
0049BCD3  CALL    00403684            ; IDCard.00403684  
0049BCD8  MOV     EDX, 28  
0049BCDD  MOV     EAX, DWORD PTR SS:[EBP-8]  
0049BCE0  CALL    004049C0            

; <--取用户名的效验位--长度=用户名*3,改试验码位 57 位继续

0049BCE5  MOV     EAX, DWORD PTR SS:[EBP-4]

; EAX<--00D955C4,(ASCII "123456789147258369")<--用户名的效验位

0049BCE8  LEA     EDX, DWORD PTR SS:[EBP-8]  
0049BCEB  CALL    0049BAF4            <--把上面的串每３个一组变换成１６进制  
0049BCF0  MOV     EAX, DWORD PTR SS:[EBP-4]  
0049BCF3  MOV     ECX, ESI  
0049BCF5  MOV     EDX, EBX  
0049BCF7  CALL    0049BA54            ; <--用上面的值计算出新值

说明：计算的方法同上

0049BCFC  XOR     EAX, EAX  
0049BCFE  POP     EDX  
0049BCFF  POP     ECX  
0049BD00  POP     ECX  
0049BD01  MOV     DWORD PTR FS:[EAX], EDX  
0049BD04  JMP     SHORT 0049BD10      ; IDCard.0049BD10  
0049BD06  JMP     00403C24            ; IDCard.00403C24  
0049BD0B  CALL    00403F8C            ; IDCard.00403F8C  
0049BD10  XOR     EAX, EAX  
0049BD12  POP     EDX  
0049BD13  POP     ECX  
0049BD14  POP     ECX  
0049BD15  MOV     DWORD PTR FS:[EAX], EDX  
0049BD18  PUSH    49BD3A  
0049BD1D  LEA     EAX, DWORD PTR SS:[EBP-C]  
0049BD20  MOV     EDX, 0B  
0049BD25  CALL    004044D4            ; IDCard.004044D4  
0049BD2A  LEA     EAX, DWORD PTR SS:[EBP-8]  
```



### 0049BD2D CALL 004044B0

```
0049BD2D  CALL   004044B0           ; IDCard.004044B0  
0049BD32  RETN
```

\-----------------------------------------  

继续:  

|  

```
004A1231  MOV     EAX, DWORD PTR SS:[EBP-18]  
004A1234  PUSH    EAX  
004A1235  LEA     EDX, DWORD PTR SS:[EBP-20]  
004A1238  MOV     EAX, DWORD PTR DS:[EBX+2F4]  
004A123E  CALL    004388D8          
004A1243  MOV     EDX, DWORD PTR SS:[EBP-20]   ; EDX<--01C19570,(ASCII "fxyang")  
004A1246  POP     EAX  
004A1247  CALL    004048AC           ; <--关键的比较  
\====>F8  
```

\------关键的比较-------  

|  

```
004048AC  PUSH    EBX  
004048AD  PUSH    ESI  
004048AE  PUSH    EDI  
004048AF  MOV     ESI, EAX
; ESI<--=0034A078 <---用户名效验位计算的值--参数
004048B1  MOV     EDI, EDX              
; EDX<--01C19570,(ASCII "fxyang")<--参数--用户名
004048B3  CMP     EAX, EDX  
004048B5  JE      0040494A           ; IDCard.0040494A  
004048BB  TEST    ESI, ESI  
004048BD  JE      SHORT 00404927     ; IDCard.00404927  
004048BF  TEST    EDI, EDI  
004048C1  JE      SHORT 0040492E     ; IDCard.0040492E  
004048C3  MOV     EAX, DWORD PTR DS:[ESI-4]  
004048C6  MOV     EDX, DWORD PTR DS:[EDI-4]  
004048C9  SUB     EAX, EDX  
004048CB  JA      SHORT 004048CF     ; IDCard.004048CF  
004048CD  ADD     EDX, EAX  
004048CF  PUSH    EDX  
004048D0  SHR     EDX, 2  
004048D3  JE      SHORT 004048FB     ; IDCard.004048FB  
004048D5  MOV     ECX, DWORD PTR DS:[ESI]  
004048D7  MOV     EBX, DWORD PTR DS:[EDI]  
004048D9  CMP     ECX, EBX  
004048DB  JNZ     SHORT 00404935     ; IDCard.00404935  
004048DD  DEC     EDX  
004048DE  JE      SHORT 004048F5     ; IDCard.004048F5  
004048E0  MOV     ECX, DWORD PTR DS:[ESI+4]  
004048E3  MOV     EBX, DWORD PTR DS:[EDI+4]  
004048E6  CMP     ECX, EBX  
004048E8  JNZ     SHORT 00404935     ; IDCard.00404935  
004048EA  ADD     ESI, 8  
004048ED  ADD     EDI, 8  
004048F0  DEC     EDX  
004048F1  JNZ     SHORT 004048D5     ; IDCard.004048D5  
004048F3  JMP     SHORT 004048FB     ; IDCard.004048FB  
004048F5  ADD     ESI, 4  
004048F8  ADD     EDI, 4  
004048FB  POP     EDX  
004048FC  AND     EDX, 3  
004048FF  JE      SHORT 00404923     ; IDCard.00404923  
00404901  MOV     ECX, DWORD PTR DS:[ESI]    
; ECX<--DS:[ESI]=0034A078 <---用户名效验位计算的值
00404903  MOV     EBX, DWORD PTR DS:[EDI]    
; EBX<--DS:[EDI]=61797866 <---用户名的 hex 值
00404905  CMP     CL, BL          \\  
00404907  JNZ     SHORT 0040494A  |             
00404909  DEC     EDX             |  
0040490A  JE      SHORT 00404923  |<--逐位比较          
0040490C  CMP     CH, BH          |  
0040490E  JNZ     SHORT 0040494A  |              
00404910  DEC     EDX             |  
00404911  JE      SHORT 00404923  /              
00404913  AND     EBX, 0FF0000  
00404919  AND     ECX, 0FF0000  
0040491F  CMP     ECX, EBX  
00404921  JNZ     SHORT 0040494A              
00404923  ADD     EAX, EAX  
00404925  JMP     SHORT 0040494A          
00404927  MOV     EDX, DWORD PTR DS:[EDI-4]  
0040492A  SUB     EAX, EDX  
0040492C  JMP     SHORT 0040494A          
0040492E  MOV     EAX, DWORD PTR DS:[ESI-4]  
00404931  SUB     EAX, EDX  
00404933  JMP     SHORT 0040494A          
00404935  POP     EDX  
00404936  CMP     CL, BL  
00404938  JNZ     SHORT 0040494A          
0040493A  CMP     CH, BH  
0040493C  JNZ     SHORT 0040494A       
0040493E  SHR     ECX, 10  
00404941  SHR     EBX, 10  
00404944  CMP     CL, BL  
00404946  JNZ     SHORT 0040494A       
00404948  CMP     CH, BH  
0040494A  POP     EDI  
0040494B  POP     ESI  
0040494C  POP     EBX  
0040494D  RETN
```

\--------------------------  

继续:  

|  

```
004A124C  JNZ     SHORT 004A1264     ; IDCard.004A1264  
004A124E  MOV     EAX, 4A1358  
004A1253  CALL    00431CFC           ; IDCard.00431CFC  
004A1258  MOV     EAX, DWORD PTR DS:[4AFE14]  
004A125D  CALL    00454E44           ; IDCard.00454E44  
004A1262  JMP     SHORT 004A1280     ; IDCard.004A1280  
004A1264  MOV     ECX, 4A1378  
004A1269  MOV     EDX, 4A1348        ; ASCII "regcode"  
004A126E  MOV     EAX, DWORD PTR SS:[EBP-4]  
004A1271  CALL    004A0A40           ; IDCard.004A0A40  
004A1276  MOV     EAX, 4A1384  
004A127B  CALL    00431CFC           ; IDCard.00431CFC  
004A1280  XOR     EAX, EAX  
004A1282  POP     EDX  
004A1283  POP     ECX  
004A1284  POP     ECX  
004A1285  MOV     DWORD PTR FS:[EAX], EDX  
004A1288  JMP     SHORT 004A12B0     ; IDCard.004A12B0  
004A128A  JMP     00403C24           ; IDCard.00403C24  
004A128F  MOV     ECX, 4A1378  
004A1294  MOV     EDX, 4A1348        ; ASCII "regcode"  
004A1299  MOV     EAX, DWORD PTR SS:[EBP-4]  
004A129C  CALL    004A0A40           ; IDCard.004A0A40  
004A12A1  MOV     EAX, 4A1384  
004A12A6  CALL    00431CFC           ; IDCard.00431CFC  
004A12AB  CALL    00403F8C           ; IDCard.00403F8C  
004A12B0  XOR     EAX, EAX  
004A12B2  POP     EDX  
004A12B3  POP     ECX  
004A12B4  POP     ECX  
004A12B5  MOV     DWORD PTR FS:[EAX], EDX  
004A12B8  PUSH    4A12D5  
004A12BD  MOV     EAX, DWORD PTR SS:[EBP-4]  
004A12C0  CALL    004A0734           ; IDCard.004A0734  
004A12C5  MOV     EAX, DWORD PTR SS:[EBP-4]  
004A12C8  CALL    00403744           ; IDCard.00403744  
004A12CD  RETN
```

\===============================================================================

到这里算法跟踪分析已完成,总结一下.

1.  **条件**：注册码的长度=`39`位+`用户名位数*3`
2.  **注册码的计算方法**：
    1.  分三次取试验码的前 `39` 位(这部分与用户名无关),效验它的正确性。即通过计算的值是数字就正确.所以只有跟踪到一个正确值就能用于任何一个用户名.
    2.  后面的`用户名位数*3`位才是注册码与用户名的效验位.下面来说说计算过程
        1.  取试验码的效验位,然后 `3` 个一组变换成 `16` 进制值
        2.  取每组 `16` 进制值的低字节进入下步计算(所以用户名的效验码不止一个)
        3.  第一组值 `XOR 03` =`用户名第一位的 hex 值`
        4.  第一组值+`3D0`(固定值)的值\*`C89`(固定值),得到一个新值+`6FB2`(固定值)
        5.  用上面值的第三字节 `XOR` 第二组值=`用户名的第二位`
        6.  用第四步计算的值+第二组的值然后再次进行第四步第五步计算，直到计算完

by `fxyang[OCN][BCG]`
`2003.4.24` `4dd466d0e1e0cc123afbe504da2462ca`
