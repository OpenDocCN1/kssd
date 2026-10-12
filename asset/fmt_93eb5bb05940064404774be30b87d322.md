
# Pwn2Own-2010-Windows7-InternetExplorer8

> 原文：[`www.pediy.com/kssd/pediy12/109669.html`](https://www.pediy.com/kssd/pediy12/109669.html)

标 题: 【翻译】Pwn2Own-2010-Windows7-InternetExplorer8  
作 者: espzj  
时 间: 2010-03-27,20:10:00  
链 接: http://bbs.pediy.com/showthread.php?t=109669  

附件包含原文和我的翻译，翻译的不足之处，希望下载者见谅

“在本届 Pwn2Own 黑客大赛上，荷兰独立黑客彼得洛格丹希尔(Peter Vreugdenhil)也成功利用 IE8 的多个漏洞攻破了 64 位 Winows 7 操作系统，他因此获得了 1 万美元的奖金和一部 Windows 电脑。微软 IE 团队全程见证了洛格丹希尔的攻击过程。微软发言人表示，目前尚未获得细节信息，但等到大赛组委会搜集了攻击信息后，他们会尽快做出响应。” http://tech.sina.com.cn/t/2010-03-25/09523984722.shtml  --引用自 tombkeeper 的和谐 Blog

之前我翻译的一篇文章“JIT\_BHDC-2010-Paper”（参见：http://bbs.pediy.com/showthread.php?t=109060）介绍了如何通过 JIT 在 Windows Vista 上绕过 DEP 和 ASLR，具体细节请参看相应的附件。本文也介绍了如何在 Windows 7 下绕过 DEP 和 ASLR，我个人认为还是很有参考意义的。

"ASLR 通过随时加载关键存储区的位置，使黑客难以预测他们的攻击代码是否能够运行。针对这个防护措施，维莱格登希尔首先利用大量的弱点获得了 IE8 加载到存储器当中的一个.dll 模块的基本地址，然后再利用这个模块去攻击 DEP 防护程序。"

## 参考链接

1：Hacker exploits IE8 on Windows 7 to win Pwn2Own，http://blogs.zdnet.com/security/?p=5855  
2：Pwn2Own 2010，http://dvlabs.tippingpoint.com/blog/2010/02/15/pwn2own-2010  
3：Win7 + IE8 exploited, again，tombkeeper 的和谐 Blog，http://hi.baidu.com/tombkeeper/blog/item/63bd3ef5f71e8629bd3109e4.html

## 上传的附件

| ![](img/aabdec8de01ec04d2bc68959c61abb93.jpg) || [Pwn2Own-2010-Windows7-InternetExplorer8.pdf](http://bbs.pediy.com/showthread.php?t=109669) |
| ![](img/b7e61e3627e1e92d7b948dd8fdd24fa6.jpg) || Pwn2Own-2010-Windows7-InternetExplorer8-chs.doc |
