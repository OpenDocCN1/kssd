
# 玲珑 3D 几何作图工具 V2.06 的注册流程全面分析:爆破分析->算法分析->注册机->探究算法的本质

> 原文：[`www.pediy.com/kssd/pediy11/125410.html`](https://www.pediy.com/kssd/pediy11/125410.html)

## 文章元数据

- 【文章标题】: 玲珑 3D 几何作图工具 V2.06 的注册流程全面分析  
- 【文章作者】: `zouzhiyong`  
- 【作者邮箱】: `zzydog@foxmail.com`  
- 【软件名称】: 玲珑 3D 几何作图工具 V2.06  
- 【软件大小】: 763KB  
- 【下载地址】: `http://www.linglong3d.net/download/Linglong3dV2.06.rar`  
- 【加壳方式】: 没有加壳  
- 【保护方式】: 序列号，实际上是 `KeyFile`  
- 【编写语言】: `Microsoft Visual C++ 6.0`  
- 【使用工具】: `od` + `peid` + `windows` 记事本 + 破解辅助计算器  
- 【操作平台】: 虚拟机 `windows XP`  
- 【软件介绍】: 简单灵活的立体几何作图软件  
- 【作者声明】: 这是自己首次详细的算法分析，文章可能有很多不当之处。还请大家来指正。

之前一直在找 `CrackMe` 来练习，感觉上有点脱离现实软件的注册算法，因此特意找一个共享软件来练习一下，总体感觉上难度不是不大，只是注册方式有点怪。至于怎么个怪法，下面开始分析。之前发过的算法分析可能是过于草率了吧，未能得到邀请码。

这次是从最简单的爆破开始逐步深入到算法分析，然后到写注册机，最后到探究注册算法的本质。分析尽量详细，比较通俗易懂。希望这次能得到大家的回复和支持，得到一个邀请码。

## 分析过程

### 初步环境分析

开始就用 `PEID` 查了一下，没有加壳的，是用 `Microsoft Visual C++ 6.0` 编写的程序，用 `OD` 载入进去，一看到那些乱七八糟的函数名就知道是 `MFC` 写的程序。小弟菜鸟一个，第一次分析 `MFC` 程序，`MFC` 的程序真的是非常的头疼，要在一大堆封装代码上找出关键算法，幸好，软件的注册提示信息还是比较全的。

### 爆破追踪

#### 跟踪入手点 1-------输入错误信息跟踪

废话说多了，切入正题。首先要说一下软件比较怪的注册流程。

软件运行后，窗口标题会显示软件没有注册，如果是超过了使用次数会显示已经超过了使用次数，并且弹出窗口要你输入解密锁密文。如图所示：

![`bbs.pediy.com/attachment.php?attachmentid=51449&stc=1&d=1290586273`](http://bbs.pediy.com/attachment.php?attachmentid=51449&stc=1&d=1290586273)

实际上这个输入框根本就没有任何意义，这仅仅是显示一个输入框而已，并没有任何功能（除了相应 `WM_CLOSE` 这个消息外），不管你输入的注册码正确与否，都不会作出反应，只能关掉。可能你会说只有正确的注册码才会有反应，但是下面就证明你是错的。

究竟是怎样看的出来的呢。我是通过这样发现的：开始想从这里入手跟踪注册流程的，下断 `bp GetDlgItemTextW`，没有断下，消息断点没有断下，既然是 `MFC` 的话就更不用下 `GetWindowTextW` 了。此时想到如果获取了密文后一定会存放于某个 `buffer` 缓冲区的，因此，可以通过搜索内存来定位数据，然后对数据下断即可。

在随便输入字符串，点击确定后，我用 `winhex` 搜索软件的全部内存，只找到一处，就是 `MFC` 资源框所在的 `buffer`。正常情况下，如果有获取到内容到缓冲区的话，就肯定不止一处的，除非是根本没有获取内容。为了验证，我又做了两次对比，在点击确定的前和后搜索内存，结果是一样的，同样的地址。

![`bbs.pediy.com/attachment.php?attachmentid=51446&stc=1&d=1290586273`](http://bbs.pediy.com/attachment.php?attachmentid=51446&stc=1&d=1290586273)

![`bbs.pediy.com/attachment.php?attachmentid=51447&stc=1&d=1290586273`](http://bbs.pediy.com/attachment.php?attachmentid=51447&stc=1&d=1290586273)

其后又做一个次验证，就是在该地址下内存访问断点（硬件访问都试过了），然后按确定，没有被断下。

根本没有对密文框数据访问，到这里就证明了这个对话框根本就是废的，只是显示你没有注册而已。就是说，注册不能通过输入的形式来实现。这就是我在前面所说的怪异支出。

#### 跟踪入手点 2-------经典的查找字符串方式

如果一开始就看字符串的话就不用走这么多弯路了，但是一开始就找字符串的话也不会发现作者的注册手段了~~，那个“虚伪”的对话框。

好，查找字符串，发现了一个关键的东西，就是窗口标题。这个很明显是切入点了。

![`bbs.pediy.com/attachment.php?attachmentid=51448&stc=1&d=1290586273`](http://bbs.pediy.com/attachment.php?attachmentid=51448&stc=1&d=1290586273)

随便跟入一个，就来到了所在的函数，下面代码：

```
代码:
```



```assembly
00402380   .  6A FF             PUSH -1
00402382   .  68 96DC4000       PUSH Geometry.0040DC96                        ;  SE 处理程序安装
00402387   .  64:A1 0000000>    MOV EAX,DWORD PTR FS:\[0\]
0040238D   .  50                PUSH EAX
0040238E   .  64:8925 00000>    MOV DWORD PTR FS:\[0\],ESP
00402395   .  81EC C0000000     SUB ESP,0C0
0040239B   .  53                PUSH EBX
0040239C   .  55                PUSH EBP
0040239D   .  56                PUSH ESI
0040239E   .  57                PUSH EDI
0040239F   .  8BF1              MOV ESI,ECX
004023A1   .  6A 00             PUSH 0
004023A3   .  E8 8CAF0000       CALL <JMP.&MFC42.#1134\_?AfxEnableControlContainer@@YAXPAVCO>
004023A8   .  83C4 04           ADD ESP,4
004023AB   .  8D4C24 1C         LEA ECX,DWORD PTR SS:\[ESP+1C\]
004023AF   .  E8 1C880000       CALL Geometry.0040ABD0
004023B4   .  8D4C24 14         LEA ECX,DWORD PTR SS:\[ESP+14\]
004023B8   .  C78424 D80000>    MOV DWORD PTR SS:\[ESP+D8\],0
004023C3   .  E8 38ECFFFF       CALL Geometry.00401000
004023C8   .  8D4C24 14         LEA ECX,DWORD PTR SS:\[ESP+14\]
004023CC   .  C68424 D80000>    MOV BYTE PTR SS:\[ESP+D8\],1
004023D4   .  E8 67ECFFFF       CALL Geometry.00401040
004023D9   .  8D4C24 14         LEA ECX,DWORD PTR SS:\[ESP+14\]
004023DD   .  E8 EEEEFFFF       CALL Geometry.004012D0
004023E2   .  8D4C24 14         LEA ECX,DWORD PTR SS:\[ESP+14\]
004023E6   .  E8 75EDFFFF       CALL Geometry.00401160
004023EB   .  8D4C24 14         LEA ECX,DWORD PTR SS:\[ESP+14\]
004023EF   .  E8 0CEEFFFF       CALL Geometry.00401200
004023F4   .  8D4C24 1C         LEA ECX,DWORD PTR SS:\[ESP+1C\]
004023F8   .  8BF8              MOV EDI,EAX
004023FA   .  E8 61880000       CALL Geometry.0040AC60                        ;  //这里就是关键 CALL 了，需要跟入
004023FF   .  8BCE              MOV ECX,ESI
00402401   .  8BE8              MOV EBP,EAX                                   ;  //ebp=上面的 CALL 返回值,返回值必须为 2
00402403   .  E8 26AF0000       CALL <JMP.&MFC42.#2621\_?Enable3dControls@CWinApp@@IAEHXZ>
00402408   .  68 5C624100       PUSH Geometry.0041625C                        ;  Local AppWizard-Generated Applications
0040240D   .  8BCE              MOV ECX,ESI
0040240F   .  E8 14AF0000       CALL <JMP.&MFC42.#6117\_?SetRegistryKey@CWinApp@@IAEXPBD@Z>
00402414   .  BB 04000000       MOV EBX,4
00402419   .  8BCE              MOV ECX,ESI
0040241B   .  53                PUSH EBX
0040241C   .  E8 01AF0000       CALL <JMP.&MFC42.#4159\_?LoadStdProfileSettings@CWinApp@@IAE>
00402421   .  6A 6C              PUSH 6C
00402423   .  E8 F4AE0000       CALL <JMP.&MFC42.#823\_??2@YAPAXI@Z>
00402428   .  83C4 04           ADD ESP,4
0040242B   .  894424 18         MOV DWORD PTR SS:\[ESP+18\],EAX
0040242F   .  85C0              TEST EAX,EAX
00402431   .  C68424 D80000>    MOV BYTE PTR SS:\[ESP+D8\],2
00402439   .  74 1D             JE SHORT Geometry.00402458
0040243B   .  68 48054100       PUSH Geometry.00410548                        ;  CGeometryView
00402440   .  68 08174100       PUSH Geometry.00411708                        ;  CMainFrame
00402445   .  68 50044100       PUSH Geometry.00410450                        ;  CGeometryDoc
0040244A   .  68 80000000       PUSH 80
0040244F   .  8BC8              MOV ECX,EAX
00402451   .  E8 C0AE0000       CALL <JMP.&MFC42.#520\_??0CSingleDocTemplate@@QAE@IPAUCRunti>
00402456   .  EB 02             JMP SHORT Geometry.0040245A
00402458   >  33C0              XOR EAX,EAX
0040245A   >  50                PUSH EAX
0040245B   .  8BCE              MOV ECX,ESI
0040245D   .  C68424 DC0000>    MOV BYTE PTR SS:\[ESP+DC\],1
00402465   .  E8 A6AE0000       CALL <JMP.&MFC42.#986\_?AddDocTemplate@CWinApp@@QAEXPAVCDocT>
0040246A   .  8D4C24 34         LEA ECX,DWORD PTR SS:\[ESP+34\]
0040246E   .  E8 97AE0000       CALL <JMP.&MFC42.#296\_??0CCommandLineInfo@@QAE@XZ>
00402473   .  8D4424 34         LEA EAX,DWORD PTR SS:\[ESP+34\]
00402477   .  8BCE              MOV ECX,ESI
00402479   .  50                PUSH EAX
0040247A   .  C68424 DC0000>    MOV BYTE PTR SS:\[ESP+DC\],3
00402482   .  E8 7DAE0000       CALL <JMP.&MFC42.#5214\_?ParseCommandLine@CWinApp@@QAEXAAVCC>
00402487   .  8D4C24 34         LEA ECX,DWORD PTR SS:\[ESP+34\]
0040248B   .  51                PUSH ECX
0040248C   .  8BCE              MOV ECX,ESI
0040248E   .  E8 6BAE0000       CALL <JMP.&MFC42.#5301\_?ProcessShellCommand@CWinApp@@QAEHAA>
00402493   .  85C0              TEST EAX,EAX
00402495   .  75 3D             JNZ SHORT Geometry.004024D4
00402497   .  8D4C24 34         LEA ECX,DWORD PTR SS:\[ESP+34\]
0040249B   .  C68424 D80000>    MOV BYTE PTR SS:\[ESP+D8\],1
004024A3   .  E8 50AE0000       CALL <JMP.&MFC42.#617\_??1CCommandLineInfo@@UAE@XZ>
004024A8   .  8D4C24 14         LEA ECX,DWORD PTR SS:\[ESP+14\]
004024AC   .  C68424 D80000>    MOV BYTE PTR SS:\[ESP+D8\],0
004024B4   .  E8 77EBFFFF       CALL Geometry.00401030
004024B9   .  8D4C24 1C         LEA ECX,DWORD PTR SS:\[ESP+1C\]
004024BD   .  C78424 D80000>    MOV DWORD PTR SS:\[ESP+D8\],-1
004024C8   .  E8 43870000       CALL Geometry.0040AC10
004024CD   .  33C0              XOR EAX,EAX
004024CF   .  E9 AA020000       JMP Geometry.0040277E
004024D4   >  8B4E 20           MOV ECX,DWORD PTR DS:\[ESI+20\]
004024D7   .  6A 03              PUSH 3
004024D9   .  E8 14AE0000       CALL <JMP.&MFC42.#6215\_?ShowWindow@CWnd@@QAEHH@Z>
004024DE   .  8B46 20           MOV EAX,DWORD PTR DS:\[ESI+20\]
004024E1   .  8B50 20           MOV EDX,DWORD PTR DS:\[EAX+20\]
004024E4   .  52                PUSH EDX                                      ;  /hWnd
004024E5   .  FF15 C8F54000     CALL DWORD PTR DS:\[<&USER32.UpdateWindow>\]     ;  \\UpdateWindow
004024EB   .  8D4C24 1C         LEA ECX,DWORD PTR SS:\[ESP+1C\]
004024EF   .  E8 2C8C0000       CALL Geometry.0040B120
004024F4   .  3BC7              CMP EAX,EDI
004024F6   .  7D 0B             JGE SHORT Geometry.00402503
004024F8   .  8D4C24 1C         LEA ECX,DWORD PTR SS:\[ESP+1C\]
004024FC   .  E8 1F8C0000       CALL Geometry.0040B120
00402501   .  EB 02             JMP SHORT Geometry.00402505
00402503   >  8BC7              MOV EAX,EDI
00402505   >  83FD 02           CMP EBP,2                                     ;  //EBP 是关键值，一定要为 2
00402508   .  75 26             JNZ SHORT Geometry.00402530                 ;  //这个是关键跳，爆破点
0040250A   .  8B4E 20           MOV ECX,DWORD PTR DS:\[ESI+20\]
0040250D   .  68 40624100       PUSH Geometry.00416240                        ;  玲珑 3D 几何作图软件 2.06 版  //这个是注册成功显示的标题
00402512   .  E8 D5AD0000       CALL <JMP.&MFC42.#6199\_?SetWindowTextA@CWnd@@QAEXPBD@Z>
00402517   .  E8 CAAD0000       CALL <JMP.&MFC42.#1175\_?AfxGetThread@@YGPAVCWinThread@@XZ>
0040251C   .  85C0              TEST EAX,EAX
0040251E   .  0F84 EA010000    JE Geometry.0040270E
00402524   .  8B10              MOV EDX,DWORD PTR DS:\[EAX\]
00402526   .  8BC8              MOV ECX,EAX
00402528   .  FF52 7C           CALL DWORD PTR DS:\[EDX+7C\]
```

根据字符串信息，非常容易就可以找到关键跳，但是究竟是如何确定关键 `CALL` 的呢，他们之间经过了这么多的代码，其实很简单，`EBP` 存放着关键值，从函数最开始单步，不难发现最后一次对 `EBP` 进行修改的是 `00402401` 处的指令：`mov ebp,eax`，`eax` 则是前一个 `CALL` 的返回值，因此可以确定那个就是算法 `CALL`，这是一般的思路而已。

总体上就分析到这里，将爆破点 `NOP` 掉即可完成爆破。

------------------------------------------------------------

下面是算法分析：

跟入前面得到的算法 `CALL`，可以得到如下代码（可能有点复杂）：

代码:



```text
0040AC60   /$   64:A1 0000000>MOV EAX,DWORD PTR FS:[0]
0040AC66   |.   6A FF         PUSH -1
0040AC68   |.   68 7EE54000   PUSH Geometry.0040E57E
0040AC6D   |.   50           PUSH EAX
0040AC6E   |.   64:8925 00000>MOV DWORD PTR FS:[0],ESP
0040AC75   |.   81EC 34010000 SUB ESP,134
0040AC7B   |.   53           PUSH EBX
0040AC7C   |.   55           PUSH EBP
0040AC7D   |.   56           PUSH ESI
0040AC7E   |.   57           PUSH EDI
0040AC7F   |.   68 60644100   PUSH Geometry.00416460                         ; /r+b
0040AC84   |.   8BD9         MOV EBX,ECX                                    ; |
0040AC86   |.   68 58644100   PUSH Geometry.00416458                         ; |gzf.gg
0040AC8B   |.   FF15 14F54000 CALL DWORD PTR DS:[<&MSVCRT.fopen>]            ; \\//打开当前目录下的 gzf.gg 文件，就是 keyfile 了，使用 C 语言函数 fopen 打开
0040AC91   |.   8BF0         MOV ESI,EAX
0040AC93   |.   83C4 08      ADD ESP,8
0040AC96   |.   85F6         TEST ESI,ESI
0040AC98   |.   0F84 5A040000 JE Geometry.0040B0F8                           ;  //打开失败就返回
0040AC9E   |.   8A4424 17    MOV AL,BYTE PTR SS:[ESP+17]
0040ACA2   |.   6A 00        PUSH 0
0040ACA4   |.   8D4C24 30    LEA ECX,DWORD PTR SS:[ESP+30]
0040ACA8   |.   884424 30    MOV BYTE PTR SS:[ESP+30],AL
0040ACAC   |.   FF15 B0F44000 CALL DWORD PTR DS:[<&MSVCP60.?\_Tidy@?$basic\_stri>;  MSVCP60.?\_Tidy@?$basic\_string@DU?$char\_traits@D@std@@V?$allocator@D@2@@std@@AAEX\_N@Z
0040ACB2   |.   BF A8644100   MOV EDI,Geometry.004164A8
0040ACB7   |.   83C9 FF      OR ECX,FFFFFFFF
0040ACBA   |.   33C0         XOR EAX,EAX
0040ACBC   |.   F2:AE        REPNE SCAS BYTE PTR ES:[EDI]
0040ACBE   |.   F7D1         NOT ECX
0040ACC0   |.   49           DEC ECX
0040ACC1   |.   51           PUSH ECX
0040ACC2   |.   68 A8644100   PUSH Geometry.004164A8
0040ACC7   |.   8D4C24 34    LEA ECX,DWORD PTR SS:[ESP+34]
0040ACCB   |.   FF15 C8F44000 CALL DWORD PTR DS:[<&MSVCP60.?assign@?$basic\_str>;  MSVCP60.?assign@?$basic\_string@DU?$char\_traits@D@std@@V?$allocator@D@2@@std@@QAEAAV12@PBDI@Z
0040ACD1   |.   8A4C24 17    MOV CL,BYTE PTR SS:[ESP+17]
0040ACD5   |.   6A 00        PUSH 0
0040ACD7   |.   884C24 20    MOV BYTE PTR SS:[ESP+20],CL
0040ACDB   |.   8D4C24 20    LEA ECX,DWORD PTR SS:[ESP+20]
0040ACDF   |.   C78424 500100>MOV DWORD PTR SS:[ESP+150],0
0040ACEA   |.   FF15 B0F44000 CALL DWORD PTR DS:[<&MSVCP60.?\_Tidy@?$basic\_stri>;  MSVCP60.?\_Tidy@?$basic\_string@DU?$char\_traits@D@std@@V?$allocator@D@2@@std@@AAEX\_N@Z
0040ACF0   |.   BF A8644100   MOV EDI,Geometry.004164A8
0040ACF5   |.   83C9 FF      OR ECX,FFFFFFFF
0040ACF8   |.   33C0         XOR EAX,EAX
0040ACFA   |.   F2:AE        REPNE SCAS BYTE PTR ES:[EDI]
0040ACFC   |.   F7D1         NOT ECX
0040ACFE   |.   49           DEC ECX
0040ACFF   |.   51           PUSH ECX
0040AD00   |.   68 A8644100   PUSH Geometry.004164A8
0040AD05   |.   8D4C24 24    LEA ECX,DWORD PTR SS:[ESP+24]
0040AD09   |.   FF15 C8F44000 CALL DWORD PTR DS:[<&MSVCP60.?assign@?$basic\_str>;  MSVCP60.?assign@?$basic\_string@DU?$char\_traits@D@std@@V?$allocator@D@2@@std@@QAEAAV12@PBDI@Z
0040AD0F   |.   8B2D 10F54000 MOV EBP,DWORD PTR DS:[<&MSVCRT.fread>]         ;  msvcrt.fread
0040AD15   |.   56           PUSH ESI                                         ; /stream
0040AD16   |.   6A 01        PUSH 1                                           ; |n = 1
0040AD18   |.   8D5424 18    LEA EDX,DWORD PTR SS:[ESP+18]                   ; |
0040AD1C   |.   6A 04        PUSH 4                                           ; |size = 4
0040AD1E   |.   52           PUSH EDX                                         ; |ptr
0040AD1F   |.   C68424 5C0100>MOV BYTE PTR SS:[ESP+15C],1                    ; |
0040AD27   |.   FFD5         CALL EBP                                         ; \\//读取一个 DWORD 型的数据 nCount1
0040AD29   |.   8B4424 20    MOV EAX,DWORD PTR SS:[ESP+20]
0040AD2D   |.   83C4 10      ADD ESP,10
0040AD30   |.   85C0         TEST EAX,EAX                                     ;  //nCount 要大于 0，比 0 小则返回
0040AD32   |.   7E 35        JLE SHORT Geometry.0040AD69
0040AD34   |.   56           PUSH ESI
0040AD35   |.   50           PUSH EAX
0040AD36   |.   8D4424 4C    LEA EAX,DWORD PTR SS:[ESP+4C]
0040AD3A   |.   6A 01        PUSH 1
0040AD3C   |.   50           PUSH EAX
0040AD3D   |.   FFD5         CALL EBP                                         ;  //接着读取 nCount1 个字节到缓冲区，记为 hex1
0040AD3F   |.   8B4C24 20    MOV ECX,DWORD PTR SS:[ESP+20]
0040AD43   |.   8D7C24 54    LEA EDI,DWORD PTR SS:[ESP+54]
0040AD47   |.   33C0         XOR EAX,EAX
0040AD49   |.   83C4 10      ADD ESP,10
0040AD4C   |.   C6440C 44 00 MOV BYTE PTR SS:[ESP+ECX+44],0
0040AD51   |.   83C9 FF      OR ECX,FFFFFFFF
0040AD54   |.   F2:AE        REPNE SCAS BYTE PTR ES:[EDI]
0040AD56   |.   F7D1         NOT ECX
0040AD58   |.   49           DEC ECX
0040AD59   |.   8D5424 44    LEA EDX,DWORD PTR SS:[ESP+44]
0040AD5D   |.   51           PUSH ECX
0040AD5E   |.   52           PUSH EDX
0040AD5F   |.   8D4C24 34    LEA ECX,DWORD PTR SS:[ESP+34]
0040AD63   |.   FF15 C8F44000 CALL DWORD PTR DS:[<&MSVCP60.?assign@?$basic\_str>;  MSVCP60.?assign@?$basic\_string@DU?$char\_traits@D@std@@V?$allocator@D@2@@std@@QAEAAV12@PBDI@Z
0040AD69   |>   56           PUSH ESI
0040AD6A   |.   6A 01        PUSH 1
0040AD6C   |.   8D4424 18    LEA EAX,DWORD PTR SS:[ESP+18]
0040AD70   |.   6A 04        PUSH 4
0040AD72   |.   50           PUSH EAX
0040AD73   |.   FFD5         CALL EBP                                         ;  //读取一个 DWORD 型数据 nCount2
0040AD75   |.   8B4424 20    MOV EAX,DWORD PTR SS:[ESP+20]
0040AD79   |.   83C4 10      ADD ESP,10
0040AD7C   |.   85C0         TEST EAX,EAX                                     ;  //nCount2 要大于 0
0040AD7E   |.   7E 35        JLE SHORT Geometry.0040ADB5
0040AD80   |.   56           PUSH ESI
0040AD81   |.   50           PUSH EAX
0040AD82   |.   8D4C24 4C    LEA ECX,DWORD PTR SS:[ESP+4C]
0040AD86   |.   6A 01        PUSH 1
0040AD88   |.   51           PUSH ECX
0040AD89   |.   FFD5         CALL EBP                                         ;  //接着读取 nCount2 个字节数据到缓冲区，记为 hex2
0040AD8B   |.   8B5424 20    MOV EDX,DWORD PTR SS:[ESP+20]
0040AD8F   |.   8D7C24 54    LEA EDI,DWORD PTR SS:[ESP+54]
0040AD93   |.   83C9 FF      OR ECX,FFFFFFFF
0040AD96   |.   33C0         XOR EAX,EAX
0040AD98   |.   83C4 10      ADD ESP,10
0040AD9B   |.   C64414 44 00 MOV BYTE PTR SS:[ESP+EDX+44],0
0040ADA0   |.   F2:AE        REPNE SCAS BYTE PTR ES:[EDI]
0040ADA2   |.   F7D1         NOT ECX
0040ADA4   |.   49           DEC ECX
0040ADA5   |.   8D4424 44    LEA EAX,DWORD PTR SS:[ESP+44]
0040ADA9   |.   51           PUSH ECX
0040ADAA   |.   50           PUSH EAX
0040ADAB   |.   8D4C24 24    LEA ECX,DWORD PTR SS:[ESP+24]
0040ADAF   |.   FF15 C8F44000 CALL DWORD PTR DS:[<&MSVCP60.?assign@?$basic\_str>;  MSVCP60.?assign@?$basic\_string@DU?$char\_traits@D@std@@V?$allocator@D@2@@std@@QAEAAV12@PBDI@Z
0040ADB5   |>   56           PUSH ESI
0040ADB6   |.   6A 01        PUSH 1
0040ADB8   |.   8D4C24 18    LEA ECX,DWORD PTR SS:[ESP+18]
0040ADBC   |.   6A 04        PUSH 4
0040ADBE   |.   51           PUSH ECX
0040ADBF   |.   FFD5         CALL EBP                                         ;  //这个再读取一个 DWORD 型的数据，记为 nRemainTime，为加密后的剩余谁用次数
0040ADC1   |.   8B4424 20    MOV EAX,DWORD PTR SS:[ESP+20]
0040ADC5   |.   8B2D 0CF54000 MOV EBP,DWORD PTR DS:[<&MSVCRT.fclose>]        ;  msvcrt.fclose
0040ADCB   |.   35 51629471  XOR EAX,71946251                                 ;  //nRemainTime = nRemainTime xor 71946251，这里是剩余使用次数
0040ADD0   |.   56           PUSH ESI                                         ; /stream
0040ADD1   |.   894424 24    MOV DWORD PTR SS:[ESP+24],EAX                   ; |
0040ADD5   |.   8943 04      MOV DWORD PTR DS:[EBX+4],EAX                    ; |
0040ADD8   |.   FFD5         CALL EBP                                         ; \\//关闭文件
0040ADDA   |.   8B43 04      MOV EAX,DWORD PTR DS:[EBX+4]
0040ADDD   |.   83C4 14      ADD ESP,14
0040ADE0   |.   83F8 1E      CMP EAX,1E                                       ;  //记录的使用次数不能超过 30 次，可能是防止胡乱修改吧
0040ADE3   |.   0F8F D8020000 JG Geometry.0040B0C1                            ;  //这里不能跳
0040ADE9   |.   85C0         TEST EAX,EAX                                     ;  //使用次数是否已经为 0
0040ADEB   |.   0F8C D0020000 JL Geometry.0040B0C1                            ;  //这里也不能跳
0040ADF1   |.   68 54644100  PUSH Geometry.00416454                           ; /w+b
0040ADF6   |.   68 58644100  PUSH Geometry.00416458                           ; |gzf.gg
0040ADFB   |.   FF15 14F54000 CALL DWORD PTR DS:[<&MSVCRT.fopen>]            ; \\//用可写入的方式打开文件
0040AE01   |.   8BF0         MOV ESI,EAX
0040AE03   |.   83C4 08      ADD ESP,8
0040AE06   |.   85F6         TEST ESI,ESI
0040AE08   |.   75 31        JNZ SHORT Geometry.0040AE3B
0040AE0A   |.   6A 01        PUSH 1
0040AE0C   |.   8D4C24 20    LEA ECX,DWORD PTR SS:[ESP+20]
0040AE10   |.   888424 500100>MOV BYTE PTR SS:[ESP+150],AL
0040AE17   |.   FF15 B0F44000 CALL DWORD PTR DS:[<&MSVCP60.?\_Tidy@?$basic\_stri>;  MSVCP60.?\_Tidy@?$basic\_string@DU?$char\_traits@D@std@@V?$allocator@D@2@@std@@AAEX\_N@Z
0040AE1D   |.   6A 01        PUSH 1
0040AE1F   |.   8D4C24 30    LEA ECX,DWORD PTR SS:[ESP+30]
0040AE23   |.   C78424 500100>MOV DWORD PTR SS:[ESP+150],-1
0040AE2E   |.   FF15 B0F44000 CALL DWORD PTR DS:[<&MSVCP60.?\_Tidy@?$basic\_stri>;  MSVCP60.?\_Tidy@?$basic\_string@DU?$char\_traits@D@std@@V?$allocator@D@2@@std@@AAEX\_N@Z
0040AE34   |.   33C0         XOR EAX,EAX
0040AE36   |.   E9 BD020000  JMP Geometry.0040B0F8
0040AE3B   |>   8B5424 34    MOV EDX,DWORD PTR SS:[ESP+34]
0040AE3F   |.   8B3D 08F54000 MOV EDI,DWORD PTR DS:[<&MSVCRT.fwrite>]        ;  msvcrt.fwrite
0040AE45   |.   56           PUSH ESI                                         ; /stream
0040AE46   |.   6A 01        PUSH 1                                           ; |n = 1
0040AE48   |.   8D4424 18    LEA EAX,DWORD PTR SS:[ESP+18]                   ; |
0040AE4C   |.   6A 04        PUSH 4                                           ; |size = 4
0040AE4E   |.   50           PUSH EAX                                         ; |ptr
0040AE4F   |.   895424 20    MOV DWORD PTR SS:[ESP+20],EDX                   ; |
0040AE53   |.   FFD7         CALL EDI                                         ; \\//将 nCount1 写回到文件中
0040AE55   |.   8B4424 20    MOV EAX,DWORD PTR SS:[ESP+20]
0040AE59   |.   83C4 10      ADD ESP,10
0040AE5C   |.   85C0         TEST EAX,EAX
0040AE5E   |.   7E 18        JLE SHORT Geometry.0040AE78
0040AE60   |.   8B4C24 30    MOV ECX,DWORD PTR SS:[ESP+30]
0040AE64   |.   85C9         TEST ECX,ECX
0040AE66   |.   75 06        JNZ SHORT Geometry.0040AE6E
0040AE68   |.   8B0D A8F44000 MOV ECX,DWORD PTR DS:[<&MSVCP60.?\_C@?1??\_Nullstr>;  MSVCP60.?\_C@?1??\_Nullstr@?$basic\_string@DU?$char\_traits@D@std@@V?$allocator@D@2@@std@@CAPBDXZ@4DB
0040AE6E   |>   56           PUSH ESI
0040AE6F   |.   50           PUSH EAX
0040AE70   |.   6A 01        PUSH 1
0040AE72   |.   51           PUSH ECX
0040AE73   |.   FFD7         CALL EDI                                         ;  //将 hex1 写回到文件中
0040AE75   |.   83C4 10      ADD ESP,10
0040AE78   |>   8B4C24 24    MOV ECX,DWORD PTR SS:[ESP+24]
0040AE7C   |.   56           PUSH ESI
0040AE7D   |.   6A 01        PUSH 1
0040AE7F   |.   8D5424 18    LEA EDX,DWORD PTR SS:[ESP+18]
0040AE83   |.   6A 04        PUSH 4
0040AE85   |.   52           PUSH EDX
0040AE86   |.   894C24 20    MOV DWORD PTR SS:[ESP+20],ECX
0040AE8A   |.   FFD7         CALL EDI                                         ;  //将 nCount2 写回到文件中
0040AE8C   |.   8B4424 20    MOV EAX,DWORD PTR SS:[ESP+20]
0040AE90   |.   83C4 10      ADD ESP,10
0040AE93   |.   85C0         TEST EAX,EAX
0040AE95   |.   7E 18        JLE SHORT Geometry.0040AEAF
0040AE97   |.   8B4C24 20    MOV ECX,DWORD PTR SS:[ESP+20]
0040AE9B   |.   85C9         TEST ECX,ECX
0040AE9D   |.   75 06        JNZ SHORT Geometry.0040AEA5
0040AE9F   |.   8B0D A8F44000 MOV ECX,DWORD PTR DS:[<&MSVCP60.?\_C@?1??\_Nullstr>;  MSVCP60.?\_C@?1??\_Nullstr@?$basic\_string@DU?$char\_traits@D@std@@V?$allocator@D@2@@std@@CAPBDXZ@4DB
0040AEA5   |>   56           PUSH ESI
0040AEA6   |.   50           PUSH EAX
0040AEA7   |.   6A 01        PUSH 1
0040AEA9   |.   51           PUSH ECX
0040AEAA   |.   FFD7         CALL EDI                                         ;  //将 hex2 写回到文件中去
0040AEAC   |.   83C4 10      ADD ESP,10
0040AEAF   |>   8B43 04      MOV EAX,DWORD PTR DS:[EBX+4]
0040AEB2   |.   85C0         TEST EAX,EAX
0040AEB4   |.   7E 04        JLE SHORT Geometry.0040AEBA
0040AEB6   |.   48           DEC EAX
0040AEB7   |.   8943 04      MOV DWORD PTR DS:[EBX+4],EAX
0040AEBA   |>   8B43 04      MOV EAX,DWORD PTR DS:[EBX+4]
0040AEBD   |.   56           PUSH ESI
0040AEBE   |.   6A 01        PUSH 1
0040AEC0   |.   8D4C24 18    LEA ECX,DWORD PTR SS:[ESP+18]
0040AEC4   |.   35 51629471  XOR EAX,71946251                                 ;  //剩余使用次数 nRemainTime 做 xor 加密
0040AEC9   |.   6A 04        PUSH 4
0040AECB   |.   51           PUSH ECX
0040AECC   |.   894424 20    MOV DWORD PTR SS:[ESP+20],EAX
0040AED0   |.   FFD7         CALL EDI                                         ;  //将加密后的 nRemainTime 写回到文件中去
0040AED2   |.   56           PUSH ESI
0040AED3   |.   FFD5         CALL EBP
0040AED5   |.   8B4424 48    MOV EAX,DWORD PTR SS:[ESP+48]
0040AED9   |.   83C4 14      ADD ESP,14
0040AEDC   |.   85C0         TEST EAX,EAX                                     ;  //判断 nCount1 是否为 0
0040AEDE   |.   0F84 E4010000 JE Geometry.0040B0C8                            ;  //不能跳
0040AEE4   |.   8B4424 24    MOV EAX,DWORD PTR SS:[ESP+24]
0040AEE8   |.   85C0         TEST EAX,EAX                                     ;  //判断 nCount2 是否为 0
0040AEEA   |.   0F84 D8010000 JE Geometry.0040B0C8                            ;  //不能跳
0040AEF0   |.   6A 0A        PUSH 0A                                          ; /pFileSystemNameSize = 0000000A
0040AEF2   |.   6A 00        PUSH 0                                           ; |pFileSystemNameBuffer = NULL
0040AEF4   |.   6A 00        PUSH 0                                           ; |pFileSystemFlags = NULL
0040AEF6   |.   8D5424 48    LEA EDX,DWORD PTR SS:[ESP+48]                   ; |
0040AEFA   |.   6A 00        PUSH 0                                           ; |pMaxFilenameLength = NULL
0040AEFC   |.   52           PUSH EDX                                         ; |pVolumeSerialNumber
0040AEFD   |.   6A 0C        PUSH 0C                                          ; |MaxVolumeNameSize = C (12.)
0040AEFF   |.   6A 00        PUSH 0                                           ; |VolumeNameBuffer = NULL
0040AF01   |.   68 50644100  PUSH Geometry.00416450                           ; |c:\\
0040AF06   |.   FF15 84F04000 CALL DWORD PTR DS:[<&KERNEL32.GetVolumeInformati>; \\//获取 c 盘信息，获取 Serial
0040AF0C   |.   8B5424 3C    MOV EDX,DWORD PTR SS:[ESP+3C]
0040AF10   |.   8D4C24 18    LEA ECX,DWORD PTR SS:[ESP+18]
0040AF14   |.   81F2 24827351 XOR EDX,51738224                                ;  //对 serial 运算，serial xor 51738224H
0040AF1A   |.   895424 3C    MOV DWORD PTR SS:[ESP+3C],EDX
0040AF1E   |.   E8 59210000  CALL <JMP.&MFC42.#540\_??0CString@@QAE@XZ>       ;  //copy 数据
0040AF23   |.   8B4424 3C    MOV EAX,DWORD PTR SS:[ESP+3C]
0040AF27   |.   8D4C24 18    LEA ECX,DWORD PTR SS:[ESP+18]
0040AF2B   |.   50           PUSH EAX
0040AF2C   |.   68 4C644100  PUSH Geometry.0041644C                           ;  %x
0040AF31   |.   51           PUSH ECX
0040AF32   |.   C68424 580100>MOV BYTE PTR SS:[ESP+158],2
0040AF3A   |.   E8 9B230000  CALL <JMP.&MFC42.#2818\_?Format@CString@@QAAXPBDZ>;  //将上面 serial 数据从 hex 转换成字符串形式，记为 string
0040AF3F   |.   8B5424 24    MOV EDX,DWORD PTR SS:[ESP+24]
0040AF43   |.   83C4 0C      ADD ESP,0C
0040AF46   |.   8BCB         MOV ECX,EBX
0040AF48   |.   52           PUSH EDX                                         ; /Arg1
0040AF49   |.   E8 32040000  CALL Geometry.0040B380                           ; \\//这里对 string 进行运算，生成 hex3，8 个字节，是机器码的算法 call
0040AF4E   |.   50           PUSH EAX
0040AF4F   |.   8D4C24 44    LEA ECX,DWORD PTR SS:[ESP+44]
0040AF53   |.   E8 E0220000  CALL <JMP.&MFC42.#537\_??0CString@@QAE@PBD@Z>    ;  //Copy 数据
0040AF58   |.   8B7C24 40    MOV EDI,DWORD PTR SS:[ESP+40]                   ;  //EDI 是 Copy 后数据的地址
0040AF5C   |.   83C9 FF      OR ECX,FFFFFFFF
0040AF5F   |.   33C0         XOR EAX,EAX
0040AF61   |.   8BF7         MOV ESI,EDI
0040AF63   |.   F2:AE        REPNE SCAS BYTE PTR ES:[EDI]
0040AF65   |.   F7D1         NOT ECX
0040AF67   |.   49           DEC ECX                                          ;  //计算数据 hex3 的字节数（长度）
0040AF68   |.   6A 01        PUSH 1
0040AF6A   |.   8BE9         MOV EBP,ECX
0040AF6C   |.   8D4C24 30    LEA ECX,DWORD PTR SS:[ESP+30]
0040AF70   |.   55           PUSH EBP
0040AF71   |.   C68424 540100>MOV BYTE PTR SS:[ESP+154],3
0040AF79   |.   FF15 ACF44000 CALL DWORD PTR DS:[<&MSVCP60.?\_Grow@?$basic\_stri>;  MSVCP60.?\_Grow@?$basic\_string@DU?$char\_traits@D@std@@V?$allocator@D@2@@std@@AAE\_NI\_N@Z
0040AF7F   |.   84C0         TEST AL,AL
0040AF81   |.   74 20        JE SHORT Geometry.0040AFA3
0040AF83   |.   8B7C24 30    MOV EDI,DWORD PTR SS:[ESP+30]                   ;  //读取 hex1 的地址
0040AF87   |.   8BCD         MOV ECX,EBP
0040AF89   |.   8BC1         MOV EAX,ECX
0040AF8B   |.   C1E9 02      SHR ECX,2
0040AF8E   |.   F3:A5        REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI]   ;  //对齐 4 个字节，进行每次 dword 数据复制(这里是覆盖 hex1)
0040AF90   |.   8BC8         MOV ECX,EAX
0040AF92   |.   83E1 03      AND ECX,3
0040AF95   |.   F3:A4        REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]     ;  //上面复制剩余的按照字节再进行复制
0040AF97   |.   8B4C24 30    MOV ECX,DWORD PTR SS:[ESP+30]
0040AF9B   |.   896C24 34    MOV DWORD PTR SS:[ESP+34],EBP
0040AF9F   |.   C60429 00    MOV BYTE PTR DS:[ECX+EBP],0                    ;  //写入结束标记
0040AFA3   |>   8B4C24 20    MOV ECX,DWORD PTR SS:[ESP+20]                   ;  //读取 hex2 地址
0040AFA7   |.   8B15 A8F44000 MOV EDX,DWORD PTR DS:[<&MSVCP60.?\_C@?1??\_Nullstr>;  MSVCP60.?\_C@?1??\_Nullstr@?$basic\_string@DU?$char\_traits@D@std@@V?$allocator@D@2@@std@@CAPBDXZ@4DB
0040AFAD   |.   33F6         XOR ESI,ESI
0040AFAF   |.   3BCE         CMP ECX,ESI                                      ;  //hex2 是否存在（hex2 是否为空）
0040AFB1   |.   75 02        JNZ SHORT Geometry.0040AFB5
0040AFB3   |.   8BCA         MOV ECX,EDX
0040AFB5   |>   8B4424 30    MOV EAX,DWORD PTR SS:[ESP+30]                   ;  //取 hex3 的地址
0040AFB9   |.   3BC6         CMP EAX,ESI                                      ;  //hex3 是否为空
0040AFBB   |.   75 02        JNZ SHORT Geometry.0040AFBF
0040AFBD   |.   8BC2         MOV EAX,EDX
0040AFBF   |>   51           PUSH ECX                                         ; /Arg2
0040AFC0   |.   50           PUSH EAX                                         ; |Arg1
0040AFC1   |.   8BCB         MOV ECX,EBX                                      ; |
0040AFC3   |.   E8 68010000  CALL Geometry.0040B130                           ; \\//将 hex2 和 hex3 压栈进行运算，这个就是注册码的比较
0040AFC8   |.   83F8 63      CMP EAX,63                                       ;  //上面的 CALL 返回值一定要为 63H
0040AFCB   |.   C68424 4C0100>MOV BYTE PTR SS:[ESP+14C],2                   ;  //写入注册成功的标记
0040AFD3   |.   8D4C24 40    LEA ECX,DWORD PTR SS:[ESP+40]
0040AFD7   |.   75 7C        JNZ SHORT Geometry.0040B055
0040AFD9   |.   E8 92200000  CALL <JMP.&MFC42.#800\_??1CString@@QAE@XZ>
0040AFDE   |.   8D4C24 18    LEA ECX,DWORD PTR SS:[ESP+18]
0040AFE2   |.   C68424 4C0100>MOV BYTE PTR SS:[ESP+14C],1
0040AFEA   |.   E8 81200000  CALL <JMP.&MFC42.#800\_??1CString@@QAE@XZ>
0040AFEF   |.   8B4C24 20    MOV ECX,DWORD PTR SS:[ESP+20]
0040AFF3   |.   3BCE         CMP ECX,ESI
0040AFF5   |.   74 1C        JE SHORT Geometry.0040B013
0040AFF7   |.   8A41 FF      MOV AL,BYTE PTR DS:[ECX-1]
0040AFFA   |.   84C0         TEST AL,AL
0040AFFC   |.   74 0B        JE SHORT Geometry.0040B009
0040AFFE   |.   3C FF        CMP AL,0FF
0040B000   |.   74 07        JE SHORT Geometry.0040B009
0040B002   |.   FEC8         DEC AL
0040B004   |.   8841 FF      MOV BYTE PTR DS:[ECX-1],AL
0040B007   |.   EB 0A        JMP SHORT Geometry.0040B013
0040B009   |>   49           DEC ECX
0040B00A   |.   51           PUSH ECX                                         ; /block
0040B00B   |.   E8 5A200000  CALL <JMP.&MFC42.#825\_??3@YAXPAX@Z>            ; \\free
0040B010   |.   83C4 04      ADD ESP,4
0040B013   |>   8B4C24 30    MOV ECX,DWORD PTR SS:[ESP+30]
0040B017   |.   897424 20    MOV DWORD PTR SS:[ESP+20],ESI
0040B01B   |.   3BCE         CMP ECX,ESI
0040B01D   |.   897424 24    MOV DWORD PTR SS:[ESP+24],ESI
0040B021   |.   897424 28    MOV DWORD PTR SS:[ESP+28],ESI
0040B025   |.   74 24        JE SHORT Geometry.0040B04B
0040B027   |.   8A41 FF      MOV AL,BYTE PTR DS:[ECX-1]
0040B02A   |.   84C0         TEST AL,AL
0040B02C   |.   74 13        JE SHORT Geometry.0040B041
0040B02E   |.   3C FF        CMP AL,0FF
0040B030   |.   74 0F        JE SHORT Geometry.0040B041
0040B032   |.   FEC8         DEC AL
0040B034   |.   8841 FF      MOV BYTE PTR DS:[ECX-1],AL
0040B037   |.   B8 02000000  MOV EAX,2
0040B03C   |.   E9 B7000000  JMP Geometry.0040B0F8
0040B041   |>   49           DEC ECX
0040B042   |.   51           PUSH ECX                                         ; /block
0040B043   |.   E8 22200000  CALL <JMP.&MFC42.#825\_??3@YAXPAX@Z>            ; \\free
0040B048   |.   83C4 04      ADD ESP,4
0040B04B   |>   B8 02000000  MOV EAX,2
0040B050   |.   E9 A3000000  JMP Geometry.0040B0F8
0040B055   |>   E8 16200000  CALL <JMP.&MFC42.#800\_??1CString@@QAE@XZ>
0040B05A   |.   8D4C24 18    LEA ECX,DWORD PTR SS:[ESP+18]
0040B05E   |.   C68424 4C0100>MOV BYTE PTR SS:[ESP+14C],1
0040B066   |.   E8 05200000  CALL <JMP.&MFC42.#800\_??1CString@@QAE@XZ>
0040B06B   |.   8B4C24 20    MOV ECX,DWORD PTR SS:[ESP+20]
0040B06F   |.   3BCE         CMP ECX,ESI
0040B071   |.   74 1C        JE SHORT Geometry.0040B08F
0040B073   |.   8A41 FF      MOV AL,BYTE PTR DS:[ECX-1]
0040B076   |.   84C0         TEST AL,AL
0040B078   |.   74 0B        JE SHORT Geometry.0040B085
0040B07A   |.   3C FF        CMP AL,0FF
0040B07C   |.   74 07        JE SHORT Geometry.0040B085
0040B07E   |.   FEC8         DEC AL
0040B080   |.   8841 FF      MOV BYTE PTR DS:[ECX-1],AL
0040B083   |.   EB 0A        JMP SHORT Geometry.0040B08F
0040B085   |>   49           DEC ECX
0040B086   |.   51           PUSH ECX                                         ; /block
0040B087   |.   E8 DE1F0000  CALL <JMP.&MFC42.#825\_??3@YAXPAX@Z>            ; \\free
0040B08C   |.   83C4 04      ADD ESP,4
0040B08F   |>   8B4C24 30    MOV ECX,DWORD PTR SS:[ESP+30]
0040B093   |.   897424 20    MOV DWORD PTR SS:[ESP+20],ESI
0040B097   |.   3BCE         CMP ECX,ESI
0040B099   |.   897424 24    MOV DWORD PTR SS:[ESP+24],ESI
0040B09D   |.   897424 28    MOV DWORD PTR SS:[ESP+28],ESI
0040B0A1   |.   74 50        JE SHORT Geometry.0040B0F3
0040B0A3   |.   8A41 FF      MOV AL,BYTE PTR DS:[ECX-1]
0040B0A6   |.   84C0         TEST AL,AL
0040B0A8   |.   74 0B        JE SHORT Geometry.0040B0B5
0040B0AA   |.   3C FF        CMP AL,0FF
0040B0AC   |.   74 07        JE SHORT Geometry.0040B0B5
0040B0AE   |.   FEC8         DEC AL
0040B0B0   |.   8841 FF      MOV BYTE PTR DS:[ECX-1],AL
0040B0B3   |.   EB 3E        JMP SHORT Geometry.0040B0F3
0040B0B5   |>   49           DEC ECX
0040B0B6   |.   51           PUSH ECX                                         ; /block
0040B0B7   |.   E8 AE1F0000  CALL <JMP.&MFC42.#825\_??3@YAXPAX@Z>            ; \\free
0040B0BC   |.   83C4 04      ADD ESP,4
0040B0BF   |.   EB 32        JMP SHORT Geometry.0040B0F3
0040B0C1   |>   C743 04 00000>MOV DWORD PTR DS:[EBX+4],0
0040B0C8   |>   6A 01        PUSH 1
0040B0CA   |.   8D4C24 20    LEA ECX,DWORD PTR SS:[ESP+20]
0040B0CE   |.   C68424 500100>MOV BYTE PTR SS:[ESP+150],0
0040B0D6   |.   FF15 B0F44000 CALL DWORD PTR DS:[<&MSVCP60.?\_Tidy@?$basic\_stri>;  MSVCP60.?\_Tidy@?$basic\_string@DU?$char\_traits@D@std@@V?$allocator@D@2@@std@@AAEX\_N@Z
0040B0DC   |.   6A 01        PUSH 1
0040B0DE   |.   8D4C24 30    LEA ECX,DWORD PTR SS:[ESP+30]
0040B0E2   |.   C78424 500100>MOV DWORD PTR SS:[ESP+150],-1
0040B0ED   |.   FF15 B0F44000 CALL DWORD PTR DS:[<&MSVCP60.?\_Tidy@?$basic\_stri>;  MSVCP60.?\_Tidy@?$basic\_string@DU?$char\_traits@D@std@@V?$allocator@D@2@@std@@AAEX\_N@Z
0040B0F3   |>   B8 01000000  MOV EAX,1
0040B0F8   |>   8B8C24 440100>MOV ECX,DWORD PTR SS:[ESP+144]
0040B0FF   |.   5F           POP EDI
0040B100   |.   5E           POP ESI
0040B101   |.   5D           POP EBP
0040B102   |.   5B           POP EBX
0040B103   |.   64:890D 00000>MOV DWORD PTR FS:[0],ECX
0040B10A   |.   81C4 40010000 ADD ESP,140
0040B110   \\.   C3           RETN
```



大家就给点耐性去看吧，这个函数总结有如下这些要点：

1、从它读取的数据来看，`keyfile` 文件总共分成 5 个部分（我在代码注解那里标明了一些变量名了），在文件中的顺序为：

`nCount1, hex1, nCount2, hex2, nRemainTime`

`nCount1` 记录了 `hex1` 的长度，`nCount2` 记录了 `hex2` 的长度，`nRemainTime` 则为加密后的剩余使用次数。

2、这段代码中有两个是这个注册中的关键算法，分别是：

地址 `0040af49` 的指令 `CALL 0040B380`，这里是生成最终的机器码 `hex3` 的

地址 `0040afc3` 的指令 `CALL 0040B130`，这里是注册成功与否的比较，即是 `hex2` 和 `hex3` 比较

3、机器码 `hex3` 的形成过程：首先调用 `GetVolumeInformation` 获取 `VolumeSerialNumber` 这个成员的值，然后转化成字符串的形式，记 `string`，然后调用 `CALL 0040B380` 这个子函数生成最后的 `hex3`

4、要注意的是，上面两个 `CALL` 都没有对 `hex1` 进行处理或运算，现在再往上看看地址为 `0040af8e` 的指令 `REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI]` 和地址 `0040af95` 的指令 `REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]`，这两条指令其实实现的功能就是将 `hex3` 的数据覆盖到 `hex1` 的 `buffer` 中。因此 `hex1` 从来都没有参与过运算，但是 `hex1` 却不能为空。所以在最后构造 `keyfile` 的时候我们可以随意构造的

跟入 `0040af49 CALL 0040B380`：

好，现在跟入 `0040af49 CALL 0040B380`，分析机器码从 `string` 到 `hex3` 的生成过程，跟入会得到如下代码：

代码:

```
0040B380  /$  81EC 08020000 SUB  ESP,208
0040B386  |.  8D8424 080100>LEA EAX,DWORD PTR SS:[ESP+108]
0040B38D  |.  53            PUSH EBX
0040B38E  |.  55            PUSH EBP
0040B38F  |.  56            PUSH ESI
0040B390  |.  57            PUSH EDI
0040B391  |.  8BBC24 1C0200>MOV EDI,DWORD PTR SS:[ESP+21C]
0040B398  |.  8BD9          MOV EBX,ECX
0040B39A  |.  57            PUSH EDI                                  ; /<%s>
0040B39B  |.  68 64644100   PUSH Geometry.00416464                   ; |%s
0040B3A0  |.  50            PUSH EAX                                  ; |s
0040B3A1  |.  FF15 7CF54000 CALL DWORD PTR DS:[<&USER32.wsprintfA>]  ; \\//格式化字符串 string
0040B3A7  |.  83C9 FF       OR ECX,FFFFFFFF
0040B3AA  |.  33C0          XOR EAX,EAX
0040B3AC  |.  83C4 0C       ADD ESP,0C
0040B3AF  |.  33F6          XOR ESI,ESI
0040B3B1  |.  F2:AE         REPNE SCAS BYTE PTR ES:[EDI]
0040B3B3  |.  F7D1          NOT ECX
0040B3B5  |.  49            DEC ECX                                   ;  //计算字符串长度
0040B3B6  |.  8BF9          MOV EDI,ECX
0040B3B8  |.  85FF          TEST EDI,EDI                              ;  //判断 string 是否为空
0040B3BA  |.  7E 35         JLE SHORT Geometry.0040B3F1
0040B3BC  |.  8B2D 18F54000 MOV EBP,DWORD PTR DS:[<&MSVCRT.div>]     ;  msvcrt.ldiv
0040B3C2  |>  0FBE8C34 1801>/MOVSX ECX,BYTE PTR SS:[ESP+ESI+118]     ;  //取 string[i],
0040B3CA  |.  68 C3000000   |PUSH 0C3                                  ; ///除数
0040B3CF  |.  6A 65         |PUSH 65                                   ; |//循环次数
0040B3D1  |.  51            |PUSH ECX                                  ; |//被除数
0040B3D2  |.  8BCB          |MOV ECX,EBX                               ; |
0040B3D4  |.  E8 D7000000   |CALL Geometry.0040B4B0                   ; \\//计算一个商值 quot
0040B3D9  |.  68 C3000000   |PUSH 0C3
0040B3DE  |.  50            |PUSH EAX
0040B3DF  |.  FFD5          |CALL EBP
0040B3E1  |.  83C4 08       |ADD ESP,8
0040B3E4  |.  885434 18     |MOV BYTE PTR SS:[ESP+ESI+18],DL          ;  //设 remainder = quot % 0c3h，保存到 buffer 里
0040B3E8  |.  46            |INC ESI                                   ;  //i++
0040B3E9  |.  894424 10     |MOV DWORD PTR SS:[ESP+10],EAX
0040B3ED  |.  3BF7          |CMP ESI,EDI
0040B3EF  |.^ 7C D1         \JL SHORT Geometry.0040B3C2
0040B3F1  |>  C6443C 18 00  MOV BYTE PTR SS:[ESP+EDI+18],0           ;  //写入结束标记
0040B3F6  |.  8D7C24 18     LEA EDI,DWORD PTR SS:[ESP+18]
0040B3FA  |.  83C9 FF       OR ECX,FFFFFFFF
0040B3FD  |.  33C0          XOR EAX,EAX
0040B3FF  |.  83C3 08       ADD EBX,8
0040B402  |.  F2:AE         REPNE SCAS BYTE PTR ES:[EDI]
0040B404  |.  F7D1          NOT ECX
0040B406  |.  49            DEC ECX                                    ;  //计算 buffer 长度
0040B407  |.  8BE9          MOV EBP,ECX
```

这里其实是一个对字符串 `string` 的每个字节循环求余数的过程，将余数（单字节）保存到 `buffer` 里，算法比较简单，看代码注解开就可以看懂的了，要注意的是，这里调用了 `c` 上的除法函数在 `MSDN` 可以查找到它的定义 `div_t div( int numer, int denom );` 调用后，返回商存放在 `eax` 上，返回的余数存放在 `edx` 上，这个函数实际上就是对汇编指令 `idiv` 进行封装。

这里还有一个算法 `CALL` 是需要分析的，就是地址 `0040b3d4 CALL 0040B4B0`。

现在就跟入这个算法 `CALL`，可以得到如下代码：

代码:

```
0040B4B0  /$  83EC 08       SUB ESP,8
0040B4B3  |.  53            PUSH EBX
0040B4B4  |.  8B1D 18F54000 MOV EBX,DWORD PTR DS:[<&MSVCRT.div>]     ;  msvcrt.ldiv
0040B4BA  |.  56            PUSH ESI
0040B4BB  |.  8B7424 1C     MOV ESI,DWORD PTR SS:[ESP+1C]
0040B4BF  |.  57            PUSH EDI
0040B4C0  |.  8B7C24 18     MOV EDI,DWORD PTR SS:[ESP+18]
0040B4C4  |.  56            PUSH ESI                                  ; ///传入参数 C3
0040B4C5  |.  57            PUSH EDI                                  ; |//传入参数 c1
0040B4C6  |.  FFD3          CALL EBX                                  ; \\//c1 / c3
0040B4C8  |.  8B4C24 24     MOV ECX,DWORD PTR SS:[ESP+24]             ;  //传入参数 c2 是循环计数变量
0040B4CC  |.  83C4 08       ADD ESP,8
0040B4CF  |.  894424 0C     MOV DWORD PTR SS:[ESP+C],EAX             ;  //保存上上面除法的商到局部变量
0040B4D3  |.  83F9 01       CMP ECX,1                               ;  //传入的参数 c2 <= 1?
0040B4D6  |.  8BC2          MOV EAX,EDX
0040B4D8  |.  7E 18         JLE SHORT Geometry.0040B4F2
0040B4DA  |.  55            PUSH EBP
0040B4DB  |.  8D69 FF       LEA EBP,DWORD PTR DS:[ECX-1]             ;  // c2--
0040B4DE  |>  0FAFC7        /IMUL EAX,EDI                             ;  //var1=除法的余数值 x 传入参数 c1
0040B4E1  |.  56            |PUSH ESI
0040B4E2  |.  50            |PUSH EAX
0040B4E3  |.  FFD3          |CALL EBX                                 ;  //var2 = var1 / 传入参数 c3
0040B4E5  |.  83C4 08       |ADD ESP,8
0040B4E8  |.  894424 10     |MOV DWORD PTR SS:[ESP+10],EAX           ;  //保存商到局部变量
0040B4EC  |.  4D            |DEC EBP                                  ;  // c2--
0040B4ED  |.  8BC2          |MOV EAX,EDX                              ;  //循环结束后，余数作为返回值
0040B4EF  |.^ 75 ED         \JNZ SHORT Geometry.0040B4DE
0040B4F1  |.  5D            POP EBP
0040B4F2  |>  5F            POP EDI
0040B4F3  |.  5E            POP ESI
0040B4F4  |.  5B            POP EBX
0040B4F5  |.  83C4 08       ADD ESP,8
0040B4F8  \\.  C2 0C00      RETN 0C
```

这个也是一个循环求余数的算法，代码注解已经很详细了。

到这里机器码 `hex3` 的生成算法已经分析完了，下面要分析的是比较的那个算法 `CALL`

跟入 `0040afc3 CALL 0040B130`：

前面说到了有两个关键的算法 `CALL`，一个是生成机器码 `hex3` 的算法 `CALL`，在上面已经分析完了，另一个是注册信息的比较算法，这个是判断你注册能否成功的算法，现在就看看它是怎样比较的。

跟入这个算法 `call` 可以得到下面代码：

代码:



```
0040B130  $  83EC 10         SUB  ESP,10
0040B133  |.  8B4424 18      MOV  EAX,DWORD PTR SS:[ESP+18]
0040B137  |.  55             PUSH EBP
0040B138  |.  56             PUSH ESI
0040B139  |.  57             PUSH EDI
0040B13A  |.  50             PUSH EAX                                   ; /Arg1
0040B13B  |.  E8 10010000    CALL Geometry.0040B250                       ; \\//在比较前对 hex2 进行运算,还是用 hex2 来表示处理后的 hex2
0040B140  |.  8A4C24 24      MOV  CL,BYTE PTR SS:[ESP+24]
0040B144  |.  8BF0           MOV  ESI,EAX
0040B146  |.  884C24 0C      MOV  BYTE PTR SS:[ESP+C],CL
0040B14A  |.  8BFE           MOV  EDI,ESI
0040B14C  |.  83C9 FF        OR   ECX,FFFFFFFF
0040B14F  |.  33C0           XOR  EAX,EAX
0040B151  |.  C74424 10 000>MOV  DWORD PTR SS:[ESP+10],0
0040B159  |.  C74424 14 000>MOV  DWORD PTR SS:[ESP+14],0
0040B161  |.  C74424 18 000>MOV  DWORD PTR SS:[ESP+18],0
0040B169  |.  6A 01          PUSH 1
0040B16B  |.  F2:AE          REPNE SCAS BYTE PTR ES:[EDI]
0040B16D  |.  F7D1           NOT  ECX
0040B16F  |.  49             DEC  ECX                                   ;  //计算 hex2 长度
0040B170  |.  8BE9           MOV  EBP,ECX
0040B172  |.  8D4C24 10      LEA  ECX,DWORD PTR SS:[ESP+10]
0040B176  |.  55             PUSH EBP
0040B177  |.  FF15 ACF44000  CALL DWORD PTR DS:[<&MSVCP60.?_Grow@?$basic_string@D>;  MSVCP60.?_Grow@?$basic_string@DU?$char_traits@D@std@@V?$allocator@D@2@@std@@AAE_NI_N@Z
0040B17D  |.  84C0           TEST AL,AL
0040B17F  |.  74 1F          JE   SHORT Geometry.0040B1A0
0040B181  |.  8B7C24 10      MOV  EDI,DWORD PTR SS:[ESP+10]
0040B185  |.  8BCD           MOV  ECX,EBP
0040B187  |.  8BD1           MOV  EDX,ECX
0040B189  |.  55             PUSH EBP
0040B18A  |.  C1E9 02        SHR  ECX,2
0040B18D  |.  F3:A5          REP  MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI]
0040B18F  |.  8BCA           MOV  ECX,EDX
0040B191  |.  83E1 03        AND  ECX,3
0040B194  |.  F3:A4          REP  MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]
0040B196  |.  8D4C24 10      LEA  ECX,DWORD PTR SS:[ESP+10]
0040B19A  |.  FF15 C4F44000  CALL DWORD PTR DS:[<&MSVCP60.?_Eos@?$basic_string@DU>;  MSVCP60.?_Eos@?$basic_string@DU?$char_traits@D@std@@V?$allocator@D@2@@std@@AAEXI@Z
0040B1A0  |>  8B7C24 10      MOV  EDI,DWORD PTR SS:[ESP+10]
0040B1A4  |.  8B35 A8F44000  MOV  ESI,DWORD PTR DS:[<&MSVCP60.?_C@?1??_Nullstr@?$b>;  MSVCP60.?_C@?1??_Nullstr@?$basic_string@DU?$char_traits@D@std@@V?$allocator@D@2@@std@@CAPBDXZ@4DB
0040B1AA  |.  85FF           TEST EDI,EDI
0040B1AC  |.  74 02          JE   SHORT Geometry.0040B1B0
0040B1AE  |.  8BF7           MOV  ESI,EDI
0040B1B0  |>  8B4424 20      MOV  EAX,DWORD PTR SS:[ESP+20]
0040B1B4  |>  8A10           /MOV  DL,BYTE PTR DS:[EAX]                   ;  //取 hex3[i]
0040B1B6  |.  8ACA           |MOV  CL,DL
0040B1B8  |.  3A16          |CMP  DL,BYTE PTR DS:[ESI]                   ;  //hex3[i] == hex2[j] ?
0040B1BA  |.  75 1C         |JNZ  SHORT Geometry.0040B1D8
0040B1BC  |.  84C9          |TEST CL,CL
0040B1BE  |.  74 14         |JE   SHORT Geometry.0040B1D4
0040B1C0  |.  8A50 01       |MOV  DL,BYTE PTR DS:[EAX+1]                 ;  //取 hex3[i+1]
0040B1C3  |.  8ACA          |MOV  CL,DL
0040B1C5  |.  3A56 01       |CMP  DL,BYTE PTR DS:[ESI+1]                 ;  //hex3[i+1] == hex2[j+1] ?
0040B1C8  |.  75 0E         |JNZ  SHORT Geometry.0040B1D8
0040B1CA  |.  83C0 02       |ADD  EAX,2                                   ;  //i+=2
0040B1CD  |.  83C6 02       |ADD  ESI,2                                   ;  //i+=2
0040B1D0  |.  84C9          |TEST CL,CL
0040B1D2  |.^ 75 E0         \JNZ  SHORT Geometry.0040B1B4
```

这里的比较是比较典型的按照每个字节来对比。要注意的是，比较前对 `hex2` 进行过一些运算处理，就在 `0040B13B` `CALL 0040B250` 处，下面跟入这个算法 `CALL`：

```
0040B250  $  8B5424 04      MOV  EDX,DWORD PTR SS:[ESP+4]
0040B254  |.  81EC 08020000 SUB  ESP,208
0040B25A  |.  33C0           XOR  EAX,EAX
0040B25C  |.  53             PUSH EBX
0040B25D  |.  55             PUSH EBP
0040B25E  |.  56             PUSH ESI
0040B25F  |.  8BD9           MOV  EBX,ECX
0040B261  |.  57             PUSH EDI
0040B262  |.  8BFA           MOV  EDI,EDX
0040B264  |.  83C9 FF        OR   ECX,FFFFFFFF
0040B267  |.  52             PUSH EDX                                   ; /<%s>
0040B268  |.  F2:AE          REPNE SCAS BYTE PTR ES:[EDI]               ; |
0040B26A  |.  F7D1           NOT  ECX                                    ; |
0040B26C  |.  8D8424 1C0100>LEA  EAX,DWORD PTR SS:[ESP+11C]             ; |
0040B273  |.  49             DEC  ECX                                    ; |
0040B274  |.  68 64644100    PUSH Geometry.00416464                     ; |%s
0040B279  |.  50             PUSH EAX                                    ; |s
0040B27A  |.  8BF9           MOV  EDI,ECX                                 ; |
0040B27C  |.  FF15 7CF54000  CALL DWORD PTR DS:[<&USER32.wsprintfA>]      ; \wsprintfA
0040B282  |.  83C4 0C        ADD  ESP,0C
0040B285  |.  33F6           XOR  ESI,ESI
0040B287  |.  85FF           TEST EDI,EDI
0040B289  |.  7E 35          JLE  SHORT Geometry.0040B2C0
0040B28B  |.  8B2D 18F54000 MOV  EBP,DWORD PTR DS:[<&MSVCRT.div>]       ;  msvcrt.ldiv
0040B291  |>  0FBE8C34 1801>/MOVSX ECX,BYTE PTR SS:[ESP+ESI+118]
0040B299  |.  68 C3000000   |PUSH 0C3                                    ; /Arg3 = 000000C3
0040B29E  |.  6A 05         |PUSH 5                                      ; |Arg2 = 00000005
0040B2A0  |.  51           |PUSH ECX                                    ; |Arg1
0040B2A1  |.  8BCB          |MOV  ECX,EBX                                 ; |
0040B2A3  |.  E8 08020000   |CALL Geometry.0040B4B0                      ; \Geometry.0040B4B0
0040B2A8  |.  68 C3000000   |PUSH 0C3
0040B2AD  |.  50           |PUSH EAX
0040B2AE  |.  FFD5          |CALL EBP
0040B2B0  |.  83C4 08       |ADD  ESP,8
0040B2B3  |.  885434 18     |MOV  BYTE PTR SS:[ESP+ESI+18],DL
0040B2B7  |.  46           |INC  ESI
0040B2B8  |.  894424 10     |MOV  DWORD PTR SS:[ESP+10],EAX
0040B2BC  |.  3BF7          |CMP  ESI,EDI
0040B2BE  |.^ 7C D1         \JL  SHORT Geometry.0040B291
```

是不是很熟悉呢，对，这就是机器码 `hex3` 的生成算法，但是细心一点就会发现其实有一个地方是不同的，就是 `0040B29E` `PSUH 5`。在 `hex3` 生成算法中的参数是 `push 65h` 的。

好，整个注册过程的算法分析就到此为止了，整个算法就是让我们来求 `hex2` 的数据，好，下面开始写注册机。

### 注册机编写

因为小弟喜欢 `WIN32` 的汇编编程，因此，再此我提供的是 `asm` 版的注册机。代码结合上面的分析应该是挺好理解的，汗～

首先是确定需要的函数：

1. 机器码 `hex3` 的生成函数 `EncryptSerial`。
2. 上面说道了在注册信息比较前会对 `hex2` 进行运算处理，而这个算法跟 `hex3` 的生成算法发很类似，出了那个参数，因此定义一个通用的加密函数 `MainEncrypt`。
3. 地址为 `0040B4B0` 的里循环除法求余数的函数 `CycleDiv`。

`EncryptSerial` 调用 `MainEncrypt`，`MainEncrypt` 调用 `CycleDiv`

现在要做的是 `hex3=MainEncrypt(string,65)` = `MainEncrpyt(hex2,5)`，如果是输入的参数都是 `65` 或 `5` 的话，两个加密过程就是等价的，因此 `hex2=string` 即可，现在是不等，其实是不能够否定他们现在是不等价的（在后面会说到其实他们是等价的）。

现在提供的是一个非常笨的方法：

4. 现在就假设不等价，既然能够让每台机都能够注册，那么肯定存在某些字节值经过运算后能与 `string` 的每个字节数据经过加密的数据对应相同（即同 `hex3` 某对应的字节的值相等），一个字节的范围 `1----0ffh`,通过循环查找即可，所以需要一个循环查找函数，生成原始的 `hex2` 数据的，`GenerateSerial`

这真的是比较笨的方法，用循环对每个值运算一次找相同的。幸好加密的数据是字节为单位，范围是 `1---0ffh`，很快就能够找到的。

`注册机源代码已经作为附件贴上`

### 注册算法的本质

上面是直观的角度去写注册算法，比较繁琐，下面来通过试验来探讨注册加密过程的本质。

其实当时看到 `hex2` 和 `string` 的处理函数非常相似的时候（不同之处在前面已经讲过了），就怀疑他们是否等价的，如果全值等价（或者在 `string` 每个字节所在的范围等价），那么 `hex2` 的数据就非常简单了，`hex2=string` 即可。

首先在前面的分析中我们就知道，`string` 是通过调用 `GetVolumeInformation` 获取 `VolumeSerialNumber` 这是一个 `dword` 型数据，做了 `xor` 运算后还是 `dword` 型数据，因此它包括 `00000000------ffffffff`，因此通过`"%x"`的格式转化为字符串的话，`string` 每个字符范围是以下字符串：`"1234567890abcdef"`中的一个，并且 `string` 的长度已经确定，为 `8` 这个字符。

现在就来验证它们的过程是否等价，如果等价的相同，也很简单，对记 `stemp="1234567890abcdef"` 使用两种加密过程对其进行加密，如果得到的数据相同，就表明在这些字符中，两个加密过程是等价的，也就 `hex2=string`。在前面已经写了一个 `hex3` 的生成函数 `MainEncrypt`，然后做如下调用：`MainEncrypt(string,65)`和 `MainEncrpyt(hex2,5)`。

最后，你会发现实际上，加密后的数据是一样的，也就表明了它们是等价的。

也就是说整个注册的算法的本质只是需要得到 `string`，而 `string` 的获取仅仅是调用一个 API 函数，异或了一下

到这里才发现自己在上面写的注册机里，根本没必要对 `string` 进行加密运算，真的是给软件的作者甩了一圈，汗~，在此也不对注册机做优化了，这就留给大家了吧



### 【遗留问题】

在上面虽然通过试验验证它们是等价的，但是并没有从逻辑或者是数学的角度去证明，希望有朋友证明出来后能告诉小弟，好让我学习学习~

### 上传的附件

|   |
|---|
|   |
|   |
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `KeyGen_src.rar`   |
