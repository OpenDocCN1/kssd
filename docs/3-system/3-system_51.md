
# 纯手工添加一个 PE 节

> 原文：[`www.pediy.com/kssd/pediy12/131142.html`](https://www.pediy.com/kssd/pediy12/131142.html)

标 题: 再写手工打造可执行程序

作 者 ID: `yulongying`

时 间: 2011-03-03

最近在公司时间比较闲暇，所以一直都在论坛闲逛，前段时间看了 `dncwbc` 一篇文章

《再写手工打造可执行程序》

<http://bbs.pediy.com/showthread.php?t=130261>

还写了一篇读后感《再写手工打造可执行程序 读后感 》

<http://bbs.pediy.com/showthread.php?t=130261>

含金量比较低，但是对于新手还是有价值的，这里在前两篇文章基础上，探讨下怎么手动添加一个以 0 填充的 section，其实原理都是一样，含金量也是比较低 呵呵 高手莫笑！

要修改的一共有以下几处

1、PE 头结构中的 `NumberOfSections` +1 = 3+1=4

![](img/a798eda9977c64d6f9ea5a9ab0e7e691.jpg)

2、然后就是 `optional header` 中的 `SizeOfImage`（加载到内存中的总大小）之前是 (1+3)\*1000=4000，这里再解释下此表达式 1 代表就是文件头 3 代表 pe 文件中使用了 3 个 section，1000 就是 `SectionAlignment` 了；现在应该是 (1+4)\*1000=5000

![](img/1314fcb9998f1be4b07fe87821d7e1c2.jpg)

3、在之前的 `.text` `.rdata` `.data` 三个 `section header` 后面的空白文件段添加添加一个 `section header`，对此 `section header` 分别域进行填充

`section name` 8 个字节：`.hawk`

加载到内存中的大小 4 个字节：1000

`VA`：最后一个 section 的 `VA`+1000=3000+1000=4000

在文件中的大小还是:200

文件偏移：最后一个 section 文件便宜+其大小=800+200=A00

`PointerToRelocations` `PointerToLinenumbers` `NumberOfRelocations` `NumberOfLinenumbers` 依然均为 0

`Characteristics` 我随便填了 40 00 00 40

![](img/68891decf80da90356a8eb97349348a6.jpg)

4、从 新增的 section 起始便宜 A00 一直填充 0 到 BFF（大小为 200h）

![](img/ef711e87998ca2717cce5c065c51ec59.jpg)

### 补充：

1、

添加一个 `section header` 我们发现是在 `file header` 与 `sections` 之间对齐之后产生的空隙中，如果不存在空隙或者空隙控件不够 40（之前笔误写了 40h 谢谢 `jsjnwms` 提醒）此方法就不使用了，应该就比较麻烦了，难道要重组 `pe`？这里还请高手解答。

2、我们 `pe fileheader` 总大小 `dncwbc` 那篇文章计算过了原话：“我们的 PE 文件头总大小为：64 + 112 + 4 + 20 + 224 = 424，3 个节表头的总大小 3 \* 40 =120。424 + 120 = 544 byte 转化成十六进制为 220h”，现在我们已经添加过一个 `section header` 现在应该为 544+40=584 转化成十六进制为 248h 经过 `file alignment` 200h 对齐之后还是 400，在这里不需要修改，如果对齐之后有变化那这块也需要修改。

![](img/fb9bd4214b539db24e1651b13955b2fd.jpg)

上面仅仅是一个做为菜鸟的见解，如有遗漏之处还请各位多多抛砖指引哈。

上传的文件跟《再写手工打造可执行程序读后感》中的文件用 `UE` 简单对比以下就知道那些改动了。

### 上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || `add_myHelloworld.rar`
|
