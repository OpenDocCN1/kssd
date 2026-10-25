
# 学用 CE，打造不死悍匪

> 原文：[`www.pediy.com/kssd/pediy10/78623.html`](https://www.pediy.com/kssd/pediy10/78623.html)

**【文章标题】**: 学用 CE，打造不死悍匪

**【文章作者】**: `stalker`

**【软件名称】**: `grand theft auto IV`

**【下载地址】**: 自己搜索下载

**【使用工具】**: `Cheat Engine` & `Ollydbg`

**【操作平台】**: `Windows XP SP2`

**【软件介绍】**: 很老但是很不错的一款单机游戏

**【作者声明】**: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

### 详细过程

`GTA` 是一个非常不错的单机游戏，虽然早就玩过，但无法上网的时候翻出来玩一玩还是很不错的。这次我们要在 `Cheat Engine` 的帮助下打造一个“永远不挂”的悍匪。

运行游戏，然后切换出来启动 `CE`（可以在这里下载 `http://www.websiteweather.com/downloads.php`），打开 `gta-vc.exe`，我们的目的是永远活着，首先就要找到存储血量的内存地址。由于我们事先不知道存储血量的变量类型，所以 `value type` 选择 `all`。如图所示

![](img/61058c3b296e859d9b1d4ec301279665.jpg)

`first scan` 结束之后，切换回到游戏，扔个炸弹在附近的地上（注意距离，不要一次把“自己”给弄死了），炸弹爆炸后残血了，记住当前血量，再切换出来进行 `next scan`，然后回到游戏，给自己来个阿司匹林（密码是 `aspirine`）把血加满，再切换出来进行一次 `next scan` 就能找到存储血量的内存单元了，在 `CE` 左边的列表框中双击该地址，将它添加到底部的列表中（在底部的列表中可以看到其类型为 `float`），如图

![](img/2cd46d4c7c60f79b837e127c53705a7a.jpg)

接下来在底部的列表中右击该地址，选择 `find out what writes to this address`

![](img/ebd61eecc73add89365a24aac98a1ec1.jpg)

此时 `CE` 会弹出一个对话框 `This will attach the debugger of Cheat Engine to the current process.Continue?`。大概的意思就是“这将会附加 `CE` 的调试器到当前进程，是否继续？”，选择 `yes` 就对了

现在需要切换回游戏，和上面一样再丢一次炸弹，然后切换出来，是否看到有结果了？

![](img/ecdefab499e1e5297d1c8c627bf18e02.jpg)

选中上图中的地址，然后点击右边的“`Show disassembler`”按钮

![](img/4e09b9e93464e899353a53c00bdd242e.jpg)

注意这三条指令，不熟悉浮点指令的朋友（比如我）可以参考 `http://www.pediy.com/tutorial/chap2/Chap2-4.htm`

```
005267DE   .  D983 54030000 fld     dword ptr [ebx+354]
005267E4   .  D86424 6C     fsub    dword ptr [esp+6C]
005267E8   .  D99B 54030000 fstp    dword ptr [ebx+354]
```

这里应该就是载入血量，减去某伤害对应的血量，存储减少后的血量，在 `CE` 的 `Memory Viewer` 中把上面的 `fsub` 改为 `fadd` 试试

切换回游戏，丢炸弹或者进行其他自残行为，是否发现血量不仅没有减少反而增加？

我们只需要将 `005267E5` 处的字节从 `64` 改为 `44` 就 `ok`，下面就来写一个简单的 `loader`

```
.386
.model flat,stdcall
option casemap:none

include \masm32\include\windows.inc
include \masm32\include\user32.inc
include \masm32\include\kernel32.inc
includelib \masm32\lib\user32.lib
includelib \masm32\lib\kernel32.lib

.data?

byteswritten dd ?

.data

dbNew db 44h
szTargetName db "gta-vc.exe",0
szErrorCreate db "启动目标进程失败",0
Startup STARTUPINFO <>
processinfo PROCESS_INFORMATION <>

.code
start:
  invoke CreateProcess,offset szTargetName,NULL,NULL,NULL,NULL,CREATE_SUSPENDED,NULL,
                    NULL,offset Startup,offset processinfo
  mov ecx,eax
  jecxz @f
  invoke WriteProcessMemory,processinfo.hProcess,005267E5h,offset dbNew,1,byteswritten
  invoke ResumeThread,processinfo.hThread
  ret
  @@:
  invoke MessageBox,0,offset szErrorCreate,0,MB_OK or MB_ICONERROR
  ret
end start
```

将编译好的 `loader.exe` 复制到 `GTA` 目录下，启动它，尽情玩耍![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")

图为"我"与 `Vice City` 的非政府武装对峙

![](img/790402844fdc4483e5715b0958ef5916.jpg)

你是否想起鲁迅先生的一句话？

> 真的猛士，敢于直面惨淡的人生，敢于正视淋漓的鲜血

--------------------------------------------------------------------------------

**【版权声明】**: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2008 年 12 月 13 日 17:45:01
