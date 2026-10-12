
# 疯狂注册表 v1.0

> 原文：[`www.pediy.com/kssd/pediy06/pediy6948.htm`](https://www.pediy.com/kssd/pediy06/pediy6948.htm)

《疯狂注册表 v1.0 Build 001》，是一款能够对注册表操作的软件，更体贴人的地方是它可以告诉你如何操作，而且还可以生成修改项安装包，让你可以更加方便的使用。而且它还提供了手工的修改，不仅包括 `Reg` 文件还包括 `INI` 文件。当然如果你感兴趣的话可以到以下地址去下载（好像说明中没有提到主页哦）：
`http://www.softreg.com.cn/shareware_view.asp?id=/4D4FF8C0-471D-429C-AB17-116E06DC2C69/`

首先我们先脱掉它 `UPx` 的壳，然后开始行动吧！说起注册，这个软件有几个后门！所以要注意啊！（不过这是破解完了的后话了 ^_^）（脱壳以后程序有 2006KB）

Let Begin:

注册名：`dengkeng[DFCG]`
注册码：`1234567890abcdefghijklmno`
软件序列号是：`0C342D4FA2A61B2D5C50E838EC`

点击“确定”，它会告诉你需要重新启动以便验证是否正确，这也是许多人不想破解重起校验的软件的原因。开始我用 `RegShot` 备份了一下，然后在扫描，果然注册信息生成在 `HKEY_LOCAL_MACHINESoftwareFolieRegReg` 下，可是这是你输入满足要求才写入注册表，如果不满足要求则不写入。刚开始输入的注册码不满 25 位，它就根本不写入注册表。我开始就上了当，然后用 `Filemon.exe` 进行跟踪，发现它在运行的时候生成了一个 `FolieReg.ldb` 文件，而关闭程序则删除它。开始还以为是注册信息。浪费了一些时间，后来才发现它不是什么注册信息。（可能是什么权限之类的？）

所以只好硬着头皮跟踪了：

```
00551095   . 8A12         MOV DL,BYTE PTR DS:[EDX] ;取注册码的第 1 位
00551097   . 8850 01      MOV BYTE PTR DS:[EAX+1],DL
0055109A   . C600 01      MOV BYTE PTR DS:[EAX],1
0055109D   . 8D55 B8      LEA EDX,DWORD PTR SS:[EBP-48]
005510A0   . 8D45 B4      LEA EAX,DWORD PTR SS:[EBP-4C]
005510A3   . E8 D81EEBFF  CALL FOLIEREG.00402F80
005510A8   . 8D45 B0      LEA EAX,DWORD PTR SS:[EBP-50]
005510AB   . 8B55 F8      MOV EDX,DWORD PTR SS:[EBP-8]
005510AE   . 8A52 18      MOV DL,BYTE PTR DS:[EDX+18] ;取注册码的 25 位
005510B1   . 8850 01      MOV BYTE PTR DS:[EAX+1],DL
005510B4   . C600 01      MOV BYTE PTR DS:[EAX],1
005510B7   . 8D55 B0      LEA EDX,DWORD PTR SS:[EBP-50]
005510BA   . 8D45 B4      LEA EAX,DWORD PTR SS:[EBP-4C]
005510BD   . B1 02        MOV CL,2
005510BF   . E8 8C1EEBFF  CALL FOLIEREG.00402F50 ;两个字符连接起来
005510C4   . 8D45 B4      LEA EAX,DWORD PTR SS:[EBP-4C]
005510C7   . 50           PUSH EAX
005510C8   . 8D45 B0      LEA EAX,DWORD PTR SS:[EBP-50]
005510CB   . 8B55 F4      MOV EDX,DWORD PTR SS:[EBP-C] ;密码表为"AAF03FF95FB68665"
005510CE   . 8A52 06      MOV DL,BYTE PTR DS:[EDX+6] ;取密码表中的第 7 个字符
005510D1   . 8850 01      MOV BYTE PTR DS:[EAX+1],DL
005510D4   . C600 01      MOV BYTE PTR DS:[EAX],1
005510D7   . 8D55 B0      LEA EDX,DWORD PTR SS:[EBP-50]
005510DA   . 8D45 AC      LEA EAX,DWORD PTR SS:[EBP-54]
005510DD   . E8 9E1EEBFF  CALL FOLIEREG.00402F80
005510E2   . 8D45 A8      LEA EAX,DWORD PTR SS:[EBP-58]
005510E5   . 8B55 F4      MOV EDX,DWORD PTR SS:[EBP-C]
005510E8   . 8A52 03      MOV DL,BYTE PTR DS:[EDX+3] ;去密码表中的第 4 个字符
005510EB   . 8850 01      MOV BYTE PTR DS:[EAX+1],DL
005510EE   . C600 01      MOV BYTE PTR DS:[EAX],1
005510F1   . 8D55 A8      LEA EDX,DWORD PTR SS:[EBP-58]
005510F4   . 8D45 AC      LEA EAX,DWORD PTR SS:[EBP-54]
005510F7   . B1 02        MOV CL,2
005510F9   . E8 521EEBFF  CALL FOLIEREG.00402F50 ;两个字符连接起来
005510FE   . 8D55 AC      LEA EDX,DWORD PTR SS:[EBP-54]
00551101   . 58           POP EAX
00551102   . 33C9         XOR ECX,ECX
00551104   . 8A08         MOV CL,BYTE PTR DS:[EAX]
00551106   . 41           INC ECX
00551107   . E8 201FEBFF  CALL FOLIEREG.0040302C ;比较两个字符是否相等
0055110C   . 0F85 8B000000 JNZ FOLIEREG.0055119D ;相等就写入注册表,不等则不写入
```

我们成功了吗？看看注册表中，已经写入了我们所输入的信息。下面就是关键了，想必重新启动以后要 `RegQueryValueExA` 注册表了！用 `W32Dasm` 察看信息，找到了如下的信息：

```
* Referenced by a CALL at Addresses:
| :00464764   , :00474F8E   , :004751EB   
|
* Reference To: advapi32.RegQueryValueExA, Ord:0000h
                      |
:00406E7C FF255C925600     Jmp dword ptr [0056925C]
:00406E82 8BC0             mov eax, eax
```

有三处调用了这个信息，我们运行程序，显然是注册信息不对，因为程序不会这么简单让我们 25 个注册信息，只是比较两个。运行完了以后，在看看注册表，里面的信息自己删除了，这就是我们用 `RegShot` 没有得到信息的原因了！因为信息被删除了，所以我们得再次输入信息：

注册名：`dengkeng[DFCG]`
注册码：`F234567890abcdefghijklmn0`

根据上面已经得出的最后两位信息是 `F0`，所以我们替换了 1,25 位。

1. 在 `00474F8E` 下断点，狂按 `F8`，到达下面：

```
00554742  |. 8D4D F0        LEA ECX,DWORD PTR SS:[EBP-10]
00554745  |. BA 504E5500    MOV EDX,FOLIEREG.00554E50     ;  ASCII "Serial"
0055474A  |. 8B45 F8        MOV EAX,DWORD PTR SS:[EBP-8]
0055474D  |. E8 AE08F2FF    CALL FOLIEREG.00475000
00554752  |. 8D55 EC        LEA EDX,DWORD PTR SS:[EBP-14]
00554755  |. 8BC3           MOV EAX,EBX
00554757  |. E8 88D2FFFF    CALL FOLIEREG.005519E4
0055475C  |. 8B45 F0        MOV EAX,DWORD PTR SS:[EBP-10]
0055475F  |. E8 E801EBFF    CALL FOLIEREG.0040494C
.....
.....从密码表中进行取数,取足 7 位
.....
00554C7E  |. B1 07          MOV CL,7
00554C80  |. E8 CBE2EAFF    CALL FOLIEREG.00402F50
00554C85  |. 8D55 90        LEA EDX,DWORD PTR SS:[EBP-70]
00554C88  |. 8D45 E8        LEA EAX,DWORD PTR SS:[EBP-18] ;一共从密码表里取了 7 位数
00554C8B  |. E8 60FCEAFF    CALL FOLIEREG.004048F0        ;以便和后面的注册码中的比较
00554C90  |. 8D45 BC        LEA EAX,DWORD PTR SS:[EBP-44] 
00554C93  |. 8B55 F0        MOV EDX,DWORD PTR SS:[EBP-10] ;注册码的地址
00554C96  |. 8A52 11        MOV DL,BYTE PTR DS:[EDX+11]   ;取其中第 12 位
00554C99  |. 8850 01        MOV BYTE PTR DS:[EAX+1],DL
.....
.....按照要求从注册码中取出 7 位,即后面的"hijklmn"
.....
00554D82  |. B1 07          MOV CL,7
00554D84  |. E8 C7E1EAFF    CALL FOLIEREG.00402F50
00554D89  |. 8D55 90        LEA EDX,DWORD PTR SS:[EBP-70]
00554D8C  |. 8D45 E0        LEA EAX,DWORD PTR SS:[EBP-20]
00554D8F  |. E8 5CFBEAFF    CALL FOLIEREG.004048F0
00554D94  |. 8D95 58FFFFFF  LEA EDX,DWORD PTR SS:[EBP-A8]
00554D9A  |. 8B45 E8        MOV EAX,DWORD PTR SS:[EBP-18]
00554D9D  |. E8 523EEBFF    CALL FOLIEREG.00408BF4 ;把密码表中取出的数字转换成大写
00554DA2  |. 8B85 58FFFFFF  MOV EAX,DWORD PTR SS:[EBP-A8]
00554DA8  |. 8B55 E0        MOV EDX,DWORD PTR SS:[EBP-20]
00554DAB  |. E8 E0FCEAFF    CALL FOLIEREG.00404A90 ;比较是否相等
00554DB0  |. 75 07          JNZ SHORT FOLIEREG.00554DB9
```

到这里我们已经得到了第二部分的注册码，替换掉原有的 `hijklmn`。

我们再次输入注册信息：

注册名：`dengkeng[DFCG]`
注册码：`F234567890abcdefg7CAC5C50`

在运行软件看看，看看标题栏（上面写着已经注册了），也没有注册窗口了！难道我们已经注册了？其实不然，这是程序给我们玩的一个花样！如果你试着点反键，选择“查看详细信息”，它会提示你注册哦！你再看看标题，又变成了未注册的样子了，我想大家应该联想到了什么吧！不错！应该又重新读取注册表的内容进行判断，所以我们必须继续跟踪注册表！

我们还是输入上面的注册信息，用 `Ollydbg` 加载。

2. `F9` 运行程序，`bp 00474F8E` 下断点，在点反键选择“查看详细信息”，被拦截了吧。下面你所要做的是狂按 `F8`：

```
.....
.....
.....
005524BB  |. 8B55 A8        MOV EDX,DWORD PTR SS:[EBP-58]
005524BE  |. B9 882A5500    MOV ECX,FOLIEREG.00552A88 ;ASCII "sTOP cRACKINg,Please ^_^"
这是作者给我们的信息,叫我们不要 Cracking,^_^
.....
.....
.....
00552956  |. 8D45 E0        LEA EAX,DWORD PTR SS:[EBP-20]
00552959  |. E8 921FEBFF    CALL FOLIEREG.004048F0
0055295E  |. 8B45 E4        MOV EAX,DWORD PTR SS:[EBP-1C] ;真正的 2~5 位,是"C2B3"
00552961  |. 8B55 E0        MOV EDX,DWORD PTR SS:[EBP-20] ;注册码的 2~5 位,是"2345"
00552964  |. E8 2721EBFF    CALL FOLIEREG.00404A90 
00552969  |. 75 07          JNZ SHORT FOLIEREG.00552972
```

这样我们又可以得到注册码的 2~5 位了。我们在修改注册信息：

注册名：`dengkeng[DFCG]`
注册码：`FC2B367890abcdefg7CAC5C50`

现在我们可以“查看详细信息”了吧！我想接下来的地方应该知道是什么了吧！就是剩下的两个部分：“生成安装包”和“生成注册表文件”这两个限制了！方法跟上面的雷同，我就不在罗嗦了！

下面两个你就自己跟踪吧！第 3 个信息得到的是 `50DE` 替换原来的 `0abc`，第 4 个信息得到的是将 `C5AC` 替换掉 `defg`。

下面总结一下分析过程，首先判断是否是注册码是否 25 个，不是则退出并不写进注册表。若是 25 个则通过取密码表，取出密码表的 7,4 位和注册码的 1,25 位比较如果相等则写进注册表等待下次重起验证，否则不写进注册表。通过加载的时候读取注册表的信息参加运算然后在从密码表中取出 7 个字符和注册信息的 18~24 位进行比较。如果成功则会提示你是所谓“注册”的注册版本，否则删除注册表的注册信息。下面是有 3 个功能受到限制，你运行这 3 个功能则又恢复成没有注册的版本了！你可以通过下断点，跟踪得到注册码 2~5 位的信息，同理你也可以得到 10~13 位和 14~17 位的信息，显然 6~9 位信息没有使用！

跟踪完了以后似乎和当时跟踪豪杰一样的感受，只要你有足够的耐心，就一定可以跟踪出正确的注册码，如果哪位高手有时间写个注册机出来，记得 `Email` 一个给我啊！

所以完整的注册信息是：

注册名：`dengkeng[DFCG]`



注册码：`FC2B3678950DEC5AC7CAC5C50`

软件序列号是：`0C342D4FA2A61B2D5C50E838EC`

Made By `dengkeng\[DFCG\]`

E-mail：`shellc0de@sohu.com`

欢迎转载，请保持文章的完整性

`4dd466d0e1e0cc123afbe504da2462ca`
