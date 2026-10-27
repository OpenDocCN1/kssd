
# [原创]易语言桃木剑系列软件之――乾坤六十四卦 V2.01算法分析

# [原创]易语言桃木剑系列软件之――乾坤六十四卦 V2.01算法分析

## 【破文标题】易语言桃木剑系列软件之――乾坤六十四卦 V2.01算法分析

**【破文作者】** `CDboy`

**【作者邮箱】** `cdboy_cn@yahoo.com.cn`

**【使用工具】** `二哥修改版OD`，`PEiD`

**【软件名称】** 桃木剑系列软件之――乾坤六十四卦 V2.01

**【下载地址】** `e1eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4C8P5h3y4F1i4K6u0W2j5$3!0E0i4K6u0r3M7$3!0X3N6q4)9J5c8U0p5@1z5o6p5H3i4K6u0W2K9s2c8E0L8l9```.``

**【保护方式】** 功能限制

**【软件语言】** 易语言

**【软件简介】** 算卦

**【破解声明】** 菜鸟学习技术，仅此而已

## 【破解过程】

应二哥要求，特找了个易的东东，正在想哪里有易的，刚好就有个送上门了，呵呵^\_^

用`PEID`检测无法识别，在安装目录下找到文件`krnln.fnr`

用`OD`加载

```
00401000 >/$  E8 06000000   CALL tmjqk64g.0040100B
00401005  |.  50            PUSH EAX                                 ; /ExitCode
00401006  \.  E8 BB010000   CALL <JMP.&KERNEL32.ExitProcess>         ; \ExitProcess
0040100B  /$  55            PUSH EBP
0040100C  |.  8BEC          MOV EBP,ESP
0040100E  |.  81C4 F0FEFFFF ADD ESP,-110
00401014  |.  E9 83000000   JMP tmjqk64g.0040109C
00401019  |.  6B 72 6E 6C 6>ASCII "krnln.fnr",0
00401023  |.  6B 72 6E 6C 6>ASCII "krnln.fne",0
0040102D  |.  47 65 74 4E 6>ASCII "GetNewSock",0
00401038  |.  53 6F 66 74 7>ASCII "Software\FlySky\"
00401048  |.  45 5C 49 6E 7>ASCII "E\Install",0
00401052  |.  50 61 74 68 0>ASCII "Path",0
00401057  |.  4E 6F 74 20 6>ASCII "Not found the ke"
00401067  |.  72 6E 65 6C 2>ASCII "rnel library or "
00401077  |.  74 68 65 20 6>ASCII "the kernel libra"
00401087  |.  72 79 20 69 7>ASCII "ry is invalid!",0
00401096  |.  45 72 72 6F 7>ASCII "Error",0
```

典型的易入口代码，综上判断就是易的东东（其实看安装界面就可以猜到是易的）

`F9`运行 点注册，机器码`2244616410` 注册码`123456`

`Alt+M`打开内存镜象

内存镜像，项目 `36`

地址=`00403000`

大小=`00080000 (524288.)`

Owner=`tmjqk64g 00400000`

区段=`.ecode`                        //这里`F2`下断点 一会儿直接中断在其核心代码处

类型=`Imag 01001002`

访问=`R`

初始访问=`RWE`

立刻断在

```
00481882    55              PUSH EBP                                //音乐还蛮好听的
00481883    8BEC            MOV EBP,ESP
00481885    81EC 30000000   SUB ESP,30
0048188B    68 04000080     PUSH 80000004
00481890    6A 00           PUSH 0
00481892    68 5E754100     PUSH tmjqk64g.0041755E
00481897    68 01000000     PUSH 1
0048189C    BB 58050000     MOV EBX,558
004818A1    E8 120B0000     CALL tmjqk64g.004823B8
004818A6    83C4 10         ADD ESP,10
004818A9    68 00000000     PUSH 0
004818AE    BB 40060000     MOV EBX,640
004818B3    E8 000B0000     CALL tmjqk64g.004823B8
004818B8    83C4 04         ADD ESP,4
004818BB    68 01030080     PUSH 80000301
004818C0    6A 00           PUSH 0
004818C2    68 01000000     PUSH 1
004818C7    68 01000000     PUSH 1
004818CC    BB 20060000     MOV EBX,620
004818D1    E8 E20A0000     CALL tmjqk64g.004823B8                //取机器码前5位的16进制
004818D6    83C4 10         ADD ESP,10                                //EAX=机器码前5位的16进制=57A1
004818D9    8945 F4         MOV DWORD PTR SS:[EBP-C],EAX
004818DC    8955 F8         MOV DWORD PTR SS:[EBP-8],EDX
004818DF    894D FC         MOV DWORD PTR SS:[EBP-4],ECX        //ECX=8000401 是个剧机器码的常数
004818E2    6A FF           PUSH -1
004818E4    6A 08           PUSH 8
004818E6    68 A8000116     PUSH 160100A8
004818EB    68 01000152     PUSH 52010001
004818F0    E8 CF0A0000     CALL tmjqk64g.004823C4                //取假码
004818F5    83C4 10         ADD ESP,10
004818F8    8945 F0         MOV DWORD PTR SS:[EBP-10],EAX        //假码保存到[EBP-10]即12F738
004818FB    68 01030080     PUSH 80000301
00481900    6A 00           PUSH 0
00481902    68 06000000     PUSH 6
00481907    68 04000080     PUSH 80000004
0048190C    6A 00           PUSH 0
0048190E    8B45 F0         MOV EAX,DWORD PTR SS:[EBP-10]        //[EBP-10]移入EAX
00481911    85C0            TEST EAX,EAX
00481913    75 05           JNZ SHORT tmjqk64g.0048191A
00481915    B8 E44E4000     MOV EAX,tmjqk64g.00404EE4
0048191A    50              PUSH EAX                                //假码入栈
0048191B    68 02000000     PUSH 2
00481920    BB 34010000     MOV EBX,134
00481925    E8 8E0A0000     CALL tmjqk64g.004823B8                //取假码前6位
0048192A    83C4 1C         ADD ESP,1C
0048192D    8945 EC         MOV DWORD PTR SS:[EBP-14],EAX        //假码前6位移入[EBP-14]
00481930    8B5D F0         MOV EBX,DWORD PTR SS:[EBP-10]        //原假码再移入EBX
00481933    85DB            TEST EBX,EBX
00481935    74 09           JE SHORT tmjqk64g.00481940
00481937    53              PUSH EBX
00481938    E8 6F0A0000     CALL tmjqk64g.004823AC
0048193D    83C4 04         ADD ESP,4
00481940    68 04000080     PUSH 80000004
00481945    6A 00           PUSH 0
00481947    8B45 EC         MOV EAX,DWORD PTR SS:[EBP-14]        //[EBP-14]=假码=EAX
0048194A    85C0            TEST EAX,EAX
0048194C    75 05           JNZ SHORT tmjqk64g.00481953
0048194E    B8 E44E4000     MOV EAX,tmjqk64g.00404EE4
00481953    50              PUSH EAX
00481954    68 01000000     PUSH 1
00481959    BB 64010000     MOV EBX,164
0048195E    E8 550A0000     CALL tmjqk64g.004823B8
00481963    83C4 10         ADD ESP,10
00481966    8945 E4         MOV DWORD PTR SS:[EBP-1C],EAX
00481969    8955 E8         MOV DWORD PTR SS:[EBP-18],EDX
0048196C    8B5D EC         MOV EBX,DWORD PTR SS:[EBP-14]
0048196F    85DB            TEST EBX,EBX
00481971    74 09           JE SHORT tmjqk64g.0048197C
00481973    53              PUSH EBX
00481974    E8 330A0000     CALL tmjqk64g.004823AC                //下面有大量的浮点运算
00481979    83C4 04         ADD ESP,4
0048197C    DD45 E4         FLD QWORD PTR SS:[EBP-1C]                //假码存入[EBP-1C]
0048197F    DC35 15764100   FDIV QWORD PTR DS:[417615]                //假码除以5
00481985    DD5D DC         FSTP QWORD PTR SS:[EBP-24]                //结果出栈[EBP-24]
00481988    8B4D FC         MOV ECX,DWORD PTR SS:[EBP-4]
0048198B    8B55 F8         MOV EDX,DWORD PTR SS:[EBP-8]
0048198E    8B45 F4         MOV EAX,DWORD PTR SS:[EBP-C]        //[EBP-C]=57AE=EAX
00481991    81F9 01010080   CMP ECX,80000101                        //ECX与80000101比较
00481997    75 07           JNZ SHORT tmjqk64g.004819A0                //跳
00481999    25 FF000000     AND EAX,0FF
0048199E    EB 26           JMP SHORT tmjqk64g.004819C6
004819A0    81F9 01020080   CMP ECX,80000201                        //ECX与80000201比较
004819A6    75 03           JNZ SHORT tmjqk64g.004819AB                //跳
004819A8    98              CWDE
004819A9    EB 1B           JMP SHORT tmjqk64g.004819C6
004819AB    8945 D4         MOV DWORD PTR SS:[EBP-2C],EAX
004819AE    81F9 01030080   CMP ECX,80000301                        //ECX与80000201比较
004819B4    74 13           JE SHORT tmjqk64g.004819C9                //没跳
004819B6    81F9 01040080   CMP ECX,80000401                        //ECX与80000201比较
004819BC    75 10           JNZ SHORT tmjqk64g.004819CE                //没跳
004819BE    8955 D8         MOV DWORD PTR SS:[EBP-28],EDX
004819C1    DF6D D4         FILD QWORD PTR SS:[EBP-2C]                //57AE存入
004819C4    EB 2A           JMP SHORT tmjqk64g.004819F0                //跳
004819C6    8945 D4         MOV DWORD PTR SS:[EBP-2C],EAX
004819C9    DB45 D4         FILD DWORD PTR SS:[EBP-2C]
004819CC    EB 22           JMP SHORT tmjqk64g.004819F0
004819CE    81F9 01050080   CMP ECX,80000501
004819D4    75 05           JNZ SHORT tmjqk64g.004819DB
004819D6    D945 D4         FLD DWORD PTR SS:[EBP-2C]
004819D9    EB 15           JMP SHORT tmjqk64g.004819F0
004819DB    81F9 01060080   CMP ECX,80000601
004819E1    74 16           JE SHORT tmjqk64g.004819F9
004819E3    68 02000000     PUSH 2
004819E8    E8 DD090000     CALL tmjqk64g.004823CA
004819ED    83C4 04         ADD ESP,4
004819F0    DD5D D4         FSTP QWORD PTR SS:[EBP-2C]                //57AE的十进制22446出栈
004819F3    8B45 D4         MOV EAX,DWORD PTR SS:[EBP-2C]
004819F6    8B55 D8         MOV EDX,DWORD PTR SS:[EBP-28]
004819F9    8945 D4         MOV DWORD PTR SS:[EBP-2C],EAX
004819FC    8955 D8         MOV DWORD PTR SS:[EBP-28],EDX
004819FF    DD45 D4         FLD QWORD PTR SS:[EBP-2C]                //机器码前5位22446存入
00481A02    DC65 DC         FSUB QWORD PTR SS:[EBP-24]                //减去[EBP-24]里的数 就是减去假码除以5的商
```
