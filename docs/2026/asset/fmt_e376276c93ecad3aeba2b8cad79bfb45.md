
```
00419FCF   .  FFD6          CALL ESI

00419FD1   .  8B4D 10       MOV ECX,DWORD PTR SS:[EBP+10]

00419FD4   .  8B55 0C       MOV EDX,DWORD PTR SS:[EBP+C]

00419FD7   .  50            PUSH EAX

00419FD8   .  51            PUSH ECX

00419FD9   .  0FBFC7        MOVSX EAX,DI

00419FDC   .  52            PUSH EDX

00419FDD   .  50            PUSH EAX

00419FDE   .  E8 9DF5FFFF   CALL 1.00419580                          ;  第二次调用重要的算法CALL1

00419FE3   .  50            PUSH EAX                                 ;  结果为SN2=11c

00419FE4   .  FF15 18104000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaStrI4>;  msvbvm60.\_\_vbaStrI4

00419FEA   .  8BD0          MOV EDX,EAX                              ;  转十进制为284

00419FEC   .  8D4D B4       LEA ECX,DWORD PTR SS:[EBP-4C]

00419FEF   .  FFD6          CALL ESI

00419FF1   .  50            PUSH EAX

00419FF2   .  FFD3          CALL EBX

00419FF4   .  8BD0          MOV EDX,EAX

00419FF6   .  8D4D C4       LEA ECX,DWORD PTR SS:[EBP-3C]

00419FF9   .  FFD6          CALL ESI

00419FFB   .  8D4D B4       LEA ECX,DWORD PTR SS:[EBP-4C]

00419FFE   .  8D55 B8       LEA EDX,DWORD PTR SS:[EBP-48]            ;  与前面的相连为973+284

0041A001   .  51            PUSH ECX

0041A002   .  8D45 BC       LEA EAX,DWORD PTR SS:[EBP-44]

0041A005   .  52            PUSH EDX

0041A006   .  50            PUSH EAX

0041A007   .  6A 03         PUSH 3

0041A009   .  FF15 E8114000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaFreeS>;  msvbvm60.\_\_vbaFreeStrList

0041A00F   .  8D4D 94       LEA ECX,DWORD PTR SS:[EBP-6C]

0041A012   .  8D55 A4       LEA EDX,DWORD PTR SS:[EBP-5C]

0041A015   .  51            PUSH ECX

0041A016   .  52            PUSH EDX

0041A017   .  6A 02         PUSH 2

0041A019   .  FF15 3C104000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaFreeV>;  msvbvm60.\_\_vbaFreeVarList

0041A01F   .  83C4 1C       ADD ESP,1C

0041A022   .  8D85 50FFFFFF LEA EAX,DWORD PTR SS:[EBP-B0]

0041A028   .  8D8D 60FFFFFF LEA ECX,DWORD PTR SS:[EBP-A0]

0041A02E   .  8D55 DC       LEA EDX,DWORD PTR SS:[EBP-24]

0041A031   .  50            PUSH EAX

0041A032   .  51            PUSH ECX

0041A033   .  52            PUSH EDX

0041A034   .  FF15 74124000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaVarFo>;  msvbvm60.\_\_vbaVarForNext

0041A03A   .  BF 02000000   MOV EDI,2

0041A03F   .^ E9 35FFFFFF   JMP 1.00419F79                           ;  反回去循环直到把用户名算完

0041A044   >  8D45 C4       LEA EAX,DWORD PTR SS:[EBP-3C]            ;  最终得字符串SN

0041A047   .  50            PUSH EAX

0041A048   .  E8 43F7FFFF   CALL 1.00419790                          ;  重要的算法CALL2跟进

0041A04D   .  8BD0          MOV EDX,EAX                              ;  把SN转为字符串记为S=“OTczKzI4NCs5NzMrMTU1KzQ1MSs2MTkrNzQw”

0041A04F   .  8D4D C8       LEA ECX,DWORD PTR SS:[EBP-38]

0041A052   .  FFD6          CALL ESI

0041A054   >  68 CCA04100   PUSH 1.0041A0CC

0041A059   .  EB 37         JMP SHORT 1.0041A092

0041A05B   .  F645 FC 04    TEST BYTE PTR SS:[EBP-4],4

0041A05F   .  74 09         JE SHORT 1.0041A06A

0041A061   .  8D4D C8       LEA ECX,DWORD PTR SS:[EBP-38]

0041A064   .  FF15 80124000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaFreeS>;  msvbvm60.\_\_vbaFreeStr

0041A06A   >  8D4D B4       LEA ECX,DWORD PTR SS:[EBP-4C]

0041A06D   .  8D55 B8       LEA EDX,DWORD PTR SS:[EBP-48]

0041A070   .  51            PUSH ECX

0041A071   .  8D45 BC       LEA EAX,DWORD PTR SS:[EBP-44]

0041A074   .  52            PUSH EDX

0041A075   .  50            PUSH EAX

0041A076   .  6A 03         PUSH 3

0041A078   .  FF15 E8114000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaFreeS>;  msvbvm60.\_\_vbaFreeStrList

0041A07E   .  8D4D 94       LEA ECX,DWORD PTR SS:[EBP-6C]

0041A081   .  8D55 A4       LEA EDX,DWORD PTR SS:[EBP-5C]

0041A084   .  51            PUSH ECX

0041A085   .  52            PUSH EDX

0041A086   .  6A 02         PUSH 2

0041A088   .  FF15 3C104000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaFreeV>;  msvbvm60.\_\_vbaFreeVarList

0041A08E   .  83C4 1C       ADD ESP,1C

0041A091   .  C3            RETN

。。。。。。跟进重要的算法CALL1 1.00419580到这。。。。。。

00419580   $  55            PUSH EBP

00419581   .  8BEC          MOV EBP,ESP

00419583   .  83EC 14       SUB ESP,14

。。。。。。

004195F1   >  85FF          TEST EDI,EDI

004195F3   .  0F8E 10010000 JLE 1.00419709                           ;   算法部分开始

004195F9   >  DB45 0C       FILD DWORD PTR SS:[EBP+C]

004195FC   .  DD9D 78FFFFFF FSTP QWORD PTR SS:[EBP-88]

00419602   .  DD85 78FFFFFF FLD QWORD PTR SS:[EBP-88]

00419608   .  833D 00C04100>CMP DWORD PTR DS:[41C000],0

0041960F   .  75 08         JNZ SHORT 1.00419619

00419611   .  DC35 C8194000 FDIV QWORD PTR DS:[4019C8]               ;  得第一部分的值/2

00419617   .  EB 11         JMP SHORT 1.0041962A

00419619   >  FF35 CC194000 PUSH DWORD PTR DS:[4019CC]

0041961F   .  FF35 C8194000 PUSH DWORD PTR DS:[4019C8]

00419625   .  E8 EA84FEFF   CALL <JMP.&msvbvm60.\_adj\_fdiv\_m64>

0041962A   >  DFE0          FSTSW AX

0041962C   .  A8 0D         TEST AL,0D

0041962E   .  0F85 4C010000 JNZ 1.00419780

00419634   .  FF15 DC104000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaFpR8>>;  msvbvm60.\_\_vbaFpR8

0041963A   .  8BC7          MOV EAX,EDI

0041963C   .  99            CDQ

0041963D   .  2BC2          SUB EAX,EDX

0041963F   .  D1F8          SAR EAX,1                                ;  右移一位相当于得第一部分的值\ 2

00419641   .  8985 74FFFFFF MOV DWORD PTR SS:[EBP-8C],EAX

00419647   .  DB85 74FFFFFF FILD DWORD PTR SS:[EBP-8C]

0041964D   .  DD9D 6CFFFFFF FSTP QWORD PTR SS:[EBP-94]

00419653   .  DC9D 6CFFFFFF FCOMP QWORD PTR SS:[EBP-94]

00419659   .  DFE0          FSTSW AX

0041965B   .  F6C4 40       TEST AH,40

0041965E   .  74 5B         JE SHORT 1.004196BB                      ;  得第一部分的值/2 = 第一部分的值\ 2就不跳走。第一次是1234不跳走。

00419660   .  8BC6          MOV EAX,ESI                              ;  即第一部分的值是偶数不跳走。如果是奇数就跳走，第二次来到这时是617要跳走。

00419662   .  0FAFC6        IMUL EAX,ESI                             ;  用户名第一个数的平方为我这是6c\*6c=2d90

00419665   .  0F80 1A010000 JO 1.00419785

0041966B   .  99            CDQ

0041966C   .  F7FB          IDIV EBX                                 ;  2d90除第三部分的十六进制值457, 取余数我这是22a

0041966E   .  8BF2          MOV ESI,EDX

00419670   .  DB45 0C       FILD DWORD PTR SS:[EBP+C]

00419673   .  DD9D 64FFFFFF FSTP QWORD PTR SS:[EBP-9C]

00419679   .  DD85 64FFFFFF FLD QWORD PTR SS:[EBP-9C]

0041967F   .  833D 00C04100>CMP DWORD PTR DS:[41C000],0

00419686   .  75 08         JNZ SHORT 1.00419690

00419688   .  DC35 C8194000 FDIV QWORD PTR DS:[4019C8]               ;  1234/2=617

0041968E   .  EB 11         JMP SHORT 1.004196A1

00419690   >  FF35 CC194000 PUSH DWORD PTR DS:[4019CC]

00419696   .  FF35 C8194000 PUSH DWORD PTR DS:[4019C8]

0041969C   .  E8 7384FEFF   CALL <JMP.&msvbvm60.\_adj\_fdiv\_m64>

004196A1   >  DFE0          FSTSW AX

004196A3   .  A8 0D         TEST AL,0D

004196A5   .  0F85 D5000000 JNZ 1.00419780

004196AB   .  FF15 38124000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaFpI4>>;  msvbvm60.\_\_vbaFpI4

004196B1   .  8BF8          MOV EDI,EAX

004196B3   .  897D 0C       MOV DWORD PTR SS:[EBP+C],EDI

004196B6   .^ E9 3EFFFFFF   JMP 1.004195F9                           ;  跳回去重复运算

004196BB   >  8975 A8       MOV DWORD PTR SS:[EBP-58],ESI

004196BE   .  B8 03000000   MOV EAX,3

004196C3   .  8945 A0       MOV DWORD PTR SS:[EBP-60],EAX

004196C6   .  895D 98       MOV DWORD PTR SS:[EBP-68],EBX

004196C9   .  8945 90       MOV DWORD PTR SS:[EBP-70],EAX

004196CC   .  8D45 A0       LEA EAX,DWORD PTR SS:[EBP-60]

004196CF   .  50            PUSH EAX                                 ;  运算结果22a

004196D0   .  8D4D D0       LEA ECX,DWORD PTR SS:[EBP-30]

004196D3   .  51            PUSH ECX                                 ;  数字 1

004196D4   .  8D55 C0       LEA EDX,DWORD PTR SS:[EBP-40]

004196D7   .  52            PUSH EDX

004196D8   .  FF15 54114000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaVarMu>;  msvbvm60.\_\_vbaVarMul

004196DE   .  50            PUSH EAX                                 ;  22a\*1=22a
```
