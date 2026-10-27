
# [盗版]发一个.net用的识别符号文件的库

发一个.net用的识别符号文件的库

在网上找了几天，没有.net用的，全是C++。虽然作为C++/CLI可以编译，但在.net工程下什么类也识别不了，一大堆类型得改。

还记得Symbo lType Viewer吗？看看它的帮助文档里有什么：“The sources will be placed at the disposal at the final version… after a good cleaning”，至今发给作者的求码E-mail杳无音讯。

实在懒得自己写，就把`SymbolTypeViewer.exe`给逆了。经过这几天的加工，基本上把逻辑代码和界面代码分离，所有的报错修正（基本上是语法问题，恶心死我）。

功能的话用过这个工具的人都知道，还原函数、结构、枚举。格式化为C或者IDC。

为了弄清楚里面的类，花了一天时间看MSDN写代码。把`Dbghelp`里许多`SymXx`搞了一遍。才敢写这篇文章（感兴趣的同学还可以看一下DIA SDK）。

简单说明一下：

![](https://bbs.kanxue.com/upload/attach/200907/112900_lnejvw7nqjutbbf.jpg)

这个空间包括各种符号类型，当然符号文件里面还有其他一些信息比如源代码行这里没有，很遗憾。还有一个重要的信息是符号地址，包括地址成员的只有`SymGeneral`。

![](https://bbs.kanxue.com/upload/attach/200907/112900_jzyanzlpmqn8h91.jpg)

`DbgService`完成一些常规例程 比如`SymInitialize`，设置符号所在目录等。

`ModuleSymbols`是主角，与一个模块对应，比如一个exe,dll或者pdb文件。

`ListSymbol`很有用，它也是`ModuleSymbols`的一个成员，在`ModuleSymbols`中，它表示模块里所有的符号，包括好几个`SortedList<uint, SymbolXx>`，`SymbolXx`是不同的符号类型，也就是`ListSymbol`包括几个以类型分类的符号列表，还有一个所有符号列表，这些列表是私有的，喜欢效率的同学可以考虑把它们`public`。

还有一个重要的空间

![](https://bbs.kanxue.com/upload/attach/200907/112900_s32z3cn7wdeyz5t.jpg)

这个空间是把符号们转换成C或者IDC用的，`Highlight`那三个类我忽略不计。

因为每个符号代表的信息有限，比如当我想要识别函数参数类型的时候，找到一个符号，它告诉我它是指针，什么指针呢？没有说，然后根据这个符号里的一个`Id`找到另一个符号，我靠它又是一个指针，那我继续找，哦它是个结构体。原来这个参数类型是个指向结构体的指针的指针。所以，我才有把这个命名空间也拿出来的动力，勤奋来源于懒惰。

`ParserSymbol.Format_C.GetUdtFormat()` 很好用，它把一个`SymUdt`打印出一个结构体，包括所有的元素在内，`SymUdt`类只有结构体的名字。`FormatMemberToString()`是把一个元素`SymData`格式化成C，比如`int32 size;`。类似的还有`ParserSymbol.FunctionFormat.GetFunctionFormat()`和`ParserSymbol.EnumFormat.GetEnumFormat()`，格式化整个函数和枚举。然后是格式化枚举的一个元素的`FormatMemberToString()`。

效果:

函数名有点怪，想显示正常些请改动程序主类除`Main`以外另一个的静态函数

![](https://bbs.kanxue.com/upload/attach/200907/112900_va6npr8g0mhahmo.jpg)

如果用`GetUdtFormat()`效果会更佳

![](https://bbs.kanxue.com/upload/attach/200907/112900_qr8mdm98a4bdo4k.jpg)

枚举

![](https://bbs.kanxue.com/upload/attach/200907/112900_wxzofylp2sxlehl.jpg)

主程序类里加了点东西。详见示例程序和类库代码

工程用VS2008创建,不知2005可否识别...

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#.NET平台](https://bbs.kanxue.com/forum-4-1-7.htm)

上传的附件：

- [testSTV.rar](javascript:void(0);) （74.01kb，90次下载）
- [Debug.jpg](javascript:void(0);) （14.78kb，345次下载）
- [Parser.jpg](javascript:void(0);) （30.83kb，344次下载）
- [Symbol.jpg](javascript:void(0);) （8.78kb，344次下载）
- [Enum.jpg](javascript:void(0);) （12.14kb，341次下载）
- [func.jpg](javascript:void(0);) （12.47kb，343次下载）
- [struct.jpg](javascript:void(0);) （18.80kb，343次下载）
