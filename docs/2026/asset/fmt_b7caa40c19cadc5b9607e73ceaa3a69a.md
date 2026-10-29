
8. 以上代码虽然能够正确地执行一些逻辑，但是在最后的`60902`调用时，返回值出错了，并且提示`SecException（1910）`。通过如下链接查找，<mark class="encrypted">355K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Z5k6h3I4H3i4K6u0W2j5h3I4A6P5i4g2F1i4K6u0W2j5$3!0E0i4K6u0r3k6r3!0U0N6h3#2W2L8Y4c8Q4y4h3k6V1k6i4c8S2K9h3I4Q4x3V1j5I4y4U0l9#2y4K6S2Q4x3V1k6X3g2Z5N6r3#2D9</mark>，该错误的原因可能是：非法的`avmpInstance`实例。

![图片描述](upload/attach/202304/941979\_NQ4SGGMKK9J3NKF.png)

![图片描述](upload/attach/202304/941979\_W98HGHF5U99PR4R.png)

10. 补充说明：使用`unidbg 080`版本进行复现。

# 求助各路大佬一起探讨一下原因

> 【注】文章中如有任何侵权，请联系说明修改！
>
> 【注】本文仅用于学习讨论，不做任何商务用途！

# 分析过程

## 文章使用`unidbg`以及代码下载

[7e1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4g2f1y4G2k6r3b7K6x3h3W2n7k6%4q4o6g2i4A6@1d9@1E0V1h3W2A6H3k6H3`.`.](https://bbs.kanxue.com/elink@c72K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4g2f1y4G2k6r3b7K6x3h3W2n7k6%4q4o6g2i4A6@1d9@1E0V1h3W2A6H3k6H3`.`.)

提取码：`5ifo`

1. 网上看了某帖子关于对`sgmain`的`doCommandNative`函数调用的复现，进一步了解了关于`sgmain`系列的安全措施，于是萌生了复现的想法。帖子链接：[981K9s2c8@1M7s2y4Q4x3@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3y4g2)9J5k6i4m8E0i4K6u0r3k6$3!0Q4x3V1k6Q4x3X3b7I4x3U0V1J5y4e0k6Q4x3X3g2Z5N6r3#2D9](https://bbs.kanxue.com/elink@610K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3y4g2)9J5k6i4m8E0i4K6u0r3k6$3!0Q4x3X3b7I4x3U0V1J5y4e0k6Q4x3X3g2Z5N6r3#2D9)

2. 由于之前对某航`app`进行过研究，并且发现其对`wToken`参数加密时也使用了`sgmain`系列进行保护，所以想要针对该`app`进行`unidbg`复现`sgmain.so`的调用。

3. 关于复现前，网上查阅了很多关于`sagmain`的帖子，其中有汇编的大致分析逻辑和`unidbg`复现的相关逻辑，感谢前辈大佬的帖子。

<https://bbs.kanxue.com/thread-267741.htm#msg_header_h2_7>

[255K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8r3!0#2k6q4)9J5k6i4c8W2L8X3y4W2L8Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3x3e0V1J5x3K6p5@1z5l9`.`.](https://bbs.kanxue.com/elink@765K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8r3!0#2k6q4)9J5k6i4c8W2L8X3y4W2L8Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3x3e0V1J5x3K6p5@1z5l9`.`.)

[7a0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8V1A6G2K9r3&6Q4y4h3k6x3k6h3&6G2L8W2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3x3e0t1&6y4e0M7J5x3U0p5%4](https://bbs.kanxue.com/elink@17fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8V1A6G2K9r3&6Q4y4h3k6x3k6h3&6G2L8W2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3x3e0t1&6y4e0M7J5x3U0p5%4)

[bf4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4q4I4i4K6g2X3x3K6t1&6y4e0f1J5x3U0y4Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0p5J5x3o6f1H3x3o6x3#2x3b7`.`.](https://bbs.kanxue.com/elink@5daK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4q4I4i4K6g2X3x3K6t1&6y4e0f1J5x3U0y4Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0p5J5x3o6f1H3x3o6x3#2x3b7`.`.)

<https://bbs.kanxue.com/thread-265017.htm>

4. 关于`doCommandNative`调用前，还需要进行初始化操作。可以从启动`app`开始对该函数调用进行`hook`，打印参数的内容和返回值，然后使用`unidbg`模拟一步步执行。

|  |  |
| --- | --- |
