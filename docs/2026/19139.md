
# 华为光猫HG8245H救砖笔记③探查JTAG并使能JTAG

原文发在我的blog：[feaK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3g2J5M7$3!0X3N6q4)9J5k6h3&6W2N6q4)9J5c8X3q4J5j5$3S2A6N6X3g2K6i4K6u0r3x3e0t1I4](https://bbs.kanxue.com/elink@ec2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3g2J5M7$3!0X3N6q4)9J5k6h3&6W2N6q4)9J5c8X3q4J5j5$3S2A6N6X3g2K6i4K6u0r3x3e0t1I4)

这篇文章开始有点干货，主要讲了，怎么验证JTAG接口是否正确。

## 正文

### 前言

华为一些老型号光猫的JTAG接口，网上可以找的到（见[链接](https://bbs.kanxue.com/elink@b8cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4K9h3E0A6i4K6u0W2L8%4m8W2L8Y4N6J5N6q4)9J5k6h3!0J5k6#2)9J5c8Y4c8G2K9q4)9J5c8X3S2#2j5i4N6W2K9g2)9J5c8X3S2Y4y4e0f1$3j5g2)9J5x3$3A6@1j5h3M7`.），是10 Pin接口的JTAG，如下图：

[![](https://bbs.kanxue.com/upload/attach/201702/423318_fda4e2a917f3a0c1.jpg)](https://bbs.kanxue.com/upload/attach/201702/423318_fda4e2a917f3a0c1.jpg)

在HG8245H上找到外观一样的接口：

[![](https://bbs.kanxue.com/upload/attach/201702/423318_4ba7bbba6ea40970.jpg)](https://bbs.kanxue.com/upload/attach/201702/423318_4ba7bbba6ea40970.jpg)

根据图示连接`TCK`、`TDO`、`TMS`、`TDI`、`GND`后，使用`zJtag`尝试探测JTAG菊花链上挂了多少设备，就能看出JTAG接口是否正确了。

配置`zJtag`使用`FT2232`系列接口后，发现无法探测到JTAG菊花链的数量。但是换别的板子的JTAG接口就可以。断定`HG8245H`的JTAG位置不正确，或者其他原因导致芯片内部的调试逻辑部分未启用。

### 看走线，识接口

为了弄清楚，JTAG连接不上的原因到底是找错了位置，还是其他原因导致芯片内部的调试逻辑部分未启用。需要观察JTAG引脚走线，如果走线不正常，那也就不必浪费时间在这个假JTAG接口上了。

背面JTAG接口过孔图：[![](https://bbs.kanxue.com/upload/attach/201702/423318_1b7d23d229e356bb.jpg)](https://bbs.kanxue.com/upload/attach/201702/423318_1b7d23d229e356bb.jpg)

正面JTAG接口过孔图：

[![](https://bbs.kanxue.com/upload/attach/201702/423318_a41bd1c3aa18b870.jpg)](https://bbs.kanxue.com/upload/attach/201702/423318_a41bd1c3aa18b870.jpg)

简单梳理下数据：

- `TCK`对应①；`TDO`对应②；`TMS`对应③；Pin 7 （未知引脚）对应④；`TDI`对应⑤、⑥
- ⑦、⑧号过孔，直连`vRef`即`VCC`（3.3 V）。
- ③、⑤号过孔通过`R1523`（4.7 KΩ）、`R1522`（4.7 KΩ），上拉到`VCC`。
- ①、④号过孔通过`R1590`（1 KΩ）、`R1586`（1 KΩ），下拉到`GND`。

查阅相关资料，在百度百科`JTAG`词条（https://bbs.kanxue.com/elink@f1fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0S2K9h3E0W2i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6A6N6r3g2E0i4K6u0r3d9W2c8m8c8#2)9J5x3K6t1`.）下面发现一段话：

> 根据相关数据手册中的说明，`TRST`、`TDI`、`TMS`引脚上需要接一个10KΩ的上拉电阻，而`TCK`需要接一个10KΩ的下拉电阻。

虽然以上内容，并不适用于所有JTAG接口的配置，但至少指明了一种情况。

这里`HG8245H`的`TMS`、`TDI`的确是上拉到`VCC`的，`TCK`也是下拉到`GND`。

这样看来，JTAG接口似乎没有找错。那问题可能是其他原因导致芯片内部的调试逻辑部分未启用。

搜索了下相关的资料，得知某些产品的JTAG，还存在一个`DBGSEL`的引脚，该引脚在CPU复位时需要上拉到`VCC`，才能使能JTAG调试逻辑。

`HG8245H`的10 Pin接口，排除掉2个`GND`，1个`VCC`，4个JTAG引脚，还剩3个。

使用万用表测量了下这3个引脚的电压，Pin 7的电压在0.12 V左右，Pin 6，Pin 8没有电压。并且用万用表的电流档，短接`VCC`与Pin 6、Pin 8后发现没有电流通过。断定Pin 6、Pin 8为`NC`。

尝试将Pin 7直接连到`VCC`，然后给`HG8245H`上电。`UART`里只输出了一遍`HuaWei StartCode`的报错信息，并且`zJtag`也成功检查到了JTAG菊花链。至此使能`HG8245H`的JTAG调试逻辑成功实现。

整理后的10 Pin定义图：

[![](https://bbs.kanxue.com/upload/attach/201702/423318_985a44d89819130d.png)](https://bbs.kanxue.com/upload/attach/201702/423318_985a44d89819130d.png)

### 后记

实际上要想通过JTAG调试`HG8245H`，不光需要上拉`DBGSEL`，还需要CPU内部使能JTAG调试，因为我的`HG8245H`搞坏了`UBoot`。所以当`StartCode`找不到`UBoot`引导，并且检测到`DBGSEL`上拉时，才使CPU进入调试状态。以上为本人推测，可能不一定准确。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [blog.csersoft.net-华为光猫HG8245H救砖笔记③探查JTAG并使能JTAG.pdf](javascript:void(0);)

（781.78kb，55次下载）
