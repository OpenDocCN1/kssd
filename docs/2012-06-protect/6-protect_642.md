
# Themida v1.8.0.0 Demo 虚拟机分析

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-808-3.htm`](https://www.pediy.com/kssd/pediy08/pediy8-808-3.htm)

## Handler 的基本执行流程

下面介绍 `handler` 的执行过程。不同类型的 `handler` 对 `VM_Context.flag` 的位有不同的解释，对于从 `PCODE.OpcodeDetail` 解码数据的使用也不同。这里只涉及 `handler` 共同的部分，与 `handler` 类型相关的细节在分析 `handler` 类别的时候再讲。

以 `Vm_Load` 指令为例，这个指令将 1 个 `imm32` 装入 `VM_Context.register`（即 `VM` 的寄存器），后续代码并未使用，实际上等同于垃圾指令。该指令不使用 `VM_Context.flag`，可以避免过多的细节。

![](img/a3708fbcae2bb4b50ad69ae669f5b69f.jpg)

进入 `handler` 后，将 `esi` 置为 `pcode` 数据地址，取 `Init_Keys` 字节，若 `b7` 为 `1` 则将 `Key1` 清 `0`。取 `InitKey1_and_NumOfRotate`（名取得很拙劣，但总得有个名字 J）字节的低 `2` 位，进一步变换 `Key1`。

![](img/7439f9a70a5359a064eb59ef7aa406d6.jpg)

`cl=1`。这些 `imm8` 是加壳时生成的随机数。

![](img/d3dea441061775fab75fd6bfefbd0f7d.jpg)

`cl = 2`。

![](img/91554d902cc5d8aa5900d64da6ea40f4.jpg)

`cl = 3`。

![](img/602a0597e8a19069c24be2e5f7660b51.jpg)

取 `InitKey1_and_NumOfRotate` 的 `b2-b4` `3` 位，这个值若为 `0`，`jz` 跳走。否则将 `VM_Context` 内的通用寄存器环滚动指定的字节数。这段代码看起来比较啰嗦，对于写解码器也没有影响，可以直接到 `jz` 的 `dst` 继续。

如果你想看看数据是怎么滚动的，可以用 `OllyDbg`，在进入 `VM` 前将各寄存器设为特定值。

![](img/81db92cc5b13b05ca56a520cd4f50552.jpg)

进入 `VM` 后，在 `dump` 窗口显示 `VM_Context`。

![](img/ef2cf6c2bec62e1dbf476fb6b73c4a1d.jpg)

设置停止条件，然后 `Ctrl-F11` 跟踪。

![](img/377d88168d7d41d537461bbe792ab0c7.jpg)

![](img/983d048f2da72ce1d629cdaddaeb7052.jpg)

![](img/e8edeacd344ca51d561a7e2ae235d3f3.jpg)

![](img/d93060b296f02a918360f32147497793.jpg)

很直观 J。注意不是每次都会滚动，寄存器的值也会因执行的操作而变化。

继续。下面解码 `PCODE.Argument`。

![](img/d9b78c4423cfe58eb43650ba544343e4.jpg)

![](img/1143298409622a806ab027744ae82361.jpg)

![](img/1e740e365802929523167e708f86ae48.jpg)

解码 `handler` 操作需要的细节数据，即上文中的 `OpcodeDetail`。`Vm_Load` 需要 `2` 个 `byte`。

![](img/a71d40de78f21043f06ee52a9dafe1a4.jpg)

这里将 `flag` 的值预置为 `4`。解码 `2` 个字节，根据解码结果设置 `VM_Context.flag`。具体的细节在分析特定类型的 `handler` 时再讲。

![](img/ac4822966c0af93d1903d21c032f96c0.jpg)

测试 `OpcodeDetail` 第 `1` 个字节的低 `7` 位，继续设置 `VM_Context.flag`。这是个 `switch-case`，无论走哪个分枝，都会在 `VM` 栈内压入 `1` 个 `dword`。细节后面再讲。

下面有很多 `caller` 的代码就是 `switch-case` 的出口。到这里，`VM` 栈内（与进入该 `handler` 时相比）压入了 `2` 个 `dword`，`[esp+4]` 为解码的 `Argument`，`[esp]` 是 `switch-case` 压入的 `dword`。

![](img/2d0c1f1fa11f3012262d57bf5b2be89a.jpg)

解码下 `1` 条指令的 `opcode2`。

![](img/4732f5568e1e884664a56cdceb69c6c0.jpg)

接下来这不起眼的 `2` 行，就是在执行 `Vm_Load` 了。将 `switch-case` 压入栈的 `dword` 送到 `VM_Context.register`。

![](img/47dce9e49b525b64c0f20aae86075d54.jpg)

解码下 `1` 条指令的 `opcode1`。

![](img/ebc0b99e72fa93a6d61e0037a802b648.jpg)

![](img/3470b445eb3e362fd67ba9e245bfac77.jpg)

解码下 `1` 条指令的 `pcode` 数据地址。

![](img/3add21558afafe085c0a977f40e9b3fd.jpg)

到这里结束当前 `handler`，执行下 `1` 条指令。

对于当前 `handler` 的执行，有几项数据是在执行实质的操作前必须解码的，即寄存器滚动字节数、`Argument`、`OpcodeDetail`。跳到下条指令需要的数据（`Next_pcode`、`Next_opcode1`、`Next_opcode2`）只要在 `jmp esi` 前解码即可，实际上 `Themida` 正是这样做的，这 `3` 项数据的解码操作位置是可变的。

在下面的部分，我们将详细分析 `4` 类 `handler` 的细节。

 4dd466d0e1e0cc123afbe504da2462ca
