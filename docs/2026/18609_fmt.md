
# 支持32/64位Win7SP1的CVE-2019-0803 PoC

### 这个漏洞在网上可以搜到一个PoC：aedK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6q4P5s2m8x3K9h3k6W2x3o6l9I4x3g2)9J5c8V1y4h3c8g2)9J5k6o6t1H3x3e0W2Q4x3X3b7H3z5o6l9K6

（这个Github仓库看上去不是原作者的，是fork过来的；原作者似乎已经把这个仓库删掉了）

但是这个PoC只能在64位Win7下运行。按照资料上的描述，这个漏洞在32位系统上也存在。于是我就想试试能不能移植到32位Win7上。

本人水平太菜，一开始对这个漏洞利用是完全懵逼的。后来搜到了[COMMSEC的这份PPT](https://bbs.kanxue.com/elink@7f3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8$3&6X3k6i4u0W2L8X3y4W2i4K6u0W2K9r3W2@1j5W2)9J5k6h3!0J5k6#2)9J5c8X3S2A6N6r3u0K6k6h3y4U0L8$3&6X3x3U0l9I4z5r3q4E0M7#2)9J5c8X3#2S2N6r3g2J5K9h3q4D9M7#2)9J5c8V1b7I4i4K6t1#2x3U0m8o6e0@1#2y4f1@1g2o6i4K6t1#2x3U0m8Q4x3X3c8Q4x3U0f1J5x3q4y4S2K9h3k6Q4x3U0f1J5x3p5g2D9M7$3S2W2M7X3g2A6i4K6t1#2x3U0m8S2L8X3c8Q4x3U0f1J5x3p5W2S2L8W2)9J5y4e0t1H3d9%4u0G2L8Y4q4#2K9i4y4@1i4K6t1#2x3U0m8Q4x3X3c8Q4x3U0f1J5x3q4c8Z5k6g2)9J5y4e0t1H3e0r3W2X3k6g2)9J5y4e0t1H3i4K6t1$3j5h3#2H3i4K6y4n7i4K6t1#2x3U0m8p5k6h3q4@1K9q4)9J5y4e0t1H3L8$3k6Q4x3U0f1J5x3p5E0W2M7X3&6W2L8q4)9J5y4e0t1H3e0$3u0B7k6h3y4@1i4K6t1#2x3U0m8m8j5Y4g2K6k6g2)9J5k6i4m8V1k6R3`.`.)，懵逼程度才稍微减少一点。同时，我也参考了一些前辈大牛的分析（[360分析](https://bbs.kanxue.com/elink@069K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6*7K9s2g2S2L8X3I4S2L8W2)9J5k6i4A6Z5K9h3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0j5J5y4e0t1H3x3o6l9$3)、[看雪分析1](https://bbs.pediy.com/thread-252447.htm)、[看雪分析2](https://bbs.pediy.com/thread-252645.htm "a_1564473481779")）。

经过一番折腾，PoC已经能在32位Win7下成功提权了，但是有一个小问题：程序退出的时候，机器会直接蓝屏——原因呢，应该是`SetWindowTextW`替换Token进行提权的时候，`EPROCESS`结构体的`WorkingSetPage`成员被字符串尾部的`0x0000`覆盖破坏掉了。64位Win7也有类似的问题，但蓝屏的概率不大。

虽然这个漏洞有其他利用方式，但本人实在太菜，所以还是套用原作者利用`tagWND`这个方法，把`WorkingSetPage`原先的数值重新写进去，这样看上去就没有蓝屏问题了。

如果是其他版本的系统，应该还有一些偏移量之类的要改一下。

##

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2019-7-30 18:55

被cleanll编辑，原因：

上传的附件：

- [CVE-2019-0803-win7sp1.zip](javascript:void(0);)

（14.55kb，155次下载）
