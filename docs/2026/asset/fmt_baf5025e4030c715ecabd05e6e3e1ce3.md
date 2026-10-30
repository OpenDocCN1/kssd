
# CVE-2019-1208漏洞调试笔记

目录

- [CVE-2019-1208漏洞调试笔记](https://bbs.kanxue.com/#cve-2019-1208漏洞调试笔记)
- [前言](https://bbs.kanxue.com/#前言)
- [漏洞成因](https://bbs.kanxue.com/#漏洞成因)
- [6月补丁分析](https://bbs.kanxue.com/#6月补丁分析)
- [PoC分析](https://bbs.kanxue.com/#poc分析)
- [修复方案](https://bbs.kanxue.com/#修复方案)
- [利用编写](https://bbs.kanxue.com/#利用编写)
- [伪造超长数组](https://bbs.kanxue.com/#伪造超长数组)
- [任意地址读取](https://bbs.kanxue.com/#任意地址读取)
- Bypass ASLR
- [虚函数劫持](https://bbs.kanxue.com/#虚函数劫持)
- [利用约束](https://bbs.kanxue.com/#利用约束)
- [代码执行](https://bbs.kanxue.com/#代码执行)
- [参考资料](https://bbs.kanxue.com/#参考资料)

## 前言

这篇文章之前已经在其他地方发过，这里再发一个格式更友好一点的版本，便于查阅。一并修改了之前的几处笔误。由于国家相关征求意见稿的要求，此类分析文章后面就不太方便公开发布了。

`CVE-2019-1208`是趋势科技的`@elli0tn0phacker`在今年`6`月发现的一个`vbscript`漏洞，[报告](https://bbs.kanxue.com/elink@3abK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4#2L8h3g2F1N6s2y4Q4x3X3g2@1M7X3g2F1k6r3#2A6j5%4u0G2i4K6u0W2j5$3!0E0i4K6u0r3j5i4y4K6k6i4c8K6i4K6u0r3g2r3g2U0K9q4)9J5k6p5u0J5K9h3g2X3i4K6u0V1b7g2)9J5k6q4m8J5L8$3!0X3i4K6u0V1L8$3k6Q4x3X3c8o6L8$3&6U0k6i4m8@1i4K6u0V1c8i4S2H3L8r3!0A6N6r3W2F1k6#2)9J5k6p5y4h3c8g2)9J5k6o6t1H3x3e0W2Q4x3X3b7I4x3U0l9^5i4K6u0V1K9h3&6Q4x3X3c8u0L8Y4c8W2M7X3&6W2N6q4)9J5k6p5g2^5M7r3I4G2M7X3g2J5i4K6u0W2M7r3c8X3)中提到这个漏洞是通过补丁比对发现的，这引起了笔者的兴趣。最近，笔者花了一些时间对该漏洞进行了比较详细的研究。在这篇文章中，笔者将从漏洞成因、修复方案、利用编写三个方面对该漏洞进行介绍。

读者将会看到，代码开发者是如何在修复旧漏洞时不经意间引入新漏洞。在这个例子中，引入的还是一个非常严重的远程代码执行漏洞。通过这个例子读者也会发现，有时候通过补丁比对就可以发现新漏洞。

该漏洞从`2019`年`6`月更新被引入，到`2019`年`9`月更新被修复，只存活了短短`3`个月，因此编写这个漏洞的利用并无价值，笔者写这个漏洞的利用只是为了概念验证。

尽管微软已经在`2019年8`月的`IE`更新中全面禁用了`vbscript`，但出于安全性考虑，完整利用代码不予公开。

## 漏洞成因

这是一个`vbscript`的`UAF`(`Use After Free`)漏洞，漏洞成因还要从微软今年`6`月的补丁说起。

### 6月补丁分析

微软在`2019`年`6`月的`vbscript`更新中引入了下面几个函数：

- `SafeArrayAddRef`
- `SafeArrayReleaseData`
- `SafeArrayReleaseDescriptor`

引入`SafeArrayAddRef`的作用是为`SafeArray`提供一种类似引用计数的机制。

源码中通过使用`STL`的 `map`将一些对象/数据指针(如`pSafeArray`和`pvData`)与一个`int`型的计数器进行绑定。

在`VbsFilter`和`VbsJoin`这两个函数中，在调用实际的`rtJoin`和`rtFilter`前，会调用`SafeArrayAddRef`对相关指针的引用计数`+1`。调用完毕后，再调用`SafeArrayReleaseData`和`SafeArrayReleaseDescriptor`在`map`中将指针对应的计数`-1`，并将指针所对应的`key`从`map`中删除。

开发者应该是用这种方式修复了一些`UAF`问题。但修复方案中没有考虑到当`Join/Filter`传入的数组中有类对象时，在`Public Default Property Get`这一潜在的回调中可以对数组进行操作(比如`ReDim`)。这样，当调用完 `rtJoin/rtFilter`后返回`VbsJoin/VbsFilter`时，对应的`pSafeArray/pvData`指针已被更新，原先的设计是将之前已在`map`中“注册”的指针传入后续的`SafeArrayReleaseData/SafeArrayReleaseDescriptor`进行引用计数减操作，但现在传入`SafeArrayReleaseData/SafeArrayReleaseDescriptor`的指针均不在`map`中(因为被重新创建了)。这导致在调用`RefCountMap<void *>::Decrement`函数时，`find`方法找不到对应的`key`，函数直接返回`0`。这个返回结果被`SafeArrayReleaseData/SafeArrayReleaseDescriptor`理解为对应的指针的引用计数为`0`，从而将对应的`SafeArray`对象和数据销毁。

具体地，开发者借助`RefCountMap`类实现了一个“伪引用计数机制”，通过一个`map<tagSAFEARRAY *,int>`将所关心的`SafeArray`指针与一个`int`型计数器绑定起来，计数值只有`0、1、不存在`，三种情况。

相关操作函数的声明如下：

![](https://bbs.kanxue.com/upload/attach/201911/628056_3BS99YER4WQTVRY.png)

`RefCountMap<void *>::Decrement`函数的伪代码如下：

![](https://bbs.kanxue.com/upload/attach/201911/628056_EUSB54A4AB76BVV.png)

了解了这些知识后，回过头去理解`@elli0tn0phacker`报告中的`Figure 5`就会容易多了。

### PoC分析

`@elli0tn0phacker`给出的`poc`大致如下：

![](https://bbs.kanxue.com/upload/attach/201911/628056_DXRWYKV37RAR97T.png)

由于漏洞的存在，我们知道`arr(0) = 1`语句执行前`arr`已被释放，而且从代码中可以看到`arr`是在回调中被`ReDim`的。那么`arr`到底存在哪里？为什么`arr(0) = 1`索引的是`ReDim`后被释放的`SafeArray`，而不是`Redim`前的`SafeArray`？

这就涉及到 `vbscript`虚拟机的相关知识。

卡巴斯基实验室的`Boris Larin`曾写过一篇关于`vbscript`虚拟机的[文章](https://bbs.kanxue.com/elink@c60K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3y4#2M7X3g2D9K9i4y4@1i4K6u0W2j5$3!0E0i4K6u0r3k6r3g2D9N6X3W2F1k6#2)9J5k6r3c8W2k6i4m8Q4x3X3c8A6L8Y4c8G2i4K6u0V1N6X3u0K6j5%4u0A6M7s2c8Q4x3X3c8S2L8X3q4D9P5i4y4A6M7#2)9J5k6r3!0X3i4K6u0V1j5%4k6W2i4K6u0V1x3U0l9I4z5q4)9J5k6o6R3I4y4K6c8Q4x3X3c8W2P5s2m8D9L8$3W2@1j5i4c8A6L8$3&6Q4x3V1j5^5y4U0x3K6x3#2)9J5c8R3`.`.)，并且开源了相关的[调试插件](https://bbs.kanxue.com/elink@a72K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6w2j5i4y4H3k6i4u0K6K9%4W2x3j5h3u0Q4x3V1k6h3b7Y4y4U0M7X3W2H3N6p5W2F1N6r3g2J5L8X3q4D9M7H3`.`.)。

在文章中，作者对`vbscript`虚拟机进行了比较细致的介绍。`vbscript`的所有代码都会先被编译为`P-Code`，随后通过`CScriptRuntime::RunNoEH`对所有`P-Code`进行解释执行，`CScriptRuntime`对象的成员变量中存储着解释所需的许多信息，比较重要的几个如下：

![](https://bbs.kanxue.com/upload/attach/201911/628056_WEQYBTQDPKV4XUE.png)

借助调试插件，我们可以得到 `PoC`代码编译后的`P-Code`：

![](https://bbs.kanxue.com/upload/attach/201911/628056_TJRBZ8V6V2RE5NM.png)

以下是上述用到的部分指令对应的字节码(全部指令请参考`Boris`的插件源码)：

![](https://bbs.kanxue.com/upload/attach/201911/628056_NEXGSPU7AU7YVUC.png)

从`P-Code`中可以看出， `arr(0) = 1`这句对应的指令索引的是本地变量栈(`OP_CallLclSt, 0x2E)`，`Call Join(arr)`这句对应的指令索引的也是本地变量栈(`OP_LocalAdr, 0x19`)，从两个指令名称中我们可以猜测`arr`被存储在本地变量栈上。

在`IDA Pro`中对`vbscript!CScriptRuntime::RunNoEH`进行逆向，我们来看一下上述两个指令解释分支的汇编代码：

![](https://bbs.kanxue.com/upload/attach/201911/628056_QQB6M2ENYZ6BDWJ.png)

![](https://bbs.kanxue.com/upload/attach/201911/628056_GE28FECD99BBAVA.png)

上述两个分支都调用了`CScriptRuntime::PvarLocal`方法，再来看一下`CScriptRuntime::PvarLocal`方法的实现：

![](https://bbs.kanxue.com/upload/attach/201911/628056_Q6XFF4SE8FQC9FF.png)

可以看到`CScriptRuntime::PvarLocal`接收一个索引，并且基于`CScriptRuntime`对象`+0x28`或`0x2C`处的值进行偏移运算。调试时发现`PoC`两处对`arr`的操作索引均为`1`，所以存储`arr`的地址为：

![](https://bbs.kanxue.com/upload/attach/201911/628056_MWPN9KHJB6MWGR2.png)

上述分析验证了上面对于指令作用的猜想，`PoC`中每次使用`arr`变量时，都会传入对应的索引去本地变量栈中进行访问。

明白了`arr`的存取原理后，我们可以清晰地在调试器中观察`arr`的变化过程，从而理解整个`UAF`的过程。

笔者在开启页堆后对`PoC`进行了调试。我们先将断点下到`OP_LocalAdr`指令的解释分支，可以看到`Join(arr)`执行时访问到的`arr`，命中断点时`ebx`即为`CScriptRuntime`，调试时`arr`从本地变量栈(`ebx+0x28`)进行索引，读者请留意下图中蓝色高亮的指针，`ReDim`语句执行后它会发生变化。

![](https://bbs.kanxue.com/upload/attach/201911/628056_3XUWSDNF4WF5TX7.png)

我们对上图中高亮数据(`SafeArray`指针)所在的内存下一个写入断点，观察这个位置上数据的几次变化过程。

第一次是在`ReDim`(`OP_ArrNamReDim`)执行时，对之前`arr`的清理阶段(`OP_ArrNamReDim`指令的解释流程在后面“修复方案”一节中会进一步说明。

## 利用编写

### 利用约束

出于利用构造的约束条件，命令行参数的前`4`个字符是由前面伪造的虚表的地址解释而来，这种情况下很容易造成前`4`个字符里面有多余字符，因此`WinExec`也就不能按预期执行后续的命令行。笔者一开始想到的将虚表伪造到`0x20202020`这个地址，这样命令行参数的前`4`个字符可以被解释为空格，不会影响整个命令行的解释。但该漏洞中对指定地址的连续写是受限的，笔者最终放弃了这个思路。

后来笔者将未加修改的`exploit`在`win10`环境试了一下，发现计算器可以成功弹出，以下为某次在`win10`下调试得到的参数及伪造的虚函数表：

![](upload/attach/201911/628056\_YHMD3PYKTNKKD8T.png)

笔者推测`win10`和`win7`下进程创建相关函数对命令行参数的处理存在一些差异，`win10`上的容错性更高一点。

### 代码执行

最终，笔者成功在`windows 10 1709 x86`系统的`2019`年`8`月全补丁环境上弹出一个计算器：

![](upload/attach/201911/628056\_ER3SJJVFTCH9UAK.gif)

## 参考资料

<a href="elink@739K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3y4#2M7X3g2D9K9i4y4@1i4K6u0W2j5$3!0E0i4K6u0r3k6r3g2D9N6X3W2F1k6#2)9J5k6r3c8W2k6i4m8Q4x3X3c8A6L8Y4c8G2i4K6u0V1N6X3u0K6j5%4u0A6M7s2c8Q4x3X3c8S2L8X3q4D9P5i4y4A6M7#2)9J5k6r3!0X3i4K6u0V1j5%4k6W2i4K6u0V1x3U0l9I4z5q4)9J5k6o6R3I4y4K6c8Q4x3X3c8W2P5s2m8D9L8$3W2@1j5i4c8A6L8$3&6Q4x3V1j5^5y4U0x3K6x3#2)9J5c8R3`.`.">《Delving deep into VBScript》</a>

<a href="elink@1abK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2N6s2u0W2L8X3c8E0K9h3y4J5L8#2)9J5k6h3y4G2L8g2)9J5c8Y4c8J5k6h3&6V1L8r3q4T1M7#2)9J5k6s2y4W2j5%4g2J5K9i4c8&6i4K6u0V1K9h3&6@1k6h3I4D9K9h3N6W2L8X3y4W2i4K6u0r3k6Y4u0G2L8g2)9J5k6r3u0A6L8X3c8A6k6X3k6Q4x3X3c8@1L8#2)9J5k6s2A6W2M7X3!0Q4x3X3c8V1j5i4W2Q4x3X3c8S2i4K6u0V1M7s2u0G2L8$3k6Q4x3X3c8G2k6W2)9J5k6r3y4G2L8X3y4W2M7s2c8Q4x3X3c8W2P5s2m8D9L8$3W2@1K9h3&6Y4i4K6u0V1j5%4k6W2i4K6u0V1x3U0l9I4z5g2)9J5k6o6p5J5x3o6S2Q4x3X3c8A6L8W2)9J5k6r3W2F1N6r3g2J5L8X3g2@1i4K6u0V1k6i4S2H3L8r3!0J5k6i4u0Q4x3V1j5`.">《From BinDiff to Zero-Day: A Proof of Concept Exploiting CVE-2019-1208 in Internet Explorer》</a>

<a href="elink@62dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2\*7k6i4u0G2k6r3q4&6K9h3&6A6N6r3W2S2N6r3W2$3k6g2)9J5k6h3y4G2L8g2)9J5c8X3u0D9L8$3N6Q4x3V1j5J5x3o6p5&6i4K6u0r3y4g2)9J5c8U0t1I4i4K6u0r3M7X3y4W2i4K6u0V1N6$3W2@1K9r3!0#2N6q4)9J5k6r3&6S2N6r3W2$3k6g2)9J5k6r3y4G2k6r3g2Q4x3X3c8W2P5s2m8D9L8$3W2@1j5i4c8A6L8$3&6Q4x3X3c8G2k6W2)9J5k6r3q4Q4x3X3c8%4M7X3W2@1k6g2)9J5k6s2N6Z5j5i4c8Q4x3X3c8%4K9r3g2J5k6g2)9J5k6r3W2F1i4K6u0V1K9h3&6@1k6i4u0F1k6i4c8Q4x3X3c8W2P5s2m8D9L8%4u0W2M7R3`.`.">《RCE WITHOUT NATIVE CODE: EXPLOITATION OF A WRITE-WHAT-WHERE IN INTERNET EXPLORER》</a>
