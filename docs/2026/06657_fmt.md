
# [翻译][12月专题]TheMida_defeating_ring0

http://bbs.pediy.com/showthread.php?s=&threadid=25001&highlight=antiantidump

http://bbs.pediy.com/showthread.php?s=&threadid=25564&highlight=antiantidump

为了帮助理解，建议大家先把上面两个连接的文章先看看

本人英语水平有限，有的地方理解不清，语文水平有限，有的地方表达不清

最近想学习RING0的东西，看了看这文章觉得单词还算简单，就翻译了。

原文：

Tutorial: TheMida Defeating Ring0

Author: deroko

```
fddK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4J5N6r3g2S2L8g2)9J5k6h3q4U0j5$3g2K6M7%4u0G2L8%4c8Q4x3X3g2U0L8$3#2Q4x3V1k6@1N6i4c8G2M7X3W2S2L8s2y4Q4x3X3g2Z5N6r3#2D9i4K6y4r3k6X3W2V1i4K6y4p5x3e0t1$3
```

大家就对照着凑合看了

另外说说我从本文总结出的脱未知壳（指没有相关脱壳文档）的一般流程：

（解决`anti dump`）-> `dump` -> 静态分析，判断是何种语言，根据该语言RTL的特点找到并修正`OEP` -> 运行该`DUMP`，得到引起异常的地址，在未脱壳程序中的该地址下内存断点，定位加密`IAT`或其他加密代码，分析然后修正。

附件在3楼。。。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
