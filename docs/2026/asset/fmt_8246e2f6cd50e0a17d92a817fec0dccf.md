
00417653   .  3B45 CC       CMP EAX,DWORD PTR SS:[EBP-34]            ;  将EAX的值同[EBP-34]中的值比较必须相等，这里其实是判断注册码倒数16-14位十进制值等于去掉后16位假码值逐个异或结果的十进制值，由于这里确定了注册码倒数16-14位值，所以必须从头来过修改假码倒数16位值通过这第四个关口，继续下面的调试

00417656   .  74 20         JE SHORT 《真正的.00417678

00417658   .  33C0          XOR EAX,EAX

0041765A   .  BA 02000000   MOV EDX,2

0041765F   .  50            PUSH EAX

00417660   .  8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]

00417663   .  FF4D EC       DEC DWORD PTR SS:[EBP-14]

00417666   .  E8 A5A90800   CALL 《真正的.004A2010

0041766B   .  58            POP EAX

0041766C   .  8B55 D0       MOV EDX,DWORD PTR SS:[EBP-30]

0041766F   .  64:8915 00000>MOV DWORD PTR FS:[0],EDX

00417676   .  EB 65         JMP SHORT 《真正的.004176DD

00417678   >  66:C745 E0 14>MOV WORD PTR SS:[EBP-20],14              ;

0041767E   .  EB 32         JMP SHORT 《真正的.004176B2

00417680   .  33C0          XOR EAX,EAX

00417682   .  BA 02000000   MOV EDX,2

00417687   .  50            PUSH EAX

00417688   .  8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]

0041768B   .  FF4D EC       DEC DWORD PTR SS:[EBP-14]

0041768E   .  E8 7DA90800   CALL 《真正的.004A2010

00417693   .  58            POP EAX

00417694   .  50            PUSH EAX

00417695   .  33D2          XOR EDX,EDX

00417697   .  8955 EC       MOV DWORD PTR SS:[EBP-14],EDX

0041769A   .  8D4D D0       LEA ECX,DWORD PTR DS:[EBP-30]

0041769D   .  51            PUSH ECX

0041769E   .  E8 20800800   CALL 《真正的.0049F6C3

004176A3   .  59            POP ECX

004176A4   .  58            POP EAX

004176A5   .  EB 36         JMP SHORT 《真正的.004176DD

004176A7   .  66:C745 E0 4C>MOV WORD PTR SS:[EBP-20],4C

004176AD   .  E8 5C790800   CALL 《真正的.0049F00E

004176B2   >  8B55 08       MOV EDX,DWORD PTR SS:[EBP+8]

004176B5   .  B0 01         MOV AL,1                                 ;  来到这里AL=1，

004176B7   .  C782 A8030000>MOV DWORD PTR DS:[EDX+3A8],1

004176C1   .  50            PUSH EAX

004176C2   .  8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]             ;

004176C5   .  FF4D EC       DEC DWORD PTR SS:[EBP-14]                ;

004176C8   .  BA 02000000   MOV EDX,2

004176CD   .  E8 3EA90800   CALL 《真正的.004A2010

004176D2   .  58            POP EAX                                  ;

004176D3   .  8B55 D0       MOV EDX,DWORD PTR SS:[EBP-30]

004176D6   .  64:8915 00000>MOV DWORD PTR FS:[0],EDX

004176DD   >  5F            POP EDI

004176DE   .  5E            POP ESI

004176DF   .  5B            POP EBX

004176E0   .  8BE5          MOV ESP,EBP

004176E2   .  5D            POP EBP

004176E3   .  C3            RETN                                        到这里这个CALL成功返回AL=1，顺利进入下面的CALL

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

CALL 《真正的.00402D98   进入这个CALL看如何计算注册码前5位值

```
00402DA8  /$  55            PUSH EBP
00402DA9  |.  8BEC          MOV EBP,ESP
00402DAB  |.  81C4 28FDFFFF ADD ESP,-2D8                             ;
00402DB1  |.  B8 1CC34B00   MOV EAX,《真正的.004BC31C
00402DB6  |.  53            PUSH EBX
00402DB7  |.  56            PUSH ESI
00402DB8  |.  57            PUSH EDI
00402DB9  |.  8B75 08       MOV ESI,DWORD PTR SS:[EBP+8]
00402DBC  |.  E8 F7140900   CALL 《真正的.004942B8
00402DC1  |.  66:C745 E4 08>MOV WORD PTR SS:[EBP-1C],8               ;
00402DC7  |.  33D2          XOR EDX,EDX
00402DC9  |.  8955 FC       MOV DWORD PTR SS:[EBP-4],EDX             ;
00402DCC  |.  FF45 F0       INC DWORD PTR SS:[EBP-10]                ;
00402DCF  |.  66:C745 E4 14>MOV WORD PTR SS:[EBP-1C],14              ;
00402DD5  |.  0FBE8E E50300>MOVSX ECX,BYTE PTR DS:[ESI+3E5]          ;
00402DDC  |.  894D C4       MOV DWORD PTR SS:[EBP-3C],ECX            ;
00402DDF  |.  0FBE86 E60300>MOVSX EAX,BYTE PTR DS:[ESI+3E6]          ;
00402DE6  |.  8945 C0       MOV DWORD PTR SS:[EBP-40],EAX            ;
00402DE9  |.  0FBEBE E70300>MOVSX EDI,BYTE PTR DS:[ESI+3E7]          ;
00402DF0  |.  0FBE86 E80300>MOVSX EAX,BYTE PTR DS:[ESI+3E8]          ;
00402DF7  |.  8945 BC       MOV DWORD PTR SS:[EBP-44],EAX            ;
00402DFA  |.  0FBE96 E90300>MOVSX EDX,BYTE PTR DS:[ESI+3E9]          ;
00402E01  |.  8955 B8       MOV DWORD PTR SS:[EBP-48],EDX            ;
00402E04  |.  0FBE8E EB0300>MOVSX ECX,BYTE PTR DS:[ESI+3EB]          ;
00402E0B  |.  894D B4       MOV DWORD PTR SS:[EBP-4C],ECX            ;
00402E0E  |.  8B45 C4       MOV EAX,DWORD PTR SS:[EBP-3C]            ;
00402E11  |.  8B55 B4       MOV EDX,DWORD PTR SS:[EBP-4C]            ;
00402E14  |.  0345 C0       ADD EAX,DWORD PTR SS:[EBP-40]            ;
00402E17  |.  03C7          ADD EAX,EDI                              ;
00402E19  |.  0345 BC       ADD EAX,DWORD PTR SS:[EBP-44]
00402E1C  |.  0345 B8       ADD EAX,DWORD PTR SS:[EBP-48]            ;
00402E1F  |.  C1E2 03       SHL EDX,3                                ;
00402E22  |.  03C2          ADD EAX,EDX                              ;
00402E24  |.  33DB          XOR EBX,EBX
00402E26  |.  8945 B0       MOV DWORD PTR SS:[EBP-50],EAX            ;
00402E29  |>  8D04DB        /LEA EAX,DWORD PTR DS:[EBX+EBX*8]
00402E2C  |.  B9 5A000000   |MOV ECX,5A
00402E31  |.  99            |CDQ
00402E32  |.  F77D C0       |IDIV DWORD PTR SS:[EBP-40]              ;
00402E35  |.  0155 B0       |ADD DWORD PTR SS:[EBP-50],EDX           ;
00402E38  |.  8BC3          |MOV EAX,EBX
00402E3A  |.  99            |CDQ
00402E3B  |.  F7F9          |IDIV ECX                                ;
00402E3D  |.  8A45 B0       |MOV AL,BYTE PTR SS:[EBP-50]             ;
00402E40  |.  81FB 3F1F0000 |CMP EBX,1F3F
00402E46  |.  888415 30FDFF>|MOV BYTE PTR SS:[EBP+EDX-2D0],AL        ;
00402E4D  |.  75 2A         |JNZ SHORT 《真正的.00402E79             ;
00402E4F  |.  DB45 C4       |FILD DWORD PTR SS:[EBP-3C]
00402E52  |.  DC0D 0C344000 |FMUL QWORD PTR DS:[40340C]
00402E58  |.  8B45 B0       |MOV EAX,DWORD PTR SS:[EBP-50]           ;
00402E5B  |.  B9 64000000   |MOV ECX,64
00402E60  |.  99            |CDQ
00402E61  |.  F7F9          |IDIV ECX
00402E63  |.  8985 2CFDFFFF |MOV DWORD PTR SS:[EBP-2D4],EAX
00402E69  |.  DB85 2CFDFFFF |FILD DWORD PTR SS:[EBP-2D4]
00402E6F  |.  DEC1          |FADDP ST(1),ST
00402E71  |.  E8 1E810900   |CALL 《真正的.0049AF94                  ;
00402E76  |.  8945 B8       |MOV DWORD PTR SS:[EBP-48],EAX           ;
00402E79  |>  0FBE96 E70300>|MOVSX EDX,BYTE PTR DS:[ESI+3E7]         ;
00402E80  |.  8D0C92        |LEA ECX,DWORD PTR DS:[EDX+EDX*4]        ;
00402E83  |.  8D0C4A        |LEA ECX,DWORD PTR DS:[EDX+ECX*2]        ;
00402E86  |.  81C1 70110100 |ADD ECX,11170                           ;
00402E8C  |.  3BD9          |CMP EBX,ECX
00402E8E  |.  75 4B         |JNZ SHORT 《真正的.00402EDB             ;
00402E90  |.  DB45 C4       |FILD DWORD PTR SS:[EBP-3C]
00402E93  |.  DC0D 0C344000 |FMUL QWORD PTR DS:[40340C]
00402E99  |.  8B45 B0       |MOV EAX,DWORD PTR SS:[EBP-50]           ;
00402E9C  |.  B9 64000000   |MOV ECX,64
00402EA1  |.  99            |CDQ
00402EA2  |.  F7F9          |IDIV ECX
00402EA4  |.  8985 2CFDFFFF |MOV DWORD PTR SS:[EBP-2D4],EAX
00402EAA  |.  DB85 2CFDFFFF |FILD DWORD PTR SS:[EBP-2D4]
00402EB0  |.  DEC1          |FADDP ST(1),ST
00402EB2  |.  D825 14344000 |FSUB DWORD PTR DS:[403414]
00402EB8  |.  0FBE86 E70300>|MOVSX EAX,BYTE PTR DS:[ESI+3E7]         ;
00402EBF  |.  8D1480        |LEA EDX,DWORD PTR DS:[EAX+EAX*4]        ;
00402EC2  |.  8D1450        |LEA EDX,DWORD PTR DS:[EAX+EDX*2]        ;
00402EC5  |.  8995 28FDFFFF |MOV DWORD PTR SS:[EBP-2D8],EDX
00402ECB  |.  DB85 28FDFFFF |FILD DWORD PTR SS:[EBP-2D8]
00402ED1  |.  DEC1          |FADDP ST(1),ST
00402ED3  |.  E8 BC800900   |CALL 《真正的.0049AF94                  ;
00402ED8  |.  8945 BC       |MOV DWORD PTR SS:[EBP-44],EAX           ;
00402EDB  |>  0FBE96 E70300>|MOVSX EDX,BYTE PTR DS:[ESI+3E7]         ;
```
