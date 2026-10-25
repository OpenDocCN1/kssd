
# 反汇编分析

## 代码段 1

```
00406052  |. 85F6           TEST ESI,ESI

00406054  |. 7E 17          JLE SHORT ezConver.0040606D

00406056  |> 0FBE8C04 88000> MOVSX ECX,BYTE PTR SS:[ESP+EAX+88]

0040605E  |. 894C24 14      |MOV DWORD PTR SS:[ESP+14],ECX

00406062  |. 40             |INC EAX

00406063  |. DB4424 14      |FILD DWORD PTR SS:[ESP+14]
```

`FILD src`  装入整数到`st(0)`   `st(0) <- src (mem16/mem32/mem64)`

```
00406067  |. 3BC6           |CMP EAX,ESI

00406069  |. DEC1           |FADDP ST(1),ST  将用户名经过转换后的字符串的前esi位字符（用户名长度减1）的十进制累加存到st(1)中，此处是浮点预算

0040606B  |.^7C E9          JL SHORT ezConver.00406056
```

`st【0】`的结果是`549.000000000000000000`

```
0040606D  |> D9C0           FLD ST
```

`FLD src`  装入实数到`st(0)`  `st(0) <- src (mem32/mem64/mem80)`

```
0040606F  |. D9FE           FSIN
```

`FSIN`  正弦函数`sin`  `st(0) <- SIN( st(0) )`

```
00406071  |. D9FF           FCOS
```

`FCOS`  余弦函数`Cos`  `st(0) <- COS( st(0) )`

```
00406073  |. D9FE           FSIN

00406075  |. D9FF           FCOS

00406077  |. D9FE           FSIN

00406079  |. DD5424 14      FST QWORD PTR SS:[ESP+14]
```

`FST dest`  保存实数`st(0)`到`dest`  `dest <- st(0) (mem32/mem64`

```
0040607D  |. DC1D 00FE4000  FCOMP QWORD PTR DS:[40FE00]     将结果和0比较
```

`FCOM`  实数比较  将标志位设置为 `st(0) - st(1)` 的结果标志位

`FCOM op`  实数比较  将标志位设置为 `st(0) - op (mem32/mem64)`的结果标志位

```
00406083  |. DFE0           FSTSW AX   ax得知等于7注册码长度
```

`FSTSW AX`  保存状态字的值到`AX`  `AX<- MSW`

`FSTSW dest`  保存状态字的值到`dest`  `dest<-MSW (mem16)`

```
00406085  |. F6C4 01        TEST AH,1

00406088  |. 74 23          JE SHORT ezConver.004060AD  //以下这段我在破解是跳过去了。我想作用和上面差不多

0040608A  |> DC0D F8FD4000  /FMUL QWORD PTR DS:[40FDF8]

00406090  |. D9C0           |FLD ST

00406092  |. D9FE           |FSIN

00406094  |. D9FF           |FCOS

00406096  |. D9FE           |FSIN

00406098  |. D9FF           |FCOS

0040609A  |. D9FE           |FSIN

0040609C  |. DD5424 14      |FST QWORD PTR SS:[ESP+14]

004060A0  |. DC1D 00FE4000  |FCOMP QWORD PTR DS:[40FE00]

004060A6  |. DFE0           |FSTSW AX

004060A8  |. F6C4 01        |TEST AH,1

004060AB  |.^75 DD          JNZ SHORT ezConver.0040608A

004060AD  |> 8B5424 18      MOV EDX,DWORD PTR SS:[ESP+18]

004060B1  |. 8B4424 14      MOV EAX,DWORD PTR SS:[ESP+14]

004060B5  |. 52             PUSH EDX

004060B6  |. 50             PUSH EAX                                 ; /<%.14f>

004060B7  |. 8D8C24 E000000>LEA ECX,DWORD PTR SS:[ESP+E0]            ; |

004060BE  |. 68 C4334100    PUSH ezConver.004133C4           ; |format = "%.14f"

004060C3  |. 51             PUSH ECX                                 ; |s

004060C4  |. DDD8           FSTP ST                                  ; |

004060C6  |. FF15 34F44000  CALL DWORD PTR DS:[<&MSVCRT.sprintf>]    ; sprintf

004060CC  |. 8DBC24 E800000>LEA EDI,DWORD PTR SS:[ESP+E8]
```

将算出的浮点结果的前十四位转移到`edi`中

内存中  【 `0012F2EC`   `0012F3D4`  ASCII "`0.69642527979317`"】

```
004060D3  |. 8BCD           MOV ECX,EBP

004060D5  |. 33C0           XOR EAX,EAX

004060D7  |. 83C4 10        ADD ESP,10

004060DA  |. 33D2           XOR EDX,EDX

004060DC  |. F2:AE          REPNE SCAS BYTE PTR ES:[EDI]

004060DE  |. F7D1           NOT ECX

004060E0  |. 49             DEC ECX

004060E1  |. 83E9 02        SUB ECX,2  //这上面是做些判断

004060E4  |. 74 27          JE SHORT ezConver.0040610D
```

//以下循环是关键他求出了注册码

//依次取小数部分的每一位

```
004060E6  |> 8A8414 DA00000> MOV AL,BYTE PTR SS:[ESP+EDX+DA]

004060ED  |. 8DBC24 D800000>|LEA EDI,DWORD PTR SS:[ESP+D8]

004060F4  |. 04 41          |ADD AL,41  将每一为的ascii值加上41

004060F6  |. 8BCD           |MOV ECX,EBP

004060F8  |. 888414 8800000>|MOV BYTE PTR SS:[ESP+EDX+88],AL  保存计算结果

004060FF  |. 33C0           |XOR EAX,EAX

00406101  |. 42             |INC EDX

00406102  |. F2:AE          |REPNE SCAS BYTE PTR ES:[EDI]

00406104  |. F7D1           |NOT ECX

00406106  |. 83C1 FD        |ADD ECX,-3

00406109  |. 3BD1           |CMP EDX,ECX

0040610B  |.^72 D9          JB SHORT ezConver.004060E6
```

第一位 ‘6’+41＝77DDDw

第二为  ‘9’+41＝7aDDDz

第三为   ‘6’+41＝77DDDw

```
0040610D  |> 8D8C24 8800000>LEA ECX,DWORD PTR SS:[ESP+88]
```

运算以后的结果存入`ecx`中

```
00406114  |. C68434 8800000>MOV BYTE PTR SS:[ESP+ESI+88],0
```

只取计算结果的前`esi`中存的位数再这里是7 实际是用户名长度减1

```
0040611C  |. 51             PUSH ECX

0040611D  |. FFD3           CALL EBX   将这些字符转化为大写字母

0040611F  |. 8D7C24 64      LEA EDI,DWORD PTR SS:[ESP+64]
```

将输入的假序列号入栈

```
00406123  |. 8BCD           MOV ECX,EBP

00406125  |. 33C0           XOR EAX,EAX

00406127  |. 8DB424 8800000>LEA ESI,DWORD PTR SS:[ESP+88]

0040612E  |. F2:AE          REPNE SCAS BYTE PTR ES:[EDI]

00406130  |. F7D1           NOT ECX

00406132  |. 49             DEC ECX

00406133  |. 8D7C24 64      LEA EDI,DWORD PTR SS:[ESP+64]

00406137  |. 33D2           XOR EDX,EDX

00406139  |. 89AC24 3001000>MOV DWORD PTR SS:[ESP+130],EBP

00406140  |. F3:A6          REPE CMPS BYTE PTR ES:[EDI],BYTE PTR DS:> 比较

00406142  |. 8D4C24 10      LEA ECX,DWORD PTR SS:[ESP+10]

00406146  |. 75 28          JNZ SHORT ezConver.00406170    //关键跳转

00406148  |. E8 43710000    CALL <JMP.&MFC42.#800>

0040614D  |. B0 01          MOV AL,1

0040614F  |. 8B8C24 2801000>MOV ECX,DWORD PTR SS:[ESP+128]

00406156  |. 64:890D 000000>MOV DWORD PTR FS:[0],ECX

0040615D  |. 5F             POP EDI

0040615E  |. 5E             POP ESI

0040615F  |. 5D             POP EBP

00406160  |. 5B             POP EBX

00406161  |. 8BE5           MOV ESP,EBP

00406163  |. 5D             POP EBP

00406164  |. C3             RETN

00406165  |> 89AC24 3001000>MOV DWORD PTR SS:[ESP+130],EBP

0040616C  |> 8D4C24 10      LEA ECX,DWORD PTR SS:[ESP+10]

00406170  |> E8 1B710000    CALL <JMP.&MFC42.#800>

00406175  |> 8B8C24 2801000>MOV ECX,DWORD PTR SS:[ESP+128]

0040617C  |. 5F             POP EDI

0040617D  |. 32C0           XOR AL,AL

0040617F  |. 64:890D 000000>MOV DWORD PTR FS:[0],ECX

00406186  |. 5E             POP ESI

00406187  |. 5D             POP EBP

00406188  |. 5B             POP EBX

00406189  |. 8BE5           MOV ESP,EBP

0040618B  |. 5D             POP EBP

0040618C  . C3             RETN

0040618D     90             NOP

0040618E     90             NOP

0040618F     90             NOP

00406190   > C3             RETN

00406191     90             NOP
```

## 代码段 2

```
004056D2   . FF15 68F04000  CALL DWORD PTR DS:[<&KERNEL32.GetCurrent>; GetCurrentDirectoryA

004056D8   . 8BCE           MOV ECX,ESI

004056DA   . E8 41070000    CALL ezConver.00405E20

004056DF   . 84C0           TEST AL,AL
```

程序返回到这

```
004056E1     0F85 C3000000  JNZ ezConver.004057AA  //因该是关键跳转了爆破的话改这

004056E7   . 57             PUSH EDI                                 ; /Arg1

004056E8   . 8BCE           MOV ECX,ESI                              ; |

004056EA   . E8 610D0000    CALL ezConver.00406450  ; ezConver.00406450

004056EF   . B9 1E000000    MOV ECX,1E  30次限制呵呵把它改为无穷大你也可以随便使用

004056F4   . 2BC8           SUB ECX,EAX   eax中存的是你已经使用的次数

004056F6   . 83F9 9C        CMP ECX,-64

004056F9   . 898E C4010000  MOV DWORD PTR DS:[ESI+1C4],ECX

004056FF   . 0F8C B9010000  JL ezConver.004058BE

00405705   . 83F9 01        CMP ECX,1

00405708   . 7D 53          JGE SHORT ezConver.0040575D
```

等于0你就不能用了

```
0040570A   . 57             PUSH EDI

0040570B   . 8D4C24 14      LEA ECX,DWORD PTR SS:[ESP+14]

0040570F   . E8 2C780000    CALL ezConver.0040CF40

00405714   . 8D4C24 10      LEA ECX,DWORD PTR SS:[ESP+10]

00405718   . 89BC24 C40A000>MOV DWORD PTR SS:[ESP+AC4],EDI

0040571F   . E8 347A0000    CALL <JMP.&MFC42.#2514>

00405724   . 8D4C24 74      LEA ECX,DWORD PTR SS:[ESP+74]

00405728   . C78424 C40A000>MOV DWORD PTR SS:[ESP+AC4],2

00405733   . E8 587B0000    CALL <JMP.&MFC42.#800>

00405738   . 8D4C24 70      LEA ECX,DWORD PTR SS:[ESP+70]

0040573C   . C68424 C40A000>MOV BYTE PTR SS:[ESP+AC4],1

00405744   . E8 477B0000    CALL <JMP.&MFC42.#800>

00405749   . C78424 C40A000>MOV DWORD PTR SS:[ESP+AC4],-1

00405754   . 8D4C24 10      LEA ECX,DWORD PTR SS:[ESP+10]

00405758   . E9 5C010000    JMP ezConver.004058B9

0040575D   > 8D4C24 08      LEA ECX,DWORD PTR SS:[ESP+8]

00405761   . E8 427B0000    CALL <JMP.&MFC42.#540>

00405766   . 8B8E C4010000  MOV ECX,DWORD PTR DS:[ESI+1C4]

0040576C   . 8D5424 08      LEA EDX,DWORD PTR SS:[ESP+8]
```
