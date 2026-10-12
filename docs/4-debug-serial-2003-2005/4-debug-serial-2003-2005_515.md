
# 网文快捕 3.6

> 原文：[`www.pediy.com/kssd/pediy06/pediy6065.htm`](https://www.pediy.com/kssd/pediy06/pediy6065.htm)

- **【软件名称】**：网文\*\*
- **【整理日期】**：2003.5.21
- **【最新版本】**：3.6
- **【文件大小】**：4315 KB
- **【软件授权】**：共享软件
- **【使用平台】**：`Win9x`/`Me`/`NT`/`2000`/`XP`
- **【软件简介】**：
  原名 `eLib`。在您上网的时候，您一定会遇到很到好文章，怎么保存？用 `IE` 的保存功能？过不了多久，您就会忘记，保存在那里了。使用电子书库软件？可惜只能保存文字信息，有用的图片不能保存了。您可能会遇到好看的图片，使用这类软件，更是无能为力了。而且，使用 `eLib` 保存网页，非常简单，通过鼠标右键，您就可以快速的将文章保存起来。还有，您可能更希望将好看的 `Flash` 动画保存下来，这次，连 `IE` 也无能为力了。怎么办？使用 `eLib`，您可以将这些内容完整无缺的保存下来。您可能还会问，这些信息，`eLib` 怎么管理呢？这是 `eLib` 的强项，它可以将所有的信息，分门别类的保存下来。就像 `windows` 资源管理器一样，左边是目录和文章（也就是您保存的网页乙级分类结构），右边是一个浏览器，您可以在这里察看您保存的网页。`eLib` 将所有的信息保存在一个文件里面，方便您进行备份，交流。同时，他还可以将您的文件打包成 `chm` 文件，在网上交流更加方便。
- **【作    者】**： 风雨无阻【`CCG`】
- **【声    明】**：纯技术交流，无任何商业目的
- **【破解工具】**：`fi2.5`, `Ollydbg1.09` 中文版，`filemon`，`regmon`，`win2000` 系统
- **【破解过程】**

### 第一步

用 `fi2.5` 对可执行文件进行分析，是 `upx1.24` 的壳，很好脱。执行程序，注册时需要重启，也就是说要读注册表或文件，用 `regmon` 和 `filemon` 进行分析发现在 `D:\Documents and Settings\Administrator\Application Data\WebCatcher` (我的 `win2000` 安装在 `d`) 目录下生成一名为 `Config.ini` 的文件，打开就会看到刚填写的注册信息,有字段为 `SerialNo` 和 `name`，用 `ollydbg` 载入搜索字符参考，查找这两个字符窜然后在调用之前下断点。

### 第二步

分析注册算法，当输入假码时，只中断在读取注册码处，程序并没有读取用户名，也就是说注册码与用户名无关。

```assembly
00406CF6  |. 3>XOR EDX,EDX
00406CF8  |. 8>MOV DWORD PTR SS:[EBP-4],EDX
00406CFB  |. 8>LEA EAX,DWORD PTR SS:[EBP-4]
00406CFE  |. F>INC DWORD PTR DS:[ESI+1C]
00406D01  |. E>CALL WebCatch.@WaGetSerialNo$qqrv                  ；读取假码 11111-22222-33333-44444-55555
00406D06  |. 8>LEA EAX,DWORD PTR SS:[EBP-4]
00406D09  |. 8>MOV EAX,DWORD PTR DS:[EAX]
00406D0B  |. E>CALL WebCatch.@WaDownloadImageFile$qqr17System@AnsiString ；关键 call ，稍后会详细分析
00406D10  |. 5>PUSH EAX                                          ; /Arg1
00406D11  |. F>DEC DWORD PTR DS:[ESI+1C]                         ; |
00406D14  |. 8>LEA EAX,DWORD PTR SS:[EBP-4]                      ; |
00406D17  |. B>MOV EDX,2                                         ; |
00406D1C  |. E>CALL WebCatch.0055053C                             ; \WebCatch.0055053C
00406D21  |. 5>POP ECX
00406D22  |. 8>TEST CL,CL
00406D24  |. 7>JE SHORT WebCatch.00406D68                        ；关键跳转
00406D26  |. 6>MOV WORD PTR DS:[ESI+10],14
00406D2C  |. B>MOV EDX,WebCatch.005CA628
00406D31  |. 8>LEA EAX,DWORD PTR SS:[EBP-8]
00406D34  |. E>CALL WebCatch.00550364
00406D39  |. F>INC DWORD PTR DS:[ESI+1C]
004551F4 >/$ >PUSH EBP
004551F5  |. >MOV EBP,ESP
004551F7  |. >ADD ESP,-80
004551FA  |. >PUSH EBX
004551FB  |. >PUSH ESI
004551FC  |. >PUSH EDI
004551FD  |. >MOV DWORD PTR SS:[EBP-4],EAX
00455200  |. >MOV ESI,WebCatch.005CA5B8                           ;  ASCII "1111122222333334444455555"
00455205  |. >MOV EDI,WebCatch.005CA5C2                            ;  ASCII "333334444455555"
0045520A  |. >MOV EAX,WebCatch.005A3B50
0045520F  |. >CALL WebCatch.00543724
00455214  |. >MOV DWORD PTR SS:[EBP-10],1
0045521B  |. >LEA EDX,DWORD PTR SS:[EBP-4]
0045521E  |. >LEA EAX,DWORD PTR SS:[EBP-4]
00455221  |. >CALL WebCatch.00550364
00455226  |. >INC DWORD PTR SS:[EBP-10]
00455229  |. >XOR EDX,EDX
0045522B  |. >MOV WORD PTR SS:[EBP-1C],8
00455231  |. >MOV WORD PTR SS:[EBP-1C],14
00455237  |. >MOV DWORD PTR SS:[EBP-8],EDX
0045523A  |. >LEA EDX,DWORD PTR SS:[EBP-8]
0045523D  |. >INC DWORD PTR SS:[EBP-10]
00455240  |. >LEA EAX,DWORD PTR SS:[EBP-4]
00455243  |. >CALL WebCatch.00550948
00455248  |. >LEA EDX,DWORD PTR SS:[EBP-8]
0045524B  |. >LEA EAX,DWORD PTR SS:[EBP-4]
0045524E  |. >CALL WebCatch.0055056C
00455253  |. >DEC DWORD PTR SS:[EBP-10]
00455256  |. >LEA EAX,DWORD PTR SS:[EBP-8]
00455259  |. >MOV EDX,2
0045525E  |. >CALL WebCatch.0055053C
00455263  |. >CMP DWORD PTR SS:[EBP-4],0
00455267  |. >JE SHORT WebCatch.0045526E
00455269  |. >MOV ECX,DWORD PTR SS:[EBP-4]
0045526C  |. >JMP SHORT WebCatch.00455273
0045526E  |> >MOV ECX,WebCatch.005A2D40
00455273  |> >MOV DWORD PTR SS:[EBP-30],ECX
00455276  |. >MOV WORD PTR SS:[EBP-1C],8
0045527C  |. >MOV EBX,DWORD PTR SS:[EBP-30]
0045527F  |. >PUSH EBX
00455280  |. >CALL WebCatch.005432F0                               ；得到注册码位数
00455285  |. >POP ECX
00455286  |. >CMP EAX,1D                                           ；位数是否为 29 位
00455289  |. >JE SHORT WebCatch.0045528F
0045528B  |. >MOV AL,1
0045528D  |. >JMP SHORT WebCatch.0045530D
0045528F  |> >MOVSX EDX,BYTE PTR DS:[EBX+5]                         ；第 5 位是否是‘-’
00455293  |. >CMP EDX,2D
00455296  |. >JNZ SHORT WebCatch.004552B3
00455298  |. >MOVSX EAX,BYTE PTR DS:[EBX+B]                          ；第 B 位是否是‘-’
0045529C  |. >CMP EAX,2D
0045529F  |. >JNZ SHORT WebCatch.004552B3
004552A1  |. >MOVSX EDX,BYTE PTR DS:[EBX+11]                         ；第 0x11 位是否是‘-’
004552A5  |. >CMP EDX,2D
004552A8  |. >JNZ SHORT WebCatch.004552B3
004552AA  |. >MOVSX ECX,BYTE PTR DS:[EBX+17]                         ；第 0x17 位是否是‘-’
004552AE  |. >CMP ECX,2D
004552B1  |. >JE SHORT WebCatch.004552B7
004552B3  |> >MOV AL,2
004552B5  |. >JMP SHORT WebCatch.0045530D
004552B7  |> >PUSH 5                                                ; /Arg3 = 00000005
004552B9  |. >PUSH EBX                                             ; |Arg2
004552BA  |. >PUSH ESI                                             ; |Arg1
004552BB  |. >CALL WebCatch.0054314C                               ; \WebCatch.0054314C
004552C0  |. >ADD ESP,0C
004552C3  |. >LEA EDX,DWORD PTR DS:[EBX+6]
004552C6  |. >PUSH 5                                              ; /Arg3 = 00000005
004552C8  |. >PUSH EDX                                            ; |Arg2
004552C9  |. >PUSH WebCatch.005CA5BD                             ; |Arg1 = 005CA5BD ASCII "22222333334444455555"
004552CE  |. >CALL WebCatch.0054314C                              ; \WebCatch.0054314C
004552D3  |. >ADD ESP,0C
004552D6  |. >LEA ECX,DWORD PTR DS:[EBX+C]
004552D9  |. >PUSH 5                                              ; /Arg3 = 00000005
004552DB  |. >PUSH ECX                                            ; |Arg2
004552DC  |. >PUSH EDI                                            ; |Arg1
004552DD  |. >CALL WebCatch.0054314C                              ; \WebCatch.0054314C
004552E2  |. >ADD ESP,0C
004552E5  |. >LEA EAX,DWORD PTR DS:[EBX+12]
004552E8  |. >PUSH 5                                              ; /Arg3 = 00000005
004552EA  |. >PUSH EAX                                            ; |Arg2
004552EB  |. >PUSH WebCatch.005CA5C7                             ; |Arg1 = 005CA5C7 ASCII "4444455555"
004552F0  |. >CALL WebCatch.0054314C                              ; \WebCatch.0054314C
004552F5  |. >ADD ESP,0C
004552F8  |. >ADD EBX,18
004552FB  |. >PUSH 5                                              ; /Arg3 = 00000005
004552FD  |. >PUSH EBX                                            ; |Arg2
004552FE  |. >PUSH WebCatch.005CA5CC                             ; |Arg1 = 005CA5CC ASCII "55555"
00455303  |. >CALL WebCatch.0054314C                              ; \WebCatch.0054314C

      ---->这几行代码是去掉注册码中间的‘-’号

00455308  |. >ADD ESP,0C
0045530B  |. >XOR EAX,EAX
0045530D  |> >TEST AL,AL
0045530F  |. >JE SHORT WebCatch.00455334
00455311  |. >XOR EAX,EAX
00455313  |. >MOV EDX,2
00455318  |. >PUSH EAX
00455319  |. >LEA EAX,DWORD PTR SS:[EBP-4]
0045531C  |. >DEC DWORD PTR SS:[EBP-10]
0045531F  |. >CALL WebCatch.0055053C
00455324  |. >POP EAX
00455325  |. >MOV EDX,DWORD PTR SS:[EBP-2C]
00455328  |. >MOV DWORD PTR FS:[0],EDX
0045532F  |. >JMP WebCatch.0045592D
00455334  |> >MOV ECX,DWORD PTR SS:[EBP-30]                       ；注册码入栈
00455337  |. >PUSH ECX                                             ; /Arg1
00455338  |. >CALL WebCatch.0045595C                              ; \WebCatch.0045595C  比较注册码的第三部分是否是数字
0045533D  |. >POP ECX
0045533E  |. >TEST EAX,EAX
00455340  |. >JE SHORT WebCatch.00455365
00455342  |. >XOR EAX,EAX
00455344  |. >MOV EDX,2
00455349  |. >PUSH EAX
```



```
0045534A  .| >LEA EAX,DWORD PTR SS:[EBP-4]

0045534D  .| >DEC DWORD PTR SS:[EBP-10]

00455350  .| >CALL WebCatch.0055053C

00455355  .| >POP EAX

00455356  .| >MOV EDX,DWORD PTR SS:[EBP-2C]

00455359  .| >MOV DWORD PTR FS:[0],EDX

00455360  .| >JMP WebCatch.0045592D

00455365  |> >MOVSX EDX,BYTE PTR DS:[EDI+1]

00455369  .| >MOVSX ECX,BYTE PTR DS:[EDI+2]

0045536D  .| >ADD EDX,EDX                  ；将去掉‘-’后的注册码的第 12 位自相加

0045536F  .| >MOVSX EBX,BYTE PTR DS:[EDI+4]         ；第 15 位送入 `ebx`

00455373  .| >LEA EDX,DWORD PTR DS:[EDX+EDX*4]      ；`edx = edx * 5`

00455376  .| >MOVSX EAX,BYTE PTR DS:[EDI]

00455379  .| >ADD EDX,ECX                  ；`edx = edx + 第 13 位`

0045537B  .| >ADD EAX,-30                  ；`eax` 转换为数字

0045537E  .| >MOVSX ECX,BYTE PTR DS:[EDI+3]         ；`ecx = 第 14 位注册码`

00455382  .| >ADD ECX,ECX                  ；`ecx` 自加

00455384  .| >ADD EDX,-210                 ；转换为数字结果

0045538A  .| >LEA ECX,DWORD PTR DS:[ECX+ECX*4]      ；`ecx = ecx * 5`

0045538D  .| >ADD ECX,EBX

0045538F  .| >ADD ECX,-210                 ；转换为数字结果

00455395  .| >CMP EAX,1                   ；下面几行检测 `eax` 是否是 1 或 9，如果是就会注册失败，也就是说去掉‘-’后的第 11 位不能是 1 或 9

00455398  .| >MOV DWORD PTR SS:[EBP-34],ECX          ；将计算结果保存

0045539B  .| >JLE SHORT WebCatch.004553B7

0045539D  .| >CMP EAX,9

004553A0  .| >JG SHORT WebCatch.004553B7

004553A2  .| >TEST EDX,EDX

004553A4  .| >JLE SHORT WebCatch.004553B7           ；如果 `edx = 0` 则会错误

004553A6  .| >CMP EDX,0C

004553A9  .| >JG SHORT WebCatch.004553B7            ；如果 `edx <= 0c` 不会出错

004553AB  .| >CMP DWORD PTR SS:[EBP-34],0

004553AF  .| >JLE SHORT WebCatch.004553B7

004553B1  .| >CMP DWORD PTR SS:[EBP-34],1F           ；如果在 455398 处保存的结果不是 `<=0x1f` 则会注册失败

004553B5  .| >JLE SHORT WebCatch.004553BB

004553B7  |> >MOV AL,4

004553B9  .| >JMP SHORT WebCatch.004553BD

004553BB  |> >XOR EAX,EAX

004553BD  |> >TEST AL,AL

004553BF  .| >JE SHORT WebCatch.004553E4

004553C1  .| >XOR EAX,EAX

004553C3  .| >MOV EDX,2

004553C8  .| >PUSH EAX

004553C9  .| >LEA EAX,DWORD PTR SS:[EBP-4]

004553CC  .| >DEC DWORD PTR SS:[EBP-10]

004553CF  .| >CALL WebCatch.0055053C

004553D4  .| >POP EAX

004553D5  .| >MOV EDX,DWORD PTR SS:[EBP-2C]

004553D8  .| >MOV DWORD PTR FS:[0],EDX

004553DF  .| >JMP WebCatch.0045592D

004553E4  |> >MOVSX ECX,BYTE PTR DS:[ESI]            ；取‘-’后注册码的第 1 位

004553E7  .| >MOVSX EAX,BYTE PTR DS:[5CA5BD]         ；取‘-’后注册码的第 6 位

004553EE  .| >ADD ECX,EAX                  ；1，6 位相加

004553F0  .| >MOVSX EDX,BYTE PTR DS:[EDI]            ；取‘-’后注册码的第 11 位

004553F3  .| >ADD ECX,EDX                  ；相加

004553F5  .| >INC ECX                      ；`ecx = ecx + 1`

004553F6  .| >MOV DWORD PTR SS:[EBP-48],ECX          ；保存结果

004553F9  .| >FILD DWORD PTR SS:[EBP-48]

004553FC  .| >ADD ESP,-8                   ; /

004553FF  .| >FSTP QWORD PTR SS:[ESP]       ; |Arg1 (8-byte)

00455402  .| >CALL WebCatch.00549AE8       ; \WebCatch.00549AE8    对 4553f6 保存的结果求出 sin 值

00455407  .| >ADD ESP,8

0045540A  .| >ADD ESP,-8                   ; /

0045540D  .| >FSTP QWORD PTR SS:[ESP]       ; |Arg1 (8-byte)

00455410  .| >CALL WebCatch.00548698       ; \WebCatch.00548698    ；取，刚刚计算的 sin 绝对值

00455415  .| >ADD ESP,8

00455418  .| >FMUL DWORD PTR DS:[455934]     ；浮点运算刚刚取得绝对值 * 1101

0045541E  .| >CALL WebCatch.005486C0       ；把浮点数转换为整数保存在 `eax` 中

00455423  .| >MOV ECX,1A

00455428  .| >CDQ

00455429  .| >IDIV ECX

0045542B  .| >ADD DL,41

0045542E  .| >MOV BYTE PTR SS:[EBP-3C],DL    ；保存结果

00455431  .| >MOVSX EAX,BYTE PTR DS:[ESI+1]          ；取‘-’后注册码的第 2 位

00455435  .| >MOVSX EDX,BYTE PTR DS:[5CA5BE]        ；取‘-’后注册码的第 7 位

0045543C  .| >ADD EAX,EDX

0045543E  .| >MOVSX ECX,BYTE PTR DS:[EDI+1]          ；取‘-’后注册码的第 12 位

00455442  .| >ADD EAX,ECX

00455444  .| >ADD EAX,2

00455447  .| >MOV DWORD PTR SS:[EBP-4C],EAX

0045544A  .| >FILD DWORD PTR SS:[EBP-4C]

0045544D  .| >ADD ESP,-8                   ; /

00455450  .| >FSTP QWORD PTR SS:[ESP]       ; |Arg1 (8-byte)

00455453  .| >CALL WebCatch.00549AE8       ; \WebCatch.00549AE8

00455458  .| >ADD ESP,8

0045545B  .| >ADD ESP,-8                   ; /

0045545E  .| >FSTP QWORD PTR SS:[ESP]       ; |Arg1 (8-byte)

00455461  .| >CALL WebCatch.00548698       ; \WebCatch.00548698

00455466  .| >ADD ESP,8

00455469  .| >FMUL DWORD PTR DS:[455938]     ；浮点运算刚刚取得绝对值 * 1019

0045546F  .| >CALL WebCatch.005486C0

00455474  .| >MOV ECX,1A

00455479  .| >CDQ

0045547A  .| >IDIV ECX

0045547C  .| >ADD DL,41

0045547F  .| >MOV BYTE PTR SS:[EBP-3B],DL    ；保存结果

00455482  .| >MOVSX EAX,BYTE PTR DS:[ESI+2]           ；取去‘-’后注册码的第 3 位

00455486  .| >MOVSX EDX,BYTE PTR DS:[5CA5BF]          ；取去‘-’后注册码的第 8 位

0045548D  .| >ADD EAX,EDX

0045548F  .| >MOVSX ECX,BYTE PTR DS:[EDI+2]           ；取去‘-’后注册码的第 13 位

00455493  .| >ADD EAX,ECX

00455495  .| >ADD EAX,3

00455498  .| >MOV DWORD PTR SS:[EBP-50],EAX

0045549B  .| >FILD DWORD PTR SS:[EBP-50]

0045549E  .| >ADD ESP,-8                   ; /

004554A1  .| >FSTP QWORD PTR SS:[ESP]       ; |Arg1 (8-byte)

004554A4  .| >CALL WebCatch.00549AE8       ; \WebCatch.00549AE8

004554A9  .| >ADD ESP,8

004554AC  .| >ADD ESP,-8                   ; /

004554AF  .| >FSTP QWORD PTR SS:[ESP]       ; |Arg1 (8-byte)

004554B2  .| >CALL WebCatch.00548698       ; \WebCatch.00548698

004554B7  .| >ADD ESP,8

004554BA  .| >FMUL DWORD PTR DS:[45593C]     ；浮点运算刚刚取得绝对值 * 1013

004554C0  .| >CALL WebCatch.005486C0

004554C5  .| >MOV ECX,1A

004554CA  .| >CDQ

004554CB  .| >IDIV ECX

004554CD  .| >ADD DL,41

004554D0  .| >MOV BYTE PTR SS:[EBP-3A],DL    ；保存结果

004554D3  .| >MOVSX EAX,BYTE PTR DS:[ESI+3]           ；取去‘-’后注册码的第 4 位

004554D7  .| >MOVSX EDX,BYTE PTR DS:[5CA5C0]          ；取去‘-’后注册码的第 9 位

004554DE  .| >ADD EAX,EDX

004554E0  .| >MOVSX ECX,BYTE PTR DS:[EDI+3]           ；取去‘-’后注册码的第 14 位

004554E4  .| >ADD EAX,ECX

004554E6  .| >ADD EAX,4

004554E9  .| >MOV DWORD PTR SS:[EBP-54],EAX

004554EC  .| >FILD DWORD PTR SS:[EBP-54]

004554EF  .| >ADD ESP,-8                   ; /

004554F2  .| >FSTP QWORD PTR SS:[ESP]       ; |Arg1 (8-byte)

004554F5  .| >CALL WebCatch.00549AE8       ; \WebCatch.00549AE8

004554FA  .| >ADD ESP,8

004554FD  .| >ADD ESP,-8                   ; /

00455500  .| >FSTP QWORD PTR SS:[ESP]       ; |Arg1 (8-byte)

00455503  .| >CALL WebCatch.00548698       ; \WebCatch.00548698

00455508  .| >ADD ESP,8

0045550B  .| >FMUL DWORD PTR DS:[455940]     ；浮点运算刚刚取得绝对值 * 1116

00455511  .| >CALL WebCatch.005486C0

00455516  .| >MOV ECX,1A

0045551B  .| >CDQ

0045551C  .| >IDIV ECX

0045551E  .| >ADD DL,41

00455521  .| >MOV BYTE PTR SS:[EBP-39],DL    ；保存结果

00455524  .| >MOVSX EAX,BYTE PTR DS:[ESI+4]           ；取去‘-’后注册码的第 5 位

00455528  .| >MOVSX EDX,BYTE PTR DS:[5CA5C1]          ；取去‘-’后注册码的第 10 位

0045552F  .| >ADD EAX,EDX

00455531  .| >MOVSX ECX,BYTE PTR DS:[EDI+4]           ；取去‘-’后注册码的第 15 位

00455535  .| >ADD EAX,ECX

00455537  .| >ADD EAX,5

0045553A  .| >MOV DWORD PTR SS:[EBP-58],EAX

0045553D  .| >FILD DWORD PTR SS:[EBP-58]

00455540  .| >ADD ESP,-8                   ; /
```



```
00455543  |. >FSTP QWORD PTR SS:\[ESP\]               ; |Arg1 (8-byte)   

00455546  |. >CALL WebCatch.00549AE8                ; \\WebCatch.00549AE8   

0045554B  |. >ADD ESP,8   

0045554E  |. >ADD ESP,-8                            ; /   

00455551  |. >FSTP QWORD PTR SS:\[ESP\]               ; |Arg1 (8-byte)   

00455554  |. >CALL WebCatch.00548698                ; \\WebCatch.00548698   

00455559  |. >ADD ESP,8   

0045555C  |. >FMUL DWORD PTR DS:\[455944\]            ；浮点运算刚刚取得绝对值 \* 1381   

00455562  |. >CALL WebCatch.005486C0   

00455567  |. >MOV ECX,1A   

0045556C  |. >CDQ   

0045556D  |. >IDIV ECX                              ；除以 1ah 的余数存入 dx   

0045556F  |. >ADD DL,41                             ；加上 41h   

00455572  |. >MOV BYTE PTR SS:\[EBP\-38\],DL           ；保存结果，并于假码的第 21-25 位进行比较   

00455575  |. >PUSH 5                                ; /Arg3 = 00000005   

00455577  |. >PUSH WebCatch.005CA5C7                ; |Arg2 = 005CA5C7 ASCII "4444455555"   

0045557C  |. >LEA EAX,DWORD PTR SS:\[EBP\-3C\]         ; |   

0045557F  |. >PUSH EAX                              ; |Arg1   

00455580  |. >CALL WebCatch.0054341C                ; \\WebCatch.0054341C   

00455585  |. >ADD ESP,0C   

00455588  |. >TEST EAX,EAX   

0045558A  |. >JE SHORT WebCatch.00455590   

0045558C  |. >MOV DL,5   

0045558E  |. >JMP SHORT WebCatch.00455592   

00455590  |> >XOR EDX,EDX   

00455592  |> >TEST DL,DL   

00455594  |. >JE SHORT WebCatch.004555B9   

00455596  |. >XOR EAX,EAX   

00455598  |. >MOV EDX,2   

0045559D  |. >PUSH EAX   

0045559E  |. >LEA EAX,DWORD PTR SS:\[EBP\-4\]   

004555A1  |. >DEC DWORD PTR SS:\[EBP\-10\]   

004555A4  |. >CALL WebCatch.0055053C   

004555A9  |. >POP EAX   

004555AA  |. >MOV EDX,DWORD PTR SS:\[EBP\-2C\]   

004555AD  |. >MOV DWORD PTR FS:\[0\],EDX   

004555B4  |. >JMP WebCatch.0045592D   

004555B9  |> >MOVSX ECX,BYTE PTR DS:\[EDI+3\]         ；取去‘-’后注册码的第 14 位           

004555BD  |. >MOVSX EAX,BYTE PTR DS:\[EDI+4\]         ；取去‘-’后注册码的第 15 位   

004555C1  |. >MOV EBX,ECX   

004555C3  |. >MOVSX EDX,BYTE PTR DS:\[ESI\]           ；取去‘-’后注册码的第 1 位   

004555C6  |. >MOVSX ECX,BYTE PTR DS:\[5CA5BD\]        ；取去‘-’后注册码的第 6 位   

004555CD  |. >ADD EBX,EBX   

004555CF  |. >ADD EDX,ECX   

004555D1  |. >MOVSX ECX,BYTE PTR DS:\[5CA5C7\]        ；取去‘-’后注册码的第 16 位   

004555D8  |. >LEA EBX,DWORD PTR DS:\[EBX+EBX\*4\]   

004555DB  |. >ADD EBX,EAX                           ；此处就算结果以后 455615 处使用   

004555DD  |. >MOVSX EAX,BYTE PTR DS:\[EDI\]           ；取去‘-’后注册码的第 11 位   

004555E0  |. >ADD EDX,EAX   

004555E2  |. >ADD EBX,-210   

004555E8  |. >ADD EDX,ECX   

004555EA  |. >ADD EDX,6   

004555ED  |. >ADD ESP,-8   

004555F0  |. >MOV DWORD PTR SS:\[EBP\-5C\],EDX         ; |   

004555F3  |. >FILD DWORD PTR SS:\[EBP\-5C\]            ; |   

004555F6  |. >FSTP QWORD PTR SS:\[ESP\]               ; |   

004555F9  |. >CALL WebCatch.00548560                ; \\WebCatch.00548560  求出 cos 值   

004555FE  |. >ADD ESP,8   

00455601  |. >ADD ESP,-8                            ; /   

00455604  |. >FSTP QWORD PTR SS:\[ESP\]               ; |Arg1 (8-byte)   

00455607  |. >CALL WebCatch.00548698                ; \\WebCatch.00548698  取决对值   

0045560C  |. >ADD ESP,8   

0045560F  |. >FMUL DWORD PTR DS:\[455948\]            ；浮点运算刚刚取得绝对值 \* 1016   

00455615  |. >MOV DWORD PTR SS:\[EBP\-60\],EBX   

00455618  |. >FILD DWORD PTR SS:\[EBP\-60\]   

0045561B  |. >FMULP ST(1),ST                        ；相乘   

0045561D  |. >CALL WebCatch.005486C0   

00455622  |. >MOV ECX,1A   

00455627  |. >CDQ   

00455628  |. >IDIV ECX                              ；除以 1ah 的余数存入 dx   

0045562A  |. >ADD DL,41                             ；保存结果   

0045562D  |. >MOV BYTE PTR SS:\[EBP\-44\],DL   

00455630  |. >MOVSX EAX,BYTE PTR DS:\[ESI+1\]         ；取去‘-’后注册码的第 2 位   

00455634  |. >MOVSX EDX,BYTE PTR DS:\[5CA5BE\]        ；取去‘-’后注册码的第 7 位   

0045563B  |. >ADD EAX,EDX   

0045563D  |. >MOVSX ECX,BYTE PTR DS:\[EDI+1\]         ；取去‘-’后注册码的第 17 位   

00455641  |. >ADD EAX,ECX   

00455643  |. >MOVSX EDX,BYTE PTR DS:\[5CA5C8\]        ；取去‘-’后注册码的第 12 位，下面的计算同上   

0045564A  |. >ADD EAX,EDX   

0045564C  |. >ADD EAX,7   

0045564F  |. >MOV DWORD PTR SS:\[EBP\-64\],EAX   

00455652  |. >FILD DWORD PTR SS:\[EBP\-64\]   

00455655  |. >ADD ESP,-8                            ; /   

00455658  |. >FSTP QWORD PTR SS:\[ESP\]               ; |Arg1 (8-byte)   

0045565B  |. >CALL WebCatch.00548560                ; \\WebCatch.00548560   

00455660  |. >ADD ESP,8   

00455663  |. >ADD ESP,-8                            ; /   

00455666  |. >FSTP QWORD PTR SS:\[ESP\]               ; |Arg1 (8-byte)   

00455669  |. >CALL WebCatch.00548698                ; \\WebCatch.00548698   

0045566E  |. >ADD ESP,8   

00455671  |. >FMUL DWORD PTR DS:\[45594C\]            ；浮点运算刚刚取得绝对值 \* 1207   

00455677  |. >MOV DWORD PTR SS:\[EBP\-68\],EBX   

0045567A  |. >FILD DWORD PTR SS:\[EBP\-68\]   

0045567D  |. >FMULP ST(1),ST   

0045567F  |. >CALL WebCatch.005486C0   

00455684  |. >MOV ECX,1A   

00455689  |. >CDQ   

0045568A  |. >IDIV ECX   

0045568C  |. >ADD DL,41   

0045568F  |. >MOV BYTE PTR SS:\[EBP\-43\],DL            ；保存结果   

00455692  |. >MOVSX EAX,BYTE PTR DS:\[ESI+2\]          ；取去‘-’后注册码的第 3 位   

00455696  |. >MOVSX EDX,BYTE PTR DS:\[5CA5BF\]         ；取去‘-’后注册码的第 8 位   

0045569D  |. >ADD EAX,EDX   

0045569F  |. >MOVSX ECX,BYTE PTR DS:\[EDI+2\]          ；取去‘-’后注册码的第 18 位   

004556A3  |. >ADD EAX,ECX   

004556A5  |. >MOVSX EDX,BYTE PTR DS:\[5CA5C9\]         ；取去‘-’后注册码的第 13 位 ，下面的计算同上   

004556AC  |. >ADD EAX,EDX   

004556AE  |. >ADD EAX,8   

004556B1  |. >MOV DWORD PTR SS:\[EBP\-6C\],EAX   

004556B4  |. >FILD DWORD PTR SS:\[EBP\-6C\]   

004556B7  |. >ADD ESP,-8                            ; /   

004556BA  |. >FSTP QWORD PTR SS:\[ESP\]               ; |Arg1 (8-byte)   

004556BD  |. >CALL WebCatch.00548560                ; \\WebCatch.00548560   

004556C2  |. >ADD ESP,8   

004556C5  |. >ADD ESP,-8                            ; /   

004556C8  |. >FSTP QWORD PTR SS:\[ESP\]               ; |Arg1 (8-byte)   

004556CB  |. >CALL WebCatch.00548698                ; \\WebCatch.00548698   

004556D0  |. >ADD ESP,8   

004556D3  |. >FMUL DWORD PTR DS:\[455950\]            ；浮点运算刚刚取得绝对值 \* 1084   

004556D9  |. >MOV DWORD PTR SS:\[EBP\-70\],EBX   

004556DC  |. >FILD DWORD PTR SS:\[EBP\-70\]   

004556DF  |. >FMULP ST(1),ST   

004556E1  |. >CALL WebCatch.005486C0   

004556E6  |. >MOV ECX,1A   

004556EB  |. >CDQ   

004556EC  |. >IDIV ECX   

004556EE  |. >ADD DL,41   

004556F1  |. >MOV BYTE PTR SS:\[EBP\-42\],DL            ；保存结果   

004556F4  |. >MOVSX EAX,BYTE PTR DS:\[ESI+3\]          ；取去‘-’后注册码的第 4 位   

004556F8  |. >MOVSX EDX,BYTE PTR DS:\[5CA5C0\]         ；取去‘-’后注册码的第 9 位   

004556FF  |. >ADD EAX,EDX   

00455701  |. >MOVSX ECX,BYTE PTR DS:\[EDI+3\]          ；取去‘-’后注册码的第 19 位     

00455705  |. >ADD EAX,ECX   

00455707  |. >MOVSX EDX,BYTE PTR DS:\[5CA5CA\]         ；取去‘-’后注册码的第 13 位 ，下面的计算同上   

0045570E  |. >ADD EAX,EDX   

00455710  |. >ADD EAX,9   

00455713  |. >MOV DWORD PTR SS:\[EBP\-74\],EAX   

00455716  |. >FILD DWORD PTR SS:\[EBP\-74\]   

00455719  |. >ADD ESP,-8                            ; /   

0045571C  |. >FSTP QWORD PTR SS:\[ESP\]               ; |Arg1 (8-byte)   

0045571F  |. >CALL WebCatch.00548560                ; \\WebCatch.00548560   

00455724  |. >ADD ESP,8   

00455727  |. >ADD ESP,-8                            ; /   

0045572A  |. >FSTP QWORD PTR SS:\[ESP\]               ; |Arg1 (8-byte)   

0045572D  |. >CALL WebCatch.00548698                ; \\WebCatch.00548698   

00455732  |. >ADD ESP,8   

00455735  |. >FMUL DWORD PTR DS:\[455954\]            ；浮点运算刚刚取得绝对值 \* 1059   

0045573B  |. >MOV DWORD PTR SS:\[EBP\-78\],EBX   

0045573E  |. >FILD DWORD PTR SS:\[EBP\-78\]   

00455741  |. >FMULP ST(1),ST   

00455743  |. >CALL WebCatch.005486C0   

00455748  |. >MOV ECX,1A   

0045574D  |. >CDQ   

0045574E  |. >IDIV ECX   

00455750  |. >ADD DL,41   
```



00455753 `|. >MOV BYTE PTR SS:[EBP-41],DL` ；保存结果

00455756 `|. >MOVSX EAX,BYTE PTR DS:[ESI+4]` ；取去‘-’后注册码的第 5 位

0045575A `|. >MOVSX EDX,BYTE PTR DS:[5CA5C1]` ；取去‘-’后注册码的第 10 位

00455761 `|. >ADD EAX,EDX`

00455763 `|. >MOVSX ECX,BYTE PTR DS:[EDI+4]` ；取去‘-’后注册码的第 20 位

00455767 `|. >ADD EAX,ECX`

00455769 `|. >MOVSX EDX,BYTE PTR DS:[5CA5CB]` ；取去‘-’后注册码的第 14 位，注意最后一位注册码的计算方法与前面不同

00455770 `|. >ADD EAX,EDX`

00455772 `|. >MOV DWORD PTR SS:[EBP-7C],EAX`

00455775 `|. >FILD DWORD PTR SS:[EBP-7C]`

00455778 `|. >ADD ESP,-8` ; /

0045577B `|. >FSTP QWORD PTR SS:[ESP]` ; |Arg1 (8-byte)

0045577E `|. >CALL WebCatch.00548560` ; \WebCatch.00548560

00455783 `|. >ADD ESP,8`

00455786 `|. >ADD ESP,-8` ; /

00455789 `|. >FSTP QWORD PTR SS:[ESP]` ; |Arg1 (8-byte)

0045578C `|. >CALL WebCatch.00548698` ; \WebCatch.00548698

00455791 `|. >ADD ESP,8`

00455794 `|. >FMUL DWORD PTR DS:[455958]` ；浮点运算刚刚取得绝对值 * 1190

0045579A `|. >MOV DWORD PTR SS:[EBP-80],EBX`

0045579D `|. >FILD DWORD PTR SS:[EBP-80]`

004557A0 `|. >FMULP ST(1),ST`

004557A2 `|. >CALL WebCatch.005486C0`

004557A7 `|. >MOV ECX,1A`

004557AC `|. >CDQ`

004557AD `|. >IDIV ECX`

004557AF `|. >ADD DL,41`

004557B2 `|. >MOV BYTE PTR SS:[EBP-40],DL` ；保存结果，下面会与假码的最后五位进行比较

004557B5 `|. >PUSH 5` ; /Arg3 = 00000005

004557B7 `|. >PUSH WebCatch.005CA5CC` ; |Arg2 = 005CA5CC ASCII "55555"

004557BC `|. >LEA EAX,DWORD PTR SS:[EBP-44]` ; |

004557BF `|. >PUSH EAX` ; |Arg1

004557C0 `|. >CALL WebCatch.0054341C` ; \WebCatch.0054341C

004557C5 `|. >ADD ESP,0C`

004557C8 `|. >TEST EAX,EAX`

004557CA `|. >JE SHORT WebCatch.004557D0`

004557CC `|. >MOV AL,6`

004557CE `|. >JMP SHORT WebCatch.004557D2`

004557D0 `|> >XOR EAX,EAX`

004557D2 `|> >TEST AL,AL`

004557D4 `|. >JE SHORT WebCatch.004557F9`

004557D6 `|. >XOR EAX,EAX`

004557D8 `|. >MOV EDX,2`

004557DD `|. >PUSH EAX`

004557DE `|. >LEA EAX,DWORD PTR SS:[EBP-4]`

004557E1 `|. >DEC DWORD PTR SS:[EBP-10]`

004557E4 `|. >CALL WebCatch.0055053C`

004557E9 `|. >POP EAX`

004557EA `|. >MOV EDX,DWORD PTR SS:[EBP-2C]`

004557ED `|. >MOV DWORD PTR FS:[0],EDX`

004557F4 `|. >JMP WebCatch.0045592D`

004557F9 `|> >CMP BYTE PTR DS:[5CA645],0`

00455800 `|. >JE SHORT WebCatch.0045584B`

00455802 `|. >PUSH 5`

00455804 `|. >CMP DWORD PTR DS:[5CA614],0`

0045580B `|. >JE SHORT WebCatch.00455815`

0045580D `|. >MOV ECX,DWORD PTR DS:[5CA614]`

00455813 `|. >JMP SHORT WebCatch.0045581A`

00455815 `|> >MOV ECX,WebCatch.005A2D41`

0045581A `|> >PUSH ECX` ; |Arg2

0045581B `|. >PUSH ESI` ; |Arg1

0045581C `|. >CALL WebCatch.0054341C` ; \WebCatch.0054341C

00455821 `|. >ADD ESP,0C`

00455824 `|. >TEST EAX,EAX`

00455826 `|. >JE SHORT WebCatch.0045584B`

00455828 `|. >XOR EAX,EAX`

0045582A `|. >MOV EDX,2`

0045582F `|. >PUSH EAX`

00455830 `|. >LEA EAX,DWORD PTR SS:[EBP-4]`

00455833 `|. >DEC DWORD PTR SS:[EBP-10]`

00455836 `|. >CALL WebCatch.0055053C`

0045583B `|. >POP EAX`

0045583C `|. >MOV EDX,DWORD PTR SS:[EBP-2C]`

0045583F `|. >MOV DWORD PTR FS:[0],EDX`

00455846 `|. >JMP WebCatch.0045592D` ；下面的语句是比较注册码的前五位是否是"ELIBA"，"ELIBB"，"ELIBC"，"ELIBD"，"ELIBT"，"ELIBZ"，"ZLINE"，"ZTEST"

如果不是就会出错

0045584B `|> >PUSH 5` ; /Arg3 = 00000005

0045584D `|. >PUSH WebCatch.005A2D42` ; |Arg2 = 005A2D42 ASCII "ELIBA"

00455852 `|. >PUSH ESI` ; |Arg1

00455853 `|. >CALL WebCatch.0054341C` ; \WebCatch.0054341C

00455858 `|. >ADD ESP,0C`

0045585B `|. >TEST EAX,EAX`

0045585D `|. >JE WebCatch.004558EF`

00455863 `|. >PUSH 5` ; /Arg3 = 00000005

00455865 `|. >PUSH WebCatch.005A2D48` ; |Arg2 = 005A2D48 ASCII "ELIBB"

0045586A `|. >PUSH ESI` ; |Arg1

0045586B `|. >CALL WebCatch.0054341C` ; \WebCatch.0054341C

00455870 `|. >ADD ESP,0C`

00455873 `|. >TEST EAX,EAX`

00455875 `|. >JE SHORT WebCatch.004558EF`

00455877 `|. >PUSH 5` ; /Arg3 = 00000005

00455879 `|. >PUSH WebCatch.005A2D4E` ; |Arg2 = 005A2D4E ASCII "ELIBC"

0045587E `|. >PUSH ESI` ; |Arg1

0045587F `|. >CALL WebCatch.0054341C` ; \WebCatch.0054341C

00455884 `|. >ADD ESP,0C`

00455887 `|. >TEST EAX,EAX`

00455889 `|. >JE SHORT WebCatch.004558EF`

0045588B `|. >PUSH 5` ; /Arg3 = 00000005

0045588D `|. >PUSH WebCatch.005A2D54` ; |Arg2 = 005A2D54 ASCII "ELIBD"

00455892 `|. >PUSH ESI` ; |Arg1

00455893 `|. >CALL WebCatch.0054341C` ; \WebCatch.0054341C

00455898 `|. >ADD ESP,0C`

0045589B `|. >TEST EAX,EAX`

0045589D `|. >JE SHORT WebCatch.004558EF`

0045589F `|. >PUSH 5` ; /Arg3 = 00000005

004558A1 `|. >PUSH WebCatch.005A2D5A` ; |Arg2 = 005A2D5A ASCII "ELIBT"

004558A6 `|. >PUSH ESI` ; |Arg1

004558A7 `|. >CALL WebCatch.0054341C` ; \WebCatch.0054341C

004558AC `|. >ADD ESP,0C`

004558AF `|. >TEST EAX,EAX`

004558B1 `|. >JE SHORT WebCatch.004558EF`

004558B3 `|. >PUSH 5` ; /Arg3 = 00000005

004558B5 `|. >PUSH WebCatch.005A2D60` ; |Arg2 = 005A2D60 ASCII "ELIBZ"

004558BA `|. >PUSH ESI` ; |Arg1

004558BB `|. >CALL WebCatch.0054341C` ; \WebCatch.0054341C

004558C0 `|. >ADD ESP,0C`

004558C3 `|. >TEST EAX,EAX`

004558C5 `|. >JE SHORT WebCatch.004558EF`

004558C7 `|. >PUSH 5` ; /Arg3 = 00000005

004558C9 `|. >PUSH WebCatch.005A2D66` ; |Arg2 = 005A2D66 ASCII "ZLINE"

004558CE `|. >PUSH ESI` ; |Arg1

004558CF `|. >CALL WebCatch.0054341C` ; \WebCatch.0054341C

004558D4 `|. >ADD ESP,0C`

004558D7 `|. >TEST EAX,EAX`

004558D9 `|. >JE SHORT WebCatch.004558EF`

004558DB `|. >PUSH 5` ; /Arg3 = 00000005

004558DD `|. >PUSH WebCatch.005A2D6C` ; |Arg2 = 005A2D6C ASCII "ZTEST"

004558E2 `|. >PUSH ESI` ; |Arg1

004558E3 `|. >CALL WebCatch.0054341C` ; \WebCatch.0054341C

004558E8 `|. >ADD ESP,0C`

004558EB `|. >TEST EAX,EAX`

004558ED `|. >JNZ SHORT WebCatch.0045590F`

004558EF `|> >MOV AL,1`

004558F1 `|. >MOV EDX,2`

004558F6 `|. >PUSH EAX`

004558F7 `|. >LEA EAX,DWORD PTR SS:[EBP-4]`

004558FA `|. >DEC DWORD PTR SS:[EBP-10]`

004558FD `|. >CALL WebCatch.0055053C`

00455902 `|. >POP EAX`

00455903 `|. >MOV EDX,DWORD PTR SS:[EBP-2C]`

00455906 `|. >MOV DWORD PTR FS:[0],EDX`

0045590D `|. >JMP SHORT WebCatch.0045592D`

0045590F `|> >XOR EAX,EAX`

00455911 `|. >MOV EDX,2`

00455916 `|. >PUSH EAX`

00455917 `|. >LEA EAX,DWORD PTR SS:[EBP-4]`

0045591A `|. >DEC DWORD PTR SS:[EBP-10]`

0045591D `|. >CALL WebCatch.0055053C`

00455922 `|. >POP EAX`

00455923 `|. >MOV EDX,DWORD PTR SS:[EBP-2C]`

00455926 `|. >MOV DWORD PTR FS:[0],EDX`

0045592D `|> >POP EDI`

0045592E `|. >POP ESI`

0045592F `|. >POP EBX`

00455930 `|. >MOV ESP,EBP`

00455932 `|. >POP EBP`

00455933 `\\. >RETN`

总结：整理出一个注册码供大家参考：`ELIBA-94321-81231-AUEXZ-CFUNE`



`4dd466d0e1e0cc123afbe504da2462ca`
