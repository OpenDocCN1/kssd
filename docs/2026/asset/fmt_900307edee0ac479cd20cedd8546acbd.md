
```
00402380   .  6A FF         PUSH -1

00402382   .  68 96DC4000   PUSH Geometry.0040DC96                                      ;  SE 处理程序安装

00402387   .  64:A1 0000000>MOV EAX,DWORD PTR FS:[0]

0040238D   .  50            PUSH EAX

0040238E   .  64:8925 00000>MOV DWORD PTR FS:[0],ESP

00402395   .  81EC C0000000 SUB ESP,0C0

0040239B   .  53            PUSH EBX

0040239C   .  55            PUSH EBP

0040239D   .  56            PUSH ESI

0040239E   .  57            PUSH EDI

0040239F   .  8BF1          MOV ESI,ECX

004023A1   .  6A 00         PUSH 0

004023A3   .  E8 8CAF0000   CALL <JMP.&MFC42.#1134_?AfxEnableControlContainer@@YAXPAVCO>

004023A8   .  83C4 04       ADD ESP,4

004023AB   .  8D4C24 1C     LEA ECX,DWORD PTR SS:[ESP+1C]

004023AF   .  E8 1C880000   CALL Geometry.0040ABD0

004023B4   .  8D4C24 14     LEA ECX,DWORD PTR SS:[ESP+14]

004023B8   .  C78424 D80000>MOV DWORD PTR SS:[ESP+D8],0

004023C3   .  E8 38ECFFFF   CALL Geometry.00401000

004023C8   .  8D4C24 14     LEA ECX,DWORD PTR SS:[ESP+14]

004023CC   .  C68424 D80000>MOV BYTE PTR SS:[ESP+D8],1

004023D4   .  E8 67ECFFFF   CALL Geometry.00401040

004023D9   .  8D4C24 14     LEA ECX,DWORD PTR SS:[ESP+14]

004023DD   .  E8 EEEEFFFF   CALL Geometry.004012D0

004023E2   .  8D4C24 14     LEA ECX,DWORD PTR SS:[ESP+14]

004023E6   .  E8 75EDFFFF   CALL Geometry.00401160

004023EB   .  8D4C24 14     LEA ECX,DWORD PTR SS:[ESP+14]

004023EF   .  E8 0CEEFFFF   CALL Geometry.00401200

004023F4   .  8D4C24 1C     LEA ECX,DWORD PTR SS:[ESP+1C]

004023F8   .  8BF8          MOV EDI,EAX

004023FA   .  E8 61880000   CALL Geometry.0040AC60                                      ;  //这里就是关键CALL了，需要跟入

004023FF   .  8BCE          MOV ECX,ESI

00402401   .  8BE8          MOV EBP,EAX                                                 ;  //ebp=上面的CALL返回值,返回值必须为2

00402403   .  E8 26AF0000   CALL <JMP.&MFC42.#2621_?Enable3dControls@CWinApp@@IAEHXZ>

00402408   .  68 5C624100   PUSH Geometry.0041625C                                      ;  Local AppWizard-Generated Applications

0040240D   .  8BCE          MOV ECX,ESI

0040240F   .  E8 14AF0000   CALL <JMP.&MFC42.#6117_?SetRegistryKey@CWinApp@@IAEXPBD@Z>

00402414   .  BB 04000000   MOV EBX,4

00402419   .  8BCE          MOV ECX,ESI

0040241B   .  53            PUSH EBX

0040241C   .  E8 01AF0000   CALL <JMP.&MFC42.#4159_?LoadStdProfileSettings@CWinApp@@IAE>

00402421   .  6A 6C         PUSH 6C

00402423   .  E8 F4AE0000   CALL <JMP.&MFC42.#823_??2@YAPAXI@Z>

00402428   .  83C4 04       ADD ESP,4

0040242B   .  894424 18     MOV DWORD PTR SS:[ESP+18],EAX

0040242F   .  85C0          TEST EAX,EAX

00402431   .  C68424 D80000>MOV BYTE PTR SS:[ESP+D8],2

00402439   .  74 1D         JE SHORT Geometry.00402458

0040243B   .  68 48054100   PUSH Geometry.00410548                                      ;  CGeometryView

00402440   .  68 08174100   PUSH Geometry.00411708                                      ;  CMainFrame

00402445   .  68 50044100   PUSH Geometry.00410450                                      ;  CGeometryDoc

0040244A   .  68 80000000   PUSH 80

0040244F   .  8BC8          MOV ECX,EAX

00402451   .  E8 C0AE0000   CALL <JMP.&MFC42.#520_??0CSingleDocTemplate@@QAE@IPAUCRunti>

00402456   .  EB 02         JMP SHORT Geometry.0040245A

00402458   >  33C0          XOR EAX,EAX

0040245A   >  50            PUSH EAX

0040245B   .  8BCE          MOV ECX,ESI

0040245D   .  C68424 DC0000>MOV BYTE PTR SS:[ESP+DC],1

00402465   .  E8 A6AE0000   CALL <JMP.&MFC42.#986_?AddDocTemplate@CWinApp@@QAEXPAVCDocT>

0040246A   .  8D4C24 34     LEA ECX,DWORD PTR SS:[ESP+34]

0040246E   .  E8 97AE0000   CALL <JMP.&MFC42.#296_??0CCommandLineInfo@@QAE@XZ>

00402473   .  8D4424 34     LEA EAX,DWORD PTR SS:[ESP+34]

00402477   .  8BCE          MOV ECX,ESI

00402479   .  50            PUSH EAX

0040247A   .  C68424 DC0000>MOV BYTE PTR SS:[ESP+DC],3

00402482   .  E8 7DAE0000   CALL <JMP.&MFC42.#5214_?ParseCommandLine@CWinApp@@QAEXAAVCC>

00402487   .  8D4C24 34     LEA ECX,DWORD PTR SS:[ESP+34]

0040248B   .  51            PUSH ECX

0040248C   .  8BCE          MOV ECX,ESI

0040248E   .  E8 6BAE0000   CALL <JMP.&MFC42.#5301_?ProcessShellCommand@CWinApp@@QAEHAA>

00402493   .  85C0          TEST EAX,EAX

00402495   .  75 3D         JNZ SHORT Geometry.004024D4

00402497   .  8D4C24 34     LEA ECX,DWORD PTR SS:[ESP+34]

0040249B   .  C68424 D80000>MOV BYTE PTR SS:[ESP+D8],1

004024A3   .  E8 50AE0000   CALL <JMP.&MFC42.#617_??1CCommandLineInfo@@UAE@XZ>

004024A8   .  8D4C24 14     LEA ECX,DWORD PTR SS:[ESP+14]

004024AC   .  C68424 D80000>MOV BYTE PTR SS:[ESP+D8],0

004024B4   .  E8 77EBFFFF   CALL Geometry.00401030

004024B9   .  8D4C24 1C     LEA ECX,DWORD PTR SS:[ESP+1C]

004024BD   .  C78424 D80000>MOV DWORD PTR SS:[ESP+D8],-1

004024C8   .  E8 43870000   CALL Geometry.0040AC10

004024CD   .  33C0          XOR EAX,EAX

004024CF   .  E9 AA020000   JMP Geometry.0040277E

004024D4   >  8B4E 20       MOV ECX,DWORD PTR DS:[ESI+20]

004024D7   .  6A 03         PUSH 3

004024D9   .  E8 14AE0000   CALL <JMP.&MFC42.#6215_?ShowWindow@CWnd@@QAEHH@Z>

004024DE   .  8B46 20       MOV EAX,DWORD PTR DS:[ESI+20]

004024E1   .  8B50 20       MOV EDX,DWORD PTR DS:[EAX+20]

004024E4   .  52            PUSH EDX                                                    ; /hWnd

004024E5   .  FF15 C8F54000 CALL DWORD PTR DS:[<&USER32.UpdateWindow>]                  ; \UpdateWindow

004024EB   .  8D4C24 1C     LEA ECX,DWORD PTR SS:[ESP+1C]

004024EF   .  E8 2C8C0000   CALL Geometry.0040B120

004024F4   .  3BC7          CMP EAX,EDI

004024F6   .  7D 0B         JGE SHORT Geometry.00402503

004024F8   .  8D4C24 1C     LEA ECX,DWORD PTR SS:[ESP+1C]

004024FC   .  E8 1F8C0000   CALL Geometry.0040B120

00402501   .  EB 02         JMP SHORT Geometry.00402505

00402503   >  8BC7          MOV EAX,EDI

00402505   >  83FD 02       CMP EBP,2                                                   ;  //EBP是关键值，一定要为2

00402508   .  75 26         JNZ SHORT Geometry.00402530                                 ;  //这个是关键跳，爆破点

0040250A   .  8B4E 20       MOV ECX,DWORD PTR DS:[ESI+20]

0040250D   .  68 40624100   PUSH Geometry.00416240                                      ;  玲珑3D几何作图软件2.06版  //这个是注册成功显示的标题

00402512   .  E8 D5AD0000   CALL <JMP.&MFC42.#6199_?SetWindowTextA@CWnd@@QAEXPBD@Z>

00402517   .  E8 CAAD0000   CALL <JMP.&MFC42.#1175_?AfxGetThread@@YGPAVCWinThread@@XZ>

0040251C   .  85C0          TEST EAX,EAX

0040251E   .  0F84 EA010000 JE Geometry.0040270E

00402524   .  8B10          MOV EDX,DWORD PTR DS:[EAX]

00402526   .  8BC8          MOV ECX,EAX

00402528   .  FF52 7C       CALL DWORD PTR DS:[EDX+7C]
```

根据字符串信息，非常容易就可以找到关键跳，但是究竟是如何确定关键`CALL`的呢，他们之间经过了这么多的代码，其实很简单，`EBP`存放着关键值，从函数最开始单步，不难发现最后一次对`EBP`进行修改的是`00402401`处的指令：`mov ebp,eax`，`eax`则是前一个`CALL`的返回值，因此可以确定那个就是算法`CALL`，这是一般的思路而已。

总体上就分析到这里，将爆破点`NOP`掉即可完成爆破。

*****************************************************************

下面是算法分析：

跟入前面得到的算法`CALL`，可以得到如下代码（可能有点复杂）：
