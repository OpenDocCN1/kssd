
# [翻译]将SMB客户端bug转向服务端

原文在：ea3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3N6Q4x3X3c8D9j5i4g2J5k6h3&6@1i4K6u0W2j5X3I4G2k6%4y4H3L8%4c8Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5H3i4K6u0r3x3o6c8Q4x3V1k6@1N6i4u0F1K9h3&6Y4i4K6u0V1M7$3#2T1i4K6u0V1j5$3I4A6k6h3&6@1i4K6u0V1j5Y4g2Y4i4K6u0V1N6r3!0Q4x3X3c8K6k6i4u0$3k6i4u0Q4x3X3c8K6K9h3c8W2i4K6u0W2K9s2c8E0L8l9`.`。在本文附加的doc文档中有中英文对照。

这篇文章介绍的是在无任何用户交互的情况下自动造成SMB客户端的bug溢出的一种技术。该漏洞溢出针对的是域主浏览器（DMB）或者主域控制器（PDC）（也只有PDC可以成为DMB），主域控制器基本上是渗透测试实验中最好的目标。针对DMB的攻击很完美，原因很简单：如果你控制了它，你将能控制所有连到DMB的计算机。

看到微软的安全研究与防御（SRD）再次低调处理SMB客户端的bug，我认为还是公开一下这种类型的漏洞利用方法比较好。

这里要重点指出浏览器和NetBIOS 命名服务（NBNS）一直不给力是一个公开的秘密，因为这些协议在设计之初就没有考虑到安全的因素。而这篇文章正是其中一个简单、真实的例子。

……

具体见附件。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [Turning SMB client bug to server side.doc](javascript:void(0);)

  （75.00kb，204次下载）
