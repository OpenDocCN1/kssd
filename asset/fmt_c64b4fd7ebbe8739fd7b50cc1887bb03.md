
00444181   . 50             PUSH EAX  

00444182   . 8D5424 34      LEA EDX,DWORD PTR SS:\[ESP+34\]  

00444186   . 52             PUSH EDX  

00444187   . C64424 50 02   MOV BYTE PTR SS:\[ESP+50\],2  

0044418C   . E8 4FFFFBFF    CALL 机械设计.004040E0          ;  连接生成 SQL 查询语句  

00444191   . 68 B02C5000    PUSH 机械设计.00502CB0  

00444196   . 50             PUSH EAX  

00444197   . 8D4424 3C      LEA EAX,DWORD PTR SS:\[ESP+3C\]  

0044419B   . 50             PUSH EAX  

0044419C   . C64424 5C 03   MOV BYTE PTR SS:\[ESP+5C\],3  

004441A1   . E8 EAFFFBFF    CALL 机械设计.00404190          ; 连接生成 SQL 查询语句  

\>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  

(跟进 0040190)  

004041F9   . 8D4424 30      LEA EAX,DWORD PTR SS:\[ESP+30\]  

004041FD   . 50             PUSH EAX  

004041FE   . E8 FDFBFFFF    CALL 机械设计.00403E00        ;  连接生成 SQL 语句的 call  

00404203   . 8B7424 34      MOV ESI,DWORD PTR SS:\[ESP+34\] ；\[ESP+34\]为生成的 sql 语句地址  

\>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  

0044425B   . 8907          MOV DWORD PTR DS:\[EDI\],EAX       ;  EAX 中为查询密码 sql 语句地址  

0044425D   . E8 FED10000    CALL 机械设计.00451460  

00444262   . A1 ECB75400    MOV EAX,DWORD PTR DS:\[54B7EC\]  

00444267   . 83C4 04        ADD ESP,4  

0044426A   . 85C0          TEST EAX,EAX  

0044426C   . 75 0F          JNZ SHORT 机械设计.0044427D  

0044426E   . 68 03400080    PUSH 80004003  

00444273   . E8 68740800    CALL 机械设计.004CB6E0  

00444278   . A1 ECB75400    MOV EAX,DWORD PTR DS:\[54B7EC\]  

0044427D   > 8B08          MOV ECX,DWORD PTR DS:\[EAX\]  

0044427F   . 8D5424 10      LEA EDX,DWORD PTR SS:\[ESP+10\]  

00444283   . 52             PUSH EDX  

00444284   . 50             PUSH EAX  

00444285   . 8BF8          MOV EDI,EAX  

00444287   . C74424 18 0000>MOV DWORD PTR SS:\[ESP+18\],0  

0044428F   . FF51 50        CALL DWORD PTR DS:\[ECX+50\]       ;  调用 msADO15.dll  

00444292   . 85C0          TEST EAX,EAX  

00444294   . 7D 0C          JGE SHORT 机械设计.004442A2  

00444296   . 68 B01C5000    PUSH 机械设计.00501CB0  

0044429B   . 57             PUSH EDI  

0044429C   . 50             PUSH EAX  

0044429D   . E8 4E740800    CALL 机械设计.004CB6F0  

004442A2   > 66:837C24 10 0>CMP WORD PTR SS:\[ESP+10\],0       ;  判断是否登录成功  

004442A8   . 0F84 36010000  JE 机械设计.004443E4  

004442AE   . 8B8E 7C010000  MOV ECX,DWORD PTR DS:\[ESI+17C\]  

004442B4   . 83F9 02        CMP ECX,2                       ;  比较登录次数  

004442B7   . 0F8D 9B000000  JGE 机械设计.00444358  

下面为我输入 `1` 后在 OD 中看到的：  

```
01FBA290  73 65 6c 65 63 74 20 2a 20 66 72 6f 6d 20 b5 c7   select * from 登
01FBA2A0  c2 bc d3 c3 bb a7 20 77 68 65 72 65 20 41 6c 69   录用户 where Ali
01FBA2b0  61 73 3d 27 41 64 6d 69 6e 69 73 74 65 72 27 41    as='Administer'A
01FBA2C0  4e 44 20 50 61 73 73 77 6f 72 64 3d 27 42 58 27    ND Password='BX'
```

很明显:    `1`   对应密码为  `BX` .  

我们只需把数据库中的用户密码改为 `BX`，输入密码 `1` 就可成功登陆.  

6.  

   下面再回过头来看看其是如何加密用户密码的:  

```
0044E660   $ 6A FF          PUSH -1
0044E662   . 68 C8754F00    PUSH 机械设计.004F75C8              ;  SE handler installation
0044E667   . 64:A1 00000000 MOV EAX,DWORD PTR FS:\[0\]
0044E66D   . 50             PUSH EAX
0044E66E   . 64:8925 000000>MOV DWORD PTR FS:\[0\],ESP
0044E675   . 83EC 1C        SUB ESP,1C
0044E678   . 53             PUSH EBX
0044E679   . 55             PUSH EBP
0044E67A   . 56             PUSH ESI
0044E67B   . 33F6           XOR ESI,ESI
0044E67D   . 57             PUSH EDI
0044E67E   . 897424 28      MOV DWORD PTR SS:\[ESP+28\],ESI
0044E682   . 897424 34      MOV DWORD PTR SS:\[ESP+34\],ESI
0044E686   . E8 884D0800    CALL 机械设计.004D3413
0044E68B   . 8B10           MOV EDX,DWORD PTR DS:\[EAX\]
0044E68D   . 8BC8           MOV ECX,EAX
0044E68F   . FF52 0C        CALL DWORD PTR DS:\[EDX+C\]
0044E692   . 83C0 10        ADD EAX,10
0044E695   . 894424 14      MOV DWORD PTR SS:\[ESP+14\],EAX
0044E699   . C64424 34 01   MOV BYTE PTR SS:\[ESP+34\],1
0044E69E   . E8 704D0800    CALL 机械设计.004D3413
0044E6A3   . 8B10           MOV EDX,DWORD PTR DS:\[EAX\]
0044E6A5   . 8BC8           MOV ECX,EAX
0044E6A7   . FF52 0C        CALL DWORD PTR DS:\[EDX+C\]
0044E6AA   . 8D58 10        LEA EBX,DWORD PTR DS:\[EAX+10\]
0044E6AD   . 895C24 18      MOV DWORD PTR SS:\[ESP+18\],EBX
0044E6B1   . 8D4424 40      LEA EAX,DWORD PTR SS:\[ESP+40\]
0044E6B5   . 50             PUSH EAX
0044E6B6   . 8D4C24 18      LEA ECX,DWORD PTR SS:\[ESP+18\]
0044E6BA   . C64424 38 02   MOV BYTE PTR SS:\[ESP+38\],2
0044E6BF   . E8 4C59FBFF    CALL 机械设计.00404010
0044E6C4   . 8B4424 40      MOV EAX,DWORD PTR SS:\[ESP+40\]        ;  \[ESP+40\]\]为输入密码地址
0044E6C8   . 3970 F4        CMP DWORD PTR DS:\[EAX-C\],ESI
0044E6CB   . 8B6C24 14      MOV EBP,DWORD PTR SS:\[ESP+14\]        ;  \[ESP+14\]为输入密码地址
0044E6CF   . 7E 5A          JLE SHORT 机械设计.0044E72B
                                                     ;  循环开始
0044E6D1   > 8A0C06         MOV CL,BYTE PTR DS:\[ESI+EAX\]         ;  依次取输入密码每一位 ASCII 码送 CL
0044E6D4   . 8B7D F4        MOV EDI,DWORD PTR SS:\[EBP-C\]         ;  \[EBP-C\]中为输入密码长度
0044E6D7   . 884C24 13      MOV BYTE PTR SS:\[ESP+13\],CL          ;  密码 ASCII 码送\[ESP+13\]
0044E6DB   . 837D FC 01     CMP DWORD PTR SS:\[EBP-4\],1
0044E6DF   . 7E 14          JLE SHORT 机械设计.0044E6F5
0044E6E1   . 8BD7           MOV EDX,EDI
0044E6E3   . 52             PUSH EDX
0044E6E4   . 8D4C24 18      LEA ECX,DWORD PTR SS:\[ESP+18\]
0044E6E8   . E8 A330FBFF    CALL 机械设计.00401790
0044E6ED   . 8B6C24 14      MOV EBP,DWORD PTR SS:\[ESP+14\]        ;  输入密码地址送 EBP
0044E6F1   . 8B4424 40      MOV EAX,DWORD PTR SS:\[ESP+40\]        ;  输入密码地址送 EAX
0044E6F5   > 8A4C24 45      MOV CL,BYTE PTR SS:\[ESP+45\]          ;  取变量\[ESP+44\]内存中第二字节（即十六进制值高 8 位）
0044E6F9   . 324C24 13      XOR CL,BYTE PTR SS:\[ESP+13\]          ;  CL 异或  每位密码 ASCII 码
0044E6FD   . 880C2E         MOV BYTE PTR DS:\[ESI+EBP\],CL        ;  用异或值代替原密码
0044E700   . 8B4C24 44      MOV ECX,DWORD PTR SS:\[ESP+44\]
0044E704   . 897D F4        MOV DWORD PTR SS:\[EBP-C\],EDI         ;  密码长度送\[EBP-C\]
0044E707   . C6042F 00      MOV BYTE PTR DS:\[EDI+EBP\],0
0044E70B   . 66:0FB6142E    MOVZX DX,BYTE PTR DS:\[ESI+EBP\]       ;  依次取输入密码每一位 ASCII 码送 DX
0044E710   . 03D1           ADD EDX,ECX
0044E712   . 69D2 93310000  IMUL EDX,EDX,3193                     ;   EDX=EDX\*3193
0044E718   . B9 BF580000    MOV ECX,58BF                          ;  ECX=58BF
0044E71D   . 2BCA           SUB ECX,EDX                           ;  ECX=ECX-EDX
0044E71F   . 894C24 44      MOV DWORD PTR SS:\[ESP+44\],ECX       ;  ECX 送变量\[ESP+44\]
0044E723   . 8B48 F4        MOV ECX,DWORD PTR DS:\[EAX-C\]         ;  输入密码长度送 ECX
0044E726   . 46             INC ESI                               ;  计数器加 1
0044E727   . 3BF1           CMP ESI,ECX                           ;  比较循环次数
0044E729   . 7C A6          JL SHORT 机械设计.0044E6D1
..........................................
........................................(略）
.........................................
```

下面程序把密码位一个字符转换为两个字符.

```
0044E7C8   . 85C9           TEST ECX,ECX
0044E7CA   . C74424 20 0000>MOV DWORD PTR SS:\[ESP+20\],0
0044E7D2   . 0F8E 15010000  JLE 机械设计.0044E8ED
0044E7D8   . EB 06          JMP SHORT 机械设计.0044E7E0
0044E7DA   > 8B4424 40      MOV EAX,DWORD PTR SS:\[ESP+40\]        ;  循环开始
0044E7DE   . 8BFF           MOV EDI,EDI
0044E7E0   > 8B4C24 20      MOV ECX,DWORD PTR SS:\[ESP+20\]
0044E7E4   . 0FB63C01       MOVZX EDI,BYTE PTR DS:\[ECX+EAX\]      ;  依次前面经过转换后密码每一位 ASCII 码送 CL
0044E7E8   . B8 B8BA5000    MOV EAX,机械设计.0050BAB8             ;  ASCII "12"
0044E7ED   . 8D50 01        LEA EDX,DWORD PTR DS:\[EAX+1\]
0044E7F0   > 8A08           MOV CL,BYTE PTR DS:\[EAX\]
0044E7F2   . 40             INC EAX
0044E7F3   . 84C9           TEST CL,CL
```
