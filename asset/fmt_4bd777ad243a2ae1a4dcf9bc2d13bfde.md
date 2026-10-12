
# 从逆向的角度看 .NET 的几个基本概念

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-315.htm`](https://www.pediy.com/kssd/pediy08/pediy8-315.htm)

从逆向的角度看几个 .NET 的基本概念

在英文版发了一篇，发现人气实在是低（人数＝喜欢 .net 的 ＆ 喜欢用英语看文章的），又想起了 `lccracker` 说过的，语言的障碍害死人啊，于是决定还是用母语写续篇。考虑到大多数人没看过前篇，因此一并写成中文，并加上新的内容。

最近我在看《Essential .NET》，因为在玩了一段时间逆向后，我觉得有必要从大局和根本上了解一下 .NET（学习－实践－再学习吗）。这本书虽主要从程序开发的角度上来讲 .NET，但对基本概念解释的比较清楚，也让我理清了脑中一些曾经模糊的概念。下文有的是直接引用原文（或翻译），有的则是我的总结。下面开始。

## Assembly

这个词用中文还不大好翻译，有的地方叫组合件，也算是表达它的意思。它比较容易和 `Module` 这个词混淆。来看书中怎么说：`An assembly is a logical collection of one or more modules`。这里有个词很关键，就是 `logical`，说明 `assembly` 只是一个逻辑上的概念。它由一个或多个 `Module` 组成，其中只有一个 `Module` 含有 `manifest information`。在程序中引用 `Assembly` 时，不需要加扩展名和路径（因为它不是文件），但调用它的最终结果是在一个文件路径（或网络地址）上找到组成它的 `Modules`。比如，.NET 系统最基础的 `mscorlib` 就是一个 `Assembly`，这里也没加扩展名。现在，你可以往下看 `Module` 的定义，两个结合起来理解就很容易了。

## Module

书中对 `Module` 的解释很到位：`A CLR module is a byte stream, typically stored as a file of the local file system or on Web server. In contrast, module is physical and assembly is logical`。很明显，我们通常说的文件就可以看成 `Module`，它是物理的概念。这里文件有多种：一是不包含 `Manifest Information` 的，这种文件没法直接运行，只能被主程序调用；一种是含有 `Manifest`，但是是 `Class Library`，由于扩展名通常是 `dll`，所以你可以把它看作 `win32` 的下的动态链接库，不同就是 `win32` 下的 `dll` 有入口程序 `DllMain`，但是 `Class Library` 不含 `Entry Point`；第三种就是我们用的最多的，含有 `Manifest`，也定义了入口程序的 `exe` 文件。（记住，定义了 `Manifest` 的 `module` 已经升级成为 `assembly` 了）下面通过命令行看一下三种文件各是怎么生成的：

```
代码:

Csc.exe /t:module md.cs
(创建名叫 md.netmodule 的 Module，当然，它不含 ManifestInfo)
Csc.exe /t:library /addmodule:md.netmodule lib.cs
(用一个等式来解释吧：lib.dll+md.netmodule->lib.dll assembly。这时 lib.dll 升级为 assembly)
Csc.exe /r:lib.dll main.cs
(创建 main.exe，它是个 assembly。它引用了（reference） lib.dll assembly，后者包含两个 Module: md and lib.)
```

## Type

`Type`（类型）是 .NET 程序最基本的构成元素，要不也不会有 `CTS`（`Common Type Sytem`）这个说法。最常用的 `Type` 就是 `Class`（类），然后是 `struct` 和 `enum`（结构和枚举）。`Type` 的成员最基本的有三类：`field`、`method` 和 `nested type`（就是类型中定义的类型）。还有两个常见的是 `property` 和 `event`，但这两个成员只是特殊的方法，比如 `property` 是实现了 `get` 和 `set` 的方法。看了这些，你应该会对 .NET `PE` 结构中 `Metadata` 数据的那几十个表，每个表是干啥的有个大体了解了，至少对我是这样。

## Interface

这个中文应该叫作接口。我在看 `Profiler` 和 `Debugger` 的源代码时，里面出现了很多接口，.NET 与 `COM` 的互操作也离不开接口。我非常想知道接口到底是什么，这本书给出的答案简单明了：`In the CLR, those categories of types are referred to as interfaces. Interfaces are always implicitly “abstract”`。可以这样理解，接口就是一类 `Type`，被归为该类的 `Type` 都具有一些功能。看一下代码片段，帮助我们理解：

```
代码:

public interface IEnglishWriter {}
public class    Person {}

public class ChineseWriter1 : Person, IEnglishWriter {}
public class ChineseWriter2 : Person {}
class PediyForumEnglishColumn
{
   void ReadPost(Person person) {}
   void PostNew (IEnglishWriter eperson) {}
}
```

接口是 `IEnglishWriter`，代表可以写英文贴子的人，然后 `ChineseWriter1` 是既懂中文又懂英文的版友，`ChineseWriter2` 就是只懂中文不懂英文的版友。看雪的英文版块(`PediyForumEnglishColumn`)里，只要是注册会员都可以看贴，相应代码是 `ReadPost(Person person)`，但由于英文版块只能发英文贴，代码是 `PostNew(IEnglishWriter eperson)`，所以 `ChineseWriter1` 可以发贴，而 `ChineseWriter2` 就没办法发贴了。

这里定义时用的“ : `IEnglishWriter` ”不叫“继承”，而叫兼容（`compatible`）。只要你兼容了某些接口，你就有了它的功能。

## hidebysig

这个词在 `ILDASM` 的反编译代码中经常出现的某些方法的前面，虽然和破解关系不大，但是理解它没什么坏处。假设基类和继承类中都有一个方法叫 `name1`，但参数不同。如果定义了 `hidebysig`，你就两个 `name1` 都可以调用，因为它的的 `signature` 是不同的。但如果定义了 `hidebyname`，完全按名字来区分的话，基类的 `name1` 方法就被“屏蔽”了。

## Reflection

这个中文叫反射，其实说白了就是对 `Metadata` 进行的操作。因为它能对元数据直接操作，所以相对来说它是比较低层的，也是比较有意思的。书中对它的描述在我看来比较详细。由于这是个大题目（大家可以参考 `rick` 在脱壳版里对某壳进行分析的代码），这里不多说了，网上资料很多。就提一下，在 `C#` 等托管代码中进行编程时可调用 `System.Reflection` 中的方法，在 `C++` 等 `win32` 平台下直接编程时，可调用 `IMetaData*` 的一系列接口进行操作，效果一样。而这种操作分为两大类，取得数据和生成数据。

## Property 和 Event

没啥多讲的，不难，略过。嘿嘿！

## CustomAttribute

自定义属性。这玩意是 .NET 比传统的编程语言先进之处的典型表现之一。说白了自定义属性就是给 `Type`（或 `Method` 等）添加额外的 `Metadata`，运行的时候可以动态的得到这些属性的值。自定义属性是存储在 `PE` 文件的 `BLOB` 中的。有两种自定义属性，一种是系统预定义的，一种完全是编程者自己定义的。剩下的和破解关系不大，不再赘述。

## Equivalent and identical

（暂且翻译为“相等”和“相同”）这是 .NET 中相等比较的两种方法，可能和破解有点关，因为很多程序将注册信息编为一个类，而不再是简单的字符串或数字。`Equivalent` 指的是数值相等，两个类，只要其中的各项 `field` 的数值相等就是 `equivalent`。但两个类永远不可能相同，这很简单，因为它们是两个类。更简单的理解方法，`win32` 下的指针，两个不同的类指针肯定不一样，这就是不相同。

## Method

（方法）这算是最重要的一个概念了，但我还没看完，简单说一点。注册码比较肯定是在某个方法中执行的，如果作者安全意识差点或者没用混淆程序，说不定方法的名称反编译出来还包含一些敏感字符串，比如 `register` 和 `activate` 等。.NET 是在第一次要调用一个方法时，开始即时编译的(`JIT`)。如果一个类中定义了方法的话，该类的结构中会含有一个方法指针，在 `JIT` 前，指针指向 `mscorwks.dll` 中，在编译后指针就指向 `asm` 代码了。这里没法贴图，建议大家去 `CSDN` 看一下这篇文章，就会对 `JIT` 的工作原理有个简单的了解。《深入理解 .NET 的 JIT 编译方式》

今天到这，下次继续，有错拍砖。

忘了，最后把我看的书的封面贴下，有兴趣阅读者可与我交流。

![](img/ce06b83e2f60953733560e10f5d9b681.jpg)

## 从逆向的角度看 .NET 的几个基本概念（续）——.NET 中的 Method

上篇讲了很多概念，但没有针对任何一个进行深入。这次我们来讲讲最有趣的 `Method`，它在 .NET 中的作用相当于 `win32` 下的 `function`。`Method` 之所以有趣正是因为它和 `JIT` 引擎及其它 .NET 内核的关系最为密切：`JIT` 得到 `Method` 的 `IL` 代码并把它编译成本机代码，如果在编译过程中遇到调用新的类，`JIT` 可能转到新类的编译中，内核要根据 `Method` 定义的属性决定代码是否能够调用一个 `Method`（当然，这个工作大多数在静态编译阶段完成），.NET 中的 `inline` 是怎样实现的。最关键的是，我们要寻找的敏感代码，肯定隐藏在某个（或数个）`Method` 中。

我们会从一个简单例子着手，通过动态调试的方法走进 .NET 的内核。

### 附件下载

例子代码如下：

```csharp
using System;

namespace tankaiha.dotnetsample.sample1

{

    public sealed class calcclass

    {

        public void calcCode(string strCode)

        {

            if(strCode=="tankaiha")

            {

                Console.WriteLine("You got it");

            }

            else

            {

                Console.WriteLine("You are wrong!");

            }

            return;

        }

    }

    class mainclass

    {

        static void Main()

        {

            Console.WriteLine("Please enter you code:");

            calcclass cs=new calcclass();

            cs.calcCode(Console.ReadLine());

            return;

        }

    }

}
```

代码有两个类，`mainclass` 类中只包含了 `Main()` 这个入口方法，并从命令行读入用户的输入。另一个类 `calcclass` 中只有一个方法 `calcCode()`，用来计算用户的输入是否等于 `tankaiha`，然后分别输出信息。

先来看看 `PE` 文件的 `Metadata`，特别是 `calcCode` 方法。.NET `PE` 头中，紧接着 `IMAGE_COR20_HEADER` 的就是 `Method Stream`，程序中的方法及其 `IL` 代码全部在这里。共有四个方法，除了我们写的 `main` 和 `calcCode` 外，还有系统自已加上的默认 `.ctor` 初始化方法。

![](img/e857221aa0cde6c0c1806ff65accca3d.jpg)

这里并没有定义 `Method` 的属性，只是 `Method Body` 和 `Method Head`。还记得过去讲过的 `Method Head` 分 `Tiny` 和 `Fat`，这里显示 `calcCode()` 方法为 `IMAGE_COR_ILMETHOD_FAT`，说明它是个 `Fat` 方法。

对于 `Method` 的定义，则是存储在 `Metadata` 的 `MethodDef` 表中。如下：

![](img/84aca3fbb3b6f932000b614ca08e0356.jpg)

这里有两项很有意思，一是 `ImplFlags`，另一个是 `Flags`。

`Flags` 分为三大类，一类是控制放问的 `Access`，比如你在程序中定义的 `public`、`private` 等。图中的 `ReuseSlot` 是牵涉到类在继承时方法的调用属性。第二类是控制 `InterOp` 的，第三类是额外属性。

`ImplFlags` 则分为两大类，一类是 `InterOp` 相关的，在我们的例子中不存在。另一类描述代码属性的，比如：

```
IL   0x0000   Method impl is CIL.

Native   0x0001   Method impl is native.

Managed   0x0000   Method impl is managed.

NoInlining   0x0008   Method may not be inlined.

NoInlining   0x0008   Method may not be inlined.
```

从图中看出我们代码属性很简单，就是 `IL` 托管代码。刚才表中最后一项 `NoInling` 说明该方法不可以被内联，本例中并没有这个属性。后面会看到它的作用。

看一下 `Main` 方法的 `IL` 代码，看一下 `C#` 编译到 `IL` 的大体情况。`Main` 方法的代码如下：

```il
.method private hidebysig static void  Main() cil managed

{

  .entrypoint

  // Code size       33 (0x21)

  .maxstack  2

  .locals init (class tankaiha.dotnetsample.sample1.calcclass V_0)

  IL_0000:  nop

  IL_0001:  ldstr     "Please enter you code:"

  IL_0006:  call      void [mscorlib]System.Console::WriteLine(string)

  IL_000b:  nop

  IL_000c:  newobj    instance void tankaiha.dotnetsample.sample1.calcclass::.ctor()

  IL_0011:  stloc.0

  IL_0012:  ldloc.0

  IL_0013:  call      string [mscorlib]System.Console::ReadLine()

  IL_0018:  callvirt  instance void tankaiha.dotnetsample.sample1.calcclass::calcCode(string)

  IL_001d:  nop

  IL_001e:  br.s      IL_0020

  IL_0020:  ret

} // end of method mainclass::Main
```

代码流程非常清楚，其中编译器为我们加上了一个局部变量用来保存对 `calcclass` 实例化的值，通过 `stloc.0` 和 `ldloc.0`。

`.locals init (class tankaiha.dotnetsample.sample1.calcclass V_0)`

而且代码中存在两种调用方式 `call` 和 `callvirt`。在本地代码中我们会看到它实现方法的不同。

下面开始动态调试，这里我们用 `OllyDbg`，这样可以直接跟踪 `JIT` 生成的本机代码。由于 `OD` 不能直接中断在 .NET 程序的入口处，所以在我们第一次运行时是无法中断的。这没事，我们可以在 `IL` 代码被第一次读取时中断。因此，先设置程序在载入新的模块时中断，当 `mscorwks` 被载入后，我们中断在它的 `79E9776D` 处。

![](img/50d21fb16804569da9f723f63f6b59db.jpg)

此处的代码如下
