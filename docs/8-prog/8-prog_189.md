
# Signatures for Library Functions in Executable Files

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-722.htm`](https://www.pediy.com/kssd/pediy09/pediy09-722.htm)

> 使用签名识别库函数确实非常有助于`dcc`工作。大大提高了输出文件的清晰度，输出尺寸得以减少，不用花时间去分析和反编译库例程。而且，原程序中出现的那些非通常的或者复杂困难的结构大都可能是其本身的一部分，而不是属于某些低级汇编库函数的。所以，在这些情况下可能要使用交互式反汇编器分析它们，以便高级用户在必要时介入分析。

> 绝大多数库函数得到识别，尽管不是所有的库函数，但对于`dcc`的目的来讲这就够了。它是个一般方法，适用于多种语言、编译器厂商和存储器模型，而且完全自动化。少数没有得到识别库函数需要用户交互帮助，这与`dcc`的完全自动化不可兼得。它能够从适当的库文件快速地生成签名文件，而且这也是完全自动的。签名文件在磁盘上的大小平均仅仅在`50k`左右。

> 有关`dcc`资料下载[`www.itee.uq.edu.au/~cristina/dcc.html`](http://www.itee.uq.edu.au/~cristina/dcc.html)

> 本文主题内容见`dcc`源码的`makedsig`子目录

`4dd466d0e1e0cc123afbe504da2462ca`
