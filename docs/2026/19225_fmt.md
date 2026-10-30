
# [原创]初玩树莓派B 1基本介绍

初玩树莓派 B 1基本介绍

bbs.pediy.com

之前早就买了一个树莓派B，但是一直没有玩起来。由于以后也准备走嵌入式开发方向。于是先提前把树莓派B玩起来把。

之所以选树莓派是因为国内外玩的人多。中英文资料也很多。为什么不选单片机，单片机可能更多需要了解底层编程，初学者门槛比较高。而对于树莓派来说，不需要对于底层开发或者硬件细节有太多了解，入门门槛较低，可以玩的应用也很多。初玩朋友可以把这个当做一个性能差一点的台式电脑主机，也可以当做性能超强的单片机。

特别提示：本人也是刚刚开始玩，不是老手，所以很多情况下也是一知半解，一半测试，一半当做记录做的这个部分，如果资料中有错误或者其他问题，欢迎大家指出。后续会慢慢更新。有些细节的东西可能由于没有用到就部过多讲了。（真的是慢慢更新，因为我也是边玩边学习边记录）。

![](https://bbs.kanxue.com/upload/attach/201609/528829_lp2mijzi1zq33wz.png)

图来源不记得了

## 1 基本介绍

树莓派分N多个型号。我买的B型是比较早期的，主频700MHZ（可以超频到1GHZ以上，伤寿命，不建议），512M内存，，VGA，HDMI视频口，以及一个音频口。B版本外接口比较少，USB2个 。GPIO口是26个。目前已经到树莓派3代+，大家要玩可以买最新的，性能更好，接口也更多。具体一些细节可以参看维基 。

`ec3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6*7K9q4)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6Q4x3U0g2q4y4W2)9J5y4f1p5H3i4K6t1#2z5e0q4Q4x3U0g2q4z5q4)9J5y4e0S2q4i4K6t1#2z5e0y4Q4x3U0g2q4y4W2)9J5y4f1t1@1i4K6t1#2b7V1f1`.

![](https://bbs.kanxue.com/upload/attach/201609/528829_01carmlg7my12rf.png)

图来源维基：

我初玩购买的设备有：

1 树莓派主机（必备）
2 散热片
3 树莓派SD卡和卡套 （其他版本可能不需要卡套，根据需要买卡套）（必备）
4 树莓派外壳，保护树莓派
5 免驱无线网卡
6 USB转TTL串口调试线

以下是设备以及当时买的价格，（现在买最新的型号也比我的便宜）

![](https://bbs.kanxue.com/upload/attach/201609/528829_nsnfemrd9ooe2e5.png)

![](https://bbs.kanxue.com/upload/attach/201609/528829_tp5ingybavfpj3n.png)

![](https://bbs.kanxue.com/upload/attach/201609/528829_pyyj7zfqe7crood.png)

关于这些设备
1 散热片随意。便宜就买几个。贴在CPU上，树莓派是CPU GPU共用一个核心。发热还是有点厉害
2 树莓派SD卡建议买C10（U1）速度的，毕竟这个就是硬盘了，速度快肯定运行也快。
3 树莓派外壳，保护树莓派，我买的这个30块的已经超级硬，绝对足够保护。（可惜没有留出GPIO的出口，我是自己割开的，由于质量太好，我用的钳子都歪了，后来用烙铁烫开了一个口子）。
4 免驱无线网卡，有些外设最好买无驱或者有提供驱动的，自己找很麻烦，树莓派最好也用无线。这样连起来方便。
5 USB转TTL串口调试线 。这个对于没有显示器的朋友是很重要的。前期需要串口登陆配置设备，无线等等，后期调试也可以使用。

有了以上基础的硬件设备就可以开始玩了。更多基础介绍可以 谷歌搜索 不再说明了

## 2 其他类型应用

树莓派的34个使用

来源：`184K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9K9h3&6#2P5s2c8G2P5g2)9J5k6h3!0J5k6#2)9J5c8X3q4J5j5$3S2A6N6X3g2K6i4K6u0r3j5$3!0G2L8q4)9J5k6r3W2V1k6h3q4K6i4K6u0V1k6X3!0J5i4K6u0V1M7X3q4K6M7r3u0W2M7Y4u0&6i4K6u0V1M7r3W2Q4x3X3g2Z5N6r3#2D9`

如果你手头有一个 `Raspberry Pi`（树莓派），你会拿它来做什么？或许以下 34 个如何使用 `Raspberry Pi` 的创意能够给你带来一些启发。

Web 服务器
家庭自动化
BitTorrent 服务器
Web Cam 服务器
天气预报站
BitCoin Wallet
QuadCopter
VoIP PBX
XMBC 多媒体中心
有声书籍播放器
Arduino Shields
NAS 服务器
Apple Time Machine 支持
Tor 中继
家用 VPN 服务器
GPS 跟踪器（带 3G 支持）
Advice Machine（无用但很酷）
模拟输入
超级电脑
Kindle 作为显示屏
PIC Programmer
PenTesting/Hacking
Android 系统
检查网络状态
Solar 数据记录器
把我发到太空
咖啡
制作一个酷坦克
电子相框
添加 WiFi
OpenSource Kiosk
Node JS
流量监视
超频

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：
- [1.png](javascript:void(0);)
  （46.02kb，28次下载）
- [2.png](javascript:void(0);)
  （203.66kb，25次下载）
- [3.png](javascript:void(0);)
  （91.66kb，30次下载）
- [4.png](javascript:void(0);)
  （26.56kb，14次下载）
- [5.png](javascript:void(0);)
  （26.98kb，16次下载）
- [初玩树莓派B 1基本介绍.pdf](javascript:void(0);)
  （445.49kb，87次下载）
