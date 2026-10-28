
但现在一旦出现`AmsiScanBuffer()`，该字符串都有可能被 Windows Defender 查杀。所以就 Egg Hunter 这种方法，也是先加载别的不相关的函数，如`DllCanUnloadNow()`：

![](https://bbs.kanxue.com/upload/attach/202103/718877_EQJUHEA9D7SEUNB.png)

然后再通过保存`AmsiScanBuffer()`前 24 个字符：

![](https://bbs.kanxue.com/upload/attach/202103/718877_BWVGGUKUCWSTGFE.png)

![](https://bbs.kanxue.com/upload/attach/202103/718877_FNEK7BUF9BB3ZX9.png)

最后调用`FindAddress()`从内存捞出`AmsiScanBuffer()`相应的地址：

![](https://bbs.kanxue.com/upload/attach/202103/718877_YP2AVFY2S6ZVUQ6.png)

完，谢谢大家观看。

## 参考链接

`378K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0L8$3c8W2P5s2m8Q4x3X3g2%4L8%4u0V1M7s2u0W2M7%4y4Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5&6i4K6u0r3x3o6k6Q4x3V1j5H3x3#2)9J5c8X3c8A6M7$3q4T1L8r3g2Q4x3X3c8S2L8i4y4A6i4K6u0V1N6$3I4V1M7q4)9J5k6r3c8G2N6r3&6W2N6q4)9J5c8R3`.`

`b8cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8$3&6@1k6i4S2@1K9i4y4Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8W2)9J5c8X3u0D9L8$3N6Q4x3V1k6S2L8i4y4A6i4K6u0V1j5Y4W2H3j5i4y4K6`

[[招生]科锐逆向工程师培训（2026年11月13日实地，远程教学同时开班，第57期）](https://bbs.kanxue.com/thread-51839.htm)

[#基础知识](https://bbs.kanxue.com/forum-41-1-130.htm)
