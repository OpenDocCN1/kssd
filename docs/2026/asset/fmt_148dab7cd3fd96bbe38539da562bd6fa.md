
这个就是算法，正确的流程大致是这样的：

1. 首先检查用户名长度和注册码长度，等于0函数就退了。

2. 转换注册码格式：从字符串转换成数值，貌似只能是0-9 a-f之间的字符。。。

注册码：`1234-5678-9abc-deff`

转换之后 `12 34 56 78 9a bc de ff 00 00`

程序中好像是一个10字节的数组来存放转换后的数值的。假设数组是

`char szPass[10]={0x12,0x34,0x56,0x78,0x9a,0xbc,0xff,0,0}`

3. 接着先比较`szPass[3]`是不是`0x9c`不是就比较是否是`FC`和`AC`（后边两种情况略过。）

4. 对注册码做运算得到两个值：

`Num1=((szPass[6]^szPass[0])^0x18)+0x3d)^a7;`

`Num2=(((((szPass[7]^szPass[1])\*100 +szPass[5]^szPass[2]) ^ 0x7892) + 0x4D30) ^ 0x3421)`

`Num2 %11==0 ? Num2=Num2/11 : Num2=0`

接着就判断`Num1`是否为0 （正确流程不为0）

`Num`是否等于0 是否大于`0x3E8` 正确流程应该是在`[1,1000）`之间。

5. 接着调用 `sub_004028EC` （`jmp 0065C500`）

该函数的作用是通过用户名计算出来一个值。

```
0065C500                   > \8B4424 04     MOV EAX,DWORD PTR SS:[ESP+0x4]        ;  参数1 用户名

0065C504                   .  83EC 0C       SUB ESP,0xC

0065C507                   .  53            PUSH EBX

0065C508                   .  55            PUSH EBP

0065C509                   .  33DB          XOR EBX,EBX

0065C50B                   .  33ED          XOR EBP,EBP

0065C50D                   .  8D50 01       LEA EDX,DWORD PTR DS:[EAX+0x1]

0065C510                   >  8A08          MOV CL,BYTE PTR DS:[EAX]

0065C512                   .  40            INC EAX

0065C513                   .  84C9          TEST CL,CL

0065C515                   .^ 75 F9         JNZ SHORT 010Edito.0065C510

0065C517                   .  2BC2          SUB EAX,EDX                           ;  求用户名长度

0065C519                   .  3BC3          CMP EAX,EBX

0065C51B                   .  894424 10     MOV DWORD PTR SS:[ESP+0x10],EAX

0065C51F                   .  0F8E 09010000 JLE 010Edito.0065C62E

0065C525                   .  8B4424 24     MOV EAX,DWORD PTR SS:[ESP+0x24]       ;  参数4

0065C529                   .  56            PUSH ESI

0065C52A                   .  57            PUSH EDI

0065C52B                   .  8BF8          MOV EDI,EAX

0065C52D                   .  C1E7 04       SHL EDI,0x4

0065C530                   .  2BF8          SUB EDI,EAX                           ;  edi=参数4<<4-参数4

0065C532                   .  8B4424 28     MOV EAX,DWORD PTR SS:[ESP+0x28]       ;  参数3

0065C536                   .  8BF0          MOV ESI,EAX

0065C538                   .  C1E6 04       SHL ESI,0x4

0065C53B                   .  895C24 14     MOV DWORD PTR SS:[ESP+0x14],EBX

0065C53F                   .  895C24 10     MOV DWORD PTR SS:[ESP+0x10],EBX

0065C543                   .  03F0          ADD ESI,EAX                           ;  esi=参数3<<4 +参数3

0065C545                   >  8B4424 20     MOV EAX,DWORD PTR SS:[ESP+0x20]       ;  参数1 字符串

0065C549                   .  0FB60C03      MOVZX ECX,BYTE PTR DS:[EBX+EAX]       ;  ebx计数器 初值0

0065C54D                   .  51            PUSH ECX                              ; /c

0065C54E                   .  FF15 60908B00 CALL DWORD PTR DS:[<&MSVCR90.toupper>>; \toupper

0065C554                   .  83C4 04       ADD ESP,0x4

0065C557                   .  837C24 24 00  CMP DWORD PTR SS:[ESP+0x24],0x0       ;  参数2 isNotFc

0065C55C                   .  74 5B         JE SHORT 010Edito.0065C5B9

0065C55E                   .  8B0C85 F0B58A>MOV ECX,DWORD PTR DS:[EAX*4+0x8AB5F0]

0065C565                   .  8D50 0D       LEA EDX,DWORD PTR DS:[EAX+0xD]

0065C568                   .  81E2 FF000000 AND EDX,0xFF

0065C56E                   .  03CD          ADD ECX,EBP

0065C570                   .  330C95 F0B58A>XOR ECX,DWORD PTR DS:[EDX*4+0x8AB5F0]

0065C577                   .  83C0 2F       ADD EAX,0x2F

0065C57A                   .  25 FF000000   AND EAX,0xFF

0065C57F                   .  0FAF0C85 F0B5>IMUL ECX,DWORD PTR DS:[EAX*4+0x8AB5F0>

0065C587                   .  8BD6          MOV EDX,ESI

0065C589                   .  81E2 FF000000 AND EDX,0xFF

0065C58F                   .  030C95 F0B58A>ADD ECX,DWORD PTR DS:[EDX*4+0x8AB5F0]

0065C596                   .  8B5424 10     MOV EDX,DWORD PTR SS:[ESP+0x10]

0065C59A                   .  8BC7          MOV EAX,EDI

0065C59C                   .  25 FF000000   AND EAX,0xFF

0065C5A1                   .  030C85 F0B58A>ADD ECX,DWORD PTR DS:[EAX*4+0x8AB5F0]

0065C5A8                   .  81E2 FF000000 AND EDX,0xFF

0065C5AE                   .  030C95 F0B58A>ADD ECX,DWORD PTR DS:[EDX*4+0x8AB5F0]

0065C5B5                   .  8BE9          MOV EBP,ECX

0065C5B7                   .  EB 58         JMP SHORT 010Edito.0065C611

0065C5B9                   >  8B1485 F0B58A>MOV EDX,DWORD PTR DS:[EAX*4+0x8AB5F0]

0065C5C0                   .  8D48 3F       LEA ECX,DWORD PTR DS:[EAX+0x3F]

0065C5C3                   .  03D5          ADD EDX,EBP

0065C5C5                   .  83C0 17       ADD EAX,0x17

0065C5C8                   .  81E1 FF000000 AND ECX,0xFF

0065C5CE                   .  33148D F0B58A>XOR EDX,DWORD PTR DS:[ECX*4+0x8AB5F0]

0065C5D5                   .  25 FF000000   AND EAX,0xFF

0065C5DA                   .  0FAF1485 F0B5>IMUL EDX,DWORD PTR DS:[EAX*4+0x8AB5F0>

0065C5E2                   .  8BC6          MOV EAX,ESI

0065C5E4                   .  25 FF000000   AND EAX,0xFF

0065C5E9                   .  031485 F0B58A>ADD EDX,DWORD PTR DS:[EAX*4+0x8AB5F0]

0065C5F0                   .  8B4424 14     MOV EAX,DWORD PTR SS:[ESP+0x14]

0065C5F4                   .  8BCF          MOV ECX,EDI

0065C5F6                   .  81E1 FF000000 AND ECX,0xFF

0065C5FC                   .  03148D F0B58A>ADD EDX,DWORD PTR DS:[ECX*4+0x8AB5F0]

0065C603                   .  25 FF000000   AND EAX,0xFF

0065C608                   .  031485 F0B58A>ADD EDX,DWORD PTR DS:[EAX*4+0x8AB5F0]

0065C60F                   .  8BEA          MOV EBP,EDX

0065C611                   >  834424 10 13  ADD DWORD PTR SS:[ESP+0x10],0x13

0065C616                   .  834424 14 07  ADD DWORD PTR SS:[ESP+0x14],0x7

0065C61B                   .  43            INC EBX

0065C61C                   .  83C6 09       ADD ESI,0x9

0065C61F                   .  83C7 0D       ADD EDI,0xD

0065C622                   .  3B5C24 18     CMP EBX,DWORD PTR SS:[ESP+0x18]

0065C626                   .^ 0F8C 19FFFFFF JL 010Edito.0065C545

0065C62C                   .  5F            POP EDI

0065C62D                   .  5E            POP ESI

0065C62E                   >  8BC5          MOV EAX,EBP

0065C630                   .  5D            POP EBP

0065C631                   .  5B            POP EBX

0065C632                   .  83C4 0C       ADD ESP,0xC

0065C635                   .  C3            RETN
```

该函数有四个参数：

参数1：`【char* 类型】`用户名字符串，不用多说

参数2：`【int 类型】`根据`szPass[3]`的值决定。如果`szPass[3]=0xFC` 这个值是0 否则1

参数3：`【BYTE 类型】`这个可能是软件版本号。根据`Num1`得来，程序中只要`Num1>2` 这个就是0，下边还有判断`Num1`的地方要求`Num1`必须大于7

> 1.——————————————————————————

> `0065E158 . 83F9 02 CMP ECX,0x2 ; ECX=((byte6^byte0)^18)+3d)^a7`

> `0065E15B . 1BFF SBB EDI,EDI ; CF位 EDI只能是0或者-1`

> `0065E15D . 23F9 AND EDI,ECX ; ecx>=2 edi=0 ecx<2 edi=ecx`

> 2.——————————————————————————

> `0065E2E6 . 3B46 1C CMP EAX,DWORD PTR DS:[ESI+0x1C] ; ((byte6^byte0)^18)+3d)^a7 必须大于7 这个是用来判断版本的`

参数4：`【BYTE/WORD类型】`类型有点不确定。这个值跟用户数量有关。

传入的就是`Num2`的值。

这个函数对用的`C`代码：
