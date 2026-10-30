
# [原创]iot初体验

## 写在前面

iot安全是早早就萌生了想法，奈何早之前当了咸鱼王，动手能力被摒弃。

转行进入安全行业后，对安全技能的需求变成了刚需，不能再当咸鱼。

趁着这边的项目延迟有大量的空闲时间，就在酒店开始了这一次的摄像头逆向之旅。

这里非常感谢[wmsuper](https://bbs.pediy.com/user-home-651413.htm)这些年来的指点.

本文主要目的是对摄像头固件进行修改进入uboot.

用到设备如下:

0、摄像头

1、万能表.

2、`RT809F`编程器

3、电洛铁

4、`UsbToTTL`

## 写在中间

### UART串口焊线

这块板子对新手极其友好，接口名称都还在，且经过万能表测试正确，只需要简单的接线即可。

![](upload/attach/202104/749276_M8ZP87S9MFDXMNG.jpg)

![](upload/attach/202104/749276_E4XPJN6TK9ABY2U.jpg)

### 连接UART

`Rx`、`Tx`接口跟`UsbToTtl`的`Rx`、`Tx`对换连接。

![](upload/attach/202104/749276_VF3PAGXKQPB4UDZ.jpg)

设置对应的端口和波特率。

![](upload/attach/202104/749276_K3BRJ538G7QXJSM.png)

连接上电

![](upload/attach/202104/749276_3WGH5FUNZR96DT4.png)

发现按任意键都无法进入u-boot，能直接进入系统且没有密码。

但是一直会有很多信息刷新，干扰正常输入输出。

![](upload/attach/202104/749276_246PNG55HNGXDWU.png)

没进入u-boot这可不行，上编程器！

### 穿针引线

通过芯片表面的信息，确定了这一小颗就是要动刀的。

![](upload/attach/202104/749276_3NSUFV4M7NK7FEU.png)

由于针脚之间距离太近加上太久没用过电洛铁热风枪极其手生，所以二者不在考虑范围(在有一个就是针脚上都涂了一层透明的东西，在以前的印象中是需要洗板水清晰后才能拆)。

最后，使用了小时候缝衣服的技术-穿线！

一开始用铜线接完发现，不导电(可能涂有绝缘漆)。

所以得把线拆了重新用别的线接，总共花费了两小时，加上酒店的灯光，眼睛都瞎了。最终的成品如下

![](upload/attach/202104/749276_3A5K7GT65T4GVTB.jpg)

![](upload/attach/202104/749276_KKBTRPE2A73MN7P.jpg)

### 修改固件

连接编程器并上电

![](upload/attach/202104/749276_XUW8JZU8KF85TQH.png)

识别出5个型号，因为不知道具体型号，所以直接选了最后一个。

点击读取，等待读取结束保存到本地后，固件提取就完毕了。

![](upload/attach/202104/749276_6YETRMURAWGVTV8.png)

提取固件不需要解包，只需要找到u-boot的程序所在位置就好了。

这里经过wmsuper的指点，知道了固件起始到`0x31000`是u-boot程序所在。

ps:`0x31000`是因为在`binwalk`中识别出该地址为内核。

将固件拉入`ida` 选择`arm` 小端，并设置加载大小为`0x31000`。

![](upload/attach/202104/749276_EA8MAWEW4BNQCVW.png)

打算通过开始的提示字符串定位函数，但发现无法找到引用。

应该是`ImageBase`需要修正。

![](upload/attach/202104/749276_8S9DP2WKA8FEDXT.png)

寻找固件加载基址参考这个文章<a href="elink@be7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2x3%4m8W2L8g2)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3x3U0l9I4z5g2)9J5c8U0l9#2i4K6u0r3x3o6W2Q4x3V1k6u0L8#2c8Q4x3V1k6S2M7X3#2Q4x3U0g2q4y4g2)9J5y4e0W2n7i4K6t1#2b7V1q4Q4x3U0g2q4y4q4)9J5y4f1u0n7i4K6t1#2b7U0k6Q4x3U0g2q4y4g2)9J5y4e0S2m8i4K6t1#2b7e0m8Q4x3U0g2q4z5q4)9J5y4f1u0p5i4K6t1#2b7V1c8Q4x3U0g2q4y4g2)9J5y4e0W2o6i4K6t1#2b7U0m8Q4x3U0g2q4y4g2)9J5y4e0W2p5i4K6t1#2z5o6m8Q4x3V1j5`.">arm固件加载地址</a>

通过搜索jump table的方式找到如下跳转表

![](upload/attach/202104/749276_CXRBSK6XY2FB3DG.png)

通过尝试，`0x80E101D0-103B0=0x80DFFE20`，而两值最后的差正好是20。

所以猜测`ImageBase`为`0x80DFFE00`。

通过测试，`ImageBbase`为正确，也定位到了关键函数

![](upload/attach/202104/749276_YQVVPSA6E268BUC.png)

通过代码可以看到，uboot的等待时间由`sub_80E17F64`决定。

最后修改保存该处。

![](upload/attach/202104/749276_9VPBEWH6NGF9SPC.png)

将修改的固件重新写入

![](upload/attach/202104/749276_3QJ93VWFVVE2XU8.png)

连接uart开始上电测试结果如下

![](upload/attach/202104/749276_MPYUFY7FN35UU9X.png)

可以看到，成功进入uboot模式。

最后收拾残局

![](upload/attach/202104/749276_65WS2Z5BUMH7BYQ.jpg)

## 写在后面

第一次整，除了用那个铜线没有测试导电就去使用之外非常顺利。

对于这一次动手过程非常开心，这个便宜的摄像头对新手出其的有好。

也非常感谢wmsuper这几年不耐烦的教导。

最后于 2021-4-24 17:13

被零加一编辑，原因：

[#固件分析](https://bbs.kanxue.com/forum-128-1-170.htm)

[#家用设备](https://bbs.kanxue.com/forum-128-1-173.htm)
