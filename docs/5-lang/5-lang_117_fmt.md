
# 揭开.NET 程序保护的秘密

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-353.htm`](https://www.pediy.com/kssd/pediy07/pediy7-353.htm)

## 主要形式

`.NET` 程序保护主要有如下几种形式：

1. 混淆
2. 编译到本地代码
3. 把代码隐藏在资源中

## 1. 混淆

这部分的保护软件以 `Dotfuscator` 和 `XenoCode` 为代表。

`Dotfuscator` 是比较初级的混淆器，采取的主要策略是名字混淆，通过把类名、方法名、变量名改成很短的形式，目的是破坏有意义的变量命名。

![](img/71526b83f95ee2cea1a9678e97028840.jpg)

`WebCombo.NET` 2.0，采用 `Dotfuscator` 进行变量名混淆

名字混淆的缺点在于：

a. 名字长度虽短，但是依然容易进行代码分析。对于熟悉汇编语言的逆向工程师，改成短的变量名对阅读代码基本不存在任何难度。

b. 在应用反射（`Reflector`）机制的程序中不能很好的工作。

`XenoCode` 是稍微高级的混淆器，它采用的技术除了变量名字混淆外，还加入流程混淆。目的是阻止 `.NET` `Reflector` 对其直接进行反编译。大部分采用了混淆器的 `.NET` 软件都是使用该混淆器进行保护。

![](img/ffc9b7af9ff187edfc77a6b14d7aa74b.jpg)

尽管 `.NET` `Reflector` 不能直接反编译为 `c#`，但是我们在语言栏把它切换到 `IL`，依然可以看到中间语言。

![](img/636c45663eca23a8fcba9b1fa3ec3830.jpg)

`.NET` `Reflector` 之所以不能反编译，是因为 `XenoCode` 把中间语言拆成若干段，每段用 `br` 或者 `br.s` (相当于 `jmp`) 衔接起来。至于为什么 `.NET` `Reflector` 不会解释成 `goto`，我也不太清楚，估计是因为与一条完整的 `C#` 语句等价的 `IL` 程序段被拆开两半。（哪位大侠知道请指教一下）。反混淆的方法也非常直接，把代码段重新排序就可以了，反混淆的结果如图。

![](img/f3229a0b744fd7ce50c6568ad17f0350.jpg)

用 `ida` `pro` 分析的流程图，未进行反混淆处理前：

![](img/3ca47bb12eb33d830eb3fbbfae636189.jpg)

反混淆处理后：

![](img/edc93d0a2f060de3574e88634349cd09.jpg)

对一个几 `M` 甚至几十 `M` 的 `il` 源文件进行反混淆不是一件轻松的事情，因此写了一个用于重新排序的小程序，正式做法类似编译原理的语句优化步骤，先把每个 `method` 分块，每个块的第一个语句是入口点，最后一个语句必须是无条件的控制转移语句（`br`、`leave`、`ret` 等）。然后把转跳点合并。这里遇到一个问题，遇到入口被 `method` 内部多处引用时，只能随机选择一个。因此部分函数还需要人手调整。但毕竟顺序结构还是占绝大多数，所以人手调整的比例还是很少的。我在转换一个几百个类的 `dll` 时，需要调整的类也只有 `10` 多个，是可以接受的。

显然，如果在有辅助工具的帮助下，`XenoCode` 流程混淆下的程序也不难做到反混淆。加上 `Reflector` 的 `FileDisassembler` 插件，完全可以把整个 `assembly` 的源码导出到本地目录下，再用 `IDE` 环境把项目源码导入后进行分析。

## 2. 编译到本地代码

保护软件以 `RemoteSoft` `Protector` 为代表，应用该保护方法的软件较少，因为正版的 `RemoteSoft` `Protector` 需要几千美金。其中代表软件为 `WebGrid.NET` 3.5。在发布的 `dll` 里面包括 `.NET` `assembly` 以及一个名为 `rscoree.dll` 的一般 `Win32` `PE` 文件。查看 `rscoree.dll` 可以发现导出表仅有 `_RSDllMain`, `_RSEEStartup`, `_RSEEUpdate`, `_RSExeMain` 四项，用 `Ollydbg` 打开发现并没加密。用 `Reflector` 打开 `.NET` `assembly`，无论是用 `C#` 语言查看还是直接查看 `IL`，所有 `method` 都只看到空的函数体。只有发现 `<PrivateImplementationDetails>` 中存在对 `rscoree.dll` 的引用信息。

```
.class private auto ansi <PrivateImplementationDetails>
       extends object
{
       .method assembly hidebysig static void $$method-1() cil managed noinlining
       {
       }
       .method assembly hidebysig static void $$method-2() cil managed noinlining
       {
       }
       .method private static pinvokeimpl("rscoree.dll" ansi nomangle) void _RSEEStartup(int32 A_0) cil forwardref managed
       {
       }
       .method private static pinvokeimpl("rscoree.dll" ansi nomangle) void _RSEEUpdate(native int A_0) cil forwardref managed
       {
       }
       .field private static bool $$started-1
       .field private static bool $$started-2
}
```

另外听说有一款国产的保护软件 `MaxtoCode`，分普及版和专业版，普及版可以免费使用。看软件的介绍信息疑也是采用类似技术。缺点是代码编译成了 native code，所以只能在 `windows` 平台运行。逆向人员需要同时懂得 `.NET` 虚拟机的知识和 `Win32` 汇编，要求较高。

## 3. 这是一种很另类的保护方法

（也许是我孤陋寡闻），目前只知道 `Reflector` 本身是采用这种方法来保护自身的核心代码库。暂时还没时间去研究具体的实现方法。

`4dd466d0e1e0cc123afbe504da2462ca`
