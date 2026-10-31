
# [原创]【勒索预警】放假之前，EvaRichter勒索病毒来袭

目前勒索病毒仍然是全球最大的威胁，最近一年针对企业的勒索病毒攻击越来越多，大部分勒索病毒是无法解密的，并且不断有新型的勒索病毒出现，各企业一定要保持高度的重视，马上放假了，一款新型勒索病毒来袭......

近日国外某独立恶意软件安全研究人员曝光了一个新型的勒索病毒，如下所示：

![](https://bbs.kanxue.com/upload/attach/201909/288797_PG959G6HNP5DD8F.png)

然后有人回复说是EvaRichter勒索病毒，并给出了id-ransomware博客地址，如下所示：

![](https://bbs.kanxue.com/upload/attach/201909/288797_XC75S9Q66WPBGQ5.png)

这款勒索病毒之所以勾起我的兴趣，主要是它与之前的GandCrab、Sodinokibi、GermanWiper、NEMTY等勒索有某些类似之处，都将桌面壁纸修改成蓝色了，

笔者通过MD5在`app.any.run`上找到了相应的样本，如下：

![](https://bbs.kanxue.com/upload/attach/201909/288797_TQYVT823GVYTZNF.png)

样本被人在24号上传到了VT上，依据上传的时间判断可能就是国外那个独立恶意软件研究员上传的，从`app.any.run`上下载样本，图标使用了类似微信的图标，如下所示：

![](https://bbs.kanxue.com/upload/attach/201909/288797_9J56ZN5YUD23P8U.png)

运行之后，加密的文件后缀为随机文件，如下所示：

![](https://bbs.kanxue.com/upload/attach/201909/288797_7H8CW9YCCPD299N.png)

勒索病毒会修改桌面壁纸为蓝色，如下所示：

![](https://bbs.kanxue.com/upload/attach/201909/288797_SVYHYK6WXUNN3PB.png)

勒索提示信息文本文件[加密后缀随机名] `_how_to_decrypt.txt`，内容如下所示：

![](https://bbs.kanxue.com/upload/attach/201909/288797_E5RJ9HGUDNQYFWM.png)

访问勒索病毒解密网址，如下所示：

![](https://bbs.kanxue.com/upload/attach/201909/288797_4XWDHCUC89MZJ5R.png)

输入勒索提示信息中的Access Code码之后，如下所示：

![](https://bbs.kanxue.com/upload/attach/201909/288797_QW2YBPARF9WXFK8.png)

黑客的BTC钱包地址：`14Biqrf2fryuGNDrMDPchPCQeEjzZkqaLi`，勒索赎金七天之内为：`0.1521163 BTC`，如下所示：

![](https://bbs.kanxue.com/upload/attach/201909/288797_EXSP9HBE56JS7CA.png)

最近BTC降了一点，我们按今天BTC的市价来算，如下：

![](https://bbs.kanxue.com/upload/attach/201909/288797_UWAF27WSM4SK72Z.png)

等于`0.1521163 X 8405.59 = 1278美元` (七天之内解密的价格)

此勒索病毒同样采用了高强度的代码混淆技术，简单的反调试技术，核心的勒索病毒代码被多层封装起来了，通过动态调试，解密出外壳的封装代码，如下所示：

![](https://bbs.kanxue.com/upload/attach/201909/288797_W2XDMUWUAF4JEFV.png)

继续跟踪调试，最后解密出核心的勒索病毒代码，如下所示：

![](https://bbs.kanxue.com/upload/attach/201909/288797_RFPWZKTQM4ANPRM.png)

解密出完整的勒索病毒核心是一个PE文件，采用Delphi语言进行编写，如下所示：

![](https://bbs.kanxue.com/upload/attach/201909/288797_HUJTDMEV232NCW8.png)

查看入口代码，如下所示：

![](https://bbs.kanxue.com/upload/attach/201909/288797_FGG2Z7DE6DXKUW5.png)

使用IDA查看字符串信息，里面包含勒索相关信息，如下所示：

![](https://bbs.kanxue.com/upload/attach/201909/288797_J5X4ZQXF7MB9JVR.png)

此勒索病毒最后还会删除磁盘卷影副本等操作，如下所示：

![](https://bbs.kanxue.com/upload/attach/201909/288797_TN9WZ5MCEBGX5EQ.png)

从勒索病毒的核心代码，我发现这款勒索病毒与之前我分析过的一款新型的勒索病毒GermanWiper创建的互斥变量一模一样，都是`HSDFSD-HFSD-3241-91E7-ASDGSDGHH`，GermanWiper勒索病毒的核心代码同样是使用Delphi语言编写的，于是我将两款勒索病毒的核心Payload代码进行对比，如下所示：

![](https://bbs.kanxue.com/upload/attach/201909/288797_ZWN62VJ9BQUB33F.png)

代码相似度也高达82%，所以我觉得这款勒索病毒应该是GermanWiper最新变种，国外安全研究人员又取了一个名字叫：EvaRichter勒索病毒，还是按国外安全研究人员的名字命名吧

自从GandCrab勒索病毒于今年6月1 号，宣布停止运营之后，虽然后面再也没有见到过GandCrab的最新版本出现，然后它的影子似乎一直都在，比如现在比较流行的Sodinokibi勒索病毒，国外很多报道直指Sodinokibi与GandCrab有千丝万缕的关系，前面我就说过GandCrab勒索病毒虽然结束了，背后的运营团队只是宣布停止GandCrab勒索病毒的运营，它会不会继续运营其他勒索病毒家族呢？也许只是GandCrab勒索病毒的开发者不想开发了，它背后的运营团队会不会接着运营其它病毒呢？勒索病毒已经成为了黑产来钱最快的方式之一，大部分做黑产的不就是为了赚大钱吗？难怪会放弃这么好赚钱的机会？

GermanWiper勒索病毒上个月流行过一段时间，这款勒索病毒后面会不会大面积传播，需要持续的关注与跟踪，以前做黑产的都喜欢在放假的时候搞点事情，因为对于做黑产是没有休息日，而且越是放假，越容易搞一些活动，捆绑一些木马让用户主动下载，做黑产的为了搞钱，每天都在寻找不同的攻击目标，通过各种漏洞+恶意软件对目标进行攻击，获取暴利

此前推出的《天天威胁情报》，每天都会收集整理全球最新恶意样本IOC信息，分享最有价值的威胁情报，感谢大家的支持，现已将《天天威胁情报》转移到知识星球，加入知识星球，可解锁每天的威胁情报，同时还可以深度研究分析全球各种恶意样本

知识星球天天威胁情报本期内容播放

Emotet银行木马、Ursnif银行木马、AZORult商业间谍软件、Predator The Thief木马、AgentTesla商业间谍木马、Amadey僵尸网络木马、Lokibot后门、Dridex银行木马、MegaCortex勒索软件、GootKit银行木马

最后欢迎大家关注此公众号，扫描下方二维码：

![](https://bbs.kanxue.com/upload/attach/201909/288797_9HN43YS7V6GFT9V.jpg)

这里没有一些“假大空”的安全概念和框架，只有实实在在最基础的安全研究、最新的威胁情报、安全事件剖析、以及笔者的一些安全观点与看法，如果你对这些恶意软件感兴趣，想深入研究学习一些恶意软件的分析技术，想了解更多恶意样本背后的信息、欢迎加入知识星球，加入星球的朋友可以加入《安全分析与研究》专业群，与群里的各位安全研究员一起交流，讨论，研究各种安全技术，让你在学习成长的路上多一个伙伴

![](https://bbs.kanxue.com/upload/attach/201909/288797_A86W6HSAMSNPY3Y.png)

安全的路很长,贵在坚持......

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
