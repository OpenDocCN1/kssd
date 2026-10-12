
# 有关 hexrays 插件时间限制的调试与解决

> 原文：[`www.pediy.com/kssd/pediy10/81056.html`](https://www.pediy.com/kssd/pediy10/81056.html)

本人现只能在此处发贴，如有不便表示歉意。

在论坛下了个 `Crack_New_Year_Presents_2009.iso`，也就是传说中的新年大礼包，并在其中翻出来了 IDA 的插件 `hexrays`，也就是把汇编翻译成伪 C 代码的软件。按照说明安装了一下。但是打开 IDA 运行插件时告知过期，很是郁闷。这么好的东西，难道非要自己破解不成。无奈之下写了这个程序供大家使用，如果觉得还可以管理员不妨把它加到新年大礼包中(谢谢)。

![](img/e76f1121a3da21b3d88ae0abaf47d31e.jpg)

简单说一下在 IDA 安装目录下的 `plugins` 子目录里找到 `hexrays.plw`，用 `winhex` 打开来到文件偏移 `FCCDC` 处，这里有连续 4 个字节存放着一个关键数据。如果全是 `00`，那么软件就把它解释成为一个初始时间就是 1970 年 1 月 1 日。插件安装完后这里是 `492c8370h`，换算成十进制是 `1227654000`，整除 `86400` 得到 `14208`，这个就是从 1970 年 1 月 1 日至今的天数，也就是 2008 年 11 月 26 日，因为 `14208` 大概就是 39 年。如果你没有按照安装说明修改，那么修改一下系统时间还可以用。如果你完全相信安装说明，你可以试试能否在 Windows 里把系统时间修改到 1970 年以前。

程序运行截了个图。

可能你对这个时间存放的地址 `FCCDC` 比较感兴趣。

当弹出显示过期时间的窗口时，你可以用个小工具看一下窗口名和窗口类，应该是用了 `createwindowexa`。

用 `od` 载入 IDA 后设置 `createwindowexa` 的断点，断下后在右边的堆栈中找到返回地址，直接跳过去是：

```
021B30DF    8B2D 10161202   MOV EBP,DWORD PTR DS:[<&USER32.CreateWindowExA>]          user32.CreateWindowExA

021B30E5    68 00008080     PUSH 80800000

021B30EA    68 88FC1F02     PUSH hexrays.021FFC88     ; ASCII "About window"

021B30EF    68 B8FC1F02     PUSH hexrays.021FFCB8     ; ASCII "Hex-RaysSplash"

021B30F4    6A 00           PUSH 0

021B30F6    FFD5            CALL EBP   ; 弹出提示过期时间的窗口
```

往上看发现：

```
021B2E87    68 70E12102     PUSH hexrays.0221E170                ; ASCII "f1bb5ea286819591579a2018b6edf166"

021B2E8C    68 E0E02102     PUSH hexrays.0221E0E0                ; ASCII "57-BF73-7BE4-3F"

021B2E91    51              PUSH ECX

021B2E92    E8 79B50100     CALL hexrays.021CE410
```

再由 Memory map `地址=0221E000`，`大小=00007000` (28672.)，`属主=hexrays`，`02120000` 可以知道 `0221E000` 处是插件的数据段，`"57-BF73-7BE4-3F"` 就在文件偏移的 `fcce0` 处，显然 `hexrays.plw` 插件数据段的内容就是读取了该处的文件数据。对比 `0221E000` 起始处的内容和插件偏移 `fcce0` 附近的数据，从而确定 `0221E000` 对应的文件偏移是 `fcc00`。

再重新调试很容易来到下面地址处：

```
0212A51C    A1 DCE02102     MOV EAX,DWORD PTR DS:[221E0DC]  ; 221E0DC 处的双字数据将会在下面生成日期

0212A521    99              CDQ

0212A522    890424          MOV DWORD PTR SS:[ESP],EAX

0212A525    8D0424          LEA EAX,DWORD PTR SS:[ESP]

0212A528    50              PUSH EAX

0212A529    895424 08       MOV DWORD PTR SS:[ESP+8],EDX

0212A52D    E8 68780A00     CALL hexrays.021D1D9A

0212A532    50              PUSH EAX

0212A533    68 60BC1F02     PUSH hexrays.021FBC60             ; ASCII "%d %b %Y"
```

所以得到日期数据相对于数据段起始地址的偏移 `0221E0DC－0221E000=dc`，这样在插件中的文件偏移就是 `fcc00+dc=fccdc`。

![](img/2ea91fd7822928858a0453416d9f7f98.jpg)

![](img/141e138e1690393c4ab76beb594aaaa6.jpg)
