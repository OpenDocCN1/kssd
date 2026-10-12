
# 《加密与解密(第三版)》学习指引(第十三章)脱壳技术

> 原文：[`www.pediy.com/kssd/pediy10/72717.html`](https://www.pediy.com/kssd/pediy10/72717.html)

两个公司，两个每天工作就是脱壳的人，在看到我在分析壳的时候都说过类似的话。这段代码没必要看，可以直接跳到后面分析。我想，如果我脱壳的时候总是能得到他们的指点，那当然很好了。正如大家看到的脱文，用的脱壳机或者脱壳脚本，能得到帮助总是好的，不能的时候呢？大牛们的话自然没错，但是对于我们初学者呢？这里有一个盲点，大牛都是在不断锻炼中，总结出自己的脱壳方法，当他们脱不了的时候，他们的经验会告诉他们该回头看看哪里些地方，而初学者却没有经验。
人容易犯这些错误
引用: 1. 过去懂，现在却不懂了。尤其是从加密解密 3 开始接触 OD 的人，从第二章介绍 OD 到开始学习第十三章，你能记得多少？
2. 明明不懂，却以为自己懂了。尤其是象我这类型通过阅读大量文章学 OD 的人，好像看得很多，很懂，事实却是常用的功能大家都反复讲，而你遇到的问题却是因为那个不常用的功能。
无论如何，在学习这章之前，先看看第二章，学习问题会少很多的

## 那些分析未知壳需要牢记的章节

引用: 1. `13.1.1 壳的加载过程`，让你不至于在跟踪未知壳时迷失方向的路标。
2. `13.3 抓取内存映像`，书中篇幅有限，你可能需要更多的扩展阅读

浅谈脱壳中的 Dump 技术 作 者: Lenus

[`bbs.pediy.com/showthread.php?t=17624`](http://bbs.pediy.com/showthread.php?t=17624)

Anti-Anti-Dump and nonintrusive tracers 翻译：kkbing aalloverred

[`bbs.pediy.com/showthread.php?t=25001`](http://bbs.pediy.com/showthread.php?t=25001)

[`bbs.pediy.com/showthread.php?t=25564`](http://bbs.pediy.com/showthread.php?t=25564)

【翻译】现代 dump 技术及保护措施 作 者: libradohko

[`bbs.pediy.com/showthread.php?t=38785`](http://bbs.pediy.com/showthread.php?t=38785)

3. `13.4 重建输入表`，依然是篇幅有限，由于更灵活更多样化，也很少有人总结。
4. `13.5.4 构造重定位表`，脱 DLL 的时候，这个是必须的。不知道是可以发挥创意的地方太少，还是没多少人对脱 DLL 有兴趣，相关文章十分少

下面是一些经验性的章节，我的建议是随便看看就好了。真遇上相同的情况再对着书来看。这也是我对待脱文的态度
引用: 1. `13.2 寻找 OEP`，相信大家都有过 ESP 定律脱不了壳的经验，这种技巧其实只适合收藏，一旦公布，很快便被壳作者跟进了。如果你有兴趣知道更多，论坛也有几篇文章。
attach 法脱壳 作者:kongfoo

[`www.pediy.com/bbshtml/BBS6/pediy6682.htm`](http://www.pediy.com/bbshtml/BBS6/pediy6682.htm)

寻找真正的入口（OEP）--广义 ESP 定律 作者：Lenus

[`www.pediy.com/bbshtml/bbs6/pediy6083.htm`](http://www.pediy.com/bbshtml/bbs6/pediy6083.htm)

一些常见语言所编写的程序的 OEP 特征 作 者: DimRacker

[`bbs.pediy.com/showthread.php?t=38612`](http://bbs.pediy.com/showthread.php?t=38612)

重建 stolen bytes 的一般方法 作者:blowfish

[`www.pediy.com/bbshtml/BBS6/pediy6840.htm`](http://www.pediy.com/bbshtml/BBS6/pediy6840.htm)

\[翻译\]利用代码注入脱壳 作 者: springkang\[DFCG

[`bbs.pediy.com/showthread.php?t=4541`](http://bbs.pediy.com/showthread.php?t=4541)

2. `13.8 压缩壳`，你也可以理解为已公布的快速脱壳方法依然能脱的壳。
3. `13.9 加密壳`，不懂的人似乎都认为高手可以凭空变出脱文、脱壳机和脱壳脚本。总是说帮忙看看，帮忙分析，指点指教什么的，事实上正因为有了加密壳的存在，逆向才变成真正的体力活，少则数小时，多起来多少天也有可能。为了简化分析工作，你需要掌握 OllyScript
调试好帮手 ODbgScript 入门系列 作 者: hnhuqiong

[`bbs.pediy.com/showthread.php?t=26029`](http://bbs.pediy.com/showthread.php?t=26029)

[`bbs.pediy.com/showthread.php?t=26053`](http://bbs.pediy.com/showthread.php?t=26053)

[`bbs.pediy.com/showthread.php?t=26130`](http://bbs.pediy.com/showthread.php?t=26130)

[`bbs.pediy.com/showthread.php?t=29572`](http://bbs.pediy.com/showthread.php?t=29572)

OllyScript v0.92 中文帮助文档 翻译：看雪 TT 小组

[`bbs.pediy.com/showthread.php?t=3781`](http://bbs.pediy.com/showthread.php?t=3781)
