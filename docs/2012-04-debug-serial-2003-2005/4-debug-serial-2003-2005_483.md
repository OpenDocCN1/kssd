
# 守财奴 v2.0

> 原文：[`www.pediy.com/kssd/pediy05/pediy50639.htm`](https://www.pediy.com/kssd/pediy05/pediy50639.htm)

**【软件名称】**：守财奴  
**【整理日期】**：2003.4.23  
**【最新版本】**：2.0  
**【文件大小】**：688KB  
**【软件授权】**：免费软件  
**【使用平台】**：Win9x/Me/NT/2000/XP  
**【下载地址】**：`http://www.onlinedown.net/TTwsetup.htm`

**【软件简介】**：

每天，当你疲于挣钱时，是否清楚自己到底挣了多少，花了多少，花在哪里？如果你想消除这些疑问的话，那么请使用本程序，并每天抽出 5 分钟时间，击几下键盘，点几下鼠标，那么即便是日积月累，你也能随时掌握自己各方面的收入支出情况。本程序为免费软件，可任意拷贝使用，绝对没有任何功能与时间上的限制。但如果您觉得此软件对您有用，并且资金允许的话，请向作者联系注册。具体注册方法请在帮助菜单中的注册中查看。《安装完成后请先阅读帮助中的---使用手册》

★ 支持 `Windows98/ME/NT/2000/XP`  
★ 数据安全，用户按角色分配权限  
★ 收支项目分类  
★ 详细的收支记录  
★ 强大的历史收支记录查询  
★ 每个用户独立的理财日记  
★ 计算个人所得税  
★ 数据备份恢复功能  
★ 打印及导出数据为 `Excel` 文件功能。

**【作者】**：风雨无阻

**【声明】**：纯技术交流，无任何商业目的

**【破解工具】**：`fi2.5`, `Ollydbg1.09` 中文版，`filemon`，`regmon`，`win98` 系统

## 破解过程

第一步：用 `fi2.5` 对可执行文件进行分析，没有加壳。执行程序，注册时需要重启，也就是说要读注册表或文件，用 `regmon` 和 `filemon` 进行分析发现在安装目录下生成一名为 `UserInfo.ini` 的文件，打开就会看到刚填写的注册信息

```ini
[RegUserInfo]
RegCode=abcdefghij
RegPass=123456，
```

从而断定程序启动得时候会读此文件进行判断。

第二步：用 `ollydbg` 打开可执行文件。在左上窗口点鼠标反键 搜索－>字符参考，在弹出的创口中点鼠标左键 搜索文本 `"RegCode"` 程序中总共用了两次，`42C97B` 处紧跟的是 `WritePrivateProfileStringA` 函数，也就是保存注册信息到文件。另外一处就是读注册信息的关键地方：`43345B`，按 `F2` 设置断点。按 `F9` 程序就会在断点处中断：

```asm
0043344A   . 68 4C114900    PUSH WBOOK.0049114C                      ; /IniFileName = ".\UserInfo.ini"
0043344F   . 8D5424 34      LEA EDX,DWORD PTR SS:[ESP+34]            ; |
00433453   . 6A 0B          PUSH 0B                                  ; |BufSize = B (11.)
00433455   . 52             PUSH EDX                                 ; |ReturnBuffer
                               --->设置 regcode 缓冲区
00433456   . 68 EC2F4900    PUSH WBOOK.00492FEC                      ; |Default = "0000000000"
0043345B   . 68 FC2C4900    PUSH WBOOK.00492CFC                      ; |Key = "RegCode"
00433460   . 68 F02C4900    PUSH WBOOK.00492CF0                      ; |Section = "RegUserInfo"
00433465   . C74424 54 0800>MOV DWORD PTR SS:[ESP+54],8              ; |
0043346D   . 895C24 58      MOV DWORD PTR SS:[ESP+58],EBX            ; |
                               --->此时 EBX＝1
00433471   . 896C24 5C      MOV DWORD PTR SS:[ESP+5C],EBP            ; |
                               --->此时 EBP＝0
00433475   . C74424 60 0600>MOV DWORD PTR SS:[ESP+60],6              ; |
0043347D   . 895C24 64      MOV DWORD PTR SS:[ESP+64],EBX            ; |
                               --->此时 EBX＝1
00433481   . C74424 68 0300>MOV DWORD PTR SS:[ESP+68],3              ; |
                               --->上面六行代码已 SS:[ESP+54]为首地址
                                      写入 8 1 0 6 1 3 留作以后使用
00433489   . FFD7           CALL EDI                                 ; \GetPrivateProfileStringA
0043348B   . 68 4C114900    PUSH WBOOK.0049114C                      ; /IniFileName = ".\UserInfo.ini"
00433490   . 8D4424 2C      LEA EAX,DWORD PTR SS:[ESP+2C]            ; |
00433494   . 6A 07          PUSH 7                                   ; |BufSize = 7
00433496   . 50             PUSH EAX                                 ; |ReturnBuffer
                               --->设置 regpass 缓冲区
00433497   . 68 E42F4900    PUSH WBOOK.00492FE4                      ; |Default = "000000"
0043349C   . 68 E82C4900    PUSH WBOOK.00492CE8                      ; |Key = "RegPass"
004334A1   . 68 F02C4900    PUSH WBOOK.00492CF0                      ; |Section = "RegUserInfo"
004334A6   . FFD7           CALL EDI                                 ; \GetPrivateProfileStringA
004334A8   . 33C0           XOR EAX,EAX
                               --->EAX 清零
004334AA   . 8D4C24 6C      LEA ECX,DWORD PTR SS:[ESP+6C]

下面开始进行关键的计算和比较：

004334AE   > 0FBE5404 30    MOVSX EDX,BYTE PTR SS:[ESP+EAX+30]
                               --->依次将 regcode 的 1，2，3，4，5，6 位送入 EDX 进行计算
                                      每次循环只取一位
004334B3   . 83EA 30        SUB EDX,30
                               --->EDX = EDX - 0x30h
004334B6   . 40             INC EAX
                               --->EAX = EAX +1
004334B7   . 8911           MOV DWORD PTR DS:[ECX],EDX
                               --->将所计算结果送入 DS:[ECX]保存
004334B9   . 83C1 04        ADD ECX,4
                               --->ECX = ECX + 4
004334BC   . 83F8 06        CMP EAX,6
004334BF   .⁷C ED          JL SHORT WBOOK.004334AE
                               --->跳转到 4334AE 处直到(EAX = 6)循环六次为止
004334C1   . 33C0           XOR EAX,EAX
                               --->EAX 清零
004334C3   . 8D4C24 54      LEA ECX,DWORD PTR SS:[ESP+54]
004334C7   > 0FBE5404 28    MOVSX EDX,BYTE PTR SS:[ESP+EAX+28]
                               --->依次将 regpass 的 1，2，3，4，5，6 位送入 EDX 进行计算
                                      每次循环只取一位
004334CC   . 83EA 30        SUB EDX,30
                               --->EDX = EDX - 0x30h
004334CF   . 40             INC EAX
                               --->EAX = EAX + 1
004334D0   . 8911           MOV DWORD PTR DS:[ECX],EDX
                               --->将计算结果送入 DS:[ECX]保存
004334D2   . 83C1 04        ADD ECX,4
                               --->ECX = ECX + 4
004334D5   . 83F8 06        CMP EAX,6
004334D8   .⁷C ED          JL SHORT WBOOK.004334C7
                               --->跳转到 4334C7 处直到(EAX = 6)循环六次为止
004334DA   . 899E 881E0000  MOV DWORD PTR DS:[ESI+1E88],EBX
                               --->EBX ＝ 1 送入 DS:[ESI+1E88]，留作判断依据
004334E0   . 33C9           XOR ECX,ECX
                               --->ECX 清零
004334E2   > 8B440C 6C      MOV EAX,DWORD PTR SS:[ESP+ECX+6C]
                               --->刚才上面不是保存了用 regpass 的 1，2，3，4，5，6 位进行计算的保存吗？
                                      现在依次把它取出来，送入 EAX，每次循环只取一位
004334E6   . 8B540C 3C      MOV EDX,DWORD PTR SS:[ESP+ECX+3C]
                               --->这里取出的是在 00433481 处写入的 8，1，0，6，3，1；每次一位，依次取出
004334EA   . 03C2           ADD EAX,EDX
                               --->EAX = EDX + EAX
004334EC   . 83F8 0A        CMP EAX,0A
004334EF   . 72 03          JB SHORT WBOOK.004334F4
004334F1   . 83E8 0A        SUB EAX,0A
                               --->上面三句是比较 EAX 与 0x0A 的大小，if(EAX > 0x0A) EAX = EAX - 0x0A
004334F4   > 3B440C 54      CMP EAX,DWORD PTR SS:[ESP+ECX+54]
004334F8   . 74 06          JE SHORT WBOOK.00433500
                               --->将 EAX 的值与刚由 regpass 计算而来的六位结果依次比较
                                      如果相等则跳转到 33500
004334FA   . 89AE 881E0000  MOV DWORD PTR DS:[ESI+1E88],EBP
                               --->EBP = 0 送入 DS:[ESI+1E88]
00433500   > 83C1 04        ADD ECX,4
00433503   . 83F9 18        CMP ECX,18
00433506   .⁷C DA          JL SHORT WBOOK.004334E2
                               --->总共循环六次
00433508   . 39AE 881E0000  CMP DWORD PTR DS:[ESI+1E88],EBP
                               --->注意这一句就是关键比较，如果上面执行了 4334FA 一行代码，那么 EBP ＝ DS:[ESI+1E88]
                                      就会注册失败。
0043350E   . 75 18          JNZ SHORT WBOOK.00433528
                               --->跳转注册成功，不跳注册失败
00433510   . 68 D82F4900    PUSH WBOOK.00492FD8
00433515   . 68 C3040000    PUSH 4C3
0043351A   . 8BCE           MOV ECX,ESI
0043351C   . E8 31340200    CALL WBOOK.00456952
00433521   . 8BC8           MOV ECX,EAX
00433523   . E8 7A350200    CALL WBOOK.00456AA2
00433528   > 399E 881E0000  CMP DWORD PTR DS:[ESI+1E88],EBX
0043352E   . 75 2F          JNZ SHORT WBOOK.0043355F
                               --->跳转注册成功，不跳注册失败
00433530   . 8D4424 30      LEA EAX,DWORD PTR SS:[ESP+30]
00433534   . 8D8E 9C1E0000  LEA ECX,DWORD PTR DS:[ESI+1E9C]
0043353A   . 50             PUSH EAX
0043353B   . 68 5C114900    PUSH WBOOK.0049115C                      ;  ASCII "%s"
00433540   . 51             PUSH ECX
00433541   . E8 D0D40100    CALL WBOOK.00450A16
00433546   . 8D5424 34      LEA EDX,DWORD PTR SS:[ESP+34]
0043354A   . 8D86 A01E0000  LEA EAX,DWORD PTR DS:[ESI+1EA0]
00433550   . 52             PUSH EDX
```



`00433551` `. 68 5C114900` `PUSH WBOOK.0049115C` `; ASCII "%s"`

`00433556` `. 50` `PUSH EAX`

`00433557` `. E8 BAD40100` `CALL WBOOK.00450A16`

`0043355C` `. 83C4 18` `ADD ESP,18`

`0043355F` `> 68 4C114900` `PUSH WBOOK.0049114C` `; ASCII ".\UserInfo.ini"`

`00433564` `. 8D8C24 C000000>` `LEA ECX,DWORD PTR SS:[ESP+C0]`

`0043356B` `. 6A 28` `PUSH 28`

`0043356D` `. 51` `PUSH ECX`

`0043356E` `. 68 60114900` `PUSH WBOOK.00491160`

### 总结
此软件的注册码只比较了六位，用户名也只需要六位就够了，多了也没用。

数学表达式：
* `G` ＝ 固定字符串 `8, 1, 0, 6, 3, 1`
* `N` ＝ 用户名
* `C` ＝ 注册码
* `T` ＝ 临时变量
* `T = N[i] - 0X30h + G[i]`
* 如果(`T > 0X0A`) 那么(`T = T - 0X0A`)
* `C[i] = T + 0X30`

我就不把他写成程序了。这个软件还是很值得一用的，如果你就得好就请向软件作者注册。由以上方法得到的注册码请在验证成功后立即删除。
`4dd466d0e1e0cc123afbe504da2462ca`
