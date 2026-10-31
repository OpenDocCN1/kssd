
# [原创]yoda's Protector V1.03.3静态脱壳机(附源码)

稀里糊涂过了一年,找了份工作.pe格式忘得差不多了.所以拿此壳来练手![](https://bbs.kanxue.com/view/img/face/16.gif)

给新手朋友们一点参考,希望大牛们多拍砖与指点.可能以后的工作会跟壳有关了![](https://bbs.kanxue.com/view/img/face/12.gif)

说下大概流程:

1. 首先解密shell代码的两处加密,由于yp的加密方式有了一点小动态.所以采取了动态抠代码的方式来解密的.(直接用opcode比较效率高一些,人懒了点.带了个反汇编引擎.见谅)
2. 对加密的节区进行三次解密.
3. 如有资源节,则还原.并替换掉`.x01`节为`.rsrc`
4. 还原IAT,将`.yp`节删掉
5. 修复`OEP`,`sizeofImage`等等信息.
6. `dump2file`.

ps:写完后才发现此壳开源的![](https://bbs.kanxue.com/view/img/face/16.gif),有意向的朋友可看源码学习

Yoda's Protector v1.03.3主程序下载地址:

9b9K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4m8W2k6r3W2&6i4K6u0W2j5$3!0E0i4K6u0r3N6r3!0G2L8s2y4Q4x3V1k6b7b7f1y4w2i4K6u0r3f1s2u0G2N6r3g2U0N6r3!0J5M7#2)9J5c8Y4W2G2k6r3q4H3i4K6u0r3P5i4l9I4i4K6u0W2x3o6y4Q4x3X3f1K6i4K6u0W2P5X3W2H3

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [bin.rar](javascript:void(0);)
  （32.23kb，1139次下载）
- [src.rar](javascript:void(0);)
  （68.33kb，949次下载）
