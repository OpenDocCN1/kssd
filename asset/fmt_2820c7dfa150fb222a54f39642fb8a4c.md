
# 发一个 .NET 用的识别符号文件的库

> 原文：[`www.pediy.com/kssd/pediy10/93344.html`](https://www.pediy.com/kssd/pediy10/93344.html)

发一个 .NET 用的识别符号文件的库。

在网上找了几天，没有 .NET 用的，全是 C++。虽然作为 C++/CLI 可以编译，但在 .NET 工程下什么类也识别不了，一大堆类型得改。

还记得 `SymbolTypeViewer` 吗？看看它的帮助文档里有什么：“The sources will be placed at the disposal at the final version… after a good cleaning”，至今发给作者的求码 E-mail 杳无音讯。

实在懒得自己写，就把 `SymbolTypeViewer.exe` 给逆了。经过这几天的加工，基本上把逻辑代码和界面代码分离，所有的报错修正（基本上是语法问题，恶心死我）。

功能的话用过这个工具的人都知道，还原函数、结构、枚举。格式化为 C 或者 IDC。

为了弄清楚里面的类，花了一天时间看 MSDN 写代码。把 `Dbghelp` 里许多 `SymXx` 搞了一遍。才敢写这篇文章（感兴趣的同学还可以看一下 `DIA SDK`）。

简单说明一下：

![](img/da27c901a8b6eb2aa398178d33d416d6.jpg)

这个空间包括各种符号类型，当然符号文件里面还有其他一些信息比如源代码行这里没有，很遗憾。还有一个重要的信息是符号地址，包括地址成员的只有 `SymGeneral`。

![](img/72b906ceef0ae303e9b6d69a41ad58c5.jpg)

`DbgService` 完成一些常规例程 比如 `SymInitialize`，设置符号所在目录等。

`ModuleSymbols` 是主角，与一个模块对应，比如一个 exe、dll 或者 pdb 文件。

`ListSymbol` 很有用，它也是 `ModuleSymbols` 的一个成员，在 `ModuleSymbols` 中，它表示模块里所有的符号，包括好几个 `SortedList<uint, SymbolXx>`，`SymbolXx` 是不同的符号类型，也就是 `ListSymbol` 包括几个以类型分类的符号列表，还有一个所有符号列表，这些列表是私有的，喜欢效率的同学可以考虑把它们 public。

还有一个重要的空间

![](img/99d98a8d8eebe08bcb82e4cd01acbd53.jpg)

这个空间是把符号们转换成 C 或者 IDC 用的，`Highlight` 那三个类我忽略不计。

因为每个符号代表的信息有限，比如当我想要识别函数参数类型的时候，找到一个符号，它告诉我它是指针，什么指针呢？没有说，然后根据这个符号里的一个 Id 找到另一个符号，我靠它又是一个指针，那我继续找，哦它是个结构体。原来这个参数类型是个指向结构体的指针的指针。所以，我才有把这个命名空间也拿出来的动力，勤奋来源于懒惰。

`ParserSymbol.Format_C.GetUdtFormat()` 很好用,它把一个 `SymUdt` 打印出一个结构体，包括所有的元素在内，`SymUdt` 类只有结构体的名字。`FormatMemberToString()` 是把一个元素 `SymData` 格式化成 C，比如“`int32 size;`”。类似的还有 `ParserSymbol.FunctionFormat.GetFunctionFormat()` 和 `ParserSymbol.EnumFormat.GetEnumFormat()`，格式化整个函数和枚举。然后是格式化枚举的一个元素的 `FormatMemberToString()`。

效果:

函数名有点怪，想显示正常些请改动程序主类除 `Main` 以外另一个的静态函数

![](img/11e4a0c075dc1477c2433b98204cc150.jpg)

如果用 `GetUdtFormat()` 效果会更佳

![](img/9d752e8153c0a12461f6c81a5c8d156a.jpg)

枚举

![](img/93a30428a1855cab92590df235209ead.jpg)

主程序类里加了点东西。详见示例程序和类库代码

工程用 `VS2008` 创建，不知 2005 可否识别...

上传的附件

| `![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg)` | `testSTV.rar` |
| :--- | :--- |
