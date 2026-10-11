
# 广电节目管理系统 v0.01

> 原文：[`www.pediy.com/kssd/pediy05/pediy50633.htm`](https://www.pediy.com/kssd/pediy05/pediy50633.htm)

**软件名称**：广电节目管理系统

**最新版本**：`v0.01`

**适用平台**：`Win9x`, `WinME`, `WinNT`, 其它

**软件作者**：常 征

**联系邮箱**：`Liyueqi@etang.com`

**作者主页**：[`Liyueqi.myetang.com/`](http://Liyueqi.myetang.com/)

**注册价格**：588 元/套

**软件大小**：232 KB

**软件语言**：简体中文

**加入时间**：2003-04-21

【软件简介】：广电节目管理系统用于电视台、广播电台、影剧院、电影院或录像厅的各类节目分类、记录、管理、编排、查询。

这是价值不菲的菜鸟级软件！（原因后面说明）

【软件限制】：不明

【作者声明】：初学 Crack，只是感兴趣，目的是找出算法！

【破解工具】：`Ollydbg1.09`、`W32Dasm 9.0` 白金版

---

【过程】：

VB 的东东，没有加任何壳！！第一次就碰上这东西 哭啊～～～

用户号：`550887224`

注册号：`987456321`

---

经小跟踪发现这个东东居然用明码做比较！！！！我晕 588 元的软件居然用明码比较，狂吐血中～～～～～～～

吐哇吐哇就吐习惯了 ：） 应该说我运气好吧～～～～～～

我们还是来简单研究研究该软件的算法吧 另说明一下，

用户号：`550887224`

注册号：`300230879`————————————>这就是真正的注册码！！

在 `66024710` 下断点：可以来到这里

```
66024710 > FF7424 08      PUSH DWORD PTR SS:[ESP+8]   //这里放的是假注册码：987456321

66024714   FF7424 08      PUSH DWORD PTR SS:[ESP+8]   //这里放的是真注册码：300230879

66024718   6A 00          PUSH 0

6602471A   E8 03000000    CALL MSVBVM60.__vbaStrComp

6602471F   C2 0800        RETN 8

66024722 > 837C24 04 02   CMP DWORD PTR SS:[ESP+4],2

66024727   0F84 B3290200  JE MSVBVM60.660470E0

6602472D   68 01000300    PUSH 30001

66024732   FF7424 08      PUSH DWORD PTR SS:[ESP+8]

66024736   FF7424 10      PUSH DWORD PTR SS:[ESP+10]  //这里放的是真注册码：300230879

6602473A   FF7424 18      PUSH DWORD PTR SS:[ESP+18]  //这里放的是假注册码：987456321

6602473E   FF15 74FE1066  CALL DWORD PTR DS:[6610FE74]        ; OLEAUT32.VarBstrCmp

                                                   //调用比较程序对真假注册码进行逐位比较，

66024744

  85C0           TEST EAX,EAX

66024746   0F8C 9B290200  JL MSVBVM60.660470E7

6602474C   48             DEC EAX

6602474D   C2 0C00        RETN 0C

66024750 > 8B4424 08      MOV EAX,DWORD PTR SS:[ESP+8]

66024754   03C0           ADD EAX,EAX

66024756   50             PUSH EAX

66024757   FF7424 08      PUSH DWORD PTR SS:[ESP+8]

6602475B   E8 03000000    CALL MSVBVM60.rtcLeftBstr

66024760   C2 0800        RETN 8

66024763 > 56             PUSH ESI

66024764   8B7424 0C      MOV ESI,DWORD PTR SS:[ESP+C]

66024768   85F6           TEST ESI,ESI

6602476A   0F8C EA260200  JL MSVBVM60.66046E5A

66024770   81FE FFFFFF7F  CMP ESI,7FFFFFFF

66024776   0F8F DE260200  JG MSVBVM60.66046E5A

6602477C   8B4424 08      MOV EAX,DWORD PTR SS:[ESP+8]

66024780   8BCE           MOV ECX,ESI

66024782   85C0           TEST EAX,EAX

66024784   0F84 D7260200  JE MSVBVM60.66046E61

6602478A   8B50 FC        MOV EDX,DWORD PTR DS:[EAX-4]

6602478D   3BF2           CMP ESI,EDX

6602478F   0F87 D3260200  JA MSVBVM60.66046E68

66024795   51             PUSH ECX

66024796   50             PUSH EAX

66024797   FF15 EC190066  CALL DWORD PTR DS:[<&OLEAUT32.#150>]  ; OLEAUT32.SysAllocStringByteLen

6602479D   8BF0           MOV ESI,EAX

6602479F   85F6           TEST ESI,ESI

660247A1   0F84 D4260200  JE MSVBVM60.66046E7B

660247A7   8BC6           MOV EAX,ESI

660247A9   5E             POP ESI

660247AA   C2 0800        RETN 8

660247AD > 53             PUSH EBX

660247AE   56             PUSH ESI

660247AF   8B7424 10      MOV ESI,DWORD PTR SS:[ESP+10]

660247B3   57             PUSH EDI

660247B4   8D7C24 18      LEA EDI,DWORD PTR SS:[ESP+18]

660247B8   33DB           XOR EBX,EBX

660247BA   8B06           MOV EAX,DWORD PTR DS:[ESI]           //用户名

660247BC   85C0           TEST EAX,EAX

660247BE   74 0A          JE SHORT MSVBVM60.660247CA

660247C0   50             PUSH EAX

\----------------------------------------------------------------
验证注册码长度，逐位比较注册码

77A142EE > 55             PUSH EBP

77A142EF   8BEC           MOV EBP,ESP

77A142F1   51             PUSH ECX//ECX=0012E408

77A142F2   53             PUSH EBX            ; MSVBVM60.__vbaStrMove

77A142F3   56             PUSH ESI

77A142F4   8B75 08        MOV ESI,DWORD PTR SS:[EBP+8]

77A142F7   57             PUSH EDI

77A142F8   85F6           TEST ESI,ESI

77A142FA   75 05          JNZ SHORT OLEAUT32.77A14301

77A142FC   2175 FC        AND DWORD PTR SS:[EBP-4],ESI

77A142FF   EB 06          JMP SHORT OLEAUT32.77A14307

77A14301   8B46 FC        MOV EAX,DWORD PTR DS:[ESI-4]

77A14304   8945 FC        MOV DWORD PTR SS:[EBP-4],EAX

77A14307   8B4D 0C        MOV ECX,DWORD PTR SS:[EBP+C]

77A1430A   85C9           TEST ECX,ECX

77A1430C   75 04          JNZ SHORT OLEAUT32.77A14312

77A1430E   33DB           XOR EBX,EBX

77A14310   EB 03          JMP SHORT OLEAUT32.77A14315

77A14312   8B59 FC        MOV EBX,DWORD PTR DS:[ECX-4]

77A14315   395D FC        CMP DWORD PTR SS:[EBP-4],EBX

77A14318   8B55 FC        MOV EDX,DWORD PTR SS:[EBP-4]

77A1431B   72 02          JB SHORT OLEAUT32.77A1431F

77A1431D   8BD3           MOV EDX,EBX

77A1431F   8B45 10        MOV EAX,DWORD PTR SS:[EBP+10]

77A14322   85C0           TEST EAX,EAX

77A14324   75 49          JNZ SHORT OLEAUT32.77A1436F

77A14326   85D2           TEST EDX,EDX

77A14328   74 4E          JE SHORT OLEAUT32.77A14378

77A1432A   8BC2           MOV EAX,EDX

77A1432C   D1E8           SHR EAX,1

77A1432E   8945 10        MOV DWORD PTR SS:[EBP+10],EAX     //EAX=00000009 注册码位数为 9

77A14331   8B7D 0C        MOV EDI,DWORD PTR SS:[EBP+C]      //真注册码放入 EDI

77A14334   8B75 08        MOV ESI,DWORD PTR SS:[EBP+8]      //假注册码放入 ESI

77A14337   8B4D 10        MOV ECX,DWORD PTR SS:[EBP+10]

77A1433A   33C0           XOR EAX,EAX                     //比较真假注册码位数是否正确，不正确则 over

77A1433C   F3:66:A7       REPE CMPS WORD PTR ES:[EDI],WORD PTR DS:[ESI]>  //对真假注册码进行逐位比较

77A1433F   74 05          JE SHORT OLEAUT32.77A14346

77A14341   1BC0           SBB EAX,EAX

77A14343   83D8 FF        SBB EAX,-1

77A14346   85C0           TEST EAX,EAX

77A14348   7F 33          JG SHORT OLEAUT32.77A1437D       //如果有一位不对则跳转；这是关键跳

77A1434A   7D 04          JGE SHORT OLEAUT32.77A14350

77A1434C   33C0           XOR EAX,EAX

77A1434E   EB 5B          JMP SHORT OLEAUT32.77A143AB

77A14350   F6C2 01        TEST DL,1

77A14353   74 23          JE SHORT OLEAUT32.77A14378

77A14355   8B45 08        MOV EAX,DWORD PTR SS:[EBP+8]

77A14358   8B4D 0C        MOV ECX,DWORD PTR SS:[EBP+C]

77A1435B   8A4402 FF      MOV AL,BYTE PTR DS:[EDX+EAX-1]

77A1435F   8A540A FF      MOV DL,BYTE PTR DS:[EDX+ECX-1]

77A14363   3AC2           CMP AL,DL

\--------------------------------
77A1433C   F3:66:A7       REPE CMPS WORD PTR ES:[EDI],WORD PTR DS:[ESI]>  //对真假注册码进行逐位比较  格式如下：

CX=0009(decimal 9)

DS:[ESI]=[0014BA54]=0039

ES:[EDI]=[0014B32C]=0033

该指令信息如下：

名称：重复串操作

操作符：`REPE`/`REPZ`

操作数形式：字符串指令

功能说明：（`CX`）<> 0 或（`ZF`）＝1 则 执行其后的串操作（`CMPS`,`SCAS`）

所以我说他是逐位比较，呵呵

\--------------------------------
【注册信息保存】：

`C:\WINDOWS\SYSTEM` 下的 `vfnlubfvt.sys` 文件

---

【整理】：

用户号：`550887224`

注册号：`300230879`

---

心得：这个软件采用了逐位比较的方法，而且是明码逐位比较，我不想仔细分析了，太简单了，适合于菜鸟练习使用 `ollydbg` ：）不适合注册方法研究，原因是他太简单了

另外你认为这个软件值 588 元吗？？？？？

Cracked By showha

2003-04-22 12:40

`4dd466d0e1e0cc123afbe504da2462ca`
