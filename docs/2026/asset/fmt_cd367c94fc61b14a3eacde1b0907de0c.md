
# Quickness 3.1

## 注册算法分析 + 注册机源代码(tc2) (15千字)

破解目标：Quickness 3.1

官方主页：`http://www.qwertysoft.com/`

软件简介：Quickness 是一个方便用户输入的小工具。如果有些文本内容是我们经常要用到的，就可以利用 Quickness 将其保存起来，并为其指定热键，这样，当我们需要用的时候，直接按热键就可以调出内容了。特别是在我们聊天的时候，有了 Quickness 的相助，可是会大大提高速度的哟。

下载页面：`http://www.hanzify.org/detail.asp?SOFT_ID=6938`（汉化版）

使用工具：PEiD 0.8、W32Dasm、Ollydbg 1.09b 汉化版、Windows 自带的计算器、32bit Calculator 1.6 by cybult、UltraEdit，另外还要保存大脑清醒^^

作者：炎之川[BCG]

时间：2003.4.13

主页：`http://skipli.yeah.net/`

声明：此文仅用于学习及交流，若要转载请保持文章完整。

首先说明，我下载的是汉化版，官方主页上不去，天空软件站好像也没有，所以英文原版没有办法找到，下面的分析都是基于汉化版的，当然从分析的角度说，无论汉化版还是英文版都是一样的。

其实头脑清醒的话，软件的算法并不是很难，只是在一个循环中要同时考虑三个寄存器中的数字的变化，以及其他一些变化的量，稍不留神就会乱掉。最好在分析的时候，随时纪录一下值的变化。

用 PEiD 0.8 征测可知软件无壳，使用 W32Dasm 分析，可以找到注册成功、失败等提示信息，稍作分析后用 OD 装入程序，在 `4029A7` 处下断点，然后 Ctrl+F2 重新载入程序，F9 运行，输入注册名及假注册码并点击“注册”：

```
Name: lovefire[BCG]
Serial: 123-456-789
```

```
004029A7  > 8B7C24 14     MOV EDI,DWORD PTR SS:[ESP+14]
;  Case 1 of switch 00402965  /在这里下断点
004029AB  . 68 1A040000   PUSH 41A                          ; /ControlID = 41A (1050.)
004029B0  . 57            PUSH EDI                          ; |hWnd
004029B1  . FF15 5C654100 CALL DWORD PTR DS:[<&USER32.GetDlgItem>] ; \GetDlgItem
004029B7  . 8BD8          MOV EBX,EAX
004029B9  . 53            PUSH EBX                          ; /hWnd
004029BA  . FF15 AC644100 CALL DWORD PTR DS:[<&USER32.GetWindowTextLengthA>] ; \GetWindowTextLengthA
004029C0  . 8BE8          MOV EBP,EAX
004029C2  . 8D45 01       LEA EAX,DWORD PTR SS:[EBP+1]
004029C5  . 50            PUSH EAX
004029C6  . E8 35500000   CALL qns31chs.00407A00
004029CB  . 83C4 04       ADD ESP,4
004029CE  . 8BF0          MOV ESI,EAX
004029D0  . 8D45 01       LEA EAX,DWORD PTR SS:[EBP+1]
004029D3  . 50            PUSH EAX                          ; /Count
004029D4  . 56            PUSH ESI                          ; |Buffer
004029D5  . 53            PUSH EBX                          ; |hWnd
004029D6  . FF15 B4644100 CALL DWORD PTR DS:[<&USER32.GetWindowTextA>] ; \GetWindowTextA
004029DC  . 56            PUSH ESI
004029DD  . 68 BC0C4100   PUSH qns31chs.00410CBC            ; ASCII "user_name"
004029E2  . C6042E 00     MOV BYTE PTR DS:[ESI+EBP],0
004029E6  . E8 C5E9FFFF   CALL qns31chs.004013B0
004029EB  . 83C4 08       ADD ESP,8
004029EE  . 56            PUSH ESI
004029EF  . E8 FC4F0000   CALL qns31chs.004079F0
004029F4  . 83C4 04       ADD ESP,4
004029F7  . B9 743F4100   MOV ECX,qns31chs.00413F74
004029FC  . 57            PUSH EDI
004029FD  . E8 6E000000   CALL qns31chs.00402A70
00402A02  . E8 C9FDFFFF   CALL qns31chs.004027D0            //关键call！F7 跟进
00402A07  . 85C0          TEST EAX,EAX                      //比较eax是否为0，为0则注册失败
00402A09  . 6A 00         PUSH 0                           ; /Style = MB_OK|MB_APPLMODAL
00402A0B  . 68 A00B4100   PUSH qns31chs.00410BA0            ; |Title = "Quickness"
00402A10  . 74 28         JE SHORT qns31chs.00402A3A        ; | //这里不能跳！！
00402A12  . 68 E80C4100   PUSH qns31chs.00410CE8            ; |Text = "正确。感谢。"
00402A17  . 57            PUSH EDI                          ; |hOwner
00402A18  . FF15 84644100 CALL DWORD PTR DS:[<&USER32.MessageBoxA>] ; \MessageBoxA
00402A1E  . 6A 01         PUSH 1                           ; /Result = 1
00402A20  . 57            PUSH EDI                          ; |hWnd
00402A21  . C705 50084100 >MOV DWORD PTR DS:[410850],0     ; |
00402A2B  . FF15 58654100 CALL DWORD PTR DS:[<&USER32.EndDialog>] ; \EndDialog
00402A31  . 33C0          XOR EAX,EAX
00402A33  . 5F            POP EDI
00402A34  . 5E            POP ESI
00402A35  . 5D            POP EBP
00402A36  . 5B            POP EBX
00402A37  . C2 1000       RETN 10
00402A3A  > 68 C80C4100   PUSH qns31chs.00410CC8            ; |Text = "注册失败。继续试用。"
00402A3F  . 57            PUSH EDI                          ; |hOwner
00402A40  . FF15 84644100 CALL DWORD PTR DS:[<&USER32.MessageBoxA>] ; \MessageBoxA
00402A46  . 33C0          XOR EAX,EAX
00402A48  . 5F            POP EDI
00402A49  . 5E            POP ESI
00402A4A  . 5D            POP EBP
00402A4B  . 5B            POP EBX
00402A4C  . C2 1000       RETN 10
00402A4F  > 8B5424 14     MOV EDX,DWORD PTR SS:[ESP+14]
00402A53  . 68 F80C4100   PUSH qns31chs.00410CF8            ; ASCII "a_register.htm"
00402A58  . 52            PUSH EDX
00402A59  . E8 92420000   CALL qns31chs.00406CF0
00402A5E  . 83C4 08       ADD ESP,8
00402A61  > 5F            POP EDI                          ; Default case of switch 00402965
00402A62  . 5E            POP ESI
00402A63  . 5D            POP EBP
00402A64  . 33C0          XOR EAX,EAX
00402A66  . 5B            POP EBX
00402A67  . C2 1000       RETN 10
```

---

跟进 `402A02` 的关键 call：

```
004027D0  /$ 81EC 94010000  SUB ESP,194
004027D6  |. 8D4424 04      LEA EAX,DWORD PTR SS:[ESP+4]
004027DA  |. 53             PUSH EBX
004027DB  |. 55             PUSH EBP
004027DC  |. 56             PUSH ESI
004027DD  |. 57             PUSH EDI
004027DE  |. 68 C8000000    PUSH 0C8
004027E3  |. 50             PUSH EAX
004027E4  |. 68 BC0C4100    PUSH qns31chs.00410CBC          ; ASCII "user_name"
004027E9  |. E8 32EBFFFF    CALL qns31chs.00401320
004027EE  |. 83C4 0C        ADD ESP,0C
004027F1  |. 85C0           TEST EAX,EAX
004027F3  |. 75 0B          JNZ SHORT qns31chs.00402800
004027F5  |. 5F             POP EDI
004027F6  |. 5E             POP ESI
004027F7  |. 5D             POP EBP
004027F8  |. 5B             POP EBX
004027F9  |. 81C4 94010000  ADD ESP,194
004027FF  |. C3             RETN
00402800  |> 8D8C24 DC00000>LEA ECX,DWORD PTR SS:[ESP+DC]
00402807  |. 68 C8000000    PUSH 0C8
0040280C  |. 51             PUSH ECX
0040280D  |. 68 B40C4100    PUSH qns31chs.00410CB4          ; ASCII "reg_key"
00402812  |. E8 09EBFFFF    CALL qns31chs.00401320
00402817  |. 83C4 0C        ADD ESP,0C
0040281A  |. 85C0           TEST EAX,EAX
0040281C  |. 75 0B          JNZ SHORT qns31chs.00402829
0040281E  |. 5F             POP EDI
0040281F  |. 5E             POP ESI
00402820  |. 5D             POP EBP
00402821  |. 5B             POP EBX
00402822  |. 81C4 94010000  ADD ESP,194
00402828  |. C3             RETN
00402829  |> 8D7C24 14      LEA EDI,DWORD PTR SS:[ESP+14]
0040282D  |. 83C9 FF        OR ECX,FFFFFFFF
00402830  |. 33C0           XOR EAX,EAX
00402832  |. F2:AE          REPNE SCAS BYTE PTR ES:[EDI]
00402834  |. F7D1           NOT ECX
00402836  |. 49             DEC ECX                         //得到注册名长度
00402837  |. 8BC1           MOV EAX,ECX                     //长度放入eax
00402839  |. 83F8 01        CMP EAX,1                       //比较是否大于1
0040283C  |. 894424 10      MOV DWORD PTR SS:[ESP+10],EAX
00402840  |. 73 0D          JNB SHORT qns31chs.0040284F     //大于1则继续
00402842  |. 33C0           XOR EAX,EAX
00402844  |. 5F             POP EDI
00402845  |. 5E             POP ESI
00402846  |. 5D             POP EBP
00402847  |. 5B             POP EBX
00402848  |. 81C4 94010000  ADD ESP,194
0040284E  |. C3             RETN
0040284F  |> 33C9           XOR ECX,ECX                     //ecx 清零，做计数器
00402851  |. BE 01000000    MOV ESI,1                       //esi 赋值 1
00402856  |. 85C0           TEST EAX,EAX
00402858  |. BD 03000000    MOV EBP,3                       //ebp 赋值 3
0040285D  |. BF 05000000    MOV EDI,5                       //edi 赋值 5
00402862  |. 76 4F          JBE SHORT qns31chs.004028B3
```

下面开始就是计算注册码的循环了，开始的时候我没有理清思路，在这里转来转去，头都大了（汗…典型的菜鸟-_-b)，后来经过仔细分析，发现这个循环的任务实际上是在计算三个值，并保存到不同的三个寄存器中，所以其他值都是无关紧要的中间变量，分析的时候，只要注意三个值的变化情况即可。

为了便于理解，我把这个循环分成三个部分，每一部分都写出了这一部分重要的中间变量和重要的寄存器的具体变化和算法。

下面均使用 C 语言的运算符表述，记住 `ESI`、`EBP`、`EDI` 这三个寄存器的值是我们需要掌握的，`EAX` 和 `EDX` 是关键的中间变量，`name[i]` 指用户名的 ASCII 值。

另外，从 C 语言运算符的优先级角度讲，其实并不需要用那么多的括号，但是我为了自己不看乱掉，所以每一运算均使用括号来分隔，不要数错了^_^

（最后差点数错的好像是我自己…狂汗-_-bbb）

```
00402864  |> 0FBE5C0C 14   /MOVSX EBX,BYTE PTR SS:[ESP+ECX+14] //逐位取注册名的ASCII值放入ebx
00402869  |. 8D0476        |LEA EAX,DWORD PTR DS:[ESI+ESI*2]  //eax=esi*2+esi=esi*3
0040286C  |. 33D2          |XOR EDX,EDX                      //edx清零
0040286E  |. C1E0 04       |SHL EAX,4                        //eax 左移4，等于4次*2运算
00402871  |. 2BC6          |SUB EAX,ESI                      //eax=eax-esi
00402873  |. BE E8030000   |MOV ESI,3E8                      //给 esi 赋值为 3E8
00402878  |. 03C3          |ADD EAX,EBX
```
