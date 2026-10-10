

# 添加记事本置顶\\托盘\\字体颜色功能

> 原文：[`www.pediy.com/kssd/pediy10/102031.html`](https://www.pediy.com/kssd/pediy10/102031.html)

【文章标题】: 打造记事本置顶功能  
【文章作者】: wxxw  
【软件名称】: 系统自带  
【保护方式】: 无壳  
【编写语言】: Microsoft Visual C++ 7.0 Method2  
【使用工具】: PEID 0.95  Olldbg1.10  resscope lordpe  
【操作平台】: XP sp3  
【软件介绍】: 不用介绍了吧  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
\--------------------------------------------------------------------------------  
【详细过程】  
   早有前人实现了该功能，虽说是别人嚼过的馍，可我还是想自己动手试试  
首先先设想下做成什么样？我们想在菜单里添加一项“置顶”，选择后完成置顶功能，取消后恢复原状，类似菜单“格式”下的“自动换行”，设想好了，开始动工  
  
因为有系统文件还原保护，所以先将 notepad.exe 复制为 notepad1.exe,然后在资源管理器--“工具”--“文件夹选项”--“文件类型”将 txt 文件更改为 notepad1.exe 打开，前备工作做完了，现在开始正式修改了，首先用资源工具(如 rescope,exescope)修改菜单资源，比如我在“查看”下添加一菜单项“置顶”，ID 自己随便写，如这里写为 28，然后用 LORDPE 在输入表里添加 user32.dll 里函数 SetWindowPos(完成置顶及释放置顶）还需要函数 GetMenu,CheckMenuItem（完成菜单项前面的选择标记和取消），这两个函数源程序里已经有了，用 OD 载入程序 notepad1.exe，F9 运行，然后在 77d1870c(这个地址由来详见 http://bbs.pediy.com/showthread.php?t=98274)下条件断点\[esp+c\]==WM\_COMMAND && \[esp+14\]==0 ,这里\[esp+c\]是消息码，\[esp+10\]是消息 wparam,\[esp+14\]是消息 lparam,因为菜单的 lparam 是 0，所以加上\[esp+14\]==0 条件，如果不加，一切换到 notepad1，就会中断了，这是因为 notepad 的 edit 控件发出了 WM\_COMMAND,好了，选择“置顶”菜单，断下来，来到窗口回调函数 01003429，F7 单步，来到处理 WM\_COMMAND 消息代码 01002b87,继续单步来到如下代码  

代码:

```
01002BBE   .  83FF 40       CMP EDI,40                               ;  Switch (cases 1..303)
01002BC1   .  8995 F0FDFFFF MOV DWORD PTR SS:\[EBP-210\],EDX
01002BC7   .  0F8F F9060000 JG notepad1.010032C6
01002BCD   .  0F84 DB060000 JE notepad1.010032AE
01002BD3   .  83FF 15       CMP EDI,15
01002BD6   .  0F8F CE020000 JG notepad1.01002EAA
01002BDC   .  0F84 EC030000 JE notepad1.01002FCE
01002BE2   .  83FF 05       CMP EDI,5
01002BE5   .  0F8F 74020000 JG notepad1.01002E5F
01002BEB   .  0F84 0B020000 JE notepad1.01002DFC
01002BF1   .  4F            DEC EDI
01002BF2   .  0F84 F8010000 JE notepad1.01002DF0
01002BF8   .  4F            DEC EDI
01002BF9   .  0F84 0E010000 JE notepad1.01002D0D
01002BFF   .  4F            DEC EDI
01002C00   .  74 0A         JE SHORT notepad1.01002C0C
01002C02   .  4F            DEC EDI
01002C03   .  74 32         JE SHORT notepad1.01002C37
01002C05   >  33C0          XOR EAX,EAX                              ;  Default case of switch 01003024
01002C07   .  E9 5E070000   JMP notepad1.0100336A
......
01002EAA   > \\83FF 1A       CMP EDI,1A
01002EAD   .  0F8F 95580000 JG notepad1.01003019
01002EB3   .  0F84 55010000 JE notepad1.0100300E
01002EB9   .  83EF 16       SUB EDI,16
01002EBC   .  0F84 F4000000 JE notepad1.01002FB6
01002EC2   .  4F            DEC EDI
01002EC3   .  0F84 93000000 JE notepad1.01002F5C
01002EC9   .  4F            DEC EDI
01002ECA   .  74 5E         JE SHORT notepad1.01002F2A
01002ECC   .  4F            DEC EDI
01002ECD   .^ 0F85 32FDFFFF JNZ notepad1.01002C05

```

其中 EDI 里是菜单 ID,"置顶"的 ID 是 28（1C),单步来到 01002EAD，就在这里处理消息吧，找一块空白的地方如 01008748 写我们的代码，再将指令改为 JG notepad1.01008748  
添加的代码如下：其中将\[1008747\]的地方用来存储"置顶“菜单项选择标记状态  
代码:

```
01008748   > \\6A 1C         PUSH 1C
0100874A   .  5B            POP EBX
0100874B   .  3BFB          CMP EDI,EBX
0100874D   .^ 0F85 C6A8FFFF JNZ notepad1.01003019
01008753   .  803D 47870001>CMP BYTE PTR DS:\[1008747\],1              ;  Case 1C of switch 01002BBE
0100875A   .  74 35         JE SHORT notepad1.01008791
0100875C   .  C605 47870001>MOV BYTE PTR DS:\[1008747\],1
01008763   .  60            PUSHAD
01008764   .  6A 03         PUSH 3                                   ; /Flags = SWP\_NOSIZE|SWP\_NOMOVE
01008766   .  6A 00         PUSH 0                                   ; |Height = 0
01008768   .  6A 00         PUSH 0                                   ; |Width = 0
0100876A   .  6A 00         PUSH 0                                   ; |Y = 0
0100876C   .  6A 00         PUSH 0                                   ; |X = 0
0100876E   .  6A FF         PUSH -1                                  ; |InsertAfter = HWND\_TOPMOST
01008770   .  52            PUSH EDX                                 ; |hWnd
01008771   .  FF15 1A300101 CALL DWORD PTR DS:\[<&USER32.SetWindowPos>; \\SetWindowPos
01008777   .  61            POPAD
01008778   .  60            PUSHAD
01008779   .  52            PUSH EDX                                 ; /hWnd
0100877A   .  FF15 64120001 CALL DWORD PTR DS:\[<&USER32.GetMenu>\]    ; \\GetMenu
01008780   .  6A 08         PUSH 8                                   ; /Flags = MF\_BYCOMMAND|MF\_ENABLED|MF\_CHECKED|MF\_STRING
01008782   .  6A 1C         PUSH 1C                                  ; |ItemId = 1C (28.)
01008784   .  50            PUSH EAX                                 ; |hMenu
01008785   .  FF15 48120001 CALL DWORD PTR DS:\[<&USER32.CheckMenuIte>; \\CheckMenuItem
0100878B   .  61            POPAD
0100878C   .^ E9 88A8FFFF   JMP notepad1.01003019
01008791   >  C605 47870001>MOV BYTE PTR DS:\[1008747\],0
01008798   .  60            PUSHAD
01008799   .  6A 03         PUSH 3                                   ; /Flags = SWP\_NOSIZE|SWP\_NOMOVE
0100879B   .  6A 00         PUSH 0                                   ; |Height = 0
0100879D   .  6A 00         PUSH 0                                   ; |Width = 0
0100879F   .  6A 00         PUSH 0                                   ; |Y = 0
010087A1   .  6A 00         PUSH 0                                   ; |X = 0
010087A3   .  6A FE         PUSH -2                                  ; |InsertAfter = HWND\_NOTOPMOST
010087A5   .  52            PUSH EDX                                 ; |hWnd
010087A6   .  FF15 1A300101 CALL DWORD PTR DS:\[<&USER32.SetWindowPos>; \\SetWindowPos
010087AC   .  61            POPAD
010087AD   .  60            PUSHAD
010087AE   .  52            PUSH EDX                                 ; /hWnd
010087AF   .  FF15 64120001 CALL DWORD PTR DS:\[<&USER32.GetMenu>\]    ; \\GetMenu
010087B5   .  6A 00         PUSH 0                                   ; /Flags = MF\_BYCOMMAND|MF\_ENABLED|MF\_STRING
010087B7   .  6A 1C         PUSH 1C                                  ; |ItemId = 1C (28.)
010087B9   .  50            PUSH EAX                                 ; |hMenu
010087BA   .  FF15 48120001 CALL DWORD PTR DS:\[<&USER32.CheckMenuIte>; \\CheckMenuItem
010087C0   .  61            POPAD
010087C1   .^ E9 53A8FFFF   JMP notepad1.01003019

```

至此改造完毕，good luck  
  
今天让我们继续改造，添加最小化托盘功能，并且右键加个菜单  
首先先想想需要用到哪些函数？LoadIconW,Shell\_NotifyIcon,ShowWindow,SetForegroundWindow,CreatePopupMenu,AppendMenuA,GetCursorPos,TrackPopupMenu,DestroyMenu  
其中 Shell\_NotifyIcon 是用于添加和删除系统托盘图标的，是 shell32.dll 的函数  
        LoadIconW 用来获取一个图标句柄，用于做托盘图标  
        CreatePopupMenu 是用来创建弹出菜单的  
        AppendMenuA 是用来添加菜单项的  
        GetCursorPos 用来获取当前鼠标位置  
        TrackPopupMenu 是用来弹出菜单的  
        DestroyMenu   删除菜单  
        ShowWindow 显示或隐藏窗口  
        SetForegroundWindow 是用来将窗口置前，为什么会用这个，是因为在鼠标点击托盘图标后，恢复的窗口没有置前，在背后显示了，前后尝试了 SetActiveWindow 及 SetWindowPos 失败，最后还是这个函数管用  
       上面这些函数都是 user32.dll 的,其中 LoadIconW，GetCursorPos，ShowWindow 是源程序里就有的，其他需要用 lordpe 添加,还需要了解 NOTIFYICONDATA 结构，详见 Suyana  
的帖子 http://bbs.pediy.com/showthread.php?t=52515，很多地方是参照他的哈  
  
         因为源程序代码段留的空白空间不多，所以必须要新加个节，这里还碰到个问题，之前在 OD 里代码段末尾空白处添加代码，保存时提示“无法定位文件位置”，仔细想想，看看节的信息就会明白了，代码节大小为 7800，而映射进内存要按 1000 对齐，所以是 8000 大小，但实际只有前 7800 是文件内容，如果在 OD 里写到 1001000（起始地址）+7800=1008800 之后，保存时自然无法写入文件了  
        另外我用 lordpe 加节好像运行不了，运行会报错，后来还是用 ZeroAdd 加的，这里又碰到一个问题，在 DIY 之前必须想好所有添加功能，然后一次性加好输入函数，再添加新代码节，我当时是先实现了最小化托盘功能，后来再添加右键功能所需函数时，由于 lordpe 先前添加用于输入函数的节.silvana 过小，居然把数据覆盖到我加的代码段里了，害得我前功尽弃  
       好了，另外一个要注意的地方是，在 OD 里修改代码后保存时，新代码节和原.text 节要分开保存，比如改了.text 节，先保存后重载程序，再改新代码节，保存后重载，千万不要改了.text,又改了新节，再保存，重载后你会发现如果是在.text 节点右键保存的，你新节改的内容都没了，如果是在新节保存的，.text 节改的东西也白改了，我就是被这个整得吐血  
       另外一件倒霉的事是修改时存了好多版本，结果最终改好后删垃圾时，将改好的误删了，垃圾反而留下了，用工具化了好长时间扫描，最终也没恢复出来，只好重做。。。。。   
       总的来说上周末对我来说是个超黑的周末！！！！！  
       其他不说了，首先找到处理 WM\_SIZE 的代码如下  
代码:

```
010034C5   > \\8B45 10       MOV EAX,DWORD PTR SS:\[EBP+10\]            ;  EAX 里为 WM\_SIZE 的 wparam，最小化时为 1
010034C8   .  33F6          XOR ESI,ESI
010034CA   .  2BC6          SUB EAX,ESI 
010034CC   .  74 06         JE SHORT notepad7.010034D4                                
010034CE   .  48            DEC EAX                                  ;  Switch (cases 1..2)
010034CF   .  74 49         JE SHORT notepad7.0100351A                              ；这里就是跳到处理最小化的消息 0100351A
 

```

所以将 0100351a 处改为  
0100351A   > \\E9 A7520000   JMP notepad7.010087C6                    ;  Case 1 of switch 010034CE  
并在 010087c6 添加代码  
代码:

```
010087C6   > \\60            PUSHAD
010087C7   .  6A 00         PUSH 0                                   ; /ShowState = SW\_HIDE
010087C9   .  FF75 08       PUSH DWORD PTR SS:\[EBP+8\]                ; |hWnd
010087CC   .  FF15 B0110001 CALL DWORD PTR DS:\[<&USER32.ShowWindow>\] ; \\ShowWindow
010087D2   .  6A 02         PUSH 2                                   ; /RsrcName = 2.
010087D4   .  68 00000001   PUSH notepad7.01000000                   ; |hInst = 01000000
010087D9   .  FF15 EC110001 CALL DWORD PTR DS:\[<&USER32.LoadIconW>\]  ; \\LoadIconW
010087DF   .- E9 1CB80000   JMP notepad7.01014000                  ;01014000 是我新加的代码节
...............
01014000    A3 14410101     MOV DWORD PTR DS:\[1014114\],EAX       ;我将 NOTIFYICONDATA 结构数据预先放在 01014100 处，这里写入 hicon     
01014005    8B45 08         MOV EAX,DWORD PTR SS:\[EBP+8\]
01014008    A3 04410101     MOV DWORD PTR DS:\[1014104\],EAX       ；这里写入 hwnd
0101400D    68 00410101     PUSH notepad7.01014100
01014012    6A 00           PUSH 0
01014014    FF15 41300101   CALL DWORD PTR DS:\[<&shell32.Shell\_Notif>; shell32.Shell\_NotifyIconA
0101401A    61              POPAD
0101401B    8BE5            MOV ESP,EBP
0101401D    5D              POP EBP
0101401E    C3              RETN

```

再添加处理左键点击托盘消息处理  
代码:

```
0100343B   .- 0F87 DE0B0100 JA notepad7.0101401F
......
0101401F    81FE 05040000   CMP ESI,405
01014025  - 0F85 57F6FEFF   JNZ notepad7.01003682
0101402B    837D 10 00      CMP DWORD PTR SS:\[EBP+10\],0
0101402F  - 0F85 4DF6FEFF   JNZ notepad7.01003682
01014035    817D 14 0102000>CMP DWORD PTR SS:\[EBP+14\],201
0101403C    75 6A           JNZ SHORT notepad7.010140A8
0101403E    60              PUSHAD
0101403F    6A 09           PUSH 9
01014041    FF75 08         PUSH DWORD PTR SS:\[EBP+8\]
01014044    FF15 B0110001   CALL DWORD PTR DS:\[<&USER32.ShowWindow>\] ; user32.ShowWindow
0101404A    FF75 08         PUSH DWORD PTR SS:\[EBP+8\]
0101404D    FF15 BC300101   CALL DWORD PTR DS:\[<&USER32.SetForegroun>; user32.SetForegroundWindow
01014053    68 00410101     PUSH notepad7.01014100
01014058    6A 02           PUSH 2
0101405A    FF15 41300101   CALL DWORD PTR DS:\[<&shell32.Shell\_Notif>; shell32.Shell\_NotifyIconA
01014060    61              POPAD
01014061    8BE5            MOV ESP,EBP
01014063    5D              POP EBP
01014064    C3              RETN

```

先在处理 WM\_CREATE 消息时创建 POPUP 菜单  
代码:

```
01003434   .- E9 2F0C0100   JMP notepad7.01014068
.......
01014068    83FE 01         CMP ESI,1
0101406B    0F85 81000000   JNZ notepad7.010140F2
01014071    60              PUSHAD
01014072    FF15 C0300101   CALL DWORD PTR DS:\[<&USER32.CreatePopupM>; user32.CreatePopupMenu
01014078    A3 00420101     MOV DWORD PTR DS:\[1014200\],EAX          ;保存菜单句柄备后用     
0101407D    68 10420101     PUSH notepad7.01014210                   ; ASCII "restore"   自己先写好的菜单字符串
01014082    6A 1D           PUSH 1D
01014084    6A 00           PUSH 0
01014086    50              PUSH EAX
01014087    FF15 C8300101   CALL DWORD PTR DS:\[<&USER32.AppendMenuA>>; user32.AppendMenuA
0101408D    68 20420101     PUSH notepad7.01014220                   ; ASCII "exit"     自己先写好的菜单字符串
01014092    6A 1E           PUSH 1E
01014094    6A 00           PUSH 0
01014096    FF35 00420101   PUSH DWORD PTR DS:\[1014200\]
0101409C    FF15 C8300101   CALL DWORD PTR DS:\[<&USER32.AppendMenuA>>; user32.AppendMenuA
010140A2    61              POPAD
010140A3    EB 4D           JMP SHORT notepad7.010140F2
......
010140F2    83FE 1C         CMP ESI,1C                                          ;这里是恢复原来 01003434 的代码，返回执行
010140F5    57              PUSH EDI
010140F6    6A 08           PUSH 8
010140F8    5A              POP EDX
010140F9  - E9 3DF3FEFF     JMP notepad7.0100343B

```

处理托盘右键消息  
代码:

```
010140A8    817D 14 0502000>CMP DWORD PTR SS:\[EBP+14\],205
010140AF  - 0F85 CDF5FEFF   JNZ notepad7.01003682
010140B5    60              PUSHAD
010140B6    68 00430101     PUSH notepad7.01014300
010140BB    FF15 38120001   CALL DWORD PTR DS:\[<&USER32.GetCursorPos>; user32.GetCursorPos
010140C1    FF75 08         PUSH DWORD PTR SS:\[EBP+8\]
010140C4    FF15 BC300101   CALL DWORD PTR DS:\[<&USER32.SetForegroun>; user32.SetForegroundWindow
010140CA    6A 00           PUSH 0
010140CC    FF75 08         PUSH DWORD PTR SS:\[EBP+8\]
010140CF    6A 00           PUSH 0
010140D1    FF35 04430101   PUSH DWORD PTR DS:\[1014304\]
010140D7    90              NOP
010140D8    FF35 00430101   PUSH DWORD PTR DS:\[1014300\]
010140DE    90              NOP
010140DF    6A 08           PUSH 8
010140E1    FF35 00420101   PUSH DWORD PTR DS:\[1014200\]
010140E7    FF15 C4300101   CALL DWORD PTR DS:\[<&USER32.TrackPopupMe>; user32.TrackPopupMenu
010140ED    61              POPAD
010140EE    8BE5            MOV ESP,EBP
010140F0    5D              POP EBP
010140F1    C3              RETN

```

处理右键菜单选择后消息,这里紧跟着“置顶”(1c)的消息处理，restore 的 id 设为 1d,exit 的 ID 设为 1E,  
代码:

```
01008748   > \\6A 1C         PUSH 1C
0100874A   .  5B            POP EBX
0100874B   .  3BFB          CMP EDI,EBX
0100874D   .- 0F85 D7B90000 JNZ notepad8.0101412A
........
0101412A    83FF 1D         CMP EDI,1D
0101412D  ^ 0F84 0BFFFFFF   JE notepad8.0101403E
01014133    83FF 1E         CMP EDI,1E
01014136  - 0F85 DDEEFEFF   JNZ notepad8.01003019
0101413C    60              PUSHAD
0101413D    68 00410101     PUSH notepad8.01014100
01014142    6A 02           PUSH 2
01014144    FF15 41300101   CALL DWORD PTR DS:\[<&shell32.Shell\_Notif>; shell32.Shell\_NotifyIconA
0101414A    FF35 00420101   PUSH DWORD PTR DS:\[1014200\]
01014150    FF15 CC300101   CALL DWORD PTR DS:\[<&USER32.DestroyMenu>>; user32.DestroyMenu
01014156    6A 00           PUSH 0
01014158    FF15 F4110001   CALL DWORD PTR DS:\[<&USER32.PostQuitMess>; user32.PostQuitMessage
0101415E    61              POPAD
0101415F    8BE5            MOV ESP,EBP
01014161    5D              POP EBP
01014162    C3              RETN

```

改的时候整体结构考虑不周，很多地方留的空间不够，所以很多 jmp，莫见笑哈，good luck!  
  
2009.12.02  
继续改造，在标题栏右键菜单添加一项“置顶”，首先在创建主窗口后就可以往主窗口的系统菜单里添加项了  
代码如下：  
代码:

```
01004667   .  53            PUSH EBX                                 ; /lParam
01004668   .  56            PUSH ESI                                 ; |hInst
01004669   .  53            PUSH EBX                                 ; |hMenu
0100466A   .  53            PUSH EBX                                 ; |hParent
0100466B   .  FF35 709A0001 PUSH DWORD PTR DS:\[1009A70\]              ; |Height = 0
01004671   .  BE 94130001   MOV ESI,notepad8.01001394                ; |
01004676   .  FF35 749A0001 PUSH DWORD PTR DS:\[1009A74\]              ; |Width = 0
0100467C   .  FF35 7C9A0001 PUSH DWORD PTR DS:\[1009A7C\]              ; |Y = 0
01004682   .  FF35 789A0001 PUSH DWORD PTR DS:\[1009A78\]              ; |X = 0
01004688   .  68 0000CF00   PUSH 0CF0000                             ; |Style = WS\_OVERLAPPED|WS\_MINIMIZEBOX|WS\_MAXIMIZEBOX|WS\_SYSMENU|WS\_THICKFRAME|WS\_CAPTION
0100468D   .  56            PUSH ESI                                 ; |WindowName => ""
0100468E   .  68 20900001   PUSH notepad8.01009020                   ; |Class = "Notepad"
01004693   .  53            PUSH EBX                                 ; |ExtStyle
01004694   .  FF15 E0110001 CALL DWORD PTR DS:\[<&USER32.CreateWindow>; \\CreateWindowExW
0100469A   .- E9 C4FA0000   JMP notepad8.01014163                       ;注意是更改 notepad 主窗口的系统菜单，程序里调用 CreateWindow 的地方还有创建 edit 控件时，edit 控件的系统菜单是在编辑区右键弹出的那个
.............
01014163    60              PUSHAD
01014164    6A 00           PUSH 0
01014166    50              PUSH EAX
01014167    FF15 CC110001   CALL DWORD PTR DS:\[<&USER32.GetSystemMen>; USER32~1.GetSystemMenu
0101416D    A3 40420101     MOV DWORD PTR DS:\[1014240\],EAX        ;保存系统菜单句柄备后用
01014172    68 30420101     PUSH notepad8.01014230                            ;“置顶”字符
01014177    6A 1F           PUSH 1F
01014179    6A 00           PUSH 0
0101417B    50              PUSH EAX
0101417C    FF15 C8300101   CALL DWORD PTR DS:\[<&USER32.AppendMenuA>>; USER32~1.AppendMenuA
01014182    61              POPAD
01014183    8BD0            MOV EDX,EAX
01014185    3BD3            CMP EDX,EBX
01014187    8915 30980001   MOV DWORD PTR DS:\[1009830\],EDX
0101418D  - E9 1205FFFF     JMP notepad8.010046A4

```

添加对菜单项的消息处理，首先弹出菜单时，我们需要根据当前窗口是否已置顶来设置菜单状态，这需要处理 WM\_INITMENUPOPUP 消息  
代码:

```
01014192    81FE 17010000   CMP ESI,117               ;esi 里是消息码，117 是 WM\_INITMENUPOPUP
01014198    0F85 B2000000   JNZ notepad8.01014250
0101419E    50              PUSH EAX
0101419F    8B45 10         MOV EAX,DWORD PTR SS:\[EBP+10\]              ;窗口句柄
010141A2    3B05 40420101   CMP EAX,DWORD PTR DS:\[1014240\]     ；\[1014240\] 里保存的系统菜单句柄
010141A8    58              POP EAX
010141A9    0F85 A1000000   JNZ notepad8.01014250
010141AF    60              PUSHAD
010141B0    0FB605 47870001 MOVZX EAX,BYTE PTR DS:\[1008747\]  ；\[1008747\] 里保存着窗口是否置顶的状态
010141B7    C1E0 03         SHL EAX,3                                       ; checked 和 unchecked 对应 0 和 8,so .....
010141BA    50              PUSH EAX
010141BB    6A 1F           PUSH 1F
010141BD    FF35 40420101   PUSH DWORD PTR DS:\[1014240\]
010141C3    FF15 48120001   CALL DWORD PTR DS:\[<&USER32.CheckMenuIte>; USER32~1.CheckMenuItem
010141C9    61              POPAD
010141CA  ^ E9 23FFFFFF     JMP notepad8.010140F2

```

然后添加系统菜单选择“置顶”时消息处理,基本照搬上面的代码  
代码:

```
01014250    81FE 12010000   CMP ESI,112                           ;112 为 WM\_SYSCOMMAND,系统菜单项发出的
01014256  ^ 0F85 96FEFFFF   JNZ notepad8.010140F2
0101425C    837D 10 1F      CMP DWORD PTR SS:\[EBP+10\],1F    ;‘置顶“的 ID
01014260  ^ 0F85 8CFEFFFF   JNZ notepad8.010140F2
01014266    803D 47870001 0>CMP BYTE PTR DS:\[1008747\],1      
0101426D    74 38           JE SHORT notepad8.010142A7
0101426F    60              PUSHAD
01014270    C605 47870001 0>MOV BYTE PTR DS:\[1008747\],1
01014277    6A 03           PUSH 3
01014279    6A 00           PUSH 0
0101427B    6A 00           PUSH 0
0101427D    6A 00           PUSH 0
0101427F    6A 00           PUSH 0
01014281    6A FF           PUSH -1
01014283    FF75 08         PUSH DWORD PTR SS:\[EBP+8\]
01014286    FF15 B8300101   CALL DWORD PTR DS:\[<&USER32.SetWindowPos>; USER32~1.SetWindowPos
0101428C    FF75 08         PUSH DWORD PTR SS:\[EBP+8\]
0101428F    FF15 64120001   CALL DWORD PTR DS:\[<&USER32.GetMenu>\]    ; USER32~1.GetMenu
01014295    6A 08           PUSH 8
01014297    6A 1C           PUSH 1C
01014299    50              PUSH EAX
0101429A    FF15 48120001   CALL DWORD PTR DS:\[<&USER32.CheckMenuIte>; USER32~1.CheckMenuItem
010142A0    61              POPAD
010142A1  ^ E9 4CFEFFFF     JMP notepad8.010140F2
010142A6    90              NOP
010142A7    C605 47870001 0>MOV BYTE PTR DS:\[1008747\],0
010142AE    60              PUSHAD
010142AF    6A 03           PUSH 3
010142B1    6A 00           PUSH 0
010142B3    6A 00           PUSH 0
010142B5    6A 00           PUSH 0
010142B7    6A 00           PUSH 0
010142B9    6A FE           PUSH -2
010142BB    FF75 08         PUSH DWORD PTR SS:\[EBP+8\]
010142BE    FF15 B8300101   CALL DWORD PTR DS:\[<&USER32.SetWindowPos>; USER32~1.SetWindowPos
010142C4    FF75 08         PUSH DWORD PTR SS:\[EBP+8\]
010142C7    FF15 64120001   CALL DWORD PTR DS:\[<&USER32.GetMenu>\]    ; USER32~1.GetMenu
010142CD    6A 00           PUSH 0
010142CF    6A 1C           PUSH 1C
010142D1    50              PUSH EAX
010142D2    FF15 48120001   CALL DWORD PTR DS:\[<&USER32.CheckMenuIte>; USER32~1.CheckMenuItem
010142D8    61              POPAD
010142D9  ^ E9 14FEFFFF     JMP notepad8.010140F2

```

至此改造彻底结束，收工，good luck!     
  
2009.12.04  
试用发现自动换行等功能不对，是消息处理逻辑有误，重新更正  
  
2009.12.08  
试用发现如果修改了 txt 文件内容，托盘右键菜单“退出”时不会保存文件更改，重新更正  
代码:

```
01014133    83FF 1E         CMP EDI,1E                                   ;1E 即为托盘右键菜单项“退出”的 ID
01014136  - 0F85 DDEEFEFF   JNZ notepad9.01003019
0101413C    60              PUSHAD
0101413D    68 00410101     PUSH notepad9.01014100
01014142    6A 02           PUSH 2
01014144    FF15 41300101   CALL DWORD PTR DS:\[<&shell32.Shell\_Notif>; shell32.Shell\_NotifyIconA
0101414A    FF35 00420101   PUSH DWORD PTR DS:\[1014200\]
01014150    FF15 CC300101   CALL DWORD PTR DS:\[<&USER32.DestroyMenu>>; user32.DestroyMenu
01014156    61              POPAD
01014157    56              PUSH ESI                                                       ;下面这段代码是点击窗口菜单"文件“---”退出“的消息处理代码的复制
01014158    56              PUSH ESI
01014159    6A 10           PUSH 10
0101415B    52              PUSH EDX
0101415C    EB 71           JMP SHORT notepad9.010141CF
.....
010141CF    FF15 A4120001   CALL DWORD PTR DS:\[<&USER32.PostMessageW>; user32.PostMessageW
010141D5  - E9 8DF1FEFF     JMP notepad9.01003367

```

并将点击窗口菜单"文件“---”退出“的消息处理改为  
代码:

```
01002E8F   >-\\E9 A8120100   JMP notepad9.0101413C                    ;  Case 7 of switch 01002BBE

```

  
2010.01.06  
今天发现用 CTRL+C,CTRL+V 不正常，调试发现原来在处理这些消息时最后调用了 01002e94 的 PostMessage 函数，而我误删了，现恢复，重新上传附件 notepad9.rar  
  
2010.01.07  
添加字体颜色功能  
由于 notepad9.exe 里原添加输入表的节太小，无法再添加新的输入函数，所以直接在 notepad1.exe 的基础上改的，添加函数 gdi32.dll 的 SetTextColor 和 comdlg32 .dll 的 ChooseColorW 函数  
如何改变 EDIT 控件里的字体颜色，查找 MSDN 资料如下  
Each time an edit control is about to be painted to the screen, Windows sends the parent window a WM\_CTLCOLOR message. The wParam parameter contains the handle to the display context of the edit control, the low-order word of the lParam parameter identifies the edit control, and the high-order word of lParam contains CTLCOLOR\_EDIT.   
就是 EDIT 控件每次显示时会发送 WM\_CTLCOLOR 消息（好像应该是 WM\_CTLCOLOREDIT 消息），其中 wparam 里是 HDC,lparam 的低位是控件 ID  
CHOOSECOLOR 结构如下：  
typedef struct {   
  DWORD        lStructSize;   
  HWND         hwndOwner;   
  HWND         hInstance;   
  COLORREF     rgbResult;   
  COLORREF   \* lpCustColors;   
  DWORD        Flags;   
  LPARAM       lCustData;   
  LPCCHOOKPROC lpfnHook;   
  LPCTSTR      lpTemplateName;   
} CHOOSECOLOR, \*LPCHOOSECOLOR;   
我在 109790 的地方存放了这个结构，大小 24h  
紧跟着存放了 lpCustColors 指向的用户自定义颜色，有 16 个 RGB 数据，总共大小 40h  
代码如下  
代码:

```
0100343B    - 0F87 BF0B0100 JA notepad1.01014000
........
0100874D   . /75 77         JNZ SHORT notepad1.010087C6
..........
010087C6   > \\6A 1D         PUSH 1D
010087C8   .  5B            POP EBX
010087C9   .  3BFB          CMP EDI,EBX
010087CB   .^ 0F85 48A8FFFF JNZ notepad1.01003019
010087D1   .  60            PUSHAD                                   ;  Case 1D of switch 01002BBE
010087D2   .  68 90970001   PUSH notepad1.01009790                   ; /pChooseColor = notepad1.01009790
010087D7   .  FF15 3E300101 CALL DWORD PTR DS:\[<&comdlg32.ChooseColo>; \\ChooseColorW
010087DD   .^ EB E1         JMP SHORT notepad1.010087C0
........
01014000    81FE 33010000   CMP ESI,133              ;133 为 WM\_CTLCOLOREDIT
01014006  - 0F85 76F6FEFF   JNZ notepad1.01003682
0101400C    90              NOP
0101400D    A1 9C970001     MOV EAX,DWORD PTR DS:\[100979C\] 
01014012    50              PUSH EAX
01014013    FF75 10         PUSH DWORD PTR SS:\[EBP+10\]
01014016    FF15 A5300101   CALL DWORD PTR DS:\<&gdi32.SetTextColor>>; gdi32.SetTextColor
0101401C    6A 00           PUSH 0
0101401E    FF15 58100001   CALL DWORD PTR DS:\[<&GDI32.GetStockObjec>; gdi32.GetStockObject
01014024    8BE5            MOV ESP,EBP
01014026    5D              POP EBP
01014027    C3              RETN

```

  
  
 --------------------------------------------------------------------------------  
【版权声明】: 看雪论坛首发，转载请注明作者并保持文章的完整, 谢谢!

上传的附件
| ![ || notepad1.rar   || ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || notepad9.rar  
 |