
```
0047CBF7   .  FFD3          CALL EBX                                            ;  <&msvbvm60.rtcMidCharBstr>        〈----取用户名的第一个字符

0047CBF9   .  8B3D C8124000 MOV EDI,DWORD PTR DS:[<&msvbvm60.\_\_vbaStrMove>]

0047CBFF   .  8BD0          MOV EDX,EAX

0047CC01   .  8D4D A0       LEA ECX,DWORD PTR SS:[EBP-60]

0047CC04   .  FFD7          CALL EDI                                            ;  <&msvbvm60.\_\_vbaStrMove>

0047CC06   .  8B35 60104000 MOV ESI,DWORD PTR DS:[<&msvbvm60.rtcAnsiValueBstr>] ;  msvbvm60.rtcAnsiValueBstr

0047CC0C   .  50            PUSH EAX

0047CC0D   .  FFD6          CALL ESI                                            ;  <&msvbvm60.rtcAnsiValueBstr>        〈--- 获得 ASC 值

0047CC0F   .  66:99         CWD

0047CC11   .  66:B9 B100    MOV CX,0B1

0047CC15   .  66:F7F9       IDIV CX

0047CC18   .  66:8BCA       MOV CX,DX

0047CC1B   .  66:83C1 07    ADD CX,7                        〈---- CX = 第一个字符的 ASC 值 ，加上 7

0047CC1F   .  0F80 0E170000 JO 扩展名仓.0047E333

0047CC25   .  FF15 74104000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaI2Abs>]          ;  msvbvm60.\_\_vbaI2Abs

0047CC2B   .  0FBFD0        MOVSX EDX,AX

0047CC2E   .  8B45 E0       MOV EAX,DWORD PTR SS:[EBP-20]

0047CC31   .  8D4D A0       LEA ECX,DWORD PTR SS:[EBP-60]

0047CC34   .  8910          MOV DWORD PTR DS:[EAX],EDX

0047CC36   .  FF15 04134000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaFreeStr>]        ;  msvbvm60.\_\_vbaFreeStr

0047CC3C   .  8D4D 8C       LEA ECX,DWORD PTR SS:[EBP-74]

0047CC3F   .  FF15 28104000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaFreeVar>]        ;  msvbvm60.\_\_vbaFreeVar

0047CC45   .  8B0D 00514A00 MOV ECX,DWORD PTR DS:[4A5100]

0047CC4B   .  C745 CC FFFFF>MOV DWORD PTR SS:[EBP-34],-1

0047CC52   .  51            PUSH ECX

0047CC53   .  FF15 3C104000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaLenBstr>]        〈----这里获得用户名的长度。

0047CC59   .  83F8 01       CMP EAX,1                                                〈---- 当用户名的长度等于 1 的时候的处理方式

0047CC5C   .  75 76         JNZ SHORT 扩展名仓.0047CCD4

0047CC5E   .  8B55 E0       MOV EDX,DWORD PTR SS:[EBP-20]

0047CC61   .  8B0A          MOV ECX,DWORD PTR DS:[EDX]

0047CC63   .  6BC9 02       IMUL ECX,ECX,2

0047CC66   .  0F80 C7160000 JO 扩展名仓.0047E333

0047CC6C   .  83C1 11       ADD ECX,11

0047CC6F   .  0F80 BE160000 JO 扩展名仓.0047E333

0047CC75   .  FF15 CC104000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaI4Abs>]          ;  msvbvm60.\_\_vbaI4Abs

0047CC7B   .  8B4D E0       MOV ECX,DWORD PTR SS:[EBP-20]

0047CC7E   .  8941 04       MOV DWORD PTR DS:[ECX+4],EAX

0047CC81   .  8B55 E0       MOV EDX,DWORD PTR SS:[EBP-20]

0047CC84   .  8B0A          MOV ECX,DWORD PTR DS:[EDX]

0047CC86   .  83C1 4D       ADD ECX,4D

0047CC89   .  0F80 A4160000 JO 扩展名仓.0047E333

0047CC8F   .  FF15 CC104000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaI4Abs>]          ;  msvbvm60.\_\_vbaI4Abs

0047CC95   .  8B4D E0       MOV ECX,DWORD PTR SS:[EBP-20]

0047CC98   .  8941 08       MOV DWORD PTR DS:[ECX+8],EAX

0047CC9B   .  8B55 E0       MOV EDX,DWORD PTR SS:[EBP-20]

0047CC9E   .  8B0A          MOV ECX,DWORD PTR DS:[EDX]

0047CCA0   .  83E9 07       SUB ECX,7

0047CCA3   .  0F80 8A160000 JO 扩展名仓.0047E333

0047CCA9   .  FF15 CC104000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaI4Abs>]          ;  msvbvm60.\_\_vbaI4Abs

0047CCAF   .  8B4D E0       MOV ECX,DWORD PTR SS:[EBP-20]

0047CCB2   .  8941 0C       MOV DWORD PTR DS:[ECX+C],EAX

0047CCB5   .  8B55 E0       MOV EDX,DWORD PTR SS:[EBP-20]

0047CCB8   .  8B0A          MOV ECX,DWORD PTR DS:[EDX]

0047CCBA   .  83C1 07       ADD ECX,7

0047CCBD   .  0F80 70160000 JO 扩展名仓.0047E333

0047CCC3   .  FF15 CC104000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaI4Abs>]          ;  msvbvm60.\_\_vbaI4Abs

0047CCC9   .  8B4D E0       MOV ECX,DWORD PTR SS:[EBP-20]

0047CCCC   .  8941 10       MOV DWORD PTR DS:[ECX+10],EAX

0047CCCF   .  E9 F0060000   JMP 扩展名仓.0047D3C4

0047CCD4   >  8B15 00514A00 MOV EDX,DWORD PTR DS:[4A5100]

0047CCDA   .  52            PUSH EDX

0047CCDB   .  FF15 3C104000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaLenBstr>]        ;  msvbvm60.\_\_vbaLenBstr

0047CCE1   .  83F8 02       CMP EAX,2                                                〈---- 当用户名的长度等于 2 的时候的处理方式

0047CCE4   .  0F85 AF000000 JNZ 扩展名仓.0047CD99

0047CCEA   .  8B0D 00514A00 MOV ECX,DWORD PTR DS:[4A5100]

0047CCF0   .  8945 8C       MOV DWORD PTR SS:[EBP-74],EAX

0047CCF3   .  8D45 8C       LEA EAX,DWORD PTR SS:[EBP-74]

0047CCF6   .  C745 94 01000>MOV DWORD PTR SS:[EBP-6C],1

0047CCFD   .  50            PUSH EAX

0047CCFE   .  6A 02         PUSH 2

0047CD00   .  51            PUSH ECX

0047CD01   .  FFD3          CALL EBX

0047CD03   .  8BD0          MOV EDX,EAX

0047CD05   .  8D4D A0       LEA ECX,DWORD PTR SS:[EBP-60]

0047CD08   .  FFD7          CALL EDI

0047CD0A   .  50            PUSH EAX

0047CD0B   .  FFD6          CALL ESI

0047CD0D   .  66:99         CWD

0047CD0F   .  66:B9 B100    MOV CX,0B1

0047CD13   .  66:F7F9       IDIV CX

0047CD16   .  66:8BCA       MOV CX,DX

0047CD19   .  66:83C1 11    ADD CX,11

0047CD1D   .  0F80 10160000 JO 扩展名仓.0047E333

0047CD23   .  FF15 74104000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaI2Abs>]          ;  msvbvm60.\_\_vbaI2Abs

0047CD29   .  0FBFD0        MOVSX EDX,AX

0047CD2C   .  8B45 E0       MOV EAX,DWORD PTR SS:[EBP-20]

0047CD2F   .  8D4D A0       LEA ECX,DWORD PTR SS:[EBP-60]

0047CD32   .  8950 04       MOV DWORD PTR DS:[EAX+4],EDX

0047CD35   .  FF15 04134000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaFreeStr>]        ;  msvbvm60.\_\_vbaFreeStr

0047CD3B   .  8D4D 8C       LEA ECX,DWORD PTR SS:[EBP-74]

0047CD3E   .  FF15 28104000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaFreeVar>]        ;  msvbvm60.\_\_vbaFreeVar

0047CD44   .  8B4D E0       MOV ECX,DWORD PTR SS:[EBP-20]

0047CD47   .  8B09          MOV ECX,DWORD PTR DS:[ECX]

0047CD49   .  83C1 1B       ADD ECX,1B

0047CD4C   .  0F80 E1150000 JO 扩展名仓.0047E333

0047CD52   .  FF15 CC104000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaI4Abs>]          ;  msvbvm60.\_\_vbaI4Abs

0047CD58   .  8B55 E0       MOV EDX,DWORD PTR SS:[EBP-20]

0047CD5B   .  8942 08       MOV DWORD PTR DS:[EDX+8],EAX

0047CD5E   .  8B45 E0       MOV EAX,DWORD PTR SS:[EBP-20]

0047CD61   .  8B48 04       MOV ECX,DWORD PTR DS:[EAX+4]

0047CD64   .  83E9 2F       SUB ECX,2F

0047CD67   .  0F80 C6150000 JO 扩展名仓.0047E333

0047CD6D   .  FF15 CC104000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaI4Abs>]          ;  msvbvm60.\_\_vbaI4Abs

0047CD73   .  8B4D E0       MOV ECX,DWORD PTR SS:[EBP-20]

0047CD76   .  8941 0C       MOV DWORD PTR DS:[ECX+C],EAX

0047CD79   .  8B55 E0       MOV EDX,DWORD PTR SS:[EBP-20]

0047CD7C   .  8B4A 08       MOV ECX,DWORD PTR DS:[EDX+8]

0047CD7F   .  83E9 11       SUB ECX,11

0047CD82   .  0F80 AB150000 JO 扩展名仓.0047E333

0047CD88   .  FF15 CC104000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaI4Abs>]          ;  msvbvm60.\_\_vbaI4Abs

0047CD8E   .  8B4D E0       MOV ECX,DWORD PTR SS:[EBP-20]

0047CD91   .  8941 10       MOV DWORD PTR DS:[ECX+10],EAX

0047CD94   .  E9 2B060000   JMP 扩展名仓.0047D3C4

0047CD99   >  8B15 00514A00 MOV EDX,DWORD PTR DS:[4A5100]

0047CD9F   .  52            PUSH EDX

0047CDA0   .  FF15 3C104000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaLenBstr>]        ;  msvbvm60.\_\_vbaLenBstr

0047CDA6   .  83F8 03       CMP EAX,3                                                〈---- 当用户名的长度等于 3 的时候的处理方式

0047CDA9   .  0F85 02010000 JNZ 扩展名仓.0047CEB1

0047CDAF   .  8B0D 00514A00 MOV ECX,DWORD PTR DS:[4A5100]

0047CDB5   .  8D45 8C       LEA EAX,DWORD PTR SS:[EBP-74]

0047CDB8   .  50            PUSH EAX

0047CDB9   .  6A 02         PUSH 2

0047CDBB   .  51            PUSH ECX

0047CDBC   .  C745 94 01000>MOV DWORD PTR SS:[EBP-6C],1

0047CDC3   .  C745 8C 02000>MOV DWORD PTR SS:[EBP-74],2

0047CDCA   .  FFD3          CALL EBX

0047CDCC   .  8BD0          MOV EDX,EAX

0047CDCE   .  8D4D A0       LEA ECX,DWORD PTR SS:[EBP-60]

0047CDD1   .  FFD7          CALL EDI

0047CDD3   .  50            PUSH EAX

0047CDD4   .  FFD6          CALL ESI

0047CDD6   .  66:99         CWD
```
