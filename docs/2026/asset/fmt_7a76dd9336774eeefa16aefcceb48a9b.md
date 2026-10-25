
***这个函数见过了，就是串联字符串达到`0x1E`

```
005B3713  ADD ESP,0C

005B3716  TEST AL,AL

005B3718  JNZ SHORT starryni.005B3721

005B371A  MOV EAX,4455EE33
```

*奇怪的返回值表示失败*

```
005B371F  JMP SHORT starryni.005B3754

005B3721  LEA EDI,DWORD PTR SS:[EBP-2C]
```
*注册名字相连而成的长度为`1E`的串*

```
005B3724  XOR EAX,EAX

005B3726  OR ECX,FFFFFFFF

005B3729  REPNE SCAS BYTE PTR ES:[EDI]

005B372B  MOV ESI,-2

005B3730  SUB ESI,ECX

005B3732  XOR EBX,EBX
```
*`ebx`是索引index*

```
005B3734  JMP SHORT starryni.005B374D

005B3736  /MOVZX ECX,BYTE PTR SS:[EBP+EBX-2C]
```
*`ecx` = ch*

```
005B373B  |MOV EDX,EBX
```
*`edx` = index*

```
005B373D  |IMUL EDX,EBX
```
*`edx` = index**2*

```
005B3740  |IMUL EDX,EBX
```
*`edx` = index**3*

```
005B3743  |IMUL EDX,ECX
```
*`edx` = index**3*ch*

```
005B3746  |IMUL EDX,ECX
```
*`edx` = index**3*ch**2*

```
005B3749  |ADD DWORD PTR SS:[EBP-30],EDX
```
*加到常数`75BCD15`上*

```
005B374C  |INC EBX

005B374D   CMP EBX,ESI

005B374F  \JL SHORT starryni.005B3736

005B3751  MOV EAX,DWORD PTR SS:[EBP-30]

005B3754  LEA ESP,DWORD PTR SS:[EBP-C]

005B3757  POP EDI

005B3758  POP ESI

005B3759  POP EBX

005B375A  POP EBP

005B375B  RETN
```

这个过程计算名字的检查和，同样先把名字串联达到`0x1E`长度，然后计算：

`checksum = sum(index**3*ch**2) + 0x75BCD15`

```
005B3508  |. 59  POP ECX

005B3509  |. 89C6           MOV ESI,EAX
```
*名字的检查和*

```
005B350B  |. 8D53 F8        LEA EDX,DWORD PTR DS:[EBX-8]

005B350E  |. 8D4D CC        LEA ECX,DWORD PTR SS:[EBP-34]
```
*注册码前4部分相连并重排后的字符串*

```
005B3511  |. 01CA           ADD EDX,ECX
```
*`edx`指向该字符串的后面8个字符*

```
005B3513  |. 52             PUSH EDX

005B3514  |. E8 6A040000    CALL starryni.005B3983
```
*转换成16进制数码*

```
005B3519  |. 59             POP ECX

005B351A  |. 39F0           CMP EAX,ESI
```
*结果与名字的检查和比较*

```
005B351C  |. 74 0B          JE SHORT starryni.005B3529
```
*必须相等，否则失败*

```
005B351E  |. 8B45 18        MOV EAX,DWORD PTR SS:[EBP+18]

005B3521  |. C600 04        MOV BYTE PTR DS:[EAX],4

005B3524  |. E9 B6000000    JMP starryni.005B35DF

005B3529  |> 8D45 CF        LEA EAX,DWORD PTR SS:[EBP-31]
```
*重排后注册码的第2部分*

```
005B352C  |. 50             PUSH EAX

005B352D  |. E8 FF030000    CALL starryni.005B3931
```
*转换成16进制数*

```
005B3532  |. 59             POP ECX

005B3533  |. 0FB7C0         MOVZX EAX,AX
```
只保留低位字

```
005B3536  |. 89C3           MOV EBX,EAX

005B3538  |. E8 2E010000    CALL starryni.005B366B
```
*这个函数没有搞懂。看上去十分复杂，很多层次*
*子程序调用，但与名字及注册码均无关，最后返回*

```
005B353D  |. 89DA           MOV EDX,EBX
```
*`0x135A`*

```
005B353F  |. 29C2           SUB EDX,EAX
```
*第2部分的值减去这个`135A`*

```
005B3541  |. 8B45 14        MOV EAX,DWORD PTR SS:[EBP+14]

005B3544  |. 8910           MOV DWORD PTR DS:[EAX],EDX
```
*如果结果小于0就会失败*

```
005B3546  |. 0FBE45 98      MOVSX EAX,BYTE PTR SS:[EBP-68]
```
*这里是注册码第二个字符，可能决定注册的版本*

```
005B354A  |. 83E8 63        SUB EAX,63

005B354D  |. 74 26          JE SHORT starryni.005B3575

005B354F  |. 83E8 0B        SUB EAX,0B

005B3552  |. 74 56          JE SHORT starryni.005B35AA

005B3554  |. 83E8 04        SUB EAX,4

005B3557  |. 74 3B          JE SHORT starryni.005B3594

005B3559  |. 83E8 02        SUB EAX,2

005B355C  |. 74 07          JE SHORT starryni.005B3565

005B355E  |. 83E8 01        SUB EAX,1

005B3561  |. 74 47          JE SHORT starryni.005B35AA

005B3563  |. EB 4D          JMP SHORT starryni.005B35B2

005B3565  |> 8B45 14        MOV EAX,DWORD PTR SS:[EBP+14]
```
*`'t'`，注册15天试用版*

```
005B3568  |. 8338 00        CMP DWORD PTR DS:[EAX],0

005B356B  |. 7C 2F          JL SHORT starryni.005B359C

005B356D  |. 837D 1C 28     CMP DWORD PTR SS:[EBP+1C],28

005B3571  |. 7E 47          JLE SHORT starryni.005B35BA

005B3573  |. EB 27          JMP SHORT starryni.005B359C

005B3575  |> 8B45 14        MOV EAX,DWORD PTR SS:[EBP+14]
```
*`'c'`，不明用处*

```
005B3578  |. 8338 00        CMP DWORD PTR DS:[EAX],0

005B357B  |. 7C 1F          JL SHORT starryni.005B359C

005B357D  |. 837D 1C 14     CMP DWORD PTR SS:[EBP+1C],14

005B3581  |. 7E 02          JLE SHORT starryni.005B3585

005B3583  |. EB 17          JMP SHORT starryni.005B359C

005B3585  |> BA 14000000    MOV EDX,14

005B358A  |. 2B55 1C        SUB EDX,DWORD PTR SS:[EBP+1C]

005B358D  |. 8B45 14        MOV EAX,DWORD PTR SS:[EBP+14]

005B3590  |. 8910           MOV DWORD PTR DS:[EAX],EDX

005B3592  |. EB 26          JMP SHORT starryni.005B35BA

005B3594  |> 8B45 14        MOV EAX,DWORD PTR SS:[EBP+14]
```
*`'r'`，好像是正式的backyard版*

```
005B3597  |. 8338 00        CMP DWORD PTR DS:[EAX],0

005B359A  |. 7D 1E          JGE SHORT starryni.005B35BA

005B359C  |> 8B45 18        MOV EAX,DWORD PTR SS:[EBP+18]

005B359F  |. C600 03        MOV BYTE PTR DS:[EAX],3

005B35A2  |. 8B45 14        MOV EAX,DWORD PTR SS:[EBP+14]

005B35A5  |. 8320 00        AND DWORD PTR DS:[EAX],0

005B35A8  |. EB 35          JMP SHORT starryni.005B35DF

005B35AA  |> 8B45 14        MOV EAX,DWORD PTR SS:[EBP+14]
```
*`'n'`和`'u'`，没看出有什么特别*

```
005B35AD  |. 8320 00        AND DWORD PTR DS:[EAX],0

005B35B0  |. EB 08          JMP SHORT starryni.005B35BA

005B35B2  |> 8B45 18        MOV EAX,DWORD PTR SS:[EBP+18]
```
; Default case of switch `005B354A`

```
005B35B5  |. C600 07        MOV BYTE PTR DS:[EAX],7

005B35B8  |. EB 25          JMP SHORT starryni.005B35DF

005B35BA  |> 8B45 0C        MOV EAX,DWORD PTR SS:[EBP+C]
```
; original code

```
005B35BD  |. 0FB610         MOVZX EDX,BYTE PTR DS:[EAX]

005B35C0  |. 3A55 20        CMP DL,BYTE PTR SS:[EBP+20]
```
*第1个字符必须是`'y'`*

```
005B35C3  |. 74 08          JE SHORT starryni.005B35CD

005B35C5  |. 8B45 18        MOV EAX,DWORD PTR SS:[EBP+18]

005B35C8  |. C600 09        MOV BYTE PTR DS:[EAX],9

005B35CB  |. EB 12          JMP SHORT starryni.005B35DF

005B35CD  |> 8B45 0C        MOV EAX,DWORD PTR SS:[EBP+C]

005B35D0  |. 0FB650 02      MOVZX EDX,BYTE PTR DS:[EAX+2]

005B35D4  |. 3A55 24        CMP DL,BYTE PTR SS:[EBP+24]
```
*第3个字符必须是`'4'`*

```
005B35D7  |. 74 0A          JE SHORT starryni.005B35E3

005B35D9  |. 8B45 18        MOV EAX,DWORD PTR SS:[EBP+18]

005B35DC  |. C600 0A        MOV BYTE PTR DS:[EAX],0A

005B35DF  |> 30C0           XOR AL,AL

005B35E1  |. EB 1C          JMP SHORT starryni.005B35FF

005B35E3  |> 807D 98 74     CMP BYTE PTR SS:[EBP-68],74

005B35E7  |. 74 06          JE SHORT starryni.005B35EF

005B35E9  |. 807D 98 63     CMP BYTE PTR SS:[EBP-68],63

005B35ED  |. 75 08          JNZ SHORT starryni.005B35F7

005B35EF  |> 8B45 18        MOV EAX,DWORD PTR SS:[EBP+18]

005B35F2  |. C600 01        MOV BYTE PTR DS:[EAX],1

005B35F5  |. EB 06          JMP SHORT starryni.005B35FD

005B35F7  |> 8B45 18        MOV EAX,DWORD PTR SS:[EBP+18]

005B35FA  |. C600 00        MOV BYTE PTR DS:[EAX],0

005B35FD  |> B0 01          MOV AL,1

005B35FF  |> 8D65 F4        LEA ESP,DWORD PTR SS:[EBP-C]

005B3602  |. 5F             POP EDI

005B3603  |. 5E             POP ESI

005B3604  |. 5B             POP EBX

005B3605  |. 5D             POP EBP

005B3606  \. C3             RETN
```

【分析总结】可用以下步骤生成注册码：

1 将名字剔除一切特殊字符（包括空格）后，重复拷贝至`0x1E`个字符长；

2 计算名字串的32-bit检查和 `checksum = sum(index**3*ch**2)+0x75BCD15`

3 将检查和数字转换成字符串（8个字符）

4 在该字符串头上加上`"yx4ssss"`,其中`'x'`可以是`t`,`r`,`n`,`u`或`c`；`s`可以是任何16进制数字但不能小于`0x135A`

5 将所得字符串复制、连接，正好达到`0x1E`个字符，计算检查和：

`checksum = sum(index*ch**2)-32EB`

6 所的检查和的低位字转换成字符串形式，这4个字符构成注册码的最后4位

7 由名字检查和生成的字符串（长度8）加上前面的任意4个16进制数字构成一个12字符长的串，
