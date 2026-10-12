
# 黑客字典 II

> 原文：[`www.pediy.com/kssd/pediy06/pediy6221.htm`](https://www.pediy.com/kssd/pediy06/pediy6221.htm)

目标：黑客字典 II（限制版）

工具：`olly`

作者：`FTBirthday`

软件说明：黑客字典 II 是在黑客字典 V0.9 发布之后，在听取广大网友意见的基础上重写而成的。

她几乎可以生成任何形式的密码组合，可用于产生密码文件，以便和一些解密软件配合使用。

如：网络刺客、`John The Ripper`、`Jack`、`CrackZip`、`CrackArj` 等。本软件是免费的，您可以自由传播。----------小榕软件实验室

注册说明：本软件是一个共享软件，如果您需要使用它的全部功能，测需要进行注册。这种注册是免费的。您只要发一封标题为“注册”的电子邮件到 `Assassin@ynmail.com` 即可收到以同样方式发回的注册码。

用 `olly` 载入，点击注册，输入

用户名 `"FTBirthday"`，注册码 `"510510510"`

设断 `GetWindowTextA`

中断于

```
0012ECC8   0042002E   /CALL to GetWindowTextA from UltraDic.00420028
0012ECCC   0045022E   |hWnd = 0045022E (class='Edit',parent=00100232)
0012ECD0   0012ED04   |Buffer = 0012ED04
0012ECD4   00000064   Count = 64 (100.)
```

最终返回到

```
00420028  .  FF15 C0B34200   CALL DWORD PTR DS:[<&USER32.GetWindowTex>; GetWindowTextA
0042002E  .  EB 12           JMP SHORT UltraDic.00420042
```

----------------------------------------------------------------

关键代码段

----------------------------------------------------------------

```
00401BF5   .  E8 1CE40100     CALL UltraDic.00420016
00401BFA   .  8D7C24 20       LEA EDI,DWORD PTR SS:[ESP+20]         ;  DWORD PTR SS:[ESP+20]="FTBirthday"
00401BFE   .  83C9 FF         OR ECX,FFFFFFFF
00401C01   .  33C0            XOR EAX,EAX                           ;  EAX 为用户名的长度
00401C03   .  F2:AE           REPNE SCAS BYTE PTR ES:[EDI]
00401C05   .  F7D1            NOT ECX                                ;  ECX=用户名的长度+1 取反
00401C07   .  49              DEC ECX                                ;  ECX=ECX-1=0000000A,判断用户名是否为空
00401C08   .  0F84 16020000   JE UltraDic.00401E24
00401C0E   .  8DBC24 8400000> LEA EDI,DWORD PTR SS:[ESP+84]         ;  DWORD PTR SS:[ESP+84]="510510510"
00401C15   .  83C9 FF         OR ECX,FFFFFFFF
00401C18   .  F2:AE           REPNE SCAS BYTE PTR ES:[EDI]
00401C1A   .  F7D1            NOT ECX                                ;  ECX=注册码的长度+1 取反
00401C1C   .  49              DEC ECX                                ;  ECX=00000009,判断注册码是否为空
00401C1D   .  0F84 01020000   JE UltraDic.00401E24
```

下面开始对用户名进行运算得出一个相应的字符串

```
00401C23   .  8D7C24 20       LEA EDI,DWORD PTR SS:[ESP+20]         ;  DWORD PTR SS:[ESP+20]="FTBirthday"
00401C27   .  83C9 FF         OR ECX,FFFFFFFF
00401C2A   .  33F6            XOR ESI,ESI                           ;  ESI 清零
00401C2C   .  F2:AE           REPNE SCAS BYTE PTR ES:[EDI]
00401C2E   .  F7D1            NOT ECX                                ;  ECX=用户名的长度+1 取反
00401C30   .  49              DEC ECX                                ;  ECX=ECX-1=0000000A,判断用户名是否为空
00401C31   .  74 79           JE SHORT UltraDic.00401CAC
00401C33   >  0FBE7C34 20     MOVSX EDI,BYTE PTR SS:[ESP+ESI+20]    ;  取用户名第 i 个字符，符号扩展给 EDI
00401C38   .  8BC7            MOV EAX,EDI                           ;  EAX=EDI=用户名第 i 个字符
00401C3A   .  B9 0A000000     MOV ECX,0A
00401C3F   .  99              CDQ                                   ;  把 EDX 清零
00401C40   .  F7F9            IDIV ECX                              ;  EAX/ECX,EAX 放商，EDX 放余数
00401C42   .  8BCA            MOV ECX,EDX
00401C44   .  81E2 01000080   AND EDX,80000001
00401C4A   .  79 05          JNS SHORT UltraDic.00401C51           ;  非负数则跳
00401C4C   .  4A             DEC EDX
00401C4D   .  83CA FE        OR EDX,FFFFFFFE
00401C50   .  42             INC EDX
00401C51   >  75 16          JNZ SHORT UltraDic.00401C69           ;  不为零则跳
00401C53   .  8BC7            MOV EAX,EDI                           ;  EAX=EDI=用户名第 i 个字符
00401C55   .  B9 1A000000     MOV ECX,1A
00401C5A   .  99              CDQ
00401C5B   .  F7F9            IDIV ECX                              ;  EAX/ECX,EAX 放商，EDX 放余数
00401C5D   .  80C2 41         ADD DL,41                             ;  余数低八位 DL+41
00401C60   .  889434 4801000> MOV BYTE PTR SS:[ESP+ESI+148],DL      ;  DL=53="S"
00401C67   .  EB 2E           JMP SHORT UltraDic.00401C97
00401C69   >  8BC1            MOV EAX,ECX                           ;  余数不为零时把余数给 EAX
00401C6B   .  BB 03000000     MOV EBX,3
00401C70   .  99              CDQ                                   ;  把 EDX 清零
00401C71   .  F7FB            IDIV EBX                              ;  EAX/EBX,EAX 放商，EDX 放余数
00401C73   .  85D2            TEST EDX,EDX                          ;  看看第二次,余数是否为零
00401C75     75 16          JNZ SHORT UltraDic.00401C8D           ;  不为零则跳
00401C77   .  8BC7            MOV EAX,EDI                           ;  EAX=EDI=用户名第 i 个字符
00401C79   .  B9 1A000000     MOV ECX,1A
00401C7E   .  99              CDQ
00401C7F   .  F7F9            IDIV ECX                              ;  EAX/ECX,EAX 放商，EDX 放余数
00401C81   .  80C2 61         ADD DL,61                             ;  余数低八位 DL+61
00401C84   .  889434 4801000> MOV BYTE PTR SS:[ESP+ESI+148],CL
00401C8B   .  EB 0A           JMP SHORT UltraDic.00401C97
00401C8D   >  80C1 31         ADD CL,31                             ;  余数低八位 CL+31
00401C90   .  888C34 4801000> MOV BYTE PTR SS:[ESP+ESI+148],CL
00401C97   >  8D7C24 20       LEA EDI,DWORD PTR SS:[ESP+20]         ;  DWORD PTR SS:[ESP+20]="FTBirthday"
00401C9B   .  83C9 FF         OR ECX,FFFFFFFF
00401C9E   .  33C0            XOR EAX,EAX
00401CA0   .  46              INC ESI                               ;  取字符计数器
00401CA1   .  F2:AE           REPNE SCAS BYTE PTR ES:[EDI]
00401CA3   .  F7D1            NOT ECX                                ;  ECX=用户名的长度+1 取反
00401CA5   .  49              DEC ECX                                ;  ECX=ECX-1=0000000A,用户名长度
00401CA6   .  3BF1            CMP ESI,ECX                           ;  比较是否已取完用户名所有字符
00401CA8   .  72 89           JB SHORT UltraDic.00401C33            ;  小于则跳回循环
```

下面算出的字符串将和输入的注册码进行比较

```
00401CAA   .  33DB            XOR EBX,EBX
00401CAC   >  55              PUSH EBP
00401CAD   .  8D8C24 EC00000> LEA ECX,DWORD PTR SS:[ESP+EC]
00401CB4   .  889C34 4C01000> MOV BYTE PTR SS:[ESP+ESI+14C],BL
00401CBB   .  E8 F0010000     CALL UltraDic.00401EB0
00401CC0   .  899C24 B401000> MOV DWORD PTR SS:[ESP+1B4],EBX
00401CC7   .  8DB424 8400000> LEA ESI,DWORD PTR SS:[ESP+84]         ;  ESI=注册码
00401CCE   .  8D8424 4801000> LEA EAX,DWORD PTR SS:[ESP+148]        ;  EAX=用户名变换后的字符
00401CD5   >  8A10            MOV DL,BYTE PTR DS:[EAX]              ;  用户名变换后的第 i 个字符给 DL
00401CD7   .  8ACA            MOV CL,DL
00401CD9   .  3A16            CMP DL,BYTE PTR DS:[ESI]              ;  和注册码的第 i 个字符比较
00401CDB   .  75 1C           JNZ SHORT UltraDic.00401CF9
00401CDD   .  3ACB            CMP CL,BL                             ;  比较是否为空
00401CDF   .  74 14           JE SHORT UltraDic.00401CF5
00401CE1   .  8A50 01         MOV DL,BYTE PTR DS:[EAX+1]            ;  用户名变换后的第 i+1 个字符给 DL
00401CE4   .  8ACA            MOV CL,DL
00401CE6   .  3A56 01         CMP DL,BYTE PTR DS:[ESI+1]            ;  和注册码的第 i+1 个字符比较
00401CE9   .  75 0E          JNZ SHORT UltraDic.00401CF9
00401CEB   .  83C0 02         ADD EAX,2
00401CEE   .  83C6 02         ADD ESI,2
00401CF1   .  3ACB            CMP CL,BL                             ;  比较是否为空 BL
00401CF3   .  75 E0           JNZ SHORT UltraDic.00401CD5           ;  不为零则跳回继续比较
```

至此，比较完成，下面是将注册信息写到 `"C: egbanyet.dat"` 文件中

```
00401CF5   >  33C0            XOR EAX,EAX
00401CF7   .  EB 05           JMP SHORT UltraDic.00401CFE
00401CF9   >  1BC0            SBB EAX,EAX
00401CFB   .  83D8 FF         SBB EAX,-1
00401CFE   >  3BC3            CMP EAX,EBX
00401D00   .  0F85 DF000000   JNZ UltraDic.00401DE5
00401D06   .  68 01100000     PUSH 1001
```



```
00401D0B   . 68 F4704300    PUSH UltraDic.004370F4                  ;  ASCII "C: egbanyet.dat"

00401D10   . 8D4C24 18      LEA ECX,DWORD PTR SS:[ESP+18]

00401D14   . E8 8FEF0100    CALL UltraDic.00420CA8

00401D19   . 8D7C24 20      LEA EDI,DWORD PTR SS:[ESP+20]

00401D1D   . 83C9 FF        OR ECX,FFFFFFFF

00401D20   . 33C0           XOR EAX,EAX

00401D22   . B2 0D          MOV DL,0D

00401D24   . F2:AE          REPNE SCAS BYTE PTR ES:[EDI]

00401D26   . F7D1           NOT ECX

00401D28   . 49             DEC ECX

00401D29   . 8DBC24 8400000>LEA EDI,DWORD PTR SS:[ESP+84]

00401D30   . C68424 B401000>MOV BYTE PTR SS:[ESP+1B4],1

00401D38   . 88540C 20      MOV BYTE PTR SS:[ESP+ECX+20],DL

00401D3C   . 41             INC ECX

00401D3D   . C6440C 20 0A   MOV BYTE PTR SS:[ESP+ECX+20],0A

00401D42   . 885C0C 21      MOV BYTE PTR SS:[ESP+ECX+21],BL

00401D46   . 83C9 FF        OR ECX,FFFFFFFF

00401D49   . F2:AE          REPNE SCAS BYTE PTR ES:[EDI]

00401D4B   . F7D1           NOT ECX

00401D4D   . 49             DEC ECX

00401D4E   . 8D7C24 20      LEA EDI,DWORD PTR SS:[ESP+20]

00401D52   . 88940C 8400000>MOV BYTE PTR SS:[ESP+ECX+84],DL

00401D59   . 41             INC ECX

00401D5A   . C6840C 8400000>MOV BYTE PTR SS:[ESP+ECX+84],0A

00401D62   . 889C0C 8500000>MOV BYTE PTR SS:[ESP+ECX+85],BL

00401D69   . 83C9 FF        OR ECX,FFFFFFFF

00401D6C   . F2:AE          REPNE SCAS BYTE PTR ES:[EDI]

00401D6E   . F7D1           NOT ECX

00401D70   . 49             DEC ECX

00401D71   . 8D4424 20      LEA EAX,DWORD PTR SS:[ESP+20]         ; 用户名给 EAX

00401D75   . 51             PUSH ECX                                 ; /Arg2

00401D76   . 50             PUSH EAX                                 ; |Arg1

00401D77   . 8D4C24 18      LEA ECX,DWORD PTR SS:[ESP+18]         ; |

00401D7B   . E8 DCF10100    CALL UltraDic.00420F5C                 ; UltraDic.00420F5C

00401D80   . 8DBC24 8400000>LEA EDI,DWORD PTR SS:[ESP+84]

00401D87   . 83C9 FF        OR ECX,FFFFFFFF

00401D8A   . 33C0           XOR EAX,EAX

00401D8C   . F2:AE          REPNE SCAS BYTE PTR ES:[EDI]

00401D8E   . F7D1           NOT ECX

00401D90   . 49             DEC ECX

00401D91   . 51             PUSH ECX                                 ; /Arg2

00401D92   . 8D8C24 8800000>LEA ECX,DWORD PTR SS:[ESP+88]         ; |

00401D99   . 51             PUSH ECX                                 ; |Arg1

00401D9A   . 8D4C24 18      LEA ECX,DWORD PTR SS:[ESP+18]         ; |

00401D9E   . E8 B9F10100    CALL UltraDic.00420F5C                 ; UltraDic.00420F5C

00401DA3   . 6A 03          PUSH 3                                   ; /FileAttributes = READONLY|HIDDEN

00401DA5   . 68 F4704300    PUSH UltraDic.004370F4                 ; |FileName = "C: egbanyet.dat"

00401DAA   . FF15 8CB24200  CALL DWORD PTR DS:[<&KERNEL32.SetFileAtt>; SetFileAttributesA

00401DB0   . 68 D8704300    PUSH UltraDic.004370D8

00401DB5   . 8D8C24 EC00000>LEA ECX,DWORD PTR SS:[ESP+EC]

00401DBC   . E8 AF010000    CALL UltraDic.00401F70                 ***[到这儿弹出注册成功信息]***

00401DC1   . 8D4C24 10      LEA ECX,DWORD PTR SS:[ESP+10]

00401DC5   . 889C24 B401000>MOV BYTE PTR SS:[ESP+1B4],BL

00401DCC   . E8 77EF0100    CALL UltraDic.00420D48

00401DD1   . 8BCD           MOV ECX,EBP

00401DD3   . E8 44B50100    CALL UltraDic.0041D31C

00401DD8   . C78424 B401000>MOV DWORD PTR SS:[ESP+1B4],3

00401DE3   . EB 1C          JMP SHORT UltraDic.00401E01

00401DE5   > 68 C0704300    PUSH UltraDic.004370C0

00401DEA   . 8D8C24 EC00000>LEA ECX,DWORD PTR SS:[ESP+EC]

00401DF1   . E8 7A010000    CALL UltraDic.00401F70

00401DF6   . C78424 B401000>MOV DWORD PTR SS:[ESP+1B4],2

00401E01   > 8D8C24 4401000>LEA ECX,DWORD PTR SS:[ESP+144]

00401E08   . E8 CCEB0100    CALL UltraDic.004209D9

00401E0D   . 8D8C24 E800000>LEA ECX,DWORD PTR SS:[ESP+E8]

00401E14   . C78424 B401000>MOV DWORD PTR SS:[ESP+1B4],-1

00401E1F   . E8 43AE0100    CALL UltraDic.0041CC67

00401E24   > 8B8C24 AC01000>MOV ECX,DWORD PTR SS:[ESP+1AC]        ;  注册名为空则直接来到这儿

00401E2B   . 5F             POP EDI

00401E2C   . 5E             POP ESI

00401E2D   . 5D             POP EBP

00401E2E   . 5B             POP EBX

00401E2F   . 64:890D 000000>MOV DWORD PTR FS:[0],ECX

00401E36   . 81C4 A8010000  ADD ESP,1A8

00401E3C   . C3             RETN
```

最后把用户名和注册码,放入到`C: egbanyet.dat`文件中，属性是只读，隐藏。

### 总结

**注册机制:**

由用户名算出一个注册码并与输入的注册码比较,并且这个真码出现在内存中,
用户名长度=注册码长度

**注册算法模拟:**

设用户名字符对应 ASCII 为 `a[i]`,相应字符串为 `b[i]`

`t1=a[i]%A;`

```
if (t1!=0)
   {t2=t1%3;
    if (t2!=0) b[i]=t1+31;
    else       b[i]=a[i]%1A+31;}
else b[i]=a[i]%1A+41;
```

于是有

用户名:`FTBirthday`
注册码:`SGO6KMAW82`
`4dd466d0e1e0cc123afbe504da2462ca`
