
# .Net 环境下的程序破解(1)

> 原文：[`www.pediy.com/kssd/pediy06/pediy6476.htm`](https://www.pediy.com/kssd/pediy06/pediy6476.htm)

首先介绍一下 .Net 环境下的程序形成过程。[参照下图]

![](img/9d3b8bd1f6cc4fe7a6c607790ff5fe27.jpg)

(1) 程序员用高级 OO 语言(`C#`,`VB.Net`,`VC.Net`,`Cobol`,`Python`..)编写代码，这些代码经过 .Net 编译器（比如 `Visual Studio.Net`），编译成统一的 `MSIL`(Microsoft Intermediate Language)。NET 应用程序是以 `MSIL` 的形式出现的，只有在程序执行的时候才通过即时编译器 `JIT`(Just-In-Time)被编译为本机代码。

(2) 程序执行的时候，由 `JIT` 装入程序的 `MSIL`，`JIT` 同时作了很多其他的工作（装载相应的运行库，安全检测。。）最后 `JIT` 将转化成本地机器码(`EXE` 或者 `DLL`)。

(3) 本地机器码被装入内存，开始执行。

就是通过这几个步骤，.net 实现了编程语言无关（都转化成 `MSIL`）；平台无关（执行的时候根据本地配置，生成相应的机器码）；安全性（`JIT` 里面做了很多检测）。

作为 `Cracker`，可以用 .Net Framework SDK 自带的工具进行 `MSIL` 和 `EXE`(`DLL`)之间的互换：

利用 `FrameworkSDK\Bin\` 目录下的 `ilasm` 和 `ildasm`，安装了 .Net Framework 以后，这两个程序已经被加为系统路径，可以在“运行”里面输入“`ilasm`”或者“`ildasm`”直接打开。

## (1) `ilasm` 示例

下面的命令对 `MSIL` 文件 `myTestFile.il` 进行汇编并产生可执行文件 `myTestFile.exe`。

```
ilasm myTestFile
```

下面的命令对 `MSIL` 文件 `myTestFile.il` 进行汇编并产生 `.dll` 文件 `myTestFile.dll`。

```
ilasm myTestFile /dll
```

下面的命令对 `MSIL` 文件 `myTestFile.il` 进行汇编并产生 `.dll` 文件 `myNewTestFile.dll`。

```
ilasm myTestFile /dll /output:myNewTestFile.dll
```

## (2) `ildasm` 示例

下面的命令使 `PE` 文件 `MyHello.exe` 的元数据和反汇编代码显示在 `Ildasm.exe` 的默认 `GUI` 中。

```
ildasm myHello.exe
```

下面的命令对 `MyFile.exe` 文件进行反汇编，并将结果 `MSIL` 汇编程程序文本存储在 `MyFile.il` 文件中。

```
ildasm MyFile.exe /output:MyFile.il
```

下面的命令对 `MyFile.exe` 文件进行反汇编，并将结果 `MSIL` 汇编程序文本显示到控制台窗口中。

```
ildasm MyFile.exe /text
```

如果文件 `MyApp.exe` 包含嵌入的托管和非托管资源，则下面的命令将产生以下 4 个文件：`MyApp.il`、`MyApp.res`、`Icons.resources` 和 `Message.resources`：

```
ildasm MyApp.exe /output:MyApp.il
```

下面的命令对 `MyFile.exe` 的 `MyClass` 类中的 `MyMethod` 方法进行反汇编，并将输出显示到控制台窗口中。

```
ildasm /item:MyClass::MyMethod MyFile.exe /text
```

在上面的示例中，可能有几个具有不同签名的 `MyMethod` 方法。下面的命令对返回类型为 `void` 且带有参数 `int32` 和 `System.string` 的 `MyMethod` 方法进行反汇编。

```
ildasm /item:"MyClass::MyMethod(void(int32,class System.String))" MyFile.exe /text
```

通过上面分析可知，我们(`Cracker`)需要分析的对象就是 `MSIL` 形式的文件了，当然估计很快就有不少“混淆器”诞生，让我们无法从 `exe` 中恢复 `MSIL`，不过肯定也会有相应得“反混淆器”被开发出来，所以不必担忧，嘿嘿~~

下一贴我将介绍 `MSIL`，本人也刚开始研究 .Net 不久，所以写得有点乱，大家不要扔鸡蛋哦~~~ 呵呵

WinHack

QQ：85436

属于 `[CCG]`,`[OCN]`,`[YCG]`,`[FCG]`

2003-03-14 于看雪论坛 4dd466d0e1e0cc123afbe504da2462ca
