
```asm
00413CE8  JL      SHORT Coursewa.00413CEF****************小于就跳!
00413CEA  CMP     EAX, 7A*****************************与7A比较!
00413CED  JLE     SHORT Coursewa.00413D00****************小于等于就跳!
00413CEF  CMP     EAX, 31 ****************************与31比较!
00413CF2  JL      SHORT Coursewa.00413CF9****************小于就跳!
00413CF4  CMP     EAX, 39*****************************与39比较!
00413CF7  JLE     SHORT Coursewa.00413D00****************小于等于就跳!
00413CF9  XOR     EAX, 0E*****************************EAX=EAX XOR 0E=3C XOR 0E=32
00413CFC  OR      AL, 31******************************AL与31相或!32 OR 31=33(3的ASCII码值)
00413CFE  JMP     SHORT Coursewa.00413CB4
00413D00  RETN **************************************返回!
==========================================================================================
00413A3D  ADD     ESP, 4
00413A40  MOV     BYTE PTR SS:[ESP+ESI+8], AL**********保存第一位运算得到的值：3
00413A44  INC     ESI*********************************ESI加1
00413A45  CMP     ESI, 4******************************与4比较!
00413A48  JL      SHORT Coursewa.00413A25****************小于就跳!这一循环得到第一组注册码：3371
00413A4A  MOV     EDI, DWORD PTR SS:[ESP+30]***********试验码："1234-5678-9009-8765"移入EDI
00413A4E  LEA     ECX, DWORD PTR SS:[ESP+8]
00413A52  MOV     ESI, EDI
00413A54  XOR     EDX, EDX
00413A56  SUB     ESI, ECX
00413A58  LEA     ECX, DWORD PTR SS:[ESP+EDX+8]
00413A5C  MOVSX   EAX, BYTE PTR DS:[ESI+ECX]***********试验码第一组的第一位1(31)扩展到EAX
00413A60  CMP     EAX, 41*****************************与41比较!
00413A63  JL      SHORT Coursewa.00413A6D****************小于就跳
00413A65  CMP     EAX, 5A*****************************与5A比较!
00413A68  JG      SHORT Coursewa.00413A6D****************大于就跳!
00413A6A  ADD     EAX, 20*****************************EAX的值加上20
00413A6D  MOVSX   ECX, BYTE PTR DS:[ECX]***************第一组注册码的第一位3(33)扩展到ECX
00413A70  CMP     EAX, ECX*****************************比较第一位注册码
00413A72  JNZ     Coursewa.00413BAA*********************不相等就跳，跳就失败!所以要继续，把JNZ改成JE
00413A78  INC     EDX
00413A79  CMP     EDX, 4
00413A7C  JL      SHORT Coursewa.00413A58******************跳回循环，直到第一组的4位注册码比较完!
00413A7E  MOV     EAX, DWORD PTR SS:[ESP+8]***************第一组注册码倒序排列：31373333移入EAX
00413A82  LEA     EDX, DWORD PTR DS:[EAX+EAX*2]*********EDX=31373333+31373333*2=93A59999
00413A85  LEA     ECX, DWORD PTR DS:[EDX*4+9]***********ECX=93A59999*4+9=4E96666D
00413A8C  IMUL    ECX, EAX******************************ECX=4E96666D*31373333=7CC91EB7
00413A8F  MOV     DWORD PTR SS:[ESP+C], ECX*************保存这个值
00413A93  XOR     ESI, ESI******************************ESI清0
00413A95  MOV     DL, BYTE PTR SS:[ESP+ESI+C]***********最后一组值B7移入DL
00413A99  PUSH    EDX
00413A9A  PUSH    ESI
00413A9B  CALL    Coursewa.00413BC0*********************调用算法CALL(1)，看上面!
00413AA0  AND     EAX, 0FF******************************313733DB AND　0FF＝DB
00413AA5  PUSH    EAX
00413AA6  CALL    Coursewa.00413CB0*********************调用算法CALL(1)，看上面!得到第二组的第一位注册码：75(u的ASCII码值)
00413AAB  ADD     ESP, 0C
00413AAE  MOV     BYTE PTR SS:[ESP+ESI+C], AL***********保存这个值
00413AB2  INC     ESI
00413AB3  CMP     ESI, 4
00413AB6  JL      SHORT Coursewa.00413A95****跳回循环，直至算出第二组注册码，最后结果是：u1r1
00413AB8  XOR     ECX, ECX
00413ABA  MOVSX   EAX, BYTE PTR DS:[EDI+ECX+5]****第二组试验码的第一位5(35)扩展到EAX
00413ABF  CMP     EAX, 41
00413AC2  JL      SHORT Coursewa.00413ACC
00413AC4  CMP     EAX, 5A
00413AC7  JG      SHORT Coursewa.00413ACC
00413AC9  ADD     EAX, 20
00413ACC  MOVSX   EDX, BYTE PTR SS:[ESP+ECX+C]****第二组注册码的第一位：75(u)扩展到ECX
00413AD1  CMP     EAX, EDX*****************************比较试验码与注册码是否相等
00413AD3  JNZ     Coursewa.00413BAA*********************不相等就跳!要继续这里也改成JE
00413AD9  INC     ECX
00413ADA  CMP     ECX, 4
00413ADD  JL      SHORT Coursewa.00413ABA****跳回循环，直至比较完4位注册码!
00413ADF  MOV     EAX, DWORD PTR SS:[ESP+C]*************第二组注册码倒序排列：31723175移入EAX
00413AE3  MOV     ECX, DWORD PTR SS:[ESP+8]*************第一组注册码倒序排列：31373333移入ECX
00413AE7  MOV     EDX, EAX********************************EAX的值移入EDX!
00413AE9  XOR     EDX, ECX********************************EDX=31723175 XOR 31373333=00450246
00413AEB  INC     EDX*************************************EDX加1=00450247
00413AEC  IMUL    EDX, ECX********************************EDX=00450247*31373333=E1749925
00413AEF  LEA     ECX, DWORD PTR DS:[EAX+EAX*4]*********ECX=31723175+31723175*4=F73AF749
00413AF2  LEA     EAX, DWORD PTR DS:[EAX+ECX*8]*********EAX=31723175+F73AF749*8=EB49EBBD
00413AF5  LEA     EAX, DWORD PTR DS:[EAX+EAX*2]*********EAX=EB49EBBD+EB49EBBD*2=C1DDC337
00413AF8  ADD     EDX, EAX********************************EDX=E1749925+C1DDC337=A3525C5C
00413AFA  XOR     ESI, ESI
00413AFC  MOV     DWORD PTR SS:[ESP+10], EDX************保存EDX的值A3525C5C
00413B00  MOVSX   ECX, BYTE PTR SS:[ESP+ESI+10]*********把最后两位数值上：5C扩展到ECX中
00413B05  PUSH    ECX
00413B06  CALL    Coursewa.00413CB0*********************调用算法CALL(1)，看上面!
00413B0B  ADD     ESP, 4
00413B0E  MOV     BYTE PTR SS:[ESP+ESI+10], AL**********得到第三组注册码的第一位：s，保存s
00413B12  INC     ESI
00413B13  CMP     ESI, 4
00413B16  JL      SHORT Coursewa.00413B00******************跳回循环，计算出第三组4位注册码：ssr3
00413B18  XOR     ECX, ECX
00413B1A  MOVSX   EAX, BYTE PTR DS:[EDI+ECX+A]***********第三组试验码的第一位：9(39)扩展到EAX
00413B1F  CMP     EAX, 41
00413B22  JL      SHORT Coursewa.00413B2C
00413B24  CMP     EAX, 5A
00413B27  JG      SHORT Coursewa.00413B2C
00413B29  ADD     EAX, 20
00413B2C  MOVSX   EDX, BYTE PTR SS:[ESP+ECX+10]*********第三组注册码的第一位：s(73)扩展到EDX
00413B31  CMP     EAX, EDX********************************比较第一个试验码与注册码!
00413B33  JNZ     SHORT Coursewa.00413BAA*****************不相等就跳，一跳就失败!
00413B35  INC     ECX
00413B36  CMP     ECX, 4
00413B39  JL      SHORT Coursewa.00413B1A******************跳回循环，直至比较完4位注册码!
00413B3B  MOV     EAX, DWORD PTR SS:[ESP+C]***************第二组注册码倒序排列：31723175移入EAX
00413B3F  MOV     ECX, DWORD PTR SS:[ESP+8]***************第一组注册码倒序排列：31373333移入ECX
00413B43  IMUL    EAX, ECX*********************************EAX=31723175*31373333=9EBD294F
00413B46  LEA     EDX, DWORD PTR DS:[EAX*8]***************EDX=9EBD294F*8=F5E94A78
00413B4D  SUB     EDX, EAX*********************************EDX=F5E94A78-9EBD294F=572C2129
00413B4F  LEA     EAX, DWORD PTR DS:[ECX+ECX*4]*********EAX=31373333+31373333*4=F613FFFF
00413B52  ADD     EDX, 5**********************************EDX=572C2129+5=572C212E
00413B55  IMUL    EDX, DWORD PTR SS:[ESP+10]*************SS:[ESP+10]保存的值是第三组注册码的倒序排列：33727373
*********************************************************EDX=33727373*572C212E=EC3691AA
00413B5A  ADD     EDX, EAX********************************EDX=EC3691AA+F613FFFF=E24A91A9
00413B5C  XOR     ESI, ESI
00413B5E  MOV     DWORD PTR SS:[ESP+14], EDX*************保存E24A91A9
00413B62  MOVSX   ECX, BYTE PTR SS:[ESP+ESI+14]*********把上述值的最后两位：A9扩展到ECX
00413B67  PUSH    ECX
00413B68  CALL    Coursewa.00413CB0*********************调用算法CALL(1)，看上面!
00413B6D  ADD     ESP, 4
```
