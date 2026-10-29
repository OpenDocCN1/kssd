
从上面代码可以看到调用`mono_image_open_from_data`获取`MonoImage`，再调用`mono_assembly_load_from`完成对`dll`的加载，所以`LoadGame`最终也会走到`libNetHTProtect.so`中`hook`函数，判断是否须要修复，函数走完后直`dump Assembly-CSharp.dll.dll`就可以了，将`dump`出来的`dll`反编译后正常，如图4、5所示。

![](https://bbs.kanxue.com/upload/attach/201908/536985_Z6GFQ3EUFKZV7VT.jpg)

图4

![](https://bbs.kanxue.com/upload/attach/201908/536985_EFZH3NPD3ZURHK2.jpg)

图5

整个反外挂流程到这里基本分析完成。

## 0x03：总结

1. 该反外挂系统主要用到了`svc 0`系统调用做反调试，再加上动态加密解，内联、混淆，在一定程度给调试分析上加大了些难度。至于是否安全，个人觉得仁者见仁，智者见智，一个安全商业产品要做到产品易用性与绝对安全兼得基本上是不太可能的。

样本下载链接：ecfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4d9f1N6G2i4K6u0V1g2e0k6Y4g2K6M7&6f1q4S2W2x3@1W2Q4x3X3c8y4K9f1k6x3k6#2p5`. 提取码：3f5x

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
