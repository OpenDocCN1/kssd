
# `armadmp` 插件

> 原文：[`www.pediy.com/kssd/pediy06/pediy6443.htm`](https://www.pediy.com/kssd/pediy06/pediy6443.htm)

`Armadillo COPYMEMEII` 之 `DUMP` 的一个 `LOADPE` 小插件，下载（或鼠标右键另存为）。

用它来 `DUMP ARM COPYMEMII` 的程序，非常酷。

很多朋友不知如何使用，现附上使用方法：

1. 先把插件放在 `LOADPE` 的 `LDE` 目录下（没有这个目录的就自己建）。
2. 用 `OD` 打开要脱壳的程序，这里以一个自缟小程序为例。
3. 下断 `BP WaitForDebugEvent`，断下后看堆栈：

```
0012DAC0   004AAD67  /CALL to WaitForDebugEvent from ScreenCo.004AAD61
0012DAC4   0012EB84  |pDebugEvent = 0012EB84
0012DAC8   000003E8  Timeout = 1000. ms
```

在内存区定位在 `12eb84` 处，这样好找 `OEP`。

清断后再 `bp WriteProcessMemory`，`F9` 后看看：

```
0012D960   004AEC6E  /CALL to WriteProcessMemory from ScreenCo.004AEC68
0012D964   0000005C  |hProcess = 0000005C (window)
0012D968   00484000  |Address = 484000
0012D96C   00CF3FF0  |Buffer = 00CF3FF0
0012D970   00001000  |BytesToWrite = 1000 (4096.)
0012D974   0012DA7C  pBytesWritten = 0012DA7C
```

这里就是把缓冲区 `cf3ff0` 处的代码共 `1000H` 字节写进 `484000` 处，`OEP` 也在附近了：

```
0012EB84  01 00 00 00 00 05 00 00 E0 02 00 00 01 00 00 80  ......?....€
0012EB94  00 00 00 00 00 00 00 00 A0 40 48 00 02 00 00 00  ........燖 H....
0012EBA4  00 00 00 00 A0 40 48 00 A0 40 48 00 00 47 12 81  ....燖 H.燖 H..G
```

`OEP` 为 `4840a0`。

现在我们要做两件事：一要改 `OEP` 为死循环，二要 `NOP` 掉加密 `CALL`。

**1. 改 `OEP`**
可以用 `PUPE` 改也可以在缓冲区直接改，我用后一种。`CTRL+G`，输入 `00cf3ff0+0a0`（这里 `CF3FF0` 是缓冲区开始处，`0a0=4840a0-484000`，前面加个 `0` 防止 `OD` 判断为负数），定位到这里：

```
00CF4090    55            PUSH    EBP
00CF4091    8BEC          MOV     EBP, ESP
00CF4093    83C4 F0       ADD     ESP, -10
00CF4096    B8 A03E4800   MOV     EAX, 483EA0
00CF409B    E8 D01CF8FF   CALL    00C75D70
```

这就是程序 `OEP` 处的代码了，`558B` 改为 `EBFE`。

**2. `NOP` 掉加密 `CALL`**
老版可用 `ALT+K` 找，新版可在 `STACK` 往下找找就可以了。这里有这样一句：

```
0012D960   004AEC6E  /CALL to WriteProcessMemory from ScreenCo.004AEC68
0012D964   0000005C  |hProcess = 0000005C (window)
0012D968   00484000  |Address = 484000
0012D96C   00CF3FF0  |Buffer = 00CF3FF0
0012D970   00001000  |BytesToWrite = 1000 (4096.)
0012D974   0012DA7C  pBytesWritten = 0012DA7C
...
0012DA88  |004AD8DC  RETURN to ScreenCo.004AD8DC from ScreenCo.004ADC24
```

看到后面那个 `RETURN` 了吧，这就是加密 `CALL` 所在。在那里右键———`FOLLOW IN DESASMBLER`，到这里：

```
004AD8D7   .  E8 48030000   CALL    004ADC24     （这就是解码 CALL)               ;  ScreenCo.004ADC24
004AD8DC   >  83C4 0C       ADD     ESP, 0C      （返回到这）
```

我们搜索另一个 `call 004adc24`，在这里：

```
004ADBAB   .  50            PUSH    EAX
004ADBAC   .  E8 73000000   CALL    004ADC24                      ;  ScreenCo.004ADC24
```

把 `4adbac` 处 `NOP` 掉，清除所有断点然后 `F9` 运行，这样就可以用 `LOADPE` 来 `DUMP` 了。

用 `LOADPE` 选中子进程，右键———选择脱壳引擎------`armadmp`，然后再在子进程右键--完全脱壳。

`OK`，你已经成功 `DUMP` 了，记得要把 `OEP` 代码改回来。

经测试在 `2000/XP` 都可以成功 `DUMP`。`98` 没试过因为我们这已经没有用 `98` 的机子了。

`4dd466d0e1e0cc123afbe504da2462ca`
