
```
0046AEE8  |. 76 0A          |JBE SHORT UNPACKED.0046AEF4

0046AEEA  |. 3C 3A          |CMP AL,3A

0046AEEC  |. 73 06          |JNB SHORT UNPACKED.0046AEF4

0046AEEE  |. 8043 02 04     |ADD BYTE PTR DS:[EBX+2],4

0046AEF2  |. EB 0E          |JMP SHORT UNPACKED.0046AF02

0046AEF4  |> 3C 2B          |CMP AL,2B

0046AEF6  |. 75 06          |JNZ SHORT UNPACKED.0046AEFE

0046AEF8  |. C643 02 3E     |MOV BYTE PTR DS:[EBX+2],3E

0046AEFC  |. EB 04          |JMP SHORT UNPACKED.0046AF02

0046AEFE  |> C643 02 3F     |MOV BYTE PTR DS:[EBX+2],3F

0046AF02  |> 8A43 03        |MOV AL,BYTE PTR DS:[EBX+3]

  ;①0046AED0跳来，取第四个加密密码,[EBX+3]=a(61)

             ;②0046AEE1跳来，取第四个密码，[EBX+3]=+(2B)

0046AF05  |. 3C 40          |CMP AL,40

                 ;①61>40,不跳

0046AF07  |. 76 0A          |JBE SHORT UNPACKED.0046AF13

 ;②2B<40,跳0046AF13---->

0046AF09  |. 3C 5B          |CMP AL,5B

0046AF0B  |. 73 06          |JNB SHORT UNPACKED.0046AF13

 ;①61>5B,跳0046AF13-->

0046AF0D  |. 806B 03 41     |SUB BYTE PTR DS:[EBX+3],41

0046AF11  |. EB 30          |JMP SHORT UNPACKED.0046AF43

0046AF13  |> 8A43 03        |MOV AL,BYTE PTR DS:[EBX+3]

  ;①0046AF0B跳来②0046AF07跳来

0046AF16  |. 3C 60          |CMP AL,60

0046AF18  |. 76 0A          |JBE SHORT UNPACKED.0046AF24

 ;①61>60,不跳

             ;②2B<60,跳0046AF24---->

0046AF1A  |. 3C 7B          |CMP AL,7B

0046AF1C  |. 73 06          |JNB SHORT UNPACKED.0046AF24

 ;①61<7B, 不跳

0046AF1E  |. 806B 03 47     |SUB BYTE PTR DS:[EBX+3],47

;①[EBX+3]=61-47=1A

             ;①[EBX]=04 25 0E 1A

0046AF22  |. EB 1F          |JMP SHORT UNPACKED.0046AF43

 ;①跳0046AF43-->

0046AF24  |> 8A43 03        |MOV AL,BYTE PTR DS:[EBX+3]

  ;②[EBX+3]=2B入AL

0046AF27  |. 3C 2F          |CMP AL,2F

0046AF29  |. 76 0A          |JBE SHORT UNPACKED.0046AF35

 ;②2B<2F,跳0046AF35---->

0046AF2B  |. 3C 3A          |CMP AL,3A

0046AF2D  |. 73 06          |JNB SHORT UNPACKED.0046AF35

0046AF2F  |. 8043 03 04     |ADD BYTE PTR DS:[EBX+3],4

0046AF33  |. EB 0E          |JMP SHORT UNPACKED.0046AF43

0046AF35  |> 3C 2B          |CMP AL,2B

                 ;②0046AF29跳来。

0046AF37  |. 75 06          |JNZ SHORT UNPACKED.0046AF3F

 ;②AL=2b，不跳。

0046AF39  |. C643 03 3E     |MOV BYTE PTR DS:[EBX+3],3E

;②将3E入[EBX+3]

0046AF3D  |. EB 04          |JMP SHORT UNPACKED.0046AF43

 ;②跳0046AF43---->

0046AF3F  |> C643 03 3F     |MOV BYTE PTR DS:[EBX+3],3F

0046AF43  |> 8A13           |MOV DL,BYTE
PTR DS:[EBX]     ;①从0046AF22来到这里,将[EBX]=4，入DL,DL=4

             ;②从0046AF3D来到这里,将[EBX]=5，入DL,DL=5

0046AF45  |. C1E2 02        |SHL EDX,2

               ;①EDX=（EBX+4=0076Fd04

）shl 2=1DBF410

             ;②EDX=（EBX+4=0076Fd05

）shl 2=1DBF414

0046AF48  |. 8A43 01        |MOV AL,BYTE PTR DS:[EBX+1]

  ;①[EBX+1]入AL,[EBX+1]=25

             ;②[EBX+1]入AL,[EBX+1]=14

0046AF4B  |. 33C9           |XOR ECX,ECX

               ;①;②ECX清零,

0046AF4D  |. 8AC8           |MOV CL,AL

                 ;①CL=AL=25

             ;②CL=AL=14

0046AF4F  |. C1E9 04        |SHR ECX,4

               ;①ECX=25 shr 4

=2

             ;②ECX=14 shr 4 =1

0046AF52  |. 80E1 03        |AND CL,3

                ;①2 and 3=2

             ;②1&3=1

0046AF55  |. 0AD1           |OR DL,CL

                  ;①DL为EDX的低位=10,DL=10

or 2= 12

             ;②DL=14,DL=DL OR CL=14

0R 1 = 15

0046AF57  |. 8855 F1        |MOV BYTE PTR SS:[EBP-F],DL

  ;①将12入[EBP-F]

             ;②将1５入[EBP-F]

0046AF5A  |. C1E0 04        |SHL EAX,4

               ;①EAX=25,EAX=25

左移四位,EAX=25 shl 4=250

             ;②EAX=14 右移四位，ＥＡＸ＝１４０

0046AF5D  |. 33D2           |XOR EDX,EDX

       ;①②EDX清零

0046AF5F  |. 8A53 02        |MOV DL,BYTE PTR DS:[EBX+2]

     ;①[EBX+2]入DL,DL=[EBX+2]=0E

             ;②[EBX+2]入DL,DL=[EBX+2]=31

0046AF62  |. C1EA 02        |SHR EDX,2

               ;①EDX=0E>>2=03

             ;②EDX=31>>2=0C

0046AF65  |. 80E2 0F        |AND DL,0F　　　　　　　　　　　;①取底位,EDX=03

             ;②取底位,EDX=0C

0046AF68  |. 0AC2           |OR AL,DL

                  ;①EAX=253

              ;②AL =  0C OR　4０＝4C

0046AF6A  |. 8845 F2        |MOV BYTE PTR SS:[EBP-E],AL

  ;①AL为EAX低位,所以AL=53,入[EBP-E]

             ;②AL=4C入[EBP-E]

0046AF6D  |. 8A43 02        |MOV AL,BYTE PTR DS:[EBX+2]

  ;①[EBX+2]=0E;EAX=20E

             ;②[EBX+2]=31;EAX=131

0046AF70  |. C1E0 06        |SHL EAX,6

               ;①EAX=20E<<6=380

             ;②EAX=131<<6=4C40

0046AF73  |. 8A53 03        |MOV DL,BYTE PTR DS:[EBX+3]

  ;①[EBX+3]=1A入DL

             ;②①[EBX+3]=3E入DL

0046AF76  |. 80E2 3F        |AND DL,3F

               ;①DL=1A&3F=1A

             ;②DL=３E &3F=3E

0046AF79  |. 0AC2           |OR AL,DL

                  ;①AL=80,AL=80

or 1A=9A

             ;②AL=40,AL=40 OR 3E =

7E

0046AF7B  |. 8845 F3        |MOV BYTE PTR SS:[EBP-D],AL

  ;①9A入[EBP-D]。[EBP-D]=9A 53 12 00    <--记住

             ;②7E入[EBP-D]。[EBP-D]=7E

4C 15 00　<--记住

0046AF7E  |. 8B45 F8        |MOV EAX,DWORD PTR SS:[EBP-8]

;①将[EBP-8]入EAX

             ;②

0046AF81  |. E8 1691F9FF    |CALL UNPACKED.0040409C

0046AF86  |. 8BD6           |MOV EDX,ESI

               ;①ESI=EDX=1

0046AF88  |. 4A             |DEC EDX

0046AF89  |. 8D1452         |LEA EDX,DWORD PTR DS:[EDX+EDX*2]

0046AF8C  |. 8D1410         |LEA EDX,DWORD PTR DS:[EAX+EDX]

0046AF8F  |. 8D45 F1        |LEA EAX,DWORD PTR SS:[EBP-F]

;

0046AF92  |. B9 03000000    |MOV ECX,3

0046AF97  |. E8 F078F9FF    |CALL UNPACKED.0040288C

0046AF9C  |. 46             |INC ESI

0046AF9D  |. 4F             |DEC EDI

                    ;①判断是否取完要取的字符。

0046AF9E  |.^0F85 8BFEFFFF  \JNZ UNPACKED.0046AE2F

    ;①没有就重新跳回去。

0046AFA4  |> 8B45 FC        MOV EAX,DWORD PTR
SS:[EBP-4]

0046AFA7  |. E8 208FF9FF    CALL UNPACKED.00403ECC

0046AFAC  |. 85C0           TEST EAX,EAX

0046AFAE  |. 0F84 83030000  JE UNPACKED.0046B337

0046AFB4  |. 8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]

0046AFB7  |. E8 108FF9FF    CALL UNPACKED.00403ECC

0046AFBC  |. 8B55 FC        MOV EDX,DWORD PTR SS:[EBP-4]

     ;②[EBP-4]=加密密码

0046AFBF  |. 8D4402 FC      LEA EAX,DWORD PTR DS:[EDX+EAX-4]

 ;②EAX=c,执行后EAX= "2k=="

0046AFC3  |. 8BD3           MOV EDX,EBX

0046AFC5  |. B9 04000000    MOV ECX,4

0046AFCA  |. E8 BD78F9FF    CALL UNPACKED.0040288C

0046AFCF  |. 807B 02 3D     CMP BYTE PTR DS:[EBX+2],3D  ;②第三个是不是=

0046AFD3  |. 0F85 B1000000  JNZ UNPACKED.0046B08A

  ;②不是=就跳,所以这里不跳

0046AFD9  |. 8A03           MOV AL,BYTE PTR
DS:[EBX]    ;②取第一个"2k=="的第一个这里为2(32)

0046AFDB  |. 3C 40          CMP AL,40

                ;②和40=@比较，小于或等于就跳

0046AFDD  |. 76 09          JBE SHORT UNPACKED.0046AFE8

;②32<40,所以跳0046AFE8---->

0046AFDF  |. 3C 5B          CMP AL,5B

0046AFE1  |. 73 05          JNB SHORT UNPACKED.0046AFE8

0046AFE3  |. 802B 41        SUB BYTE PTR DS:[EBX],41

0046AFE6  |. EB 2B          JMP SHORT UNPACKED.0046B013

0046AFE8  |> 8A03           MOV AL,BYTE PTR
DS:[EBX]     ;②从0046AFDD跳来,

0046AFEA  |. 3C 60          CMP AL,60

                 ;②和60比较

0046AFEC  |. 76 09          JBE SHORT UNPACKED.0046AFF7

 ;②32<60,所以跳0046AFF7---->

0046AFEE  |. 3C 7B          CMP AL,7B

0046AFF0  |. 73 05          JNB SHORT UNPACKED.0046AFF7

0046AFF2  |. 802B 47        SUB BYTE PTR DS:[EBX],47

0046AFF5  |. EB 1C          JMP SHORT UNPACKED.0046B013

0046AFF7  |> 803B 2F        CMP BYTE PTR DS:[EBX],2F

     ;②从0046AFEC跳,

0046AFFA  |. 76 0A          JBE SHORT UNPACKED.0046B006

  ;②32>2F,不跳

0046AFFC  |. 803B 3A        CMP BYTE PTR DS:[EBX],3A

0046AFFF  |. 73 05          JNB SHORT UNPACKED.0046B006

  ;②不小于跳,因为32<3A,所以不跳

0046B001  |. 8003 04        ADD BYTE PTR DS:[EBX],4

      ;②加上4,32+4=36,[EBX]=36

0046B004  |. EB 0D          JMP SHORT UNPACKED.0046B013

  ;②继续跳0046B013-->

0046B006  |> 803B 2B        CMP BYTE PTR DS:[EBX],2B

0046B009  |. 75 05          JNZ SHORT UNPACKED.0046B010

0046B00B  |. C603 3E        MOV BYTE PTR DS:[EBX],3E

0046B00E  |. EB 03          JMP SHORT UNPACKED.0046B013

0046B010  |> C603 3F        MOV BYTE PTR DS:[EBX],3F

0046B013  |> 8A43 01        MOV AL,BYTE PTR DS:[EBX+1]

    ;②取第二个,这里[EBX+1]=K(6B)

0046B016  |. 3C 40          CMP AL,40
```
