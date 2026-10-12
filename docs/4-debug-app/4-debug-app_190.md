
# 扫雷 DIY

> 原文：[`www.pediy.com/kssd/pediy12/107602.html`](https://www.pediy.com/kssd/pediy12/107602.html)

扫雷都被大家玩烂了，我没有大家那么深的程度，只能在老婆的要求下小改改了，老婆碰到开不了雷而看到计时器上数字哗哗的跑很不爽，于是要求可以暂停，先考虑加个“暂停计时”菜单项，后来想想做隐蔽点好些，直接用热键吧，按一下暂停，再按一下恢复计时。

开始动工，首先用资源编辑工具添加热键 `F3`，对应 `ID` 为 `520`，^\_^。

接下来让我们研究下程序是如何改变计时数字的。

既然是计时，猜测程序是创建了计时器，并处理 `WM_TIMER` 消息，于是用 `OD` 载入运行，开一局开始计时，然后下 `WM_TIMER` 消息断点，来到处理代码。

代码:

```
01002FE0  /$  833D 64510001>CMP DWORD PTR DS:[1005164],0
01002FE7  |.  74 1E          JE SHORT winmine.01003007
01002FE9  |.  813D 9C570001>CMP DWORD PTR DS:[100579C],3E7
01002FF3  |.  7D 12          JGE SHORT winmine.01003007
01002FF5  |.  FF05 9C570001 INC DWORD PTR DS:[100579C]
01002FFB  |.  E8 B5F8FFFF    CALL winmine.010028B5
01003000  |.  6A 01          PUSH 1
01003002  |.  E8 E6080000   CALL winmine.010038ED
01003007  \> C3             RETN
```

从上面的代码可以猜测 `[1005164]` 里放的是计时是否开始的标志，`[100579C]` 里放的是计时数字，每次加 1，最大 999(`3e7`)。

所以我们只需将 `[1005164]` 改为 0 即可暂停，改为 1 就会恢复。

思路已定，现在来添加处理 `ID` 为 `520` 的 `WM_COMMAND` 消息对应代码，下 `WM_COMMAND` 消息断点，来到如下代码。

代码:

```
01001DE6  |.  3D 08020000   CMP EAX,208                      ;EAX 里为 ID 号
01001DEB  |.  0F8E B8030000 JLE winmine.010021A9
01001DF1  |.  3D 0B020000   CMP EAX,20B
01001DF6  |.  7E 61         JLE SHORT winmine.01001E59
01001DF8  |.  3D 0C020000   CMP EAX,20C
01001DFD  |.  74 50         JE SHORT winmine.01001E4F
01001DFF  |.  3D 0E020000   CMP EAX,20E
01001E04  |.  74 20         JE SHORT winmine.01001E26
01001E06  |.  3D 0F020000   CMP EAX,20F
01001E0B  |.  0F85 98030000 JNZ winmine.010021A9
01001E11  |.  33C0          XOR EAX,EAX
```

将代码改为：

代码:

```
01001DE6   .  3D 08020000   CMP EAX,208
01001DEB      0F8E 652C0000 JLE winmine.01004A56
01001DF1   .  3D 0B020000   CMP EAX,20B
01001DF6   .  7E 61         JLE SHORT winmine.01001E59
01001DF8   .  3D 0C020000   CMP EAX,20C
01001DFD   .  74 50         JE SHORT winmine.01001E4F
01001DFF   .  3D 0E020000   CMP EAX,20E
01001E04   .  74 20         JE SHORT winmine.01001E26
01001E06   .  3D 0F020000   CMP EAX,20F
01001E0B      0F85 98030000 JNZ winmine.010021A9
01001E11   .  33C0          XOR EAX,EAX
.....
01004A56   >  \3D 08020000   CMP EAX,208
01004A5B   .^ 0F85 48D7FFFF JNZ winmine.010021A9
01004A61   .  8035 64510001>XOR BYTE PTR DS:[1005164],1
01004A68   .^ E9 3CD7FFFF    JMP winmine.010021A9
```

呵呵，这下计时器乖乖的啦。

![](img/755d44573ea0045d4eb19f2164c82d4e.jpg "扮鬼脸")

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) | [`winmine.rar`](#) |
| :--- | :--- |
