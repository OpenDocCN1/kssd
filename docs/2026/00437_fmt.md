
# OllyHTML脚本详解（一）结构

【文章标题】: OllyHTML脚本详解（一）结构

【文章作者】: dreaman

【作者邮箱】: `dreaman_163@163.com`

【作者主页】: `f12K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8J5k6h3q4E0j5h3&6Q4x3X3g2Z5j5h3!0D9K9h3&6B7N6g2)9J5k6h3&6W2N6l9`.`.

【软件名称】: OllyHTML

【软件大小】: `800KB`

【下载地址】: `55fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8J5k6h3q4E0j5h3&6Q4x3X3g2Z5j5h3!0D9K9h3&6B7N6g2)9J5k6h3&6W2N6l9`.`.

【加壳方式】: 无

【保护方式】: 无

【编写语言】: `c++`

【使用工具】: `vc7.1`

【操作平台】: `win2000`以上系统

【软件介绍】: Ollydbg的插件，用以支持用DHTML作脚本

【作者声明】: 个人觉得用DHTML作脚本是能做一些比较复杂的事情的，希望大家能用用：）

写完OllyHTML与IdaHTML插件后一直希望能看到有人能用上它，没想到事与愿违，因为我自己的破解功底很浅，所以又写不出好的脚本来，
但我又感觉用HTML+javascript的方式是可以适用于需求变化频繁然而又确实有规律可循的场合的（我想破解与逆向算是这种场合的一种吧）
，这便有了详细介始一下OllyHTML脚本的想法，希望我能够把它说明白，更大的愿望是希望能有人用它写点什么，比如把自己的破解与逆向
的经验表达成一些脚本：）

本教程计划分为五个主要部分：

        OllyHTML脚本详解（一）结构

        OllyHTML脚本详解（二）基本原理

        OllyHTML脚本详解（三）静态分析API

        OllyHTML脚本详解（四）动态调试API

        OllyHTML脚本详解（五）其它API

主要以介始OllyHTML插件提供的API为主，配以少量的例子说明API的用法，因为OllyHTML插件的API主要是对Ollydbg插件API的封装，所以有许
多API能直接对应到Ollydbg的插件API上，这样的API的说明都是按我的理解所写，错误之处还望大家海涵：）我在IdaHTML插件中也加了调试功
能，不过感觉Ida Pro对调试的支持还是没法与OD相提并论，而且也不稳定，不过OD的静态分析没有Ida Pro强大，因为OllyHTML与IdaHTML插件
是基于同样的内核实现的，它们之间是可以通信的，如果写完上面说的五个部分还有空并且能找到需要结合OD与IdaPro的实例，偶可能会加一个
OllyHTML与IdaHTML协作的部分，这是后话，暂且搁下吧。

今天是第一部分，偶是第一次写教程，所以请大家不要拍砖：）

## Hello world 示例

按照一般教程的做法，我们首先来一个Hello world，因为是要写脚本，所以我们假设您已经了解HTML与javascript语言的简单用法。

```html
<html>
        <head>
                <title>Hello world</title>
                <script src="res://OllyHTML.dll/const.js"></script>
        </head>
        <body>
                <table style="width:100%;height:100%">
                        <tr>
                                <td>
                                        <textarea id="resultArea" style="width:100%;height:100%" rows="10"></textarea>
                                </td>
                        </tr>
                        <tr style="height:24px">
                                <td>
                                        <input type="button" value="执行" onclick="doStart()">
                                </td>
                        </tr>
                </table>
                <script>
                        function doStart()
                        {
                                resultArea.value="Hello , World !";
                        }
                </script>
        </body>
</html>
```

将上面这段脚本保存成`test.htm`，然后用`OllyHTML=>Load=>Html`，指定文件`test.htm`，就会看到脚本装入后的样子了，点一下上面的“执行”
按钮，就会看到上面的文本显示区出现：Hello , World !

(对大多数人看到这里就可打住了，请直接看下一篇！)

这个脚本其实与OllyHTML没有太大关系（唯一有关的是引用了插件的常量资源文件），只是一个普通的HTML网页，不过它展示了OllyHTML
脚本的主要框架。

脚本首先是一个HTML文件，所以它的最外层架子是这样的：

```html
<html>
        <head>
        </head>
        <body>
        </body>
</html>
```

也就是有一个HTML头部与一个HTML BODY。

HTML头部主要用来放置标题，样式表，脚本引用之类，我们的脚本里是一个标题与一个插件常量文件引用，标题的写法如下：

```html
<title>Hello world</title>
```

您只需要将其中的Hello world替换为您自己的脚本标题就可以了。插件常量文件引用的写法总是这样的，在您的脚本里原样拷贝就可以了：

```html
<script src="res://OllyHTML.dll/const.js"></script>
```

HTML BODY部分可以放置用户可见的各种元素与用户脚本（脚本也可以放在HTML头里，不过放在BODY的尾部可以更好的访问可见元素），
用户可见的元素总是需要布局的，所以用户可见的部分嵌在一个TABLE里面，我们想像一下一个脚本通常需要与用户交互的东东：

1、执行信息显示
2、执行用户命令的按钮

我们的hello world里就是只用了这两个简单的东东：

```html
<table style="width:100%;height:100%">
        <tr>
                <td>
                        <textarea id="resultArea" style="width:100%;height:100%" rows="10"></textarea>
                </td>
        </tr>
        <tr style="height:24px">
                <td>
                        <input type="button" value="执行" onclick="doStart()">
                </td>
        </tr>
</table>
```

我们用了一个二行一列的TABLE来布局我们的两个与用户交互的元素，用来显示信息的我们用的是TEXTAREA，它是一个多行文本控件，
我们给了它一个ID：`resultArea`，这就相当于在脚本里定义了一个名为`resultArea`的变量，它的类型就是TEXTAREA,不了解这些也没关系，
因为我们差不多总是只用到一个访问它的方法：

```javascript
resultArea.value
```

执行用户命令的按钮如下：

```html
<input type="button" value="执行" onclick="doStart()">
```

我们给它一个显示名“执行”，然后为它指定一个当用户点击时要调用的脚本函数:

```javascript
doStart()
```

上面的UI虽然简单，但我们的大多数脚本只需要它们就足够了，当然，您对DHTML很熟的话，您可以做得很炫：）

接下来是脚本的主体了，脚本总是嵌在一个这样的标签里：

```html
<script>
</script>
```

好了，现在关于HTML的部分就说完了，下面是关于javascript语言的了。

## javascript 基本用法

javascript语言是相当灵活与强大的，不过我们现在只说最简单的用法，也就是如何写一个前面UI里指定的当用户点击时要调用的函数，
它是这样的：

```javascript
function doStart()
{
        resultArea.value="Hello , World !";
}
```

上面我们定义了当用户点击“执行”按钮时调用的函数，函数名为`doStart`，它没有参数，函数体只有一条语句，它将“hello, world !”字符
串赋给`reaultArea.value`，也就是显示信息给用户的那个控件。

javascript是一种类C语言，它的主要语句与操作都与C相同，主要的不同是javascript不是强类型的，也就是javascript里的所有变量都
不需要指定类型，我们定义一个变量时只需要这样：

```javascript
var varName;
```

或者，在定义变量的同时给变量指定一个初始值：

```javascript
var varName=0;
```

其它的我们就不多说了，因为我们主要目的是写脚本，这一篇主要是为了说明一个脚本像什么样子，呵呵，有关HTML与javascript的内容，
可以参考MS的MSDN的WEB编程部分或者上网随便搜一下，遍地都是。

(估计有人开始大呼上当了，这搞什么嘛，明白的不看就明白，不明白的看N遍还是不明白！下回我们开始入正题，着重介始OllyHTML为脚本
提供的对象模型。)

## JSON 简写语法

附带的提一下javascript语言的一类简写语法，这些语法被广泛用于ajax技术中，这就是JSON （JavaScript Object Notation），我们
看一下其中的对象与数组的简写语法。

### 对象

1、对象是名字/值对的集合，名字与值用冒号分隔，每个名字/值对间用逗号分隔，然后整个对象用花括号包含。比如：

```javascript
var obj=
{
        data:"test",
        getData:function()
        {
                return this.data;
        }
}
```

这便定义了一个对象实例变量`obj`，这个对象有一个数据成员`data`，它的值是"test"，它还有一个函数成员`getData`,执行它返回数据成员的值。

### 数组

2、数组是值的集合，值之间用逗号分隔，然后整个数组用方括号包含。比如：

```javascript
var datas=[1,2,3,4];
```

这便定义了一个数组变量`datas`，它是由1，2，3，4四个整数构成的数组。

3、对象定义与数组定义中的值可以是javascript语言里的字符串、数字、对象（函数也是一种对象，所以也包括函数）、数组等，这样对象与数
组的定义可以是自嵌套的，比如可以这样写一个数组：

```javascript
var ds=[[11,12],[21,22]];
```

这是一个有两个元素的数组，每个元素又是一个有两个元素的数组。也可以定义一个嵌套的对象：

```javascript
var obj=
{
        obj1:
        {
                t11:1,
                t12:2
        }
        obj2:
        {
                t21:1,
                t22:2
        }
}
```

这样定义的对象实例`obj`有两个数据成员`obj1`、`obj2`，这两个数据成员分别是两个对象实例。

## javascript 调试

最后提一下javascript的调试，VS.NET是个不错的调试器，不过因为我们的脚本是嵌入在ollydbg里面执行的，没办法直接用vs.net装入，所以只
能用attach方式来调试，ms为它的javascript实现加了一个语句:`debugger`，相当于win32程序里的API `DebugBreak`，我们在脚本里要设断的地方写
上

```javascript
debugger;
```

然后运行，就会中断并提示选择调试器，然后在vs.net里就可以单步跟踪每条语句、查看各个变量的值、执行堆栈等等了。

--------------------------------------------------------------------------------

                                                       2006年09月05日 10:01:39

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
