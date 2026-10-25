
# Steel Box

> 原文：[`www.pediy.com/kssd/pediy06/pediy6907.htm`](https://www.pediy.com/kssd/pediy06/pediy6907.htm)

这个壳总算搞定了。但我的方法也许并不好，`fly` 兄弟有更好的办法，可以在到达入口前完成脱壳，所以还是等 `fly` 兄弟的脱文出来。我的脱文只不过是提供一下思路。请大侠们指教。

## 一、找入口

用 `OD` 载入，隐藏 `OD`，`bp GetWindowTextAF9`，运行，随便输入验证码，断下后返回，一直到：

```
02CC5164    E8 3A690000      CALL 02CCBAA3

02CC5169    8B85 68FFFFFF    MOV EAX,DWORD PTR SS:[EBP-98]

02CC516F    E8 84440000      CALL 02CC95F8

02CC5174    8D9D 60FEFFFF    LEA EBX,DWORD PTR SS:[EBP-1A0]

02CC517A    B9 00010000      MOV ECX,100

02CC517F    E8 65690000      CALL 02CCBAE9

02CC5184    FFB5 50FEFFFF    PUSH DWORD PTR SS:[EBP-1B0]////注册码

02CC518A    8B55 80          MOV EDX,DWORD PTR SS:[EBP-80]////验证码

02CC518D    E8 4C660000      CALL 02CCB7DE////比较

02CC5192    0F85 0B000000    JNZ 02CC51A3////不能跳。

02CC5198    C7C7 40D91202    MOV EDI,212D940

02CC519E    E9 06000000      JMP 02CC51A9

02CC51A3    C7C7 9B38CA01    MOV EDI,1CA389B

02CC51A9    DD85 48FEFFFF    FLD QWORD PTR SS:[EBP-1B8]

02CC51AF    E8 9E620000      CALL 02CCB452

02CC51B4    DEE1             FSUBRP ST(1),ST

02CC51B6    DE1D 44D5CC02    FICOMP WORD PTR DS:[2CCD544]

02CC51BC    DFE0             FSTSW AX

02CC51BE    9E               SAHF

02CC51BF    0F86 30000000    JBE 02CC51F5////必须跳!

02CC51C5    C7C7 46DD0B00    MOV EDI,0BDD46

02CC51CB    8D9D 60FEFFFF    LEA EBX,DWORD PTR SS:[EBP-1A0]

02CC51D1    53               PUSH EBX

02CC51D2    E8 33090000      CALL 02CC5B0A

02CC51D7    8985 6CFFFFFF    MOV DWORD PTR SS:[EBP-94],EAX

02CC51DD    83BD 6CFFFFFF F>CMP DWORD PTR SS:[EBP-94],-1

02CC51E4    0F84 0B000000    JE 02CC51F5

02CC51EA    FFB5 6CFFFFFF    PUSH DWORD PTR SS:[EBP-94]
```

`02CC51BF` 处跳转后程序会在目录下生成两个 `DLL` 文件。这样就可以进入程序界面了。

重新用 `OD` 载入程序，隐藏 `OD`，`bp VirtualAlloc`，一直 `F9` 运行，直到堆栈区如下：

```
0012DE88   7FF61433   /CALL 到 VirtualAlloc 来自 7FF6142D

0012DE8C   02CC0000   |Address = 02CC0000

0012DE90   000004C8   |Size = 4C8 (1224.)

0012DE94   00001000   |AllocationType = MEM_COMMIT

0012DE98   00000040   \Protect = PAGE_EXECUTE_READWRITE
```

这时打开内存镜像。

内存镜像，项目 12

地址=00401000/////在这里下内存访问断点。

大小=0001A000 (106496.)

Owner=anota2   00400000

区段=.text

包含=code,imports,resources

类型=Imag 01001004

访问=RW

初始访问=RWE

运行程序后断在：

```
0040E024    A3 12E04000     MOV DWORD PTR DS:[40E012],EAX////断在这里。

0040E029    68 0BE04000     PUSH anota2.0040E00B                     ; ASCII "TestIt"

0040E02E    FF35 12E04000   PUSH DWORD PTR DS:[40E012]

0040E034    E8 29000000     CALL anota2.0040E062

0040E039    A3 16E04000     MOV DWORD PTR DS:[40E016],EAX

0040E03E    FF15 16E04000   CALL NEAR DWORD PTR DS:[40E016]////进入!

0040E044    8BD0            MOV EDX,EAX

0040E046    EB 04           JMP SHORT anota2.0040E04C/////跳!

0040E048    0041 40         ADD BYTE PTR DS:[ECX+40],AL

0040E04B    00FF            ADD BH,BH

0040E04D    35 48E04000     XOR EAX,40E048

0040E052    C3              RETN

0040E053    0000            ADD BYTE PTR DS:[EAX],AL
```

跳到：

```
0040E04C    FF35 48E04000   PUSH DWORD PTR DS:[40E048] ////入口地址!

0040E052    C3              RETN/////返回到入口处。

00404100    55              PUSH EBP ////入口处。

00404101    8BEC            MOV EBP,ESP

00404103    53              PUSH EBX

00404104    56              PUSH ESI

00404105    57              PUSH EDI

00404106    BB 00704000     MOV EBX,anota2.00407000

0040410B    66:2E:F705 8E47>TEST WORD PTR CS:[40478E],4

00404115    0F85 DB000000   JNZ anota2.004041F6

0040411B    6A 00           PUSH 0

0040411D    FF15 D0824000   CALL NEAR DWORD PTR DS:[4082D0]

00404123    E8 9C020000     CALL anota2.004043C4

00404128    68 00F04000     PUSH anota2.0040F000

0040412D    C3              RETN

0040412E    0100            ADD DWORD PTR DS:[EAX],EAX

00404130    0000            ADD BYTE PTR DS:[EAX],AL
```

到入口后，看看跳转表：

```
001541F5   - E9 71C49177     JMP ole32.CoInitialize

001541FA   - E9 28A49077     JMP ole32.CoUninitialize

001541FF   - E9 3C568677     JMP OLEAUT32.SafeArrayCreate

00154204   - E9 47538677     JMP OLEAUT32.SysAllocStringByteLen

00154209   - E9 C2518677     JMP OLEAUT32.SysFreeString

0015420E   - E9 1D538677     JMP OLEAUT32.SysStringByteLen

00154213   - E9 88758677     JMP OLEAUT32.VariantClear

00154218   - E9 F3768677     JMP OLEAUT32.VariantCopy

0015421D   - E9 1FBBDF7F     JMP 7FF4FD41////象这类地址都是被处理过的。

00154222   - E9 CFB4DF7F     JMP 7FF4F6F6

00154227   - E9 321DCA77     JMP USER32.DispatchMessageA

0015422C   - E9 2924CA77     JMP USER32.EnableWindow

00154231   - E9 273CCB77     JMP USER32.EndDialog

00154236   - E9 43EAC977     JMP USER32.GetClientRect

0015423B   - E9 4A16CA77     JMP USER32.GetDlgItem

00154240   - E9 5621CA77     JMP USER32.GetWindowLongA

00154245   - E9 2122CA77     JMP USER32.GetWindowRect

0015424A   - E9 94FFC977     JMP USER32.GetWindowTextA

0015424F   - E9 4039CA77     JMP USER32.GetWindowTextLengthA

00154254   - E9 EB22CC77     JMP USER32.MessageBoxA

00154259   - E9 46C8DF7F     JMP 7FF50AA4

0015425E   - E9 E4F9C977     JMP USER32.SetFocus

00154263   - E9 9124CA77     JMP USER32.SetWindowPos

00154268   - E9 0F00CA77     JMP USER32.SetWindowTextA

0015426D   - E9 9524CA77     JMP USER32.ShowWindow

00154272   - E9 F5D2C977     JMP USER32.TranslateMessage

00154277   - E9 2A13CB77     JMP USER32.DefFrameProcA

0015427C   - E9 731ACB77     JMP USER32.DefMDIChildProcA

00154281   - E9 CAC7DF7F     JMP 7FF50A50

00154286   - E9 E11CCA77     JMP USER32.GetMessageA

0015428B   - E9 8E0BCA77     JMP USER32.GetSystemMetrics

00154290   - E9 1C27CA77     JMP USER32.IsZoomed

00154295   - E9 77BBDF7F     JMP 7FF4FE11

0015429A   - E9 E8BBDF7F     JMP 7FF4FE87

0015429F   - E9 70C0DF7F     JMP 7FF50314

001542A4   - E9 9CBBDF7F     JMP 7FF4FE45

001542A9   - E9 A9BDDF7F     JMP 7FF50057

001542AE   - E9 DE24CA77     JMP USER32.MoveWindow

001542B3   - E9 95B7DF7F     JMP 7FF4FA4D

001542B8   - E9 30B9DF7F     JMP 7FF4FBED

001542BD   - E9 6C7BCB77     JMP USER32.TranslateAcceleratorA

001542C2   - E9 977CCB77     JMP USER32.TranslateMDISysAccel

001542C7   - E9 3BF9C977     JMP USER32.UpdateWindow

001542CC   - E9 BE74CB77     JMP USER32.PostQuitMessage

001542D1   - E9 D0FD5C71     JMP COMCTL32.InitCommonControls

001542D6   - E9 09FC5C71     JMP COMCTL32.ImageList_ReplaceIcon

001542DB   - E9 CFDC5D71     JMP COMCTL32.ImageList_Remove

001542E0   - E9 7C1B5D71     JMP COMCTL32.ImageList_GetIcon

001542E5   - E9 51D25D71     JMP COMCTL32.ImageList_LoadImageA

001542EA   - E9 71C8DF7F     JMP 7FF50B60

001542EF   - E9 1AD25F71     JMP COMCTL32.CreateStatusWindow

001542F4   - E9 C1E09976     JMP COMDLG32.PrintDlgA

001542F9   - E9 AA219A76     JMP COMDLG32.GetOpenFileNameA

001542FE   - E9 22559A76     JMP COMDLG32.GetSaveFileNameA

00154303   - E9 9023E17F     JMP 7FF66698

00154308   - E9 9E24E17F     JMP 7FF667AB
```

## 二、查找加密 IAT 处

下面来找处理 `IAT` 的地方。重新用 `OD` 载入程序，在上面跳转表地址的随便哪个下硬件写入断点。就可以找到关键处，过程就不说了。不难找到这里：

```
00415066    57              PUSH EDI

00415067    56              PUSH ESI

00415068    8BCD            MOV ECX,EBP

0041506A    81C1 65664200   ADD ECX,426665

00415070    8D39            LEA EDI,DWORD PTR DS:[ECX]

00415072    3E:8B77 04      MOV ESI,DWORD PTR DS:[EDI+4]

00415076    8932            MOV DWORD PTR DS:[EDX],ESI

00415078    2BC6            SUB EAX,ESI

0041507A    83E8 05         SUB EAX,5

0041507D    C606 E9         MOV BYTE PTR DS:[ESI],0E9/////移入 JMP 指令。

00415080    8946 01         MOV DWORD PTR DS:[ESI+1],EAX/////函数地址移入。

00415083    3E:8347 04 05   ADD DWORD PTR DS:[EDI+4],5

00415088    5E              POP ESI

00415089    5F              POP EDI

0041508A    59              POP ECX

0041508B    83C1 04         ADD ECX,4

0041508E    83C2 04         ADD EDX,4

00415091   ^ E9 09FFFFFF    JMP anota2.00414F9F
```



#### 代码段分析

```
00415096    83C6 0C         ADD ESI,0C
00415099   ^ E9 6BFEFFFF    JMP anota2.00414F09
0041509E    33C0           XOR EAX,EAX
004150A0    40             INC EAX
```

上面的 `EAX` 的值是从哪里来的？跟踪可以发现：

```
00414F48    53             PUSH EBX
00414F49    8BD5          MOV EDX,EBP
00414F4B    81C2 423E4200 ADD EDX,423E42
00414F51    FF12          CALL NEAR DWORD PTR DS:[EDX]/////这里就是给 EAX 赋值的 CALL，进入。
00414F53    85C0          TEST EAX,EAX
00414F55    0F84 46010000 JE anota2.004150A1
00414F5B    52            PUSH EDX
00414F5C    50            PUSH EAX
00414F5D    8BD5          MOV EDX,EBP
00414F5F    81C2 3D654200 ADD EDX,42653D
00414F65    F702 04000000 TEST DWORD PTR DS:[EDX],4
00414F6B    74 12         JE SHORT anota2.00414F7F
00414F6D    8BD5          MOV EDX,EBP
00414F6F    81C2 B55B4200 ADD EDX,425BB5
```

进入后，跟踪来到：

```
7FF667CA    8365 E4 00     AND DWORD PTR SS:[EBP-1C],0
7FF667CE    6A 00         PUSH 0
7FF667D0    8D45 EC       LEA EAX,DWORD PTR SS:[EBP-14]
7FF667D3    50            PUSH EAX
7FF667D4    FF75 08       PUSH DWORD PTR SS:[EBP+8]
7FF667D7    E8 8B92FFFF   CALL 7FF5FA67
7FF667DC    83C4 0C       ADD ESP,0C
7FF667DF    0FB6C0        MOVZX EAX,AL
7FF667E2    85C0          TEST EAX,EAX
7FF667E4    74 48         JE SHORT 7FF6682E/////强制跳转!
7FF667E6    6A 01         PUSH 1
7FF667E8    6A 00         PUSH 0
7FF667EA    FF75 0C       PUSH DWORD PTR SS:[EBP+C]
7FF667ED    68 1065F87F   PUSH 7FF86510                                  ; ASCII "GetProcAddress"
7FF667F2    8B4D EC       MOV ECX,DWORD PTR SS:[EBP-14]
7FF667F5    E8 7028FEFF   CALL 7FF4906A
7FF667FA    8945 E4       MOV DWORD PTR SS:[EBP-1C],EAX
7FF667FD    837D E4 00    CMP DWORD PTR SS:[EBP-1C],0
7FF66801    75 1B         JNZ SHORT 7FF6681E
```

上面 `7FF667E4` 强制跳转后，就不会破坏 `IAT` 了。不过程序会检测到代码被修改，然后产生异常。但不要紧，这里 `IAT` 已经完整了，用 `ImportREC` 可以找到全部 `IAT`。如下：

```
; Syntax for each function in a thunk (the separator is a TAB)

; ------------------------------------------------------------

; Flag   RVA   ModuleName   Ordinal   Name

;

; Details for <有效 > parameter:

; ------------------------------

; Flag:  0 = valid: no  -> - Name contains the address of the redirected API (you can set

;                            it to zero if you edit it).

;                          - Ordinal is not considered but you should let '0000' as value.

;                          - ModuleName is not considered but you should let '?' as value.

;

;        1 = valid: yes -> All next parameters on the line will be considered.

;                          Function imported by ordinal must have no name (the 4th TAB must

;                                                                   be there though).

;

;        2 = Equivalent to 0 but it is for the loader.

;

;        3 = Equivalent to 1 but it is for the loader.

;

;        4 = Equivalent to 0 with (R) tag.

;

;        5 = Equivalent to 1 with (R) tag.

;

; 决定在于你 , 编辑这个文件需要承担风险 !       :-)

目标  : C:\Documents and Settings\Administrator\桌面\anota2.rar\_620\anota2.exe

OEP: 00004100   IATRVA: 00008234   IAT 大小: 00000188

FThunk: 00008238   NbFunc: 00000002

1  00008238  advapi32.dll  01A5  RegOpenKeyExA

1  0000823C  advapi32.dll  018C  RegCloseKey

FThunk: 00008244   NbFunc: 00000001

1  00008244  gdi32.dll  0168  GetStockObject

FThunk: 0000824C   NbFunc: 00000020

1  0000824C  kernel32.dll  001F  CloseHandle

1  00008250  kernel32.dll  0039  CreateFileA

1  00008254  kernel32.dll  0091  ExitProcess

1  00008258  kernel32.dll  00BE  FlushFileBuffers

1  0000825C  kernel32.dll  00C6  FreeEnvironmentStringsA

1  00008260  kernel32.dll  00DF  GetCommandLineA

1  00008264  kernel32.dll  010C  GetCurrentDirectoryA

1  00008268  kernel32.dll  011E  GetEnvironmentStrings

1  0000826C  kernel32.dll  012A  GetFileSize

1  00008270  kernel32.dll  012D  GetFileType

1  00008274  kernel32.dll  0132  GetLastError

1  00008278  kernel32.dll  013F  GetModuleHandleA

1  0000827C  kernel32.dll  016B  GetStartupInfoA

1  00008280  kernel32.dll  0177  GetSystemInfo

1  00008284  kernel32.dll  0194  GetVersionExA

1  00008288  kernel32.dll  01A1  GlobalAlloc

1  0000828C  kernel32.dll  01A8  GlobalFree

1  00008290  kernel32.dll  0209  MultiByteToWideChar

1  00008294  kernel32.dll  0244  ReadFile

1  00008298  kernel32.dll  028F  SetCurrentDirectoryA

1  0000829C  kernel32.dll  0293  SetEndOfFile

1  000082A0  kernel32.dll  0296  SetErrorMode

1  000082A4  kernel32.dll  029C  SetFilePointer

1  000082A8  kernel32.dll  02A3  SetLastError

1  000082AC  kernel32.dll  02CA  Sleep

1  000082B0  kernel32.dll  02D7  TlsAlloc

1  000082B4  kernel32.dll  02D8  TlsFree

1  000082B8  kernel32.dll  02D9  TlsGetValue

1  000082BC  kernel32.dll  02DA  TlsSetValue

1  000082C0  kernel32.dll  0308  WideCharToMultiByte

1  000082C4  kernel32.dll  0315  WriteFile

1  000082C8  kernel32.dll  0193  GetVersion

FThunk: 000082D0   NbFunc: 00000002

1  000082D0  ole32.dll  003C  CoInitialize

1  000082D4  ole32.dll  0064  CoUninitialize

FThunk: 000082DC   NbFunc: 00000006

1  000082DC  oleaut32.dll  000F  SafeArrayCreate

1  000082E0  oleaut32.dll  0096  SysAllocStringByteLen

1  000082E4  oleaut32.dll  0006  SysFreeString

1  000082E8  oleaut32.dll  0095  SysStringByteLen

1  000082EC  oleaut32.dll  0009  VariantClear

1  000082F0  oleaut32.dll  000A  VariantCopy

FThunk: 000082F8   NbFunc: 00000024

1  000082F8  user32.dll  0051  CreateDialogParamA

1  000082FC  user32.dll  005B  CreateWindowExA

1  00008300  user32.dll  0098  DispatchMessageA

1  00008304  user32.dll  00BA  EnableWindow

1  00008308  user32.dll  00BC  EndDialog

1  0000830C  user32.dll  00F4  GetClientRect

1  00008310  user32.dll  0106  GetDlgItem

1  00008314  user32.dll  015B  GetWindowLongA

1  00008318  user32.dll  0161  GetWindowRect

1  0000831C  user32.dll  0163  GetWindowTextA

1  00008320  user32.dll  0164  GetWindowTextLengthA

1  00008324  user32.dll  01C4  MessageBoxA

1  00008328  user32.dll  0219  SendMessageA

1  0000832C  user32.dll  0234  SetFocus

1  00008330  user32.dll  0261  SetWindowPos

1  00008334  user32.dll  0264  SetWindowTextA

1  00008338  user32.dll  0270  ShowWindow

1  0000833C  user32.dll  0288  TranslateMessage

1  00008340  user32.dll  0083  DefFrameProcA

1  00008344  user32.dll  0085  DefMDIChildProcA

1  00008348  user32.dll  0096  DialogBoxParamA

1  0000834C  user32.dll  012E  GetMessageA

1  00008350  user32.dll  014A  GetSystemMetrics

1  00008354  user32.dll  0198  IsZoomed

1  00008358  user32.dll  019B  LoadAcceleratorsA

1  0000835C  user32.dll  019F  LoadCursorA

1  00008360  user32.dll  01A3  LoadIconA

1  00008364  user32.dll  01AB  LoadMenuA

1  00008368  user32.dll  01B0  LoadStringA

1  0000836C  user32.dll  01CF  MoveWindow

1  00008370  user32.dll  01F7  RegisterClassA

1  00008374  user32.dll  01F8  RegisterClassExA

1  00008378  user32.dll  0284  TranslateAccelerator

1  0000837C  user32.dll  0287  TranslateMDISysAccel

1  00008380  user32.dll  0297  UpdateWindow

1  00008384  user32.dll  01E6  PostQuitMessage

FThunk: 0000838C   NbFunc: 00000007

1  0000838C  comctl32.dll  0011  InitCommonControls

1  00008390  comctl32.dll  0046  ImageList_ReplaceIcon

1  00008394  comctl32.dll  0044  ImageList_Remove

1  00008398  comctl32.dll  003A  ImageList_GetIcon

1  0000839C  comctl32.dll  003F  ImageList_LoadImage

1  000083A0  comctl32.dll  0016  CreateToolbarEx

1  000083A4  comctl32.dll  0006  CreateStatusWindowA

FThunk: 000083AC   NbFunc: 00000003

1  000083AC  comdlg32.dll  0075  PrintDlgA

1  000083B0  comdlg32.dll  006E  GetOpenFileNameA
```



## 1. 000083B4 `comdlg32.dll` 0070 `GetSaveFileNameA`

## 三、修复主程序

修复很简单。到入口后：

```
00404100    55                PUSH EBP                        ; anota2.00413E4A
00404101    8BEC              MOV EBP,ESP
00404103    53                PUSH EBX
00404104    56                PUSH ESI
00404105    57                PUSH EDI
00404106    BB 00704000       MOV EBX,anota2.00407000
0040410B    66:2E:F705 8E47> TEST WORD PTR CS:[40478E],4
00404115    0F85 DB000000     JNZ anota2.004041F6
0040411B    6A 00             PUSH 0
0040411D    FF15 D0824000     CALL NEAR DWORD PTR DS:[4082D0]
00404123    E8 9C020000       CALL anota2.004043C4
00404128    68 00F04000       PUSH anota2.0040F000////修复处 1。
0040412D    C3                RETN
0040412E    0100              ADD DWORD PTR DS:[EAX],EAX
00404130    0000              ADD BYTE PTR DS:[EAX],AL
00404132    8D83 94020000     LEA EAX,DWORD PTR DS:[EBX+294]
00404138    50                PUSH EAX
00404139    FF15 84824000     CALL NEAR DWORD PTR DS:[408284]
0040413F    83EC 44           SUB ESP,44
00404142    C70424 44000000   MOV DWORD PTR SS:[ESP],44
00404149    C74424 2C 00000>  MOV DWORD PTR SS:[ESP+2C],0
00404151    54                PUSH ESP
00404152    FF15 7C824000     CALL NEAR DWORD PTR DS:[40827C]
00404158    B8 0A000000       MOV EAX,0A
0040415D    F74424 2C 01000>  TEST DWORD PTR SS:[ESP+2C],1
00404165    74 05             JE SHORT anota2.0040416C
00404167    0FB74424 30       MOVZX EAX,WORD PTR SS:[ESP+30]
0040416C    83C4 44           ADD ESP,44
0040416F    8983 FE000000     MOV DWORD PTR DS:[EBX+FE],EAX
00404175    FF15 60824000     CALL NEAR DWORD PTR DS:[408260]
0040417B    E8 21040000       CALL anota2.004045A1
00404180    8983 F4000000     MOV DWORD PTR DS:[EBX+F4],EAX
00404186    FF15 68824000     CALL NEAR DWORD PTR DS:[408268]
0040418C    8983 F0000000     MOV DWORD PTR DS:[EBX+F0],EAX
00404192    6A 00             PUSH 0
00404194    FF15 78824000     CALL NEAR DWORD PTR DS:[408278]
0040419A    8983 04010000     MOV DWORD PTR DS:[EBX+104],EAX
004041A0    E8 07010000       CALL anota2.004042AC
004041A5    68 00004100       PUSH anota2.00410000////修复处 2。
004041AA    C3                RETN
004041AB    36:E8 5E020000    CALL anota2.0040440F                  ; 多余的前缀
004041B1    E8 3B040000       CALL anota2.004045F1
004041B6    68 00104100       PUSH anota2.00411000////修复处 3。
004041BB    C3                RETN
004041BC    07                POP ES                                ; 修正的段位寄存器
004041BD    53                PUSH EBX
004041BE    E8 57010000       CALL anota2.0040431A
004041C3    5B                POP EBX
004041C4    50                PUSH EAX
004041C5    E8 55010000       CALL anota2.0040431F
004041CA    E8 37010000       CALL anota2.00404306
004041CF    68 00204100       PUSH anota2.00412000////修复处 4。
004041D4    C3                RETN
004041D5    FF15 5C824000     CALL NEAR DWORD PTR DS:[40825C]
004041DB    FF4B 1C           DEC DWORD PTR DS:[EBX+1C]
```

共有 4 处要修复。其中第 4 处是程序退出时运行的。`004041D5` 处下断，运行程序，然后退出程序。程序就会断在 `004041D5` 处。此时要修复的代码都已解开：

```
00404100    55                PUSH EBP
00404101    8BEC              MOV EBP,ESP
00404103    53                PUSH EBX
00404104    56                PUSH ESI
00404105    57                PUSH EDI
00404106    BB 00704000       MOV EBX,anota2.00407000
0040410B    66:2E:F705 8E47> TEST WORD PTR CS:[40478E],4
00404115    0F85 DB000000     JNZ anota2.004041F6
0040411B    6A 00             PUSH 0
0040411D    FF15 D0824000     CALL NEAR DWORD PTR DS:[4082D0]
00404123    E8 9C020000       CALL anota2.004043C4
00404128    C783 08010000 0>  MOV DWORD PTR DS:[EBX+108],1////已修复处 1
00404132    8D83 94020000     LEA EAX,DWORD PTR DS:[EBX+294]
00404138    50                PUSH EAX
00404139    FF15 84824000     CALL NEAR DWORD PTR DS:[408284]
0040413F    83EC 44           SUB ESP,44
00404142    C70424 44000000   MOV DWORD PTR SS:[ESP],44
00404149    C74424 2C 00000>  MOV DWORD PTR SS:[ESP+2C],0
00404151    54                PUSH ESP
00404152    FF15 7C824000     CALL NEAR DWORD PTR DS:[40827C]
00404158    B8 0A000000       MOV EAX,0A
0040415D    F74424 2C 01000>  TEST DWORD PTR SS:[ESP+2C],1
00404165    74 05             JE SHORT anota2.0040416C
00404167    0FB74424 30       MOVZX EAX,WORD PTR SS:[ESP+30]
0040416C    83C4 44           ADD ESP,44
0040416F    8983 FE000000     MOV DWORD PTR DS:[EBX+FE],EAX
00404175    FF15 60824000     CALL NEAR DWORD PTR DS:[408260]
0040417B    E8 21040000       CALL anota2.004045A1
00404180    8983 F4000000     MOV DWORD PTR DS:[EBX+F4],EAX
00404186    FF15 68824000     CALL NEAR DWORD PTR DS:[408268]
0040418C    8983 F0000000     MOV DWORD PTR DS:[EBX+F0],EAX
00404192    6A 00             PUSH 0
00404194    FF15 78824000     CALL NEAR DWORD PTR DS:[408278]
0040419A    8983 04010000     MOV DWORD PTR DS:[EBX+104],EAX
004041A0    E8 07010000       CALL anota2.004042AC
004041A5    B8 FF000000       MOV EAX,0FF////已修复处 2
004041AA    72 36             JB SHORT anota2.004041E2
004041AC    E8 5E020000       CALL anota2.0040440F
004041B1    E8 3B040000       CALL anota2.004045F1
004041B6    B8 FF000000       MOV EAX,0FF////已修复处 3
004041BB    72 07             JB SHORT anota2.004041C4
004041BD    53                PUSH EBX
004041BE    E8 57010000       CALL anota2.0040431A
004041C3    5B                POP EBX
004041C4    50                PUSH EAX
004041C5    E8 55010000       CALL anota2.0040431F
004041CA    E8 37010000       CALL anota2.00404306
004041CF    FFB3 F0000000     PUSH DWORD PTR DS:[EBX+F0]////已修复处 4
004041D5    FF15 5C824000     CALL NEAR DWORD PTR DS:[40825C]
```

把这些代码复制后覆盖到主程序 `404100`－`4041D4` 处就完成了。

`4dd466d0e1e0cc123afbe504da2462ca`
