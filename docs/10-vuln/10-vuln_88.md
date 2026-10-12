
# 纯字母 shellcode 揭秘

> 原文：[`www.pediy.com/kssd/pediy11/113177.html`](https://www.pediy.com/kssd/pediy11/113177.html)

![](img/110d920e4e703968a2a60c986d363163.jpg "尴尬")最近仙果兄弟有一篇 exploit 分析的文章，里面提及了“纯字母 shellcode”这个概念，回头翻翻以前调试过的一个 exploit，里面就是用的这种 shellcode 编码技术，现在把它提出来跟大家交流交流。

现在很多 exploit 的中使用的 shellcode 看起来貌似离我们常见的常规汇编的机器码大相径庭，好像只是一段来来回回的有效字符串（如字母，数字等等），而且单单对这段字符串进行反汇编看到的东西往往让我们匪夷所思，这里我们就来看看这样的 shellcode 是怎么回事。

## 首先举个例子

前几天抓到的一个 exploit 样本里面，shellcode 的主体是这样的一段：

![](img/9d5cc567e70300b1fbc9aec4db035cdc.jpg)

这个看起来只是一段来来回回略有规律的字符串，但是从整个 exploit 来看这里确实是一段 shellcode 而且代码中是以 shellcode 为变量名来命名的，我们对它进行静态反汇编看看有没有什么发现：

![](img/50026024e66a8aa97e3aa9685812b8e3.jpg)

我们应该会觉得这个如果是 shellcode 的话，那也太神奇了吧！

难道是反汇编器欺骗了我们？

呵呵，其实应该是我们没有深入的理解吧，或许是忽略了 exploit 里面一些重要的东西？唯一的解释就是----我们看到的 shellcode 是被进行了特殊处理的 shellcode。

这个如果是 shellcode 主体的话，那只能说明在 exploit 中还是有另外一小段代码是对这段 shellcode 进行解码的。

再对 exploit 进行动态分析，我们察觉溢出之后跳到的一段代码并不是我们现在看到的纯字母 shellcode，而是下面这一段：

![](img/e9afff144df0e95716ba3d9a9eeb67e6.jpg)

呵呵，多了前面的 33 个字符：

![](img/d05e02d8bad1e339453ae03238155daf.jpg)

真相就在这里：

![](img/9387b667aafc3e7662fbb4f98a59484f.jpg)

看来确实是对下面的一段“纯字母 shellcode”进行解码的。

对应的编码代码，我们将它翻译成 C 代码如下：

```
for (int i = 0;i < codeLen; i++)
  {
    low_al = ((byte *)decodescEnd)[i] & 0x0f;
    hi_ah = (((byte *)decodescEnd)[i] & 0xf0) >> 4;
    if (low_al > 0x8)
    {
      newscBuff[decodeLen+2*i + 1] = 0x40 + low_al;
    }
    else
    {
      newscBuff[decodeLen+2*i + 1] = 0x60 + low_al;
    }
    if (hi_ah > 0x8)
    {
      newscBuff[decodeLen+2*i] = 0x60 + hi_ah;
    } 
    else
    {
      newscBuff[decodeLen+2*i] = 0x40 + hi_ah;
    }
  }
```

从这个代码我们可以对其进行解码后如下：

![](img/302702965a19a17c55612e451d585411.jpg)

反汇编如下：

![](img/706092bbabe9913b0fb147ae51e554e5.jpg)

这下似曾相识了吧!对了，这就是 shellcode 的本来面貌。

## 总结

其实所谓的纯字母 shellcode 只不过是对真正 shellcode 的一种巧妙的编码方式而已，从思路上讲，效果无异于异或之类的编码方式。作为一个 exploit 学习者来讲，真相往往就是置身在 exploit 其中，我们能不能看穿实际上就是看学习者本身的研究深入程度了。

本篇文章没有多大技术含量只是说明一些小问题，各位达人见笑了~~![](img/cb6288ad81b189b02921e98a92da6abe.jpg "124.gif")
