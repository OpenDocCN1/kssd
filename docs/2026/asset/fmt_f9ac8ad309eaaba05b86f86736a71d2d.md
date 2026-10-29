
# [原创]跟着crownless学Web之（1）神奇的计算器

本文备份于<a href="elink@fb4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6\*7K9r3!0#2P5r3W2F1j5h3&6Q4x3X3g2Y4K9i4c8Z5N6h3u0Q4x3X3g2A6L8H3`.`.">我的博客</a>

大家好，我是Web安全板块的新版主crownless。从今天起我将以CTF赛题讲解的形式介绍一系列的Web基础知识，讲解的顺序将是循序渐进，因此不需要读者有任何基础知识。希望能吸引更多人关注看雪的Web安全板块，为板块增加人气和活力。

在这篇文章中，你将学到：

1. 一点HTML语言
2. 什么是前端和后端
3. HTTP GET请求
4. 一点php语言
5. 基础的shell命令

话不多说，让我们开始吧。首先打开这次的CTF赛题网站：<a href="elink@205K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5K6z5g2)9J5k6e0t1J5y4q4)9J5k6e0t1J5x3q4)9J5k6e0j5%4i4K6y4m8x3K6l9H3x3o6f1`."><mark class="encrypted">4c2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5K6z5g2)9J5k6e0t1J5y4q4)9J5k6e0t1J5x3q4)9J5k6e0j5%4i4K6y4m8x3K6l9H3x3o6f1`.</mark></a>

你将看到如下界面：

![](upload/attach/201901/833800\_DYBHWWKVS2CPJE2.png)

对于刚接触Web安全甚至是Web编程的你来说，这个界面或许让你有些困惑。不要紧，请耐心仔细地阅读我的讲解，当你读完这篇文章后一定能对Web安全有一个基本的了解。

首先，页面上方居中部分是一个大标题`calc`，接着有一个输入框（左边有`input:`文本）、一个`Submit`按钮、以及`result:`文本。

这个界面和我们平时在各种网站上见到的界面很像，可以说是最简单的网页界面之一。那么，这个界面是如何实现的呢？事实上，这是用HTML编写的。HTML即HyperText Markup Language，中文翻译是“超文本标记语言”。接下来，我就教你怎么编写如上的界面。何谓编写界面？就是通过编写HTML语言源代码来达到我们想要的界面效果。可能细心的你已经发现，网页下方已经有了一堆源代码。正如你猜想的那样，这是我为了方便你的学习而特意将源代码显示在了这里。但是不要急着把这些代码直接拷贝出来，因为里面内嵌了一些php语言的代码。我们要做的是删除彩色的php代码。那么，我们可以得到如下代码：

```
<center>
<h2>calc</h2>
<form action="./index.php">
input: <input type="text" name="calc" value="">
<input type="submit" value="Submit">
</form>
</center>
```

将其保存为`1.html`，并用浏览器打开`1.html`，你会发现除了最后一行的`result:`文本消失了之外，其他部分和CTF赛题网站上的界面是相同的。`result:`文本之所以消失，是因为我们删除了包含`result:`的彩色的php代码。恭喜你，你已经完成了人生中的第一张网页！但是别高兴得太早，接下来我来讲解一下`1.html`的结构。

HTML语言的特色是拥有数量众多的“标签”。何谓“标签”？暂时你可以理解为类似于`<aaa>bbb</aaa>`这样的东西。标签的功能多种多样。以`1.html`为例，`<center>`标签可以让其包裹的内容在网页中居中显示。`<h2>`标签定义了一个大标题。`<form>`标签定义了供用户输入的HTML表单（表单的作用在后文中会讲解）。`<input>`标签定义了输入控件（`type="text"`代表输入框，`type="submit"`代表提交表单的按钮）。好了，现在你已经知道怎么用HTML编写一个简单的网页了，但是你想问：在`1.html`中，这个`Submit`按钮点上去似乎没什么用嘛！不像在CTF赛题网站上，在文本框里输入`1 + 2`，result就会显示为`3`。（事实上，这就是为什么这道题叫做“神奇的计算器”。）

这里，就涉及到了前端和后端的概念。事实上，你刚才编写的`1.html`正是前端代码。一般流行的前端技术有HTML、CSS、JavaScript。HTML用来构建网页的基本框架，CSS用于美化网页，JavaScript用于为静态的HTML网页增加动态的交互行为。前端代码是运行在浏览器上的。比如说，你打开`1.html`时，浏览器会读取`1.html`的内容，并将其转化（行话叫“渲染”）成图像显示在显示器上。而后端代码是运行在服务器上的。比如你刚才在文本框里输入`1 + 2`并点击`Submit`，即是将`1 + 2`这个字符串传递到了`./index.php`（可以从`<form>`标签的`action`属性看出来）。`./index.php`即`<mark class="encrypted">706K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5K6z5g2)9J5k6e0t1J5y4q4)9J5k6e0t1J5x3q4)9J5k6e0j5%4i4K6y4m8x3K6l9H3x3o6g2Q4x3V1k6A6L8X3c8W2P5q4)9J5k6i4m8Z5M7q4)9$3x3q4!0q4y4#2)9&6b7g2)9^5y4q4!0q4y4#2!0m8c8g2)9^5x3q4!0q4y4g2)9^5y4W2)9&6z5g2!0q4x3#2)9^5x3q4)9^5x3W2!0q4y4W2)9&6b7#2)9^5c8q4!0q4y4g2)9^5b7g2!0m8x3g2!0q4y4g2)9&6z5g2!0m8z5q4!0q4y4q4!0n7z5q4)9^5b7g2!0q4z5q4!0n7c8W2)9&6x3q4!0q4z5q4!0m8x3g2)9^5b7#2!0q4y4#2)9&6b7g2)9^5y4s2m8Z5M7q4!0q4y4#2!0m8z5q4)9^5b7W2!0q4y4g2!0n7b7g2)9^5c8W2!0q4y4W2)9^5c8g2!0m8y4g2!0q4y4g2)9^5c8W2)9&6y4#2!0q4y4g2)9^5z5q4!0n7x3q4!0q4y4q4!0n7c8q4!0m8x3q4!0q4y4#2)9&6b7g2)9^5y4q4)9$3x3o6p5`.</mark> + 2`字符串，调用`shell_exec`函数获得执行结果`3`，再嵌入网页形成“动态网页”发送回你的浏览器，你的浏览器渲染“动态网页”显示在屏幕上，这样你就看到了`3`。综上所述，我们称`./index.php`为一个后端，因为它能根据用户在前端的输入生成“动态网页”。

那么用户的输入是怎么从浏览器发送到后端的呢？事实上，这里使用的是HTTP协议，全称HyperText Transfer Protocol（超文本传输协议）。当你点击`Submit`按钮时，浏览器就会找到包含这个按钮的表单元素，即`<form action="./index.php">`，并找到这个表单元素的`method`属性（如果没有，那么默认为GET）。如果`method`属性是GET，那么就发送一个HTTP GET请求，其中包含着浏览器传给后端的参数。比如，当我们在输入框里输入`1+2`并点击`Submit`时，将会发送以下HTTP请求给后端：

![](upload/attach/201901/833800\_W5NRJ9YKZ6FYR2R.png)

可以看到，`calc`参数的内容是`1%2B2`，事实上这是对`1+2`的URLEncode（URL编码）。为什么是通过`calc`参数传递`1%2B2`呢？事实上，这是在输入框标签`<input type="text" name="calc" value="">`的`name`属性约定的。也就是说，在这个输入框中输入的内容都将被作为`calc`参数发送给后端。

当`index.php`后端接受到任何HTTP请求后，都会顺序执行`index.php`这个脚本内的任何内容。如果遇到了HTML代码，那么原封不动地将其返回给浏览器。如果遇到了php代码，那么就执行php代码并将结果返回给浏览器。那么后端是怎么从HTTP GET请求中取得这个参数的呢？接下来，让我们分析一下php代码。

```
<?php
$str="";
if(!empty($_GET)){
$str=$_GET["calc"];
}
?>
```

这段代码的意思是，首先将`str`变量赋值为空字符串，然后检查`$_GET`变量是否为空（`empty`函数的文档<a href="elink@328K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4m8Z5M7q4)9J5k6h3&6W2N6q4)9J5c8X3#2S2L8Y4g2S2L8q4)9J5c8Y4A6Z5i4K6u0r3k6Y4g2F1j5%4c8A6L8$3&6Q4x3X3g2W2L8i4m8@1P5g2)9J5k6i4m8Z5M7l9`.`.">在这</a>），如果不为空，那么将`calc`参数的内容赋值给`str`。事实上，`$_GET`是一个数组变量，`$_GET["calc"]`中存放着从HTTP GET请求中获得的`calc`参数。

接着再看如下php代码：

```
<?php echo $str;?>
```

`echo`的作用是将`str`变量写入输出。也就是说，你发送给后端的`1+2`参数会被后端原封不动地嵌入输入框标签的`value`属性。这样你点击`Submit`后，输入框里就会出现`1+2`，而不是空的输入框。

接着再看最后一段php代码：

```
<?php
echo "result:".shell_exec("echo $str | bc");
?>
```

首先，输出一个`result:`，然后这里使用了`.`运算符。在php中，这代表字符串连接。然后，执行<a href="elink@862K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8Z5M7q4)9J5k6h3&6W2N6q4)9J5c8X3#2S2L8Y4g2S2L8q4)9J5c8Y4A6Z5i4K6u0r3k6Y4g2F1j5%4c8A6L8$3&6Q4x3X3g2K6K9r3g2D9L8q4)9J5k6r3g2^5k6h3y4Q4x3X3g2H3K9s2l9`.">`shell_exec`</a>函数。这个函数的作用是“通过shell环境执行命令，并且将完整的输出以字符串的方式返回”。那么执行的是什么命令呢？很明显，是`echo $str | bc`。`echo`在shell中的作用和在php中的作用一致，比如你在shell中执行`echo crownless`，你会得到`crownless`。而`bc`是一个简易的计算器程序。在你的shell中试试看执行`bc`指令，再输入`1+2`并按回车，你会得到`3`。在`echo`和`bc`当中有一个`|`符号，这是shell中的管道符号。使用管道，就相当于将前一个程序的输出传递给下一个程序。在你的shell中试试看`echo 1+2 | bc`，你将会得到`3`。

这里再介绍两个常用的shell命令：

1. `ls`可以列出当前目录下的所有文件。
2. `cat`可以打印一个文件的所有内容。

好了，聪明的你一定想到了我们可以通过控制`str`参数来实现任意命令执行。首先，我们要尝试闭合`echo`语句。那么我们就要用到`;`符号。好，现在我们可以给`str`传入`;`，那么将会执行shell命令`echo ; | bc`。但是我们还要想个办法让`;`后的内容失效。这里我们就要使用shell下的单行注释符号`#`。也就是说，`echo ; # | bc`的作用相当于`echo ; `。那么现在我们就可以在`;`和`#`之间插入任意我们想要注入的命令了，如`ls;`。试试看在输入框中输入`; ls; #`，并按`Submit`，我们发现目录下有一个`there_1s_4_fl4g`文件。我们再输入`; cat there_1s_4_fl4g; #`，我们就可以得到这道CTF题的flag。

怎么样，通过一道题，我们学会了Web的基础知识，很有成就感吧？敬请期待crownless的后续作品。

本文备份于[我的博客](https://bbs.kanxue.com/elink@b40K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6*7K9r3!0#2P5r3W2F1j5h3&6Q4x3X3g2Y4K9i4c8Z5N6h3u0Q4x3X3g2A6L8H3`.`.`)
