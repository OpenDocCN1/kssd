
# 基于 OPENGL 引擎 3D 游戏逆向分析及汉化修改实例（上）（下）

> 原文：`www.pediy.com/kssd/pediy11/125694.html`

游戏全名：`Air Aces: Pacific`

中文译名：空中王牌：太平洋

下载地址：`http://www.verycd.com/topics/2832366/`

[【原创】基于 OPENGL 引擎 3D 游戏逆向分析及汉化修改实例（下）](http://bbs.pediy.com/showthread.php?t=125695)

这是一款模拟空战游戏，画面和操控性和市面上其他游戏相比较，算是比较粗糙的。不过这不是我关心的，因为游戏发行只有英文版，所以我想通过 `pediy` 的方法加入中文支持，也就是汉化。当然最好能借此找出一些通用的游戏汉化方法。

`OD` 载入主程序，输入表如下（部分）：

```
0B029B20   .idata     输入         OPENGL32.glRotatef
0B029B24   .idata     输入         OPENGL32.glScalef
0B029B28   .idata     输入         OPENGL32.glShadeModel
0B029B2C   .idata     输入         OPENGL32.glStencilFunc
0B029B30   .idata     输入         OPENGL32.glStencilOp
0B029B34   .idata     输入         OPENGL32.glTexCoord2f
0B029B38   .idata     输入         OPENGL32.glTexCoordPointer
0B029B3C   .idata     输入         OPENGL32.glTexEnvf
0B029B40   .idata     输入         OPENGL32.glTexEnvi
```

很明显这是一款基于 `OPENGL` 引擎的游戏。既然要想显示汉字，那么先了解 `OPENGL` 是如何显示文字的是非常必要的。`OPENGL` 引擎中显示文字通常有下列两种方法：

1. 常规字体。具体方法是首先用 `GDI` 的 `CreateFont` 为当前 `DC` 创建一个字体，然后调用 `wglUseFontBitmaps` 将当前 `DC` 中的字符批量转入显示列表，然后调用 `glCallLists` 显示文本串。
2. 纹理字体。即调用 `glGenLists` 创建显示列表，将字符纹理依次放入对应位置，然后调用 `glCallLists` 显示文本串。

以上两种方法要实现还是比较麻烦的，有需要的话可以参阅相关文档，这里不再赘言，毕竟我们无意开发游戏。通过对游戏的简单调试分析，可以确定，游戏使用的是第二种方法，也就是使用纹理字体来显示文字。并且很容易就能在 `gfx` 目录找到这个用于存放字体的纹理 `png`，如图：

![](img/22e4a8d211c1851f45a6b1de0bc4eb51.jpg)

看到这图有人可能会说，只要把字母 `ps` 成汉字，就可以显示中文了。的确是这样的，但是英文字母只有 26 个，而翻译完文本后使用的汉字远不止 26，所以我们必须想办法扩容字库。首先应该统计翻译时使用了多少个汉字，用这些汉字重新建立一张码表。然后根据码表扩容上面的字库。

这也正是一些单字节游戏在汉化时的难点所在：必须要找到游戏生成显示列表的代码并修改。所幸的是，这款游戏比我想象的简单。

既然已经确定了是第二种方法，那么在 `glGenLists` 下断试试，选中输入函数直接回车，查找输入函数参考发现以下几处调用：

```
参考位于 AAP:.text 到 OPENGL32.glGenLists
地址       反汇编                     注释
00415872   call    <jmp.&OPENGL32.glGenLists>
0041595B   call    <jmp.&OPENGL32.glGenLists>
00439A56   call    <jmp.&OPENGL32.glGenLists>
0048E748   jmp     dword ptr [<&OPENGL32.glGenLists  OPENGL32.glGenLists
```

第一个 `call`：

```
0041586B   .  C70424 010000>mov     dword ptr [esp], 1
00415872   .  E8 D18E0700   call    <jmp.&OPENGL32.glGenLists>
```

第二个 `call`：

```
00415954   .  C70424 010000>mov     dword ptr [esp], 1
0041595B   .  E8 E88D0700   call    <jmp.&OPENGL32.glGenLists>
```

第三个 `call`：

```
00439A4F  |> \\C70424 000100>mov     dword ptr [esp], 100
00439A56  |.  E8 ED4C0500   call    <jmp.&OPENGL32.glGenLists>
```

`glGenLists` 接受一个参数，表示要创建的显示列表的数量，前两个参数为 1，显然不可能只有一个字符，所以前两个 `call` 都排除掉。而第三个 `call` 参数为 `0x100` 即 256，正好与 `16*16` 的字体纹理 `png` 对应，应该就是这里了。

我们详细看看 `00439A56` 处的 `call` 完整的函数：

```
