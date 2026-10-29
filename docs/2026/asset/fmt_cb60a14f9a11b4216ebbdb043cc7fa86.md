
2. 在特殊情况下，广告服务器可能会要求使用特定的代理服务器或 IP 地址进行广告请求。这时候，动态代理就可以被用来实现这些特殊的网络访问要求，确保广告请求能够成功发送和接收。

进一步分析，我们可以想到广告不太会是在软件刚出来时就加上，一定是后续附加上去的功能。后续除了广告之外肯定也会陆续附加其他功能，如何做到这些功能扩展呢？这就可以用 `proxy` 代理类了，将播放器核心功能（播放视频）融入到代理类中，让其负责对核心功能进行扩展（如在播放视频之前添加广告）。这样既方便后续软件更新，也会使逻辑更加清晰、出错时能快速定位。

**android 广告 SDK 原理流程图**

![](https://bbs.kanxue.com/upload/attach/202304/939796_CEU8XPWD2F2YU8D.png)

[参考链接]：

[Android反编译实战-去广告\_安卓反编译去除广告\_sam.li的博客](https://bbs.kanxue.com/elink@cbcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4y4S2L8h3I4A6M7X3!0F1k6%4y4Z5k6h3&6Y4i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5I4x3e0p5$3z5o6b7@1x3K6t1`.)

[android广告SDK原理详解（附源码） - 爱码网 (likecs.com)](https://bbs.kanxue.com/elink@28eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2D9K9h3E0W2j5%4y4Q4x3X3g2U0L8$3#2Q4x3V1k6K6K9r3!0%4i4K6u0V1x3U0l9@1y4e0V1^5y4U0t1@1i4K6u0W2K9s2c8E0L8l9`.`.)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2023-4-7 12:51 被 I鲸落I 编辑，原因：

[#逆向分析](https://bbs.kanxue.com/forum-161-1-118.htm)

[#HOOK注入](https://bbs.kanxue.com/forum-161-1-125.htm)
