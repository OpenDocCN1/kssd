
### 5. 最终目的时调用参数`60902`参数的`doCommandNative`方法对请求内容进行加密获得`wToken`值，思路明确后准备`unidbg`进行如下的调用：（这里借用：<mark class="encrypted">3ffK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3y4g2)9J5k6i4m8E0i4K6u0r3k6$3!0Q4x3X3b7I4x3U0V1J5y4e0k6Q4x3X3g2Z5N6r3#2D9</mark> 帖子大佬分析的执行流程图）

![图片描述](upload/attach/202304/941979\_P56JKBJC39NNNRB.png)

### 6. 在`unidbg`复现过程中，在第一个`10101`的调用时候，返回值一直是`null`，而不是`hook`的`0`，经过多方测试，最终在`unidbg`低版本并且对`32`位`so`进行模拟执行时，会返回`0`（猜测：新版本`unidbg`应该是帮我们补了一些环境，而旧版本则没有，所以旧版本运行会提示补环境，照补即可，这里补充环境参考了以下两个链接的帖子：<mark class="encrypted">34eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8V1A6G2K9r3&6Q4y4h3k6x3k6h3&6G2L8W2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3x3e0t1&6y4e0M7J5x3U0p5%4</mark> 和 <mark class="encrypted">45fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4q4I4i4K6g2X3x3K6t1&6y4e0f1J5x3U0y4Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0p5J5x3o6f1H3x3o6x3#2x3g2!0q4c8W2!0n7b7#2)9^5z5b7`.`.</mark>

### 7. `unidbg`代码如下：
