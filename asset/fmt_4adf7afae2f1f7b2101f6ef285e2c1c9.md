
``` 
004010C4    8B4424 08         mov     eax, dword ptr \[esp+8\]
004010C8    2D 10010000       sub     eax, 110                          ; Switch (cases 110..111)
004010CD    74 34             je      short Nag\_down.00401103
004010CF    48                dec     eax
004010D0    75 2D             jnz     short Nag\_down.004010FF
004010D2    8B4424 0C         mov     eax, dword ptr \[esp+C\]            ; Case 111 of switch 004010C8
004010D6    48                dec     eax
004010D7    75 26             jnz     short Nag\_down.004010FF
004010D9    6A 00             push    0
004010DB    FF7424 08         push    dword ptr \[esp+8\]
004010DF    FF15 18104000     call    dword ptr \[<&USER32.EndDialog>\]   ; USER32.EndDialog
004010E5    6A 00             push    0
004010E7    68 09114000       push    Nag\_down.00401109
004010EC    6A 00             push    0
004010EE    6A 65             push    65
004010F0    6A 00             push    0
004010F2    FF15 00104000     call    dword ptr \[<&KERNEL32.GetModuleHa>; kernel32.GetModuleHandleA
004010F8    50                push    eax
004010F9    FF15 10104000     call    dword ptr \[<&USER32.DialogBoxPara>; USER32.DialogBoxParamA
004010FF    33C0              xor     eax, eax                          ; Default case of switch 
004010C8
00401101    EB 03             jmp     short Nag.00401106
00401103    6A 01             push    1                                 ; Case 110 of switch 004010C8
00401105    58                pop     eax
00401106    C2 1000           ret     10
```

以上是警告框的处理程序，我们发现主对话框的显示调用是在警告框的 OK 按钮按下之后调用的。

也即是说，主对话框是在警告框的处理函数中调用的，我之前把这个警告框的处理函数全部跳过，当然也就不会调用主对话框显示程序了。

所以是我们的跳转跳得过度了。

所以我们要跳到警告框结束之后，并且在主对话框显示之前的代码处，那就是

`004010E5` . `6A 00` push 0 ; `/lParam = NULL`

把 `00401051` |.`6A 00` push 0 ; `/lParam = NULL`

双击或在 `00401051` 代码行右键-->汇编 `00401051`，在弹出的对话框中输入“`jmp 004010E5`”

这时，我们的代码变成红色，提醒我已经完成了改写

同样，再次右键-->复制到可执行文件-->保存文件，命名为 `Neg_nop.exe`

运行 `Neg_nop.exe` 没有警告框，直接显示了主对话框。

以上是方法一

------------------------------------------------------------

## 方法二

这个思路我是看到书中的方法

其实也不难想到

因为警告框和主对话框都是调用 `DialogBoxParamA` 函数，只是参数不同，所以就出现了不同的对话框，那么把警告框的参数改为主对话框的参数，不就是间接地跳过了警告框吗？

下面是警告框的参数：

```
00401051      6A 00         push    0
00401053      68 C4109090   push    909010C4                         ;  NEG 对话框处理函数
00401058  |.  6A 00         push    0                                ; |hOwner = NULL
0040105A  |.  6A 79         push    79                               ; |79 资源 ID，即 NEG
0040105C  |.  50            push    eax                              ; |hInst
0040105D  |.  A3 9C114000   mov     dword ptr \[40119C\], eax          ; |
00401062  |.  FF15 10104000 call    dword ptr \[<&USER32.DialogBoxPar>; \\DialogBoxParamA
```

下面是主对话框参数：

```
004010E5   .  6A 00         push    0                                ; /lParam = NULL
004010E7   .  68 09114000   push    Nag.00401109                     ; |DlgProc = Nag.00401109
004010EC   .  6A 00         push    0                                ; |hOwner = NULL 主对话框处理函数
004010EE   .  6A 65         push    65                               ; |pTemplate = 65  主对话框资源 ID
004010F0   .  6A 00         push    0                                ; |/pModule = NULL
004010F2   .  FF15 00104000 call    dword ptr \[<&KERNEL32.GetModuleH>; |\\GetModuleHandleA
004010F8   .  50            push    eax                              ; |hInst
004010F9   .  FF15 10104000 call    dword ptr \[<&USER32.DialogBoxPar>; \\DialogBoxParamA
```

不同的地方有 3 处：资源 ID 号、处理函数、父窗口句柄

资源 ID 号:由 `79H` 改为 `65H`

处理函数：由 push `004010C4` 改为 push `00401109`

父窗口句柄:这个参数就用默认的实例句柄，我不作修改。

好，改变参数后，再同样右键--->复制到可执行文件--->选择--->右键--->备份--->保存数据到文件

```
0040104D  /$  8B4424 04     mov     eax, dword ptr \[esp+4\]
00401051  |.  6A 00         push    0                                ; /lParam = NULL
00401053  |.  68 09114000   push    Nag\_ok03.00401109                ; |DlgProc = Nag\_ok03.00401109
00401058  |.  6A 00         push    0                                ; |hOwner = NULL
0040105A  |.  6A 65         push    65                               ; |pTemplate = 65
0040105C  |.  50            push    eax                              ; |hInst
0040105D  |.  A3 9C114000   mov     dword ptr \[40119C\], eax          ; |
00401062  |.  FF15 10104000 call    dword ptr \<&USER32.DialogBoxPar>; \\DialogBoxParamA
00401068  |.  33C0          xor     eax, eax
0040106A  \\.  C2 1000       ret     10
```

如下图所示，运行后程序运行成功，没有出现 Nag

![ (我最爱的仙剑四)......Nag 警告框的去除就学到这里。 5.3 时间限制 《加》书中谈了许多类的时间限制技术，但只给了一个 用 SetTimer 来定时的 CM。 这里先按书中来，以后有相关的其它时间限制的保护，再来补充 \--------------------------------------------------------------------------------------- 首先，运行 Timer.exe 看到每过 1 秒，右下方的时间计数框中时间加 1（现在是 6 秒），到 20 时就关闭了程序。 ![](img/b321100a5f89236c4367174b04e52756.jpg)

查下壳，不知道是什么语言写的，类型未知。

从哪里下手呢？

这时得从大脑的信息库中得到与时间相关的资料：

`SetTimer`---设定一个定时器。

`WM_TIMER`---定时消息，定义为常量 `0x113`

`KillTimer`---释放定时器（因为系统的定时器资源是有限的）

`GetTickCount`---获得从系统启动以来的运行时间。

`GetSystemTime`

`GetLocalTime`---上面这 2 个也是经常用了

`GetFileTime`

`FileTimetoSystemTime`

\--------------------------------

现在真刀实战开始。

OD 载入

如何看到用了什么 API 呢？

老方法---Ctrl+N,查看导入函数如下图：

![](img/cb7b73a5ce27ff9cb7c88f56214e354d.jpg)

与时间直接相关的有：

`KillTimer`

`SetTimer`

OK，从 `SetTimer` 下手“在每个参考上设置断点”

![](img/525fdfdf727eb032ffcc9545147a4516.jpg)

F9，运行，断下

```
 
004010C2 . 8B7424 08 mov esi, dword ptr \[esp+8\] ; Case 110 (WM\_INITDIALOG) of switch 004010A5
004010C6 . 6A 00 push 0 ; /Timerproc = NULL
004010C8 . 68 E8030000 push 3E8 ; |Timeout = 1000. ms
004010CD . 6A 01 push 1 ; |TimerID = 1
004010CF . 56 push esi ; |hWnd
004010D0 . FF15 30204000 call dword ptr \[<&USER32.SetTimer>\] ; \\SetTimer 断在这里
```

问题来了，怎么确定这个 `SetTimer` 就是关键的那个呢？

会不会有其它的 `SetTimer` 呢？

好，我觉得可以确定是这个，为什么呢？

我们看看参数，

`004010C8` . `68 E8030000` push `3E8` ; `|Timeout = 1000. ms`

说明定时为 1000 毫秒即 1 秒。

如果还不能确定，我们可以往下走，看看在 `WM_INITDIALOG` 消息中还加载了什么定时器。

```
 
004010C2 . 8B7424 08 mov esi, dword ptr \[esp+8\] ; Case 110(WM\_INITDIALOG) of switch 004010A5
004010C6 . 6A 00 push 0 ; /Timerproc = NULL
004010C8 . 68 E8030000 push 3E8 ; |Timeout = 1000. ms
004010CD . 6A 01 push 1 ; |TimerID = 1
004010CF . 56 push esi ; |hWnd
004010D0 . FF15 30204000 call dword ptr \[<&USER32.SetTimer>\] ; \\SetTimer 断在这里
004010D6 . A1 04304000 mov eax, dword ptr \[403004\]
004010DB . 6A 70 push 70 ; /RsrcName = 112.
004010DD . 50 push eax ; |hInst => 00400000
004010DE . FF15 2C204000 call dword ptr \[<&USER32.LoadIconA>\] ; \\LoadIconA
004010E4 . 50 push eax ; /lParam
004010E5 . 6A 01 push 1 ; |wParam = 1
004010E7 . 68 80000000 push 80 ; |Message = WM\_SETICON
004010EC . 56 push esi ; |hWnd
004010ED . FF15 14204000 call dword ptr \[<&USER32.SendMessageA>; \\SendMessageA
004010F3 . B8 01000000 mov eax, 1
004010F8 . 5E pop esi
004010F9 . C2 1000 ret 10
```

可以看到，在 RET 之前，只有 SetTimer+LoadIconA+SendMessageA,就是说，只有自定义的一个定时器加载和程序图标的加载操作。

所以说，这个 SetTimer 就是我们的解决目标，好，搞死搞残......

怎么搞呢？

我觉得：

1、把这个 SetTimer 给跳过，就没有定时器，也不会处理定时消息了，虽然源代码中会有 `WM_TIMER` 分支，但那已经成了空架子，不会被调用的。

2、为了不让它定时到 20 就自动结束程序，可以在计数变量加+1 处理，改为不加 1，不就 OK 了。

再就是可以在判断时，如 `if(i==20)`,我们改变代码，让 `i==20` 永远不会为真，不就 OK 了。

还可以做其它小动作，不过本质都是改变程序流程，让它不发送 `WM_CLOSE` 消息，就不会调用 `KillTimer`, `DestroyWindow` 函数了，不就 OK 了。

这个程序，让我有全逆向的冲动，虽然不是什么大程序，但很清晰了...后话。

先跳过 SetTimer 来实现去时间限制

`004010C6` `6A 00` push 0

改为

`004010C6` `/EB 0E` jmp short `Timer.004010D6`

```
```
