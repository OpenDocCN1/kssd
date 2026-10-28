
```
00410287  |.  E8 8D2F0000   CALL <JMP.&USER32.ShowWindow>          ; \ShowWindow
```

我们只要注意以上红色部分，我们可以得到：`hInstance`保存在`[430028]`中，`hWnd`保存在`[43023C]`中，而`0041187E`则是窗口函数地址：

跟到窗口函数：

```
0041187E   .  53            PUSH EBX
0041187F   .  56            PUSH ESI
00411880   .  57            PUSH EDI
00411881   .  55            PUSH EBP
00411882   .  83EC 68       SUB ESP,68
00411885   .  8B9C24 800000>MOV EBX,DWORD PTR SS:[ESP+80]          ;  取得消息种类
0041188C   .  8BB424 880000>MOV ESI,DWORD PTR SS:[ESP+88]
```

在地址`41188C`上，下条件断点：`bp 41188c if ebx==111`，然后随便按下一个菜单，跟着找到：

```
00411FA3   > /8B8424 840000>MOV EAX,DWORD PTR SS:[ESP+84]          ;  Case 111 of switch 004118B1
00411FAA   . |66:3D 0002    CMP AX,200
00411FAE   . |72 28         JB SHORT Winline1.00411FD8
00411FB0   .^|0F86 F5FEFFFF JBE Winline1.00411EAB
00411FB6   . |66:3D 0003    CMP AX,300
00411FBA   . |72 0D         JB SHORT Winline1.00411FC9
00411FBC   .^|76 A3         JBE SHORT Winline1.00411F61
00411FBE   . |66:3D 0103    CMP AX,301
00411FC2   .^|74 BE         JE SHORT Winline1.00411F82
00411FC4   .^|E9 B9FAFFFF   JMP Winline1.00411A82
00411FC9   > |66:3D 0102    CMP AX,201
00411FCD   .^|0F84 13FFFFFF JE Winline1.00411EE6
00411FD3   .^|E9 AAFAFFFF   JMP Winline1.00411A82
00411FD8   > |66:3D 0101    CMP AX,101
00411FDC   . |72 15         JB SHORT Winline1.00411FF3
00411FDE   .^|0F86 92FEFFFF JBE Winline1.00411E76
00411FE4   . |66:3D 0201    CMP AX,102
00411FE8   .^|0F84 AEFEFFFF JE Winline1.00411E9C
00411FEE   .^|E9 8FFAFFFF   JMP Winline1.00411A82
00411FF3   > |66:3D 0001    CMP AX,100
00411FF7   .^|0F84 68FEFFFF JE Winline1.00411E65
```

OD 已经自动注释了`00411FA3`正是处理受到`WM_COMMAND`的地方，`AX`中放的正是各个菜单的 ID ，类似`CMP AX,XXX`的代码正是判断消息来自于哪个菜单

```
00411FA3   > /8B8424 840000>MOV EAX,DWORD PTR SS:[ESP+84]          ;  Case 111 of switch 004118B1
00411FAA   . |66:3D 0002    CMP AX,200
00411FAE   . |72 28         JB SHORT Winline1.00411FD8
```

改为：

```
00411FA3   > /E9 B9130000   JMP Winline1.00413361                  ;  Case 111 of switch 004118B1
00411FA8     |90            NOP                                    ;  上面的修改，为了跳向我们自己的代码
00411FA9     |90            NOP
00411FAA   > |66:3D 0002    CMP AX,200
00411FAE   . |72 28         JB SHORT Winline1.00411FD8
```

PeiD告诉我们，`.text:00413361`向后是全0区，但是空间不大，开始没有准备加老板键功能，所以一些代码放这儿了：

```
00413361   > \8B8424 840000>MOV EAX,DWORD PTR SS:[ESP+84]          ;  原来代码搬来到
00413368   .  66:3D 0C03    CMP AX,30C                             ;  判断是不是按下了 F6
0041336C   .  75 0E         JNZ SHORT Winline1.0041337C
0041336E   .  60            PUSHAD
0041336F   .  BA 64000000   MOV EDX,64                             ;  准备加上 100 分
00413374   .  8BC7          MOV EAX,EDI
00413376   .  E8 98DFFFFF   CALL Winline1.00411313                 ;  加分，并且调整小人的高度
0041337B   .  61            POPAD
0041337C   >  66:3D 0D03    CMP AX,30D                             ;  是不是按下了 F7
00413380   .  75 1C         JNZ SHORT Winline1.0041339E
00413382   .  C605 B41D4100>MOV BYTE PTR DS:[411DB4],0             ;  将每次分发小球数改为 0
00413389   .  60            PUSHAD
0041338A   .  6A 30         PUSH 30                                ; /Style = MB\_OK|MB\_ICONEXCLAMATION|MB\_APPLMODAL
0041338C   .  68 508D4200   PUSH Winline1.00428D50                 ; |Title = "秘笈提示"
00413391   .  68 108D4200   PUSH Winline1.00428D10                 ; |Text = "晕，你想作弊！现在开始吧！"
00413396   .  6A 00         PUSH 0                                 ; |hOwner = NULL
00413398   .  E8 22FEFFFF   CALL <JMP.&USER32.MessageBoxA>         ; \MessageBoxA
0041339D   .  61            POPAD
0041339E   >  66:3D 0E03    CMP AX,30E                             ;  是不是按下了 F8
004133A2   .  75 1C         JNZ SHORT Winline1.004133C0
004133A4   .  C605 B41D4100>MOV BYTE PTR DS:[411DB4],3             ;  将每次分发小球数改为 3
004133AB   .  60            PUSHAD
004133AC   .  6A 30         PUSH 30                                ; /Style = MB\_OK|MB\_ICONEXCLAMATION|MB\_APPLMODAL
004133AE   .  68 508D4200   PUSH Winline1.00428D50                 ; |Title = "秘笈提示"
004133B3   .  68 308D4200   PUSH Winline1.00428D30                 ; |Text = "恢复正常，你得靠自己的实力哦！"
004133B8   .  6A 00         PUSH 0                                 ; |hOwner = NULL
004133BA   .  E8 00FEFFFF   CALL <JMP.&USER32.MessageBoxA>         ; \MessageBoxA
004133BF   .  61            POPAD
004133C0   >  66:3D 0F03    CMP AX,30F                             ;  是不是按下了"删除记录文件"菜单
004133C4   .  75 24         JNZ SHORT Winline1.004133EA
004133C6   .  60            PUSHAD
004133C7   .  6A 24         PUSH 24                                ; /Style = MB\_YESNO|MB\_ICONQUESTION|MB\_APPLMODAL
004133C9   .  68 548D4200   PUSH Winline1.00428D54                 ; |Title = "提示"
004133CE   .  68 708D4200   PUSH Winline1.00428D70                 ; |Text = "重启程序分数会被清0，你确认么？"
004133D3   .  6A 00         PUSH 0                                 ; |hOwner = NULL
004133D5   .  E8 E5FDFFFF   CALL <JMP.&USER32.MessageBoxA>         ; \MessageBoxA
004133DA   .  83F8 07       CMP EAX,7
004133DD   .  74 0A         JE SHORT Winline1.004133E9
004133DF   .  68 608D4200   PUSH Winline1.00428D60                 ; /FileName = "winlines.res"
004133E4   .  E8 84FEFFFF   CALL <JMP.&kernel32.DeleteFileA>       ; \DeleteFileA
004133E9   >  61            POPAD
004133EA   >^ E9 BBEBFFFF   JMP Winline1.00411FAA
```

PeiD查看下，`.text`段到`413400`前结束，只剩下17字节就到段尾了。汗！

另外不要忘记了，在`.data`段为上述对话框添加文本，即在下面内存中写入以下数据：

```
00428D10  D4 CE A3 AC C4 E3 CF EB D7 F7 B1 D7 A3 A1 CF D6  晕，你想作弊！现
00428D20  D4 DA BF AA CA BC B0 C9 A3 A1 00 00 00 00 00 00  在开始吧！......
00428D30  BB D6 B8 B4 D5 FD B3 A3 A3 AC C4 E3 B5 C3 BF BF  恢复正常，你得靠
00428D40  D7 D4 BC BA B5 C4 CA B5 C1 A6 C5 B6 A3 A1 00 00  自己的实力哦！..
00428D50  C3 D8 F3 C5 CC E1 CA BE 00 00 00 00 00 00 00 00  秘笈提示........
00428D60  77 69 6E 6C 69 6E 65 73 2E 72 65 73 00 00 00 00  winlines.res....
00428D70  D6 D8 C6 F4 B3 CC D0 F2 B7 D6 CA FD BB E1 B1 BB  重启程序分数会被
00428D80  C7 E5 30 A3 AC C4 E3 C8 B7 C8 CF C3 B4 A3 BF 00  清0，你确认么？.
```

## 第七步：为程序增加老板键功能

说是老板键，其实没有用键盘，而用鼠标右键，我想当用左键点小球时顺手右键隐藏窗口比按键盘快些，可惜程序中没有针对`WM_RBOTTONDOWN`处理的功能，自己添加。

### 实现鼠标右键隐藏窗口

```
004118E8   .  81FB 01020000 CMP EBX,201                           ; 判断是否是 WM\_LBOTTONDOWN
004118EE   .  0F86 1E020000 JBE Winline1.00411B12
004118F4   .  81FB 02020000 CMP EBX,202                           ; 判断是否是 WM\_LBOTTONUP
004118FA   .  0F84 48020000 JE Winline1.00411B48
```

改为：

```
004118E8   .  81FB 01020000 CMP EBX,201
004118EE   .  0F86 1E020000 JBE Winline1.00411B12
004118F4   .  81FB 02020000 CMP EBX,202
004118FA   .- E9 01E70500   JMP Winline1.00470000                  ;  跳去我们的代码 .text空间不够了，放到自己添加的段中吧
004118FF      90            NOP
00470000  - 0F84 421BFAFF   JE Winline1.00411B48                   ; 原来的代码搬来到
00470006    81FB 04020000   CMP EBX,204                            ; 是不是 WM\_RBOTTONDOWN
0047000C    75 31           JNZ SHORT Winline1.0047003F
0047000E    60              PUSHAD                                 ; 保护现场
0047000F    A1 3C024300     MOV EAX,DWORD PTR DS:[43023C]          ; hInstance
00470014    6A 00           PUSH 0                                 ; /ShowState = SW\_HIDE
00470016    50              PUSH EAX                               ; |hWnd = 00110174 ('Color linez 1.21 rdsnow[BCG][...',class='LinezWindow')
00470017    E8 FD31FAFF     CALL <JMP.&USER32.ShowWindow>          ; \隐藏窗口
0047001C    A1 908D4200     MOV EAX,DWORD PTR DS:[428D90]          ; [428D90]保存了 hHook
00470021    85C0            TEST EAX,EAX                           ; [428D90]!=0，说明已经安装了钩子
00470023    75 19           JNZ SHORT Winline1.0047003E
00470025    6A 00           PUSH 0                                 ; /ThreadID = 0
00470027    FF35 28004300   PUSH DWORD PTR DS:[430028]             ; |hModule = 00400000 (Winline1)
0047002D    68 4C004700     PUSH Winline1.0047004C                 ; |Hookproc = Winline1.0047004C
00470032    6A 00           PUSH 0                                 ; |HookType = WH\_JOURNALRECORD
```
