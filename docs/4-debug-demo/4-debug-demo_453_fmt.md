
# 让记事本[状态栏]不再受[自动换行]影响

> 原文：[`www.pediy.com/kssd/pediy11/115930.html`](https://www.pediy.com/kssd/pediy11/115930.html)

【软件名称】: `NOTEPAD.EXE`（来自 WinServer2003）

【下载地址】: 附件中

【操作环境】: Windows 7，OD

【作者声明】: 只是学习，没有其他目的。失误之处恳请各位指出!

Windows 自带的记事本轻巧方便，但使用中我们发现一个问题，就是不能同时选取【自动换行】和【状态栏】两个菜单项。

开发人员这么做必然有他自己的理由，但我们今天就试着来改变一下。

很自然的，我们想到修改记事本的窗口处理过程。那么我们开始吧！

OD 载入 `notepade.exe`，命令行下 `bp SendMessageW`。

F9 运行，断在 `USER32.SendMessageW`。

Alt+M 打开内存映射，在 `NOTEPAD` 的代码段（`.text`）F2 下访问断点。

F9 运行，断在下面：

```
01003449      8BFF          MOV EDI,EDI
0100344B  /.  55            PUSH EBP
0100344C  |.  8BEC          MOV EBP,ESP
0100344E  |.  51            PUSH ECX
0100344F  |.  51            PUSH ECX
01003450  |.  56            PUSH ESI
01003451  |.  8B75 0C       MOV ESI,DWORD PTR SS:[EBP+C]
01003454  |.  83FE 1C       CMP ESI,1C                       ;  Switch (cases 2..8001)
```

看看，呵，不错，直接到窗口函数入口了。往下拉一拉，也就看到了 `WM_COMMAND` 分支，如下：

```
010038E7  |> \3B3D 18980001 CMP EDI,DWORD PTR DS:[1009818]    ;  Case 111 (WM_COMMAND) of switch 01003454
```

现在就可以设置断点跟踪了。当然，如果想更有针对性的话，可以下条件断点。比如在入口 `01003449` 处，Shift+F4 打开条件记录断点窗口，如下图设置确定即可，同时删除其它干扰断点。

![](img/87f773debf2e48d7b2b8d0a6f6e33f9f.jpg)

为什么要这么设，原因我也讲不好![](img/110d920e4e703968a2a60c986d363163.jpg "尴尬")，大家参考论坛中的同类帖子和下面的链接（这里【自动换行】菜单 ID 为 0x20）：

http://bbs.pediy.com/showthread.php?threadid=8899

http://msdn.microsoft.com/en-us/library/ms647591(VS.85).aspx

好，F9 运行，记事本窗体出现了。现在点击【自动换行】项，不出意外的话，程序就断在了 `01003449` 处。F8 单步，很快就来到了下面：

![](img/90258473c8dbdfa1343977f857ed1b22.jpg)

```
01003942  |.  E8 60F2FFFF   CALL NOTEPAD.01002BA7            ; \关键，F7 进去。
```

上面 F7 跟进去，F8 单步，很快来到了下面：

![](img/d3b63926651175a45d4e74540cc65ac2.jpg)

```
0100315E  |> \A1 30980001   MOV EAX,DWORD PTR DS:[1009830]    ;  Case 20 of switch 01002BDE
01003163  |.  F7D8          NEG EAX                            ;  这个分支对应【自动换行】菜单项。
01003165  |.  1BC0          SBB EAX,EAX
01003167  |.  25 00001000   AND EAX,100000
0100316C  |.  05 04012050   ADD EAX,50200104
01003171  |.  50            PUSH EAX                           ; /Arg1
01003172  |.  E8 EA290000   CALL NOTEPAD.01005B61             ; \NOTEPAD.01005B61
01003177  |.  85C0          TEST EAX,EAX
01003179  |.  74 12         JE SHORT NOTEPAD.0100318D
0100317B  |.  33C0          XOR EAX,EAX
0100317D  |.  3935 30980001 CMP DWORD PTR DS:[1009830],ESI    ;  DS:[1009830]-->【自动换行】先前状态：0-未选中；1-选中。
01003183  |.  0F94C0        SETE AL
01003186  |.  A3 30980001   MOV DWORD PTR DS:[1009830],EAX
0100318B  |.  EB 1A         JMP SHORT NOTEPAD.010031A7
0100318D  |>  6A 30         PUSH 30                            ; /Style = MB_OK|MB_ICONEXCLAMATION|MB_APPLMODAL
0100318F  |.  FF35 3C900001 PUSH DWORD PTR DS:[100903C]        ; |Title = "记事本"
01003195  |.  FF35 54900001 PUSH DWORD PTR DS:[1009054]       ; |Text = "不能进行自动换行，因为该文件的正文太长。"
0100319B  |.  FF35 10980001 PUSH DWORD PTR DS:[1009810]       ; |hOwner = 00011564 ('无标题 - 记事本',class='Notepad')
010031A1  |.  FF15 64120001 CALL DWORD PTR DS:[<&USER32.MessageBoxW>>; \MessageBoxW
010031A7  |>  3935 30980001 CMP DWORD PTR DS:[1009830],ESI
010031AD      74 50         JE SHORT NOTEPAD.010031FF          ;  跳走即说明将要取消自动换行。
010031AF  |.  A1 20980001   MOV EAX,DWORD PTR DS:[1009820]    ;  【状态栏】先前状态：0-未选中；1-选中。
010031B4  |.  3BC6          CMP EAX,ESI
010031B6  |.  A3 24980001   MOV DWORD PTR DS:[1009824],EAX
010031BB      74 13         JE SHORT NOTEPAD.010031D0          ;  若已被选中，则取消选中并使之变灰。
010031BD  |.  56            PUSH ESI                           ; /lParam
010031BE  |.  53            PUSH EBX                           ; |wParam
010031BF  |.  68 11010000   PUSH 111                           ; |Message = WM_COMMAND
010031C4  |.  FFB5 F0FDFFFF PUSH DWORD PTR SS:[EBP-210]        ; |hWnd
010031CA  |.  FF15 3C120001 CALL DWORD PTR DS:[<&USER32.SendMessageW>; \SendMessageW
010031D0  |>  FF35 10980001 PUSH DWORD PTR DS:[1009810]       ; /hWnd = 00011564 ('无标题 - 记事本',class='Notepad')
010031D6  |.  FF15 60120001 CALL DWORD PTR DS:[<&USER32.GetMenu>]    ; \GetMenu
010031DC  |.  56            PUSH ESI                           ; /Flags
010031DD  |.  8B35 5C120001 MOV ESI,DWORD PTR DS:[<&USER32.GetSubMen>; |USER32.GetSubMenu
010031E3  |.  53            PUSH EBX                           ; |ItemId
010031E4  |.  8BF8          MOV EDI,EAX                        ; |
010031E6  |.  6A 03         PUSH 3                              ; |/Pos = 3
010031E8  |.  57            PUSH EDI                             ; ||hMenu
010031E9  |.  FFD6          CALL ESI                             ; |\GetSubMenu
010031EB  |.  50            PUSH EAX                             ; |hMenu
010031EC  |.  FF15 44120001 CALL DWORD PTR DS:[<&USER32.CheckMenuIte>; \CheckMenuItem（注：MF_CHECKED=0x8）
010031F2  |.  6A 01         PUSH 1                               ;  注：MF_GRAYED=0x1，使【状态栏】变灰。
010031F4  |.  53            PUSH EBX
010031F5  |.  6A 03         PUSH 3                              ; /Pos = 3
010031F7  |.  57            PUSH EDI                             ; |hMenu
010031F8  |.  FFD6          CALL ESI                             ; \GetSubMenu
010031FA  |.^ E9 3FFDFFFF   JMP NOTEPAD.01002F3E
```

```
01002F3E  |> /50            PUSH EAX                             ; |hMenu
01002F3F  |. |FF15 58120001 CALL DWORD PTR DS:[<&USER32.EnableMenuIt>; \EnableMenuItem
01002F45  |. |E9 3D040000   JMP NOTEPAD.01003387
```

```
010031FF  |> \FF35 10980001 PUSH DWORD PTR DS:[1009810]        ; /hWnd = 00011564 ('无标题 - 记事本',class='Notepad')
01003205  |.  FF15 60120001 CALL DWORD PTR DS:[<&USER32.GetMenu>]    ; \GetMenu
0100320B  |.  56            PUSH ESI                           ; /Flags
0100320C  |.  53            PUSH EBX                           ; |ItemID
0100320D  |.  6A 03         PUSH 3                              ; |/Pos = 3
0100320F  |.  50            PUSH EAX                             ; ||hMenu
01003210  |.  FF15 5C120001 CALL DWORD PTR DS:[<&USER32.GetSubMenu>] ; |\GetSubMenu
01003216  |.  50            PUSH EAX                             ; |hMenu
01003217  |.  FF15 58120001 CALL DWORD PTR DS:[<&USER32.EnableMenuIt>; \EnableMenuItem
0100321D  |.  3935 24980001 CMP DWORD PTR DS:[1009824],ESI    ;  以下恢复【状态栏】先前状态。
01003223      0F84 5E010000 JE NOTEPAD.01003387
01003229  |.  56            PUSH ESI
0100322A  |.  53            PUSH EBX
0100322B  |.  68 11010000   PUSH 111
01003230  |.  FFB5 F0FDFFFF PUSH DWORD PTR SS:[EBP-210]
01003236  |.^ E9 69FCFFFF   JMP NOTEPAD.01002EA4
```

```
01002EA4  |> /FF15 3C120001 CALL DWORD PTR DS:\<&USER32.SendMessageW>; \SendMessageW
01002EAA  |. |E9 D8040000   JMP NOTEPAD.01003387
```

可以看出，将

```
010031AD     /74 50        JE SHORT NOTEPAD.010031FF
```

修改为

```
010031AD     /EB 50        JMP SHORT NOTEPAD.010031FF
```

即可解决问题，实践证明确实可行。

修改并复制到可执行文件，运行，发现有个瑕疵，就是当记事本关闭时如果【自动换行】是勾选的，下次打开记事本的时候【状态栏】是灰色的。

现在解决这个问题，程序必定是在加载过程中对【自动换行】是否选中进行了判断。好，Ctrl+F2 重新开始，删除无用断点（若有的话），命令行下 `bp EnableMenuItem`，F9 运行。

断下，Alt+F9 返回到下面：

![

呵，又见 `DS:[1009830]`,稍微分析一下就很清楚了，运气不错，一下子就到我们想要的地方了。![](img/45dd044a2743704cbb333715eae4a5f6.jpg "狂笑")

```
0100483E  |. /74 20        JE SHORT NOTEPAD.01004860
```

修改为

```
0100483E     /EB 20        JMP SHORT NOTEPAD.01004860
```

保存为可执行文件（不要丢掉了上面的那个修改）。OK，运行，嗯，挺好，没问题了。

当然，修改这个貌似没什么必要，主要是学习，呵呵。

其它说明：

记事本的配置信息保存在注册表里，如下：




# Notepad 注册表与调试分析

```
Windows Registry Editor Version 5.00

\[HKEY\_CURRENT\_USER\\Software\\Microsoft\\Notepad\]
"iWindowPosX"=dword:00000106
"iWindowPosY"=dword:000000ff
"iWindowPosDX"=dword:000003af
"iWindowPosDY"=dword:00000197
"fWrap"=dword:00000000
"StatusBar"=dword:00000000
"lfEscapement"=dword:00000000
"lfOrientation"=dword:00000000
"lfWeight"=dword:00000190
"lfItalic"=dword:00000000
"lfUnderline"=dword:00000000
"lfStrikeOut"=dword:00000000
"lfCharSet"=dword:00000000
"lfOutPrecision"=dword:00000001
"lfClipPrecision"=dword:00000002
"lfQuality"=dword:00000002
"lfPitchAndFamily"=dword:00000031
"iPointSize"=dword:00000064
"fSaveWindowPositions"=dword:00000000
"lfFaceName"="Lucida Console"
"szHeader"="&f"
"szTrailer"="第 &p 页"
"iMarginTop"=dword:000009c4
"iMarginBottom"=dword:000009c4
"iMarginLeft"=dword:000007d0
"iMarginRight"=dword:000007d0
"fMLE\_is\_broken"=dword:00000000
```

各项含义比较容易理解，这里当然关注下面两项了：

`"fWrap"=dword:00000000`

`"StatusBar"=dword:00000000`

## 验证过程

为了验证上面的 `DS:[1009830]` 和 `DS:[1009820]` 存放的数据，`Ctrl+F2` 重新开始。在数据窗口中 `Ctrl+G` 跳转到 `1009830` 处，并在该处设置内存写入断点，如下图，同时删除无用断点：

![](img/67475bb227c7f7a53718cfb835376c44.jpg)

`F9` 运行，断在下面：

![](img/688dd78f97a605d56e18d9746edf37f6.jpg)

```
01003E2D  . |.  E8 7BFBFFFF    CALL NOTEPAD.010039AD                     ; \NOTEPAD.010039AD
```

上面的这个 `CALL` 我们重点关注下，在 `01003E1F` `F2` 下断。`Ctrl+F2` 重新开始，`F9` 运行，断在 `01003E1F` 处，`F7` 进入 `01003E2D` 处的 `CALL`，如下图：

![](img/18d7ff309501f2fccda4e37d86ae4016.jpg)

此时堆栈数据如下：

![](img/14825cc362d3ab92a166c17a1d2403e6.jpg)

可以看到，上面所说是正确的。同样，`1009820` 处的数据也可类似验证之，但这里我再重点介绍一下条件断点。

`Ctrl+F2` 重新开始，删除无用、干扰断点，在命令行下条件断点 `bp RegQueryValueExW,[UNICODE [esp+8]]=="StatusBar"`。`F9` 运行，断下，此时堆栈窗口数据如下：

![](img/d497bb8cd529b642e2f858209124f45a.jpg)

`Alt+F9` 返回到下面：

![](img/5a4a2f72c29bbb5eb986ebb593e80fae.jpg)

`F8` 单步直到返回，如下：

![](img/89c6038c2bbcbfbdeb13bb0892eb2072.jpg)

```
01003E4E  . |.  A3 20980001    MOV DWORD PTR DS:[1009820],EAX           ; |
```

上面已经可以说明问题了。

---

By D.L.  
2010/06/29

## 附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || notepad.rar  
|
