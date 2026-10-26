
# [原创]Black Stone 黑石算法分析

## 黑石（BlackStone）

连珠(五子棋)对弈/辅助计算软件。BlackStone本意为棋子中的黑子，在国内通译为黑石。

黑石由俄罗斯人Victor Barykin开发，最早的版本发布于1997年。黑石被公认为目前已公开发布的连珠/五子棋对弈软件中综合棋力最强的，曾获得第二、三届世界计算机连珠程序锦标赛(Renju Computer World Championship)竞赛组冠军。黑石采用的对弈规则为三手交换，五手两打，有禁手，可调节难度(剪枝方法、计算深度和时间)，4.0以后的版本可调节棋盘线数。黑石的局部计算能力强大，被普遍用于局面研究拆解和网络五子棋对弈。2000年传入中国后，对连珠/五子棋开局理论的研究起到了前所未有的巨大推进作用，并成为网络连珠/五子棋对弈中最常用的辅助计算软件。在黑石传入中国后的短短几年中，依靠黑石发展、修正、建立起的连珠/五子棋开局理论，超过了过去一百多年全世界连珠/五子棋开局理论总和的数百甚至上千倍之多；目前高水平的网络五子棋对弈、比赛，参赛选手绝大多选用黑石作为辅助计算工具。

软件明码比较，压缩壳，我们带壳调试,使用`a_p`大哥修改的野猪`OD`.

```
00517000 >  53              PUSH EBX                                 ; OD载入后来到这里

00517001    55              PUSH EBP

00517002    89C5            MOV EBP,EAX

00517004    33DB            XOR EBX,EBX

00517006    EB 60           JMP SHORT Wrenju.00517068                ; F8单步一次
```

```
00517068    E8 00000000     CALL Wrenju.0051706D                     ; JMP后来到这里

0051706D    58              POP EAX

0051706E    2D 6D000000     SUB EAX,6D

00517073    50              PUSH EAX

00517074    60              PUSHAD                                   ; 我们看到这里有PUSHAD

00517075    33C9            XOR ECX,ECX                              ; 所以我们向下找一下POPAD
```

```
00517282    61              POPAD                                    ; 在这里看到了POPAD

00517283    58              POP EAX

00517284    8BE8            MOV EBP,EAX

00517286    2E:0385 9502000>ADD EAX,DWORD PTR CS:[EBP+295]

0051728D    05 99020000     ADD EAX,299

00517292    5D              POP EBP

00517293    5B              POP EBX

00517294  ^ E9 17B0F6FF     JMP Wrenju.004822B0                      ; JMP到OEP // 我们在该行左键单击 然后F4来到这里
```

```
004822B0    55              PUSH EBP                                 ; 来到OEP

004822B1    8BEC            MOV EBP,ESP

004822B3    83C4 F4         ADD ESP,-0C

004822B6    53              PUSH EBX
```

F9运行，我们用`OD`的插件下万能断点来拦截注册部分：

```
77D3353D    F3:A5           REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS>; 断到这里，ESI中显示我们输入的假码

77D3353F    8BC8            MOV ECX,EAX

77D33541    83E1 03         AND ECX,3

77D33544    F3:A4           REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[>

77D33546    E8 E3FBFFFF     CALL user32.77D3312E

77D3354B    5F              POP EDI

77D3354C    5E              POP ESI

77D3354D    8BC3            MOV EAX,EBX

77D3354F    5B              POP EBX

77D33550    5D              POP EBP

77D33551    C2 1000         RETN 10                                  ; 我们Ctrl+F9一路返回到程序领空
```

```
00469084    55              PUSH EBP

00469085    8BEC            MOV EBP,ESP

00469087    83C4 E4         ADD ESP,-1C

0046908A    53              PUSH EBX

0046908B    56              PUSH ESI

0046908C    57              PUSH EDI

0046908D    33C9            XOR ECX,ECX

0046908F    894D E4         MOV DWORD PTR SS:[EBP-1C],ECX

00469092    894D E8         MOV DWORD PTR SS:[EBP-18],ECX

00469095    8BD8            MOV EBX,EAX

00469097    33C0            XOR EAX,EAX

00469099    55              PUSH EBP

0046909A    68 F2924600     PUSH Wrenju.004692F2

0046909F    64:FF30         PUSH DWORD PTR FS:[EAX]

004690A2    64:8920         MOV DWORD PTR FS:[EAX],ESP

004690A5    8D55 E8         LEA EDX,DWORD PTR SS:[EBP-18]

004690A8    8B83 FC010000   MOV EAX,DWORD PTR DS:[EBX+1FC]

004690AE    E8 E95DFBFF     CALL Wrenju.0041EE9C                     ; 该CALL取假码

004690B3    8B45 E8         MOV EAX,DWORD PTR SS:[EBP-18]            ; 我们返回到这里

004690B6    8D55 FC         LEA EDX,DWORD PTR SS:[EBP-4]

004690B9    E8 769BF9FF     CALL Wrenju.00402C34

004690BE    8BF8            MOV EDI,EAX

004690C0    A1 E8424800     MOV EAX,DWORD PTR DS:[4842E8]

004690C5    8B00            MOV EAX,DWORD PTR DS:[EAX]

004690C7    8BCF            MOV ECX,EDI

004690C9    BA 08000000     MOV EDX,8

004690CE    E8 1D2CFFFF     CALL Wrenju.0045BCF0

004690D3    8D55 E8         LEA EDX,DWORD PTR SS:[EBP-18]

004690D6    8B83 00020000   MOV EAX,DWORD PTR DS:[EBX+200]

004690DC    E8 BB5DFBFF     CALL Wrenju.0041EE9C

004690E1    8B45 E8         MOV EAX,DWORD PTR SS:[EBP-18]            ; 该CALL取得Serial Number数值

004690E4    E8 5FE2F9FF     CALL Wrenju.00407348

004690E9    8BF0            MOV ESI,EAX                              ; 将Serial Number数值转化为16进制后保存到ESI

004690EB    81C6 6B030000   ADD ESI,36B                              ; Serial Number数值+36BH

004690F1    A1 E8424800     MOV EAX,DWORD PTR DS:[4842E8]

004690F6    8B00            MOV EAX,DWORD PTR DS:[EAX]

004690F8    8BCE            MOV ECX,ESI

004690FA    BA 09000000     MOV EDX,9

004690FF    E8 EC2BFFFF     CALL Wrenju.0045BCF0

00469104    A1 E8424800     MOV EAX,DWORD PTR DS:[4842E8]

00469109    8B00            MOV EAX,DWORD PTR DS:[EAX]

0046910B    8B0D 70274A00   MOV ECX,DWORD PTR DS:[4A2770]

00469111    BA 07000000     MOV EDX,7

00469116    E8 D52BFFFF     CALL Wrenju.0045BCF0

0046911B    56              PUSH ESI

0046911C    8B0D 74274A00   MOV ECX,DWORD PTR DS:[4A2774]            ; Computer Number数值放ECX

00469122    8B15 70274A00   MOV EDX,DWORD PTR DS:[4A2770]            ; Serial Number  数值放EDX

00469128    8BC7            MOV EAX,EDI

0046912A    E8 3932FFFF     CALL Wrenju.0045C368                     ; 算法CALL 我们跟进

0046912F    8B15 28434800   MOV EDX,DWORD PTR DS:[484328]            ; Wrenju.004A272D

00469135    C602 00         MOV BYTE PTR DS:[EDX],0

00469138    48              DEC EAX

00469139    0F84 8E010000   JE Wrenju.004692CD

0046913F    48              DEC EAX

00469140    74 64           JE SHORT Wrenju.004691A6

00469142    48              DEC EAX

00469143    74 0C           JE SHORT Wrenju.00469151                 ; 这里跳向注册成功  若想实现这里跳转算法CALL返回时EAX数值应为3

00469145    48              DEC EAX

00469146    0F84 9B000000   JE Wrenju.004691E7                       ; 这里跳向注册成功  若想实现这里跳转算法CALL返回时EAX数值应为4

0046914C    E9 CD000000     JMP Wrenju.0046921E

00469151    8B0D 78444800   MOV ECX,DWORD PTR DS:[484478]            ; Wrenju.004A2784

00469157    8B09            MOV ECX,DWORD PTR DS:[ECX]

00469159    A1 E8424800     MOV EAX,DWORD PTR DS:[4842E8]

0046915E    8B00            MOV EAX,DWORD PTR DS:[EAX]

00469160    BA 01000000     MOV EDX,1

00469165    E8 4A2DFFFF     CALL Wrenju.0045BEB4

0046916A    83F8 01         CMP EAX,1

0046916D    74 05           JE SHORT Wrenju.00469174

0046916F    83F8 16         CMP EAX,16

00469172    75 15           JNZ SHORT Wrenju.00469189

00469174    6A 00           PUSH 0

00469176    68 00934600     PUSH Wrenju.00469300                     ; ASCII "Success"

0046917B    68 08934600     PUSH Wrenju.00469308                     ; ASCII "Application authorization complete"

00469180    6A 00           PUSH 0

00469182    E8 F9CFF9FF     CALL <JMP.&user32.MessageBoxA>

00469187    EB 0B           JMP SHORT Wrenju.00469194

00469189    BA 2C934600     MOV EDX,Wrenju.0046932C                  ; ASCII "Application Violation - Error #"

0046918E    92              XCHG EAX,EDX

0046918F    E8 28080000     CALL Wrenju.004699BC

00469194    E8 8FFCFFFF     CALL Wrenju.00468E28

00469199    A1 28434800     MOV EAX,DWORD PTR DS:[484328]

0046919E    C600 01         MOV BYTE PTR DS:[EAX],1

004691A1    E9 27010000     JMP Wrenju.004692CD

004691A6    8B15 78444800   MOV EDX,DWORD PTR DS:[484478]            ; Wrenju.004A2784

004691AC    8B12            MOV EDX,DWORD PTR DS:[EDX]

004691AE    A1 E8424800     MOV EAX,DWORD PTR DS:[4842E8]

004691B3    8B00            MOV EAX,DWORD PTR DS:[EAX]

004691B5    E8 EE2DFFFF     CALL Wrenju.0045BFA8

004691BA    83F8 01         CMP EAX,1

004691BD    74 10           JE SHORT Wrenju.004691CF

004691BF    BA 4C934600     MOV EDX,Wrenju.0046934C                  ; ASCII "De-authorization failed - Error #"

004691C4    92              XCHG EAX,EDX

004691C5    E8 F2070000     CALL Wrenju.004699BC

004691CA    E9 FE000000     JMP Wrenju.004692CD

004691CF    6A 00           PUSH 0

004691D1    68 00934600     PUSH Wrenju.00469300                     ; ASCII "Success"
```
