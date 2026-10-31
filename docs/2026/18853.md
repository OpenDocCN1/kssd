
# [翻译][翻译]逆向Google Voice设备OBi200 (二)

# 逆向Google Voice设备OBi200 (二)

在本文的第一部分,我详细介绍了对OBi200的固件分析和获取root Shell的过程.在本文,我将展示区分串口针脚及利用串口获取控制台访问.

串口通信协议.

回忆一下上篇文章中监听`/dev/ttyS0`的串口:

![](https://bbs.kanxue.com/upload/attach/201710/747320_yiuhqtey1565nmo.png)

这是一个非常赞的开端,其透露出两点信息:第一,这明确了其串口为 8250 UART(典型的4针脚接口),极大地缩小了我们的搜索范围;第二,其指明了该接口的波特率.这些是与该端口通信所必须的条件.

## 鉴别针脚

尽管UART接口并未在业界形成统一的标准,但是要区分出OBi200上潜在的接口还是很容易的:

![](https://i.imgur.com/jE1EWzi.jpg)

从上图可以看到,除了一个名为`J17`的4针模块,其他大多数测试点都已标记完好.由于最右边的那个针脚是方形的,暂且将它命名为1号吧.

现在只剩这几个针脚,接下来要做的就是区分这些UART针脚.看,我已经准备好了USB UART转换器:

![](https://i.imgur.com/RbKExfl.png)

我首先开始找出接地端.用万用表连续测量各个针脚对RJ45外壳的电压,我迅速地确定4号引脚为接地:

![](https://i.imgur.com/f5vUtG7.png)

搞定了1个,剩下的是略显复杂的另外3个脚的区分.为此,我专门测量了在设备启动过程中这3个脚的直流电压(详细过程请浏览[这篇文章](https://bbs.kanxue.com/elink@14eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3c8W2N6Y4c8@1P5i4x3H3i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9I4x3W2)9J5c8U0p5I4i4K6u0r3M7X3g2$3k6i4u0K6k6g2)9J5k6r3g2F1k6$3W2F1k6h3g2J5K9h3&6Y4i4K6u0V1M7$3g2J5K9h3q4D9i4K6u0V1M7r3!0J5N6s2y4Q4x3V1j5`.)),结果如下:

>1号脚电压在启动过程中从2.3V到3.2V波动

>2号脚电压恰巧维持在略低于3.2V

>3号脚为3.3V 恒定电压

1号针脚的活动与其他针脚明显不同,其电压的波动就像是与主板进行数据发送的过程,故此,认定1号脚为Tx端.

2号脚和3号脚的电压测量结果极为接近,但是仍有足够的差异区分他们. 3号脚,电压稳定在3.3V,表明其为 VCC;2号脚电压仅是略低与VCC,故为Rx(仅剩的选择).特别注意,如上图所示,尽管连接USB UART转换器并不需要用到3号脚,我们测试该脚仅仅是为了消除其为Tx 或 Rx 的可能性.

## 控制台连接

确定`J17`的针脚后,用USB UART转换器连接.注意,主板上的Rx 接转换器的Tx ,如下图所示:

![](https://bbs.kanxue.com/upload/attach/201710/747320_vu01wkhf7kzb15x.png)

![](https://bbs.kanxue.com/upload/attach/201710/747320_j1r61etzlstxf24.png)

接下来,我打开了一个终端并开启了一个`screen`会话,波特率设置为上面第一幅截屏所示是一致:

```
screen /dev/ttyUSB0 115200 8N1
```

这样,我就成功获得了控制台的访问权,并且无需密码哦!

### Bootloader

成功连接上控制台后,我还想看下它的bootloader,于是,我重启设备:

![](https://bbs.kanxue.com/upload/attach/201710/747320_y9rpu7dxweflynn.png)

通过与U-boot交互,我得以了解更多有关主板和内存的信息:

![](https://bbs.kanxue.com/upload/attach/201710/747320_fk9j3myiahffeto.png)

通过对硬件底层的访问.我们能够看到更多在启动过程的信息.例如,系统控制台消息输出stdout:

![](https://bbs.kanxue.com/upload/attach/201710/747320_cfbeh24su9cyn3m.png)

由图可知,Telnet服务在系统启动时随之启动,只是在载入OBi 服务后被kill掉了(由上篇文章中的脚本部分可知).

## 接下来>>>

现在,我已获得控制台的root访问权限,我希望进一步深入探究其运行的服务和`obiapp`二进制文件.

相关内容,敬请期待本文第三部分!

原文:[358K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5j5h3&6V1P5i4N6W2M7%4c8W2M7X3N6J5k6h3&6Q4x3X3g2U0L8$3#2Q4x3V1k6J5k6i4k6W2M7Y4y4W2i4K6u0V1k6h3&6Y4K9h3&6W2k6i4u0A6L8X3N6Q4x3X3c8G2j5X3V1J5x3o6m8Q4x3X3c8Y4L8$3!0Y4L8r3g2Q4x3X3c8$3L8$3W2U0k6g2)9J5k6r3q4H3M7r3I4A6j5h3&6U0k6g2)9J5k6s2m8S2M7Y4c8Q4x3X3b7J5i4K6u0r3](https://bbs.kanxue.com/elink@38fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5j5h3&6V1P5i4N6W2M7%4c8W2M7X3N6J5k6h3&6Q4x3X3g2U0L8$3#2Q4x3V1k6J5k6i4k6W2M7Y4y4W2i4K6u0V1k6h3&6Y4K9h3&6W2k6i4u0A6L8X3N6Q4x3X3c8G2j5X3V1J5x3o6m8Q4x3X3c8Y4L8$3!0Y4L8r3g2Q4x3X3c8$3L8$3W2U0k6g2)9J5k6r3q4H3M7r3I4A6j5h3&6U0k6g2)9J5k6s2m8S2M7Y4c8Q4x3X3b7J5i4K6u0r3)

译/ 看雪翻译小组

StrokMitream

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
