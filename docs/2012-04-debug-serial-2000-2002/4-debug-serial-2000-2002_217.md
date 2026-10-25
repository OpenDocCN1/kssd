
# Applet Marquee Wizard v3.5

> 原文：[`www.pediy.com/kssd/pediy03/forum312.htm`](https://www.pediy.com/kssd/pediy03/forum312.htm)

**破解者**：`punisher`（译者注：`punisher` 为原文作者）

**保护机制**：姓名/注册码

**破解过程**：
运行程序，出现主窗口和一个要你注册的 nag screen。点击这个破框上的 "Register" 按钮，弹出注册窗口（还有一个 preview 窗口，关掉它）。

键入你的名字和随便一个注册码。我用 `punisher` 和 `45454545`。

按 `Ctrl-d` 激活 `SOFTICE`，下断点 `BPX HMEMCPY`，按 `F5` 返回注册窗口。现在点击 "REGISTER"，程序被 `SOFTICE` 拦截。程序在到达保护机制前总共调用了三次 `HMEMCPY` 函数，我们必须略过前两次，所以按 `F5` 两次。我们现在是处在第三个调用 `HMEMCPY` 函数的 `CALL` 中，按 `F10` 直到这里：

```
0117:9E8E    shr  ecx, 2
0117:9E92    repz movsd       <- 将你的名字移到另一个位址
0117:9E96    pop  ecx
0117:9E98    and  ecx, 3
0117:9E9C    repz movsb
0117:9E9F    xor  dx, dx
```

在 `REPZ MOVSD` 时看一下 `EDI` 的内容：

```
D ES:EDI
```

将光标带过 `REPZ MOVSB` 时你会看见你的名字被拷贝到你 `D` 过的位址中。注意一下你名字的起始地址，它的形式是 `253F:00000000`（位址由于计算机的不同而各异）。我们将通过以下指令发现你名字的真实地址并在其上下断点：

```
PAGE 253F:00000000 <-你的地址会不同
```

你会看到：

```
Linear             Physical       Attributes       Type
xxxxxxxx           xxxxxxxx       P D A U         SYSTEM
```

你想要的是 `Linear` 的地址，所以你可以根据名字的字符数在该范围上下断点。你选择的段基址必须是 `30`，如下：

```
BPR 30:xxxxxxxx 30:xxxxxxxx+8 rw
```

现在按 `F5` 让程序继续运行，`SOFTICE` 会中断在此：

```
:00408DC9    repz cmpsb  <- 我们的名字和真名比较
:00408DCB    jz    00408DF1
:00408DCD    mov   al, [esi-01]
:00408DD0    cmp   al, 61
:00408DD3    jb    00408DDD
:00408DD5    cmp   al, 7A
:00408DD8    ja    00408DDD
```

在 `:00408dc9 repz cmpsb` 处，`D ESI` 会看到你的名字，`D EDI` 会看到两处 ALPHA 字符。第一处是和你的名字比较，第二处就是你的注册码了。还不快记下来!我看到的是 `p97z` 和 `23ws3`，输入后 OK! `4dd466d0e1e0cc123afbe504da2462ca`
