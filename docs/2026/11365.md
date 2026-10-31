
# [原创]Bypass“微点主动防御 2.0 公测版”

微点主动防御软件 测试版

- 程序版本： 2.0.20266.0108
- 特征版本： 1.6.1852.110421
- 更新时间： 2011-04-21 16:22:26
- 下载地址：[1c3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0T1M7#2)9J5k6h3#2A6j5%4u0G2M7r3!0A6L8Y4c8Q4x3X3g2U0L8$3#2Q4x3X3g2U0L8W2)9J5c8Y4y4Z5L8%4N6@1K9s2u0W2j5h3c8Q4x3X3g2S2M7%4m8Q4x3@1k6@1K9h3c8Q4x3@1b7^5x3e0b7^5z5b7`.`.](https://bbs.kanxue.com/elink@cf7K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0T1M7#2)9J5k6h3#2A6j5%4u0G2M7r3!0A6L8Y4c8Q4x3X3g2U0L8$3#2Q4x3X3g2U0L8W2)9J5c8Y4y4Z5L8%4N6@1K9s2u0W2j5h3c8Q4x3X3g2S2M7%4m8Q4x3@1k6@1K9h3c8Q4x3@1b7^5x3e0b7^5z5b7`.`.)

微点官网出了微点2.0公测版，于是装了一个，界面不错。。。

可惜由于微点不监控驱动加载，2年前的一个老方法依然完美过微点。即将自身程序加入微点的白名单。

`sudami1.exe`作用是远程注入桌面导致`explorer`崩溃，看看效果，仅供娱乐。

![](https://bbs.kanxue.com/upload/attach/201104/57632_20iw3nxf4r1oain.gif)

测试方法：

1. 直接运行`sudami1.exe`，会被微点提示木马

2. 用`InstDrvA.exe`加载`1.sys`，再次运行`sudami1.exe`，桌面被成功结束

附件中的`RemoteThreadMonitor.c`为关键代码，当时胡乱测试用的，供参考

2年前的帖子见：[077K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2A6i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6K6N6h3c8S2L8h3W2Q4x3V1k6T1L8r3!0Y4i4K6u0r3K9i4c8W2L8g2)9J5c8X3g2U0j5X3q4W2y4h3g2U0k6e0j5K6k6X3c8T1x3$3p5J5y4K6V1%4z5e0p5$3y4#2)9J5k6h3S2@1L8h3H3`.](https://bbs.kanxue.com/elink@4abK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2A6i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6K6N6h3c8S2L8h3W2Q4x3V1k6T1L8r3!0Y4i4K6u0r3K9i4c8W2L8g2)9J5c8X3g2U0j5X3q4W2y4h3g2U0k6e0j5K6k6X3c8T1x3$3p5J5y4K6V1%4z5e0p5$3y4#2)9J5k6h3S2@1L8h3H3`.)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#系统底层](https://bbs.kanxue.com/forum-4-1-2.htm)

上传的附件：

- [`测试效果.gif`](javascript:void(0);)（302.57kb，753次下载）
- [`BypassMP2.0.rar`](javascript:void(0);)（249.30kb，133次下载）
