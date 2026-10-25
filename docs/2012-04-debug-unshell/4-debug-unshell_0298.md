
# ASPack v1.083

> 原文：[`www.pediy.com/kssd/tutorial/chap8-4-4.htm`](https://www.pediy.com/kssd/tutorial/chap8-4-4.htm)

**说明：如果用 OD 来脱，步骤比较简单，不必参考此文。(2010.12)**

压缩文件：下载。

如用 `TRW2000` 调试，装载后，下命令：`pnewsec` 就可中断到入口点。为了提高跟踪水平，下面以 `SOFTICE` 为例，讲述跟踪过程。

---

## 寻找程序的入口点（Entry Point）

用 `Symbol Loader` 打开 `Notepad-ASPACK.exe`，点击 `Symbol loader` 的第二个图标（"Load the currently open module")

如 `SOFTICE` 装载成功，它应中断在起始程序入口处。

当跟踪时，你会经过许多条件跳转指令及循环指令，要用点技巧跳出这些圈子，其形式一般这样：

```
aaaaaaaa
...
wwwwwwww
xxxxxxxx JNZ zzzzzzzz <-- 循环返回到 aaaaaaaa
yyyyyyyy JMP aaaaaaaa
zzzzzzzz 新的指令
```

如果你跟踪时，你将跳转到 `aaaaaaa` 处多次，你可将光标移到 `zzzzzzzz` 一行，按 `F7` 让指令直接跳到此处，可省不少时间。

在跟踪过程会来到：

```
167:0040D558 POPAD
0167:0040D559 JNZ 0040D563 (JUMP)
0167:0040D55B MOV EAX,00000001
0167:0040D560 RET 000C
0167:0040D563 PUSH EAX <-- EAX 的值就是入口点的值＝4010CC
0167:0040D564 RET <-- 返回到记事本的真正的入口点
```

```
0167:004010CC PUSH EBP <-- 真正的入口点
0167:004010CD MOV EBP,ESP
0167:004010CF SUB ESP,00000044
0167:004010D2 PUSH ESI
```

经过 `0167:0040D564 RET` 一行，程序将来到 `0167:004010CC` 刚完全解压的真正程序第一条指令处。

当我们跟踪时发现 `POPAD` 或 `POPFD`，就要提高警惕，一般入口点就在附近了，在此程序的原始入口点是 `004010CC`

在 `0167:0040D564 RET` 一行键入：

```
a eip
jmp eip
F5
```

这个方法是让程序挂起，按 `F5` 回到 windows 下。

运行 `Procdump`，在 `Task` 的列表中的第一个 `list` 上点击鼠标右键,然后选择"Refresh list". 在 `Task` 列表中找到 `notepad.exe`,在它的上面点击鼠标右键. 然后,选中"Dump (Full)",给脱壳的程序起名存盘. 再在 `notepad.exe` 上点击鼠标右键,然后选中"Kill Task".

---

## 修正入口点

脱壳的 `notepad.exe` 程序入口是 `004010CC`.

再次使用 `ProcDump` 的 `PE Editor` 功能, 打开已脱壳的 `notepad.exe`.

修正入口点值＝`4010CC`-`00400000`（基址）＝`10CC`

点击"OK".

现在, 运行脱壳后的 `notepad.exe` 吧,它应该正常运行了。

`4dd466d0e1e0cc123afbe504da2462ca`
