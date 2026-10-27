
# [原创]揭开.NET程序保护的秘密

.NET程序保护主要有如下几种形式：

1. 混淆
2. 编译到本地代码
3. 把代码隐藏在资源中

## 1. 混淆

这部分的保护软件以Dotfuscator和XenoCode为代表。Dotfuscator是比较初级的混淆器，采取的主要策略是名字混淆，通过把类名、方法名、变量名改成很短的形式，目的是破坏有意义的变量命名。

![](http://bbs.pediy.com/upload/2005/4/image/pic1.jpg)

WebCombo.NET 2.0，采用Dotfuscator进行变量名混淆

名字混淆的缺点在于

1. 名字长度虽短，但是依然容易进行代码分析。对于熟悉汇编语言的逆向工程师，改成短的变量名对阅读代码基本不存在任何难度。
2. 在应用反射（Reflector）机制的程序中不能很好的工作。

XenoCode是稍微高级的混淆器，它采用的技术除了变量名字混淆外，还加入流程混淆。目的是阻止.NET Reflector对其直接进行反编译。大部分采用了混淆器的.NET软件都是使用该混淆器进行保护。

![](http://bbs.pediy.com/upload/2005/4/image/pic2.jpg)

尽管.NET Reflector不能直接反编译为C#，但是我们在语言栏把它切换到IL，依然可以看到中间语言。

![](http://bbs.pediy.com/upload/2005/4/image/pic3.jpg)

.NET Reflector之所以不能反编译，是因为XenoCode把中间语言拆成若干段，每段用`br`或者`br.s`（相当于`jmp`）衔接起来。至于为什么.NET Reflector不会解释成`goto`，我也不太清楚，估计是因为与一条完整的C#语句等价的IL程序段被拆开两半。（哪位大侠知道请指教一下）。反混淆的方法也非常直接，把代码段重新排序就可以了，反混淆的结果如图

![](http://bbs.pediy.com/upload/2005/4/image/pic4.jpg)

用IDA Pro分析的流程图，未进行反混淆处理前：

![](http://bbs.pediy.com/upload/2005/4/image/pic5.jpg)

反混淆处理后：

![](http://bbs.pediy.com/upload/2005/4/image/pic6.jpg)

对一个几M甚至几十M的IL源文件进行反混淆不是一件轻松的事情，因此写了一个用于重新排序的小程序，正式做法类似编译原理的语句优化步骤，先把每个`method`分块，每个块的第一个语句是入口点，最后一个语句必须是无条件的控制转移语句（`br`、`leave`、`ret`等）。然后把转跳点合并。这里遇到一个问题，遇到入口被`method`内部多处引用时，只能随机选择一个。因此部分函数还需要人手调整。但毕竟顺序结构还是占绝大多数，所以人手调整的比例还是很少的。我在转换一个几百个类的DLL时，需要调整的类也只有10多个，是可以接受的。

显然，如果在有辅助工具的帮助下，XenoCode流程混淆下的程序也不难做到反混淆。加上Reflector的FileDisassembler插件，完全可以把整个assembly的源码导出到本地目录下，再用IDE环境把项目源码导入后进行分析。

## 2. 编译到本地代码

保护软件以RemoteSoft Protector为代表，应用该保护方法的软件较少，因为正版的RemoteSoft Protector需要几千美金。其中代表软件为WebGrid.NET 3.5。在发布的DLL里面包括.NET assembly以及一个名为`rscoree.dll`的一般Win32 PE文件。查看`rscoree.dll`可以发现导出表仅有`_RSDllMain`、`_RSEEStartup`、`_RSEEUpdate`、`_RSExeMain`四项，用OllyDbg打开发现并没加密。用Reflector打开.NET assembly，无论是用C#语言查看还是直接查看IL，所有`method`都只看到空的函数体。只有发现`<PrivateImplementationDetails>`中存在对`rscoree.dll`的引用信息。

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

另外听说有一款国产的保护软件MaxtoCode，分普及版和专业版，普及版可以免费使用。看软件的介绍信息疑也是采用类似技术。缺点是代码编译成了naïve code，所以只能在Windows平台运行。逆向人员需要同时懂得.NET虚拟机的知识和Win32汇编，要求较高。

## 3. 把代码隐藏在资源中

这是一种很另类的保护方法（也许是我孤陋寡闻），目前只知道Reflector本身是采用这种方法来保护自身的核心代码库。暂时还没时间去研究具体的实现方法。

[\[招生\]科锐逆向工程师培训（2026年11月13日实地，远程教学同时开班，第57期）](https://bbs.kanxue.com/thread-51839.htm)
