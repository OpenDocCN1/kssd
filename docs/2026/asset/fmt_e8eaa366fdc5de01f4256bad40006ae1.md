
# IDA简易教程

作者：www.datarescue.com

mailto:winroot@126.com

2004-11-20 初步翻译完成,希望大家指正错误,谢谢!

我的鸟语太差大概翻译了一下，大家凑合着能看懂就行了:-)有些地方用自己理解的意思改写了一下！

本文的主要内容就是讲如何用IDA来辅助识别各种类型的数据.

版权信息:所有版权归原文作者所有，如许转载清保持文章完整性。

索引

1

第一节:C语言的小程序

2

第二节:基本类型的识别

[3

第三节：操作数格式](https://www.kanxue.com/#第三节：操作数格式)

[4

第四节：字符和字符串的操作](https://www.kanxue.com/#第四节：字符和字符串的操作)

[5 第五节：数组](https://www.kanxue.com/#第五节：数组)

[6

第六节：枚举类型](https://www.kanxue.com/#第六节：枚举类型)

[7

第七节：Bit-fields（位域）](https://www.kanxue.com/#第七节：Bit-fields（位域）)

[8

第八节：结构体](https://www.kanxue.com/#第八节：结构体)

[9

第九节：结构变量和结构数组](https://www.kanxue.com/#第九节：结构变量和结构数组)

[10

第十节：联合体和结构体中的结构体](https://www.kanxue.com/#第十节：联合体和结构体中的结构体)

11

第十一节:可变的结构体

[12

第十二节：结构体偏移](https://www.kanxue.com/#第十二节：结构体偏移)

[13

第十三节：联合体偏移量](https://www.kanxue.com/#第十三节：联合体偏移量)

[14

第十四节：地址偏移量](https://www.kanxue.com/#第十四节：地址偏移量)

[15

第十五节：最终逆向结果](https://www.kanxue.com/#第十五节：最终逆向结果)

[附录](https://www.kanxue.com/#附录)

## 第一节:C语言的小程序

为了演示IDA的功能先写一段小程序。(代码在附录)[程序在此](https://www.kanxue.com/ida.files/data.rar) [源代码](https://www.kanxue.com/ida.files/data.CPP)

程序执行结果:

CUSTOMERS:

CUSTOMER 0001:

Peter (m)

CUSTOMER 0002: John (m)

CUSTOMER 0003: Mary

(f)

PRODUCTS:

PRODUCT 0001: BOOK: IDA QuickStart Guide

PRODUCT

0002: SOFTWARE: IDA Pro: PC; WINDOWS DOS; DISASSEMBLY

PRODUCT 0003: SOFTWARE:

PhotoRescue: PC MAC; WINDOWS OS-X; RECOVERY

PRODUCT 0004: SOFTWARE: aCrypt:

PC; WINDOWS; CRYPTOGRAPHY

[TOP](https://www.kanxue.com/#索引)

----------------------------------------------------------------------

## 第二节:基本类型的识别

用IDA分析我们前面的程序,我们会发现下图的类型

![](https://www.kanxue.com/ida.files/dbdwdd.gif)

只要按"D"我们就可以任意转换这些不确定的类型.可以变成byte,word,dword(`db`,`dw`,`dd`)。

当然你也可以设置更多的数据转换类型：

选择"*Options*"菜单的"*Setup data types*"命令就可以设置了

![](https://www.kanxue.com/ida.files/datatypedlg.gif)

值得注意的是：你在数据转换的时候，它是依据数据自身的结构来转换的。我们按"D"的时候，

如果下一个字节已经被你转换过，你的本次转换，IDA将会提示让你确认。

![](https://www.kanxue.com/ida.files/autoundef.gif)

注：如果你想改变这种默认设置可以在"*Options*"菜单"*Convert already defined bytes*"命令里设置

![](https://www.kanxue.com/ida.files/autoundefopt.gif)

**撤销你的所有转换按"U"键。**

[TOP](https://www.kanxue.com/#索引)

-------------------------------------------------------------------------------------

## 第三节：操作数格式

数据类型自定义转换后，被操作过的数据（就是你按过"D"的）的进制IDA也是可以自定义转变的，

通过在"*Operands*"工具栏的"*Number*"命令我们可以随意转换数字的进制

![](https://www.kanxue.com/ida.files/opnumber.gif)

最下面的"Toggle leading point"就是填补数据前的空位为0（就是说如果当前数据未占满数据格式的所有位高位用0来填补）

![](https://www.kanxue.com/ida.files/opzero.gif)

IDA还可以转换数据的标志位（就是正负）具体操作如下图：

![](https://www.kanxue.com/ida.files/opsignnot.gif)

最后呢~~如果这些转换你还不满意（够BT）当然你还可以自定义数据进制如图：

![](https://www.kanxue.com/ida.files/opmanual.gif)

[TOP](https://www.kanxue.com/#索引)

-----------------------------------------------------------------------------------------

## 第四节：字符和字符串的操作

作者又说话了：很多程序都是包含字符串的，一些被操作过的数据（就是你按过"D"的）可以转化为

字符，使用的命令就在"*Operands*"工具栏上

![](https://www.kanxue.com/ida.files/opchar.gif)

由于编程语言的不同造成字符串也有不同的格式，当然IDA就支持所有的格式了。

IDA在转化后会在地址添加一个名字。因为我们的程序是c的所以就找到c的字符串。具体操作如图：

![](https://www.kanxue.com/ida.files/string.gif)

如果不是C写的程序怎么办呢？我们可以在"*Options*"菜单"*ASCII string style*"命令中设置。

允许你修改其它类型为默认设置，使用默认设置的快捷键是"A"，或者自定义一种类型可以使用不常用的终止字符。

![](https://www.kanxue.com/ida.files/stringopt.gif)

[TOP](https://www.kanxue.com/#索引)

--------------------------------------------------------------------------

## 第五节：数组

在c中,ASCII字符串被认为是字符数组，IDA是如何处理数组的呢？

我们用最常用命令来定义数组中的第一个元素，设置第一个元素类型为byte，格式为char，然后点击"*"号键（或者"Edition"工具栏的"Array"命令）来创建数组。这时弹出一个对话框，可以设置很多变量。

![](https://www.kanxue.com/ida.files/array1.gif)

你可以定义数组一行的显示个数，还可以使用"Element width"来设置他们之间的宽度。

使用"Use dup construct"选项可以合并连贯的相似字节，"Display index"选项可以像注释一样显示数组的下标。

例如我们设置一个有64个元素的数组，一行有8个元素，每个元素之间的宽度为4，不选取"dup constructs"，选取"Display index"，我们就可以得到下面的数组。

![](https://www.kanxue.com/ida.files/array2.gif)

当IDA遇到未被识别的字节他会用红色的高亮显示。

当然你也可以选择一个范围来创建数组，IDA会自适应的设定。

![](https://www.kanxue.com/ida.files/array3.gif)

[TOP](https://www.kanxue.com/#索引)

------------------------------------------------------------------------------------

## 第六节：枚举类型

还记得我们在C程序中定义的`product_category_t`类型吗？

让我们用IDA的"Enumerations"来定义一下。

首先，我们打开"Enumerations"窗口来创建一个新的枚举类型

![](https://www.kanxue.com/ida.files/enumadd.gif)

我们输入我们的枚举类型值

![](https://www.kanxue.com/ida.files/enumaddmember.gif)

在`check_product()`函数，我们可以用枚举类型重新定义一些操作数。

右键点击在数值上，就会弹出一个菜单，选择"Symbolic constant"。

IDA就会自动列举枚举值，用以匹配当前的数值。

![](https://www.kanxue.com/ida.files/enumapply.gif)

操作完成，我们就会得到下面的结果：

![](https://www.kanxue.com/ida.files/enumapply.gif)

[TOP](https://www.kanxue.com/#索引)

-----------------------------------------------------------------------------------------------------

## 第七节：Bit-fields（位域）

BTW：Bit-fields，我的理解就是在结构体中的位标志。太菜！希望高手指正！

现在，我们来定义一下在`software_info_t`结构中的bitfields。

IDA的观点就是，bitefields是一种特殊的枚举类型。

我们可以选择在枚举类型创造窗口中的"Bitfield"选项。

![](https://www.kanxue.com/ida.files/bfadd.gif)

还记得我们曾经在我们的程序中建立了两种不同类型的bitfields，

`plateform`和`os`包含了一种隐藏的模式：用来包含组合模式（用逻辑或来操作）。因为一种产品可以同时在几种`plateforms`和OS的组合。另一方面，`category` bitfield中每一个数字表示一种类别：一种产品每次只能属于一种类别。

在IDA中一种指定的模式，bitfield只能包含一个值。所以在描述`plateform`和`category` bitfields时为了显示组合模式,我们必须创建一个小的bitfields，每个值的一个bit.

![](https://www.kanxue.com/ida.files/bfaddmember1.gif)

现在我们开始创建`category` bitfield。mask值为`0x3` (2 bits).我们指定一个名字、一个值、还有bitfield mask。我们还需要定义mask的名字：这个我们不用IDA自动生成的，IDA有一个内存助手可以帮助生成。

![](https://www.kanxue.com/ida.files/bfaddmember2.gif)

当所有的bitfields被输入，我们就会得到下面的结果：

![](https://www.kanxue.com/ida.files/bf.gif)

用`Operands`工具栏上的`Enum member`命令就可以定义我们程序中的`Enum member`数据

![](https://www.kanxue.com/ida.files/bfapply.gif)

[TOP](https://www.kanxue.com/#索引)

---------------------------------------------------------------------------------------------------------

## 第八节：结构体

我们的程序当中包含了很多结构体。现在让我们来在IDA中描述一下结构体，

看看是怎么提高汇编代码的可读性。

第一步，我们必须打开`Structures`窗口，来创建一个新的结构体类型。

![](https://www.kanxue.com/ida.files/stradd.gif)

结构体的成员是一汇编的模式定义的。让我们来定义`software_t`结构中的第一个成员。

一直按"D"知道它变成"dd"意思就是这个成员的值为`dword`类型。

把它的格式定义为我们以前定义好的`software_info_t`枚举类型，然后我们用`Rename`命令输入一个适当的名字：`info`

![](https://www.kanxue.com/ida.files/straddmember1.gif)

开始定义第二个成员，这次使用ASCII命令（按"A"），在这个环节

IDA会弹出一个专用对话框用来设定字符串的大小

![](https://www.kanxue.com/ida.files/straddmember2.gif)

我们还可以从已经分析好的数据中来建立结构体。

举个例子：假设我们选择了一块数据正好是和我们的`customer_t`结构体的数据格式一样，我们就可以用IDA的"Create struct from data"命令来创建结构体

![](https://www.kanxue.com/ida.files/strfromdata.gif)

一旦使用了这个命令,IDA就会在`Structures`窗口创建一个相对应的结构体

我们使用"A"键来修改`name`成员的长度为32bytes（和我们源代码中定义的一样），然后再给结构体一个好听的名字。

我们拿这些结构体有什么用呢？IDA提供给我们两种方法：

・Apply structure types to initialized data in the program.

・Convert operands as offsets inside structures.

我们将在下面的教程当中来介绍这两种方法

[TOP](https://www.kanxue.com/#索引)

---------------------------------------------------------------------------------------------------------------------

## 第九节：结构变量和结构数组

现在让我们用来`customer_t`结构体整理另外一个客户信息John。

把鼠标指针放在我们定义的结构体的第一个自己上面，然后使用`Struct var`命令。

![](https://www.kanxue.com/ida.files/strapply.gif)

这样我们就得到了一个新的结构体变量。IDA会自动在结构体成员的后面加上变量名的。

![](https://www.kanxue.com/ida.files/strapplied.gif)

通过我们的源码，我们知道有一个包含个4元素的`customers`数组，我们先前把

Peter和John都定义为`customer_t`结构体了。现在取消对Join结构变量的定义，然后在Peter结构上按"*"键创建我们的`customer`数组，这样IDA就弹出一个数组设置框他会自动检测出来我们要创建的数组最大数是4。

![](https://www.kanxue.com/ida.files/strarray1.gif)

下来我们就看到创建好的数组了，剩下的就是改一下数组的名字。

![](https://www.kanxue.com/ida.files/strarray2.gif)

[TOP](https://www.kanxue.com/#索引)

-------------------------------------------------------------------------------------------------------

## 第十节：联合体和结构体中的结构体

IDA中可以像定义标准结构体那样来定义联合体。

让我们来试着定义`product_u`这个联合体吧。

`book_t`和`software_t`这两个结构体我们已经定义过了。

IDA认为联合体就是一种特别的结构体：因此我们打开`Structures`窗口，运行`Add struct type`命令，在对话框中我们选择创建`Create union`选项。

![](https://www.kanxue.com/ida.files/unionadd.gif)

我们可以使用IDA常用的命令来创建联合体成员，分别添加一个`book_t`结构体类型的`book`和一个`software_t`结构体的`software`联合体成员

![](https://www.kanxue.com/ida.files/unionadded.gif)

当然结构体也可以嵌套一个结构体。事实上，我们刚才做的例子就实现了。

记住IDA认为联合体只不过是一种特殊的结构体

![](https://www.kanxue.com/ida.files/unionhint.gif)

[TOP](https://www.kanxue.com/#索引)
