
# [原创]简单的so脱壳器

自己总结了一下壳的三个发展流程：

1. 文件本地加密，导入内存解密，壳加载器跑完不再做其他事情。
2. 在程序正常运行时，壳可以重新接管控制权。
3. 虚拟机保护。

而当前众多android保护厂商，在so方面都停留在1阶段，还有很多没有。有so保护的不免费。

我们的so保护是免费放出。

可访问 6c2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3&6S2k6$3q4H3N6q4)9J5k6h3y4G2L8b7`.`. <- 这里打个广告，免费。想加直接上传so就好。![](https://bbs.kanxue.com/view/img/face/16.gif)

这个简单的脱壳器，就是针对第一阶段而言。对于当前强度不是很高android保护而言，直接从内存中dump就好了。修复其实就是对重定位表以及got表的修复。这份代码中，没搞那么复杂。我太懒了，有需求的朋友自己修改吧。

这个简单的脱壳器就是`linker`代码的简单应用。用法直接看`option.cpp`就可以了。

下来之后直接`make DEBUG=1 all`即可编译调试版本。`make all`可以编辑发布版本。

配置ndk在最外层的`Makevars.global`中配置ndk的路径。

从github下载时，切换到dev版本即可。代码很乱，随意修改了一下`linker`而已。

以下是下载地址:

695K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1k6i4k6A6L8r3!0Y4K9h3y4Q4x3V1k6#2k6r3!0Y4

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
