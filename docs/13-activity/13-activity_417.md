

# 已经过 5 月 03 日 23 点了，我来吐槽一下

> 原文：[`www.pediy.com/kssd/pediy12/133333.html`](https://www.pediy.com/kssd/pediy12/133333.html)

对 Kernel Exploit Me 感兴趣  
然后习惯性地把 exploit1.sys 丢到 IDA 里面  
然后习惯性地在 DriverEntry 函数里面找 70h  
然后根据 Irp 确定出 IO\_STACK\_LOCATION、UserBuffer、IoControlCode  
然后看到一个不认识的函数：VidSetTextColor  
然后在 MSDN 里面搜一下，无果  
然后习惯性在百度里面搜一下  
然后……  
就没然后了  
  
不知出题者是考察分析、解决问题的能力，还是考察用搜索引擎的能力。。。![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")

引用: *最初由 **zhuliang**发布* ![查看帖子](img/showthread.php?p=954695#post954695) 泉哥，能否行知是哪个漏洞吗？我只分析过第一个，第二个没分析过。 

第二个漏洞原理类似 EVEREST (kerneld.sys)漏洞：  
http://www.ntinternals.org/ntiadv0803/ntiadv0803.html