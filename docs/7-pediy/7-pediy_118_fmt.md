
# epoxUSDM

> 原文：[`www.pediy.com/kssd/pediy03/forum577.htm`](https://www.pediy.com/kssd/pediy03/forum577.htm)

入门级 `smc` (self modifying code) 例子一个。

台湾磐英主板附带光盘上的监控软件 ===> `epoxUSDM.cpl` version 2.48

1. Easy to use, and convenient ControlPanel Applet
2. Can graphically show the status of the Power Supply and System-internal Temperature
3. Can show an Alarm Notification Window and Audible Alerts to potentially serious system problems before they threaten system stability

此程序必须用控制面板 `COMCTL32.DLL` 打开。

当然你只要双击 `epoxUSDM.cpl` 即可，就像其他可执行文件一样。

`smc` 原因：软件安装在开始/程序/启动中，每次开机自动运行后，在桌面显示一窗口并在任务栏右下角形成一小图标。

我愿望：开机时仅要任务栏右下角小图标不要在桌面出现窗口，这样较美观。但是双击任务栏右下角小图标时还能在桌面出现窗口。

难点：`cpl` 文件需用 `ControlPanel` 打开，不像 `exe`、`com` 等好跟踪。

首先：`PROCDUMP` 用 `PE EDITOR` 挑 `epoxUSDM.cpl` 选 `SECTIONS` 把 `.text` 的 `CHARACTERICTICS` 由 `60000020` 改为 `E0000020`，此步骤非常非常重要!!!!

原因：`60000020` 表示 `.text` 部分 readable executable
`E0000020` 表示 `.text` 部分 readable executable writeable
必须要 writeable!!!!!!!! 你可试试不改的下场。Ken 的 `smc` 教程就少了此步! 当然要视软件性质而定。你若无法确定改何处，可将整个 `section header` 的 `CHARACTERICTICS` 值全改 `E0000020`。

`TRW2K122` 跟踪发现：

在 `0167:10004E15` 处往下跳就出现桌面视窗，不往下跳就仅在任务栏出现小图标。

程序运行到 `10004E15` 时却总往下跳。你可能想，EASY，`NOP` 掉 `10004E15` 不就行了？的确，第一次运行是行了，但当你双击小图标时却无法出现桌面视窗，因为程序还要经过 `10004E15`，看来只好用 `smc` 法了。谢谢 Ken。

看看下面的程序（在 `epoxUSDM.cpl` 中），你会惊喜发现 `JZ` 附近很多 `NOP`，GOOOOOD！就改此处！因程序运行时没经过此 `NOP`，你可下中断试试，修改后不会影响原程序 `RUN`，不用 `TOPO` 找 `NOP` 了。

改前：

```
0167:10004DF4 90               NOP
0167:10004DF5 90               NOP
0167:10004DF6 90               NOP
0167:10004DF7 90               NOP
0167:10004DF8 90               NOP
0167:10004DF9 90               NOP
0167:10004DFA 90               NOP
0167:10004DFB 90               NOP
0167:10004DFC 90               NOP
0167:10004DFD 90               NOP
0167:10004DFE 90               NOP
0167:10004DFF 90               NOP
0167:10004E00 56               PUSH    ESI
0167:10004E01 6880C30010       PUSH    DWORD 1000C380
0167:10004E06 68F0A90010       PUSH    DWORD 1000A9F0
0167:10004E0B FF1518910010     CALL    `USER32!FindWindowA`
0167:10004E11 8BF0             MOV     ESI,EAX
0167:10004E13 85F6             TEST    ESI,ESI
0167:10004E15 7421             JZ      10004E38            ==>here!! 总是 jmp
0167:10004E17 56               PUSH    ESI
```

改为：

```
0167:10004DF4 66C705154E00107421 MOV     WORD [10004E15],2174  ==>change [10004E15]
0167:10004DFD EB18               JMP     SHORT 10004E17        ==>change 跳到 10004E17
0167:10004DFF 90               NOP
0167:10004E00 56               PUSH    ESI
0167:10004E01 6880C30010       PUSH    DWORD 1000C380
0167:10004E06 68F0A90010       PUSH    DWORD 1000A9F0
0167:10004E0B FF1518910010     CALL    `USER32!FindWindowA`
0167:10004E11 8BF0             MOV     ESI,EAX
0167:10004E13 85F6             TEST    ESI,ESI
0167:10004E15 EBDD             JMP     SHORT 10004DF4          ==>change 跳到 10004DF4
0167:10004E17 56               PUSH    ESI
```

运行完一次后，`10004E15` 变成原样：

```
0167:10004DF4 66C705154E00107421 MOV     WORD [10004E15],2174  ==>不会到此了
0167:10004DFD EB18               JMP     SHORT 10004E17        ==>不会到此了
0167:10004DFF 90               NOP
0167:10004E00 56               PUSH    ESI
0167:10004E01 6880C30010       PUSH    DWORD 1000C380
0167:10004E06 68F0A90010       PUSH    DWORD 1000A9F0
0167:10004E0B FF1518910010     CALL    `USER32!FindWindowA`
0167:10004E11 8BF0             MOV     ESI,EAX
0167:10004E13 85F6             TEST    ESI,ESI
0167:10004E15 7421             JZ      10004E38            ==>change 总是 jmp
0167:10004E17 56               PUSH    ESI
```

怎么样，改得简单吧，真心希望大家能看懂！

试了试，一点问题都没有！COOOOOOOOL！！

                              /\\zest/\\
                                2001.2  4dd466d0e1e0cc123afbe504da2462ca
