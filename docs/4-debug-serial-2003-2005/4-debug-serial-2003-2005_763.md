
# My Notes Keeper V1.4 注册算法分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-751.htm`](https://www.pediy.com/kssd/pediy07/pediy7-751.htm)

- 【破文作者】lnn1123
- 【作者主页】http://blog.csdn.net/lnn1123
- 【E-mail】lnn11231123@163.com
- 【文章题目】My Notes Keeper V1.4 注册算法分析
- 【软件名称】My Notes Keeper V1.4
- 【下载地址】天空软件
- 【加密方式】注册码
- 【加壳方式】`aspack`
- 【破解工具】`PEID`, `OLLYDBG`
- 【破解平台】`WIN200` AND `WINXP`

=======================================================================================================

## 软件简介

`My Notes Keeper` 是一款功能强大、简单易用的树状标签结构个人数据库管理软件，能进行个人信息管理和文字、表格处理，有密码保护功能。你可以通过它管理你的通讯簿、网址收藏和安排日程表等，甚至可以用它来制作电子书。软件的操作方式跟 `Word` 几乎没有多大区别，推荐使用！

=======================================================================================================

## 文章简介

要过年了，在看雪也混了一年了，没学到什么技术惭愧！，写一篇文章安慰一下自己

=======================================================================================================

## 解密过程

`PEID` 发现是 `ASPACK`，不想脱直接调试了，找到 `user32.dll` 的 `messageboxa` 函数并下断，很容易到达这里.

```
00703207   68 66337000      PUSH MyNotesK.00703366
0070320C   64:FF30          PUSH DWORD PTR FS:[EAX]
0070320F   64:8920          MOV DWORD PTR FS:[EAX],ESP
00703212   8D55 FC          LEA EDX,DWORD PTR SS:[EBP-4]
00703215   8B83 00030000    MOV EAX,DWORD PTR DS:[EBX+300]
0070321B   E8 A473D7FF      CALL MyNotesK.0047A5C4                  ; 取注册名，长度返回在 EAX
00703220   837D FC 00       CMP DWORD PTR SS:[EBP-4],0              ; 是否输入
00703224   74 14            JE SHORT MyNotesK.0070323A              ; 不输入就死
00703226   8D55 F8          LEA EDX,DWORD PTR SS:[EBP-8]
00703229   8B83 04030000    MOV EAX,DWORD PTR DS:[EBX+304]
0070322F   E8 9073D7FF      CALL MyNotesK.0047A5C4                  ; 取注册码，长度返回在 EAX
00703234   837D F8 00       CMP DWORD PTR SS:[EBP-8],0              ; 是否输入
00703238   75 5B            JNZ SHORT MyNotesK.00703295             ; 输入就跳了
0070323A   6A 10            PUSH 10
0070323C   68 74337000      PUSH MyNotesK.00703374                  ; ASCII "My Notes Keeper"
00703241   8D45 F4          LEA EAX,DWORD PTR SS:[EBP-C]
00703244   50               PUSH EAX
00703245   8D55 F0          LEA EDX,DWORD PTR SS:[EBP-10]
00703248   A1 B8997700      MOV EAX,DWORD PTR DS:[7799B8]
0070324D   E8 DE3BD0FF      CALL MyNotesK.00406E30
00703252   8B55 F0          MOV EDX,DWORD PTR SS:[EBP-10]
00703255   B9 8C337000      MOV ECX,MyNotesK.0070338C               ; ASCII "Reg"
0070325A   B8 98337000      MOV EAX,MyNotesK.00703398               ; ASCII "strBothNameCode"
0070325F   E8 1CCFF1FF      CALL MyNotesK.00620180
00703264   8B45 F4          MOV EAX,DWORD PTR SS:[EBP-C]
00703267   E8 EC1BD0FF      CALL MyNotesK.00404E58
0070326C   50               PUSH EAX
0070326D   8BC3             MOV EAX,EBX
0070326F   E8 70DBD7FF      CALL MyNotesK.00480DE4
00703274   50               PUSH EAX
00703275   E8 024DD0FF      CALL MyNotesK.00407F7C                  ; JMP to user32.MessageBoxA
0070327A   8B83 00030000    MOV EAX,DWORD PTR DS:[EBX+300]
00703280   8B10             MOV EDX,DWORD PTR DS:[EAX]
00703282   FF92 C4000000    CALL DWORD PTR DS:[EDX+C4]
00703288   33C0             XOR EAX,EAX
0070328A   8983 4C020000    MOV DWORD PTR DS:[EBX+24C],EAX
00703290   E9 8F000000      JMP MyNotesK.00703324
00703295   8D55 EC          LEA EDX,DWORD PTR SS:[EBP-14]
00703298   8B83 04030000    MOV EAX,DWORD PTR DS:[EBX+304]
0070329E   E8 2173D7FF      CALL MyNotesK.0047A5C4
007032A3   8B45 EC          MOV EAX,DWORD PTR SS:[EBP-14]          ; 注册码
007032A6   50               PUSH EAX                                ; PUSH 注册码
007032A7   8D55 E8          LEA EDX,DWORD PTR SS:[EBP-18]
007032AA   8B83 00030000    MOV EAX,DWORD PTR DS:[EBX+300]
007032B0   E8 0F73D7FF      CALL MyNotesK.0047A5C4
007032B5   8B45 E8          MOV EAX,DWORD PTR SS:[EBP-18]          ; 注册名
007032B8   5A               POP EDX
007032B9   E8 5ED7F1FF      CALL MyNotesK.00620A1C                  ; 注册验证函数
007032BE   84C0             TEST AL,AL                              ; AL 位返回值，不为 0 就注册成功
007032C0   74 0C            JE SHORT MyNotesK.007032CE
007032C2   C783 4C020000 01>MOV DWORD PTR DS:[EBX+24C],1
007032CC   EB 56            JMP SHORT MyNotesK.00703324
007032CE   6A 10            PUSH 10
007032D0   68 74337000      PUSH MyNotesK.00703374                  ; ASCII "My Notes Keeper"
007032D5   8D45 E4          LEA EAX,DWORD PTR SS:[EBP-1C]
007032D8   50               PUSH EAX
007032D9   8D55 E0          LEA EDX,DWORD PTR SS:[EBP-20]
007032DC   A1 A89E7700      MOV EAX,DWORD PTR DS:[779EA8]
007032E1   E8 4A3BD0FF      CALL MyNotesK.00406E30
007032E6   8B55 E0          MOV EDX,DWORD PTR SS:[EBP-20]
007032E9   B9 8C337000      MOV ECX,MyNotesK.0070338C               ; ASCII "Reg"
007032EE   B8 B0337000      MOV EAX,MyNotesK.007033B0               ; ASCII "strInvalidNameCode"
007032F3   E8 88CEF1FF      CALL MyNotesK.00620180
007032F8   8B45 E4          MOV EAX,DWORD PTR SS:[EBP-1C]
007032FB   E8 581BD0FF      CALL MyNotesK.00404E58
00703300   50               PUSH EAX
00703301   8BC3             MOV EAX,EBX
00703303   E8 DCDAD7FF      CALL MyNotesK.00480DE4
00703308   50               PUSH EAX
00703309   E8 6E4CD0FF      CALL MyNotesK.00407F7C                  ; JMP to user32.MessageBoxA
0070330E   8B83 00030000    MOV EAX,DWORD PTR DS:[EBX+300]
```

==================================`CALL` `MyNotesK.00620A1C`===================================

```
00620A1C   55               PUSH EBP
00620A1D   8BEC             MOV EBP,ESP
00620A1F   83C4 F4          ADD ESP,-0C
00620A22   53               PUSH EBX
00620A23   56               PUSH ESI
00620A24   33C9             XOR ECX,ECX
00620A26   894D FC          MOV DWORD PTR SS:[EBP-4],ECX
00620A29   8BF2             MOV ESI,EDX
00620A2B   8BD8             MOV EBX,EAX
00620A2D   33C0             XOR EAX,EAX
00620A2F   55               PUSH EBP
00620A30   68 A60A6200      PUSH MyNotesK.00620AA6
00620A35   64:FF30          PUSH DWORD PTR FS:[EAX]
00620A38   64:8920          MOV DWORD PTR FS:[EAX],ESP
00620A3B   66:B9 1F00       MOV CX,1F                         ; 常数填充 CX
00620A3F   66:BA 0C00       MOV DX,0C                         ; 常数填充 DX
00620A43   66:B8 0F27       MOV AX,270F                       ; 常数填充 AX
00620A47   E8 D4B9DEFF      CALL MyNotesK.0040C420            ; 进行运算得到一个值，不过跟注册没关系
00620A4C   83C4 F8          ADD ESP,-8
00620A4F   DD1C24           FSTP QWORD PTR SS:[ESP]           ; 浮点数
00620A52   9B               WAIT
00620A53   8D4D F4          LEA ECX,DWORD PTR SS:[EBP-C]
00620A56   B8 80987600      MOV EAX,MyNotesK.00769880
00620A5B   8BD3             MOV EDX,EBX
00620A5D   E8 5A92F2FF      CALL MyNotesK.00549CBC            ; 关键 CALL，注册码产生就在这里
00620A62   8D4D FC          LEA ECX,DWORD PTR SS:[EBP-4]
00620A65   8D45 F4          LEA EAX,DWORD PTR SS:[EBP-C]
00620A68   BA 08000000      MOV EDX,8
00620A6D   E8 DE73F2FF      CALL MyNotesK.00547E50
00620A72   8BD6             MOV EDX,ESI                       ; 假码
00620A74   8B45 FC          MOV EAX,DWORD PTR SS:[EBP-4]      ; 真码
00620A77   E8 908CDEFF      CALL MyNotesK.0040970C            ; 比较注册码
00620A7C   85C0             TEST EAX,EAX                      ; EAX 是否为 0
00620A7E   75 08            JNZ SHORT MyNotesK.00620A88
00620A80   85DB             TEST EBX,EBX
00620A82   74 04            JE SHORT MyNotesK.00620A88
00620A84   85F6             TEST ESI,ESI
00620A86   75 04            JNZ SHORT MyNotesK.00620A8C
00620A88   33C0             XOR EAX,EAX
00620A8A   EB 02            JMP SHORT MyNotesK.00620A8E
```



00620A8C   B0 01                MOV AL,1                         ; 如果注册成功，AL 填充 1

00620A8E   8BD8                 MOV EBX,EAX

00620A90   33C0                 XOR EAX,EAX

00620A92   5A                   POP EDX

00620A93   59                   POP ECX

00620A94   59                   POP ECX

00620A95   64:8910              MOV DWORD PTR FS:[EAX],EDX

00620A98   68 AD0A6200          PUSH MyNotesK.00620AAD

00620A9D   8D45 FC              LEA EAX,DWORD PTR SS:[EBP-4]

00620AA0   E8 F33EDEFF          CALL MyNotesK.00404998

00620AA5   C3                   RETN

============================== CALL MyNotesK.00549CBC ==========================

00549CBC   55                   PUSH EBP

00549CBD   8BEC                 MOV EBP,ESP

00549CBF   6A 00                PUSH 0

00549CC1   6A 00                PUSH 0

00549CC3   53                   PUSH EBX

00549CC4   56                   PUSH ESI

00549CC5   57                   PUSH EDI

00549CC6   8BF1                 MOV ESI,ECX

00549CC8   8BDA                 MOV EBX,EDX

00549CCA   8BF8                 MOV EDI,EAX

00549CCC   33C0                 XOR EAX,EAX

00549CCE   55                   PUSH EBP

00549CCF   68 629D5400          PUSH MyNotesK.00549D62

00549CD4   64:FF30              PUSH DWORD PTR FS:[EAX]

00549CD7   64:8920              MOV DWORD PTR FS:[EAX],ESP

00549CDA   66:C706 F6D9         MOV WORD PTR DS:[ESI],0D9F6

00549CDF   FF75 0C              PUSH DWORD PTR SS:[EBP+C]

00549CE2   FF75 08              PUSH DWORD PTR SS:[EBP+8]

00549CE5   E8 32FDFFFF          CALL MyNotesK.00549A1C

00549CEA   66:8946 02           MOV WORD PTR DS:[ESI+2],AX

00549CEE   8D45 FC              LEA EAX,DWORD PTR SS:[EBP-4]

00549CF1   8BD3                 MOV EDX,EBX                        ; 注册名

00549CF3   E8 38ADEBFF          CALL MyNotesK.00404A30

00549CF8   8B45 FC              MOV EAX,DWORD PTR SS:[EBP-4]

00549CFB   E8 58AFEBFF          CALL MyNotesK.00404C58

00549D00   8BD8                 MOV EBX,EAX

00549D02   83FB 01              CMP EBX,1

00549D05   7C 1F                JL SHORT MyNotesK.00549D26

00549D07   8B45 FC              MOV EAX,DWORD PTR SS:[EBP-4]               ; EAX 指向注册名

00549D0A   8A4418 FF            MOV AL,BYTE PTR DS:[EAX+EBX-1]             ; 倒取注册名一个字节

00549D0E   3C 7F                CMP AL,7F                              ; 与 0X7F 比较，是否大于 ASCII 表达字符

00549D10   76 0F                JBE SHORT MyNotesK.00549D21

00549D12   8D45 FC              LEA EAX,DWORD PTR SS:[EBP-4]

00549D15   B9 01000000          MOV ECX,1

00549D1A   8BD3                 MOV EDX,EBX

00549D1C   E8 D7B1EBFF          CALL MyNotesK.00404EF8

00549D21   4B                   DEC EBX                              ; 计数器--

00549D22   85DB                 TEST EBX,EBX

00549D24   75 E1                JNZ SHORT MyNotesK.00549D07

00549D26   8D55 F8              LEA EDX,DWORD PTR SS:[EBP-8]

00549D29   8B45 FC              MOV EAX,DWORD PTR SS:[EBP-4]

00549D2C   E8 53FAEBFF          CALL MyNotesK.00409784                 ; 把注册名中小写字符转化为大写

00549D31   8B45 F8              MOV EAX,DWORD PTR SS:[EBP-8]           ; EAX 指向转化后的字符

00549D34   E8 DBF5FFFF          CALL MyNotesK.00549314                 ; 运算得到注册中间值

00549D39   8946 04              MOV DWORD PTR DS:[ESI+4],EAX

00549D3C   8BD6                 MOV EDX,ESI

00549D3E   8BC7                 MOV EAX,EDI

00549D40   B1 01                MOV CL,1

00549D42   E8 A5F4FFFF          CALL MyNotesK.005491EC                 ; 重要运算子过程

00549D47   33C0                 XOR EAX,EAX

00549D49   5A                   POP EDX

00549D4A   59                   POP ECX

00549D4B   59                   POP ECX

00549D4C   64:8910              MOV DWORD PTR FS:[EAX],EDX

00549D4F   68 699D5400          PUSH MyNotesK.00549D69

00549D54   8D45 F8              LEA EAX,DWORD PTR SS:[EBP-8]

00549D57   BA 02000000          MOV EDX,2

00549D5C   E8 5BACEBFF          CALL MyNotesK.004049BC

============================ CALL MyNotesK.00549314 ==============================

00549314   53                   PUSH EBX

00549315   8BD8                 MOV EBX,EAX

00549317   8BC3                 MOV EAX,EBX

00549319   E8 3AB9EBFF          CALL MyNotesK.00404C58

0054931E   50                   PUSH EAX

0054931F   8BC3                 MOV EAX,EBX

00549321   E8 32BBEBFF          CALL MyNotesK.00404E58

00549326   5A                   POP EDX

00549327   E8 B0FFFFFF          CALL MyNotesK.005492DC

{

005492DC   53                   PUSH EBX

005492DD   56                   PUSH ESI

005492DE   33C9                 XOR ECX,ECX                      ; 清 0

005492E0   8BDA                 MOV EBX,EDX

005492E2   4B                   DEC EBX

005492E3   85DB                 TEST EBX,EBX

005492E5   7C 25                JL SHORT MyNotesK.0054930C

005492E7   43                   INC EBX

005492E8   C1E1 04              SHL ECX,4                        ; ECX 左移 4 位

005492EB   33D2                 XOR EDX,EDX

005492ED   8A10                 MOV DL,BYTE PTR DS:[EAX]             ; 取变换后注册名的一个字节

005492EF   03CA                 ADD ECX,EDX                      ; ECX=ECX+EDX

005492F1   8BD1                 MOV EDX,ECX                      ; EDX《=ECX

005492F3   81E2 000000F0        AND EDX,F0000000                 ; EDX=EDX AND 0XF0000000

005492F9   85D2                 TEST EDX,EDX                     ; 是否为 0

005492FB   74 07                JE SHORT MyNotesK.00549304

005492FD   8BF2                 MOV ESI,EDX                      ; ESI〈=EDX

005492FF   C1EE 18              SHR ESI,18                       ; 右移 0X18 位

00549302   33CE                 XOR ECX,ESI                      ; ECX=ECX XOR ESI

00549304   F7D2                 NOT EDX                          ; 取反

00549306   23CA                 AND ECX,EDX                      ; ECX=ECX AND EDX

00549308   40                   INC EAX                          ; 指向下一个字符

00549309   4B                   DEC EBX                          ; 计数器--

0054930A   75 DC                JNZ SHORT MyNotesK.005492E8

0054930C   8BC1                 MOV EAX,ECX

0054930E   5E                   POP ESI

0054930F   5B                   POP EBX

00549310   C3                   RETN

}

逆向 C 函数代码如下:

```
unsigned long reg(char name[])

{

    int len;

    unsigned long ecx,edx,esi,c;

    len=strlen(name);

    ecx=0;

    edx=0;

    esi=0;

    for(int b=0;b<len;b++)

    {

        c=name[b];

        if(c>=97 && c<=122)

          {c=c-32;             //把注册名中小写的转化为大写

           name[b]=c;}

    }

    for(int a=0;a<len;a++)     //对转化后的注册名进行运算

    {

        ecx=ecx << 4;

        edx=name[a];

        ecx=ecx + edx;

        edx=ecx;

        edx=edx & 0xf0000000;

        if(edx==0)

           {edx=~edx;

            ecx=ecx & edx;}

        else

            {esi=edx;

            esi=esi >> 0x18;

            ecx=ecx ^ esi;

            edx=~edx;

            ecx=ecx & edx;}

    }

   return ecx;                 //返回运算值

}
```

0054932C   5B                   POP EBX

0054932D   C3                   RETN

============================= CALL MyNotesK.005491EC ===========================

005491EC   53                   PUSH EBX

005491ED   56                   PUSH ESI

005491EE   57                   PUSH EDI

005491EF   83C4 E8              ADD ESP,-18

005491F2   884C24 08            MOV BYTE PTR SS:[ESP+8],CL               ; 填充 1

005491F6   895424 04            MOV DWORD PTR SS:[ESP+4],EDX

005491FA   890424               MOV DWORD PTR SS:[ESP],EAX

005491FD   8B4424 04            MOV EAX,DWORD PTR SS:[ESP+4]

00549201   8B00                 MOV EAX,DWORD PTR DS:[EAX]

00549203   894424 0C            MOV DWORD PTR SS:[ESP+C],EAX             ; 填充 0XD9F6

00549207   8B4424 04            MOV EAX,DWORD PTR SS:[ESP+4]

0054920B   8B40 04              MOV EAX,DWORD PTR DS:[EAX+4]

0054920E   894424 10            MOV DWORD PTR SS:[ESP+10],EAX            ; 填充值



```
00549212   C74424 14 040000>MOV DWORD PTR SS:[ESP+14],4                ; 计数器为 4  

0054921A   BE 00D07500      MOV ESI,MyNotesK.0075D000  

0054921F   8B5424 0C        MOV EDX,DWORD PTR SS:[ESP+C]               ; EDX〈=0XD9F6  

00549223   33C0             XOR EAX,EAX                                ; 清 0  

00549225   8A4424 08        MOV AL,BYTE PTR SS:[ESP+8]                 ; 取刚才填充的一个字节  

00549229   8BD8             MOV EBX,EAX                                ; EBX〈=EAX  

0054922B   03DB             ADD EBX,EBX                                ; EBX=EBX+EBX  

0054922D   8D1C5B           LEA EBX,DWORD PTR DS:[EBX+EBX*2]           ; EBX=EBX*3  

00549230   8B04DE           MOV EAX,DWORD PTR DS:[ESI+EBX*8]           ; 查表  

表是这样的{0,3,1,2,1,3,1,0,2,3,2,0,3,2,0,1,0,2,2,1,3,0,3,1}  

00549233   8B0C24           MOV ECX,DWORD PTR SS:[ESP]  

00549236   8B0C81           MOV ECX,DWORD PTR DS:[ECX+EAX*4]           ; 又是查表（不同的数组）  

这里的表是{0x55147626,0x8d0bf107,0xF9492A40,0x2874514A}  

00549239   8B44DE 04        MOV EAX,DWORD PTR DS:[ESI+EBX*8+4]        ; 查表  

0054923D   8B3C24           MOV EDI,DWORD PTR SS:[ESP]  

00549240   8B0487           MOV EAX,DWORD PTR DS:[EDI+EAX*4]           ; 查表  

00549243   8B5CDE 08        MOV EBX,DWORD PTR DS:[ESI+EBX*8+8]        ; 查表  

00549247   8B3C24           MOV EDI,DWORD PTR SS:[ESP]  

0054924A   8B1C9F           MOV EBX,DWORD PTR DS:[EDI+EBX*4]          ; 查表  

0054924D   03D3             ADD EDX,EBX                                ; EDX=EDX+EBX  

0054924F   03DA             ADD EBX,EDX                                ; EBX=EBX+EDX  

00549251   8BFA             MOV EDI,EDX                                ; EDI〈=EDX  

00549253   C1EF 07          SHR EDI,7                                  ; EDI 右移 7 位  

00549256   33D7             XOR EDX,EDI                                ; EDX=EDX XOR EDI  

00549258   03CA             ADD ECX,EDX                                ; ECX=ECX+EDX  

0054925A   03D1             ADD EDX,ECX                                ; EDX=EDX+ECX  

0054925C   8BF9             MOV EDI,ECX                                ; EDI〈=ECX  

0054925E   C1E7 0D          SHL EDI,0D                                 ; 左移 0XD 位  

00549261   33CF             XOR ECX,EDI                                ; ECX=ECX XOR EDI  

00549263   03C1             ADD EAX,ECX                                ; EAX=EAX+ECX  

00549265   03C8             ADD ECX,EAX                                ; ECX=ECX+EAX  

00549267   8BF8             MOV EDI,EAX                                ; EDI〈=EAX  

00549269   C1EF 11          SHR EDI,11                                 ; EDI 右移 0X11 位  

0054926C   33C7             XOR EAX,EDI                                ; EAX=EAX XOR EDI  

0054926E   03D8             ADD EBX,EAX                                ; EBX=EBX+EAX  

00549270   03C3             ADD EAX,EBX                                ; EAX=EAX+EBX  

00549272   8BFB             MOV EDI,EBX                                ; EDI〈=EBX  

00549274   C1E7 09          SHL EDI,9                                  ; EDI 左移 9 位  

00549277   33DF             XOR EBX,EDI                                ; EBX=EBX XOR EDI  

00549279   03D3             ADD EDX,EBX                                ; EDX=EDX+EBX  

0054927B   03DA             ADD EBX,EDX                                ; EBX=EBX+EDX  

0054927D   8BFA             MOV EDI,EDX                                ; EDI〈=EDX  

0054927F   C1EF 03          SHR EDI,3                                  ; EDI 右移 3 位  

00549282   33D7             XOR EDX,EDI                                ; EDX=EDX XOR EDI  

00549284   03CA             ADD ECX,EDX                                ; ECX=ECX+EDX  

00549286   8BD1             MOV EDX,ECX                                ; EDX〈=ECX  

00549288   C1E2 07          SHL EDX,7                                  ; EDX 左移 7 位  

0054928B   33CA             XOR ECX,EDX                                ; ECX=ECX XOR EDX  

0054928D   03C1             ADD EAX,ECX                                ; EAX=EAX+ECX  

0054928F   8BD3             MOV EDX,EBX                                ; EDX〈=EBX  

00549291   C1EA 0F          SHR EDX,0F                                 ; 右移 0XF 位  

00549294   33C2             XOR EAX,EDX                                ; EAX=EAX XOR EDX  

00549296   03D8             ADD EBX,EAX                                ; EBX=EBX+EAX  

00549298   8BC3             MOV EAX,EBX                                ; EAX〈=EBX  

0054929A   C1E0 0B          SHL EAX,0B                                  ; 左移 0XB 位  

0054929D   33D8             XOR EBX,EAX                                ; EBX=EBX XOR EAX  

0054929F   8B4424 10        MOV EAX,DWORD PTR SS:[ESP+10]              ; 取 4 个字节到 EAX  

005492A3   33C3             XOR EAX,EBX                                ; EAX=EAX XOR EBX  

005492A5   8B5424 0C        MOV EDX,DWORD PTR SS:[ESP+C]               ; 取 4 个字节到 EDX  

005492A9   895424 10        MOV DWORD PTR SS:[ESP+10],EDX              ; 填充  

005492AD   894424 0C        MOV DWORD PTR SS:[ESP+C],EAX               ; 填充  

005492B1   83C6 0C          ADD ESI,0C                                  ; ESI=ESI+0XC（这个是把数组初始地址加高）  

005492B4   FF4C24 14        DEC DWORD PTR SS:[ESP+14]                   ; 计数器--  

005492B8  ⁰F85 61FFFFFF    JNZ MyNotesK.0054921F  

005492BE   8B4424 04        MOV EAX,DWORD PTR SS:[ESP+4]  

005492C2   8B5424 10        MOV EDX,DWORD PTR SS:[ESP+10]  

005492C6   8910             MOV DWORD PTR DS:[EAX],EDX                 ; 填充  

005492C8   8B4424 04        MOV EAX,DWORD PTR SS:[ESP+4]  

005492CC   8B5424 0C        MOV EDX,DWORD PTR SS:[ESP+C]  

005492D0   8950 04          MOV DWORD PTR DS:[EAX+4],EDX               ; 填充  

005492D3   83C4 18          ADD ESP,18  

005492D6   5F               POP EDI  

005492D7   5E               POP ESI  

005492D8   5B               POP EBX  

005492D9   C3               RETN  
```

这里的运算，我认为就是变换数组里的值，第一次填充的是常数(`0XD9F6`，注册中间值)，然后进行复杂运算，变换数组里的值，里面还用到了 2 个表第一个是`{0,3,1,2,1,3,1,0,2,3,2,0,3,2,0,1,0,2,2,1,3,0,3,1}`，第 2 个是`{0x55147626,0x8d0bf107,0xF9492A40,0x2874514A}`

不知道为什么复杂的运算后是明码比较，悲。

### 算法小结

先运算得到注册中间值，然后根据(`注册中间值`，和常数 `0XD9F6`)，进行运算，得到最后的注册码。

### `KeyGen` 代码如下(C 语言)

```c
//为了省事没有对中文注册名进行处理,所以只能输入英文注册名

#include <stdio.h>

#include <math.h>

#include <string.h>

unsigned long  temp[8];

int regcode[8];

unsigned long reg(char name[])    //根据注册名运算得到一个注册中间值,后面运算将用到

{

    int len;

    unsigned long ecx,edx,esi,c;

    len=strlen(name);

    ecx=0;

    edx=0;

    esi=0;

    for(int b=0;b<len;b++)

    {

        c=name[b];

        if(c>=97 && c<=122)

          {c=c-32;              //把注册名中小写的转化为大写

           name[b]=c;}

    }

    for(int a=0;a<len;a++)     //对转化后的注册名进行运算

    {

        ecx=ecx << 4;

        edx=name[a];

        ecx=ecx + edx;

        edx=ecx;

        edx=edx & 0xf0000000;

        if(edx==0)

           {edx=~edx;

            ecx=ecx & edx;}

        else

            {esi=edx;

            esi=esi >> 0x18;

            ecx=ecx ^ esi;

            edx=~edx;

            ecx=ecx & edx;}

    }

   return ecx;                   //返回运算值

}

void DecToHex(unsigned long t)  //dectohex 后变换一下字符并输出

{

    unsigned long a;

    long c;

    int i=0;

    int p;

    do

    {

        c=t%16;

        temp[ i]=c;

        i++;

    }while((t/=16)!=0);

    for(i--;i>=0;i--)

    {

        if(temp[ i]>=10&&temp[ i]<=15)//是字母？

            temp[i]=(temp[ i]+55);    //转化为 ascii

    }

    for (int w=0;w<8;w++)

    {

        if(w%2==0)                    //根据情况变换

         regcode[w]=temp[w+1];

        else

         regcode[w]=temp[w-1];

    }

    for(int o=0;o<=7;o++)

    {

       if(regcode[o]>=10)             //是字母?

        printf("%c",regcode[o]);

       else

        printf("%d",regcode[o]);

    }

}

main()

{

   char name[100];
```



```c
unsigned long a,eax,ebx,edi,ecx,edx;

unsigned long cs[4]={0x55147626,0x8d0bf107,0xF9492A40,0x2874514A};

//程序内定 16 进制值

unsigned long hh[4]={0x1,0xd9f6,0x0,0x4};

//自己根据汇编代码定义的数组,第 2 个元素待填充(先设为 0)

unsigned long zz[]={0,3,1,2,1,3,1,0,2,3,2,0,3,2,0,1,0,2,2,1,3,0,3,1};

//程序中定义的表

int b,pp;

printf("\t\t****************************************************************\n");

printf("\t\t*          KeyGen For MyNotesKeeper V1.4          *\n");

printf("\t\t*                Coded By lnn1123                 *\n");

printf("\t\t****************************************************************\n");

printf("Please input your name:\n");

gets(name);

a=reg(name);

b=1;

hh[2]=a;

pp=4;

edx=0xd9f6;

int t=0;

for(int s=0;s<4;s++)    //循环 4 次
{
    edx=hh[1];
    eax=hh[0];
    ebx=eax;
    ebx=ebx+ebx;
    ebx=ebx*3;
    if(t==0)               //根据 t 值对数组访问,可能效率比较低
      eax=zz[ebx*2];
    if(t==3)
      eax=zz[ebx*2+3];
    if(t==6)
      eax=zz[ebx*2+6];
    if(t==9)
      eax=zz[ebx*2+9];
    ecx=cs[eax];
    if(t==0)               //根据 t 值对数组访问,可能效率比较低
      eax=zz[ebx*2+1];
    if(t==3)
      eax=zz[ebx*2+3+1];
    if(t==6)
      eax=zz[ebx*2+6+1];
    if(t==9)
      eax=zz[ebx*2+9+1];
    eax=cs[eax];
    if(t==0)               //根据 t 值对数组访问,可能效率比较低
      ebx=zz[ebx*2+2];
    if(t==3)
      ebx=zz[ebx*2+3+2];
    if(t==6)
      ebx=zz[ebx*2+6+2];
    if(t==9)
      ebx=zz[ebx*2+9+2];
    ebx=cs[ebx];            //下面是是进行运算
    edx=edx+ebx;
    ebx=ebx+edx;
    edi=edx;
    edi=edi >> 7;
    edx=edx ^ edi;
    ecx=ecx+edx;
    edx=edx+ecx;
    edi=ecx;
    edi=edi << 0xd;
    ecx=ecx ^ edi;
    eax=eax+ecx;
    ecx=ecx+eax;
    edi=eax;
    edi=edi >> 0x11;
    eax=eax ^ edi;
    ebx=ebx+eax;
    eax=eax+ebx;
    edi=ebx;
    edi=edi << 0x9;
    ebx=ebx ^ edi;
    edx=edx+ebx;
    ebx=ebx+edx;
    edi=edx;
    edi=edi >> 0x3;
    edx=edx ^ edi;
    ecx=ecx +edx;
    edx=ecx;
    edx=edx << 0x7;
    ecx=ecx ^ edx;
    eax=eax+ecx;
    edx=ebx;
    edx=edx >> 0xf;
    eax=eax ^ edx;
    ebx=ebx+eax;
    eax=ebx;
    eax=eax << 0xb;
    ebx=ebx ^ eax;
    eax=hh[2];
    eax=eax ^ ebx;
    edx=hh[1];
    hh[2]=edx;
    hh[1]=eax;
    t=t+3;
}
printf("Your Regcode is :\n");
DecToHex(hh[2]);
DecToHex(hh[1]);
getchar();
}
```

特别感谢：

Phoenix, 爱情诗人和所有帮助过我的人。^\_^

=======================================================================================================

【破解声明】我是一个小小菜虫子, 文章如有错误, 请高手指正!

【版权声明】本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

=======================================================================================================

文章完成于 2005-12-22 18:08:46 4dd466d0e1e0cc123afbe504da2462ca
